---
layout: page
title: 
permalink: /Errors/
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

5.  Смешал View и Model в одном класе (так многие делают по первости). 
    читай про MVC, или MVP, или MVVM, ну или даже про PIDOR 
    (чекни что это за паттерт, но не используй)

6.  Ты вообще не знаешь как ползоваться abstract. какой смысл создавать 
    абстрактный класс, если у него в наследниках один единственый ОБЩИЙ класс
    который вешается куда не лень.
    кроче 1 класс Character как говориться make no sense его просто надо разделить на 
    мелки классы на пример: 
    InteractionHandler(/Controller зависит от предпочтений в архитектуре), IInteractable, 
    Mover(или MoveController и MovableAgent и какя-то View которая за анимашки отвечает)
    Observer(или что то в этом духе либо можно кста RotationHandler) и тд 
    ну и можно сделать класс Character но он теперь будет выступать как Controller

7.  Не тащи в зависимости боьше чем надо. Если тебе просто нужна позиция обекта в скрипте 
    то тащи только Transform, а не весь GameObject, если тебе нужно задать позицию для движения
    в скрипте то не GameObject, а создай Mover(с методом MoveToPosition(Vector3 position)) 
    и тащи его в скрипт, и тд.

# немного хорошего 

1. Ты используешь SerializedField и RequiredComponent это заебись так и надо. 
   (жалко, что не везде, но я видел код и по хуже)

2. Для 4 проекта это замечательный результат.

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

не ну tcp ip чтобы просто даные с python скрипта получить. 
я такое в первые вижу вы конешь изврат полный
можно просто sub процесс запустить, а данные кидать через
[standard I/O](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process.standardoutput?view=net-7.0)
в python это просто ```print(/*some data*/)``` 
а в шарпе запуск процесса и получения данных через Stream

кста про Stream почитай и про StreamReader в частности.

# о переносе кода на Godot 
эм легче просто все с нуля написать учась на ошибках.
знаеш не бойся писать говно, и небойся удалять весь код который написал.

а если все таки хочешь именно перенести, то можешь попробывать перекинуть весь код в Godot проект и заменить все GameObject на Node и все что ошибкой подчеркивается тоже просто потихоньку заминяй аналогами которые найдешь.

# что почитать и интересные Links

[DotNext](https://www.youtube.com/@DotNextConf) русская ежигодная конфиренция по 
програмированию там дохуя про C# и про архитектуры и куча кеков (по GameDev там мб есть, 
но тебе все равно будет полезно, ведь програмирования на C# не отличаеться от платформы).

книга кодят там на C#: [паттерны проектирования на платформе .NET](https://vk.com/doc44301783_411162088?hash=lqZ9j2wAQg9Wa5UFsY6uCP5z2bi8AqEPw2FxZl2U9JD&dl=b7s5Cn08pjflEfSX0qBZ4t4cczG3Z9OdUnzD5CjbBBH)

Мб еще ченибуть по Unity, Godot или про GameDev в общем.

если по c# то можешь чекнуть еще аот этот видос: 
[secret link](https://youtu.be/w8rRhAup4kg?t=147) вообщем видос норм. 
Он разделен на секции просто, что знаешь то скипаешь.

по архитектуре можеш чекнуть репозиторий 
[osu lazer](https://github.com/ppy/osu)
и 
[osu-framework](https://github.com/ppy/osu-framework).
это простая(отнасительно) игра написаная без движка. Можешь посмотреть 
как там происходит деление на классы.

и самое главное: 
[secret link](https://learn.microsoft.com/en-us/dotnet/csharp/), 
[another secret link](https://learn.microsoft.com/en-us/dotnet/fundamentals/) - 
это все мои иконы в мире програмирования, ну там еще куча есть, но тебе они не нужны.