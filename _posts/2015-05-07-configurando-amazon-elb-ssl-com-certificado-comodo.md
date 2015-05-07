---
layout: post
title: Configurando o SSL da Amazon ELB com um certificado da Comodo
---

Se você comprou um certificado da Comodo e recebeu uma penca de arquivos do
tipo:

`
 AddTrustExternalCARoot.crt
 COMODORSAAddTrustCA.crt
 www_example_com.crt
 COMODORSADomainValidationSecureServerCA.crt
`

e está tentando configurar seu load balancer na Amazon usando eles,
provavelmente você está passando raiva.

Pare de passar raiva, é só rodar o seguinte comando, **exatamente nessa ordem**:

`cat www_example_com.crt COMODORSADomainValidationSecureServerCA.crt  COMODORSAAddTrustCA.crt AddTrustExternalCARoot.crt | pbcopy`

Simplesmente cole o resultado no campo "Public Key Certificate" e deixe o campo
"Certificate Chain" em branco. No campo "Private Key" use a chave que você usou
para fazer o request do seu certificado.

Seguindo esses passos você vai poder ser feliz e parar de passar raiva com essa
configuração mequetrefe!

Happy Hacking :)
