---
layout: post
title: [Fundamentos] O problema da "escrita-dupla"
published: true
description: O problema da escrita-dupla em um sistema real
tags: sistemasdistribuidos, iniciante, microserviços, java
//cover_image: https://direct_url_to_image.jpg
---

> Aviso: Esse artigo é baseado em fatos reais 😬.

Nesse artigo eu vou te apresentar o problema da escrita dupla ("dual write") e as consequência desse problema em um sistema real que eu trabalhei. Eu não espero te apresentar uma resposta definitiva sobre como lidar com o problema mas eu espero que essa introdução faça você entender o problema, pensar sobre ele, saber identificá-lo e evitá-lo.

## Problema

Vamos começar descrevendo o nosso problema. Nós tínhamos um sistema que fazia o seguinte:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0qeeqa8ct18vpd5y20n4.png)

1. Usuário cria uma nova review
2. O reviewService cria a review no repositório como pendente.
3. O reviewService invoca o approvalService enviando uma mensagem para que o sistema de moderação aprove ou não a review. Esse processamente ocorre assíncronamente.
4. O sistema responde a requisição com o ID da review pendente para que o usuário possa verificar o status depois, se ela foi aprovada ou rejeitada.

Em código o nosso caso de uso ficaria assim:

```java

package com.hugomarques;

public class ReviewController {

    private final ReviewRepository reviewsRepository;
    private final ApprovalWorkflow approvalService;

    ReviewController(ReviewRepository repository) {
        this.repository = repository;
    }

    @PostMapping("/reviews")
    @Transactional
    Review newReview(Review newReview) {
        var pendingReview = reviewsRepository.save(newReview);
        approvalService.send(pendingReview);
        return pendingReview;
    }
}
```

Anos atrás, em meados de 2009-2010, esse código provavelmente seria implantado em um servidor JBoss, o repository seria um MySQL ou PostgresSQL e o approvalService estaria por detrás de uma fila JMS. E por que esses detalhes são importantes? Essas tecnologias em conjunto garantiam o controle transacional, ou seja, se enviar a mensagem para fila falha, a transação seria desfeita no banco de dados.

Hoje em dia, esse mesmo sistema (por vários motivos) poderia ser implementado sem o JBoss como servidor de aplicação, o repository poderia usar o AWS DynamoDB e o messageria poderia utilizar o AWS SQS. Qual o problema que isso nos trás? Nós não temos mais o controle transacional. Se a nossa mensagem não for enviada, o DynamoDB não tem como fazer rollback 😱.

## Uma história real

Um dos times que eu trabalhei se deparou com esse problema um tempo atrás. Um belo dia a equipe começou a receber tickets que algumas reviews estavam ficando bloqueadas em status `pendente`.

Eu era o engenheiro *on-call*  e decidi fazer uma análise. Eu verifiquei que toda vez que a *review* estava pendente o nosso código emitia uma *exception* de "Falha ao enviar mensagem para a fila".

Eu fui analisar o código e vi algo parecido com o exemplo desse artigo. Pra fazer as coisas ainda piores o código que enviava a mensagem estava contido dentro de um bloco `try/catch`, o catch logava o erro e não fazia mais nada. Ou seja, a operação era retornada com sucesso para o usuário!

Se você não entendeu o problema, em vez do fluxo ideal que temos no exemplo esse fluxo de erro era o seguinte:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fw4ji8wqrpjr0r7trfy8.png)

1. Usuário cria uma nova review
2. O reviewService cria a review no repositório como pendente.
3. O reviewService invoca o approvalService enviando uma mensagem para que o sistema de moderação aprove ou não a review. Esse processamente ocorre assíncronamente.
4. A mensagem por qualquer motivo que seja falha ao ser enviada.
5. O reviewService loga uma mensagem "Falha ao enviar mensagem para a fila".
6. O reviewService retorna a review pendente ao usuário.
7. Como a review não foi enviada para o fluxo de aprovação, ela ficará pendente para SEMPRE!.

## Uma solução "ingênua"

Depois que achamos o problema, nós decidimos aplicar a solução mais simples possível: Tratar o erro diretamente no código, capturando o erro/exception e executando uma chamada de rollback para o serviço 1. O nosso novo fluxo fica da seguinte forma:

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rwp605d9785ckufzgrzd.png)

Vamos ver a implementação:

```Java

package com.hugomarques;

public class ReviewController {

    private final ReviewRepository repository;
    private final ApprovalWorkflow approvalWorkflowService;

    ReviewController(ReviewRepository repository) {
        this.repository = repository;
    }

    @PostMapping("/reviews")
    Review newReview(Review newReview) {
        var pendingReview = repository.save(newReview);
        try {
           var approvedReview
 = approvalWorkflowService.start(pendingReview);
           return approvedReview;
        catch (Exception e) {
           repository.delete(pendingReview);
        }
    }
}
```

Notem que eu chamei a solução de ingênua. Por quê? O que acontece se o seu serviço tiver um problema de erro justo no rollback?

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ftz9sveyolft8tdw3fw1.png)

Observe no diagrama acima, quando tentamos fazer o rollback chamando `repository.delete` o DynamoDB pode nos retornar um erro 400. Logo, nesse cenário a nossa review continuará *pendente*.

Embora ingênua, a solução reduziu o número de inconsistências em mais de 90%. Por ser uma solução barata e simples, ela foi suficiente para "estancar o sangramento".

Para uma solução mais robusta o time teria que rearquitetar algumas partes do sistema e usar alguns padrões de microserviços como SAGAs e/ou "Transactional Outbox" mas isso são cenas para os próximos capítulos.

## Conclusão

Recapitulando o que aprendemos até aqui:
1. Cuidado ao invocar sistemas distribuídos em sequência sem contexto transacional. Isso é receita para problemas de escrita-dupla e inconsistência dos dados.
2. Algumas vezes uma solução simples pode ser tudo que você precisa dependendo da sua escala.
3. Embora não seja o foco aqui, você agora sabe que existem outros padrões para lidar com esse tipo de erro.

Eu espero que você tenha curtido. Se gostou, não deixe de acompanhar as minhas dicas no twitter [@hugaomarques](https://twitter.com/hugaomarques).

Agradecimentos especiais ao @rponte e ao @zanfranceschi por revisarem o artigo e colaborarem com idéias.

## Quer saber mais?

[A discussão segue bem interessante no twitter com referências à outras soluções e padrões que lidam com esse problema](https://twitter.com/hugaomarques/status/1522681876208521216?s=20&t=_DVahG4vsows_rPVJlFFzg).