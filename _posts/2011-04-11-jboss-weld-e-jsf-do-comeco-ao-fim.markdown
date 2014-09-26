---
layout: post
title:  "Jboss Weld e JSF 2.0 do começo ao fim"
date:   2011-04-11 22:19:45
categories: java
---

Neste artigo vamos falar sobre injeção de dependências com Java, agora com o CDI temos uma maneira oficial de realizar as injeções, já é de conhecimento que outros frameworks já realizavam esse tipo de tarefa, porém esse é um conceito novo, pois somente com o JSR 299 (Java Context e Dependency Injection ou simplesmente CDI), o Java tem uma maneira oficial de realizar o DI. Essa JSR é liderada pelo criador do Hibernate e JBoss Seam, Gavin King, devido a isso a JSR foi muito influenciada por esses frameworks. Teremos a oportunidade de ver que a JSR traz muitos conceitos já conhecidos, vindo do Joss Seam e que fez com que o framework conquista-se um grande espaço na comunidade Java.

O que torna a JSR ainda mais poderoso (e confortável) é que ela oferece todas essas facilidades de uma maneira typesafe, ou seja, o CDI nunca confia em identificadores baseados em strings para determinar como os objetos se relacionam. Em vez disso, CDI utiliza a informação de tipagem que já está disponível no modelo de objeto Java, o CDI traz um novo padrão de programação, chamada de anotações qualificadoras, para unir os Beans, suas dependências, seus interceptadores, decoradores e seus consumidores de eventos. A utilização de XML é minimizada para simplesmente informação específica de implantação, ou seja, vamos utilizar muito mais anotações relacionadas à nossa regra de negócios.

Mas ele não é um modelo de programação restritivo, ou seja, ele não diz como você deve estruturar sua aplicação em camadas, ou como você deve lidar com a persistência, ou qual framework web você tem que usar. Você terá que decidir esses tipos de coisas por conta própria, tonando arbitraria os frameworks que você irá utilizar dentro do seu projeto.

### O que é o JBoss Weld? ###

O JBoss Weld é a implementação de referência para a JSR 299: Java Contexts and Dependency Injection ou simplesmente CDI. Ele é um padrão Java para injeção de dependência e gerenciamento do ciclo de vida contextual, onde é especialmente útil no contexto de desenvolvimento de aplicações web. A JSR 299 é parte do padrão proposto pelo Java Community Process (JCP) para uma melhor integração com a plataforma Java EE. Qualquer servidor de aplicação que seja Java EE 6 Full fornece suporte para a JSR-299.

A especificação JSR-299 define um leque com vários serviços complementares para nos ajudar a melhorar a estrutura do código da nossa aplicação. Os serviços que o CDI fornece são:

 - Um ciclo de vida melhorado para objetos stateful, vinculado a contextos bem definidos;
 - Uma resolução segura de tipos (TypeSafe) para injeção de dependência;
 - Interação com objetos através de um mecanismo de notificação de eventos;
 - Uma melhor abordagem para vincular interceptors a objetos, junto com um novo tipo de interceptador chamado decorator, que é mais apropriado para resolver problemas de negócios;
 - Um SPI (Service Provider Interface, ou Interface de Provedor de Serviços) para desenvolvimento de extensões para o contêiner.

### No que iremos trabalhar ao longo do artigo ###
Aplicações que utilizarem o Jboss Weld ou CDI podem rodar em qualquer Servlet Contêiner, tais como Tomcat 6 e Jetty 6, sendo assim, vamos criar uma aplicação para rodar nesses contêineres. Uma vez que eles não são Full Java EE, há a necessidade de algumas adaptações em nosso projeto. Abordaremos como podemos utilizar do gerenciamento do ciclo de vida do Weld junto com JSF 2.0 e visualizar como ficou mais simples quando comparado com versões de JSF anteriores.

Em uma segunda parte do artigo vamos entender melhor como funciona a injeção de dependências do Weld, iremos abordar mais sobre o conceito de qualificadores do CDI, essa é a mudança mais significativa, quando comparado com os modelos que já estamos acostumados a utilizar. Em cima da aplicação já configurada vamos realizar as injeções de dependências na criação de um jogo de dados, onde é sorteado um número e tentamos adivinhar esse número. Nossa primeira classe de implementação será a de um dado com seis faces e iremos trocar por uma com doze faces, para isso vamos utilizar das anotações qualificadoras do CDI, demonstrando como é simples trocar a implementação de referencia de uma interface Java.

### Criando um projeto base ###
Para criar nosso projeto vamos utilizar o Maven2, você pode utilizar sua IDE de preferência como Eclipse ou NetBeans e como Servlet Conteiner o Jetty ou Tomcat. Para inicia nosso projeto base vamos utilizar do Archetype Webapp do Maven2 e vamos criar um projeto, para isso basta rodar comando conforme Listagem 1.

Listagem 1. Projeto base no Maven2
{% highlight shell linenos %}
mvn archetype:generate -DinteractiveMode=n -DarchetypeArtifactId=maven-archetype-webapp -Dversion=0.0.1-SNAPSHOT -DgroupId=br.com.brenooliveira.weld -DartifactId=weld_tutorial
{% endhighlight %}

O comando da Listagem 1 irá criar um projeto Java para web dentro de uma pasta com o nome do nosso artifactId weld_tutorial, nosso groupId é br.com.brenooliveira e a versão do projeto é 0.0.1-SNAPSHOT.

Há a necessidade de editar nosso pom.xml para adicionar as dependencias necessárias para o funcionamento do projeto com o Jboss Weld e o JSF 2.0. Para isso vamos editar nosso pom.xml conforme Listagem 3.

Listagem 3.Adicionando as dependências no pom.xml

{% highlight xml lineos %}
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemalocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/maven-v4_0_0.xsd">
	<modelversion>4.0.0</modelversion>
	<groupid>teste-weld</groupid>
	<artifactid>weld-tutorial</artifactid>
<packaging>war</packaging>
	<version>1.0.0-SNAPSHOT</version>
	<name>Jboss Weld</name>
	<url>http://seamframework.org/Weld</url>

	<repositories>
		<repository>
			<id>jboss-public-repository-group</id>
			<name>JBoss Public Repository Group</name>
			<url>http://repository.jboss.org/nexus/content/groups/public/</url>
			<layout>default</layout>
			<releases>
				<enabled>true</enabled>
				<updatepolicy>never</updatepolicy>
			</releases>
			<snapshots>
				<enabled>true</enabled>
				<updatepolicy>never</updatepolicy>
			</snapshots>
		</repository>
	</repositories>

	<dependencies>
		<dependency>
			<groupid>org.jboss.weld</groupid>
			<artifactid>weld-api</artifactid>
			<version>1.0</version>
		</dependency>

		<dependency>
			<groupid>org.jboss.weld.servlet</groupid>
			<artifactid>weld-servlet</artifactid>
			<version>1.0.0-CR1</version>
		</dependency>

		<dependency>
			<groupid>javax.faces</groupid>
			<artifactid>jsf-api</artifactid>
			<version>2.0.2-FCS</version>
		</dependency>

		<dependency>
			<groupid>javax.faces</groupid>
			<artifactid>jsf-impl</artifactid>
			<version>2.0.2-FCS</version>
		</dependency>
	</dependencies>
	<build>
		<finalname>weldtutorial</finalname>
<plugins>
<plugin>
				<groupid>org.apache.maven.plugins</groupid>
				<artifactid>maven-compiler-plugin</artifactid>
				<configuration>
					<source>1.6
					<target>1.6</target>
				</configuration>
			</plugin>
<plugin>
				<groupid>org.mortbay.jetty</groupid>
				<artifactid>maven-jetty-plugin</artifactid>
				<configuration>
					<!-- Coloca um path com um nome mais amigavel -->
					<contextpath>${build.finalName}</contextpath>
					<!-- Aqui é onde declaramos BeanManager é contruido. -->
					<jettyenvxml>${basedir}/src/test/resources/jetty-env.xml</jettyenvxml>
					<scanintervalseconds>3</scanintervalseconds>
				</configuration>
			</plugin>
</plugins>
	</build>
</project>

{% endhighlight %}

A primeira coisa que temos para observar em nosso pom.xml é que adicionamos o repositório do JBoss, assim teremos certeza que nossos JARs serão encontrados pelo Maven. Logo em seguida adicionamos as nossas dependências do JBoss Weld. A principal dependência é a do weld-api, que implementa toda a nossa JSR-299, ele também já importa o jars necessários do CDI. Como vamos rodar nossa aplicação em um servidor não Full Java EE 6, então precisamos adicionar o jar weld-servlet.jar ele contém o filtro que temos de utilizar me nosso web.xml. Observe, ainda nosso pom.xml, que adicionamos um plugin do Jetty para podermos rodar a nossa aplicação a partir de um comando Maven. Outra feature muito interessante desse plugin é que, ele procura por alterações no código Java e caso encontre ele já atualiza a  aplicação caso esteja rodando, aumentando nossa produtividade durante o desenvolvimento.

### Criando o arquivo beans.xml ###
Para que o nosso servidor identifique que esse é um projeto Weld e inicialize o injetor de dependências corretamente, é preciso criarmos um arquivo vazio, chamado beans.xml dentro da nossa pasta WEB-INF. A simples presença desse arquivo informa ao conteiner que em WEB-INF/classes é um Bean Deployment Archive, então ele passa a procurar por Beans nesta aplicação e ativa os serviços CDI.

### Configurando o JSF 2.0 ###
Ainda temos que fazer a configuração mínima em nosso projeto para o JSF 2.0, assim como já era feito nas versões de JSF anteriores, precisamos criar um arquivo chamado faces-config.xml dentro de nosso WEB-INF. Nesta nova versão de JSF o Facelets já é o view handler padrão, então não há mais a necessidade de declarar ele no nosso arquivo de configurações do JSF. Então nosso faces-config.xml só precisa ter o nó XML principal e mais nada, conforme Listagem 4.

Listagem 4.Configurando o faces-config.xml

{% highlight xml lineos %}
<faces-config version="2.0" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemalocation="

http://java.sun.com/xml/ns/javaee

      http://java.sun.com/xml/ns/javaee/web-facesconfig_2_0.xsd">
</faces-config>
{% endhighlight %}

### Configurando nossa aplicação para rodar em um Web Conteiner ###
Como já mencionado o Jboss Weld pode rodar em qualquer servlet conteiner, tais como Tomcat 6 ou Jetty 6.1, desde que seja adicionado o jar weld-servlet, por isso importamos em ele nosso projeto, o Jetty tem apenas read-only JNDI, ou seja, ele não pode fazer o bind no Manager do Weld automaticamente. Para fazer esse bind entre o Manager e o JNDI nós precisamos criar um arquivo jetty-env.xml, onde vamos declarar uma Referência para no nosso BeanManager, conforme Listagem 5:

Listagem 5.Configurando o jetty-env.xml

{% highlight xml lineos %}
<!--?xml version="1.0" encoding="UTF-8"?-->

<configure id="webAppCtx" class="org.mortbay.jetty.webapp.WebAppContext">
	<new id="appManager" class="org.mortbay.jetty.plus.naming.Resource">
		<arg>
			<ref id="webAppCtx">
		</ref></arg>
		<arg>BeanManager</arg>
		<arg>
			<new class="javax.naming.Reference">
				<arg>javax.enterprise.inject.spi.BeanManager</arg>
				<arg>org.jboss.weld.resources.ManagerObjectFactory</arg>
				<arg>
			</arg></new>
		</arg>
	</new>
</configure>
{% endhighlight %}

O Jetty não tem suporte para construir o javax.naming.spi.ObjectFactory como o Tomcat, sendo assim é necessário criar manualmente uma referência para javax.naming.Reference onde iremos encapsular o ManagerObjectFactory.

Para o Tomcat, assim como Jetty, precisamos que declarar manualmente o bind do Manager, para isso vamos precisar criar o arquivo context.xml e nele vamos declarar um Resource do nosso BeanManager, conforme Listagem 6.

Listagem 6.Configurando o context.xml para Tomcat

{% highlight xml lineos %}
<!--?xml version="1.0" encoding="UTF-8"?-->
<context>
	<resource name="BeanManager" auth="Container" type="javax.enterprise.inject.spi.BeanManager" factory="org.jboss.weld.resources.ManagerObjectFactory">
</resource></context>
{% endhighlight %}

Para usuários do Tomcat ainda podemos fazer a injeção de dependências do Weld diretamente em Servlets, ou seja, nas servlets não precisamos declara-lá com um Bean. Para utilizar desta feature basta adicionarmos, novamente, em nosso context.xml o seguinte Listener, conforme Listagem 7, observação: somente o Tomcat tem essa Feature.

Listagem 7.Configurando o context.xml para Tomcat

{% highlight xml lineos %}
<listener classname="org.jboss.weld.environment.tomcat.WeldLifecycleListener"></listener>
{% endhighlight %}

### Configurando nosso web.xml ###
Para o correto funcionamento da nossa aplicação ainda temos algumas configurações para fazem  em nosso arquivo web.xml, tais como configurar o filtro do Jboss Weld, configurações básicas do JSF 2.0 e ainda declarar nosso BeanManager, tudo conforme Listagem 8.

Listagem 8.Configurando o web.xml

{% highlight xml lineos %}
<!--?xml version="1.0" encoding="UTF-8"?-->
<web-app version="2.5" xmlns="http://java.sun.com/xml/ns/javaee" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemalocation="

http://java.sun.com/xml/ns/javaee

	  http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd">
	<display-name>Weld Quick Start</display-name>
	<!-- Observação 1 -->
<context-param>
<param-name>javax.faces.DEFAULT_SUFFIX</param-name>
<param-value>.xhtml</param-value>
	</context-param>
	<!-- Observação 2 -->
<listener>
<listener-class>org.jboss.weld.environment.servlet.Listener</listener-class>
	</listener>
<!-- Observação 3 -->
	<servlet>
		<servlet-name>Faces Servlet</servlet-name>
		<servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
		<load-on-startup>1</load-on-startup>
	</servlet>
	<servlet-mapping>
		<servlet-name>Faces Servlet</servlet-name>
		<url-pattern>*.jsf</url-pattern>
	</servlet-mapping>
	<session-config>
		<session-timeout>10</session-timeout>
	</session-config>
	<!-- Observação 4 -->
<resource-env-ref>
		<description>Object factory para o CDI Bean Manager</description>
		<resource-env-ref-name>BeanManager</resource-env-ref-name>
		<resource-env-ref-type>javax.enterprise.inject.spi.BeanManager</resource-env-ref-type>
	</resource-env-ref>
</web-app>
{% endhighlight %}


Na primeira observação, da listagem 8, nos informamos ao JSF que nossos JSF Views(Facelets Templates) todos são teminados com a extensão .xhtml, já utilizavamos muito essa abordagem desde as versões anteriores de JSF. Na segunda observação, da listagem 8, estamos configurando um listener do Jboss Weld pois não estamos trabalhando em um conteiner Full JEE, conforme já explicado anteriormente. Já na terceira observação, da listagem 8, estamos configurando a Servlet do JSF 2.0 para que todos os Requests em nosssas URL sejam terminadas com .jsf, mas poderia ser qualquer extensão. E na quarta observação, da listagem 8, estamos terminando de fazer o bind entre o Manager e o JNDI, declaramos que vamos utilizar nosso BeanManager dentro do nosso projeto.

### Criando nosso primeiro Controller Bean ###
Seguido todos os passos acima nos temos um ambiente configurado, agora vamos ver como o Weld e o JSF 2.0, juntos, trabalham de maineira mais elegante e simples do que nas versões anteriores de JSF. Anteriormente tinhamos que sair declarando em nosso faces-config.xml cada Bean que gostariamos de acessar a partir do XHTML. Com o JSF 2.0 e o Weld isso muda, para quem já usava o Jboss Seam irá ver a primeira semelhança entre o Seam e o Weld. Sempre que precisarmos declarar um managed-bean do JSF vamos basta anotar a classe com @Named. Você ainda pode passar como parametro o nome deste managed-bean, caso contrario o nome dele será o mesmo nome da classe porém com a primeira letra em minusculo (por convenção). E é através desse nome que conseguiremos acessar nosso managed-bean em nossos XHTMLs. Entendido isso vamos criar nosso primeiro controller bean, onde vamos adicionar uma propriedade de String com um texto e exibir ele posteriormente no XHTML.

Listagem 9.Criando nosso primeiro Bean

{% highlight java lineos %}
package br.com.brenooliveira;

import java.io.Serializable;
import javax.enterprise.context.RequestScoped;
import javax.inject.Named;

@Named
@RequestScoped
public class HelloWeld implements Serializable {

	private static final long serialVersionUID = -862215173216584370L;

	private String text = "Hello Weld";

	public String getText() {
		return text;
	}

}
{% endhighlight %}

Na da Listagem 9, apenas criamos um controller bean, onde anotamos com @Named, isso significa que essa classe é um Bean onde podemos acessar seus métodos públicos a partir de nossos .xhtmls, esse acesso é  feito através de linguagem EL(Expression Language). O simples fato de transformar nossa classe em um Controller Bean não significa que podemos injetala em qualquer outra clase ou injetar outros Beans dentro dela. Por padrão o Weld vai registrar esse Bean com o nome da classe, mas a primeira letra minuscula, porém, também podemos alterar o nome de nosso Bean, apenas anotando o objeto @Named(“outroNome”) passando o novo valor do nome do Bean e o Weld vai disponibilizar esse Bean com outroNome(para acessar a partir do XHTML, por exemplo).

Ainda anotamos a classe com um escopo no caso @RequestScoped, assim como faziamos nas versões anteriores de JSF por xml, com isso informamos a aplicação que seu escopo é de Request, o que é muito comum nos aplicativos web, esse escopo começa quando é feita uma requisão do navegador ao aplicativo web e termina quando o servidor responde a requisição. Ainda existem outros tipos de escopos de Beans os já conhecidos como session (@SessionScope), application(@ApplicationScope) e o novo tipo de escopo de conversation(@ConversationScope) já conhecido pelos desenvolvedores que utilizam o Jboss Seam, esse escopo é maior que o request e menor que o session. Geralmente utilizado para criação de formularios para cadastros, onde dividimos em várias etapas o famoso Wizard.

### Criando nosso xhtml (Facelets Template) ###
Agora vamos criar nosso primeiro XHTML, onde vamos chamar os metodos do nosso Controller Bean helloWeld e exibir em nossa página web o valor da nossa String text. O JSF 2.0 usa por padrão o handler do facelets então podemos acessar nosso Controller Bean facilmente através de Expression Language, sem a necessidade de utilizar tags JSF como <h:outputText />, veja a Listagem 10.

Listagem 10.Criando nosso index.xhtml

{% highlight xml lineos %}
<!--?xml version="1.0" encoding="UTF-8"?-->
<h:head>

</h:head>
<h:body>
<h1>Funcionou?</h1>

Nosso weld-injected bean: #{helloWeld.text}

</h:body>
{% endhighlight %}

Da mesma forma como já era feito nas versões anteriores do JSF, vamos utilizar de Experssion Language(EL), #{nomeDoBean.NomeDaPropriedade}, observe que, na listagem 9, nos criamos os getters e os setters da propriedade, isso para que  o JSF consiga encontrar e editar o valor da propriedade quando necessário, assim como já era feito nas versões anteriores, ainda temos essa necessidade.

### Testando nosso projeto ###
Na listagem 3 no nosso pom.xml, você pode observar que adicionamos um plugin do Jetty, o jetty-maven-plugin, este plugin contribui desenvolver e testar nosso projeto sem a necessidade de reiniciar o nosso servidor a cada alteração em nossas classes. Nós podemos utilizar desse plugin do maven2 em qualquer projeto web que seja criado nos padrões do Maven2. O plugin, periodicamente,esse tempo fica configura dentro da tag scanIntervalSeconds, então o plugin procura em nosso projeto por classes que foram alteradas e já faz o redeploy automaticamente sem a nossa intervenção. Ele também permite rodar um servidor Jetty junto com o deploy de nosso projeto, para isso basta rodar o comando conforme Listagem 11, na mesma pasta onde se encontra o pom.xml.

Listagem 11.Rodando nosso aplicativo

{% highlight xml lineos %}
mvn war:inplace jetty:run
{% endhighlight %}

Feito isso podemos ver nosso primeiro resultado, basta acessar a partir de qualquer navegador a seguinte URL http://localhost:8080/weldtutorial/index.jsf, essa URL pode variar de acordo com as configurações de seu servidor. Caso não tenha tido nenhum problema na geração do pacote e no deploy da aplicação você deve visualizar uma imagem conforme Imagem 1.

Imagem 1.Nosso aplicativo exibindo um Hello Weld

![Nosso aplicativo exibindo um Hello Weld]({{ site.url }}/assets/img1.jpg)



## Entendo um pouco mais sobre o CDI e Jboss Weld ##
Agora vamos entrar um pouco mais afundo no Weld, sem dúvidas umas das features mais esperadas do CDI é a injeção de dependência, nele a injeção é feita através de typesafe(resolução segura de tipos), ou seja, ele nunca confia em identificadores baseados em strings para determinar como os objetos se relacionam. Em vez disso, CDI utiliza a informação de tipagem que já está disponível no modelo de objeto Java, que trabalha junto com um novo padrão de programação, chamada de anotações qualificadoras, para unir os Beans, suas dependências, seus interceptadores e decoradores e seus consumidores de eventos. Influenciado por inúmeros frameworks como Jboss Seam e Spring, o CDI tem suas próprias características: mais alto nível de typesafe que o Jboss Seam e menos centrada em XML que o Spring, embora nas versões mais recentes do Spring já conseguimos fazer as injeções através de anotações.

No início pode parece ser um pouco complexo como as injeções são obtidas com o CDI, pois ele usa o typesafe, mas após a compressão do processo tudo fica mais simples. O typesafe é realizado durante a inicialização do sistema, o que significa que o contêiner informará ao desenvolvedor imediatamente se alguma das dependências de um Bean não puder ser satisfeita.

O objetivo é permitir que múltiplos Beans implementem o mesmo tipo de Bean e também:

 - Permitir que o cliente escolha qual implementação ele necessite utilizando um qualificador;
 - Permitir ao desenvolvedor da aplicação escolher qual das implantações é adequada para uma determinada implantação, sem alterações para o cliente, ao ativar ou desativar um alternativo;
 - Permitir que os Beans sejam isolados dentro de módulos separados.
### Compreendendo um pouco mais sobre Qualificadores ###
Um qualificador nada mais que uma anotação para você resolver a implementação a ser injetada. Sempre que um Bean ou ponto de injeção não declara explicitamente um qualificador, o contêiner assume o qualificador @Default. Em algum momento, você precisará declarar um ponto de injeção sem especificar um qualificador. Existe um qualificador para isso também. Todos os Beans possuem o qualificador @Any. Portanto, ao especificar explicitamente @Any em um ponto de injeção, você suprime o qualificador padrão, sem restringir os Beans que são elegíveis para injeção.

### Criando nosso Jogo de Dados ###
Entendido como o CDI funciona vamos compreender como nosso joguinho de dados irá funcionar. Em um campo do tipo input text, vamos preencher com um dos valores dos valores das faces do dado, ou seja, de um a seis, esse será o nosso range de números válidos. Seguida vamos tentar jogar o nosso dado e ver se tivermos sorte ou não. Após isso iremos trocar a nossa implementação de dado, onde vamos passar a utilizar um dado com doze faces, ou seja, vamos demonstrar como o iremos definir qual implementação desejamos usar. Nosso maior foco será em compreender como fazer injeção de dependências com CDI toda a parte gráfica será deixada de lado, apenas uma telinha não muito elaborada.

### Criando nossos Qualificadores ###
A nossa primeira tarefa será de criar nossos Qualificadores, ou seja, vamos criar três anotações para definir qual é a implementação a ser injetada em nossos objetos. Onde uma das anotações será a MaxNumber, Listagem 12, essa anotação representará a nossa quantidade de faces do nosso dado, no caso esse número é 6, mas poderia ser um número qualquer. A nossa segunda anotação será a de valor mínimo a MinNumber, Listagem 13, pois queremos A nossa segunda anotação é a Random, Listagem 14, ela será irá indicar o número aleatório entre MinNumber e MaxNumber.

Listagem 12.Criando nosso Qualificador MaxNumber

{% highlight java lineos %}
package br.com.brenooliveira.qualifiers;

import static java.lang.annotation.ElementType.TYPE;
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.FIELD;

import java.lang.annotation.Retention;
import static java.lang.annotation.RetentionPolicy.RUNTIME;
import java.lang.annotation.Target;

import javax.inject.Qualifier;

@Qualifier
@Target({ TYPE, METHOD, PARAMETER, FIELD })
@Retention(RUNTIME)
public @interface MaxNumber {}
{% endhighlight %}


Listagem 13.Criando nosso Qualificador MinNumber

{% highlight java lineos %}
package br.com.brenooliveira.qualifiers;

import static java.lang.annotation.ElementType.FIELD;
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.TYPE;
import static java.lang.annotation.RetentionPolicy.RUNTIME;

import java.lang.annotation.Retention;
import java.lang.annotation.Target;

import javax.inject.Qualifier;

@Qualifier
@Target({ TYPE, METHOD, PARAMETER, FIELD })
@Retention(RUNTIME)
public @interface MinNumber {}
{% endhighlight %}

Listagem 14.Criando nosso Qualificador Random

{% highlight java lineos %}
package br.com.brenooliveira.qualifiers;

import static java.lang.annotation.ElementType.TYPE;
import static java.lang.annotation.ElementType.METHOD;
import static java.lang.annotation.ElementType.PARAMETER;
import static java.lang.annotation.ElementType.FIELD;

import java.lang.annotation.Retention;
import static java.lang.annotation.RetentionPolicy.RUNTIME;
import java.lang.annotation.Target;

import javax.inject.Qualifier;

@Qualifier
@Target({ TYPE, METHOD, PARAMETER, FIELD })
@Retention(RUNTIME)
public @interface Random {}
{% endhighlight %}

Nas Listagens 12, 13 e 14 foram criadas apenas três anotações do Java, até nesse momento não temos novidades, pois apenas utilizamos as anotações de @Target para TYPE, METHOD, PARAMETER e FIELD que são os locais onde podemos anotar com qualificadores. A única novidade é a anotação de @Qualifier, ela que vai indicar que essa anotação é um Custom Qualifier do CDI.

Seguindo com o nosso jogo, vamos desenvolver a classe onde serão produzidos os valores aleatórios e também vamos definir nossos MinNumber e MaxNumber, ou seja, essa é a classe de implementação para nossos Qualificadores observe a Listagem  15.

Listagem 15.Criando nosso Generator

{% highlight java lineos %}
package br.com.brenooliveira;

import java.io.Serializable;

import javax.enterprise.context.ApplicationScoped;
import javax.enterprise.inject.Produces;

import br.com.brenooliveira.qualifiers.MaxNumber;
import br.com.brenooliveira.qualifiers.MinNumber;
import br.com.brenooliveira.qualifiers.Random;

@ApplicationScoped
public class Generator implements Serializable {

	private static final long serialVersionUID = -9218778769573700274L;

	private java.util.Random random = new java.util.Random(System.currentTimeMillis());

	private int maxNumber = 6;

	private int minNumber = 1;

	java.util.Random getRandom() {
		return random;
	}

	@Produces
	@Random
	int generate() {
		return getRandom().nextInt(maxNumber);
	}

	@Produces
	@MaxNumber
	int getMaxNumber() {
		return maxNumber;
	}

	@Produces
	@MinNumber
	int getMinNumber() {
		return minNumber;
	}
}
{% endhighlight %}


Na Listagem 15, criamos um novo Bean, onde definimos que seu escopo será de aplicação, assim não será criado sempre uma nova instância de Random diferente cada vez que for chamado. Agora vem a mágica do CDI, observe que temos três métodos um chamado aleatorio(), getMinNumber() e outro chamado getMaxNumber(), cada um deles tem a sua anotação de seu propósito.

No método, aleatorio() da Listagem 15, observe que anotamos ele com @Random essa é o nosso qualificador, ou seja, ele demarca o propósito desse método. Já a anotação @Produces irá expor o valor na hora da injeção de dependência, ou seja, é ele que define o que deve ser injetado, no caso o retorno do método aleatório() é o que será injetado. O mesmo acontece para os métodos getMaxNumber() e getMinNumber(), no caso do getMaxNumber() anotamos ele com @MaxNumber e com @Produces, assim como fizemos com o método aleatório(). O mesmo fizemos com o método getMinNumber(), anotamos ele com @MinNumber e @Produces. Para definir a classe de implementação é sempre a combinação de uma anotação qualificadora e a anotação de produces.

O último Bean da aplicação é a classe DiceGame com escopo de sessão. Este é o principal ponto de entrada da nossa aplicação. Ele é responsável elas jogadas ou redefinir o jogo, podendo capturar e validar o palpite do usuário, verificando se é um número válido, ouse dentro dos valores de @MinNumber e @MaxNumber e fornecendo resposta ao usuário. Nós utilizamos o método de pós-construção (@PostConstruct) para inicializar o jogo recuperando um número aleatório a partir do Bean @Random.

Você notará que também adicionamos a anotação @Named nesta classe. Esta anotação somente é necessária quando você quer tornar o Bean acessível em uma página JSF por meio de EL (ou seja, #{game}), como fizemos em nosso primeiro exemplo. Veja a listagem 15.

Listagem 16.Criando nosso DiceGame

{% highlight java lineos %}
package br.com.brenooliveira;

import java.io.Serializable;

import javax.annotation.PostConstruct;
import javax.enterprise.context.SessionScoped;
import javax.enterprise.inject.Instance;
import javax.faces.application.FacesMessage;
import javax.faces.component.UIComponent;
import javax.faces.component.UIInput;
import javax.faces.context.FacesContext;
import javax.inject.Inject;
import javax.inject.Named;

import br.com.brenooliveira.qualifiers.MaxNumber;
import br.com.brenooliveira.qualifiers.MinNumber;
import br.com.brenooliveira.qualifiers.Random;

@Named("diceGame")
@SessionScoped
public class DiceGame implements Serializable {

	private static final long serialVersionUID = -8315665137856783662L;

	private int palpite = 1;

	private int numero;

	@Inject
	@MaxNumber
	private int maxNumber;

	@Inject
	@MinNumber
	private int minNumber;

	@Inject
	@Random
	Instance<integer> randomNumber;

	public void jogar() {
		if(numero == palpite) {
			FacesContext.getCurrentInstance().addMessage(null,
					new FacesMessage("Você ganhou nos dados!"));
		} else {
			FacesContext.getCurrentInstance().addMessage(null,
					new FacesMessage("Tente outra vez ..."));
		}
	}

	@PostConstruct
	public void reset() {
		this.palpite= 1;
		this.numero = randomNumber.get();
		System.out.println("Seu numero: " + numero);
	}

	public void validateNumberRange(FacesContext context, UIComponent toValidate, Object value) {
		int input = (Integer) value;

		if (input < minNumber || input > maxNumber) {
			((UIInput) toValidate).setValid(false);

			FacesMessage message = new FacesMessage("Número invalido");
			context.addMessage(toValidate.getClientId(context), message);
		}
	}

	public Instance<integer> getRandomNumber() {
		return randomNumber;
	}

	public void setRandomNumber(Instance<integer> randomNumber) {
		this.randomNumber = randomNumber;
	}

	public int getPalpite() {
		return palpite;
	}

	public void setPalpite(int palpite) {
		this.palpite = palpite;
	}

	public int getNumero() {
		return numero;
	}

	public void setNumero(int numero) {
		this.numero = numero;
	}
}
{% endhighlight %}


Observe na Listagem 16 que as variáveis minNumber, maxNumber e randomNumber, elas foram anotadas com @MinNumber, @MaxNumber e @Random respectivamente, além da anotação de qualificação ainda anotamos com @Inject,  isso faz com o CDI, naquele atributo, injete o valor que for produzido pela combinação da anotação de qualificação e @Produces, que se encontra na nossa classe Generator, da Listagem 15.

Os demais métodos fazem parte das nossas regras do jogo, onde reset() apaga os valores de todos os atributos já existentes, ele também está anotado com @PostConstruct para zerar as configurações do nosso jogo assim que o objeto for criado. O método jogar() verifica se o número que o usuário colocou no input é o mesmo do número sorteado pelo nosso @Random. E o nosso último método que temos é o validateNumberRange(), esse método é um validator do JSF, ele funciona da mesma maneira que nas versões anteriores do JSF, onde passamos o FacesContext, UIComponent e o Objeto valor que desejamos validar e então podemos fazer um vinculo entre o método e o input que vai receber o valor, dentro desse método validamos se é um valor válido para nossa regra de negócios.

### Criando nosso XHTML do jogo ###
Para concluir o nosso jogo ainda temos que criar um novo XHTML para fazer as interações do jogo, para isso vamos apenas criar um input que é onde vamos dar o palpite, apenas um número e que esteja no intervalo válido, um botão de enviar esse nosso palpite e outro botão para reiniciar a partida. Veja na Listagem 17.

Listagem 17.Criando nosso XHTML do jogo

{% highlight xml lineos %}
   <ui:composition template="/template.xhtml">
      <ui:define name="content">
<h1>Jogo de Dados</h1>

         <h:form id="\&quot;numberGuess\&quot;">
<div style="color: red">
               <h:messages id="messages" globalonly="false">
            </h:messages></div>
<div>
               Seu Palpite:
               <h:inputtext id="inputGuess" value="#{diceGame.palpite}" required="true" size="3" validator="#{diceGame.validateNumberRange}" disabled="#{diceGame.palpite eq diceGame.numero}">
               <h:commandbutton id="guessButton" value="Tentar" action="#{diceGame.jogar}" disabled="#{diceGame.palpite eq diceGame.numero}">
            </h:commandbutton></h:inputtext></div>
<div>
               <h:commandbutton id="restartButton" value="Reiniciar" action="#{diceGame.reset}" immediate="true">
            </h:commandbutton></div>

         </h:form>

Boa sorte

      </ui:define>
   </ui:composition>
{% endhighlight %}

Na listagem 17 temos o XHTML necessário para podemos interagir com nossos Beans já desenvolvidos anteriormente, no XHTML que vimos logo no inicio, não estávamos utilizando quase nada de Facelets. A partir desse segundo exemplo, você pode observar que estamos utilizando de Compositions do Facelets, uma vez que o handler dele é o padrão no JSF 2.0 já podemos utilizar dele para fazer template de nossa aplicação.

Ainda na Listagem 17, observe que adicionamos um <h:messages /> para exibirmos as mensagens vindas da nossa classe DiceGame, essas messagens serão exibidas através do nosso FacesMessages. Seguida adicionamos um <h:inputText /> esse é o campo de input responsável pela interação do usuário com o nosso jogo. Observe que nele adicionamos um validator fazendo um vinculo com o método validateNumberRange que declaramos na nossa Classe DiceGame. Também estamos desabilitando o campo quando o usuário acertar o número dos dados. Seguindo temos mais dois <h:commandButton /> o primeiro é apenas para chamar nosso método do Controller Bean para validar se acertamos ou não, já o segundo chama o método de reiniciar nosso jogo.

### Testando nosso Jogo ###
Basta executar o comando da Listagem 11, acessar a seguinte URL http://localhost:8080/weldtutorial/dados.jsf, caso esteja tudo certo você deve visualizar uma imagem conforme Imagem 2.

Imagem 2.Tela do nosso jogo

![Tela do nosso jogo]({{ site.url }}/assets/img2.jpg)


## Conclusão ##
Ao construir nosso aplicativo pudemos ver como se tornou simples a injeção de dependência do CDI com Jboss Weld, tudo configurado através de anotações, que podem ser anotações padrões (@Default ou @Any) ou ainda melhor podemos usar anotações customizadas para a nossa regra de negócio. E como o fato do CDI trabalhar fortemente com typesafe, não teremos surpresas em tempo de execução, pois o CDI valida se a injeção que irá acontecer é válida tudo isso é realizado durante a inicialização do sistema. Observamos como o Weld pode pode controlar facilmente o ciclo de vida de nossos Beans apenas anotando com o escopo que desejamos que ele tenha. Para desenvolvedores que já utilizavam o Jboss Seam sabem que o escopo de conversação é muito útil para Wizards e com o Jboss Weld já podemos utilizar deste escopo. A partir do lançamento do Jboss Weld temos mais uma opção para realizar nossa Inversão de Controle (Inversion of Control ou Ioc).
