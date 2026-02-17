# Azure Functions Hosting Plans (Планы размещения Azure Functions)

## Key Concepts (Ключевые понятия)

- **Consumption plan** — оплата за выполнение, автоматическое масштабирование
- **Flex Consumption plan** — расширенная версия Consumption с большим контролем
- **Premium plan** — предразогретые инстансы, VNet, без лимита по времени выполнения
- **Dedicated plan** — размещение в App Service Plan с фиксированной оплатой
- **Container Apps** — запуск функций в контейнерах (кастомные образы)

---

## Hosting Plan Overview (Обзор планов)

### Decision Factors (Критерии выбора)

При выборе плана учитывайте:

1. **Масштабирование** — автоматическое или ручное
2. **Стоимость** — pay-per-use или фиксированная оплата
3. **Время выполнения** — ограничения по timeout
4. **Сеть** — требуется ли VNet
5. **Ресурсы** — CPU и память
6. **Cold start** — критична ли задержка запуска

---

# Consumption Plan (План Consumption)

## Characteristics (Характеристики)

- 📦 План по умолчанию
- 💰 **Оплата только за выполнение**
- 📈 Автоматическое масштабирование
- ⚡ Масштабируется по событиям
- 💤 Нет оплаты за простой

---

## Scaling Behavior (Поведение масштабирования)

| Aspect | Details |
|----------|----------|
| **Scale-out** | Автоматическое, по событиям |
| **Max instances** | Windows: 200, Linux: 100 |
| **Scale-in** | Автоматическое при простое |
| **Cold start** | Возможен (масштабируется до 0) |
| **Scale unit** | На уровне Function App |

---

## Timeout (Ограничения по времени)

| Timeframe | Duration |
|------------|----------|
| **Default** | 5 минут |
| **Maximum** | 10 минут |

⚠️ **HTTP timeout** — максимум 230 секунд  
(ограничение Azure Load Balancer)

---

## Billing (Оплата)

Оплачивается:

- ⏱ **Execution time** — GB-seconds (память × время)
- 🔁 **Executions** — количество вызовов

### Бесплатный лимит:

- 1 000 000 выполнений в месяц
- 400 000 GB-seconds в месяц

---

## When to Use (Когда использовать)

✅ Непредсказуемая нагрузка  
✅ Чувствительность к стоимости  
✅ Короткие функции (< 10 минут)  
✅ Редкие вызовы  
✅ Нет необходимости в VNet

---

## Limitations (Ограничения)

❌ Максимум 10 минут выполнения  
❌ Возможен cold start  
❌ Нет VNet интеграции  
❌ Ограниченные ресурсы инстанса

---

## Важно для AZ-204

- Consumption масштабируется до нуля
- Cold start возможен при отсутствии активности
- HTTP-триггер ограничен 230 секундами
- Timeout и HTTP timeout — разные ограничения
- Оплата зависит от памяти и времени выполнения

### CLI Example
```bash
# Create Consumption plan function app
az functionapp create \
  --name <app-name> \
  --resource-group <rg-name> \
  --consumption-plan-location <region> \
  --runtime node \
  --runtime-version 18 \
  --storage-account <storage-name>
```
# Flex Consumption Plan (План Flex Consumption)

## Characteristics (Характеристики)

- 🚀 Улучшенная версия Consumption
- 🎯 **Per-function scaling** — масштабирование отдельно для каждой функции
- 🖥 Возможность выбора размера инстанса
- 🔒 Поддержка **VNet integration**
- 💰 Модель оплаты pay-as-you-go (как в Consumption)

> 💡 Flex Consumption сочетает serverless-модель с большим контролем над масштабированием.

---

## Scaling Behavior (Поведение масштабирования)

| Aspect | Details |
|----------|----------|
| **Scale-out** | На уровне отдельной функции (более предсказуемо) |
| **Max instances** | Ограничено общей памятью региона |
| **Instance concurrency** | Настраивается для каждой функции |
| **Pre-provisioned instances** | Always-ready инстансы (снижают cold start) |

---

## Timeout (Ограничения по времени)

| Timeframe | Duration |
|------------|----------|
| **Default** | 30 минут |
| **Maximum** | Без явного лимита (60 мин grace при scale-in) |

> ⚠️ Во время scale-in даётся до 60 минут для завершения выполнения.

---

## Advanced Features (Расширенные возможности)

- ⚙️ **Per-instance concurrency** — контроль числа одновременных выполнений
- 🔥 **Always ready instances** — уменьшение cold start
- 🌐 **VNet integration** — приватная сеть
- 🧩 **Flexible compute** — выбор размера инстанса

---

## When to Use (Когда использовать)

✅ Требуется VNet при serverless-модели  
✅ Нужно минимизировать cold start  
✅ Требуется контроль масштабирования по функциям  
✅ Нужна более предсказуемая модель масштабирования  
✅ Требуется больший timeout (30 минут по умолчанию)

---

## Важно для AZ-204

- Flex Consumption = Consumption + VNet + контроль масштабирования
- Timeout больше, чем в обычном Consumption
- Поддерживает always-ready инстансы
- Подходит для production-нагрузок с serverless-экономикой

### CLI Example
```bash
# Create Flex Consumption plan function app
az functionapp create \
  --name <app-name> \
  --resource-group <rg-name> \
  --flexconsumption-location <region> \
  --runtime node \
  --runtime-version 18 \
  --storage-account <storage-name> \
  --max-instances 100 \
  --always-ready-instances 5
```

# Premium Plan (План Premium)

## Characteristics (Характеристики)

- 🔥 **Prewarmed workers** — отсутствие cold start
- ⏳ **Неограниченное время выполнения** (60 минут grace при scale-in)
- 🌐 **VNet connectivity** — подключение к приватным сетям
- 🖥 Более мощные инстансы (больше CPU и памяти)
- 📊 Предсказуемая производительность

> 💡 Premium = serverless-масштабирование + выделенные ресурсы.

---

## Instance Types (Размеры инстансов)

| Size | vCPU | Memory |
|-------|------|--------|
| **EP1** | 1 | 3.5 GB |
| **EP2** | 2 | 7 GB |
| **EP3** | 4 | 14 GB |

---

## Scaling Behavior (Поведение масштабирования)

| Aspect | Details |
|----------|----------|
| **Scale-out** | Автоматическое, по событиям |
| **Max instances** | Windows: до 100, Linux: 20–100 |
| **Prewarmed workers** | Всегда готовы (без cold start) |
| **Min instances** | Настраиваемый минимум (оплачивается даже при простое) |

---

## Timeout (Ограничения по времени)

| Timeframe | Duration |
|------------|----------|
| **Default** | 30 минут |
| **Maximum** | Без лимита (60 мин grace при scale-in) |

---

## Billing (Оплата)

- 💰 Фиксированная стоимость в месяц
- Зависит от:
  - Количества инстансов
  - Размера (EP1, EP2, EP3)
- Минимальное количество инстансов оплачивается всегда

---

## When to Use (Когда использовать)

✅ Часто выполняющиеся функции  
✅ Требуется VNet  
✅ Нужны больше CPU/памяти  
✅ Долгоживущие функции (> 10 минут)  
✅ Нельзя допустить cold start  
✅ Очень большое число коротких вызовов (дорого в Consumption)  
✅ Несколько Function Apps на одном плане  
✅ Нужен кастомный Linux-образ

---

## Limitations (Ограничения)

⚠️ Дороже, чем Consumption  
⚠️ Оплата даже при простое

---

## Важно для AZ-204

- Premium устраняет cold start
- Поддерживает VNet
- Нет жёсткого timeout (в отличие от Consumption)
- Подходит для production-нагрузок с требованиями к стабильности
- Позволяет размещать несколько Function Apps в одном плане

### CLI Example
```bash
# Create Premium plan
az functionapp plan create \
  --name <plan-name> \
  --resource-group <rg-name> \
  --location <region> \
  --sku EP1 \
  --is-linux

# Create function app on Premium plan
az functionapp create \
  --name <app-name> \
  --resource-group <rg-name> \
  --plan <plan-name> \
  --runtime node \
  --storage-account <storage-name>
```
# Dedicated Plan (App Service Plan)

## Characteristics (Характеристики)

- 🖥 Работает в рамках **App Service Plan** (как Web Apps)
- 💰 Предсказуемая фиксированная оплата
- 📈 Масштабирование вручную или через autoscale rules
- ⏳ Подходит для долгоживущих и непрерывных задач
- 🔒 Поддержка полной изоляции через App Service Environment (ASE)

> 💡 Функции используют те же ресурсы, что и Web Apps в этом плане.

---

## Scaling Behavior (Поведение масштабирования)

| Aspect | Details |
|----------|----------|
| **Scale-out** | Ручное или через autoscale rules |
| **Max instances** | 10–30 (до 100 с ASE) |
| **Minimum instances** | Всегда минимум 1 |
| **No event-driven scale** | Нет автоматического масштабирования по событиям |

> ⚠️ В отличие от Consumption/Premium, масштабирование не event-driven.

---

## Timeout (Ограничения по времени)

| Timeframe | Duration |
|------------|----------|
| **Default** | 30 минут |
| **Maximum** | Без ограничений (при включённом Always On) |

⚠️ Для неограниченного timeout необходимо включить **Always On**.

---

## When to Use (Когда использовать)

✅ Нужна полностью предсказуемая стоимость  
✅ Уже есть недогруженный App Service Plan  
✅ Нужно размещать Web Apps и Functions вместе  
✅ Требуется ручной контроль масштабирования  
✅ Нужен большой размер вычислений  
✅ Требуется App Service Environment  
✅ Высокое потребление памяти

---

## App Service Environment (ASE)

- 🏢 Полностью изолированная среда
- 📈 Масштабирование до 100 инстансов
- 🔐 Приватная сеть
- 📜 Подходит для compliance-требований

---

## Важно для AZ-204

- Dedicated = фиксированная оплата независимо от выполнения
- Нет автоматического масштабирования по событиям
- Always On обязателен для длительных задач
- Подходит для постоянных фоновых процессов
- Можно использовать вместе с Web Apps в одном плане

### CLI Example
```bash
# Create App Service Plan
az appservice plan create \
  --name <plan-name> \
  --resource-group <rg-name> \
  --location <region> \
  --sku S1 \
  --is-linux

# Create function app on Dedicated plan
az functionapp create \
  --name <app-name> \
  --resource-group <rg-name> \
  --plan <plan-name> \
  --runtime python \
  --runtime-version 3.9 \
  --storage-account <storage-name>

# Enable Always On (required for unbounded timeout)
az functionapp config set \
  --name <app-name> \
  --resource-group <rg-name> \
  --always-on true
```
# Container Apps (Azure Container Apps для Functions)

## Characteristics (Характеристики)

- 🐳 **Containerized functions** — запуск функций в кастомных контейнерах
- ☁️ Размещение в **Azure Container Apps** (полностью управляемая среда)
- ⚡ Event-driven serverless модель (поддержка Functions programming model)
- 🧩 Подходит для микросервисной архитектуры
- 📦 Возможность использовать **custom images** с нужными зависимостями

> 💡 Позволяет запускать функции рядом с API, web apps и другими контейнерными сервисами.

---

## Scaling Behavior (Поведение масштабирования)

| Aspect | Details |
|----------|----------|
| **Scale-out** | Автоматическое, по событиям |
| **Max instances** | 10–300 (настраивается) |
| **Min instances** | Настраивается (может быть 0) |
| **Scale to zero** | Да (если min replicas = 0) |

---

## Timeout (Ограничения по времени)

| Timeframe | Duration |
|------------|----------|
| **Default** | 30 минут |
| **Maximum** | Без ограничений (зависит от триггера, если min replicas = 0) |

---

## When to Use (Когда использовать)

✅ Требуются кастомные библиотеки или зависимости  
✅ Миграция с on-premises в контейнерную модель  
✅ Нужно избежать управления Kubernetes  
✅ Требуются более мощные CPU-ресурсы  
✅ Функции работают вместе с другими микросервисами  
✅ Нужны кастомные Linux-образы

---

## Важно для AZ-204

- Container Apps поддерживает serverless scaling
- Можно масштабироваться до нуля
- Подходит для контейнерной архитектуры без Kubernetes
- Хороший выбор при сложных зависимостях
- Поддерживает гибкую конфигурацию ресурсов

### CLI Example
```bash
# Create Container Apps environment
az containerapp env create \
  --name <env-name> \
  --resource-group <rg-name> \
  --location <region>

# Create function app on Container Apps
az functionapp create \
  --name <app-name> \
  --resource-group <rg-name> \
  --environment <env-name> \
  --image <docker-image> \
  --min-replicas 0 \
  --max-replicas 30
```

# Hosting Plan Comparison (Сравнение планов размещения)

## Feature Matrix (Матрица возможностей)

| Feature | Consumption | Flex Consumption | Premium | Dedicated | Container Apps |
|----------|-------------|------------------|---------|-----------|----------------|
| **Auto scale** | ✅ | ✅ | ✅ | ⚠️ Manual / Autoscale rules | ✅ |
| **Max timeout** | 10 мин | Без лимита | Без лимита | Без лимита | Без лимита |
| **VNet** | ❌ | ✅ | ✅ | ✅ | ✅ |
| **Cold start** | Возможен | Снижен | ❌ Нет | ❌ Нет | Возможен |
| **Billing** | Per-execution | Per-execution | Fixed | Fixed | Per-execution |
| **Linux containers** | ❌ | ❌ | ✅ | ✅ | ✅ |
| **Custom image** | ❌ | ❌ | ✅ | ✅ | ✅ |

> 💡 Consumption — самый дешёвый для нерегулярной нагрузки.  
> Premium — лучший баланс производительности и serverless.

---

## Cost Comparison (Оценка стоимости в месяц)

| Plan | Light Usage | Medium Usage | Heavy Usage |
|-------|-------------|--------------|-------------|
| **Consumption** | $0–20 | $50–200 | $500+ |
| **Flex Consumption** | $0–30 | $60–250 | $600+ |
| **Premium (EP1)** | ~$146 | ~$146 | $146+ |
| **Dedicated (S1)** | ~$70 | ~$70 | $70+ |
| **Container Apps** | $0–40 | $80–300 | $800+ |

💡 Consumption наиболее выгоден при редких вызовах.  
Premium и Dedicated выгоднее при постоянной нагрузке.

---

## Max Instances Comparison (Максимальное число инстансов)

| Plan | Windows | Linux |
|--------|----------|--------|
| **Consumption** | 200 | 100 |
| **Flex Consumption** | Ограничено памятью региона | Ограничено памятью региона |
| **Premium** | До 100 | 20–100 |
| **Dedicated** | 10–30 | 10–30 (до 100 с ASE) |
| **Container Apps** | 10–300 | 10–300 |

---

## Важно для AZ-204

- Consumption ограничен 10 минутами выполнения
- Premium устраняет cold start
- Flex Consumption поддерживает VNet и больше контроля
- Dedicated требует Always On для неограниченного timeout
- Container Apps подходят для контейнерной архитектуры
- Выбор плана зависит от:
  - Частоты вызовов
  - Требований к сети
  - Времени выполнения
  - Чувствительности к cold start

## Function Timeout Configuration

### host.json Settings
```json
{
  "version": "2.0",
  "functionTimeout": "00:05:00",  // 5 minutes (Consumption default)
  "extensions": {}
}
```

### Timeout Values by Plan
```json
// Consumption
"functionTimeout": "00:10:00"  // Max 10 minutes

// Flex Consumption, Premium, Dedicated
"functionTimeout": "00:30:00"  // Default 30 minutes, unbounded max

// No timeout (Durable Functions pattern)
// Use async HTTP pattern for long-running operations
```

## Choosing the Right Plan

### Decision Tree
```
START: What are your requirements?

Need predictable billing?
├─ Yes → Dedicated Plan or Premium Plan
└─ No → Continue

Need VNet connectivity?
├─ Yes → Premium, Flex Consumption, or Container Apps
└─ No → Continue

Functions run > 10 minutes?
├─ Yes → Premium, Dedicated, or Flex Consumption
└─ No → Continue

Need to eliminate cold starts?
├─ Yes → Premium Plan
└─ No → Continue

Using custom containers?
├─ Yes → Container Apps or Premium
└─ No → Continue

Result: Consumption Plan (best cost/simplicity)
```

## Common Scenarios (Типовые сценарии)

### Scenario 1: HTTP API (низкая нагрузка)

**Лучший план**: Consumption

- Нерегулярные запросы
- Время выполнения < 10 минут
- Минимальная стоимость

---

### Scenario 2: High-frequency processing (частая обработка)

**Лучший план**: Premium

- Отсутствие cold start
- Стабильная производительность
- Поддержка VNet

---

### Scenario 3: Background jobs (фоновые задачи)

**Лучший план**: Dedicated (существующий App Service Plan)

- Запуск рядом с Web App
- Предсказуемая стоимость
- Ручной контроль масштабирования

---

### Scenario 4: Microservices architecture (микросервисная архитектура)

**Лучший план**: Container Apps

- Кастомные зависимости
- Запуск вместе с другими сервисами
- Контейнерная модель деплоя

---

# Critical Notes (Критически важные моменты)

- 💡 **Consumption — план по умолчанию**  
  Лучший выбор для нерегулярной нагрузки
- ⚠️ **HTTP-лимит 230 секунд**  
  Ограничение Azure Load Balancer (для всех планов)
- 🎯 **Always On требуется**  
  Для неограниченного timeout в Dedicated
- 📊 **Premium prewarmed**  
  Нет cold start — лучше для production
- ✅ **VNet поддерживается** в:
  - Flex Consumption
  - Premium
  - Dedicated
  - Container Apps
- 🔄 План можно изменить (с ограничениями)
- ⏱️ Default timeout:
  - 5 минут (Consumption)
  - 30 минут (остальные)
- 🔒 ASE используется для полной изоляции (Dedicated)

---

# Exam Tips (Советы для экзамена)

- **Consumption**:
  - План по умолчанию
  - Pay-per-execution
  - Максимум 10 минут
- **Flex Consumption**:
  - Улучшенный Consumption
  - VNet
  - Масштабирование на уровне функции
- **Premium**:
  - Нет cold start (prewarmed)
  - VNet
  - Неограниченное время выполнения
- **Dedicated**:
  - Работает в App Service Plan
  - Фиксированная стоимость
  - Always On для неограниченного timeout
- **Container Apps**:
  - Кастомные образы
  - Подходит для микросервисов

---

## Важные лимиты

- HTTP timeout: **230 секунд**
- Consumption max instances:
  - Windows — 200
  - Linux — 100
- Premium max instances:
  - Windows — 100
  - Linux — 20–100
- VNet не поддерживается в обычном Consumption
- Cold start возможен в:
  - Consumption
  - Flex Consumption
  - Container Apps
- Cold start отсутствует в:
  - Premium
  - Dedicated
- Для длительных операций используйте **Durable Functions**

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-functions/3-compare-azure-functions-hosting-options)
