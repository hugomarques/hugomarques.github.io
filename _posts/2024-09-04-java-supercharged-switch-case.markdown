---
title: "Java: O poderoso `switch case` do Java 17 🚀"
published: true
description: Aprenda as facilidades do novo switch case do Java 17+
tags: java, iniciante, java21, braziliandevs
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2024-09-04 17:38 +0000
---

> Disclaimer: Esse post apareceu primeiro no [meu bluesky](https://bsky.app/profile/hellohugomarques.bsky.social/post/3l3ds62fabr22).

O `switch case` no Java 17 e 21 trouxe várias melhorias que vão facilitar a vida de quem programa em Java. De compile-time errors ao `yield`, temos bastante coisa bacana pra explorar! Vamos conferir as principais novidades! 🎉

### 1. Esqueça o `default`!

Agora o compilador avisa se você esquecer um case, tornando o código mais seguro. Se todos os valores possíveis forem cobertos, você pode deixar de lado o `default`. O melhor de tudo? Compile-time errors garantem que nenhum cenário fique de fora! 🎯

Vamos dar uma olhada em um exemplo que mostra um erro de compilação quando um valor é adicionado à enum, mas não é coberto no `switch case`.

```java
enum Shape { CIRCLE, SQUARE }

// Depois adicionamos TRIANGLE à enum
enum Shape { CIRCLE, SQUARE, TRIANGLE }

Shape shape = ...;
switch (shape) {
    case CIRCLE -> System.out.println("It's a circle!");
    case SQUARE -> System.out.println("It's a square!");
}
```

Aqui, o compilador irá lançar um erro dizendo que o valor `TRIANGLE` não foi tratado no `switch`. Isso acontece porque todos os valores da enum precisam ser cobertos no `switch case` — garantindo que nenhum cenário fique de fora e mantendo o código mais robusto e seguro! Sem o `default`, você terá certeza de que todos os casos estão sendo tratados. ✨

Erro de compilação:
```
Error: The switch statement does not cover all possible values of the enum: TRIANGLE
```

### 2. Yield!

Agora o `switch` pode retornar valores com `yield`, deixando o código mais limpo e expressivo. Esqueça os retornos confusos e aproveite a simplicidade. O resultado? Código mais claro e direto! 🔄

### Exemplo com `yield`

```java
String message = switch (day) {
    case MONDAY -> {
        System.out.println("Starting the week!");
        yield "Back to work!";
    }
    case FRIDAY -> {
        System.out.println("Weekend is coming!");
        yield "Almost there!";
    }
    case SATURDAY, SUNDAY -> "Enjoy the weekend!";
};
```

Aqui, o `yield` permite que retornemos valores diretamente no switch sem complicações. 🚀

### 3. Suporte para Enums e Sealed Types

O suporte para `enums` e `sealed types` no `switch case` foi melhorado! Agora, `enums` são tratados de maneira mais fluida, e com `sealed types`, podemos garantir que todos os subtipos sejam cobertos no switch, tornando o código mais previsível e seguro. 🚀

### Exemplo com Sealed Types

```java
sealed interface Shape permits Circle, Square {
    // Pode adicionar métodos ou propriedades comuns
}

final class Circle implements Shape {
    // Implementação da classe Circle
}

final class Square implements Shape {
    // Implementação da classe Square
}

Shape shape = ...;
switch (shape) {
    case Circle c -> System.out.println("It's a circle with radius: " + c.getRadius());
    case Square s -> System.out.println("It's a square with side: " + s.getSide());
}
```

Neste exemplo, `Shape` é uma `sealed interface` e garantimos que todas as possíveis subclasses de `Shape` (no caso, `Circle` e `Square`) sejam cobertas no `switch case`. O compilador ajuda a garantir que não deixemos nenhum caso de fora, aumentando a segurança do código. 🔒

### 4. Desconstrução de Tipos com Pattern Matching

E não para por aí! Também temos a **desconstrução de tipos** no `switch case`, o que nos permite fazer `pattern matching` diretamente nos objetos. Isso torna o código mais poderoso e flexível.

### Exemplo com Pattern Matching

```java
switch (obj) {
    case Point(int x, int y) -> System.out.println("Point with coordinates: (" + x + ", " + y + ")");
    case Circle c -> System.out.println("Circle with radius: " + c.getRadius());
}
```

O poder do pattern matching nos dá um controle maior sobre os tipos! 🔥

---

### Fiquem Ligados! 🚨

Em breve, vamos falar sobre **Sealed Types** e como, junto com o novo `switch`, eles abrem um novo paradigma de programação no Java. 👀.