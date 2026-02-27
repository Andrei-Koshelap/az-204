# Итоги и подготовка к экзамену

## Итоги курса

Поздравляем! Вы завершили **Тему 11: Monitor, Troubleshoot, and Optimize Azure Solutions** — финальный раздел подготовки к экзамену AZ-204.

В этом модуле вы изучили ключевые инструменты мониторинга и диагностики в Azure.

---

## Основные концепции

### ✅ Обзор Application Insights

- Расширение Azure Monitor для APM (Application Performance Monitoring)
- Поддерживает проактивный и реактивный мониторинг
- Автоматически собирает метрики, логи и трассировки

---

### ✅ Типы телеметрии

- **Metrics** — числовые временные ряды (быстро, предагрегировано)
- **Logs** — события с богатым контекстом (запросы через KQL)
- **Traces** — сквозное отслеживание запросов (correlation)

---

### ✅ Log-Based vs Standard Metrics

- **Log-based**: гибкие, зависят от sampling, медленнее
- **Standard**: быстрые, near real-time, не зависят от sampling

Использование:
- Standard → дашборды и алёрты
- Log-based → анализ и расследование

---

### ✅ Методы инструментирования

- **Autoinstrumentation** — без изменений кода (App Service, Functions)
- **Manual SDK** — полный контроль, кастомная телеметрия
- **OpenTelemetry** — vendor-neutral стандарт

---

### ✅ Availability Tests

- **Standard tests** — современный и рекомендуемый вариант
- **Custom TrackAvailability** — для сложных сценариев
- Несколько регионов уменьшают ложные срабатывания

---

### ✅ Application Map

- Визуализация распределённой архитектуры
- Цветовая индикация состояния
- Возможность drill-down
- Основан на `cloud_RoleName` и distributed tracing

---

### ✅ Мониторинг и анализ

- **Metrics Explorer** — real-time графики
- **KQL** — мощный анализ логов
- **Distributed Tracing** — end-to-end поток запроса
- **Alerts** — проактивные уведомления

---

## Матрица ключевых инструментов

| Функция | Назначение | Когда использовать | Важность на экзамене |
|----------|------------|-------------------|----------------------|
| **Live Metrics** | Мониторинг в реальном времени | Деплой, инциденты | Средняя |
| **Smart Detection** | AI-обнаружение аномалий | Автоматические алёрты | Средняя |
| **Application Map** | Визуализация топологии | Диагностика микросервисов | Высокая |
| **Availability Tests** | Проактивный мониторинг доступности | Контроль SLA | Высокая |
| **Metrics Explorer** | Real-time дашборды | Мониторинг производительности | Высокая |
| **Log Analytics (KQL)** | Глубокий анализ | Root cause investigation | Высокая |
| **Distributed Tracing** | Отслеживание цепочки вызовов | Отладка микросервисов | Средняя |
| **Workbooks** | Кастомные отчёты | Командные дашборды | Низкая |

---

## Финальный экзаменационный фокус (AZ-204)

Часто проверяются сценарии:

- Как быстро включить мониторинг? → Autoinstrumentation
- Как снизить стоимость? → Sampling + Standard metrics + GetMetric()
- Как найти причину сбоя? → KQL + Distributed Tracing
- Как настроить SLA-мониторинг? → Availability Tests
- Как визуализировать архитектуру? → Application Map
- Как настроить real-time алёрт? → Standard metrics

---

## Стратегия для экзамена

1. Определите цель вопроса:
 - Мониторинг?
 - Анализ?
 - Стоимость?
 - SLA?
 - Архитектура?

2. Выберите самый простой инструмент, который покрывает требования.

3. Помните ключевые различия:
 - Metrics → быстро
 - Logs → глубоко
 - Tracing → связать всё
 - Sampling → экономия
 - Standard metrics → быстрые алёрты

---

### Заключение

Мониторинг в Azure — это не один инструмент, а экосистема:

- Application Insights
- Azure Monitor
- Log Analytics
- Distributed tracing
- Availability tests

Умение правильно комбинировать их — ключ к успешной сдаче AZ-204 и к реальной работе в production.


## Decision Flowcharts

### When to Use Each Instrumentation Method

```
┌────────────────────────────────────────┐
│ Do you need custom business metrics?  │
└───────────┬─────────────┬──────────────┘
            │             │
           YES           NO
            │             │
            ▼             ▼
┌────────────────┐  ┌────────────────────┐
│  Manual SDK    │  │ Is it App Service, │
│  ─────────────│  │ Functions, or AKS? │
│  • Custom      │  └──────┬──────┬──────┘
│    events      │         │      │
│  • Custom      │        YES    NO
│    metrics     │         │      │
│  • Filtering   │         ▼      ▼
└────────────────┘  ┌──────────┐ ┌──────────┐
                    │ Autoinst │ │ Manual   │
                    │ rumentat │ │ SDK      │
                    │ ion ✅   │ │          │
                    └──────────┘ └──────────┘
```

### Choosing Between Metrics and Logs

```
┌─────────────────────────────────────┐
│     What's your use case?           │
└──────┬──────────────┬───────────────┘
       │              │
       ▼              ▼
  ┌─────────┐   ┌──────────┐
  │Dashboard│   │ Debug /  │
  │ Alert   │   │ Analysis │
  │ Real-   │   │ Custom   │
  │ time    │   │ Query    │
  └────┬────┘   └─────┬────┘
       │              │
       ▼              ▼
┌──────────────┐ ┌──────────────┐
│ STANDARD     │ │ LOG-BASED    │
│ METRICS ✅   │ │ METRICS ✅   │
│              │ │              │
│ • Fast       │ │ • Flexible   │
│ • Real-time  │ │ • Rich data  │
│ • No sampling│ │ • Ad-hoc     │
│   impact     │ │   queries    │
└──────────────┘ └──────────────┘
```

### Troubleshooting Performance Issues

```
Performance Issue Detected
           │
           ▼
┌────────────────────────────────┐
│ Start with Application Map     │
│ • Identify slow component      │
│ • Check dependency health      │
└───────────┬────────────────────┘
            │
            ▼
┌────────────────────────────────┐
│ Click slow component           │
│ • View detailed metrics        │
│ • Check recent failures        │
└───────────┬────────────────────┘
            │
            ▼
┌────────────────────────────────┐
│ Use KQL for deep dive          │
│ • Query slow operations        │
│ • Analyze percentiles          │
│ • Check dependencies           │
└───────────┬────────────────────┘
            │
            ▼
┌────────────────────────────────┐
│ Review distributed trace       │
│ • End-to-end request flow      │
│ • Identify bottleneck          │
│ • Check timing waterfall       │
└────────────────────────────────┘
```

## Чек-лист лучших практик

### Этап настройки (Setup Phase)

- ✅ Включить autoinstrumentation для App Service и Functions
- ✅ Настроить `cloud_RoleName` для всех сервисов
- ✅ Включить sampling (рекомендуется adaptive)
- ✅ Настроить availability tests из 5+ регионов
- ✅ Создать alert rules для критических метрик
- ✅ Настроить action groups для уведомлений

---

### Этап разработки (Development Phase)

- ✅ Использовать `GetMetric()` для кастомных метрик (не `TrackMetric()`)
- ✅ Отслеживать бизнес-события через `TrackEvent()`
- ✅ Реализовать фильтрацию телеметрии для снижения шума
- ✅ Передавать correlation headers для distributed tracing
- ✅ Добавлять кастомные свойства через telemetry initializers
- ✅ Тестировать мониторинг в dev-среде

---

### Этап эксплуатации (Operations Phase)

- ✅ Мониторить Live Metrics во время деплоя
- ✅ Ежедневно просматривать Application Map
- ✅ Оперативно реагировать на Smart Detection
- ✅ Использовать Workbooks для командных дашбордов
- ✅ Анализировать тренды через KQL
- ✅ Ежемесячно пересматривать пороги алёртов

---

### Оптимизация стоимости (Cost Optimization)

- ✅ Включить adaptive sampling (цель — ~5 элементов/сек)
- ✅ Исключить health-check endpoints
- ✅ Использовать `GetMetric()` для метрик с высокой частотой
- ✅ Установить daily cap при необходимости
- ✅ Настроить retention (по умолчанию 90 дней)
- ✅ Архивировать старые данные в Storage при необходимости

---

## Частые экзаменационные сценарии

### Сценарий 1: Включить мониторинг без изменений кода

**Вопрос:**  
У вас есть ASP.NET Core веб-приложение, развернутое в Azure App Service. Нужно включить мониторинг без изменений кода. Что делать?

### ✅ Правильный ответ:

Включить **Autoinstrumentation** в настройках App Service и подключить Application Insights через Configuration (Connection String).

### Почему:

- Не требует изменений кода
- Не требует повторной сборки приложения
- Самый простой и рекомендуемый способ

### Экзаменационный акцент:

Если в вопросе есть:
- App Service
- Azure Functions
- Требование "без изменений кода"

→ Почти всегда правильный ответ — **Autoinstrumentation**.

---

Если нужно, можем разобрать ещё 5–10 типовых сценариев в формате "вопрос → правильное решение → почему".

**Answer:** 
```bash
# Enable autoinstrumentation via App Service settings
az webapp config appsettings set \
  --name MyWebApp \
  --resource-group MyRG \
  --settings \
    APPLICATIONINSIGHTS_CONNECTION_STRING="<connection-string>" \
    ApplicationInsightsAgent_EXTENSION_VERSION="~3"
```

### Scenario 2: Track Custom Business Metrics

**Question:** You need to track order values in your e-commerce application. The solution must minimize data ingestion costs. What should you use?

**Answer:**
```csharp
// Use GetMetric() for preaggregated custom metrics
var orderValueMetric = telemetryClient.GetMetric("OrderValue");
orderValueMetric.TrackValue(149.99);
```

### Scenario 3: Alert on Performance Degradation

**Question:** You need to alert when average response time exceeds 500ms. The alert must evaluate within 1 minute. What should you use?

**Answer:**
```bash
# Create metric alert (standard metrics, fast evaluation)
az monitor metrics alert create \
  --name "High Response Time" \
  --condition "avg requests/duration > 500" \
  --window-size 5m \
  --evaluation-frequency 1m
```

### Scenario 4: Investigate Failed Requests

**Question:** Users report errors during checkout. You need to find all failed checkout requests with full details. What should you do?

**Answer:**
```kusto
// Use KQL for detailed log analysis
requests
| where name contains "checkout"
| where success == false
| where timestamp > ago(24h)
| project timestamp, url, resultCode, duration, user_AuthenticatedId
| order by timestamp desc
```

### Scenario 5: Monitor External Dependency Health

**Question:** Your application depends on an external payment API. You need to identify when this API is slow. What should you use?

**Answer:**
```
1. Check Application Map (shows dependency nodes)
2. Click dependency node to view metrics
3. Query slow dependencies:

dependencies
| where target contains "payment-api.com"
| where duration > 1000
| summarize count(), avg(duration) by bin(timestamp, 5m)
```

## Quick Reference Guide

### Essential Azure CLI Commands

```bash
# Create Application Insights
az monitor app-insights component create \
  --app MyApp \
  --location eastus \
  --resource-group MyRG \
  --application-type web

# Get connection string
az monitor app-insights component show \
  --app MyApp \
  --resource-group MyRG \
  --query connectionString -o tsv

# Enable App Service monitoring
az webapp config appsettings set \
  --name MyWebApp \
  --resource-group MyRG \
  --settings APPLICATIONINSIGHTS_CONNECTION_STRING="<string>"

# Create availability test
az monitor app-insights web-test create \
  --name "Homepage-Test" \
  --request-url "https://example.com" \
  --locations "us-ca-sjc-azr" "emea-nl-ams-azr" \
  --frequency 300

# Create metric alert
az monitor metrics alert create \
  --name "High Response Time" \
  --condition "avg requests/duration > 500" \
  --window-size 5m \
  --evaluation-frequency 1m
```

### Essential KQL Queries

```kusto
// Failed requests
requests
| where success == false
| summarize count() by name, resultCode

// Performance percentiles
requests
| summarize 
    p50 = percentile(duration, 50),
    p95 = percentile(duration, 95),
    p99 = percentile(duration, 99)

// Slow dependencies
dependencies
| where duration > 1000
| summarize count(), avg(duration) by target

// Exceptions with context
exceptions
| join kind=inner requests on operation_Id
| project timestamp, type, outerMessage, name, url

// Error rate over time
requests
| summarize 
    Total = count(),
    Failed = countif(success == false)
    by bin(timestamp, 5m)
| extend ErrorRate = 100.0 * Failed / Total

// Distributed trace
union requests, dependencies
| where operation_Id == "<trace-id>"
| project timestamp, itemType, name, duration
| order by timestamp asc
```
## Советы по подготовке к экзамену

### Ключевые темы для уверенного владения

### 🔥 Высокий приоритет (наиболее вероятны на экзамене)

1. ✅ Autoinstrumentation vs Manual SDK
2. ✅ Настройка Connection String
3. ✅ Standard metrics vs log-based metrics
4. ✅ Типы Availability Tests и их конфигурация
5. ✅ Использование Application Map
6. ✅ Основы KQL (where, summarize, join)
7. ✅ Настройка alert’ов (metric alerts vs log alerts)

---

### ⚡ Средний приоритет

8. ✅ `GetMetric()` vs `TrackMetric()`
9. ✅ Основы distributed tracing
10. ✅ Типы sampling и их влияние
11. ✅ Telemetry processors
12. ✅ Настройка `cloud_RoleName`
13. ✅ Интеграция OpenTelemetry

---

### 📌 Низкий приоритет

14. ✅ Создание Workbooks
15. ✅ Продвинутые функции KQL
16. ✅ Custom TrackAvailability tests

---

## Факты, которые нужно помнить

| Тема | Ключевые факты |
|------|----------------|
| **Connection String** | Заменяет instrumentation key (новый стандарт) |
| **Autoinstrumentation** | App Service, Functions, AKS (без изменений кода) |
| **Sampling** | Влияет только на log-based metrics |
| **GetMetric()** | Предагрегированная, экономичная, точная |
| **Standard Tests** | Частота 5 минут, 5+ регионов рекомендуется |
| **Application Map** | Требует `cloud_RoleName`, показывает распределённую топологию |
| **KQL** | summarize, where, project, order by, join |
| **Частота алёртов** | Metric: ~1 минута, Log: минимум ~5 минут |
| **Retention** | Standard metrics: 93 дня, Logs: 30–730 дней |
| **Smart Detection** | Автоматический ML-механизм, без настройки |

---

## Финальный чек-лист

Перед экзаменом убедитесь, что вы умеете:

- [ ] Создать Application Insights ресурс
- [ ] Включить autoinstrumentation для App Service
- [ ] Настроить connection string в app settings
- [ ] Различать standard и log-based metrics
- [ ] Писать базовые KQL-запросы (where, summarize)
- [ ] Настраивать availability tests из нескольких регионов
- [ ] Создавать metric и log alerts
- [ ] Использовать Application Map для диагностики
- [ ] Настраивать `cloud_RoleName`
- [ ] Отправлять custom events и metrics через SDK
- [ ] Понимать типы sampling и их влияние
- [ ] Интерпретировать distributed traces

---

## Следующие шаги

### Продолжение подготовки

1. Практика в Azure Free Tier
2. Прохождение пробных экзаменов AZ-204
3. Повтор всех 11 тем на Microsoft Learn
4. Развёртывание и мониторинг собственного приложения
5. Участие в Azure Developer сообществах

---

## Вы завершили весь путь AZ-204 🎉

Все 11 тем курса:

1. ✅ Azure App Service Web Apps
2. ✅ Azure Functions
3. ✅ Azure Blob Storage
4. ✅ Azure Cosmos DB
5. ✅ Контейнерные решения
6. ✅ Аутентификация и авторизация
7. ✅ Безопасные решения в Azure
8. ✅ API Management
9. ✅ Event-based решения
10. ✅ Message-based решения
11. ✅ **Мониторинг, диагностика и оптимизация решений**

---

## Итоговые выводы по всему курсу AZ-204

### Compute
App Service, Functions, Containers (ACI, ACA)

### Storage
Blob Storage, Cosmos DB, Queue Storage

### Security
Key Vault, Managed Identity, Azure AD

### Integration
API Management, Event Grid, Event Hubs, Service Bus

### Monitoring
Application Insights, Azure Monitor, Distributed Tracing

---

### Финальный совет

На экзамене всегда:

1. Определяйте цель задачи (мониторинг? безопасность? интеграция?).
2. Выбирайте самый простой и управляемый вариант.
3. Помните различия между похожими инструментами.
4. Учитывайте стоимость, масштабируемость и производительность.

You're now ready to take the **AZ-204: Developing Solutions for Microsoft Azure** certification exam!

---

**🎯 Good luck with your certification!**

В Application Insights для поиска узких мест (performance bottlenecks) анализируют:

время выполнения запросов

зависимости (Dependency calls)

распределение задержек (latency distribution)

Histogram:

показывает распределение времени отклика

помогает выявить медленные операции

позволяет увидеть, какие страницы или зависимости тормозят

Особенно важно анализировать:

Server response time

Dependency duration (SQL, HTTP, Redis и т.д.)


Как обычно ищут bottleneck
Открывают Performance → Requests
Сортируют по Duration
Анализируют зависимости (Dependencies)
 Смотрят распределение задержек (histogram)


Live Metrics Stream показывает:
🔹 Request rate (RPS)
🔹 Failure rate
🔹 Response times
🔹 CPU / Memory
🔹 Dependency failures
🔹 Incoming requests

Profiler
Анализирует performance на уровне кода
Не показывает failure counts в real-time

🔹 Smart Detection
AI-based аномалии
Не real-time мониторинг
Не real-time dashboard,
а автоматическое обнаружение проблем.

🔹 Snapshot Debugger
Делает snapshot при exception
Для debugging, не для live мониторинга

Logs (Log Analytics / KQL)
Полный доступ к данным через KQL (Kusto Query Language).
📌 Позволяет:
Фильтровать ошибки
Искать конкретные request
Делать кастомную аналитику
Создавать alert rules



**📚 Further Resources:**
- [Application Insights Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
- [Azure Monitor Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/)
- [KQL Reference](https://learn.microsoft.com/en-us/azure/data-explorer/kusto/query/)
- [AZ-204 Learning Path](https://learn.microsoft.com/en-us/training/paths/az-204-develop-solutions-that-use-azure-services/)
