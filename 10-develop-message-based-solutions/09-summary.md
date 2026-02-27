# Итог: Разработка решений на основе сообщений

## Обзор

В этом модуле рассмотрены **решения на основе сообщений** в Azure с использованием:

- **Azure Service Bus**
- **Azure Queue Storage**

Вы изучили:

- когда использовать каждый сервис
- как реализовать шаблоны очередей
- лучшие практики построения отказоустойчивых распределённых систем

---

# Azure Service Bus

**Enterprise message broker** с расширенными возможностями надёжной доставки.

---

## Основные компоненты

- **Namespace** — контейнер для очередей и топиков
- **Queues** — point-to-point (competing consumers)
- **Topics** — publish-subscribe (fan-out)
- **Subscriptions** — независимые получатели сообщений топика

---

## Ключевые возможности

- ✅ FIFO (через sessions)
- ✅ Транзакции
- ✅ Обнаружение дубликатов
- ✅ Автоматическая Dead-letter очередь
- ✅ Продвинутая маршрутизация и фильтрация
- ✅ Размер сообщения:
   - 256 КБ (Standard)
   - 100 МБ (Premium)
- ✅ Протоколы: AMQP 1.0, HTTP/REST, JMS 2.0

---

## Уровни (Tiers)

| Tier | Возможности | Сценарий |
|------|-------------|----------|
| **Basic** | Только очереди, 256 КБ | Dev/test |
| **Standard** | Очереди + топики + транзакции | Production |
| **Premium** | Выделенные ресурсы, 100 МБ, geo-DR | Mission-critical |

---

# Azure Queue Storage

**Простая и экономичная** очередь для высоконагруженных сценариев.

---

## Основные компоненты

- **Storage Account** — контейнер для очередей
- **Queue** — хранит сообщения
- **Message** — до 64 КБ данных

---

## Ключевые возможности

- ✅ HTTP/HTTPS протокол
- ✅ Огромный объём хранения (до 500 ТБ на аккаунт)
- ✅ Низкая стоимость
- ✅ Visibility timeout (по умолчанию 30 сек)
- ✅ Peek без извлечения
- ✅ Модель at-least-once

---

## Ограничения

- ❌ Нет строгого FIFO
- ❌ Нет транзакций
- ❌ Нет дедупликации
- ❌ Нет pub/sub
- ❌ Лимит 64 КБ на сообщение

---

# Матрица выбора

## Когда использовать Service Bus

Используйте **Azure Service Bus**, если требуется:

1. **Гарантированный FIFO**
   - Sessions + SessionId

2. **Publish-Subscribe**
   - Topics + несколько subscriptions

3. **Транзакции**
   - Атомарные операции

4. **Обнаружение дубликатов**
   - Автоматическая дедупликация по MessageId

5. **Продвинутая маршрутизация**
   - SQL-фильтры

6. **Сообщения > 64 КБ**
   - До 256 КБ (Standard)
   - До 100 МБ (Premium)

7. **Enterprise-функциональность**
   - Автоматический DLQ
   - Scheduled messages
   - Message deferral
   - Geo-DR

---

## Когда использовать Queue Storage

Используйте **Azure Queue Storage**, если требуется:

1. Простая очередь
2. Минимальная стоимость
3. Большой объём хранения
4. Простая модель retry через DequeueCount
5. HTTP-доступ без сложных протоколов

---

# Сравнительная таблица

| Возможность | Service Bus | Queue Storage |
|--------------|-------------|---------------|
| **Размер сообщения** | 256 КБ / 100 МБ | 64 КБ |
| **Размер очереди** | Практически неограничен | 500 ТБ |
| **FIFO** | ✅ Через sessions | ❌ Нет гарантии |
| **Транзакции** | ✅ Да | ❌ Нет |
| **Дедупликация** | ✅ Да | ❌ Нет |
| **Pub/Sub** | ✅ Topics | ❌ Нет |
| **Dead-letter** | ✅ Автоматически | ❌ Вручную |
| **Протокол** | AMQP, HTTP | HTTP/HTTPS |
| **Стоимость** | Выше | Ниже |
| **Лучше подходит для** | Enterprise | Простые сценарии |

---

# Главное для AZ-204

Если в вопросе фигурируют:

- FIFO
- транзакции
- topics
- дедупликация
- сложная маршрутизация

→ Ответ: **Service Bus**

Если речь о:

- простой очереди
- дешёвом решении
- большом объёме хранения
- базовом producer-consumer

→ Ответ: **Queue Storage**

---

## Ключевая мысль

- **Service Bus** = enterprise messaging
- **Queue Storage** = простая, дешёвая очередь

Умение быстро определить, какой сервис подходит под требования, — один из самых частых типов вопросов на AZ-204.
---

## Common Messaging Patterns

### 1. Competing Consumers (Load Balancing)

**Use queue** to distribute work across multiple consumers.

```
Producer          Queue           Consumers (Competing)
┌────────┐      ┌──────┐         ┌────────┐
│        │─────>│ Msg1 │────────>│ Cons 1 │
│        │      │ Msg2 │─┐       └────────┘
│        │      │ Msg3 │ │       ┌────────┐
│        │      │ Msg4 │ └──────>│ Cons 2 │
└────────┘      │ Msg5 │   ┌────>└────────┘
                └──────┘   │     ┌────────┐
                           └────>│ Cons 3 │
                                 └────────┘

• Each message delivered to ONE consumer
• Horizontal scaling: add more consumers
• Load leveling: smooth out traffic bursts
```

**Implementation:**
- **Service Bus**: Queue with multiple receivers
- **Queue Storage**: Queue with multiple ReceiveMessages calls

### 2. Publish-Subscribe (Fan-Out)

**Use topic** to broadcast messages to multiple subscribers.

```
Publisher         Topic          Subscriptions
┌────────┐      ┌──────┐        ┌─────────────┐
│        │─────>│      │───────>│ Analytics   │
│ Event  │      │Topic │        └─────────────┘
│ Source │      │      │───────>┌─────────────┐
└────────┘      │      │        │Notifications│
                │      │        └─────────────┘
                └──────┘───────>┌─────────────┐
                                │  Archival   │
                                └─────────────┘

• Each subscription receives COPY of message
• Independent processing per subscription
• Filtering per subscription
```

**Implementation:**
- **Service Bus**: Topics with subscriptions
- **Queue Storage**: ❌ Not supported (use multiple queues + code)

### 3. Request-Reply

**Use CorrelationId and ReplyTo** to link request and response.

```
Client                Server              
┌──────┐             ┌──────┐            
│      │──Request───>│      │            
│      │ CorrelationId│      │            
│      │ ReplyTo     │      │            
│      │             │      │            
│      │<──Response──│      │            
│      │ CorrelationId      │            
└──────┘             └──────┘            

1. Client sends request with CorrelationId and ReplyTo
2. Server processes and sends response to ReplyTo queue
3. Client matches response using CorrelationId
```

**Implementation:**
```csharp
// CLIENT: Send request
var request = new ServiceBusMessage("Query");
request.CorrelationId = Guid.NewGuid().ToString();
request.ReplyTo = "clientReplyQueue";
await sender.SendMessageAsync(request);

// SERVER: Send reply
var reply = new ServiceBusMessage("Response");
reply.CorrelationId = request.CorrelationId;
var replySender = client.CreateSender(request.ReplyTo);
await replySender.SendMessageAsync(reply);

// CLIENT: Receive reply
var replyReceiver = client.CreateReceiver("clientReplyQueue");
var response = await replyReceiver.ReceiveMessageAsync();
if (response.CorrelationId == request.CorrelationId) { /* process */ }
```

### 4. Load Leveling

**Use queue as buffer** to smooth out traffic bursts.

```
Traffic Bursts       Queue (Buffer)      Steady Processing
┌────────┐          ┌──────────┐         ┌────────┐
│ Spike  │─────────>│          │────────>│ Stable │
│        │          │ Messages │         │ Worker │
│        │          │          │         │ Pool   │
└────────┘          └──────────┘         └────────┘

Benefits:
• Protects backend from overload
• Queue grows during spikes, drains during lulls
• Workers process at steady rate
```

**Implementation:**
- Web API writes to queue
- Worker service processes from queue at controlled rate

---

## Service Bus Receive Modes

### Peek Lock (Recommended)

**Two-stage receive** for fault-tolerant processing.

```
1. Receive → Lock message (invisible 30s)
2. Process
3. Complete → Delete from queue
   OR Abandon → Requeue for retry
   OR Dead-letter → Move to DLQ
```

**Use when:**
- ✅ Critical data (no loss acceptable)
- ✅ Fault tolerance required
- ✅ Transaction support needed

### Receive and Delete

**One-stage receive** for simple scenarios.

```
1. Receive → Message deleted immediately
```

# Когда использовать (Service Bus / Messaging)

**Использовать, если:**

- ✅ Потеря данных допустима (телеметрия, логи)
- ✅ Критична производительность
- ❌ НЕ использовать для критичных бизнес-данных

📌 Если данные нельзя терять — необходимо использовать механизмы гарантированной доставки, транзакции и дедупликацию (например, Service Bus).

---

# Свойства сообщений (Message Properties)

В Service Bus сообщения имеют два типа свойств:

- **Broker properties (системные)**
- **Application properties (пользовательские)**

---

## Broker Properties (Системные свойства)

Эти свойства определяются брокером сообщений и используются для маршрутизации, управления и обеспечения надёжности.

| Свойство | Назначение | Пример |
|------------|-------------|----------|
| **MessageId** | Уникальный идентификатор (дедупликация) | `"order-12345"` |
| **CorrelationId** | Связывает связанные сообщения | `"correlation-abc"` |
| **SessionId** | Группировка для FIFO | `"customer-123"` |
| **ContentType** | Формат сериализации | `"application/json"` |
| **ReplyTo** | Очередь для ответа | `"replyQueue"` |
| **TimeToLive** | Время жизни сообщения | `TimeSpan.FromHours(1)` |

---

## Пояснения

### 🔹 MessageId
- Используется для обнаружения дубликатов
- Обязателен при включённой duplicate detection

---

### 🔹 CorrelationId
- Применяется для отслеживания цепочек сообщений
- Используется в request-response сценариях

---

### 🔹 SessionId
- Обеспечивает FIFO внутри одной session
- Сообщения с одинаковым SessionId обрабатываются последовательно

---

### 🔹 TimeToLive (TTL)
- Определяет срок жизни сообщения
- После истечения может попасть в Dead-letter queue

---

## Что важно для AZ-204

- FIFO достигается через **SessionId**
- Дедупликация требует **MessageId**
- Request/response — через **ReplyTo + CorrelationId**
- TTL управляет временем жизни сообщения

---

## Ключевая идея

Свойства сообщения позволяют реализовать:

- гарантированный порядок
- дедупликацию
- корреляцию
- автоматическое истечение

Понимание назначения этих свойств часто проверяется в вопросах AZ-204.### User Properties (Application-Defined)

Custom key-value pairs for **filtering and routing**.

```csharp
message.ApplicationProperties["Priority"] = "High";
message.ApplicationProperties["Region"] = "US-West";
message.ApplicationProperties["Amount"] = 599.99;

// Filter: Priority = 'High' AND Region = 'US-West'
```

---

## Best Practices

### 1. Design for Idempotency

**Messages may be delivered more than once** - design for at-least-once delivery.

```csharp
// ✅ Good: Idempotent processing
var orderId = message.MessageId;
if (!await _orderRepository.ExistsAsync(orderId))
{
    await ProcessOrderAsync(orderId);
}
```

### 2. Use Appropriate Message Size

```csharp
// ✅ Good: Store large data externally
var message = new ServiceBusMessage(JsonSerializer.Serialize(new
{
    OrderId = order.Id,
    BlobUrl = "https://storage.blob.core.windows.net/orders/12345"
}));

// ❌ Bad: Embed large data in message
var message = new ServiceBusMessage(largeByteArray);  // > 256 KB
```

### 3. Implement Poison Message Handling

```csharp
// ✅ Service Bus: Automatic dead-lettering
if (message.DeliveryCount >= maxRetries)
{
    await receiver.DeadLetterMessageAsync(message);
}

// ✅ Queue Storage: Manual poison queue
if (message.DequeueCount >= 5)
{
    await poisonQueueClient.SendMessageAsync(message.Body.ToString());
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
}
```

### 4. Set Appropriate Timeouts

```csharp
// ✅ Match timeout to processing time
// Service Bus
var receiver = client.CreateReceiver(queueName);
var message = await receiver.ReceiveMessageAsync(TimeSpan.FromMinutes(5));

// Queue Storage
var messages = await queueClient.ReceiveMessagesAsync(
    maxMessages: 1,
    visibilityTimeout: TimeSpan.FromMinutes(5));
```

### 5. Use Batching for Performance

```csharp
// ✅ Good: Batch operations
var messages = await receiver.ReceiveMessagesAsync(maxMessages: 10);

// ❌ Bad: Single message in loop
for (int i = 0; i < 10; i++)
{
    var message = await receiver.ReceiveMessageAsync();  // 10 round trips!
}
```

### 6. Monitor Queue Depth

```csharp
// Service Bus
var properties = await administrationClient.GetQueueRuntimePropertiesAsync(queueName);
int activeMessages = properties.Value.ActiveMessageCount;
int deadLetterMessages = properties.Value.DeadLetterMessageCount;

// Queue Storage
var properties = await queueClient.GetPropertiesAsync();
int approximateCount = properties.Value.ApproximateMessagesCount;

// Alert if backlog too large
if (activeMessages > 10000)
{
    await SendAlertAsync($"Queue backlog: {activeMessages}");
}
```

### 7. Reuse Clients

```csharp
// ✅ Good: Singleton clients
private static readonly ServiceBusClient _serviceBusClient = 
    new ServiceBusClient(connectionString);

private static readonly QueueClient _queueClient = 
    new QueueClient(connectionString, queueName);

// ❌ Bad: Create new client per operation
var client = new ServiceBusClient(connectionString);  // Don't repeat
```

---

## Security Best Practices

### 1. Use Azure AD Authentication

```csharp
// ✅ Recommended: Azure AD with managed identity
var credential = new DefaultAzureCredential();

var serviceBusClient = new ServiceBusClient(
    "namespace.servicebus.windows.net",
    credential);

var queueClient = new QueueClient(
    new Uri("https://storageaccount.queue.core.windows.net/queue"),
    credential);
```

### 2. Use Least Privilege

**Assign minimal required permissions:**
- **Service Bus**: Data Sender, Data Receiver, Data Owner
- **Queue Storage**: Queue Data Contributor, Reader, Message Processor

### 3. Use SAS Tokens with Expiration

```csharp
// ✅ Time-limited SAS token
var sasUrl = GenerateSasToken(
    expiresOn: DateTimeOffset.UtcNow.AddHours(1));
```

### 4. Enable Encryption

- ✅ Enable HTTPS/TLS for Queue Storage
- ✅ Use AMQP with TLS for Service Bus
- ✅ Enable encryption at rest

---

# Советы к экзамену AZ-204 (Message-Based Solutions)

## Ключевые концепции

1. **Service Bus** = Enterprise-месседжинг  
   (FIFO, транзакции, pub/sub)

2. **Queue Storage** = Простая и экономичная очередь

3. **FIFO** = Только через Service Bus sessions

4. **Pub/Sub** = Только через Service Bus topics

5. **Размер сообщения:**
   - 64 КБ → Queue Storage
   - 256 КБ → Service Bus (Standard)
   - 100 МБ → Service Bus (Premium)

6. **Peek-Lock** = Двухэтапная обработка (рекомендуется)
   - Receive → Complete
   - Обеспечивает отказоустойчивость

7. **Sessions** = FIFO-упорядочивание (через SessionId)

8. **CorrelationId** = Связь request/reply

---

# Типовые экзаменационные сценарии

| Сценарий | Решение |
|------------|-----------|
| Нужен FIFO | Service Bus + sessions |
| Рассылка нескольким системам | Service Bus topics |
| Дешёвая простая очередь | Queue Storage |
| Обработка строго по порядку | Service Bus sessions |
| > 80 ГБ хранения | Queue Storage |
| Нужны транзакции | Service Bus |
| Сообщения > 64 КБ | Service Bus |
| Предотвратить дубликаты | Service Bus duplicate detection |
| Обработка недоставленных сообщений | Service Bus DLQ |

---

## Быстрая логика выбора на экзамене

Если в вопросе упоминаются:

- FIFO
- транзакции
- topics
- дедупликация
- dead-letter queue
- correlation

→ Ответ почти всегда **Service Bus**

Если речь идёт о:

- простой очереди
- низкой стоимости
- большом объёме хранения
- базовом producer-consumer

→ Ответ **Queue Storage**

---

## Ключевая мысль для AZ-204

- **Service Bus** = сложная корпоративная интеграция
- **Queue Storage** = простая и дешёвая очередь

Умение быстро отличить требования enterprise-месседжинга от простого сценария — критично для успешной сдачи AZ-204.
### Quick Reference

**Service Bus:**
```csharp
// Send
var client = new ServiceBusClient(connectionString);
var sender = client.CreateSender(queueName);
await sender.SendMessageAsync(new ServiceBusMessage("data"));

// Receive (Peek Lock)
var receiver = client.CreateReceiver(queueName);
var message = await receiver.ReceiveMessageAsync();
await receiver.CompleteMessageAsync(message);

// Session
message.SessionId = "session-123";
var sessionReceiver = await client.AcceptSessionAsync(queueName, "session-123");
```

**Queue Storage:**
```csharp
// Send
var queueClient = new QueueClient(connectionString, queueName);
await queueClient.SendMessageAsync("data");

// Receive
var messages = await queueClient.ReceiveMessagesAsync(1);
var message = messages.Value[0];
await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);

// Extend timeout
await queueClient.UpdateMessageAsync(
    message.MessageId,
    message.PopReceipt,
    visibilityTimeout: TimeSpan.FromMinutes(5));
```

### Remember for Exam

**Decision flowchart:**
```
Need pub/sub? → YES → Service Bus Topics
              → NO → Continue

Need FIFO? → YES → Service Bus with Sessions
           → NO → Continue

Need transactions? → YES → Service Bus
                  → NO → Continue

Message > 64 KB? → YES → Service Bus
                 → NO → Continue

Budget constrained? → YES → Queue Storage
                    → NO → Service Bus
```

---

# Итог: Message-Based Solutions в Azure

## Azure Service Bus

- ✅ Enterprise message broker
- ✅ Очереди (point-to-point) и топики (pub/sub)
- ✅ FIFO через sessions
- ✅ Транзакции, дедупликация, автоматический Dead-letter
- ✅ Три уровня: Basic, Standard, Premium
- ✅ Лучший выбор для enterprise-сценариев

---

## Azure Queue Storage

- ✅ Простая и экономичная очередь
- ✅ HTTP/HTTPS протокол
- ✅ Огромная ёмкость (до 500 ТБ)
- ✅ Visibility timeout для безопасной обработки
- ✅ Лучший выбор для простых и высоконагруженных сценариев

---

# Выбор сервиса по требованиям

- **Нужны расширенные возможности (FIFO, pub/sub, транзакции)** → Service Bus
- **Нужна простая и дешёвая очередь** → Queue Storage

---

# Ключевые архитектурные паттерны

- **Competing Consumers** — балансировка нагрузки между несколькими обработчиками
- **Publish-Subscribe** — fan-out сообщений нескольким подписчикам
- **Request-Reply** — через CorrelationId + ReplyTo
- **Load Leveling** — очередь как буфер для сглаживания нагрузки

---

# Лучшие практики

- Проектировать систему с учётом идемпотентности (at-least-once delivery)
- Обрабатывать poison messages (DLQ или отдельная очередь)
- Мониторить глубину очереди и настраивать алерты
- Настраивать корректные таймауты обработки
- Переиспользовать клиентов для производительности
- Использовать Azure AD для аутентификации

---

## Финальная мысль для AZ-204

- **Service Bus** = корпоративные сценарии с расширенными требованиями
- **Queue Storage** = простые и экономичные решения

Умение правильно сопоставить требования задачи с возможностями сервиса — один из ключевых навыков для успешной сдачи AZ-204.

📌 Как работает geo-filtering в Azure CDN

Geo-filtering позволяет:

ограничивать доступ к контенту

разрешать или блокировать трафик

на основе географического местоположения пользователя

Правила применяются:

к конкретному относительному пути (relative path)

или к рекурсивным папкам

с указанием действия: Allow или Block

И фильтрация выполняется по:

списку стран (countries)

🔎 Почему именно страны

Azure CDN geo-filtering поддерживает:
ISO-коды сран
Allow / Block действия

|                          | CDN              | Front Door    | App Gateway     |
| ------------------------ | ---------------- | ------------- | --------------- |
| Основная цель            | Кэш              | Глобальный LB | Региональный LB |
| География                | Edge worldwide   | Global        | Regional        |
| Кэширование              | Да               | Да            | Нет             |
| WAF                      | Нет (в классике) | Да            | Да              |
| Failover между регионами | Нет              | Да            | Нет             |
| Работает в VNet          | Нет              | Нет           | Да              |

Если нужно:

Быстро отдавать картинки → CDN
Глобальный failover и routing → Front Door
Балансировка внутри VNet → App Gateway


Перенос в Azure Web Apps позволяет:
Настраивать HTTP headers
Добавлять security headers
Использовать web.config / app settings
Полный контроль над response
Это полноценная замена static hosting.

**You're now ready to build reliable, scalable message-based solutions in Azure!** 🎉