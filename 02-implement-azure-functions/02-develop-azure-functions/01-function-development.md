# Azure Functions Development (Разработка Azure Functions)

## Key Concepts (Ключевые понятия)

- **Function App** — контейнер для группы функций
- **host.json** — конфигурация всего Function App
- **local.settings.json** — настройки для локальной разработки
- **function.json** — конфигурация конкретной функции (не для C# compiled)
- **Local development** — разработка и тестирование локально с последующим деплоем в Azure

---

# Function App Structure (Структура Function App)

## What Is a Function App? (Что такое Function App?)

**Function App** — это контейнер, который управляет, деплоит и масштабирует функции вместе.

### Основные характеристики

- 🧩 **Execution context**  
  Все функции работают в одном runtime-окружении

- 📦 **Deployment unit**  
  Деплой происходит целиком на уровне Function App

- 💰 **Pricing plan**  
  Все функции используют один и тот же hosting plan

- 🔄 **Runtime version**  
  Все функции используют одну версию runtime (например, 4.x)

- 🧑‍💻 **Language constraint**  
  Начиная с версии 2.x — все функции в Function App должны быть написаны на одном языке

> 💡 Масштабирование также происходит на уровне Function App  
> (кроме Flex Consumption, где возможно масштабирование на уровне отдельных функций).

---

## Важно для AZ-204

- Function App = единица масштабирования и деплоя
- Все функции делят:
    - план
    - runtime
    - конфигурацию
- host.json влияет на поведение всех функций
- Нельзя смешивать разные языки в одном Function App


### Organizational Benefits
```
Function App: OrderProcessing
├── ValidateOrder (HTTP trigger)
├── ProcessPayment (Queue trigger)
├── SendConfirmation (Queue trigger)
└── UpdateInventory (Queue trigger)

Benefits:
- Logical grouping of related functions
- Shared configuration (host.json)
- Single deployment
- Unified monitoring
```

## Project Structure

### Essential Files

#### host.json
**Function app-wide configuration**:

```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond": 20
      }
    }
  },
  "functionTimeout": "00:05:00",
  "extensions": {
    "http": {
      "routePrefix": "api",
      "maxConcurrentRequests": 100
    },
    "queues": {
      "batchSize": 16,
      "maxDequeueCount": 5
    }
  }
}
```

### host.json

Глобальный файл конфигурации для всего Function App.

**Key settings:**

- `version` — версия схемы (обычно `"2.0"`)
- `functionTimeout` — максимальное время выполнения функции
- `extensions` — настройки для конкретных триггеров и биндингов
- `logging` — конфигурация логирования (Application Insights и др.)

> 💡 host.json влияет на все функции внутри Function App.

---

### local.settings.json

Файл конфигурации для **локальной разработки**  
(в Azure не деплоится).

Используется для:

- Хранения connection strings
- Настроек среды разработки
- Локальных значений переменных окружения

Пример структуры:

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "dotnet"
  }
}
```
⚠️ Этот файл не публикуется в Azure —
для продакшена используются Application Settings в портале.
```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "MyDatabaseConnection": "Server=localhost;...",
    "ApiKey": "local-dev-key-123"
  },
  "Host": {
    "LocalHttpPort": 7071,
    "CORS": "*"
  },
  "ConnectionStrings": {
    "SQLConnectionString": "..."
  }
}
```

### local.settings.json (структура)

**Structure:**

- `Values` — App Settings  
  (connection strings, API keys, environment variables)
- `Host` — настройки локального Functions Host
- `ConnectionStrings` — строки подключения к БД
- `IsEncrypted` — флаг шифрования значений

⚠️ **Важно:**  
Никогда не коммитить `local.settings.json` в систему контроля версий —  
файл содержит секреты.

> 💡 В Azure используйте Application Settings вместо этого файла.

---

### function.json (JavaScript / Python / PowerShell)

Файл конфигурации **конкретной функции**.

Используется для:

- Определения trigger
- Настройки input/output bindings
- Конфигурации направления данных

Пример структуры:

```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "authLevel": "function",
      "direction": "in",
      "name": "req",
      "methods": [ "get", "post" ]
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```
Основные элементы
- type — тип триггера или биндинга
- direction — in или out
- name — имя параметра в коде
- authLevel — уровень авторизации (для HTTP)
- methods — допустимые HTTP-методы

📌 В C# compiled функции конфигурация задаётся через атрибуты,
а function.json генерируется автоматически.

```json
{
  "disabled": false,
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["get", "post"],
      "authLevel": "function"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```

**C# equivalent** (attributes instead of function.json):
```csharp
[FunctionName("HttpExample")]
public static IActionResult Run(
    [HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req,
    ILogger log)
{
    // Function code
}
```

### Directory Structure

#### Node.js / Python / PowerShell
```
MyFunctionApp/
├── host.json
├── local.settings.json
├── package.json (Node.js)
├── requirements.txt (Python)
├── HttpTriggerFunction/
│   ├── function.json
│   └── index.js
├── QueueTriggerFunction/
│   ├── function.json
│   └── index.js
└── TimerTriggerFunction/
    ├── function.json
    └── index.js
```

#### C# (.NET)
```
MyFunctionApp/
├── MyFunctionApp.csproj
├── host.json
├── local.settings.json
├── HttpTriggerFunction.cs
├── QueueTriggerFunction.cs
└── TimerTriggerFunction.cs
```

# Local Development (Локальная разработка)

## Benefits (Преимущества)

✅ **Полноценный runtime** — локально запускается полный Azure Functions Runtime  
✅ **Live connections** — можно подключаться к реальным Azure-сервисам  
✅ **Debugging** — полноценная отладка  
✅ **Быстрая итерация** — тестирование без деплоя в облако

> 💡 Позволяет разрабатывать и тестировать функции так же, как обычное приложение.

---

## Prerequisites (Необходимые инструменты)

| Tool | Purpose |
|------|----------|
| **Azure Functions Core Tools** | Локальный runtime и CLI |
| **Visual Studio Code** | Редактор кода (рекомендуется) |
| **Azure Functions extension** | Интеграция с VS Code |
| **Language runtime** | Node.js, Python, .NET и т.д. |

---

## Важно для AZ-204

- Core Tools позволяют запускать функции локально
- Можно подключаться к Azure Storage и другим сервисам
- local.settings.json используется только локально
- Отладка полностью поддерживается в VS Code и Visual Studio

### Installation

#### Azure Functions Core Tools
```bash
# macOS (Homebrew)
brew tap azure/functions
brew install azure-functions-core-tools@4

# Windows (Chocolatey)
choco install azure-functions-core-tools

# Ubuntu/Debian
curl https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > microsoft.gpg
sudo mv microsoft.gpg /etc/apt/trusted.gpg.d/microsoft.gpg
sudo sh -c 'echo "deb [arch=amd64] https://packages.microsoft.com/repos/microsoft-ubuntu-$(lsb_release -cs)-prod $(lsb_release -cs) main" > /etc/apt/sources.list.d/dotnetdev.list'
sudo apt-get update
sudo apt-get install azure-functions-core-tools-4
```

#### VS Code Extension
```
1. Open VS Code
2. Extensions (Ctrl+Shift+X)
3. Search "Azure Functions"
4. Install Microsoft's Azure Functions extension
```

### Create New Project

#### Using Core Tools
```bash
# Initialize new function app
func init MyFunctionApp --worker-runtime node

# Navigate to project
cd MyFunctionApp

# Create function
func new --name HttpExample --template "HTTP trigger"

# Run locally
func start
```

#### Using VS Code
```
1. Command Palette (Ctrl+Shift+P)
2. "Azure Functions: Create New Project"
3. Select folder
4. Choose language
5. Select runtime version
6. Choose template
7. Provide function name
```

### Local Testing

#### Start Local Runtime
```bash
# Start Functions host
func start

# Output:
# Azure Functions Core Tools
# Core Tools Version:       4.0.5455
# Function Runtime Version: 4.27.5.21554
#
# Functions:
#   HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
#
# For detailed output, run func with --verbose flag.
```

#### Test HTTP Function
```bash
# Test with curl
curl http://localhost:7071/api/HttpExample?name=Azure

# Test with body
curl -X POST http://localhost:7071/api/HttpExample \
  -H "Content-Type: application/json" \
  -d '{"name":"Azure"}'
```

#### Debug in VS Code
```
1. Set breakpoints in code
2. Press F5 (Start Debugging)
3. Functions host starts with debugger attached
4. Trigger function (HTTP request, queue message, etc.)
5. Breakpoint hits, inspect variables
```

### Testing Triggers Locally

#### HTTP Trigger
```bash
# Direct call to localhost endpoint
curl http://localhost:7071/api/HttpExample
```

#### Storage Queue Trigger
**Option 1**: Use Azurite emulator
```bash
# Install Azurite
npm install -g azurite

# Start Azurite
azurite --silent --location c:\azurite --debug c:\azurite\debug.log

# In local.settings.json:
"AzureWebJobsStorage": "UseDevelopmentStorage=true"
```

**Option 2**: Use live Azure Storage
```json
// local.settings.json
{
  "Values": {
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=..."
  }
}
```

#### Timer Trigger
```bash
# Runs automatically on schedule
# Use short interval for testing: "*/10 * * * * *" (every 10 seconds)
```

#### Manual Trigger
```bash
# Admin endpoint for non-HTTP triggers
curl -X POST http://localhost:7071/admin/functions/QueueTriggerFunction \
  -H "Content-Type: application/json" \
  -d '{"input":"test data"}'
```

## Settings Synchronization

### Deploy Settings to Azure
```bash
# Upload local.settings.json to Azure
func azure functionapp publish <function-app-name> --publish-settings-only

# Or manually add each setting
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings \
    "DatabaseConnection=Server=..." \
    "ApiKey=prod-key-456"
```

### Download Settings from Azure
```bash
# Download app settings to local.settings.json
func azure functionapp fetch-app-settings <function-app-name>

# Decrypt settings (if encrypted)
func settings decrypt
```

## Best Practices (Лучшие практики)

✅ **Разделяйте окружения**  
Используйте разные настройки для dev / test / prod

✅ **Используйте Azure Key Vault**  
Храните секреты вне кода и конфигурационных файлов

✅ **Никогда не коммитьте секреты**  
Добавьте `local.settings.json` в `.gitignore`

✅ **Понятные имена подключений**  
Используйте описательные имена для connection strings  
(например: `OrdersStorageConnection`, а не `Storage1`)

> 💡 В Azure используйте Application Settings + Key Vault references.

---

# Portal Development Limitations (Ограничения разработки в портале)

## Limited Portal Editing (Ограниченное редактирование)

❌ **C# compiled** — нельзя редактировать в портале  
❌ **Python** — нельзя редактировать в портале  
❌ **Java** — нельзя редактировать в портале  
❌ **TypeScript** — нельзя редактировать в портале

✅ **C# Script (.csx)** — можно редактировать  
✅ **JavaScript** — можно редактировать  
✅ **PowerShell** — можно редактировать

---

## Recommendation (Рекомендация)

💡 Для production-разработки всегда лучше разрабатывать локально:

- Полноценная IDE
- Полная поддержка debugging
- Интеграция с системой контроля версий
- CI/CD pipelines
- Поддержка unit-тестирования

---

## Важно для AZ-204

- Portal подходит только для простых сценариев
- Production-разработка должна вестись локально
- Секреты не хранятся в коде
- Key Vault — рекомендуемый способ хранения чувствительных данных

## Configuration Examples

### host.json - Production Settings
```json
{
  "version": "2.0",
  "logging": {
    "applicationInsights": {
      "samplingSettings": {
        "isEnabled": true,
        "maxTelemetryItemsPerSecond": 20
      }
    },
    "logLevel": {
      "default": "Information",
      "Host.Results": "Error",
      "Function": "Error",
      "Host.Aggregator": "Trace"
    }
  },
  "functionTimeout": "00:05:00",
  "extensions": {
    "http": {
      "routePrefix": "api",
      "maxConcurrentRequests": 100,
      "maxOutstandingRequests": 200,
      "dynamicThrottlesEnabled": true
    },
    "queues": {
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8,
      "visibilityTimeout": "00:00:30"
    },
    "serviceBus": {
      "prefetchCount": 100,
      "messageHandlerOptions": {
        "maxConcurrentCalls": 32,
        "autoComplete": true
      }
    }
  }
}
```

### local.settings.json - Development
```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "UseDevelopmentStorage=true",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "APPINSIGHTS_INSTRUMENTATIONKEY": "",
    "DatabaseConnection": "Server=localhost;Database=TestDB;",
    "ExternalApiUrl": "https://dev-api.example.com",
    "ApiKey": "dev-key-123"
  },
  "Host": {
    "LocalHttpPort": 7071,
    "CORS": "*",
    "CORSCredentials": false
  }
}
```
## Critical Notes (Критически важные моменты)

- 💡 **Function App** — единица деплоя, содержит несколько функций
- ⚠️ Начиная с версии 2.x — все функции в одном приложении должны быть на одном языке
- 🎯 `host.json` — конфигурация всего Function App
- 📊 `local.settings.json` — нельзя коммитить в систему контроля версий
- ✅ Локальная разработка рекомендуется для всех сценариев
- 🔄 Синхронизация настроек через CLI (загрузка/выгрузка settings)
- ⏱️ **Azurite** — локальный эмулятор Azure Storage
- 🔒 Ограничения портала — C#, Python, Java не редактируются в портале

---

## Exam Tips (Советы для экзамена)

- Function App = контейнер для связанных функций
- Все функции внутри приложения разделяют:
    - Runtime
    - Hosting plan
    - Deployment
- Functions 2.x+ → один язык на приложение
- `host.json` управляет timeout, extensions, logging
- `local.settings.json` используется только локально
- Никогда не коммитьте `local.settings.json`
- C# (compiled) использует атрибуты  
  Другие языки используют `function.json`
- В портале можно редактировать только:
    - C# Script
    - JavaScript
    - PowerShell
- Локальная разработка предпочтительна
- Azure Functions Core Tools — CLI для локального запуска
- Azurite — локальный Storage emulator
- Синхронизация настроек:

```json
func azure functionapp publish --publish-settings-only
```
- Админ endpoint для ручного запуска:
```json
http://localhost:7071/admin/functions/{name}
```

Важно для AZ-204

--publish-settings-only обновляет только конфигурацию

Admin endpoint используется только локально

Позволяет тестировать триггеры, которые нельзя вызвать напрямую

Работает через Functions Core Tools
[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-azure-functions/2-azure-function-development-overview)
