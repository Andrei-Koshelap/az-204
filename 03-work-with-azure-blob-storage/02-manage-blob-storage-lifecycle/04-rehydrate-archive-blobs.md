# Rehydrate Blob Data from Archive Tier (Восстановление данных из Archive tier)

## What is Rehydration? (Что такое Rehydration?)

**Rehydration** — это процесс перевода blob из **офлайн Archive tier** в **онлайн tier** (Hot, Cool или Cold), чтобы данные можно было читать или изменять.

---

### Archive Tier Characteristics (Характеристики Archive tier)

| Characteristic | Detail |
|----------------|--------|
| **Status** | Offline (нельзя читать или изменять) |
| **Access** | Требуется предварительное восстановление |
| **Storage Cost** | Самая низкая |
| **Access Cost** | Самая высокая |
| **Rehydration Time** | Несколько часов |

⚠️ **Критично:**  
Blob в Archive tier являются **офлайн** и должны быть восстановлены (rehydrated) перед доступом.

---

## Rehydration Methods (Способы восстановления)

Существует **два варианта** восстановления архивных blob.

### Method Comparison (Сравнение методов)

| Method | Operation | Source Blob | Recommended For |
|----------|------------|--------------|------------------|
| **Copy to Online Tier** | Copy Blob / Copy Blob From URL | Остаётся в Archive | Рекомендуется в большинстве случаев |
| **Change Blob Tier** | Set Blob Tier | Перемещается в онлайн tier | Простая смена tier |

---

## Method 1: Copy Archived Blob to Online Tier (Recommended)

### Overview (Обзор)

**Microsoft рекомендует этот метод** ✅

### Process (Процесс)

1. Скопировать архивный blob в **новый blob** в Hot, Cool или Cold tier
2. Исходный blob **остаётся в Archive tier**
3. Новый blob становится доступным после завершения rehydration

---

### Key Rules (Ключевые правила)

⚠️ **Требования к именованию:**

- Копирование должно выполняться в **другое имя blob** ИЛИ в **другой контейнер**
- Нельзя копировать blob поверх самого себя (то же имя в том же контейнере)

---

### Service Version Support (Поддержка версий сервиса)

| Service Version | Scope | Support |
|-----------------|--------|---------|
| **< 2021-02-12** | Только внутри одного storage account | Восстановление в пределах аккаунта |
| **≥ 2021-02-12** | Между storage account (в одном регионе) | Восстановление между аккаунтами |

💡 Начиная с версии сервиса **2021-02-12+**:  
Можно выполнить rehydration, копируя blob в **другой Storage Account**, если оба аккаунта находятся в **одном регионе**.

---

### Почему этот метод предпочтителен?

- Оригинальные архивные данные сохраняются
- Можно протестировать восстановленные данные отдельно
- Нет риска потерять исходный blob
- Подходит для сценариев аудита и комплаенса

> 🎯 Экзаменационный момент AZ-204:  
> Archive tier — офлайн. Для доступа требуется rehydration, а рекомендуемый способ — **Copy to Online Tier**.

### REST API: Copy Blob

```http
PUT https://<dest-account>.blob.core.windows.net/<dest-container>/<dest-blob>
x-ms-copy-source: https://<source-account>.blob.core.windows.net/<source-container>/<source-blob>
x-ms-access-tier: Hot
x-ms-rehydrate-priority: High
Authorization: <auth-header>
```

### Azure CLI Example

```bash
# Copy archived blob to Hot tier in same account
az storage blob copy start \
    --source-account-name <source-account> \
    --source-container <source-container> \
    --source-blob <source-blob> \
    --account-name <dest-account> \
    --destination-container <dest-container> \
    --destination-blob <dest-blob> \
    --tier Hot \
    --rehydrate-priority High \
    --auth-mode login

# Copy archived blob to different account (same region, v2021-02-12+)
az storage blob copy start \
    --source-account-name sourceaccount \
    --source-container archive \
    --source-blob data.bin \
    --account-name destaccount \
    --destination-container hot \
    --destination-blob data-restored.bin \
    --tier Hot \
    --rehydrate-priority Standard \
    --auth-mode login
```

### PowerShell Example

```powershell
# Get source blob URI
$sourceContext = New-AzStorageContext -StorageAccountName <source-account>
$sourceBlob = Get-AzStorageBlob `
    -Container <source-container> `
    -Blob <source-blob> `
    -Context $sourceContext

# Copy to Hot tier
$destContext = New-AzStorageContext -StorageAccountName <dest-account>
Start-AzStorageBlobCopy `
    -SrcBlob $sourceBlob.Name `
    -SrcContainer <source-container> `
    -Context $sourceContext `
    -DestContainer <dest-container> `
    -DestBlob <dest-blob> `
    -DestContext $destContext `
    -StandardBlobTier Hot `
    -RehydratePriority High
```

### .NET SDK Example

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

// Source archived blob
var sourceClient = new BlobClient(connectionString, "archive-container", "archived-blob");

// Destination blob in Hot tier
var destClient = new BlobClient(connectionString, "hot-container", "restored-blob");

// Copy operation with rehydration
var options = new BlobCopyFromUriOptions
{
    AccessTier = AccessTier.Hot,
    RehydratePriority = RehydratePriority.High
};

await destClient.StartCopyFromUriAsync(sourceClient.Uri, options);
```

### Advantages of Copy Method (Преимущества метода копирования)

✅ **Сохранение источника**: Оригинальный blob остаётся в Archive  
✅ **Безопасность**: Нет риска потери исходных данных  
✅ **Изоляция**: Можно проверить восстановленные данные отдельно  
✅ **Межаккаунтное копирование**: Поддерживается (v2021-02-12+) в пределах одного региона  
✅ **Рекомендуемый метод**: Подход, рекомендованный Microsoft

> 💡 Хороший выбор для сценариев аудита, восстановления после инцидентов и комплаенса.

---

### Disadvantages (Недостатки)

❌ **Дублирование хранения**: Временно существуют две копии (повышенные расходы)  
❌ **Ручная очистка**: Нужно удалить исходный blob после проверки

---

## Method 2: Change Blob's Access Tier (Изменение Access Tier)

### Overview (Обзор)

### Process (Процесс)

1. Использовать операцию **Set Blob Tier**
2. Изменить tier с Archive на Hot / Cool / Cold
3. Blob изменяет tier **в том же месте (in place)**
4. После запуска процесс **нельзя отменить**
5. Во время rehydration blob продолжает отображаться как `"archived"`

---

### Особенности метода

- Нет создания новой копии
- Нет временного удвоения объёма хранения
- Процесс восстановления занимает несколько часов
- После запуска отменить rehydration нельзя

> ⚠️ Важно:  
> Хотя tier изменяется in place, доступ к данным появится только после завершения rehydration.


### REST API: Set Blob Tier

```http
PUT https://<account>.blob.core.windows.net/<container>/<blob>?comp=tier
x-ms-access-tier: Hot
x-ms-rehydrate-priority: High
Authorization: <auth-header>
```

### Azure CLI Example

```bash
# Change blob tier from Archive to Hot
az storage blob set-tier \
    --account-name <account-name> \
    --container-name <container-name> \
    --name <blob-name> \
    --tier Hot \
    --rehydrate-priority High \
    --auth-mode login

# Change to Cool tier
az storage blob set-tier \
    --account-name <account-name> \
    --container-name <container-name> \
    --name <blob-name> \
    --tier Cool \
    --rehydrate-priority Standard \
    --auth-mode login
```

### PowerShell Example

```powershell
# Get blob
$blob = Get-AzStorageBlob `
    -Container <container-name> `
    -Blob <blob-name> `
    -Context $ctx

# Change tier to Hot with high priority
$blob.ICloudBlob.SetStandardBlobTier(
    [Microsoft.Azure.Storage.Blob.StandardBlobTier]::Hot,
    [Microsoft.Azure.Storage.Blob.RehydratePriority]::High
)
```

### .NET SDK Example

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

var blobClient = new BlobClient(connectionString, containerName, blobName);

// Change tier to Hot with high priority
await blobClient.SetAccessTierAsync(
    AccessTier.Hot,
    rehydratePriority: RehydratePriority.High
);

// Check tier status
var properties = await blobClient.GetPropertiesAsync();
Console.WriteLine($"Current Tier: {properties.Value.AccessTier}");
Console.WriteLine($"Archive Status: {properties.Value.ArchiveStatus}");
Console.WriteLine($"Rehydrate Priority: {properties.Value.RehydratePriority}");
```

### Important Considerations (Важные моменты)

⚠️ **Нельзя отменить:**  
После запуска операции **Set Blob Tier** её **нельзя отменить**.

⚠️ **Last Modified Time:**  
Изменение tier **не обновляет** поле *Last Modified*.

⚠️ **Риск с Lifecycle Policy:**  
Если настроена lifecycle policy, blob может быть автоматически отправлен обратно в Archive после восстановления.

---

## Lifecycle Policy Scenario (Типичный сценарий проблемы)

### Problem (Проблема)

1. Blob был изменён 120 дней назад
2. Lifecycle policy: отправлять в Archive после 90 дней
3. Выполняется rehydration в Hot tier
4. Last modified по-прежнему = 120 дней
5. Lifecycle policy снова срабатывает → blob возвращается в Archive

---

### Solution (Решение)

- Обновить blob (тем самым изменить last modified) после rehydration
- Использовать `daysAfterLastTierChangeGreaterThan` в lifecycle policy
- Исключить конкретные blob из действия политики (через prefix или tags)

> 💡 Лучший практический вариант — использовать `daysAfterLastTierChangeGreaterThan`, чтобы избежать повторной архивации.

---

### Advantages of Set Tier Method (Преимущества метода Set Tier)

✅ Нет дублирования хранения  
✅ Проще — одна операция  
✅ Blob остаётся на том же месте

---

### Disadvantages (Недостатки)

❌ Нельзя отменить операцию  
❌ Риск повторной архивации  
❌ Last modified не изменяется  
❌ Нет резервной копии исходного архивного состояния

> ⚠️ Если данные критичны — безопаснее использовать метод копирования.

---

## Rehydration Priority (Приоритет восстановления)

При восстановлении можно задать приоритет через заголовок:
        x-ms-rehydrate-priority


---

### Priority Options (Варианты приоритета)

| Priority | Processing | Completion Time | Use Case |
|------------|-------------|------------------|------------|
| **Standard** | В порядке очереди | До **15 часов** | Не срочно, экономия средств |
| **High** | Приоритетная обработка | Менее **1 часа** (для объектов < 10 GB) | Срочный доступ |

---

### Дополнительные замечания

- High priority стоит дороже
- Время зависит от размера blob
- Приоритет можно задать только при начале rehydration

> 🎯 Экзаменационный момент AZ-204:  
> Standard — до 15 часов,  
> High — менее 1 часа (для небольших объектов).

### How to Set Priority

```bash
# Azure CLI - High priority
az storage blob copy start \
    --source-blob <source> \
    --destination-blob <dest> \
    --tier Hot \
    --rehydrate-priority High

# Azure CLI - Standard priority (default)
az storage blob set-tier \
    --name <blob-name> \
    --tier Cool \
    --rehydrate-priority Standard
```

### Check Rehydration Status

#### Azure CLI

```bash
# Check rehydration priority
az storage blob show \
    --account-name <account-name> \
    --container-name <container-name> \
    --name <blob-name> \
    --query "properties.rehydrationStatus" \
    --auth-mode login
```

#### PowerShell

```powershell
# Get blob properties
$blob = Get-AzStorageBlob `
    -Container <container> `
    -Blob <blob-name> `
    -Context $ctx

# Check archive status and rehydrate priority
$blob.ICloudBlob.Properties.RehydrationStatus
$blob.ICloudBlob.Properties.StandardBlobTier
```

#### .NET SDK

```csharp
// Get blob properties
var properties = await blobClient.GetPropertiesAsync();

// Check rehydration status
Console.WriteLine($"Archive Status: {properties.Value.ArchiveStatus}");
Console.WriteLine($"Rehydrate Priority: {properties.Value.RehydratePriority}");
Console.WriteLine($"Access Tier: {properties.Value.AccessTier}");

// Archive status values:
// - rehydrate-pending-to-hot
// - rehydrate-pending-to-cool
// - rehydrate-pending-to-cold
```

### Rehydration Status Values (Статусы восстановления)

| Status | Meaning |
|--------|----------|
| `rehydrate-pending-to-hot` | Идёт восстановление в Hot tier |
| `rehydrate-pending-to-cool` | Идёт восстановление в Cool tier |
| `rehydrate-pending-to-cold` | Идёт восстановление в Cold tier |
| `null` | Blob не в Archive или восстановление завершено |

> 💡 Когда rehydration завершён, статус становится `null`, и blob полностью доступен в выбранном online tier.

---

## Rehydration Performance (Производительность восстановления)

### Timeline Comparison (Сравнение по времени)

| Priority | Size | Expected Time |
|------------|--------|----------------|
| **High** | < 10 GB | < 1 часа |
| **High** | > 10 GB | Зависит от размера, приоритетная обработка |
| **Standard** | Любой размер | До 15 часов |

---

### Практическая рекомендация

💡 Для оптимальной производительности:

- Лучше восстанавливать **крупные blob**, чем большое количество мелких
- Массовое восстановление множества маленьких объектов может занять больше времени
- Планируйте rehydration заранее для критичных данных

---

## Cost Considerations (Финансовые аспекты)

| Aspect | Standard Priority | High Priority |
|---------|-------------------|---------------|
| **Rehydration Cost** | Ниже | Выше |
| **Processing Time** | До 15 часов | < 1 часа (< 10 GB) |
| **Best For** | Несрочные задачи, оптимизация затрат | Срочные сценарии |

---

> 🎯 Экзаменационный момент AZ-204:
> - Standard — дешевле, но до 15 часов
> - High — быстрее (< 1 часа для < 10 GB), но дороже
> - Blob остаётся недоступным до завершения rehydration

---

## Complete Rehydration Workflow

### Workflow 1: Copy Method (Recommended)

```
1. Identify archived blob
   ↓
2. Start copy operation to online tier (Hot/Cool/Cold)
   - Set rehydration priority (Standard/High)
   ↓
3. Monitor copy status
   ↓
4. New blob available in online tier (source remains in Archive)
   ↓
5. Verify data integrity
   ↓
6. (Optional) Delete archived source blob
```

### Workflow 2: Set Tier Method

```
1. Identify archived blob
   ↓
2. Execute Set Blob Tier operation to online tier
   - Set rehydration priority (Standard/High)
   ↓
3. Monitor rehydration status (cannot cancel)
   ↓
4. Blob becomes available in online tier (in place)
   ↓
5. (Optional) Update last modified time to prevent lifecycle re-archival
```

---

## Best Practices (Лучшие практики)

---

### Rehydration Strategy (Стратегия восстановления)

✅ **DO (Рекомендуется):**

- Использовать **Copy method** в большинстве сценариев (рекомендованный Microsoft подход)
- Выбирать **High priority** для срочного восстановления (< 10 GB)
- Использовать **Standard priority** для оптимизации затрат
- Отслеживать статус rehydration
- Проверять данные после завершения восстановления
- Планировать восстановление заранее (особенно при Standard priority)

> 💡 Практический совет:  
> Если восстановление связано с инцидентом или аудитом — безопаснее использовать Copy method, чтобы сохранить исходный архивный blob.

---

❌ **DON'T (Не рекомендуется):**

- Использовать Set Tier, если lifecycle policy может повторно архивировать blob
- Ожидать мгновенного доступа к данным из Archive
- Восстанавливать большое количество маленьких blob одновременно
- Игнорировать стоимость rehydration

> ⚠️ Помните:  
> Archive — это офлайн tier. Данные остаются недоступными до завершения процесса восстановления.
### Lifecycle Policy Protection

When using **Set Tier method**, protect against re-archival:

```json
{
  "rules": [
    {
      "name": "archive-with-cooldown",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90,
              "daysAfterLastTierChangeGreaterThan": 7
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

**Key**: `daysAfterLastTierChangeGreaterThan: 7` ensures blob stays in online tier for at least 7 days after rehydration.

---
## Cost Optimization (Оптимизация затрат)

### Rehydration Costs (Затраты на восстановление)

| Component | Cost Factor |
|------------|-------------|
| **Rehydration operation** | Оплата за GB (выше при High priority) |
| **Data retrieval** | Оплата за извлечение данных (per-GB) |
| **Storage during rehydration** | Хранение в Archive продолжается |
| **Destination storage** | Хранение в online tier (при копировании) |

---

### Cost Optimization Tips (Советы по оптимизации)

💰 **Как снизить расходы:**

1. Использовать **Standard priority**, если нет срочности
2. Восстанавливать только необходимые blob
3. Группировать (batch) запросы на восстановление
4. Использовать **Copy method** для проверки перед удалением источника
5. Удалять архивный blob после проверки данных
6. Планировать восстановление заранее, чтобы избежать High priority

> 💡 Важно:  
> Во время rehydration blob продолжает тарифицироваться как Archive.  
> При Copy method дополнительно оплачивается хранение новой online-копии.

---

## Exam Tips (Советы к экзамену AZ-204)

🎯 **Два метода восстановления:**  
Copy Blob (рекомендуется) и Set Blob Tier

🎯 **Copy method — рекомендованный подход**

🎯 **Правило именования при Copy:**  
Нужно копировать в другое имя или контейнер (перезапись запрещена)

🎯 **Set Tier особенности:**  
Нельзя отменить, не обновляет last modified

🎯 **Риск с lifecycle policy:**  
Set Tier может привести к повторной архивации

🎯 **Два приоритета:**  
Standard (до 15 часов), High (< 1 часа для < 10 GB)

🎯 **Rehydration не мгновенный:**  
Занимает часы

🎯 **Статусы восстановления:**  
`rehydrate-pending-to-hot` / `cool` / `cold`

🎯 **Cross-account поддержка:**  
Версия сервиса 2021-02-12+ позволяет копирование между аккаунтами (в одном регионе)

🎯 **Проверка статуса:**  
Использовать Get Blob Properties и проверять заголовки, включая `x-ms-rehydrate-priority`

🎯 **Совет по производительности:**  
Лучше восстанавливать меньшее количество крупных blob

🎯 **REST API операции:**  
Copy Blob, Copy Blob From URL, Set Blob Tier

---


## Quick Reference Commands

### Check if Blob is Archived

```bash
# Azure CLI
az storage blob show \
    --account-name <account> \
    --container-name <container> \
    --name <blob> \
    --query "properties.blobTier" \
    --auth-mode login
```

### Rehydrate via Copy (Recommended)

```bash
# Azure CLI
az storage blob copy start \
    --source-container <source-container> \
    --source-blob <source-blob> \
    --destination-container <dest-container> \
    --destination-blob <dest-blob> \
    --account-name <account> \
    --tier Hot \
    --rehydrate-priority High \
    --auth-mode login
```

### Rehydrate via Set Tier

```bash
# Azure CLI
az storage blob set-tier \
    --account-name <account> \
    --container-name <container> \
    --name <blob> \
    --tier Hot \
    --rehydrate-priority High \
    --auth-mode login
```

### Monitor Rehydration Status

```bash
# Azure CLI
az storage blob show \
    --account-name <account> \
    --container-name <container> \
    --name <blob> \
    --query "properties.[blobTier,rehydrationStatus]" \
    --auth-mode login
```

---

## Decision Tree

```
Need to access archived blob?
│
├─ Need immediate preservation of source?
│  └─ YES → Use Copy Blob method ✅
│             - Source preserved in Archive
│             - New blob in online tier
│             - Verify before deleting source
│
└─ Simple in-place tier change acceptable?
   └─ YES → Check: Do you have lifecycle policies?
              │
              ├─ NO → Use Set Blob Tier ✅
              │        - Simpler, no duplication
              │
              └─ YES → Risk of re-archival!
                       → Add daysAfterLastTierChangeGreaterThan
                       → OR use Copy Blob method instead
```

---

## Additional Resources

- [Rehydrate an archived blob to an online tier](https://learn.microsoft.com/en-us/azure/storage/blobs/archive-rehydrate-to-online-tier)
- [Archive tier best practices](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-best-practices)

[Microsoft Learn - Rehydrate blob data from the archive tier](https://learn.microsoft.com/en-us/training/modules/manage-azure-blob-storage-lifecycle/5-rehydrate-blob-data)
