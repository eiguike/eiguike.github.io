---
layout: post
author: eiguike
title:  "yeelight com openwrt"
date:   2020-01-05 00:14:01 -0300
categories: openwrt yeelight blog post development
---
olá!

isso daqui será um dos meus primeiros posts deste blog - uma das coisas que queria muito fazer. a ideia não é preocupar com muitas coisas e apenas escrever, vamos ver até aonde isso vai levar.

eu não sei muito bem como começar mas gostaria de falar um pouco sobre como consegui controlar uma lâmpada Yeelight com um roteador.

quem me conhece sabe das minhas dificuldades em gerenciar o sono, uma das grandes motivações por trás deste projeto. a idéia é simplesmente me ajudar a acordar cedo e quem sabe realizar um pouco dessa meta de 2020.

todo código baseado neste artigo se encontr nesse link [aqui](https://github.com/eiguike/alarm-yeelight), fiquem à vontade para mandar pullrequests também!

### tecnologias envolvidas

o projeto em si ele é executado em um roteador porquê é o único dispositivo que fica ligado 24/7 aqui em casa, e também ele fica conectado diretamente com a lâmpada yeelight (duh~). Uma das coisas legais de citar é que o roteador permite a instalação do OpenWRT, um sistema operacional Linux especializado para roteadores. 

o roteador utilizado foi o Asus RT-AC51U que possui umas especificações bem legais, porém não possui tanta memória ram, por esse motivo acabei decidindo utilizar a linguagem C para desenvolvimento.

a yeelight ela á uma smart lâmpada que é utilizada para automação residencial, no caso, utilizarei ela como um alarme que ficará acendendo e apagando no período de 5 minutos. ela utiliza o protocolo SSDP (UDP, broadcast) para que outros serviços possam identificar a lâmpada na rede, e o envio de comando feito por TCP simples.

iremos explicar melhor os protocolos, segue comigo.

### execução

#### preparar infraestrutura
o primeiro passo foi preparar a infraestrutura da aplicação no que consiste em instalar o OpenWRT no roteador; no geral não tive muitos problemas, porém acabei perdendo algumas funcionalidaes do equipamento que ainda não foram implementadas na versão que utilizei (era a única versão disponível para esse rotador 18.06.5).

depois que você instala, já é possível acessar por ssh o roteador!
```
BusyBox v1.28.4 () built-in shell (ash)

  _______                     ________        __
 |       |.-----.-----.-----.|  |  |  |.----.|  |_
 |   -   ||  _  |  -__|     ||  |  |  ||   _||   _|
 |_______||   __|_____|__|__||________||__|  |____|
          |__| W I R E L E S S   F R E E D O M
 -----------------------------------------------------
 OpenWrt 18.06.5, r7897-9d401013fc
 -----------------------------------------------------
```

a partir daqui já tenho um lugar disponível para deixar o alarme executando!

#### identificar yeelight
a lâmpada yeelight utiliza o protocolo ssdp (modificado) para realizar o descobrimento do dispositivo na rede por outros serviços, um desses serviços poderia o ser Google Assistant assistant ou então o aplicativo da Yeelight mesmo.

```
                                                                   +-------------+
                                                                 +>+ Dispositivo |
 +-------------+     UDP M-SEARCH    +------------+              | +-------------+
 |             |  (239.255.255.250)  |            |  *Broadcast  |
 | Dispositivo +--------------------->  Roteador  +-------------------->--+  +-+
 |             |                     |            |              |
 +-------------+                     +------------+              | +-------------+
                                                                 +>+ Dispositivo |
                                                                   +-------------+
```

no exemplo acima, temos esquematizado o funcionamento do ssdp, basicamente é enviado um comando de identificação `M-SEARCH` em broadcast (para todos os dispositivos conectados na rede), dessa forma, o dispositivo que saiba responder essa mensagem normalmente irá responder com as características do aparelho. 

```
M-SEARCH * HTTP/1.1
MAN: "ssdp:discover"
ST: wifi_bulb
```

na yeelight por exemplo, as características como estado da lâmpada, nome, cores disponíveis, ip, id e entre outros serão ser retornados na resposta.

```
HTTP/1.1 200 OK
Cache-Control: max-age=3600
Date:
Ext:
Location: yeelight://192.168.64.136:55443
Server: POSIX UPnP/1.0 YGLC/1
id: 0x000000000100ae44
model: color
fw_ver: 51
support: get_prop set_default set_power toggle set_bright start_cf
stop_cf set_scene cron_add cron_get cron_del set_ct_abx set_rgb
set_hsv set_adjust adjust_bright adjust_ct adjust_color set_music
set_name
power: on
bright: 100
color_mode: 2
ct: 6468
rgb: 16750848
hue: 36
sat: 100
```

*um fato engraçado durante o o desenvolvimento, é que o protocolo ssdp utiliza a camada de transporte UDP, porém o protocolo aplicação HTTP 1.1 é totalmente TCP ¯\_(ツ)_/¯*

pela resposta da lâmpada apenas extraímos o IP e o ID, dessa forma já conseguimos enviar comandos!

todo passo acima esta sendo feito na função `yeelight_integration_get_lamps` do arquivo `src/library/yeelight_integration.c`.

#### enviar comandos para yeelight

os comandos da yeelight são enviados via TCP com o corpo da mensagem em JSON. 

há uma grande quantidade de comandos em que você pode enviar, aqui iremos utilizar apenas o `set_power` para controlar a lâmpada. no manual de operações da yeelight há informações sobre outros comandos e também os possíveis parâmetros.

o comando `set_power` tem 4 parâmetros (power, effect, duration e mode), porém só consegui utilizar apenas o primeiro dele :( talvez no update do firmware eles tenham removido a possibilidade.

para enviar o comando, enviamos para o IP identificado na primeira parte e porta `55443`.

```
{"id": 141278408, "method": "set_power", "params": ["off"]}
```

```
{"id":141278408,"result":["ok"]}
```

no código essa operação é feita na função `yeelight_send_command` do arquivo `src/library/yeelight_lamp.c`. 

| ![image alt](https://i.imgur.com/wwT3XYS.gif) |
| :--------:
| bem que poderia tocar um som agora|

#### juntando tudo no roteador

### conclusão
no geral a experiência foi bem legal, pude aprender bastante coisas tanto na parte de criar softwares para o OpenWRT (toda a parte de `packging`), aprender sobre os protocolos de aplicação, relembrar e tentar umas coisas novas na linguagem C (make, cmake, funções variádicas e libs terceiras) e também a utilização do Wireshark (apesar de não tê-lo citado no artigo).


### referências
- [Yeelight Inter Operation Specs](https://www.yeelight.com/download/Yeelight_Inter-Operation_Spec.pdf)
- [Hello world! for OpenWrt](https://openwrt.org/docs/guide-developer/helloworld/chapter1)
