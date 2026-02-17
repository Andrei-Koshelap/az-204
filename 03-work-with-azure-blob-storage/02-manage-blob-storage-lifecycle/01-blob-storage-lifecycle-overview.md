# Azure Blob Storage Lifecycle (Жизненный цикл Blob в Azure)

## Data Lifecycle Patterns (Паттерны жизненного цикла данных)

Наборы данных имеют **уникальные жизненные циклы**. Понимание этих паттернов критически важно для оптимизации затрат на хранение.

| Lifecycle Stage | Access Pattern | Optimal Tier | Example |
|-----------------|---------------|--------------|----------|
| **Early (Начальный)** | Частый доступ | **Hot** | Недавние загрузки пользователей, активные логи |
| **Middle (Средний)** | Нечастый доступ | **Cool / Cold** | Еженедельные отчёты, данные комплаенса |
| **Late (Поздний)** | Редкий доступ | **Archive** | Долгосрочные бэкапы, исторические данные |

### Common Data Lifecycle Scenario

Day 0-14: [Hot Tier] → Частый доступ (аналитика, отчёты)
Day 15-30: [Cool Tier] → Периодический доступ (еженедельный анализ)
Day 31-90: [Cold Tier] → Редкий доступ (ежемесячный комплаенс)
Day 91+: [Archive Tier] → Архив (долговременное хранение)

> 💡 Экзаменационный фокус AZ-204: запомнить минимальные сроки хранения и штрафы за раннее удаление.

---

## Access Tiers Overview (Обзор уровней доступа)

Azure Storage предоставляет **четыре уровня доступа**, оптимизированные под разные сценарии использования.

### Tier Comparison Table

| Tier | Type | Minimum Duration | Storage Cost | Access Cost | Latency | Use Case |
|------|------|------------------|--------------|-------------|----------|----------|
| **Hot** | Online | None | Самая высокая | Самая низкая | Миллисекунды | Часто используемые данные |
| **Cool** | Online | 30 days | Ниже | Выше | Миллисекунды | Нечастый доступ (>30 дней) |
| **Cold** | Online | 90 days | Ещё ниже | Ещё выше | Миллисекунды | Редкий доступ (>90 дней) |
| **Archive** | Offline | 180 days | Самая низкая | Самая высокая | Часы | Долговременный архив (>180 дней) |

---

## Hot Tier

**Оптимизация:** часто используемые данные

### Characteristics

- Online-tier (мгновенный доступ)
- Нет минимального срока хранения
- Самая высокая стоимость хранения
- Самая низкая стоимость операций чтения
- Задержка: миллисекунды

### Best For

- Активные данные приложения
- Часто изменяемые данные
- Данные в процессе обработки
- Real-time аналитика

### Example Scenarios

- Пользовательские изображения
- Активные логи приложения
- Данные для CDN
- Временные данные обработки

---

## Cool Tier

**Оптимизация:** нечасто используемые данные

### Characteristics

- Online-tier
- Минимальный срок хранения: **30 дней**
- Хранение дешевле, чем Hot
- Доступ дороже, чем Hot
- Миллисекундная задержка

### Best For

- Краткосрочные бэкапы
- Ежемесячные отчёты
- Завершённые проекты
- Данные для периодического аудита

⚠️ **Early Deletion Fee:** удаление или перемещение до 30 дней приводит к дополнительным расходам.

---

## Cold Tier

**Оптимизация:** очень редко используемые данные

### Characteristics

- Online-tier
- Минимальный срок хранения: **90 дней**
- Хранение дешевле, чем Cool
- Доступ дороже, чем Cool
- Миллисекундная задержка

### Best For

- Бэкапы 3+ месяцев
- Квартальные отчёты
- Исторические данные
- Legal hold данные

⚠️ **Early Deletion Fee:** удаление или перемещение до 90 дней приводит к дополнительным расходам.

---

## Archive Tier

**Оптимизация:** долговременное архивирование

### Characteristics

- Offline-tier (требуется rehydration)
- Минимальный срок хранения: **180 дней**
- Самая низкая стоимость хранения
- Самая высокая стоимость доступа
- Задержка: часы (требуется восстановление)

### Best For

- Архивы 7+ лет
- Данные регуляторного соответствия
- Disaster Recovery
- Историческое хранение

⚠️ **Rehydration Required:** перед доступом blob необходимо перевести в Hot/Cool/Cold.

⚠️ **Early Deletion Fee:** удаление до 180 дней приводит к дополнительным расходам.

> ❗ Вопрос AZ-204: можно ли читать данные напрямую из Archive?  
> Ответ: Нет, требуется rehydration.

---

## Data Storage Limits

> 💡 Лимиты хранения устанавливаются на уровне Storage Account, а не на уровне tier.

Можно:
- Использовать весь лимит в одном tier
- Распределять данные между tier
- Перемещать данные между tier при необходимости

---

## Lifecycle Management

### What is Lifecycle Management?

Azure Blob Storage поддерживает **управление жизненным циклом на основе правил (rule-based policy)**, которое позволяет:

- Автоматически переводить данные в более холодные tier
- Удалять данные по завершении жизненного цикла
- Оптимизировать затраты без ручного вмешательства

Это особенно важно для логов, бэкапов и больших объёмов исторических данных.

---

### Lifecycle Management Capabilities

| Capability | Description |
|------------|------------|
| Tier Transitions | Автоматический перевод blob по возрасту |
| Auto-Tier to Hot | Перевод Cool → Hot при обращении |
| Delete Blobs | Удаление текущих версий, предыдущих версий и snapshot |
| Scope Flexibility | Применение ко всему аккаунту или отдельным контейнерам |
| Filters | Фильтрация по prefix или blob index tags |

---

### What You Can Do with Lifecycle Policies

- Переводить Cool → Hot при доступе
- Переводить текущие версии в более холодные tier
- Переводить предыдущие версии
- Переводить snapshot
- Удалять текущие версии
- Удалять предыдущие версии
- Удалять snapshot
- Применять правила ко всему аккаунту
- Применять к конкретным контейнерам
- Применять фильтры (prefix / tags)

---

## Lifecycle Management Example Scenario

### Business Requirement

Компания хочет оптимизировать хранение application-логов.

**Access Pattern:**

- Первые 2 недели → Частый доступ → **Hot**
- 3–4 неделя → Периодический аудит → **Cool**
- После 1 месяца → Хранение для комплаенса → **Archive**

> 💡 Практический совет: в production такие сценарии реализуются через JSON Lifecycle Policy на уровне Storage Account, чтобы автоматизировать процесс и исключить ручное управление.


### Lifecycle Policy Solution

```json
{
  "rules": [
    {
      "name": "log-lifecycle-policy",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        },
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 14
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 30
            }
          }
        }
      }
    }
  ]
}
```

### Cost Optimization Result


Days 0-14:   Hot tier (frequent access, high storage cost)
Days 15-30:  Cool tier (occasional access, lower storage cost)
Days 31+:    Archive tier (rare access, lowest storage cost)

💰 Result: Significant cost savings while maintaining compliance


---

## Cost Optimization Strategy

### The Cost Tradeoff

| As Tier Gets Colder | Storage Cost | Access Cost | Latency |
|----------------------|--------------|-------------|---------|
| Hot → Cool | ↓ Decreases | ↑ Increases | Same |
| Cool → Cold | ↓ Decreases | ↑ Increases | Same |
| Cold → Archive | ↓ Decreases | ↑ Increases | ↑ Hours |

### Decision Framework

```
High Access Frequency → Hot Tier
│
├─ Moderate Access (>30 days) → Cool Tier
│  
├─ Low Access (>90 days) → Cold Tier
│
└─ Rare Access (>180 days) → Archive Tier
```

### Best Practices (Лучшие практики)

✅ **DO (Рекомендуется):**

- Использовать lifecycle policies для автоматического перевода данных между tier
- Выбирать tier на основе **реальных паттернов доступа**, а не предположений
- Регулярно мониторить access patterns и корректировать политики
- Учитывать минимальные сроки хранения перед планированием миграции
- Использовать Archive tier для долгосрочного хранения (7+ лет)
- Включать auto-tier to Hot для критичных по производительности данных

> 💡 Практический совет:  
> Перед созданием политики полезно проанализировать метрики Azure Storage (Transactions, Access Frequency, Egress), чтобы избежать неправильного выбора tier и лишних затрат.

---

❌ **DON'T (Не рекомендуется):**

- Удалять или перемещать данные до окончания минимального срока хранения (штраф за раннее удаление)
- Использовать Archive tier для данных, которым требуется мгновенный доступ
- Игнорировать реальные паттерны доступа при выборе tier
- Настраивать нереалистичные сроки перехода между tier
- Часто переключать данные между tier без экономического обоснования (может увеличить расходы)

> ⚠️ Экзаменационный момент AZ-204:  
> Если данные требуются с миллисекундной задержкой — Archive tier не подходит.


---

## Key Concepts (Ключевые концепции)

### Online vs. Offline Tiers (Онлайн и оффлайн уровни)

| Category | Tiers | Access | Latency |
|----------|-------|--------|----------|
| **Online** | Hot, Cool, Cold | Мгновенный | Миллисекунды |
| **Offline** | Archive | Требуется rehydration | Часы |

> 💡 Важно:  
> Hot, Cool и Cold — это **online tiers**, данные доступны сразу.  
> Archive — **offline tier**, перед чтением требуется восстановление (rehydration).

---

### Minimum Storage Durations (Минимальные сроки хранения)

⚠️ **Критично для оптимизации затрат:**

| Tier | Minimum Duration | Early Deletion Impact |
|------|------------------|----------------------|
| Hot | Нет | Нет штрафа |
| Cool | 30 дней | Начисляется штраф |
| Cold | 90 дней | Начисляется штраф |
| Archive | 180 дней | Начисляется штраф |

💡 **Best Practice:**  
Планируйте retention-политику так, чтобы она совпадала с минимальными сроками хранения — это позволяет избежать лишних расходов.

> 🎯 Частый вопрос AZ-204:  
> Можно ли удалить Cool blob через 10 дней без последствий?  
> Ответ: Нет, будет начислен early deletion fee.

---

### Rule-Based Policies (Политики на основе правил)

Lifecycle Management использует **правила (rules)** для автоматизации:

- Перевода в другой tier на основе **возраста (age)**
- Удаления на основе **возраста**
- Фильтрации по **container**, **prefix** или **blob index tags**

> 💡 Политики задаются в виде JSON-конфигурации на уровне Storage Account.

---

## Exam Tips (Советы к экзамену AZ-204)

🎯 **Четыре уровня доступа**: Hot, Cool, Cold, Archive — нужно знать различия

🎯 **Минимальные сроки хранения**:
- Cool = 30 дней
- Cold = 90 дней
- Archive = 180 дней

🎯 **Online vs Offline**:  
Hot / Cool / Cold — онлайн (мгновенный доступ)  
Archive — оффлайн (требуется rehydration)

🎯 **Задержка Archive**: часы, а не миллисекунды

🎯 **Lifecycle management** — это rule-based policy для автоматического


---

## Quick Reference Commands

### Set Blob Tier Manually

```bash
# Azure CLI
az storage blob set-tier \
  --account-name <account-name> \
  --container-name <container-name> \
  --name <blob-name> \
  --tier <Hot|Cool|Cold|Archive> \
  --auth-mode login

# PowerShell
Set-AzStorageBlobTier `
  -Container <container-name> `
  -Blob <blob-name> `
  -StandardBlobTier <Hot|Cool|Cold|Archive> `
  -Context $ctx
```

### Check Blob Tier

```bash
# Azure CLI
az storage blob show \
  --account-name <account-name> \
  --container-name <container-name> \
  --name <blob-name> \
  --query "properties.blobTier" \
  --auth-mode login

# PowerShell
(Get-AzStorageBlob `
  -Container <container-name> `
  -Blob <blob-name> `
  -Context $ctx).ICloudBlob.Properties.StandardBlobTier
```

---

## Tier Selection Decision Tree

```
What's the data access pattern?

├─ Accessed frequently (multiple times per day/week)
│  └─ Hot Tier ✅
│
├─ Accessed occasionally (monthly, >30 days old)
│  └─ Cool Tier ✅
│
├─ Accessed rarely (quarterly, >90 days old)
│  └─ Cold Tier ✅
│
└─ Archived (compliance, >180 days, hours to access OK)
   └─ Archive Tier ✅
```

---

## Additional Resources

- [Access tiers for blob data](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)
- [Blob storage lifecycle management](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview)

[Microsoft Learn - Explore the Azure Blob storage lifecycle](https://learn.microsoft.com/en-us/training/modules/manage-azure-blob-storage-lifecycle/2-blob-storage-lifecycle)
