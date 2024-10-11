---
title: "Java: Quem comeu o meu CompletableFuture?"
published: true
description: Post curtinho sobre onde CompletableFuture são executados. 
tags: #java #thread #concurrency #completableFuture
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2024-07-19 18:13 +0000
---

O `CompletableFuture` é uma classe do Java que faz parte do pacote `java.util.concurrent` e é utilizada para trabalhar com programação assíncrona. Ele permite que você execute tarefas de forma assíncrona e, posteriormente, combine os resultados dessas tarefas ou execute ações adicionais quando elas forem concluídas. Com `CompletableFuture`, você pode encadear várias operações assíncronas, lidar com exceções e combinar múltiplas tarefas de maneira eficiente. Isso é particularmente útil em aplicações que precisam realizar operações demoradas, como chamadas a APIs externas ou processamento de dados intensivo, sem bloquear a thread principal.

## Olhando a API 👀

Se olharmos a API de `CompletableFuture` nós vamos perceber algo bem curioso. Uma porrada dos métodos tem 3 versões, por exemplo:
1. thenApply(Function<? super T,? extends U> fn)
2. thenApplyAsync(Function<? super T,? extends U> fn)
3. 
thenApplyAsync(Function<? super T,? extends U> fn, Executor executor)

Você sabia que dependendo do método que você encadear o seu `CompletableFuture` ele vai executar em um thread-pool completamente diferente? Vamos aos exemplos...

## Execução na mesma thread de invocação

```java
class Scratch {
    public static void main(String[] args) {
       mainThread();
    }

    public static void mainThread() {
        CompletableFuture<String> future = CompletableFuture.completedFuture("Hello World!");

        // thenApply is executed in the main thread
        String result = future.thenApply(s -> {
            System.out.println("ThenApply: " + Thread.currentThread().getName());
            return s + " World";
        }).join();  // join is used to wait for the future to complete
        System.out.println(result);
    }
}
```

Se rodarmos o exemplo acima veremos que o nosso `Hello World!` é executado na mesma thread que invocou o `CompletableFuture`, a `main-thread`. Por quê? Porque usamos o mesmo `thenApply` sem usarmos a versão async que executa em outras thread-pools.

## Execução no ForkJoinPool

Agora, o que acontece se usarmos a segunda versão?

```java
class Scratch {
    public static void main(String[] args) {
       forkJoinPool();
    }

    public static void forkJoinPool() {
        CompletableFuture<String> future = CompletableFuture.completedFuture("Hello World!");

        // thenApply is executed in the main thread
        String result = future.thenApplyAsync(s -> {
            System.out.println("ThenApply: " + Thread.currentThread().getName());
            return s + " World";
        }).join();  // join is used to wait for the future to complete
        System.out.println(result);
    }
}
```

Note que ao executarmos com a versão `async` o `CompletableFuture` é executado no `ForkJoinPool`. Se você leu meus artigos anteriores sobre Paralelismo você deve estar familiarizada(o) com o `ForkJoinPool`. 

## Execução em um custom executor

Finalmente, a última versão nos permite passar um executor de nossa escolha. Essa é minha versão favorita já que me permite maior controle sobre minha aplicação.

```java
class Scratch {
    public static void main(String[] args) {
        customPool();
    }

    public static void customPool() {
        final ExecutorService executorService = Executors.newFixedThreadPool(2);
        CompletableFuture<String> future = CompletableFuture.completedFuture("Hello World!");

        // thenApply is executed in the main thread
        String result = future.thenApplyAsync(s -> {
            System.out.println("ThenApply: " + Thread.currentThread().getName());
            return s + " World";
        }, executorService).join();  // join is used to wait for the future to complete
        System.out.println(result);
    }

}
```

## Sumário

Nesse pequeno post aprendemos as diversas maneiras que um `CompletableFuture`. Conhecê-las é extremamente útil pra evitar surpresas, como recentemente eu enfrentei ao executar meu `CompletableFuture` com um `thenApply` e achar estranho a thread que tava executando aquele código.

Esse post é curtinho, espero que vocês curtam esse formato! Happy coding 💻.