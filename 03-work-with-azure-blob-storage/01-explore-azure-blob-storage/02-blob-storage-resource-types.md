# Azure Blob Storage Resource Types
(Типы ресурсов в Azure Blob Storage)

## Resource Hierarchy (Иерархия ресурсов)

Blob Storage организован в **трёхуровневую иерархию ресурсов**:

---

## 1️⃣ Storage Account (Учётная запись хранения)

- Верхний уровень
- Глобально уникальное имя
- Определяет:
    - Регион
    - SKU (Standard / Premium)
    - Redundancy (LRS, GRS, ZRS и т.д.)
- Содержит контейнеры

Пример базового URL:
        https://mystorageaccount.blob.core.windows.net


---

## 2️⃣ Container (Контейнер)

- Аналог папки верхнего уровня
- Группирует blob-объекты
- Управляет:
    - Уровнем публичного доступа
    - Политиками доступа (SAS, RBAC)
- Обязателен для хранения blob
Пример:
  https://mystorageaccount.blob.core.windows.net/images


---

## 3️⃣ Blob (Объект)

- Фактические данные (файл)
- Может быть:
    - Block blob
    - Append blob
    - Page blob
- Хранит неструктурированные данные

Пример полного пути:
        https://mystorageaccount.blob.core.windows.net/images/photo.jpg


---

## Важно для AZ-204

- Storage Account → Container → Blob
- Blob не может существовать без контейнера
- Контейнер управляет уровнем публичного доступа
- Полный URL всегда включает account + container + blob


```
Storage Account (mystorageaccount)
└── Container (mycontainer)
    └── Blob (myblob)
        ├── Block Blob
        ├── Append Blob
        └── Page Blob
```

---

## 1. Storage Account

### Overview
A storage account provides a **unique namespace in Azure** for your data. Every object stored in Azure Storage has an address that includes your unique account name.

### Naming and Addressing

**Base endpoint format:**
```
http://<account-name>.blob.core.windows.net
```

**Example:**
```
http://mystorageaccount.blob.core.windows.net
```

## Key Characteristics (Storage Account)

| Characteristic | Details |
|----------------|---------|
| **Uniqueness** | Имя аккаунта должно быть глобально уникальным во всём Azure |
| **Namespace** | Создаёт уникальное пространство имён для всех данных |
| **Endpoint** | Формирует базовый URL для всех объектов |
| **Capacity** | Может содержать неограниченное количество контейнеров |

> 💡 Имя Storage Account становится частью публичного URL.

---

# 2️⃣ Containers (Контейнеры)

## Overview (Обзор)

Container — это логическая группировка blob-объектов,  
аналог директории в файловой системе.

Используется для:
- Организации данных
- Управления доступом
- Разделения логических областей хранения

---

## Container Characteristics (Характеристики)

| Characteristic | Details |
|----------------|---------|
| **Number per account** | Неограниченно |
| **Blobs per container** | Неограниченно |
| **Naming** | Должно быть валидным DNS-именем |
| **URL part** | Является частью URI blob |

Пример URI:
        https://<account>.blob.core.windows.net/<container>/<blob>  


---

## Container Naming Rules (Правила именования)

✅ Должны соблюдаться следующие требования:

1. **Длина**: от 3 до 63 символов
2. **Первый символ**: буква или цифра
3. **Допустимые символы**:
    - только строчные буквы (a-z)
    - цифры (0-9)
    - дефис (-)
4. ❌ Нельзя использовать два и более дефиса подряд

---

## Container Naming Examples (Примеры)

| Name | Valid? | Reason |
|------|--------|--------|
| `mycontainer` | ✅ Yes | Только строчные буквы |
| `my-container` | ✅ Yes | Дефис разрешён |
| `my-container-123` | ✅ Yes | Цифры и дефисы допустимы |
| `MyContainer` | ❌ No | Заглавные буквы запрещены |
| `my--container` | ❌ No | Двойной дефис запрещён |
| `-mycontainer` | ❌ No | Нельзя начинать с дефиса |
| `my` | ❌ No | Меньше 3 символов |

---

## Важно для AZ-204

- Container имя должно соответствовать DNS-формату
- Только lowercase
- 3–63 символа
- Нет consecutive dashes
- Container — обязательный уровень между account и blob

### Container URI Format

```
https://<account-name>.blob.core.windows.net/<container-name>
```

**Example:**
```
https://myaccount.blob.core.windows.net/mycontainer
```

---

# 3️⃣ Blobs (Объекты)

## Overview (Обзор)

Blob — это фактический объект данных, который хранится внутри контейнера.

Azure Blob Storage поддерживает **три типа blob**,  
каждый оптимизирован под разные сценарии.

---

## Blob Types Comparison (Сравнение типов)

| Blob Type | Composition | Max Size | Primary Use Case |
|------------|------------|----------|------------------|
| **Block Blobs** | Состоит из блоков | ~190.7 TiB | Общие данные, файлы |
| **Append Blobs** | Блоки для append-операций | ~190.7 TiB | Логи, стриминг |
| **Page Blobs** | Случайный доступ (random access) | 8 TB | VHD, диски VM |

---

# Block Blobs (Блочные объекты)

## Best for:
Хранение текстовых и бинарных данных общего назначения.

---

## Key Features (Особенности)

- Состоит из отдельных блоков
- Каждый блок может быть разного размера
- Поддерживает параллельную загрузку блоков
- Максимальный размер: **~190.7 TiB**
- Поддерживает access tiers (Hot, Cool, Cold, Archive)

> 💡 Самый часто используемый тип blob.

---

## Use Cases (Сценарии использования)

- Документы
- Изображения
- Видео и аудио
- Бэкапы
- Архивы
- Данные приложений
- Любые бинарные и текстовые файлы

---

## Важно для AZ-204

- Block Blob — default и самый распространённый тип
- Поддерживает Archive tier
- Может загружаться по частям (block-by-block)
- Подходит для почти всех обычных сценариев хранения


#### CLI Example
```bash
# Upload file as block blob
az storage blob upload \
  --account-name <account-name> \
  --container-name <container-name> \
  --name <blob-name> \
  --file <local-file-path> \
  --auth-mode login
```

#### PowerShell Example
```powershell
# Upload file as block blob
Set-AzStorageBlobContent `
  -Container <container-name> `
  -File <local-file-path> `
  -Blob <blob-name> `
  -Context $ctx
```

# Append Blobs (Append-объекты)

## Best for:
Операции добавления данных в конец (logging-сценарии).

---

## Key Features (Особенности)

- Состоят из блоков (как Block Blob)
- **Оптимизированы для append-операций**
- Блоки можно добавлять только в конец
- Нельзя изменять или удалять существующие блоки
- Максимальный размер: **~190.7 TiB**

---

## Use Cases (Сценарии использования)

- Логи виртуальных машин
- Логи приложений
- Audit-логи
- Непрерывные потоки данных
- Time-series данные

---

## Key Characteristic (Главная особенность)

⚠️ **Append-only модель**  
После записи блок нельзя изменить или удалить —  
можно только добавить новый блок в конец.

---

## Важно для AZ-204

- Используется в logging-сценариях
- Подходит для сценариев "write-once, append-many"
- Не подходит, если требуется обновление существующих данных
- Структурно похож на Block Blob, но с ограничением append-only


#### CLI Example
```bash
# Create append blob
az storage blob append create \
  --account-name <account-name> \
  --container-name <container-name> \
  --name <blob-name> \
  --auth-mode login

# Append data
az storage blob append upload \
  --account-name <account-name> \
  --container-name <container-name> \
  --name <blob-name> \
  --file <data-file> \
  --auth-mode login
```

# Page Blobs (Страничные объекты)

## Best for:
Сценарии с произвольным доступом (random access), особенно для дисков виртуальных машин.

---

## Key Features (Особенности)

- Хранят **файлы с произвольным доступом**
- Максимальный размер: **8 TB**
- Оптимизированы для случайных операций чтения/записи
- Организованы как набор страниц по **512 байт**
- Основное применение — файлы виртуальных дисков (VHD)

---

## Use Cases (Сценарии использования)

- Диски виртуальных машин (OS и data disks)
- Хранилище Azure VM
- Файлы баз данных
- Любые сценарии с random read/write

---

## Key Characteristic (Главная особенность)

💡 **Random Access модель**  
Предназначены для произвольного чтения/записи,  
в отличие от Block и Append Blob, которые оптимизированы под последовательный доступ.

---

## Важно для AZ-204

- Используются для VHD-дисков
- Максимальный размер — 8 TB
- Страницы фиксированного размера (512 байт)
- Не предназначены для обычного файлового хранения (для этого Block Blob)


#### CLI Example
```bash
# Create page blob (for VHD)
az storage blob upload \
  --account-name <account-name> \
  --container-name <container-name> \
  --name <blob-name> \
  --file <vhd-file-path> \
  --type page \
  --auth-mode login
```

---

## Blob URI Formats

### Simple Blob URI
```
https://<account-name>.blob.core.windows.net/<container-name>/<blob-name>
```

**Example:**
```
https://myaccount.blob.core.windows.net/mycontainer/myblob
```

### Blob with Virtual Directory
```
https://<account-name>.blob.core.windows.net/<container-name>/<virtual-directory>/<blob-name>
```

**Example:**
```
https://myaccount.blob.core.windows.net/mycontainer/myvirtualdirectory/myblob
```

💡 **Virtual Directories**: Blob storage is flat, but you can simulate hierarchical structure using forward slashes (/) in blob names.

---

## Resource Hierarchy Visualization

```
mystorageaccount.blob.core.windows.net (Storage Account)
│
├── images (Container)
│   ├── photo1.jpg (Block Blob)
│   ├── photo2.jpg (Block Blob)
│   └── vacation/photo3.jpg (Block Blob - virtual directory)
│
├── logs (Container)
│   ├── app.log (Append Blob)
│   └── system.log (Append Blob)
│
└── vhds (Container)
    ├── os-disk.vhd (Page Blob)
    └── data-disk.vhd (Page Blob)
```

---

## Key Concepts

### Storage Account Namespace
- **Globally unique** across all Azure
- Forms the base URL for all resources
- Cannot be changed after creation

### Container as Logical Grouping
- Similar to folders/directories
- Organize related blobs
- Apply access policies at container level
- Unlimited number per account

### Blob Type Selection
- **Block Blobs**: Default choice for most scenarios
- **Append Blobs**: When you need append-only operations
- **Page Blobs**: When you need random access (VMs)

---

## Naming Best Practices

### Storage Account Names
✅ **DO:**
- Use 3-24 characters
- Use only lowercase letters and numbers
- Make it globally unique
- Make it descriptive

❌ **DON'T:**
- Use uppercase letters
- Use special characters (except numbers)
- Use names already taken in Azure

### Container Names
✅ **DO:**
- Use 3-63 characters
- Start with letter or number
- Use lowercase letters, numbers, and dashes
- Make it descriptive of contents

❌ **DON'T:**
- Use uppercase letters
- Use consecutive dashes
- Start or end with dash

### Blob Names
✅ **DO:**
- Use descriptive names
- Use forward slashes (/) to simulate directories
- Include file extensions for clarity

❌ **DON'T:**
- Use special characters that need URL encoding unnecessarily

---

## Exam Tips

🎯 **Know the hierarchy**: Storage Account → Container → Blob (3 levels)

🎯 **Container naming**: 3-63 chars, lowercase only, no consecutive dashes

🎯 **Three blob types**: Block (general), Append (logging), Page (VM disks)

🎯 **Block blob size**: ~190.7 TiB (most scenarios)

🎯 **Page blob size**: 8 TB (VM disks)

🎯 **Append blob characteristic**: Append-only, cannot modify existing blocks

🎯 **URI format**: `https://<account>.blob.core.windows.net/<container>/<blob>`

🎯 **Unlimited**: Unlimited containers per account, unlimited blobs per container

🎯 **Virtual directories**: Simulated using forward slashes in blob names

---

## Quick Reference Commands

### Create Container
```bash
# Azure CLI
az storage container create \
  --name <container-name> \
  --account-name <account-name> \
  --auth-mode login

# PowerShell
New-AzStorageContainer `
  -Name <container-name> `
  -Context $ctx
```

### List Containers
```bash
# Azure CLI
az storage container list \
  --account-name <account-name> \
  --auth-mode login

# PowerShell
Get-AzStorageContainer -Context $ctx
```

### Upload Blob
```bash
# Azure CLI (Block Blob - default)
az storage blob upload \
  --account-name <account-name> \
  --container-name <container-name> \
  --name <blob-name> \
  --file <local-file> \
  --auth-mode login

# PowerShell
Set-AzStorageBlobContent `
  -Container <container-name> `
  -File <local-file> `
  -Blob <blob-name> `
  -Context $ctx
```

### List Blobs
```bash
# Azure CLI
az storage blob list \
  --account-name <account-name> \
  --container-name <container-name> \
  --auth-mode login

# PowerShell
Get-AzStorageBlob `
  -Container <container-name> `
  -Context $ctx
```

---
# Comparison Summary (Сводное сравнение)

| Aspect | Storage Account | Container | Blob |
|--------|------------------|------------|------|
| **Purpose** | Верхний уровень, namespace | Логическая группировка | Фактический объект данных |
| **Capacity** | Неограниченное число контейнеров | Неограниченное число blob | До 190.7 TiB (block/append) или 8 TB (page) |
| **Naming Rules** | 3–24 символа, lowercase, цифры | 3–63 символа, lowercase, цифры, дефис | Жёстких правил нет, использовать URL-safe символы |
| **Uniqueness** | Глобально уникально | Уникально внутри аккаунта | Уникально внутри контейнера |
| **URL Format** | `<account>.blob.core.windows.net` | `<account>.blob.core.windows.net/<container>` | `<account>.blob.core.windows.net/<container>/<blob>` |

---

## Важно для AZ-204

- Storage Account — глобально уникальный
- Container — уникален в рамках аккаунта
- Blob — уникален в рамках контейнера
- Иерархия всегда трёхуровневая
- Block и Append поддерживают ~190.7 TiB
- Page Blob ограничен 8 TB


---

## Additional Resources

- [Storage account overview](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview)
- [Container and blob naming](https://learn.microsoft.com/en-us/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata)
- [Understanding block blobs, append blobs, and page blobs](https://learn.microsoft.com/en-us/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs)

[Microsoft Learn - Discover Azure Blob storage resource types](https://learn.microsoft.com/en-us/training/modules/explore-azure-blob-storage/3-blob-storage-resources)
