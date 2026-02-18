# Изучение Managed Identities

## Обзор

**Managed Identities** позволяют разработчикам не управлять credential’ами (секретами, сертификатами, ключами) при доступе к ресурсам Azure.

Приложения используют Managed Identity для получения токенов Microsoft Entra без хранения и обслуживания каких-либо учётных данных.

---

## Какая проблема решается?

### ❌ Раньше

- Хранение credential’ов в конфигурации
- Секреты в Key Vault
- Сложная ротация
- Риск утечки
- Bootstrap-проблема (где хранить secret для доступа к секретам)

### ✅ Теперь

- Azure автоматически управляет identity
- Нет credential’ов в коде
- Нет необходимости в ручной ротации
- Минимизация риска компрометации

---

# Почему стоит использовать Managed Identities?

## Проблема управления credential’ами (Credential Management Challenge)

### Важно для AZ-204

- Если приложение работает в Azure → Managed Identity почти всегда правильный ответ.
- Устраняет bootstrap-проблему.
- Работает через Microsoft Entra ID.
- Не требует client secret или сертификата.


**В традиционном подходе::**
- Приложение требует доступ к ресурсу (например, Key Vault или Storage)
- Необходимо создать Service Principal
- Нужно хранить client secret или сертификат
- Требуется регулярная ротация
- Возникает риск утечки через:
  - исходный код
  - конфигурационные файлы
  - CI/CD пайплайны

```csharp
// ❌ BAD: Credentials in code/config
string connectionString = "AccountName=storage;AccountKey=ABC123...";
string clientSecret = "super-secret-password-12345";
```

**Managed Identity устраняет все эти проблемы.**
```csharp
// ✅ GOOD: No credentials needed!
var credential = new DefaultAzureCredential();  // Uses managed identity automatically
var client = new BlobServiceClient(new Uri("https://myaccount.blob.core.windows.net"), credential);
```


### Ключевые преимущества

- 🔐 Нет хранения секретов
- 🔄 Автоматическая ротация
- 🧩 Нативная интеграция с Azure SDK
- ⚙️ Работает с `DefaultAzureCredential`
- 🛡 Минимизация человеческого фактора

---

## What is a Managed Identity?

A managed identity is an **automatically managed service principal** in Microsoft Entra ID that applications use to authenticate to Azure resources.

**Key characteristics:**
- Created and managed by Azure
- Locked to only be used with Azure resources
- Automatically deleted when associated resource is deleted (system-assigned)
- No credential management required

**Architecture:**
```
Application (VM, App Service, Function)
    ↓ Uses
Managed Identity (Service Principal)
    ↓ Authenticates to
Microsoft Entra ID
    ↓ Issues token for
Azure Resource (Key Vault, Storage, SQL, etc.)
```

---

## Типы Managed Identities

### 1. System-Assigned Managed Identity

**Определение:**  
Включается непосредственно на экземпляре Azure-ресурса. Жизненный цикл полностью связан с этим ресурсом.

---

### Как работает

1. Вы включаете Managed Identity на ресурсе (VM, App Service и т.д.)
2. Azure автоматически создаёт Service Principal в Microsoft Entra ID
3. Credential’ы автоматически привязываются к ресурсу
4. При удалении ресурса identity удаляется автоматически

---

### Характеристики

| Аспект | System-Assigned |
|--------|-----------------|
| **Создание** | Создаётся вместе с Azure-ресурсом |
| **Жизненный цикл** | Связан с родительским ресурсом |
| **Совместное использование** | Нельзя использовать повторно (1:1) |
| **Удаление** | Автоматически при удалении ресурса |
| **Сценарий** | Один ресурс — одна identity |

---

### Когда использовать

- Приложение работает на одном Azure-ресурсе
- Не требуется совместное использование identity
- Нужна простая конфигурация без управления lifecycle

---

### Важно для AZ-204

- System-assigned — самый простой вариант.
- Подходит для большинства сценариев.
- Если identity должна использоваться несколькими ресурсами → нужен User-assigned.
- Частый экзаменационный сценарий:
  > Один App Service требует доступ к Key Vault  
  → включить System-Assigned Managed Identity.


**Example:**
```bash
# Enable on Virtual Machine
az vm identity assign \
  --name myvm \
  --resource-group myresourcegroup

# Enable on App Service
az webapp identity assign \
  --name myappservice \
  --resource-group myresourcegroup

# Enable on Azure Function
az functionapp identity assign \
  --name myfunctionapp \
  --resource-group myresourcegroup
```

**Visual representation:**
```
VM-1 ←→ System-Assigned Identity A (dedicated, cannot share)
VM-2 ←→ System-Assigned Identity B (dedicated, cannot share)
AppService-1 ←→ System-Assigned Identity C (dedicated, cannot share)
```

### Когда использовать System-Assigned Managed Identity

- ✅ Нагрузки с одним ресурсом (single resource workloads)
- ✅ Ресурсы, которым требуется собственная независимая identity
- ✅ Простые сценарии с моделью 1:1 (один ресурс — одна identity)
- ✅ Ресурсы, которые не пересоздаются часто

---

### Дополнительные рекомендации

- Подходит для большинства типовых сценариев (App Service → Key Vault).
- Минимум конфигурации — включается одной настройкой.
- Если ресурс часто пересоздаётся (например, в CI/CD с полным пересозданием инфраструктуры), стоит учитывать, что identity будет создаваться заново.
- Если требуется одна identity для нескольких ресурсов — лучше использовать **User-Assigned Managed Identity**.

---

### Важно для AZ-204

- Если в вопросе указан один Azure-ресурс и нет требований к совместному использованию identity → выбирать **System-Assigned Managed Identity**.
- Это самый простой и часто правильный вариант.


**Example scenario:**
```
Single web application on App Service
    ↓ Uses system-assigned identity
Accesses Key Vault to retrieve secrets
```

### 2. User-Assigned Managed Identity

**Определение:**  
Создаётся как отдельный Azure-ресурс. Жизненный цикл не зависит от ресурсов, которые её используют.

---

### Как работает

1. Создаётся User-Assigned Managed Identity как отдельный ресурс
2. Azure создаёт Service Principal в Microsoft Entra ID
3. Identity назначается одному или нескольким Azure-ресурсам
4. Identity продолжает существовать независимо от удаления ресурсов

---

### Характеристики

| Аспект | User-Assigned |
|--------|---------------|
| **Создание** | Отдельный Azure-ресурс |
| **Жизненный цикл** | Независимый (удаляется вручную) |
| **Совместное использование** | Можно назначать нескольким ресурсам |
| **Удаление** | Не удаляется автоматически |
| **Сценарий** | Несколько ресурсов используют одну identity |

---

### Когда использовать

- Несколько ресурсов должны иметь одинаковые права
- Нужен контроль жизненного цикла identity
- Инфраструктура часто пересоздаётся
- Требуется централизованное управление доступом

---

### Важно для AZ-204

- Если в вопросе говорится:
  > Несколько ресурсов должны использовать одну identity  
  → выбирать **User-Assigned Managed Identity**.
- Позволяет избежать повторной настройки ролей при пересоздании ресурсов.
- Более гибкий, но немного сложнее в управлении.


**Example:**
```bash
# 1. Create user-assigned identity
az identity create \
  --name myIdentity \
  --resource-group myresourcegroup

# 2. Assign to VM
az vm identity assign \
  --name myvm \
  --resource-group myresourcegroup \
  --identities myIdentity

# 3. Assign to App Service (same identity)
az webapp identity assign \
  --name myappservice \
  --resource-group myresourcegroup \
  --identities myIdentity
```

**Visual representation:**
```
User-Assigned Identity X
    ↓ Shared by
    ├── VM-1
    ├── VM-2
    ├── AppService-1
    └── Function-1
```
### Когда использовать User-Assigned Managed Identity

- ✅ Несколько ресурсов должны иметь одинаковые разрешения
- ✅ Ресурсы часто пересоздаются (при этом права остаются неизменными)
- ✅ Требуется предварительная авторизация на этапе provisioning
- ✅ Ресурсам нужен идентичный доступ к другим ресурсам

---

### Практические преимущества

- Можно заранее назначить роли (RBAC), ещё до создания основного ресурса.
- При пересоздании App Service / VM не требуется повторная настройка доступа.
- Удобно в инфраструктуре как код (ARM, Bicep, Terraform).

---

### Важно для AZ-204

- Если требуется **одна identity для нескольких ресурсов** → выбирать User-Assigned.
- Если ресурс часто удаляется и создаётся заново → User-Assigned снижает операционные риски.
- Если нет требования к совместному использованию → проще выбрать System-Assigned.


**Example scenario:**
```
3 VMs running same application
    ↓ All use same user-assigned identity
    ↓ Grant permissions once to identity
Access same Key Vault, Storage, and SQL Database
```

---

## Сравнение: System-Assigned vs User-Assigned

| Характеристика | System-Assigned | User-Assigned |
|----------------|----------------|---------------|
| **Создание** | Включается как часть ресурса | Отдельный Azure-ресурс |
| **Жизненный цикл** | Связан с ресурсом | Независимый |
| **Совместное использование** | Нет (только 1:1) | Да (many:1) |
| **Удаление** | Автоматическое | Ручное |
| **Управление правами** | Для каждого ресурса отдельно | Централизованное |
| **Пересоздание ресурса** | Identity теряется | Identity сохраняется |
| **Сложность** | Простая настройка | Немного сложнее |
| **Лучше всего подходит** | Один ресурс | Несколько ресурсов |

---

### Как быстро выбрать (для AZ-204)

- Один ресурс → **System-Assigned**
- Несколько ресурсов с одинаковыми правами → **User-Assigned**
- Частое пересоздание инфраструктуры → **User-Assigned**
- Нужна максимально простая конфигурация → **System-Assigned**

---

### Частая экзаменационная логика

- “One-to-one relationship” → System-Assigned
- “Shared identity across services” → User-Assigned
- “Independent lifecycle required” → User-Assigned
- “Minimal configuration, single app” → System-Assigned

---

## Common Use Cases

### System-Assigned Use Cases

1. **Single VM Application**
```
VM running web app
    ↓ System-assigned identity
Access Key Vault for app secrets
```

2. **Isolated Microservice**
```
App Service hosting API
    ↓ System-assigned identity
Access SQL Database and Storage Account
```

3. **Function App**
```
Azure Function processing data
    ↓ System-assigned identity
Read/write to Cosmos DB
```

### User-Assigned Use Cases

1. **Auto-Scaling Web Farm**
   **Сценарий:**  
   Веб-приложение работает на нескольких VM / экземплярах App Service с автоскейлингом.
```
User-Assigned Identity "WebAppIdentity"
    ↓ Shared by
    ├── VM Instance 1
    ├── VM Instance 2
    ├── VM Instance 3 (auto-scaled)
    └── VM Instance 4 (auto-scaled)
    ↓ All access
Key Vault, Storage, Application Insights
```

### Преимущества использования Managed Identity

- ✅ Можно добавлять и удалять VM без изменения прав доступа
- ✅ Identity можно настроить заранее до создания VM
- ✅ Одинаковые разрешения для всех экземпляров

---

### Какая identity подходит?

- Если все экземпляры должны использовать одинаковые разрешения → **User-Assigned Managed Identity**
- Если каждый экземпляр должен иметь собственную identity → **System-Assigned**

---

### Почему это важно

В автоскейлинге ресурсы создаются и удаляются динамически.  
User-Assigned identity позволяет:

- не переназначать RBAC при каждом масштабировании
- избежать потери прав при пересоздании экземпляров
- централизованно управлять доступом

---

### Важно для AZ-204

Если в вопросе есть:
- auto-scaling
- web farm
- multiple VM instances
- consistent permissions across instances

→ чаще всего правильный ответ — **User-Assigned Managed Identity**.


2. **Blue-Green Deployment**
   Два идентичных окружения (Blue и Green). Одно активно в production, второе используется для тестирования и переключения без простоя.

```
User-Assigned Identity "AppIdentity"
    ↓ Shared by
    ├── Blue Environment (Production)
    └── Green Environment (Staging)
    ↓ Both access
Same Azure resources with identical permissions
```
### Преимущества использования Managed Identity

- ✅ Переключение между окружениями без изменения прав доступа
- ✅ Возможность тестировать с теми же production-разрешениями
- ✅ Развёртывание без простоя (zero-downtime)

---

### Какая identity подходит?

- 🔹 Если оба окружения должны иметь одинаковые права → **User-Assigned Managed Identity**
- 🔹 Если каждое окружение изолировано и требует отдельной identity → **System-Assigned**

---

### Почему это важно

При Blue-Green deployment:

- Новый слот или среда создаётся заранее
- После тестирования происходит переключение трафика
- RBAC-права не должны меняться в момент переключения

User-Assigned identity позволяет:

- назначить права один раз
- использовать одну identity для обоих окружений
- избежать ошибок доступа при switch-over

---

### Важно для AZ-204

Если в вопросе упоминается:
- blue-green
- deployment slots
- zero downtime
- switching environments without reconfiguring permissions

→ вероятный правильный ответ — **User-Assigned Managed Identity**.

### 3. Консистентность Dev/Test/Prod
Одинаковая модель доступа во всех средах — разработка, тестирование и production.
```
User-Assigned Identity per environment
    ├── Dev Identity → Dev resources
    ├── Test Identity → Test resources
    └── Prod Identity → Prod resources
```

### Преимущества

- ✅ Единая модель identity во всех средах
- ✅ Разрешения закреплены за identity, а не за конкретными ресурсами
- ✅ Легко воспроизводить окружения (Infrastructure as Code)

---

### Почему это важно

- Можно заранее назначить RBAC-права identity.
- При создании новой среды не требуется повторная настройка доступа.
- Уменьшается риск ошибок при миграции dev → prod.

---

### Для AZ-204

Если требуется:
- одинаковая модель безопасности в разных средах
- перенос инфраструктуры без перенастройки RBAC
  → часто подходит **User-Assigned Managed Identity**.

---

# Поддерживаемые сервисы Azure

## Сервисы, поддерживающие Managed Identities

| Категория | Сервисы |
|------------|----------|
| **Compute** | Virtual Machines, VM Scale Sets, App Service, Functions, Container Instances, AKS |
| **Data** | SQL Database, Synapse Analytics, Data Factory, Data Lake Storage |
| **Analytics** | Azure Databricks, Stream Analytics, HDInsight |
| **Integration** | Logic Apps, API Management, Event Grid |
| **Management** | Automation, Azure DevOps |
| **Security** | Key Vault, App Configuration |

---

## Сервисы, поддерживающие аутентификацию Microsoft Entra

Managed Identity может использоваться с любым сервисом, поддерживающим **Microsoft Entra authentication**:

- ✅ Key Vault
- ✅ Azure Storage (Blob, Queue, Table, Files)
- ✅ Azure SQL Database
- ✅ Azure Cosmos DB
- ✅ Azure Service Bus
- ✅ Azure Event Hubs
- ✅ Azure Container Registry
- ✅ Azure Resource Manager
- ✅ Azure Data Lake Storage
- ✅ Azure App Configuration

---

### Что важно запомнить для AZ-204

- Managed Identity работает только с сервисами, поддерживающими Entra authentication.
- Не требует хранения connection string или access keys.
- Частый сценарий вопроса:
  > Нужно безопасно подключиться к Storage или SQL без хранения секретов  
  → использовать Managed Identity.

## How Managed Identities Work

### Authentication Flow

```
1. Application requests access to Azure resource
        ↓
2. Azure Instance Metadata Service (IMDS) provides token
   Endpoint: http://169.254.169.254/metadata/identity/oauth2/token
        ↓
3. Microsoft Entra ID validates managed identity
        ↓
4. Microsoft Entra ID issues JWT access token
        ↓
5. Application includes token in Authorization header
   Authorization: Bearer <access_token>
        ↓
6. Azure resource validates token and grants access
```
### Что происходит «за кулисами» (Behind the Scenes)

#### Создание System-Assigned Managed Identity

1. **Azure Resource Manager (ARM)** получает запрос на включение Managed Identity
2. В **Microsoft Entra ID** создаётся Service Principal
3. Credential’ы автоматически передаются в **Azure Instance Metadata Service (IMDS)**
4. Ресурс получает возможность запрашивать токены через IMDS

---

### Как приложение получает токен

- Приложение обращается к локальному endpoint IMDS
- IMDS запрашивает токен у Microsoft Entra ID
- Возвращается access token для нужного ресурса (например, Key Vault)
- Приложение использует токен в заголовке `Authorization: Bearer <token>`

---

### Почему это безопасно

- Credential’ы не покидают Azure инфраструктуру
- Нет хранения секретов в коде
- Токены имеют ограниченный срок действия
- Azure управляет всей ротацией автоматически

---

### Важно для AZ-204

- IMDS используется только внутри Azure.
- Managed Identity не работает вне Azure.
- Если в вопросе упоминается:
  - Instance Metadata Service
  - локальный endpoint для токена
    → речь идёт о Managed Identity.


**Token Acquisition:**
```bash
# Inside Azure VM/App Service/Function
curl 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net' \
  -H Metadata:true

# Response:
# {
#   "access_token": "eyJ0eXAiOi...",
#   "expires_in": "3599",
#   "expires_on": "1577836800",
#   "resource": "https://vault.azure.net",
#   "token_type": "Bearer"
# }
```

---

## Role Assignments

After enabling managed identity, you must grant it permissions:

### Grant Access with Azure RBAC

```bash
# Get managed identity principal ID
PRINCIPAL_ID=$(az vm identity show \
  --name myvm \
  --resource-group myresourcegroup \
  --query principalId -o tsv)

# Grant Key Vault Secrets User role
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee $PRINCIPAL_ID \
  --scope /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/myvault

# Grant Storage Blob Data Contributor role
az role assignment create \
  --role "Storage Blob Data Contributor" \
  --assignee $PRINCIPAL_ID \
  --scope /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Storage/storageAccounts/mystorageaccount
```

## Часто используемые RBAC-роли для Managed Identity

| Ресурс | Роль | Разрешения |
|---------|------|------------|
| **Key Vault** | Key Vault Secrets User | Чтение секретов |
| **Key Vault** | Key Vault Secrets Officer | Управление секретами |
| **Storage** | Storage Blob Data Reader | Чтение blob |
| **Storage** | Storage Blob Data Contributor | Чтение и запись blob |
| **SQL Database** | SQL DB Contributor | Управление базами данных |
| **Cosmos DB** | Cosmos DB Account Reader | Чтение учётной записи Cosmos DB |

---

# Exam Tips (Советы к AZ-204)

🎯 **Два типа Managed Identity**
- System-assigned (привязана к ресурсу)
- User-assigned (отдельный ресурс)

🎯 **Жизненный цикл System-assigned**  
Удаляется автоматически вместе с ресурсом.

🎯 **Жизненный цикл User-assigned**  
Независимый, требуется ручное удаление.

🎯 **Совместное использование**
- System-assigned → нельзя делить (1:1)
- User-assigned → можно назначать нескольким ресурсам

🎯 **Service Principal**  
Managed Identity — это особый тип Service Principal.

🎯 **Аутентификация**  
Всегда через Microsoft Entra ID.

🎯 **Token endpoint (IMDS)**
http://169.254.169.254/metadata/identity/oauth2/token


🎯 **Best practice**  
Использовать Managed Identity вместо Service Principal с secret.

🎯 **Role assignments**  
После включения Managed Identity необходимо назначить RBAC-роль.

🎯 **DefaultAzureCredential**  
Автоматически использует Managed Identity, если она доступна.

🎯 **Поддерживаемые сервисы**  
VM, App Service, Functions, Container Instances и другие.

---

### Часто проверяют на экзамене

- Разницу между System и User-assigned.
- Нужно ли назначать роль после включения identity (да).
- Как работает IMDS.
- Почему Managed Identity безопаснее client secret.

## Quick Reference

### Enable System-Assigned Identity
```bash
# VM
az vm identity assign --name myvm --resource-group myrg

# App Service
az webapp identity assign --name myapp --resource-group myrg

# Function
az functionapp identity assign --name myfunc --resource-group myrg
```

### Create and Assign User-Assigned Identity
```bash
# Create
az identity create --name myidentity --resource-group myrg

# Assign to VM
az vm identity assign --name myvm --resource-group myrg --identities myidentity

# Assign to App Service
az webapp identity assign --name myapp --resource-group myrg --identities myidentity
```

### Grant Permissions
```bash
# Get principal ID
PRINCIPAL_ID=$(az vm identity show --name myvm --resource-group myrg --query principalId -o tsv)

# Grant role
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee $PRINCIPAL_ID \
  --scope /subscriptions/{sub}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/myvault
```

---

## Additional Resources

- [Managed Identities for Azure Resources](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/)
- [Services Supporting Managed Identities](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/services-support-managed-identities)
- [Azure Identity SDK](https://learn.microsoft.com/en-us/dotnet/api/overview/azure/identity-readme)

[Microsoft Learn - Explore managed identities](https://learn.microsoft.com/en-us/training/modules/implement-managed-identities/2-managed-identities-overview)
