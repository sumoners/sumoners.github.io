---
layout: post
title: Dica rápida, Usando union no activerecord
post_author: João Paulo Lethier
post_gravatar: 72ee02196394ca8c0b03f2ada05f5857
---

Me deparei com uma situação hoje onde precisava trazer para o rails uma query que a gente até então fazia direto no banco de dados. Essa query era feita para pegar todos os clientes que tiveram alguma interação. Por exemplo, vamos imaginar que temos um sistema com a classe Pessoa, que pode ter cadastrado filmes, desenhos e séries no sistema, e Desenho, Filme e Série são classes e tabelas diferentes.

{% highlight ruby %}
class Pessoa < ActiveRecord::Base
  has_many :filmes
  has_many :desenhos
  has_many :series
end

class Desenho < ActiveRecord::Base
  belongs_to :pessoa
end

class Filme < ActiveRecord::Base
  belongs_to :pessoa
end

class Serie < ActiveRecord::Base
  belongs_to :pessoa
end
{% endhighlight %}

<!-- more -->

No meu caso eu precisava pegar todas as pessoas que tivessem cadastrado alguma coisa no sistema, independente se foi um desenho, se foi uma série ou se foi um filme, ou seja, precisava pegar todas as Pessoas que tivessem interagido com o sistema realizando algum cadastro. Montando o sql, nós fazíamos algo assim:

{% highlight sql %}
  select * from (
    select distinct pessoas.* from pessoas
    inner join desenhos on desenhos.pessoa_id = pessoas.id

    union

    select distinct pessoas.* from pessoas
    inner join filmes on filmes.pessoa_id = pessoas.id

    union

    select distinct pessoas.* from pessoas
    inner join series on series.pessoa_id = pessoas.id
  )
{% endhighlight %}

No Rails, atualmente, não é possível usar o union diretamente no ActiveRecord, mas felizmente achei uma gem que resolve o problema: https://github.com/brianhempel/active_record_union. Instalando essa gem, se tornou possível fazer algo assim:

{% highlight ruby %}
class Pessoa < ActiveRecord::Base
  has_many :filmes
  has_many :desenhos
  has_many :series

  scope :cadastrou_filmes,   -> { joins(:filmes) }
  scope :cadastrou_desenhos, -> { joins(:desenhos) }
  scope :cadastrou_series,   -> { joins(:series) }

  scope :cadastrou_algo,     -> { cadastrou_filmes.union(cadastrou_desenhos).union(cadastrou_series) }
end
{% endhighlight %}

Usando o union a gem já garante que será retornado a relação de pessoas sem duplicar nenhum objeto, mas se quiser que retorne objetos duplicados, a gem também tem o método union_all.

Também seria possível usar a gem Arel para fazer o union, mas usando o union do arel, o retorno da query é um Arel::Nodes::Union, e não um ActiveRecord::Relation, o que faz ser necessário ser executado mais duas coisas para conseguir ter o retorno esperado e sem duplicações, o que tornaria o código bem complexo e me fez decidir não usar o Arel.

Por hoje é só! :)
