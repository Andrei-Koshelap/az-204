# Set and Retrieve Properties and Metadata Using REST
(Управление свойствами и метаданными через REST API)

## Overview (Обзор)

Вы можете управлять свойствами и метаданными контейнеров и blob с помощью **Azure Storage REST API**.

В этом разделе рассматриваются:

- Основные REST-операции
- Использование HTTP-заголовков
- Формат передачи metadata

---

## Metadata in REST API (Метаданные в REST API)

### Префикс `x-ms-meta-`

Пользовательские метаданные передаются как **HTTP-заголовки** с префиксом:
    x-ms-meta-


---

### Пример установки metadata (Set Blob Metadata)

```http
PUT https://{account}.blob.core.windows.net/{container}/{blob}?comp=metadata
x-ms-version: 2021-12-02
x-ms-date: Wed, 17 Jan 2026 12:00:00 GMT
Authorization: SharedKey <account>:<signature>

x-ms-meta-department: finance
x-ms-meta-project: migration
x-ms-meta-costcenter: cc123
```

## Важные правила (Metadata в REST API)

- Ключ metadata передаётся после префикса `x-ms-meta-`  
- Ключи **не чувствительны к регистру**  
- Azure сохраняет ключи в **lowercase**  
- Общий размер metadata — до **8 KB**  
- Установка metadata **полностью заменяет** предыдущий набор  

---

## Получение Metadata

```http
GET https://{account}.blob.core.windows.net/{container}/{blob}
```
Metadata возвращаются в response headers:

x-ms-meta-department: finance
x-ms-meta-project: migration

```http
x-ms-meta-name: value
```
Основные REST-операции

| Operation                | HTTP Method | Query Parameter  |
| ------------------------ | ----------- | ---------------- |
| Get Blob Properties      | HEAD        | —                |
| Set Blob Metadata        | PUT         | `?comp=metadata` |
| Get Container Properties | HEAD        | —                |
| Set Container Metadata   | PUT         | `?comp=metadata` |

```http
ETag: "0x8DAF123456789AB"
Last-Modified: Wed, 17 Jan 2026 11:55:00 GMT
x-ms-lease-status: unlocked
x-ms-blob-type: BlockBlob

```
🎯 Экзаменационный момент AZ-204
Metadata передаётся через заголовки x-ms-meta-*
Установка metadata выполняется через PUT ?comp=metadata
Операция полностью заменяет предыдущие значения
Свойства и metadata можно получить через HEAD-запрос


### Metadata Naming Rules

✅ **Valid names (Допустимые имена metadata в REST API):**

- Должны быть корректными HTTP header именами
- Допускаются только буквы, цифры и символ `_`
- Не чувствительны к регистру (Azure сохраняет в lowercase)
- Не могут содержать пробелы или специальные символы

❌ **Invalid names:**
```http
x-ms-meta-cost-center: 12345    ❌ Hyphen in metadata name
x-ms-meta-2ndproject: value     ❌ Starts with number
x-ms-meta-project name: value   ❌ Space in name
```

✅ **Correct format:**
```http
x-ms-meta-costcenter: 12345     ✅
x-ms-meta-projectname: az204    ✅
x-ms-meta-department: engineering ✅
```

---

## Container Operations

### Set Container Metadata (PUT)

**Request:**
```http
PUT https://myaccount.blob.core.windows.net/mycontainer?restype=container&comp=metadata HTTP/1.1
x-ms-version: 2021-06-08
x-ms-date: Fri, 03 Jan 2026 10:00:00 GMT
x-ms-meta-department: engineering
x-ms-meta-project: az204
x-ms-meta-costcenter: 12345
Authorization: SharedKey myaccount:signature
```

**Response:**
```http
HTTP/1.1 200 OK
Date: Fri, 03 Jan 2026 10:00:00 GMT
ETag: "0x8DCB123456789AB"
Last-Modified: Fri, 03 Jan 2026 10:00:00 GMT
x-ms-request-id: 12345678-1234-1234-1234-123456789abc
x-ms-version: 2021-06-08
```

**cURL Example:**
```bash
curl -X PUT "https://myaccount.blob.core.windows.net/mycontainer?restype=container&comp=metadata" \
  -H "x-ms-version: 2021-06-08" \
  -H "x-ms-date: $(date -u +"%a, %d %b %Y %H:%M:%S GMT")" \
  -H "x-ms-meta-department: engineering" \
  -H "x-ms-meta-project: az204" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

### Get Container Properties and Metadata (GET/HEAD)

**Request (GET):**
```http
GET https://myaccount.blob.core.windows.net/mycontainer?restype=container HTTP/1.1
x-ms-version: 2021-06-08
x-ms-date: Fri, 03 Jan 2026 10:00:00 GMT
Authorization: SharedKey myaccount:signature
```

**Response:**
```http
HTTP/1.1 200 OK
Date: Fri, 03 Jan 2026 10:00:00 GMT
ETag: "0x8DCB123456789AB"
Last-Modified: Fri, 03 Jan 2026 10:00:00 GMT
x-ms-meta-department: engineering
x-ms-meta-project: az204
x-ms-meta-costcenter: 12345
x-ms-lease-status: unlocked
x-ms-lease-state: available
x-ms-has-immutability-policy: false
x-ms-has-legal-hold: false
x-ms-request-id: 12345678-1234-1234-1234-123456789abc
x-ms-version: 2021-06-08
Content-Length: 0
```

**Request (HEAD - Recommended for metadata only):**
```http
HEAD https://myaccount.blob.core.windows.net/mycontainer?restype=container HTTP/1.1
x-ms-version: 2021-06-08
x-ms-date: Fri, 03 Jan 2026 10:00:00 GMT
Authorization: SharedKey myaccount:signature
```

HEAD returns the same headers but **no response body** (more efficient).

---

## Blob Operations

### Set Blob Metadata (PUT)

**Request:**
```http
PUT https://myaccount.blob.core.windows.net/mycontainer/myblob.txt?comp=metadata HTTP/1.1
x-ms-version: 2021-06-08
x-ms-date: Fri, 03 Jan 2026 10:00:00 GMT
x-ms-meta-author: Alice
x-ms-meta-version: 1.2
x-ms-meta-classification: public
Authorization: SharedKey myaccount:signature
```

**Response:**
```http
HTTP/1.1 200 OK
Date: Fri, 03 Jan 2026 10:00:00 GMT
ETag: "0x8DCB987654321CD"
Last-Modified: Fri, 03 Jan 2026 10:00:00 GMT
x-ms-request-id: 87654321-4321-4321-4321-876543218765
x-ms-version: 2021-06-08
```

### Get Blob Properties (GET/HEAD)

**Request (HEAD):**
```http
HEAD https://myaccount.blob.core.windows.net/mycontainer/myblob.txt HTTP/1.1
x-ms-version: 2021-06-08
x-ms-date: Fri, 03 Jan 2026 10:00:00 GMT
Authorization: SharedKey myaccount:signature
```

**Response:**
```http
HTTP/1.1 200 OK
Date: Fri, 03 Jan 2026 10:00:00 GMT
Content-Length: 1024
Content-Type: text/plain
Content-Encoding: utf-8
Content-Language: en-US
Cache-Control: max-age=3600
ETag: "0x8DCB987654321CD"
Last-Modified: Fri, 03 Jan 2026 10:00:00 GMT
x-ms-blob-type: BlockBlob
x-ms-lease-status: unlocked
x-ms-lease-state: available
x-ms-meta-author: Alice
x-ms-meta-version: 1.2
x-ms-meta-classification: public
x-ms-request-id: 87654321-4321-4321-4321-876543218765
x-ms-version: 2021-06-08
```

### Set Blob HTTP Headers (PUT)

**Request:**
```http
PUT https://myaccount.blob.core.windows.net/mycontainer/myblob.txt?comp=properties HTTP/1.1
x-ms-version: 2021-06-08
x-ms-date: Fri, 03 Jan 2026 10:00:00 GMT
x-ms-blob-content-type: application/json
x-ms-blob-content-encoding: gzip
x-ms-blob-content-language: en-US
x-ms-blob-cache-control: max-age=7200
x-ms-blob-content-disposition: attachment; filename="data.json"
Authorization: SharedKey myaccount:signature
```

**Response:**
```http
HTTP/1.1 200 OK
Date: Fri, 03 Jan 2026 10:00:00 GMT
ETag: "0x8DCB111222333DD"
Last-Modified: Fri, 03 Jan 2026 10:00:00 GMT
x-ms-request-id: 11112222-3333-4444-5555-666677778888
x-ms-version: 2021-06-08
```

---

## REST API Reference (Справочник REST API)

### Container Operations (Операции с контейнером)

| Operation | Method | URI | Query Parameters |
|------------|--------|-----|------------------|
| **Set Metadata** | PUT | `/{container}` | `?restype=container&comp=metadata` |
| **Get Properties** | GET / HEAD | `/{container}` | `?restype=container` |
| **List Blobs** | GET | `/{container}` | `?restype=container&comp=list` |

---

### Blob Operations (Операции с blob)

| Operation | Method | URI | Query Parameters |
|------------|--------|-----|------------------|
| **Set Metadata** | PUT | `/{container}/{blob}` | `?comp=metadata` |
| **Get Properties** | HEAD | `/{container}/{blob}` | — |
| **Set Properties** | PUT | `/{container}/{blob}` | `?comp=properties` |
| **Get Blob** | GET | `/{container}/{blob}` | — |
| **Put Blob** | PUT | `/{container}/{blob}` | — |

---

## Standard HTTP Headers (Blobs)
(Стандартные HTTP-заголовки для blob)

### Request Headers for Set Blob Properties
(Заголовки запроса для установки свойств blob)

| Header | Purpose | Example |
|----------|----------|----------|
| **x-ms-blob-content-type** | MIME тип | `application/json` |
| **x-ms-blob-content-encoding** | Кодировка | `gzip` |
| **x-ms-blob-content-language** | Язык | `en-US` |
| **x-ms-blob-cache-control** | Поведение кэширования | `max-age=3600` |
| **x-ms-blob-content-disposition** | Поведение скачивания | `attachment; filename="file.pdf"` |
| **x-ms-blob-content-md5** | MD5-хэш | Base64-encoded MD5 |

---

### Response Headers (Ответные заголовки)

| Header | Type | Description |
|----------|------|-------------|
| **Content-Type** | Standard | MIME тип blob |
| **Content-Length** | Standard | Размер в байтах |
| **Content-Encoding** | Standard | Применённая кодировка |
| **Content-Language** | Standard | Язык содержимого |
| **Cache-Control** | Standard | Директивы кэширования |
| **ETag** | Standard | Entity tag (контроль конкуренции) |
| **Last-Modified** | Standard | Время последнего изменения |
| **x-ms-blob-type** | Azure | BlockBlob, AppendBlob, PageBlob |
| **x-ms-lease-status** | Azure | Locked, Unlocked |
| **x-ms-lease-state** | Azure | Available, Leased и др. |
| **x-ms-meta-*** | Azure | Пользовательские metadata |

---

> 🎯 Экзаменационный момент AZ-204:
> - `HEAD` используется для получения свойств без загрузки содержимого
> - `PUT ?comp=metadata` заменяет весь набор metadata
> - HTTP-заголовки управляют Content-Type, Cache-Control и другими свойствами

---

## Authentication Methods

### 1. Shared Key (Account Key)

```http
Authorization: SharedKey myaccount:signature
```

**Signature calculation:**
```
StringToSign = VERB + "\n" +
               Content-Encoding + "\n" +
               Content-Language + "\n" +
               Content-Length + "\n" +
               ... (multiple headers)
               CanonicalizedHeaders +
               CanonicalizedResource

Signature = Base64(HMAC-SHA256(UTF8(StringToSign), Base64Decode(AccountKey)))
```

### 2. Shared Access Signature (SAS)

```http
GET https://myaccount.blob.core.windows.net/mycontainer/myblob.txt?sv=2021-06-08&ss=b&srt=o&sp=r&se=2026-01-03T12:00:00Z&sig=signature
```

### 3. Azure Active Directory (OAuth 2.0)

```http
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJhbGc...
```

---

## cURL Examples

### Set Container Metadata (Azure AD)

```bash
# Get OAuth token
ACCESS_TOKEN=$(az account get-access-token \
  --resource https://storage.azure.com \
  --query accessToken -o tsv)

# Set metadata
curl -X PUT "https://mystorageaccount.blob.core.windows.net/mycontainer?restype=container&comp=metadata" \
  -H "x-ms-version: 2021-06-08" \
  -H "x-ms-date: $(date -u +"%a, %d %b %Y %H:%M:%S GMT")" \
  -H "x-ms-meta-department: engineering" \
  -H "x-ms-meta-project: az204" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -v
```

### Get Container Metadata

```bash
curl -I "https://mystorageaccount.blob.core.windows.net/mycontainer?restype=container" \
  -H "x-ms-version: 2021-06-08" \
  -H "x-ms-date: $(date -u +"%a, %d %b %Y %H:%M:%S GMT")" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

### Set Blob Metadata

```bash
curl -X PUT "https://mystorageaccount.blob.core.windows.net/mycontainer/sample.txt?comp=metadata" \
  -H "x-ms-version: 2021-06-08" \
  -H "x-ms-date: $(date -u +"%a, %d %b %Y %H:%M:%S GMT")" \
  -H "x-ms-meta-author: Alice" \
  -H "x-ms-meta-version: 1.0" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -v
```

### Get Blob Properties

```bash
curl -I "https://mystorageaccount.blob.core.windows.net/mycontainer/sample.txt" \
  -H "x-ms-version: 2021-06-08" \
  -H "x-ms-date: $(date -u +"%a, %d %b %Y %H:%M:%S GMT")" \
  -H "Authorization: Bearer $ACCESS_TOKEN"
```

### Set Blob HTTP Headers

```bash
curl -X PUT "https://mystorageaccount.blob.core.windows.net/mycontainer/sample.txt?comp=properties" \
  -H "x-ms-version: 2021-06-08" \
  -H "x-ms-date: $(date -u +"%a, %d %b %Y %H:%M:%S GMT")" \
  -H "x-ms-blob-content-type: application/json" \
  -H "x-ms-blob-cache-control: max-age=3600" \
  -H "Authorization: Bearer $ACCESS_TOKEN" \
  -v
```

---

## PowerShell Examples

### Set Container Metadata

```powershell
# Get OAuth token
$token = (Get-AzAccessToken -ResourceUrl "https://storage.azure.com").Token

# Set metadata
$uri = "https://mystorageaccount.blob.core.windows.net/mycontainer?restype=container&comp=metadata"
$headers = @{
    "x-ms-version" = "2021-06-08"
    "x-ms-date" = [DateTime]::UtcNow.ToString("R")
    "x-ms-meta-department" = "engineering"
    "x-ms-meta-project" = "az204"
    "Authorization" = "Bearer $token"
}

Invoke-RestMethod -Uri $uri -Method Put -Headers $headers
```

### Get Container Metadata

```powershell
$token = (Get-AzAccessToken -ResourceUrl "https://storage.azure.com").Token

$uri = "https://mystorageaccount.blob.core.windows.net/mycontainer?restype=container"
$headers = @{
    "x-ms-version" = "2021-06-08"
    "x-ms-date" = [DateTime]::UtcNow.ToString("R")
    "Authorization" = "Bearer $token"
}

$response = Invoke-WebRequest -Uri $uri -Method Head -Headers $headers

# Display metadata headers
$response.Headers | Where-Object { $_.Key -like "x-ms-meta-*" }
```

### Set Blob Metadata

```powershell
$token = (Get-AzAccessToken -ResourceUrl "https://storage.azure.com").Token

$uri = "https://mystorageaccount.blob.core.windows.net/mycontainer/sample.txt?comp=metadata"
$headers = @{
    "x-ms-version" = "2021-06-08"
    "x-ms-date" = [DateTime]::UtcNow.ToString("R")
    "x-ms-meta-author" = "Alice"
    "x-ms-meta-version" = "1.0"
    "Authorization" = "Bearer $token"
}

Invoke-RestMethod -Uri $uri -Method Put -Headers $headers
```

### Get Blob Properties

```powershell
$token = (Get-AzAccessToken -ResourceUrl "https://storage.azure.com").Token

$uri = "https://mystorageaccount.blob.core.windows.net/mycontainer/sample.txt"
$headers = @{
    "x-ms-version" = "2021-06-08"
    "x-ms-date" = [DateTime]::UtcNow.ToString("R")
    "Authorization" = "Bearer $token"
}

$response = Invoke-WebRequest -Uri $uri -Method Head -Headers $headers

# Display all headers
$response.Headers | Format-Table -AutoSize

# Display only metadata
$response.Headers | Where-Object { $_.Key -like "x-ms-meta-*" } | Format-Table
```

---
## Response Status Codes (Коды ответов REST API)

| Status | Meaning | Common Causes |
|----------|------------|----------------|
| **200 OK** | Успешно | Операция выполнена успешно |
| **400 Bad Request** | Некорректный запрос | Неверное имя metadata, ошибочный URI |
| **401 Unauthorized** | Ошибка аутентификации | Неверные учётные данные, истёкший токен |
| **403 Forbidden** | Доступ запрещён | Недостаточно прав (отсутствует RBAC-роль) |
| **404 Not Found** | Ресурс не найден | Контейнер или blob не существует |
| **409 Conflict** | Конфликт | Активный lease, конкурентная модификация |
| **412 Precondition Failed** | Условие не выполнено | Ошибка If-Match / If-None-Match |

---

### Важно понимать

- **401** — проблема с аутентификацией
- **403** — аутентификация успешна, но нет прав
- **409** — чаще всего связано с lease или версионированием
- **412** — используется при оптимистичной блокировке (ETag, условия If-Match)

---

> 🎯 Экзаменационный момент AZ-204:
> - 403 = нет RBAC прав
> - 409 = конфликт (lease / конкурентная операция)
> - 412 = условная проверка ETag не прошла


---

## Complete REST Example (Python)

```python
import requests
from datetime import datetime

# Configuration
storage_account = "mystorageaccount"
container_name = "mycontainer"
blob_name = "sample.txt"
access_token = "YOUR_OAUTH_TOKEN"

# Set blob metadata
url = f"https://{storage_account}.blob.core.windows.net/{container_name}/{blob_name}?comp=metadata"
headers = {
    "x-ms-version": "2021-06-08",
    "x-ms-date": datetime.utcnow().strftime("%a, %d %b %Y %H:%M:%S GMT"),
    "x-ms-meta-author": "Alice",
    "x-ms-meta-version": "1.0",
    "x-ms-meta-category": "documentation",
    "Authorization": f"Bearer {access_token}"
}

response = requests.put(url, headers=headers)

if response.status_code == 200:
    print("✓ Metadata set successfully")
    print(f"ETag: {response.headers['ETag']}")
    print(f"Last-Modified: {response.headers['Last-Modified']}")
else:
    print(f"❌ Error: {response.status_code}")
    print(response.text)

# Get blob properties
url = f"https://{storage_account}.blob.core.windows.net/{container_name}/{blob_name}"
headers = {
    "x-ms-version": "2021-06-08",
    "x-ms-date": datetime.utcnow().strftime("%a, %d %b %Y %H:%M:%S GMT"),
    "Authorization": f"Bearer {access_token}"
}

response = requests.head(url, headers=headers)

if response.status_code == 200:
    print("\n✓ Properties retrieved")
    
    # Display metadata
    print("\nMetadata:")
    for key, value in response.headers.items():
        if key.startswith("x-ms-meta-"):
            metadata_name = key.replace("x-ms-meta-", "")
            print(f"  {metadata_name}: {value}")
    
    # Display system properties
    print("\nSystem Properties:")
    print(f"  Content-Type: {response.headers.get('Content-Type')}")
    print(f"  Content-Length: {response.headers.get('Content-Length')}")
    print(f"  ETag: {response.headers.get('ETag')}")
    print(f"  Last-Modified: {response.headers.get('Last-Modified')}")
else:
    print(f"❌ Error: {response.status_code}")
```

---

## Key Differences: REST vs .NET SDK
(Основные различия REST API и .NET SDK)

| Aspect | REST API | .NET SDK |
|----------|------------|------------|
| **Metadata headers** | `x-ms-meta-name: value` | `metadata["name"] = "value"` |
| **Set metadata** | `PUT ?comp=metadata` | `SetMetadataAsync(metadata)` |
| **Get properties** | `HEAD` запрос | `GetPropertiesAsync()` |
| **Authentication** | Ручные заголовки (Authorization, x-ms-date) | Встроенная (`DefaultAzureCredential`) |
| **Error handling** | HTTP status codes | Исключения (`RequestFailedException`) |
| **Date format** | Требуется формат RFC 1123 | Обрабатывается автоматически |
| **Signature** | Ручная генерация HMAC-SHA256 | Генерируется автоматически |

---

### Что важно понимать

- REST требует ручной работы с заголовками и подписью запроса
- SDK инкапсулирует детали аутентификации и подписи
- В SDK ошибки обрабатываются через исключения, а не через HTTP-коды напрямую
- SDK значительно снижает вероятность ошибок в подписи и формате дат

---

> 🎯 Экзаменационный момент AZ-204:
> - REST = полный контроль, но больше ручной работы
> - SDK = абстракция поверх REST
> - `RequestFailedException` — основной тип ошибки в .NET SDK


---

## Best Practices

### 1. Use HEAD for Metadata Retrieval

✅ **Do**: Use HEAD to get metadata without downloading blob
```bash
curl -I "https://account.blob.core.windows.net/container/blob.txt"
```

❌ **Don't**: Use GET unnecessarily (downloads blob body)
```bash
curl "https://account.blob.core.windows.net/container/blob.txt"  # Downloads entire blob
```

### 2. Validate Metadata Names

✅ **Do**: Check metadata name format before sending
```python
import re

def is_valid_metadata_name(name):
    # Must be valid C# identifier
    return re.match(r'^[a-zA-Z_][a-zA-Z0-9_]*$', name) is not None

if is_valid_metadata_name("cost_center"):
    # Proceed
```

❌ **Don't**: Send invalid names (results in 400 Bad Request)
```python
metadata_name = "cost-center"  # Hyphen not allowed
```

### 3. Handle Rate Limits

```python
import time

def set_metadata_with_retry(url, headers, max_retries=3):
    for attempt in range(max_retries):
        response = requests.put(url, headers=headers)
        
        if response.status_code == 200:
            return response
        elif response.status_code == 503:  # Service Unavailable
            wait_time = 2 ** attempt  # Exponential backoff
            print(f"Rate limited. Retrying in {wait_time}s...")
            time.sleep(wait_time)
        else:
            raise Exception(f"Error {response.status_code}: {response.text}")
    
    raise Exception("Max retries exceeded")
```

### 4. Include x-ms-version Header

✅ **Do**: Always specify API version
```http
x-ms-version: 2021-06-08
```

❌ **Don't**: Omit version header (may use older API version)

### 5. Use Conditional Headers for Concurrency

```http
PUT https://account.blob.core.windows.net/container/blob.txt?comp=metadata HTTP/1.1
If-Match: "0x8DCB123456789AB"
x-ms-meta-version: 1.1
```

If ETag doesn't match (concurrent modification), returns **412 Precondition Failed**.

---

## Exam Tips (Советы к экзамену AZ-204)

🎯 **Префикс `x-ms-meta-`**  
Обязателен для передачи metadata через REST API

🎯 **HEAD vs GET**  
`HEAD` используется для получения свойств и metadata без загрузки содержимого blob

🎯 **Query parameters**
- `?comp=metadata` — операции с metadata
- `?comp=properties` — операции со свойствами

🎯 **Операции с контейнером**  
Требуют параметр `?restype=container`

🎯 **`x-ms-version`**  
Обязательный заголовок с указанием версии API  
(например, `2021-06-08`)

🎯 **`x-ms-date`**  
Обязателен при Shared Key аутентификации  
Формат даты — RFC 1123

🎯 **Authorization header**  
Три способа аутентификации:
- SharedKey
- SAS
- Bearer (OAuth / Azure AD)

🎯 **SetMetadata заменяет всё**  
`PUT ?comp=metadata` полностью заменяет metadata (не merge)

🎯 **HTTP-заголовки blob**  
Используются `x-ms-blob-content-type`,  
`x-ms-blob-cache-control` и другие

🎯 **Коды ответа**
- 200 OK — успешно
- 400 — некорректный запрос
- 403 — недостаточно прав
- 404 — ресурс не найден

🎯 **Правила именования metadata**  
Такие же, как в .NET (валидные C#-идентификаторы)

🎯 **Лимит 8 KB**  
Ограничение metadata действует и для REST API


---

## Additional Resources

- [Blob Service REST API](https://learn.microsoft.com/en-us/rest/api/storageservices/blob-service-rest-api)
- [Set Blob Metadata](https://learn.microsoft.com/en-us/rest/api/storageservices/set-blob-metadata)
- [Get Blob Properties](https://learn.microsoft.com/en-us/rest/api/storageservices/get-blob-properties)
- [Set Blob Properties](https://learn.microsoft.com/en-us/rest/api/storageservices/set-blob-properties)
- [Authentication for Azure Storage](https://learn.microsoft.com/en-us/rest/api/storageservices/authorize-requests-to-azure-storage)

[Microsoft Learn - Set and retrieve properties and metadata for blob resources by using REST](https://learn.microsoft.com/en-us/training/modules/work-azure-blob-storage/6-set-retrieve-properties-metadata-rest)
