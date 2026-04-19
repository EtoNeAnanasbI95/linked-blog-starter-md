# Введение и архитектура

## 1.1 Что такое MediatR  
  
MediatR - это open-source библиотека для .NET, реализующая паттерн [Mediator]() (посредник).
  
Ключевая идея: вместо прямого вызова зависимости, компонент отправляет сообщение (request), а медиатор находит обработчик и передает ему это сообщение.  
  
```  
[Controller]--Send(command) --> [IMediator]--resolve --> [CommandHandler]  
```  
  
MediatR не является:  
- Message broker (как RabbitMQ, Kafka)  
- Event bus (как MassTransit)  
- Service bus  
  
MediatR - это `in-process` медиатор. Все вызовы происходят в рамках одного процесса.  

> [!TIP]
> MediatR реализует паттерн CQRS?
> Ответ: Нет.  
> 
> MediatR - это "Mediator pattern" (маршрутизация сообщений). CQRS - это архитектурный паттерн (разделение моделей записи и чтения). 
> 
> MediatR помогает реализовать CQRS (Commands и Queries как отдельные Request-типы), но сам по себе CQRS не реализует. Можно использовать MediatR без CQRS, а CQRS без MediatR.
> 
> Настоящий CQRS подразумевает:  
>- Разные модели данных для записи и чтения (возможно, разные БД)  
>- Event Sourcing (опционально)  
>- Eventual consistency между write и read моделью  
>
>MediatR с  интерфейсами `ICommand<T>`/`IQuery<T>` - это некий 'CQRS-lite': разделение на уровне кода, но не на уровне хранилища.

---

Немного о паттернах...
## 1.2   Паттерн Mediator (GoF) vs MediatR
### Классический Mediator (GoF)  
  
```csharp  
// GoF: объекты знают о медиаторе, медиатор знает обо всех участниках  
public interface IChatMediator  
{  
    void SendMessage(string message, User sender);    
    void AddUser(User user);
}  
  
public class ChatRoom : IChatMediator  
{  
    private readonly List<User> _users = new();  
    
    public void AddUser(User user) => _users.Add(user);  
    
    public void SendMessage(string message, User sender)    
    {
        foreach (var user in _users.Where(u => u != sender))
	        user.Receive(message);    
	}
}  
```  
  
### MediatR  
  
MediatR не реализует классический Mediator, а скорее комбинацию:  
- Mediator - единая точка маршрутизации  
- Command - сообщения как объекты  
- Observer - notifications как события  
  
```csharp  
// MediatR: отправитель не знает получателя. Медиатор резолвит handler из DI. Комамнда:  
public record SendMessageCommand(string message, string senderId)   
    : IRequest<MessageResult>;  
  
// обработчик (Application уровень)
public class CSendMessageCommandHandler : IRequestHandler<SendMessageCommand, MessageResult>  
{  
    public async Task<MessageResult> Handle(SendMessageCommand request, CancellationToken ct)    
    {
	    // бизнес-логика отправки сообщения
	    return new MessageResult(messageId);    
	}
}  
  
// Вызов:  
var result = await mediator.Send(new SendMessageCommand("Привет, засранцы!", "user-12345"));  
```  
  
---
## 1.3 CQRS и место MediatR в архитектуре  
  
CQRS (Command Query Responsibility Segregation) - разделение операций записи (Commands) и чтения (Queries).  
  
MediatR идеально ложится на CQRS:  
  
```  
┌─────────────────────────────────────────────────┐  
│                   API Layer                     │  
│  POST /orders → Send(CreateOrderCommand)        │  
│  GET /orders  → Send(GetOrdersQuery)            │  
└──────────────────────┬──────────────────────────┘  
                       │ IMediator.Send()           
┌─────────────────────────────────────────────────┐  
│              Application Layer                  │  
│  ┌─────────────────┐  ┌──────────────────────┐  │  
│  │    Commands     │  │      Queries         │  │  
│  │  CreateOrder    │  │   GetOrders          │  │  
│  │  CancelOrder    │  │   GetOrderById       │  │  
│  └────────┬────────┘  └──────────┬───────────┘  │  
│           │ Handlers             │ Handlers     │  
│  ┌────────-────────┐  ┌────────────────────-─┐  │  
│  │ Write to DB     │  │  Read from ReadModel │  │  
│  │ Publish events  │  │  Return DTO          │  │  
│  └─────────────────┘  └──────────────────────┘  │  
└─────────────────────────────────────────────────┘  
```  
  
### Разделение Commands и Queries - интерфейсы  
  
```csharp  
// интерфейсы для CQRS  
public interface ICommand<TResponse> : IRequest<TResponse>;  
public interface IQuery<TResponse> : IRequest<TResponse>;  
  
// Использование  
public record CreateOrderCommand(string CustomerId) : ICommand<Guid>;  
public record GetOrderQuery(Guid OrderId) : IQuery<OrderDto>;  
```  
  
Это позволяет применять разные Pipeline Behaviors к командам и запросам (например, транзакция - только для команд).  
  
---  
## 1.4 Когда использовать MediatR  
  
###  Использовать, если:  
  
- Проект среднего/большого размера - десятки use-case'ов (удобно когда несколько разработчиков и у каждого свой кейс)
- Нужна CQRS - четкое разделение команд и запросов
- Нужна сквозная функциональность (обработка всех запросов в Pipeline) - логирование, валидация, транзакции через Pipeline Behaviors  
- Clean Architecture - MediatR отлично разделяет слои
- Нужна единообразная структура - каждый use-case = Request + Handler  
  
###  Не использовать, если:  
  
- Маленький проект / CRUD (до 10 сущностей CRUD без сложной логики) - MediatR добавит оверхед без пользы  
- Команда не понимаешь паттерн - MediatR не спасет от плохой архитектуры  
- Нужна inter-process коммуникация - используйте брокеры сообщений. 
- Один handler вызывает другой через mediator - это антипаттерн (см. ниже)  
  
---
 
## 1.5 Сравнение с альтернативами  (запросил у ИИ)
  
| Критерий           | MediatR  | Wolverine      | Brighter     | Ручная реализация |
| ------------------ | -------- | -------------- | ------------ | ----------------- |
| Сложность входа    | Низкая   | Средняя        | Высокая      | Зависит от задачи |
| In-process         | ✅        | ✅              | ✅            | ✅                 |
| Out-of-process     | ❌        | ✅ (встроен)    | ✅ (встроен)  | ❌                 |
| Pipeline Behaviors | ✅        | ✅ (middleware) | ✅ (policies) | Ручная реализация |
| Community          | Огромное | Растущее       | Среднее      | —                 |
| Производительность | Высокая  | Очень высокая  | Высокая      | Максимальная      |
| Source Generator   | ❌        | ✅              | ❌            | —                 |
### Ручная реализация (когда MediatR - overkill)  
  
```csharp  
// Простой медиатор за 20 строк (для микропроектов, реализовывал ранее)  
public interface IRequestHandler<in TRequest, TResponse>  
{  
    Task<TResponse> Handle(TRequest request, CancellationToken ct);
}  
  
public class SimpleMediator(IServiceProvider provider)  
{  
    public Task<TResponse> Send<TResponse>(IRequest<TResponse> request, CancellationToken ct = default)    
    {
	    var handlerType = typeof(IRequestHandler<,>)
		    .MakeGenericType(request.GetType(), typeof(TResponse));                
		dynamic handler = provider.GetRequiredService(handlerType);  
        return handler.Handle((dynamic)request, ct);
    }
}  
```  
  
Из минусов (как потом увидим, у MediatR это огромные плюсы):
- нет pipeline,
- нет notifications,
- нет stream requests,

---  
  
## 1.6 Антипаттерны  

> [!Danger] Как не делать!
  ### Антипаттерн: Handler вызывает Mediator  
  
```csharp  
// handler вызывает другой handler через mediatorpublic class CreateOrderHandler : IRequestHandler<CreateOrderCommand, Guid>  
{  
    private readonly IMediator _mediator;  
    public CreateOrderHandler(IMediator mediator) => _mediator = mediator;  
    public async Task<Guid> Handle(CreateOrderCommand request, CancellationToken ct)    
    {
        var orderId = Guid.NewGuid();        // ... создали заказ  
        // Вызов другого command через mediator внутри handler        
        await _mediator.Send(new SendEmailCommand(request.CustomerId, orderId), ct);        
        return orderId;    
    }
}  
```
  
> [!Tip] 
 Скрытая зависимость - невозможно понять flow. 
 Повторное прохождение pipeline.  

Как решать подобные задачи правильно:

```csharp  
// используем Notification для side-effects и обработаем отдельно сообщение
public class CreateOrderHandler : IRequestHandler<CreateOrderCommand, Guid>  
{  
    private readonly IPublisher _publisher;
    private readonly IOrderRepository _repo;  
    
    public CreateOrderHandler(IOrderRepository repo, IPublisher publisher)    
    {
	    _repo = repo;
	    _publisher = publisher;    
	}  
    
    public async Task<Guid> Handle(CreateOrderCommand request, CancellationToken ct)    
    {
	    var order = Order.Create(request.CustomerId, request.Items);
	    await _repo.SaveAsync(order, ct);  
        // Публикуем событие, а не вызываем другой command
        await _publisher.Publish(new OrderCreatedNotification(order.Id), ct);
        return order.Id;
	}
}  
```  
  
### Антипаттерн: MediatR повсюду  
  
```csharp  
// MediatR ради MediatR - даже для вызова простого сервиса
public record GetCurrentTimeQuery : IRequest<DateTime>;  
  
public class GetCurrentTimeHandler : IRequestHandler<GetCurrentTimeQuery, DateTime>  
{  
    public Task<DateTime> Handle(GetCurrentTimeQuery request, CancellationToken ct) => Task.FromResult(DateTime.UtcNow); // Зачем козе баян?
}  
```  

> [!Tip] 
Если handler - одна строка и не нужен pipeline, просто вызовите сервис напрямую.  
  
### Антипаттерн: God Request  
  
```csharp  
// один запрос делает все
public record ProcessEverythingCommand(
	Customer Customer, 
	Order Order, 
	Payment Payment, 
	Shipping Shipping, 
	Notification Notification) : IRequest<ProcessResult>;  
```  
  
> [!Tip]  
> Один Request === одна бизнес-операция.


## 1.7 Невредные советы: 

>[!Success] Некие Bset Practices

- Используйте record types для Request'ов - неизменяемость, value equality  
- Разделяйте Commands и Queries через маркерные интерфейсы  
- Один Handler - одна ответственность (Single Responsibility)  
- Pipeline Behaviors для cross-cutting concerns - не дублируйте код в handlers  
- Notifications для side-effects - не вызывайте Send() внутри handler'а  
- Тестируйте handlers изолированно - они обычные классы с конструкторными зависимостями

Этот вводный док относится к [[S&P Digital & УБО Софт]] и продолжается более прикладной заметкой [[02-configuration]].
