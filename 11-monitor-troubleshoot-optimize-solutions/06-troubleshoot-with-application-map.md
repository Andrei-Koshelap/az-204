# Диагностика приложений с помощью Application Map

## Обзор

Application Map — это инструмент визуализации распределённой архитектуры приложения, который помогает быстро выявлять:

- узкие места производительности
- точки повышенного количества ошибок
- деградацию внешних зависимостей
- проблемные сервисы в цепочке вызовов

Он особенно важен при работе с микросервисами и распределёнными системами.

---

## Что такое Application Map?

Application Map автоматически обнаруживает и отображает топологию приложения, отслеживая HTTP-зависимости между сервисами, где установлен Application Insights SDK.

### Как это происходит:

- SDK добавляет correlation ID к каждому запросу
- Входящие запросы фиксируются как **Requests**
- Исходящие вызовы фиксируются как **Dependencies**
- Сервисы группируются по свойству `cloud_RoleName`
- На основе этих данных строится карта взаимодействий

---

## Что отображается на карте

- Узлы (сервисы, базы данных, внешние API)
- Связи между ними
- Частота вызовов
- Время ответа
- Процент ошибок
- Цветовая индикация состояния (здоров / деградирует / ошибка)

---

## Когда использовать

- При инциденте в production
- Для поиска bottleneck’ов
- Для анализа цепочки вызовов (end-to-end flow)
- При переходе к микросервисной архитектуре
- Для понимания реальной топологии системы

---

## Важно для AZ-204

Если требуется:

- Визуализировать архитектуру → Application Map
- Найти проблемный сервис → Application Map
- Отследить цепочку запроса → Distributed Tracing + Application Map

Главная цель — быстро определить компонент, вызывающий сбой или задержку.
```
┌──────────────────── APPLICATION MAP ────────────────────────┐
│                                                               │
│  Users → [Frontend] → [API Gateway] → [Services] → [Data]   │
│                                                               │
│  Each node shows:                                            │
│  • Request rate                                              │
│  • Response time (avg, p95)                                  │
│  • Failure rate                                              │
│  • Health status (color-coded)                               │
│                                                               │
│  Click any component for:                                    │
│  • Detailed metrics                                          │
│  • Failed requests                                           │
│  • Performance investigation                                 │
│  • Sample transactions                                       │
└───────────────────────────────────────────────────────────────┘
```

## Обнаружение компонентов (Component Discovery)

Application Map автоматически обнаруживает компоненты системы на основе телеметрии.

### Как происходит обнаружение:

1. **HTTP Dependency Tracking**  
   Отслеживаются вызовы между сервисами (Requests → Dependencies).  
   Если один сервис вызывает другой по HTTP, создаётся связь на карте.

2. **Cloud Role Name (`cloud_RoleName`)**  
   Телеметрия группируется по имени сервиса.  
   Каждый уникальный `cloud_RoleName` отображается как отдельный узел.

3. **Correlation IDs**  
   Связывают связанные операции в одну цепочку выполнения (trace).  
   Позволяют корректно отобразить направление вызовов.

---

## Что важно понимать

- Если SDK установлен не на всех сервисах, карта будет неполной.
- Внешние зависимости (SQL, Redis, HTTP API) отображаются автоматически.
- Для корректной работы необходима передача trace context между сервисами.

---

## Практический вывод

Чтобы Application Map работала корректно:

- SDK должен быть установлен на всех компонентах
- `cloud_RoleName` должен быть настроен корректно
- Correlation должен передаваться между сервисами

---

## Для AZ-204

Если в вопросе требуется:

- Автоматическое обнаружение сервисов → Application Map
- Группировка по сервисам → `cloud_RoleName`
- Связать операции в одну цепочку → Correlation ID

Главное — понимать, что карта строится на основе Requests + Dependencies + Correlation.

### Setting Cloud Role Name

The `cloud_RoleName` property determines how services appear on the map.

**.NET Configuration:**
```csharp
// Telemetry initializer
public class CloudRoleNameInitializer : ITelemetryInitializer
{
    public void Initialize(ITelemetry telemetry)
    {
        telemetry.Context.Cloud.RoleName = "OrderService";
    }
}

// Register in Program.cs
builder.Services.AddSingleton<ITelemetryInitializer, CloudRoleNameInitializer>();
```

**Node.js:**
```javascript
appInsights.defaultClient.context.tags[appInsights.defaultClient.context.keys.cloudRole] = "PaymentService";
```

**Python:**
```python
from opencensus.trace.span_context import SpanContext
from opencensus.trace.tracer import Tracer

def callback_function(envelope):
    envelope.tags['ai.cloud.role'] = 'InventoryService'
    return True
```

## Using Application Map

### Accessing Application Map

```
Azure Portal → Application Insights → Investigate → Application Map
```

### Visual Indicators

**Health Status Colors:**
- 🟢 Green: Healthy (< 5% failures, fast response)
- 🟡 Yellow: Warning (5-10% failures or slow response)
- 🔴 Red: Critical (> 10% failures or very slow)

**Component Information:**
```
┌─────────────────────────┐
│    OrderService         │
│    ─────────────        │
│    1,247 req/sec        │
│    Avg: 180ms           │
│    Failures: 2.3%  🟡   │
└─────────────────────────┘
```

### Investigating Performance Issues

**Scenario:** Slow checkout process

**Step 1: Identify Bottleneck**
```
Application Map shows:

Users
  └─> Frontend (150ms) ✅
      └─> API Gateway (220ms) ✅
          └─> Order Service (180ms) ✅
              ├─> Inventory (45ms) ✅
              ├─> Payment (1.2s) 🔴 SLOW
              └─> Notification (120ms) ✅
```

**Step 2: Click Payment Service**
```
PAYMENT SERVICE DETAILS
═══════════════════════
Request Rate:    458/sec
Avg Duration:    1,240ms ⚠️
P95 Duration:    2,800ms
Failure Rate:    0.8%

Top Operations:
• POST /charge        1,180ms (avg)
• GET /methods          45ms (avg)

Dependencies:
• Stripe API          1,150ms (avg) 🔴 ROOT CAUSE
• Redis Cache           12ms (avg) ✅
```

**Step 3: Investigate Dependency**
```kusto
// Query Stripe API performance
dependencies
| where name contains "stripe.com"
| where timestamp > ago(1h)
| summarize 
    count(),
    avg(duration),
    percentile(duration, 95)
    by bin(timestamp, 5m)
| render timechart
```

## Common Patterns

### Microservices Architecture

```
Users
  │
  ▼
┌─────────────┐
│   Frontend  │  React SPA
└──────┬──────┘
       │
       ▼
┌─────────────┐
│ API Gateway │  ASP.NET Core
└──────┬──────┘
       │
       ├──────────────┬──────────────┬──────────────┐
       ▼              ▼              ▼              ▼
  ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐
  │ Orders │    │Inventory│   │Payment │    │ Email  │
  │Service │    │Service  │   │Service │    │Service │
  └────┬───┘    └────┬───┘    └────┬───┘    └────┬───┘
       │             │              │              │
       ▼             ▼              ▼              ▼
  ┌────────┐    ┌────────┐    ┌────────┐    ┌────────┐
  │   SQL  │    │ Cosmos │    │Stripe  │    │SendGrid│
  │Database│    │   DB   │    │  API   │    │  API   │
  └────────┘    └────────┘    └────────┘    └────────┘
```

### Serverless Architecture

```
Users
  │
  ▼
┌──────────────────┐
│  Static Web App  │
└────────┬─────────┘
         │
         ▼
┌──────────────────┐
│ API Management   │
└────────┬─────────┘
         │
         ├─────────────┬─────────────┬─────────────┐
         ▼             ▼             ▼             ▼
    ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
    │Function │  │Function │  │Function │  │ Logic   │
    │Orders   │  │Products │  │Users    │  │ App     │
    └────┬────┘  └────┬────┘  └────┬────┘  └────┬────┘
         │            │            │            │
         ▼            ▼            ▼            ▼
    ┌─────────┐  ┌─────────┐  ┌─────────┐  ┌─────────┐
    │Cosmos DB│  │Cosmos DB│  │   SQL   │  │  Blob   │
    └─────────┘  └─────────┘  └─────────┘  └─────────┘
```

## Troubleshooting Scenarios

### Scenario 1: High Failure Rate

**Problem:** 15% of checkout requests failing

**Investigation Steps:**

1. **Check Application Map**
```
Payment Service: 15.2% failures 🔴
```

2. **Click Component → View Failures**
```
Top Failing Operations:
• POST /api/payments/charge  (94% of failures)

Exception Types:
• HttpRequestException: 87 occurrences
  "The operation has timed out"
```

3. **Query Dependencies**
```kusto
dependencies
| where timestamp > ago(1h)
| where success == false
| where target contains "stripe"
| summarize count() by resultCode
```

**Result:**
```
resultCode    count
──────────    ─────
Timeout        89
502            12
```

**Root Cause:** Stripe API timeouts
**Solution:** Implement retry logic with exponential backoff

### Scenario 2: Slow Response Times

**Problem:** P95 response time degraded from 500ms to 2.5s

**Investigation:**

1. **Application Map → Identify Slow Component**
```
Order Service: 2.3s avg (was 180ms) 🔴
```

2. **Check Dependencies**
```
SQL Database:  2.1s avg (was 28ms) 🔴
Cosmos DB:     15ms avg ✅
Redis Cache:   8ms avg ✅
```

3. **SQL Performance Query**
```kusto
dependencies
| where type == "SQL"
| where timestamp > ago(1h)
| where duration > 2000
| summarize count() by data
| order by count_ desc
```

**Result:**
```
Query: SELECT * FROM Orders WHERE CustomerId = @id
Count: 1,247
```

**Root Cause:** Missing index on CustomerId
**Solution:** Add database index

### Scenario 3: Cascading Failures

**Problem:** Frontend errors increasing

**Investigation:**

Application Map shows:
```
Frontend (5% failures) 🟡
  └─> API Gateway (8% failures) 🟡
      └─> Auth Service (45% failures) 🔴 ROOT CAUSE
          └─> Redis Cache (OFFLINE) 🔴
```

**Root Cause:** Redis cache failure cascading to Auth Service
**Solution:** Implement circuit breaker pattern

## Advanced Features

### Filtering by Time Range

```
Application Map → Time range selector
• Last 30 minutes
• Last hour
• Last 24 hours
• Custom range
```

### Filtering by Operation

Show only specific operations:
```kusto
// Filter to checkout operations only
requests
| where name contains "checkout"
| where timestamp > ago(1h)
```

### Multi-Resource Maps

View dependencies across multiple Application Insights resources:

```
Settings → Properties → Composite Application Map
• Enable cross-resource queries
• Select related resources
```

## Correlation and Distributed Tracing

Application Map relies on distributed tracing via correlation IDs.

**Request Flow:**
```
Request ID: 4bf92f3577b34da6a3ce929d0e0e4736

Frontend generates trace ID
  │
  ├─> HTTP Request to API Gateway
  │   Header: traceparent: 00-4bf92f3577b34da6-span1-01
  │
  API Gateway receives and propagates
  │
  ├─> HTTP Request to Order Service
  │   Header: traceparent: 00-4bf92f3577b34da6-span2-01
  │
  Order Service receives and propagates
  │
  └─> SQL Database query
      Linked by operation_Id: 4bf92f3577b34da6
```

**Query Correlated Telemetry:**
```kusto
let traceId = "4bf92f3577b34da6a3ce929d0e0e4736";
union requests, dependencies, exceptions, traces
| where operation_Id == traceId
| project timestamp, itemType, name, duration, success
| order by timestamp asc
```

## Лучшие практики

✅ **Настройте `cloud_RoleName` для всех сервисов**  
Это обеспечивает корректную группировку компонентов на карте.

✅ **Установите SDK на все компоненты**  
Иначе карта будет неполной и не покажет реальные зависимости.

✅ **Проверяйте Application Map регулярно**  
Это помогает вовремя замечать новые ошибки или деградацию производительности.

✅ **Используйте фильтр временного диапазона (Time range)**  
Позволяет анализировать конкретный инцидент или период времени.

✅ **Переходите к детальным метрикам (Click-through)**  
Для проведения root cause analysis.

✅ **Включите distributed tracing**  
Особенно важно для микросервисной архитектуры.

✅ **Проверяйте изменения топологии после деплоя**  
Новые сервисы или зависимости должны корректно отображаться на карте.

---

## Основные выводы

✅ **Application Map** автоматически визуализирует распределённую архитектуру приложения

✅ **Цветовая индикация узлов** показывает состояние:
- 🟢 Зелёный — здоров
- 🟡 Жёлтый — предупреждение
- 🔴 Красный — критическая проблема

✅ **`cloud_RoleName`** определяет группировку компонентов

✅ **Distributed tracing** связывает операции между сервисами

✅ **Click-through** позволяет перейти к детальному анализу ошибок и метрик

✅ **Лучше всего подходит для** диагностики микросервисов и поиска узких мест

---

## Советы для экзамена AZ-204

💡 **Application Map показывает распределённую топологию**  
Используется для диагностики multi-service приложений.

💡 **`cloud_RoleName` — ключевой параметр группировки**

💡 **Автоматическое обнаружение** происходит через HTTP dependency tracking

💡 **Цветовая индикация состояния**:  
Green — healthy  
Yellow — warning  
Red — critical

💡 **Используется вместе с distributed tracing**  
Для анализа полного пути запроса (end-to-end flow).

---

### Экзаменационный акцент

Если в вопросе требуется:

- Визуализировать взаимодействие сервисов → Application Map
- Найти bottleneck → Application Map
- Проанализировать цепочку запроса → Distributed Tracing + Application Map

Главная цель — быстро определить проблемный компонент в распределённой системе.

Premium поддерживает:

Redis clustering (шардинг)

RDB persistence

AOF persistence

Большие объёмы данных

Standard этого не поддерживает.


📌 Как работают списки (Lists) в Redis

Redis поддерживает структуру данных List — это упорядоченная коллекция элементов.

Есть две основные команды добавления:

🔹 LPUSH

Добавляет элемент в начало списка (left side)

LPUSH mylist "value1"
🔹 RPUSH ✅

Добавляет элемент в конец списка (right side)

RPUSH mylist "value2"

Если ключ не существует — Redis создаёт список автоматически.


IDatabase — это основной интерфейс для работы с Redis.

🔎 Что он делает

Он предоставляет методы, которые являются C#-обёрткой над Redis-командами:

Redis CLI	C# через IDatabase
SET	db.StringSet(...)
GET	db.StringGet(...)
RPUSH	db.ListRightPush(...)
SADD	db.SetAdd(...)

Все методы работают с типами:

RedisKey
RedisValue

Border Gateway Protocol (BGP) — это протокол маршрутизации между автономными системами (AS) в интернете.

Он:

обменивается маршрутами между провайдерами

выбирает лучший путь к сети назначения

основывается на атрибутах маршрута (в том числе AS Path)

---

**📚 Further Reading:**
- [Application Map](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-map)
- [Distributed tracing](https://learn.microsoft.com/en-us/azure/azure-monitor/app/distributed-tracing)
