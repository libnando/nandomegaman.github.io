---
layout: post
title:  "Nosso amiguinho Promises"
date:   2019-03-05 23:18:00
tags: [web, js, es6, javascript, promise]
---

Olá, faz um tempinho que "brinquei" com promises js, mais específicamente com api fetch, que já utiliza promise como retorno, então anotei para posteriormente abordar aqui de forma objetiva sobre esse recurso maroto já nativo nos browsers atuais (não vou entrar em detalhes sobre pollyfills, etc). ;)

### Wtf, promises?

A minha forma de explanar esse recurso é simples, uma promessa retorna algo resolvido ou não. Vou tentar abaixo via código mostrar a funcionalidade desse recurso simplório mas muito eficaz.

Vou criar um exemplo simples para comprar itens em um supermercado.

Contextualizando juntamente com o critério citado acima, a função **_buySomething** é chamada sempre que preciso comprar algo. Ela retorna uma promessa de compra, que poderá ser efetivada ou não, o critério que utilizei ali foi simplismente o financeiro, se tem dinheiro compra o item (resolve), senão não irá comprar (reject).


```javascript

_buySomething = (val, itemName) => {

        const _myCash = 10;

        return new Promise(function(resolve, reject){

            setTimeout(function(){

                if(val > _myCash){                
                    reject("vocẽ não pode comprar ".concat(itemName));
                }
    
                resolve("você comprou ".concat(itemName));

            }, 1000);

        });

    };

```

