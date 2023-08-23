---
layout: page
title: Errors
permalink: /errors/
---

# этот фай укаже на основные ошибки проекта

1.  В коде нет общей привычки наименования полей и форматирования кода. 
    Советую почитать: [C# Coding Standards and Naming Conventions](https://github.com/ktaranov/naming-convention/blob/master/C%23%20Coding%20Standards%20and%20Naming%20Conventions.md) 
    или хотя бы [Common C# code conventions](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/coding-conventions).

2.  В коде полно класов делающих все, например класс AIThing. 
    Вместо этого класса лучше всего декомпозировать AIThing на отдельные класы,
    на пример всю работу с YoutubeApi можно перенести в отдельный класс YoutubeService,
    а и вместо парсинга HTML можно пользоваться библиотекой от Google. 
    Советую прочитать про принцыпы SOLID.

3.  В коде много логики завязанной на строковых литиралах.
    В место использования строковых литиралов можно использовать привязку к типам.
    Пояснение, логики по типу: 
    ```CSharp 
    private void FindAndAssignCharacterInstances()
    {
        characters = new GameObject[8];
        characters[0] = GameObject.Find("mrkrabs(Clone)");
        characters[1] = GameObject.Find("squidward(Clone)");
        characters[2] = GameObject.Find("spongebob(Clone)");
        characters[3] = GameObject.Find("gary(Clone)");
        characters[4] = GameObject.Find("larry(Clone)");
        characters[5] = GameObject.Find("sandy(Clone)");
        characters[6] = GameObject.Find("plankton(Clone)");
        characters[7] = GameObject.Find("patrick(Clone)");
        // Без коментариев, я думаю ты понел
    }
    ```
    или
    ```CSharp
    // этот код вообще вне Unity не запуститься, если не откопировать 
    // blacklist.json и topics.json по таким же локальным путям
    private void UpdateBlacklist(List<string> blacklist, string topic, Queue<string> topics)
    {
        string blacklistPath = $"{Environment.CurrentDirectory}\\Assets\\Scripts\\blacklist.json";
        // строчка выше вообще константой должна быть и вместо 
        // Enviroment.CurrentDirectory можно написать так
        // "./Asests/Scripts/blacklist.json"
        if (!blacklist.Contains(topic))
        {
            blacklist.Add(topic);
            File.WriteAllText(blacklistPath, JsonConvert.SerializeObject(blacklist));
        }

        // Write the remaining topics back to the topics file
        File.WriteAllText($"{Environment.CurrentDirectory}\\Assets\\Scripts\\topics.json", JsonConvert.SerializeObject(topics.ToList()));
        // тоже что и выше
    }
    ``` 

4.  Все классы унаследованы от MonoBehaviour. 
    Bобщем это не всегда ошибка, но это зависит от выстроенной архитектуры.
    На будущие можно делать что-то такое так:
    ```CSharp
    public class StartUp : MonoBehaviour
    {
        // This is Dependency Injection or DI for short. 
        // В Unity есть своя реализация DI, но я хз как ей ползоваться
        private IServiceCollection _services;

        public StartUp()
        {
            _services = new ServiceCollection();
        }

        private void Start()
        {
            // some others invokes or adding IConfiguration to the Container

            ConfigureServices();
        }

        private void Distroy()
        {
            // code
        } 

        public ConfigureServices()
        {
            _services.AddHostedService<YoutubeService>(); 
            // мне пох мне лень я просто HostedService ебану
            // предположим что там какое-то постоянное подключение будет
            // some other configuration
        }
    }

    public class YoutubeService : IHostedService
    {
        private readonly ILogger<YoutubeService> _logger;
        private readonly SecureString _apiToken;
        private readonly YoutubeApi _youtubeApi;

        private Thread _watchUpdatesThread;
        private CancellationTokenSource _tokenSource;

        public YoutubeService(
            ILogger<YoutubeService> logger,
            IYoutubeApiConfiguration configuration) // зависимости
        {
            _logger = logger;
            _apiToken = configuration.Token;
            _youtubeApi = new YoutubeApi();
        }

        public async Task StartService(CancellationToken token)
        {
            _logger.LogInformatino("Service Starting...");
            _tokenSource = CancellationTokenSource
                .CreateLinkedTokenSource(token)

            await _youtubeApi.AuthohorizeAsync(_apiToken);

            _watchUpdatesThread = new Thread(
                t => WatchUpdates((CancellationToken)t));

            _watchUpdatesThread.Start(_tokenSource.Token)
        }

        public Task StopService(CancellationToken token)
        {
            _logger.LogInformation("Service Stoping...")
            _tokenSource.Cancel();
            _watchupdatesThread.Join(); 

            return Task.ComplitedTask;
        }

        public void WatchUpdates(CancellationToken token)
        {
            try 
            {
                while (!token.IsCancellationRequested)
                {
                   // some code 
                }
            }
            try (Exception ex)
            {
                _logger.Fatal(ex, "Service Stoped."); // maybe not Fatal just Error
            }
        }
    }
    ```
    Вобщем просто научись делить класы по их обязоностям, использовать DI 
    (и не надо создавать классы с больше чем 3-4 зависимостями в идеале 1)
    и самое главное не наследуйся от MonoBehaviour когда тебе его логика нах ненужна
    Можно кста и без DI справится. 

5. Смешал View и Model в одном класе (так многие делают по первости). 
   читай про MVC, или MVP, или MVVM, ну или даже про PIDOR 
   (чекни что это за паттерт, но не используй)

# немного хорошего 

1. Ты используешь SerializedField и RequiredComponent это заебись так и надо. 
   (жалко, что не везде, но я видел код и по хуже)

2. Ты 

# еще неболшые советы 

почитай про синтаксис C# 9.0
на пример можно писать так:

```CSharp
using FileStream stream = File.OpenFile(path, FileMode.OpenOrCreate);

// some code
```
вместо 
```CSharp
using (FileStream stream = File.OpenFile(path, FileMode.OpenOrCreate))
{
    // some code
}
```