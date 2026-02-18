# Упражнение: Получение информации профиля пользователя через Microsoft Graph

## Обзор
Практическое упражнение: создать .NET console-приложение, которое получает данные профиля пользователя из Microsoft Graph, используя интерактивную аутентификацию.

## Предварительные требования
- ✅ Подписка Azure
- ✅ Visual Studio Code
- ✅ .NET 8.0 SDK или новее
- ✅ Расширение C# Dev Kit для VS Code
- ✅ Аккаунт Azure с правами на регистрацию приложений

## Длительность упражнения
⏱️ **Около 15 минут**

## Цели обучения
- Зарегистрировать приложение в Microsoft identity platform
- Настроить аутентификацию для public client
- Создать консольное приложение с Microsoft Graph SDK
- Выполнять интерактивную аутентификацию пользователя
- Получить профиль пользователя через Microsoft Graph API
- Обработать flow согласия (consent)

---

## Задача 1: Регистрация приложения в Azure Portal

### Шаг 1: Перейти в App Registrations

1. Войдите в [Azure Portal](https://portal.azure.com)
2. В поиске найдите **Microsoft Entra ID** (ранее Azure Active Directory)
3. В левом меню выберите **App registrations**
4. Нажмите **+ New registration**

### Шаг 2: Настроить регистрацию приложения

**Параметры приложения**:

| Поле | Значение |
|------|----------|
| Name | `myGraphApplication` |
| Supported account types | **Accounts in this organizational directory only (Single tenant)** |
| Redirect URI | Выбрать **Public client/native (mobile & desktop)** |
| Redirect URI value | `http://localhost` |

Нажмите **Register**

### Шаг 3: Сохранить идентификаторы приложения

На странице **Overview** скопируйте и сохраните:

- **Application (client) ID** — пример: `11111111-1111-1111-1111-111111111111`
- **Directory (tenant) ID** — пример: `22222222-2222-2222-2222-222222222222`

💡 Эти значения понадобятся для конфигурации приложения.

---

### Зачем такие настройки?

- **Single tenant**: приложение доступно только пользователям вашей организации
- **Public client**: не требуется client secret (интерактивная аутентификация)
- **http://localhost**: redirect для локальной разработки

---

### Дополнение от себя (полезно в реальной жизни и для AZ-204)

- Если в организации включены политики безопасности (Conditional Access), интерактивный вход может потребовать MFA — это нормально.
- Для console-app чаще всего хватает делегированных разрешений `User.Read` для чтения собственного профиля.
- Если позже понадобится работа без пользователя (daemon) — тогда уже будет другой flow (client credentials) и **Application permissions**.

## Task 2: Create Console Application

### Step 1: Create Project Directory

```bash
# Create project folder
mkdir graphapp
cd graphapp

# Create new console application
dotnet new console
```

### Step 2: Install Required NuGet Packages

```bash
# Azure Identity - For authentication
dotnet add package Azure.Identity

# Microsoft Graph SDK - For Graph API calls
dotnet add package Microsoft.Graph

# DotEnv - For environment variables (optional but recommended)
dotnet add package dotenv.net
```

**Package versions** (latest stable):
- `Azure.Identity` - 1.10.0 or later
- `Microsoft.Graph` - 5.0.0 or later
- `dotenv.net` - 3.1.2 or later

### Step 3: Create Environment Configuration

Create `.env` file in project root:

```bash
# .env
CLIENT_ID=11111111-1111-1111-1111-111111111111
TENANT_ID=22222222-2222-2222-2222-222222222222
```

⚠️ **Security**: Add `.env` to `.gitignore` to avoid committing secrets

```bash
# .gitignore
.env
bin/
obj/
```

## Task 3: Write Application Code

### Complete Program.cs

Replace contents of `Program.cs`:

```csharp
using Azure.Identity;
using Microsoft.Graph;
using Microsoft.Graph.Models;

// Load environment variables from .env file
DotEnv.Load();

// Get configuration from environment variables
var clientId = Environment.GetEnvironmentVariable("CLIENT_ID") 
    ?? throw new ArgumentNullException("CLIENT_ID is not set");
var tenantId = Environment.GetEnvironmentVariable("TENANT_ID") 
    ?? throw new ArgumentNullException("TENANT_ID is not set");

// Define required scopes
var scopes = new[] { "User.Read" };

// Configure interactive browser authentication
var options = new InteractiveBrowserCredentialOptions
{
    ClientId = clientId,
    TenantId = tenantId,
    AuthorityHost = AzureAuthorityHosts.AzurePublicCloud,
    RedirectUri = new Uri("http://localhost")
};

// Create credential
var credential = new InteractiveBrowserCredential(options);

// Create Microsoft Graph client
var graphClient = new GraphServiceClient(credential, scopes);

// Retrieve user profile
Console.WriteLine("Retrieving user profile...\n");

try
{
    // Call Microsoft Graph /me endpoint
    var user = await graphClient.Me.GetAsync();

    // Display user information
    Console.WriteLine("User Profile Information:");
    Console.WriteLine("=========================");
    Console.WriteLine($"Display Name: {user?.DisplayName}");
    Console.WriteLine($"User Principal Name: {user?.UserPrincipalName}");
    Console.WriteLine($"User ID: {user?.Id}");
    Console.WriteLine($"Job Title: {user?.JobTitle ?? "Not specified"}");
    Console.WriteLine($"Office Location: {user?.OfficeLocation ?? "Not specified"}");
    Console.WriteLine($"Mobile Phone: {user?.MobilePhone ?? "Not specified"}");
    Console.WriteLine($"Mail: {user?.Mail ?? "Not specified"}");
}
catch (Exception ex)
{
    Console.WriteLine($"Error: {ex.Message}");
}
```

### Code Breakdown

**1. Load environment variables**:
```csharp
DotEnv.Load();
var clientId = Environment.GetEnvironmentVariable("CLIENT_ID");
var tenantId = Environment.GetEnvironmentVariable("TENANT_ID");
```

**2. Define scopes**:
```csharp
var scopes = new[] { "User.Read" };  // Minimum permission for profile
```

**3. Configure authentication**:
```csharp
var options = new InteractiveBrowserCredentialOptions
{
    ClientId = clientId,          // Your app ID
    TenantId = tenantId,          // Your directory ID
    AuthorityHost = AzureAuthorityHosts.AzurePublicCloud,  // Azure cloud
    RedirectUri = new Uri("http://localhost")  // Must match registration
};
```

**4. Create credential**:
```csharp
var credential = new InteractiveBrowserCredential(options);
```

**5. Create Graph client**:
```csharp
var graphClient = new GraphServiceClient(credential, scopes);
```

**6. Call Graph API**:
```csharp
var user = await graphClient.Me.GetAsync();  // GET /me
```

## Task 4: Run the Application

### Step 1: Build and Run

```bash
dotnet run
```

### Step 2: Authentication Flow

**First run**:

1. **Console output**:
   ```
   Retrieving user profile...
   ```

2. **Browser opens automatically** with authentication prompt

3. **Sign in** with your Azure account

4. **Consent screen** appears:
   ```
   myGraphApplication wants to:
   - View your basic profile
   - Maintain access to data you have given it access to
   
   [Accept] [Cancel]
   ```

5. **Click Accept**

6. **Browser shows success**:
   ```
   Authentication complete. You can close this window.
   ```

7. **Console displays profile**:
   ```
   User Profile Information:
   =========================
   Display Name: John Doe
   User Principal Name: john.doe@contoso.com
   User ID: 87d349ed-44d7-43e1-9a83-5f2406dee5bd
   Job Title: Software Engineer
   Office Location: Building 4
   Mobile Phone: +1 555 0102
   Mail: john.doe@contoso.com
   ```

**Subsequent runs**:
- No browser prompt (uses cached token)
- Direct output of profile information
- Token cached in user profile directory

### Token Cache Location

```
Windows: C:\Users\{username}\.IdentityService\msal.cache
Linux: /home/{username}/.IdentityService/msal.cache
macOS: /Users/{username}/.IdentityService/msal.cache
```

## Task 5: Extend the Application

### Add More User Properties

```csharp
// Retrieve specific properties
var user = await graphClient.Me.GetAsync(config =>
{
    config.QueryParameters.Select = new[] 
    { 
        "id", 
        "displayName", 
        "mail", 
        "jobTitle",
        "officeLocation",
        "mobilePhone",
        "department",
        "companyName"
    };
});

Console.WriteLine($"Department: {user?.Department ?? "Not specified"}");
Console.WriteLine($"Company: {user?.CompanyName ?? "Not specified"}");
```

### Retrieve User's Manager

```csharp
// Get user with expanded manager
var user = await graphClient.Me.GetAsync(config =>
{
    config.QueryParameters.Expand = new[] { "manager" };
});

if (user?.Manager is Microsoft.Graph.Models.User manager)
{
    Console.WriteLine($"\nManager: {manager.DisplayName}");
    Console.WriteLine($"Manager Email: {manager.Mail}");
}
```

### Retrieve User's Photo

```csharp
try
{
    // Get user photo
    var photoStream = await graphClient.Me.Photo.Content.GetAsync();
    
    if (photoStream != null)
    {
        // Save to file
        using var fileStream = File.Create("profile-photo.jpg");
        await photoStream.CopyToAsync(fileStream);
        Console.WriteLine("Profile photo saved to profile-photo.jpg");
    }
}
catch (Exception ex)
{
    Console.WriteLine($"No profile photo available: {ex.Message}");
}
```

### Retrieve Recent Emails

**Update scopes** to include mail:

```csharp
var scopes = new[] { "User.Read", "Mail.Read" };
```

**Retrieve messages**:

```csharp
// Get last 10 messages
var messages = await graphClient.Me.Messages.GetAsync(config =>
{
    config.QueryParameters.Top = 10;
    config.QueryParameters.Select = new[] { "subject", "from", "receivedDateTime" };
    config.QueryParameters.Orderby = new[] { "receivedDateTime desc" };
});

Console.WriteLine("\nRecent Emails:");
Console.WriteLine("==============");

foreach (var message in messages?.Value ?? Enumerable.Empty<Message>())
{
    Console.WriteLine($"Subject: {message.Subject}");
    Console.WriteLine($"From: {message.From?.EmailAddress?.Address}");
    Console.WriteLine($"Received: {message.ReceivedDateTime}");
    Console.WriteLine();
}
```

## Task 6: Clean Up Resources

### Delete App Registration

1. Go to [Azure Portal](https://portal.azure.com)
2. Navigate to **Microsoft Entra ID** > **App registrations**
3. Find **myGraphApplication**
4. Click application name
5. Click **Delete** at top
6. Confirm deletion

### Delete Local Token Cache

```bash
# Windows
del %USERPROFILE%\.IdentityService\msal.cache

# Linux/macOS
rm ~/.IdentityService/msal.cache
```

### Delete Project Files

```bash
# From project parent directory
rm -rf graphapp
```

## Troubleshooting

### Error: "AADSTS700016: Application not found"

**Cause**: CLIENT_ID is incorrect

**Solution**: Verify Application (client) ID from Azure Portal

### Error: "AADSTS50011: Redirect URI mismatch"

**Cause**: Redirect URI doesn't match registration

**Solution**: Ensure `http://localhost` is configured in app registration

### Error: "Insufficient privileges to complete the operation"

**Cause**: User.Read permission not granted

**Solution**: 
1. Go to app registration > API permissions
2. Add **Microsoft Graph** > **Delegated permissions** > **User.Read**
3. Click **Grant admin consent** (if required)

### Browser doesn't open

**Cause**: No default browser or firewall blocking

**Solution**: 
1. Check firewall settings
2. Manually copy URL from console
3. Open in browser

### Token expired

**Cause**: Cached token is stale

**Solution**: Delete token cache file (see Clean Up section)

## Key Takeaways

### Authentication Flow

```mermaid
sequenceDiagram
    participant App
    participant Browser
    participant Azure AD
    participant Graph API
    
    App->>Browser: Open auth URL
    Browser->>Azure AD: User signs in
    Azure AD->>Browser: Authorization code
    Browser->>App: Return code
    App->>Azure AD: Exchange for token
    Azure AD->>App: Access token
    App->>Graph API: Request with token
    Graph API->>App: User profile
```

### Scopes vs Permissions (Scopes и Permissions)

| В регистрации приложения | В коде | Назначение |
|--------------------------|--------|-----------|
| API Permissions | массив Scopes | К чему приложение может получить доступ |
| User.Read | "User.Read" | Базовый профиль пользователя |
| Mail.Read | "Mail.Read" | Чтение почты |
| Calendars.Read | "Calendars.Read" | Чтение календаря |

> 💡 Практически: *permissions* настраиваются в Entra ID (App Registration), а *scopes* — это то, что приложение **запрашивает** при получении токена.

---

### Типы аутентификации (Authentication Types)

| Тип | Сценарий | Взаимодействие с пользователем |
|------|----------|-------------------------------|
| InteractiveBrowserCredential | Desktop-приложения | Да — через браузер |
| DeviceCodeCredential | Headless/CLI | Да — ввод кода |
| ClientSecretCredential | Daemon/Service | Нет |
| ManagedIdentityCredential | Azure ресурсы | Нет |

---

## Critical Notes (Ключевые моменты)

- 💡 **Single tenant** — приложение зарегистрировано для одной организации
- 🔒 **Public client** — интерактивная аутентификация, без secret’ов
- ✅ **User.Read** — минимальный scope для доступа к профилю
- 🎯 **InteractiveBrowserCredential** — автоматически открывает браузер для входа
- ⚠️ **Первый запуск** — требуется согласие пользователя (consent)
- 🔄 **Token caching** — последующие запуски используют кэш токена
- 📊 **Endpoint `/.me`** — возвращает текущего аутентифицированного пользователя
- 💡 **$select** — запрашивать только нужные свойства
- ✅ **$expand** — включить связанные сущности (например, manager)
- ⚠️ **Redirect URI** — должен *точно* совпадать с тем, что в регистрации приложения

---

## Exam Tips (Советы к AZ-204)

- При регистрации приложения сохранить:
   - Application (client) ID
   - Directory (tenant) ID
- Supported account types:
   - Single tenant (одна организация)
   - Multi-tenant (любая организация)
   - Personal accounts (личные Microsoft аккаунты)
- Redirect URI:
   - Public client — для desktop/mobile
   - Web — для веб-приложений
- Нужные пакеты:
   - Azure.Identity (аутентификация)
   - Microsoft.Graph (SDK)
- InteractiveBrowserCredential:
   - открывает браузер для аутентификации
- Параметры credential (часто встречаются в примерах):
   - ClientId, TenantId, AuthorityHost, RedirectUri
- Scopes:
   - массив строк вроде `["User.Read", "Mail.Read"]`
- GraphServiceClient:
   - `new GraphServiceClient(credential, scopes)`
- Получить текущего пользователя:
   - `await graphClient.Me.GetAsync()`
- Consent при первом запуске:
   - пользователь подтверждает запрашиваемые права в браузере
- Token caching:
   - при повторных запусках берётся токен из кэша (пример: `.IdentityService/msal.cache`)
- Query parameters:
   - Select (properties), Expand (related entities), Filter, Top, Orderby
- Error handling:
   - ловить исключения для auth/permissions проблем
- Cleanup:
   - удалить app registration, удалить token cache, удалить проект
- Troubleshooting:
   - проверить CLIENT_ID
   - проверить Redirect URI
   - убедиться, что API permissions выданы/подтверждены (consent)

---

### Дополнение от себя (частые ловушки)

- **Scopes в коде должны соответствовать разрешениям**, которые реально доступны (и выдано согласие). Если нет — получите 403/consent prompt.
- Для интерактивного сценария удобнее начинать с **Delegated permissions** (`User.Read`), а потом расширять.
- Если приложение запускается в Azure (Functions/App Service/VM) — часто лучший вариант **ManagedIdentityCredential** вместо secret’ов.

[Learn More](https://learn.microsoft.com/en-us/training/modules/microsoft-graph/5a-exercise-microsoft-graph-user-profile)
