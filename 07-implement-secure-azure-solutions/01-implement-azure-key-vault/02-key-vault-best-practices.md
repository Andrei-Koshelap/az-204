# Лучшие практики Azure Key Vault

## Обзор

Azure Key Vault — это сервис для безопасного хранения и доступа к секретам.  
**Secret (секрет)** — любые данные, доступ к которым должен быть строго ограничен: API-ключи, пароли, сертификаты, токены, строки подключения.  
**Vault** — логическая группа секретов с централизованным управлением доступом.

---

## Методы аутентификации

Перед выполнением операций с Key Vault необходимо пройти **аутентификацию**. Существует три способа:

---

### 1. Managed Identities для ресурсов Azure ✅ **РЕКОМЕНДУЕТСЯ**

**Как работает:**

- Назначаете Managed Identity ресурсу Azure (VM, App Service, Function и т.д.)
- Azure автоматически управляет ротацией учётных данных
- Нет секретов в коде или конфигурации
- Service principal создаётся и обслуживается Azure

**Преимущества:**

- ✅ **Нет управления credential’ами** — Azure делает всё автоматически
- ✅ **Автоматическая ротация** — нет риска истечения срока действия
- ✅ **Максимальная безопасность** — нечего утечь
- ✅ **Простая интеграция** — нативная поддержка в Azure сервисах

---

### Почему это best practice

- Не нужно хранить `client secret`
- Нет необходимости использовать Azure Key Vault для хранения секретов, чтобы получить доступ к Azure Key Vault
- Минимизация человеческого фактора
- Часто это **правильный ответ в вопросах AZ-204**

---

### Когда использовать

- Azure App Service
- Azure Functions
- Azure VM
- Azure Kubernetes Service (через workload identity)
- Azure Container Apps

---

### Дополнение (что важно для экзамена)

- Managed Identity = разновидность Service Principal, управляемая Azure.
- Требует назначения роли (обычно через Azure RBAC).
- Работает только внутри Azure.
- Если приложение размещено вне Azure → потребуется Service Principal.

**Example:**
```bash
# Enable system-assigned managed identity on App Service
az webapp identity assign \
  --name myappservice \
  --resource-group myresourcegroup

# Grant Key Vault access to the managed identity
az keyvault set-policy \
  --name mykeyvault \
  --object-id <managed-identity-object-id> \
  --secret-permissions get list
```

**Code example (.NET):**
```csharp
// Managed identity automatically authenticates
var client = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net"),
    new DefaultAzureCredential()  // Uses managed identity automatically
);

var secret = await client.GetSecretAsync("MySecret");
Console.WriteLine($"Secret value: {secret.Value.Value}");
```

### 2. Service Principal + Certificate ⚠️ **ДОПУСТИМО**

**Как работает:**

- Создаётся Service Principal в Microsoft Entra ID
- К нему привязывается X.509 сертификат
- Приложение использует сертификат для аутентификации

---

### Особенности и ограничения

- ⚠️ **Ручная ротация** — сертификат нужно обновлять до истечения срока действия
- ⚠️ **Безопасное хранение** — приватный ключ должен храниться защищённо
- ✅ **Безопаснее, чем secret** — сертификаты надёжнее паролей

---

### Когда использовать

- Приложение работает вне Azure (on-premise, сторонний хостинг)
- Нельзя использовать Managed Identity
- Требуется более высокий уровень безопасности, чем при использовании client secret

---

### Дополнение (важно для AZ-204)

- Лучше использовать **сертификат**, а не client secret.
- Срок действия сертификата нужно контролировать (monitoring + alert).
- В production желательно хранить сам сертификат в:
    - Azure Key Vault
    - защищённом хранилище
- Частый экзаменационный сценарий:
  > Приложение работает вне Azure, нужен безопасный доступ к Key Vault  
  → Service Principal + Certificate.


**Example:**
```bash
# Create service principal with certificate
az ad sp create-for-rbac \
  --name myapp \
  --create-cert \
  --cert MyAppCert \
  --keyvault mykeyvault

# Application authenticates with certificate
```

**Code example (.NET):**
```csharp
var credential = new ClientCertificateCredential(
    tenantId: "your-tenant-id",
    clientId: "your-client-id",
    certificatePath: "/path/to/cert.pfx"
);

var client = new SecretClient(
    new Uri("https://mykeyvault.vault.azure.net"),
    credential
);
```

### 3. Service Principal + Secret ❌ **НЕ РЕКОМЕНДУЕТСЯ**

**Как работает:**

- Создаётся Service Principal с client secret (пароль)
- Приложение использует Client ID + Secret для аутентификации

---

### Почему стоит избегать

- ❌ **Сложная ротация** — автоматизировать обновление secret непросто
- ❌ **Bootstrap-проблема** — где хранить secret для доступа к Key Vault?
- ❌ **Разрастание секретов (secret sprawl)** — противоречит идее централизованного хранения
- ❌ **Риск безопасности** — secret можно утечь, украсть или скомпрометировать

---

### Когда допустимо использовать

- Managed Identity недоступна (on-premises, сторонние облака)
- В тестовой или dev-среде
- В краткосрочных PoC-сценариях

---

### Дополнение (что важно для AZ-204)

- Если в вопросе есть выбор:
    - Managed Identity → почти всегда правильный ответ
    - Service Principal + Certificate → допустимо
    - Service Principal + Secret → последний вариант
- Client secret — это обычный пароль.
- В production предпочтительнее:
    - Managed Identity
    - или Service Principal + Certificate


---

## Сравнение методов аутентификации

| Метод | Безопасность | Ротация | Сложность | Сценарий использования |
|--------|-------------|----------|------------|------------------------|
| **Managed Identity** | ⭐⭐⭐⭐⭐ | Автоматическая | Низкая | Ресурсы Azure (VM, App Service, Functions) |
| **Service Principal + Certificate** | ⭐⭐⭐⭐ | Ручная | Средняя | Вне Azure, cross-tenant сценарии |
| **Service Principal + Secret** | ⭐⭐ | Ручная | Средняя | Крайний случай, dev/test |

---

### Ключевые выводы

- 🥇 **Managed Identity** — самый безопасный и простой вариант.
- 🥈 **Service Principal + Certificate** — хороший компромисс, если MI недоступна.
- 🥉 **Service Principal + Secret** — использовать только при отсутствии других вариантов.

---

### Что важно для AZ-204

- Если приложение работает в Azure → выбирать **Managed Identity**.
- Если приложение вне Azure → выбирать **Service Principal + Certificate**.
- Client secret — это по сути пароль, и он требует:
    - безопасного хранения
    - регулярной ротации
- Вопросы экзамена часто проверяют выбор **наиболее безопасного способа** с минимальным управлением credential’ами.


**Decision tree:**
```
Is your application running in Azure?
├─ YES → Use Managed Identity ✅
└─ NO → Is certificate management feasible?
    ├─ YES → Use Service Principal + Certificate
    └─ NO → Use Service Principal + Secret (with caution)
```

---

## Шифрование данных при передаче (Encryption of Data in Transit)

Azure Key Vault использует протокол **Transport Layer Security (TLS)** для защиты данных, передаваемых между клиентом и сервисом.

---

## Возможности безопасности TLS

| Возможность | Описание |
|-------------|----------|
| **Протокол** | TLS 1.2+ (TLS 1.0/1.1 устарели и отключены) |
| **Аутентификация** | Надёжная взаимная аутентификация |
| **Конфиденциальность** | Шифрование трафика с использованием AES |
| **Целостность** | Обнаружение подмены, перехвата или модификации |
| **Длина ключа** | Минимум RSA 2048-bit |

---

## Perfect Forward Secrecy (PFS)

### Как это работает

- Каждое соединение использует уникальные сессионные ключи
- Компрометация одного сессионного ключа не влияет на другие
- Даже если долгосрочный приватный ключ будет скомпрометирован, предыдущие сессии останутся защищёнными

---

### Ключевые особенности PFS

- 🔑 **Уникальный ключ на каждое соединение** — отдельный ключ для каждой TLS-сессии
- ⏳ **Эфемерные ключи** — уничтожаются после завершения сессии
- 🔐 **RSA 2048-bit** — надёжное шифрование при обмене ключами

---

### Дополнение (что важно для AZ-204)

- Все взаимодействия с Key Vault происходят по HTTPS.
- TLS защищает данные **в транзите**, но не отвечает за хранение — за это отвечает шифрование на стороне сервиса.
- Если в вопросе упоминается защита данных между приложением и Key Vault → правильный ответ связан с **TLS 1.2+**.
- PFS означает, что перехват трафика сегодня не позволит расшифровать старые сессии в будущем.

**Client negotiation:**
```
Client → Server: ClientHello (supported ciphers, TLS versions)
Server → Client: ServerHello (chosen cipher, certificate)
Client → Server: Key exchange (encrypted with server's public key)
Server → Client: Server finished
[Secure encrypted communication begins]
```

**Supported cipher suites:**
- TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256
- TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384
- TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256

---

## Azure Key Vault Best Practices

### 1. Use Separate Key Vaults ✅

**Pattern: One vault per application per environment**

```
Organization
├── App1-Dev-KeyVault
├── App1-PreProd-KeyVault
├── App1-Prod-KeyVault
├── App2-Dev-KeyVault
├── App2-PreProd-KeyVault
└── App2-Prod-KeyVault
```

### Преимущества (Benefits)

- ✅ **Изоляция** — секреты не разделяются между средами (dev, test, prod)
- ✅ **Безопасность** — компрометация одного vault не влияет на другие
- ✅ **Гибкий контроль доступа** — разные роли и разрешения для каждой среды
- ✅ **Соответствие требованиям (Compliance)** — проще проводить аудит и соблюдать регуляторные нормы

---

### Дополнение (Best Practice)

Рекомендуется создавать отдельный Key Vault для каждой среды:

- `kv-app-dev`
- `kv-app-test`
- `kv-app-prod`

Это:

- снижает риск случайного использования production-секретов в dev
- упрощает управление доступом
- помогает соблюдать принцип least privilege
- часто является правильным архитектурным решением в вопросах AZ-204


**Example naming convention:**
```
{AppName}-{Environment}-kv-{Region}

Examples:
- contoso-dev-kv-eastus
- contoso-prod-kv-westus
- billing-uat-kv-northeurope
```

**Implementation:**
```bash
# Create vaults for each environment
for env in dev uat prod; do
  az keyvault create \
    --name "myapp-${env}-kv" \
    --resource-group "myapp-${env}-rg" \
    --location eastus \
    --tags Environment=$env Application=MyApp
done
```

### 2. Контролируйте доступ к вашему Vault 🔒

**Принцип наименьших привилегий (Least Privilege):**  
Предоставляйте только те разрешения, которые действительно необходимы.

---

### Роли Azure RBAC (рекомендуемый подход)

| Роль | Область | Разрешения |
|------|----------|------------|
| **Key Vault Administrator** | Полный контроль | Создание/удаление vault, управление всеми объектами |
| **Key Vault Secrets Officer** | Управление секретами | Создание, чтение, обновление, удаление секретов |
| **Key Vault Secrets User** | Только чтение | Чтение значений секретов |
| **Key Vault Crypto Officer** | Управление ключами | Создание, чтение, обновление, удаление ключей |
| **Key Vault Crypto User** | Использование ключей | Шифрование, расшифровка, подпись, проверка |
| **Key Vault Reader** | Только метаданные | Просмотр метаданных (без значений) |

---

### Best Practices

- ✅ Назначайте роли на минимально возможном уровне (resource group / vault / объект).
- ✅ Разделяйте доступ к секретам и ключам.
- ✅ Используйте Managed Identity вместо выдачи прав пользователям.
- ❌ Не назначайте Administrator без необходимости.
- ❌ Не выдавайте доступ ко всему subscription, если нужен доступ только к одному vault.

---

### Важно для AZ-204

- RBAC предпочтительнее Access Policies.
- Частая ловушка:  
  если нужно только читать секрет → выбрать **Key Vault Secrets User**, а не Administrator.
- Различайте:
    - Управление объектами (management plane)
    - Доступ к значениям секретов (data plane)

**Best practices:**
```bash
# ✅ GOOD: Grant specific role to specific principal
az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee <app-service-managed-identity-id> \
  --scope /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.KeyVault/vaults/mykeyvault

# ❌ BAD: Grant broad permissions
az keyvault set-policy \
  --name mykeyvault \
  --object-id <id> \
  --secret-permissions all  # Too permissive!
```
### Чек-лист по контролю доступа 🔐

- ✅ Использовать **Azure RBAC**, а не Access Policies
- ✅ Назначать права **Managed Identity**, а не отдельным пользователям
- ✅ Использовать **группы** для управления доступом пользователей
- ✅ Регулярно проводить аудит и пересмотр прав
- ✅ Удалять неиспользуемые разрешения
- ❌ Никогда не выдавать «полный доступ ко всему» без необходимости
- ❌ Избегать использования access keys и connection strings вместо RBAC

---

## 3. Делайте резервное копирование Vault 💾

### Зачем нужен backup

- Случайное удаление секретов, ключей или сертификатов
- Ошибки конфигурации или некорректные изменения
- Требования compliance и аудита

---

### Best Practices

- Включить **Soft Delete** (по умолчанию включён).
- Включить **Purge Protection** для production.
- Использовать встроенные команды backup/restore для:
    - Secrets
    - Keys
    - Certificates
- Хранить резервные копии в защищённом месте (например, отдельный Storage Account).

---

### Важно для AZ-204

- Soft Delete ≠ Backup, но защищает от случайного удаления.
- Purge Protection предотвращает окончательное удаление до окончания периода хранения.
- Если в вопросе говорится о защите от случайного удаления → включить Soft Delete + Purge Protection.
- Если требуется восстановление в другом регионе → использовать механизм backup/restore.


**Backup strategy:**
```bash
# Backup individual secret
az keyvault secret backup \
  --vault-name mykeyvault \
  --name MySecret \
  --file MySecret.backup

# Backup all secrets (script)
for secret in $(az keyvault secret list --vault-name mykeyvault --query "[].name" -o tsv); do
  az keyvault secret backup \
    --vault-name mykeyvault \
    --name "$secret" \
    --file "${secret}.backup"
done
```

**Restore process:**
```bash
# Restore secret from backup
az keyvault secret restore \
  --vault-name mykeyvault \
  --file MySecret.backup
```

### Лучшие практики резервного копирования (Backup Best Practices)

- ✅ **Регулярность** — выполнять резервное копирование ежедневно или еженедельно
- ✅ **Отдельное хранилище** — хранить backup в другом Azure Storage Account
- ✅ **Шифрование** — резервные копии автоматически шифруются
- ✅ **Тестовое восстановление** — периодически проверять, что restore действительно работает
- ✅ **Версионирование** — хранить несколько версий резервных копий
- ✅ **Автоматизация** — использовать Azure Automation или Logic Apps

---

### Дополнение (важно для практики и AZ-204)

- Backup/Restore выполняется на уровне объекта (secret/key/certificate).
- Для production рекомендуется:
    - отдельный subscription или resource group для хранения backup
    - ограниченный доступ к storage с backup-файлами
- Soft Delete защищает от случайного удаления,  
  но полноценный backup нужен для:
    - восстановления в другой регион
    - миграции
    - disaster recovery сценариев
- Частый экзаменационный сценарий:
  > Нужно защититься от случайного удаления и обеспечить восстановление  
  → включить Soft Delete + Purge Protection + настроить регулярный backup.

**Automated backup example:**
```json
// Logic App trigger: Daily at 2 AM
{
  "schedule": {
    "frequency": "Day",
    "interval": 1,
    "timeZone": "UTC",
    "startTime": "2026-01-01T02:00:00Z"
  },
  "actions": {
    "BackupSecrets": {
      "type": "AzureKeyVault.BackupSecret",
      "inputs": {
        "vaultName": "mykeyvault",
        "storageAccount": "backupstorage"
      }
    }
  }
}
```

### 4. Включите логирование 📊

### Почему логирование важно

- 🔐 **Аудит безопасности** — кто, к чему и когда получил доступ
- 🛠 **Диагностика проблем доступа** — ошибки авторизации, отказ в доступе
- 📋 **Соответствие требованиям (Compliance)** — SOC 2, ISO 27001, HIPAA
- 🚨 **Обнаружение аномалий** — подозрительная активность, массовые запросы

---

### Что рекомендуется включить

- **Diagnostic settings** для Key Vault
- Логи операций с:
    - Secrets
    - Keys
    - Certificates
- Логи аутентификации и авторизации

---

### Куда отправлять логи

- **Azure Monitor Logs** — анализ через KQL
- **Storage Account** — долгосрочное хранение
- **Event Hub** — интеграция с SIEM

---

### Best Practices

- ✅ Включать логирование для всех production vault
- ✅ Настроить alert’ы на:
    - множественные ошибки доступа
    - частые попытки аутентификации
    - операции purge/delete
- ✅ Ограничить доступ к логам (они могут содержать чувствительную информацию)

---

### Важно для AZ-204

- Логирование настраивается через **Diagnostic Settings**.
- Частый вопрос:
  > Нужно отслеживать доступ к секретам  
  → включить Diagnostic logs и отправить в Azure Monitor.
- Для compliance почти всегда требуется аудит доступа к секретам.


**Enable diagnostic logs:**
```bash
# Create Log Analytics workspace (if needed)
az monitor log-analytics workspace create \
  --resource-group myresourcegroup \
  --workspace-name mylogworkspace

# Enable Key Vault logging
az monitor diagnostic-settings create \
  --name KeyVaultAuditLogs \
  --resource $(az keyvault show --name mykeyvault --query id -o tsv) \
  --logs '[
    {
      "category": "AuditEvent",
      "enabled": true,
      "retentionPolicy": {
        "enabled": true,
        "days": 90
      }
    },
    {
      "category": "AzurePolicyEvaluationDetails",
      "enabled": true
    }
  ]' \
  --metrics '[
    {
      "category": "AllMetrics",
      "enabled": true
    }
  ]' \
  --workspace $(az monitor log-analytics workspace show --resource-group myresourcegroup --workspace-name mylogworkspace --query id -o tsv)
```

### Назначения логов (Log Destinations)

| Назначение | Сценарий использования | Стоимость |
|------------|------------------------|-----------|
| **Log Analytics** | Запросы, анализ, создание alert’ов | Средняя |
| **Storage Account** | Долгосрочное архивное хранение | Низкая |
| **Event Hub** | Потоковая передача в SIEM (Splunk, QRadar) | Средняя |

---

### Как выбрать

- 🔎 Нужен анализ и алерты → **Log Analytics**
- 🗄 Нужно просто хранить для аудита → **Storage Account**
- 🛡 Используется внешняя SIEM-система → **Event Hub**

---

### Best Practices

- Для production часто используют комбинацию:
    - Log Analytics (операционный мониторинг)
    - Storage Account (архив)
- Настраивать retention policy в Log Analytics.
- Ограничивать доступ к логам (они могут содержать чувствительные метаданные).

---

### Важно для AZ-204

- Логирование включается через **Diagnostic Settings**.
- Если требуется мониторинг и alerting → выбирать **Log Analytics**.
- Если требуется интеграция с SIEM → выбирать **Event Hub**.


**Key metrics to monitor:**

```kql
// Failed authentication attempts
AzureDiagnostics
| where ResourceType == "VAULTS"
| where ResultType == "Unauthorized"
| summarize FailedAttempts = count() by CallerIPAddress, TimeGenerated
| where FailedAttempts > 10
```

```kql
// Successful secret access
AzureDiagnostics
| where ResourceType == "VAULTS"
| where OperationName == "SecretGet"
| where ResultType == "Success"
| summarize AccessCount = count() by identity_claim_upn_s, Resource, bin(TimeGenerated, 1h)
```

**Set up alerts:**
```bash
# Alert on multiple failed attempts
az monitor metrics alert create \
  --name "KeyVault-FailedAuth" \
  --resource-group myresourcegroup \
  --scopes $(az keyvault show --name mykeyvault --query id -o tsv) \
  --condition "avg Availability < 99" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --action email notify-team@contoso.com
```

### 5. Enable Recovery Options 🔄

#### Soft Delete (Enabled by Default)

**What it does:**
- Deleted secrets/keys/certificates retained for 7-90 days
- Vault itself retained if deleted
- Can be recovered during retention period

**Configuration:**
```bash
# Soft delete is enabled by default with 90-day retention
# Update retention period (7-90 days)
az keyvault update \
  --name mykeyvault \
  --retention-days 90
```

**Recovery process:**
```bash
# List deleted secrets
az keyvault secret list-deleted --vault-name mykeyvault

# Recover deleted secret
az keyvault secret recover \
  --vault-name mykeyvault \
  --name MyDeletedSecret

# Permanently delete (purge) - only if purge protection is off
az keyvault secret purge \
  --vault-name mykeyvault \
  --name MyDeletedSecret
```

#### Purge Protection (Рекомендуется для Production)

### Что делает

- 🔒 Предотвращает **окончательное удаление** объектов в период Soft Delete
- 🚫 Даже администраторы не могут выполнить purge удалённых объектов
- ⏳ Обязательный период ожидания перед возможностью полного удаления

---

### Как это работает

1. Объект (secret/key/certificate) удаляется → переходит в состояние *soft-deleted*.
2. В течение retention-периода (до 90 дней) его можно восстановить.
3. Если включена **Purge Protection**, принудительное удаление (purge) невозможно до окончания этого периода.

---

### Зачем это нужно

- Защита от:
    - случайного удаления
    - злонамеренных действий
    - атак с повышением привилегий
- Требование многих стандартов compliance (финансовый сектор, healthcare и др.)

---

### Важно для AZ-204

- Soft Delete включён по умолчанию.
- Purge Protection нужно включать отдельно.
- В production-сценариях почти всегда правильный ответ:
  → включить **Soft Delete + Purge Protection**.
- Если в вопросе говорится о защите от администраторов или внутренних угроз → это про Purge Protection.

**Enable purge protection:**
```bash
# Enable during vault creation (cannot be disabled later!)
az keyvault create \
  --name mykeyvault \
  --resource-group myresourcegroup \
  --enable-purge-protection true \
  --retention-days 90

# Enable on existing vault (IRREVERSIBLE!)
az keyvault update \
  --name mykeyvault \
  --enable-purge-protection true
```

⚠️ **Warning**: Once purge protection is enabled, it **cannot be disabled**. Plan carefully before enabling.

**Recovery scenario with purge protection:**

```
Day 0: Secret deleted
       ↓
Days 1-90: Secret in soft-deleted state
           ↓ Can be recovered
           └─ Cannot be purged (protected)
       ↓
Day 91: Soft-delete retention expires
       ↓
       Automatic permanent deletion
```

### Лучшие практики (Purge Protection)

- ✅ Включать для **production**-сред
- ✅ Включать при требованиях compliance (GDPR, HIPAA)
- ⚠️ Сначала протестировать влияние в dev/test (отключить нельзя)
- ❌ Не требуется для краткоживущих dev/test vault

---

# Security Checklist

## Этап настройки (Setup Phase)

- ✅ Создавать отдельный vault для каждого приложения и каждой среды
- ✅ Включить Soft Delete (включён по умолчанию)
- ✅ Включить Purge Protection для production
- ✅ Настроить диагностическое логирование
- ✅ Создать Log Analytics workspace

---

## Контроль доступа (Access Control)

- ✅ Использовать Managed Identity для ресурсов Azure
- ✅ Использовать Azure RBAC (а не Access Policies)
- ✅ Следовать принципу наименьших привилегий
- ✅ Назначать права группам, а не отдельным пользователям
- ✅ Регулярно пересматривать назначения ролей

---

## Сетевая безопасность (Network Security)

- ✅ Использовать Private Endpoints для production
- ✅ Настроить firewall, если используется public endpoint
- ✅ Отключить публичный доступ, если он не нужен
- ✅ Использовать Service Endpoints для ресурсов в VNet

---

## Операционная деятельность (Operations)

- ✅ Реализовать стратегию резервного копирования (рекомендуется ежедневно)
- ✅ Регулярно тестировать восстановление
- ✅ Мониторить логи на подозрительную активность
- ✅ Настроить alert’ы на ошибки аутентификации
- ✅ Регулярно ротировать секреты
- ✅ Документировать владельца и назначение каждого секрета

---

## Соответствие требованиям (Compliance)

- ✅ Включить аудит логирования
- ✅ Хранить логи согласно регуляторным требованиям (90+ дней)
- ✅ Использовать HSM-ключи при необходимости compliance
- ✅ Настроить политики ротации ключей
- ✅ Документировать меры безопасности

---

# Частые ошибки (Common Pitfalls)

| Ошибка | Риск | Решение |
|--------|------|----------|
| Один vault для нескольких приложений | Разрастание секретов, неясная ответственность | Отдельный vault на приложение |
| Назначение «всех прав» | Избыточные привилегии | Назначать конкретные роли |
| Игнорирование логов | Пропущенные инциденты | Включить логирование и alert’ы |
| Нет стратегии backup | Потеря данных | Регулярный автоматизированный backup |
| Хардкод credential’ов | Утечка секретов | Использовать Managed Identity |
| Public endpoint без firewall | Несанкционированный доступ | Private Endpoint или firewall |
| Нет Soft Delete/Purge Protection | Безвозвратное удаление | Включить оба для production |
| Использование Access Policies | Несогласованная модель прав | Использовать Azure RBAC |

---

# Exam Tips (Советы к AZ-204)

🎯 **Managed Identity** — основной рекомендуемый способ аутентификации в Azure

🎯 **Три метода аутентификации**:
- Managed Identity (лучший)
- Service Principal + Certificate (допустимо)
- Service Principal + Secret (избегать)

🎯 **TLS** — шифрование данных в транзите через TLS 1.2+

🎯 **Perfect Forward Secrecy (PFS)** — уникальные ключи для каждой сессии, минимум RSA 2048-bit

🎯 **Отдельные vault** — один vault на приложение и среду

🎯 **Azure RBAC** — предпочтительнее Access Policies

🎯 **Soft Delete** — включён по умолчанию (7–90 дней, по умолчанию 90)

🎯 **Purge Protection** — предотвращает окончательное удаление, нельзя отключить после включения

🎯 **Назначения логов**:
- Storage Account — архив
- Event Hub — потоковая передача
- Log Analytics — анализ

🎯 **Backup** — выполняется на уровне отдельных объектов (secrets, keys, certificates), а не всего vault

🎯 **Принцип безопасности** — Least Privilege

🎯 **Bootstrap-проблема** — не использовать secret для доступа к Key Vault → использовать Managed Identity


---

## Additional Resources

- [Key Vault Best Practices](https://learn.microsoft.com/en-us/azure/key-vault/general/best-practices)
- [Soft Delete Overview](https://learn.microsoft.com/en-us/azure/key-vault/general/soft-delete-overview)
- [Logging and Monitoring](https://learn.microsoft.com/en-us/azure/key-vault/general/logging)
- [Azure RBAC for Key Vault](https://learn.microsoft.com/en-us/azure/key-vault/general/rbac-guide)

[Microsoft Learn - Discover Azure Key Vault best practices](https://learn.microsoft.com/en-us/training/modules/implement-azure-key-vault/3-key-vault-concepts)
