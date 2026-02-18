# Service Principals and Application Objects

## Ключевые понятия

- **Application Object** — глобальный шаблон (blueprint) приложения
- **Service Principal** — локальный экземпляр приложения в конкретном tenant
- **Три типа service principal** — Application, Managed Identity, Legacy
- **Регистрация приложения** — автоматически создаёт оба объекта

---

# Обзор

Service Principals и Application Objects используются для настройки идентификации приложений в Microsoft Entra ID.

## Application Object

- Создаётся один раз при регистрации приложения.
- Существует в **home tenant**.
- Содержит глобальную конфигурацию:
    - redirect URI
    - разрешения (API permissions)
    - scopes
    - app roles
    - сертификаты и client secrets

> 💡 Это «шаблон» приложения, описывающий его возможности и настройки.

---

## Service Principal

- Создаётся в каждом tenant, где используется приложение.
- Представляет приложение как объект безопасности.
- Используется для:
    - назначения ролей
    - выдачи разрешений
    - контроля доступа к ресурсам

> 💡 Если приложение multi-tenant, в каждом новом tenant будет создан свой service principal.

---

# Связь между объектами

- **Application Object** — глобальное описание приложения.
- **Service Principal** — конкретная реализация этого приложения в отдельном tenant.
- Один application object может иметь несколько service principals.
- Service principal создаётся автоматически при первом использовании приложения в новом tenant.

---

# Три типа Service Principal

1. **Application**  
   Создаётся для зарегистрированного приложения.

2. **Managed Identity**  
   Используется Azure-ресурсами для доступа к другим сервисам без хранения секретов.

3. **Legacy**  
   Устаревшие объекты, созданные до текущей модели регистрации.

---

# Важно для AZ-204

- Регистрация приложения создаёт Application Object в home tenant.
- Service Principal создаётся автоматически.
- Для multi-tenant приложений создаются service principals в каждом tenant.
- Разрешения и роли назначаются service principal, а не application object.
- Managed Identity — это особый тип service principal.

---

> 🎯 Частый экзаменационный вопрос:  
> Application Object — это шаблон.  
> Service Principal — это объект безопасности, который используется для авторизации.

Готов продолжать следующий раздел.

## Application Registration

### Registration Process

**Register app in Azure Portal**:

```bash
# Navigate to
Azure Portal → Microsoft Entra ID → App registrations → New registration
```

**Configuration**:

| Setting | Options | Description |
|---------|---------|-------------|
| **Name** | Your app name | Display name for the application |
| **Supported account types** | Single-tenant, Multi-tenant | Who can use the application |
| **Redirect URI** | Web, SPA, Mobile | Where auth responses are sent |

### Tenant Types

**1. Single-Tenant**:
- Accessible only in your tenant
- Most common for line-of-business apps
- More restrictive, more secure

```json
{
  "signInAudience": "AzureADMyOrg"
}
```

**2. Multi-Tenant**:
- Accessible in other tenants
- Common for SaaS applications
- Requires consent in each tenant

```json
{
  "signInAudience": "AzureADMultipleOrgs"
}
```

### Automatic Creation

**При регистрации приложения Azure автоматически создаёт:**

1. ✅ **Application Object** — в вашем home tenant
2. ✅ **Service Principal** — в вашем home tenant
3. ✅ **Application (Client) ID** — глобально уникальный идентификатор

> 💡 Application (Client) ID используется приложением для идентификации при запросе токенов.

---

# Application Object

## Что такое Application Object?

**Глобальный шаблон (template)** приложения в Microsoft Entra ID.

### Основные характеристики

- **Местоположение** — существует только в home tenant (где зарегистрировано приложение)
- **Уникальность** — один объект на приложение (глобально уникален)
- **Назначение** — служит шаблоном для service principals
- **Свойства** — содержит статическую конфигурацию, применяемую ко всем экземплярам приложения

> 💡 Это логическое описание приложения, а не объект, которому напрямую назначаются роли доступа к ресурсам.

---

## Три ключевых аспекта, определяемых Application Object

| Аспект | Описание |
|--------|----------|
| **Token Issuance** | Определяет, как выдаются токены для доступа к приложению |
| **Resource Access** | Указывает, к каким ресурсам требуется доступ |
| **Actions** | Описывает операции, которые приложение может выполнять |

---

## Что входит в конфигурацию

- Redirect URI
- Поддерживаемые типы аккаунтов
- API permissions
- App roles
- Scopes
- Client secrets и сертификаты

---

## Важно для AZ-204

- Application Object создаётся один раз и находится только в home tenant.
- Он не используется напрямую для назначения ролей Azure RBAC.
- Service Principal создаётся на его основе.
- Client ID связан именно с Application Object.

> 🎯 Частый вопрос: где хранится глобальная конфигурация приложения?  
Ответ: в Application Object.


### Application Object Properties

**Defined via Microsoft Graph [Application entity](https://learn.microsoft.com/en-us/graph/api/resources/application)**:

```json
{
  "id": "00000000-0000-0000-0000-000000000000",
  "appId": "00001111-aaaa-2222-bbbb-3333cccc4444",
  "displayName": "My Application",
  "signInAudience": "AzureADMyOrg",
  "web": {
    "redirectUris": [
      "https://localhost:5001/signin-oidc"
    ],
    "implicitGrantSettings": {
      "enableIdTokenIssuance": true,
      "enableAccessTokenIssuance": false
    }
  },
  "requiredResourceAccess": [
    {
      "resourceAppId": "00000003-0000-0000-c000-000000000000",
      "resourceAccess": [
        {
          "id": "e1fe6dd8-ba31-4d61-89e7-88639da4683d",
          "type": "Scope"
        }
      ]
    }
  ],
  "keyCredentials": [],
  "passwordCredentials": []
}
```

### Key Properties

```csharp
// Reading application object
using Microsoft.Graph;

var graphClient = new GraphServiceClient(...);

var application = await graphClient.Applications["{application-object-id}"]
    .Request()
    .GetAsync();

Console.WriteLine($"App ID: {application.AppId}");
Console.WriteLine($"Display Name: {application.DisplayName}");
Console.WriteLine($"Sign-in Audience: {application.SignInAudience}");
```

# Service Principal Object

## Что такое Service Principal?

**Локальное представление приложения** в конкретном tenant.

### Основные характеристики

- **Назначение** — представляет экземпляр приложения в определённом tenant
- **Security Principal** — объект безопасности, которому назначаются роли и разрешения
- **Создание** — создаётся в каждом tenant, где используется приложение
- **Связь** — ссылается на глобальный Application Object

> 💡 Service Principal — это «учётная запись» приложения в конкретном каталоге.

---

## Зачем нужны Service Principals?

Service Principal — это объект безопасности для приложения, аналогичный user principal для пользователя.
User → User Principal → Права доступа пользователя
App → Service Principal → Права доступа приложения

### Обеспечивает

- ✅ Аутентификацию приложения
- ✅ Авторизацию при доступе к ресурсам
- ✅ Определение политик доступа
- ✅ Управление разрешениями

> 🎯 Все роли и разрешения назначаются именно Service Principal, а не Application Object.

---

# Три типа Service Principals

## 1️⃣ Application Service Principal

**Стандартный экземпляр приложения**

### Характеристики

- Наиболее распространённый тип
- Создаётся при регистрации приложения или при первом использовании в tenant
- Ссылается на глобальный Application Object
- Определяет, что приложение может делать в конкретном tenant

### Использование

- Web-приложения
- API
- SaaS-приложения
- Daemon-сервисы

---

## Важно для AZ-204

- Service Principal — объект безопасности, которому назначаются роли Azure RBAC.
- Multi-tenant приложение создаёт Service Principal в каждом tenant.
- Application Object — шаблон, Service Principal — рабочий объект.

> 🎯 Частый экзаменационный вопрос:  
Кому назначаются роли и разрешения? Ответ — Service Principal.


**Creation**:

```csharp
// Automatically created when:
// 1. You register the app (in home tenant)
// 2. Admin/user consents to app in their tenant (multi-tenant apps)
```

**Example scenario**:

```
1. Company A registers a multi-tenant SaaS app
   → Application Object created in Company A's tenant
   → Service Principal created in Company A's tenant

2. Company B consents to use the app
   → Service Principal created in Company B's tenant
   → References same Application Object from Company A

3. Company C consents to use the app
   → Service Principal created in Company C's tenant
   → References same Application Object from Company A
```

### 2. Managed Identity Service Principal

**Represents a [Managed Identity](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)**:

## 2️⃣ Managed Identity Service Principal

### Назначение

- **Identity для Azure-ресурсов**  
  Используется для аутентификации Azure-сервисов при доступе к другим ресурсам.

- **Без учётных данных**  
  Не требуется управлять паролями или сертификатами.

- **Автоматическое создание**  
  Создаётся при включении Managed Identity для ресурса.

- **Системное управление**  
  Свойства управляются Azure и не редактируются вручную.

- **Используется Azure-сервисами**  
  VM, App Service, Azure Functions и другие сервисы.

> 💡 Managed Identity — это специальный тип Service Principal, управляемый платформой.

---

## Типы Managed Identities

| Тип | Описание | Типовой сценарий |
|------|----------|------------------|
| **System-Assigned** | Привязана к одному Azure-ресурсу | Ресурс получает доступ к Key Vault |
| **User-Assigned** | Отдельный ресурс, который можно использовать повторно | Несколько ресурсов используют одну и ту же идентичность |

---

### System-Assigned

- Создаётся и удаляется вместе с ресурсом.
- Связана только с одним конкретным ресурсом.
- Проста в настройке.

### User-Assigned

- Создаётся как отдельный Azure-ресурс.
- Может быть назначена нескольким ресурсам.
- Удобна для повторного использования и централизованного управления доступом.

---

## Важно для AZ-204

- Managed Identity устраняет необходимость хранения client secret.
- Используется для безопасного доступа к Azure-ресурсам.
- System-assigned удаляется вместе с ресурсом.
- User-assigned можно повторно использовать.
- Это разновидность Service Principal.

> 🎯 Частый вопрос:  
Как безопасно предоставить VM доступ к Key Vault без хранения секретов?  
Ответ — использовать Managed Identity.

**Example - System-Assigned Managed Identity**:

```bash
# Enable managed identity on Azure VM
az vm identity assign \
    --resource-group MyResourceGroup \
    --name MyVM

# Output includes principalId (service principal object ID)
{
  "principalId": "11111111-1111-1111-1111-111111111111",
  "tenantId": "22222222-2222-2222-2222-222222222222",
  "type": "SystemAssigned"
}
```

**Using in code**:

```csharp
using Azure.Identity;
using Azure.Security.KeyVault.Secrets;

// Use DefaultAzureCredential - automatically uses managed identity
var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential()
);

// No credentials needed - managed identity handles authentication
var secret = await client.GetSecretAsync("MySecret");
```

## Преимущества Managed Identity

- ✅ Отсутствие учётных данных в коде
- ✅ Автоматическая ротация учётных данных
- ✅ Управление жизненным циклом со стороны Azure
- ✅ Невозможность ручного изменения критических свойств

> 💡 Managed Identity снижает риск утечки секретов и упрощает эксплуатацию.

---

# 3️⃣ Legacy Service Principal

## Что это такое?

**Устаревший тип service principal**, созданный до внедрения современной модели регистрации приложений.

### Характеристики

- Создавался через старые механизмы управления Azure AD
- Не имеет связанного Application Object
- Можно редактировать вручную
- Подход считается устаревшим
- Рекомендуется миграция на современную модель App Registration

---

## Какие свойства могут иметь Legacy Service Principals

- Credentials (секреты или сертификаты)
- Service principal names
- Reply URLs
- Дополнительные параметры конфигурации

---

## Важно для AZ-204

- Современная модель использует связку Application Object + Service Principal.
- Legacy service principals не имеют полноценной поддержки современной архитектуры.
- Для новых решений следует использовать App Registration.
- Managed Identity — предпочтительный вариант для Azure-ресурсов.

> 🎯 Экзамен может проверять различие между Application Service Principal, Managed Identity и Legacy Service Principal.


**Example (deprecated pattern)**:

```powershell
# Old way (deprecated)
New-AzADServicePrincipal -DisplayName "LegacyApp"

# Modern way (recommended)
New-AzADApplication -DisplayName "ModernApp"
```

## Relationship Between Objects

### One-to-Many Relationship

```
┌─────────────────────────────────────┐
│   Application Object (Global)       │
│   Home Tenant: Contoso              │
│   - App ID: 00001111-aaaa...        │
│   - Display Name: My SaaS App       │
│   - Configuration/Template          │
└──────────────┬──────────────────────┘
               │
               │ Creates/References
               │
    ┌──────────┴─────────────┬─────────────────┐
    │                        │                 │
    ▼                        ▼                 ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│ Service         │  │ Service         │  │ Service         │
│ Principal       │  │ Principal       │  │ Principal       │
│ (Contoso)       │  │ (Fabrikam)      │  │ (Woodgrove)     │
│ - Local config  │  │ - Local config  │  │ - Local config  │
│ - Permissions   │  │ - Permissions   │  │ - Permissions   │
└─────────────────┘  └─────────────────┘  └─────────────────┘
    Home Tenant         Tenant 2             Tenant 3
```

## Application Object Relationships

### Связи Application Object

**Application Object имеет следующие отношения:**

- ✅ **One-to-one** — соответствует одному конкретному программному приложению
- ✅ **One-to-many** — может иметь несколько Service Principal (в разных tenant)

> 💡 Один Application Object → несколько Service Principals (по одному в каждом tenant, где используется приложение).

---

## Основные свойства Application Object

- **Глобально уникален**  
  Связан с уникальным Application (Client) ID.

- **Существует в home tenant**  
  Создаётся в tenant, где было зарегистрировано приложение.

- **Является шаблоном**  
  Используется как основа для создания Service Principals.

---


- Application Object хранит глобальную конфигурацию.
- Service Principal реализует доступ и разрешения в конкретном tenant.

---

## Важно для AZ-204

- Application Object существует только один раз.
- Service Principals создаются по мере использования приложения в других tenant.
- Роли и разрешения назначаются Service Principal.
- Multi-tenant приложения имеют несколько Service Principals.

> 🎯 Частый вопрос:  
Где хранится глобальная конфигурация приложения?  
Ответ — в Application Object.



### Service Principal Creation

**Single-Tenant Application**:

```
1. Register app
   → Application Object created
   → Service Principal created (home tenant only)
   → Consent given during registration

2. Users sign in
   → Use service principal in home tenant
```

**Multi-Tenant Application**:

```
1. Register app (Tenant A)
   → Application Object created in Tenant A
   → Service Principal created in Tenant A

2. User from Tenant B consents
   → Service Principal created in Tenant B
   → References Application Object in Tenant A

3. User from Tenant C consents
   → Service Principal created in Tenant C
   → References Application Object in Tenant A
```

## Creating Service Principals

### Automatic Creation

**When registering in portal**:

```
Azure Portal → Microsoft Entra ID → App registrations → New
→ Both Application Object and Service Principal created automatically
```

### Programmatic Creation

**Using Azure CLI**:

```bash
# Create app registration and service principal
az ad sp create-for-rbac --name "MyApp" \
    --role "Contributor" \
    --scopes /subscriptions/{subscription-id}

# Output
{
  "appId": "00001111-aaaa-2222-bbbb-3333cccc4444",
  "displayName": "MyApp",
  "password": "...",
  "tenant": "22222222-2222-2222-2222-222222222222"
}
```

**Using PowerShell**:

```powershell
# Create application
$app = New-AzADApplication -DisplayName "MyApp"

# Create service principal from application
$sp = New-AzADServicePrincipal -ApplicationId $app.AppId

Write-Host "Application ID: $($app.AppId)"
Write-Host "Service Principal ID: $($sp.Id)"
```

**Using Microsoft Graph API**:

```http
POST https://graph.microsoft.com/v1.0/applications
Content-Type: application/json

{
  "displayName": "My Application",
  "signInAudience": "AzureADMyOrg"
}

# Response includes application object ID

# Then create service principal
POST https://graph.microsoft.com/v1.0/servicePrincipals
Content-Type: application/json

{
  "appId": "00001111-aaaa-2222-bbbb-3333cccc4444"
}
```

## Managing Application Objects

### Read Application

```csharp
using Microsoft.Graph;
using Azure.Identity;

var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
var graphClient = new GraphServiceClient(credential);

// Get application by object ID
var app = await graphClient.Applications["{object-id}"]
    .Request()
    .GetAsync();

Console.WriteLine($"App ID: {app.AppId}");
Console.WriteLine($"Display Name: {app.DisplayName}");
```

### Update Application

```csharp
var app = await graphClient.Applications["{object-id}"]
    .Request()
    .GetAsync();

// Update properties
app.DisplayName = "Updated Name";
app.Web.RedirectUris.Add("https://new-redirect-uri.com");

await graphClient.Applications["{object-id}"]
    .Request()
    .UpdateAsync(app);
```

### Delete Application

```csharp
// Deleting application also deletes associated service principals
await graphClient.Applications["{object-id}"]
    .Request()
    .DeleteAsync();
```

## Managing Service Principals

### List Service Principals

```bash
# Azure CLI
az ad sp list --display-name "MyApp"

# Get by App ID
az ad sp show --id "00001111-aaaa-2222-bbbb-3333cccc4444"
```

```powershell
# PowerShell
Get-AzADServicePrincipal -DisplayName "MyApp"

# Get by Application ID
Get-AzADServicePrincipal -ApplicationId "00001111-aaaa-2222-bbbb-3333cccc4444"
```

### Assign Roles to Service Principal

```bash
# Azure CLI - Assign Contributor role
az role assignment create \
    --assignee "{service-principal-id}" \
    --role "Contributor" \
    --scope "/subscriptions/{subscription-id}"
```

```powershell
# PowerShell
New-AzRoleAssignment `
    -ObjectId "{service-principal-object-id}" `
    -RoleDefinitionName "Contributor" `
    -Scope "/subscriptions/{subscription-id}"
```

## Best Practices

### 1. Use Managed Identities When Possible

```csharp
// ✅ Good: Use managed identity (no credentials)
var credential = new DefaultAzureCredential();

// ❌ Bad: Use client secret in code
var credential = new ClientSecretCredential(tenantId, clientId, secret);
```

### 2. Single-Tenant for Internal Apps

```
✅ Good: Single-tenant for line-of-business apps
   - More restrictive
   - Better security
   - Simpler management

❌ Bad: Multi-tenant for internal apps
   - Unnecessary complexity
   - Broader attack surface
```

### 3. Descriptive Names

```csharp
// ✅ Good: Clear, descriptive names
DisplayName = "Contoso-HR-System-Production"

// ❌ Bad: Generic names
DisplayName = "App1"
```

### 4. Minimize Permissions

```
✅ Good: Grant only required permissions
   - Read-only when possible
   - Specific scopes

❌ Bad: Grant broad permissions
   - Full control unnecessarily
   - Administrative access by default
```

## Common Patterns

### Pattern 1: Multi-Tenant SaaS

```
1. Register app as multi-tenant
2. Application Object created in your tenant
3. Service Principal created in your tenant
4. Customers consent to app
5. Service Principal created in each customer tenant
6. Each Service Principal has customer-specific permissions
```

### Pattern 2: Managed Identity for Azure Resource

```
1. Enable managed identity on Azure resource (VM, App Service, etc.)
2. Service Principal created automatically
3. No application object needed
4. Assign Azure RBAC roles to service principal
5. Resource uses identity to access other Azure services
```

### Pattern 3: Daemon Application

```
1. Register app as single-tenant
2. Create client secret or certificate
3. Application Object and Service Principal created
4. Grant application permissions (not delegated)
5. Admin consent required
6. App acquires tokens using client credentials
```

# Critical Notes

- 💡 **Application Object** — глобальный шаблон в home tenant
- 🎯 **Service Principal** — локальный экземпляр приложения в каждом tenant
- ✅ **One-to-many** — один application object → несколько service principals
- ⚠️ **Три типа** — Application, Managed Identity, Legacy
- 🔄 **Регистрация приложения** — автоматически создаёт оба объекта
- 📊 **Single-tenant** — доступно только в одном tenant, более строгая модель
- 💡 **Multi-tenant** — доступно в нескольких tenant, требуется согласие (consent)
- ✅ **Managed Identity** — оптимальный вариант для Azure-ресурсов (без учётных данных)
- ⚠️ **Service Principal** — представляет приложение в конкретном tenant
- 🔒 **Security principal** — определяет политики доступа и разрешения
- 🎯 **Автоматическое создание** — через Portal, CLI, PowerShell, Graph API
- 💡 **Azure RBAC** — роли назначаются service principal
- ⚠️ **Best practice** — использовать managed identities, когда это возможно

---

# Exam Tips (AZ-204)

## Базовые определения

- **Application Object** — глобальный шаблон в home tenant.
- **Service Principal** — локальное представление приложения в tenant.
- Связь: один application object → много service principals.

---

## Типы Service Principals

1. **Application Service Principal**  
   Наиболее распространённый тип, представляет зарегистрированное приложение.

2. **Managed Identity Service Principal**  
   Используется Azure-ресурсами, не требует управления секретами.

3. **Legacy Service Principal**  
   Устаревшая модель, рекомендуется миграция.

---

## Регистрация приложения

- Создаёт:
    - Application Object
    - Service Principal (в home tenant)

- Service Principal в другом tenant создаётся автоматически при consent.

---

## Single vs Multi-tenant

- **Single-tenant**  
  Доступно только в вашем tenant.

- **Multi-tenant**  
  Доступно в других tenant после согласия администратора или пользователя.

---

## Managed Identities

- **System-assigned** — привязана к одному ресурсу.
- **User-assigned** — отдельный ресурс, может использоваться несколькими сервисами.
- Используются через DefaultAzureCredential.
- Рекомендуются для доступа к Azure-ресурсам.

---

## Что определяет Application Object

- Выдачу токенов (token issuance)
- Доступ к ресурсам (resource access)
- Разрешённые действия (actions)

---

## RBAC

- Роли Azure назначаются **Service Principal**.
- Именно service principal получает доступ к ресурсам.

---

## Управление через инструменты

- Microsoft Graph — программное управление объектами.
- Azure CLI — создание и управление service principals.
- PowerShell — управление через командлеты.

---

## Часто проверяется на экзамене

- Различие между Application Object и Service Principal.
- Связь one-to-many.
- Когда создаётся service principal в другом tenant.
- Различие между Managed Identity и обычным service principal.
- Почему managed identity предпочтительнее client secret.
- Кому назначаются роли Azure RBAC.

---

> 🎯 Ключевая мысль:  
> Application Object — описание приложения.  
> Service Principal — объект безопасности, которому назначаются права.


[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-microsoft-identity-platform/3-app-service-principals)
