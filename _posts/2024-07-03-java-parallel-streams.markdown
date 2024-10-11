---
title: "Paralelismo e Concorrência 102: Java parallel streams na prática"
published: true
description: Frite sua CPU com o poder do parallel streams
tags: #java #parallel #concurrency #threads
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/xfszopvhn1gmdbe085z8.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2024-07-03
---

No artigo anterior, ["Paralelismo e Concorrência 101"](https://dev.to/hugaomarques/paralelismo-e-concorrencia-101-2pgc), exploramos os conceitos fundamentais sobre esses dois tópicos. Discutimos como esses conceitos permitem que programas realizem múltiplas tarefas simultaneamente, melhorando o desempenho e a eficiência.

Neste segundo artigo da série, vamos nos aprofundar no uso do `parallel stream` em Java. Introduzido no Java 8, o `parallel stream` é uma funcionalidade que facilita o processamento paralelo de coleções, aproveitando múltiplos núcleos da CPU para melhorar o desempenho de operações em grandes volumes de dados. 

Vamos explorar como o `parallel stream` funciona, suas vantagens e desvantagens, e como personalizar o pool de threads utilizado. Também discutiremos a técnica de "work-stealing" implementada pelo `ForkJoinPool` e sua importância para o balanceamento de carga e eficiência.

Vamos lá que temos muito conteúdo pra cobrir!

## Tabela de conteúdo
- [Parallel streams: Paralelismo de forma fácil](#parallel-stream-paralelismo-de-forma-fácil)
- [Paralelismo e concorrência](#paralelismo-e-concorrência)
- [ ForkJoinPool - é o que homi 😳?](#forkjoinpool-é-o-que-homi-)
- [Work-stealing no ForkJoinPool](#-raw-workstealing-endraw-no-forkjoinpool)
- [Performance entre Sequential e Parallel](#performance-entre-raw-sequential-endraw-e-raw-parallel-endraw-)
- [Conclusão](#conclusão)

## Parallel stream: Paralelismo de forma fácil

O Java 8 introduziu streams como uma nova forma de iterar e realizar operações em coleções de forma declarativa. Os streams fornecem uma API rica para manipulação de dados, permitindo operações como filtro, mapeamento, redução e muito mais.

Os streams podem ser criados no modo sequencial ou no modo de execução paralela. Vejamos como criar e usar ambos os tipos de streams com exemplos.

### Sequential

```java
import java.util.Arrays;
import java.util.List;

public class SequentialStreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Criando um stream sequencial
        numbers.stream()
               .forEach(n -> System.out.println("Thread: " + Thread.currentThread().getName() + " - Número: " + n));
    }
}
```

Neste exemplo, o método stream() cria um stream sequencial a partir da lista de números. A operação `forEach` itera sobre cada elemento da lista e imprime o número juntamente com o nome da thread que está processando o elemento. Como é um stream sequencial, todos os elementos são processados pela mesma thread.

Se rodarmos o exemplo acima, nós veremos o seguinte resultado:

```java
➜  sandbox java SequentialStreamExample      
Thread: main - Número: 1
Thread: main - Número: 2
Thread: main - Número: 3
Thread: main - Número: 4
Thread: main - Número: 5
Thread: main - Número: 6
Thread: main - Número: 7
Thread: main - Número: 8
Thread: main - Número: 9
Thread: main - Número: 10
```

### Parallel

Agora vamos rodar um exemplo com `parallel`. Para isso, basta criarmos o nosso stream com `parallel stream`:

```java
import java.util.Arrays;
import java.util.List;

public class ParallelStreamExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Criando um stream paralelo
        numbers.parallelStream()
                .forEach(n -> System.out.println("Thread: " + Thread.currentThread().getName() + " - Número: " + n));
    }
}
```

Se executarmos esse exemplo múltiplas vezes, a ordem sempre vai ser algo diferente:

```java
➜  sandbox java ParallelStreamExample  
Thread: ForkJoinPool.commonPool-worker-2 - Número: 2
Thread: main - Número: 7
Thread: ForkJoinPool.commonPool-worker-6 - Número: 4
Thread: ForkJoinPool.commonPool-worker-3 - Número: 5
Thread: ForkJoinPool.commonPool-worker-4 - Número: 9
Thread: ForkJoinPool.commonPool-worker-1 - Número: 3
Thread: ForkJoinPool.commonPool-worker-9 - Número: 10
Thread: ForkJoinPool.commonPool-worker-8 - Número: 6
Thread: ForkJoinPool.commonPool-worker-7 - Número: 8
Thread: ForkJoinPool.commonPool-worker-5 - Número: 1
```

Existem outras formas de criar streams, tanto sequenciais quanto paralelos, mas eu deixo isso como dever de casa pra vocês 🤓.

## Paralelismo e concorrência

Agora que vimos um exemplo simples com `parallel`, como ele se relaciona com os conceitos que discutimos no [artigo anterior de paralelismo e concorrência](https://dev.to/hugaomarques/paralelismo-e-concorrencia-101-2pgc)?

### Como o `parallel stream` utiliza concorrência?

Quando você transforma um Stream em um `parallel stream`, se possível, o Java divide a tarefa em várias subtarefas que podem ser executadas simultaneamente. Cada subtarefa é atribuída a uma thread separada, que pode ser executada em um núcleo diferente do processador. O gerenciamento dessas threads envolve conceitos de concorrência, como:

* Thread Pool: O `ForkJoinPool` é frequentemente usado para gerenciar threads em `parallel stream`.
* Sincronização: Garantia de que as operações em dados compartilhados são seguras para threads.
* Troca de contexto: Através do work-stealing o `ForkJoinPool` pega tasks da fila de threads ocupadas para serem executadas em threads que estão desocupadas.

### Como o `parallel stream` utiliza paralelismo?

O `parallel stream` aproveita o paralelismo ao executar essas subtarefas simultaneamente em múltiplos núcleos. Isso pode resultar em uma execução mais rápida, especialmente para operações que são independentes e podem ser realizadas em paralelo sem interferência entre si.

## ForkJoinPool - é o que homi 😳 ? 

Por padrão, o `parallel stream` utiliza o `ForkJoinPool.commonPool()`, que é um pool de threads compartilhado disponível para todas as tarefas de fork/join. Esse pool é configurado para usar um número de threads igual ao número de núcleos disponíveis no processador, o que permite que as tarefas sejam executadas em paralelo de maneira eficiente.

Se quisermos verificar quantas threads o nosso `forkJoinPool` terá, basta fazermos print do número de cores disponíveis no Java. Por exemplo, se eu rodar a seguinte linha em qualquer programa Java no meu mac:

```java
System.out.println("Number of cores available: "+ Runtime.getRuntime().availableProcessors());

Number of cores available: 12
```

Notem, que o número acima pode ser mais complicado um pouco, em especial, quando se envolve containers.

### Modificando o tamanho do `ForkJoinPool` padrão

> 🛑 Dica: NÃO faça isso! 

O tamanho padrão do `ForkJoinPool.commonPool()` pode ser modificado configurando a propriedade do sistema `java.util.concurrent.ForkJoinPool.common.parallelism`. Isso pode ser feito ao iniciar a JVM com a opção -D, por exemplo:

```java
java -Djava.util.concurrent.ForkJoinPool.common.parallelism=8 MinhaAplicacao
```

Este comando configura o `ForkJoinPool.commonPool()` para utilizar 8 threads.

### Por que você **NÃO** deve modificar o tamanho padrão do ForkJoinPool?

Modificar o tamanho padrão do `ForkJoinPool.commonPool()` pode afetar negativamente outras partes da aplicação ou bibliotecas que também utilizam o pool comum. 

O `commonPool` é um recurso compartilhado, e alterar seu comportamento pode introduzir problemas de desempenho e concorrência difíceis de diagnosticar. Em vez disso, é recomendável criar e utilizar um `ForkJoinPool` personalizado para tarefas específicas que requerem paralelismo ajustado, garantindo que outras partes da aplicação permaneçam estáveis e previsíveis.

### Qual é uma alternativa melhor?

> ✅ Dica: Se necessário, faça isso!

Para modificar o pool de threads localmente você pode usar o método `ForkJoinPool#submit` para submeter uma tarefa que executa o `parallel stream` no contexto do pool personalizado.

```java
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.ForkJoinPool;

public class CustomForkJoinPoolExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Configurando o ForkJoinPool para usar um número específico de threads
        ForkJoinPool customThreadPool = new ForkJoinPool(4);

        try {
            customThreadPool.submit(() ->
                    numbers.parallelStream()
                            .forEach(n -> {
                                System.out.println("Thread: " + Thread.currentThread().getName() + " - Número: " + n);
                            })
            ).get();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            customThreadPool.shutdown();
        }
    }
}
```

Note que quando eu fiz o print anterior, eu tinha 12 cores disponíveis, porém observe que ao executar o código acima, eu só terei 4 threads.

```java
Thread: ForkJoinPool-1-worker-4 - Número: 6
Thread: ForkJoinPool-1-worker-4 - Número: 8
Thread: ForkJoinPool-1-worker-1 - Número: 7
Thread: ForkJoinPool-1-worker-2 - Número: 3
Thread: ForkJoinPool-1-worker-3 - Número: 9
Thread: ForkJoinPool-1-worker-3 - Número: 1
Thread: ForkJoinPool-1-worker-4 - Número: 10
Thread: ForkJoinPool-1-worker-1 - Número: 2
Thread: ForkJoinPool-1-worker-2 - Número: 5
Thread: ForkJoinPool-1-worker-3 - Número: 4
```

Também note que no primeiro exemplo, nós temos tarefas sendo executadas pela `main` thread. Porém, no `custom` nossas tarefas são executadas apenas pelos workers definidos no nosso pool.

Agora que sabemos sobre como criar `parallel stream` e onde eles são executados (`ForkJoinPool`), vamos discutir mais algumas coisas legais sobre essa funcionalidade.

## `Work-Stealing` no ForkJoinPool

Como mencionamos anteriormente, o `ForkJoinPool` é o pool de threads padrão utilizado pelo `parallel stream`. Esse pool implementa uma técnica chamada "work-stealing" (roubo de trabalho). Esta técnica é fundamental para garantir a eficiência e o balanceamento de carga entre as threads.

### Como Funciona o Work-Stealing?

* Divisão de Tarefas: Quando uma tarefa é submetida ao `ForkJoinPool`, ela é dividida em sub-tarefas menores distribuídas entre as threads do pool. Cada thread mantém uma fila de tarefas.
* Execução Local: Cada thread tenta executar as tarefas da sua própria fila. Se uma thread termina suas tarefas ou ficar ociosa, ela verifica se há mais trabalho a ser feito.
* Roubo de Trabalho: Se uma thread fica sem tarefas, ela tenta roubar tarefas das filas de outras threads. Isso é feito pegando tarefas do final da fila de outra thread, enquanto a própria thread vítima continua pegando tarefas do início de sua fila.

Este mecanismo de roubo de trabalho ajuda a manter todas as threads ocupadas e a balancear a carga de trabalho de maneira eficiente.

### Exemplo de Observação do Work-Stealing

Vamos criar um ForkJoinPool personalizado com um número específico de threads e simular tarefas com diferentes tempos de execução. O objetivo é observar como as threads ociosas roubam trabalho das threads ainda ocupadas.

```java
import java.util.Arrays;
import java.util.List;
import java.util.concurrent.ForkJoinPool;

public class WorkStealingExample {
    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

        // Configurando o ForkJoinPool para usar um número específico de threads
        ForkJoinPool customThreadPool = new ForkJoinPool(4);

        try {
            customThreadPool.submit(() ->
                    numbers.parallelStream()
                            .forEach(n -> {
                                try {
                                    if (n % 2 == 0) {
                                        // Simulando uma tarefa que leva tempo
                                        Thread.sleep(2000);
                                    } else {
                                        Thread.sleep(100);
                                    }
                                } catch (InterruptedException e) {
                                    e.printStackTrace();
                                }
                                System.out.println("Thread: " + Thread.currentThread().getName() + " - Número: " + n);
                            })
            ).get();
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            customThreadPool.shutdown();
        }
    }
}
```

Note que se você executar o código acima, vai haver uma tendência das tarefas ímpares terminarem primeiro. Por exemplo, uma das minhas execuções foi:

```java
Thread: ForkJoinPool-1-worker-2 - Número: 3
Thread: ForkJoinPool-1-worker-1 - Número: 7
Thread: ForkJoinPool-1-worker-3 - Número: 9
Thread: ForkJoinPool-1-worker-2 - Número: 5
Thread: ForkJoinPool-1-worker-4 - Número: 6
Thread: ForkJoinPool-1-worker-4 - Número: 1
Thread: ForkJoinPool-1-worker-3 - Número: 10
Thread: ForkJoinPool-1-worker-1 - Número: 2
Thread: ForkJoinPool-1-worker-2 - Número: 4
Thread: ForkJoinPool-1-worker-4 - Número: 8
```

E o que acontece se o `forkJoinPool` não tivesse `work-stealing`? Pois bem, as threads com tarefas menores terminariam suas tarefas primeiro e ficariam ociosas, enquanto as threads com tarefas maiores estariam ocupadas, gerando um gargalo desnecessário na aplicação.

Maravilha! Você deve estar pensando "Agora que eu sei tudo isso, eu sempre vou criar `parallel streams` para aumentar a performance das minhas aplicações!"

Calma jovem padawan, como sempre, depende...

## Performance entre `Sequential` e `Parallel`

Existe uma linha onde utilizar parallel não é interessante. O Java é excelente em otimizar o código da aplicação, então muitas vezes, um simples sequential stream() vai ter uma performance excelente! Vamos ver um exemplo com código:

```java
// Imports removidos pra manter o código breve. Veja o repo para o código completo.

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@State(Scope.Thread)
@Fork(value = 2)
public class SeqParallelBenchmark {

    @Param({"100", "1000000"})
    private int size;
    private List<Integer> data;

    @Setup
    public void setup() {
        data = IntStream.rangeClosed(1, size).boxed().collect(Collectors.toList());
    }

    @Benchmark
    public void test_sequential() {
        data.stream().mapToInt(Integer::intValue).sum();

    }

    @Benchmark
    public void test_parallel() {
        data.stream().parallel().mapToInt(Integer::intValue).sum();
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(SeqParallelBenchmark.class.getName()) // specify the benchmark class here
                .forks(2)
                .build();
        new Runner(opt).run();
    }
}

```

Quando eu rodei o benchmark acima na minha máquina eu obtive o seguinte resultado:

```java
Benchmark                              (size)  Mode  Cnt   Score    Error  Units
SeqParallelBenchmark.test_sequential      100  avgt   10  ≈ 10⁻⁴           ms/op
SeqParallelBenchmark.test_parallel        100  avgt   10   0.022 ±  0.008  ms/op

SeqParallelBenchmark.test_sequential  1000000  avgt   10   0.482 ±  0.016  ms/op
SeqParallelBenchmark.test_parallel    1000000  avgt   10   0.117 ±  0.016  ms/op
```

Observe que com 100 elementos, o método sequential roda em 0.0001 ms enquanto que o parallel leva 0.022 ms, quase 200x mais lento. Porém, quando rodamos para milhões de elemento, o parallel se sai 4x mais rápido to que o resultado sequencial.

Eu rodei outro benchmark com uma operação diferente dessa vez:

```java
// Imports removidos pra manter o código breve. Veja o repo para o código completo.

@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
@State(Scope.Thread)
public class AnotherSeqParallelBenchmark {

    @Param({"100", "1000000"})
    private int size;

    private List<Integer> data;

    @Setup
    public void setup() {
        data = IntStream.rangeClosed(1, size).boxed().collect(Collectors.toList());
    }

    @Benchmark
    public List<Double> testSequentialStream() {
        return data.stream()
                .map(Math::sin)
                .collect(Collectors.toList());
    }

    @Benchmark
    public List<Double> testParallelStream() {
        return data.parallelStream()
                .map(Math::sin)
                .collect(Collectors.toList());
    }

    public static void main(String[] args) throws RunnerException {
        Options opt = new OptionsBuilder()
                .include(AnotherSeqParallelBenchmark.class.getName()) // specify the benchmark class here
                .forks(2)
                .build();

        new Runner(opt).run();
    }
}
```

E o resultado foi bem similar:

```java
Benchmark                                          (size)  Mode  Cnt   Score    Error  Units
AnotherSeqParallelBenchmark.testSequentialStream      100  avgt   10   0.001 ±  0.001  ms/op
AnotherSeqParallelBenchmark.testParallelStream        100  avgt   10   0.027 ±  0.005  ms/op

AnotherSeqParallelBenchmark.testParallelStream    1000000  avgt   10   2.234 ±  0.025  ms/op
AnotherSeqParallelBenchmark.testSequentialStream  1000000  avgt   10  16.164 ±  0.573  ms/op
```

Note como mais uma vez em cima uma de collection pequena o parallel teve uma performance pior que o sequential (~27x pior). Porém, ao se deparar com milhões de cálculo a performance foi ~8x mais rápida do que sequential.

## Conclusão

**Não utilize `parallel`:**

1. **Se sua operação é I/O bound**: Seja uma chamada de rede para um microserviço ou escrita de arquivos. Por quê? Você estará utilizando muito mais o I/O do que a CPU, inclusive, você pode abrir muito mais operações do que apenas o número de núcleos da sua aplicação. Por exemplo, em um cenário recente eu abri ~500 conexões com outro microserviço 😱, muito acima dos 8 núcleos da minha máquina.
2. **Se a sua massa de dados é pequena**: `Parallel` adiciona um overhead, como vimos acima, com a utilização de uma thread pool, coordenação de tarefas, roubo de tarefas paradas, etc. Dado isso, se sua massa de dados for pequena, é provável que um `sequential` execute mais rápido do que um `parallel`. Na dúvida, execute testes na sua aplicação para observar se vale a pena.

**Utilize `parallel`:**

1. **Se o seu cenário é CPU bound**: Transformação de dados, cálculos, etc. Por quê? Se o número padrão de threads no `parallel stream` é o número de núcleos da máquina e sua operação é CPU bound, isso significa que você vai conseguir utilizar todos os núcleos em paralelo, aproveitando ao máximo os recursos disponíveis.
2. **Se as suas tarefas são independentes**: Como você quer usar paralelismo, é interessante que suas tarefas sejam independentes e isoladas, já que elas serão executadas em contextos diferentes e em uma ordem imprevisível.
3. **Se sua métrica mostra ganho de performance**: Realize alguns testes, por exemplo, utilizando uma ferramenta como JMH.

`Parallel streams` é um modo fácil de embarcar no mundo da programação paralela, já que a funcionalidade abstrai diversas preocupações. Ela não é aplicável para todos os casos, mas quando BEM aplicada pode ser uma mão na roda para aumentar a performance de suas aplicações.

Todos os exemplos deste post estão disponíveis no meu repositório: <a href="https://github.com/hugomarques" target="_blank" style="text-decoration: none;">
  <i class="fab fa-fw fa-github"></i> GitHub
</a>

No próximo post, vamos explorar mais o uso de threads tradicionais e thread pools com executors. Nos vemos por lá!
