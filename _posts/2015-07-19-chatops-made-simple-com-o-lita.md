---
layout: post
title: Chatops made simple com o Lita
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

## Um bot pra chamar de seu

No TDC de 2015, assisti a uma talk do [@brodock](http://github.com/brodock)
falando de como implementar [Chatops](http://blog.flowdock.com/2014/11/11/chatops-devops-with-hubot/)
de forma simlpes usando o projeto [Lita](https://www.lita.io/), desenvolvido com
o nosso tão amado Ruby.

Desde então fiquei com vontade de entender melhor como isso funcionava e como a
SumOne poderia se beneficiar com um bot simpático em nosso Slack. Descobri que
colocar o bot pra rodar não só é muito simples como também extremamente
divertido.

<!-- more -->

## Entendendo um pouco do Lita

O lita é um projeto _open source_ extremamente simples ao mesmo tempo em que é
extremamente poderoso. Baseado no Hubot, ele possui três tipos de adaptadores
que você pode plugar para personalizar o comportamento do bot:

* **Adapters** são adaptadores que fazem o bot conseguir interagir com
  diferentes plataformas de IM (Slack, HipChat, Campfire, ...). Já existem
  diversos _adapters_ disponíveis, é só pegar, plugar e aproveitar.
* **Handlers** são o que dá vida ao seu bot. Nos handlers você configura como
  ele deve reagir a mensagens postadas pelo seu time, como ele deve se comportar
  ao se conectar, enfim, todas as coisas que ele deve fazer.
* **Extensions** são plugins que permitem alterar o comportamento do _core_ do
  lita, necessário apenas se você quer fazer alguma mudança _hardcore_ no seu
  bot. A própria documentação do lita fala que dificilmente você precisará mexer
  em um deles.

### Instalando o lita

Instalar o Lita é ridiculamente fácil, é uma daquelas coisas que você fica se
perguntando: _"é só isso?"_. Basicamente você roda o comando
`lita new _diretório_` e ele cria um scaffold de um bot pra você, que é
basicamente um arquivo de configuração e um _Gemfile_.

Colocando novos _adatpers_ e _handlers_ no próprio _Gemfile_ já faz com que o
seu bot ganhe o comportamento oferecido por eles, precisando apenas de alguma
configuração que possa ser necessária. No meu _Gemfile_ inicial, coloquei as
seguintes _gems_:

{% highlight ruby %}
gem 'lita-slack'

# Useful
gem 'lita-answers'
gem 'lita-googl'
gem 'lita-google'
gem 'lita-google-images'
gem 'lita-hangout'
gem 'lita-wikipedia'

# BS
gem 'lita-xkcd'
gem 'lita-applause'
gem 'lita-ascii-art'
gem 'lita-chuck_norris'
gem 'lita-computer-dogs'
gem 'lita-greet'
{% endhighlight %}

Uma vez configurado o plugin `lita-slack` com as informações de conexão ao
Slack, o bot já estava pronto para o _prime time_. Foi só rodar o comando `lita`
e pronto! O bot entrou no slack e já estava pronto pra interagir com a equipe!

## Nasce o Peanut Butter
![Peanut Butter](http://vignette2.wikia.nocookie.net/bojackhorseman/images/d/d0/Mr_peanut_butter.JPG/revision/latest?cb=20140826220021)

Depois de configurar o lita com alguns plugins, demos vida ao nosos bot,
carinhosamente denominado "Peanut Butter", em homenagem a um personagem do
seriado da Netflix Bojack Horseman.

Fazer o _deploy_ para o Heroku foi tão simples quanto criar um _Procfile_:

{% highlight ruby %}
web: bundle exec lita
{% endhighlight %}

Colocar na configuração do bot as seguintes diretivas:

{% highlight ruby %}
config.http.port = ENV['PORT'] || 3131
config.redis.url = ENV['REDIS_URL']
{% endhighlight %}

E ativar o plugin _heroku-redis_. Depois disso foi só executar o _deploy_
normalmente, configurar o heroku para rodar no _dyno_ de US$7/mês (no _tier_
_free_ o bot iria ter que "dormir" durante algumas horas no dia).

## Polite Peanut
Queremos que o Peanut Butter traga sempre um comportamento alegre e virtuoso
para nossa equipe, não só faça o _grunt work_ por nós. Para isso, estamos aos
poucos criando _handlers_ que ajudem a dar esse comportamento a ele, como por
exemplo um que faz ele dar bom dia a todo mundo no canal **#general** as 8 da
manhã, reforçando um hábito que já temos:

{% highlight ruby %}
require 'active_support'
require 'active_support/core_ext'

# Handler que da bom dia todo dia as 08hs
class GoodMorningHandler < Lita::Handler
  MESSAGES = [
    'Bom dia galera!',
    'Goodie Goodie Morning pessoal!',
    'Que esse dia seja produtivo e delicioso! Bom dia amigos!',
    'Bom dia.. e alegria.. vamos sorrir e cantar! Dia galera!',
    'DIAAAAAAAAA',
    'Bão dia fi!',
    'Bom Bom Bom Bom diaaaaaaaaaa!',
    'Woof Woof! Bom dia!',
    'Good morning america!',
    'A quem chega, bom dia! Let\'s get to work!',
    'Let\'s do it to it pessoal! Bom Dia!',
    'Bundinha! Quer dizer.. Bom dia!',
    'Dia rapeizee!',
    'Dia dia dia diiiiaaa!',
    'Mais um dia começando, que esse seja ótimo galera!']

  on :connected, :check_if_its_hello_time

  route(/channel id/, :channel_id, command: true, help: 'Retorna o ID do canal')

  def initialize(*args)
    log.info 'Handler de bom dia inicializado!'
    @room = Lita::Source.new(room: 'C03HUG8SG') # General
    super
  end

  def channel_id(response)
    response.reply_with_mention response.message.source.room
  end

  def check_if_its_hello_time(*_)
    every(50) do |_|
      Time.zone = 'Brasilia' # Needed since this will run in another thread..

      # Todo dia de semana as 8 da manha...
      send_good_morning_message if Time.zone.now.strftime('%H:%M') == '08:00' &&
                                   ![0, 6].include?(Time.zone.now.wday)
    end
  end

  private

  def send_good_morning_message
    # Only one good morning per day :)
    return if redis.get 'gave_good_morning'

    robot.send_message(@room, MESSAGES.sample)

    redis.set 'gave_good_morning', true
    redis.expire 'gave_good_morning', 60
  end
end

Lita.register_handler(GoodMorningHandler)
{% endhighlight %}

E pronto! Aos poucos vamo continuar aprimorando o "PB" para fazer mais coisas e
reforçar nossos bons hábitos:

![Bom Dia ASCII](https://www.dropbox.com/s/jb69tykbovap2tp/Screenshot%202015-07-19%2010.04.49.png?raw=1)
