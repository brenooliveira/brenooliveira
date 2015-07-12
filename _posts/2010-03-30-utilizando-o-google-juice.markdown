---
layout: post
title:  "Utilizando o Google Guice"
date:   2010-03-30 22:19:45
categories: java google juice
---

O Google Guice é um OpenSource framework que fornece Injeção de Dependência(DI), atráves de anotações. Há outros frameworks que fazem isso também, os mais conhecidos são PicoConteiner e o Spring. A principal diferença entre eles e o Google Guice é que o PicoConteiner faz a injeção de dependências programaticamente, o Spring atráves de XML e o Google Guice pode ser feito através de anotações. O Google Guice também influênciou bastante a JSR-330, se não me engano até o pessoal de lá participou do desenvolvimento da JSR, Google Guice já implementa essa JSR conforme no site deles http://code.google.com/p/google-guice/wiki/JSR330.

O que é a Injeção de Dependência?
A injeção de dependência é um design pattern que visa desacoplar os componentes da aplicação. Os componentes são instanciados externamente a classe. Um gerenciador controla essas instancias. Esse gerenciador, através de uma configuração, liga os componentes de forma a montar a aplicação, ou seja, utilizamos a Injeção de Dependência quando é necessário manter baixo o nível de acoplamento entre diferentes módulos de um sistema.

Vejo também que a Injeção de Dependência falicita o desenvolvimento voltado a Interfaces, ou seja, podemos trocar a implementação sem nos preocupar com as classes que já utilizavam desta interface e com a injeção de dependência podemos trocar as implementações apenas em um único local e nosso aplicativo continuará funcionando.

Um caso de uso
Um caso de uso, por exemplo, onde Injeção de Dempendência manteria um baixo acoplamento em nosso software é:

Você tem uma página na Web que envia SMS, esse envio é feito por uma outra empresa, e você chama os serviços SOAP está empresa. Na hora de renovar o seu contrato com essa empresa ela aumentou o preço em 100%. Mas felizmente existem outras empresas que fazem o mesmo serviço, porém cada uma fornece o serviço a sua maneira e a que você fechou o contrato com uma empresa que utiliza Rest Services. E sua página Web está com aquele código que chamava o SOAP está espalhado.

Se no desenvolvimento dessa página Web você tivesse utilizado a Injeção de Dependências a troca da API de envio de SMS seria o mais simples e o menos intrusivo em seu código.

A solução com Injeção de Dependência (Google Guice)
Feitas as apresentações sobre o Google Guice e como ele será util para nosso aplicativo vamos ao que interessa.

Vamos partir do principio de que você já criou uma estrutura java, agora precisamos adicionar as dependências do Google Guice, você pode fazer o download desses arquivos http://google-guice.googlecode.com/files/guice-2.0.zip e adicionar os JARs em seu projeto, ou se você utilizar o maven vc precisará dessas dependências:

{% highlight xml %}
<dependency>
   <groupId>com.google.code.guice</groupId>
   <artifactId>guice</artifactId>
   <version>2.0</version>
</dependency>

<dependency>
   <groupId>aopalliance</groupId>
   <artifactId>aopalliance</artifactId>
   <version>1.0</version>
</dependency>
{% endhighlight %}

Primeiro vamos criar nossa Interface SmsService que é onde vamos fazer a injeção de dependência.

{% highlight xml %}
public interface SmsServices {
   String send();
}
{% endhighlight %}

Agora vamos criar duas implementações, uma para a empresa ABC e outra para XPTO.

{% highlight java %}
public class SmsServicesAbcImpl implements SmsServices {
    @Override
    public String send() {
        return "SMS enviado pela empresa ABC";
    }
}
public class SmsServicesXPTOImpl implements SmsServices {
    @Override
    public String send() {
        return "SMS enviado pela empresa XPTO";
    }
}
{% endhighlight %}

Agora vamos criar uma classe que extends a AbstractModule do Google Guice, nela iremos definir as implementações para que vão ser injetadas na interface.

{% highlight java %}
public class GuiceModule extends AbstractModule {
    @Override
    protected void configure() {
        bind(SmsServices.class).to(SmsServicesAbcImpl.class);
    }
}
{% endhighlight %}

O que fizemos foi declarar que na interface SmsServices sempre será injetado SmsServicesAbcImpl, agora vamos criar um programa main para testar.

{% highlight java %}
public class MainProgram {
    public static void main(final String[] args) {
        Injector injector = Guice.createInjector(new GuiceModule());
        SmsServices services = injector.getInstance(SmsServices.class);
        System.out.println(services.send());
    }
}
{% endhighlight %}

O que fizemos foi pegar uma instância do Injector passar a classe, onde está configurado os binds entre interface e implementação. Então através do injector.getInstance(SmsServices.class) o Google Guice irá devolver a instância que definimos no GuiceModule.

Se tudo deu certo seu resultado no console será isso:

SMS enviado pela empresa ABC
Agora precisar trocar a implementação para a da empresa XPTO basta alterar o arquivo GuiceModule e adicionar a implementação de XPTO. E pronto nosso código ficou com o acoplamento bem mais baixo.

Concluimos que nosso código ficou bem mais limpo e de fácil manipulação. E se aparecer mais e mais implementações pode trocar facilmente sem afetar nosso código.
