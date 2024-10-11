---
title: 'Testes - Testando das trincheiras: Usando um "clock" fixo'
published: true
description: Vamos aprender como usar um relógio fixo para tornar nossas classes mais testáveis.
tags: #java #testes #oop #patterns
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fkyo1zxrt96bhev8c0h0.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2023-12-17 14:47 +0000
---

Outro curtinho sobre testes. Um dos problemas mais comuns que eu vejo é o uso do tempo variável dentro do código. Como assim? Imagine o seguinte exemplo:

```java
@Component
public class TaskScheduler {

    private static final LocalTime START_OF_WORKING_DAY = LocalTime.of(8, 0);
    private static final LocalTime END_OF_WORKING_DAY = LocalTime.of(22, 0);

    public void scheduleTask(Task task) {
        if (shouldSchedule()) {
            executeTaskNow(task);
        }
    }

    public static boolean shouldSchedule() {
        // Get the current time in the system default time zone
        LocalDateTime now = LocalDateTime.ofInstant(Instant.now(), ZoneId.systemDefault());
        LocalTime currentTime = now.toLocalTime();

        // Check if the current time is within the working hours
        return !currentTime.isBefore(START_OF_WORKING_DAY) && !currentTime.isAfter(END_OF_WORKING_DAY);
    }
}
```

Qual o problema com o código acima? Devido ao `Instant.now()` no meio do seu código, você não consegue testar o seu método! Como sua lógica é não-determinística e depende do tempo, o seu teste vai passar/falhar conforme o horário que o teste é executado.

## Como corrigir esse problema?

Uma alternativa bem simples a partir do java 8 é utilizar a classe `Clock` para injetar sua dependência que controla o tempo. 

No nosso exemplo acima, nosso código ficaria:

```java
@Component
public class TaskScheduler {

    private static final LocalTime START_OF_WORKING_DAY = LocalTime.of(8, 0);
    private static final LocalTime END_OF_WORKING_DAY = LocalTime.of(22, 0);

   private final Clock clock;
 
   @Autowired
   public TaskScheduler(Clock clock) {
     this.clock = clock;
   }

    public void scheduleTask(Task task) {
        if (shouldSchedule()) {
            executeTaskNow(task);
        }
    }

    public static boolean shouldSchedule() {
        // Get the current time in the current clock
        LocalDateTime now = LocalDateTime.ofInstant(clock);
        LocalTime currentTime = now.toLocalTime();

        // Check if the current time is within the working hours
        return !currentTime.isBefore(START_OF_WORKING_DAY) && !currentTime.isAfter(END_OF_WORKING_DAY);
    }
}
```

Dessa forma, você consegue escrever os testes passando o `Clock` com o tempo que você deseja.

```java
    @Test
    public void testIsNowWithinWorkingHours_withinHours() {
        // Arrange: set a fixed instant within working hours
        Instant fixedInstant = LocalDateTime.of(2024, 6, 1, 10, 0)
                                            .toInstant(ZoneOffset.UTC);
        Clock fixedClock = Clock.fixed(fixedInstant, ZoneId.systemDefault());

        TaskScheduler t = new TaskScheduler(fixedClock);

        // Act: call the method with the fixed clock
        boolean result = t.shouldSchedule();

        // Assert: should be within working hours
        assertTrue(result, "The time should be within working hours");
    }
```

## Dicas interessantes!

### 1. Eu uso `Spring`, como eu crio esse `Clock` pra ser injetado?

Simples, você pode declarar o seu Clock padrão pro sistema em uma classe de `@Configuration`. 

```java
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

import java.time.Clock;

@Configuration
public class AppConfiguration {

    @Bean
    public Clock clock() {
        // Retorna o relógio do sistema na zona padrão do sistema
        return Clock.systemDefaultZone();
    }
}
```

E aí na sua classe é só fazer o `@Autowired` no construtor que nem fizemos no nosso exemplo acima.

### 2. Eu uso o meu construtor default em 50 locais diferentes, eu vou ter que alterar todos esses locais pra injetar o Clock agora?
 
Nada jovem padawan! Um truque bacana é fazer um overloaded constructor:

```java

@Component
public class TaskScheduler {

    private static final LocalTime START_OF_WORKING_DAY = LocalTime.of(8, 0);
    private static final LocalTime END_OF_WORKING_DAY = LocalTime.of(22, 0);

   private final Clock clock;
 
   // Essa anotação fala pro nosso Spring da massa usar esse construtor
   @Autowired 
   public TaskScheduler() {
     this.clock = Clock.systemDefaultZone();
   } 
   
  // Esse construtor aqui a gente pode usar pros testes.
   public TaskScheduler(Clock clock) {
     this.clock = clock;
   }
```

E pronto! Com os dois construtores, você mantém a classe funcionando onde ela já existia, além de permitir a escrita de testes automatizados de forma simples.

## Sumário

1. Evite o uso de tempo variável no meio do código.
2. Use injeção de dependências para adicionar o seu relógio.
3. Use construtores padrões e sobrecarga no construtor para permitir adicionar os testes com o mínimo de refatoramento.

Espero que vocês estejam curtindo essas dicas rápidas sobre testes. 

Em breve, vou escrever também meus aprendizados sobre paralelismo! 

Happy coding!

