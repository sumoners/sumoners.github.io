---
layout: post
title: Bookmarklets for fun and profit
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

Não tanto _profit_ quanto _fun_, não tão _fun_ quanto programar em Rails mas,
ainda assim, divertido o suficiente para postar no blog da SumOne.

Como quase toda boa ideia, tudo começa com um trabalho de corno que você não
aguenta mais fazer. Pra quem não sabe, trabalho de corno é um trabalho
repetitivo, sem novidades, caminhando no limite do tédio.

Meu trabalho de corno era o seguinte: Eu comecei a usar um aplicativo de
controle orçamentário pessoal (isso soa mais chato do que verdadeiramente é)
chamado apropriadamente de _You Need a Budget_.
O aplicativo é sensacional, fácil de usar e tem um conjunto de quatro regras que
não te deixam passar muito dos seus limites financeiros mas, para realmente
funcionar, precisa que você digite todas as suas despesas nele todos os dias,
nada pode passar!
Meu trabalho de corno era entrar no site do meu banco - Credifiesc (também
recomendo. Muito) - olhar as despesas que eu não tinha digitado ainda no app e
digitá-las uma a uma. Um saco!

Como não seria muito prudente criar um script que "entrasse no banco pra mim"
por razões óbvias de segurança, resolvi criar um _bookmarklet_ que exportasse
minhas transações para um formato que o _YNAB_ (nome abreviado do app de
controle orçamentário pessoal) conseguisse ler.

## Mãos a obra!

Criar um _bookmarklet_ é relativamente simples: você escreve javascript que vai
ser executado no contexto em que o navegador se encontra na hora em que o
_bookmark_ ("favoritos" no bom português tupiniquim) for clicado.
Pra explicar, vou começar simples, da mesma forma que eu comecei:

<a href="javascript:(function() { alert('oi! eu sou um bookmarklet feliz!'); })();">
Arraste isso pra sua barra de _favoritos_ e clique</a>

{% highlight html %}
<a href="javascript:(function() { alert('oi! eu sou um bookmarklet feliz!'); })();">
Arraste isso pra sua barra de _favoritos_ e clique</a>
{% endhighlight %}

Nada mais trivial que isso, certo? O bom e velho _alert_, amigo dos _debuggers_
da década passada, que ainda irão conhecer o javascript console.

A _priori_, é isso: javascript executado no browser. Daria pra terminar o post
por aqui se não fosse a maldita limitação de 512 bytes na URL que alguns
navegadores possuem, impedindo você de fazer algo com um nível moderado de
complexidade sem ter que apelar para _minificar_ ou [fazer algo que nem esses
atormentados fazem](http://js1k.com/).

Como sempre, existe um _workaround_ nessa limitação: usar _javascript_ para...
Injetar outro javascript de um servidor externo dentro da página! Não tão seguro
quanto útil, essa técnica foi o que me permitiu escrever um script grande o
suficiente para extrair as informações da página do banco. [Não sei por que mas,
juntando as palavras "banco" e a frase "não tão seguro quanto útil" me deu uma
coçeirinha.. Não deve ser nada!]

{% highlight javascript %}
javascript:(function(){
  sc=document.createElement('script');
  sc.type='text/javascript';
  sc.src='http://endereco.para.outro.javascript.beeem.maior';
  document.getElementsByTagName('head')[0].appendChild(sc);
})();
{% endhighlight %}

Pronto! Esse código vai carregar o código de outro _javascript_ na página atual
e executá-lo assim que carregar. Agora é só usar a imaginação e fazer qualquer
coisa na página, desde [inserir unicórnios na página](http://www.cornify.com/)
até coisas úteis como [exportar transações da credifiesc em CSV](https://github.com/sumoners/credifiesc-bookmarklet)
(sim, jabá.).

Para quem quiser ver como eu implementei o _javascript_ que puxa os dados da
credifiesc, o código está abaixo e no [github](https://github.com/sumoners/credifiesc-bookmarklet).

{% highlight javascript %}
// Credifiesc bookmarklet by Lucas Prim
// Use this at your own risk!

(function() {
  var EXPECT_TITLE = 'Movimentos não faturados';

  var titleElement = document.getElementById('titlePhase');

  if(!titleElement || titleElement.innerText != EXPECT_TITLE) {
    console.log("Bookmarklet won't load: Not on Credifiesc page");
    console.log("Current page title: ", titleElement.innerText);
    console.log("Expected page title: ", EXPECT_TITLE);
    return;
  }

  var tables = document.getElementsByClassName('dadosCartaoTitular');
  var confirmedTable = tables[1].getElementsByTagName('tbody')[0];
  var installmentTable = tables[2].getElementsByTagName('tbody')[0];

  var lines = [].slice.call(confirmedTable.getElementsByTagName('tr'));

  var csvContent = "data:text/csv;charset=utf-8,";
  csvContent += "Date,Payee,Category,Memo,Outflow,Inflow\n";

  lines.forEach(function(line, i) {
    var columns = line.getElementsByTagName('td');

    var date = columns[0].innerText;

    if(date === 'Total:') { return; } // Skip if its the total column

    var payee = columns[1].innerText;

    if(payee.match(/ - \d+\/\d+$/)) { return; } // Skip split pmts

    var memo = columns[2].innerText;
    var value = columns[4].innerText.replace('.', '').replace(',','.');

    csvContent += date + "," + payee + ",," + memo + ",";

    if(value.slice(0,1) == '-') {
      // Negative values are inflow
      csvContent += "," + value.slice(1) + "\n";
    } else {
      csvContent += value + ",\n";
    }
  });

  var iLines = [].slice.call(installmentTable.getElementsByTagName('tr'));

  iLines.forEach(function(line, i) {
    var columns = line.getElementsByTagName('td');

    var date = columns[0].innerText;
    var payee = columns[1].innerText;
    var memo = columns[2].innerText;
    var value = columns[3].innerText.replace('.', '').replace(',','.');

    csvContent += date + "," + payee + ",," + memo + "," + value + ",\n";
  });

  // Date,Payee,Category,Memo,Outflow,Inflow
  var encodedUri = encodeURI(csvContent);
  window.open(encodedUri);
})();
{% endhighlight %}
