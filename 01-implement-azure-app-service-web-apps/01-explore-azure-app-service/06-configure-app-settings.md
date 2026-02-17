# Настройка Application Settings в Azure App Service

## Ключевые концепции

- **Application Settings** — это переменные окружения для приложения.
- Значения **всегда шифруются при хранении (encrypted at rest)**.
- В ASP.NET-приложениях могут переопределять значения из:
    - `Web.config`
    - `appsettings.json`
- Изменение настроек вызывает **автоматический перезапуск приложения**.
- Можно помечать настройки как **slot-specific** (привязанные к конкретному deployment slot).

💡 Application Settings — предпочтительный способ хранения конфигурации в Azure, вместо жёстко заданных значений в коде.

---

## Application Settings

### Правила именования

- ✅ Разрешены:
    - буквы
    - цифры
    - точки (`.`)
    - подчёркивания (`_`)

### Особенности для Linux

- ⚠️ Двоеточие `:` заменяется на `__` (двойное подчёркивание) для вложенных JSON-ключей.
- ⚠️ Точка `.` автоматически заменяется на `_` (одно подчёркивание).

Пример:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information"
    }
  }
}
```
В Linux Application Settings это будет:

```json
Logging__LogLevel__Default=Information

```
### Examples

```bash
# Set app setting (CLI)
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg-name> \
  --settings KEY1=value1 KEY2=value2

# Get app settings
az webapp config appsettings list \
  --name <app-name> \
  --resource-group <rg-name>

# Delete app setting
az webapp config appsettings delete \
  --name <app-name> \
  --resource-group <rg-name> \
  --setting-names KEY1 KEY2
```

### PowerShell

```powershell
# Set app settings
Set-AzWebApp -ResourceGroupName <rg-name> `
  -Name <app-name> `
  -AppSettings @{"DB_HOST"="server.database.azure.com"}
```

## Slot Settings (Настройки, привязанные к слоту)

### Что такое Slot Settings?

- Это настройки, которые **не участвуют в swap** между deployment slots.
- Они **привязаны к конкретному слоту** (например, staging или production).
- Предотвращают перенос production-конфигурации в staging при swap.

💡 Используются для изоляции конфигурации между средами.

---

## Когда использовать Slot Settings

Используйте slot-specific настройки для:

- ✅ Connection strings (например, production-база данных)
- ✅ Feature flags (включены только в staging)
- ✅ Конфигурации, зависящей от среды (environment-specific config)
- ✅ API-ключей, отличающихся для разных сред

---

## Важные замечания

- Slot settings помечаются галочкой "Deployment slot setting" в портале.
- При выполнении swap значения этих настроек **остаются в своём слоте**.
- Без использования slot settings можно случайно подключить staging к production-базе данных.

---

## Экзаменационные ловушки (AZ-204)

- Swap переносит код и обычные настройки, но **не переносит slot settings**.
- Connection strings для production почти всегда должны быть slot-specific.
- Если в задаче требуется изолировать конфигурацию между средами → использовать slot settings.

### Configuration

```json
[
  {
    "name": "ASPNETCORE_ENVIRONMENT",
    "value": "Production",
    "slotSetting": true  // Doesn't swap
  },
  {
    "name": "FEATURE_FLAG",
    "value": "enabled",
    "slotSetting": false  // Swaps with slot
  }
]
```

## Connection Strings в Azure App Service

### Назначение

- Используются преимущественно в **ASP.NET и ASP.NET Core**.
- Сохраняются вместе с приложением при резервном копировании (в отличие от App Settings).
- Для других языков рекомендуется использовать Application Settings.

💡 В современных архитектурах чаще рекомендуется использовать Managed Identity вместо хранения connection strings.

---

## Типы Connection Strings

| Тип | Префикс переменной окружения | Пример использования |
|------|------------------------------|----------------------|
| **SQLServer** | `SQLCONNSTR_` | SQL Server |
| **MySQL** | `MYSQLCONNSTR_` | MySQL |
| **SQLAzure** | `SQLAZURECONNSTR_` | Azure SQL Database |
| **PostgreSQL** | `POSTGRESQLCONNSTR_` | PostgreSQL |
| **Custom** | `CUSTOMCONNSTR_` | Кастомные подключения |
| **Redis Cache** | `REDISCACHECONNSTR_` | Redis |
| **Document DB** | `DOCDBCONNSTR_` | Cosmos DB |
| **Event Hub** | `EVENTHUBCONNSTR_` | Event Hubs |
| **Service Bus** | `SERVICEBUSCONNSTR_` | Service Bus |
| **Notification Hub** | `NOTIFICATIONHUBCONNSTR_` | Notification Hubs |

---

## Доступ к Connection Strings в коде

### В ASP.NET Core

Через `IConfiguration`:

```csharp
// C# - Access connection string
var connString = Environment.GetEnvironmentVariable("SQLCONNSTR_MyDatabase");
```

```javascript
// Node.js - Access connection string
const connString = process.env.SQLCONNSTR_MyDatabase;
```

### Configure Connection String

```bash
# Set connection string
az webapp config connection-string set \
  --name <app-name> \
  --resource-group <rg-name> \
  --connection-string-type SQLAzure \
  --settings MyDb="Server=tcp:server.database.azure.com..."
```

### JSON Format

```json
[
  {
    "name": "MyDatabase",
    "value": "Server=tcp:server.database.azure.com;Database=mydb",
    "type": "SQLAzure",
    "slotSetting": true
  },
  {
    "name": "RedisCache",
    "value": "mycache.redis.cache.windows.net:6380,password=...",
    "type": "Custom",
    "slotSetting": false
  }
]
```

## Важные замечания

- Connection Strings доступны как переменные окружения.
- Можно пометить как **slot-specific** (Deployment slot setting).
- Поддерживают шифрование при хранении (encrypted at rest).
- Используются преимущественно в .NET-приложениях.

---

## Экзаменационные ловушки (AZ-204)

- Connection Strings предназначены в первую очередь для ASP.NET.
- Для других языков предпочтительнее использовать **App Settings**.
- Slot settings не переносятся при swap.
- В production рекомендуется использовать **Managed Identity** вместо хранения секретов.
- Connection Strings сохраняются при backup приложения.

---

## Best Practice

Если ресурс поддерживает Azure AD / Entra ID:

- ✔ Используйте **Managed Identity + RBAC**
- ✖ Избегайте хранения connection strings с логином и паролем


## Custom Containers

### Set Environment Variables

```bash
# Bash
az webapp config appsettings set \
  --resource-group <rg-name> \
  --name <app-name> \
  --settings KEY1=value1 KEY2=value2

# PowerShell
Set-AzWebApp -ResourceGroupName <rg-name> `
  -Name <app-name> `
  -AppSettings @{"DB_HOST"="myserver.database.azure.com"}
```

### Verify Container Environment

```
URL: https://<app-name>.scm.azurewebsites.net/Env
```

### Injection Method
- Settings injected with `--env` flag (Linux containers)
- Available as environment variables at app startup

## Quick Reference

| Setting Type | Encrypted | Backed Up | Naming Restrictions |
|--------------|-----------|-----------|---------------------|
| **App Settings** | ✅ Yes | ❌ No | Letters, numbers, `.`, `_` |
| **Connection Strings** | ✅ Yes | ✅ Yes | Same as above |
| **Slot Settings** | ✅ Yes | ❌ No | Must be flagged explicitly |

## Bulk Edit JSON Format

```json
[
  {
    "name": "ASPNETCORE_ENVIRONMENT",
    "value": "Production",
    "slotSetting": true
  },
  {
    "name": "ApplicationInsights__InstrumentationKey",
    "value": "key-value-here",
    "slotSetting": false
  }
]
```

## Критические замечания

- 💡 Используйте **Key Vault** для хранения секретов, а не App Settings.
- 🔄 Изменение настроек вызывает **перезапуск приложения** — учитывайте это при обновлении конфигурации.
- 🎯 Slot settings не участвуют в swap — идеально подходят для environment-specific конфигурации.
- ⚠️ В Linux для вложенных ключей используйте `__` вместо `:`.
- 📝 Connection strings предназначены в основном для .NET-приложений.
- 🔐 Значения шифруются при хранении (encrypted at rest), но это не означает сквозное шифрование.
- ⚠️ Для .NET + PostgreSQL рекомендуется указывать тип connection string как **"Custom"** (известный workaround).

---

## Лучшие практики

1. ✅ Используйте **Key Vault references** для хранения секретов.
2. ✅ Помечайте настройки, зависящие от среды, как **slot settings**.
3. ✅ Используйте **connection strings** для подключения к БД в .NET.
4. ✅ Используйте **app settings** для приложений не на .NET.
5. ⚠️ Никогда не храните секреты в системе контроля версий.

---

## Экзаменационные советы (AZ-204)

- Различайте **app settings** и **connection strings**.
- Понимайте, какие настройки участвуют в swap, а какие нет.
- Запомните префиксы переменных окружения для connection strings.
- Помните, что изменение настроек вызывает перезапуск приложения.
- Учитывайте правила именования в Linux (`:` → `__`, `.` → `_`).


[Learn More](https://learn.microsoft.com/en-us/training/modules/configure-web-app-settings/2-configure-application-settings)
