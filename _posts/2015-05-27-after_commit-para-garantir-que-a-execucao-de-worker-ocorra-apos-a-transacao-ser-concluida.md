---
layout: post
title: Usando after_commit para garantir que a execução de um worker ocorra após a transação ser concluída
post_author: João Paulo Lethier
post_gravatar: 72ee02196394ca8c0b03f2ada05f5857
---

Temos uma funcionalidade que é o envio de uma ação por email, e quando essa ação está ativa, cada cliente novo que se cadastra no app deve receber esse email. Para isso, era utilizado um callback after_create, que chamava um worker para cuidar desse envio do email.

Olhando nosso rollbar hoje, percebi que em alguns momentos esse worker funcionava e enviava o email corretamente, e em outros momentos dava erro alegando que o cliente não foi encontrado no banco de dados. Percebi que isso acontecia pois algumas vezes o tempo de demora entre a chamada do worker no after_create e a execução do find no worker era menor que o tempo entre a chamada do worker no callback e o commit da transação, ou seja, o find era executado pelo worker antes da transação terminar.

Procurei uma alternativa para manter essa chamada no model, pois como a criação de um cliente pode acontecer a partir de mais de uma action dos controllers, preferi concentrar isso no model para facilitar a manutenção desse código. O activerecord possui um callback chamado after_commit, que só executa após toda a transação ser concluída e terminar com o commit, e é possível passar para o callback quando ele deve ser executado.

{% highlight ruby %}
class Cliente < ActiveRecord::Base
  after_commit :do_something, on: [:create]

  def do_something
    #codigo que sera executado apos a transacao ser concluida e fechada
  end
end
{% endhighlight %}

Com isso, foi possível garantir que sempre que o worker fosse executado, o objeto já estivesse salvo e disponível no banco de dados.

O problema que apareceu após isso era que o rspec não estava mais conseguindo verificar se a chamada para o código do after_commit estava sendo executada, pois o rspec não dispara um trigger para o after_commit.

Para resolver isso havia 3 alternativas:
- Incluir um código no setup do rspec(https://gist.github.com/cmaitchison/5168104)
- Usar a gem https://github.com/grosser/test_after_commit
- Executar o run_callbacks(:commit) após salvar o objeto no teste

Como eu vi no site da gem que no rails 5 não será mais necessário fazer nada disso pois o after_commit será executado no ambiente de testes por padrão, preferi a última alternativa, deixando o código comentado que após atualizarmos o projeto para o rails 5, a chamada run_callbacks pode ser removida.

{% highlight ruby %}
it 'calls a worker after creates a object' do
  Worker.should_receive(:perform_async)
  object.save
  object.run_callbacks(:commit)
end
{% endhighlight %}

Por hoje é só! :)