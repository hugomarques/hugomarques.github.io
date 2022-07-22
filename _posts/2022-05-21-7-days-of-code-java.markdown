---
layout: post
title: 7 Days of Code - Java
published: true
description:
tags: java, iniciante, 7daysofcode, OO
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/4hxawn5gsmzyf8omfuxa.png
---

Nesse post eu vou compartilhar a minha jornada no [7daysofcode](7daysofcode.io).

A idéia do 7 days of code é receber um email todo dia com uma pequena tarefa pra ser executada na linguagem escolhida para o desafio. No meu caso, eu decidi fazer a versão em Java.

Nesse post eu vou descrever as minhas dificuldades, pensamentos, aprendizados e tentar compartilhar um pouco como eu resolvo problemas. Vamos lá!

## Dia 1 - Chamando uma API
O problema começa de forma bem direta. Invocar a API do IMDB e obter um JSON com os 250 filmes mais populares. Eu tentei manter a coisa toda o mais simples possível, o código todo dentro de um única classe e um único método `main` embora o meu instinto tente me guiar para encapsular os objetos em classes. Eu literalmente falei para mim mesmo *"Keep it simple"*.

A URL no email do desafio estava errada. A URL que eu recebi era `https://imdb-api.com/en/API/Top250Movies/<apiKey>https://imdb-api.com/en/API/Top250Movies/`. Alguém mais experiente vai bater o olho e ver o `https//` concatenando à `<apiKey>` e vai notar o erro. A URL correta é `https://imdb-api.com/en/API/Top250Movies/<apiKey>`, onde <apiKey> deve ser substituída pelo key gerada para você.

> **Dica:** Em vez de escrever a chave do IMDB que você criou no código e apagar ela toda vez que for postar o código no github (você não vai querer as pessoas tendo acesso à sua chave), você pode armazená-la em uma variável de ambiente.
É assim, por exemplo, que você pode obter alguns valores quando você executa código no ambiente do AWS Lambda. **Nunca escreva suas keys diretamente no código**, uma hora você vai esquecer, dar um commit e sua máquina no AWS vai ser usada pra minerar bitcoins.

## Dia 2 - Fazer parse do JSON
Naturalmente eu acho mais fácil correr pro OO pra modelar cada filme mas eu me decidi seguir o desafio e me manter focado no que era pedido cada dia.

Mesmo depois de 14 anos de carreira, regex AINDA são confusas. Eu precisei pesquisar BASTANTE pra lembrar como extrair grupos e obter informação entre dois caracteres.

Inicialmente eu tinha pensando em extrair todos os atributos de uma vez e armazená-los em um `Map<Map<Atributo, Valor>>`. Eu desisti dessa idéia por ser ligeiramente mais complicada e optei por manter o código simples.
```Java
List<titles> titles = parseTitle(moviesJson);
```
A API de Matcher também me fez perder um tempo. Depois de chamar o método `matcher` se você não utilizar `.find()` ou `.match()` o método `.group(0)`vai retornar. Um exemplo de uso:
```Java
var matcher = Pattern.compile("\""+attribute+"\":\"(.*?)\"").matcher(movie);
    matcher.find(); //precisa chamar .find() antes de group()
    return matcher.group(1);
```

Foi importante em alguns casos entender o conceito de regex greedy vs non-greedy e quando usar a expressão `(.*)` vs `(.*?)`. Os resultados podem ser bem diferentes.

> **Dica**: Eu acabei escrevendo o método parseAttribute por preguiça de ficar duplicando código nos métodos, o que no final é uma boa prática (DRY). Ou seja, quando você notar que vai fazer muito da mesma coisa e ficar com preguiça... pense se aquilo não pode ser abstraído.

## Dia 3 - Modelagem de domínio com OO
Hora de modelar OO, só de ler o desafio eu já dei um respiro por estar na minha zona de conforto.

Eu fiz o desafio em uns 10 min onde uns 7 min foi usando atalhos do intelliJ pra gerar getters, toString, equals and hashCode e o resto copiando o código do `Main.java` para o novo `Movie.java`.

Algumas decisões:
1. Movies são imutáveis. Depois de instanciados eles não podem ser alterados.
2. Eu gosto de usar static factory methods (effectiva java item #1), então pra mim foi natural criar o método `instanceOf`.
3. Stream API faz a operação inteira ficar mais enxuta. Em vez de:
```Java
var movies = new Arraylist<Movie>();
for each(String movieString:moviesAsJson) {
  movies.add(Movie.instanceOf(movieString);
}
```

Nós podemos fazer:
```Java
final var movies = moviesJson.stream()
            .map(Movie::instanceOf)
            .collect(Collectors.toList());
```

> **Dica**: Note que equals e hashcode não fazem parte do desafio mas eu tenho esse hábito de SEMPRE implementar equals e hashcode para os meus objetos de domínio (Effectiva Java #11).
Eu também sempre implemento o toString (Effective Java #12). Mais de uma vez eu vi um problema em produção e eu não conseguia ver a causa porque a pessoa Dev não implementou o toString().

## Dia 4 - Escrevendo HTML
O desafio foi bem direto. Eu gosto do modelo de ter um HTML generator mas fiquei na dúvida se isso fosse um projeto "real" se eu manteria o gerador de HTML junto do mesmo objeto que escreve na saída com o PrintWriter. Talvez sim já que o writer em si é abstrato e pode escrever em diversos canais diferentes.

A parte bacana desse código mais uma vez foi decompor o problema usando a API de streams.

```Java
  public void generate(List<Movie> movies) {
    try {
      outputWritter.write(HEAD);
      final var moviesAsHtml = movies.stream().map(HTMLGenerator::fromMovieToMovieHTML).collect(
          Collectors.joining());
      outputWritter.write(String.format(HTML_BODY_TEMPLATE, moviesAsHtml));
    } catch (IOException e) {
      throw new RuntimeException(e);
    }
  }
```

Em resumo a operação acima faz:
1. O map mapeia a Lista de Movie para uma Lista de String.
2. O joining "junta" essa lista de String em uma única String. Como eu não passei nenhum separador, as Strings são coladas umas nas outras.

> **Dica**: O outro ponto interessante é o tratamento de exceções. Notem que eu encapsulo a checked exception em uma Runtime exception. Por quê?
1. Se der erro ali, não tem nada que eu possa fazer. (Effective Java items #70 e #71)
2. Eu não quero expôr esse detalhe para os meus clientes. Imagine que eu tenho 10 classes chamado esse método e no futuro nós mudamos a exception. Agora nós temos 10 locais quebrados no código.

Escrever o HTML e o CSS foi uma dor, faz ANOS que eu escrevi HTML e pra ser sincero eu nunca me senti confortável com isso. Eu espero que isso ajude você leitor a entender que a área tech é GIGANTE e você pode sim ter uma carreira de sucesso sem saber de tudo 🙂.

## Dia 4 - Extra - Usando auto closeable.

Acabei percebendo que no meu commit eu não fechei o meu writer. Meu primeiro bug 🤭. Pra corrigir isso vamos usar a feature de auto closing do Java:

```Java
try (var printWriter = new PrintWriter(new FileWriter("movies.html"))) {
  final var htmlGenerator = new HTMLGenerator(printWriter);
  htmlGenerator.generate(movies);
}
```

Pra quem não conhece o código acima é a mesma coisa de fazer:

```Java
try {
  var printWriter = new PrintWriter(new FileWriter("movies.html"))
  final var htmlGenerator = new HTMLGenerator(printWriter);
  htmlGenerator.generate(movies);
} finally {
  printWriter.close();
}
```

Bem mais enxuto, né?

## Dia 5 - Mais encapsulamento e código limpo

No dia 5 nós estamos fazendo mais encapsulamentos e o nosso main ficou bem bonitinho. A classe tem poquíssimas dependências e o método só é responsável pela orquestração da lógica. Algumas decisões que eu consigo pensar:
1. Criar o novo objeto pro ImdbClient com a API_TOKEN no construtor faz sentido. Clients normalmente não mudam muito na execução do código.
2. Já o ImdbParser, embora eu possa passar o json no construtor a desvantagem é que pra cada parsing eu preciso criar um novo parser quando o parser em si é muito mais uma função do que um objeto que guarda estado. Uma alternativa seria:
```Java
final var movies = new ImdbJsonParser().parse(responseContent);
```
Assim podemos usar o mesmo parser pra tratar diferentes chamadas da API.

> **Dica**: Mais uma vez eu usei o intelliJ pesado. `alt + ENTER` é seu atalho amigo, com isso você cria métodos que faltam, classes, corrige os imports, unifica múltiplos catches em uma expressão única. Talvez eu faça um demo depois sobre como usar esse atalho no intelliJ.

## Dia 6

Criar as abstrações para API e para o Parser foi interessante. Eu já estava esperando ir nesse caminho. Faz sentido termos uma triple: `<API, Parser, Content>` para cada tipo de content. De fato, eu já implementei algo parecido em mais de um sistema que eu trabalhei.

A parte da Marvel foi a mais chata. Eu não consegui entender os requisitos pra saber exatamente qual API exatamente deveria ser chamada. Depois eu fiquei confuso como fazer um Hash MD5 no Java. Eu admito que esses 2 eu olhei a solução final pra me dar um norte pra onde eu devia ir.

Esse método foi o método que mais me causou confusão no projeto e que eu tiver que ir estudar pra entender o que realmente era feito ali:
```Java
private String md5Hex(String value) {
    try {
      final var messageDigestMD5 = MessageDigest.getInstance("MD5");
      final var digest = new BigInteger(1, messageDigestMD5.digest(value.getBytes(StandardCharsets.UTF_8)));
      return digest.toString(16);
    } catch (NoSuchAlgorithmException e) {
      throw new RuntimeException(e);
    }
  }
```

Depois foi hora de fazer mais parsing. Dessa vez eu não consegui generalizar tão bem quanto a API do Imdb, logo eu deixei as regex em cada método de parser.

Uma coisa que eu tava incomodado e que aqui fez sentido foi transferir todo o parser pros objetos de parser e deixar os objetos de modelo sem nada relacionado a JSON. É mais limpo e faz sentido que o modelo não saiba nada sobre o formato do input.

## Dia 7

Lição simples mas bacana sobre como usar sort e implementar tanto a interface `Comparable` quanto usar o `Comparator`. Um ponto que não fica claro nos requisitos é: Quando eu juntar as duas listas, elas devem ser ordenadas usando o que?

Enquanto no Imbd nós usamos o "Rank", na Marvel nós temos o "Rating" e Rating não se traduz como "Nota" mas sim como classificação. Mas tudo bem com isso, é o tipo de problema que ocorre no trabalho e você discute com o time pra resolver a ambiguidade.

## Conclusões finais

Eu adoro esse tipo de problema/atividade de programação. Me lembra muito a época de faculdade e alguns projetos que fiz durante algumas seleções em empresas que não usam "whiteboard questions".

O código que eu escrevi [está disponível no github](https://github.com/hugomarques/7daysofcode-java). Eu coloquei tags em cada commit se você quiser ver o meu passo-a-passo através do desafio.

O que eu mais gostei?
* Eu gostei no foco em abstração mas ainda sim mantendo as coisas simples.
* Eu adorei que começamos já chamando uma API externa. É bem legal começar com algo mais complexo do que um `hello world`.
* Fazer o print pro html foi uma idéia bacana para quem gosta de ter algo mais visual.
* A maioria dos dias é um exercício bem pequeno que dá pra resolver se dedicando 1 hora por dia.

O que eu senti falta?
* Eu acho que faltou 1 dia dedicado a testes. JUnit e testes são parte integrante do Java e teria sido bacana trabalhar com testes pra fazer o parser. Por outro lado, eu entendo que não adicionando testes a gente conseguiu focar só no Java sem ter que entrar em gerenciamento de dependências já que contrário de linguagens mais modernas, as libs de teste do Java não fazem parte da JDK 😢.
* No dia 6, eu senti falta dos requisitos mais detalhados e de uma explicação sobre aquela API de `md5`. Eu consigo imaginar facilmente alguém mais novo perdido nesse dia.
* Também teria sido legal ter colocado o exercício pra ser invocado do terminal. Eu acho que isso caberia no dia 7 mostrando que o projeto pode ser invocado de diversas fontes e ter diversas `views`. Só uma idéia pra uma v2.0 😗

No geral, eu gostei de todos os dias, quase todos eles eu aprendi algo novo. Eu já vinha querendo criar alguns exercícios em Java pra construir projetos pequenos e exercitar algumas APIs. Esse desafio me mostrou um bom caminho a seguir.

O saldo é extremamente positivo, é um respiro ver esse tipo de exercício disponível pra comunidade exercitar que foge dos tutoriais simples. Uma excelente dose entre *breadth* e *depth*.

Finalmente eu deixo os meus elogios ao pessoal do 7daysofcode.io e ao Paulo como autor dessa versão do exercício. Eu vou acabar implementando ele de novo em Scala 🙂.

Happy coding pessoal 💻!
