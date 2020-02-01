---
layout: post
title:  "Polêmica! Chegou a hora de falar sobre abort..."
date:   2020-02-02 13:05:00
tags: [web, js, es6, javascript, fetch-api, promisses, abortcontroller]
---

<br />

### AbortController

Hahahaha, clickbait? Até pode ser... ¬¬ mas a idéia aqui é falarmos brevemente sobre um recurso interessante e recente da web.

Hoje na web com "js nativo" (vanilla js) temos o fofinho e clássico 'XMLHttpRequest" que dá conta do recado quando assunto é requisições assíncronas, mas o fato é que a web evoluiu juntamente com a comunidade open source pautando as demandas. Esse recurso é só mais um desses casos (dá uma olhada <a href="https://github.com/whatwg/fetch/issues/27" target="_blank" title="https://github.com/whatwg/fetch/issues/27">aqui nessa issue</a>).

Então, como estou utilizando o <b>fetch api</b> (que também é um recurso relativamente novo e ainda há pollyfills pra ele) para minhas requisições assíncronas nativas e o meu contexto era básico: Eu precisava colocar uma base simples de tempo limite que a requisição poderia durar, coisa simples né? Nada que um simples timeout do XMLHttpRequest não resolva, mas conforme fui pesquisando o fetch api não há nativamente uma propriedade pra configurar isso, e depois de algumas pesquisas rápidas pra evitar "workarounds" (haha) acabei encontrando esse recurso chamado AbortController que surgiu exatamente pra isso (cancelar requisições ao solicitar).

### Vamos brincar

Abaixo um exemplo simples de uso desse recurso, no caso em questão, defino um tempo de 15 segundos para abortar uma requisição.

```javascript

const controller = new AbortController();
const signal = controller.signal;
const url = "https://...";

setTimeout(() => controller.abort(), 15000);

fetch(url, { signal })
.then(response => {
  return response.json();
}).then(json => {
  console.log(json);
});

```

Note que passamos o <b>signal</b> do <b>AbortController</b> por paramêtro para o fetch e juntamente com a function nativa <b>setTimeout</b> definimos um tempo pra abortar. Simples né?

Agora vamos a outro exemplo mais "interativo".

<div style="padding:20px; border:1px solid #CCC; max-height:400px;">

<button data-for="bstart">Iniciar</button>
<button data-for="babort" disabled>Abortar</button>
<pre data-for="bout"></pre>

</div>

### Concluindo

Sobre <b>fetch api</b>, caso queira saber um pouco sobre o retorno desse recurso, que são <b>promisses</b>, dá uma olhada aqui <a href="https://libnando.com/2019/03/promises-precisamos-falar-js.html" target="_blank" title="Promisses, precisamos falar sobre">nesse post</a>, pois explanei aqui sobre isso. =)

Basicamente era essa a mensagem amiguinhos, agradeço a presença aqui. o/

<b>Refêrencias</b>: https://developer.mozilla.org/en-US/docs/Web/API/AbortController

<script>
      const bOut = document.querySelector('[data-for="bout"]');
      const bStartBtn = document.querySelector('[data-for="bstart"]');
      const bAbortBtn = document.querySelector('[data-for="babort"]');
      const decoder = self.TextDecoder && new TextDecoder();
      let fetching = false;
      let controller;
      
      function log(message) {
        const txt = document.createTextNode(message);
        bOut.appendChild(txt);
      }
      
      function badTextDecoder(bytes) {
        if (decoder) return decoder.decode(bytes, {stream: true});        
        return bytes.reduce((str, byte) => str + String.fromCharCode(byte), '');
      }
      
      bStartBtn.addEventListener('click', async event => {
        bStartBtn.disabled = true;
        bAbortBtn.disabled = false;
        
        try {
          controller = new AbortController();
          const signal = controller.signal;
        
          const response = await fetch('data', {signal});
          
          if (response.body) {
            const reader = response.body.getReader();

            while (true) {
              const {value} = await reader.read();
              log(badTextDecoder(value));
            }
          }
          else {
            log(`Fetching…`);
            await response.text();
          }
          
        }
        catch (e) {
          console.log(e);
          log(`\nError: ${e.name}: ${e.message}\n`);
        }
        
        bStartBtn.disabled = false;
        bAbortBtn.disabled = true;
        
      });
      
      bAbortBtn.addEventListener('click', event => {
        controller.abort();
      });
    </script>