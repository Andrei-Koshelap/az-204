# Изучение Azure Key Vault

## Обзор

**Azure Key Vault** — это облачный сервис для безопасного хранения и доступа к секретам, ключам и сертификатам. Он устраняет необходимость хранить чувствительные данные в коде приложения и обеспечивает централизованное управление секретами с жёстким контролем доступа.

---

## Что такое Azure Key Vault?

Azure Key Vault решает три ключевые задачи:

| Проблема | Решение |
|----------|----------|
| **Управление секретами (Secrets Management)** | Безопасное хранение и контроль доступа к токенам, паролям, сертификатам, API-ключам и другим секретам |
| **Управление ключами (Key Management)** | Создание и контроль ключей шифрования для защиты данных |
| **Управление сертификатами (Certificate Management)** | Выпуск, хранение и автоматическое управление SSL/TLS сертификатами |

---

## Типы Key Vault

Azure Key Vault поддерживает два типа контейнеров:

### 1. Vaults (Standard)

- **Хранение**: ключи (software и HSM-backed), секреты, сертификаты
- **Защита**: программное шифрование
- **Сценарий**: большинство приложений
- **Стоимость**: более экономичный вариант

### 2. Managed HSM Pools (Premium)

- **Хранение**: только HSM-backed ключи
- **Защита**: аппаратный модуль безопасности (HSM)
- **Сценарий**: регуляторные требования, повышенная безопасность
- **Стоимость**: выше, соответствует FIPS 140-2 Level 3

---

## Сравнение уровней сервиса

| Возможность | Standard | Premium |
|-------------|----------|----------|
| **Защита ключей** | Software-protected | HSM-protected |
| **Шифрование** | AES 256-bit | AES 256-bit |
| **FIPS соответствие** | FIPS 140-2 Level 1 | FIPS 140-2 Level 2 (Vaults)<br>FIPS 140-2 Level 3 (Managed HSM) |
| **Типы ключей** | RSA, EC | RSA, EC, OCT (только Managed HSM) |
| **Ценообразование** | За операцию | За операцию + плата за HSM |
| **Лучше всего подходит** | Большинство приложений | Регуляторные требования, high-security |

---

## Основные преимущества Azure Key Vault

### 1. Централизованное хранение секретов

✅ **Единый источник истины (Single source of truth)**

- Хранение connection strings, API-ключей и паролей централизованно
- Приложения обращаются к секретам по URI вместо хардкода
- Возможность получать конкретную версию секрета
- Ротация секретов без изменения кода

**Пример формата URI:**
```
    https://{vault-name}.vault.azure.net/secrets/{secret-name}/{version}
```

---

### 2. Безопасное хранение секретов и ключей

✅ **Многоуровневая безопасность**

- **Аутентификация**: Microsoft Entra ID
- **Авторизация**:
    - Azure RBAC (рекомендуется)
    - Access Policies (устаревающий подход)
- **Шифрование**:
    - Standard — программно защищённые ключи
    - Premium — HSM (FIPS 140-2 Level 2)

| Метод авторизации | Management Plane | Data Plane |
|------------------|------------------|------------|
| **Azure RBAC** | ✅ Поддерживается | ✅ Поддерживается |
| **Access Policies** | ❌ Нет | ✅ Да |

**Best practice**: использовать Azure RBAC для единообразной модели доступа.

---

### 3. Мониторинг доступа

✅ **Полное логирование**

- Включение логирования для всех vault
- Аудит доступа к секретам
- Отслеживание изменений

**Варианты назначения логов:**

| Назначение | Цель |
|------------|------|
| **Storage Account** | Долгосрочное хранение |
| **Event Hub** | Интеграция с SIEM |
| **Azure Monitor Logs** | Анализ через KQL |

**Примеры метрик:**

- Общее количество API-запросов
- Неудачные попытки аутентификации
- Получение секретов
- Средняя задержка

---

### 4. Упрощённое администрирование

✅ **Снижение операционной сложности**

| Традиционный подход | С Azure Key Vault |
|---------------------|-------------------|
| Покупка и поддержка HSM | Azure управляет HSM |
| Ручное масштабирование | Автомасштабирование |
| Ручная репликация | Автоматическая региональная репликация |
| Ручной failover | Автоматический failover |
| Сложный lifecycle сертификатов | Автоматическое продление |

**Ключевые упрощения:**

- Не требуется знание HSM
- Автоматическая обработка пиков нагрузки
- Высокая доступность
- Автоматическое продление сертификатов
- Управление через Portal, CLI или PowerShell

---

### Дополнение от себя (что важно для AZ-204)

- Никогда не хранить секреты в `appsettings.json` или в коде.
- Использовать **Managed Identity** для доступа к Key Vault из Azure сервисов.
- В production включать:
    - Soft Delete
    - Purge Protection
- Для CI/CD лучше использовать Key Vault references или Azure App Configuration + Key Vault.
- Вопросы на экзамене часто проверяют:
    - RBAC vs Access Policies
    - Managed Identity
    - Разницу Standard vs Premium
    - Логирование и аудит доступа

## How Key Vault Works

### Architecture

```
┌─────────────────────────────────────────────────────┐
│                    Application                       │
│  ┌─────────────────────────────────────────────┐   │
│  │   Azure Identity (DefaultAzureCredential)    │   │
│  └─────────────────┬───────────────────────────┘   │
└────────────────────┼─────────────────────────────────┘
                     │
                     │ Authenticate (Microsoft Entra ID)
                     ▼
┌─────────────────────────────────────────────────────┐
│              Microsoft Entra ID                      │
│         (Authentication & Authorization)             │
└─────────────────┬───────────────────────────────────┘
                  │
                  │ Access Token
                  ▼
┌─────────────────────────────────────────────────────┐
│              Azure Key Vault                         │
│  ┌───────────────────────────────────────────────┐ │
│  │  Secrets  │  Keys  │  Certificates             │ │
│  └───────────────────────────────────────────────┘ │
│  ┌───────────────────────────────────────────────┐ │
│  │  Audit Logs → Azure Monitor / Storage / Event │ │
│  └───────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

### Поток доступа (Access Flow)

1. **Приложение запрашивает доступ** → Использует Managed Identity или Service Principal
2. **Microsoft Entra ID выполняет аутентификацию** → Проверяет личность
3. **Azure RBAC выполняет авторизацию** → Проверяет разрешения
4. **Key Vault возвращает секрет** → Приложение получает значение секрета
5. **Операция логируется** → Формируется audit trail

---

## Что можно хранить в Azure Key Vault?

### 1. Secrets (Секреты)

**Что это**: Любые чувствительные строковые данные

**Примеры:**
- Строки подключения к БД
- API-ключи
- Пароли
- SAS-токены
- Ключи Storage Account

**Ограничение размера**: до 25 KB на один секрет

---

### Дополнение (важно для AZ-204 и практики)

- Секреты имеют версии — можно выполнять безопасную ротацию.
- Поддерживается Soft Delete (защита от случайного удаления).
- В production рекомендуется включать:
    - Soft Delete
    - Purge Protection
- Лучше использовать Managed Identity вместо хранения client secrets.
- Никогда не хранить секреты в коде или в репозитории.


**Example:**
```bash
az keyvault secret set \
  --vault-name mykeyvault \
  --name "DatabaseConnection" \
  --value "Server=myserver;Database=mydb;User=admin;Password=secret123"
```

### 2. Keys

**What**: Cryptographic keys for encryption/decryption

**Key types:**
- **RSA**: 2048-bit, 3072-bit, 4096-bit
- **EC (Elliptic Curve)**: P-256, P-384, P-521, P-256K
- **OCT (Symmetric)**: 128-bit, 192-bit, 256-bit (Managed HSM only)

**Operations:**
- Encrypt/Decrypt
- Sign/Verify
- Wrap/Unwrap (key wrapping)

**Example:**
```bash
az keyvault key create \
  --vault-name mykeyvault \
  --name "MyEncryptionKey" \
  --protection software \
  --kty RSA \
  --size 2048
```

### 3. Certificates (Сертификаты)

**Что это**: Сертификаты X.509 (SSL/TLS)

Используются для:
- HTTPS
- Шифрования трафика
- Аутентификации сервисов
- Mutual TLS (mTLS)

---

### Возможности

- ✅ Автоматическое управление жизненным циклом сертификата
- ✅ Интеграция с центрами сертификации (Certificate Authorities, CA)
- ✅ Автоматическое продление (для поддерживаемых CA)
- ✅ Поддержка форматов PFX и PEM

---

### Поддерживаемые центры сертификации (CA)

- DigiCert
- GlobalSign
- Self-signed сертификаты

---

### Дополнение (что важно для AZ-204)

- Key Vault может автоматически запрашивать и продлевать сертификаты у поддерживаемых CA.
- Сертификат хранится как:
    - Certificate (метаданные)
    - Secret (PFX/PEM содержимое)
    - Key (закрытый ключ)
- Частый сценарий: хранение SSL-сертификатов для App Service или Application Gateway.
- Для production рекомендуется включать уведомления о скором истечении срока действия сертификата.

**Example:**
```bash
az keyvault certificate create \
  --vault-name mykeyvault \
  --name "MyCertificate" \
  --policy @policy.json
```

---

## Региональная доступность и репликация

### Репликация данных

| Тип | Поведение |
|------|-----------|
| **Primary region** | Все данные реплицируются внутри региона |
| **Secondary region** | Автоматическая репликация в парный регион Azure |
| **Failover** | Выполняется автоматически (без действий администратора) |
| **Доступ на чтение** | Только из primary (до момента failover) |

**Примеры парных регионов:**
- East US ↔ West US
- North Europe ↔ West Europe
- Southeast Asia ↔ East Asia

---

### Высокая доступность (High Availability)

- **RPO (Recovery Point Objective)**: минуты
- **RTO (Recovery Time Objective)**: автоматический failover
- **Надёжность хранения (Data durability)**: 99.999999999% (11 девяток)
- **SLA сервиса**: 99.99% доступности

---

### Дополнение (что важно для AZ-204)

- Репликация выполняется автоматически — вручную настраивать её не нужно.
- Secondary регион активируется только при сбое primary.
- Key Vault обеспечивает региональную избыточность без дополнительной конфигурации.
- В экзаменационных вопросах часто проверяют:
    - понимание RPO vs RTO
    - что failover автоматический
    - что чтение выполняется только из primary до момента переключения
    - различие SLA сервиса и durability хранения

---

## Common Use Cases

### 1. Secure Application Configuration

**Before (Insecure):**
```csharp
// ❌ Hardcoded in appsettings.json
{
  "ConnectionStrings": {
    "Database": "Server=sql.database.windows.net;Password=MySecret123"
  }
}
```

**After (Secure with Key Vault):**
```csharp
// ✅ Reference from Key Vault
{
  "ConnectionStrings": {
    "Database": "@Microsoft.KeyVault(SecretUri=https://mykv.vault.azure.net/secrets/DbConnection)"
  }
}
```

### 2. Encryption Key Management

```csharp
// Encrypt data with Key Vault key
var keyClient = new KeyClient(new Uri("https://mykv.vault.azure.net"), new DefaultAzureCredential());
var key = await keyClient.GetKeyAsync("MyEncryptionKey");

var cryptoClient = new CryptographyClient(key.Value.Id, new DefaultAzureCredential());
byte[] plaintext = Encoding.UTF8.GetBytes("Sensitive data");
var encryptResult = await cryptoClient.EncryptAsync(EncryptionAlgorithm.RsaOaep, plaintext);
```

### 3. Управление жизненным циклом сертификатов (Certificate Lifecycle Management)

- Развёртывание SSL/TLS сертификатов в сервисы Azure
- Автоматическое продление сертификатов до истечения срока действия
- Централизованный реестр всех сертификатов
- Отслеживание сроков действия сертификатов

---

### Что это даёт на практике

- Исключает ручное обновление сертификатов
- Снижает риск простоя из-за просроченного SSL
- Позволяет управлять всеми сертификатами из одного места
- Упрощает аудит и соответствие требованиям безопасности

---

### Важно для AZ-204

- Key Vault может автоматически продлевать сертификаты у поддерживаемых CA.
- Сертификаты можно привязывать к:
    - Azure App Service
    - Application Gateway
    - Azure Front Door
- Частый экзаменационный сценарий:  
  требуется автоматическое продление SSL без хранения приватного ключа в коде → использовать Key Vault.

---

## Возможности безопасности (Security Features)

### Варианты аутентификации (Authentication Options)

1. **Managed Identity** (Рекомендуется)
    - Нет учётных данных в коде
    - Автоматическая ротация credential’ов
    - Работает нативно с Azure сервисами
    - Идеально для App Service, Azure Functions, VM, AKS

2. **Service Principal**
    - Client ID + Certificate (предпочтительный вариант)
    - Client ID + Secret (менее безопасно)
    - Используется для внешних сервисов или CI/CD

3. **User Identity**
    - Для интерактивных сценариев
    - Аутентификация через Azure CLI / PowerShell
    - Удобно для разработки и администрирования

---

## Модели авторизации (Authorization Models)

### Azure RBAC (Рекомендуется)

| Роль | Разрешения |
|------|------------|
| **Key Vault Administrator** | Полный доступ к Key Vault и всем объектам |
| **Key Vault Secrets Officer** | Полный доступ к секретам |
| **Key Vault Secrets User** | Только чтение секретов |
| **Key Vault Crypto Officer** | Полный доступ к ключам |
| **Key Vault Crypto User** | Использование ключей для криптоопераций |
| **Key Vault Certificates Officer** | Управление сертификатами |
| **Key Vault Reader** | Чтение метаданных (без значений секретов) |

---

### Access Policies (Устаревающая модель)

- **Permissions**: назначаются по типу объекта (secrets, keys, certificates)
- **Гранулярность**: на пользователя / service principal
- **Ограничение**: не управляет Management Plane

> ✅ Best practice: использовать Azure RBAC для унифицированной модели безопасности.

---

## Сетевые возможности и защита (Networking and Security)

### Варианты сетевого доступа

| Опция | Описание | Сценарий |
|--------|-----------|-----------|
| **Public endpoint** | Доступ из интернета | По умолчанию, большинство приложений |
| **Service endpoints** | Ограничение доступа из Azure VNet | Внутренние ресурсы Azure |
| **Private endpoints** | Приватный IP внутри VNet | Без доступа из интернета |
| **Firewall rules** | Разрешённые/запрещённые IP | Контроль конкретных диапазонов |

**Best practice**: для production использовать Private Endpoints.

---

## Soft Delete и Purge Protection

| Возможность | Назначение | По умолчанию |
|-------------|------------|--------------|
| **Soft Delete** | Хранение удалённых объектов 7–90 дней | Включено (90 дней) |
| **Purge Protection** | Запрет окончательного удаления в период хранения | Опционально (рекомендуется) |

---

### Дополнение (что часто спрашивают на AZ-204)

- Managed Identity — лучший вариант для Azure сервисов.
- RBAC предпочтительнее Access Policies.
- В production:
    - включать Soft Delete
    - включать Purge Protection
    - использовать Private Endpoints
- Разница:
    - Authentication = кто вы?
    - Authorization = что вам разрешено?
- Частый сценарий вопроса:
  > Нужно безопасно хранить connection string и не хранить секрет в коде  
  → Использовать Key Vault + Managed Identity.


**Recovery process:**
```bash
# List deleted vaults
az keyvault list-deleted

# Recover deleted vault
az keyvault recover --name mykeyvault

# Recover deleted secret
az keyvault secret recover --vault-name mykeyvault --name MySecret
```

---
## Ценообразование (Pricing)

### Standard Tier

| Операция | Стоимость (примерно) |
|-----------|----------------------|
| Операции с секретами | $0.03 за 10 000 транзакций |
| Операции с ключами (software) | $0.03 за 10 000 транзакций |
| Операции с сертификатами | $3.00 за продление |
| Управляемые ключи Storage Account | $3.00 за аккаунт в месяц |

---

### Premium Tier

- **Все расходы Standard** +
- **HSM-защищённые ключи**: ~$1.00 за ключ в месяц
- **Операции HSM**: более высокая стоимость за транзакцию

---

### Managed HSM

- **Стоимость пула**: ~$4.00 в час (за один HSM)
- **Высокая доступность**: минимум 3 HSM-реплики
- **Месячная стоимость**: ~$3 000+ в месяц

💡 **Совет по стоимости**:  
Для большинства нагрузок достаточно Standard.  
Premium и Managed HSM используются только при требованиях регуляторов и повышенной безопасности.

---

## Exam Tips (Советы к AZ-204)

🎯 **Два типа контейнеров**:
- Vaults (наиболее распространённый вариант)
- Managed HSM Pools

🎯 **Три типа объектов**:
- Secrets
- Keys
- Certificates

🎯 **Уровни сервиса**:
- Standard — программная защита ключей
- Premium — HSM-защита

🎯 **Аутентификация**:
- Требуется Microsoft Entra ID

🎯 **Авторизация**:
- Azure RBAC (рекомендуется)
- Access Policies (устаревающий подход)

🎯 **Managed Identity**:
- Лучшая практика для аутентификации
- Нет хранения credential’ов

🎯 **Soft delete**:
- Включён по умолчанию (хранение 90 дней)

🎯 **Региональная репликация**:
- Автоматическая внутри региона и в парный регион

🎯 **Логирование**:
- Поддержка Storage Account
- Event Hub
- Azure Monitor Logs

🎯 **Формат URI**:
```bash
https://{vault-name}.vault.azure.net/{object-type}/{object-name}
```

🎯 **Ограничения размера**:
- Secrets — до 25 KB
- Keys и Certificates — без строгого лимита размера (но с ограничениями по типу ключа)

🎯 **Высокая доступность**:
- SLA 99.99%
- Автоматический failover

---

### Что часто проверяют на экзамене

- Когда выбирать Standard vs Premium.
- Разницу Vault vs Managed HSM.
- RBAC vs Access Policies.
- Managed Identity как лучший способ доступа.
- Soft Delete + Purge Protection в production.
- Формат URI и ограничения размера секрета.


## Quick Reference Commands

### Create Key Vault
```bash
az keyvault create \
  --name mykeyvault \
  --resource-group myresourcegroup \
  --location eastus \
  --sku standard
```

### Set Secret
```bash
az keyvault secret set \
  --vault-name mykeyvault \
  --name "MySecret" \
  --value "MySecretValue"
```

### Get Secret
```bash
az keyvault secret show \
  --vault-name mykeyvault \
  --name "MySecret" \
  --query value -o tsv
```

### Create Key
```bash
az keyvault key create \
  --vault-name mykeyvault \
  --name "MyKey" \
  --protection software
```

### Enable Logging
```bash
az monitor diagnostic-settings create \
  --name KeyVaultLogs \
  --resource $(az keyvault show --name mykeyvault --query id -o tsv) \
  --logs '[{"category":"AuditEvent","enabled":true}]' \
  --workspace myworkspace
```

---

## Additional Resources

- [Azure Key Vault Documentation](https://learn.microsoft.com/en-us/azure/key-vault/)
- [Azure Key Vault Pricing](https://azure.microsoft.com/pricing/details/key-vault/)
- [Key Vault Developer's Guide](https://learn.microsoft.com/en-us/azure/key-vault/general/developers-guide)
- [Best Practices for Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/best-practices)

[Microsoft Learn - Explore Azure Key Vault](https://learn.microsoft.com/en-us/training/modules/implement-azure-key-vault/2-key-vault-overview)
