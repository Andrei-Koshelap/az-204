# Изучение интеграции Dapr с Azure Container Apps

## Ключевые понятия

- **Dapr (Distributed Application Runtime)** — рантайм для построения распределённых микросервисных приложений
- **Building blocks** — готовые API для типовых паттернов микросервисной архитектуры
- **Components** — подключаемые реализации (Azure, AWS и др.)
- **Sidecar-архитектура** — Dapr работает рядом с вашим контейнером как отдельный процесс

---

## Что такое Dapr?

**Distributed Application Runtime** — переносимый событийно-ориентированный runtime для распределённых систем.

Основные характеристики:

- **Упрощает микросервисы** — реализует распространённые паттерны через стандартные API
- **Независим от языка** — взаимодействие через HTTP или gRPC
- **Независим от платформы** — может работать в облаке, на edge-устройствах и on-premises
- **Open-source проект** — развивается под эгидой CNCF

> 💡 Dapr абстрагирует инфраструктуру, позволяя разработчику работать с единым API независимо от используемого облачного провайдера.

---

## Преимущества Dapr в Azure Container Apps

✅ **Без управления инфраструктурой** — платформа управляет Dapr автоматически  
✅ **Автоматический sidecar** — включается одной настройкой  
✅ **Минимальные изменения кода** — взаимодействие через HTTP/gRPC  
✅ **Готовые компоненты** — преднастроенная интеграция с сервисами Azure  
✅ **Поддержка микросервисных паттернов** — service invocation, pub/sub, управление состоянием

---

## Дополнительные важные моменты для AZ-204

- Dapr запускается как sidecar-контейнер рядом с основным контейнером приложения.
- Взаимодействие между сервисами происходит через локальный HTTP/gRPC endpoint.
- В Azure Container Apps включение Dapr относится к изменениям уровня ревизии.
- Dapr помогает уменьшить связность (coupling) между сервисами за счёт абстракции инфраструктуры.

---

## Dapr Architecture

### Sidecar Pattern
```
Container App Environment
├── Your Container (App Code)
│   ├── Port 8080 (your app)
│   └── Calls Dapr API
│       ↓
└── Dapr Sidecar (Auto-injected)
    ├── Port 3500 (HTTP API)
    ├── Port 50001 (gRPC API)
    └── Connects to Azure Services
        ├── Azure Service Bus
        ├── Azure Cosmos DB
        ├── Azure Storage
        └── Application Insights
```

### Communication Flow
```
Your App → HTTP localhost:3500 → Dapr Sidecar → Azure Service
                                      ↓
                               (Handles retries,
                                timeouts,
                                telemetry)
```

## Dapr Building Blocks

### Основные API (Core APIs)

| Building Block        | Назначение | Типовой сценарий использования |
|-----------------------|------------|--------------------------------|
| **Service Invocation** | Вызов сервисов по имени | Взаимодействие между микросервисами |
| **State Management**   | CRUD для key/value состояния | Сессии, пользовательские настройки |
| **Pub/Sub**            | Публикация и подписка на события | Event-driven архитектура |
| **Bindings**           | Интеграция с внешними системами | Очереди, файловые хранилища |
| **Secrets**            | Безопасное получение секретов | Пароли БД, API-ключи |
| **Actors**             | Stateful virtual actors | IoT-устройства, игровые объекты |
| **Observability**      | Распределённая трассировка | Мониторинг межсервисных вызовов |
| **Configuration**      | Динамическое получение конфигурации | Feature flags, настройки |

---

## Наиболее часто используемые возможности в Azure Container Apps

В контексте Azure Container Apps чаще всего используются следующие building blocks:

- **Service Invocation**
- **Pub/Sub**
- **State Management**
- **Secrets**

---

### 1. Service Invocation

Позволяет вызывать другой сервис по его **app ID**, без знания его физического адреса.

Основные особенности:

- Dapr автоматически выполняет service discovery.
- Вызовы происходят через HTTP или gRPC.
- Поддерживается безопасное взаимодействие между сервисами.
- Упрощает внутреннюю коммуникацию в микросервисной архитектуре.

> 💡 В Azure Container Apps каждый сервис может иметь свой app ID, который используется Dapr для маршрутизации вызовов.

---

### Важно для AZ-204

- Service Invocation уменьшает связанность между сервисами.
- Нет необходимости управлять внутренними URL-адресами.
- Dapr работает как sidecar и перехватывает вызовы.
- Включение Dapr относится к изменениям уровня ревизии.

---

```bash
# Your code calls Dapr sidecar
curl http://localhost:3500/v1.0/invoke/order-service/method/create-order \
  -H "Content-Type: application/json" \
  -d '{"productId": "123", "quantity": 2}'

# Dapr resolves 'order-service' and calls it
```

#### 2. Pub/Sub Messaging
**Publish events**:

```bash
# Publish to topic
curl http://localhost:3500/v1.0/publish/pubsub/orders \
  -H "Content-Type: application/json" \
  -d '{"orderId": "123", "status": "completed"}'
```

**Subscribe to topics**:

```bash
# Your app exposes endpoint for Dapr
# Dapr calls your app when messages arrive
POST http://your-app/orders
```

#### 3. State Management
**Store/retrieve state**:

```bash
# Save state
curl -X POST http://localhost:3500/v1.0/state/statestore \
  -H "Content-Type: application/json" \
  -d '[{"key": "session-123", "value": {"userId": "456"}}]'

# Get state
curl http://localhost:3500/v1.0/state/statestore/session-123
```

#### 4. Bindings
**Trigger from queue**:

```bash
# Dapr polls queue and calls your app
POST http://your-app/queue-message
```

**Output to storage**:

```bash
# Write to blob storage
curl -X POST http://localhost:3500/v1.0/bindings/blob-storage \
  -d '{"data": "file content", "metadata": {"blobName": "file.txt"}}'
```

## Dapr Components

### Что такое Components?

**Components** — это подключаемые реализации (pluggable implementations) для building blocks Dapr.  
Они определяют, какая конкретная технология используется под капотом.

Примеры:

- **State store** → Azure Cosmos DB, Redis, Azure Table Storage
- **Pub/Sub** → Azure Service Bus, Event Hubs, Redis Streams
- **Secret store** → Azure Key Vault, Kubernetes secrets
- **Bindings** → Azure Storage Queue, Azure Blob Storage

> 💡 Dapr абстрагирует инфраструктуру: код работает с универсальным API, а конкретная реализация задаётся через компонент.

---

## Что важно понимать

- Компонент определяет **конкретный backend-сервис**, но приложение взаимодействует только через Dapr API.
- Можно заменить реализацию (например, Redis → Cosmos DB) без изменения бизнес-логики.
- Компоненты позволяют строить переносимые (portable) архитектуры.

---

## Component Definition

Компоненты Dapr описываются через **YAML-конфигурацию**.

В конфигурации указывается:

- Тип компонента (state, pubsub, binding, secret store)
- Версия API
- Конкретная реализация
- Метаданные подключения
- Настройки аутентификации

В Azure Container Apps компоненты обычно создаются и управляются через:

- Azure CLI
- Azure Portal
- Infrastructure as Code (Bicep / ARM / Terraform)

> ⚠️ Для AZ-204 важно помнить: компоненты — это не код приложения, а инфраструктурная конфигурация.

---

## Экзаменационный фокус

- Building block — это API.
- Component — это конкретная реализация этого API.
- Один building block может иметь разные компоненты.
- Замена компонента не требует изменения логики приложения.


```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.azure.cosmosdb
  version: v1
  metadata:
  - name: url
    value: "https://myaccount.documents.azure.com:443/"
  - name: masterKey
    secretRef: cosmos-key
  - name: database
    value: "mydb"
  - name: collection
    value: "state"
scopes:
- order-service
- inventory-service
```

### Component Scopes

**Limit component access** to specific apps:

```yaml
scopes:
- order-service     # Only order-service can use this component
- payment-service   # Only payment-service can use this component
```

**No scopes** = Available to all apps in environment

### Azure Components

#### State Store: Azure Cosmos DB
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: statestore
spec:
  type: state.azure.cosmosdb
  version: v1
  metadata:
  - name: url
    value: "https://myaccount.documents.azure.com:443/"
  - name: masterKey
    secretRef: cosmos-key
  - name: database
    value: "statedb"
```

#### Pub/Sub: Azure Service Bus
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: pubsub
spec:
  type: pubsub.azure.servicebus.topics
  version: v1
  metadata:
  - name: connectionString
    secretRef: servicebus-connection
```

#### Secret Store: Azure Key Vault
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: azurekeyvault
spec:
  type: secretstores.azure.keyvault
  version: v1
  metadata:
  - name: vaultName
    value: "myvault"
  - name: azureClientId
    value: "<managed-identity-client-id>"
```

#### Binding: Azure Storage Queue
```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: queue-binding
spec:
  type: bindings.azure.storagequeues
  version: v1
  metadata:
  - name: accountName
    value: "mystorageaccount"
  - name: accountKey
    secretRef: storage-key
  - name: queue
    value: "myqueue"
  - name: ttlInSeconds
    value: "60"
```

## Enable Dapr in Container Apps

### At App Creation

```bash
# Enable Dapr when creating container app
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myapp:latest \
  --enable-dapr \
  --dapr-app-id myapp \
  --dapr-app-port 8080 \
  --dapr-app-protocol http
```

### Enable on Existing App

```bash
# Enable Dapr on existing app
az containerapp dapr enable \
  --name myapp \
  --resource-group myResourceGroup \
  --dapr-app-id myapp \
  --dapr-app-port 8080 \
  --dapr-app-protocol http
```

### Параметры конфигурации Dapr

| Параметр | Описание | Обязательный |
|----------|----------|--------------|
| `--enable-dapr` | Включает Dapr sidecar | ✅ Да |
| `--dapr-app-id` | Уникальный идентификатор приложения | ✅ Да |
| `--dapr-app-port` | Порт, на котором слушает ваше приложение | Нет (если только исходящие вызовы) |
| `--dapr-app-protocol` | Протокол: `http` или `grpc` | Нет (по умолчанию: http) |

---

## Пояснения к параметрам

### `--enable-dapr`
Активирует запуск sidecar-контейнера Dapr рядом с вашим приложением.  
Без этого флага Dapr работать не будет.

### `--dapr-app-id`
Уникальный идентификатор сервиса внутри среды Container Apps.  
Используется для:
- service invocation
- маршрутизации запросов
- взаимодействия между микросервисами

> ⚠️ App ID должен быть уникальным в пределах Container Apps Environment.

### `--dapr-app-port`
Указывает порт, на котором работает ваше приложение.  
Требуется, если:
- сервис принимает входящие вызовы через Dapr

Не обязателен, если приложение только публикует события или вызывает другие сервисы.

### `--dapr-app-protocol`
Определяет протокол взаимодействия:
- `http`
- `grpc`

По умолчанию используется HTTP.

---

## Важно для AZ-204

- Включение Dapr относится к изменениям уровня ревизии.
- App ID — ключевой параметр для межсервисного взаимодействия.
- Sidecar автоматически управляется платформой, дополнительная инфраструктура не требуется.
- Dapr работает локально через sidecar, а не напрямую между контейнерами.


### ARM Template Example

```json
{
  "properties": {
    "configuration": {
      "dapr": {
        "enabled": true,
        "appId": "order-service",
        "appProtocol": "http",
        "appPort": 8080
      }
    },
    "template": {
      "containers": [
        {
          "image": "myapp:latest",
          "name": "order-service"
        }
      ]
    }
  }
}
```

## Using Dapr APIs

### Service Invocation

#### Call Another Service
```bash
# From your app code
curl http://localhost:3500/v1.0/invoke/inventory-service/method/check-stock/123 \
  -H "Content-Type: application/json"

# Dapr:
# 1. Resolves 'inventory-service' to actual endpoint
# 2. Handles service discovery
# 3. Adds retries and timeouts
# 4. Provides distributed tracing
```

#### Service Invocation Format
```
http://localhost:3500/v1.0/invoke/<app-id>/method/<method-name>
```

### Pub/Sub Messaging

#### Publish Event
```bash
# Publish to topic
curl -X POST http://localhost:3500/v1.0/publish/pubsub/orders \
  -H "Content-Type: application/json" \
  -d '{
    "orderId": "123",
    "status": "shipped",
    "timestamp": "2026-01-03T10:00:00Z"
  }'
```

#### Subscribe to Topics

**Option 1: Programmatic**
```json
// GET http://localhost:8080/dapr/subscribe
[
  {
    "pubsubname": "pubsub",
    "topic": "orders",
    "route": "/orders"
  }
]

// Dapr calls POST http://localhost:8080/orders with message
```

**Option 2: Declarative**
```yaml
apiVersion: dapr.io/v2alpha1
kind: Subscription
metadata:
  name: order-subscription
spec:
  pubsubname: pubsub
  topic: orders
  route: /orders
scopes:
- order-processor
```

### State Management

#### Save State
```bash
# Save single item
curl -X POST http://localhost:3500/v1.0/state/statestore \
  -H "Content-Type: application/json" \
  -d '[
    {
      "key": "user-123",
      "value": {
        "name": "John Doe",
        "email": "john@example.com"
      }
    }
  ]'

# Save multiple items (transaction)
curl -X POST http://localhost:3500/v1.0/state/statestore \
  -H "Content-Type: application/json" \
  -d '[
    {"key": "key1", "value": "value1"},
    {"key": "key2", "value": "value2"}
  ]'
```

#### Get State
```bash
# Get single item
curl http://localhost:3500/v1.0/state/statestore/user-123

# Get multiple items (bulk)
curl -X POST http://localhost:3500/v1.0/state/statestore/bulk \
  -H "Content-Type: application/json" \
  -d '{"keys": ["key1", "key2"]}'
```

#### Delete State
```bash
# Delete item
curl -X DELETE http://localhost:3500/v1.0/state/statestore/user-123
```

### Bindings

#### Input Binding (Trigger)
```yaml
# Dapr polls queue and calls your app
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: queue-input
spec:
  type: bindings.azure.storagequeues
  # ...
```

**Your app endpoint**:
```bash
# Dapr POSTs messages to this endpoint
POST http://localhost:8080/queue-message
{
  "data": "message content",
  "metadata": { ... }
}
```

#### Output Binding
```bash
# Write to blob storage
curl -X POST http://localhost:3500/v1.0/bindings/blob-storage \
  -H "Content-Type: application/json" \
  -d '{
    "data": "file content",
    "metadata": {
      "blobName": "output.txt"
    },
    "operation": "create"
  }'
```

### Secrets

#### Get Secret
```bash
# Retrieve from Azure Key Vault
curl http://localhost:3500/v1.0/secrets/azurekeyvault/db-password

# Response:
{
  "db-password": "actual-password-value"
}
```

## Dapr Component Management

### Add Component to Environment

```bash
# Create component (state store example)
az containerapp env dapr-component set \
  --name myenvironment \
  --resource-group myResourceGroup \
  --dapr-component-name statestore \
  --yaml component.yaml
```

**component.yaml**:
```yaml
componentType: state.azure.cosmosdb
version: v1
metadata:
- name: url
  value: "https://myaccount.documents.azure.com:443/"
- name: masterKey
  secretRef: cosmos-key
- name: database
  value: "statedb"
- name: collection
  value: "state"
scopes:
- order-service
secrets:
- name: cosmos-key
  value: "<cosmos-db-key>"
```

### List Components

```bash
# List Dapr components in environment
az containerapp env dapr-component list \
  --name myenvironment \
  --resource-group myResourceGroup \
  --output table
```

### Remove Component

```bash
# Remove Dapr component
az containerapp env dapr-component remove \
  --name myenvironment \
  --resource-group myResourceGroup \
  --dapr-component-name statestore
```

## Example: Order Processing System

### Architecture
```
Order API (Dapr enabled)
  ↓ Service Invocation
Inventory Service (Dapr enabled)
  ↓ Pub/Sub (Service Bus)
Order Processor (Dapr enabled)
  ↓ State Management (Cosmos DB)
Order Database
```

### Order API (Frontend)

**Enable Dapr**:
```bash
az containerapp create \
  --name order-api \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image order-api:latest \
  --enable-dapr \
  --dapr-app-id order-api \
  --dapr-app-port 8080 \
  --dapr-app-protocol http
```

**Call Inventory Service**:
```python
import requests

# Check stock via Dapr service invocation
response = requests.get(
    f"http://localhost:3500/v1.0/invoke/inventory-service/method/check-stock/{product_id}"
)
stock = response.json()
```

**Publish Order Event**:
```python
# Publish order created event
requests.post(
    "http://localhost:3500/v1.0/publish/pubsub/orders",
    json={"orderId": order_id, "status": "created"}
)
```

### Inventory Service (Backend)

**Enable Dapr**:
```bash
az containerapp create \
  --name inventory-service \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image inventory-service:latest \
  --enable-dapr \
  --dapr-app-id inventory-service \
  --dapr-app-port 8080
```

**Expose Method**:
```python
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/check-stock/<product_id>')
def check_stock(product_id):
    # Check inventory
    return jsonify({"available": 10})
```

### Order Processor (Worker)

**Enable Dapr**:
```bash
az containerapp create \
  --name order-processor \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image order-processor:latest \
  --enable-dapr \
  --dapr-app-id order-processor \
  --dapr-app-port 8080
```

**Subscribe to Orders**:
```python
# Dapr subscription endpoint
@app.route('/dapr/subscribe', methods=['GET'])
def subscribe():
    return jsonify([{
        'pubsubname': 'pubsub',
        'topic': 'orders',
        'route': '/orders'
    }])

# Message handler
@app.route('/orders', methods=['POST'])
def process_order():
    order = request.json
    # Save to state store
    requests.post(
        "http://localhost:3500/v1.0/state/statestore",
        json=[{"key": f"order-{order['orderId']}", "value": order}]
    )
    return jsonify({"status": "processed"})
```

## Observability

### Distributed Tracing (Распределённая трассировка)

**Работает автоматически при использовании Dapr**:

- Dapr добавляет trace-заголовки (W3C Trace Context)
- Отправляет трассировки в Application Insights
- Коррелирует запросы между несколькими сервисами

---

## Как это работает

- Каждый входящий запрос получает уникальный trace identifier.
- При межсервисных вызовах Dapr автоматически передаёт trace-контекст дальше.
- Это позволяет видеть полный путь запроса через все микросервисы.
- Логи, метрики и трассировки объединяются в единую цепочку.

---

## Почему это важно

В распределённых системах сложно определить:
- где возникла ошибка
- какой сервис вызвал другой
- на каком этапе увеличилась задержка

Distributed tracing решает эту проблему, обеспечивая сквозную видимость (end-to-end visibility).

---

## Важно для AZ-204

- Трассировка работает автоматически при включённом Dapr.
- Используется стандарт W3C Trace Context.
- Интеграция с Application Insights позволяет анализировать цепочку вызовов.
- Dapr уменьшает необходимость ручной реализации корреляции запросов.


### Configure Application Insights

```bash
# Set instrumentation key in environment
az containerapp env update \
  --name myenvironment \
  --resource-group myResourceGroup \
  --dapr-instrumentation-key "<app-insights-key>"
```

### View Traces

**Application Insights → Transaction Search**:
```
Request: POST /orders
  ├── Service Invocation: inventory-service
  │   └── GET /check-stock/123
  ├── Pub/Sub Publish: orders topic
  └── State Save: order-123
```

## Best Practices

### 1. Use Scopes for Security
```yaml
# Limit component access
scopes:
- order-service  # Only order-service can use
```

### 2. Leverage Managed Identity
```yaml
# Use managed identity for Azure services
- name: azureClientId
  value: "<managed-identity-client-id>"
```

### 3. Enable Distributed Tracing
```bash
# Configure Application Insights
--dapr-instrumentation-key "<key>"
```

### 4. Используйте подходящие Building Blocks

Правильный выбор building block упрощает архитектуру и снижает связанность сервисов.

- **Взаимодействие сервис–сервис** → Service Invocation  
  Используется для синхронных вызовов между микросервисами.

- **Асинхронные события** → Pub/Sub  
  Подходит для событийно-ориентированной архитектуры и слабой связанности компонентов.

- **Сессионное состояние** → State Management  
  Применяется для хранения пользовательского состояния и временных данных.

- **Опрос очередей** → Input Bindings  
  Позволяет реагировать на внешние источники событий без прямого подключения к ним в коде.

- **Загрузка файлов** → Output Bindings  
  Используется для отправки данных во внешние системы или хранилища.

---

## Архитектурный принцип

- Синхронное взаимодействие → Service Invocation
- Асинхронное взаимодействие → Pub/Sub
- Хранение состояния → State
- Интеграция с внешними системами → Bindings

> 🎯 Для AZ-204 важно уметь сопоставить сценарий с правильным building block.
- Если требуется слабая связанность — выбирайте Pub/Sub.
- Если требуется прямой вызов — Service Invocation.
- Если требуется хранение состояния — State Management.


### 5. Test Locally with Dapr CLI
```bash
# Run app with Dapr locally
dapr run --app-id myapp --app-port 8080 --dapr-http-port 3500 -- python app.py
```

## Критически важные моменты (Critical Notes)

- 💡 **Dapr** — Distributed Application Runtime для построения микросервисных систем
- ✅ **Sidecar** — автоматически добавляется при включении Dapr
- 🎯 **Building blocks** — service invocation, pub/sub, state management, bindings, secrets, actors
- 🔄 **Components** — подключаемые реализации (Azure, AWS и др.)
- 📊 **HTTP API** — используется локальный endpoint `localhost:3500` для вызовов Dapr
- 🔒 **Scopes** — позволяют ограничить доступ компонента определённым приложениям
- ⚠️ **App ID** — должен быть уникальным в пределах Container Apps Environment
- 💡 **App port** — обязателен, если приложение принимает вызовы от Dapr
- ✅ **Observability** — автоматическая распределённая трассировка
- 🎯 **Zero infrastructure** — полностью управляется платформой Azure Container Apps

---

# Exam Tips (AZ-204)

## Основы

- Dapr — runtime для микросервисной архитектуры.
- Архитектура sidecar — Dapr работает рядом с вашим контейнером.
- Включение Dapr требует указания флага активации и уникального app ID.

---

## Building Blocks

Основные возможности:

- **Service invocation** — вызов сервисов по app ID
- **Pub/Sub** — публикация и подписка на события
- **State management** — операции CRUD для хранения состояния
- **Bindings** — интеграция с внешними системами
- **Secrets** — безопасное получение секретов
- **Actors** — stateful виртуальные акторы

---

## Service Invocation

- Сервисы вызываются по **app ID**, а не по URL.
- Вызовы проходят через локальный endpoint Dapr.
- Поддерживаются HTTP и gRPC.

---

## Pub/Sub

- Публикация выполняется через API публикации событий.
- Подписка реализуется через специальный endpoint приложения.
- Используется для слабосвязанной событийной архитектуры.

---

## State Management

- Поддерживает операции создания, чтения, обновления и удаления.
- Работает через настроенный state store компонент.
- Позволяет сохранять состояние независимо от инфраструктуры.

---

## Bindings

- **Input bindings** — получение событий из внешних источников.
- **Output bindings** — отправка данных во внешние системы.

---

## Components

- Описываются через YAML-конфигурацию.
- Содержат тип, версию, метаданные и scopes.
- Примеры типов:
    - state.azure.cosmosdb
    - pubsub.azure.servicebus
    - secretstores.azure.keyvault

---

## API Endpoints Sidecar

- HTTP API — `localhost:3500`
- gRPC API — `localhost:50001`

---

## Важные ограничения

- App ID должен быть уникальным в среде.
- App port обязателен, если приложение принимает входящие вызовы.
- Можно ограничивать доступ компонентов через scopes.

---

## Observability

- Распределённая трассировка работает автоматически.
- Интеграция с Application Insights.
- Поддержка W3C Trace Context.

---

## Дополнительные моменты

- Не требуется изменение бизнес-логики — используется HTTP/gRPC API.
- Azure предоставляет управляемые компоненты для популярных сервисов.
- Для локального тестирования используется Dapr CLI.

---

## Частые экзаменационные вопросы

- Как включается Dapr?
- Что такое sidecar-архитектура?
- В чём разница между building block и component?
- Как ограничить доступ к компоненту?
- Когда требуется указание app port?
- Как реализуется service-to-service communication?
- Как обеспечивается distributed tracing?

---


[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/7-explore-distributed-application-runtime)
