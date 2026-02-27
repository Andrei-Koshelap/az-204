# Изучение Azure Queue Storage

## Что такое Azure Queue Storage?

**Azure Queue Storage** — это сервис для хранения большого количества сообщений, к которым можно получить доступ из любой точки через аутентифицированные HTTP или HTTPS-запросы.

Он предоставляет простое и экономичное решение для реализации асинхронного обмена сообщениями между компонентами системы.

---

## Основные характеристики

- Хранение миллионов сообщений
- Доступ через REST API
- Поддержка SDK для различных языков
- Интеграция с Azure Storage Account
- Поддержка масштабируемых облачных приложений

---

## Для чего используется

Azure Queue Storage подходит для:

- Разделения компонентов системы (decoupling)
- Асинхронной обработки задач
- Фоновых заданий
- Очередей обработки заказов
- Балансировки нагрузки

---

## Как это работает

1. Producer отправляет сообщение в очередь
2. Сообщение сохраняется в Storage Account
3. Consumer считывает сообщение
4. После успешной обработки сообщение удаляется

---

## Важные особенности

- Сообщения хранятся в текстовом формате (до 64 КБ)
- Поддерживается visibility timeout
- Поддерживается TTL (время жизни сообщения)
- Обеспечивается как минимум однократная доставка (at-least-once delivery)

---

## Что важно для AZ-204

- Azure Queue Storage — часть Azure Storage
- Используется для простых сценариев очередей
- Не поддерживает сложную маршрутизацию (в отличие от Service Bus)
- Хороший выбор для дешёвых и масштабируемых фоновых задач

---

## Ключевая идея

Azure Queue Storage — это простой механизм очередей для:

- асинхронной обработки
- масштабирования
- снижения связности между сервисами

Для более сложных сценариев (транзакции, ordering, topics, dead-letter queues) чаще используется Azure Service Bus.

### Key Characteristics

```
Azure Queue Storage Architecture
═══════════════════════════════════════════════════════════

Storage Account: mystorageaccount
├── Blob Storage
├── File Storage
├── Table Storage
└── Queue Storage
    ├── Queue 1: orders
    │   ├── Message 1 (up to 64 KB)
    │   ├── Message 2
    │   └── Message N
    ├── Queue 2: notifications
    │   ├── Message 1
    │   └── Message N
    └── Queue N: tasks
        ├── Message 1
        └── Message N

URL Format:
https://<storage-account>.queue.core.windows.net/<queue-name>

Example:
https://mystorageaccount.queue.core.windows.net/orders
```

## Основные возможности (Core Capabilities)

| Возможность | Описание |
|-------------|----------|
| **Простая модель очереди** | Базовая FIFO-очередь (строгий порядок не гарантируется) |
| **Доступ по HTTP/HTTPS** | REST API доступен из любой точки |
| **Масштабируемое хранилище** | Хранение миллионов сообщений (до 500 ТБ на аккаунт) |
| **Размер сообщения** | До 64 КБ на одно сообщение |
| **Экономичность** | Низкая стоимость при больших объёмах |
| **Visibility Timeout** | Временное скрытие сообщения во время обработки |
| **Peek без блокировки** | Просмотр сообщений без извлечения |

---

## Пояснения к возможностям

### 🔹 Простая модель очереди
- Сообщения обрабатываются приблизительно в порядке FIFO
- Однако строгая гарантия порядка отсутствует
- Подходит для сценариев, где порядок не критичен

---

### 🔹 Visibility Timeout
Когда consumer получает сообщение:

- Оно становится невидимым для других consumers
- Если обработка успешна — сообщение удаляется
- Если не удалено до истечения timeout — снова становится доступным

📌 Это обеспечивает модель доставки **at-least-once**.

---

### 🔹 Peek без извлечения
Позволяет:

- Просматривать сообщения
- Не менять их статус
- Использовать для мониторинга или диагностики

---

## Что важно для AZ-204

- Максимальный размер сообщения — 64 КБ
- Нет строгой гарантии порядка
- Поддерживается visibility timeout
- Используется модель at-least-once delivery
- Экономичное решение для простых очередей

---

## Ключевая идея

Azure Queue Storage — это:

- простая
- дешёвая
- масштабируемая

очередь для фоновых задач и асинхронной обработки,  
без сложной логики маршрутизации и транзакционных гарантий.

---

## Queue Storage Components

### 1. Storage Account

A **storage account** provides a unique namespace for your Azure Storage data.

**URL format:**
```
https://<storage-account-name>.queue.core.windows.net
```

**Account types:**
| Account Type | Performance | Redundancy Options | Use Case |
|--------------|-------------|-------------------|----------|
| **Standard (General-purpose v2)** | Standard | LRS, ZRS, GRS, RA-GRS, GZRS, RA-GZRS | Most scenarios |
| **Premium (Block blobs)** | Premium | LRS, ZRS | Low-latency scenarios |

```bash
# Create storage account
az storage account create \
  --name mystorageaccount \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard_LRS \
  --kind StorageV2
```

### 2. Queue

A **queue** contains a set of messages. All messages must be in a queue.

**Queue naming rules:**
- ✅ Lowercase letters, numbers, hyphens
- ✅ 3-63 characters
- ✅ Must start with letter or number
- ❌ No uppercase, underscores, or special characters

**URL format:**
```
https://<storage-account>.queue.core.windows.net/<queue-name>
```

**Examples:**
```
https://mystorageaccount.queue.core.windows.net/orders
https://mystorageaccount.queue.core.windows.net/notifications
https://mystorageaccount.queue.core.windows.net/image-processing-tasks
```

```bash
# Create queue
az storage queue create \
  --name orders \
  --account-name mystorageaccount \
  --account-key <storage-account-key>
```

### 3. Message

A **message** is a piece of data in any format, up to 64 KB in size.

**Message characteristics:**
| Property | Value |
|----------|-------|
| **Max size** | 64 KB |
| **Format** | String (UTF-8) or byte array |
| **Default TTL** | 7 days |
| **Max TTL** | 7 days (messages created before 2017-07-29)<br>Unlimited (messages created after 2017-07-29) |
| **Visibility timeout** | 30 seconds (default) |

**Message lifecycle:**

```
1. Enqueue         2. Dequeue           3. Process       4. Delete
┌────────┐        ┌────────┐           ┌──────┐        ┌────────┐
│Message │───────>│Visible │──────────>│Invisi│───────>│Deleted │
│Created │        │30s timer│          │ble   │        │        │
└────────┘        └────────┘           └──────┘        └────────┘
                       │                    │
                       │                    └─> If not deleted:
                       │                        Message becomes
                       └─────────────────────> visible again
                                               (auto retry)

Steps:
1. Producer sends message → Queue (visible)
2. Consumer receives message → Queue (invisible for 30s)
3. Consumer processes message
4. Consumer deletes message → Removed from queue

If consumer crashes before step 4:
→ Message becomes visible again after 30s timeout
→ Another consumer can process it (automatic retry)
```

---

## Queue Storage vs Service Bus

### Быстрое сравнение

| Возможность | Queue Storage | Service Bus Queue |
|--------------|---------------|-------------------|
| **Макс. размер сообщения** | 64 КБ | 256 КБ (Standard)<br>100 МБ (Premium) |
| **Макс. размер очереди** | 500 ТБ | Практически неограничен |
| **Гарантия порядка** | ❌ Нет (best-effort) | ✅ Да (при использовании sessions) |
| **Гарантия доставки** | At-least-once | At-least-once или at-most-once |
| **Протокол** | HTTP/HTTPS | AMQP, HTTP, SBMP |
| **Транзакции** | ❌ Нет | ✅ Да |
| **Обнаружение дубликатов** | ❌ Нет | ✅ Да |
| **Dead-Letter Queue** | ❌ Нет (вручную) | ✅ Да (автоматически) |
| **Publish-Subscribe** | ❌ Нет | ✅ Да (topics) |
| **Message Sessions** | ❌ Нет | ✅ Да |
| **TTL** | 7 дней (по умолчанию), можно без ограничения | Без ограничения |
| **Стоимость** | Низкая (~$0.05 за ГБ/мес.) | Оплата за операции или фиксированная (Premium) |
| **Лучше подходит для** | Простые и дешёвые очереди | Enterprise-месседжинг |

---

## Матрица выбора

| Требование | Queue Storage | Service Bus |
|--------------|----------------|--------------|
| Нужна простая очередь | ✅ Да | Избыточно |
| Важна минимальная стоимость | ✅ Да | Дороже |
| Нужно хранить > 80 ГБ | ✅ Да | Дорого при большом объёме |
| Требуется строгий FIFO | ❌ Нет | ✅ Да (sessions) |
| Нужен pub/sub | ❌ Нет | ✅ Да (topics) |
| Нужны транзакции | ❌ Нет | ✅ Да |
| Сообщения > 64 КБ | ❌ Нет | ✅ Да |
| Нужна дедупликация | ❌ Нет | ✅ Да |

---

# Возможности Queue Storage

## 1. Message TTL (Time-To-Live)

- **По умолчанию:** 7 дней
- **Настраивается:**
    - От 1 секунды до 7 дней (старый API)
    - Без ограничения (новый API)

Если сообщение не обработано до истечения TTL — оно автоматически удаляется.

---

## Что важно для AZ-204

- Queue Storage дешевле и проще
- Service Bus предоставляет расширенные возможности
- FIFO в Queue Storage не гарантируется
- Dead-letter и дедупликация есть только в Service Bus
- Если в вопросе фигурируют транзакции, sessions или topics — почти всегда правильный ответ Service Bus

---

## Ключевая идея

Выбор зависит от требований:

- Простая, дешёвая, масштабируемая очередь → Queue Storage
- Сложная корпоративная интеграция → Service Bus

Понимание различий между этими сервисами — одна из самых частых тем на AZ-204.

```csharp
// Send message with custom TTL
await queueClient.SendMessageAsync(
    "Message content",
    timeToLive: TimeSpan.FromHours(1));  // Expires in 1 hour

// Send message with unlimited TTL (-1)
await queueClient.SendMessageAsync(
    "Important message",
    timeToLive: TimeSpan.FromSeconds(-1));  // Never expires
```

### 2. Visibility Timeout

**Concept:** When a message is received, it becomes **invisible** to other consumers for a specified time (default 30 seconds).

**Why?** Prevents multiple consumers from processing the same message simultaneously.

```
Consumer 1               Queue                    Consumer 2
┌────────┐              ┌──────┐                 ┌────────┐
│        │──Receive────>│ Msg  │                 │        │
│        │              │(invisible              │        │
│        │              │30 sec)│                 │        │
│        │              │      │──Receive────────│Can't   │
│        │              │      │   (no messages) │see msg │
└────────┘              └──────┘                 └────────┘
     │                       │
     │                       │ After 30 seconds:
     └──Delete───────────────│ If not deleted, message
                             │ becomes visible again
```

**Default:** 30 seconds
**Max:** 7 days

```csharp
// Receive message with custom visibility timeout
var messages = await queueClient.ReceiveMessagesAsync(
    maxMessages: 1,
    visibilityTimeout: TimeSpan.FromMinutes(5));  // Invisible for 5 minutes
```

### 3. Peek Messages

**View messages** without removing them or making them invisible.

```csharp
// Peek next message (doesn't affect visibility)
var peekedMessages = await queueClient.PeekMessagesAsync(maxMessages: 10);

foreach (var message in peekedMessages)
{
    Console.WriteLine($"Peeked: {message.Body}");
    // Message still visible to other consumers
}
```

### 4. Update Message

**Modify message content** and extend visibility timeout during processing.

```csharp
// Receive message
var messages = await queueClient.ReceiveMessagesAsync(1);
var message = messages.Value[0];

// Update message content and extend visibility
await queueClient.UpdateMessageAsync(
    message.MessageId,
    message.PopReceipt,
    "Updated content",
    visibilityTimeout: TimeSpan.FromMinutes(5));  // Extend by 5 minutes
```

### 5. Dequeue Count

**Track delivery attempts** using `DequeueCount` property.

```csharp
var messages = await queueClient.ReceiveMessagesAsync(1);
var message = messages.Value[0];

Console.WriteLine($"Delivery attempt: {message.DequeueCount}");

if (message.DequeueCount > 5)
{
    // Give up after 5 retries
    // Move to poison message queue
    await poisonQueueClient.SendMessageAsync(message.Body.ToString());
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
}
```

### 6. Approximate Message Count

**Get queue length** (approximate count of messages).

```csharp
var properties = await queueClient.GetPropertiesAsync();
int approximateMessageCount = properties.Value.ApproximateMessagesCount;

Console.WriteLine($"Approx messages in queue: {approximateMessageCount}");
```

---

## Access Control and Security

### Authentication Methods

#### 1. Storage Account Key (Shared Key)

**Full access** to storage account.

```csharp
using Azure.Storage.Queues;

string connectionString = "DefaultEndpointsProtocol=https;" +
    "AccountName=mystorageaccount;" +
    "AccountKey=<account-key>;" +
    "EndpointSuffix=core.windows.net";

var queueClient = new QueueClient(connectionString, "orders");
```

**Get account key:**
```bash
az storage account keys list \
  --resource-group myResourceGroup \
  --account-name mystorageaccount \
  --query "[0].value" \
  --output tsv
```

#### 2. Shared Access Signature (SAS)

**Limited access** with specific permissions and expiration.

```bash
# Generate SAS token for queue
az storage queue generate-sas \
  --name orders \
  --account-name mystorageaccount \
  --account-key <account-key> \
  --permissions raup \
  --expiry 2024-12-31T23:59:59Z \
  --output tsv
```

**SAS permissions:**
- `r` - Read (peek, receive)
- `a` - Add (send messages)
- `u` - Update (update messages)
- `p` - Process (receive and delete messages)

```csharp
// Use SAS token
string sasUrl = "https://mystorageaccount.queue.core.windows.net/orders?<sas-token>";
var queueClient = new QueueClient(new Uri(sasUrl));
```

#### 3. Azure AD (RBAC)

**Identity-based access** using Azure Active Directory.

```csharp
using Azure.Identity;
using Azure.Storage.Queues;

// Use default Azure credential (managed identity, Azure CLI, etc.)
var credential = new DefaultAzureCredential();
var queueClient = new QueueClient(
    new Uri("https://mystorageaccount.queue.core.windows.net/orders"),
    credential);
```

**Azure RBAC roles:**
- `Storage Queue Data Contributor`: Read, write, delete messages
- `Storage Queue Data Reader`: Read and peek messages
- `Storage Queue Data Message Processor`: Peek, receive, delete messages
- `Storage Queue Data Message Sender`: Send messages only

```bash
# Assign role
az role assignment create \
  --role "Storage Queue Data Contributor" \
  --assignee <user-or-service-principal> \
  --scope /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount
```

---

## Monitoring and Logging

### Storage Analytics

**Enable logging and metrics:**

```bash
# Enable Storage Analytics logging
az storage logging update \
  --account-name mystorageaccount \
  --account-key <account-key> \
  --services q \
  --log rwd \
  --retention 7

# Enable metrics
az storage metrics update \
  --account-name mystorageaccount \
  --account-key <account-key> \
  --services q \
  --hour true \
  --minute false \
  --retention 7
```

**Logs include:**
- Authenticated requests
- Anonymous requests
- Success/failure status
- Error codes
- Request/response details

### Azure Monitor Integration

```bash
# Create diagnostic settings
az monitor diagnostic-settings create \
  --resource /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.Storage/storageAccounts/mystorageaccount/queueServices/default \
  --name "queue-diagnostics" \
  --logs '[{"category": "StorageRead", "enabled": true}, {"category": "StorageWrite", "enabled": true}]' \
  --metrics '[{"category": "Transaction", "enabled": true}]' \
  --workspace /subscriptions/<subscription-id>/resourceGroups/myResourceGroup/providers/Microsoft.OperationalInsights/workspaces/myWorkspace
```

---

## Best Practices

### 1. Use Poison Message Queue

**Handle messages that fail repeatedly:**

```csharp
var message = messages.Value[0];

if (message.DequeueCount > 5)
{
    // Move to poison message queue
    await poisonQueueClient.SendMessageAsync(message.Body.ToString());
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
    Console.WriteLine("Moved to poison queue");
}
```

### 2. Batch Operations

**Improve performance with batching:**

```csharp
// ✅ Good: Batch receive
var messages = await queueClient.ReceiveMessagesAsync(maxMessages: 32);

// ❌ Bad: Single message receive in loop
for (int i = 0; i < 32; i++)
{
    var message = await queueClient.ReceiveMessageAsync();  // 32 round trips!
}
```

### 3. Reuse QueueClient

```csharp
// ✅ Good: Reuse client
private static QueueClient _queueClient = new QueueClient(connectionString, queueName);

// ❌ Bad: Create new client per operation
var client = new QueueClient(connectionString, queueName);  // Don't repeat
```

### 4. Set Appropriate Visibility Timeout

```csharp
// ✅ Match timeout to processing time
var messages = await queueClient.ReceiveMessagesAsync(
    maxMessages: 1,
    visibilityTimeout: TimeSpan.FromMinutes(5));  // 5 minutes to process
```

### 5. Implement Exponential Backoff

```csharp
int retryCount = 0;
while (retryCount < 5)
{
    var messages = await queueClient.ReceiveMessagesAsync(1);
    
    if (messages.Value.Length == 0)
    {
        // Exponential backoff: 1s, 2s, 4s, 8s, 16s
        await Task.Delay(TimeSpan.FromSeconds(Math.Pow(2, retryCount)));
        retryCount++;
        continue;
    }
    
    // Process message
    break;
}
```

### 6. Monitor Queue Depth

```csharp
// Alert if queue grows too large
var properties = await queueClient.GetPropertiesAsync();
int count = properties.Value.ApproximateMessagesCount;

if (count > 10000)
{
    // Alert: Queue backlog too large, scale out consumers
    await SendAlertAsync($"Queue depth: {count}");
}
```

---

# Советы к экзамену AZ-204 (Azure Queue Storage)

## Ключевые концепции

1. **Queue Storage** — простая и экономичная очередь
2. **Максимальный размер сообщения** — 64 КБ
3. **Максимальный размер очереди** — до 500 ТБ на storage account
4. **Visibility timeout** — по умолчанию 30 секунд
5. **TTL** — 7 дней по умолчанию, возможно без ограничения
6. **FIFO не гарантируется** — порядок best-effort

---

## Что обязательно помнить

| Характеристика | Queue Storage |
|----------------|---------------|
| **Макс. размер сообщения** | 64 КБ |
| **Макс. размер очереди** | 500 ТБ на аккаунт |
| **Гарантия порядка** | Нет (best-effort) |
| **Гарантия доставки** | At-least-once |
| **TTL** | 7 дней (по умолчанию), возможно без ограничения |
| **Visibility timeout** | 30 секунд (по умолчанию) |
| **Протокол** | HTTP/HTTPS |
| **Лучше подходит для** | Простых и высоконагруженных очередей |

---

## Типовые сценарии

- Нужна дешёвая очередь → **Queue Storage**
- Нужен большой объём хранения → **Queue Storage**
- Простой producer-consumer → **Queue Storage**
- Нужно отслеживать количество повторных попыток → `DequeueCount`
- Нужно предотвратить двойную обработку → Visibility timeout
- Обработка ошибок → Poison message queue

---

# Итог

**Azure Queue Storage предоставляет:**

- ✅ Простую HTTP/HTTPS очередь
- ✅ Огромный объём хранения (до 500 ТБ)
- ✅ Экономичное решение
- ✅ Сообщения до 64 КБ
- ✅ Visibility timeout для безопасной обработки
- ✅ Возможность просмотра сообщений (peek) без удаления
- ✅ Несколько методов аутентификации (ключ, SAS, Azure AD)

---

## Главное для AZ-204

Используйте **Queue Storage**, если требуется:

- простая
- дешёвая
- масштабируемая очередь

Используйте **Service Bus**, если нужны:

- строгий FIFO
- pub/sub
- транзакции
- дедупликация
- автоматический dead-letter

Понимание различий между этими сервисами — частая тема экзамена AZ-204.