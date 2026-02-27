# Explore Azure Service Bus

## What is Azure Service Bus?

**Azure Service Bus** is a fully managed enterprise message broker with message queues and publish-subscribe topics. It enables decoupling of applications and services, providing reliable message delivery and advanced features for enterprise integration patterns.

### Key Characteristics

```
Azure Service Bus Architecture
═══════════════════════════════════════════════════════════

                    Service Bus Namespace
         ┌──────────────────────────────────────────┐
         │                                          │
         │   Queues (Point-to-Point)                │
         │   ├── Queue 1                            │
         │   ├── Queue 2                            │
         │   └── Queue N                            │
         │                                          │
         │   Topics (Publish-Subscribe)             │
         │   ├── Topic 1                            │
         │   │   ├── Subscription A                 │
         │   │   ├── Subscription B                 │
         │   │   └── Subscription C                 │
         │   └── Topic 2                            │
         │       ├── Subscription X                 │
         │       └── Subscription Y                 │
         │                                          │
         └──────────────────────────────────────────┘

Producers ──────> Service Bus ──────> Consumers
(Senders)         (Broker)            (Receivers)
```

### Основные возможности (Core Capabilities)

| Возможность | Описание |
|-------------|----------|
| **Message Queuing** | Хранение сообщений до тех пор, пока принимающее приложение их не обработает |
| **Load Balancing** | Распределение нагрузки между несколькими competing consumers |
| **Temporal Decoupling** | Продюсер и потребитель могут работать независимо во времени |
| **Load Leveling** | Сглаживание пиков нагрузки |
| **Reliable Delivery** | Гарантия доставки: at-least-once или at-most-once |
| **Publish-Subscribe** | Рассылка сообщений нескольким независимым подписчикам |
| **Advanced Routing** | Фильтрация и маршрутизация сообщений по свойствам |

---

## Краткое объяснение по каждому пункту

### 📦 Message Queuing
Позволяет приложениям обмениваться сообщениями асинхронно, не требуя одновременной доступности обеих сторон.

---

### ⚖️ Load Balancing
Несколько consumers могут обрабатывать сообщения параллельно, повышая throughput.

---

### ⏳ Temporal Decoupling
Продюсер отправляет сообщение и не ждёт немедленного ответа. Потребитель может обработать его позже.

---

### 📊 Load Leveling
Очередь служит буфером при резких скачках нагрузки.

---

### 🔒 Reliable Delivery
- **At-least-once** — сообщение будет доставлено минимум один раз
- **At-most-once** — сообщение может быть доставлено не более одного раза

---

### 📡 Publish-Subscribe
Один отправитель → несколько получателей. Каждый подписчик получает собственную копию сообщения.

---

### 🎯 Advanced Routing
Сообщения могут направляться в разные подписки на основе:
- свойств сообщения
- SQL-фильтров
- correlation-фильтров

---

## Экзаменационный акцент (AZ-204)

- Publish-Subscribe → Service Bus Topics
- Advanced Routing → Service Bus
- Простая очередь → Queue Storage
- Load leveling → подходят оба варианта
- Enterprise-механизмы доставки → Service Bus

Главное — понимать, какие возможности относятся к базовой очереди, а какие к enterprise-месседжингу.

---

## Service Bus Concepts

### Namespace

A **namespace** is a container for all messaging components (queues and topics).

**Properties:**
- Unique name (DNS name): `{namespace-name}.servicebus.windows.net`
- Region/location
- Pricing tier (Basic, Standard, Premium)
- Capacity units (Premium only)

```bash
# Create namespace
az servicebus namespace create \
  --resource-group myResourceGroup \
  --name myServiceBusNamespace \
  --location eastus \
  --sku Standard
```

### Queues (Очереди)

**Point-to-point** сущности обмена сообщениями, которые хранят сообщения в порядке **FIFO** (при включённых sessions в Service Bus).

---

## Основные характеристики

- **Competing Consumers pattern**  
  Несколько потребителей могут обрабатывать сообщения параллельно.

- **Одно сообщение → один получатель**  
  Каждое сообщение обрабатывается только одним consumer’ом.

- **Персистентность сообщений**  
  Сообщения сохраняются, пока не будут получены и удалены.

- **Балансировка нагрузки**  
  Сообщения автоматически распределяются между активными consumers.

---

## Как это работает

1. Producer отправляет сообщение в очередь
2. Сообщение хранится до получения
3. Consumer получает сообщение
4. После успешной обработки сообщение удаляется

Если обработка не удалась:
- В Service Bus сообщение может попасть в DLQ
- В Queue Storage сообщение снова станет видимым после visibility timeout

---

## Когда использовать

- Асинхронная обработка задач
- Фоновая обработка (background jobs)
- Разгрузка веб-приложения
- Масштабируемая обработка через несколько worker’ов

---

## Важно для AZ-204

- FIFO гарантируется **только в Service Bus при использовании sessions**
- Queue Storage не гарантирует строгий порядок
- Queue = 1:1 взаимодействие
- Для 1:N используйте Topics

Главная идея: Queue — это базовый механизм асинхронной и надёжной обработки задач.

```bash
# Create queue
az servicebus queue create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --name myQueue \
  --max-size 1024
```

### Topics и Subscriptions

**Publish-Subscribe** паттерн, при котором несколько подписчиков получают копию каждого сообщения.

---

## Ключевые особенности

- **One-to-many коммуникация**  
  Один отправитель публикует сообщение в Topic, и оно доставляется нескольким подписчикам.

- **Каждая Subscription работает как отдельная очередь**  
  У каждой подписки своя изолированная очередь сообщений.

- **Независимая обработка**  
  Каждый подписчик обрабатывает сообщения в своём темпе. Замедление одного не влияет на других.

- **Фильтрация на уровне подписки**  
  Можно настраивать правила, по которым подписка будет получать только определённые сообщения (например, по региону или типу события).

---

## Как это работает

1. Producer отправляет сообщение в Topic
2. Service Bus создаёт копию сообщения для каждой подписки
3. Каждый consumer получает сообщение из своей subscription
4. Обработка полностью независима

---

## Когда использовать

- Event-driven архитектура
- Рассылка событий нескольким сервисам
- IoT-сценарии
- Микросервисная архитектура
- Системы с разной логикой обработки одного события

---

## Важно для AZ-204

- Publish/Subscribe реализуется через **Service Bus Topics**
- Queue Storage не поддерживает pub/sub
- Фильтрация сообщений доступна только в Service Bus
- Каждая subscription масштабируется независимо

Главная идея:  
Queue → 1:1  
Topic → 1:N  
Subscription = отдельная очередь с собственной логикой обработки

```bash
# Create topic
az servicebus topic create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --name myTopic

# Create subscription
az servicebus topic subscription create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --topic-name myTopic \
  --name mySubscription
```

---

## Тарифные планы Service Bus

Azure Service Bus предлагает три тарифных уровня с разными возможностями и характеристиками производительности.

---

## Сравнение тарифов

| Возможность | Basic | Standard | Premium |
|-------------|--------|----------|----------|
| **Queues** | ✅ Да | ✅ Да | ✅ Да |
| **Topics / Subscriptions** | ❌ Нет | ✅ Да | ✅ Да |
| **Макс. размер сообщения** | 256 KB | 256 KB | 100 MB |
| **Макс. размер очереди / topic** | 1 GB | 1–5 GB | 1–80 GB |
| **Throughput** | Низкий | Переменный (shared) | Высокий (dedicated) |
| **Latency** | Стандартная | Стандартная | Низкая (<10 ms) |
| **Изоляция ресурсов** | ❌ Shared | ❌ Shared | ✅ Выделенные CPU/Memory |
| **Транзакции** | ❌ Нет | ✅ Да | ✅ Да |
| **Duplicate Detection** | ❌ Нет | ✅ Да | ✅ Да |
| **Sessions (FIFO)** | ❌ Нет | ✅ Да | ✅ Да |
| **Batching** | ✅ Да | ✅ Да | ✅ Да |
| **Auto-Forwarding** | ❌ Нет | ✅ Да | ✅ Да |
| **Scheduled Messages** | ❌ Нет | ✅ Да | ✅ Да |
| **Dead-Letter Queue** | ✅ Да | ✅ Да | ✅ Да |
| **Geo-Disaster Recovery** | ❌ Нет | ❌ Нет | ✅ Да |
| **Availability Zones** | ❌ Нет | ❌ Нет | ✅ Да |
| **Private Endpoints** | ❌ Нет | ❌ Нет | ✅ Да |
| **JMS 2.0 Support** | ❌ Нет | ❌ Нет | ✅ Да |
| **Модель оплаты** | Оплата за операции | Оплата за операции | Фиксированная ежемесячная |
| **Типичный сценарий** | Dev/Test | Production | Mission-critical |

---

## Basic Tier

### Подходит для:
- Разработки и тестирования
- Простых сценариев с очередями
- Проектов с ограниченным бюджетом

### Ограничения:
- Нет поддержки Topics / Subscriptions
- Нет sessions (FIFO)
- Нет транзакций
- Нет duplicate detection
- Ограниченная производительность

📌 Basic — минимальный функционал без enterprise-возможностей.

---

## Standard Tier

### Подходит для:
- Production-нагрузки
- Большинства бизнес-приложений
- Сценариев с pub/sub
- FIFO через sessions
- Транзакционных сценариев

### Особенности:
- Shared инфраструктура
- Поддержка Topics
- Поддержка advanced-функций

📌 Это наиболее часто используемый тариф для production.

---

## Premium Tier

### Подходит для:
- Mission-critical систем
- Высокой нагрузки
- Низкой задержки
- Требований к изоляции ресурсов
- Интеграции с JMS 2.0

### Преимущества:
- Выделенные ресурсы (CPU/Memory)
- Высокий throughput
- Низкая latency
- Поддержка Geo-DR
- Поддержка Availability Zones
- Сообщения до 100 MB

📌 Premium — для высоконагруженных enterprise-систем.

---

## Экзаменационный акцент (AZ-204)

- Нужны Topics → минимум Standard
- Нужны Sessions / Transactions → минимум Standard
- Нужна высокая производительность и изоляция → Premium
- Dev/Test без сложных требований → Basic

Главный ориентир:

Basic → простая очередь  
Standard → полноценный production  
Premium → критически важные высоконагруженные системы
```csharp
// Basic tier usage
await using var client = new ServiceBusClient(connectionString);
var sender = client.CreateSender("myqueue");
await sender.SendMessageAsync(new ServiceBusMessage("Hello"));
```

### Standard Tier

## Подходит для:

- Production-нагрузок
- Сценариев с Topics и Subscriptions (pub/sub)
- Переменной нагрузки
- Большинства бизнес-приложений

---

## Возможности:

- **Topics и Subscriptions** (поддержка publish/subscribe)
- **Транзакции** (атомарные операции)
- **Duplicate Detection** (защита от повторной обработки)
- **Sessions (FIFO)** (гарантированный порядок обработки)
- **Продвинутая маршрутизация** (фильтры и правила подписок)

---

## Важно понимать

- Использует shared-инфраструктуру (ресурсы не выделенные)
- Подходит для большинства production-сценариев
- Самый распространённый тариф для корпоративных приложений

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:

- Нужны Topics
- Требуется FIFO
- Требуются транзакции
- Нужна фильтрация сообщений

→ Минимальный правильный ответ — **Standard Tier**.

```csharp
// Standard tier with topic
await using var client = new ServiceBusClient(connectionString);
var sender = client.CreateSender("mytopic");

var message = new ServiceBusMessage("Event occurred");
message.ApplicationProperties["EventType"] = "OrderCreated";
await sender.SendMessageAsync(message);
```

### Premium Tier

## Подходит для:

- Mission-critical систем
- Сценариев с предсказуемой производительностью
- Очень высокой пропускной способности (80 000+ сообщений/сек)
- Сообщений большого размера (до 100 MB)

---

## Возможности:

- **Выделенные ресурсы (CPU и память)**
- **Предсказуемая низкая задержка (<10 ms)**
- **Изоляция ресурсов** (нет "шумных соседей")
- **Availability Zones**
- **Geo-Disaster Recovery**
- **Private Endpoints**
- **Поддержка JMS 2.0**

---

## Важно понимать

- Фиксированная ежемесячная стоимость (через Messaging Units)
- Подходит для высоконагруженных и критичных систем
- Гарантирует стабильную производительность независимо от других клиентов

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:

- Требуется гарантированная производительность
- Очень высокий throughput
- Нужна изоляция ресурсов
- Требуется Geo-DR или Availability Zones
- Сообщения > 256 KB

→ Правильный выбор — **Premium Tier**.

```csharp
// Premium tier with large message
await using var client = new ServiceBusClient(connectionString);
var sender = client.CreateSender("myqueue");

// Send up to 100 MB message
byte[] largePayload = new byte[50 * 1024 * 1024]; // 50 MB
var message = new ServiceBusMessage(largePayload);
await sender.SendMessageAsync(message);
```

### Capacity Units (Premium Only)

Premium tier uses **Messaging Units (MU)** for capacity:

| Messaging Units | Throughput | Approximate Cost |
|----------------|------------|------------------|
| **1 MU** | ~1,000 msg/sec | ~$667/month |
| **2 MU** | ~2,000 msg/sec | ~$1,334/month |
| **4 MU** | ~4,000 msg/sec | ~$2,668/month |
| **8 MU** | ~8,000 msg/sec | ~$5,336/month |

---

## Common Messaging Scenarios

### 1. Messaging (Asynchronous Communication)

Decouple applications using queues for reliable message transfer.

```
Producer                Queue               Consumer
┌────────┐            ┌──────┐            ┌────────┐
│ Web    │ ─────────> │ Msg1 │ ─────────> │Worker  │
│ API    │            │ Msg2 │            │Service │
└────────┘            │ Msg3 │            └────────┘
                      └──────┘
```

**Use case:** Web API queues orders for background processing.

```csharp
// Producer (Web API)
var message = new ServiceBusMessage(JsonSerializer.Serialize(order));
message.MessageId = order.OrderId.ToString();
await sender.SendMessageAsync(message);

// Consumer (Worker Service)
await foreach (ServiceBusReceivedMessage message in receiver.ReceiveMessagesAsync())
{
    var order = JsonSerializer.Deserialize<Order>(message.Body.ToString());
    await ProcessOrderAsync(order);
    await receiver.CompleteMessageAsync(message);
}
```

### 2. Decoupling Applications

Separate application tiers to enable independent scaling and deployment.

```
Front-End              Service Bus          Back-End Services
┌────────┐            ┌──────────┐         ┌─────────────┐
│        │            │  Queue   │         │  Inventory  │
│  Web   │ ────────> │  Topic   │ ─────> │  Payment    │
│  App   │            │          │         │  Shipping   │
└────────┘            └──────────┘         └─────────────┘
```

**Benefits:**
- Front-end doesn't wait for back-end processing
- Back-end services can be updated independently
- Failures don't cascade across tiers

### 3. Topics and Subscriptions (Fan-Out)

Broadcast events to multiple independent subscribers.

```
Publisher              Topic                Subscriptions
┌────────┐            ┌──────┐            ┌──────────────┐
│        │            │      │───────────>│ Analytics    │
│ Event  │ ────────> │ Topic│            └──────────────┘
│ Source │            │      │───────────>┌──────────────┐
└────────┘            │      │            │ Notifications│
                      │      │            └──────────────┘
                      └──────┘───────────>┌──────────────┐
                                          │ Archival     │
                                          └──────────────┘
```

**Use case:** Order events sent to analytics, notifications, and archival systems.

```csharp
// Publisher
var message = new ServiceBusMessage(JsonSerializer.Serialize(orderEvent));
message.ApplicationProperties["EventType"] = "OrderCreated";
message.ApplicationProperties["Priority"] = "High";
await topicSender.SendMessageAsync(message);

// Subscription 1: Analytics (all events)
// Subscription 2: Notifications (only High priority)
// Subscription 3: Archival (all events)
```

### 4. Message Sessions (FIFO Processing)

Process related messages in order using sessions.

```
Producer               Session-Enabled Queue        Consumer
┌────────┐            ┌─────────────────────┐      ┌────────┐
│        │ ──Order1─> │ Session: Customer1  │ ───> │        │
│ Order  │ ──Order2─> │ - Order1            │      │Process │
│ System │ ──Order3─> │ - Order2            │      │in FIFO │
│        │            │                     │      │order   │
│        │ ──Order4─> │ Session: Customer2  │      │        │
│        │ ──Order5─> │ - Order4            │      │        │
└────────┘            │ - Order5            │      └────────┘
                      └─────────────────────┘
```

**Use case:** Process orders for each customer in sequence.

```csharp
// Send with session
var message = new ServiceBusMessage(JsonSerializer.Serialize(order));
message.SessionId = $"customer-{order.CustomerId}";
await sender.SendMessageAsync(message);

// Receive with session
await using var sessionReceiver = await client.AcceptSessionAsync(
    queueName, 
    sessionId: $"customer-{customerId}");

await foreach (var message in sessionReceiver.ReceiveMessagesAsync())
{
    // Messages processed in FIFO order per session
    await ProcessOrderAsync(message);
    await sessionReceiver.CompleteMessageAsync(message);
}
```

---

## Расширенные возможности

### 1️⃣ Message Sessions

**Гарантия FIFO** достигается за счёт группировки связанных сообщений по `SessionId`.

---

## Возможности

- **Строгий порядок внутри сессии**  
  Все сообщения с одинаковым `SessionId` обрабатываются строго последовательно.

- **Хранение состояния сессии**  
  Можно сохранять состояние обработки между сообщениями одной сессии.

- **Один получатель на сессию**  
  В каждый момент времени только один consumer может обрабатывать конкретную сессию.

- **Параллельная обработка между сессиями**  
  Разные `SessionId` могут обрабатываться параллельно разными consumers.

---

## Когда использовать

- Обработка заказов одного клиента
- Workflow-сценарии с несколькими шагами
- Финансовые транзакции
- Любые процессы, где критичен порядок обработки

---

## Важно для AZ-204

- FIFO гарантируется **только через Sessions**
- Queue Storage не поддерживает sessions
- Без включённых sessions порядок обработки не гарантируется

Главная идея:  
Session = логическая группа сообщений с гарантированным порядком обработки.

```csharp
// Enable sessions on queue
az servicebus queue create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --name sessionQueue \
  --enable-session true

// Send with session
var message = new ServiceBusMessage("Order data");
message.SessionId = "session-123";
await sender.SendMessageAsync(message);

// Receive with session
var sessionReceiver = await client.AcceptSessionAsync("sessionQueue", "session-123");
var message = await sessionReceiver.ReceiveMessageAsync();
```

### 2. Auto-Forwarding

**Automatically forward** messages from one queue/subscription to another queue/topic.

**Use cases:**
- Chain processing steps
- Aggregate messages from multiple sources
- Route messages to different regions

```bash
# Create auto-forwarding rule
az servicebus queue create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --name sourceQueue \
  --forward-to destinationQueue
```

```
Source Queue    Auto-Forward     Destination Queue
┌──────────┐   ──────────────>  ┌────────────┐
│  Step 1  │                    │   Step 2   │
└──────────┘                    └────────────┘
```

### 3️⃣ Dead-Letter Queue (DLQ)

**Автоматическое перемещение** необрабатываемых сообщений в отдельную очередь для анализа.

DLQ — это встроенный механизм Service Bus для изоляции проблемных сообщений без потери данных.

---

## Когда сообщение попадает в DLQ:

- **Превышено максимальное количество попыток доставки**  
  Сообщение несколько раз не было успешно обработано.

- **Истёк срок жизни сообщения (TTL)**  
  Сообщение не было обработано вовремя.

- **Потеря блокировки сессии (Session lock lost)**  
  Consumer не завершил обработку в отведённое время.

- **Явно отправлено в DLQ получателем**  
  Приложение вручную пометило сообщение как ошибочное.

- **Ошибка фильтрации в подписке**  
  Сообщение не прошло условия фильтра.

---

## Зачем нужен DLQ

- Диагностика ошибок
- Повторная обработка (replay)
- Анализ некорректных сообщений
- Защита основной очереди от "залипания"

---

## Важно для AZ-204

- DLQ — встроенная функция Service Bus
- Queue Storage не имеет встроенной DLQ (реализуется вручную)
- Часто в вопросах DLQ указывает на выбор Service Bus

Главная идея:  
DLQ = безопасная изоляция проблемных сообщений без их потери.

```csharp
// Process with dead-lettering
try
{
    await ProcessMessageAsync(message);
    await receiver.CompleteMessageAsync(message);
}
catch (Exception ex)
{
    // Move to dead-letter queue
    await receiver.DeadLetterMessageAsync(
        message, 
        deadLetterReason: "ProcessingFailed",
        deadLetterErrorDescription: ex.Message);
}

// Process dead-letter queue
var dlqReceiver = client.CreateReceiver(
    queueName, 
    new ServiceBusReceiverOptions 
    { 
        SubQueue = SubQueue.DeadLetter 
    });

await foreach (var dlqMessage in dlqReceiver.ReceiveMessagesAsync())
{
    // Inspect, fix, and resubmit
    Console.WriteLine($"Dead-letter reason: {dlqMessage.DeadLetterReason}");
    Console.WriteLine($"Description: {dlqMessage.DeadLetterErrorDescription}");
}
```

### 4. Scheduled Delivery

**Schedule messages** for future delivery.

```csharp
// Schedule message for delivery in 1 hour
var message = new ServiceBusMessage("Reminder: Meeting in 10 minutes");
var scheduleTime = DateTimeOffset.UtcNow.AddHours(1);

long sequenceNumber = await sender.ScheduleMessageAsync(message, scheduleTime);

// Cancel scheduled message
await sender.CancelScheduledMessageAsync(sequenceNumber);
```

### 5. Message Deferral

**Defer message processing** to retrieve later by sequence number.

```csharp
// Defer message
long sequenceNumber = message.SequenceNumber;
await receiver.DeferMessageAsync(message);

// Retrieve deferred message later
var deferredMessage = await receiver.ReceiveDeferredMessageAsync(sequenceNumber);
```

### 6. Transactions

**Group operations** into atomic transaction scopes.

```csharp
using var ts = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled);

// All operations succeed or fail together
await receiver.CompleteMessageAsync(message1);
await sender.SendMessageAsync(message2);
await receiver.CompleteMessageAsync(message3);

ts.Complete(); // Commit transaction
```

### 7. Duplicate Detection

**Automatically detect** and discard duplicate messages.

**Enable duplicate detection:**
```bash
az servicebus queue create \
  --resource-group myResourceGroup \
  --namespace-name myServiceBusNamespace \
  --name dedupeQueue \
  --enable-duplicate-detection true \
  --duplicate-detection-history-time-window 10
```

**Send with MessageId:**
```csharp
var message = new ServiceBusMessage("Order data");
message.MessageId = $"order-{order.OrderId}"; // Used for deduplication

// If sent twice within detection window, second message is discarded
await sender.SendMessageAsync(message);
```

### 8. Filtering and Actions

**Filter messages** in topic subscriptions using SQL-like expressions.

```csharp
// Create subscription with filter
var options = new CreateSubscriptionOptions("myTopic", "highPrioritySubscription");
var rule = new CreateRuleOptions("HighPriorityFilter")
{
    Filter = new SqlRuleFilter("Priority = 'High'")
};

await administrationClient.CreateSubscriptionAsync(options, rule);

// Send message with property
var message = new ServiceBusMessage("Important event");
message.ApplicationProperties["Priority"] = "High";
await sender.SendMessageAsync(message);
```

### 9. Security

**Authentication options:**
- **Shared Access Signatures (SAS)**: Token-based access
- **Azure AD (RBAC)**: Role-based access control
- **Managed Identity**: Identity-based authentication

```csharp
// Using Azure AD (Managed Identity)
var credential = new DefaultAzureCredential();
await using var client = new ServiceBusClient(
    "myNamespace.servicebus.windows.net",
    credential);

// Using SAS connection string
await using var client = new ServiceBusClient(connectionString);
```

**Azure RBAC roles:**
- `Azure Service Bus Data Owner`: Full access
- `Azure Service Bus Data Sender`: Send messages only
- `Azure Service Bus Data Receiver`: Receive messages only

### 10. Geo-Disaster Recovery (Premium Only)

**Pair namespaces** across regions for disaster recovery.

```bash
# Create geo-pairing
az servicebus georecovery-alias create \
  --resource-group myResourceGroup \
  --namespace-name primaryNamespace \
  --alias myAlias \
  --partner-namespace secondaryNamespaceResourceId
```

## Дополнительные возможности

### Geo-Disaster Recovery (Geo-DR)

Обеспечивает устойчивость к региональным сбоям на уровне метаданных.

### Возможности:

- **Репликация метаданных**  
  Очереди, topics и subscriptions копируются во вторичный регион.

- **Автоматическое переключение (failover)**  
  При сбое можно выполнить переключение на вторичный namespace.

- **Единая строка подключения (alias)**  
  Клиенты используют alias вместо конкретного namespace.

- **Нет репликации сообщений**  
  Данные сообщений не копируются автоматически — для полной отказоустойчивости требуется отдельная стратегия (например, cross-region архитектура).

📌 Geo-DR защищает структуру, но не сами сообщения.

---

# Протоколы

## AMQP 1.0 (Рекомендуется)

**Advanced Message Queuing Protocol** — открытый стандарт ISO/IEC для обмена сообщениями.

---

### Преимущества:

- **Бинарный протокол**  
  Более эффективен, чем текстовые протоколы.

- **Multiplexing**  
  Несколько сессий могут работать через одно TCP-соединение.

- **Кроссплатформенность**  
  Поддерживается многими языками и платформами.

- **Долгоживущие соединения**  
  Подходит для высоконагруженных систем.

- **Поддержка транзакций**  
  Встроенная поддержка атомарных операций.

---

## Экзаменационный акцент (AZ-204)

- AMQP 1.0 — предпочтительный протокол для Service Bus
- Поддерживает транзакции и sessions
- HTTP используется, но AMQP — более эффективный вариант

Если в вопросе упоминается высокая производительность или enterprise-месседжинг → выбирайте AMQP.
```csharp
// Uses AMQP by default
await using var client = new ServiceBusClient(connectionString);
```

### HTTP / REST

**HTTP-базированный протокол** для выполнения REST-операций с очередями и топиками.

---

## Преимущества:

- **Проходит через firewall**  
  Использует стандартный порт 443 (HTTPS).

- **Проще для отладки**  
  Можно использовать обычные HTTP-инструменты (Postman, curl и т.д.).

- **Совместимость**  
  Работает с любым HTTP-клиентом, независимо от платформы.

---

## Ограничения:

- **Более высокий overhead**, чем у AMQP  
  Каждый запрос содержит HTTP-заголовки и дополнительную служебную информацию.

- **Нет long-polling**  
  Используется короткий polling, что менее эффективно при высокой нагрузке.

- **Новое соединение на каждый запрос**  
  Нет постоянного TCP-соединения, как в AMQP.

---

## Когда использовать

- Простые сценарии
- Ограничения сети (только HTTPS)
- Отладка и тестирование
- Интеграции без специализированных SDK

---

## Экзаменационный акцент (AZ-204)

- AMQP — предпочтительный протокол для production
- HTTP — допустим, но менее эффективен
- Если важна производительность и долгоживущие соединения → выбирайте AMQP

Главная идея:  
HTTP — проще  
AMQP — эффективнее и производительнее

```bash
# REST API example
curl -X POST "https://myNamespace.servicebus.windows.net/myQueue/messages" \
  -H "Authorization: SharedAccessSignature sr=..." \
  -H "Content-Type: application/json" \
  -d '{"body": "Hello World"}'
```

### JMS 2.0 (Premium Only)

**Java Message Service** - Java/Jakarta EE standard API.

**Benefits:**
- Java/Jakarta EE compliance
- Portable across JMS providers
- Enterprise Java integration

```java
// JMS 2.0 with Service Bus
ConnectionFactory factory = new ServiceBusJmsConnectionFactory(connectionString);
Connection connection = factory.createConnection();
Session session = connection.createSession();

Queue queue = session.createQueue("myQueue");
MessageProducer producer = session.createProducer(queue);

TextMessage message = session.createTextMessage("Hello World");
producer.send(message);
```

---

## Client Libraries

### .NET (Azure.Messaging.ServiceBus)

```bash
dotnet add package Azure.Messaging.ServiceBus
```

```csharp
await using var client = new ServiceBusClient(connectionString);

// Send
var sender = client.CreateSender(queueName);
await sender.SendMessageAsync(new ServiceBusMessage("Hello"));

// Receive
var receiver = client.CreateReceiver(queueName);
var message = await receiver.ReceiveMessageAsync();
await receiver.CompleteMessageAsync(message);
```

### Java (azure-messaging-servicebus)

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-messaging-servicebus</artifactId>
    <version>7.13.0</version>
</dependency>
```

```java
ServiceBusClient client = new ServiceBusClientBuilder()
    .connectionString(connectionString)
    .buildClient();

// Send
ServiceBusSender sender = client.createSender(queueName);
sender.sendMessage(new ServiceBusMessage("Hello"));

// Receive
ServiceBusReceiver receiver = client.createReceiver(queueName);
ServiceBusReceivedMessage message = receiver.receiveMessages(1).iterator().next();
receiver.complete(message);
```

### Python (azure-servicebus)

```bash
pip install azure-servicebus
```

```python
from azure.servicebus import ServiceBusClient, ServiceBusMessage

client = ServiceBusClient.from_connection_string(connection_string)

# Send
with client.get_queue_sender(queue_name) as sender:
    sender.send_messages(ServiceBusMessage("Hello"))

# Receive
with client.get_queue_receiver(queue_name) as receiver:
    for message in receiver:
        print(message)
        receiver.complete_message(message)
```

### JavaScript/TypeScript (@azure/service-bus)

```bash
npm install @azure/service-bus
```

```typescript
import { ServiceBusClient } from "@azure/service-bus";

const client = new ServiceBusClient(connectionString);

// Send
const sender = client.createSender(queueName);
await sender.sendMessages({ body: "Hello" });

// Receive
const receiver = client.createReceiver(queueName);
const messages = await receiver.receiveMessages(1);
await receiver.completeMessage(messages[0]);
```

---

## Best Practices

### 1. Connection Management

✅ **Reuse ServiceBusClient**
```csharp
// ✅ Good: Singleton client
private static ServiceBusClient _client = new ServiceBusClient(connectionString);

// ❌ Bad: Create new client per operation
var client = new ServiceBusClient(connectionString); // Don't do this repeatedly
```

### 2. Error Handling

✅ **Implement retry logic**
```csharp
var options = new ServiceBusClientOptions
{
    RetryOptions = new ServiceBusRetryOptions
    {
        MaxRetries = 3,
        Delay = TimeSpan.FromSeconds(1),
        MaxDelay = TimeSpan.FromSeconds(30),
        Mode = ServiceBusRetryMode.Exponential
    }
};

var client = new ServiceBusClient(connectionString, options);
```

### 3. Message Size

✅ **Keep messages small** (< 256 KB)
```csharp
// ✅ Good: Store large data externally
var message = new ServiceBusMessage(JsonSerializer.Serialize(new
{
    OrderId = order.Id,
    BlobUrl = "https://storage.blob.core.windows.net/orders/123"
}));

// ❌ Bad: Embed large data in message (unless Premium tier)
var message = new ServiceBusMessage(largeByteArray); // > 256 KB
```

### 4. Idempotency

✅ **Design for duplicate processing**
```csharp
// ✅ Good: Idempotent processing
var orderId = message.MessageId;
if (!await _orderRepository.ExistsAsync(orderId))
{
    await ProcessOrderAsync(orderId);
}
```

### 5. Monitoring

✅ **Enable diagnostics and metrics**
```bash
az monitor diagnostic-settings create \
  --resource /subscriptions/.../namespaces/myNamespace \
  --name myDiagnostics \
  --logs '[{"category": "OperationalLogs", "enabled": true}]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]' \
  --workspace /subscriptions/.../workspaces/myWorkspace
```

### 6. Scaling

✅ **Use multiple receivers for parallel processing**
```csharp
// Scale out with multiple receivers
var tasks = Enumerable.Range(0, 10).Select(async i =>
{
    var receiver = client.CreateReceiver(queueName);
    await ProcessMessagesAsync(receiver);
});

await Task.WhenAll(tasks);
```

---

## Советы для экзамена AZ-204

### Ключевые концепции

1. **Service Bus = Enterprise messaging**  
   FIFO, транзакции, pub/sub, DLQ, фильтрация.

2. **Queues = Point-to-point**  
   Одно сообщение → один получатель.

3. **Topics = Publish-subscribe**  
   Одно сообщение → несколько независимых подписчиков.

4. **Sessions = FIFO порядок**  
   Гарантируют последовательную обработку внутри одной сессии.

5. **Premium = Выделенные ресурсы**  
   Предсказуемая производительность и высокая нагрузка.

---

## Что нужно запомнить

| Возможность | Требует |
|-------------|---------|
| **Topics / Subscriptions** | Standard или Premium |
| **FIFO порядок** | Включённые Sessions |
| **Транзакции** | Standard или Premium |
| **Duplicate Detection** | Standard или Premium |
| **Сообщения до 100 MB** | Только Premium |
| **Geo-Disaster Recovery** | Только Premium |
| **JMS 2.0** | Только Premium |

---

## Типовые экзаменационные сценарии

- **Обработка строго по порядку** → Sessions + SessionId
- **Рассылка событий нескольким сервисам** → Topics + Subscriptions
- **Mission-critical система** → Premium tier
- **Большие сообщения (>256 KB)** → Premium tier
- **Развязать приложения** → Queues или Topics
- **Фильтрация сообщений** → Topic Subscriptions + SQL filters

---

## Экзаменационный ориентир

Если в вопросе встречаются:

- "ordered processing"
- "transactions"
- "duplicate detection"
- "filtering"
- "enterprise messaging"

→ Почти всегда это **Service Bus (Standard или Premium)**.

Если акцент на:

- простоте
- низкой стоимости
- базовой очереди

→ вероятнее всего **Queue Storage**.

Главное правило:  
Выбирайте минимальный тариф, который покрывает требования.
### Quick Reference

```csharp
// Send
var client = new ServiceBusClient(connectionString);
var sender = client.CreateSender(queueOrTopic);
await sender.SendMessageAsync(new ServiceBusMessage("data"));

// Receive
var receiver = client.CreateReceiver(queueOrSubscription);
var message = await receiver.ReceiveMessageAsync();
await receiver.CompleteMessageAsync(message);

// Session
var sessionReceiver = await client.AcceptSessionAsync(queue, sessionId);
var message = await sessionReceiver.ReceiveMessageAsync();
```

---

## Итог

**Azure Service Bus** — полностью управляемый enterprise-брокер сообщений, предоставляющий:

- ✅ Надёжную доставку сообщений
- ✅ Очереди (point-to-point) и топики (publish-subscribe)
- ✅ Три тарифа: Basic, Standard, Premium
- ✅ Расширенные возможности: sessions, транзакции, duplicate detection
- ✅ Поддержку нескольких протоколов: AMQP 1.0, HTTP/REST, JMS 2.0
- ✅ Enterprise-функции: geo-DR, private endpoints, RBAC

---

## Когда использовать Service Bus

- Развязка приложений (decoupling)
- Сглаживание нагрузки (load leveling)
- Асинхронная обработка
- Event-driven архитектура
- Надёжные распределённые системы

---

## Главное для AZ-204

Service Bus =  
надежность + порядок + транзакции + pub/sub + enterprise-возможности.

Если задача требует чего-то больше, чем просто «простая очередь» — скорее всего, это Service Bus.