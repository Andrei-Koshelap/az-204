# Упражнение: Реализовать интерактивную аутентификацию с MSAL.NET

## Обзор

Создайте .NET консольное приложение, которое использует **MSAL.NET** для интерактивной аутентификации через Microsoft Entra ID и получения **access token** для **Microsoft Graph**.

---

## Цели обучения

- Зарегистрировать приложение в Microsoft identity platform
- Использовать `PublicClientApplicationBuilder` для настройки аутентификации
- Получать токены интерактивно с разрешениями Microsoft Graph
- Понять кеширование токенов и silent-аутентификацию

## Prerequisites
- Azure subscription ([sign up for free](https://azure.microsoft.com/))
- [Visual Studio Code](https://code.visualstudio.com/)
- [.NET 8](https://dotnet.microsoft.com/en-us/download/dotnet/8.0) or greater
- [C# Dev Kit](https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csdevkit) for VS Code

## Exercise Duration
⏱️ Approximately 15 minutes

## Part 1: Register Application in Azure Portal

### Step 1: Navigate to App Registrations

```
Azure Portal → Search "App registrations" → + New registration
```

### Шаг 2: Настройка регистрации (Configure Registration)

| Параметр | Значение |
|----------|----------|
| **Name** | `myMsalApplication` |
| **Supported account types** | Только аккаунты в этом организационном каталоге (Single tenant) |
| **Redirect URI** | Платформа: `Public client/native (mobile & desktop)`<br>URI: `http://localhost` |

---

## Примечания

- Выбор **Single tenant** означает, что вход смогут выполнять только пользователи вашего tenant.
- Тип redirect URI **Public client/native** соответствует сценарию консольного/desktop приложения.
- `http://localhost` используется как стандартный redirect для public client в desktop/console сценариях.

---

### Шаг 3: Запишите важные значения (Record Important Values)

После регистрации приложения на странице **Overview** сохраните следующие параметры:

| Параметр | Где найти |
|----------|-----------|
| **Application (client) ID** | Overview → раздел Essentials |
| **Directory (tenant) ID** | Overview → раздел Essentials |

💡 **Совет**: скопируйте эти ID в отдельный файл — они понадобятся на следующих шагах.

**Где искать на странице**: в блоке **Essentials** найдите строки
- “Application (client) ID”
- “Directory (tenant) ID”


## Part 2: Create .NET Console Application

### Step 1: Create Project Folder

```bash
# Create project folder
mkdir authapp
cd authapp

# Open in VS Code
code .
```

### Step 2: Create Console Application

Open terminal in VS Code (`View → Terminal`) and run:

```bash
# Create new console app
dotnet new console
```

### Step 3: Add Required Packages

```bash
# Add Microsoft Identity Client library
dotnet add package Microsoft.Identity.Client

# Add DotEnv for environment variables
dotnet add package dotenv.net
```

**Packages installed**:
- `Microsoft.Identity.Client` - MSAL.NET library for authentication
- `dotenv.net` - Load configuration from .env file

### Step 4: Create Environment Configuration

Create a file named `.env` in the project root:

**File**: `.env`

```env
CLIENT_ID="YOUR_CLIENT_ID"
TENANT_ID="YOUR_TENANT_ID"
```

⚠️ **Replace**:
- `YOUR_CLIENT_ID` - Application (client) ID from Azure Portal
- `YOUR_TENANT_ID` - Directory (tenant) ID from Azure Portal

💡 **Security tip**: Add `.env` to `.gitignore` to avoid committing secrets:

```bash
echo ".env" >> .gitignore
```

## Part 3: Implement Authentication Code

### Step 1: Add Starter Code

Open `Program.cs` and replace content with:

```csharp
using Microsoft.Identity.Client;
using dotenv.net;

// Load environment variables from .env file
DotEnv.Load();
var envVars = DotEnv.Read();

// Retrieve Azure AD Application ID and tenant ID from environment variables
string _clientId = envVars["CLIENT_ID"];
string _tenantId = envVars["TENANT_ID"];

// ADD CODE TO DEFINE SCOPES AND CREATE CLIENT



// ADD CODE TO ACQUIRE AN ACCESS TOKEN


```

### Step 2: Define Scopes and Create MSAL Client

Replace `// ADD CODE TO DEFINE SCOPES AND CREATE CLIENT` with:

```csharp
// Define the scopes required for authentication
string[] _scopes = { "User.Read" };

// Build the MSAL public client application with authority and redirect URI
var app = PublicClientApplicationBuilder.Create(_clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, _tenantId)
    .WithDefaultRedirectUri()
    .Build();
```

**Code explanation**:
- `User.Read` - Microsoft Graph permission to read user profile
- `PublicClientApplicationBuilder.Create()` - Initialize public client for desktop app
- `.WithAuthority()` - Specify Azure AD authority with tenant
- `.WithDefaultRedirectUri()` - Use platform default redirect (http://localhost)

### Step 3: Implement Token Acquisition

Replace `// ADD CODE TO ACQUIRE AN ACCESS TOKEN` with:

```csharp
// Attempt to acquire an access token silently or interactively
AuthenticationResult result;
try
{
    // Try to acquire token silently from cache for the first available account
    var accounts = await app.GetAccountsAsync();
    result = await app.AcquireTokenSilent(_scopes, accounts.FirstOrDefault())
                .ExecuteAsync();
}
catch (MsalUiRequiredException)
{
    // If silent token acquisition fails, prompt the user interactively
    result = await app.AcquireTokenInteractive(_scopes)
                .ExecuteAsync();
}

// Output the acquired access token to the console
Console.WriteLine($"Access Token:\n{result.AccessToken}");
```

**Code explanation**:
- **Try block**: Attempt silent authentication (from cache)
  - `GetAccountsAsync()` - Get cached user accounts
  - `AcquireTokenSilent()` - Try to get token without UI
- **Catch block**: If silent fails, show interactive login
  - `MsalUiRequiredException` - No cached token available
  - `AcquireTokenInteractive()` - Show browser for user login
- **Result**: Access token printed to console

### Complete Program.cs

**Final code**:

```csharp
using Microsoft.Identity.Client;
using dotenv.net;

// Load environment variables from .env file
DotEnv.Load();
var envVars = DotEnv.Read();

// Retrieve Azure AD Application ID and tenant ID from environment variables
string _clientId = envVars["CLIENT_ID"];
string _tenantId = envVars["TENANT_ID"];

// Define the scopes required for authentication
string[] _scopes = { "User.Read" };

// Build the MSAL public client application with authority and redirect URI
var app = PublicClientApplicationBuilder.Create(_clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, _tenantId)
    .WithDefaultRedirectUri()
    .Build();

// Attempt to acquire an access token silently or interactively
AuthenticationResult result;
try
{
    // Try to acquire token silently from cache for the first available account
    var accounts = await app.GetAccountsAsync();
    result = await app.AcquireTokenSilent(_scopes, accounts.FirstOrDefault())
                .ExecuteAsync();
}
catch (MsalUiRequiredException)
{
    // If silent token acquisition fails, prompt the user interactively
    result = await app.AcquireTokenInteractive(_scopes)
                .ExecuteAsync();
}

// Output the acquired access token to the console
Console.WriteLine($"Access Token:\n{result.AccessToken}");
```

## Part 4: Run and Test the Application

### First Run: Interactive Login

```bash
dotnet run
```

**Expected behavior**:
1. **Browser opens automatically** - Default browser launches
2. **Account selection** - Choose the account associated with your tenant
3. **Permissions request** - First-time consent prompt:
   ```
   Permissions requested
   
   myMsalApplication needs permission to:
   - Sign you in and read your profile
   - Maintain access to data you have given it access to
   
   [Accept] [Cancel]
   ```
4. **Click Accept** - Grant permissions
5. **Browser redirects** - Returns to application
6. **Console output** - Access token displayed:
   ```
   Access Token:
   eyJ0eXAiOiJKV1QiLCJub25jZSI6IlZF...
   ```

### Second Run: Silent Authentication

```bash
dotnet run
```

**Expected behavior**:
1. **No browser window** - Token retrieved from cache
2. **No consent prompt** - Previously granted permissions cached
3. **Immediate output** - Access token displayed instantly

**Why no prompt?**
- Permissions cached after first acceptance
- Account information stored in token cache
- Silent acquisition successful

💡 **Note**: With some account configurations, you might see the consent prompt again even on subsequent runs.

## Part 5: Verify Token Content

### Decode Access Token

Use [jwt.ms](https://jwt.ms) to decode the token:

1. Copy access token from console
2. Navigate to https://jwt.ms
3. Paste token
4. View decoded claims

**Expected claims**:

```json
{
  "aud": "00000003-0000-0000-c000-000000000000",  // Microsoft Graph
  "iss": "https://sts.windows.net/{tenant-id}/",
  "name": "Your Name",
  "scp": "User.Read",  // Granted scopes
  "tid": "{tenant-id}",
  "upn": "your.email@domain.com"
}
```

## Part 6: Clean Up Resources

### Delete App Registration

```
Azure Portal → App registrations → myMsalApplication → Delete
```

### Remove Local Project

```bash
cd ..
rm -rf authapp
```

## Project Structure

```
authapp/
├── .env                    # Environment variables (not in git)
├── .gitignore             # Ignore .env file
├── Program.cs             # Main application code
├── authapp.csproj         # Project file
├── obj/                   # Build artifacts
└── bin/                   # Compiled binaries
```

## Code Flow Diagram

```
┌─────────────────────────────────────────────────────┐
│ 1. Load environment variables (.env)               │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│ 2. Build PublicClientApplication                   │
│    - clientId, tenantId, authority                 │
└────────────────┬────────────────────────────────────┘
                 │
┌────────────────▼────────────────────────────────────┐
│ 3. Try AcquireTokenSilent                          │
│    (Check cache for existing token)                │
└────────────────┬────────────────────────────────────┘
                 │
         ┌───────┴───────┐
         │               │
    ✅ Success      ❌ MsalUiRequiredException
         │               │
         │       ┌───────▼────────┐
         │       │ AcquireToken   │
         │       │ Interactive    │
         │       │ (Browser login)│
         │       └───────┬────────┘
         │               │
         └───────┬───────┘
                 │
┌────────────────▼────────────────────────────────────┐
│ 4. Output access token                             │
└─────────────────────────────────────────────────────┘
```

## Authentication Flow Details

### First Run Flow

```
Application → MSAL.NET → Browser Opens
                        ↓
                User selects account
                        ↓
                User consents to permissions
                        ↓
            Microsoft Entra ID validates
                        ↓
        Returns authorization code to app
                        ↓
    MSAL.NET exchanges code for tokens
                        ↓
        Tokens cached for future use
                        ↓
            Access token returned
```

### Subsequent Run Flow

```
Application → MSAL.NET → Check cache
                        ↓
            Token found and valid?
                        ↓
                    Yes → Return cached token
                        ↓
                    No → Interactive flow
```

## Token Caching

### Cache Location

| Platform | Cache Location |
|----------|----------------|
| **Windows** | `%LOCALAPPDATA%\.IdentityService\msal.cache` |
| **macOS** | Keychain Access |
| **Linux** | `~/.local/share/.IdentityService/msal.cache` |

### Cache Contents

```
- Access tokens
- Refresh tokens
- ID tokens
- Account information
```

### Cache Encryption

- **Windows**: DPAPI (Data Protection API)
- **macOS**: Keychain
- **Linux**: libsecret

## Common Issues and Solutions

### Issue 1: "Redirect URI mismatch"

**Error**:
```
AADSTS50011: The redirect URI 'http://localhost' specified in the request 
does not match the redirect URIs configured for the application
```

**Solution**:
```
Azure Portal → App registrations → myMsalApplication → Authentication
→ Add platform → Mobile and desktop applications
→ Add http://localhost
```

### Issue 2: "Invalid client"

**Error**:
```
AADSTS700016: Application with identifier '...' was not found
```

**Solution**:
- Verify `CLIENT_ID` in `.env` matches Azure Portal
- Ensure no extra spaces or quotes in `.env`

### Issue 3: "Insufficient privileges"

**Error**:
```
Insufficient privileges to complete the operation
```

**Solution**:
- Verify `User.Read` permission added in Azure Portal
- Grant admin consent if required:
  ```
  Azure Portal → App registrations → API permissions
  → Grant admin consent for {tenant}
  ```

### Issue 4: Browser doesn't open

**Solution**:
```csharp
// Add system browser launcher
result = await app.AcquireTokenInteractive(_scopes)
    .WithUseEmbeddedWebView(false)  // Use system browser
    .ExecuteAsync();
```

## Key Concepts Reinforced

### 1. Public Client Application
✅ Used for applications that **cannot keep secrets secure**
- Desktop applications
- Mobile applications
- Browser-based applications

### 2. Interactive Authentication
✅ User **must be present** to sign in
- Browser-based login flow
- User consent required on first run
- Tokens cached for subsequent runs

### 3. Silent Authentication
✅ Tokens retrieved from cache **without user interaction**
- Faster authentication
- Better user experience
- Falls back to interactive if cache miss

### 4. Microsoft Graph Scopes
✅ Permissions that define what the app can access
- `User.Read` - Read signed-in user's profile
- Format: `{resource}/{permission}`
- Can request multiple scopes

### 5. Token Types
- **Access Token** - Used to call Microsoft Graph API
- **ID Token** - Contains user identity information
- **Refresh Token** - Used to obtain new access tokens

## Best Practices Demonstrated

### ✅ Environment Variables
- Secrets stored in `.env` file
- Not hardcoded in source code
- Excluded from version control

### ✅ Error Handling
- Try-catch for `MsalUiRequiredException`
- Graceful fallback to interactive auth
- Clear error messages

### ✅ Silent-First Pattern
- Always try silent authentication first
- Fall back to interactive only when needed
- Improves user experience

### ✅ Default Redirect URI
- Use `.WithDefaultRedirectUri()` for simplicity
- Platform-appropriate defaults
- No manual configuration needed

## Extension Exercises

### 1. Display User Information

Add after token acquisition:

```csharp
using System.Net.Http;
using System.Net.Http.Headers;

// Call Microsoft Graph to get user info
var httpClient = new HttpClient();
httpClient.DefaultRequestHeaders.Authorization = 
    new AuthenticationHeaderValue("Bearer", result.AccessToken);

var response = await httpClient.GetStringAsync("https://graph.microsoft.com/v1.0/me");
Console.WriteLine($"User Info:\n{response}");
```

### 2. Request Multiple Scopes

Change scopes array:

```csharp
string[] _scopes = { 
    "User.Read", 
    "Mail.Read",
    "Calendars.Read" 
};
```

### 3. Implement Sign Out

Add sign-out functionality:

```csharp
// Get all accounts
var accounts = await app.GetAccountsAsync();

// Remove accounts (sign out)
foreach (var account in accounts)
{
    await app.RemoveAsync(account);
}

Console.WriteLine("Signed out successfully");
```

### 4. Add Logging

Enable MSAL logging:

```csharp
var app = PublicClientApplicationBuilder.Create(_clientId)
    .WithAuthority(AzureCloudInstance.AzurePublic, _tenantId)
    .WithDefaultRedirectUri()
    .WithLogging((level, message, containsPii) =>
    {
        Console.WriteLine($"[{level}] {message}");
    }, LogLevel.Verbose, enablePiiLogging: false)
    .Build();
```
# Critical Notes

- 🎯 **Public client** — используется для desktop/mobile приложений, не хранит секреты
- 💡 **Интерактивная аутентификация** — пользователь входит через браузер
- ✅ **Silent-first подход** — сначала попытка получить токен из кеша
- ⚠️ **Scopes** — `User.Read` для базового доступа к профилю
- 🔄 **Кеширование токенов** — автоматически управляется MSAL.NET
- 📊 **Consent** — при первом запуске требуется подтверждение разрешений
- 💡 **MsalUiRequiredException** — сигнал, что требуется интерактивная аутентификация
- ✅ **Переменные окружения** — безопасный способ хранения Client ID и Tenant ID
- ⚠️ **Redirect URI** — должен совпадать с настройкой в Azure Portal
- 🔒 **Очистка** — после упражнения удалить App Registration

---

# Exam Tips (AZ-204)

## Основы

- `PublicClientApplicationBuilder` используется для public client.
- `AcquireTokenInteractive` инициирует вход через браузер.
- `AcquireTokenSilent` пытается получить токен из кеша.
- Если silent-аутентификация не удалась — возникает `MsalUiRequiredException`.

---

## Конфигурация

- `WithAuthority` — указывает cloud instance и tenant.
- `WithDefaultRedirectUri` — автоматически устанавливает подходящий redirect.
- `User.Read` — базовое разрешение Microsoft Graph.
- Redirect URI должен быть зарегистрирован в настройках приложения.

---

## Поведение приложения

- **Первый запуск** — пользователь видит окно consent.
- **Повторные запуски** — токен берётся из кеша без взаимодействия.
- Кеш токенов автоматически шифруется (зависит от платформы).

---

## Работа с кешем

- `GetAccountsAsync` — получение учётных записей из кеша.
- Рекомендуемый паттерн:
    1. Попытка silent получения токена
    2. При ошибке — интерактивная аутентификация

---

## Безопасность

- Никогда не хардкодить секреты.
- Использовать переменные окружения или Azure Key Vault.
- Загружать конфигурацию из `.env` или конфигурационных файлов.
- Токен — это JWT, содержит claims (aud, iss, scp, tid, upn).

---

## Что часто проверяется

- Разница между silent и interactive.
- Когда возникает `MsalUiRequiredException`.
- Почему redirect URI должен совпадать.
- Какой builder использовать для desktop-приложения.
- Почему public client не использует secret.

---

> 🎯 Ключевая идея:  
> Попробовать silent → при необходимости перейти к interactive → использовать кеш токенов автоматически.


[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-authentication-by-using-microsoft-authentication-library/4-interactive-authentication-msal)
