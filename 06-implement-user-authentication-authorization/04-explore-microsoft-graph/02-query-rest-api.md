# Query Microsoft Graph by using REST

## Ключевые понятия

- **REST API** — HTTP API, построенный по принципам REST
- **Структура запроса** — Метод + Endpoint + Версия + Ресурс + Query-параметры
- **HTTP методы** — GET, POST, PATCH, PUT, DELETE
- **OData** — Open Data Protocol для параметров запросов

---

## Что это означает

Microsoft Graph предоставляет REST-интерфейс, где взаимодействие строится через:

- URL ресурса
- HTTP метод
- Заголовки (включая `Authorization`)
- (Опционально) тело запроса для создания/изменения данных
- OData query-параметры для фильтрации и выборки

---
## Общая структура запроса

```http
{HTTP method} https://graph.microsoft.com/{version}/{resource}?{query-parameters}
```
---
``Где:

- `{version}` — `v1.0` (для production) или `beta` (preview)
- `{resource}` — путь к сущности (например, `users`, `me`, `groups`)
- `{query-params}` — параметры OData (`$select`, `$filter`, `$top` и т.д.)

``
## HTTP методы в Microsoft Graph

- **GET** — чтение данных
- **POST** — создание объекта
- **PATCH** — частичное обновление
- **PUT** — полная замена (используется реже)
- **DELETE** — удаление

---

## OData (Open Data Protocol)

OData используется для управления выборкой данных:

- `$select` — выбрать поля
- `$filter` — фильтрация
- `$orderby` — сортировка
- `$expand` — включить связанные сущности
- `$top` / `$skip` — пагинация

---

## Важно для AZ-204

- Всегда использовать `v1.0` для production.
- Для вызова Graph нужен access token (OAuth 2.0).
- Важно правильно выбирать HTTP метод под операцию.
- OData помогает оптимизировать запросы и уменьшить объём ответа.

> 🎯 Частый экзаменационный вопрос:  
Как получить данные из Microsoft Graph через REST?  
Ответ — отправить HTTP запрос на `https://graph.microsoft.com/v1.0/...` с `Authorization: Bearer <token>`.

## Компоненты REST-запроса к Microsoft Graph

| Компонент | Описание | Пример |
|------------|----------|---------|
| **HTTP Method** | Тип операции | `GET`, `POST`, `PATCH`, `PUT`, `DELETE` |
| **Base URL** | Базовый endpoint Microsoft Graph | `https://graph.microsoft.com` |
| **Version** | Версия API | `v1.0` или `beta` |
| **Resource** | Сущность или коллекция | `me`, `users`, `groups` |
| **Query Parameters** | Дополнительные параметры запроса | `?$select=displayName&$top=10` |

# Важно для AZ-204

- Правильная структура URL — частый экзаменационный сценарий.
- Версия указывается сразу после base URL.
- Query-параметры начинаются с `?`.
- OData-параметры всегда начинаются с `$`.

> 🎯 Экзаменационный момент:  
Как получить 10 пользователей с указанием только displayName?  
Ответ — использовать `GET` + `v1.0/users` + `?$select=displayName&$top=10`
> 
### Example Requests

```http
# Get current user
GET https://graph.microsoft.com/v1.0/me

# Get user's messages
GET https://graph.microsoft.com/v1.0/me/messages

# Get filtered users
GET https://graph.microsoft.com/v1.0/users?$filter=startsWith(displayName,'John')

# Get specific user
GET https://graph.microsoft.com/v1.0/users/user@contoso.com

# Get group members
GET https://graph.microsoft.com/v1.0/groups/{group-id}/members
```

## HTTP Methods

### Обзор методов

| Метод   | Назначение | Тело запроса | Частые коды ответа |
|----------|------------|--------------|---------------------|
| **GET**    | Получение данных (чтение ресурса) | ❌ Нет | 200 OK, 404 Not Found |
| **POST**   | Создание ресурса или выполнение действия | ✅ Да (JSON) | 201 Created, 202 Accepted |
| **PATCH**  | Частичное обновление ресурса | ✅ Да (JSON) | 200 OK, 204 No Content |
| **PUT**    | Полная замена ресурса | ✅ Да (JSON) | 200 OK, 204 No Content |
| **DELETE** | Удаление ресурса | ❌ Нет | 204 No Content, 404 Not Found |

---

### Дополнительные пояснения

#### 🔹 GET
- Используется только для чтения данных.
- Должен быть **идемпотентным** (повторный вызов не меняет состояние).
- Не должен изменять данные на сервере.
- Параметры обычно передаются через query string.

#### 🔹 POST
- Применяется для создания нового ресурса.
- Не является идемпотентным.
- Часто используется для отправки формы или запуска серверной операции.
- `201 Created` — ресурс создан.
- `202 Accepted` — запрос принят в обработку (часто при асинхронной обработке).

#### 🔹 PUT
- Полностью заменяет существующий ресурс.
- Идемпотентный метод.
- Если ресурс не существует — в некоторых API может быть создан (зависит от реализации).

#### 🔹 PATCH
- Частичное обновление ресурса.
- Изменяет только переданные поля.
- Обычно используется для более эффективного обновления данных по сравнению с PUT.

#### 🔹 DELETE
- Удаляет ресурс.
- Идемпотентный (повторный вызов обычно возвращает 404).
- Часто возвращает `204 No Content`.

---

### Важно для AZ-204

- Понимать разницу между **идемпотентными** (GET, PUT, DELETE) и **неидемпотентными** (POST) методами.
- Уметь выбирать корректный HTTP-метод при проектировании REST API.
- Знать, когда использовать `201` vs `202`.
- Понимать различие между `PUT` и `PATCH` в контексте RESTful сервисов.

### 1. GET - Read Data

**Read a single resource**:

```http
GET https://graph.microsoft.com/v1.0/me
Authorization: Bearer {token}
```

**Response**:
```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
  "id": "48d31887-5fad-4d73-a9f5-3c356e68a038",
  "businessPhones": ["+1 555 0100"],
  "displayName": "John Doe",
  "givenName": "John",
  "jobTitle": "Software Engineer",
  "mail": "john@contoso.com",
  "mobilePhone": "+1 555 0101",
  "officeLocation": "Seattle",
  "preferredLanguage": "en-US",
  "surname": "Doe",
  "userPrincipalName": "john@contoso.com"
}
```

**Read a collection**:

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer {token}
```

**Response** (paginated):
```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
  "@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$skiptoken=X'4453707...'",
  "value": [
    {
      "id": "48d31887-5fad-4d73-a9f5-3c356e68a038",
      "displayName": "John Doe",
      "mail": "john@contoso.com"
    },
    {
      "id": "7d54cb02-aab3-4016-9944-56e6adee8787",
      "displayName": "Jane Smith",
      "mail": "jane@contoso.com"
    }
  ]
}
```

### 2. POST - Create or Action

**Create a resource**:

```http
POST https://graph.microsoft.com/v1.0/me/events
Authorization: Bearer {token}
Content-Type: application/json

{
  "subject": "Team Meeting",
  "body": {
    "contentType": "HTML",
    "content": "Discuss Q4 planning"
  },
  "start": {
    "dateTime": "2024-12-15T10:00:00",
    "timeZone": "Pacific Standard Time"
  },
  "end": {
    "dateTime": "2024-12-15T11:00:00",
    "timeZone": "Pacific Standard Time"
  },
  "location": {
    "displayName": "Conference Room A"
  },
  "attendees": [
    {
      "emailAddress": {
        "address": "jane@contoso.com",
        "name": "Jane Smith"
      },
      "type": "required"
    }
  ]
}
```

**Response** (201 Created):
```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('me')/events/$entity",
  "id": "AAMkAGI1...",
  "subject": "Team Meeting",
  "start": { "dateTime": "2024-12-15T10:00:00", "timeZone": "Pacific Standard Time" },
  "end": { "dateTime": "2024-12-15T11:00:00", "timeZone": "Pacific Standard Time" }
}
```

**Perform an action**:

```http
POST https://graph.microsoft.com/v1.0/me/sendMail
Authorization: Bearer {token}
Content-Type: application/json

{
  "message": {
    "subject": "Hello from Microsoft Graph",
    "body": {
      "contentType": "Text",
      "content": "This is a test email sent via Microsoft Graph API."
    },
    "toRecipients": [
      {
        "emailAddress": {
          "address": "recipient@contoso.com"
        }
      }
    ]
  },
  "saveToSentItems": "true"
}
```

**Response** (202 Accepted):
```
202 Accepted
(No body)
```

### 3. PATCH - Partial Update

**Update specific properties**:

```http
PATCH https://graph.microsoft.com/v1.0/me
Authorization: Bearer {token}
Content-Type: application/json

{
  "mobilePhone": "+1 555 0102",
  "officeLocation": "San Francisco"
}
```

**Response** (204 No Content or 200 OK with updated resource):
```
204 No Content
```

**Note**: PATCH is preferred over PUT for updates because it only modifies specified properties.

### 4. PUT - Full Replace

**Replace entire resource** (less common):

```http
PUT https://graph.microsoft.com/v1.0/applications/{id}/logo
Authorization: Bearer {token}
Content-Type: image/jpeg

[Binary image data]
```

**Response**:
```
204 No Content
```

### 5. DELETE - Remove Resource

**Delete a resource**:

```http
DELETE https://graph.microsoft.com/v1.0/me/messages/{message-id}
Authorization: Bearer {token}
```

**Response**:
```
204 No Content
```

**Delete with confirmation**:

```http
DELETE https://graph.microsoft.com/v1.0/groups/{group-id}
Authorization: Bearer {token}
```

## API Versions

### v1.0 - Production

**Stable, generally available**:

```http
GET https://graph.microsoft.com/v1.0/me
```

**Характеристики**:
- ✅ **Отсутствие ломающих изменений (No breaking changes)**
- ✅ **Готовность к использованию в production**
- ✅ **Поддерживается Microsoft**
- ✅ **Полная документация**
- ✅ **Доступно SLA (гарантированный уровень обслуживания)**

---

### Дополнение для AZ-204

В контексте Azure это обычно относится к сервисам со статусом **General Availability (GA)**:

- Интерфейсы и API стабильны.
- Обновления не нарушают обратную совместимость.
- Сервис официально поддерживается.
- Предоставляется SLA (Service Level Agreement) — финансовые гарантии доступности.
- Рекомендуется для production-нагрузки.

Для сравнения:
- **Preview**-версии могут не иметь SLA.
- Возможны breaking changes.
- Не рекомендуется для критичных production-сценариев.


**Use for**: All production applications

### beta - Preview

**Preview features, unstable**:

```http
GET https://graph.microsoft.com/beta/me
```

**Характеристики**:
- ⚠️ **Возможны ломающие изменения (breaking changes)**
- ⚠️ **Только для разработки и тестирования**
- 🔄 **Ранний доступ к новым возможностям**
- 📊 **Возможность предоставить обратную связь**
- ❌ **Отсутствует SLA**

---

### Дополнение для AZ-204

Обычно это относится к сервисам со статусом **Preview** в Azure:

- API может изменяться без сохранения обратной совместимости.
- Поведение сервиса может меняться.
- Нет финансовых гарантий доступности.
- Не рекомендуется для production-среды.
- Часто используется для оценки новых возможностей перед выходом в **General Availability (GA)**.

На экзамене AZ-204 важно помнить:
- Preview → нет SLA.
- GA → есть SLA и стабильный контракт API.


**Use for**: Testing and development only

**Version selection**:

```csharp
// ✅ Good: Production
var user = await graphClient.Me.GetAsync();

// ❌ Bad: Beta in production
// Don't use beta in production apps
```

## Resources and Paths

### Top-Level Resources

```http
# User resources
GET /me                      # Current user
GET /users                   # All users
GET /users/{id}              # Specific user

# Group resources
GET /groups                  # All groups
GET /groups/{id}             # Specific group

# Application resources
GET /applications            # All applications
GET /servicePrincipals       # Service principals

# Organization resources
GET /organization            # Organization details
GET /domains                 # Domains
```

### Relationships and Navigation

**Navigate entity relationships**:

```http
# User's manager
GET /me/manager

# User's direct reports
GET /me/directReports

# User's messages
GET /me/messages

# User's calendar
GET /me/calendar

# User's drive
GET /me/drive

# Group members
GET /groups/{id}/members

# Group owners
GET /groups/{id}/owners
```

### Resource IDs

**Three ways to reference resources**:

```http
# 1. By ID (GUID)
GET /users/48d31887-5fad-4d73-a9f5-3c356e68a038

# 2. By user principal name
GET /users/john@contoso.com

# 3. Relative path (me = current user)
GET /me
```

## Query Parameters (Параметры запроса)

### Системные параметры запроса OData

| Параметр | Назначение | Пример |
|-----------|------------|---------|
| **$select** | Выбор конкретных свойств | `?$select=displayName,mail` |
| **$filter** | Фильтрация результатов | `?$filter=startsWith(displayName,'J')` |
| **$orderby** | Сортировка результатов | `?$orderby=displayName` |
| **$expand** | Загрузка связанных сущностей | `?$expand=manager` |
| **$top** | Ограничение количества результатов | `?$top=10` |
| **$skip** | Пропуск указанного количества записей | `?$skip=10` |
| **$count** | Добавить общее количество записей в ответ | `?$count=true` |
| **$search** | Полнотекстовый поиск | `?$search="displayName:John"` |

---

### Дополнение для AZ-204

OData активно используется в:
- Microsoft Graph
- Azure Data Services
- некоторых REST API Azure

Важно понимать:

- `$select` уменьшает размер ответа (оптимизация трафика).
- `$filter` выполняется на стороне сервера.
- `$top` + `$skip` используются для пагинации.
- `$count=true` возвращает общее количество элементов (полезно для UI-пагинации).
- `$expand` позволяет избежать дополнительных запросов (аналог join).

На экзамене часто проверяют понимание:
- серверной фильтрации,
- оптимизации объёма данных,
- правильного построения REST-запросов.


### 1. $select - Choose Properties

**Return only specific fields**:

```http
GET /me?$select=displayName,mail,jobTitle
```

**Response** (only requested properties):
```json
{
  "displayName": "John Doe",
  "mail": "john@contoso.com",
  "jobTitle": "Software Engineer"
}
```

### 2. $filter - Filter Results

**Filter by condition**:

```http
# Starts with
GET /users?$filter=startsWith(displayName,'John')

# Equals
GET /me/messages?$filter=from/emailAddress/address eq 'sender@contoso.com'

# Greater than
GET /me/messages?$filter=receivedDateTime gt 2024-01-01

# Multiple conditions
GET /users?$filter=startsWith(displayName,'J') and department eq 'Sales'
```

**Filter operators**:
```
eq      # Equal
ne      # Not equal
gt      # Greater than
ge      # Greater than or equal
lt      # Less than
le      # Less than or equal
and     # Logical AND
or      # Logical OR
not     # Logical NOT
```

### 3. $orderby - Sort Results

**Sort by property**:

```http
# Ascending (default)
GET /users?$orderby=displayName

# Descending
GET /users?$orderby=displayName desc

# Multiple properties
GET /users?$orderby=department,displayName
```

### 4. $expand - Include Related Entities

**Include related data**:

```http
# Include manager
GET /me?$expand=manager

# Include direct reports
GET /users/{id}?$expand=directReports

# Include members (for groups)
GET /groups/{id}?$expand=members
```

**Response** (manager included):
```json
{
  "id": "48d31887-5fad-4d73-a9f5-3c356e68a038",
  "displayName": "John Doe",
  "manager": {
    "id": "7d54cb02-aab3-4016-9944-56e6adee8787",
    "displayName": "Jane Smith",
    "jobTitle": "Engineering Manager"
  }
}
```

### 5. $top - Limit Results

**Return first N results**:

```http
GET /users?$top=10
GET /me/messages?$top=25
```

### 6. $skip - Pagination

**Skip N results**:

```http
# Page 1 (results 1-10)
GET /users?$top=10

# Page 2 (results 11-20)
GET /users?$top=10&$skip=10

# Page 3 (results 21-30)
GET /users?$top=10&$skip=20
```

### 7. $count - Include Total Count

**Get total count**:

```http
GET /users?$count=true
```

**Response**:
```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
  "@odata.count": 142,
  "value": [...]
}
```

### 8. $search - Full-Text Search

**Search across properties**:

```http
GET /users?$search="displayName:John"
GET /me/messages?$search="subject:meeting"
```

### Combining Query Parameters

**Use multiple options**:

```http
GET /users?$select=displayName,mail&$filter=startsWith(displayName,'J')&$orderby=displayName&$top=10
```

## Response Format

### Successful Response

**Status codes**:
```
200 OK              # GET, PATCH (with body)
201 Created         # POST (resource created)
202 Accepted        # POST (action accepted)
204 No Content      # DELETE, PATCH (no body)
```

### Error Response

**Status codes**:
```
400 Bad Request     # Invalid request
401 Unauthorized    # Missing/invalid token
403 Forbidden       # Insufficient permissions
404 Not Found       # Resource doesn't exist
429 Too Many Requests  # Rate limit exceeded
500 Internal Server Error  # Server error
```

**Error response body**:
```json
{
  "error": {
    "code": "InvalidAuthenticationToken",
    "message": "Access token has expired.",
    "innerError": {
      "request-id": "b31e5fad-4d73-a9f5-3c356e68a038",
      "date": "2024-01-15T10:00:00"
    }
  }
}
```

## Pagination

**Handle large result sets**:

```http
GET /users
```

**First page response**:
```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
  "@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$skiptoken=X'4453707...'",
  "value": [
    { "id": "...", "displayName": "User 1" },
    { "id": "...", "displayName": "User 2" }
  ]
}
```

**Follow `@odata.nextLink`** to get next page:

```http
GET https://graph.microsoft.com/v1.0/users?$skiptoken=X'4453707...'
```

**Last page** (no `@odata.nextLink`):
```json
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users",
  "value": [
    { "id": "...", "displayName": "Last User" }
  ]
}
```

## Testing Tools

### 1. Graph Explorer

**Interactive web tool**:

```
URL: https://developer.microsoft.com/graph/graph-explorer
```

**Features**:
- Test requests without writing code
- Sign in with Microsoft account
- View sample queries
- See formatted responses
- Generate code snippets

### 2. Postman

**API development tool**:

```
URL: https://www.getpostman.com/
```

**Setup**:
1. Import Microsoft Graph collection
2. Configure OAuth 2.0
3. Send requests
4. Save collections

### 3. cURL

**Command-line tool**:

```bash
# Get access token first
TOKEN="eyJ0eXAiOiJKV1QiLCJub..."

# Make request
curl -X GET \
  'https://graph.microsoft.com/v1.0/me' \
  -H 'Authorization: Bearer '$TOKEN
```

## Critical Notes (Ключевые моменты)

- 💡 **Формат запроса** – `{Method} https://graph.microsoft.com/{version}/{resource}?{params}`
- 🎯 **HTTP-методы** – GET (чтение), POST (создание/действие), PATCH (обновление), PUT (полная замена), DELETE (удаление)
- ✅ **Версии API** – v1.0 (стабильная, production), beta (preview, нестабильная)
- ⚠️ **CRUD-методы** – GET и DELETE не требуют тела запроса
- 🔄 **POST/PATCH/PUT** – требуют JSON-тело запроса
- 📊 **Query-параметры** – $select, $filter, $orderby, $expand, $top, $skip, $count, $search
- 💡 **Пагинация** – использовать `@odata.nextLink` для получения следующей страницы
- ✅ **Обработка ошибок** – проверять коды статуса (401, 403, 404, 429, 500)
- ⚠️ **Ограничение запросов (Rate limiting)** – 429 Too Many Requests при превышении лимитов
- 🔒 **Authorization header** – требуется Bearer token во всех запросах

---

## Exam Tips (Советы к экзамену AZ-204)

- Структура запроса: `{HTTP method} + https://graph.microsoft.com + {version} + {resource} + {query params}`
- HTTP-методы:
    - GET – чтение
    - POST – создание
    - PATCH – обновление
    - PUT – полная замена
    - DELETE – удаление
- GET и DELETE → без тела запроса
- POST, PATCH, PUT → JSON-тело с данными ресурса
- Версии:
    - v1.0 → production, стабильная
    - beta → preview, только для разработки
- Для production-приложений всегда использовать **v1.0**
- OData-параметры:
    - `$select` – вернуть только указанные свойства (уменьшает размер ответа)
    - `$filter` – фильтрация по условию (eq, ne, gt, lt, startsWith и др.)
    - `$orderby` – сортировка (по умолчанию по возрастанию, `desc` – по убыванию)
    - `$expand` – включить связанные сущности
    - `$top` – ограничить количество записей
    - `$skip` – пропустить первые N записей
- Пагинация – использовать `@odata.nextLink` из ответа
- Основные коды ответа:
    - 200 OK
    - 201 Created
    - 204 No Content
    - 400 Bad Request
    - 401 Unauthorized
    - 403 Forbidden
    - 404 Not Found
    - 429 Too Many Requests
- Ошибка в ответе содержит объект `error` с полями `code`, `message`, `innerError`
- Graph Explorer – интерактивный инструмент для тестирования запросов: https://developer.microsoft.com/graph/graph-explorer
- Авторизация – Bearer token в заголовке `Authorization`

---

### Дополнение от себя (что часто спрашивают)

- При получении 429 важно обрабатывать заголовок `Retry-After`.
- Для production желательно реализовывать retry-политику (exponential backoff).
- Не забывать про принцип least privilege при настройке разрешений (Permissions).
- Делать `$select`, чтобы уменьшить payload — это часто фигурирует в вопросах на оптимизацию.


[Learn More](https://learn.microsoft.com/en-us/training/modules/microsoft-graph/3-microsoft-graph-api)
