---
title:  "Java 8 dates com Jackson JSON"
published: true
tags: java lib
---

Recentemente, descobri que Jackson não suportava os tipos disponíveis no pacote *java.time* do *Java 8*. Esse post demonstra o problema e como solucioná-lo.

Embora não seja requisito para o post, eu utilizo [*Lombok*][5] para gerar automaticamente os métodos *get*, *set* e *toString*.

## Dependências

* Adicione as seguintes dependências do *MAVEN*

{% gist hugomarques/eddb46a4d43f457c9af6a1a3949b8a77 pom-no-module.xml %}

## Problema

Tente executar o código a seguir e observe a exceção.

{% gist hugomarques/eddb46a4d43f457c9af6a1a3949b8a77 CreatureJava8Problem.java %}

```Exception in thread "main" com.fasterxml.jackson.databind.JsonMappingException: Can not instantiate value of type [simple type, class java.time.LocalDate]
```

Esse erro ocorre ao tentar converter o *JSON* para qualquer um dos tipos presentes no pacote *java.time*.

## Solução

Adicione a dependência do módulo *Jackson jsr310* ao seu *pom.xml*

{% gist hugomarques/eddb46a4d43f457c9af6a1a3949b8a77 pom.xml %}

Agora, registre o *JavaTimeModule* no *ObjectMapper*.

{% gist hugomarques/eddb46a4d43f457c9af6a1a3949b8a77 CreatureJava8.java %}

Dessa vez, sua classe *Creature* deve ter sido executada com sucesso.

## Bônus: Spring Boot

O [*Spring Boot*][4] já possui *Jackson* integrado por padrão. Dessa forma, o *ObjectMapper* pode ser injetado como dependência em seus componentes. Por exemplo:

{% gist hugomarques/eddb46a4d43f457c9af6a1a3949b8a77 JSONUtils.java %}

Além disso, todas as *requests* recebidas no seus *controllers* são automaticamente tratadas pelo *Jackson*.

Para registrar o módulo *JavaTimeModule* no contexto do *Spring* é necessário redefinir o *bean ObjectMapper*. Podemos utilizar *Java config* do *Spring* ou *XML*, se você preferir. Não abordarei a configuração com *XML* nesse post, para isso verifique a documentação oficial do [*Spring*][3].

{% gist hugomarques/eddb46a4d43f457c9af6a1a3949b8a77 JSONConfig.java %}

[1]: https://github.com/FasterXML/jackson
[2]: https://github.com/FasterXML/jackson-datatype-jsr310
[3]: http://docs.spring.io/spring/docs/current/spring-framework-reference/htmlsingle/#beans-definition
[4]: http://hugodesmarques.com/java/spring/2016/04/13/iniciando-um-projeto-java-com-spring-boot.html
[5]: http://hugodesmarques.com/java/lib/2015/09/07/java-libs-introducao-ao-lombok.html
