---
layout: post
title: Gerenciando o z-index dos elementos da interface com Sass
post_author: Thiago Felipe Victorino
post_gravatar: d61374b6c66fddb2c76c2512d43ed65e
---

Quem nunca setou `z-index: 9999;` no seu CSS que atire a primeira pedra!

Z-index é a propriedade do CSS que define a *stacking order* no eixo Z dos elementos que vão aparecer na interface. O elemento com o maior z-index sobrepõe os menores, e por aí vai.

Na falta de uma forma sã de organizar isso, não é raro setar essa propriedade de forma aleatória, na esperança que o z-index de determinado elemento "ganhe" dos outros. Por isso, é comum encontrarmos `z-index: 9999`, `z-index: 999999` por aí. Eu já fiz isso! Mas com a ajuda de um pré-processador, podemos fazer isso de forma mais organizada. :)

<!-- more -->

Começamos criando um arquivo separado chamado `_z-index.scss`. Nele, definimos uma lista do Sass chamada `$elements` (ou como você preferir) com o nome dos componentes que terão z-index:

{% highlight scss %}
$elements:
  navbar,
  popover,
  modal-backdrop,
  modal
;
{% endhighlight %}

Os itens dentro da lista recebem um número de acordo com a sua posição. Neste caso, `navbar` é igual a 1, `popover` é igual a 2, e assim por diante. Cuidado aqui para não se confundir: o elemento que receberá o maior z-index é o último da lista, então temos uma hierarquia de importância de cima para baixo.

Agora, precisamos buscar esse valor dentro do arquivo do componente. Para isso, utilizamos a função `index` do Sass. Dentro do arquivo `_navbar.scss`, fazemos o seguinte:

{% highlight scss %}
.navbar {
  z-index: index($elements, navbar);
}
{% endhighlight %}

Os parâmetros definidos são o **nome da lista** e o **nome do elemento** respectivamente. Fazemos isso com todos os componentes, nos seus respectivos arquivos.

{% highlight scss%}
.modal {
  z-index: index($elements, modal);
}

// ...e por aí vai
{% endhighlight %}

Assim, o *output* do CSS fica:

{% highlight scss%}
.navbar {
  z-index: 1;
}

.modal {
  z-index: 4;
}
{% endhighlight %}

E é isso! Mudou de ideia e quer que a navbar fique posicionada acima do modal? Basta mudar a ordem de importância na lista `$elements`. Criou um novo componente que deve sobrepor todos os outros? É só posicioná-lo na maior posição da lista, e todos os outros z-index se atualizarão quando o CSS for compilado.


Fonte: [http://www.smashingmagazine.com/2014/06/12/sassy-z-index-management-for-complex-layouts](http://www.smashingmagazine.com/2014/06/12/sassy-z-index-management-for-complex-layouts)
