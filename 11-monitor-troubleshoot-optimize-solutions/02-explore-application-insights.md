# Изучаем Azure Monitor и Application Insights

## Обзор

Application Insights — это расширение Azure Monitor, предоставляющее возможности **Application Performance Monitoring (APM)**.

Это комплексное решение для мониторинга, которое позволяет:

- понимать производительность приложений;
- выявлять узкие места;
- обнаруживать аномалии;
- находить ошибки до того, как они повлияют на пользователей.

---

## В этом разделе вы изучите

- Архитектуру Azure Monitor и типы данных
- Возможности Application Insights
- Как собирается и хранится телеметрия
- Ключевые концепции APM и инструменты мониторинга

---

# Azure Monitor: Основа мониторинга

Azure Monitor — это единая платформа мониторинга для всех ресурсов Azure.

Он предоставляет централизованное хранилище для:

- Метрик (Metrics)
- Логов (Logs)
- Трассировок (Traces)

---

## Что делает Azure Monitor

- Собирает телеметрию из ресурсов Azure
- Обрабатывает и агрегирует данные
- Хранит данные в Metrics store и Log Analytics
- Позволяет анализировать информацию через KQL
- Поддерживает алерты и автоматические реакции

---

## Архитектурная модель

1. Ресурсы Azure и приложения генерируют телеметрию
2. Данные поступают в Azure Monitor
3. Сохраняются в соответствующих хранилищах
4. Анализируются через инструменты (Metrics Explorer, Log Analytics, Workbooks)
5. Используются для алертов и автоматизации

---

## Важно для AZ-204

Нужно чётко понимать:

- Azure Monitor — центральная платформа
- Application Insights — APM-компонент внутри неё
- Мониторинг охватывает метрики, логи и трассировки
- Данные можно анализировать через KQL

Экзамен проверяет архитектурное понимание, а не только знание интерфейса портала.

### Azure Monitor Architecture

```
┌─────────────────────── DATA SOURCES ──────────────────────────┐
│                                                                 │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────────────┐   │
│  │ Applications │  │   Azure      │  │    Guest OS /     │   │
│  │              │  │  Resources   │  │   Infrastructure  │   │
│  └──────┬───────┘  └──────┬───────┘  └─────────┬─────────┘   │
│         │                  │                    │              │
└─────────┼──────────────────┼────────────────────┼──────────────┘
          │                  │                    │
          ▼                  ▼                    ▼
┌─────────────────────── AZURE MONITOR ──────────────────────────┐
│                                                                  │
│  ┌────────────────────── DATA PLATFORM ──────────────────────┐ │
│  │                                                             │ │
│  │  ┌───────────────────┐         ┌───────────────────────┐ │ │
│  │  │  METRICS          │         │  LOGS                 │ │ │
│  │  │  ─────────        │         │  ─────                │ │ │
│  │  │  • Time-series    │         │  • Events/Records     │ │ │
│  │  │  • Numerical      │         │  • Structured/        │ │ │
│  │  │  • Aggregated     │         │    Unstructured       │ │ │
│  │  │  • Real-time      │         │  • Rich query (KQL)   │ │ │
│  │  │  • Fast queries   │         │  • Deep analysis      │ │ │
│  │  └───────────────────┘         └───────────────────────┘ │ │
│  │                                                             │ │
│  └─────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────── INSIGHTS & ANALYSIS ───────────────────┐ │
│  │                                                              │ │
│  │  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐ │ │
│  │  │  Application   │  │   Container    │  │      VM      │ │ │
│  │  │   Insights     │  │   Insights     │  │   Insights   │ │ │
│  │  └────────────────┘  └────────────────┘  └──────────────┘ │ │
│  │                                                              │ │
│  │  ┌────────────────┐  ┌────────────────┐  ┌──────────────┐ │ │
│  │  │   Network      │  │     Storage    │  │    Others    │ │ │
│  │  │   Insights     │  │    Insights    │  │              │ │ │
│  │  └────────────────┘  └────────────────┘  └──────────────┘ │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────── VISUALIZE & ANALYZE ───────────────────┐ │
│  │  • Metrics Explorer    • Workbooks        • Power BI       │ │
│  │  • Dashboards          • Log Analytics    • Grafana        │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                                                                  │
│  ┌──────────────────── RESPOND & INTEGRATE ───────────────────┐ │
│  │  • Alerts & Actions    • Autoscale        • Event Hubs     │ │
│  │  • Action Groups       • Logic Apps       • Partner Tools  │ │
│  └──────────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────────┘
```

### Типы данных в Azure Monitor

#### 1. **Metrics** (Azure Monitor Metrics)
Числовые значения, собираемые через регулярные интервалы времени.

**Характеристики:**
- Лёгковесные и быстро обрабатываются
- Поддерживают сценарии, близкие к реальному времени (near real-time)
- Хранятся 93 дня (по умолчанию)
- Оптимизированы для алёртов и дашбордов

**Дополнительно:**
- Используются для отслеживания производительности ресурсов (CPU, Memory, Disk I/O, Requests и т.д.)
- Отлично подходят для построения autoscale-правил
- Имеют предагрегированную структуру (min, max, avg, count, sum), что ускоряет построение графиков
- Не предназначены для хранения детализированных логов или сложного текстового анализа

**Examples:**
```
CPU Percentage: 68.5% (avg over 5 min)
Memory Used:    4.2 GB
Request Rate:   1,247 requests/sec
Response Time:  120ms (p50)
```

#### 2. **Logs** (Azure Monitor Logs / Log Analytics)
Событийные записи, организованные в таблицы.

**Характеристики:**
- Содержат расширенный контекст (structured + semi-structured данные)
- Поддерживают сложные запросы с использованием KQL (Kusto Query Language)
- Срок хранения: от 30 дней до 2 лет (настраивается)
- Оптимизированы для анализа, расследований и поиска первопричин (root cause analysis)

**Дополнительно:**
- Подходят для хранения application logs, audit logs, security events
- Позволяют выполнять join’ы между таблицами, агрегации, фильтрацию и корреляцию событий
- Хорошо подходят для post-mortem анализа инцидентов
- Используются в Azure Sentinel и других security-решениях
- Стоимость зависит от объёма ingest’а и срока хранения

**Examples:**
```kusto
// Query: Failed requests with details
requests
| where success == false
| where timestamp > ago(1h)
| project timestamp, name, resultCode, duration, cloud_RoleName
| order by timestamp desc
| take 100
```

#### 3. **Activity Logs** (Control Plane)
Журнал операций, выполненных над ресурсами Azure (уровень управления — control plane).

**Примеры:**
- Создание или удаление ресурса
- Изменения конфигурации
- Назначение ролей (RBAC)
- События, связанные со здоровьем сервисов (Service Health)

**Дополнительно:**
- Activity Log фиксирует операции управления (ARM), а не события внутри самого приложения.
- Используется для аудита и контроля изменений инфраструктуры.
- Может быть отправлен в Log Analytics, Event Hub или Storage Account для дальнейшего анализа.

---

### Сравнение Metrics и Logs

| Feature | Metrics | Logs |
|---------|---------|------|
| **Тип данных** | Числовые временные ряды | Структурированные / неструктурированные записи |
| **Сбор данных** | Автоматически для платформенных ресурсов | Требуется настройка diagnostic settings |
| **Хранение** | 93 дня (по умолчанию) | 30 дней – 2 года |
| **Скорость запросов** | Очень высокая (предагрегированные данные) | Зависит от объёма данных |
| **Основное назначение** | Дашборды, алёрты, тренды | Анализ причин сбоев, отладка |
| **Стоимость** | Бесплатно (платформенные метрики) | Оплата за объём ingest’а |
| **Retention** | Фиксированное | Настраиваемое |
| **Примеры** | CPU%, частота запросов, длительность | Исключения, трассировки, кастомные события |

**Коротко для экзамена AZ-204:**
- Metrics → быстро, просто, числовые значения.
- Logs → глубоко, гибко, аналитика через KQL.

---

## Подробно об Application Insights

Application Insights — это APM (Application Performance Monitoring) компонент Azure Monitor, специально предназначенный для мониторинга приложений.

### Что такое Application Insights?

Application Insights предоставляет:

---

### 1. **Проактивный мониторинг производительности**

- Понимание того, как приложение работает до возникновения проблем
- Выявление трендов и аномалий
- Smart Detection использует машинное обучение для обнаружения нетипичных паттернов

**Дополнительно:**
- Поддерживает automatic dependency tracking (HTTP, SQL, Azure services)
- Позволяет отслеживать SLA/SLO через availability tests
- Интегрируется с алёртами Azure Monitor

---

### 2. **Реактивное расследование инцидентов**

- Анализ данных выполнения приложения для определения причины инцидентов
- Детальная телеметрия для troubleshooting
- Distributed tracing между сервисами (особенно важно в микросервисной архитектуре)

**Дополнительно:**
- Поддержка correlation ID для связывания запросов
- Просмотр end-to-end цепочки вызовов
- Интеграция с Log Analytics через KQL

---

### 3. **Аналитика использования (Usage Analytics)**

- Анализ того, как пользователи взаимодействуют с приложением
- Отслеживание кастомных бизнес-событий
- Анализ воронок (funnel analysis) и пользовательских сценариев (user flows)

**Дополнительно:**
- Поддержка custom metrics и custom events
- Возможность сегментации пользователей
- Помогает принимать продуктовые решения на основе данных

### Application Insights Architecture

```
┌────────── YOUR APPLICATION ──────────┐
│                                       │
│  ┌────────────────────────────────┐  │
│  │  Application Code              │  │
│  │  ────────────────              │  │
│  │  • Web App / API               │  │
│  │  • Functions                   │  │
│  │  • Background Jobs             │  │
│  └───────────┬────────────────────┘  │
│              │                        │
│  ┌───────────▼────────────────────┐  │
│  │  Application Insights SDK      │  │
│  │  OR Autoinstrumentation        │  │
│  │  ─────────────────────────     │  │
│  │  • Collects telemetry          │  │
│  │  • Preaggregates metrics       │  │
│  │  • Batches and transmits       │  │
│  └───────────┬────────────────────┘  │
└──────────────┼───────────────────────┘
               │ HTTPS
               │ (Telemetry Channel)
               ▼
┌────────── APPLICATION INSIGHTS ───────────┐
│                                            │
│  ┌──────────────────────────────────────┐ │
│  │  Ingestion Endpoint                  │ │
│  │  • v2.1/track (REST API)             │ │
│  │  • Validates & enriches data         │ │
│  │  • Applies sampling (if configured)  │ │
│  └──────────────┬───────────────────────┘ │
│                 │                          │
│  ┌──────────────▼───────────────────────┐ │
│  │  Storage (Azure Monitor Logs)        │ │
│  │  • Requests, Dependencies, Exceptions│ │
│  │  • Traces, Custom Events/Metrics    │ │
│  │  • Availability Results              │ │
│  └──────────────┬───────────────────────┘ │
│                 │                          │
│  ┌──────────────▼───────────────────────┐ │
│  │  Processing & Analysis               │ │
│  │  • Smart Detection (ML)              │ │
│  │  • Metric preaggregation             │ │
│  │  • Correlation & tracing             │ │
│  └──────────────┬───────────────────────┘ │
└─────────────────┼────────────────────────┘
                  │
                  ▼
┌────────── VISUALIZATION & ALERTS ─────────┐
│  • Azure Portal (Application Insights)     │
│  • Live Metrics Stream                     │
│  • Application Map                         │
│  • Failures, Performance, Usage            │
│  • Alerts & Action Groups                  │
│  • Log Analytics (KQL queries)             │
│  • Dashboards & Workbooks                  │
└────────────────────────────────────────────┘
```

### Ключевые возможности

#### 1. **Live Metrics Stream**

Панель телеметрии в реальном времени с задержкой менее секунды.

**Что отображается:**
- Частота входящих запросов (график в реальном времени)
- Ошибочные запросы (количество и список)
- Исходящие вызовы зависимостей (dependency calls)
- Исключения (по мере возникновения)
- Использование памяти и CPU
- Количество серверов и их состояние

**Сценарии использования:**
- Проверка деплоя (мгновенная обратная связь после релиза)
- Нагрузочное тестирование (мониторинг производительности в реальном времени)
- Реагирование на инциденты (live-расследование)

**Дополнительно:**
- Не требует сохранения данных в Log Analytics — работает напрямую с потоком телеметрии.
- Идеально подходит для проверки "живости" приложения сразу после выката.
- Часто используется при blue/green или canary deployment.
- Не предназначен для долгосрочного анализа — для этого используются Logs и KQL.

**Example View:**
```
LIVE METRICS STREAM
═══════════════════
Incoming Requests: ▁▃▅▇██▇▅▃▁ (1,247/sec)
Failed Requests:   2 (0.16%)
Avg Duration:      118ms
Servers:           3 (all healthy)

Recent Exceptions:
  • NullReferenceException in OrderController.Checkout
  • SqlException: Timeout expired (30s)

Recent Requests:
  ✓ GET /api/products      89ms
  ✓ POST /api/orders      245ms
  ✗ GET /api/users/123    503 Service Unavailable
```

#### 2. **Smart Detection**

Обнаружение аномалий на основе ИИ, которое изучает нормальное поведение вашего приложения.

**Что обнаруживает:**
- **Аномалии отказов (Failure anomalies)**: Необычный рост количества неуспешных запросов
- **Аномалии производительности (Performance anomalies)**: Ненормальная деградация времени ответа
- **Утечки памяти (Memory leaks)**: Постепенное увеличение потребления памяти
- **Проблемы безопасности (Security issues)**: Нетипичные шаблоны трассировок
- **Медленные зависимости (Slow dependency)**: Деградация внешних сервисов

**Дополнительно:**
- Работает автоматически после накопления достаточного объёма телеметрии.
- Не требует ручной настройки порогов (в отличие от классических алёртов).
- Использует исторические данные для построения baseline.
- Отправляет уведомления через Azure Monitor Alerts.
- Полезен в продакшене, где сложно заранее определить корректные threshold’ы.

**Важно для AZ-204:**
Smart Detection — это ML-based механизм, который дополняет, но не заменяет обычные alert rules.


**Example Alert:**
```
🔔 Smart Detection Alert
═══════════════════════
Application: ecommerce-api
Severity: Warning

Anomaly Detected: Failure Rate Increase
───────────────────────────────────────
Normal failure rate: 0.2%
Current failure rate: 3.8% ⚠️ (19x increase)

Affected endpoint: POST /api/checkout
Time window: Last 15 minutes
Possible cause: Payment gateway timeout

Recommended Action:
→ Check Application Map for dependency issues
→ Review recent deployments
→ Investigate payment service health
```

**Configuration:**
```bash
# Enable Smart Detection (enabled by default)
az monitor app-insights component update \
  --app MyApp \
  --resource-group MyResourceGroup \
  --set kind=web

# Configure email notifications
az monitor app-insights component billing update \
  --app MyApp \
  --resource-group MyResourceGroup \
  --cap 10
```

#### 3. **Availability Tests** (Synthetic Monitoring)

Проактивный мониторинг доступности путём отправки запросов к вашему приложению из разных географических регионов Azure.

**Идея:**
Вместо ожидания жалоб пользователей система сама регулярно проверяет доступность и корректность ответа приложения.

---

### Типы тестов

#### 1. **Standard Test** (Рекомендуется)

- Один HTTP/HTTPS-запрос
- Проверка кода ответа и содержимого
- Валидация TLS/SSL-сертификата
- Поддержка кастомных заголовков и аутентификации
- Таймаут запроса (по умолчанию 30 секунд)

**Особенности:**
- Можно запускать из нескольких регионов одновременно.
- Позволяет настроить alert при недоступности из определённого количества локаций.
- Подходит для проверки публичных API и веб-приложений.

---

#### 2. **URL Ping Test** (Классический, будет выведен из эксплуатации в сентябре 2026)

- Простой HTTP GET-запрос
- Измерение времени ответа
- Базовая проверка содержимого

**Важно:**
- Более ограниченный функционал по сравнению со Standard Test.
- Постепенно заменяется Standard Test.

---

#### 3. **Custom TrackAvailability Test**

- Написание собственного тестового кода
- Поддержка сложных сценариев (многошаговые процессы, аутентификация, workflow)
- Реализация через Azure Functions или WebJobs

**Когда использовать:**
- Нужно протестировать login flow, корзину, оплату и другие multi-step сценарии.
- Требуется сложная бизнес-логика в проверке.

---

### Дополнительно

- Availability Tests — это synthetic monitoring (искусственная нагрузка), в отличие от real user monitoring.
- Результаты сохраняются в Application Insights и доступны для анализа через KQL.
- Часто используются для проверки SLA и глобальной доступности сервиса.
- Можно комбинировать с alert rules для автоматического реагирования.

**Для AZ-204 важно помнить:**
- Standard Test — основной и рекомендуемый вариант.
- Тесты могут запускаться из нескольких регионов.
- Поддерживается интеграция с alerting и dashboard.
**Configuration Example:**
```bash
# Create availability test
az monitor app-insights web-test create \
  --resource-group MyResourceGroup \
  --name "Homepage Test" \
  --location "eastus" \
  --web-test-name "prod-homepage-test" \
  --web-test-kind "standard" \
  --locations "us-west-ca-sjc-azr" "us-va-ash-azr" "emea-nl-ams-azr" \
  --frequency 300 \
  --timeout 30 \
  --enabled true \
  --synthetic-monitor-id "homepage-availability" \
  --request-url "https://www.contoso.com" \
  --expected-http-status-code 200
```

**Test Locations (Examples):**
- us-ca-sjc-azr: West US (California)
- us-va-ash-azr: East US (Virginia)
- emea-nl-ams-azr: West Europe (Netherlands)
- apac-jp-kaw-azr: Japan East
- apac-sg-sin-azr: Southeast Asia (Singapore)

**Alert Setup:**
```bash
# Create alert rule for availability test
az monitor metrics alert create \
  --name "Homepage Availability Alert" \
  --resource-group MyResourceGroup \
  --scopes "/subscriptions/{sub-id}/resourceGroups/MyResourceGroup/providers/Microsoft.Insights/webtests/Homepage Test" \
  --condition "avg availabilityResults/availabilityPercentage < 90" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action-group-ids "/subscriptions/{sub-id}/resourceGroups/MyResourceGroup/providers/microsoft.insights/actionGroups/EmailAdmins"
```

#### 4. **Application Map**

Visual representation of your application's architecture and component health.

**What It Shows:**
```
┌────────────────────────────────────────────────────────────────┐
│                      APPLICATION MAP                            │
├────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Users                                                         │
│    │                                                            │
│    ▼                                                            │
│  ┌─────────────────┐        ┌─────────────────┐              │
│  │  Web Frontend   │───────▶│  API Gateway    │              │
│  │  ─────────────  │        │  ────────────   │              │
│  │  1,247 req/sec  │        │  1,189 req/sec  │              │
│  │  ✓ Healthy      │        │  ✓ Healthy      │              │
│  │  Avg: 150ms     │        │  Avg: 220ms     │              │
│  └─────────────────┘        └────────┬────────┘              │
│                                       │                         │
│                     ┌─────────────────┼─────────────────┐      │
│                     │                 │                 │      │
│                     ▼                 ▼                 ▼      │
│           ┌────────────────┐ ┌──────────────┐ ┌─────────────┐│
│           │ Order Service  │ │  Inventory   │ │  Payment    ││
│           │ ────────────── │ │  Service     │ │  Service    ││
│           │ 458 req/sec    │ │  312 req/sec │ │  458 req/sec││
│           │ ✓ Healthy      │ │  ✓ Healthy   │ │  ⚠️ Slow    ││
│           │ Avg: 180ms     │ │  Avg: 45ms   │ │  Avg: 1.2s  ││
│           └───────┬────────┘ └──────┬───────┘ └──────┬──────┘│
│                   │                 │                 │        │
│                   ▼                 ▼                 ▼        │
│          ┌────────────────┐ ┌──────────────┐ ┌──────────────┐│
│          │  SQL Database  │ │  Cosmos DB   │ │  Stripe API  ││
│          │  ────────────  │ │  ──────────  │ │  ──────────  ││
│          │  ✓ Healthy     │ │  ✓ Healthy   │ │  🔴 Failing ││
│          │  Avg: 28ms     │ │  Avg: 12ms   │ │  Avg: 5.2s  ││
│          └────────────────┘ └──────────────┘ └──────────────┘│
│                                                                 │
└────────────────────────────────────────────────────────────────┘

Insight: Payment Service is slow due to Stripe API issues (5.2s avg)
Action: Consider implementing circuit breaker or fallback mechanism
```

**Возможности:**
- **Состояние компонентов (Component health)**: Цветовая индикация статуса (зелёный / жёлтый / красный)
- **Показатели производительности (Performance indicators)**: Частота запросов, время ответа, процент ошибок
- **Отслеживание зависимостей (Dependency tracking)**: Внешние сервисы, базы данных, хранилища
- **Переход к деталям (Click-through)**: Возможность провалиться в конкретный компонент
- **Выбор временного диапазона (Time range)**: Просмотр исторических данных

**Как это работает:**
- Использует distributed tracing (через correlation ID)
- Автоматически обнаруживает компоненты через HTTP-вызовы
- Группирует сервисы по свойству `cloud_RoleName`
- Требует установленного Application Insights SDK на всех компонентах системы

**Дополнительно:**
- Позволяет быстро определить "узкое место" в микросервисной архитектуре.
- Особенно полезно при большом количестве сервисов и внешних зависимостей.
- Работает корректно только при правильной передаче trace context между сервисами.

---

#### 5. **Distributed Tracing**

Сквозное (end-to-end) отслеживание запроса через несколько микросервисов.

Позволяет понять:
- где именно возникла задержка,
- какой сервис вернул ошибку,
- как распределяется время выполнения по цепочке вызовов.

---

### Основные понятия трассировки

| Термин | Определение | Пример |
|--------|------------|---------|
| **Trace** | Полный путь запроса | Процесс оформления заказа пользователем |
| **Trace ID** | Уникальный идентификатор всей операции | `4bf92f3577b34da6a3ce929d0e0e4736` |
| **Span** | Отдельная операция внутри Trace | SQL-запрос, HTTP-вызов |
| **Span ID** | Уникальный ID конкретного span | `00f067aa0ba902b7` |
| **Parent Span ID** | Связывает дочерний span с родительским | Формирует иерархию вызовов |
| **Operation Name** | Читаемое имя операции | `POST /api/checkout` |
| **Duration** | Время выполнения операции | 245ms |

---

### Дополнительно

- Distributed tracing основан на стандартах W3C Trace Context.
- Каждый входящий HTTP-запрос получает уникальный Trace ID.
- Все downstream-вызовы наследуют этот ID.
- Позволяет визуализировать "waterfall" выполнения запроса.

**Для AZ-204 важно:**
- Нужно установить SDK на все сервисы.
- Корреляция работает автоматически для HTTP и популярных библиотек.
- Ключевая цель — найти bottleneck и источник ошибок в распределённой системе.
**Example Distributed Trace:**
```
Trace ID: 4bf92f3577b34da6a3ce929d0e0e4736
Operation: POST /api/checkout

┌─ Web Frontend (span: 00f067aa0ba902b7)
│  Duration: 1,580ms
│  │
│  └─▶ API Gateway (span: 11e067bb1ca913c8, parent: 00f067aa0ba902b7)
│      Duration: 1,420ms
│      │
│      ├─▶ Inventory Service (span: 22f078cc2db924d9, parent: 11e067bb1ca913c8)
│      │   Duration: 85ms
│      │   └─▶ Cosmos DB Query (span: 33g089dd3ec035ea, parent: 22f078cc2db924d9)
│      │       Duration: 42ms
│      │       Query: SELECT * FROM c WHERE c.productId = @id
│      │
│      ├─▶ Order Service (span: 44h09aee4fd146fb, parent: 11e067bb1ca913c8)
│      │   Duration: 210ms
│      │   └─▶ SQL Database Insert (span: 55i0abff5ge257gc, parent: 44h09aee4fd146fb)
│      │       Duration: 125ms
│      │       Query: INSERT INTO Orders...
│      │
│      └─▶ Payment Service (span: 66j0bcgg6hf368hd, parent: 11e067bb1ca913c8)
│          Duration: 1,180ms ⚠️ BOTTLENECK
│          │
│          └─▶ Stripe API (span: 77k0cdh07ig479ie, parent: 66j0bcgg6hf368hd)
│              Duration: 1,150ms ⚠️ SLOW DEPENDENCY
│              URL: https://api.stripe.com/v1/charges
│              Result: 200 OK
```

**Trace Correlation (HTTP Headers):**
```http
Request-Id: |4bf92f3577b34da6a3ce929d0e0e4736.00f067aa0ba902b7.
traceparent: 00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01
tracestate: rojo=00f067aa0ba902b7,congo=t61rcWkgMzE
```

#### 6. **Failures Analysis**

Dedicated view for investigating failed requests and exceptions.

**Failures Dashboard:**
```
FAILURES
════════
Last 24 hours

Top Failing Operations:
┌────────────────────────────────────────────────────────────┐
│ Operation                    Count  %     Trend             │
├────────────────────────────────────────────────────────────┤
│ POST /api/checkout            187  3.8%  ▁▃▅▇██▇▅▃▁ ⬆️    │
│ GET /api/users/{id}            64  2.1%  ▃▃▃▅▅▅▃▃▃ →      │
│ POST /api/orders               23  0.9%  ▁▁▁▃▃▃▁▁▁ →      │
└────────────────────────────────────────────────────────────┘

Exception Types:
┌────────────────────────────────────────────────────────────┐
│ Exception                           Count  Instances       │
├────────────────────────────────────────────────────────────┤
│ System.TimeoutException               94  [View Details]   │
│ System.NullReferenceException         48  [View Details]   │
│ System.Data.SqlClient.SqlException    31  [View Details]   │
│ System.UnauthorizedAccessException    14  [View Details]   │
└────────────────────────────────────────────────────────────┘

Failed Dependencies:
┌────────────────────────────────────────────────────────────┐
│ Dependency              Failure %  Avg Duration            │
├────────────────────────────────────────────────────────────┤
│ api.stripe.com              12.3%  5,240ms ⚠️              │
│ SQL Database                 2.1%     89ms ✓              │
│ Blob Storage                 0.8%    145ms ✓              │
└────────────────────────────────────────────────────────────┘
```

**Exception Details:**
```json
{
  "timestamp": "2024-01-15T14:23:45.678Z",
  "type": "System.TimeoutException",
  "message": "The operation has timed out.",
  "stack": [
    "   at System.Net.Http.HttpClient.SendAsync(HttpRequestMessage request)",
    "   at PaymentService.ProcessPayment(PaymentRequest request)",
    "   at OrderController.Checkout(CheckoutRequest request)"
  ],
  "operation": {
    "id": "4bf92f3577b34da6a3ce929d0e0e4736",
    "name": "POST /api/checkout",
    "duration": 30000
  },
  "request": {
    "url": "https://api.stripe.com/v1/charges",
    "method": "POST",
    "responseCode": null
  },
  "user": {
    "id": "user_12345",
    "authenticatedUserId": "john.doe@example.com"
  },
  "custom": {
    "orderId": "ord_abc123",
    "amount": 149.99,
    "currency": "USD"
  }
}
```

#### 7. **Performance View**

Analyze application performance with detailed breakdowns.

**Performance Metrics:**
```
PERFORMANCE
═══════════
Last 24 hours

Overall Performance:
  Requests:       1,247,583
  Avg Duration:   218ms (p50: 120ms, p95: 850ms, p99: 2.3s)
  Success Rate:   97.8%
  
Slowest Operations:
┌────────────────────────────────────────────────────────────┐
│ Operation                 Calls    p50    p95    p99        │
├────────────────────────────────────────────────────────────┤
│ POST /api/checkout        45.2k   245ms  1.2s   3.8s  ⚠️   │
│ GET /api/orders           38.7k   89ms   450ms  1.1s  →    │
│ POST /api/orders          22.1k   180ms  680ms  1.8s  →    │
│ GET /api/products         89.4k   45ms   180ms  420ms ✓    │
└────────────────────────────────────────────────────────────┘

Dependency Performance:
┌────────────────────────────────────────────────────────────┐
│ Dependency             Calls    Avg     p95    Failure %   │
├────────────────────────────────────────────────────────────┤
│ Stripe API             45.2k   850ms   2.1s   12.3%  ⚠️    │
│ SQL Database          187.6k    28ms    89ms   2.1%  ✓    │
│ Cosmos DB             134.2k    12ms    45ms   0.3%  ✓    │
│ Redis Cache           421.8k     4ms    12ms   0.1%  ✓    │
└────────────────────────────────────────────────────────────┘
```

#### 8. **Usage Analytics**

Understand how users interact with your application.

**Key Reports:**

1. **Users, Sessions, Events**
```
USAGE
═════
Last 7 days

Users:          47,238 (↑ 12.3% vs last week)
Sessions:       89,421 (↑ 8.7%)
Page Views:    412,847 (↑ 5.2%)
Avg Session:    8m 24s

Top Pages:
  1. /products          127,483 views
  2. /                   89,234 views
  3. /cart               45,891 views
  4. /checkout           22,456 views
  5. /account            18,932 views

User Retention:
  Day 1:  42%
  Day 7:  23%
  Day 30: 12%
```

2. **Funnels**
```
CHECKOUT FUNNEL
═══════════════
Last 30 days

/products     ══════════════════════════ 100% (45,238 users)
    │
    ▼
/cart         ══════════════════         68% (30,762 users)
    │                                    ↓ 32% abandoned
    ▼
/checkout     ═════════════              45% (20,357 users)
    │                                    ↓ 23% abandoned
    ▼
/confirmation ═══════════                38% (17,190 users)
                                         ↓ 7% abandoned

Conversion Rate: 38%
Biggest Drop: Products → Cart (32% abandoned)
```

3. **Custom Events**
```csharp
// Track custom business events
telemetryClient.TrackEvent("ProductViewed", 
    new Dictionary<string, string> {
        { "ProductId", "prod_123" },
        { "Category", "Electronics" }
    },
    new Dictionary<string, double> {
        { "Price", 299.99 }
    });

telemetryClient.TrackEvent("AddedToCart", 
    new Dictionary<string, string> {
        { "ProductId", "prod_123" },
        { "UserId", userId }
    },
    new Dictionary<string, double> {
        { "Quantity", 2 }
    });
```

### Типы телеметрии

Application Insights собирает несколько типов телеметрии:

| Тип телеметрии | Описание | Примеры | Назначение |
|----------------|----------|----------|------------|
| **Requests** | Входящие HTTP-запросы | GET /api/products, POST /api/orders | Производительность, доступность |
| **Dependencies** | Исходящие вызовы | SQL-запросы, HTTP-вызовы, Redis | Поиск узких мест |
| **Exceptions** | Перехваченные и неперехваченные ошибки | NullReferenceException, SqlException | Отслеживание ошибок |
| **Traces** | Лог-сообщения | Debug, Info, Warning, Error | Диагностика, отладка |
| **Events** | Кастомные бизнес-события | UserLoggedIn, ProductPurchased | Бизнес-аналитика |
| **Metrics** | Кастомные числовые показатели | CartValue, ItemsInStock | Бизнес-KPI |
| **Page Views** | Загрузки страниц фронтенда | URL страницы, время загрузки | Пользовательский опыт |
| **Availability** | Результаты синтетических тестов | Статус теста, регион, время ответа | Мониторинг доступности |

---

### Дополнительные пояснения

- **Requests + Dependencies** вместе формируют полную картину выполнения запроса (входящий запрос → обращения к БД и внешним сервисам).
- **Exceptions** автоматически коррелируются с конкретным Request или Dependency.
- **Traces** часто отправляются через стандартные логгеры (например, ILogger, Log4j и т.д.).
- **Events и Metrics** — это способ добавить бизнес-контекст к технической телеметрии.
- **Page Views** актуальны для SPA-приложений и веб-клиентов.
- **Availability** генерируется системой, а не реальными пользователями.

---

### Для AZ-204 важно помнить

- Application Insights автоматически собирает Requests, Dependencies и Exceptions.
- Custom Events и Custom Metrics нужно отправлять вручную через SDK.
- Вся телеметрия может анализироваться через KQL.
- Все типы телеметрии коррелируются через Trace ID.

### Getting Started with Application Insights

#### Step 1: Create Application Insights Resource

```bash
# Create resource
az monitor app-insights component create \
  --app MyApp \
  --location eastus \
  --resource-group MyResourceGroup \
  --application-type web \
  --kind web \
  --retention-time 90

# Get connection string
az monitor app-insights component show \
  --app MyApp \
  --resource-group MyResourceGroup \
  --query connectionString \
  --output tsv
```

**Output:**
```
InstrumentationKey=12345678-1234-1234-1234-123456789012;IngestionEndpoint=https://eastus-1.in.applicationinsights.azure.com/;LiveEndpoint=https://eastus.livediagnostics.monitor.azure.com/
```

#### Step 2: Configure Application (Multiple Options)

**Option A: App Service Autoinstrumentation (No Code Changes)**
```bash
az webapp config appsettings set \
  --name MyWebApp \
  --resource-group MyResourceGroup \
  --settings APPLICATIONINSIGHTS_CONNECTION_STRING="<connection-string>"

az webapp config appsettings set \
  --name MyWebApp \
  --resource-group MyResourceGroup \
  --settings ApplicationInsightsAgent_EXTENSION_VERSION="~3"
```

**Option B: .NET Core with SDK**
```bash
# Install NuGet package
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

```csharp
// Program.cs
using Microsoft.ApplicationInsights.AspNetCore.Extensions;

var builder = WebApplication.CreateBuilder(args);

// Add Application Insights
builder.Services.AddApplicationInsightsTelemetry(options =>
{
    options.ConnectionString = builder.Configuration["APPLICATIONINSIGHTS_CONNECTION_STRING"];
});

builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();
app.Run();
```

**Option C: Node.js**
```bash
npm install applicationinsights
```

```javascript
// app.js
const appInsights = require("applicationinsights");
appInsights.setup("<connection-string>")
    .setAutoDependencyCorrelation(true)
    .setAutoCollectRequests(true)
    .setAutoCollectPerformance(true, true)
    .setAutoCollectExceptions(true)
    .setAutoCollectDependencies(true)
    .setAutoCollectConsole(true)
    .setUseDiskRetryCaching(true)
    .start();

const express = require('express');
const app = express();
// ... rest of your app
```

**Option D: Python**
```bash
pip install opencensus-ext-azure
```

```python
# app.py
from opencensus.ext.azure.log_exporter import AzureLogHandler
from opencensus.ext.azure import metrics_exporter
import logging

# Configure logging
logger = logging.getLogger(__name__)
logger.addHandler(AzureLogHandler(
    connection_string='<connection-string>')
)

# Track metrics
exporter = metrics_exporter.new_metrics_exporter(
    connection_string='<connection-string>')

# Your application code
logger.warning('This is a warning message')
```

#### Step 3: Verify Data Collection

```bash
# Query recent requests
az monitor app-insights query \
  --app MyApp \
  --resource-group MyResourceGroup \
  --analytics-query "requests | where timestamp > ago(1h) | summarize count() by bin(timestamp, 5m)" \
  --offset 1h

# Get performance metrics
az monitor app-insights metrics show \
  --app MyApp \
  --resource-group MyResourceGroup \
  --metric "requests/count" \
  --interval PT1H
```

### Особенности ценообразования

Application Insights работает по модели Pay-As-You-Go (оплата по фактическому использованию).

| Компонент | Стоимость | Бесплатно включено |
|------------|------------|-------------------|
| **Ingestion данных** | $2.30/GB (после бесплатного лимита) | 5 GB/месяц (на подписку) |
| **Хранение данных** | $0.10/GB/месяц (после 90 дней) | 90 дней включено |
| **Standard Tests** | $0.006 за тест | Нет |
| **Multi-step Tests** | $0.015 за тест | Нет |

> ⚠️ Цены могут меняться — на экзамене важно понимать модель оплаты, а не точные цифры.

---

### Что влияет на стоимость

- Объём отправляемой телеметрии (Requests, Dependencies, Traces и т.д.)
- Частота логирования
- Количество Availability Tests
- Срок хранения данных
- Уровень детализации логов (особенно Debug)

---

### Рекомендации по оптимизации затрат

1. **Использовать Sampling**  
   Снижение объёма телеметрии на 50–90%.  
   Особенно полезно при высокой нагрузке.

2. **Фильтрация телеметрии**  
   Исключать ненужные данные:
   - health-check запросы
   - статические файлы
   - шумные Debug-логи в продакшене

3. **Использовать предагрегированные метрики**  
   Предпочитать `GetMetric()` вместо `TrackMetric()` для снижения объёма ingest’а.

4. **Настроить срок хранения (Retention)**  
   Хранить только необходимый период (по умолчанию — 90 дней).

5. **Ограничить дневной лимит (Daily Cap)**  
   Установить дневной лимит, чтобы избежать неожиданного перерасхода бюджета.

---

### Практический совет

В продакшене:
- Включайте sampling.
- Ограничивайте Debug-логи.
- Мониторьте ingestion в Cost Analysis.
- Проверяйте рост телеметрии после каждого релиза.

**Для AZ-204 важно помнить:**
- Основная статья расходов — Data Ingestion.
- Sampling — ключевой инструмент оптимизации.
- Retention сверх 90 дней оплачивается отдельно.

```bash
# Set daily cap
az monitor app-insights component billing update \
  --app MyApp \
  --resource-group MyResourceGroup \
  --cap 5
```
## Основные выводы

✅ **Application Insights — это расширение Azure Monitor**, предназначенное для мониторинга производительности приложений (APM)

✅ **Три типа данных**:
- Metrics (быстро, числовые значения)
- Logs (богатый контекст)
- Traces (поток выполнения запроса)

✅ **Live Metrics Stream** обеспечивает видимость в реальном времени с задержкой менее секунды

✅ **Smart Detection** использует ИИ для автоматического обнаружения аномалий

✅ **Application Map** визуализирует распределённую архитектуру приложения и состояние компонентов

✅ **Availability Tests** проактивно проверяют доступность endpoints из разных регионов мира

✅ **Distributed Tracing** отслеживает запросы между микросервисами через correlation ID

✅ **Несколько вариантов инструментирования**:
- Autoinstrumentation (без изменения кода)
- SDK (кастомизация)
- OpenTelemetry (стандарт индустрии)

---

## Советы для экзамена AZ-204

💡 **Autoinstrumentation vs SDK**  
Для App Service и Azure Functions чаще всего правильный ответ — autoinstrumentation (проще, без изменений кода).

💡 **Live Metrics**  
Используется во время деплоя для мгновенной проверки работоспособности.

💡 **Smart Detection**  
Включён по умолчанию, основан на машинном обучении, не требует настройки.

💡 **Application Map**  
Лучший инструмент для анализа распределённых приложений и поиска bottleneck’ов.

💡 **Availability Tests**  
Используйте Standard Test (URL Ping будет выведен из эксплуатации в 2026 году).

💡 **Connection String**  
Современный стандарт подключения (заменяет instrumentation key).

💡 **Pricing**  
Первые 5 GB в месяц бесплатно, далее оплата за объём ingest’а.

---

## Что дальше

В следующем разделе вы узнаете:

- Разницу между **log-based metrics и standard metrics**
- Как предагрегация повышает производительность
- Когда использовать каждый тип метрик
- Стратегии sampling и фильтрации

---

### Финальный акцент для AZ-204

Если в вопросе речь о:
- мониторинге производительности приложения → Application Insights
- инфраструктурных метриках → Azure Monitor Metrics
- глубоком анализе и KQL → Log Analytics

Главное — понимать различия и правильно выбирать инструмент под задачу.
---

Когда приложение отправляет телеметрию:
App → Telemetry Initializer → Telemetry Processor → AI backend

1️⃣ Telemetry Initializers
Добавляют или изменяют свойства
Например: добавить cloud role name
Не могут удалить telemetry item

2️⃣ Telemetry Processors
Это middleware в telemetry pipeline.
Они могут:
✔ Фильтровать
✔ Полностью удалить telemetry item
✔ Изменить объект перед отправкой
✔ Реализовать кастомную логику

Distributed tracing:
Отслеживает зависимые вызовы
Коррелирует запросы
Не управляет фильтрацией телеметрии

Funnels
Аналитика пользовательских потоков
Не влияет на сбор телеметрии

|                         | Initializer | Processor |
| ----------------------- | ----------- | --------- |
| Изменить свойства       | ✅           | ✅         |
| Отфильтровать / удалить | ❌           | ✅         |
| Работает до отправки    | ✅           | ✅         |
| Middleware pipeline     | ❌           | ✅         |

**📚 Further Reading:**
- [Application Insights Overview](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)
- [OpenTelemetry with Azure Monitor](https://learn.microsoft.com/en-us/azure/azure-monitor/app/opentelemetry-enable)
- [Application Insights Pricing](https://azure.microsoft.com/en-us/pricing/details/monitor/)
