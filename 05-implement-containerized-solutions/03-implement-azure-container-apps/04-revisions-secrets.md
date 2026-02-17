# Управление ревизиями и секретами в Azure Container Apps

## Ключевые понятия

- **Revisions (Ревизии)** — неизменяемые снимки версии контейнерного приложения
- **Изменения уровня ревизии (Revision-scope changes)** — изменения, которые приводят к созданию новой ревизии
- **Traffic splitting (Разделение трафика)** — распределение входящего трафика между несколькими ревизиями
- **Secrets (Секреты)** — безопасное хранение конфиденциальных настроек и данных

> 💡 Важно для экзамена AZ-204: нужно чётко понимать, какие изменения создают новую ревизию, а какие — нет.

---

## Revisions (Ревизии)

### Что такое ревизии?

**Immutable snapshots** — это неизменяемые снимки версии контейнерного приложения в Azure Container Apps.

Основные характеристики:

- Создаются автоматически при изменениях уровня ревизии
- После создания не могут быть изменены
- Позволяют реализовать версионирование и откат (rollback)
- Поддерживают распределение трафика между версиями
- Можно задавать собственные имена ревизий

---

### Дополнение (что важно понимать глубже)

Ревизия создаётся при изменениях, влияющих на конфигурацию контейнера, например:

- Изменение образа контейнера (image)
- Изменение переменных окружения
- Изменение ресурсов CPU/Memory
- Изменение настроек масштабирования (scaling rules)

⚠️ Но изменения уровня приложения (например, метаданные или настройки ingress без влияния на контейнер) могут не создавать новую ревизию.

---

### Практическое применение

Ревизии позволяют:

- Реализовать **blue-green deployment**
- Делать **canary deployments**
- Постепенно переключать трафик на новую версию
- Быстро откатываться к предыдущей стабильной версии

Это делает Azure Container Apps удобным инструментом для безопасного CI/CD.

---

Если нужно, присылай следующий блок — продолжим перевод в том же формате.

### Revision Lifecycle
```
Create App → Deploy (Revision 1)
           ↓
Update Configuration → Revision 2 (new)
           ↓
Traffic Split: 80% Rev 1, 20% Rev 2
           ↓
Full Rollout → 100% Rev 2
           ↓
Deactivate Rev 1
```

### Изменения уровня ревизии (Revision-Scope Changes)

## Изменения, которые создают новую ревизию

Следующие изменения автоматически приводят к созданию **новой ревизии** контейнерного приложения:

✅ Изменение образа контейнера (container image)  
✅ Изменение переменных окружения (environment variables)  
✅ Изменение выделенных ресурсов CPU и памяти  
✅ Изменение правил масштабирования (scale rules)  
✅ Изменение конфигурации Dapr  
✅ Изменение подключаемых томов (volume mounts)

> 📌 Логика простая: если изменение влияет на поведение или конфигурацию контейнера во время выполнения — создаётся новая ревизия.

---

## Изменения, которые НЕ создают новую ревизию

Следующие изменения относятся к уровню приложения (application-scoped) и **не приводят к созданию новой ревизии**:

❌ Секреты (Secrets)  
❌ Настройки Ingress  
❌ Режим ревизий (Single / Multiple revision mode)

---

## Важные уточнения для AZ-204

- **Secrets — application-scoped**, но если секрет используется как переменная окружения, потребуется перезапуск/новая ревизия, чтобы контейнер начал использовать новое значение.
- Настройки ingress (например, внешний/внутренний доступ) изменяются без создания новой версии контейнера.
- Режим ревизий:
    - **Single revision mode** — активна только одна ревизия.
    - **Multiple revision mode** — можно распределять трафик между несколькими версиями.

> 🎯 Экзаменационный лайфхак: запоминайте разделение на *container-scoped* и *application-scoped*. Это частый источник каверзных вопросов.

---


## Revision Naming

### Default Naming
```
<app-name>--<random-suffix>

Example:
myapp--7d7n3xz (automatically generated)
```

### Custom Naming
```bash
# Set custom revision suffix
az containerapp update \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-suffix v2-hotfix \
  --image myapp:v2.1

# Results in: myapp--v2-hotfix
```

### Best Practices по именованию ревизий

Правильное именование ревизий упрощает поддержку, откаты и анализ трафика.

✅ **Описательные (Descriptive)**  
Включайте номер версии, номер билда или название фичи.  
Пример: `api-v2-login-fix`

✅ **Последовательные (Sequential)**  
Используйте нумерацию (`v1`, `v2`, `v3`) или дату деплоя (`2024-01-15`).  
Это упрощает отслеживание истории изменений.

✅ **В нижнем регистре (Lowercase)**  
Допустимы только:
- строчные буквы
- цифры
- дефисы (`-`)

❌ Пробелы и специальные символы не допускаются.

✅ **Краткие (Short)**  
Имя должно быть лаконичным, но информативным.  
Слишком длинные имена усложняют чтение в портале Azure и CLI.

---

## Дополнительные рекомендации

- Добавляйте номер сборки из CI/CD (`v3-build-145`).
- Избегайте избыточных слов вроде `revision`, `new`, `final`.
- Поддерживайте единый стиль именования в команде.

> 🎯 Для AZ-204 важно помнить ограничения формата имени и понимать, что имя ревизии помогает при traffic splitting и rollback.

---

Examples:
- `v1-0-0`
- `2026-01-03`
- `feature-payment`
- `hotfix-auth`

## Managing Revisions

### List Revisions
```bash
# List all revisions
az containerapp revision list \
  --name myapp \
  --resource-group myResourceGroup \
  --output table

# Output:
# NAME                      ACTIVE  TRAFFIC  CREATED
# myapp--v1-0-0            True    80%      2026-01-01
# myapp--v2-0-0            True    20%      2026-01-03
# myapp--v1-5-0            False   0%       2026-01-02
```

### Show Revision Details
```bash
# Get specific revision info
az containerapp revision show \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--v2-0-0
```

### Activate Revision
```bash
# Activate inactive revision
az containerapp revision activate \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--v1-0-0
```

### Deactivate Revision
```bash
# Deactivate revision (stop routing traffic)
az containerapp revision deactivate \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--v1-0-0
```

### Copy Revision
```bash
# Create new revision from existing
az containerapp revision copy \
  --name myapp \
  --resource-group myResourceGroup \
  --from-revision myapp--v1-0-0 \
  --revision-suffix v1-0-1 \
  --image myapp:v1.0.1
```

## Traffic Splitting

### Traffic Management Modes

#### Single Revision Mode
**Default** - Only latest revision receives traffic:

```bash
# Single revision mode (default)
az containerapp update \
  --name myapp \
  --resource-group myResourceGroup \
  --image myapp:v2.0

# Old revision deactivated automatically
# 100% traffic to new revision
```

#### Multiple Revisions Mode
**Manual control** - Distribute traffic across revisions:

```bash
# Enable multiple revisions mode
az containerapp revision set-mode \
  --name myapp \
  --resource-group myResourceGroup \
  --mode multiple
```

### Split Traffic

#### Blue/Green Deployment
```bash
# 100% to blue (v1), 0% to green (v2)
az containerapp ingress traffic set \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-weight myapp--v1=100 myapp--v2=0

# Gradually shift to green
az containerapp ingress traffic set \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-weight myapp--v1=50 myapp--v2=50

# Complete rollout to green
az containerapp ingress traffic set \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-weight myapp--v1=0 myapp--v2=100
```

#### Canary Release
```bash
# 95% stable, 5% canary
az containerapp ingress traffic set \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-weight myapp--stable=95 myapp--canary=5

# Increase canary traffic
az containerapp ingress traffic set \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-weight myapp--stable=80 myapp--canary=20
```

#### A/B Testing
```bash
# 50% version A, 50% version B
az containerapp ingress traffic set \
  --name myapp \
  --resource-group myResourceGroup \
  --revision-weight myapp--version-a=50 myapp--version-b=50
```

### Traffic Distribution Example
```
User Requests (100%)
├── 70% → Revision v2.0 (stable)
├── 20% → Revision v2.1 (canary)
└── 10% → Revision v1.9 (fallback)
```
## Управление секретами (Secrets Management)

### Что такое Secrets?

**Secrets** — это безопасное хранилище для конфиденциальных данных и настроек в Azure Container Apps.

Используются для хранения:

- строк подключения (connection strings)
- паролей
- API-ключей
- токенов
- чувствительных параметров конфигурации

Основные свойства:

- Область действия — уровень приложения (application-scoped), а не ревизии
- Шифруются при хранении (encrypted at rest)
- Используются через переменные окружения или правила масштабирования
- Изменение значения секрета не создаёт новую ревизию

> 💡 Важно: Secret сам по себе не привязан к конкретной ревизии — он общий для всего Container App.

---

### Характеристики Secrets

✅ **Application-scoped**  
Один и тот же секрет доступен всем ревизиям приложения.

✅ **Encrypted**  
Данные шифруются в Azure и не хранятся в открытом виде.

✅ **Нет автоматического обновления (No auto-update)**  
Если вы изменили секрет, уже запущенные контейнеры не начнут автоматически использовать новое значение.  
Требуется:
- перезапуск контейнера  
  или
- деплой новой ревизии

✅ **Множественные ссылки (Multiple references)**  
Несколько ревизий могут использовать один и тот же секрет.

---

## Важные нюансы для AZ-204

- Secret ≠ Environment Variable.  
  Secret — это защищённое хранилище.  
  Environment variable — способ передать значение в контейнер.

- Частая экзаменационная ловушка:  
  Изменение секрета **не создаёт новую ревизию**, но для применения нового значения контейнер нужно перезапустить.

- Для production-сценариев рекомендуется использовать интеграцию с Azure Key Vault (если требуется централизованное управление секретами).

---

### Define Secrets

#### At Creation
```bash
# Create app with secrets
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myapp:latest \
  --secrets "queue-connection=$QUEUE_CONNECTION" "api-key=$API_KEY"
```

#### Add Secrets
```bash
# Add secret to existing app
az containerapp secret set \
  --name myapp \
  --resource-group myResourceGroup \
  --secrets "db-password=$DB_PASSWORD"
```

### Reference Secrets

#### In Environment Variables
```bash
# Reference secret in environment variable
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myapp:latest \
  --secrets "queue-connection=$CONNECTION_STRING" \
  --env-vars \
    "QueueName=myqueue" \
    "ConnectionString=secretref:queue-connection"

# secretref:<secret-name> references the secret
```

#### In ARM Template
```json
{
  "properties": {
    "configuration": {
      "secrets": [
        {
          "name": "queue-connection",
          "value": "DefaultEndpointsProtocol=https;..."
        }
      ]
    },
    "template": {
      "containers": [
        {
          "env": [
            {
              "name": "ConnectionString",
              "secretRef": "queue-connection"
            }
          ]
        }
      ]
    }
  }
}
```

### List Secrets
```bash
# List secret names (not values)
az containerapp secret list \
  --name myapp \
  --resource-group myResourceGroup \
  --output table

# Output shows names only, not values
# NAME               
# queue-connection
# api-key
# db-password
```

### Update Secrets
```bash
# Update secret value
az containerapp secret set \
  --name myapp \
  --resource-group myResourceGroup \
  --secrets "api-key=$NEW_API_KEY"
```

### Remove Secrets
```bash
# Remove secret
az containerapp secret remove \
  --name myapp \
  --resource-group myResourceGroup \
  --secret-names "old-secret"
```

## Secret Updates and Revisions

### Secret Change Impact

**Secrets are application-scoped**, not revision-scoped:

```
Update Secret → No new revision created
              ↓
Existing revisions don't see change
              ↓
Must take action:
  1. Deploy new revision, OR
  2. Restart existing revision
```

### Responding to Secret Changes

#### Option 1: Deploy New Revision
```bash
# Trigger new revision
az containerapp update \
  --name myapp \
  --resource-group myResourceGroup \
  --image myapp:latest
```

#### Option 2: Restart Revision
```bash
# Restart existing revision
az containerapp revision restart \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--v1-0-0
```

### Перед удалением секретов (Before Deleting Secrets)

Удаление секрета требует аккуратности, особенно если он используется активными ревизиями.

Рекомендуемая последовательность действий:

1. **Задеплойте новую ревизию**  
   Создайте новую ревизию без ссылки на удаляемый секрет  
   (удалите его из переменных окружения или scale rules).

2. **Деактивируйте старые ревизии**, которые используют этот секрет  
   Убедитесь, что трафик больше не направляется на них.

3. **Безопасно удалите секрет**  
   Только после того, как ни одна активная ревизия его не использует.

---

## Почему это важно?

- Если удалить секрет, который используется активной ревизией, контейнер может перестать корректно запускаться.
- Это может привести к ошибкам масштабирования или падению приложения.
- В режиме multiple revisions важно проверить, что ни одна ревизия не получает трафик.

---

## Практический совет для AZ-204

На экзамене могут описать сценарий, где приложение начинает падать после удаления секрета.  
Правильный ответ обычно связан с тем, что:

- секрет использовался активной ревизией  
  или
- не была создана новая ревизия без зависимости от этого секрета

> 🎯 Запомните правило: сначала убрать зависимость → потом отключить старые ревизии → только затем удалить секрет.

---

```bash
# Step 1: Remove secret reference
az containerapp update \
  --name myapp \
  --resource-group myResourceGroup \
  --replace-env-vars "ConnectionString=new-value"

# Step 2: Deactivate old revisions
az containerapp revision deactivate \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--old-version

# Step 3: Delete secret
az containerapp secret remove \
  --name myapp \
  --resource-group myResourceGroup \
  --secret-names "old-connection"
```

## Key Vault Integration

⚠️ **No native Key Vault integration** for secrets:

**Workaround**: Use managed identity + Key Vault SDK in app

```csharp
// App code retrieves secrets from Key Vault
var client = new SecretClient(
    new Uri("https://myvault.vault.azure.net/"),
    new DefaultAzureCredential());

var secret = await client.GetSecretAsync("db-password");
var connectionString = secret.Value.Value;
```

💡 **Recommendation**: Use Key Vault for sensitive secrets, Container Apps secrets for non-critical config

## Best Practices

### 1. Use Descriptive Revision Names
```bash
--revision-suffix v2-0-1-hotfix
```

### 2. Test with Traffic Splitting
```bash
# Start with small percentage
--revision-weight new=5 stable=95
```

### 3. Keep Active Revisions Minimal
```bash
# Deactivate unused revisions
az containerapp revision deactivate ...
```

### 4. Store Secrets Securely
```bash
# Use environment variables or CI/CD secrets
az containerapp secret set --secrets "key=$SECRET_FROM_ENV"
```

### 5. Update Secrets Safely
```bash
# 1. Update secret
# 2. Deploy new revision OR restart existing
# 3. Test
# 4. Deactivate old revisions
```

## Критически важные моменты (Critical Notes)

- 💡 **Revisions** — неизменяемые снимки версии контейнерного приложения, создаются при определённых изменениях
- ⚠️ **Revision-scope изменения** — изменения образа, переменных окружения, ресурсов и правил масштабирования создают новую ревизию
- 🎯 **Traffic splitting** — позволяет реализовывать Blue/Green, Canary и A/B тестирование
- ✅ **Secrets** — относятся к уровню приложения (application-scoped), а не ревизии
- 📊 **Изменение секрета** — не создаёт новую ревизию, требуется перезапуск или redeploy
- 🔄 **Режимы ревизий** — Single (автоматический) или Multiple (ручное управление)
- 🔒 **Azure Key Vault** — не имеет нативной автоматической интеграции; используется через Managed Identity и SDK
- ⚠️ **Именование** — рекомендуется использовать настраиваемые суффиксы ревизий для удобной организации

---

# Exam Tips (AZ-204)

## Ревизии

- Ревизии — это неизменяемые снимки версии приложения.
- Изменения уровня ревизии включают:
    - образ контейнера
    - переменные окружения
    - CPU и память
    - правила масштабирования
    - конфигурацию Dapr
- Изменения, не создающие ревизию:
    - Secrets
    - Ingress
    - Revision mode

---

## Именование ревизий

- Используется формат с суффиксом ревизии.
- Суффикс можно задавать вручную.
- Рекомендуется придерживаться коротких и последовательных имён.

---

## Traffic Splitting

- Позволяет распределять трафик между несколькими активными ревизиями.
- Поддерживает стратегии постепенного переключения трафика.
- Используется для безопасного развертывания новых версий.

---

## Secrets

- Относятся к уровню приложения.
- Общие для всех ревизий.
- Используются через ссылку в переменных окружения.
- Обновление секрета не создаёт новую ревизию.
- Для применения изменений требуется перезапуск или redeploy.
- Перед удалением секрета необходимо убрать зависимость от него в активных ревизиях.

---

## Azure Key Vault

- Используется через Managed Identity.
- Доступ осуществляется программно через SDK.
- Автоматической нативной интеграции нет.

---

## Управление ревизиями через CLI

- Доступны команды для перезапуска ревизии.
- Поддерживается управление активными ревизиями и распределением трафика.

---

## Что важно запомнить для экзамена

- Какие изменения создают новую ревизию.
- Какие изменения относятся к уровню приложения.
- Различие между Single и Multiple revision mode.
- Поведение приложения при изменении секрета.
- Механизм распределения трафика между ревизиями.


[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/6-container-apps-revisions-secrets)
