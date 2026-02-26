# Управление доступом к событиям Azure Event Grid

## Обзор

Azure Event Grid предоставляет несколько механизмов защиты:

- **Azure RBAC** — управление доступом и авторизация
- **Managed Identities** — безопасная аутентификация
- **Webhook validation** — подтверждение владения endpoint
- **Private Endpoints** — сетовая изоляция

Эти механизмы позволяют реализовать принцип **least privilege** и защитить event-driven архитектуру.

---

# Встроенные роли RBAC

Azure Event Grid включает **четыре встроенные роли** для гибкого контроля доступа.

---

## Таблица ролей Event Grid

| Название роли | Разрешения | Область действия | Сценарий использования |
|---------------|------------|------------------|------------------------|
| **Event Grid Subscription Reader** | Чтение подписок | Subscription | Просмотр конфигурации |
| **Event Grid Subscription Contributor** | Управление подписками (создание, обновление, удаление) | Subscription | Операционная команда |
| **Event Grid Contributor** | Полный контроль над ресурсами (topics, subscriptions, domains) | Topic / Domain | Администраторы |
| **Event Grid Data Sender** | Публикация событий в topic | Topic | Приложения, публикующие события |

---

## Архитектурное значение

RBAC позволяет разделить ответственность:

- Разработчики публикуют события → **Data Sender**
- DevOps управляют подписками → **Subscription Contributor**
- Администраторы управляют инфраструктурой → **Contributor**
- Аудиторы → **Subscription Reader**

Это повышает безопасность и упрощает контроль.

---

## Best Practices

- Назначайте минимально необходимые права
- Используйте Managed Identity вместо access keys
- Разделяйте роли между командами
- Не давайте Contributor без необходимости

---

## Важно для AZ-204

Нужно помнить:

- Data Sender публикует события
- Subscription Contributor управляет подписками
- Contributor управляет инфраструктурой
- Reader — только просмотр

Экзамен часто проверяет выбор правильной RBAC-роли для сценария.
### Role Permissions Breakdown

#### Event Grid Subscription Reader
**Actions Allowed:**
```
Microsoft.EventGrid/eventSubscriptions/read
Microsoft.EventGrid/topicTypes/eventSubscriptions/read
```

**Use Cases:**
- Auditing subscription configurations
- Viewing event filters and destinations
- Monitoring team

**Assignment Example:**
```bash
az role assignment create \
  --role "EventGrid Subscription Reader" \
  --assignee user@example.com \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic"
```

#### Event Grid Subscription Contributor
**Actions Allowed:**
```
Microsoft.EventGrid/eventSubscriptions/*
Microsoft.EventGrid/topicTypes/eventSubscriptions/*
```

**Use Cases:**
- Creating and configuring event subscriptions
- Updating filters and retry policies
- Operations team

**Assignment Example:**
```bash
az role assignment create \
  --role "EventGrid Subscription Contributor" \
  --assignee <managed-identity-principal-id> \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage"
```

#### Event Grid Contributor
**Actions Allowed:**
```
Microsoft.EventGrid/*
```

**Use Cases:**
- Creating and managing topics
- Creating and managing domains
- Full Event Grid administration

**Assignment Example:**
```bash
az role assignment create \
  --role "EventGrid Contributor" \
  --assignee user@example.com \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG"
```

#### Event Grid Data Sender
**Actions Allowed:**
```
Microsoft.EventGrid/topics/send/action
Microsoft.EventGrid/domains/send/action
```

**Use Cases:**
- Applications publishing custom events
- Service-to-service event publishing
- Managed identity authentication

**Assignment Example:**
```bash
# Assign to managed identity
az role assignment create \
  --role "EventGrid Data Sender" \
  --assignee <managed-identity-object-id> \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic"
```

---

## Permissions for Event Subscriptions

Different permission requirements apply based on the topic type.

### System Topics Permissions

**System topics** are tied to Azure resources (Storage, IoT Hub, etc.).

**Required Permission:**
```
Microsoft.EventGrid/EventSubscriptions/Write
```

**Resource Scope:**
- Permission must be granted at the **source resource scope**
- Example: To subscribe to Storage account events, need Write permission on the storage account

**Example:** Subscribe to Azure Storage Events

```bash
# Get storage account resource ID
STORAGE_ID=$(az storage account show \
  --name mystorage \
  --resource-group myRG \
  --query "id" --output tsv)

# Create event subscription (requires Write permission on storage account)
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id $STORAGE_ID \
  --endpoint https://myfunction.azurewebsites.net/api/handler \
  --included-event-types Microsoft.Storage.BlobCreated

# Grant permission to user/identity
az role assignment create \
  --role "EventGrid Subscription Contributor" \
  --assignee user@example.com \
  --scope $STORAGE_ID
```

**Full Resource Path Format:**
```
/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/{resource-provider}/{resource-type}/{resource-name}

Example:
/subscriptions/abc123-def4-5678-90ab-cdef12345678/resourceGroups/myRG/providers/Microsoft.Storage/storageAccounts/mystorage
```

### Custom Topics Permissions

**Custom topics** are standalone Event Grid resources.

**Required Permission:**
```
Microsoft.EventGrid/EventSubscriptions/Write
```

**Resource Scope:**
- Permission must be granted at the **Event Grid topic scope**

**Example:** Subscribe to Custom Topic

```bash
# Get custom topic resource ID
TOPIC_ID=$(az eventgrid topic show \
  --name myTopic \
  --resource-group myRG \
  --query "id" --output tsv)

# Create event subscription (requires Write permission on topic)
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id $TOPIC_ID \
  --endpoint https://myfunction.azurewebsites.net/api/handler

# Grant permission
az role assignment create \
  --role "EventGrid Subscription Contributor" \
  --assignee user@example.com \
  --scope $TOPIC_ID
```

**Full Resource Path Format:**
```
/subscriptions/{subscription-id}/resourceGroups/{resource-group}/providers/Microsoft.EventGrid/topics/{topic-name}

Example:
/subscriptions/abc123-def4-5678-90ab-cdef12345678/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myCustomTopic
```

### Event Domains Permissions

**Event domains** support multi-tenant scenarios with multiple topics.

**Required Permissions:**
- Domain-level subscription: `Microsoft.EventGrid/EventSubscriptions/Write` on domain
- Domain topic subscription: Write permission on specific domain topic

**Example:** Subscribe to Event Domain

```bash
# Create subscription at domain level
az eventgrid event-subscription create \
  --name myDomainSubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/domains/myDomain" \
  --endpoint https://myfunction.azurewebsites.net/api/handler

# Create subscription for specific domain topic
az eventgrid event-subscription create \
  --name myTopicSubscription \
  --source-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/domains/myDomain/topics/topic1" \
  --endpoint https://myfunction.azurewebsites.net/api/handler
```

---

## Publishing Events Authentication

### Access Keys (Basic Authentication)

Default authentication method for custom topics.

**Get Topic Access Keys:**
```bash
# Get topic endpoint
ENDPOINT=$(az eventgrid topic show \
  --name myTopic \
  --resource-group myRG \
  --query "endpoint" --output tsv)

# Get access key
KEY=$(az eventgrid topic key list \
  --name myTopic \
  --resource-group myRG \
  --query "key1" --output tsv)

# Publish event with access key
curl -X POST $ENDPOINT \
  -H "aeg-sas-key: $KEY" \
  -H "Content-Type: application/cloudevents+json" \
  -d '[{
    "specversion": "1.0",
    "type": "com.example.someevent",
    "source": "/mycontext",
    "id": "event-001",
    "data": { "key": "value" }
  }]'
```

**C# SDK with Access Key:**
```csharp
using Azure;
using Azure.Messaging.EventGrid;

var endpoint = new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events");
var credential = new AzureKeyCredential(topicKey);
var client = new EventGridPublisherClient(endpoint, credential);

var cloudEvent = new CloudEvent(
    source: "/myapp",
    type: "com.example.event",
    jsonSerializableData: new { message = "Hello Event Grid" }
);

await client.SendEventAsync(cloudEvent);
```

**Rotate Access Keys:**
```bash
# Regenerate key1
az eventgrid topic key regenerate \
  --name myTopic \
  --resource-group myRG \
  --key-name key1

# Regenerate key2
az eventgrid topic key regenerate \
  --name myTopic \
  --resource-group myRG \
  --key-name key2
```

### Managed Identity (Recommended)

**System-Assigned Managed Identity:**

```bash
# Enable system-assigned identity on Azure Function
az functionapp identity assign \
  --name myFunctionApp \
  --resource-group myRG

# Get identity principal ID
PRINCIPAL_ID=$(az functionapp identity show \
  --name myFunctionApp \
  --resource-group myRG \
  --query "principalId" --output tsv)

# Grant Event Grid Data Sender role
az role assignment create \
  --role "EventGrid Data Sender" \
  --assignee $PRINCIPAL_ID \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic"
```

**C# SDK with Managed Identity:**
```csharp
using Azure.Identity;
using Azure.Messaging.EventGrid;

// Use DefaultAzureCredential (works with Managed Identity, Azure CLI, Visual Studio)
var endpoint = new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events");
var credential = new DefaultAzureCredential();
var client = new EventGridPublisherClient(endpoint, credential);

var cloudEvent = new CloudEvent(
    source: "/myapp",
    type: "com.example.event",
    jsonSerializableData: new { message = "Hello from Managed Identity" }
);

await client.SendEventAsync(cloudEvent);
```

**User-Assigned Managed Identity:**

```bash
# Create user-assigned identity
az identity create \
  --name myEventGridIdentity \
  --resource-group myRG

# Get identity details
IDENTITY_ID=$(az identity show \
  --name myEventGridIdentity \
  --resource-group myRG \
  --query "id" --output tsv)

PRINCIPAL_ID=$(az identity show \
  --name myEventGridIdentity \
  --resource-group myRG \
  --query "principalId" --output tsv)

# Assign identity to Azure Function
az functionapp identity assign \
  --name myFunctionApp \
  --resource-group myRG \
  --identities $IDENTITY_ID

# Grant permissions
az role assignment create \
  --role "EventGrid Data Sender" \
  --assignee $PRINCIPAL_ID \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic"
```

### Azure AD Service Principal

```bash
# Create service principal
az ad sp create-for-rbac \
  --name "EventGridPublisher" \
  --role "EventGrid Data Sender" \
  --scopes "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic"

# Output:
# {
#   "appId": "app-id",
#   "password": "password",
#   "tenant": "tenant-id"
# }
```

**C# SDK with Service Principal:**
```csharp
using Azure.Identity;
using Azure.Messaging.EventGrid;

var credential = new ClientSecretCredential(
    tenantId: "tenant-id",
    clientId: "app-id",
    clientSecret: "password"
);

var endpoint = new Uri("https://mytopic.eastus-1.eventgrid.azure.net/api/events");
var client = new EventGridPublisherClient(endpoint, credential);

await client.SendEventAsync(cloudEvent);
```

---

## Event Handler Authentication

### Webhook Authentication

Event Grid can add authentication information when delivering to webhooks.

**Azure AD OAuth Token:**
```json
{
  "type": "Microsoft.EventGrid/eventSubscriptions",
  "properties": {
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
}
```

**Custom Headers (API Keys):**
```bash
az eventgrid event-subscription create \
  --name mySubscription \
  --source-resource-id $TOPIC_ID \
  --endpoint https://myapi.example.com/webhooks/events \
  --delivery-attribute-mapping \
    X-API-Key static "your-api-key-here"
```

### Azure Service Authentication

**Azure Function with Managed Identity:**
```bash
# Create subscription to Azure Function
az eventgrid event-subscription create \
  --name functionSubscription \
  --source-resource-id $TOPIC_ID \
  --endpoint-type azurefunction \
  --endpoint "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.Web/sites/myFunctionApp/functions/EventHandler" \
  --delivery-with-resource-identity systemassigned
```

**Event Hubs with Managed Identity:**
```bash
# Create subscription to Event Hubs
az eventgrid event-subscription create \
  --name eventhubSubscription \
  --source-resource-id $TOPIC_ID \
  --endpoint-type eventhub \
  --endpoint "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventHub/namespaces/mynamespace/eventhubs/myhub" \
  --delivery-with-resource-identity systemassigned
```

---

## Network Security

### Private Endpoints

**Enable private endpoint access:**

```bash
# Create private endpoint
az network private-endpoint create \
  --name myPrivateEndpoint \
  --resource-group myRG \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic" \
  --group-id topic \
  --connection-name myConnection

# Disable public network access
az eventgrid topic update \
  --name myTopic \
  --resource-group myRG \
  --public-network-access disabled
```

### IP Filtering

**Configure IP firewall rules:**

```bash
# Allow specific IP addresses
az eventgrid topic update \
  --name myTopic \
  --resource-group myRG \
  --inbound-ip-rules \
    "[
      {
        'ipMask': '203.0.113.0/24',
        'action': 'Allow'
      },
      {
        'ipMask': '198.51.100.42',
        'action': 'Allow'
      }
    ]"
```

**ARM Template:**
```json
{
  "type": "Microsoft.EventGrid/topics",
  "properties": {
    "publicNetworkAccess": "Enabled",
    "inboundIpRules": [
      {
        "ipMask": "203.0.113.0/24",
        "action": "Allow"
      },
      {
        "ipMask": "198.51.100.42",
        "action": "Allow"
      }
    ]
  }
}
```

---

## Security Best Practices

### Publishing Events

1. **Use Managed Identity** instead of access keys
   ```csharp
   // ✅ Good: Managed Identity
   var credential = new DefaultAzureCredential();
   var client = new EventGridPublisherClient(endpoint, credential);
   
   // ❌ Avoid: Hardcoded keys
   var credential = new AzureKeyCredential("hardcoded-key");
   ```
## Публикация событий — рекомендации по безопасности

2️⃣ **Регулярная ротация access keys** (если используются ключи)

- Внедрите политику ротации (например, каждые 90 дней)
- Храните ключи в Azure Key Vault
- Используйте secondary key для безостановочной ротации

3️⃣ **Принцип наименьших привилегий (Least Privilege)**

- Назначайте только необходимые разрешения
- Для публикации используйте **Event Grid Data Sender**, а не Contributor
- Разделяйте роли для разработки и администрирования

---

## Получение событий

1️⃣ **Корректная валидация Webhook**

- Обрабатывайте validation handshake
- Подтверждайте владение endpoint

2️⃣ **Используйте только HTTPS**

- Действительный сертификат
- Современные версии TLS

3️⃣ **Реализуйте аутентификацию**

- OAuth 2.0
- API keys
- Managed Identity
- Custom headers с токенами

4️⃣ **Проверка подписи события** (если используется)

- Проверяйте источник события
- Не доверяйте payload без валидации

---

## Сетевая безопасность

1️⃣ **Private Endpoints**

- Используйте для чувствительных нагрузок
- Изолируйте трафик внутри VNet

2️⃣ **IP-фильтрация**

- Ограничивайте доступ publisher’ов
- Разрешайте только доверенные диапазоны IP

3️⃣ **Отключение публичного доступа**

- Если внешний доступ не требуется
- Используйте Private Link вместо public endpoint

---

## Архитектурный акцент

Безопасность Event Grid должна охватывать:

- Аутентификацию publisher’ов
- Авторизацию через RBAC
- Безопасность доставки (HTTPS, валидация)
- Сетевую изоляцию

---

## Важно для AZ-204

На экзамене важно помнить:

- Предпочитайте Managed Identity вместо access keys
- Используйте Event Grid Data Sender для публикации
- Webhook требует HTTPS и валидации
- Private Endpoints обеспечивают изоляцию
- Принцип least privilege — обязательный подход

Вопросы по безопасности часто проверяют правильный выбор механизма защиты.
---

## RBAC Assignment Examples

### Scenario 1: Application Publishing Events

```bash
# 1. Create managed identity for the application
az identity create --name myAppIdentity --resource-group myRG

# 2. Get principal ID
PRINCIPAL_ID=$(az identity show \
  --name myAppIdentity \
  --resource-group myRG \
  --query "principalId" --output tsv)

# 3. Grant Data Sender role to topic
az role assignment create \
  --role "EventGrid Data Sender" \
  --assignee $PRINCIPAL_ID \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.EventGrid/topics/myTopic"

# 4. Assign identity to Azure resource (e.g., App Service)
az webapp identity assign \
  --name myWebApp \
  --resource-group myRG \
  --identities "/subscriptions/{sub-id}/resourceGroups/myRG/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myAppIdentity"
```

### Scenario 2: DevOps Team Managing Subscriptions

```bash
# Grant Subscription Contributor to DevOps group
az role assignment create \
  --role "EventGrid Subscription Contributor" \
  --assignee-object-id <devops-group-object-id> \
  --assignee-principal-type Group \
  --scope "/subscriptions/{sub-id}/resourceGroups/myRG"
```

### Scenario 3: Monitoring Team (Read-Only)

```bash
# Grant Subscription Reader to monitoring team
az role assignment create \
  --role "EventGrid Subscription Reader" \
  --assignee monitoring-team@example.com \
  --scope "/subscriptions/{sub-id}"
```

---

# Советы к экзамену AZ-204

## Ключевые концепции

1️⃣ **Четыре роли RBAC**

- Event Grid Subscription Reader
- Event Grid Subscription Contributor
- Event Grid Contributor
- Event Grid Data Sender

2️⃣ **Event Grid Data Sender**

- Используется для публикации событий
- Рекомендуемая роль для приложений

3️⃣ **Права на подписки**

- Различаются для system topics и custom topics

4️⃣ **System Topics**

- Требуется разрешение Write на исходный ресурс

5️⃣ **Custom Topics**

- Требуется разрешение Write на Event Grid topic

6️⃣ **Managed Identity**

- Предпочтительнее access keys

7️⃣ **Private Endpoints**

- Используются для сетевой изоляции

---

# Частые экзаменационные сценарии

### Сценарий 1
Приложение публикует события в custom topic

- ✅ Назначить роль **Event Grid Data Sender**
- ❌ Не назначать **Event Grid Contributor** (слишком широкие права)

---

### Сценарий 2
Пользователь создаёт подписку на события Storage Account

- ✅ Назначить **Event Grid Subscription Contributor** на Storage Account
- ❌ Не назначать права на Event Grid topic (неверный ресурс)

---

### Сценарий 3
Безопасная публикация без использования ключей

- ✅ Использовать Managed Identity + Data Sender
- ❌ Не хранить access keys в коде

---

### Сценарий 4
Ограничить источники событий по IP

- ✅ Настроить IP filtering на Event Grid topic
- ✅ Использовать Private Endpoints для доступа из VNet

---

# Что обязательно помнить

- **Event Grid Data Sender** — только публикация
- **Event Grid Contributor** — полный контроль
- **Subscription Contributor** — управление подписками
- **System topic subscriptions** — права на исходный ресурс
- **Custom topic subscriptions** — права на Event Grid topic
- **Managed Identity** — предпочтительный способ аутентификации
- **Access keys** — два ключа для ротации (key1, key2)
- **Private Endpoints** — изоляция на уровне сети
- **IP filtering** — ограничение IP publisher’ов

---

## Экзаменационный акцент

Если вопрос про:

- публикацию событий → Data Sender
- создание подписки → Subscription Contributor
- безопасность без ключей → Managed Identity
- изоляцию сети → Private Endpoints
- ограничение источников → IP filtering

RBAC + Managed Identity + Private Endpoint — стандартный безопасный паттерн.
### Quick Command Reference

```bash
# Grant Data Sender role
az role assignment create \
  --role "EventGrid Data Sender" \
  --assignee <identity> \
  --scope <topic-resource-id>

# Get topic keys
az eventgrid topic key list \
  --name <topic> \
  --resource-group <rg>

# Regenerate key
az eventgrid topic key regenerate \
  --name <topic> \
  --resource-group <rg> \
  --key-name key1

# Enable private endpoint
az network private-endpoint create \
  --name <endpoint-name> \
  --resource-group <rg> \
  --vnet-name <vnet> \
  --subnet <subnet> \
  --private-connection-resource-id <topic-id> \
  --group-id topic
```

---

# Итоги по управлению доступом в Azure Event Grid

## Методы контроля доступа

- **RBAC-роли**  
  Четыре встроенные роли для точного разграничения прав

- **Managed Identity**  
  Рекомендуемый способ аутентификации для публикации событий

- **Access Keys**  
  Два ключа (key1, key2) для ротации

- **Private Endpoints**  
  Изоляция на уровне сети

- **IP Filtering**  
  Ограничение IP-адресов publisher’ов

---

## Ключевые роли RBAC

- **Event Grid Data Sender**  
  Публикация событий в topic

- **Event Grid Subscription Contributor**  
  Управление подписками (создание, обновление, удаление)

- **Event Grid Contributor**  
  Полный контроль над ресурсами Event Grid

- **Event Grid Subscription Reader**  
  Просмотр конфигурации подписок

---

## Права для подписок

- **System Topics**  
  Требуется разрешение Write на исходный Azure-ресурс

- **Custom Topics**  
  Требуется разрешение Write на Event Grid topic

- **Event Domains**  
  Права могут назначаться на уровне домена или domain-topic

---

## Best Practices

- ✅ Использовать Managed Identity вместо access keys
- ✅ Применять принцип least privilege
- ✅ Использовать Private Endpoints для чувствительных систем
- ✅ Настраивать IP filtering
- ✅ Регулярно ротировать access keys (если используются)
- ✅ Использовать custom headers для защиты webhook
- ✅ Назначать только необходимые RBAC-роли

---

## Архитектурный акцент

Безопасная конфигурация Event Grid включает:

- Минимальные права (RBAC)
- Безключевую аутентификацию (Managed Identity)
- Сетевую изоляцию (Private Endpoint)
- Ограничение источников (IP filtering)

---

## Что важно для AZ-204

Нужно чётко понимать:

- Какая роль используется для публикации (Data Sender)
- Где назначаются права для system vs custom topics
- Почему Managed Identity предпочтительнее ключей
- Как Private Endpoints усиливают безопасность

Вопросы по безопасности часто проверяют правильный выбор роли и механизма аутентификации.