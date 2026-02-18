# Shared Access Signatures (SAS) Overview

## Ключевые понятия

- **SAS (Shared Access Signature)** — подписанный URI, предоставляющий делегированный доступ к ресурсам Azure Storage
- **Токен** — набор query-параметров с подписью для авторизации
- **Три типа** — User Delegation, Service, Account
- **Безопасность** — ограничение по времени и правам без передачи ключа аккаунта

---

# Что такое Shared Access Signature?

**SAS** — это подписанный URI, который указывает на один или несколько ресурсов хранения и содержит токен с query-параметрами.

Токен определяет:

- Какие операции разрешены
- На какой срок предоставляется доступ
- К какому ресурсу разрешён доступ
- Кем подписан доступ

> 💡 SAS позволяет предоставить временный и ограниченный доступ без раскрытия ключей Storage Account.

---

# Назначение

SAS используется для **делегирования доступа** к ресурсам Azure Storage без передачи account key.

Позволяет:

- Выдавать конкретные разрешения (read, write, delete, list)
- Ограничивать доступ по времени (start time, expiry time)
- Обеспечивать криптографическую подпись
- Отзывать доступ через stored access policy

---

# Типы Shared Access Signatures

## Сравнительная таблица

| Тип | Чем защищён | Уровень доступа | Поддерживаемые сервисы | Когда использовать |
|------|------------|----------------|------------------------|--------------------|
| **User Delegation SAS** | Учётные данные Microsoft Entra ID | Уровень сервиса | Blob Storage, Data Lake Storage | **Наиболее безопасный**, рекомендуется |
| **Service SAS** | Ключ Storage Account | Уровень сервиса | Blob, Queue, Table, Files | Доступ к конкретному сервису |
| **Account SAS** | Ключ Storage Account | Уровень аккаунта | Все сервисы хранения | Кросс-сервисные операции |

---

## Кратко о каждом типе

### User Delegation SAS
- Основан на Azure AD (Microsoft Entra ID).
- Не использует account key.
- Рекомендуется как наиболее безопасный вариант.
- Поддерживается для Blob и Data Lake.

### Service SAS
- Подписывается с использованием account key.
- Ограничен одним сервисом.
- Подходит для сценариев с конкретным типом хранилища.

### Account SAS
- Подписывается account key.
- Позволяет доступ ко всем сервисам аккаунта.
- Используется для операций на уровне всего аккаунта.

---

## Важно для AZ-204

- SAS предоставляет временный доступ.
- Не требуется передавать account key.
- User Delegation SAS — наиболее безопасный вариант.
- Service и Account SAS используют account key.
- Доступ можно ограничить по времени и операциям.

> 🎯 Частый экзаменационный вопрос:  
Как безопасно предоставить временный досту

### 1. User Delegation SAS (Recommended)

**Most secure option** - Uses Microsoft Entra ID credentials:

```csharp
// Create user delegation SAS
BlobServiceClient blobServiceClient = new BlobServiceClient(
    new Uri("https://storageaccount.blob.core.windows.net"),
    new DefaultAzureCredential()
);

// Get user delegation key (requires Microsoft Entra authentication)
UserDelegationKey userDelegationKey = await blobServiceClient
    .GetUserDelegationKeyAsync(DateTimeOffset.UtcNow, DateTimeOffset.UtcNow.AddHours(1));

// Create SAS token
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "container-name",
    BlobName = "blob-name",
    Resource = "b",  // b = blob
    StartsOn = DateTimeOffset.UtcNow,
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};

sasBuilder.SetPermissions(BlobSasPermissions.Read);

BlobUriBuilder uriBuilder = new BlobUriBuilder(blobClient.Uri)
{
    Sas = sasBuilder.ToSasQueryParameters(userDelegationKey, blobServiceClient.AccountName)
};

Uri sasUri = uriBuilder.ToUri();
```

## Преимущества User Delegation SAS

✅ **Без передачи ключа аккаунта**  
Использует аутентификацию через Microsoft Entra ID вместо account key.

✅ **Аудируемость**  
Действия пользователя можно отслеживать через журналы входа и аудит.

✅ **Отзыв доступа**  
Достаточно отключить или удалить учётную запись пользователя, чтобы аннулировать выданные SAS.

✅ **Рекомендуемый подход**  
Microsoft рекомендует использовать именно User Delegation SAS как наиболее безопасный вариант.

---

## Применяется к

- **Blob Storage**
- **Data Lake Storage Gen2**

---

## Важно для AZ-204

- User Delegation SAS основан на Entra ID, а не на account key.
- Обеспечивает более высокий уровень безопасности.
- Позволяет централизованно управлять доступом.
- Поддерживается только для Blob и Data Lake Gen2.

> 🎯 Частый экзаменационный вопрос:  
Какой тип SAS наиболее безопасен?  
Ответ — User Delegation SAS.


### 2. Service SAS

**Service-level access** secured with storage account key:

```csharp
// Create service SAS for blob
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "documents",
    BlobName = "report.pdf",
    Resource = "b",
    StartsOn = DateTimeOffset.UtcNow,
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(2)
};

sasBuilder.SetPermissions(BlobSasPermissions.Read);

// Sign with storage account key
BlobClient blobClient = new BlobClient(
    new Uri("https://storageaccount.blob.core.windows.net/documents/report.pdf"),
    new StorageSharedKeyCredential(accountName, accountKey)
);

Uri sasUri = blobClient.GenerateSasUri(sasBuilder);
```

## Service SAS — сценарии использования

### Когда применяется

- Доступ к одному конкретному сервису хранения  
  (Blob, Queue, Table или Files)

- Когда Microsoft Entra ID недоступен  
  или не используется в архитектуре

- В legacy-приложениях  
  где аутентификация построена на account key

---

## Поддерживаемые сервисы

- **Blob Storage**
- **Queue Storage**
- **Table Storage**
- **Azure Files**

---

## Особенности

- Подписывается с использованием **Storage Account Key**.
- Ограничивается конкретным сервисом.
- Позволяет задать:
    - Разрешения (read, write, delete и др.)
    - Временные рамки
    - Ограничения по IP (при необходимости)

---

## Важно для AZ-204

- Service SAS использует account key.
- Менее безопасен по сравнению с User Delegation SAS.
- Подходит для сценариев без Entra ID.
- Доступ ограничивается одним сервисом хранения.

> 🎯 Частый экзаменационный вопрос:  
Какой тип SAS использовать для доступа только к Blob Storage без Entra ID?  
Ответ — Service SAS.


### 3. Account SAS

**Account-level access** to multiple services:

```csharp
// Create account SAS
AccountSasBuilder sasBuilder = new AccountSasBuilder()
{
    Services = AccountSasServices.Blobs | AccountSasServices.Queues,
    ResourceTypes = AccountSasResourceTypes.Service | AccountSasResourceTypes.Container | AccountSasResourceTypes.Object,
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1),
    Protocol = SasProtocol.Https
};

sasBuilder.SetPermissions(AccountSasPermissions.Read | AccountSasPermissions.List);

StorageSharedKeyCredential credential = new StorageSharedKeyCredential(accountName, accountKey);
string sasToken = sasBuilder.ToSasQueryParameters(credential).ToString();

string sasUrl = $"https://{accountName}.blob.core.windows.net?{sasToken}";
```

## Account SAS — сценарии использования

### Когда применяется

- Доступ сразу к нескольким сервисам хранения
- Операции между разными сервисами
- Копирование данных между Storage Account
- Массовые операции на уровне аккаунта

---

## Поддерживаемые сервисы

- Blob Storage
- Queue Storage
- Table Storage
- Azure Files

> 💡 Account SAS работает на уровне всего Storage Account.

---

# Как работает Shared Access Signature

## Структура SAS URI

**Полный URI с SAS-токеном** состоит из:

1. Базового URL ресурса хранения
2. Набора query-параметров (SAS token)
3. Криптографической подписи

### Общая структура
https://<storage-account>.blob.core.windows.net/<container>/<blob>?<SAS-token>


---

## Что включает SAS-токен

SAS-токен содержит параметры, определяющие:

- Разрешения (sp)
- Время начала и окончания действия (st, se)
- Версию API (sv)
- Тип ресурса (sr)
- Подпись (sig)

> 💡 Подпись (sig) создаётся на основе ключа аккаунта или user delegation key.

---

## Важно для AZ-204

- SAS передаётся как query-параметры в URL.
- Доступ возможен только в пределах заданных разрешений и времени.
- Account SAS даёт доступ ко всем сервисам хранения.
- Копирование между аккаунтами часто требует SAS.

> 🎯 Частый экзаменационный вопрос:  
Как предоставить временный доступ к нескольким сервисам хранения одновременно?  
Ответ — использовать Account SAS.



```
https://storageaccount.blob.core.windows.net/container/blob.jpg?sp=r&st=2020-01-20T11:42:32Z&se=2020-01-20T19:42:32Z&spr=https&sv=2019-02-02&sr=b&sig=SrW1HZ5Nb6MbRzTbXCaPm%2BJiSEn15tC91Y4umMPwVZs%3D
```


---

# Компоненты SAS-токена

| Параметр | Описание | Пример | Возможные значения |
|------------|----------|---------|--------------------|
| **sp** | **Permissions (разрешения)** | `sp=r` | `r` (read), `w` (write), `d` (delete), `l` (list), `a` (add), `c` (create) |
| **st** | **Start time** (UTC) | `st=2020-01-20T11:42:32Z` | Формат ISO 8601 |
| **se** | **Expiry time** (UTC) | `se=2020-01-20T19:42:32Z` | Формат ISO 8601 |
| **spr** | **Protocol** | `spr=https` | `https`, `http,https` |
| **sv** | **Storage API version** | `sv=2019-02-02` | Строка версии API |
| **sr** | **Resource type** | `sr=b` | `b` (blob), `c` (container), `bs` (blob service) |
| **sig** | **Signature** | `sig=SrW1HZ5...` | Base64-кодированная HMAC-SHA256 подпись |

---

## Что важно понимать

### `sp` — Permissions
Определяет, какие операции разрешены.  
Можно комбинировать, например: `sp=rw`.

### `st` и `se` — Временные ограничения
Определяют период действия SAS.  
Если текущее время вне диапазона — доступ запрещён.

### `spr` — Протокол
Рекомендуется использовать только `https`.

### `sig` — Подпись
Криптографическая подпись, подтверждающая подлинность токена.  
Создаётся с использованием:

- Account key  
  или
- User delegation key

---

## Важно для AZ-204

- SAS-токен передаётся в query-параметрах URL.
- Разрешения и срок действия строго ограничены.
- `sig` обеспечивает безопасность.
- Без корректной подписи SAS недействителен.
- Использование только `https` — best practice.

> 🎯 Частый экзаменационный вопрос:  
Как ограничить SAS только чтением и сроком на 1 час?  
Ответ — задать `sp=r` и корректно настроить `se`.

### Permissions Values

```
a = Add (Queue, Table)
c = Create (Blob, File)
d = Delete (Blob, Queue, Table, File)
l = List (Blob container, Queue, Table, File share)
r = Read (Blob, Queue, Table, File)
w = Write (Blob, Queue, Table, File)
```

**Combined permissions**:

```
sp=rl     # Read + List
sp=rw     # Read + Write
sp=acdlrw # All permissions
```

### Example: Decode a SAS Token

```
https://medicalrecords.blob.core.windows.net/patient-images/patient-116139-nq8z7f.jpg?sp=r&st=2020-01-20T11:42:32Z&se=2020-01-20T19:42:32Z&spr=https&sv=2019-02-02&sr=b&sig=SrW1HZ5Nb6MbRzTbXCaPm%2BJiSEn15tC91Y4umMPwVZs%3D
```

**Decoded parameters**:

| Parameter | Value | Meaning |
|-----------|-------|---------|
| `sp` | `r` | **Read-only** access |
| `st` | `2020-01-20T11:42:32Z` | Valid **from** 11:42:32 UTC |
| `se` | `2020-01-20T19:42:32Z` | Valid **until** 19:42:32 UTC (8 hours) |
| `spr` | `https` | **HTTPS only** |
| `sv` | `2019-02-02` | Storage API version 2019-02-02 |
| `sr` | `b` | Access to **blob** |
| `sig` | `SrW1HZ5...` | Cryptographic signature |

## Creating SAS Tokens

### Using Azure Storage SDK (.NET)

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Sas;

// Create blob client
BlobClient blobClient = new BlobClient(
    new Uri("https://storageaccount.blob.core.windows.net/container/file.txt"),
    new StorageSharedKeyCredential(accountName, accountKey)
);

// Configure SAS
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "container",
    BlobName = "file.txt",
    Resource = "b",
    StartsOn = DateTimeOffset.UtcNow,
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};

// Set permissions
sasBuilder.SetPermissions(BlobSasPermissions.Read);

// Generate SAS URI
Uri sasUri = blobClient.GenerateSasUri(sasBuilder);
Console.WriteLine($"SAS URI: {sasUri}");
```

### Using Azure CLI

```bash
# Generate blob SAS token
az storage blob generate-sas \
    --account-name mystorageaccount \
    --container-name mycontainer \
    --name myblob.txt \
    --permissions r \
    --expiry 2024-12-31T23:59:00Z \
    --https-only \
    --output tsv

# Generate container SAS token
az storage container generate-sas \
    --account-name mystorageaccount \
    --name mycontainer \
    --permissions rl \
    --expiry 2024-12-31T23:59:00Z \
    --https-only \
    --output tsv

# Generate account SAS token
az storage account generate-sas \
    --account-name mystorageaccount \
    --account-key <key> \
    --services b \
    --resource-types sco \
    --permissions rl \
    --expiry 2024-12-31T23:59:00Z \
    --https-only \
    --output tsv
```

### Using PowerShell

```powershell
# Connect to storage account
$context = New-AzStorageContext -StorageAccountName "mystorageaccount" -StorageAccountKey $accountKey

# Generate blob SAS
$sasToken = New-AzStorageBlobSASToken `
    -Context $context `
    -Container "mycontainer" `
    -Blob "myblob.txt" `
    -Permission r `
    -ExpiryTime (Get-Date).AddHours(2) `
    -Protocol HttpsOnly

# Full URI
$blobUri = "https://mystorageaccount.blob.core.windows.net/mycontainer/myblob.txt$sasToken"
Write-Host $blobUri
```

## Best Practices

### 1. Always Use HTTPS

```csharp
// ✅ Good: HTTPS only
sasBuilder.Protocol = SasProtocol.Https;

// ❌ Bad: Allows HTTP
sasBuilder.Protocol = SasProtocol.HttpsAndHttp;
```

**Why**: Prevents man-in-the-middle attacks and token interception.

### 2. Use User Delegation SAS When Possible

```csharp
// ✅ Best: User delegation SAS with Microsoft Entra ID
var userDelegationKey = await blobServiceClient.GetUserDelegationKeyAsync(
    DateTimeOffset.UtcNow,
    DateTimeOffset.UtcNow.AddHours(1)
);

var sas = sasBuilder.ToSasQueryParameters(userDelegationKey, accountName);

// ⚠️ OK but less secure: Service SAS with storage key
var credential = new StorageSharedKeyCredential(accountName, accountKey);
var sas = sasBuilder.ToSasQueryParameters(credential);
```

**Why**: Eliminates need to store account keys in code, provides better audit trail.

### 3. Minimum Required Permissions

```csharp
// ✅ Good: Only what's needed
sasBuilder.SetPermissions(BlobSasPermissions.Read);

// ❌ Bad: Excessive permissions
sasBuilder.SetPermissions(
    BlobSasPermissions.Read | 
    BlobSasPermissions.Write | 
    BlobSasPermissions.Delete
);
```

**Why**: Reduces impact if SAS is compromised.

### 4. Shortest Useful Expiration Time

```csharp
// ✅ Good: Short-lived (1 hour)
sasBuilder.ExpiresOn = DateTimeOffset.UtcNow.AddHours(1);

// ⚠️ Risky: Long-lived (1 year)
sasBuilder.ExpiresOn = DateTimeOffset.UtcNow.AddYears(1);
```

**Why**: Limits exposure window if token is compromised.

### 5. Use Stored Access Policies for Service SAS

```csharp
// ✅ Good: Use stored access policy (revocable)
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "container",
    Identifier = "policy-id"  // References stored policy
};

// ⚠️ Less flexible: Direct SAS (not revocable without key rotation)
BlobSasBuilder sasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "container",
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};
```

## Почему использовать Stored Access Policy

**Позволяет отозвать SAS без регенерации ключей Storage Account.**

Если SAS связан со Stored Access Policy:

- Можно изменить срок действия
- Можно изменить разрешения
- Можно полностью отозвать доступ
- Не требуется регенерация account key

> 💡 Без stored access policy отозвать уже выданный SAS невозможно, пока не истечёт срок его действия или не будет регенерирован ключ.

---

# 6️⃣ Рассмотрите использование Middle-Tier Service

## Когда НЕ стоит использовать SAS напрямую

- Высокие требования к безопасности
- Необходима валидация бизнес-логики
- Сложные правила доступа
- Работа с чувствительными данными

---

## Альтернатива: Middle-Tier Service

Создайте промежуточный сервис, который:

- Аутентифицирует пользователей
- Проверяет бизнес-правила
- Генерирует краткоживущие SAS-токены
- Логирует действия для аудита

---

## Преимущества такого подхода

- Централизованный контроль доступа
- Возможность применять сложные правила
- Полный аудит операций
- Минимизация времени жизни SAS

---

## Важно для AZ-204

- SAS удобен, но не всегда подходит для high-security сценариев.
- Stored Access Policy позволяет отзывать SAS.
- Middle-tier обеспечивает дополнительный уровень контроля.
- Для чувствительных данных предпочтителен серверный контроль доступа.

> 🎯 Частый экзаменационный вопрос:  
Как обеспечить дополнительную валидацию перед доступом к Blob Storage?  
Ответ — использовать middle-tier сервис вместо прямого SAS.


### 7. IP Address Restrictions (When Possible)

```csharp
// Restrict to specific IP
sasBuilder.IPRange = new SasIPRange(IPAddress.Parse("203.0.113.5"));

// Restrict to IP range
sasBuilder.IPRange = new SasIPRange(
    IPAddress.Parse("203.0.113.0"),
    IPAddress.Parse("203.0.113.255")
);
```
## 8️⃣ Мониторинг использования SAS

### Используйте Azure Monitor для отслеживания:

- Ошибок аутентификации через SAS
- Нетипичных шаблонов доступа
- Географических аномалий
- Чрезмерных операций чтения или записи

---

## Что рекомендуется контролировать

### 🔎 Authentication failures
- Повторяющиеся неудачные попытки доступа
- Возможные попытки подбора или использования истёкшего SAS

### 🌍 Географические аномалии
- Доступ из неожиданных регионов
- Резкая смена географии запросов

### 📈 Аномальная активность
- Необычно большое количество операций
- Массовые скачивания или загрузки

---

## Инструменты мониторинга

- Azure Monitor
- Diagnostic logs Storage Account
- Log Analytics
- Azure Alerts

---

## Best Practices

- Настроить оповещения (Alerts) на подозрительную активность
- Анализировать журналы входа и операций
- Ограничивать SAS по IP, если возможно
- Использовать короткие сроки действия

---

## Важно для AZ-204

- SAS сам по себе не логирует бизнес-контекст — нужен мониторинг.
- Необходимо включить диагностические логи.
- Аномальная активность может указывать на компрометацию SAS.
- Мониторинг — часть общей стратегии безопасности.

> 🎯 Частый экзаменационный вопрос:  
Как обнаружить злоупотребление SAS?  
Ответ — использовать Azure Monitor и диагностические логи.


```kusto
// Azure Monitor query for SAS usage
StorageBlobLogs
| where AuthenticationType == "SAS"
| where TimeGenerated > ago(1d)
| summarize count() by bin(TimeGenerated, 1h), StatusText
| render timechart
```

# Security Considerations

## Риски использования SAS

| Риск | Последствия | Меры снижения |
|------|------------|--------------|
| **Перехват токена** | Неавторизованный доступ | Использовать только HTTPS |
| **Распространение токена** | Неконтролируемый доступ | Короткий срок действия |
| **Избыточные разрешения** | Изменение или удаление данных | Принцип минимальных привилегий |
| **Долгоживущие токены** | Увеличенная зона риска | Регулярная ротация |
| **Компрометация account key** | Недействительность всех SAS | Использовать User Delegation SAS |

---

## Когда НЕ стоит использовать SAS

Рассмотрите альтернативы, если:

- Недопустим риск утечки токена
- Требуется немедленный отзыв доступа
- Нужна сложная логика авторизации
- Работа с высокочувствительными данными (PHI, PCI и др.)

---

## Альтернативы

- **Managed Identity** — для взаимодействия Azure-сервисов
- **Microsoft Entra ID** — для аутентификации пользователей и приложений
- **Middle-tier сервис** — для реализации бизнес-логики
- **Azure AD B2C** — для сценариев с внешними пользователями

---

# SAS vs Другие методы аутентификации

| Метод | Когда использовать | Плюсы | Минусы |
|--------|-------------------|--------|--------|
| **SAS** | Временный делегированный доступ | Нет передачи ключей, гибкие права | Возможен перехват |
| **Storage Account Key** | Полный административный доступ | Максимальный контроль | Высокий риск безопасности |
| **Microsoft Entra ID** | Аутентификация пользователей/приложений | Наиболее безопасный, отзыв доступа | Требует настройки Azure AD |
| **Managed Identity** | Azure service-to-service | Нет секретов в коде | Только для Azure-сервисов |
| **Anonymous Access** | Публичные данные | Простота | Отсутствие защиты |

---

# Critical Notes

- 💡 **Три типа SAS** — User Delegation (лучший), Service, Account
- 🎯 **User Delegation SAS** — самый безопасный, использует Entra ID
- ✅ **Service SAS** — доступ к одному сервису через account key
- ⚠️ **Account SAS** — доступ к нескольким сервисам через account key
- 🔄 **Компоненты токена** — `sp`, `st`, `se`, `sig`
- 📊 **HTTPS обязателен** — предотвращает перехват
- 💡 **Короткий срок действия** — минимизирует риск
- ✅ **Минимальные разрешения** — только необходимые операции
- ⚠️ **Отзыв доступа** — использовать stored access policies
- 🔒 **Рекомендация** — User Delegation > Service > Account

---

# Exam Tips (AZ-204)

## Основы

- SAS — подписанный URI для делегированного доступа к Azure Storage.
- Не требует передачи account key клиенту.
- Ограничивается временем и разрешениями.

---

## Типы SAS

- **User Delegation SAS**
    - Основан на Entra ID
    - Самый безопасный
    - Только Blob и Data Lake

- **Service SAS**
    - Использует account key
    - Один сервис

- **Account SAS**
    - Использует account key
    - Несколько сервисов

---

## Параметры токена

- `sp` — разрешения
- `st` — время начала
- `se` — время окончания
- `sr` — тип ресурса
- `sig` — криптографическая подпись

Разрешения:
- `r` — read
- `w` — write
- `d` — delete
- `l` — list
- `a` — add
- `c` — create

---

## Best Practices

- Использовать только HTTPS (`spr=https`)
- Минимальные разрешения
- Минимально возможный срок действия
- Использовать User Delegation SAS
- Применять stored access policies для отзыва

---

## Часто проверяется

- Какой тип SAS наиболее безопасен.
- Различие между Service и Account SAS.
- Какие параметры управляют сроком действия.
- Как отозвать SAS без регенерации ключа.
- Когда использовать middle-tier вместо прямого SAS.

---

> 🎯 Ключевая идея:  
> Делегировать доступ безопасно, минимально и на короткий срок.  
> Предпочитать User Delegation SAS.


[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-shared-access-signatures/2-shared-access-signatures-overview)
