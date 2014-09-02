---
layout: post
title:  "Melhorando como Pragramador com princípios SOLID"
date:   2014-08-30 22:19:45
categories: engenharia de software
---

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
{% highlight ruby %}
module Workers
  class ReportWorker

    def work message

      begin

        #configurações de ambiente
        config_file = ReportsApi.settings.reports_config_file
        config_aws = ReportsApi.settings.reports_config_aws
        config_mandrill = ReportsApi.settings.reports_config_mandrill

        #get report request
      	report = PendingReports.find(message.to_i)
        params = from_json report.params

        #in progress
        report.in_progress
        report.save

        #parse transaction
      	transactions = parse_query(params, report.account_id)
        pagination = pagination_params(params)

        #count and limit
        count = transactions.count
        sum   = (transactions.sum('total_value') * 100).to_i
        transactions = transactions.limit(pagination["limit"])
        transactions = transactions.offset(pagination["offset"])

        logger.debug("count: #{count} | sum: #{sum}")
        orders = OrderArraySerializer.new(transactions, { root: false }.merge(pagination).merge(count: count).merge(totalValueSum: sum))

        #parse para objeto compativel com metodo export
        orders_serialized = OrdersReportSerializer.new(orders)

        #create link
        file = Export.to(orders_serialized, report, config_file)

        #Uploading on Amazon S3
        UploadManager.instance.connect config_aws
        link = UploadManager.instance.upload file
        logger.debug "Generated link: #{link}"

        report.processed(link)
        report.save

        #Send email notification
        EmailNotification.send("extrato de pedidos", config_mandrill, report)

        #accept message
        ack!

      rescue => e
        logger.error e
        logger.error e.backtrace

        #report fail
        report.fail(e.message)
        report.save

        #reject message
        false

      end
    end
  end
end
{% endhighlight %}

Esse código pegamos de produção de um nossos serviços da empresa onde trabalho, ele ilustra tão bem que o programador teve de fazer vários comentários durante o código para que não ficasse perdido. Isso já pode ser um cheiro que seu código está fazendo mais do que deve.

Repare que esse código sabe desde as configurações de ambiente, informações sobre o relatório, busca, serialização, upload e envio de email.
