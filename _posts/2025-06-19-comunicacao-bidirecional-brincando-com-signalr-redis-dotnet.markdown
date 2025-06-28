---
layout: post
published: true
title:  "Comunicação bidirecional com SignalR + Redis (pub/sub) + Worker + Webhook + Webapp em .NET"
date:   2025-06-19 17:00:00
tags: [dev, dotnet, c#, code, signalR, redis]
---

Olá meus caros, de vez em quando apareço por aqui... haha

Bem, vamos lá. Hoje vou exemplificar de forma prática uma simples comunicação bidirecional com uma mini aplicação que fiz em .NET usando SinalR + Redis (pub/sub). A aplicação também conta com <b>worker</b> que irá notificar o <b>webhook</b> sobre determinado evento ocorrido em qualquer contexto aleatório. Na outra ponta há uma aplicação web que irá receber e enviar informações das "manipulações" em tela.

A imagem abaixo representa o fluxo explanado acima.

![YZCollab](/assets/images/posts/2025/06/yzcollab.png)

Detalhado o escopo, criei 3 aplicações: CliWeb, CliWorker e Srv.

- <b>CliWorker</b> é um worker que fica batendo no webhook a cada minuto notificando algo.
- <b>CliWeb</b> é uma SPA em que o usuário abre a página, digita o nome e fica "monitorando" as atividades. Ao entrar/sair também notifica demais usuários.
- <b>Srv</b> é o "core" da parada toda, nessa aplicação há o Hub para comunicação bidirecional e também um endpoint para o contexo de hook.

### Redis - Pub/Sub

O Pub/Sub do Redis é um mecanismo simples de notificações em tempo real, onde publicadores enviam mensagens para canais, e assinantes destes, recebem as mensagens.

Mais detalhes aqui na <a target="_blank" href="https://redis.io/docs/latest/develop/interact/pubsub/">documentação oficial</a>.

### SignalR

SignalR é uma biblioteca nativa .NET que facilita a comunicação em tempo real entre servidor e cliente, que permite atualizações instantâneas do servidor para o cliente.

Geralmente utiliza websockets quando disponível, mas pode alternar automaticamente para long polling ou server-sent events se necessário (fallback automático).

<a target="_blank" href="https://learn.microsoft.com/en-us/aspnet/signalr/overview/getting-started/introduction-to-signalr">Aqui</a> tem tudo sobre.

### Show me the code

Ok, vamos começar criando uma aplicação <b>WorkerService</b> e basicamente substituir a classe padrão <b>Worker.cs</b> pela abaixo.

```cs
public class Worker : BackgroundService
{
    private readonly ILogger<Worker> _logger;
    private readonly IConfiguration _config;

    public Worker(ILogger<Worker> logger, IConfiguration config)
    {
        _logger = logger;
        _config = config;
    }

    private record HookRequestDTO(string Message);

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        var httpClient = new HttpClient();
        var urlHook = _config.GetValue<string>("UrlHook"); //pega rota do appsettings.json

        while (!stoppingToken.IsCancellationRequested)
        {
            var msg = new HookRequestDTO($"Serviço s-{new Random().Next()} executando às: {DateTimeOffset.Now:g}");
            await httpClient.PostAsJsonAsync(urlHook, msg, stoppingToken);

            _logger.LogInformation(msg.Message);
            await Task.Delay(15000, stoppingToken);
        }
    }
}
```

Perfeito, aplicação 1 (CliWorker) tá ok. 

Agora vamos para aplicação 2 (CliWeb), que é uma aplicação <b>AspNetCore Razor Pages</b>. Para instalar o SignalR é usado libman, logo,basta criar o arquivo libman.json na raiz do projeto.

```json
{
  "version": "1.0",
  "defaultProvider": "unpkg",
  "libraries": [
    {
      "library": "@microsoft/signalr@latest",
      "destination": "wwwroot/lib/js/signalr/",
      "files": [
        "dist/browser/signalr.js",
        "dist/browser/signalr.min.js"
      ]
    }
  ]
}
```

Feito isso, criar um arquivo chamado app.js dentro da pasta padrão (wwwroot/js) e colocar código abaixo.

```js
const app = (function () {

    const modal = document.getElementById("z-modal");
    const form = modal.querySelector("form");
    const userInput = form.querySelector("input[name='user']");
    const btnSubmit = form.querySelector("button[name='submit']");
    const keySubmit = 13;

    function submit(event) {
        event.preventDefault();
        modal.style.display = "none";

        hub.init(userInput.value);
    }

    form.addEventListener("submit", function (event) {
        event.preventDefault();
    });

    btnSubmit.addEventListener("click", function (event) {
        submit(event);
    });

    userInput.addEventListener("keydown", function (event) {
        if (event.keyCode === keySubmit) {
            submit(event);

            return false;
        }
    });

    function init() {
        modal.style.display = "block";
    }

    return {
        init: init
    };
})();


const hub = (function () {

    function registerEvent(section, message) {
        const li = document.createElement("li");
        li.innerHTML = `${message}`;
        section.prepend(li);
    }

    async function start(user) {
        try {
           
            const urlHub = document.querySelector("main").getAttribute("data-url-hub");
            const logsSectionList = document.getElementById("z-logs").querySelector("ul");
            const usersSectionList = document.getElementById("z-users").querySelector("ul");

            const connection = new signalR.HubConnectionBuilder()
                .withUrl(`${urlHub}?user=${user}`)
                .configureLogging(signalR.LogLevel.Information)
                .build();

            console.log(`connectionId: ${connection.connectionId}`);

            connection.on("RegisterLog", (message) => {
                registerEvent(logsSectionList, message);
            });

            connection.on("RegisterUser", (message) => {
                registerEvent(usersSectionList, message);
            });

            connection.onclose(async () => {
                await start(user);
            });

            await connection.start();

            console.log(`connectionId: ${connection.connectionId}`);
        } catch (err) {
            console.log(err);
            setTimeout(start, 5000);
        }
    };

    function init(user) {
        start(user);
    }

    return {
        init: init
    };
})();

app.init();
```

 Repare nos eventos RegisterLog/RegisterUser, mais a frente vamos falar deles.

 Agora vamos substituir o arquivo padrão chamado <b>Index.cshtml</b>.

```html
@page
@model IndexModel
@{
    ViewData["Title"] = "Playground";
}
<main data-url-hub="@Model.Config["UrlHub"]">
    <div id="z-modal">
        <div class="modal-content">
            <form>
                <h2>YZCollab</h2>
                <input type="text" name="user" placeholder="Qual seu nome?" maxlength="20" required>
                <button type="button" name="submit">Entrar</button>
            </form>
        </div>
    </div>
    <section id="z-container">
        <div id="z-logs">
            <h3>Activities</h3>
            <ul>
            </ul>
        </div>
        <div id="z-users">
            <h4>Users</h4>
            <ul>
            </ul>
        </div>
    </section>
</main>
<footer>
</footer>
```

Feito isso, vamos adicionar referências no arquivo <b>__Layout.cshtml</b>.

```html
...
<script src="~/lib/js/signalr/dist/browser/signalr.min.js"></script>
<script src="~/js/app.js" asp-append-version="true"></script>
```

Tem estilização também... mas isso foge da idéia aqui, logo, aplicação 2 ok! =)

Agora vamos criar a terceira aplicação (Srv) que é uma aplicação <b>AspNetCore Api</b>.

As dependências são basicamente estas:

```html
<ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Versioning" Version="5.1.0" />
	<PackageReference Include="Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer" Version="5.1.0" />
	<PackageReference Include="Microsoft.AspNetCore.SignalR.Common" Version="9.0.6" />
	<PackageReference Include="Microsoft.AspNetCore.SignalR.Protocols.Json" Version="9.0.6" />
	<PackageReference Include="Microsoft.AspNetCore.SignalR.StackExchangeRedis" Version="9.0.6" />
	<PackageReference Include="Swashbuckle.AspNetCore" Version="9.0.1" />		
</ItemGroup>
```

Agora vamos criar 3 arquivos: MessageHub.cs, IMessageHubService.cs e MessageHubService.cs.

```cs
//pasta Hubs

public class MessageHub : Hub
{

    private string? GetUserName() => $"{Context.GetHttpContext()?.Request?.Query["user"]} ({Context.ConnectionId})";

    public override Task OnConnectedAsync()
    {
        return Clients.All.SendAsync("RegisterUser", $"<i>{GetUserName()}</i> <strong>entrou.</strong>");
    }

    public override Task OnDisconnectedAsync(Exception? exception)
    {
        return Clients.All.SendAsync("RegisterUser", $"<i>{GetUserName()}</i> <strong>saiu.</strong>.");
    }

    public Task RegisterLog(string message)
    {
        return Clients.All.SendAsync("RegisterLog", $"{message}");
    }
}

//pasta Services

public interface IMessageHubService
{
    Task RegisterLogAsync(string message); 
}

public class MessageHubService : IMessageHubService
{
    private readonly IHubContext<MessageHub> _hubContext;

    public MessageHubService(IHubContext<MessageHub> hubContext)
    {
        _hubContext = hubContext;
    }

    public async Task RegisterLogAsync(string message) => await _hubContext.Clients.All.SendAsync("RegisterLog", message);        
}
```

A classe MessageHub.cs é quem faz a magia acontecer, ela que comunica com os eventos RegisterLog/RegisterUser citados previamente.

Feito isso, vamos criar um arquivo chamado <b>HookController.cs</b> dentro da pasta Controllers/v1. Essa classe terá o endpoint para o hook.

```cs
[ApiController]
[ApiVersion("1.0")]
[Route("v{version:apiVersion}/hook")]
public class HookController : ControllerBase
{
    private readonly IMessageHubService _messageHubService;

    public HookController(IMessageHubService messageHubService) {
        _messageHubService = messageHubService;
    }

    public record HookRequestDTO(string Message);
    public record HookResponseDTO(int Code);

    [HttpPost]
    [ProducesResponseType(typeof(HookResponseDTO), StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> Post(HookRequestDTO request)
    {
        await _messageHubService.RegisterLogAsync($"{request.Message}");

        var response = new HookResponseDTO(new Random().Next());
            
        return Ok(response);
    }
}
```

Sempre que baterem nesse endpoint, uma notificação é enviada para o Hub.

OBS: Poderia ter usado minimal apis (que é o que mais uso atualmente), mas isso poderia ser assunto pra um outro post talvez, vamos manter o tradicional.

Agora vamos configurar o arquivo <b>Program.cs</b>.

```cs
...
builder.Services.AddTransient<IMessageHubService, MessageHubService>();

builder.Services
    .AddSignalR()
    .AddJsonProtocol()
    .AddStackExchangeRedis($"{configuration.GetValue<string>("Redis")}", redisOptions =>
 {
     redisOptions.ConnectionFactory = async writer =>
     {
         var config = new ConfigurationOptions
         {
             AbortOnConnectFail = false
         };

         config.EndPoints.Add(IPAddress.Loopback, 0);
         config.SetDefaultPorts();

         var connection = await ConnectionMultiplexer.ConnectAsync(config, writer);

         connection.ConnectionFailed += (_, e) =>
         {
             Console.WriteLine("Connection to Redis failed.");
         };

         if (!connection.IsConnected)
         {
             Console.WriteLine("Did not connect to Redis.");
         }

         return connection;
     };
});

...
app.MapHub<MessageHub>("/hub");
```

Agora sim! A estrutura do projeto vai ficar assim.

![YZCollab](/assets/images/posts/2025/06/estrutura.png)

### Só brincar!

Depois de tudo certo basta rodar os 3 projetos. 

Abaixo a tela da aplicação CliWeb.

![YZCollab](/assets/images/posts/2025/06/playground.png)

No Redis as notificações.

![YZCollab](/assets/images/posts/2025/06/redispubsub.png)

### Acho que era isso

Aqui no meu  <a target="_blank" href="https://github.com/libnando/YZCollab">github</a> tem o projeto todo.

Até mais. o/
