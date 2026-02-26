# API Management Policies

## Что такое Policies?

**Policies** — это набор правил (инструкций), которые последовательно выполняются при обработке запроса или ответа API.

Они позволяют изменять поведение API без изменения кода backend-сервисов.

---

### Формат

XML-конфигурация.

Политики описываются в виде XML-правил и применяются внутри Azure API Management.

---

### Где выполняются

В рамках request/response pipeline:

- Входящий запрос проходит через набор политик
- Затем отправляется в backend
- Ответ backend также может быть обработан политиками перед возвратом клиенту

---

### Назначение

Политики используются для:

- Трансформации запросов и ответов
- Обеспечения безопасности
- Ограничения скорости (throttling / rate limiting)
- Контроля доступа
- Валидации данных
- Маршрутизации запросов
- Кэширования

---

### Архитектурный смысл

Policies позволяют реализовать кросс-сервисную логику на уровне gateway:

- не нужно менять backend-код;
- можно централизованно управлять поведением API;
- можно быстро внедрять новые правила безопасности;
- легко адаптировать API под разные клиентские сценарии.

Это особенно важно в микросервисной архитектуре, где backend-сервисы должны оставаться простыми и сфокусированными на бизнес-логике.

---

### Важно для AZ-204

Если в вопросе говорится о:
- изменении поведения API без изменения backend;
- трансформации заголовков или тела запроса;
- ограничении количества вызовов;
- проверке токенов или claims;

— решение, скорее всего, связано с использованием API Management Policies.
---

## Policy Structure

Policies are organized into four sections that execute at different stages:

```xml
<policies>
  <inbound>
    <!-- Applied on the request before it's forwarded to backend -->
  </inbound>
  <backend>
    <!-- Applied before/after calling backend service -->
  </backend>
  <outbound>
    <!-- Applied on the response before sending to client -->
  </outbound>
  <on-error>
    <!-- Applied if an error occurs at any stage -->
  </on-error>
</policies>
```

### Execution Flow

```
┌──────────────┐
│  Client      │
│  Request     │
└──────┬───────┘
       │
       ▼
┌─────────────────────────────┐
│  inbound section            │
│  • Authentication           │
│  • Rate limiting            │
│  • Transform request        │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  backend section            │
│  • Forward to backend       │
│  • Cache lookup             │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  Backend Service            │
│  (Your API)                 │
└──────────┬──────────────────┘
           │
           ▼
┌─────────────────────────────┐
│  outbound section           │
│  • Transform response       │
│  • Add headers              │
│  • Cache response           │
└──────────┬──────────────────┘
           │
           ▼
┌──────────────┐
│  Client      │
│  Response    │
└──────────────┘

           │ (if error occurs)
           ▼
┌─────────────────────────────┐
│  on-error section           │
│  • Log error                │
│  • Return custom response   │
└─────────────────────────────┘
```

---

## Разделы Policies

### 1. **inbound Section**

Выполняется **до** передачи запроса в backend.

Этот раздел используется для обработки входящего запроса ещё на уровне gateway.

---

### Типовые сценарии использования

- ✅ Аутентификация и авторизация
- ✅ Ограничение скорости (rate limiting) и квоты
- ✅ Трансформация запроса
- ✅ Манипуляция HTTP-заголовками
- ✅ Валидация query-параметров
- ✅ Фильтрация по IP-адресам

---

### Что важно понимать

Все проверки и модификации выполняются **до** обращения к backend-сервису.

Это означает, что:

- Невалидные или неавторизованные запросы могут быть отклонены сразу
- Backend защищён от лишней нагрузки
- Можно изменить структуру запроса без изменения кода сервиса

---

### Архитектурное значение

Раздел `inbound` отвечает за безопасность и контроль входящего трафика.

Чаще всего в экзаменационных вопросах операции, связанные с:
- проверкой токенов,
- ограничением количества вызовов,
- изменением заголовков запроса,

— реализуются именно в `inbound` section.

---

### Важно для AZ-204

Если требуется:
- проверить JWT до вызова backend,
- заблокировать IP,
- ограничить количество запросов,
- изменить заголовок перед отправкой в сервис,

— правильный ответ: использовать `inbound` policy.

**Example**:
```xml
<inbound>
  <!-- Validate subscription key -->
  <check-header name="Ocp-Apim-Subscription-Key" failed-check-httpcode="401" />
  
  <!-- Rate limit: 100 calls per minute -->
  <rate-limit calls="100" renewal-period="60" />
  
  <!-- Add custom header -->
  <set-header name="X-API-Version" exists-action="override">
    <value>1.0</value>
  </set-header>
  
  <!-- Transform request body -->
  <set-body>@{
    var body = context.Request.Body.As<JObject>();
    body["timestamp"] = DateTime.UtcNow.ToString();
    return body.ToString();
  }</set-body>
  
  <!-- Base policy (inherit parent policies) -->
  <base />
</inbound>
```

### 2. **backend Section**

Выполняется **вокруг вызова backend-сервиса**.

Этот раздел управляет тем, как gateway взаимодействует с backend.

---

### Типовые сценарии использования

- ✅ Перенаправление запроса в backend
- ✅ Динамическое изменение URL backend
- ✅ Настройка таймаута
- ✅ Повторные попытки (retry)
- ✅ Реализация circuit breaker
- ✅ Возврат mock-ответов

---

### Что важно понимать

Раздел `backend` определяет поведение шлюза во время обращения к сервису:

- можно изменить адрес backend в зависимости от условий;
- можно управлять устойчивостью системы (retry / circuit breaker);
- можно имитировать ответы без реального вызова сервиса.

Это особенно полезно при:

- тестировании,
- миграции сервисов,
- постепенном переключении трафика,
- реализации отказоустойчивости.

---

### Архитектурное значение

`backend` section отвечает за устойчивость и маршрутизацию.

Он позволяет:

- реализовать failover;
- ограничить время ожидания ответа;
- управлять логикой обращения к нескольким backend-ресурсам.

---

### Важно для AZ-204

Если в задаче говорится о:
- динамическом выборе backend,
- настройке таймаута,
- повторных попытках при ошибке,
- имитации ответа без вызова сервиса,

— решение связано с использованием `backend` policy.

**Example**:
```xml
<backend>
  <!-- Set backend URL dynamically -->
  <set-backend-service base-url="@{
    return context.Request.Headers.GetValueOrDefault("X-Environment") == "production" 
      ? "https://prod.backend.com"
      : "https://test.backend.com";
  }" />
  
  <!-- Forward with timeout -->
  <forward-request timeout="30" />
  
  <!-- Or implement retry logic -->
  <retry condition="@(context.Response.StatusCode >= 500)" count="3" interval="5">
    <forward-request timeout="10" />
  </retry>
</backend>
```

### 3. **outbound Section**

Выполняется **после получения ответа от backend-сервиса**.

Этот раздел позволяет изменить или обработать ответ перед отправкой клиенту.

---

### Типовые сценарии использования

- ✅ Трансформация ответа
- ✅ Добавление или удаление HTTP-заголовков ответа
- ✅ Кэширование ответа
- ✅ Фильтрация содержимого ответа
- ✅ Установка HTTP-статуса ответа
- ✅ Преобразование формата (например, XML → JSON)

---

### Что важно понимать

`outbound` section работает с уже полученным ответом backend.

Это позволяет:

- адаптировать формат данных под требования клиента;
- скрывать внутренние детали реализации;
- удалять чувствительные поля;
- стандартизировать структуру ответа;
- управлять кэшированием на уровне gateway.

Backend при этом остаётся неизменным.

---

### Архитектурное значение

Раздел `outbound` отвечает за:

- совместимость API;
- контроль возвращаемых данных;
- унификацию форматов;
- реализацию BFF-подхода.

Он особенно полезен при:

- миграции API;
- поддержке нескольких версий;
- работе с устаревшими backend-сервисами.

---

### Важно для AZ-204

Если в задаче требуется:
- изменить тело ответа,
- преобразовать формат данных,
- удалить или добавить заголовок ответа,
- реализовать кэширование,

— решение связано с использованием `outbound` policy.

**Example**:
```xml
<outbound>
  <!-- Remove server header for security -->
  <set-header name="Server" exists-action="delete" />
  <set-header name="X-Powered-By" exists-action="delete" />
  
  <!-- Add CORS headers -->
  <cors>
    <allowed-origins>
      <origin>https://www.contoso.com</origin>
    </allowed-origins>
    <allowed-methods>
      <method>GET</method>
      <method>POST</method>
    </allowed-methods>
  </cors>
  
  <!-- Cache response for 1 hour -->
  <cache-store duration="3600" />
  
  <!-- Transform response -->
  <set-body>@{
    var response = context.Response.Body.As<JObject>();
    response["generatedAt"] = DateTime.UtcNow.ToString();
    return response.ToString();
  }</set-body>
  
  <base />
</outbound>
```

### 4. **on-error Section**

Выполняется **только при возникновении ошибки** в любом из разделов (inbound, backend, outbound).

Этот раздел позволяет централизованно обрабатывать исключительные ситуации.

---

### Типовые сценарии использования

- ✅ Логирование ошибок
- ✅ Возврат пользовательского (custom) ответа об ошибке
- ✅ Отправка уведомлений об ошибках
- ✅ Реализация fallback-ответов
- ✅ Трансформация ошибок

---

### Что важно понимать

`on-error` section срабатывает, если:

- backend возвращает ошибку;
- происходит таймаут;
- политика завершилась с исключением;
- сработал circuit breaker;
- нарушена валидация или политика безопасности.

Это позволяет:

- скрывать внутренние детали ошибок;
- возвращать унифицированный формат ошибок;
- предотвращать утечку технической информации;
- реализовать graceful degradation.

---

### Архитектурное значение

Раздел `on-error` повышает устойчивость системы и улучшает пользовательский опыт.

Он помогает:

- централизованно управлять обработкой ошибок;
- реализовать стандарт error-handling;
- соблюдать требования безопасности и комплаенса.

---

### Важно для AZ-204

Если требуется:
- вернуть кастомный HTTP-ответ при ошибке,
- перехватить исключение,
- отправить уведомление при сбое,
- реализовать fallback-логику,

— используется `on-error` policy.
**Example**:
```xml
<on-error>
  <!-- Log error to Application Insights -->
  <trace source="error-handler">
    @{
      return string.Format("Error: {0}", context.LastError.Message);
    }
  </trace>
  
  <!-- Return custom error response -->
  <return-response>
    <set-status code="500" reason="Internal Server Error" />
    <set-header name="Content-Type" exists-action="override">
      <value>application/json</value>
    </set-header>
    <set-body>@{
      return new JObject(
        new JProperty("error", "An error occurred"),
        new JProperty("message", context.LastError.Message),
        new JProperty("timestamp", DateTime.UtcNow)
      ).ToString();
    }</set-body>
  </return-response>
</on-error>
```

---

## Policy Expressions

**Policy expressions** are C# code snippets enclosed in `@(...)` that execute during policy evaluation.

### Syntax

```xml
<!-- Inline expression -->
<set-header name="X-User-Id" exists-action="override">
  <value>@(context.User.Id)</value>
</set-header>

<!-- Multi-line expression -->
<set-header name="X-Custom" exists-action="override">
  <value>@{
    string value = "default";
    if (context.User != null) {
      value = context.User.Email;
    }
    return value;
  }</value>
</set-header>
```

### Context Object

Переменная `context` предоставляет доступ к информации о текущем запросе, ответе и окружении выполнения политики.

Она используется внутри policy-выражений для динамической логики и условной обработки.

---

| Свойство | Описание | Пример |
|-----------|------------|---------|
| `context.Api` | Информация о текущем API | `context.Api.Id` |
| `context.Deployment` | Информация о развертывании | `context.Deployment.Region` |
| `context.Operation` | Текущая операция API | `context.Operation.Id` |
| `context.Product` | Текущий продукт | `context.Product.Name` |
| `context.Request` | HTTP-запрос | `context.Request.Headers`, `context.Request.Body` |
| `context.Response` | HTTP-ответ | `context.Response.StatusCode` |
| `context.Subscription` | Текущая подписка | `context.Subscription.Key` |
| `context.User` | Текущий пользователь | `context.User.Id`, `context.User.Email` |
| `context.Variables` | Пользовательские переменные | `context.Variables["myvar"]` |
| `context.LastError` | Последняя ошибка (только в on-error) | `context.LastError.Message` |

---

### Что важно понимать

`context` позволяет:

- реализовывать условные политики;
- динамически изменять backend URL;
- проверять заголовки и параметры;
- анализировать статус ответа;
- использовать данные пользователя и подписки;
- работать с ошибками в `on-error`.

---

### Архитектурное значение

Объект `context` — основа динамической логики в API Management Policies.

С его помощью можно:

- адаптировать поведение API под регион или продукт;
- применять разные политики для разных пользователей;
- реализовывать сложные сценарии маршрутизации;
- создавать кастомную обработку ошибок.

---

### Важно для AZ-204

Если в вопросе говорится о:
- доступе к заголовкам запроса,
- проверке региона развертывания,
- использовании данных пользователя или подписки,
- условной логике в policy,

— используется объект `context`.

### Common Expressions

**Get Request Header**:
```xml
<set-variable name="auth" value="@(context.Request.Headers.GetValueOrDefault("Authorization"))" />
```

**Get Query Parameter**:
```xml
<set-variable name="userId" value="@(context.Request.Url.Query.GetValueOrDefault("userId"))" />
```

**Get URL Path Parameter**:
```xml
<set-variable name="id" value="@(context.Request.MatchedParameters["id"])" />
```

**Get Request Body**:
```xml
<set-variable name="requestBody" value="@(context.Request.Body.As<string>())" />

<!-- Parse JSON body -->
<set-variable name="userEmail" value="@{
  var body = context.Request.Body.As<JObject>();
  return body["email"].ToString();
}" />
```

**Conditional Logic**:
```xml
<set-header name="X-Environment" exists-action="override">
  <value>@{
    return context.Deployment.Region == "East US" ? "production" : "staging";
  }</value>
</set-header>
```

**Random Selection**:
```xml
<set-backend-service base-url="@{
  var backends = new[] {
    "https://backend1.contoso.com",
    "https://backend2.contoso.com"
  };
  return backends[new Random().Next(backends.Length)];
}" />
```

---

## Policy Scopes

Policies can be applied at different scopes, creating a hierarchy:

```
Global Scope
    │
    ├─→ Product Scope
    │       │
    │       └─→ API Scope
    │               │
    │               └─→ Operation Scope
```

### 1. **Global Scope**

Applies to **all APIs** in the API Management instance.

**Use Cases**:
- Common authentication
- Logging
- CORS
- Security headers

**Configuration**:
```bash
# Azure Portal: APIs → All APIs → Policies
```

**Example**:
```xml
<policies>
  <inbound>
    <!-- Applied to ALL APIs -->
    <ip-filter action="allow">
      <address>203.0.113.0/24</address>
    </ip-filter>
    <set-header name="X-API-Gateway" exists-action="override">
      <value>Azure APIM</value>
    </set-header>
  </inbound>
</policies>
```

### 2. **Product Scope**

Applies to **all APIs in a product**.

**Use Cases**:
- Product-specific rate limits
- Product-specific quotas
- Subscription validation

**Example**:
```xml
<policies>
  <inbound>
    <!-- Limit: 1000 calls/month for this product -->
    <quota calls="1000" renewal-period="2592000" />
    <rate-limit calls="10" renewal-period="60" />
    <base />
  </inbound>
</policies>
```

### 3. **API Scope**

Applies to **all operations in an API**.

**Use Cases**:
- API-specific authentication
- API-specific backend URL
- API-specific transformation

**Example**:
```xml
<policies>
  <inbound>
    <set-backend-service base-url="https://users-api.contoso.com" />
    <base />
  </inbound>
  <outbound>
    <json-to-xml apply="always" consider-accept-header="false" />
    <base />
  </outbound>
</policies>
```

### 4. **Operation Scope**

Applies to **a specific operation** (endpoint).

**Use Cases**:
- Operation-specific validation
- Operation-specific transformation
- Operation-specific caching

**Example**:
```xml
<policies>
  <inbound>
    <!-- Cache GET requests only -->
    <cache-lookup vary-by-developer="false" vary-by-developer-groups="false" />
    <base />
  </inbound>
  <outbound>
    <cache-store duration="600" />
    <base />
  </outbound>
</policies>
```

### Policy Inheritance with `<base />`

The `<base />` element controls policy inheritance:

**Without `<base />`**: Parent policies are **ignored**
```xml
<inbound>
  <rate-limit calls="100" renewal-period="60" />
  <!-- Parent policies NOT executed -->
</inbound>
```

**With `<base />` at start**: Parent policies execute **first**
```xml
<inbound>
  <base />
  <!-- Parent policies execute BEFORE this -->
  <rate-limit calls="100" renewal-period="60" />
</inbound>
```

**With `<base />` at end**: Parent policies execute **last**
```xml
<inbound>
  <rate-limit calls="100" renewal-period="60" />
  <!-- Parent policies execute AFTER this -->
  <base />
</inbound>
```

**Example Hierarchy**:

## Scope и порядок выполнения Policies

В Azure API Management политики могут применяться на разных уровнях (scope):

- **Global** — ко всем API в экземпляре APIM
- **Product** — к API, опубликованным в конкретном продукте
- **API** — к конкретному API
- **Operation** — к отдельной операции (endpoint)

---

### Пример

- Global: IP filter
- Product: Quota (1000/month)
- API: Set backend URL
- Operation: Cache response

---

### Порядок выполнения (если используется `<base />` в начале)

1. IP filter (Global)
2. Quota (Product)
3. Set backend URL (API)
4. Cache response (Operation)

Политики выполняются от более общего уровня к более конкретному.

---

## Как определить, какая policy к какому уровню относится?

### 1️⃣ По месту настройки в Azure Portal

Уровень определяется тем, **где именно вы добавили политику**:

- Если политика добавлена в разделе **All APIs** → это Global
- Если в конкретном **Product** → это Product-level
- Если внутри конкретного API → это API-level
- Если внутри конкретной операции → это Operation-level

То есть scope задаётся не самой XML-политикой, а контекстом её применения.

---

### 2️⃣ По поведению

- Если правило применяется ко всем API — это Global
- Если ограничение зависит от подписки/продукта — это Product
- Если правило специфично для одного API — это API
- Если правило касается конкретного endpoint — это Operation

---

### 3️⃣ Роль `<base />`

`<base />` наследует политики с более высокого уровня.

Если `<base />` размещён в начале секции, порядок будет:

Global → Product → API → Operation

Если `<base />` отсутствует, политики верхнего уровня не будут выполнены.

---

## Важно для AZ-204

На экзамене могут спросить:

- В каком порядке выполняются политики?
- Где нужно настроить quota?
- Где реализовать IP-фильтрацию?
- Где кэшировать только конкретный endpoint?

Запомнить просто:

- Безопасность для всех API → Global
- Ограничения по подписке → Product
- Конфигурация backend → API
- Кэширование/логика конкретного метода → Operation
- `<base />` управляет наследованием.
```
Global: IP filter
Product: Quota (1000/month)
API: Set backend URL
Operation: Cache response

Execution order (with <base /> at start):
1. IP filter (global)
2. Quota (product)
3. Set backend URL (API)
4. Cache response (operation)
```

---

## Common Policy Examples

### 1. **Rate Limiting**

```xml
<policies>
  <inbound>
    <!-- 100 calls per minute per subscription -->
    <rate-limit calls="100" renewal-period="60" />
    <base />
  </inbound>
</policies>
```

### 2. **Quota (Monthly Limit)**

```xml
<policies>
  <inbound>
    <!-- 10,000 calls per month per subscription -->
    <quota calls="10000" renewal-period="2592000" />
    <base />
  </inbound>
</policies>
```

### 3. **IP Filtering**

```xml
<policies>
  <inbound>
    <ip-filter action="allow">
      <address>203.0.113.0/24</address>
      <address>198.51.100.14</address>
    </ip-filter>
    <base />
  </inbound>
</policies>
```

### 4. **JWT Validation**

```xml
<policies>
  <inbound>
    <validate-jwt header-name="Authorization" failed-validation-httpcode="401">
      <openid-config url="https://login.microsoftonline.com/<tenant-id>/v2.0/.well-known/openid-configuration" />
      <required-claims>
        <claim name="aud">
          <value>api://my-api</value>
        </claim>
      </required-claims>
    </validate-jwt>
    <base />
  </inbound>
</policies>
```

### 5. **Set Header**

```xml
<policies>
  <inbound>
    <set-header name="X-Custom-Header" exists-action="override">
      <value>CustomValue</value>
    </set-header>
    <base />
  </inbound>
  <outbound>
    <!-- Remove sensitive headers -->
    <set-header name="X-Powered-By" exists-action="delete" />
    <set-header name="Server" exists-action="delete" />
    <base />
  </outbound>
</policies>
```

### 6. **CORS**

```xml
<policies>
  <inbound>
    <cors allow-credentials="true">
      <allowed-origins>
        <origin>https://www.contoso.com</origin>
        <origin>https://app.contoso.com</origin>
      </allowed-origins>
      <allowed-methods preflight-result-max-age="300">
        <method>GET</method>
        <method>POST</method>
        <method>PUT</method>
        <method>DELETE</method>
      </allowed-methods>
      <allowed-headers>
        <header>Content-Type</header>
        <header>Authorization</header>
      </allowed-headers>
      <expose-headers>
        <header>X-Custom-Header</header>
      </expose-headers>
    </cors>
    <base />
  </inbound>
</policies>
```

### 7. **Response Caching**

```xml
<policies>
  <inbound>
    <cache-lookup vary-by-developer="false" vary-by-developer-groups="false">
      <vary-by-query-parameter>category</vary-by-query-parameter>
      <vary-by-query-parameter>page</vary-by-query-parameter>
    </cache-lookup>
    <base />
  </inbound>
  <outbound>
    <!-- Cache for 1 hour -->
    <cache-store duration="3600" />
    <base />
  </outbound>
</policies>
```

### 8. **Filter Response Content**

```xml
<policies>
  <inbound>
    <base />
  </inbound>
  <outbound>
    <base />
    <!-- Filter response based on product -->
    <choose>
      <when condition="@(context.Product.Name == "Starter")">
        <!-- Remove sensitive fields for Starter product -->
        <set-body>@{
          var response = context.Response.Body.As<JObject>();
          response.Remove("ssn");
          response.Remove("salary");
          return response.ToString();
        }</set-body>
      </when>
    </choose>
  </outbound>
</policies>
```

### 9. **Transform Request Body**

```xml
<policies>
  <inbound>
    <set-body>@{
      var body = context.Request.Body.As<JObject>();
      // Add timestamp
      body["timestamp"] = DateTime.UtcNow.ToString("o");
      // Add API version
      body["apiVersion"] = "1.0";
      return body.ToString();
    }</set-body>
    <base />
  </inbound>
</policies>
```

### 10. **Conditional Backend Routing**

```xml
<policies>
  <backend>
    <choose>
      <when condition="@(context.Request.Headers.GetValueOrDefault("X-Environment") == "production")">
        <set-backend-service base-url="https://prod.backend.com" />
      </when>
      <when condition="@(context.Request.Headers.GetValueOrDefault("X-Environment") == "staging")">
        <set-backend-service base-url="https://staging.backend.com" />
      </when>
      <otherwise>
        <set-backend-service base-url="https://dev.backend.com" />
      </otherwise>
    </choose>
    <forward-request />
  </backend>
</policies>
```

---

## Complete Policy Example

Here's a comprehensive policy with all sections:

```xml
<policies>
  <inbound>
    <!-- 1. Validate subscription key -->
    <check-header name="Ocp-Apim-Subscription-Key" failed-check-httpcode="401" />
    
    <!-- 2. IP filtering -->
    <ip-filter action="allow">
      <address>203.0.113.0/24</address>
    </ip-filter>
    
    <!-- 3. Rate limiting -->
    <rate-limit calls="100" renewal-period="60" />
    <quota calls="10000" renewal-period="2592000" />
    
    <!-- 4. CORS -->
    <cors allow-credentials="true">
      <allowed-origins>
        <origin>https://www.contoso.com</origin>
      </allowed-origins>
      <allowed-methods>
        <method>*</method>
      </allowed-methods>
    </cors>
    
    <!-- 5. Add request headers -->
    <set-header name="X-Correlation-Id" exists-action="override">
      <value>@(Guid.NewGuid().ToString())</value>
    </set-header>
    <set-header name="X-User-Id" exists-action="override">
      <value>@(context.User?.Id ?? "anonymous")</value>
    </set-header>
    
    <!-- 6. Cache lookup -->
    <cache-lookup vary-by-developer="false" />
    
    <!-- 7. Inherit parent policies -->
    <base />
  </inbound>
  
  <backend>
    <!-- 8. Set backend URL dynamically -->
    <set-backend-service base-url="@{
      return context.Request.Headers.GetValueOrDefault("X-Environment") == "production" 
        ? "https://prod.backend.com"
        : "https://test.backend.com";
    }" />
    
    <!-- 9. Forward with retry -->
    <retry condition="@(context.Response.StatusCode >= 500)" count="3" interval="5">
      <forward-request timeout="30" />
    </retry>
  </backend>
  
  <outbound>
    <!-- 10. Remove security-sensitive headers -->
    <set-header name="Server" exists-action="delete" />
    <set-header name="X-Powered-By" exists-action="delete" />
    <set-header name="X-AspNet-Version" exists-action="delete" />
    
    <!-- 11. Add response headers -->
    <set-header name="X-Response-Time" exists-action="override">
      <value>@(context.Elapsed.TotalMilliseconds.ToString())</value>
    </set-header>
    
    <!-- 12. Filter sensitive data for non-premium products -->
    <choose>
      <when condition="@(context.Product.Name != "Premium")">
        <set-body>@{
          var response = context.Response.Body.As<JObject>();
          if (response["ssn"] != null) response.Remove("ssn");
          if (response["creditCard"] != null) response.Remove("creditCard");
          return response.ToString();
        }</set-body>
      </when>
    </choose>
    
    <!-- 13. Cache response -->
    <cache-store duration="3600" />
    
    <base />
  </outbound>
  
  <on-error>
    <!-- 14. Log error -->
    <trace source="error-handler" severity="error">
      @{
        return new JObject(
          new JProperty("message", context.LastError.Message),
          new JProperty("source", context.LastError.Source),
          new JProperty("reason", context.LastError.Reason),
          new JProperty("timestamp", DateTime.UtcNow)
        ).ToString();
      }
    </trace>
    
    <!-- 15. Return custom error response -->
    <return-response>
      <set-status code="500" reason="Internal Server Error" />
      <set-header name="Content-Type" exists-action="override">
        <value>application/json</value>
      </set-header>
      <set-body>@{
        return new JObject(
          new JProperty("error", true),
          new JProperty("message", "An error occurred processing your request"),
          new JProperty("correlationId", context.Request.Headers.GetValueOrDefault("X-Correlation-Id")),
          new JProperty("timestamp", DateTime.UtcNow)
        ).ToString();
      }</set-body>
    </return-response>
  </on-error>
</policies>
```

---

## Best Practices

### 1. **Use `<base />` for Policy Inheritance**

✅ **Do**: Include `<base />` to inherit parent policies
```xml
<inbound>
  <base />
  <rate-limit calls="100" renewal-period="60" />
</inbound>
```

❌ **Don't**: Omit `<base />` unless intentional override

### 2. **Apply Policies at Appropriate Scope**

✅ **Do**:
- Global: Authentication, logging, CORS
- Product: Quotas, rate limits (per tier)
- API: Backend URL, API-specific transformation
- Operation: Caching (GET only), operation-specific validation

### 3. **Validate Input Early (inbound)**

✅ **Do**: Validate requests before forwarding to backend
```xml
<inbound>
  <check-header name="Content-Type" failed-check-httpcode="415">
    <value>application/json</value>
  </check-header>
  <base />
</inbound>
```

### 4. **Handle Errors Gracefully**

✅ **Do**: Use `<on-error>` for custom error responses
```xml
<on-error>
  <return-response>
    <set-status code="500" />
    <set-body>{"error": "Service unavailable"}</set-body>
  </return-response>
</on-error>
```

### 5. **Remove Sensitive Headers (outbound)**

✅ **Do**: Remove server implementation details
```xml
<outbound>
  <set-header name="Server" exists-action="delete" />
  <set-header name="X-Powered-By" exists-action="delete" />
  <base />
</outbound>
```

### 6. **Use Named Values for Configuration**

✅ **Do**: Store backend URLs, keys in Named Values
```xml
<set-backend-service base-url="{{backend-url}}" />
```

```bash
az apim nv create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --named-value-id backend-url \
  --value "https://backend.contoso.com"
```

---

## Советы к экзамену

### Ключевые концепции для AZ-204

1. **Четыре раздела policy**:  
   `inbound`, `backend`, `outbound`, `on-error`

2. **Порядок выполнения**:  
   `inbound → backend → outbound`  
   (или `on-error`, если возникает ошибка)

3. **Policy expressions**:  
   C#-выражения внутри `@(...)` или `@{ ... }`

4. **Объект context**:  
   Доступ к данным запроса/ответа через переменную `context`

5. **Scope политик**:  
   Global > Product > API > Operation

6. **Элемент `<base />`**:  
   Управляет наследованием политик родительского уровня

7. **Часто используемые политики**:  
   `rate-limit`, `quota`, `cache`, `set-header`, `CORS`, проверка JWT

8. **Где можно применять политики**:  
   Azure Portal, Azure CLI, REST API, ARM templates

9. **Тестирование политик**:  
   Test console в Developer Portal

10. **Обработка ошибок**:  
    Используйте раздел `<on-error>` для возврата кастомных ошибок

---

## Частые экзаменационные сценарии

**Сценарий 1**:  
"Ограничить количество вызовов API до 1000 в месяц на подписку"  
→ **Ответ**: Использовать `<quota calls="1000" renewal-period="2592000" />` в Product policy

---

**Сценарий 2**:  
"Добавить кастомный заголовок ко всем запросам"  
→ **Ответ**: Использовать `<set-header>` в Global inbound policy

---

**Сценарий 3**:  
"Кэшировать GET-ответы на 1 час"  
→ **Ответ**: Использовать `<cache-lookup>` в inbound и  
`<cache-store duration="3600">` в outbound (уровень Operation)

---

**Сценарий 4**:  
"Маршрутизировать запросы в разные backend в зависимости от заголовка"  
→ **Ответ**: Использовать `<choose>` с `<set-backend-service>` в разделе backend

---

**Сценарий 5**:  
"Удалить чувствительные данные из ответа для пользователей бесплатного тарифа"  
→ **Ответ**: Использовать `<choose>` с `<set-body>` в разделе outbound, проверяя `context.Product.Name`

---

### Финальный акцент для AZ-204

- Если требуется изменить запрос → `inbound`
- Если требуется управлять backend → `backend`
- Если нужно изменить ответ → `outbound`
- Если требуется обработка ошибки → `on-error`
- Ограничения по подписке → Product scope
- Общая безопасность → Global scope
- Кэширование конкретного метода → Operation scope
- `<base />` влияет на порядок выполнения

Экзаменационные вопросы часто проверяют не синтаксис, а понимание **где и в каком разделе должна быть реализована политика**.

## Quick Reference Commands

```bash
# Apply policy to all APIs (Global)
az apim api policy create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --xml-policy @policy.xml

# Apply policy to specific API
az apim api policy create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --api-id my-api \
  --xml-policy @policy.xml

# Apply policy to specific operation
az apim api operation policy create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --api-id my-api \
  --operation-id get-users \
  --xml-policy @policy.xml

# Get current policy
az apim api policy show \
  --resource-group rg-apim \
  --service-name apim-instance \
  --api-id my-api

# Delete policy
az apim api policy delete \
  --resource-group rg-apim \
  --service-name apim-instance \
  --api-id my-api

# Create named value
az apim nv create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --named-value-id backend-url \
  --value "https://backend.contoso.com"
```

---

## Learn More

- [API Management Policies Reference](https://docs.microsoft.com/azure/api-management/api-management-policies)
- [Policy Expressions](https://docs.microsoft.com/azure/api-management/api-management-policy-expressions)
- [Advanced Policies](https://docs.microsoft.com/azure/api-management/api-management-advanced-policies)
- [Error Handling](https://docs.microsoft.com/azure/api-management/api-management-error-handling-policies)
