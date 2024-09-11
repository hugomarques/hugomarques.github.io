---
title: "[Relato] Os testes atrasaram meu projeto /s"
published: true
description: Esse artigo é uma pequena história como a decisão de NÃO escrever testes automatizados quase nos custou a entrega de um projeto. 
tags: #tdd #testing #unittests #sistemasdistribuidos
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/fluc2ebjc8vwp863ady4.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2023-10-29 02:30 +0000
---

> /s é uma gíria da internet pra dizer que aquela frase contém sarcasmo 😆

Quem me segue nas redes sociais ou por aqui sabe que eu sou um entusiasta de testes automatizados e teste de software no geral (não confundir com TDD). 

![Pessoa levantando a mão pronta pra discordar e desistindo quando vê que eu gosto de testes](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/a8tf15yv26u96hf5lrrn.jpg)

Esse artigo é um relato da decisão que fizemos no final de 2022, quando eu estava liderando um projeto com prazo apertadíssimo.  Nele, abordarei nosso contexto, nossas hipóteses, as consequências e o que aprendemos com essa decisão. Eu não fornecerei detalhes específicos do projeto por motivos de NDA, mas tentarei ser específico o suficiente para abordar os pontos desejados.

## Contexto

Em outubro de 2022, eu estava envolvido em um projeto com um prazo extremamente desafiador. O projeto era ambíguo, com várias frentes, envolvendo aproximadamente 4 equipes diferentes e cerca de 20 pessoas, entre engenharia e produto.

Quando a equipe de engenharia se reuniu com o gerente para analisar o escopo restante, a projeção indicava a entrega para junho de 2023. O problema era que tínhamos um prazo "não-negociável" de abril de 2023 por razões comerciais. A única solução viável? Reduzir o escopo.

Ao analisar o escopo a ser reduzido, incluindo funcionalidades e infraestrutura adicional, nosso gerente fez a seguinte pergunta: "E quanto a toda essa tarefa de testes de integração?". Antes de continuar, uma pausa para definir "testes de integração".

Testes de integração? O quê? Vamos para uma pausa estratégica sobre testes de integração.

![Plantão da globo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/narh97v8oj0zlxivg81b.gif)

> No contexto desse artigo, eu me refiro a testes de integração como testes onde é necessário você subir sua aplicação e/ou interagir com componentes através da rede. 
Por exemplo, um `@SpringBootTest` que sobe o contexto e te permite testar a API conversando com o banco de dados.
No meu caso, os testes acima iriam fazer deploy do nosso código no AWS (na minha conta e depois na conta de homologação) e de lá eu poderia testar os vários microserviços interagindo uns com os outros através da orquestração.

Agora continuamos com a nossa programação normal...

Onde estávamos? Ah sim, o gerente. Você, que está lendo, deve estar pensando: "Claro, só podia ser um gerente...".

![Boromir reclamando da sugestão de remover testes](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/gvr7vf6ezk60s0j7sxk7.jpg)

"Respire fundo" e permita-me acrescentar alguns detalhes:

1. Eu trabalhei com esse gerente por ANOS, e confio extremamente no trabalho dessa pessoa.
2. Ele possui experiência em testes. Mais do que qualquer um, ele estimulava a equipe de engenharia a trabalhar em testes e aprimorar nossa qualidade.
3. Além disso, ele costumava ouvir o time, principalmente as pessoas mais experientes da equipe. Se naquele momento eu tivesse dito 'Nem pensar', havia uma grande possibilidade de ele acatar minha decisão."

## A nossa hipótese

A pergunta feita naquele momento me fez refletir: "Será que realmente conseguiremos economizar tempo e ainda entregar com sucesso?".

O processo de decisão foi o seguinte:

1. "Nós temos MUITOS testes de unidade; se houver alguma regressão no sistema, provavelmente esses testes identificarão qualquer erro".
2. Além disso, a parte do sistema que desejávamos testar não era trivial; não tínhamos garantia de que os novos testes, do jeito que os estávamos projetando, realmente fariam alguma diferença.
3. Por fim, a equipe trabalhando neste projeto era extremamente experiente, e conhecíamos o negócio e o sistema de ponta a ponta. Consequentemente, pensamos que os testes poderiam ser apenas o time seguindo as melhores práticas por seguir, sem necessariamente serem úteis naquele momento.

Assim, com esse risco documentado, a decisão foi tomada. Optamos por prosseguir com os testes de unidade e ignorar o segundo método de testes de integração até a entrega em abril. No entanto, estávamos enganados...

## Consequências

O nosso primeiro "milestone" era em Dezembro de 2022. Naquele momento, nós poderíamos integrar o trabalho sendo feito pelos 4 times e testar os primeiros casos de uso de ponta a ponta. Foi aí que os problemas começaram.
Nós documentamos cerca de 16 casos de uso que teriam que ser testados. O setup para esses casos de uso era não trivial e envolvia configurar as contas na AWS das pessoas engenheiras envolvidas nos testes.

Porém, o problema de verdade acontecia quando nós encontrávamos um "bug". O time tinha que corrigir o bug, enviar a correção e testar manualmente novamente em ambiente de homologação. Obviamente, esse processo todo de re-testar era muito lento e atrasou muito a entrega do nosso primeiro milestone.

Mas o problemas não pararam por aí... logo elas chegaram, elas que são as melhores amigas de nós pessoas devs quando estamos com projetos atrasados. Sim, elas, as "regressões". Com cada correção que nós enviávamos, invariavelmente, nós introduzíamos bugs em outras partes do sistema.

![Garotinha olhando casa pegar fogo porque ela confiou nos testes de unidade](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/rpf3t4k151aak0h5d7et.jpg)

Mas por quê isso estava acontecendo?

## A causa

Pra entender o motivo desses bugs apesar de 90%+ de cobertura de testes é importante entender algumas detalhes sobre o nosso sistema:
1. Esse projeto envolvia mudanças em 5 serviços diferentes. 
2. Muitas vezes, o que era corrigido no sistema A, tinha o único objetivo de propagar a informação até o sistema C. Apenas no sistema C poderíamos verificar que tudo estava correto.
3. Um desses sistemas, era uma orquestração implementada com *AWS Step Functions*. Embora eu goste do controle que esse tipo de arquitetura proporciona, o trade-off é que você pode introduzir problemas sem perceber ao modificar um dos estados que afeta indiretamente outro estado na máquina de estados. Além disso, testar unicamente um estado não garante que sua máquina de estados está correta. A melhor forma de testar esse tipo de arquitetura é escrever testes que exercitem a máquina de estados como um todo, e tratar o output final da máquina de estados como o que deve ser verificado pelos testes.

Os testes que nós decidimos não escrever afetava justamente o item #3 acima. Ou seja, quando nós quebrávamos algo na máquina de estado, as vezes nós não percebíamos a consequência disso em outras partes do sistema.

![Cpt Pickard frustrado por quebrar o sistema](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/0d4l9rlkmnn4znubyei3.jpg)

## Voltando aos eixos

Logo que entregamos o primeiro milestone, nós decidimos dar um "rollback" naquela decisão inicial. Os testes repetitivos e as regressões estavam causando tanto dano que era melhor sacrificar uma parte da nossa "gordura" no projeto e implementá-los pra resolver o problema de uma vez por todas.

Acabou que entregar os testes foi mais rápido do que tínhamos estimado inicialmente 🥳 e depois que adicionamos eles na nossa pipeline de CI/CD as regressões pararam 100%. 

## Aprendizado

Essa jornada me ensinou lições valiosas: 
1. *Testes são importantes independente do nível de experiência do time.* Mesmo com o contexto adequado e com a experiência o nosso time ainda cometeu erros e causou regressões. Os testes são os nossos paraquedas pra nos proteger justamente nessas situações.
2. *Essa experiência reforçou ainda mais o meu entusiasmo por testes.* Eu sempre implementei estratégias de testes em todos os projetos que eu passei e justamente no projeto que eu pensei "sabe de uma coisa, talvez dessa vez a gente não precise?", foi a vez que mais precisamos deles. "Lei de murphy" que fala né?
3. Embora o time tenha salvo 3 semanas iniciais pra implementar os testes. Nós perdemos muito mais tempo com regressões e testes manuais repetitivos, ou seja, *testes vão te economizar tempo na cauda longa*.

![He-man nos explica uma valiosa lição](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/j9qfsvccjt6925p0hknk.jpg)

## Conclusão

Espero que esse relato abra os olhos de vocês sobre a importância de testes automatizados. Não apenas isso, mas mesmo como um time experiente com 90%+ de cobertura em testes de unidade não foi suficiente pra nos prevenir de enfrentar problemas.

Finalmente, o meu amigo @rponte fala sobre a importância de testes de integração em sistemas distribuídos. Eu recomendo MUITO vocês assistirem a palestra do Ponte.

<iframe width="560" height="315" src="https://www.youtube.com/embed/ZV4Fl1uEbqw?si=hUpPQfpdKeBFsNuB" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>


Inspirado pela talk acima, em sequência a esse artigo eu vou escrever mais sobre estratégias de testes em sistemas distribuídos. 

De repente eu consigo trazer mais pessoas pra escrever testes e observar a importância deles em nossos projetos. Abraços e se curtiu, me segue aí já que meu conteúdo técnico vai ficar mais por aqui a partir de agora 🥰.
