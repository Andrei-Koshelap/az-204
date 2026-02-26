# API Management Service — Обзор

## Что такое Azure API Management?

Azure API Management (APIM) — это **полностью управляемый сервис**, который позволяет организациям публиковать, 
защищать, трансформировать, сопровождать и мониторить API.

Он выступает в роли фасада (API Gateway) для backend-сервисов, предоставляя единую точку входа для 
клиентов и изолируя их от внутренней реализации.

**Ключевая цель**: создать единый, современный API-шлюз для существующих backend-сервисов, 
независимо от того, где они размещены (Azure, on-premises, другие облака).

> 💡 На экзамене AZ-204 важно понимать, что APIM — это не просто прокси, а полноценный API Gateway 
> с политиками безопасности, трансформации и мониторинга.

---

## Основные компоненты

Azure API Management состоит из трёх ключевых компонентов:

### 1. **API Gateway (Data Plane)**


::contentReference[oaicite:0]{index=0}


**API Gateway** — это runtime-компонент, который обрабатывает входящие API-запросы.

### Зоны ответственности:

- ✅ **Маршрутизация запросов** к соответствующим backend-сервисам
- ✅ **Проверка учетных данных** (API keys, JWT-токены, сертификаты)
- ✅ **Применение квот и rate limiting**
- ✅ **Трансформация запросов и ответов** через policies
- ✅ **Кэширование ответов** для повышения производительности
- ✅ **Генерация телеметрии** (логи, метрики, трассировки)

---

### 🔎 Дополнение для понимания архитектуры

Data Plane отвечает за **обработку трафика**, тогда как управление конфигурацией (создание API, настройка политик и т.д.) происходит через **Control Plane**.

На практике это означает:

- Gateway масштабируется для обработки нагрузки
- Конфигурация централизованно управляется через Azure Portal, ARM, CLI или REST API
- Изменения политик применяются без необходимости изменять backend

---

### ⚠️ Важно для AZ-204

Если в вопросе говорится о:

- ограничении количества запросов → **rate-limit policy**
- защите API через подписку → **subscription key**
- трансформации JSON ↔ XML → **policy transformation**
- централизованной публикации API → **API Management**

— почти всегда правильный ответ связан с APIM.

**Architecture**:
```
┌──────────────────────────────────────────────────────────┐
│  API Consumers (Mobile, Web, Partners)                   │
└─────────────────────┬────────────────────────────────────┘
                      │
                      ▼
        ┌─────────────────────────────┐
        │  API Gateway (Data Plane)   │
        │  • Authentication           │
        │  • Rate Limiting            │
        │  • Request Transformation   │
        │  • Response Caching         │
        │  • Logging & Monitoring     │
        └──────────┬──────────────────┘
                   │
     ┌─────────────┼─────────────┐
     │             │             │
     ▼             ▼             ▼
┌─────────┐  ┌─────────┐  ┌─────────┐
│Backend  │  │Backend  │  │Backend  │
│Service 1│  │Service 2│  │Service 3│
└─────────┘  └─────────┘  └─────────┘
```

### 2. **Management Plane (Azure Portal)**


::contentReference[oaicite:0]{index=0}


**Management Plane** — это административный уровень управления APIM, через который выполняется настройка и конфигурация сервиса.

### Возможности:

- ✅ **Развёртывание и настройка** экземпляра API Management
- ✅ **Определение или импорт** API-схем (OpenAPI, WADL, WSDL)
- ✅ **Группировка API в продукты (Products)**
- ✅ **Настройка политик (policies)** — квоты, трансформации, безопасность
- ✅ **Управление пользователями** и подписками
- ✅ **Просмотр аналитики** и метрик

### Способы доступа:

- Azure Portal (GUI)
- Azure CLI
- Azure PowerShell
- REST API
- ARM Templates / Bicep

> 💡 Важно для экзамена:  
> Management Plane = **Control Plane**.  
> Он отвечает за конфигурацию, а не за обработку трафика.

---

### 3. **Developer Portal**


::contentReference[oaicite:1]{index=1}


**Developer Portal** — это автоматически создаваемый и настраиваемый веб-сайт для потребителей API.

Он служит точкой взаимодействия между разработчиками и опубликованными API.

### Возможности для разработчиков:

- ✅ **Просмотр документации API** (интерактивный reference)
- ✅ **Тестирование API** через встроенную консоль
- ✅ **Подписка на продукты** для получения API-ключей
- ✅ **Управление ключами** (перегенерация, просмотр использования)
- ✅ **Просмотр собственной статистики использования**
- ✅ **Скачивание спецификаций API** (OpenAPI / Swagger)

### Кастомизация:

- Брендирование (логотипы, цвета, темы)
- Добавление пользовательских страниц
- Интеграция OAuth 2.0 / OpenID Connect
- Самостоятельная регистрация пользователей (self-service)

> 🔎 На экзамене часто проверяют понимание разницы:
> - Developer Portal — для **потребителей API**
> - Management Plane — для **администраторов API**

---

## Ключевые понятия

### APIs

В контексте APIM, **API** — это логическая группа операций (endpoint’ов), доступных для вызова через шлюз.

### Свойства API:

- Имя и описание
- URL backend-сервиса
- Путь (например, `/api/users`)
- Поддерживаемые протоколы (HTTP, HTTPS, WebSocket)
- Операции (GET, POST, PUT, DELETE и т.д.)

> 💡 APIM позволяет импортировать существующие API (например, из OpenAPI/Swagger) и «обернуть» их политиками безопасности и управления трафиком без изменения backend-кода.

**Example**:
```
API: User Management API
Base URL: https://apim-instance.azure-api.net/users
Operations:
  - GET /users          → List all users
  - GET /users/{id}     → Get user by ID
  - POST /users         → Create new user
  - PUT /users/{id}     → Update user
  - DELETE /users/{id}  → Delete user
```

### Products


::contentReference[oaicite:0]{index=0}


**Products (Продукты)** — это способ публикации API для разработчиков.  
Продукт объединяет один или несколько API и определяет условия их использования.

Именно через продукты разработчики получают доступ к API (через подписки).

---

### Типы продуктов

- **Open Products (Открытые)**  
  Доступны без подписки (subscription key не требуется)

- **Protected Products (Защищённые)**  
  Требуют оформления подписки для получения доступа

> 💡 На экзамене важно помнить:  
> Доступ к API обычно контролируется через **product subscription**, а не напрямую через сам API.

---

### Свойства продукта

- Заголовок и описание
- Условия использования (Terms of use)
- Требование подписки
- Процесс одобрения (автоматический или через администратора)
- Квоты и ограничения частоты запросов (rate limits)

---

### Архитектурная логика

Связь выглядит так:
```
Developer → Subscribes to Product → Gets Subscription Key → Calls API via Gateway
```

То есть:
- API входят в продукт
- Пользователь подписывается на продукт
- Подписка генерирует ключ
- Ключ передаётся в запросе к API

---

### ⚠️ Частые экзаменационные сценарии

**Сценарий**: «Ограничить доступ к API только для зарегистрированных пользователей»  
→ Использовать **Protected Product** с подпиской

**Сценарий**: «Ограничить 1000 запросов в минуту для клиентов»  
→ Настроить rate-limit policy на уровне продукта

**Сценарий**: «Предоставить публичный API без ключей»  
→ Создать **Open Product**

---

### Практическое замечание

Продукты позволяют реализовать монетизацию API, разграничение тарифов (Free / Standard / Premium) и изоляцию клиентов без изменения backend-сервисов.


**Example**:
```
Product: Starter
  - APIs: Users API, Orders API (read-only)
  - Subscription: Required
  - Quota: 1,000 calls/month
  - Rate Limit: 10 calls/minute
  - Price: Free

Product: Enterprise
  - APIs: Users API, Orders API, Analytics API (full access)
  - Subscription: Required (admin approval)
  - Quota: 1,000,000 calls/month
  - Rate Limit: 1,000 calls/minute
  - Price: $500/month
```

### Groups


::contentReference[oaicite:0]{index=0}


**Groups (Группы)** управляют видимостью продуктов для разработчиков.  
Через группы определяется, какие пользователи могут видеть и подписываться на конкретные продукты.

> 💡 Важно: доступ к продукту = членство в группе + подписка (если требуется).

---

## Встроенные системные группы

| Группа | Описание | Членство |
|--------|----------|----------|
| **Administrators** | Управляют экземпляром APIM, создают API и продукты | Администраторы подписки Azure |
| **Developers** | Аутентифицированные пользователи Developer Portal | Зарегистрированные разработчики |
| **Guests** | Неаутентифицированные посетители портала | Анонимные пользователи |

---

## Пользовательские группы (Custom Groups)

Можно создавать собственные группы для сегментации разработчиков:

- Разделение по партнёрам / клиентам
- Разные тарифные планы
- Внутренние vs внешние пользователи

### Возможности:

- Интеграция с **Microsoft Entra ID (Azure AD)** группами
- Назначение разных уровней доступа к продуктам
- Централизованное управление доступом через корпоративную директорию

---

## Как это работает вместе
```
Group → Has Access to Product → Contains APIs
User → Member of Group → Can Subscribe to Product
```

То есть:

- Группа определяет, какие продукты видны
- Пользователь должен состоять в группе
- После подписки получает subscription key
- Затем вызывает API через Gateway

---

## ⚠️ Частые экзаменационные сценарии

**Сценарий**: «Ограничить доступ к API только для внутренней команды»  
→ Создать custom group и предоставить доступ только ей

**Сценарий**: «Использовать корпоративную аутентификацию»  
→ Интеграция с Microsoft Entra ID

**Сценарий**: «Сделать API полностью публичным»  
→ Продукт доступен группе Guests и не требует подписки

---

### Практическое замечание

Groups позволяют реализовать:

- RBAC-модель на уровне API-публикации
- Разделение партнёров
- Многоуровневый доступ
- Enterprise SSO-интеграцию

Это один из ключевых механизмов разграничения доступа в Azure API Management.

**Example**:
```
Group: Premium Partners
  - Members: partner1@contoso.com, partner2@fabrikam.com
  - Products: Enterprise (full access)
  - Quota: Custom (10M calls/month)
```

### Developers


::contentReference[oaicite:0]{index=0}


**Developers (Разработчики)** — это пользовательские учётные записи, которые потребляют API через Developer Portal.

Они не управляют APIM, а используют опубликованные API.

---

## Жизненный цикл разработчика

1. **Регистрация (Sign up)** через Developer Portal  
   *(или приглашение администратором)*
2. **Просмотр продуктов** и документации API
3. **Подписка (Subscribe)** на продукт для получения API-ключа
4. **Тестирование API** через интерактивную консоль
5. **Интеграция API** в свои приложения
6. **Мониторинг использования** и аналитики

---

## Управление разработчиками

Администратор может:

- Приглашать разработчиков по email
- Назначать их в группы
- Одобрять или отклонять запросы на подписку
- Просматривать статистику использования API

> 💡 На экзамене важно помнить:  
> Разработчик получает доступ не к API напрямую, а через **продукт и подписку**.

---

# Subscriptions


::contentReference[oaicite:1]{index=1}


**Subscriptions (Подписки)** предоставляют доступ к API, входящим в продукт.

Каждая подписка генерирует **два ключа**: primary и secondary.

Это позволяет безопасно выполнять ротацию ключей без простоя.

---

## Области действия подписки (Subscription Scopes)

| Scope | Описание | Сценарий использования |
|--------|----------|------------------------|
| **All APIs** | Доступ ко всем API в APIM | Администрирование / тестирование |
| **Single API** | Доступ к одному конкретному API | Ограниченная интеграция |
| **Product** | Доступ ко всем API в продукте | Наиболее распространённый вариант (рекомендуется) |

> ✅ В большинстве production-сценариев используется **Product scope**.

---

## Свойства подписки

- **Primary key** и **Secondary key**
- Статус (active, suspended, cancelled)
- Scope (product, API или all APIs)
- Владелец (разработчик или группа)

---

## Как используется subscription key

Ключ передаётся:

- В HTTP-заголовке (обычно `Ocp-Apim-Subscription-Key`)
- Или как query-параметр

Gateway проверяет ключ и применяет политики (квоты, лимиты и т.д.).

---

## ⚠️ Частые экзаменационные сценарии

**Сценарий**: «Нужно выполнить ротацию ключа без остановки клиентов»  
→ Использовать secondary key, затем регенерировать primary

**Сценарий**: «Временно заблокировать доступ клиенту»  
→ Перевести subscription в состояние *suspended*

**Сценарий**: «Дать доступ только к определённому API»  
→ Создать подписку со scope = Single API

---

### Практическое замечание

Subscriptions — это основной механизм:

- Аутентификации клиентов
- Отслеживания потребления
- Ограничения трафика
- Реализации тарифных планов

Это один из самых часто проверяемых механизмов в вопросах AZ-204.

**Key Rotation**:
```bash
# Regenerate primary key
az apim api subscription update \
  --resource-group rg-apim \
  --service-name apim-instance \
  --sid subscription-id \
  --primary-key $(uuidgen)
```

### Policies


::contentReference[oaicite:0]{index=0}


**Policies (Политики)** — это набор XML-выражений, которые изменяют поведение API во время обработки запроса.

Они позволяют управлять трафиком, безопасностью и трансформацией данных **без изменения backend-кода**.

> 💡 Ключевая идея: Policies выполняются в API Gateway и формируют полноценный middleware pipeline.

---

## Типичные сценарии использования

- Ограничение частоты запросов (rate limiting) и квоты
- Трансформация запросов и ответов (JSON ↔ XML, изменение заголовков)
- Аутентификация и авторизация (JWT validation, OAuth 2.0)
- Кэширование ответов
- Обработка ошибок
- Логирование и отправка телеметрии

---

## Области применения политик (Policy Scopes)

Политики могут применяться на разных уровнях.  
Если политики заданы на нескольких уровнях, применяется принцип наследования и объединения.

**Порядок приоритета (от общего к частному):**

1. **Global** — применяется ко всем API
2. **Product** — ко всем API внутри продукта
3. **API** — ко всем операциям конкретного API
4. **Operation** — к конкретной операции

> ⚠️ Чем ниже уровень, тем более специфичной является политика.  
> Operation-level политика позволяет переопределить поведение для одного endpoint.

---

## Pipeline выполнения

Политики выполняются в следующих секциях:

- **Inbound** — до отправки запроса в backend
- **Backend** — при взаимодействии с backend
- **Outbound** — перед отправкой ответа клиенту
- **On-error** — при возникновении ошибки

Это важно для понимания, где именно происходит трансформация или проверка.

---

## Частые экзаменационные сценарии

**Сценарий**: «Ограничить 100 вызовов в минуту для клиентов»  
→ Использовать `rate-limit` policy

**Сценарий**: «Проверить JWT-токен перед передачей запроса в backend»  
→ Использовать `validate-jwt` в секции inbound

**Сценарий**: «Изменить структуру ответа API без изменения backend»  
→ Использовать transformation policy в outbound

**Сценарий**: «Кэшировать GET-запросы для повышения производительности»  
→ Использовать caching policy

---

### Практическое замечание

Policies — это один из самых мощных механизмов APIM:

- Позволяют реализовать Zero-Trust модель
- Централизуют безопасность
- Упрощают версионирование API
- Исключают необходимость дублирования логики в backend

Вопросы про APIM на AZ-204 очень часто связаны именно с правильным выбором политики и её уровня применения.


**Example**:
```xml
<policies>
  <inbound>
    <rate-limit calls="100" renewal-period="60" />
    <set-header name="X-API-Version" exists-action="override">
      <value>v1.0</value>
    </set-header>
  </inbound>
  <backend>
    <forward-request />
  </backend>
  <outbound>
    <set-header name="X-Powered-By" exists-action="delete" />
  </outbound>
  <on-error>
    <set-status code="500" reason="Internal Server Error" />
  </on-error>
</policies>
```

---
## Service Tiers (Тарифные планы)


::contentReference[oaicite:0]{index=0}


Azure API Management предоставляет несколько тарифных планов, ориентированных на разные сценарии — от serverless-разработки до enterprise-уровня.

| Tier | Возможности | Сценарий использования | SLA |
|------|-------------|------------------------|-----|
| **Consumption** | Serverless, оплата за выполнение | Serverless-приложения, dev/test | Нет |
| **Developer** | Полный функционал, без SLA | Разработка и тестирование | Нет |
| **Basic** | До 2 unit'ов, ограниченные возможности | Небольшие production-нагрузки | 99.95% |
| **Standard** | До 4 unit'ов, полный функционал | Средние production-нагрузки | 99.95% |
| **Premium** | Multi-region, VNet, высокая масштабируемость | Enterprise production | 99.99% |
| **Isolated** | Выделенная среда | Требования комплаенса и изоляции | 99.99% |

---

## Сравнение тарифов

| Функция | Consumption | Developer | Basic | Standard | Premium | Isolated |
|----------|-------------|-----------|-------|----------|---------|----------|
| **Max Units** | Автомасштабирование | 1 | 2 | 4 | Без ограничений | Настраивается |
| **Макс. пропускная способность** | Переменная | ~500 req/sec | ~1K req/sec | ~2.5K req/sec | Высокая | Очень высокая |
| **SLA** | ❌ Нет | ❌ Нет | ✅ 99.95% | ✅ 99.95% | ✅ 99.99% | ✅ 99.99% |
| **Multi-region** | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| **VNet Integration** | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| **Self-hosted Gateway** | ❌ | ✅ | ❌ | ✅ | ✅ | ✅ |
| **Caching** | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Developer Portal** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Custom Domains** | ❌ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **OAuth 2.0** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Стоимость** | Pay-per-use | ~$50/мес | ~$150/мес | ~$700/мес | ~$2800+/мес | Индивидуально |

> ⚠️ Цены ориентировочные и могут отличаться по региону.

---

## Как выбрать тариф

### 🔹 Consumption Tier

- ✅ Serverless-сценарии (Azure Functions, Logic Apps)
- ✅ Непредсказуемый или нерегулярный трафик
- ✅ Dev/test
- ❌ Нет SLA
- ❌ Нет VNet и кэширования

Подходит для облачно-нативных и event-driven архитектур.

---

### 🔹 Developer Tier

- ✅ Полный функционал для тестирования
- ✅ Оценка возможностей APIM
- ❌ Нет SLA
- ❌ Не предназначен для production

Используется как «sandbox».

---

### 🔹 Basic / Standard

- ✅ Production-нагрузки
- ✅ Предсказуемый трафик
- ✅ SLA 99.95%
- ❌ Только один регион

Подходят для малого и среднего бизнеса.

---

### 🔹 Premium

- ✅ Enterprise production
- ✅ Развёртывание в нескольких регионах
- ✅ Интеграция с VNet
- ✅ Высокая доступность
- ✅ Масштабирование

Часто используется в микросервисной архитектуре и гибридных сценариях.

---

### 🔹 Isolated

- ✅ Строгие требования по безопасности
- ✅ Полная сетевая изоляция
- ✅ Выделенная инфраструктура

Подходит для финансового сектора, госорганизаций и компаний с регуляторными требованиями.

---

## ⚠️ Частые экзаменационные сценарии

**Сценарий**: «Нужно multi-region и VNet integration»  
→ Ответ: **Premium**

**Сценарий**: «Production workload с SLA, но без multi-region»  
→ Ответ: **Basic или Standard**

**Сценарий**: «Serverless приложение с нерегулярным трафиком»  
→ Ответ: **Consumption**

**Сценарий**: «Нужна тестовая среда с полным функционалом»  
→ Ответ: **Developer**

---

### Практическое замечание

Выбор тарифа — это баланс между:

- Требованиями к SLA
- Масштабируемостью
- Сетевой интеграцией
- Стоимостью

На AZ-204 чаще всего проверяется понимание различий между **Consumption**, **Standard** и **Premium**, особенно в контексте VNet и multi-region.

## Common Use Cases

### 1. **Microservices API Gateway**

**Scenario**: Expose multiple microservices through unified API

```
Mobile App → APIM Gateway → Order Service
                          → User Service
                          → Payment Service
                          → Inventory Service
```

**Преимущества**:
- Единая точка входа для потребителей
- Единая (консистентная) аутентификация для всех сервисов
- Rate limiting для каждого потребителя
- Трансформация запросов и ответов

> 💡 От себя (для практики и AZ-204):
> - APIM хорошо «прячет» внутреннюю топологию: можно менять backend’ы без изменений у клиентов.
> - Политики позволяют централизованно применять безопасность и ограничения, не размазывая это по микросервисам.
> - Для мониторинга почти всегда подключают Application Insights/Log Analytics, чтобы видеть SLA и проблемные места.

---

### 2. **Модернизация legacy API**

**Сценарий**: добавить современные возможности API для устаревших SOAP-сервисов

**Настройка APIM**:
- Импортировать WSDL из SOAP-сервиса
- Трансформировать SOAP в REST
- Добавить аутентификацию OAuth 2.0
- Применить rate limiting
- Кэшировать ответы

> 🔎 Что обычно имеют в виду под «SOAP → REST» в APIM:
> - фронт для клиентов становится RESTful (понятные URL + JSON),
> - а внутри APIM может вызывать SOAP backend и/или преобразовывать payload через политики.

> ⚠️ Экзаменационный акцент:
> Если в вопросе фигурируют **WSDL/WSDL import, SOAP, transformation, OAuth** — APIM почти наверняка правильный сервис.

**Before**:
```
Client → SOAP/XML → Legacy Service
```

**After**:
```
Client → REST/JSON → APIM → SOAP/XML → Legacy Service
```
### 3. **Монетизация Partner API**

**Сценарий**: предоставить партнёрам доступ к API с разными тарифными планами.

---

## Продукты (тарифная модель)

- **Free Tier** — ограниченный объём вызовов, базовый доступ
- **Standard Tier** — расширенные лимиты и возможности
- **Premium Tier** — максимальный доступ и отсутствие ограничений

Каждый тариф реализуется как отдельный **Product** с индивидуальными настройками доступа.

---

## Политики и механизмы

- Отдельные subscription keys для каждого тарифа
- Применение квот (quota)
- Ограничение частоты запросов (rate limiting)
- Сбор аналитики по каждому партнёру
- Интеграция с системой биллинга

---

### Архитектурная логика

- Партнёр подписывается на продукт
- Получает ключ подписки
- Выполняет вызовы API через Gateway
- APIM применяет политики и фиксирует использование

---

### Что важно для AZ-204

- Монетизация реализуется через **Products + Subscriptions + Policies**
- Квоты ограничивают общее количество вызовов за период
- Rate limiting ограничивает частоту запросов
- Аналитика позволяет учитывать использование по подписке

---

## 4. **Версионирование API**

**Сценарий**: поддерживать несколько версий API одновременно без прерывания работы клиентов.

APIM поддерживает механизм **Version Sets**, который позволяет логически объединять версии одного API.

---

## Стратегии версионирования

- Через путь URL
- Через query-параметр
- Через HTTP-заголовок

---

## Ключевые моменты для экзамена

- Несколько версий API могут существовать параллельно
- Версии объединяются в Version Set
- Backend для разных версий может отличаться
- Клиенты могут постепенно мигрировать на новую версию
- Политики можно применять отдельно для каждой версии

---

### Практическое замечание

Корректная стратегия версионирования позволяет:

- Сохранять обратную совместимость
- Избегать breaking changes
- Управлять миграцией клиентов
- Минимизировать риски при обновлении API

Версионирование — частая тема экзаменационных вопросов, особенно в контексте поддержки backward compatibility.
```
https://apim.azure-api.net/v1/users
https://apim.azure-api.net/v2/users
```

**Query String**:
```
https://apim.azure-api.net/users?api-version=1.0
https://apim.azure-api.net/users?api-version=2.0
```

**Header**:
```
GET /users HTTP/1.1
Api-Version: 1.0
```

### 5. **Multi-Cloud/Hybrid Integration**

**Scenario**: APIs hosted in Azure, AWS, on-premises

```
┌──────────────────────────────────────┐
│  Azure API Management (Premium)      │
└────────┬─────────────────────────────┘
         │
    ┌────┴────┬──────────┬──────────┐
    │         │          │          │
    ▼         ▼          ▼          ▼
 Azure     AWS       On-Prem   GCP
 App       Lambda    APIs      Cloud
 Service                       Run
```

**Преимущества**:

- Единый и унифицированный API-опыт для потребителей
- Развёртывание в нескольких регионах (multi-region)
- Возможность использования self-hosted gateway для on-premises инфраструктуры
- Централизованная безопасность и мониторинг

---

### Дополнительно (важно для понимания архитектуры)

- Централизация управления API снижает сложность распределённых систем
- Multi-region повышает отказоустойчивость и снижает задержки
- Self-hosted gateway позволяет применять политики APIM вне Azure
- Единые механизмы логирования упрощают аудит и диагностику

Эти преимущества часто фигурируют в вопросах, где требуется выбрать решение для enterprise-архитектуры.

---

## Quick Start Example

### Create APIM Instance

```bash
# Variables
RESOURCE_GROUP="rg-apim"
LOCATION="eastus"
APIM_NAME="apim-mycompany"
PUBLISHER_EMAIL="admin@mycompany.com"
PUBLISHER_NAME="My Company"

# Create resource group
az group create --name $RESOURCE_GROUP --location $LOCATION

# Create API Management instance (Developer tier)
az apim create \
  --name $APIM_NAME \
  --resource-group $RESOURCE_GROUP \
  --location $LOCATION \
  --publisher-email $PUBLISHER_EMAIL \
  --publisher-name "$PUBLISHER_NAME" \
  --sku-name Developer \
  --sku-capacity 1

# Note: Creation takes 30-40 minutes
```

### Import API from OpenAPI

```bash
# Import Swagger/OpenAPI spec
az apim api import \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --path /users \
  --specification-url https://example.com/api/swagger.json \
  --specification-format OpenApiJson \
  --display-name "Users API" \
  --protocols https
```

### Create Product

```bash
# Create product
az apim product create \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --product-id starter \
  --product-name "Starter" \
  --description "Starter tier for developers" \
  --subscription-required true \
  --approval-required false \
  --state published

# Add API to product
az apim product api add \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --product-id starter \
  --api-id users-api
```

### Configure Rate Limiting Policy

```bash
# Apply rate limiting policy
az apim api operation policy create \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --api-id users-api \
  --operation-id get-users \
  --xml-policy '<policies>
    <inbound>
      <rate-limit calls="100" renewal-period="60" />
      <base />
    </inbound>
  </policies>'
```

---

## Best Practices

### 1. **Use Products for API Grouping**

✅ **Do**: Organize APIs into products
```
Product: Internal APIs
  - User Management API
  - Order Processing API
  
Product: Partner APIs
  - Public Catalog API
  - Shipping Status API
```

❌ **Don't**: Expose individual APIs directly

### 2. **Implement Proper Versioning**

✅ **Do**: Use version sets
```
- API: Users v1 (/v1/users)
- API: Users v2 (/v2/users)
- API: Users v3 (/v3/users)
```

### 3. **Apply Policies at Appropriate Scope**

✅ **Do**: Apply common policies globally
```
Global: Authentication, logging
Product: Rate limiting (per tier)
API: Transformation (per API)
Operation: Specific validation
```

### 4. **Enable Caching**

```xml
<policies>
  <inbound>
    <cache-lookup vary-by-developer="false" vary-by-developer-groups="false" />
  </inbound>
  <outbound>
    <cache-store duration="3600" />
  </outbound>
</policies>
```

### 5. **Use Named Values for Configuration**

```bash
# Store backend URL as named value
az apim nv create \
  --resource-group $RESOURCE_GROUP \
  --service-name $APIM_NAME \
  --named-value-id backend-url \
  --display-name "Backend URL" \
  --value "https://backend.mycompany.com"
```

### 6. **Мониторинг использования API**


::contentReference[oaicite:0]{index=0}


- Включить интеграцию с Application Insights
- Отслеживать метрики API (задержка, ошибки, пропускная способность)
- Настроить оповещения при аномалиях
- Регулярно анализировать отчёты и статистику

---

### Что важно понимать

- Метрики позволяют выявлять узкие места и проблемы производительности
- Логи помогают диагностировать ошибки на уровне Gateway
- Alerts позволяют реагировать до того, как пользователи заметят проблему
- Аналитика по подпискам помогает контролировать использование и SLA

Мониторинг — критически важная часть production-сценариев и частая тема вопросов на экзамене.

---

# Exam Tips

## Ключевые концепции для AZ-204

1. **Три компонента**: API Gateway, Management Plane, Developer Portal

2. **Products**: контейнер для одного или нескольких API, могут быть Open или Protected

3. **Scopes подписок**: All APIs, Single API, Product

4. **Groups**: Administrators, Developers, Guests (+ пользовательские группы)

5. **Policy scopes**: Global > Product > API > Operation

6. **Тарифы**:
    - Consumption (serverless)
    - Developer (без SLA)
    - Basic/Standard (production)
    - Premium (multi-region, VNet)

7. **Developer Portal**: автоматически создаётся, настраивается, поддерживает self-service

8. **Subscription keys**: primary и secondary ключ на каждую подписку

9. **Порядок выполнения политик**: inbound → backend → outbound → on-error

10. **VNet integration**: доступна только в Premium и Isolated

11. **Multi-region**: доступен только в Premium и Isolated

12. **Self-hosted gateway**: развёртывание gateway в собственной инфраструктуре (Premium tier)

---

## Частые экзаменационные сценарии

**Сценарий 1**:  
«Опубликовать несколько backend-сервисов через одну точку входа»  
→ Использовать Azure API Management как API Gateway

**Сценарий 2**:  
«Контролировать доступ к API с разными квотами»  
→ Создать разные продукты с собственными квотами и rate limiting

**Сценарий 3**:  
«Требовать подписку для production, но разрешить бесплатное тестирование»  
→ Создать Open product и Protected product

**Сценарий 4**:  
«Развернуть API gateway в on-premises дата-центре»  
→ Использовать self-hosted gateway (Premium tier)

**Сценарий 5**:  
«Трансформировать SOAP backend в REST для мобильных клиентов»  
→ Использовать политики APIM для трансформации запросов и ответов

---

### Финальный совет

В задачах AZ-204 почти всегда нужно определить:

- Где применяется политика
- Какой тариф подходит
- Нужна ли подписка
- Требуется ли multi-region или VNet
- Кто управляет доступом (Group / Product / Subscription)

Правильный ответ обычно строится вокруг комбинации **Products + Policies + Tier + Gateway возможностей**.

## Quick Reference Commands

```bash
# Create APIM instance
az apim create --name <name> --resource-group <rg> --publisher-email <email> --publisher-name <name> --sku-name Developer

# List APIM instances
az apim list --resource-group <rg>

# Import API from OpenAPI
az apim api import --path <path> --specification-url <url> --specification-format OpenApiJson

# Create product
az apim product create --product-id <id> --product-name <name> --subscription-required true

# Add API to product
az apim product api add --product-id <product-id> --api-id <api-id>

# Create subscription
az apim subscription create --product-id <product-id> --subscription-id <sub-id>

# List subscriptions
az apim subscription list

# Get developer portal URL
az apim show --name <name> --query developerPortalUrl -o tsv

# Update APIM tier
az apim update --name <name> --sku-name Standard

# Enable VNet integration (Premium only)
az apim update --name <name> --virtual-network External

# Get API gateway URL
az apim show --name <name> --query gatewayUrl -o tsv
```
Metric-based alerts
Activity log alerts
Log Analytics (KQL) alerts

APIM не обеспечивает полноценную защиту на уровне WAF и расширенную защиту от DDoS-атак.
Для этого обычно используют внешние сервисы Azure, например:
Azure Application Gateway (с WAF)
Azure Front Door
Azure DDoS Protection
---

## Learn More

- [Azure API Management Documentation](https://docs.microsoft.com/azure/api-management/)
- [API Management Policies Reference](https://docs.microsoft.com/azure/api-management/api-management-policies)
- [API Management Pricing](https://azure.microsoft.com/pricing/details/api-management/)
- [Developer Portal Overview](https://docs.microsoft.com/azure/api-management/api-management-howto-developer-portal)
