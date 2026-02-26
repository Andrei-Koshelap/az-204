# Получение событий через Webhooks в Azure Event Grid

## Обзор

**Webhook** — это HTTP/HTTPS endpoint, который получает события от Azure Event Grid через POST-запросы.  
Event Grid использует push-модель доставки, отправляя события напрямую в ваш endpoint.

---

## Основные возможности

- **Push delivery**  
  Event Grid сам отправляет события

- **Endpoint validation**  
  Подтверждение владения endpoint перед началом доставки

- **Гибкость размещения**  
  Любой HTTP endpoint (cloud, on-premises, контейнеры)

- **Автоматический retry**  
  Повторная отправка с exponential backoff

- **Кастомная аутентификация**  
  Возможность добавлять заголовки (API keys, OAuth)

---

# Требования к Webhook Endpoint

## Базовые требования

| Требование | Описание |
|------------|-----------|
| **Протокол** | HTTPS (HTTP поддерживается, но не рекомендуется) |
| **Время ответа** | Ответ должен быть отправлен в течение **30 секунд** |
| **Код ответа** | Для успешной обработки — **HTTP 200 OK** |
| **Сертификат** | Действительный TLS/SSL сертификат |
| **Доступность** | Публичный доступ или через Azure Relay / Private Endpoint |
| **Валидация** | Обработка validation handshake |

---

# Требования к сертификатам

## Production

- ✅ Сертификат от доверенного центра сертификации (CA)
- ✅ Domain validation или Extended validation
- ❌ Self-signed сертификаты запрещены

## Development / Testing

- ⚠️ Self-signed сертификаты допускаются (через Azure CLI флаг)
- Не рекомендуется использовать в production

---

## Архитектурные рекомендации

- Используйте HTTPS всегда
- Быстро возвращайте 200 OK
- Переносите тяжёлую обработку в асинхронные процессы
- Реализуйте идемпотентность
- Настройте мониторинг DeliveryFailCount

---

## Важно для AZ-204

Нужно помнить:

- Webhook использует push-модель
- Ответ должен быть в течение 30 секунд
- 200 OK подтверждает успешную доставку
- Self-signed сертификаты не подходят для production
- Endpoint должен пройти validation handshake

Вопросы часто проверяют требования к webhook и поведение retry.

```bash
# Allow self-signed certificates (development only)
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id $TOPIC_ID \
  --endpoint https://localhost:5001/api/events \
  --azure-active-directory-tenant-id $TENANT_ID \
  --endpoint-type webhook \
  --deadletter-endpoint $DEADLETTER_ENDPOINT
```

---

# Валидация Webhook Endpoint

Azure Event Grid требует обязательную **endpoint validation**, чтобы подтвердить, что вы действительно владеете указанным webhook URL.

---

## Зачем нужна валидация

- **Защита от злоупотреблений**  
  Предотвращает отправку событий на чужие endpoint’ы

- **Подтверждение владения**  
  Убеждается, что вы контролируете указанный URL

- **Исключение ошибок конфигурации**  
  Предотвращает подписки на неверные адреса

---

# Автоматическая валидация

Следующие Azure-сервисы выполняют валидацию автоматически:

| Сервис | Примечание |
|--------|------------|
| **Azure Functions** | При использовании Event Grid trigger |
| **Logic Apps** | Через встроенный Event Grid connector |
| **Azure Automation** | Webhook-triggered runbooks |

Для этих сервисов **не требуется писать код для валидации**.

---

# Методы валидации

## Метод 1: Синхронный Handshake (рекомендуется)

### Как работает

1️⃣ Event Grid отправляет событие типа  
`SubscriptionValidationEvent`

2️⃣ Ваш endpoint извлекает `validationCode` из `data`

3️⃣ Endpoint возвращает код в ответе **синхронно**

4️⃣ Подписка становится активной сразу

---

## Что важно

- Ответ должен быть HTTP 200
- Код должен быть возвращён в теле ответа
- Валидация должна происходить в течение 30 секунд
- Без корректного ответа подписка не активируется

---

## Архитектурный смысл

Validation handshake:

- предотвращает случайные или вредоносные подписки
- обеспечивает контроль доступа
- повышает безопасность webhook-интеграций

---

## Важно для AZ-204

Нужно помнить:

- Webhook обязан пройти endpoint validation
- Используется событие `SubscriptionValidationEvent`
- Требуется вернуть `validationCode`
- Azure Functions и Logic Apps обрабатывают это автоматически

Если в вопросе говорится о проблеме активации подписки — почти всегда причина в неправильной обработке validation handshake.

**Validation Event Format:**
```json
[{
  "id": "2d1781af-3a4c-4d7c-bd0c-e34b19da4e66",
  "topic": "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic",
  "subject": "",
  "data": {
    "validationCode": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6",
    "validationUrl": "https://rp-eastus2.eventgrid.azure.net:553/eventsubscriptions/mySubscription/validate?id=512d38b6-c7b8-40c8-89fe-f46f9e9622b6&t=2024-01-15T14:30:00.0000000Z&apiVersion=2018-05-01-preview&token=..."
  },
  "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
  "eventTime": "2024-01-15T14:30:00.0000000Z",
  "metadataVersion": "1",
  "dataVersion": "2"
}]
```

**Response Format:**
```json
{
  "validationResponse": "512d38b6-c7b8-40c8-89fe-f46f9e9622b6"
}
```

**C# Implementation:**
```csharp
using Microsoft.AspNetCore.Mvc;
using Azure.Messaging.EventGrid;
using Azure.Messaging.EventGrid.SystemEvents;

[ApiController]
[Route("api/[controller]")]
public class EventsController : ControllerBase
{
    [HttpPost]
    [HttpOptions] // Support OPTIONS requests for validation
    public async Task<IActionResult> Post()
    {
        // Read request body
        using var reader = new StreamReader(Request.Body);
        string requestBody = await reader.ReadToEndAsync();
        
        // Parse events
        EventGridEvent[] events = EventGridEvent.ParseMany(BinaryData.FromString(requestBody));
        
        foreach (EventGridEvent eventGridEvent in events)
        {
            // Handle validation event
            if (eventGridEvent.EventType == "Microsoft.EventGrid.SubscriptionValidationEvent")
            {
                var validationData = eventGridEvent.Data.ToObjectFromJson<SubscriptionValidationEventData>();
                var validationCode = validationData.ValidationCode;
                
                // Return validation response
                return Ok(new { validationResponse = validationCode });
            }
            
            // Handle business events
            await ProcessEvent(eventGridEvent);
        }
        
        return Ok();
    }
    
    private async Task ProcessEvent(EventGridEvent eventGridEvent)
    {
        // Your event processing logic
        Console.WriteLine($"Event Type: {eventGridEvent.EventType}");
        Console.WriteLine($"Subject: {eventGridEvent.Subject}");
        Console.WriteLine($"Data: {eventGridEvent.Data}");
    }
}
```

**Python Flask Implementation:**
```python
from flask import Flask, request, jsonify
import json

app = Flask(__name__)

@app.route('/api/events', methods=['POST', 'OPTIONS'])
def handle_events():
    # Handle OPTIONS request
    if request.method == 'OPTIONS':
        return '', 200
    
    events = request.json
    
    for event in events:
        event_type = event.get('eventType')
        
        # Handle validation event
        if event_type == 'Microsoft.EventGrid.SubscriptionValidationEvent':
            validation_code = event['data']['validationCode']
            return jsonify({'validationResponse': validation_code}), 200
        
        # Handle business events
        elif event_type == 'Microsoft.Storage.BlobCreated':
            blob_url = event['data']['url']
            print(f'Blob created: {blob_url}')
            # Process the event
        
        # Handle other event types
        else:
            print(f'Received event: {event_type}')
    
    return jsonify({'status': 'success'}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000, ssl_context='adhoc')
```

**Node.js Express Implementation:**
```javascript
const express = require('express');
const app = express();

app.use(express.json());

app.post('/api/events', (req, res) => {
    const events = req.body;
    
    for (const event of events) {
        // Handle validation event
        if (event.eventType === 'Microsoft.EventGrid.SubscriptionValidationEvent') {
            const validationCode = event.data.validationCode;
            return res.status(200).json({ validationResponse: validationCode });
        }
        
        // Handle business events
        console.log(`Event Type: ${event.eventType}`);
        console.log(`Subject: ${event.subject}`);
        console.log(`Data: ${JSON.stringify(event.data)}`);
    }
    
    res.status(200).send('OK');
});

app.listen(5000, () => {
    console.log('Webhook listening on port 5000');
});
```

### Method 2: Asynchronous Handshake (Manual)

**How it works:**
1. Event Grid sends validation event with `validationUrl`
2. You (or automated process) make GET request to `validationUrl` within **5 minutes**
3. Subscription provisioning completes
4. Events start flowing

**When to use:**
- Can't modify webhook code to handle validation event
- Third-party endpoints
- Legacy systems

**Validation Process:**

```bash
# 1. Create subscription (stays in "AwaitingManualAction" state)
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id $TOPIC_ID \
  --endpoint https://mylegacyapp.example.com/events

# 2. Check subscription status
az eventgrid event-subscription show \
  --name mySubscription \
  --source-resource-id $TOPIC_ID \
  --query "provisioningState"
# Output: "AwaitingManualAction"

# 3. Get validation URL from the validation event
# Event Grid sends validation event to your endpoint
# Extract validationUrl from the event data

# 4. Make GET request to validation URL
curl -X GET "https://rp-eastus2.eventgrid.azure.net:553/eventsubscriptions/mySubscription/validate?id=...&apiVersion=2018-05-01-preview&token=..."

# 5. Check subscription status again
az eventgrid event-subscription show \
  --name mySubscription \
  --source-resource-id $TOPIC_ID \
  --query "provisioningState"
# Output: "Succeeded"
```

## Срок действия Validation URL

- ⏱️ **5 минут** на завершение валидации
- ❌ Через 5 минут validation URL становится недействительным
- 🔄 Если время истекло — удалите и создайте подписку заново

Важно: если validation handshake не завершён вовремя, подписка не активируется.

---

# Состояния Provisioning подписки

| Состояние | Описание |
|------------|-----------|
| `Creating` | Подписка создаётся |
| `AwaitingManualAction` | Ожидается ручная валидация (асинхронный handshake) |
| `Succeeded` | Подписка активна и получает события |
| `Failed` | Ошибка валидации или другая ошибка |

---

## Что это означает

- Если статус `AwaitingManualAction` — webhook не завершил валидацию.
- Если статус `Failed` — произошла ошибка или истёк срок validation URL.
- Только статус `Succeeded` означает, что события начнут доставляться.

---

## Важно для AZ-204

Нужно помнить:

- На validation отводится **5 минут**
- При истечении срока требуется пересоздание подписки
- Подписка не получает события, пока статус не `Succeeded`
- `AwaitingManualAction` означает, что handshake не завершён

Частый экзаменационный вопрос — почему подписка не активируется.

---

## Webhook Event Delivery

### Request Format

**HTTP Headers:**
```
POST /api/events HTTP/1.1
Host: mywebhook.example.com
Content-Type: application/json; charset=utf-8
aeg-subscription-name: mySubscription
aeg-event-type: Notification
aeg-data-version: 1.0
aeg-metadata-version: 1
aeg-delivery-count: 1
```

**Request Body (CloudEvents):**
```json
[
  {
    "specversion": "1.0",
    "type": "Microsoft.Storage.BlobCreated",
    "source": "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage",
    "subject": "/blobServices/default/containers/images/blobs/photo.jpg",
    "id": "9aeb0fdf-c01e-0131-0922-9eb54906e209",
    "time": "2024-01-15T14:30:00Z",
    "datacontenttype": "application/json",
    "data": {
      "api": "PutBlob",
      "contentType": "image/jpeg",
      "contentLength": 524288,
      "blobType": "BlockBlob",
      "url": "https://mystorage.blob.core.windows.net/images/photo.jpg"
    }
  }
]
```

### Event Grid HTTP Headers

| Header | Description | Example |
|--------|-------------|---------|
| `aeg-subscription-name` | Name of the event subscription | `mySubscription` |
| `aeg-event-type` | Type of delivery | `Notification`, `SubscriptionValidation` |
| `aeg-data-version` | Data schema version | `1.0` |
| `aeg-metadata-version` | Event metadata version | `1` |
| `aeg-delivery-count` | Delivery attempt number | `1` (first attempt) |

### Response Requirements

**Success Response:**
```
HTTP/1.1 200 OK
Content-Length: 0
```

**Processing Time:**
- Must respond within **30 seconds**
- Longer = timeout and retry
- Process events asynchronously if needed

**Asynchronous Processing Pattern:**
```csharp
[HttpPost]
public async Task<IActionResult> Post()
{
    var events = await ParseEventsAsync(Request.Body);
    
    // Queue events for background processing
    foreach (var evt in events)
    {
        await _queue.EnqueueAsync(evt);
    }
    
    // Return 200 immediately
    return Ok();
}
```

---

## Authentication Options

### Option 1: Custom HTTP Headers (API Keys)

```bash
# Add API key as custom header
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id $TOPIC_ID \
  --endpoint https://myapi.example.com/webhooks/events \
  --delivery-attribute-mapping \
    X-API-Key static "your-api-key-here" \
    X-Client-ID static "client-123"
```

**Webhook Validation:**
```csharp
[HttpPost]
public async Task<IActionResult> Post()
{
    // Validate API key
    if (!Request.Headers.TryGetValue("X-API-Key", out var apiKey) ||
        apiKey != "your-api-key-here")
    {
        return Unauthorized();
    }
    
    // Process events
    await ProcessEventsAsync(Request.Body);
    return Ok();
}
```

### Option 2: Azure AD OAuth Token

**Configure OAuth authentication:**
```json
{
  "deliveryWithResourceIdentity": {
    "identity": {
      "type": "SystemAssigned"
    },
    "destination": {
      "endpointType": "WebHook",
      "properties": {
        "endpointUrl": "https://myapi.azurewebsites.net/api/events",
        "azureActiveDirectoryTenantId": "tenant-id",
        "azureActiveDirectoryApplicationIdOrUri": "api://myapi"
      }
    }
  }
}
```

**Validate JWT Token in Webhook:**
```csharp
using Microsoft.Identity.Web;

[Authorize]
[HttpPost]
public async Task<IActionResult> Post()
{
    // Token automatically validated by [Authorize] attribute
    var userId = User.Identity?.Name;
    
    await ProcessEventsAsync(Request.Body);
    return Ok();
}
```

### Option 3: Query String Parameters

```bash
# Add authentication token in URL
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id $TOPIC_ID \
  --endpoint "https://myapi.example.com/webhooks/events?code=abc123xyz"
```

---

## Webhook Best Practices

### Design Patterns

**Pattern 1: Quick Acknowledgment + Background Processing**
```csharp
[HttpPost]
public async Task<IActionResult> Post()
{
    var events = await ParseEventsAsync(Request.Body);
    
    // Queue for background processing
    foreach (var evt in events)
    {
        await _backgroundQueue.QueueBackgroundWorkItemAsync(async token =>
        {
            await ProcessEventAsync(evt, token);
        });
    }
    
    // Return immediately
    return Ok();
}
```

**Pattern 2: Idempotent Processing**
```csharp
private static readonly ConcurrentDictionary<string, bool> _processedEvents 
    = new ConcurrentDictionary<string, bool>();

public async Task<IActionResult> Post()
{
    var events = await ParseEventsAsync(Request.Body);
    
    foreach (var evt in events)
    {
        // Skip if already processed
        if (_processedEvents.ContainsKey(evt.Id))
        {
            continue;
        }
        
        await ProcessEventAsync(evt);
        _processedEvents.TryAdd(evt.Id, true);
    }
    
    return Ok();
}
```

**Pattern 3: Circuit Breaker for Downstream Services**
```csharp
using Polly;
using Polly.CircuitBreaker;

private static readonly AsyncCircuitBreakerPolicy _circuitBreaker =
    Policy
        .Handle<Exception>()
        .CircuitBreakerAsync(
            exceptionsAllowedBeforeBreaking: 5,
            durationOfBreak: TimeSpan.FromSeconds(30)
        );

[HttpPost]
public async Task<IActionResult> Post()
{
    if (_circuitBreaker.CircuitState == CircuitState.Open)
    {
        // Return 503 to trigger retry
        return StatusCode(503, "Circuit breaker open");
    }
    
    try
    {
        await _circuitBreaker.ExecuteAsync(async () =>
        {
            await ProcessEventsAsync(Request.Body);
        });
        
        return Ok();
    }
    catch
    {
        return StatusCode(500);
    }
}
```

### Error Handling

```csharp
[HttpPost]
public async Task<IActionResult> Post()
{
    try
    {
        var events = await ParseEventsAsync(Request.Body);
        
        foreach (var evt in events)
        {
            try
            {
                // Validate event
                if (string.IsNullOrEmpty(evt.Subject))
                {
                    _logger.LogWarning($"Invalid event: {evt.Id}");
                    // Return 400 - don't retry invalid events
                    return BadRequest("Invalid event format");
                }
                
                // Process event
                await ProcessEventAsync(evt);
            }
            catch (InvalidOperationException ex)
            {
                _logger.LogError($"Validation error: {ex.Message}");
                // Return 400 - don't retry validation errors
                return BadRequest(ex.Message);
            }
            catch (Exception ex)
            {
                _logger.LogError($"Processing error: {ex.Message}");
                // Return 500 - retry transient errors
                return StatusCode(500);
            }
        }
        
        return Ok();
    }
    catch (Exception ex)
    {
        _logger.LogError($"Webhook error: {ex.Message}");
        return StatusCode(500);
    }
}
```

# Рекомендации по безопасности Webhook

1️⃣ **Только HTTPS**  
Используйте исключительно HTTPS endpoint’ы.

2️⃣ **Проверка сертификатов**  
Сертификат должен быть действительным и выдан доверенным CA.

3️⃣ **Аутентификация запросов**  
Используйте:
- API keys
- OAuth 2.0
- Custom headers с токенами

4️⃣ **Проверка источника события**  
Проверяйте заголовок `aeg-subscription-name`.

5️⃣ **Rate Limiting**  
Ограничивайте частоту запросов для защиты от перегрузки.

6️⃣ **Валидация входных данных**  
Проверяйте структуру и содержимое `data`.

7️⃣ **Логирование**  
Логируйте события для аудита и диагностики.

---

# Troubleshooting

## Частые проблемы

| Проблема | Причина | Решение |
|-----------|----------|----------|
| Валидация не проходит | Неверная обработка validation event | Проверить логику возврата validationCode |
| Timeout ошибки | Обработка занимает > 30 секунд | Реализовать асинхронную обработку |
| Ошибка сертификата | Self-signed или просроченный сертификат | Использовать сертификат от доверенного CA |
| 401 Unauthorized | Неверная аутентификация | Проверить ключи или OAuth-конфигурацию |
| События не приходят | Слишком строгие фильтры | Проверить настройки фильтрации |
| Дублирующиеся события | Retry после таймаута | Реализовать идемпотентную обработку |

---

## Архитектурные рекомендации

- Быстро возвращайте HTTP 200
- Тяжёлую обработку выносите в очередь
- Реализуйте idempotency по `id`
- Настройте мониторинг DeliveryFailCount
- Проверяйте заголовок `aeg-event-type`

---

## Важно для AZ-204

Нужно помнить:

- Webhook должен отвечать в течение 30 секунд
- Используется модель at-least-once
- Повторная доставка возможна
- Validation handshake обязателен
- HTTPS — обязательное требование

Большинство проблем в вопросах по webhook связаны с валидацией, таймаутами или отсутствием идемпотентности.
### Validation Troubleshooting

**Check validation response:**
```bash
# Test validation manually
curl -X POST https://mywebhook.example.com/api/events \
  -H "Content-Type: application/json" \
  -d '[{
    "id": "test-id",
    "eventType": "Microsoft.EventGrid.SubscriptionValidationEvent",
    "data": {
      "validationCode": "test-code-123"
    }
  }]'

# Expected response:
# {"validationResponse":"test-code-123"}
```

### Delivery Troubleshooting

**Check subscription status:**
```bash
az eventgrid event-subscription show \
  --name mySubscription \
  --source-resource-id $TOPIC_ID \
  --query "{State:provisioningState, Endpoint:destination.endpointUrl}"
```

**Check delivery metrics:**
```bash
az monitor metrics list \
  --resource $TOPIC_ID \
  --metric "DeliveryFailedCount" \
  --start-time 2024-01-15T00:00:00Z
```

---

# Советы к экзамену AZ-204

## Ключевые моменты

1️⃣ **Валидация обязательна**  
Каждый webhook должен подтвердить владение endpoint.

2️⃣ **Два метода валидации**
- Синхронный (рекомендуется)
- Асинхронный (ручной)

3️⃣ **Таймаут 30 секунд**  
Webhook обязан ответить в течение 30 секунд.

4️⃣ **HTTP 200 OK**  
Требуется для успешной обработки события.

5️⃣ **Автоматическая валидация**  
Azure Functions, Logic Apps и Automation обрабатывают её автоматически.

6️⃣ **Окно 5 минут**  
Асинхронная валидация должна завершиться в течение 5 минут.

7️⃣ **HTTPS обязателен**  
Действительный TLS/SSL сертификат (без self-signed в production).

---

# Частые экзаменационные сценарии

### Сценарий 1
Webhook не проходит валидацию

- ✅ Реализовать синхронный handshake (вернуть `validationResponse`)
- ✅ Проверить формат ответа
- ❌ Не игнорировать validation event

---

### Сценарий 2
События завершаются по таймауту

- ✅ Реализовать асинхронную обработку (например, через очередь)
- ✅ Немедленно возвращать HTTP 200
- ❌ Не обрабатывать события синхронно, если это занимает > 30 секунд

---

### Сценарий 3
Нужно защитить webhook

- ✅ Использовать custom headers с API-ключами
- ✅ Использовать OAuth (Azure AD)
- ❌ Не полагаться на «скрытый URL»

---

### Сценарий 4
Legacy-система не поддерживает синхронную валидацию

- ✅ Использовать асинхронный handshake (ручной GET-запрос)
- ⏱️ Завершить валидацию в течение 5 минут

---

# Что обязательно помнить

- **Тип validation события**: `Microsoft.EventGrid.SubscriptionValidationEvent`
- **Поле ответа**: `validationResponse`
- **Таймаут**: 30 секунд
- **Окно асинхронной валидации**: 5 минут
- **Автоматическая обработка**: Functions, Logic Apps, Automation
- **Требуемый ответ**: HTTP 200 OK
- **Сертификат**: Действительный CA-signed
- **Повторы доставки**: Exponential backoff
- **Идемпотентность**: Обязательна

---

## Экзаменационный акцент

Если в вопросе:

- Подписка не активируется → проблема с validation
- Таймаут → обработка > 30 секунд
- Дубли → нет идемпотентности
- Безопасность → HTTPS + аутентификация

Webhook — одна из самых часто проверяемых тем по Event Grid в AZ-204.
### Quick Command Reference

```bash
# Create webhook subscription
az eventgrid event-subscription create \
  --name <name> \
  --source-resource-id <topic-id> \
  --endpoint https://<webhook-url>

# Add API key header
--delivery-attribute-mapping \
  X-API-Key static "<api-key>"

# Check subscription status
az eventgrid event-subscription show \
  --name <name> \
  --source-resource-id <topic-id> \
  --query "provisioningState"
```

---

# Итоги по Webhooks в Azure Event Grid

## Валидация endpoint

- **Синхронный handshake**  
  Немедленно вернуть `validationResponse` (рекомендуется)

- **Асинхронный handshake**  
  Выполнить ручной GET-запрос в течение 5 минут

- **Автоматическая обработка**  
  Azure Functions, Logic Apps и Automation выполняют валидацию автоматически

---

# Требования к Webhook

- HTTPS endpoint с действительным сертификатом
- Ответ в течение 30 секунд
- Возврат HTTP 200 OK
- Обработка validation события

---

# Best Practices

- ✅ Использовать асинхронную обработку для долгих операций
- ✅ Проектировать идемпотентные обработчики
- ✅ Реализовать аутентификацию (API keys, OAuth)
- ✅ Логировать все события
- ✅ Возвращать корректные HTTP-коды
    - 400 — ошибка валидации
    - 500 — временная ошибка
- ✅ Реализовать circuit breaker для зависимостей

---

# Безопасность

- Использовать только HTTPS
- Действительный TLS/SSL сертификат
- Аутентификация через custom headers или OAuth
- Проверка источника события
- Реализация rate limiting

---

## Ключевые выводы

- Validation обязательна для webhook
- Таймаут ответа — 30 секунд
- Возможна повторная доставка (at-least-once)
- Асинхронная обработка повышает надёжность
- Идемпотентность обязательна

---

## Экзаменационный акцент (AZ-204)

Нужно помнить:

- Синхронная валидация — предпочтительный метод
- Окно асинхронной валидации — 5 минут
- HTTP 200 — признак успешной обработки
- Дубли возможны
- HTTPS обязателен

Webhook — одна из самых часто проверяемых тем в разделе Event Grid.

| Куда доставлять событие       | Когда выбирать                                                                                                                                  | Сильные стороны                                                                          | Минусы / когда НЕ надо                                                                                                                   |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Webhook (HTTP endpoint)**   | Нужно уведомить **внешний сервис по HTTP** (SaaS/CRM/свой API), “push”, near real-time                                                          | Просто, универсально, подходит для интеграции с любым HTTP                               | Надёжность ограничена (есть ретраи, но нет очереди на стороне получателя), получатель должен быть доступен; нужно делать **idempotency** |
| **Azure Function**            | Нужна **кастомная обработка кода** (валидировать файл, записать в БД, дернуть API, сгенерить thumbnail)                                         | Минимум инфраструктуры, легко писать код, хорошо масштабируется, удобно для event-driven | Если нужна сложная оркестрация/человеческие approvals — лучше Logic App; для строгих гарантий доставки иногда добавляют очередь          |
| **Logic App**                 | Нужны **готовые коннекторы** и low-code: O365, Dynamics, SAP, ServiceNow, approvals, расписания, интеграционные пайплайны                       | Быстро собрать интеграцию без кода, богатые коннекторы, удобно для бизнес-процессов      | Может быть дороже/медленнее, сложнее дебажить как код; для high-throughput часто берут Functions                                         |
| **Service Bus (Queue/Topic)** | Нужно **максимально надёжно**: буферизация, **dead-letter**, контроль нагрузки, очереди, порядок/сессии, “работаем даже если потребитель лежит” | Enterprise-messaging, DLQ, back-pressure, конкурирующие консьюмеры, изоляция от пиков    | Требует consumer’а (Function/worker), чуть больше компонентов и настройки                                                                |
