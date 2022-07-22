---
layout: post
title: Sistemas distribuídos são estranhos
published: true
description: O que são sistemas distribuídos e por que eles são complexos?
tags: #iniciante #sistemasdistribuidos #microserviços
//cover_image: https://direct_url_to_image.jpg
---

> Nota: Nenhuma das idéias aqui é nova. Essa é a minha síntese sobre sistemas distribuídos baseada nas fontes disponíveis nas referências.

## O que são sistemas distribuídos

Se eu tivesse que definir sistemas distribuídos eu diria que:

> Sistemas distribuídos são sistemas cujas suas partes estão distribuídas entre várias maquinas. Essas máquinas se comunicam umas com as outras pela rede.

Atualmente, praticamente todo sistema moderno é uma espécie de sistema distribuído, seja eles mais simples ou mais complexo. Afinal de contas, no melhor dos casos nós teremos uma máquina cliente e outra máquina servidor. As exceções que são aplicações desktop ou mobile se elas rodam totalmente offline.

## Por que precisamos de sistemas distribuídos?

Agora que sabemos o que é um sistema distribuído, por que precisamos deles? No geral, nós criamos sistemas distribuídos para resolver problemas que uma máquina só não é capaz de resolver. Em sua maioria esses problemas são:
- Escalabilidade: 1 máquina não consegue atender seus requisitos de requisições.
- Disponibilidade: Caso 1 máquina morra, você tem outra máquina para continuar atendendo suas requisições.
- Latência: Distância importa. O limite físico para transferência de dados é a velocidade da luz. Logo, se você possui máquinas mais próximos dos seus usuários o tempo de resposta das requisições vai ser menor.

## Por que sistemas distribuídos são estranhos?

### Um serviço local

Vamos imaginar o seguinte cenário. Você decidiu criar um sistema simples que roda na sua própria máquina e salva reviews de jogos que você gostou. Se você fosse escrever o código que salva novas reviews, poderia ser algo do tipo:

```Java
package com.hugomarques;

public class ReviewController {

    private final ReviewRepository repository;

    ReviewController(ReviewRepository repository) {
        this.repository = repository;
    }

    @PostMapping("/reviews")
    Review newReview(Review newReview) {
        return repository.save(newReview);
    }

}
```

Agora pense que já que o sistema vai ficar na sua própria máquina e é algo bem pequeno, o `repository` salva os dados localmente, escrevendo eles no disco.

### Diferenças entre local vs distribuído

Passado o tempo, te pediram para escrever um sistema parecido no seu trampo. A diferença crucial é que agora em vez de salvar em disco, você vai salvar os dados chamando outro serviço ou um banco de dados. Vamos pensar juntos:
1. Quais os problemas que isso pode causar?
2. Quais as preocupações que você não tinha antes que agora você tem que ter?

O primeiro problema que você tem que entender é que antes, o seu código de salvar executava uma função local. 99.99999% das vezes isso vai executar tranquilo, no máximo você vai ter que lidar com uma exception porque o arquivo não existe ou o disco está cheio.

Agora, a sua chamada vai trafegar pela rede, ou seja, antes de salvar o arquivo os seus dados precisam ser enviados pela rede até a outra máquina, a outra máquina precisa salvar os dados e enviar a confirmação que tudo foi feito com sucesso. Vamos entender esse passo a passo?

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0j3sjuw48c9ewzlz13hm.png)

1. ReviewService envia a requisição pela rede.
2. A rede "entrega" a mensagem pro banco de dados.
3. O banco de dados valida que a requisição está correta.
4. O banco de dados atualiza o seu estado interno.
5. O banco de dados envia uma resposta pela rede.
6. A rede entrega a resposta ao nosso ReviewService .
7. O ReviewService valida se a resposta retornada está correta.
8. Review service atualiza seus dados internos e envia a resposta para o usuário.

Perceba que em cada um dos passos acima, o processo pode falhar. As vezes, essa falha é simples:
* No passo 1, por exemplo, a rede foi desconectada e a requisição não pôde ser enviada.

Outras vezes, a resposta pode ser ambígua. Por exemplo, se no passo 5, o banco de dados falhou em enviar a resposta pela rede. O nosso ReviewService agora não sabe:
* Foi algum problema de rede?
* O banco de dados foi atualizado?
* Foi algum problema no tipo de dado enviado?

Esses problemas em lidar com as requisições navegando pela rede e como erros podem acontecer é o que faz sistemas distribuídos tão complexos.

## As 8 falácias dos sistemas distribuídos

Alguns dos problemas que enfrentamos com sistemas distribuídos são explicados pelas "8 falácias dos sistemas distribuídos". Erros comuns que programadores fazem ao lidar com a rede e que acabam gerando bugs e comportamentos indesejados difíceis de identificar e corrigir.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/y5322vqofheme7lba9ep.png)
source: [@deniseyu](https://deniseyu.io/art/)

### Falácia #1: A rede é confiável

A rede vai falhar. Não é uma questão de se mas de quando. Pacotes não vão chegar ao destino, cabos vão ser rompidos e roteadores são reiniciados o tempo todo. Lembra do erro no passo 5 acima? Um das possíveis causas poderia ser essa falácia aqui.

### Falácia #2: A latência é zero

A latência não é zero. Se lembre que a velocidade com que os dados transitam é no máximo a velocidade da luz assumindo um cabo sem interferência nenhuma. Ou seja, uma chamada local no seu datacenter que custaria 10ms agora pode custar 40ms se você precisar chamar um datacenter mais distante. Para algumas aplicações, esses 30ms extras tem impacto.

### Falácia #3: A banda é infinita

A banda não é infinita. Existe um limite na quantidade de dados que são trafegados pela rede. Se limitarmos o tamanho dos pacotes que enviamos, isso pode significar mais viagens o que faz com que a latência (falácia #2) aumente.

### Falácia #4: A rede é segura

Sempre assumimos que a rede está comprometida. A partir daí trabalhamos com modelos como *"Threat modeling"* para identificar possíveis ameaças e o que estamos ativamente fazendo para que as nossas aplicações respondam a tais ameaças. Por exemplo, encriptação dos dados quando em trânsito.

### Falácia #5: A topologia não muda

A topologia da rede muda o tempo inteiro. Nos tempos de hoje que usamos Cloud, servidores são removidos ou adicionados elasticamente à medida que a escala necessita.

### Falácia #6: Existe apenas 1 administrador

No passado, talvez até existisse uma pessoa responsável por grandes pedaços da rede. Hoje em dia, a rede está cada vez mais fragmentada. Nem os próprios desenvolvedores mantém a rede de suas aplicações, delegando essa atividade para plataformas de cloud como AWS, Azure ou Digital Ocean.

### Falácia #7: O custo de transporte é zero

Existe um custo relacionado a trafegar dados na rede. Custo de manutenção, de hardware e software.

### Falácia #8: A rede é homogênea

Especialmente hoje em dia, a rede é cada vez mais heterogênea. Aparelhos mobile, aparelhos IoT como câmeras, termostatos ou mesmo laptops e desktops das mais variadas marcas e modelos podem se conectar a uma aplicação.

## Conclusão

Recapitulando o que aprendemos:
1. Sistemas distribuídos são sistemas cujas partes estão divididas e se comunicando pela rede.
2. Sistemas distribuídos nos ajudam a lidar com problemas de escalabilidade, disponibilidade e latência.
3. Sistemas distribuídos são complexos porque ao depender da rede, vários erros podem acontecer.
4. As 8 falácias descrevem vários dos desafios enfrentados por sistemas distribuídos

Eu espero que esse seja o primeiro artigo de muitos que estão por vir onde vamos discutir mais sobre fundamentos de sistemas distribuídos.

Se você gostou, não deixe de acompanhar as minhas dicas no twitter [@hugaomarques](https://twitter.com/hugaomarques).


## Referências

1. [Challenges with distributed systems](https://aws.amazon.com/builders-library/challenges-with-distributed-systems/?did=ba_card&trk=ba_card)
2. [Why distributed systems are hard](https://youtu.be/w9GP7MNbaRc)
3. [The eight fallacies of distributed computing](https://medium.com/geekculture/the-eight-fallacies-of-distributed-computing-44d766345ddb)
4. [Escrevendo Clients e Services Tolerantes a Falhas com Rafael Ponte ](https://www.youtube.com/watch?v=TMmN9cR_IsM&ab_channel=ZUP)