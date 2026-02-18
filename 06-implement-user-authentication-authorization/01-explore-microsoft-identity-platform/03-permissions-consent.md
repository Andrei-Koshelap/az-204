# Permissions and Consent

## Ключевые понятия

- **Delegated permissions** — пользователь присутствует, приложение действует от его имени
- **Application permissions** — пользователь отсутствует, приложение действует от своего имени
- **Static consent** — все разрешения запрашиваются заранее
- **Dynamic consent** — разрешения запрашиваются по мере необходимости
- **Admin consent** — требуется для разрешений с повышенными привилегиями

---

# Обзор

**Модель авторизации** Microsoft Identity Platform основана на OAuth 2.0.

Основные элементы:

- **OAuth 2.0** — стандартный протокол авторизации
- **Scopes** — детализированные разрешения
- **Consent** — согласие пользователя или администратора
- **Контроль доступа** — пользователи и администраторы управляют доступом к данным

> 💡 Приложение не получает доступ к данным без явного согласия.

---

# Типы разрешений

## 1️⃣ Delegated Permissions (Delegated Access)

**Приложение действует от имени вошедшего пользователя**

### Характеристики

- **Пользователь должен быть авторизован**
- **Согласие может дать пользователь или администратор**
- **Эффективные права** = пересечение прав пользователя и разрешений приложения
- Используется в интерактивных приложениях

> 💡 Приложение не может получить больше прав, чем есть у пользователя.

---

### Когда используется

- Web-приложения
- SPA
- Мобильные приложения
- Клиентские приложения с входом пользователя

---

## Важно для AZ-204

- Delegated permissions требуют присутствия пользователя.
- Токен содержит scopes, отражающие разрешения.
- Даже если приложение запрашивает разрешение, оно ограничено правами пользователя.
- Администратор может выдать согласие сразу для всей организации.

> 🎯 Частый вопрос:  
Если пользователь не имеет доступа к ресурсу, сможет ли приложение получить доступ через delegated permission?  
Ответ — нет.

**How it works**:

```
User permissions:          Can read all emails
App delegated permission:  Can read user email
Effective permissions:     Can read user's own emails only
```

**Example**:

```http
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=00001111-aaaa-2222-bbbb-3333cccc4444
&response_type=code
&redirect_uri=https://localhost:5001/signin-oidc
&scope=https://graph.microsoft.com/User.Read https://graph.microsoft.com/Mail.Read
&state=12345
```

**In code**:

```csharp
using Microsoft.Identity.Client;

var app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri("http://localhost")
    .Build();

// Delegated permissions - user signs in
var scopes = new[] { "User.Read", "Mail.Read" };
var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();

// Access token contains user's identity
// App can only access data user has access to
```

## Часто используемые Delegated Permissions

| Permission | Описание |
|------------|----------|
| `User.Read` | Чтение профиля вошедшего пользователя |
| `Mail.Read` | Чтение почты пользователя |
| `Mail.Send` | Отправка почты от имени пользователя |
| `Calendars.Read` | Чтение календарей пользователя |
| `Files.ReadWrite` | Чтение и запись файлов пользователя |

> 💡 Эти разрешения работают только при наличии вошедшего пользователя и ограничены его правами.

---

# 2️⃣ Application Permissions (App-Only Access)

**Приложение действует от своего имени, без пользователя**

### Характеристики

- **Пользователь отсутствует**  
  Используется в фоновом режиме или автоматизированных процессах.

- **Согласие даёт только администратор**  
  Пользователь не может предоставить consent для application permissions.

- **Эффективные права**  
  Приложение получает полный объём разрешений, который был предоставлен администратором.

- **Типовые сценарии**  
  Фоновые сервисы, daemon-приложения, batch-задачи, автоматизированные процессы.

---

## Отличие от Delegated Permissions

| Delegated | Application |
|------------|------------|
| Требуется пользователь | Пользователь не требуется |
| Ограничены правами пользователя | Работают с выданными правами напрямую |
| Consent может дать пользователь | Consent даёт только администратор |
| Используются в интерактивных приложениях | Используются в сервисах и daemon |

---

## Важно для AZ-204

- Application permissions требуют admin consent.
- Используются с confidential client.
- Часто применяются в client credentials flow.
- Предоставляют более высокий уровень доступа.

> 🎯 Частый вопрос:  
Как реализовать фоновый сервис без пользователя?  
Ответ — использовать Application permissions.


**How it works**:

```
App permission:         Can read all emails in organization
Effective permissions:  Can read all emails in organization
```

**Example**:

```csharp
using Microsoft.Identity.Client;

var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)
    .WithAuthority(new Uri($"https://login.microsoftonline.com/{tenantId}"))
    .Build();

// Application permissions - no user
var scopes = new[] { "https://graph.microsoft.com/.default" };
var result = await app.AcquireTokenForClient(scopes)
    .ExecuteAsync();

// Access token does NOT contain user identity
// App can access data across entire organization
```

## Часто используемые Application Permissions

| Permission | Описание |
|------------|----------|
| `User.Read.All` | Чтение профилей всех пользователей |
| `Mail.Read` | Чтение всех почтовых ящиков |
| `Mail.Send` | Отправка почты от имени любого пользователя |
| `Group.ReadWrite.All` | Чтение и изменение всех групп |
| `Directory.ReadWrite.All` | Чтение и изменение данных каталога |

> ⚠️ Эти разрешения предоставляют доступ на уровне организации и требуют согласия администратора.

---

# Сравнение типов разрешений

| Аспект | Delegated | Application |
|--------|------------|-------------|
| **Пользователь присутствует** | Да | Нет |
| **Кто даёт согласие** | Пользователь или администратор | Только администратор |
| **Приложение действует как** | Вошедший пользователь | Само приложение |
| **Эффективные права** | Пересечение прав пользователя и приложения | Полный объём выданных прав |
| **Типовой сценарий** | Интерактивные приложения | Фоновые сервисы |
| **Тип токена** | Claims пользователя + приложения | Только claims приложения |

---

# Scopes и Permissions

## Понимание Scopes

**Scopes** — это разрешения, определяющие доступ к ресурсам.

### Основные характеристики

- Термин из OAuth 2.0
- Позволяют разделять доступ на мелкие функциональные части
- Представляются строкой
- Передаются в параметре `scope` при запросе токена

> 💡 Scopes реализуют принцип наименьших привилегий — приложение запрашивает только то, что действительно необходимо.

---

## Важно для AZ-204

- Delegated permissions реализуются через scopes.
- Application permissions выдаются через admin consent.
- Scopes определяют, какие операции разрешены.
- Чем выше привилегия — тем чаще требуется согласие администратора.

> 🎯 Частый вопрос:  
Почему приложение не может получить доступ к ресурсу?  
Ответ — не запрошен или не выдан соответствующий scope.


### Scope Format

**Short form** (Microsoft Graph only):

```
User.Read
Mail.Send
Calendars.Read
```

**Full URI form**:

```
https://graph.microsoft.com/User.Read
https://graph.microsoft.com/Mail.Send
https://outlook.office.com/Mail.Read
https://vault.azure.net/user_impersonation
```

### OpenID Connect Scopes

## Стандартные OIDC Scopes

Эти scopes используются в протоколе OpenID Connect для аутентификации пользователя.

| Scope | Описание |
|--------|----------|
| `openid` | Базовый вход пользователя, обязателен для OpenID Connect |
| `profile` | Доступ к информации профиля пользователя |
| `email` | Доступ к email пользователя |
| `offline_access` | Позволяет получить refresh token |

---

## Разбор каждого scope

### `openid`
- Обязателен для получения ID token.
- Активирует использование OpenID Connect поверх OAuth 2.0.
- Без него ID token выдан не будет.

### `profile`
- Добавляет стандартные claims в ID token.
- Позволяет получить базовые данные профиля пользователя.

### `email`
- Позволяет получить email пользователя.
- Используется, если приложению требуется контактная информация.

### `offline_access`
- Позволяет получить refresh token.
- Используется для долгосрочного доступа без повторного входа пользователя.
- Особенно важен для мобильных и desktop-приложений.

---

## Важно для AZ-204

- `openid` обязателен для аутентификации через OpenID Connect.
- `offline_access` нужен для получения refresh token.
- OIDC scopes относятся к аутентификации, а не к доступу к API.
- Access к API определяется другими scopes (например, ресурсными).

> 🎯 Частый вопрос:  
Как получить refresh token?  
Ответ — запросить scope `offline_access`.


**Example**:

```http
scope=openid profile email offline_access User.Read
```

### Microsoft Graph Scopes

**Common Graph API permissions**:

```csharp
// User operations
"User.Read"                    // Read signed-in user's profile
"User.ReadWrite"               // Read and update user's profile
"User.Read.All"                // Read all users' profiles (admin)

// Mail operations
"Mail.Read"                    // Read user's mail
"Mail.ReadWrite"               // Read and write user's mail
"Mail.Send"                    // Send mail as user

// Calendar operations
"Calendars.Read"               // Read user's calendars
"Calendars.ReadWrite"          // Read and write user's calendars

// Files operations
"Files.Read"                   // Read user's files
"Files.ReadWrite.All"          // Read and write all user's files

// Directory operations (admin only)
"Directory.Read.All"           // Read directory data
"Directory.ReadWrite.All"      // Read and write directory data
```

### .default Scope

**Special scope for application permissions**:

```csharp
// Request all pre-configured application permissions
var scopes = new[] { "https://graph.microsoft.com/.default" };

// Equivalent to requesting all application permissions
// configured in Azure Portal for the app
```

# Consent Types

## 1️⃣ Static User Consent

**Все разрешения определяются заранее**

### Характеристики

- **Где настраивается**  
  Разрешения конфигурируются заранее в Azure Portal (в разделе API permissions).

- **Когда происходит согласие**  
  Пользователь подтверждает все запрошенные разрешения при первом входе в приложение.

- **Преимущество**  
  Простая и предсказуемая модель — все права известны заранее.

- **Недостаток**  
  Длинный список разрешений может насторожить пользователя и снизить доверие.

---

## Как это работает

- Приложение запрашивает полный набор scopes при первом входе.
- Пользователь видит список разрешений.
- После согласия повторный запрос не требуется (если не добавлены новые scopes).

---

## Когда использовать

- Приложение изначально требует фиксированный набор разрешений.
- Нет необходимости запрашивать права поэтапно.
- Корпоративные сценарии с централизованным управлением.

---

## Важно для AZ-204

- Static consent означает, что все разрешения запрашиваются сразу.
- Изменение списка разрешений требует нового согласия.
- Администратор может выдать согласие сразу для всей организации.

> 🎯 Частый экзаменационный вопрос:  
Когда пользователь видит окно согласия?  
Ответ — при первом входе, если используется static consent.

**Configuration**:

```
Azure Portal → App registrations → Your app → API permissions
→ Add permissions (select all needed)
→ User sees all permissions on first sign-in
```

## Проблемы Static User Consent

❌ **Длинный список разрешений**  
При первом входе пользователь видит полный перечень запрашиваемых прав, что может вызвать недоверие или отказ.

❌ **Необходимость заранее знать все ресурсы**  
Приложение должно заранее определить полный набор необходимых разрешений, даже если часть из них понадобится позже.

❌ **Отсутствие гибкости**  
Невозможно адаптировать запрос разрешений под конкретные действия пользователя в момент использования.

---

## Почему это важно

- Большое количество прав увеличивает риск отказа от использования приложения.
- Нарушается принцип наименьших привилегий.
- Пользователь может не понимать, зачем приложению все эти разрешения.

---

## Важно для AZ-204

- Static consent удобен, но менее гибок.
- Может быть проблемой в пользовательских (consumer) сценариях.
- Для более гибкой модели используется Dynamic Consent.

> 🎯 Часто проверяется понимание различий между static и dynamic consent.

**Example**:

```
Your app requests:
- Read your profile
- Read your email
- Send email on your behalf
- Read your calendar
- Access your files
- Read all users in directory

User sees this overwhelming list and may decline
```

## 2️⃣ Incremental and Dynamic Consent

**Разрешения запрашиваются по мере необходимости**

### Характеристики

- **Где задаётся**  
  Запрос scopes происходит в коде приложения (через параметр `scope`).

- **Когда запрашивается**  
  В момент использования конкретной функции, требующей дополнительного доступа.

- **Преимущество**  
  Улучшенный пользовательский опыт — пользователь видит только те разрешения, которые действительно нужны в данный момент.

- **Недостаток**  
  Требуется поддержка динамического consent и дополнительная логика в приложении.

- **Применяется только к**  
  Delegated permissions (не работает с application permissions).

---

## Как это работает

- При первом входе запрашивается минимальный набор scopes.
- Когда пользователь пытается использовать функцию, требующую новых прав, приложение запрашивает дополнительный scope.
- Пользователь подтверждает только новые разрешения.

---

## Почему это важно

- Соответствует принципу наименьших привилегий.
- Снижает вероятность отказа на этапе первого входа.
- Позволяет гибко управлять доступом.

---

## Важно для AZ-204

- Dynamic consent работает только для delegated permissions.
- Application permissions требуют admin consent и не поддерживают поэтапный запрос.
- Приложение должно корректно обрабатывать необходимость повторного запроса токена.
- Может потребоваться интерактивный вход при добавлении нового scope.

> 🎯 Частый экзаменационный вопрос:  
Как улучшить UX при большом количестве разрешений?  
Ответ — использовать Incremental/Dynamic Consent.

**How it works**:

```csharp
// Initial sign-in - minimal permissions
var initialScopes = new[] { "User.Read" };
var result = await app.AcquireTokenInteractive(initialScopes)
    .ExecuteAsync();

// Later - when user needs email feature
var emailScopes = new[] { "Mail.Read" };
var emailResult = await app.AcquireTokenSilent(emailScopes, account)
    .ExecuteAsync();
// User prompted to consent only if not already consented
```

**Example flow**:

```
1. App launch → Request: User.Read
   User consents to: Read your profile
   ✅ Minimal, non-threatening

2. User clicks "View Email"
   → Request: Mail.Read
   User consents to: Read your email
   ✅ Contextual, user understands why

3. User clicks "Send Email"
   → Request: Mail.Send
   User consents to: Send email on your behalf
   ✅ Progressive, just-in-time
```

## Преимущества Dynamic / Incremental Consent

✅ **Лучший пользовательский опыт**  
Запрашиваются только необходимые разрешения в конкретный момент времени.

✅ **Постепенное раскрытие доступа**  
Дополнительные scopes запрашиваются при использовании соответствующей функции.

✅ **Более высокая вероятность согласия**  
Пользователь понимает, зачем приложению требуется конкретное разрешение.

---

## Важные ограничения

⚠️ **Admin consent**  
Dynamic consent не может автоматически запросить разрешения, требующие согласия администратора.  
Если разрешение относится к категории повышенных привилегий, потребуется предварительное администраторское согласие.

⚠️ **Регистрация разрешений обязательна**  
Даже при использовании dynamic consent все возможные разрешения должны быть заранее зарегистрированы в Azure Portal (в разделе API permissions).

> 💡 Dynamic consent управляет моментом запроса разрешений, но не отменяет необходимость их предварительной регистрации.

---

## Важно для AZ-204

- Dynamic consent работает только для delegated permissions.
- Разрешения с высоким уровнем доступа требуют admin consent.
- Все scopes должны быть добавлены в App Registration заранее.
- Приложение должно корректно обрабатывать повторный запрос токена.

> 🎯 Частый вопрос:  
Можно ли запросить разрешение, которое не зарегистрировано в приложении?  
Ответ — нет.


```csharp
// Good: Register all in portal (for admin visibility)
// Then request dynamically in code

// Bad: Only register minimal permissions
// Admin can't consent to permissions they can't see
```

## 3️⃣ Admin Consent

**Требуется для разрешений с повышенными привилегиями**

### Основные характеристики

- **Кто может предоставить согласие**  
  Только администратор tenant (обычные пользователи не могут).

- **Когда требуется**  
  При запросе разрешений с высоким уровнем доступа, например:
    - доступ ко всем пользователям
    - доступ ко всем группам
    - изменение данных каталога

- **Как предоставляется**
    - Через специальный admin consent flow
    - Через Azure Portal (Grant admin consent)

- **Область действия**  
  Применяется ко всей организации (для всех пользователей tenant).

---

## Почему это важно

- Предотвращает выдачу критических разрешений обычными пользователями.
- Защищает данные организации.
- Обеспечивает централизованный контроль доступа.

---

## Когда используется

- Application permissions
- Delegated permissions с высоким уровнем доступа
- Multi-tenant SaaS-приложения

---

## Важно для AZ-204

- Application permissions всегда требуют admin consent.
- Некоторые delegated permissions также требуют admin consent.
- После выдачи согласия пользователи больше не видят окно запроса.
- Admin consent может быть выдан заранее до первого входа пользователя.

> 🎯 Частый экзаменационный вопрос:  
Кто может выдать разрешение на чтение всех пользователей в tenant?  
Ответ — только администратор.


**Permissions requiring admin consent**:

```
User.Read.All                  ← Read all users
Mail.Read (application)        ← Read all mailboxes
Directory.ReadWrite.All        ← Modify directory
Group.ReadWrite.All            ← Manage all groups
```

**Admin consent flow**:

```http
GET https://login.microsoftonline.com/{tenant}/adminconsent?
client_id=00001111-aaaa-2222-bbbb-3333cccc4444
&redirect_uri=https://localhost:5001/adminconsent
&state=12345
```

**Admin consent in portal**:

```
Azure Portal → Microsoft Entra ID → App registrations → Your app
→ API permissions → Grant admin consent for [Organization]
```

**Example code**:

```csharp
// Check if admin consent is required
var scopes = new[] { "User.Read.All" };  // Requires admin consent

try
{
    var result = await app.AcquireTokenInteractive(scopes)
        .ExecuteAsync();
}
catch (MsalUiRequiredException ex)
{
    if (ex.ErrorCode == "admin_consent_required")
    {
        // Redirect to admin consent URL
        var adminConsentUrl = 
            $"https://login.microsoftonline.com/{tenantId}/adminconsent" +
            $"?client_id={clientId}" +
            $"&redirect_uri={redirectUri}";
        
        // Redirect user to this URL (admin must sign in)
    }
}
```

**Admin consent for organization**:

```
Admin consents → All users can use app without individual consent
Admin revokes  → App stops working for all users
```

## Requesting Permissions

### OAuth 2.0 Authorization Request

**Structure**:

```http
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=00001111-aaaa-2222-bbbb-3333cccc4444        ← Your app ID
&response_type=code                                    ← Authorization code flow
&redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F       ← Where to send response
&response_mode=query                                   ← How to send response
&scope=https%3A%2F%2Fgraph.microsoft.com%2FUser.Read  ← Permissions requested
       %20https%3A%2F%2Fgraph.microsoft.com%2FMail.Send
&state=12345                                           ← Anti-forgery token
```

### Scope Parameter

**Space-separated list** of permissions:

```http
scope=https://graph.microsoft.com/User.Read 
      https://graph.microsoft.com/Mail.Read 
      https://graph.microsoft.com/Calendars.Read
```

**URL-encoded**:

```http
scope=https%3A%2F%2Fgraph.microsoft.com%2FUser.Read%20https%3A%2F%2Fgraph.microsoft.com%2FMail.Read
```

### Consent Flow

```
1. App requests permissions (scope parameter)
   ↓
2. User redirected to Microsoft identity platform
   ↓
3. User authenticates (username/password/MFA)
   ↓
4. Microsoft checks consent status:
   - If already consented → Skip to step 6
   - If new permissions → Continue to step 5
   ↓
5. User shown consent prompt
   - Lists requested permissions
   - User approves or denies
   ↓
6. User redirected back to app with authorization code
   ↓
7. App exchanges code for access token
   ↓
8. Access token includes consented scopes
```

### Consent Prompt Example

**User sees**:

```
[App Name] wants to:

✓ Read your profile
✓ Read your email  
✓ Send email on your behalf

This app is not published by Microsoft.

[Cancel] [Accept]
```

### Checking Consented Permissions

**In access token**:

```json
{
  "aud": "https://graph.microsoft.com",
  "scp": "User.Read Mail.Read Mail.Send",
  "appid": "00001111-aaaa-2222-bbbb-3333cccc4444"
}
```

**In code**:

```csharp
var result = await app.AcquireTokenSilent(scopes, account)
    .ExecuteAsync();

// Check granted scopes
var grantedScopes = result.Scopes;
foreach (var scope in grantedScopes)
{
    Console.WriteLine($"Granted: {scope}");
}
```

# Resource Identifiers

## Основные ресурсы

В Microsoft Identity Platform доступ к API определяется через **Application ID URI** — уникальный идентификатор ресурса.

| Ресурс | Identifier (Application ID URI) |
|---------|--------------------------------|
| **Microsoft Graph** | `https://graph.microsoft.com` |
| **Microsoft 365 Mail API** | `https://outlook.office.com` |
| **Azure Key Vault** | `https://vault.azure.net` |
| **Azure Storage** | `https://storage.azure.com` |
| **Azure Management** | `https://management.azure.com` |

---

## Что важно понимать

- Identifier указывает, к какому ресурсу запрашивается access token.
- Access token выдается **для конкретного ресурса**.
- Нельзя использовать токен для одного ресурса при обращении к другому.

> 💡 Один токен = один ресурс.

---

## Как используется

- В запросе токена указывается scope, связанный с конкретным ресурсом.
- В access token поле `aud` (audience) содержит идентификатор ресурса.
- API проверяет, что токен выдан именно для него.

---

## Важно для AZ-204

- Каждый API имеет свой уникальный identifier.
- Microsoft Graph — самый часто используемый ресурс.
- Неверный resource identifier приведёт к ошибке авторизации.
- Токены не являются универсальными между сервисами.

> 🎯 Частый вопрос:  
Можно ли использовать токен, полученный для Microsoft Graph, для вызова Azure Management API?  
Ответ — нет.


### Permission String Format

**Full format**:

```
{resource_identifier}/{permission_name}
```

**Examples**:

```
https://graph.microsoft.com/User.Read
https://graph.microsoft.com/Mail.Send
https://outlook.office.com/Mail.Read
https://vault.azure.net/user_impersonation
```

### Short Form (Graph only)

**When resource identifier is omitted**:

```
scope=User.Read

# Equivalent to:
scope=https://graph.microsoft.com/User.Read
```

## Best Practices

### 1. Request Minimum Permissions

```csharp
// ✅ Good: Request only what you need
var scopes = new[] { "User.Read" };

// ❌ Bad: Request all possible permissions
var scopes = new[] { 
    "User.Read.All", 
    "Mail.ReadWrite", 
    "Directory.ReadWrite.All" 
};
```

### 2. Use Incremental Consent

```csharp
// ✅ Good: Progressive disclosure
// Initial: User.Read
// Later: Mail.Read (when email feature used)

// ❌ Bad: Request all upfront
// User sees overwhelming list on first sign-in
```

### 3. Request Delegated When Possible

```csharp
// ✅ Good: Delegated (user context)
scopes = new[] { "User.Read" };

// ❌ Bad: Application when not needed
scopes = new[] { "User.Read.All" };  // Requires admin
```

### 4. Handle Consent Gracefully

```csharp
try
{
    var result = await app.AcquireTokenSilent(scopes, account)
        .ExecuteAsync();
}
catch (MsalUiRequiredException)
{
    // User needs to consent
    try
    {
        result = await app.AcquireTokenInteractive(scopes)
            .ExecuteAsync();
    }
    catch (MsalServiceException ex) when (ex.ErrorCode == "consent_required")
    {
        // User declined consent - handle gracefully
        ShowMessage("This feature requires permission to access your email.");
    }
}
```

### 5. Document Why Permissions Are Needed

```
Privacy Policy / Terms of Service:

"We request access to your profile to personalize your experience."
"We request access to your email to send notifications on your behalf."
"We request access to your calendar to schedule meetings."

Clear explanations increase consent acceptance rate.
```

## Common Patterns

### Pattern 1: Progressive Web App

```csharp
// Login: Minimal permissions
await app.AcquireTokenInteractive(new[] { "openid", "profile" })
    .ExecuteAsync();

// View Profile: User info
await app.AcquireTokenSilent(new[] { "User.Read" }, account)
    .ExecuteAsync();

// Email Feature: Email access
await app.AcquireTokenSilent(new[] { "Mail.Read" }, account)
    .ExecuteAsync();

// Send Feature: Send permission
await app.AcquireTokenSilent(new[] { "Mail.Send" }, account)
    .ExecuteAsync();
```

### Pattern 2: Daemon with Application Permissions

```csharp
// Register in portal: Grant admin consent for all application permissions

// In code: Request .default scope
var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)
    .WithAuthority(new Uri($"https://login.microsoftonline.com/{tenantId}"))
    .Build();

var result = await app.AcquireTokenForClient(
    new[] { "https://graph.microsoft.com/.default" }
).ExecuteAsync();

// Token includes all pre-consented application permissions
```

### Pattern 3: Admin Consent First

```
1. Admin visits admin consent URL
2. Admin signs in and consents for organization
3. All users can now use app without individual consent
4. App requests permissions dynamically
5. Users automatically have access (no prompt)
```

# Critical Notes

- 💡 **OAuth 2.0** — протокол авторизации для управления доступом
- 🎯 **Два типа разрешений** — Delegated (есть пользователь) и Application (без пользователя)
- ✅ **Delegated** — приложение действует от имени вошедшего пользователя
- ⚠️ **Application** — приложение действует от своего имени, требуется admin consent
- 🔄 **Scopes** — детализированные разрешения
- 📊 **Static consent** — все разрешения запрашиваются заранее (через портал)
- 💡 **Dynamic consent** — разрешения запрашиваются поэтапно (в коде)
- ✅ **Admin consent** — обязателен для высокопривилегированных разрешений
- ⚠️ **Effective permissions (delegated)** — пересечение прав пользователя и приложения
- 🔒 **`.default` scope** — запрашивает все заранее настроенные application permissions
- 🎯 **Incremental consent** — улучшает UX за счёт постепенного запроса прав
- 💡 **Resource identifier** — уникальный идентификатор API
- ⚠️ **Best practice** — запрашивать минимально необходимые разрешения

---

# Exam Tips (AZ-204)

## Типы разрешений

- **Delegated permissions**
    - Пользователь присутствует
    - Consent может дать пользователь или администратор
    - Права ограничены правами пользователя

- **Application permissions**
    - Пользователь отсутствует
    - Consent даёт только администратор
    - Приложение получает выданные ему права напрямую

---

## OAuth и Scopes

- OAuth 2.0 — протокол авторизации, используемый Microsoft Identity Platform.
- Scopes — наборы разрешений.
- Передаются в параметре `scope` как список.
- Пример короткой формы: `User.Read`.
- Полная форма включает Application ID URI ресурса.

---

## Consent модели

- **Static consent**  
  Все разрешения определены заранее и отображаются при первом входе.

- **Dynamic / Incremental consent**  
  Разрешения запрашиваются по мере необходимости (только для delegated).

- **Admin consent**  
  Обязателен для разрешений с высоким уровнем доступа.

---

## Важные моменты

- `.default` используется для запроса всех заранее настроенных application permissions.
- OpenID Connect scopes:
    - `openid`
    - `profile`
    - `email`
    - `offline_access`
- Resource identifier определяет, для какого API выдается токен.
- Multi-tenant приложение создаёт Service Principal при согласии в новом tenant.

---

## Обработка consent и токенов

- Consent prompt показывает список запрошенных разрешений.
- При отсутствии согласия может возникнуть необходимость интерактивного входа.
- Администратор может выдать согласие через портал.
- Multi-tenant приложения требуют согласия в каждом tenant.

---

## Что часто проверяется

- Различие Delegated и Application permissions.
- Кто может выдать согласие.
- Что означает effective permissions.
- Назначение `.default`.
- Различие static и incremental consent.
- Почему нужно запрашивать минимальные разрешения.
- Как создаётся service principal в другом tenant.

---

> 🎯 Ключевая идея:  
> Delegated = пользователь + приложение.  
> Application = только приложение.  
> Consent определяет, какие данные доступны.


[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-microsoft-identity-platform/4-permission-consent)
