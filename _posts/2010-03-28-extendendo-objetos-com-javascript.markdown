---
layout: post
title:  "Extendendo o Objetos com Javascript"
date:   2010-03-28 22:19:45
categories: javascript
---

Embora JavaScript não tenha recursos como Java, PHP entre outras linguagens, é possivel extender classes com JavaScript através do prototype, com ele podemos extender qualquer classe JavaScript inclusive objetos já existentes dele. Nosso objetivo aqui é demonstrar como extender objetos com JavaScript.

No nosso primeiro caso vamos criar um replaceAll, note que o Object String já possui um método replace, mas ele apenas remove o primeiro elemento que ele encontrar. O nosso replaceAll irá remover todos os elementos que queremos.

{% highlight javascript %}
var texto = "http://www.brenooliveira.com.br/";
texto.replaceAll("/","");
{% endhighlight %}

O exemplo acima é maneira de como queremos que nosso novo método se comporte, em uma String ele faça todas as substituições para isso vamos utilizar o nosso prototype. Podemos falar que essa é uma maneira mais elegante de se adicionar métodos referentes ao objeto String. Também podemos simplemente criar um método que iria receber os três parametros, mas isso seria menos légivel, porque a substituição seria feita em uma outra String que seria retornada e seria necessário atribuir esse novo valor a nossa a String.  Para usar o Prototype:

{% highlight javascript %}
String.prototype.replaceAll() = function () { }
{% endhighlight %}

Feito isso declaramos qualquer String possa chamar o nosso novo método replaceAll. Agora o que falta é adicionarmos o corpo do novo método, o mais importante que precisamos saber agora é como acessar o valor da String, para isso basta usarmos a palavra reservada this no nosso caso ireia devolver o conteúdo de nossa String. Para fazer um teste vamos adicionar apenas um alert:

{% highlight javascript %}
String.prototype.replaceAll() = function () {
   alert(this);
};
{% endhighlight %}

Esse código acima irá retornar o conteúdo da nossa String. A Implementção completa do replaceAll poderia ser:

{% highlight javascript %}
String.prototype.replaceAll() = function () {
    var str = this;
    var pos = str.indexOf(de);
    while (pos > -1){
		str = str.replace(de, para);
		pos = str.indexOf(de);
	}
    return (str);
};
{% endhighlight %}

Resumindo podemos extender qualquer classe com JavaScript, nos próximos Posts colocarei mais exemplos do uso de prototype.
