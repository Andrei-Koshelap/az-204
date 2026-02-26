# Examine Autoscale Options (Механизмы автомасштабирования)

## Key Concepts (Ключевые понятия)

- **Autoscaling** — автоматическое добавление или удаление инстансов в зависимости от нагрузки
- **Scale Out / In** — горизонтальное масштабирование (добавление/удаление инстансов)
- **Scale Up / Down** — вертикальное масштабирование (изменение размера инстанса)
- **Два автоматических варианта**:
    - **Autoscale** (на основе правил)
    - **Automatic scaling** (управляется платформой)

---

## Autoscaling vs Automatic Scaling

| Feature | **Autoscale** | **Automatic Scaling** |
|----------|---------------|----------------------|
| **Tiers** | Standard+ | PremiumV2, PremiumV3 |
| **Rule-based** | ✅ Yes | ❌ No (platform-managed) |
| **Schedule-based** | ✅ Yes | ❌ No |
| **Always ready instances** | ❌ No | ✅ Yes (min 1) |
| **Prewarmed instances** | ❌ No | ✅ Yes (default 1) |
| **Per-app maximum** | ❌ No | ✅ Yes |
| **Configuration** | Manual rules | Platform decides |

> 💡 Automatic Scaling доступен только в PremiumV2 / PremiumV3 и работает на уровне приложения.

---

## When to Use Autoscaling (Когда использовать Autoscale)

### ✅ Подходит для

- Предсказуемых паттернов (например, праздничные пики)
- Переменной нагрузки (рабочие часы vs ночь)
- Оптимизации затрат (scale-in при низкой нагрузке)
- Масштабирования по нескольким метрикам (CPU, память, очередь)
- Повышения отказоустойчивости

---

### ❌ Не подходит для

- Ресурсоёмкой обработки каждого запроса
- Долгосрочного линейного роста (лучше вручную scale up)
- Защиты от DoS-атак (нужно использовать фильтрацию)
- Сценариев с минимальным числом инстансов (сначала увеличить baseline)
- Ситуаций, где мониторинг дороже выгоды

---

## Autoscaling Fundamentals (Основы Autoscale)

### Как это работает

1. Мониторинг метрик (CPU, память, HTTP requests и т.д.)
2. Сравнение со значениями из правил
3. Выполнение действия масштабирования
4. Период cooldown (ожидание перед следующим действием)
5. Балансировка нагрузки между инстансами

---

### Scale Actions

- **Scale Out** — увеличение количества инстансов
- **Scale In** — уменьшение количества инстансов
- ❗ Не влияет на ресурсы одного инстанса (CPU, RAM, storage)

---

## Automatic Scaling (PremiumV2 / PremiumV3)

### Features (Возможности)

- ✅ Управляется платформой Azure
- ✅ Основано на HTTP-трафике
- ✅ Always ready instances (минимум 1)
- ✅ Prewarmed instances (по умолчанию 1)
- ✅ Масштабирование на уровне приложения (per-app scaling)

> 💡 В одном App Service Plan приложения могут масштабироваться независимо.

---

### Use Cases (Когда выбирать Automatic Scaling)

| Scenario | Why Automatic Scaling |
|----------|----------------------|
| Нет опыта настройки метрик | Платформа принимает решения |
| Нужно независимое масштабирование | Каждое приложение масштабируется отдельно |
| Ограничения backend (например, БД) | Можно задать максимум инстансов |
| Нужна простота | Нет правил для конфигурации |

---

## Дополнительно (Важно для AZ-204)

- Autoscale поддерживает **расписание (schedule-based rules)**
- Можно комбинировать несколько правил
- Cooldown предотвращает "flapping" (частое scale in/out)
- Vertical scaling (scale up) требует рестарта приложения
- Horizontal scaling не требует изменения кода (при stateless архитектуре)

---

## Exam Focus (Что важно запомнить)

- Разница между Autoscale и Automatic Scaling
- Automatic Scaling доступен только в PremiumV2/V3
- Autoscale работает на основе правил и метрик
- Scale out ≠ Scale up
- Automatic Scaling поддерживает per-app scaling
- Cooldown — важный механизм предотвращения частых изменений


## Quick Commands

```bash
# Enable autoscale (Standard+ tier)
az monitor autoscale create \
  --resource-group <rg-name> \
  --resource <app-service-plan-id> \
  --min-count 2 \
  --max-count 10 \
  --count 2

# Enable automatic scaling (PremiumV2/V3)
az webapp update \
  --resource-group <rg-name> \
  --name <app-name> \
  --enable-automatic-scaling true \
  --minimum-elastic-instance-count 1 \
  --maximum-elastic-instance-count 10

# Check autoscale settings
az monitor autoscale show \
  --resource-group <rg-name> \
  --name <autoscale-name>
```

## Tier Requirements (Требования по тарифам)

| Tier | Manual Scale | Autoscale | Automatic Scaling |
|------|--------------|-----------|-------------------|
| **Free, Shared** | ❌ No | ❌ No | ❌ No |
| **Basic** | ✅ Up to 3 | ❌ No | ❌ No |
| **Standard** | ✅ Up to 10 | ✅ Yes | ❌ No |
| **Premium** | ✅ Up to 20 | ✅ Yes | ❌ No |
| **PremiumV2** | ✅ Up to 30 | ✅ Yes | ✅ Yes |
| **PremiumV3** | ✅ Up to 30 | ✅ Yes | ✅ Yes |

> 💡 Минимальный тариф для Autoscale — **Standard**.  
> Automatic Scaling доступен только в **PremiumV2 / PremiumV3**.

---

## Critical Notes (Критически важные моменты)

- 💡 **Autoscaling = горизонтальное масштабирование** (количество инстансов),  
  не вертикальное (размер инстанса)
- ⚠️ **DoS-атаки** — использовать WAF/фильтрацию, а не масштабирование
- 🎯 Минимальный тариф для Autoscale — Standard
- 📊 Мониторинг метрик имеет накладные расходы
- 🔄 **Cooldown period** предотвращает частые скачки масштабирования
- ⚠️ Нельзя масштабироваться выше лимита App Service Plan
- 💰 Больше инстансов = выше стоимость

---

## Exam Tips (Советы для экзамена)

- Чётко различать:
    - **Scale Out / In** (горизонтально)
    - **Scale Up / Down** (вертикально)
- Понимать, когда Autoscale оправдан, а когда нет
- Запомнить: Autoscale доступен с Standard
- PremiumV2 / PremiumV3 поддерживают Automatic Scaling
- Autoscaling изменяет только количество


Как работает autoscale в Azure App Service?
Чтобы масштабирование могло происходить:

Должен быть диапазон масштабирования
(min instances ≠ max instances)


| Сценарий                    | Лучше WebJobs | Лучше Functions |
| --------------------------- | ------------- | --------------- |
| Уже есть App Service        | ✅             | ❌               |
| Постоянный worker           | ✅             | ❌               |
| Event-driven burst          | ❌             | ✅               |
| Платить только за execution | ❌             | ✅               |
| Serverless архитектура      | ❌             | ✅               |


| План        | Как платишь             |
| ----------- | ----------------------- |
| Consumption | per execution + runtime |
| Premium     | core-seconds + memory   |
| App Service | фиксированная VM цена   |


Если в вопросе есть:
no execution charges
memory allocation
core seconds
→ Premium plan

Триерм для Azure Functions, который поддерживает автоматическое масштабирование на основе количества запросов и ресурсов, используемых функциями. 

| Категория   | Примеры            |
| ----------- | ------------------ |
| HTTP        | HTTP Trigger       |
| Очереди     | Queue, Service Bus |
| Стриминг    | Event Hub, Kafka   |
| Файлы       | Blob               |
| БД          | Cosmos DB          |
| Планировщик | Timer              |
| События     | Event Grid         |

[Learn More](https://learn.microsoft.com/en-us/training/modules/scale-apps-app-service/2-autoscale-factors)
