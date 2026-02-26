# Создание Advanced Policies

## Обзор

**Advanced policies** обеспечивают более сложное и гибкое управление поведением API, включая условную логику, маршрутизацию запросов, контроль конкурентности, логирование, мок-ответы, повторы (retries) и кастомные ответы.

Такие политики обычно применяются в разделах **backend** и **outbound** для реализации сложных сценариев.

---

## 1. Control Flow Policy

Политики **control-flow** позволяют выполнять набор операторов **условно**, на основе булевых выражений (boolean expressions).

Их используют, когда нужно:

- разветвлять логику обработки запросов/ответов;
- выбирать backend в зависимости от заголовков/параметров;
- по-разному обрабатывать пользователей, продукты, подписки;
- возвращать разные ответы при разных условиях.

---

### Архитектурный смысл (дополнение)

Control-flow — это способ реализовать «умный gateway», не трогая backend-код.

Чаще всего control-flow строится вокруг конструкций вроде:

- условного выбора (if/else логика),
- остановки pipeline,
- установки переменных для последующих шагов.

---

### Важно для AZ-204

Если в вопросе звучит:
- “в зависимости от заголовка/параметра сделать X иначе Y”,
- “маршрутизировать в разные backend по условию”,
- “для одного продукта вернуть один формат, для другого — другой”,

— это прямой сигнал к применению control-flow логики в policies (обычно через `choose/when/otherwise` и выражения на `context`).

### Syntax

```xml
<choose>
  <when condition="Boolean expression">
    <!-- Policy statements if condition is true -->
  </when>
  <when condition="Boolean expression">
    <!-- Policy statements if condition is true -->
  </when>
  <otherwise>
    <!-- Policy statements if no conditions match -->
  </otherwise>
</choose>
```

### Example 1: Route Based on Environment

```xml
<policies>
  <backend>
    <choose>
      <when condition="@(context.Request.Headers.GetValueOrDefault("X-Environment") == "production")">
        <set-backend-service base-url="https://prod.backend.com/api" />
      </when>
      <when condition="@(context.Request.Headers.GetValueOrDefault("X-Environment") == "staging")">
        <set-backend-service base-url="https://staging.backend.com/api" />
      </when>
      <otherwise>
        <set-backend-service base-url="https://dev.backend.com/api" />
      </otherwise>
    </choose>
    <forward-request />
  </backend>
</policies>
```

### Example 2: Different Rate Limits by User Group

```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.User.Groups.Any(g => g.Name == "Premium"))">
        <!-- Premium users: 1000 calls/minute -->
        <rate-limit calls="1000" renewal-period="60" />
      </when>
      <when condition="@(context.User.Groups.Any(g => g.Name == "Standard"))">
        <!-- Standard users: 100 calls/minute -->
        <rate-limit calls="100" renewal-period="60" />
      </when>
      <otherwise>
        <!-- Free users: 10 calls/minute -->
        <rate-limit calls="10" renewal-period="60" />
      </otherwise>
    </choose>
    <base />
  </inbound>
</policies>
```

### Example 3: Filter Response Content by Product

```xml
<policies>
  <outbound>
    <choose>
      <when condition="@(context.Product.Name == "Starter")">
        <!-- Remove sensitive fields for Starter product -->
        <set-body>@{
          var response = context.Response.Body.As<JObject>();
          response.Remove("ssn");
          response.Remove("salary");
          response.Remove("creditCard");
          return response.ToString();
        }</set-body>
      </when>
      <when condition="@(context.Product.Name == "Basic")">
        <!-- Remove some fields for Basic product -->
        <set-body>@{
          var response = context.Response.Body.As<JObject>();
          response.Remove("ssn");
          return response.ToString();
        }</set-body>
      </when>
      <!-- Premium product: return full response -->
    </choose>
    <base />
  </outbound>
</policies>
```

### Example 4: API Versioning

```xml
<policies>
  <backend>
    <choose>
      <when condition="@(context.Request.Headers.GetValueOrDefault("Api-Version") == "2.0")">
        <set-backend-service base-url="https://api.contoso.com/v2" />
      </when>
      <when condition="@(context.Request.Headers.GetValueOrDefault("Api-Version") == "1.0")">
        <set-backend-service base-url="https://api.contoso.com/v1" />
      </when>
      <otherwise>
        <!-- Default to latest version -->
        <set-backend-service base-url="https://api.contoso.com/v2" />
      </otherwise>
    </choose>
    <forward-request />
  </backend>
</policies>
```

---

## 2. Forward Request Policy

The **forward-request** policy forwards the request to the backend service specified in the request context.

### Syntax

```xml
<forward-request timeout="time in seconds" follow-redirects="true | false" />
```

### Параметры

| Параметр | Описание | Значение по умолчанию |
|-----------|------------|------------------------|
| `timeout` | Максимальное время ожидания ответа от backend (в секундах) | 300 |
| `follow-redirects` | Следовать HTTP-редиректам от backend | `true` |

---

### Пояснение

**`timeout`**  
Определяет, сколько секунд gateway будет ожидать ответа от backend-сервиса.

Если время превышено:
- запрос завершится ошибкой;
- управление перейдёт в раздел `on-error` (если он настроен).

Это важно для:
- предотвращения зависаний;
- защиты от медленных сервисов;
- повышения устойчивости системы.

---

**`follow-redirects`**  
Определяет, будет ли gateway автоматически следовать HTTP-редиректам (например, 301 или 302), возвращённым backend.

- `true` — gateway перейдёт по новому URL
- `false` — редирект будет возвращён клиенту

---

### Архитектурные рекомендации

- Устанавливайте разумный `timeout` для предотвращения долгих ожиданий.
- В микросервисной архитектуре лучше минимизировать использование редиректов на уровне backend.

---

### Важно для AZ-204

Если в вопросе говорится о:
- настройке времени ожидания ответа,
- обработке таймаутов,
- контроле поведения при HTTP-редиректах,

— следует обратить внимание на параметры `timeout` и `follow-redirects`.

### Example 1: Basic Forward with Timeout

```xml
<policies>
  <backend>
    <forward-request timeout="30" />
  </backend>
</policies>
```

### Example 2: Don't Follow Redirects

```xml
<policies>
  <backend>
    <forward-request timeout="60" follow-redirects="false" />
  </backend>
</policies>
```

### Example 3: Dynamic Timeout Based on Operation

```xml
<policies>
  <backend>
    <choose>
      <when condition="@(context.Operation.Id == "long-running-operation")">
        <!-- Long timeout for slow operations -->
        <forward-request timeout="120" />
      </when>
      <otherwise>
        <!-- Short timeout for fast operations -->
        <forward-request timeout="10" />
      </otherwise>
    </choose>
  </backend>
</policies>
```

### Example 4: Skip Backend Call (for testing)

```xml
<policies>
  <backend>
    <!-- Don't call backend - return mock response -->
    <!-- Omit <forward-request /> -->
  </backend>
  <outbound>
    <return-response>
      <set-status code="200" />
      <set-body>{"message": "Mock response for testing"}</set-body>
    </return-response>
  </outbound>
</policies>
```

---

## 3. Limit Concurrency Policy

The **limit-concurrency** policy prevents more than the specified number of requests from executing at any time.

### Syntax

```xml
<limit-concurrency key="expression" max-count="number">
  <!-- Policy statements to execute with concurrency limit -->
</limit-concurrency>
```

### Параметры

| Параметр | Описание | Обязательный |
|------------|------------|--------------|
| `key` | Выражение для группировки конкурентных запросов | Нет (по умолчанию — на экземпляр) |
| `max-count` | Максимальное количество одновременных запросов | Да |

---

### Пояснение

Эти параметры используются для управления **конкурентностью (concurrency control)** на уровне gateway.

**`max-count`**  
Определяет максимальное число одновременно выполняемых запросов.  
Когда лимит превышен — новые запросы будут ожидать или завершаться ошибкой (в зависимости от конфигурации).

**`key`**  
Позволяет группировать ограничения по определённому признаку.

Например, можно ограничивать:

- по пользователю,
- по подписке,
- по региону,
- по заголовку запроса.

Если `key` не задан, ограничение применяется ко всему экземпляру gateway.

---

### Архитектурное значение

Control concurrency позволяет:

- защитить backend от перегрузки;
- предотвратить ресурсное истощение;
- обеспечить стабильную производительность;
- реализовать «bulkhead»-подобную стратегию изоляции.

---

### Важно для AZ-204

Если в вопросе говорится о:
- ограничении количества одновременных запросов,
- защите backend от параллельной нагрузки,
- управлении конкурентностью,

— решение связано с использованием политики контроля конкурентности и параметра `max-count`.

### Example 1: Global Concurrency Limit

```xml
<policies>
  <backend>
    <!-- Allow max 100 concurrent requests to backend -->
    <limit-concurrency max-count="100">
      <forward-request timeout="30" />
    </limit-concurrency>
  </backend>
</policies>
```

**Behavior**: If 100 requests are already being processed, the 101st request receives `429 Too Many Requests`.

### Example 2: Per-User Concurrency Limit

```xml
<policies>
  <backend>
    <!-- Each user can have max 5 concurrent requests -->
    <limit-concurrency key="@(context.User.Id)" max-count="5">
      <forward-request timeout="30" />
    </limit-concurrency>
  </backend>
</policies>
```

### Example 3: Per-Subscription Concurrency Limit

```xml
<policies>
  <backend>
    <!-- Each subscription can have max 10 concurrent requests -->
    <limit-concurrency key="@(context.Subscription.Key)" max-count="10">
      <forward-request timeout="30" />
    </limit-concurrency>
  </backend>
</policies>
```

### Example 4: Protect Backend Service

```xml
<policies>
  <backend>
    <!-- Prevent overwhelming backend -->
    <limit-concurrency max-count="50">
      <retry condition="@(context.Response.StatusCode == 503)" count="3" interval="5">
        <forward-request timeout="60" />
      </retry>
    </limit-concurrency>
  </backend>
  <on-error>
    <choose>
      <when condition="@(context.LastError.Reason == "ConcurrencyLimitReached")">
        <return-response>
          <set-status code="429" reason="Too Many Concurrent Requests" />
          <set-body>{"error": "Server is busy, please try again later"}</set-body>
        </return-response>
      </when>
    </choose>
  </on-error>
</policies>
```

---

## 4. Log to Event Hub Policy

The **log-to-eventhub** policy sends messages to an Event Hub for logging and analytics.

### Syntax

```xml
<log-to-eventhub logger-id="logger-id">
  @{
    return "message to log";
  }
</log-to-eventhub>
```

### Prerequisites

1. Create Event Hub
2. Register Event Hub logger in APIM
3. Reference logger in policy

### Setup Event Hub Logger

```bash
# Create Event Hubs namespace
az eventhubs namespace create \
  --name eh-namespace \
  --resource-group rg-apim \
  --location eastus \
  --sku Standard

# Create Event Hub
az eventhubs eventhub create \
  --name api-logs \
  --namespace-name eh-namespace \
  --resource-group rg-apim

# Get connection string
az eventhubs namespace authorization-rule keys list \
  --resource-group rg-apim \
  --namespace-name eh-namespace \
  --name RootManageSharedAccessKey \
  --query primaryConnectionString -o tsv

# Register logger in APIM (Azure Portal)
# APIs → Loggers → Add → Event Hub
```

### Example 1: Log All Requests

```xml
<policies>
  <inbound>
    <log-to-eventhub logger-id="apim-logger">
      @{
        return new JObject(
          new JProperty("timestamp", DateTime.UtcNow),
          new JProperty("method", context.Request.Method),
          new JProperty("url", context.Request.Url.ToString()),
          new JProperty("userId", context.User?.Id ?? "anonymous"),
          new JProperty("subscriptionKey", context.Subscription?.Key ?? "none")
        ).ToString();
      }
    </log-to-eventhub>
    <base />
  </inbound>
</policies>
```

### Example 2: Log Errors Only

```xml
<policies>
  <on-error>
    <log-to-eventhub logger-id="error-logger">
      @{
        return new JObject(
          new JProperty("timestamp", DateTime.UtcNow),
          new JProperty("errorMessage", context.LastError.Message),
          new JProperty("errorSource", context.LastError.Source),
          new JProperty("errorReason", context.LastError.Reason),
          new JProperty("requestUrl", context.Request.Url.ToString()),
          new JProperty("userId", context.User?.Id ?? "anonymous"),
          new JProperty("statusCode", context.Response?.StatusCode ?? 0)
        ).ToString();
      }
    </log-to-eventhub>
  </on-error>
</policies>
```

### Example 3: Log Slow Requests

```xml
<policies>
  <outbound>
    <choose>
      <when condition="@(context.Elapsed.TotalMilliseconds > 1000)">
        <!-- Log requests that take more than 1 second -->
        <log-to-eventhub logger-id="performance-logger">
          @{
            return new JObject(
              new JProperty("timestamp", DateTime.UtcNow),
              new JProperty("url", context.Request.Url.ToString()),
              new JProperty("duration", context.Elapsed.TotalMilliseconds),
              new JProperty("operation", context.Operation.Id),
              new JProperty("userId", context.User?.Id ?? "anonymous")
            ).ToString();
          }
        </log-to-eventhub>
      </when>
    </choose>
    <base />
  </outbound>
</policies>
```

---

## 5. Mock Response Policy

The **mock-response** policy returns a mocked response without calling the backend.

### Syntax

```xml
<mock-response status-code="code" content-type="type" />
```

### Example 1: Return Mock for Testing

```xml
<policies>
  <inbound>
    <!-- Mock responses when X-Mock header is present -->
    <choose>
      <when condition="@(context.Request.Headers.GetValueOrDefault("X-Mock") == "true")">
        <mock-response status-code="200" content-type="application/json" />
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### Example 2: Return Sample Response

```xml
<policies>
  <inbound>
    <base />
  </inbound>
  <backend>
    <!-- Skip backend call -->
  </backend>
  <outbound>
    <return-response>
      <set-status code="200" />
      <set-header name="Content-Type" exists-action="override">
        <value>application/json</value>
      </set-header>
      <set-body>@{
        return new JObject(
          new JProperty("id", 1),
          new JProperty("name", "Sample User"),
          new JProperty("email", "user@example.com"),
          new JProperty("createdAt", DateTime.UtcNow)
        ).ToString();
      }</set-body>
    </return-response>
  </outbound>
</policies>
```

### Example 3: Mock Different Responses by Product

```xml
<policies>
  <backend>
    <choose>
      <when condition="@(context.Product.Name == "Free")">
        <!-- Return limited mock for free tier -->
        <return-response>
          <set-status code="200" />
          <set-body>{"message": "Free tier: limited data"}</set-body>
        </return-response>
      </when>
      <otherwise>
        <!-- Call real backend for paid tiers -->
        <forward-request />
      </otherwise>
    </choose>
  </backend>
</policies>
```

---

## 6. Retry Policy

The **retry** policy executes child policies repeatedly until a condition is met or retry count is exhausted.

### Syntax

```xml
<retry condition="boolean expression" 
       count="number" 
       interval="seconds" 
       max-interval="seconds"
       delta="seconds"
       first-fast-retry="true | false">
  <!-- Policy statements to retry -->
</retry>
```

### Параметры

| Параметр | Описание | Значение по умолчанию |
|------------|------------|------------------------|
| `condition` | Выражение, которое проверяется после каждой попытки | Обязательный |
| `count` | Максимальное количество повторных попыток | Обязательный |
| `interval` | Интервал ожидания между попытками (в секундах) | Обязательный |
| `max-interval` | Максимальный интервал ожидания (для exponential backoff) | Необязательный |
| `delta` | Приращение для exponential backoff | Необязательный |
| `first-fast-retry` | Пропустить задержку перед первой повторной попыткой | `false` |

---

### Пояснение

Эти параметры используются в политике **retry**, которая позволяет повторять вызов backend при определённых условиях.

---

### `condition`

Булево выражение, которое определяет, нужно ли выполнять повторную попытку.

Обычно проверяется:
- код ответа (например, 5xx),
- таймаут,
- конкретные значения `context.Response.StatusCode`.

---

### `count`

Максимальное число повторных попыток.  
После достижения лимита запрос завершится ошибкой.

---

### `interval`

Фиксированная задержка между попытками (если не используется exponential backoff).

---

### `max-interval` и `delta`

Используются для реализации **exponential backoff** — увеличения задержки после каждой попытки.

Это снижает нагрузку на нестабильный backend и повышает устойчивость системы.

---

### `first-fast-retry`

Если `true`, первая повторная попытка выполняется без ожидания.  
Полезно при кратковременных сбоях.

---

### Архитектурное значение

Retry-политика повышает отказоустойчивость системы:

- сглаживает кратковременные сбои;
- уменьшает вероятность ошибок из-за временной недоступности;
- снижает нагрузку при использовании exponential backoff;
- улучшает стабильность интеграций.

Однако чрезмерное использование retry может:

- увеличить задержку ответа;
- усилить нагрузку на backend;
- усугубить ситуацию при массовом сбое.

---

### Важно для AZ-204

Если в вопросе говорится о:
- временных сбоях backend,
- необходимости повторить запрос при 5xx,
- реализации exponential backoff,
- повышении устойчивости API,

— следует использовать политику `retry` с параметрами `condition`, `count` и `interval`.

### Example 1: Retry on Server Errors

```xml
<policies>
  <backend>
    <!-- Retry up to 3 times on 5xx errors, wait 5 seconds between -->
    <retry condition="@(context.Response.StatusCode >= 500)" count="3" interval="5">
      <forward-request timeout="30" />
    </retry>
  </backend>
</policies>
```

### Example 2: Exponential Backoff

```xml
<policies>
  <backend>
    <!-- Retry with exponential backoff: 2s, 4s, 8s -->
    <retry condition="@(context.Response.StatusCode >= 500)" 
           count="3" 
           interval="2" 
           max-interval="10"
           delta="2">
      <forward-request timeout="30" />
    </retry>
  </backend>
</policies>
```

### Example 3: Retry Specific Error

```xml
<policies>
  <backend>
    <!-- Retry on 503 Service Unavailable only -->
    <retry condition="@(context.Response.StatusCode == 503)" 
           count="5" 
           interval="3"
           first-fast-retry="true">
      <forward-request timeout="60" />
    </retry>
  </backend>
</policies>
```

### Example 4: Retry with Fallback

```xml
<policies>
  <backend>
    <retry condition="@(context.Response.StatusCode >= 500)" count="2" interval="5">
      <forward-request timeout="30" />
    </retry>
  </backend>
  <on-error>
    <choose>
      <when condition="@(context.LastError.Reason == "RetryCountExceeded")">
        <!-- Switch to backup backend after retries exhausted -->
        <set-backend-service base-url="https://backup.contoso.com" />
        <forward-request timeout="30" />
      </when>
    </choose>
  </on-error>
</policies>
```

---

## 7. Return Response Policy

The **return-response** policy aborts pipeline execution and returns a response directly to the caller.

### Syntax

```xml
<return-response>
  <set-status code="code" reason="reason" />
  <set-header name="name" exists-action="override | skip | append | delete">
    <value>value</value>
  </set-header>
  <set-body>body</set-body>
</return-response>
```

### Example 1: Return Maintenance Message

```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Headers.GetValueOrDefault("X-Maintenance") == "true")">
        <!-- Return maintenance response immediately -->
        <return-response>
          <set-status code="503" reason="Service Unavailable" />
          <set-header name="Retry-After" exists-action="override">
            <value>3600</value>
          </set-header>
          <set-body>{"message": "Service under maintenance, please try again later"}</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### Example 2: Return Cached Response

```xml
<policies>
  <inbound>
    <cache-lookup vary-by-developer="false" vary-by-developer-groups="false">
      <vary-by-query-parameter>id</vary-by-query-parameter>
    </cache-lookup>
    
    <!-- If cache hit, return immediately -->
    <choose>
      <when condition="@(context.Variables.ContainsKey("cachedResponse"))">
        <return-response>
          <set-status code="200" />
          <set-header name="X-Cache" exists-action="override">
            <value>HIT</value>
          </set-header>
          <set-body>@((string)context.Variables["cachedResponse"])</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### Example 3: Block Requests from Specific IP

```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.IpAddress == "203.0.113.100")">
        <!-- Block and return 403 -->
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>{"error": "Access denied from your IP address"}</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### Example 4: API Deprecation Warning

```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Url.Path.Contains("/v1/"))">
        <!-- Warn about deprecated API -->
        <return-response>
          <set-status code="410" reason="Gone" />
          <set-header name="X-Deprecated-Version" exists-action="override">
            <value>v1</value>
          </set-header>
          <set-header name="X-New-Version" exists-action="override">
            <value>v2</value>
          </set-header>
          <set-body>@{
            return new JObject(
              new JProperty("error", "This API version is deprecated"),
              new JProperty("deprecatedVersion", "v1"),
              new JProperty("currentVersion", "v2"),
              new JProperty("migrationGuide", "https://docs.contoso.com/migrate-v1-to-v2")
            ).ToString();
          }</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

---

## Combining Advanced Policies

### Example: Production-Ready Policy

```xml
<policies>
  <inbound>
    <!-- 1. Check maintenance mode -->
    <choose>
      <when condition="@(context.Api.Id == "maintenance-api")">
        <return-response>
          <set-status code="503" reason="Under Maintenance" />
          <set-body>{"message": "Scheduled maintenance in progress"}</set-body>
        </return-response>
      </when>
    </choose>
    
    <!-- 2. Rate limiting by user group -->
    <choose>
      <when condition="@(context.User.Groups.Any(g => g.Name == "Premium"))">
        <rate-limit calls="1000" renewal-period="60" />
      </when>
      <otherwise>
        <rate-limit calls="100" renewal-period="60" />
      </otherwise>
    </choose>
    
    <!-- 3. Cache lookup -->
    <cache-lookup vary-by-developer="false" />
    
    <base />
  </inbound>
  
  <backend>
    <!-- 4. Limit concurrent requests -->
    <limit-concurrency key="@(context.Subscription.Key)" max-count="10">
      <!-- 5. Conditional backend routing -->
      <choose>
        <when condition="@(context.Request.Headers.GetValueOrDefault("X-Environment") == "production")">
          <set-backend-service base-url="https://prod.backend.com" />
        </when>
        <otherwise>
          <set-backend-service base-url="https://test.backend.com" />
        </otherwise>
      </choose>
      
      <!-- 6. Retry on errors with exponential backoff -->
      <retry condition="@(context.Response.StatusCode >= 500)" 
             count="3" 
             interval="2" 
             max-interval="10"
             delta="2"
             first-fast-retry="true">
        <forward-request timeout="30" />
      </retry>
    </limit-concurrency>
  </backend>
  
  <outbound>
    <!-- 7. Cache response -->
    <cache-store duration="3600" />
    
    <!-- 8. Log slow requests -->
    <choose>
      <when condition="@(context.Elapsed.TotalMilliseconds > 1000)">
        <log-to-eventhub logger-id="performance-logger">
          @{
            return new JObject(
              new JProperty("url", context.Request.Url.ToString()),
              new JProperty("duration", context.Elapsed.TotalMilliseconds),
              new JProperty("timestamp", DateTime.UtcNow)
            ).ToString();
          }
        </log-to-eventhub>
      </when>
    </choose>
    
    <base />
  </outbound>
  
  <on-error>
    <!-- 9. Log errors -->
    <log-to-eventhub logger-id="error-logger">
      @{
        return new JObject(
          new JProperty("error", context.LastError.Message),
          new JProperty("timestamp", DateTime.UtcNow)
        ).ToString();
      }
    </log-to-eventhub>
    
    <!-- 10. Return custom error response -->
    <return-response>
      <set-status code="500" reason="Internal Server Error" />
      <set-body>{"error": "An error occurred", "correlationId": "@(Guid.NewGuid().ToString())"}</set-body>
    </return-response>
  </on-error>
</policies>
```

---

## Best Practices

### 1. **Use `choose` for Complex Logic**

✅ **Do**: Use control-flow for conditional execution
```xml
<choose>
  <when condition="@(context.User.Groups.Any(g => g.Name == "Admin"))">
    <!-- Admin logic -->
  </when>
  <otherwise>
    <!-- Regular user logic -->
  </otherwise>
</choose>
```

### 2. **Implement Retry with Exponential Backoff**

✅ **Do**: Use increasing intervals for retries
```xml
<retry condition="@(context.Response.StatusCode >= 500)" 
       count="3" 
       interval="2" 
       delta="2">
```

### 3. **Limit Concurrency to Protect Backend**

✅ **Do**: Prevent backend overload
```xml
<limit-concurrency max-count="100">
  <forward-request timeout="30" />
</limit-concurrency>
```

### 4. **Log Important Events**

✅ **Do**: Log errors, slow requests, security events
```xml
<log-to-eventhub logger-id="audit-logger">
  @{ return "security event details"; }
</log-to-eventhub>
```

### 5. **Use Mock Responses for Development**

✅ **Do**: Enable mocking with header
```xml
<when condition="@(context.Request.Headers.GetValueOrDefault("X-Mock") == "true")">
  <mock-response status-code="200" content-type="application/json" />
</when>
```

---

## Советы к экзамену

### Ключевые концепции для AZ-204

1. **control-flow**  
   Используются конструкции `<choose>`, `<when>`, `<otherwise>` для реализации условной логики.

2. **forward-request**  
   Отправляет запрос в backend. Позволяет настраивать параметры, включая `timeout`.

3. **limit-concurrency**  
   Ограничивает количество одновременных запросов.  
   При превышении лимита возвращается HTTP 429 (Too Many Requests).

4. **log-to-eventhub**  
   Отправляет логи в Event Hub. Требует указания `logger-id`.

5. **mock-response**  
   Возвращает тестовый (mock) ответ без вызова backend.

6. **retry**  
   Реализует повторные попытки с параметрами `condition`, `count`, `interval` (возможен exponential backoff).

7. **return-response**  
   Немедленно завершает pipeline и возвращает кастомный ответ клиенту.

8. **Типовые сценарии использования**:

   - `control-flow` → Условная маршрутизация, разные правила для разных групп пользователей
   - `limit-concurrency` → Защита backend от перегрузки
   - `retry` → Обработка временных сбоев (transient failures)
   - `return-response` → Режим обслуживания, блокировка запросов, кэш-ответ

---

## Частые экзаменационные сценарии

**Сценарий 1**:  
"Маршрутизировать запросы в разные backend в зависимости от заголовка"  
→ **Ответ**: Использовать `<choose>` с `<set-backend-service>` в разделе backend

---

**Сценарий 2**:  
"Повторять неудачные запросы 3 раза с задержкой"  
→ **Ответ**: Использовать  
`<retry condition="@(context.Response.StatusCode >= 500)" count="3" interval="5">`

---

**Сценарий 3**:  
"Запретить более 100 одновременных запросов к backend"  
→ **Ответ**: Использовать `<limit-concurrency max-count="100">` вокруг `<forward-request>`

---

**Сценарий 4**:  
"Логировать все ошибки в Event Hub"  
→ **Ответ**: Использовать `<log-to-eventhub logger-id="...">` в разделе `<on-error>`

---

**Сценарий 5**:  
"Вернуть сообщение о техническом обслуживании без вызова backend"  
→ **Ответ**: Использовать `<return-response>` в разделе `<inbound>`

---

### Финальный акцент для AZ-204

- Если требуется условная логика → `control-flow`
- Если нужно защитить backend → `limit-concurrency`
- Если нужно повысить отказоустойчивость → `retry`
- Если нужно немедленно вернуть ответ → `return-response`
- Если требуется централизованное логирование → `log-to-eventhub`

Экзамен проверяет понимание **где и какую политику применять**, а не только знание синтаксиса.
---

## Learn More

- [Advanced Policies Reference](https://docs.microsoft.com/azure/api-management/api-management-advanced-policies)
- [Control Flow Policy](https://docs.microsoft.com/azure/api-management/api-management-advanced-policies#choose)
- [Retry Policy](https://docs.microsoft.com/azure/api-management/api-management-advanced-policies#Retry)
- [Event Hub Integration](https://docs.microsoft.com/azure/api-management/api-management-howto-log-event-hubs)
