---
layout: post
title:  "DRY pra toda vida"
date:   2022-07-09 17:00:00
tags: [dev, dotnet, c#, code, patterns]
---

Olá amiguinhos, hoje vamos falar sobre um princípio maroto com o acrônimo <b>DRY</b>. 

Vou começar explicando esse princípio com uma analogia simples que me veio na cabeça agora.

### O problema da repetição

Imagine que você tem duas filhas (que é o meu caso) e agora você precisa informar elas que a partir da data de hoje elas passarão a receber mesada, então você chama a filha nº 1 para informar ela que a partir de hoje ela vai começar receber mesada. 

Em seguida você chama a filha nº 2 para informá-la que ela também passará a receber mesada a partir de hoje.

### Pare de ser repetitivo e seja objetivo

Essa analogia simples que descrevi acima foi basicamente para exemplificar casos do cotidiano que involuntariamente ocorrem na vida do desenvolvedor. 

Ainda sobre a analogia acima, seria bem mais simples chamar as duas filhas uma única vez e proferir a informação para as duas filhas juntas e tudo estaria estupidamente claro, nesse ponto já entro em outro princípio que futuramente abordarei aqui. <b>(KISS - Keep it simple stupid)</b>.

### Estudo de caso

Abaixo vou compartilhar um exemplo básico de duplicidade de código (em c#) e em seguida apresento uma simples solução.

Basicamente o código irá imprimir na tela: 'Hello!', 'Goodbye!' e 'Hello! Welcome!'.

```cs
//my methods...
    
string SayHello() => "Hello!";
	
string SayGoodbye() => "Goodbye!";    
	
string SayHelloWelcome() => "Hello! Welcome!";
	
void Prophesy(){
    //Hello!
    Console.WriteLine(SayHello());
        
    //Goodbye!
    Console.WriteLine(SayGoodbye());

    //Hello! Welcome!!
    Console.WriteLine(SayHelloWelcome());
}

//go go go!
Prophesy();
```

Agora, abaixo uma pequena refatoração para o código 'não se repetir'.

```cs
//my methods...
    
string SayHello() => "Hello!";
	
string SayGoodbye() => "Goodbye!";    
	
string SayHelloWelcome() => $"{SayHello()} Welcome!";
	
//...
```

### Não se repita amiguinho

Conforme o exemplo 'mamão com açucar' listado acima, não há menor necessidade de reescrever a mesma palavra 'Hello!' duas vezes, sendo que apenas um método gerando-a já irá atender o outro método. Creio que deu pra ter uma boa idéia de poder abstrair a essência desse princípio né! =)

Claro que fui bem simplista e objetivo na minha analogia e exemplo, porém reforço aqui que para mantermos nosso código com qualidade, alta legibilidade e excelência é fundamental estar com esse princípio fresquinho na mente diariamente.