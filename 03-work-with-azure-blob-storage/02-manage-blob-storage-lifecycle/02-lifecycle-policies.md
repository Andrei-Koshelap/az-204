# Blob Storage Lifecycle Policies (Политики жизненного цикла Blob Storage)

## What is a Lifecycle Policy? (Что такое Lifecycle Policy?)

**Lifecycle management policy** — это набор **правил в формате JSON**, которые автоматически:

- Переводят blob в более «холодные» уровни доступа на основе возраста
- Удаляют blob по завершении их жизненного цикла
- Применяют действия на основе фильтров (container, prefix, blob index tags)

---

### Ключевая идея

Политика работает **на уровне Storage Account** и позволяет автоматизировать управление данными без ручного вмешательства.

Она помогает:

- Оптимизировать стоимость хранения
- Управлять ретенцией данных (retention policy)
- Соблюдать требования комплаенса
- Минимизировать человеческий фактор

---

### Как это работает

Lifecycle policy:

- Описывается в виде **JSON-документа**
- Содержит одну или несколько **rules**
- Каждое правило включает:
    - `filters` (какие blob затрагиваются)
    - `actions` (что с ними делать)

---

### Пример логики правила

- Если blob старше 30 дней → перевести в Cool
- Если старше 90 дней → перевести в Cold
- Если старше 180 дней → удалить

> 💡 На экзамене AZ-204 важно помнить:  
> Lifecycle Management — это **rule-based automation**, а не ручное управление tier.


---

## Policy Structure

### Basic Policy Format

```json
{
  "rules": [
    {
      "name": "rule1",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "filters": {...},
        "actions": {...}
      }
    },
    {
      "name": "rule2",
      "type": "Lifecycle",
      "definition": {...}
    }
  ]
}
```

### Policy Components (Компоненты политики)

| Component | Description | Required |
|-----------|------------|----------|
| **rules** | Массив объектов правил | Да (минимум 1, максимум 100) |

> 💡 Политика может содержать до 100 правил в одном JSON-документе.

---

## Rule Parameters (Параметры правила)

Каждое правило содержит следующие параметры:

| Parameter | Type | Description | Required | Details |
|------------|------|------------|----------|----------|
| **name** | String | Идентификатор правила | Да | До 256 символов, регистрозависимое, уникальное |
| **enabled** | Boolean | Включено/выключено | Нет | По умолчанию `true`, позволяет временно отключить правило |
| **type** | Enum | Тип правила | Да | В настоящее время допустимо только `"Lifecycle"` |
| **definition** | Object | Определение правила | Да | Содержит `filters` и `actions` |

---

### Rule Name Best Practices (Лучшие практики именования правил)

✅ **DO (Рекомендуется):**

- Использовать понятные имена (например, `move-logs-to-cool`)
- Следовать единому стилю (kebab-case или camelCase)
- Делать имена уникальными и отражающими смысл правила

❌ **DON'T (Не рекомендуется):**

- Использовать абстрактные имена (`rule1`, `rule2`)
- Превышать 256 символов
- Использовать повторяющиеся имена

> 💡 Хорошее имя упрощает поддержку политики в больших Storage Account с множеством правил.

---

## Rule Definition (Определение правила)

Каждое правило содержит два основных набора:

### 1️⃣ Filter Set (Набор фильтров)

Ограничивает применение правила к **конкретному подмножеству blob**.

### 2️⃣ Action Set (Набор действий)

Определяет, какие действия выполнять над отфильтрованными blob:
- Перевод в другой tier
- Удаление

---

## Filter Set (Набор фильтров)

Фильтры определяют, **к каким blob применяется правило**.

### Available Filters (Доступные фильтры)

| Filter | Type | Description | Required |
|---------|------|------------|----------|
| **blobTypes** | Array of enum | Тип blob (например, `blockBlob`, `appendBlob`) | **Да** |
| **prefixMatch** | Array of strings | Префиксы контейнера или имени blob | Нет |
| **blobIndexMatch** | Array of dictionary | Условия по blob index tags (key-value) | Нет |

---

### Filter Logic (Логика фильтрации)

💡 Если указано несколько фильтров, они объединяются через **логическое AND**.

Это означает:
- Blob должен удовлетворять **всем условиям**, чтобы правило было применено.

> 🎯 Вопрос AZ-204:  
> Если заданы `prefixMatch` и `blobIndexMatch`, применится ли правило к blob, удовлетворяющему только одному условию?  
> Ответ: Нет, требуется выполнение всех условий (AND).


```
(blobTypes = blockBlob) AND (prefix = logs/) AND (tag = status:archived)
```

### 1. blobTypes Filter

**Purpose**: Specify blob types to target.

**Valid Values:**
- `blockBlob` (most common)
- `appendBlob`

```json
"filters": {
  "blobTypes": ["blockBlob"]
}
```

⚠️ **Note:** Page blobs не поддерживаются в lifecycle management policies.

---

### 2. `prefixMatch` Filter

**Purpose (Назначение):**  
Позволяет применять правило к blob с определёнными префиксами имени.

---

### Characteristics (Характеристики)

- Представляет собой массив строк (`array of strings`)
- В одном правиле можно указать **до 10 префиксов**
- Префикс **обязательно должен начинаться с имени контейнера**

---

### Как работает `prefixMatch`

Фильтр сравнивает начало полного пути blob:
container-name/path/to/blob.txt


Пример:

```json
"prefixMatch": [
"logs/",
"archive/2024/"
]
```
⚠️ В реальной конфигурации корректнее указывать с контейнером:
```json
"prefixMatch": [
"logs-container/app-logs/",
"archive-container/2024/"
]
```
Когда использовать
- Для разделения логов по директориям
- Для обработки данных конкретного приложения
- Для применения разных retention-политик к разным папкам
- Для сегментации по годам (например, reports/2023/, reports/2024/)

💡 Экзаменационный момент AZ-204:
prefixMatch — это строковое сравнение по началу имени blob, а не полноценная поддержка «папок».
В Blob Storage нет настоящих директорий — только имя blob с разделителями /.

**Examples:**

```json
// Single container
"prefixMatch": ["logs/"]

// Multiple containers
"prefixMatch": ["logs/", "backups/"]

// Specific path within container
"prefixMatch": ["container1/folder1/subfolder/"]

// Multiple specific patterns
"prefixMatch": [
  "sample-container/blob1",
  "sample-container/blob2"
]
```

#### Prefix Matching Examples (Примеры работы prefixMatch)

| Prefix | Matches | Doesn't Match |
|----------|----------|----------------|
| `logs/` | `logs/app.log`, `logs/2024/error.log` | `oldlogs/app.log` |
| `container1/data/` | `container1/data/file.txt` | `container1/file.txt` |
| `images/photos/` | `images/photos/pic1.jpg` | `images/pic1.jpg` |

> 💡 Важно:  
> Сравнение выполняется по **началу строки имени blob**.  
> Если префикс не совпадает строго с началом пути — правило не применяется.

---

### 3. `blobIndexMatch` Filter

**Purpose (Назначение):**  
Позволяет применять правило к blob на основе **index tags (ключ-значение)**.

---

### Characteristics (Характеристики)

- Представляет собой массив словарей (key-value пары)
- В одном правиле можно указать **до 10 условий по тегам**
- Все условия объединяются через **логическое AND**

---

### Как работает `blobIndexMatch`

Blob index tags — это пользовательские метаданные, которые можно назначить blob:

```json
{
  "Project": "Finance",
  "Environment": "Production",
  "Retention": "LongTerm"
}
```
Пример фильтра:
```json
"blobIndexMatch": [
{
"name": "Project",
"op": "==",
"value": "Finance"
}
]
```

Правило применится только к blob, у которых:

Project = Finance


**Examples:**

```json
// Single tag
"blobIndexMatch": [
  {
    "name": "status",
    "op": "==",
    "value": "archived"
  }
]

// Multiple tags (AND logic)
"blobIndexMatch": [
  {
    "name": "status",
    "op": "==",
    "value": "archived"
  },
  {
    "name": "priority",
    "op": "==",
    "value": "low"
  }
]
```

---

## Action Set (Набор действий)

Actions определяют, **что происходит** с blob, прошедшими фильтрацию.

---

### Available Actions (Доступные действия)

| Action | Supported Blob Types | Current Version | Previous Versions | Snapshots |
|----------|----------------------|-----------------|-------------------|-----------|
| **tierToCool** | Block blobs | ✅ Supported | ✅ Supported | ✅ Supported |
| **tierToCold** | Block blobs | ✅ Supported | ✅ Supported | ✅ Supported |
| **enableAutoTierToHotFromCool** | Block blobs | ✅ Supported | ❌ Not supported | ❌ Not supported |
| **tierToArchive** | Block blobs | ✅ Supported | ✅ Supported | ✅ Supported |
| **delete** | Block blobs, Append blobs | ✅ Supported | ✅ Supported | ✅ Supported |

---

### Разбор действий

#### `tierToCool`
Переводит blob в Cool tier.  
Используется для данных с нечастым доступом (30+ дней).

#### `tierToCold`
Переводит blob в Cold tier.  
Подходит для редко используемых данных (90+ дней).

#### `tierToArchive`
Переводит blob в Archive tier.  
Используется для долгосрочного хранения (180+ дней).

#### `enableAutoTierToHotFromCool`
Автоматически переводит blob из Cool в Hot при обращении.  
⚠️ Работает только для **текущей версии** blob.

#### `delete`
Удаляет blob (включая текущие версии, предыдущие версии и snapshot).  
Поддерживается для Block и Append blob.

---

### Action Priority (Приоритет действий)

⚠️ **Если к одному blob применяются несколько действий**,  
Lifecycle Management выполняет **наименее затратное действие**.

Это означает:
- Система выбирает действие, которое минимизирует расходы.
- Например, если blob должен быть переведён в Cool и одновременно удалён — будет применено более экономичное действие.

> 💡 Экзаменационный момент AZ-204:  
> Lifecycle policy не выполняет все действия подряд — применяется только одно, наиболее экономически выгодное.


**Cost Order (cheapest to most expensive):**
```
delete < tierToArchive < tierToCold < tierToCool < no action
```
**Example (Пример):**  
Если blob одновременно соответствует правилам `tierToCool` и `delete`, будет применено действие **delete**.

---

## Run Conditions (Условия выполнения)

Действия запускаются на основе **условий по возрасту (age conditions)**.

---

### Condition Types (Типы условий)

| Condition | Description | Used For |
|------------|------------|----------|
| **daysAfterModificationGreaterThan** | Количество дней с момента последней модификации | Действия для базового blob |
| **daysAfterCreationGreaterThan** | Количество дней с момента создания | Действия для snapshot |
| **daysAfterLastAccessTimeGreaterThan** | Количество дней с момента последнего доступа | Текущая версия (требуется access tracking) |
| **daysAfterLastTierChangeGreaterThan** | Дней с момента последнего изменения tier | Контроль минимального срока перед Archive |

---

### 1️⃣ `daysAfterModificationGreaterThan`

**Use Case (Сценарий использования):**  
Применяется для:

- Перевода base blob в другой tier
- Удаления base blob

Условие проверяет, сколько дней прошло с момента **последнего изменения blob**.

Пример логики:

- > 30 дней после изменения → перевести в Cool
- > 90 дней → перевести в Cold
- > 180 дней → удалить

---

> 💡 Важно для AZ-204:  
> Это наиболее часто используемое условие для tier transition и delete операций над текущей версией blob.

**Based On**: Blob's **last modified time**.

```json
"actions": {
  "baseBlob": {
    "tierToCool": {
      "daysAfterModificationGreaterThan": 30
    },
    "tierToArchive": {
      "daysAfterModificationGreaterThan": 90
    },
    "delete": {
      "daysAfterModificationGreaterThan": 365
    }
  }
}
```

**Timeline:**
```
Day 0:   Blob created/modified
Day 31:  Moved to Cool tier
Day 91:  Moved to Archive tier
Day 366: Deleted
```

### 2. daysAfterCreationGreaterThan

**Use Case**: Blob snapshot management.

**Based On**: Snapshot **creation time**.

```json
"actions": {
  "snapshot": {
    "tierToCool": {
      "daysAfterCreationGreaterThan": 90
    },
    "delete": {
      "daysAfterCreationGreaterThan": 365
    }
  }
}
```

### 3. daysAfterLastAccessTimeGreaterThan

**Use Case**: Transition based on actual access patterns.

**Requirements:**
- **Access tracking** must be enabled
- Applies to **current version** only

```json
"actions": {
  "baseBlob": {
    "tierToCool": {
      "daysAfterLastAccessTimeGreaterThan": 30
    }
  }
}
```

💡 **Access Tracking (Отслеживание доступа):**  
Для использования `daysAfterLastAccessTimeGreaterThan` необходимо включить **last access time tracking** на уровне Storage Account.

⚠️ Важно:
- Эта функция не включена по умолчанию.
- Может повлечь дополнительные расходы.
- Без включённого access tracking условие работать не будет.

---

### 4️⃣ `daysAfterLastTierChangeGreaterThan`

**Use Case (Сценарий использования):**  
Предотвращает немедленный повторный перевод в Archive после rehydration.

**Applies To:**  
Только для действия `tierToArchive`.

---

### Purpose (Назначение)

Гарантирует, что blob останется в Hot / Cool / Cold tier определённое минимальное время после восстановления (rehydation) из Archive.

---

### Почему это важно

Когда blob восстанавливается из Archive:

1. Он переводится в Hot / Cool / Cold
2. Без дополнительного условия policy может снова отправить его в Archive почти сразу
3. Это создаёт:
    - лишние расходы
    - ненужные операции
    - нестабильное поведение хранения

`daysAfterLastTierChangeGreaterThan` решает эту проблему.

---

### Пример логики

- Blob восстановлен из Archive
- Политика настроена:  
  `daysAfterLastTierChangeGreaterThan = 30`
- Blob не может быть повторно переведён в Archive, пока не пройдёт 30 дней после смены tier

---

> 🎯 Экзаменационный момент AZ-204:  
> Это условие применяется **только** к `tierToArchive` и используется для контроля минимального времени пребывания blob вне Archive.


```json
"actions": {
  "baseBlob": {
    "tierToArchive": {
      "daysAfterModificationGreaterThan": 90,
      "daysAfterLastTierChangeGreaterThan": 7
    }
  }
}
```

**Scenario:**
```
1. Blob rehydrated from Archive to Hot
2. Must stay in Hot for at least 7 days
3. Then can be moved back to Archive (if >90 days since modification)
```

---

## Complete Policy Example

### Scenario: Log Management

**Requirements:**
- Logs in `sample-container`
- Blob names start with `blob1`
- Tier to Cool after 30 days
- Tier to Archive after 90 days (but wait 7 days after any tier change)
- Delete after 7 years (2,555 days)
- Delete snapshots after 90 days

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "sample-rule",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90,
              "daysAfterLastTierChangeGreaterThan": 7
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          },
          "snapshot": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["sample-container/blob1"]
        }
      }
    }
  ]
}
```

### Timeline Visualization

```
Day 0:      Blob created (Hot tier)
Day 31:     → Cool tier (after 30 days)
Day 91:     → Archive tier (after 90 days, 7+ days since last tier change)
Day 2556:   → Deleted (after 2,555 days)

Snapshots:
Day 91:     → Deleted (after 90 days from creation)
```

---

## Additional Policy Examples

### Example 1: Simple Archival Policy

Move all logs to Archive after 180 days:

```json
{
  "rules": [
    {
      "name": "archive-old-logs",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 180
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["logs/"]
        }
      }
    }
  ]
}
```

### Example 2: Multi-Container Policy

Different retention for different containers:

```json
{
  "rules": [
    {
      "name": "archive-backups",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 30
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["backups/"]
        }
      }
    },
    {
      "name": "delete-temp-files",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "delete": {
              "daysAfterModificationGreaterThan": 7
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["temp/"]
        }
      }
    }
  ]
}
```

### Example 3: Tag-Based Policy

Target blobs with specific tags:

```json
{
  "rules": [
    {
      "name": "archive-tagged-data",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "blobIndexMatch": [
            {
              "name": "status",
              "op": "==",
              "value": "completed"
            }
          ]
        }
      }
    }
  ]
}
```

---

## Best Practices (Лучшие практики)

---

### Policy Design (Проектирование политики)

✅ **DO (Рекомендуется):**

- Использовать понятные и описательные имена правил
- Начинать с небольших пилотных правил
- Использовать `prefixMatch` для точного таргетинга
- Включать правила постепенно
- Мониторить выполнение политики
- Документировать назначение каждого правила

> 💡 Практический совет:  
> Сначала протестируйте политику на отдельном контейнере, прежде чем применять ко всему Storage Account.

❌ **DON'T (Не рекомендуется):**

- Создавать чрезмерно сложные политики
- Превышать лимит в 100 правил
- Игнорировать минимальные сроки хранения tier
- Создавать перекрывающиеся правила без необходимости

---

### Filter Strategy (Стратегия фильтрации)

✅ **DO:**

- Использовать конкретные префиксы вместо общих
- Комбинировать фильтры для точности (логическое AND)
- Тестировать фильтры перед применением действий
- Использовать blob index tags для гибкой категоризации

❌ **DON'T:**

- Использовать слишком широкие фильтры
- Забывать указывать имя контейнера в `prefixMatch`
- Превышать лимит: 10 префиксов или 10 тегов на правило

---

### Action Strategy (Стратегия действий)

✅ **DO:**

- Учитывать минимальные сроки хранения tier
- Использовать `daysAfterLastTierChangeGreaterThan` для Archive
- Планировать удаление внимательно (действие необратимо)
- Понимать приоритет действий (на основе стоимости)

❌ **DON'T:**

- Архивировать данные, которым нужен частый доступ
- Удалять данные без достаточного retention-периода
- Игнорировать early deletion fees

---

## Exam Tips (Советы к экзамену AZ-204)

🎯 **Структура политики:**  
JSON-документ с массивом `rules` (максимум 100 правил)

🎯 **Компоненты правила:**  
`name`, `enabled`, `type`, `definition` (filters + actions)

🎯 **Три типа фильтров:**
- `blobTypes` (обязательный)
- `prefixMatch` (опциональный)
- `blobIndexMatch` (опциональный)

🎯 **Логика фильтрации:**  
Несколько фильтров объединяются через **логическое AND**

🎯 **Поддерживаемые типы blob:**  
Только `blockBlob` и `appendBlob` (pageBlob не поддерживаются)

🎯 **Приоритет действий (cost-based):**  
Применяется наименее затратное действие:  
`delete` > `tierToArchive` > `tierToCold` > `tierToCool`

🎯 **Четыре age-условия:**

- `daysAfterModificationGreaterThan` — для base blob
- `daysAfterCreationGreaterThan` — для snapshot
- `daysAfterLastAccessTimeGreaterThan` — требует access tracking
- `daysAfterLastTierChangeGreaterThan` — для Archive

🎯 **Лимиты:**

- До 10 префиксов на правило
- До 10 tag-условий на правило
- До 100 правил в одной политике
- Имя правила — до 256 символов, регистрозависимое, уникальное

🎯 **Особенность tierToArchive:**  
Используйте `daysAfterLastTierChangeGreaterThan`, чтобы избежать немедленного повторного архивирования после rehydration.


---

## Quick Reference

### Rule Template

```json
{
  "name": "<rule-name>",
  "enabled": true,
  "type": "Lifecycle",
  "definition": {
    "filters": {
      "blobTypes": ["blockBlob"],
      "prefixMatch": ["<container>/<prefix>"]
    },
    "actions": {
      "baseBlob": {
        "tierToCool": {
          "daysAfterModificationGreaterThan": <days>
        },
        "tierToArchive": {
          "daysAfterModificationGreaterThan": <days>
        },
        "delete": {
          "daysAfterModificationGreaterThan": <days>
        }
      }
    }
  }
}
```

### Common Day Values (Типовые значения сроков хранения)

| Retention Period | Days | Recommended Tier / Action |
|------------------|------|----------------------------|
| 1 week | 7 | Удаление временных файлов |
| 1 month | 30 | Перевод в Cool tier |
| 3 months | 90 | Cold tier или Archive |
| 6 months | 180 | Archive |
| 1 year | 365 | Удаление или Archive |
| 7 years | 2,555 | Долговременное хранение (Compliance) |

---

### Практические рекомендации

- **7 дней (временные файлы)** — подходит для temp-данных, staging-файлов и промежуточных результатов обработки.
- **30 дней** — минимальный срок для Cool tier (частый экзаменационный вопрос).
- **90 дней** — минимальный срок для Cold tier.
- **180 дней** — минимальный срок для Archive tier.
- **365 дней** — часто используется для годовой отчётности.
- **2 555 дней (~7 лет)** — типичный срок для регуляторных требований (финансы, медицина, юридические данные).

> 💡 Экзаменационный фокус AZ-204:  
> Помнить соответствие:  
> Cool = 30 дней  
> Cold = 90 дней  
> Archive = 180 дней

> ⚠️ Важно:  
> Если срок хранения меньше минимального для выбранного tier — будет начислен early deletion fee.


---

## Additional Resources

- [Lifecycle management policy schema](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-policy-configure)
- [Optimize costs with lifecycle management](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-overview)

[Microsoft Learn - Discover Blob storage lifecycle policies](https://learn.microsoft.com/en-us/training/modules/manage-azure-blob-storage-lifecycle/3-blob-storage-lifecycle-policies)
