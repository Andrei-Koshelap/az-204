# Microsoft Authentication Library (MSAL) Overview

## Ключевые понятия

- **MSAL** — Microsoft Authentication Library
- **Кроссплатформенность** — поддержка .NET, JavaScript, Java, Python, Android, iOS
- **Управление токенами** — автоматическое кеширование и обновление
- **Два типа клиентов** — Public и Confidential

---

# Что такое MSAL?

**Microsoft Authentication Library** — библиотека для получения токенов безопасности.

### Назначение

- Аутентификация пользователей
- Получение токенов для доступа к защищённым Web API
- Интеграция с Microsoft identity platform

### Поддерживаемые API

- Microsoft Graph
- Microsoft API
- Сторонние API
- Собственные API

### Поддержка платформ

Работает на различных языках и фреймворках.

---

# Преимущества использования MSAL

| Преимущество | Описание |
|--------------|----------|
| **Без ручной реализации OAuth** | Нет необходимости напрямую реализовывать протокол OAuth |
| **Получение токенов** | Для пользователей или приложений |
| **Кеширование токенов** | Хранение токенов и автоматическое обновление |
| **Автоматическое обновление** | Обработка истечения срока действия |
| **Настройка аудитории** | Простая конфигурация tenant и audience |
| **Конфигурационный подход** | Настройка через параметры и конфиги |
| **Диагностика** | Поддержка логирования и понятные исключения |

---

# Почему использовать MSAL?

✅ Упрощает разработку и абстрагирует сложность OAuth  
✅ Автоматически обрабатывает истечение токенов  
✅ Обеспечивает единый API-подход на разных платформах  
✅ Реализует лучшие практики безопасности  
✅ Поддерживается Microsoft и готова к production

> 💡 MSAL — рекомендуемый способ работы с Microsoft identity platform.

---

# Поддерживаемые платформы

## Библиотеки MSAL

| Библиотека | Платформа / Фреймворк | Типовой сценарий |
|------------|------------------------|------------------|
| **MSAL.NET** | .NET, .NET Framework, .NET MAUI, Xamarin, UWP, WinUI | Desktop, mobile, web |
| **MSAL.js** | JavaScript/TypeScript, Vue, Ember, др. | Browser-приложения |
| **MSAL Angular** | Angular | Single-page apps |
| **MSAL React** | React, Next.js, Gatsby | React SPA |
| **MSAL Node** | Express, Electron, console apps | Server-side Node.js |
| **MSAL Java** | Windows, macOS, Linux | Java-приложения |
| **MSAL Python** | Windows, macOS, Linux | Python-приложения |
| **MSAL Android** | Android | Мобильные приложения |
| **MSAL iOS/macOS** | iOS, macOS | Apple-приложения |
| **MSAL Go** | Windows, macOS, Linux | Go-приложения (Preview) |

---

## Важно для AZ-204

- MSAL — основной инструмент для работы с токенами.
- Поддерживает public и confidential клиенты.
- Автоматически обрабатывает кеширование и refresh.
- Используется для получения access и ID токенов.
- Рекомендуется вместо ручной реализации OAuth.

> 🎯 Частый экзаменационный вопрос:  
Нужно ли вручную реализовывать OAuth 2.0 flow?  
Ответ — нет, используйте MSAL.

### Installation

**MSAL.NET (NuGet)**:

```bash
dotnet add package Microsoft.Identity.Client
```

**MSAL.js (npm)**:

```bash
npm install @azure/msal-browser
npm install @azure/msal-node
npm install @azure/msal-angular
npm install @azure/msal-react
```

**MSAL Python (pip)**:

```bash
pip install msal
```

**MSAL Java (Maven)**:

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>msal4j</artifactId>
    <version>1.14.0</version>
</dependency>
```

## Application Types and Scenarios

### Desktop Applications

**Windows, macOS, Linux**:

```csharp
// MSAL.NET example
var app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri("http://localhost")
    .Build();

var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();
```

**Use cases**:
- Native Windows applications
- Cross-platform desktop apps
- Command-line tools

### Mobile Applications

**Android and iOS**:

```csharp
// MSAL.NET with Xamarin/MAUI
var app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri($"msal{clientId}://auth")
    .WithParentActivityOrWindow(() => ParentWindow)
    .Build();

var result = await app.AcquireTokenInteractive(scopes)
    .WithParentActivityOrWindow(ParentWindow)
    .ExecuteAsync();
```

**Use cases**:
- Mobile apps (iOS, Android)
- Cross-platform mobile with MAUI/Xamarin

### Single-Page Applications (SPA)

**JavaScript, Angular, React**:

```javascript
// MSAL.js
import { PublicClientApplication } from "@azure/msal-browser";

const msalConfig = {
    auth: {
        clientId: "your-client-id",
        authority: "https://login.microsoftonline.com/your-tenant-id",
        redirectUri: "http://localhost:3000"
    }
};

const msalInstance = new PublicClientApplication(msalConfig);

// Acquire token
const loginResponse = await msalInstance.loginPopup({
    scopes: ["User.Read"]
});
```

**Use cases**:
- React applications
- Angular applications
- Vue.js applications
- Vanilla JavaScript apps

### Web Applications

**Server-side web apps**:

```csharp
// ASP.NET Core with Microsoft.Identity.Web
services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAd"));
```

**Use cases**:
- ASP.NET Core web apps
- Node.js Express apps
- Python Flask/Django apps
- Java Spring Boot apps

### Web APIs

**Protected backend APIs**:

```csharp
// ASP.NET Core Web API
services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApi(Configuration.GetSection("AzureAd"));

[Authorize]
[ApiController]
[Route("api/[controller]")]
public class DataController : ControllerBase
{
    [HttpGet]
    public IActionResult Get()
    {
        return Ok(new { data = "Protected data" });
    }
}
```

**Use cases**:
- REST APIs
- GraphQL APIs
- gRPC services

### Daemon/Service Applications

**Background services, no user**:

```csharp
// MSAL.NET confidential client
var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)
    .WithAuthority(new Uri($"https://login.microsoftonline.com/{tenantId}"))
    .Build();

var result = await app.AcquireTokenForClient(scopes)
    .ExecuteAsync();
```

## Типовые сценарии использования (App-Only)

Используются в сценариях без пользователя:

- Плановые задачи (Scheduled jobs)
- Фоновые сервисы (Background services)
- Автоматизированные скрипты
- Взаимодействие server-to-server

> 💡 В таких случаях обычно применяются Application permissions и confidential client.

---

# Authentication Flows

## 1️⃣ Authorization Code Flow

**Самый распространённый flow** — пользователь входит в систему, приложение получает authorization code и обменивает его на токен.

| Свойство | Значение |
|-----------|----------|
| **Flow** | Редирект пользователя → Вход → Получение кода → Обмен кода на токен |
| **Используется в** | Desktop, Mobile, SPA (с PKCE), Web-приложениях |
| **Безопасность** | Для public clients рекомендуется PKCE |
| **Пользователь присутствует** | Да |

---

## Как работает

1. Пользователь перенаправляется на страницу входа.
2. После успешной аутентификации приложение получает authorization code.
3. Приложение обменивает код на access token (и ID token).
4. При необходимости получает refresh token.

---

## Почему это основной flow

- Наиболее безопасный для приложений с пользователем.
- Поддерживает delegated permissions.
- Совместим с MFA и Conditional Access.
- Рекомендуется вместо implicit flow.

---

## Важно для AZ-204

- Authorization Code Flow используется для интерактивных приложений.
- PKCE обязателен для public clients.
- ID token используется для аутентификации.
- Access token используется для вызова API.
- Может возвращать refresh token при запросе соответствующего scope.

> 🎯 Частый экзаменационный вопрос:  
Какой flow рекомендуется для SPA вместо implicit?  
Ответ — Authorization Code Flow с PKCE.


**Example**:

```csharp
// Desktop/Mobile app
var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();
```

## 2️⃣ Client Credentials Flow

**Приложение действует от своего имени (без пользователя)**

| Свойство | Значение |
|-----------|----------|
| **Flow** | Приложение аутентифицируется с помощью секрета или сертификата → Получает токен |
| **Используется в** | Daemon-приложениях, фоновых сервисах |
| **Безопасность** | Требуется confidential client |
| **Пользователь присутствует** | Нет |

---

## Как работает

1. Приложение отправляет запрос в identity platform.
2. Аутентифицируется с использованием:
    - Client secret
    - Сертификата
3. Получает access token.
4. Использует токен для вызова защищённого API.

---

## Особенности

- Используются **Application permissions**.
- Требуется **admin consent**.
- Токен содержит только claims приложения (без пользователя).
- Refresh token обычно не используется — приложение просто запрашивает новый access token.

---

## Когда использовать

- Фоновые задачи
- Интеграционные сервисы
- Server-to-server взаимодействие
- Автоматизированные процессы

---

## Важно для AZ-204

- Нет пользователя — значит нет delegated permissions.
- Используется confidential client.
- Требуется client secret или сертификат.
- Подходит для daemon-сценариев.
- Не поддерживает интерактивную аутентификацию.

Работает без пользователя
Использует application permissions
Это machine-to-machine flow.

> 🎯 Частый экзаменационный вопрос:  
Как реализовать доступ к API без пользователя?  
Ответ — использовать Client Credentials Flow.


**Example**:

```csharp
// Daemon application
var result = await app.AcquireTokenForClient(scopes)
    .ExecuteAsync();
```

### 3. On-Behalf-Of (OBO) Flow

## 3️⃣ On-Behalf-Of (OBO) Flow

**Промежуточный сервис (middle-tier)** вызывает downstream API от имени пользователя.

| Свойство | Значение |
|-----------|----------|
| **Flow** | API получает пользовательский токен → Обменивает его на токен для downstream API |
| **Используется в** | Web API, вызывающем другое API |
| **Безопасность** | Делегирует права пользователя |
| **Пользователь присутствует** | Да (в исходном контексте) |

---

## Как работает

1. Клиент (web/SPA/mobile) получает access token.
2. Отправляет его в Web API.
3. Web API использует этот токен для получения нового токена для downstream API.
4. Downstream API получает токен с делегированными правами пользователя.

---

## Ключевые особенности

- Используются **delegated permissions**.
- Права ограничены правами пользователя.
- Требуется confidential client на уровне middle-tier.
- Часто используется в микросервисной архитектуре.

---

## Важно для AZ-204

- OBO применяется, когда API вызывает другой API.
- Middle-tier не использует client credentials flow.
- Токен обменивается через специальный grant type.
- Conditional Access может потребовать обработки claims challenge.
- Пользователь физически не взаимодействует со вторым API, но его права делегируются.

> 🎯 Частый экзаменационный вопрос:  
Как Web API может вызвать Microsoft Graph от имени пользователя?  
Ответ — использовать On-Behalf-Of flow.

**Example**:

```csharp
// Middle-tier Web API
var userAssertion = new UserAssertion(accessToken);

var result = await app.AcquireTokenOnBehalfOf(scopes, userAssertion)
    .ExecuteAsync();
```

### 4. Device Code Flow

## 4️⃣ Device Code Flow

**Для устройств с ограниченным вводом данных** (input-constrained devices)

| Свойство | Значение |
|-----------|----------|
| **Flow** | Устройство отображает код → Пользователь вводит код на другом устройстве |
| **Используется в** | Smart TV, IoT-устройства, CLI-инструменты |
| **Безопасность** | Аутентификация происходит на отдельном устройстве |
| **Пользователь присутствует** | Да (на другом устройстве) |

---

## Как работает

1. Устройство запрашивает device code у identity platform.
2. Пользователю отображается код и URL для входа.
3. Пользователь открывает URL на телефоне или компьютере.
4. Вводит код и проходит аутентификацию.
5. Устройство получает access token после подтверждения.

---

## Когда использовать

- Устройства без браузера
- Консольные утилиты
- IoT-оборудование
- Smart TV

---

## Особенности

- Поддерживает delegated permissions.
- Пользователь аутентифицируется на другом устройстве.
- Подходит для публичных клиентов (public clients).
- Может использоваться вместе с MFA и Conditional Access.

---

## Важно для AZ-204

- Используется для устройств с ограниченным вводом.
- Пользователь присутствует, но на другом устройстве.
- Не требует client secret.
- Применяется в CLI и IoT-сценариях.

> 🎯 Частый экзаменационный вопрос:  
Как реализовать вход на устройстве без браузера?  
Ответ — использовать Device Code Flow.


**Example**:

```csharp
// CLI application
var result = await app.AcquireTokenWithDeviceCode(scopes, callback =>
{
    Console.WriteLine(callback.Message);
    return Task.CompletedTask;
}).ExecuteAsync();

// User sees: "Go to https://microsoft.com/devicelogin and enter code: ABCD1234"
```

### 5. Integrated Windows Authentication (IWA)

## 5️⃣ Integrated Windows Authentication (IWA)

**Для доменно-присоединённых машин** с поддержкой тихой (silent) аутентификации.

| Свойство | Значение |
|-----------|----------|
| **Flow** | Приложение использует учётные данные Windows без ввода пароля |
| **Используется в** | Domain-joined Windows устройствах |
| **Безопасность** | Основано на Kerberos |
| **Пользователь присутствует** | Да (вошедший в Windows пользователь) |

---

## Как работает

1. Пользователь уже вошёл в Windows под доменной учётной записью.
2. Приложение использует текущие Windows-учётные данные.
3. Аутентификация происходит автоматически без отображения UI.
4. Приложение получает токен без дополнительного ввода данных.

---

## Когда использовать

- Внутренние корпоративные приложения
- Desktop-приложения в доменной среде
- Интранет-сценарии

---

## Особенности

- Работает только в доменной инфраструктуре.
- Не подходит для публичных интернет-приложений.
- Основано на существующей сессии Windows.
- Часто используется в enterprise-среде.

---

## Важно для AZ-204

- Пользователь физически присутствует, но повторный вход не требуется.
- Используется для silent authentication.
- Поддерживается только в Windows-доменной среде.
- Не применяется для мобильных или cloud-only сценариев.

> 🎯 Частый экзаменационный вопрос:  
Как реализовать бесшовный вход в корпоративной сети Windows?  
Ответ — использовать Integrated Windows Authentication.


**Example**:

```csharp
// Domain-joined machine
var result = await app.AcquireTokenByIntegratedWindowsAuth(scopes)
    .ExecuteAsync();
```

### 6. Username/Password (ROPC) - **NOT RECOMMENDED**

## 6️⃣ Resource Owner Password Credentials (ROPC)

**Прямой ввод логина и пароля в приложении** — устаревший подход.

| Свойство | Значение |
|-----------|----------|
| **Flow** | Приложение получает имя пользователя и пароль → Отправляет их для аутентификации |
| **Используется в** | Только legacy-приложениях |
| **Безопасность** | ❌ Низкая — пароль передаётся приложению |
| **Пользователь присутствует** | Да |

---

## Как работает

1. Пользователь вводит логин и пароль прямо в приложении.
2. Приложение отправляет учётные данные в identity platform.
3. При успешной проверке возвращается access token.

---

## Почему считается небезопасным

- Приложение получает пароль пользователя.
- Повышенный риск утечки учётных данных.
- Не поддерживает современные механизмы безопасности:
    - MFA
    - Conditional Access
    - Passwordless authentication

---

## Когда может использоваться

- Старые системы, которые невозможно модернизировать.
- Ограниченные сценарии автоматизации (с осторожностью).

---

## Важно для AZ-204

- ROPC считается устаревшим и не рекомендуется.
- Не поддерживает современные политики безопасности.
- Не следует использовать в новых приложениях.
- Предпочтение отдаётся Authorization Code Flow.

> 🎯 Частый экзаменационный вопрос:  
Какой flow не рекомендуется для новых приложений из-за рисков безопасности?  
Ответ — Resource Owner Password Credentials (ROPC).


**Example (not recommended)**:

```csharp
// NOT RECOMMENDED - Use only for migration scenarios
var result = await app.AcquireTokenByUsernamePassword(
    scopes, 
    username, 
    securePassword
).ExecuteAsync();
```

## 7️⃣ Implicit Grant Flow — **DEPRECATED**

**Устаревший flow для SPA** — рекомендуется использовать Authorization Code Flow с PKCE.

| Свойство | Значение |
|-----------|----------|
| **Flow** | Токен возвращается напрямую в URL fragment |
| **Используется в** | Legacy SPA |
| **Безопасность** | ❌ Менее безопасен, чем authorization code |
| **Статус** | Устарел — использовать auth code + PKCE |

---

## Как работал

1. Пользователь проходил аутентификацию.
2. Access token возвращался напрямую в URL.
3. Приложение извлекало токен из URL fragment.

---

## Почему deprecated

- Токен передаётся через браузер.
- Повышенный риск утечки токена.
- Не соответствует современным требованиям безопасности.
- Не поддерживает современные сценарии (PKCE, усиленные политики).

---

## Современная альтернатива

**Authorization Code Flow с PKCE**:

- Более безопасен для public clients.
- Не возвращает токен напрямую в URL.
- Поддерживает современные механизмы защиты.

---

## Важно для AZ-204

- Implicit Flow не рекомендуется для новых приложений.
- Для SPA следует использовать Authorization Code + PKCE.
- PKCE обязателен для public clients.
- Может встречаться в legacy-сценариях.

> 🎯 Частый экзаменационный вопрос:  
Какой flow следует использовать для SPA вместо implicit?  
Ответ — Authorization Code Flow с PKCE.


# Public vs Confidential Client Applications

## Public Client Applications

**Не могут безопасно хранить секреты**

| Аспект | Описание |
|--------|----------|
| **Где выполняются** | На устройстве пользователя (desktop, mobile, браузер) |
| **Уровень доверия** | Нельзя доверять хранение секретов |
| **Секреты** | Не используют client secrets или сертификаты |
| **Исходный код** | Может быть просмотрен или декомпилирован |
| **Примеры** | Desktop-приложения, мобильные приложения, SPA |
| **Поддерживаемые flow** | Authorization Code (с PKCE), Device Code, IWA |

---

## Почему называются Public?

- Пользователь имеет доступ к исходному или скомпилированному коду.
- Приложение работает в неконтролируемой среде.
- Нет гарантированно защищённого хранилища для client secret.
- Пользователь может получить доступ к файловой системе или памяти приложения.

> 💡 Любой секрет, встроенный в public client, считается скомпрометированным.

---

## Особенности

- Используют PKCE для повышения безопасности.
- Не применяют client secret.
- Часто используют delegated permissions.
- Поддерживают интерактивные сценарии аутентификации.

---

## Важно для AZ-204

- SPA и мобильные приложения — это public clients.
- PKCE обязателен для Authorization Code Flow в public clients.
- Client secret нельзя использовать в public client.
- Confidential client применяется для серверных сценариев.

> 🎯 Частый экзаменационный вопрос:  
Почему SPA не может использовать client secret?  
Ответ — потому что это public client и секрет нельзя защитить.

**Example**:

```csharp
// Public client - no secret
IPublicClientApplication app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri("http://localhost")
    .Build();

// User signs in interactively
var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();
```

## Confidential Client Applications

**Могут безопасно хранить секреты**

| Аспект | Описание |
|--------|----------|
| **Где выполняются** | На сервере (web apps, web APIs, daemon-сервисы) |
| **Уровень доверия** | Можно доверять хранение секретов |
| **Секреты** | Используют client secret или сертификат |
| **Исходный код** | Недоступен конечным пользователям |
| **Примеры** | Web-приложения, Web API, фоновые сервисы |
| **Поддерживаемые flow** | Authorization Code, Client Credentials, OBO |

---

## Почему называются Confidential?

- Работают на сервере, а не на устройстве пользователя.
- Код и конфигурация недоступны пользователю.
- Возможность безопасного хранения:
    - Client secret
    - Сертификата
- Используют защищённый back-channel для обмена кодов на токены.

> 💡 Confidential client — это доверенная серверная среда.

---

## Особенности

- Обязателен для Client Credentials Flow.
- Используется в On-Behalf-Of Flow.
- Может безопасно использовать сертификаты вместо секретов.
- Часто применяется с application permissions.

---

## Важно для AZ-204

- Confidential client требуется для server-to-server сценариев.
- Использует client secret или сертификат.
- Не подходит для SPA и мобильных приложений.
- Может выполнять обмен authorization code на токен безопасно.

---

## Ключевое различие

- **Public client** — выполняется на устройстве пользователя, без секретов.
- **Confidential client** — выполняется на сервере, может хранить секреты.

> 🎯 Частый экзаменационный вопрос:  
Какой тип клиента требуется для Client Credentials Flow?  
Ответ — Confidential client.


**Example**:

```csharp
// Confidential client - with secret
IConfidentialClientApplication app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)  // Secret stored on server
    .WithAuthority(new Uri($"https://login.microsoftonline.com/{tenantId}"))
    .WithRedirectUri("https://myapp.azurewebsites.net/signin-oidc")
    .Build();

// App authenticates without user
var result = await app.AcquireTokenForClient(scopes)
    .ExecuteAsync();
```

## Сравнение Public и Confidential Client

| Характеристика | Public Client | Confidential Client |
|----------------|--------------|---------------------|
| **Местоположение** | Устройство пользователя | Сервер |
| **Client secret** | ❌ Нет | ✅ Да |
| **Сертификат** | ❌ Нет | ✅ Да |
| **Взаимодействие с пользователем** | Обычно требуется | Необязательно |
| **Получение токена** | От имени пользователя | От имени пользователя или приложения |
| **Примеры** | Desktop, mobile, SPA | Web app, API, daemon |
| **Builder (MSAL)** | `PublicClientApplicationBuilder` | `ConfidentialClientApplicationBuilder` |

---

## Ключевые различия

- **Public Client**
    - Работает в недоверенной среде.
    - Не может безопасно хранить секреты.
    - Использует PKCE вместо client secret.
    - Подходит для приложений с пользователем.

- **Confidential Client**
    - Работает на сервере.
    - Может безопасно хранить client secret или сертификат.
    - Используется для server-side сценариев.
    - Поддерживает application permissions.

---

## Важно для AZ-204

- Client Credentials Flow требует Confidential Client.
- SPA и мобильные приложения — это Public Client.
- On-Behalf-Of Flow выполняется Confidential Client.
- Builder зависит от типа приложения.

> 🎯 Запомнить просто:  
> Public = устройство пользователя, без секретов.  
> Confidential = сервер, с секретами.


## Token Caching and Refresh

### Automatic Token Caching

**MSAL handles caching automatically**:

```csharp
// First call - acquires token, stores in cache
var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();

// Later calls - tries cache first
try
{
    var accounts = await app.GetAccountsAsync();
    result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
        .ExecuteAsync();
    // Returns cached token if still valid
}
catch (MsalUiRequiredException)
{
    // Cache miss or token expired - user interaction required
    result = await app.AcquireTokenInteractive(scopes)
        .ExecuteAsync();
}
```

### Token Refresh

**MSAL refreshes tokens automatically**:

```
Access token expires in: 1 hour
Refresh token valid for: 90 days

MSAL behavior:
- Token valid → Return from cache
- Token expires soon → Refresh automatically
- Refresh token expired → Request user sign-in
```

**No manual handling needed**:

```csharp
// ✅ Good: Let MSAL handle refresh
var result = await app.AcquireTokenSilent(scopes, account)
    .ExecuteAsync();

// ❌ Bad: Manual token expiration checking
// Not needed - MSAL does this automatically
```

## Common Patterns

### Pattern 1: Interactive Desktop App

```csharp
var app = PublicClientApplicationBuilder
    .Create(clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, tenantId)
    .WithRedirectUri("http://localhost")
    .Build();

// First time - interactive sign-in
var result = await app.AcquireTokenInteractive(scopes)
    .ExecuteAsync();

// Subsequent calls - silent
var accounts = await app.GetAccountsAsync();
if (accounts.Any())
{
    try
    {
        result = await app.AcquireTokenSilent(scopes, accounts.FirstOrDefault())
            .ExecuteAsync();
    }
    catch (MsalUiRequiredException)
    {
        result = await app.AcquireTokenInteractive(scopes)
            .ExecuteAsync();
    }
}
```

### Pattern 2: Daemon Service

```csharp
var app = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)
    .WithAuthority(new Uri($"https://login.microsoftonline.com/{tenantId}"))
    .Build();

// No user interaction
var result = await app.AcquireTokenForClient(
    new[] { "https://graph.microsoft.com/.default" }
).ExecuteAsync();
```

### Pattern 3: Web API (On-Behalf-Of)

```csharp
[Authorize]
[HttpGet("data")]
public async Task<IActionResult> GetData()
{
    // Get user's access token from request
    var accessToken = Request.Headers["Authorization"].ToString().Replace("Bearer ", "");
    
    var userAssertion = new UserAssertion(accessToken);
    
    // Exchange for downstream API token
    var result = await _app.AcquireTokenOnBehalfOf(
        new[] { "https://graph.microsoft.com/User.Read" },
        userAssertion
    ).ExecuteAsync();
    
    // Call downstream API
    var data = await CallGraphApiAsync(result.AccessToken);
    
    return Ok(data);
}
```

## Best Practices

### 1. Use Correct Client Type

```csharp
// ✅ Good: Public client for desktop
IPublicClientApplication desktopApp = PublicClientApplicationBuilder.Create(clientId).Build();

// ✅ Good: Confidential client for server
IConfidentialClientApplication serverApp = ConfidentialClientApplicationBuilder
    .Create(clientId)
    .WithClientSecret(clientSecret)
    .Build();

// ❌ Bad: Confidential client for desktop (secret exposed)
```

### 2. Try Silent Acquisition First

```csharp
// ✅ Good: Try cache first
try
{
    result = await app.AcquireTokenSilent(scopes, account).ExecuteAsync();
}
catch (MsalUiRequiredException)
{
    result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();
}

// ❌ Bad: Always interactive
result = await app.AcquireTokenInteractive(scopes).ExecuteAsync();
```

### 3. Use Appropriate Flow

```csharp
// ✅ Good: Authorization code for interactive
await app.AcquireTokenInteractive(scopes).ExecuteAsync();

// ✅ Good: Client credentials for daemon
await app.AcquireTokenForClient(scopes).ExecuteAsync();

// ❌ Bad: Username/password (ROPC)
// Security risk, avoid unless absolutely necessary
```

### 4. Singleton Pattern for App Instance

```csharp
// ✅ Good: Single instance, reuse
private static IPublicClientApplication _app;

public static IPublicClientApplication GetApp()
{
    if (_app == null)
    {
        _app = PublicClientApplicationBuilder.Create(clientId).Build();
    }
    return _app;
}
```

# Critical Notes

- 💡 **MSAL** — библиотека Microsoft для получения токенов безопасности
- 🎯 **Кроссплатформенность** — поддержка .NET, JavaScript, Java, Python, Android, iOS
- ✅ **Автоматическое кеширование** — токены сохраняются и обновляются автоматически
- ⚠️ **Два типа клиентов** — Public (desktop, mobile, SPA) и Confidential (server)
- 🔄 **Автоматическое обновление токенов** — не требуется вручную обрабатывать expiration
- 📊 **Поддержка нескольких flow** — Authorization Code, Client Credentials, OBO, Device Code
- 💡 **Best practice** — выбирать правильный тип клиента и подходящий flow
- ✅ **Silent сначала** — сначала пытаться получить токен без UI
- ⚠️ **Избегать ROPC** — небезопасный и устаревший подход
- 🔒 **Singleton** — использовать один экземпляр MSAL-приложения

---

# Exam Tips (AZ-204)

## Основы MSAL

- MSAL используется для получения access и ID токенов.
- Упрощает работу с OAuth 2.0.
- Автоматически управляет кешем и обновлением токенов.
- Предоставляет единый API для разных платформ.

---

## Типы клиентов

### Public Client
- Desktop, mobile, SPA.
- Не может хранить client secret.
- Использует PKCE.
- Работает от имени пользователя.

### Confidential Client
- Web apps, Web APIs, daemon-сервисы.
- Может хранить client secret или сертификат.
- Используется для server-side сценариев.

---

## Основные Authentication Flows

- **Authorization Code Flow**  
  Самый распространённый, используется с пользователем.

- **Client Credentials Flow**  
  Для daemon-приложений, без пользователя.

- **On-Behalf-Of (OBO)**  
  Middle-tier API вызывает downstream API от имени пользователя.

- **Device Code Flow**  
  Для устройств с ограниченным вводом.

- **IWA**  
  Для доменно-присоединённых Windows машин.

- **ROPC**  
  Не рекомендуется.

- **Implicit Flow**  
  Устарел — использовать Authorization Code + PKCE.

---

## Работа с токенами

- MSAL автоматически:
    - Кеширует токены
    - Обновляет их
    - Обрабатывает истечение срока действия

- Сначала всегда пытаться получить токен без интерактивного входа.
- Если silent-получение не удалось — выполнить интерактивный вход.

---

## Важно помнить

- Client Credentials Flow требует Confidential Client.
- SPA и мобильные приложения — Public Client.
- OBO используется в микросервисной архитектуре.
- Singleton-паттерн повышает производительность.

---

## Часто проверяется

- Различие Public vs Confidential client.
- Какой flow использовать в конкретном сценарии.
- Почему не следует использовать ROPC.
- Почему Implicit Flow deprecated.
- Когда использовать OBO.
- Почему важно сначала пробовать silent acquisition.

---

> 🎯 Ключевая идея:  
> MSAL абстрагирует OAuth, управляет токенами автоматически и требует правильного выбора типа клиента и authentication flow.

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-authentication-by-using-microsoft-authentication-library/2-microsoft-authentication-library-overview)
