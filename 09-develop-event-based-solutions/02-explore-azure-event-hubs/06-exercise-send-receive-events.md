# Практическое задание: Отправка и получение событий в Event Hubs

## Обзор упражнения

В этом практическом задании вы:

- создадите Event Hubs namespace и Event Hub
- реализуете producer-приложение
- реализуете consumer-приложение
- протестируете масштабируемую обработку
- отследите поток событий через Azure Portal

**Оценочное время:** 30–40 минут

---

## Чему вы научитесь

- Создавать Event Hubs namespace и Event Hub
- Отправлять события с помощью EventHubProducerClient
- Читать события через EventHubConsumerClient
- Обрабатывать события через EventProcessorClient
- Мониторить метрики и поток событий

---

# Шаг 1. Создание инфраструктуры

### 1️⃣ Создать Event Hubs Namespace

В Azure Portal:

- Создать ресурс **Event Hubs**
- Выбрать tier (Standard рекомендуется для практики)
- Настроить регион и resource group

---

### 2️⃣ Создать Event Hub

Внутри namespace:

- Создать новый Event Hub
- Указать количество партиций (например, 4)
- Настроить retention (например, 1 день)

📌 Количество партиций влияет на масштабируемость.

---

# Шаг 2. Создание Producer-приложения

Задача:

- Подключиться к Event Hub
- Отправить события
- Использовать batching

Что проверить:

- Используется ли batch-отправка
- Указан ли partition key (при необходимости)
- Нет ли ошибок отправки

После запуска:

- Проверить метрику **Incoming Messages** в Azure Portal

---

# Шаг 3. Создание Consumer-приложения (простой сценарий)

Использовать EventHubConsumerClient:

- Подключиться к конкретной consumer group
- Начать чтение с `EventPosition.Earliest`
- Вывести события в консоль

📌 Подходит для прототипирования, но не для production.

---

# Шаг 4. Production-обработка через EventProcessorClient

Задача:

- Настроить Azure Blob Storage для checkpoint
- Использовать EventProcessorClient
- Реализовать обработчик событий
- Периодически выполнять checkpoint

Что проверить:

- Автоматическая балансировка
- Корректная запись checkpoint
- Поведение при перезапуске приложения

---

# Шаг 5. Мониторинг через Azure Portal

Проверить:

- Incoming Messages
- Outgoing Messages
- Throttled Requests
- Consumer Lag

📌 Метрики помогают понять производительность и узкие места.

---

# Что закрепляет это упражнение

- Разницу между Producer и Consumer
- Использование batching
- Понимание partition key
- Работу checkpointing
- Балансировку нагрузки

---

# Связь с AZ-204

Это упражнение покрывает:

- Отправку и получение событий
- Масштабируемую обработку
- Checkpointing
- Балансировку партиций
- Мониторинг Event Hubs

Если вы понимаете каждый этап этого задания — тема клиентской библиотеки Event Hubs для AZ-204 у вас хорошо проработана.

---
### Architecture

```
┌────────────────────┐
│  Producer App      │
│  (Console App)     │
│  • Send 100 events │
│  • Batching        │
│  • Partition key   │
└─────────┬──────────┘
          │
          ▼
┌─────────────────────────────┐
│  EVENT HUBS NAMESPACE       │
│  ┌───────────────────────┐  │
│  │  Event Hub: telemetry │  │
│  │  ┌────┬────┬────┬───┐ │  │
│  │  │ P0 │ P1 │ P2 │P3 │ │  │
│  │  └────┴────┴────┴───┘ │  │
│  └───────────────────────┘  │
└─────────────┬───────────────┘
              │
              ▼
┌──────────────────────────────┐
│  Consumer App                │
│  (Console App)               │
│  • Read all partitions       │
│  • Display events            │
└──────────────────────────────┘
              │
              ▼
┌──────────────────────────────┐
│  Event Processor App         │
│  (Console App)               │
│  • Automatic load balancing  │
│  • Checkpointing             │
│  • Fault tolerance           │
└──────────────────────────────┘
```

---

## Prerequisites

### Required Tools

- **Azure Subscription**: Free or paid subscription
- **.NET SDK**: 6.0 or later ([Download](https://dotnet.microsoft.com/download))
- **Azure CLI**: Latest version ([Install](https://docs.microsoft.com/cli/azure/install-azure-cli))
- **Code Editor**: Visual Studio Code or Visual Studio

### Verify Prerequisites

```bash
# Check .NET SDK
dotnet --version
# Output: 6.0.x or later

# Check Azure CLI
az --version
# Output: azure-cli 2.x.x

# Login to Azure
az login
```

---

## Part 1: Create Azure Resources

### Step 1: Define Variables

```bash
# Resource group
RESOURCE_GROUP="rg-eventhubs-lab"
LOCATION="eastus"

# Event Hubs
NAMESPACE="ehns-lab-$RANDOM"  # Must be globally unique
EVENTHUB="telemetry"

echo "Namespace: $NAMESPACE"
```

### Step 2: Create Resource Group

```bash
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION

# Output:
# {
#   "id": "/subscriptions/.../resourceGroups/rg-eventhubs-lab",
#   "location": "eastus",
#   "name": "rg-eventhubs-lab",
#   "properties": {
#     "provisioningState": "Succeeded"
#   }
# }
```

### Step 3: Create Event Hubs Namespace

```bash
az eventhubs namespace create \
  --name $NAMESPACE \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard \
  --capacity 1

# Wait for provisioning (1-2 minutes)
echo "Namespace created: $NAMESPACE.servicebus.windows.net"
```

**Namespace SKUs:**

| SKU | Max TUs | Features | Cost |
|-----|---------|----------|------|
| Basic | 20 | Basic features | Lowest |
| Standard | 40 | Consumer groups, Capture | Moderate |
| Premium | Processing Units | Dedicated, isolation | Higher |

### Step 4: Create Event Hub

```bash
az eventhubs eventhub create \
  --name $EVENTHUB \
  --namespace-name $NAMESPACE \
  --resource-group $RESOURCE_GROUP \
  --partition-count 4 \
  --message-retention 1

# Output:
# {
#   "name": "telemetry",
#   "partitionCount": 4,
#   "status": "Active",
#   "messageRetentionInDays": 1
# }
```

### Step 5: Create Consumer Group

```bash
az eventhubs eventhub consumer-group create \
  --name processor \
  --eventhub-name $EVENTHUB \
  --namespace-name $NAMESPACE \
  --resource-group $RESOURCE_GROUP

echo "Consumer group 'processor' created"
```

### Step 6: Get Connection String

```bash
# Get connection string
CONNECTION_STRING=$(az eventhubs namespace authorization-rule keys list \
  --name RootManageSharedAccessKey \
  --namespace-name $NAMESPACE \
  --resource-group $RESOURCE_GROUP \
  --query primaryConnectionString \
  --output tsv)

echo "Connection String: $CONNECTION_STRING"

# Save to file for later use
echo $CONNECTION_STRING > connection-string.txt
```

**⚠️ Security Note**: Connection string contains sensitive information. Never commit to source control!

---

## Part 2: Create Producer Application

### Step 1: Create Console Application

```bash
# Create directory
mkdir EventHubsProducer
cd EventHubsProducer

# Create console app
dotnet new console

# Add Event Hubs package
dotnet add package Azure.Messaging.EventHubs

# Restore packages
dotnet restore
```

### Step 2: Producer Code

Edit `Program.cs`:

```csharp
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Producer;
using System.Text;
using System.Text.Json;

// Configuration
string connectionString = "<YOUR-CONNECTION-STRING>";
string eventHubName = "telemetry";

Console.WriteLine("Event Hubs Producer");
Console.WriteLine("===================");
Console.WriteLine();

// Create producer client
await using var producer = new EventHubProducerClient(connectionString, eventHubName);

Console.WriteLine($"Connected to Event Hub: {eventHubName}");
Console.WriteLine();

// Get Event Hub properties
EventHubProperties properties = await producer.GetEventHubPropertiesAsync();
Console.WriteLine($"Event Hub: {properties.Name}");
Console.WriteLine($"Created: {properties.CreatedOn}");
Console.WriteLine($"Partitions: {properties.PartitionIds.Length}");
Console.WriteLine($"Partition IDs: {string.Join(", ", properties.PartitionIds)}");
Console.WriteLine();

// Send events
int eventCount = 100;
int eventsSent = 0;

Console.WriteLine($"Sending {eventCount} events...");
Console.WriteLine();

// Create batch
using EventDataBatch eventBatch = await producer.CreateBatchAsync();

for (int i = 1; i <= eventCount; i++)
{
    // Create telemetry data
    var telemetry = new
    {
        DeviceId = $"device-{i % 10:D3}",  // 10 devices (device-000 to device-009)
        Temperature = 20 + (i % 30),        // Temperature: 20-50°C
        Humidity = 30 + (i % 40),           // Humidity: 30-70%
        Timestamp = DateTime.UtcNow
    };
    
    string jsonData = JsonSerializer.Serialize(telemetry);
    
    // Create event data
    var eventData = new EventData(Encoding.UTF8.GetBytes(jsonData));
    
    // Add application properties
    eventData.Properties["DeviceId"] = telemetry.DeviceId;
    eventData.Properties["MessageType"] = "Telemetry";
    
    // Set partition key (all events from same device go to same partition)
    var batchOptions = new CreateBatchOptions
    {
        PartitionKey = telemetry.DeviceId
    };
    
    // Try to add to batch
    if (!eventBatch.TryAdd(eventData))
    {
        // Batch full, send it
        await producer.SendAsync(eventBatch);
        eventsSent += eventBatch.Count;
        Console.WriteLine($"Sent batch: {eventBatch.Count} events (Total: {eventsSent}/{eventCount})");
        
        // Create new batch and add current event
        eventBatch = await producer.CreateBatchAsync(batchOptions);
        
        if (!eventBatch.TryAdd(eventData))
        {
            throw new Exception($"Event {i} is too large for an empty batch");
        }
    }
}

// Send remaining events
if (eventBatch.Count > 0)
{
    await producer.SendAsync(eventBatch);
    eventsSent += eventBatch.Count;
    Console.WriteLine($"Sent final batch: {eventBatch.Count} events (Total: {eventsSent}/{eventCount})");
}

Console.WriteLine();
Console.WriteLine($"✓ Successfully sent {eventsSent} events!");
Console.WriteLine();
Console.WriteLine("Press any key to exit...");
Console.ReadKey();
```

### Step 3: Run Producer

```bash
# Update connection string in Program.cs
# Replace <YOUR-CONNECTION-STRING> with actual value

# Run application
dotnet run

# Expected output:
# Event Hubs Producer
# ===================
#
# Connected to Event Hub: telemetry
#
# Event Hub: telemetry
# Created: 2024-01-15 10:00:00Z
# Partitions: 4
# Partition IDs: 0, 1, 2, 3
#
# Sending 100 events...
#
# Sent batch: 50 events (Total: 50/100)
# Sent final batch: 50 events (Total: 100/100)
#
# ✓ Successfully sent 100 events!
```

---

## Part 3: Create Consumer Application

### Step 1: Create Console Application

```bash
# Go back to parent directory
cd ..

# Create directory
mkdir EventHubsConsumer
cd EventHubsConsumer

# Create console app
dotnet new console

# Add Event Hubs package
dotnet add package Azure.Messaging.EventHubs

# Restore packages
dotnet restore
```

### Step 2: Consumer Code

Edit `Program.cs`:

```csharp
using Azure.Messaging.EventHubs.Consumer;
using System.Text;
using System.Text.Json;

// Configuration
string connectionString = "<YOUR-CONNECTION-STRING>";
string eventHubName = "telemetry";
string consumerGroup = EventHubConsumerClient.DefaultConsumerGroupName;

Console.WriteLine("Event Hubs Consumer");
Console.WriteLine("===================");
Console.WriteLine();

// Create consumer client
await using var consumer = new EventHubConsumerClient(
    consumerGroup,
    connectionString,
    eventHubName
);

Console.WriteLine($"Connected to Event Hub: {eventHubName}");
Console.WriteLine($"Consumer Group: {consumerGroup}");
Console.WriteLine();

// Configure read options
var readOptions = new ReadEventOptions
{
    MaximumWaitTime = TimeSpan.FromSeconds(5)
};

Console.WriteLine("Reading events (Ctrl+C to stop)...");
Console.WriteLine();

int eventCount = 0;

// Create cancellation token (stop after 30 seconds for demo)
using var cancellationSource = new CancellationTokenSource();
cancellationSource.CancelAfter(TimeSpan.FromSeconds(30));

try
{
    // Read events
    await foreach (PartitionEvent partitionEvent in consumer.ReadEventsAsync(
        readOptions,
        cancellationSource.Token))
    {
        // Skip null events (timeout)
        if (partitionEvent.Data == null)
        {
            continue;
        }
        
        eventCount++;
        
        // Parse event data
        string body = Encoding.UTF8.GetString(partitionEvent.Data.EventBody.ToArray());
        
        // Display event information
        Console.WriteLine($"Event #{eventCount}");
        Console.WriteLine($"  Partition: {partitionEvent.Partition.PartitionId}");
        Console.WriteLine($"  Offset: {partitionEvent.Data.Offset}");
        Console.WriteLine($"  Sequence: {partitionEvent.Data.SequenceNumber}");
        Console.WriteLine($"  Enqueued: {partitionEvent.Data.EnqueuedTime:yyyy-MM-dd HH:mm:ss}");
        
        // Display application properties
        if (partitionEvent.Data.Properties.Count > 0)
        {
            Console.WriteLine($"  Properties:");
            foreach (var prop in partitionEvent.Data.Properties)
            {
                Console.WriteLine($"    {prop.Key}: {prop.Value}");
            }
        }
        
        // Display body
        Console.WriteLine($"  Body: {body}");
        Console.WriteLine();
        
        // Optional: Parse JSON
        try
        {
            var telemetry = JsonSerializer.Deserialize<JsonElement>(body);
            string deviceId = telemetry.GetProperty("DeviceId").GetString();
            double temperature = telemetry.GetProperty("Temperature").GetDouble();
            double humidity = telemetry.GetProperty("Humidity").GetDouble();
            
            // Check for alerts
            if (temperature > 40)
            {
                Console.WriteLine($"  ⚠️  ALERT: High temperature detected! Device: {deviceId}, Temp: {temperature}°C");
                Console.WriteLine();
            }
        }
        catch
        {
            // Ignore JSON parsing errors
        }
    }
}
catch (TaskCanceledException)
{
    Console.WriteLine("Reading stopped (timeout reached)");
}

Console.WriteLine();
Console.WriteLine($"✓ Read {eventCount} events");
Console.WriteLine();
Console.WriteLine("Press any key to exit...");
Console.ReadKey();
```

### Step 3: Run Consumer

**Terminal 1 (Producer):**
```bash
cd EventHubsProducer
dotnet run
```

**Terminal 2 (Consumer):**
```bash
cd EventHubsConsumer
dotnet run

# Expected output:
# Event Hubs Consumer
# ===================
#
# Connected to Event Hub: telemetry
# Consumer Group: $Default
#
# Reading events (Ctrl+C to stop)...
#
# Event #1
#   Partition: 2
#   Offset: 12345
#   Sequence: 67890
#   Enqueued: 2024-01-15 10:30:00
#   Properties:
#     DeviceId: device-001
#     MessageType: Telemetry
#   Body: {"DeviceId":"device-001","Temperature":21,"Humidity":31,"Timestamp":"..."}
#
# Event #2
#   Partition: 1
#   ...
```

---

## Part 4: Create Event Processor Application

### Step 1: Create Console Application

```bash
# Go back to parent directory
cd ..

# Create directory
mkdir EventHubsProcessor
cd EventHubsProcessor

# Create console app
dotnet new console

# Add Event Hubs packages
dotnet add package Azure.Messaging.EventHubs
dotnet add package Azure.Messaging.EventHubs.Processor
dotnet add package Azure.Storage.Blobs

# Restore packages
dotnet restore
```

### Step 2: Create Storage Account

```bash
# Storage account name (must be globally unique)
STORAGE_ACCOUNT="stehlab$RANDOM"

# Create storage account
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS

# Get connection string
STORAGE_CONNECTION_STRING=$(az storage account show-connection-string \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --query connectionString \
  --output tsv)

echo "Storage Connection String: $STORAGE_CONNECTION_STRING"

# Create container for checkpoints
az storage container create \
  --name checkpoints \
  --account-name $STORAGE_ACCOUNT \
  --connection-string "$STORAGE_CONNECTION_STRING"
```

### Step 3: Event Processor Code

Edit `Program.cs`:

```csharp
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Consumer;
using Azure.Messaging.EventHubs.Processor;
using Azure.Storage.Blobs;
using System.Text;
using System.Text.Json;

// Configuration
string eventHubsConnectionString = "<YOUR-EVENTHUBS-CONNECTION-STRING>";
string eventHubName = "telemetry";
string consumerGroup = "processor";  // Use custom consumer group

string blobStorageConnectionString = "<YOUR-STORAGE-CONNECTION-STRING>";
string blobContainerName = "checkpoints";

Console.WriteLine("Event Hubs Processor");
Console.WriteLine("====================");
Console.WriteLine();

// Create blob container client for checkpoint store
BlobContainerClient storageClient = new BlobContainerClient(
    blobStorageConnectionString,
    blobContainerName
);

// Create container if it doesn't exist
await storageClient.CreateIfNotExistsAsync();

Console.WriteLine($"Checkpoint store: {blobContainerName}");
Console.WriteLine();

// Create event processor client
EventProcessorClient processor = new EventProcessorClient(
    storageClient,
    consumerGroup,
    eventHubsConnectionString,
    eventHubName
);

// Track statistics
int eventCount = 0;
int checkpointCount = 0;
DateTime startTime = DateTime.UtcNow;

// Event handler
processor.ProcessEventAsync += async (ProcessEventArgs args) =>
{
    try
    {
        // Skip null events
        if (args.Data == null)
        {
            return;
        }
        
        // Parse event data
        string body = Encoding.UTF8.GetString(args.Data.EventBody.ToArray());
        
        // Increment counter (thread-safe)
        int currentCount = Interlocked.Increment(ref eventCount);
        
        // Display event
        Console.WriteLine($"[{args.Partition.PartitionId}] Event #{currentCount}: {body}");
        
        // Process event (your business logic)
        await ProcessEventAsync(body);
        
        // Checkpoint every 10 events (balance fault tolerance vs performance)
        if (currentCount % 10 == 0)
        {
            await args.UpdateCheckpointAsync();
            Interlocked.Increment(ref checkpointCount);
            Console.WriteLine($"  ✓ Checkpointed at event {currentCount}");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Error processing event: {ex.Message}");
        // Don't checkpoint on error - event will be reprocessed
    }
};

// Error handler
processor.ProcessErrorAsync += (ProcessErrorEventArgs args) =>
{
    Console.WriteLine($"Error in partition {args.PartitionId}: {args.Exception.Message}");
    return Task.CompletedTask;
};

// Start processing
Console.WriteLine("Starting event processor...");
await processor.StartProcessingAsync();
Console.WriteLine("✓ Event processor started");
Console.WriteLine();

Console.WriteLine("Processing events (press Enter to stop)...");
Console.WriteLine();

// Wait for user input
Console.ReadLine();

// Display statistics
TimeSpan duration = DateTime.UtcNow - startTime;
Console.WriteLine();
Console.WriteLine("Statistics:");
Console.WriteLine($"  Events processed: {eventCount}");
Console.WriteLine($"  Checkpoints: {checkpointCount}");
Console.WriteLine($"  Duration: {duration.TotalSeconds:F1} seconds");
Console.WriteLine($"  Throughput: {eventCount / duration.TotalSeconds:F1} events/sec");
Console.WriteLine();

// Stop processing
Console.WriteLine("Stopping event processor...");
await processor.StopProcessingAsync();
Console.WriteLine("✓ Event processor stopped");

async Task ProcessEventAsync(string eventData)
{
    // Your business logic here
    // Example: Parse JSON, save to database, send alert, etc.
    
    try
    {
        var telemetry = JsonSerializer.Deserialize<JsonElement>(eventData);
        string deviceId = telemetry.GetProperty("DeviceId").GetString();
        double temperature = telemetry.GetProperty("Temperature").GetDouble();
        
        // Check for alerts
        if (temperature > 40)
        {
            Console.WriteLine($"  ⚠️  ALERT: High temperature! Device: {deviceId}, Temp: {temperature}°C");
        }
    }
    catch
    {
        // Ignore parsing errors
    }
    
    // Simulate processing time
    await Task.Delay(10);
}
```

### Step 4: Run Event Processor

**Terminal 1 (Producer - continuous):**
```bash
cd EventHubsProducer

# Modify Program.cs to loop continuously
# Add: while (true) { ... await Task.Delay(1000); }

dotnet run
```

**Terminal 2 (Event Processor):**
```bash
cd EventHubsProcessor
dotnet run

# Expected output:
# Event Hubs Processor
# ====================
#
# Checkpoint store: checkpoints
#
# Starting event processor...
# ✓ Event processor started
#
# Processing events (press Enter to stop)...
#
# [2] Event #1: {"DeviceId":"device-001","Temperature":21,...}
# [1] Event #2: {"DeviceId":"device-002","Temperature":22,...}
# [0] Event #3: {"DeviceId":"device-003","Temperature":43,...}
#   ⚠️  ALERT: High temperature! Device: device-003, Temp: 43°C
# ...
# [2] Event #10: {"DeviceId":"device-000","Temperature":30,...}
#   ✓ Checkpointed at event 10
```

**Terminal 3 (Second Event Processor Instance - Load Balancing Test):**
```bash
cd EventHubsProcessor
dotnet run

# Observe partition rebalancing
# Instance 1 will release some partitions
# Instance 2 will claim those partitions
# Events distributed between both instances
```

---

## Часть 5: Мониторинг через Azure Portal

Мониторинг позволяет понять:

- поступают ли события
- читаются ли они consumer-приложениями
- есть ли throttling
- равномерно ли распределена нагрузка

---

## Шаг 1. Просмотр метрик

1. Перейдите в Azure Portal
2. Откройте ваш **Event Hubs namespace**
3. В разделе **Monitoring** выберите **Metrics**
4. Добавьте следующие метрики:

   - **Incoming Messages** — количество отправленных событий
   - **Outgoing Messages** — количество прочитанных событий
   - **Throttled Requests** — случаи ограничения (throttling)
   - **User Errors** — ошибки клиента

📌 Если Incoming растёт, а Outgoing — нет, значит consumer не читает события.

---

## Шаг 2. Просмотр информации об Event Hub

1. Выберите ваш Event Hub (например, `telemetry`)
2. Откройте вкладку **Overview**
3. Проверьте:

   - **Message Count** — общее количество сообщений
   - **Throughput Units** — текущая загрузка
   - **Partitions** — распределение партиций

📌 Если наблюдается throttling — возможно, нужно увеличить Throughput Units.

---

## Шаг 3. Просмотр информации по партициям

1. В разделе **Entities** выберите **Partitions**
2. Просмотрите метрики по каждой партиции:

   - Incoming messages
   - Outgoing messages
   - Active connections

📌 Неравномерная нагрузка может указывать на отсутствие partition key или на «горячую» партицию.

---

## Шаг 4. Просмотр Consumer Groups

1. В разделе **Entities** выберите **Consumer groups**
2. Проверьте доступные группы:

   - `$Default` — стандартная consumer group
   - `processor` — пользовательская группа

📌 Каждая consumer group получает собственный поток чтения.  
Это позволяет нескольким приложениям независимо обрабатывать одни и те же события.

---

## Что важно для AZ-204

- Incoming vs Outgoing помогает диагностировать проблемы
- Throttled Requests указывает на нехватку Throughput Units
- Партиции влияют на масштабируемость
- Consumer groups позволяют независимое чтение

---

## Ключевая идея

Мониторинг Event Hubs — важная часть production-систем:

- помогает выявлять узкие места
- позволяет корректно масштабировать систему
- даёт понимание распределения нагрузки

На экзамене часто проверяется понимание различий между:

- Namespace-level метриками
- Event Hub-level метриками
- Partition-level метриками
- Consumer group поведением
---

## Part 6: Test Scenarios

### Scenario 1: Partition Distribution

**Test:** Verify events distributed across partitions

```bash
# Send 100 events with partition key
cd EventHubsProducer
dotnet run

# Check Azure Portal
# Navigate to: Event Hub → Partitions
# Verify events distributed based on partition key hash
```

### Scenario 2: Load Balancing

**Test:** Multiple processor instances share load

```bash
# Terminal 1: Start first processor
cd EventHubsProcessor
dotnet run

# Terminal 2: Start second processor (in separate directory)
cd EventHubsProcessor
dotnet run

# Observe:
# - Partitions rebalanced between instances
# - Each instance processes subset of partitions
# - Total throughput increases
```

### Scenario 3: Fault Tolerance

**Test:** Processor recovery after crash

```bash
# Terminal 1: Start processor
dotnet run

# Process some events (observe checkpointing)
# Press Ctrl+C to stop (simulate crash)

# Restart processor
dotnet run

# Observe:
# - Processor resumes from last checkpoint
# - Events after checkpoint reprocessed
# - No events lost
```

### Scenario 4: Consumer Lag

**Test:** Monitor consumer lag

```bash
# Send events faster than consumption
# Terminal 1: Send 1000 events
cd EventHubsProducer
# Modify: Send 1000 events
dotnet run

# Terminal 2: Slow consumer
cd EventHubsConsumer
# Modify: Add Task.Delay(100) per event
dotnet run

# Azure Portal:
# Navigate to: Event Hub → Metrics
# Add metric: Consumer Lag
# Observe lag increasing
```

---

## Part 7: Cleanup Resources

### Delete Resource Group

```bash
# Delete all resources
az group delete \
  --name $RESOURCE_GROUP \
  --yes \
  --no-wait

echo "Resource group deletion initiated"
```

## ⚠️ Внимание

Удаление resource group приведёт к удалению **ВСЕХ** ресурсов внутри неё, включая:

- Event Hubs namespace
- Event Hub
- Storage account
- Все данные

Перед удалением убедитесь, что данные больше не нужны.

---

# Ключевые выводы

## Изученные концепции

### 1. Архитектура Event Hubs
- Namespace → Event Hub → Partitions
- Consumer groups обеспечивают независимое чтение
- Параллелизм достигается за счёт партиций

---

### 2. EventHubProducerClient
- Использование batching для повышения производительности
- Partition key для группировки связанных событий
- Application properties для передачи метаданных

---

### 3. EventHubConsumerClient
- Чтение событий из всех партиций
- Использование итеративного подхода (iterator pattern)
- Подходит для прототипирования

---

### 4. EventProcessorClient
- Автоматическая балансировка нагрузки
- Отказоустойчивость на основе checkpoint
- Production-ready масштабируемость

---

### 5. Checkpointing
- Отслеживает прогресс обработки
- Позволяет восстановиться после сбоя
- Требует Azure Blob Storage

---

# Применённые лучшие практики

✅ **Batching** — использование `CreateBatchAsync()` для эффективной отправки

✅ **Partition Key** — группировка связанных событий (например, по устройству)

✅ **Checkpointing** — разумная частота (например, каждые 10 событий)

✅ **Обработка ошибок** — try-catch без выполнения checkpoint при ошибке

✅ **Управление ресурсами** — корректное освобождение ресурсов

✅ **Балансировка нагрузки** — несколько инстансов автоматически распределяют партиции

✅ **Мониторинг** — использование метрик Azure Portal для наблюдаемости

---

## Главное для AZ-204

После выполнения этого упражнения вы понимаете:

- разницу между producer и consumer
- зачем нужен partition key
- как работает балансировка
- почему важен checkpoint
- как масштабируется обработка

Если вы уверенно объясняете каждый из этих пунктов — тема Event Hubs для AZ-204 у вас закрыта на хорошем уровне.
---

## Troubleshooting Guide

### Issue 1: Connection Failed

**Error:**
```
Azure.Messaging.EventHubs.EventHubsException: The Azure Active Directory access token has expired.
```

**Solution:**
```bash
# Refresh Azure login
az login

# Or check connection string
az eventhubs namespace authorization-rule keys list \
  --name RootManageSharedAccessKey \
  --namespace-name $NAMESPACE \
  --resource-group $RESOURCE_GROUP
```

### Issue 2: Partition Not Found

**Error:**
```
The specified partition '5' does not exist.
```

**Solution:**
```bash
# Check partition count
az eventhubs eventhub show \
  --name $EVENTHUB \
  --namespace-name $NAMESPACE \
  --resource-group $RESOURCE_GROUP \
  --query partitionCount

# Update code to use valid partition IDs (0 to partitionCount-1)
```

### Issue 3: Checkpoint Store Access Denied

**Error:**
```
Azure.RequestFailedException: This request is not authorized to perform this operation.
```

**Solution:**
```bash
# Verify storage connection string
az storage account show-connection-string \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP

# Ensure container exists
az storage container create \
  --name checkpoints \
  --account-name $STORAGE_ACCOUNT
```

### Issue 4: No Events Received

**Possible Causes:**
- Events sent to different Event Hub
- Using wrong consumer group
- Events expired (retention period)

**Solution:**
```bash
# Verify Event Hub has events
az monitor metrics list \
  --resource /subscriptions/.../providers/Microsoft.EventHub/namespaces/$NAMESPACE/eventhubs/$EVENTHUB \
  --metric "IncomingMessages" \
  --start-time "2024-01-15T00:00:00Z"

# Check retention period
az eventhubs eventhub show \
  --name $EVENTHUB \
  --namespace-name $NAMESPACE \
  --resource-group $RESOURCE_GROUP \
  --query messageRetentionInDays
```

---

# Советы к экзамену AZ-204

## Ключевые концепции из упражнения

### 1. Настройка Event Hubs
- Создать **Namespace** (Standard SKU — поддержка consumer groups)
- Создать **Event Hub** с нужным количеством партиций
- Получить connection string или настроить Azure AD

---

### 2. Producer-паттерн
- Использовать `EventHubProducerClient`
- Всегда применять batching (`CreateBatchAsync()`)
- Указывать partition key для связанных событий

---

### 3. Consumer-паттерн
- `EventHubConsumerClient` — только для прототипирования
- `EventProcessorClient` — production (требует Blob Storage)

---

### 4. Checkpointing
- Вызывать `UpdateCheckpointAsync()` периодически
- Не выполнять checkpoint при ошибке
- Балансировать частоту (отказоустойчивость vs производительность)

---

### 5. Балансировка нагрузки
- Несколько экземпляров EventProcessorClient
- Автоматическое распределение партиций
- Горизонтальное масштабирование

---

# Что обязательно помнить

- **Namespace** — контейнер для Event Hubs (аналог SQL Server)
- **Event Hub** — append-only лог (аналог таблицы)
- **Partitions** — упорядоченные последовательности (нельзя изменить после создания)
- **Consumer Group** — независимое представление Event Hub
- **Checkpoint Store** — Azure Blob Storage (обязателен для EventProcessorClient)
- **Batching**: `CreateBatchAsync()` → `TryAdd()` → `SendAsync()`
- **Partition Key** — хеш-распределение с сохранением порядка
- **EventProcessorClient** — production-решение

---

# Типовые экзаменационные сценарии

### Сценарий 1: Система приёма телеметрии

✔ Использовать Event Hubs  
✔ Producer с batching  
✔ Partition key для группировки по устройствам

---

### Сценарий 2: Масштабирование обработки

✔ Использовать EventProcessorClient  
✔ Развернуть несколько инстансов  
✔ Настроить checkpoint store

---

### Сценарий 3: Отслеживание прогресса

✔ Использовать checkpointing  
✔ Blob Storage как checkpoint store  
✔ Восстановление после сбоя

---

# Итог упражнения

В ходе задания вы:

✅ Создали Event Hubs namespace и Event Hub  
✅ Реализовали producer с batching  
✅ Реализовали consumer с iterator-подходом  
✅ Настроили EventProcessorClient с балансировкой  
✅ Протестировали отказоустойчивость  
✅ Проанализировали метрики в Azure Portal

---

# Production Checklist

- ✅ Использовать EventProcessorClient (не EventHubConsumerClient)
- ✅ Настроить checkpoint store (Azure Blob Storage)
- ✅ Реализовать корректную обработку ошибок
- ✅ Использовать batching
- ✅ Настроить разумную частоту checkpoint
- ✅ Мониторить lag и throughput
- ✅ Использовать Managed Identity вместо connection string
- ✅ Настроить auto-scaling для consumer-приложений

---

## Финальная мысль для AZ-204

Если в вопросе фигурируют:

- масштабирование
- отказоустойчивость
- checkpoint
- балансировка
- production-нагрузка

— правильный ответ почти всегда связан с **EventProcessorClient + Azure Blob Storage + batching + partition key**.

Понимание этих механизмов — ключ к успешной сдаче темы Event Hubs на AZ-204.