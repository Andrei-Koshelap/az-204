# Настройка General Settings в Azure App Service

## Ключевые концепции

- Позволяет настраивать **платформенные и runtime-параметры**.
- Управление **параметрами безопасности и отладки**.
- Контроль **производительности и поведения приложения** (Always On, ARR Affinity).
- Настройки доступны в: Configuration > General settings


💡 General Settings влияют на работу платформы, а не на код приложения.

---

## Stack Settings

### Настройка Runtime

- Выбор языка и версии SDK:
- .NET 8
- Node.js 18
- Python 3.11
- и другие поддерживаемые версии

- Для Linux-приложений можно указать:
- Startup command
- Startup file

- Возможна настройка специфичных параметров платформы (Windows/Linux).

---

## Важные замечания

- Изменение версии runtime может вызвать перезапуск приложения.
- Неподдерживаемая версия SDK приведёт к ошибке запуска.
- Startup command в Linux используется для кастомных сценариев запуска.
- В контейнерных приложениях runtime определяется Docker-образом.

---

## Экзаменационные ловушки (AZ-204)

- Версия runtime выбирается в General Settings, а не в Application Settings.
- Linux-приложения позволяют указать startup command.
- Контейнерные приложения игнорируют выбор runtime в портале — используется образ.
- Always On доступен не во всех tier.
- ARR Affinity управляет привязкой сессии к конкретному инстансу.

---

## Экзаменационный акцент

Если в задаче:
- приложение "засыпает" → включить Always On.
- требуется sticky session → использовать ARR Affinity.
- требуется изменить версию .NET → изменить Stack Settings.


```bash
# Set runtime stack (CLI)
az webapp config set \
  --name <app-name> \
  --resource-group <rg-name> \
  --linux-fx-version "NODE|18-lts"

# Windows app
az webapp config set \
  --name <app-name> \
  --resource-group <rg-name> \
  --net-framework-version "v8.0"
```

## Platform Settings (Платформенные настройки)

### Bitness (только для Windows)

| Опция | Когда использовать |
|--------|--------------------|
| **32-bit** | Legacy-приложения, меньшее потребление памяти |
| **64-bit** | Современные приложения, требуется больше памяти |

💡 Если приложение требует более 2 ГБ памяти — используйте 64-bit.

⚠️ В Linux этот параметр недоступен.

---

### FTP State

| Опция | Описание |
|--------|------------|
| **All allowed** | Разрешены FTP и FTPS |
| **FTPS only** | Только защищённый FTP (рекомендуется, если используется FTP) |
| **Disabled** | FTP отключён (наиболее безопасный вариант) |

💡 В production рекомендуется отключать FTP, если он не используется.

---

### HTTP Version

- **HTTP/1.1**
    - Значение по умолчанию
    - Поддерживается всеми клиентами

- **HTTP/2.0**
    - Более высокая производительность
    - Требует HTTPS/TLS
    - Поддерживает multiplexing (несколько запросов по одному соединению)

⚠️ HTTP/2 работает только при включённом HTTPS.

---

## Экзаменационные ловушки (AZ-204)

- HTTP/2 требует HTTPS.
- 64-bit предпочтителен для production-нагрузки.
- FTP лучше

- ⚠️ Most browsers only support HTTP/2 over TLS

```bash
# Enable HTTP/2
az webapp config set \
  --name <app-name> \
  --resource-group <rg-name> \
  --http20-enabled true
```

### Web Sockets
- ✅ Enable for: **ASP.NET SignalR**, **socket.io**, real-time apps
- ❌ Disable for: Standard REST APIs, static sites

```bash
# Enable Web Sockets
az webapp config set \
  --name <app-name> \
  --resource-group <rg-name> \
  --web-sockets-enabled true
```

## Always On

### Что делает Always On

- Поддерживает приложение **загруженным в памяти**, даже при отсутствии трафика.
- Предотвращает **cold start** (по умолчанию приложение выгружается через ~20 минут бездействия).
- Отправляет **GET-запрос к корню приложения** каждые 5 минут для поддержания активности.

💡 Без Always On приложение может «засыпать», что приведёт к задержке при первом запросе.

---

### Когда включать Always On

| Сценарий | Always On |
|------------|------------|
| **Continuous WebJobs** | ✅ Обязательно |
| **CRON-triggered WebJobs** | ✅ Обязательно |
| **Production-приложения** | ✅ Рекомендуется |
| **Приложения с низким трафиком** | ✅ Предотвращает выгрузку |
| **Dev/Test среда** | ❌ Необязательно |

---

## Важные замечания

- Always On доступен начиная с **Basic tier**.
- В Free и Shared tier недоступен.
- Особенно важен для фоновых задач и WebJobs.
- Увеличивает потребление ресурсов (приложение всегда активно).

---

## Экзаменационные ловушки (AZ-204)

- Если WebJob должен выполняться постоянно → включить Always On.
- Если приложение «медленно отвечает на первый запрос» → включить Always On.
- Always On недоступен в Free tier.
- Cold start возникает при отсутствии входящего трафика.

---

## Практический акцент

Если в задаче:
- фоновая задача не запускается корректно
- приложение «просыпается» слишком долго
- требуется стабильная производительность

Ответ → включить Always On.


```bash
# Enable Always On
az webapp config set \
  --name <app-name> \
  --resource-group <rg-name> \
  --always-on true
```

⚠️ **Недоступно в Free tier**

---

## ARR Affinity (Session Affinity)

### Что это такое

- Маршрутизирует клиента к **одному и тому же инстансу** на протяжении всей сессии.
- Использует **cookies** для отслеживания сессии.
- Также известно как **Sticky Sessions**.

💡 ARR (Application Request Routing) обеспечивает привязку пользователя к конкретному worker-инстансу.

---

### Когда использовать ARR Affinity

| Тип приложения | ARR Affinity |
|----------------|--------------|
| **Stateful-приложения** | ✅ Включено |
| **Приложения с сессиями** | ✅ Включено |
| **Stateless-приложения** | ❌ Выключено (лучшее распределение нагрузки) |
| **Микросервисы** | ❌ Выключено |
| **REST API** | ❌ Выключено |

---

## Важные замечания

- ARR Affinity ухудшает балансировку нагрузки при масштабировании.
- Для stateless-архитектур рекомендуется отключать.
- Если используется distributed cache (Redis) — ARR не требуется.
- Работает на уровне App Service, не требует изменений в коде.

---

## Экзаменационные ловушки (AZ-204)

- Если приложение хранит сессию в памяти → включить ARR Affinity.
- Если используется Redis или внешнее хранилище сессий → отключить ARR.
- Для микросервисной архитектуры ARR обычно отключён.
- ARR влияет только на входящий HTTP-трафик.

---

## Практический акцент

Если в задаче:
- пользователи «теряют сессию» при масштабировании → включить ARR.
- требуется равномерное распределение нагрузки → отключить ARR.
- используется stateless REST API → ARR выключен.


```bash
# Disable ARR Affinity (for stateless apps)
az webapp config set \
  --name <app-name> \
  --resource-group <rg-name> \
  --client-affinity-enabled false
```

💡 **Best Practice**: Turn OFF for stateless apps to improve load balancing

### HTTPS Only
- **Redirects all HTTP traffic** to HTTPS
- ✅ **Always enable for production**
- Required for security compliance

```bash
# Enforce HTTPS
az webapp update \
  --name <app-name> \
  --resource-group <rg-name> \
  --https-only true
```
## Минимальная версия TLS (Minimum TLS Version)

| Версия | Статус | Рекомендации |
|---------|----------|----------------|
| **TLS 1.0** | ⚠️ Устарела | Только для legacy-систем |
| **TLS 1.1** | ⚠️ Устарела | Только для legacy-систем |
| **TLS 1.2** | ✅ Рекомендуется | Стандарт для production |
| **TLS 1.3** | ✅ Актуальная | Максимальная безопасность |

---

## Важные замечания

- TLS 1.2 является минимально рекомендуемой версией для production.
- TLS 1.3 обеспечивает более быструю и безопасную установку соединения.
- Устаревшие версии TLS могут нарушать требования безопасности и compliance.
- Настройка выполняется в: Configuration > General Settings


---

## Экзаменационные ловушки (AZ-204)

- Если в задаче требуется повысить безопасность → установить минимум TLS 1.2 или 1.3.
- Если упоминается compliance или security baseline → TLS 1.0 и 1.1 недопустимы.
- HTTP/2 требует HTTPS, а значит TLS.
- TLS-настройка влияет только на входящие HTTPS-соединения.

---

## Практический акцент

Если в вопросе:
- требуется соответствие современным стандартам безопасности → выбрать TLS 1.2+.
- требуется максимальный уровень защиты → TLS 1.3.

```bash
# Set minimum TLS version
az webapp config set \
  --name <app-name> \
  --resource-group <rg-name> \
  --min-tls-version "1.2"
```

## Отладка (Debugging)

### Remote Debugging

- Доступно для:
    - **ASP.NET**
    - **ASP.NET Core**
    - **Node.js**
- Автоматически отключается через **48 часов** (мера безопасности)
- Не предназначено для production-среды

⚠️ Используйте только для временной диагностики проблем.

---

### А как насчёт Java?

Для Java в Azure App Service:

- Классическое Visual Studio Remote Debugging **не поддерживается**.
- Отладка выполняется через:
    - Java Debug Protocol (JDWP)
    - Подключение через IDE (например, IntelliJ или Eclipse)
- Требуется включение соответствующих параметров JVM.

⚠️ Remote debugging для Java требует дополнительных настроек и не включается автоматически через тот же механизм, что для .NET.

---

## Важные замечания

- Remote debugging снижает безопасность приложения.
- В production рекомендуется использовать:
    - Application Insights
    - Log Stream
    - Diagnostic logs
- Включённый remote debugging может повлиять на производительность.

---

## Экзаменационные ловушки (AZ-204)

- Remote debugging автоматически отключается через 48 часов.
- Не предназначен для production.
- Для диагностики production-проблем предпочтительнее использовать логирование.
- Поддержка remote debugging зависит от runtime.


```bash
# Enable remote debugging
az webapp config set \
  --name <app-name> \
  --resource-group <rg-name> \
  --remote-debugging-enabled true
```

## Client Certificates (Mutual TLS)

### Настройка

- Можно включить требование **клиентских сертификатов** для аутентификации.
- Используется механизм **mutual TLS (mTLS)**.
- Доступ к приложению ограничивается через проверку сертификата клиента.

---

## Как это работает

В стандартном HTTPS:

- Клиент проверяет сертификат сервера.

В mutual TLS:

- Сервер проверяет сертификат клиента.
- Клиент проверяет сертификат сервера.

Таким образом обеспечивается **двусторонняя аутентификация**.

---

## Когда использовать

Подходит для:

- Внутренних API
- B2B-интеграций
- Высоких требований безопасности
- Сценариев без OAuth / JWT

---

## Важные замечания

- Работает только по HTTPS.
- Требует корректной настройки сертификатов на стороне клиента.
- Может использоваться вместе с другими механизмами аутентификации.
- Не заменяет RBAC или Entra ID, а дополняет их.

---

## Экзаменационные ловушки (AZ-204)

- Client Certificates относятся к **входящему (inbound)** трафику.
- Mutual TLS ≠ обычный HTTPS.
- Если требуется certificate-based client authentication → включить Client Certificates.
- Работает на уровне платформы, до выполнения кода приложения.

---

## Практический акцент

Если в задаче:

- требуется проверка клиента по сертификату
- требуется повышенный уровень безопасности API
- нет использования OAuth

Ответ → включить Client Certificates (mTLS).


```bash
# Require client certificates
az webapp update \
  --name <app-name> \
  --resource-group <rg-name> \
  --client-cert-enabled true
```

### Access Certificate in Code

```csharp
// C# - Access client certificate
var cert = Request.HttpContext.Connection.ClientCertificate;
var thumbprint = cert?.Thumbprint;
```

## Быстрая справка (Quick Reference)

| Настройка | Назначение | Требуемый Tier |
|------------|------------|----------------|
| **Always On** | Поддерживает приложение активным | Basic+ |
| **ARR Affinity** | Sticky Sessions (привязка к инстансу) | Все tier |
| **HTTPS Only** | Принудительное использование HTTPS | Все tier |
| **HTTP/2** | Повышение производительности | Все tier |
| **Web Sockets** | Поддержка real-time соединений | Все tier |
| **Remote Debugging** | Удалённая отладка | Все tier |
| **Client Certificates** | Mutual TLS | Все tier |

---

## Матрица лучших практик

| Тип приложения | Always On | ARR Affinity | HTTPS Only | Минимальная версия TLS |
|----------------|-----------|--------------|------------|------------------------|
| **Production Web App** | ✅ Вкл | ⚠️ Зависит от архитектуры | ✅ Вкл | 1.2+ |
| **REST API (Stateless)** | ✅ Вкл | ❌ Выкл | ✅ Вкл | 1.2+ |
| **SignalR / Real-time** | ✅ Вкл | ✅ Вкл | ✅ Вкл | 1.2+ |
| **Background Jobs** | ✅ Вкл | ❌ Выкл | ✅ Вкл | 1.2+ |
| **Dev/Test** | ❌ Выкл | ❌ Выкл | ⚠️ Опционально | 1.2 |

---

## Критические замечания

- 💡 Always On обязателен для Continuous и CRON WebJobs.
- ⚠️ Always On недоступен в Free tier.
- 🎯 Для stateless-приложений ARR Affinity следует отключать (лучшее распределение нагрузки).
- 🔐 В production всегда включайте HTTPS Only.
- ⏰ Remote debugging автоматически отключается через 48 часов.
- 📊 HTTP/2 требует TLS — браузеры поддерживают HTTP/2 только поверх HTTPS.
- 🔄 ARR Affinity = Sticky Sessions = клиент закрепляется за одним инстансом.

---

## Экзаменационные советы (AZ-204)

- Понимайте, когда нужен Always On (WebJobs, production).
- Различайте stateful и stateless приложения при настройке ARR.
- Помните, что remote debugging отключается автоматически через 48 часов.
- HTTPS Only перенаправляет весь HTTP-трафик на HTTPS.
- HTTP/2 требует TLS.
- Для production минимальная версия TLS — 1.2.


[Learn More](https://learn.microsoft.com/en-us/training/modules/configure-web-app-settings/3-configure-general-settings)
