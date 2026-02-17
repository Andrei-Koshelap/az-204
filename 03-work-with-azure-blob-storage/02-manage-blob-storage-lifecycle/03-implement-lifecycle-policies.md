# Implement Blob Storage Lifecycle Policies (Внедрение политик жизненного цикла Blob Storage)

## Implementation Methods (Способы внедрения)

Политики Lifecycle Management можно добавлять, изменять и удалять через:

| Method | Use Case | Complexity |
|--------|----------|------------|
| **Azure Portal** | Визуальная настройка, быстрый старт | Низкая |
| **Azure PowerShell** | Скрипты, автоматизация, массовые операции | Средняя |
| **Azure CLI** | Автоматизация из командной строки, CI/CD | Средняя |
| **REST APIs** | Программная интеграция, кастомные инструменты | Высокая |

> 💡 Практический комментарий:  
> Для production и CI/CD чаще выбирают **Azure CLI** или **PowerShell**, чтобы хранить policy как код (IaC-подход).  
> Portal удобен для прототипа и быстрой проверки гипотез.

---

## Azure Portal Implementation (Реализация через Azure Portal)

### Two Approaches in Portal (Два подхода в Portal)

1. **List View**: визуальный мастер (wizard), удобно для простых сценариев
2. **Code View**: прямое редактирование JSON (более гибко и мощно)

### Code View (Recommended) (Рекомендуемый вариант)

**Steps (Шаги):**

1. Перейти в нужный Storage Account в Azure Portal
2. В разделе **Data management** выбрать **Lifecycle Management**
3. Открыть вкладку **Code View**
4. Определить или отредактировать lifecycle management policy в JSON
5. Нажать **Save**

> 🎯 Экзаменационный момент AZ-204:  
> Политика lifecycle management задаётся **в виде JSON** и применяется на уровне **Storage Account**.


### Example: Move Logs to Cool Tier

**Requirement**: Move block blobs starting with `log` to Cool tier after 30 days

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "move-to-cool",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["sample-container/log"]
        }
      }
    }
  ]
}
```

### Portal Advantages (Преимущества Azure Portal)

✅ Наглядный визуальный интерфейс  
✅ Проверка синтаксиса JSON (валидация)  
✅ Удобно для настройки одной политики  
✅ Не требует написания скриптов

> 💡 Хороший вариант для обучения, тестирования и быстрой настройки.

---

### Portal Limitations (Ограничения Azure Portal)

❌ Не подходит для автоматизации  
❌ Ручная настройка при работе с несколькими Storage Account  
❌ Нет интеграции с системой контроля версий (Git)

> ⚠️ Для production-окружений предпочтительнее использовать CLI, PowerShell или IaC (ARM/Bicep/Terraform), чтобы хранить политики как код.


---

## Azure CLI Implementation

### Prerequisites

```bash
# Ensure Azure CLI is installed and logged in
az login

# Verify subscription
az account show
```

### Implementation Steps

#### 1. Create Policy JSON File

Save your policy as a JSON file (e.g., `policy.json`):

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "move-to-cool",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["sample-container/log"]
        }
      }
    }
  ]
}
```

#### 2. Create/Update Policy

```bash
# Create or update lifecycle management policy
az storage account management-policy create \
    --account-name <storage-account> \
    --policy @policy.json \
    --resource-group <resource-group>
```

⚠️ **Important**: A lifecycle management policy must be **read or written in full**. Partial updates are NOT supported.

#### 3. View Existing Policy

```bash
# Get current lifecycle policy
az storage account management-policy show \
    --account-name <storage-account> \
    --resource-group <resource-group>
```

#### 4. Delete Policy

```bash
# Delete lifecycle policy
az storage account management-policy delete \
    --account-name <storage-account> \
    --resource-group <resource-group>
```

### CLI Advantages (Преимущества Azure CLI)

✅ Поддержка скриптов и автоматизации  
✅ Лёгкая интеграция в CI/CD pipeline  
✅ Удобно хранить policy в системе контроля версий (Git)  
✅ Возможность массовых операций для нескольких Storage Account

> 💡 Практический подход:  
> Хранить JSON-политику в репозитории и применять через `az storage account management-policy create/update` в рамках deployment pipeline.  
> Это обеспечивает воспроизводимость и контроль изменений.


---

## Azure PowerShell Implementation

### Prerequisites

```powershell
# Install Azure PowerShell module (if not installed)
Install-Module -Name Az -AllowClobber -Scope CurrentUser

# Connect to Azure
Connect-AzAccount

# Set subscription context
Set-AzContext -SubscriptionId <subscription-id>
```

### Implementation Steps

#### 1. Create Policy JSON File

Save policy as `policy.json` (same format as CLI).

#### 2. Create/Update Policy

```powershell
# Set variables
$resourceGroup = "<resource-group>"
$storageAccount = "<storage-account>"
$policyFile = "policy.json"

# Read policy from file
$policy = Get-Content -Path $policyFile -Raw

# Create or update lifecycle policy
Set-AzStorageAccountManagementPolicy `
    -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount `
    -Policy $policy
```

#### 3. View Existing Policy

```powershell
# Get current lifecycle policy
Get-AzStorageAccountManagementPolicy `
    -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount
```

#### 4. Delete Policy

```powershell
# Remove lifecycle policy
Remove-AzStorageAccountManagementPolicy `
    -ResourceGroupName $resourceGroup `
    -StorageAccountName $storageAccount
```
### PowerShell Advantages (Преимущества Azure PowerShell)

✅ Интеграция с существующими PowerShell-скриптами  
✅ Удобно для автоматизации в Windows-среде  
✅ Поддержка Azure Automation Runbooks  
✅ Богатая работа с объектами (object-based модель вместо текстового CLI-вывода)

> 💡 Практический сценарий:  
> PowerShell часто используется в корпоративной среде для централизованного управления несколькими Storage Account через автоматизированные скрипты и runbooks.


---

## Complete Policy Examples (Полные примеры политик)

### Example 1: Comprehensive Log Management (Комплексное управление логами)

**Requirement (Требование):**
- Логи находятся в контейнере `logs/`
- Перевести в Cool через 30 дней
- Перевести в Archive через 90 дней
- Удалить через 2 года (730 дней)
- Удалять snapshots через 90 дней

---

### Policy (JSON)

> 💡 Примечание: ниже пример структуры policy для **одного правила**.  
> В реальном Storage Account можно хранить несколько правил в массиве `rules`.

**policy.json:**
```json
{
  "rules": [
    {
      "enabled": true,
      "name": "log-management",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            },
            "delete": {
              "daysAfterModificationGreaterThan": 730
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
          "prefixMatch": ["logs/"]
        }
      }
    }
  ]
}
```
Комментарии к примеру
    prefixMatch: ["logs/"] — таргетинг на blob, путь которых начинается с logs/
    baseBlob — дествия для текущих версий blob
    snapshot — отдельные действия для snapshot
    Переходы tier и delete завязаны на daysAfterModificationGreaterThan (последняя модификация)
    ⚠️ Важно:
    Archive требует rehydration перед чтением.
    При удалении/перемещении раньше минимальных сроков tier возможны early deletion fees.

**Deploy:**
```bash
az storage account management-policy create \
    --account-name mylogstore \
    --policy @policy.json \
    --resource-group myResourceGroup
```

### Example 2: Multi-Container Policy

**Requirement:**
- Backups: Archive after 30 days, delete after 7 years
- Temp files: Delete after 7 days
- Reports: Move to Cool after 60 days

**policy.json:**
```json
{
  "rules": [
    {
      "enabled": true,
      "name": "backup-retention",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 30
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
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
      "enabled": true,
      "name": "delete-temp-files",
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
    },
    {
      "enabled": true,
      "name": "cool-old-reports",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 60
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["reports/"]
        }
      }
    }
  ]
}
```

### Example 3: Auto-Tier to Hot from Cool

**Requirement:** Automatically move frequently accessed blobs from Cool to Hot

**policy.json:**
```json
{
  "rules": [
    {
      "enabled": true,
      "name": "auto-hot-on-access",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "enableAutoTierToHotFromCool": {
              "daysAfterLastAccessTimeGreaterThan": 0
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"]
        }
      }
    }
  ]
}
```

⚠️ **Note**: Requires **last access time tracking** to be enabled on the storage account.

---

## Workflow for Implementing Policies

### Development Workflow

```
1. Analyze data access patterns
   ↓
2. Design policy rules
   ↓
3. Create policy.json file
   ↓
4. Validate JSON syntax
   ↓
5. Test on non-production account
   ↓
6. Apply to production
   ↓
7. Monitor policy execution
   ↓
8. Adjust as needed
```
### Testing Strategy (Стратегия тестирования)

✅ **DO (Рекомендуется):**

1. Начинать с `"enabled": false` для новых правил
2. Тестировать на клоне или dev Storage Account
3. Использовать короткие значения дней для тестирования (например, 1–2 дня)
4. Мониторить метрики через Azure Monitor
5. Включать правила постепенно в production

> 💡 Практический совет:  
> При тестировании удобно создать отдельный контейнер с тестовыми blob и применить к нему ограниченное правило через `prefixMatch`.

---

❌ **DON'T (Не рекомендуется):**

- Разворачивать политику сразу в production без тестирования
- Использовать агрессивные правила удаления на первом этапе
- Игнорировать мониторинг последствий работы политики

> ⚠️ Lifecycle policy выполняется автоматически и может удалить данные без возможности восстановления (если не включены дополнительные механизмы защиты, например, soft delete).

## Monitoring and Validation

### Check Policy Execution

```bash
# Azure CLI - Check policy
az storage account management-policy show \
    --account-name <storage-account> \
    --resource-group <resource-group> \
    --output json
```

```powershell
# PowerShell - Check policy
Get-AzStorageAccountManagementPolicy `
    -ResourceGroupName <resource-group> `
    -StorageAccountName <storage-account> | ConvertTo-Json
```

### Monitor Blob Tiers

```bash
# Azure CLI - Check blob tier
az storage blob show \
    --account-name <storage-account> \
    --container-name <container> \
    --name <blob-name> \
    --query "properties.blobTier" \
    --auth-mode login
```

```powershell
# PowerShell - Check blob tier
(Get-AzStorageBlob `
    -Container <container> `
    -Blob <blob-name> `
    -Context $ctx).ICloudBlob.Properties.StandardBlobTier
```

### Azure Monitor Integration (Интеграция с Azure Monitor)

Для контроля работы Lifecycle Management рекомендуется включить **Diagnostic Settings** на уровне Storage Account.

Это позволит отслеживать:

- Операции перехода между tier (tier transition operations)
- Операции удаления (deletion operations)
- Логи выполнения политики (policy execution logs)

---

### Как включить мониторинг

1. Перейти в нужный **Storage Account**
2. Открыть раздел **Monitoring → Diagnostic settings**
3. Создать новую настройку диагностики
4. Выбрать категории логов (например, StorageRead, StorageWrite, StorageDelete)
5. Настроить отправку в:
    - Log Analytics workspace
    - Event Hub
    - Storage Account

---

### Зачем это нужно

- Проверка корректности работы lifecycle policy
- Анализ неожиданных удалений
- Аудит и соответствие требованиям комплаенса
- Оптимизация затрат

> 💡 Практический совет:  
> В production рекомендуется отправлять логи в Log Analytics и настраивать алерты на массовые удаления или частые tier transitions.


---

## Common Scenarios

### Scenario 1: Cost Optimization for Logs

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "optimize-logs",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {"daysAfterModificationGreaterThan": 14},
            "tierToArchive": {"daysAfterModificationGreaterThan": 30}
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

### Scenario 2: Compliance Retention

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "compliance-7-years",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToArchive": {"daysAfterModificationGreaterThan": 90},
            "delete": {"daysAfterModificationGreaterThan": 2555}
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["compliance/"]
        }
      }
    }
  ]
}
```

### Scenario 3: Temporary File Cleanup

```json
{
  "rules": [
    {
      "enabled": true,
      "name": "cleanup-temp",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "delete": {"daysAfterModificationGreaterThan": 7}
          }
        },
        "filters": {
          "blobTypes": ["blockBlob", "appendBlob"],
          "prefixMatch": ["temp/", "cache/"]
        }
      }
    }
  ]
}
```

---
## Best Practices (Лучшие практики)

---

### JSON Policy Management (Управление JSON-политиками)

✅ **DO (Рекомендуется):**

- Хранить файлы политик в системе контроля версий (Git)
- Использовать понятные имена файлов (`logs-policy.json`, `compliance-policy.json`)
- Добавлять пояснения в отдельной документации (README, Wiki)
- Проверять JSON-синтаксис перед деплоем
- Хранить резервные копии предыдущих версий политики

> 💡 Практический подход:  
> Рассматривать lifecycle policy как код (Policy as Code) и управлять через pull request.

❌ **DON'T (Не рекомендуется):**

- Редактировать production-политику напрямую в Portal
- Терять историю изменений
- Деплоить без предварительной валидации

---

### Deployment Strategy (Стратегия развертывания)

✅ **DO:**

- Сначала деплоить в dev/test среду
- Использовать Infrastructure as Code (Terraform, ARM, Bicep)
- Автоматизировать развертывание через CI/CD
- Документировать назначение и логику политики
- Настроить алерты на ошибки выполнения

❌ **DON'T:**

- Разворачивать вручную в нескольких Storage Account
- Пропускать этап тестирования
- Не документировать изменения

---

### Maintenance (Сопровождение)

✅ **DO:**

- Пересматривать политики ежеквартально
- Анализировать стоимость хранения и паттерны доступа
- Корректировать правила на основе фактического использования
- Включать/отключать правила при необходимости
- Делать правила простыми и понятными

❌ **DON'T:**

- Настраивать и забывать (set and forget)
- Создавать чрезмерно сложные правила
- Игнорировать логи выполнения политики

---

## Troubleshooting (Устранение неполадок)

### Common Issues (Частые проблемы)

| Issue | Cause | Solution |
|--------|--------|----------|
| Policy not applying | Правило отключено | Установить `"enabled": true` |
| Syntax error | Некорректный JSON | Проверить синтаксис |
| Blobs not transitioning | Неверный prefix | Проверить `prefixMatch` |
| Permission denied | Недостаточно RBAC прав | Назначить роль Storage Account Contributor |
| Policy conflicts | Перекрывающиеся правила | Пересмотреть и объединить правила |

---

### Validation Checklist (Чек-лист проверки)

- ✅ JSON валиден
- ✅ Имена правил уникальны
- ✅ Указан обязательный фильтр `blobTypes`
- ✅ Значения дней корректны
- ✅ Префиксы содержат имя контейнера
- ✅ Флаг `enabled` установлен правильно
- ✅ Действия соответствуют бизнес-логике
- ✅ Не более 100 правил
- ✅ Не более 10 префиксов на правило

---

## Exam Tips (Советы к экзамену AZ-204)

🎯 **Способы внедрения:** Portal (Code View), Azure CLI, PowerShell, REST APIs

🎯 **CLI команда:**  
`az storage account management-policy create --policy @policy.json`

🎯 **PowerShell cmdlet:**  
`Set-AzStorageAccountManagementPolicy -Policy $policy`

🎯 **Full replacement:**  
Политика читается и записывается **полностью** (частичное обновление не поддерживается)

🎯 **Расположение в Portal:**  
Storage Account → Data management → Lifecycle Management

🎯 **Code View vs List View:**  
Code View даёт полный контроль и прямое редактирование JSON

🎯 **Формат файла:**  
JSON-документ с массивом `rules`

🎯 **Тестирование:**  
Всегда тестировать вне production

🎯 **Мониторинг:**  
Использовать Azure Monitor для отслеживания выполнения

🎯 **Version control:**  
Хранить JSON-политики в Git для отслеживания изменений

---


## Quick Reference Commands

### Azure CLI

```bash
# Create/update policy
az storage account management-policy create \
    --account-name <account> \
    --policy @policy.json \
    --resource-group <rg>

# View policy
az storage account management-policy show \
    --account-name <account> \
    --resource-group <rg>

# Delete policy
az storage account management-policy delete \
    --account-name <account> \
    --resource-group <rg>
```

### Azure PowerShell

```powershell
# Create/update policy
$policy = Get-Content -Path policy.json -Raw
Set-AzStorageAccountManagementPolicy `
    -ResourceGroupName <rg> `
    -StorageAccountName <account> `
    -Policy $policy

# View policy
Get-AzStorageAccountManagementPolicy `
    -ResourceGroupName <rg> `
    -StorageAccountName <account>

# Delete policy
Remove-AzStorageAccountManagementPolicy `
    -ResourceGroupName <rg> `
    -StorageAccountName <account>
```

---

## Additional Resources

- [Configure lifecycle management policy](https://learn.microsoft.com/en-us/azure/storage/blobs/lifecycle-management-policy-configure)
- [Azure CLI storage account management-policy](https://learn.microsoft.com/en-us/cli/azure/storage/account/management-policy)
- [PowerShell Storage Management Policy cmdlets](https://learn.microsoft.com/en-us/powershell/module/az.storage/)

[Microsoft Learn - Implement Blob storage lifecycle policies](https://learn.microsoft.com/en-us/training/modules/manage-azure-blob-storage-lifecycle/4-add-policy-blob-storage)
