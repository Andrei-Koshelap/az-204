# API Gateways

## Что такое API Gateway?

**API Gateway** — это централизованная точка входа, расположенная между клиентами API и backend-сервисами. Он работает как reverse proxy: маршрутизирует запросы, применяет политики безопасности, может агрегировать ответы от нескольких сервисов и возвращать единый результат клиенту.

**Ключевая функция**: отделить (decouple) потребителей API от деталей реализации backend-сервисов.

Иными словами — клиент больше не знает, где именно живёт сервис, на каком порту он работает и как разбита микросервисная архитектура. Всё взаимодействие идёт через единую точку входа.

---

## Проблемы при отсутствии API Gateway

### 1. **Tight Coupling (жёсткая связность)**

**Проблема**: Клиенты напрямую привязаны к URL backend-сервисов.

Это означает, что:
- при изменении адреса сервиса нужно обновлять клиентов;
- при рефакторинге архитектуры (например, разделении сервиса на два) потребуется изменение клиентского кода;
- усложняется поддержка версионирования API.

Дополнительно:
- мобильные и web-клиенты начинают зависеть от внутренней структуры микросервисов;
- нарушается принцип изоляции доменов;
- увеличивается технический долг.

---

## Что обычно делает API Gateway (дополнение)

В реальных системах API Gateway также выполняет:

- 🔐 Аутентификацию и авторизацию (JWT, OAuth2)
- 🚦 Rate limiting (ограничение количества запросов)
- 📦 Агрегацию данных (Backend for Frontend паттерн)
- 🔁 Retry и circuit breaker
- 📊 Логирование и мониторинг
- 🔀 Маршрутизацию по версиям API
- 🌍 TLS termination

В контексте AZ-204 важно понимать, что в Azure эту роль часто выполняют:
- Azure API Management
- Azure Application Gateway
- Azure Front Door (в более сложных сценариях)


|                            | Application Gateway | APIM Gateway           |
| -------------------------- | ------------------- | ---------------------- |
| Уровень                    | Инфраструктура      | API management         |
| WAF                        | ✅                   | ❌ (нужен WAF отдельно) |
| Load balancing             | ✅                   | Нет как основная цель  |
| API keys                   | ❌                   | ✅                      |
| Rate limit                 | ❌                   | ✅                      |
| Transform request/response | ❌                   | ✅                      |
| Developer portal           | ❌                   | ✅                      |

---

## Архитектурный смысл

API Gateway особенно важен в микросервисной архитектуре:

Без него:
```
Client → Service A
Client → Service B
Client → Service C

С ним:

Client → API Gateway → Services

Mobile App ──→ https://users.internal.com/api/users
          ──→ https://orders.internal.com/api/orders
          ──→ https://payments.internal.com/api/payments
```


Это упрощает:
- эволюцию архитектуры,
- масштабирование,
- безопасность,
- управление трафиком.

---

## Ключевая мысль для экзамена AZ-204

Если в вопросе говорится о:
- централизованном управлении API,
- политике безопасности,
- трансформации запросов,
- ограничении скорости,
- версии API,
- защите backend от прямого доступа,

— скорее всего правильным ответом будет API Gateway или Azure API Management.


**Issues**:
- Client must know multiple service endpoints
- URL changes require client updates
- Cannot reorganize backend without breaking clients
- Service discovery complexity

### 2. **Complex Client Code**

**Problem**: Each client must implement cross-cutting concerns

```javascript
// Every client must implement:
- Authentication (OAuth tokens, API keys)
- Rate limiting (track own usage)
- Retry logic (handle failures)
- Circuit breakers (detect outages)
- Logging and monitoring
- Error handling
- Request/response formatting
- SSL certificate validation
```

**Issues**:
- Code duplication across clients
- Inconsistent implementations
- Difficult to update policies
- Different behavior per platform

### 3. **Security Risks**

**Problem**: Backend services exposed directly to internet

```
Internet ──→ Backend Services (public IPs)
              - Authentication logic in each service
              - Multiple attack surfaces
              - Difficult to apply consistent security
              - Hard to audit access
```

**Проблемы**:

- Каждый сервис должен самостоятельно реализовывать аутентификацию
- Несколько точек TLS-терминации
- Несогласованная авторизация
- Сложно реализовать rate limiting
- Отсутствие централизованного логирования
- Трудно обнаруживать атаки

### Пояснение

При отсутствии API Gateway каждый микросервис вынужден самостоятельно реализовывать механизмы безопасности и кросс-сервисные политики. Это приводит к дублированию логики, разной реализации авторизации, усложнению аудита и повышенному риску уязвимостей.

Такой подход нарушает принцип разделения ответственности: безопасность, контроль трафика и мониторинг должны быть вынесены в отдельный инфраструктурный слой.

---

### 4. **Chatty Communication (избыточное сетевое взаимодействие)**

**Проблема**: Для получения всех необходимых данных клиенту требуется выполнять несколько последовательных сетевых запросов.

### Почему это проблема

- Увеличивается задержка (latency)
- Возрастает нагрузка на сеть
- Усложняется клиентская логика
- Снижается производительность мобильных и распределённых приложений

### Как помогает API Gateway

API Gateway позволяет агрегировать данные из нескольких сервисов и возвращать единый ответ клиенту. Это уменьшает количество сетевых вызовов и упрощает архитектуру взаимодействия.

---

### Важно для AZ-204

Если в вопросе акцент делается на:
- сокращении количества сетевых вызовов,
- агрегировании данных,
- оптимизации взаимодействия клиента с микросервисами,

— правильным решением, как правило, будет использование API Gateway или подхода Backend for Frontend.

```
Mobile App:
  GET /users/123         → Backend 1
  GET /orders?user=123   → Backend 2
  GET /profile/123       → Backend 3
  GET /preferences/123   → Backend 4
  
Total: 4 separate HTTP requests
```

**Issues**:
- High latency (especially on mobile)
- Increased bandwidth usage
- Complex error handling
- Poor user experience

### 5. **Difficult Protocol Translation**

**Problem**: Clients must speak different protocols

```
Client needs to handle:
  - REST/JSON for Service A
  - SOAP/XML for Service B
  - gRPC for Service C
  - GraphQL for Service D
```

**Issues**:
- Complex client code
- Multiple libraries required
- Inconsistent data formats
- Hard to maintain

---

## Benefits of API Gateway

### ✅ Decoupling

**Benefit**: Clients only know the gateway URL

```
Before:
Mobile App ──→ Service A (URL 1)
          ──→ Service B (URL 2)
          ──→ Service C (URL 3)

After:
Mobile App ──→ API Gateway ──→ Service A
                           ──→ Service B
                           ──→ Service C
```

**Преимущества**:

- Сервисы можно реорганизовывать без изменений на стороне клиента
- Легко добавлять или удалять backend-сервисы
- Gateway берёт на себя service discovery
- Backend-URL могут оставаться приватными

### Пояснение

API Gateway изолирует клиентов от внутренней структуры системы. Это позволяет:

- менять топологию микросервисов без влияния на потребителей API;
- скрывать внутренние адреса и порты сервисов;
- централизованно управлять маршрутизацией;
- гибко масштабировать backend-компоненты.

Дополнительно это упрощает DevOps-процессы: деплой и масштабирование сервисов становятся прозрачными для клиентов.

---

### ✅ Упрощённый клиентский код

**Преимущество**: Кросс-сервисные задачи (cross-cutting concerns) обрабатываются на уровне gateway.

К таким задачам относятся:

- Аутентификация
- Авторизация
- Rate limiting
- Логирование
- Мониторинг
- Трансформация запросов и ответов
- Версионирование API

В результате клиентская часть:

- содержит меньше инфраструктурной логики;
- не занимается обработкой токенов или политик доступа;
- взаимодействует с единым стабильным endpoint.

---

### Важно для AZ-204

Если в вопросе упоминается:
- централизованная обработка политик,
- упрощение клиентов,
- скрытие внутренней архитектуры,
- управление доступом к backend-сервисам,

— правильным выбором будет API Gateway или Azure API Management.

```
Client only needs to:
  1. Call gateway URL
  2. Include API key/token
  3. Handle response

Gateway handles:
  - Authentication
  - Authorization
  - Rate limiting
  - Retry logic
  - Logging
  - Monitoring
  - Caching
  - Transformation
```

### ✅ SSL Termination

**Benefit**: Single TLS termination point

```
Client ──HTTPS──> Gateway ──HTTP──> Backend Services
        (encrypted)       (internal network)
```

**Преимущества**:

- Снижается нагрузка SSL/TLS на backend-сервисы
- Централизованное управление сертификатами
- Backend-сервисам не требуется собственная настройка сертификатов
- Упрощается процесс продления сертификатов

### Пояснение

API Gateway может выполнять TLS termination — расшифровывать HTTPS-трафик и передавать запросы во внутреннюю сеть по защищённому или внутреннему каналу.

Это даёт следующие преимущества:

- Сервисы не тратят ресурсы на криптографические операции
- Управление сертификатами сосредоточено в одном месте
- Уменьшается риск ошибок конфигурации
- Процесс обновления и продления сертификатов становится проще

В облачных средах это особенно важно при большом количестве микросервисов.

---

### ✅ Аутентификация и авторизация

**Преимущество**: Централизованное применение политик безопасности.

API Gateway может:

- Проверять JWT-токены
- Интегрироваться с OAuth2 / OpenID Connect
- Проверять роли и claims
- Применять политики доступа
- Блокировать неавторизованные запросы до попадания в backend

Это обеспечивает:

- единые правила безопасности,
- снижение дублирования кода,
- уменьшение поверхности атаки,
- защиту внутренних сервисов от прямого доступа.

---

### Важно для AZ-204

Если в вопросе говорится о:
- централизованной аутентификации,
- применении политик безопасности,
- проверке токенов перед доступом к сервисам,
- защите backend без изменения их кода,

— корректным решением будет использование API Gateway или Azure API Management.

```xml
<policies>
  <inbound>
    <!-- Validate JWT token -->
    <validate-jwt header-name="Authorization">
      <issuer-signing-keys>
        <key>{{jwt-signing-key}}</key>
      </issuer-signing-keys>
      <required-claims>
        <claim name="scope" match="any">
          <value>read</value>
          <value>write</value>
        </claim>
      </required-claims>
    </validate-jwt>
  </inbound>
</policies>
```

**Advantages**:
- Consistent authentication across all APIs
- Backend services trust gateway
- Easy to change auth mechanisms
- Centralized token validation

### ✅ Rate Limiting & Throttling

**Benefit**: Protect backend from overload

```xml
<policies>
  <inbound>
    <!-- Limit to 100 calls per minute per subscription -->
    <rate-limit calls="100" renewal-period="60" />
    
    <!-- Limit to 10,000 calls per month per subscription -->
    <quota calls="10000" renewal-period="2592000" />
  </inbound>
</policies>
```

**Advantages**:
- Prevent backend overload
- Fair resource allocation
- Monetization (tiered limits)
- DDoS protection

### ✅ Request/Response Transformation

**Benefit**: Adapt APIs without changing backends

```xml
<policies>
  <inbound>
    <!-- Transform REST to SOAP -->
    <set-header name="Content-Type" exists-action="override">
      <value>text/xml</value>
    </set-header>
    <set-body template="liquid">
      <soap:Envelope>
        <soap:Body>
          <GetUser>
            <userId>{{context.Request.MatchedParameters["id"]}}</userId>
          </GetUser>
        </soap:Body>
      </soap:Envelope>
    </set-body>
  </inbound>
  <outbound>
    <!-- Transform SOAP response to JSON -->
    <xml-to-json kind="direct" />
  </outbound>
</policies>
```

**Advantages**:
- Expose legacy SOAP as REST
- Change response format without backend changes
- Add/remove fields from responses
- Combine multiple backend calls

### ✅ Caching

**Benefit**: Reduce backend load and latency

```xml
<policies>
  <inbound>
    <!-- Check cache before calling backend -->
    <cache-lookup vary-by-developer="false" vary-by-developer-groups="false">
      <vary-by-query-parameter>category</vary-by-query-parameter>
    </cache-lookup>
  </inbound>
  <outbound>
    <!-- Cache response for 1 hour -->
    <cache-store duration="3600" />
  </outbound>
</policies>
```

**Преимущества**:

- Более быстрое время отклика
- Снижение нагрузки на backend
- Сокращение затрат
- Улучшенная масштабируемость

### Пояснение

API Gateway может кэшировать ответы, оптимизировать маршрутизацию и централизованно управлять трафиком.

Это приводит к:

- уменьшению количества повторных запросов к backend;
- снижению потребления вычислительных ресурсов;
- более эффективному горизонтальному масштабированию;
- снижению затрат в облачной инфраструктуре.

Дополнительно gateway может применять политики throttling и rate limiting, предотвращая перегрузку сервисов.

---

### ✅ Мониторинг и аналитика

**Преимущество**: Централизованная наблюдаемость (observability).

API Gateway выступает единой точкой контроля трафика, что позволяет собирать метрики по всем API без внедрения логики мониторинга в каждый сервис.

---

### Собираемые метрики

- Количество запросов
- Время ответа (latency)
- Процент ошибок (error rate)
- Пропускная способность (throughput)
- Использование пропускной способности сети (bandwidth)
- Топ потребителей API
- Географическое распределение запросов

---

### Архитектурное значение

Централизованный сбор метрик позволяет:

- быстро выявлять узкие места;
- анализировать поведение клиентов;
- обнаруживать аномалии и атаки;
- принимать решения по масштабированию;
- формировать SLA и отчётность.

---

### Важно для AZ-204

Если в вопросе упоминается:
- централизованный мониторинг API,
- анализ использования API,
- отслеживание производительности,
- выявление ошибок и аномалий,

— правильным выбором будет API Gateway или Azure API Management.

**Integration**:
```bash
# Enable Application Insights
az apim update \
  --name apim-instance \
  --resource-group rg-apim \
  --application-insights-instrumentation-key <key>
```

---

## Типы Gateway

Azure API Management предоставляет два типа gateway.

---

### 1. **Managed Gateway** (по умолчанию)

**Managed Gateway** — это стандартный шлюз, размещённый и управляемый в Azure.

### Характеристики

- ✅ Полностью управляется Microsoft
- ✅ Размещён в Azure
- ✅ Автоматическое масштабирование
- ✅ Встроенная высокая доступность (High Availability)
- ✅ Отсутствие необходимости управлять инфраструктурой
- ✅ Интеграция с другими сервисами Azure

---

### Пояснение

Managed Gateway — это PaaS-решение. Вам не нужно:

- управлять виртуальными машинами;
- настраивать балансировку нагрузки;
- следить за отказоустойчивостью;
- обновлять инфраструктуру.

Microsoft отвечает за:

- масштабирование,
- патчи безопасности,
- обновления платформы,
- SLA.

Это оптимальный выбор для облачных решений, где backend размещён в Azure или доступен через публичные endpoints.

---

### Когда использовать

Managed Gateway подходит, если:

- вся инфраструктура находится в Azure;
- не требуется размещение gateway в локальной сети;
- важна минимизация операционных затрат;
- нужен быстрый запуск без DevOps-нагрузки.

---

### Важно для AZ-204

Если в вопросе говорится о:
- полностью управляемом сервисе,
- автоматическом масштабировании,
- отсутствии необходимости управлять инфраструктурой,

— речь, скорее всего, идёт о Managed Gateway в Azure API Management.

**Architecture**:
```
┌──────────────────────────────────────┐
│  Internet                            │
└────────────┬─────────────────────────┘
             │
             ▼
┌────────────────────────────────────┐
│  Azure API Management              │
│  ┌──────────────────────────────┐  │
│  │  Managed Gateway (Azure)     │  │
│  │  • Azure-hosted              │  │
│  │  • Auto-scaling              │  │
│  │  • High availability         │  │
│  └──────────────┬───────────────┘  │
└─────────────────┼──────────────────┘
                  │
     ┌────────────┼────────────┐
     │            │            │
     ▼            ▼            ▼
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Azure   │  │ Azure   │  │ Azure   │
│ App     │  │ Function│  │ AKS     │
│ Service │  │         │  │         │
└─────────┘  └─────────┘  └─────────┘
```

**Use Cases**:
- Cloud-native applications
- APIs hosted in Azure
- Standard API management scenarios
- No hybrid/on-premises requirements

**Configuration**:
```bash
# Managed gateway is created by default
az apim create \
  --name apim-instance \
  --resource-group rg-apim \
  --publisher-email admin@contoso.com \
  --publisher-name Contoso \
  --sku-name Standard

# Gateway URL is automatically assigned
# Example: https://apim-instance.azure-api.net
```

**Regions** (Premium Tier Only):
```bash
# Add gateway in additional region
az apim api versionset create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --location westus2 \
  --sku-capacity 1
```

### 2. **Self-Hosted Gateway**

The **self-hosted gateway** is a containerized version that you deploy.

**Characteristics**:
- ✅ Containerized (Docker/Kubernetes)
- ✅ Deploy anywhere (on-premises, other clouds, edge)
- ✅ Same features as managed gateway
- ✅ Hybrid and multi-cloud scenarios
- ✅ Low latency for local services
- ❌ You manage infrastructure

**Architecture**:
```
┌────────────────────────────────────────────────┐
│  Azure API Management (Control Plane)          │
│  • Configuration                               │
│  • Policies                                    │
│  • Analytics                                   │
└────────────┬──────────────────────────────────┘
             │ (Configuration sync)
             │
     ┌───────┴────────┬──────────────┐
     │                │              │
     ▼                ▼              ▼
┌──────────┐    ┌──────────┐   ┌──────────┐
│Self-hosted│   │Self-hosted│  │Self-hosted│
│Gateway    │   │Gateway    │  │Gateway    │
│(On-Prem)  │   │(AWS)      │  │(Edge)     │
└─────┬─────┘   └─────┬─────┘  └─────┬─────┘
      │               │              │
      ▼               ▼              ▼
┌──────────┐    ┌──────────┐   ┌──────────┐
│Internal  │    │AWS       │   │IoT       │
│APIs      │    │Lambda    │   │Devices   │
└──────────┘    └──────────┘   └──────────┘
```

**Use Cases**:
- On-premises APIs (hybrid cloud)
- Multi-cloud deployments
- Edge computing / IoT
- Data sovereignty requirements
- Low latency requirements

**Deployment**:

**Docker**:
```bash
# Get gateway credentials from Azure
az apim gateway show \
  --resource-group rg-apim \
  --service-name apim-instance \
  --gateway-id my-gateway

# Run self-hosted gateway container
docker run \
  -d \
  -p 8080:8080 \
  -p 8081:8081 \
  --name apim-gateway \
  -e config.service.endpoint=<gateway-url> \
  -e config.service.auth=<gateway-key> \
  mcr.microsoft.com/azure-api-management/gateway:latest
```

**Kubernetes**:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: apim-gateway
spec:
  replicas: 2
  selector:
    matchLabels:
      app: apim-gateway
  template:
    metadata:
      labels:
        app: apim-gateway
    spec:
      containers:
      - name: apim-gateway
        image: mcr.microsoft.com/azure-api-management/gateway:latest
        ports:
        - containerPort: 8080
        - containerPort: 8081
        env:
        - name: config.service.endpoint
          value: <gateway-url>
        - name: config.service.auth
          valueFrom:
            secretKeyRef:
              name: apim-gateway-secret
              key: auth-token
---
apiVersion: v1
kind: Service
metadata:
  name: apim-gateway-service
spec:
  type: LoadBalancer
  ports:
  - port: 80
    targetPort: 8080
  selector:
    app: apim-gateway
```

## Конфигурация (Azure Portal)

1. Перейдите в экземпляр API Management
2. Откройте раздел **Deployment + infrastructure** → **Gateways**
3. Нажмите **+ Add**
4. Укажите имя gateway и описание локации
5. Скачайте конфигурацию для развёртывания
6. Разверните gateway с помощью Docker / Kubernetes / VM

### Пояснение

После создания gateway в Azure API Management вы получаете конфигурационные данные, которые используются при запуске контейнера Self-Hosted Gateway.

Развёртывание возможно:

- как Docker-контейнер,
- в Kubernetes-кластере,
- на виртуальной машине.

Azure API Management остаётся точкой централизованного управления, а сам gateway выполняет обработку трафика в выбранной инфраструктуре.

---

## Сравнение Gateway

| Возможность | Managed Gateway | Self-Hosted Gateway |
|-------------|------------------|----------------------|
| **Размещение** | Azure | Ваша инфраструктура |
| **Управление** | Полностью управляется | Вы управляете |
| **Масштабирование** | Автоматическое | Ручное |
| **Обновления** | Автоматические | Ручные |
| **Высокая доступность** | Встроенная | Настраивается вами |
| **Стоимость** | Включена в тариф | Затраты на инфраструктуру |
| **Задержка (Latency)** | Регион Azure | Локально рядом с сервисами |
| **Сценарий использования** | Cloud-native | Hybrid / multi-cloud |
| **Конфигурация** | Через Azure Portal | Через конфигурацию контейнера |
| **Мониторинг** | Встроенный | Настраивается вами |
| **Multi-region** | Доступно в Premium | Можно развернуть где угодно |
| **Интеграция с VNet** | Premium tier | Используется ваша сеть |
| **Пользовательские домены** | Да | Да |
| **Политики (Policies)** | Все поддерживаются | Все поддерживаются |
| **TLS termination** | Да | Да |
| **Кэширование** | Да | Да |
| **Аутентификация** | Все методы | Все методы |

---

### Ключевой вывод для AZ-204

- Managed Gateway — выбор по умолчанию для облачных решений в Azure.
- Self-Hosted Gateway — решение для гибридных, распределённых и multi-cloud архитектур.
- Premium tier часто упоминается в вопросах, связанных с VNet и multi-region.

Если в задаче требуется минимальное администрирование — выбирайте Managed.  
Если важна гибкость размещения — Self-Hosted.
---

## Gateway Routing Patterns

### 1. **Simple Pass-Through**

```
Client → Gateway → Single Backend
```

**Policy**:
```xml
<policies>
  <inbound>
    <set-backend-service base-url="https://backend.contoso.com/api" />
  </inbound>
  <backend>
    <forward-request />
  </backend>
</policies>
```

### 2. **Load Balancing**

```
Client → Gateway → Backend 1
                 → Backend 2
                 → Backend 3
```

**Policy**:
```xml
<policies>
  <inbound>
    <set-variable name="backend" value="@{
      var backends = new[] {
        "https://backend1.contoso.com",
        "https://backend2.contoso.com",
        "https://backend3.contoso.com"
      };
      return backends[new Random().Next(backends.Length)];
    }" />
    <set-backend-service base-url="@((string)context.Variables["backend"])" />
  </inbound>
</policies>
```

### 3. **Request Aggregation**

```
Client → Gateway → Backend A (get user)
                 → Backend B (get orders)
                 → Backend C (get profile)
                 
Gateway → Client (combined response)
```

**Policy**:
```xml
<policies>
  <inbound>
    <send-request mode="new" response-variable-name="user">
      <set-url>https://users.contoso.com/api/users/@(context.Request.MatchedParameters["id"])</set-url>
    </send-request>
    <send-request mode="new" response-variable-name="orders">
      <set-url>https://orders.contoso.com/api/orders?userId=@(context.Request.MatchedParameters["id"])</set-url>
    </send-request>
  </inbound>
  <outbound>
    <return-response>
      <set-body>@{
        var user = ((IResponse)context.Variables["user"]).Body.As<JObject>();
        var orders = ((IResponse)context.Variables["orders"]).Body.As<JArray>();
        var result = new JObject();
        result["user"] = user;
        result["orders"] = orders;
        return result.ToString();
      }</set-body>
    </return-response>
  </outbound>
</policies>
```

### 4. **Circuit Breaker**

```
Client → Gateway → Backend (healthy)  → OK
Client → Gateway → Backend (failing)  → Cached response
```

**Policy**:
```xml
<policies>
  <inbound>
    <cache-lookup vary-by-developer="false" />
  </inbound>
  <backend>
    <retry condition="@(context.Response.StatusCode >= 500)" count="3" interval="5">
      <forward-request timeout="10" />
    </retry>
  </backend>
  <outbound>
    <cache-store duration="300" />
  </outbound>
  <on-error>
    <return-response>
      <set-status code="503" reason="Service Unavailable" />
      <set-body>Service temporarily unavailable</set-body>
    </return-response>
  </on-error>
</policies>
```

---

## Best Practices

### 1. **Используйте Managed Gateway для облачных нагрузок**

✅ **Рекомендуется**: использовать Managed Gateway для API, размещённых в Azure

- Отсутствие необходимости управлять инфраструктурой
- Автоматическое масштабирование и обновления
- Встроенная высокая доступность

❌ **Не рекомендуется**: разворачивать Self-Hosted Gateway для сервисов, уже работающих в Azure

Это создаёт лишнюю операционную нагрузку без архитектурной необходимости.

---

### 2. **Размещайте Self-Hosted Gateway рядом с сервисами**

✅ **Рекомендуется**: разворачивать Self-Hosted Gateway в той же сети, что и backend-сервисы

Это позволяет:

- снизить latency;
- минимизировать сетевые переходы;
- повысить производительность;
- упростить доступ к приватным ресурсам.

Особенно важно в hybrid-сценариях, когда backend находится в on-prem или в изолированной сети.

---

### Архитектурная рекомендация

- Managed Gateway — для cloud-native решений в Azure.
- Self-Hosted Gateway — когда требуется контроль над размещением, изоляцией сети или минимальной задержкой.

---

### Важно для AZ-204

Если в вопросе подчёркивается:
- минимизация администрирования — выбирайте Managed Gateway;
- размещение рядом с приватными сервисами — выбирайте Self-Hosted Gateway.
```
On-Premises:
  Self-Hosted Gateway → Internal APIs (low latency)
  
AWS:
  Self-Hosted Gateway → AWS Lambda (low latency)
```

### 3. **Implement Health Checks**

✅ **Do**: Configure backend health checks
```xml
<policies>
  <inbound>
    <set-backend-service backend-id="backend-pool" />
  </inbound>
  <backend>
    <forward-request timeout="30" />
  </backend>
  <on-error>
    <choose>
      <when condition="@(context.LastError.Reason == "TimedOut")">
        <!-- Switch to backup backend -->
        <set-backend-service base-url="https://backup.contoso.com" />
        <forward-request timeout="30" />
      </when>
    </choose>
  </on-error>
</policies>
```

### 4. **Enable Gateway Logging**

```bash
# Enable diagnostic logs
az monitor diagnostic-settings create \
  --name apim-diagnostics \
  --resource <apim-resource-id> \
  --logs '[{"category": "GatewayLogs", "enabled": true}]' \
  --workspace <log-analytics-workspace-id>
```

### 5. **Use Multiple Regions (Premium)**

```bash
# Add gateway in West Europe
az apim update \
  --name apim-instance \
  --resource-group rg-apim \
  --add additionalLocations location=westeurope sku-capacity=1
```

---

## Советы к экзамену

### Ключевые концепции для AZ-204

1. **Два типа gateway**:
    - Managed (размещён в Azure)
    - Self-hosted (контейнеризированный)

2. **Managed Gateway**:  
   По умолчанию, полностью управляется Microsoft, размещён в Azure, поддерживает авто-масштабирование.

3. **Self-Hosted Gateway**:  
   Можно развернуть где угодно, работает в контейнере, подходит для hybrid и multi-cloud сценариев.

4. **Преимущества Gateway**:
    - Разделение (decoupling) клиентов и backend
    - TLS termination
    - Централизованная аутентификация
    - Rate limiting
    - Трансформация запросов и ответов

5. **Доступность Self-Hosted Gateway**:  
   Поддерживается в тарифах Developer, Standard и Premium.

6. **Multi-region**:  
   Поддерживается только Managed Gateway и только в Premium tier.

7. **Gateway URL по умолчанию**:  
   `https://<apim-name>.azure-api.net`

8. **Развёртывание Self-Hosted Gateway**:  
   Docker, Kubernetes или виртуальная машина.

9. **Синхронизация конфигурации**:  
   Self-hosted gateway получает конфигурацию из Azure API Management.

10. **Типовые сценарии использования**:
    - Managed → Cloud-native решения, API размещены в Azure
    - Self-hosted → Hybrid, on-premises, multi-cloud, edge-сценарии

---

## Частые экзаменационные сценарии

**Сценарий 1**:  
"Потребители API не должны знать URL backend-сервисов"  
→ **Ответ**: Использовать API Gateway для отделения клиентов от backend.

---

**Сценарий 2**:  
"Необходимо снизить задержку для on-premises API"  
→ **Ответ**: Развернуть Self-Hosted Gateway в локальной инфраструктуре.

---

**Сценарий 3**:  
"Объединить ответы от нескольких backend-сервисов"  
→ **Ответ**: Использовать политики gateway (например, send-request) для агрегации.

---

**Сценарий 4**:  
"Развернуть API gateway в AWS и Azure"  
→ **Ответ**: Использовать Self-Hosted Gateway (контейнеризированный) в обоих облаках.

---

**Сценарий 5**:  
"Автоматически масштабировать gateway в зависимости от нагрузки"  
→ **Ответ**: Использовать Managed Gateway (авто-масштабирование встроено).

---

### Финальный акцент для AZ-204

- Managed = меньше администрирования, больше автоматизации.
- Self-hosted = гибкость размещения и гибридная архитектура.
- Premium tier часто является ключевым условием в вопросах про multi-region и VNet.
- Если в задаче говорится о контейнеризации gateway — это почти всегда Self-Hosted вариант.
---

## Quick Reference Commands

```bash
# Create APIM instance with managed gateway
az apim create --name <name> --resource-group <rg> --publisher-email <email> --publisher-name <name> --sku-name Standard

# Get gateway URL
az apim show --name <name> --resource-group <rg> --query gatewayUrl -o tsv

# Create self-hosted gateway
az apim gateway create --resource-group <rg> --service-name <apim-name> --gateway-id <gateway-id> --location-data "On-Premises Data Center" --description "Self-hosted gateway"

# List gateways
az apim gateway list --resource-group <rg> --service-name <apim-name>

# Get gateway key
az apim gateway key list --resource-group <rg> --service-name <apim-name> --gateway-id <gateway-id>

# Add multi-region gateway (Premium)
az apim update --name <name> --resource-group <rg> --add additionalLocations location=<region> sku-capacity=1

# Deploy self-hosted gateway (Docker)
docker run -d -p 8080:8080 --name apim-gateway -e config.service.endpoint=<url> -e config.service.auth=<key> mcr.microsoft.com/azure-api-management/gateway:latest

# Deploy self-hosted gateway (Kubernetes)
kubectl apply -f apim-gateway-deployment.yaml

# Check gateway health
curl http://<gateway-url>/status-0123456789abcdef
```

---

## Learn More

- [API Gateway Pattern](https://docs.microsoft.com/azure/architecture/microservices/design/gateway)
- [Self-Hosted Gateway Documentation](https://docs.microsoft.com/azure/api-management/self-hosted-gateway-overview)
- [Multi-Region Deployment](https://docs.microsoft.com/azure/api-management/api-management-howto-deploy-multi-region)
- [Gateway Policies](https://docs.microsoft.com/azure/api-management/api-management-policies)
