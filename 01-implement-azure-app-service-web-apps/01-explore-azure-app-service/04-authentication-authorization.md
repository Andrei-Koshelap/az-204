# Аутентификация и авторизация в Azure App Service

## Ключевые концепции

- **Встроенная аутентификация** — не требует изменений в коде приложения
- **Middleware платформы** — выполняется до выполнения кода приложения
- **Федеративная идентификация** — аутентификацию выполняют внешние провайдеры
- **Token Store** — автоматическое управление токенами
- **Отсутствие зависимости от SDK** — работает с любым языком и фреймворком

💡 Важно: аутентификация происходит на уровне платформы, а не внутри приложения.

---

## Почему стоит использовать встроенную аутентификацию?

- ✅ Экономия времени разработки
- ✅ Поддержка нескольких identity-провайдеров
- ✅ Не требуется подключать SDK
- ✅ Автоматическое управление токенами
- ✅ Работает с любым языком

⚠️ На экзамене часто это называют **Easy Auth**.

---

## Провайдеры идентификации

| Провайдер | Endpoint входа | Примечание |
|------------|----------------|-------------|
| **Microsoft Entra ID** | `/.auth/login/aad` | Ранее Azure AD |
| **Facebook** | `/.auth/login/facebook` | - |
| **Google** | `/.auth/login/google` | - |
| **X (Twitter)** | `/.auth/login/x` | - |
| **GitHub** | `/.auth/login/github` | - |
| **Apple** | `/.auth/login/apple` | Preview |
| **OpenID Connect** | `/.auth/login/<providerName>` | Кастомный провайдер |

---

# Поток аутентификации (Authentication Flow)

## 1️⃣ Server-Directed Flow (без SDK провайдера)

**Лучше всего подходит для:** браузерных приложений.

| Шаг | Действие | Описание |
|------|----------|------------|
| 1 | Вход пользователя | Перенаправление на `/.auth/login/<provider>` |
| 2 | Callback | Провайдер возвращает на `/.auth/login/<provider>/callback` |
| 3 | Установка сессии | App Service добавляет cookie |
| 4 | Доступ к контенту | Браузер автоматически отправляет cookie |

---

## 2️⃣ Client-Directed Flow (через SDK провайдера)

**Лучше всего подходит для:** мобильных приложений, SPA, REST API.

| Шаг | Действие | Описание |
|------|----------|------------|
| 1 | Вход пользователя | Клиентский SDK выполняет вход |
| 2 | Передача токена | Клиент отправляет токен на `/.auth/login/<provider>` |
| 3 | Создание сессии | App Service возвращает auth token |
| 4 | Доступ | Клиент передаёт заголовок `X-ZUMO-AUTH` |

---

# Поведение авторизации

## Вариант 1: Разрешить анонимный доступ

Configuration: "Allow anonymous"
```

- ✅ Приложение само управляет авторизацией
- ✅ Гибкость
- ✅ Данные аутентификации передаются в HTTP-заголовках
- 📝 Подходит для публичных сайтов с опциональным входом

---

## Вариант 2: Требовать аутентификацию

```
Configuration: "Require authentication"
```

- ❌ Блокирует весь неаутентифицированный трафик
- 🔄 Перенаправляет браузер на `/.auth/login/<provider>`
- ⚠️ Для API возвращает `HTTP 401 Unauthorized`
- ⚠️ Может возвращать `HTTP 403 Forbidden`

📝 Подходит для внутренних приложений.

⚠️ Важно: если включить обязательную аутентификацию, будут заблокированы ВСЕ страницы, включая публичные (например, home page SPA).

---

# Как работает middleware

Auth middleware:

1. Аутентифицирует пользователя через провайдера
2. Валидирует и обновляет OAuth-токены
3. Управляет сессией
4. Передаёт информацию о пользователе в HTTP-заголовках

### Особенности

- Работает на той же VM (Windows)
- Работает в отдельном контейнере (Linux/containers)
- Не требует изменения кода
- Настраивается через Azure Portal или ARM
- Независим от языка программирования

---

# Token Store

## Что это такое

- Встроенное хранилище токенов пользователей
- Доступно для Web Apps, API, мобильных приложений
- Автоматически включается при использовании провайдера

---

## Access Tokens

Токены доступны через:

- Переменные окружения
- HTTP-заголовки
- Только при включённой встроенной аутентификации

💡 Это позволяет приложению обращаться к внешним API от имени пользователя.

---

# Logging и Tracing

Для диагностики аутентификации можно:

- Включить Application Insights
- Использовать Log Stream
- Проверять заголовки запроса
- Анализировать 401/403 ответы

---

# Экзаменационные ловушки AZ-204 (Authentication)

## 1️⃣ Easy Auth ≠ Managed Identity

- Easy Auth → для пользователей
- Managed Identity → для сервис-к-сервис

⚠️ Очень частая путаница на экзамене.

---

## 2️⃣ Require Authentication блокирует ВСЁ

Если в задаче:
- есть публичная страница
- но нужен login для части сайта

Правильный вариант → Allow anonymous + проверка в коде.

---

## 3️⃣ 401 vs 403

- 401 → неаутентифицирован
- 403 → аутентифицирован, но нет прав

---

## 4️⃣ Middleware работает до кода

Если включена обязательная аутентификация — запрос даже не попадёт в контроллер.

---

## 5️⃣ Token Store работает только с Built-in Auth

Если используется кастомная аутентификация в коде — Token Store недоступен.

---

# Итог

Azure App Service предоставляет:

- Встроенную аутентификацию (Easy Auth)
- Поддержку популярных identity-провайдеров
- Автоматическое управление токенами
- Отсутствие зависимости от SDK
- Возможность гибкой авторизации

Это упрощает безопасность приложения без усложнения кода.

## Token Store

### What It Is
- **Built-in repository** of tokens for authenticated users
- Available for web apps, APIs, mobile apps
- Automatically enabled with any provider

### Access Tokens
- Available via **environment variables**
- Available via **HTTP headers**
- Only with built-in authentication enabled

## Logging and Tracing

```bash
# Enable application logging
az webapp log config \
  --name <app-name> \
  --resource-group <rg-name> \
  --application-logging filesystem

# View logs
az webapp log tail \
  --name <app-name> \
  --resource-group <rg-name>
```

- ✅ Auth traces collected in app logs
- ✅ Find auth errors in existing logs
- ✅ Convenient debugging

## Configuration Examples

### Configure Microsoft Entra ID

```bash
# Enable auth with Azure AD
az webapp auth update \
  --name <app-name> \
  --resource-group <rg-name> \
  --enabled true \
  --action LoginWithAzureActiveDirectory

# Configure Azure AD
az webapp auth microsoft update \
  --name <app-name> \
  --resource-group <rg-name> \
  --client-id <app-id> \
  --tenant-id <tenant-id>
```

### Access User Claims in Code

```csharp
// C# - Access claims
var identity = (ClaimsIdentity)User.Identity;
var claims = identity.Claims;

// Get user info from headers
var userId = Request.Headers["X-MS-CLIENT-PRINCIPAL-ID"];
var userName = Request.Headers["X-MS-CLIENT-PRINCIPAL-NAME"];
```

```javascript
// Node.js - Access user info
app.get('/api/user', (req, res) => {
  const userId = req.headers['x-ms-client-principal-id'];
  const userName = req.headers['x-ms-client-principal-name'];
  res.json({ userId, userName });
});
```

## Быстрая справка (Quick Reference)

| Функция | Детали |
|----------|---------|
| **Endpoints** | `/.auth/login/<provider>` |
| **Проверка токена** | Выполняется автоматически |
| **Управление сессией** | Встроено в платформу |
| **Данные пользователя в заголовках** | `X-MS-CLIENT-PRINCIPAL-*` |
| **Заголовок токена** | `X-ZUMO-AUTH` (для client flow) |
| **Настройка** | Через портал, ARM или конфигурационный файл |

---

## Критические замечания

- 💡 **Не требует изменений кода** — только конфигурация.
- 🎯 Поддерживается **несколько провайдеров одновременно**.
- ⚠️ В Linux аутентификация работает в отдельном контейнере (не in-process).
- 🔐 Token Store включается автоматически и управляет обновлением токенов.
- 📝 Auth middleware выполняется **до кода приложения**.
- 🚀 Работает с любым языком — SDK не требуется.
- ⚠️ Включение "Require authentication" блокирует весь трафик, включая домашнюю страницу.

---

## Экзаменационные советы

- Знайте два типа потоков аутентификации:
    - server-directed
    - client-directed
- Понимайте, что middleware выполняется до выполнения кода.
- Помните шаблон endpoint:

```bash
  /.auth/login/<provider>
```
- Token Store активируется автоматически при включении встроенной аутентификации.
- Изменения кода не требуются — всё настраивается через конфигурацию.
- Данные пользователя доступны в HTTP-заголовках:

```bash
  X-MS-CLIENT-PRINCIPAL-*
```
[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-app-service/5-authentication-authorization-app-service)
