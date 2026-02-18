# Применение лучших практик Microsoft Graph

## Ключевые концепции
- **Аутентификация** — использовать MSAL для получения токенов
- **Разрешения (Permissions)** — запрашивать минимально необходимые права (least privilege)
- **Consent (согласие)** — понимать разницу между delegated и application permissions
- **Пагинация** — корректно обрабатывать большие наборы данных
- **Throttling** — учитывать ограничения по количеству запросов
- **Evolvable Enums** — предусматривать появление новых значений enum

---

## Аутентификация и авторизация

### Использование Microsoft Authentication Library (MSAL)

**Преимущества MSAL**:

- **Получение токена** — реализует OAuth 2.0 flow
- **Кэширование токенов** — снижает количество повторных аутентификаций
- **Автоматическое обновление токена** — refresh выполняется прозрачно
- **Поддержка нескольких аккаунтов** — работа с несколькими идентификациями

---

### Delegated vs Application Permissions

- **Delegated permissions**
    - Используются от имени пользователя
    - Требуют входа пользователя
    - Права ограничены разрешениями пользователя

- **Application permissions**
    - Работают без пользователя (daemon/service)
    - Требуют admin consent
    - Часто используются для background-процессов

---

## Пагинация

- Microsoft Graph возвращает частичные результаты при больших выборках.
- Использовать `@odata.nextLink` для получения следующей страницы.
- В SDK — применять `PageIterator`.

---

## Throttling (ограничение запросов)

- При превышении лимита возвращается `429 Too Many Requests`.
- Обязательно обрабатывать заголовок `Retry-After`.
- Использовать экспоненциальную стратегию повторов (exponential backoff).
- Минимизировать количество запросов через:
    - `$select`
    - серверную фильтрацию (`$filter`)
    - batching

---

## Evolvable Enums

- Значения enum могут расширяться со временем.
- Нельзя полагаться на фиксированный набор значений.
- Всегда предусматривать обработку неизвестных значений (default case).
- Это часто встречается в beta-версии API.

---

### Дополнение от себя (что важно для AZ-204)

- Всегда использовать **принцип минимальных привилегий**.
- Проверять, нужен ли admin consent.
- Использовать Managed Identity при работе в Azure.
- Не хранить client secrets в коде — применять Azure Key Vault.
- Для production использовать `v1.0`, а не `beta`.


**Integrate with Graph SDK**:

```csharp
using Microsoft.Identity.Client;
using Microsoft.Graph;

// Configure MSAL
var app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri("http://localhost")
    .Build();

// Acquire token
var scopes = new[] { "User.Read" };
var result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();

// Use with Graph
var authProvider = new DelegateAuthenticationProvider(async (request) =>
{
    request.Headers.Authorization = 
        new System.Net.Http.Headers.AuthenticationHeaderValue("Bearer", result.AccessToken);
});

var graphClient = new GraphServiceClient(authProvider);
```

### Token Acquisition Patterns

**Interactive (user present)**:

```csharp
// First time or token expired
var result = await app.AcquireTokenInteractive(scopes)
    .WithPrompt(Prompt.SelectAccount)
    .ExecuteAsync();
```

**Silent (from cache)**:

```csharp
// Try to get cached token first
var accounts = await app.GetAccountsAsync();
AuthenticationResult result;

try
{
    result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
        .ExecuteAsync();
}
catch (MsalUiRequiredException)
{
    // Token expired or not in cache, fallback to interactive
    result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();
}
```

**Device code (headless apps)**:

```csharp
var result = await app.AcquireTokenWithDeviceCode(scopes, deviceCodeResult =>
{
    Console.WriteLine(deviceCodeResult.Message);
    return Task.FromResult(0);
}).ExecuteAsync();
```

## Permissions and Consent

### Principle of Least Privilege

**Request minimum permissions**:

```csharp
// ✅ Good: Specific, read-only
var scopes = new[] { "User.Read", "Calendars.Read" };

// ❌ Bad: Too broad, write access
var scopes = new[] { "User.ReadWrite.All", "Calendars.ReadWrite" };
```

**Распространённые шаблоны разрешений (Common permission patterns)**

| Сценарий | Разрешение | Почему |
|-----------|------------|--------|
| Чтение профиля пользователя | `User.Read` | Минимально необходимое для базовой информации профиля |
| Чтение всех пользователей | `User.Read.All` | Доступ к данным всей организации |
| Отправка email от имени пользователя | `Mail.Send` | Конкретное действие — отправка почты |
| Чтение всей почты | `Mail.Read.All` | Административные сценарии |
| Управление календарём | `Calendars.ReadWrite` | Работа с календарём пользователя |
| Доступ к файлам | `Files.Read.All` | Доступ ко всем файлам |

---

### Выбор правильного типа разрешений

#### Delegated permissions (присутствует пользователь)

- Пользователь выполняет вход (sign-in)
- Приложение действует **от имени пользователя**
- Применяются права самого пользователя
- Consent может дать:
    - сам пользователь (для низкоуровневых прав)
    - администратор (для расширенных прав)

---

### Дополнение (важно для AZ-204)

- Если приложение работает в фоне (daemon, background service) → нужны **Application permissions**.
- Если приложение — веб/мобильное и пользователь вошёл в систему → чаще используются **Delegated permissions**.
- Всегда выбирать **наименее привилегированное** разрешение:
    - `User.Read` лучше, чем `User.ReadWrite.All`
    - `Mail.Send` лучше, чем `Mail.ReadWrite`
- Некоторые разрешения требуют **admin consent** — это часто фигурирует в экзаменационных вопросах.
- При проектировании учитывать принцип least privilege и аудит безопасности.

```csharp
// Delegated - user context
var scopes = new[] { "User.Read", "Mail.Send" };
```

#### Application permissions (daemon / service)

- ❌ Нет входа пользователя (no user sign-in)
- 🤖 Приложение действует **от своего имени**
- 🔓 Обычно предоставляется широкий доступ к ресурсу
- 🛡 Consent может дать только администратор (Admin consent)

---

### Когда использовать Application permissions

- Фоновые сервисы (background jobs)
- Интеграции между сервисами
- ETL / синхронизация данных
- Автоматические процессы без участия пользователя

---

### Важно для AZ-204

- Application permissions требуют **admin consent**.
- Используются вместе с:
    - Client credentials flow
    - Managed Identity (в Azure — предпочтительный вариант)
- Права не ограничиваются пользователем — нужно особенно строго соблюдать принцип least privilege.
- Частая экзаменационная ловушка:  
  если нет пользователя → Delegated permissions не подойдут.


```csharp
// Application - app-only context
var scopes = new[] { "https://graph.microsoft.com/.default" };
```
### Матрица выбора (Decision matrix)

| Вопрос | Ответ | Тип разрешения |
|--------|--------|----------------|
| Есть интерактивный пользователь? | Да | Delegated |
| Есть интерактивный пользователь? | Нет | Application |
| Это фоновый сервис? | Да | Application |
| Веб-приложение с авторизованным пользователем? | Да | Delegated |
| Запланированная задача (Scheduled task)? | Да | Application |

---

### Как быстро принять решение на экзамене

1. 🔎 Есть ли вход пользователя (sign-in)?  
   → Да → **Delegated**

2. 🤖 Работает ли приложение без пользователя (daemon / background)?  
   → Да → **Application**

3. 🛡 Нужен ли доступ ко всем данным организации независимо от конкретного пользователя?  
   → Обычно **Application**

---

### Важно для AZ-204

- **Delegated** = действует от имени пользователя.
- **Application** = действует от имени приложения.
- Если в вопросе указано:
    - background job
    - daemon service
    - scheduled task
    - server-to-server integration  
      → почти всегда правильный ответ — **Application permissions**.
- Если указано:
    - пользователь вошёл в систему
    - веб-приложение
    - мобильное приложение  
      → чаще всего — **Delegated permissions**.

### Consider End-User Experience

**Dynamic consent** (add permissions as needed):

```csharp
// Initial request
var initialScopes = new[] { "User.Read" };
var user = await graphClient.Me.GetAsync();

// Later, request additional permission
var mailScopes = new[] { "Mail.Read" };
var result = await app.AcquireTokenInteractive(mailScopes).ExecuteAsync();
```

**Incremental consent** (progressive enhancement):

```csharp
// Step 1: Basic profile
public async Task<User> GetBasicProfile()
{
    var scopes = new[] { "User.Read" };
    // ... acquire token with basic scope
    return await graphClient.Me.GetAsync();
}

// Step 2: Add mail access when needed
public async Task<MessageCollectionResponse> GetMail()
{
    var scopes = new[] { "User.Read", "Mail.Read" };
    // ... acquire token with additional scope
    return await graphClient.Me.Messages.GetAsync();
}
```

### Admin Consent (Согласие администратора)

**Требуется для**:

- 🔐 **Application permissions**
- ⚠️ **Delegated permissions с повышенными привилегиями**
- 🏢 **Доступа ко всей организации (organization-wide access)**

---

### Что это означает

- Обычный пользователь **не может** выдать такие разрешения.
- Только администратор Azure AD / Entra ID может подтвердить (grant consent).
- После admin consent приложение получает доступ в рамках запрошенных scopes.

---

### Важно для AZ-204

- Если используется **Application permissions** → всегда нужен admin consent.
- Некоторые delegated-разрешения (например `User.ReadWrite.All`) тоже требуют admin consent.
- В сценариях enterprise-приложений часто фигурирует:
    - “requires admin approval”
    - “needs organization-wide access”
      → это явный индикатор admin consent.

- Если в вопросе указано, что пользователи не могут предоставить согласие самостоятельно → правильный ответ будет связан с **admin consent workflow**.


**Admin consent URL**:

```
https://login.microsoftonline.com/{tenant}/adminconsent
  ?client_id={client-id}
  &state={state}
  &redirect_uri={redirect-uri}
```

**Check consent status**:

```csharp
try
{
    var users = await graphClient.Users.GetAsync();
}
catch (ServiceException ex) when (ex.StatusCode == System.Net.HttpStatusCode.Forbidden)
{
    if (ex.Error.Code == "Authorization_RequestDenied")
    {
        Console.WriteLine("Admin consent required");
        // Redirect to admin consent URL
    }
}
```

### Multi-Tenant Considerations

**Support multiple organizations**:

```csharp
// Multi-tenant authority
var app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, "common")  // Not specific tenant
    .WithRedirectUri("http://localhost")
    .Build();
```

**Tenant-specific resources**:

```csharp
// Access resources in user's tenant
var tenantId = result.TenantId;
var users = await graphClient.Users.GetAsync();  // Users from user's tenant
```

## Handle Responses

### Pagination

**OData nextLink pattern**:

```csharp
var allUsers = new List<User>();
var usersPage = await graphClient.Users.GetAsync();

// First page
allUsers.AddRange(usersPage.Value);

// Subsequent pages
while (usersPage.OdataNextLink != null)
{
    var nextPageRequest = new Uri(usersPage.OdataNextLink);
    
    // The SDK doesn't have a built-in way to follow nextLink
    // Use PageIterator instead (shown below)
}
```

**PageIterator approach** (recommended):

```csharp
var usersPage = await graphClient.Users.GetAsync();

var pageIterator = PageIterator<User, UserCollectionResponse>
    .CreatePageIterator(
        graphClient,
        usersPage,
        user =>
        {
            // Process each user
            Console.WriteLine(user.DisplayName);
            
            // Return true to continue, false to stop
            return true;
        }
    );

await pageIterator.IterateAsync();
```

**With processing logic**:

```csharp
var allAdmins = new List<User>();

var usersPage = await graphClient.Users.GetAsync(config =>
{
    config.QueryParameters.Filter = "accountEnabled eq true";
});

var pageIterator = PageIterator<User, UserCollectionResponse>
    .CreatePageIterator(
        graphClient,
        usersPage,
        user =>
        {
            // Custom processing
            if (user.JobTitle?.Contains("Admin") == true)
            {
                allAdmins.Add(user);
            }
            
            return true;
        }
    );

await pageIterator.IterateAsync();

Console.WriteLine($"Found {allAdmins.Count} admins");
```

**Pause and resume**:

```csharp
var pageIterator = PageIterator<User, UserCollectionResponse>
    .CreatePageIterator(
        graphClient,
        usersPage,
        user =>
        {
            Console.WriteLine(user.DisplayName);
            
            // Stop after 50 users
            return allUsers.Count < 50;
        }
    );

await pageIterator.IterateAsync();

// Later, resume from where we stopped
if (pageIterator.State != PagingState.Complete)
{
    await pageIterator.ResumeAsync();
}
```

### Evolvable Enumerations

**Problem**: New enum values added without breaking changes

**Solution**: Use `Prefer: graph.microsoft.com-unknown-enum-members` header

**SDK handling**:

```csharp
// Set header globally
var graphClient = new GraphServiceClient(credential, scopes);

// When retrieving entities with enums
var message = await graphClient.Me.Messages["message-id"].GetAsync(config =>
{
    config.Headers.Add("Prefer", "graph.microsoft.com-unknown-enum-members");
});

// Check for unknown values
if (message.Importance == Importance.Normal || 
    message.Importance == Importance.High || 
    message.Importance == Importance.Low)
{
    // Known value
}
else
{
    // Handle unknown/new enum value
    Console.WriteLine($"Unknown importance value: {message.Importance}");
}
```

**Type-safe handling**:

```csharp
switch (message.Importance)
{
    case Importance.Normal:
        Console.WriteLine("Normal priority");
        break;
    case Importance.High:
        Console.WriteLine("High priority");
        break;
    case Importance.Low:
        Console.WriteLine("Low priority");
        break;
    default:
        // New enum value not yet in SDK
        Console.WriteLine("Unknown priority, treating as normal");
        break;
}
```

## Store Data Locally

### Prefer Real-Time Calls

**Call Microsoft Graph directly** when possible:

```csharp
// ✅ Good: Direct call
public async Task<User> GetUserProfile(string userId)
{
    return await graphClient.Users[userId].GetAsync();
}

// ❌ Bad: Storing in local database
public async Task<User> GetUserProfile(string userId)
{
    // Check local cache
    var cachedUser = await _database.Users.FindAsync(userId);
    if (cachedUser != null) return cachedUser;
    
    // Fetch and store
    var user = await graphClient.Users[userId].GetAsync();
    await _database.Users.AddAsync(user);
    await _database.SaveChangesAsync();
    
    return user;
}
```

### Когда допустимо кэширование (When Caching Is Acceptable)

#### Сценарии

- **Производительность (Performance)** — снижение задержек при частом доступе к данным
- **Оффлайн-режим (Offline)** — поддержка работы без постоянного соединения
- **Снижение стоимости (Cost)** — уменьшение количества API-запросов

---

#### Требования

- ✅ Соответствовать Microsoft API Terms of Use  
  https://docs.microsoft.com/legal/microsoft-apis/terms-of-use
- ✅ Соблюдать Microsoft Privacy Statement  
  https://privacy.microsoft.com/privacystatement
- ✅ Учитывать политики хранения данных (data retention policies)
- ✅ Реализовать корректную стратегию инвалидации кэша
- ✅ Обрабатывать запросы на удаление пользовательских данных

---

### Важно для AZ-204

- Нельзя кэшировать данные бесконтрольно — особенно персональные.
- Кэш должен:
    - иметь TTL (time-to-live),
    - корректно обновляться,
    - очищаться при удалении пользователя.
- При работе с Microsoft Graph необходимо соблюдать требования по защите персональных данных (GDPR и аналогичные нормы).
- Если в вопросе фигурирует хранение данных пользователя — всегда учитывать privacy и retention требования.


**Caching pattern**:

```csharp
public class GraphCacheService
{
    private readonly IMemoryCache _cache;
    private readonly GraphServiceClient _graphClient;

    public async Task<User> GetUserWithCache(string userId)
    {
        var cacheKey = $"user:{userId}";
        
        if (!_cache.TryGetValue(cacheKey, out User user))
        {
            // Fetch from Graph
            user = await _graphClient.Users[userId].GetAsync();
            
            // Cache for 15 minutes
            _cache.Set(cacheKey, user, TimeSpan.FromMinutes(15));
        }
        
        return user;
    }
}
```

## Rate Limiting and Throttling

### Understand Throttling

**Microsoft Graph limits**:

| Resource | Requests per second |
|----------|---------------------|
| Any API | 2,000 per app |
| /me | 1,000 per user |
| /users | 1,000 per user |
| /groups | 500 per app |
| /applications | 500 per app |## Rate Limiting и Throttling

### Понимание throttling

Microsoft Graph ограничивает количество запросов, чтобы защитить сервис и обеспечить стабильную работу для всех клиентов.

**Типовые лимиты Microsoft Graph**:

| Ресурс | Запросов в секунду |
|----------|-------------------|
| Любой API | 2 000 на приложение |
| /me | 1 000 на пользователя |
| /users | 1 000 на пользователя |
| /groups | 500 на приложение |
| /applications | 500 на приложение |

> ⚠️ Лимиты могут изменяться и зависят от типа tenant, сценария и нагрузки.

---

### Что происходит при превышении лимита

- Возвращается статус: **429 Too Many Requests**
- В ответе присутствует заголовок: `Retry-After`
- Запрос необходимо повторить **после указанного времени**

---

### Best Practices

- ✅ Использовать экспоненциальный backoff
- ✅ Учитывать заголовок `Retry-After`
- ✅ Минимизировать количество запросов через:
    - `$select`
    - `$filter`
    - batching
- ✅ Кэшировать данные, если это допустимо
- ❌ Не выполнять агрессивные повторные запросы без задержки

---

### Важно для AZ-204

- Если в вопросе фигурирует ошибка 429 → правильный ответ связан с:
    - обработкой `Retry-After`
    - retry-политикой
- SDK уже включает базовую поддержку retry, но логику нужно понимать.
- Throttling — это не ошибка сервера, а механизм защ


**Throttling response**:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 3600
Content-Type: application/json

{
  "error": {
    "code": "TooManyRequests",
    "message": "Rate limit exceeded. Retry after 3600 seconds."
  }
}
```

### Handle Throttling

**Retry with exponential backoff**:

```csharp
using Microsoft.Graph.Models.ODataErrors;

public async Task<User> GetUserWithRetry(string userId, int maxRetries = 3)
{
    int retries = 0;
    
    while (retries < maxRetries)
    {
        try
        {
            return await graphClient.Users[userId].GetAsync();
        }
        catch (ODataError ex) when (ex.ResponseStatusCode == 429)
        {
            // Get Retry-After header
            var retryAfter = ex.Error?.InnerError?.AdditionalData?["Retry-After"];
            var delaySeconds = retryAfter != null ? int.Parse(retryAfter.ToString()) : Math.Pow(2, retries);
            
            Console.WriteLine($"Throttled. Waiting {delaySeconds} seconds...");
            await Task.Delay(TimeSpan.FromSeconds(delaySeconds));
            
            retries++;
        }
    }
    
    throw new Exception("Max retries exceeded");
}
```

**SDK automatic retry** (built-in):

```csharp
// The SDK automatically retries on 429, 503, 504
// Default: 3 retries with exponential backoff

var user = await graphClient.Users[userId].GetAsync();
// SDK handles retries automatically
```

### Reduce Request Frequency

**Batching** (combine requests):

```csharp
var batchContent = new BatchRequestContent();

// Add multiple requests
var user1Request = graphClient.Users["user1@contoso.com"].ToGetRequestInformation();
var user2Request = graphClient.Users["user2@contoso.com"].ToGetRequestInformation();
var user3Request = graphClient.Users["user3@contoso.com"].ToGetRequestInformation();

await batchContent.AddBatchRequestStepAsync(user1Request);
await batchContent.AddBatchRequestStepAsync(user2Request);
await batchContent.AddBatchRequestStepAsync(user3Request);

// Single request, 3 operations
var batchResponse = await graphClient.Batch.PostAsync(batchContent);
```

**Delta query** (get only changes):

```csharp
// Initial request
var usersPage = await graphClient.Users.Delta.GetAsync();

// Store delta link
var deltaLink = usersPage.OdataDeltaLink;

// Later, get only changes
var changesRequest = new Uri(deltaLink);
var changes = await graphClient.Users.Delta.GetAsync();  // Only changed users
```

## Обработка ошибок (Error Handling)

### Частые коды ошибок

| Код | HTTP статус | Значение |
|------|------------|----------|
| `InvalidAuthenticationToken` | 401 | Токен истёк или недействителен |
| `AccessDenied` | 403 | Недостаточно прав (permissions) |
| `ResourceNotFound` | 404 | Ресурс не существует |
| `TooManyRequests` | 429 | Превышен лимит запросов (throttling) |
| `ServiceNotAvailable` | 503 | Сервис временно недоступен |
| `GatewayTimeout` | 504 | Таймаут запроса (шлюз/прокси) |

---

### Дополнение от себя (практика + AZ-204)

- **401**: чаще всего проблема с токеном
    - получить новый токен (MSAL / Azure.Identity),
    - проверить audience/scopes.
- **403**: токен есть, но прав не хватает
    - проверить scopes/roles,
    - delegated vs application permissions,
    - нужен ли admin consent.
- **404**: неверный id/endpoint или ресурс действительно удалён/не создан.
- **429**: обязательно учитывать `Retry-After` и делать retry с backoff.
- **503/504**: временные сбои
    - повторить запрос (retry),
    - добавить таймауты и circuit breaker (в проде).

Также Microsoft Graph часто возвращает объект `error` с `code`, `message`, `innerError` — это помогает быстро диагностировать причину.


### Comprehensive Error Handling

```csharp
using Microsoft.Graph.Models.ODataErrors;

public async Task<User> GetUserSafely(string userId)
{
    try
    {
        return await graphClient.Users[userId].GetAsync();
    }
    catch (ODataError ex) when (ex.ResponseStatusCode == 401)
    {
        Console.WriteLine("Authentication failed. Token may be expired.");
        // Re-authenticate
        throw;
    }
    catch (ODataError ex) when (ex.ResponseStatusCode == 403)
    {
        Console.WriteLine($"Access denied: {ex.Error?.Message}");
        Console.WriteLine("Check application permissions.");
        throw;
    }
    catch (ODataError ex) when (ex.ResponseStatusCode == 404)
    {
        Console.WriteLine($"User not found: {userId}");
        return null;  // Handle gracefully
    }
    catch (ODataError ex) when (ex.ResponseStatusCode == 429)
    {
        var retryAfter = ex.Error?.InnerError?.AdditionalData?["Retry-After"];
        Console.WriteLine($"Throttled. Retry after {retryAfter} seconds.");
        throw;
    }
    catch (ODataError ex)
    {
        Console.WriteLine($"Graph API error: {ex.Error?.Code}");
        Console.WriteLine($"Message: {ex.Error?.Message}");
        throw;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Unexpected error: {ex.Message}");
        throw;
    }
}
```

## Performance Optimization

### Use $select

**Request only needed properties**:

```csharp
// Reduces payload size
var user = await graphClient.Me.GetAsync(config =>
{
    config.QueryParameters.Select = new[] { "id", "displayName", "mail" };
});
```

### Use $filter

**Filter server-side**:

```csharp
// Server filters, returns fewer results
var activeUsers = await graphClient.Users.GetAsync(config =>
{
    config.QueryParameters.Filter = "accountEnabled eq true";
});
```

### Use $top

**Limit result count**:

```csharp
// Get only 50 results
var recentMessages = await graphClient.Me.Messages.GetAsync(config =>
{
    config.QueryParameters.Top = 50;
    config.QueryParameters.Orderby = new[] { "receivedDateTime desc" };
});
```

### Use $expand

**Reduce round trips**:

```csharp
// Single request instead of two
var userWithManager = await graphClient.Me.GetAsync(config =>
{
    config.QueryParameters.Expand = new[] { "manager" };
});

Console.WriteLine($"User: {userWithManager.DisplayName}");
Console.WriteLine($"Manager: {userWithManager.Manager?.DisplayName}");
```
## Critical Notes (Ключевые моменты)

- 💡 **MSAL** — использовать для получения токена, кэширования и автоматического обновления
- 🔒 **Least privilege** — запрашивать минимально необходимые разрешения
- 🎯 **Типы разрешений** — Delegated (есть пользователь), Application (daemon/service)
- ✅ **Consent** — использовать динамический / инкрементальный запрос прав для лучшего UX
- ⚠️ **Пагинация** — применять PageIterator для больших коллекций
- 📊 **Evolvable enums** — использовать заголовок `Prefer` для обработки неизвестных enum-значений
- 💡 **Реальные вызовы** — по возможности выполнять прямые запросы к Graph вместо избыточного кэширования
- 🔄 **Throttling** — обрабатывать 429 и учитывать заголовок `Retry-After`
- ✅ **Batching** — объединять несколько запросов в один HTTP-вызов
- ⚠️ **Delta query** — получать только изменения с момента последнего запроса
- 🔒 **Обработка ошибок** — перехватывать `ODataError` для точной диагностики
- 📊 **Оптимизация** — использовать `$select`, `$filter`, `$top`, `$expand`

---

## Exam Tips (Советы к AZ-204)

- **Аутентификация** — использовать MSAL для получения и кэширования токенов
- **Least privilege** — `User.Read` лучше, чем `User.ReadWrite.All`
- **Типы разрешений**:
    - Delegated — есть пользователь
    - Application — daemon/service
- **Consent**:
    - Dynamic — запрашивать права по мере необходимости
    - Incremental — постепенно расширять доступ
- **Admin consent** — обязателен для Application permissions и высокопривилегированных delegated
- **Multi-tenant** — использовать authority `common`, ресурсы из tenant пользователя
- **Пагинация** — использовать PageIterator для автоматического обхода страниц
- **Evolvable enums** — заголовок  
  `Prefer: graph.microsoft.com-unknown-enum-members`
- **Хранение данных** — предпочитать real-time вызовы Graph; кэшировать только при необходимости
- **Требования к кэшированию** — соблюдать terms, retention policies, реализовать инвалидацию
- **Throttling**:
    - HTTP 429
    - учитывать `Retry-After`
    - использовать exponential backoff
- **Лимиты**:
    - 2 000 запросов/сек на приложение
    - 1 000 на пользователя
- **Batching** — объединять несколько операций в один HTTP-запрос
- **Delta query** — использовать `@odata.deltaLink` для получения инкрементальных изменений
- **Обработка ошибок**:
    - перехватывать `ODataError`
    - проверять `ResponseStatusCode`
- **Типичные ошибки**:
    - 401 — проблемы с аутентификацией
    - 403 — недостаточно прав
    - 404 — ресурс не найден
    - 429 — превышен лимит
- **Оптимизация**:
    - `$select` — уменьшить payload
    - `$filter` — фильтрация на сервере
    - `$expand` — уменьшить количество round trips
- **Автоматические retry в SDK** — встроены для 429, 503, 504

---

### Что часто проверяют в вопросах

- Правильный выбор между Delegated и Application.
- Нужно ли admin consent.
- Как обработать 429.
- Как получить только изменения (delta query).
- Как уменьшить количество запросов и объём данных.


[Learn More](https://learn.microsoft.com/en-us/training/modules/microsoft-graph/5-microsoft-graph-best-practices)
