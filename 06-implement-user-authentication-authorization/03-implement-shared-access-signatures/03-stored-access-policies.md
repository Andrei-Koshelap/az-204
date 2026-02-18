# Stored Access Policies

## Ключевые понятия

- **Stored access policy** — серверная политика для Service SAS
- **Отзыв доступа** — можно изменить или отменить SAS без регенерации ключей
- **Централизованное управление** — управление сроками и правами из одного места
- **Максимум 5 политик** — на контейнер, очередь, таблицу или file share

---

# Что такое Stored Access Policy?

**Stored access policy** — это дополнительный уровень контроля для **service-level SAS**, реализованный на стороне сервера.

> 💡 Работает только с Service SAS (не с Account SAS).

---

## Назначение

Позволяет **группировать и управлять SAS-токенами**:

- Централизованно управлять сроком действия
- Изменять разрешения для всех связанных SAS
- Отзывать доступ без регенерации account key
- Контролировать время начала действия

---

# Поддерживаемые ресурсы

| Тип ресурса | Поддержка Stored Policy |
|--------------|--------------------------|
| **Blob containers** | ✅ Да |
| **File shares** | ✅ Да |
| **Queues** | ✅ Да |
| **Tables** | ✅ Да |
| **Отдельные blobs/files** | ❌ Нет (используется политика контейнера/share) |
| **Account SAS** | ❌ Нет (только для Service SAS) |

---

# Параметры SAS: Policy vs Token

## Способы задания параметров

Параметры SAS можно определить **тремя способами**:

| Вариант | Start Time | Expiry Time | Permissions | Когда использовать |
|----------|------------|-------------|-------------|--------------------|
| **Все в SAS токене** | В токене | В токене | В токене | Разовый доступ |
| **Все в Stored Policy** | В политике | В политике | В политике | Управляемый и отзывной доступ |
| **Смешанный вариант** | В токене | В политике | В политике | Запланированный старт с централизованным окончанием |

---

⚠️ **Правило:**  
Нельзя указывать один и тот же параметр одновременно в SAS-токене и в Stored Access Policy.

---

## Почему это важно

- Позволяет централизованно менять срок действия.
- Позволяет отзывать доступ без смены ключей.
- Упрощает управление большим количеством SAS.

---

## Важно для AZ-204

- Работает только для Service SAS.
- Максимум 5 политик на контейнер или share.
- Позволяет изменять права для уже выданных SAS.
- Нельзя комбинировать одинаковые параметры в токене и политике.

> 🎯 Частый экзаменационный вопрос:  
Как отозвать доступ для уже выданных SAS без смены ключа?  
Ответ — использовать Stored Access Policy.


### Example: All Parameters in Policy

```csharp
// Create stored access policy
BlobSignedIdentifier identifier = new BlobSignedIdentifier
{
    Id = "read-policy",
    AccessPolicy = new BlobAccessPolicy
    {
        StartsOn = DateTimeOffset.UtcNow,
        ExpiresOn = DateTimeOffset.UtcNow.AddDays(7),
        Permissions = "r"  // Read-only
    }
};

await blobContainer.SetAccessPolicyAsync(permissions: new[] { identifier });

// Generate SAS referencing policy (no expiry/permissions in token)
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "documents",
    BlobName = "report.pdf",
    Resource = "b",
    Identifier = "read-policy"  // Reference policy by ID
};

BlobClient blobClient = containerClient.GetBlobClient("report.pdf");
Uri sasUri = blobClient.GenerateSasUri(sasBuilder);
```

**SAS token** will look like:

```
https://storageaccount.blob.core.windows.net/documents/report.pdf?si=read-policy&sig=...
```

Notice: No `sp`, `st`, `se` parameters (they're in the policy).

### Example: Mixed Parameters

```csharp
// Policy has expiry and permissions
BlobSignedIdentifier identifier = new BlobSignedIdentifier
{
    Id = "standard-access",
    AccessPolicy = new BlobAccessPolicy
    {
        ExpiresOn = DateTimeOffset.UtcNow.AddDays(30),
        Permissions = "rl"  // Read + List
    }
};

await blobContainer.SetAccessPolicyAsync(permissions: new[] { identifier });

// SAS token specifies start time (not in policy)
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "documents",
    BlobName = "report.pdf",
    Resource = "b",
    Identifier = "standard-access",
    StartsOn = DateTimeOffset.UtcNow.AddHours(1)  // Delayed start
};

Uri sasUri = blobClient.GenerateSasUri(sasBuilder);
```

## Creating Stored Access Policies

### Using C# (.NET)

**Blob container policy**:

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

// Get container client
BlobContainerClient containerClient = new BlobContainerClient(
    connectionString,
    "my-container"
);

// Create policy
BlobSignedIdentifier identifier = new BlobSignedIdentifier
{
    Id = "my-policy-id",  // Unique identifier (up to 64 chars)
    AccessPolicy = new BlobAccessPolicy
    {
        StartsOn = DateTimeOffset.UtcNow,
        ExpiresOn = DateTimeOffset.UtcNow.AddHours(1),
        Permissions = "rw"  // Read + Write
    }
};

// Set access policy on container
await containerClient.SetAccessPolicyAsync(
    permissions: new BlobSignedIdentifier[] { identifier }
);

Console.WriteLine("Stored access policy created");
```

**Queue policy**:

```csharp
using Azure.Storage.Queues;
using Azure.Storage.Queues.Models;

QueueClient queueClient = new QueueClient(connectionString, "my-queue");

QueueSignedIdentifier identifier = new QueueSignedIdentifier
{
    Id = "queue-policy",
    AccessPolicy = new QueueAccessPolicy
    {
        StartsOn = DateTimeOffset.UtcNow,
        ExpiresOn = DateTimeOffset.UtcNow.AddDays(1),
        Permissions = "raup"  // Read, Add, Update, Process
    }
};

await queueClient.SetAccessPolicyAsync(new[] { identifier });
```

**Table policy**:

```csharp
using Azure.Data.Tables;
using Azure.Data.Tables.Models;

TableClient tableClient = new TableClient(connectionString, "MyTable");

TableSignedIdentifier identifier = new TableSignedIdentifier("table-policy")
{
    AccessPolicy = new TableAccessPolicy
    {
        StartsOn = DateTimeOffset.UtcNow,
        ExpiresOn = DateTimeOffset.UtcNow.AddDays(7),
        Permissions = "raud"  // Read, Add, Update, Delete
    }
};

await tableClient.SetAccessPolicyAsync(new[] { identifier });
```

**File share policy**:

```csharp
using Azure.Storage.Files.Shares;
using Azure.Storage.Files.Shares.Models;

ShareClient shareClient = new ShareClient(connectionString, "my-share");

ShareSignedIdentifier identifier = new ShareSignedIdentifier
{
    Id = "share-policy",
    AccessPolicy = new ShareAccessPolicy
    {
        StartsOn = DateTimeOffset.UtcNow,
        ExpiresOn = DateTimeOffset.UtcNow.AddMonths(1),
        Permissions = "rcwdl"  // Read, Create, Write, Delete, List
    }
};

await shareClient.SetAccessPolicyAsync(new[] { identifier });
```

### Using Azure CLI

**Create blob container policy**:

```bash
az storage container policy create \
    --name my-policy-id \
    --container-name my-container \
    --account-name mystorageaccount \
    --account-key $ACCOUNT_KEY \
    --start 2024-01-01T00:00:00Z \
    --expiry 2024-12-31T23:59:59Z \
    --permissions rw
```

**Create queue policy**:

```bash
az storage queue policy create \
    --name queue-policy \
    --queue-name my-queue \
    --account-name mystorageaccount \
    --account-key $ACCOUNT_KEY \
    --start 2024-01-01T00:00:00Z \
    --expiry 2024-12-31T23:59:59Z \
    --permissions raup
```

**Create table policy**:

```bash
az storage table policy create \
    --name table-policy \
    --table-name MyTable \
    --account-name mystorageaccount \
    --account-key $ACCOUNT_KEY \
    --start 2024-01-01T00:00:00Z \
    --expiry 2024-12-31T23:59:59Z \
    --permissions raud
```

**Create file share policy**:

```bash
az storage share policy create \
    --name share-policy \
    --share-name my-share \
    --account-name mystorageaccount \
    --account-key $ACCOUNT_KEY \
    --start 2024-01-01T00:00:00Z \
    --expiry 2024-12-31T23:59:59Z \
    --permissions rcwdl
```

### Using PowerShell

```powershell
# Get storage context
$context = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey $accountKey

# Create policy
$policy = New-AzStorageContainerStoredAccessPolicy `
    -Context $context `
    -Container "my-container" `
    -Policy "my-policy" `
    -Permission rw `
    -StartTime (Get-Date) `
    -ExpiryTime (Get-Date).AddDays(7)

Write-Host "Policy created: $($policy.Policy)"
```
## Создание Stored Access Policy через Azure Portal

### Пошаговая инструкция

1. Перейдите в нужный **Storage Account**
2. Выберите раздел **Containers**  
   (или **Queues**, **Tables**, **File shares**)
3. Откройте нужный контейнер
4. Нажмите **Access policy**  
   (или **Stored access policies**)
5. Нажмите **+ Add policy**
6. Заполните параметры:
   - **Identifier** — имя политики (например, `read-policy`)
   - **Permissions** — выберите разрешения (Read, Write, Delete, List и др.)
   - **Start time** — время начала действия (необязательно)
   - **Expiry time** — время окончания действия
7. Нажмите **OK**
8. Нажмите **Save**

---

## Важно помнить

- Максимум 5 политик на контейнер/очередь/table/share.
- Политика применяется только к Service SAS.
- После изменения политики все связанные SAS автоматически обновляют своё поведение.
- Удаление политики немедленно аннулирует связанные SAS.

---

## Best Practices

- Использовать понятные имена для Identifier.
- Устанавливать минимальные разрешения.
- Ограничивать срок действия.
- Использовать HTTPS для всех SAS.

---

## Важно для AZ-204

- Stored Access Policy создаётся на уровне контейнера или share.
- Позволяет централизованно управлять SAS.
- Используется для отзыва доступа без регенерации ключей.
- Работает только с Service SAS.

> 🎯 Частый экзаменационный вопрос:  
Где создаётся Stored Access Policy?  
Ответ — на уровне контейнера, очереди, таблицы или file share.


## Using Stored Access Policies with SAS

### Generate SAS with Policy Reference

```csharp
// After creating stored access policy, generate SAS
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "my-container",
    BlobName = "document.pdf",
    Resource = "b",
    Identifier = "my-policy-id"  // Reference stored policy
};

// No need to specify permissions, start time, or expiry
// They come from the stored access policy

BlobClient blobClient = containerClient.GetBlobClient("document.pdf");
Uri sasUri = blobClient.GenerateSasUri(sasBuilder);

Console.WriteLine($"SAS URI: {sasUri}");
```

### Azure CLI

```bash
# Generate SAS using stored policy
az storage blob generate-sas \
    --account-name mystorageaccount \
    --container-name my-container \
    --name document.pdf \
    --policy-name my-policy-id \
    --output tsv
```

## Modifying Stored Access Policies

### Update Policy Parameters

```csharp
// Get existing policies
var existingPolicies = await containerClient.GetAccessPolicyAsync();
var policies = existingPolicies.Value.SignedIdentifiers.ToList();

// Find and modify policy
var policyToUpdate = policies.FirstOrDefault(p => p.Id == "my-policy-id");
if (policyToUpdate != null)
{
    // Change permissions from read-write to read-only
    policyToUpdate.AccessPolicy.Permissions = "r";
    
    // Extend expiry by 7 days
    policyToUpdate.AccessPolicy.ExpiresOn = DateTimeOffset.UtcNow.AddDays(7);
}

// Save updated policies
await containerClient.SetAccessPolicyAsync(permissions: policies);

Console.WriteLine("Policy updated - all associated SAS now affected");
```

### Azure CLI

```bash
# Update existing policy
az storage container policy update \
    --name my-policy-id \
    --container-name my-container \
    --account-name mystorageaccount \
    --account-key $ACCOUNT_KEY \
    --expiry 2025-12-31T23:59:59Z \
    --permissions r
```

**Effect**: All SAS tokens associated with this policy are immediately affected.

## Revoking Access

### Method 1: Delete Policy

```csharp
// Get existing policies
var existingPolicies = await containerClient.GetAccessPolicyAsync();
var policies = existingPolicies.Value.SignedIdentifiers.ToList();

// Remove policy
policies.RemoveAll(p => p.Id == "my-policy-id");

// Save updated list (without deleted policy)
await containerClient.SetAccessPolicyAsync(permissions: policies);

Console.WriteLine("Policy deleted - all associated SAS immediately revoked");
```

**Azure CLI**:

```bash
az storage container policy delete \
    --name my-policy-id \
    --container-name my-container \
    --account-name mystorageaccount \
    --account-key $ACCOUNT_KEY
```

### Method 2: Change Policy Identifier

```csharp
// Rename policy (breaks association with existing SAS)
var existingPolicies = await containerClient.GetAccessPolicyAsync();
var policies = existingPolicies.Value.SignedIdentifiers.ToList();

var policy = policies.FirstOrDefault(p => p.Id == "my-policy-id");
if (policy != null)
{
    // Remove old policy
    policies.Remove(policy);
    
    // Add with new identifier
    policy.Id = "new-policy-id";
    policies.Add(policy);
}

await containerClient.SetAccessPolicyAsync(permissions: policies);

Console.WriteLine("Policy ID changed - existing SAS tokens no longer work");
```

### Method 3: Change Expiry to Past

```csharp
// Set expiry time in the past
var existingPolicies = await containerClient.GetAccessPolicyAsync();
var policies = existingPolicies.Value.SignedIdentifiers.ToList();

var policy = policies.FirstOrDefault(p => p.Id == "my-policy-id");
if (policy != null)
{
    policy.AccessPolicy.ExpiresOn = DateTimeOffset.UtcNow.AddMinutes(-1);
}

await containerClient.SetAccessPolicyAsync(permissions: policies);

Console.WriteLine("Policy expired - all associated SAS immediately invalid");
```
## Сравнение способов отзыва доступа (Revocation Methods)

| Метод | Эффект | Восстановление | Когда использовать |
|--------|--------|---------------|--------------------|
| **Удаление политики** | Немедленный отзыв всех связанных SAS | Невозможно восстановить | Постоянный отзыв доступа |
| **Изменение identifier** | Разрывает связь с существующими SAS | Можно создать новую политику с теми же параметрами | Ротация доступа |
| **Установка прошедшей даты окончания** | Немедленное истечение срока действия | Можно продлить позже | Временная блокировка |

---

## Разбор методов

### Удаление политики
- Все SAS, связанные с этой политикой, сразу становятся недействительными.
- Используется при полном отзыве доступа.
- Восстановление невозможно без создания новой политики и новых SAS.

### Изменение identifier
- SAS больше не сможет найти соответствующую политику.
- Подходит для ротации и обновления доступа.
- Позволяет создать новую политику с теми же настройками.

### Установка прошедшего срока действия
- Доступ прекращается немедленно.
- Позже можно продлить срок действия.
- Подходит для временной блокировки.

---

## Важно для AZ-204

- Stored Access Policy позволяет отзывать SAS без регенерации ключей.
- Удаление политики — самый радикальный способ.
- Изменение срока действия — гибкий метод управления.
- Account SAS не поддерживает Stored Access Policy.

---

> 🎯 Частый экзаменационный вопрос:  
Как временно приостановить доступ по SAS без его удаления?  
Ответ — установить прошедшую дату окончания действия в Stored Access Policy.

## Managing Multiple Policies

### List All Policies

```csharp
var accessPolicy = await containerClient.GetAccessPolicyAsync();

foreach (var policy in accessPolicy.Value.SignedIdentifiers)
{
    Console.WriteLine($"Policy ID: {policy.Id}");
    Console.WriteLine($"  Permissions: {policy.AccessPolicy.Permissions}");
    Console.WriteLine($"  Starts: {policy.AccessPolicy.StartsOn}");
    Console.WriteLine($"  Expires: {policy.AccessPolicy.ExpiresOn}");
}
```

**Azure CLI**:

```bash
# List container policies
az storage container policy list \
    --container-name my-container \
    --account-name mystorageaccount \
    --account-key $ACCOUNT_KEY
```

### Create Multiple Policies

```csharp
var policies = new List<BlobSignedIdentifier>
{
    new BlobSignedIdentifier
    {
        Id = "read-only",
        AccessPolicy = new BlobAccessPolicy
        {
            ExpiresOn = DateTimeOffset.UtcNow.AddDays(7),
            Permissions = "r"
        }
    },
    new BlobSignedIdentifier
    {
        Id = "read-write",
        AccessPolicy = new BlobAccessPolicy
        {
            ExpiresOn = DateTimeOffset.UtcNow.AddDays(7),
            Permissions = "rw"
        }
    },
    new BlobSignedIdentifier
    {
        Id = "full-access",
        AccessPolicy = new BlobAccessPolicy
        {
            ExpiresOn = DateTimeOffset.UtcNow.AddDays(1),
            Permissions = "rwdl"
        }
    }
};

await containerClient.SetAccessPolicyAsync(permissions: policies);
```

⚠️ **Limit**: Maximum **5 policies** per container, queue, table, or file share.

### Remove All Policies

```csharp
// Pass empty array to remove all policies
await containerClient.SetAccessPolicyAsync(permissions: Array.Empty<BlobSignedIdentifier>());

Console.WriteLine("All policies removed");
```

**Azure CLI**:

```bash
# Set empty policy (removes all)
az storage container policy list \
    --container-name my-container \
    --account-name mystorageaccount \
    --account-key $ACCOUNT_KEY \
    --query "[]" \
    | az storage container set-metadata \
        --container-name my-container \
        --account-name mystorageaccount
```

## Best Practices

### 1. Use Stored Policies for Long-Lived SAS

```csharp
// ✅ Good: Stored policy (revocable)
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "data",
    BlobName = "file.csv",
    Resource = "b",
    Identifier = "long-term-access"  // References policy
};

// ❌ Bad: Direct SAS for long-lived access (cannot revoke)
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "data",
    BlobName = "file.csv",
    Resource = "b",
    ExpiresOn = DateTimeOffset.UtcNow.AddYears(1),  // 1 year, not revocable!
    Permissions = "rw"
};
```

### 2. Create Policy Before Distributing SAS

```csharp
// ✅ Good: Policy exists first
await CreateStoredAccessPolicy("my-policy");
var sasUri = GenerateSasWithPolicy("my-policy");
await DistributeToClients(sasUri);

// ❌ Bad: SAS before policy
var sasUri = GenerateSasWithPolicy("non-existent-policy");  // Will fail
await CreateStoredAccessPolicy("my-policy");
```

⏱️ **Note**: Policy may take up to **30 seconds** to take effect after creation.

### 3. Use Descriptive Policy Names

```csharp
// ✅ Good: Descriptive
"read-only-reports-2024"
"partner-upload-access"
"temp-download-q1"

// ❌ Bad: Generic
"policy1"
"temp"
"access"
```

### 4. Monitor Policy Expiration

```csharp
// Check expiring policies
var policies = await containerClient.GetAccessPolicyAsync();

foreach (var policy in policies.Value.SignedIdentifiers)
{
    var daysUntilExpiry = (policy.AccessPolicy.ExpiresOn - DateTimeOffset.UtcNow).TotalDays;
    
    if (daysUntilExpiry < 7)
    {
        Console.WriteLine($"⚠️ Policy '{policy.Id}' expires in {daysUntilExpiry:F1} days");
    }
}
```

### 5. Rotate Policies Regularly

```csharp
// Monthly policy rotation
public async Task RotateMonthlyPolicy()
{
    var currentMonth = DateTime.UtcNow.ToString("yyyy-MM");
    var newPolicyId = $"access-{currentMonth}";
    
    // Create new policy
    var newPolicy = new BlobSignedIdentifier
    {
        Id = newPolicyId,
        AccessPolicy = new BlobAccessPolicy
        {
            ExpiresOn = DateTimeOffset.UtcNow.AddMonths(1),
            Permissions = "r"
        }
    };
    
    var policies = new[] { newPolicy };
    await containerClient.SetAccessPolicyAsync(permissions: policies);
    
    // Issue new SAS tokens referencing new policy
    Console.WriteLine($"New policy created: {newPolicyId}");
}
```

## Limitations and Considerations

### Activation Delay

⏱️ **30-second delay**: Policy may take up to 30 seconds to activate.

```csharp
// Create policy
await containerClient.SetAccessPolicyAsync(permissions: new[] { identifier });

// Wait for policy to activate
await Task.Delay(TimeSpan.FromSeconds(35));

// Now safe to generate SAS
var sasUri = blobClient.GenerateSasUri(sasBuilder);
```

**During activation period**: SAS requests may fail with **403 Forbidden**.

### Table Entity Range Restrictions

⚠️ **Cannot specify in stored policy**:
- `startpk` (start partition key)
- `startrk` (start row key)
- `endpk` (end partition key)
- `endrk` (end row key)

These must be specified on the SAS token itself.

### Policy Limit

Maximum **5 stored access policies** per:
- Blob container
- File share
- Queue
- Table

```csharp
// ❌ Will fail: 6th policy
var policies = new List<BlobSignedIdentifier>();
for (int i = 1; i <= 6; i++)
{
    policies.Add(new BlobSignedIdentifier { Id = $"policy{i}", AccessPolicy = new BlobAccessPolicy() });
}

await containerClient.SetAccessPolicyAsync(permissions: policies);  // ERROR
```

### User Delegation SAS Not Supported

❌ Stored access policies only work with **service-level SAS**, not **user delegation SAS**.

```csharp
// ✅ Works: Service SAS
var sasBuilder = new BlobSasBuilder { Identifier = "policy-id" };

// ❌ Does not work: User delegation SAS cannot use policies
var userDelegationKey = await blobServiceClient.GetUserDelegationKeyAsync(...);
// Cannot reference stored policy with user delegation key
```

# Critical Notes

- 💡 **Stored Access Policy** — серверная политика для Service SAS
- 🎯 **Преимущества** — отзыв SAS без регенерации ключей, централизованное управление
- ✅ **Поддержка** — Blob containers, file shares, queues, tables
- ⚠️ **Ограничения** — максимум 5 политик на ресурс, только для Service SAS (не для User Delegation и не для Account SAS)
- 🔄 **Параметры** — могут быть заданы в политике, в токене или разделены между ними
- 📊 **Методы отзыва** — удаление политики, изменение identifier, установка прошедшей даты окончания
- 💡 **Эффект** — изменения применяются ко всем SAS, связанным с политикой
- ✅ **Задержка активации** — до 30 секунд после создания или изменения
- ⚠️ **Best practice** — использовать политики для долгоживущих SAS
- 🔒 **Нельзя указывать** — один и тот же параметр одновременно в политике и в токене

---

# Exam Tips (AZ-204)

## Основы

- Stored Access Policy — серверная политика для дополнительного контроля Service SAS.
- Позволяет централизованно управлять сроками и разрешениями.
- Позволяет отзывать SAS без смены ключей Storage Account.

---

## Поддержка

Поддерживается для:
- Blob containers
- File shares
- Queues
- Tables

Не поддерживается для:
- Отдельных blob/file
- Account SAS
- User Delegation SAS

---

## Ограничения

- Максимум 5 политик на контейнер/share/queue/table.
- Нельзя указывать одинаковые параметры в SAS и в политике.
- После изменения политики может быть задержка до 30 секунд.

---

## Отзыв доступа

- Удаление политики — немедленный и окончательный отзыв.
- Изменение identifier — разрыв связи с SAS.
- Установка прошедшей даты окончания — временная блокировка.

---

## SDK и инструменты

- **Методы SDK**:
   - `SetAccessPolicyAsync`
   - `GetAccessPolicyAsync`
   - `BlobSignedIdentifier`

- **CLI**:
   - `az storage container policy create`
   - `az storage container policy update`
   - `az storage container policy delete`
   - `az storage container policy list`

---

## Важно помнить

- Использовать Stored Access Policy для долгоживущих SAS.
- Для краткоживущих SAS можно задавать параметры напрямую в токене.
- Stored Access Policy позволяет избежать регенерации ключей.

---

> 🎯 Ключевой экзаменационный момент:  
> Как отозвать Service SAS без смены ключа Storage Account?  
> Ответ — изменить или удалить Stored Access Policy.


[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-shared-access-signatures/4-stored-access-policies)
