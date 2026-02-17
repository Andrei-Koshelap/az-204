# Сетевые возможности Azure App Service

## Ключевые концепции

- **Мультитенантный сервис** — приложения используют общую инфраструктуру (кроме Isolated tier).
- **Inbound-функции** — управление входящим трафиком (к приложению).
- **Outbound-функции** — управление исходящим трафиком (от приложения).
- **Нет прямого сетевого подключения к VM** — используются встроенные сетевые механизмы платформы.

💡 App Service не предоставляет прямого доступа к виртуальным машинам — управление сетью выполняется через платформенные инструменты.

---

## Типы развертывания

| Тип | Tier | Сетевая модель |
|------|------|----------------|
| **Multitenant** | Free, Shared, Basic, Standard, Premium, PremiumV2, PremiumV3 | Общая инфраструктура |
| **Single-tenant (ASE)** | Isolated, IsolatedV2 | Выделенная Azure VNet |

⚠️ Если требуется полная изоляция сети и инфраструктуры — используется ASE (App Service Environment).

---

# Обзор сетевых возможностей

## Inbound-функции (контроль входящего трафика)

| Функция | Назначение | Сценарий |
|-----------|------------|------------|
| **App-assigned address** | Выделенный IP для приложения | IP-based SSL, фиксированный inbound IP |
| **Access restrictions** | Фильтрация по IP или VNet | Разрешить только определённые IP/подсети |
| **Service endpoints** | Безопасный доступ из VNet | Ограничить доступ определённой VNet |
| **Private endpoints** | Приватный IP внутри VNet | Полностью приватное приложение |

---

## Outbound-функции (контроль исходящего трафика)

| Функция | Назначение | Сценарий |
|-----------|------------|------------|
| **Hybrid Connections** | Подключение к on-prem | Доступ к локальным БД или API |
| **Gateway VNet Integration** | Устаревшая VNet-интегация | Legacy-развертывания |
| **VNet Integration** | Современная интеграция | Доступ к ресурсам внутри VNet |

---

# Частые сценарии (Inbound)

| Требование | Решение |
|-------------|----------|
| IP-based SSL | App-assigned address |
| Выделенный inbound IP | App-assigned address |
| Ограничение по IP | Access restrictions |
| Ограничение доступа только из VNet | Service endpoints или Private endpoints |
| Полностью приватное приложение | Private endpoints |

---

# Исходящие IP-адреса (Outbound IP)

## Как это работает

- IP-адреса разделяются внутри одного семейства VM.
- При смене tier может измениться VM family → изменятся outbound IP.
- Приложение может использовать несколько outbound IP одновременно.
- Список outbound IP доступен в свойствах приложения (Portal или CLI).

💡 Это важно, если внешний firewall требует whitelist IP-адресов.

---

## Как найти outbound IP

Через Azure CLI:

```bash
# Get current outbound IPs
az webapp show \
  --resource-group <rg-name> \
  --name <app-name> \
  --query outboundIpAddresses \
  --output tsv

# Get ALL possible outbound IPs (across all tiers)
az webapp show \
  --resource-group <rg-name> \
  --name <app-name> \
  --query possibleOutboundIpAddresses \
  --output tsv
```
### Важное замечание (экзаменационная ловушка)
⚠️ Outbound IP:
Не фиксированный по умолчанию.
Может измениться при масштабировании или смене tier.
Может включать несколько адресов.
Если требуется фиксированный outbound IP — необходимо использовать NAT Gateway через VNet Integration.
Дополнительные рекомендации
Для строгих требований безопасности предпочтительно использовать Private Endpoint вместо Access Restrictions.
Service Endpoints ограничивают доступ на уровне Azure backbone, но не делают приложение полностью приватным.
Hybrid Connections подходят для точечных подключений к on-prem, но не заменяют полноценную VNet Integration.
ASE используется, если требуется максимальная сетевая изоляция и контроль.

### Итог
- App Service Networking позволяет:
- Контролировать входящий и исходящий трафик.
- Ограничивать доступ по IP или VNet.
- Делать приложение полностью приватным.
- Интегрировать приложение с локальной инфраструктурой.
- Управлять исходящими IP-адресами.

️️⚠️ Правильный выбор механизма зависит от требований к безопасности, изоляции и архитектуре решения.

### When Outbound IPs Change
- ⚠️ **Scale between tiers** - Different VM families
- ⚠️ **Delete and recreate** - New resources
- ✅ **Scale within tier** - Same IPs
- ✅ **Scale out/in** - Same IPs

## Особенности сетевой модели по уровням (Tiers)

### Free и Shared

- Приложения работают на **общих (shared) multitenant worker-узлах**
- Ограниченные сетевые возможности
- Исходящие и входящие IP могут использоваться совместно с другими клиентами

⚠️ Эти уровни:
- используют общую инфраструктуру
- не предназначены для production
- имеют ограничения по сетевой изоляции и масштабированию

💡 Подходят только для разработки и тестирования.

---

### Basic и выше

- Приложения работают на **выделенных (dedicated) worker-узлах**
- Все приложения в одном App Service Plan используют одни и те же worker-инстансы
- Deployment slots работают на тех же worker-узлах
- Scale out означает добавление новых worker-инстансов

⚠️ Важно понимать:
- При масштабировании создаются дополнительные worker-VM.
- Все приложения в плане масштабируются одновременно.
- Ресурсы остаются общими для всех приложений внутри одного плана.

💡 Если требуется изоляция нагрузки или независимое масштабирование — создаётся отдельный App Service Plan.


## Access Restrictions

### Configure IP Restrictions

```bash
# Add IP restriction
az webapp config access-restriction add \
  --resource-group <rg-name> \
  --name <app-name> \
  --rule-name AllowOfficeIP \
  --action Allow \
  --ip-address 203.0.113.0/24 \
  --priority 100

# Add VNet restriction
az webapp config access-restriction add \
  --resource-group <rg-name> \
  --name <app-name> \
  --rule-name AllowVNet \
  --action Allow \
  --vnet-name <vnet-name> \
  --subnet <subnet-name> \
  --priority 200
```

### Priority Rules
- **Lower number** = higher priority
- **Allow or Deny** actions
- **Evaluated in order** until match

## VNet Integration

### Regional VNet Integration
```bash
# Enable VNet integration
az webapp vnet-integration add \
  --resource-group <rg-name> \
  --name <app-name> \
  --vnet <vnet-name> \
  --subnet <subnet-name>
```

## Преимущества VNet Integration

- ✅ Доступ к ресурсам внутри VNet (например, VM, базы данных)
- ✅ Доступ к on-prem инфраструктуре через ExpressRoute
- ✅ Поддержка Service Endpoints
- ✅ Возможность маршрутизации исходящего трафика через VNet

💡 VNet Integration управляет **исходящим (outbound)** трафиком приложения.

---

## Требования для VNet Integration

- Требуется **Standard tier или выше**
- Нужна выделенная подсеть (не может использоваться другими ресурсами)
- Подсеть должна иметь делегирование:


```bash
# Create private endpoint
az network private-endpoint create \
  --resource-group <rg-name> \
  --name <endpoint-name> \
  --vnet-name <vnet-name> \
  --subnet <subnet-name> \
  --private-connection-resource-id <app-resource-id> \
  --group-ids sites \
  --connection-name <connection-name>
```

## Преимущества

- ✅ Безопасный входящий доступ к приложению
- ✅ Отсутствие публичного доступа из интернета
- ✅ Доступ из VNet или из локальной инфраструктуры (on-premises)

💡 Такие преимущества обычно достигаются при использовании Private Endpoint.

---

# Hybrid Connections

## Ключевые концепции

- Позволяет подключаться к ресурсам:
    - в локальной инфраструктуре (on-premises)
    - в других сетях
- Работает только с TCP-эндпоинтами
- Не требует VPN или VNet Peering

---

## Когда использовать Hybrid Connections

Подходит для:

- Подключения к локальной базе данных
- Доступа к внутренним API
- Интеграции с legacy-системами
- Сценариев, где нет полноценной сетевой интеграции

---

## Ограничения

- Поддерживает только TCP (не UDP)
- Не заменяет полноценную VNet Integration
- Не обеспечивает полный сетевой контроль

---

## Экзаменационный акцент (AZ-204)

Если в задаче:

- требуется доступ к on-prem ресурсам
- не используется VPN
- нет VNet Peering
- нужен TCP-доступ

Ответ → Hybrid Connections.

Если требуется доступ к ресурсам внутри Azure VNet → VNet Integration.


```bash
# Requires Hybrid Connection Manager on-premises
# Configured through Azure Portal
```
## Типовые сценарии использования

- Доступ к локальному SQL Server (on-premises)
- Подключение к legacy-системам
- Доступ к TCP-эндпоинтам в других сетях

💡 Для этих сценариев чаще всего используется Hybrid Connections или VNet Integration (в зависимости от архитектуры).

---

## Быстрая матрица (Quick Reference Matrix)

| Функция | Требуемый Tier | Inbound / Outbound | Сценарий |
|-----------|----------------|---------------------|------------|
| Access Restrictions | Basic+ | Inbound | Whitelist IP-адресов |
| Service Endpoints | Basic+ | Inbound | Ограничение доступа из VNet |
| Private Endpoints | Basic+ | Inbound | Полностью приватный доступ |
| VNet Integration | Standard+ | Outbound | Доступ к ресурсам внутри VNet |
| Hybrid Connections | Basic+ | Outbound | Доступ к on-premises ресурсам |

---

## Критические замечания

- 💡 Outbound IP-адреса могут измениться при смене tier (изменение VM family).
- 🎯 Inbound-функции и Outbound-функции решают разные задачи.
- ⚠️ VNet Integration требует минимум Standard tier.
- 🔐 Private Endpoints делают приложение недоступным из интернета.
- 📊 Все приложения в одном App Service Plan используют одни и те же outbound IP.
- 🌍 В Multitenant-среде нет прямого подключения к VNet (кроме Isolated tier / ASE).

---

## Экзаменационные советы (AZ-204)

- Чётко различайте inbound и outbound механизмы.
- Помните, что outbound IP может измениться при смене tier.
- VNet Integration требует выделенной подсети.
- Private Endpoint = приватный IP внутри VNet, без публичного доступа.
- Access Restrictions позволяют фильтровать трафик по IP или VNet.
- Service Endpoint не делает приложение полностью приватным (в отличие от Private Endpoint).

---

## Частая ловушка на экзамене

Если в вопросе:
- требуется фиксированный outbound IP → нужен NAT Gateway.
- требуется полный приватный доступ → Private Endpoint.
- требуется доступ к локальной инфраструктуре → Hybrid Connections.



# Сравнение App Service Tiers: Networking и Isolation

| Характеристика | Free | Basic | Standard | Premium |
|----------------|------|--------|----------|----------|
| Тип инфраструктуры | Multitenant (shared) | Dedicated workers | Dedicated workers | Dedicated workers (улучшенные VM) |
| Изоляция от других клиентов | ❌ Нет (общие VM) | ✅ Да (выделенные VM в рамках плана) | ✅ Да | ✅ Да |
| Изоляция между приложениями | ❌ Делят общие ресурсы | ⚠️ Делят worker внутри плана | ⚠️ Делят worker внутри плана | ⚠️ Делят worker внутри плана |
| Deployment Slots | ❌ Нет | ❌ Нет | ✅ Да | ✅ Да |
| VNet Integration | ❌ Нет | ❌ Нет | ✅ Да | ✅ Да |
| Private Endpoint | ❌ Нет | ❌ Нет | ✅ Да | ✅ Да |
| Access Restrictions | ⚠️ Ограниченно | ✅ Да | ✅ Да | ✅ Да |
| Фиксированный outbound IP | ❌ Нет | ❌ Нет (требуется NAT Gateway) | ❌ Нет (требуется NAT Gateway) | ❌ Нет (требуется NAT Gateway) |
| Поддержка Hybrid Connections | ❌ Нет | ✅ Да | ✅ Да | ✅ Да |
| Масштабирование (Scale Out) | ❌ Нет | ✅ До 3 | ✅ До 10 | ✅ До 20+ |
| Поддержка Linux | ⚠️ Ограниченно | ✅ Да | ✅ Да | ✅ Да |
| Подходит для production | ❌ Нет | ⚠️ Базовый уровень | ✅ Да | ✅ Да (высокая нагрузка) |

---

## Ключевые выводы

### 🔴 Free
- Только для разработки и тестирования
- Нет сетевой изоляции
- Нет VNet Integration
- Нет deployment slots

---

### 🟡 Basic
- Выделенные VM (нет shared с другими клиентами)
- Нет deployment slots
- Нет VNet Integration
- Подходит для простого production без сложной сети

---

### 🟢 Standard
- Поддержка deployment slots
- Поддержка VNet Integration
- Подходит для production с сетевыми требованиями
- Чаще всего используется в реальных проектах

---

### 🔵 Premium
- Более производительные VM
- Больше инстансов
- Подходит для высоконагруженных приложений
- Поддержка всех сетевых функций Standard

---

## Экзаменационные акценты (AZ-204)

- Если в задаче требуется VNet Integration → минимум Standard
- Если требуется zero-downtime deployment → Standard+
- Если указана высокая производительность → Premium
- Если упоминается compliance и полная изоляция → рассмотреть Isolated (ASE)
- Free/Shared почти всегда неправильный ответ для production


[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-app-service/6-network-features)
