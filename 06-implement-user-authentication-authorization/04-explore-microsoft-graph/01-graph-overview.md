# Microsoft Graph Overview

## Ключевые понятия

- **Microsoft Graph** — единый API для доступа к данным и аналитике Microsoft 365
- **Единая точка входа** — `https://graph.microsoft.com`
- **RESTful API** — использует стандартные HTTP-методы (GET, POST, PATCH, PUT, DELETE)
- **Три компонента** — Graph API, Graph Connectors, Graph Data Connect

---

# Что такое Microsoft Graph?

**Microsoft Graph — это единая точка доступа к данным и интеллекту Microsoft 365.**

Он предоставляет унифицированную модель программирования для работы с данными из:

- **Microsoft 365** — пользователи, почта, календари, файлы, Teams
- **Windows** — информация об устройствах и активности
- **Enterprise Mobility + Security** — идентификация, политики доступа

---

## Назначение

Предоставляет доступ к облачным ресурсам Microsoft через **единый endpoint**:

- Унифицированный API для различных сервисов Microsoft
- Единая модель аутентификации и авторизации
- Поддержка связей между объектами (пользователь → почта → файлы → группы)
- Поддержка уведомлений в реальном времени

---

## Архитектурная идея

Вместо работы с разными API (Exchange API, SharePoint API и т.д.)  
используется **единый REST endpoint**, который объединяет их.

---

## Важно для AZ-204

- Microsoft Graph — основной API для доступа к данным Microsoft 365.
- Использует OAuth 2.0 и Microsoft Entra ID.
- Поддерживает как delegated, так и application permissions.
- Работает через `https://graph.microsoft.com`.

> 🎯 Частый экзаменационный вопрос:  
> Как получить доступ к данным Microsoft 365 программно?  
> Ответ — использовать Microsoft Graph API.

## Microsoft Graph Components

### 1. Microsoft Graph API

**Primary interface** for accessing data:

```
Endpoint: https://graph.microsoft.com
```

**Key features**:
- Single endpoint for all Microsoft 365 services
- REST APIs and SDKs available
- Identity and access management
- Security and compliance services

**Example**:
```http
GET https://graph.microsoft.com/v1.0/me
GET https://graph.microsoft.com/v1.0/users
GET https://graph.microsoft.com/v1.0/groups
GET https://graph.microsoft.com/v1.0/me/messages
GET https://graph.microsoft.com/v1.0/me/drive/root/children
```

### 2. Microsoft Graph Connectors

**Ingest external data** into Microsoft 365:

```
External Data → Graph Connectors → Microsoft Graph → Microsoft 365
```

## Microsoft Graph Connectors

### Назначение

Позволяют интегрировать **внешние источники данных** в экосистему Microsoft 365.

> 💡 Данные из сторонних систем становятся доступны через Microsoft Search и другие сервисы Microsoft.

---

## Популярные коннекторы

- Box
- Google Drive
- Jira
- Salesforce
- ServiceNow
- Confluence

---

## Сценарии использования

- Единый поиск по внутренним и внешним данным
- Расширение возможностей Microsoft Search
- Поиск и обнаружение контента между различными платформами

---

## Что это даёт

- Централизованный доступ к данным из разных систем
- Улучшенный пользовательский опыт
- Повышение продуктивности за счёт единого поиска

---

## Важно для AZ-204

- Graph Connectors интегрируют внешние данные в Microsoft 365.
- Используются для расширения поиска.
- Не заменяют Graph API, а дополняют его.

> 🎯 Частый экзаменационный вопрос:  
> Как включить данные сторонней системы в Microsoft Search?  
> Ответ — использовать Microsoft Graph Connectors.

### 3. Microsoft Graph Data Connect

**Bulk data access** to Azure:

```
Microsoft Graph → Data Connect → Azure Storage → Analytics Tools
```
## Microsoft Graph Data Connect

### Назначение

Позволяет **масштабно выгружать данные Microsoft 365 в Azure-хранилища** для аналитики и обработки.

> 💡 В отличие от Microsoft Graph API (операционные запросы), Data Connect предназначен для больших объёмов данных и аналитических сценариев.

---

## Основные возможности

- Безопасная и масштабируемая доставка данных
- Интеграция с Azure Synapse Analytics
- Использование Azure Data Factory для построения пайплайнов
- Кэширование данных для аналитических задач

---

## Типовые сценарии

- Машинное обучение на данных Microsoft 365
- Продвинутая аналитика и отчётность
- Построение хранилищ данных (Data Warehouse)
- Резервное копирование и архивирование

---

## Отличие от Graph API

| Microsoft Graph API | Graph Data Connect |
|----------------------|-------------------|
| Операционные запросы | Массовая выгрузка данных |
| REST-запросы | Batch-передача в Azure |
| Реальное время | Аналитические сценарии |
| Ограничения по throttling | Оптимизирован для Big Data |

---

## Важно для AZ-204

- Graph Data Connect используется для аналитики и ML.
- Предназначен для больших объёмов данных.
- Интегрируется с Azure Synapse и Data Factory.
- Не предназначен для real-time API-запросов.

> 🎯 Частый экзаменационный вопрос:  
> Как выгрузить большие объёмы данных Microsoft 365 в Azure для аналитики?  
> Ответ — использовать Microsoft Graph Data Connect.

## Architecture Diagram

```
┌─────────────────────────────────────────────────────┐
│                 Your Application                    │
│                                                     │
│  ┌──────────────┐              ┌──────────────┐   │
│  │  Web App     │              │  Mobile App  │   │
│  └──────┬───────┘              └──────┬───────┘   │
│         │                              │           │
└─────────┼──────────────────────────────┼───────────┘
          │                              │
          └──────────────┬───────────────┘
                         │
         ┌───────────────▼────────────────┐
         │    Microsoft Graph API         │
         │   https://graph.microsoft.com  │
         └───────────────┬────────────────┘
                         │
         ┌───────────────▼────────────────────────────┐
         │        Microsoft 365 Platform              │
         │                                            │
         │  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
         │  │  Users   │  │  Email   │  │  Files  │ │
         │  └──────────┘  └──────────┘  └─────────┘ │
         │                                            │
         │  ┌──────────┐  ┌──────────┐  ┌─────────┐ │
         │  │ Calendar │  │  Teams   │  │ OneDrive│ │
         │  └──────────┘  └──────────┘  └─────────┘ │
         └────────────────────────────────────────────┘
```

## What You Can Access with Microsoft Graph

### User Data

```http
GET /me                          # My profile
GET /me/messages                 # My emails
GET /me/calendar/events          # My calendar
GET /me/contacts                 # My contacts
GET /me/drive/root/children      # My files
GET /me/photo/$value             # My photo
GET /me/memberOf                 # My groups
```

### Organization Data

```http
GET /users                       # All users
GET /groups                      # All groups
GET /applications                # All applications
GET /domains                     # All domains
GET /organization                # Organization details
GET /directoryRoles              # Directory roles
```

### Communication & Collaboration

```http
GET /me/mailFolders              # Mail folders
GET /me/events                   # Calendar events
GET /me/chats                    # Teams chats
GET /teams                       # Teams
GET /sites                       # SharePoint sites
```

### Files & Content

```http
GET /drives                      # OneDrive and SharePoint drives
GET /me/drive                    # My OneDrive
GET /sites/{site-id}/drive       # SharePoint site drive
GET /groups/{group-id}/drive     # Group drive
```

### Identity & Access

```http
GET /identity/conditionalAccess  # Conditional Access policies
GET /servicePrincipals           # Service principals
GET /oauth2PermissionGrants      # Permission grants
```

### Security & Compliance

```http
GET /security/alerts             # Security alerts
GET /auditLogs/directoryAudits   # Audit logs
GET /identityGovernance          # Identity governance
```

## Microsoft Graph API Versions

### v1.0 (Production)

**Generally available APIs**:

```
https://graph.microsoft.com/v1.0/...
```

## Характеристики (Microsoft Graph v1.0)

- ✅ **Стабильная версия** — без breaking changes
- ✅ **Готова к продакшену** — рекомендуется для production-приложений
- ✅ **Поддерживается Microsoft** — доступна официальная поддержка
- ✅ **Полностью документирована** — актуальная и завершённая документация

---

## Что это означает

- Контракты API не меняются неожиданно.
- Подходит для бизнес-критичных приложений.
- Можно рассчитывать на долгосрочную поддержку.
- Документация соответствует реальному поведению API.

---

## Важно для AZ-204

- Для production следует использовать **v1.0**, а не beta.
- Версия указывается в URL:  
  `https://graph.microsoft.com/v1.0/...`
- Beta-версия может содержать изменения и не предназначена для продакшена.

> 🎯 Экзаменационный момент:  
> Какую версию Microsoft Graph использовать в production?  
> Ответ — v1.0.


**Example**:
```http
GET https://graph.microsoft.com/v1.0/me
```

### beta (Preview)

**APIs in preview**:

```
https://graph.microsoft.com/beta/...
```

**Characteristics**:
- ⚠️ **Unstable** - May have breaking changes
- ⚠️ **Development only** - Don't use in production
- 🔄 **Latest features** - Access new functionality
- 📊 **Testing** - Provide feedback to Microsoft

**Example**:
```http
GET https://graph.microsoft.com/beta/me
```

**Best practice**:
```csharp
// ✅ Good: Use v1.0 for production
var user = await graphClient.Me.GetAsync();

// ❌ Bad: Use beta in production
// var user = await graphClient.Beta.Me.GetAsync();
```

## Authentication and Permissions

### OAuth 2.0 / OpenID Connect

**Microsoft Graph uses OAuth 2.0** for authentication:

```http
# Get access token
POST https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
&client_id=<client-id>
&scope=https://graph.microsoft.com/User.Read
&code=<authorization-code>
&redirect_uri=<redirect-uri>
&client_secret=<client-secret>
```

## Типы разрешений (Permission Types)

| Тип | Описание | Пример Scope |
|------|----------|--------------|
| **Delegated** | Пользователь присутствует, приложение действует от имени пользователя | `User.Read` |
| **Application** | Пользователь отсутствует, приложение действует от своего имени | `User.Read.All` |

---

## Delegated Permissions

- Требуется вошедший пользователь.
- Используются в web, mobile, SPA приложениях.
- Эффективные права = пересечение прав пользователя и приложения.
- Требуется user consent или admin consent (в зависимости от разрешения).

---

## Application Permissions

- Используются в daemon, background services.
- Пользователь не участвует.
- Требуется admin consent.
- Приложение получает полный объём разрешённых прав.

---

## Важно для AZ-204

- Delegated → пользователь есть.
- Application → пользователь отсутствует.
- Delegated permissions часто используются с Authorization Code Flow.
- Application permissions используются с Client Credentials Flow.
- `User.Read` — delegated.
- `User.Read.All` — application (требует admin consent).

> 🎯 Экзаменационный момент:  
> Какое разрешение использовать для background-сервиса без пользователя?  
> Ответ — Application permission.

### Common Permissions

```
User.Read                    # Read signed-in user profile
User.ReadWrite              # Read and write user profile
Mail.Read                   # Read user mail
Mail.Send                   # Send mail as user
Calendars.Read              # Read calendars
Files.Read                  # Read files
Group.Read.All              # Read all groups
Directory.Read.All          # Read directory data (admin)
```

## OData Support

**Microsoft Graph implements OData** (Open Data Protocol):

### Query Options

```http
# Select specific properties
GET /me?$select=displayName,mail

# Filter results
GET /users?$filter=startsWith(displayName,'John')

# Order results
GET /users?$orderby=displayName

# Expand related entities
GET /me?$expand=manager

# Top N results
GET /users?$top=10

# Skip results (pagination)
GET /users?$skip=10

# Count results
GET /users?$count=true

# Search
GET /users?$search="displayName:John"
```

### Combining Options

```http
GET /users?$select=displayName,mail&$filter=startsWith(displayName,'A')&$orderby=displayName&$top=10
```

## Microsoft Graph Explorer

**Interactive tool** to test Graph API:

```
URL: https://developer.microsoft.com/graph/graph-explorer
```

## Microsoft Graph Explorer

### Возможности

- Тестирование API-запросов без написания кода
- Просмотр примеров запросов и ответов
- Аутентификация под своей учётной записью
- Изучение документации API
- Генерация готовых code snippets для разных языков

---

## Пример рабочего процесса

1. Перейти в **Graph Explorer**
2. Выполнить вход с Microsoft-аккаунтом
3. Выбрать пример запроса или написать свой
4. Нажать **Run query**
5. Просмотреть ответ сервера
6. Скопировать сгенерированный код для нужного языка

---

## Зачем использовать

- Быстрая проверка разрешений (scopes)
- Отладка запросов к Microsoft Graph
- Изучение структуры JSON-ответов
- Проверка формата URL и параметров

---

## Важно для AZ-204

- Graph Explorer полезен для тестирования delegated permissions.
- Требует согласия (consent) на запрашиваемые разрешения.
- Можно увидеть реальный HTTP-запрос и ответ.
- Удобен для изучения структуры Graph API.

> 🎯 Частый экзаменационный вопрос:  
> Как быстро протестировать Microsoft Graph API без написания приложения?  
> Ответ — использовать Microsoft Graph Explorer.


## Metadata

**Microsoft Graph API metadata**:

```
https://graph.microsoft.com/v1.0/$metadata
https://graph.microsoft.com/beta/$metadata
```

## OData и пространство имён Microsoft Graph

### Namespace

`microsoft.graph`

Все сущности Microsoft Graph находятся в этом пространстве имён.

---

## Для чего используется

- Генерация строго типизированных клиентов (strongly-typed clients)
- Понимание связей между сущностями
- Обнаружение доступных операций
- Валидация запросов

---

## Что это означает на практике

Microsoft Graph построен на **OData-модели**, что позволяет:

- Использовать стандартные OData-параметры (`$select`, `$filter`, `$expand`, `$orderby`)
- Получать метаданные сервиса
- Работать с навигационными свойствами (relationships)

---

## Примеры возможностей OData

- Выбрать только нужные поля
- Фильтровать данные
- Сортировать результаты
- Разворачивать связанные сущности

---

## Важно для AZ-204

- Microsoft Graph основан на OData.
- Namespace: `microsoft.graph`.
- Поддерживает стандартные OData query parameters.
- Позволяет строить оптимизированные запросы.

> 🎯 Экзаменационный момент:  
> Как ограничить возвращаемые поля в Microsoft Graph?  
> Ответ — использовать OData параметр `$select`.

## Benefits of Microsoft Graph

### 1. Single Endpoint

**Access all Microsoft 365 services** through one API:

```csharp
// One client for everything
var graphClient = new GraphServiceClient(credential);

// Access different services
var user = await graphClient.Me.GetAsync();
var messages = await graphClient.Me.Messages.GetAsync();
var events = await graphClient.Me.Events.GetAsync();
var files = await graphClient.Me.Drive.Root.Children.GetAsync();
```

### 2. Consistent Authentication

**One authentication mechanism** for all services:

```csharp
// Authenticate once
var credential = new ClientSecretCredential(tenantId, clientId, clientSecret);
var graphClient = new GraphServiceClient(credential);

// Access all services with same credential
```

## 3️⃣ Unified Developer Experience

### Основные принципы

- **Единые API-паттерны** во всех сервисах
- **SDK для разных языков** (.NET, JavaScript, Java, Python и др.)
- **Подробная документация и примеры**
- **Типобезопасные модели сущностей**

---

## Что это даёт разработчику

- Одинаковая логика работы с пользователями, файлами, почтой, группами и др.
- Снижение времени на изучение отдельных API.
- Возможность использовать официальные SDK вместо ручной работы с HTTP.
- Автоматическую сериализацию и десериализацию объектов.

---

## SDK поддерживаются для

- .NET
- JavaScript / TypeScript
- Java
- Python
- Go (preview)
- PowerShell

---

## Преимущества SDK

- Автоматическая работа с токенами (через MSAL)
- Поддержка pagination
- Обработка throttling
- Strongly-typed модели вместо "сырых" JSON

---

## Важно для AZ-204

- Microsoft Graph предоставляет унифицированный API.
- Использование SDK упрощает разработку.
- Типобезопасные модели снижают ошибки.
- Рекомендуется использовать официальные SDK вместо ручных REST-запросов.

> 🎯 Экзаменационный момент:  
> Как упростить работу с Microsoft Graph в .NET-приложении?  
> Ответ — использовать официальный Microsoft Graph SDK.


### 4. Rich Data Relationships

**Navigate entity relationships**:

```http
# Get user's manager
GET /me/manager

# Get user's direct reports
GET /me/directReports

# Get group members
GET /groups/{id}/members

# Get file sharing links
GET /drives/{drive-id}/items/{item-id}/permissions
```

### 5. Delta Queries

**Track changes over time**:

```http
# Initial query
GET /me/messages/delta

# Later query with delta token
GET /me/messages/delta?$deltatoken={token}
```

### 6. Webhooks / Change Notifications

**Real-time updates**:

```http
# Subscribe to changes
POST /subscriptions
{
  "changeType": "created,updated",
  "notificationUrl": "https://myapp.com/notifications",
  "resource": "/me/messages",
  "expirationDateTime": "2024-12-31T18:00:00Z"
}
```

### 7. Batching

**Combine multiple requests**:

```http
POST /$batch
{
  "requests": [
    { "id": "1", "method": "GET", "url": "/me" },
    { "id": "2", "method": "GET", "url": "/me/messages?$top=5" },
    { "id": "3", "method": "GET", "url": "/me/events?$top=5" }
  ]
}
```

## Common Use Cases

### 1. User Profile Management

```csharp
// Get current user
var me = await graphClient.Me.GetAsync();
Console.WriteLine($"Hello, {me.DisplayName}!");

// Update user
var user = new User { MobilePhone = "+1 555-0123" };
await graphClient.Me.PatchAsync(user);
```

### 2. Email Operations

```csharp
// Read emails
var messages = await graphClient.Me.Messages
    .GetAsync(config => config.QueryParameters.Top = 10);

// Send email
var message = new Message
{
    Subject = "Hello from Graph",
    Body = new ItemBody { Content = "Email body" },
    ToRecipients = new[] { 
        new Recipient { EmailAddress = new EmailAddress { Address = "user@contoso.com" } }
    }
};
await graphClient.Me.SendMail.PostAsync(new SendMailPostRequestBody { Message = message });
```

### 3. Calendar Management

```csharp
// Get calendar events
var events = await graphClient.Me.Events.GetAsync();

// Create event
var newEvent = new Event
{
    Subject = "Team Meeting",
    Start = new DateTimeTimeZone { DateTime = "2024-12-15T10:00:00", TimeZone = "UTC" },
    End = new DateTimeTimeZone { DateTime = "2024-12-15T11:00:00", TimeZone = "UTC" }
};
await graphClient.Me.Events.PostAsync(newEvent);
```

### 4. File Operations

```csharp
// List files
var driveItems = await graphClient.Me.Drive.Root.Children.GetAsync();

// Upload file
using var fileStream = File.OpenRead("document.pdf");
await graphClient.Me.Drive.Root.ItemWithPath("document.pdf").Content.PutAsync(fileStream);

// Download file
var downloadStream = await graphClient.Me.Drive.Items["item-id"].Content.GetAsync();
```

### 5. Teams Collaboration

```csharp
// List teams
var teams = await graphClient.Me.JoinedTeams.GetAsync();

// Send Teams message
var chatMessage = new ChatMessage
{
    Body = new ItemBody { Content = "Hello from Graph API!" }
};
await graphClient.Teams["team-id"].Channels["channel-id"].Messages.PostAsync(chatMessage);
```

# Critical Notes

- 💡 **Единый endpoint** — `https://graph.microsoft.com` для всех сервисов Microsoft 365
- 🎯 **Три компонента** — Graph API, Graph Connectors (внешние данные), Graph Data Connect (массовая выгрузка в Azure)
- ✅ **Две версии** — v1.0 (стабильная для production) и beta (preview, нестабильная)
- ⚠️ **Production-приложения** — использовать только v1.0
- 🔄 **Аутентификация** — OAuth 2.0 / OpenID Connect через Microsoft Entra ID
- 📊 **Типы разрешений** — Delegated (с пользователем) и Application (без пользователя)
- 💡 **Поддержка OData** — `$select`, `$filter`, `$orderby`, `$expand`, `$top`, `$skip`
- ✅ **Преимущества** — единый API, единая модель авторизации, связи между объектами, batching
- ⚠️ **Graph Explorer** — интерактивный инструмент для тестирования API
- 🔒 **Namespace** — `microsoft.graph`

---

# Exam Tips (AZ-204)

## Основы

- Microsoft Graph — единая точка доступа к данным Microsoft 365.
- Endpoint: `https://graph.microsoft.com`.
- Использует REST и стандартные HTTP-методы.

---

## Компоненты

- **Graph API** — доступ к данным Microsoft 365, Windows, EMS.
- **Graph Connectors** — подключение внешних данных (Box, Jira, Salesforce и др.).
- **Graph Data Connect** — массовая выгрузка данных в Azure для аналитики.

---

## Версии API

- **v1.0** — стабильная, использовать в production.
- **beta** — preview, возможны breaking changes.

---

## Аутентификация и разрешения

- Использует OAuth 2.0 и OpenID Connect.
- Delegated — пользователь присутствует.
- Application — приложение действует самостоятельно.
- Часто используемые scopes:
    - `User.Read`
    - `Mail.Read`
    - `Calendars.Read`
    - `Files.Read`

---

## OData-поддержка

- `$select` — выбор полей
- `$filter` — фильтрация
- `$orderby` — сортировка
- `$expand` — связанные сущности
- `$top`, `$skip` — пагинация
- `$count`, `$search` — подсчёт и поиск

---

## Дополнительные возможности

- **Batching** — объединение нескольких запросов (`/$batch`)
- **Delta queries** — отслеживание изменений (`/delta`)
- **Webhooks** — уведомления в реальном времени
- **Metadata** — доступно по `/\$metadata`

---

## Часто проверяется

- Какой endpoint использовать? → `https://graph.microsoft.com`
- Какую версию API использовать в production? → `v1.0`
- Какой тип разрешения нужен для daemon? → Application
- Как отфильтровать данные? → `$filter`

---

> 🎯 Ключевая идея:  
> Microsoft Graph — это единый REST API для всей экосистемы Microsoft 365.

Д. Используя конвейеры Azure Data Factory с механизмом согласия на основе Azure AD, можно добиться этого.
Microsoft Graph Data Connect предназначен для:
работы с большими объёмами данных
пакетного экспорта данных (bulk extraction)
строгого контроля доступа
соблюдения требований compliance
Он работает через:

Azure Data Factory
управляемые пайплайны
механизм согласия (consent) на уровне Azure AD
RBAC и администрируемые разрешения

Процесс выглядит так:
Администратор даёт согласие (admin consent)
Настраивается Azure Data Factory pipeline
Данные экспортируются в Azure Data Lake Storage
Доступ контролируется через Azure AD
🔒 Почему это важно

Graph API:
хорош для онлайн-запросов
не предназначен для массового экспорта миллионов записей
Graph Data Connect:
предназначен именно для масштабной аналитики
не влияет на лимиты Graph API
работает через управляемые Azure-сервисы

🎯 Что значит "optimize query results" в Microsoft Graph
уменьшить объём возвращаемых данных
сократить нагрузку на сеть
сократить время выполнения запроса
уменьшить потребление ресурсов

$expand наоборот увеличивает объём ответа,
потому что подтягивает связанные объекты.

Если в вопросе:
optimize results / reduce payload / reduce response size
Ответ почти всегда:
$filter
$select
[Learn More](https://learn.microsoft.com/en-us/training/modules/microsoft-graph/2-microsoft-graph-overview)
