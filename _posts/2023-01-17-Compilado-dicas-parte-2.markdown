---
title: Compilado dicas de carreira - parte 2
published: true
description: Compilado das threads de carreira do Twitter @hugaomarques
tags: #carreira #java #leetcode #algoritmos
# cover_image: https://direct_url_to_image.jpg
# Use a ratio of 100:42 for best results.
# published_at: 2023-01-17 01:46 +0000
---

Olá pessoal, essa é a parte 2 das dicas de carreira. Pra quem caiu de paraquedas, as "dicas de carreira" são threads que eu escrevo no Twitter. 
À pedidos, eu estou compilando as dicas em formato de artigo pra ficar mais fácil a leitura e indexação.
E lembre, essa aqui é a parte 2. A parte 2 você pode achar [nesse link aqui](https://dev.to/hugaomarques/compilado-dicas-de-carreira-parte-1-kif).

## Dica #11 - Bons comentários

Quando for documentar o seu código (comentário/javadoc/pydoc) foque no porquê (why) e não no o quê ou como (what/how). Bons nomes de métodos e variáveis ajudam a tornar o código legível e fácil de entender o que está acontecendo. No entanto, saber o contexto do que a pessoa desenvolvedora estava pensando não tem como. 

Comentários servem para dar contexto "Aqui eu estou usando um timestamp na data X porque isso é necessário para capturar todos os eventos Y". Mais de uma vez eu fui perguntar a uma colega "Por que você fez desse jeito aqui? Você lembra?" e a pessoa não lembrava. Os comentários ajudam a documentar esse contexto perdido. 

Finalmente, dê exemplos na sua documentação. Tá esperando um mapa no formato X? Escreve o formato X na documentação. Vai retornar no formato Y? Escrever o formato Y na documentação para ficar explicíto input/output (para métodos mais complexos).

Do livro "A philosophy of software design": 

> "Comments should describe things that aren't obvious from the code". 

Ele tem um capítulo inteiro dedicado ao tópico, recomendadíssimo: https://amzn.to/3zm37Zj

## Dica #12 - Aprenda Terminal

> Nota: Mais recentemente eu gosto de refrasear essa dica para "aprenda a usar o terminal". 

Aprenda Linux. É sim possível começar na carreira (e não tem problema nenhum) programando no Windows. Eu passei ANOS programando no Windows porque era o ambiente fornecido pelas empresas no Brasil, porém quase todo servidor de produção atualmente roda uma versão de Linux. As grandes empresas todas mantém datacenters rodando linux. Muitas delas te entregam uma máquina na nuvem com Linux no primeiro dia. Mesmo que você use um Mac, muito do conhecimento de terminal é transferível.

Não saber Linux foi uma das coisas que mais me prejudicou quando entrei na Amazon. Depois que eu aprendi o básico do terminal, minha produtividade subiu muito. Eu estou LONGE de realmente saber Linux mas eu conheço o suficiente pra ser produtivo e eu convido que você faça o mesmo

Eu não sei se tem algum curso focado em Linux para desenvolvedores, todo curso que eu vejo é focado na parte de admin. O curso da @LINUXtipsBR cobre praticamente tudo que eu precisei nos últimos anos e mais um pouco. Eu acredito que a 
@AluraOnline tem uma trilha de linux também.

> Nota: Atualmente eu recomendo o curso GRATUITO [The Missing Semester](https://missing.csail.mit.edu/) para você se familiarizar com o terminal. 

## Dica #13 - Times de plataforma tem que ser self-service

Não faça o seu time virar uma barreira na agilidade da sua empresa. Se o seu time provê um serviço para a empresa: ferramentas, processos, APIs. Tente ao máximo fazer com que o processo de usar tais recursos sejam self-service. 

Os próprios usuários devem ser capazes de se registrar e usar a ferramenta. O objetivo do seu time deve ser prover a plataforma e não autorizar/revisar cada uso. Exemplos:
1. Se você mantém um ferramenta como um JIRA, permita que os próprios times administrem os fluxos de automação deles.
2. Se você mantém um serviço que comunica mudanças no código, permite que times usem APIs para registrarem outros serviços que reajam aos eventos que seu serviço publica.
3. Se você mantém um API Gateway/Router. Crie APIs que permites os próprios serviços se registrarem nas rotas que eles desejam atender sem a necessidade de um novo deployment ou revisão de código por parte do seu time. 

Por quê? Se a sua ferramenta for realmente boa e útil, muitas outras pessoas/times vão querer usá-la. Vai ter mais gente usando do que você tem membros do time disponíveis para revisar e aprovar o uso. Logo, se tudo precisa da sua aprovação, você vai atrasar a inovação da sua empresa.

## Dica #14 - Escreva testes automáticos

Essa parece óbvia mas vale ressaltar. Escreva testes automáticos para suas aplicações e features que você desenvolve. Por que?

1. Velocidade, o ciclo de desenvolve - testa da feature fica MUITO mais rápido quando se tem testes automáticos. 
2. Os testes ficam lá no código e protegem a funcionalidade contra regressões introduzidas por qualquer pessoa que vá mexer naquele código no futuro.
3. Confiabilidade, depois de escritos os testes continuam os mesmos. Quando você está testando manualmente sem querer você pode cometer erros e "poluir" o resultado do seu teste.

Exemplo: Uma vez no AWS eu fui ajudar uma pessoa e toda vez que ela ia testar a funcionalidade ela abria a UI do framework pra mandar uma requisição RPC pro servidor. Toda vez, ela tinha que preencher todos os parâmetros, clicar em uma série de botões e enviar. Solução? Eu ajudei a pessoa a escrever um teste automatizado, reduzimos o clica/clica e o copia/cola e o tempo de teste caiu de minutos para segundos.

## Dica #15 - Cuidado com teste e2e.

Cuidado com testes e2e. Dê preferência a testes de unidade ou testes de integração. Testes E2E são caros em termos de setup e tempo de execução. Ao abusar de testes e2e você cria o que chamamos de “monolito distribuído”, ou seja, a única forma de testar o seu software é rodando todo o seu ecosistema de uma vez 🤡. 

O Google tem um excelente artigo sobre os vários problemas com testes e2e: https://bit.ly/3frOCKg. Eu já passei por vários desses problemas na prática.

A minha recomendação costuma ser: tenha o pesado mesmo em testes de unidade, alguns de integração e menos ainda em testes e2e. No blog to Martin Fowler tem mais detalhes sobre esse approach:

https://martinfowler.com/articles/practical-test-pyramid.html

## Dica #16 - Seu trabalho é "resolver problemas".

Dependendo do tipo de empresa que você trabalha o seu trabalho como programador é resolver problemas e não escrever código. Dito isso, não faz muito sentindo calibrar o seu level com base no seu conhecimento de Java, React, Node, etc.

Por exemplo, no meu time atual usamos PHP e Python. Meu level em PHP é próximo de zero, eu sei no máximo a sintaxe mais básica da linguagem. Meu level em Python é provavelmente “iniciante”. Eu sei resolver problemas de algoritmos com python e só. Ainda assim, eu fui contratado como Senior SWE. 

Por quê? Eu consigo identificar gargalos onde pessoas com menos xp não vêem, eu sugiro melhorias no processo que melhoram a performance do time, eu faço code reviews do ponto de vista de arquitetura e escalabilidade. E.g. tem uma mudança saindo que afeta timeout e eu faço uma série de perguntas “por que isso aí?”, “como isso afeta os nossos recursos?”, “que métrica a gente precisa monitora?”. 

O PHP e Python? Eu vou aprendendo aos poucos. Aprender tecnologia é importante (e divertido) mas se lembre que o caminho é saber identificar e resolver problemas.

## Dica #17 - Implementer → Solver → Finder

**Implementer** é a pessoa que normalmente resolve tasks pequenas e tarefas onde o foco é escrever código ou resolver um problema com escopo bem limitado e problema bem definido. Por que implementer? O foco aqui é executar uma boa solução com qualidade.

 No próximo level, temos o **Solver**. Você normalmente sabe que um problema existe ou te passam um problema mas não a solução exata. Cabe ao solver ir atrás de resolver o problema, e qual solução seguir. Os problemas aqui são de complexidade/ambiguidade média.

Por fim, temos o **Finder**. O objetivo do Finder é achar problemas, de preferências antes que eles aconteçam. O Finder está sempre analisando o que o time pode fazer diferente, propondo melhorias e pensando "grande". A autonomia aqui é enorme e os problemas são bem ambiguos.

Algumas considerações finais: 

Essas idéias NÃO são minhas, mas eu achei interessante o conceito pra compartilhar com vocês. Não são regras, muito provavelmente existem mistos entre os levels especialmente quando você vai ficando mais sênior.

## Dica #18 - O método STAR

ntem vi um pergunta no Twitter “Durante uma entrevista, como se dar bem em questões de soft skills” e isso me lembrou de falar do método STAR. Segue o fio:
STAR é um acrônimo formado a partir das palavras em inglês: 
1. Situation
2. Task
3. Action
4. Result

O método STAR descreve como responder perguntas de comportamento/soft skills em entrevistas de forma objetiva e com detalhes suficientes. Vamos entender mais do método:
1. Situation → Você precisa explicar a situação, dar um overview do que aconteceu pra seu entrevistador ter o contexto necessário pra entender sobre o que você está falando. Descreva onde e quando a situação aconteceu.
2. Task → Descreva o problema e o que tinha que ser resolvido, descreve o porquê resolver aquele problema é importante.
3. Action → Explique o que VOCÊ fez. Que ações tomou, o que você pesquisou. Aqui vale enfatizar o você. O entrevistador está interessado em saber como você contribuiu para resolver o problema e não o que o seu time fez. Foque no método e nos passos que você tomou na situação.
4. Result → Fale os resultados que você atingiu com a solução executada na etapa de action. A solução foi entregue no tempo? Foi colocada em produção ou a melhoria foi adotada pelo time? Deu certo ou não? Se não, o que você teria feito diferente? O que você aprendeu?

O STAR é muito útil e nunca me falhou nas entrevistas. Você não precisa ficar engessado na ordem mas, no início, eu recomendo segui-la até que você tenha prática suficiente para seguir o STAR naturalmente. Treinar antes da entrevista com um amigo/colega também ajuda.

Gostou? Dá o like ♥️ e compartilha com o pessoal para mais dicas.

## Dica #19 - O básico que uma pessoa Junior deve saber

> Nota: Essa aqui merece um post único e dedicado a ela. Se houver interesse eu posso escrever mais a respeito depois.

IMHO, pra começar a procurar emprego na área, foque em:

1. Escrever código em 1 linguagem de programação. 
2. Saber o básico de um framework da sua linguagem #1
3. Saber o básico de ler/escrever em um banco de dados
4. Saber o básico de Web/HTML

Segue o fio 🧵👇:
1. Você deve se sentir confortável pra escrever código e resolver problemas na linguagem de sua escolha. Não precisa saber fazer sistemas escaláveis nem nada muito complexo. Saiba: escrever um algoritmo, uma API que escreve/lê de um DB SQL ou uma webapp simples é essencial.
2. Saber um framework vai te ajudar um pouco a ter um pouco mais de contato como software é desenvolvido na indústria, além de te deixar mais proficiente quando você aceitar a tão famigerada vaga.
3. Eu estou assumindo que a pessoa por ser Jr é um iniciante generalista. Se você for total front end talvez você possa pular essa parte mas de toda forma, eu acho que todo mundo se beneficia de ver algo sendo escrito no banco.
4. Novamente aqui depende. Eu assumo um generalista, então saber o básico de web como http, verbos e status codes. Se você for focado em frontend então você pode mergulhar mais fundo em saber um pouco mais de HTML e CSS.

Eu acho isso o básico para um dev Jr., lembrando que um dev Jr é diferente de um estagiário. Um estagiário pode não saber nem codar bem em uma linguagem, a pessoa tá lá pra aprender.

Finalmente, eu tenho um viés pra generalistas. Eu questiono se a pessoa que tá começando já sabe realmente se quer focar em front, back, infra, etc...

p.s. Essa thread também ajuda vocês Sêniors botarem a mão na consciência quando acharem que um Jr tem que chegar hackeando a empresa inteira.

Disclaimer: As dicas assumem um perfil generalista que trabalha com web. Mas é super comum ter outros perfis, tipoquem começa com mobile. Obrigado ao @rodrigodsousa que me chamou atenção sobre isso.

Nesse caso, eu acho que 1 e 2 ainda são válidos. 3 e 4 vai depender bastante.

Em resumo, o importante é, foque nos fundamentos, aprenda a programar bem em 1 linguagem que atue no domínio que você tem interesse. O resto vai aprendendo com o tempo.

## Dica #20 - É mais fácil adicionar do que remover

“É mais fácil adicionar do que remover”. Sempre que estou planejando novas funcionalidades eu lembro o time sobre isso, especialmente, quando discutimos MVPs. Quer saber mais? 

Segue o fio... 👇🧵

Toda vez que você entrega uma funcionalidade pra alguém, por pior que ela seja 😬, sempre vai ter alguém que curte aquela funcionalidade. Se você remove aquela funcionalidade, você está impedindo aquela pessoa(s) de realizar o que ela fazia antes e logo você gera frustração.

Quando você entrega algo menor, as pessoas podem até sentir falta da funcionalidade mas aí você pode me mensurar se é algo que realmente as pessoas precisam e aí sim você pode decidir se vale a pena investir o teu time nisso.

o meu time, sempre que estamos discutindo entregar um MVP eu tento focar em entregar a feature central, bem feita, bem polida e cortar tudo que é “legal de ter” mas não essencial pra depois SE houver necessidade/oportunidade.

✅ Foque em 20% das funcionalidades que atendem 80% do público
❌ Evite fazer 80% das funcionalidades que apenas 20% das pessoas precisam.

## Conclusão

É isso pessoal, com esse post nós já temos o compilado das 20 primeiras dicas. Até a parte 3!
