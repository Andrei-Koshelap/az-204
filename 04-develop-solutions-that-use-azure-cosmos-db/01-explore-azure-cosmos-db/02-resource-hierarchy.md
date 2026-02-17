# Azure Cosmos DB Resource Hierarchy
(Иерархия ресурсов Azure Cosmos DB)

## Key Concepts (Ключевые элементы иерархии)

- **Account**  
  — Верхний уровень ресурса с уникальным DNS-именем  
  (например: `https://myaccount.documents.azure.com`)

- **Database**  
  — Логическое пространство имён для контейнеров

- **Container**  
  — Основная единица масштабирования  
  (throughput и storage настраиваются на уровне container или database)

- **Items**  
  — Отдельные сущности данных  
  (документы, строки, узлы — в зависимости от API)

- **Partition key**  
  — Критически важный элемент для распределения данных и масштабирования

---

## Визуальная иерархия

```
Azure Subscription
     ↓
Azure Cosmos DB Account (myaccount.documents.azure.com)
     ↓
Database (logical namespace)
     ├── Container 1 (unlimited RU/s & storage)
     │   ├── Partition Key: /category
     │   ├── Item 1 (document/row/node)
     │   ├── Item 2
     │   └── Item n
     ├── Container 2
     └── Container n
```

## Что важно понимать

- **Account** может быть multi-region
- **Container** — это единица масштабирования (RU/s распределяется по partition)
- **Items** хранятся в формате JSON (для NoSQL API)
- **Partition key** определяет, как данные распределяются по физическим partition

---

> 🎯 Экзаменационный момент AZ-204:
> Throughput (RU/s) применяется на уровне **container**  
> и масштабирование напрямую зависит от правильного выбора **partition key**.


### Hierarchy Levels (Уровни иерархии)

| Level | Description | Purpose | Limit |
|--------|------------|----------|--------|
| **Subscription** | Биллинговая единица Azure | Организация ресурсов | Много аккаунтов |
| **Account** | Единица глобального распределения | DNS, регионы | 50 на подписку* |
| **Database** | Пространство имён | Логическая группировка | Неограничено |
| **Container** | Единица масштабирования | Хранение данных | Неограничено |
| **Item** | Сущность данных | Фактические данные | Неограничено |

\*Лимит можно увеличить через запрос в поддержку.

---

## Azure Cosmos DB Account
(Аккаунт Cosmos DB)

### What is an Account? (Что такое Account?)

**Верхнеуровневый ресурс**, отвечающий за глобальное распределение и конфигурацию базы.

### Основные характеристики

- 🌐 **Уникальное DNS-имя**  
  `https://<account-name>.documents.azure.com`

- 🌍 **Global distribution**  
  Управление регионами и репликацией

- 🛡 **High availability**  
  Настройка failover и приоритетов регионов

- ⚖ **Consistency settings**  
  Уровень консистентности по умолчанию задаётся на уровне аккаунта

- 🔌 **API selection**  
  API (NoSQL, MongoDB, PostgreSQL, Cassandra, Gremlin, Table) выбирается при создании аккаунта

---

### Важно помнить

- Регион можно добавить или удалить без downtime
- API нельзя изменить после создания аккаунта
- SLA зависит от конфигурации регионов (single vs multi-region)

---

> 🎯 Экзаменационный момент AZ-204:
> API выбирается **при создании аккаунта** и не может быть изменён позже.


### Account Properties

```json
{
  "name": "mycosmosaccount",
  "location": "East US",
  "kind": "GlobalDocumentDB",
  "databaseAccountOfferType": "Standard",
  "locations": [
    {
      "locationName": "East US",
      "failoverPriority": 0
    },
    {
      "locationName": "West US",
      "failoverPriority": 1
    }
  ],
  "consistencyPolicy": {
    "defaultConsistencyLevel": "Session"
  }
}
```

### Create Account

```bash
# Create Cosmos DB account
az cosmosdb create \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --locations regionName=eastus failoverPriority=0 \
  --locations regionName=westus failoverPriority=1 \
  --default-consistency-level Session \
  --enable-multiple-write-locations true

# Account limits per subscription
Default: 50 accounts
Can increase: Submit support request
```

### Account Management Tools (Инструменты управления аккаунтом)

| Tool | Use Case |
|--------|------------|
| **Azure Portal** | Графическое управление (GUI) |
| **Azure CLI** | Автоматизация через командную строку |
| **PowerShell** | Скрипты и администрирование в Windows |
| **.NET SDK** | Программное управление |
| **Java SDK** | Программное управление |
| **Python SDK** | Программное управление |

---

## Azure Cosmos DB Database
(База данных Cosmos DB)

### What is a Database? (Что такое Database?)

**Логическое пространство имён** для контейнеров.

### Основные функции

- 📂 **Группировка контейнеров** — объединение связанных данных
- ⚡ **Shared throughput (опционально)** — общий пул RU/s для нескольких контейнеров
- 🧱 **Логическая граница управления** — единица администрирования
- 🧩 **Schema-less** — отсутствует жёсткая схема

---

### Database Characteristics (Характеристики базы данных)

✅ Неограниченное количество баз данных в одном аккаунте  
✅ Неограниченное количество контейнеров в базе  
✅ Данные хранятся только в контейнерах  
✅ Возможность выделить общий throughput для всех контейнеров базы

---

### Важно понимать

- Throughput можно назначить:
    - на уровне **container**
    - или на уровне **database (shared throughput)**
- Shared throughput подходит для небольших контейнеров с переменной нагрузкой

---

> 🎯 Экзаменационный момент AZ-204:
> Данные хранятся в **container**,  
> database — это логическая единица группировки.


### Database vs Container Throughput

#### Shared Throughput (Database-level)
```
Database: 1,000 RU/s (shared)
├── Container 1 (shares 1,000 RU/s)
├── Container 2 (shares 1,000 RU/s)
└── Container 3 (shares 1,000 RU/s)

Total cost: 1,000 RU/s
Use case: Many small containers, variable traffic
```

#### Dedicated Throughput (Container-level)
```
Database (no throughput)
├── Container 1: 400 RU/s (dedicated)
├── Container 2: 1,000 RU/s (dedicated)
└── Container 3: 10,000 RU/s (dedicated)

Total cost: 11,400 RU/s
Use case: Predictable per-container needs
```

### Create Database

```bash
# Create database
az cosmosdb sql database create \
  --account-name mycosmosaccount \
  --resource-group myResourceGroup \
  --name mydb

# Create database with shared throughput
az cosmosdb sql database create \
  --account-name mycosmosaccount \
  --resource-group myResourceGroup \
  --name mydb \
  --throughput 1000

# Create database with autoscale
az cosmosdb sql database create \
  --account-name mycosmosaccount \
  --resource-group myResourceGroup \
  --name mydb \
  --max-throughput 4000
```

## Azure Cosmos DB Container
(Контейнер Cosmos DB)

### What is a Container? (Что такое контейнер?)

**Основная единица масштабирования и хранения данных.**

### Основные возможности

- ⚡ **Неограниченный throughput** — выделение RU/s
- 💾 **Неограниченное хранение** — автоматическое партиционирование
- 🔑 **Partition key** — обязательна для распределения данных
- 📊 **Indexing policy** — настройка индексации
- ⏳ **TTL (Time-to-Live)** — автоматическое удаление данных

---

## Container Characteristics (Характеристики контейнера)

✅ **Schema-agnostic** — отсутствует фиксированная схема  
✅ **Автоматическое партиционирование** — на основе partition key  
✅ **Независимое масштабирование** — RU/s и storage на уровне контейнера  
✅ **API-специфичное название** — зависит от выбранного API

---

## Container Naming by API (Название контейнера в разных API)

| API | Container Name |
|------|----------------|
| **NoSQL (Core/SQL)** | Container |
| **MongoDB** | Collection |
| **Cassandra** | Table |
| **Gremlin** | Graph |
| **Table** | Table |

---

## Partition Key (Ключ партиционирования)

🔴 **Критически важное решение**  
Выбирается при создании контейнера и **не может быть изменён позже**.

### Роль Partition Key

- Определяет распределение данных по физическим partition
- Влияет на производительность
- Влияет на стоимость (RU consumption)
- Влияет на масштабируемость

---

### Правильный partition key должен:

- Иметь высокую кардинальность
- Равномерно распределять нагрузку
- Использоваться в большинстве запросов

---

> 🎯 Экзаменационный момент AZ-204:
> Partition key выбирается при создании контейнера  
> и изменить его позже нельзя.


```json
// Container with partition key
{
  "id": "products",
  "partitionKey": {
    "paths": ["/category"],
    "kind": "Hash"
  }
}

// Items distributed by category
{
  "id": "product-1",
  "category": "electronics",  // ← Partition key value
  "name": "Laptop"
}
```

## Partition Key Best Practices (Лучшие практики выбора partition key)

- ✅ **Высокая кардинальность** — много уникальных значений
- ✅ **Равномерное распределение** — баланс хранения и нагрузки
- ✅ **Соответствие шаблону запросов** — фильтрация по partition key
- ❌ Избегать **hot partitions** — неравномерной нагрузки на одну партицию

---

## Physical vs Logical Partitions
(Физические и логические партиции)

### Physical Partition
(Физическая партиция — управляется Azure)

- 📦 **Фиксированный размер** — до 50 GB хранения
- ⚡ **Фиксированный throughput** — до 10 000 RU/s
- 🔄 **Создаётся автоматически** по мере роста данных
- 👻 **Прозрачна для пользователя** — напрямую не управляется

---

### Logical Partition
(Логическая партиция — определяется пользователем)

- 📦 **Максимум 20 GB** на одно уникальное значение partition key
- 📚 **Группировка данных** — все items с одинаковым ключом вместе
- 🚀 **Быстрые запросы** внутри одной партиции
- 🔒 **Транзакции** ограничены одной логической партицией

---

## Важно понимать

- Logical partition определяется значением partition key
- Несколько logical partition размещаются внутри physical partition
- Если logical partition превышает 20 GB — возникнет ошибка
- Hot partition возникает, если один ключ получает большую часть трафика

---

> 🎯 Экзаменационный момент AZ-204:
> - Physical partition: 50 GB + 10 000 RU/s
> - Logical partition: 20 GB на одно значение partition key
> - Транзакции возможны только в пределах одной logical partition.


```
Partition Key: /category
     ↓
Logical Partitions:
├── category=electronics (15 GB) → Physical Partition 1
├── category=books (8 GB) → Physical Partition 2
├── category=clothing (18 GB) → Physical Partition 3
└── category=toys (5 GB) → Physical Partition 4
```
> 💡 Замечание:  
> Это иллюстрация принципа. В реальности Cosmos DB может размещать несколько logical partitions
> в одной physical partition и перераспределять их по мере роста данных и throughput.


### Throughput Provisioning Modes

#### 1. Dedicated Throughput (Container-level)
```bash
# Create container with dedicated RU/s
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/category" \
  --throughput 1000

# Exclusively for this container
# More expensive but guaranteed
```

**Types**:
- **Standard (manual)** - Fixed RU/s
- **Autoscale** - Auto-scale between min (10% of max) and max

#### 2. Shared Throughput (Database-level)
```bash
# Create container sharing database throughput
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/category"

# Shares RU/s with up to 25 containers
# More cost-effective
# Excludes containers with dedicated throughput
```

### Shared Throughput Limitations (Ограничения общего throughput)

⚠️ Максимум **25 контейнеров** могут использовать общий RU/s

⚠️ Контейнеры с **dedicated throughput** не участвуют в sharing

⚠️ База данных должна быть создана с включённым **shared throughput**

---

### Важно понимать

- Shared throughput распределяется динамически между контейнерами
- Если один контейнер потребляет больше RU — другие получают меньше
- Для изолированной нагрузки лучше использовать **dedicated throughput**

---

> 🎯 Экзаменационный момент AZ-204:
> Shared throughput доступен только при создании database  
> и ограничен 25 контейнерами.

### Create Container

```bash
# Basic container
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/id" \
  --throughput 400

# Container with autoscale
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/category" \
  --max-throughput 4000

# Container with TTL
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/deviceId" \
  --throughput 1000 \
  --ttl 3600  # Expire after 1 hour
```

### Container Settings (Настройки контейнера)

| Setting | Purpose | Changeable |
|-----------|------------|------------|
| **Partition key** | Распределение данных | ❌ Нет |
| **Throughput** | Выделение RU/s | ✅ Да |
| **Indexing policy** | Конфигурация индексов | ✅ Да |
| **TTL** | Авто-удаление данных | ✅ Да |
| **Unique keys** | Обеспечение уникальности | ❌ Нет |
| **Conflict resolution** | Разрешение конфликтов (multi-write) | ❌ Нет |

---

### Важно помнить

- Partition key и Unique keys выбираются при создании контейнера
- TTL можно включить или изменить позже
- Throughput можно масштабировать в любой момент
- Conflict resolution задаётся только при создании

---

## Azure Cosmos DB Items
(Элементы данных Cosmos DB)

### What are Items? (Что такое Items?)

**Отдельные сущности данных**, хранящиеся в контейнерах.

В зависимости от API это могут быть:

- 📄 **JSON-документы** (NoSQL API)
- 📦 **BSON-документы** (MongoDB API)
- 📊 **Строки (Rows)** (Cassandra API)
- 🔗 **Вершины/Рёбра** (Gremlin API)
- 🗂 **Сущности (Entities)** (Table API)

---

## Item Representation by API (Представление данных)

| API | Item Type | Example |
|------|------------|------------|
| **NoSQL** | JSON document | `{"id": "1", "name": "John"}` |
| **MongoDB** | BSON document | `{_id: ObjectId(), name: "John"}` |
| **Cassandra** | Row | `id=1, name='John'` |
| **Gremlin** | Vertex/Edge | `g.V('1').property('name','John')` |
| **Table** | Entity | `PartitionKey='pk1', RowKey='1', Name='John'` |

---

### Важно понимать

- Каждый item должен иметь `id` (или `_id` для MongoDB)
- Item всегда принадлежит конкретной logical partition
- Размер одного item ограничен 2 MB (для NoSQL API)

---

> 🎯 Экзаменационный момент AZ-204:
> - Данные хранятся в **container**
> - Item — это минимальная единица хранения
> - Partition key обязателен для каждого item


### Item Properties

**Required properties** (NoSQL API):

```json
{
  "id": "unique-id",           // Required: Unique within partition
  "category": "electronics",   // Partition key value
  "name": "Laptop",            // Custom properties
  "price": 999.99,
  "_rid": "...",               // System: Resource ID
  "_self": "...",              // System: Self-link
  "_etag": "...",              // System: Concurrency
  "_ts": 1640000000            // System: Timestamp (epoch)
}
```
## System Properties (Системные свойства items)

Системные свойства автоматически добавляются Cosmos DB  
и имеют префикс `_`:

- `_rid` — уникальный Resource ID
- `_self` — self-link URI
- `_etag` — контроль конкуренции (optimistic locking)
- `_ts` — timestamp последнего изменения (Unix time)
- `_attachments` — ссылка на вложения (устаревшее, deprecated)

> 💡 `_etag` используется для реализации optimistic concurrency  
> через заголовок `If-Match`.

---

## Item Size Limits (Ограничения размера item)

| Item Property | Limit |
|---------------|-------|
| **Max item size** | 2 MB |
| **Max property name length** | Официального лимита нет (разумные значения) |
| **Max property nesting** | 128 уровней |
| **Max array length** | Нет отдельного лимита (в пределах 2 MB) |

---

### Важно учитывать

- 2 MB — ограничение на весь JSON-документ
- Глубокая вложенность может повлиять на RU consumption
- Большие документы увеличивают стоимость операций (RU)

---

> 🎯 Экзаменационный момент AZ-204:
> - Максимальный размер item — **2 MB**
> - `_etag` используется для optimistic concurrency
> - `_ts` — Unix timestamp последнего изменения

### CRUD Operations

```csharp
// Create item
var product = new Product
{
    Id = "product-1",
    Category = "electronics",
    Name = "Laptop",
    Price = 999.99m
};
await container.CreateItemAsync(product, new PartitionKey("electronics"));

// Read item (point read - 1 RU for 1KB)
var response = await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics")
);

// Update item
product.Price = 899.99m;
await container.UpsertItemAsync(product, new PartitionKey("electronics"));

// Delete item
await container.DeleteItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics")
);
```

## Hierarchy Best Practices

### 1. Account Planning
```
Production: prod-cosmos-account
Development: dev-cosmos-account
Testing: test-cosmos-account

Reason: Isolated environments, separate billing
```

### 2. Database Organization
```
Account: myapp-cosmos
├── Database: UserData (user profiles, preferences)
├── Database: Orders (orders, payments, invoices)
└── Database: Catalog (products, categories)

Reason: Logical separation, optional shared throughput
```

### 3. Container Design
```
Database: UserData
├── Container: users (partition key: /userId)
├── Container: sessions (partition key: /userId, TTL: 3600)
└── Container: preferences (partition key: /userId)

Reason: Different scaling needs, TTL requirements
```

### 4. Partition Key Selection
```
❌ Bad: /id (every item in different partition)
❌ Bad: /type (few unique values, hot partitions)
✅ Good: /userId (high cardinality, even distribution)
✅ Good: /tenantId (multi-tenant apps)
✅ Good: /category + synthetic (e.g., /category-region)
```

## Cost Implications

### Throughput vs Storage

```
Container A: 10,000 RU/s, 10 GB storage
Cost: ~$58/month (RU/s) + $2.50 (storage) = $60.50/month

Container B: 400 RU/s, 1 TB storage
Cost: ~$23/month (RU/s) + $250 (storage) = $273/month

Insight: Throughput often costs more than storage
```

### Shared vs Dedicated

```
Scenario: 10 small containers, 100 RU/s each

Dedicated: 10 × 400 RU/s (min) = 4,000 RU/s = $230/month
Shared: 1,000 RU/s database = $58/month

Savings: $172/month (75% reduction)
```

## Critical Notes (Критически важные моменты)

- 💡 **Account** — верхний уровень с уникальным DNS-именем, максимум 50 на подписку
- 🎯 **Database** — логическое пространство имён, опционально shared throughput
- ✅ **Container** — единица масштабирования (RU/s и storage)
- ⚠️ **Partition key** — нельзя изменить после создания
- 🔄 **Physical partition** — 50 GB хранения, 10 000 RU/s (управляется Azure)
- 📊 **Logical partition** — максимум 20 GB, определяется значением partition key
- 💡 **Items** — до 2 MB, формат JSON / BSON / Row (зависит от API)
- ✅ **Throughput modes** — Dedicated (container) или Shared (database)
- ⚠️ **Shared throughput** — максимум 25 контейнеров, не включает dedicated контейнеры
- 🔒 **Autoscale** — масштабирование между 10% и 100% от максимального RU/s

---

## Exam Tips (Советы к экзамену AZ-204)

- Иерархия ресурсов:  
  **Account → Database → Container → Items**

- **Account**  
  — Уникальный DNS, максимум 50 на подписку (можно увеличить)

- **Database**  
  — Логическая группировка, поддержка shared throughput

- **Container**  
  — Основная единица масштабирования

- **Partition key**  
  — Обязателен, нельзя изменить, критичен для производительности

- **Physical partition**  
  — 50 GB + 10 000 RU/s (Azure-managed)

- **Logical partition**  
  — До 20 GB на одно значение partition key

- **Максимальный размер item** — 2 MB

- **Throughput modes**
    - Dedicated — уровень контейнера
    - Shared — уровень базы

- **Dedicated throughput**  
  — Гарантированный, дороже, изолирован

- **Shared throughput**  
  — Экономичнее, до 25 контейнеров

- **Autoscale**  
  — Автоматическое масштабирование между 10% и max RU/s

- **Названия по API**
    - NoSQL = Container
    - MongoDB = Collection
    - Cassandra = Table
    - Gremlin = Graph

- **Point read**  
  — ~1 RU для 1 KB (id + partition key)

- **System properties**  
  `_rid`, `_self`, `_etag`, `_ts` (read-only)

- **Partition key best practices**  
  — Высокая кардинальность  
  — Равномерное распределение  
  — Использование в запросах

- ❌ Нельзя изменить:  
  Partition key, Unique keys, Conflict resolution

- ✅ Можно изменить:  
  Throughput, Indexing policy, TTL

---


[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-cosmos-db/3-cosmos-db-resource-hierarchy)
