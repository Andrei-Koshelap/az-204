# Azure Cosmos DB Change Feed

## Ключевые понятия (Key Concepts)

- **Persistent log** — постоянный журнал всех изменений контейнера
- **Push model** — обработка через Azure Functions или Change Feed Processor
- **Pull model** — ручная обработка через SDK
- **Time-ordered** — изменения упорядочены по времени внутри партиции

---

# Что такое Change Feed?

**Постоянный, упорядоченный журнал изменений** контейнера Azure Cosmos DB.

- **Фиксирует**: операции создания и обновления  
  (удаления по умолчанию не включаются)
- **Порядок**: гарантирован внутри одного partition key
- **Хранение**: зависит от TTL контейнера
- **Модели обработки**: Push (автоматическая) или Pull (ручная)
- **Сценарии использования**: real-time обработка, синхронизация данных, event sourcing

---

## Основные характеристики

| Свойство | Описание |
|-----------|------------|
| **Область действия** | Уровень контейнера (отслеживает один контейнер) |
| **Операции** | Insert и Update (delete — при использовании соответствующего режима) |
| **Порядок** | Гарантирован внутри одного partition key |
| **Между партициями** | Порядок не гарантируется |
| **Retention** | Зависит от TTL контейнера или checkpoint-механизма |
| **Точка старта** | С начала, с текущего момента или с конкретного времени |
| **Обработка** | Реальное время или batch |

---

## Что важно помнить

- Change Feed не блокирует основную нагрузку.
- Используется для реактивной архитектуры.
- Часто применяется в микросервисах.
- Поддерживает горизонтальное масштабирование.

---

## Экзаменационный акцент (AZ-204)

- Отслеживает изменения на уровне контейнера.
- Порядок гарантируется только внутри partition key.
- По умолчанию не включает delete.
- Может запускаться с начала или «сейчас».
- Используется для real-time обработки и синхронизации.


### Change Feed Modes

```csharp
// Latest version mode (default) - captures creates and updates
ChangeFeedMode.LatestVersion

// All versions and deletes mode - captures all changes including deletes
ChangeFeedMode.AllVersionsAndDeletes
```

## Push Model

### Option 1: Azure Functions Trigger

**Easiest way** - Automatic processing with Azure Functions:

```csharp
using Microsoft.Azure.Documents;
using Microsoft.Azure.WebJobs;
using Microsoft.Extensions.Logging;
using System.Collections.Generic;

public static class ChangeFeedFunction
{
    [FunctionName("ProcessChangeFeed")]
    public static void Run(
        [CosmosDBTrigger(
            databaseName: "myDatabase",
            collectionName: "myContainer",
            ConnectionStringSetting = "CosmosDBConnection",
            LeaseCollectionName = "leases",
            CreateLeaseCollectionIfNotExists = true)]
        IReadOnlyList<Document> documents,
        ILogger log)
    {
        if (documents != null && documents.Count > 0)
        {
            log.LogInformation($"Processing {documents.Count} documents");
            
            foreach (var doc in documents)
            {
                log.LogInformation($"Document ID: {doc.Id}");
                
                // Process each changed document
                ProcessDocument(doc);
            }
        }
    }
    
    private static void ProcessDocument(Document doc)
    {
        // Your processing logic
        // - Update search index
        // - Send notification
        // - Trigger workflow
        // - Synchronize data
    }
}
```

**Function configuration** (local.settings.json):

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet",
    "CosmosDBConnection": "AccountEndpoint=https://...;AccountKey=..."
  }
}
```

**Key benefits**:
- ✅ Zero infrastructure management
- ✅ Automatic scaling
- ✅ Built-in retry logic
- ✅ Checkpoint management handled
- ✅ Easy deployment

### Option 2: Change Feed Processor

**Programmatic processing** with full control:

#### Four Components

1. **Monitored container** - Source of changes
2. **Lease container** - Stores processing state (checkpoints)
3. **Compute instance** - Host that runs processor
4. **Delegate** - Your code that processes changes

#### Basic Implementation

```csharp
using Microsoft.Azure.Cosmos;
using System;
using System.Collections.Generic;
using System.Threading;
using System.Threading.Tasks;

public class ChangeFeedProcessorExample
{
    private CosmosClient cosmosClient;
    private Container monitoredContainer;
    private Container leaseContainer;
    private ChangeFeedProcessor changeFeedProcessor;
    
    public async Task StartAsync()
    {
        // Initialize Cosmos Client
        cosmosClient = new CosmosClient(
            "AccountEndpoint=https://...;AccountKey=..."
        );
        
        // Get containers
        monitoredContainer = cosmosClient
            .GetContainer("myDatabase", "myContainer");
        
        leaseContainer = cosmosClient
            .GetContainer("myDatabase", "leases");
        
        // Build change feed processor
        changeFeedProcessor = monitoredContainer
            .GetChangeFeedProcessorBuilder<Product>(
                processorName: "productProcessor",
                onChangesDelegate: HandleChangesAsync)
            .WithInstanceName("consoleApp-instance1")
            .WithLeaseContainer(leaseContainer)
            .WithStartTime(DateTime.UtcNow.AddHours(-1))  // Start from 1 hour ago
            .Build();
        
        // Start processing
        await changeFeedProcessor.StartAsync();
        
        Console.WriteLine("Change feed processor started. Press any key to stop...");
        Console.ReadKey();
        
        // Stop processing
        await changeFeedProcessor.StopAsync();
    }
    
    // Delegate that processes changes
    static async Task HandleChangesAsync(
        ChangeFeedProcessorContext context,
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken)
    {
        Console.WriteLine($"Processing {changes.Count} changes...");
        
        foreach (var product in changes)
        {
            Console.WriteLine($"Product changed: {product.id}");
            Console.WriteLine($"  Name: {product.name}");
            Console.WriteLine($"  Price: {product.price}");
            
            // Process the change
            await ProcessProductChangeAsync(product);
        }
    }
    
    static async Task ProcessProductChangeAsync(Product product)
    {
        // Your business logic
        // - Update search index
        // - Send to event hub
        // - Update cache
        // - Trigger notification
        
        await Task.CompletedTask;
    }
}

public class Product
{
    public string id { get; set; }
    public string name { get; set; }
    public decimal price { get; set; }
    public string category { get; set; }
}
```

#### Configuration Options

```csharp
changeFeedProcessor = container
    .GetChangeFeedProcessorBuilder<MyDocument>(
        processorName: "myProcessor",
        onChangesDelegate: HandleChangesAsync)
    
    // Lease configuration
    .WithInstanceName("instance-1")                    // Unique instance identifier
    .WithLeaseContainer(leaseContainer)                // Container for checkpoints
    
    // Starting position
    .WithStartTime(DateTime.UtcNow.AddDays(-1))       // Start from 1 day ago
    // OR
    .WithStartTime(DateTime.MinValue)                  // Start from beginning
    
    // Polling configuration
    .WithPollInterval(TimeSpan.FromSeconds(5))         // Check for changes every 5s
    
    // Batch size
    .WithMaxItems(100)                                 // Max items per batch
    
    // Error handling
    .WithErrorNotification((leaseToken, exception) =>
    {
        Console.WriteLine($"Error on lease {leaseToken}: {exception.Message}");
        return Task.CompletedTask;
    })
    
    .Build();
```

#### Multiple Processors (Scale Out)

```csharp
// Instance 1
var processor1 = container
    .GetChangeFeedProcessorBuilder<Product>("productProcessor", HandleChangesAsync)
    .WithInstanceName("instance-1")  // Unique name
    .WithLeaseContainer(leaseContainer)
    .Build();

// Instance 2
var processor2 = container
    .GetChangeFeedProcessorBuilder<Product>("productProcessor", HandleChangesAsync)
    .WithInstanceName("instance-2")  // Unique name
    .WithLeaseContainer(leaseContainer)
    .Build();

// Both process different partitions automatically
await processor1.StartAsync();
await processor2.StartAsync();
```

### Lease Container

**Stores processing state** for each partition:

```csharp
// Create lease container
Database database = cosmosClient.GetDatabase("myDatabase");

ContainerProperties leaseContainerProperties = new ContainerProperties
{
    Id = "leases",
    PartitionKeyPath = "/id"
};

Container leaseContainer = await database.CreateContainerIfNotExistsAsync(
    leaseContainerProperties,
    throughput: 400  // Manual throughput (can be low)
);
```

**Lease document structure**:

```json
{
  "id": "myContainer.lease.0",
  "LeaseToken": "0",
  "Owner": "instance-1",
  "ContinuationToken": "\"8400022\"",
  "Timestamp": "2024-01-15T10:30:00Z"
}
```

## Lease и масштабирование в Change Feed

### Ключевые моменты

- Один lease-документ на каждую партицию
- Отслеживает, какой экземпляр обработчика владеет какой партицией
- Хранит continuation token (checkpoint)
- Обеспечивает автоматическое перераспределение партиций (rebalancing)

---

## Что это означает

- Для каждой физической партиции создаётся отдельный lease-документ.
- Lease хранится в отдельном lease-контейнере.
- Continuation token позволяет продолжить обработку с последней позиции.
- При добавлении или удалении экземпляров обработчика:
    - Партиции автоматически перераспределяются.
    - Нагрузка балансируется между инстансами.

---

## Почему это важно

- Поддерживается горизонтальное масштабирование.
- Обеспечивается fault tolerance.
- Не требуется ручное управление распределением партиций.
- Обработка может возобновиться после сбоя без потери данных.

---

## Экзаменационный акцент (AZ-204)

- Lease container обязателен для Change Feed Processor.
- Один lease на партицию.
- Continuation token хранится в lease-документе.
- Поддерживается автоматический rebalancing.


## Pull Model

### Manual Processing with SDK

**Full control** over when and how to process changes:

```csharp
using Microsoft.Azure.Cosmos;
using System;
using System.Collections.Generic;
using System.Threading.Tasks;

public class PullModelExample
{
    public async Task ProcessChangesManuallyAsync()
    {
        CosmosClient client = new CosmosClient(
            "AccountEndpoint=https://...;AccountKey=..."
        );
        
        Container container = client.GetContainer("myDatabase", "myContainer");
        
        // Option 1: Process all partitions
        await ProcessAllPartitionsAsync(container);
        
        // Option 2: Process specific partition
        await ProcessSinglePartitionAsync(container, "electronics");
    }
    
    async Task ProcessAllPartitionsAsync(Container container)
    {
        // Get change feed iterator for all partitions
        FeedIterator<Product> iterator = container
            .GetChangeFeedIterator<Product>(
                ChangeFeedStartFrom.Beginning(),
                ChangeFeedMode.LatestVersion
            );
        
        while (iterator.HasMoreResults)
        {
            FeedResponse<Product> response = await iterator.ReadNextAsync();
            
            // Check if there are changes
            if (response.StatusCode == System.Net.HttpStatusCode.NotModified)
            {
                Console.WriteLine("No new changes. Waiting...");
                await Task.Delay(TimeSpan.FromSeconds(5));
                continue;
            }
            
            // Process changes
            Console.WriteLine($"Processing {response.Count} changes");
            
            foreach (var product in response)
            {
                Console.WriteLine($"Changed: {product.id} - {product.name}");
                await ProcessChangeAsync(product);
            }
            
            // Save continuation token for resuming later
            string continuationToken = response.ContinuationToken;
            await SaveCheckpointAsync(continuationToken);
        }
    }
    
    async Task ProcessSinglePartitionAsync(Container container, string partitionKey)
    {
        // Get change feed iterator for specific partition
        FeedIterator<Product> iterator = container
            .GetChangeFeedIterator<Product>(
                ChangeFeedStartFrom.Now(
                    FeedRange.FromPartitionKey(new PartitionKey(partitionKey))
                ),
                ChangeFeedMode.LatestVersion
            );
        
        while (iterator.HasMoreResults)
        {
            FeedResponse<Product> response = await iterator.ReadNextAsync();
            
            if (response.StatusCode != System.Net.HttpStatusCode.NotModified)
            {
                foreach (var product in response)
                {
                    Console.WriteLine($"Product in {partitionKey}: {product.id}");
                }
            }
            else
            {
                await Task.Delay(TimeSpan.FromSeconds(5));
            }
        }
    }
    
    async Task ProcessChangeAsync(Product product)
    {
        // Your processing logic
        await Task.CompletedTask;
    }
    
    async Task SaveCheckpointAsync(string continuationToken)
    {
        // Save to persistent storage (database, file, etc.)
        // For resuming processing later
        await Task.CompletedTask;
    }
}
```

### Starting Points

```csharp
// 1. From beginning (all historical changes)
ChangeFeedStartFrom.Beginning()

// 2. From now (only new changes)
ChangeFeedStartFrom.Now()

// 3. From specific time
ChangeFeedStartFrom.Time(DateTime.UtcNow.AddDays(-7))

// 4. Resume from checkpoint (continuation token)
ChangeFeedStartFrom.ContinuationToken(savedToken)

// 5. Specific partition from beginning
ChangeFeedStartFrom.Beginning(
    FeedRange.FromPartitionKey(new PartitionKey("electronics"))
)
```

### Pull Model with Checkpointing

```csharp
public class ManualCheckpointingExample
{
    private string checkpointFile = "checkpoint.txt";
    
    public async Task ProcessWithCheckpointAsync()
    {
        CosmosClient client = new CosmosClient("...");
        Container container = client.GetContainer("myDatabase", "myContainer");
        
        // Load last checkpoint
        string continuationToken = await LoadCheckpointAsync();
        
        FeedIterator<Product> iterator;
        
        if (string.IsNullOrEmpty(continuationToken))
        {
            // Start from beginning
            iterator = container.GetChangeFeedIterator<Product>(
                ChangeFeedStartFrom.Beginning(),
                ChangeFeedMode.LatestVersion
            );
        }
        else
        {
            // Resume from checkpoint
            iterator = container.GetChangeFeedIterator<Product>(
                ChangeFeedStartFrom.ContinuationToken(continuationToken),
                ChangeFeedMode.LatestVersion
            );
        }
        
        while (iterator.HasMoreResults)
        {
            try
            {
                FeedResponse<Product> response = await iterator.ReadNextAsync();
                
                if (response.StatusCode == System.Net.HttpStatusCode.NotModified)
                {
                    await Task.Delay(TimeSpan.FromSeconds(5));
                    continue;
                }
                
                // Process changes
                foreach (var product in response)
                {
                    await ProcessChangeAsync(product);
                }
                
                // Save checkpoint after successful processing
                await SaveCheckpointAsync(response.ContinuationToken);
                
                Console.WriteLine($"Processed {response.Count} items. RU: {response.RequestCharge}");
            }
            catch (Exception ex)
            {
                Console.WriteLine($"Error processing changes: {ex.Message}");
                // Retry or handle error
            }
        }
    }
    
    async Task<string> LoadCheckpointAsync()
    {
        if (File.Exists(checkpointFile))
        {
            return await File.ReadAllTextAsync(checkpointFile);
        }
        return null;
    }
    
    async Task SaveCheckpointAsync(string token)
    {
        await File.WriteAllTextAsync(checkpointFile, token);
    }
    
    async Task ProcessChangeAsync(Product product)
    {
        // Your processing logic
        await Task.CompletedTask;
    }
}
```

## Change Feed Modes

### Latest Version Mode (Default)

**Captures creates and updates**:

```csharp
FeedIterator<Product> iterator = container
    .GetChangeFeedIterator<Product>(
        ChangeFeedStartFrom.Beginning(),
        ChangeFeedMode.LatestVersion  // Default mode
    );

// Returns latest version of each changed document
// Does NOT capture deletes
```

### All Versions and Deletes Mode

**Captures all operations including deletes**:

```csharp
FeedIterator<ChangeFeedItem<Product>> iterator = container
    .GetChangeFeedIterator<ChangeFeedItem<Product>>(
        ChangeFeedStartFrom.Beginning(),
        ChangeFeedMode.AllVersionsAndDeletes  // Includes deletes
    );

while (iterator.HasMoreResults)
{
    FeedResponse<ChangeFeedItem<Product>> response = await iterator.ReadNextAsync();
    
    foreach (var item in response)
    {
        if (item.Metadata.OperationType == ChangeFeedOperationType.Create)
        {
            Console.WriteLine($"Created: {item.Current.id}");
        }
        else if (item.Metadata.OperationType == ChangeFeedOperationType.Replace)
        {
            Console.WriteLine($"Updated: {item.Current.id}");
            // item.Previous contains previous version
        }
        else if (item.Metadata.OperationType == ChangeFeedOperationType.Delete)
        {
            Console.WriteLine($"Deleted: {item.Previous.id}");
            // item.Current is null for deletes
        }
    }
}
```

## Common Use Cases

### 1. Real-Time Search Index Updates

```csharp
static async Task HandleChangesAsync(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<Product> changes,
    CancellationToken cancellationToken)
{
    var searchClient = new SearchClient("...");
    
    foreach (var product in changes)
    {
        // Update search index in real-time
        await searchClient.IndexDocumentAsync(new
        {
            id = product.id,
            name = product.name,
            description = product.description,
            category = product.category,
            price = product.price
        });
    }
}
```

### 2. Data Synchronization

```csharp
static async Task SyncToSqlDatabase(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<Order> changes,
    CancellationToken cancellationToken)
{
    using var connection = new SqlConnection("...");
    await connection.OpenAsync();
    
    foreach (var order in changes)
    {
        // Sync to SQL database
        var command = new SqlCommand(
            "MERGE INTO Orders ...",
            connection
        );
        
        await command.ExecuteNonQueryAsync();
    }
}
```

### 3. Event-Driven Workflows

```csharp
static async Task TriggerWorkflow(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<Order> changes,
    CancellationToken cancellationToken)
{
    var eventHubClient = new EventHubProducerClient("...");
    
    foreach (var order in changes)
    {
        if (order.status == "paid")
        {
            // Trigger fulfillment workflow
            await eventHubClient.SendAsync(new EventData(
                JsonSerializer.Serialize(new
                {
                    orderId = order.id,
                    action = "fulfill"
                })
            ));
        }
    }
}
```

### 4. Cache Invalidation

```csharp
static async Task InvalidateCache(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<Product> changes,
    CancellationToken cancellationToken)
{
    var cache = ConnectionMultiplexer.Connect("redis-connection");
    var db = cache.GetDatabase();
    
    foreach (var product in changes)
    {
        // Invalidate cache entry
        await db.KeyDeleteAsync($"product:{product.id}");
        await db.KeyDeleteAsync($"category:{product.category}");
    }
}
```

### 5. Analytics and Reporting

```csharp
static async Task UpdateAnalytics(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<Sale> changes,
    CancellationToken cancellationToken)
{
    var analyticsClient = new AnalyticsClient("...");
    
    foreach (var sale in changes)
    {
        // Update real-time analytics
        await analyticsClient.TrackEventAsync("sale", new Dictionary<string, string>
        {
            ["productId"] = sale.productId,
            ["amount"] = sale.amount.ToString(),
            ["category"] = sale.category
        });
    }
}
```

# Push vs Pull Model — Сравнение моделей обработки Change Feed

| Аспект | Push Model (Processor / Azure Functions) | Pull Model (Manual) |
|--------|--------------------------------------------|----------------------|
| **Простота использования** | Легко, автоматизировано | Более сложная реализация |
| **Контроль** | Меньше контроля | Полный контроль |
| **Checkpointing** | Автоматический | Ручной |
| **Масштабирование** | Автоматическое | Ручное |
| **Лучше всего подходит для** | Real-time обработки | Batch-обработки, кастомной логики |
| **Инфраструктура** | Управляемая | Самостоятельное управление |
| **Обработка ошибок** | Встроенные retry | Реализуется вручную |
| **Точка старта** | Настраиваемая | Полная гибкость |

---

## Когда выбирать Push Model

- Реактивная архитектура
- Интеграция с Azure Functions
- Минимальная инфраструктурная логика
- Auto-scaling без ручной настройки
- Реальное время (event-driven processing)

---

## Когда выбирать Pull Model

- Нужна тонкая настройка обработки
- Batch-обработка больших объёмов
- Полный контроль над checkpoint
- Специальная логика retry и обработки ошибок
- Интеграция с нестандартной инфраструктурой

---

## Экзаменационный акцент (AZ-204)

- «Минимум кода, автоматическая обработка» → Push Model
- «Полный контроль, ручная логика» → Pull Model
- Push → автоматический checkpoint и scaling
- Pull → разработчик управляет continuation token


## Error Handling

### In Change Feed Processor

```csharp
changeFeedProcessor = container
    .GetChangeFeedProcessorBuilder<Product>("processor", HandleChangesAsync)
    .WithInstanceName("instance-1")
    .WithLeaseContainer(leaseContainer)
    .WithErrorNotification(async (leaseToken, exception) =>
    {
        // Log error
        Console.WriteLine($"Error on partition {leaseToken}");
        Console.WriteLine($"Exception: {exception.Message}");
        
        // Custom error handling
        // - Send alert
        // - Log to monitoring system
        // - Implement retry logic
        
        await Task.CompletedTask;
    })
    .Build();
```

### In Delegate

```csharp
static async Task HandleChangesAsync(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<Product> changes,
    CancellationToken cancellationToken)
{
    foreach (var product in changes)
    {
        try
        {
            await ProcessProductAsync(product);
        }
        catch (Exception ex)
        {
            // Handle error for individual item
            Console.WriteLine($"Error processing {product.id}: {ex.Message}");
            
            // Options:
            // 1. Continue to next item
            // 2. Save to dead-letter queue
            // 3. Retry with backoff
            // 4. Throw to mark entire batch as failed
        }
    }
}
```

### In Pull Model

```csharp
while (iterator.HasMoreResults)
{
    try
    {
        FeedResponse<Product> response = await iterator.ReadNextAsync();
        
        // Process changes
        foreach (var product in response)
        {
            await ProcessAsync(product);
        }
        
        // Save checkpoint
        await SaveCheckpointAsync(response.ContinuationToken);
    }
    catch (CosmosException ex) when (ex.StatusCode == System.Net.HttpStatusCode.TooManyRequests)
    {
        // Rate limited - wait and retry
        Console.WriteLine("Rate limited. Waiting...");
        await Task.Delay(TimeSpan.FromSeconds(ex.RetryAfter?.TotalSeconds ?? 5));
    }
    catch (Exception ex)
    {
        // Other errors
        Console.WriteLine($"Error: {ex.Message}");
        // Implement retry logic or dead-letter handling
    }
}
```

## Performance Considerations

### 1. Batch Size

```csharp
// Larger batches = fewer round trips but more memory
changeFeedProcessor = container
    .GetChangeFeedProcessorBuilder<Product>("processor", HandleChangesAsync)
    .WithMaxItems(1000)  // Process up to 1000 items per batch
    .Build();
```

### 2. Polling Interval

```csharp
// Balance between latency and RU consumption
changeFeedProcessor = container
    .GetChangeFeedProcessorBuilder<Product>("processor", HandleChangesAsync)
    .WithPollInterval(TimeSpan.FromSeconds(5))  // Check every 5 seconds
    .Build();
```

### 3. Parallel Processing

```csharp
// Scale out with multiple instances
// Each instance processes different partitions automatically
var tasks = new List<Task>();

for (int i = 0; i < 5; i++)
{
    var processor = container
        .GetChangeFeedProcessorBuilder<Product>("processor", HandleChangesAsync)
        .WithInstanceName($"instance-{i}")
        .WithLeaseContainer(leaseContainer)
        .Build();
    
    tasks.Add(processor.StartAsync());
}

await Task.WhenAll(tasks);
```

## Best Practices

### 1. Use Change Feed Processor for Most Cases

```csharp
// ✅ Recommended: Change Feed Processor for real-time scenarios
// - Automatic checkpoint management
// - Built-in error handling
// - Automatic scaling
```

### 2. Idempotent Processing

```csharp
static async Task HandleChangesAsync(
    ChangeFeedProcessorContext context,
    IReadOnlyCollection<Product> changes,
    CancellationToken cancellationToken)
{
    // Make processing idempotent
    // Same change processed multiple times = same result
    
    foreach (var product in changes)
    {
        // Check if already processed
        if (await IsAlreadyProcessedAsync(product.id, product._etag))
        {
            continue;
        }
        
        await ProcessAsync(product);
        await MarkAsProcessedAsync(product.id, product._etag);
    }
}
```

### 3. Separate Lease Container

```csharp
// ✅ Good: Dedicated lease container
Container leaseContainer = database.GetContainer("leases");

// ❌ Bad: Using monitored container for leases
// Can cause performance issues
```

### 4. Handle NotModified Status

```csharp
FeedResponse<Product> response = await iterator.ReadNextAsync();

if (response.StatusCode == System.Net.HttpStatusCode.NotModified)
{
    // No new changes - wait before checking again
    await Task.Delay(TimeSpan.FromSeconds(5));
    continue;
}
```

### 5. Monitor RU Consumption

```csharp
FeedResponse<Product> response = await iterator.ReadNextAsync();

Console.WriteLine($"RU consumed: {response.RequestCharge}");

// Adjust polling frequency based on RU consumption
```

# Critical Notes — Change Feed

- 💡 **Persistent log** — постоянный, упорядоченный журнал изменений контейнера
- 🎯 **Операции** — фиксирует create и update (delete — при специальном режиме)
- ✅ **Time-ordered** — порядок гарантирован внутри одного partition key
- ⚠️ **Нет порядка между партициями** — межпартиционный порядок не гарантируется
- 🔄 **Push model** — Azure Functions или Change Feed Processor (автоматизация)
- 📊 **Pull model** — ручная обработка через SDK (полный контроль)
- 💡 **Lease container** — хранит checkpoint и информацию о владении партициями
- ✅ **Масштабирование** — несколько инстансов обрабатывают разные партиции
- ⚠️ **Точка старта** — Beginning, Now, Time или ContinuationToken
- 🔒 **Checkpointing** — сохранение continuation token для возобновления обработки
- 🎯 **Режимы Change Feed** — LatestVersion (по умолчанию) или AllVersionsAndDeletes
- 💡 **Azure Functions** — самый простой вариант через CosmosDBTrigger
- ⚠️ **Polling interval** — баланс между задержкой и потреблением RU
- ✅ **Idempotent дизайн** — обработка должна учитывать возможные дубликаты
- 🔄 **Обработка ошибок** — WithErrorNotification для ошибок процессора

---

# Exam Tips (AZ-204)

- Change Feed — постоянный, упорядоченный журнал изменений контейнера
- Фиксирует create и update (delete — при режиме AllVersionsAndDeletes)
- Порядок гарантирован только внутри partition key
- Push model — Azure Functions (CosmosDBTrigger) или Change Feed Processor
- Pull model — ручная обработка через GetChangeFeedIterator

---

## Компоненты

- Monitored container — отслеживаемый контейнер
- Lease container — хранит checkpoint и владение партициями
- Compute instance — обработчик изменений
- Delegate — метод обработки изменений

---

## Масштабирование

- Lease container управляет распределением партиций
- Несколько инстансов автоматически делят нагрузку
- Change Feed Processor масштабируется горизонтально

---

## Точки старта

- Beginning() — с начала журнала
- Now() — только новые изменения
- Time() — с указанного времени
- ContinuationToken() — с сохранённой позиции

---

## Реализация

- Azure Functions — самый простой способ с авто-масштабированием
- Change Feed Processor — через GetChangeFeedProcessorBuilder()
- Delegate — HandleChangesAsync получает batch изменений
- Pull model — через GetChangeFeedIterator<T>()

---

## Дополнительно

- StatusCode NotModified — нет новых изменений
- ContinuationToken — используется для checkpoint
- Режимы Change Feed — LatestVersion (по умолчанию), AllVersionsAndDeletes
- WithErrorNotification() — обработка ошибок процессора
- Лучший выбор в большинстве случаев — Change Feed Processor
- Обработка должна быть idempotent
- Контролируйте RU через RequestCharge и настраивайте polling

---

## Типовые сценарии

- Real-time индексирование
- Синхронизация данных
- Event-driven архитектура
- Инвалидация кэша
- Реактивные микросервисы


[Learn More](https://learn.microsoft.com/en-us/training/modules/work-with-cosmos-db/6-cosmos-db-change-feed)
