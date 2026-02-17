# Azure Functions Overview

## Key Concepts
- **Serverless compute** - No infrastructure management required
- **Event-driven** - Code runs in response to triggers
- **Triggers** - Events that start function execution
- **Bindings** - Simplified input/output data connections
- **Pay-per-execution** - Only charged when code runs

# What Is Azure Functions? (Что такое Azure Functions?)

## Definition (Определение)

**Azure Functions** — это serverless-сервис вычислений,  
позволяющий запускать событийный код без управления инфраструктурой.

- ✍️ **Меньше кода** — фокус только на бизнес-логике
- 🖥 **Без управления серверами** — инфраструктура управляется Azure
- 💰 **Оплата за выполнение** — платите только за время исполнения
- ⚡ **Автомасштабирование** — масштабируется автоматически при нагрузке

> 💡 Это классический пример модели *Function-as-a-Service (FaaS)*.

---

## Use Cases (Сценарии использования)

- **Web APIs** — HTTP-triggered REST API
- **Обработка данных** — файлы, изображения, стримы
- **Интеграция систем** — связывание различных сервисов
- **IoT-потоки** — обработка телеметрии устройств
- **Очереди сообщений** — workflow на основе очередей
- **Запланированные задачи** — cron / timer-trigger
- **Микросервисы** — небольшие изолированные сервисы

---

# Core Concepts (Базовые концепции)

---

## Triggers (Триггеры)

**Запускают выполнение функции**

| Trigger Type | Description | Example |
|--------------|-------------|----------|
| **HTTP** | Веб-запрос | REST API |
| **Timer** | Расписание (cron) | Ночная очистка |
| **Queue** | Сообщение в очереди | Обработка заказа |
| **Blob** | Загрузка файла | Ресайз изображения |
| **Event Hub** | Поток событий | IoT-телеметрия |
| **Event Grid** | Событие Azure | Изменение ресурса |
| **Service Bus** | Enterprise messaging | Обработка заказов |
| **Cosmos DB** | Изменение в БД | Синхронизация данных |

📌 **Правило:**  
Каждая функция имеет **ровно один trigger**.

---

## Bindings (Биндинги)

Упрощают подключение к внешним сервисам.

- **Input binding** — чтение данных из внешнего источника
- **Output binding** — запись данных во внешний сервис
- **Declarative** — настраиваются через:
    - `function.json`
    - атрибуты в коде

📌 **Правило:**  
Функция может иметь **несколько bindings**.

---

## Важно для AZ-204

- Trigger всегда один
- Bindings может быть несколько
- Azure Functions автоматически масштабируются в Consumption Plan
- Serverless ≠ без серверов, а без управления ими


### Example: Bindings in Action
```csharp
[FunctionName("ProcessOrder")]
public static void Run(
    [QueueTrigger("orders")] string order,          // Trigger
    [Blob("receipts/{id}")] out string receipt,      // Output binding
    [CosmosDB("orders", "history")] out Order record // Output binding
)
{
    // Process order
    receipt = GenerateReceipt(order);
    record = SaveToHistory(order);
}
```
**Result**:  
Функция запускается по событию из очереди и автоматически записывает результат в Blob Storage и Cosmos DB через output bindings.

---

# Azure Functions vs Azure Logic Apps

## Comparison Table (Сравнение)

| Feature | Azure Functions | Azure Logic Apps |
|----------|-----------------|------------------|
| **Development** | Code-first (императивный подход) | Designer-first (декларативный) |
| **Skill set** | Разработчики (C#, JS, Python и др.) | Аналитики и разработчики |
| **Orchestration** | Durable Functions | Визуальный workflow designer |
| **Connectivity** | ~12 встроенных bindings + код | 400+ коннекторов, B2B pack |
| **Actions** | Нужно писать код | Готовые действия |
| **Monitoring** | Application Insights | Azure Portal, Monitor logs |
| **Management** | REST API, Visual Studio, CLI | Portal, REST API, PowerShell |
| **Execution** | Azure, локально | Azure, локально, on-premises |
| **Pricing** | Оплата за выполнение | Оплата за действие/коннектор |
| **Source control** | Git integration | JSON-definition |

---

## When to Choose Azure Functions (Когда выбирать Functions)

✅ Нужно писать собственную бизнес-логику  
✅ Есть разработчики  
✅ Требуется контроль над выполнением  
✅ Большие объёмы при низкой стоимости  
✅ Интеграция с Application Insights

---

## When to Choose Logic Apps (Когда выбирать Logic Apps)

✅ Бизнес-процессы (approval, routing)  
✅ Нужен визуальный дизайнер  
✅ Enterprise-интеграции (SAP, Oracle и др.)  
✅ B2B-сценарии (EDI, AS2, X12)  
✅ Low-code / no-code подход

---

# Azure Functions vs WebJobs

## Comparison Table (Сравнение)

| Feature | Azure Functions | WebJobs + WebJobs SDK |
|----------|-----------------|----------------------|
| **Serverless model** | ✅ Да | ❌ Нет |
| **Automatic scaling** | ✅ Да | ❌ Нет (масштабирование через App Service) |
| **Browser development** | ✅ Portal editor | ❌ Требуется IDE |
| **Pay-per-use pricing** | ✅ Consumption plan | ❌ Оплата App Service Plan |
| **Logic Apps integration** | ✅ Да | ❌ Нет |
| **Trigger events** | HTTP, Timer, Storage, Service Bus, Cosmos DB, Event Hubs, Event Grid, Webhooks | Timer, Storage, Service Bus, Cosmos DB, Event Hubs, File system |
| **Host** | Managed Functions host | App Service Plan |
| **Deployment** | Отдельный Function App | Привязан к Web App |

---

## When to Choose Azure Functions

✅ Нужна serverless-архитектура  
✅ Требуется автоматическое масштабирование  
✅ Модель оплаты за выполнение  
✅ HTTP-triggered API  
✅ Интеграция с Event Grid  
✅ Быстрая разработка в портале

---

## When to Choose WebJobs

✅ Уже есть App Service Plan  
✅ Нужен полный контроль над хостом  
✅ Фоновая обработка для существующего Web App  
✅ Нужен File system trigger  
✅ Долгоживущие continuous jobs

---

# Architecture Components (Архитектурные компоненты)

---

## Function App

- 📦 Контейнер для одной или нескольких функций
- 🔗 Делят один hosting plan, deployment и runtime
- 🗂 Объединяет связанные функции
- ⚖️ Единица масштабирования и деплоя

> 💡 Масштабирование происходит на уровне Function App.

---

## Functions Runtime

- ▶️ Выполняет код функций
- 🔌 Управляет triggers и bindings
- 📈 Обеспечивает масштабирование
- 🔄 Управляет жизненным циклом

### Versions

- **4.x** — текущая версия
- 3.x
- 2.x
- 1.x

---

## Важно для AZ-204

- Function App — это единица деплоя и масштабирования
- Один trigger на функцию, bindings — несколько
- Functions = code-first serverless
- Logic Apps = workflow-first orchestration
- WebJobs не являются serverless


### Host Configuration
Configured in `host.json`:
```json
{
  "version": "2.0",
  "functionTimeout": "00:05:00",
  "extensions": {
    "http": {
      "routePrefix": "api",
      "maxConcurrentRequests": 100
    }
  }
}
```

# Supported Languages (Поддерживаемые языки)

## In-Portal Development (Разработка прямо в портале)

| Language | Runtime | Portal Support |
|-----------|----------|----------------|
| **C# Script** | .NET 6, 7, 8 | ✅ Yes |
| **JavaScript** | Node.js 18, 20 | ✅ Yes |
| **PowerShell** | PowerShell 7.2, 7.4 | ✅ Yes |

> 💡 Эти языки можно редактировать прямо в Azure Portal (inline editor).

---

## Local Development Only (Только локальная разработка)

| Language | Runtime | Portal Support |
|-----------|----------|----------------|
| **C# (compiled)** | .NET 6, 7, 8, Isolated | ❌ No |
| **Python** | 3.8, 3.9, 3.10, 3.11 | ❌ No |
| **Java** | Java 8, 11, 17 | ❌ No |
| **TypeScript** | Node.js 18, 20 | ❌ No |

⚠️ Разработка для C#, Python, Java и TypeScript выполняется локально  
(Visual Studio, VS Code, CLI), затем деплой в Azure.

---

## Важно для AZ-204

- Portal поддерживает только C# Script, JavaScript и PowerShell
- Compiled C# (Isolated) требует локальной сборки
- Python и Java всегда разрабатываются локально
- Runtime version должен соответствовать поддерживаемой версии Functions Runtime (4.x)


## Development Workflow

### 1. Create Function App
```bash
# CLI
az functionapp create \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --consumption-plan-location <region> \
  --runtime node \
  --runtime-version 18 \
  --storage-account <storage-name>
```

### 2. Create Function
```bash
# Using Azure Functions Core Tools
func init MyFunctionApp --worker-runtime node
cd MyFunctionApp
func new --name HttpExample --template "HTTP trigger"
```

### 3. Test Locally
```bash
func start

# Output:
# HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
```

### 4. Deploy
```bash
func azure functionapp publish <function-app-name>
```
## Function Execution Flow (Поток выполнения функции)

```
Происходит событие (HTTP-запрос, таймер, сообщение и т.д.)
↓

Срабатывает trigger
↓

Functions Runtime вызывает функцию
↓

Input bindings получают данные (если настроены)
↓

Выполняется код функции
↓

Output bindings записывают данные (если настроены)
↓

Функция завершает выполнение
↓

Возвращается ответ (для синхронных триггеров, например HTTP)
```


> 💡 Для асинхронных триггеров (Queue, Event Hub и др.) HTTP-ответа нет — выполнение просто завершается.

---

# Integration Services Comparison (Сравнение интеграционных сервисов)

| Service | Purpose | Best For | Development |
|-----------|----------|-----------|--------------|
| **Azure Functions** | Serverless compute | Кастомная логика, обработка данных | Code-first |
| **Logic Apps** | Оркестрация процессов | Бизнес-процессы, интеграция | Designer-first |
| **Event Grid** | Распределение событий | Pub/Sub, реактивные сценарии | Конфигурация |
| **Service Bus** | Корпоративный messaging | Надёжная доставка сообщений | Messaging |
| **Power Automate** | Бизнес-автоматизация | Office 365, citizen developers | Low-code |

---

## Как запомнить для AZ-204

- **Functions** → код + serverless
- **Logic Apps** → workflow + визуальный дизайнер
- **Event Grid** → события (publish/subscribe)
- **Service Bus** → гарантированная доставка сообщений
- **Power Automate** → low-code автоматизация

> 🎯 Часто Functions и Logic Apps используются вместе.

## Quick Command Reference

```bash
# Create function app
az functionapp create \
  --name <app-name> \
  --resource-group <rg> \
  --consumption-plan-location <region> \
  --runtime <node|python|dotnet|java> \
  --storage-account <storage>

# List function apps
az functionapp list \
  --resource-group <rg> \
  --output table

# Get function app URL
az functionapp show \
  --name <app-name> \
  --resource-group <rg> \
  --query "defaultHostName" -o tsv

# View logs
az functionapp log tail \
  --name <app-name> \
  --resource-group <rg>

# Update app settings
az functionapp config appsettings set \
  --name <app-name> \
  --resource-group <rg> \
  --settings KEY=VALUE
```

## Critical Notes (Критически важные моменты)

- 💡 **Serverless** — не требуется управление инфраструктурой
- ⚠️ **Один trigger** — у каждой функции ровно один trigger
- 🎯 **Несколько bindings** — можно использовать много input/output bindings
- 📊 **Event-driven модель** — код выполняется по событию
- ✅ **Автомасштабирование** — масштабируется автоматически при нагрузке
- 🔄 **Интеграция** — работает с Logic Apps, Event Grid, Service Bus и др.
- ⏱️ **Pay-per-execution** — оплата только за выполнение (Consumption plan)
- 🔒 **Managed runtime** — обновления и патчи управляются Microsoft

---

## Exam Tips (Советы для экзамена)

- Azure Functions = serverless + event-driven compute
- У функции всегда **ровно один trigger**
- Bindings могут быть множественными
- Functions vs Logic Apps:
  - Code-first vs Designer-first
- Functions vs WebJobs:
  - Serverless + autoscale vs App Service hosted
- WebJobs не поддерживают HTTP trigger и Event Grid
- Trigger запускает выполнение, bindings упрощают ввод/вывод
- Поддерживаемые языки: C#, JavaScript, Python, Java, PowerShell, TypeScript
- Разработка в портале: только C# Script, JavaScript, PowerShell
- Function App — контейнер для группы функций
- `host.json` управляет поведением всего Function App
- Для мониторинга рекомендуется Application Insights


[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-functions/2-azure-functions-overview)
