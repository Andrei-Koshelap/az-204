# Conditional Access

## Ключевые понятия

- **Policy-based security** — безопасность на основе политик
- **Context-aware** — учитывает контекст (локация, устройство, риск)
- **Enterprise-функция** — требует лицензии Microsoft Entra ID P1 или P2
- **Обработка в коде** — приложения могут столкнуться с дополнительными проверками (CA challenges)

---

# Что такое Conditional Access?

**Функция Microsoft Entra ID** для защиты приложений и сервисов.

### Назначение

- Защищать ресурсы на основе условий.
- Применять политики в зависимости от контекста.
- Работать на этапе аутентификации или доступа к ресурсу.
- Управляться администраторами через централизованные политики.

> 💡 Conditional Access = «если выполняются условия → применить контроль».

---

# Сигналы Conditional Access

| Тип сигнала | Примеры |
|-------------|----------|
| **User/Group** | Конкретные пользователи, группы, роли |
| **Location** | Доверенные локации, страны, диапазоны IP |
| **Device** | Соответствующие требованиям или управляемые устройства |
| **Application** | Конкретные приложения или категории |
| **Risk** | Риск входа, риск пользователя |
| **Client app** | Браузер, мобильное или десктопное приложение |

---

# Контроли Conditional Access

| Контроль | Описание |
|----------|----------|
| **Block access** | Полная блокировка входа |
| **Grant access** | Разрешить доступ при выполнении условий |
| **Require MFA** | Обязательная многофакторная аутентификация |
| **Require compliant device** | Требуется управляемое устройство |
| **Require hybrid Azure AD joined** | Устройство должно быть доменно присоединено |
| **Require approved client app** | Разрешены только определённые приложения |
| **Require app protection policy** | Требуется политика защиты приложений |
| **Require password change** | Принудительная смена пароля |
| **Terms of use** | Необходимо принять условия использования |

---

## Как это работает

1. Пользователь пытается войти в систему.
2. Entra ID анализирует сигналы.
3. Применяются соответствующие политики.
4. Доступ разрешается, ограничивается или блокируется.

---

## Важно для AZ-204

- Conditional Access применяется на уровне tenant.
- Может потребовать MFA даже если приложение этого не запрашивало.
- Может блокировать доступ в зависимости от контекста.
- Требует соответствующей лицензии.
- Приложения должны корректно обрабатывать возможные дополнительные шаги аутентификации.

> 🎯 Частый экзаменационный вопрос:  
Почему пользователь должен пройти MFA, если приложение не запрашивает его?  
Ответ — из-за политики Conditional Access.


## Common Conditional Access Scenarios

### 1. Multifactor Authentication

**Require MFA** based on conditions:

```
Policy: Require MFA
Conditions:
  - When: User signs in from outside corporate network
  - What: All cloud apps
  - Who: All users

Controls:
  - Grant access
  - Require multi-factor authentication
```

**Example scenario**:
- User signs in from office IP → No MFA required
- User signs in from home → MFA required
- User signs in from coffee shop → MFA required

### 2. Device Compliance

**Only allow compliant devices**:

```
Policy: Require Compliant Device
Conditions:
  - When: Accessing sensitive data
  - What: Microsoft 365 apps
  - Who: All users

Controls:
  - Grant access
  - Require device to be marked as compliant
```

## Пример сценария Conditional Access

Рассмотрим политику, основанную на соответствии устройства требованиям безопасности.

### Сценарий доступа

- **Личное устройство (не зарегистрировано / не управляется)** → ❌ Доступ заблокирован
- **Корпоративное устройство (управляется через Intune)** → ✅ Доступ разрешён
- **Устройство, не соответствующее политикам (нарушения безопасности)** → ❌ Доступ заблокирован

---

## Что происходит технически

- Entra ID проверяет статус устройства во время аутентификации.
- Если политика требует соответствие (compliant device), доступ предоставляется только управляемым и соответствующим требованиям устройствам.
- Несоответствие политике автоматически приводит к блокировке или дополнительным требованиям.

---

## Почему это важно

- Позволяет защитить корпоративные данные.
- Предотвращает доступ с небезопасных или скомпрометированных устройств.
- Интегрируется с Microsoft Intune и управлением устройствами.

---

## Важно для AZ-204

- Conditional Access может зависеть от состояния устройства.
- Политики применяются до выдачи токена.
- Даже при корректной аутентификации доступ может быть заблокирован.
- Compliant device — это устройство, соответствующее требованиям безопасности организации.

> 🎯 Частый экзаменационный вопрос:  
Почему вход успешен, но доступ к ресурсу запрещён?  
Ответ — из-за политики Conditional Access.


### 3. Location-Based Access

**Restrict by location**:

```
Policy: Block Specific Countries
Conditions:
  - When: Sign-in from untrusted locations
  - What: All cloud apps
  - Who: All users

Controls:
  - Block access
```

**Example scenario**:
- Sign-in from US, UK, Canada → Allowed
- Sign-in from suspicious country → Blocked

### 4. Approved Client Apps

**Only allow specific apps**:

```
Policy: Require Approved Client App
Conditions:
  - When: Accessing Exchange Online
  - What: Office 365 Exchange Online
  - Who: All users

Controls:
  - Grant access
  - Require approved client app (Outlook mobile)
```

**Example scenario**:
- Outlook mobile app → Allowed
- Third-party mail app → Blocked

### 5. Risk-Based Access

**Based on sign-in risk**:

```
Policy: High Risk Sign-In
Conditions:
  - When: Sign-in risk is high
  - What: All cloud apps
  - Who: All users

Controls:
  - Grant access
  - Require multi-factor authentication
  - Require password change
```

## Индикаторы риска (Risk Indicators)

Microsoft Entra ID анализирует поведение пользователя и контекст входа.

Основные сигналы риска:

- **Atypical travel (impossible travel)**  
  Вход из двух географически удалённых мест за короткое время.

- **Anonymous IP address**  
  Использование Tor, VPN или анонимных прокси.

- **Malware-linked IP address**  
  IP-адрес, связанный с вредоносной активностью.

- **Unfamiliar sign-in properties**  
  Необычные параметры входа (устройство, браузер, локация).

- **Password spray attacks**  
  Массовые попытки подбора пароля.

- **Leaked credentials**  
  Учётные данные, обнаруженные в утечках.

> 💡 Эти сигналы могут инициировать MFA, блокировку или другие меры через Conditional Access.

---

# Влияние Conditional Access на приложения

## Когда CA влияет на ваше приложение

Conditional Access может потребовать изменений в коде в зависимости от сценария.

| Сценарий | Требуются изменения в коде |
|-----------|----------------------------|
| **Простое web-приложение** | ❌ Нет (редирект обрабатывает всё автоматически) |
| **Мобильное/десктопное приложение** | ❌ Нет (MSAL обрабатывает вызовы) |
| **SPA (single-page app)** | ⚠️ Возможно (корректная обработка через MSAL.js) |
| **On-behalf-of flow** | ✅ Да — необходимо обрабатывать CA challenges |
| **Доступ к нескольким сервисам** | ✅ Да — требуется обработка дополнительных требований |
| **Вызов защищённого API** | ✅ Да — приложение должно корректно реагировать на CA |

---

## Что такое CA Challenge

Если политика Conditional Access требует дополнительную проверку:

- MFA
- Соответствие устройства
- Дополнительную аутентификацию

Токен может быть отклонён, и приложение должно инициировать повторный интерактивный вход.

---

## Почему это важно

- CA применяется на уровне tenant, а не приложения.
- Даже корректно настроенное приложение может получать ошибки из-за CA.
- Backend-сервисы должны уметь корректно обрабатывать повторный запрос токена.

---

## Важно для AZ-204

- CA может потребовать повторную аутентификацию.
- MSAL автоматически обрабатывает большинство стандартных сценариев.
- On-behalf-of flow требует явной обработки ошибок и повторного запроса токена.
- Ошибка авторизации может быть вызвана политикой CA, а не неверной конфигурацией приложения.

> 🎯 Частый экзаменационный вопрос:  
Почему API возвращает ошибку, хотя токен получен?  
Ответ — из-за политики Conditional Access или дополнительного требования MFA.


### Scenarios Requiring Code Changes

#### 1. On-Behalf-Of Flow

**Middle-tier service** calling downstream API:

```
User → Web App → Middle-tier API → Downstream API
                      ↑
                 CA policy here
```

**Problem**: CA policy applied to downstream API, not middle tier

**Example**:

```csharp
// Middle-tier API
[Authorize]
[HttpGet("data")]
public async Task<IActionResult> GetData()
{
    try
    {
        // Try to call downstream API
        var result = await CallDownstreamApiAsync();
        return Ok(result);
    }
    catch (MsalUiRequiredException ex)
    {
        // CA challenge from downstream API
        // Need to pass challenge back to client
        
        // Extract claims challenge
        var claims = ex.Claims;
        
        // Return 401 with WWW-Authenticate header
        Response.Headers.Add("WWW-Authenticate", 
            $"Bearer error=\"insufficient_claims\", claims=\"{claims}\"");
        
        return Unauthorized();
    }
}
```

**Client handles challenge**:

```csharp
// Client (Web App)
try
{
    // Call middle-tier API
    var response = await httpClient.GetAsync("/data");
    
    if (response.StatusCode == HttpStatusCode.Unauthorized)
    {
        // Check for CA challenge
        var authHeader = response.Headers.WwwAuthenticate.FirstOrDefault();
        
        if (authHeader != null && authHeader.Parameter.Contains("claims"))
        {
            // Extract claims
            var claims = ExtractClaims(authHeader.Parameter);
            
            // Re-authenticate with claims
            var result = await app.AcquireTokenInteractive(scopes)
                .WithClaims(claims)
                .ExecuteAsync();
            
            // Retry with new token
            httpClient.DefaultRequestHeaders.Authorization = 
                new AuthenticationHeaderValue("Bearer", result.AccessToken);
            
            response = await httpClient.GetAsync("/data");
        }
    }
}
catch (Exception ex)
{
    // Handle error
}
```

#### 2. Multiple Services/Resources

**App accessing multiple APIs**:

```
User → App → Graph API ✓
      └────→ SharePoint API ← CA policy
```

**Problem**: CA policy on one service affects app flow

**Example**:

```csharp
public async Task AccessMultipleServicesAsync()
{
    // First service (no CA)
    var graphResult = await graphClient.Me.Request().GetAsync();
    
    try
    {
        // Second service (with CA policy)
        var sharePointResult = await sharePointClient.GetDataAsync();
    }
    catch (ServiceException ex) when (ex.StatusCode == HttpStatusCode.Unauthorized)
    {
        // Handle CA challenge
        if (ex.ResponseHeaders.Contains("WWW-Authenticate"))
        {
            var claims = ExtractClaimsFromHeader(ex.ResponseHeaders);
            
            // Re-authenticate with claims
            var result = await app.AcquireTokenInteractive(sharePointScopes)
                .WithClaims(claims)
                .ExecuteAsync();
            
            // Retry
            sharePointClient.AuthenticationProvider = 
                new DelegateAuthenticationProvider(async (request) =>
                {
                    request.Headers.Authorization = 
                        new AuthenticationHeaderValue("Bearer", result.AccessToken);
                });
            
            sharePointResult = await sharePointClient.GetDataAsync();
        }
    }
}
```

#### 3. Single-Page Apps (MSAL.js)

**SPA with CA policies**:

```javascript
// MSAL.js configuration
const msalConfig = {
    auth: {
        clientId: "your-client-id",
        authority: "https://login.microsoftonline.com/your-tenant-id"
    },
    cache: {
        cacheLocation: "localStorage"
    }
};

const msalInstance = new msal.PublicClientApplication(msalConfig);

// Acquire token with CA challenge handling
async function getTokenWithCA(scopes) {
    const account = msalInstance.getAllAccounts()[0];
    
    const tokenRequest = {
        scopes: scopes,
        account: account
    };
    
    try {
        // Try silent acquisition
        const response = await msalInstance.acquireTokenSilent(tokenRequest);
        return response.accessToken;
    } catch (error) {
        if (error instanceof msal.InteractionRequiredAuthError) {
            // CA challenge or consent required
            
            if (error.errorMessage.includes("AADSTS50076")) {
                // MFA required (CA policy)
                const response = await msalInstance.acquireTokenPopup(tokenRequest);
                return response.accessToken;
            }
        }
        throw error;
    }
}

// Call API with CA handling
async function callProtectedApi() {
    try {
        const token = await getTokenWithCA(["User.Read"]);
        
        const response = await fetch("https://graph.microsoft.com/v1.0/me", {
            headers: {
                Authorization: `Bearer ${token}`
            }
        });
        
        if (response.status === 401) {
            // Check for CA challenge
            const authHeader = response.headers.get("WWW-Authenticate");
            
            if (authHeader && authHeader.includes("claims")) {
                // Extract and handle claims challenge
                const claims = extractClaims(authHeader);
                
                const tokenRequest = {
                    scopes: ["User.Read"],
                    claims: claims
                };
                
                const result = await msalInstance.acquireTokenPopup(tokenRequest);
                
                // Retry with new token
                return await fetch("https://graph.microsoft.com/v1.0/me", {
                    headers: {
                        Authorization: `Bearer ${result.accessToken}`
                    }
                });
            }
        }
        
        return response;
    } catch (error) {
        console.error("Error calling API:", error);
    }
}
```

## Conditional Access Examples

### Example 1: Single-Tenant iOS App

**Scenario**: iOS app with CA policy requiring MFA

**No code changes needed**:

```swift
// User signs in
let result = try await app.acquireToken(
    with: parameters,
    for: account
)

// If CA policy requires MFA:
// 1. Microsoft identity platform detects CA policy
// 2. User automatically prompted for MFA
// 3. After MFA, authentication completes
// 4. App receives token

// MSAL handles CA automatically - no code changes
```

### Example 2: Multi-Tier App with CA on Downstream API

**Scenario**: Web app → Middle-tier → Downstream API with CA

**Code changes required**:

```csharp
// Web App (client)
public async Task<string> CallMiddleTierAsync()
{
    var token = await GetAccessTokenAsync();
    
    var request = new HttpRequestMessage(HttpMethod.Get, "https://api.contoso.com/data");
    request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", token);
    
    var response = await httpClient.SendAsync(request);
    
    if (response.StatusCode == HttpStatusCode.Unauthorized)
    {
        // Check for CA challenge
        var authHeader = response.Headers.WwwAuthenticate.FirstOrDefault();
        
        if (authHeader?.Parameter?.Contains("claims") == true)
        {
            // Handle CA challenge
            var claims = ExtractClaims(authHeader.Parameter);
            
            // Re-acquire token with claims
            var result = await app.AcquireTokenInteractive(scopes)
                .WithClaims(claims)
                .ExecuteAsync();
            
            // Retry request
            request.Headers.Authorization = 
                new AuthenticationHeaderValue("Bearer", result.AccessToken);
            
            response = await httpClient.SendAsync(request);
        }
    }
    
    return await response.Content.ReadAsStringAsync();
}

// Middle-tier API
[Authorize]
public async Task<IActionResult> GetData()
{
    try
    {
        // Call downstream API with on-behalf-of
        var result = await GetOnBehalfOfTokenAsync();
        var data = await CallDownstreamApiAsync(result.AccessToken);
        
        return Ok(data);
    }
    catch (MsalUiRequiredException ex)
    {
        // CA challenge from downstream API
        // Pass challenge back to client
        
        Response.Headers.Add("WWW-Authenticate", 
            $"Bearer error=\"insufficient_claims\", claims=\"{ex.Claims}\"");
        
        return new UnauthorizedResult();
    }
}
```

## Handling CA Challenges

### 1. Detect CA Challenge

**Check HTTP response**:

```csharp
if (response.StatusCode == HttpStatusCode.Unauthorized)
{
    var authHeader = response.Headers.WwwAuthenticate.FirstOrDefault();
    
    if (authHeader != null && authHeader.Parameter.Contains("claims"))
    {
        // CA challenge detected
        var claims = ExtractClaims(authHeader.Parameter);
    }
}
```

### 2. Extract Claims

**Parse WWW-Authenticate header**:

```csharp
private string ExtractClaims(string authenticateHeader)
{
    // Example header:
    // Bearer error="insufficient_claims", 
    //        claims="eyJhY2Nlc3NfdG9rZW4iOnsiYWNy..."
    
    var claimsMatch = Regex.Match(
        authenticateHeader, 
        "claims=\"([^\"]+)\""
    );
    
    if (claimsMatch.Success)
    {
        return claimsMatch.Groups[1].Value;
    }
    
    return null;
}
```

### 3. Re-Authenticate with Claims

**Include claims in token request**:

```csharp
var result = await app.AcquireTokenInteractive(scopes)
    .WithClaims(claims)  // Include CA claims
    .ExecuteAsync();

// New token satisfies CA policy
```

### 4. Retry Request

**Use new token**:

```csharp
request.Headers.Authorization = 
    new AuthenticationHeaderValue("Bearer", result.AccessToken);

var response = await httpClient.SendAsync(request);
```

## Common CA Policies

### Policy: Require MFA from Untrusted Locations

```
Conditions:
  Users: All users
  Cloud apps: All cloud apps
  Locations: Any location except trusted IPs

Controls:
  Grant: Require multi-factor authentication
```

### Policy: Block Legacy Authentication

```
Conditions:
  Users: All users
  Cloud apps: All cloud apps
  Client apps: Exchange ActiveSync, Other clients

Controls:
  Block access
```

### Policy: Require Compliant Device for Admins

```
Conditions:
  Users: Directory admin roles
  Cloud apps: All cloud apps
  Locations: Any location

Controls:
  Grant: Require device to be marked as compliant
  OR: Require hybrid Azure AD joined device
```

### Policy: Require Approved Apps for Mobile

```
Conditions:
  Users: All users
  Cloud apps: Office 365
  Device platforms: iOS, Android

Controls:
  Grant: Require approved client app
```

## Best Practices

### 1. Handle CA Challenges Gracefully

```csharp
// ✅ Good: Handle CA challenges
try
{
    var result = await CallApiAsync();
}
catch (UnauthorizedException ex) when (HasCAChallengeex))
{
    // Re-authenticate with claims
    await HandleCAChallenge(ex.Claims);
}

// ❌ Bad: Ignore CA challenges
// App will fail when CA policy is applied
```

### 2. Test with CA Policies

```
Testing checklist:
✅ Test without CA policies (baseline)
✅ Test with MFA requirement
✅ Test with device compliance
✅ Test with location restrictions
✅ Test on-behalf-of flow with CA
✅ Test multi-service access with CA
```

### 3. Use MSAL for Automatic Handling

```csharp
// ✅ Good: MSAL handles most CA scenarios automatically
var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();

// ❌ Bad: Manual OAuth implementation
// Won't handle CA challenges correctly
```

### 4. Document CA Requirements

```
App documentation:
- "This app supports Conditional Access policies"
- "Tested with MFA, device compliance, location restrictions"
- "Handles on-behalf-of CA challenges"
```

# Critical Notes

- 💡 **Conditional Access** — политика безопасности на уровне tenant для защиты приложений
- 🎯 **Сигналы** — локация, устройство, риск, пользователь, приложение
- ✅ **Контроли** — MFA, соответствующее требованиям устройство, блокировка доступа
- ⚠️ **Изменения в коде** — требуются для on-behalf-of flow, multi-service и некоторых SPA
- 🔄 **CA challenge** — HTTP 401 с заголовком `WWW-Authenticate`
- 📊 **Claims challenge** — дополнительные требования аутентификации
- 💡 **MSAL** — автоматически обрабатывает большинство стандартных сценариев
- ✅ **Best practice** — корректно обрабатывать CA challenges
- ⚠️ **Enterprise-функция** — требуется лицензия Microsoft Entra ID P1 или P2
- 🔒 **Тестирование** — проверять приложение с разными CA-политиками до продакшена

---

# Exam Tips (AZ-204)

## Основы

- Conditional Access — политика безопасности в Microsoft Entra ID.
- Цель — защита сервисов на основе условий.
- Политики применяются во время аутентификации или доступа к ресурсу.

---

## Сигналы (Signals)

- Локация (IP, страны, trusted locations)
- Состояние устройства (соответствие требованиям, управляемость)
- Риск входа и риск пользователя
- Тип клиентского приложения
- Конкретное приложение

---

## Контроли (Controls)

- Требование MFA
- Требование compliant device
- Разрешение только определённых приложений
- Блокировка доступа
- Принятие условий использования

---

## Когда требуются изменения в коде

### Требуются

- On-behalf-of flow
- Доступ к нескольким сервисам
- Некоторые SPA
- Вызов защищённых API с дополнительными требованиями

### Не требуются

- Простые web-приложения (редирект обрабатывает CA)
- Мобильные приложения с MSAL (автоматическая обработка)

---

## CA Challenge

- Возвращается как HTTP 401.
- Содержит заголовок `WWW-Authenticate`.
- Включает дополнительные `claims`, требуемые политикой CA.

### Обработка

- Извлечь claims из ответа.
- Выполнить повторную аутентификацию с передачей этих claims.
- Повторить запрос к ресурсу.

---

## On-Behalf-Of Flow

- Средний слой (middle tier) может получить CA challenge.
- Он обязан передать требование клиенту.
- Клиент должен пройти дополнительную аутентификацию.

---

## Multi-Service Access

- Разные сервисы могут иметь разные CA-политики.
- Приложение должно корректно обрабатывать multiple challenges.

---

## MSAL

- Автоматически обрабатывает большинство интерактивных сценариев.
- Упрощает повторную аутентификацию.
- Минимизирует необходимость ручной логики.

---

## Лицензирование

- Conditional Access требует Microsoft Entra ID Premium P1 или P2.
- Risk-based политики требуют соответствующего уровня лицензии.

---

## Что часто проверяется

- Что такое CA challenge.
- Когда требуется обработка HTTP 401.
- Разница между простым web-приложением и OBO flow.
- Как влияет CA на multi-service архитектуру.
- Почему токен отклонён несмотря на успешную аутентификацию.

---

> 🎯 Ключевая идея:  
> Conditional Access может изменить требования к аутентификации без изменений в коде приложения,  
> но сложные архитектурные сценарии должны корректно обрабатывать CA challenges.

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-microsoft-identity-platform/5-conditional-access)
