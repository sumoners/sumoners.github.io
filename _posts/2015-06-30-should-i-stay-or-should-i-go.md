---
layout: post
title: Should I Stay or Should i Go?
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

## Falling in love

Recentemente tenho lido alguns artigos a respeito de como a linguagem
[Go](http://www.golang.com) é excelente em termos de performance.

Alguns devs criticam a linguagem por sua extrema simplicidade enquanto outros a
adoram pelo mesmo motivo. _Flame wars_ a parte, resolvi experimentar para ter
minha própria opinião a respeito.

Construí dois projetos em Go, um deles [_open source_](http://www.github.com/lucasprim/s1-twitter-crawler-go),
para ter uma noção de como a linguagem funciona.

Vindo do Ruby, tudo pareceu no inicio bastante complexo: é sempre complicado
começar a programar em uma linguagem na qual você não domina os conceitos
básicos, não conhece as _libs_ mais populares nem as melhores práticas de
organização, design e arquitetura do código.

Com o tempo, entendendo os conceitos da linguagem melhor, fui ganhando
confiança e pude ver na prática o que é construír um aplicativo em uma
linguagem que tem um _compile time_ tão rápido que compilar o
código é quase tão rápido quanto digitar `ruby myapp.rb`.

Fora o compile time excepcional, pude, após programar em linguagens
interpretadas por anos, sentir novamente o "vento na cara" de usar uma linguagem
extremamente performática. E, acredite, o código em Go é rápido, muito
rápido, mais do que eu imaginava.

Empolgado com a rapidez da linguagem, pensei: _"quão rápido é esse troço?"_,
e nada melhor do que comparar com algo que você já conhece para ter um
parâmetro: Mãos a obra para construir um servidorzinho web básico em Ruby e
em Go para ver como eles performam:

## Quão rápido é o troço?

**Servidor em Ruby:**

{% highlight ruby %}
# config.ru
run Proc.new { |env| ['200', {'Content-Type' => 'text/html'}, ['Hello World!']] }
{% endhighlight %}

**Servidor em Go:**

{% highlight go %}
package main

import(
  "net/http"
  "fmt"
)

func main() {
  http.HandleFunc("/", func(w http.ResponseWriter, req *http.Request) {
    fmt.Fprintf(w, "Hello World!")
  })

	http.ListenAndServe(":8081", nil)
}
{% endhighlight %}

## Resultados

Usando unicorn para rodar a aplicação em ruby e compilando a aplicação em Go,
os resultados foram os seguintes:

`ab -n 1000 -c 100`

| N | C | Servidor | Req/s | Time per request | Time taken |
| - | - | ----| ------- | ------- | --------- | ------- |
| 1000  | 10  | Go      | 8073.18 | 1.239ms   | 0.124s  |
| 1000  | 10  | Unicorn | 969.92  | 10.310ms  | 1.031s  |

| 5000  | 100 | Go      | 8262.54 | 12.103ms  | 0.605s  |
| 5000  | 100 | Unicorn | 1004.96 | 99.506ms  | 4.975s  |

| 10000 | 100 | Go      | 8934.00 | 11.193ms  | 1.119s  |
| 10000 | 100 | Unicorn | 1023.64 | 97.690ms  | 9.769s  |

| 15000 | 100 | Go      | 9244.37 | 10.817ms  | 1.623s  |
| 15000 | 100 | Unicorn | 982.48  | 101.783ms | 15.267s |

Percebe-se que a performance do Go é constantemente melhor, em média sendo
**10x** mais rápido.
Lógico que esse benchmark é extremamente simples e não poderia ser
usado em uma comparação mais séria entre, mas uma coisa é fato:
Go performa muito melhor do que Ruby + Unicorn.

## Conclusão

Não acho que Go seja uma bala de prata, assim como nenhuma linguagem é. As
críticas de que programar em Go é complicado fazem sentido se você olhar do ponto
de vista de quem vem de um paradigma de orientação forte a objetos, tipagem
dinâmica e de uso de muita _dark magic_, mas não fazem sentido
se você enxerga que o objetivo da linguagem é justamente ser simples e permitir
que programadores dos mais diversos níveis possam participar de um mesmo projeto
sem comprometer seu design e sua estrutura.

Particularmente, vejo que Rails oferece muito mais **flexibilidade**, enquanto
Go oferece mais **performance**. Sendo assim, enxergo um futuro brilhante para
apps em Go nas nossas APIs, que tomam muita porrada, e em crawlers e trackers,
que precisam de centenas de conexões simultâneas.
Em nossa _customer-facing application_, onde o mais importante é ter capacidade
de se adaptar para satisfazer nossos clientes acima de tudo, vamos permanecer
com o bom e velho Rails.
