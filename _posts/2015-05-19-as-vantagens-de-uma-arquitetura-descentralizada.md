---
layout: post
title: As vantagens de uma arquitetura descentralizada
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

Na SumOne trabalhamos com sistemas complexos que lidam com uma porção de partes
móveis. Temos integração com serviços de terceiros, com nossos próprios
sistemas, uma API que pode chegar a receber centenas de milhares de reqs/min e
uma customer-facing application que precisa estar sempre com alta performance e
com uma taxa de erros no limite do zero.

## Como fazer isso em um sistema com tantas partes móveis?

Se utilizássemos uma aplicação Rails monolítica que englobasse todas as partes,
rapidamente nosso sistema ficaria cheios de abstrações, responsabilidades e
funcionalidades diferentes. Isso geraria diversos _downsides_ mas
principalmente comprometeria nossa capacidade de entregar um sistema
performático e livre de bugs para nossos clientes, que são a peça mais
importante de todo o sistema.

Por isso, optamos por utilizar uma arquitetura descentralizada, onde nossa
_customer-facing application_ fica blindada da selva de fluxo de dados que
trafega entre as outras aplicações, mais ou menos assim:

![Data Jungle](https://www.dropbox.com/s/v7z58pusyazhhj4/Rough%20Vision%20of%20Yb%20Architecture.png?raw=1)

Todos os pontos que estão em contato com o mundo externo possuem um queue e
uma conexão com a Data API que alimenta a _customer-facing app_. O queue serve
para que, caso a Data API caia, os dados fiquem retidos nos serviços
responsáveis pela sua coleta. Também serve para fazermos um _throttling_ dos
dados e controlarmos qual o fluxo que deve estar entrando na nossa Data API
a qualquer momento.

## O que você ganha com isso?

Diversas coisas, mesmo, como por exemplo:

* **Pontos de falha distribuidos**
* **Fácil escalabilidade**
* **Maior facilidade de manutenção**
* **Diminuição da barreira de entrada para novos devs**
* **Performance extraordinária para o usuário final, com ganhos de UX**
* **Controle sobre o seu fluxo de dados**
* **Independência de linguagem**

Entre outras.

## O que você perde com isso?

* **Overhead de comunicação**
* **Necessidade de documentação**
* **Infoestrutura mais complexa**
* **Menor velocidade de implementação de novos projetos**
* **Manutenção distribuida**

## Nossa conclusão

Como o Youbiquity é uma aplicação que tem como requisito básico a premissa
de **Extraordinary User Experience**, já não teriamos alternativa senão ter
uma aplicação performática blindada dos múltiplos serviços aos quais nos
conectamos.
Não obstante, julgamos que poder utilizar a linguagem mais adequada para cada
aplicação e diminuir a barreira de entrada para novos devs é motivo mais que
suficiente para adotar uma arquitetura descentralizada.

No entanto, cada projeto é diferente um do outro! No nosso caso essa solução
era trivial mas consigo enxergar outros casos em que essa separação não é tão
benéfica quanto no nosso caso.

Happy Hacking!
