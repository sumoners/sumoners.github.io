---
layout: post
title: Trabalhando com json no Factory Girl
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

Com o advento das document-oriented databases e do crescimento do NoSQL, uma
atitude que se tornou bastante comum em nossos fluxos de desenvolvimento é
salvar objetos JSON dentro do nosso DB.

Na SumOne, usamos em alguns projetos a versão 9.4 do Postgres, que possui o
data type `jsonb`, que armazena dados em JSON de forma binária, permitindo
aliar flexibilidade com velocidade.

<!-- more -->

Usamos também a gem [FactoryGirl](https://github.com/thoughtbot/factory_girl)
para nos auxiliar no setup de models em nossos testes. Até então, vinhamos
passando um trabalhão para lidar com estruturas gravadas em JSON como por
exemplo:

{% highlight ruby %}
factory :ecommerce_feedback_new_interaction do
  metadata do
    {
      'nps' => 70,
      'grade' => 10,
      'purchase_id' => 287_937,
      'qualitative' => '',
      'affiliation' => {
        'id' => 1,
        'name' => 'Loja do Zeca'
      }
    }
  end
  action 'ecommerce:feedback:new'
end
{% endhighlight %}

Imagine que eu queira mexer somente no campo `affiliation -> id` para testar a
nossa capacidade de escopar os dados por esse campo. Até então, como estamos
validando a estrutura do nosso campo JSON, estávamos tendo que reescrever todo
o campo `metadata`, gerando muito ruído:

{% highlight ruby %}
feedback = create :ecommerce_feedback_new_interaction,
                  metadata: {
                    [ ... ]
                    'affiliation' => {
                      'id' => 2,
                      'name' => 'Loja do Juca'
                    }
                    [ ... ]
                  }
{% endhighlight %}

Para contornar esse problema, usamos os chamados [Transient Attributes](http://www.rubydoc.info/gems/factory_girl/file/GETTING_STARTED.md#Transient_Attributes)
do FactoryGirl para permitir dar um `merge` em metadados adicionais em nossos
testes:

*ecommerce_interaction_factory.rb*
{% highlight ruby %}
factory :ecommerce_feedback_new_interaction do
  transient do
    extra_metadata {}
  end

  metadata do
    {
      [... lots of stuff ...]
    }
  end
  action 'ecommerce:feedback:new'

  before(:create) do |interaction, evaluator|
    interaction.metadata = interaction.metadata
                           .deep_merge(evaluator.extra_metadata)
  end
end
{% endhighlight %}

Usamos o método [deep_merge](http://apidock.com/rails/Hash/deep_merge) do
*ActiveSupport* para garantir que um campo que esteja *nested* no outro
também sofra o *merge*.

Dessa forma, passamos a escrever nossas factories assim:

*feedback_spec.rb*
{% highlight ruby %}
feedback_do_juca = create :ecommerce_feedback_new_interaction,
                   extra_metadata: {
                     'affiliation' => {
                       'id' => 5
                       'name' => 'Loja do Juca'
                     }
                   }
{% endhighlight %}

Bem melhor, não? Com os campos JSON se tornando cada vez mais comuns, novas
necessidades para lidar melhor com eles em nosso _workflow_ vão surgindo.

Fica a dica pra quando você se deparar com esse problema!

Happy Hacking! :)
