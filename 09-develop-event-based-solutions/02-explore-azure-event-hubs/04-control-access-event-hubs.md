# Контроль доступа к Event Hubs

## Обзор аутентификации и авторизации

Azure Event Hubs поддерживает несколько механизмов аутентификации для защиты доступа к namespace и конкретным Event Hub, а также для управления правами на отправку и получение событий.

---

## Методы аутентификации

| Метод | Описание | Сценарий использования | Рекомендуется |
|--------|-----------|------------------------|----------------|
| **Azure Active Directory (Azure AD)** | Аутентификация по OAuth 2.0 токену | Production-приложения | ✅ Да |
| **Managed Identity** | Без хранения секретов в коде | Приложения, размещённые в Azure | ✅ Да |
| **Shared Access Signature (SAS)** | Токен на основе ключа | Legacy или не-Azure приложения | ⚠️ С осторожностью |
| **Connection String** | Содержит shared key | Разработка и тестирование | ❌ Не рекомендуется |

---

# Авторизация через Azure Active Directory (Azure AD)

**Azure AD** обеспечивает аутентификацию на основе идентичности с использованием:

- OAuth 2.0
- Microsoft identity platform

Это рекомендуемый способ для production-сценариев.

---

## Преимущества Azure AD

- ✅ **Отсутствие секретов в коде**  
  Токены выдаются и управляются Azure AD

- ✅ **Гибкая модель доступа**  
  Используется Role-Based Access Control (RBAC)

- ✅ **Централизованное управление**  
  Настройка прав через Azure Portal

- ✅ **Аудит и логирование**  
  Возможность отслеживать, кто и когда получил доступ

- ✅ **Автоматическое истечение токенов**  
  Снижается риск компрометации

- ✅ **Поддержка MFA**  
  Дополнительный уровень безопасности

---

## Что важно для AZ-204

- Для production рекомендуется Azure AD или Managed Identity
- SAS используется при интеграции с внешними системами
- Connection string с ключом — не best practice для продакшена
- RBAC управляет доступом на уровне namespace или Event Hub

---

## Ключевая идея

Современный подход к безопасности Event Hubs —  
**identity-based access (Azure AD / Managed Identity)** вместо хранения ключей.

На экзамене почти всегда правильный ответ — отказаться от shared keys и использовать RBAC через Azure AD.

### How Azure AD Authorization Works

```
┌──────────────────────────────────────────────────────────────┐
│                      APPLICATION                              │
│  1. Request Azure AD token                                   │
│     • Client ID + Client Secret (Service Principal)          │
│     • OR Managed Identity (Azure-hosted apps)                │
└────────────────────┬─────────────────────────────────────────┘
                     │
                     ▼
        ┌────────────────────────────┐
        │      AZURE AD (OAuth 2.0)  │
        │  2. Validate identity      │
        │  3. Issue access token     │
        │     • Scope: Event Hubs    │
        │     • Expiry: 1 hour       │
        └────────────┬───────────────┘
                     │
                     ▼ (Access Token)
        ┌────────────────────────────┐
        │      EVENT HUBS NAMESPACE  │
        │  4. Validate token         │
        │  5. Check RBAC permissions │
        │  6. Allow/Deny operation   │
        └────────────────────────────┘
```

## Встроенные роли RBAC (Built-in RBAC Roles)

Azure Event Hubs предоставляет три встроенные роли для управления доступом к данным.

| Роль | Описание | Разрешения | Область применения (Scope) |
|------|-----------|------------|----------------------------|
| **Azure Event Hubs Data Owner** | Полный доступ к данным Event Hubs | Отправка, получение, управление | Namespace или конкретный Event Hub |
| **Azure Event Hubs Data Sender** | Доступ только на отправку | Отправка событий | Namespace или конкретный Event Hub |
| **Azure Event Hubs Data Receiver** | Доступ только на получение | Получение событий | Namespace или конкретный Event Hub |

---

## Пояснения

### 🔹 Azure Event Hubs Data Owner
- Может отправлять и получать события
- Может управлять consumer groups
- Подходит для административных или сервисных ролей

---

### 🔹 Azure Event Hubs Data Sender
- Может только публиковать события
- Не имеет доступа к чтению
- Идеально для producer-приложений

---

### 🔹 Azure Event Hubs Data Receiver
- Может только читать события
- Используется consumer-приложениями
- Не имеет прав на отправку

---

## Scope (область назначения роли)

Роль можно назначить:

- На уровне **Namespace** — доступ ко всем Event Hub внутри
- На уровне конкретного **Event Hub** — более точечный контроль

📌 Рекомендуется назначать роль на минимально необходимом уровне (principle of least privilege).

---

## Что важно для AZ-204

- RBAC используется вместе с Azure AD
- Роли Data Sender и Data Receiver чаще всего используются в production
- Роль назначается через Azure Portal, CLI или ARM/Bicep
- Лучше использовать Managed Identity + RBAC вместо SAS

---

## Экзаменационная логика

Если в вопросе говорится:

- «Приложение должно только отправлять события» → Data Sender
- «Приложение должно только читать события» → Data Receiver
- «Полный доступ» → Data Owner

Понимание различий между этими ролями — обязательный элемент темы безопасности Event Hubs.
**Detailed Permissions:**

```
Azure Event Hubs Data Owner:
├── Microsoft.EventHub/namespaces/eventhubs/send/action
├── Microsoft.EventHub/namespaces/eventhubs/receive/action
└── Microsoft.EventHub/namespaces/eventhubs/manage/action

Azure Event Hubs Data Sender:
└── Microsoft.EventHub/namespaces/eventhubs/send/action

Azure Event Hubs Data Receiver:
└── Microsoft.EventHub/namespaces/eventhubs/receive/action
```

### Assign RBAC Roles

**Azure Portal:**
1. Navigate to Event Hubs namespace or specific Event Hub
2. Select **Access control (IAM)**
3. Click **+ Add role assignment**
4. Select role: **Azure Event Hubs Data Sender**
5. Assign to: User, Group, Service Principal, or Managed Identity
6. Click **Save**

**Azure CLI:**

```bash
# Get resource ID
EH_RESOURCE_ID=$(az eventhubs namespace show \
  --name myeventhubns \
  --resource-group myResourceGroup \
  --query id \
  --output tsv)

# Assign Data Sender role to user
az role assignment create \
  --role "Azure Event Hubs Data Sender" \
  --assignee user@example.com \
  --scope $EH_RESOURCE_ID

# Assign Data Receiver role to service principal
az role assignment create \
  --role "Azure Event Hubs Data Receiver" \
  --assignee <service-principal-id> \
  --scope $EH_RESOURCE_ID

# Assign Data Owner role to managed identity
az role assignment create \
  --role "Azure Event Hubs Data Owner" \
  --assignee <managed-identity-id> \
  --scope $EH_RESOURCE_ID
```

**PowerShell:**

```powershell
# Assign role
New-AzRoleAssignment `
  -SignInName user@example.com `
  -RoleDefinitionName "Azure Event Hubs Data Sender" `
  -Scope "/subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{namespace}"
```

---

## Managed Identity Authorization

**Managed Identity** eliminates the need to store credentials in code by leveraging Azure AD authentication for Azure-hosted applications.

### Types of Managed Identities

| Type | Description | Use Case |
|------|-------------|----------|
| **System-assigned** | Tied to Azure resource lifecycle | Single application |
| **User-assigned** | Standalone identity resource | Multiple applications |

### Enable Managed Identity

**Azure Portal:**
1. Navigate to your Azure resource (VM, Function App, Container Instance, etc.)
2. Select **Identity**
3. Toggle **System assigned** to **On**
4. Click **Save**

**Azure CLI:**

```bash
# Enable system-assigned managed identity for VM
az vm identity assign \
  --name myVM \
  --resource-group myResourceGroup

# Enable for Azure Function
az functionapp identity assign \
  --name myFunctionApp \
  --resource-group myResourceGroup

# Create user-assigned managed identity
az identity create \
  --name myManagedIdentity \
  --resource-group myResourceGroup

# Assign to VM
az vm identity assign \
  --name myVM \
  --resource-group myResourceGroup \
  --identities /subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{identity-name}
```

### Use Managed Identity in Code

**C# - EventHubProducerClient with Managed Identity:**

```csharp
using Azure.Identity;
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Producer;

// Event Hubs namespace (not connection string!)
string fullyQualifiedNamespace = "myeventhubns.servicebus.windows.net";
string eventHubName = "myeventhub";

// Create producer with managed identity
var producer = new EventHubProducerClient(
    fullyQualifiedNamespace,
    eventHubName,
    new DefaultAzureCredential()  // Automatically uses managed identity
);

// Send events
using EventDataBatch eventBatch = await producer.CreateBatchAsync();
eventBatch.TryAdd(new EventData("Event 1"));
eventBatch.TryAdd(new EventData("Event 2"));

await producer.SendAsync(eventBatch);
Console.WriteLine("Events sent using managed identity!");

await producer.DisposeAsync();
```

**C# - EventProcessorClient with Managed Identity:**

```csharp
using Azure.Identity;
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Consumer;
using Azure.Messaging.EventHubs.Processor;
using Azure.Storage.Blobs;

// Event Hubs configuration
string fullyQualifiedNamespace = "myeventhubns.servicebus.windows.net";
string eventHubName = "myeventhub";
string consumerGroup = EventHubConsumerClient.DefaultConsumerGroupName;

// Storage for checkpoints (also using managed identity)
string storageAccountUrl = "https://mystorageaccount.blob.core.windows.net";
string blobContainerName = "checkpoints";

// Create blob container client with managed identity
var blobContainerClient = new BlobContainerClient(
    new Uri($"{storageAccountUrl}/{blobContainerName}"),
    new DefaultAzureCredential()
);

// Create event processor with managed identity
var processor = new EventProcessorClient(
    blobContainerClient,
    consumerGroup,
    fullyQualifiedNamespace,
    eventHubName,
    new DefaultAzureCredential()  // Managed identity for Event Hubs
);

// Register handlers
processor.ProcessEventAsync += async (args) =>
{
    Console.WriteLine($"Event: {args.Data.EventBody}");
    await args.UpdateCheckpointAsync();
};

processor.ProcessErrorAsync += (args) =>
{
    Console.WriteLine($"Error: {args.Exception.Message}");
    return Task.CompletedTask;
};

// Start processing
await processor.StartProcessingAsync();
```

**Python - Using Managed Identity:**

```python
from azure.eventhub import EventHubProducerClient, EventData
from azure.identity import DefaultAzureCredential

# Event Hubs namespace (not connection string!)
fully_qualified_namespace = "myeventhubns.servicebus.windows.net"
eventhub_name = "myeventhub"

# Create producer with managed identity
credential = DefaultAzureCredential()
producer = EventHubProducerClient(
    fully_qualified_namespace=fully_qualified_namespace,
    eventhub_name=eventhub_name,
    credential=credential
)

# Send events
with producer:
    event_data_batch = producer.create_batch()
    event_data_batch.add(EventData("Event 1"))
    event_data_batch.add(EventData("Event 2"))
    producer.send_batch(event_data_batch)
    print("Events sent using managed identity!")
```

**JavaScript - Using Managed Identity:**

```javascript
const { EventHubProducerClient } = require("@azure/event-hubs");
const { DefaultAzureCredential } = require("@azure/identity");

// Event Hubs namespace (not connection string!)
const fullyQualifiedNamespace = "myeventhubns.servicebus.windows.net";
const eventHubName = "myeventhub";

// Create producer with managed identity
const credential = new DefaultAzureCredential();
const producer = new EventHubProducerClient(
    fullyQualifiedNamespace,
    eventHubName,
    credential
);

// Send events
async function main() {
    const batch = await producer.createBatch();
    batch.tryAdd({ body: "Event 1" });
    batch.tryAdd({ body: "Event 2" });
    
    await producer.sendBatch(batch);
    console.log("Events sent using managed identity!");
    
    await producer.close();
}

main().catch(console.error);
```

## Цепочка DefaultAzureCredential

**DefaultAzureCredential** автоматически пытается использовать доступные механизмы аутентификации в следующем порядке:

1. **Переменные окружения**  
   `AZURE_TENANT_ID`, `AZURE_CLIENT_ID`, `AZURE_CLIENT_SECRET`

2. **Managed Identity**  
   System-assigned или user-assigned

3. **Visual Studio**  
   Кэшированные учетные данные разработчика

4. **Azure CLI**  
   Учетные данные из `az login`

5. **Azure PowerShell**  
   Учетные данные из `Connect-AzAccount`

6. **Интерактивный вход через браузер**  
   Предлагает выполнить авторизацию вручную

---

### Лучшие практики

- ✅ Использовать `DefaultAzureCredential` для автоматического выбора способа аутентификации
- ✅ Работает локально (через Azure CLI) и в Azure (через Managed Identity)
- ✅ Не требует изменения кода между средами (dev → prod)

📌 Это рекомендуемый способ аутентификации для production-приложений.

---

# Shared Access Signatures (SAS)

**Shared Access Signature (SAS)** — механизм аутентификации на основе токена, использующий shared keys.

Чаще применяется в legacy-сценариях или при интеграции с внешними системами.

---

## Компоненты SAS

| Компонент | Описание |
|------------|-----------|
| **Shared Access Policy** | Именованная политика с набором разрешений и ключами |
| **Primary Key** | Основной ключ (можно регенерировать) |
| **Secondary Key** | Вторичный ключ (используется для ротации) |
| **SAS Token** | Подписанный токен с ограниченным сроком действия и разрешениями |

---

## Правила авторизации и разрешения

### Доступные разрешения:

| Разрешение | Описание | Операции |
|-------------|-----------|-----------|
| **Send** | Отправка событий | Публикация событий в Event Hub |
| **Listen** | Получение событий | Чтение событий из Event Hub |
| **Manage** | Управление Event Hub | Создание, обновление и удаление сущностей Event Hub |

---

## Важно понимать

- SAS-токены имеют срок действия (expiration)
- Ключи можно регенерировать без остановки сервиса (через Primary/Secondary rotation)
- SAS предоставляет доступ на основе ключа, а не идентичности
- В production предпочтительнее использовать Azure AD + RBAC

---

## Что важно для AZ-204

- `DefaultAzureCredential` — рекомендуемый подход
- Managed Identity — лучший вариант для Azure-hosted приложений
- SAS используется, если Azure AD недоступен
- Разрешения SAS: Send, Listen, Manage
- Принцип наименьших привилегий (least privilege)

---

## Экзаменационная логика

Если требуется:

- безопасный production-доступ → Azure AD / Managed Identity
- ротация ключей → использовать Primary/Secondary keys
- ограниченный доступ по времени → SAS token

Понимание различий между identity-based и key-based доступом — обязательная часть темы безопасности в AZ-204.
### Create Authorization Rule

**Azure CLI:**

```bash
# Create authorization rule with Send permission
az eventhubs eventhub authorization-rule create \
  --name SendOnlyRule \
  --eventhub-name myeventhub \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --rights Send

# Create authorization rule with Listen permission
az eventhubs eventhub authorization-rule create \
  --name ListenOnlyRule \
  --eventhub-name myeventhub \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --rights Listen

# Create authorization rule with Send and Listen
az eventhubs eventhub authorization-rule create \
  --name SendListenRule \
  --eventhub-name myeventhub \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --rights Send Listen

# Create authorization rule at namespace level (all Event Hubs)
az eventhubs namespace authorization-rule create \
  --name RootManageSharedAccessKey \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --rights Manage Send Listen
```

### Get Connection String and Keys

**Azure CLI:**

```bash
# Get connection string
az eventhubs eventhub authorization-rule keys list \
  --name SendOnlyRule \
  --eventhub-name myeventhub \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --query primaryConnectionString \
  --output tsv

# Get primary key
az eventhubs eventhub authorization-rule keys list \
  --name SendOnlyRule \
  --eventhub-name myeventhub \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --query primaryKey \
  --output tsv
```

**Connection String Format:**

```
Endpoint=sb://myeventhubns.servicebus.windows.net/;
SharedAccessKeyName=SendOnlyRule;
SharedAccessKey=<base64-encoded-key>;
EntityPath=myeventhub
```

### Generate SAS Token (Programmatically)

**C# - Generate SAS Token:**

```csharp
using System;
using System.Globalization;
using System.Net;
using System.Security.Cryptography;
using System.Text;

public static string CreateSasToken(string resourceUri, string keyName, string key, TimeSpan ttl)
{
    var expiry = DateTimeOffset.UtcNow.Add(ttl).ToUnixTimeSeconds().ToString();
    string stringToSign = WebUtility.UrlEncode(resourceUri) + "\n" + expiry;
    
    using (var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(key)))
    {
        var signature = Convert.ToBase64String(hmac.ComputeHash(Encoding.UTF8.GetBytes(stringToSign)));
        var sasToken = $"SharedAccessSignature sr={WebUtility.UrlEncode(resourceUri)}&sig={WebUtility.UrlEncode(signature)}&se={expiry}&skn={keyName}";
        return sasToken;
    }
}

// Usage
string resourceUri = "myeventhubns.servicebus.windows.net/myeventhub";
string keyName = "SendOnlyRule";
string key = "<shared-access-key>";
TimeSpan ttl = TimeSpan.FromHours(1);

string sasToken = CreateSasToken(resourceUri, keyName, key, ttl);
Console.WriteLine($"SAS Token: {sasToken}");
```

**Python - Generate SAS Token:**

```python
import hmac
import hashlib
import base64
import time
from urllib.parse import quote_plus

def create_sas_token(uri, key_name, key, ttl_seconds=3600):
    expiry = int(time.time() + ttl_seconds)
    string_to_sign = f"{quote_plus(uri)}\n{expiry}"
    
    signature = base64.b64encode(
        hmac.new(
            key.encode('utf-8'),
            string_to_sign.encode('utf-8'),
            hashlib.sha256
        ).digest()
    ).decode()
    
    sas_token = f"SharedAccessSignature sr={quote_plus(uri)}&sig={quote_plus(signature)}&se={expiry}&skn={key_name}"
    return sas_token

# Usage
resource_uri = "myeventhubns.servicebus.windows.net/myeventhub"
key_name = "SendOnlyRule"
key = "<shared-access-key>"

sas_token = create_sas_token(resource_uri, key_name, key)
print(f"SAS Token: {sas_token}")
```

### Use SAS Token in Code

**C# - EventHubProducerClient with SAS:**

```csharp
using Azure.Messaging.EventHubs;
using Azure.Messaging.EventHubs.Producer;
using Azure.Core;

string fullyQualifiedNamespace = "myeventhubns.servicebus.windows.net";
string eventHubName = "myeventhub";
string sasToken = "<generated-sas-token>";

// Create credential from SAS token
var credential = new AzureSasCredential(sasToken);

// Create producer
var producer = new EventHubProducerClient(
    fullyQualifiedNamespace,
    eventHubName,
    credential
);

// Send events
using EventDataBatch batch = await producer.CreateBatchAsync();
batch.TryAdd(new EventData("Event with SAS"));

await producer.SendAsync(batch);
await producer.DisposeAsync();
```

### SAS Publisher Policies (Fine-Grained Access)

**Publisher policies** allow per-device or per-publisher authentication.

**Create Publisher SAS Token:**

```csharp
// Publisher-specific SAS token
string publisherName = "device-001";
string resourceUri = $"myeventhubns.servicebus.windows.net/myeventhub/publishers/{publisherName}";
string sasToken = CreateSasToken(resourceUri, keyName, key, TimeSpan.FromDays(7));

// Device uses this token to publish
// Only events from this publisher are allowed
```

**Benefits:**
- ✅ **Revocation**: Revoke individual publisher without affecting others
- ✅ **Audit**: Track events by publisher
- ✅ **Security**: Limit scope to single publisher

---

## Connection Strings

**Connection strings** contain shared keys and should be used with caution.

### Connection String Format

```
Endpoint=sb://<namespace>.servicebus.windows.net/;
SharedAccessKeyName=<policy-name>;
SharedAccessKey=<base64-key>;
EntityPath=<eventhub-name>
```

**Example:**

```
Endpoint=sb://myeventhubns.servicebus.windows.net/;
SharedAccessKeyName=RootManageSharedAccessKey;
SharedAccessKey=ABC123...XYZ789;
EntityPath=myeventhub
```

### Use Connection String in Code

**C# - EventHubProducerClient:**

```csharp
string connectionString = "<connection-string>";
string eventHubName = "myeventhub";  // Can omit if in connection string

var producer = new EventHubProducerClient(connectionString, eventHubName);

// Send events
using EventDataBatch batch = await producer.CreateBatchAsync();
batch.TryAdd(new EventData("Event from connection string"));
await producer.SendAsync(batch);

await producer.DisposeAsync();
```

**Best Practices:**
- ⚠️ **Don't hardcode**: Store in Azure Key Vault or environment variables
- ⚠️ **Rotate keys**: Regularly rotate shared access keys
- ⚠️ **Prefer Azure AD**: Use managed identity when possible

---

## Network Security

### Virtual Network (VNet) Integration

**Restrict access** to Event Hubs from specific VNets and subnets.

**Enable VNet Service Endpoint:**

```bash
# Enable service endpoint on subnet
az network vnet subnet update \
  --name mySubnet \
  --vnet-name myVNet \
  --resource-group myResourceGroup \
  --service-endpoints Microsoft.EventHub

# Add VNet rule to Event Hubs
az eventhubs namespace network-rule add \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --subnet /subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet}/subnets/{subnet}
```

### IP Firewall

**Allow specific IP addresses** to access Event Hubs.

**Azure CLI:**

```bash
# Add IP rule (allow specific IP)
az eventhubs namespace network-rule add \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --ip-address 203.0.113.5

# Add IP range (CIDR notation)
az eventhubs namespace network-rule add \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --ip-address 203.0.113.0/24
```

**Azure Portal:**
1. Navigate to Event Hubs namespace
2. Select **Networking**
3. Select **Selected networks**
4. Add IP addresses or ranges
5. Click **Save**

### Private Endpoints

**Private Link** provides private connectivity to Event Hubs from VNet using private IP addresses.

**Create Private Endpoint:**

```bash
# Create private endpoint
az network private-endpoint create \
  --name myPrivateEndpoint \
  --resource-group myResourceGroup \
  --vnet-name myVNet \
  --subnet mySubnet \
  --private-connection-resource-id /subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{namespace} \
  --group-id namespace \
  --connection-name myConnection

# Create private DNS zone
az network private-dns zone create \
  --name privatelink.servicebus.windows.net \
  --resource-group myResourceGroup

# Link DNS zone to VNet
az network private-dns link vnet create \
  --name myDnsLink \
  --zone-name privatelink.servicebus.windows.net \
  --resource-group myResourceGroup \
  --virtual-network myVNet \
  --registration-enabled false
```

---

## Лучшие практики

---

# Лучшие практики аутентификации

### 1. Предпочитать Azure AD и Managed Identity

- ✅ Нет хранения секретов в коде
- ✅ Централизованное управление доступом
- ✅ Автоматическая ротация токенов
- ✅ Аудит доступа

---

### 2. Принцип наименьших привилегий (Least Privilege)

- Назначать только необходимые разрешения
- Использовать **Data Sender** для publisher
- Использовать **Data Receiver** для consumer
- Роль **Data Owner** оставлять администраторам

---

### 3. Регулярная ротация ключей

- Ротировать SAS-ключи каждые ~90 дней
- Использовать secondary key для бесшовной ротации
- Автоматизировать ротацию через Azure Key Vault

---

### 4. Безопасное хранение секретов

- Использовать Azure Key Vault для хранения connection strings
- Никогда не хардкодить учетные данные
- Использовать переменные окружения или конфигурацию

---

### 5. Мониторинг доступа

- Включить диагностическое логирование
- Отслеживать неудачные попытки аутентификации
- Настроить алерты на подозрительную активность

---

# Лучшие практики сетевой безопасности

### 1. Ограничение сетевого доступа

- Использовать VNet service endpoints
- Настроить IP firewall rules
- Использовать Private Endpoints для чувствительных систем

---

### 2. Отключение публичного доступа

- Использовать только private endpoints
- Отключить public network access в настройках namespace

---

### 3. Использование TLS 1.2+

- Установить минимальную версию TLS 1.2
- Отключить устаревшие версии TLS

---

# Устранение проблем (Troubleshooting)

## Распространённые проблемы

---

### Проблема 1: Unauthorized (401)

**Симптомы:**
- Ошибка «Unauthorized» при отправке или получении событий

**Возможные причины:**

- Неверные учетные данные
- Просроченный SAS-токен
- Отсутствует назначение RBAC-роли
- Managed Identity не включена

**Что проверить:**

- Корректность токена или connection string
- Срок действия SAS
- Назначена ли нужная роль (Data Sender / Receiver)
- Активирована ли Managed Identity
- Совпадает ли tenant Azure AD

---

## Что важно для AZ-204

- В production — Azure AD / Managed Identity
- SAS требует контроля срока действия
- RBAC-роль обязательна при использовании Azure AD
- 401 обычно означает проблему с аутентификацией или авторизацией

Понимание разницы между ошибками аутентификации (401) и сетевыми проблемами — частый экзаменационный момент.
**Resolution:**

```bash
# Verify RBAC role assignment
az role assignment list \
  --scope /subscriptions/{subscription-id}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{namespace} \
  --assignee <principal-id>

# Verify managed identity is enabled
az vm identity show --name myVM --resource-group myResourceGroup

# Test SAS token expiration
# Decode SAS token and check 'se' (expiry) field
```

**Issue 2: Forbidden (403)**

**Symptoms:**
- "Forbidden" error after successful authentication

**Possible Causes:**
- Insufficient permissions (e.g., Listen role trying to Send)
- Incorrect authorization rule
- Network access denied (firewall)

**Resolution:**

```bash
# Check assigned roles
az role assignment list --assignee <principal-id>

# Verify network rules
az eventhubs namespace network-rule list \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup
```

---

# Советы к экзамену AZ-204 (Контроль доступа к Event Hubs)

## Ключевые концепции

1. **Azure AD**  
   Предпочтительный способ аутентификации  
   (OAuth 2.0, без хранения секретов в коде)

2. **Managed Identity**  
   Лучший вариант для приложений, размещённых в Azure  
   (автоматическое управление учетными данными)

3. **RBAC-роли**
   - Data Owner — полный доступ
   - Data Sender — только отправка
   - Data Receiver — только получение

4. **SAS (Shared Access Signature)**  
   Токен на основе shared keys

5. **Connection String**  
   Содержит shared key (менее безопасный вариант)

6. **VNet Integration**  
   Ограничение доступа к Event Hubs по сети

7. **Private Endpoints**  
   Доступ через приватный IP внутри VNet

---

## Типовые экзаменационные сценарии

### Сценарий 1: Безопасная отправка событий из Azure Function

✔ Включить Managed Identity для Function App  
✔ Назначить роль **Azure Event Hubs Data Sender**  
✔ Использовать `DefaultAzureCredential()`

---

### Сценарий 2: Гибкий контроль доступа

✔ Использовать Azure AD + RBAC  
✔ Data Sender для publisher  
✔ Data Receiver для consumer

---

### Сценарий 3: Ротация ключей без простоя

✔ Использовать SAS с primary и secondary ключами  
✔ Перевести приложения на secondary ключ  
✔ Регенерировать primary ключ  
✔ Перевести приложения на новый primary ключ  
✔ Регенерировать secondary ключ

---

### Сценарий 4: Ограничение сетевого доступа

✔ Настроить VNet service endpoints  
✔ Добавить IP firewall rules  
✔ Использовать private endpoints для чувствительных систем

---

## Что обязательно помнить

- Предпочтительный вариант: **Azure AD + Managed Identity**
- Встроенные RBAC-роли: Owner, Sender, Receiver
- SAS — токен с ограниченным сроком действия и набором разрешений
- Connection string содержит shared key
- `DefaultAzureCredential` автоматически выбирает способ аутентификации
- Для ограничения сети использовать Service Endpoints или Private Endpoints
- Всегда применять принцип **наименьших привилегий (Least Privilege)**

---

## Экзаменационная логика

Если вопрос касается:

- безопасности production-системы → Azure AD / Managed Identity
- минимизации прав → RBAC + Least Privilege
- временного доступа → SAS
- сетевой изоляции → VNet или Private Endpoint

Понимание различий между identity-based и key-based доступом — ключевой момент для успешной сдачи AZ-204.### Quick Reference

```csharp
// Managed Identity (Recommended)
var producer = new EventHubProducerClient(
    "namespace.servicebus.windows.net",
    "eventhub",
    new DefaultAzureCredential()
);

// Connection String (Less secure)
var producer = new EventHubProducerClient(
    "<connection-string>",
    "eventhub"
);

// RBAC Role Assignment (Azure CLI)
az role assignment create \
  --role "Azure Event Hubs Data Sender" \
  --assignee <principal-id> \
  --scope <resource-id>
```

---

## Итог

**Контроль доступа** к Event Hubs включает два аспекта:

- **Аутентификация** — кто вы
- **Авторизация** — что вам разрешено делать

---

## Методы аутентификации

- **Azure AD (OAuth 2.0)** — рекомендуемый способ
- **Managed Identity** — лучший вариант для приложений в Azure
- **Shared Access Signatures (SAS)** — токены на основе shared keys
- **Connection Strings** — содержат shared keys (менее безопасно)

---

## Авторизация

- **RBAC-роли:**
   - Data Owner
   - Data Sender
   - Data Receiver

- **Разрешения:**
   - Send
   - Listen
   - Manage

- **Scope назначения роли:**
   - На уровне Namespace
   - На уровне конкретного Event Hub

---

## Сетевая безопасность

- VNet service endpoints
- IP firewall rules
- Private endpoints (Private Link)

---

## Лучшие практики

- Предпочитать Azure AD и Managed Identity
- Применять принцип наименьших привилегий
- Регулярно ротировать ключи
- Хранить секреты в Azure Key Vault
- Включать мониторинг доступа и аудит

---

## Главное для AZ-204

Если требуется:

- безопасный production-доступ → Azure AD + Managed Identity
- ограничение прав → RBAC + Least Privilege
- сетовая изоляция → VNet или Private Endpoint

Понимание различий между методами аутентификации и моделями авторизации — ключевой элемент темы безопасности в AZ-204.
  
- | Если нужно                               | Используй              |
  | ---------------------------------------- | ---------------------- |
  | Простой доступ одного сервиса к ресурсам | **System-assigned MI** |
  | Одна identity для нескольких сервисов    | User-assigned MI       |
- 
- System-assigned managed identity
Создаётся автоматически вместе с App Service
Жёстко привязана к ресурсу
Удаляется автоматически при удалении ресурса
Нелья удалить отдельно вручную

User-assigned managed identity:
Создаётся как отдельный ресурс
Может быть назначена нескольким сервисам
Может быть удалена отдельно
