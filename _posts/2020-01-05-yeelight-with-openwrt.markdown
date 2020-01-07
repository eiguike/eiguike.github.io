---
layout: post
author: eiguike
title:  "yeelight com openwrt"
date:   2020-01-05 00:14:01 -0300
categories: openwrt yeelight blog post development
---
olá! isso daqui será um dos meus primeiros posts deste blog - uma das coisas que queria muito fazer.

eu não sei muito bem como começar ele mas gostaria de falar um pouco sobre como consegui controlar uma lâmpada yeelight com um roteador.

quem me conhece sabe das minhas dificuldades em gerenciar o sono, e que foi a grande motivação desse projeto. a idéia é simplesmente me ajudar a acordar cedo e quem sabe realizar um pouco dessa meta de 2020.

### tecnologias envolvidas

o projeto em si ele é executado em um roteador porquê é o único dispositivo que fica ligado 24/7 aqui em casa, e também ele fica conectado diretamente com a lâmpada yeelight (duh~). Uma das coisas legais de citar é que o roteador permite a instalação do OpenWRT, um sistema operacional Linux especializado para roteadores. 

o roteador utilizado foi o Asus RT-AC51U que possui umas especificações bem legais, porém não possui tanta memória ram, por esse motivo acabei decidindo utilizar a linguagem C para desenvolvimento.  (citar outras libs em outras linguas?)

a yeelight ela á uma smart lâmpada que é utilizada para automação residencial, no caso, utilizarei ela como um alarme que ficará acendendo e apagando no período de 5 minutos. ela utiliza o protocolo SSDP (UDP, broadcast) para que outros serviços possam identificar a lâmpada na rede, e o envio de comando feito por TCP simples.

iremos explicar melhor os protocolos, segue comigo.

### execução

o primeiro passo foi preparar a infraestrutura da aplicação no que consiste em instalar o OpenWRT no roteador; no geral não tive muitos problemas, porém acabei perdendo algumas funcionalidaes do equipamento que ainda não foram implementadas na versão que utilizei (era a única versão disponível para esse rotador 18.06.5).

a lâmpada yeelight utliiza o protocolo ssdp (modificado) para realizar o descobrimento do dispositivo na rede por outros serviços, um desses serviços poderia o ser google assistant ou então o aplicativo da Yeelight mesmo.

| foto do ssdp |
| -------- |
| Funcionamento do SSDP |

para realizar o descobrimento devemos mandar a seguinte mensagem para o ip `239.255.255.250` e porta `1982`: 

<<< imagem do wireshark com o packet com as informações >>
```
M-SEARCH * HTTP/1.1
MAN: "ssdp:discover"
ST: wifi_bulb
```

<< explicação do broadcast >>

após o envio da mensagem a lâmpada responderá com a seguinte mensagem:
```
mensagem de resposta
```
os únicos que iremos extrair da lâmpada é o IP e o ID único, dessa forma já conseguimos enviar comandos.





--- explicar sobre o comando simples tcp



desenvolvimetno do alarm-yeelight começando com uma versão simples do ssdp, posteriormente

-> ssdp passos


após a identifcação da lâmpada

### dificuldades

### o que eu aprendi?

### conclusão

### referências
