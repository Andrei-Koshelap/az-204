# Exercise: Create Blob Storage Resources Using .NET Client Library
(Упражнение: создание ресурсов Blob Storage с помощью .NET Client Library)

## Exercise Overview (Обзор упражнения)

В этом упражнении вы создадите Azure Storage Account и напишете .NET консольное приложение, которое:

- Создаёт blob-контейнер
- Загружает файл в Blob Storage
- Выводит список blob в контейнере
- Скачивает blob в локальный файл

**Оценочное время:** 30 минут

---

## Prerequisites (Предварительные требования)

- Подписка Azure
- Azure Cloud Shell или локальная среда разработки
- .NET SDK 6.0 или новее

> 💡 От себя:  
> Рекомендуется установить пакеты `Azure.Storage.Blobs` и `Azure.Identity`,  
> а также использовать `DefaultAzureCredential` (особенно если запускаете код в Azure Cloud Shell или в Azure-hosted среде с Managed Identity).

---

## Step 1: Prepare Azure Resources

### Create Storage Account

```bash
# Set variables
RESOURCE_GROUP="az204-blob-rg"
LOCATION="eastus"
STORAGE_ACCOUNT="staz204$(openssl rand -hex 5)"

# Create resource group
az group create \
    --name $RESOURCE_GROUP \
    --location $LOCATION

# Create storage account
az storage account create \
    --name $STORAGE_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --location $LOCATION \
    --sku Standard_LRS \
    --kind StorageV2

# Display account name for later use
echo "Storage Account: $STORAGE_ACCOUNT"
```

### Assign RBAC Role

Assign the **Storage Blob Data Contributor** role to yourself:

```bash
# Get your user object ID
USER_ID=$(az ad signed-in-user show --query id -o tsv)

# Get storage account resource ID
STORAGE_ID=$(az storage account show \
    --name $STORAGE_ACCOUNT \
    --resource-group $RESOURCE_GROUP \
    --query id -o tsv)

# Assign role
az role assignment create \
    --role "Storage Blob Data Contributor" \
    --assignee $USER_ID \
    --scope $STORAGE_ID

echo "Role assigned. Waiting 60 seconds for propagation..."
sleep 60
```

---

## Step 2: Create .NET Console Application

### Initialize Project

```bash
# Create project directory
mkdir ~/BlobStorage
cd ~/BlobStorage

# Create new console app
dotnet new console -n BlobStorageApp
cd BlobStorageApp

# Add required packages
dotnet add package Azure.Storage.Blobs
dotnet add package Azure.Identity
```

### Verify Package Installation

```bash
# Check installed packages
dotnet list package
```

Expected output:
```
Project 'BlobStorageApp' has the following package references
   [net6.0]:
   Top-level Package      Requested   Resolved
   > Azure.Identity       1.x.x       1.x.x
   > Azure.Storage.Blobs  12.x.x      12.x.x
```

---

## Step 3: Write the Application Code

### Complete Program.cs

Replace the contents of `Program.cs` with the following code:

```csharp
using Azure.Identity;
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Models;

namespace BlobStorageApp
{
    class Program
    {
        static async Task Main(string[] args)
        {
            Console.WriteLine("Azure Blob Storage Exercise\n");

            // TODO: Update with your storage account name
            string storageAccountName = "REPLACE_WITH_YOUR_STORAGE_ACCOUNT_NAME";
            string containerName = $"demo-container-{Guid.NewGuid()}";
            string localPath = "./data";
            string fileName = $"quickstart-{Guid.NewGuid()}.txt";
            string localFilePath = Path.Combine(localPath, fileName);

            try
            {
                // Step 1: Create BlobServiceClient
                Console.WriteLine("1. Creating BlobServiceClient...");
                var serviceClient = new BlobServiceClient(
                    new Uri($"https://{storageAccountName}.blob.core.windows.net"),
                    new DefaultAzureCredential());
                Console.WriteLine("   ✓ BlobServiceClient created\n");

                // Step 2: Create container
                Console.WriteLine($"2. Creating container: {containerName}...");
                BlobContainerClient containerClient = await serviceClient.CreateBlobContainerAsync(containerName);
                Console.WriteLine($"   ✓ Container created: {containerClient.Uri}\n");

                // Step 3: Create local file with data
                Console.WriteLine("3. Creating local file...");
                Directory.CreateDirectory(localPath);
                await File.WriteAllTextAsync(localFilePath, 
                    $"Hello from Azure Blob Storage!\nCreated at: {DateTime.Now}");
                Console.WriteLine($"   ✓ Created file: {localFilePath}\n");

                // Step 4: Upload file to blob storage
                Console.WriteLine($"4. Uploading file to blob: {fileName}...");
                BlobClient blobClient = containerClient.GetBlobClient(fileName);
                
                using FileStream uploadFileStream = File.OpenRead(localFilePath);
                await blobClient.UploadAsync(uploadFileStream, overwrite: true);
                uploadFileStream.Close();
                
                Console.WriteLine($"   ✓ File uploaded: {blobClient.Uri}\n");

                // Step 5: Verify blob exists
                Console.WriteLine("5. Verifying blob exists...");
                bool exists = await blobClient.ExistsAsync();
                Console.WriteLine($"   ✓ Blob exists: {exists}\n");

                // Step 6: List blobs in container
                Console.WriteLine("6. Listing blobs in container...");
                await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
                {
                    Console.WriteLine($"   - {blobItem.Name}");
                    Console.WriteLine($"     Size: {blobItem.Properties.ContentLength} bytes");
                    Console.WriteLine($"     Content Type: {blobItem.Properties.ContentType}");
                    Console.WriteLine($"     Created: {blobItem.Properties.CreatedOn}");
                }
                Console.WriteLine();

                // Step 7: Download blob
                Console.WriteLine("7. Downloading blob...");
                string downloadFilePath = localFilePath.Replace(".txt", "_DOWNLOADED.txt");
                
                using FileStream downloadFileStream = File.OpenWrite(downloadFilePath);
                await blobClient.DownloadToAsync(downloadFileStream);
                downloadFileStream.Close();
                
                Console.WriteLine($"   ✓ File downloaded: {downloadFilePath}\n");

                // Step 8: Verify downloaded content
                Console.WriteLine("8. Verifying downloaded content...");
                string downloadedContent = await File.ReadAllTextAsync(downloadFilePath);
                Console.WriteLine($"   Content:\n   {downloadedContent.Replace("\n", "\n   ")}\n");

                Console.WriteLine("✓ Exercise completed successfully!");
                Console.WriteLine("\nPress any key to clean up resources...");
                Console.ReadKey();

                // Step 9: Cleanup (optional)
                Console.WriteLine("\n9. Cleaning up resources...");
                await containerClient.DeleteAsync();
                Console.WriteLine("   ✓ Container deleted");

                // Delete local files
                if (Directory.Exists(localPath))
                {
                    Directory.Delete(localPath, true);
                    Console.WriteLine("   ✓ Local files deleted");
                }
            }
            catch (Exception ex)
            {
                Console.WriteLine($"\n❌ Error: {ex.Message}");
                Console.WriteLine($"Stack Trace: {ex.StackTrace}");
            }
        }
    }
}
```

---

## Step 4: Configure and Run the Application

### Update Storage Account Name

```bash
# Replace placeholder with your storage account name
sed -i "s/REPLACE_WITH_YOUR_STORAGE_ACCOUNT_NAME/$STORAGE_ACCOUNT/" Program.cs

# Verify the change
grep "storageAccountName" Program.cs
```

### Run the Application

```bash
# Build and run
dotnet run
```

### Expected Output

```
Azure Blob Storage Exercise

1. Creating BlobServiceClient...
   ✓ BlobServiceClient created

2. Creating container: demo-container-abc123...
   ✓ Container created: https://staz204xyz.blob.core.windows.net/demo-container-abc123

3. Creating local file...
   ✓ Created file: ./data/quickstart-def456.txt

4. Uploading file to blob: quickstart-def456.txt...
   ✓ File uploaded: https://staz204xyz.blob.core.windows.net/demo-container-abc123/quickstart-def456.txt

5. Verifying blob exists...
   ✓ Blob exists: True

6. Listing blobs in container...
   - quickstart-def456.txt
     Size: 59 bytes
     Content Type: text/plain
     Created: 1/3/2026 10:30:00 AM +00:00

7. Downloading blob...
   ✓ File downloaded: ./data/quickstart-def456_DOWNLOADED.txt

8. Verifying downloaded content...
   Content:
   Hello from Azure Blob Storage!
   Created at: 1/3/2026 10:30:00 AM

✓ Exercise completed successfully!

Press any key to clean up resources...

9. Cleaning up resources...
   ✓ Container deleted
   ✓ Local files deleted
```

---

## Step 5: Verify in Azure Portal

### View Container (Before Cleanup)

```bash
# List containers (run before cleanup)
az storage container list \
    --account-name $STORAGE_ACCOUNT \
    --auth-mode login \
    --output table
```

### View Blobs

```bash
# List blobs in container (update container name)
az storage blob list \
    --account-name $STORAGE_ACCOUNT \
    --container-name <container-name> \
    --auth-mode login \
    --output table
```

### Azure Portal Steps (Шаги в Azure Portal)

1. Откройте Azure Portal
2. Перейдите в ваш **Storage Account**
3. В разделе **Data storage** выберите **Containers**
4. Найдите нужный контейнер (например, `demo-container-*`)
5. Откройте контейнер, чтобы увидеть список blob
6. Нажмите на конкретный blob, чтобы посмотреть его свойства (Properties)

> 💡 Практический совет:  
> В свойствах blob полезно проверять **Access tier**, **Last modified**, **Content-Type** и наличие **metadata/tags** — это часто помогает при отладке и проверке lifecycle/policy сценариев.


---

## Step 6: Clean Up Resources

### Delete Resource Group

```bash
# Delete entire resource group
az group delete \
    --name $RESOURCE_GROUP \
    --yes \
    --no-wait

echo "Resource group deletion initiated"
```

---

## Code Breakdown

### 1. Create BlobServiceClient with DefaultAzureCredential

```csharp
var serviceClient = new BlobServiceClient(
    new Uri($"https://{storageAccountName}.blob.core.windows.net"),
    new DefaultAzureCredential());
```
### DefaultAzureCredential (Как работает цепочка аутентификации)

`DefaultAzureCredential` последовательно пытается использовать несколько способов аутентификации:

1. Переменные окружения (Environment variables)
2. Managed Identity (если приложение запущено в Azure)
3. Учетные данные Visual Studio
4. Azure CLI
5. Azure PowerShell

> 💡 Это делает `DefaultAzureCredential` удобным для разработки и production:  
> локально используется CLI/Visual Studio,  
> в Azure — автоматически применяется Managed Identity.

### 2. Create Container

```csharp
BlobContainerClient containerClient = await serviceClient.CreateBlobContainerAsync(containerName);
```

- Creates a new container in the storage account
- Returns a `BlobContainerClient` for further operations
- Container name must be lowercase, 3-63 characters

### 3. Upload File

```csharp
BlobClient blobClient = containerClient.GetBlobClient(fileName);
using FileStream uploadFileStream = File.OpenRead(localFilePath);
await blobClient.UploadAsync(uploadFileStream, overwrite: true);
```

- Opens file as stream
- Uploads to blob storage
- `overwrite: true` replaces existing blob if present

### 4. List Blobs

```csharp
await foreach (BlobItem blobItem in containerClient.GetBlobsAsync())
{
    Console.WriteLine($"   - {blobItem.Name}");
}
```

- Asynchronously iterates through blobs
- Returns `BlobItem` with properties

### 5. Download Blob

```csharp
using FileStream downloadFileStream = File.OpenWrite(downloadFilePath);
await blobClient.DownloadToAsync(downloadFileStream);
```

- Creates write stream to local file
- Downloads blob content to stream

---

## Troubleshooting

### Issue: Authentication Failed

**Error:**
```
Status: 403 (This request is not authorized to perform this operation using this permission.)
```

**Solution:**
```bash
# Verify role assignment
az role assignment list \
    --assignee $(az ad signed-in-user show --query id -o tsv) \
    --scope $(az storage account show --name $STORAGE_ACCOUNT --resource-group $RESOURCE_GROUP --query id -o tsv)

# Wait 60 seconds for role propagation
sleep 60
```

### Issue: Storage Account Not Found

**Error:**
```
The specified storage account does not exist.
```

**Solution:**
```bash
# Verify storage account name
az storage account show \
    --name $STORAGE_ACCOUNT \
    --resource-group $RESOURCE_GROUP
```

### Issue: Package Not Found

**Error:**
```
error NU1101: Unable to find package Azure.Storage.Blobs
```

**Solution:**
```bash
# Restore packages
dotnet restore

# Clear package cache if needed
dotnet nuget locals all --clear
dotnet restore
```

---

## Key Takeaways (Ключевые выводы)

✅ **BlobServiceClient** — точка входа для операций с Blob Storage  
✅ **DefaultAzureCredential** — гибкая аутентификация с автоматическим выбором метода  
✅ **RBAC роли** — обязательны для Azure AD аутентификации  
✅ **Async/await** — все операции выполняются асинхронно  
✅ **Уникальные имена** — используйте GUID для предотвращения конфликтов  
✅ **Работа со stream** — эффективна для больших файлов  
✅ **Очистка ресурсов** — удаляйте контейнеры и blob после завершения работы

---

## Exam Tips (Советы к экзамену AZ-204)

🎯 **DefaultAzureCredential** автоматически перебирает методы аутентификации  
(Environment, Managed Identity, Azure CLI и др.)

🎯 **RBAC обязателен:**  
Для операций записи требуется роль `Storage Blob Data Contributor`

🎯 **CreateBlobContainerAsync** возвращает объект `BlobContainerClient`

🎯 **GetBlobClient** — переход от контейнера к конкретному blob

🎯 **UploadAsync** — загрузка из stream; параметр `overwrite` управляет перезаписью

🎯 **GetBlobsAsync** — асинхронное перечисление blob

🎯 **DownloadToAsync** — скачивание в stream (файл, память и т.д.)

🎯 **ExistsAsync** — проверка существования blob перед операциями

🎯 **Требования к имени контейнера:**
- только lowercase
- 3–63 символа
- без последовательных дефисов

---

## Additional Exercises (Дополнительные упражнения)

### Exercise Variations (Варианты задания)

1. **Загрузка нескольких файлов** — модифицировать код для загрузки всех файлов из директории
2. **Установка metadata** — добавить пользовательские метаданные при загрузке
3. **Установка access tier** — загрузить blob в Cool или Archive tier
4. **Условные операции** — использовать If-Match для контроля конкуренции
5. **Копирование blob** — копировать blob в другой контейнер
6. **Snapshots** — создать и восстановить snapshot blob

> 💡 Практический совет:  
> Для подготовки к экзамену полезно реализовать все вариации руками — это закрепляет понимание SDK и REST-модели Blob Storage.


---

## Additional Resources

- [Azure Blob Storage quickstart for .NET](https://learn.microsoft.com/en-us/azure/storage/blobs/storage-quickstart-blobs-dotnet)
- [DefaultAzureCredential](https://learn.microsoft.com/en-us/dotnet/api/azure.identity.defaultazurecredential)
- [BlobServiceClient](https://learn.microsoft.com/en-us/dotnet/api/azure.storage.blobs.blobserviceclient)

[Microsoft Learn - Exercise: Create Blob storage resources by using the .NET client library](https://learn.microsoft.com/en-us/training/modules/work-azure-blob-storage/4-develop-blob-storage-dotnet)
