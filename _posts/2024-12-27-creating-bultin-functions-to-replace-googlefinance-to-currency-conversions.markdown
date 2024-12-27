---
layout: post
author: eiguike
title:  "criando funcão customizada no Google Sheets para substituir o GOOGLEFINANCE para conversão de moedas"
date:   2024-12-27 01:30:01 -0300
categories: google sheets googlefinance custom scripts
---

no dia de ontem (26 de dezembro de 2024), [o Google fez uma caquinha na hora de exibir os valores da moeda dolar](https://g1.globo.com/economia/noticia/2024/12/26/google-suspende-cotacao-dolar.ghtml) (USD) basicamente exibiu o valor com 30 cents há mais convertidos em BRL.

pode não parecer muito, mas para a moeda do Brasil (BRL) que é desvalorizada, qualquer aumento desse valor pode trazer um grande impacto, seja em inflação, investimentos, viagens e etc. e aí está o problema, muitos dos sites de comércio/serviços utilizam do site do Google para avaliar o preço, e provavelmente podem ter feito algumas negociacṍes não tão favoráveis visto que o valor real não era aquele que estava sendo exibido.

e para evitar problemas maiores, o Google basicamente desativou a funcionalidade de conversão de moedas do próprio site e também no próprio `Google Sheets`, se você utiliza a função `GOOGLEFINANCE`, provavelmente neste dia ficou com todas as planilhas com erro.

# e por quê deste artigo?

bom...

eu tenho uma planilha de investimentos que utilizo já faz uns 5 anos desde que comecei minha jornada de investimentos, e corriqueiramente sempre tenho esses problemas do `GOOGLEFINANCE` retornar com erro.

e hoje foi o dia para tentar contornar de vez por toda esse problema dessa função retornar erro, ou pelo menos, contornar de uma forma que quebre menos.

# qual é a ideia da solução?

na minha planilha eu já utilizo corriqueiramente o Google Scripts para criar pequenas funcionalidades de automação, se você tem experiência com `Javascript` será um mão na roda, dessa forma, basicamente criarei uma função customizada que consegue extrair o valor da conversão da seguinte forma:

```
=CUSTOMFINANCE("USDBRL")
```

# instruções

1. criar uma conta no [OpenExchangeRates](https://openexchangerates.org/)
  - utilizaremos um serviço de API oferecido pela [OpenExchangeRates](https://openexchangerates.org/) nele você consegue todos os valores das moedas convertidos pelo dolar
  - estamos utilizando o plano `free` deste serviço, dessa forma, implementaremos um mecanismo de cache de 1 hora para não termos que toda hora consultar os valores de conversão

2. crie um app id e salve

3. na sua planilha do Google, vá em *Extensions* -> *Apps scripts*

4. adicione o seguinte código e substituia os valores necessários:

```javascript
/**
 * Retorna o valor da moeda convertida.
 * @param {text} input Ex: "USDBRL", "BRLUSD", "USDDKK".
 * @return A conversão atual da moeda requisitada.
 * @customfunction
*/
function CUSTOMCURRENCY(input) {
  const ONE_HOUR_IN_SECONDS = 3600;
  const API_KEY = "SUBSTITUA_ESSE_VALOR_AQUI";
  const openExchangeCurrencyApiUrl = `https://openexchangerates.org/api/latest.json?app_id=${API_KEY}`;

  const cache = CacheService.getScriptCache();
  let currencyText = cache.get("currency");

  if (!currencyText) {
     currencyText = UrlFetchApp.fetch(openExchangeCurrencyApiUrl, {
          method: "get",
          contentType: "application/json"
     }).getContentText();
     cache.put("currency", currencyText, ONE_HOUR_IN_SECONDS);
  }

  const currencyObject = JSON.parse(currencyText);

  const currencies = input.match(/.{1,3}/g);
  if (currencies.length != 2) {
     return 0;
  }

  const [denominator, numerator] = currencies;
  return (
     currencyObject.rates[numerator] / currencyObject.rates[denominator]
  );
}
```

5. adicione na sua planilha o comando

6. seja feliz sem o `GOOGLEFINANCE` para conversão de moedas!

# referências

- [OpenExchangeRates API documentation](https://docs.openexchangerates.org/reference/latest-json)
- [Custom functions in Google Sheets](https://developers.google.com/apps-script/guides/sheets/functions)
- [GOOGLEFINANCE documentação](https://support.google.com/docs/answer/3093281?hl=en)
