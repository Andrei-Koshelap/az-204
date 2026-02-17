# Configure Security Certificates (Настройка сертификатов безопасности)

## Key Concepts (Ключевые понятия)

- **Managed certificates** — бесплатные сертификаты с автоматическим продлением
- **App Service certificates** — приобретаются через Azure, управляются Azure, хранятся в Key Vault
- **Private certificates** — собственные сертификаты (формат PFX)
- **Public certificates** — используются для доступа к удалённым ресурсам

> 💡 В контексте AZ-204 речь идёт о TLS/SSL-сертификатах для Azure App Service.

---

## Certificate Options (Варианты сертификатов)

| Option | Cost | Management | Use Case |
|--------|------|------------|----------|
| **Free managed certificate** | Free | Fully automated | Защита custom domain |
| **App Service certificate** | Paid | Azure-managed, Key Vault | Production-сценарии с гибкостью |
| **Import from Key Vault** | Varies | Self-managed | Уже существующая инфраструктура Key Vault |
| **Upload private** | Varies | Self-managed | Сертификаты от стороннего CA |
| **Upload public** | Free | Self-managed | Доступ к внешним ресурсам |

---

## Private Certificate Requirements (Требования к приватному сертификату)

### Basic Requirements (Базовые требования)

- ✅ **PFX-файл**, защищённый паролем
- ✅ **Triple DES** шифрование
- ✅ **Private key** минимум 2 048 бит
- ✅ Полная **цепочка сертификатов** (intermediate + root)

> ⚠️ Отсутствие полной цепочки — частая причина ошибок TLS binding.

---

### TLS Binding Requirements (Для привязки к custom domain)

- ✅ **Extended Key Usage**: Server authentication  
  OID = `1.3.6.1.5.5.7.3.1`
- ✅ Подписан доверенным **Certification Authority (CA)**

---

## Free Managed Certificate (Бесплатный управляемый сертификат)

### Requirements (Требования)

- **Tier**: Basic, Standard, Premium или Isolated  
  (не поддерживается в Free / Shared)
- Может потребоваться **CAA record**:  
  `0 issue digicert.com`
- **Issuer**: DigiCert

---

### Features (Возможности)

- ✅ **Автопродление** каждые 6 месяцев  
  (обновление начинается за 45 дней до истечения)
- ✅ Полностью управляется Azure
- ✅ Поддержка TLS/SSL для сервера

---

### Limitations (Ограничения)

- ❌ Нет поддержки wildcard-сертификатов (`*.domain.com`)
- ❌ Нельзя использовать как client certificate (thumbprint-based)
- ❌ Нельзя экспортировать
- ❌ Не поддерживается в App Service Environment (ASE)
- ❌ Не работает с private DNS
- ❌ В имени домена допускаются только буквы, цифры, дефис и точка
- ❌ Максимальная длина custom domain — 64 символа

---

## Дополнительно (Что важно для AZ-204)

- TLS binding настраивается после добавления custom domain
- Для production чаще используется App Service Certificate или импорт из Key Vault
- Managed certificate — самый быстрый способ защитить сайт HTTPS
- Если используется Key Vault, нужно настроить Managed Identity для доступа

---

## Exam Focus (Что чаще всего спрашивают)

- Разница между Managed certificate и App Service certificate
- Ограничения free managed certificate
- Требования к PFX-файлу
- Когда требуется CAA record
- Поддержка wildcard-сертификатов
- Поддержка сертификатов в разных pricing tiers


```bash
# Free managed certificate is created via portal
# Configuration > Custom domains > Add binding
# Select "App Service Managed Certificate"
```

## App Service Certificate (Сертификат App Service)

### Azure Manages (Что управляется Azure)

- ✅ Процесс покупки у провайдера
- ✅ Верификация домена
- ✅ Хранение в Azure Key Vault
- ✅ Автоматическое продление
- ✅ Синхронизация с App Service приложениями

> 💡 Сертификат автоматически размещается в связанном Key Vault и может быть использован несколькими App Service.

---

### Operations (Операции)

- 🔄 **Renew (Продление)** — выполняется автоматически, но можно инициировать вручную
- 🔐 **Bind (Привязка)** — привязка сертификата к custom domain через TLS/SSL binding
- 🔁 **Rekey (Перевыпуск ключа)** — создание нового приватного ключа
- 📤 **Export (Экспорт)** — возможен при наличии доступа к Key Vault
- 🔗 **Import to App Service** — импорт сертификата из Key Vault в приложение
- 🗑 **Delete (Удаление)** — удаление сертификата и связей

---

## Дополнительно (Важно для AZ-204)

- Для доступа к сертификату из Key Vault требуется настроенная Managed Identity
- App Service Certificate поддерживает wildcard-сертификаты
- Может использоваться в production-сценариях с повышенными требованиями безопасности
- Подходит для централизованного управления сертификатами в нескольких приложениях


```bash
# Create App Service Certificate
az appservice certificate create \
  --resource-group <rg-name> \
  --name <cert-name> \
  --hostname <domain-name> \
  --key-vault <vault-name>

# Import to app
az webapp config ssl bind \
  --resource-group <rg-name> \
  --name <app-name> \
  --certificate-thumbprint <thumbprint> \
  --ssl-type SNI

# List certificates
az webapp config ssl list \
  --resource-group <rg-name>
```

### Benefits (Преимущества)

- ✅ Автоматизированное управление
- ✅ Гибкость продления
- ✅ Возможность экспорта
- ✅ Интеграция с Azure Key Vault

> ⚠️ **Важно**: Не поддерживается в Azure National Clouds

---

## Upload Private Certificate (Загрузка приватного сертификата)

Используется, если у вас уже есть сертификат от стороннего центра сертификации (CA).

### Requirements (Требования)

- 🔐 Формат **PFX**
- 🔑 Обязательно наличие **private key**
- 🔒 Минимум 2048 бит
- 🔗 Полная цепочка сертификатов (intermediate + root)
- 🔑 Защита паролем

---

### When to Use (Когда использовать)

- Сертификат приобретён вне Azure
- Используется корпоративный CA
- Нужен wildcard-сертификат
- Требуется перенос существующей инфраструктуры в Azure

---

### Important Notes (Важные моменты)

- После загрузки необходимо выполнить **TLS/SSL binding**
- Сертификат можно использовать в нескольких App Service
- Автопродление не выполняется — обновление вручную
- Можно хранить в Azure Key Vault для централизованного управления

```bash
# Upload private certificate
az webapp config ssl upload \
  --resource-group <rg-name> \
  --name <app-name> \
  --certificate-file <path-to-pfx> \
  --certificate-password <password>

# Bind certificate to custom domain
az webapp config ssl bind \
  --resource-group <rg-name> \
  --name <app-name> \
  --certificate-thumbprint <thumbprint> \
  --ssl-type SNI
```

### SSL Types (Типы SSL)

| Type | Description | Use Case |
|------|-------------|----------|
| **SNI SSL** | Server Name Indication — сертификат привязывается по имени хоста | Современные браузеры, несколько доменов на одном IP |
| **IP-based SSL** | Выделенный IP-адрес для сертификата | Старые браузеры, один домен |

> 💡 **SNI SSL** — наиболее распространённый и рекомендуемый вариант.  
> IP-based SSL увеличивает стоимость (требуется выделенный IP).

---

## Import from Key Vault (Импорт из Key Vault)

Позволяет использовать сертификат, уже хранящийся в Azure Key Vault.

### Requirements (Требования)

- 🔐 Сертификат должен быть сохранён в Key Vault
- 🔑 У App Service должна быть включена **Managed Identity**
- 📜 Должны быть выданы разрешения на доступ к секретам (Get)
- 🌍 Key Vault должен быть доступен (учитывать private endpoints / firewall)

---

### Advantages (Преимущества)

- Централизованное управление сертификатами
- Возможность использования одного сертификата в нескольких приложениях
- Поддержка автоматического продления (если сертификат управляемый)
- Повышенная безопасность (нет хранения PFX в приложении)

---

### Important for AZ-204

- Managed Identity обязательна для доступа к Key Vault
- Нужно понимать разницу между upload PFX и import from Key Vault
- Key Vault permissions — частый экзаменационный сценарий
- TLS binding всё равно требуется после импорта


```bash
# Import certificate from Key Vault
az webapp config ssl import \
  --resource-group <rg-name> \
  --name <app-name> \
  --key-vault <vault-name> \
  --key-vault-certificate-name <cert-name>
```
### Requirements (Требования для импорта из Key Vault)

- 🔐 Включённая **Managed Identity** у App Service
- 🔑 Предоставленный доступ к Key Vault (минимум permission: `Get`)
- 📜 Сертификат должен быть сохранён в Azure Key Vault

> 💡 Без корректно настроенных прав доступа App Service не сможет получить сертификат.

---

## Upload Public Certificate (Загрузка публичного сертификата)

Используется для установки **публичного (без приватного ключа)** сертификата в App Service.

### Characteristics (Особенности)

- 📄 Формат: `.cer`
- 🔓 Не содержит private key
- 🔐 Используется для доверия к внешним ресурсам
- ❌ Не может использоваться для TLS binding custom domain

---

### When to Use (Когда использовать)

- Подключение к защищённому внешнему API
- Доверие к внутреннему корпоративному CA
- Валидация исходящих HTTPS-запросов

---

### Important Notes (Важно помнить)

- Публичный сертификат не шифрует входящий трафик
- Используется только для исходящих соединений
- Не подходит для HTTPS-конфигурации веб-приложения
- Не требует TLS binding


```bash
# Upload public certificate (via portal)
# Configuration > Certificates > Public Key Certificates (.cer)

# Access in code
var cert = X509Certificate2Collection.Find(
    X509FindType.FindByThumbprint,
    "<thumbprint>",
    validOnly: false
);
```

### Use Cases (Сценарии использования)

- Вызов внешних API с аутентификацией по сертификату
- Mutual TLS (mTLS) с внешними сервисами
- ❌ Не используется для защиты custom domain

---

## Certificate Storage (Хранение сертификатов)

**Location**: Deployment unit (webspace)

- Привязка к комбинации **Resource Group + Region**
- Доступны приложениям в том же RG и регионе

> 💡 Это означает, что сертификаты нельзя использовать между разными регионами.

---

## Quick Reference (Краткая выжимка)

### Tier Requirements (Минимальный тариф)

| Feature | Minimum Tier |
|---------|--------------|
| Free managed cert | Basic |
| App Service cert | Basic |
| Custom domain SSL | Basic |
| SNI SSL | All tiers (Basic+) |
| IP-based SSL | Standard+ |

---

### Certificate Formats (Форматы сертификатов)

- **PFX** — приватные сертификаты (с паролем, содержат private key)
- **CER** — публичные сертификаты (без private key)
- **Triple DES** — требуемый алгоритм шифрования для PFX

---

## Critical Notes (Критически важные моменты)

- 💡 **Free certificates** продлеваются автоматически за 45 дней до истечения
- ⚠️ Бесплатные сертификаты имеют ограничения (нет wildcard, нельзя экспортировать)
- 🎯 **App Service certificates** автоматически хранятся в Key Vault
- 🔐 Минимальный размер private key — 2 048 бит
- 📝 **SNI SSL** предпочтительнее IP-based SSL
- ⚠️ Free tier не поддерживает custom domains и сертификаты
- 🌍 National Clouds не поддерживают App Service Certificates

---

## Exam Tips (Советы для экзамена)

- Знать ограничения free managed certificate (нет wildcard, нельзя экспортировать)
- Понимать различия между типами сертификатов и сценариями их применения
- Помнить минимальный тариф для SSL — Basic
- Знать требования к приватному сертификату (PFX, 2048-bit key)
- Понимать разницу между SNI SSL и IP-based SSL
- Бесплатные сертификаты выпускаются DigiCert
- App Service Certificates автоматически сохраняются в Key Vault


[Learn More](https://learn.microsoft.com/en-us/training/modules/configure-web-app-settings/6-configure-security-certificates)
