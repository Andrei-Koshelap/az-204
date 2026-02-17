# Perform Slot Swaps (Выполнение swap между слотами)

## Key Concepts (Ключевые понятия)

- **Manual swap** — инициируется вручную через портал или CLI
- **Swap with preview** — двухфазный swap с предварительной проверкой
- **Auto swap** — автоматический swap после деплоя
- **Custom warm-up** — указание конкретных URL для прогрева
- **Rollback** — возврат к предыдущей версии через обратный swap

---

## Manual Swap (Ручной swap)

### Portal Workflow (Через Azure Portal)

1️⃣ Открыть **Web App**  
2️⃣ Перейти в раздел **Deployment slots**  
3️⃣ Нажать **Swap**  
4️⃣ Выбрать:
- **Source slot** (обычно staging)
- **Target slot** (обычно production)
  5️⃣ При необходимости включить **Swap with preview**
  6️⃣ Нажать **Start swap**

---

### Что происходит дальше

- Применяется конфигурация target к source
- Source перезапускается и прогревается
- После успешной проверки выполняется переключение маршрутизации
- Production начинает обслуживать новую версию

---

### CLI Example (Azure CLI)

```bash
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production
```

## Swap with Preview (Swap с предварительным просмотром)

Позволяет выполнить swap в два этапа.

### Phase 1 — Apply Configuration

- Настройки target применяются к source
- Приложение прогревается
- Трафик ещё не переключается

### Phase 2 — Complete Swap

- После проверки нажать **Complete swap**
- Выполняется финальное переключение маршрутизации

> 💡 Удобно для проверки production-конфигурации до фактического релиза.

---

## Auto Swap (Автоматический swap)

- Настраивается в свойствах слота
- После успешного деплоя автоматически выполняется swap
- Работает только при деплое в non-production слот

⚠️ Доступен начиная с **Standard tier**.

---

## Custom Warm-Up (Пользовательский прогрев)

Можно указать специальные пути для прогрева приложения:

- Через `applicationInitialization` в `web.config`
- Через переменные среды
- Использовать отдельный health-check endpoint

> 🎯 Позволяет убедиться, что приложение полностью инициализировано перед swap.

---

## Rollback (Откат)

Если после swap обнаружены проблемы:

1. Выполнить обратный swap
2. Production вернётся к предыдущей версии
3. Ошибочная версия останется в staging

---

## Важно для AZ-204

- Manual swap — самый частый экзаменационный сценарий
- Swap with preview — двухфазный процесс
- Auto swap работает только для non-production слотов
- Rollback = обычный повторный swap
- Для пользователя swap происходит мгновенно (zero downtime)

#### Simple Swap
```
1. Navigate to: App Service → Deployment slots
2. Click "Swap" button at top
3. Configure swap dialog:
   ├── Source: [Select slot, e.g., staging]
   ├── Target: [Usually production]
   ├── Source Changes tab: Preview what changes
   └── Target Changes tab: Preview what changes
4. Click "Swap" button
5. Wait for completion
6. Click "Close" when done
```

#### Before Clicking Swap
- ✅ Verify source has correct code deployed
- ✅ Check configuration changes are expected
- ✅ Ensure target is production (for zero downtime)
- ✅ Review both "Source Changes" and "Target Changes" tabs

### CLI Commands

#### Simple Swap
```bash
# Swap staging to production
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production

# Swap with specific source and target
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot dev \
  --target-slot staging
```

#### Check Slot Configuration Before Swap
```bash
# View staging slot config
az webapp config appsettings list \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging

# Compare with production
az webapp config appsettings list \
  --name <app-name> \
  --resource-group <rg-name>
```

### PowerShell Commands
```powershell
# Swap slots
Switch-AzWebAppSlot `
  -ResourceGroupName <rg-name> `
  -Name <app-name> `
  -SourceSlotName staging `
  -DestinationSlotName production

# Swap to production (default)
Switch-AzWebAppSlot `
  -ResourceGroupName <rg-name> `
  -Name <app-name> `
  -SourceSlotName staging
```

## Swap with Preview (Multi-Phase Swap)

### What Is Swap with Preview?
- **Two-phase operation** with manual validation between phases
- **Phase 1**: Apply target config to source, warm up source
- **Validation**: Test source with production config
- **Phase 2**: Complete the swap (or cancel)

### Benefits
- Validate app runs correctly with production settings
- Test before committing to swap
- Mission-critical applications need this
- Catch configuration issues before production impact

### Portal Workflow

#### Step 1: Initiate Swap with Preview
```
1. Navigate to: App Service → Deployment slots
2. Click "Swap"
3. ☑ Check "Perform swap with preview"
4. Select source and target
5. Review changes
6. Click "Start Swap"
```

#### Step 2: Validate Source Slot
```
Phase 1 completes → Notification appears

1. Open source slot URL in browser:
   https://<app-name>-<source-slot>.azurewebsites.net

2. Test application thoroughly:
   ├── Check functionality
   ├── Verify database connections
   ├── Test authentication
   ├── Check external API calls
   └── Review logs

3. Confirm app works with production config
```

#### Step 3: Complete or Cancel

##### Option A: Complete Swap
```
1. Return to swap dialog
2. Select "Complete Swap" in Swap action dropdown
3. Click "Complete Swap" button
4. Swap finishes (Phase 2)
5. Click "Close"
```

##### Option B: Cancel Swap
```
1. Return to swap dialog
2. Select "Cancel Swap" in Swap action dropdown
3. Click "Cancel Swap" button
4. Source slot reverted to original configuration
5. Click "Close"
```

### CLI - Swap with Preview

#### Start Swap with Preview
```bash
# Phase 1: Start preview
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production \
  --action preview

# Output shows pending swap state
```

#### Complete Swap
```bash
# Phase 2: Complete the swap
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production \
  --action swap
```

#### Cancel Swap
```bash
# Revert Phase 1 changes
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production \
  --action reset
```

### Testing During Preview Phase
```bash
# Test source slot with production config
curl https://<app-name>-staging.azurewebsites.net

# Check logs
az webapp log tail \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging

# Check application insights
# Review metrics in portal
```

## Auto Swap (Автоматический swap)

### What Is Auto Swap? (Что такое Auto Swap?)

- 🔄 **Автоматический swap** после деплоя кода в слот
- 🚀 Упрощает CI/CD-процессы
- 🔥 **Без cold start** — инстансы прогреваются до переключения
- ✅ **Zero downtime** — автоматическое продвижение в production

> 💡 После успешного деплоя в staging слот Azure автоматически выполнит swap в production.

---

### Requirements (Требования)

- ❌ **Не поддерживается**:
    - Linux Web Apps
    - Web App for Containers
- ✅ **Поддерживается**:
    - Windows Web Apps
- 📦 Требуемый тариф:
    - Standard
    - Premium
    - Isolated

> ⚠️ Auto Swap доступен только для Windows App Service и начиная с Standard tier.


### Configure Auto Swap in Portal

```
1. Navigate to: App Service → Deployment slots → [Select slot]
2. Click slot name (e.g., "staging")
3. Go to: Configuration → General settings
4. Find "Auto swap enabled" setting
5. Set to "On"
6. Select "Auto swap deployment slot": [Target slot, e.g., production]
7. Click "Save" in command bar
8. Confirm restart
```

### Configure Auto Swap via CLI
```bash
# Enable auto swap from staging to production
az webapp deployment slot auto-swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --auto-swap-slot production

# Disable auto swap
az webapp deployment slot auto-swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --disable
```

### Auto Swap Workflow
```
1. Deploy code to staging slot
   ├── Via Git push
   ├── Via Azure DevOps
   ├── Via GitHub Actions
   └── Via az webapp deployment

2. Deployment completes

3. Auto swap triggers automatically
   ├── Source slot warmed up
   ├── Custom warm-up executed (if configured)
   └── Swap to production occurs

4. Production updated
5. Staging has previous production version
```

### CI/CD Integration Example
```yaml
# Azure DevOps pipeline example
- task: AzureWebApp@1
  inputs:
    azureSubscription: '<subscription>'
    appName: '<app-name>'
    deployToSlotOrASE: true
    resourceGroupName: '<rg-name>'
    slotName: 'staging'
    package: '$(Build.ArtifactStagingDirectory)/**/*.zip'

# Auto swap configured on staging slot
# After deployment, auto swap occurs automatically
```

## Custom Warm-Up (Пользовательский прогрев)

### Why Custom Warm-Up? (Зачем нужен кастомный прогрев?)

Используется, когда стандартного запроса к `/` недостаточно.

- ⏳ **Долгая инициализация** — приложению требуется время на запуск
- 🧠 **Заполнение кеша** — предварительная загрузка данных в память
- 🔌 **Пулы соединений** — установление соединений с БД заранее
- 🛣 **Специальные маршруты** — необходимо прогреть определённые endpoints

> 💡 По умолчанию Azure делает HTTP-запрос к корню (`/`).  
> Custom warm-up позволяет указать более подходящий путь (например, `/health` или `/init`).

---

### Как настраивается

- Через `applicationInitialization` в `web.config`
- Через переменные среды
- Можно использовать отдельный health-check endpoint

> 🎯 Цель — гарантировать, что приложение полностью готово к приёму production-трафика до выполнения swap.


### Method 1: applicationInitialization (web.config)

#### Configuration
```xml
<configuration>
  <system.webServer>
    <applicationInitialization>
      <add initializationPage="/" hostName="[app hostname]" />
      <add initializationPage="/Home/About" hostName="[app hostname]" />
      <add initializationPage="/api/health" hostName="[app hostname]" />
    </applicationInitialization>
  </system.webServer>
</configuration>
```

#### Behavior (Поведение)

- Swap ожидает, пока указанные страницы ответят
- Каждая страница запрашивается до завершения swap
- Любой HTTP-ответ считается успешным (success)

---

### Method 2: App Settings (Способ 2: App Settings)

#### `WEBSITE_SWAP_WARMUP_PING_PATH`

Позволяет указать кастомный путь, который будет пинговаться во время warm-up:

- Значение — путь относительно корня приложения
- Обычно используют health-check endpoint

**Примеры**:

- `/health`
- `/api/health`
- `/ready`

> 💡 Это удобнее, чем прогрев через `/`, если корневая страница тяжёлая или зависит от внешних сервисов.


```bash
# Set custom warm-up path
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg-name> \
  --settings WEBSITE_SWAP_WARMUP_PING_PATH="/api/warmup"
```

**Default**: `/` (root path)

#### WEBSITE_SWAP_WARMUP_PING_STATUSES
Valid HTTP response codes for warm-up:

```bash
# Only accept 200 and 202 as successful warm-up
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg-name> \
  --settings WEBSITE_SWAP_WARMUP_PING_STATUSES="200,202"
```

**Default**: All response codes valid

**Behavior**: If response code not in list → Stop warm-up and swap

#### WEBSITE_WARMUP_PATH
Path to ping on **any restart** (not just swaps):

```bash
# Set warmup path for all restarts
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg-name> \
  --settings WEBSITE_WARMUP_PATH="/health"
```

**Use case**: General warm-up for all app starts

### Complete Custom Warm-Up Example
```bash
# Configure comprehensive warm-up
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg-name> \
  --settings \
    WEBSITE_SWAP_WARMUP_PING_PATH="/api/warmup" \
    WEBSITE_SWAP_WARMUP_PING_STATUSES="200" \
    WEBSITE_WARMUP_PATH="/health"

# Plus web.config:
# <applicationInitialization>
#   <add initializationPage="/api/cache/load" />
#   <add initializationPage="/api/connections/init" />
# </applicationInitialization>
```

## Rollback and Monitoring

### Quick Rollback
If production has issues after swap:

```bash
# Immediate rollback - swap back
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production

# Previous production version now in production again
# Problem version now in staging for investigation
```

### Monitor Swap Operation

#### Activity Log (Portal)
```
1. Navigate to: App Service → Activity log (left menu)
2. Filter:
   ├── Operation: "Swap Web App Slots"
   ├── Time range: Last 24 hours
3. Click operation to see details
4. Expand suboperations to see:
   ├── Configuration changes
   ├── Instance restarts
   ├── Errors (if any)
```

#### CLI - Query Activity Log
```bash
# Get recent swap operations
az monitor activity-log list \
  --resource-group <rg-name> \
  --resource-id "/subscriptions/<sub-id>/resourceGroups/<rg>/providers/Microsoft.Web/sites/<app>" \
  --caller "SlotSwap" \
  --start-time 2024-01-01 \
  --query "[].{Time:eventTimestamp, Status:status.value, Operation:operationName.value}" \
  --output table
```

#### Check Current Slot Configuration
```bash
# Verify production configuration after swap
az webapp show \
  --name <app-name> \
  --resource-group <rg-name> \
  --query "{Name:name, State:state, HostNames:hostNames}"

# Check staging after swap
az webapp show \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --query "{Name:name, State:state, HostNames:hostNames}"
```

### Troubleshooting Failed Swaps

#### Common Issues
1. **Instance restart failed** → Check application logs
2. **Warm-up timed out** → Review warm-up settings, increase timeout
3. **Configuration error** → Verify sticky settings, connection strings
4. **Resource exhausted** → Check memory/CPU, scale up if needed

#### Check Swap Status
```bash
# View deployment operations
az webapp deployment list-publishing-profiles \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging
```

## Swap Strategies (Стратегии использования swap)

---

### Strategy 1: Safe Deployment (Безопасный деплой)

1. Деплой в staging
2. Выполнить **Swap with preview**
3. Проверить staging с production-конфигурацией
4. Завершить swap (Complete swap)
5. Мониторить production
6. Оставить staging готовым для возможного rollback

> 💡 Минимизирует риски и позволяет проверить конфигурацию до релиза.

---

### Strategy 2: Blue-Green Deployment (Blue-Green стратегия)

1. Blue (production) обслуживает трафик
2. Деплой в Green (staging)
3. Swap Green → Blue
4. Green становится production, Blue — резервной версией
5. При проблемах: Swap Blue → Green (rollback)

> 🎯 Обеспечивает быстрый откат и минимальный риск.

---

### Strategy 3: Canary Release (Канареечный релиз)

1. Деплой в staging
2. Swap staging → production
3. Направить 5% трафика в staging (старая версия для сравнения)
4. Мониторить обе версии
5. Постепенно увеличивать долю трафика на новую версию
6. Полный переход после подтверждения стабильности

> 📊 Позволяет выявить проблемы на небольшой группе пользователей.


## Complete Swap Example

### Scenario: Deploy new version with validation

```bash
# 1. Deploy to staging
az webapp deployment source config-zip \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --src app-v2.zip

# 2. Start swap with preview
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production \
  --action preview

# 3. Test staging with production config
curl https://myapp-staging.azurewebsites.net
curl https://myapp-staging.azurewebsites.net/health
# ... more tests ...

# 4a. If tests pass: Complete swap
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production \
  --action swap

# 4b. If tests fail: Cancel swap
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production \
  --action reset

# 5. Monitor production (if swapped)
az webapp log tail \
  --name <app-name> \
  --resource-group <rg-name>

# 6. Rollback if needed
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production
```

## Critical Notes (Критически важные моменты)

- 💡 Для критичных приложений используйте **Swap with preview**
- ⚠️ **Auto swap** поддерживается только для Windows Web Apps (не Linux/Containers)
- 🎯 Всегда проверяйте приложение в preview-фазе перед завершением swap
- 📊 Все операции swap фиксируются в **Activity Log**
- ✅ Откат выполняется мгновенно через повторный swap
- 🔄 Настраивайте **Custom warm-up** для приложений с долгим стартом
- ⏱️ Swap выполняется без прерывания обслуживания (zero downtime)
- 🔒 Production должен быть **target** для гарантии zero downtime

---

## Exam Tips (Советы для экзамена)

- Manual swap выполняется через Portal или CLI:  
  `az webapp deployment slot swap`
- Swap with preview состоит из 3 шагов:  
  **Start → Validate → Complete / Cancel**
- Auto swap работает только для Windows Web Apps
- Auto swap запускается автоматически после деплоя в слот
- Custom warm-up настраивается через:
    - `applicationInitialization`
    - App Settings
- `WEBSITE_SWAP_WARMUP_PING_PATH` — задаёт путь для прогрева
- `WEBSITE_SWAP_WARMUP_PING_STATUSES` — определяет допустимые HTTP-коды
- Rollback = немедленный повторный swap
- В Activity Log операция отображается как  
  **"Swap Web App Slots"**
- Cancel в режиме preview откатывает изменения Phase 1
- Auto swap не поддерживается для Linux и Container Apps


[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-app-service-deployment-slots/4-swap-deployment-slots)
