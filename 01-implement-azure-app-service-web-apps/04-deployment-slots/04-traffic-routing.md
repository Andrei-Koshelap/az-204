# Traffic Routing (Маршрутизация трафика)

## Key Concepts (Ключевые понятия)

- **Automatic routing** — Azure автоматически направляет процент трафика в разные слоты
- **Manual routing** — пользователь вручную выбирает слот через параметр запроса
- **x-ms-routing-name** — cookie или query parameter для привязки к слоту
- **Client affinity** — пользователь закрепляется за слотом на время сессии
- **A/B testing** — сравнение версий на реальном трафике
- **Canary releases** — постепенный вывод новой версии в production

---

## Automatic Routing (Автоматическая маршрутизация)

- Можно задать процент трафика для каждого слота
- Например:
    - Production — 90%
    - Staging — 10%
- Azure распределяет входящие запросы автоматически

> 💡 Используется для постепенного релиза или тестирования новой версии.

---

## Manual Routing (Ручная маршрутизация)

Пользователь может явно указать слот через:

- Query parameter
- Cookie `x-ms-routing-name`

Пример:
```
https://app.azurewebsites.net/?x-ms-routing-name=staging
```

> 🎯 Полезно для тестирования конкретной версии без изменения глобального распределения.


## Client Affinity (Привязка клиента)

- После первого запроса пользователь закрепляется за выбранным слотом
- Используется cookie `x-ms-routing-name`
- Обеспечивает консистентность сессии

> ⚠️ Важно для приложений с session state.

---

## A/B Testing (A/B тестирование)

- Направление части пользователей на новую версию
- Сравнение:
    - Производительности
    - Ошибок
    - Конверсии
- Позволяет принимать решения на основе реальных данных

---

## Canary Releases (Канареечный релиз)

- Новая версия получает небольшой процент трафика (например, 5%)
- Постепенное увеличение доли при отсутствии проблем
- Минимизирует риски при релизе

---

## Важно для AZ-204

- Traffic routing работает между deployment slots
- Пользователь закрепляется за слотом через cookie
- Можно комбинировать swap и traffic routing
- Отличать Automatic routing от Manual routing

## Default Behavior

### Production URL
All requests go to production by default:
```
URL: https://<app-name>.azurewebsites.net
Result: Routed to production slot (100% traffic)
```

### Direct Slot Access
Each slot has unique URL:
```
Production: https://<app-name>.azurewebsites.net
Staging:    https://<app-name>-staging.azurewebsites.net
Dev:        https://<app-name>-dev.azurewebsites.net
```

## Automatic Traffic Routing (Автоматическая маршрутизация трафика)

### What Is Automatic Routing? (Что это такое?)

- Разделение production-трафика между несколькими слотами
- Процентное распределение (percentage-based)
- Azure автоматически маршрутизирует запросы согласно настройкам
- Пользователь **закрепляется за выбранным слотом** на время сессии

> 💡 Используется для A/B тестирования и постепенного (canary) релиза.

---

### How It Works (Как это работает)

1. Вы задаёте процент трафика для каждого слота  
   Например:
    - Production — 80%
    - Staging — 20%

2. Azure распределяет входящие запросы случайным образом согласно процентам

3. После первого запроса пользователь:
    - Получает cookie `x-ms-routing-name`
    - Закрепляется за конкретным слотом (client affinity)

---

### Important Behavior (Важное поведение)

- Распределение происходит только при первом запросе
- Повторные запросы идут в тот же слот
- Изменение процентов влияет только на новых пользователей

> 🎯 Это предотвращает "прыжки" пользователя между версиями.


### Configure in Portal

```
1. Navigate to: App Service → Deployment slots
2. Find "Traffic %" column
3. For each slot, enter percentage (0-100)
   ├── Production: 90%
   ├── Staging: 10%
   └── Total must equal 100%
4. Click "Save" at top
```

### Visual Example
```
┌─────────────────────────────────────┐
│ Deployment Slots                    │
├─────────────┬───────────┬──────────┤
│ Name        │ State     │ Traffic %│
├─────────────┼───────────┼──────────┤
│ production  │ Running   │ 80%      │
│ staging     │ Running   │ 20%      │
└─────────────┴───────────┴──────────┘

Result:
- 80 out of 100 users → production
- 20 out of 100 users → staging
```

### CLI Configuration
```bash
# Route 20% traffic to staging slot
az webapp traffic-routing set \
  --name <app-name> \
  --resource-group <rg-name> \
  --distribution staging=20

# Route 10% to staging, 5% to dev
az webapp traffic-routing set \
  --name <app-name> \
  --resource-group <rg-name> \
  --distribution staging=10 dev=5

# Clear traffic routing (all to production)
az webapp traffic-routing clear \
  --name <app-name> \
  --resource-group <rg-name>

# Show current traffic distribution
az webapp traffic-routing show \
  --name <app-name> \
  --resource-group <rg-name>
```

### PowerShell Configuration
```powershell
# Set traffic distribution
Set-AzWebAppSlot `
  -ResourceGroupName <rg-name> `
  -Name <app-name> `
  -Slot staging `
  -TrafficReroutePercentage 20

# View current traffic routing
Get-AzWebAppSlot `
  -ResourceGroupName <rg-name> `
  -Name <app-name> `
  -Slot staging | Select TrafficReroutePercentage
```

## Client Affinity (Session Pinning)

### How It Works
Once user routed to a slot, they **stay on that slot**:

```
User's first request:
1. Arrives at: https://myapp.azurewebsites.net
2. Azure routes to: staging (based on 20% config)
3. Response includes: Set-Cookie: x-ms-routing-name=staging

User's subsequent requests:
1. Browser sends: Cookie: x-ms-routing-name=staging
2. Azure reads cookie
3. All requests → staging slot (for lifetime of session)
```

### Cookie Details
```http
Cookie: x-ms-routing-name=staging
Cookie: x-ms-routing-name=self     (for production)
```

- **`x-ms-routing-name=staging`** - Routed to staging slot
- **`x-ms-routing-name=self`** - Routed to production slot
- **Session lifetime** - Until browser closed or cookie expires

### Verify Routing
```bash
# Check which slot handling request
curl -v https://myapp.azurewebsites.net

# Look for in response headers:
# Set-Cookie: x-ms-routing-name=staging; path=/
# Or
# Set-Cookie: x-ms-routing-name=self; path=/
```

## Manual Traffic Routing

### What Is Manual Routing?
Users explicitly choose which slot to access via query parameter:
- **Opt-in** to beta/staging version
- **Opt-out** back to production
- **Testing** specific slot without automatic routing

### Query Parameter
```
x-ms-routing-name=<slot-name>
```

### Use Cases

#### 1. Opt Out of Beta (Return to Production)
```html
<!-- Link on webpage -->
<a href="https://myapp.azurewebsites.net/?x-ms-routing-name=self">
  Go back to production app
</a>
```

**Result**: User routed to production, cookie set to `self`

#### 2. Opt In to Beta (Access Staging)
```html
<!-- Link on webpage -->
<a href="https://myapp.azurewebsites.net/?x-ms-routing-name=staging">
  Try our beta version
</a>
```

**Result**: User routed to staging, cookie set to `staging`

#### 3. Test Specific Slot
```bash
# Force request to staging
curl https://myapp.azurewebsites.net/?x-ms-routing-name=staging

# Force request to production
curl https://myapp.azurewebsites.net/?x-ms-routing-name=self

# Force request to custom slot
curl https://myapp.azurewebsites.net/?x-ms-routing-name=dev
```

### After Manual Routing
Cookie persists for session:
```http
Request 1: https://myapp.azurewebsites.net/?x-ms-routing-name=staging
Response: Set-Cookie: x-ms-routing-name=staging

Request 2: https://myapp.azurewebsites.net/page2
Cookie sent: x-ms-routing-name=staging
Routed to: staging slot
```

## Hidden Slot Access

### Scenario: Internal Testing
Configure slot for internal team access only:

```
Portal Configuration:
Slot: staging
Traffic %: 0% (shown in grey)

Result:
- Not automatically routed (0%)
- Still accessible via manual routing
- Hidden from public automatic routing
```

### Access Hidden Slot
```bash
# Automatic routing won't send traffic (0%)
curl https://myapp.azurewebsites.net
# → Always production

# Manual routing still works
curl "https://myapp.azurewebsites.net/?x-ms-routing-name=staging"
# → Staging slot

# Or direct URL
curl https://myapp-staging.azurewebsites.net
# → Staging slot
```

### Display in Portal (Отображение в портале)

| Traffic % | Display | Meaning |
|------------|----------|----------|
| **0% (grey)** | Серым цветом | Значение по умолчанию, явно не задано |
| **0% (black)** | Чёрным текстом | Явно установлено 0% |

**Разница:**  
Явно заданные 0% позволяют учитывать слот при ручной маршрутизации.

---

# Common Traffic Routing Scenarios (Типовые сценарии маршрутизации)

---

## Scenario 1: A/B Testing (A/B тестирование)

**Цель:** сравнить две версии

### Конфигурация

- Production: v1.0 (50% трафика)
- Staging: v2.0 (50% трафика)

### Мониторинг

- Конверсия по версиям
- Частота ошибок
- Метрики производительности

### Результат

- Выбор версии с лучшими показателями

---

## Scenario 2: Canary Release (Канареечный релиз)

**Цель:** постепенное внедрение новой версии

### Phase 1

- Production: v1.0 (95%)
- Staging: v2.0 (5%)
- Мониторинг стабильности

### Phase 2 (если стабильно)

- Production: v1.0 (80%)
- Staging: v2.0 (20%)

### Phase 3

- Swap staging → production
- v2.0 становится production (100%)

---

## Scenario 3: Beta Program (Бета-программа)

**Цель:** дать пользователям возможность добровольно участвовать

### Конфигурация

- Production: стабильная версия (100% auto)
- Staging: beta-версия (0% auto)

### Поведение пользователя

- По умолчанию → production
- Opt-in ссылка:  
  `?x-ms-routing-name=staging`
- Opt-out ссылка:  
  `?x-ms-routing-name=self`

> 💡 Позволяет тестировать beta без изменения глобального трафика.

---

## Scenario 4: Geographic Testing (Региональное тестирование)

**Цель:** протестировать в ограниченном регионе

### Phase 1

- Production: v1.0 (100%)
- Staging: v2.0 (0%)
- Деплой в staging
- Ручная маршрутизация для внутренних тестировщиков

### Phase 2

- Production: v1.0 (90%)
- Staging: v2.0 (10%)
- Ограниченный публичный rollout

### Phase 3

- Swap staging → production
- Полный релиз новой версии

---

## Важно для AZ-204

- Automatic routing работает через процентное распределение
- Пользователь закрепляется за слотом через cookie
- Изменение процентов влияет только на новых пользователей
- 0% (grey) ≠ 0% (black)


## Traffic Routing Commands Reference

### View Current Traffic Distribution
```bash
# CLI
az webapp traffic-routing show \
  --name <app-name> \
  --resource-group <rg-name>

# Output:
# [
#   {
#     "actionHostName": "myapp-staging.azurewebsites.net",
#     "reroutePercentage": 20.0
#   }
# ]
```

### Set Traffic Distribution
```bash
# Single slot
az webapp traffic-routing set \
  --name <app-name> \
  --resource-group <rg-name> \
  --distribution staging=15

# Multiple slots
az webapp traffic-routing set \
  --name <app-name> \
  --resource-group <rg-name> \
  --distribution staging=15 dev=5
```

### Clear Traffic Routing
```bash
# Remove all traffic routing (100% to production)
az webapp traffic-routing clear \
  --name <app-name> \
  --resource-group <rg-name>
```

## Monitoring Traffic Distribution

### Application Insights
Track which slot served requests:
```csharp
// Custom telemetry
telemetryClient.TrackEvent("PageView", new Dictionary<string, string>
{
    { "Slot", Environment.GetEnvironmentVariable("WEBSITE_SLOT_NAME") }
});
```

### Log Analytics Query
```kusto
requests
| where timestamp > ago(1h)
| extend slot = tostring(customDimensions.Slot)
| summarize RequestCount = count() by slot
| project slot, RequestCount, Percentage = (RequestCount * 100.0) / sum(RequestCount)
```

### Portal Metrics
```
App Service → Metrics
├── Metric: Requests
├── Split by: cloud_RoleInstance
└── Shows distribution across instances (and slots)
```

## Traffic Routing Best Practices

### 1. Start Small
```
Phase 1: 5% to new version
Phase 2: 10% to new version (if stable)
Phase 3: 25% to new version
Phase 4: 50% to new version
Phase 5: 100% (full swap)
```

### 2. Monitor Closely
- Error rates per slot
- Response times per slot
- Exception counts
- User feedback

### 3. Have Rollback Plan
```bash
# If issues with canary:
# Option 1: Reduce traffic
az webapp traffic-routing set \
  --name <app-name> \
  --resource-group <rg-name> \
  --distribution staging=0

# Option 2: Swap back
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production
```

### 4️⃣ Clear Communication (Чёткая коммуникация)

- Информируйте пользователей о beta-программе
- Предоставляйте ссылки для opt-in / opt-out
- Заранее обозначайте особенности поведения beta-версии

> 💡 Пользователи должны понимать, что участвуют в тестировании.

---

## Critical Notes (Критически важные моменты)

- 💡 **Client affinity** — пользователь закрепляется за слотом на время сессии
- ⚠️ Используется cookie `x-ms-routing-name`
- 🎯 Query parameter позволяет вручную выбрать слот
- 📊 0% routing скрывает слот из авто-маршрутизации, но ручной доступ возможен
- ✅ Постепенно увеличивайте процент трафика при rollout
- 🔄 A/B testing выполняется на реальном production-трафике
- ⏱️ Пользователь остаётся в одном слоте в течение всей сессии
- 🔒 Весь трафик идёт через production URL, не через URL слота

---

## Exam Tips (Советы для экзамена)

- По умолчанию весь трафик (100%) идёт в production
- Traffic routing распределяет трафик production URL между слотами
- Каждый слот имеет собственный прямой URL  
  (например: `myapp-staging.azurewebsites.net`)
- Cookie `x-ms-routing-name` закрепляет пользователя за слотом
- `x-ms-routing-name=self` направляет в production
- `x-ms-routing-name=<slot>` направляет в конкретный слот
- Ручная маршрутизация через:  
  `?x-ms-routing-name=<slot>`
- 0% (grey) позволяет ручной доступ без авто-распределения
- CLI-команды:  
  `az webapp traffic-routing set`  
  `az webapp traffic-routing show`  
  `az webapp traffic-routing clear`
- Используется для A/B testing, canary releases и beta-программ
- Сумма процентов ≤ 100% (остаток автоматически идёт в production)


[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-app-service-deployment-slots/5-route-traffic-app-service)
