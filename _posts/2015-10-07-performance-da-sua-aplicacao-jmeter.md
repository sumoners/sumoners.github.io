---
layout: post
title: Medindo a performance da sua aplicação com o JMeter
post_author: Jaison Erick
post_gravatar: 9d7cd56baf4cd888f1fcb551132691ad
---

Para garantir a qualidade de uma aplicação, precisamos entender como ela
funciona em um cenário real, antes que este cenário aconteça. Utilizando o
[**JMeter**](http://jmeter.apache.org/), podemos configurar um teste de
performance simulando cenários reais da nossa aplicação. Realizar benchmarks com
o **apache benchmark** é algo muito simples e embora sejam muito úteis para ter
uma noção de como a aplicação performa, não são suficientes.

<!-- more -->

## Defina os objetivos do seu teste

O primeiro passo para avaliar uma aplicação em termos de capacidade, é definir
os objetivos desta avaliação:

- Testar a maior carga suportada pela aplicação (teste destrutivo).
- Testar o comportamento da aplicação de acordo com um cenário real.

Ambos são importantes em diferentes situações. O primeiro é essencial para
validar se os dispositivos de segurança contra sobre sobrecarga foram
configurados corretamente, verificar se você está recebendo sinais de alerta da
sua aplicação nestes casos e validar o plano de ação para estas situações. O
retorno sobre o investimento em um teste para este caso ocorre em um evento
inesperado, portanto incomum. Dessa forma eu considero este objetivo de menor
prioridade comparado ao segundo objetivo.

O segundo caso é essencial para a aplicação, visto que valida a sua capacidade
para a carga diária planejada de usuários. O retorno é imediato, visto que no
momento em que sua aplicação for ao ar (caso o teste tenha sido feito
corretamente), sua aplicação tende a sofrer a mesma carga testada.


Vamos tentar executar um teste básico?

## Instalação

Primeiramente é preciso instalar o JMeter, o que é muito fácil. Lembrando que
você precisa ter o Java instalado no seu computador, a partir da versão 6.

Caso esteja no Mac:

    $> brew install jmeter.
    $> jmeter

Em outros casos, você pode baixar aqui:

[http://jmeter.apache.org/download_jmeter.cgi](http://jmeter.apache.org/download_jmeter.cgi)

Estou usando a versão 2.13 aqui, caso tenha pressa, este é o link direto:

[http://mirror.nbtelecom.com.br/apache//jmeter/binaries/apache-jmeter-2.13.tgz](http://mirror.nbtelecom.com.br/apache//jmeter/binaries/apache-jmeter-2.13.tgz)

Após baixar, descompacte em algum local da sua máquina. O arquivo que você vai
precisar está na pasta “bin” do pacote baixado. Caso esteja utilizando windows,
o arquivo é o jmeter.bat, caso contrário, jmeter.

## Os componentes básicos do JMeter.

A interface do JMeter é bastante simples. Na lateral esquerda temos uma árvore
com os componentes do teste, e do lado direito, a configuração de cada um destes
componentes.

O JMeter é composto basicamente dos seguintes componentes:

- **Plano de Teste**: É o “container” de todo o teste.
- **Grupos de Usuários**: É a “persona” do teste. Para um teste real, você deve
  configurar um grupo de usuário para cada cenário de uso que você tiver.
Exemplos: Usuário que acessa a primeira página e sai, Usuário já registrado, que
acessa e executa algumas ações básicas diariamente. Usuário que se registra e
acessa apenas as páginas iniciais.
- **Testes**: Existem diversos tipos de testes que podem ser realizados. O mais
  comum (para aplicativos web/mobile) é o tipo **Requisição HTTP**. Você deve
configurar os testes dentro de cada grupo de usuário e é aqui que você irá
definir as ações dos seus usuários.
- **Ouvintes**: Aqui é onde você colhe os resultados do seu teste. Você pode ter
  ouvintes do tipo Gráfico ou Tabelas. Cada um irá te fornecer os dados de uma
forma distinta.

## Teste como sua aplicação se comporta em um cenário mínimo.

Antes de validar sua aplicação com uma carga completa, verifique como ela se
comporta em um cenário normal. Neste testes, vamos tentar entender como a
aplicação se comporta com uma carga de 50 usuários acessando a aplicação no
período de 1 minuto.

### Crie um Grupo de Usuários.

O primeiro passo é criar um **grupo de usuários**:

- Clique com o botão direito em **“Plano de Teste"**.
- Clique em **“Adicionar > Threads (Users) > Grupos de Usuários"**
- Estamos esperando um grupo de 50 usuários. Insira **“50”** em **“Número de
  Usuários Virtuais (threads)”**.
- O tempo de chegada destes usuários é ao longo de 1 minuto. Insira **“60”** em
  **“Tempo de Inicialização”**.

Um detalhe importante é que não iremos testar que no primeiro segundo, 50
usuários farão uma requisição, pois estamos tentando simular um cenário real.
Aqui, esperamos que ao longo de um minuto, 50 usuários realizarão uma uma
requisição (pouco mais de 1 segundo por usuário).

### Defina o comportamento dos seus usuários.

Com o grupo de usuários criado, o próximo passo é configurar suas ações
fazer.

- Clique com o botão direito em **“Grupo de Usuários”**.
- Clique em **“Adicionar > Testador > Requisição HTTP”**.
- Insira o endereço do servidor no campo **“Nome do Servidor ou IP”**. Aqui você
  deve inserir apenas o domínio ou IP, exemplo: “google.com”, sem “HTTP” ou
“HTTPS”.
- Nossos usuários querem acessar nossa página inicial, então em **“Caminho”**
  inserimos **“/“**.

Existem diversas outras opções que você pode experimentar para testes mais
completos. Caso queira tentar um registro de usuário por exemplo, você pode
utilizar o campo **“Body Data”** e enviar um “json” para o servidor, bem como
alterar o campo **“Método”** para “POST”.

### Escolha o formato para a leitura dos resultados.

Finalmente, precisamos configurar como iremos visualizar o retorno dos nossos
testes. Para isso utilizaremos o ouvinte **“Ver Resultados em Tabela”**:

- Clique com o botão direito em **“Grupo de Usuários”**.
- Clique em **“Adicionar > Ouvinte > Ver Resultados em Tabela”**.

O último passo é salvar nosso plano. Clique em **“Arquivo > Salvar”**

### Execute o teste.

Agora podemos executar nosso teste. No menu superior selecione **“Executar >
Iniciar”** ou clique em “Play” na barra superior. Durante a execução, você deve
visualizar no ouvinte **“Ver Resultados em Tabela”**, cada uma das requisições
que estão sendo realizadas, nas seguintes colunas:

- **Tempo da amostra**: É um valor em ms, que informa o tempo total daquela
  requisição.
- **Estado**: Informa se a requisição foi um sucesso ou se ocorreu com erros.
- **Bytes**: Conteúdo transferido naquela requisição.
- **Latency**: Tempo da requisição, até o primeiro resultado. Inclui o tempo que
  o próprio JMeter levou para gerar a requisição e o tempo que o JMeter levou
para processar a primeira parte da resposta.
- **Connect Time**: Tempo para estabelecer uma conexão com o servidor de destino
  (inclui _handshake ssl_).

## Utilize a linha de comando para testes de performance reais.

Embora seja fácil executar os testes utilizando a interface do JMeter, isso não
é recomendado. Você deve utilizar no máximo 50 usuários neste ambiente, e
utilizá-lo apenas para validar seu plano de testes. Após validar o plano e
salvar, você deve rodar o plano utilizando o jmeter em linha de comando.

Abra o terminal e acesse a pasta onde o JMeter está instalado e execute o
comando abaixo, substituindo **“your_plan.jmx”** pelo plano salvo.

    $> bin/jmeter -n -t your_plan.jmx -l log.jtl

O formato padrão de saída é chamado **“Summarizer”**, e é mais do que suficiente
para um teste eficaz. Ele exibirá no próprio terminal a seguinte saída a cada
(cerca de) 30 segundos:

<script src="https://gist.github.com/jaisonerick/51620d4f5062d18cd045.js"></script>

Os valores após `summary +` dizem respeito aos novos testes, desde o último
output. Os valores após `summary =` são o agregado de toda a execução do teste.

Tomando como exemplo a primeira linha:

<script src="https://gist.github.com/jaisonerick/eb343d8dc2dc5dd17719.js"></script>

O teste executou **8 requisições** em **10 segundos**, ou seja **0,8 por
segundo**. O tempo médio de amostra foi de **1750 ms**, o mínimo foi de **1023
ms** e o máximo foi de **2646 ms**. Não houve nenhum erro. No momento deste
resultado, **2 usuários estão ativos**, **9 foram iniciados** no período
observado e **7 foram finalizados**.

O resultado final do teste está na última linha.

## Analise a Capacidade, Tempo de Resposta e Taxa de Erros.

O JMeter é excelente para executar testes de performance e de stress, porém é
ainda mais importante saber quais resultados são julgados aceitáveis. Em geral
estes resultados contemplam 3 métricas:

- **Capacidade**: É a métrica de **“usuários por segundo”** do jmeter.
- **Tempo de resposta**: Eu costumo utilizar o campo **“Avg”** como principal,
  porém é importante dar uma olhada nos outros campos também.
- **Taxa de erros**: É importante manter este valor baixo. Em testes não
  destrutivos, este valor deve sempre permanecer em **0%**.

Você sempre deve determinar valores para as 3 métricas em seu objetivo para ser
eficaz, caso contrário você corre o risco de aumentar o tempo de resposta
enquanto aumenta a quantidade de usuários por segundo e considerar seu servidor
um sucesso.

## Teste o que acontece quando as coisas saem do controle.

Os cenários testados acima ocorrem diariamente, porém o mesmo plano de testes
pode ser usado para fazer alguns testes destrutivos.

Um caso típico que pode ser testado, é o processo de deploy. Você sabe como seu
servidor reage ao deploy **enquanto** usuários utilizam sua aplicação? Utilize o
seu plano de testes básico e deixe ele rodando, enquanto simula um processo de
deploy.

Outros casos interessantes:

- O que acontece se seu banco de dados principal sair do ar? Você tem um banco
  de dados backup? Quantos erros acontecem até que o seu backup assuma?
- O que acontece se algum serviço auxiliar ([Redis](http://redis.io/),
  [Memcached](http://memcached.org/)) cair?
- Caso um servidor seu cair? Você possui servidores auxiliares? Existe um
  controle de carga entre eles? Qual o percentual de erros no caso de queda de
um servidor?

Para testar os limites da sua infraestrutura, aumente progressivamente o número
de threads e o número de ciclos, até o número de erros começar a aumentar. Neste
ponto, você chegou ao limite de 100% de respostas do seu servidor. Caso este
limite possa vir a acontecer em picos de acesso recorrentes, considere uma
estratégia de **“auto-scaling”** para atender a demanda.

## Use o bom e velho Ruby para construir seus planos de teste.

Usar a interface do JMeter é um ótimo caminho para o aprendizado, mas em linhas
gerais é bastante lento pois existem muitas opções a serem configuradas e a
usabilidade não é sua prioridade. Pensando nisso, foi criada a biblioteca
[ruby-jmeter](https://github.com/flood-io/ruby-jmeter) que permite que você
construa seus planos de teste utilizando Ruby.

Para visualizar melhor a facilidade, o código abaixo descreve o mesmo cenário
que utilizamos acima:

<script src="https://gist.github.com/jaisonerick/a6ef93efde501145e645.js"></script>

Este script gera o conteúdo de um arquivo de plano de testes do JMeter (`.jmx`).
Para capturar e salvar em um arquivo, utilize o seguinte comando (supondo que o
nome do arquivo do script seja `test_plan.rb`:

    $> ruby test_plan.rb > test_plan.jmx

Agora você pode executar este plano com o JMeter:

    $> bin/jmeter -n -t test_plan.jmx -l log.jtl

