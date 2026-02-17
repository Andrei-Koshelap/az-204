# Azure Cosmos DB Request Units (RU)

## Ключевые понятия (Key Concepts)

- **Request Units (RU)** — нормализованная стоимость операций в базе данных
- **Provisioning throughput** — резервирование пропускной способности в RU/сек
- **Режимы выделения** — Manual, Autoscale, Serverless
- **Модель стоимости** — оплата за throughput (RU/сек) + хранение (GB)

---

# Что такое Request Units?

## Единая метрика стоимости

**Request Unit (RU)** — это абстракция вычислительных ресурсов, объединяющая:

- CPU
- IOPS (дисковые операции)
- Память
- Сетевые ресурсы

RU позволяет измерять стоимость любой операции в одной унифицированной единице.

---

## Пример стоимости операций

| Операция | Примерная стоимость |
|-----------|---------------------|
| Чтение 1KB документа | ~1 RU |
| Запись 1KB документа | ~5 RU |
| Сложный запрос | Зависит от фильтрации и индексов |
| Кросс-партиционный запрос | Выше, чем внутри одной партиции |

> Запись дороже чтения.  
> Чем больше документ — тем больше RU.

---

## Что влияет на потребление RU

- Размер документа
- Количество свойств
- Использование индексов
- Фильтрация и сортировка
- Кросс-партиционные запросы
- Тип операции (read vs write)

---

# Provisioning Modes (Режимы выделения)

## 1️⃣ Manual Throughput
- Фиксированное RU/сек
- Подходит для стабильной нагрузки
- Оплата за выделенную мощность

## 2️⃣ Autoscale
- Автоматическое масштабирование RU
- Диапазон: 10%–100% от max RU
- Хорошо подходит для переменной нагрузки

## 3️⃣ Serverless
- Нет резервирования RU
- Оплата только за фактическое потребление
- Подходит для непредсказуемых или редких нагрузок

---

# Важные замечания

- RU — это пропускная способность в секунду (RU/с).
- Если превышаете лимит RU → получите 429 (throttling).
- Можно увеличить RU без downtime.
- RU настраивается на уровне контейнера или базы данных.

---

# Экзаменационный акцент (AZ-204)

- 1KB read ≈ 1 RU
- Writes дороже reads
- 429 status code → превышение RU
- Autoscale подходит для переменной нагрузки
- Serverless — для нерегулярной нагрузки
- Manual — для предсказуемой нагрузки
- RU зависит от размера документа и сложности запроса


```
1 RU = Resources to read 1 KB item by ID + partition key

System Resources:
├── CPU
├── IOPS (disk I/O)
├── Memory
└── Network bandwidth

All normalized into single metric: RU
```

### Why RUs Matter

**Predictable performance**:

```
Traditional Databases:
├── CPU % → unpredictable
├── Memory GB → complex
├── IOPS → variable
└── Cost → hard to estimate

Azure Cosmos DB:
├── RU → predictable
├── 1 RU = 1 KB point read
├── All operations measured in RUs
└── Cost → simple calculation
```

## RU Consumption Examples

### Point Read

**Most efficient operation**:

```csharp
// Read 1 KB item = 1 RU
await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics")
);
// Cost: 1 RU

// Read 5 KB item = 5 RU
await container.ReadItemAsync<LargeProduct>(
    "product-2",
    new PartitionKey("electronics")
);
// Cost: 5 RU (scales linearly with size)
```

**Point read formula**:
```
RU cost = Item size (KB) × 1 RU
```

### Write Operations

**More expensive than reads**:

```csharp
// Create 1 KB item ≈ 5 RU
await container.CreateItemAsync(product);
// Cost: ~5 RU

// Update 1 KB item ≈ 5 RU
await container.ReplaceItemAsync(product, product.Id);
// Cost: ~5 RU

// Delete 1 KB item ≈ 5 RU
await container.DeleteItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics")
);
// Cost: ~5 RU
```

**Write formula** (approximate):
```
RU cost = Item size (KB) × 5 RU
```

### Query Operations

**Most expensive, variable cost**:

```csharp
// Simple query with partition key
var query = container.GetItemQueryIterator<Product>(
    "SELECT * FROM c WHERE c.category = 'electronics' AND c.price < 1000"
);
// Cost: 2-10 RU (efficient with partition key)

// Cross-partition query
var query = container.GetItemQueryIterator<Product>(
    "SELECT * FROM c WHERE c.price < 1000"
);
// Cost: 10-1000+ RU (scans multiple partitions)

// Complex query with aggregations
var query = container.GetItemQueryIterator<Product>(
    "SELECT c.category, COUNT(1), AVG(c.price) FROM c GROUP BY c.category"
);
// Cost: 100-10000+ RU (depends on data volume)
```

## Факторы запроса, влияющие на стоимость RU

- ✅ Фильтрация по Partition Key — наиболее эффективно
- ⚠️ Кросс-партиционные запросы — значительно дороже
- ⚠️ Агрегации (COUNT, AVG и т.д.)
- ⚠️ ORDER BY — дополнительные затраты на сортировку
- ⚠️ Размер результирующего набора

---

# Сравнение стоимости операций

## Таблица затрат

| Операция | Размер элемента | RU | Примечание |
|-----------|----------------|-----|-------------|
| **Point read** | 1 KB | 1 RU | Самый эффективный вариант |
| **Point read** | 10 KB | 10 RU | Линейный рост |
| **Create** | 1 KB | ~5 RU | Стоимость индексации |
| **Update** | 1 KB | ~5 RU | Переиндексация |
| **Delete** | 1 KB | ~5 RU | Очистка индексов |
| **Query (в пределах партиции)** | - | 2–10 RU | При наличии partition key |
| **Query (кросс-партиционный)** | - | 10–1000+ RU | Сканирование нескольких партиций |

---

# Факторы, влияющие на потребление RU

### 1️⃣ Размер элемента
Чем больше документ — тем больше RU.  
Рост почти линейный.

### 2️⃣ Тип операции
Запись (create/update/delete) дороже чтения.

### 3️⃣ Индексация
Чем больше индексов — тем выше стоимость записи.

### 4️⃣ Уровень согласованности
Strong ≈ ~2× дороже Session/Eventual.

### 5️⃣ Сложность запроса
Агрегации, JOIN (внутри документа), сортировка увеличивают RU.

### 6️⃣ Partition key
- In-partition query → дешевле
- Cross-partition query → дороже

---

# Практические рекомендации

- Используйте **point reads** вместо query, если знаете id + partition key.
- Всегда проектируйте хороший partition key.
- Избегайте кросс-партиционных запросов.
- Минимизируйте размер документов.
- Настраивайте индексацию (убирайте лишние индексы).

---

# Экзаменационный акцент (AZ-204)

Если в вопросе:
- «минимальная стоимость RU» → point read + partition key
- «429 error» → превышение RU
- «как снизить стоимость?» → оптимизировать partition key и запрос
- «Strong consistency» → выше RU

Главное правило:  
**Partition key + Point Read = самая дешёвая операция.**

### Optimize RU Usage

```csharp
// ❌ Expensive: Cross-partition query
var query = "SELECT * FROM c WHERE c.price < 100";
// Scans all partitions

// ✅ Efficient: In-partition query
var query = "SELECT * FROM c WHERE c.category = 'electronics' AND c.price < 100";
// Only scans electronics partition

// ❌ Expensive: Read entire item to check one field
var item = await container.ReadItemAsync<Product>(id, pk);
if (item.Resource.IsActive) { /* ... */ }
// Cost: Full item read

// ✅ Efficient: Query only needed fields
var query = "SELECT c.id, c.isActive FROM c WHERE c.id = @id";
// Cost: Minimal (only selected fields)
```

## Throughput Provisioning Modes

### 1. Manual (Provisioned Throughput)

**Fixed RU/s capacity**:

```bash
# Provision 1,000 RU/s
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/category" \
  --throughput 1000

# Result:
# - 1,000 RU/s capacity
# - Steady, predictable cost
# - Can exceed briefly (burst), then throttled
```

## Manual Throughput (Provisioned RU/s)

### Характеристики

- ✅ Предсказуемая стоимость (~$0.008 в час за 100 RU/s)
- ✅ Самый выгодный вариант при стабильной нагрузке
- ⚠️ Масштабирование выполняется вручную
- ⚠️ При превышении лимита — throttling (ошибка 429)

---

## Пример расчёта стоимости

```
1,000 RU/s × 730 hours/month × $0.008/100 RU/s
= $58.40/month
```

### Как считать правильно

1. Делим RU/s на 100  
   1000 / 100 = 10
2. Умножаем на стоимость в час  
   10 × $0.008 = $0.08 в час
3. Умножаем на часы в месяце (~730)  
   0.08 × 730 = $58.40
---
## Когда выбирать Manual


### 2. Autoscale (Provisioned Throughput)

**Automatic scaling**:

```bash
# Configure autoscale (max 4,000 RU/s)
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/category" \
  --max-throughput 4000

# Result:
# - Scales between 400 (10%) and 4,000 RU/s
# - Automatic based on usage
# - Pay for highest RU/s used per hour
```

**Scaling behavior**:
```
Idle: 400 RU/s (10% of max)
Low traffic: 800 RU/s (20% of max)
Medium: 2,000 RU/s (50% of max)
Peak: 4,000 RU/s (100% of max)

Scale up: Instant
Scale down: Gradual (no abrupt drops)
```

**Characteristics**:
- ✅ Automatic scaling
- ✅ No throttling (scales up instantly)
- ✅ Pay for usage (per hour)
- ⚠️ 1.5× cost of manual (convenience premium)

**Pricing example**:
```
Max: 4,000 RU/s
Actual usage:
- 20 hours at 4,000 RU/s = 20 hours × $3.20/hour = $64
- 10 hours at 2,000 RU/s = 10 hours × $1.60/hour = $16
Total: $80

vs Manual 4,000 RU/s always:
730 hours × $3.20/hour = $2,336/month
```

### 3. Serverless

**Pay-per-request** (no provisioning):

```bash
# Create serverless account
az cosmosdb create \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --capabilities EnableServerless

# Create container (no throughput specified)
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/category"

# No throughput parameter needed
```

**Characteristics**:
- ✅ No provisioning required
- ✅ Pay only for RUs consumed
- ✅ No throttling (elastic capacity)
- ⚠️ Higher per-RU cost ($0.25 per million RUs)
- ⚠️ Max 5,000 RU/s per container
- ⚠️ Max 50 GB per container
- ❌ No SLA on latency/throughput

**Pricing example**:
```
Monthly consumption: 100 million RUs
Cost: 100 × $0.25 = $25/month

vs Manual 400 RU/s (lowest):
$23.36/month (close for low usage)

vs Manual 10,000 RU/s:
$584/month (serverless much cheaper for sporadic use)
```

## Сравнение режимов выделения throughput (Mode Comparison)

| Возможность | Manual | Autoscale | Serverless |
|-------------|--------|------------|-------------|
| **Provisioning** | Фиксированные RU/s | Максимальный RU/s (динамически) | Нет резервирования |
| **Масштабирование** | Вручную | Автоматически | Автоматически |
| **Минимальный RU/s** | 400 | 400 (10% от max) | 0 |
| **Максимальный RU/s** | Без ограничений | Без ограничений | 5 000 на контейнер |
| **Стоимость** | Самая низкая (при стабильной нагрузке) | ~1.5× Manual | Оплата по факту |
| **Throttling (429)** | Да | Нет (в пределах max RU) | Нет |
| **Лучше всего подходит для** | Стабильного трафика | Переменной нагрузки | Редкого трафика / Dev |
| **SLA** | Да | Да | Нет (best-effort) |
| **Макс. объём хранения** | Без ограничений | Без ограничений | 50 GB на контейнер |

---

## Как выбирать режим

### 🔹 Manual
- Минимальная стоимость при постоянной нагрузке.
- Требует ручного масштабирования.
- Подходит для production с предсказуемым трафиком.

### 🔹 Autoscale
- RU автоматически масштабируются от 10% до max.
- Лучше для приложений с пиками нагрузки.
- Дороже примерно на 50% по сравнению с Manual.

### 🔹 Serverless
- Нет фиксированного RU.
- Оплата только за фактические операции.
- Нет SLA.
- Ограничения по объёму и RU.
- Идеально для dev/test или нерегулярных нагрузок.

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:
- «переменная нагрузка» → **Autoscale**
- «редкие запросы» или «dev/test» → **Serverless**
- «стабильный постоянный трафик» → **Manual**
- «минимальная стоимость при steady workload» → **Manual**
- «нужен SLA» → не Serverless


## Provisioning Levels

### Container-Level Throughput

**Dedicated RU/s per container**:

```bash
# Container with dedicated 1,000 RU/s
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --partition-key-path "/category" \
  --throughput 1000

# Guaranteed capacity:
# - Exclusively for this container
# - Not shared with other containers
# - More expensive but predictable
```

### Database-Level Throughput (Shared)

**Shared RU/s across containers**:

```bash
# Database with 1,000 RU/s shared
az cosmosdb sql database create \
  --account-name mycosmosaccount \
  --name mydb \
  --throughput 1000

# Containers share capacity:
az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name container1 \
  --partition-key-path "/category"
  # No --throughput (shares database RU/s)

az cosmosdb sql container create \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name container2 \
  --partition-key-path "/id"
  # Also shares same 1,000 RU/s
```

## Shared Throughput (Общий throughput на уровне базы данных)

### Характеристики

- ✅ Экономически выгодно для множества небольших контейнеров
- ✅ До 25 контейнеров могут совместно использовать RU
- ⚠️ Минимум 400 RU/s на базу данных
- ⚠️ Нет гарантированного RU для каждого контейнера (динамическое распределение)
- ⚠️ Нельзя смешивать режимы — контейнеры либо все shared, либо с выделенным throughput

---

## Как это работает

- RU выделяются **на уровне базы данных**, а не контейнера.
- Контейнеры «конкурируют» за общий пул RU.
- Если один контейнер потребляет больше, другим может не хватить RU.
- При превышении лимита — возможен 429 (throttling).

---

## Когда использовать

✅ Много небольших контейнеров
✅ Низкая или непостоянная нагрузка
✅ Экономия затрат важнее изоляции
❌ Нужны гарантированные RU для каждого контейнера
❌ Один контейнер сильно нагружен


---

## Экзаменационный акцент (AZ-204)

- Shared throughput настраивается на уровне **database**.
- Минимум 400 RU/s на базу.
- До 25 контейнеров.
- Нет изоляции производительности между контейнерами.
- Нельзя смешивать shared и dedicated в одной базе данных.

### Cost Comparison

```
Scenario: 5 containers, each needs ~200 RU/s

Option 1: Dedicated throughput
5 × 400 RU/s (min per container) = 2,000 RU/s
Cost: $116.80/month

Option 2: Shared throughput
1,000 RU/s database (shared)
Cost: $58.40/month

Savings: $58.40/month (50%)
```

## Scaling Throughput

### Manual Scaling

```bash
# Scale up
az cosmosdb sql container throughput update \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --throughput 10000
# Instant scale-up

# Scale down
az cosmosdb sql container throughput update \
  --account-name mycosmosaccount \
  --database-name mydb \
  --name mycontainer \
  --throughput 1000
# May take time if data stored > scale-down allows
```

### Programmatic Scaling

```csharp
// Get current throughput
ThroughputResponse throughput = await container.ReadThroughputAsync();
int? currentRUs = throughput.Resource?.Throughput;

// Scale up
await container.ReplaceThroughputAsync(10000);

// Scale down
await container.ReplaceThroughputAsync(1000);

// Autoscale
await container.ReplaceThroughputAsync(
    ThroughputProperties.CreateAutoscaleThroughput(4000)  // max
);
```

### Scaling Limitations

**Scale-down limitations**:

```
Current: 10,000 RU/s with 50 GB data

Can scale down to:
Minimum = Max(400 RU/s, Storage GB / 100 × 10 RU/s)
        = Max(400, 50 / 100 × 10)
        = Max(400, 5)
        = 400 RU/s ✓

With 100 GB data:
Minimum = Max(400, 100 / 100 × 10)
        = Max(400, 10)
        = 400 RU/s ✓

With 10 TB (10,000 GB) data:
Minimum = Max(400, 10000 / 100 × 10)
        = Max(400, 1000)
        = 1,000 RU/s (can't go below)
```

**Scale-down formula**:
```
Minimum RU/s = Max(400, Storage (GB) / 100 × 10)
```

## Monitoring RU Consumption

### View RU Charge

```csharp
// Check RU cost of operation
var response = await container.CreateItemAsync(product);
double ruCharge = response.RequestCharge;
Console.WriteLine($"Operation consumed {ruCharge} RUs");

// Query RU cost
var query = container.GetItemQueryIterator<Product>(
    "SELECT * FROM c WHERE c.category = 'electronics'"
);

double totalRUs = 0;
while (query.HasMoreResults)
{
    var page = await query.ReadNextAsync();
    totalRUs += page.RequestCharge;
}
Console.WriteLine($"Query consumed {totalRUs} RUs");
```

### Response Headers

```
HTTP/1.1 200 OK
x-ms-request-charge: 5.43
x-ms-retry-after-ms: 0

If throttled (429):
HTTP/1.1 429 Request Rate Too Large
x-ms-request-charge: 10.0
x-ms-retry-after-ms: 50
```

## Мониторинг RU в Azure Portal

### Где смотреть метрики

1. Откройте **Cosmos DB account**
2. Перейдите в раздел **Metrics**
3. Выберите нужные показатели

---

### Основные метрики

- **Request Units**
    - Показывает потребление RU/s во времени
    - Помогает определить пики нагрузки

- **Throttled Requests**
    - Отображает количество 429 ошибок
    - Индикатор нехватки выделенного throughput

- **Normalized RU Consumption**
    - Процент использования выделенных RU
    - Если близко к 100% → риск throttling

---

## Как интерпретировать

- 📈 Частые пики до 100% → стоит увеличить RU или включить Autoscale
- 🔁 Постоянное использование <30% → возможно переплата
- 🚨 429 ошибки → превышение лимита RU/s
- 📊 Резкие скачки → переменная нагрузка (рассмотрите Autoscale)

---

## Практический совет

- Для production регулярно отслеживайте:
    - Normalized RU Consumption
    - Throttled Requests
- Настройте Azure Monitor Alerts на 429 ошибки.
- Оптимизируйте запросы перед увеличением RU.

---

## Экзаменационный акцент (AZ-204)

Если в вопросе:
- «429 errors» → нехватка RU
- «как проверить нагрузку?» → Metrics → Request Units
- «как понять перегрузку?» → Normalized RU Consumption ≈ 100%
- «как решить проблему?» → увеличить RU или оптимизировать запрос

## Throttling (429 Errors)

### What is Throttling?

**Request Rate Too Large**:

```
Provisioned: 1,000 RU/s
Consumed: 1,200 RU/s (burst)
     ↓
Result: 429 Too Many Requests
Retry-After: 50 ms
```

### Handling Throttling

```csharp
// SDK handles retries automatically
var clientOptions = new CosmosClientOptions
{
    MaxRetryAttemptsOnRateLimitedRequests = 9,
    MaxRetryWaitTimeOnRateLimitedRequests = TimeSpan.FromSeconds(30)
};

var client = new CosmosClient(endpoint, key, clientOptions);

// SDK will:
// 1. Catch 429 errors
// 2. Wait retry-after duration
// 3. Retry up to 9 times
// 4. Total wait up to 30 seconds
```

### Avoiding Throttling

**Solutions**:

1. **Increase provisioned RU/s**
```bash
az cosmosdb sql container throughput update \
  --name mycontainer \
  --throughput 2000
```

2. **Use autoscale**
```bash
az cosmosdb sql container update \
  --name mycontainer \
  --max-throughput 4000
```

3. **Optimize queries**
```csharp
// ❌ Expensive
"SELECT * FROM c"

// ✅ Efficient
"SELECT c.id, c.name FROM c WHERE c.category = @category"
```

4. **Batch operations**
```csharp
// ❌ Individual creates (5 RU × 100 = 500 RU)
for (int i = 0; i < 100; i++)
{
    await container.CreateItemAsync(items[i]);
}

// ✅ Batch create (more efficient)
var batch = container.CreateTransactionalBatch(partitionKey);
for (int i = 0; i < 100; i++)
{
    batch.CreateItem(items[i]);
}
await batch.ExecuteAsync();
```

## Cost Optimization

### 1. Right-Size Throughput

```
Monitor actual usage:
Peak: 800 RU/s
Average: 400 RU/s

Provisioned: 1,000 RU/s → Wasted capacity

Optimize:
Use autoscale: 400-1,000 RU/s
Save: ~40% on off-peak hours
```

### 2. Use Serverless for Dev/Test

```
Development environment:
Provisioned: 400 RU/s = $23.36/month × 24/7

Serverless: ~10 million RUs/month = $2.50/month
Savings: $20.86/month per environment
```

### 3. Shared Throughput

```
10 small containers:
Dedicated: 10 × 400 RU/s = $233.60/month
Shared: 1,000 RU/s = $58.40/month
Savings: $175.20/month (75%)
```

### 4. Optimize Indexing

```csharp
// Reduce indexing overhead
var indexingPolicy = new IndexingPolicy
{
    IndexingMode = IndexingMode.Consistent,
    IncludedPaths =
    {
        new IncludedPath { Path = "/category/?" },
        new IncludedPath { Path = "/price/?" }
    },
    ExcludedPaths =
    {
        new ExcludedPath { Path = "/*" }  // Exclude all others
    }
};

// Result: Lower write costs (less indexing)
```

# Critical Notes (Критически важные моменты)

- 💡 **1 RU** — стоимость чтения элемента 1 KB по ID + partition key
- 🎯 **Режимы** — Manual, Autoscale, Serverless
- ✅ **Manual** — фиксированные RU/s, самая низкая стоимость при стабильной нагрузке
- ⚠️ **Autoscale** — 10%–100% от max RU, автоматическое масштабирование, ~1.5× дороже Manual
- 🔄 **Serverless** — оплата за фактические запросы, без резервирования, ~$0.25 за 1 млн RU
- 📊 **Point read** — 1 RU за 1 KB (самый эффективный вариант)
- 💡 **Записи (writes)** — ~5 RU за 1 KB (из-за индексации)
- ✅ **Запросы (queries)** — 2–1000+ RU (зависит от сложности)
- ⚠️ **Throttling** — 429 ошибка при превышении лимита RU
- 🔒 **Минимум** — 400 RU/s (Manual), 400 RU/s минимум для Autoscale (10% от max)

---

# Exam Tips (AZ-204)

## Основы RU

- RU — нормализованная стоимость операций.
- 1 RU → чтение 1 KB по ID + partition key.
- Формула point read:  
  `Размер (KB) × 1 RU`
- Формула записи:  
  `Размер (KB) × ~5 RU`

---

## Режимы выделения

### Manual
- Фиксированные RU/s.
- Предсказуемая стоимость.
- Возможен throttling (429).

### Autoscale
- Масштабируется от 10% до 100% max RU.
- Автоматически реагирует на нагрузку.
- ~1.5× дороже Manual.
- Минимум = 10% от max (например: max 4000 → min 400).

### Serverless
- Нет резервирования.
- Оплата за фактическое потребление.
- ~$0.25 за 1 млн RU.
- Ограничения: 5 000 RU/s на контейнер, 50 GB хранения.

---

## Throughput

- Минимум 400 RU/s (Manual).
- Shared throughput → до 25 контейнеров делят RU базы.
- Dedicated throughput → гарантированные RU для контейнера (дороже).

---

## Ошибки и масштабирование

- 429 → превышен лимит RU.
- Retry-After header → SDK автоматически ждёт перед повтором.
- Scale-down ограничение:  
  `Max(400, Storage GB / 100 × 10) RU/s`

---

## Согласованность и RU

- Strong ≈ ~2× дороже Session/Eventual.
- Session — лучший баланс.
- Eventual — минимальная стоимость.

---

## Оптимизация запросов

- Используйте partition key в WHERE.
- Избегайте cross-partition query.
- Предпочитайте point read.

---

## Расчёт стоимости
RU/s × часы × $0.008 / 100 RU/s


---

## Free Tier

- 1000 RU/s бесплатно
- 25 GB хранения бесплатно
- Только для первого аккаунта

---

## Частые экзаменационные ловушки

- «Минимальная стоимость» → Manual.
- «Переменная нагрузка» → Autoscale.
- «Редкие запросы / dev» → Serverless.
- «429 error» → недостаточно RU.
- «Самая дешёвая операция» → Point read + partition key.


[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-cosmos-db/7-cosmos-db-request-units)
