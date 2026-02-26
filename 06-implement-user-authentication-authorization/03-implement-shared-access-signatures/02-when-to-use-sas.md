# When to Use Shared Access Signatures

## Ключевые понятия

- **Delegated access** — предоставление доступа без передачи ключей аккаунта
- **Time-limited** — временный доступ к ресурсам хранения
- **Два архитектурных подхода** — Front-end proxy и SAS provider service
- **Copy operations** — SAS требуется для копирования между аккаунтами

---

# Основные сценарии использования SAS

## Главный сценарий

Используйте **SAS**, когда необходимо предоставить безопасный доступ к ресурсам Storage клиенту, который не имеет прямых прав доступа.

> 💡 SAS позволяет делегировать доступ без раскрытия account key.

---

# Типовые сценарии

| Сценарий | Почему SAS? | Альтернатива |
|------------|------------|--------------|
| **Хранение пользовательских данных** | Пользователь читает/записывает только свои данные | Сервис аутентифицирует пользователя и генерирует SAS | Front-end proxy (весь трафик через сервер) |
| **Доступ третьей стороны** | Временный доступ для внешнего приложения | Ограниченный по времени SAS с конкретными правами | Передача account key (небезопасно) |
| **Mobile/desktop приложения** | Клиенту нужен прямой доступ к Storage | SAS позволяет избежать хранения ключей в приложении | Проксирование всех запросов (медленнее) |
| **Копирование между аккаунтами** | Требуется доступ к исходному ресурсу | SAS авторизует чтение источника | Ручное скачивание и загрузка |
| **Временное публичное предоставление файла** | Предоставить доступ к конкретному файлу | Генерация SAS-ссылки | Сделать контейнер публичным (менее безопасно) |

---

# Два архитектурных подхода

## 1️⃣ Front-End Proxy

- Клиент отправляет запрос серверу.
- Сервер обращается к Storage.
- Контроль полностью на сервере.
- Повышенная безопасность, но большая нагрузка.

## 2️⃣ SAS Provider Service

- Клиент аутентифицируется.
- Сервер проверяет права.
- Сервер генерирует короткоживущий SAS.
- Клиент обращается к Storage напрямую.

> 💡 SAS provider снижает нагрузку на сервер и сохраняет контроль.

---

## Когда SAS особенно полезен

- Нужен временный доступ.
- Необходимо ограничить операции.
- Нельзя раскрывать ключи Storage Account.
- Требуется доступ без создания полноценной RBAC-модели.

---

## Важно для AZ-204

- SAS используется для делегированного доступа.
- Часто применяется в мобильных и SPA приложениях.
- Обязателен для копирования между Storage Account.
- Предпочтителен для временного доступа к отдельным объектам.
- User Delegation SAS — самый безопасный вариант.

---

> 🎯 Частый экзаменационный вопрос:  
Как предоставить мобильному приложению доступ к Blob Storage без передачи account key?  
Ответ — использовать SAS.


## Design Patterns

### Pattern 1: Front-End Proxy Service

**All data flows through proxy**:

```
Client → Front-End Proxy → Authenticate → Storage Account
         ↑                                        ↓
         └────────────── Data Flow ──────────────┘
```

**Architecture**:

```
┌─────────┐      ┌──────────────────┐      ┌─────────────┐
│ Client  │─────→│  Proxy Service   │─────→│   Storage   │
│         │←─────│  (Auth + Logic)  │←─────│   Account   │
└─────────┘      └──────────────────┘      └─────────────┘
```

## Характеристики подхода Front-End Proxy

| Аспект | Детали |
|--------|--------|
| **Аутентификация** | Прокси-сервис полностью обрабатывает аутентификацию |
| **Поток данных** | Весь трафик проходит через прокси |
| **Бизнес-логика** | Возможна валидация, трансформация, логирование |
| **Производительность** | Может стать узким местом при больших объёмах данных |
| **Масштабирование** | Дорого масштабируется при высокой нагрузке |
| **Контроль** | Полный контроль над входящими и исходящими запросами |

---

## Когда использовать

- Требуется строгий контроль доступа.
- Необходима сложная бизнес-логика.
- Нужно централизованное логирование и аудит.
- Работа с чувствительными данными.

---

## Плюсы

- Максимальный контроль.
- Простота реализации правил безопасности.
- Централизованная точка проверки.

## Минусы

- Повышенная нагрузка на сервер.
- Потенциальные проблемы с производительностью.
- Более высокая стоимость масштабирования.

---

## Важно для AZ-204

- Front-end proxy подходит для high-security сценариев.
- Может быть альтернативой прямому использованию SAS.
- Неэффективен для больших файлов и массовых загрузок.
- Используется, когда SAS-подход неприемлем по рискам.

> 🎯 Частый экзаменационный вопрос:  
Как обеспечить полный контроль и аудит всех запросов к Storage?  
Ответ — использовать Front-End Proxy.

**Example implementation**:

```csharp
// ASP.NET Core API - Front-end proxy
[ApiController]
[Route("api/[controller]")]
public class FilesController : ControllerBase
{
    private readonly BlobServiceClient _blobServiceClient;

    public FilesController(BlobServiceClient blobServiceClient)
    {
        _blobServiceClient = blobServiceClient;
    }

    [HttpGet("{fileName}")]
    [Authorize]  // User must authenticate
    public async Task<IActionResult> DownloadFile(string fileName)
    {
        // Validate business rules
        if (!await ValidateUserAccess(fileName))
        {
            return Forbid();
        }

        // Get blob using service credentials
        var containerClient = _blobServiceClient.GetBlobContainerClient("user-files");
        var blobClient = containerClient.GetBlobClient(fileName);

        // Download and stream to client
        var download = await blobClient.OpenReadAsync();
        return File(download, "application/octet-stream", fileName);
    }

    [HttpPost]
    [Authorize]
    public async Task<IActionResult> UploadFile(IFormFile file)
    {
        // Validate business rules (file size, type, naming)
        if (file.Length > 10_000_000)  // 10 MB limit
        {
            return BadRequest("File too large");
        }

        // Upload using service credentials
        var containerClient = _blobServiceClient.GetBlobContainerClient("user-files");
        var blobClient = containerClient.GetBlobClient(file.FileName);
        
        using var stream = file.OpenReadStream();
        await blobClient.UploadAsync(stream, overwrite: true);

        return Ok(new { message = "File uploaded successfully" });
    }
}
```

## Преимущества и недостатки подхода Front-End Proxy

### ✅ Плюсы

- Полный контроль над бизнес-логикой
- Централизованная аутентификация и авторизация
- Возможность валидации, трансформации и логирования данных
- Поддержка сложных правил доступа
- Сокрытие внутренней структуры Storage от клиентов

---

### ❌ Минусы

- Весь трафик проходит через прокси (затраты на пропускную способность)
- Дорогое масштабирование при высокой нагрузке
- Единая точка отказа
- Повышенная задержка при работе с крупными файлами
- Дополнительные инфраструктурные расходы

---

## Когда оправдано использование

- Требуется строгий контроль доступа
- Необходим аудит и централизованная проверка
- Работа с чувствительными или регулируемыми данными
- Нужна сложная логика авторизации

---

## Важно для AZ-204

- Front-End Proxy даёт максимальный контроль, но увеличивает нагрузку.
- Не подходит для сценариев с большими файлами и высокой пропускной способностью.
- Альтернатива — SAS provider service для снижения нагрузки.
- Часто применяется в high-security архитектурах.

> 🎯 Ключевая идея:  
> Больше контроля — больше нагрузки и затрат.

### Pattern 2: SAS Provider Service

**Lightweight service generates SAS, clients access storage directly**:

```
Client → SAS Provider → Authenticate → Generate SAS
   ↓                                          ↓
   └──────────→ Storage Account ←────────────┘
                 (Direct Access)
```

**Architecture**:

```
┌─────────┐      ┌──────────────────┐
│ Client  │─────→│  SAS Provider    │
│         │←─────│  (Auth Only)     │
└────┬────┘      └──────────────────┘
     │                     
     │ Direct Access with SAS
     ↓                     
┌─────────────┐
│   Storage   │
│   Account   │
└─────────────┘
```

## Характеристики подхода SAS Provider Service

| Аспект | Детали |
|--------|--------|
| **Аутентификация** | Сервис аутентифицирует клиента и генерирует SAS |
| **Поток данных** | После выдачи SAS клиент обращается к Storage напрямую |
| **Бизнес-логика** | Ограничена проверкой и генерацией SAS |
| **Производительность** | Высокая — прямой доступ к Storage |
| **Масштабирование** | Проще масштабировать, меньше серверного трафика |
| **Контроль** | Ограничен после выдачи SAS |

---

## Как работает

1. Клиент проходит аутентификацию в сервисе.
2. Сервис проверяет права доступа.
3. Сервис генерирует короткоживущий SAS.
4. Клиент напрямую взаимодействует с Azure Storage.

---

## Преимущества

- Минимальная нагрузка на сервер.
- Высокая производительность.
- Подходит для больших файлов.
- Легче масштабировать при высоком трафике.

## Ограничения

- После выдачи SAS контроль ограничен.
- Невозможно отозвать SAS мгновенно без stored access policy.
- Требуется строгий контроль срока действия и разрешений.

---

## Когда использовать

- Загрузка и скачивание больших файлов.
- Mobile/SPA приложения.
- Высоконагруженные сценарии.
- Временный делегированный доступ.

---

## Важно для AZ-204

- SAS Provider Service — компромисс между безопасностью и производительностью.
- Предпочтителен для high-volume сценариев.
- Необходим короткий срок действия SAS.
- User Delegation SAS — наиболее безопасный вариант.

> 🎯 Частый экзаменационный вопрос:  
Как уменьшить нагрузку на backend при загрузке больших файлов?  
Ответ — использовать SAS Provider Service.


**Example implementation**:

```csharp
// ASP.NET Core API - SAS Provider
[ApiController]
[Route("api/[controller]")]
public class SasController : ControllerBase
{
    private readonly BlobServiceClient _blobServiceClient;

    public SasController(BlobServiceClient blobServiceClient)
    {
        _blobServiceClient = blobServiceClient;
    }

    [HttpGet("download/{fileName}")]
    [Authorize]
    public async Task<IActionResult> GetDownloadSas(string fileName)
    {
        // Validate user has permission to access this file
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        if (!await UserOwnsFile(userId, fileName))
        {
            return Forbid();
        }

        // Get blob client
        var containerClient = _blobServiceClient.GetBlobContainerClient("user-files");
        var blobClient = containerClient.GetBlobClient(fileName);

        // Generate short-lived read-only SAS
        BlobSasBuilder sasBuilder = new BlobSasBuilder()
        {
            BlobContainerName = "user-files",
            BlobName = fileName,
            Resource = "b",
            StartsOn = DateTimeOffset.UtcNow,
            ExpiresOn = DateTimeOffset.UtcNow.AddMinutes(15)  // 15 min expiry
        };

        sasBuilder.SetPermissions(BlobSasPermissions.Read);

        Uri sasUri = blobClient.GenerateSasUri(sasBuilder);

        return Ok(new 
        { 
            sasUrl = sasUri.ToString(),
            expiresAt = sasBuilder.ExpiresOn
        });
    }

    [HttpGet("upload/{fileName}")]
    [Authorize]
    public IActionResult GetUploadSas(string fileName)
    {
        var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;

        // Validate file name, check quota, etc.
        if (!ValidateFileName(fileName))
        {
            return BadRequest("Invalid file name");
        }

        // Generate write SAS
        var containerClient = _blobServiceClient.GetBlobContainerClient("user-files");
        var blobClient = containerClient.GetBlobClient($"{userId}/{fileName}");

        BlobSasBuilder sasBuilder = new BlobSasBuilder()
        {
            BlobContainerName = "user-files",
            BlobName = $"{userId}/{fileName}",
            Resource = "b",
            StartsOn = DateTimeOffset.UtcNow,
            ExpiresOn = DateTimeOffset.UtcNow.AddMinutes(30)
        };

        sasBuilder.SetPermissions(BlobSasPermissions.Create | BlobSasPermissions.Write);

        Uri sasUri = blobClient.GenerateSasUri(sasBuilder);

        return Ok(new 
        { 
            sasUrl = sasUri.ToString(),
            expiresAt = sasBuilder.ExpiresOn
        });
    }
}
```

**Client usage**:

```javascript
// JavaScript client using SAS
async function downloadFile(fileName) {
    // Step 1: Get SAS from service
    const response = await fetch(`/api/sas/download/${fileName}`, {
        headers: {
            'Authorization': `Bearer ${accessToken}`
        }
    });
    
    const { sasUrl, expiresAt } = await response.json();
    
    // Step 2: Download directly from storage using SAS
    const blob = await fetch(sasUrl);
    const data = await blob.blob();
    
    // Step 3: Save or display file
    const url = URL.createObjectURL(data);
    const a = document.createElement('a');
    a.href = url;
    a.download = fileName;
    a.click();
}

async function uploadFile(file) {
    // Step 1: Get upload SAS from service
    const response = await fetch(`/api/sas/upload/${file.name}`, {
        headers: {
            'Authorization': `Bearer ${accessToken}`
        }
    });
    
    const { sasUrl } = await response.json();
    
    // Step 2: Upload directly to storage using SAS
    await fetch(sasUrl, {
        method: 'PUT',
        headers: {
            'x-ms-blob-type': 'BlockBlob',
            'Content-Type': file.type
        },
        body: file
    });
    
    console.log('File uploaded successfully');
}
```
## Преимущества и недостатки SAS Provider Service

### ✅ Плюсы

- Прямой доступ к Storage (выше скорость, ниже задержка)
- Снижение затрат на пропускную способность прокси
- Проще масштабировать (меньше трафика через сервис)
- Более низкие инфраструктурные расходы
- Клиент может самостоятельно повторять загрузки/скачивания

---

### ❌ Минусы

- Ограниченный контроль после выдачи SAS
- Невозможно изменять данные «на лету»
- Менее детализированное логирование на уровне бизнес-логики
- Бизнес-логика ограничена этапом генерации SAS
- Клиент должен работать напрямую с API Storage

---

## Когда выбирать этот подход

- Большие файлы и массовые загрузки
- Высокая нагрузка
- Mobile/SPA клиенты
- Не требуется сложная серверная обработка данных

---

## Важно для AZ-204

- SAS Provider снижает нагрузку на backend.
- Контроль доступа ограничен временем действия SAS.
- Требуется минимальный срок жизни и минимальные разрешения.
- Предпочтительно использовать User Delegation SAS.

> 🎯 Ключевая идея:  
> Максимальная производительность — при ограниченном контроле после выдачи SAS.

### Pattern 3: Hybrid Approach

**Combine both patterns** for optimal balance:

```csharp
[ApiController]
[Route("api/[controller]")]
public class HybridController : ControllerBase
{
    [HttpPost("small-file")]
    [Authorize]
    public async Task<IActionResult> UploadSmallFile(IFormFile file)
    {
        // Small files: Use proxy for validation and transformation
        if (file.Length < 1_000_000)  // < 1 MB
        {
            // Validate, scan for viruses, create thumbnail, etc.
            var processedData = await ProcessFile(file);
            
            // Upload to storage
            await UploadToStorage(processedData);
            
            return Ok();
        }
        
        return BadRequest("Use large file endpoint for files > 1 MB");
    }

    [HttpGet("large-file-sas/{fileName}")]
    [Authorize]
    public IActionResult GetLargeFileUploadSas(string fileName)
    {
        // Large files: Generate SAS for direct upload
        var sasUri = GenerateSasUri(fileName, write: true, expiryMinutes: 60);
        
        return Ok(new { sasUrl = sasUri.ToString() });
    }
}
```

## Типовые сценарии комбинированного подхода

- **Малые файлы через proxy** — требуется обработка или валидация
- **Крупные файлы через SAS** — приоритет производительности
- **Чувствительные данные через proxy** — дополнительное шифрование и контроль
- **Публичные данные через SAS** — снижение затрат и нагрузки

> 💡 Часто используется гибридная архитектура: proxy для контроля + SAS для масштабируемости.

---

# Copy Operations с использованием SAS

## Требования для копирования между аккаунтами

При копировании данных между разными Storage Account **требуется SAS** для доступа к источнику.

| Операция копирования | SAS обязателен для | SAS опционален для |
|----------------------|--------------------|---------------------|
| **Blob → Blob (разные аккаунты)** | Исходный blob | Целевой blob |
| **File → File (разные аккаунты)** | Исходный файл | Целевой файл |
| **Blob → File** | Исходный объект | Целевой объект |
| **File → Blob** | Исходный объект | Целевой объект |
| **Копирование внутри одного аккаунта** | Не требуется | Не требуется |

---

## Что важно понимать

- SAS используется для авторизации чтения из источника.
- Если у клиента уже есть права на целевой ресурс — SAS для назначения может быть не нужен.
- Для cross-account копирования SAS почти всегда необходим.

---

## Важно для AZ-204

- Копирование между разными Storage Account требует SAS.
- SAS чаще всего требуется для исходного объекта.
- Внутри одного аккаунта SAS не обязателен.
- Copy operations — частый экзаменационный сценарий.

> 🎯 Частый экзаменационный вопрос:  
Нужно ли SAS при копировании blob внутри одного Storage Account?  
Ответ — нет.


### Example: Cross-Account Blob Copy

```csharp
// Source account: Generate read SAS
BlobClient sourceBlobClient = new BlobClient(
    new Uri("https://sourceaccount.blob.core.windows.net/source-container/file.txt"),
    new StorageSharedKeyCredential(sourceAccountName, sourceAccountKey)
);

BlobSasBuilder sourceSasBuilder = new BlobSasBuilder()
{
    BlobContainerName = "source-container",
    BlobName = "file.txt",
    Resource = "b",
    ExpiresOn = DateTimeOffset.UtcNow.AddHours(1)
};

sourceSasBuilder.SetPermissions(BlobSasPermissions.Read);
Uri sourceSasUri = sourceBlobClient.GenerateSasUri(sourceSasBuilder);

// Destination account: Start copy
BlobClient destBlobClient = new BlobClient(
    new Uri("https://destaccount.blob.core.windows.net/dest-container/file.txt"),
    new StorageSharedKeyCredential(destAccountName, destAccountKey)
);

// Copy from source SAS URI
await destBlobClient.StartCopyFromUriAsync(sourceSasUri);

// Monitor copy status
BlobProperties properties = await destBlobClient.GetPropertiesAsync();
while (properties.CopyStatus == CopyStatus.Pending)
{
    await Task.Delay(1000);
    properties = await destBlobClient.GetPropertiesAsync();
}

if (properties.CopyStatus == CopyStatus.Success)
{
    Console.WriteLine("Copy completed successfully");
}
```

### Azure CLI Copy Example

```bash
# Generate source SAS
SOURCE_SAS=$(az storage blob generate-sas \
    --account-name sourceaccount \
    --container-name source-container \
    --name file.txt \
    --permissions r \
    --expiry 2024-12-31T23:59:00Z \
    --https-only \
    --output tsv)

# Build source URL with SAS
SOURCE_URL="https://sourceaccount.blob.core.windows.net/source-container/file.txt?${SOURCE_SAS}"

# Start copy to destination account
az storage blob copy start \
    --account-name destaccount \
    --destination-container dest-container \
    --destination-blob file.txt \
    --source-uri "$SOURCE_URL"

# Check copy status
az storage blob show \
    --account-name destaccount \
    --container-name dest-container \
    --name file.txt \
    --query "properties.copy" \
    --output table
```

## Decision Tree: When to Use SAS

```
Need storage access?
│
├─ Azure service? → Use Managed Identity
│
├─ Internal app? → Use Microsoft Entra ID
│
└─ External client or temporary access?
   │
   ├─ Large data volumes? → SAS Provider Pattern
   │
   ├─ Need business logic? → Front-End Proxy Pattern
   │
   ├─ Both? → Hybrid Pattern
   │
   └─ High security? → Consider Middle-Tier Service
```

## Real-World Use Cases

### 1. User File Upload (SPA + Azure Storage)

**Scenario**: React app needs to upload user profile photos

```typescript
// React component
async function uploadProfilePhoto(file: File) {
    // Get upload SAS from backend
    const sasResponse = await fetch('/api/profile/photo-upload-sas', {
        headers: { 'Authorization': `Bearer ${token}` }
    });
    
    const { sasUrl } = await sasResponse.json();
    
    // Upload directly to blob storage
    await fetch(sasUrl, {
        method: 'PUT',
        headers: {
            'x-ms-blob-type': 'BlockBlob',
            'Content-Type': file.type
        },
        body: file
    });
}
```

### 2. Report Generation

**Scenario**: Generate large reports, let users download directly

```csharp
[HttpPost("generate-report")]
public async Task<IActionResult> GenerateReport(ReportRequest request)
{
    // Generate report (heavy operation)
    var reportData = await _reportService.GenerateAsync(request);
    
    // Save to blob storage
    var blobName = $"reports/{Guid.NewGuid()}.pdf";
    var blobClient = _containerClient.GetBlobClient(blobName);
    await blobClient.UploadAsync(new BinaryData(reportData));
    
    // Generate short-lived SAS for download
    var sasUri = blobClient.GenerateSasUri(new BlobSasBuilder
    {
        BlobContainerName = "reports",
        BlobName = blobName,
        Resource = "b",
        ExpiresOn = DateTimeOffset.UtcNow.AddHours(24)
    }.SetPermissions(BlobSasPermissions.Read));
    
    return Ok(new { downloadUrl = sasUri.ToString() });
}
```

### 3. Mobile App Media Upload

**Scenario**: Mobile app uploads photos/videos directly to storage

```swift
// iOS Swift
func uploadMedia(mediaData: Data, fileName: String) async throws {
    // Get SAS from backend
    let sasUrl = try await getSasUrl(for: fileName)
    
    // Upload directly using URLSession
    var request = URLRequest(url: URL(string: sasUrl)!)
    request.httpMethod = "PUT"
    request.setValue("BlockBlob", forHTTPHeaderField: "x-ms-blob-type")
    request.setValue("image/jpeg", forHTTPHeaderField: "Content-Type")
    request.httpBody = mediaData
    
    let (_, response) = try await URLSession.shared.data(for: request)
    
    guard (response as? HTTPURLResponse)?.statusCode == 201 else {
        throw UploadError.failed
    }
}
```

### 4. Third-Party Integration

**Scenario**: Grant external partner temporary access to specific files

```csharp
[HttpGet("partner/{partnerId}/files/{fileId}/access")]
[Authorize(Roles = "Admin")]
public IActionResult GrantPartnerAccess(string partnerId, string fileId)
{
    // Validate partner and file
    if (!ValidatePartnerAccess(partnerId, fileId))
    {
        return Forbid();
    }
    
    var blobClient = _containerClient.GetBlobClient($"partner-files/{fileId}");
    
    // Generate SAS valid for 7 days
    var sasUri = blobClient.GenerateSasUri(new BlobSasBuilder
    {
        BlobContainerName = "partner-files",
        BlobName = $"partner-files/{fileId}",
        Resource = "b",
        StartsOn = DateTimeOffset.UtcNow,
        ExpiresOn = DateTimeOffset.UtcNow.AddDays(7)
    }.SetPermissions(BlobSasPermissions.Read));
    
    // Log access grant for audit
    await _auditService.LogPartnerAccess(partnerId, fileId, sasUri.ToString());
    
    return Ok(new 
    { 
        accessUrl = sasUri.ToString(),
        validUntil = DateTimeOffset.UtcNow.AddDays(7)
    });
}
```
# Critical Notes

- 💡 **Основное назначение** — делегировать доступ без передачи account key
- 🎯 **Два архитектурных паттерна** — Front-end proxy (весь трафик через сервис) и SAS provider (прямой доступ)
- ✅ **Front-end proxy** — полный контроль и валидация, но дорого масштабируется
- ⚠️ **SAS provider** — быстро и масштабируемо, но контроль ограничен после выдачи SAS
- 🔄 **Гибридный подход** — малые файлы через proxy, большие через SAS
- 📊 **Copy operations** — SAS требуется при копировании между разными аккаунтами
- 💡 **Реальные сценарии** — загрузка файлов пользователями, скачивание отчётов, mobile-приложения, партнёрский доступ
- ✅ **Факторы выбора** — объём данных, требования к бизнес-логике, безопасность
- ⚠️ **Компромиссы** — контроль vs производительность, безопасность vs стоимость
- 🔒 **Лучше всего подходит** — для временного доступа, внешних клиентов, mobile/desktop приложений

---

# Exam Tips (AZ-204)

## Когда использовать SAS

- Предоставить временный и ограниченный доступ.
- Избежать передачи ключей Storage Account.
- Делегировать доступ внешним клиентам.

---

## Архитектурные паттерны

### Front-End Proxy
- Весь трафик проходит через сервис.
- Поддержка валидации, трансформации, логирования.
- Дорого масштабируется.
- Подходит для high-security сценариев.

### SAS Provider
- Лёгкий сервис, генерирующий SAS.
- Клиент обращается к Storage напрямую.
- Быстрее и дешевле.
- Контроль ограничен временем действия SAS.

### Hybrid
- Комбинация двух подходов.
- Выбор зависит от размера файла или требований безопасности.

---

## Copy Operations

- При копировании между разными Storage Account требуется SAS.
- SAS нужен для авторизации доступа к исходному объекту.
- Внутри одного аккаунта SAS не обязателен.

---

## Преимущества SAS Provider

- Более высокая производительность.
- Меньше затрат.
- Легче масштабировать.

## Преимущества Front-End Proxy

- Полный контроль.
- Централизованное логирование.
- Поддержка сложной бизнес-логики.

---

## Реальные сценарии

- Загрузка пользовательских файлов.
- Скачивание отчётов.
- Доступ мобильных приложений к Storage.
- Временный доступ партнёрам.

---

## Важные моменты

- Генерировать SAS с коротким сроком действия (часто 15–60 минут).
- Проверять права пользователя перед генерацией SAS.
- Рассматривать middle-tier сервис при повышенных требованиях безопасности.

---

Delegated permissions:
Используются, когда есть signed-in user
Приложение работает on behalf of user
Токен содержит scp

Application permissions:
Используются без пользователя
Background services / daemons
Токен содержит roles
> 🎯 Ключевая идея:  
> SAS — это инструмент для временного делегирования доступа.  
> Выбор архитектуры зависит от баланса между контролем, производительностью и стоимостью.


[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-shared-access-signatures/3-shared-access-signatures)
