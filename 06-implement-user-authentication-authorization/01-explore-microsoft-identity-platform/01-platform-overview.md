# Обзор Microsoft Identity Platform

## Ключевые понятия

- **OAuth 2.0 и OpenID Connect** — отраслевые стандарты протоколов аутентификации и авторизации
- **MSAL** — Microsoft Authentication Libraries
- **Microsoft Entra ID** — сервис управления идентификацией и доступом
- **Поддержка нескольких типов учётных записей** — рабочие, учебные, личные и социальные аккаунты

---

## Что такое Microsoft Identity Platform?

**Комплексная платформа управления идентификацией и доступом** для Azure и облачных приложений.

Основные характеристики:

- **Сервис аутентификации** — соответствует стандартам OAuth 2.0 и OpenID Connect
- **Поддержка разных типов учётных записей**
- **Библиотеки** — MSAL для различных платформ и языков
- **Управление** — настройка через Azure Portal или API
- **Современные механизмы безопасности** — passwordless, MFA, Conditional Access

> 💡 Платформа объединяет механизмы аутентификации, выдачи токенов и контроля доступа для приложений и API.

---

## Назначение

Позволяет создавать приложения, в которых пользователи могут:

- ✅ Выполнять вход с использованием Microsoft-учётных записей
- ✅ Входить через социальные аккаунты
- ✅ Безопасно получать доступ к вашим API
- ✅ Получать доступ к Microsoft API (например, Microsoft Graph)

---

# Компоненты платформы

## 1. Authentication Service

**Соответствует стандартам OAuth 2.0 и OpenID Connect**

Поддерживаемые типы идентификаций:

| Тип идентификации | Описание | Пример |
|-------------------|----------|--------|
| **Work/School Accounts** | Учётные записи, созданные в Microsoft Entra ID | Корпоративные пользователи |
| **Personal Microsoft Account** | Потребительские аккаунты | Skype, Xbox, Outlook.com |
| **Social/Local (B2C)** | Azure AD B2C | Вход через Facebook или Google |
| **Social/Local (External ID)** | Microsoft Entra External ID | Аккаунты клиентов |

---

## 2. Microsoft Authentication Libraries (MSAL)

**Open-source библиотеки для аутентификации**

Назначение:

- Получение и обновление токенов доступа
- Поддержка OAuth 2.0 flows
- Работа с различными платформами (веб, мобильные, десктопные приложения)
- Автоматическое управление кешированием токенов

Основные особенности:

- Поддержка интерактивной и безынтерактивной аутентификации
- Поддержка различных grant flows
- Унифицированная модель работы с токенами

---

## Важно для AZ-204

- Microsoft Identity Platform основана на стандартах OAuth 2.0 и OpenID Connect.
- Microsoft Entra ID является провайдером идентификации.
- MSAL используется для получения токенов в приложении.
- Поддерживаются разные типы аккаунтов в зависимости от сценария.
- Токены используются для доступа к защищённым API.


```csharp
// .NET example
using Microsoft.Identity.Client;

var app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri("http://localhost")
    .Build();

// Acquire token interactively
var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();
```

**Supported platforms**:
- .NET / .NET Framework
- JavaScript / TypeScript
- Java
- Python
- Android
- iOS / macOS
- Universal Windows Platform (UWP)

### 3. Microsoft Identity Platform Endpoint

**OAuth 2.0 endpoint** for authentication and authorization:

```
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize
https://login.microsoftonline.com/{tenant}/oauth2/v2.0/token
```

### Основные возможности (Features)

- **Человекочитаемые scopes**  
  Используются стандартные разрешения (scopes), понятные разработчикам и соответствующие отраслевым стандартам.

- **Совместимость с MSAL и другими библиотеками**  
  Работает как с Microsoft Authentication Libraries, так и с любыми библиотеками, поддерживающими стандарты OAuth 2.0 и OpenID Connect.

- **Поддержка протоколов OAuth 2.0 и OpenID Connect**  
  Обеспечивает:
   - Аутентификацию пользователей
   - Выдачу access token и ID token
   - Делегированный и application-доступ

---

## Важно для AZ-204

- Scopes определяют, к каким ресурсам запрашивается доступ.
- MSAL упрощает работу с токенами, но можно использовать и другие совместимые библиотеки.
- OAuth 2.0 отвечает за авторизацию, OpenID Connect — за аутентификацию.

> 🎯 Часто проверяется понимание различия между authentication (кто пользователь) и authorization (к каким ресурсам есть доступ).


**Example authorization request**:

```http
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?
client_id=00001111-aaaa-2222-bbbb-3333cccc4444
&response_type=code
&redirect_uri=https%3A%2F%2Flocalhost%3A5001%2Fsignin-oidc
&response_mode=form_post
&scope=openid%20profile%20email%20offline_access
&state=12345
```

### 4. Application Management Portal

**Azure Portal** for app registration and configuration:

```bash
# Navigate to Azure Portal
https://portal.azure.com

# Go to: Microsoft Entra ID → App registrations → New registration
```

## Параметры конфигурации (Configuration Options)

При настройке приложения в Microsoft Identity Platform доступны следующие параметры:

- **App registration** — регистрация приложения (single-tenant или multi-tenant)
- **Client secrets и сертификаты** — учетные данные приложения для аутентификации
- **API permissions и scopes** — разрешения на доступ к ресурсам
- **Redirect URIs** — адреса возврата после аутентификации
- **Branding customization** — настройка внешнего вида страницы входа
- **Authentication settings** — параметры протоколов и потоков аутентификации

---

## Создание App Registration (через Azure Portal)

1. **Перейти**: Microsoft Entra ID → App registrations
2. **Выбрать**: New registration
3. **Настроить параметры**:
   - **Name** — имя приложения
   - **Supported account types** — выбор между Single-tenant и Multi-tenant
   - **Redirect URI** — URL возврата (callback) вашего приложения
4. **Сохранить** — система создаёт Application (client) ID

---

## Что важно понимать для AZ-204

- Application (client) ID используется в коде приложения для идентификации клиента.
- Single-tenant — приложение доступно только пользователям одного каталога.
- Multi-tenant — приложение может использоваться пользователями из разных организаций.
- Redirect URI должен точно совпадать с тем, что используется в приложении.
- Client secret или сертификат применяются в серверных сценариях (confidential clients).

> 🎯 Часто проверяется понимание различий между single-tenant и multi-tenant приложениями, а также назначение redirect URI и client secret.
> # Client Secret — зачем нужен?

**Client secret** — это учётные данные приложения, аналог «пароля» для приложения.

## Используется в сценариях:

- Server-side приложениях (confidential clients)
- Обмене authorization code на access token
- Client credentials flow

## Важно понимать

- Никогда не хранится в frontend-приложениях.
- Должен храниться безопасно (например, в Azure Key Vault).
- Имеет срок действия.

> 🔐 Client secret подтверждает, что именно ваше приложение запрашивает токен.


### 5. Application Configuration API

**Programmatic configuration** via Microsoft Graph API:

```http
POST https://graph.microsoft.com/v1.0/applications
Content-Type: application/json
Authorization: Bearer {token}

{
  "displayName": "My Application",
  "signInAudience": "AzureADMyOrg",
  "web": {
    "redirectUris": ["https://localhost:5001/signin-oidc"],
    "implicitGrantSettings": {
      "enableIdTokenIssuance": true
    }
  },
  "requiredResourceAccess": [
    {
      "resourceAppId": "00000003-0000-0000-c000-000000000000",
      "resourceAccess": [
        {
          "id": "e1fe6dd8-ba31-4d61-89e7-88639da4683d",
          "type": "Scope"
        }
      ]
    }
  ]
}
```

**PowerShell configuration**:

```powershell
# Connect to Microsoft Graph
Connect-MgGraph -Scopes "Application.ReadWrite.All"

# Create app registration
$app = New-MgApplication -DisplayName "My App" `
    -SignInAudience "AzureADMyOrg" `
    -Web @{
        RedirectUris = @("https://localhost:5001/signin-oidc")
    }

# Add API permissions
New-MgApplicationPermission -ApplicationId $app.Id `
    -ResourceAppId "00000003-0000-0000-c000-000000000000" `
    -Scopes @("User.Read")
```

# Modern Identity Innovations

Современные механизмы безопасности в Microsoft Identity Platform доступны «из коробки» и интегрированы с Microsoft Entra ID.

---

## 1️⃣ Passwordless Authentication

**Аутентификация без использования пароля**

Поддерживаемые методы:

- Windows Hello for Business
- FIDO2 security keys
- Microsoft Authenticator
- Вход по SMS или телефону

### Что это даёт

- Снижение риска фишинга
- Устранение атак с подбором паролей
- Улучшенный пользовательский опыт
- Соответствие современным требованиям безопасности

> 💡 Пароль считается слабым звеном безопасности. Passwordless снижает зависимость от него.

---

## 2️⃣ Step-Up Authentication

**Адаптивная аутентификация на основе уровня риска**

Система может усиливать требования к проверке личности:

- Низкий риск → достаточно одного фактора
- Средний риск → требуется MFA
- Высокий риск → дополнительная проверка

### Принцип работы

Оценка риска выполняется автоматически на основе поведения пользователя, устройства и других сигналов.

> 🎯 Часто используется вместе с Conditional Access.

---

## 3️⃣ Conditional Access

**Политики доступа на основе условий**

Позволяет управлять доступом в зависимости от контекста:

- Геолокация пользователя
- Соответствие устройства требованиям безопасности
- Уровень риска входа
- Конкретное приложение

### Возможности

- Требование MFA при определённых условиях
- Блокировка доступа из небезопасных регионов
- Ограничение доступа с unmanaged-устройств
- Применение разных политик к разным приложениям

> 💡 Это механизм централизованного контроля доступа на основе политик.

---

## 4️⃣ Risk Detection

**Автоматическое обнаружение угроз**

Система анализирует сигналы и выявляет подозрительную активность:

- Нетипичное перемещение пользователя
- Использование анонимных IP-адресов
- IP-адреса, связанные с вредоносной активностью
- Необычные параметры входа
- Password spray атаки
- Утёкшие учётные данные

### Назначение

- Автоматическая оценка риска входа
- Интеграция с Conditional Access
- Усиление требований к аутентификации

---

# Важно для AZ-204

- Passwordless и MFA — встроенные механизмы безопасности.
- Conditional Access применяет политики на основе условий.
- Risk Detection автоматически оценивает угрозы.
- Step-Up Authentication усиливает проверку в зависимости от уровня риска.

> 🎯 Экзамен часто проверяет понимание различий между MFA, Conditional Access и Risk-based authentication.


## Authentication Flow

### Basic OAuth 2.0 Authorization Code Flow

```
┌─────────┐                                           ┌──────────────┐
│         │                                           │              │
│  User   │                                           │  Microsoft   │
│ Browser │                                           │  Identity    │
│         │                                           │  Platform    │
└────┬────┘                                           └──────┬───────┘
     │                                                       │
     │ 1. Sign in request                                   │
     ├──────────────────────────────────────────────────────>
     │                                                       │
     │ 2. Authentication prompt                             │
     │<──────────────────────────────────────────────────────┤
     │                                                       │
     │ 3. User credentials                                  │
     ├──────────────────────────────────────────────────────>
     │                                                       │
     │ 4. Authorization code                                │
     │<──────────────────────────────────────────────────────┤
     │                                                       │
     │ 5. Send code to app                                  │
     ├────────────────────────>│                            │
                                │  Your Application          │
                                │                            │
                                │ 6. Exchange code for token │
                                ├───────────────────────────>│
                                │                            │
                                │ 7. Access token + Refresh │
                                │<───────────────────────────┤
                                │                            │
                                │ 8. Call API with token     │
                                ├───────────────────────────>│
                                │                            │
```

### Implementation Example

```csharp
using Microsoft.Identity.Client;

public class AuthenticationService
{
    private readonly IPublicClientApplication _app;
    private readonly string[] _scopes = new[] { "User.Read" };

    public AuthenticationService(string clientId, string tenantId)
    {
        _app = PublicClientApplicationBuilder
            .Create(clientId)
            .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
            .WithRedirectUri("http://localhost")
            .Build();
    }

    public async Task<AuthenticationResult> SignInAsync()
    {
        try
        {
            // Try to get token silently (from cache)
            var accounts = await _app.GetAccountsAsync();
            var result = await _app.AcquireTokenSilent(_scopes, accounts.FirstOrDefault())
                .ExecuteAsync();
            
            return result;
        }
        catch (MsalUiRequiredException)
        {
            // Acquire token interactively
            var result = await _app.AcquireTokenInteractive(_scopes)
                .ExecuteAsync();
            
            return result;
        }
    }

    public async Task<AuthenticationResult> GetTokenAsync()
    {
        var accounts = await _app.GetAccountsAsync();
        
        return await _app.AcquireTokenSilent(_scopes, accounts.FirstOrDefault())
            .ExecuteAsync();
    }

    public async Task SignOutAsync()
    {
        var accounts = await _app.GetAccountsAsync();
        
        foreach (var account in accounts)
        {
            await _app.RemoveAsync(account);
        }
    }
}
```

## Token Types

### 1. Access Token

**Used to access protected resources**:

```json
{
  "aud": "https://graph.microsoft.com",
  "iss": "https://sts.windows.net/{tenantId}/",
  "iat": 1234567890,
  "nbf": 1234567890,
  "exp": 1234571490,
  "acr": "1",
  "aio": "...",
  "appid": "00001111-aaaa-2222-bbbb-3333cccc4444",
  "scp": "User.Read Mail.Read"
}
```

## Свойства Access Token

Access token используется для доступа к защищённым API и выдаётся после успешной аутентификации и авторизации.

### Основные свойства

- **Короткий срок жизни**  
  Обычно действует около 1 часа. После истечения требуется получение нового токена (через refresh token или повторный flow).

- **Передаётся в заголовке Authorization**  
  Используется в HTTP-запросах для доступа к API.

- **Содержит claims**  
  Включает утверждения (claims) о пользователе и приложении:
   - идентификатор пользователя
   - tenant
   - роли
   - разрешения

- **Scopes определяют права доступа**  
  Токен содержит scopes, которые указывают, к каким ресурсам разрешён доступ.

---

## Что важно понимать

- Access token предназначен для API, а не для самого клиента.
- API должно проверять валидность токена и его claims.
- Scopes реализуют принцип наименьших привилегий.
- Токен подписывается и проверяется с использованием публичных ключей.

---

## Важно для AZ-204

- Access token ≠ ID token.  
  ID token используется для аутентификации пользователя, access token — для доступа к API.

- Срок жизни токена ограничен по соображениям безопасности.
- Permissions задаются через scopes или app roles.

> 🎯 Частый экзаменационный вопрос: какой токен используется для вызова API? Ответ — access token.


### 2. ID Token

**Contains user identity information**:

```json
{
  "aud": "00001111-aaaa-2222-bbbb-3333cccc4444",
  "iss": "https://login.microsoftonline.com/{tenantId}/v2.0",
  "iat": 1234567890,
  "exp": 1234571490,
  "name": "John Doe",
  "preferred_username": "john@contoso.com",
  "oid": "00000000-0000-0000-0000-000000000000",
  "sub": "AAAAAAAAAAAAAAAAAAAAAIkzqFVrSaSaFHy782bbtaQ",
  "tid": "11111111-1111-1111-1111-111111111111"
}
```

## Свойства ID Token

ID token используется для подтверждения личности пользователя после успешной аутентификации.

### Основные свойства

- **Формат JWT (JSON Web Token)**  
  Представляет собой подписанный токен в формате JSON Web Token.

- **Содержит пользовательские claims**  
  Включает информацию о пользователе:
   - идентификатор (sub)
   - имя
   - email
   - tenant
   - время аутентификации

- **Используется для проверки аутентификации**  
  Подтверждает, что пользователь успешно вошёл в систему.

- **Не должен использоваться для авторизации**  
  Не предназначен для проверки прав доступа к API.

---

## Важно понимать

- ID token предназначен для клиента (приложения), а не для API.
- Используется для создания пользовательской сессии.
- Подписывается Microsoft Identity Platform и должен проверяться по подписи и аудитории.
- Содержит минимальный набор информации для подтверждения личности.

---

## Отличие от Access Token

| ID Token | Access Token |
|-----------|--------------|
| Для аутентификации | Для доступа к API |
| Предназначен клиенту | Предназначен API |
| Не используется для проверки прав | Используется для проверки scopes и ролей |

---

## Важно для AZ-204

- ID token подтверждает, кто пользователь.
- Access token определяет, что пользователь может делать.
- Использование ID token для вызова API — ошибка архитектуры.

> 🎯 Экзамен часто проверяет понимание различия между authentication и authorization.


### 3. Refresh Token

**Used to get new access tokens**:

```
Refresh tokens are long-lived (90 days to 1 year)
Opaque to application
Stored securely
Used to renew expired access tokens
```

## Supported Scenarios

### 1. Web Application

**Server-side app** with users:

```csharp
// ASP.NET Core
services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"));
```

### 2. Single-Page Application (SPA)

**JavaScript app** in browser:

```javascript
const msalConfig = {
    auth: {
        clientId: "your-client-id",
        authority: "https://login.microsoftonline.com/your-tenant-id",
        redirectUri: "http://localhost:3000"
    }
};

const msalInstance = new msal.PublicClientApplication(msalConfig);

// Acquire token
const loginResponse = await msalInstance.loginPopup({
    scopes: ["User.Read"]
});
```

### 3. Mobile/Desktop Application

**Native apps** on devices:

```csharp
var app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri("http://localhost")
    .Build();

var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();
```

### 4. Daemon/Service

**Background service** without user:

```csharp
var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)
    .WithAuthority(new Uri($"https://login.microsoftonline.com/{tenantId}"))
    .Build();

var result = await app.AcquireTokenForClient(scopes)
    .ExecuteAsync();
```

### 5. Web API

**Protected backend API**:

```csharp
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(Configuration.GetSection("AzureAd"));

// In controller
[Authorize]
[ApiController]
public class WeatherForecastController : ControllerBase
{
    // Protected endpoints
}
```

## Common Scopes

### Microsoft Graph API

```
https://graph.microsoft.com/.default
https://graph.microsoft.com/User.Read
https://graph.microsoft.com/Mail.Send
https://graph.microsoft.com/Calendars.Read
https://graph.microsoft.com/Files.ReadWrite
```

### OpenID Connect

```
openid               - Basic sign-in
profile              - User profile info
email                - Email address
offline_access       - Refresh token
```

### Azure Services

```
https://management.azure.com/.default           - Azure Management
https://vault.azure.net/.default                - Key Vault
https://storage.azure.com/.default              - Azure Storage
```

## Best Practices

### 1. Use MSAL Libraries

```csharp
// ✅ Good: Use MSAL
var app = PublicClientApplicationBuilder.Create(clientId).Build();

// ❌ Bad: Manual OAuth implementation
// Complex, error-prone, missing security features
```

### 2. Token Caching

```csharp
// ✅ Good: Try cache first
var result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync();

// ❌ Bad: Always acquire interactively
// Poor UX, unnecessary prompts
```

### 3. Request Minimum Scopes

```csharp
// ✅ Good: Request only what you need
var scopes = new[] { "User.Read" };

// ❌ Bad: Request all permissions
var scopes = new[] { "User.Read", "Mail.ReadWrite", "Files.ReadWrite.All" };
```

### 4. Handle Token Expiration

```csharp
// ✅ Good: Refresh automatically
try
{
    var result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync();
}
catch (MsalUiRequiredException)
{
    var result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();
}
```

### 5. Secure Token Storage

```csharp
// ✅ Good: Use token cache serialization
// MSAL handles secure storage automatically

// ❌ Bad: Store tokens in plain text
// Security risk, token theft
```

# Critical Notes

- 💡 **OAuth 2.0 & OpenID Connect** — отраслевые стандарты аутентификации и авторизации
- 🎯 **MSAL** — Microsoft Authentication Libraries для разных платформ
- ✅ **Поддержка нескольких типов учётных записей** — рабочие, учебные, личные и социальные
- ⚠️ **Endpoint** — `login.microsoftonline.com/{tenant}/oauth2/v2.0`
- 🔄 **Типы токенов** — Access token (вызовы API), ID token (идентификация), Refresh token (обновление)
- 📊 **Azure Portal** — регистрация и управление приложениями
- 💡 **Microsoft Graph API** — доступ к данным Microsoft 365
- ✅ **Современные механизмы безопасности** — Passwordless, MFA, Conditional Access
- ⚠️ **Scopes** — определяют разрешения
- 🔒 **Кеширование токенов** — сначала попытка silent-получения
- 🎯 **DevOps-автоматизация** — через Microsoft Graph API и PowerShell
- 💡 **Типы приложений** — Web, SPA, mobile, desktop, daemon, API
- ⚠️ **Best practice** — использовать MSAL, запрашивать минимальные scopes, обрабатывать истечение токена

---

# Exam Tips (AZ-204)

## Основы платформы

- Microsoft identity platform — сервис аутентификации на базе OAuth 2.0 и OpenID Connect.
- Основные компоненты:
   - Authentication service
   - MSAL
   - Endpoints
   - Azure Portal
   - Configuration API

---

## MSAL

- Библиотеки доступны для .NET, JavaScript, Java, Python.
- Управляют получением, кешированием и обновлением токенов.
- Рекомендуется использовать вместо прямой реализации OAuth flow.

---

## Endpoints

- Используется endpoint авторизации формата:
  `login.microsoftonline.com/{tenant}/oauth2/v2.0/authorize`

- Tenant может быть:
   - конкретный ID каталога
   - `common`
   - `organizations`
   - `consumers`

---

## Типы токенов

- **Access token**
   - Короткий срок жизни (примерно 1 час)
   - Используется для вызова API

- **ID token**
   - Формат JWT
   - Содержит claims пользователя

- **Refresh token**
   - Более долгий срок жизни
   - Используется для получения нового access token

---

## Типы идентификаций

- Work/School — Microsoft Entra ID
- Personal — Microsoft account
- Social — B2C
- Customer — External ID

---

## Scopes

- Определяют разрешения на доступ к ресурсам.
- Примеры: доступ к профилю пользователя, почте, базовой информации.
- Реализуют принцип наименьших привилегий.

---

## Управление через Azure Portal

- App registration
- Настройка redirect URI
- Создание client secrets
- Управление сертификатами
- Назначение API permissions

---

## Microsoft Graph

- Используется для программной настройки и автоматизации.
- Позволяет управлять пользователями, группами, приложениями.

---

## Современные возможности безопасности

- Passwordless authentication
- Multi-Factor Authentication
- Conditional Access
- Risk detection

---

## Сценарии приложений

- Web application
- SPA
- Mobile app
- Desktop app
- Daemon/service
- Web API

---

## Рекомендации (Best Practices)

- Использовать MSAL.
- Кешировать токены.
- Запрашивать минимально необходимые scopes.
- Обрабатывать истечение токенов.
- Разделять public и confidential клиенты.

---

## Типы клиентов в MSAL

- **PublicClientApplication**  
  Используется для приложений с участием пользователя.

- **ConfidentialClientApplication**  
  Используется для сервисов и daemon-приложений без пользовательского интерфейса.

---

## Методы получения токена

- **AcquireTokenInteractive**  
  Интерактивный вход пользователя через UI.

- **AcquireTokenSilent**  
  Попытка получить токен из кеша без показа UI.

- **MsalUiRequiredException**  
  Исключение возникает, если требуется интерактивный вход.

---

## Часто проверяется на экзамене

- Различие между access, ID и refresh токенами.
- Когда используется public vs confidential client.
- Как работают scopes.
- Почему важно кеширование токенов.
- Как обрабатывать истечение срока действия токена.

---


[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-microsoft-identity-platform/2-microsoft-identity-platform-overview)
