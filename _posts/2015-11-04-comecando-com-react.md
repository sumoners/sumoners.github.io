---
layout: post
title: Olá, React!
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

Confesso que eu já estava um pouco anestesiado com as novas "balas de prata" do
mundo do ~~faroeste~~ Javascript quando ouvi falar do **React**. Olhei um pouco
do código e vi HTML misturado com JS, e isso foi o suficiente para eu jogar meu
interesse pelo projeto no lixo.

Depois de um ano, vendo as centenas de artigos postados sobre o projeto e
ouvindo discursos acalorados de _fanboys_ rasgando seda, resolvi dar mais uma
olhada para ver como a coisa funciona.

<!-- more -->

Comecei a construir um protótipo que exibiria as lojas em que nosso app Bonuz
estaria sendo usado no momento, em tempo real, para aprender a lidar com a
tecnologia.
Passado o choque inicial do _JSX_, que mistura HTML e Javascript no
mesmo arquivo, comecei a entender o conceito por detrás disso e a gostar muito
da experiência de desenvolver o aplicativo usando React.

O ponto principal para mim foi não ter que fazer uma porção de "códigos
macarrão" em jQuery para gerenciar mudanças na interface. Em React, tudo o que
você pretende construir é feito em forma de _componentes_, e esses componentes
se encaixam um dentro do outro, sendo que quando o estado de um é modificado,
todos os outros que possuem relação com ele também são atualizados.

## Uma passagem rápida pelo universo do React

Não faltam tutoriais para ensinar você a mexer bem no React, então não vou
_chover no molhado_ e dar uma passada básica nos conceitos mais interessantes e
no que eles podem te ajudar.

O primeiro conceito bacana é o de **Componentes**. Tudo em React é um
componente, isso ajuda você a pensar na sua aplicação de forma modular.
Por exemplo, um ranking com foto, nome e pontuação de clientes pode ser visto
como a junção de dois componentes:

![Ranking React](http://i.imgur.com/npPqOpZ.png)

Neste caso, usamos os componentes **RankingView** e **UserView** como
componentes.
Criar componentes é muito simples, basta usar o método de criação fornecido
pelo React e implementar um método _render()_ que retorne uma espécie de "HTML":

{% highlight javascript %}
// Cria o componente
var RankingView = React.createClass({
  render: function() {
    return(
      <div>
        <h1>Ranking</h1>
        <ul>
          <li>Usuário</li>
        </ul>
      </div>
    );
  }
});

// Adiciona o componente no DOM, dentro de um elemento com o ID "content"
ReactDOM.render(
  <RankingView />,
  document.getElementById('content')
);
{% endhighlight %}

Outro conceito bacana é a _composability_, a capacidade de compor interfaces
complexas a partir da junção de componentes. No mockup do ranking que eu mostrei
anteriormente, temos dois componentes básicos, sendo que um se repete 4x.
Para construir o mockup, você pode criar uma função dentro do componente
_RankingView_ que retorne componentes _UserView_. Sim, você vai ter que escrever
um "HTML no Javascript" pra fazer isso acontecer, mas não deixe isso te abalar:

{% highlight javascript %}
var UserView = React.createClass({
  // Renderiza uma _li_ com a foto do usuário, o nome dele e o número de pontos
  // que ele possui. Tudo o que está entre chaves {}, é código javascript normal
  // executado no contexto deste objeto.
  // Props são propriedades imutáveis que o "pai" deste componente passa para
  // ele no momento da sua criação.
  render: function() {
    return(
      <li className="user">
        <img src={this.props.picture} />
        <span className="name">{this.props.name}</span>
        <span className="points">{this.props.points}</span>
      </li>);
  }
});

// Vamos criar uma lista estática de usuários para o ranking. Poderíamos usar
// AJAX para torná-la dinâmica ou sockets para atualizá-la em tempo real. Enfim,
// dá pra inventar o que você quiser :)

var users = [
  { name: "Lord Voldemort", picture: "voldemort.jpg", points: 12 },
  { name: "Chuck Norris", picture: "chuckie.jpg", points: 8 },
  { name: "Angelina Jolie", picture: "angelina.jpg", points: 5 },
  { name: "Bruce Willis", picture: "baldman.jpg", points: 2 }
];

var RankingView = React.createClass({
  usersInTheRank: function() {
    return(users.map(function(u) {
      return(<UserView name={u.name} picture={u.picture} points={u.points} key={u.name} />);
    }));
  },

  // Uma coisa sensacional é que funções podem retornar outros componentes e,
  // já que são "javascript normal" (dilate bem sua concepção de "normal" aqui),
  // podem ser chamados como código entre chaves {}.
  render: function() {
    return(<div>
      <h1>Ranking</h1>
      <ul>
        {this.usersInTheRank()}
      </ul>
    </div>);
  }
});
{% endhighlight %}

Não sei se você sente a mesma coisa que eu ao olhar este código: a coisa bizarra
que é escrever XML(HTML) no meio do Javascript começa a fazer sentido!
Racionalizando, acho que é a sensação de ter "todo o código em um só lugar". É
muito trivial ver o que está acontecendo ao chamar _this.usersInTheRank()_ no
código de renderização já que se trata de uma função Javascript, e isso é muito
bacana!

Existem mais alguns conceitos bacanas como o de "estados", onde cada componente
sabe se precisa ou não se atualizar/montar no DOM de acordo com a mudança de seu
estado, bem como outros conceitos como o de DOM Virtual, que ajuda a ganhar
performance evitando escrever no DOM de verdade, mas esse artigo já está ficando
grande demais. Quer saber mais? [Acesse o site oficial](https://facebook.github.io/react/index.html)
do React num sábado a tarde e divirta-se :)
