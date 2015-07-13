---
layout: post
title: Lidando com reflection em Go
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

## Metaprogramação em Go?

Durante meus estudos de final de semana, comecei a construir um _dashboard_
utilizando Go. Achei que era a oportunidade perfeita de aprender diversas
features da linguagem a partir das seguintes perguntas: Como construir um
servidor HTTP em Go (para exibir o dashboard no browser)? Como utilizar
_goroutines_ e _channels_ para fazer a comunicação entre as fontes de informação
e um _websocket_ que transporta as informações em _realtime_ para os clientes
conectados? Como implementar um _websocket_ que seja capaz de fazer _broadcast_
para múltiplas conexões? Como fazer a compilação de _templates HTML_ dos
_widgets_ no estilo "Go"? Enfim, dá pra entender aonde eu quero chegar.

<!-- more -->

Uma pergunta que eu não esperava ter tão cedo surgiu logo no inicio do projeto:
"Existe metaprogramação em Go?". Isso porque eu queria fazer um dashboard
configurável por um arquivo de configuração em JSON, que teria o seguinte
_schema_ (ainda um draft):

{% highlight javascript %}
{
  dashboards: [
    {
      id: 'sprint',
      title: 'Informações do Sprint Atual',
      widgets: [
        {
          id: 'closed-issues',
          type: 'Number'
        },
        {
          id: 'opened-issues',
          type: 'List'
        }
      ],
      data_sources: [
        {
          type: 'GitHubDataSource',
          config: {
            username: 'my_username@github.com',
            password: 'anicepassword'
          }
        }
      ]
    }
  ]
};
{% endhighlight %}

O problema surgiu nos `DataSources`: Eu queria que cada dashboard pudesse
configurar e re-utilizar uma fonte de dados específica para, por exemplo, eu
poder ter um dashboard com os dados de um projeto A no GitHub e outro dashboard
com os dados de um outro projeto B, tudo usando a mesma implementação do
`DataSource` mas com configurações diferentes.

## Achando o caminho

Carregar o arquivo de configuração do disco foi moleza, parsear o JSON e
colocá-lo dentro de um _struct_ foi "mel na chupeta". Agora, instanciar um
_pointer_ para o `struct` que cuidava da conexão com o GitHub... Foi um inferno!

Depois de dar uma olhada rápida no stack overflow, me deparei com um artigo que
apontava para a biblioteca _reflect_ da _standard library_ do Go. Olhando com
carinho, percebi que ela tinha dois tipos principais: `Type` e `Value`. Através
do `Type` conseguimos ter acesso ao tipo que estamos querendo lidar.

Pensei: Se eu tenho o tipo do _struct_ que eu quero lidar, instanciar esse mesmo
struct deve ser moleza. De fato não tem nada de complicado depois que você acha
o caminho das pedras, mas as pedras são inclinadas e cheia de perigos!

## A segunda barreira

Depois de entender um pouco como a library `reflect` funcionava, descobri que
não existe o conceito de um _type registry_ em Go: todos os tipos que você cria
não ficam guardados em um lugar acessível. Então é preciso criar o seu, o que
não é tão complexo assim:

{% highlight go %}
package main

import(
  "reflect"
  "fmt"
)

var(
  typeRegistry = make(map[string]reflect.Type)
)

type GitHubDataSource struct {
  Run func(chan string) chan bool
}

func init() {
  // Isso aqui roda sempre antes de tudo.
  typeRegistry["GitHubDataSource"] = reflect.TypeOf(GitHubDataSource{})
}

func main() {
  fmt.Println(typeRegistry["GitHubDataSource"]) // => main.GitHubDataSource
}
{% endhighlight %}

Implementado o _registry_, faltava apenas conseguir instanciar um struct a
partir de uma string contendo o seu tipo.

## Chegando na solução

Descobri que, tendo o `reflect.Type` do `struct` que eu queria instanciar, eu
poderia obter um novo _pointer_ para uma instância zerada daquele mesmo tipo
usando o método `reflect.New()`:

{% highlight go %}
func main() {
  data_source := reflect.New(typeRegistry["GitHubDataSource"])
  fmt.Printf("%#v", data_source) // => reflect.Value
}
{% endhighlight %}

Tendo esse pointer, eu consigo, usando o método `reflect.Value.Interface()` esse
tipo como uma `interface{}` que, usando type assertion, posso utilizar
como sendo uma interface implementada pelos meus data sources:

{% highlight go %}
package main

import(
  "reflect"
  "fmt"
)

var(
  typeRegistry = make(map[string]reflect.Type)
)

type DataSource interface {
  Run(chan string) chan bool
}

type GitHubDataSource struct {
  DataSource
}

func (g \*GitHubDataSource) Run(chan string) chan bool {
  fmt.Println("Running!")
  return make(chan bool)
}

func init() {
  typeRegistry["GitHubDataSource"] = reflect.TypeOf(GitHubDataSource{})
}

func main() {
  data_source := reflect.New(typeRegistry["GitHubDataSource"]).Interface().(DataSource)
  data_source.Run(make(chan string)) // => "Running!"
}
{% endhighlight %}

E pronto! Agora é só fazer com que cada novo `DataSource` se registre no
_registry_ através da função `init()` em seu próprio arquivo. Toda a lógica do
`DataSource` vai poder ficar encapsulada dentro dele mesmo e criar um novo vai
ser tão fácil quanto colocar um arquivo contendo um struct que obedeça a
_interface_ `DataSource` dentro de uma pasta do projeto.

A ideia é que cada `DataSource` receba um canal para postar suas notificações,
que posteriormente serão enviadas pelos _websockets_ para os clientes adequados,
e retorne um canal "quit", onde pode ser requisitado que o `DataSource` pare de
rodar.

Começar com uma nova linguagem onde sabemos muito pouco é um excelente e
divertido desafio, pois nos coloca em uma posição de aprendiz, onde tudo é novo
e as respostas nunca são triviais. É uma ótima maneira de melhorar suas _skills_
e se tornar um profissional mais capaz de entregar valor para diferentes tipos
de projeto. E você, já pensou em [aprender Go?](http://golang.org)
