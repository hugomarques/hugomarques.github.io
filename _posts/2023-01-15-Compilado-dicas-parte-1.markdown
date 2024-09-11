---
title: "Compilado dicas de carreira - parte 1"
published: true
description: Compilado das threads de dicas de carreiras do Twitter 
tags: #carreira #junior 
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2023-01-14 23:55 +0000
---

Olá pessoal, a pedidos eu decidi compilar as dicas em artigos. Nesse artigo eu vou compilar as primeiras 10 dicas.
As dicas sobre entrevistas em big tech eu vou postar em um artigo à parte. Vamos lá?

## Dica #1 - Onboarding em um time novo

Sempre que entrar em um time novo, procure entender onde estão as métricas e dashboards e onde estão os logs. Isso ajuda muito a entender a stack e se preparar pro oncall. Como colateral, você verifica a ops do time e onde vocês podem melhorar.

## Dica #2 - Gentileza em code reviews

Seja gentil em code reviews, existe um ser humano do outro lado. Dê sugestões quando possível e tente fazer mais perguntas pra tentar entender a lógica da outra pessoa. Elogie as partes que você acha legal. 
Se algo for crítico/blocker, explique o problema. Proponha uma sugestão ou se for complexo demais, proponha um pair programming pra resolver o problema juntos. O principal intuito de code reviews é a colaboração.

## Dica 3 - O que monitorar em um sistema?

Não sabe o que monitorar no seu sistema? Que tal começar com os 4 "golden signals"? 
1. Latência, 
2. Trafégo (rps), 
3. Errors, 
4. Saturação (CPU/memória/disco/network).

E por favor, NÃO monitorem a média. Olhem os "percentiles" p50, p90, p99.
Curiosidade: Embora eu tenha aprendido a usar essas métricas só recentemente eu aprendi que elas eram chamadas do 4 "golden signals". Acabei aprendendo quando eu fiz uma entrevista e o entrevistador mencionou isso pra mim 😃.

## Dica #4 - O que fazer quando o alarme de oncall toca?

Quando o pager tocar, tente estancar o sangramento primeiro. Procure a "root cause" depois. Um bom exemplo disso é, rolou deployment recente? Faça o rollback. Rollback feito, análise se o problem foi embora.

Um erro comum é ficar tentando identificar o que aconteceu de errado, nisso se passa 1 - 2h de usuários sendo impactados. A melhor estratégia é fazer rollback pra eliminar de cara a chance de ser um erro novo. 

O mesmo vale para suas dependências. Se o erro tá vindo de uma dependência, questione se houve um deployment e peça pra fazer rollback. Uma vez, a dependência ficou negando por horas que o erro não era com eles, eu achei um deployment, fizemos rollback e o erro foi mitigado.

Erro mitigado, podemos gastar o tempo necessário para analisar a root cause e corrigir o erro de uma vez por todas. Note que rollback é apenas um exemplo, as vezes, rodar um comando manual na máquina ou qualquer outra intervenção manual pode ser o passo de mitigação.

## Dica #5 - Monitore suas dependências

Monitore suas dependências no dashboard. O que monitorar? Throttling, error rate e latência é um bom começo.  Por quê? Reduz o teu tempo de reação. Imagine você ser pageado 3am e ao abrir o dashboard você já vê o serviço X retornando 500. Bem mais rápido do que ir caçar logs né? Outro uso bacana é fazer load testing e rapidamente identificar no teu dashboard quais dependências que são o gargalo da tua aplicação.

## Dica #6 - Desconfie da frase "é seguro, nós já fizemos isso antes"

Desconfie da frase "... é seguro eu/nós já fiz isso 1000x (ou mais) antes...". Não é porque algo deu certo antes que vai dar certo de novo. Exemplo, em 2017 um SDE do AWS rodou um script e derrubou o S3 (https://go.aws/3Fcnda8).

Revise seus procedimentos, automatize e teste. Insista que as coisas podem dar errado e tente pensar "o que pode dar errado?". Não confie no histórico do passado, um dia a sorte pode acabar.

## Dica #7 - Utilize o protocolo de 2 pessoas para mudanças manuais em produção

Evite tocar em produção manualmente, mas se for realmente necessário, faça a mudança seguindo o protocolo de "2-person rule". Em resumo, 2 pessoas executem a mudança juntos, tais quais filme para ativar códigos nucleares.

Pessoa 1: "Executando script xpto na tabela y, ok?"
Pessoa 2: Verifica e confirma.

Esse protocolo se tornou bem difundido no AWS depois do evento no S3 (ver dicas anteriores). Para uma boa 2-person rule execute os passos devagar, perguntando por confirmação do seu peer.

Se a mudança for muito complicada, eu recomendo escrever uma "checklist de mudança" mas essa eu deixo para uma futura dica. Finalmente, se você está pensando "nossa, tudo isso?". Pois é, melhor automatizar para evitar ficar tocando em Prod né? 😀

## Dica #8 - Faça uma ata da reunião

Anote o que é discutido na reunião, qual foi a decisão, se existem "action items" E QUEM é responsável por cada um deles. Depois da reunião envie as anotações pros participantes pra todo mundo ficar alinhado com os próximos passos.

Se você está ocupado discutindo e não pode anotar, peça para alguém se voluntariar pra anotar. Essa "besteira" já evitou tanto mal entendido. Eu também já vi não ser feito e as pessoas perderem horas discutindo porque ninguém lembrava o que foi acordado.

Um template bacana que eu uso:
- Lista com principais pontos discutidos.
- Lista de próximos passos e para CADA um eu coloco um "responsável: <nome da pessoa>", por exemplo, "Enviar o dica #8. Owner: Hugo".

Isso serve para todos ficarem cientes de quem tem que fazer o que.

## Dica #9 - Aprenda a usar suas ferramentas

Otimize suas ferramentas. Gosta de usar Vim? Aprendar a usar Vim de uma forma eficiente. Gosta de IDE? Aprenda os atalhos e os comandos que sua IDE te oferece. Pode parecer besteira mas no longo prazo isso vai te salvar muito tempo.

xemplo real: Mais uma de vez eu fui chamado pra ajudar alguém com um bug. Chegando lá a IDE da pessoa estava toda vermelha, nada compilava, basicamente um editor de texto mais lento. Eu ajeitei a IDE e na hora o erro foi acusado pela IDE.

A regra se aplica a qualquer ferramenta que você esteja usando com frequência: terminal, CLI, IDEs. Aprenda os atalhos, evite repetições, configure a ferramenta pra fazer o seu tempo ser mais eficiente.

Dica #10 - O que eu fiz nos meus primeiros 90 dias no Twitter

> Disclaimer: Eu não estou mais no Twitter mas a dica ainda é válida mesmo assim.

Meus primeiros 90 dias no @Twitter como Sr SWE foram:

1. Perguntas, perguntas e mais perguntas.
2. Assuma que tudo tem um motivo em vez de chegar "no trampo antigo a gente faz Y e funciona". 
3. Pergunte pra todo mundo, Jr, Pleno, Sênior, Staff.
4. Tenha paciência, com o tempo a produtividade e velocidade voltam.
5. Eu gasto um bom tempo investigando como as coisas são feitas ao redor da empresa e não apenas no meu time. Eu leio docs, procuro em wikis, toma um bom tempo mas as vezes acho coisas interessantes.

# Conclusão

É isso aí, aguardem que vou compilando o resto das dicas em breve. 
