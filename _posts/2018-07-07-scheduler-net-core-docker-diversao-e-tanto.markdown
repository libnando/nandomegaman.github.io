---
layout: post
title:  "Scheduler + .Net Core + Docker! Diversão e tanto!"
tags : [.net core, container, docker, quartz, scheduler, tutoriais]
date:  2018-07-07 22:00:00
---

![docker-dotnetcore](/assets/images/posts/2018/07/dockernetcore.jpg)

Faz tempo que não escrevo meus pensamentos por aqui, na medida que vou conseguindo me organizar espero começar popular também conteúdos que me interessam no contexto profissional.  :)

Esse post será um mini tutorial com base em uma pequena aplicação de agendamento tarefas (utilizando quartznet) rodando em um contêiner (docker) .net core.

Pra contextualizar meu "workspace", utilizei o sistema operacional ubuntu juntamente com vscode, a versão do .net core foi a última 2.1 (até o momento que esse post está sendo escrito).

Considerando que você já tenha instalado .net core e docker, vamos lá. 

Iniciando a aplicação

Abra o terminal e digite: 

```
dotnet new console -o NomeDaSuaApp
```

Em seguida, acesse via console o diretório da aplicação recém criada, vamos instalar o quartznet:

```
dotnet add package Quartz
```

Bacana! Agora vamos abrir nossa IDE e editar o arquivo "Program.cs" com o conteúdo abaixo:

```
using System;
using System.Threading.Tasks;
using System.Collections.Specialized;
using Quartz;
using Quartz.Impl;
using Quartz.Logging;

namespace NomeDaSuaApp
{
    public class Program
    {
        private static void Main(string[] args)
        {            
            Run().GetAwaiter().GetResult();            
        }

        private static async Task Run()
        {
            try
            {
                NameValueCollection props = new NameValueCollection
                {
                    { "quartz.serializer.type", "binary" }
                };
                
                StdSchedulerFactory factory = new StdSchedulerFactory(props);
                IScheduler scheduler = await factory.GetScheduler();

                //agendador iniciado
                await scheduler.Start();

                //setar configurações (job e trigger), na prática isso é montado dinamicamente via configurações
                IJobDetail job = JobBuilder.Create()
                    .WithIdentity("jobTeste1", "groupTeste1")
                    .Build();

                ITrigger trigger = TriggerBuilder.Create()
                    .WithIdentity("triggerTeste1", "groupTeste1")
                    .StartNow()
                    .WithSimpleSchedule(x => x
                        .WithIntervalInSeconds(5)
                        .RepeatForever())
                    .Build();

                await scheduler.ScheduleJob(job, trigger);

                //infinito
                await Task.Delay(-1);
                await scheduler.Shutdown();

            }
            catch (SchedulerException se)
            {
                Console.WriteLine(se);
            }
        }
        
    }
    
    //Nossa tarefa teste, tem por objetivo printar no console "Hello" a cada 5 segundos
    public class Teste1Job : IJob
    {
        public async Task Execute(IJobExecutionContext context)
        {
            await Console.Out.WriteLineAsync($"Hello! ¬¬ {DateTime.Now.ToString("HH:MM:ss")}");
        }
    }

}
```

Não vou entrar em detalhes sobre o funcionamento do quartz, mas deixo um link de documentação de um mini tutorial: https://www.quartz-scheduler.net/documentation/quartz-3.x/tutorial/index.html pra dar uma luz para quem nunca utilizou esse maravilhoso package.

OBS: No meu github criei algo mais elaborado referente ao scheduler, com arquivo de configuração, parâmetros, etc https://github.com/nandomegaman/HeyTask

Agora, antes se seguir para montar nossa imagem docker, vamos rodar nosso scheduler criado. Volte ao console, dentro do diretório do projeto rode:

```
dotnet run
```

A tarefa exibe um "Hello! ¬¬" no console a cada 5 segundos. Você irá ver algo similar a isso no seu console:

```
Hello! ¬¬ 22:07:30
Hello! ¬¬ 22:07:35
Hello! ¬¬ 22:07:40
Hello! ¬¬ 22:07:45
Hello! ¬¬ 22:07:50
Hello! ¬¬ 22:07:55
Hello! ¬¬ 22:07:00
```

Uhuuuuuuuuuuuu! Agora vamos para a próxima etapa.

**Docker, aí vamos nós!**

- .dockerignore : Arquivo similar ao ".gitignore" que ignora arquivos/diretórios para o caso de empurrarmos nossa imagem para o dockehub (repositório de imagens publicas do docker). Esse arquivo vamos deixar assim: 

```
bin\
obj\
```

- Dockerfile : Este arquivo contém as instruções de build do docker, que inclusive são executadas sequencialmente. Esse arquivo irá ficar da seguinte forma:

```
FROM microsoft/dotnet:2.1-sdk
WORKDIR /app

COPY *.csproj ./
RUN dotnet restore

COPY . ./
RUN dotnet publish -c Release -o out
ENTRYPOINT ["dotnet", "out/NomeDaSuaApp.dll"]
```

OBS: Sobre as instruções acima, está bem explícito a proposta da imagem, isso também é o bacana do docker, essa simplicidade e objetividade! 

Como visto acima, a imagem base que vamos utilizar é da versão .net 2.1. Com essa base vamos rodar nossa aplicação recém criada.

Agora, voltamos para o console, vamos construir nosso container:

```
docker build -t NomeDoSeuContainer .
```

Com o container construído, agora é só executar!

```
docker run NomeDoSeuContainer
```

Aeeeeee! Agora nosso scheduler está rodando em um container docker fofinho. o/

**OBS: Caso você queira enviar sua imagem para o hub do docker basta fazer login:**

```
docker login
```

Em seguida:

```
docker tag NomeDoSeuContainer seuUsuarioDocker/NomeDaSuaImagem
docker push seuUsuarioDocker/NomeDaSuaImagem
```

Logo, chegamos ao fim. o/