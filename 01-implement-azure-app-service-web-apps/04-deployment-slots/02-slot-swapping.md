# Slot Swapping Mechanics (Механика swap между слотами)

## Key Concepts (Ключевые понятия)

- **Swap operation** — обмен кодом и конфигурацией между двумя слотами
- **Source slot** — слот, который готовится к продакшену
- **Target slot** — целевой слот (обычно production)
- **Sticky settings** — настройки, которые не меняются при swap
- **Swap with preview** — двухфазный swap с предварительной проверкой

---

## Swap Operation Phases (Фазы выполнения swap)

---

### Phase 1: Apply Target Settings to Source
(Применение настроек target к source)

App Service применяет конфигурацию target-слота ко всем инстансам source:

1. **Slot-specific app settings** (если помечены как sticky)
2. **Connection strings** (если sticky)
3. **Continuous deployment settings**
4. **App Service authentication settings**

📌 Результат:  
Все инстансы source перезапускаются с конфигурацией target.

> 💡 Это гарантирует, что после swap приложение будет работать с production-настройками.

---

### Phase 2: Wait for Restart
(Ожидание перезапуска)

- Платформа отслеживает перезапуск всех инстансов source
- ❌ Если хотя бы один инстанс не стартует → swap отменяется
- ✅ Если все стартовали успешно → переход к следующей фазе

---

### Phase 3: Local Cache Initialization (если включено)

- Отправляется HTTP-запрос к `/` на каждый инстанс
- Ожидание ответа
- Это вызывает дополнительный restart

> ⚠️ Local Cache добавляет дополнительный этап прогрева.

---

### Phase 4: Application Initialization / Warm-Up
(Инициализация приложения)

#### A. Auto Swap + Custom Warm-Up

- Запускается `applicationInitialization` из web.config
- HTTP-запрос к `/` на каждый инстанс
- Инстанс считается прогретым при любом HTTP-ответе

#### B. Standard Swap

- Отправляется HTTP-запрос к `/`
- Ожидание любого HTTP-ответа

> 🎯 Цель — убедиться, что приложение готово к обработке трафика.

---

### Phase 5: Perform the Swap
(Фактическое переключение)

- Меняются правила маршрутизации
- Target начинает обслуживать прогретую версию source
- Обслуживание запросов продолжается без прерывания

✅ **Zero downtime**

---

### Phase 6: Finalize Other Slot
(Завершение для второго слота)

- Source получает прежнюю версию target
- Применяются соответствующие настройки
- Выполняется перезапуск

Оба слота полностью синхронизированы и готовы к работе.

---

## Важно для AZ-204

- Swap сначала применяет конфигурацию, затем переключает маршрутизацию
- Если warm-up не проходит — swap отменяется
- Sticky settings остаются привязанными к слоту
- Zero downtime достигается за счёт предварительного прогрева
- Swap можно выполнить обратно для быстрого rollback


## Visual Swap Flow

```
Before Swap:
┌──────────────┐           ┌──────────────┐
│  Production  │           │   Staging    │
│              │           │              │
│  App v1.0    │           │  App v2.0    │
│  Prod config │           │  Stage config│
└──────────────┘           └──────────────┘

Phase 1-4: Warm up staging with production config
┌──────────────┐           ┌──────────────┐
│  Production  │  Config   │   Staging    │
│              │  ──────>  │              │
│  App v1.0    │           │  App v2.0    │
│  Prod config │           │  + Prod cfg  │
└──────────────┘           └──────────────┘
                           Warmed & Ready

Phase 5: Swap routing
┌──────────────┐           ┌──────────────┐
│  Production  │  <═══>    │   Staging    │
│  App v2.0    │           │  App v1.0    │
│  Prod config │           │  Stage config│
└──────────────┘           └──────────────┘

After Swap:
- Production serves v2.0 (from staging)
- Staging has v1.0 (rollback ready)
- Zero downtime during entire process
```

## Configuration Behavior During Swap (Поведение конфигурации при swap)

Во время swap часть настроек перемещается вместе с приложением,  
а часть остаётся привязанной к конкретному слоту.

---

### Settings That SWAP (Перемещаются вместе с приложением)

| Setting | Swaps? | Notes |
|----------|--------|-------|
| **General settings** | ✅ | Framework, 32/64-bit, WebSockets |
| **App settings** | ✅ | Если не отмечены как slot-specific |
| **Connection strings** | ✅ | Если не отмечены как slot-specific |
| **Handler mappings** | ✅ | |
| **Public certificates** | ✅ | |
| **WebJobs content** | ✅ | |
| **Hybrid connections** | ✅ | |
| **Azure CDN** | ✅ | |
| **Service endpoints** | ✅ | |
| **Path mappings** | ✅ | |

> 💡 По умолчанию App Settings и Connection Strings участвуют в swap.

---

### Settings That DON'T Swap (Остаются в слоте)

| Setting | Swaps? | Notes |
|----------|--------|-------|
| **Publishing endpoints** | ❌ | Slot-specific |
| **Custom domain names** | ❌ | Slot-specific |
| **Non-public certificates & TLS/SSL** | ❌ | Slot-specific |
| **Scale settings** | ❌ | Slot-specific |
| **WebJobs schedulers** | ❌ | Slot-specific |
| **IP restrictions** | ❌ | Slot-specific |
| **Always On** | ❌ | Slot-specific |
| **Diagnostic logs** | ❌ | Slot-specific |
| **CORS** | ❌ | Slot-specific |
| **Virtual network integration** | ❌ | Slot-specific |
| **Managed identities** | ❌ | Никогда не участвуют в swap |
| **Settings ending in `_EXTENSION_VERSION`** | ❌ | Slot-specific |

> ⚠️ Managed Identity никогда не перемещается между слотами.

---

## Slot-Specific Settings (Sticky Settings)

### Что такое Sticky Settings?

Это настройки, которые **остаются в своём слоте** при swap:

- Отмечаются галочкой **"Deployment slot setting"**
- Не переносятся в target при swap
- Используются для environment-specific конфигурации

---

### Типичные сценарии использования

- Разные строки подключения к БД (prod vs staging)
- Разные API keys
- Разные connection strings к очередям/кешу
- Feature flags для тестирования

> 🎯 Sticky settings позволяют безопасно тестировать staging,  
> не затрагивая production-конфигурацию.

---

## Важно для AZ-204

- По умолчанию настройки участвуют в swap
- Sticky settings остаются в слоте
- Managed Identity никогда не меняется
- Неправильная настройка sticky может привести к ошибкам после swap


### Common Use Cases
```
Production slot:
  DATABASE_URL = prod-database.azure.com (sticky)
  API_KEY = prod-key-123 (sticky)
  FEATURE_FLAG = false

Staging slot:
  DATABASE_URL = staging-database.azure.com (sticky)
  API_KEY = staging-key-456 (sticky)
  FEATURE_FLAG = true

After swap:
  DATABASE_URL values don't swap (stay with slots)
  API_KEY values don't swap (stay with slots)
  FEATURE_FLAG values don't swap (stay with slots)
```

### Configure Sticky Settings

#### Portal
```
App Service → Configuration → Application settings
1. Add or edit setting
2. Check "Deployment slot setting" checkbox
3. Save

OR for existing setting:
1. Click setting
2. Check "Deployment slot setting"
3. OK → Save
```

#### CLI
```bash
# Mark app setting as sticky (slot-specific)
az webapp config appsettings set \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --settings KEY=VALUE \
  --slot-settings KEY

# Mark connection string as sticky
az webapp config connection-string set \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --connection-string-type SQLAzure \
  --settings CONN="Server=..." \
  --slot-settings CONN
```

## Override Sticky Settings Behavior

### Make All Settings Swappable
Add this app setting to **every slot**:

```
Name: WEBSITE_OVERRIDE_PRESERVE_DEFAULT_STICKY_SLOT_SETTINGS
Value: 0 or false
```

**Effect**: Все настройки становятся учаcтвующими в swap  
(кроме Managed Identity)

⚠️ **All or nothing** — нельзя выборочно сделать часть настроек swappable

---

### When to Use (Когда использовать)

- Тестирование поведения swap
- Временные сценарии
- Продвинутые конфигурации

---

# Key Swap Principles (Ключевые принципы swap)

---

## 1️⃣ Target Slot остаётся онлайн

- Source подготавливается, пока target обслуживает трафик
- Во время подготовки downtime отсутствует
- Target затрагивается только в момент финального переключения (мгновенно)

---

## 2️⃣ Production всегда Target

Всегда выполнять swap **в production**:

✅ Правильно:  
`staging (source) → production (target)`

❌ Неправильно:  
`production (source) → staging (target)`

**Почему:**  
Production должен оставаться онлайн во время подготовки.

---

## 3️⃣ Вся подготовка происходит на Source

На source выполняется:

- Применение конфигурации
- Перезапуск инстансов
- Прогрев приложения
- Target остаётся нетронутым до финального swap

---

## 4️⃣ Zero Downtime Guarantee

- Переключение маршрутизации мгновенное
- Запросы не теряются
- Пользователь не замечает релиза

---

# Swap Scenarios (Типовые сценарии)

---

## Scenario 1: Simple Deploy

**Цель:** выкатить staging в production

1. Деплой кода в staging
2. Полное тестирование
3. Swap staging → production
4. Staging получает старую production-версию (готово к rollback)

---

## Scenario 2: Rollback

**Проблема:** после swap обнаружены ошибки

1. Немедленный swap обратно
2. Production получает предыдущую версию
3. Ошибочная версия остаётся в staging для анализа

---

## Scenario 3: Staged Rollout

**Цель:** протестировать на части пользователей

1. Swap staging → production
2. Направить 10% трафика в staging
3. Мониторить обе версии
4. Постепенно менять распределение трафика

---

# Critical Notes (Критически важные моменты)

- 💡 Swap — это 6-фазный процесс
- ⚠️ Вся подготовка выполняется на source
- 🎯 Production всегда должен быть target
- 📊 Sticky settings остаются в своём слоте
- ✅ Любой HTTP-ответ считается успешным warm-up
- 🔄 Возможны два перезапуска (конфигурация + local cache)
- ⏱️ При ошибке перезапуска swap автоматически отменяется
- 🔒 Managed Identity никогда не участвует в swap

---

# Exam Tips (Советы для экзамена)

- Запомнить 6 фаз swap:  
  `apply config → restart → cache → warmup → swap → finalize`
- Target остаётся онлайн до финального переключения
- Production должен быть target
- Настройки с флагом "Deployment slot setting" не участвуют в swap
- Managed Identity всегда slot-specific
- Если любой инстанс не стартует — swap откатывается
- Local Cache вызывает дополнительный restart
- Custom warm-up использует `applicationInitialization` в web.config
- Переменная  
  `WEBSITE_OVERRIDE_PRESERVE_DEFAULT_STICKY_SLOT_SETTINGS=0`  
  делает все настройки swappable
- Для пользователя swap выполняется мгновенно (zero downtime)

[Learn More](https://learn.microsoft.com/en-us/training/modules/understand-app-service-deployment-slots/3-app-service-slot-swapping)
