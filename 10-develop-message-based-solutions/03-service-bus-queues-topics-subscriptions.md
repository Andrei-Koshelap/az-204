# Service Bus Queues, Topics, and Subscriptions

## Service Bus Queues

**Queues** обеспечивают модель доставки сообщений **point-to-point** с использованием паттерна **competing consumers**.  
Сообщения сохраняются в очереди до тех пор, пока получатель не извлечёт и не обработает их.

---

## Архитектура очереди

Типичный поток работы:

1. **Producer** отправляет сообщение в очередь
2. Сообщение сохраняется в Service Bus
3. Один из **Consumers** получает сообщение
4. После успешной обработки сообщение удаляется

---

### Ключевые свойства архитектуры

- Одно сообщение обрабатывается только одним consumer’ом
- Несколько consumers могут работать параллельно (масштабирование)
- Очередь выступает буфером при скачках нагрузки
- Поддерживается надёжная доставка (at-least-once)

---

## Когда использовать очереди

- Асинхронная обработка задач
- Разгрузка веб-приложений
- Микросервисное взаимодействие (1:1)
- Фоновые worker-процессы
- Batch-обработка

---

## Важно для AZ-204

- Queue = 1:1 взаимодействие
- FIFO гарантируется только при использовании Sessions
- Подходит для decoupling и load leveling
- Для 1:N используйте Topics

Главная идея:  
Queue — это базовый строительный блок надёжной асинхронной архитектуры.

```
Senders (Multiple)              Queue                Receivers (Multiple)
┌──────────┐                  ┌───────┐            ┌──────────┐
│ Sender 1 │ ───────────────> │ Msg 1 │ ────┐      │Receiver 1│
└──────────┘                  │ Msg 2 │     │  ───>└──────────┘
┌──────────┐                  │ Msg 3 │     └─┐    ┌──────────┐
│ Sender 2 │ ───────────────> │ Msg 4 │       ───> │Receiver 2│
└──────────┘                  │ Msg 5 │       ┌──> └──────────┘
┌──────────┐                  │  ...  │       │    ┌──────────┐
│ Sender 3 │ ───────────────> │       │ ──────┘    │Receiver 3│
└──────────┘                  └───────┘            └──────────┘

Key Characteristics:
• One message delivered to ONE receiver only
• Multiple senders can send to same queue
• Multiple receivers compete for messages (load balancing)
• Messages persist until successfully processed
```

### Основные возможности

| Возможность | Описание |
|-------------|----------|
| **FIFO Ordering** | Гарантируется при включённых sessions (внутри одной сессии) |
| **Competing Consumers** | Несколько получателей могут параллельно обрабатывать сообщения |
| **Load Balancing** | Равномерное распределение нагрузки между получателями |
| **Load Leveling** | Сглаживание резких пиков нагрузки |
| **Temporal Decoupling** | Отправитель и получатель могут работать независимо по времени |
| **Message Persistence** | Сообщения надёжно сохраняются до обработки |

---

## Пояснения

- **FIFO** работает только при использовании Sessions (через SessionId).
- **Competing Consumers** позволяют масштабировать обработку горизонтально.
- **Load Leveling** делает очередь буфером при всплесках трафика.
- **Temporal Decoupling** снижает связанность между сервисами.
- **Message Persistence** защищает от потери данных при сбоях.

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:
- требуется надёжность доставки
- нужно масштабирование через несколько consumers
- важна асинхронность

→ Service Bus Queue подходит идеально.

Если требуется 1:N или фильтрация → используйте Topics.

### Creating a Queue

```bash
# Create queue with Azure CLI
az servicebus queue create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --name myQueue \
  --max-size 1024 \
  --default-message-time-to-live P14D \
  --lock-duration PT30S \
  --max-delivery-count 10 \
  --enable-duplicate-detection true \
  --duplicate-detection-history-time-window PT10M
```

### Свойства очереди (Queue Properties)

| Свойство | Описание | По умолчанию | Диапазон |
|-----------|----------|--------------|-----------|
| **MaxSizeInMegabytes** | Максимальный размер очереди | 1024 MB | 1024–5120 MB (Standard)<br>1024–81920 MB (Premium) |
| **DefaultMessageTimeToLive** | Время жизни сообщения (TTL) | 14 дней | От 1 секунды до неограниченного |
| **LockDuration** | Время блокировки в режиме Peek Lock | 30 сек | 5 сек – 5 мин |
| **MaxDeliveryCount** | Количество попыток доставки до DLQ | 10 | 1–2000 |
| **RequiresDuplicateDetection** | Включить дедупликацию | false | true/false |
| **DuplicateDetectionHistoryTimeWindow** | Окно дедупликации | — | 20 сек – 7 дней |
| **EnableBatchedOperations** | Серверный batching | true | true/false |
| **RequiresSession** | Включить Sessions (FIFO) | false | true/false |
| **DeadLetteringOnMessageExpiration** | Перемещать просроченные сообщения в DLQ | false | true/false |

---

## Режимы получения сообщений (Receive Modes)

Service Bus поддерживает два режима получения сообщений с разным балансом между простотой и надёжностью.

---

### Сравнение режимов

| Характеристика | Receive and Delete | Peek Lock |
|----------------|-------------------|------------|
| **Гарантия доставки** | At-most-once | At-least-once |
| **Шаги обработки** | 1 шаг | 2 шага |
| **Простота** | ✅ Очень просто | Более сложный |
| **Отказоустойчивость** | ❌ Нет | ✅ Да |
| **Потеря сообщения при сбое?** | ✅ Да | ❌ Нет (авто-повтор) |
| **Lock / Timeout** | Нет | ✅ Есть блокировка |
| **Лучше всего подходит для** | Логирование, некритичные данные | Критичные данные |
| **Производительность** | Чуть быстрее | Чуть медленнее |

---

## 1️⃣ Receive and Delete

**Самая простая модель:**  
Сообщение помечается как обработанное сразу после отправки получателю.

### Особенности:

- Нет механизма подтверждения обработки
- Если consumer упал — сообщение теряется
- Подходит для некритичных сценариев
- Более высокая производительность

📌 Используется, когда потеря сообщения допустима (например, telemetry, логирование).

---

## 2️⃣ Peek Lock

**Надёжная модель:**  
Сообщение сначала блокируется, затем после успешной обработки подтверждается.

### Особенности:

- Сообщение блокируется на время LockDuration
- Если обработка завершена успешно → сообщение удаляется
- Если произошёл сбой → сообщение станет доступным повторно
- При превышении MaxDeliveryCount → сообщение перемещается в DLQ

📌 Это рекомендуемый режим для production и критичных операций.

---

## Экзаменационный акцент (AZ-204)

- Нужна надёжность → Peek Lock
- Допустима потеря сообщений → Receive and Delete
- Сценарии с транзакциями → Peek Lock
- DLQ работает только с Peek Lock

Главная идея:

Receive and Delete → просто  
Peek Lock → надёжно

```
Client              Service Bus              Outcome
┌──────┐            ┌──────────┐            
│      │ ─Request─> │  Queue   │            
│      │ <─Message──│ (marks   │  ✅ Success: Message delivered
│      │            │ consumed)│            
│      │ [CRASH]    │          │  ❌ Failure: Message LOST
└──────┘            └──────────┘            

Risk: If application crashes before processing, message is lost forever
```

**Use when:**
- ✅ Data loss is acceptable (e.g., telemetry, logs)
- ✅ Performance is critical
- ✅ Processing is very reliable
- ❌ NOT for critical data

**Code example:**
```csharp
// Receive and Delete mode
var receiver = client.CreateReceiver(
    queueName,
    new ServiceBusReceiverOptions 
    { 
        ReceiveMode = ServiceBusReceiveMode.ReceiveAndDelete 
    });

// Message automatically deleted after receive
var message = await receiver.ReceiveMessageAsync();

// No need to call CompleteMessage
// If crash occurs here, message is LOST
await ProcessMessageAsync(message);
```

```python
# Python: Receive and Delete
from azure.servicebus import ServiceBusClient, ServiceBusReceiveMode

client = ServiceBusClient.from_connection_string(connection_string)
receiver = client.get_queue_receiver(
    queue_name,
    receive_mode=ServiceBusReceiveMode.RECEIVE_AND_DELETE)

with receiver:
    for message in receiver:
        # Message already deleted from queue
        process_message(message)
```

### 2. Peek Lock Mode (Recommended)

**Two-stage receive**: Lock message, process it, then explicitly complete or abandon.

```
Client              Service Bus                   Outcome
┌──────┐            ┌──────────┐            
│      │ ─Request─> │  Queue   │            
│      │ <─Message──│ (LOCKED  │  Lock acquired
│      │            │ 30 sec)  │            
│      │            │          │            
│      │  Process   │          │            
│      │            │          │            
│      │ ─Complete─>│ (delete) │  ✅ Success: Message deleted
│      │            │          │            
│      │ [CRASH]    │          │  ✅ Auto-redelivery after timeout
│      │            │ (unlock  │     Message NOT lost
│      │            │ & retry) │            
└──────┘            └──────────┘            
```

## Workflow (Peek Lock Mode)

### Последовательность обработки:

1️⃣ **Receive**  
Сообщение блокируется (становится невидимым для других получателей) на время `LockDuration`  
(по умолчанию — 30 секунд).

2️⃣ **Process**  
Приложение обрабатывает сообщение.

3️⃣ **Завершение обработки:**
- **Complete** — сообщение явно удаляется из очереди
- **Abandon** — блокировка снимается, сообщение снова становится доступным
- **Dead-letter** — сообщение перемещается в DLQ
- **Defer** — обработка откладывается на более позднее время

---

## Тайм-аут блокировки (Lock Timeout)

- По умолчанию: **30 секунд**
- Настраивается: **от 5 секунд до 5 минут**
- Если время истекло до вызова Complete или Abandon:
   - Блокировка снимается автоматически
   - Сообщение снова становится доступным
- Блокировку можно продлить во время обработки (Lock Renewal)

---

## Когда использовать Peek Lock

- ✅ Критичные данные (потеря недопустима)
- ✅ Длительная обработка сообщений
- ✅ Необходима поддержка транзакций
- ✅ Требуется обработка ошибок (DLQ, повторные попытки)

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:

- важна надёжность доставки
- требуется повторная попытка при сбое
- используется DLQ
- обрабатываются финансовые или бизнес-критичные данные

→ Правильный выбор: **Peek Lock Mode**.

Главная идея:  
Peek Lock = контроль + надёжность + повторная доставка при сбое.

**Code example:**
```csharp
// Peek Lock mode (default)
var receiver = client.CreateReceiver(
    queueName,
    new ServiceBusReceiverOptions 
    { 
        ReceiveMode = ServiceBusReceiveMode.PeekLock  // Default
    });

var message = await receiver.ReceiveMessageAsync();

try
{
    // Process message (lock held for 30 seconds by default)
    await ProcessMessageAsync(message);
    
    // Explicitly complete (delete from queue)
    await receiver.CompleteMessageAsync(message);
}
catch (Exception ex)
{
    // Option 1: Abandon (requeue for retry)
    await receiver.AbandonMessageAsync(message);
    
    // Option 2: Dead-letter (move to DLQ)
    await receiver.DeadLetterMessageAsync(
        message,
        deadLetterReason: "ProcessingFailed",
        deadLetterErrorDescription: ex.Message);
    
    // Option 3: Defer (process later)
    await receiver.DeferMessageAsync(message);
}
```

**Renew lock for long processing:**
```csharp
var message = await receiver.ReceiveMessageAsync();

// Start background task to renew lock every 20 seconds
using var cts = new CancellationTokenSource();
var renewTask = Task.Run(async () =>
{
    while (!cts.Token.IsCancellationRequested)
    {
        await Task.Delay(TimeSpan.FromSeconds(20), cts.Token);
        await receiver.RenewMessageLockAsync(message);
    }
}, cts.Token);

try
{
    // Long-running processing (> 30 seconds)
    await LongRunningProcessAsync(message);
    await receiver.CompleteMessageAsync(message);
}
finally
{
    cts.Cancel(); // Stop renewing
}
```

```javascript
// JavaScript: Peek Lock with error handling
const receiver = client.createReceiver(queueName);

async function processMessages() {
    const messages = await receiver.receiveMessages(1);
    const message = messages[0];
    
    try {
        await processMessage(message);
        await receiver.completeMessage(message);
    } catch (error) {
        // Abandon: message will be redelivered
        await receiver.abandonMessage(message);
        
        // Or dead-letter if max retries exceeded
        if (message.deliveryCount >= 3) {
            await receiver.deadLetterMessage(message, {
                deadLetterReason: "MaxRetriesExceeded",
                deadLetterErrorDescription: error.message
            });
        }
    }
}
```

### Delivery Count and Retries

**Delivery count** tracks how many times a message has been delivered.

```csharp
var message = await receiver.ReceiveMessageAsync();

Console.WriteLine($"Delivery count: {message.DeliveryCount}");

if (message.DeliveryCount > 5)
{
    // Give up after 5 retries
    await receiver.DeadLetterMessageAsync(
        message,
        deadLetterReason: "MaxRetriesExceeded");
}
else
{
    try
    {
        await ProcessMessageAsync(message);
        await receiver.CompleteMessageAsync(message);
    }
    catch
    {
        await receiver.AbandonMessageAsync(message); // Retry
    }
}
```

---

## Topics and Subscriptions

**Topics** enable **publish-subscribe (pub/sub)** messaging pattern where messages are broadcast to multiple independent subscribers.

### Topic and Subscription Architecture

```
Publishers                Topic                 Subscriptions                Receivers
┌─────────┐             ┌────────┐           ┌──────────────┐            ┌──────────┐
│ App 1   │ ─────────> │        │ ────────> │Subscription 1│ ────────> │Receiver 1│
└─────────┘             │  Topic │           │(All messages)│            └──────────┘
┌─────────┐             │        │           └──────────────┘            
│ App 2   │ ─────────> │        │           ┌──────────────┐            ┌──────────┐
└─────────┘             │        │ ────────> │Subscription 2│ ────────> │Receiver 2│
┌─────────┐             │        │           │(Filtered)    │            └──────────┘
│ App 3   │ ─────────> │        │           └──────────────┘            
└─────────┘             └────────┘           ┌──────────────┐            ┌──────────┐
                                             │Subscription 3│ ────────> │Receiver 3│
                                             │(Filtered)    │            └──────────┘
                                             └──────────────┘            

Key Characteristics:
• Publishers send to topic (not subscriptions)
• Each subscription receives COPY of messages
• Subscriptions can filter messages
• Multiple receivers per subscription (competing consumers)
```

### Ключевые различия: Queue vs Topic

| Аспект | Queue | Topic |
|--------|--------|--------|
| **Паттерн** | Point-to-point | Publish-subscribe |
| **Доставка сообщения** | Один получатель | Несколько получателей (по одному на subscription) |
| **Типичный сценарий** | Распределение задач | Рассылка событий |
| **Копирование сообщения** | Одна копия | Копия для каждой subscription |
| **Фильтрация** | Не поддерживается | Поддерживается (на уровне подписки) |

---

## Пояснение

### Queue
- Используется для распределения задач между worker’ами
- Каждое сообщение обрабатывается только одним consumer’ом
- Подходит для 1:1 взаимодействия

---

### Topic
- Используется для событийной архитектуры
- Каждая subscription получает свою копию сообщения
- Поддерживает фильтрацию по свойствам
- Подходит для 1:N взаимодействия

---

## Экзаменационный ориентир (AZ-204)

- 1:1 → Queue
- 1:N → Topic
- Нужна фильтрация → Topic
- Нужно распределение задач → Queue

Главная разница:

Queue = задача  
Topic = событие

### Creating Topics and Subscriptions

```bash
# Create topic
az servicebus topic create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --name orderEvents \
  --max-size 1024

# Create subscription (all messages)
az servicebus topic subscription create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --topic-name orderEvents \
  --name allOrders

# Create subscription with SQL filter
az servicebus topic subscription rule create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --topic-name orderEvents \
  --subscription-name highPriorityOrders \
  --name HighPriorityFilter \
  --filter-sql-expression "Priority = 'High'"
```

### Publishing to Topic

```csharp
// Send to topic
await using var client = new ServiceBusClient(connectionString);
var sender = client.CreateSender("orderEvents");

var message = new ServiceBusMessage(JsonSerializer.Serialize(order));
message.ApplicationProperties["Priority"] = "High";
message.ApplicationProperties["OrderType"] = "Express";
message.ApplicationProperties["Region"] = "US-West";

await sender.SendMessageAsync(message);
```

### Receiving from Subscription

```csharp
// Receive from subscription (same as queue)
var receiver = client.CreateReceiver("orderEvents", "highPriorityOrders");

await foreach (var message in receiver.ReceiveMessagesAsync())
{
    var order = JsonSerializer.Deserialize<Order>(message.Body.ToString());
    await ProcessOrderAsync(order);
    await receiver.CompleteMessageAsync(message);
}
```

---

## Фильтрация сообщений (Message Filtering)

Подписки (Subscriptions) могут **фильтровать сообщения** с помощью SQL-подобных выражений.  
Только сообщения, соответствующие условию фильтра, будут доставлены в конкретную подписку.

Это позволяет реализовать интеллектуальную маршрутизацию внутри одного Topic.

---

## Типы фильтров

| Тип фильтра | Описание | Пример |
|-------------|----------|---------|
| **SQL Filter** | SQL-92 выражение по свойствам сообщения | `Priority = 'High' AND Region = 'US'` |
| **Correlation Filter** | Сравнение конкретных свойств (оптимизированный вариант) | `CorrelationId = '123'` |
| **Boolean Filter** | Логический фильтр (все или ни одного сообщения) | `TrueFilter`, `FalseFilter` |

---

## Пояснение

### SQL Filter
- Самый гибкий вариант
- Позволяет использовать условия AND, OR, сравнения
- Подходит для сложной маршрутизации

---

### Correlation Filter
- Быстрее SQL-фильтра
- Оптимизирован для точного совпадения свойств
- Часто используется для correlationId, label и других конкретных значений

---

### Boolean Filter
- `TrueFilter` — принимает все сообщения
- `FalseFilter` — не принимает ни одного
- Используется для управления логикой подписок

---

## Когда применять фильтрацию

- Multi-tenant приложения
- Разделение сообщений по регионам
- Разделение по типу события
- Разная бизнес-логика обработки

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:

- требуется маршрутизация по свойствам
- нужно отправлять сообщения разным сервисам по условиям
- используется publish-subscribe

→ правильный выбор — **Service Bus Topics + Subscription Filters**.

Queue Storage не поддерживает фильтрацию сообщений.

### SQL Filter Examples

```csharp
// Filter 1: High priority orders
var filter1 = new SqlRuleFilter("Priority = 'High'");

// Filter 2: Orders from specific region
var filter2 = new SqlRuleFilter("Region = 'US-West'");

// Filter 3: Complex filter (multiple conditions)
var filter3 = new SqlRuleFilter(
    "Priority = 'High' AND (Region = 'US-West' OR Region = 'US-East')");

// Filter 4: Numeric comparison
var filter4 = new SqlRuleFilter("Amount > 1000");

// Filter 5: String operations
var filter5 = new SqlRuleFilter("OrderType LIKE 'Express%'");

// Filter 6: NULL checks
var filter6 = new SqlRuleFilter("CustomProperty IS NOT NULL");

// Create subscription with filter
var ruleOptions = new CreateRuleOptions
{
    Name = "HighPriorityFilter",
    Filter = filter1
};

await administrationClient.CreateSubscriptionAsync(
    topicName,
    subscriptionName,
    ruleOptions);
```

### SQL Filter Operators

| Operator | Description | Example |
|----------|-------------|---------|
| `=` | Equal | `Priority = 'High'` |
| `!=` or `<>` | Not equal | `Status != 'Completed'` |
| `>`, `>=` | Greater than | `Amount > 1000` |
| `<`, `<=` | Less than | `Quantity <= 100` |
| `AND` | Logical AND | `Priority = 'High' AND Region = 'US'` |
| `OR` | Logical OR | `Priority = 'High' OR Priority = 'Critical'` |
| `NOT` | Logical NOT | `NOT (Status = 'Cancelled')` |
| `IN` | In list | `Region IN ('US-West', 'US-East')` |
| `LIKE` | Pattern match | `OrderType LIKE 'Express%'` |
| `IS NULL` | Null check | `CustomField IS NULL` |
| `IS NOT NULL` | Not null check | `CustomField IS NOT NULL` |

### Correlation Filter (Optimized)

**Correlation filters** are optimized for matching specific property values without expression evaluation.

```csharp
// Correlation filter (faster than SQL filter)
var correlationFilter = new CorrelationRuleFilter
{
    ContentType = "application/json",
    CorrelationId = "order-123",
    Subject = "OrderCreated"
};

correlationFilter.ApplicationProperties["Priority"] = "High";
correlationFilter.ApplicationProperties["Region"] = "US-West";

await administrationClient.CreateSubscriptionAsync(
    topicName,
    subscriptionName,
    new CreateRuleOptions 
    { 
        Name = "CorrelationFilter", 
        Filter = correlationFilter 
    });
```

**Performance:**
- ✅ Correlation filters are faster than SQL filters
- ✅ Use correlation filters when matching exact property values
- ✅ Use SQL filters for complex expressions

### Filter Actions

**Actions** modify message properties as they are copied to the subscription.

```csharp
// Create filter with action
var ruleOptions = new CreateRuleOptions
{
    Name = "USWestFilter",
    Filter = new SqlRuleFilter("Region = 'US-West'"),
    Action = new SqlRuleAction("SET ProcessedBy = 'WestRegionProcessor'")
};

await administrationClient.CreateSubscriptionAsync(
    topicName,
    subscriptionName,
    ruleOptions);
```

**Action operations:**
- `SET property = value`: Set or update property
- `REMOVE property`: Remove property

```csharp
// Multiple actions
var action = new SqlRuleAction(
    "SET ProcessedBy = 'Processor1'; SET ProcessedDate = GetDate(); REMOVE TempProperty");
```

---

## Real-World Scenarios

### Scenario 1: Order Processing (Queue)

**Competing consumers** pattern for distributing orders across multiple workers.

```csharp
// Producer (Web API)
var message = new ServiceBusMessage(JsonSerializer.Serialize(order));
message.MessageId = order.OrderId.ToString();
await queueSender.SendMessageAsync(message);

// Consumer 1, 2, 3 (Worker Services) - Compete for messages
var receiver = client.CreateReceiver("orderQueue");
await foreach (var message in receiver.ReceiveMessagesAsync())
{
    var order = JsonSerializer.Deserialize<Order>(message.Body.ToString());
    await ProcessOrderAsync(order);
    await receiver.CompleteMessageAsync(message);
}
```

### Scenario 2: Event Broadcasting (Topic)

**Publish-subscribe** pattern for notifying multiple systems about events.

```
Event Source          Topic: OrderEvents        Subscriptions
┌──────────┐         ┌────────────────┐       ┌──────────────────┐
│          │         │                │──────>│Analytics         │
│ E-commerce│────────>│ OrderCreated   │       │(all events)      │
│ Website  │         │ OrderUpdated   │       └──────────────────┘
│          │         │ OrderCancelled │       ┌──────────────────┐
└──────────┘         │                │──────>│Notifications     │
                     │                │       │(Priority=High)   │
                     │                │       └──────────────────┘
                     │                │       ┌──────────────────┐
                     │                │──────>│Inventory         │
                     └────────────────┘       │(OrderCreated only)│
                                              └──────────────────┘
```

```csharp
// Publisher
var message = new ServiceBusMessage(JsonSerializer.Serialize(orderEvent));
message.ApplicationProperties["EventType"] = "OrderCreated";
message.ApplicationProperties["Priority"] = "High";
await topicSender.SendMessageAsync(message);

// Subscription 1: Analytics (all events)
// No filter - receives all messages

// Subscription 2: Notifications (high priority only)
// Filter: Priority = 'High'

// Subscription 3: Inventory (OrderCreated only)
// Filter: EventType = 'OrderCreated'
```

### Scenario 3: Regional Routing (Topic with Filters)

Route messages to different processors based on region.

```csharp
// Publisher
var message = new ServiceBusMessage(JsonSerializer.Serialize(data));
message.ApplicationProperties["Region"] = "US-West";
await topicSender.SendMessageAsync(message);

// Subscription filters
// US-West subscription: Region = 'US-West'
// US-East subscription: Region = 'US-East'
// EU subscription: Region LIKE 'EU-%'
// Global subscription: TrueFilter (all messages)
```

### Scenario 4: FIFO Processing with Sessions

Guarantee order processing per customer using sessions.

```csharp
// Send with session (FIFO per customer)
var message = new ServiceBusMessage(JsonSerializer.Serialize(order));
message.SessionId = $"customer-{order.CustomerId}";
message.MessageId = $"order-{order.OrderId}";
await sender.SendMessageAsync(message);

// Receive with session
await using var sessionReceiver = await client.AcceptSessionAsync(
    queueName,
    new ServiceBusSessionReceiverOptions());

// Messages for this session processed in FIFO order
await foreach (var message in sessionReceiver.ReceiveMessagesAsync())
{
    var order = JsonSerializer.Deserialize<Order>(message.Body.ToString());
    await ProcessOrderAsync(order); // Guaranteed FIFO per customer
    await sessionReceiver.CompleteMessageAsync(message);
}
```

---

## Лучшие практики

### 1️⃣ Выбор правильной сущности

---

### ✅ Используйте **Queue**, если:

- Нужна модель point-to-point (1:1)
- Требуется распределение задач между worker’ами (competing consumers)
- Необходима балансировка нагрузки
- Каждое сообщение должно быть обработано только одним получателем
- Нет необходимости в рассылке нескольким системам

📌 Queue подходит для фоновых задач и асинхронной обработки.

---

### ✅ Используйте **Topic**, если:

- Требуется рассылка сообщений нескольким подписчикам (1:N)
- Реализуется event-driven архитектура
- Несколько систем должны независимо обрабатывать одно и то же событие
- Нужна фильтрация сообщений по свойствам

📌 Topic идеален для событий и микросервисной архитектуры.

---

## Экзаменационный ориентир (AZ-204)

- 1:1 → Queue
- 1:N → Topic
- Task distribution → Queue
- Event broadcasting → Topic

Главное правило:  
Queue = задачи  
Topic = события

### 2. Implement Idempotency

```csharp
// ✅ Good: Idempotent processing
var orderId = message.MessageId;
if (!await _orderRepository.ExistsAsync(orderId))
{
    await ProcessOrderAsync(orderId);
}
await receiver.CompleteMessageAsync(message);
```

### 3. Handle Errors Gracefully

```csharp
try
{
    await ProcessMessageAsync(message);
    await receiver.CompleteMessageAsync(message);
}
catch (TransientException ex)
{
    // Retry by abandoning
    if (message.DeliveryCount < 5)
    {
        await receiver.AbandonMessageAsync(message);
    }
    else
    {
        await receiver.DeadLetterMessageAsync(message);
    }
}
catch (PermanentException ex)
{
    // Don't retry - dead-letter immediately
    await receiver.DeadLetterMessageAsync(message);
}
```

### 4. Use Sessions for Ordering

```csharp
// ✅ Enable sessions for FIFO
az servicebus queue create \
  --name sessionQueue \
  --enable-session true

// Send with SessionId
message.SessionId = groupIdentifier;
```

### 5. Monitor Dead-Letter Queue

```csharp
// Periodically check DLQ
var dlqReceiver = client.CreateReceiver(
    queueName,
    new ServiceBusReceiverOptions 
    { 
        SubQueue = SubQueue.DeadLetter 
    });

await foreach (var dlqMessage in dlqReceiver.ReceiveMessagesAsync())
{
    Console.WriteLine($"Dead-letter reason: {dlqMessage.DeadLetterReason}");
    // Inspect, fix, and resubmit
}
```

### 6. Optimize Filters

```csharp
// ✅ Good: Correlation filter (faster)
var filter = new CorrelationRuleFilter
{
    ApplicationProperties = { ["Region"] = "US-West" }
};

// ❌ Avoid: Complex SQL expressions (slower)
var filter = new SqlRuleFilter(
    "SQRT(Amount) > 100 AND SUBSTRING(Name, 1, 5) = 'Order'");
```

---
## Советы для экзамена AZ-204

### Ключевые концепции

1. **Queue** = Point-to-point  
   Одно сообщение → один получатель.

2. **Topic** = Publish-subscribe  
   Одно сообщение → несколько получателей.

3. **Peek Lock** = Отказоустойчивый режим  
   Двухэтапное получение с подтверждением.

4. **Receive and Delete** = Простой, но с риском потери  
   Сообщение удаляется сразу при получении.

5. **Sessions** = Гарантия FIFO  
   Порядок гарантируется внутри одной сессии.

6. **Filters** = Маршрутизация сообщений  
   Используются в подписках Topic.

---

## Что нужно помнить

| Сценарий | Решение |
|-----------|----------|
| **Обработка строго по порядку** | Queue с включёнными sessions |
| **Рассылка событий** | Topic с несколькими subscriptions |
| **Отказоустойчивая обработка** | Peek Lock |
| **Высокая производительность, допустима потеря** | Receive and Delete |
| **Маршрутизация по свойствам** | Topic + SQL filters |
| **Распределение задач** | Queue + competing consumers |

---

## Типовые вопросы на экзамене

**Как гарантировать FIFO?**  
→ Включить sessions и использовать SessionId.

**Как отправить события нескольким системам?**  
→ Использовать Topic с несколькими subscriptions.

**Как предотвратить потерю сообщения при сбое?**  
→ Использовать Peek Lock и явный Complete.

**Как маршрутизировать сообщения по свойствам?**  
→ Topic + фильтры подписок.

**Как обработать "ядовитые" сообщения?**  
→ Проверять DeliveryCount и перемещать в DLQ после превышения лимита.

---

## Итог

### Service Bus Queues
- ✅ Point-to-point модель
- ✅ Competing consumers
- ✅ Одно сообщение — один получатель
- ✅ FIFO через sessions

### Service Bus Topics
- ✅ Publish-subscribe модель
- ✅ Одно сообщение — несколько подписчиков
- ✅ Фильтрация на уровне подписки
- ✅ Независимая обработка

### Режимы получения

- **Peek Lock** — надёжный, двухэтапный (рекомендуется)
- **Receive and Delete** — простой, но возможна потеря

---

### Главное правило

- Sessions → порядок
- Filters → маршрутизация
- Peek Lock → надёжность

Понимание этих трёх механизмов покрывает большинство вопросов по Service Bus на AZ-204.