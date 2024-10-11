---
title: Compilado dicas de carreira - parte 3
published: true
description: Dicas de engenharia de software parte 3
tags: #iniciante #java #algoritmos #leetcode
---

## Dica #21: Nem tudo é só vitória

Cuidado ao achar que a galera que você segue com sucesso, trabalhando em grandes empresas, só teve vitórias na carreira. Uma coisa que pouco se fala nas redes sociais são as tentativas, fracassos e o trabalho até chegar lá.

É bem comum a gente seguir nas redes sociais pessoas trabalhando em todo lugar do mundo e fazendo coisas que achamos incríveis. Muitas vezes sentimos aquela sensação de que somos impostores e não somos tão bons quanto essa galera...

O que as redes sociais não mostram são as tentativas que deram errado. Por exemplo: quem me segue sabe que eu comecei a postar sobre minha carreira, especialmente nos últimos 6 meses. O que as pessoas não sabem... 😬

Eu reprovei em 3 disciplinas na faculdade, incluindo Java, que é minha linguagem mais forte hoje em dia. Recebi uma porção de "nãos" quando tentei entrar em projetos de iniciação científica...

Na minha cidade, só havia 1 empresa contratando quando me formei. Eu levei não em 2 seleções, só entrei na 3ª. Em 2020, fiz seleção para 9 big techs. Recebi 1 sim, e não era no nível que eu queria. A vaga que eu queria só veio em 2021 😃...

Eu compartilho esse trecho da minha jornada para ser mais transparente com todo mundo e mostrar que muita coisa deu errado no meio do caminho. Não é só vitória e coisas legais, não.

## Dica #22: Como se preparar pra entrevistas de algoritmos e estrutura de dados

Você vê a galera trabalhando em big tech e fica se perguntando como esse pessoal conseguiu? Não tem segredo, não. É tudo muito treino, estudo e preparação. Segue o fio que eu te falo como eu me preparei para as entrevistas... 🧵👇

Nosso foco vai ser estudar algoritmos e estrutura de dados. Existem outros tópicos como system design (fica para a próxima) e behavioral questions (veja a dica #18), mas sem algoritmos não adianta nem tentar. O “algoritmo” 😎 para estudar algoritmos é:
1. Estudar algoritmos e estrutura de dados.
2. Praticar #1 no [LeetCode](http://leetcode.com).
3. Repetir o ciclo.

Você precisa saber os conceitos básicos das seguintes categorias:
- Strings/Arrays
- Listas
- Mapas/Dicionários
- Algoritmos de ordenação
- Busca binária
- Filas
- Pilhas
- Árvores e BSTs
- Heaps/Filas de prioridade
- Grafos
- BFS
- DFS
- Tries
- Union-Find...

Eu nunca encontrei programação dinâmica ou algoritmos gulosos nas entrevistas. Backtracking eu só vi cair uma vez, então, em termos de custo/benefício, recomendo focar nesses aí...

Se você não sabe NADA de algoritmos, eu recomendo começar com o livro "Entendendo Algoritmos: Um Guia Ilustrado": [link para o livro](https://amzn.to/3GHeltA).

Depois dele, você pode checar "Cracking the Coding Interview", que vai te ensinar os algoritmos mas bem focado no preparo para as entrevistas...

Agora que você sabe a base, está na hora de encarar o [LeetCode](http://leetcode.com). O LeetCode é o site que a galera aqui nos EUA usa de fato para se preparar, com centenas de questões reais que já caíram em várias das minhas entrevistas...

Recomendo começar na aba "explore" e selecionar tópicos, por exemplo, "arrays". Esses tópicos vão te ensinar os problemas mais comuns daquela categoria e fortalecer ainda mais a base...

Agora vamos repetir mais exercícios e voltar para a teoria toda vez que se enrolar em algo. Recomendo fazer ainda mais exercícios no LeetCode, desta vez "avulsos" da aba problems...

Ordene por dificuldade e comece fazendo os "easy" primeiro. É normal tomar pau no início. Não desista. Quando estiver mandando bem nos "easy", passe para os medium. Nosso objetivo é ficar bom em resolver problemas medium...

Por que medium? A maioria das empresas não pergunta questões hard. Não acho que vale a pena o esforço. As hard é bom fazer por curiosidade e/ou se você gostar...

Vou deixar aqui umas listas públicas com perguntas boas para ir praticando:

1. ["Must do easy questions"](https://leetcode.com/list/xip8yt562).
2. ["Must do medium questions"](https://leetcode.com/list/xineettm3).
3. ["Community curated 75 questions"](https://leetcode.com/list/x84cr1pj).

## Dica #23: Na entrevista

Você leu a dica #22, se preparou, treinou e agora chegou o dia da entrevista. A pessoa entrevistadora te pergunta um problema de algoritmos. E agora? Como proceder? O que fazer primeiro? Ficou curioso(a)? Segue a thread... 🧵👇

O maior erro que você pode cometer é já sair respondendo. Pior ainda, já sair codificando. Por quê? Como pessoa entrevistadora, a gente está se perguntando:
1. A pessoa assume demais?
2. A pessoa entendeu o problema?
3. A pessoa consegue comunicar o que está pensando?

Se você já sai codificando, pode estar sinalizando para a pessoa que te entrevista que, no trabalho do dia-a-dia, você pode sair codificando para resolver o problema errado ou incompleto...

Para evitar isso, nós vamos usar o seguinte processo:
1. Ouça o problema.
2. Trabalhe com exemplos.
3. Resolva o problema, na força bruta se necessário.
4. Tente otimizar sua solução.
5. Codifique.
6. Teste seu código.
7. Analise sua solução.

Vamos olhar cada um desses passos:

1. **Ouça bem o problema**, preste atenção a qualquer restrição que a pessoa entrevistadora mencionar. Preste atenção aos exemplos.
2. **Peça mais exemplos**, dê exemplos e clarifique se o problema muda com os exemplos que você dá. Tente achar condições extraordinárias que possam alterar sua possível solução.
3. **Resolva o problema na força bruta**, se necessário. Se o primeiro algoritmo que você pensar for sub-ótimo, não tem problema. Converse com a pessoa entrevistadora, explique como o algoritmo resolveria o problema e clarifique de antemão que você sabe dos problemas dessa primeira solução.
4. **Tente otimizar sua solução**, pense se você pode usar mais espaço em memória para ganhar tempo de execução. Tente analisar se você pode usar algumas das restrições para ganhar alguma vantagem. É possível precomputar algum dos passos antes? É possível organizar os dados em uma hashtable para melhorar o tempo de execução? Existe alguma computação duplicada que você pode armazenar e evitar computar de novo?
5. **Escreva o código**, foque em código que funcione, mas mantenha um código organizado. Seja claro com a pessoa entrevistadora sobre suas decisões quanto ao nome de variáveis, funções, etc. Evite refatorar agora, deixe isso para se sobrar tempo no final. Foque em escrever o código principal; se houver funções menores, como pegar um input ou fazer uma troca de variáveis, você pode escrever a chamada de função e falar que fará isso depois se sobrar tempo.
6. **Teste seu código**. Ao terminar, não diga "terminei". Comunique "terminei o código principal e vou testá-lo agora". Rode o seu algoritmo mentalmente com alguns exemplos, execute esses exemplos passo-a-passo com a pessoa entrevistadora.
7. **Analise sua solução**. Faça a análise assintótica (big O) do seu algoritmo. Analise espaço e tempo de execução para o caso médio.

Ufa... é isso 😣. Eu sei que não é fácil, mas com treino e prática vai ficando mais natural.

Grande parte desse processo é descrito em mais detalhes no livro que recomendei antes: ["Cracking the Coding Interview"](https://amzn.to/3GHeltA).


## Dica #24: Diário Dev

Tá preparando o currículo para uma vaga? Precisou escrever o seu documento de promoção e não lembra os resultados que você obteve 1, 2 anos atrás? Segue o fio pra você não esquecer mais suas contribuições e facilitar sua vida na hora de escrever o documento com suas conquistas... 🧵

Em meados de 2020, eu tive que escrever o meu documento de promoção descrevendo todas as minhas contribuições e resultados. Aquele documento era o último passo na minha promoção de SDE 2 para Senior SDE...

O problema? Eu tinha feito muitos projetos bacanas em 2018, 2019 e 2020. Muitos artefatos como métricas, gráficos, code reviews eu nem lembrava mais onde estavam...

Resultado: Eu demorei um tempão escrevendo o documento. Catando e-mails antigos, procurando por code reviews, documentos de design, feedbacks que dei para os meus colegas. Depois dessa experiência, eu passei a ensinar todo mundo a manter um “diário dev” 📔...

📔 Mas o que é um diário dev?
No diário dev, você vai escrever tudo que fez de bacana com links para artefatos, code reviews, design docs, métricas. Imagine escrever um tweet onde você lista o que contribuiu para a empresa naquela semana ou mês...

❓ O que escrever no diário dev? Tudo que você contribuiu que gerou resultados.
1. Fez deploy de um código que reduziu a latência do serviço? Escreva no diário, anexe link para a code review e um printscreen do gráfico.
2. Fez o design de um sistema? Anote no diário e coloque o documento no diário.
3. Fez uma code review bacana para outra pessoa desenvolvedora? Anote no diário e coloque o link para a code review que demonstre seus comentários/boas práticas.

⏰ Quando escrever no diário dev? Você deveria atualizar algo pelo menos 1x por semana. 
Não deixe passar mais de 1 mês sem escrever. Você vai esquecer. É igual a código, em 2 meses você não vai lembrar mais os detalhes do que fez...

🎉 Se você tiver tudo documentado, quando chegar a hora de escrever o currículo ou documento de promoção, é só uma questão de passar o olho no diário e escolher seus melhores exemplos. E aí, só sucesso! 🥳

## Dica #25: Material de estudo para algoritmos.

Esse curso de Princeton tem 2 partes. A linguagem usada é Java. Recomendo:
[Algorithms, Part I](https://coursera.org/learn/algorithms-part1)
Offered by Princeton University.

A Coursera também tem um curso de Stanford, nunca fiz mas dizem que é excelente também:
[Algorithms](https://coursera.org/learn/algorithms)
Offered by Stanford University. 

O William Fiset é um engenheiro da Google, eu acho. O canal dele tem várias explicações sobre algoritmos:
[WilliamFiset](https://youtube.com/WilliamFiset)

Se você quiser praticar, pode tentar as trilhas na aba explore do LeetCode (é preciso estar logado):
[LeetCode Explore](https://leetcode.com/explore/)

Tem pessoas que preferem o HackerRank:
[HackerRank](https://hackerrank.com)

Se você prefere livros, existem vários:

Focado em entrevistas, você pode pegar o "Cracking the Coding Interview: 189 Programming Questions and Solutions":
[Cracking the Coding Interview](https://www.amazon.com/dp/0984782850)

Se você quer algo leve para iniciantes, pode começar com o “Entendendo Algoritmos: Um guia ilustrado para programadores e curiosos”:
[Entendendo Algoritmos](https://www.amazon.com.br/dp/8575225631)
Um guia ilustrado para programadores e outros curiosos. Um algoritmo nada mais é do que um procedimento passo a passo para a resolução de um problema. Os algoritmos que você mais utilizará como um...

O curso que eu falei de Princeton também tem um livro texto chamado “Algorithms”:
[Algorithms (4th Edition)](https://www.amazon.com.br/dp/032157351X)

Na parte de clássicos, você pode olhar o famoso livro do Cormen 
[Introduction to Algorithms, fourth edition](https://www.amazon.com/dp/026204630X)

Um outro clássico que vi muita gente usando é o “The Algorithm Design Manual”:
[The Algorithm Design Manual](https://www.amazon.com/dp/1848000693).