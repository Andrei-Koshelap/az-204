# Create Blob Client Objects (Создание клиентских объектов Blob)

## Overview (Обзор)

Работа с Azure Blob Storage через SDK начинается с **создания клиентских объектов**.  
В этом разделе рассматриваются три основных типа клиентов:

1. **BlobServiceClient** — уровень Storage Account
2. **BlobContainerClient** — уровень контейнера
3. **BlobClient** — уровень конкретного blob

Иерархия выглядит так:

BlobServiceClient
↓
BlobContainerClient
↓
BlobClient

## Authentication Prerequisites

### DefaultAzureCredential

The examples in this unit use **DefaultAzureCredential** from the `Azure.Identity` package for authentication.

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;
```

### Authentication Process (Процесс аутентификации)

1. Получается **access token** для авторизации
2. Токен передаётся как credential при создании клиента
3. Credential используется на протяжении всего жизненного цикла клиента

> 💡 Токен автоматически обновляется SDK при использовании `DefaultAzureCredential`.

---

### Required RBAC Roles (Необходимые RBAC-роли)

Security principal (пользователь, приложение или Managed Identity), запрашивающий токен, должен иметь соответствующую **Azure RBAC роль**, предоставляющую доступ к данным blob.

| Role | Permissions | Use Case |
|-------|------------|----------|
| **Storage Blob Data Owner** | Полный доступ (read, write, delete, управление ACL) | Администрирование |
| **Storage Blob Data Contributor** | Чтение, запись, удаление blob | Доступ приложения на запись |
| **Storage Blob Data Reader** | Чтение и просмотр списка blob | Read-only доступ |

---

> 🎯 Экзаменационный момент AZ-204:  
> Для работы с blob через Azure AD необходимо назначить одну из ролей  
> `Storage Blob Data *`, иначе запросы к данным будут отклонены.

### Assign RBAC Role

```bash
# Azure CLI - Assign role to current user
az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee <user-email-or-object-id> \
    --scope /subscriptions/<subscription-id>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>

# Assign to service principal
az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee <service-principal-id> \
    --scope /subscriptions/<subscription-id>/resourceGroups/<rg>/providers/Microsoft.Storage/storageAccounts/<account>
```

---

## 1. Create BlobServiceClient Object
(Создание объекта BlobServiceClient)

### Purpose (Назначение)

`BlobServiceClient` позволяет приложению работать с ресурсами на уровне **Storage Account**.

Это точка входа для всех операций с Blob Storage.

---

### Capabilities (Возможности)

- ✅ Получение и настройка свойств аккаунта
- ✅ Получение списка контейнеров
- ✅ Создание контейнеров
- ✅ Удаление контейнеров
- ✅ Получение статистики аккаунта

---

### Пример создания клиента

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;

public BlobServiceClient GetBlobServiceClient(string accountName)
{
    BlobServiceClient client = new(
        new Uri($"https://{accountName}.blob.core.windows.net"),
        new DefaultAzureCredential());

    return client;
}
```

### URI Format

```
https://<account-name>.blob.core.windows.net
```

**Example:**
```
https://mystorageaccount.blob.core.windows.net
```

### With Client Options

```csharp
public BlobServiceClient GetBlobServiceClientWithOptions(string accountName)
{
    var options = new BlobClientOptions
    {
        Retry =
        {
            MaxRetries = 5,
            Delay = TimeSpan.FromSeconds(2)
        }
    };

    var client = new BlobServiceClient(
        new Uri($"https://{accountName}.blob.core.windows.net"),
        new DefaultAzureCredential(),
        options);

    return client;
}
```

### Common Operations

```csharp
// List all containers
await foreach (var container in serviceClient.GetBlobContainersAsync())
{
    Console.WriteLine($"Container: {container.Name}");
}

// Create a container
BlobContainerClient newContainer = await serviceClient.CreateBlobContainerAsync("mycontainer");

// Delete a container
await serviceClient.DeleteBlobContainerAsync("mycontainer");

// Get account info
var accountInfo = await serviceClient.GetAccountInfoAsync();
Console.WriteLine($"Account Kind: {accountInfo.Value.AccountKind}");
```

---

## 2. Create BlobContainerClient Object
(Создание объекта BlobContainerClient)

### Purpose (Назначение)

`BlobContainerClient` используется для работы с **конкретным контейнером** в Storage Account.

Через него выполняются операции на уровне контейнера и его blob.

---

### Capabilities (Возможности)

- ✅ Создание контейнера
- ✅ Удаление контейнера
- ✅ Настройка свойств контейнера
- ✅ Получение списка blob в контейнере
- ✅ Загрузка blob
- ✅ Удаление blob

---

### Пример создания через BlobServiceClient

```csharp
public BlobContainerClient GetBlobContainerClient(
    BlobServiceClient blobServiceClient,
    string containerName)
{
    // Create the container client using the service client object
    BlobContainerClient client = blobServiceClient.GetBlobContainerClient(containerName);
    return client;
}
```

**Advantages:**
- ✅ Leverages existing service client
- ✅ Shares configuration and credentials
- ✅ Cleaner hierarchy

### Method 2: Create Directly

If your work is narrowly scoped to a single container, you can create a `BlobContainerClient` directly:

```csharp
public BlobContainerClient GetBlobContainerClient(
    string accountName,
    string containerName,
    BlobClientOptions clientOptions)
{
    // Append the container name to the end of the URI
    BlobContainerClient client = new(
        new Uri($"https://{accountName}.blob.core.windows.net/{containerName}"),
        new DefaultAzureCredential(),
        clientOptions);

    return client;
}
```

### URI Format

```
https://<account-name>.blob.core.windows.net/<container-name>
```

**Example:**
```
https://mystorageaccount.blob.core.windows.net/mycontainer
```

### Common Operations

```csharp
// Create container if it doesn't exist
await containerClient.CreateIfNotExistsAsync();

// Upload a blob
await containerClient.UploadBlobAsync("file.txt", File.OpenRead("./file.txt"));

// List blobs
await foreach (var blobItem in containerClient.GetBlobsAsync())
{
    Console.WriteLine($"Blob: {blobItem.Name}");
}

// Delete blob
await containerClient.DeleteBlobAsync("file.txt");

// Get container properties
var properties = await containerClient.GetPropertiesAsync();
Console.WriteLine($"Last Modified: {properties.Value.LastModified}");
```

---

## 3. Create BlobClient Object
(Создание объекта BlobClient)

### Purpose (Назначение)

`BlobClient` используется для работы с **конкретным blob** внутри контейнера.

Это уровень, на котором выполняются операции над самим файлом.

---

### Capabilities (Возможности)

- ✅ Загрузка blob
- ✅ Скачивание blob
- ✅ Удаление blob
- ✅ Копирование blob
- ✅ Получение и изменение свойств
- ✅ Получение и изменение метаданных
- ✅ Управление access tier

---

### Создание через BlobContainerClient Create 

### Method 1: from Service/Container Client (Recommended)


```csharp
public BlobClient GetBlobClient(
    BlobServiceClient blobServiceClient,
    string containerName,
    string blobName)
{
    BlobClient client =
        blobServiceClient.GetBlobContainerClient(containerName).GetBlobClient(blobName);
    return client;
}
```

### Method 2: Create from Container Client

```csharp
public BlobClient GetBlobClientFromContainer(
    BlobContainerClient containerClient,
    string blobName)
{
    BlobClient client = containerClient.GetBlobClient(blobName);
    return client;
}
```

### Method 3: Create Directly

```csharp
public BlobClient GetBlobClientDirect(
    string accountName,
    string containerName,
    string blobName)
{
    var client = new BlobClient(
        new Uri($"https://{accountName}.blob.core.windows.net/{containerName}/{blobName}"),
        new DefaultAzureCredential());
    
    return client;
}
```

### URI Format

```
https://<account-name>.blob.core.windows.net/<container-name>/<blob-name>
```

**Example:**
```
https://mystorageaccount.blob.core.windows.net/mycontainer/myfile.txt
```

### Common Operations

```csharp
// Upload file
await blobClient.UploadAsync("./localfile.txt", overwrite: true);

// Download to file
await blobClient.DownloadToAsync("./downloadedfile.txt");

// Download to stream
using var stream = new MemoryStream();
await blobClient.DownloadToAsync(stream);

// Check if blob exists
bool exists = await blobClient.ExistsAsync();

// Get blob properties
var properties = await blobClient.GetPropertiesAsync();
Console.WriteLine($"Content Type: {properties.Value.ContentType}");
Console.WriteLine($"Size: {properties.Value.ContentLength} bytes");

// Set access tier
await blobClient.SetAccessTierAsync(AccessTier.Cool);

// Delete blob
await blobClient.DeleteAsync();
```

---

## Client Creation Patterns

### Pattern 1: Top-Down Navigation (Recommended)

```csharp
// Start at service level
var serviceClient = new BlobServiceClient(
    new Uri("https://account.blob.core.windows.net"),
    new DefaultAzureCredential());

// Navigate to container
var containerClient = serviceClient.GetBlobContainerClient("mycontainer");

// Navigate to blob
var blobClient = containerClient.GetBlobClient("myfile.txt");
```
### Advantages (Преимущества иерархического подхода)

- ✅ Самая понятная и логичная иерархия (Service → Container → Blob)
- ✅ Общая конфигурация (credential, retry, diagnostics) используется на всех уровнях
- ✅ Удобная навигация между уровнями клиентов

> 💡 Рекомендуемый подход — создавать `BlobServiceClient` один раз  
> и получать из него `BlobContainerClient`, а затем `BlobClient`.


### Pattern 2: Direct Client Creation

```csharp
// Create container client directly
var containerClient = new BlobContainerClient(
    new Uri("https://account.blob.core.windows.net/mycontainer"),
    new DefaultAzureCredential());

// Create blob client directly
var blobClient = new BlobClient(
    new Uri("https://account.blob.core.windows.net/mycontainer/myfile.txt"),
    new DefaultAzureCredential());
```

### Use When (Когда использовать прямое создание клиента)

- ✅ Работа ограничена конкретным контейнером или blob
- ✅ Не требуются операции на уровне Storage Account
- ✅ Есть конкретный URI ресурса

> 💡 Прямое создание `BlobContainerClient` или `BlobClient` удобно,
> если приложение работает только с одним контейнером
> или получает полный URI blob извне (например, из БД).

### Pattern 3: Using Connection String

```csharp
string connectionString = "DefaultEndpointsProtocol=https;AccountName=...";

// Service client
var serviceClient = new BlobServiceClient(connectionString);

// Container client
var containerClient = new BlobContainerClient(connectionString, "mycontainer");

// Blob client
var blobClient = new BlobClient(connectionString, "mycontainer", "myfile.txt");
```

⚠️ **Note:** Connection strings contain sensitive credentials. Use DefaultAzureCredential in production when possible.

---

## Complete Example

### Full Workflow with All Three Clients

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

public class BlobStorageExample
{
    private readonly string accountName = "mystorageaccount";
    
    public async Task CompleteWorkflowAsync()
    {
        // 1. Create service client (account level)
        var serviceClient = new BlobServiceClient(
            new Uri($"https://{accountName}.blob.core.windows.net"),
            new DefaultAzureCredential());
        
        Console.WriteLine("Created BlobServiceClient");
        
        // 2. List existing containers
        Console.WriteLine("\nExisting containers:");
        await foreach (var container in serviceClient.GetBlobContainersAsync())
        {
            Console.WriteLine($"  - {container.Name}");
        }
        
        // 3. Create container client
        var containerName = $"demo-{Guid.NewGuid()}";
        var containerClient = serviceClient.GetBlobContainerClient(containerName);
        
        Console.WriteLine($"\nCreating container: {containerName}");
        await containerClient.CreateAsync();
        
        // 4. Create blob client
        var blobName = "sample.txt";
        var blobClient = containerClient.GetBlobClient(blobName);
        
        // 5. Upload data
        Console.WriteLine($"\nUploading blob: {blobName}");
        await blobClient.UploadAsync(
            BinaryData.FromString("Hello, Azure Blob Storage!"),
            overwrite: true);
        
        // 6. Verify blob exists
        bool exists = await blobClient.ExistsAsync();
        Console.WriteLine($"Blob exists: {exists}");
        
        // 7. Download blob
        Console.WriteLine("\nDownloading blob...");
        var download = await blobClient.DownloadContentAsync();
        Console.WriteLine($"Content: {download.Value.Content}");
        
        // 8. Cleanup
        Console.WriteLine("\nCleaning up...");
        await containerClient.DeleteAsync();
        Console.WriteLine("Container deleted");
    }
}
```

---

## URI Construction with BlobUriBuilder

### Manual URI Construction

```csharp
// Manually construct URI
var uri = new Uri($"https://{accountName}.blob.core.windows.net/{containerName}/{blobName}");
```

### Using BlobUriBuilder

```csharp
var builder = new BlobUriBuilder(new Uri($"https://{accountName}.blob.core.windows.net"))
{
    BlobContainerName = containerName,
    BlobName = blobName
};

Uri blobUri = builder.ToUri();
var blobClient = new BlobClient(blobUri, new DefaultAzureCredential());
```
### Advantages (Преимущества использования BlobUriBuilder)

- ✅ Типобезопасное построение URI
- ✅ Простое изменение компонентов URI (account, container, blob)
- ✅ Корректная обработка URL-encoding

---

## Best Practices (Лучшие практики)

### Client Lifecycle (Жизненный цикл клиентов)

✅ **DO:**

- Создавать клиентов один раз и переиспользовать (singleton)
- Использовать `BlobServiceClient` для операций уровня аккаунта
- Навигироваться Service → Container → Blob
- Использовать клиент, соответствующий области задачи
- Разделять credential и client options между клиентами

❌ **DON'T:**

- Создавать новый клиент на каждую операцию
- Создавать клиентов внутри циклов
- Без необходимости смешивать connection string и credential-объекты

---

### Authentication (Аутентификация)

✅ **DO:**

- Использовать `DefaultAzureCredential` в production
- Назначать корректные RBAC роли
- Использовать Managed Identity в Azure
- Хранить connection strings в безопасном месте (например, Key Vault)

❌ **DON'T:**

- Хардкодить connection strings в коде
- Использовать connection strings при доступной Managed Identity
- Назначать избыточные RBAC-права

---

### URI Construction (Построение URI)

✅ **DO:**

- Использовать `BlobUriBuilder` для сложных URI
- Проверять корректность URI
- Добавлять обработку ошибок

❌ **DON'T:**

- Склеивать строки URI вручную без encoding
- Забывать кодировать специальные символы в имени blob

---

## Exam Tips (Советы к экзамену AZ-204)

🎯 **Три типа клиентов:**  
`BlobServiceClient` (account),  
`BlobContainerClient` (container),  
`BlobClient` (blob)

🎯 **DefaultAzureCredential** — рекомендуемый способ аутентификации

🎯 **RBAC роли:**  
`Storage Blob Data Owner / Contributor / Reader`

🎯 **Иерархия клиентов:**  
Service → Container → Blob

🎯 **Формат URI:**  
`https://{account}.blob.core.windows.net/{container}/{blob}`

🎯 **Повторное использование клиентов:**  
Создать один раз, использовать многократно

🎯 **Навигационные методы:**  
`GetBlobContainerClient()` — переход к контейнеру  
`GetBlobClient()` — переход к blob

🎯 **BlobUriBuilder** — удобное построение и модификация URI

🎯 **Прямое создание:**  
Можно создавать container/blob client напрямую без `BlobServiceClient`

---

## Quick Reference Commands

### Create Service Client

```csharp
var serviceClient = new BlobServiceClient(
    new Uri($"https://{accountName}.blob.core.windows.net"),
    new DefaultAzureCredential());
```

### Create Container Client (from service)

```csharp
var containerClient = serviceClient.GetBlobContainerClient(containerName);
```

### Create Blob Client (from container)

```csharp
var blobClient = containerClient.GetBlobClient(blobName);
```

### Create Blob Client (from service, one line)

```csharp
var blobClient = serviceClient
    .GetBlobContainerClient(containerName)
    .GetBlobClient(blobName);
```

---

## Additional Resources

- [BlobServiceClient Class](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobserviceclient)
- [BlobContainerClient Class](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobcontainerclient)
- [BlobClient Class](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobclient)
- [DefaultAzureCredential Class](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredential)

[Microsoft Learn - Create a client object](https://learn.microsoft.com/en-us/training/modules/work-azure-blob-storage/3-create-client-object)
