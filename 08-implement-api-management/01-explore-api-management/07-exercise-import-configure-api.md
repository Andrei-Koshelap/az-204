# Практическое задание: Импорт и настройка API

## Обзор

В этом практическом упражнении вы:

1. Создадите экземпляр Azure API Management
2. Импортируете API из спецификации OpenAPI
3. Настроите параметры backend
4. Добавите продукт и подписку
5. Примените политики (rate limiting, трансформация)
6. Протестируете операции API
7. Очистите ресурсы

**Продолжительность**: ~30 минут

---

## Предварительные требования

Перед началом убедитесь, что у вас есть:

- ✅ Активная подписка Azure
- ✅ Установленный Azure CLI (или используйте Azure Cloud Shell)
- ✅ Права Contributor для создания ресурсов
- ✅ Базовые знания REST API

---

## Цель упражнения

После выполнения задания вы сможете:

- развернуть API Management;
- импортировать OpenAPI-спецификацию;
- настроить backend и политики;
- управлять доступом через продукты и подписки;
- протестировать API через встроенную консоль.

---

## Экзаменационный фокус (AZ-204)

Практика закрепляет темы:

- Импорт OpenAPI в APIM
- Настройка backend
- Product и Subscription
- Policies (rate-limit, transformation)
- Тестирование через Developer Portal

Если в экзамене описывается сценарий с импортом OpenAPI и последующей настройкой доступа — это именно тот процесс, который вы проходите в этом упражнении.

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Azure                              │
│                                                     │
│  ┌────────────────────────────────────────────┐   │
│  │  API Management Instance                   │   │
│  │  • Gateway: apim-demo.azure-api.net        │   │
│  │  • Product: Starter                        │   │
│  │  • API: Conference API                     │   │
│  │  • Policies: Rate limiting, headers        │   │
│  └──────────────┬─────────────────────────────┘   │
│                 │                                   │
│                 ▼                                   │
│  ┌────────────────────────────────────────────┐   │
│  │  Backend API                               │   │
│  │  conferenceapi.azurewebsites.net           │   │
│  │  (OpenAPI/Swagger specification)           │   │
│  └────────────────────────────────────────────┘   │
│                                                     │
└─────────────────────────────────────────────────────┘
```

---

## Task 1: Create API Management Instance

### Step 1: Set Variables

```bash
# Define variables
RESOURCE_GROUP="rg-apim-lab"
LOCATION="eastus"
APIM_NAME="apim-demo-$RANDOM"  # Must be globally unique
PUBLISHER_EMAIL="admin@contoso.com"  # Replace with your email
PUBLISHER_NAME="Contoso"
```

### Step 2: Create Resource Group

```bash
# Create resource group
az group create \
  --name $RESOURCE_GROUP \
  --location $LOCATION
```

**Expected Output**:
```json
{
  "id": "/subscriptions/.../resourceGroups/rg-apim-lab",
  "location": "eastus",
  "name": "rg-apim-lab",
  "properties": {
    "provisioningState": "Succeeded"
  }
}
```

### Step 3: Create API Management Instance

```bash
# Create APIM instance (takes 30-40 minutes)
az apim create \
  --name $APIM_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --publisher-email $PUBLISHER_EMAIL \
  --publisher-name "$PUBLISHER_NAME" \
  --sku-name Consumption

# Note: Use Consumption tier for faster provisioning (5-10 minutes)
# For production, use Developer, Basic, Standard, or Premium
```

**⏱️ Примечание**: Развёртывание API Management занимает время:

- **Consumption tier**: 5–10 минут
- **Developer tier**: 30–40 минут
- **Standard / Premium tiers**: 40–60 минут

---

### Почему это важно

Экземпляр APIM — это не просто логическая настройка, а полноценный управляемый сервис с сетевой инфраструктурой, масштабированием и шлюзом.

Особенно долго разворачиваются:

- Developer — из-за выделенной инфраструктуры
- Standard / Premium — из-за поддержки масштабирования, multi-region и VNet

---

### Практический совет

Во время ожидания можно:

- подготовить OpenAPI-файл;
- продумать структуру продукта и подписок;
- написать политики заранее;
- изучить настройки backend.

---

### Важно для AZ-204

На экзамене могут упоминаться разные tier’ы APIM.  
Помните:

- Consumption — быстрее и дешевле
- Premium — поддерживает multi-region и VNet
- Developer — не предназначен для production

**Check Provisioning Status**:
```bash
az apim show \
  --name $APIM_NAME \
  --resource-group $RESOURCE_GROUP \
  --query "{name:name, state:provisioningState, gatewayUrl:gatewayUrl}"
```

**Expected Output**:
```json
{
  "gatewayUrl": "https://apim-demo-12345.azure-api.net",
  "name": "apim-demo-12345",
  "state": "Succeeded"
}
```

**Save Gateway URL**:
```bash
GATEWAY_URL=$(az apim show \
  --name $APIM_NAME \
  --resource-group $RESOURCE_GROUP \
  --query gatewayUrl -o tsv)

echo "Gateway URL: $GATEWAY_URL"
```

---

## Task 2: Import API from OpenAPI Specification

### Step 1: Import Conference API

Microsoft provides a demo Conference API for testing:

**API Details**:
- **Name**: Conference API
- **Base URL**: `https://conferenceapi.azurewebsites.net`
- **OpenAPI Spec**: `https://conferenceapi.azurewebsites.net?format=json`
- **Operations**: Get Sessions, Get Session by ID, Get Topics, Get Speakers

```bash
# Import API from OpenAPI specification
az apim api import \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --path /conference \
  --api-id conference-api \
  --display-name "Conference API" \
  --service-url "https://conferenceapi.azurewebsites.net" \
  --specification-url "https://conferenceapi.azurewebsites.net?format=json" \
  --specification-format OpenApiJson \
  --protocols https
```

**Expected Output**:
```json
{
  "apiRevision": "1",
  "displayName": "Conference API",
  "id": "/subscriptions/.../apis/conference-api",
  "name": "conference-api",
  "path": "conference",
  "protocols": ["https"],
  "serviceUrl": "https://conferenceapi.azurewebsites.net"
}
```

### Step 2: Verify Import

```bash
# List APIs
az apim api list \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --query "[].{name:name, displayName:displayName, path:path}" -o table
```

**Expected Output**:
```
Name             DisplayName       Path
---------------  ----------------  -----------
conference-api   Conference API    conference
```

### Step 3: List Operations

```bash
# List operations in Conference API
az apim api operation list \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --api-id conference-api \
  --query "[].{id:name, displayName:displayName, method:method, urlTemplate:urlTemplate}" -o table
```

**Expected Output**:
```
Id                    DisplayName         Method    UrlTemplate
--------------------  ------------------  --------  -------------------
GetSessions           GetSessions         GET       /sessions
GetSession            GetSession          GET       /session/{id}
GetTopics             GetTopics           GET       /topics
GetSpeakers           GetSpeakers         GET       /speakers
```

---

## Task 3: Create Product and Subscription

### Step 1: Create Product

```bash
# Create "Starter" product
az apim product create \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --product-id starter \
  --product-name "Starter" \
  --description "Starter tier for developers" \
  --subscription-required true \
  --approval-required false \
  --state published
```

**Expected Output**:
```json
{
  "approvalRequired": false,
  "displayName": "Starter",
  "id": "/subscriptions/.../products/starter",
  "name": "starter",
  "state": "published",
  "subscriptionRequired": true
}
```

### Step 2: Add API to Product

```bash
# Associate Conference API with Starter product
az apim product api add \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --product-id starter \
  --api-id conference-api
```

### Step 3: Create Subscription

```bash
# Create subscription to Starter product
az apim subscription create \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --subscription-id starter-subscription \
  --scope /products/starter \
  --display-name "Starter Subscription"
```

### Step 4: Get Subscription Key

```bash
# Get subscription keys
SUBSCRIPTION_KEY=$(az apim subscription show \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --subscription-id starter-subscription \
  --query primaryKey -o tsv)

echo "Subscription Key: $SUBSCRIPTION_KEY"
```

**Save this key** - you'll need it for testing!

---

## Task 4: Configure Backend Settings

### Step 1: Update Backend Service URL

```bash
# Verify backend URL is correct
az apim api show \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --api-id conference-api \
  --query serviceUrl
```

### Step 2: Enable Subscription Requirement

```bash
# Ensure subscription is required for this API
az apim api update \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --api-id conference-api \
  --subscription-required true
```

---

## Task 5: Apply Policies

### Step 1: Create Rate Limiting Policy

Create a policy file named `policy.xml`:

```xml
<policies>
  <inbound>
    <!-- Rate limit: 10 calls per minute -->
    <rate-limit calls="10" renewal-period="60" />
    
    <!-- Add custom headers -->
    <set-header name="X-API-Version" exists-action="override">
      <value>1.0</value>
    </set-header>
    <set-header name="X-Powered-By" exists-action="override">
      <value>Azure API Management</value>
    </set-header>
    
    <!-- Log request -->
    <trace source="api-request">
      @{
        return $"Request: {context.Request.Method} {context.Request.Url}";
      }
    </trace>
    
    <base />
  </inbound>
  <backend>
    <forward-request timeout="30" />
  </backend>
  <outbound>
    <!-- Remove backend headers -->
    <set-header name="X-AspNet-Version" exists-action="delete" />
    <set-header name="X-Powered-By" exists-action="delete" />
    
    <!-- Add response time header -->
    <set-header name="X-Response-Time-ms" exists-action="override">
      <value>@(context.Elapsed.TotalMilliseconds.ToString())</value>
    </set-header>
    
    <base />
  </outbound>
  <on-error>
    <!-- Custom error response -->
    <return-response>
      <set-status code="500" reason="Internal Server Error" />
      <set-header name="Content-Type" exists-action="override">
        <value>application/json</value>
      </set-header>
      <set-body>@{
        return new JObject(
          new JProperty("error", true),
          new JProperty("message", context.LastError.Message),
          new JProperty("timestamp", DateTime.UtcNow)
        ).ToString();
      }</set-body>
    </return-response>
  </on-error>
</policies>
```

### Step 2: Apply Policy to API

```bash
# Apply policy to Conference API
az apim api policy create \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --api-id conference-api \
  --xml-policy @policy.xml
```

**Expected Output**:
```
Policy applied successfully to conference-api
```

### Step 3: Verify Policy

```bash
# Show current policy
az apim api policy show \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --api-id conference-api
```

---

## Task 6: Test the API

### Test 1: Call API Without Subscription Key

```bash
# This should fail with 401 Unauthorized
curl -i $GATEWAY_URL/conference/sessions
```

**Expected Response**:
```
HTTP/1.1 401 Unauthorized
WWW-Authenticate: AzureApiManagementKey realm="...",name="Ocp-Apim-Subscription-Key",type="header"

{
  "statusCode": 401,
  "message": "Access denied due to invalid subscription key..."
}
```

### Test 2: Call API With Subscription Key (Header)

```bash
# This should succeed
curl -i $GATEWAY_URL/conference/sessions \
  -H "Ocp-Apim-Subscription-Key: $SUBSCRIPTION_KEY"
```

**Expected Response**:
```
HTTP/1.1 200 OK
X-API-Version: 1.0
X-Powered-By: Azure API Management
X-Response-Time-ms: 145

[
  {
    "id": 100,
    "title": "Keynote",
    "description": "...",
    "startsAt": "2024-01-01T09:00:00Z",
    "endsAt": "2024-01-01T10:00:00Z"
  },
  ...
]
```

### Test 3: Call API With Subscription Key (Query String)

```bash
# Alternative method (less secure)
curl -i "$GATEWAY_URL/conference/sessions?subscription-key=$SUBSCRIPTION_KEY"
```

### Test 4: Test Rate Limiting

```bash
# Make 15 requests quickly (limit is 10 per minute)
for i in {1..15}; do
  echo "Request $i:"
  curl -s -o /dev/null -w "%{http_code}\n" \
    $GATEWAY_URL/conference/sessions \
    -H "Ocp-Apim-Subscription-Key: $SUBSCRIPTION_KEY"
done
```

**Expected Output**:
```
Request 1: 200
Request 2: 200
...
Request 10: 200
Request 11: 429  ← Rate limit exceeded
Request 12: 429
...
```

**429 Response**:
```json
{
  "statusCode": 429,
  "message": "Rate limit is exceeded. Try again in X seconds."
}
```

### Test 5: Get Session by ID

```bash
# Get specific session
curl $GATEWAY_URL/conference/session/100 \
  -H "Ocp-Apim-Subscription-Key: $SUBSCRIPTION_KEY" \
  -H "Accept: application/json" | jq
```

### Test 6: Test Other Operations

```bash
# Get topics
curl $GATEWAY_URL/conference/topics \
  -H "Ocp-Apim-Subscription-Key: $SUBSCRIPTION_KEY" | jq

# Get speakers
curl $GATEWAY_URL/conference/speakers \
  -H "Ocp-Apim-Subscription-Key: $SUBSCRIPTION_KEY" | jq
```

---

## Task 7: Monitor API Usage

### View Analytics in Azure Portal

1. Navigate to Azure Portal
2. Go to your API Management instance
3. Select **Analytics** from left menu
4. View:
   - Total requests
   - Response times
   - Error rates
   - Top operations
   - Geographic distribution

### Check Logs with Azure CLI

```bash
# Get API metrics
az monitor metrics list \
  --resource $(az apim show \
    --name $APIM_NAME \
    --resource-group $RESOURCE_GROUP \
    --query id -o tsv) \
  --metric "Requests" \
  --start-time $(date -u -d '1 hour ago' +%Y-%m-%dT%H:%M:%SZ) \
  --end-time $(date -u +%Y-%m-%dT%H:%M:%SZ)
```

---

## Task 8: Advanced Configuration (Optional)

### Add Caching Policy

```xml
<policies>
  <inbound>
    <!-- Cache lookup -->
    <cache-lookup vary-by-developer="false" vary-by-developer-groups="false" />
    <base />
  </inbound>
  <outbound>
    <!-- Cache for 10 minutes -->
    <cache-store duration="600" />
    <base />
  </outbound>
</policies>
```

Apply:
```bash
az apim api operation policy create \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --api-id conference-api \
  --operation-id GetSessions \
  --xml-policy @cache-policy.xml
```

### Add CORS Policy

```xml
<policies>
  <inbound>
    <cors allow-credentials="true">
      <allowed-origins>
        <origin>https://www.contoso.com</origin>
        <origin>https://app.contoso.com</origin>
      </allowed-origins>
      <allowed-methods>
        <method>GET</method>
        <method>POST</method>
      </allowed-methods>
      <allowed-headers>
        <header>Content-Type</header>
        <header>Authorization</header>
      </allowed-headers>
    </cors>
    <base />
  </inbound>
</policies>
```

---

## Task 9: Test in Developer Portal

### Step 1: Access Developer Portal

```bash
# Get developer portal URL
az apim show \
  --name $APIM_NAME \
  --resource-group $RESOURCE_GROUP \
  --query developerPortalUrl -o tsv
```

### Step 2: Sign Up (if not already)

1. Open developer portal URL
2. Click **Sign up**
3. Create account
4. Verify email (if required)

### Step 3: Subscribe to Product

1. Login to developer portal
2. Navigate to **Products**
3. Select **Starter** product
4. Click **Subscribe**
5. Confirm subscription

### Step 4: Test API in Console

1. Navigate to **APIs** → **Conference API**
2. Select **GetSessions** operation
3. Click **Try it**
4. Select your subscription
5. Click **Send**
6. View response

---

## Task 10: Cleanup Resources

### Delete Resource Group

```bash
# Delete everything (API Management + all resources)
az group delete \
  --name $RESOURCE_GROUP \
  --yes \
  --no-wait
```

**⚠️ Warning**: This deletes ALL resources in the resource group.

### Verify Deletion

```bash
# Check deletion status
az group show \
  --name $RESOURCE_GROUP \
  --query "{name:name, state:properties.provisioningState}"
```

---

## Troubleshooting

### Issue 1: APIM Creation Takes Too Long

**Solution**:
- Use **Consumption tier** for faster provisioning (5-10 minutes)
- For Developer/Standard tiers, expect 30-60 minutes

### Issue 2: 401 Unauthorized Error

**Causes**:
- ✅ Missing subscription key
- ✅ Incorrect key
- ✅ Subscription not associated with product
- ✅ Subscription suspended/cancelled

**Solutions**:
```bash
# Verify subscription
az apim subscription show \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --subscription-id starter-subscription

# Check subscription state (should be "active")
```

### Issue 3: 404 Not Found

**Causes**:
- ✅ Wrong URL path
- ✅ API not published
- ✅ Backend service URL incorrect

**Solutions**:
```bash
# Verify API path
az apim api show \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --api-id conference-api \
  --query "{path:path, serviceUrl:serviceUrl}"

# Correct URL format:
# https://<apim-name>.azure-api.net/<api-path>/<operation-path>
```

### Issue 4: 429 Rate Limit Exceeded

**Expected Behavior**: Policy working correctly!

**Solution**: Wait 60 seconds for renewal period

### Issue 5: Backend Not Responding

**Symptoms**: 500 or 504 errors

**Solutions**:
```bash
# Test backend directly
curl https://conferenceapi.azurewebsites.net/sessions

# Check backend timeout in policy
<forward-request timeout="60" />
```

---

## Итоги

### Что вы изучили

1. ✅ Создание экземпляра APIM с помощью Azure CLI
2. ✅ Импорт API из OpenAPI-спецификации
3. ✅ Создание продуктов и подписок
4. ✅ Применение политик (rate limiting, заголовки, кэширование)
5. ✅ Тестирование API с использованием subscription keys
6. ✅ Мониторинг использования и аналитики
7. ✅ Использование Developer Portal для self-service

---

### Применённые best practices

- ✅ Использован **Consumption tier** для быстрого лабораторного развёртывания
- ✅ Настроен **rate limiting** для предотвращения злоупотреблений
- ✅ Включена обязательная проверка **subscription keys**
- ✅ Удалены **чувствительные заголовки** в outbound policy
- ✅ Настроено логирование **запросов**
- ✅ Реализована обработка ошибок с кастомными ответами

---

### Рекомендации для production

Для боевых развёртываний:

- 🎯 Использовать **Standard или Premium tier** (SLA, multi-region)
- 🎯 Включить интеграцию с **Application Insights**
- 🎯 Настроить **custom domains** с SSL
- 🎯 Реализовать **IP filtering** и **JWT validation**
- 🎯 Настроить **VNet integration** (Premium tier)
- 🎯 Настроить **backup и restore**
- 🎯 Использовать **Named Values** для конфигурации
- 🎯 Применять **version sets** для управления версиями API

---

### Архитектурный вывод

Azure API Management — это не просто прокси, а полноценный слой управления API:

- безопасность,
- контроль доступа,
- мониторинг,
- масштабирование,
- централизованные политики.

---

### Важно для AZ-204

На экзамене важно понимать:

- когда использовать Consumption vs Premium;
- как импортировать OpenAPI;
- где применять политики;
- как управлять доступом через продукты и подписки;
- какие функции обязательны для production-сценариев.

Экзамен проверяет понимание архитектуры и сценариев применения, а не только знание интерфейса портала.
---

## Additional Resources

### Documentation

- [Azure API Management Documentation](https://docs.microsoft.com/azure/api-management/)
- [Import OpenAPI Specification](https://docs.microsoft.com/azure/api-management/import-api-from-oas)
- [Policy Reference](https://docs.microsoft.com/azure/api-management/api-management-policies)
- [Developer Portal](https://docs.microsoft.com/azure/api-management/api-management-howto-developer-portal)

### Sample APIs for Testing

- **Conference API**: `https://conferenceapi.azurewebsites.net`
- **JSONPlaceholder**: `https://jsonplaceholder.typicode.com`
- **ReqRes**: `https://reqres.in/api`
- **HTTPBin**: `https://httpbin.org`

### Azure CLI Quick Reference

```bash
# Create APIM
az apim create --name <name> --resource-group <rg> --publisher-email <email> --publisher-name <name> --sku-name Consumption

# Import API
az apim api import --api-id <id> --path <path> --specification-url <url> --specification-format OpenApiJson

# Create product
az apim product create --product-id <id> --product-name <name> --subscription-required true --state published

# Add API to product
az apim product api add --product-id <id> --api-id <api-id>

# Create subscription
az apim subscription create --subscription-id <id> --scope /products/<product-id>

# Apply policy
az apim api policy create --api-id <id> --xml-policy @policy.xml

# Get gateway URL
az apim show --name <name> --query gatewayUrl -o tsv

# Delete resource group
az group delete --name <rg> --yes --no-wait
```

---

## Congratulations! 🎉

Вы успешно:

- ✅ Создали экземпляр Azure API Management
- ✅ Импортировали и настроили API
- ✅ Применили механизмы безопасности и политики
- ✅ Протестировали операции API
- ✅ Настроили мониторинг использования

Теперь у вас есть практический опыт работы с Azure API Management.

---

## Следующие шаги

1. **Изучить дополнительные политики**  
   Попробуйте реализовать JWT validation, IP filtering, трансформацию запросов.

2. **Multi-region deployment**  
   Настройте Premium tier с развёртыванием в нескольких регионах.

3. **Custom domains**  
   Подключите пользовательский домен и SSL-сертификат.

4. **OAuth 2.0**  
   Интегрируйте API с Azure AD для полноценной аутентификации.

5. **Self-hosted gateway**  
   Разверните gateway on-premises или в Kubernetes.

6. **Монетизация**  
   Настройте тарифные планы с квотами и ограничениями.

7. **Application Insights**  
   Включите расширенный мониторинг и аналитику.

---

## Советы к экзамену

### Ключевые концепции

1. **Provisioning APIM**
   - Consumption — самый быстрый (5–10 минут)
   - Developer — 30–40 минут

2. **Импорт API**  
   Используется OpenAPI/Swagger, команда `az apim api import`.

3. **Products**  
   Контейнер для API. Для защищённых продуктов требуется подписка.

4. **Subscriptions**  
   Предоставляют Primary и Secondary ключи для доступа к API.

5. **Policies**  
   XML-основанные правила, применяемые на уровнях: Global / Product / API / Operation.

6. **Rate limiting**  
   `<rate-limit calls="10" renewal-period="60" />`  
   При превышении возвращается 429.

7. **Subscription key**  
   Передавать в заголовке `Ocp-Apim-Subscription-Key` (рекомендуемый способ).

8. **401 vs 429**
   - 401 → отсутствует или неверный ключ
   - 429 → превышен лимит запросов

9. **Developer Portal**  
   Self-service инструмент для подписки и тестирования API.

10. **Очистка ресурсов**  
    Удаление resource group удаляет все связанные ресурсы.

---

### Финальный акцент для AZ-204

На экзамене проверяется:

- понимание tier’ов APIM;
- знание механизма продуктов и подписок;
- различие между 401 и 429;
- понимание, где применять политики;
- умение выбрать правильный механизм защиты (subscription, JWT, mTLS).

Главное — понимать архитектуру и сценарии применения, а не только команды CLI.
---

## Learn More

- [Azure API Management Overview](https://docs.microsoft.com/azure/api-management/api-management-key-concepts)
- [API Import Tutorial](https://docs.microsoft.com/azure/api-management/import-and-publish)
- [Policy Samples](https://docs.microsoft.com/azure/api-management/policy-samples)
- [AZ-204 Exam Guide](https://docs.microsoft.com/learn/certifications/exams/az-204)
