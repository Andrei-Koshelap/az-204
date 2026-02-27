# Масштабирование приложения обработки событий

## Проблемы масштабирования при работе с потоками событий

### Проблема

Представим сценарий: **100 000 домов** отправляют данные с датчиков в Event Hubs.

### Требования:

- ✅ **Масштабируемость (Scale)** — динамическое добавление или удаление обработчиков
- ✅ **Балансировка нагрузки (Load Balance)** — равномерное распределение партиций между consumer-инстансами
- ✅ **Отказоустойчивость (Fault Tolerance)** — продолжение обработки после сбоя consumer
- ✅ **Эффективность** — отсутствие дублирования и потери данных
- ✅ **Checkpointing** — отслеживание прогресса обработки по каждой партиции

---

## В чём сложность?

Event Hub делит поток событий на **партиции**.

Каждая партиция:

- сохраняет порядок событий
- может обрабатываться только одним consumer в рамках одной consumer group

Если запустить несколько инстансов без координации:

- возможны конфликты при чтении
- дублирование обработки
- потеря прогресса
- неравномерная нагрузка

---

## Решение

Использовать:

- **EventProcessorClient** — рекомендуемый способ  
  или
- **EventHubConsumerClient** с ручной координацией (более сложный вариант)

---

## Почему EventProcessorClient — лучший выбор?

EventProcessorClient автоматически обеспечивает:

### 1. Балансировку партиций
- Распределяет партиции между активными инстансами
- Перераспределяет их при добавлении или удалении consumer

### 2. Отказоустойчивость
- При падении одного инстанса его партиции перераспределяются
- Обработка продолжается другими инстансами

### 3. Checkpointing
- Сохраняет позицию чтения в Azure Blob Storage
- Позволяет продолжить обработку с последней зафиксированной позиции

### 4. Отсутствие конфликтов
- Использует механизм lease для координации
- Только один consumer обрабатывает конкретную партицию

---



---

## Partitioned Consumer Pattern

### Traditional Pattern: Competing Consumers

```
Queue (Single Stream)
├── Event 1
├── Event 2
├── Event 3        ┌──────────────┐
├── Event 4  ───→  │  Consumer 1  │ (reads any available event)
├── Event 5        └──────────────┘
├── Event 6        ┌──────────────┐
├── Event 7  ───→  │  Consumer 2  │ (reads any available event)
└── Event 8        └──────────────┘

Problem: Bottleneck at queue, limited scalability
```

### Event Hubs Pattern: Partitioned Consumers

```
Event Hub (Multiple Partitions)
├── Partition 0  ───→  ┌──────────────┐
├── Partition 1  ───→  │  Consumer 1  │ (owns P0, P1)
├── Partition 2  ───→  └──────────────┘
├── Partition 3        ┌──────────────┐
├── Partition 4  ───→  │  Consumer 2  │ (owns P2, P3, P4)
├── Partition 5  ───→  └──────────────┘
└── Partition 6        ┌──────────────┐
    Partition 7  ───→  │  Consumer 3  │ (owns P6, P7)
                       └──────────────┘

Benefit: Parallel processing, high scalability
```

**Key Principles:**
1. Each **partition** is assigned to **one consumer** at a time
2. A **consumer** can own **multiple partitions**
3. Partition ownership is **dynamically distributed**
4. When consumers join/leave, partitions are **rebalanced**

---

## EventProcessorClient Architecture

### Component Overview

```
┌──────────────────────────────────────────────────────────────┐
│                    CONSUMER GROUP                             │
│                                                               │
│  ┌──────────────────────┐         ┌──────────────────────┐  │
│  │ Consumer Instance 1  │         │ Consumer Instance 2  │  │
│  │                      │         │                      │  │
│  │ EventProcessorClient │         │ EventProcessorClient │  │
│  │  • Partition 0       │         │  • Partition 2       │  │
│  │  • Partition 1       │         │  • Partition 3       │  │
│  └──────────┬───────────┘         └──────────┬───────────┘  │
│             │                                 │              │
│             │     ┌──────────────────────┐   │              │
│             │     │ Consumer Instance 3  │   │              │
│             │     │                      │   │              │
│             │     │ EventProcessorClient │   │              │
│             │     │  • Partition 4       │   │              │
│             │     │  • Partition 5       │   │              │
│             │     └──────────┬───────────┘   │              │
│             │                │                │              │
└─────────────┼────────────────┼────────────────┼──────────────┘
              │                │                │
              ▼                ▼                ▼
    ┌─────────────────────────────────────────────────────┐
    │         CHECKPOINT STORE (Blob Storage)             │
    │                                                     │
    │  Partition Ownership:                               │
    │  ├── Partition 0 → Instance 1 (lease expires: ...)│
    │  ├── Partition 1 → Instance 1 (lease expires: ...)│
    │  ├── Partition 2 → Instance 2 (lease expires: ...)│
    │  ├── Partition 3 → Instance 2 (lease expires: ...)│
    │  ├── Partition 4 → Instance 3 (lease expires: ...)│
    │  └── Partition 5 → Instance 3 (lease expires: ...)│
    │                                                     │
    │  Checkpoints (per partition):                       │
    │  ├── Partition 0: Offset 12345, Sequence 67890    │
    │  ├── Partition 1: Offset 23456, Sequence 78901    │
    │  ├── Partition 2: Offset 34567, Sequence 89012    │
    │  └── ...                                           │
    └─────────────────────────────────────────────────────┘
              ▲                ▲                ▲
              │                │                │
              │ Read/Write     │ Read/Write     │ Read/Write
              │ Ownership      │ Ownership      │ Ownership
              │ Checkpoints    │ Checkpoints    │ Checkpoints
              │                │                │
              └────────────────┴────────────────┘
```

### Как работает EventProcessorClient

EventProcessorClient — это высокоуровневый механизм координации обработки событий из Event Hubs с автоматической балансировкой и отказоустойчивостью.

---

## 1. Захват владения партицией (Partition Ownership Claim)

- Каждый экземпляр EventProcessorClient пытается «захватить» одну или несколько партиций
- Владение отслеживается через **blob leases** в Azure Blob Storage
- Длительность lease: обычно 15–60 секунд (настраивается)
- Lease регулярно продлевается (каждые несколько секунд), чтобы сохранить владение

Если lease не продлён — считается, что инстанс недоступен.

📌 Blob Storage здесь используется как координационный механизм (distributed lock).

---

## 2. Балансировка нагрузки (Load Balancing)

EventProcessorClient постоянно отслеживает распределение партиций.

Перераспределение происходит, когда:

- Запускается новый consumer-инстанс
- Останавливается существующий инстанс
- Не удаётся продлить lease (например, из-за сбоя)

Цель — обеспечить максимально равномерное распределение партиций между активными инстансами.

Важно:

- Одна партиция обрабатывается только одним consumer в рамках consumer group
- При увеличении числа инстансов нагрузка автоматически распределяется

---

## 3. Checkpointing (Сохранение прогресса)

Consumer периодически сохраняет состояние обработки.

Checkpoint включает:

- ID партиции
- Offset (позиция в партиции)
- Sequence number

Checkpoint хранится в **checkpoint store** (обычно Azure Blob Storage).

📌 Рекомендуется сохранять checkpoint после успешной обработки события или батча событий.

Если checkpoint не выполняется корректно — возможна повторная обработка при перезапуске.

---

## 4. Восстановление после сбоя (Failure Recovery)

Когда consumer падает:

1. Lease истекает
2. Другой инстанс захватывает партицию
3. Обработка продолжается с последнего checkpoint

Это обеспечивает:

- отказоустойчивость
- отсутствие потери данных
- минимизацию дублирования (при корректном checkpointing)

---

## Что важно помнить для AZ-204

- EventProcessorClient автоматически управляет lease и балансировкой
- Координация выполняется через Blob Storage
- Checkpointing критичен для корректного восстановления
- При сбое обработка продолжается другим инстансом
- Без checkpoint возможна повторная обработка

---

## Ключевая идея

EventProcessorClient решает сразу несколько задач распределённой системы:

- распределение нагрузки
- синхронизация
- обработка отказов
- отслеживание прогресса

Это стандартный способ построения масштабируемого и устойчивого consumer-приложения для Event Hubs.

---

## EventProcessorClient Implementation

### .NET Implementation

**Install NuGet Package:**
```bash
dotnet add package Azure.Messaging.EventHubs
dotnet add package Azure.Messaging.EventHubs.Processor
dotnet add package Azure.Storage.Blobs
```

**Basic Implementation:**

```csharp
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Consumer;
using Azure.Messaging.EventHubs.Processor;
using Azure.Storage.Blobs;
using System.Text;

// Connection strings
string eventHubsConnectionString = "<event-hubs-connection-string>";
string eventHubName = "telemetry";
string consumerGroup = EventHubConsumerClient.DefaultConsumerGroupName;

// Checkpoint store
string blobStorageConnectionString = "<storage-connection-string>";
string blobContainerName = "checkpoints";

// Create blob container client for checkpoint store
BlobContainerClient storageClient = new BlobContainerClient(
    blobStorageConnectionString,
    blobContainerName
);

// Create the container if it doesn't exist
await storageClient.CreateIfNotExistsAsync();

// Create event processor client
EventProcessorClient processor = new EventProcessorClient(
    storageClient,
    consumerGroup,
    eventHubsConnectionString,
    eventHubName
);

// Register event handler
processor.ProcessEventAsync += async (ProcessEventArgs args) =>
{
    try
    {
        // Access event data
        string partition = args.Partition.PartitionId;
        byte[] eventBody = args.Data.EventBody.ToArray();
        string bodyText = Encoding.UTF8.GetString(eventBody);
        
        Console.WriteLine($"Partition: {partition}");
        Console.WriteLine($"Offset: {args.Data.Offset}");
        Console.WriteLine($"Sequence: {args.Data.SequenceNumber}");
        Console.WriteLine($"Event: {bodyText}");
        Console.WriteLine("---");
        
        // Process event (your business logic)
        await ProcessEventAsync(bodyText);
        
        // Update checkpoint
        await args.UpdateCheckpointAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error processing event: {ex.Message}");
        // Don't checkpoint on error
    }
};

// Register error handler
processor.ProcessErrorAsync += (ProcessErrorEventArgs args) =>
{
    Console.WriteLine($"Error in partition {args.PartitionId}: {args.Exception.Message}");
    return Task.CompletedTask;
};

// Start processing
await processor.StartProcessingAsync();
Console.WriteLine("Event processor started. Press Enter to stop.");
Console.ReadLine();

// Stop processing
await processor.StopProcessingAsync();
Console.WriteLine("Event processor stopped.");

async Task ProcessEventAsync(string eventData)
{
    // Your business logic here
    // Example: Parse JSON, save to database, send alert, etc.
    await Task.Delay(10); // Simulate processing
}
```

**Advanced Configuration:**

```csharp
var options = new EventProcessorClientOptions
{
    // Maximum wait time for events
    MaximumWaitTime = TimeSpan.FromSeconds(30),
    
    // Track last enqueued event properties
    TrackLastEnqueuedEventProperties = true,
    
    // Retry options
    RetryOptions = new EventHubsRetryOptions
    {
        MaximumRetries = 5,
        Delay = TimeSpan.FromSeconds(1),
        MaximumDelay = TimeSpan.FromSeconds(30),
        Mode = EventHubsRetryMode.Exponential
    },
    
    // Connection options
    ConnectionOptions = new EventHubConnectionOptions
    {
        TransportType = EventHubsTransportType.AmqpTcp
    }
};

EventProcessorClient processor = new EventProcessorClient(
    storageClient,
    consumerGroup,
    eventHubsConnectionString,
    eventHubName,
    options
);
```

### Python Implementation

**Install Package:**
```bash
pip install azure-eventhub
pip install azure-eventhub-checkpointstoreblob
pip install azure-storage-blob
```

**Implementation:**

```python
from azure.eventhub import EventHubConsumerClient
from azure.eventhub.extensions.checkpointstoreblobaio import BlobCheckpointStore
import asyncio

# Connection strings
connection_string = "<event-hubs-connection-string>"
eventhub_name = "telemetry"
consumer_group = "$Default"

# Checkpoint store
storage_connection_string = "<storage-connection-string>"
container_name = "checkpoints"

async def on_event(partition_context, event):
    """Process event"""
    try:
        # Access event data
        partition_id = partition_context.partition_id
        offset = event.offset
        sequence_number = event.sequence_number
        body = event.body_as_str()
        
        print(f"Partition: {partition_id}")
        print(f"Offset: {offset}")
        print(f"Sequence: {sequence_number}")
        print(f"Event: {body}")
        print("---")
        
        # Process event (your business logic)
        await process_event(body)
        
        # Update checkpoint
        await partition_context.update_checkpoint(event)
        
    except Exception as e:
        print(f"Error processing event: {e}")
        # Don't checkpoint on error

async def on_error(partition_context, error):
    """Handle errors"""
    if partition_context:
        print(f"Error in partition {partition_context.partition_id}: {error}")
    else:
        print(f"Error: {error}")

async def process_event(event_data):
    """Your business logic"""
    await asyncio.sleep(0.01)  # Simulate processing

async def main():
    # Create checkpoint store
    checkpoint_store = BlobCheckpointStore.from_connection_string(
        storage_connection_string,
        container_name
    )
    
    # Create consumer client
    client = EventHubConsumerClient.from_connection_string(
        connection_string,
        consumer_group,
        eventhub_name=eventhub_name,
        checkpoint_store=checkpoint_store
    )
    
    async with client:
        # Start processing
        await client.receive(
            on_event=on_event,
            on_error=on_error
        )

if __name__ == "__main__":
    asyncio.run(main())
```

### JavaScript/TypeScript Implementation

**Install Packages:**
```bash
npm install @azure/event-hubs
npm install @azure/storage-blob
```

**Implementation:**

```javascript
const { EventHubConsumerClient } = require("@azure/event-hubs");
const { ContainerClient } = require("@azure/storage-blob");
const { BlobCheckpointStore } = require("@azure/eventhubs-checkpointstore-blob");

// Connection strings
const connectionString = "<event-hubs-connection-string>";
const eventHubName = "telemetry";
const consumerGroup = "$Default";

// Checkpoint store
const storageConnectionString = "<storage-connection-string>";
const containerName = "checkpoints";

async function main() {
    // Create checkpoint store
    const containerClient = new ContainerClient(
        storageConnectionString,
        containerName
    );
    
    await containerClient.createIfNotExists();
    
    const checkpointStore = new BlobCheckpointStore(containerClient);
    
    // Create consumer client
    const consumerClient = new EventHubConsumerClient(
        consumerGroup,
        connectionString,
        eventHubName,
        checkpointStore
    );
    
    // Subscribe to events
    const subscription = consumerClient.subscribe({
        processEvents: async (events, context) => {
            for (const event of events) {
                try {
                    console.log(`Partition: ${context.partitionId}`);
                    console.log(`Offset: ${event.offset}`);
                    console.log(`Sequence: ${event.sequenceNumber}`);
                    console.log(`Event: ${event.body}`);
                    console.log("---");
                    
                    // Process event
                    await processEvent(event.body);
                    
                    // Update checkpoint
                    await context.updateCheckpoint(event);
                } catch (err) {
                    console.error(`Error processing event: ${err}`);
                }
            }
        },
        processError: async (err, context) => {
            console.error(`Error in partition ${context.partitionId}: ${err}`);
        }
    });
    
    // Wait for termination signal
    console.log("Event processor started. Press Ctrl+C to stop.");
    await new Promise(() => {});
}

async function processEvent(eventData) {
    // Your business logic
    await new Promise(resolve => setTimeout(resolve, 10));
}

main().catch((err) => {
    console.error("Error:", err);
});
```

---

## Partition Ownership and Load Balancing

### Partition Ownership Tracking

**Storage Structure (Blob Storage):**

```
Container: "checkpoints"
├── <eventhub-name>/
│   ├── <consumer-group>/
│   │   ├── ownership/
│   │   │   ├── 0  ← Partition 0 ownership (blob lease)
│   │   │   ├── 1  ← Partition 1 ownership
│   │   │   ├── 2  ← Partition 2 ownership
│   │   │   └── 3  ← Partition 3 ownership
│   │   └── checkpoint/
│   │       ├── 0  ← Partition 0 checkpoint
│   │       ├── 1  ← Partition 1 checkpoint
│   │       ├── 2  ← Partition 2 checkpoint
│   │       └── 3  ← Partition 3 checkpoint
```

**Ownership Blob Content (JSON):**

```json
{
  "ownerIdentifier": "consumer-instance-1",
  "partitionId": "0",
  "eventHubName": "telemetry",
  "consumerGroup": "$Default",
  "fullyQualifiedNamespace": "myeventhubns.servicebus.windows.net",
  "lastModifiedTime": "2024-01-15T10:30:00Z",
  "eTag": "0x8D9E5F7A8B3C1D2"
}
```

**Checkpoint Blob Content (JSON):**

```json
{
  "partitionId": "0",
  "offset": "12345",
  "sequenceNumber": 67890,
  "eventHubName": "telemetry",
  "consumerGroup": "$Default",
  "fullyQualifiedNamespace": "myeventhubns.servicebus.windows.net"
}
```

### Load Balancing Example

**Scenario: 8 Partitions, Consumer Instances Scale**

```
Time T0: 1 Consumer Instance
┌────────────────────────┐
│   Consumer Instance 1  │
│   Owns: P0-P7 (all 8)  │
└────────────────────────┘

Time T1: 2 Consumer Instances (scale out)
┌────────────────────────┐  ┌────────────────────────┐
│   Consumer Instance 1  │  │   Consumer Instance 2  │
│   Owns: P0-P3 (4)      │  │   Owns: P4-P7 (4)      │
└────────────────────────┘  └────────────────────────┘
         ▲                           ▲
         └───── Load Balanced ───────┘

Time T2: 4 Consumer Instances (scale out further)
┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│  Instance 1  │  │  Instance 2  │  │  Instance 3  │  │  Instance 4  │
│  Owns: P0-P1 │  │  Owns: P2-P3 │  │  Owns: P4-P5 │  │  Owns: P6-P7 │
└──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘

Time T3: 1 Consumer Instance (scale in - Instance 2, 3, 4 stopped)
┌────────────────────────┐
│   Consumer Instance 1  │
│   Owns: P0-P7 (all 8)  │
└────────────────────────┘
         ▲
         └─── Takes over all partitions
```

## Алгоритм балансировки нагрузки (Load Balancing Algorithm)

EventProcessorClient выполняет балансировку циклически.

### 1. Цикл захвата владения (Ownership Claim Cycle)
(обычно каждые 10–30 секунд)

- Подсчитывается общее количество партиций
- Подсчитывается количество активных consumer-инстансов
- Вычисляется «справедливая доля»:

  `количество партиций / количество consumers`

Это число определяет, сколько партиций должен обрабатывать каждый инстанс.

---

### 2. Распределение партиций

- Если consumer владеет **меньше**, чем справедливая доля → он пытается захватить дополнительные партиции
- Если consumer владеет **больше**, чем справедливая доля → он освобождает лишние партиции
- Баланс достигается постепенно, за несколько циклов

📌 Балансировка не мгновенная — она сходится итеративно.

---

### 3. Управление lease

- Длительность lease: 15–60 секунд
- Продление lease: каждые несколько секунд
- Если lease истёк → партиция становится доступной для захвата

Если consumer падает, его lease не продлевается — другие инстансы автоматически забирают партиции.

---

# Стратегии Checkpointing

Checkpoint определяет, с какой позиции продолжится обработка после сбоя.

## Когда выполнять checkpoint?

| Стратегия | Частота | Плюсы | Минусы | Подходит для |
|------------|----------|--------|--------|--------------|
| **Каждое событие** | После каждого события | Максимальная отказоустойчивость | Высокая нагрузка, снижение производительности | Критичные финансовые операции |
| **Каждый батч** | После обработки батча | Хороший баланс | Возможна повторная обработка части данных | Стандартные приложения |
| **По времени** | Каждые N секунд | Предсказуемость | Возможна потеря части прогресса | Высоконагруженные системы |
| **По количеству** | Каждые N событий | Контролируемый объём | Время между checkpoint может варьироваться | Средняя нагрузка |
| **Гибридная** | Батч + таймер | Оптимальный баланс | Более сложная реализация | Рекомендуется для production |

---

## Практические рекомендации

- Не выполняйте checkpoint слишком часто — это увеличивает нагрузку на Blob Storage
- Не выполняйте checkpoint слишком редко — возрастает риск повторной обработки
- В production чаще всего используется гибридный подход
- Всегда выполняйте checkpoint **после успешной обработки**, а не до неё

---

## Что важно для AZ-204

- Балансировка выполняется циклически
- Используются blob leases
- Checkpoint хранится в Blob Storage
- При сбое обработка продолжается с последнего checkpoint
- Без checkpoint возможна повторная обработка

---

## Ключевая идея

Балансировка + lease + checkpoint =

масштабируемая, отказоустойчивая и согласованная обработка событий в распределённой системе.

Понимание этих механизмов — основа для правильного ответа на вопросы по масштабированию Event Hubs в AZ-204.

### Checkpointing Examples

**Example 1: Checkpoint Every Batch (Recommended)**

```csharp
processor.ProcessEventAsync += async (ProcessEventArgs args) =>
{
    try
    {
        await ProcessEventAsync(args.Data);
        
        // Checkpoint after each event (simple but overhead)
        await args.UpdateCheckpointAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
        // Don't checkpoint on error
    }
};
```

**Example 2: Checkpoint Every N Events**

```csharp
private static int eventCount = 0;
private static readonly int checkpointFrequency = 100;

processor.ProcessEventAsync += async (ProcessEventArgs args) =>
{
    try
    {
        await ProcessEventAsync(args.Data);
        
        // Increment counter
        Interlocked.Increment(ref eventCount);
        
        // Checkpoint every 100 events
        if (eventCount % checkpointFrequency == 0)
        {
            await args.UpdateCheckpointAsync();
            Console.WriteLine($"Checkpointed at event {eventCount}");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
    }
};
```

**Example 3: Checkpoint Every N Seconds**

```csharp
private static DateTime lastCheckpoint = DateTime.UtcNow;
private static readonly TimeSpan checkpointInterval = TimeSpan.FromSeconds(30);

processor.ProcessEventAsync += async (ProcessEventArgs args) =>
{
    try
    {
        await ProcessEventAsync(args.Data);
        
        // Checkpoint every 30 seconds
        if (DateTime.UtcNow - lastCheckpoint >= checkpointInterval)
        {
            await args.UpdateCheckpointAsync();
            lastCheckpoint = DateTime.UtcNow;
            Console.WriteLine($"Checkpointed at {lastCheckpoint}");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
    }
};
```

**Example 4: Hybrid Strategy (Best Practice)**

```csharp
private static int eventCount = 0;
private static DateTime lastCheckpoint = DateTime.UtcNow;
private static readonly int checkpointEventCount = 100;
private static readonly TimeSpan checkpointInterval = TimeSpan.FromSeconds(30);

processor.ProcessEventAsync += async (ProcessEventArgs args) =>
{
    try
    {
        await ProcessEventAsync(args.Data);
        
        Interlocked.Increment(ref eventCount);
        
        // Checkpoint if either condition met
        bool shouldCheckpoint = 
            eventCount >= checkpointEventCount ||
            DateTime.UtcNow - lastCheckpoint >= checkpointInterval;
        
        if (shouldCheckpoint)
        {
            await args.UpdateCheckpointAsync();
            eventCount = 0;
            lastCheckpoint = DateTime.UtcNow;
            Console.WriteLine($"Checkpointed: Events={eventCount}, Time={lastCheckpoint}");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error: {ex.Message}");
    }
};
```

---

## Thread Safety and Concurrency

### EventProcessorClient Thread Safety

**Key Guarantees:**

1. **Sequential per Partition**: Events from same partition processed sequentially
2. **Concurrent across Partitions**: Different partitions processed in parallel
3. **Thread-Safe**: EventProcessorClient is thread-safe

**Processing Model:**

```
EventProcessorClient
├── Partition 0 → Thread 1 (sequential: E1 → E2 → E3)
├── Partition 1 → Thread 2 (sequential: E4 → E5 → E6)
├── Partition 2 → Thread 3 (sequential: E7 → E8 → E9)
└── Partition 3 → Thread 4 (sequential: E10 → E11 → E12)

Thread 1, 2, 3, 4 run concurrently (parallel processing)
Within each thread, events processed sequentially
```

**Implications:**

✅ **Safe**: Share EventProcessorClient across threads
✅ **Ordering**: Events in same partition maintain order
✅ **Parallelism**: Maximum parallelism = number of partitions
❌ **Blocking**: Slow processing in one partition doesn't block others

### Handling Concurrent Processing

**Bad Practice: Blocking Processing**

```csharp
// DON'T DO THIS - Blocks processing
processor.ProcessEventAsync += async (ProcessEventArgs args) =>
{
    Thread.Sleep(5000);  // Blocks thread for 5 seconds!
    await ProcessEventAsync(args.Data);
    await args.UpdateCheckpointAsync();
};
```

**Good Practice: Async Processing**

```csharp
// DO THIS - Non-blocking async processing
processor.ProcessEventAsync += async (ProcessEventArgs args) =>
{
    await Task.Delay(5000);  // Async delay, doesn't block
    await ProcessEventAsync(args.Data);
    await args.UpdateCheckpointAsync();
};
```

**Advanced: Parallel Processing within Partition (Use Carefully)**

```csharp
processor.ProcessEventAsync += async (ProcessEventArgs args) =>
{
    // Process event in background task
    _ = Task.Run(async () =>
    {
        try
        {
            await ProcessEventAsync(args.Data);
            // Note: Cannot checkpoint here (lost context)
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Error: {ex.Message}");
        }
    });
    
    // Immediately return (fast acknowledgment)
    // Warning: Events may be processed out of order!
};
```

⚠️ **Warning**: Parallel processing within partition breaks ordering guarantees!

---

## Лучшие практики

---

# Оптимизация производительности

### 1. Batch Checkpointing
- Выполнять checkpoint каждые 50–100 событий или каждые ~30 секунд
- Снижает количество операций записи в Blob Storage
- Баланс между отказоустойчивостью и производительностью

---

### 2. Асинхронная обработка
- Использовать async/await на всех уровнях
- Не блокировать потоки
- Эффективно использовать I/O-конкурентность

---

### 3. Управление ресурсами
- Переиспользовать клиентов и соединения
- Корректно освобождать ресурсы
- Использовать connection pooling

---

### 4. Количество партиций
- Больше партиций → больше параллелизма
- Количество партиций должно соответствовать ожидаемому числу consumer-инстансов
- Планировать масштабирование заранее

---

# Отказоустойчивость

### 1. Обработка ошибок
- Перехватывать исключения в обработчике событий
- Не выполнять checkpoint при ошибке (чтобы событие могло быть обработано повторно)
- Логировать ошибки
- Реализовать dead-letter очередь для «ядовитых» сообщений

---

### 2. Корректное завершение работы (Graceful Shutdown)

- Перед остановкой приложения корректно завершать обработку
- Освобождать партиции
- Гарантировать сохранение последнего checkpoint

---

### 3. Идемпотентность
- Проектировать обработку как идемпотентную
- Корректно обрабатывать дубликаты
- Использовать уникальные идентификаторы событий

---

### 4. Мониторинг
- Отслеживать lag (задержку обработки)
- Контролировать распределение партиций
- Настраивать алерты на ошибки обработки

---

# Масштабируемость

### 1. Горизонтальное масштабирование
- Добавлять новые consumer-инстансы
- Автоматическая балансировка нагрузки
- Максимальное количество инстансов = количество партиций

---

### 2. Вертикальное масштабирование
- Увеличивать CPU / память
- Оптимизировать логику обработки
- Использовать быстрое хранилище для checkpoint

---

### 3. Автомасштабирование
- Масштабировать на основе consumer lag
- Использовать Azure Container Instances или AKS
- Применять KEDA для event-driven autoscaling

---

# Советы к экзамену AZ-204

## Ключевые концепции

1. **EventProcessorClient** — рекомендуемый production-клиент (автоматическая балансировка)
2. **Checkpoint Store** — Azure Blob Storage (хранит прогресс и владение партициями)
3. **Partition Ownership** — одна партиция принадлежит одному consumer одновременно
4. **Load Balancing** — автоматическое распределение партиций
5. **Checkpointing** — фиксация позиции обработки (offset + sequence number)
6. **Fault Tolerance** — восстановление с последнего checkpoint
7. **Thread Safety** — последовательная обработка внутри партиции, параллельная между партициями

---

## Типовые экзаменационные сценарии

### Сценарий 1: Динамическое масштабирование обработки

✔ Использовать EventProcessorClient  
✔ Развернуть несколько инстансов  
✔ Автоматическая балансировка

---

### Сценарий 2: Отслеживание прогресса обработки

✔ Использовать checkpointing  
✔ Требуется Blob Storage  
✔ При сбое обработка продолжается с последнего checkpoint

---

### Сценарий 3: Максимальный параллелизм

✔ Количество consumer-инстансов ≤ количество партиций  
✔ Пример: 16 партиций → максимум 16 consumer

---

### Сценарий 4: Гарантия порядка событий

✔ Использовать partition key  
✔ Внутри одной партиции порядок сохраняется  
✘ Между партициями порядок не гарантируется

---

## Что обязательно помнить

- **EventProcessorClient** — рекомендован для production
- **EventHubConsumerClient** — больше подходит для прототипирования
- Checkpoint store требует Azure Blob Storage
- Владение партициями отслеживается через blob leases (15–60 секунд)
- Балансировка выполняется автоматически каждые 10–30 секунд
- Максимальное количество consumer ограничено числом партиций
- Обработка последовательная внутри партиции и параллельная между партициями

---

## Ключевая идея

Масштабирование обработки в Event Hubs строится вокруг трёх механизмов:

- Партиционирование
- Lease-механизм
- Checkpointing

Именно их понимание позволяет правильно отвечать на вопросы по масштабированию и отказоустойчивости в AZ-204.
### Quick Reference

```csharp
// EventProcessorClient setup
var storageClient = new BlobContainerClient(storageConnString, containerName);
var processor = new EventProcessorClient(storageClient, consumerGroup, ehConnString, ehName);

// Event handler
processor.ProcessEventAsync += async (args) => {
    await ProcessEvent(args.Data);
    await args.UpdateCheckpointAsync();  // Checkpoint
};

// Error handler
processor.ProcessErrorAsync += (args) => {
    Console.WriteLine($"Error: {args.Exception.Message}");
    return Task.CompletedTask;
};

// Start/stop
await processor.StartProcessingAsync();
await processor.StopProcessingAsync();
```

---

## Итог

**Масштабирование обработки событий** требует координации между несколькими consumer-инстансами, чтобы эффективно обрабатывать события из нескольких партиций без конфликтов и потери данных.

---

## Ключевые компоненты

- **EventProcessorClient**  
  Обеспечивает автоматическую балансировку нагрузки и отказоустойчивость

- **Checkpoint Store**  
  Azure Blob Storage для хранения прогресса обработки и информации о владении партициями

- **Partition Ownership**  
  Одна партиция принадлежит одному consumer в рамках consumer group  
  (динамическое распределение)

- **Checkpointing**  
  Отслеживание прогресса обработки (offset + sequence number)

---

## Преимущества

- Горизонтальная масштабируемость  
  (динамическое добавление и удаление consumer-инстансов)

- Отказоустойчивость  
  (восстановление обработки с последнего checkpoint)

- Автоматическая балансировка нагрузки  
  (равномерное распределение партиций)

- Конкурентность  
  (параллельная обработка между партициями)

---

## Лучшие практики

- Выполнять checkpoint периодически, а не после каждого события
- Не выполнять checkpoint при ошибке обработки
- Проектировать систему с учётом идемпотентности
- Мониторить распределение партиций и lag обработки

---

## Главное для AZ-204

Если в вопросе говорится о:

- масштабировании consumer-приложения
- автоматической балансировке
- восстановлении после сбоя
- отслеживании прогресса обработки

— правильный ответ почти всегда связан с использованием **EventProcessorClient + Azure Blob Storage checkpoint store**.

Понимание этих механизмов — ключ к правильным ответам в теме масштабирования Event Hubs.