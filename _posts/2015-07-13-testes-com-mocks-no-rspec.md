---
layout: post
title: Testes com mocks no RSpec
post_author: Jaison Erick
post_gravatar: 9d7cd56baf4cd888f1fcb551132691ad
---

Um dos fundamentos de um bom projeto de software é uma suíte de testes bem
construída. Quando os testes são dificeis de manter eles acabam sendo deixados
de lado, desatualizados, ou pior, podem ser "corrigidos" apenas para passar o
build e testar a coisa errada dando uma falsa sensação de confiança.

<!-- more -->

## Testes de Integração versus Unitários

Um conceito interessante para garantir esta facilidade é a difereciação entre
testes de integração e unitários. Os testes de integração garantem que as
unidades da sua apliacação se comuniquem corretamente, mas por lidarem com
diferentes camadas, acabam sendo mais complexos de manter, visto que qualquer
alteração em uma camada de mais baixo nível pode quebrar uma quantidade
substancial de testes não relacionados.

Os testes unitários por sua vez testam o comportamento de uma unidade especifica
do projeto. A definição de uma unidade varia bastante de projeto para projeto. É
importante lembrar que uma unidade pode, mas não necessariamente é, um método de
uma classe. Uma forma interessante de saber se você está no caminho certo é
vendo o nome que você dá para os testes que escreve. Os testes devem descrever
qual o comportamento esperado da unidade e não sua implementação, então testes
com a descrição "should call method X" normalmente não estão testando muita
coisa.

Qual a distribuição correta entre os diferentes tipos de teste? Depende muito da
sua realidade, mas em termos gerais recomenda-se seguir o conceito da [pirâmide
de testes](http://blog.codeclimate.com/blog/2013/10/09/rails-testing-pyramid/).

## Mocks facilitam a manutenção de uma suíte de testes

Para facilitar a escrita de testes unitários existem os mocks. Mocks são objetos
falsos que substituem objetos reais, para facilitar a construção de testes mais
desacoplados e consequentemente fazer com que sua suíte dure por mais tempo.

No exemplo abaixo temos uma classe que recebe o ID de uma lista de emails, e é
responsável enviar os emails para toda a lista, utilizando um adaptador de
envio. Este adaptador pode ser facilmente substituido, portanto é facilmente
"mockável".

{% highlight ruby %}
describe "MailDispatcher" do
  let(:list) { create(:email_list) }

  it 'send all emails' do
    adapter = double('MailAdapter')
    expect(adapter)
      .to receive(:send).with(list.emails).and_return(list.emails.size)

    dispatcher = MailDispatcher.new(adapter)
    dispatcher.send(list.id)
  end
end
{% endhighlight %}

Embora esta solução seja aceitável, nem sempre é facil substituir o adaptador de
uma classe. Algumas vezes a classe foi criada para trabalhar em conjunto com uma
segunda classe específica, e nestes casos não existe a possibilidade de
substituir esta segunda classe por um mock no construtor.

## Partial Test Doubles

Por sorte, o RSpec nos ajuda nestes casos também, através dos [Partial test
doubles](http://www.relishapp.com/rspec/rspec-mocks/v/3-3/docs/basics/partial-test-doubles).
Este tipo de mock nos permite redefinir métodos de classe no namespace
global. Vamos usar o mesmo exemplo de antes, mas digamos que desta vez o
construtor da classe MailDispatcher não aceite mais o adaptador, e sim chame o
método de classe `MailAdapter#send`.

{% highlight ruby %}
describe MailDispatcher do
  let(:list) { build(:email_list) }

  it 'send all emails' do
    expect(MailAdapter)
      .to receive(:send).with(list.emails).and_return(list.emails.size)

    subject.send(list.id)
  end
end
{% endhighlight %}

Perceba que agora estamos usando a instrução `expect(MailAdapter)`, ou seja,
estamos substituindo o método `send` da classe `MailAdapter` por um mock.

## Mocks como auxiliares através de `allow`

Nos exemplos acima os mocks especificaram as condições para que um teste passe
ou não. Algumas vezes essa não é a intenção. No exemplo abaixo vamos dar a opção
para a classe chamar o método de um adaptador, porém caso este não seja chamado,
ainda assim o teste não será considerado como falho.

{% highlight ruby %}
describe MailDispatcher do
  let(:list) { build(:email_list) }

  it 'send all emails' do
    allow(MailAdapter).to receive(:send)
    expect(subject.send(list.id)).to be(list.length)
  end
end
{% endhighlight %}

Este teste tira o foco da interação entre os objetos, e coloca o foco no retorno
do método send da classe `MailDispatcher`.

## Utilize mocks de forma estratégica

A utilização de mocks sempre é um tradeoff. De um lado se ganha na facilidade
de testar um comportamento, por outro se perde na abrangência do teste.
Lembre-se que sempre que a assinatura de um método for alterada (o que não deve
acontecer com frequência), você deve atualizar todos os mocks que utilizam
aquele método, e este pode ser um trabalho árduo, dependendo de quanto aquele
método é utilizado.
