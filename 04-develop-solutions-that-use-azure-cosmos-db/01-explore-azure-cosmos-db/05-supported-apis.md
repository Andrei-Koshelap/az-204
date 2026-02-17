# Поддерживаемые API в Azure Cosmos DB (Azure Cosmos DB Supported APIs)

## Ключевые понятия (Key Concepts)

- **Мультимодельная база данных (Multi-model database)**  
  Один сервис — несколько API и моделей данных.

- **Совместимость по wire protocol**  
  Можно использовать существующие драйверы, инструменты и SDK.

- **Выбор API при создании аккаунта**  
  API выбирается при создании аккаунта Cosmos DB и **не может быть изменён позже**.

- **Native vs Compatible**  
  API NoSQL — нативный.  
  Остальные API — совместимы по протоколу (wire-compatible).

---

## Что такое API в Cosmos DB?

Это разные интерфейсы доступа к одним и тем же возможностям платформы.

- **Один сервис** — Azure Cosmos DB  
  (глобальная дистрибуция, низкая задержка, SLA)

- **Несколько API** — выбор зависит от задачи и стека разработки.

- **Одинаковые преимущества** — все API получают:
    - SLA по доступности и латентности
    - Глобальную репликацию
    - Автомасштабирование
    - Управление RU/s

---

## Доступные API

| API | Модель данных | Когда использовать |
|------|---------------|-------------------|
| **NoSQL** | Документная (JSON) | Новые современные приложения |
| **MongoDB** | Документная (BSON) | Миграция существующих MongoDB-приложений |
| **PostgreSQL** | Реляционная (Citus) | Горизонтально масштабируемый PostgreSQL |
| **Cassandra** | Column-family | Wide-column, высокая масштабируемость |
| **Gremlin** | Графовая | Сложные связи и графовые запросы |
| **Table** | Key-value | Миграция с Azure Table Storage |

---

## Дополнительные пояснения

### 🔹 NoSQL API (Core API)
- Нативный API Cosmos DB.
- Использует JSON.
- Полная поддержка всех возможностей платформы.
- Лучший выбор для новых cloud-native приложений.

### 🔹 MongoDB API
- Совместим с MongoDB драйверами.
- Подходит для lift-and-shift миграции.
- Не 100% feature parity с последними версиями MongoDB.

### 🔹 PostgreSQL API
- Основан на Citus (распределённый PostgreSQL).
- Поддерживает SQL, ACID, расширения PostgreSQL.
- Подходит для распределённых OLTP-сценариев.

### 🔹 Cassandra API
- Совместим с Cassandra Query Language (CQL).
- Используется для высоконагруженных систем с wide-column моделью.

### 🔹 Gremlin API
- Поддерживает графовые запросы.
- Хорошо подходит для recommendation systems, fraud detection, social graphs.

### 🔹 Table API
- Совместим с Azure Table Storage.
- Простой key-value сценарий.

---

## Экзаменационный акцент (AZ-204)

- API выбирается **один раз при создании аккаунта**.
- Нельзя изменить API позже — только создать новый аккаунт.
- Все API получают преимущества Cosmos DB (SLA, multi-region, auto-scale).
- Если в вопросе говорится о:
    - JSON-документах → NoSQL
    - MongoDB драйверах → MongoDB API
    - SQL + горизонтальное масштабирование → PostgreSQL
    - CQL → Cassandra
    - Граф → Gremlin
    - Key-value + Azure Table → Table API

### API Selection

**Choose at account creation**:

```bash
# API for NoSQL (default)
az cosmosdb create \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --kind GlobalDocumentDB  # NoSQL API

# API for MongoDB
az cosmosdb create \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --kind MongoDB

# API for Cassandra
az cosmosdb create \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --capabilities EnableCassandra

# API for Gremlin
az cosmosdb create \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --capabilities EnableGremlin

# API for Table
az cosmosdb create \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --capabilities EnableTable
```

⚠️ **Нельзя изменить API после создания аккаунта** — выбирайте осознанно!

---

## Матрица сравнения API (API Comparison Matrix)

### Сравнение возможностей

| Возможность | NoSQL | MongoDB | PostgreSQL | Cassandra | Gremlin | Table |
|-------------|--------|----------|------------|------------|----------|--------|
| **Нативный для Cosmos DB** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Совместимость по wire protocol** | N/A | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Полная поддержка возможностей Cosmos DB** | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |
| **Глобальная дистрибуция** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Multi-region writes** | ✅ | ✅ | ❌ | ✅ | ✅ | ✅ |
| **Serverless режим** | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ |
| **Autoscale (автомасштабирование)** | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |

---

## Что важно понимать

### 🔹 NoSQL
- Единственный **нативный API**.
- Поддерживает все возможности Cosmos DB.
- Лучший выбор для новых cloud-native приложений.

### 🔹 MongoDB
- Подходит для миграции существующих MongoDB-приложений.
- Не все функции Cosmos DB доступны напрямую.
- Поддерживает multi-region writes и serverless.

### 🔹 PostgreSQL
- Основан на распределённом PostgreSQL (Citus).
- Нет поддержки multi-region writes.
- Нет serverless режима.
- Хорош для SQL-нагрузок с горизонтальным масштабированием.

### 🔹 Cassandra
- Совместим с CQL.
- Подходит для high-scale систем.
- Нет serverless режима.

### 🔹 Gremlin
- Для графовых сценариев.
- Поддерживает multi-region writes.
- Подходит для сложных связей (social graph, fraud detection).

### 🔹 Table
- Для key-value сценариев.
- Хорош для миграции Azure Table Storage.
- Поддерживает serverless и multi-region writes.

---

## Экзаменационный акцент (AZ-204)

Запомните:

- Только **NoSQL** — нативный API.
- Только **NoSQL** имеет полный доступ к first-class возможностям.
- **PostgreSQL API** не поддерживает multi-region writes.
- **Serverless** отсутствует у PostgreSQL и Cassandra.
- API нельзя изменить после создания аккаунта.

Если вопрос о:
- Максимальной интеграции с Cosmos DB → **NoSQL**
- Использовании существующих MongoDB драйверов → **MongoDB**
- SQL и горизонтальном масштабировании → **PostgreSQL**
- CQL → **Cassandra**
- Графовых данных → **Gremlin**
- Key-value → **Table**


### When to Use Each API

```
Decision Tree:

New application?
├── Modern JSON-based → NoSQL (best choice)
└── Existing app migration ↓

What database are you using?
├── MongoDB → API for MongoDB
├── Cassandra → API for Cassandra
├── PostgreSQL → API for PostgreSQL
├── Azure Table Storage → API for Table
└── Need graph queries → API for Gremlin
```

## API for NoSQL

### Характеристики

**Нативный API Azure Cosmos DB**

- ✅ Лучший end-to-end опыт разработки
- ✅ Первым получает новые возможности платформы
- ✅ Полный контроль над интерфейсом и официальными SDK
- ✅ SQL-подобный синтаксис запросов
- ✅ Хранение данных в формате JSON (документы)

---

## Что это означает на практике

- Использует собственный SQL-подобный язык запросов (не T-SQL).
- Поддерживает вложенные JSON-структуры.
- Глубокая интеграция с:
    - RU-моделью
    - Autoscale
    - Multi-region writes
    - Change Feed
- Максимальная совместимость с новыми возможностями Cosmos DB.

---

## Когда выбирать NoSQL API
✅ Новые cloud-native приложения
✅ Микросервисная архитектура
✅ REST/JSON-first backend
✅ Нужна максимальная интеграция с Cosmos DB
❌ Требуется 100% совместимость с MongoDB/PostgreSQL

---

## Экзаменационный акцент (AZ-204)

Если в вопросе говорится:
- «максимальные возможности Cosmos DB»
- «первичная поддержка новых функций»
- «JSON документы»
- «SQL-like queries»

→ правильный выбор: **API for NoSQL**.

Это рекомендуемый вариант для большинства новых проектов.


### Document Format

```json
{
  "id": "product-1",
  "category": "electronics",
  "name": "Laptop",
  "price": 999.99,
  "specs": {
    "cpu": "Intel i7",
    "ram": "16GB",
    "storage": "512GB SSD"
  },
  "tags": ["laptop", "computer", "portable"]
}
```

### Query Language

**SQL-like syntax**:

```sql
-- Select all
SELECT * FROM products p

-- Filter
SELECT * FROM products p 
WHERE p.category = 'electronics' AND p.price < 1000

-- Project specific fields
SELECT p.name, p.price FROM products p

-- Join arrays
SELECT p.name, tag
FROM products p
JOIN tag IN p.tags

-- Aggregations
SELECT p.category, COUNT(1) AS count, AVG(p.price) AS avgPrice
FROM products p
GROUP BY p.category
```

### SDK Support

```csharp
// .NET SDK
using Microsoft.Azure.Cosmos;

var client = new CosmosClient(endpoint, key);
var container = client.GetContainer("mydb", "products");

// Create
await container.CreateItemAsync(product);

// Read
var response = await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics")
);

// Query
var query = container.GetItemQueryIterator<Product>(
    "SELECT * FROM c WHERE c.category = 'electronics'"
);
```

### Когда использовать API for NoSQL

✅ **Новые приложения** — современные cloud-native решения  
✅ **JSON-данные** — естественная документная модель  
✅ **Гибкая схема** — schema-less дизайн  
✅ **Богатые запросы** — SQL-подобные возможности  
✅ **Новейшие функции** — первыми получают обновления

❌ **Не использовать**, если уже есть приложение на MongoDB или Cassandra  
→ В этом случае выбирайте совместимый API.

---

# API for MongoDB

## Характеристики

**Совместимость с MongoDB wire protocol**

- ✅ Использование существующих MongoDB-инструментов и драйверов
- ✅ Не требуется менять код подключения
- ✅ Формат данных — BSON (Binary JSON)
- ✅ Поддержка MongoDB Query Language
- ⚠️ Совместимость, но не полная идентичность (есть отличия в возможностях)

---

## Что это означает на практике

- Подходит для lift-and-shift миграции в облако.
- Поддерживает большинство популярных MongoDB-функций.
- Масштабируется глобально через возможности Cosmos DB.
- Получает SLA, multi-region distribution и autoscale.

Важно:  
Cosmos DB API for MongoDB — это не «настоящий MongoDB», а совместимая реализация.

---

## Поддержка версий

| Версия | Возможности |
|---------|-------------|
| **6.0** | Новейшие функции, лучшая производительность |
| **5.0** | Time-series коллекции, нативная агрегация |
| **4.2** | Распределённые транзакции |
| **4.0** | ACID-транзакции для нескольких документов |
| **3.6** | Change streams, улучшенная агрегация |

---

## Когда выбирать MongoDB API
✅ Уже есть приложение на MongoDB
✅ Нужно быстро мигрировать без переписывания кода
✅ Команда владеет MongoDB-экосистемой
❌ Нужен полный контроль над нативными возможностями Cosmos DB

---

## Экзаменационный акцент (AZ-204)

Если в вопросе говорится:
- «использовать существующие MongoDB драйверы»
- «миграция MongoDB в Azure»
- «минимальные изменения кода»

→ правильный выбор: **API for MongoDB**.


### Connection String

```javascript
// MongoDB connection (minimal code change)
const { MongoClient } = require('mongodb');

const connectionString = "mongodb://mycosmosaccount:key@mycosmosaccount.mongo.cosmos.azure.com:10255/?ssl=true&replicaSet=globaldb";

const client = new MongoClient(connectionString);
await client.connect();

const db = client.db('mydb');
const collection = db.collection('products');

// Standard MongoDB operations
await collection.insertOne({ name: "Laptop", price: 999.99 });
const product = await collection.findOne({ name: "Laptop" });
```

### Compatibility


### Совместимость (Compatibility)

#### Поддерживаемые операции

- ✅ CRUD-операции (insert, find, update, delete)
- ✅ Aggregation pipeline
- ✅ Индексы (single-field, compound, geospatial)
- ✅ Change streams
- ✅ Транзакции (начиная с версии 4.0+)

---

### Отличия от нативного MongoDB

- ⚠️ Иные характеристики производительности
- ⚠️ Биллинг основан на RU (Request Units), а не на CPU/памяти
- ⚠️ Некоторые возможности могут иметь ограничения

Важно понимать:  
Хотя API совместим по протоколу, внутренняя архитектура — это Cosmos DB, а не оригинальный MongoDB.

---

## Когда использовать MongoDB API

✅ Уже существует приложение на MongoDB (миграция без переписывания)
✅ Команда обладает экспертизой MongoDB
✅ Используются MongoDB-инструменты и драйверы
✅ Требуется работа с BSON (Binary JSON)
❌ Новое приложение без опыта MongoDB → лучше выбрать NoSQL API

---

## Практический совет

Если задача — **миграция** → MongoDB API.  
Если задача — **новая cloud-native разработка** → NoSQL API.

---

## Экзаменационный акцент (AZ-204)

Если в вопросе говорится:
- «миграция MongoDB без изменений кода»
- «использовать существующие MongoDB драйверы»
- «минимальный рефакторинг»

→ выбирайте **API for MongoDB**.

Если же упор на:
- максимальные возможности Cosmos DB,
- нативную интеграцию,
- новые проекты,

→ правильный ответ — **API for NoSQL**.


### Migration Benefit

```
Before (Hosted MongoDB):
├── Manual scaling
├── Manual replication
├── Manual failover
├── Regional only
└── Manual backups

After (Cosmos DB with MongoDB API):
├── Automatic scaling ✅
├── Turnkey global distribution ✅
├── Automatic failover ✅
├── Multi-region ✅
└── Automatic backups ✅

Code changes: Minimal (just connection string)
```

# API for PostgreSQL

## Характеристики

**Распределённый PostgreSQL на базе Citus**

- ✅ Совместимость с PostgreSQL wire protocol
- ✅ Расширение Citus для горизонтального масштабирования
- ✅ Реляционная модель данных с scale-out архитектурой
- ✅ Поддержка стандартных PostgreSQL-инструментов и драйверов
- ✅ Возможность single-node или multi-node конфигурации

---

## Что это означает

- Полная поддержка SQL.
- ACID-транзакции.
- Горизонтальное шардирование через Citus.
- Подходит для распределённых OLTP-нагрузок.
- Используется привычная экосистема PostgreSQL (psql, pgAdmin, ORM).

---

## Архитектура

- **Coordinator node** — принимает SQL-запросы.
- **Worker nodes** — хранят шарды данных.
- Citus распределяет таблицы по шардам.
- Поддерживается parallel query execution.

---

## Когда использовать
✅ Нужна реляционная модель (таблицы, JOIN, foreign keys)
✅ Требуется горизонтальное масштабирование PostgreSQL
✅ Существующее PostgreSQL-приложение
✅ SQL-first архитектура
❌ Нужны multi-region writes (не поддерживается)
❌ Требуется serverless режим
---

## Ограничения

- Нет поддержки multi-region writes.
- Нет serverless режима.
- Это отдельный вариант Cosmos DB (не JSON-документная модель).
- Требует продуманного выбора shard key для масштабирования.

---

## Экзаменационный акцент (AZ-204)

Если в вопросе говорится:
- «распределённый PostgreSQL»
- «Citus»
- «горизонтальное масштабирование SQL»
- «реляционная модель»

→ правильный выбор: **API for PostgreSQL**.

Если требуется:
- multi-region writes,
- document model (JSON),
- максимальная интеграция с Cosmos DB,

→ это не PostgreSQL API.


### Architecture Options

#### Single Node
```
Use case: Small databases (<100 GB)
Benefits: Simple, PostgreSQL-compatible
Limitations: No distribution
```

#### Multi-Node (Citus)
```
Use case: Large databases (>100 GB)
Architecture:
  Coordinator Node
       ↓
  Worker Node 1 | Worker Node 2 | Worker Node n
       ↓
  Distributed tables across workers
  Automatic query routing
  Parallel query execution
```

### Connection

```python
import psycopg2

# Standard PostgreSQL connection
conn = psycopg2.connect(
    host="c-mycosmospostgres.12345.postgres.cosmos.azure.com",
    database="mydb",
    user="citus",
    password="password",
    port=5432
)

cursor = conn.cursor()
cursor.execute("SELECT * FROM products WHERE category = 'electronics'")
rows = cursor.fetchall()
```

### When to Use

✅ **Relational workloads** - Need SQL, joins, constraints
✅ **PostgreSQL apps** - Existing PostgreSQL applications
✅ **Large datasets** - Multi-terabyte databases
✅ **Multi-tenant** - Distribute by tenant_id

❌ **Don't use if** - Pure document/NoSQL model (use NoSQL API)

# API for Apache Cassandra

## Характеристики

**Совместимость с Cassandra wire protocol**

- ✅ Поддержка CQL (Cassandra Query Language)
- ✅ Column-family модель данных
- ✅ Wide-column хранилище
- ✅ Высокая пропускная способность записи (high write throughput)
- ✅ Использование существующих Cassandra-драйверов

---

## Что это означает на практике

- Подходит для систем с большим объёмом записей (write-heavy workloads).
- Хорошо масштабируется горизонтально.
- Модель данных ориентирована на ключ + набор колонок.
- Отличный выбор для time-series и large-scale ingestion сценариев.

---

## Когда использовать


### Data Model

```cql
-- Keyspace (like database)
CREATE KEYSPACE store WITH replication = {
  'class': 'NetworkTopologyStrategy',
  'datacenter1': 3
};

-- Table (column-family)
CREATE TABLE products (
  category text,
  product_id uuid,
  name text,
  price decimal,
  PRIMARY KEY (category, product_id)
);

-- Insert
INSERT INTO products (category, product_id, name, price)
VALUES ('electronics', uuid(), 'Laptop', 999.99);

-- Query
SELECT * FROM products WHERE category = 'electronics';
```

### Key Features

**Wide-column model**:
```
Row key: category=electronics
├── product_id=uuid1 → name=Laptop, price=999.99
├── product_id=uuid2 → name=Phone, price=699.99
└── product_id=uuid3 → name=Tablet, price=499.99

Efficient for time-series, IoT, high write throughput
```

## Когда использовать API for Apache Cassandra

✅ Высокая пропускная способность записи — IoT, логирование, time-series
✅ Уже есть приложение на Cassandra — миграция без переписывания
✅ Wide-column модель — гибкая схема колонок
✅ Есть экспертиза Cassandra в команде
❌ Лучше документная модель — используйте NoSQL или MongoDB API


---

# API for Apache Gremlin

## Характеристики

**Графовый API**

- ✅ Поддержка Apache TinkerPop Gremlin
- ✅ Модель «вершины и рёбра» (vertices & edges)
- ✅ Сложные запросы по связям
- ✅ Графовые обходы (graph traversals)
- ✅ Property graph model

---

## Что это означает

- Данные представлены как узлы (vertices) и связи (edges).
- Каждая вершина и ребро могут содержать свойства (key-value).
- Подходит для deeply connected data.
- Запросы выполняются через Gremlin traversal language.

---

## Когда использовать

✅ Социальные сети (друзья, подписки)
✅ Fraud detection (поиск подозрительных связей)
✅ Recommendation systems
✅ Knowledge graphs
❌ Простая key-value модель
❌ Обычная документная структура без сложных связей


---

## Экзаменационный акцент (AZ-204)

Если в вопросе говорится:
- «graph»
- «relationships»
- «recommendation engine»
- «fraud detection»
- «Gremlin traversal»

→ правильный выбор: **API for Apache Gremlin**.

### Graph Model

```
Vertices (Nodes):
├── Person: {id: "john", name: "John", age: 30}
├── Person: {id: "jane", name: "Jane", age: 28}
└── Product: {id: "laptop", name: "Laptop", price: 999.99}

Edges (Relationships):
├── john -[knows]-> jane
├── john -[purchased]-> laptop
└── jane -[likes]-> laptop
```

### Gremlin Queries

```groovy
// Add vertex
g.addV('person')
 .property('id', 'john')
 .property('name', 'John')
 .property('age', 30)

// Add edge
g.V('john').addE('knows').to(g.V('jane'))

// Traversal queries
// Find John's friends
g.V('john').out('knows').values('name')

// Find products liked by John's friends
g.V('john').out('knows').out('likes').hasLabel('product')

// Find common interests
g.V('john').out('knows').where(out('likes').hasId('laptop'))
```

## Когда использовать API for Apache Gremlin

✅ Социальные сети — друзья, связи, рекомендации
✅ Fraud detection — паттерны транзакций, связанные сущности
✅ Recommendation engines — предложения на основе графа
✅ Knowledge graphs — связи между сущностями
✅ Сетевая топология — инфраструктура, маршрутизация
❌ Простая key-value или документная модель → лучше NoSQL API


---

# API for Table

## Характеристики

**Совместимость с Azure Table Storage**

- ✅ Key-value хранилище
- ✅ Модель PartitionKey + RowKey
- ✅ Drop-in замена Azure Table Storage
- ✅ Более высокая производительность и глобальная дистрибуция
- ✅ Обратная совместимость

---

## Что это означает

- Простая и предсказуемая модель данных.
- Горизонтальное масштабирование через PartitionKey.
- Поддерживает существующие Table Storage SDK.
- Получает преимущества Cosmos DB:
    - Глобальную репликацию
    - SLA
    - Autoscale
    - Multi-region writes

---

## Когда использовать

✅ Миграция с Azure Table Storage
✅ Простые key-value сценарии
✅ Метаданные
✅ Конфигурационные данные
❌ Нужны сложные запросы
❌ Требуется граф или документная модель


---

## Экзаменационный акцент (AZ-204)

Если в вопросе говорится:
- «PartitionKey / RowKey»
- «миграция Azure Table Storage»
- «простая key-value модель»

→ правильный выбор: **API for Table**.

### Data Model

```csharp
public class ProductEntity : ITableEntity
{
    public string PartitionKey { get; set; }  // category
    public string RowKey { get; set; }        // product-id
    public string Name { get; set; }
    public double Price { get; set; }
    public DateTimeOffset? Timestamp { get; set; }
    public ETag ETag { get; set; }
}

// Insert
var entity = new ProductEntity
{
    PartitionKey = "electronics",
    RowKey = "product-1",
    Name = "Laptop",
    Price = 999.99
};
await tableClient.AddEntityAsync(entity);

// Query
var entities = tableClient.Query<ProductEntity>(
    e => e.PartitionKey == "electronics"
);
```

### Migration from Azure Table Storage

```
Before (Azure Table Storage):
├── Single region
├── Limited throughput
├── No global distribution
├── Basic SLAs

After (Cosmos DB Table API):
├── Multi-region ✅
├── Unlimited throughput ✅
├── Global distribution ✅
├── 99.999% SLA ✅

Migration: Change connection string only!
```

## Когда использовать API for Table

✅ Миграция Azure Table Storage — обновление с минимальными изменениями
✅ Простая key-value модель — доступ по PartitionKey + RowKey
✅ OLTP-нагрузки — транзакционная обработка
✅ Уже есть приложение на Table Storage — drop-in замена
❌ Нужны сложные запросы → лучше NoSQL API
---

# Выбор правильного API (Choosing the Right API)

## Матрица принятия решения

| Текущее состояние | Рекомендуемый API | Причина |
|-------------------|------------------|---------|
| **Новое приложение, JSON-данные** | NoSQL | Нативный API, максимум возможностей |
| **Существующий MongoDB** | MongoDB | Минимальная миграция, совместимость по протоколу |
| **Существующий Cassandra** | Cassandra | Совместимость с CQL, wide-column модель |
| **Существующий PostgreSQL** | PostgreSQL | Реляционная модель + масштабирование через Citus |
| **Графовые связи** | Gremlin | Оптимизирован для graph traversal |
| **Azure Table Storage** | Table | Drop-in замена, улучшенные SLA |

---

## Факторы выбора

### Технические

- ✅ Существующий код (wire protocol совместимость)
- ✅ Экспертиза команды (использование знакомых инструментов)
- ✅ Модель данных (document, graph, wide-column, relational, key-value)
- ✅ Паттерны запросов (простые чтения vs сложные JOIN/traversal)

### Бизнес-факторы

- ✅ Стоимость миграции (переписывание vs минимальные изменения)
- ✅ Time-to-market (быстрее с совместимым API)
- ✅ Требования к функциям (проверяйте ограничения конкретного API)

---

# Ограничения API

## Общие для всех API

✅ Доступно во всех вариантах:
- Глобальная дистрибуция
- Автомасштабирование
- Низкая задержка (<10 мс)
- Высокая доступность (99.999%)
- Шифрование данных (at rest и in transit)

---

## Эксклюзивно для NoSQL API

- Serverless (доступен во всех API, кроме PostgreSQL и Cassandra)
- Новые функции появляются сначала здесь
- Самая богатая поддержка SDK

---

## Ограничения конкретных API

| API | Существенные ограничения |
|-----|--------------------------|
| **MongoDB** | Некоторые операции отличаются от нативного MongoDB |
| **PostgreSQL** | Нет serverless, для масштабирования нужен multi-node |
| **Cassandra** | Нет serverless |
| **Gremlin** | Ограничен поддерживаемыми Gremlin-шагами |
| **Table** | Ограниченные возможности запросов по сравнению с NoSQL |

---

# Важные замечания

- 💡 **NoSQL API** — нативный, максимум возможностей, первым получает обновления
- 🎯 **Wire protocol совместимость** — MongoDB, Cassandra, Gremlin, Table
- ✅ **Выбор при создании аккаунта** — изменить API позже нельзя
- ⚠️ **Миграция** — совместимые API минимизируют изменения кода
- 🔄 **Все API** — одинаковые преимущества (глобальность, SLA, масштабирование)
- 📊 **Модели данных** — document, column-family, graph, key-value, relational
- 💡 **NoSQL для новых проектов** — лучший выбор для greenfield
- 🔒 **Serverless поддержка** — NoSQL, MongoDB, Gremlin, Table (не PostgreSQL, не Cassandra)

---

# Подсказки для экзамена (AZ-204)

- Доступные API: NoSQL, MongoDB, PostgreSQL, Cassandra, Gremlin, Table
- NoSQL → нативный API, JSON-документы, SQL-like запросы
- MongoDB → wire protocol, BSON, существующие MongoDB-приложения
- PostgreSQL → реляционная модель, Citus, масштабирование через multi-node
- Cassandra → CQL, column-family, высокая скорость записи
- Gremlin → графовая модель, вершины и рёбра
- Table → PartitionKey + RowKey, совместимость с Azure Table Storage
- API выбрать можно только при создании аккаунта
- NoSQL → лучший выбор для новых современных приложений
- Совместимые API → используйте при миграции
- Все API → глобальная дистрибуция, autoscale, высокая доступность
- Serverless → доступен не во всех API
- Новые функции → сначала появляются в NoSQL API

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-cosmos-db/6-cosmos-db-supported-apis)
