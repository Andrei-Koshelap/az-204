# Защита данных Azure App Configuration

## Обзор

Защита данных в Azure App Configuration строится на нескольких уровнях:

- 🔐 Аутентификация
- 🛡 Авторизация
- 🔒 Шифрование
- 🌐 Сетевая изоляция

В этом разделе рассматриваются механизмы безопасности и лучшие практики для защиты конфигурации приложений.

---

## Основные уровни защиты

### 1. Аутентификация

- Используется **Microsoft Entra ID**
- Рекомендуется **Managed Identity**
- Поддерживается аутентификация через Azure Identity SDK
- REST API требует Bearer-токен

---

### 2. Авторизация

- Используется **Azure RBAC**
- Роли назначаются на уровне ресурса
- Пример роли:
    - App Configuration Data Reader
    - App Configuration Data Owner

Принцип: **Least Privilege** — давать только необходимые права.

---

### 3. Шифрование

- 🔐 Данные шифруются:
    - при хранении (at rest)
    - при передаче (TLS)
- Поддержка **Customer-Managed Keys (CMK)** в Standard tier

---

### 4. Сетевая изоляция

- Поддержка **Private Endpoints**
- Возможность отключения публичного доступа
- Интеграция с VNet
- Использование firewall-правил

---

## Лучшие практики

- Использовать Managed Identity вместо connection string.
- В production включать Private Endpoints.
- Использовать Key Vault для хранения секретов.
- Настроить логирование и аудит.
- Разделять App Configuration по средам (dev/test/prod).

---

## Важно для AZ-204

- Аутентификация → Entra ID.
- Авторизация → Azure RBAC.
- Шифрование → включено по умолчанию.
- Network isolation → Private Endpoint.
- App Configuration не предназначен для хранения секретов.

---

## Security Layers

```
┌───────────────────────────────────────────────────────────┐
│  Network Security                                          │
│  • Private Endpoints                                       │
│  • Firewall Rules                                          │
│  • Virtual Network Integration                             │
└───────────────┬───────────────────────────────────────────┘
                │
                ▼
┌───────────────────────────────────────────────────────────┐
│  Authentication                                            │
│  • Managed Identities (Recommended)                        │
│  • Azure AD Service Principals                             │
│  • Connection Strings (Development Only)                   │
└───────────────┬───────────────────────────────────────────┘
                │
                ▼
┌───────────────────────────────────────────────────────────┐
│  Authorization                                             │
│  • Azure RBAC Roles                                        │
│  • App Configuration Data Owner                            │
│  • App Configuration Data Reader                           │
└───────────────┬───────────────────────────────────────────┘
                │
                ▼
┌───────────────────────────────────────────────────────────┐
│  Encryption                                                │
│  • Microsoft-Managed Keys (Default)                        │
│  • Customer-Managed Keys (CMK)                             │
│  • Data encrypted at rest and in transit                   │
└───────────────────────────────────────────────────────────┘
```

---

## Managed Identities

### Что такое Managed Identity?

**Managed Identity** в Microsoft Entra ID позволяет Azure App Configuration получать доступ к защищённым ресурсам (например, Azure Key Vault) без хранения и управления credential’ами.

---

### Преимущества

- 🔐 Нет секретов в коде или конфигурации
- 🔄 Автоматическая ротация credential’ов
- 🔗 Нативная интеграция с сервисами Azure
- ⚙️ Azure управляет жизненным циклом identity

---

## Типы Managed Identity

| Тип | Жизненный цикл | Совместное использование | Сценарий |
|------|---------------|--------------------------|----------|
| **System-Assigned** | Связан с App Configuration | Нельзя делить | Простой сценарий, 1:1 |
| **User-Assigned** | Независимый ресурс | Можно использовать повторно | Несколько хранилищ, multi-region |

---

# Добавление System-Assigned Identity

## Когда использовать

- Один App Configuration store
- Простая схема аутентификации
- Identity должна удаляться вместе с store
- Нет необходимости делиться identity

---

## Важно для AZ-204

- Managed Identity — рекомендуемый способ аутентификации.
- Если нет требований к совместному использованию identity → выбирать System-Assigned.
- После включения identity нужно назначить RBAC-роль на целевой ресурс (например, Key Vault).

### Setup with Azure CLI

```bash
# Variables
RESOURCE_GROUP="rg-appconfig"
APP_CONFIG_NAME="myappconfig"

# Assign system-assigned identity
az appconfig identity assign \
  --name $APP_CONFIG_NAME \
  --resource-group $RESOURCE_GROUP
```

**Output**:
```json
{
  "principalId": "12345678-1234-1234-1234-123456789012",
  "tenantId": "87654321-4321-4321-4321-210987654321",
  "type": "SystemAssigned"
}
```

### Grant Access to Key Vault

After enabling identity, grant permissions to access Key Vault:

```bash
# Get App Configuration identity Principal ID
PRINCIPAL_ID=$(az appconfig identity show \
  --name $APP_CONFIG_NAME \
  --resource-group $RESOURCE_GROUP \
  --query principalId -o tsv)

# Grant access to Key Vault
KEY_VAULT_NAME="myvault"

az keyvault set-policy \
  --name $KEY_VAULT_NAME \
  --object-id $PRINCIPAL_ID \
  --secret-permissions get list
```

**Alternative: Using Azure RBAC** (Recommended):
```bash
# Get Key Vault resource ID
VAULT_ID=$(az keyvault show --name $KEY_VAULT_NAME --query id -o tsv)

# Assign role
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Key Vault Secrets User" \
  --scope $VAULT_ID
```

### Verify Identity

```bash
# Show identity details
az appconfig identity show \
  --name $APP_CONFIG_NAME \
  --resource-group $RESOURCE_GROUP
```

---

## Добавление User-Assigned Identity

### Когда использовать

- Несколько App Configuration store
- Необходимо использовать одну identity для нескольких store
- Сценарии с несколькими регионами (cross-region)
- Требуется независимое управление жизненным циклом identity

---

## Почему выбирать User-Assigned

- 🔁 Можно назначать нескольким ресурсам
- 🔐 Централизованное управление правами (RBAC)
- ♻️ Identity сохраняется при удалении store
- 🧩 Удобно при Infrastructure as Code

---

## Важно для AZ-204

- Если требуется shared identity → выбирать User-Assigned.
- Если требуется независимый lifecycle → User-Assigned.
- После назначения identity необходимо:
    - выдать роль (например, доступ к Key Vault),
    - указать Client ID при использовании нескольких identity.


### Setup with Azure CLI

#### Step 1: Create User-Assigned Identity

```bash
# Variables
IDENTITY_NAME="id-appconfig-reader"
RESOURCE_GROUP="rg-identities"
LOCATION="eastus"

# Create identity
az identity create \
  --resource-group $RESOURCE_GROUP \
  --name $IDENTITY_NAME \
  --location $LOCATION
```

**Output**:
```json
{
  "clientId": "abcdef12-ab12-ab12-ab12-abcdef123456",
  "id": "/subscriptions/{sub-id}/resourcegroups/rg-identities/providers/Microsoft.ManagedIdentity/userAssignedIdentities/id-appconfig-reader",
  "location": "eastus",
  "name": "id-appconfig-reader",
  "principalId": "98765432-9876-9876-9876-987654321098",
  "resourceGroup": "rg-identities",
  "type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

#### Step 2: Get Identity Resource ID

```bash
IDENTITY_ID=$(az identity show \
  --resource-group $RESOURCE_GROUP \
  --name $IDENTITY_NAME \
  --query id -o tsv)

echo $IDENTITY_ID
# Output: /subscriptions/{sub-id}/resourcegroups/rg-identities/providers/Microsoft.ManagedIdentity/userAssignedIdentities/id-appconfig-reader
```

#### Step 3: Assign Identity to App Configuration

```bash
# Assign to App Configuration store
az appconfig identity assign \
  --name myappconfig \
  --resource-group rg-appconfig \
  --identities $IDENTITY_ID
```

#### Step 4: Grant Permissions

```bash
# Get principal ID
PRINCIPAL_ID=$(az identity show \
  --resource-group $RESOURCE_GROUP \
  --name $IDENTITY_NAME \
  --query principalId -o tsv)

# Grant Key Vault access
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "Key Vault Secrets User" \
  --scope "/subscriptions/{sub-id}/resourceGroups/rg-keyvault/providers/Microsoft.KeyVault/vaults/myvault"
```

### Assign to Multiple Stores

```bash
# Assign same identity to multiple App Configuration stores
az appconfig identity assign --name appconfig-dev --identities $IDENTITY_ID
az appconfig identity assign --name appconfig-staging --identities $IDENTITY_ID
az appconfig identity assign --name appconfig-prod --identities $IDENTITY_ID
```

---

## Customer-Managed Encryption Keys (CMK)

### Шифрование по умолчанию

По умолчанию Azure App Configuration шифрует все данные при хранении с использованием **ключей, управляемых Microsoft**:

- 🔐 AES-256
- Отдельный ключ для каждого store
- Полностью управляется Microsoft
- Не требует дополнительной настройки

---

## Customer-Managed Keys (CMK)

Для дополнительного контроля можно использовать **ключи, управляемые клиентом**, хранящиеся в Azure Key Vault.

---

### Преимущества CMK

- 🔑 Полный контроль над ключами шифрования
- 🔄 Управление ротацией ключей
- 📋 Соответствие требованиям compliance
- 🚫 Возможность отзыва доступа к данным

---

### Требования

- ✅ App Configuration уровня **Standard**
- ✅ Azure Key Vault с включённым **soft-delete**
- ✅ Azure Key Vault с включённой **purge-protection**
- ✅ Ключ типа **RSA или RSA-HSM**
- ✅ Ключ должен быть включён
- ✅ Ключ должен поддерживать операции **wrap** и **unwrap**

---

## Как работает CMK

1. App Configuration получает Managed Identity
2. Managed Identity получает разрешения `GET`, `WRAP`, `UNWRAP` в Key Vault
3. App Configuration вызывает Key Vault для обёртывания своего ключа шифрования
4. Обёрнутый ключ сохраняется
5. Развёрнутый (unwrapped) ключ кэшируется на 1 час
6. Ключ обновляется автоматически каждый час

---

## Важно для AZ-204

- CMK доступен только в Standard tier.
- Требуется Managed Identity + разрешения в Key Vault.
- Soft Delete и Purge Protection обязательны.
- Если нужно соответствие регуляторным требованиям → использовать CMK.
- По умолчанию используется Microsoft-managed encryption (достаточно для большинства сценариев).

```
┌──────────────────────────────────────────────────────┐
│  Azure App Configuration                              │
│  ┌──────────────────────────────────────────┐        │
│  │  Configuration Data                      │        │
│  │  (encrypted with Data Encryption Key)    │        │
│  └──────────────────────────────────────────┘        │
│  ┌──────────────────────────────────────────┐        │
│  │  Data Encryption Key (DEK)               │        │
│  │  (wrapped by Key Encryption Key)         │◄───────┼──┐
│  └──────────────────────────────────────────┘        │  │
│  ┌──────────────────────────────────────────┐        │  │
│  │  Managed Identity                        │────────┼──┤
│  │  (authenticates to Key Vault)            │        │  │
│  └──────────────────────────────────────────┘        │  │
└──────────────────────────────────────────────────────┘  │
                                                           │
                     ┌─────────────────────────────────────┘
                     │
                     ▼
         ┌───────────────────────────────┐
         │  Azure Key Vault              │
         │  ┌─────────────────────────┐  │
         │  │  Key Encryption Key     │  │
         │  │  (RSA or RSA-HSM)       │  │
         │  │  • Wrap capability      │  │
         │  │  • Unwrap capability    │  │
         │  └─────────────────────────┘  │
         └───────────────────────────────┘
```

### Enable CMK

#### Step 1: Create Key Vault with Required Features

```bash
# Variables
KEY_VAULT_NAME="kv-appconfig-cmk"
RESOURCE_GROUP="rg-appconfig"
LOCATION="eastus"

# Create Key Vault with soft-delete and purge-protection
az keyvault create \
  --name $KEY_VAULT_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --enable-soft-delete true \
  --enable-purge-protection true
```

#### Step 2: Create RSA Key

```bash
# Create RSA key with wrap and unwrap capabilities
az keyvault key create \
  --vault-name $KEY_VAULT_NAME \
  --name appconfig-encryption-key \
  --kty RSA \
  --size 2048 \
  --ops wrapKey unwrapKey
```

#### Step 3: Enable Managed Identity on App Configuration

```bash
APP_CONFIG_NAME="myappconfig"

# Assign system-assigned identity
az appconfig identity assign \
  --name $APP_CONFIG_NAME \
  --resource-group $RESOURCE_GROUP

# Get principal ID
PRINCIPAL_ID=$(az appconfig identity show \
  --name $APP_CONFIG_NAME \
  --resource-group $RESOURCE_GROUP \
  --query principalId -o tsv)
```

#### Step 4: Grant Key Vault Permissions

```bash
# Grant GET, WRAP, and UNWRAP permissions
az keyvault set-policy \
  --name $KEY_VAULT_NAME \
  --object-id $PRINCIPAL_ID \
  --key-permissions get wrapKey unwrapKey
```

#### Step 5: Configure App Configuration to Use CMK

```bash
# Get Key Vault key identifier
KEY_IDENTIFIER=$(az keyvault key show \
  --vault-name $KEY_VAULT_NAME \
  --name appconfig-encryption-key \
  --query key.kid -o tsv)

# Enable CMK on App Configuration
az appconfig update \
  --name $APP_CONFIG_NAME \
  --resource-group $RESOURCE_GROUP \
  --encryption-key-name appconfig-encryption-key \
  --encryption-key-vault-uri https://${KEY_VAULT_NAME}.vault.azure.net \
  --identity-client-id $(az appconfig identity show \
    --name $APP_CONFIG_NAME \
    --resource-group $RESOURCE_GROUP \
    --query principalId -o tsv)
```

### Key Rotation

**Automatic rotation** when key version changes:
```bash
# Create new key version
az keyvault key create \
  --vault-name $KEY_VAULT_NAME \
  --name appconfig-encryption-key \
  --kty RSA \
  --size 2048

# App Configuration automatically detects and uses new version within 1 hour
```

---

## Private Endpoints

### Что такое Private Endpoint?

**Private Endpoint** позволяет клиентам внутри виртуальной сети (VNet) безопасно подключаться к Azure App Configuration через **Private Link**, используя приватный IP-адрес из этой сети.

То есть трафик проходит по внутренней сети Azure, а не через публичный интернет.

---

### Преимущества

- 🔒 Отсутствие доступа через публичный интернет
- 🌐 Безопасный доступ из VNet или из on-premises (через VPN/ExpressRoute)
- 🚫 Снижение риска утечки данных (data exfiltration)
- 📋 Соответствие требованиям сетевой безопасности и compliance

---

### Как это работает

- App Configuration получает Private Endpoint
- Ему назначается приватный IP в VNet
- Публичный доступ можно отключить
- DNS настраивается для разрешения имени сервиса в приватный IP

---

### Важно для AZ-204

- Private Endpoint используется для сетевой изоляции.
- Частый сценарий:
  > Нужно запретить доступ из интернета  
  → использовать Private Endpoint.
- Для production часто требуется:
    - Private Endpoint
    - отключённый публичный доступ.


### Architecture

```
┌─────────────────────────────────────────────────────────┐
│  Virtual Network (10.0.0.0/16)                          │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Subnet: app-subnet (10.0.1.0/24)                 │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────┐        │  │
│  │  │ Web App  │  │ Function │  │   VM     │        │  │
│  │  │          │  │   App    │  │          │        │  │
│  │  └────┬─────┘  └────┬─────┘  └────┬─────┘        │  │
│  │       │             │              │              │  │
│  │       └─────────────┴──────────────┘              │  │
│  │                     │                             │  │
│  │                     ▼                             │  │
│  └───────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────┐  │
│  │  Subnet: pe-subnet (10.0.2.0/24)                  │  │
│  │  ┌──────────────────────────────────────┐         │  │
│  │  │  Private Endpoint                    │         │  │
│  │  │  Private IP: 10.0.2.4                │         │  │
│  │  │  DNS: myappconfig.privatelink...     │         │  │
│  │  └────────────────┬─────────────────────┘         │  │
│  │                   │                               │  │
│  └───────────────────┼───────────────────────────────┘  │
└────────────────────┼─────────────────────────────────────┘
                     │ Private Link
                     ▼
   ┌──────────────────────────────────────┐
   │  Azure App Configuration             │
   │  • Public endpoint: DISABLED         │
   │  • Private endpoint: ENABLED         │
   └──────────────────────────────────────┘
```
### Создание Private Endpoint

#### Предварительные требования

- 🌐 Виртуальная сеть (VNet) с выделенной подсетью
- 📦 Экземпляр App Configuration уровня **Standard**

---

### Почему это важно

- Private Endpoint требует отдельной подсети.
- Поддержка Private Endpoint доступна только в Standard tier.
- Free tier не поддерживает сетевую изоляцию.

---

### Что обычно настраивается дополнительно

- Приватная DNS-зона для корректного разрешения имени сервиса
- Отключение публичного доступа (опционально, но рекомендуется)
- Проверка доступа через VNet или VPN/ExpressRoute

---

### Важно для AZ-204

Если в вопросе есть требования:

- запретить публичный доступ
- обеспечить доступ только из VNet
- соответствие строгим сетевым политикам

→ использовать **Private Endpoint + Standard tier**.


#### Step 1: Create Subnet for Private Endpoint

```bash
# Variables
VNET_NAME="vnet-prod"
SUBNET_NAME="subnet-pe-appconfig"
RESOURCE_GROUP="rg-network"

# Create subnet with private endpoint network policies disabled
az network vnet subnet create \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --name $SUBNET_NAME \
  --address-prefixes 10.0.2.0/24 \
  --disable-private-endpoint-network-policies true
```

#### Step 2: Create Private Endpoint

```bash
# Variables
PE_NAME="pe-appconfig"
APP_CONFIG_NAME="myappconfig"
APP_CONFIG_RG="rg-appconfig"

# Get App Configuration resource ID
APP_CONFIG_ID=$(az appconfig show \
  --name $APP_CONFIG_NAME \
  --resource-group $APP_CONFIG_RG \
  --query id -o tsv)

# Create private endpoint
az network private-endpoint create \
  --name $PE_NAME \
  --resource-group $RESOURCE_GROUP \
  --vnet-name $VNET_NAME \
  --subnet $SUBNET_NAME \
  --private-connection-resource-id $APP_CONFIG_ID \
  --group-id configurationStores \
  --connection-name appconfig-connection
```

#### Step 3: Configure Private DNS Zone

```bash
# Create private DNS zone
az network private-dns zone create \
  --resource-group $RESOURCE_GROUP \
  --name privatelink.azconfig.io

# Link DNS zone to VNet
az network private-dns link vnet create \
  --resource-group $RESOURCE_GROUP \
  --zone-name privatelink.azconfig.io \
  --name appconfig-dns-link \
  --virtual-network $VNET_NAME \
  --registration-enabled false

# Create DNS zone group (automatic DNS records)
az network private-endpoint dns-zone-group create \
  --resource-group $RESOURCE_GROUP \
  --endpoint-name $PE_NAME \
  --name appconfig-zone-group \
  --private-dns-zone privatelink.azconfig.io \
  --zone-name privatelink.azconfig.io
```

#### Step 4: Disable Public Access (Optional)

```bash
# Disable public network access
az appconfig update \
  --name $APP_CONFIG_NAME \
  --resource-group $APP_CONFIG_RG \
  --enable-public-network false
```

### Verify Private Endpoint

```bash
# From a VM in the VNet
nslookup myappconfig.azconfig.io

# Should resolve to private IP (10.0.2.4)
# Instead of public IP
```

---

## Public Access Firewall Rules

If private endpoints are not used, configure firewall rules to restrict public access.

### Disable Public Access Completely

```bash
az appconfig update \
  --name myappconfig \
  --resource-group rg-appconfig \
  --enable-public-network false
```

### Allow Specific IP Ranges

```bash
# Enable public access (required before adding rules)
az appconfig update \
  --name myappconfig \
  --resource-group rg-appconfig \
  --enable-public-network true

# Add IP rule (single IP)
az appconfig network-rule add \
  --name myappconfig \
  --resource-group rg-appconfig \
  --ip-address 203.0.113.25

# Add IP rule (CIDR range)
az appconfig network-rule add \
  --name myappconfig \
  --resource-group rg-appconfig \
  --ip-address 203.0.113.0/24

# Allow Azure services
az appconfig update \
  --name myappconfig \
  --resource-group rg-appconfig \
  --bypass AzureServices
```

### List Network Rules

```bash
az appconfig network-rule list \
  --name myappconfig \
  --resource-group rg-appconfig
```

### Remove Network Rule

```bash
az appconfig network-rule remove \
  --name myappconfig \
  --resource-group rg-appconfig \
  --ip-address 203.0.113.25
```

---

## Azure RBAC роли

Azure App Configuration использует **Azure RBAC** для управления доступом.

---

## Встроенные роли

| Роль | Разрешения | Сценарий использования |
|------|------------|------------------------|
| **App Configuration Data Owner** | Чтение, изменение, удаление конфигурации | Администраторы, CI/CD пайплайны |
| **App Configuration Data Reader** | Только чтение конфигурации | Приложения и сервисы (рекомендуется) |
| **App Configuration (Contributor)** | Управление ресурсом App Configuration | Управление инфраструктурой |

---

## Что важно понимать

- Data Reader / Data Owner — это **data plane** доступ (доступ к данным).
- Contributor — это **management plane** (управление самим ресурсом).
- Для приложений в production рекомендуется назначать:
    - **App Configuration Data Reader**
    - через Managed Identity.

---

## Важно для AZ-204

- RBAC — рекомендуемая модель авторизации.
- Приложениям не нужен Contributor.
- Использовать принцип **Least Privilege**.
- Частый экзаменационный сценарий:
  > Приложение должно читать конфигурацию  
  → назначить роль **App Configuration Data Reader**.

### Grant Access to Application

```bash
# Variables
APP_CONFIG_NAME="myappconfig"
RESOURCE_GROUP="rg-appconfig"
APP_NAME="mywebapp"

# Get application's managed identity principal ID
PRINCIPAL_ID=$(az webapp identity show \
  --name $APP_NAME \
  --resource-group $RESOURCE_GROUP \
  --query principalId -o tsv)

# Get App Configuration resource ID
APP_CONFIG_ID=$(az appconfig show \
  --name $APP_CONFIG_NAME \
  --resource-group $RESOURCE_GROUP \
  --query id -o tsv)

# Grant Data Reader role
az role assignment create \
  --assignee $PRINCIPAL_ID \
  --role "App Configuration Data Reader" \
  --scope $APP_CONFIG_ID
```

### Grant Access to User

```bash
# Grant to specific user
az role assignment create \
  --assignee user@contoso.com \
  --role "App Configuration Data Owner" \
  --scope $APP_CONFIG_ID
```

### Grant Access to Service Principal

```bash
# Get service principal app ID
SP_APP_ID="12345678-1234-1234-1234-123456789012"

# Grant access
az role assignment create \
  --assignee $SP_APP_ID \
  --role "App Configuration Data Reader" \
  --scope $APP_CONFIG_ID
```

---

## Authenticate Applications

### Option 1: Managed Identity (Recommended)

**.NET Example**:
```csharp
using Azure.Identity;
using Microsoft.Extensions.Configuration.AzureAppConfiguration;

var builder = WebApplication.CreateBuilder(args);

// Use managed identity
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(
        new Uri("https://myappconfig.azconfig.io"),
        new DefaultAzureCredential());
});

var app = builder.Build();
app.Run();
```

**Requirements**:
1. Enable managed identity on application (Web App, VM, Function)
2. Grant "App Configuration Data Reader" role to identity

### Option 2: Connection String (Development Only)

```bash
# Get connection string
CONNECTION_STRING=$(az appconfig credential list \
  --name myappconfig \
  --resource-group rg-appconfig \
  --query "[?name=='Primary'].connectionString" -o tsv)

# Set as environment variable
export APP_CONFIG_CONNECTION_STRING="$CONNECTION_STRING"
```

**.NET Example**:
```csharp
builder.Configuration.AddAzureAppConfiguration(
    Environment.GetEnvironmentVariable("APP_CONFIG_CONNECTION_STRING"));
```

⚠️ **Warning**: Connection strings contain secrets. Use only for local development.

### Option 3: Service Principal (CI/CD)

```bash
# Create service principal
SP=$(az ad sp create-for-rbac --name "AppConfigReader" --json)
SP_APP_ID=$(echo $SP | jq -r .appId)
SP_PASSWORD=$(echo $SP | jq -r .password)
SP_TENANT=$(echo $SP | jq -r .tenant)

# Grant access
az role assignment create \
  --assignee $SP_APP_ID \
  --role "App Configuration Data Reader" \
  --scope $APP_CONFIG_ID

# Use in CI/CD
export AZURE_CLIENT_ID=$SP_APP_ID
export AZURE_CLIENT_SECRET=$SP_PASSWORD
export AZURE_TENANT_ID=$SP_TENANT
```

**.NET Example**:
```csharp
var credential = new ClientSecretCredential(
    Environment.GetEnvironmentVariable("AZURE_TENANT_ID"),
    Environment.GetEnvironmentVariable("AZURE_CLIENT_ID"),
    Environment.GetEnvironmentVariable("AZURE_CLIENT_SECRET"));

builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(new Uri("https://myappconfig.azconfig.io"), credential);
});
```

---

## Best Practices

### 1. **Use Managed Identities**

✅ **Recommended**:
```csharp
options.Connect(new Uri("https://myappconfig.azconfig.io"), 
               new DefaultAzureCredential());
```

❌ **Avoid**:
```csharp
options.Connect(connectionString); // Contains secrets
```

### 2. **Apply Least Privilege**

- Applications: Grant **Data Reader** (read-only)
- CI/CD pipelines: Grant **Data Owner** (read-write)
- Administrators: Use Azure AD groups

### 3. **Use Private Endpoints for Production**

```bash
# Disable public access
az appconfig update --name myappconfig --enable-public-network false

# Use private endpoint from VNet
```

### 4. **Enable Customer-Managed Keys for Compliance**

Required for:
- Healthcare (HIPAA)
- Financial services (PCI-DSS)
- Government (FedRAMP)

### 5. **Separate Stores per Environment**

```
appconfig-dev    (Standard tier, public access)
appconfig-staging (Standard tier, private endpoint)
appconfig-prod   (Standard tier, private endpoint, CMK)
```

### 6. **Audit Access**

```bash
# Enable diagnostic logs
az monitor diagnostic-settings create \
  --name appconfig-logs \
  --resource $APP_CONFIG_ID \
  --workspace $LOG_ANALYTICS_WORKSPACE_ID \
  --logs '[{"category":"HttpRequest","enabled":true}]' \
  --metrics '[{"category":"AllMetrics","enabled":true}]'
```

---
# Exam Tips — AZ-204 (Security в App Configuration)

## Ключевые концепции

1. **Managed Identity**  
   Устраняет необходимость хранить credential’ы в коде.

2. **System-Assigned Identity**  
   Связана с конкретным App Configuration store  
   Удаляется вместе с ресурсом.

3. **User-Assigned Identity**  
   Независимый ресурс  
   Может использоваться несколькими store.

4. **Требования для CMK**
    - Standard tier
    - Soft-delete включён
    - Purge-protection включён
    - RSA или RSA-HSM ключ

5. **Private Endpoint**  
   Безопасный доступ через Private Link из VNet.

6. **RBAC роли**
    - Data Owner — чтение/запись
    - Data Reader — только чтение

7. **Connection strings**  
   Использовать только для разработки  
   Содержат секреты.

8. **DefaultAzureCredential**  
   Сначала пробует Managed Identity, затем другие методы.

9. **Firewall для публичного доступа**  
   Можно ограничить доступ конкретными IP-диапазонами.

10. **Обновление CMK**  
    Расшифрованный ключ кэшируется на 1 час  
    Обновляется автоматически каждый час.

11. **Private DNS зона**  
    `privatelink.azconfig.io`

12. **Права для CMK в Key Vault**  
    Требуются разрешения:  
    `GET`, `WRAP`, `UNWRAP`

---

# Частые экзаменационные сценарии

### Сценарий 1
> Web App должен читать App Configuration

→ Включить Managed Identity  
→ Назначить роль **App Configuration Data Reader**

---

### Сценарий 2
> Нужно защитить App Configuration от публичного интернета

→ Создать Private Endpoint  
→ Отключить публичный доступ

---

### Сценарий 3
> Требуется соответствие регуляции с контролем ключей шифрования

→ Использовать Customer-Managed Keys в Key Vault

---

### Сценарий 4
> CI/CD pipeline должен обновлять конфигурацию

→ Использовать Service Principal  
→ Назначить роль **App Configuration Data Owner**

---

### Сценарий 5
> Нужно использовать одну identity для нескольких store

→ Использовать User-Assigned Managed Identity

---

## Главное для запоминания

- Managed Identity — рекомендуемый способ аутентификации.
- RBAC — модель авторизации.
- CMK нужен для compliance.
- Private Endpoint — для сетевой изоляции.
- Connection strings — не для production.

## Quick Reference Commands

```bash
# Enable system-assigned identity
az appconfig identity assign --name <name> --resource-group <rg>

# Create user-assigned identity
az identity create --name <name> --resource-group <rg>

# Assign user-assigned identity
az appconfig identity assign --name <name> --identities <identity-id>

# Grant RBAC role
az role assignment create \
  --assignee <principal-id> \
  --role "App Configuration Data Reader" \
  --scope <appconfig-resource-id>

# Create private endpoint
az network private-endpoint create \
  --name <pe-name> \
  --vnet-name <vnet> \
  --subnet <subnet> \
  --private-connection-resource-id <appconfig-id> \
  --group-id configurationStores

# Disable public access
az appconfig update --name <name> --enable-public-network false

# Add firewall rule
az appconfig network-rule add --name <name> --ip-address <ip-or-cidr>

# Get connection string
az appconfig credential list --name <name> --resource-group <rg>
```

---

## Learn More

- [Managed Identities with App Configuration](https://docs.microsoft.com/azure/azure-app-configuration/howto-integrate-azure-managed-service-identity)
- [Use customer-managed keys](https://docs.microsoft.com/azure/azure-app-configuration/concept-customer-managed-keys)
- [Use private endpoints](https://docs.microsoft.com/azure/azure-app-configuration/concept-private-endpoint)
