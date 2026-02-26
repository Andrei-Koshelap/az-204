# Azure Event Grid — Обзор

## Что такое Azure Event Grid?


::contentReference[oaicite:0]{index=0}


**Azure Event Grid** — это полностью управляемый сервис маршрутизации событий, предназначенный для построения реактивных, event-driven архитектур по модели **publish-subscribe**.

Он обеспечивает надёжную доставку сообщений в большом масштабе и позволяет строить loosely coupled системы, реагирующие на события.

---

## Ключевые характеристики

- **Serverless и полностью управляемый сервис**  
  Не требуется управлять инфраструктурой

- **Поддержка HTTP и MQTT**
    - **HTTP** — доставка событий и интеграция с облачными сервисами
    - **MQTT** — сценарии IoT и двусторонняя коммуникация

- **Совместимость с CloudEvents v1.0**  
  Используется индустриальный стандарт формата событий

- **Масштабируемость**  
  Поддержка миллионов событий в секунду

- **Модель оплаты Pay-as-you-go**  
  Платите только за обработанные события

- **Высокая доступность**  
  SLA 99.99%

- **Продвинутая фильтрация**  
  Маршрутизация событий на основе содержимого

---

## Архитектурная идея

Event Grid реализует модель:

### Event Grid Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        EVENT PUBLISHERS                          │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │ Azure    │  │ Custom   │  │ Azure    │  │ Partner  │       │
│  │ Services │  │ Apps     │  │ IoT      │  │ Services │       │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘       │
└───────┼─────────────┼─────────────┼─────────────┼──────────────┘
        │             │             │             │
        └─────────────┴─────────────┴─────────────┘
                      │
                      ▼
        ┌─────────────────────────────┐
        │     AZURE EVENT GRID        │
        │  ┌───────────────────────┐  │
        │  │   Event Routing &     │  │
        │  │   Filtering Engine    │  │
        │  └───────────────────────┘  │
        │  ┌───────────────────────┐  │
        │  │   Topics (Endpoints)  │  │
        │  │   - Custom Topics     │  │
        │  │   - System Topics     │  │
        │  │   - Partner Topics    │  │
        │  └───────────────────────┘  │
        └─────────────┬───────────────┘
                      │
        ┌─────────────┴─────────────┐
        │   EVENT SUBSCRIPTIONS     │
        │   (Filters & Routes)      │
        └─────────────┬───────────────┘
                      │
        ┌─────────────┴─────────────────────────┐
        │                                       │
        ▼                                       ▼
┌───────────────────┐               ┌───────────────────┐
│  PUSH DELIVERY    │               │  PULL DELIVERY    │
│                   │               │                   │
│ ┌───────────────┐ │               │ ┌───────────────┐ │
│ │ Azure         │ │               │ │ HTTP Client   │ │
│ │ Functions     │ │               │ │ Applications  │ │
│ ├───────────────┤ │               │ └───────────────┘ │
│ │ Logic Apps    │ │               └───────────────────┘
│ ├───────────────┤ │
│ │ Event Hubs    │ │
│ ├───────────────┤ │
│ │ Service Bus   │ │
│ ├───────────────┤ │
│ │ Webhooks      │ │
│ └───────────────┘ │
└───────────────────┘
```

---

## Основные концепции Event Grid

### 1. Events (События)


::contentReference[oaicite:0]{index=0}


**Event (Событие)** — это минимальная единица информации, которая полностью описывает произошедшее действие в системе.

Событие фиксирует факт изменения состояния, но не содержит бизнес-логики обработки.

---

## Характеристики события

- **Неизменяемость (Immutable)**  
  События описывают уже произошедший факт и не изменяются после публикации

- **Лёгковесность**  
  Максимальный размер — 1 МБ на одно событие

- **Структурированность**  
  Формат JSON согласно схеме CloudEvents или Event Grid Schema

- **Метка времени (Timestamped)**  
  Содержит время генерации события

---

## Архитектурный смысл

Событие обычно содержит:

- Кто инициировал изменение
- Что произошло
- Когда это произошло
- Дополнительные метаданные

Важно: Event Grid передаёт **уведомление о событии**, а не сам объект целиком (например, не сам файл, а факт его создания).

---

## Что важно для AZ-204

- События являются **immutable**
- Поддерживаются стандартные схемы (CloudEvents)
- Максимальный размер события — 1 МБ
- Event Grid маршрутизирует события, но не хранит их длительное время

Если в вопросе говорится о реакции на факт изменения ресурса — это классический сценарий использования Event Grid.

**Common Event Properties:**
```json
{
  "specversion": "1.0",
  "type": "com.example.someevent",
  "source": "/mycontext",
  "subject": "resource/operation",
  "id": "A234-1234-1234",
  "time": "2024-01-15T10:30:00Z",
  "datacontenttype": "application/json",
  "data": {
    "appinfoA": "value",
    "appinfoB": "another value"
  }
}
```

## Event Size and Billing

- Максимальный размер события: **1 MB**
- Тарификация происходит блоками по **64 KB**
- Пример: событие размером 130 KB тарифицируется как 3 операции  
  (192 KB — округление вверх до ближайшего блока 64 KB)

---

### Что это означает

Event Grid тарифицирует события по объёму, а не просто по количеству.

Если размер события превышает 64 KB:
- происходит округление вверх;
- каждое увеличение на 64 KB считается дополнительной операцией.

---

### Архитектурная рекомендация

- Минимизируйте payload события.
- Не передавайте большие данные напрямую — передавайте ссылку на ресурс.
- Используйте lightweight event pattern (event notification, а не data transfer).

---

## 2. Publishers

**Publisher** — это приложение или сервис, который отправляет события в Event Grid.

---

### Типы Publisher’ов

| Тип Publisher | Описание | Примеры |
|---------------|------------|----------|
| **Azure Services** | Встроенные ресурсы Azure, генерирующие события | Storage accounts, Resource Manager, IoT Hub |
| **Custom Applications** | Пользовательские приложения, публикующие события | Web apps, фоновые сервисы, микросервисы |
| **Partner Services** | SaaS-провайдеры, интегрированные с Event Grid | Auth0, SAP, Microsoft Graph API |
| **IoT Devices** | IoT-устройства, публикующие события по MQTT | Датчики, шлюзы, edge-устройства |

---

### Архитектурное значение

Event Grid — это fully managed event routing service, который:

- поддерживает event-driven архитектуру;
- отделяет publisher от subscriber;
- обеспечивает масштабируемую доставку событий;
- поддерживает фильтрацию и маршрутизацию.

Publisher не знает, кто будет обрабатывать событие — это обеспечивает слабую связанность (loose coupling).

---

### Важно для AZ-204

Запомните:

- Максимальный размер события — **1 MB**.
- Тарификация — блоками по **64 KB**.
- Azure services могут быть нативными publisher’ами.
- Custom приложения публикуют события через REST API или SDK.
- Event Grid — ключевой сервис для event-driven архитектуры.

**Publishing Methods:**
```bash
# Publish events using Azure CLI
az eventgrid event publish \
  --topic-name myTopic \
  --resource-group myResourceGroup \
  --events '[{
    "id": "event-001",
    "eventType": "recordInserted",
    "subject": "myapp/vehicles/motorcycles",
    "eventTime": "2024-01-15T10:30:00Z",
    "data": {
      "make": "Ducati",
      "model": "Monster"
    },
    "dataVersion": "1.0"
  }]'
```

### 3. Event Sources

**Event source** — это ресурс, в котором происходит событие.  
Каждый источник событий связан с одним или несколькими типами событий (event types).

---

### Встроенные источники событий (Built-in Event Sources)

| Источник событий | Типичные события | Сценарий использования |
|------------------|------------------|-------------------------|
| **Azure Blob Storage** | `Microsoft.Storage.BlobCreated`, `Microsoft.Storage.BlobDeleted` | Запуск процессов при загрузке или удалении файлов |
| **Azure Resource Manager** | `Microsoft.Resources.ResourceWriteSuccess` | Отслеживание развёртывания ресурсов |
| **Azure Event Hubs** | `Microsoft.EventHub.CaptureFileCreated` | Обработка сохранённых (captured) событий |
| **Azure IoT Hub** | `Microsoft.Devices.DeviceCreated` | Управление жизненным циклом устройств |
| **Azure Media Services** | `Microsoft.Media.JobStateChange` | Мониторинг задач кодирования |
| **Azure Container Registry** | `Microsoft.ContainerRegistry.ImagePushed` | Триггер CI/CD пайплайнов |
| **Azure Service Bus** | `Microsoft.ServiceBus.ActiveMessagesAvailableWithNoListeners` | Мониторинг состояния очередей |
| **Azure App Configuration** | `Microsoft.AppConfiguration.KeyValueModified` | Динамическое обновление конфигурации |

---

### Архитектурное значение

Event Sources позволяют реализовать event-driven архитектуру:

- сервисы реагируют на события, а не опрашивают ресурсы;
- повышается масштабируемость;
- уменьшается связность между компонентами;
- упрощается автоматизация.

---

## 4. Topics

**Topic** — это endpoint, в который publishers отправляют события.  
Topic служит контейнером для группировки связанных событий.

---

### Типы Topics

#### System Topics

- **Определение**: Topic для событий от встроенных Azure-сервисов
- **Создание**: Создаётся автоматически (вручную создавать не нужно)
- **Scope**: Привязан к конкретному ресурсу Azure
- **Имя**: Формируется на основе ID ресурса
- **Жизненный цикл**: Удаляется вместе с исходным ресурсом

---

### Что важно понимать

System Topic создаётся автоматически, когда вы:

- настраиваете подписку на события ресурса;
- включаете поддержку Event Grid для Azure-сервиса.

Вам не нужно вручную управлять таким topic.

---

### Важно для AZ-204

Запомните:

- Event Source — это место возникновения события.
- Topic — это endpoint для приёма событий.
- System Topics создаются автоматически для Azure-ресурсов.
- Они удаляются вместе с ресурсом-источником.
```bash
# Subscribe to system topic events
az eventgrid event-subscription create \
  --name myStorageSubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorageaccount" \
  --endpoint https://myfunction.azurewebsites.net/api/handler \
  --included-event-types Microsoft.Storage.BlobCreated Microsoft.Storage.BlobDeleted
```

#### Custom Topics
- **Definition**: Topics for custom application events
- **Creation**: Explicitly created by users
- **Scope**: Independent resources
- **Naming**: User-defined
- **Flexibility**: Full control over event schema

```bash
# Create custom topic
az eventgrid topic create \
  --name myCustomTopic \
  --resource-group myResourceGroup \
  --location eastus

# Get topic endpoint
az eventgrid topic show \
  --name myCustomTopic \
  --resource-group myResourceGroup \
  --query "endpoint" \
  --output tsv
```
#### Partner Topics

- **Определение**: Topic для событий от SaaS-провайдеров
- **Создание**: Создаётся партнёром, вы активируете
- **Примеры**: Auth0, Microsoft Graph, SAP
- **Сценарий использования**: Интеграция событий сторонних систем

Partner Topic позволяет получать события из внешних сервисов так же, как и из Azure-ресурсов, но без необходимости писать собственную интеграцию с их webhook-механизмами.

---

## Сравнение типов Topic

| Характеристика | System Topics | Custom Topics | Partner Topics |
|----------------|--------------|---------------|----------------|
| **Создание** | Автоматическое | Вручную | Создаёт партнёр |
| **Схема события** | Определена Azure | Определяется пользователем | Определена партнёром |
| **Жизненный цикл** | Привязан к ресурсу | Независимый | Управляется партнёром |
| **Сценарий использования** | События Azure-сервисов | События пользовательских приложений | События сторонних сервисов |
| **Стоимость** | Включена в ресурс | Стандартная тарификация | Зависит от партнёра |

---

### Архитектурное различие

- **System Topic** — для встроенных Azure-событий.
- **Custom Topic** — для ваших приложений.
- **Partner Topic** — для интеграции SaaS и внешних платформ.

Выбор зависит от источника события.

---

## 5. Event Subscriptions

**Event Subscription** определяет:

- какие события из topic нужно получать;
- куда их доставлять;
- какие фильтры применять;
- какие параметры доставки использовать.

Event Subscription связывает:

**Topic → Endpoint (Subscriber)**

Без подписки события не доставляются.

---

### Архитектурное значение

Event Grid реализует модель:

- Publisher → Topic → Subscription → Subscriber

Это обеспечивает:

- слабую связанность;
- гибкую маршрутизацию;
- возможность добавлять новых подписчиков без изменения publisher.

---

### Важно для AZ-204

Запомните:

- Topic — это контейнер событий.
- Subscription — это правило маршрутизации.
- Без Event Subscription события никуда не отправляются.
- Разные подписчики могут получать разные события из одного Topic.
**Subscription Configuration:**

```json
{
  "name": "mySubscription",
  "properties": {
    "destination": {
      "endpointType": "WebHook",
      "properties": {
        "endpointUrl": "https://myapp.com/api/events"
      }
    },
    "filter": {
      "includedEventTypes": [
        "Microsoft.Storage.BlobCreated"
      ],
      "subjectBeginsWith": "/blobServices/default/containers/images/",
      "subjectEndsWith": ".jpg",
      "advancedFilters": [
        {
          "operatorType": "NumberGreaterThan",
          "key": "data.contentLength",
          "value": 1024
        }
      ]
    },
    "retryPolicy": {
      "maxDeliveryAttempts": 30,
      "eventTimeToLiveInMinutes": 1440
    }
  }
}
```

## Ключевые свойства Event Subscription

| Свойство | Описание | Возможные варианты |
|------------|------------|-------------------|
| **Destination** | Куда отправлять события | Webhook, Azure Function, Event Hubs, Service Bus, Storage Queue, Hybrid Connection |
| **Filter** | Какие события получать | Типы событий, шаблоны subject, расширенные фильтры |
| **Retry Policy** | Поведение повторной доставки | Макс. попыток (1–30), TTL (1–1440 минут) |
| **Dead Letter** | Хранение недоставленных событий | Blob-контейнер в Storage Account |
| **Expiration** | Срок действия подписки | DateTime или продолжительность |

---

### Пояснение

### 1️⃣ Destination

Определяет конечную точку доставки событий.

Наиболее частые варианты:

- **Azure Function** — serverless-обработка
- **Webhook** — вызов внешнего HTTP endpoint
- **Service Bus / Event Hubs** — интеграция с messaging-системами
- **Storage Queue** — асинхронная обработка

---

### 2️⃣ Filter

Позволяет получать только нужные события.

Можно фильтровать:

- по типу события;
- по `subject`;
- по значениям в payload (advanced filters).

Это снижает нагрузку и уменьшает ненужную обработку.

---

### 3️⃣ Retry Policy

Если endpoint недоступен:

- Event Grid автоматически выполняет повторные попытки;
- можно настроить количество попыток;
- TTL (time-to-live) определяет максимальное время повторной доставки.

---

### 4️⃣ Dead Letter

Если событие не удалось доставить после всех попыток, оно отправляется в Blob Storage.

Это позволяет:

- не терять события;
- анализировать ошибки доставки;
- реализовать повторную обработку вручную.

---

### 5️⃣ Expiration

Позволяет задать срок действия подписки.

Полезно для:

- временных интеграций;
- тестовых сценариев;
- динамических подписчиков.

---

### Архитектурное значение

Event Subscription — это гибкий механизм маршрутизации и доставки:

- обеспечивает надёжную доставку;
- поддерживает повторные попытки;
- реализует dead-letter pattern;
- позволяет фильтровать события на уровне инфраструктуры.

---

### Важно для AZ-204

Если в вопросе говорится о:

- фильтрации событий,
- настройке повторной доставки,
- сохранении недоставленных событий,
- выборе endpoint,

— решение связано с настройкой **Event Subscription properties**.

**Create Event Subscription:**

```bash
# Subscribe with Azure Function endpoint
az eventgrid event-subscription create \
  --name imageProcessingSubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage" \
  --endpoint-type azurefunction \
  --endpoint "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Web/sites/myFunctionApp/functions/processImage" \
  --included-event-types Microsoft.Storage.BlobCreated \
  --subject-begins-with "/blobServices/default/containers/images/" \
  --subject-ends-with ".jpg"
```

## 6. Event Handlers

**Event Handler** — это конечная точка (destination), куда доставляются события.  
Handler обрабатывает или реагирует на полученные события.

Event Grid доставляет события подписчику через настроенную Event Subscription.

---

## Поддерживаемые Event Handlers

| Тип обработчика | Сценарий использования | Способ доставки | Ограничения |
|-----------------|------------------------|------------------|-------------|
| **Azure Functions** | Serverless-обработка событий | Прямой вызов | Таймаут 230 секунд |
| **Webhooks** | Пользовательские HTTP endpoint | HTTP POST | Требуется валидация endpoint |
| **Logic Apps** | Автоматизация workflow | Прямой триггер | Таймаут 120 секунд |
| **Azure Event Hubs** | Потоковая обработка событий | Push в поток | Нет |
| **Azure Service Bus** | Очереди и topic | Push в очередь/topic | 256 KB размер сообщения |
| **Storage Queues** | Простая очередь | Push в очередь | 64 KB размер сообщения |
| **Hybrid Connections** | On-premises endpoint | Через Azure Relay | Требуется настройка Relay |

---

### Что важно понимать

- Event Grid использует модель **push** — события отправляются подписчику.
- Webhook должен подтвердить владение endpoint (validation handshake).
- Таймауты означают максимальное время ожидания ответа от обработчика.
- Если обработчик не отвечает, включается retry policy.

---

### Архитектурные рекомендации

- Для serverless-сценариев используйте **Azure Functions**.
- Для сложных интеграций — **Logic Apps**.
- Для высокой пропускной способности — **Event Hubs**.
- Для гарантированной доставки и очередей — **Service Bus**.
- Для on-prem интеграции — **Hybrid Connections**.

---

### Важно для AZ-204

Запомните:

- Event Grid работает по push-модели.
- Webhook требует подтверждения при создании подписки.
- Таймаут Azure Functions — 230 секунд.
- Storage Queue ограничен 64 KB.
- Service Bus ограничен 256 KB.

Экзамен часто проверяет выбор правильного handler для конкретного сценария.
**Azure Function Handler Example (C#):**

```csharp
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.EventGrid;
using Microsoft.Extensions.Logging;
using Azure.Messaging.EventGrid;

public static class BlobCreatedHandler
{
    [FunctionName("BlobCreatedHandler")]
    public static void Run(
        [EventGridTrigger] EventGridEvent eventGridEvent,
        ILogger log)
    {
        log.LogInformation($"Event Type: {eventGridEvent.EventType}");
        log.LogInformation($"Event Subject: {eventGridEvent.Subject}");
        log.LogInformation($"Event Data: {eventGridEvent.Data}");
        
        // Process the event
        if (eventGridEvent.EventType == "Microsoft.Storage.BlobCreated")
        {
            var blobData = eventGridEvent.Data.ToObjectFromJson<BlobCreatedEventData>();
            log.LogInformation($"Blob URL: {blobData.Url}");
            
            // Your processing logic here
        }
    }
}
```

## Требования к Webhook Handler

При использовании Webhook в качестве Event Handler необходимо соблюдать следующие требования:

---

### 1️⃣ Подтверждение endpoint (Endpoint Validation)

При создании подписки Event Grid отправляет **validation event**.

Webhook обязан:

- обработать событие валидации;
- вернуть validation code в ответе;
- подтвердить владение endpoint.

Без успешной валидации подписка создана не будет.

---

### 2️⃣ HTTP 200 Response

Webhook должен:

- вернуть HTTP 200 (OK);
- сделать это в течение **30 секунд**.

Если ответ не получен или превышен таймаут:

- считается, что доставка не удалась;
- запускается retry policy.

---

### 3️⃣ TLS / SSL

Endpoint должен:

- использовать HTTPS;
- иметь действительный сертификат;
- поддерживать современную версию TLS.

Небезопасные HTTP endpoint не поддерживаются.

---

### 4️⃣ Идемпотентность (Idempotency)

Event Grid может повторно отправить событие при сбое доставки.

Webhook должен:

- корректно обрабатывать дубликаты;
- не выполнять одну и ту же операцию повторно;
- использовать `eventId` для проверки уникальности.

---

### Архитектурное значение

Webhook должен быть:

- устойчивым к повторной доставке;
- быстрым в обработке;
- безопасным (TLS);
- способным подтверждать владение endpoint.

Рекомендуется:

- выполнять тяжёлую обработку асинхронно;
- возвращать 200 как можно быстрее;
- использовать очередь внутри приложения.

---

### Важно для AZ-204

Если в вопросе говорится о:

- валидации Webhook,
- необходимости вернуть 200 в течение 30 секунд,
- использовании HTTPS,
- обработке дубликатов,

— речь идёт о требованиях к Webhook handler в Event Grid.

```python
# Python Flask webhook example
from flask import Flask, request, jsonify

app = Flask(__name__)

@app.route('/api/events', methods=['POST'])
def handle_event():
    events = request.json
    
    for event in events:
        # Handle validation event
        if event['eventType'] == 'Microsoft.EventGrid.SubscriptionValidationEvent':
            validation_code = event['data']['validationCode']
            return jsonify({'validationResponse': validation_code})
        
        # Handle business events
        elif event['eventType'] == 'Microsoft.Storage.BlobCreated':
            blob_url = event['data']['url']
            print(f"New blob created: {blob_url}")
            # Process the event
    
    return jsonify({'status': 'success'}), 200
```

---

## Модели доставки событий

### Push Delivery (по умолчанию)

Event Grid **сам отправляет (push)** события в настроенные endpoint’ы.

---

### Характеристики

- **Автоматическая доставка**  
  Событие отправляется сразу после публикации.

- **At-least-once delivery**  
  Гарантируется доставка как минимум один раз (возможны дубликаты).

- **Retry Policy**  
  Используется экспоненциальный backoff  
  (от 30 секунд до 1 дня).

- **Dead Lettering**  
  Недоставленные события сохраняются в Storage для последующей обработки.

---

### Что это означает

- Publisher не ждёт подтверждения от подписчика.
- Event Grid самостоятельно управляет повторными попытками.
- Подписчик должен быть идемпотентным.

Push-модель обеспечивает высокую масштабируемость и низкую задержку.

---

### Лучшие сценарии использования

- **Azure Functions**
- **Logic Apps**
- Webhook с высокой доступностью
- Event-driven архитектуры

---

### Архитектурное значение

Push delivery:

- уменьшает сложность subscriber’а;
- снижает задержку обработки;
- подходит для реактивных систем;
- поддерживает автоматическое масштабирование.

---

### Важно для AZ-204

Запомните:

- Event Grid по умолчанию использует push-модель.
- Доставка — at-least-once.
- Повторная отправка выполняется автоматически.
- Subscriber должен корректно обрабатывать дубликаты.

**Push Delivery Flow:**
```
Publisher → Event Grid → [Filter] → [Retry if needed] → Handler
                ↓ (if failed after retries)
           Dead Letter Location
```

### Pull Delivery

Consumers pull events from Event Grid on demand.

**Characteristics:**
- **On-Demand**: Consumer controls when to receive events
- **Batch Processing**: Retrieve multiple events at once
- **Client Acknowledgment**: Consumer acknowledges processed events
- **No Endpoint**: No webhook validation required

**Best For:**
- Batch processing scenarios
- Applications behind firewalls
- Client-controlled event processing

**Pull Delivery Code Example (C#):**

```csharp
using Azure;
using Azure.Messaging.EventGrid.Namespaces;

var endpoint = new Uri("https://mynamespace.eastus-1.eventgrid.azure.net");
var credential = new AzureKeyCredential(topicKey);
var client = new EventGridReceiverClient(endpoint, "mytopic", "mysubscription", credential);

// Receive events
ReceiveResult result = await client.ReceiveAsync(maxEvents: 10, maxWaitTime: TimeSpan.FromSeconds(30));

foreach (ReceiveDetails details in result.Value)
{
    CloudEvent cloudEvent = details.Event;
    BrokerProperties brokerProperties = details.BrokerProperties;
    
    // Process the event
    Console.WriteLine($"Event Type: {cloudEvent.Type}");
    Console.WriteLine($"Event Data: {cloudEvent.Data}");
    
    // Acknowledge the event
    await client.AcknowledgeAsync(new[] { brokerProperties.LockToken });
}
```

---

## Безопасность в Event Grid

### Аутентификация

#### Аутентификация Publisher

Publisher должен подтвердить право публиковать события в topic.

Поддерживаются следующие механизмы:

- **Access Keys**  
  Общие ключи доступа для Custom Topics.

- **SAS Tokens**  
  Токены с ограниченным сроком действия.

- **Azure AD**  
  OAuth 2.0, включая Managed Identities (рекомендуемый способ).

---

#### Аутентификация Handler

Когда события доставляются подписчику:

- **Endpoint Validation**  
  Подтверждение владения endpoint при создании подписки.

- **Event Delivery Authentication**  
  Возможность передавать дополнительные заголовки аутентификации.

---

### Авторизация

Event Grid использует **Azure RBAC** для управления доступом.

---

### Роли Azure RBAC

| Роль | Права | Сценарий использования |
|------|--------|------------------------|
| **Event Grid Contributor** | Полный контроль над ресурсами Event Grid | Администраторы |
| **Event Grid Data Sender** | Публикация событий в topic | Приложения |
| **Event Grid Subscription Reader** | Чтение подписок | Мониторинг и аудит |
| **Event Grid Subscription Contributor** | Управление подписками | Операционная команда |

---

### Архитектурные рекомендации

- Для production используйте **Azure AD + Managed Identity** вместо access keys.
- Ограничивайте права по принципу least privilege.
- Используйте SAS только для временного доступа.
- Для webhook’ов применяйте дополнительную аутентификацию (например, секрет в заголовке).

---

### Важно для AZ-204

Запомните:

- Publisher аутентифицируется через Access Key, SAS или Azure AD.
- Handler подтверждает endpoint при создании подписки.
- RBAC управляет доступом к ресурсам Event Grid.
- Для безопасной архитектуры предпочтителен Azure AD.
```bash
# Grant Data Sender role to managed identity
az role assignment create \
  --role "EventGrid Data Sender" \
  --assignee <managed-identity-object-id> \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic"
```

### Network Security

- **Private Endpoints**: Access topics via VNet
- **IP Filtering**: Restrict publisher IP addresses
- **Managed Identity**: No credentials in code

---

## Common Use Cases

### 1. Serverless Application Integration

**Scenario**: Process uploaded images automatically

```
Azure Blob Storage → Event Grid → Azure Functions → Thumbnail Creation
                                 → Azure Functions → Image Recognition
                                 → Azure Functions → Database Update
```

### 2. Ops Automation

**Scenario**: Auto-tag resources when created

```
Azure Resource Manager → Event Grid → Azure Function → Apply Tags
                                    → Logic App → Send Notification
```

### 3. Application Integration

**Scenario**: Order processing pipeline

```
Order API → Event Grid (Custom Topic) → Azure Function (Validate Order)
                                      → Service Bus Queue (Inventory)
                                      → Event Hubs (Analytics)
```

### 4. IoT Telemetry

**Scenario**: Process device telemetry

```
IoT Devices (MQTT) → Event Grid → Azure Functions → Time Series Insights
                                 → Event Hubs → Stream Analytics
```

---

## Event Grid vs. Другие сервисы обмена сообщениями в Azure

| Характеристика | **Event Grid** | **Event Hubs** | **Service Bus** |
|---------------|---------------|---------------|----------------|
| **Паттерн** | Pub/Sub (реактивная модель) | Streaming (Big Data) | Очередь сообщений |
| **Размер сообщения** | 1 MB | 1 MB | 256 KB (Premium: до 100 MB) |
| **Гарантия порядка** | Нет | В пределах partition | FIFO (через sessions) |
| **Модель доставки** | Push + Pull | Pull | Pull |
| **Хранение сообщений** | Нет (моментальная доставка) | 1–90 дней | До 14 дней |
| **Пропускная способность** | Миллионы/сек | Миллионы/сек | Тысячи/сек |
| **Задержка** | Менее секунды | Почти в реальном времени | Низкая |
| **Сценарий использования** | Уведомления о событиях | Приём телеметрии | Транзакционные сообщения |
| **Фильтрация** | Расширенная фильтрация | Consumer groups | Message filters |

---

## Когда использовать Event Grid

- ✅ Реагировать на изменения состояния ресурсов Azure
- ✅ Интегрировать несколько сервисов через event-driven паттерн
- ✅ Создавать serverless-приложения
- ✅ Использовать продвинутую фильтрацию и маршрутизацию
- ✅ Нужна push-модель доставки

---

## Когда НЕ использовать Event Grid

- ❌ Требуется строгая гарантия порядка сообщений
- ❌ Нужна долговременная ретенция событий
- ❌ Требуются сложные workflow или транзакционность (используйте Service Bus)
- ❌ Нужен высоконагруженный потоковый ingestion (используйте Event Hubs)

---

## Архитектурный выбор

- **Event Grid** → события и уведомления
- **Event Hubs** → поток телеметрии и big data ingestion
- **Service Bus** → гарантированная доставка и бизнес-транзакции

---

### Важно для AZ-204

На экзамене часто проверяют:

- различие между push и pull моделями;
- отсутствие гарантии порядка в Event Grid;
- отсутствие хранения событий;
- выбор правильного сервиса под конкретный сценарий.

Главный критерий:  
**События → Event Grid**  
**Поток данных → Event Hubs**  
**Очередь и транзакции → Service Bus**
## Quick Start Example

### Step 1: Create Custom Topic

```bash
# Create resource group
az group create --name rg-eventgrid --location eastus

# Create custom topic
az eventgrid topic create \
  --name mytopic \
  --resource-group rg-eventgrid \
  --location eastus

# Get topic endpoint and key
TOPIC_ENDPOINT=$(az eventgrid topic show \
  --name mytopic \
  --resource-group rg-eventgrid \
  --query "endpoint" --output tsv)

TOPIC_KEY=$(az eventgrid topic key list \
  --name mytopic \
  --resource-group rg-eventgrid \
  --query "key1" --output tsv)
```

### Step 2: Create Event Subscription

```bash
# Subscribe with webhook endpoint
az eventgrid event-subscription create \
  --name mysubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/rg-eventgrid/providers/Microsoft.EventGrid/topics/mytopic" \
  --endpoint https://mywebhook.azurewebsites.net/api/events \
  --included-event-types MyApp.Orders.OrderCreated
```

### Step 3: Publish Event

```bash
# Publish event
az eventgrid event publish \
  --topic-name mytopic \
  --resource-group rg-eventgrid \
  --events '[
    {
      "id": "order-001",
      "eventType": "MyApp.Orders.OrderCreated",
      "subject": "orders/motorcycles",
      "eventTime": "2024-01-15T10:30:00Z",
      "data": {
        "orderId": "12345",
        "customerId": "CUST-001",
        "amount": 15000.00
      },
      "dataVersion": "1.0"
    }
  ]'
```

---

## Best Practices

### Design Patterns

1. **Event Naming**: Use clear, hierarchical event types
   ```
   ✅ MyApp.Orders.OrderCreated
   ❌ orderCreated
   ```

2. **Subject Hierarchy**: Structure subjects for easy filtering
   ```
   ✅ /orders/region/west/store/101
   ❌ order-west-101
   ```

3. **Idempotency**: Design handlers to process duplicate events safely
   ```csharp
   // Store processed event IDs
   if (await processedEvents.Contains(eventId))
   {
       return; // Already processed
   }
   ```

4. **Error Handling**: Implement proper retry and dead-letter handling

5. **Event Versioning**: Include data version for schema evolution
   ```json
   {
     "dataVersion": "2.0",
     "data": { /* new schema */ }
   }
   ```

## Оптимизация производительности

- **Batch Publishing**  
  Отправляйте несколько событий в одном HTTP-запросе, чтобы снизить накладные расходы.

- **Async Handlers**  
  Обрабатывайте события асинхронно, чтобы быстрее возвращать 200 OK и избежать повторной доставки.

- **Parallel Processing**  
  Используйте несколько экземпляров обработчиков для масштабирования.

- **Filter Early**  
  Настраивайте фильтры на уровне Event Subscription, чтобы уменьшить количество ненужных событий.

---

## Best Practices по безопасности

- ✅ Используйте **Managed Identity** вместо access keys
- ✅ Включайте **Private Endpoints** для чувствительных нагрузок
- ✅ Корректно реализуйте валидацию Webhook endpoint
- ✅ Используйте HTTPS для всех endpoint’ов
- ✅ Применяйте принцип **least privilege** через RBAC

---

# Советы к экзамену AZ-204

## Ключевые концепции

1. **Event Grid предназначен для реактивного программирования**  
   Push-модель распределения событий.

2. **System Topics создаются автоматически**,  
   **Custom Topics требуют ручного создания**.

3. **Event Subscription фильтрует и маршрутизирует события** к обработчикам.

4. **CloudEvents 1.0** — предпочтительный стандарт схемы событий.

5. **Максимальный размер события — 1 MB**,  
   тарификация блоками по 64 KB.

---

## Частые экзаменационные сценарии

### Сценарий 1
Триггер Azure Function при загрузке blob

- ✅ Использовать Event Grid с событием BlobCreated
- ❌ Не использовать polling или таймеры

---

### Сценарий 2
Обработка событий от стороннего SaaS

- ✅ Использовать Partner Topics
- ❌ Не реализовывать собственную интеграцию

---

### Сценарий 3
Требуется гарантированный порядок сообщений

- ❌ Event Grid не гарантирует порядок
- ✅ Использовать Service Bus с sessions

---

### Сценарий 4
Высоконагруженный ingestion телеметрии

- ❌ Event Grid не предназначен для потоковой передачи
- ✅ Использовать Event Hubs

---

## Финальный акцент

На экзамене важно:

- выбрать правильный сервис под задачу;
- помнить про отсутствие гарантии порядка в Event Grid;
- понимать push-модель доставки;
- отличать System, Custom и Partner Topics;
- знать ограничения по размеру события и тарификации.
### Important Commands

```bash
# Create custom topic
az eventgrid topic create --name <name> --resource-group <rg> --location <location>

# Create event subscription
az eventgrid event-subscription create --name <name> --source-resource-id <id> --endpoint <url>

# Publish event
az eventgrid event publish --topic-name <name> --resource-group <rg> --events <json>

# List event types
az eventgrid topic-type list

# Grant access
az role assignment create --role "EventGrid Data Sender" --assignee <identity>
```

## Чек-лист по устранению неполадок

- ❓ События не доставляются?  
  Проверьте валидацию endpoint и настройки retry policy.

- ❓ Handler завершает работу по таймауту?  
  Убедитесь, что ответ возвращается в течение 30 секунд.

- ❓ События «пропадают»?  
  Проверьте фильтры и конфигурацию Event Subscription.

- ❓ Ошибка аутентификации?  
  Проверьте access keys или корректность настройки managed identity.

- ❓ Нет dead-letter хранения?  
  Убедитесь, что настроен blob-контейнер для недоставленных событий.

---

## Что помнить для экзамена

- **At-least-once delivery**  
  События могут быть доставлены более одного раза.

- **30 секунд для Webhook**  
  Обработчик должен быстро вернуть ответ.

- **Retry policy по умолчанию**  
  До 30 попыток, TTL — до 24 часов.

- **System Topics**  
  Привязаны к жизненному циклу ресурса.

- **Advanced filtering**  
  Поддерживается до 25 условий на подписку.

- **RBAC роли**  
  Знать роли: Data Sender, Contributor, Subscription Reader.

- **Нет хранения событий**  
  Event Grid не хранит события — для ретенции используйте Event Hubs.

---

# Итоги

Azure Event Grid реализует **event-driven архитектуру**, где есть:

- **Publishers**, отправляющие события
- **Topics**, группирующие события
- **Subscriptions**, фильтрующие и маршрутизирующие события
- **Handlers**, обрабатывающие события

---

## Ключевые выводы

- Используйте **System Topics** для событий Azure-ресурсов.
- Используйте **Custom Topics** для событий приложений.
- Используйте **Partner Topics** для интеграции сторонних сервисов.
- Применяйте **фильтрацию**, чтобы получать только нужные события.
- Настраивайте **retry policy** для надёжной доставки.
- Используйте **dead-letter storage** для анализа ошибок.
- Реализуйте **идемпотентные обработчики**, чтобы корректно обрабатывать дубликаты.

---

## Архитектурный акцент

Event Grid идеально подходит для:

- реактивного программирования;
- serverless-приложений;
- интеграции сервисов;
- автоматизации процессов при изменении состояния ресурсов.

Если задача — реагировать на событие «здесь и сейчас»,  
Event Grid — правильный выбор.

Различие между:
SendGrid action
SendGrid binding

Action → Logic Apps
Binding → Azure Functions

🔥 Ключевая архитектурная идея

Bindings в Azure Functions = декларативный способ подключения к сервисам.

Есть:
Cosmos DB trigger
Event Hub trigger
Service Bus trigger
SendGrid output binding
Blob input/output binding
Это как dependency injection для внешних сервисов.

🎯 Простыми словами

SendGrid binding = встроенный адаптер для отправки email из Function.

Azure Event Grid (и в целом в event-driven архитектуре):

Event Subscription содержит:
✔ к какому topic он подписывается
✔ куда отправлять события (endpoint обработчика)
✔ фильтрацию событий — какие типы событий он хочет получать

В Azure Event Grid Event Domain используется в сценариях, где:
есть много клиентов / арендаторов (multi-tenant scenario)
требуется централизованное управление
нужно азграничить доступ между подписчиками
Event Domain позволяет:
✔ централизованно управлять большим количеством topics
✔ изолировать подписчиков по безопасности
✔ делегировать управление подписками разным командам или клиентам
✔ применять RBAC на уровне домена и отдельных domain topic

В Azure Event Grid можно использовать Advanced Filters на уровне подписки.
Они позволяют:
Фильтровать по строковым значениям (department == "Finance")
Фильтровать по строке (fileType == "CSV")
Фильтровать по числовым значениям (transactionAmount > 10000)
Комбинровать условия через логическое AND
Это означает:
Событие вообще не будет доставлено функции, если не проходит фильтр
Функция не будет запускаться лишний раз
Снижается количество executions
Снижается стоимость
Уменшается нагрузка

В связке с Azure Event Hubs именно Azure Stream Analytics (ASA) используется для:
Потоковой обработки событий в реальном времени
Сложной фильтрации данных
Агрегаций
Оконных функций (tumbling / hopping / sliding windows)
Интеграции с Azure Machine Learning для применения ML-моделей к потоковым данным
Stream Analytics позволяет:
Выполнять SQLподобные запросы к потоку
Фильтровать события по условиям
Подключать ML-функции через Azure ML endpoints
Почему остальныеварианты неверны

A. Azure Event Grid ❌
Event Grid — это маршрутизатор событий, а не сервис потоковой аналитики.

B. Azure Synapse Data Explorer ❌
Используется для аналитики больших данных, но не является основным real-time фильтрующим сервисом в связке с Event Hubs.

C. Azure SAS tokens ❌
Это механизм авторизации, а не сервис обработки событий.

В паттерне асинхронного взаимодействия (queue-based communication), 
который часто используется в микросервисной архитектуре:
Producer (отправитель) помещает сообщение в очередь.
Сообщение хранится в Azure Queue Storage.
Consumer (получатель) читает сообщение и обрабатывает его.
После успешной обработки сообщение удаляется из очереди.
Это обеспечивает:
Слабую связанность (loose coupling)
Масштабируемость
Надёжность
Повышенную устойчивость к сбоям
Особенно эо важно для микросервисов в Kubernetes, где сервисы могут перезапускаться и 
масштабироваться динамически.

В Azure Queue Storage метод peek:
Позволяет посмотреть сообщение
Не удаляет его из очереди
Не блокирует его
Не делает его невидимым для других consumers
Не изменяет порядок сообщений
То есть сообщение остаётся в очереди как есть.

При создании Azure Service Bus queue или topic необходимо учитывать параметры конфигурации, которые зависят от выбранного pricing tier (Basic, Standard, Premium):
Во время provisioning задаются:
Message storage quota (например, до 5 GB в Standard)
Max delivery count
Default message time-to-live (TTL)
Lock duration
Включение partitioning
Включение duplicate detection
Включение sessions