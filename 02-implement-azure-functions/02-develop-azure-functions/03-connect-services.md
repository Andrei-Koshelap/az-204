# Connect Functions to Azure Services (Подключение Functions к сервисам Azure)

## Key Concepts (Ключевые понятия)

- **Application Settings** — зашифрованные пары ключ-значение для конфигурации
- **Connection strings** — хранятся в настройках приложения, а не в коде
- **Identity-based connections** — подключение через Managed Identity вместо секретов
- **Managed Identity** — системная или пользовательская управляемая идентичность
- **Least privilege** — минимально необходимые права доступа

---

# Application Settings (Настройки приложения)

## Purpose (Назначение)

Безопасное хранение конфигурации:

- Строки подключения
- API-ключи
- URL сервисов
- Feature flags
- Настройки для разных окружений

> 💡 Позволяет разделить код и конфигурацию (12-factor app подход).

---

## Storage (Хранение)

✅ **Encrypted at rest** — значения шифруются в Azure  
✅ Доступны как **переменные окружения** во время выполнения  
✅ Можно задавать разные значения для dev / test / prod

---

## Как используются в коде

- Через `Environment.GetEnvironmentVariable()` (C#)
- Через `process.env.NAME` (Node.js)
- Через `os.environ["NAME"]` (Python)

---

## Важно для AZ-204

- Никогда не храните connection strings в коде
- Bindings используют имя настройки (`connection`), а не саму строку
- Application Settings можно помечать как slot-specific
- Использование Managed Identity предпочтительнее, чем хранение секретов

### Configuration Sources

#### Azure (Production)
```
Function App → Configuration → Application settings
├── Name: DatabaseConnection
├── Value: Server=prod-server;Database=...
└── Deployment slot setting: ☐ (optional)
```

#### Local Development
```json
// local.settings.json
{
  "Values": {
    "DatabaseConnection": "Server=localhost;Database=...",
    "ApiKey": "dev-key-123",
    "ExternalApiUrl": "https://dev-api.example.com"
  }
}
```

## Connection String Pattern

### ❌ Don't: Hardcode Connections
```csharp
// BAD - Never do this
public static void Run([QueueTrigger("orders")] string message)
{
    var connectionString = "DefaultEndpointsProtocol=https;AccountName=...";
    // Use connection string
}
```

**Problems**:
- Security risk
- Can't change without redeployment
- No separation between environments

### ✅ Do: Use App Settings
```csharp
// GOOD - Reference app setting
public static void Run(
    [QueueTrigger("orders", Connection = "MyStorageConnection")] string message)
{
    // Connection retrieved from app setting automatically
}
```

### Binding Connection Property
```json
{
  "type": "queueTrigger",
  "direction": "in",
  "name": "message",
  "queueName": "orders",
  "connection": "MyStorageConnection"
}
```

## How It Works (Как это работает)

1️⃣ Функция ищет Application Setting с именем `MyStorageConnection`  
2️⃣ Во время выполнения получает значение  
3️⃣ Использует его для подключения к сервису

> 💡 В bindings указывается **имя настройки**, а не сама строка подключения.

---

# Set App Settings (Как задать настройки)

## Через Azure Portal

1. Откройте **Function App**
2. Перейдите в **Configuration → Application settings**
3. Нажмите **+ New application setting**
4. Укажите:
    - **Name**: `MyStorageConnection`
    - **Value**: `DefaultEndpointsProtocol=https;...`
5. Нажмите **OK → Save**

⚠️ После сохранения приложение перезапустится.

---

## Важно для AZ-204

- `connection` в binding ссылается на имя настройки
- Изменение Application Settings вызывает перезапуск
- Можно пометить настройку как **Deployment slot setting**
- Секреты не должны храниться в коде


#### CLI
```bash
# Set single setting
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "MyStorageConnection=DefaultEndpointsProtocol=https;..."

# Set multiple settings
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings \
    "DatabaseConnection=Server=..." \
    "ApiKey=prod-key-456" \
    "FeatureFlag=enabled"
```

#### Upload from local.settings.json
```bash
# Upload all settings at once
func azure functionapp publish <function-app-name> --publish-settings-only
```

## Identity-Based Connections

### What Is Identity-Based Connection?
**Use managed identity** instead of connection strings/secrets:

Traditional (secret-based):
```
Connection = "AccountKey=supersecretkey123..."
```

Identity-based (no secrets):
```
Connection = "MyStorageConnection"
MyStorageConnection__serviceUri = "https://mystorageaccount.blob.core.windows.net"
```

## Benefits (Преимущества Identity-based подключения)

✅ **Без секретов** — нет ключей, которые нужно хранить и ротировать  
✅ **Azure AD authentication** — безопасная аутентификация через Entra ID  
✅ **Least privilege** — доступ контролируется через RBAC  
✅ **Автоматическая ротация** — нет ручного управления ключами

> 💡 Рекомендуемый способ подключения к Azure-сервисам — через Managed Identity.

---

## Supported Services (Поддерживаемые сервисы)

| Service | Identity Support | Extension |
|----------|------------------|------------|
| **Azure Storage** | ✅ Yes | Blobs, Queues, Tables |
| **Azure Cosmos DB** | ✅ Yes | SQL API |
| **Azure Service Bus** | ✅ Yes | Queues, Topics |
| **Azure Event Hubs** | ✅ Yes | Event streams |
| **Azure SQL Database** | ✅ Yes | SQL connections |

---

## ⚠️ Azure Files Exception

Storage-аккаунт, который используется самой Function App  
(`WEBSITE_AZUREFILESCONNECTIONSTRING`),

должен использовать **connection string**, а не Managed Identity.

> 📌 Это системное требование платформы.


### Configuration

#### Step 1: Enable Managed Identity
```bash
# Enable system-assigned identity
az functionapp identity assign \
  --name <function-app-name> \
  --resource-group <rg-name>

# Output includes principalId:
# {
#   "principalId": "12345678-1234-1234-1234-123456789012",
#   "tenantId": "...",
#   "type": "SystemAssigned"
# }
```

#### Step 2: Configure Connection Settings
```bash
# Storage Blob identity-based connection
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings \
    "MyStorageConnection__serviceUri=https://mystorageaccount.blob.core.windows.net" \
    "MyStorageConnection__credential=managedidentity"
```

**Setting format**:
```
<ConnectionName>__serviceUri = <service-endpoint>
<ConnectionName>__credential = managedidentity
<ConnectionName>__clientId = <client-id>  (optional, for user-assigned)
```

#### Step 3: Grant Permissions
```bash
# Get function app identity
PRINCIPAL_ID=$(az functionapp identity show \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --query principalId -o tsv)

# Grant Storage Blob Data Contributor role
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Storage Blob Data Contributor" \
  --scope "/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<storage-account>"
```

### User-Assigned Identity
```bash
# Create user-assigned identity
az identity create \
  --name MyFunctionIdentity \
  --resource-group <rg-name>

# Assign to function app
az functionapp identity assign \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --identities <identity-resource-id>

# Configure connection with clientId
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings \
    "MyConnection__serviceUri=https://..." \
    "MyConnection__credential=managedidentity" \
    "MyConnection__clientId=<user-assigned-identity-client-id>"
```

## Common Azure Service Connections

### Azure Storage (Blob)

#### Connection String Method
```bash
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "MyStorageConnection=DefaultEndpointsProtocol=https;AccountName=mystorageaccount;AccountKey=..."
```

#### Identity Method
```bash
# Configure settings
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "MyStorageConnection__serviceUri=https://mystorageaccount.blob.core.windows.net"

# Grant role
az role assignment create \
  --assignee <function-app-principal-id> \
  --role "Storage Blob Data Contributor" \
  --scope <storage-account-resource-id>
```

### Azure Cosmos DB

#### Connection String Method
```bash
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "CosmosDBConnection=AccountEndpoint=https://mycosmosdb.documents.azure.com:443/;AccountKey=..."
```

#### Identity Method
```bash
# Configure settings
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "CosmosDBConnection__accountEndpoint=https://mycosmosdb.documents.azure.com"

# Grant role (Cosmos DB Built-in Data Contributor)
az cosmosdb sql role assignment create \
  --account-name <cosmos-account> \
  --resource-group <rg-name> \
  --role-definition-name "Cosmos DB Built-in Data Contributor" \
  --principal-id <function-app-principal-id> \
  --scope "/"
```

### Azure Service Bus

#### Connection String Method
```bash
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "ServiceBusConnection=Endpoint=sb://myservicebus.servicebus.windows.net/;SharedAccessKeyName=..."
```

#### Identity Method
```bash
# Configure settings
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "ServiceBusConnection__fullyQualifiedNamespace=myservicebus.servicebus.windows.net"

# Grant role
az role assignment create \
  --assignee <function-app-principal-id> \
  --role "Azure Service Bus Data Receiver" \
  --scope <service-bus-namespace-resource-id>
```

### Azure SQL Database

#### Connection String Method
```bash
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "SqlConnection=Server=tcp:myserver.database.windows.net,1433;Database=..."
```

#### Identity Method
```csharp
// In code, use managed identity token
var connectionString = $"Server=tcp:myserver.database.windows.net;Database=mydb;";
var connection = new SqlConnection(connectionString);
connection.AccessToken = await new DefaultAzureCredential().GetTokenAsync(
    new TokenRequestContext(new[] { "https://database.windows.net/.default" }));
```

```bash
# Grant SQL permissions
# In SQL Database:
# CREATE USER [function-app-name] FROM EXTERNAL PROVIDER;
# ALTER ROLE db_datareader ADD MEMBER [function-app-name];
# ALTER ROLE db_datawriter ADD MEMBER [function-app-name];
```

## Local Development with Identity

### DefaultAzureCredential
**Automatic credential chain** for local dev:

1. Environment variables
2. Managed identity (when running in Azure)
3. Visual Studio
4. Azure CLI
5. Azure PowerShell

### Local Testing
```bash
# Login with Azure CLI (for local dev)
az login

# Function uses your Azure CLI credentials locally
# In Azure, switches to managed identity automatically
```

### Code Example (C#)
```csharp
using Azure.Identity;
using Azure.Storage.Blobs;

[FunctionName("BlobFunction")]
public static async Task Run(
    [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
    ILogger log)
{
    var credential = new DefaultAzureCredential();
    var blobServiceClient = new BlobServiceClient(
        new Uri("https://mystorageaccount.blob.core.windows.net"),
        credential);
    
    // Use client
}
```

# Required Permissions (RBAC Roles)

## Azure Storage

| Operation | Role |
|------------|------|
| **Read blobs** | Storage Blob Data Reader |
| **Write blobs** | Storage Blob Data Contributor |
| **Read queues** | Storage Queue Data Reader |
| **Write queues** | Storage Queue Data Contributor |
| **Read tables** | Storage Table Data Reader |
| **Write tables** | Storage Table Data Contributor |

---

## Azure Cosmos DB

| Operation | Role |
|------------|------|
| **Read data** | Cosmos DB Built-in Data Reader |
| **Write data** | Cosmos DB Built-in Data Contributor |

---

## Azure Service Bus

| Operation | Role |
|------------|------|
| **Receive messages** | Azure Service Bus Data Receiver |
| **Send messages** | Azure Service Bus Data Sender |
| **Full access** | Azure Service Bus Data Owner |

---

## Azure Event Hubs

| Operation | Role |
|------------|------|
| **Receive events** | Azure Event Hubs Data Receiver |
| **Send events** | Azure Event Hubs Data Sender |
| **Full access** | Azure Event Hubs Data Owner |

---

# Best Practices (Лучшие практики)

## 1️⃣ Use Identity-Based Connections

✅ Предпочтительно — **Managed Identity**  
⚠️ Используйте connection strings только если identity не поддерживается

> 💡 Identity-based доступ безопаснее и не требует хранения секретов.

---

## 2️⃣ Separate Environments (Разделяйте окружения)

Пример:
Dev: MyStorage → dev-storage-account
Test: MyStorage → test-storage-account
Prod: MyStorage → prod-storage-account


Один и тот же ключ настройки (`MyStorage`),  
но разные значения в разных окружениях.

---

## 3️⃣ Least Privilege (Минимальные права)

Назначайте только необходимые роли:

- Только чтение, если функция не пишет данные
- Ограничение на конкретный контейнер или очередь (если возможно)

> 🎯 Никогда не давайте Data Owner без необходимости.

---

## Важно для AZ-204

- Identity-based подключение предпочтительнее connection strings
- RBAC роли назначаются на ресурс или на уровень контейнера
- Настройки окружения различаются для dev/test/prod
- Следуйте принципу минимальных привилегий

### 4. Key Vault References
For secrets that must be stored:
```bash
# Store in Key Vault, reference in app settings
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "ApiKey=@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/ApiKey/)"
```

### 5. Never Commit Secrets
```
# .gitignore
local.settings.json
*.publish.xml
*.user
```

## Troubleshooting

### Connection Issues
```bash
# Check app settings
az functionapp config appsettings list \
  --name <function-app-name> \
  --resource-group <rg-name>

# Check managed identity
az functionapp identity show \
  --name <function-app-name> \
  --resource-group <rg-name>

# Check role assignments
az role assignment list \
  --assignee <principal-id> \
  --output table
```

# Common Errors (Типичные ошибки)

### ❌ "Identity not found"
✅ Включите **Managed Identity** для Function App  
(Identity → System assigned → On)

---

### ❌ "Insufficient permissions"
✅ Назначьте соответствующую **RBAC роль** ресурсу

---

### ❌ "Connection string missing"
✅ Добавьте Application Setting с корректным именем  
(имя должно совпадать со значением `connection` в binding)

---

### ❌ "Identity not supported locally"
✅ Выполните `az login`  
или задайте переменные окружения вручную

> 💡 Для локальной разработки часто используется `DefaultAzureCredential`.

---

# Critical Notes (Критически важные моменты)

- 💡 **Application Settings** — зашифрованные пары ключ-значение
- ⚠️ Свойство `connection` указывает на **имя настройки**, а не на её значение
- 🎯 Identity-based подключение предпочтительнее connection strings
- 📊 Managed Identity бывает:
    - System-assigned
    - User-assigned
- ✅ Принцип **least privilege**
- 🔄 **DefaultAzureCredential** работает локально и в Azure
- ⏱️ Для identity-based подключения требуются RBAC роли
- 🔒 Никогда не коммитьте `local.settings.json`

---

# Exam Tips (Советы для экзамена)

- Application Settings:
    - Хранятся зашифрованными
    - Доступны как переменные окружения
- `connection` в binding → имя App Setting
- Никогда не хардкодьте connection strings
- Identity-based подключение не требует секретов
- **System-assigned identity**
    - Привязана к жизненному циклу Function App
- **User-assigned identity**
    - Независима
    - Может использоваться несколькими ресурсами
- **DefaultAzureCredential**
    - Автоматическая цепочка аутентификации
    - Работает локально и в Azure

---

## Важные детали

- Исключение Azure Files:
    - Для `WEBSITE_AZUREFILESCONNECTIONSTRING` нужен connection string
- Формат настроек identity-based подключения:
    - `<Name>__serviceUri`
    - `<Name>__credential`
- Часто используемые RBAC роли:
    - Storage Blob Data Contributor
    - Cosmos DB Built-in Data Contributor
- Key Vault reference:
  @Microsoft.KeyVault(SecretUri=...)

- Локальное тестирование:
- `az login`
- Azurite для Storage bindings

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-azure-functions/4-connect-azure-services)
