# Надёжность доставки событий в Azure Event Grid

## Обзор

Azure Event Grid обеспечивает **надёжную доставку событий** благодаря встроенным механизмам повторной отправки, dead-lettering и гибким настройкам доставки.

---

## Ключевые возможности

- **At-least-once delivery**  
  Событие гарантированно будет доставлено хотя бы один раз  
  (возможны дубликаты)

- **Retry с экспоненциальной задержкой**  
  Автоматические повторные попытки при ошибке доставки

- **Dead-lettering**  
  Сохранение недоставленных событий для последующего анализа

- **Output batching**  
  Отправка нескольких событий одним HTTP-запросом

- **Delayed delivery**  
  Автоматическая пауза при недоступности endpoint

- **Custom delivery properties**  
  Возможность добавлять пользовательские HTTP-заголовки

---

# Механизм повторной доставки (Retry)

Event Grid автоматически повторяет доставку события, если:

- endpoint не отвечает;
- возвращается ошибка;
- происходит таймаут;
- возвращаются определённые HTTP-коды (например, 5xx).

---

## График повторных попыток

Используется **exponential backoff**:

- первая повторная попытка через ~30 секунд;
- далее интервал постепенно увеличивается;
- максимальное время жизни события — до 24 часов (по умолчанию);
- до 30 попыток доставки.

Если за это время доставка не удалась:

→ событие считается недоставленным  
→ при наличии настроенного dead-letter оно сохраняется

---

## Что это означает

- Subscriber должен быть **идемпотентным**
- Нельзя предполагать exactly-once delivery
- Ошибка 5xx запускает retry
- 200 OK означает успешную доставку

---

## Архитектурные рекомендации

- Быстро возвращайте HTTP 200
- Выполняйте тяжёлую обработку асинхронно
- Реализуйте защиту от дубликатов (через `id`)
- Настройте dead-letter storage

---

## Важно для AZ-204

На экзамене нужно помнить:

- Модель доставки — **at-least-once**
- Используется **exponential backoff**
- По умолчанию: до 30 попыток, TTL — 24 часа
- Dead-lettering предотвращает потерю событий
- Subscriber должен корректно обрабатывать дубликаты

Это одна из ключевых тем по Event Grid.

```
Attempt    Wait Time       Cumulative Time
1          0 seconds       0 seconds
2          30 seconds      30 seconds
3          1 minute        1.5 minutes
4          2 minutes       3.5 minutes
5          4 minutes       7.5 minutes
6          8 minutes       15.5 minutes
7          16 minutes      31.5 minutes
8          32 minutes      63.5 minutes
...        ...             ...
30         ~13 hours       ~24 hours (default TTL)
```

**Retry Behavior:**
1. **First attempt**: Immediate delivery
2. **Second attempt**: After 30 seconds
3. **Subsequent attempts**: Exponential backoff (doubles each time)
4. **Randomization**: Small random delay added to prevent thundering herd
5. **Maximum attempts**: Configurable (1-30, default 30)
6. **Time-to-live (TTL)**: Configurable (1-1440 minutes, default 1440 = 24 hours)

### Retry Flow Diagram

```
┌─────────────┐
│   Publish   │
│    Event    │
└──────┬──────┘
       │
       ▼
┌─────────────────────┐
│   Event Grid        │
│   (Queue Event)     │
└──────┬──────────────┘
       │
       ▼
┌─────────────────────┐
│ Deliver to Endpoint │
└──────┬──────────────┘
       │
       ├─────────────────────────┐
       │                         │
       ▼                         ▼
  ┌─────────┐            ┌────────────┐
  │ Success │            │   Failure  │
  │ (2xx)   │            │  (see list)│
  └────┬────┘            └─────┬──────┘
       │                       │
       ▼                       ▼
  ┌─────────┐          ┌──────────────┐
  │  Done   │          │ Retry Policy │
  └─────────┘          │  Evaluation  │
                       └─────┬────────┘
                             │
                ┌────────────┴────────────┐
                │                         │
                ▼                         ▼
    ┌───────────────────┐      ┌──────────────────┐
    │ Within TTL &      │      │ Exceeded TTL or  │
    │ Max Attempts?     │      │ Max Attempts?    │
    └────┬──────────────┘      └────────┬─────────┘
         │                              │
         ▼                              ▼
┌─────────────────┐           ┌──────────────────┐
│ Wait (backoff)  │           │  Dead Letter     │
│ Then Retry      │           │  (if configured) │
└─────────────────┘           │  or Drop Event   │
                              └──────────────────┘
```

---

## Retry Policy Configuration

### Default Retry Policy

```json
{
  "retryPolicy": {
    "maxDeliveryAttempts": 30,
    "eventTimeToLiveInMinutes": 1440
  }
}
```

### Retry Policy Properties

| Property | Description | Min | Max | Default |
|----------|-------------|-----|-----|---------|
| `maxDeliveryAttempts` | Maximum number of delivery attempts | 1 | 30 | 30 |
| `eventTimeToLiveInMinutes` | Time before event expires | 1 | 1440 (24 hours) | 1440 |

### Configure Retry Policy (Azure CLI)

```bash
# Create subscription with custom retry policy
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage" \
  --endpoint https://myfunction.azurewebsites.net/api/handler \
  --max-delivery-attempts 10 \
  --event-ttl 60

# Update existing subscription retry policy
az eventgrid event-subscription update \
  --name mySubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage" \
  --max-delivery-attempts 5 \
  --event-ttl 30
```

### Configure Retry Policy (Azure Portal)

1. Navigate to **Event Grid Topic** or **System Topic**
2. Select **Event Subscriptions**
3. Create or edit subscription
4. In **Additional Features** tab:
   - Set **Max delivery attempts**: 1-30
   - Set **Event time to live**: 1-1440 minutes
5. Save configuration

### Configure Retry Policy (ARM Template)

```json
{
  "type": "Microsoft.EventGrid/eventSubscriptions",
  "apiVersion": "2022-06-15",
  "name": "mySubscription",
  "properties": {
    "destination": {
      "endpointType": "WebHook",
      "properties": {
        "endpointUrl": "https://myfunction.azurewebsites.net/api/handler"
      }
    },
    "retryPolicy": {
      "maxDeliveryAttempts": 10,
      "eventTimeToLiveInMinutes": 60
    }
  }
}
```

---
# Retryable и Non-Retryable ошибки

Azure Event Grid по-разному реагирует на различные HTTP-ответы endpoint’а.

---

## Retryable ошибки (Event Grid выполнит повторную попытку)

| Тип ошибки | HTTP-код | Описание | Повтор? |
|------------|----------|----------|---------|
| Server Error | 500 | Внутренняя ошибка сервера | ✅ Да |
| Service Unavailable | 503 | Сервис временно недоступен | ✅ Да |
| Gateway Timeout | 504 | Таймаут шлюза | ✅ Да |
| Request Timeout | 408 | Таймаут запроса | ✅ Да |
| Too Many Requests | 429 | Превышен лимит запросов | ✅ Да |
| Network Errors | N/A | DNS, проблемы соединения | ✅ Да |

---

## Non-Retryable ошибки (повтор не выполняется)

| Тип ошибки | HTTP-код | Описание | Повтор? |
|------------|----------|----------|---------|
| Bad Request | 400 | Неверный формат запроса | ❌ Нет |
| Unauthorized | 401 | Ошибка аутентификации (webhook) | ❌ Нет |
| Not Found | 404 | Endpoint не найден | ❌ Нет |
| Payload Too Large | 413 | Слишком большой размер | ❌ Нет |
| URI Too Long | 414 | Слишком длинный URI | ❌ Нет |
| Unsupported Media Type | 415 | Неподдерживаемый тип данных | ❌ Нет |

---

## Важные замечания

- **401 Unauthorized (Webhook)**  
  Не повторяется — предполагается ошибка конфигурации.

- **401 для Azure-сервисов**  
  Может повторяться (временные проблемы с Azure AD).

- **Любой non-2xx ответ**  
  Обычно вызывает retry, если не относится к non-retryable.

- **Отсутствие ответа (silent failure)**  
  Запускает retry-механизм.

---

## Архитектурные рекомендации

- Возвращайте корректные HTTP-коды.
- Не используйте 400/401 для временных ошибок.
- Для перегрузки используйте 429 или 503.
- Обрабатывайте повторные доставки (идемпотентность).

---

## Важно для AZ-204

Нужно помнить:

- 5xx → повторная попытка
- 429 → повторная попытка
- 400/404 → повтор не выполняется
- Модель доставки — at-least-once
- Отсутствие ответа = retry

Экзамен часто проверяет понимание различия между retryable и non-retryable ошибками.

### Handling Non-Retryable Errors

```csharp
[FunctionName("EventHandler")]
public static async Task<IActionResult> Run(
    [EventGridTrigger] EventGridEvent eventGridEvent,
    ILogger log)
{
    try
    {
        // Validate event
        if (string.IsNullOrEmpty(eventGridEvent.Subject))
        {
            log.LogError("Invalid event: missing subject");
            // Return 400 - don't retry invalid events
            return new BadRequestObjectResult("Event validation failed");
        }
        
        // Process event
        await ProcessEvent(eventGridEvent);
        
        // Return 200 - success
        return new OkResult();
    }
    catch (InvalidOperationException ex)
    {
        log.LogError($"Validation error: {ex.Message}");
        // Return 400 - don't retry validation errors
        return new BadRequestResult();
    }
    catch (Exception ex)
    {
        log.LogError($"Processing error: {ex.Message}");
        // Return 500 - retry transient errors
        return new StatusCodeResult(500);
    }
}
```

---

## Dead-Lettering

**Dead-lettering** stores events that can't be delivered after all retry attempts are exhausted.

### Dead-Letter Configuration

```bash
# Create subscription with dead-letter storage
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage" \
  --endpoint https://myfunction.azurewebsites.net/api/handler \
  --deadletter-endpoint "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/deadletterstorage/blobServices/default/containers/eventgrid-deadletter"
```

### Dead-Letter Blob Container Structure

```
eventgrid-deadletter/
├── 2024/
│   ├── 01/
│   │   ├── 15/
│   │   │   ├── 14/
│   │   │   │   ├── 30/
│   │   │   │   │   └── deadletter-event-1.json
│   │   │   │   │   └── deadletter-event-2.json
```

**Naming Pattern:**
```
{container}/{year}/{month}/{day}/{hour}/{minute}/{event-id}.json
```

### Dead-Letter Event Format

```json
{
  "id": "9aeb0fdf-c01e-0131-0922-9eb54906e209",
  "eventTime": "2024-01-15T14:30:00Z",
  "eventType": "Microsoft.Storage.BlobCreated",
  "dataVersion": "1.0",
  "metadataVersion": "1",
  "topic": "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage",
  "subject": "/blobServices/default/containers/images/blobs/photo.jpg",
  "data": {
    "api": "PutBlob",
    "contentType": "image/jpeg",
    "url": "https://mystorage.blob.core.windows.net/images/photo.jpg"
  },
  "deadLetterReason": "MaximumDeliveryAttemptsExceeded",
  "deliveryAttempts": 30,
  "lastDeliveryAttemptTime": "2024-01-16T14:30:00Z",
  "lastHttpStatusCode": 503
}
```

## Причины отправки в Dead Letter

Если событие не удалось доставить, Event Grid может сохранить его в настроенном хранилище (Dead Letter Storage).

### Возможные причины

| Причина | Описание |
|----------|-----------|
| `MaxDeliveryAttemptsExceeded` | Превышено максимальное количество попыток доставки |
| `EventTimeToLiveExceeded` | Истёк срок жизни события (TTL) |
| `DestinationEndpointNotFound` | Endpoint удалён или не существует |
| `EndpointDisabled` | Endpoint отключён Event Grid |

---

## Что это означает

Dead Letter используется для:

- предотвращения потери событий;
- анализа ошибок доставки;
- повторной обработки вручную;
- расследования проблем конфигурации.

---

## Архитектурные рекомендации

- Всегда настраивайте dead-letter storage для production.
- Используйте отдельный Blob container.
- Мониторьте количество событий в dead-letter.
- Реализуйте процесс повторной обработки.

---

## Важно для AZ-204

Нужно помнить:

- Dead Letter защищает от потери событий.
- Основные причины: превышение retry или истечение TTL.
- TTL по умолчанию — до 24 часов.
- MaxDeliveryAttempts по умолчанию — до 30 попыток.

Понимание причин dead-letter — частая тема в вопросах по Event Grid.

### Processing Dead-Letter Events

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using System.Text.Json;

public class DeadLetterProcessor
{
    private readonly BlobServiceClient _blobServiceClient;
    
    public DeadLetterProcessor(string connectionString)
    {
        _blobServiceClient = new BlobServiceClient(connectionString);
    }
    
    public async Task ProcessDeadLetterEvents(string containerName)
    {
        var containerClient = _blobServiceClient.GetBlobContainerClient(containerName);
        
        await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
        {
            var blobClient = containerClient.GetBlobClient(blobItem.Name);
            BlobDownloadResult download = await blobClient.DownloadContentAsync();
            string content = download.Content.ToString();
            
            var deadLetterEvent = JsonSerializer.Deserialize<DeadLetterEvent>(content);
            
            Console.WriteLine($"Dead Letter Event ID: {deadLetterEvent.Id}");
            Console.WriteLine($"Reason: {deadLetterEvent.DeadLetterReason}");
            Console.WriteLine($"Attempts: {deadLetterEvent.DeliveryAttempts}");
            Console.WriteLine($"Last Status: {deadLetterEvent.LastHttpStatusCode}");
            
            // Reprocess or investigate
            await ReprocessEvent(deadLetterEvent);
            
            // Optionally delete after processing
            await blobClient.DeleteAsync();
        }
    }
    
    private async Task ReprocessEvent(DeadLetterEvent deadLetterEvent)
    {
        // Implement reprocessing logic
        // Option 1: Manual investigation
        // Option 2: Republish to Event Grid
        // Option 3: Send to alternate processing pipeline
    }
}
```

## Best Practices для Dead Letter

1️⃣ **Всегда настраивайте dead-letter storage** для production-подписок  
2️⃣ **Мониторьте контейнер dead-letter** через alerts  
3️⃣ **Автоматизируйте обработку** типовых ошибок  
4️⃣ **Анализируйте повторяющиеся паттерны** в недоставленных событиях  
5️⃣ **Регулярно очищайте старые события**

---

### Архитектурные рекомендации

- Используйте отдельный Blob container.
- Настройте алерт при увеличении количества dead-letter событий.
- Храните метаданные для диагностики.
- Автоматизируйте повторную публикацию при временных ошибках.

Dead-letter — это механизм защиты от потери данных, а не просто лог ошибок.

---

# Delayed Delivery

Event Grid автоматически **замедляет доставку** событий к endpoint’ам, которые стабильно возвращают ошибки.

---

## Поведение delayed delivery

### Условия срабатывания

- Несколько подряд неудачных попыток доставки
- Повторяющиеся ошибки (500, 503, 504)
- Проблемы сетевого подключения

---

### Продолжительность задержки

- Начинается с **5 минут**
- Постепенно увеличивается
- Максимум — **1 час**
- Доставка автоматически возобновляется после восстановления endpoint

---

## Преимущества

- Снижает нагрузку на проблемный сервис
- Даёт время на восстановление
- Предотвращает лавинообразные повторные запросы
- Повышает общую устойчивость системы

---

## Архитектурный смысл

Delayed delivery — это защитный механизм, который:

- уменьшает cascading failures;
- предотвращает перегрузку;
- делает систему более отказоустойчивой.

---

## Важно для AZ-204

Нужно помнить:

- Dead-letter предотвращает потерю событий
- Delayed delivery активируется при повторяющихся ошибках
- Начальная задержка — 5 минут
- Максимальная — до 1 часа
- Механизм работает автоматически

Эти механизмы — ключевые элементы надёжности Event Grid.

### Monitoring Delayed Delivery

```bash
# Check delivery metrics
az monitor metrics list \
  --resource "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic" \
  --metric "DeliveryFailedCount" \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z

# Check matched events
az monitor metrics list \
  --resource "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic" \
  --metric "MatchedEventCount" \
  --start-time 2024-01-15T00:00:00Z
```

---

## Output Batching

**Output batching** delivers multiple events in a single HTTP request to improve throughput and reduce costs.

### Batch Configuration

```bash
# Create subscription with batching
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage" \
  --endpoint https://myfunction.azurewebsites.net/api/handler \
  --max-events-per-batch 100 \
  --preferred-batch-size-in-kilobytes 128
```

### Batch Properties

| Property | Description | Min | Max | Default |
|----------|-------------|-----|-----|---------|
| `maxEventsPerBatch` | Maximum events in single request | 1 | 5000 | 1 |
| `preferredBatchSizeInKilobytes` | Preferred batch size in KB | 1 | 1024 | 64 |

**Batching Behavior:**
- Event Grid waits for either **max events** or **preferred size**, whichever comes first
- Small wait time (~1 second) to collect events
- Does NOT wait indefinitely (optimizes for latency)
- Useful for high-volume scenarios

### Handling Batched Events

```csharp
[FunctionName("BatchEventHandler")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "post")] HttpRequest req,
    ILogger log)
{
    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    var events = JsonSerializer.Deserialize<EventGridEvent[]>(requestBody);
    
    log.LogInformation($"Received batch of {events.Length} events");
    
    var tasks = events.Select(async eventGridEvent =>
    {
        try
        {
            await ProcessEvent(eventGridEvent);
            log.LogInformation($"Processed event: {eventGridEvent.Id}");
        }
        catch (Exception ex)
        {
            log.LogError($"Failed to process event {eventGridEvent.Id}: {ex.Message}");
            throw; // Fail entire batch
        }
    });
    
    await Task.WhenAll(tasks);
    
    return new OkResult();
}
```

### Batch Processing Strategies

**Strategy 1: Parallel Processing**
```csharp
// Process all events in parallel
await Task.WhenAll(events.Select(ProcessEvent));
```

**Strategy 2: Sequential Processing**
```csharp
// Process events one at a time
foreach (var evt in events)
{
    await ProcessEvent(evt);
}
```

**Strategy 3: Batched Database Operations**
```csharp
// Batch insert to database
var records = events.Select(evt => new Record
{
    Id = evt.Id,
    Data = evt.Data.ToString()
});

await dbContext.Records.AddRangeAsync(records);
await dbContext.SaveChangesAsync();
```

---

## Custom Delivery Properties

Add **custom HTTP headers** to event deliveries for authentication, routing, or metadata.

### Configure Custom Headers

```bash
# Create subscription with custom headers
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage" \
  --endpoint https://myapi.example.com/webhooks/events \
  --delivery-attribute-mapping \
    X-Custom-Header static myValue \
    X-Event-Source dynamic subject \
    X-Event-Id dynamic id \
    X-Event-Time dynamic eventTime \
    X-API-Key static "api-key-12345"
```

### Custom Header Types

| Type | Description | Example |
|------|-------------|---------|
| **Static** | Fixed value for all events | API key, tenant ID |
| **Dynamic** | Value from event property | Subject, ID, timestamp |

### Delivery Attribute Mapping (ARM Template)

```json
{
  "type": "Microsoft.EventGrid/eventSubscriptions",
  "properties": {
    "destination": {
      "endpointType": "WebHook",
      "properties": {
        "endpointUrl": "https://myapi.example.com/webhooks/events"
      }
    },
    "deliveryWithResourceIdentity": {
      "identity": {
        "type": "SystemAssigned"
      },
      "destination": {
        "endpointType": "WebHook",
        "properties": {
          "endpointUrl": "https://myapi.example.com/webhooks/events",
          "deliveryAttributeMappings": [
            {
              "name": "X-API-Key",
              "type": "Static",
              "properties": {
                "value": "api-key-12345",
                "isSecret": true
              }
            },
            {
              "name": "X-Event-Subject",
              "type": "Dynamic",
              "properties": {
                "sourceField": "subject"
              }
            },
            {
              "name": "X-Event-Id",
              "type": "Dynamic",
              "properties": {
                "sourceField": "id"
              }
            }
          ]
        }
      }
    }
  }
}
```

## Ограничения для пользовательских заголовков

- **Максимальное количество заголовков**: 10
- **Максимальный размер одного заголовка**: 4096 байт
- **Secret headers**: Можно пометить как секретные (не отображаются в портале)

---

## Сценарии использования пользовательских заголовков

1️⃣ **Аутентификация**  
API-ключи, токены, секреты

2️⃣ **Маршрутизация**  
Tenant ID, регион, идентификаторы окружения

3️⃣ **Корреляция**  
Trace ID, Request ID

4️⃣ **Метаданные**  
Версия приложения, среда (dev/test/prod), источник системы

---

## Архитектурные рекомендации

- Не передавайте чувствительные данные в открытом виде.
- Используйте secret headers для токенов.
- Ограничивайте количество заголовков.
- Для сложной логики используйте structured payload вместо перегрузки заголовков.

---

# Мониторинг доставки событий

## Ключевые метрики

| Метрика | Описание | Порог для алерта |
|----------|-----------|------------------|
| **PublishSuccessCount** | Успешно опубликованные события | N/A |
| **PublishFailCount** | Ошибки публикации | > 0 |
| **MatchedEventCount** | События, сопоставленные подпискам | Отклонение от baseline |
| **DeliverySuccessCount** | Успешно доставленные события | Отклонение от baseline |
| **DeliveryFailCount** | Ошибки доставки | > 10% |
| **DeadLetterCount** | События в dead-letter | > 0 |
| **DroppedEventCount** | Потерянные события | > 0 |
| **DestinationProcessingDurationInMs** | Время обработки endpoint’ом | > 1000 ms |

---

## Что важно мониторить

- Рост DeliveryFailCount
- Появление DeadLetterCount
- Увеличение времени обработки
- Резкие изменения в MatchedEventCount

---

## Архитектурный вывод

Мониторинг Event Grid позволяет:

- выявлять деградацию endpoint’ов;
- обнаруживать ошибки конфигурации;
- предотвращать потерю событий;
- поддерживать SLA.

---

## Важно для AZ-204

На экзамене могут проверить:

- понимание метрик публикации и доставки;
- роль DeadLetterCount;
- значение DeliveryFailCount;
- использование алертов;
- ограничение на количество и размер custom headers.
### Create Delivery Alert (Azure CLI)

```bash
# Alert on delivery failures
az monitor metrics alert create \
  --name "Event Delivery Failures" \
  --resource-group myRG \
  --scopes "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic" \
  --condition "avg DeliveryFailCount > 10" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action-group "/subscriptions/{sub-id}/resourceGroups/myRG/providers/microsoft.insights/actionGroups/myActionGroup"
```

### Azure Monitor Query (KQL)

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.EVENTGRID"
| where Category == "DeliveryFailures"
| project TimeGenerated, SubscriptionName, EventSubscriptionName, Message, DeliveryAttempts
| order by TimeGenerated desc
| take 100
```

---

# Best Practices для надёжной и эффективной доставки

## Проектирование Retry Policy

1️⃣ **Production-нагрузка**  
Используйте настройки по умолчанию  
(30 попыток, TTL — 24 часа)

2️⃣ **События, чувствительные ко времени**  
Уменьшайте TTL (например, до 60 минут)

3️⃣ **Идемпотентность**  
Обработчики должны корректно обрабатывать повторные доставки

4️⃣ **Быстрая ошибка при валидации**  
Возвращайте 400 для ошибок данных  
(повторная попытка не требуется)

5️⃣ **Временные ошибки**  
Возвращайте 500 для запуска retry

---

## Управление Dead Letter

1️⃣ **Всегда настраивайте**  
Dead-letter storage должен быть включён для всех production-подписок

2️⃣ **Регулярный мониторинг**  
Проверяйте контейнер ежедневно

3️⃣ **Автоматические алерты**  
Настройте уведомления при появлении событий

4️⃣ **Пайплайн повторной обработки**  
Автоматизируйте reprocessing для типовых ошибок

5️⃣ **Политика хранения**  
Удаляйте старые события (например, старше 90 дней)

---

## Оптимизация производительности

1️⃣ **Включайте batching**  
Для high-volume сценариев (100–1000 событий в batch)

2️⃣ **Асинхронная обработка**  
Не выполняйте тяжёлую работу в синхронном потоке

3️⃣ **Быстрый ответ**  
Возвращайте HTTP 200 в течение 30 секунд

4️⃣ **Параллельные обработчики**  
Масштабируйте экземпляры handler’ов

5️⃣ **Мониторинг задержки**  
Отслеживайте `DestinationProcessingDurationInMs`

---

## Архитектурные принципы

- Обработчики должны быть идемпотентными
- Ошибки должны возвращать корректные HTTP-коды
- Dead-letter обязателен в production
- Batching снижает сетевые накладные расходы
- Retry + Dead-letter = надёжность доставки

---

## Важно для AZ-204

На экзамене часто проверяют:

- TTL по умолчанию — 24 часа
- До 30 попыток доставки
- Разницу между 400 и 500
- Идемпотентность обработчиков
- Настройку dead-letter

Понимание этих best practices критично для вопросов о надёжности Event Grid.
### Error Handling

```csharp
public static class EventHandlerPatterns
{
    // Pattern 1: Return appropriate status codes
    public static IActionResult HandleValidationError()
    {
        return new BadRequestResult(); // Don't retry
    }
    
    public static IActionResult HandleTransientError()
    {
        return new StatusCodeResult(500); // Retry
    }
    
    // Pattern 2: Idempotent processing
    private static HashSet<string> processedEventIds = new();
    
    public static async Task<IActionResult> HandleIdempotent(EventGridEvent evt)
    {
        if (processedEventIds.Contains(evt.Id))
        {
            return new OkResult(); // Already processed
        }
        
        await ProcessEvent(evt);
        processedEventIds.Add(evt.Id);
        
        return new OkResult();
    }
    
    // Pattern 3: Circuit breaker for downstream services
    public static async Task<IActionResult> HandleWithCircuitBreaker(EventGridEvent evt)
    {
        if (await IsDownstreamHealthy())
        {
            await ProcessEvent(evt);
            return new OkResult();
        }
        else
        {
            return new StatusCodeResult(503); // Service unavailable, retry
        }
    }
}
```

---

# Советы к экзамену AZ-204

## Ключевые концепции

1️⃣ **At-least-once delivery**  
События могут быть доставлены более одного раза.

2️⃣ **Retry policy по умолчанию**  
30 попыток, TTL — 24 часа (1440 минут).

3️⃣ **Exponential backoff**  
Первая повторная попытка через 30 секунд,  
затем интервал увеличивается экспоненциально.

4️⃣ **Non-retryable ошибки**  
400, 401 (для webhook), 413.

5️⃣ **Dead-lettering**  
Требует Azure Storage (blob container).

6️⃣ **Output batching**  
До 5000 событий или 1024 KB в одном batch.

7️⃣ **Custom headers**  
До 10 заголовков, каждый до 4096 байт.

---

# Частые экзаменационные сценарии

### Сценарий 1
События не повторяются

- ✅ Проверьте, не возвращает ли endpoint non-retryable код (400, 413)
- ✅ Проверьте настройки retry policy
- ❌ Не предполагайте, что любая ошибка вызывает retry

---

### Сценарий 2
Минимизация стоимости при высоком объёме событий

- ✅ Включить batching
- ✅ Увеличить количество событий в batch
- ❌ Не обрабатывать события по одному

---

### Сценарий 3
События «пропадают»

- ❌ Dead-letter не настроен
- ✅ Настроить blob container для dead-letter
- ✅ Мониторить DroppedEventCount

---

### Сценарий 4
Endpoint перегружен повторными попытками

- ✅ Event Grid использует exponential backoff
- ✅ Учитывать delayed delivery
- ✅ Масштабировать endpoint

---

# Что обязательно помнить

- **Максимум попыток по умолчанию**: 30
- **TTL по умолчанию**: 1440 минут (24 часа)
- **Первая повторная попытка**: через 30 секунд
- **Дальнейшие попытки**: экспоненциальная задержка
- **Формат dead-letter**: JSON в blob storage
- **Максимум batch**: 5000 событий или 1024 KB
- **Custom headers**: до 10, по 4096 байт
- **Non-retryable коды**: 400, 401 (webhook), 413, 414, 415
- **Retryable коды**: 500, 503, 504, 408, 429

---

## Экзаменационный акцент

Вопросы часто проверяют:

- различие между retryable и non-retryable ошибками;
- значения TTL и max attempts по умолчанию;
- необходимость dead-letter;
- поведение exponential backoff;
- ограничения batching и custom headers.

Если видите вопрос про «надёжность доставки» — думайте о retry, dead-letter и idempotency.
### Quick Command Reference

```bash
# Configure retry policy
az eventgrid event-subscription create \
  --max-delivery-attempts 10 \
  --event-ttl 60

# Configure dead-letter
--deadletter-endpoint "/subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/{sa}/blobServices/default/containers/{container}"

# Configure batching
--max-events-per-batch 100 \
--preferred-batch-size-in-kilobytes 128

# Add custom headers
--delivery-attribute-mapping \
  X-Custom-Header static myValue \
  X-Event-Id dynamic id
```

---

# Итоги по надёжности доставки событий

## Возможности Event Grid

- **Retry mechanism**  
  Автоматические повторные попытки с экспоненциальной задержкой  
  (от 30 секунд до ~13 часов)

- **Retry policy**  
  Настраиваемое количество попыток (1–30)  
  и TTL (1–1440 минут)

- **Dead-lettering**  
  Сохранение недоставленных событий в Blob Storage

- **Output batching**  
  До 5000 событий в одном запросе

- **Delayed delivery**  
  Автоматическая пауза для нестабильных endpoint’ов

- **Custom headers**  
  До 10 пользовательских HTTP-заголовков

---

# Best Practices

- ✅ Использовать настройки retry по умолчанию  
  (30 попыток, 24 часа)

- ✅ Всегда настраивать dead-letter storage

- ✅ Включать batching при высоком объёме событий

- ✅ Проектировать идемпотентные обработчики

- ✅ Возвращать корректные HTTP-коды  
  400 — ошибки валидации  
  500 — временные ошибки

- ✅ Мониторить метрики доставки и настраивать алерты

- ✅ Регулярно обрабатывать события из dead-letter

---

# Ключевые выводы

- Event Grid гарантирует **at-least-once delivery**
- События могут быть доставлены **несколько раз**
- **Non-retryable ошибки** (400, 413) не вызывают повторную отправку
- Dead-letter предотвращает потерю событий
- Batching повышает пропускную способность и снижает стоимость

---

## Экзаменационный акцент (AZ-204)

Обязательно помнить:

- TTL по умолчанию — 24 часа
- До 30 попыток доставки
- Exponential backoff начинается с 30 секунд
- Модель доставки — at-least-once
- Обработчики должны быть идемпотентными
- Dead-letter — обязательный элемент production-архитектуры

Если вопрос о надёжности доставки — думайте о retry, TTL, dead-letter и idempotency.