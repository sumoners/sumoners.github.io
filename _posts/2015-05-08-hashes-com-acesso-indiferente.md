---
layout: post
title: Hashes com acesso indiferente
post_author: Paulo Geyer
post_gravatar: e0ffb3df640bdfc20a10ee433268cea4
---

No Youbiquity usamos frequentemente o tipo jsonb do PostgreSQL, em alguns momentos pegamos os dados da sessão e salvamos diretamente no jsonb (.to_json converte os símbolos em strings). Mas isso gera alguns problemas, o Ruby permite usar tanto símbolos (:foo) quanto strings ('bar') como índices de um Hash, podendo existir um h['foo'] e h[:foo] com valores diferentes.

Outra fonte de confusão, é que o session do Rails não tem esse problema, h['foo'] e h[:foo] retornam o mesmo valor. Foi aí que descobri a classe HashWithIndifferentAccess, que é usada na session do Rails.

<!-- more -->

O HashWithindifferentaccess usa strings como índice, independente de você usar session[:foo], internamente a chave é uma string, evitando essa confusão de symbol e string.

## pro tip

HashWithIndifferentAccess.new é muito longo, felizmente o Rails cria um método no Hash que facilita o uso, que é o .with_indifferent_access, enão HashWithIndifferentAccess.new e {}.with_indifferent_access geram o mesmo resultado

esses são os meus 2 centavos essa semana, happy hacking!
