---
layout: post
title: Recursão de cauda para iniciantes
published: true
description: Vamos entender o problema de perfomance com algoritmos recursivos e como recursão de cauda resolve o problema.
tags: algoritmo, recursao, estruturadedados, iniciante
---

No post mais recente do helloworldcomics nós discutimos sobre recursão.

Conferiu lá o post? Se não, volta lá!

## Problema

Depois de ler o post você aprendeu que a recursão empilha suas operações em memória.

Se essa pilha for muito longa você pode gerar um "Stack Overflow error". Quer ver?

Primeiro eu escrevi um programa simples para calcular o fatorial de um número. Pra relembrar, o fatorial é calculado da seguinte forma:

```
f(n) = f(n) * f(n-1) * f(n-2) ... f(1) * f(0)
```

Como explicado no post do helloworldcomics, esse problema pode ser modelado em uma recursão.

O exemplo abaixo mostra um código simples de cálculo de fatorial. No nosso exemplo, estou calculando o fatorial de 5.

{% gist https://gist.github.com/hugomarques/9948d1918e34dff382d12e186175b7fb %}

Porém, note o que acontece quando tentamos calcular o fatorial de um número bem grande, por exemplo, 11_000.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j84e4dnu57yrrz6yx6y8.png)

Ao calcular o fatorial nós acumulamos as chamadas de função em uma "pilha" em memória. No caso do fatorial, o tamanho da pilha em memória vai ser igual ao número que queremos calcular.
No exemplo acima, nós tentamos acumular 11_000 chamadas de função! 😨

Agora que você conhece o problema. E se existisse um jeito de calcularmos usando recursão sem ficar aumentando essa pilha? Em "espaço constante".

## Solução: Recursão de cauda

Para obter esse resultado, nós vamos utilizar uma técnica chamada de "recursão de cauda".

O princípio é bem simples, a última coisa que você retorna na função tem que ser a sua função recursiva. O return não pode ficar esperando acumular com o retorno da função.

Que ver na prática? Se liga no código abaixo que fica mais fácil entender:

{% gist https://gist.github.com/hugomarques/e0095b6378acbcbfe70703d81911d7f7 %}

Pontos importantes:

1. Na linha 40 nós chamamos a função `factorial`. Essa função serve só pra manter a mesma interface. Quem vai fazer a recursão é a função `factorialTail` declarada na linha 6.
2. Na função `factorialTail` note como nós mudamos a função para em vez de retornar `n * factorial(n-1)` ela agora acumula o resultado dentro do parâmetro da própria função. Veja como o valor do parâmetro `acc` vai incrementando a cada chamada da recursão.

Calma, não se desespere, vamos ver com um exemplo...

Antes a gente tinha:
```
factorial(num = 3) -> 3 * factorial(2)
factorial(num = 2) -> 2 * factorial(1)
factorial(num = 1) -> 1 * factorial(0)
factorial(num = 0) = 1 (agora a gente desempilha)
factorial(num = 1) -> 1 * 1
factorial(num = 2) -> 2 * 1
factorial(num = 3) -> 3 * 2 = 6
```

Já com a recursão em cauda nós fazemos:
```
factorial(3) ->
factorialTail(num = 3, acc = 1 * 3) ->
factorialTail(num = 2, acc = 3 * 2) ->
factorialTail(num = 1, acc = 6 * 1) ->
factorialTail(num = 0, acc = 6) -> 6
```

Viu como não precisa desempilhar?! O resultado já tá pronto na variável `acc` 😍.

Ao implementar o código dessa forma, em algumas liguagens o compilador otimiza a recursão e mantém o "espaço constante".

Importante mencionar que o código acima só funciona se sua linguagem implementar a otimização de recursão em cauda. O exemplo acima funciona em Scala mas em Java deu ruim... 😭

Curiosidades:
1. Se você executar o código para o número 21 o resultado é negativo... você sabe por quê?
2. Se você tentar executar para um número gigantesco, o resultado é 0... 🧐

## Conclusão

Nesse artigo nós aprendemos:
1. Recursão é uma técnica útil mas que pode ter problemas de perfomance a depender do tamanho da pilha de recursão.
2. Para solucionar o problema de performance com tamanho da pilha nós podemos utilizar a técnica de recursão de cauda.
3. Nem toda linguagem implementa recursão de cauda. Vai lá verificar se a sua implementa, faz o teste e deixa um comentário.

Se você chegou aqui parabéns! Se ficou difícil de entender, não desista, leva tempo e persistência.