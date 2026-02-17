# Разбор Azure App Service Plans

## Ключевые концепции

- **App Service Plan** — это набор вычислительных ресурсов (VM), на которых работают приложения.
- Несколько приложений могут использовать один App Service Plan.
- Все приложения в одном плане работают на одних и тех же VM-инстансах.
- План определяет:
    - ОС (Windows/Linux)
    - Регион
    - Количество VM
    - Размер VM
    - Ценовой уровень (Pricing Tier)

💡 Важно: App Service Plan — это единица масштабирования.

---

## Ценовые уровни (Pricing Tiers)

### Shared Compute (только Dev/Test)

| Tier | Тип VM | Scale Out | Назначение |
|------|--------|------------|------------|
| **Free** | Shared VM | ❌ Нет | Разработка/тестирование |
| **Shared** | Shared VM | ❌ Нет | Разработка/тестирование |

⚠️ Используются CPU-квоты, не подходят для production.

---

### Dedicated Compute (Production)

| Tier | Тип VM | Scale Out | Основные возможности |
|------|--------|------------|----------------------|
| **Basic** | Dedicated | ✅ До 3 | Базовая production-нагрузка |
| **Standard** | Dedicated | ✅ До 10 | Deployment slots, авто-масштабирование |
| **Premium** | Dedicated | ✅ До 20 | Улучшенная производительность |
| **PremiumV2** | Dedicated | ✅ До 30 | Более быстрые CPU, SSD |
| **PremiumV3** | Dedicated | ✅ До 30 | Новейшее оборудование, максимальная производительность |

---

### Isolated (Enterprise-уровень)

| Tier | Тип VM | Сеть | Scale Out |
|------|--------|------|------------|
| **Isolated** | Dedicated VM в выделенной VNet | ✅ Сетевая изоляция | ✅ До 100 |
| **IsolatedV2** | Dedicated VM в выделенной VNet | ✅ Сетевая изоляция | ✅ До 100 |

Используется для:
- корпоративных решений
- compliance-задач
- повышенной безопасности

---

## Как работают приложения и масштабирование

### Free/Shared

- Приложения получают ограниченное количество CPU-минут.
- Масштабирование недоступно.
- ⚠️ Не для production.

---

### Остальные уровни

- Приложения работают на **всех VM-инстансах** в плане.
- Несколько приложений в одном плане **делят ресурсы VM**.
- Deployment slots используют те же VM.
- Логи, бэкапы, WebJobs используют ресурсы плана.

💡 Если одно приложение перегружено — страдают все остальные в плане.

---

## Когда масштабировать или изолировать

### Scale Up (смена Pricing Tier)

Используется, если:
- Нужно больше CPU или RAM
- Требуются дополнительные функции (например, deployment slots)

Изменение выполняется через портал или CLI.

---

### Создать отдельный App Service Plan

Стоит изолировать приложение, если:

- ⚡ Оно ресурсоёмкое
- 🎯 Нужно независимое масштабирование
- 🌍 Требуется другой регион
- 💰 Нужен отдельный контроль затрат

---

## Оптимизация затрат

- ✅ Объединяйте несколько приложений в один план для экономии.
- ⚠️ Контролируйте нагрузку — ресурсы общие.
- 📊 Проверяйте текущую загрузку перед добавлением нового приложения.

---

## Основные команды CLI

### Создание App Service Plan

```bash
az appservice plan create \
  --name <plan-name> \
  --resource-group <rg-name> \
  --sku B1 \  # F1, B1, S1, P1V2, P1V3, I1V2
  --is-linux

## Essential Commands

```bash
# Create App Service Plan
az appservice plan create \
  --name <plan-name> \
  --resource-group <rg-name> \
  --sku B1 \  # F1, B1, S1, P1V2, P1V3, I1V2
  --is-linux

# Scale App Service Plan
az appservice plan update \
  --name <plan-name> \
  --resource-group <rg-name> \
  --number-of-workers 3

# Change pricing tier
az appservice plan update \
  --name <plan-name> \
  --resource-group <rg-name> \
  --sku S1
```

## Quick Reference
| Сценарий                   | Рекомендуемый Tier |
| -------------------------- | ------------------ |
| Dev/Test                   | Free, Shared       |
| Малый production           | Basic              |
| Production со слотами      | Standard+          |
| Высокая производительность | PremiumV3          |
| Изоляция / compliance      | IsolatedV2         |


## Critical Notes
- 💡 **Plan = Scale Unit** - All apps in plan scale together
- ⚠️ Free/Shared are **dev/test only** - use CPU quotas
- 🎯 Standard+ required for **deployment slots**
- 📊 Apps in same plan share VM resources - plan capacity accordingly
- 🌍 Each plan is **region-specific**


# Экзаменационные ловушки AZ-204 по App Service Plan

## 1️⃣ Plan = единица масштабирования

⚠️ Частая ошибка: думать, что масштабируется отдельное приложение.

На самом деле:
- Масштабируется **App Service Plan**
- Все приложения внутри плана масштабируются вместе
- Deployment slots используют те же VM

💡 Если одно приложение нагружено — страдают все остальные в том же плане.

---

## 2️⃣ Free / Shared ≠ Production

- Используют общие VM
- Имеют CPU-квоты
- Не поддерживают масштабирование
- Нет deployment slots

⚠️ Если в вопросе production-нагрузка — Free/Shared сразу исключаются.

---

## 3️⃣ Deployment Slots требуют Standard+

Если в задаче:
- требуется zero-downtime deployment
- staging environment
- swap перед production

Правильный ответ: **Standard tier или выше**

---

## 4️⃣ План привязан к региону

App Service Plan нельзя переместить в другой регион.

Если приложение должно работать:
- в другом регионе
- ближе к пользователям

Нужно создать **новый App Service Plan**.

---

## 5️⃣ Несколько приложений в одном плане делят ресурсы

Экзамен часто проверяет:

> "Одно приложение потребляет 80% CPU. Что произойдёт с другим?"

Ответ: второе приложение будет испытывать деградацию производительности.

---

## 6️⃣ Scale Up vs Scale Out

- **Scale Up** — изменение SKU (больше CPU/RAM)
- **Scale Out** — увеличение количества инстансов

⚠️ Если нужно больше функций (например, deployment slots) — требуется **смена tier**, а не просто увеличение инстансов.

---

## 7️⃣ Изоляция приложения

Если нужно:
- независимое масштабирование
- изоляция ресурсов
- контроль затрат

Правильный ответ:  
Создать **отдельный App Service Plan**.

---

## 8️⃣ Diagnostic Logs и WebJobs используют ресурсы плана

Логи, WebJobs, backup — всё потребляет ресурсы App Service Plan.

⚠️ В задачах про высокую нагрузку это может быть скрытым фактором.

---

## 9️⃣ Standard vs Premium

Premium выбирают, если:
- высокая производительность
- больше инстансов
- более современное оборудование

Не всегда нужен Premium — экзамен проверяет разумный выбор.

---

## 🔟 Isolated ≠ просто дорогой план

Isolated используется, если:
- нужна сетевая изоляция
- compliance-требования
- работа внутри VNet

Если в вопросе звучит:
- "regulatory requirements"
- "network isolation"
- "internal access only"

→ вероятный ответ: **Isolated tier (ASE)**

---

# Быстрая стратегия на экзамене

Если в вопросе:

- zero-downtime → Deployment Slots → Standard+
- фиксированный outbound IP → NAT Gateway (не tier)
- независимое масштабирование → новый App Service Plan
- высокая безопасность/изоляция → Isolated
- production-нагрузка → не Free/Shared

---

# Главное правило

💡 App Service Plan определяет:
- масштабирование
- производительность
- доступные функции
- стоимость

Приложение наследует всё от плана.

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-app-service/3-azure-app-service-plans)
