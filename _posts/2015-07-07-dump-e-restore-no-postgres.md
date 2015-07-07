---
layout: post
title: Dump e Restore no Postgres
post_author: Lucas Prim
post_gravatar: d7556befd6b2a40d5618be1c0340e54a
---

## Dumping e Restoring com eficiência

Vira e mexe surge algum problema em produção que é dificil de reproduzir
localmente. A solução: dump no db remoto, seguido de algum tempo de download,
terminando com mais algum tempo de restore.

Hoje, fazendo isso, me perguntei: Será que tem uma forma mais eficiente de se
fazer isso? A resposta é: tem!

O postgres possui um protocolo "custom" que permite que você faça dumps de
postgres para postgres com arquivos de tamanho extremamente reduzidos.
Quão reduzidos? Bom hoje eu consegui fazer um dump de **800MB** virar um dump de
**94MB**:

![Milagre da redução](https://www.dropbox.com/s/e0j6ud9lfuh6huh/Screenshot%202015-07-07%2017.19.37.png?raw=1)

O comando que estou usando para gerar o dump é o seguinte:

`pg_dump -Fc --no-acl --no-owner -h HOST_DO_DB -U SEU_USUARIO DATABASE > ~/db.dump`

Para restaurar esse dump, basta usar o comando a seguir:

`pg_restore --verbose --clean --no-acl --no-owner -h HOST -U SEU_USUARIO -d DATABASE db.dump`

Me salvou alguns minutos de vida, espero que salve tempo seu também!
