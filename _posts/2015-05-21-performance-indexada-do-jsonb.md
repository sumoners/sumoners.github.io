---
layout: post
title: Performance indexada do campo jsonb no Postgres
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

Hoje, ao realizar algumas manutenções em uma tabela que contém centenas de
milhares registros que guardam suas informações em campos do tipo `jsonb`
percebi que qualquer query que eu fazia acabava demorando alguns segundos para
ser executado:

![Query demorado pra cacete](https://www.dropbox.com/s/wu07t9tgnzte3q9/Screenshot%202015-05-19%2015.06.26.png?raw=1)

Achei estranho pois tinhamos adicionado um index do tipo `gin` nesse campo e,
com apenas algumas centenas de milhares de registros, a performance já estava
bastante degradada.
Fui na [documentação do json datatype](http://www.postgresql.org/docs/9.4/static/datatype-json.html)
procurar uma resposta e encontrei a seguinte frase:

> The default GIN operator class for jsonb supports queries with the @>, ?, ?&
> and ?| operators.

Rapidinho corri pro console e executei a mesma query anterior, só que usando o
operador `@>` no lugar do operador `->>` e tive uma surpresa imensamente
agradável:

![2000x mais rápido!](https://www.dropbox.com/s/1b698wx7ojhc6ng/Screenshot%202015-05-19%2015.06.47.png?raw=1)

A query executou em ~5ms no lugar de ~7s, um ganho impressionante!

Explorando mais um pouco descobri que é fácil colocar tudo a perder também:

![Lento novamente](https://www.dropbox.com/s/opmtqg3kkvuqy6y/Screenshot%202015-05-19%2015.07.16.png?raw=1)

Se você coloca um limite de apenas 1 registro a query ganha um statement de
ordenação e de limite, o que parece colocar toda a performance do index a
perder, mesmo com o campo de ordenação indexado.

Então, fica a dica para quando você for fazer queries em seus campos `jsonb`:
Prefira os operadores `@>, ?, ?& e ?|` e ganhe performance, muita performance!

Happy Hacking!
