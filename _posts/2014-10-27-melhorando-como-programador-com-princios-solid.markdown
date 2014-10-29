---
layout: post
title:  "Melhorando como Pragramador com princípios SOLID - Parte II"
date:   2014-10-27 22:19:45
categories: engenharia de software
---

Bom hoje vamos continuar falando sobre princípios SOLID. No último post entendemos mais sobre
*[Single Responsability](http://brenooliveira.com.br/engenharia/de/software/2014/08/30/melhorando-como-programador-com-princios-solid/)*.
Agora entraremos em talvez um dos mais complicados ou aparentemente contraditórios para se entender Open/Close Principle.

#### Open/Close class ####
> "Software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification"
> [Bertrand Meyer](http://en.wikipedia.org/wiki/Bertrand_Meyer) em 1988  

#### Mas o que isso significa? ####
Hummm parece meio contraditório, mas é simples significa que você deve ser capaz de extender a classe, porém, sem modificar. Em linguagem como Java, C# ou outras com tipagem estática podemos facilmente resolver isso com hereança, com métodos abstratos, ou polimorfismo.

#### Como identificar onde utilizar? ####
Uma maneira mais fácil de identificar é você localizar pontos onde possam sofrer mudanças, vamos pensar naquela classe de relatório do post anterior. Lá temos uma classe chamada *OrderWeeklyReport* que é a reponsavél por gerar relatório de pedidos semanais.

{% highlight ruby lineos %}
class OrderWeeklyReport
  def self.name
    "Relatorio de pedidos"
  end

  def self.formatter
    CsvCompiler
  end

  def self.data
    User.order_weekly
  end

  def public_link
    ReportUpload
  end
end
{% endhighlight %}

Na nossa atual implementação estamos assumindo que o único formato de relatório é CSV, mas e se o cliente decidiu que quer também em formato em Excel?

O legal é temos várias maneiras de resolver isso, uma maneira se estivessemos programando em uma linguagem de tipagem estática poderiamos implementar *formatter* como abstract e teriamos várias classes do *OrderWeekReportCsv* ou *OrderWeekReportExcel*.

Mas uma solução mais elegante e simples seria a gente refatorar a classe *OrderWeekReport* no método *formatter* para receber qual a implementação utilizar.

{% highlight ruby lineos %}
class OrderWeeklyReport
  def self.name
    "Relatorio de pedidos"
  end

  def self.formatter(formatter: CsvCompiler)
    formatter
  end

  def self.data
    User.order_weekly
  end

  def public_link
    ReportUpload
  end
end
{% endhighlight %}

Pronto com isso a nossa implementação ficou mais flexível, pois estamos utilizando a *dependency injection*, agora para utilizar basta chamar o método *formatter* passando o tipo de relatório que você deseja.

{% highlight ruby lineos %}
OrderWeeklyReport.formatter ExcelCompiler
{% endhighlight %}


Assim concluimos mais uma parte de nosso SOLID. Na próxima vamos falar mais sobre o um princípio que mais parace nome de Vodka.
