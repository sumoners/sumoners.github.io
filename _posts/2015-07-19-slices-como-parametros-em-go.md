---
layout: post
title: Slices como parâmetros em Go
post_author: Jaison Erick
post_gravatar: 9d7cd56baf4cd888f1fcb551132691ad
---

Go é uma linguagem moderna e atual que tem como intenção evoluir as aplicações
em performance, mas sem deixar de ser inteligível. É um C com esteróides, que
possui características de baixo nível, mas uma cara e um estilo superiores.

Mas não se deixe enganar, pois embora facilitado (e muito) lidar com ponteiros,
referências, etc. gerenciar memória está longe de ser uma tarefa simples. Não
precisamos mais desalocar memória, saber o que está usando que espaço ou coisas
parecidas. O risco de memory leak é quase nulo, mas ainda precisamos dominar
estes recursos para alcançarmos nossos objetivos.

<!-- read ore -->

## Um Slice é um container para um ponteiro

Um problema bastante comum e difícil de ser identificado quando implementado
erroneamente na primeira vez, é a passagem de slices como parâmetros de funções.
Como slices representam um range dentro do espaço de memória de um array,
qualquer modificação do slice acaba por modificar o array que o originou. Veja
no código abaixo (clique em *Run* para ver o resultado):

<iframe src="http://play.golang.org/p/ucRWTa4IKB" frameborder="0" style="width:
100%; height: 450px"><a href="http://play.golang.org/p/ucRWTa4IKB">see this code
in play.golang.org</a></iframe>

Perceba que a funcão `ChangeSlice` altera o slice, mas esta alteração não fica
contida em seu escopo. O motivo para isso é que os slice `key` na verdade
armazena um ponteiro para o array definido em `s` e, ao ser alterado, altera o
valor no endereço de memória deste array.

## Slices são passados como valor, não como referência.

Uma propriedade interessante é que ao passar o slice para a função ChangeSlice
ele é na verdade copiado para a funcão, ou seja, alterar o slice em si não afeta
o mesmo slice fora do contexto desta função. Veja no exemplo abaixo:

<iframe src="http://play.golang.org/p/dvGzqUwLK7" frameborder="0" style="width:
100%; height: 450px"><a href="http://play.golang.org/p/dvGzqUwLK7">see this code
in play.golang.org</a></iframe>

Ao adicionar o valor *20* ao slice `key`, este slice percebe que a memória
disponível para o seu array não será mais suficiente e procura alocar um novo
espaço para comportar este novo valor. Ao fazer isso, ele altera o seu conteúdo
para este novo local. Por `key` ter sido passado por valor (e não por
referência), essa alteração não pode ser percebida no valor de `s` na função
`main`. Por este motivo, sempre que alterarmos um slice em uma função, devemos
retorná-lo a função chamadora, para que esta consiga operar com os novos
valores.

## Para garantir a imutabilidade de um slice, aloque um novo espaço de memória

Existe ainda um terceiro caso onde uma função precisa receber um slice, porém
ela precisa que caso o valor seja alterado fora do seu contexto, que seu valor
permaneça constante. Isso seria o mesmo que passar uma array por valor. Para
alcançar o mesmo objetivo, veja o trecho abaixo, extraído do pacote `hmac` da
biblioteca standard do Go:

{% highlight go %}

// New returns a new HMAC hash using the given hash.Hash type and key.
func New(h func() hash.Hash, key []byte) hash.Hash {
  hm := new(hmac)
  hm.outer = h()
  hm.inner = h()
  hm.size = hm.inner.Size()
  hm.blocksize = hm.inner.BlockSize()
  hm.tmp = make([]byte, hm.blocksize+hm.size)
  if len(key) > hm.blocksize {
    // If key is too big, hash it.
    hm.outer.Write(key)
    key = hm.outer.Sum(nil)
  }
  hm.key = make([]byte, len(key))
  copy(hm.key, key)
  hm.Reset()
  return hm
}

{% endhighlight %}

O objetivo deste código é inicializar a estrutura para geração de um novo hash
**HMAC**. Você pode desconsiderar o conteúdo até a linha 15. A slice `key` contém
uma sequencia de **bytes** que serãm usados como chave para a validação do hash.
Perceba que como nos exemplos anteriores, este slice é passado como valor, porém
possui uma referência a um array definido no exterior desta função, que se
modificado, pode afetar o comportamento deste pacote, e portando não pode ser
confiável.

Para resolver este problema, foi alocado um novo slice com uma referência a um
novo endereço de memória, com espaço suficiente para armazenar todos os
elementos de `key` (veja em `len(key)`). Após a alocação, a função `copy` copiou
os valores de `key` para este novo espaço de memória. A partir de agora,
qualquer alteração feita externamente não terá qualquer efeito em `hm.key`.
