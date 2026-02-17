# Manage Container Properties and Metadata Using .NET
(Управление свойствами и метаданными контейнера через .NET)

## Overview (Обзор)

Контейнеры Azure Blob Storage имеют:

- **System properties** (системные свойства)
- **User-defined metadata** (пользовательские метаданные)

В этом разделе рассматривается управление обоими типами с помощью .NET SDK (Azure.Storage.Blobs 12.x).

---

## System Properties vs User-Defined Metadata
(Системные свойства vs пользовательские метаданные)

| Aspect | System Properties | User-Defined Metadata |
|---------|------------------|------------------------|
| **Definition** | Свойства, управляемые Azure | Пользовательские пары ключ-значение |
| **Examples** | ETag, Last-Modified, Lease Status | Department, Project, CostCenter |
| **Modification** | В основном только для чтения | Полностью изменяемы |
| **Size Limit** | Не применяется | До 8 KB на ресурс |
| **Name Format** | Фиксированные имена | Должны быть допустимыми C#-идентификаторами |
| **Purpose** | Состояние и техническая информация | Организация и категоризация данных |

---

## Важно помнить

- Системные свойства отражают текущее состояние ресурса (например, `ETag`, `LastModified`).
- Метаданные используются для бизнес-логики (например, распределение по проектам).
- При обновлении metadata предыдущие значения **перезаписываются полностью**.

> 🎯 Экзаменационный момент AZ-204:  
> Метаданные — это key-value пары до 8 KB на ресурс,  
> обновление metadata заменяет весь набор, а не добавляет частично.

## Container System Properties

### Read-Only Properties

```csharp
BlobContainerProperties properties = await containerClient.GetPropertiesAsync();

// System properties
DateTimeOffset lastModified = properties.LastModified;
string eTag = properties.ETag.ToString();
LeaseStatus leaseStatus = properties.LeaseStatus;
LeaseState leaseState = properties.LeaseState;
PublicAccessType publicAccess = properties.PublicAccess;
bool hasImmutabilityPolicy = properties.HasImmutabilityPolicy;
bool hasLegalHold = properties.HasLegalHold;
```

### System Properties Reference (Справочник системных свойств)

| Property | Type | Description |
|------------|----------------|-------------|
| **LastModified** | DateTimeOffset | Время последнего изменения (UTC) |
| **ETag** | ETag | Entity tag для контроля конкуренции |
| **LeaseStatus** | LeaseStatus | Locked, Unlocked |
| **LeaseState** | LeaseState | Available, Leased, Expired, Breaking, Broken |
| **LeaseDuration** | LeaseDuration | Infinite, Fixed |
| **PublicAccess** | PublicAccessType | None, Blob, Container |
| **HasImmutabilityPolicy** | bool | Наличие политики неизменяемости |
| **HasLegalHold** | bool | Наличие юридической блокировки |

---

## User-Defined Metadata (Пользовательские метаданные)

Метаданные — это пары **ключ-значение**, которые позволяют логически группировать и классифицировать контейнеры и blob.

### Metadata Naming Rules (Правила именования)

✅ **Допустимые имена метаданных:**

- Должны быть корректными C#-идентификаторами
- Не чувствительны к регистру (Azure хранит в lowercase)
- Только буквы, цифры и символ `_`
- Не могут начинаться с цифры

---

### Дополнительные ограничения

- Общий размер metadata на ресурс — до **8 KB**
- При обновлении metadata предыдущий набор полностью заменяется
- Метаданные не индексируются автоматически (в отличие от blob index tags)

---

> 🎯 Экзаменационный момент AZ-204:  
> Metadata — это key-value пары до 8 KB,  
> регистр не имеет значения,  
> при обновлении происходит полная перезапись.


❌ **Invalid metadata names:**
```
metadata["Cost-Center"]    // Hyphen not allowed
metadata["2ndProject"]     // Starts with number
metadata["Project Name"]   // Space not allowed
```

✅ **Valid examples:**
```
metadata["CostCenter"]
metadata["Project_Name"]
metadata["Department"]
```

### Metadata Size Limits

- **Maximum size**: 8 KB total (name + value pairs combined)
- **Includes**: HTTP header overhead
- **Best practice**: Keep names short and values concise

---

## Retrieve Container Properties and Metadata

### Get All Properties and Metadata

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

BlobContainerClient containerClient = serviceClient.GetBlobContainerClient("mycontainer");

// Retrieve properties and metadata in one call
BlobContainerProperties properties = await containerClient.GetPropertiesAsync();

// Access system properties
Console.WriteLine($"Last Modified: {properties.LastModified}");
Console.WriteLine($"ETag: {properties.ETag}");
Console.WriteLine($"Lease Status: {properties.LeaseStatus}");

// Access metadata
if (properties.Metadata.Count > 0)
{
    Console.WriteLine("\nMetadata:");
    foreach (var metadataItem in properties.Metadata)
    {
        Console.WriteLine($"  {metadataItem.Key}: {metadataItem.Value}");
    }
}
else
{
    Console.WriteLine("\nNo metadata found");
}
```

### Metadata Structure

```csharp
// Metadata is IDictionary<string, string>
IDictionary<string, string> metadata = properties.Metadata;

// Access specific metadata
if (metadata.ContainsKey("department"))
{
    string department = metadata["department"];
    Console.WriteLine($"Department: {department}");
}

// Iterate all metadata
foreach (KeyValuePair<string, string> item in metadata)
{
    Console.WriteLine($"{item.Key} = {item.Value}");
}
```

---

## Set Container Metadata

### Add or Update Metadata

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

BlobContainerClient containerClient = serviceClient.GetBlobContainerClient("mycontainer");

// Create metadata dictionary
IDictionary<string, string> metadata = new Dictionary<string, string>
{
    { "department", "engineering" },
    { "project", "az204-training" },
    { "costcenter", "12345" },
    { "environment", "production" }
};

// Set metadata on container
await containerClient.SetMetadataAsync(metadata);

Console.WriteLine("Metadata set successfully");
```

### Update Existing Metadata

```csharp
// Retrieve current properties (includes metadata)
BlobContainerProperties properties = await containerClient.GetPropertiesAsync();

// Modify metadata
IDictionary<string, string> metadata = properties.Metadata;
metadata["lastreviewed"] = DateTime.UtcNow.ToString("yyyy-MM-dd");
metadata["reviewedby"] = "admin@contoso.com";

// Update metadata (replaces all metadata)
await containerClient.SetMetadataAsync(metadata);
```

⚠️ **Important**: `SetMetadataAsync` **replaces all** metadata. Always retrieve current metadata first if you want to preserve existing values.

---

## Complete Container Metadata Example

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

namespace ContainerMetadataExample
{
    class Program
    {
        static async Task Main(string[] args)
        {
            string storageAccountName = "mystorageaccount";
            string containerName = "mycontainer";

            // Create BlobServiceClient
            var serviceClient = new BlobServiceClient(
                new Uri($"https://{storageAccountName}.blob.core.windows.net"),
                new DefaultAzureCredential());

            // Get container client
            BlobContainerClient containerClient = serviceClient.GetBlobContainerClient(containerName);

            // Ensure container exists
            await containerClient.CreateIfNotExistsAsync();

            // === SET METADATA ===
            Console.WriteLine("Setting container metadata...");
            
            IDictionary<string, string> metadata = new Dictionary<string, string>
            {
                { "department", "engineering" },
                { "project", "blobstorage" },
                { "costcenter", "12345" },
                { "owner", "alice@contoso.com" },
                { "createdate", DateTime.UtcNow.ToString("yyyy-MM-dd") }
            };

            await containerClient.SetMetadataAsync(metadata);
            Console.WriteLine("✓ Metadata set\n");

            // === RETRIEVE PROPERTIES AND METADATA ===
            Console.WriteLine("Retrieving container properties and metadata...");
            
            BlobContainerProperties properties = await containerClient.GetPropertiesAsync();

            // Display system properties
            Console.WriteLine("System Properties:");
            Console.WriteLine($"  Last Modified: {properties.LastModified}");
            Console.WriteLine($"  ETag: {properties.ETag}");
            Console.WriteLine($"  Lease Status: {properties.LeaseStatus}");
            Console.WriteLine($"  Lease State: {properties.LeaseState}");
            Console.WriteLine($"  Public Access: {properties.PublicAccess}");
            Console.WriteLine();

            // Display metadata
            Console.WriteLine("User-Defined Metadata:");
            foreach (var item in properties.Metadata)
            {
                Console.WriteLine($"  {item.Key}: {item.Value}");
            }
            Console.WriteLine();

            // === UPDATE METADATA ===
            Console.WriteLine("Updating metadata...");
            
            IDictionary<string, string> updatedMetadata = properties.Metadata;
            updatedMetadata["lastmodified"] = DateTime.UtcNow.ToString("yyyy-MM-dd HH:mm:ss");
            updatedMetadata["modifiedby"] = "bob@contoso.com";

            await containerClient.SetMetadataAsync(updatedMetadata);
            Console.WriteLine("✓ Metadata updated\n");

            // === VERIFY UPDATE ===
            Console.WriteLine("Verifying metadata update...");
            
            properties = await containerClient.GetPropertiesAsync();
            
            Console.WriteLine("Updated Metadata:");
            foreach (var item in properties.Metadata)
            {
                Console.WriteLine($"  {item.Key}: {item.Value}");
            }
        }
    }
}
```

### Expected Output

```
Setting container metadata...
✓ Metadata set

Retrieving container properties and metadata...
System Properties:
  Last Modified: 1/3/2026 10:30:00 AM +00:00
  ETag: "0x8DCB123456789AB"
  Lease Status: Unlocked
  Lease State: Available
  Public Access: None

User-Defined Metadata:
  department: engineering
  project: blobstorage
  costcenter: 12345
  owner: alice@contoso.com
  createdate: 2026-01-03

Updating metadata...
✓ Metadata updated

Verifying metadata update...
Updated Metadata:
  department: engineering
  project: blobstorage
  costcenter: 12345
  owner: alice@contoso.com
  createdate: 2026-01-03
  lastmodified: 2026-01-03 10:30:15
  modifiedby: bob@contoso.com
```

---

## Manage Blob Properties and Metadata

### Blob System Properties

```csharp
BlobClient blobClient = containerClient.GetBlobClient("myblob.txt");

// Get properties
BlobProperties blobProperties = await blobClient.GetPropertiesAsync();

// System properties
string contentType = blobProperties.ContentType;
long contentLength = blobProperties.ContentLength;
DateTimeOffset lastModified = blobProperties.LastModified;
string eTag = blobProperties.ETag.ToString();
BlobType blobType = blobProperties.BlobType;
string contentEncoding = blobProperties.ContentEncoding;
string cacheControl = blobProperties.CacheControl;
```

### Set Blob HTTP Headers

```csharp
// Create HTTP headers
var headers = new BlobHttpHeaders
{
    ContentType = "text/plain",
    ContentLanguage = "en-US",
    ContentEncoding = "utf-8",
    CacheControl = "max-age=3600",
    ContentDisposition = "attachment; filename=myfile.txt"
};

// Set headers on blob
await blobClient.SetHttpHeadersAsync(headers);

Console.WriteLine("Blob HTTP headers updated");
```

### Set Blob Metadata

```csharp
BlobClient blobClient = containerClient.GetBlobClient("myblob.txt");

// Create metadata
var blobMetadata = new Dictionary<string, string>
{
    { "author", "Alice Smith" },
    { "version", "1.2" },
    { "classification", "public" },
    { "uploadedon", DateTime.UtcNow.ToString("o") }
};

// Set metadata
await blobClient.SetMetadataAsync(blobMetadata);

Console.WriteLine("Blob metadata set");
```

### Retrieve Blob Metadata

```csharp
// Get properties (includes metadata)
BlobProperties blobProperties = await blobClient.GetPropertiesAsync();

// Access metadata
Console.WriteLine("Blob Metadata:");
foreach (var item in blobProperties.Metadata)
{
    Console.WriteLine($"  {item.Key}: {item.Value}");
}
```

---

## Complete Blob Metadata Example

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

namespace BlobMetadataExample
{
    class Program
    {
        static async Task Main(string[] args)
        {
            string storageAccountName = "mystorageaccount";
            string containerName = "mycontainer";
            string blobName = "sample.txt";

            // Create clients
            var serviceClient = new BlobServiceClient(
                new Uri($"https://{storageAccountName}.blob.core.windows.net"),
                new DefaultAzureCredential());

            BlobContainerClient containerClient = serviceClient.GetBlobContainerClient(containerName);
            await containerClient.CreateIfNotExistsAsync();

            BlobClient blobClient = containerClient.GetBlobClient(blobName);

            // Upload sample blob
            using (var stream = new MemoryStream(System.Text.Encoding.UTF8.GetBytes("Hello, Blob!")))
            {
                await blobClient.UploadAsync(stream, overwrite: true);
            }

            // === SET BLOB HTTP HEADERS ===
            Console.WriteLine("Setting blob HTTP headers...");
            
            var headers = new BlobHttpHeaders
            {
                ContentType = "text/plain",
                ContentLanguage = "en-US",
                ContentEncoding = "utf-8",
                CacheControl = "max-age=3600"
            };

            await blobClient.SetHttpHeadersAsync(headers);
            Console.WriteLine("✓ HTTP headers set\n");

            // === SET BLOB METADATA ===
            Console.WriteLine("Setting blob metadata...");
            
            var metadata = new Dictionary<string, string>
            {
                { "author", "Alice" },
                { "version", "1.0" },
                { "category", "documentation" },
                { "createdate", DateTime.UtcNow.ToString("yyyy-MM-dd") }
            };

            await blobClient.SetMetadataAsync(metadata);
            Console.WriteLine("✓ Metadata set\n");

            // === RETRIEVE PROPERTIES ===
            Console.WriteLine("Retrieving blob properties...");
            
            BlobProperties properties = await blobClient.GetPropertiesAsync();

            Console.WriteLine("HTTP Headers:");
            Console.WriteLine($"  Content-Type: {properties.ContentType}");
            Console.WriteLine($"  Content-Length: {properties.ContentLength}");
            Console.WriteLine($"  Content-Encoding: {properties.ContentEncoding}");
            Console.WriteLine($"  Cache-Control: {properties.CacheControl}");
            Console.WriteLine();

            Console.WriteLine("System Properties:");
            Console.WriteLine($"  ETag: {properties.ETag}");
            Console.WriteLine($"  Last Modified: {properties.LastModified}");
            Console.WriteLine($"  Blob Type: {properties.BlobType}");
            Console.WriteLine();

            Console.WriteLine("Metadata:");
            foreach (var item in properties.Metadata)
            {
                Console.WriteLine($"  {item.Key}: {item.Value}");
            }
        }
    }
}
```

---

## Key Methods Summary (Ключевые методы)

### Container Methods (Методы контейнера)

| Method | Purpose | Returns |
|----------|----------|----------|
| **GetPropertiesAsync()** | Получить свойства и metadata контейнера | BlobContainerProperties |
| **SetMetadataAsync()** | Установить / полностью заменить metadata | Response |
| **CreateIfNotExistsAsync()** | Создать контейнер, если отсутствует | BlobContainerClient |

> 💡 Важно: `SetMetadataAsync()` заменяет весь набор metadata, а не добавляет частично.

---

### Blob Methods (Методы blob)

| Method | Purpose | Returns |
|----------|----------|----------|
| **GetPropertiesAsync()** | Получить свойства, HTTP-заголовки и metadata | BlobProperties |
| **SetMetadataAsync()** | Установить / заменить metadata | Response |
| **SetHttpHeadersAsync()** | Обновить HTTP-заголовки | Response |
| **UploadAsync()** | Загрузить blob (можно указать metadata) | Response |

---

### Практические замечания

- `GetPropertiesAsync()` возвращает объект со всеми системными свойствами
- `SetHttpHeadersAsync()` используется для изменения `ContentType`, `ContentEncoding` и других HTTP-заголовков
- `UploadAsync()` позволяет задать metadata при загрузке

---

> 🎯 Экзаменационный момент AZ-204:  
> Обновление metadata полностью заменяет предыдущий набор.  
> Для получения ETag и LastModified используется `GetPropertiesAsync()`.


## Best Practices

### 1. Efficient Metadata Retrieval

✅ **Do**: Get properties and metadata in one call
```csharp
BlobContainerProperties props = await containerClient.GetPropertiesAsync();
// Access both props and props.Metadata
```

❌ **Don't**: Make separate calls unnecessarily
```csharp
// Less efficient
var props = await containerClient.GetPropertiesAsync();
var metadata = await containerClient.GetPropertiesAsync().Metadata; // Redundant
```

### 2. Metadata Naming

✅ **Do**: Use clear, descriptive names
```csharp
metadata["department"] = "engineering";
metadata["cost_center"] = "CC-12345";
metadata["project_code"] = "AZ204";
```

❌ **Don't**: Use cryptic abbreviations
```csharp
metadata["dept"] = "eng";  // Less clear
metadata["cc"] = "12345";  // Ambiguous
```

### 3. Preserve Existing Metadata

✅ **Do**: Retrieve before updating
```csharp
var props = await containerClient.GetPropertiesAsync();
var metadata = props.Metadata;
metadata["newkey"] = "newvalue";  // Add to existing
await containerClient.SetMetadataAsync(metadata);
```

❌ **Don't**: Overwrite without retrieving
```csharp
var newMetadata = new Dictionary<string, string> { { "newkey", "value" } };
await containerClient.SetMetadataAsync(newMetadata); // Loses existing metadata!
```

### 4. Handle Case-Insensitivity

```csharp
// Azure stores metadata keys as lowercase
metadata["Department"] = "Engineering";

// Later retrieval (case doesn't matter)
var dept = props.Metadata["department"];  // Works
var dept2 = props.Metadata["DEPARTMENT"]; // Also works
```

### 5. Size Management

```csharp
// Calculate approximate metadata size
int totalSize = 0;
foreach (var item in metadata)
{
    totalSize += item.Key.Length + item.Value.Length;
}

if (totalSize > 7000) // Leave buffer for headers
{
    Console.WriteLine("Warning: Metadata approaching 8 KB limit");
}
```

---

## Error Handling

```csharp
try
{
    await containerClient.SetMetadataAsync(metadata);
}
catch (Azure.RequestFailedException ex) when (ex.Status == 400)
{
    Console.WriteLine("Invalid metadata: " + ex.Message);
    // Likely invalid metadata name
}
catch (Azure.RequestFailedException ex) when (ex.Status == 404)
{
    Console.WriteLine("Container not found");
}
catch (Azure.RequestFailedException ex) when (ex.Status == 403)
{
    Console.WriteLine("Access denied - check RBAC roles");
}
```

---

## Exam Tips (Советы к экзамену AZ-204)

🎯 **GetPropertiesAsync**  
Получает системные свойства И metadata за один вызов

🎯 **SetMetadataAsync**  
ПОЛНОСТЬЮ заменяет metadata (это не merge-операция)

🎯 **Правила именования metadata**  
Должны быть допустимыми C#-идентификаторами  
(без дефисов, пробелов и специальных символов)

🎯 **Case-insensitive**  
Azure хранит ключи metadata в lowercase

🎯 **Лимит 8 KB**  
Максимальный размер metadata (имена + значения вместе)

🎯 **IDictionary<string, string>**  
Тип коллекции metadata в .NET SDK

🎯 **System properties**  
Read-only (ETag, LastModified, LeaseStatus и др.)

🎯 **BlobHttpHeaders**  
Используется с `SetHttpHeadersAsync()` для изменения  
Content-Type, Cache-Control и других HTTP-заголовков

🎯 **Сохранение metadata**  
Перед обновлением получите текущие metadata,  
чтобы не потерять существующие значения

🎯 **Аутентификация**  
Для операций записи требуется роль  
`Storage Blob Data Contributor`

---

## Additional Resources

- [Container properties and metadata (.NET)](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-container-properties-metadata)
- [Blob properties and metadata (.NET)](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-blob-properties-metadata)
- [BlobContainerClient.GetPropertiesAsync](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobcontainerclient.getpropertiesasync)
- [BlobContainerClient.SetMetadataAsync](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobcontainerclient.setmetadataasync)

[Microsoft Learn - Manage container properties and metadata by using .NET](https://learn.microsoft.com/en-us/training/modules/work-azure-blob-storage/5-manage-container-properties-metadata-dotnet)
