---
layout: post
title:  "Melhorando como Pragramador com princípios SOLID"
date:   2014-08-30 22:19:45
categories: engenharia de software
---
## TL;DR ##
Defina uma única responsabilidade para as suas classes, isso vai tornar seu cógido mais legível e irá facilitar criação de testes unitários.
Refatore suas classes extraindo comportamentos para outras classes e compartilhe esses comportamentos através da composição.

### Melhorando como Pragramador com princípios SOLID ###
Vamos falar de SOLID? WTF? Vamos falar de estado solido da matéria? É sobre química ou desenvolvimento de software que vamos falar?
Na verdade SOLID é um termo de desenvolvimento de software elaborado por [Robert C. Martin (Uncle Bob)](http://en.wikipedia.org/wiki/Robert_Cecil_Martin) no começo dos anos 2000.

#### Mas afinal o que é SOLID? ####

|Letra  | Princípio             |
|-------|-----------------------|
|**S**  | Single responsibility |
|**O**  | Open/Close            |
|**L**  | Liskov substitution   |
|**I**  | Interface segregation |
|**D**  | Dependency inversion  |

Para não ficar muito pesado hoje vamos falar de sobre o Princípio de Responsabilidade Única.

#### Single responsibility ####
O princípio de responsabilidade única segundo Uncle Bob é que uma classe deve fazer apenas uma coisa e muito bem feita, ou seja, ela deve ter apenas uma responsabilidade. Isso também facilita a crição de seus testes, afinal o trabalho daquela classe é muito bem definido, ela tem um proposíto e realizar seus teste unitários ficam bem simples.

Agora vamos ilustrar melhor o cenário. Abaixo temos uma famosa classe sabe tudo.
{% highlight ruby lineos %}
class Relatorio

  def relatorio_pedidos(pending_report_id)
      #pega pedidos
    	lines = Pedidos.where("pedidos_da_semana >= ?", 1.week.ago)

      #Gera arquivos csv
      options[:col_sep] = ";"
		  file = CSV.generate(options) do |csv|
  			csv << "cabeçalho"
  			lines.each do |line|
  				csv << line
  			end
  		end

      #Uploading para Amazon S3
      s3 = AWS::S3.new(access_key_id: "ACCESS_KEY_ID",
				secret_access_key: "SECRET_ACCESS_KEY")
			bucket = s3.buckets.create("bucket_name")
      o.write(file: file[:full_name], content_type: file[:mime_type],
				expires: (DateTime.now() + 0.001).httpdate(), expiration_time: 30, single_request: true,
				acl: :public_read)

      #Gera mensagem
      link = o.public_url.to_s
      message = "Seu relatorio está disponivel em #{link}"

      #Envia e-mail
      Mail.deliver do
        from "bo@gmail.com"
        to "qq@example.com"
        subject "Seu relatório"
        body message
      end

    end
  end
end
{% endhighlight %}

Esse código pegamos de produção de um nossos serviços da empresa onde trabalho, foram omitidas algumas partes e exagerada em outras para ilustrarmos melhor, no caso o programador teve de fazer vários comentários durante o código para que não ficasse perdido. Isso já pode ser um cheiro que seu código está fazendo mais do que deve.

Repare que esse código sabe desde as configurações de ambiente, informações sobre o relatório, busca, upload e envio de email, ou seja, isso não é nada bom.

### O que podemos fazer para esse código ficar melhor ###
A primeira coisa que devemos fazer é começar a extrair determinados comportamentos para clases mais específicas. Exemplo devemos extrair a busca do relatório, envio para Amazon S3 e envio de e-mail, deixando nosso código mais limpo.
Alguns detalhes de implementação vou pular pois nosso objetivo é apenas mostrar como podemos deixar o código mais elegante.

#### Buscando registros ####
A primeira coisa que podemos fazer é separar a query do activerecord para um scope.
{% highlight ruby lineos %}
class User
  scope :order_weekly, -> { where("pedidos_da_semana >= ?", 1.week.ago) }
end
{% endhighlight %}

#### Gerar CSV ####
Um próximo paso bem interessante seria separar a classe responsável por gerar o CSV, podemos deixar da seguinte forma:
{% highlight ruby lineos %}
class CsvCompiler
  attr_accessor :data
  def initialize(data)
    self.data = Array(data)
  end

  def format
    options[:col_sep] = ";"
    file = CSV.generate(options) do |csv|
      csv << "cabeçalho"
      lines.each do |line|
        csv << line
      end
    end
  end
end
{% endhighlight %}

#### Upload para Amazon S3 ####
Outra parte que deve ser separada é o upload do relatório, apenas retornando sua URL pública.

{% highlight ruby lineos %}
class ReportUpload
  def upload(file)
    s3 = AWS::S3.new(access_key_id: "ACCESS_KEY_ID", secret_access_key: "SECRET_ACCESS_KEY")
    bucket = s3.buckets.create("bucket_name")

    o.write(file: file, content_type: "application/csv",
      expires: (DateTime.now() + 0.001).httpdate(), expiration_time: 30, single_request: true,
      acl: :public_read)

    o.public_url
  end
end
{% endhighlight %}


#### Envio de e-mail ####
Outra tarefa importante é o envio de e-mail

{% highlight ruby lineos %}
class ReportMailer
  attr_accessor :report, :recipient

  def intiialize(report:, recipient:)
    self.report = report
    self.recipient = recipient
  end

  def deliver!
    mail = Mail.new do
      from "teste@gmail.com"
      to recipient
      subject "Seu relátorio está pronto"
    end

    mail.deliver!
  end
end
{% endhighlight %}

#### Juntando as partes ####
Bom agora precisamos de um cara para poder chamar todas as partes necessárias para montar esse relátorio. No caso estamos fazendo um relátorio de pedidos da semana então vamos criar uma classe para isso.

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

Para utilizar seria apenas

{% highlight ruby lineos %}
ReportMailer.new(OrderWeeklyReport, "breno@example.com")
{% endhighlight %}

Como pode observar dividimos o código em diversas partes, pense em dividir para conquistar, agora cada parte tem sua responsabilidade bem definida facilitando a legibilidade do código e criação de testes.
