---
layout: post
title: Tratando milhões de registros JSON sem explodir a memória em Ruby
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

No Youbiquity trabalhamos com alguns TB de dados que precisam ser importados
dos bancos de dados de nossos clientes para o nosso banco de dados.
Isso representa um desafio gigantesco, afinal, como trafegar TB de dados de
forma eficiente, sem atolar a nossa API e sem acabar com a experiência dos
nossos clientes.

A solução veio de uma forma muito simples: Transportar os dados em arquivos
JSON, que podem usar a [FedEx net](https://what-if.xkcd.com/31/), muito mais
rápida do que a Internet :) e ser inseridos paralelamente por workers escritos
em ruby em uma instância do Postgres levantada especialmente pra tomar porrada.

Eis que colocamos o sistema para rodar e logo veio uma surpresinha:

![Memória indo pro saco](https://www.dropbox.com/s/ms95dbyzco03jkq/Screenshot%202015-05-13%2017.07.45.png?raw=1)
_a memória indo pro saco_

O processo Ruby começou a ler os arquivos `.json` contendo em média 50.000
registros e começou a deixar um rastro gigante na memória, e como a Amazon não
está barata com o dólar a 3 reais, isso iria ser um problema na hora da
migração.

## Yajl for the win!

Eis que encontrei a gem `yajl-ruby` que faz bindings pra biblioteca homônima em
C, capaz de fazer _streaming_ de arquivos json. Daí foi só alegria, como os
arquivos já tinham os registros separados um por linha, a transição do bom e
velho `JSON.parse()` pro novo código:

{% highlight ruby %}
file = File.new(file, 'r')
parser = Yajl::Parser.new

parser.parse(file) do |object|
  object # => { 'oi' => 'eu sou uma hash!' }
end
{% endhighlight %}

Bastou colocar pra rodar pra ver a diferença que fez, o processo não passou de
110MB para a importação de cerca de 300.000 registros JSON!

Então agora você já sabe, comeu memória parseando JSON? Yajl for the win!

Happy Hacking!
