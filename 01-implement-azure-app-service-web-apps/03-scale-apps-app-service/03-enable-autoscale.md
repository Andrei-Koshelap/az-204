# Enable Autoscale in App Service (Включение автомасштабирования в App Service)

## Key Concepts (Ключевые понятия)

- **Custom autoscale** — масштабирование по метрикам или расписанию
- **Scale conditions** — правила, определяющие когда и как масштабировать
- **Default condition** — применяется, если другие условия не активны
- **Run history** — журнал событий масштабирования

> 💡 Autoscale настраивается на уровне **App Service Plan**, а не конкретного Web App.

---

## Prerequisites (Предварительные требования)

### Required Pricing Tier (Необходимый тариф)

Autoscale доступен только начиная с **Standard (S1)**:

| Tier | Autoscale Support |
|------|------------------|
| **F1 (Free)** | ❌ Только 1 инстанс |
| **D1 (Shared)** | ❌ Только 1 инстанс |
| **B1 (Basic)** | ❌ Только ручное масштабирование |
| **S1+ (Standard)** | ✅ Полная поддержка autoscale |
| **P1V2+ (Premium)** | ✅ Полная поддержка autoscale |

⚠️ Перед включением autoscale необходимо перейти на **S1 или выше**.

---

## Enable Autoscale in Portal (Настройка через Azure Portal)

### Step-by-Step Process (Пошаговая настройка)

### 1️⃣ Перейти к App Service Plan

- Открыть **Azure Portal**
- Выбрать **App Service Plan**  
  (не Web App, а именно план)

---

### 2️⃣ Открыть настройки масштабирования

- В левом меню → **Settings**
- Выбрать **Scale out (App Service plan)**

---

### 3️⃣ Выбрать метод масштабирования

- В разделе **Scale out method** выбрать:
   - **Rules Based**
- Нажать **Configure**

---

### 4️⃣ Включить Custom Autoscale

- Выбрать опцию **Custom autoscale**
- Откроются группы условий (condition groups)

---

## Важно для AZ-204

- Autoscale настраивается на уровне плана, поэтому влияет на все приложения в этом плане
- Для независимого масштабирования приложений нужен PremiumV2/V3 с Automatic Scaling
- Run history помогает анализировать, почему произошло масштабирование
- После изменения тарифа возможен рестарт инстансов


### Portal View
```
App Service Plan → Settings → Scale out
├── Manual scale (default)
├── Rules Based → Configure
│   └── Custom autoscale ← Enable this
│       ├── Default scale condition
│       └── + Add a scale condition
```

## Configure Scale Conditions (Настройка условий масштабирования)

### Default Scale Condition (Условие по умолчанию)

- ✅ Активно всегда, если нет других активных условий
- ❌ Нельзя удалить
- ✏️ Можно редактировать и настраивать правила
- 🔁 Используется как fallback (например, ночью или вне расписания)

> 💡 Default condition гарантирует, что масштабирование всегда имеет базовую конфигурацию.

---

### Custom Scale Conditions (Пользовательские условия)

- Создаются для конкретных сценариев
- Могут быть:
   - **Metric-based** (по метрикам)
   - **Schedule-based** (по расписанию)
- Выполняются, когда активен их schedule
- При срабатывании переопределяют default condition

> 🎯 Полезно для сценариев «рабочие часы / вне рабочих часов».

---

### Condition Properties (Свойства условия)

| Property | Description | Required |
|-----------|-------------|----------|
| **Condition name** | Описательное имя условия | Yes |
| **Scale mode** | Metric-based или Specific count | Yes |
| **Instance limits** | Минимум, максимум, значение по умолчанию | Yes |
| **Schedule** | Когда условие активно | Optional |
| **Rules** | Правила scale-out и scale-in | Yes (для metric-based) |

---

## Дополнительно (Важно для AZ-204)

- Instance limits ограничивают диапазон масштабирования
- Default instance count используется при старте условия
- Schedule-based условия удобны для предсказуемых нагрузок
- Metric-based условия позволяют реагировать на реальные метрики
- Если несколько условий активны одновременно — применяется соответствующее расписанию

> ⚠️ Min instance всегда должен быть ≥ 1

### Example Conditions
```
Condition 1: Business Hours
- Schedule: Mon-Fri, 9 AM - 5 PM
- Min: 3, Max: 10, Default: 3
- Rules: CPU-based autoscale

Condition 2: Weekend
- Schedule: Sat-Sun, All day
- Min: 1, Max: 3, Default: 2
- Rules: HTTP queue-based

Default Condition:
- No schedule (always active otherwise)
- Min: 2, Max: 5, Default: 2
- Rules: Basic CPU monitoring
```

## Create Autoscale Rules (Создание правил автомасштабирования)

### Add Rules in Portal (Добавление правила в портале)

1. Открыть нужное **scale condition**
2. Нажать **+ Add a rule**
3. Настроить критерии правила (метрика и порог)
4. Задать действие масштабирования (scale action)
5. Сохранить конфигурацию

---

### Rule Configuration Fields (Поля конфигурации правила)

| Field | Description | Example |
|--------|-------------|----------|
| **Metric source** | Ресурс, откуда берётся метрика | App Service Plan |
| **Metric name** | Тип метрики | CPU Percentage |
| **Time grain** | Период агрегации метрики | 1 minute |
| **Statistic** | Тип агрегации (Average, Max и т.д.) | Average |
| **Operator** | Условие сравнения | Greater than |
| **Threshold** | Пороговое значение | 70 |
| **Duration** | Окно анализа | 5 minutes |
| **Operation** | Scale Out / Scale In | Increase count by 1 |
| **Cooldown** | Период стабилизации | 5 minutes |

---

## Пример правила (Scale Out)

- Metric: CPU Percentage
- Time grain: 1 minute
- Statistic: Average
- Operator: Greater than
- Threshold: 70%
- Duration: 5 minutes
- Action: Increase by 1 instance
- Cooldown: 5 minutes

---

## Важно для AZ-204

- Для каждого scale-out правила должно быть соответствующее scale-in
- Duration и Cooldown — разные параметры
- Threshold должен учитывать реальные рабочие нагрузки
- Можно увеличивать/уменьшать не на 1, а на несколько инстансов
- Metric source может быть другим Azure ресурсом (например, очередь)

> 💡 Правильная настройка правил предотвращает частые колебания масштабирования.


#### Metric Source
```
Resource: Current App Service Plan
Metric namespace: App Service Plan standard metrics
Metric name: CPU Percentage, Memory Percentage, etc.
```

#### Criteria
```
Time aggregation: Average, Minimum, Maximum, Sum
Operator: Greater than, Less than, Equal to
Threshold: Numeric value (e.g., 70)
Duration: Minutes to evaluate (minimum 5)
Time grain: Sampling interval (typically 1 minute)
```

#### Action
```
Operation: Increase/Decrease/Set to
Instance count: Number to change by
Cool down: Minutes to wait (minimum 5)
```

### Portal Screenshot Reference
```
┌─────────────────────────────────────┐
│ Scale rule                          │
├─────────────────────────────────────┤
│ Metric source                       │
│   Resource: [App Service Plan]     │
├─────────────────────────────────────┤
│ Criteria                            │
│   Metric: CPU Percentage            │
│   Operator: Greater than            │
│   Threshold: 70                     │
│   Duration: 10 minutes              │
├─────────────────────────────────────┤
│ Action                              │
│   Operation: Increase by            │
│   Instance count: 1                 │
│   Cool down: 5 minutes              │
└─────────────────────────────────────┘
```

## CLI Configuration

### Create Autoscale Setting
```bash
# Create autoscale setting with instance limits
az monitor autoscale create \
  --resource-group <rg-name> \
  --resource <app-service-plan-id> \
  --name MyAutoscaleSetting \
  --min-count 2 \
  --max-count 10 \
  --count 3
```

### Add Scale-Out Rule
```bash
# Scale out when CPU > 70%
az monitor autoscale rule create \
  --autoscale-name MyAutoscaleSetting \
  --resource-group <rg-name> \
  --condition "Percentage CPU > 70 avg 10m" \
  --scale out 1 \
  --cooldown 5
```

### Add Scale-In Rule
```bash
# Scale in when CPU < 30%
az monitor autoscale rule create \
  --autoscale-name MyAutoscaleSetting \
  --resource-group <rg-name> \
  --condition "Percentage CPU < 30 avg 10m" \
  --scale in 1 \
  --cooldown 5
```

### Add Schedule-Based Condition
```bash
# Create condition for business hours
az monitor autoscale rule create \
  --autoscale-name MyAutoscaleSetting \
  --resource-group <rg-name> \
  --condition "Percentage CPU > 60 avg 5m" \
  --scale out 2 \
  --cooldown 5 \
  --timegrain "PT1M" \
  --schedule "0 9 * * 1-5"
```

### List Current Settings
```bash
# View autoscale settings
az monitor autoscale show \
  --name MyAutoscaleSetting \
  --resource-group <rg-name>

# List all rules
az monitor autoscale rule list \
  --autoscale-name MyAutoscaleSetting \
  --resource-group <rg-name>
```

## Monitor Autoscale Activity (Мониторинг автомасштабирования)

### Run History Chart (История выполнения)

Расположение в портале:  
`App Service Plan → Scale out → Run history`

Отображает:

- 📈 **Timeline** — временная шкала событий масштабирования
- 🔢 Изменение **количества инстансов**
- 📋 Условие, которое вызвало масштабирование
- ✅/❌ Статус выполнения (успех или ошибка)

> 💡 Полезно для анализа, почему произошло scale out/in.

---

### Metrics Integration (Связь с метриками)

События масштабирования можно сопоставить с метриками:

- **CPU Percentage** — нагрузка на процессор
- **Memory Percentage** — использование памяти
- **Data In / Data Out** — сетевой трафик
- **HTTP Queue Length** — количество ожидающих запросов

> 🎯 Хорошая практика — анализировать метрики до и после масштабирования.

---

### Activity Log (Журнал активности)

Все события autoscale фиксируются в **Azure Activity Log**:

- Инициация масштабирования
- Завершённые операции
- Ошибки масштабирования
- Проблемы с доступностью метрик

> ⚠️ Если масштабирование не происходит — проверьте Activity Log и доступность метрик.

---

## Важно для AZ-204

- Run history помогает понять, какое правило сработало
- Activity Log используется для диагностики ошибок
- Отсутствие метрик может блокировать autoscale
- Анализ метрик обязателен перед изменением порогов


### Query Activity Log
```bash
# Get recent autoscale events
az monitor activity-log list \
  --resource-group <rg-name> \
  --namespace "Microsoft.Insights/AutoscaleSettings" \
  --start-time "2024-01-01" \
  --max-events 50
```

## Configure Notifications (Настройка уведомлений)

Autoscale может отправлять уведомления при выполнении действий масштабирования.

---

### Notification Options (Варианты уведомлений)

1️⃣ **Email**

- Отправка уведомлений на указанные адреса
- Можно включить уведомления для:
   - Подписчиков Azure
   - Администраторов
   - Конкретных email-адресов
- Уведомляет о scale-out, scale-in и ошибках

---

2️⃣ **Webhook**

- Отправка **HTTP POST** запроса на указанный endpoint
- Подходит для:
   - Интеграции с внешними системами
   - CI/CD процессов
   - ITSM/Service Desk систем
- Позволяет автоматизировать реакции на масштабирование

> 💡 Часто используется вместе с Azure Functions или Logic Apps.

---

3️⃣ **Activity Log Alerts**

- Интеграция с **Azure Monitor**
- Создание alert rules на основе событий autoscale
- Можно настроить:
   - Email
   - SMS
   - Push
   - Action Groups

---

## Важно для AZ-204

- Уведомления помогают отслеживать неожиданные масштабирования
- Webhook даёт возможность автоматизировать реакцию
- Activity Log Alerts — более гибкий и production-ready вариант
- Для сложных сценариев лучше использовать Action Groups

### Configure in Portal
```
App Service Plan → Scale out → Notify tab
├── Email
│   └── Add email addresses
├── Webhook
│   └── Configure endpoint URL
└── Service administrators (checkbox)
```

### CLI Configuration
```bash
# Add email notification
az monitor autoscale update \
  --name MyAutoscaleSetting \
  --resource-group <rg-name> \
  --add-action email admin@company.com

# Add webhook notification
az monitor autoscale update \
  --name MyAutoscaleSetting \
  --resource-group <rg-name> \
  --add-action webhook https://myapp.com/webhook
```

### Create Activity Log Alert
```bash
# Alert on autoscale events
az monitor activity-log alert create \
  --name AutoscaleAlert \
  --resource-group <rg-name> \
  --condition category=Autoscale \
  --action-group <action-group-id>
```

## Complete Configuration Example

### Portal Workflow
```
1. Scale up to S1 tier (if needed)
2. Navigate to App Service Plan → Scale out
3. Select "Rules Based" → Configure
4. Enable "Custom autoscale"
5. Configure default condition:
   - Min: 2, Max: 10, Default: 2
   - Add scale-out rule: CPU > 70%
   - Add scale-in rule: CPU < 30%
6. Add business hours condition:
   - Schedule: Mon-Fri 9-5
   - Min: 3, Max: 15, Default: 5
   - Rules: CPU and HTTP queue
7. Configure notifications
8. Save settings
```

### CLI Equivalent
```bash
# 1. Scale up (if needed)
az appservice plan update \
  --name <plan-name> \
  --resource-group <rg-name> \
  --sku S1

# 2. Create autoscale setting
az monitor autoscale create \
  --resource-group <rg-name> \
  --resource <plan-id> \
  --name ProductionAutoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2

# 3. Add default rules
az monitor autoscale rule create \
  --autoscale-name ProductionAutoscale \
  --resource-group <rg-name> \
  --condition "Percentage CPU > 70 avg 10m" \
  --scale out 1 \
  --cooldown 5

az monitor autoscale rule create \
  --autoscale-name ProductionAutoscale \
  --resource-group <rg-name> \
  --condition "Percentage CPU < 30 avg 10m" \
  --scale in 1 \
  --cooldown 5

# 4. Configure notifications
az monitor autoscale update \
  --name ProductionAutoscale \
  --resource-group <rg-name> \
  --add-action email devops@company.com
```

## Critical Notes (Критически важные моменты)

- 💡 **Требование по тарифу** — минимум S1 (Standard) или выше
- ⚠️ Настраивается на уровне **App Service Plan**, а не Web App
- 🎯 **Default condition** — должен быть безопасным fallback-сценарием
- 📊 Перед настройкой порогов сначала проанализируйте метрики
- ✅ Тестируйте постепенно — начните с консервативных значений
- 🔔 Включите уведомления о событиях масштабирования
- ⏱️ Используйте **Run history** для проверки корректности работы

---

## Exam Tips (Советы для экзамена)

- Autoscale настраивается на уровне **App Service Plan**
- Минимальный тариф — Standard (S1) или Premium
- Default condition всегда активен и работает как fallback
- Custom conditions переопределяют default при активном расписании
- Все события фиксируются в **Azure Activity Log**
- Уведомления доступны через email, webhook и action groups
- Run history показывает, какое условие вызвало масштабирование
- Путь в портале:  
  `App Service Plan → Settings → Scale out`


[Learn More](https://learn.microsoft.com/en-us/training/modules/scale-apps-app-service/4-autoscale-app-service)
