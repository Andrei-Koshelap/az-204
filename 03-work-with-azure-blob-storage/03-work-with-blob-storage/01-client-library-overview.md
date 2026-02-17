# Azure Blob Storage Client Library Overview
(Обзор клиентской библиотеки Azure Blob Storage)

## Introduction (Введение)

Клиентские библиотеки **Azure Storage для .NET** предоставляют удобный API для работы с Azure Blob Storage.

Microsoft рекомендует использовать **версию 12.x** для новых приложений (SDK нового поколения с улучшенной архитектурой и async-first подходом).

> 💡 Версия 12.x использует современную модель клиентов и поддерживает dependency injection, что особенно важно в ASP.NET Core приложениях.

---

## Core Client Classes (Основные классы клиента)

Ниже перечислены ключевые классы библиотеки Azure Blob Storage:

| Class | Purpose |
|--------|----------|
| **BlobServiceClient** | Управление ресурсами Blob Service и контейнерами. Точка входа в сервис. |
| **BlobContainerClient** | Управление контейнерами и их blob. |
| **BlobClient** | Работа с конкретным blob (загрузка, скачивание, удаление, свойства). |
| **BlobClientOptions** | Конфигурация клиента (retry, logging, transport и др.). |
| **BlobUriBuilder** | Удобное построение и модификация URI (account, container, blob). |

---

## Архитектурная модель (Client Hierarchy)

## SDK построен по иерархии:


```
BlobServiceClient (Storage Account Level)
    ↓
BlobContainerClient (Container Level)
    ↓
BlobClient (Blob Level)
```

- `BlobServiceClient` — уровень Storage Account
- `BlobContainerClient` — уровень контейнера
- `BlobClient` — уровень конкретного blob

> 🎯 Экзаменационный момент AZ-204:  
> Нужно понимать иерархию клиентов и какой класс используется на каждом уровне.


### Navigation Pattern

```csharp
// Start at service level
BlobServiceClient serviceClient = new BlobServiceClient(connectionString);

// Navigate to container level
BlobContainerClient containerClient = serviceClient.GetBlobContainerClient("mycontainer");

// Navigate to blob level
BlobClient blobClient = containerClient.GetBlobClient("myblob.txt");
```

---
## NuGet Packages (Пакеты NuGet)

Клиентская библиотека Azure Blob Storage распространяется через несколько пакетов NuGet.

---

### 1. Azure.Storage.Blobs (Основной пакет)

**Назначение:**  
Содержит основные классы (client objects) для работы с:

- Blob Service
- Контейнерами
- Blob

Это основной пакет, который используется в большинстве приложений.

---

### Key Classes (Ключевые классы)

- `BlobServiceClient`
- `BlobContainerClient`
- `BlobClient`
- `BlobClientOptions`
- `BlobUriBuilder`

---

### Когда использовать

- Для загрузки и скачивания файлов
- Для создания и удаления контейнеров
- Для работы со свойствами и метаданными blob
- Для управления access tier

---

> 💡 Практический совет:  
> В ASP.NET Core обычно регистрируют `BlobServiceClient` через Dependency Injection и затем получают `BlobContainerClient` через фабричный метод.

> 🎯 Экзаменационный момент AZ-204:  
> Основной пакет для работы с Blob Storage в .NET — **Azure.Storage.Blobs**.


**Installation:**
```bash
dotnet add package Azure.Storage.Blobs
```

### 2. Azure.Storage.Blobs.Specialized

**Назначение:**  
Содержит специализированные классы для работы с конкретными типами blob:

- Block blobs
- Append blobs
- Page blobs
- Lease-операции

Используется, когда требуется доступ к функциональности, специфичной для определённого типа blob.

---

### Key Classes (Ключевые классы)

- `BlockBlobClient` — операции, специфичные для block blob (загрузка блоками, commit block list и др.)
- `AppendBlobClient` — добавление данных в конец blob (логирование, streaming append)
- `PageBlobClient` — работа со страничными blob (виртуальные диски, random write)
- `BlobLeaseClient` — управление lease (блокировка blob для предотвращения конкурентных изменений)

---

### Когда использовать

- `BlockBlobClient` — при загрузке больших файлов по частям
- `AppendBlobClient` — для логов и сценариев append-only
- `PageBlobClient` — для VHD и сценариев с произвольной записью
- `BlobLeaseClient` — для реализации распределённой блокировки

---

> 🎯 Экзаменационный момент AZ-204:  
> Block blob — самый распространённый тип.  
> Append blob — для добавления данных.  
> Page blob — для виртуальных дисков и random write.


**Example Usage:**
```csharp
using Azure.Storage.Blobs.Specialized;

// Work with block blobs specifically
BlockBlobClient blockBlobClient = containerClient.GetBlockBlobClient("data.bin");
await blockBlobClient.UploadAsync(stream);

// Work with append blobs (logging scenarios)
AppendBlobClient appendBlobClient = containerClient.GetAppendBlobClient("log.txt");
await appendBlobClient.CreateAsync();
await appendBlobClient.AppendBlockAsync(logStream);
```

### 3. Azure.Storage.Blobs.Models

### 3. Общие типы и вспомогательные элементы (Utility Types)

**Назначение:**  
Содержит вспомогательные классы, структуры и перечисления, используемые при работе с Azure Blob Storage.

---

### Contains (Содержит)

- Перечисления (enumerations), например:
    - `AccessTier` — Hot, Cool, Cold, Archive
    - `BlobType` — BlockBlob, AppendBlob, PageBlob
    - `PublicAccessType` — Blob, Container, None

- Response models (модели ответов)
- Request options (параметры запроса, retry-настройки и т.д.)
- Типы ошибок (исключения SDK)

---

### Практическое значение

Эти типы используются:

- При установке уровня доступа (`AccessTier`)
- При создании контейнеров с публичным доступом
- При обработке исключений
- При конфигурации поведения запросов

---

> 🎯 Экзаменационный момент AZ-204:  
> Нужно понимать разницу между типами blob (`BlobType`) и уровнями доступа (`AccessTier`) — это разные концепции.


**Example:**
```csharp
using Azure.Storage.Blobs.Models;

// Use enumerations and models
BlobUploadOptions options = new BlobUploadOptions
{
    AccessTier = AccessTier.Cool,
    HttpHeaders = new BlobHttpHeaders
    {
        ContentType = "application/json"
    }
};
```

---

## Package Namespace Structure

```
Azure.Storage.Blobs
├── BlobServiceClient
├── BlobContainerClient
├── BlobClient
├── BlobClientOptions
└── BlobUriBuilder

Azure.Storage.Blobs.Specialized
├── BlockBlobClient
├── AppendBlobClient
├── PageBlobClient
└── BlobLeaseClient

Azure.Storage.Blobs.Models
├── AccessTier (enum)
├── BlobType (enum)
├── BlobProperties
├── BlobHttpHeaders
└── ... (other models and types)
```

---

## Installation

### Using .NET CLI

```bash
# Install primary package
dotnet add package Azure.Storage.Blobs

# Install specialized package (if needed)
dotnet add package Azure.Storage.Blobs.Specialized

# Install identity package for authentication
dotnet add package Azure.Identity
```

### Using Package Manager Console

```powershell
Install-Package Azure.Storage.Blobs
Install-Package Azure.Storage.Blobs.Specialized
Install-Package Azure.Identity
```

### Using Visual Studio

1. Right-click on project → Manage NuGet Packages
2. Search for "Azure.Storage.Blobs"
3. Click Install

---

## Basic Usage Example

### Complete Example with All Packages

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;
using Azure.Storage.Blobs.Specialized;

namespace BlobStorageExample
{
    public class BlobOperations
    {
        private readonly string accountName = "mystorageaccount";
        private readonly string containerName = "mycontainer";
        
        public async Task BasicOperationsAsync()
        {
            // 1. Create service client (Azure.Storage.Blobs)
            var serviceClient = new BlobServiceClient(
                new Uri($"https://{accountName}.blob.core.windows.net"),
                new DefaultAzureCredential());
            
            // 2. Get container client (Azure.Storage.Blobs)
            var containerClient = serviceClient.GetBlobContainerClient(containerName);
            await containerClient.CreateIfNotExistsAsync();
            
            // 3. Upload blob (Azure.Storage.Blobs)
            var blobClient = containerClient.GetBlobClient("data.txt");
            await blobClient.UploadAsync("./data.txt");
            
            // 4. Work with specialized blob client (Azure.Storage.Blobs.Specialized)
            var blockBlobClient = containerClient.GetBlockBlobClient("block.bin");
            await blockBlobClient.UploadAsync(stream);
            
            // 5. Use models and enums (Azure.Storage.Blobs.Models)
            var options = new BlobUploadOptions
            {
                AccessTier = AccessTier.Cool,
                HttpHeaders = new BlobHttpHeaders
                {
                    ContentType = "text/plain"
                }
            };
        }
    }
}
```

---

## BlobClientOptions Configuration

The `BlobClientOptions` class provides configuration for client behavior:

```csharp
var options = new BlobClientOptions
{
    // Retry configuration
    Retry =
    {
        MaxRetries = 5,
        Delay = TimeSpan.FromSeconds(2),
        MaxDelay = TimeSpan.FromSeconds(10),
        Mode = RetryMode.Exponential
    },
    
    // Diagnostics
    Diagnostics =
    {
        IsLoggingEnabled = true,
        ApplicationId = "MyApp"
    }
};

var serviceClient = new BlobServiceClient(connectionString, options);
```

---

## Version Recommendations (Рекомендации по версиям)

| Version | Status | Recommendation |
|----------|---------|----------------|
| **12.x** | Current (Track 2 SDK) | ✅ **Рекомендуется для новых приложений** |
| 11.x | Legacy (Track 1 SDK) | ⚠️ Рекомендуется миграция на 12.x |
| < 11.x | Deprecated | ❌ Больше не поддерживается |

---

### Почему 12.x — стандарт для новых проектов

Версия 12.x относится к **Track 2 Azure SDK**, полностью переработанной архитектуре библиотек Azure.

### Version 12.x Benefits (Преимущества 12.x)

✅ Современная модель `async/await` (async-first API)  
✅ Улучшенная производительность  
✅ Более предсказуемая и унифицированная обработка ошибок  
✅ Интеграция с **Azure.Identity** (Managed Identity, DefaultAzureCredential и др.)  
✅ Единый и консистентный API-стиль во всех Azure SDK  
✅ Активная поддержка и регулярные обновления

---

### Что изменилось по сравнению с 11.x

- Новая иерархия клиентов (BlobServiceClient → BlobContainerClient → BlobClient)
- Отказ от старых CloudBlob* классов
- Улучшенная DI-интеграция (особенно для ASP.NET Core)
- Более чистая модель response/exception

---

### Актуальность

На текущий момент:
- 12.x — основная поддерживаемая ветка
- 11.x не развивается и используется только в legacy-проектах
- Новые возможности Azure Storage добавляются только в Track 2 SDK

---

> 🎯 Экзаменационный момент AZ-204:  
> Для новых .NET-приложений следует использовать **Azure.Storage.Blobs 12.x**.


---

## Migration Guide: 11.x → 12.x (Краткая шпаргалка по миграции)

Переход с версии 11.x (Track 1) на 12.x (Track 2) требует изменений в API, так как архитектура SDK была переработана.

---

### 1️⃣ Основное изменение — модель клиентов

**11.x (Track 1):**
```csharp
CloudStorageAccount
CloudBlobClient
CloudBlobContainer
CloudBlockBlob
```
12.x (Track 2):

BlobServiceClient
BlobContainerClient
BlobClient
BlockBlobClient
---

2️⃣ Создание клиента

11.x:

CloudStorageAccount.Parse(connectionString);


12.x:

new BlobServiceClient(connectionString);


Или через Azure Identity:

new BlobServiceClient(new Uri(endpoint), new DefaultAzureCredential());


💡 12.x нативно поддерживает Managed Identity и Azure AD.

3️⃣ Async-first подход

В 12.x большинство методов имеют async-версии по умолчанию:

await blobClient.UploadAsync(stream);
await blobClient.DownloadToAsync(filePath);


В 11.x async-поддержка была менее консистентной.

4️⃣ Response модель

В 12.x методы возвращают:

Response<T>


Например:

Response<BlobContentInfo> response = await blobClient.UploadAsync(stream);


Это позволяет получать:

HTTP статус

Headers

Value (результат операции)

5️⃣ Исключения

В 12.x используется единый тип:

RequestFailedException


В 11.x применялись разные типы исключений.

6️⃣ Консистентность API

Track 2 SDK (12.x):

Использует единый стиль во всех Azure SDK

Имеет одинаковую модель клиентов

Поддерживает Dependency Injection

Когда обязательно мигрировать?

Новый проект → сразу 12.x

Нужна Azure AD / Managed Identity → 12.x

Планируется долгосрочная поддержка → 12.x

Что учитывать при миграции

⚠️ Потребуется:

Переписать код создания клиентов

Обновить обработку исключений

Проверить логику загрузки/скачивания

Протестировать работу с метаданными и Access Tier

🎯 Экзаменационный момент AZ-204:
Новые .NET-приложения используют Azure.Storage.Blobs 12.x (Track 2 SDK),
классы CloudBlob* относятся к legacy (11.x).

## Authentication Integration (Интеграция аутентификации)

Версия 12.x нативно интегрируется с пакетом **Azure.Identity**:

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;

// DefaultAzureCredential (recommended)
var serviceClient = new BlobServiceClient(
    new Uri("https://account.blob.core.windows.net"),
    new DefaultAzureCredential());

// Managed Identity
var serviceClient = new BlobServiceClient(
    new Uri("https://account.blob.core.windows.net"),
    new ManagedIdentityCredential());

// Connection String
var serviceClient = new BlobServiceClient(connectionString);
```

---

## Key Features by Package (Основные возможности по пакетам)

### Azure.Storage.Blobs

- ✅ Создание и удаление контейнеров
- ✅ Загрузка и скачивание blob
- ✅ Получение списка blob
- ✅ Копирование blob
- ✅ Работа со свойствами и метаданными
- ✅ Управление access tier
- ✅ Генерация SAS-токенов

---

### Azure.Storage.Blobs.Specialized

- ✅ Операции с block blob (stage blocks, commit block list)
- ✅ Операции с append blob (append blocks)
- ✅ Операции с page blob (upload pages, clear pages)
- ✅ Lease-операции (acquire, release, renew)
- ✅ Работа с версионированием blob

---

### Azure.Storage.Blobs.Models

- ✅ Перечисления уровней доступа (`AccessTier`)
- ✅ Перечисления типов blob (`BlobType`)
- ✅ Настройка HTTP-заголовков
- ✅ Response-модели
- ✅ Условия (ETag, If-Match и другие фильтры)

---

## Best Practices (Лучшие практики)

### Package Usage (Использование пакетов)

✅ **DO:**

- Использовать `Azure.Storage.Blobs` в большинстве сценариев
- Подключать `Azure.Storage.Blobs.Specialized` только при необходимости
- Использовать версию 12.x для новых приложений
- Устанавливать `Azure.Identity` для аутентификации
- Настраивать `BlobClientOptions` (retry, diagnostics, transport)

❌ **DON'T:**

- Смешивать пакеты v11 и v12
- Устанавливать ненужные зависимости
- Использовать устаревшие версии SDK

---

### Client Lifecycle (Жизненный цикл клиентов)

✅ **DO:**

- Создавать клиентов один раз и переиспользовать
- Использовать singleton-паттерн для `BlobServiceClient`
- Освобождать ресурсы при необходимости (`IDisposable`)

❌ **DON'T:**

- Создавать новый клиент на каждую операцию
- Создавать клиентов внутри tight loop

> 💡 Рекомендуется регистрировать `BlobServiceClient` через Dependency Injection как Singleton.

---

## Exam Tips (Советы к экзамену AZ-204)

🎯 **Version 12.x** — рекомендована для новых приложений

🎯 **Три основных класса:**
- `BlobServiceClient` — уровень сервиса
- `BlobContainerClient` — уровень контейнера
- `BlobClient` — уровень blob

🎯 **Три пакета:**
- `Azure.Storage.Blobs` — основной
- `Specialized` — операции по типам blob
- `Models` — вспомогательные типы

🎯 **Иерархия:**  
Service → Container → Blob

🎯 **BlobClientOptions** — настройка retry, диагностики и транспорта

🎯 **Specialized-клиенты:**  
`BlockBlobClient`, `AppendBlobClient`, `PageBlobClient`

🎯 **Authentication:**  
Интеграция с `Azure.Identity` (`DefaultAzureCredential`)

🎯 **BlobUriBuilder:**  
Удобное построение и модификация URI

---

## Quick Reference

### Required Packages

```xml
<ItemGroup>
  <PackageReference Include="Azure.Storage.Blobs" Version="12.x" />
  <PackageReference Include="Azure.Identity" Version="1.x" />
</ItemGroup>
```

### Basic Client Creation

```csharp
// Service client
var serviceClient = new BlobServiceClient(uri, credential);

// Container client
var containerClient = new BlobContainerClient(uri, credential);

// Blob client
var blobClient = new BlobClient(uri, credential);
```

### Specialized Clients

```csharp
// Block blob
var blockBlob = new BlockBlobClient(uri, credential);

// Append blob
var appendBlob = new AppendBlobClient(uri, credential);

// Page blob
var pageBlob = new PageBlobClient(uri, credential);
```

---

## Additional Resources

- [Azure.Storage.Blobs API Reference](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs)
- [Azure.Storage.Blobs.Specialized API Reference](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.specialized)
- [Azure.Storage.Blobs.Models API Reference](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.models)
- [Azure Blob Storage client library for .NET](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-dotnet)

[Microsoft Learn - Explore Azure Blob storage client library](https://learn.microsoft.com/en-us/training/modules/work-azure-blob-storage/2-blob-storage-client-library-overview)
