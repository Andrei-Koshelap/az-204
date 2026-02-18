# Обзор Azure App Configuration

## Что такое Azure App Configuration?

**Azure App Configuration** — это полностью управляемый сервис для централизованного хранения настроек приложения и feature flags.

Он решает проблему управления конфигурацией в распределённых облачных приложениях.

---

## Какую проблему он решает?

Современные cloud-приложения часто состоят из:

- микросервисов
- serverless-функций
- контейнеров
- нескольких окружений (dev / test / prod)

Без централизованного подхода возникают проблемы:

- ❌ **Configuration sprawl** — настройки разбросаны по разным файлам и сервисам
- ❌ **Сложность диагностики** — разные конфигурации в разных средах
- ❌ **Связка с деплоем** — для изменения настройки нужно пересобирать приложение
- ❌ **Риски безопасности** — секреты хранятся в коде или config-файлах

---

## Решение

Azure App Configuration предоставляет:

- ✅ Единый источник истины для настроек
- ✅ Динамическое обновление конфигурации без redeploy
- ✅ Управление feature flags
- ✅ Интеграцию с Azure Key Vault для хранения секретов

---

# Основные преимущества

## 1. Полностью управляемый сервис

- Разворачивается за минуты через Portal или CLI
- Нет инфраструктуры для поддержки
- Встроенная высокая доступность
- Обновления и патчи управляются Microsoft

---

## 2. Гибкая структура ключей

- Иерархические ключи:  
  `AppName:Service1:ApiEndpoint`
- Варианты по label (один ключ — разные значения для окружений)
- Поддержка выборки по шаблону
- Поддержка Unicode

---

## 3. Использование Labels

Labels позволяют:

- Разделять окружения (dev, staging, production)
- Управлять версиями (v1.0, v2.0)
- Конфигурации для feature branches
- A/B тестирование

---

## 4. Point-in-Time Replay

- Получение конфигурации на определённый момент времени
- Аудит изменений
- Откат к предыдущей версии
- Поддержка troubleshooting и compliance

---

## 5. Управление Feature Flags

- Отдельный UI для управления фичами
- Процентный rollout
- Таргетинг по группам пользователей или регионам
- Включение/отключение фич в реальном времени

---

## 6. Повышенная безопасность

- Интеграция с Managed Identity
- Azure RBAC для контроля доступа
- Private Endpoints
- Шифрование данных в транзите и при хранении
- Поддержка Customer-Managed Keys (CMK)

---

## 7. Интеграция с фреймворками

- .NET Configuration Provider
- Spring Cloud (Java)
- JavaScript / Node.js
- Python
- REST API для кастомных решений

---

# Важно для AZ-204

- App Configuration ≠ Key Vault.
    - App Configuration — для настроек и feature flags.
    - Key Vault — для секретов.
- Лучшая практика:
    - Настройки → App Configuration
    - Секреты → Key Vault (через reference)
- Частый сценарий вопроса:
  > Нужно менять конфигурацию без redeploy  
  → использовать Azure App Configuration.

---

## Частые сценарии использования (Common Use Cases)
### Централизованное управление конфигурацией

**Сценарий:**  
Микросервисная архитектура с 20+ сервисами в нескольких регионах.

```yaml
# Hierarchical organization
MyApp:Database:ConnectionString
MyApp:Cache:RedisEndpoint
MyApp:Logging:Level
MyApp:Feature:EnableNewUI

# Environment-specific with labels
Key: MyApp:Database:ConnectionString, Label: Development
Key: MyApp:Database:ConnectionString, Label: Production
```
**Преимущества:**

- ✅ Одно изменение применяется ко всем сервисам
- ✅ Консистентная конфигурация между инстансами
- ✅ Упрощённое продвижение между средами (dev → test → prod)
- ✅ Уменьшение ошибок из-за «разъехавшихся» настроек

---

### Динамическое обновление конфигурации

**Сценарий:**  
Необходимо изменить уровень логирования без перезапуска приложения.

---

### Традиционный подход

1. Изменить конфигурационный файл
2. Собрать новый контейнерный образ
3. Задеплоить в production
4. Дождаться перезапуска pod’ов
5. ⛔ **Простой**: 5–10 минут

---

### С использованием Azure App Configuration

1. Изменить значение в App Configuration
2. Приложение обнаруживает изменение (polling или push)
3. Новая конфигурация применяется динамически
4. ✅ **Простой**: 0 минут

---

## Почему это важно

- Можно изменять:
    - уровень логирования
    - endpoint’ы
    - feature flags
    - параметры кэширования
- Без redeploy
- Без перезапуска контейнеров
- Без downtime

---

## Важно для AZ-204

Если в вопросе говорится:

- нужно изменить настройку без перезапуска
- требуется zero-downtime
- микросервисная архитектура
- централизованная конфигурация

→ правильный ответ: **Azure App Configuration**.


```csharp
// .NET Core example with automatic refresh
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .ConfigureRefresh(refresh =>
           {
               refresh.Register("MyApp:Logging:Level")
                      .SetCacheExpiration(TimeSpan.FromSeconds(30));
           });
});
```

### Feature Flag Management

**Scenario**: Gradual rollout of new checkout process

**Implementation**:
```json
{
  "FeatureManagement": {
    "NewCheckout": {
      "EnabledFor": [
        {
          "Name": "Percentage",
          "Parameters": { "Value": 10 }
        }
      ]
    }
  }
}
```

**Rollout Plan**:
- Day 1: Enable for 10% of users
- Day 3: Increase to 50% if metrics are good
- Day 7: Enable for 100%
- No code deployment needed

### Multi-Environment Configuration

**Scenario**: Consistent configuration structure across environments with environment-specific values

```bash
# Development
Key: MyApp:ApiEndpoint, Label: dev
Value: https://api-dev.contoso.com

# Staging
Key: MyApp:ApiEndpoint, Label: staging
Value: https://api-staging.contoso.com

# Production
Key: MyApp:ApiEndpoint, Label: prod
Value: https://api.contoso.com
```

**Application Code**:
```csharp
// Select environment automatically
string environment = builder.Environment.EnvironmentName;
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .Select(KeyFilter.Any, environment);
});
```

---

## App Configuration vs. Key Vault

| Аспект | App Configuration | Key Vault |
|----------|------------------|-----------|
| **Основное назначение** | Настройки приложения и feature flags | Секреты, ключи и сертификаты |
| **Чувствительность данных** | Нечувствительная конфигурация | Чувствительные данные (пароли, ключи) |
| **Лимит размера** | 10 KB на key-value | 25 KB для секретов |
| **Паттерн доступа** | Частые чтения, массовая выборка | Редкие обращения, точечный доступ |
| **Feature Flags** | ✅ Встроенная поддержка | ❌ Не предназначен для этого |
| **Динамическое обновление** | ✅ Поддержка из коробки | ⚠️ Требует ручного polling |
| **Лучше всего подходит для** | Endpoint’ов, строк подключения (без секретов), настроек | Паролей, сертификатов, API-ключей |

---

## Главное различие

- **App Configuration** → управление конфигурацией приложения.
- **Key Vault** → безопасное хранение секретов.

---

## Лучшая практика

Использовать их вместе:

- Настройки → **App Configuration**
- Секреты → **Key Vault**
- В App Configuration хранить **reference** на секрет в Key Vault

---

## Важно для AZ-204

- Если вопрос про feature flags → App Configuration.
- Если вопрос про хранение пароля → Key Vault.
- Если требуется динамическое обновление без redeploy → App Configuration.
- Если требуется высокая защита чувствительных данных → Key Vault.

### Integration Pattern (Recommended)

**Use both services together**:

```csharp
// Store non-sensitive config in App Configuration
MyApp:Database:Server=sql.database.windows.net
MyApp:Database:Name=productiondb

// Store sensitive data in Key Vault
MyApp:Database:Password --> Key Vault Reference

// App Configuration stores reference
{
  "uri": "https://myvault.vault.azure.net/secrets/DbPassword"
}
```

**Benefits**:
- Centralized configuration in App Configuration
- Secrets secured in Key Vault
- Unified access pattern for developers

---

## Клиентские библиотеки (Client Libraries)

Azure App Configuration предоставляет нативные библиотеки для популярных платформ:

| Язык / Фреймворк | Пакет | Документация |
|------------------|---------|---------------|
| **.NET Core** | `Microsoft.Extensions.Configuration.AzureAppConfiguration` | https://docs.microsoft.com/azure/azure-app-configuration/quickstart-dotnet-core-app |
| **ASP.NET Core** | `Microsoft.Azure.AppConfiguration.AspNetCore` | https://docs.microsoft.com/azure/azure-app-configuration/quickstart-aspnet-core-app |
| **.NET Framework** | `Microsoft.Configuration.ConfigurationBuilders.AzureAppConfiguration` | https://docs.microsoft.com/azure/azure-app-configuration/quickstart-dotnet-app |
| **Java Spring** | `spring-cloud-azure-appconfiguration-config` | https://docs.microsoft.com/azure/azure-app-configuration/quickstart-java-spring-app |
| **JavaScript / Node.js** | `@azure/app-configuration` | https://docs.microsoft.com/azure/azure-app-configuration/quickstart-javascript |
| **Python** | `azure-appconfiguration` | https://docs.microsoft.com/azure/azure-app-configuration/quickstart-python |
| **REST API** | Прямой HTTPS-доступ | https://docs.microsoft.com/rest/api/appconfiguration/ |

---

## Что важно понимать

- Для .NET есть глубокая интеграция с `IConfiguration`.
- Поддерживается динамическое обновление конфигурации.
- Можно подключать Azure App Configuration как отдельный provider.
- Работает совместно с Managed Identity через Azure Identity.

---

## Для AZ-204

- Для .NET чаще всего используется пакет:  
  `Microsoft.Extensions.Configuration.AzureAppConfiguration`
- Если требуется динамическое обновление конфигурации в ASP.NET Core → использовать соответствующий provider.
- REST API используется для кастомных или нестандартных интеграций.
- Managed Identity + App Configuration — рекомендуемая связка.


---
## Тарифные планы (Pricing Tiers)

### Free Tier

- **Стоимость**: Бесплатно
- **Запросы**: 1 000 в день
- **Хранилище**: 10 MB
- **Лучше всего подходит для**: разработки, тестирования, небольших приложений

---

### Standard Tier

- **Стоимость**: $1.20 в день + $0.06 за 10 000 запросов
- **Запросы**: Без ограничений
- **Хранилище**: 1 GB включено (дополнительное — оплачивается отдельно)

**Дополнительные возможности:**

- Soft delete (7–90 дней хранения)
- Customer-Managed Keys (CMK)
- Private endpoints
- SLA 99.9%

**Лучше всего подходит для:** production-приложений

---

## Что важно для AZ-204

- Free — для dev/test.
- Standard — для production.
- Private endpoints и CMK доступны только в Standard.
- Если в вопросе есть требования:
    - SLA
    - network isolation
    - customer-managed encryption  
      → выбирать Standard tier.


**Example Cost Calculation** (Standard Tier):
```
Base cost: $1.20/day × 30 days = $36/month
Requests: 10 million/month = 1,000 × 10,000 requests
Request cost: 1,000 × $0.06 = $60/month
Total: $96/month
```

---

## Quick Start Example

### Create App Configuration Store

```bash
# Variables
RESOURCE_GROUP="rg-appconfig"
LOCATION="eastus"
CONFIG_STORE_NAME="appconfig-myapp-prod"

# Create resource group
az group create --name $RESOURCE_GROUP --location $LOCATION

# Create App Configuration store (Standard tier)
az appconfig create \
  --name $CONFIG_STORE_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --sku Standard
```

### Add Configuration Values

```bash
# Add key-values
az appconfig kv set \
  --name $CONFIG_STORE_NAME \
  --key "MyApp:Database:Server" \
  --value "sql.database.windows.net"

# Add with label for environment
az appconfig kv set \
  --name $CONFIG_STORE_NAME \
  --key "MyApp:ApiEndpoint" \
  --value "https://api-dev.contoso.com" \
  --label "Development"

az appconfig kv set \
  --name $CONFIG_STORE_NAME \
  --key "MyApp:ApiEndpoint" \
  --value "https://api.contoso.com" \
  --label "Production"
```

### .NET Application Integration

```csharp
using Microsoft.Extensions.Configuration;
using Azure.Identity;

var builder = WebApplication.CreateBuilder(args);

// Get connection string from environment variable
string connectionString = Environment.GetEnvironmentVariable("APP_CONFIG_CONNECTION_STRING");

// Add Azure App Configuration
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .Select(KeyFilter.Any, builder.Environment.EnvironmentName)
           .ConfigureRefresh(refresh =>
           {
               refresh.Register("MyApp:Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromMinutes(5));
           });
});

var app = builder.Build();

// Access configuration
var apiEndpoint = app.Configuration["MyApp:ApiEndpoint"];
Console.WriteLine($"API Endpoint: {apiEndpoint}");

app.Run();
```

---

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Applications                            │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐   │
│  │ Web App  │  │ Function │  │   VM     │  │Container │   │
│  │          │  │   App    │  │          │  │ Instance │   │
│  └─────┬────┘  └─────┬────┘  └─────┬────┘  └─────┬────┘   │
│        │             │              │              │         │
│        └─────────────┴──────────────┴──────────────┘         │
│                           │                                  │
│                           ▼                                  │
│              ┌────────────────────────┐                      │
│              │  Managed Identity      │                      │
│              │  Authentication        │                      │
│              └────────────┬───────────┘                      │
└───────────────────────────┼──────────────────────────────────┘
                            │
                            ▼
          ┌─────────────────────────────────────┐
          │  Azure App Configuration           │
          │  ┌────────────────────────────┐    │
          │  │   Configuration Data       │    │
          │  │   • Key-Value Pairs        │    │
          │  │   • Feature Flags          │    │
          │  │   • Labels (environments)  │    │
          │  └────────────────────────────┘    │
          │  ┌────────────────────────────┐    │
          │  │   Key Vault References     │────┼──┐
          │  │   • Connection strings     │    │  │
          │  │   • API keys               │    │  │
          │  └────────────────────────────┘    │  │
          └─────────────────────────────────────┘  │
                                                    │
                                                    ▼
                                    ┌───────────────────────┐
                                    │  Azure Key Vault      │
                                    │  • Secrets            │
                                    │  • Certificates       │
                                    │  • Keys               │
                                    └───────────────────────┘
```

## Поток запроса (Request Flow)

### 1. Запуск приложения (Application Startup)

- Приложение инициализируется через connection string или Managed Identity
- Configuration provider подключается к Azure App Configuration
- Начальная конфигурация загружается в память

---

### 2. Получение конфигурации (Configuration Retrieval)

- Приложение запрашивает значение настройки
- Provider проверяет локальный кэш
- Если кэш устарел — выполняется запрос в App Configuration
- Ссылки на Key Vault (Key Vault references) автоматически разрешаются

---

### 3. Динамическое обновление (Dynamic Refresh, опционально)

- Provider периодически опрашивает App Configuration
- Отслеживается sentinel key (ключ-индикатор массового обновления)
- Обновлённые значения применяются к приложению
- Перезапуск приложения не требуется

---

## Что важно понимать

- Кэширование снижает нагрузку и стоимость запросов.
- Sentinel key позволяет обновить множество настроек одним изменением.
- Key Vault reference позволяет хранить секреты в Key Vault, а не в App Configuration.

---

## Важно для AZ-204

- Dynamic refresh работает без redeploy.
- Sentinel key используется для массового обновления конфигурации.
- App Configuration + Key Vault — рекомендуемая архитектура.
- Managed Identity предпочтительнее connection string.


## Security Features

### Authentication Options

1. **Managed Identity** (Recommended)
   ```bash
   # Enable system-assigned managed identity on App Service
   az webapp identity assign --name myapp --resource-group rg
   
   # Grant access to App Configuration
   az role assignment create \
     --assignee <principal-id> \
     --role "App Configuration Data Reader" \
     --scope /subscriptions/<sub-id>/resourceGroups/rg/providers/Microsoft.AppConfiguration/configurationStores/myappconfig
   ```

2. **Connection String** (Development)
   ```bash
   # Get connection string
   az appconfig credential list --name myappconfig --resource-group rg
   
   # Use in application
   export APP_CONFIG_CONNECTION_STRING="Endpoint=https://myappconfig.azconfig.io;Id=xxx;Secret=xxx"
   ```

3. **Azure AD Service Principal** (CI/CD)
   ```bash
   # Create service principal
   az ad sp create-for-rbac --name "AppConfigReader"
   
   # Assign role
   az role assignment create \
     --assignee <app-id> \
     --role "App Configuration Data Reader" \
     --scope <config-store-resource-id>
   ```

### Network Security

**Private Endpoints**:
```bash
# Create private endpoint
az network private-endpoint create \
  --name pe-appconfig \
  --resource-group rg-network \
  --vnet-name vnet-prod \
  --subnet subnet-appconfig \
  --private-connection-resource-id <config-store-resource-id> \
  --group-id configurationStores \
  --connection-name appconfig-connection
```

**Public Access Firewall**:
```bash
# Disable public access
az appconfig update \
  --name myappconfig \
  --resource-group rg \
  --enable-public-network false

# Allow specific IP ranges
az appconfig update \
  --name myappconfig \
  --resource-group rg \
  --enable-public-network true

az appconfig network-rule add \
  --name myappconfig \
  --resource-group rg \
  --ip-address 203.0.113.0/24
```

---

## Best Practices

### 1. **Use Hierarchical Key Naming**

✅ **Good**:
```
MyApp:Database:ConnectionString
MyApp:Database:Timeout
MyApp:Cache:RedisEndpoint
MyApp:Logging:Level
```

❌ **Bad**:
```
db_connection_string
database-timeout
RedisEndpoint
log_level
```

### 2. **Leverage Labels for Environments**

```bash
# Same key, different values per environment
az appconfig kv set --key "MyApp:ApiUrl" --value "https://api-dev.contoso.com" --label "dev"
az appconfig kv set --key "MyApp:ApiUrl" --value "https://api-staging.contoso.com" --label "staging"
az appconfig kv set --key "MyApp:ApiUrl" --value "https://api.contoso.com" --label "prod"
```

### 3. **Implement Configuration Refresh**

```csharp
// Use sentinel key pattern for bulk refresh
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(connectionString)
           .Select(KeyFilter.Any, label)
           .ConfigureRefresh(refresh =>
           {
               // When Sentinel changes, refresh all config
               refresh.Register("MyApp:Sentinel", refreshAll: true)
                      .SetCacheExpiration(TimeSpan.FromMinutes(5));
           });
});
```

### 4. **Store Secrets in Key Vault**

```bash
# Create Key Vault reference
az appconfig kv set-keyvault \
  --name myappconfig \
  --key "MyApp:ConnectionString" \
  --secret-identifier "https://myvault.vault.azure.net/secrets/DbPassword"
```

### 5. **Use Managed Identities**

```csharp
// Production code - no connection strings
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(new Uri("https://myappconfig.azconfig.io"), 
                   new DefaultAzureCredential())
           .Select(KeyFilter.Any, label);
});
```

---

# Exam Tips — AZ-204 (Azure App Configuration)

## Ключевые концепции

1. **Назначение сервиса**  
   App Configuration предназначен для **настроек приложения и feature flags**,  
   а не для хранения секретов (для секретов используется Key Vault).

2. **Labels**  
   Позволяют хранить несколько значений одного ключа  
   (варианты для dev, staging, prod).

3. **Интеграция с Key Vault**  
   Можно хранить ссылку (reference) на секрет из Key Vault.

4. **Аутентификация**  
   Рекомендуется использовать Managed Identity.

5. **Dynamic Refresh**  
   Приложение может обновлять конфигурацию без перезапуска  
   (polling или push-механизм).

6. **Feature Flags**  
   Встроенная поддержка:
    - процентный rollout
    - таргетинг по пользователям и группам

7. **Тарифы**
    - Free — 1 000 запросов/день
    - Standard — без ограничений, SLA, Private Endpoint, CMK

8. **Ограничение размера**  
   10 KB на одну пару key-value  
   (не хранить большие данные или payload’ы).

9. **Клиентские библиотеки**  
   Поддержка для:
    - .NET
    - Java Spring
    - JavaScript
    - Python

10. **Point-in-Time**  
    Поддержка snapshot’ов конфигурации для:
    - аудита
    - отката
    - compliance

11. **Private Endpoints**  
    Позволяют изолировать сервис от публичного интернета.

12. **Не замена Key Vault**
    - Connection strings → App Configuration
    - Пароли → Key Vault

---

# Частые экзаменационные сценарии

### Сценарий 1
> Нужно изменить уровень логирования без redeploy

→ Использовать **App Configuration + Dynamic Refresh**

---

### Сценарий 2
> Нужно безопасно хранить пароль базы данных

→ Хранить в **Key Vault**,  
в App Configuration — только reference

---

### Сценарий 3
> Включить новую функцию для 25% пользователей

→ Использовать **Feature Flags + Percentage Filter**

---

### Сценарий 4
> Разные API endpoint для dev/staging/prod

→ Использовать **Labels**

---

### Сценарий 5
> Аутентифицировать App Service к App Configuration

→ Включить **Managed Identity**  
→ Назначить роль **App Configuration Data Reader**

---

## Главное, что нужно запомнить

- App Configuration = настройки + feature flags.
- Key Vault = секреты.
- Managed Identity — рекомендуемый способ аутентификации.
- Dynamic refresh = без перезапуска.
- Labels = окружения.

---

## Quick Reference Commands

```bash
# Create App Configuration store
az appconfig create --name <name> --resource-group <rg> --location <location> --sku Standard

# Add key-value
az appconfig kv set --name <name> --key <key> --value <value>

# Add with label
az appconfig kv set --name <name> --key <key> --value <value> --label <label>

# List all key-values
az appconfig kv list --name <name>

# List with specific label
az appconfig kv list --name <name> --label <label>

# Delete key-value
az appconfig kv delete --name <name> --key <key>

# Get connection string
az appconfig credential list --name <name> --resource-group <rg>

# Add Key Vault reference
az appconfig kv set-keyvault --name <name> --key <key> --secret-identifier <vault-uri>

# Enable managed identity
az appconfig identity assign --name <name> --resource-group <rg>

# Grant access
az role assignment create \
  --assignee <principal-id> \
  --role "App Configuration Data Reader" \
  --scope <config-store-resource-id>
```

---

## Learn More

- [Azure App Configuration Documentation](https://docs.microsoft.com/azure/azure-app-configuration/)
- [Best Practices](https://docs.microsoft.com/azure/azure-app-configuration/howto-best-practices)
- [Feature Management](https://docs.microsoft.com/azure/azure-app-configuration/concept-feature-management)
- [.NET Quickstart](https://docs.microsoft.com/azure/azure-app-configuration/quickstart-dotnet-core-app)
