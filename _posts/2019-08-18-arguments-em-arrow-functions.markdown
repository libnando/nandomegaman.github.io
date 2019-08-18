---
layout: post
title:  "Arguments em arrow functions"
date:   2019-08-18 17:05:00
tags: [web, js, es6, javascript, arguments, arrow-functions, spread-operator]
---

Uma das grandes vantagens do poder js é a flexibilidade e a liberdade que essa linguagem fornece para o desenvolvedor web. 

Inclusive, recentemente (nem tanto mais) houve uma das maiores features da linguagem (ES6/ES2015), muitos recursos maneiros foram adicionados, segue um resuminho básico <a href="https://www.w3schools.com/js/js_es6.asp" target="_blank">aqui</a>. 

### Vamos ao caso

Poucas vezes precisei recuperar os argumentos de função desconsiderando os parâmetros e utilizando a variável local "arguments", mas dessa vez eu precisava disso e algo deu errado. Como tenho o costume de usar sem moderação arrow functions fui pego de surpresa!

Abaixo vou exibir um exemplo do que eu queria fazer:

```javascript

const fnA = () => {    
    let args = Array.from(arguments);
    console.log(args);    
    //ERROR: arguments is not defined!  
};

```

Como assim? Não tenho mais acesso a arguments! Numa busca rápida me informo que para escopo de arrow functions o contexto interno "arguments" não existe mais. :(

Implementando com tradicional "function" funciona normalmente:

```javascript

const fnB = function(){        
    let args = Array.from(arguments);
    console.log(args);    
    //OK
};

```

Aí, me restou utilizar um carinha que eu já tinha visto em códigos, mas nunca havia aplicado. 
  
### Rest parameter

Também um recurso recente do es6, rest parameter dispensa utilizar a variável interna "arguments" mas força o uso de "..." em um parâmetro na função. Mais detalhes <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Functions/rest_parameters" target="_blank">aqui</a>

O legal é que esse recurso deixa o código com uma legibilidade melhor.

Abaixo exemplo:

```javascript

const fnC = (...args) => {        
    console.log(args);    
    //OK    
};

//Combinando com parâmetros

const fnD = (p1, p2, ...args) => {        
    console.log(p1); 
    console.log(p2); 
    console.log(args);    
};

//Chamadas

fnA("o/", 12, 30); //error (arguments is not defined)
fnB("o/", 12, 30); // ["o/", 12, 30]
fnC("o/", 12, 30); // ["o/", 12, 30]
fnD("o/", 12, 30); // o/,12,[30]


```

Bacana! 

Só um adendo para o conceito "..." no js, tem esse carinha aqui "spread operator", que é utilizado como um auxiliar de variáveis expandido os valores para array, maiores detalhes <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Operators/Spread_operator" target="_blank">aqui</a>.