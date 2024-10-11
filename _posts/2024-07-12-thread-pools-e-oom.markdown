---
title: "Java: Thread Pools e OOM"
published: true
description: Um exemplo na prática de como utilizar uma única thread pool pode causar um out-of-memory error (OOM).
tags: #java #concurrency #threads #error
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2024-07-12 18:05 +0000
---

Em sistemas que utilizam processamento assíncrono, como chamadas gRPC não bloqueantes, é comum encontrar cenários onde o gerenciamento inadequado de pools de threads pode levar a problemas de desempenho e até mesmo a um OutOfMemoryError (OOM). Um caso típico ocorre quando o mesmo pool de threads é utilizado tanto para enviar requisições quanto para processar as respostas. Isso pode resultar em um acúmulo de tarefas pendentes que consomem toda a memória disponível, causando um OOM.

Esse artigo descreve o problema que eu passei a semana investigando e como podemos atacá-lo! Vamos lá 👊!

## Simulando o problema

Vamos considerar um exemplo onde um FixedThreadPool é utilizado para enviar milhões de requisições a um "fake" cliente gRPC e, simultaneamente, processar as respostas dessas requisições. O código abaixo demonstra como esse cenário pode levar a um OOM:

```java
public class ThreadPoolsOOMExample {
    private static final int THREAD_POOL_SIZE = 10;
    private static final int TOTAL_TASKS = 1_000_000;

    private final ExecutorService mainExecutor = Executors.newFixedThreadPool(THREAD_POOL_SIZE);

    public static void main(String[] args) {
        ThreadPoolsOOMExample example = new ThreadPoolsOOMExample();
        example.run();
    }

    public void run() {
        for (int i = 0; i < TOTAL_TASKS; i++) {
            mainExecutor.submit(this::submitRequest);
        }
    }

    private void submitRequest() {
        // Simula o envio de uma requisição para o cliente gRPC
        CompletableFuture<Response> future = asyncGrpcCall();

        // Processa a resposta usando o mesmo executor
        future.thenApplyAsync(this::processResponse, mainExecutor);
    }

    private CompletableFuture<Response> asyncGrpcCall() {
        // Simula uma chamada gRPC assíncrona
        CompletableFuture<Response> future = new CompletableFuture<>();
        new Thread(() -> {
            try {
                Thread.sleep(100); // Simula o atraso da rede
                future.complete(new Response(512));
            } catch (InterruptedException e) {
                future.completeExceptionally(e);
            }
        }).start();
        int queueSize = ((ThreadPoolExecutor) mainExecutor).getQueue().size();
        System.out.println("Current queue size: " + queueSize);
        printHeapSize();
        return future;
    }

    private Response processResponse(Response response) {
        // Processa a resposta do cliente gRPC
        System.out.println("Processando resposta");
        return response;
    }

    private void printHeapSize() {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();

        System.out.println("Heap size (total): " + totalMemory / (1024 * 1024) + " MB");
        System.out.println("Heap size (used): " + usedMemory / (1024 * 1024) + " MB");
        System.out.println("Heap size (max): " + maxMemory / (1024 * 1024) + " MB");
    }

    public static class Response {
        private byte[] data;

        public Response(int sizeInKB) {
            this.data = new byte[sizeInKB * 1024]; // 1 KB = 1024 bytes
        }

        public byte[] getData() {
            return data;
        }
    }
}
```

Neste exemplo, o mainExecutor é utilizado tanto para enviar requisições quanto para processar as respostas. Como o número de tarefas é muito grande (TOTAL_TASKS = 1000000), as respostas ficam acumuladas no final da fila do mainExecutor, esperando que todas as requisições sejam enviadas primeiro. Isso leva a um acúmulo de respostas pendentes, consumindo muita memória e eventualmente causando um OutOfMemoryError.

## Possíveis soluções

### 1. Use Pools de Threads Separados
Uma solução eficaz é utilizar pools de threads separados para a produção de requisições e o processamento de respostas. Isso garante que as respostas possam ser processadas independentemente do envio de novas requisições, evitando o acúmulo de tarefas na fila.

```java
package com.hugodesmarques.threads;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.ThreadPoolExecutor;

public class SeparateThreadPoolsExample {
    private static final int PRODUCER_THREAD_POOL_SIZE = 10;
    private static final int CONSUMER_THREAD_POOL_SIZE = 10;
    private static final int TOTAL_TASKS = 1_000_000;

    private final ExecutorService producerThreadPool = Executors.newFixedThreadPool(PRODUCER_THREAD_POOL_SIZE);
    private final ExecutorService consumerThreadPool = Executors.newFixedThreadPool(CONSUMER_THREAD_POOL_SIZE);

    public static void main(String[] args) {
        SeparateThreadPoolsExample example = new SeparateThreadPoolsExample();
        example.run();
    }

    public void run() {
        for (int i = 0; i < TOTAL_TASKS; i++) {
            producerThreadPool.submit(this::submitRequest);
            int producerQueueSize = ((ThreadPoolExecutor) producerThreadPool).getQueue().size();
            System.out.println("Producer queue size: " + producerQueueSize);
        }
    }

    private void submitRequest() {
        // Simula o envio de uma requisição para o cliente gRPC
        CompletableFuture<Response> future = asyncGrpcCall();

        // Processa a resposta usando o mesmo executor
        future.thenApplyAsync(this::processResponse, consumerThreadPool);
    }

    private CompletableFuture<Response> asyncGrpcCall() {
        // Simula uma chamada gRPC assíncrona
        CompletableFuture<Response> future = new CompletableFuture<>();
        new Thread(() -> {
            try {
                Thread.sleep(100); // Simula o atraso da rede
                future.complete(new Response(512));
            } catch (InterruptedException e) {
                future.completeExceptionally(e);
            }
        }).start();
        int consumerQueueSize = ((ThreadPoolExecutor) consumerThreadPool).getQueue().size();
        System.out.println("Consumer queue size: " + consumerQueueSize);
        printHeapSize();
        return future;
    }

    private Response processResponse(Response response) {
        // Processa a resposta do cliente gRPC
        System.out.println("Processando resposta");
        return response;
    }

    private void printHeapSize() {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();

        System.out.println("Heap size (total): " + totalMemory / (1024 * 1024) + " MB");
        System.out.println("Heap size (used): " + usedMemory / (1024 * 1024) + " MB");
        System.out.println("Heap size (max): " + maxMemory / (1024 * 1024) + " MB");
    }

    public static class Response {
        private byte[] data;

        public Response(int sizeInKB) {
            this.data = new byte[sizeInKB * 1024]; // 1 KB = 1024 bytes
        }

        public byte[] getData() {
            return data;
        }
    }
}
```

### 2. Use um Executor Com Capacidade Limitada
Outra abordagem é limitar a capacidade do ThreadPoolExecutor para evitar que ele aceite mais tarefas do que pode processar. Isso pode ser feito usando um BlockingQueue com capacidade limitada e uma política de rejeição adequada.

```java
package com.hugodesmarques.threads;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.RejectedExecutionHandler;
import java.util.concurrent.ThreadFactory;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class ThreadPoolWithLimitsExample {

    private static final int TOTAL_TASKS = 1_000_000;
    private static final int THREAD_POOL_SIZE = 10;
    private static final int QUEUE_CAPACITY = 100;

    private final ExecutorService mainExecutor = new ThreadPoolExecutor(
            THREAD_POOL_SIZE,
            THREAD_POOL_SIZE,
            0L,
            TimeUnit.MILLISECONDS,
            new ArrayBlockingQueue<>(QUEUE_CAPACITY),
            new NamedThreadFactory("Producer"), // ThreadFactory com expressão lambda)
            new RejectionLoggingPolicy(new ThreadPoolExecutor.CallerRunsPolicy()) // Política de rejeição com log
    );

    public static void main(String[] args) {
        ThreadPoolWithLimitsExample example = new ThreadPoolWithLimitsExample();
        example.run();
    }

    public void run() {
        for (int i = 0; i < TOTAL_TASKS; i++) {
            mainExecutor.submit(this::submitRequest);
        }
    }

    private void submitRequest() {
        // Simula o envio de uma requisição para o cliente gRPC
        CompletableFuture<Response> future = asyncGrpcCall();

        // Processa a resposta usando o mesmo executor
        future.thenApplyAsync(this::processResponse, mainExecutor);
    }

    private CompletableFuture<Response> asyncGrpcCall() {
        // Simula uma chamada gRPC assíncrona
        CompletableFuture<Response> future = new CompletableFuture<>();
        new Thread(() -> {
            try {
                Thread.sleep(100); // Simula o atraso da rede
                future.complete(new Response(512));
            } catch (InterruptedException e) {
                future.completeExceptionally(e);
            }
        }).start();
        int queueSize = ((ThreadPoolExecutor) mainExecutor).getQueue().size();
        System.out.println("Current queue size: " + queueSize);
        printHeapSize();
        return future;
    }

    private Response processResponse(Response response) {
        // Processa a resposta do cliente gRPC
        System.out.println("Processando resposta... thread: " + Thread.currentThread().getName());
        return response;
    }

    private void printHeapSize() {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();

        System.out.println("Heap size (total): " + totalMemory / (1024 * 1024) + " MB");
        System.out.println("Heap size (used): " + usedMemory / (1024 * 1024) + " MB");
        System.out.println("Heap size (max): " + maxMemory / (1024 * 1024) + " MB");
    }

    // Política de rejeição personalizada que registra um log quando uma tarefa é rejeitada
    static class RejectionLoggingPolicy implements RejectedExecutionHandler {
        private final RejectedExecutionHandler handler;

        public RejectionLoggingPolicy(RejectedExecutionHandler handler) {
            this.handler = handler;
        }

        @Override
        public void rejectedExecution(Runnable r, ThreadPoolExecutor executor) {
            System.out.println("Tarefa rejeitada: " + r.toString() + " thread: " + Thread.currentThread().getName());
            handler.rejectedExecution(r, executor);
        }
    }

    // ThreadFactory personalizado para nomear threads
    static class NamedThreadFactory implements ThreadFactory {
        private final AtomicInteger threadNumber = new AtomicInteger(1);
        private final String namePrefix;

        public NamedThreadFactory(String namePrefix) {
            this.namePrefix = namePrefix;
        }

        @Override
        public Thread newThread(Runnable r) {
            return new Thread(r, namePrefix + "-thread-" + threadNumber.getAndIncrement());
        }
    }

    public static class Response {
        private byte[] data;

        public Response(int sizeInKB) {
            this.data = new byte[sizeInKB * 1024]; // 1 KB = 1024 bytes
        }

        public byte[] getData() {
            return data;
        }
    }
}
```

### 3. Controle o fluxo de produção com o uso de semáforos

Implementar controle de fluxo é uma abordagem eficaz para garantir que a produção de novas requisições não ultrapasse a capacidade do sistema de processar respostas. Isso pode ser feito usando semáforos (Semaphore) ou outras técnicas de controle de fluxo. A ideia é limitar o número de requisições simultâneas para evitar sobrecarregar o sistema.

```java
package com.hugodesmarques.threads;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Semaphore;

public class ThreadPoolsWithSemaphores {
    private static final int THREAD_POOL_SIZE = 10;
    private static final int TOTAL_TASKS = 1_000_000;
    private static final int MAX_CONCURRENT_REQUESTS = 100;
    private final Semaphore semaphore = new Semaphore(MAX_CONCURRENT_REQUESTS);

    private final ExecutorService mainExecutor = Executors.newFixedThreadPool(THREAD_POOL_SIZE);

    public static void main(String[] args) {
        ThreadPoolsWithSemaphores example = new ThreadPoolsWithSemaphores();
        example.run();
    }

    public void run() {
        for (int i = 0; i < TOTAL_TASKS; i++) {
            try {
                System.out.println("Submetendo request: " + i);
                semaphore.acquire();
                mainExecutor.submit(this::submitRequest);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }

    private void submitRequest() {
        try {
            // Simula o envio de uma requisição para o cliente gRPC
            CompletableFuture<Response> future = asyncGrpcCall();

            // Processa a resposta usando o mesmo executor
            future.thenApplyAsync(this::processResponse, mainExecutor)
                    .whenComplete((result, ex) -> semaphore.release());
        } catch (Exception e) {
            semaphore.release();
        }
    }

    private CompletableFuture<Response> asyncGrpcCall() {
        // Simula uma chamada gRPC assíncrona
        CompletableFuture<Response> future = new CompletableFuture<>();
        new Thread(() -> {
            try {
                Thread.sleep(100); // Simula o atraso da rede
                future.complete(new Response(512));
            } catch (InterruptedException e) {
                future.completeExceptionally(e);
            }
        }).start();
        printHeapSize();
        return future;
    }

    private Response processResponse(Response response) {
        // Processa a resposta do cliente gRPC
        System.out.println("Processando resposta");
        return response;
    }

    private void printHeapSize() {
        Runtime runtime = Runtime.getRuntime();
        long totalMemory = runtime.totalMemory();
        long freeMemory = runtime.freeMemory();
        long usedMemory = totalMemory - freeMemory;
        long maxMemory = runtime.maxMemory();

        System.out.println("Heap size (total): " + totalMemory / (1024 * 1024) + " MB");
        System.out.println("Heap size (used): " + usedMemory / (1024 * 1024) + " MB");
        System.out.println("Heap size (max): " + maxMemory / (1024 * 1024) + " MB");
    }

    public static class Response {
        private byte[] data;

        public Response(int sizeInKB) {
            this.data = new byte[sizeInKB * 1024]; // 1 KB = 1024 bytes
        }

        public byte[] getData() {
            return data;
        }
    }
}
```

## Conclusão

O gerenciamento adequado de pools de threads em sistemas assíncronos é crucial para evitar problemas de desempenho e OutOfMemoryError (OOM). Utilizar pools de threads separados, limitar a capacidade do ThreadPoolExecutor e implementar controle de fluxo são abordagens eficazes para garantir que a produção de novas requisições não ultrapasse a capacidade do sistema de processar respostas.

Ao aplicar essas soluções, você pode garantir que seu sistema permaneça eficiente e responsivo, mesmo sob alta carga, evitando o acúmulo excessivo de tarefas na fila e prevenindo OOM.

Todos os exemplos deste post estão disponíveis no meu repositório: <a href="https://github.com/hugomarques" target="_blank" style="text-decoration: none;">
  <i class="fab fa-fw fa-github"></i> GitHub
</a>

Fica a dica, pra você não passar pela mesma dor de cabeça que eu passei 😅.