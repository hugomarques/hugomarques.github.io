---
title: Testes - ocks e stubs dinâmico com mockito em java
published: true
description: Criando mocks e stubs dinâmicos com Mockito em Java para simplificar testes quando a construção de objetos reais é impraticável.
tags: #java #mockito #junit 
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2024-05-24 01:00 +0000
---

Esse vai ser curtinho. Hoje eu estava tentando testar uma classe que segue o seguinte comportamento:

```java
Book book = bookManager.getBook(id);
book.getId();
```

Por vários motivos que não vêm ao caso agora, imagine que você não consegue construir o objeto `BookManager` e também não consegue criar um `FakeBook` para injetar o ID conforme você deseja.

Pois bem, eu lembrei que era possível criar um mock dinâmico usando `Answer` do Mockito.

## Solução: Um mock dinâmico

A solução fica assim:

```java
@ExtendWith(MockitoExtension.class)
public class MyBookManagerTest {

@Mock 
private BookManager bookManager;
@Mock 
private Book book;

    @Test
    public void testMyMethod() {
        // Define the behavior of the bookManager mock
        when(bookManager.getBook(anyInt())).thenAnswer(new Answer<Book>() {
            @Override
            public Book answer(InvocationOnMock invocation) throws Throwable {
                Object[] args = invocation.getArguments();
                int id = (Integer) args[0];
                when(book.getId()).thenReturn(id);
                return book;
            }
        });
        // Use the mock in the test
        Book book = bookManager.getBook(12345);
        // Verify the behavior of the mock
        assertEquals(12345, book.getId());
    }
}
```

Note que, ao definirmos o comportamento do `BookManager`, retornamos uma `Answer`. Nessa `Answer`, capturamos o parâmetro passado (veja como usamos a `invocation`) e o configuramos no mock `Book` para ser retornado quando fizermos a chamada `book.getId()`.

Dessa forma, em vez de definirmos o mock diversas vezes, podemos definir apenas uma vez e fazer várias chamadas:

```java
// Use the mock in the test
        Book book = bookManager.getBook(12345);
        // Verify the behavior of the mock
        assertEquals(12345, book.getId());

// Esse aqui também funciona porque o nosso mock é configurável
        Book book = bookManager.getBook(6789);
        // Verify the behavior of the mock
        assertEquals(6789, book.getId());
``` 

## Simplificando: Java 8 + Lambdas 🥰

Se usarmos lambdas em vez da `anonymous` classe, o nosso exemplo fica ainda mais simples:

```java
        when(bookManager.getBook(anyInt()))
.thenAnswer(invocation -> {
            int id = invocation.getArgument(0);
            when(book.getId()).thenReturn(id);
            return book;
        });
```

É isso, essa foi direto das trincheiras. Normalmente, eu gosto de evitar mocks se possível e tento usar os objetos reais. No meu caso específico criar o objeto ia ser um trampo do cão e aí eu decidi usar a ferramenta pra simplicar a minha vida.

Keep coding! 💻