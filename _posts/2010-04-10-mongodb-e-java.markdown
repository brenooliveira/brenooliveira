---
layout: post
title:  "MongoDB e Java"
date:   2010-04-10 22:19:45
categories: mongodb nosql java
---

Para começar esse post sugiro que você, durante a leitura do post, esqueça um pouco tudo que você aprendeu sobre bancos de dados relacionais. Vamos falar um pouco sobre esse novo paradigma de bancos de dados, os bancos de dados não relacional ou NoSQL(Not Only SQL).  Com a notícia que anunciada pelo Twitter da migração do MySQL para o Cassandra esse assunto entrou muito em discussão.

Banco dados de alto desempenho, ou seja, você troca a integridade de dados pelo o alto desempenho. Foi o que o Twitter fez quando seu volume de tweets cresceu monstruosamente. Aqui nós vamos falar um pouco sobre o MongoDB, que é uma base de dados open source, escalavel e de alta performace todo escrito em C++.

O MongoDB possui uma boa API para Java junto com uma boa documentação, ele também tem APIs ou Drivers, como eles se referem no site deles, para C#, JavaScript, JVM Languages, Python, PHP, Ruby, C++ e Perl, nosso foco aqui será sobre a API Java.

Instalando o MongoDB
===
Para fazer a intalação dele é bem simples, faça download do MongoDB em http://www.mongodb.org/display/DOCS/Downloads escolha o versão de acordo com o seu Sistema Operacional, para o nosso caso vamos utilizar em uma máquina Linux 32 bits.

Descompacte os arquivos e dentro e inicie o processo mongod:

{% highlight shell %}
$ bin/mongod --dbpath /minha/pasta/meu/bd
{% endhighlight %}

Por padrão o MongoDB armazena em /data/db por isso passamos o parametro –dbpath, onde especificamos onde ficará armazenado nosso store. Se tudo deu certo vamos abrir o console do MongoDB:

{% highlight shell %}
$ bin/mongo

MongoDB shell version: 0.9.8
url: test
connecting to: test
type "help" for help
>
{% endhighlight %}

Nosso intuito é demonstrar a API Java do MongoDB, então não entrarei em muitos datalhes, do console do MongoDB.  O MongoDB utiliza um esquema de Collections, que você pode armazenar sua estrutura como a do JSON eles chamam de BSON. Podemos também utilizar da sintax do JavaScript para armazenar os dados no console do MongoDB.

{% highlight json %}
> j = { name: "BrenoOliveira", site : "www.brenooliveira.com.br"};
{"name" : "BrenoOliveira", site : "www.brenooliveira.com.br"}
> db.suaColecao.save(j);
> db.suaColecao.findOne();
{% endhighlight %}

Armazenamos na Collection suaColecao a estrutura JSON j = { name: “BrenoOliveira”, site : “www.brenooliveira.com.br”}; nosso findOne iráretornar o primeiro resultado da nossa Collection suaColecao.

Fazendo a integração com Java
====
O MongoDB disponibiliza sua API ou um Driver para Java, faça o download deste Driver em http://github.com/mongodb/mongo-java-driver/downloads e adicione esse JAR ao seu projeto.

Criando uma cenexão com o MongoDB
====
Com o serviço do mongod rodando e ter importado a API de Java para o MongoDB vamos para nossos códigos Java.

{% highlight java %}
Mongo m = new Mongo( "localhost" , 27017 );
DB db = m.getDB("test");
DBCollection coll = db.getCollection("suaColecao");
{% endhighlight %}

O que fizemos foi simples:

Criamos uma nova instância de Mongo, passando o endereço e a port do nosso serviço(neste caso como é tudo o padão do MongoDB poderiamos ter usado o construtor sem parametros Mongo());
Selecionamos nossa base de dados test;
Seguido disso selecionamos nossa Collection suaColecao.
Inserindo dados no MongoDB
A partir de agora podemos inserir/apagar/selecionar/atualizar registros na nossa Collection. Para representar nossos dados que queremos persistir no MongoDB vamos utilizar a classe BasicDBObject, para quem já utilizou a API do JSON para Java não irá sentir muita dificuldade, pois BasicDBObject é bem parecida com JsonObject.

{% highlight java %}
BasicDBObject doc = new BasicDBObject();
doc.put("nome", "Breno Oliveira");
doc.put("site", "www.brenooliveira.com.br");

BasicDBObject info = new BasicDBObject();
info.put("xpto", "zapata");

doc.put("info", info);
{% endhighlight %}

Acredito que o código seja bem explicativo e simples, ele tem uma seguinte repesentação em Json:

{% highlight json %}
{
    "nome" : "Breno Oliveira" ,
    "site" : "www.brenooliveira.com.br" ,
    "info" : {
        "xpto" : "zapata"
    }
}
{% endhighlight %}

O que fizemos acima foi bem simples:

Criamos uma instância de BasicDBObject;
Adicionamos informações de nome e site através do método put(chave, valor);
Criamos nova instância de BasicDBObejct e adicionamso novos atributos a ela;
Seguido adicionamos um BasicDBObject em outro BasicDBObject.
Agora vamos gravar esse nosso objeto no banco de dados:

{% highlight java %}
coll.insert(doc);
{% endhighlight %}

O que fizemos foi o seguinte com a nosso DBCollection, que selecionamos junto com a conexão com o MongoDB, chamamos o métodos insert() passando como parâmetro nosso BasicDBObject no caso a variável chamada doc.

Buscando nossos dados
====
Após ter inserido nossos dados vamos realizar pesquisas em nosso MongoDB.  Nossa busca será realizada pelo elemento interno do nosso JSon, no caso o nó info. Na busca temos que passar como parâmetro { “info” : { “xpto” : “zapata”}} e como você já pode imaginar vamos utilizar o BasicDBObject para representar essa estrutura de JSon, o código é o seguinte:

{% highlight java %}
BasicDBObject info = new BasicDBObject();
info.put("xpto", "zapata");
BasicDBObject search = new BasicDBObject();
search.put("info", info);
{% endhighlight %}

Criamos novamente 2 BasicDBObject e inserimos um dentro do outro, assim como fizemos para inserir os dados. Pois foi desta maneira que inserimos então desta maneira iremos buscar. Agora vamos criar o código Java para fazer a busca no MongoDB:

{% highlight java %}
DBCursor cur = coll.find(search);
System.out.println(cur.count());
while(cur.hasNext()) {
         System.out.println(cur.next());
}
{% endhighlight %}

Na nossa Collection, vamos fazer a busca chamamos o método find que nos retornará um DBCursor, depois através dele navegaremos dentre os resultados da nossa pesquisa. No caso chamamos o cur.count(); que nos devolve o total de resultados encontrados. Depois para o while apenas chamamos o hasNext() para vericar se tem um próximo resultado e então imprimimos o resultado na tela com next().

É provavel que você tenha como resposta em seu console o seguinte Json:

{% highlight json %}
{
    "_id" : "4bbf7e99e9c14911d1294f76" ,
    "name" : "Breno Oliveira" ,
    "site" : "www.brenoolievira.com.br",
    "info" : {
        "xpto" : "zapata"
    }
}
{% endhighlight %}

Você pode observar que o MongoDB adicionou um par chave/valor a mais, que é o _id que o primeiro elemento do nosso Json. Esse valor é um indice que o MongoDB usa, ele é unico para toda a collection.

Conclusão
====
O Hibernate não tem uma API para o MongoDB até mesmo porque o são conseitos diferentes, o Hibernate precisa da garantia da integridade de dados e isso pode não acontecer em um banco de dados NoSQL. Se você fizer testes de desempenho os bancos de dados NoSQL(MongoDB, Cassandra, HBase, BigTable) terá uma enorme vantagem sobre banco de dados relacional.
