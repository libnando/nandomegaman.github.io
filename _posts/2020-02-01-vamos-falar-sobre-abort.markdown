---
layout: post
title:  "Chegou a hora de falar sobre abort..."
date:   2020-02-02 13:05:00
tags: [web, js, es6, javascript, fetch-api, promisses, abortcontroller]
---

<br />

### AbortController!

Hahahaha, clickbait? Até pode ser... ¬¬ mas a idéia aqui é falarmos brevemente sobre um recurso interessante e recente da web.

<strike>Na verdade, sobre aborto mesmo, eu dei uns <a href="https://libnando.com/2017/12/principio-da-nao-agressao.html" target="_blank">pitacos aqui</a> faz um tempinho, mais no contexto da filosofia política que eu tenho apreço, libertarianismo. Mas esse não é o caso agora.</strike>

<div class="padding:5px;">&nbsp;</div>

Hoje na web com "js nativo" (vanilla js) temos o fofinho e clássico <b>XMLHttpRequest</b> que dá conta do recado quando assunto é requisições assíncronas, mas o fato é que a web evoluiu juntamente com a comunidade open source pautando as demandas. Esse recurso é só mais um desses casos (dá uma olhada <a href="https://github.com/whatwg/fetch/issues/27" target="_blank" title="https://github.com/whatwg/fetch/issues/27">aqui nessa issue</a>).

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

<div style="max-height:400px; margin-bottom:20px; border:1px solid #DDD; padding-left:10px;">

<iframe src="https://fetch-abort-demo.glitch.me/" style="margin: 0; padding: 0; border: 0; width: 100%;"></iframe>

</div>

### Concluindo

Sobre <b>fetch api</b>, caso queira saber um pouco sobre o retorno desse recurso, que são <b>promises</b>, dá uma olhada aqui <a href="https://libnando.com/2019/03/promises-precisamos-falar-js.html" target="_blank" title="Promisses, precisamos falar sobre">nesse post</a>, pois explanei aqui sobre isso. =)

Basicamente era essa a mensagem amiguinhos, agradeço a presença aqui. o/

<b>Refêrencias</b>: 

- <a target="_blank" href="https://developer.mozilla.org/en-US/docs/Web/API/AbortController">https://developer.mozilla.org/en-US/docs/Web/API/AbortController</a>
- <a target="_blank" href="https://fetch-abort-demo.glitch.me/">https://fetch-abort-demo.glitch.me/</a>

<script>
     
</script>