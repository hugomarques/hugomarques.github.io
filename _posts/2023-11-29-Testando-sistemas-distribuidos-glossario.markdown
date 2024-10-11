---
title: "Testes - Testando sistemas distribuídos: Fundamentos, conceitos e glossário"
published: true
description: 
tags: #testes #java #bolhadev #microservicos
cover_image: https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jbgtxv83hxz476hzrmld.png
# Use a ratio of 100:42 for best results.
# published_at: 2023-11-01 18:33 +0000
---

Opa pessoal, muita gente elogiou o último post [[Relato] Testes atrasaram o meu projeto 😱 /s](https://dev.to/hugaomarques/relato-os-testes-atrasaram-meu-projeto-s-3lgb). Um dos feedbacks que recebi foi sobre definir melhor os tipos de testes para ajudar no entendimento daquele e de futuros artigos nessa trilha de testes.

Definir os tipos de testes é um tema espinhoso e sem muito consenso para ser honesto. O meu objetivo não é dar a palavra final no assunto, mas sim estabelecer um vocabulário padrão que usarei para me comunicar com vocês. Assim, toda vez que você falar comigo sobre testes, no mínimo, nós teremos um lugar-comum para a discussão.

## Tipos de testes

Os tipos de testes listados nesse artigo são os mais comuns nas empresas que trabalhei e normalmente são esses os tipos que definem a linguagem que uso para me comunicar.

Notem que estou restringindo esse post apenas à testes funcionais, ou seja, testes que validam o funcionamento de negócio da aplicação. Testes não-funcionais, como testes de segurança, de desempenho ou stress também existem, mas não são o foco agora.

![A tradicional pirâmide de testes](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/7fibfa1bl5ja28tsg05x.png)

> A tradicional pirâmide de testes. Mas será que ela tá certa mesmo? 🤔

Tendo dado todos esses *disclaimers*, vamos ao que interessa.

## Tabela de conteúdo
- [Testes de unidade](#testes-de-unidade)
- [Testes de integração](#testes-de-integração)
- [Testes *end-to-end* ou e2e](#testes-end-to-end-ou-e2e
)
- [Canários](#canários)
- [Testes de canário](#testes-de-canário)
- [Mas que testes devo usar](#mas-que-testes-devo-usar)
- [Conclusão](#conclusão)

## Testes de unidade

Os testes de unidade são os testes mais comuns na maioria dos projetos. Eles validam se nossas classes, métodos, funções estão funcionando como esperamos. Tem pessoas que só consideram testes de unidade quando apenas uma classe é testada em isolamento. Eu não me prendo a essa regra. 

Na minha opinião, é mais importante garantir que a funcionalidade que se deseja testar seja testada da forma mais fácil e efetiva possível. Como assim fácil e efetiva?

Fácil porque devemos escrever muitos testes. É importante que cada teste seja o mais direto possível de ser escrito. Logo, se é tranquilo eu criar vários objetos juntos e testá-los juntos, eu irei testar esses objetos trabalhando em conjunto. Da mesma forma, se fica difícil escrever o teste com os objetos trabalhando em conjunto, nós escrevemos o teste para o objeto específico e usamos *stubs* e/ou *mocks* para isolar as dependências.

Já no campo de testes efetivos, é importante que os testes exercitem o nosso código da forma mais interessante possível. Vejamos o exemplo abaixo:

```java

public class Calculadora {

    public static double realizarOperacao(double a, double b, OperacaoAritmetica operacao) {
        return switch (operacao) {
            case ADICAO -> a + b;
            case SUBTRACAO -> a - b;
            case MULTIPLICACAO -> a * b;
            case DIVISAO -> b != 0 ? a / b : Double.NaN;
        };
    }
}

public record AdicaoComando(double a, double b) {

  public int execute() {
    return Calculadora.realizarOperacao(this.a, this.b, 
  OperacaoAritmetica.ADICAO);
  }

}

```

Temos uma classe que implementa uma calculadora e outra classe que é um comando gerado para 2 números e usa a nossa calculadora para executar a operação daquele comando. Ignoremos o fato do `AdicaoComando` ser um objeto super simples, o exemplo poderia ser qualquer classe estática usada por outra classe.

Quando escrevemos um teste para a classe `AdicaoComando`, nós mockamos a `Calculadora`? Na minha opinião, NÃO! O objeto calculadora é fácil de ser usado, não tem porque não usar o código o mais próximo possível do que acontece em produção.

Como testaríamos a classe `AdicaoComando`?

```java
public class AdicaoCommandTest {

    @Test
    void executeDeveRetornarSomaCorreta() {
        // Arrange
        AdicaoCommand adicaoCommand = new AdicaoCommand(3.0, 2.0);

        // Act
        double resultado = adicaoCommand.execute();

        // Assert
        assertEquals(5.0, resultado, 0.0001); // Considerando uma precisão de 0.0001 para comparação de números de ponto flutuante
    }
}
```

Note como não usamos mocks. 

Mas você deve estar se perguntando: vale à pena escrever um teste para essa classe `AdicaoComando` sendo ela tão simples? SIM, vale! Lembre-se que testes também existem para prevenirem erros futuros. 

Imagine que no futuro, sem querer, alguém altera nossa classe `AdicaoComando`:

```java
public record AdicaoComando(double a, double b) {

public int execute() {
  return Calculadora.realizarOperacao(this.a, this.a, OperacaoAritmetica.ADICAO);
}

}
```

Erro simples, mas que sem testes, poderia passar sem percebemos. Você percebeu o erro acima? 😆

### Mas eu preciso escrever testes de unidade para todas as classes?

Não. Lembre-se, é importante que testes sejam fáceis e efetivos. Testes de unidade não são ótimos para testar classes que interagem com componentes externos ao seu sistema ou que haja dependência do seu framework. Por exemplo, no nosso *controller* abaixo:

```java
@RestController
@RequestMapping("/api")
@Validated
public class ExemploController {

    @PostMapping("/exemplo")
    @ResponseStatus(HttpStatus.CREATED)
    public String criarExemplo(@Valid @RequestBody ExemploRequest exemploRequest) {
        // Lógica de processamento aqui
        return "Exemplo criado com sucesso!";
    }
}
```

Note como a nossa *request* é validada através do *Spring* usando as anotações `@Validated` e `@Valid`. Nesse caso, um teste de unidade não faz um bom trabalho para exercitar esse cenário de validação. 

Como regra geral, eu gosto de usar testes de unidade para validar algoritmos internos da minha aplicação, por exemplo, uma regra de negócio que não depende do framework e nem de uso do banco de dados. No nosso exemplo acima, testar o switch/case da calculadora é um excelente candidato para testes de unidade, da mesma forma, a nossa classe `AdicaoComando`. 

Mas se testes de unidade não são suficientes? O que podemos fazer a respeito? É aqui que entram os testes de integração.

## Testes de integração

Nos testes de integração (também chamados de Testes de Serviço no livro *Building Microservices*), nós começamos a usar as nossas classes trabalhando em conjunto com outros componentes: *frameworks*, banco de dados, filas de comunicação, etc. Mais importante ainda, nós subimos a nossa aplicação em um servidor que os nossos testes vão invocar.

![Diagrama com um teste de integração invocando um serviço e suas dependências.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/jbgtxv83hxz476hzrmld.png)

### Por que precisamos de testes de integração?

Como você deve ter percebido, os testes de unidade são bem restritos. Eles não cobrem nossas validações e funcionalidades criadas pelos nossos *frameworks*, nem restrições impostas pelos nossos protocolos de comunicação ou estruturas de armazenamento. Com o uso de testes de integração, nós conseguimos cobrir todos esses cenários extras, já que com testes de integração rodamos a nossa aplicação como parte dos testes.

### Mas como eu escrevo testes de integração?

Felizmente, os *frameworks* modernos já contém ferramentas que nos permitem escrever testes de integração de uma forma muito próxima de como escrevemos testes de unidade. Por exemplo, para testar o nosso *controller* do exemplo anterior, nós poderíamos utilizar o próprio *Spring* e sua anotação `@SpringBootTest`: 

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class ExemploControllerTest {

    @Autowired
    private TestRestTemplate restTemplate;

    @Test
    void criarExemplo_QuandoIdadeMenorQue18_DeveRetornarBadRequest() {
        HttpHeaders headers = new HttpHeaders();
        headers.setContentType(MediaType.APPLICATION_JSON);

        String requestBody = "{\"idade\": 17}";
        HttpEntity<String> requestEntity = new HttpEntity<>(requestBody, headers);

        ResponseEntity<String> response = restTemplate.exchange("/api/exemplo", HttpMethod.POST, requestEntity, String.class);

        assertEquals(400, response.getStatusCodeValue());
        assertEquals("A idade deve ser no mínimo 18", response.getBody());
    }

    // Outros testes semelhantes para os diferentes cenários
}
```

Como esse teste através da anotação `@SpringBootTest` sobe a aplicação real tal qual ela rodando em produção, é possível validarmos como o *Spring* valida utilizando as anotações `@Valid` e `@Validated`.

### Eu não trabalho com *Spring*. Uso um framework da casa e/ou outra ferramenta de nuvem como `AWS Lambda`. Como posso escrever testes de integração?

É possível escrever testes de integração para aplicações que rodam sem um framework de apoio. Vamos aos exemplos:

Primeiro, vamos escrever a mesma aplicação do *controller* acima, mas usando apenas Java padrão:

```java
public class IdadeValidationServer {

    public static void main(String[] args) throws IOException {
        // Criação de um servidor HTTP na porta 8080
        HttpServer server = HttpServer.create(new InetSocketAddress(8080), 0);

        // Criando um contexto para lidar com solicitações no caminho "/validarIdade"
        server.createContext("/validarIdade", new IdadeValidationHandler());

        // Iniciando o servidor
        server.start();
    }

    // Implementação do manipulador (handler) para o caminho "/validarIdade"
    static class IdadeValidationHandler implements HttpHandler {
        @Override
        public void handle(HttpExchange exchange) throws IOException {
            // Obtendo a solicitação do corpo
            String requestBody = new String(exchange.getRequestBody().readAllBytes(), StandardCharsets.UTF_8);

            // Realizando a validação de idade
            int idade = Integer.parseInt(requestBody.trim());
            String response = validarIdade(idade) ? "Permitido" : "Proibido";

            // Configurando cabeçalhos de resposta
            exchange.getResponseHeaders().set("Content-Type", "text/plain");
            exchange.sendResponseHeaders(200, response.length());

            // Enviando a resposta
            try (OutputStream os = exchange.getResponseBody()) {
                os.write(response.getBytes(StandardCharsets.UTF_8));
            }
        }

        // Função de validação de idade
        private boolean validarIdade(int idade) {
            // Simulando a validação
            return idade >= 18;
        }
    }
}
```

Obviamente, nosso código é muito maior (a solução acima é apenas um exemplo e não é adequada para uso em produção, você foi avisado(a) 😬). E como diabo eu testo isso?

Primeiro, você precisa rodar o servidor. Você pode fazer isso na sua IDE de escolha ou usar o java para rodar a aplicação. 

```
# Compila o .java
javac IdadeValidationServer.java
# Executa o .class gerado. O servidor deve estar rodando na porta 8080. Use ctrl+c para interromper a execução.
java IdadeValidationServer 
```

E agora podemos escrever o teste:
 
```java
class IdadeValidationServerTest {

    @Test
    void testIdadeValidationServer() throws IOException {

        // Faz uma solicitação HTTP para validar a idade (altere a idade conforme necessário)
        int idade = 20;
        URL url = new URL("http://localhost:8080/validarIdade");
        HttpURLConnection connection = (HttpURLConnection) url.openConnection();
        connection.setRequestMethod("POST");
        connection.setDoOutput(true);

        connection.getOutputStream().write(Integer.toString(idade).getBytes());

        // Lê a resposta do servidor
        BufferedReader reader = new BufferedReader(new InputStreamReader(connection.getInputStream()));
        String response = reader.readLine();
        reader.close();

        // Para os propósitos deste exemplo, esperamos que a resposta seja "Permitido"
        assertEquals("Permitido", response);

        // Fecha a conexão e desliga o servidor
        connection.disconnect();
    }
}
``` 

Também é possível executar o próprio servidor a partir do teste. Basta adicionar:

```java
...
    @Test
    void testIdadeValidationServer() throws IOException {
        // Inicia o servidor (pode ser feito automaticamente usando algo como o JUnit5 extensions)
        IdadeValidationServer.startServer();
// Código de teste
        IdadeValidationServer.stop();
}
```

Eu já trabalhei em ambientes onde não era possível executar o servidor como parte integrada dos testes. Por isso, achei importante mostrar as duas formas.

Agora que temos um exemplo sem framework, e se eu quiser, por exemplo, testar uma aplicação rodando no `AWS Lambda` ou no `AWS Step Functions`? Na minha experiência, o desafio está em simular o ambiente. Código que roda *"serverless"* no cloud se beneficia bastante de testarmos o código justamente onde ele é executado em produção, lembre-se, testes devem ser *efetivos*. Essa diferença tem por consequência que antes de executarmos nossos testes, nós precisamos fazer *deploy* do código para o cloud, por exemplo, uma conta AWS do desenvolvedor. Após feito o *deploy*, nós podemos invocar o código usando a API do AWS ou chamadas HTTP se o nosso lambda estiver usando um HTTP endpoint.

```java
public class LambdaIntegrationTest {

    @Test
    public void testLambdaInvocation() {
        // Configure a client
        LambdaClient lambdaClient = LambdaClient.create();

        // Configurar a solicitação de invocação
        InvokeRequest invokeRequest = InvokeRequest.builder()
                .functionName("nome-da-sua-funcao-lambda")
                .build();

        // Chamar a função Lambda
        InvokeResponse invokeResponse = lambdaClient.invoke(invokeRequest);

        // Verificar a resposta
        String responseBody = StandardCharsets.UTF_8.decode(invokeResponse.payload()).toString();
        assertEquals("Hello, World!", responseBody);

        // Fechar o cliente
        lambdaClient.close();
    }
}
```
Também é possível usar ferramentas que simulam o cloud em sua máquina local. Eu não tenho muita experiência com esse tipo de solução, mas posso inferir que se ganha agilidade e custo em troca de acurácia, dado que por mais perfeito que o ambiente local seja, ele não é igual ao ambiente de nuvem. 


### E as dependências? Como eu faço pra executar testes de integração com dependências?

Aqui existem algumas opções:
1. Você pode usar stubs/mocks para suas dependências de fora do seu domínio.
2. Você pode usar suas dependências reais, especialmente se forem operações só de leitura (Lembre-se que testes precisam ser fáceis de serem escritos). 
3. Você pode subir servidores que simulam suas dependência. Uma versão mais rebuscada dos mocks.

O importante é que essa decisão não comprometa a velocidade de execução e a confiabilidade dos seus testes. Foque em escrever testes efetivos que exercitam o comportamento que você está buscando.

### Mas executar o teste na nuvem não seria um teste *end-to-end (e2e)*?

Depende 🤠. Se a sua aplicação envolve apenas o lambda e alguns outros serviços da AWS, eu considero que isso ainda é um teste de integração. A diferença é que em vez do *Spring*, nós temos o lambda como suporte de execução da nossa aplicação. 

Na minha opinião, a diferença entre um teste de integração de um teste e2e é a quantidade de sistemas envolvidos e a quantidade de contextos de negócio. Se o teste se mantém restrito a um contexto de negócio, que envolve apenas o seu time direto e um serviço, eu ainda considero que estamos falando de um teste de integração, independente se ele roda em container, em lambda, na máquina local.

### Mas se testes de integração são tão bacanas, por que precisamos de testes de unidade?

Em aplicações simples, com poucas regras de negócio, onde o foco é salvar os dados no banco de dados, ou levar o dado de entrada até o local de saída, os testes de integração brilham. Existem até modelos de testes que focam em testes de integração, como o modelo *honeycomb* ou coméia.

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/f3z7duttme7nn5a5sqdl.png)

Porém, tudo depende do contexto. Se você tem uma classe como a nossa calculadora com métodos simples, é exaustivo levantar todo o contexto do *Spring* só para validar que as 4 operações são efetuadas com sucesso. Nesse caso, é melhor escrever testes de unidade para validar o negócio e focar os testes de integração nas interações com banco de dados, servidores, *frameworks*.

Em um dos meus projetos recentes, estive envolvido com trabalhos que processam dados de entrada A, realizam uma computação mínima e armazenam os resultados na saída B. Nesse contexto, eu prefiro utilizar testes de integração. Por quê? É mais importante verificar que o dado transita com sucesso ao longo do fluxo, que o *Spring* injetou os objetos corretos, que a saída foi a esperada do que verificar a lógica interna do negócio que é mínima.

Os testes de unidade, eu uso apenas para detalhar algumas decisões e fluxos internos que não valem à pena serem escritas como testes de integração por serem um pouco mais difíceis de escrever e mais lentos de executar.

## Testes end-to-end ou e2e

![Vários testes e2e testam a aplicação como um todo](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/2dk40gki46pnj1hozcvy.png)

Quando eu me refiro à testes e2e, normalmente a requisição é enviada a um (ou vários) servidor de aplicação. Aqui a aplicação se comunica com todas as dependências tal qual seria em ambiente de produção.

Não é incomum encontrar testes desse tipo exercitando a própria UI. Embora, eu admito que nunca trabalhei nessa situação dado que o meu contexto é mais no *backend*.

Normalmente, eu recomendo escrever poucos desses testes. Com todas essas dependências, o teste é mais difícil de escrever e muitas vezes quando há uma falha a causa não é explícita, gerando os famosos *testes instáveis*.

> "Teste flaky" (ou "teste instável") refere-se a um tipo de teste automatizado que pode produzir resultados inconsistentes ou imprevisíveis, mesmo quando aplicado ao mesmo código sob condições idênticas.

No geral, eu tento escrever o caso feliz da aplicação e modelar testes *e2e* em cima deles. O objetivo é validar  se os testes e2e falharem, uma regressão foi inserida pelo meu time OU por times de dependência. Esses testes também são ótimos para servirem de modelo para testes de fumaça. 

### Por que apenas poucos testes e2e?

Testes e2e são difíceis de escrever e muitas vezes acabam não sendo muito efetivos como diagnóstico já que eles possuem muitos pontos de falha.

## Canários

Os canários são muito usados como monitoramento. Na minha experiência, eles compreendem um *subset* dos testes funcionais para serem executados rapidamente. O objetivo aqui é usar os testes e2e como gerador de tráfego para a aplicação em um ambiente similar a produção. Dessa forma, podemos monitorar as métricas da aplicação e reverter nossas mudanças caso haja um problema no ambiente após a implantação de uma nova funcionalidade.

![Os canários usam os testes e2e para gerar tráfego para a aplicação.](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/x78n7t245otxuctmc3vc.png)
> Os canários usam os testes e2e para gerar tráfego para a aplicação
 
## Testes de canário

O termo é parecido com o canário, mas o objetivo aqui é outro. Essa estratégia de *release* permite fazermos implantação da funcionalidade nova em uma pequena parcela dos nossos servidores. A partir daí, monitoramos esses servidores a fim de verificar se houve alterações nas métricas da aplicação. Caso haja alteração nas métricas, nós interrompemos o *release* da funcionalidade, evitando um impacto maior aos nossos usuários.

## Mas que testes devo usar?

![Image description](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/qbskl03k9l915cegm1qz.jpg)

A resposta clássica seria depende 😎, mas para não deixar você que veio até aqui de mão abanando eu vou tentar dar um direcionamento.

1. Use testes de unidade em classes que implementam algoritmos ou fluxos decisórios que rodam independentemente da infraestrutura ou *frameworks* de sua aplicação. Favoreça esses testes de sua aplicação tem um domínio rico, cheio de regras e cálculos de negócio. 
2. Use testes de integração para exercitar tudo que toca sua infraestrutura ou framework. Esses testes são excelentes para exercitar funcionalidades implementadas com seus *frameworks* ou com sua infraestrutura, por exemplo, *constraints* no seu banco de dados ou requisições HTTP.
3. Use testes e2e para validar cenários básicos de sua aplicação, de preferência, use-os como canários e os associe a mecanismos de *retry* onde caso haja uma falha, o time tem uma boa segurança que algo está errado.

## Conclusão

Testes são um assunto fabuloso e geram bons debates na comunidade. Mais importante do que ficar cravando regra que uma aplicação precisa ter 100x testes de unidade, 10x testes de integração e 1x testes e2e é entender qual o contexto daquela aplicação e pensar de forma estratégica quais os melhores testes que se adéquam aquele contexto.

Por fim, eu deixo aqui uma citação do maravilhoso livro do Maurício Aniche "Effective Software Testing", pg 17.

> Não existe bala de prata em teste de software. Em outras palavras, não há uma única técnica de teste que você possa sempre aplicar para encontrar todos os possíveis bugs. Diferentes técnicas de teste ajudam a revelar diferentes bugs. Se você usar apenas uma técnica, pode encontrar todos os bugs possíveis com essa técnica e mais nenhum.
...
Isso é conhecido como o paradoxo do pesticida: cada método que você usa para prevenir ou encontrar bugs deixa um resíduo de bugs mais sutis contra os quais esses métodos são ineficazes. Os testadores devem usar diferentes estratégias de teste para minimizar o número de bugs deixados no software.

![Mind blowing](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/3vnabh6p78fu39qrtf61.gif)

Eu recomendo demais vocês lerem o livro do Aniche se vocês curtem esse assunto. É isso, espero que vocês tenham curtido o post e até a próxima.