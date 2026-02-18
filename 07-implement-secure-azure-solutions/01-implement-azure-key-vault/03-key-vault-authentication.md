# Аутентификация в Azure Key Vault

## Обзор

Аутентификация в Key Vault выполняется через **Microsoft Entra ID** (ранее Azure Active Directory).  
Entra ID подтверждает личность любого **security principal**, который запрашивает доступ к ресурсам Azure.

---

## Что такое Security Principal?

**Security principal** — это сущность, которая может запрашивать доступ к ресурсам Azure.

| Тип | Описание | Примеры |
|------|----------|----------|
| **User** | Реальный пользователь в Microsoft Entra ID | alice@contoso.com, bob@contoso.com |
| **Group** | Группа пользователей | Developers, Admins, Data Scientists |
| **Service Principal** | Представляет приложение или сервис | Web app, API, background job |
| **Managed Identity** | Автоматически управляемый service principal | VM, App Service, Function App |

💡 Service Principal можно воспринимать как «учётную запись пользователя», но для приложения.

---

## Способы создания Service Principal

Для приложений существует **два основных способа** получить service principal:

---

## 1. Managed Identity (Рекомендуется) ✅

### Как работает

- Azure автоматически создаёт и управляет service principal
- Нет credential’ов для хранения или ротации
- Интеграция с библиотеками Azure Identity
- Поддерживается Azure сервисами: App Service, Functions, VM, Container Instances и др.

---

### Типы Managed Identity

| Тип | Жизненный цикл | Сценарий |
|------|---------------|-----------|
| **System-assigned** | Связан с ресурсом (удаляется вместе с ним) | Один ресурс, простой сценарий |
| **User-assigned** | Независимый жизненный цикл | Несколько ресурсов, общий доступ |

---

### Преимущества

- ✅ Нет управления credential’ами
- ✅ Автоматическая ротация
- ✅ Нет секретов в коде или конфигурации
- ✅ Azure полностью управляет безопасностью

---

### Когда использовать

- Приложение работает в Azure
- Требуется максимально безопасная модель
- Нужно избежать хранения client secret

---

### Важно для AZ-204

- Managed Identity — почти всегда правильный ответ для Azure-ресурсов.
- После создания identity нужно назначить роль (обычно через Azure RBAC).
- System-assigned проще для одиночного ресурса.
- User-assigned удобна, если одна identity используется несколькими сервисами.


**Example - Enable system-assigned managed identity:**
```bash
# For App Service
az webapp identity assign \
  --name myappservice \
  --resource-group myresourcegroup

# For Virtual Machine
az vm identity assign \
  --name myvm \
  --resource-group myresourcegroup

# For Azure Function
az functionapp identity assign \
  --name myfunctionapp \
  --resource-group myresourcegroup
```

**Grant Key Vault access:**
```bash
# Get the managed identity principal ID
PRINCIPAL_ID=$(az webapp identity show \
  --name myappservice \
  --resource-group myresourcegroup \
  --query principalId -o tsv)

# Grant Key Vault Secrets User role
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee $PRINCIPAL_ID \
  --scope /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/mykeyvault
```

## 2. Ручная регистрация приложения (Manually Register Application)

### Как работает

- Приложение регистрируется вручную в Microsoft Entra ID
- Создаётся **Application object** и соответствующий **Service Principal**
- Application object идентифицирует приложение между tenant’ами
- Управление credential’ами (сертификат или secret) выполняется вами

---

### Что важно понимать

- **Application object** — глобальное определение приложения
- **Service Principal** — локальное представление приложения в конкретном tenant
- Для аутентификации используются:
    - Client ID
    - Tenant ID
    - Certificate (предпочтительно) или Secret

---

### Когда использовать

- Приложение работает вне Azure (on-premises, другой cloud)
- Multi-tenant сценарии
- Managed Identity недоступна
- Требуется интеграция с внешними системами

---

### Рекомендации

- Предпочитать **Certificate**, а не client secret
- Настроить мониторинг срока действия сертификата
- Хранить сертификат в безопасном хранилище (например, Key Vault)

---

### Важно для AZ-204

- Если приложение размещено вне Azure → Managed Identity недоступна.
- Multi-tenant приложения требуют ручной регистрации.
- В production избегать использования client secret.
- Частая экзаменационная ловушка:
  > Приложение работает вне Azure и требует безопасный доступ  
  → Service Principal + Certificate.


**Steps:**
```bash
# 1. Create app registration
az ad app create --display-name "MyApplication"

# 2. Create service principal
az ad sp create-for-rbac \
  --name "MyApplication" \
  --role "Key Vault Secrets User" \
  --scopes /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/mykeyvault

# Output (SAVE THIS - shown only once):
# {
#   "appId": "12345678-1234-1234-1234-123456789abc",
#   "displayName": "MyApplication",
#   "password": "super-secret-password",
#   "tenant": "87654321-4321-4321-4321-876543219876"
# }
```

## Варианты учётных данных (Credential Options)

| Тип credential | Безопасность | Ротация | Рекомендация |
|----------------|--------------|----------|--------------|
| **Certificate** | Высокая | Ручная | ✅ Рекомендуется |
| **Client Secret** | Средняя | Ручная | ⚠️ Использовать с осторожностью |

---

### Вывод

- 🔐 **Certificate** безопаснее, чем client secret.
- 🔄 Ротацию сертификатов нужно планировать заранее.
- ❗ Client secret — по сути пароль, требует хранения и регулярной замены.

---

# Сравнение методов аутентификации

| Метод | Безопасность | Управление | Сценарий |
|--------|--------------|------------|----------|
| **System-Assigned Managed Identity** | ⭐⭐⭐⭐⭐ | Автоматическое | Один Azure-ресурс |
| **User-Assigned Managed Identity** | ⭐⭐⭐⭐⭐ | Автоматическое | Несколько Azure-ресурсов |
| **Service Principal + Certificate** | ⭐⭐⭐⭐ | Ручное | Вне Azure, multi-tenant |
| **Service Principal + Secret** | ⭐⭐⭐ | Ручное | Крайний случай |

---

### Что важно запомнить для AZ-204

- 🥇 Managed Identity — лучший вариант для Azure.
- 🥈 Service Principal + Certificate — допустимо вне Azure.
- 🥉 Service Principal + Secret — последний вариант.
- Если в вопросе требуется **минимизировать управление credential’ами** → правильный ответ почти всегда Managed Identity.
- Если нужно поддержать **несколько Azure-ресурсов одной identity** → User-assigned Managed Identity.

**Decision tree:**
```
Running in Azure?
├─ YES → Use Managed Identity ✅
│   └─ Single resource? → System-assigned
│       Multiple resources? → User-assigned
└─ NO → Running on-premises/other cloud?
    └─ Use Service Principal
        └─ Certificate > Secret
```

---

## Аутентификация в коде приложения

### Библиотеки Azure Identity

SDK для Key Vault использует **Azure Identity client library**, что позволяет использовать одинаковый код для аутентификации в разных средах (dev, test, prod).

---

### Доступные SDK

| Язык | Пакет | Версия |
|------|--------|---------|
| **.NET** | Azure.Identity | 1.10+ |
| **Python** | azure-identity | 1.14+ |
| **Java** | azure-identity | 1.10+ |
| **JavaScript** | @azure/identity | 4.0+ |

---

## DefaultAzureCredential

**Рекомендуемый способ аутентификации** — автоматически пробует несколько источников credential’ов.

### Цепочка credential (по порядку)

1. **EnvironmentCredential** — переменные окружения
2. **WorkloadIdentityCredential** — Kubernetes workload identity
3. **ManagedIdentityCredential** — Managed Identity (VM, App Service и др.)
4. **SharedTokenCacheCredential** — общий кэш токенов
5. **VisualStudioCredential** — аутентификация через Visual Studio
6. **VisualStudioCodeCredential** — аутентификация через VS Code
7. **AzureCliCredential** — вход через Azure CLI
8. **AzurePowerShellCredential** — вход через Azure PowerShell
9. **AzureDeveloperCliCredential** — Azure Developer CLI

---

### Преимущества

- ✅ Работает в разработке (Azure CLI, VS Code) и в production (Managed Identity)
- ✅ Не требует изменений кода между средами
- ✅ Автоматически переходит к следующему способу при ошибке

---

### Почему это важно

- В dev-среде разработчик может быть залогинен через Azure CLI.
- В production тот же код будет использовать Managed Identity.
- Это устраняет необходимость писать условную логику под разные окружения.

---

### Важно для AZ-204

- DefaultAzureCredential — почти всегда правильный выбор.
- Позволяет избежать хранения client secret.
- Частый экзаменационный сценарий:
  > Приложение должно работать локально и в Azure без изменения кода  
  → использовать DefaultAzureCredential.


### .NET Example

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

// DefaultAzureCredential tries multiple auth methods
var client = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net"),
    new DefaultAzureCredential()
);

// Get secret
KeyVaultSecret secret = await client.GetSecretAsync("MySecret");
Console.WriteLine($"Secret value: {secret.Value}");
```

**Install packages:**
```bash
dotnet add package Azure.Identity
dotnet add package Azure.Security.KeyVault.Secrets
```

**How it works:**
```
DefaultAzureCredential attempts:
1. Environment variables? ❌ Not set
2. Managed Identity? ✅ Found (App Service)
   → Uses managed identity to authenticate
   → Gets access token from Azure Instance Metadata Service (IMDS)
   → Access token used to call Key Vault API
```

### Python Example

```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# DefaultAzureCredential handles authentication
credential = DefaultAzureCredential()
client = SecretClient(
    vault_url="https://mykeyvault.vault.azure.net",
    credential=credential
)

# Get secret
secret = client.get_secret("MySecret")
print(f"Secret value: {secret.value}")
```

**Install packages:**
```bash
pip install azure-identity azure-keyvault-secrets
```

### Java Example

```java
import com.azure.identity.DefaultAzureCredentialBuilder;
import com.azure.security.keyvault.secrets.SecretClient;
import com.azure.security.keyvault.secrets.SecretClientBuilder;
import com.azure.security.keyvault.secrets.models.KeyVaultSecret;

// DefaultAzureCredential for authentication
SecretClient client = new SecretClientBuilder()
    .vaultUrl("https://mykeyvault.vault.azure.net")
    .credential(new DefaultAzureCredentialBuilder().build())
    .buildClient();

// Get secret
KeyVaultSecret secret = client.getSecret("MySecret");
System.out.println("Secret value: " + secret.getValue());
```

**Maven dependency:**
```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-identity</artifactId>
    <version>1.10.0</version>
</dependency>
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-security-keyvault-secrets</artifactId>
    <version>4.6.0</version>
</dependency>
```

### JavaScript/TypeScript Example

```javascript
const { DefaultAzureCredential } = require("@azure/identity");
const { SecretClient } = require("@azure/keyvault-secrets");

// DefaultAzureCredential for authentication
const credential = new DefaultAzureCredential();
const client = new SecretClient(
    "https://mykeyvault.vault.azure.net",
    credential
);

// Get secret
async function getSecret() {
    const secret = await client.getSecret("MySecret");
    console.log(`Secret value: ${secret.value}`);
}

getSecret();
```

**Install packages:**
```bash
npm install @azure/identity @azure/keyvault-secrets
```

---

## Authentication with REST API

Access tokens must be sent to Key Vault using the **HTTP Authorization header**.

### Request Format

```http
PUT /keys/MYKEY?api-version=7.4 HTTP/1.1
Host: mykeyvault.vault.azure.net
Authorization: Bearer <access_token>
Content-Type: application/json

{
  "kty": "RSA",
  "key_size": 2048
}
```

### Get Access Token

**Using Azure CLI:**
```bash
# Get access token for Key Vault
ACCESS_TOKEN=$(az account get-access-token \
  --resource https://vault.azure.net \
  --query accessToken -o tsv)

echo $ACCESS_TOKEN
```

**Using PowerShell:**
```powershell
# Get access token
$token = (Get-AzAccessToken -ResourceUrl "https://vault.azure.net").Token
```

**Using cURL:**
```bash
# Get secret using access token
curl -X GET "https://mykeyvault.vault.azure.net/secrets/MySecret?api-version=7.4" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

### Unauthorized Response (401)

When access token is missing or invalid, Key Vault returns **HTTP 401** with `WWW-Authenticate` header:

```http
HTTP/1.1 401 Not Authorized
WWW-Authenticate: Bearer authorization="https://login.microsoftonline.com/tenant-id", resource="https://vault.azure.net"
```

**Header parameters:**

| Parameter | Description |
|-----------|-------------|
| **authorization** | OAuth2 authorization service URL to obtain access token |
| **resource** | Resource identifier (`https://vault.azure.net`) for authorization request |

**Example - Get token manually:**
```bash
# Extract tenant ID from WWW-Authenticate header
TENANT_ID="your-tenant-id"

# Get access token using service principal
curl -X POST "https://login.microsoftonline.com/$TENANT_ID/oauth2/v2.0/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "client_id=your-client-id" \
  -d "client_secret=your-client-secret" \
  -d "scope=https://vault.azure.net/.default" \
  -d "grant_type=client_credentials"

# Response:
# {
#   "token_type": "Bearer",
#   "expires_in": 3599,
#   "access_token": "eyJ0eXAiOiJKV1QiLCJhbGc..."
# }
```

---

## Authentication Scenarios

### Scenario 1: Azure App Service

**Setup:**
```bash
# 1. Enable managed identity
az webapp identity assign \
  --name myappservice \
  --resource-group myresourcegroup

# 2. Get principal ID
PRINCIPAL_ID=$(az webapp identity show \
  --name myappservice \
  --resource-group myresourcegroup \
  --query principalId -o tsv)

# 3. Grant Key Vault access
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee $PRINCIPAL_ID \
  --scope /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/mykeyvault
```

**Code (.NET):**
```csharp
// Automatically uses App Service managed identity
var client = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net"),
    new DefaultAzureCredential()
);

var secret = await client.GetSecretAsync("ConnectionString");
```

### Scenario 2: Azure Functions

**Setup:**
```bash
# 1. Enable managed identity
az functionapp identity assign \
  --name myfunctionapp \
  --resource-group myresourcegroup

# 2. Grant access (same as App Service)
```

**Code (.NET - Function):**
```csharp
[FunctionName("GetSecret")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Function, "get")] HttpRequest req,
    ILogger log)
{
    var client = new SecretClient(
        new Uri("https://mykeyvault.vault.azure.net"),
        new DefaultAzureCredential()
    );

    var secret = await client.GetSecretAsync("ApiKey");
    return new OkObjectResult($"Secret retrieved: {secret.Value.Value}");
}
```

### Scenario 3: Virtual Machine

**Setup:**
```bash
# 1. Enable managed identity on VM
az vm identity assign \
  --name myvm \
  --resource-group myresourcegroup

# 2. Grant Key Vault access (same as above)
```

**Code (Python on VM):**
```python
from azure.identity import DefaultAzureCredential
from azure.keyvault.secrets import SecretClient

# Automatically uses VM managed identity
credential = DefaultAzureCredential()
client = SecretClient(
    vault_url="https://mykeyvault.vault.azure.net",
    credential=credential
)

secret = client.get_secret("DatabasePassword")
print(f"Retrieved secret: {secret.value}")
```

### Scenario 4: Local Development

**Setup Azure CLI authentication:**
```bash
# Login to Azure CLI
az login

# Set subscription
az account set --subscription "my-subscription"
```

**Code (same code works locally and in Azure):**
```csharp
// In Azure: Uses managed identity
// Locally: Uses Azure CLI credentials
var client = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net"),
    new DefaultAzureCredential()
);

var secret = await client.GetSecretAsync("MySecret");
```

**How DefaultAzureCredential works locally:**
```
DefaultAzureCredential attempts:
1. Environment variables? ❌ Not set
2. Managed Identity? ❌ Not in Azure
3. Shared Token Cache? ❌ Not available
4. Visual Studio? ❌ Not signed in
5. VS Code? ❌ Not signed in
6. Azure CLI? ✅ Authenticated!
   → Uses Azure CLI token
   → Same code works in Azure and locally!
```

### Scenario 5: Service Principal (Non-Azure)

**Setup:**
```bash
# Create service principal with certificate
az ad sp create-for-rbac \
  --name "MyOnPremApp" \
  --create-cert \
  --cert MyAppCert \
  --keyvault mykeyvault \
  --role "Key Vault Secrets User" \
  --scopes /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/mykeyvault
```

**Code (on-premises application):**
```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

// Use service principal with certificate
var credential = new ClientCertificateCredential(
    tenantId: "your-tenant-id",
    clientId: "your-client-id",
    certificatePath: "/path/to/cert.pfx",
    certificatePassword: "cert-password"  // if encrypted
);

var client = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net"),
    credential
);

var secret = await client.GetSecretAsync("ApiKey");
```

---

# Exam Tips (Советы к AZ-204)

🎯 **Microsoft Entra ID**  
Обязателен для любой аутентификации в Key Vault.

🎯 **Типы security principal**  
User, Group, Service Principal, Managed Identity.

🎯 **Два способа получить service principal**
- Managed Identity (рекомендуется)
- Ручная регистрация приложения

🎯 **System-assigned Managed Identity**  
Рекомендуется для Azure-ресурсов (VM, App Service, Functions).

🎯 **DefaultAzureCredential**  
Лучшая практика — автоматически перебирает несколько способов аутентификации.

🎯 **Цепочка credential**  
Environment → Managed Identity → Azure CLI → остальные методы.

🎯 **Azure Identity библиотеки**  
Доступны для .NET, Python, Java, JavaScript.

🎯 **REST API**  
Требует Bearer token в заголовке `Authorization`.

🎯 **HTTP 401**  
Возвращается при отсутствии/некорректном токене.  
Содержит заголовок `WWW-Authenticate`.

🎯 **Resource URL для получения токена**

https://vault.azure.net


🎯 **Локальная разработка**  
Использовать Azure CLI + DefaultAzureCredential.

🎯 **Без изменений кода**  
Один и тот же код работает:
- локально (Azure CLI),
- в Azure (Managed Identity).

---

### Часто проверяют на экзамене

- Когда выбирать Managed Identity.
- Как работает DefaultAzureCredential.
- Что означает ошибка 401.
- Какой resource URI использовать при получении токена.
- Принцип: не хранить credential’ы в коде.

---

## Additional Resources

- [Azure Key Vault Developer's Guide](https://learn.microsoft.com/en-us/azure/key-vault/general/developers-guide)
- [Azure Identity Documentation](https://learn.microsoft.com/en-us/dotnet/api/overview/azure/identity-readme)
- [DefaultAzureCredential Class](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredential)
- [Managed Identities for Azure Resources](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/)

[Microsoft Learn - Authenticate to Azure Key Vault](https://learn.microsoft.com/en-us/training/modules/implement-azure-key-vault/4-key-vault-authentication)
