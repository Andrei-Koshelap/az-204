# Выбор решения для очередей сообщений

## Обзор технологий очередей сообщений

Azure предоставляет две основные технологии очередей для построения надёжных и слабо связанных (decoupled) приложений:

| Технология | Тип | Лучше всего подходит для | Макс. размер сообщения | Макс. размер очереди |
|-----------|-----|--------------------------|------------------------|----------------------|
| **Azure Service Bus** | Enterprise message broker | Enterprise messaging, сложная маршрутизация | 256 KB (Standard)<br>100 MB (Premium) | Не ограничен (практически) |
| **Azure Queue Storage** | Простая очередь | Простые очереди, большой объём | 64 KB | До 500 TB на storage account |

---

## Архитектурный смысл

- **Service Bus** — когда нужны гарантии, маршрутизация, сложные сценарии обработки и enterprise-возможности.
- **Queue Storage** — когда нужна простая и дешёвая очередь для больших объёмов сообщений без сложной логики.

---

## Важно для AZ-204

Часто проверяют:

- лимит 64 KB у Queue Storage
- лимит 256 KB (Standard) и до 100 MB (Premium) у Service Bus
- Service Bus — enterprise брокер
- Queue Storage — простая очередь в Storage

Если в вопросе упоминаются «topics/subscriptions», «dead-letter», «sessions», «transactions» — это почти всегда Service Bus.
### Key Differences

```
Service Bus                          Queue Storage
├── Enterprise features              ├── Simple and cost-effective
├── FIFO guarantee (sessions)        ├── At-least-once delivery
├── Topics/subscriptions (pub/sub)   ├── Simple queue model only
├── Advanced routing and filtering   ├── No advanced features
├── Transactions                     ├── HTTP/HTTPS only
├── AMQP 1.0 protocol                ├── Peek without locking
└── Higher cost                      └── Lower cost
```

---

## Когда использовать Service Bus Queues

### Основные сценарии

✅ **Используйте Service Bus Queues, если требуется:**

---

### 1️⃣ Гарантия FIFO (First-In-First-Out)

- **Сценарий**: Система обработки заказов, где порядок критичен
- **Функция**: Message Sessions обеспечивают строгую последовательность
- **Пример**: Конвейер обработки заказов в e-commerce

📌 Важно: FIFO работает при включённых sessions.

---

### 2️⃣ Автоматическое обнаружение дубликатов

- **Сценарий**: Предотвращение повторной обработки транзакций
- **Функция**: Duplicate detection по `MessageId`
- **Пример**: Платёжная система без двойных списаний

---

### 3️⃣ Транзакционное поведение

- **Сценарий**: Атомарные операции с несколькими сообщениями
- **Функция**: Отправка/получение нескольких сообщений в рамках одной транзакции
- **Пример**: Финансовые операции

📌 Поддержка транзакций — ключевое отличие от Storage Queues.

---

### 4️⃣ Долгоживущие параллельные потоки

- **Сценарий**: Обработка связанных сообщений как единого потока
- **Функция**: Message Sessions для группировки
- **Пример**: Многошаговый workflow

---

### 5️⃣ Role-Based Access Control (RBAC)

- **Сценарий**: Разные права доступа для разных приложений
- **Функция**: Интеграция с Azure AD и RBAC
- **Пример**: Несколько команд работают с одной очередью

---

### 6️⃣ Long Polling (Push-модель)

- **Сценарий**: Немедленное получение сообщений
- **Функция**: TCP-based long polling
- **Пример**: Система real-time уведомлений

---

### 7️⃣ Publish/Subscribe

- **Сценарий**: Рассылка сообщений нескольким получателям
- **Функция**: Topics и Subscriptions с фильтрацией
- **Пример**: Система событий с несколькими потребителями

⚠️ Для 1:N используйте Topic, а не Queue.

---

### 8️⃣ Продвинутая маршрутизация сообщений

- **Сценарий**: Маршрутизация по свойствам сообщения
- **Функция**: Subscription filters и actions
- **Пример**: Multi-tenant система

---

### 9️⃣ Поддержка больших сообщений

- **Сценарий**: Сообщения от 64 KB до 256 KB (до 100 MB в Premium)
- **Функция**: Premium tier поддерживает до 100 MB
- **Пример**: Обработка документов

---

### 🔟 Dead-Letter Queue (DLQ)

- **Сценарий**: Обработка недоставленных сообщений
- **Функция**: Автоматическое перемещение в DLQ
- **Пример**: Повторная обработка или анализ ошибок

---

## Матрица выбора: Queue vs Topic

| Требование | Service Bus Queue | Service Bus Topic |
|-------------|------------------|-------------------|
| **1:1 взаимодействие** | ✅ Да | ❌ Используйте очередь |
| **1:N взаимодействие** | ❌ Используйте topic | ✅ Да |
| **FIFO (сессии)** | ✅ Да | ✅ Да |
| **Competing consumers** | ✅ Да | ✅ Да (на уровне подписки) |
| **Фильтрация сообщений** | ❌ Ограничено | ✅ Расширенная |
| **Duplicate detection** | ✅ Да | ✅ Да |
| **Транзакции** | ✅ Да | ✅ Да |

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:

- Требуется строгий порядок → Queue с sessions
- Нужна фильтрация и 1:N → Topic
- Требуются транзакции → Service Bus
- Нужна DLQ → Service Bus
- Нужен простой и дешёвый вариант → возможно Storage Queue

Главный принцип:  
Queue — 1:1  
Topic — 1:N  
Sessions → порядок  
Premium → большие сообщения  
DLQ → обработка ошибок
### Code Example: Service Bus Queue

```csharp
using Azure.Messaging.ServiceBus;

// Create client
await using ServiceBusClient client = new ServiceBusClient(connectionString);
ServiceBusSender sender = client.CreateSender(queueName);

// Send message
ServiceBusMessage message = new ServiceBusMessage("Order #12345");
message.SessionId = "session-001";  // FIFO guarantee
message.MessageId = Guid.NewGuid().ToString();  // Duplicate detection

await sender.SendMessageAsync(message);
```

---

## Когда использовать Azure Queue Storage

### Основные сценарии

✅ **Используйте Queue Storage, если требуется:**

---

### 1️⃣ Простая очередь сообщений

- **Сценарий**: Базовый producer-consumer паттерн
- **Функция**: Простая HTTP-based очередь
- **Пример**: Генерация thumbnail для изображений

📌 Идеально подходит для фоновых задач без сложной логики.

---

### 2️⃣ Большой объём хранения (> 80 GB)

- **Сценарий**: Хранение миллионов сообщений
- **Функция**: До 500 TB на storage account
- **Пример**: Агрегация логов из распределённых систем

---

### 3️⃣ Серверное логирование операций

- **Сценарий**: Аудит всех операций очереди
- **Функция**: Storage Analytics logging
- **Пример**: Соответствие требованиям compliance

---

### 4️⃣ Отслеживание прогресса обработки

- **Сценарий**: Контроль обработки сообщений
- **Функция**: Visibility timeout + dequeue count
- **Пример**: Долгие batch-задачи

📌 Если сообщение не обработано — оно снова станет видимым.

---

### 5️⃣ Экономичное решение

- **Сценарий**: Ограниченный бюджет
- **Функция**: Ниже стоимость по сравнению с Service Bus
- **Пример**: Стартап или dev/test среда

---

### 6️⃣ Простой HTTP/HTTPS доступ

- **Сценарий**: Нет необходимости в AMQP
- **Функция**: REST API через HTTP/HTTPS
- **Пример**: Кроссплатформенные приложения

---

### 7️⃣ Отсутствие требования FIFO

- **Сценарий**: Порядок сообщений не критичен
- **Функция**: At-least-once delivery
- **Пример**: Независимая обработка задач

⚠️ FIFO не гарантируется.

---

### 8️⃣ Просмотр сообщений без блокировки

- **Сценарий**: Мониторинг очереди
- **Функция**: Peek без изменения visibility timeout
- **Пример**: Инспекция сообщений

---

## Матрица принятия решения

| Вопрос | Ответ | Рекомендация |
|--------|--------|--------------|
| Нужен FIFO? | Нет | ✅ Queue Storage |
| Нужен FIFO? | Да | ❌ Service Bus |
| Нужен pub/sub? | Да | ❌ Service Bus Topics |
| Нужен pub/sub? | Нет | ✅ Queue Storage |
| Размер сообщения > 64 KB? | Да | ❌ Service Bus |
| Размер сообщения ≤ 64 KB? | Да | ✅ Queue Storage |
| Нужны транзакции? | Да | ❌ Service Bus |
| Нужны транзакции? | Нет | ✅ Queue Storage |
| Нужна сложная маршрутизация? | Да | ❌ Service Bus |
| Нужна сложная маршрутизация? | Нет | ✅ Queue Storage |
| Ограничен бюджет? | Да | ✅ Queue Storage |
| Нужны enterprise-функции? | Да | ❌ Service Bus |

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:

- Простая очередь → Queue Storage
- Низкая стоимость → Queue Storage
- Нет требований к порядку → Queue Storage
- Требуется FIFO / транзакции / DLQ / routing → Service Bus

Главное различие:

Queue Storage → просто, дешево, масштабируемо  
Service Bus → enterprise-функции, порядок, транзакции, routing

### Code Example: Queue Storage

```csharp
using Azure.Storage.Queues;
using Azure.Storage.Queues.Models;

// Create client
QueueClient queueClient = new QueueClient(connectionString, queueName);
await queueClient.CreateIfNotExistsAsync();

// Send message
await queueClient.SendMessageAsync("Process image IMG_001.jpg");

// Receive message
QueueMessage[] messages = await queueClient.ReceiveMessagesAsync(maxMessages: 1);
QueueMessage message = messages[0];

// Process and delete
Console.WriteLine($"Processing: {message.Body}");
await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
```

---
## Детальное сравнение возможностей

---

# Паттерны обмена сообщениями

| Возможность | Service Bus | Queue Storage |
|-------------|-------------|---------------|
| **Point-to-Point (Queue)** | ✅ Да | ✅ Да |
| **Publish-Subscribe (Topic)** | ✅ Да | ❌ Нет |
| **Competing Consumers** | ✅ Да | ✅ Да |
| **Load Leveling** | ✅ Да | ✅ Да |
| **FIFO порядок** | ✅ Да (с sessions) | ❌ Не гарантируется |
| **At-Least-Once Delivery** | ✅ Да | ✅ Да |
| **At-Most-Once Delivery** | ✅ Да (ReceiveAndDelete) | ❌ Нет |

📌 Если нужен pub/sub или строгий порядок — выбираем Service Bus.

---

# Обработка сообщений

| Возможность | Service Bus | Queue Storage |
|-------------|-------------|---------------|
| **Макс. размер сообщения** | 256 KB (Standard)<br>100 MB (Premium) | 64 KB |
| **Макс. размер очереди** | Практически не ограничен | 500 TB на storage account |
| **TTL (Time-to-Live)** | Не ограничен | 7 дней (по умолчанию)<br>Можно сделать неограниченным |
| **Visibility Timeout** | Lock duration (настраиваемый) | 30 сек (по умолчанию)<br>До 7 дней |
| **Batching** | ✅ Да | ✅ Да |
| **Отложенная доставка (Scheduled)** | ✅ Да | ❌ Нет |
| **Message Deferral** | ✅ Да | ❌ Нет |

📌 Scheduled delivery и deferral — только в Service Bus.

---

# Расширенные возможности

| Возможность | Service Bus | Queue Storage |
|-------------|-------------|---------------|
| **Duplicate Detection** | ✅ Да | ❌ Нет |
| **Транзакции** | ✅ Да | ❌ Нет |
| **Sessions (FIFO)** | ✅ Да | ❌ Нет |
| **Auto-Forwarding** | ✅ Да | ❌ Нет |
| **Dead-Letter Queue (DLQ)** | ✅ Да | ❌ Нет (только вручную) |
| **Фильтрация сообщений** | ✅ Да (SQL filters) | ❌ Нет |
| **Auto-Delete on Idle** | ✅ Да | ❌ Нет |

📌 Enterprise-функции → Service Bus.

---

# Безопасность и доступ

| Возможность | Service Bus | Queue Storage |
|-------------|-------------|---------------|
| **Azure AD** | ✅ Да | ✅ Да |
| **Managed Identity** | ✅ Да | ✅ Да |
| **RBAC** | ✅ Да (детализированный) | ✅ Да |
| **SAS Tokens** | ✅ Да | ✅ Да |
| **VNet Integration** | ✅ Да | ✅ Да |
| **Private Endpoints** | ✅ Да (Premium) | ✅ Да |

---

# Протоколы и API

| Возможность | Service Bus | Queue Storage |
|-------------|-------------|---------------|
| **AMQP 1.0** | ✅ Да | ❌ Нет |
| **HTTP/HTTPS** | ✅ Да | ✅ Да |
| **SBMP** | ✅ Да | ❌ Нет |
| **Long Polling** | ✅ Да | ❌ Нет (short polling) |
| **Peek Lock** | ✅ Да | ❌ Нет (visibility timeout) |
| **Receive and Delete** | ✅ Да | ✅ Да |

📌 Если нужен AMQP или long polling → Service Bus.

---

# Мониторинг и управление

| Возможность | Service Bus | Queue Storage |
|-------------|-------------|---------------|
| **Azure Monitor** | ✅ Да | ✅ Да |
| **Диагностика** | ✅ Да | ✅ Да |
| **Метрики** | ✅ Расширенные | ✅ Базовые |
| **Alerts** | ✅ Да | ✅ Да |
| **Точный счётчик сообщений** | ✅ Точный | ⚠️ Приблизительный |
| **Geo-Replication** | ✅ DR | ✅ Репликация Storage |

---

# Производительность и масштабирование

| Метрика | Service Bus (Standard) | Service Bus (Premium) | Queue Storage |
|----------|----------------------|---------------------|---------------|
| **Throughput** | Переменный | Высокий | Высокий |
| **Latency** | ~10 ms | <10 ms | ~10 ms |
| **Макс. throughput** | ~2000 msg/sec | 80,000+ msg/sec | ~20,000 msg/sec |
| **Partitioning** | ✅ Да | ✅ Да | ❌ Нет |
| **Изоляция ресурсов** | ❌ Shared | ✅ Dedicated | ❌ Shared |

📌 Высокая нагрузка и изоляция → Premium Service Bus.

---

# Сравнение стоимости

| Тариф | Service Bus Basic | Service Bus Standard | Service Bus Premium | Queue Storage |
|--------|------------------|---------------------|-------------------|---------------|
| **Базовая стоимость** | ~$0.05 / млн операций | ~$0.05 / млн операций | ~$667 / месяц за MU | ~$0.05 / GB / месяц |
| **Размер сообщения** | 256 KB | 256 KB | 100 MB | 64 KB |
| **Topics** | ❌ Нет | ✅ Да | ✅ Да | ❌ Нет |
| **Лучше всего подходит для** | Dev/Test | Production | Mission-critical | Экономичные решения |

---

# Итоговый вывод (AZ-204)

### Используйте Queue Storage если:
- Нужна простая очередь
- Нет требований к порядку
- Ограничен бюджет
- Нет необходимости в транзакциях и routing

### Используйте Service Bus если:
- Нужен FIFO
- Нужны транзакции
- Требуется pub/sub
- Нужна DLQ
- Требуется фильтрация сообщений
- Нужны enterprise-функции

---

## Экзаменационный лайфхак

Если в вопросе встречаются слова:
- **Sessions**
- **Duplicate detection**
- **Transactions**
- **Dead-letter**
- **Filtering**
- **Pub/Sub**

→ Почти всегда правильный ответ — **Service Bus**.

Если задача звучит просто и дешево → **Queue Storage**.
---

## Decision Flow Chart

```
START: Need message queue?
│
├─→ Need pub/sub (1:N messaging)?
│   ├─→ YES → Use Service Bus Topics ✅
│   └─→ NO → Continue
│
├─→ Need FIFO ordering guarantee?
│   ├─→ YES → Use Service Bus with Sessions ✅
│   └─→ NO → Continue
│
├─→ Need transactions or duplicate detection?
│   ├─→ YES → Use Service Bus ✅
│   └─→ NO → Continue
│
├─→ Message size > 64 KB?
│   ├─→ YES → Use Service Bus ✅
│   └─→ NO → Continue
│
├─→ Need advanced routing/filtering?
│   ├─→ YES → Use Service Bus Topics ✅
│   └─→ NO → Continue
│
├─→ Need > 80 GB storage?
│   ├─→ YES → Use Queue Storage ✅
│   └─→ NO → Continue
│
├─→ Budget constrained or simple use case?
│   ├─→ YES → Use Queue Storage ✅
│   └─→ NO → Use Service Bus ✅
│
END
```

---
## Реальные сценарии

### Сценарий 1: Обработка заказов в E-Commerce

**Требования:**
- Обрабатывать заказы строго по порядку (FIFO)
- Выполнять платёжные операции атомарно (транзакционно)
- Предотвращать повторную обработку одного и того же заказа
- Маршрутизировать заказы в конкретные центры выполнения (fulfillment)

**Решение: Service Bus Queue + Sessions** ✅

**Почему:**
- **Sessions** обеспечивают гарантию FIFO (при правильной настройке `SessionId`)
- **Transactions** позволяют атомарно отправлять/получать несколько сообщений
- **Duplicate Detection** защищает от повторной обработки (по `MessageId`)
- **Свойства сообщения** позволяют реализовать маршрутизацию (например, по региону/складу)

---

### Дополнение (практика)

- Для маршрутизации по центрам выполнения часто используют:
   - **Topic + Subscriptions** с SQL-фильтрами (если нужно 1:N или много правил)
   - **Queue** (если строго один потребитель/группа competing consumers и логика routing выполняется в приложении)

Но если по условиям нужен строгий порядок + транзакции + антидублирование — выбор Service Bus остаётся самым правильным.

---

## Экзаменационный акцент (AZ-204)

Увидели требования:
- FIFO → **Sessions**
- Атомарность → **Transactions**
- Антидублирование → **Duplicate detection**
- Маршрутизация → **Properties / Filters**

→ Ответ почти всегда **Azure Service Bus** (а не Queue Storage).
```csharp
// Send order with session
var message = new ServiceBusMessage(JsonSerializer.Serialize(order));
message.SessionId = $"customer-{order.CustomerId}";  // FIFO per customer
message.MessageId = order.OrderId.ToString();  // Duplicate detection

await sender.SendMessageAsync(message);
```

### Сценарий 2: Генерация thumbnail для изображений

**Требования:**
- Обработка миллионов изображений
- Простая очередь задач (порядок не важен)
- Минимальная стоимость
- Возможность отслеживать прогресс обработки

**Решение: Queue Storage** ✅

---

### Почему:

- **Простая модель очереди** полностью покрывает задачу (producer → worker)
- **Экономичность** — дешевле, чем Service Bus при высоком объёме сообщений
- **Большая ёмкость хранения** (до 500 TB на storage account)
- **Отслеживание прогресса** через:
   - `dequeueCount`
   - `visibilityTimeout`
   - повторное появление сообщения при ошибке обработки

---

### Практический подход

Типичная архитектура:

1. Веб-приложение загружает изображение в Blob Storage
2. В очередь добавляется сообщение с ссылкой на blob
3. Worker (например, Azure Function) обрабатывает сообщение
4. Создаёт thumbnail и сохраняет обратно в Blob Storage
5. Сообщение удаляется из очереди

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:

- Простая фоновая обработка
- Нет требований к FIFO
- Нет транзакций
- Важна низкая стоимость
- Высокий объём сообщений

→ Правильный ответ: **Azure Queue Storage**, а не Service Bus.

Главный критерий — простота и экономичность.

```csharp
// Send image processing task
await queueClient.SendMessageAsync($"{{\"imageId\": \"{imageId}\", \"url\": \"{imageUrl}\"}}");

// Process with retry tracking
QueueMessage[] messages = await queueClient.ReceiveMessagesAsync(1);
if (messages[0].DequeueCount > 5)
{
    // Move to error queue after 5 retries
    await errorQueueClient.SendMessageAsync(messages[0].Body.ToString());
}
```

### Сценарий 3: Маршрутизация IoT-телеметрии

**Требования:**
- Принимать телеметрию от устройств
- Доставлять данные нескольким подписчикам (аналитика, алёрты, storage)
- Фильтровать по типу устройства или геолокации
- Высокая пропускная способность

**Решение: Service Bus Topic + Subscriptions** ✅

---

### Почему:

- **Publish/Subscribe**: один publisher → несколько независимых consumers
- **Независимая обработка**: каждый подписчик читает свою subscription и не влияет на других
- **Продвинутая фильтрация**: SQL filters / correlation filters по свойствам сообщения (например, `deviceType`, `region`, `location`)
- **Высокая производительность**: Premium tier даёт выделенные ресурсы и высокий throughput

---

### Практические детали

- Publisher отправляет сообщения в **Topic** с properties, например:
   - `deviceType = "sensor"` / `"camera"`
   - `region = "EU"` / `"US"`
- Подписки:
   - `analytics-sub` получает все сообщения
   - `alerts-sub` получает только критические (например, `severity >= 3`)
   - `storage-sub` получает данные для архивирования

---

## Экзаменационный акцент (AZ-204)

Если в вопросе есть:
- **1:N доставка**
- **несколько независимых потребителей**
- **фильтрация по properties**
- **маршрутизация**

→ Ответ почти всегда: **Service Bus Topic + Subscriptions**.

Queue Storage не подходит, потому что не поддерживает pub/sub и фильтры.

```csharp
// Publish telemetry to topic
var message = new ServiceBusMessage(JsonSerializer.Serialize(telemetry));
message.ApplicationProperties["DeviceType"] = "sensor";
message.ApplicationProperties["Location"] = "warehouse-01";

await topicSender.SendMessageAsync(message);

// Subscription 1: Analytics (all devices)
// Subscription 2: Alerts (only temperature sensors)
// Filter: DeviceType = 'temperature-sensor'
// Subscription 3: Storage (only warehouse-01)
// Filter: Location = 'warehouse-01'
```

### Сценарий 4: Агрегация логов

**Требования:**
- Сбор логов с 1000+ серверов
- Хранение для пакетной (batch) обработки
- Очень большой объём сообщений
- Экономичное хранение

**Решение: Queue Storage** ✅

---

### Почему:

- **Огромная ёмкость хранения** (до 500 TB на storage account)
- **Низкая стоимость** при большом объёме сообщений
- **Простой ingestion через HTTP/HTTPS** (REST API)
- **Нет необходимости в enterprise-функциях** (FIFO, транзакции, routing, pub/sub)

---

### Практический вариант архитектуры

- Агенты на серверах отправляют сообщения в Queue Storage (например, ссылки на blob с лог-файлом или батч логов)
- Worker/Job периодически читает очередь и:
   - складывает данные в Blob/Data Lake
   - запускает обработку (Spark, Databricks, Synapse, batch job)

📌 Часто в сообщениях хранят не сами логи, а **ссылки на blobs**, чтобы не упираться в лимит 64 KB.

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:

- огромный объём сообщений
- нужна простая очередь
- важна стоимость
- нет требований к FIFO/транзакциям/pub-sub

→ правильный ответ: **Queue Storage**.

Service Bus здесь обычно избыточен и дороже.

```csharp
// Collect logs from servers
foreach (var logEntry in logEntries)
{
    await queueClient.SendMessageAsync(JsonSerializer.Serialize(logEntry));
}

// Batch process every hour
var messages = await queueClient.ReceiveMessagesAsync(32);  // Get up to 32 messages
foreach (var message in messages)
{
    // Process and delete
    await ProcessLogAsync(message.Body.ToString());
    await queueClient.DeleteMessageAsync(message.MessageId, message.PopReceipt);
}
```

### Сценарий 5: Взаимодействие микросервисов

**Требования:**
- Развязать микросервисы (decouple)
- Обеспечить надёжную доставку сообщений
- Поддержать паттерн request-reply
- Нужны enterprise-возможности

**Решение: Service Bus Queues** ✅

---

### Почему:

- **Надёжная доставка** (at-least-once + механизмы lock/ack)
- **Request-reply** поддерживается через свойства `ReplyTo` / `CorrelationId`
- **Dead-letter queue (DLQ)** для сообщений, которые не удалось обработать
- **Безопасность**: интеграция с Azure AD и RBAC (вместо общих ключей)

---

### Практическая деталь (request-reply)

Типичный подход:

- Сервис A отправляет сообщение в очередь сервиса B
- Указывает:
   - `ReplyTo` = имя очереди/топика для ответа
   - `CorrelationId` = ID запроса
- Сервис B обрабатывает и отправляет ответ в очередь ответа
- Сервис A получает ответ и сопоставляет по `CorrelationId`

📌 Это асинхронный request-reply: сервисы остаются decoupled, но появляется корреляция запрос-ответ.

---

## Экзаменационный акцент (AZ-204)

Если в вопросе присутствуют:
- "enterprise-grade"
- "DLQ"
- "reliable messaging"
- "request-reply"
- "security через Azure AD"

→ почти всегда выбираем **Azure Service Bus** (Queues или Topics в зависимости от 1:1 vs 1:N).

```csharp
// Service A: Send request
var message = new ServiceBusMessage(JsonSerializer.Serialize(request));
message.ReplyTo = "service-a-replies";
message.CorrelationId = Guid.NewGuid().ToString();
await sender.SendMessageAsync(message);

// Service B: Process and reply
var request = await receiver.ReceiveMessageAsync();
var response = ProcessRequest(request.Body.ToString());

var reply = new ServiceBusMessage(JsonSerializer.Serialize(response));
reply.CorrelationId = request.CorrelationId;
await replyToSender.SendMessageAsync(reply);
```

---
## Сценарии миграции

---

### Миграция с Queue Storage на Service Bus

**Когда мигрировать:**
- Появилась необходимость в FIFO
- Требуется publish-subscribe
- Нужны транзакции
- Размер сообщений превышает 64 KB

### Шаги миграции:

1. Создать Service Bus namespace и очередь
2. Развернуть параллельных consumers (Queue Storage + Service Bus)
3. Переключить producers на Service Bus
4. Дождаться опустошения старой очереди
5. Декомиссия Queue Storage

📌 Важно: избегайте "big bang" переключения — используйте поэтапную миграцию.

---

### Миграция с Service Bus на Queue Storage

**Когда мигрировать:**
- Требуется снижение стоимости
- Расширенные возможности не используются
- Размер сообщений < 64 KB
- Нет требований к порядку обработки

### Шаги миграции:

1. Создать Storage Account и очередь
2. Развернуть параллельных consumers
3. Переключить producers на Queue Storage
4. Отслеживать глубину очереди Service Bus
5. Декомиссия namespace Service Bus

📌 Перед миграцией убедитесь, что не используются sessions, транзакции, DLQ или фильтры.

---

# Лучшие практики

## Выбор правильного решения

### 1️⃣ Начинайте с требований

- Определите функциональные требования
- Определите нефункциональные (стоимость, производительность, масштаб)
- Определите must-have функции

---

### 2️⃣ Используйте decision matrix

- Сопоставьте требования и возможности
- Исключите неподходящий вариант
- Выберите минимально достаточное решение

---

### 3️⃣ Учитывайте будущее масштабирование

- Планируйте рост нагрузки
- Оцените необходимость новых функций
- Оцените сложность будущей миграции

---

### 4️⃣ Делайте Proof of Concept

- Протестируйте оба варианта
- Измерьте latency и throughput
- Рассчитайте стоимость

---

# Частые анти-паттерны

❌ Использовать Service Bus для простой очереди  
→ Сложнее и дороже  
✅ Для простого producer-consumer используйте Queue Storage

❌ Использовать Queue Storage для строгого порядка  
→ FIFO не гарантируется  
✅ Для FIFO используйте Service Bus sessions

❌ Игнорировать лимиты размера сообщений
- Queue Storage: 64 KB
- Service Bus Standard: 256 KB  
  ✅ Разбивайте сообщения или используйте Premium

❌ Не учитывать стоимость при масштабе  
→ Service Bus может быть дорогим при большом объёме  
✅ Рассчитывайте TCO заранее

---

# Советы для экзамена AZ-204

## Ключевые различия

1. **Service Bus** → Enterprise messaging
2. **Queue Storage** → Простая и дешёвая очередь
3. **FIFO** → Только Service Bus sessions
4. **Pub/Sub** → Только Service Bus Topics
5. **Размер сообщения** → 64 KB vs 256 KB / 100 MB
6. **Транзакции** → Только Service Bus
7. **Duplicate detection** → Только Service Bus

---

## Типовые экзаменационные сценарии

### Сценарий 1
Требуется обработка строго по порядку  
→ ✅ Service Bus + Sessions  
→ ❌ Не Queue Storage

### Сценарий 2
Нужно отправить сообщение нескольким подписчикам  
→ ✅ Service Bus Topics  
→ ❌ Не Queue Storage

### Сценарий 3
Миллионы небольших сообщений, важна стоимость  
→ ✅ Queue Storage  
→ ❌ Service Bus

### Сценарий 4
Нужны транзакции между несколькими сообщениями  
→ ✅ Service Bus  
→ ❌ Queue Storage

### Сценарий 5
Сообщения больше 64 KB  
→ ✅ Service Bus  
→ ❌ Queue Storage

---

# Финальная таблица для запоминания

| Возможность | Service Bus | Queue Storage |
|-------------|-------------|---------------|
| **FIFO** | ✅ С sessions | ❌ Нет гарантии |
| **Pub/Sub** | ✅ Topics | ❌ Нет |
| **Транзакции** | ✅ Да | ❌ Нет |
| **Duplicate Detection** | ✅ Да | ❌ Нет |
| **Макс. размер сообщения** | 256 KB / 100 MB | 64 KB |
| **Стоимость** | Выше | Ниже |
| **Лучше всего подходит для** | Enterprise | Простых очередей |

---

## Экзаменационный лайфхак

Если в вопросе упоминаются:

- sessions
- DLQ
- transactions
- filtering
- pub/sub
- duplicate detection

→ выбирайте **Service Bus**.

Если акцент на простоте и стоимости → **Queue Storage**.
### Quick Reference

```csharp
// Service Bus
await using var client = new ServiceBusClient(connectionString);
var sender = client.CreateSender(queueName);
await sender.SendMessageAsync(new ServiceBusMessage("data"));

// Queue Storage
var queueClient = new QueueClient(connectionString, queueName);
await queueClient.SendMessageAsync("data");
```

---

## Итоговое сравнение

### Выбирайте **Service Bus**, если требуется:

- ✅ Гарантия FIFO (через Sessions)
- ✅ Publish/Subscribe (Topics + Subscriptions)
- ✅ Транзакции
- ✅ Duplicate detection
- ✅ Продвинутая маршрутизация и фильтрация
- ✅ Enterprise-функции (DLQ, авто-forwarding, scheduled delivery)
- ✅ Сообщения больше 64 KB

📌 Ключевая идея: Service Bus = расширенные возможности + контроль + надёжность.

---

### Выбирайте **Queue Storage**, если требуется:

- ✅ Простая очередь (producer-consumer)
- ✅ Экономичное решение
- ✅ Большая ёмкость хранения (> 80 GB)
- ✅ Серверное логирование операций
- ✅ Сообщения ≤ 64 KB
- ✅ Нет требований к порядку обработки

📌 Ключевая идея: Queue Storage = просто, дёшево, масштабируемо.

---

## Финальный экзаменационный ориентир (AZ-204)

Если в вопросе звучит:

- «FIFO», «sessions», «transactions», «pub/sub», «DLQ» → **Service Bus**
- «Simple queue», «cost-effective», «millions of small messages» → **Queue Storage**

Главный принцип выбора:

**Минимально достаточное решение**, которое покрывает требования задачи.

**Remember**: Start simple (Queue Storage), upgrade to Service Bus when you need advanced features!