---
layout: post
title:  "Promises, precisamos falar sobre"
date:   2019-03-05 23:18:00
tags: [web, js, es6, javascript, promise]
---

Olá, faz um tempinho que "brinquei" com promises js, mais específicamente com api fetch, que já utiliza promise como retorno, então anotei para posteriormente abordar aqui de forma objetiva sobre esse recurso maroto já nativo nos browsers atuais (não vou entrar em detalhes sobre pollyfills, etc). ;)

### Wtf, promises?



A minha forma de explanar esse recurso é simples, uma promessa manipula de forma amigável o retorno da funcionalidade executada, e, inclusive com processamento assíncrono. 

Vou tentar abaixo via código mostrar a funcionalidade desse recurso simplório mas muito eficaz.

Por padrão uma promessa recebe 2 argumentos, a função para resolução e outra para tratar a rejeição (erro) caso a promessa não for "cumprida".

Não vou entrar nos inúmeros detalhes desse recurso, ao final do post vou deixar links que de forma técnica e bem específica explana todos detalhes do recurso.

O exemplo abaixo tem por objetivo comprar itens de um supermercado.

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

Contextualizando juntamente com o critério citado acima, a função **_buySomething** é chamada sempre que preciso comprar algo. Ela retorna uma promessa de compra, que poderá ser efetivada ou não, o critério que utilizei ali foi simplesmente o financeiro, se tem dinheiro compra o item (resolve), senão não irá comprar (reject).

Note que caso o custo do produto for maior que o dinheiro "em caixa" irá executar o callback **reject** passando uma mensagem de falha, caso contrário, por padrão executa o callback **resolve**;

Abaixo vamos realizar as compras:

```javascript

const mainDiv = document.getElementById("main");
    
    //lista de compras com valor
    const items = [
        {val:8, name :"cerveja"},
        {val:11, name:"brócolis"},
        {val:5, name:"café"},
        {val:15, name:"salada"},                
        {val:2, name:"chocolate"},
        {val:6, name:"água"}
    ];

    //percorre a lista
    items.forEach(e => {
               
        //meu elemento html que receberá o resultado da promessa de compra               
        let lineDiv = document.createElement("div");         

        //executa a compra
        _buySomething(e.val, e.name)
        .then((result) => {
            //caso a compra for concluída
            lineDiv.innerHTML = result;
            mainDiv.append(lineDiv);
        })
        .catch((err) => {
            //caso for rejeitada irá cair aqui
            lineDiv.innerHTML = err;
            mainDiv.append(lineDiv);
        });

    });

```

Utilizamos **then** para o estado resolvido da promessa e **catch** para o estado rejeitado da promessa.

O exemplo acima irá desenhar no html o seguinte resultado:

```

você comprou cerveja
vocẽ não pode comprar brócolis
você comprou café
vocẽ não pode comprar salada
você comprou chocolate
você comprou água

```

Legal né! o/

Novos recursos nativos do ecmascript já estão utilizando promises como retorno, como anteriormente citado api fetch e também cache api.

Um exemplo bacana também pra testar promessas é consumindo requisições do nosso amigo eterno **XMLHttpRequest**, nesse exemplo <a target="_blank" href="https://github.com/mdn/js-examples/blob/master/promises-test/index.html">aqui</a> é simulado o carregamento de uma imagem de forma assíncrona. Recomendo dar uma olhada.

Caso queira saber mais sobre promises, a <a target="_blank" href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Promise">mozilla</a> nunca decepciona. <a target="_blank" href="https://www.promisejs.org/">Aqui</a> também tem coisas legais sobre promises.

O exemplo do post está no meu **github**, caso queira conferir está <a target="_blank" href="https://github.com/libnando/cute-promises">bem aqui</a>.

Por hoje era isso amiguinhos.