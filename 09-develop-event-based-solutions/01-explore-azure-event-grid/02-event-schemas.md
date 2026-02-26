# Схемы событий в Azure Event Grid

## Обзор

Azure Event Grid поддерживает два формата схем событий:

1. **Event Grid Schema** — нативный формат Azure
2. **CloudEvents Schema** — отраслевой стандарт (спецификация v1.0)

Обе схемы описывают:

- **что произошло**
- **когда произошло**
- **где произошло**
- дополнительные данные события

---

# CloudEvents v1.0 (Рекомендуемый формат)

**CloudEvents** — это открытая спецификация для описания событий в унифицированном формате.

Она обеспечивает совместимость между различными облаками, сервисами и платформами.

---

## Почему стоит использовать CloudEvents?

- ✅ **Отраслевой стандарт**  
  Поддерживается CNCF (Cloud Native Computing Foundation)

- ✅ **Интероперабельность**  
  Работает в разных облаках и системах

- ✅ **Перспективность**  
  Активно развивается и поддерживается

- ✅ **Поддержка инструментов**  
  Широкая экосистема SDK и библиотек

- ✅ **Рекомендован Microsoft**  
  Предпочтительный формат для новых приложений

---

## Архитектурный смысл

Использование CloudEvents:

- упрощает миграцию между облаками;
- снижает vendor lock-in;
- облегчает интеграцию с внешними системами;
- стандартизирует структуру событий.

---

## Когда выбирать CloudEvents

- При разработке новых решений
- При интеграции с несколькими облаками
- В гибридных и multi-cloud архитектурах
- Если требуется соответствие индустриальным стандартам

---

## Важно для AZ-204

На экзамене важно помнить:

- CloudEvents v1.0 — рекомендуемый формат
- Event Grid поддерживает оба формата
- CloudEvents обеспечивает лучшую совместимость
- Microsoft рекомендует использовать CloudEvents для новых решений

### CloudEvents Schema Structure

```json
{
  "specversion": "1.0",
  "type": "com.example.someevent",
  "source": "/mycontext/subcontext",
  "subject": "larger-context/specific-resource",
  "id": "A234-1234-1234",
  "time": "2024-01-15T13:45:30.0000000Z",
  "datacontenttype": "application/json",
  "dataschema": "https://mycompany.com/schemas/v1",
  "data": {
    "appinfoA": "abc",
    "appinfoB": 123,
    "appinfoC": true
  }
}
```

## Свойства CloudEvents

| Свойство | Тип | Обязательное | Описание |
|-----------|------|--------------|-----------|
| `specversion` | string | ✅ Да | Версия спецификации CloudEvents (всегда `"1.0"`) |
| `type` | string | ✅ Да | Тип события (например, `"com.example.object.created"`) |
| `source` | URI | ✅ Да | Контекст, в котором произошло событие |
| `id` | string | ✅ Да | Уникальный идентификатор события |
| `time` | timestamp | ❌ Нет | Время события (формат RFC 3339) |
| `subject` | string | ❌ Нет | Конкретный объект события в контексте source |
| `datacontenttype` | string | ❌ Нет | Тип содержимого данных (например, `"application/json"`) |
| `dataschema` | URI | ❌ Нет | Схема, которой соответствует поле data |
| `data` | object | ❌ Нет | Payload события (бизнес-данные) |

---

## Подробности по ключевым свойствам

### `specversion`

- **Назначение**: Указывает версию спецификации CloudEvents
- **Значение**: Всегда `"1.0"`
- **Пример**:  
  `"specversion": "1.0"`

Это поле обязательно и позволяет обработчику понять структуру события.

---

### `type`

- **Назначение**: Описывает тип события
- **Рекомендуемый формат**: Reverse domain notation

Примеры:

- `com.contoso.order.created`
- `com.example.user.deleted`
- `com.myapp.payment.completed`

---

### Архитектурный смысл

`type` используется для:

- маршрутизации событий;
- фильтрации на уровне Event Subscription;
- определения логики обработки.

Хорошая практика — использовать иерархическую структуру имени типа события.

---

## Важно для AZ-204

Запомните:

- `specversion`, `type`, `source`, `id` — обязательные поля.
- CloudEvents v1.0 — рекомендуемый стандарт.
- `type` используется для фильтрации.
- `id` должен быть уникальным (важно для идемпотентности).
  ```
  "com.microsoft.storage.blobcreated"
  "com.example.orders.ordercreated"
  "io.github.pull_request.opened"
  ```

#### source
- **Purpose**: Identifies the context where the event occurred
- **Format**: URI reference
- **Examples**:
  ```
  "/subscriptions/{sub-id}/resourceGroups/rg/providers/Microsoft.Storage/storageAccounts/myaccount"
  "/tenants/my-tenant/applications/my-app"
  "https://api.example.com/v1/orders"
  ```

#### id
- **Purpose**: Uniquely identifies the event
- **Requirements**: 
  - Must be unique within the scope of the source
  - Should be immutable
- **Examples**:
  ```
  "A234-1234-1234"
  "550e8400-e29b-41d4-a716-446655440000"
  "event-2024-01-15-001"
  ```

#### time
- **Purpose**: Timestamp when event occurred
- **Format**: RFC 3339 (ISO 8601)
- **Example**: `"2024-01-15T13:45:30.0000000Z"`

#### subject
- **Purpose**: Describes the subject of the event in the context of the source
- **Use**: Provides filtering capability
- **Examples**:
  ```
  "/blobServices/default/containers/images/blobs/photo.jpg"
  "/orders/12345"
  "/users/user@example.com/profile"
  ```

#### datacontenttype
- **Purpose**: Describes the content type of the data value
- **Common Values**: `application/json`, `application/xml`, `text/plain`
- **Default**: `application/json` if omitted

#### data
- **Purpose**: Contains event-specific information
- **Type**: Any JSON-serializable value
- **Size Limit**: Maximum 1 MB for entire event

### Complete CloudEvents Examples

#### Example 1: Azure Storage Blob Created

```json
{
  "specversion": "1.0",
  "type": "Microsoft.Storage.BlobCreated",
  "source": "/subscriptions/abc123/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage",
  "subject": "/blobServices/default/containers/images/blobs/vacation-photo.jpg",
  "id": "9aeb0fdf-c01e-0131-0922-9eb54906e209",
  "time": "2024-01-15T13:45:30.0000000Z",
  "datacontenttype": "application/json",
  "data": {
    "api": "PutBlob",
    "clientRequestId": "6d79dbfb-0e37-4fc4-981f-442c9ca65760",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "eTag": "0x8D4BCC2E4835CD0",
    "contentType": "image/jpeg",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://mystorage.blob.core.windows.net/images/vacation-photo.jpg",
    "sequencer": "00000000000004420000000000028963",
    "storageDiagnostics": {
      "batchId": "b68529f3-68cd-4744-baa4-3c0498ec19f0"
    }
  }
}
```

#### Example 2: Custom Application Event

```json
{
  "specversion": "1.0",
  "type": "com.example.ecommerce.orders.OrderCreated",
  "source": "https://api.example.com/v1/orders",
  "subject": "/orders/ORD-2024-001",
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "time": "2024-01-15T14:30:00Z",
  "datacontenttype": "application/json",
  "dataschema": "https://api.example.com/schemas/order/v2",
  "data": {
    "orderId": "ORD-2024-001",
    "customerId": "CUST-12345",
    "orderDate": "2024-01-15T14:30:00Z",
    "totalAmount": 1599.99,
    "currency": "USD",
    "items": [
      {
        "productId": "PROD-001",
        "quantity": 2,
        "price": 799.99
      }
    ],
    "shippingAddress": {
      "street": "123 Main St",
      "city": "Seattle",
      "state": "WA",
      "zipCode": "98101",
      "country": "USA"
    }
  }
}
```

#### Example 3: IoT Device Telemetry

```json
{
  "specversion": "1.0",
  "type": "io.example.iot.sensors.TemperatureReading",
  "source": "/devices/sensor-001",
  "subject": "/buildings/building-a/floor-3/room-301",
  "id": "temp-reading-20240115-143500",
  "time": "2024-01-15T14:35:00Z",
  "datacontenttype": "application/json",
  "data": {
    "deviceId": "sensor-001",
    "temperature": 22.5,
    "humidity": 45.2,
    "unit": "celsius",
    "batteryLevel": 87,
    "location": {
      "building": "building-a",
      "floor": 3,
      "room": "301"
    }
  }
}
```

---

## Event Grid Schema (Legacy)

The **Event Grid Schema** is Azure's native format, supported for backward compatibility. New applications should use CloudEvents.

### Event Grid Schema Structure

```json
{
  "topic": "/subscriptions/{subscription-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage",
  "subject": "/blobServices/default/containers/images/blobs/photo.jpg",
  "eventType": "Microsoft.Storage.BlobCreated",
  "id": "9aeb0fdf-c01e-0131-0922-9eb54906e209",
  "eventTime": "2024-01-15T13:45:30.0000000Z",
  "data": {
    "api": "PutBlob",
    "contentType": "image/jpeg",
    "contentLength": 524288,
    "blobType": "BlockBlob",
    "url": "https://mystorage.blob.core.windows.net/images/photo.jpg"
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}
```


---

### Архитектурный смысл

`topic` позволяет:

- однозначно определить источник события;
- фильтровать события по ресурсу;
- отслеживать происхождение события.

---

### Отличие от CloudEvents

- В Event Grid Schema используется `eventType` вместо `type`.
- Поле `topic` является частью нативной схемы Azure.
- Все поля, кроме `metadataVersion`, обязательны.

---

## Важно для AZ-204

Нужно понимать:

- Разницу между CloudEvents и Event Grid Schema.
- В Event Grid Schema все ключевые поля обязательны.
- `dataVersion` определяет версию структуры payload.
- `topic` автоматически заполняется для системных событий.

Экзамен может проверять выбор схемы или различие между `type` и `eventType`.
```
  "/subscriptions/abc123/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage"
  ```

#### subject
- **Purpose**: Publisher-defined path to the event subject
- **Use**: Enables filtering by subject patterns
- **Best Practice**: Use hierarchical structure
- **Examples**:
  ```
  "/blobServices/default/containers/images/blobs/photo.jpg"
  "/orders/region/west/store/101/order/12345"
  ```

#### eventType
- **Purpose**: Identifies the type of event
- **Format**: Dot-notation recommended
- **Azure Examples**:
  ```
  "Microsoft.Storage.BlobCreated"
  "Microsoft.Resources.ResourceWriteSuccess"
  "Microsoft.EventHub.CaptureFileCreated"
  ```
- **Custom Examples**:
  ```
  "MyApp.Orders.OrderCreated"
  "MyApp.Inventory.StockLevelLow"
  ```

#### id
- **Purpose**: Unique identifier for the event
- **Uniqueness**: Must be unique per event
- **Use**: Implement idempotency in handlers

#### eventTime
- **Purpose**: Time the event was generated
- **Format**: ISO 8601 datetime string
- **Timezone**: Always UTC
- **Example**: `"2024-01-15T13:45:30.0000000Z"`

#### data
- **Purpose**: Contains event-specific payload
- **Type**: Object (JSON)
- **Schema**: Defined by event source
- **Versioning**: Tracked via dataVersion

#### dataVersion
- **Purpose**: Schema version of the data object
- **Set By**: Publisher
- **Use**: Handle schema evolution
- **Example**: `"1.0"`, `"2.1"`

### Complete Event Grid Examples

#### Example 1: Storage Blob Deleted

```json
{
  "topic": "/subscriptions/abc123/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage",
  "subject": "/blobServices/default/containers/archive/blobs/old-file.txt",
  "eventType": "Microsoft.Storage.BlobDeleted",
  "id": "7c5d6de5-eb70-4de2-b788-c8e3c9862eaa",
  "eventTime": "2024-01-15T15:20:00.0000000Z",
  "data": {
    "api": "DeleteBlob",
    "requestId": "831e1650-001e-001b-66ab-eeb76e000000",
    "contentType": "text/plain",
    "blobType": "BlockBlob",
    "url": "https://mystorage.blob.core.windows.net/archive/old-file.txt",
    "sequencer": "00000000000004420000000000028963"
  },
  "dataVersion": "1.0",
  "metadataVersion": "1"
}
```

#### Example 2: Resource Manager Deployment

```json
{
  "topic": "/subscriptions/abc123/resourceGroups/myRG",
  "subject": "/subscriptions/abc123/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/newstorage",
  "eventType": "Microsoft.Resources.ResourceWriteSuccess",
  "id": "72f988bf-86f1-41af-91ab-2d7cd011db47",
  "eventTime": "2024-01-15T16:00:00.0000000Z",
  "data": {
    "authorization": {
      "scope": "/subscriptions/abc123/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/newstorage",
      "action": "Microsoft.Storage/storageAccounts/write",
      "evidence": {
        "role": "Owner"
      }
    },
    "claims": {
      "aud": "https://management.core.windows.net/",
      "iss": "https://sts.windows.net/{tenant-id}/",
      "iat": "1705330800",
      "nbf": "1705330800",
      "exp": "1705334400"
    },
    "correlationId": "72f988bf-86f1-41af-91ab-2d7cd011db47",
    "httpRequest": {
      "clientRequestId": "72f988bf-86f1-41af-91ab-2d7cd011db47",
      "clientIpAddress": "203.0.113.42",
      "method": "PUT"
    },
    "resourceProvider": "Microsoft.Storage",
    "resourceUri": "/subscriptions/abc123/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/newstorage",
    "operationName": "Microsoft.Storage/storageAccounts/write",
    "status": "Succeeded",
    "subscriptionId": "abc123",
    "tenantId": "tenant-id"
  },
  "dataVersion": "2.0",
  "metadataVersion": "1"
}
```

---

# Сравнение схем

## CloudEvents vs. Event Grid Schema

| Характеристика | CloudEvents | Event Grid Schema |
|---------------|-------------|-------------------|
| **Стандарт** | CNCF open standard | Собственный формат Azure |
| **Интероперабельность** | Кросс-платформенная | Azure-специфичная |
| **Обязательные поля** | 4 (specversion, type, source, id) | 6 (topic, subject, eventType, id, eventTime, data) |
| **Поле времени** | `time` (необязательное) | `eventTime` (обязательное) |
| **Поле типа** | `type` | `eventType` |
| **Источник события** | `source` | `topic` |
| **Версия схемы данных** | `dataschema` | `dataVersion` |
| **Тип содержимого** | `datacontenttype` | Определяется автоматически |
| **Рекомендация** | ✅ Использовать для новых приложений | ⚠️ Для совместимости и legacy |

---

## Соответствие полей

| CloudEvents | Event Grid Schema | Комментарий |
|-------------|-------------------|-------------|
| `specversion` | `metadataVersion` | Версия схемы |
| `type` | `eventType` | Тип события |
| `source` | `topic` | Источник события |
| `id` | `id` | Уникальный идентификатор |
| `time` | `eventTime` | Временная метка |
| `subject` | `subject` | Объект события |
| `data` | `data` | Payload события |
| `datacontenttype` | N/A | Тип содержимого |
| `dataschema` | `dataVersion` | Версия схемы данных |

---

## Архитектурный вывод

- **CloudEvents** — стандарт для multi-cloud и интеграций.
- **Event Grid Schema** — нативный формат Azure.
- CloudEvents обеспечивает лучшую переносимость.
- Event Grid Schema чаще используется в старых решениях.

---

## Что выбрать?

- Новые приложения → **CloudEvents**
- Совместимость со старыми Azure-сценариями → Event Grid Schema

---

## Важно для AZ-204

На экзамене важно помнить:

- CloudEvents — рекомендуемый формат.
- В Event Grid Schema больше обязательных полей.
- `type` ≠ `eventType`.
- `source` ≠ `topic`.

Часто проверяется понимание различий и соответствия полей.

---

## Event Size and Billing

### Size Limits

| Limit Type | Value | Notes |
|------------|-------|-------|
| **Maximum Event Size** | 1 MB | Entire event including headers |
| **Minimum Billing Unit** | 64 KB | Rounded up |
| **Batch Size** | 1 MB | Total for all events in batch |

### Billing Examples

```
Event Size    → Billing Units (64 KB) → Operations Billed
32 KB         → 1 unit (64 KB)        → 1 operation
64 KB         → 1 unit (64 KB)        → 1 operation
65 KB         → 2 units (128 KB)      → 2 operations
130 KB        → 3 units (192 KB)      → 3 operations
256 KB        → 4 units (256 KB)      → 4 operations
1 MB (max)    → 16 units (1024 KB)    → 16 operations
```

## Оптимизация размера событий

Правильная структура события снижает стоимость и повышает производительность доставки.

---

### 1️⃣ Минимизируйте payload

Включайте только необходимые данные.

Рекомендуется:

- передавать идентификаторы вместо больших объектов;
- не дублировать информацию;
- избегать вложенных массивов большого размера.

Помните: максимальный размер события — 1 MB, тарификация блоками по 64 KB.

---

### 2️⃣ Используйте ссылки вместо данных

Если данные большие:

- сохраните их в Blob Storage, базе данных или другом хранилище;
- передайте в событии ссылку (URI) или ID ресурса.

Это уменьшает:

- размер события;
- стоимость;
- время доставки.

---

### 3️⃣ Сжимайте данные

Для крупных payload’ов:

- используйте компрессию;
- минимизируйте JSON (без лишних полей и пробелов).

Однако лучше избегать больших payload’ов полностью.

---

### 4️⃣ Используйте batch-отправку

Можно отправлять несколько небольших событий в одном HTTP-запросе.

Преимущества:

- меньше сетевых вызовов;
- снижение накладных расходов;
- более эффективная обработка.

---

## Архитектурный вывод

Event Grid предназначен для уведомлений о событиях, а не для передачи больших данных.

Оптимальный подход:

- Событие сообщает **что произошло**;
- Данные хранятся отдельно;
- Подписчик при необходимости загружает данные по ссылке.

---

## Важно для AZ-204

На экзамене могут проверять:

- лимит 1 MB;
- тарификацию по 64 KB;
- best practice: передавать ссылку вместо больших данных;
- использование batch-публикации для оптимизации.

**Example - Inefficient (large payload):**
```json
{
  "type": "ImageUploaded",
  "data": {
    "imageBase64": "iVBORw0KGgoAAAANSUhEUgAA... (500 KB of base64 data)"
  }
}
```

**Example - Efficient (reference-based):**
```json
{
  "type": "ImageUploaded",
  "data": {
    "imageUrl": "https://mystorage.blob.core.windows.net/images/photo.jpg",
    "imageSizeBytes": 524288,
    "contentType": "image/jpeg"
  }
}
```

---

## Subject Pattern Best Practices

The `subject` field enables powerful filtering. Design subjects hierarchically for maximum flexibility.

### Hierarchical Subject Structure

```
Format: /category/subcategory/resource/action

Examples:
✅ /blobServices/default/containers/images/blobs/photo.jpg
✅ /orders/region/west/store/101/order/12345
✅ /users/domain/example.com/user/john.doe@example.com
✅ /buildings/building-a/floor-3/room-301/sensor/temp-001

❌ photo.jpg
❌ order-12345
❌ john-doe-user
```

### Filtering with Subjects

```bash
# Filter by subject prefix (all blobs in "images" container)
--subject-begins-with "/blobServices/default/containers/images/"

# Filter by subject suffix (only .jpg files)
--subject-ends-with ".jpg"

# Combine both
--subject-begins-with "/blobServices/default/containers/images/" \
--subject-ends-with ".jpg"
```

### Subject Design Patterns

**Pattern 1: Resource Hierarchy**
```
/resource-type/region/group/instance
Example: /databases/eastus/production/db-001
```

**Pattern 2: Namespace Hierarchy**
```
/namespace/entity/id/operation
Example: /myapp/orders/12345/created
```

**Pattern 3: Domain-Driven Design**
```
/bounded-context/aggregate/id
Example: /inventory/products/SKU-001
```

---

## Publishing Events with Different Schemas

### Publish CloudEvents (Azure CLI)

```bash
# Publish CloudEvents to custom topic
az eventgrid topic publish \
  --name myTopic \
  --resource-group myRG \
  --events '[
    {
      "specversion": "1.0",
      "type": "com.example.orders.OrderCreated",
      "source": "https://api.example.com/orders",
      "subject": "/orders/12345",
      "id": "event-001",
      "time": "2024-01-15T14:30:00Z",
      "datacontenttype": "application/json",
      "data": {
        "orderId": "12345",
        "amount": 99.99
      }
    }
  ]' \
  --schema cloudevents
```

### Publish Event Grid Schema (Azure CLI)

```bash
# Publish Event Grid schema events
az eventgrid topic publish \
  --name myTopic \
  --resource-group myRG \
  --events '[
    {
      "id": "event-001",
      "eventType": "MyApp.Orders.OrderCreated",
      "subject": "/orders/12345",
      "eventTime": "2024-01-15T14:30:00Z",
      "data": {
        "orderId": "12345",
        "amount": 99.99
      },
      "dataVersion": "1.0"
    }
  ]'
```

### Publish CloudEvents (C# SDK)

```csharp
using Azure;
using Azure.Messaging;
using Azure.Messaging.EventGrid;

var endpoint = new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events");
var credential = new AzureKeyCredential(topicKey);
var client = new EventGridPublisherClient(endpoint, credential);

// Create CloudEvent
var cloudEvent = new CloudEvent(
    source: "https://api.example.com/orders",
    type: "com.example.orders.OrderCreated",
    jsonSerializableData: new
    {
        orderId = "12345",
        amount = 99.99
    })
{
    Id = "event-001",
    Subject = "/orders/12345",
    Time = DateTimeOffset.UtcNow
};

// Publish event
await client.SendEventAsync(cloudEvent);

// Publish multiple events
var events = new[]
{
    cloudEvent,
    new CloudEvent("https://api.example.com/orders", "com.example.orders.OrderCreated", 
        new { orderId = "12346", amount = 149.99 })
};
await client.SendEventsAsync(events);
```

### Publish CloudEvents (Python SDK)

```python
from azure.eventgrid import EventGridPublisherClient
from azure.core.credentials import AzureKeyCredential
from azure.core.messaging import CloudEvent
from datetime import datetime

endpoint = "https://mytopic.eastus-1.eventgrid.azure.net/api/events"
credential = AzureKeyCredential(topic_key)
client = EventGridPublisherClient(endpoint, credential)

# Create CloudEvent
cloud_event = CloudEvent(
    source="https://api.example.com/orders",
    type="com.example.orders.OrderCreated",
    data={
        "orderId": "12345",
        "amount": 99.99
    },
    subject="/orders/12345",
    time=datetime.utcnow()
)

# Publish event
client.send(cloud_event)

# Publish multiple events
events = [
    cloud_event,
    CloudEvent(
        source="https://api.example.com/orders",
        type="com.example.orders.OrderCreated",
        data={"orderId": "12346", "amount": 149.99}
    )
]
client.send(events)
```

---

## Schema Validation

### Validating CloudEvents

```python
from jsonschema import validate
import json

# CloudEvents schema
cloudevents_schema = {
    "type": "object",
    "required": ["specversion", "type", "source", "id"],
    "properties": {
        "specversion": {"type": "string", "const": "1.0"},
        "type": {"type": "string"},
        "source": {"type": "string"},
        "id": {"type": "string"},
        "time": {"type": "string", "format": "date-time"},
        "data": {"type": "object"}
    }
}

# Validate event
event = {
    "specversion": "1.0",
    "type": "com.example.someevent",
    "source": "/mycontext",
    "id": "A234-1234-1234",
    "data": {"key": "value"}
}

try:
    validate(instance=event, schema=cloudevents_schema)
    print("Event is valid")
except Exception as e:
    print(f"Event validation failed: {e}")
```

---

# Советы к экзамену AZ-204

## Ключевые моменты, которые нужно запомнить

1️⃣ **CloudEvents рекомендуется** для новых приложений  
Используется спецификация v1.0.

2️⃣ **Event Grid Schema** поддерживается  
В основном для обратной совместимости.

3️⃣ **Максимальный размер события — 1 MB**

4️⃣ **Тарификация — блоками по 64 KB**

5️⃣ Поле **`subject`** позволяет выполнять гибкую фильтрацию  
(по префиксу, суффиксу, шаблону).

6️⃣ В **CloudEvents** — 4 обязательных поля  
(`specversion`, `type`, `source`, `id`)

В **Event Grid Schema** — 6 обязательных полей  
(`topic`, `subject`, `eventType`, `id`, `eventTime`, `data`)

7️⃣ В CloudEvents используется поле **`datacontenttype`**  
В Event Grid Schema тип данных определяется автоматически.

---

## Что чаще всего проверяют

- Разницу между `type` и `eventType`
- Разницу между `source` и `topic`
- Какое поле используется для фильтрации
- Ограничение 1 MB
- Расчёт стоимости по 64 KB

---

## Экзаменационный лайфхак

Если в вопросе речь о:

- новом приложении → выбирайте **CloudEvents**
- legacy или совместимости → Event Grid Schema
- фильтрации → обращайте внимание на `subject`

Понимание различий между схемами — частая тема в вопросах AZ-204.
### Schema Selection Decision Tree

```
New Application?
├─ Yes → Use CloudEvents ✅
│
└─ No (Existing)
   ├─ Need cross-platform compatibility? → CloudEvents ✅
   ├─ Backward compatibility required? → Event Grid Schema
   └─ Migrating? → CloudEvents (recommended)
```

## Частые экзаменационные сценарии

### Сценарий 1
Проектирование схемы событий для multi-cloud приложения

- ✅ Использовать CloudEvents (интероперабельность)
- ❌ Не использовать Event Grid Schema (Azure-специфичная)

---

### Сценарий 2
Минимизация стоимости событий

- ✅ Держать размер события менее 64 KB при возможности
- ✅ Использовать ссылочную модель (URL, ID)
- ❌ Не встраивать большие payload’ы

---

### Сценарий 3
Реализация сложной фильтрации

- ✅ Проектировать иерархическую структуру `subject`
- ✅ Использовать единые соглашения об именовании
- ❌ Не использовать плоские и неструктурированные subject

---

## Что помнить на экзамене

- **Обязательные поля CloudEvents**:  
  `specversion`, `type`, `source`, `id`

- **Обязательные поля Event Grid Schema**:  
  `topic`, `subject`, `eventType`, `id`, `eventTime`, `data`

- **Content-Type для CloudEvents**:  
  `application/cloudevents+json`

- **Максимальный размер события**: 1 MB

- **Тарификация**: блоками по 64 KB

- **Фильтрация по subject**:  
  `subjectBeginsWith`, `subjectEndsWith`

- **`dataVersion`**: отслеживание версии схемы в Event Grid Schema
- **`dataschema`**: URI схемы в CloudEvents

---

## Быстрое сравнение

| Вопрос | CloudEvents | Event Grid |
|--------|-------------|------------|
| Отраслевой стандарт? | ✅ Да (CNCF) | ❌ Только Azure |
| Рекомендуется? | ✅ Да | ⚠️ Для legacy |
| Обязательных полей | 4 | 6 |
| Версионирование схемы | `dataschema` (URI) | `dataVersion` (string) |
| Content-Type | Явно указан | Определяется автоматически |

---

# Итоги

## Схемы событий

- **CloudEvents v1.0** — отраслевой стандарт, рекомендован для новых решений
- **Event Grid Schema** — нативный формат Azure для обратной совместимости

---

## Ключевые различия

- В CloudEvents меньше обязательных полей (4 против 6)
- CloudEvents поддерживает multi-cloud
- CloudEvents явно определяет Content-Type

---

## Best Practices

- ✅ Использовать **CloudEvents** для новых приложений
- ✅ Проектировать **иерархический subject**
- ✅ Держать события **менее 64 KB**
- ✅ Передавать **ссылки вместо больших данных**
- ✅ Использовать `dataVersion` или `dataschema` для версионирования

---

## Ограничения

- Максимальный размер события: **1 MB**
- Тарификация: блоками по **64 KB**
- Общий размер batch-запроса: **до 1 MB**

---

### Экзаменационный акцент

Если в вопросе:

- multi-cloud → CloudEvents
- новое приложение → CloudEvents
- legacy Azure → Event Grid Schema
- оптимизация стоимости → уменьшение payload

Различие между схемами — частая проверяемая тема в AZ-204.
| Сервис                 | Очередь | FIFO | Размер        | Назначение           |
| ---------------------- | ------- | ---- | ------------- | -------------------- |
| Storage Queues         | ✅       | ❌    | Очень большой | Simple queue         |
| **Service Bus Queues** | ✅       | ✅    | **до 80 GB**  | Enterprise messaging |
| Event Hubs             | ❌       | ❌    | Huge          | Streaming            |
| Event Grid             | ❌       | ❌    | N/A           | Event routing        |

Event Hubs — это сервис для:
массового приёма событий (telemetry ingestion)
высокой пропускной способности
потоковой обработки
сценариев IoT / device data

Devices → Event Hubs → (consumer) → Blob Storage

Экзаменационное правило
Если видишь:
telemetry
много устройств/источников
поток событий
ingestion
👉 выбирай Azure Event Hubs (или IoT Hub, если он среди вариантов).

Интеграция Service Bus с Event Grid поддерживается только в Premium tier.