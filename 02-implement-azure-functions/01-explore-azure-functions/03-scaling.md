# Azure Functions Scaling (Масштабирование Azure Functions)

## Key Concepts (Ключевые понятия)

- **Scale controller** — компонент, который принимает решение о масштабировании
- **Event-driven scaling** — масштабирование на основе событий (глубина очереди, поток событий и т.д.)
- **Per-function app scaling** — в Consumption масштабируется весь Function App
- **Per-function scaling** — в Flex Consumption масштабируются отдельные функции
- **Instance** — отдельная VM с Functions Host и выполняемыми функциями

> 💡 Масштабирование происходит автоматически (в serverless-планах).

---

## How Scaling Works (Как работает масштабирование)

### Scale Controller (Контроллер масштабирования)

Центральный компонент, который анализирует нагрузку и управляет масштабированием.

### Что делает Scale Controller:

1️⃣ **Мониторит источники триггеров**
- Глубина очереди
- Скорость поступления событий
- HTTP-нагрузка

2️⃣ **Использует эвристики**
- Определяет, нужно ли масштабироваться
- Оценивает backlog и throughput

3️⃣ **Добавляет или удаляет инстансы**
- Scale-out при росте нагрузки
- Scale-in при снижении

4️⃣ **Использует разную логику для разных триггеров**
- Queue trigger — по глубине очереди
- Event Hub — по количеству партиций
- HTTP — по входящим запросам

---

## Важно для AZ-204

- Масштабирование происходит автоматически в Consumption, Flex, Premium
- Dedicated требует ручной настройки autoscale
- Flex Consumption масштабирует функции отдельно
- Consumption масштабирует весь Function App целиком
- Scale controller принимает решения на основе метрик триггера

### Scaling Process
```
1. Events arrive (messages, requests, etc.)
   ↓
2. Scale controller evaluates trigger queue depth
   ↓
3. Decision: Scale out, scale in, or maintain
   ↓
4. New instance provisioned (if scale out)
   ↓
5. Functions host starts on new instance
   ↓
6. Trigger bindings registered
   ↓
7. Function code loaded and ready
   ↓
8. Instance starts processing events
```

**Duration**:  
Новый instance обычно запускается за несколько секунд,  
но **cold start** может добавить дополнительную задержку.

---

# Scaling Behavior by Plan (Поведение масштабирования по планам)

## Consumption Plan

### Characteristics (Характеристики)

- ⚡ Автоматическое **event-driven** масштабирование
- 📦 Масштабируется весь **Function App**
- 🔄 Инстансы динамически добавляются и удаляются
- 📈 **Scale-out** при росте нагрузки
- 💤 **Scale-in до нуля** при отсутствии событий

> 💡 Consumption может полностью остановить инстансы при простое.

---

### Max Instances (Максимальное число инстансов)

| Platform | Max Instances | Limit Type |
|----------|---------------|------------|
| **Windows** | 200 | На Function App |
| **Linux** | 100 | На Function App |

⚠️ Linux также имеет ограничение на скорость масштабирования в пределах подписки.

---

## Важно для AZ-204

- Consumption масштабируется автоматически
- Возможен cold start при первом запросе
- Масштабирование происходит на уровне Function App
- Scale-in может уменьшить количество инстансов до 0

⚠️ **Linux limit**: 500 instances/subscription/hour during scale-out

#### Scaling Example
```
Time    Events/sec   Instances   Reason
────────────────────────────────────────
00:00   0            0           Idle, scaled to zero
00:05   50           1           First event, cold start
00:10   200          3           High load, scale out
00:15   500          8           Continued growth
00:20   100          5           Load decreased, scale in
00:30   0            0           Idle, scale to zero
```

## Flex Consumption Plan

### Characteristics (Характеристики)

- 🎯 **Per-function scaling** — каждая функция масштабируется независимо
- 📊 Более предсказуемое поведение масштабирования
- ⚙️ **Configurable concurrency** — можно задать количество одновременных выполнений на instance для каждой функции
- 🔥 **Always-ready instances** — возможность заранее подготовленных инстансов (снижение cold start)

> 💡 В отличие от обычного Consumption, масштабирование происходит не на уровне всего Function App, а на уровне отдельных функций.

---

### Max Instances (Максимальное число инстансов)

- Жёсткого лимита по количеству инстансов нет
- Ограничение определяется **общим объёмом доступной памяти в регионе**

> ⚠️ Фактический предел зависит от региональных ресурсов и выбранной конфигурации памяти.


#### Per-Function Scaling
```
Function App with 3 functions:

Function A (HTTP): 20 instances (high traffic)
Function B (Queue): 5 instances (moderate traffic)  
Function C (Timer): 1 instance (scheduled)

Total: 26 instances for this function app
```

## Premium Plan

### Characteristics (Характеристики)

- 🔥 **Pre-warmed workers** — инстансы всегда готовы (нет cold start)
- ⚡ **Event-driven scaling** — автоматическое масштабирование по нагрузке
- 🖥 Более мощные ресурсы на instance (CPU и память)

> 💡 Premium сочетает serverless-масштабирование и гарантированную готовность инстансов.

---

### Max Instances (Максимальное число инстансов)

| Platform | Max Instances | Notes |
|----------|---------------|-------|
| **Windows** | 100 | На уровень плана |
| **Linux** | 20–100 | На уровень плана, зависит от региона |

⚠️ Лимит применяется ко всему плану, а не к отдельной Function App.


#### Scaling Example with Pre-warmed
```
Configuration:
- Min instances: 3 (always running)
- Max instances: 10

Time    Events/sec   Instances   Status
─────────────────────────────────────────────
00:00   0            3           Min always running
00:05   100          3           Handled by pre-warmed
00:10   500          6           Scale out (no cold start)
00:15   1000         10          Max reached
00:20   200          5           Scale in (keep above min)
00:30   0            3           Back to minimum
```

## Dedicated Plan (App Service)

### Characteristics (Характеристики)

- ⚙️ **Manual or autoscale** — масштабирование вручную или через autoscale rules
- 📊 **Not event-driven** — используется механизм масштабирования App Service
- 🔗 Может быть общим с Web Apps (если размещены в одном плане)

> 💡 Масштабирование основано на метриках CPU, памяти и правилах autoscale, а не на глубине очереди триггеров.

---

### Max Instances (Максимальное число инстансов)

| Environment | Max Instances |
|-------------|---------------|
| **Standard / Premium** | 10–30 |
| **App Service Environment (ASE)** | 100 |

⚠️ Лимиты применяются на уровне App Service Plan.


#### Autoscale Configuration
```bash
# Create autoscale rule for function app on App Service Plan
az monitor autoscale create \
  --resource-group <rg-name> \
  --resource <plan-id> \
  --name FunctionAutoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Add scale-out rule
az monitor autoscale rule create \
  --autoscale-name FunctionAutoscale \
  --resource-group <rg-name> \
  --condition "Percentage CPU > 70 avg 5m" \
  --scale out 1
```

## Container Apps

### Characteristics (Характеристики)

- ⚡ **Event-driven scaling** на уровне функций
- ⚙️ Настраиваемое максимальное количество реплик
- 📈 Масштабирование на основе **KEDA** (Kubernetes Event-Driven Autoscaling)

> 💡 KEDA анализирует события (очереди, Event Hub, HTTP и др.) и управляет количеством реплик.

---

### Max Instances (Максимальное число инстансов)

**10–300** инстансов  
(настраивается, зависит от квоты CPU cores)

---

# Scaling Limits Summary (Сводка лимитов масштабирования)

## Instance Limits by Plan

| Plan | Windows | Linux | Scope |
|------|----------|--------|--------|
| **Consumption** | 200 | 100 | На Function App |
| **Flex Consumption** | Ограничено памятью | Ограничено памятью | На регион |
| **Premium** | 100 | 20–100 | На план |
| **Dedicated** | 10–30 (100 ASE) | 10–30 (100 ASE) | На план |
| **Container Apps** | 10–300 | 10–300 | Настраиваемо |

---

## Regional Limits (Региональные ограничения)

- **Consumption (Linux)**: максимум 500 инстансов на подписку в час при scale-out
- Количество Function Apps в плане: не ограничено  
  (делят ресурсы одного плана)

---

# Scaling Behavior by Trigger Type (Масштабирование по типу триггера)

---

## HTTP Trigger

- Новый instance добавляется постепенно (по одному)
- Ограничение 200 (Windows) / 100 (Linux)
- Нет глубины очереди — масштабирование по скорости запросов

---

## Queue Trigger (Storage Queue)

- Использует batch processing
- Масштабируется по:
    - длине очереди
    - возрасту сообщений
- Цель — минимизировать backlog

**Heuristic:**

---
Queue messages / target messages per instance

## Service Bus Trigger

- Масштабирование по количеству сообщений
- Учитывается lock duration
- Цель — минимизировать задержку обработки

---

## Timer Trigger

- Работает в одном instance
- Не масштабируется
- Использует singleton lock для предотвращения дублей

---

## Blob Trigger

- По умолчанию polling
- Можно использовать Event Grid для ускоренного обнаружения
- Масштабируется по количеству blob-файлов

---

## Event Hub Trigger

- Максимум instance = количество partition
- Использует checkpoint для отслеживания прогресса
- Масштабируется по partition (1 instance на partition максимум)

---

# Cold Start (Холодный старт)

## What Is Cold Start?

Задержка при масштабировании с 0 до первого instance:

- Инициализация Functions Host
- Загрузка runtime
- Регистрация trigger bindings
- Загрузка кода

> 💡 Наиболее заметен в Consumption и Container Apps  
> (устраняется в Premium благодаря pre-warmed instances).

```
Event arrives → Allocate infrastructure → Start host → Load code → Execute function
                 ↓                         ↓              ↓            ↓
                500ms                    2-10s          1-5s         <1s

Total cold start: 3-15 seconds (varies by language, dependencies)
```

## Cold Start by Plan (Cold Start по планам)

| Plan | Cold Start? | Mitigation |
|------|-------------|------------|
| **Consumption** | ✅ Yes | Keep warm через Timer-trigger или перейти на Premium |
| **Flex Consumption** | ✅ Yes | Использовать Always-ready instances |
| **Premium** | ❌ No | Pre-warmed workers (инстансы всегда готовы) |
| **Dedicated** | ❌ No | Включить Always On |
| **Container Apps** | ✅ Yes | Установить Min replicas > 0 |

---

### Пояснения

- **Consumption** — масштабируется до нуля, поэтому возможен cold start.
- **Flex Consumption** — можно заранее держать подготовленные инстансы.
- **Premium** — cold start отсутствует благодаря pre-warmed workers.
- **Dedicated** — при включённом Always On инстанс постоянно работает.
- **Container Apps** — если min replicas = 0, возможен cold start.

---

## Важно для AZ-204

- Cold start характерен для serverless-планов с scale-to-zero.
- Premium и Dedicated устраняют cold start.
- Always-ready instances и Min replicas — ключевые механизмы снижения задержки.


### Reducing Cold Start

#### Method 1: Premium Plan
```bash
# Create Premium plan (eliminates cold start)
az functionapp plan create \
  --name <plan-name> \
  --resource-group <rg-name> \
  --sku EP1 \
  --is-linux \
  --min-burst 3  # Pre-warmed instances
```

#### Method 2: Keep Warm (Consumption)
```csharp
// Timer function to keep app warm
[FunctionName("KeepWarm")]
public static void Run(
    [TimerTrigger("0 */5 * * * *")] TimerInfo timer,  // Every 5 minutes
    ILogger log)
{
    log.LogInformation("Keep-warm ping executed");
}
```

⚠️ **Note**: Keep-warm uses executions (free tier: 1M/month)

#### Method 3: Always-Ready Instances (Flex Consumption)
```bash
# Configure always-ready instances
az functionapp update \
  --name <app-name> \
  --resource-group <rg-name> \
  --always-ready-instances 2
```

## Scale-Out Strategies

### Gradual Scale-Out
```
Event spike detected:
  Minute 0: 1 instance
  Minute 1: 2 instances (+1)
  Minute 2: 4 instances (+2)
  Minute 3: 8 instances (+4)
  Minute 4: 16 instances (+8)
```

**Pattern**: Exponential growth, but controlled

### Queue-Based Scaling
```
Queue trigger scaling logic:

Queue length: 1000 messages
Target per instance: 100 messages
Desired instances: 1000 / 100 = 10

Current: 3 instances
Action: Scale out to 10 instances
```

### HTTP Scaling
```
HTTP trigger scaling logic:

Current instances: 5
Max concurrent requests per instance: 100
Total capacity: 5 × 100 = 500 requests

Incoming rate: 800 requests/sec
Action: Scale out to 8 instances (800 / 100)
```

## Monitoring Scaling Activity

### Application Insights Metrics
```
Metrics to track:
- Function execution count (per instance)
- Instance count
- Memory usage
- CPU percentage
- Request rate
- Queue length
```

### Portal View
```
Function App → Overview → Metrics
├── Function Execution Count
├── Function Execution Units
├── Instance Count
└── Response Time
```

### CLI Query
```bash
# Get instance count over time
az monitor metrics list \
  --resource <function-app-id> \
  --metric "FunctionExecutionCount" \
  --start-time 2024-01-01T00:00:00Z \
  --end-time 2024-01-01T23:59:59Z \
  --interval PT1M
```

### Log Analytics Query
```kusto
// Function scaling events
AzureMetrics
| where TimeGenerated > ago(1h)
| where MetricName == "InstanceCount"
| project TimeGenerated, Average, Maximum
| render timechart
```

# Best Practices (Лучшие практики)

---

## 1️⃣ Choose the Right Plan (Выбирайте правильный план)

✅ **Consumption**
- Нерегулярная нагрузка
- Чувствительность к стоимости

✅ **Premium**
- Частые вызовы
- Требуется стабильная производительность
- Нельзя допустить cold start

✅ **Dedicated**
- Предсказуемая нагрузка
- Уже используется App Service Plan

> 💡 Выбор плана напрямую влияет на масштабирование, стоимость и latency.

---

## 2️⃣ Optimize Function Code (Оптимизируйте код функции)

- ⚡ **Держите функции быстрыми**  
  Чем меньше время выполнения — тем лучше масштабирование

- 🔄 Используйте **async/await**  
  Неблокирующий код повышает throughput

- 🔌 **Connection pooling**  
  Повторно используйте:
    - HTTP clients
    - Database connections

- 🧩 Делайте функции **stateless**  
  Не храните состояние локально — масштабирование может создать новые инстансы

---

## Важно для AZ-204

- Масштабирование эффективно только при правильно написанном коде
- Долгие блокирующие операции ухудшают масштабируемость
- Повторное создание HTTP-клиентов — частая ошибка
- Stateless-дизайн — основа serverless-архитектуры


### 3. Configure Appropriate Limits
```json
// host.json
{
  "version": "2.0",
  "extensions": {
    "http": {
      "maxConcurrentRequests": 100,
      "maxOutstandingRequests": 200
    },
    "queues": {
      "batchSize": 16,
      "maxDequeueCount": 5,
      "newBatchThreshold": 8
    }
  }
}
```

### 4. Monitor and Alert
```bash
# Create alert for high instance count
az monitor metrics alert create \
  --name HighInstanceCount \
  --resource-group <rg-name> \
  --scopes <function-app-id> \
  --condition "max InstanceCount > 50" \
  --description "Function app scaled beyond 50 instances"
```

### 5️⃣ Test Scaling Behavior (Тестируйте масштабирование)

- 🧪 **Load test перед production**  
  Проверьте, как приложение масштабируется под реальной нагрузкой

- 📊 **Мониторинг в пиковые периоды**  
  Отслеживайте:
    - Instance count
    - Execution time
    - Failures
    - Queue length

- ⚙️ **Корректируйте конфигурацию**  
  Настраивайте:
    - Concurrency
    - Min/Max instances
    - Always-ready instances
    - Timeout

> 💡 Масштабирование должно быть проверено заранее, а не впервые в продакшене.

---

## Важно для AZ-204

- Проверяйте поведение scale-out и scale-in
- Следите за cold start latency
- Используйте Application Insights для анализа нагрузки
- Тюнинг масштабирования — итеративный процесс

## Common Scaling Patterns

### Pattern 1: Burst Traffic (HTTP)
```
Best plan: Premium or Flex Consumption
- Pre-warmed instances handle initial burst
- Auto-scale handles sustained load
- No cold start delays
```

### Pattern 2: Queue Processing
```
Best plan: Consumption or Premium
- Scales based on queue depth
- Cost-effective for variable load
- Monitor queue length metrics
```

### Pattern 3: Scheduled Jobs
```
Best plan: Consumption
- Timer triggers don't scale out
- Single instance sufficient
- Minimal cost (only during execution)
```

### Pattern 4: Event Stream Processing
```
Best plan: Premium or Dedicated
- Consistent throughput needed
- Partition-based scaling (Event Hub)
- Predictable performance
```

## Critical Notes (Критически важные моменты)

- 💡 **Event-driven масштабирование** — Consumption и Premium масштабируются автоматически
- ⚠️ **Cold start** возможен в:
    - Consumption
    - Flex Consumption
    - Container Apps (если min replicas = 0)
- 🎯 **Максимум инстансов (Consumption)**:
    - Windows — 200
    - Linux — 100
- 📊 **Scale controller** — центральный компонент, который анализирует триггеры
- ✅ В Consumption масштабируется **весь Function App**
- 🔄 В Flex Consumption масштабируются **отдельные функции**
- ⏱️ Scale-out происходит постепенно (не мгновенно)
- 🔒 **Timer trigger** — singleton, всегда один instance

---

## Exam Tips (Советы для экзамена)

- **Consumption**:
    - Event-driven
    - До 200 (Windows) / 100 (Linux) инстансов на Function App
- **Flex Consumption**:
    - Масштабирование на уровне функции
    - Ограничение по памяти региона
- **Premium**:
    - Pre-warmed workers (нет cold start)
    - До 100 (Windows) / 20–100 (Linux) инстансов на план
- **Dedicated**:
    - Ручное или autoscale
    - 10–30 инстансов (до 100 с ASE)

---

### Trigger Scaling Rules (Что важно запомнить)

- Scale controller анализирует источники триггеров
- Cold start = задержка при масштабировании с 0
- **HTTP trigger** — масштабируется по количеству запросов
- **Queue trigger** — по длине очереди и возрасту сообщений
- **Timer trigger** — не масштабируется (singleton)
- **Event Hub** — максимум инстансов = количество partition
- Linux Consumption: 500 инстансов на подписку в час при scale-out
- Premium устраняет cold start благодаря always-ready instances
- Разница:
    - Consumption → per-function app scaling
    - Flex → per-function scaling


[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-functions/4-scale-azure-functions)
