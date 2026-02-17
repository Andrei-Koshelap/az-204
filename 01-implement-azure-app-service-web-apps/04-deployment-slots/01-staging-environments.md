# Deployment Slots Overview (Обзор deployment slots)

## Key Concepts (Ключевые понятия)

- **Deployment slot** — отдельный экземпляр приложения со своим hostname
- **Staging environment** — среда для тестирования перед продакшеном
- **Slot swapping** — обмен содержимым и конфигурацией между слотами
- **Zero downtime** — переключение без прерывания запросов
- **Rollback** — быстрый откат через обратный swap

---

## What Are Deployment Slots? (Что такое deployment slots?)

### Definition (Определение)

- 🟢 Это **полноценные live-приложения** со своим hostname
- 🧪 Используются как отдельные среды: staging, test, dev
- ⚙️ Работают в рамках одного **App Service Plan**
- 📦 Делят вычислительные ресурсы с production
- ✅ Доступны только в тарифах:
    - Standard
    - Premium
    - Isolated

> 💡 Deployment slots позволяют деплоить новую версию без риска для production.

---

## Основная идея

Вместо деплоя напрямую в production:

1. Деплой в **staging slot**
2. Проверка работоспособности
3. Выполнение **swap**
4. Staging становится production

---

## Почему это важно для AZ-204

- Deployment slots = способ обеспечить zero-downtime deployment
- Swap можно выполнить мгновенно
- Можно быстро откатиться через повторный swap
- Слоты используют те же ресурсы App Service Plan


### URL Format
```
Production:  https://<app-name>.azurewebsites.net
Staging slot: https://<app-name>-staging.azurewebsites.net
Custom slot:  https://<app-name>-<slot-name>.azurewebsites.net

Limits:
- Site name: Max 40 characters
- Site name + slot name: Max 59 characters
```

## Benefits of Deployment Slots (Преимущества deployment slots)

### 1️⃣ Проверка перед production

- Тестирование изменений в staging-среде
- Проверка с production-конфигурацией
- Обнаружение проблем до влияния на пользователей

> 💡 Можно подключить staging к production-базе (осторожно!) для максимально реалистичной проверки.

---

### 2️⃣ Прогрев инстансов (Warm-Up)

- Все инстансы запускаются и прогреваются до swap
- Нет cold start задержек
- Производительность не падает после релиза

---

### 3️⃣ Zero Downtime Deployment

- Переключение трафика происходит мгновенно
- Запросы не теряются
- Пользователь не замечает релиза

> 🎯 Это ключевое преимущество для production-сценариев.

---

### 4️⃣ Быстрый Rollback

- Предыдущая версия автоматически оказывается в staging после swap
- Один повторный swap — и система откатывается
- Можно мгновенно вернуть "last known good version"

---

### 5️⃣ Постепенный Rollout

- Можно направить часть трафика на новую версию
- Поддержка A/B testing и canary deployment
- Мониторинг поведения перед полным переходом

---

## Slot Availability by Tier (Доступность слотов по тарифам)

| Pricing Tier | Deployment Slots | Manual Scaling | Auto Swap |
|--------------|------------------|----------------|-----------|
| **Free (F1)** | 0 | ❌ Single instance | ❌ |
| **Shared (D1)** | 0 | ❌ Single instance | ❌ |
| **Basic (B1-B3)** | 0 | ✅ Manual only | ❌ |
| **Standard (S1-S3)** | 5 | ✅ | ✅ |
| **Premium (P1V2-P3V3)** | 20 | ✅ | ✅ |
| **Isolated (I1V2-I6V2)** | 20 | ✅ | ✅ |

⚠️ **Важно**: За сами deployment slots отдельная плата не взимается.  
Они используют ресурсы существующего App Service Plan.

---

## Важно для AZ-204

- Deployment slots доступны начиная с Standard
- Максимальное количество слотов зависит от тарифа
- Auto Swap позволяет автоматически выполнять swap после деплоя
- Rollback выполняется обычным повторным swap
- Слоты делят ресурсы App Service Plan


## Scaling Considerations

### Tier Limitations
```
Current: Standard tier (5 slots used)
Problem: Can't scale down to Basic (0 slots supported)
Solution: Delete slots first, then scale down

Current: Standard tier (3 slots used)
Action: Scale to Premium ✅ (supports 20 slots)
```

### Rule
Target tier must support **current number of slots** in use

## Creating New Slots

### Initial State
- **No content** by default (empty slot)
- Can **clone settings** from another slot
- Settings are editable after cloning

### Deployment Options
- Deploy from **different repository branch**
- Deploy from **different repository**
- Independent deployment pipeline

### Portal Creation
```
App Service → Deployment slots → Add Slot
├── Name: staging, dev, qa, etc.
├── Clone settings from: [select slot or None]
└── Click Add
```

### CLI Creation
```bash
# Create new deployment slot
az webapp deployment slot create \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot <slot-name>

# Clone configuration from production
az webapp deployment slot create \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot <slot-name> \
  --configuration-source <app-name>

# Clone from another slot
az webapp deployment slot create \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot <slot-name> \
  --configuration-source <source-slot-name>
```

### PowerShell Creation
```powershell
# Create deployment slot
New-AzWebAppSlot `
  -ResourceGroupName <rg-name> `
  -Name <app-name> `
  -Slot <slot-name>

# With cloned configuration
New-AzWebAppSlot `
  -ResourceGroupName <rg-name> `
  -Name <app-name> `
  -Slot <slot-name> `
  -SourceWebApp (Get-AzWebApp -ResourceGroupName <rg-name> -Name <app-name>)
```

## Common Slot Naming Patterns

### Environment-Based
```
production (default slot)
staging
dev
qa
uat
```

### Version-Based
```
production (default slot)
v2-0
v2-1
beta
```

### Feature-Based
```
production (default slot)
feature-auth
feature-payment
hotfix-123
```

## Typical Deployment Workflow

### Simple Staging → Production
```
1. Deploy to staging slot
2. Test in staging environment
3. Swap staging → production
4. If issues: Swap back immediately
```

### Multi-Slot Workflow
```
1. Develop in dev slot
2. Deploy to qa slot for testing
3. Promote to staging for final validation
4. Swap staging → production
5. Monitor production
6. Previous version in staging (rollback ready)
```

### CI/CD Integration
```
1. Code committed to repository
2. CI/CD pipeline builds and tests
3. Deploy to staging slot automatically
4. Run smoke tests on staging
5. Manual approval gate
6. Auto swap to production (or manual swap)
```

## Slot Resources (Ресурсы deployment slots)

### Shared Resources (Общие ресурсы — в рамках одного App Service Plan)

- **Compute** — виртуальные машины общие с production
- **Storage** — общая файловая система
- **Memory** — общий пул памяти
- **CPU** — общее распределение процессорных ресурсов

> 💡 Все слоты работают в рамках одного App Service Plan и делят его ресурсы.

---

### Separate Resources (Отдельные ресурсы — на уровне слота)

- **Application code** — разворачивается отдельно в каждом слоте
- **Configuration** — может отличаться (app settings, connection strings)
- **Hostname** — уникальный URL для каждого слота  
  (например: `appname-staging.azurewebsites.net`)
- **Database connections** — можно подключать разные базы данных

> 🎯 Это позволяет тестировать новую версию с отдельной конфигурацией,  
> не затрагивая production.

---

## Важно для AZ-204

- Ресурсы вычислений общие, но код и настройки — независимые
- Нагрузка в staging влияет на production (один план)
- Можно пометить настройки как **slot-specific**, чтобы они не менялись при swap
- Hostname каждого слота уникален


## Managing Multiple Slots

### List All Slots
```bash
# CLI
az webapp deployment slot list \
  --name <app-name> \
  --resource-group <rg-name>

# PowerShell
Get-AzWebAppSlot `
  -ResourceGroupName <rg-name> `
  -Name <app-name>
```

### Delete Slot
```bash
# CLI
az webapp deployment slot delete \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot <slot-name>

# PowerShell
Remove-AzWebAppSlot `
  -ResourceGroupName <rg-name> `
  -Name <app-name> `
  -Slot <slot-name>
```

## Deployment to Slots

### Deploy Code to Specific Slot
```bash
# Deploy ZIP file to staging slot
az webapp deployment source config-zip \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --src <zip-file-path>

# Deploy from GitHub to slot
az webapp deployment source config \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot dev \
  --repo-url https://github.com/user/repo \
  --branch develop
```

## Critical Notes (Критически важные моменты)

- 💡 Минимальный тариф — **Standard (S1)** для использования deployment slots
- ⚠️ Все слоты работают в рамках одного **App Service Plan** и делят его ресурсы
- 🎯 Production slot создаётся автоматически по умолчанию
- 📊 За сами слоты дополнительная плата не взимается
- ✅ Swap выполняется без потери запросов (zero downtime)
- 🔄 Быстрый rollback — повторный swap возвращает предыдущую версию
- ⏱️ Инстансы прогреваются перед завершением swap

---

## Exam Tips (Советы для экзамена)

- Deployment slots доступны только в Standard, Premium и Isolated
- Production slot существует всегда, остальные создаются вручную
- Каждый слот имеет уникальный hostname:  
  `<app>-<slot>.azurewebsites.net`
- Слоты делят ресурсы App Service Plan (нет отдельного compute)
- Новые слоты создаются без кода (даже если клонируются настройки)
- Standard поддерживает 5 слотов, Premium/Isolated — 20
- Swap не прерывает обработку запросов
- После swap предыдущая production-версия оказывается в staging
- Перед понижением тарифа нужно удалить все дополнительные слоты

[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-app-service-deployment-slots/2-app-service-staging-environments)
