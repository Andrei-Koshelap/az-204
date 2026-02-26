# Защита API с помощью Subscriptions

## Обзор

**Subscriptions (подписки)** — это основной механизм защиты API в Azure API Management.  
Подписка предоставляет доступ к API внутри продукта и идентифицируется с помощью **subscription key**.

**Назначение**: контроль и аутентификация доступа к API без использования сложных OAuth-флоу.

Это простой и эффективный способ защитить API, особенно для внутренних сервисов, партнерских интеграций и публичных API начального уровня.

---

## Что такое Subscription?

**Subscription** — это механизм авторизации, который предоставляет доступ к API.

### Ключевые характеристики

- ✅ Каждая подписка имеет **уникальный subscription key**
- ✅ Подписки имеют **scope** (All APIs, Single API или Product)
- ✅ Ключи **автоматически генерируются** Azure API Management
- ✅ Поддерживаются **primary и secondary ключи** для ротации без downtime
- ✅ Подписка может быть **приостановлена или отменена**
- ✅ Подписка привязана к **developer account**

---

## Subscription Keys

### Что такое Subscription Key?

**Subscription key** — это уникальная строка, используемая для аутентификации запросов к API.

- Передаётся в каждом запросе
- Проверяется на уровне API Gateway
- Не требует сложной логики на backend

**Формат**:  
`32-символьная шестнадцатеричная строка`

**Пример**:  
`a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6`

---

### Архитектурный смысл (дополнение)

Subscription-based security позволяет:

- быстро ограничить доступ к API;
- управлять доступом без изменения backend-кода;
- легко отзывать доступ (revocation);
- применять quota и rate limiting на уровне подписки;
- различать потребителей API.

Это не замена OAuth/JWT, а **дополнение или более простой альтернативный механизм**, часто используемый:

- для внутренних API;
- для B2B-интеграций;
- на ранних этапах API.

---

### Важно для AZ-204

Если в вопросе говорится о:
- простом механизме защиты API,
- использовании subscription key,
- контроле доступа без OAuth,
- применении quota или rate limit,

— речь идёт о **Subscriptions в Azure API Management**.
### How Subscription Keys Work

```
┌──────────────────┐
│  Client Request  │
│  + Subscription  │
│    Key           │
└────────┬─────────┘
         │
         ▼
┌────────────────────────────────────┐
│  API Gateway                       │
│  1. Extract subscription key       │
│  2. Validate key                   │
│  3. Check subscription state       │
│  4. Enforce rate limits/quotas     │
└────────┬───────────────────────────┘
         │
         ▼ (if valid)
┌────────────────────┐
│  Backend Service   │
└────────────────────┘

         │ (if invalid)
         ▼
┌────────────────────┐
│  401 Unauthorized  │
└────────────────────┘
```

### Primary и Secondary Keys

Каждая подписка содержит **два ключа**:

| Тип ключа | Назначение | Сценарий использования |
|------------|------------|------------------------|
| **Primary** | Основной рабочий ключ | Активные вызовы API |
| **Secondary** | Резервный ключ для ротации | Обновление ключа без downtime |

---

### Зачем нужны два ключа?

Наличие двух ключей позволяет выполнять **безостановочную ротацию (zero-downtime rotation)**.

Типовой процесс:

1. Клиент использует Primary key
2. Вы обновляете Secondary key
3. Клиент переключается на новый Secondary key
4. Обновляете Primary key
5. При необходимости снова переключаете клиента

Таким образом можно регулярно менять ключи без остановки сервиса.

---

### Архитектурное значение

Два ключа позволяют:

- повысить безопасность;
- реализовать безопасную ротацию;
- минимизировать риски утечки;
- избежать простоев при обновлении секретов.

Это особенно важно в production-средах и при внешних интеграциях.

---

### Важно для AZ-204

Если в вопросе говорится о:
- безопасной смене ключей,
- обновлении ключа без простоя,
- наличии двух ключей в подписке,

— правильный ответ связан с использованием Primary и Secondary subscription keys.

### Процесс ротации ключей (Zero-Downtime Rotation)
```
**Шаг 1:** Клиент использует Primary Key (Key A)  

**Шаг 2:** Администратор регенерирует Secondary Key  
(Key B → Key C)

**Шаг 3:** Клиент переключается на новый Secondary Key  
(Key C)

**Шаг 4:** Администратор регенерирует Primary Key  
(Key A → Key D)

**Шаг 5:** Клиент переключается на обновлённый Primary Key  
(Key D)
```

### Что происходит в результате

- В каждый момент времени существует хотя бы один валидный ключ
- Нет перерыва в работе API
- Можно безопасно менять ключи по регламенту безопасности

---

### Архитектурный смысл

Такая схема позволяет:

- регулярно выполнять ротацию секретов;
- минимизировать риски при компрометации ключа;
- соблюдать требования безопасности и комплаенса;
- поддерживать бесперебойную работу интеграций.

---

### Важно для AZ-204

Если в вопросе говорится о:
- смене subscription key без остановки сервиса,
- безопасной ротации ключей,
- наличии двух активных ключей,

— речь идёт о механизме Primary / Secondary key rotation.
---

## Subscription Scopes

Subscriptions can be scoped at three levels:

### 1. **All APIs Scope**

Grants access to **every API** in the APIM instance.

**Use Cases**:
- ✅ Admin/internal testing
- ✅ Service-to-service communication
- ❌ **NOT recommended** for external developers

**Creation**:
```bash
az apim subscription create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --subscription-id all-apis-sub \
  --scope "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ApiManagement/service/{serviceName}" \
  --display-name "All APIs Access"
```

**Security Note**: ⚠️ Use sparingly - provides broad access

### 2. **Single API Scope**

Grants access to **one specific API** only.

**Use Cases**:
- ✅ Limited integration (partner needs only one API)
- ✅ Separate billing per API
- ✅ Granular access control

**Creation**:
```bash
az apim subscription create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --subscription-id users-api-sub \
  --scope "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ApiManagement/service/{serviceName}/apis/users-api" \
  --display-name "Users API Access"
```

**Example**:
```
Subscription: "Partner Integration"
Scope: Users API only
Access:
  ✅ GET /users
  ✅ GET /users/{id}
  ✅ POST /users
  ❌ Orders API (not included)
  ❌ Analytics API (not included)
```

### 3. **Product Scope** (Most Common)

Grants access to **all APIs in a product**.

**Use Cases**:
- ✅ **Recommended approach** for most scenarios
- ✅ Group related APIs together
- ✅ Tiered pricing (Starter, Basic, Premium)
- ✅ Different quotas per product

**Creation**:
```bash
az apim subscription create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --subscription-id starter-sub \
  --scope "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.ApiManagement/service/{serviceName}/products/starter" \
  --display-name "Starter Product Subscription"
```

**Example**:
```
Product: "Starter"
  APIs:
    - Users API
    - Orders API (read-only)
  Quota: 1,000 calls/month
  Rate Limit: 10 calls/minute

Product: "Premium"
  APIs:
    - Users API
    - Orders API (full access)
    - Analytics API
  Quota: 1,000,000 calls/month
  Rate Limit: 1,000 calls/minute
```

---

## Passing Subscription Keys

Subscription keys can be passed in **two ways**:

### 1. **HTTP Header** (Recommended)

**Header Name**: `Ocp-Apim-Subscription-Key`

**Example**:
```bash
curl -X GET https://apim-instance.azure-api.net/api/users \
  -H "Ocp-Apim-Subscription-Key: a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"
```

**JavaScript (fetch)**:
```javascript
fetch('https://apim-instance.azure-api.net/api/users', {
  method: 'GET',
  headers: {
    'Ocp-Apim-Subscription-Key': 'a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6'
  }
})
.then(response => response.json())
.then(data => console.log(data));
```

**C# (HttpClient)**:
```csharp
using var client = new HttpClient();
client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6");

var response = await client.GetAsync("https://apim-instance.azure-api.net/api/users");
var content = await response.Content.ReadAsStringAsync();
```

**Python (requests)**:
```python
import requests

headers = {
    'Ocp-Apim-Subscription-Key': 'a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6'
}

response = requests.get('https://apim-instance.azure-api.net/api/users', headers=headers)
print(response.json())
```

### 2. **Query String**

**Parameter Name**: `subscription-key`

**Example**:
```bash
curl -X GET "https://apim-instance.azure-api.net/api/users?subscription-key=a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6"
```

**⚠️ Security Warning**: Query parameters can be logged in server logs, proxies, and browser history. **Use headers in production**.

**Use Cases for Query String**:
- Quick testing
- Simple demos
- Legacy systems that can't modify headers

---

## Subscription Lifecycle

### 1. **Request Subscription** (Developer)

Developers request subscriptions via:
- Developer portal
- Email to admin
- Automated approval

**Developer Portal Flow**:
```
1. Developer browses products
2. Clicks "Subscribe" on desired product
3. Fills subscription request form
4. Submits request
```

### 2. **Approve/Reject** (Admin)

Admins manage subscription requests:

**Auto-Approval** (configured on product):
```xml
<product>
  <approvalRequired>false</approvalRequired>
</product>
```

**Manual Approval**:
```bash
# List pending subscriptions
az apim subscription list \
  --resource-group rg-apim \
  --service-name apim-instance \
  --query "[?state=='submitted']"

# Approve subscription
az apim subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --state active
```

### 3. **Получение ключей** (Developer)

После одобрения подписки разработчик получает:

- ✅ Primary subscription key
- ✅ Secondary subscription key
- ✅ Детали подписки (scope, квоты, ограничения скорости)

Это означает, что доступ к API официально предоставлен и можно начинать интеграцию.

---

### 4. **Использование API**

Разработчик выполняет вызовы API, передавая subscription key в каждом запросе.

Ключ может передаваться:

- в HTTP-заголовке
- в query-параметре

Проверка ключа выполняется на уровне API Gateway, до передачи запроса в backend.

---

### 5. **Мониторинг использования**

И разработчики, и администраторы могут отслеживать использование API:

- Количество запросов
- Процент ошибок
- Использование квоты
- Срабатывания rate limit

---

### Где смотреть аналитику

**Azure Portal**:  
API Management → Subscriptions → Analytics

---

### Архитектурное значение

Мониторинг подписок позволяет:

- контролировать нагрузку;
- выявлять злоупотребления;
- анализировать потребителей API;
- принимать решения о масштабировании;
- управлять тарифными планами.

---

### Важно для AZ-204

Если в вопросе говорится о:
- контроле использования API по подписке,
- анализе потребления квот,
- мониторинге rate limit,

— следует использовать аналитику в Azure API Management.
### 6. **Rotate Keys**

**Regenerate Primary Key**:
```bash
az apim subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --primary-key $(uuidgen)
```

**Regenerate Secondary Key**:
```bash
az apim subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --secondary-key $(uuidgen)
```

### 7. **Suspend/Cancel**

**Suspend** (temporary):
```bash
az apim subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --state suspended
```

**Cancel** (permanent):
```bash
az apim subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --state cancelled
```

---

## Состояния Subscription

| Состояние | Описание | Доступ к API |
|------------|------------|---------------|
| **submitted** | Ожидает одобрения | ❌ Нет |
| **active** | Одобрена и активна | ✅ Да |
| **suspended** | Временно отключена | ❌ Нет |
| **rejected** | Запрос отклонён администратором | ❌ Нет |
| **cancelled** | Подписка отменена | ❌ Нет |
| **expired** | Истёк срок действия | ❌ Нет |

---

### Что важно понимать

Только состояние **active** позволяет выполнять вызовы API.

Все остальные состояния блокируют доступ на уровне API Gateway — запрос не передаётся в backend.

Это означает:

- backend не получает невалидные запросы;
- безопасность обеспечивается централизованно;
- администратор может управлять доступом без изменения кода сервисов.

---

## Ответ при неверном или отсутствующем ключе

Если subscription key отсутствует или недействителен:

**HTTP Status**: `401 Unauthorized`

Запрос отклоняется на уровне API Management до передачи в backend.

---

### Важно для AZ-204

Если в вопросе говорится о:
- недействительном subscription key,
- приостановленной подписке,
- истёкшей подписке,

— правильный ответ: API вернёт **401 Unauthorized**, и запрос не будет передан в backend.

**Response Headers**:
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: AzureApiManagementKey realm="https://apim-instance.azure-api.net/api",name="Ocp-Apim-Subscription-Key",type="header"
Content-Type: application/json
```

**Response Body**:
```json
{
  "statusCode": 401,
  "message": "Access denied due to invalid subscription key. Make sure to provide a valid key for an active subscription."
}
```

---

## Policy Examples

### Require Subscription Key

```xml
<policies>
  <inbound>
    <!-- Validate subscription key -->
    <check-header name="Ocp-Apim-Subscription-Key" failed-check-httpcode="401" failed-check-error-message="Subscription key is required" />
    <base />
  </inbound>
</policies>
```

### Custom Header Name

```xml
<policies>
  <inbound>
    <!-- Use custom header name -->
    <set-header name="Ocp-Apim-Subscription-Key" exists-action="override">
      <value>@(context.Request.Headers.GetValueOrDefault("X-API-Key"))</value>
    </set-header>
    <base />
  </inbound>
</policies>
```

### Log Subscription Usage

```xml
<policies>
  <inbound>
    <log-to-eventhub logger-id="usage-logger">
      @{
        return new JObject(
          new JProperty("timestamp", DateTime.UtcNow),
          new JProperty("subscriptionKey", context.Subscription?.Key ?? "none"),
          new JProperty("subscriptionName", context.Subscription?.Name ?? "none"),
          new JProperty("userId", context.User?.Id ?? "anonymous"),
          new JProperty("url", context.Request.Url.ToString())
        ).ToString();
      }
    </log-to-eventhub>
    <base />
  </inbound>
</policies>
```

---

## CLI Reference

### Create Subscription

```bash
# Product-scoped subscription
az apim subscription create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --subscription-id my-subscription \
  --scope "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.ApiManagement/service/{service}/products/{product-id}" \
  --display-name "My Subscription"

# API-scoped subscription
az apim subscription create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --subscription-id api-sub \
  --scope "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.ApiManagement/service/{service}/apis/{api-id}" \
  --display-name "API Subscription"

# All APIs subscription
az apim subscription create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --subscription-id all-apis \
  --scope "/subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.ApiManagement/service/{service}" \
  --display-name "All APIs"
```

### List Subscriptions

```bash
# List all subscriptions
az apim subscription list \
  --resource-group rg-apim \
  --service-name apim-instance

# List active subscriptions
az apim subscription list \
  --resource-group rg-apim \
  --service-name apim-instance \
  --query "[?state=='active']"

# List subscriptions for specific product
az apim subscription list \
  --resource-group rg-apim \
  --service-name apim-instance \
  --query "[?contains(scope, 'products/starter')]"
```

### Show Subscription

```bash
az apim subscription show \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id
```

### Update Subscription

```bash
# Suspend subscription
az apim subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --state suspended

# Activate subscription
az apim subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --state active

# Regenerate primary key
az apim subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --primary-key $(uuidgen)

# Regenerate secondary key
az apim subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --secondary-key $(uuidgen)
```

### Delete Subscription

```bash
az apim subscription delete \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id
```

### Get Subscription Keys

```bash
# Get secret (shows both primary and secondary keys)
az apim subscription show \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --query "{primary: primaryKey, secondary: secondaryKey}"
```

---

## Best Practices

### 1. **Use Product Scope for Most Scenarios**

✅ **Do**: Scope subscriptions to products
```bash
az apim subscription create \
  --scope "/products/starter" \
  --display-name "Starter Subscription"
```

❌ **Don't**: Use All APIs scope for external developers

### 2. **Pass Keys in Headers, Not Query Strings**

✅ **Do**: Use `Ocp-Apim-Subscription-Key` header
```bash
curl -H "Ocp-Apim-Subscription-Key: key" https://api.contoso.com/users
```

❌ **Don't**: Use query string in production
```bash
curl "https://api.contoso.com/users?subscription-key=key"  # Insecure
```

### 3. **Регулярно выполняйте ротацию ключей**

✅ **Рекомендуется**: внедрить регламент регулярной смены ключей

```
Регенерировать secondary key

Переключить клиента на secondary key

Проверить корректность работы клиента

Регенерировать primary key

Переключить клиента обратно на primary key
```


Регулярная ротация:

- снижает риск компрометации;
- соответствует требованиям безопасности;
- позволяет избежать простоев.

---

### 4. **Используйте оба ключа (Primary и Secondary)**

✅ **Рекомендуется**: предоставлять клиентам оба ключа

- **Primary** — основной рабочий ключ
- **Secondary** — для ротации и резервного использования

Наличие двух ключей — это встроенный механизм безопасного обновления секретов без остановки API.

---

### 5. **Мониторьте использование подписок**

✅ **Рекомендуется**: включить аналитику и оповещения

- Отслеживать потребление квоты
- Анализировать аномальные паттерны использования
- Настраивать алерты при превышении rate limit

Это позволяет:

- предотвращать злоупотребления;
- выявлять утечки ключей;
- прогнозировать нагрузку;
- управлять тарифными планами.

---

### Важно для AZ-204

Если в задаче говорится о:
- безопасной эксплуатации API,
- управлении доступом по подписке,
- мониторинге использования,

— правильные действия включают ротацию ключей, использование Primary/Secondary и включение аналитики.
### 6. **Implement Approval Workflow**

✅ **Do**: Require approval for sensitive APIs
```xml
<product>
  <approvalRequired>true</approvalRequired>
</product>
```

### 7. **Set Expiration Dates**

✅ **Do**: Configure subscription expiration
```bash
az apim subscription create \
  --expires-on "2024-12-31T23:59:59Z"
```

---

## Common Scenarios

### Scenario 1: Tiered API Access

```
Free Tier Product:
  - Subscription: Required
  - Rate Limit: 10 calls/minute
  - Quota: 1,000 calls/month
  - APIs: Users API (read-only)

Standard Tier Product:
  - Subscription: Required
  - Rate Limit: 100 calls/minute
  - Quota: 100,000 calls/month
  - APIs: Users API, Orders API

Premium Tier Product:
  - Subscription: Required (approval needed)
  - Rate Limit: 1,000 calls/minute
  - Quota: Unlimited
  - APIs: All APIs (full access)
```

### Scenario 2: Partner Integration

```
Partner: Contoso
  - Subscription: API-scoped (Orders API only)
  - Keys: Primary and secondary
  - Rate Limit: Custom (500 calls/minute)
  - Monitoring: Dedicated dashboard
```

### Scenario 3: Internal Services

```
Service: Internal Microservice
  - Subscription: All APIs scope
  - Keys: Managed in Key Vault
  - Rate Limit: High (10,000 calls/minute)
  - Approval: Auto-approved
```

---

## Советы к экзамену

### Ключевые концепции для AZ-204

1. **Subscription key**  
   Уникальная 32-символьная строка, используемая для аутентификации запросов.

2. **Два ключа на подписку**  
   Primary и Secondary — используются для безопасной ротации.

3. **Scope подписки**
    - All APIs
    - Single API
    - Product (наиболее распространённый вариант)

4. **Имя HTTP-заголовка**  
   `Ocp-Apim-Subscription-Key`

5. **Query-параметр**  
   `subscription-key` (менее безопасный способ передачи)

6. **Ответ 401**  
   Возвращается, если ключ отсутствует или недействителен.

7. **Состояния подписки**  
   submitted, active, suspended, cancelled, rejected, expired

8. **Product scope**  
   Рекомендуется для большинства сценариев, особенно при разграничении тарифов.

9. **Ротация ключей**  
   Использовать Secondary key во время обновления Primary для zero downtime.

10. **Developer Portal**  
    Поддерживает self-service управление подписками.

---

## Частые экзаменационные сценарии

**Сценарий 1**:  
"Защитить API простым механизмом аутентификации"  
→ **Ответ**: Требовать subscription key (на уровне Product)

---

**Сценарий 2**:  
"Сменить API-ключи без простоя"  
→ **Ответ**: Использовать Primary/Secondary ключи и регенерировать их поочерёдно

---

**Сценарий 3**:  
"Предоставить доступ только к одному конкретному API"  
→ **Ответ**: Создать подписку со scope на конкретный API

---

**Сценарий 4**:  
"Передать ключ аутентификации в API"  
→ **Ответ**: Использовать заголовок `Ocp-Apim-Subscription-Key`

---

**Сценарий 5**:  
"Реализовать разные уровни доступа (Free, Standard, Premium)"  
→ **Ответ**: Создать продукты с разными API и квотами, подписка создаётся на каждый продукт

---

### Финальный акцент для AZ-204

- Subscription key — это простой, но эффективный механизм защиты.
- Product scope чаще всего используется в реальных сценариях.
- 401 означает проблему с ключом.
- Ротация ключей — стандартная практика безопасности.
- Разные тарифы реализуются через продукты и подписки, а не через отдельные API.---

## Learn More

- [Subscriptions in API Management](https://docs.microsoft.com/azure/api-management/api-management-subscriptions)
- [How to Secure APIs Using Subscription Keys](https://docs.microsoft.com/azure/api-management/api-management-howto-create-subscriptions)
- [Products Documentation](https://docs.microsoft.com/azure/api-management/api-management-howto-add-products)
- [Developer Portal](https://docs.microsoft.com/azure/api-management/api-management-howto-developer-portal)
