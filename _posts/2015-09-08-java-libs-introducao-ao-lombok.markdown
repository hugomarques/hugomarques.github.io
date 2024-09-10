---
title:  "Java Libs: Introdução ao Lombok"
date: 2015-09-08 00:00:00
categories: java lib
comments: true
---
Java sempre foi conhecida como uma linguagem lenta e verborrágica. Apesar de muitas dessas afirmações serem devido ao estigma herdado das primeiras versões da JVM, outras ainda são verdadeiras. Para usuários de outras linguagens como Python ou Ruby, é fato que em Java, tal como em português, muitas vezes se escreve muito para se falar pouco.

Recentemente, diversas *APIs* e *libs* estão surgindo para minimizar o esforço de se escrever código Java e tornar a linguagem mais produtiva para o desenvolvedor; Lombok é uma delas.

##O que é Lombok?

Lombok é uma *lib* disponibilizada no formato de *jar*, que quando adicionado ao seu *classpath*, permite o uso de anotações para descrever diversos comportamentos comumente usados na linguagem.

Este tutorial serve como uma rápida introdução à *API*. 

##Mãos à obra

###@NonNull

{%highlight java linenos %}

public Creature getInstanceOf(@NonNull String name, 
	@NonNull CreatureType creatureType) {
		return new Creature(name, creatureType);
	}
}
{% endhighlight %}

Como você já deve suspeitar, se o método acima for chamado com *name* ou *creatureType* com valor *null*, ele irá lançar uma *NullPointerException*. 

###@Getter @Setter

#### Field level ####

{%highlight java linenos %}
public class Creature {
	
	@Getter 
	String name;

	@Getter @Setter 
	CreatureType type;	
}
{% endhighlight %}

No exemplo acima, *@Getter String name* cria automaticamente um método *public String getName()* para a classe *Creature*.

As linhas *@Getter @Setter CreatureType type* criam os métodos *public CreatureType getType()* e *public void setType(CreatureType type)* para a classe *Creature*.

#### Class level ####
{%highlight java linenos %}
@Getter @Setter
public class Creature {
	
	String name;

	CreatureType type;	
}
{% endhighlight %}

As anotações *@Getter @Setter* presentes na linha 1 geram os seguintes métodos:

* *public String getName()*
* *public void setName(String name)*
* *public CreatureType getType()*
* *public void setType(CreatureType type)*


###@EqualsAndHashCode

{%highlight java linenos %}
@EqualsAndHashCode
public class Creature {
	String name;
	CreatureType type;	
}
{% endhighlight %}


Esqueça seu gerador de *equals* e *hashcode* da sua IDE. A anotação *@EqualsAndHashCode* gera exatamente esse código para você. Outras opções também são possíveis, como exclusão de atributos.

###@ToString

{%highlight java linenos %}
@ToString
public class Creature {
	String name;
	CreatureType type;	
}
{% endhighlight %}

A anotação *@ToString* gera o método *toString()* para o seu objeto. O método retorna uma String no formato *ClassName(field1, field2, ..., fieldN)*. No exemplo acima, a seguinte String é retornada: *Creature(name, type)*. Assim como o *@EqualsAndHashCode*, também é possível excluir atributos.

###@Data

{%highlight java linenos %}
@Data
public class Creature {
	String name;
	CreatureType type;	
}
{% endhighlight %}

Mais conhecida como "adeus java bean verborrágico", a anotação *@Data* gera o mesmo efeito que anotar sua classe com *@Getter*, *@Setter*, *@EqualsAndHashCode* e *@ToString*.

###@AllArgsConstructor

{%highlight java linenos %}
@AllArgsConstructor
public class Creature {
	String name;
	CreatureType type;	
}
{% endhighlight %}

A anotação no exemplo acima gera o seguinte construtor:
{%highlight java linenos %}
public Creature(Strine name, CreatureType type) {
	this.name = name;
	this.type = type;
}
{% endhighlight %}

Da mesma forma que as opções anteriores, também é possível excluir atributos do construtor.

###@Slf4j

{%highlight java linenos %}
@Slf4j
public class CreatureResource {

	public void doGet() {
		log.info("Entrei no doGet");
	}	
}
{% endhighlight %}


O código acima não causa um erro de compilação na linha 5. A variável *log* é criada pela anotação *@Slf4j* similar à seguinte linha de código: 
{%highlight java linenos %}
	private static final org.slf4j.Logger log = 
		org.slf4j.LoggerFactory.getLogger(CreatureResource.class);
{% endhighlight %}

##E ainda tem mais...

As anotações demonstradas aqui possuem diversas configurações possíveis. Além disso, existem inúmeras outras, como por exemplo [@Builder](https://projectlombok.org/features/Builder.html), que implementa o Design Pattern de mesmo nome.
Não irei me aprofundar muito, pois acredito que o [material oficial](https://projectlombok.org) e este [excelente artigo](http://jnb.ociweb.com/jnb/jnbJan2010.html) são mais do que suficientes.

##Polêmicas
Por fim, não posso deixar de mencionar que inúmeras pessoas são contra o uso de ferramentas como Lombok. Um dos principais argumentos defende que anotações são formas de se construir metadata e não implementar novas funcionalidades através de geração de código. Por exemplo, se você remover a anotação *@Slf4j*, o código para de compilar, diferente do que aconteceria se você removesse uma anotação *@Entity* de um bean qualquer.

##Conclusão
No meu dia a dia, tenho achado o Lombok uma ferramenta extremamente produtiva, que quando bem usada, ajuda bastante a diminuir o trabalho repetitivo do programador.

Como toda boa ferramenta, deve ser usado sem abuso. Particularmente, eu evito usar *@Data*, pois não gosto de expor meus *sets*. Além disso, implementações do *equals* e do *hashcode* gerados devem ser validados através de testes unitários para garantir que o comportamento é o esperado pelo desenvolvedor.

Espero que tenham gostado do post e fico aguardando dúvidas e sugestões nos comentários. 



