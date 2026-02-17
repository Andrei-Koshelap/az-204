# Authentication and Authorization в Azure Container Apps

## Ключевые понятия (Key Concepts)

- **Built-in auth** — встроенная аутентификация с минимальным кодом
- **Identity providers** — Microsoft, Google, Facebook, GitHub, X, OpenID Connect
- **Sidecar-архитектура** — аутентификация обрабатывается платформенным middleware
- **Server-directed flow** — браузерная аутентификация
- **Client-directed flow** — аутентификация для мобильных приложений и API

---

# Built-In Authentication

## Что это такое?

**Аутентификация на уровне платформы** без необходимости писать собственный код.

- Поддержка федеративных identity providers
- Работает как sidecar-контейнер
- Перехватывает HTTP-запросы до попадания в приложение
- Управляет сессией пользователя
- Добавляет информацию о пользователе в HTTP-заголовки

---

## Преимущества

✅ Не требует реализации аутентификации в коде  
✅ Поддержка нескольких провайдеров  
✅ Управление токенами выполняется платформой  
✅ Информация о пользователе передаётся через заголовки  
✅ Работает только через HTTPS

⚠️ Важно: Работает **только по HTTPS**. Необходимо отключить `allowInsecure` в настройках ingress.

---

# Identity Providers

## Поддерживаемые провайдеры

| Провайдер | Endpoint входа | Описание |
|------------|----------------|------------|
| **Microsoft Identity Platform** | `/.auth/login/aad` | Microsoft Entra ID (Azure AD) |
| **Facebook** | `/.auth/login/facebook` | Аккаунты Facebook |
| **GitHub** | `/.auth/login/github` | Аккаунты GitHub |
| **Google** | `/.auth/login/google` | Аккаунты Google |
| **X (Twitter)** | `/.auth/login/twitter` | Аккаунты X/Twitter |
| **OpenID Connect** | `/.auth/login/<providerName>` | Любой OIDC-провайдер |

---

## Что важно понимать

- Аутентификация выполняется до попадания запроса в приложение
- Приложение получает данные пользователя через HTTP-заголовки
- Можно подключать корпоративные или внешние identity providers
- HTTPS обязателен для работы встроенной аутентификации

---

## Экзаменационный акцент (AZ-204)

- Built-in authentication не требует кода
- Работает через sidecar-механизм
- Поддерживает несколько identity providers
- Работает только через HTTPS
- Информация о пользователе передаётся через headers

### Provider Configuration

#### Microsoft Identity Platform (Entra ID)
```bash
# Configure Microsoft provider
az containerapp auth microsoft update \
  --name myapp \
  --resource-group myResourceGroup \
  --client-id <app-id> \
  --client-secret-setting-name microsoft-provider-authentication-secret \
  --issuer https://login.microsoftonline.com/<tenant-id>/v2.0
```

#### Generic OpenID Connect
```bash
# Configure custom OIDC provider
az containerapp auth openid-connect add \
  --name myapp \
  --resource-group myResourceGroup \
  --provider-name auth0 \
  --client-id <client-id> \
  --client-secret-setting-name auth0-secret \
  --openid-issuer https://<tenant>.auth0.com/
```

## Feature Architecture

### How It Works

**Sidecar container** handles authentication:

```
Internet
    ↓
Ingress (HTTPS)
    ↓
Auth Sidecar Container (middleware)
├── Authenticates user
├── Manages session
├── Validates tokens
└── Injects identity headers
    ↓
Application Container
└── Receives authenticated requests
```
## Auth Middleware Responsibilities

Платформенный middleware выполняет следующие функции:

✅ **Аутентификация пользователей** — проверка учётных данных через identity provider  
✅ **Управление сессиями** — работа с токенами и cookie  
✅ **Инъекция идентификации** — добавление информации о пользователе в HTTP-заголовки  
✅ **Проверка токенов** — валидация JWT

---

## Нет интеграции внутри процесса приложения

⚠️ **Отдельный контейнер** — механизм аутентификации работает изолированно от кода приложения  
💡 **Идентификация через заголовки** — приложение получает данные пользователя из HTTP-заголовков

---

# Authentication Flows

## 1. Server-Directed Flow (Browser-приложения)

**Платформа управляет процессом входа:**

- Пользователь обращается к защищённому endpoint
- Платформа перенаправляет на страницу входа identity provider
- После успешной аутентификации создаётся сессия
- Запрос возвращается в приложение с информацией о пользователе

---

## Когда использовать Server-Directed Flow

- Веб-приложения с браузерной аутентификацией
- UI-приложения с редиректом на страницу входа
- Минимизация логики аутентификации в коде

---

## Экзаменационный акцент (AZ-204)

- Аутентификация выполняется вне кода приложения
- Middleware добавляет identity в HTTP-заголовки
- Browser-сценарии → Server-Directed Flow
- JWT валидируется платформой


```
1. User → GET /protected-resource
2. Platform → Redirect to /.auth/login/<provider>
3. User → Sign in at provider
4. Provider → Return to /.auth/login/<provider>/callback
5. Platform → Set auth cookie
6. Platform → Redirect to /protected-resource
7. App → Receives request with identity headers
```

## Использовать для (Server-Directed Flow)

- Веб-приложения
- Браузерные приложения
- Server-side rendered приложения

**Пример:** пользователь нажимает кнопку «Sign in with Google», платформа выполняет редирект и управляет всей аутентификацией.

---

# 2. Client-Directed Flow (Mobile / API-приложения)

## Приложение выполняет вход, платформа проверяет токен

В этом сценарии:

- Клиентское приложение (mobile, SPA, API client) самостоятельно получает токен у identity provider
- Токен (обычно JWT) отправляется в запросе к Container App
- Платформа валидирует токен
- При успешной проверке запрос передаётся в приложение

---

## Когда использовать Client-Directed Flow

- Мобильные приложения
- SPA (Single Page Applications)
- Чистые API без браузерных редиректов
- Сценарии с OAuth2 / OpenID Connect

---

## Что важно понимать

- Приложение отвечает за получение access token
- Платформа отвечает за валидацию токена
- Токен передаётся в заголовке Authorization
- Нет редиректа на страницу входа

---

## Экзаменационный акцент (AZ-204)

- Browser → Server-Directed Flow
- Mobile / API → Client-Directed Flow
- В Client-flow приложение получает токен само
- Платформа выполняет валидацию JWT


```
1. Mobile App → Sign in with provider SDK
2. Provider → Return access token to app
3. Mobile App → POST token to /.auth/login/<provider>
4. Platform → Validate token with provider
5. Platform → Return session token
6. Mobile App → Include session token in requests
7. App → Receives authenticated requests
```

## Использовать для (Client-Directed Flow)

- Нативные мобильные приложения
- Single-page applications (SPA)
- Приложения без браузера
- API-клиенты

**Пример:** мобильное приложение использует Google SDK для получения токена, затем отправляет его в Azure Container Apps, где платформа выполняет валидацию.

---

## Что происходит в этом сценарии

1. Клиент получает access token у identity provider
2. Токен передаётся в запросе к Container App
3. Платформа проверяет подпись и валидность JWT
4. При успешной проверке запрос передаётся в приложение

---

## Экзаменационный акцент (AZ-204)

- Mobile / SPA → Client-Directed Flow
- Клиент получает токен самостоятельно
- Container Apps валидирует JWT
- Нет редиректов, как в браузерном сценарии


## Access Restrictions

### Require Authentication
```bash
# Require authentication for all requests
az containerapp auth update \
  --name myapp \
  --resource-group myResourceGroup \
  --unauthenticated-client-action RedirectToLoginPage

# Options:
# - RedirectToLoginPage: Redirect to sign-in
# - Return401: Return HTTP 401
# - Return403: Return HTTP 403
# - AllowAnonymous: Allow unauthenticated requests
```

### Allow Unauthenticated Access
```bash
# Allow anonymous access (validate tokens if present)
az containerapp auth update \
  --name myapp \
  --resource-group myResourceGroup \
  --unauthenticated-client-action AllowAnonymous
```

# Configuration Comparison (Режимы поведения аутентификации)

| Режим | Поведение | Сценарий |
|--------|------------|------------|
| **RedirectToLoginPage** | Автоматический редирект на страницу входа | Web-приложения |
| **Return401** | Возврат 401 Unauthorized | API, SPA |
| **Return403** | Возврат 403 Forbidden | API |
| **AllowAnonymous** | Необязательная аутентификация | Смешанный публичный/приватный контент |

---

## Что важно понимать

- **RedirectToLoginPage** подходит для браузерных приложений
- **Return401** используется, когда клиент сам управляет логикой входа
- **Return403** — доступ запрещён даже при наличии токена
- **AllowAnonymous** позволяет комбинировать защищённые и публичные маршруты

---

# Identity Information in Headers

## Стандартные заголовки, добавляемые платформой

| Заголовок | Описание |
|------------|------------|
| `X-MS-CLIENT-PRINCIPAL-ID` | Уникальный идентификатор пользователя |
| `X-MS-CLIENT-PRINCIPAL-NAME` | Имя пользователя |
| `X-MS-CLIENT-PRINCIPAL-IDP` | Используемый identity provider |
| `X-MS-CLIENT-PRINCIPAL` | Base64-закодированный JSON с claims пользователя |
| `X-MS-TOKEN-<provider>-ACCESS-TOKEN` | Access token от провайдера |
| `X-MS-TOKEN-<provider>-ID-TOKEN` | ID token от провайдера |

---

## Что это означает для приложения

- Приложение может читать данные пользователя из HTTP-заголовков
- Нет необходимости самостоятельно валидировать JWT
- Claims доступны через декодирование `X-MS-CLIENT-PRINCIPAL`
- Можно использовать access token для вызова внешних API

---

## Экзаменационный акцент (AZ-204)

- Built-in auth передаёт identity через HTTP headers
- 401 → не аутентифицирован
- 403 → доступ запрещён
- RedirectToLoginPage → для web-приложений
- API-сценарии → Return401


### Reading Identity in Application

#### Node.js/Express
```javascript
app.get('/profile', (req, res) => {
  const userId = req.headers['x-ms-client-principal-id'];
  const userName = req.headers['x-ms-client-principal-name'];
  const provider = req.headers['x-ms-client-principal-idp'];
  
  res.json({
    id: userId,
    name: userName,
    provider: provider
  });
});
```

#### Python/Flask
```python
@app.route('/profile')
def profile():
    user_id = request.headers.get('X-MS-CLIENT-PRINCIPAL-ID')
    user_name = request.headers.get('X-MS-CLIENT-PRINCIPAL-NAME')
    provider = request.headers.get('X-MS-CLIENT-PRINCIPAL-IDP')
    
    return {
        'id': user_id,
        'name': user_name,
        'provider': provider
    }
```

#### .NET/C#
```csharp
[HttpGet("profile")]
public IActionResult GetProfile()
{
    var userId = Request.Headers["X-MS-CLIENT-PRINCIPAL-ID"];
    var userName = Request.Headers["X-MS-CLIENT-PRINCIPAL-NAME"];
    var provider = Request.Headers["X-MS-CLIENT-PRINCIPAL-IDP"];
    
    return Ok(new {
        Id = userId,
        Name = userName,
        Provider = provider
    });
}
```

## Token Management

### Access Tokens
```bash
# Access provider tokens via special endpoint
GET /.auth/me

# Returns:
# - User information
# - Access tokens for configured providers
# - ID tokens
# - Refresh tokens (if available)
```

### Token Refresh
```bash
# Refresh tokens automatically
GET /.auth/refresh

# Platform refreshes tokens with provider
# Returns new tokens
```

### Logout
```bash
# Sign out user
GET /.auth/logout

# Clears session
# Optionally redirects to provider logout
```

## Configuration Example

### Complete Auth Setup
```bash
# 1. Enable Microsoft authentication
az containerapp auth microsoft update \
  --name myapp \
  --resource-group myResourceGroup \
  --client-id $CLIENT_ID \
  --client-secret-setting-name microsoft-secret \
  --issuer https://login.microsoftonline.com/$TENANT_ID/v2.0

# 2. Store client secret
az containerapp secret set \
  --name myapp \
  --resource-group myResourceGroup \
  --secrets microsoft-secret=$CLIENT_SECRET

# 3. Require authentication
az containerapp auth update \
  --name myapp \
  --resource-group myResourceGroup \
  --unauthenticated-client-action RedirectToLoginPage

# 4. Configure allowed redirect URLs in Azure AD:
# https://myapp.<environment-id>.<region>.azurecontainerapps.io/.auth/login/aad/callback
```

## Best Practices

### 1. Always Use HTTPS
```bash
# Disable insecure HTTP
az containerapp ingress update \
  --name myapp \
  --resource-group myResourceGroup \
  --allow-insecure false
```

### 2. Store Secrets Securely
```bash
# Store provider secrets in Container App secrets
az containerapp secret set \
  --name myapp \
  --resource-group myResourceGroup \
  --secrets provider-secret=$SECRET
```

### 3. Use Appropriate Flow
- **Server-directed**: Web apps with browsers
- **Client-directed**: Mobile apps, SPAs

### 4. Configure Token Expiration
```bash
# Set session timeout
az containerapp auth update \
  --name myapp \
  --resource-group myResourceGroup \
  --token-store true \
  --sas-url-secret session-secret
```

### 5. Implement Logout
```html
<!-- Add logout link -->
<a href="/.auth/logout">Sign Out</a>
```

# Limitations (Ограничения встроенной аутентификации)

❌ **Требуется HTTPS** — не работает по HTTP  
❌ **External ingress** — приложение должно быть доступно извне  
❌ **Нет офлайн-валидации** — требуется подключение к identity provider  
❌ **Session cookies** — для некоторых mobile-сценариев лучше использовать client-directed flow

---

# Critical Notes

- 💡 **Built-in auth** — не требует кода, всё управляется платформой
- ⚠️ **Только HTTPS** — необходимо отключить `allowInsecure`
- 🎯 **Sidecar-архитектура** — аутентификация выполняется в отдельном контейнере
- ✅ **Identity headers** — информация о пользователе передаётся через HTTP-заголовки
- 📊 **Server-directed flow** — для браузерных приложений (редирект к провайдеру)
- 🔄 **Client-directed flow** — для мобильных приложений (клиент получает токен)
- 🔒 **Контроль доступа** — Require auth, AllowAnonymous, 401/403
- ⚠️ **Несколько провайдеров** — можно подключить разные identity systems

---

# Exam Tips (AZ-204)

## Основы

- Built-in authentication — федеративная аутентификация без кода
- Поддерживаемые провайдеры:
    - Microsoft
    - Google
    - Facebook
    - GitHub
    - X
    - OpenID Connect

---

## Endpoint входа

- `/.auth/login/<provider>`

---

## Архитектура

- Аутентификация работает через sidecar
- Нет интеграции в коде приложения
- Identity передаётся только через HTTP headers

---

## Authentication Flows

- **Server-directed flow** → браузерные приложения
- **Client-directed flow** → мобильные приложения / API

---

## Identity Headers

- `X-MS-CLIENT-PRINCIPAL-ID`
- `X-MS-CLIENT-PRINCIPAL-NAME`
- другие `X-MS-*` заголовки

---

## Access Restriction Modes

- RedirectToLoginPage
- Return401
- Return403
- AllowAnonymous

---

## HTTPS

- Обязательно отключить `allowInsecure`
- Работает только через HTTPS

---

## Token Endpoints

- `/.auth/me` — информация о пользователе
- `/.auth/refresh` — обновление токена
- `/.auth/logout` — выход

---

## Ответственность платформы

- Аутентификация пользователя
- Управление сессией
- Инъекция identity в headers
- Валидация JWT

---

## Частые экзаменационные ловушки

- Нет HTTPS → аутентификация не работает
- Identity передаётся через заголовки, не через SDK
- Browser → Server-directed flow
- Mobile/API → Client-directed flow
- Встроенная аутентификация не требует кода


[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/5-container-apps-authentication)
