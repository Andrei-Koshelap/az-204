# Практическое задание: Отправка и получение сообщений из Service Bus Queue

## Обзор

В этом упражнении вы:

- Создадите Service Bus namespace и очередь
- Отправите сообщения в очередь
- Получите сообщения в режиме Peek Lock
- Используете message sessions для FIFO
- Разберётесь с dead-letter queue (DLQ)
- Посмотрите метрики очереди

**Оценочное время:** 30–40 минут

---

## Предварительные требования

- Подписка Azure
- Установленный Azure CLI
- .NET 6.0 или новее
- Редактор кода (VS Code, Visual Studio или аналог)

---

## Что важно (экзаменационный фокус)

- Peek Lock — рекомендуемый режим для надёжной обработки
- Sessions — механизм FIFO (порядок внутри сессии)
- DLQ — место для "ядовитых" сообщений после превышения числа попыток или при ошибках
- Мониторинг — через Azure Monitor / Metrics / Diagnostics

---

## План упражнения (без кода)

### Шаг 1: Создать Service Bus namespace
- Создайте namespace в выбранном регионе (лучше ближе к вашему приложению)
- Выберите tier (для Topics, Sessions, Transactions нужен минимум Standard)

### Шаг 2: Создать очередь
- Создайте queue в namespace
- Проверьте ключевые параметры:
   - TTL (DefaultMessageTimeToLive)
   - MaxDeliveryCount (когда попадёт в DLQ)
   - RequiresSession (если хотите FIFO через sessions)
   - Duplicate detection (если требуется)

### Шаг 3: Настроить доступ
- Выберите способ доступа:
   - Connection string (проще для учебного упражнения)
   - Managed Identity / RBAC (предпочтительно для production)
- Сохраните данные доступа в переменные окружения или конфигурацию приложения

### Шаг 4: Отправить сообщения
- Отправьте несколько сообщений в очередь
- Для сценария с FIFO:
   - назначьте одинаковый SessionId связанным сообщениям
- Для проверки дедупликации:
   - попробуйте отправить сообщения с одинаковым MessageId (при включённой duplicate detection)

### Шаг 5: Получить сообщения (Peek Lock)
- Получите сообщение и обработайте его
- После успешной обработки подтвердите завершение (Complete)
- Для проверки поведения при ошибках:
   - не подтверждайте сообщение (или "отпустите" его), чтобы оно вернулось в очередь
- Проверьте рост DeliveryCount при повторных попытках

### Шаг 6: Проверить Dead-Letter Queue (DLQ)
- Создайте условия, чтобы сообщение попало в DLQ:
   - превысить MaxDeliveryCount
   - либо явно отправить сообщение в DLQ при обработке
- Получите сообщение из DLQ и проанализируйте причину (Reason / ErrorDescription)

### Шаг 7: Мониторинг и метрики
В Azure Portal посмотрите:
- Active messages
- Dead-lettered messages
- Incoming / Outgoing requests
- Throttling (если есть)
- Server errors

---

## Итог

После выполнения упражнения вы будете понимать:

- как работает надежная обработка сообщений через Peek Lock
- как sessions обеспечивают FIFO
- как и почему сообщения попадают в DLQ
- как мониторить очередь и её состояние

Если хочешь, следующим шагом могу перевести “Exercise Steps” (начиная с создания namespace/queue) в таком же стиле, но тоже без кода.

---

## Architecture

```
Your Application                Service Bus                    Your Application
┌───────────────┐              ┌───────────┐                 ┌───────────────┐
│               │              │ Namespace │                 │               │
│   Producer    │ ──Send──────>│           │                 │   Consumer    │
│  (Sender)     │              │   Queue   │──────Receive───>│  (Receiver)   │
│               │              │           │                 │               │
└───────────────┘              └───────────┘                 └───────────────┘
                                     │
                                     v
                              ┌──────────────┐
                              │ Dead-Letter  │
                              │    Queue     │
                              └──────────────┘
```

---

## Part 1: Create Service Bus Resources

### Step 1: Create Resource Group

```bash
# Set variables
RESOURCE_GROUP="rg-servicebus-demo"
LOCATION="eastus"
NAMESPACE_NAME="sb-namespace-$RANDOM"
QUEUE_NAME="demoqueue"

# Create resource group
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION
```

### Step 2: Create Service Bus Namespace

```bash
# Create Service Bus namespace (Standard tier)
az servicebus namespace create \
  --resource-group $RESOURCE_GROUP \
  --name $NAMESPACE_NAME \
  --location $LOCATION \
  --sku Standard

# Wait for namespace to be created
echo "Namespace created: $NAMESPACE_NAME"
```

**Tier options:**
- `Basic`: Queues only, 256 KB messages
- `Standard`: Queues + topics, 256 KB messages, transactions
- `Premium`: Dedicated resources, 100 MB messages, geo-DR

### Step 3: Create Queue

```bash
# Create queue
az servicebus queue create \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name $QUEUE_NAME \
  --max-size 1024 \
  --default-message-time-to-live P14D \
  --lock-duration PT30S \
  --max-delivery-count 10 \
  --enable-duplicate-detection true \
  --duplicate-detection-history-time-window PT10M

echo "Queue created: $QUEUE_NAME"
```

**Queue properties explained:**
- `max-size`: Maximum queue size (1024 MB)
- `default-message-time-to-live`: Message TTL (14 days)
- `lock-duration`: Lock timeout for Peek Lock (30 seconds)
- `max-delivery-count`: Max retries before dead-lettering (10)
- `enable-duplicate-detection`: Prevent duplicate messages
- `duplicate-detection-history-time-window`: Deduplication window (10 minutes)

### Step 4: Get Connection String

```bash
# Get connection string
CONNECTION_STRING=$(az servicebus namespace authorization-rule keys list \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name RootManageSharedAccessKey \
  --query primaryConnectionString \
  --output tsv)

echo "Connection string: $CONNECTION_STRING"
```

**Save this connection string** - you'll need it in your code.

---

## Part 2: Send Messages to Queue

### Step 1: Create .NET Console Application

```bash
# Create new console app
dotnet new console -n ServiceBusSender
cd ServiceBusSender

# Add Service Bus package
dotnet add package Azure.Messaging.ServiceBus
```

### Step 2: Sender Code

Create `Program.cs`:

```csharp
using Azure.Messaging.ServiceBus;
using System;
using System.Threading.Tasks;

class Program
{
    // Replace with your connection string and queue name
    static string connectionString = "<YOUR_CONNECTION_STRING>";
    static string queueName = "demoqueue";

    static async Task Main(string[] args)
    {
        // Create Service Bus client
        await using ServiceBusClient client = new ServiceBusClient(connectionString);
        ServiceBusSender sender = client.CreateSender(queueName);

        try
        {
            // Send single message
            await SendSingleMessageAsync(sender);

            // Send batch of messages
            await SendBatchMessagesAsync(sender);

            // Send message with properties
            await SendMessageWithPropertiesAsync(sender);

            Console.WriteLine("All messages sent successfully!");
        }
        finally
        {
            await sender.DisposeAsync();
            await client.DisposeAsync();
        }
    }

    static async Task SendSingleMessageAsync(ServiceBusSender sender)
    {
        var message = new ServiceBusMessage("Hello from Service Bus!");
        message.MessageId = Guid.NewGuid().ToString();
        
        await sender.SendMessageAsync(message);
        Console.WriteLine($"Sent single message: {message.MessageId}");
    }

    static async Task SendBatchMessagesAsync(ServiceBusSender sender)
    {
        // Create batch
        using ServiceBusMessageBatch messageBatch = 
            await sender.CreateMessageBatchAsync();

        // Add messages to batch
        for (int i = 1; i <= 5; i++)
        {
            var message = new ServiceBusMessage($"Message {i} in batch");
            message.MessageId = $"batch-msg-{i}";

            if (!messageBatch.TryAddMessage(message))
            {
                throw new Exception($"Message {i} is too large for the batch");
            }
        }

        // Send batch
        await sender.SendMessagesAsync(messageBatch);
        Console.WriteLine($"Sent batch of {messageBatch.Count} messages");
    }

    static async Task SendMessageWithPropertiesAsync(ServiceBusSender sender)
    {
        var message = new ServiceBusMessage("Order #12345");
        message.MessageId = "order-12345";
        message.CorrelationId = "correlation-001";
        message.ContentType = "application/json";
        message.Subject = "OrderCreated";
        message.TimeToLive = TimeSpan.FromMinutes(10);

        // Add custom properties
        message.ApplicationProperties["Priority"] = "High";
        message.ApplicationProperties["Region"] = "US-West";
        message.ApplicationProperties["OrderAmount"] = 599.99;

        await sender.SendMessageAsync(message);
        Console.WriteLine($"Sent message with properties: {message.MessageId}");
    }
}
```

### Step 3: Run Sender

```bash
# Update connection string in Program.cs
# Then run
dotnet run
```

**Expected output:**
```
Sent single message: <guid>
Sent batch of 5 messages
Sent message with properties: order-12345
All messages sent successfully!
```

---

## Part 3: Receive Messages from Queue

### Step 1: Create Receiver Application

```bash
# Create new console app
cd ..
dotnet new console -n ServiceBusReceiver
cd ServiceBusReceiver

# Add Service Bus package
dotnet add package Azure.Messaging.ServiceBus
```

### Step 2: Receiver Code

Create `Program.cs`:

```csharp
using Azure.Messaging.ServiceBus;
using System;
using System.Threading.Tasks;

class Program
{
    static string connectionString = "<YOUR_CONNECTION_STRING>";
    static string queueName = "demoqueue";

    static async Task Main(string[] args)
    {
        await using ServiceBusClient client = new ServiceBusClient(connectionString);
        
        // Peek Lock mode (default, recommended)
        ServiceBusReceiver receiver = client.CreateReceiver(queueName);

        try
        {
            Console.WriteLine("Receiving messages...");
            Console.WriteLine("Press any key to stop receiving");

            // Receive messages until key pressed
            await ReceiveMessagesAsync(receiver);
        }
        finally
        {
            await receiver.DisposeAsync();
            await client.DisposeAsync();
        }
    }

    static async Task ReceiveMessagesAsync(ServiceBusReceiver receiver)
    {
        while (!Console.KeyAvailable)
        {
            // Receive up to 10 messages
            var messages = await receiver.ReceiveMessagesAsync(
                maxMessages: 10,
                maxWaitTime: TimeSpan.FromSeconds(5));

            foreach (var message in messages)
            {
                try
                {
                    Console.WriteLine($"\nReceived message:");
                    Console.WriteLine($"  MessageId: {message.MessageId}");
                    Console.WriteLine($"  Body: {message.Body}");
                    Console.WriteLine($"  DeliveryCount: {message.DeliveryCount}");
                    Console.WriteLine($"  EnqueuedTime: {message.EnqueuedTime}");

                    // Display custom properties
                    if (message.ApplicationProperties.Count > 0)
                    {
                        Console.WriteLine("  Application Properties:");
                        foreach (var prop in message.ApplicationProperties)
                        {
                            Console.WriteLine($"    {prop.Key}: {prop.Value}");
                        }
                    }

                    // Simulate processing
                    await ProcessMessageAsync(message);

                    // Complete the message (remove from queue)
                    await receiver.CompleteMessageAsync(message);
                    Console.WriteLine("  ✅ Message completed");
                }
                catch (Exception ex)
                {
                    Console.WriteLine($"  ❌ Error processing message: {ex.Message}");
                    
                    // Abandon the message (requeue for retry)
                    await receiver.AbandonMessageAsync(message);
                    Console.WriteLine("  ⚠️  Message abandoned (will be redelivered)");
                }
            }

            if (messages.Count == 0)
            {
                Console.WriteLine("No messages available. Waiting...");
            }
        }
    }

    static async Task ProcessMessageAsync(ServiceBusReceivedMessage message)
    {
        // Simulate processing time
        await Task.Delay(100);
        
        // Your business logic here
        Console.WriteLine("  Processing message...");
    }
}
```

### Step 3: Run Receiver

```bash
# Update connection string in Program.cs
# Then run
dotnet run
```

**Expected output:**
```
Receiving messages...
Press any key to stop receiving

Received message:
  MessageId: <guid>
  Body: Hello from Service Bus!
  DeliveryCount: 1
  EnqueuedTime: 2024-01-15T10:30:00Z
  Processing message...
  ✅ Message completed

Received message:
  MessageId: order-12345
  Body: Order #12345
  DeliveryCount: 1
  EnqueuedTime: 2024-01-15T10:30:01Z
  Application Properties:
    Priority: High
    Region: US-West
    OrderAmount: 599.99
  Processing message...
  ✅ Message completed

No messages available. Waiting...
```

---

## Part 4: Message Sessions (FIFO Ordering)

### Step 1: Create Session-Enabled Queue

```bash
# Create queue with sessions enabled
az servicebus queue create \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name sessionqueue \
  --enable-session true

echo "Session queue created: sessionqueue"
```

### Step 2: Send Messages with SessionId

```csharp
using Azure.Messaging.ServiceBus;
using System;
using System.Threading.Tasks;

class SessionSender
{
    static string connectionString = "<YOUR_CONNECTION_STRING>";
    static string queueName = "sessionqueue";

    static async Task Main(string[] args)
    {
        await using var client = new ServiceBusClient(connectionString);
        var sender = client.CreateSender(queueName);

        try
        {
            // Send messages for Customer A (session)
            await SendSessionMessagesAsync(sender, "customer-A", 5);

            // Send messages for Customer B (session)
            await SendSessionMessagesAsync(sender, "customer-B", 3);

            // Send messages for Customer C (session)
            await SendSessionMessagesAsync(sender, "customer-C", 4);

            Console.WriteLine("Session messages sent!");
        }
        finally
        {
            await sender.DisposeAsync();
        }
    }

    static async Task SendSessionMessagesAsync(
        ServiceBusSender sender, 
        string sessionId, 
        int count)
    {
        for (int i = 1; i <= count; i++)
        {
            var message = new ServiceBusMessage($"Order {i} for {sessionId}");
            message.SessionId = sessionId;  // FIFO guarantee per session
            message.MessageId = $"{sessionId}-order-{i}";

            await sender.SendMessageAsync(message);
            Console.WriteLine($"Sent: {message.Body} (Session: {sessionId})");
        }
    }
}
```

### Step 3: Receive Messages from Session

```csharp
using Azure.Messaging.ServiceBus;
using System;
using System.Threading.Tasks;

class SessionReceiver
{
    static string connectionString = "<YOUR_CONNECTION_STRING>";
    static string queueName = "sessionqueue";

    static async Task Main(string[] args)
    {
        await using var client = new ServiceBusClient(connectionString);

        // Accept next available session
        var sessionReceiver = await client.AcceptNextSessionAsync(queueName);

        try
        {
            Console.WriteLine($"Processing session: {sessionReceiver.SessionId}");

            // Receive messages from session (FIFO order)
            await foreach (var message in sessionReceiver.ReceiveMessagesAsync())
            {
                Console.WriteLine($"  Received: {message.Body}");
                Console.WriteLine($"  SequenceNumber: {message.SequenceNumber}");
                
                await sessionReceiver.CompleteMessageAsync(message);
            }

            Console.WriteLine($"Session {sessionReceiver.SessionId} complete!");
        }
        finally
        {
            await sessionReceiver.DisposeAsync();
        }
    }
}
```

**Process multiple sessions in parallel:**

```csharp
// Process 3 sessions concurrently
var tasks = Enumerable.Range(0, 3).Select(async i =>
{
    var sessionReceiver = await client.AcceptNextSessionAsync(queueName);
    
    Console.WriteLine($"Worker {i} processing session: {sessionReceiver.SessionId}");
    
    await foreach (var message in sessionReceiver.ReceiveMessagesAsync())
    {
        Console.WriteLine($"  [{sessionReceiver.SessionId}] {message.Body}");
        await sessionReceiver.CompleteMessageAsync(message);
    }
    
    await sessionReceiver.DisposeAsync();
});

await Task.WhenAll(tasks);
```

---

## Part 5: Dead-Letter Queue (DLQ)

### Step 1: Simulate Failed Processing

```csharp
using Azure.Messaging.ServiceBus;
using System;
using System.Threading.Tasks;

class DLQDemo
{
    static string connectionString = "<YOUR_CONNECTION_STRING>";
    static string queueName = "demoqueue";

    static async Task Main(string[] args)
    {
        await using var client = new ServiceBusClient(connectionString);
        var receiver = client.CreateReceiver(queueName);

        try
        {
            var message = await receiver.ReceiveMessageAsync();

            if (message != null)
            {
                Console.WriteLine($"Received: {message.Body}");
                Console.WriteLine($"DeliveryCount: {message.DeliveryCount}");

                try
                {
                    // Simulate processing failure
                    throw new Exception("Simulated processing error");
                }
                catch (Exception ex)
                {
                    if (message.DeliveryCount >= 3)
                    {
                        // Give up after 3 retries, move to DLQ
                        await receiver.DeadLetterMessageAsync(
                            message,
                            deadLetterReason: "MaxRetriesExceeded",
                            deadLetterErrorDescription: ex.Message);
                        
                        Console.WriteLine("❌ Message moved to dead-letter queue");
                    }
                    else
                    {
                        // Retry by abandoning
                        await receiver.AbandonMessageAsync(message);
                        Console.WriteLine($"⚠️  Retry {message.DeliveryCount}/3");
                    }
                }
            }
        }
        finally
        {
            await receiver.DisposeAsync();
        }
    }
}
```

### Step 2: Process Dead-Letter Queue

```csharp
using Azure.Messaging.ServiceBus;
using System;
using System.Threading.Tasks;

class DLQProcessor
{
    static string connectionString = "<YOUR_CONNECTION_STRING>";
    static string queueName = "demoqueue";

    static async Task Main(string[] args)
    {
        await using var client = new ServiceBusClient(connectionString);
        
        // Create receiver for dead-letter queue
        var dlqReceiver = client.CreateReceiver(
            queueName,
            new ServiceBusReceiverOptions 
            { 
                SubQueue = SubQueue.DeadLetter 
            });

        try
        {
            Console.WriteLine("Processing dead-letter queue...\n");

            var messages = await dlqReceiver.ReceiveMessagesAsync(
                maxMessages: 10,
                maxWaitTime: TimeSpan.FromSeconds(5));

            foreach (var message in messages)
            {
                Console.WriteLine($"Dead-letter message:");
                Console.WriteLine($"  MessageId: {message.MessageId}");
                Console.WriteLine($"  Body: {message.Body}");
                Console.WriteLine($"  DeadLetterReason: {message.DeadLetterReason}");
                Console.WriteLine($"  DeadLetterErrorDescription: {message.DeadLetterErrorDescription}");
                Console.WriteLine($"  DeliveryCount: {message.DeliveryCount}");
                Console.WriteLine($"  EnqueuedTime: {message.EnqueuedTime}");

                // Option 1: Fix and resubmit to main queue
                // Option 2: Log and delete
                // Option 3: Move to error storage for later analysis

                await dlqReceiver.CompleteMessageAsync(message);
                Console.WriteLine("  ✅ DLQ message processed\n");
            }

            if (messages.Count == 0)
            {
                Console.WriteLine("No messages in dead-letter queue");
            }
        }
        finally
        {
            await dlqReceiver.DisposeAsync();
        }
    }
}
```

---

## Часть 6: Мониторинг метрик очереди

### Просмотр метрик в Azure Portal

1. Откройте Azure Portal
2. Перейдите в ваш **Service Bus namespace**
3. Выберите раздел **Queues** → выберите нужную очередь
4. На вкладке **Overview** проверьте основные показатели:

- **Active message count** — количество активных сообщений
- **Dead-letter message count** — количество сообщений в DLQ
- **Size (MB)** — текущий размер очереди
- **Incoming / Outgoing messages** — входящий и исходящий поток

---

### Что анализировать

- Рост **Active messages** → consumers не успевают обрабатывать
- Рост **Dead-letter messages** → проблемы обработки
- Резкие пики **Incoming messages** → всплеск нагрузки
- Низкий Outgoing при высоком Incoming → узкое место

---

### Запрос метрик через Azure CLI

Через CLI можно:

- Получить общее состояние очереди
- Проверить количество сообщений
- Автоматизировать мониторинг
- Интегрировать проверку в CI/CD или скрипты

---

## Экзаменационный акцент (AZ-204)

Важно понимать:

- Active messages → backlog
- Dead-letter count → ошибки обработки
- DeliveryCount → повторные попытки
- Метрики доступны через Azure Monitor

Если в вопросе говорится о мониторинге очереди или проблемах обработки — нужно анализировать метрики namespace и конкретной очереди.

Главная идея:

Мониторинг = контроль нагрузки + контроль ошибок + предотвращение инцидентов.

```bash
# Get active message count
az servicebus queue show \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name $QUEUE_NAME \
  --query "countDetails.activeMessageCount" \
  --output tsv

# Get dead-letter message count
az servicebus queue show \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name $QUEUE_NAME \
  --query "countDetails.deadLetterMessageCount" \
  --output tsv

# Get all queue properties
az servicebus queue show \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name $QUEUE_NAME
```

### Monitor with Azure Monitor

```bash
# Enable diagnostic settings
az monitor diagnostic-settings create \
  --resource /subscriptions/<subscription-id>/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.ServiceBus/namespaces/$NAMESPACE_NAME \
  --name "servicebus-diagnostics" \
  --logs '[{"category": "OperationalLogs", "enabled": true}]' \
  --metrics '[{"category": "AllMetrics", "enabled": true}]' \
  --workspace /subscriptions/<subscription-id>/resourceGroups/$RESOURCE_GROUP/providers/Microsoft.OperationalInsights/workspaces/myWorkspace
```

---

## Part 7: Cleanup

```bash
# Delete resource group (removes all resources)
az group delete \
  --name $RESOURCE_GROUP \
  --yes \
  --no-wait

echo "Cleanup initiated"
```

---

## Troubleshooting

### Issue: "Unauthorized access" error

**Solution:**
```bash
# Regenerate connection string
az servicebus namespace authorization-rule keys renew \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name RootManageSharedAccessKey \
  --key PrimaryKey
```

### Issue: Messages not received

**Check:**
1. Connection string is correct
2. Queue name matches exactly
3. Messages haven't expired (TTL)
4. Queue is not full (check max size)

```bash
# Verify queue exists and has messages
az servicebus queue show \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name $QUEUE_NAME \
  --query "countDetails"
```

### Issue: Lock timeout errors

**Solution:** Increase lock duration or process faster
```bash
az servicebus queue update \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name $QUEUE_NAME \
  --lock-duration PT5M  # 5 minutes
```

### Issue: Too many dead-letter messages

**Solution:** Check DLQ and increase max-delivery-count
```bash
az servicebus queue update \
  --resource-group $RESOURCE_GROUP \
  --namespace-name $NAMESPACE_NAME \
  --name $QUEUE_NAME \
  --max-delivery-count 20
```

---
## Основные выводы

✅ **Создали Service Bus namespace и очередь**
- Развернули ресурсы через Azure CLI
- Настроили параметры очереди (LockDuration, TTL, MaxDeliveryCount)

✅ **Отправили сообщения в очередь**
- Одиночные сообщения
- Пакетная отправка (batch)
- Сообщения с пользовательскими свойствами

✅ **Получили сообщения в режиме Peek Lock**
- Завершали (Complete) после успешной обработки
- Освобождали (Abandon) для повторной попытки
- Перемещали в DLQ после превышения числа попыток

✅ **Использовали sessions для FIFO**
- Группировали сообщения по SessionId
- Обрабатывали разные сессии параллельно

✅ **Работали с Dead-Letter Queue**
- Перемещали проблемные сообщения в DLQ
- Анализировали и повторно обрабатывали их

✅ **Мониторили метрики очереди**
- Просматривали показатели в Azure Portal
- Запрашивали метрики через Azure CLI

---

## Советы для экзамена AZ-204

1. **Peek Lock — режим по умолчанию и рекомендованный** для надёжной обработки
2. **Sessions обеспечивают FIFO** (в пределах сессии)
3. **Dead-Letter Queue** хранит недоставленные сообщения
4. **MaxDeliveryCount** определяет, когда сообщение попадёт в DLQ
5. **LockDuration** должен соответствовать времени обработки
6. **MessageId** используется для duplicate detection и идемпотентности
7. **Batch-отправка** повышает производительность

---

## Главное правило

В режиме Peek Lock сообщение обязательно нужно:

- Завершить (Complete),
- Освободить (Abandon),
- Или отправить в DLQ.

Иначе произойдёт истечение блокировки и повторная доставка.

Контроль обработки = надёжность системы.