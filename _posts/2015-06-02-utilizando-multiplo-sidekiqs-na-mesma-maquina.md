---
layout: post
title: Utilizando multiplos Sidekiq's na mesma máquina
post_author: Ricardo Pontes
post_gravatar: b6b7a6177f1058a9f1ab22e9f1b51f9e
---

Ao realizar o deploy de duas aplicações no mesmo servidor, sendo que em ambas estão utilizando o Sidekiq para executar os Background Jobs.

Nos deparamos com um problema ao executar um worker, pelo fato do Sidekiq utilizar o Redis, ocorria que um processo era executado pela outra aplicação.

Ao notar que estava ocorrendo o problema devido a base compartilhada pelos Sidekiq’s, fomos atrás de uma solução para o ocorrido.

A solução foi utilizar o mesmo Redis só que separando em Databases e namespaces diferentes.

<!-- more -->

Para isso, a implementação é simples, apenas crie um arquivo de configuração na inicialização de cada aplicação, onde serão apontadas para Databases diferentes:

**Servidor 1(Tomate):** config.redis = { namespace: 'Tomate', url: 'redis://127.0.0.1:6379/0' }

**Servidor 2(Abobrinha):** config.redis = { namespace: 'Abobrinha', url: 'redis://127.0.0.1:6379/1' }

{% highlight ruby %}
# config/initializer/sidekiq.rb

require 'sidekiq'

Sidekiq.configure_client do |config|
  config.redis = { namespace: 'Abobrinha',
                   url: 'redis://127.0.0.1:6379/1' }
end

Sidekiq.configure_server do |config|
  config.redis = { namespace: 'Abobrinha',
                   url: 'redis://127.0.0.1:6379/1' }
end
{% endhighlight %}


Fonte: [https://codedecoder.wordpress.com/2014/02/27/multiple-project-single-sidekiq-daemon-instance-one-machine/](https://codedecoder.wordpress.com/2014/02/27/multiple-project-single-sidekiq-daemon-instance-one-machine/)
