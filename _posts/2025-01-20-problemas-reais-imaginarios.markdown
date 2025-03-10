---
title: Problemas reais x Problemas imaginários
published: true
description: Uma reflexão sobre as diferenças entre os desafios enfrentados por engenheiros que lidam com o mundo físico e por engenheiros de software. Exploramos como problemas "reais" e "imaginários" impactam a forma de planejar, executar e resolver questões em cada área.
tags: #carreira #software #braziliandevs
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/9ejsq51j71pmdti1xcq9.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2025-01-20 18:43 +0000
---

Essas férias eu estava tentando explicar pro meu irmão qual é a complexidade inerente ao trabalho que eu faço. 

Pra contexto meu irmão é Engenheiro de Minas, ele trabalha com o gerenciamento da operação de máquinas de grande porte em uma mina de minério de ferro, ele também é programador nas horas vagas, tendo feito mestrado na parte de *data science* e já fez algumas *apps Android* no passado. 

Depois de discutirmos o que ele faz versus o que eu faço, eu cheguei à conclusão que embora ambos os trabalhos sejam inerentemente complexos, a complexidade reside em particularidades diferentes. Na falta de um nome melhor, eu chamei isso de **"Problemas reais" vs "Problemas imaginários"**.

Antes de mais nada, é importante mencionar que esse texto não é uma tentativa de dizer o que é ou não mais difícil, ou que um trabalho X ou Y se comporta dessa ou tal forma. O mundo não é binário, existem exceções, nuances e regras a serem quebradas. A minha intenção aqui é mais documentar a conversa que tivemos que eu achei interessante o suficiente pra ser compartilhada.

## Problemas do "mundo real"

Eu chamei de problemas do mundo real os problemas que meu irmão enfrenta. As máquinas que ele opera existem fisicamente. Elas quebram, têm desgaste, é preciso movimentá-las, o minério existe, é preciso ser extraído, pra extrair pessoas precisam ir lá tirar o minério ou levar a máquina até ele. 

Qualquer problema que lide com o mundo **"físico"** se encaixa nisso: infraestrutura, indústria de energia, etc.

Uma característica que eu percebi ouvindo relatos do meu irmão, é que o desafio normalmente está no **planejamento**. Você precisa planejar manutenção, precisa planejar quando a máquina vai pra lá, quando não vai. Cada recurso é escasso e, mais importante do que isso, eles são difíceis de modificar/remanejar. Dessa forma, planejamento meticuloso, orçamento sob controle e normas bem específicas têm que ser seguidas à risca porque qualquer problema pode parar a cadeia de produção inteira de forma que pode levar dias pra se recompor. 

## Problemas "imaginários"

Já deve ficar óbvio que eu tratei nossos problemas de engenharia de software como **"imaginários"**. Por quê? Primeiro, que tudo é virtual, não existe um produto físico que você pode tocar. Além disso, o que o sistema em questão faz pode mudar com linhas de código, você não precisa realmente construir uma máquina que ocupa lugar no mundo físico (pelo menos em linhas gerais). 

A consequência direta dos problemas imaginários é que, em sua maioria, eles são o que chamamos de *"two-way door"*, ou seja, você consegue desfazer algo feito ou consegue modificar/alterar com linhas de código o problema. O meu pai, também Engenheiro de Minas, tem uma frase muito boa: 
> **"Depois que eu tirei a pedra, não tem como colocar de volta"**.

No nosso mundo seria como se não houvesse *rollback*, apenas *rollforwards*.

Porém, os problemas imaginários têm outra característica interessante. Dificilmente um ajuste que seja feito em uma máquina vai impactar outra máquina mais embaixo na linha de produção. No mundo do software, no entanto, isso é comum. Nós estamos cercados de regras de negócio onde a alteração de uma regra em um sistema pode ter impactos devastadores em 2 ou 3 sistemas de distância daquele sistema central. Isso faz com que mudanças críticas precisem ser discutidas, compartilhadas e revisadas por vários times antes do desenvolvimento continuar. 

## Consequências

As consequências desses dois tipos de problemas são interessantes. O meu irmão gasta **MUITO** do tempo dele planejando e tentando evitar que o plano os leve a um caminho que eles vão ter que parar ou desfazer algo. Os desafios dele estão muito no campo **estratégico/gerencial**. Também existe uma preocupação muito grande com segurança, já que um erro pode custar uma vida humana. 

Já eu, gasto bastante tempo documentando qual será a minha mudança e qual o impacto que eu acredito que ela irá causar. Daí eu gasto um tempo discutindo com vários *stakeholders* se aquelas hipóteses estão corretas e se devemos seguir em frente. É um desafio por vezes muito **"intelectual"**, não no sentido de ser mais inteligente, mas no sentido de gastar muito tempo pensando em todas essas possibilidades e variações. 

Claro, também é importante notar que há exceções. Nem toda mudança de software pode ser facilmente desfeita, as famosas *"one-way doors"* e nem todo trabalho no mundo físico pode não causar consequências inesperadas.

## Desafios em comun

Apesar de todas as diferenças mencionadas acima, ambas as áreas compartilham um aspecto fundamental: lidar com pessoas. É interessante notar que, em diversas conversas que temos, técnicas de gerenciamento que eu utilizo no meu trabalho também se aplicam ao que meu irmão enfrenta. Da mesma forma, situações que ele vivencia no dia a dia da mineração eu já presenciei em empresas de tecnologia pelas quais passei.

Por isso, independentemente da sua área de atuação, as *soft skills* são indispensáveis.

## Sumário

Neste artigo, exploramos as diferenças entre os desafios enfrentados no mundo físico e no mundo digital, destacando como a natureza dos problemas – tangíveis ou virtuais – impacta o planejamento, a execução e a resolução de questões em cada área. Enquanto no mundo físico o foco está em recursos escassos e planejamento rigoroso, no mundo do software a flexibilidade das mudanças exige colaboração intensa e análise de impacto.

E vocês, o que acham? Qual o desafio do trampo de vocês?
