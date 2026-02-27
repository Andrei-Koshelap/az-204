# Практическое задание: Маршрутизация пользовательских событий в Webhook Endpoint

## Обзор упражнения

В этом практическом задании вы выполните полный цикл работы с Azure Event Grid:

1. Создадите пользовательский Topic (Custom Event Grid Topic)
2. Развернёте webhook endpoint (Azure Function)
3. Создадите подписку на события с фильтрацией
4. Опубликуете пользовательские события
5. Проверите доставку событий
6. Протестируете фильтрацию и механизм повторных попыток (retry)

**Оценочное время выполнения:** 30–40 минут

---

## Предварительные требования

- Активная подписка Azure
- Установленный Azure CLI
- Базовые знания Azure Functions
- Понимание принципов event-driven архитектуры

---

## Шаг 1. Создание Custom Event Grid Topic

Создайте пользовательский Topic — это источник событий.

Что важно:

- Custom Topic используется для публикации собственных (application-generated) событий
- В отличие от системных тем (System Topics), вы полностью контролируете структуру событий
- Topic будет выступать в роли Publisher в архитектуре

Проверьте после создания:
- имя ресурса
- endpoint URL
- access key (понадобится для публикации событий)

---

## Шаг 2. Развёртывание Webhook Endpoint (Azure Function)

Создайте Azure Function с HTTP-триггером.

Важно:

- Event Grid выполняет проверку endpoint (validation handshake)
- Функция должна корректно обрабатывать событие валидации
- Endpoint должен быть публично доступен

После деплоя получите URL функции — он понадобится при создании подписки.

---

## Шаг 3. Создание Event Subscription с фильтрацией

Создайте подписку на события для вашего Custom Topic.

Настройте:

- Тип события (если требуется)
- Фильтрацию по `subject`
- При необходимости — Advanced Filtering

Помните:

- Фильтры применяются в порядке: Type → Subject → Advanced
- Все условия работают по логике AND
- Максимум 25 advanced-фильтров

На экзамене AZ-204 часто проверяется понимание именно этого шага.

---

## Шаг 4. Публикация пользовательских событий

Опубликуйте тестовые события в Custom Topic.

Убедитесь, что:

- события соответствуют схеме Event Grid
- указаны корректные `eventType`, `subject`, `data`
- используются правильные ключи доступа

Можно опубликовать несколько событий, чтобы протестировать фильтрацию.

---

## Шаг 5. Проверка доставки событий

Проверьте:

- Выполняется ли Azure Function
- Получает ли она только отфильтрованные события
- Отображаются ли события в логах

Если события не доставляются:

- проверьте фильтры
- проверьте endpoint validation
- проверьте статус подписки

---

## Шаг 6. Тестирование фильтрации и механизма повторных попыток

### Проверка фильтрации

- Опубликуйте событие, которое должно пройти фильтр
- Опубликуйте событие, которое не должно пройти

Убедитесь, что endpoint получает только ожидаемые события.

---

### Проверка retry-механизма

Event Grid автоматически повторяет доставку при ошибках.

Протестируйте:

- временно возвращайте ошибку из webhook
- проверьте, что Event Grid выполняет повторную попытку

Важно помнить:

- Event Grid использует экспоненциальную стратегию повторов
- Существует срок хранения события (retry window)
- После исчерпания попыток событие может быть отправлено в dead-letter endpoint (если настроен)

---

## Что это упражнение закрепляет

- Понимание Custom Topics
- Настройку Event Subscription
- Применение фильтров
- Обработку webhook validation
- Понимание retry и delivery semantics

---

## Связь с экзаменом AZ-204

Это упражнение охватывает:

- Создание и настройку Event Grid
- Реализацию webhook endpoint
- Конфигурацию фильтрации
- Обработку ошибок доставки

Если вы понимаете каждый этап этого сценария — тема Event Grid для AZ-204 у вас закрыта на хорошем уровне.

---

## Architecture Diagram

```
┌─────────────────────┐
│  Your Application   │
│  (Event Publisher)  │
└──────────┬──────────┘
           │ POST events
           ▼
┌─────────────────────────┐
│  Azure Event Grid Topic │
│   (Custom Topic)        │
└──────────┬──────────────┘
           │ Filter & Route
           ▼
┌─────────────────────────┐
│  Event Subscription     │
│  (Filter Configuration) │
└──────────┬──────────────┘
           │ Push events
           ▼
┌─────────────────────────┐
│  Azure Function         │
│  (Webhook Endpoint)     │
└─────────────────────────┘
```

---

## Part 1: Set Up Environment

### Step 1.1: Define Variables

```bash
# Set variables
RESOURCE_GROUP="rg-eventgrid-lab"
LOCATION="eastus"
TOPIC_NAME="topic-orders-$(openssl rand -hex 4)"
FUNCTION_APP_NAME="func-eventhandler-$(openssl rand -hex 4)"
STORAGE_ACCOUNT="steventgrid$(openssl rand -hex 4)"
SUBSCRIPTION_NAME="order-subscription"

echo "Resource Group: $RESOURCE_GROUP"
echo "Topic Name: $TOPIC_NAME"
echo "Function App: $FUNCTION_APP_NAME"
```

### Step 1.2: Create Resource Group

```bash
# Create resource group
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION

echo "✓ Resource group created"
```

---

## Part 2: Create Event Grid Custom Topic

### Step 2.1: Create Topic

```bash
# Create custom topic
az eventgrid topic create \
  --name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION

echo "✓ Event Grid topic created"
```

### Step 2.2: Get Topic Endpoint and Keys

```bash
# Get topic endpoint
TOPIC_ENDPOINT=$(az eventgrid topic show \
  --name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --query "endpoint" \
  --output tsv)

# Get topic key
TOPIC_KEY=$(az eventgrid topic key list \
  --name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --query "key1" \
  --output tsv)

# Get topic resource ID
TOPIC_ID=$(az eventgrid topic show \
  --name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --query "id" \
  --output tsv)

echo "Topic Endpoint: $TOPIC_ENDPOINT"
echo "Topic Key: $TOPIC_KEY"
echo "Topic ID: $TOPIC_ID"
```

---

## Part 3: Create Webhook Endpoint (Azure Function)

### Step 3.1: Create Storage Account

```bash
# Create storage account for Function App
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard_LRS

echo "✓ Storage account created"
```

### Step 3.2: Create Function App

```bash
# Create Function App (Linux, .NET 8)
az functionapp create \
  --name $FUNCTION_APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --storage-account $STORAGE_ACCOUNT \
  --consumption-plan-location $LOCATION \
  --runtime dotnet-isolated \
  --runtime-version 8 \
  --functions-version 4 \
  --os-type Linux

echo "✓ Function App created"
```

### Step 3.3: Get Function App URL

```bash
# Get Function App default hostname
FUNCTION_APP_URL="https://${FUNCTION_APP_NAME}.azurewebsites.net"

echo "Function App URL: $FUNCTION_APP_URL"
```

### Step 3.4: Create Event Handler Function

Create a file named `EventGridTriggerFunction.cs`:

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Extensions.Logging;
using Azure.Messaging.EventGrid;
using Azure.Messaging.EventGrid.SystemEvents;

namespace EventGridFunctions
{
    public class EventGridTriggerFunction
    {
        private readonly ILogger<EventGridTriggerFunction> _logger;

        public EventGridTriggerFunction(ILogger<EventGridTriggerFunction> logger)
        {
            _logger = logger;
        }

        [Function("EventGridTrigger")]
        public void Run([EventGridTrigger] EventGridEvent eventGridEvent)
        {
            _logger.LogInformation($"=== Event Received ===");
            _logger.LogInformation($"Event ID: {eventGridEvent.Id}");
            _logger.LogInformation($"Event Type: {eventGridEvent.EventType}");
            _logger.LogInformation($"Event Subject: {eventGridEvent.Subject}");
            _logger.LogInformation($"Event Time: {eventGridEvent.EventTime}");
            _logger.LogInformation($"Event Data: {eventGridEvent.Data}");
            
            // Handle different event types
            switch (eventGridEvent.EventType)
            {
                case "MyApp.Orders.OrderCreated":
                    HandleOrderCreated(eventGridEvent);
                    break;
                case "MyApp.Orders.OrderShipped":
                    HandleOrderShipped(eventGridEvent);
                    break;
                case "MyApp.Orders.OrderCancelled":
                    HandleOrderCancelled(eventGridEvent);
                    break;
                default:
                    _logger.LogInformation($"Unknown event type: {eventGridEvent.EventType}");
                    break;
            }
        }

        private void HandleOrderCreated(EventGridEvent evt)
        {
            _logger.LogInformation("Processing OrderCreated event...");
            // Your business logic here
            var data = evt.Data.ToObjectFromJson<OrderCreatedData>();
            _logger.LogInformation($"Order ID: {data.OrderId}");
            _logger.LogInformation($"Customer: {data.CustomerEmail}");
            _logger.LogInformation($"Amount: ${data.Amount}");
        }

        private void HandleOrderShipped(EventGridEvent evt)
        {
            _logger.LogInformation("Processing OrderShipped event...");
            // Your business logic here
        }

        private void HandleOrderCancelled(EventGridEvent evt)
        {
            _logger.LogInformation("Processing OrderCancelled event...");
            // Your business logic here
        }
    }

    // Data models
    public class OrderCreatedData
    {
        public string OrderId { get; set; }
        public string CustomerEmail { get; set; }
        public decimal Amount { get; set; }
        public string Status { get; set; }
    }
}
```

**Alternative: HTTP Webhook Function (if not using Event Grid trigger)**

```csharp
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Http;
using Microsoft.Extensions.Logging;
using System.Net;
using System.Text.Json;
using Azure.Messaging.EventGrid;
using Azure.Messaging.EventGrid.SystemEvents;

namespace EventGridFunctions
{
    public class WebhookFunction
    {
        private readonly ILogger<WebhookFunction> _logger;

        public WebhookFunction(ILogger<WebhookFunction> logger)
        {
            _logger = logger;
        }

        [Function("Webhook")]
        public async Task<HttpResponseData> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", "options")] HttpRequestData req)
        {
            // Handle OPTIONS request
            if (req.Method == "OPTIONS")
            {
                var optionsResponse = req.CreateResponse(HttpStatusCode.OK);
                return optionsResponse;
            }

            // Read request body
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            
            // Parse Event Grid events
            EventGridEvent[] events = EventGridEvent.ParseMany(BinaryData.FromString(requestBody));

            foreach (EventGridEvent eventGridEvent in events)
            {
                // Handle validation event
                if (eventGridEvent.EventType == "Microsoft.EventGrid.SubscriptionValidationEvent")
                {
                    var validationData = eventGridEvent.Data.ToObjectFromJson<SubscriptionValidationEventData>();
                    var validationResponse = new { validationResponse = validationData.ValidationCode };
                    
                    var response = req.CreateResponse(HttpStatusCode.OK);
                    await response.WriteAsJsonAsync(validationResponse);
                    return response;
                }

                // Log event details
                _logger.LogInformation($"Event ID: {eventGridEvent.Id}");
                _logger.LogInformation($"Event Type: {eventGridEvent.EventType}");
                _logger.LogInformation($"Subject: {eventGridEvent.Subject}");
                _logger.LogInformation($"Data: {eventGridEvent.Data}");
                
                // Process event
                await ProcessEvent(eventGridEvent);
            }

            var successResponse = req.CreateResponse(HttpStatusCode.OK);
            return successResponse;
        }

        private async Task ProcessEvent(EventGridEvent evt)
        {
            // Your event processing logic
            _logger.LogInformation($"Processing event: {evt.EventType}");
            await Task.CompletedTask;
        }
    }
}
```

### Step 3.5: Deploy Function

```bash
# Deploy function (if using local development)
# Navigate to function project directory
cd EventGridFunctions

# Publish to Azure
func azure functionapp publish $FUNCTION_APP_NAME

echo "✓ Function deployed"
```

---

## Part 4: Create Event Subscription

### Step 4.1: Get Function Endpoint

**For Event Grid Trigger:**
```bash
# Get system key for Event Grid trigger
FUNCTION_KEY=$(az functionapp keys list \
  --name $FUNCTION_APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --query "systemKeys.eventgrid_extension" \
  --output tsv)

FUNCTION_ENDPOINT="${FUNCTION_APP_URL}/runtime/webhooks/EventGrid?functionName=EventGridTrigger&code=${FUNCTION_KEY}"
```

**For HTTP Trigger Webhook:**
```bash
# Get function key
FUNCTION_KEY=$(az functionapp function keys list \
  --name $FUNCTION_APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --function-name Webhook \
  --query "default" \
  --output tsv)

FUNCTION_ENDPOINT="${FUNCTION_APP_URL}/api/Webhook?code=${FUNCTION_KEY}"
```

### Step 4.2: Create Subscription with Filters

```bash
# Create event subscription with filters
az eventgrid event-subscription create \
  --name $SUBSCRIPTION_NAME \
  --source-resource-id $TOPIC_ID \
  --endpoint $FUNCTION_ENDPOINT \
  --included-event-types \
    MyApp.Orders.OrderCreated \
    MyApp.Orders.OrderShipped \
    MyApp.Orders.OrderCancelled \
  --subject-begins-with "/orders/" \
  --max-delivery-attempts 5 \
  --event-ttl 60

echo "✓ Event subscription created"
```

### Step 4.3: Verify Subscription

```bash
# Check subscription status
az eventgrid event-subscription show \
  --name $SUBSCRIPTION_NAME \
  --source-resource-id $TOPIC_ID \
  --query "{State:provisioningState, Endpoint:destination.endpointBaseUrl}"
```

---

## Part 5: Publish Custom Events

### Step 5.1: Publish Single Event

```bash
# Publish OrderCreated event
az eventgrid event publish \
  --topic-name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --events '[
    {
      "id": "order-001",
      "eventType": "MyApp.Orders.OrderCreated",
      "subject": "/orders/region/west/order/001",
      "eventTime": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'",
      "data": {
        "orderId": "ORD-2024-001",
        "customerEmail": "customer@example.com",
        "amount": 149.99,
        "status": "Pending",
        "items": [
          {
            "productId": "PROD-001",
            "quantity": 2,
            "price": 74.99
          }
        ]
      },
      "dataVersion": "1.0"
    }
  ]'

echo "✓ Event published"
```

### Step 5.2: Publish Multiple Events (Batch)

```bash
# Publish multiple events
az eventgrid event publish \
  --topic-name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --events '[
    {
      "id": "order-002",
      "eventType": "MyApp.Orders.OrderCreated",
      "subject": "/orders/region/east/order/002",
      "eventTime": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'",
      "data": {
        "orderId": "ORD-2024-002",
        "customerEmail": "alice@example.com",
        "amount": 299.99,
        "status": "Pending"
      },
      "dataVersion": "1.0"
    },
    {
      "id": "order-003",
      "eventType": "MyApp.Orders.OrderShipped",
      "subject": "/orders/region/west/order/003",
      "eventTime": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'",
      "data": {
        "orderId": "ORD-2024-003",
        "trackingNumber": "TRACK-12345",
        "carrier": "UPS"
      },
      "dataVersion": "1.0"
    },
    {
      "id": "order-004",
      "eventType": "MyApp.Orders.OrderCancelled",
      "subject": "/orders/region/east/order/004",
      "eventTime": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'",
      "data": {
        "orderId": "ORD-2024-004",
        "reason": "Customer request",
        "refundAmount": 199.99
      },
      "dataVersion": "1.0"
    }
  ]'

echo "✓ Batch events published"
```

### Step 5.3: Publish with Python Script

Create `publish_event.py`:

```python
from azure.eventgrid import EventGridPublisherClient
from azure.core.credentials import AzureKeyCredential
from datetime import datetime
import os

# Configuration
endpoint = os.environ['TOPIC_ENDPOINT']
key = os.environ['TOPIC_KEY']

# Create client
credential = AzureKeyCredential(key)
client = EventGridPublisherClient(endpoint, credential)

# Create event
event = {
    "id": "order-005",
    "eventType": "MyApp.Orders.OrderCreated",
    "subject": "/orders/region/west/order/005",
    "eventTime": datetime.utcnow().isoformat() + "Z",
    "data": {
        "orderId": "ORD-2024-005",
        "customerEmail": "bob@example.com",
        "amount": 499.99,
        "status": "Pending"
    },
    "dataVersion": "1.0"
}

# Publish event
client.send(event)
print("✓ Event published from Python")
```

Run the script:
```bash
export TOPIC_ENDPOINT=$TOPIC_ENDPOINT
export TOPIC_KEY=$TOPIC_KEY
python publish_event.py
```

---

## Part 6: Verify Event Delivery

### Step 6.1: Check Function Logs

```bash
# Stream function logs
az functionapp log tail \
  --name $FUNCTION_APP_NAME \
  --resource-group $RESOURCE_GROUP
```

**Expected Output:**
```
2024-01-15T14:30:00.123 [Information] === Event Received ===
2024-01-15T14:30:00.124 [Information] Event ID: order-001
2024-01-15T14:30:00.125 [Information] Event Type: MyApp.Orders.OrderCreated
2024-01-15T14:30:00.126 [Information] Event Subject: /orders/region/west/order/001
2024-01-15T14:30:00.127 [Information] Processing OrderCreated event...
2024-01-15T14:30:00.128 [Information] Order ID: ORD-2024-001
2024-01-15T14:30:00.129 [Information] Customer: customer@example.com
2024-01-15T14:30:00.130 [Information] Amount: $149.99
```

### Step 6.2: Check Event Grid Metrics

```bash
# Check published events
az monitor metrics list \
  --resource $TOPIC_ID \
  --metric "PublishSuccessCount" \
  --start-time $(date -u -d '1 hour ago' +"%Y-%m-%dT%H:%M:%SZ") \
  --interval PT1M

# Check matched events
az monitor metrics list \
  --resource $TOPIC_ID \
  --metric "MatchedEventCount" \
  --start-time $(date -u -d '1 hour ago' +"%Y-%m-%dT%H:%M:%SZ") \
  --interval PT1M

# Check delivered events
az monitor metrics list \
  --resource $TOPIC_ID \
  --metric "DeliverySuccessCount" \
  --start-time $(date -u -d '1 hour ago' +"%Y-%m-%dT%H:%M:%SZ") \
  --interval PT1M

# Check failed deliveries
az monitor metrics list \
  --resource $TOPIC_ID \
  --metric "DeliveryFailedCount" \
  --start-time $(date -u -d '1 hour ago' +"%Y-%m-%dT%H:%M:%SZ") \
  --interval PT1M
```

---

## Part 7: Test Event Filtering

### Step 7.1: Test Subject Filter (Should Match)

```bash
# Event with matching subject
az eventgrid event publish \
  --topic-name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --events '[{
    "id": "test-match",
    "eventType": "MyApp.Orders.OrderCreated",
    "subject": "/orders/region/south/order/999",
    "eventTime": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'",
    "data": { "test": "This should be delivered" },
    "dataVersion": "1.0"
  }]'

# Check function logs - should see event
```

### Step 7.2: Test Subject Filter (Should NOT Match)

```bash
# Event with non-matching subject
az eventgrid event publish \
  --topic-name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --events '[{
    "id": "test-no-match",
    "eventType": "MyApp.Orders.OrderCreated",
    "subject": "/products/product/123",
    "eventTime": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'",
    "data": { "test": "This should NOT be delivered" },
    "dataVersion": "1.0"
  }]'

# Check function logs - should NOT see event
# Event filtered out by subject filter
```

### Step 7.3: Test Event Type Filter (Should NOT Match)

```bash
# Event with non-matching type
az eventgrid event publish \
  --topic-name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --events '[{
    "id": "test-wrong-type",
    "eventType": "MyApp.Products.ProductCreated",
    "subject": "/orders/region/west/order/888",
    "eventTime": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'",
    "data": { "test": "Wrong event type" },
    "dataVersion": "1.0"
  }]'

# Check function logs - should NOT see event
# Event filtered out by event type filter
```

---

## Part 8: Configure Advanced Features

### Step 8.1: Add Dead-Letter Storage

```bash
# Create storage container for dead-letter
DEADLETTER_CONTAINER="eventgrid-deadletter"

az storage container create \
  --name $DEADLETTER_CONTAINER \
  --account-name $STORAGE_ACCOUNT

# Get storage account resource ID
STORAGE_ID=$(az storage account show \
  --name $STORAGE_ACCOUNT \
  --resource-group $RESOURCE_GROUP \
  --query "id" \
  --output tsv)

# Update subscription with dead-letter
az eventgrid event-subscription update \
  --name $SUBSCRIPTION_NAME \
  --source-resource-id $TOPIC_ID \
  --deadletter-endpoint "${STORAGE_ID}/blobServices/default/containers/${DEADLETTER_CONTAINER}"

echo "✓ Dead-letter configured"
```

### Step 8.2: Add Advanced Filtering

```bash
# Create new subscription with advanced filters
az eventgrid event-subscription create \
  --name "high-value-orders" \
  --source-resource-id $TOPIC_ID \
  --endpoint $FUNCTION_ENDPOINT \
  --included-event-types MyApp.Orders.OrderCreated \
  --advanced-filter data.amount NumberGreaterThan 1000 \
  --advanced-filter data.status StringIn Pending Confirmed

echo "✓ Advanced filtering configured"
```

### Step 8.3: Enable Output Batching

```bash
# Update subscription with batching
az eventgrid event-subscription update \
  --name $SUBSCRIPTION_NAME \
  --source-resource-id $TOPIC_ID \
  --max-events-per-batch 10 \
  --preferred-batch-size-in-kilobytes 64

echo "✓ Output batching enabled"
```

---

## Part 9: Monitor and Troubleshoot

### Step 9.1: View Subscription Details

```bash
# Get full subscription details
az eventgrid event-subscription show \
  --name $SUBSCRIPTION_NAME \
  --source-resource-id $TOPIC_ID
```

### Step 9.2: Test Retry Mechanism

**Simulate failed delivery:**

1. Stop the Function App:
```bash
az functionapp stop --name $FUNCTION_APP_NAME --resource-group $RESOURCE_GROUP
```

2. Publish event:
```bash
az eventgrid event publish \
  --topic-name $TOPIC_NAME \
  --resource-group $RESOURCE_GROUP \
  --events '[{
    "id": "retry-test",
    "eventType": "MyApp.Orders.OrderCreated",
    "subject": "/orders/test",
    "eventTime": "'$(date -u +"%Y-%m-%dT%H:%M:%SZ")'",
    "data": { "test": "retry" },
    "dataVersion": "1.0"
  }]'
```

3. Check delivery attempts:
```bash
# Wait a few minutes, check metrics
az monitor metrics list \
  --resource $TOPIC_ID \
  --metric "DeliveryFailedCount" \
  --start-time $(date -u -d '10 minutes ago' +"%Y-%m-%dT%H:%M:%SZ")
```

4. Restart Function App:
```bash
az functionapp start --name $FUNCTION_APP_NAME --resource-group $RESOURCE_GROUP
```

5. Event Grid will retry delivery automatically

---

## Part 10: Clean Up Resources

```bash
# Delete entire resource group
az group delete \
  --name $RESOURCE_GROUP \
  --yes \
  --no-wait

echo "✓ Resources deleted"
```

---

## Ключевые выводы (Key Takeaways)

1. **Custom Topics** — создан пользовательский Event Grid Topic для публикации собственных событий
2. **Webhook Endpoint** — развернута Azure Function в роли обработчика событий
3. **Event Subscription** — настроена подписка с фильтрацией для маршрутизации событий
4. **Публикация событий** — отправка событий через Azure CLI и SDK
5. **Фильтрация** — протестирована фильтрация по типу события и subject
6. **Дополнительные возможности** — настроены dead-letter и batching
7. **Мониторинг** — проверена доставка событий через метрики и логи

---

## Руководство по устранению проблем (Troubleshooting Guide)

| Проблема | Возможная причина | Решение |
|-----------|------------------|----------|
| Ошибка валидации (Validation failing) | Функция не отвечает | Проверить логи функции, убедиться в использовании HTTPS |
| События не доставляются | Слишком строгая фильтрация | Проверить конфигурацию фильтров |
| Ошибки таймаута | Обработка функции > 30 секунд | Реализовать асинхронную обработку |
| Ошибка 401 | Неверный ключ функции | Сгенерировать новый ключ и обновить endpoint |
| События попадают в dead-letter | Постоянные ошибки доставки | Проверить контейнер dead-letter и проанализировать причину |

---

## Дополнительные задания (Additional Challenges)

**Задание 1:**  
Добавить аутентификацию webhook через пользовательские HTTP-заголовки

**Задание 2:**  
Реализовать идемпотентную обработку событий (отслеживать `eventId`)

**Задание 3:**  
Создать несколько подписок с маршрутизацией в разные функции в зависимости от типа события

**Задание 4:**  
Реализовать паттерн Circuit Breaker в обработчике событий

**Задание 5:**  
Настроить автоматическую повторную обработку событий из dead-letter

---

## Советы для экзамена AZ-204

### Важно помнить:

- Custom Topics создаются явно (не автоматически)
- Webhook требует обязательной endpoint validation
- Фильтрация снижает лишнюю доставку событий
- Dead-letter требует контейнер Azure Storage Blob
- Политика повторных попыток по умолчанию:
    - до 30 попыток
    - TTL — 24 часа
- Максимальный размер события: 1 МБ
- Биллинг рассчитывается блоками по 64 КБ

---

### Типовые экзаменационные сценарии:

- Создание Custom Topic для событий приложения
- Настройка подписки с фильтрацией
- Реализация webhook validation
- Настройка dead-letter хранения
- Диагностика проблем доставки событий

---

## Итог

Вы успешно:

✅ Создали пользовательский Event Grid Topic  
✅ Развернули webhook endpoint (Azure Function)  
✅ Настроили подписку с фильтрацией  
✅ Опубликовали пользовательские события  
✅ Проверили доставку событий  
✅ Настроили расширенные возможности (dead-letter, batching)  
✅ Проанализировали метрики и логи  
✅ Протестировали механизм повторных попыток

---

## Следующие шаги

- Изучить интеграцию Event Grid с Azure Storage и IoT Hub
- Реализовать более сложные сценарии расширенной фильтрации
- Построить production-ready event-driven архитектуру
- Изучить Event Grid Domains для multi-tenant решений

Если вы уверенно понимаете все эти пункты — тема Event Grid для AZ-204 у вас проработана на хорошем уровне.