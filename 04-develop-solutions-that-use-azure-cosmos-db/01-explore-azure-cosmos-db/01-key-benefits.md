# Azure Cosmos DB Key Benefits (Ключевые преимущества Azure Cosmos DB)

## Key Concepts (Основные концепции)

- 🌍 **Глобальная распределённость** — репликация в несколько регионов с поддержкой multi-master (запись в нескольких регионах)
- ⚡ **Низкая задержка** — < 10 мс на 99-м перцентиле для чтения и записи
- 🛡 **Высокая доступность** — SLA до 99.999% для multi-region конфигураций
- 📈 **Эластичное масштабирование** — практически неограниченные throughput и объём хранения

---

## What is Azure Cosmos DB? (Что такое Azure Cosmos DB?)

**Полностью управляемая NoSQL-база данных** для современных облачных приложений.

### Основные характеристики

- ⚡ **Low latency** — однозначные миллисекунды ответа
- 📈 **Elastic scalability** — мгновенное масштабирование throughput
- 🌍 **Global distribution** — встроенная репликация по регионам
- 🧩 **Multi-model API** — поддержка разных API:
    - Core (SQL) API
    - MongoDB API
    - Cassandra API
    - Gremlin API
    - Table API
- 🛡 **Гарантированные SLA** — на доступность, задержку, throughput и консистентность

---

## Почему Cosmos DB часто выбирают

- Подходит для глобальных SaaS-приложений
- Обеспечивает предсказуемую производительность (через RU/s модель)
- Позволяет выбрать уровень консистентности
- Не требует управления инфраструктурой

---

> 🎯 Экзаменационный момент AZ-204:  
> Cosmos DB предоставляет SLA не только на доступность,  
> но и на latency, throughput и consistency — это уникальная особенность сервиса.


### Core Value Proposition
```
Traditional Database Azure Cosmos DB
├── Single region →  ├── Multi-region (готовая глобальная репликация)
├── Fixed capacity → ├── Неограниченное эластичное масштабирование
├── Higher latency → ├── < 10 мс @ P99
├── Manual scaling → ├── Автоматическое масштабирование
└── 99.9% SLA      → └── До 99.999% SLA
```

## Global Distribution Benefits

### Multi-Master Replication

## Novel Replication Protocol (Инновационный протокол репликации)

Azure Cosmos DB использует собственный протокол репликации, который обеспечивает:

✅ **Неограниченное эластичное масштабирование записи**  
— Возможность записи в любой регион (multi-master)

✅ **Неограниченное эластичное масштабирование чтения**  
— Чтение из любого региона

✅ **99.999% доступность**  
— Гарантированная доступность операций чтения и записи по всему миру

✅ **< 10 мс задержка**  
— Гарантия на 99-м перцентиле

✅ **Мгновенный failover**  
— Автоматическое переключение при сбое (multi-homing)

---

### Почему это важно

- Приложения остаются доступными даже при отказе региона
- Пользователи подключаются к ближайшему региону
- Нет необходимости вручную настраивать репликацию

---

> 🎯 Экзаменационный момент AZ-204:  
> Cosmos DB поддерживает multi-region writes и гарантирует SLA на latency и availability.


### How Global Distribution Works

```
User Request
     ↓
Nearest Region (Auto-selected)
     ↓
Read: Served from nearest replica
Write: Replicated to all regions
     ↓
Consistency Level Applied
     ↓
Response <10ms @ P99
```

### Multi-Region Configuration (Конфигурации по регионам)

| Configuration | Reads | Writes | Availability | Use Case |
|---------------|--------|---------|--------------|-----------|
| **Single region** | Один регион | Один регион | 99.99% | Разработка, минимальные затраты |
| **Multi-region, single write** | Все регионы | Один регион | 99.99% | Нагрузка с преобладанием чтения |
| **Multi-region, multi-write** | Все регионы | Все регионы | 99.999% | Критически важные приложения |

---

### Объяснение сценариев

🔹 **Single region**
- Самая простая конфигурация
- Нет глобальной отказоустойчивости
- Подходит для dev/test

🔹 **Multi-region, single write**
- Чтение из ближайшего региона
- Запись только в primary регион
- Хорошо подходит для read-heavy workloads

🔹 **Multi-region, multi-write (multi-master)**
- Чтение и запись в любом регионе
- Максимальная доступность (99.999%)
- Подходит для глобальных SaaS и mission-critical систем

---

> 🎯 Экзаменационный момент AZ-204:  
> 99.999% SLA достигается при **multi-region + multi-write** конфигурации.


### Add/Remove Regions

**Dynamic region management** without downtime:

```bash
# Add region
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --locations regionName=eastus failoverPriority=0 isZoneRedundant=False \
  --locations regionName=westus failoverPriority=1 isZoneRedundant=False

# Application continues running - no pause/redeploy needed
```
### Key Features (Ключевые особенности)

- ✅ Без простоя приложения (no downtime)
- ✅ Не требуется redeployment
- ✅ Автоматическая репликация между регионами
- ✅ Мгновенная доступность чтения/записи в новом регионе

---

## Availability Guarantees (Гарантии доступности)

### SLA Comparison (Сравнение SLA)

| Database Type | Read Availability | Write Availability |
|---------------|-------------------|-------------------|
| **Single region** | 99.99% | 99.99% |
| **Multi-region (single write)** | 99.999% | 99.99% |
| **Multi-region (multi-write)** | 99.999% | 99.999% |

---

### Downtime Calculation (Расчёт простоя)

- **99.999% availability** ≈ ~5 минут простоя в год
- **99.99% availability** ≈ ~52 минуты простоя в год

---

### Что это означает на практике

- Multi-region увеличивает **read availability**
- Multi-write увеличивает **write availability**
- Для mission-critical систем выбирают multi-region + multi-write

---

> 🎯 Экзаменационный момент AZ-204:  
> 99.999% write SLA достигается только при multi-region + multi-write конфигурации.

### Automatic Failover

```
Primary Region Fails
     ↓
Automatic Detection (<5 seconds)
     ↓
Traffic Routed to Next Region
     ↓
Client Auto-Reconnects
     ↓
Zero Data Loss (depending on consistency)
```

### Failover Features (Особенности failover)

- **Automatic** — переключение происходит без ручного вмешательства
- **Transparent** — SDK автоматически переподключается к новому региону
- **Configurable** — можно задать приоритеты регионов (failover priority)
- **Zero downtime** — переход выполняется без остановки приложения

---

## Performance Guarantees (Гарантии производительности)

### Latency SLA (SLA по задержке)

| Operation | Latency Guarantee | Percentile |
|------------|------------------|------------|
| **Point read** | < 10 мс | 99th |
| **Write** | < 10 мс | 99th |
| **Query** | Зависит от сложности | N/A |

---

### Что такое Point Read?

**Point read** — это получение одного документа по:

- `id`
- `partition key`

Это самая быстрая и дешёвая операция в Cosmos DB (минимальное потребление RU).

---

### Важно понимать

- SLA < 10 мс применяется к point read и write
- Query не имеет фиксированного SLA по latency
- Производительность напрямую зависит от выбора partition key

---

> 🎯 Экзаменационный момент AZ-204:  
> Самая производительная операция — **point read (id + partition key)**  
> SLA < 10 мс гарантируется на 99-м перцентиле.

### Throughput Scalability

```
Application Growth
     ↓
Increase RU/s (Request Units per second)
     ↓
Instant Scale (no downtime)
     ↓
Linear performance increase
```

### Elastic Scalability Characteristics (Характеристики эластичного масштабирования)

- **Unlimited** — практически нет верхнего предела по throughput (RU/s)
- **Instant** — масштабирование вверх или вниз за секунды
- **Elastic** — поддержка autoscale в зависимости от нагрузки
- **Granular** — можно выделять throughput на уровне контейнера или базы данных

---

### Что это означает на практике

- RU/s можно изменять без остановки приложения
- Autoscale автоматически увеличивает RU при росте нагрузки
- Можно изолировать нагрузку, назначая RU отдельным контейнерам
- Подходит для burst-нагрузок и глобальных приложений

---

> 🎯 Экзаменационный момент AZ-204:  
> Throughput в Cosmos DB измеряется в **RU/s**  
> и может быть назначен на уровне **container** или **database**.


## Data Placement Strategy

### Place Data Near Users

**Reduce latency** by geographic proximity:

```
Users in Asia → Asia Region
Users in Europe → Europe Region
Users in US → US Regions
     ↓
All data replicated across regions
     ↓
Reads: Local (fast)
Writes: Replicated (async)
```

### Choosing Regions

### Factors to Consider (Факторы при выборе регионов)

1. **User location**  
   — Где находятся ваши пользователи?  
   Размещайте данные ближе к пользователям для минимальной задержки.

2. **Data residency**  
   — Есть ли юридические требования к хранению данных?  
   Некоторые страны требуют хранение данных внутри своей территории.

3. **Paired regions**  
   — Используйте региональные пары Azure для Disaster Recovery.  
   Это повышает устойчивость к сбоям на уровне региона.

4. **Cost**  
   — Стоимость отличается в зависимости от региона.  
   Multi-region развертывание увеличивает расходы.

---

### Практический подход

- Выберите primary регион рядом с основной аудиторией
- Добавьте secondary регион для DR
- Для mission-critical систем — включайте multi-write
- Проверяйте compliance и требования по резидентности данных

---

> 🎯 Экзаменационный момент AZ-204:  
> Регион выбирается с учётом latency, compliance, DR и стоимости.


**Example strategy**:
```bash
# Global application
Primary: East US (HQ location)
Secondary: West Europe (EU users)
Secondary: Southeast Asia (APAC users)

# Result: <50ms latency for 95% of global users
```

## Elastic Scalability

### Throughput Scaling

**Provision Request Units (RU/s)**:

```bash
# Start small
Initial: 400 RU/s

# Scale up instantly
Peak traffic: 100,000 RU/s

# Scale down after peak
Off-peak: 1,000 RU/s

# Pay only for provisioned throughput
```
### Throughput Modes (Режимы настройки throughput)

**Modes:**

- **Manual (Provisioned Throughput)**  
  — Вы вручную задаёте фиксированное значение RU/s.  
  Подходит для предсказуемой нагрузки.

- **Autoscale**  
  — Автоматическое масштабирование RU/s в пределах min/max.  
  Хорошо подходит для переменной или burst-нагрузки.

- **Serverless**  
  — Оплата за фактические запросы (без предварительного provision).  
  Подходит для нерегулярной или низкой нагрузки.

---

### Когда что использовать

- 📊 Стабильная нагрузка → Manual
- 📈 Переменная нагрузка → Autoscale
- 🧪 Dev/Test или редкие запросы → Serverless

---

> 🎯 Экзаменационный момент AZ-204:  
> RU/s применяется в Manual и Autoscale режимах,  
> Serverless не требует предварительного выделения throughput.


### Storage Scaling

**Unlimited storage per container**:

```
Data Growth: 1 GB → 1 TB → 1 PB
     ↓
Automatic partitioning
     ↓
No manual intervention
     ↓
Consistent performance maintained
```

## Multi-Model Support (Поддержка нескольких моделей данных)

### Supported APIs (Поддерживаемые API)

| API | Use Case | Data Model |
|------|------------|-------------|
| **NoSQL (Core/SQL)** | Современные cloud-приложения | Document (JSON) |
| **MongoDB** | Миграция существующих MongoDB-приложений | Document |
| **PostgreSQL** | Реляционные нагрузки (на базе Citus) | Relational |
| **Cassandra** | Wide-column, высоконагруженные системы | Column-family |
| **Gremlin** | Графовые связи | Graph |
| **Table** | Key-value, миграция с Azure Table | Key-value |

---

### Benefit (Преимущество)

- Можно использовать привычный API
- Минимальные изменения в коде при миграции
- Нет жёсткой привязки к конкретному движку

> 💡 Cosmos DB предоставляет разные API поверх одной глобально распределённой инфраструктуры.

---

## Cost Model (Модель оплаты)

### Pay for What You Use (Оплата за фактическое использование)

Стоимость рассчитывается по двум основным параметрам:

1️⃣ **Throughput**  
— Provisioned RU/s (почасовая тарификация)

2️⃣ **Storage**  
— Фактически использованные GB (помесячная тарификация)

---

### Что важно учитывать

- RU/s — главный фактор стоимости при высокой нагрузке
- Storage оплачивается отдельно
- Multi-region увеличивает стоимость (репликация данных)
- Autoscale стоит дороже, но снижает риск throttling

---

> 🎯 Экзаменационный момент AZ-204:  
> Cosmos DB тарифицируется по двум измерениям:  
> **RU/s + Storage (GB)**.


**Example**:
```
Container: 1,000 RU/s ($0.008/hour) = $5.76/month
Storage: 100 GB ($0.25/GB) = $25/month
Total: ~$31/month
```

### Free Tier (Бесплатный уровень)

**Для первого Azure Cosmos DB аккаунта:**

- ✅ Первые **1000 RU/s** бесплатно
- ✅ Первые **25 GB хранения** бесплатно
- ✅ Предложение действует бессрочно (lifetime offer)

> 💡 Отличный вариант для pet-проектов, обучения и подготовки к экзамену.

---

## Developer Experience (Удобство разработки)

### SDKs Available (Доступные SDK)

**Поддерживаемые языки:**

- .NET / .NET Core
- Java
- Python
- Node.js
- Go

---

### Tools (Инструменты)

- Azure Portal
- Azure CLI
- Azure PowerShell
- Расширение для VS Code
- **Data Explorer** (встроенный браузерный инструмент)

---

### Что это даёт разработчику

- Быстрое прототипирование
- Полноценная работа через SDK
- Возможность тестировать запросы прямо в браузере
- Интеграция с CI/CD через CLI и PowerShell

---

> 🎯 Экзаменационный момент AZ-204:  
> Free Tier предоставляет **1000 RU/s + 25 GB** бесплатно для первого аккаунта Cosmos DB.


### Sample Connection

```csharp
// Connect to Cosmos DB
var client = new CosmosClient(
    "https://myaccount.documents.azure.com:443/",
    "your-primary-key"
);

// Get database and container
var database = client.GetDatabase("mydb");
var container = database.GetContainer("mycollection");

// Read item (<10ms)
var item = await container.ReadItemAsync<Product>(
    "product-123",
    new PartitionKey("electronics")
);
```

## Use Cases (Сценарии использования)

### Ideal Workloads (Идеальные нагрузки)

✅ **Web / Mobile приложения** — низкая задержка и глобальный масштаб  
✅ **IoT** — большое количество записей, time-series данные  
✅ **Игровые платформы** — профили игроков, лидерборды  
✅ **Retail** — каталоги товаров, корзины  
✅ **Финансовые сервисы** — транзакции в реальном времени  
✅ **Content management** — статьи, метаданные медиа

---

## When to Use Cosmos DB (Когда выбирать Cosmos DB)

| Requirement | Cosmos DB Solution |
|-------------|-------------------|
| Глобальная аудитория | Multi-region репликация |
| Низкая задержка | < 10 мс @ P99 |
| Переменная нагрузка | Autoscale RU/s |
| 99.999% uptime | Multi-write регионы |
| Гибкая схема | NoSQL (JSON документы) |
| Масштабирование | Практически неограниченные RU/s и storage |

---

## When NOT to Use (Когда не стоит использовать)

❌ **Малые объёмы данных (<10 GB)** — может быть экономически невыгодно  
❌ **Сложные транзакции** — ограниченная поддержка multi-document транзакций  
❌ **On-premises требования** — сервис полностью облачный  
❌ **Чисто реляционная модель** — лучше Azure SQL (если не подходит PostgreSQL API)

---

## Critical Notes (Ключевые моменты)

- 💡 Полностью управляемый сервис — без серверов и патчей
- 🌍 Multi-master — запись в любой регион
- 🛡 99.999% SLA — при multi-region + multi-write
- ⚡ < 10 мс — SLA на 99-м перцентиле
- 🔄 Эластичное масштабирование RU/s
- 🌐 Добавление/удаление регионов без downtime
- 🧩 6 API — выбор подходящей модели
- 🎁 Free tier — 1000 RU/s + 25 GB бесплатно (первый аккаунт)

---

## Exam Tips (Советы к экзамену AZ-204)

- Cosmos DB — глобально распределённая управляемая NoSQL БД
- Multi-master — запись и чтение в любом регионе
- SLA 99.999% — только при multi-region + multi-write
- Latency SLA — < 10 мс @ P99 (read/write)
- Throughput измеряется в **RU/s**
- Глобальная репликация — без redeployment
- 5 уровней консистентности (Strong → Eventual)
- Поддерживаемые API: NoSQL, MongoDB, PostgreSQL, Cassandra, Gremlin, Table
- Free tier: 1000 RU/s + 25 GB
- Стоимость = RU/s + Storage
- Режимы throughput: Manual, Autoscale, Serverless
- **Point read** (id + partition key) — самая быстрая операция (~1 RU для 1KB)
- Автоматический failover — прозрачный для приложения
- Поддержка SDK: .NET, Java, Python, Node.js, Go
- Масштабирование происходит мгновенно, без простоя

---

Azure Storage Explorer ❌
→ Работает с Blob / Table / Queue, не Mongo.

AzCopy ❌
→ Копирует blob-файлы, не Mongo базы.

No change required ❌
→ Data Management Gateway тут не подходит.

Cosmos DB with MongoDB API
Mongo tools работают
Можно использовать mongodump/mongorestore


| API           | Тип модели  | Когда выбирать    |
| ------------- | ----------- | ----------------- |
| SQL API       | Document    | Новая разработка  |
| Mongo API     | Document    | Миграция Mongo    |
| Cassandra API | Wide-column | Time-series / IoT |
| Gremlin API   | Graph       | Связанные данные  |


| Нужно узнать | Команда                |
| ------------ | ---------------------- |
| Publisher    | Get-AzVMImagePublisher |
| Offer        | Get-AzVMImageOffer     |
| SKU          | Get-AzVMImageSku       |
| Image        | Get-AzVMImage          |

AD Connect — это синхронизация on-prem AD с Azure AD.

| SKU      | Storage  | Performance  | Geo-replication | Private endpoint |
| -------- | -------- | ------------ | --------------- | ---------------- |
| Basic    | 10–20 GB | Низкая       | ❌               | ❌                |
| Standard | 100 GB   | Выше         | ❌               | ❌                |
| Premium  | 500 GB   | Максимальная | ✅               | ✅                |



Если для оптимизации нужен composite index, то:
в Azure Portal → Query → Index Advisor
система предложит добавить соответствующий composite index
его можно применить автоматически


В Azure Cosmos DB (SQL API) UDF:
выполняются на уровне запроса
не используют индекс
обрабатываются построчно (per document)
Если UDF используется в WHERE, например:
SELECT * FROM c
WHERE udf.isValid(c.Name)
то:
Cosmos DB сначала считывает документы
затем применяет UDF
индекс при этом не используется эффективно
увеличивается потребление RU
запрос становится медленным и дорогим
Поэтому Microsoft рекомендует избегать UDF в WHERE, если это влияет на фильтрацию больших объёмов данных.

Change Feed Processor library
Это рекомендуемый механизм для production-сценариев:
автоматически распределяет partition между несколькими host-инстансами
обеспечивает масштабируемую параллельную обработку
хранит lease-информацию
устойчив к сбоям
поддерживает replay

Глобальный порядок требует централизованной координации.
Централизация убивает масштабирование.

Cosmos DB — это massively distributed система.
Поэтому она гарантирует порядок там, где это дешёво — внутри partition.

❌ A. Shared + autoscale на каждом контейнере
Autoscale на shared не применяется “на каждый контейнер” — autoscale применяется к shared throughput на уровне базы.

🟢 Почему Dedicated throughput (вариант B) правильный
Dedicated throughput означает:
у каждого контейнера свой RU пул
нет конкуренции
гарантированная производительность
предсказуемый latency
Да, это дороже.


Как работают уровни консистентности в Cosmos DB
От самого строгого к самому слабому:
Strong

Bounded staleness

Session

Consistent prefix

Eventual ← самый слабый

Eventual consistency:
❌ Не гарантирует порядок
❌ Не гарантирует read-your-writes
✅ Даёт максимальную доступность
✅ Минимальную задержку
✅ Лучший SLA по availability

| Требование            | Ответ    |
| --------------------- | -------- |
| Max throughput        | Eventual |
| Highest availability  | Eventual |
| No ordering guarantee | Eventual |
| Read-your-writes      | Session  |
| Strict consistency    | Strong   |

| Уровень               | Гарантии                                       | Latency          | Throughput      | Availability    | Когда использовать    |
| --------------------- | ---------------------------------------------- | ---------------- | --------------- | --------------- | --------------------- |
| **Strong**            | Полная линейная согласованность                | 🔴 Самая высокая | 🔴 Самый низкий | Ниже            | Банковские транзакции |
| **Bounded Staleness** | Ограниченная задержка (K versions / T seconds) | 🔴 Высокая       | 🔴 Низкий       | Ниже            | Финансовые системы    |
| **Session**           | Read-your-writes                               | 🟡 Средняя       | 🟡 Хороший      | Высокая         | Web / mobile apps     |
| **Consistent Prefix** | Порядок операций сохраняется                   | 🟢 Низкая        | 🟢 Высокий      | Очень высокая   | Логи, события         |
| **Eventual**          | Нет гарантий порядка                           | 🟢 Самая низкая  | 🟢 Максимальный | 🟢 Максимальная | IoT, analytics        |

1️⃣ Monitored container
– Это контейнер, за которым мы следим (где данные меняются)

2️⃣ Lease container
– Это контейнер, который:
Хранит состояние обработки
Отслеживает progress
Делит работу между несколькими инстансами

| Компонент           | Назначение                |
| ------------------- | ------------------------- |
| Monitored container | Источник изменений        |
| Lease container     | Хранит checkpoint / state |
| Delegate            | Твоя бизнес-логика        |
| Compute instance    | Где исполняется код       |


Cosmos DB Change Feed tuning =
игра с параметрами:
feedPollDelay
lease configuration
batch size
[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-cosmos-db/2-cosmos-db-benefits)
