# Обзор Azure Event Hubs

## Что такое Azure Event Hubs?

**Azure Event Hubs** — это полностью управляемая платформа для потоковой передачи данных в реальном времени и сервис приёма событий, способный принимать и обрабатывать **миллионы событий в секунду** с минимальной задержкой.

Event Hubs предназначен для сценариев high-throughput ingestion и последующей потоковой или пакетной обработки.

---

## Ключевые характеристики

- **Big Data Streaming**  
  Приём миллионов событий в секунду

- **Низкая задержка**  
  Обработка в режиме near real-time (sub-second latency)

- **Совместимость с Apache Kafka**  
  Возможность запускать Kafka-нагрузку без изменения кода

- **Долговременное хранение**  
  События сохраняются для последующей обработки

- **Partitioned Consumer Model**  
  Масштабирование через параллельных потребителей

- **Поддержка нескольких протоколов**  
  AMQP, Kafka, HTTPS

- **Полностью управляемый сервис**  
  Нет необходимости администрировать инфраструктуру

---

# Основные сценарии использования

| Сценарий | Описание | Пример |
|------------|-----------|---------|
| **Telemetry & IoT** | Приём телеметрии от IoT-устройств | Умные датчики, автомобили |
| **Application Logging** | Централизованный сбор логов | Агрегация логов микросервисов |
| **Clickstream Analytics** | Отслеживание поведения пользователей | Аналитика e-commerce |
| **Live Dashboarding** | Метрики в реальном времени | Операционные панели |
| **Anomaly Detection** | Обнаружение аномалий | Fraud detection |
| **Archiving** | Долговременное хранение потоков | Финансовые логи |
| **Transaction Processing** | Поточная обработка транзакций | Платёжные системы |

---

## Архитектурный акцент

Event Hubs — это:

- Высокопроизводительный ingestion layer
- Буфер между producers и stream processors
- Основа для real-time analytics
- Часто используется вместе с:
   - Azure Stream Analytics
   - Azure Functions
   - Azure Databricks
   - Apache Spark

---

## Важно для AZ-204

Нужно понимать:

- Event Hubs = streaming + big data ingestion
- Поддерживает Kafka-протокол
- Масштабируется через partitions
- Подходит для high-throughput telemetry
- Отличается от Event Grid (reactive events)
- Отличается от Service Bus (transactional messaging)

Если в вопросе говорится о миллионах событий в секунду или потоковой аналитике — почти всегда правильный ответ — Event Hubs.

---

## Event Hubs Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        EVENT PRODUCERS                           │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ IoT      │  │ Web/     │  │ Mobile   │  │ Kafka    │       │
│  │ Devices  │  │ Mobile   │  │ Apps     │  │ Apps     │       │
│  │          │  │ Apps     │  │          │  │          │       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
└───────┼─────────────┼─────────────┼─────────────┼──────────────┘
        │             │             │             │
        │ AMQP/HTTPS  │ HTTPS       │ HTTPS       │ Kafka Protocol
        └─────────────┴─────────────┴─────────────┘
                      │
                      ▼
        ┌─────────────────────────────────┐
        │   EVENT HUBS NAMESPACE          │
        │  ┌───────────────────────────┐  │
        │  │   Event Hub (Topic)       │  │
        │  │  ┌────┬────┬────┬────┐   │  │
        │  │  │ P0 │ P1 │ P2 │ P3 │   │  │  ← Partitions
        │  │  └────┴────┴────┴────┘   │  │
        │  └───────────────────────────┘  │
        │                                 │
        │  ┌───────────────────────────┐  │
        │  │   Consumer Groups         │  │
        │  │  • $Default               │  │
        │  │  • StreamAnalytics        │  │
        │  │  • ApplicationProcessing  │  │
        │  └───────────────────────────┘  │
        └─────────────┬───────────────────┘
                      │
        ┌─────────────┴──────────────────────┐
        │                                    │
        ▼                                    ▼
┌───────────────────┐            ┌───────────────────┐
│  CONSUMER GROUP 1 │            │  CONSUMER GROUP 2 │
│                   │            │                   │
│ ┌───────────────┐ │            │ ┌───────────────┐ │
│ │ Consumer 1    │ │            │ │ Stream        │ │
│ │ (reads P0-P1) │ │            │ │ Analytics     │ │
│ ├───────────────┤ │            │ │ (reads all)   │ │
│ │ Consumer 2    │ │            │ └───────────────┘ │
│ │ (reads P2-P3) │ │            │                   │
│ └───────────────┘ │            └───────────────────┘
└───────────────────┘
        │
        ▼
┌───────────────────┐
│ Event Processor   │
│ (Checkpointing)   │
│                   │
│ Azure Blob        │
│ Storage           │
└───────────────────┘
```

---

# Основные понятия Azure Event Hubs

## 1️⃣ Event Hubs Namespace

**Namespace** — это логический контейнер управления, внутри которого создаются один или несколько Event Hubs (или Kafka topics).

Это уровень конфигурации инфраструктуры и безопасности.

---

## Характеристики Namespace

- **Региональность**  
  Ресурс создаётся в конкретном регионе Azure

- **Емкость (Capacity)**  
  Определяется через:
   - Throughput Units (Standard)
   - Processing Units (Premium/Dedicated)

- **Сетевые настройки**
   - VNet integration
   - Firewall rules
   - Private Endpoints

- **Безопасность**
   - Shared Access Policies
   - Access keys
   - Azure AD / RBAC

- **Уникальный endpoint**  
  Формат:

<namespace>.servicebus.windows.net

---

## Архитектурный смысл

Namespace:

- Изолирует среду (dev/test/prod)
- Управляет масштабированием
- Контролирует сетевой доступ
- Централизует безопасность

---

## Важно для AZ-204

Нужно помнить:

- Namespace — это контейнер для Event Hubs
- Он региональный
- Throughput Units определяют пропускную способность
- Endpoint namespace используется producers и consumers
- Сетевые и security-настройки применяются на уровне namespace

Если вопрос касается масштабирования, сетевой изоляции или ключей — это уровень namespace.


**Create Namespace (Azure CLI):**
```bash
az eventhubs namespace create \
  --name myeventhubns \
  --resource-group myResourceGroup \
  --location eastus \
  --sku Standard \
  --capacity 1
```

## Уровни Namespace (Tiers)

| Tier | Пропускная способность | Возможности | Сценарий использования |
|------|------------------------|-------------|------------------------|
| **Basic** | До 20 Throughput Units | Базовый функционал, хранение 1 день | Разработка, тестирование |
| **Standard** | До 40 Throughput Units | Consumer groups, Capture, auto-inflate | Production-нагрузка |
| **Premium** | Processing Units | Выделенные ресурсы, изоляция, увеличенное хранение | Mission-critical системы |
| **Dedicated** | Capacity Units | Single-tenant развертывание | Enterprise, compliance |

---

### Что важно понимать

- **Basic** — ограниченный функционал.
- **Standard** — наиболее распространённый вариант для production.
- **Premium** — выделенные ресурсы и изоляция.
- **Dedicated** — отдельный кластер (enterprise-сценарии).

---

# 2️⃣ Event Hub (Kafka Topic)

**Event Hub** — это append-only поток событий, логически похожий на Kafka topic.

События записываются последовательно и не изменяются после публикации.

---

## Свойства Event Hub

- **Partitions**
   - 1–32 (Standard)
   - До 100 (Premium)
     Используются для масштабирования и параллельной обработки.

- **Retention**
   - От 1 до 90 дней (настраивается)
     События хранятся фиксированное время независимо от чтения.

- **Размер сообщения**
   - До 1 MB на событие

- **Throughput**
   - Ограничивается Throughput Units или Processing Units

- **Ordering**
   - Гарантируется **внутри одной partition**
   - Между partitions порядок не гарантируется

---

## Архитектурный акцент

- Event Hub — это distributed log
- Масштабирование достигается через partitions
- Producers пишут в partitions
- Consumers читают независимо
- Retention не зависит от того, прочитано ли сообщение

---

## Важно для AZ-204

Нужно помнить:

- Порядок гарантируется только внутри partition
- Retention — временной, а не «пока не прочитано»
- Максимальный размер события — 1 MB
- Throughput Units ограничивают скорость записи
- Premium поддерживает больше partitions

Если в вопросе речь о потоковой обработке с миллионами событий — это Event Hubs.

**Create Event Hub (Azure CLI):**
```bash
az eventhubs eventhub create \
  --name myeventhub \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --partition-count 4 \
  --message-retention 7
```

### 3. Partitions

**Partitions** are ordered sequences of events within an Event Hub, similar to lanes on a freeway.

**Why Partitions?**
- **Scalability**: Distribute load across multiple consumers
- **Parallelism**: Process events in parallel
- **Ordering**: Events in same partition maintain order
- **Performance**: Each partition has independent throughput

**Partition Architecture:**

```
Event Hub: "telemetry-data"
├── Partition 0: [E1] [E2] [E3] [E4] [E5] → oldest...newest
├── Partition 1: [E6] [E7] [E8] [E9] [E10]
├── Partition 2: [E11] [E12] [E13] [E14] [E15]
└── Partition 3: [E16] [E17] [E18] [E19] [E20]
```

**Partition Key:**
- **Purpose**: Determines which partition receives an event
- **Hashing**: Event Hubs hashes the key to select partition
- **Use**: Group related events (e.g., all events from device-001)

**Example - Partition Key:**
```csharp
// Events with same partition key go to same partition
var eventData = new EventData(Encoding.UTF8.GetBytes(jsonData))
{
    PartitionKey = "device-001"  // All events from this device go to same partition
};

await producer.SendAsync(eventData);
```

# Выбор количества partitions

Количество partitions напрямую влияет на масштабируемость и стоимость.

| Количество partitions | Пропускная способность | Стоимость | Сценарий |
|------------------------|------------------------|-----------|----------|
| **1–2** | Низкая (1–2 MB/s) | Ниже | Разработка, небольшой объём |
| **4–8** | Средняя (4–8 MB/s) | Средняя | Стандартная production-нагрузка |
| **16–32** | Высокая (16–32 MB/s) | Выше | High-throughput приложения |

---

## Важные замечания

- ⚠️ **Количество partitions нельзя изменить** после создания Event Hub
- ✅ Планируйте с учётом будущего роста
- ✅ Больше partitions = больше параллелизма
- ⚠️ Но выше стоимость

---

## Архитектурный акцент

- Partitions определяют уровень параллельной обработки
- Каждый consumer читает partition эксклюзивно
- Ordering гарантируется только внутри partition
- Scaling-out = увеличение числа partitions

---

# 4️⃣ Consumer Groups

**Consumer group** — это независимое представление потока, позволяющее нескольким приложениям читать одни и те же события независимо друг от друга.

---

## Характеристики Consumer Groups

- **Независимые offset’ы**  
  Каждая группа отслеживает свою позицию чтения

- **Параллельная обработка**  
  Разные группы могут обрабатывать один и тот же поток одновременно

- **Группа по умолчанию**  
  `$Default` создаётся автоматически

- **Максимум групп**  
  До 20 consumer groups в Standard tier

---

## Архитектурный пример

Один Event Hub может обслуживать:

- Real-time analytics
- Архивирование
- Fraud detection
- Мониторинг

Каждый сценарий использует отдельную consumer group.

---

## Важно для AZ-204

Нужно помнить:

- Consumer group = независимое чтение одного потока
- Offsets хранятся отдельно для каждой группы
- `$Default` создаётся автоматически
- Partitions нельзя изменить после создания
- Ordering гарантируется только внутри partition

Если в вопросе требуется несколько независимых обработчиков одного потока — используйте consumer groups.
**Consumer Group Use Cases:**

```
Event Hub: "orders"
├── Consumer Group: "$Default"
│   └── Real-time order processing application
│
├── Consumer Group: "analytics"
│   └── Azure Stream Analytics (near real-time analytics)
│
├── Consumer Group: "archiving"
│   └── Event Hubs Capture (long-term storage)
│
└── Consumer Group: "monitoring"
    └── Application monitoring dashboard
```

**Create Consumer Group (Azure CLI):**
```bash
az eventhubs eventhub consumer-group create \
  --name analytics \
  --eventhub-name myeventhub \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup
```

**Example Scenario:**
```
Single Event Hub: "telemetry-data"

Consumer Group 1: Real-time alerting
  └── Processes events immediately for anomaly detection

Consumer Group 2: Batch analytics
  └── Reads events for hourly aggregation

Consumer Group 3: Machine learning
  └── Reads events for model training

All groups read the SAME events independently!
```

### 5. Event Producers

**Event producers** are applications that send data to Event Hubs.

**Producer Protocols:**
- **AMQP 1.0**: Recommended for high-throughput scenarios
- **Kafka**: Use existing Kafka producer clients
- **HTTPS**: Simple REST API for occasional publishers

**Producer Patterns:**

**Pattern 1: Round-Robin (No Partition Key)**
```csharp
// Events distributed evenly across partitions
var eventData = new EventData(Encoding.UTF8.GetBytes(jsonData));
await producer.SendAsync(eventData);
```

**Pattern 2: Partition Key (Related Events)**
```csharp
// Events with same key go to same partition (ordered)
var eventData = new EventData(Encoding.UTF8.GetBytes(jsonData))
{
    PartitionKey = customerId  // Group by customer
};
await producer.SendAsync(eventData);
```

**Pattern 3: Specific Partition**
```csharp
// Send directly to partition 2
var sendOptions = new SendEventOptions
{
    PartitionId = "2"
};
await producer.SendAsync(eventData, sendOptions);
```

**Batching for Performance:**
```csharp
// Create batch (more efficient)
EventDataBatch batch = await producer.CreateBatchAsync();

foreach (var data in dataList)
{
    var eventData = new EventData(Encoding.UTF8.GetBytes(data));
    
    if (!batch.TryAdd(eventData))
    {
        // Batch full, send it
        await producer.SendAsync(batch);
        
        // Create new batch
        batch = await producer.CreateBatchAsync();
        batch.TryAdd(eventData);
    }
}

// Send remaining events
if (batch.Count > 0)
{
    await producer.SendAsync(batch);
}
```

# 6️⃣ Event Consumers

**Event consumers** — это приложения или сервисы, которые читают и обрабатывают данные из Event Hubs.

Они получают события из partitions и обрабатывают их в реальном времени или пакетно.

---

## Типы Consumers

| Тип Consumer | Сценарий | Пример |
|--------------|----------|---------|
| **EventHubConsumerClient** | Прототипирование, тестирование | Ручное чтение событий |
| **EventProcessorClient** | Production-приложения | Масштабируемая и отказоустойчивая обработка |
| **Azure Stream Analytics** | Поточная аналитика | SQL-подобные запросы к потокам |
| **Azure Functions** | Serverless-обработка | Автоматический trigger при новых событиях |
| **Apache Spark** | Big Data обработка | Масштабная batch-обработка |

---

## Разница между ConsumerClient и EventProcessorClient

### EventHubConsumerClient
- Простой API
- Не управляет распределением partitions
- Подходит для тестирования

### EventProcessorClient
- Автоматически распределяет partitions
- Управляет checkpointing
- Обеспечивает масштабирование
- Используется в production

---

## Архитектурный акцент

- Consumers читают данные независимо
- Каждая consumer group имеет собственные offsets
- Один partition обрабатывается одним consumer в рамках группы
- Scaling достигается увеличением числа consumer-инстансов

---

## Важно для AZ-204

Нужно помнить:

- EventProcessorClient — production-паттерн
- Stream Analytics — SQL-подобная потоковая аналитика
- Functions — serverless-обработка
- Spark — big data processing
- Consumers работают через consumer groups
- Offsets управляют позицией чтения

Если в вопросе требуется масштабируемая и отказоустойчивая обработка — правильный выбор обычно EventProcessorClient.

**Consumer Pattern - EventHubConsumerClient:**
```csharp
var consumerGroup = EventHubConsumerClient.DefaultConsumerGroupName;
var consumer = new EventHubConsumerClient(consumerGroup, connectionString, eventHubName);

await foreach (PartitionEvent evt in consumer.ReadEventsAsync())
{
    Console.WriteLine($"Event: {evt.Data.EventBody}");
    Console.WriteLine($"Partition: {evt.Partition.PartitionId}");
    Console.WriteLine($"Offset: {evt.Data.Offset}");
}
```

**Consumer Pattern - EventProcessorClient (Recommended):**
```csharp
var storageClient = new BlobContainerClient(storageConnectionString, containerName);
var processor = new EventProcessorClient(storageClient, consumerGroup, connectionString, eventHubName);

processor.ProcessEventAsync += async (args) =>
{
    // Process event
    Console.WriteLine($"Event: {args.Data.EventBody}");
    
    // Update checkpoint
    await args.UpdateCheckpointAsync();
};

processor.ProcessErrorAsync += async (args) =>
{
    Console.WriteLine($"Error: {args.Exception.Message}");
};

await processor.StartProcessingAsync();
```

### 7. Checkpoints and Offsets

**Offset**: Position of an event within a partition

**Checkpoint**: Marking a specific offset as processed

```
Partition 0: [E0] [E1] [E2] [E3] [E4] [E5] [E6] [E7]
              ↑              ↑              ↑
           Offset=0     Offset=3      Offset=6
                           ↑
                      Checkpoint
              (Consumer has processed up to E3)
```

# Checkpointing в Event Hubs

Checkpointing — это механизм сохранения позиции чтения (offset) consumer’а.

Он позволяет возобновить обработку с последнего сохранённого места.

---

## Зачем нужен Checkpointing?

- **Fault Tolerance**  
  После сбоя consumer продолжит чтение с последнего checkpoint.

- **Практическая идемпотентность**  
  Позволяет минимизировать повторную обработку.

- **Отслеживание прогресса**  
  Позволяет мониторить lag (отставание от текущей позиции).

---

## Где хранится Checkpoint?

- **Azure Blob Storage**  
  Наиболее распространённый вариант (используется EventProcessorClient)

- **Другие хранилища**  
  Возможна кастомная реализация

---

## Архитектурный акцент

- Checkpoint сохраняется на уровне consumer group
- Каждый partition имеет собственный offset
- Без checkpointing при перезапуске произойдёт повторное чтение

---

# Throughput Units (TUs) и Processing Units (PUs)

## Throughput Units (Standard Tier)

**Throughput Unit (TU)** — единица пропускной способности в Standard tier.

Она определяет лимиты на входящий и исходящий трафик.

---

## Пропускная способность 1 TU

- **Ingress (входящий поток)**  
  До 1 MB/s или 1 000 событий/сек

- **Egress (исходящий поток)**  
  До 2 MB/s или 4 096 событий/сек

---

## Что важно понимать

- Превышение лимита приводит к throttling
- Можно увеличить количество TU
- Доступна функция auto-inflate (автоматическое масштабирование)

---

## Архитектурный смысл

TUs определяют:

- Максимальную скорость записи
- Максимальную скорость чтения
- Стоимость использования

---

## Важно для AZ-204

Нужно помнить:

- 1 TU = 1 MB/s ingress
- 1 TU = 2 MB/s egress
- Throttling при превышении лимита
- Checkpointing обеспечивает устойчивость
- EventProcessorClient использует Blob Storage для checkpoint

Если в вопросе говорится о high-throughput ingestion — нужно рассчитать количество TUs.

**Example Calculations:**

```
Scenario 1: 5,000 events/second, 500 bytes per event
- Ingress: 5,000 × 0.5 KB = 2.5 MB/s → Need 3 TUs
- Cost: Based on TU hours

Scenario 2: 500 KB events, 100 events/second
- Ingress: 100 × 0.5 MB = 50 MB/s → Need 50 TUs
- Consider Premium tier for better value
```

**Auto-Inflate:**
- Automatically scale TUs based on load
- Set maximum TU limit
- Useful for variable workloads

```bash
# Enable auto-inflate
az eventhubs namespace update \
  --name myeventhubns \
  --resource-group myResourceGroup \
  --enable-auto-inflate true \
  --maximum-throughput-units 20
```

# Processing Units (Premium / Dedicated)

**Processing Unit (PU)** — единица мощности в Premium tier.

В отличие от Throughput Units (Standard), PU предоставляет выделенные ресурсы.

---

## Особенности PU

- Более высокая производительность по сравнению с TU
- Выделенные CPU и память
- Изоляция от других клиентов (no noisy neighbors)
- Предсказуемая производительность
- Подходит для mission-critical workloads

---

## Когда использовать Premium / Dedicated

- Высокая нагрузка
- Требуется стабильная производительность
- Повышенные требования к изоляции
- Увеличенный срок хранения данных
- Enterprise и compliance-сценарии

---

# Retention (Период хранения событий)

**Retention Period** — это время, в течение которого события сохраняются в Event Hubs независимо от того, были ли они прочитаны.

---

## Периоды хранения по tier

| Tier | Минимум | Максимум | По умолчанию |
|------|----------|-----------|--------------|
| **Basic** | 1 день | 1 день | 1 день |
| **Standard** | 1 день | 7 дней | 1 день |
| **Premium** | 1 день | 90 дней | 1 день |
| **Dedicated** | 1 день | 90 дней | 1 день |

---

## Важные моменты

- Retention основан на времени, а не на факте чтения
- После истечения срока события удаляются автоматически
- Увеличение retention увеличивает стоимость
- Premium и Dedicated поддерживают до 90 дней

---

## Архитектурный акцент

- Event Hubs — это временное хранилище потоков
- Не предназначен для долговременного архива
- Для долгосрочного хранения используется Capture (в Blob или Data Lake)

---

## Важно для AZ-204

Нужно помнить:

- Basic — всегда 1 день
- Standard — до 7 дней
- Premium / Dedicated — до 90 дней
- Retention не зависит от чтения consumer’ом
- PU используется в Premium tier

Если в вопросе требуется хранение событий более 7 дней — Standard не подходит.

**Configure Retention:**
```bash
az eventhubs eventhub update \
  --name myeventhub \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --message-retention 7
```

**Use Cases by Retention:**
- **1 day**: Real-time processing only, no replay needed
- **3-7 days**: Buffer for consumer recovery, debugging
- **30-90 days**: Compliance, reprocessing, batch analytics

---

## Apache Kafka Compatibility

Event Hubs supports **Apache Kafka protocol** natively.

**Benefits:**
- Use existing Kafka applications
- No code changes required
- Fully managed (no Kafka cluster management)
- Azure integration (security, monitoring, networking)

**Connection String Format:**
```
Bootstrap servers: <namespace>.servicebus.windows.net:9093
SASL mechanism: PLAIN
Security protocol: SASL_SSL
Username: $ConnectionString
Password: <connection-string>
```

**Kafka Producer Example (Java):**
```java
Properties props = new Properties();
props.put("bootstrap.servers", "myeventhubns.servicebus.windows.net:9093");
props.put("security.protocol", "SASL_SSL");
props.put("sasl.mechanism", "PLAIN");
props.put("sasl.jaas.config", "org.apache.kafka.common.security.plain.PlainLoginModule required username=\"$ConnectionString\" password=\"Endpoint=sb://myeventhubns.servicebus.windows.net/;SharedAccessKeyName=RootManageSharedAccessKey;SharedAccessKey=<key>\";");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);

ProducerRecord<String, String> record = new ProducerRecord<>("myeventhub", "key", "value");
producer.send(record);
```

*# Соответствие Kafka и Event Hubs

## Kafka → Event Hubs Mapping

| Концепция Kafka | Эквивалент в Event Hubs |
|------------------|-------------------------|
| Kafka Cluster | Event Hubs Namespace |
| Kafka Topic | Event Hub |
| Partition | Partition |
| Consumer Group | Consumer Group |
| Offset | Offset |
| Broker | Не применяется (управляемый сервис) |

---

## Что это означает

- Namespace ≈ Kafka cluster
- Event Hub ≈ Kafka topic
- Partitions и offsets работают аналогично
- Нет необходимости управлять брокерами

Event Hubs скрывает инфраструктурную сложность Kafka.

---

# Azure Schema Registry

**Azure Schema Registry** — централизованное хранилище схем для потоковых приложений.

---

## Возможности

- Хранение и версионирование схем
- Поддержка Avro и JSON Schema
- Интеграция с Kafka и Event Hubs SDK
- Управление эволюцией схем

---

## Преимущества

- **Type Safety**  
  Гарантия соответствия контракту данных

- **Совместимость**  
  Контроль изменений схемы

- **Эффективность**  
  Передача ссылок на схему вместо полного описания

- **Governance**  
  Централизованное управление контрактами

---

# Event Hubs vs другие сервисы Azure

| Характеристика | **Event Hubs** | **Event Grid** | **Service Bus** |
|----------------|---------------|---------------|----------------|
| **Паттерн** | Потоковая обработка | Распределение событий | Очереди сообщений |
| **Throughput** | Миллионы/сек | Миллионы/сек | Тысячи/сек |
| **Задержка** | Sub-second | Sub-second | Низкая |
| **Retention** | 1–90 дней | Нет хранения | До 14 дней |
| **Ordering** | Внутри partition | Не гарантируется | FIFO (sessions) |
| **Размер сообщения** | 1 MB | 1 MB | 256 KB (1 MB Premium) |
| **Протоколы** | AMQP, Kafka, HTTPS | HTTP/HTTPS | AMQP, HTTP |
| **Use Case** | Телеметрия | Reactive events | Транзакционные сообщения |
| **Push / Pull** | Pull | Push (+ Pull) | Pull |

---

# Когда использовать Event Hubs

- ✅ High-volume ingestion (IoT, логи, метрики)
- ✅ Real-time analytics
- ✅ Replay и reprocessing
- ✅ Миграция Kafka
- ✅ Big data pipeline

---

# Когда НЕ использовать Event Hubs

- ❌ Сложные message workflows → Service Bus
- ❌ Push-based event routing → Event Grid
- ❌ Низкообъёмные транзакционные сообщения → Service Bus
- ❌ Гарантированный глобальный порядок сообщений → Service Bus (sessions)

---

## Архитектурный акцент

- Event Hubs = ingestion + streaming
- Event Grid = reactive event routing
- Service Bus = transactional messaging

---

## Важно для AZ-204

Нужно чётко различать:

- Streaming → Event Hubs
- Pub/Sub событий → Event Grid
- Очереди и бизнес-процессы → Service Bus
- Ordering только внутри partition
- Retention временной

Если в вопросе фигурируют миллионы событий или потоковая аналитика — это Event Hubs.
---

## Quick Start Example

### Step 1: Create Event Hubs Resources

```bash
# Variables
RG="rg-eventhubs"
LOCATION="eastus"
NAMESPACE="myeventhubns"
EVENTHUB="myeventhub"

# Create resource group
az group create --name $RG --location $LOCATION

# Create namespace
az eventhubs namespace create \
  --name $NAMESPACE \
  --resource-group $RG \
  --location $LOCATION \
  --sku Standard

# Create event hub
az eventhubs eventhub create \
  --name $EVENTHUB \
  --namespace-name $NAMESPACE \
  --resource-group $RG \
  --partition-count 4 \
  --message-retention 7

# Get connection string
az eventhubs namespace authorization-rule keys list \
  --name RootManageSharedAccessKey \
  --namespace-name $NAMESPACE \
  --resource-group $RG \
  --query primaryConnectionString \
  --output tsv
```

### Step 2: Send Events (C#)

```csharp
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Producer;

string connectionString = "<connection-string>";
string eventHubName = "myeventhub";

await using var producer = new EventHubProducerClient(connectionString, eventHubName);

// Create batch
using EventDataBatch eventBatch = await producer.CreateBatchAsync();

for (int i = 0; i < 10; i++)
{
    eventBatch.TryAdd(new EventData($"Event {i}"));
}

// Send batch
await producer.SendAsync(eventBatch);
Console.WriteLine("Events sent successfully!");
```

### Step 3: Receive Events (C#)

```csharp
using Azure.Messaging.EventHubs.Consumer;

string connectionString = "<connection-string>";
string eventHubName = "myeventhub";
string consumerGroup = EventHubConsumerClient.DefaultConsumerGroupName;

await using var consumer = new EventHubConsumerClient(consumerGroup, connectionString, eventHubName);

await foreach (PartitionEvent partitionEvent in consumer.ReadEventsAsync())
{
    Console.WriteLine($"Partition: {partitionEvent.Partition.PartitionId}");
    Console.WriteLine($"Event: {partitionEvent.Data.EventBody}");
}
```

---

# Best Practices для Azure Event Hubs

## Design Patterns

### 1️⃣ Стратегия Partition Key

- Используйте partition key для группировки связанных событий
- Обеспечьте равномерное распределение нагрузки
- Примеры: User ID, Device ID, Session ID

Правильный partition key:
- сохраняет порядок внутри partition
- предотвращает hot partition

---

### 2️⃣ Отправка батчами (Batching)

- Отправляйте события пакетами
- Уменьшайте сетевые накладные расходы
- Используйте `CreateBatchAsync()`

Batching увеличивает throughput и снижает стоимость.

---

### 3️⃣ Consumer Groups

- Создавайте отдельную consumer group для каждого приложения
- Не используйте одну группу для нескольких сервисов
- Используйте понятные имена

---

### 4️⃣ Обработка ошибок

- Реализуйте retry с exponential backoff
- Обрабатывайте transient ошибки
- Настройте мониторинг persistent ошибок

---

### 5️⃣ Checkpointing

- Делайте checkpoint регулярно (но не после каждого события)
- Балансируйте отказоустойчивость и производительность
- Лучше делать checkpoint после обработки batch

---

# Оптимизация производительности

- **Compression** — уменьшение размера payload
- **Batching** — группировка отправки
- **Равномерное распределение по partitions**
- **Переиспользование клиентов** (connection pooling)
- **Асинхронные операции (async/await)**

---

# Security Best Practices

- ✅ Использовать Managed Identity вместо connection string
- ✅ Настроить VNet integration и Private Endpoints
- ✅ Конфигурировать IP firewall
- ✅ Использовать Azure RBAC
- ✅ Регулярно ротировать ключи
- ✅ Включить diagnostic logging
- ✅ Использовать TLS 1.2+

---

# Советы к экзамену AZ-204

## Ключевые моменты

1️⃣ Event Hubs = платформа для big data streaming  
2️⃣ Partitions — единица масштабирования (нельзя изменить после создания)  
3️⃣ Consumer Groups — независимые представления потока  
4️⃣ 1 TU = 1 MB/s ingress, 2 MB/s egress  
5️⃣ Checkpointing требует Blob Storage  
6️⃣ Поддержка Kafka без изменения кода  
7️⃣ Retention зависит от tier (1–90 дней)

---

# Частые экзаменационные сценарии

### Сценарий 1
IoT-телеметрия большого объёма
- ✅ Event Hubs
- ❌ Не Event Grid

---

### Сценарий 2
Несколько приложений читают один поток
- ✅ Отдельные consumer groups
- ❌ Не использовать одну группу

---

### Сценарий 3
Требуется порядок событий
- ✅ Использовать partition key
- ❌ Не ожидать глобального порядка

---

### Сценарий 4
Миграция Kafka
- ✅ Kafka endpoint Event Hubs
- ✅ Без изменений кода

---

### Сценарий 5
Масштабирование обработки
- ✅ EventProcessorClient
- ✅ Несколько экземпляров → авто-балансировка

---

# Что обязательно помнить

- Partition count нельзя изменить
- Максимальный размер события — 1 MB
- До 20 consumer groups (Standard)
- Partition key определяет распределение
- EventProcessorClient — production-паттерн
- Checkpointing требует Blob Storage
- Kafka порт: 9093 (SASL_SSL)

---

## Архитектурный вывод

Event Hubs используется, когда нужны:

- Высокий throughput
- Потоковая аналитика
- Replay и reprocessing
- Kafka-совместимость
- Масштабируемая обработка

Если в вопросе фигурируют миллионы событий в секунду — это почти всегда Event Hubs.

### Quick Command Reference

```bash
# Create namespace
az eventhubs namespace create --name <ns> --resource-group <rg> --sku Standard

# Create event hub
az eventhubs eventhub create --name <eh> --namespace-name <ns> --resource-group <rg> --partition-count 4

# Create consumer group
az eventhubs eventhub consumer-group create --name <cg> --eventhub-name <eh> --namespace-name <ns> --resource-group <rg>

# Get connection string
az eventhubs namespace authorization-rule keys list --name RootManageSharedAccessKey --namespace-name <ns> --resource-group <rg>
```

---

# Итоги по Azure Event Hubs

**Azure Event Hubs** — это полностью управляемая платформа потоковой передачи данных в реальном времени, предназначенная для сценариев Big Data и high-throughput ingestion.

---

# Основные компоненты

- **Namespace**  
  Контейнер управления ресурсами Event Hubs

- **Event Hub**  
  Append-only распределённый лог событий

- **Partitions**  
  Упорядоченные последовательности событий (единица масштабирования)

- **Consumer Groups**  
  Независимые представления потока

- **Producers**  
  Отправляют события в Event Hub

- **Consumers**  
  Читают и обрабатывают события

---

# Ключевые возможности

- Обработка миллионов событий в секунду
- Retention от 1 до 90 дней (зависит от tier)
- Совместимость с Apache Kafka
- Partitioned consumer model
- Масштабирование через Throughput Units / Processing Units

---

# Когда использовать Event Hubs

- Приём IoT-телеметрии
- Централизованный сбор логов и метрик
- Clickstream-аналитика
- Реалтайм-дашборды
- Big data pipeline
- Миграция Kafka-нагрузки в Azure

---

## Архитектурный вывод

Event Hubs — это:

- Ingestion layer для потоковых данных
- Буфер между producers и аналитикой
- Основа для real-time и batch processing

Если задача связана с high-volume streaming — правильный выбор обычно Event Hubs.