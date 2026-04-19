# 2. Конфигурация MediatR

## 2.1 Установка
  
```bash  
# Основной пакет 
dotnet add package MediatR  
```  
  
---  
## 2.2 Базовая регистрация  

### Минимальная конфигурация  

Даная конфигурация подходит для одиночных проектов (minimal API, однопроектные решения), когда все реализации внутри одной сборки
  
```csharp  

var builder = WebApplication.CreateBuilder(args);  
  
// Сканирует сборку, содержащую Program, и регистрирует все handler'ы
builder.Services.AddMediatR(cfg =>   
    cfg.RegisterServicesFromAssemblyContaining<Program>());  
  
var app = builder.Build();  
```  
  
### Регистрация из нескольких сборок  

Если мы используем разделение на сборки (проекты) внутри солюшена (к примеру Clean Architect), MediatR и его Handler'ы будут размазаны по сборкам. Для регистрации необходимо будет указать все сборки, в которых присутсвуют handler'ы. Тут удобнее, как я говорил выше, использовать маркерные классы.

```csharp  
builder.Services.AddMediatR(cfg =>  
	{  
	    // Из конкретных типов маркеров (согласитесь, выглядит аккуратнее, чем код ниже)    
	    cfg.RegisterServicesFromAssemblyContaining<ApplicationMarker>();    
	    cfg.RegisterServicesFromAssemblyContaining<InfrastructureMarker>();        
	    
	    // Или из массива сборок (и тут главное не забыть зарегистрировать handler), но так тоже допустимо и вполне рабочий вариант
	    cfg.RegisterServicesFromAssemblies(
		    typeof(CreateOrderCommand).Assembly,
		    typeof(OrderCreatedEventHandler).Assembly
		);
	}
);  
```  

Так что же такое, класс маркер? Это просто пустой класс, присутсвующий в сборке.
### Класс-маркер для сборки  
  
```csharp  
// В проекте Application  
namespace SuperApp.Application;  
  
/// <summary>  
/// Маркерный класс (для получеия конкретной Assembly. Просто и удобно.)
/// </summary>  
public sealed class ApplicationMarker;  
```  
  
---  
  
## 2.3 Расширенная конфигурация  

Кроме hander'ов, MediatR позволяет сконфигурировать много всего полезного и интересного, покажу на коде:

```csharp  
builder.Services.AddMediatR(cfg =>  
{  
    // 1. Сканирование сборок (подключение handler)
    cfg.RegisterServicesFromAssemblyContaining<ApplicationMarker>();  
    
    // 2. Lifetime для handlers (по умолчанию - Transient)  Расскажу немного подробнее ниже
    cfg.Lifetime = ServiceLifetime.Scoped;  
    
    // 3. Регистрация Pipeline Behaviors (порядок обработки === порядок регистрации!!!)
        cfg.AddBehavior<LoggingBehavior>();           // 1-й в pipeline    
        cfg.AddBehavior<ValidationBehavior>();        // 2-й в pipeline    
        cfg.AddBehavior<TransactionBehavior>();       // 3-й в pipeline  
    
    // 4. Регистрация open-generic behaviors    
    cfg.AddOpenBehavior(typeof(LoggingBehavior<,>));    
    cfg.AddOpenBehavior(typeof(ValidationBehavior<,>));  
    
    // 5. Кастомный NotificationPublisher    
    cfg.NotificationPublisher = new TaskWhenAllPublisher();
    cfg.NotificationPublisherType = typeof(TaskWhenAllPublisher);  
    
    // 6. Автоматическая регистрация пре/пост через open generics    
    cfg.AddOpenRequestPreProcessor(typeof(LoggingPreProcessor<>));    
    cfg.AddOpenRequestPostProcessor(typeof(LoggingPostProcessor<,>));        
    
    // 7. Stream behaviors  
    cfg.AddOpenStreamBehavior(typeof(StreamLoggingBehavior<,>));});  
```  

### Что из этого использовать на старте  
  
Для нового проекта достаточно минимума (1, 2, 4) - остальное добавите по мере необходимости:  
  
```csharp  
builder.Services.AddMediatR(cfg =>  
{  
    cfg.RegisterServicesFromAssemblyContaining<ApplicationMarker>();  // 1. Сборки    
    
    cfg.Lifetime = ServiceLifetime.Scoped;                            // 2. Lifetime    
    
    cfg.AddOpenBehavior(typeof(LoggingBehavior<,>));                  // 4. Логирование    
    cfg.AddOpenBehavior(typeof(ValidationBehavior<,>));               // 4. Валидация});  
```

Пункты 3, 5, 6, 7 подключаются когда реально нужны:

- 3 (конкретный behavior) - крайне редко, когда необходимо обработать конкретный запрос, почти всегда хватает open generic
- 5 (NotificationPublisher) - когда нужна параллельная или кастомная стратегия публикации т обработки (разберем отдельно в следующих разделах)
- 6 (Pre/Post Processors) - когда нужна простая логика до/после без возможности прервать pipeline
- 7 (Stream behaviors) - когда используется `IStreamRequest` для потоковых данных

---

## 2.4 Lifetime - Transient, Scoped, Singleton  
  
```csharp  
// По умолчанию: Transient (новый экземпляр на каждый запрос mediator.Send())  
cfg.Lifetime = ServiceLifetime.Transient;  
  
// Scoped - один экземпляр на scope (обычно HTTP-запрос)  
cfg.Lifetime = ServiceLifetime.Scoped;  
  
// Singleton - один экземпляр на все приложение (ОСТОРОЖНО!)  
cfg.Lifetime = ServiceLifetime.Singleton;  
```  
  
### Когда какой lifetime  
  
| Lifetime      | Когда использовать                                | Риски                 | Особенности                                                  |
| ------------- | ------------------------------------------------- | --------------------- | ------------------------------------------------------------ |
| **Transient** | По умолчанию. Безопасно в 99% случаев             | Чуть больше аллокаций | Безопасно, без побочных эффектов, но с ограничениями         |
| **Scoped**    | Handler использует scoped-зависимости (DbContext) | Надо следить за scope | Совпадает с lifetime большинства зависимостей в ASP.NET Core |
| **Singleton** | Handler без состояния и без scoped-зависимостей   | Captive dependency!   | Максимальная производительность, но опасно (см. ниже)        |
Почему зачастую следует мнять дефолтный Lifetime?

Проблема не в коде MediatR и не в handler'е, а в зависимостях этого самого handler'а. И, основная причина, которая присутсвует в handler'ах на 99% - DbContext.  
  
DbContext в ASP.NET Core регистрируется как Scoped - один экземпляр на HTTP-запрос. Это важно: Change Tracker отслеживает изменения, и все операции в рамках одного запроса должны видеть одни и те же данные.  
  
Transient handler + Scoped DbContext - формально работает.

Когда начинаются проблемы:  
- Background Service, где нет HTTP-scope и вы создаете scope вручную 
- Singleton-сервис, который случайно захватывает Transient-handler, а через него - Scoped DbContext  
- Сложные сценарии с вложенными вызовами и ручным управлением scope  

Вывод:
Scoped handler + Scoped DbContext - всегда безопасно. Handler и DbContext живут в одном ритме. Созданются вместе, умирают вместе. Нет шансов на рассинхронизацию.  

> [!Tip]
> Если в проекте есть DbContext (а он есть почти всегда) - ставьте Scoped и не думайте.  
  
```csharp  
cfg.Lifetime = ServiceLifetime.Scoped;  // безопасный дефолт при наличии DbContext
```  
  
Это не слепое переопределение, а осознанный выбор: убрать целую категорию потенциальных багов.  
  
### Антипаттерн: Captive Dependency  
  
Самая опасная ошибка с lifetime - **Singleton захватывает Scoped-зависимость**:  
  
```csharp  
// Конфигурация:  
cfg.Lifetime = ServiceLifetime.Singleton;  
  
// Handler:  
public class GetOrderHandler : IRequestHandler<GetOrderQuery, OrderDto>  
{  
    // DbContext - scoped, а handler - singleton!    
    // DbContext будет жить вечно → утечки памяти, stale data, конкурентный доступ    
    private readonly AppDbContext _db;
    public GetOrderHandler(IAppDbContext db) => _db = db;  
}  
```  
  
**Что происходит:** Handler создается один раз (Singleton). DbContext передается в конструктор при первом создании и никогда не пересоздается. Все последующие HTTP-запросы используют один и тот же DbContext, следовательно утечки памяти (Change Tracker растет), stale data (данные не обновляются), возникают гонки при параллельных запросах.  
  
```csharp  
// Решение если нужен Singleton: инжектировать IServiceScopeFactory
public class GetOrderHandler : IRequestHandler<GetOrderQuery, OrderDto>  
{  
    private readonly IServiceScopeFactory _scopeFactory;
    public GetOrderHandler(IServiceScopeFactory scopeFactory) => _scopeFactory = scopeFactory;  
  
    public async Task<OrderDto> Handle(GetOrderQuery request, CancellationToken ct)    
    {
	    using var scope = _scopeFactory.CreateScope();
	            var db = scope.ServiceProvider.GetRequiredService<AppDbContext>();        // работаем всегда со свежим DbContext на каждый вызов :)
    }
}  
```  

Но, это решение некий work around и практически не нуэжен.

---  
  
## 2.5 Не знаю, стоит ли описывать регистрацию через плагины?

Возможные практические кейсы?

- SaaS приложение с мульти тенантностью. У каждого свои нюансы бизнес логики...
- Лицензирование приложения. Разные уровни лицензии, разные доступные модули. (некая модульнусть)

Если скажешь да, опишу, но надо бы на примере реального кейса...

==Если есть какие-то советы и best practice для написания мульти тенантных приложений, то надо. Я просто их никогда не писал, и слабо себе представляю что-то кроме разных handler'ов под разные команды==

---  
## 2.6 Условная регистрация (Feature Flags, Environment)  

Тут надо наверное придумать какой то поясняющия текст, но я уже туплю )

```csharp  
builder.Services.AddMediatR(cfg =>  
{  
    cfg.RegisterServicesFromAssemblyContaining<ApplicationMarker>();
 
   // Behavior только для Development, это стандартно, но многие об этом не знают
    if (builder.Environment.IsDevelopment())    
    {
	    cfg.AddOpenBehavior(typeof(DebugLoggingBehavior<,>));    
	}  
	
    // Behavior по feature flag (Частый кейс на включение/отключения кэшированя из конфигурации)
    var featureFlags = builder.Configuration
	    .GetSection("FeatureFlags")
	    .Get<FeatureFlags>();  
	    
    if (featureFlags?.EnableCaching)    
    {
	    cfg.AddOpenBehavior(typeof(CachingBehavior<,>));    
	}  

    // Behavior по окружению, часто подобные фичи задают через Environment    
    if (builder.Configuration.GetValue<bool>("EnableDetailedMetrics"))    
    {
	    cfg.AddOpenBehavior(typeof(MetricsBehavior<,>));
	}
});  
```  
  
---  
  
## 2.7 Кастомный Mediator (Service Resolver)  

Редкая, но все же возможноть, пробовал применять один раз, но отказался по причине запустанности кода и отсутсвия прозрачности решения.
	Кейс: Мультитенантное приложение, нужно обрабатывать по разному запрос на формирвание отчетности для каждого тенанта. 

Для этого нужно перехватить процесс резолвинга, делается это примерно вот так:  
  
```csharp  
public class ReportMediator : IMediator  
{  
    private readonly IMediator _reportMediatR;
    private readonly ILogger<ReportMediator> _logger;
    
    public InstrumentedMediator(
    Mediator reportMediatR,      // конкретная реализация MediatR для обработки запросов отчетности
    ILogger<InstrumentedMediator> logger,
    )
    {
	    _reportMediatR = reportMediatR;
	    _logger = logger;
	}
	  
    public async Task<TResponse> Send<TResponse>(IRequest<TResponse> request, CancellationToken ct = default)
    {
	    _logger.LogDebug("Sending {RequestType}", request.GetType().Name);  
	    return await _reportMediatR.Send(request, ct);  
    }  
    
    public async Task Publish(object notification, CancellationToken ct = default)    
    {
	    await _reportMediatR.Publish(notification, ct);    
	}  
    
    public async Task Publish<TNotification>(TNotification notification, CancellationToken ct = default) where TNotification : INotification    
    {
	    await _reportMediatR.Publish(notification, ct);    
	}  
    
    public Task Send<TRequest>(TRequest request, CancellationToken ct = default) where TRequest : IRequest  
    {
	    return _reportMediatR.Send(request, ct);
    }  

    public IAsyncEnumerable<TResponse> CreateStream<TResponse>(IStreamRequest<TResponse> request, CancellationToken ct = default)    
    {
	    return _reportMediatR.CreateStream(request, ct);
    }  

    public IAsyncEnumerable<object?> CreateStream(object request, CancellationToken ct = default)    
    {
	    return _reportMediatR.CreateStream(request, ct);
    }        
    public Task<TResponse> Send<TResponse>(object request, CancellationToken ct = default)    
    {
	    return _reportMediatR.Send<TResponse>(request, ct);    
	} 
	
    public Task<object?> Send(object request, CancellationToken ct = default)    
    {
	    return _reportMediatR.Send(request, ct);    
	}
}  
  
// Регистрация:  
builder.Services.AddMediatR(cfg =>   
    cfg.RegisterServicesFromAssemblyContaining<ApplicationMarker>());  
  
// Декорируем IMediator  
builder.Services.Decorate<IMediator, ReportMediator>();  // Вот тут возникает необходимости еще в одном пакете - Scrutor: dotnet add package Scrutor  
```  

> [!Tip]
> Мой совет: На практике, не занимайтесь извращением, используйте просто разные handler'ы

---  
  
## 2.8 Типичные ошибки конфигурации  

### Забыли зарегистрировать сборку  
  
```  
System.InvalidOperationException:   
  No service for type 'MediatR.IRequestHandler`2[SupperApp.CreateOrderCommand, System.Guid]'   
  has been registered.  
```  

Частный случай:
```csharp  
// Зарегистрировали только сборку с API, но забыли про сборку с handler'ами :)
Applicationbuilder.Services.AddMediatR(cfg =>   
    cfg.RegisterServicesFromAssemblyContaining<Program>());  // handler'ов тут нет!  
  
// Исправление: Регистрируем сборку, где лежат handler'ы
builder.Services.AddMediatR(cfg =>   
    cfg.RegisterServicesFromAssemblyContaining<ApplicationMarker>());  
```  
  
### Два handler'а на один Request  

В MediatR - Request:Handler это всегда 1:1. Попытка зарегистрировать два handler'а на один request приведет либо InvalidOperationException, если они в одной сборке, либо к неявному поведению в обработке, если они в разных сборках...
  
```csharp  
// Handler 1  
public class CreateOrderHandlerV1 : IRequestHandler<CreateOrderCommand, Guid> { ... }  
  
// Handler 2 (в другой сборке)  
public class CreateOrderHandlerV2 : IRequestHandler<CreateOrderCommand, Guid> { ... }  
  
// Будет использован последний зарегистрированный - неявное поведение и хрен найдешь почему и где!
```  
  
###  Регистрация AddMediatR()  дважды  
в принципе не так сташно, но: Второй вызов затрет behaviors из первого
  
```csharp  
//  
builder.Services.AddMediatR(cfg =>  
{  
    cfg.RegisterServicesFromAssemblyContaining<ApplicationMarker>();    
    cfg.AddOpenBehavior(typeof(LoggingBehavior<,>)); // Тут добавили сквозной LoggingBehavior
});  
  
builder.Services.AddMediatR(cfg =>  
{  
    cfg.RegisterServicesFromAssemblyContaining<InfrastructureMarker>();    // LoggingBehavior - потерян! И потеряны handle'ы из ApplicationMarker
});  

```  
  
---  
  
## 2.9 Best Practices конфигурации  

>[!Tip] Невредные советы

- Используйте ServiceLifetime.Scope, если ваши handler'ы работают с DbContext  
- Один вызов AddMediatR() - соберите всю конфигурацию в одном месте  
- Вынесите конфигурацию в extension method:  
  
```csharp
//На уровне Infrastructure  
public static class MediatrRegistration  
{  
    public static IServiceCollection AddApplicationMediatR(
	    this IServiceCollection services, 
	    IConfiguration configuration,  
        IHostEnvironment environment)    
    {
	    services.AddMediatR(cfg =>
		    {
		    // Assemblies            
		    cfg.RegisterServicesFromAssemblyContaining<ApplicationMarker>();  
            // Lifetime
            cfg.Lifetime = ServiceLifetime.Scoped;  
            // Pipeline (порядок важен!)
            cfg.AddOpenBehavior(typeof(UnhandledExceptionBehavior<,>));            
            cfg.AddOpenBehavior(typeof(LoggingBehavior<,>));
            cfg.AddOpenBehavior(typeof(ValidationBehavior<,>));            

        return services;    
    }
}  
  
// Уже в API:  
builder.Services.AddApplicationMediatR(builder.Configuration, builder.Environment);  
```  

Это конфигурационное продолжение [[01-introduction]] внутри заметок по [[S&P Digital & УБО Софт]].
