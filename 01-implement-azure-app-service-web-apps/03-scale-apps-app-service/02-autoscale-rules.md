# Autoscale Conditions and Rules (Условия и правила автомасштабирования)

## Key Concepts (Ключевые понятия)

- **Autoscale conditions** — условия, определяющие *когда* и *как* выполнять масштабирование
- **Autoscale rules** — пороговые значения метрик, запускающие масштабирование
- **Time grain** — период агрегации метрики (обычно 1 минута)
- **Duration** — окно времени для оценки (минимум 5 минут)
- **Cooldown period** — пауза между действиями масштабирования (минимум 5 минут)

---

## Autoscale Conditions (Условия автомасштабирования)

### Types (Типы)

1. **Metric-based** — масштабирование по метрикам
2. **Schedule-based** — масштабирование по расписанию
3. **Default condition** — условие по умолчанию (без расписания)

---

### Combining Conditions (Комбинирование условий)

- ✅ Можно создавать несколько условий
- ✅ В одном условии можно комбинировать метрики и расписание
- ✅ Масштабирование происходит, если выполняется **любое** активное условие
- ✅ Default condition применяется, если нет других активных условий

> 💡 Это означает логическое **OR** между условиями.

---

## Metrics for Autoscale (Метрики для масштабирования)

| Metric | Description | High Value Indicates |
|--------|-------------|---------------------|
| **CPU Percentage** | Использование CPU на всех инстансах | CPU-bound, возможны задержки |
| **Memory Percentage** | Использование памяти | Недостаток памяти, возможные сбои |
| **Disk Queue Length** | Количество ожидающих I/O операций | Конкуренция за диск |
| **HTTP Queue Length** | Количество ожидающих HTTP-запросов | Возможны тайм-ауты (HTTP 408) |
| **Data In** | Входящий трафик (байты) | Высокая входящая нагрузка |
| **Data Out** | Исходящий трафик (байты) | Высокая исходящая нагрузка |

> 📌 Можно использовать метрики других сервисов Azure  
> (например, длина очереди Service Bus или Storage Queue)

---

## How Autoscale Analyzes Metrics (Как анализируются метрики)

### Two-Step Process (Двухэтапный процесс)

---

### Step 1: Time Grain Aggregation

- **Период**: обычно 1 минута
- **Тип агрегации**:
    - Average
    - Min
    - Max
    - Sum
    - Last
    - Count
- **Результат**: одно агрегированное значение в минуту по всем инстансам

> 💡 На этом этапе данные "сжимаются" в поминутные значения.

---

### Step 2: Duration Aggregation

- **Период**: минимум 5 минут (задаётся пользователем)
- **Тип агрегации**: может отличаться от Time grain  
  (например, *Maximum of Averages*)
- **Результат**: итоговое значение, сравниваемое с порогом

---

## Пример логики

Если правило задано так:

- CPU > 70%
- Duration = 10 минут
- Time grain = 1 минута
- Aggregation = Average

Тогда:

1. Azure считает средний CPU за каждую минуту
2. Затем берёт среднее значение за последние 10 минут
3. Если результат > 70% → выполняется scale out

---

## Важно для AZ-204

- Duration всегда ≥ 5 минут
- Cooldown предотвращает частые scale in/out
- Можно комбинировать разные агрегации
- Метрики анализируются по всем инстансам App Service Plan
- Autoscale — это горизонтальное масштабирование

### Example
```
Metric: CPU Percentage
Time Grain: 1 minute (Average)
Duration: 10 minutes (Maximum)

Process:
1. Each minute: Average CPU across all instances
2. Over 10 minutes: Get maximum of those 10 averages
3. Compare result to threshold (e.g., 70%)
```

## Autoscale Actions (Действия автомасштабирования)

### Action Types (Типы действий)

- **Scale Out** — увеличение количества инстансов
- **Scale In** — уменьшение количества инстансов
- **Set to specific count** — установка фиксированного количества инстансов

> 💡 Autoscale влияет только на количество инстансов,  
> размер (CPU/RAM) остаётся прежним.

---

### Operators (Операторы условий)

| Action | Typical Operator | Example |
|--------|------------------|---------|
| **Scale Out** | Greater than (>) | CPU > 70% |
| **Scale In** | Less than (<) | CPU < 30% |

> 🎯 Обычно используют разные пороги для Scale Out и Scale In,  
> чтобы избежать частого переключения (flapping).

---

## Cooldown Period (Период ожидания)

- **Назначение**: дать системе стабилизироваться после масштабирования
- **Минимум**: 5 минут
- **Предотвращает**: частые и резкие изменения количества инстансов
- **Почему нужен**:
    - запуск нового инстанса занимает время
    - остановка инстанса тоже не мгновенная
    - метрики могут кратковременно колебаться

> ⚠️ Без cooldown возможны постоянные Scale Out / Scale In при нестабильной нагрузке.

---

## Важно для AZ-204

- Cooldown применяется после каждого действия масштабирования
- Порог для Scale Out должен быть выше, чем для Scale In
- Duration и Cooldown — разные параметры
- Можно задать увеличение/уменьшение на определённое количество инстансов (например, +1, −1)

```bash
# Example autoscale rule
az monitor autoscale rule create \
  --autoscale-name <autoscale-name> \
  --resource-group <rg-name> \
  --condition "Percentage CPU > 70 avg 10m" \
  --scale out 1 \
  --cooldown 5
```

## Pairing Rules

### Best Practice: Define Pairs
Always create pairs of rules:
1. **Scale-out rule** - When to add instances
2. **Scale-in rule** - When to remove instances

### Example Pair
```bash
# Scale out rule
Metric: CPU Percentage
Operator: Greater than
Threshold: 70%
Action: Increase by 1
Cooldown: 5 minutes

# Scale in rule
Metric: CPU Percentage
Operator: Less than
Threshold: 30%
Action: Decrease by 1
Cooldown: 5 minutes
```

⚠️ **Important**: Используйте разные пороги для Scale Out и Scale In,  
чтобы избежать flapping (например, 70% для scale out и 30% для scale in)

---

## Combining Rules in Same Condition (Комбинирование правил в одном условии)

В рамках одного условия можно задать несколько правил.

### Scale-Out Logic: OR (Логика масштабирования вверх)

Scale out выполняется, если срабатывает **любое** правило:

- HTTP queue > 10  
  **OR**
- CPU > 70%

> 💡 Достаточно превышения одного показателя, чтобы добавить инстансы.

---

### Scale-In Logic: AND (Логика масштабирования вниз)

Scale in выполняется только если выполняются **все** правила:

- HTTP queue = 0  
  **AND**
- CPU < 50%

> 🎯 Это предотвращает преждевременное уменьшение количества инстансов,  
> если нагрузка всё ещё сохраняется по одному из показателей.

---

## Важно для AZ-204

- Scale Out → логика **OR**
- Scale In → логика **AND**
- Это сделано для обеспечения стабильности и отказоустойчивости
- Неправильная настройка может привести к oscillation (частым колебаниям масштабирования)


### Example Configuration
```bash
Condition: Business Hours (9 AM - 5 PM)

Rules:
1. If HTTP queue > 10 → Scale out by 1
2. If CPU > 70% → Scale out by 1
3. If HTTP queue = 0 → Scale in by 1
4. If CPU < 50% → Scale in by 1

Result:
- Scale out: HTTP queue > 10 OR CPU > 70%
- Scale in: HTTP queue = 0 AND CPU < 50%
```

### Separate Conditions for OR on Scale-In
If you need OR logic for scale-in, use separate conditions:

```
Condition 1: HTTP-based scaling
- Scale in if HTTP queue = 0

Condition 2: CPU-based scaling
- Scale in if CPU < 50%
```

## Configuration Example

```bash
# Create autoscale setting
az monitor autoscale create \
  --resource-group <rg-name> \
  --resource <plan-id> \
  --name MyAutoscale \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Add scale-out rule (CPU)
az monitor autoscale rule create \
  --autoscale-name MyAutoscale \
  --resource-group <rg-name> \
  --condition "Percentage CPU > 70 avg 10m" \
  --scale out 1 \
  --cooldown 5

# Add scale-in rule (CPU)
az monitor autoscale rule create \
  --autoscale-name MyAutoscale \
  --resource-group <rg-name> \
  --condition "Percentage CPU < 30 avg 10m" \
  --scale in 1 \
  --cooldown 5

# Add scale-out rule (HTTP queue)
az monitor autoscale rule create \
  --autoscale-name MyAutoscale \
  --resource-group <rg-name> \
  --condition "Http Queue Length > 10 avg 5m" \
  --scale out 2 \
  --cooldown 5
```

## Quick Reference (Краткая выжимка)

### Time Settings (Параметры времени)

| Setting | Minimum | Typical | Purpose |
|----------|---------|---------|----------|
| **Time Grain** | 1 minute | 1 minute | Сбор и агрегация метрик |
| **Duration** | 5 minutes | 5–10 minutes | Окно анализа |
| **Cooldown** | 5 minutes | 5–10 minutes | Период стабилизации |

> 💡 Time Grain — это частота сбора данных,  
> Duration — окно анализа перед сравнением с порогом,  
> Cooldown — пауза после масштабирования.

---

### Aggregation Options (Типы агрегации)

- **Average** — среднее значение
- **Minimum** — минимальное значение
- **Maximum** — максимальное значение
- **Sum** — суммарное значение
- **Last** — последнее зафиксированное значение
- **Count** — количество точек данных

> 🎯 Можно комбинировать разные агрегации для Time Grain и Duration  
> (например, Maximum of Averages).

---

## Critical Notes (Критически важные моменты)

- 💡 Всегда настраивайте правила парами: scale-out и scale-in
- ⚠️ Используйте разные пороги, чтобы избежать flapping (например, 70% / 30%)
- 🎯 Минимальный cooldown — 5 минут
- 📊 Минимальный duration — 5 минут для анализа тренда
- 🔄 Scale-out работает по логике **OR**
- 🔄 Scale-in работает по логике **AND**
- ⏱️ Учитывайте время запуска новых инстансов при выборе cooldown

---

## Exam Tips (Советы для экзамена)

- Понимать двухэтапный анализ метрик:  
  **Time Grain → Duration → Threshold comparison**
- Запомнить:
    - Scale-out = OR
    - Scale-in = AND
- Минимальный cooldown — 5 минут
- Знать основные метрики App Service (CPU, Memory, HTTP Queue, Disk Queue, Data In/Out)
- Понимать важность парных правил
- Для логики OR при scale-in требуются отдельные условия


[Learn More](https://learn.microsoft.com/en-us/training/modules/scale-apps-app-service/3-app-service-autoscale-conditions-rules)
