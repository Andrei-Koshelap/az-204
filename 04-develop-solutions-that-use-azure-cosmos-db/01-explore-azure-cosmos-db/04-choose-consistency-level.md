# Выбор правильного уровня согласованности (Choose the Right Consistency Level)

## Ключевые понятия (Key Concepts)

- **Согласованность по умолчанию (Default consistency)** — задаётся на уровне аккаунта и применяется ко всем операциям, если не указано иное.
- **Переопределение на уровне запроса (Per-request override)** — можно ослабить уровень согласованности для конкретной операции (усилить нельзя).
- **Соответствие бизнес-требованиям (Use case alignment)** — уровень должен соответствовать требованиям предметной области.
- **Понимание компромиссов (Tradeoff understanding)** — необходимо балансировать между согласованностью, доступностью, производительностью и стоимостью (RU).

---

## Как правильно выбирать уровень

### 1️⃣ Определите критичность данных
- Деньги, остатки, голосования → **Strong**
- Данные пользователя (профиль, корзина) → **Session**
- Ленты, события → **Consistent Prefix**
- Метрики, аналитика → **Eventual**

### 2️⃣ Учитывайте географию
- При многорегиональной репликации Strong увеличивает задержки.
- Для глобальных приложений чаще всего оптимален **Session**.

### 3️⃣ Оцените стоимость (RU)
- Strong ≈ выше потребление RU.
- Eventual и Session — более экономичные варианты.
- Чем выше согласованность, тем выше latency.

### 4️⃣ Используйте override разумно
Пример стратегии:
- По умолчанию — **Session**
- Для аналитических чтений — override на **Eventual**
- Для критической операции — заранее выбрать Strong на уровне аккаунта (если требуется)

---

## Быстрая шпаргалка для экзамена AZ-204

- Default → задаётся на уровне аккаунта
- Override → только ослабление
- Strong → максимальная точность
- Session → лучший баланс (по умолчанию)
- Eventual → максимум производительности
- Всегда думайте о компромиссе:  
  **Consistency ↔ Availability + Latency + Throughput + Cost**

## Configuring Default Consistency

### Account-Level Setting

**Applies to all operations** unless overridden:

```bash
# Set default consistency at account creation
az cosmosdb create \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --locations regionName=eastus \
  --default-consistency-level Session

# Update existing account
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --default-consistency-level BoundedStaleness \
  --max-staleness-prefix 100 \
  --max-interval 300
```

### Что получает уровень согласованности по умолчанию? (What Gets the Default?)

**Наследуют уровень, заданный на уровне аккаунта:**

- ✅ Все базы данных
- ✅ Все контейнеры
- ✅ Все операции чтения (read operations)
- ✅ Все операции запросов (query operations)
- ⚠️ Если уровень явно не переопределён в конкретном запросе

---

### Важно помнить

- Значение по умолчанию задаётся **на уровне аккаунта Cosmos DB** и автоматически применяется ко всем ресурсам внутри него.
- Это означает, что при создании новой базы или контейнера дополнительная настройка согласованности не требуется.
- Переопределение возможно **только для операций чтения** (write-операции всегда используют уровень аккаунта).
- Если override не указан — всегда используется account-level consistency.

---

### Экзаменационный акцент (AZ-204)

Если в вопросе:
- не указано переопределение,
- нет mention о session token,
- нет явного указания уровня в запросе,

→ значит применяется **уровень согласованности аккаунта**.


```
Account: Session consistency
     ↓
Database 1: Session (inherited)
├── Container A: Session
└── Container B: Session
     ↓
Database 2: Session (inherited)
└── Container C: Session

All operations use Session unless overridden
```

## Consistency Level Guarantees

### Strong Consistency

**Linearizability guarantee**:

```
Timeline:
T1: Client A writes value=100
T2: Write committed to quorum in all regions
T3: Client B reads from any region → value=100 (guaranteed)

Guarantee: Always reads most recent committed write
Cost: Highest latency, lowest throughput
```

## Характеристики (Strong Consistency)

**Гарантии:**

- ✅ Никогда не возвращает неподтверждённые (uncommitted) записи
- ✅ Никогда не возвращает частично записанные данные
- ✅ Всегда возвращает последнюю подтверждённую версию данных
- ❌ Требует координации между регионами (cross-region coordination)
- ❌ Может снижать доступность при сбоях

---

## Когда использовать


**When to use**:
```
✅ Банкинг: Балансы счетов, переводы
✅ Инвентаризация: Остатки товаров, резервации
✅ Голосование: Подсчёт результатов
✅ Регуляторные требования: Комплаенс, аудит
❌ Социальные сети: Не оправдывает стоимость
❌ Аналитика: Избыточно для агрегированных данных
```
---

## Дополнительные пояснения

- Обеспечивает **линеаризуемость (linearizability)** — каждое чтение видит самый последний успешный write.
- В глобально распределённой системе увеличивает задержки из-за необходимости синхронизации.
- При сетевых разделениях (network partition) может временно снижать доступность.
- Потребление RU выше, чем у Session или Eventual.

---

## Экзаменационный ориентир (AZ-204)

Если в вопросе:
- требуется «всегда последнее значение»,
- важна абсолютная корректность,
- система связана с деньгами или юридической ответственностью,

→ правильный выбор: **Strong**.


### Bounded Staleness Consistency (Ограниченная устаревание)

**Данные могут быть устаревшими, но строго в заданных пределах.**


Configuration:
K = 100 versions
T = 5 minutes

#### Сценарий

- Основной регион (Primary) находится на версии **1000**
- Вторичный регион (Secondary) может вернуть версии **900–1000** (в пределах K)

Если задержка превышает заданные границы:
- Записи (writes) начинают **throttle-иться**
- Реплики должны «догнать» основной регион
- Гарантия устаревания сохраняется

---

### Параметры окна устаревания (Staleness Window)

| Тип ограничения | Минимум | Максимум | Когда использовать |
|-----------------|----------|-----------|--------------------|
| **K версий** | 1 | 1 000 000 | Когда важна предсказуемая разница в версиях |
| **T времени** | 5 сек | 86 400 сек (24 часа) | Когда нужны гарантии по времени |
**Multi-Region Behavior**:

Write Region: East US (version 1000)
     ↓
Read Region: West US
     ↓
If lag > K or T:
     ├── Reads see data within bounds
     └── Writes throttled to maintain guarantee


**Single-Region Behavior**:

⚠️ Important: For single-region accounts:
Bounded Staleness = Session + Eventual consistency
No staleness bounds enforced (only one region to sync)


**When to use**:

✅ Stock quotes: Near real-time (5-10 sec lag OK)
✅ Monitoring: Dashboard metrics (recent data acceptable)
✅ Leaderboards: Gaming scores (slight delay OK)
❌ Banking: Need exact values
❌ User profiles: Session better for read-your-writes


### Session Consistency

**Read-your-writes within session**:


Client Session:
├── Write: value=100
├── Read: value=100 (guaranteed - read-your-write)
├── Write: value=200
└── Read: value=200 (guaranteed - monotonic reads)

Different Client:
└── Might see value=100 or 200 (eventual with other clients)


---

## Гарантии Session

| Гарантия | Описание | Пример |
|-----------|------------|---------|
| **Read-your-writes** | Видите свои записи сразу | Добавили товар в корзину — он отображается |
| **Monotonic reads** | Никогда не «откатитесь назад» | Увидели v2 — позже не увидите v1 |
| **Monotonic writes** | Порядок записей сохраняется | Write A перед Write B |
| **Write-follows-reads** | Запись учитывает ранее прочитанные данные | Прочитали v1 → обновили на основе v1 |

---

## Дополнительные пояснения

- Это уровень по умолчанию в Cosmos DB.
- Session token управляется SDK автоматически.
- Даёт лучший баланс между стоимостью и корректностью.
- В глобальных системах используется чаще всего.

---

## Экзаменационный ориентир (AZ-204)

Если в вопросе:
- пользователь должен видеть свои изменения,
- система глобальная,
- не требуется абсолютная строгость,

→ выбирайте **Session**.

**Session Token Management**:

```csharp
// SDK manages automatically for same client instance
CosmosClient client = new CosmosClient(endpoint, key);
var container = client.GetContainer("db", "container");

// Write
await container.CreateItemAsync(new Product { Id = "1", Name = "Laptop" });

// Read - automatically sees own write (session token managed by SDK)
var item = await container.ReadItemAsync<Product>("1", new PartitionKey("1"));

// Manual session token (advanced scenarios)
ItemResponse<Product> writeResponse = await container.CreateItemAsync(product);
string sessionToken = writeResponse.Headers.Session;

// Use in subsequent read
await container.ReadItemAsync<Product>(
    "1",
    new PartitionKey("1"),
    new ItemRequestOptions { SessionToken = sessionToken }
);
```

**When to use**:
```
✅ Shopping carts: User sees their actions
✅ User profiles: See own edits
✅ Document editing: See your changes
✅ Most web apps: Default choice for user-facing apps
❌ Multi-user collaboration: Users don't see each other's changes immediately
```

### Consistent Prefix Consistency

**Ordered eventual consistency**:

```
Writes: A → B → C → D (in order)
     ↓
Possible reads:
✅ Empty
✅ A
✅ A, B
✅ A, B, C
✅ A, B, C, D
❌ B (missing A)
❌ A, C (missing B)
❌ D, A (out of order)

Guarantee: Never out-of-order, but may be stale
```

**Transaction Behavior**:

```csharp
// Single writes: eventual consistency
await container.CreateItemAsync(doc1);  // Write 1
await container.CreateItemAsync(doc2);  // Write 2
// Readers might see doc2 without doc1 ⚠️

// Transactional batch: consistent prefix guaranteed
TransactionalBatch batch = container.CreateTransactionalBatch(
    new PartitionKey("category")
);
batch.CreateItem(doc1);
batch.CreateItem(doc2);
await batch.ExecuteAsync();
// Readers see neither or both (in order) ✅
```

**When to use**:
```
✅ Social media feeds: Posts in chronological order
✅ Comment threads: Replies after parent
✅ Event streams: Sequence matters
✅ Audit logs: Ordered history
❌ User-facing writes: Need read-your-writes (use Session)
❌ Analytics: Order doesn't matter (use Eventual)
```

### Eventual Consistency

**No ordering guarantee**:

```
Timeline:
T0: value=0 (all regions)
T1: Write value=100 (Region A)
T2: Read Region A → 100
T3: Read Region B → 0 (hasn't replicated yet)
T4: Read Region A → 0 (time warp! ⚠️)
T5: All regions → 100 (eventually consistent)

Guarantee: Replicas eventually converge
No guarantees on timing or ordering
```

## Eventual Consistency (Согласованность «в конечном итоге»)

### Характеристики

- ❌ Нет гарантии порядка операций
- ❌ Можно получить устаревшие данные
- ❌ Следующее чтение может вернуть более старую версию, чем предыдущее
- ✅ Минимальная задержка (lowest latency)
- ✅ Максимальная пропускная способность (highest throughput)
- ✅ Максимальная доступность
- ✅ Минимальная стоимость (наименьшее потребление RU)

---

### Когда использовать
```
✅ Счётчики: лайки, просмотры, репосты (допустима eventual-точность)
✅ Аналитика: агрегаты, dashboards
✅ Телеметрия: IoT-датчики
✅ Кэширование: много чтений, редкие обновления
❌ Критичные пользовательские данные: лучше Session
❌ Финансовые данные: Strong
❌ Инвентаризация: Strong или Bounded Staleness
```

## Relaxing Consistency Per Request

### Override Rules

**Can only relax, not strengthen**:

```csharp
// Account default: Session

// ✅ Relax to Eventual (weaker)
var response = await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics"),
    new ItemRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Eventual
    }
);

// ❌ Strengthen to Strong (ERROR!)
var response2 = await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics"),
    new ItemRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Strong  // Throws exception!
    }
);
```

### Матрица переопределения (Override Matrix)

| Уровень по умолчанию (Account Default) | Можно переопределить до |
|----------------------------------------|--------------------------|
| **Strong** | Нет (уже самый строгий уровень) |
| **Bounded Staleness** | Session, Consistent Prefix, Eventual |
| **Session** | Consistent Prefix, Eventual |
| **Consistent Prefix** | Eventual |
| **Eventual** | Нет (уже самый слабый уровень) |

---

### Ключевые правила

- Переопределение возможно **только в сторону ослабления согласованности**.
- Нельзя «усилить» уровень согласованности в рамках запроса.
- Если аккаунт настроен на **Strong**, все операции будут выполняться с Strong.
- Если аккаунт настроен на **Eventual**, изменить уровень нельзя.

---

### Логика запоминания для AZ-204

Двигаться можно только **вправо по спектру согласованности**:

Strong → Bounded Staleness → Session → Consistent Prefix → Eventual

Никогда в обратную сторону.

---

### Практический совет

В реальных проектах:
- Обычно устанавливают **Session** как уровень по умолчанию.
- Для менее критичных чтений делают override на **Eventual**.
- Strong задаётся только если бизнес-требования требуют абсолютной точности.


### Why Relax Consistency?

**Scenarios for per-request override**:

1. **Analytics queries**:
```csharp
// Account default: Session (for user-facing operations)
// Analytics query can use Eventual (performance + cost savings)
var query = container.GetItemQueryIterator<Product>(
    new QueryDefinition("SELECT COUNT(1) FROM c"),
    requestOptions: new QueryRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Eventual
    }
);
```

2. **Background processing**:
```csharp
// User operations: Session (read-your-writes)
await container.CreateItemAsync(product);

// Background sync: Eventual (performance)
var items = container.GetItemQueryIterator<Product>(
    "SELECT * FROM c WHERE c.status = 'pending'",
    requestOptions: new QueryRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Eventual
    }
);
```

3. **Non-critical reads**:
```csharp
// Critical: User's own cart (Session)
var cart = await container.ReadItemAsync<Cart>(
    userId,
    new PartitionKey(userId)
);

// Non-critical: Product recommendations (Eventual - performance boost)
var recommendations = container.GetItemQueryIterator<Product>(
    "SELECT TOP 10 * FROM c WHERE c.category = @category",
    requestOptions: new QueryRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Eventual
    }
);
```

## Практический фреймворк выбора (Practical Decision Framework)

### Шаг 1: Определите требования

Прежде чем выбирать уровень согласованности, ответьте на ключевые вопросы о бизнес-логике и поведении системы.

### Задайте себе эти вопросы

| Вопрос | Ответ → Уровень |
|---------|----------------|
| Нужна ли линеаризуемость (всегда последнее значение)? | Да → **Strong** |
| Требуется ли многопользовательская работа в реальном времени? | Да → **Strong** или **Bounded Staleness** |
| Пользователь должен видеть собственные изменения? | Да → **Session** |
| Важен ли порядок операций? | Да → **Consistent Prefix** |
| Только аналитика или счётчики? | Да → **Eventual** |

---

## Как рассуждать на экзамене (AZ-204)

1. Если видите слова:  
   *“most recent write”, “absolute accuracy”, “financial”, “critical system”*  
   → выбирайте **Strong**.

2. Если указано:  
   *“user should immediately see their changes”*  
   → **Session**.

3. Если важно:  
   *“maintain order of events”*  
   → **Consistent Prefix**.

4. Если упор на:  
   *“high availability”, “maximum performance”, “analytics”*  
   → **Eventual**.

---

## Дополнительный профессиональный совет

- В 80% реальных облачных приложений используется **Session**.
- **Strong** выбирается редко — только при строгих требованиях.
- **Eventual** применяется там, где масштаб и стоимость важнее точной синхронности.
- Если сомневаетесь между Strong и Bounded Staleness — подумайте, допустима ли небольшая контролируемая задержка.

---

### Быстрая логика запоминания

Точность важнее всего → Strong  
Пользовательский опыт → Session  
Порядок важен → Consistent Prefix  
Производительность важнее → Eventual  
Контролируемая задержка → Bounded Staleness


### Шаг 2: Учитывайте многорегиональность (Multi-Region)

## Один регион записи (Single Write Region)
```
Scenario: Application in one region, read replicas globally

Strong: High latency for writes (cross-region quorum)
Bounded Staleness: Good choice (predictable lag)
Session: Best for user-facing (read-your-writes)
```

**Multi-write regions**:
```
Scenario: Write from any region

Strong: Very high latency (all-region quorum)
Bounded Staleness: Conflicts possible, good for certain scenarios
Session: Best default (per-user consistency)
Eventual: Conflicts common, use conflict resolution
```

### Step 3: Evaluate Tradeoffs

**Performance vs Consistency**:

```
                Consistency
                    ↑
                  Strong
                    |
              Bounded Staleness
                    |
                 Session (DEFAULT - sweet spot)
                    |
            Consistent Prefix
                    |
                 Eventual
                    ↓
              Performance →
```

### Step 4: Cost Consideration

**RU consumption comparison** (same operation):

```
Strong:             2x RU
Bounded Staleness:  1.5x RU
Session:            1x RU (baseline)
Consistent Prefix:  1x RU
Eventual:           1x RU

Example: 10,000 RU/s provisioned
Strong: Effective ~5,000 ops/sec
Eventual: Effective ~10,000 ops/sec
```

## Common Patterns

### Pattern 1: Hybrid Approach

```csharp
// Account default: Session

public class OrderService
{
    // Critical operations: Use default (Session)
    public async Task CreateOrder(Order order)
    {
        await container.CreateItemAsync(order);
        // Session ensures user sees their order immediately
    }

    // Analytics: Relax to Eventual
    public async Task<int> GetTotalOrders()
    {
        var query = container.GetItemQueryIterator<Order>(
            "SELECT VALUE COUNT(1) FROM c",
            requestOptions: new QueryRequestOptions
            {
                ConsistencyLevel = ConsistencyLevel.Eventual
            }
        );
        // Eventual: Better performance, eventual accuracy OK
    }
}
```

### Pattern 2: Progressive Consistency

```csharp
// Start with Session, upgrade to Strong if needed
public async Task<Product> GetProduct(string id, bool critical = false)
{
    var options = new ItemRequestOptions();
    
    if (critical)
    {
        // Critical operation: Cannot override to Strong if account is Session
        // Must set account to Strong and relax others
        // For now, use Session (best available)
    }
    
    return await container.ReadItemAsync<Product>(
        id,
        new PartitionKey(id),
        options
    );
}
```

### Pattern 3: Read-Heavy Optimization

```csharp
// Account: Session (for writes and user-facing reads)
// Background reads: Eventual (performance)

public async Task SyncToCache()
{
    // Background job - use Eventual for better performance
    var query = container.GetItemQueryIterator<Product>(
        "SELECT * FROM c",
        requestOptions: new QueryRequestOptions
        {
            ConsistencyLevel = ConsistencyLevel.Eventual
        }
    );
    
    await foreach (var product in query)
    {
        await cache.SetAsync(product.Id, product);
    }
}
```

## Anti-Patterns

### ❌ Anti-Pattern 1: Wrong Default

```csharp
// BAD: Account default = Eventual
// User writes then immediately reads → might not see their write ⚠️

await container.CreateItemAsync(newOrder);
var order = await container.ReadItemAsync<Order>(orderId, pk);
// Might not see the order just created!

// GOOD: Account default = Session
// Guarantees read-your-writes
```

### ❌ Anti-Pattern 2: Overusing Strong

```csharp
// BAD: Everything uses Strong consistency
// Account default: Strong
// Every operation pays 2x RU cost and high latency

// GOOD: Use Strong only where needed
// Account default: Session
// Override to Strong only for critical operations
```

### ❌ Anti-Pattern 3: Trying to Strengthen

```csharp
// BAD: Account default = Session
await container.ReadItemAsync<Product>(
    id,
    pk,
    new ItemRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Strong  // ERROR! ❌
    }
);

// GOOD: Change account default to Strong, relax others to Session
```

## Важные замечания (Critical Notes)

- 💡 **По умолчанию — Session**  
  Session consistency используется по умолчанию и подходит для большинства приложений.

- 🎯 **Уровень аккаунта (Account-level)**  
  Настраивается на уровне аккаунта и применяется ко всем операциям.

- ✅ **Правило override**  
  Можно только ослабить уровень согласованности, усилить нельзя.

- ⚠️ **Переопределение на уровне запроса (Per-request)**  
  Override применяется к конкретным операциям чтения или запросам для оптимизации.

- 🔄 **Strong**  
  Линеаризуемость (linearizability), самая высокая стоимость RU, использовать только при необходимости.

- 📊 **Session**  
  Read-your-writes в рамках клиентской сессии, лучший баланс.

- 💡 **Eventual**  
  Подходит для аналитики и счётчиков, максимальная производительность.

- ✅ **100% SLA**  
  Все уровни согласованности гарантируются SLA Azure Cosmos DB.

- 🔒 **Многорегиональность (Multi-region)**  
  Уровень согласованности применяется одинаково во всех регионах.

- ⚠️ **Bounded Staleness в одном регионе**  
  В single-region аккаунте фактически ведёт себя как Session + Eventual.

---

## Подсказки для экзамена (AZ-204)

- Default consistency → **Session** (read-your-writes).
- Конфигурируется на → **уровне аккаунта**.
- Override → только ослабление.
- Strong → линеаризуемость, всегда самое последнее значение.
- Session → read-your-writes, monotonic reads, monotonic writes.
- Bounded Staleness → задержка ограничена K версиями ИЛИ T временем.
- Consistent Prefix → порядок сохраняется, но данные могут быть устаревшими.
- Eventual → нет порядка, максимальная производительность, минимальная стоимость.

---

## Типовые сценарии

- **Strong** → банковские системы, складские остатки, голосование.
- **Session** → корзины, профили пользователей, большинство web-приложений.
- **Eventual** → аналитика, счётчики, телеметрия.

---

## Стоимость RU (условно)

- Strong ≈ ~2x
- Session ≈ 1x
- Eventual ≈ 1x

---

## Дополнительные моменты

- Bounded Staleness (single-region) ≈ Session + Eventual.
- Session token управляется автоматически SDK.
- Override часто используется для:
  - аналитических чтений (Eventual),
  - фоновых задач (Eventual).
- Multi-write + Strong → очень высокая задержка (кворум всех регионов).
- Multi-write + Session → лучший баланс для записи в нескольких регионах.
- Все чтения соответствуют SLA выбранного уровня.
- Уровень по умолчанию можно изменить в любой момент без downtime.


[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-cosmos-db/5-choose-cosmos-db-consistency-level)
