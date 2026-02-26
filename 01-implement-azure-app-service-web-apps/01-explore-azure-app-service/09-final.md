# 🚀 Azure App Service — Performance & Security Cheat Sheet (AZ-204)

---

# 1️⃣ Runtime и Platform

## Stack Settings
- Выбор версии: .NET, Node.js, Python, Java
- Linux: поддерживает startup command
- Контейнеры игнорируют runtime из портала

## Bitness (Windows)
- 32-bit → legacy
- 64-bit → production / больше памяти

---

# 2️⃣ Производительность

## Always On (Basic+)

✔ Предотвращает cold start  
✔ Обязателен для Continuous и CRON WebJobs  
✔ Рекомендуется для production

❌ Недоступен в Free tier

---

## ARR Affinity (Sticky Sessions)

✔ Нужен для stateful приложений  
❌ Отключать для stateless API  
❌ Отключать при использовании Redis / distributed cache

---

## HTTP/2

✔ Улучшает производительность  
✔ Работает только через HTTPS

---

## WebSockets

✔ Требуется для real-time приложений (SignalR и др.)

---

# 3️⃣ Безопасность

## HTTPS Only

✔ Всегда включать в production  
✔ Перенаправляет HTTP → HTTPS

---

## Minimum TLS

| Версия | Рекомендация |
|---------|--------------|
| TLS 1.0 / 1.1 | ❌ Не использовать |
| TLS 1.2 | ✅ Production standard |
| TLS 1.3 | ✅ Максимальная безопасность |

---

## Client Certificates (mTLS)

✔ Используется для certificate-based authentication  
✔ Работает на уровне платформы  
✔ Только inbound

---

# 4️⃣ Networking

## Inbound vs Outbound

| Inbound | Outbound |
|----------|----------|
| Access Restrictions | VNet Integration |
| Private Endpoint | Hybrid Connections |
| Service Endpoint | NAT Gateway |

---

## Частые сценарии

- Фиксированный outbound IP → NAT Gateway
- Полностью приватное приложение → Private Endpoint
- Доступ к on-prem без VPN → Hybrid Connections
- Доступ к ресурсам VNet → VNet Integration

---

# 5️⃣ Конфигурация

## App Settings
- Переменные окружения
- Encrypted at rest
- Вызывают restart при изменении
- Linux: `:` → `__`

## Slot Settings
- Не участвуют в swap
- Используются для production/staging конфигурации

## Connection Strings
- В основном для .NET
- Имеют префиксы (`SQLCONNSTR_` и др.)
- Поддерживают slot-specific

---

# 6️⃣ Deployment

## Best Practice

CI/CD → Staging Slot → Test → Swap → Production

✔ Zero downtime  
✔ Быстрый rollback (swap обратно)  
✔ Не использовать тег `latest` для контейнеров

---

# 7️⃣ Remote Debugging

✔ Поддерживается для .NET и Node.js  
⚠️ Авто-отключается через 48 часов  
❌ Не использовать в production

---

# 8️⃣ Частые ловушки AZ-204

- Plan = единица масштабирования
- Always On недоступен в Free
- ARR Affinity ухудшает балансировку нагрузки
- Slot settings не свапаются
- HTTP/2 требует TLS
- VNet Integration = outbound
- Private Endpoint = inbound
- Easy Auth ≠ Managed Identity
- Изменение настроек вызывает restart

---

# 🧠 Экзаменационная логика выбора

Если в вопросе:

- Zero-downtime → Deployment Slots
- Background jobs не работают → Always On
- Stateless API → ARR Off
- Compliance / Private access → Private Endpoint
- On-prem доступ → Hybrid Connections
- Fixed outbound IP → NAT Gateway
- Production security → HTTPS Only + TLS 1.2+

---

# 🎯 Главное правило

App Service = платформа.

Ты управляешь:
- конфигурацией
- сетью
- безопасностью
- масштабированием
  Но не VM и не ОС.

| Что нужно сделать       | Используется         |
| ----------------------- | -------------------- |
| Сменить физический хост | Redeploy             |
| Сменить регион          | Move resources       |
| Сменить подписку        | Move to subscription |
| Обновить ОС             | Update management    |



Что нужно создать?
✅ 1. An Azure Key Vault
Это хранилище секретов.

✅ 2. An access policy
Она даёт ARM template или сервису право:
читать секрет
получать пароль
Почему остальные варианты не подходят?

Azure Storage account — для файлов и blob'ов

Azure AD Identity Protection — для анализа рисков входа

Azure policy — для контроля соответствия

Backup policy — для резервного копирования

| Если в вопросе сказано            | Правильный выбор |
| --------------------------------- | ---------------- |
| Hide password in deployment       | secureString     |
| Store secret securely             | Key Vault        |
| Access secret without credentials | Managed Identity |
| Control who can read secret       | Access Policy    |

Если видишь:
Git integration
ACR
Automation
👉 ACR Tasks + Webhook trigger

В ASP.NET Core используется ILogger<T>
Типичные уровни логирования:
LogTrace
LogDebug
LogInformation
LogWarning
LogError
LogCritical


Если вопрос про:
ZIP deployment
Build automation
Same behavior as Git deployment
Ответ почти всегда:
SCM_DO_BUILD_DURING_DEPLOYMENT = true

Политики APIM позволяют:
<validate-jwt> — OAuth / Entra ID
<rate-limit-by-key> — защита от злоупотреблений
<check-header> — дополнительные проверки

Архитектурное правило APIM
Если нужно:
Контроль над API surface
Securty
Mocking
Rate limiting
👉 Используем:
Blank API
Явное описание операций
Policies