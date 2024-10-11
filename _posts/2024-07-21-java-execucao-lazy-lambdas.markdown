---
title: "Java: Execução preguiçosa com Lambdas"
published: true
description: Lembre de que lambdas são executados de forma "lazy"
tags: #java #lambda #streams 
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2024-07-19 18:51 +0000
---

Outro post curtinho! Esta semana eu peguei um bug interessante que achei bacana compartilhar com vocês. O bug em questão me fez lembrar como lambdas são executados de forma "lazy" no Java.

Vamos para o exemplo?

Execução preguiçosa da forma errada
Vamos analisar o seguinte exemplo. Imagine que você tem um stream em Java que você só quer executar dentro de um bloco com lock.

```java
public class LazyStreamExample {
   private static final Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // Execução do Stream fora do Supplier
        System.out.println("Execução do Stream fora do Supplier");
        List<Integer> doubledNumbers = 
            numbers.stream()
                   .map(n -> {
                      System.out.println("Dobro de: " + n);
                      return n * 2;
                   })
                   .collect(Collectors.toList());

        writeToFileWithLock("output.txt", () -> doubledNumbers);
        System.out.println("Doubled Numbers: " + doubledNumbers);
    }

    private static void writeToFileWithLock(String filePath, Supplier<List<Integer>> supplier) {
        lock.lock();
        System.out.println("Lock adquirido!");
        try {
            List<Integer> result = supplier.get();
            String content = result.stream()
                                   .map(String::valueOf)
                                   .collect(Collectors.joining(", "));
            Files.write(Paths.get(filePath), content.getBytes(), StandardOpenOption.CREATE, StandardOpenOption.APPEND);
            System.out.println("Resultado escrito no arquivo: " + content);
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            lock.unlock();
            System.out.println("Lock released!");
        }
    }
}
```

Se execurtamos o código acima veja o resultado: 

```java
Execução do Stream fora do Supplier
Dobro de: 1
Dobro de: 2
Dobro de: 3
Dobro de: 4
Dobro de: 5
Lock adquirido!
Resultado escrito no arquivo: 2, 4, 6, 8, 10
Lock released!
Doubled Numbers: [2, 4, 6, 8, 10]
```

O nosso código executou fora do nosso lock 😱

## Execução preguiçosa do jeito certo!

A correção do problema acima é trivial; basta entendermos que, para usarmos o poder "lazy" dos lambdas, precisamos executar o stream inteiro como parte do Supplier e não em uma variável separada.

Podemos fazer isso encapsulando o código em um método e chamando o método a partir do Supplier OU passando o bloco de código diretamente no Supplier. Vou utilizar a segunda opção aqui:

```java
public class LazyLambdaExampleCorreto {
    private static final Lock lock = new ReentrantLock();

    public static void main(String[] args) {
        List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

        // Execução do Stream fora do Supplier
        System.out.println("Execução do Stream fora do Supplier");

        writeToFileWithLock("output.txt", () ->
                numbers.stream()
                        .map(n -> {
                            System.out.println("Dobro de: " + n);
                            return n * 2;
                        })
                        .collect(Collectors.toList()));
    }

    ...
}
```

O código de escrita em arquivo continua o mesmo. A única coisa que mudamos foi passarmos o stream dentro do próprio supplier. Isso garante que só vamos executar o stream quando o `supplier.get()` for invocado.

O nosso novo resultado será:

```java
Execução do Stream fora do Supplier
Lock adquirido!
Dobro de: 1
Dobro de: 2
Dobro de: 3
Dobro de: 4
Dobro de: 5
Resultado escrito no arquivo: 2, 4, 6, 8, 10
Lock released!
```

## Conclusão

Esse é um "erro bobo" que me pegou desprevenido. Naquele momento, tentando manter o código mais legível, eu isolei o meu stream em uma variável. Isso fez com que meu código rodasse fora do lock 😅.

Espero que este texto sirva de lembrete para vocês evitarem o mesmo problema.