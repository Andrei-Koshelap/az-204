# Azure Blob Storage Overview

## What is Azure Blob Storage?

**Azure Blob Storage** — это облачное объектное хранилище для хранения больших объёмов **неструктурированных данных**  
(текст, бинарные файлы, изображения, видео и т.д.).

> 💡 Объектное хранилище = данные хранятся как объекты (blob), а не как таблицы или файлы в традиционной файловой системе.

Blob Storage оптимизировано для:
- Масштабируемости
- Высокой доступности
- Хранения больших объёмов данных

---

# Use Cases (Сценарии использования)

| Use Case | Description |
|------------|-------------|
| **Browser Access** | Раздача изображений и документов напрямую в браузер |
| **Distributed Access** | Доступ к файлам из распределённых приложений |
| **Streaming** | Потоковая передача видео и аудио |
| **Logging** | Хранение логов и диагностических данных |
| **Backup / DR** | Резервное копирование и архивирование |
| **Analysis** | Хранение данных для аналитики |

---

# Access Methods (Способы доступа)

- 🌐 **HTTP/HTTPS** — доступ из любой точки мира
- 🔗 **Azure Storage REST API** — прямые REST-запросы
- 🖥 **Azure PowerShell**
- 💻 **Azure CLI**
- 📦 **Client Libraries (SDK)** — .NET, Java, Python, JavaScript и др.

> 🎯 Для production обычно используются SDK или Managed Identity.

---

# Storage Account (Учётная запись хранения)

**Storage Account** — верхний уровень для Blob Storage.

Предоставляет:

- 🌍 **Уникальное пространство имён**
- 🔐 Глобальный доступ через HTTP/HTTPS
- 📍 Базовый адрес для объектов:
             http://<account-name>.blob.core.windows.net

Структура адреса blob:
            https://<account-name>.blob.core.windows.net/<container-name>/<blob-name>

---

## Важно для AZ-204

- Blob Storage предназначен для неструктурированных данных
- Storage Account — корневой контейнер
- Blob доступен через публичный URL (если разрешено)
- Используйте Managed Identity вместо ключей доступа

---

# Types of Storage Accounts (Типы Storage Account)

## Performance Levels (Уровни производительности)

| Performance Level | Description | Use Case |
|-------------------|-------------|----------|
| **Standard** | General-purpose v2 (рекомендуется) | Большинство сценариев |
| **Premium** | SSD-диски, высокая производительность | Высокая нагрузка, низкая задержка |

> 💡 В 90% случаев на экзамене правильный выбор — **Standard general-purpose v2**.

---

# Account Types (Типы аккаунтов)

| Account Type | Supported Services | Redundancy Options | Description |
|--------------|-------------------|-------------------|-------------|
| **Standard general-purpose v2** | Blob (вкл. Data Lake), Queue, Table, Files | LRS, GRS, RA-GRS, ZRS, GZRS, RA-GZRS | Универсальный тип. **Рекомендуется по умолчанию** |
| **Premium block blobs** | Blob (block + append) | LRS, ZRS | Высокие транзакции, маленькие объекты, низкая задержка |
| **Premium file shares** | Azure Files | LRS, ZRS | Высокопроизводительные файловые шары |
| **Premium page blobs** | Page blobs | LRS, ZRS | Используется для дисков виртуальных машин |

---

# Redundancy Options (Варианты отказоустойчивости)

| Option | Full Name | Description |
|--------|-----------|-------------|
| **LRS** | Locally Redundant Storage | 3 копии в одном дата-центре |
| **ZRS** | Zone-Redundant Storage | Репликация в 3 availability zones |
| **GRS** | Geo-Redundant Storage | Репликация в другой регион |
| **RA-GRS** | Read-Access Geo-Redundant Storage | GRS + доступ на чтение во вторичном регионе |
| **GZRS** | Geo-Zone-Redundant Storage | ZRS + георепликация |
| **RA-GZRS** | Read-Access Geo-Zone-Redundant Storage | GZRS + доступ на чтение |

> 🎯 RA-* варианты позволяют читать из secondary региона.

---

# Access Tiers for Block Blob Data (Уровни доступа)

Каждый уровень — компромисс между:
- Стоимостью хранения
- Стоимостью доступа
- Минимальным сроком хранения

---

## Access Tier Comparison

| Tier | Optimization | Minimum Duration | Storage Cost | Access Cost | Use Case |
|------|--------------|-----------------|--------------|-------------|----------|
| **Hot** | Частый доступ | Нет | Высокая | Низкая | Часто используемые данные |
| **Cool** | Редкий доступ | 30 дней | Ниже | Выше | Бэкапы, DR |
| **Cold** | Очень редкий доступ | 90 дней | Ниже Cool | Выше Cool | Баланс между Cool и Archive |
| **Archive** | Долгосрочное хранение | 180 дней | Самая низкая | Самая высокая | Архив |

---

## Tier Characteristics (Характеристики)

### 🔥 Hot
- Самые высокие storage costs
- Самые низкие access costs
- Default tier

---

### ❄️ Cool
- Минимум хранения: 30 дней
- Дешевле хранить, дороже читать

---

### 🧊 Cold
- Минимум хранения: 90 дней
- Ещё дешевле хранение
- Быстрее доступ, чем Archive

---

### 📦 Archive
- Только для **individual block blobs**
- Минимум хранения: 180 дней
- Требуется **rehydration**
- Доступ может занимать часы

---

# Tier Switching (Переключение уровней)

💡 Можно менять tier в любое время.

⚠️ Раннее удаление (< минимального срока) → штраф (early deletion fee).

---

# Key Concepts (Ключевые понятия)

## Unstructured Data (Неструктурированные данные)

- Текстовые файлы
- Бинарные файлы
- Изображения, видео
- Логи
- Резервные копии

---

# Важно для AZ-204

- General-purpose v2 — основной тип аккаунта
- Archive требует rehydration
- Early deletion charges важны
- RA-GRS позволяет читать secondary
- Blob Storage хранит неструктурированные данные

### Storage Account Namespace
Each storage account has a unique namespace:
```
http://<account-name>.blob.core.windows.net
```

Все объекты внутри Storage Account доступны по базовому URL:

https://<account-name>.blob.core.windows.net


Структура полного пути:

https://<account-name>.blob.core.windows.net/<container-name>/<blob-name>
---

# Best Practices (Лучшие практики)

✅ Используйте **Standard general-purpose v2** в большинстве сценариев  
✅ Выбирайте **Premium block blobs** при высокой нагрузке и требованиях к низкой задержке  
✅ Подбирайте **access tier** исходя из частоты доступа  
✅ Используйте **Hot tier** для часто используемых данных  
✅ Используйте **Cool / Cold tier** для редко используемых данных
- Cool — минимум 30 дней
- Cold — минимум 90 дней  
  ✅ Используйте **Archive tier** для долгосрочного хранения (минимум 180 дней)  
  ✅ Переключайте tier при изменении паттернов использования

> 💡 Правильный выбор tier может существенно снизить стоимость хранения.

---

# Exam Tips (Советы для экзамена)

🎯 **Типы аккаунтов**
- Standard general-purpose v2 — основной и самый частый выбор
- Premium block blobs — высокая производительность

🎯 **Access tiers**
- Hot — частый доступ
- Cool — минимум 30 дней
- Cold — минимум 90 дней
- Archive — минимум 180 дней

🎯 **Минимальные сроки хранения**
- Cool = 30 дней
- Cold = 90 дней
- Archive = 180 дней

🎯 **Ограничения Archive**
- Только для individual block blobs
- Требуется rehydration (может занять часы)

🎯 **Cost tradeoff**
- Чем "холоднее" tier → ниже стоимость хранения
- Но выше стоимость доступа

🎯 **Redundancy**
- LRS — локальная репликация
- ZRS — по зонам
- GRS — георепликация
- RA-GRS — георепликация + доступ на чтение

🎯 **Premium vs Standard**
- Premium использует SSD
- Поддерживает только LRS и ZRS
- Предназначен для высокопроизводительных сценариев


---

## Quick Reference Commands

### Create Storage Account (Azure CLI)
```bash
# Standard general-purpose v2
az storage account create \
  --name <account-name> \
  --resource-group <resource-group> \
  --location <location> \
  --sku Standard_LRS \
  --kind StorageV2

# Premium block blobs
az storage account create \
  --name <account-name> \
  --resource-group <resource-group> \
  --location <location> \
  --sku Premium_LRS \
  --kind BlockBlobStorage
```

### Create Storage Account (PowerShell)
```powershell
# Standard general-purpose v2
New-AzStorageAccount `
  -ResourceGroupName <resource-group> `
  -Name <account-name> `
  -Location <location> `
  -SkuName Standard_LRS `
  -Kind StorageV2

# Premium block blobs
New-AzStorageAccount `
  -ResourceGroupName <resource-group> `
  -Name <account-name> `
  -Location <location> `
  -SkuName Premium_LRS `
  -Kind BlockBlobStorage
```

# SKU Names (Имена SKU для Storage Account)

SKU определяет комбинацию:
- Уровня производительности (Standard / Premium)
- Типа отказоустойчивости (LRS, ZRS, GRS и т.д.)

---

## Standard SKU

| SKU Name | Description |
|------------|-------------|
| `Standard_LRS` | Standard + локальная репликация (3 копии в одном дата-центре) |
| `Standard_GRS` | Standard + георепликация во второй регион |
| `Standard_RAGRS` | Standard + георепликация + доступ на чтение secondary |
| `Standard_ZRS` | Standard + репликация по availability zones |
| `Standard_GZRS` | Standard + ZRS + георепликация |
| `Standard_RAGZRS` | Standard + GZRS + доступ на чтение secondary |

---

## Premium SKU

| SKU Name | Description |
|------------|-------------|
| `Premium_LRS` | Premium (SSD) + локальная репликация |
| `Premium_ZRS` | Premium (SSD) + зональная репликация |

---

## Важно для AZ-204

- **Premium поддерживает только LRS и ZRS**
- `RA` означает **Read Access** к secondary региону
- GZRS = ZRS + георепликация
- Standard_LRS — самый простой и дешёвый вариант
- Для высокой доступности между регионами — выбирайте GRS / RA-GRS / GZRS

> 🎯 На экзамене часто спрашивают различие между GRS и RA-GRS.


В Azure Blob Storage есть три типа blob:
1️⃣ Block Blob
для хранения файлов
документов
изображений
видео
используется чаще всего

2️⃣ Page Blob
оптимизирован для random read/write
используется для VHD (виртуальные диски Azure VM)

3️⃣ Append Blob ✅

Специально предназначен для сценариев:
логирования
аудита
трассировки
append-only workloads
Особенности:
данныеможно только добавлять в конец
нельзя изменять существующие блоки
идеально подходит для log-файлов
---


В Azure Blob Storage есть минимальный срок хранения для каждого tier:
Tier	Минимальный срок хранения
Hot	нет
Cool	30 дней
Archive	180 дней
## Additional Resources


Когда в Azure Blob Storage включается Static website hosting, автоматически создаётся специальный контейнер:
$web
После этого контент становится доступным через:
https://<storage-account>.zXX.web.core.windows.net

- [Azure Blob Storage documentation](https://learn.microsoft.com/en-us/azure/storage/blobs/)
- [Storage account overview](https://learn.microsoft.com/en-us/azure/storage/common/storage-account-overview)
- [Access tiers for blob data](https://learn.microsoft.com/en-us/azure/storage/blobs/access-tiers-overview)

[Microsoft Learn - Explore Azure Blob storage](https://learn.microsoft.com/en-us/training/modules/explore-azure-blob-storage/2-blob-storage-overview)

Blob Storage:
Использует PUT для загрузки блоба
Метаданные передаются именно в PUT-запросе
POST здесь не используется для установки метаданных