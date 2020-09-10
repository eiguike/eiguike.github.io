---
layout: post
author: eiguike
title:  "usando npm link"
date:   2020-09-10 00:14:01 -0300
categories: openwrt yeelight blog post development
---
durante o desenvolvimento podemos encontrar alguns problemas, tanto em nossa aplicações como também nas dependências utilizadas, e sendo esse último, não muito fácil de modificar; dessa forma, o comando `npm link` vem para ajudar nisso.

### o que o comando npm link faz?
a principal função deste comando é trocar a forma de utilizar as dependências, no caso, a forma convencional é baixar as dependências e deixa-lás no `node_modules`, com o comando, é possível trocar o código da dependência por uma que você modificou.

#### como utilizar o npm link?
- devemos baixar a dependência que queremos modificar (git clone ...), posteriormente acessar a pasta e digitar o comando npm link
- logo após, dentro da pasta da nossa aplicação, devemos digitar `npm link <nome_do_pacote>` 

#### como remover o npm link?
- devemos na pasta da dependência executar o comando: `npm unlink`
- dentro da pasta da nossa aplicação, devemos executar o comando: `npm unlink <nome_do_pacote>`

### conclusões
essa é uma das maneiras de modificar código dependente no nodejs.

### referências
[https://docs.npmjs.com/cli/link](https://docs.npmjs.com/cli/link) - Documentação do comando
