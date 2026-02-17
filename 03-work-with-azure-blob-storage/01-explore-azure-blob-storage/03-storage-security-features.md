# Azure Storage Security Features

## Security Overview (Обзор безопасности)

Azure Storage предоставляет комплексные механизмы защиты данных:

1️⃣ **Шифрование данных при хранении (at rest)**  
2️⃣ **Шифрование при передаче (in transit, HTTPS)**  
3️⃣ **Client-side encryption**  
4️⃣ **Аутентификация и авторизация**  
5️⃣ **Сетевая безопасность**

---

# Azure Storage Encryption for Data at Rest

## Automatic Encryption (Автоматическое шифрование)

Azure Storage использует **Service-Side Encryption (SSE)**  
для автоматического шифрования данных при сохранении.

---

## Key Features (Основные характеристики)

| Feature | Details |
|----------|----------|
| **Algorithm** | AES-256 (256-bit Advanced Encryption Standard) |
| **Compliance** | FIPS 140-2 |
| **Status** | **Всегда включено** |
| **Scope** | Все storage accounts |
| **Performance tiers** | Standard и Premium |
| **Access tiers** | Hot, Cool, Cold, Archive |
| **Blob types** | Block, Append, Page |
| **Redundancy** | Все варианты репликации |
| **Metadata** | Метаданные также шифруются |
| **Cost** | Без дополнительной платы |

---

## Transparent Encryption (Прозрачное шифрование)

💡 Данные шифруются и расшифровываются автоматически.

✅ Изменения в коде не требуются  
⚠️ Шифрование нельзя отключить

---

## Encryption Scope (Что шифруется)

- ✅ Blobs
- ✅ Disks
- ✅ Files
- ✅ Queues
- ✅ Tables

---

## Geo-Replication Encryption

При включённой георепликации:

- Данные в primary регионе зашифрованы
- Данные в secondary регионе также зашифрованы
- Шифрование сохраняется на всех репликах

---

# Encryption Key Management (Управление ключами)

Azure Storage предлагает **3 варианта управления ключами**.

---

## 1️⃣ Microsoft-Managed Keys (По умолчанию)

Microsoft полностью управляет ключами.

### Characteristics

| Aspect | Details |
|--------|----------|
| **Key Storage** | Хранилище ключей Microsoft |
| **Key Rotation** | Автоматически Microsoft |
| **Key Control** | Microsoft |
| **Scope** | Account (по умолчанию), container или blob |
| **Supported Services** | Все сервисы Storage |
| **Complexity** | Минимальная |
| **Cost** | Включено |

✅ Подходит для большинства сценариев.

---

## 2️⃣ Customer-Managed Keys (CMK)

Клиент управляет ключами через:

- Azure Key Vault
- Azure Key Vault Managed HSM

### Characteristics

| Aspect | Details |
|--------|----------|
| **Key Storage** | Key Vault или Managed HSM |
| **Key Rotation** | Ответственность клиента |
| **Key Control** | Клиент |
| **Scope** | Account (по умолчанию), container или blob |
| **Supported Services** | Blob Storage, Azure Files |
| **Complexity** | Средняя |

> 💡 Используется при требованиях compliance или контроле над ключами.

---

## Важно для AZ-204

- SSE всегда включено
- Используется AES-256
- Нет дополнительной стоимости
- Microsoft-managed — по умолчанию
- CMK требует Key Vault
- Шифрование распространяется на geo-replication


#### Key Vault Integration

```bash
# Create Key Vault
az keyvault create \
  --name <keyvault-name> \
  --resource-group <resource-group> \
  --location <location>

# Create encryption key
az keyvault key create \
  --vault-name <keyvault-name> \
  --name <key-name> \
  --protection software

# Configure storage account to use customer-managed key
az storage account update \
  --name <account-name> \
  --resource-group <resource-group> \
  --encryption-key-name <key-name> \
  --encryption-key-vault <keyvault-uri> \
  --encryption-key-source Microsoft.Keyvault
```

#### PowerShell Example

```powershell
# Create Key Vault key
$key = Add-AzKeyVaultKey `
  -VaultName <keyvault-name> `
  -Name <key-name> `
  -Destination Software

# Configure storage account
Set-AzStorageAccount `
  -ResourceGroupName <resource-group> `
  -Name <account-name> `
  -KeyvaultEncryption `
  -KeyName <key-name> `
  -KeyVaultUri <keyvault-uri>
```

✅ **Best for**  
Сценарии, где требуется прямой контроль над ключами шифрования  
(например, строгие требования compliance или внутренние политики безопасности).

⚠️ **Requirements (Требования)**

- Azure Key Vault или Key Vault Managed HSM
- Назначенные разрешения (RBAC / access policies) к Key Vault
- Управление ротацией ключей на стороне клиента

---

# 3️⃣ Customer-Provided Keys (CPK)

## Overview (Обзор)

Вы предоставляете ключ шифрования **в каждом запросе** на чтение/запись.

Это самый детальный уровень контроля.

---

## Characteristics (Характеристики)

| Aspect | Details |
|--------|----------|
| **Key Storage** | Собственное хранилище ключей клиента |
| **Key Rotation** | Ответственность клиента |
| **Key Control** | Полный контроль клиента |
| **Scope** | На уровне каждого запроса |
| **Supported Services** | Только Blob Storage |
| **Complexity** | Самая высокая |

---

## Важно для AZ-204

- Microsoft-managed keys — по умолчанию
- CMK — через Key Vault
- CPK — ключ передаётся в каждом запросе
- CPK поддерживается только для Blob Storage
- Чем выше контроль — тем выше сложность управления

#### Usage Pattern

```csharp
// Create customer-provided key
var cpk = new CustomerProvidedKey(keyBytes);

// Upload with customer-provided key
BlobUploadOptions options = new BlobUploadOptions
{
    CustomerProvidedKey = cpk
};

await blobClient.UploadAsync(stream, options);

// Download with customer-provided key
BlobDownloadOptions downloadOptions = new BlobDownloadOptions
{
    CustomerProvidedKey = cpk
};

await blobClient.DownloadToAsync(stream, downloadOptions);
```

✅ **Best for**  
Максимально детальный контроль — возможность использовать разные ключи для разных blob.

⚠️ **Requirement**  
Ключ должен передаваться **в каждом read/write запросе**.

---

# Key Management Comparison (Сравнение управления ключами)

| Feature | Microsoft-Managed | Customer-Managed (CMK) | Customer-Provided (CPK) |
|----------|------------------|------------------------|--------------------------|
| **Key Storage** | Хранилище Microsoft | Azure Key Vault / HSM | Собственное хранилище |
| **Rotation Responsibility** | Microsoft | Клиент | Клиент |
| **Control Level** | Microsoft | Клиент | Клиент |
| **Granularity** | Account / container / blob | Account / container / blob | Per-request |
| **Services** | Все сервисы | Blob Storage, Azure Files | Только Blob Storage |
| **Complexity** | Низкая | Средняя | Высокая |
| **Setup Required** | Нет | Настройка Key Vault | Собственная система ключей |
| **Request Overhead** | Нет | Нет | Каждый запрос |

---

# Client-Side Encryption (Шифрование на стороне клиента)

## Overview (Обзор)

Client-side encryption позволяет:

- 🔐 Шифровать данные в клиентском приложении **до отправки в Azure**
- 🔓 Расшифровывать данные после скачивания

В этом случае Azure никогда не видит данные в открытом виде.

---

## Supported SDKs (Поддерживаемые SDK)

Client-side encryption поддерживается для:

- ✅ .NET
- ✅ Java
- ✅ Python

---

## Encryption Versions (Версии шифрования)

| Version | Algorithm | Mode | Supported Services |
|----------|-----------|------|-------------------|
| **Version 2** (Recommended) | AES | **GCM (Galois/Counter Mode)** | Blob Storage, Queue Storage |
| **Version 1** (Legacy) | AES | **CBC (Cipher Block Chaining)** | Blob, Queue, Table |

💡 **Recommendation**  
Используйте **Version 2 (GCM)** — более безопасный и эффективный режим.

---

## Важно для AZ-204

- SSE (server-side encryption) всегда включено
- CMK требует Key Vault
- CPK передаётся в каждом запросе
- Client-side encryption шифрует данные до отправки
- Version 2 (GCM) — предпочтительный вариант

### How It Works

```
Client Application
    ↓ (Encrypt data with AES-GCM)
Azure Blob Storage
    ↓ (Also encrypted at rest by Azure)
Storage Account
```

### Client-Side Encryption (.NET Example)

```csharp
using Azure.Storage.Blobs;
using Azure.Storage.Blobs.Specialized;

// Create blob client with client-side encryption
var clientOptions = new SpecializedBlobClientOptions
{
    ClientSideEncryption = new ClientSideEncryptionOptions(
        ClientSideEncryptionVersion.V2_0)
    {
        KeyEncryptionKey = keyEncryptionKey,
        KeyResolver = keyResolver,
        KeyWrapAlgorithm = "RSA-OAEP-256"
    }
};

var blobClient = new BlobClient(connectionString, containerName, blobName, clientOptions);

// Upload - data encrypted on client before upload
await blobClient.UploadAsync(stream);

// Download - data decrypted on client after download
await blobClient.DownloadToAsync(stream);
```

# Layered Encryption (Многоуровневое шифрование)

При использовании **Client-Side Encryption** применяется защита в несколько слоёв:

1️⃣ Клиент шифрует данные перед загрузкой (AES-GCM)  
2️⃣ Azure дополнительно шифрует данные при хранении (AES-256, SSE)  
3️⃣ В итоге получается **двойное шифрование**

> 💡 Это принцип *defense in depth* — защита на нескольких уровнях.

---

# Encryption at Rest vs. In Transit

| Type | When | How | Configuration |
|------|------|-----|---------------|
| **At Rest** | Когда данные хранятся на диске | AES-256 | Всегда включено автоматически |
| **In Transit** | При передаче по сети | HTTPS / TLS | Использовать HTTPS endpoints |

---

## Важно для AZ-204

- At Rest шифрование всегда включено и не настраивается
- In Transit шифрование требует HTTPS
- Client-side encryption даёт дополнительный уровень защиты
- Double encryption возможно при сочетании client-side + SSE
- Azure Storage поддерживает TLS для передачи данных


### Secure Transfer Required

```bash
# Enable secure transfer (HTTPS only)
az storage account update \
  --name <account-name> \
  --resource-group <resource-group> \
  --https-only true
```

```powershell
# PowerShell
Set-AzStorageAccount `
  -ResourceGroupName <resource-group> `
  -Name <account-name> `
  -EnableHttpsTrafficOnly $true
```

⚠⚠️ **Best Practice**  
Всегда включайте **"Secure transfer required"**, чтобы принудительно использовать HTTPS.

---

# Security Best Practices (Лучшие практики безопасности)

## 🔐 Encryption

### ✅ DO

- Доверяйте встроенному шифрованию (всегда включено)
- Используйте **HTTPS** для всех подключений
- Включайте **Secure transfer required**
- Используйте **Customer-Managed Keys (CMK)** при требованиях compliance
- Применяйте **client-side encryption** для чувствительных данных
- Используйте **AES-GCM (Version 2)** для client-side encryption

### ❌ DON'T

- Не считайте шифрование опциональным (оно всегда включено)
- Не используйте HTTP для чувствительных данных
- Не применяйте устаревший CBC (Version 1) в новых приложениях

---

## 🔑 Key Management

### ✅ DO

- Используйте **Microsoft-managed keys** в большинстве сценариев
- Используйте **CMK**, если требуется контроль над ключами
- Храните CMK в **Azure Key Vault**
- Регулярно ротируйте customer-managed ключи
- Документируйте процедуры ротации

### ❌ DON'T

- Не управляйте ключами без процессов и контроля
- Не храните ключи в коде или конфигурационных файлах
- Не забывайте про ротацию ключей

---

## 🔐 Access Control

### ✅ DO

- Используйте **Azure AD authentication**
- Следуйте принципу **least privilege**
- Используйте **SAS-токены** с минимальными правами и коротким сроком действия
- Мониторьте логи доступа

---

# Exam Tips (Советы для экзамена)

🎯 **Encryption always enabled**  
Шифрование Azure Storage всегда включено и не может быть отключено

🎯 **AES-256**  
Используется 256-bit AES, соответствие FIPS 140-2

🎯 **No extra cost**  
Шифрование at rest бесплатно

🎯 **Three key options**
- Microsoft-managed (по умолчанию)
- Customer-managed (через Key Vault)
- Customer-provided (per-request)

🎯 **Client-side encryption**
- Поддержка: .NET, Java, Python
- V2 (GCM) — рекомендовано
- V1 (CBC) — legacy

🎯 **CMK**
- Требует Azure Key Vault или Managed HSM
- Поддерживается для Blob Storage и Azure Files

🎯 **CPK**
- Поддерживается только для Blob Storage
- Ключ передаётся в каждом запросе

🎯 **Transparent encryption**
- Для SSE изменения в коде не требуются

🎯 **Secure transfer**
- Включайте HTTPS-only для шифрования in transit


---

## Quick Reference Commands

### Check Encryption Status
```bash
# Azure CLI
az storage account show \
  --name <account-name> \
  --resource-group <resource-group> \
  --query encryption

# PowerShell
(Get-AzStorageAccount `
  -ResourceGroupName <resource-group> `
  -Name <account-name>).Encryption
```

### Enable Secure Transfer (HTTPS Only)
```bash
# Azure CLI
az storage account update \
  --name <account-name> \
  --resource-group <resource-group> \
  --https-only true

# PowerShell
Set-AzStorageAccount `
  -ResourceGroupName <resource-group> `
  -Name <account-name> `
  -EnableHttpsTrafficOnly $true
```

### Configure Customer-Managed Key
```bash
# Azure CLI
az storage account update \
  --name <account-name> \
  --resource-group <resource-group> \
  --encryption-key-name <key-name> \
  --encryption-key-vault <keyvault-uri> \
  --encryption-key-source Microsoft.Keyvault

# PowerShell
Set-AzStorageAccount `
  -ResourceGroupName <resource-group> `
  -Name <account-name> `
  -KeyvaultEncryption `
  -KeyName <key-name> `
  -KeyVaultUri <keyvault-uri>
```

---

## Encryption Decision Tree

```
Do you need control over encryption keys?
├── No → Use Microsoft-managed keys (default)
│         ✅ Simplest option
│         ✅ No additional setup
│         ✅ Microsoft handles rotation
│
└── Yes → Do you need per-request granularity?
          ├── No → Use Customer-managed keys (CMK)
          │         ✅ Store in Azure Key Vault
          │         ✅ Control rotation
          │         ✅ Works with Blob Storage & Files
          │
          └── Yes → Use Customer-provided keys (CPK)
                    ✅ Maximum control
                    ✅ Different key per blob
                    ⚠️  Blob Storage only
                    ⚠️  Must provide on every request
```
# Encryption Key Decision Tree (Выбор типа управления ключами)

Нужен ли вам контроль над ключами шифрования?
│
├── ❌ Нет
│ → Используйте Microsoft-managed keys (по умолчанию)
│ ✅ Самый простой вариант
│ ✅ Не требует дополнительной настройки
│ ✅ Microsoft управляет ротацией ключей
│
└── ✅ Да
│
├── Нужна ли детализация на уровне каждого запроса?
│
├── ❌ Нет
│ → Используйте Customer-Managed Keys (CMK)
│ ✅ Хранятся в Azure Key Vault / HSM
│ ✅ Контроль ротации у клиента
│ ✅ Поддержка Blob Storage и Azure Files
│
└── ✅ Да
→ Используйте Customer-Provided Keys (CPK)
✅ Максимальный уровень контроля
✅ Можно использовать разные ключи для разных blob
⚠️ Поддерживается только для Blob Storage
⚠️ Ключ должен передаваться в каждом запросе


---

## Важно для AZ-204

- По умолчанию используется Microsoft-managed keys
- CMK требует Azure Key Vault
- CPK работает только с Blob Storage
- Чем выше контроль — тем выше сложность управления
- CPK = per-request гранулярность
---

## Additional Resources

- [Azure Storage encryption for data at rest](https://learn.microsoft.com/en-us/azure/storage/common/storage-service-encryption)
- [Customer-managed keys for Azure Storage encryption](https://learn.microsoft.com/en-us/azure/storage/common/customer-managed-keys-overview)
- [Client-side encryption for blobs](https://learn.microsoft.com/en-us/azure/storage/blobs/client-side-encryption)

[Microsoft Learn - Explore Azure Storage security features](https://learn.microsoft.com/en-us/training/modules/explore-azure-blob-storage/4-blob-storage-security)
