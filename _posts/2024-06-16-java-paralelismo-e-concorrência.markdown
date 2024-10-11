---
title: "Paralelismo e Concorrência 101: Conceitos"
published: true
description: Vamos explorar conceitos de paralelismo e concorrência.
tags: #java #concurrency #parallelism #ptbr
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2024-06-16 19:13 +0000
---

> O termo 101 é usado aqui nos US para denotar algo para iniciantes e quem tá começando. É comum as primeiras disciplinas na faculdade serem <tópico 101>.

Recentemente, tenho escrito bastante no X sobre alguns aprendizados que tive lidando com programação concorrente em Java.

Este post é o primeiro de uma série para documentar o que aprendi neste período. O assunto é extenso, repleto de nuances e "dependes". É bom começarmos com a base para termos uma terminologia comum. Vamos lá?

## Concorrência e Paralelismo

**Concorrência** refere-se à capacidade de um sistema de lidar com múltiplas tarefas ao mesmo tempo. Não significa necessariamente que essas tarefas estão sendo executadas simultaneamente, mas sim que o sistema pode trocar rapidamente entre as tarefas, de modo que parece que estão ocorrendo ao mesmo tempo. Um exemplo de concorrência é como o seu SO troca o tempo de CPU entre diversos programas. Isso dá a impressão que todos eles estão executando ao mesmo tempo (talvez estejam se você tiver vários "cores"), mas, na verdade, a sua CPU só é MUITO rápida entre trocar quem tem a vez na CPU.

Paralelismo, por outro lado, refere-se à execução simultânea de várias tarefas. Isso requer múltiplos núcleos de CPU, onde cada núcleo pode executar uma tarefa diferente ao mesmo tempo. O paralelismo é sobre fazer várias coisas ao mesmo tempo. Um exemplo seria no caso que você está executando os diversos programas acima e eles realmente estão rodando em paralelo, cada um roda isoladamente na sua própria CPU ou core sem precisar "ceder" a vez. 

<figure>
  <img src="https://dev-to-uploads.s3.amazonaws.com/uploads/articles/liy4nb34fxlz22pucrf9.png" alt="Processo concorrente vs paralelo"/>
  <figcaption>*Processamento concorrente vs paralelo. Fonte: Baeldung CS.*</figcaption>
</figure>

Observe no exemplo acima como a CPU salta de task em task no processamento concorrente. Isso é feito tão rápido que dá a sensação que as tasks são executadas ao mesmo tempo.

Já no processamento paralelo, cada core é responsável por um task e essas tasks realmente acontecem ao mesmo tempo. 

As aplicações podem ser: 
1. Nem concorrentes e nem paralelas.
2. Concorrentes, mas não paralelas.
3. Paralelas, mas não concorrentes.
4. Concorrentes e paralelas.

#### Não-concorrentes e não-paralelas

Imagina o seu programa que roda sequencial em uma única thread. Cada comando é executado em sequência até seu final.

#### Concorrentes e não-paralelas

Uma aplicação pode ser concorrente sem ser paralela. Isso significa que ela pode gerenciar e trocar rapidamente entre várias tarefas, mas não as executa simultaneamente.

Exemplo:
Um sistema de tempo compartilhado em um único núcleo de CPU. Aqui, o sistema operacional alterna entre diferentes tarefas tão rapidamente que parece que todas estão sendo executadas ao mesmo tempo. Na realidade, o processador está executando apenas uma tarefa por vez.

#### Não-concorrentes e paralelas

Uma aplicação pode ser paralela sem ser concorrente. Isso significa que ela executa várias tarefas simultaneamente, mas essas tarefas não são gerenciadas de forma que possam cooperar ou competir pelo mesmo recurso.

Exemplo:
Uma aplicação que realiza cálculos massivamente paralelos em um cluster de computadores, onde cada nó do cluster está executando um cálculo diferente de forma independente. Não há interação entre as tarefas enquanto elas estão sendo executadas.

#### Concorrentes e paralelas

Uma aplicação pode ser concorrente e paralela ao mesmo tempo. Isso significa que ela não só gerencia várias tarefas ao mesmo tempo, mas também executa algumas delas simultaneamente em múltiplos núcleos ou processadores.

Exemplo:
Um servidor web que atende várias requisições de clientes. Ele pode utilizar threads para gerenciar várias requisições ao mesmo tempo quando ele está ocupado fazendo IO e, se estiver rodando em um sistema multi-core, ele pode processar várias dessas requisições simultaneamente (paralelismo).

### Benefícios

O exemplo clássico de benefício da concorrência é sua CPU poder trabalhar em alguma coisa enquanto a tarefa que ela estava executando espera algum outro recurso. Por exemplo, imagine que você está fazendo download de dados pela rede. Grande parte do trabalho ali é de IO (input/output) e não precisa da CPU. Nesse meio tempo, em vez de ficar esperando, a CPU pode pular pra outra tarefa.

<figure>
  <img src="https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ztbdcjd5eymaczc0h4sg.png" alt="Processamento concorrente vs IO"/>
  <figcaption>*CPU alterna entre tarefas enquanto espera IO. Fonte: Baeldung CS.*</figcaption>
</figure>

No caso do paralelismo, o benefício é mais direto. Se eu tenho 10 tarefas, em vez de executá-las 1 por vez, eu posso executar N tarefas onde N é o número de cores da minha CPU.

😣 cansativo, mas bacana né? Calma que ainda tem mais...

## IO vs CPU intensivo

Quando a gente lida com concorrência e paralelismo, é sempre importante entendermos se a nossa tarefa é *IO intensiva* ou * CPU intensiva*. 

Uma operação *IO intensiva* normalmente vai ser limitada pela velocidade de entrada e saída do sistema. Por exemplo, consumir dados de um microserviço, salvar um arquivo em disco, se comunicar com o banco de dados.

Já uma operação *CPU intensiva* vai ser limitada pela velocidade de cálculo e processamento da CPU. Exemplos: Realizar cálculos matemáticos, transformar dados durante uma chamada de serviço, o banco de dados processar uma query de leitura.

#### E o que *CPU intensiva* e *IO intensiva* tem a ver com concorrência e paralelismo?

Se a sua aplicação é bem *IO intensiva* ela vai se beneficiar bastante da capacidade de usar concorrência. Assim toda vez que sua aplicação estiver usando os dispositivos de entrada/saída a CPU pode trocar o contexto pra ir fazer outra coisa.

<figure>
  <img src="https://dev-to-uploads.s3.amazonaws.com/uploads/articles/6lgrq5gm7cktfbrk2zjt.png" alt="IO bound"/>
  <figcaption>*Tarefa IO intesiva. Fonte: leimao.github.io*</figcaption>
</figure>

Já se sua aplicação é *CPU intensiva*, concorrência não vai te ajudar muito. Por quê? Oras, o seu cálculo precisa da CPU pra terminar. Se há uma troca de contexto, o cálculo para até o contexto voltar pra sua aplicação. Porém, você se beneficia muito de paralelismo, porque cada cálculo pode ser executado sem parada no seu próprio núcleo. Quanto mais núcleos, mais execuções podemos fazer ao mesmo tempo.

<figure>
  <img src="https://dev-to-uploads.s3.amazonaws.com/uploads/articles/241en8elo5a505canvn8.png" alt="CPU bound"/>
  <figcaption>*Tarefa CPU intesiva. Fonte: leimao.github.io*</figcaption>
</figure>

## E agora?

Se você chegou até aqui, parabéns 😉! Não rolou muito código nesse artigo porque todo o código depende muito de onde ele está sendo executado pra gente definir se ele é concorrente e/ou paralelo.

Os conceitos acima são bem legais e bem importantes para as próximas lições. Durante muito tempo eu não entendia o que eram atividades "IO intensiva" 🤓. Saber esses fundamentos te ajuda, por exemplo, a ter uma ideia do número de threads que você quer usar e/ou se você deveria usar um `parallelStream()` ou um `thread pool`. Mas isso são tópicos para o próximo capítulo!

Se curtiu, dá o joinha e manda uma pergunta pra gente responder em breve. Happy coding! 💻

p.s. Quando eu estava para publicar esse texto eu percebi que o mestre fidelissauro tem uma versão SUPER completa sobre o assunto. Coloquei o link nas referências.


## Referências

* https://leimao.github.io/blog/Python-Concurrency-High-Level/
* https://www.baeldung.com/cs/concurrency-vs-parallelism#:~:text=Simultaneously%20executing%20processes%20and%20threads,terms%20for%20the%20parallelism%20taxonomy.
* https://medium.com/@peterlee2068/concurrency-and-parallelism-in-java-f625bc9b0ca4
* https://fidelissauro.dev/concorrencia-paralelismo/
