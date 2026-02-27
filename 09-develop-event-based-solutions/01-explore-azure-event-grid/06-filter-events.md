# Фильтрация событий в Azure Event Grid

## Обзор

**Фильтрация событий (Event filtering)** позволяет управлять тем, какие события будут доставлены в конкретную подписку (subscription endpoint).  
Это помогает:

- уменьшить лишний сетевой трафик
- снизить нагрузку на обработчики
- сократить вычислительные затраты
- повысить производительность системы
- упростить логику downstream-сервисов

Фактически, фильтрация — это первый уровень оптимизации событийно-ориентированной архитектуры (event-driven architecture) в Azure.

---

## Типы фильтрации

### 1. Фильтрация по типу события (Event Type Filtering)

Позволяет доставлять только события определённого типа (`eventType`).

Пример:
- `Microsoft.Storage.BlobCreated`
- `Microsoft.Storage.BlobDeleted`

Используется, когда:
- сервису нужны только события создания объектов
- логика обработки зависит строго от типа события

💡 **Практический совет:**  
Лучше сразу ограничивать подписку конкретными типами событий, чем фильтровать их уже в коде обработчика.

---

### 2. Фильтрация по Subject (Subject Filtering)

Позволяет фильтровать события по полю `subject`, используя:
- `beginsWith`
- `endsWith`

`subject` обычно содержит путь к ресурсу.


---

## Event Type Filtering

Filter events based on their `eventType` (Event Grid schema) or `type` (CloudEvents schema).

### Filter by Specific Event Types

**Azure CLI:**
```bash
# Subscribe to specific event types only
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage" \
  --endpoint https://myfunction.azurewebsites.net/api/handler \
  --included-event-types Microsoft.Storage.BlobCreated Microsoft.Storage.BlobDeleted
```

**ARM Template:**
```json
{
  "type": "Microsoft.EventGrid/eventSubscriptions",
  "properties": {
    "destination": {
      "endpointType": "WebHook",
      "properties": {
        "endpointUrl": "https://myfunction.azurewebsites.net/api/handler"
      }
    },
    "filter": {
      "includedEventTypes": [
        "Microsoft.Storage.BlobCreated",
        "Microsoft.Storage.BlobDeleted"
      ]
    }
  }
}
```

### Subscribe to All Event Types

**Azure CLI:**
```bash
# Receive all event types (default behavior)
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --included-event-types All
```

**ARM Template:**
```json
{
  "filter": {
    "includedEventTypes": []
  }
}
```
*Empty array = all event types*

### Распространённые типы событий Azure Event Grid

Ниже приведены наиболее часто встречающиеся типы событий, которые различные сервисы Azure публикуют в Azure Event Grid.  
Эти события часто встречаются в сценариях интеграции и могут появляться в вопросах экзамена AZ-204.

| Azure Service | Типы событий |
|--------------|--------------|
| **Blob Storage** | `Microsoft.Storage.BlobCreated` — создание Blob<br>`Microsoft.Storage.BlobDeleted` — удаление Blob<br>`Microsoft.Storage.BlobTierChanged` — изменение уровня хранения |
| **Resource Manager** | `Microsoft.Resources.ResourceWriteSuccess` — успешное создание или обновление ресурса<br>`Microsoft.Resources.ResourceDeleteSuccess` — успешное удаление ресурса<br>`Microsoft.Resources.ResourceActionSuccess` — успешное выполнение действия над ресурсом |
| **Event Hubs** | `Microsoft.EventHub.CaptureFileCreated` — создан файл Capture |
| **IoT Hub** | `Microsoft.Devices.DeviceCreated` — устройство создано<br>`Microsoft.Devices.DeviceDeleted` — устройство удалено<br>`Microsoft.Devices.DeviceConnected` — устройство подключено<br>`Microsoft.Devices.DeviceDisconnected` — устройство отключено |
| **Container Registry** | `Microsoft.ContainerRegistry.ImagePushed` — образ загружен<br>`Microsoft.ContainerRegistry.ImageDeleted` — образ удалён<br>`Microsoft.ContainerRegistry.ChartPushed` — Helm-чарт загружен |
| **Media Services** | `Microsoft.Media.JobStateChange` — изменение состояния задания<br>`Microsoft.Media.JobOutputStateChange` — изменение состояния результата задания<br>`Microsoft.Media.LiveEventEncoderConnected` — подключение энкодера к Live Event |
| **Service Bus** | `Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners` — есть активные сообщения без подписчиков<br>`Microsoft.ServiceBus.DeadletterMessagesAvailableWithNoListener` — есть сообщения в DLQ без подписчиков |
| **App Configuration** | `Microsoft.AppConfiguration.KeyValueModified` — значение ключа изменено<br>`Microsoft.AppConfiguration.KeyValueDeleted` — значение ключа удалено |

---

## Что важно для AZ-204

- Blob Storage и Resource Manager — самые часто встречающиеся сервисы в вопросах.
- IoT Hub и Container Registry часто используются в интеграционных и DevOps-сценариях.
- Service Bus события связаны с мониторингом и обработкой сообщений.
- App Configuration полезен в сценариях динамического изменения конфигурации приложений.

📌 На экзамене не требуется запоминать все типы событий дословно, но важно понимать:
- какой сервис какие события генерирует
- в каком сценарии эти события используются
- как их можно фильтровать через Event Grid

### Custom Event Types

For custom topics, define your own event types:

```bash
az eventgrid event-subscription create \
  --name orderSubscription \
  --source-resource-id $CUSTOM_TOPIC_ID \
  --endpoint $WEBHOOK_URL \
  --included-event-types \
    MyApp.Orders.OrderCreated \
    MyApp.Orders.OrderShipped \
    MyApp.Orders.OrderCancelled
```

---

## Subject Filtering

Filter events based on the beginning or ending of the `subject` field.

### Subject Structure Best Practices

Design hierarchical subjects for flexible filtering:

```
Format: /category/subcategory/resource

Examples:
/blobServices/default/containers/images/blobs/photo.jpg
/blobServices/default/containers/documents/blobs/report.pdf
/orders/region/west/store/101/order/12345
/users/domain/example.com/user/john.doe
```

### Filter by Subject Prefix (subjectBeginsWith)

**Use Case:** Filter events from specific container or folder

```bash
# Filter: Only blobs from "images" container
az eventgrid event-subscription create \
  --name imageSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --subject-begins-with "/blobServices/default/containers/images/"
```

**ARM Template:**
```json
{
  "filter": {
    "subjectBeginsWith": "/blobServices/default/containers/images/"
  }
}
```

**What matches:**
- ✅ `/blobServices/default/containers/images/blobs/photo1.jpg`
- ✅ `/blobServices/default/containers/images/blobs/subfolder/photo2.jpg`
- ❌ `/blobServices/default/containers/documents/blobs/doc.pdf`

### Filter by Subject Suffix (subjectEndsWith)

**Use Case:** Filter events for specific file types

```bash
# Filter: Only .jpg files
az eventgrid event-subscription create \
  --name jpgSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --subject-ends-with ".jpg"
```

**ARM Template:**
```json
{
  "filter": {
    "subjectEndsWith": ".jpg"
  }
}
```

**What matches:**
- ✅ `/blobServices/default/containers/images/blobs/photo.jpg`
- ✅ `/blobServices/default/containers/archive/blobs/old-photo.jpg`
- ❌ `/blobServices/default/containers/images/blobs/photo.png`

### Combine Prefix and Suffix

**Use Case:** Filter .jpg files from specific container

```bash
# Filter: .jpg files from "images" container only
az eventgrid event-subscription create \
  --name imageJpgSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --subject-begins-with "/blobServices/default/containers/images/" \
  --subject-ends-with ".jpg"
```

**ARM Template:**
```json
{
  "filter": {
    "subjectBeginsWith": "/blobServices/default/containers/images/",
    "subjectEndsWith": ".jpg"
  }
}
```

**What matches:**
- ✅ `/blobServices/default/containers/images/blobs/photo.jpg`
- ✅ `/blobServices/default/containers/images/blobs/vacation/beach.jpg`
- ❌ `/blobServices/default/containers/images/blobs/photo.png` (wrong extension)
- ❌ `/blobServices/default/containers/documents/blobs/scan.jpg` (wrong container)

### Subject Filtering Examples

**Example 1: Filter by Region**
```bash
# Orders from West region only
--subject-begins-with "/orders/region/west/"
```

**Example 2: Filter by Customer Domain**
```bash
# Users from example.com domain
--subject-begins-with "/users/domain/example.com/"
```

**Example 3: Filter by File Extension**
```bash
# PDF documents only
--subject-ends-with ".pdf"

# Images only (.jpg, .png, .gif)
# Note: Need separate subscriptions for each extension
--subject-ends-with ".jpg"  # Subscription 1
--subject-ends-with ".png"  # Subscription 2
--subject-ends-with ".gif"  # Subscription 3
```

**Example 4: Filter by Blob Container**
```bash
# Production container only
--subject-begins-with "/blobServices/default/containers/production/"

# Exclude system containers
--subject-begins-with "/blobServices/default/containers/" \
--subject-does-not-begin-with "/blobServices/default/containers/$"
```

---

## Расширенная фильтрация (Advanced Filtering)

Позволяет фильтровать события на основе **значений полей внутри `data`**, используя операторы сравнения.

Это самый гибкий механизм фильтрации в Azure Event Grid.  
Фильтрация выполняется на стороне сервиса, до доставки события в подписку.

---

### Операторы расширенной фильтрации

| Оператор | Тип данных | Описание |
|-----------|------------|----------|
| `NumberIn` | Число | Значение входит в список |
| `NumberNotIn` | Число | Значение не входит в список |
| `NumberLessThan` | Число | Значение меньше указанного |
| `NumberLessThanOrEquals` | Число | Значение меньше или равно указанному |
| `NumberGreaterThan` | Число | Значение больше указанного |
| `NumberGreaterThanOrEquals` | Число | Значение больше или равно указанному |
| `BoolEquals` | Boolean | Значение равно true или false |
| `StringIn` | Строка | Значение входит в список |
| `StringNotIn` | Строка | Значение не входит в список |
| `StringBeginsWith` | Строка | Значение начинается с указанной строки |
| `StringEndsWith` | Строка | Значение заканчивается указанной строкой |
| `StringContains` | Строка | Значение содержит подстроку |
| `StringNotContains` | Строка | Значение не содержит подстроку |
| `StringNotBeginsWith` | Строка | Значение не начинается с указанной строки |
| `StringNotEndsWith` | Строка | Значение не заканчивается указанной строкой |
| `IsNullOrUndefined` | Любой | Поле равно null или отсутствует |
| `IsNotNull` | Любой | Поле не равно null |

---

### Ограничения расширенной фильтрации

- **Максимальное количество фильтров на одну подписку**: 25
- **Максимальное количество значений для операторов In / NotIn**: 25
- **Максимальная длина строкового значения**: 512 символов
- **Максимальная длина ключа (имени поля)**: 64 символа

---

## Практические замечания

- Все условия работают по принципу **AND** — событие должно соответствовать всем заданным фильтрам.
- Поля указываются относительно объекта `data` (например, `data.propertyName`).
- Если поле отсутствует в событии, оператор может не сработать так, как ожидается — это важно учитывать при проектировании схемы событий.
- Расширенная фильтрация особенно полезна в высоконагруженных системах и serverless-сценариях, где важно минимизировать лишние вызовы обработчиков.

---

## Что важно помнить для AZ-204

- Advanced Filtering применяется к данным события (`data`), а не к `eventType` или `subject`.
- Поддерживаются числовые, строковые и логические операторы.
- Существуют ограничения на количество условий и длину значений.
- Фильтрация выполняется до доставки события конечной точке.

Расширенная фильтрация — ключевой инструмент для построения эффективной событийной архитектуры в Azure.
### Number Filtering Examples

**Example 1: Filter by Blob Size**

```bash
# Only blobs larger than 1 MB (1,048,576 bytes)
az eventgrid event-subscription create \
  --name largeBlobSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --included-event-types Microsoft.Storage.BlobCreated \
  --advanced-filter data.contentLength NumberGreaterThan 1048576
```

**ARM Template:**
```json
{
  "filter": {
    "advancedFilters": [
      {
        "operatorType": "NumberGreaterThan",
        "key": "data.contentLength",
        "value": 1048576
      }
    ]
  }
}
```

**Example 2: Filter by Range**

```bash
# Blobs between 100 KB and 10 MB
az eventgrid event-subscription create \
  --name mediumBlobSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --advanced-filter data.contentLength NumberGreaterThanOrEquals 102400 \
  --advanced-filter data.contentLength NumberLessThanOrEquals 10485760
```

**ARM Template:**
```json
{
  "filter": {
    "advancedFilters": [
      {
        "operatorType": "NumberGreaterThanOrEquals",
        "key": "data.contentLength",
        "value": 102400
      },
      {
        "operatorType": "NumberLessThanOrEquals",
        "key": "data.contentLength",
        "value": 10485760
      }
    ]
  }
}
```

### String Filtering Examples

**Example 1: Filter by Blob Type**

```bash
# Only Block Blobs
az eventgrid event-subscription create \
  --name blockBlobSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --advanced-filter data.blobType StringIn BlockBlob
```

**ARM Template:**
```json
{
  "filter": {
    "advancedFilters": [
      {
        "operatorType": "StringIn",
        "key": "data.blobType",
        "values": ["BlockBlob"]
      }
    ]
  }
}
```

**Example 2: Filter by Content Type**

```bash
# Only image files (MIME type starts with "image/")
az eventgrid event-subscription create \
  --name imageTypeSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --advanced-filter data.contentType StringBeginsWith image/
```

**ARM Template:**
```json
{
  "filter": {
    "advancedFilters": [
      {
        "operatorType": "StringBeginsWith",
        "key": "data.contentType",
        "values": ["image/"]
      }
    ]
  }
}
```

**Example 3: Filter by Multiple Content Types**

```bash
# Images and videos only
az eventgrid event-subscription create \
  --name mediaSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --advanced-filter data.contentType StringIn image/jpeg image/png video/mp4 video/mpeg
```

**ARM Template:**
```json
{
  "filter": {
    "advancedFilters": [
      {
        "operatorType": "StringIn",
        "key": "data.contentType",
        "values": ["image/jpeg", "image/png", "video/mp4", "video/mpeg"]
      }
    ]
  }
}
```

**Example 4: Exclude System Blobs**

```bash
# Exclude blobs starting with "."
az eventgrid event-subscription create \
  --name userBlobSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --advanced-filter data.url StringNotContains "/.well-known/" \
  --advanced-filter data.url StringNotContains "/$logs/"
```

### Boolean Filtering Examples

**Example: Filter by Blob Tier**

```bash
# Only hot tier blobs
az eventgrid event-subscription create \
  --name hotTierSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --advanced-filter data.accessTier StringIn Hot
```

### Null/Undefined Filtering

**Example: Filter Events with Missing Fields**

```bash
# Events where metadata is not set
az eventgrid event-subscription create \
  --name noMetadataSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --advanced-filter data.metadata IsNullOrUndefined
```

### Custom Event Advanced Filtering

**Example: Order Processing**

```bash
# Orders over $1000 from premium customers
az eventgrid event-subscription create \
  --name premiumOrderSubscription \
  --source-resource-id $CUSTOM_TOPIC_ID \
  --endpoint $WEBHOOK_URL \
  --included-event-types MyApp.Orders.OrderCreated \
  --advanced-filter data.amount NumberGreaterThan 1000 \
  --advanced-filter data.customerTier StringIn Premium Gold
```

**ARM Template:**
```json
{
  "filter": {
    "includedEventTypes": ["MyApp.Orders.OrderCreated"],
    "advancedFilters": [
      {
        "operatorType": "NumberGreaterThan",
        "key": "data.amount",
        "value": 1000
      },
      {
        "operatorType": "StringIn",
        "key": "data.customerTier",
        "values": ["Premium", "Gold"]
      }
    ]
  }
}
```

---

## Combining Filter Types

Combine event type, subject, and advanced filters for precise event routing.

### Example 1: Image Processing Pipeline

**Requirement:** Process .jpg images over 500 KB from "uploads" container

```bash
az eventgrid event-subscription create \
  --name imageProcessingSubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint $WEBHOOK_URL \
  --included-event-types Microsoft.Storage.BlobCreated \
  --subject-begins-with "/blobServices/default/containers/uploads/" \
  --subject-ends-with ".jpg" \
  --advanced-filter data.contentLength NumberGreaterThan 512000
```

**ARM Template:**
```json
{
  "filter": {
    "includedEventTypes": ["Microsoft.Storage.BlobCreated"],
    "subjectBeginsWith": "/blobServices/default/containers/uploads/",
    "subjectEndsWith": ".jpg",
    "advancedFilters": [
      {
        "operatorType": "NumberGreaterThan",
        "key": "data.contentLength",
        "value": 512000
      }
    ]
  }
}
```

### Example 2: Multi-Region Order Processing

**Requirement:** High-value orders from West or East regions

```bash
# West region orders over $5000
az eventgrid event-subscription create \
  --name westHighValueOrders \
  --source-resource-id $CUSTOM_TOPIC_ID \
  --endpoint $WEBHOOK_URL_WEST \
  --included-event-types MyApp.Orders.OrderCreated \
  --subject-begins-with "/orders/region/west/" \
  --advanced-filter data.amount NumberGreaterThan 5000

# East region orders over $5000
az eventgrid event-subscription create \
  --name eastHighValueOrders \
  --source-resource-id $CUSTOM_TOPIC_ID \
  --endpoint $WEBHOOK_URL_EAST \
  --included-event-types MyApp.Orders.OrderCreated \
  --subject-begins-with "/orders/region/east/" \
  --advanced-filter data.amount NumberGreaterThan 5000
```

### Example 3: Document Processing Workflow

**Requirement:** PDF and Word documents from specific folders

```bash
# Subscription 1: PDF documents
az eventgrid event-subscription create \
  --name pdfProcessing \
  --source-resource-id $STORAGE_ID \
  --endpoint $PDF_PROCESSOR_URL \
  --included-event-types Microsoft.Storage.BlobCreated \
  --subject-begins-with "/blobServices/default/containers/documents/" \
  --subject-ends-with ".pdf"

# Subscription 2: Word documents
az eventgrid event-subscription create \
  --name wordProcessing \
  --source-resource-id $STORAGE_ID \
  --endpoint $WORD_PROCESSOR_URL \
  --included-event-types Microsoft.Storage.BlobCreated \
  --subject-begins-with "/blobServices/default/containers/documents/" \
  --advanced-filter data.contentType StringIn \
    application/msword \
    application/vnd.openxmlformats-officedocument.wordprocessingml.document
```

---

## Порядок применения фильтров (Filter Evaluation Order)

Фильтры в Azure Event Grid применяются в следующем порядке:

1. **Фильтр по типу события (Event Type Filter)** — самый быстрый, проверяется первым
2. **Фильтр по Subject (Subject Filter)** — сопоставление по префиксу / суффиксу строки
3. **Расширенные фильтры (Advanced Filters)** — наиболее ресурсоёмкие, выполняются последними

---

## Почему это важно

Azure Event Grid оптимизирует обработку событий, начиная с самых дешёвых операций:

- Проверка `eventType` — простое сравнение строки
- Проверка `subject` — сопоставление начала или конца строки
- Advanced Filtering — анализ значений внутри `data`, потенциально с несколькими условиями

Чем раньше событие будет «отфильтровано», тем меньше ресурсов потребуется системе.

---

## Рекомендация по оптимизации

**Старайтесь использовать фильтрацию по типу события и Subject там, где это возможно.**

Это позволяет:

- уменьшить нагрузку на Event Grid
- повысить производительность
- сократить задержки доставки событий
- минимизировать вычислительные затраты

Расширенную фильтрацию имеет смысл применять только тогда, когда более простые механизмы недостаточны.

---

## Что помнить для AZ-204

- Фильтры выполняются последовательно
- Event Type — самый быстрый уровень фильтрации
- Advanced Filtering — самый «дорогой» по вычислениям
- Правильная комбинация фильтров влияет на производительность архитектуры

Понимание порядка выполнения фильтров помогает выбирать наиболее эффективную стратегию обработки событий.

---

## Testing Filters

### Test with Azure CLI

```bash
# Publish test event
az eventgrid event publish \
  --topic-name myTopic \
  --resource-group myRG \
  --events '[{
    "id": "test-001",
    "eventType": "Microsoft.Storage.BlobCreated",
    "subject": "/blobServices/default/containers/images/blobs/test.jpg",
    "eventTime": "2024-01-15T14:30:00Z",
    "data": {
      "api": "PutBlob",
      "contentType": "image/jpeg",
      "contentLength": 1048576,
      "blobType": "BlockBlob",
      "url": "https://mystorage.blob.core.windows.net/images/test.jpg"
    },
    "dataVersion": "1.0"
  }]'
```

### Test with PowerShell

```powershell
# Test event that should match
$event = @{
    id = "test-001"
    eventType = "Microsoft.Storage.BlobCreated"
    subject = "/blobServices/default/containers/images/blobs/test.jpg"
    eventTime = (Get-Date -Format o)
    data = @{
        contentType = "image/jpeg"
        contentLength = 2000000
        blobType = "BlockBlob"
    }
    dataVersion = "1.0"
}

Invoke-RestMethod -Method Post -Uri $topicEndpoint -Headers @{"aeg-sas-key"=$topicKey} -Body ($event | ConvertTo-Json)
```

### Check Subscription Metrics

```bash
# Check matched events
az monitor metrics list \
  --resource $TOPIC_ID \
  --metric "MatchedEventCount" \
  --dimension "EventSubscriptionName=mySubscription" \
  --start-time 2024-01-15T00:00:00Z

# Check delivered events
az monitor metrics list \
  --resource $TOPIC_ID \
  --metric "DeliverySuccessCount" \
  --dimension "EventSubscriptionName=mySubscription" \
  --start-time 2024-01-15T00:00:00Z
```

---

## Best Practices

### Filter Design

1. **Use event type filters first** - Most efficient
2. **Design hierarchical subjects** - Enable flexible prefix filtering
3. **Minimize advanced filters** - Most expensive to evaluate
4. **Avoid overlapping subscriptions** - Can cause duplicate processing
5. **Test filters thoroughly** - Verify with sample events

### Subject Patterns

```
✅ Good patterns (hierarchical):
/resource-type/region/group/instance
/orders/region/west/store/101/order/12345
/blobServices/default/containers/images/blobs/photo.jpg

❌ Bad patterns (flat):
order-12345
photo.jpg
west-store-101
```

### Performance Optimization

1. **Event Type**: Fastest filter
2. **Subject Prefix/Suffix**: Fast string matching
3. **Advanced Filters**: Slower, limit to ≤ 25 per subscription

### Multiple Subscriptions vs. Complex Filters

**Scenario:** Route events to different endpoints based on region

**Option 1: Multiple Subscriptions (Recommended)**
```bash
# Subscription 1: West region
az eventgrid event-subscription create \
  --name westSubscription \
  --endpoint $WEST_ENDPOINT \
  --subject-begins-with "/orders/region/west/"

# Subscription 2: East region
az eventgrid event-subscription create \
  --name eastSubscription \
  --endpoint $EAST_ENDPOINT \
  --subject-begins-with "/orders/region/east/"
```

**Option 2: Single Subscription with Complex Logic (Not Recommended)**
```bash
# Single endpoint processes all regions
az eventgrid event-subscription create \
  --name allRegionsSubscription \
  --endpoint $ROUTING_ENDPOINT
# Routing logic in handler code ❌
```

---

## Советы к экзамену AZ-204

## Ключевые концепции, которые нужно помнить

1. **Три типа фильтрации**:
    - по типу события (Event Type)
    - по subject
    - расширенная (Advanced)

2. **Фильтры subject**:
    - `subjectBeginsWith`
    - `subjectEndsWith`

3. **Расширенная фильтрация**:
    - максимум 25 условий на одну подписку

4. **Порядок применения фильтров**:  
   Event Type → Subject → Advanced

5. **Чувствительность к регистру**:  
   фильтры subject **чувствительны к регистру**

6. **Операторы расширенной фильтрации**:  
   `NumberGreaterThan`, `StringContains`, `BoolEquals`, `StringIn`, `IsNullOrUndefined` и другие

---

## Типовые экзаменационные сценарии

### Сценарий 1: Отфильтровать `.jpg` изображения из конкретного контейнера

✔ Использовать `subjectBeginsWith` для контейнера  
✔ Использовать `subjectEndsWith` для `.jpg`  
✘ Не использовать advanced-фильтры (менее эффективно)

Почему: subject-фильтры быстрее и дешевле по вычислениям.

---

### Сценарий 2: Фильтрация по значению поля внутри `data`

✔ Использовать расширенные фильтры с подходящим оператором  
✘ Не пытаться использовать subject-фильтры для полей `data`

Почему: subject работает только с полем `subject`, а не с содержимым события.

---

### Сценарий 3: Оптимизация производительности

✔ Сначала ограничить события по типу (самый быстрый фильтр)  
✔ Использовать subject вместо advanced, если возможно  
✘ Не использовать 25 advanced-фильтров, если задачу можно решить проще

На экзамене часто проверяется умение выбрать **самый эффективный вариант**, а не просто «работающий».

---

### Сценарий 4: Несколько условий

✔ Можно комбинировать Event Type + Subject + Advanced  
✔ Все условия должны выполняться (логика AND)

Важно: если хотя бы одно условие не выполняется — событие не будет доставлено.

---

## Что обязательно запомнить

- **Event Type** — самый эффективный фильтр
- **Subject** — чувствителен к регистру, работает по префиксу/суффиксу
- **Advanced** — максимум 25 условий, самый ресурсоёмкий
- **Операторы** — знать базовые:  
  `NumberGreaterThan`, `StringContains`, `StringIn`, `BoolEquals`, `IsNullOrUndefined`
- **Порядок выполнения** — Type → Subject → Advanced
- **Все фильтры работают по логике AND**
- Если список типов событий пуст — будут приниматься **все типы событий**

---

## Финальный совет

В вопросах AZ-204 почти всегда нужно выбрать:

- наиболее производительный вариант
- минимально сложную конфигурацию
- правильный тип фильтра для конкретного поля

Думайте не только «работает ли», но и «оптимально ли».
### Quick Command Reference

```bash
# Event type filter
--included-event-types Microsoft.Storage.BlobCreated

# Subject filters
--subject-begins-with "/blobServices/default/containers/images/"
--subject-ends-with ".jpg"

# Advanced filter
--advanced-filter data.contentLength NumberGreaterThan 1048576
--advanced-filter data.blobType StringIn BlockBlob
--advanced-filter data.contentType StringBeginsWith image/
```

---

## Итог

## Типы фильтрации

- **Event Type** — фильтрация по типу события (самый быстрый вариант)
- **Subject** — фильтрация по префиксу или суффиксу поля `subject`
- **Advanced** — фильтрация по значениям полей внутри `data` (самый гибкий механизм)

---

## Фильтрация по Subject

- `subjectBeginsWith` — сопоставление по префиксу (например, путь контейнера)
- `subjectEndsWith` — сопоставление по суффиксу (например, расширение файла)
- Сопоставление строк **чувствительно к регистру**

Subject-фильтрация особенно эффективна, если структура `subject` продумана заранее.

---

## Расширенная фильтрация (Advanced)

- **Операторы**:
    - Числовые (>, <, ≥, ≤, In, NotIn)
    - Строковые (contains, beginsWith, endsWith, In)
    - Boolean
    - Проверка на null

- **Ограничения**:
    - максимум 25 условий на подписку

- **Поля данных**:  
  Можно фильтровать по любому полю внутри `data` события.

---

## Лучшие практики

✅ Проектируйте иерархические `subject`, чтобы упростить фильтрацию  
✅ Используйте фильтр по типу события, когда это возможно (самый эффективный способ)  
✅ Комбинируйте фильтры для точной маршрутизации  
✅ Тестируйте фильтры на примерах событий  
✅ Используйте несколько подписок для разных конечных точек

❌ Не переносите сложную фильтрацию в код обработчика  
❌ Не превышайте лимит в 25 advanced-фильтров на подписку

---

## Распространённые паттерны

- Только изображения: фильтрация по суффиксу `.jpg`
- Конкретный контейнер: фильтрация по префиксу пути контейнера
- Большие файлы: фильтрация по `data.contentLength` с условием больше заданного значения
- Определённый тип контента: фильтрация по `data.contentType` с использованием строкового оператора

---

## Ключевая идея

Всегда выбирайте:

- самый простой возможный фильтр
- самый ранний уровень фильтрации (Type → Subject → Advanced)
- минимально необходимое количество условий

Эффективная фильтрация — это не только корректность, но и производительность архитектуры.