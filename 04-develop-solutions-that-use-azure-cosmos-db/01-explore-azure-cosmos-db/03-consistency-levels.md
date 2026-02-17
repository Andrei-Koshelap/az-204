# Azure Cosmos DB Consistency Levels
(Уровни консистентности в Azure Cosmos DB)

## Key Concepts (Ключевые концепции)

- 🔄 **Consistency spectrum**  
  — 5 уровней консистентности: от **Strong** до **Eventual**

- ⚖ **Trade-offs**  
  — Баланс между:
    - Консистентностью
    - Доступностью
    - Задержкой (Latency)
    - Throughput (RU/s)

- 🌍 **Global scope**  
  — Уровень консистентности применяется ко всем регионам аккаунта

- 🎛 **Per-request override**  
  — Можно ослабить консистентность для конкретного запроса чтения

---

## Почему это важно

Cosmos DB позволяет выбрать подходящий баланс:

- Максимальная точность данных (Strong)
- Максимальная производительность (Eventual)
- Компромиссные варианты (Bounded Staleness, Session, Consistent Prefix)

---

> 🎯 Экзаменационный момент AZ-204:
> Cosmos DB поддерживает **5 уровней консистентности**  
> и позволяет переопределять уровень на уровне запроса.

## Consistency Spectrum

### The Five Levels

```
Strongest ←────────────────────────────→ Weakest
Highest Cost                        Lowest Cost
Lowest Availability                 Highest Availability
Highest Latency                     Lowest Latency

Strong → Bounded Staleness → Session → Consistent Prefix → Eventual
```

### Visual Representation


## Consistency Levels Overview (Обзор уровней консистентности)

### 1️⃣ Strong

- Всегда возвращает **последнюю подтверждённую запись**  
- Гарантия **линеаризуемости (linearizability)**  
- Самая высокая задержка  
- Может снижать доступность (особенно в multi-region)

> Подходит для критичных финансовых и банковских операций.

---

### 2️⃣ Bounded Staleness

- Отставание ограничено **K версиями** или **T временем**  
- Консистентность в пределах заданного окна устаревания  
- Хорошо подходит для single-write, multi-read сценариев  
- Предсказуемая степень устаревания данных  

> Используется, когда допускается небольшая задержка обновлений.

---

### 3️⃣ Session (DEFAULT)

- Консистентность в рамках **клиентской сессии**  
- Гарантия **read-your-writes**  
- Самый распространённый выбор  
- Баланс между производительностью и консистентностью  

> Рекомендуемый уровень по умолчанию для большинства приложений.

---

### 4️⃣ Consistent Prefix

- Чтение никогда не увидит записи вне порядка  
- По сути eventual consistency, но с сохранением порядка  
- Гарантирует последовательность операций записи  
- Подходит для лент новостей, социальных сетей  

---

### 5️⃣ Eventual

- Нет гарантии порядка чтения  
- Максимальная доступность  
- Минимальная задержка  
- Реплики со временем сходятся  

> Подходит для аналитики, счётчиков, non-critical данных.

---

> 🎯 Экзаменационный момент AZ-204:
> - По умолчанию используется **Session**
> - Strong снижает доступность при multi-region
> - Eventual даёт максимальную производительность
> - Всего 5 уровней консистентности


## Consistency Level Comparison (Сравнение уровней консистентности)

### Quick Reference Table (Краткая таблица)

| Level | Reads | Latency | Throughput | Availability | Use Case |
|--------|--------|----------|------------|--------------|------------|
| **Strong** | Самые актуальные | Самая высокая | Самый низкий | Самая низкая | Банкинг, склад |
| **Bounded Staleness** | В пределах заданного лага | Высокая | Низкий | Средняя | Котировки акций |
| **Session** | Ваши записи | Средняя | Средний | Высокая | Корзины, профили |
| **Consistent Prefix** | В правильном порядке | Низкая | Высокий | Высокая | Ленты новостей |
| **Eventual** | Возможна устаревшая версия | Самая низкая | Самый высокий | Самая высокая | Лайки, просмотры, аналитика |

---

## Ключевые различия

- 🔒 **Strong** — максимальная точность, но выше latency
- ⚖ **Session** — лучший баланс (используется по умолчанию)
- 🚀 **Eventual** — максимальная производительность

---

## Логика trade-off

Больше консистентности →  
⬆ Latency  
⬇ Throughput  
⬇ Availability

Меньше консистентности →  
⬇ Latency  
⬆ Throughput  
⬆ Availability

---

> 🎯 Экзаменационный момент AZ-204:
> - По умолчанию используется **Session**
> - Strong снижает доступность при multi-region
> - Eventual обеспечивает максимальную производительность

### CAP Theorem Tradeoffs

```
CAP Theorem: Choose 2 of 3
- Consistency (C)
- Availability (A)  
- Partition Tolerance (P)

Azure Cosmos DB provides:
├── Strong: C + P (may sacrifice A)
├── Bounded Staleness: C + P (within bounds)
├── Session: Balanced (C + A + P within session)
├── Consistent Prefix: A + P (ordered)
└── Eventual: A + P (maximum)
```

## Strong Consistency (Строгая консистентность)

### Characteristics (Характеристики)

**Гарантия линеаризуемости (Linearizability)**

- ✅ Всегда возвращает **последнюю подтверждённую запись**
- ✅ Запись сразу видна всем читателям
- ✅ Никогда не возвращаются частично записанные или неподтверждённые данные
- ❌ Самая высокая задержка (требуется координация между регионами)
- ❌ Возможны блокировки при сбоях региона

---

### Когда использовать

- Финансовые транзакции
- Управление складскими остатками
- Критичные бизнес-операции
- Сценарии, где допустима только актуальная версия данных

---

### Что важно понимать

- В multi-region конфигурации требует синхронной репликации
- Может снижать общую доступность системы
- Самый «дорогой» уровень с точки зрения latency

---

> 🎯 Экзаменационный момент AZ-204:
> Strong = всегда актуальные данные + максимальная задержка.


### How It Works

```
Write Request
     ↓
Primary Region Commits
     ↓
Wait for quorum in ALL regions
     ↓
Acknowledge write to client
     ↓
Read from ANY region sees latest data

Result: Highest consistency, highest cost
```

---

### Что это означает

- Запись считается успешной только после подтверждения всеми регионами
- Все регионы синхронизированы перед ответом клиенту
- Любое чтение из любого региона возвращает последнюю версию данных

---

### Итог

**Result:**  
🔒 Максимальная консистентность  
💰 Самая высокая стоимость  
⏱ Самая высокая задержка

---

> 🎯 Экзаменационный момент AZ-204:
> При Strong запись подтверждается только после репликации во все регионы.


### Multi-Region Behavior

```
App writes "value=100" to Region A
     ↓
Cosmos DB replicates to ALL regions
     ↓
Waits for majority quorum in EACH region
     ↓
Write acknowledged
     ↓
Read from Region B immediately sees "value=100"

Cost: Cross-region round-trip latency
```

---

### Что происходит

- Запись выполняется в одном регионе
- Данные синхронно реплицируются во все регионы
- Ожидается кворум в каждом регионе
- Только после этого клиент получает подтверждение
- Любое чтение из любого региона возвращает актуальное значение

---

### Цена Strong consistency

💰 **Cross-region round-trip latency**  
— задержка из-за синхронной межрегиональной репликации

---

> 🎯 Экзаменационный момент AZ-204:
> При Strong consistency запись подтверждается  
> только после согласования во всех регионах.


### Use Cases (Когда использовать Strong consistency)

✅ **Финансовые транзакции**  
— переводы денег, балансы счетов

✅ **Управление складом**  
— остатки товаров, бронирование

✅ **Системы голосования**  
— выборы, подсчёт результатов

✅ **Compliance-сценарии**  
— строгие регуляторные требования

---

### Общий принцип

Используйте **Strong consistency**, когда:

- Нельзя допустить устаревших данных
- Любая ошибка может привести к финансовым или юридическим последствиям
- Приоритет — точность данных, а не latency

---

> 🎯 Экзаменационный момент AZ-204:
> Strong consistency применяется в критичных бизнес-сценариях,  
> где данные должны быть абсолютно актуальными.


### Configuration

```bash
# Set strong consistency at account level
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --default-consistency-level Strong
```

```csharp
// Override per request (can only relax, not strengthen)
var response = await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics"),
    new ItemRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Strong
    }
);
```

## Bounded Staleness Consistency
(Консистентность с ограниченным отставанием)

### Characteristics (Характеристики)

**Гарантия настраиваемого лага**

- ✅ Отставание ограничено **K версиями** ИЛИ **T временем**
- ✅ Предсказуемая максимальная устарелость данных
- ✅ Подходит для single-write / multi-read сценариев
- ⚠️ Записи могут быть throttled, если превышены заданные границы лага

---

## Configuration Options (Параметры настройки)

### Staleness Window (Окно устаревания)

| Parameter | Description | Min | Max |
|------------|------------|------|------|
| **K versions** | Максимальное число версий отставания | 1 | 1 000 000 |
| **T time (sec)** | Максимальный временной лаг (в секундах) | 5 | 86 400 (24 часа) |

---

### Как это работает

- Если используется **K versions**, чтение может отставать не более чем на K обновлений
- Если используется **T time**, чтение может отставать не более чем на T секунд
- Cosmos DB гарантирует, что данные никогда не будут старше заданного окна

---

### Когда использовать

- Котировки акций
- Каталоги товаров
- Системы, где допустима небольшая задержка обновлений

---

> 🎯 Экзаменационный момент AZ-204:
> Bounded Staleness ограничивает устаревание  
> либо количеством версий (K), либо временем (T).


### How It Works

```
Multi-Region Configuration:
K = 100 versions
T = 300 seconds (5 minutes)

Write to Region A: version 1000
     ↓
Region B might read: version 900-1000 (within K)
     ↓
If lag > K versions or T time:
     ↓
Writes throttled until caught up
```

### Single-Region Behavior

⚠️ **Important**: For single-region accounts:
- Bounded Staleness = Session + Eventual consistency guarantees
- No staleness bounds enforced (only one region)

### Multi-Region Behavior

```
Primary Region (Write)
     ↓ Async replication
Secondary Regions (Read)
     ↓
Lag monitored per physical partition
     ↓
If lag > configured bounds:
     ↓
Writes throttled to maintain guarantee
```

### Use Cases (Когда использовать Bounded Staleness)

✅ **Биржевые котировки**  
— данные могут отставать на несколько секунд

✅ **Мониторинговые панели (Dashboards)**  
— допустима небольшая задержка отображения

✅ **Лидерборды**  
— ранжирование в режиме «почти реального времени»

✅ **Новостные ленты**  
— небольшая задержка публикации приемлема

---

### Общая логика

Используйте **Bounded Staleness**, когда:

- Нужна предсказуемая задержка
- Полная строгая консистентность избыточна
- Важно сохранить баланс между производительностью и точностью

---

> 🎯 Экзаменационный момент AZ-204:
> Bounded Staleness гарантирует ограниченное устаревание  
> (по времени или по количеству версий).


### Configuration

```bash
# Set bounded staleness with K versions
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --default-consistency-level BoundedStaleness \
  --max-staleness-prefix 100

# Set bounded staleness with T time
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --default-consistency-level BoundedStaleness \
  --max-interval 300  # 5 minutes
```

## Session Consistency (DEFAULT)

### Session Consistency (DEFAULT)

### Characteristics (Характеристики)

**Read-your-writes в рамках одной сессии**

- ✅ Клиент сразу видит свои собственные записи
- ✅ Monotonic reads — чтения не «откатываются назад»
- ✅ Monotonic writes — записи сохраняют порядок
- ✅ Гарантия read-your-writes
- ✅ Гарантия write-follows-reads
- ⚠️ Гарантии действуют только в рамках одной клиентской сессии

---

### Что это означает

- Один пользователь всегда видит свои изменения
- Другие пользователи могут видеть данные с небольшой задержкой
- Лучший баланс между latency и консистентностью

---

### Почему это уровень по умолчанию

- Подходит для большинства приложений
- Не требует межрегиональной синхронной координации
- Обеспечивает хорошую производительность

---

> 🎯 Экзаменационный момент AZ-204:
> По умолчанию используется **Session consistency**,  
> гарантирующая read-your-writes в рамках сессии клиента.


### Session Scope

```
Client A Session:
Write "value=100"
     ↓
Read sees "value=100" (guaranteed)
     ↓
Write "value=200"
     ↓
Read sees "value=200" (monotonic)

Client B Session (different):
Might see "value=100" (eventual with A's writes)
```

### Session Token

**How sessions work**:

```csharp
// Write returns session token
ItemResponse<Product> writeResponse = await container.CreateItemAsync(product);
string sessionToken = writeResponse.Headers.Session;

// Use session token in subsequent reads
ItemResponse<Product> readResponse = await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics"),
    new ItemRequestOptions { SessionToken = sessionToken }
);

// SDK manages session tokens automatically for same client
```

### Session Consistency Guarantees (Гарантии Session consistency)

| Guarantee | Description | Example |
|------------|------------|------------|
| **Read-your-writes** | Клиент видит свои собственные записи | Записал → сразу прочитал |
| **Monotonic reads** | Чтения не возвращаются к более старой версии | Прочитал v2 → больше никогда не увидишь v1 |
| **Monotonic writes** | Записи сохраняют порядок | Если A записано до B, порядок будет одинаков для всех |
| **Write-follows-reads** | Запись учитывает ранее прочитанные данные | Прочитал v1 → записал изменения на основе v1 |

---

### Что это даёт

- Предсказуемое поведение для одного пользователя
- Отсутствие «скачков назад» в данных
- Удобно для веб- и мобильных приложений

---

> 🎯 Экзаменационный момент AZ-204:
> Session гарантирует read-your-writes и monotonic reads  
> только в рамках одной клиентской сессии.

### Multi-Region Behavior

```
Write to Region A
     ↓
Session token includes write LSN (Logical Sequence Number)
     ↓
Read from Region B with session token
     ↓
Region B ensures read is >= LSN from token
     ↓
If not caught up, waits or reads from another region
```

### Use Cases (Когда использовать Session consistency)

✅ **Shopping carts**  
— пользователь видит товары, которые только что добавил

✅ **User profiles**  
— пользователь сразу видит свои изменения

✅ **Редактирование документов**  
— отображаются собственные правки

✅ **Большинство приложений**  
— оптимальный баланс между производительностью и консистентностью

---

### Общий принцип

Используйте **Session consistency**, когда:

- Важно, чтобы пользователь видел свои изменения
- Полная строгая консистентность избыточна
- Нужен баланс между latency, throughput и доступностью

---

> 🎯 Экзаменационный момент AZ-204:
> Session — уровень по умолчанию и лучший выбор для большинства приложений.


### Configuration

```bash
# Set session consistency (default)
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --default-consistency-level Session
```

💡 **Session — уровень по умолчанию**  
Не нужно явно настраивать, если вы не меняете уровень с другого значения.

---

## Consistent Prefix Consistency
(Консистентность с сохранением порядка)

### Characteristics (Характеристики)

**Ordered eventual consistency**

- ✅ Чтения никогда не видят записи вне порядка
- ✅ Сохраняются причинно-следственные связи
- ✅ В конечном итоге данные становятся консистентными
- ❌ Возможны устаревшие данные
- ❌ Нет гарантированного ограничения по времени или версиям устаревания

---

### Что это означает

Если операции записи происходят в порядке:
A → B → C
Чтение может увидеть:
A
A → B
Но никогда:
A → C (без B)

### How It Works

```
Writes: A → B → C (in order)
     ↓
Possible reads:
✅ A
✅ A, B
✅ A, B, C
❌ B (without A)
❌ A, C (without B)
❌ C, A (out of order)

Guarantee: If you see B, you've seen A
```

### Causal Consistency Example


Social Media Scenario (Пример для Consistent Prefix)

Write 1: User posts → "Check out this photo!"
Write 2: User uploads → photo.jpg

### Что гарантирует Consistent Prefix

- ✅ Никогда не увидим **photo.jpg без самого поста**
- ✅ Никогда не увидим события вне логического порядка
- ⏳ Может потребоваться время, чтобы увидеть оба изменения
- 🔢 Но порядок всегда будет корректным

---

### Как это может выглядеть при чтении

Возможные состояния:
(ничего)
↓
Post
↓
Post + Photo

Но никогда:
Photo (без Post) ❌

Consistent Prefix ensures:
- Never see photo without post
- Never see post without context
- May take time to see both
- But always in correct order


---

### Когда это полезно

- Социальные сети
- Ленты активности
- Event-driven системы
- Сценарии, где важен порядок, но допустима задержка

---

> 🎯 Экзаменационный момент AZ-204:
> Consistent Prefix сохраняет порядок операций,
> но не гарантирует их немедленную видимость.


### Single vs Batch Writes

#### Single Document Writes
```csharp
// Single writes: eventual consistency
await container.CreateItemAsync(doc1);
await container.CreateItemAsync(doc2);

// Readers might see:
// - Neither
// - doc1 only
// - Both
// - doc2 only (NOT guaranteed ordered for separate writes)
```

#### Transactional Batch
```csharp
// Batch writes: consistent prefix guaranteed
TransactionalBatch batch = container.CreateTransactionalBatch(
    new PartitionKey("category")
);
batch.CreateItem(doc1);
batch.CreateItem(doc2);
await batch.ExecuteAsync();

// Readers see:
// - Neither
// - Both (in order)
// - Never doc2 without doc1
```

### Use Cases

✅ **Social media feeds** - Posts in chronological order
✅ **Comment threads** - Replies after parent comments
✅ **Event streams** - Events in sequence
✅ **Audit logs** - Ordered history

### Configuration

```bash
# Set consistent prefix
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --default-consistency-level ConsistentPrefix
```

## Eventual Consistency
(Слабая консистентность)

### Characteristics (Характеристики)

**Минимальные гарантии, максимальная производительность**

- ❌ Нет гарантии порядка операций
- ❌ Возможны устаревшие данные
- ❌ Можно увидеть значение старее предыдущего чтения
- ✅ Минимальная задержка
- ✅ Максимальный throughput
- ✅ Максимальная доступность
- ✅ Реплики со временем сходятся (eventually converge)

---

### Что это означает

- Чтение может вернуть старую версию данных
- Разные регионы могут временно содержать разные значения
- Со временем данные синхронизируются

---

### Когда использовать

- Лайки и просмотры
- Счётчики
- Аналитика
- Telemetry / IoT

---

### Главное преимущество

Максимальная производительность и доступность  
при минимальных требованиях к консистентности.

---

> 🎯 Экзаменационный момент AZ-204:
> Eventual — самый слабый уровень консистентности  
> и самый быстрый по latency.

### How It Works

```
Write "value=100"
     ↓
Asynchronous replication to all regions
     ↓
Read from Region A: might see "value=100"
Read from Region B: might see old value
Read again from Region A: might see older value (time warp)
     ↓
Eventually all regions converge to "value=100"

No guarantees on timing or ordering
```

### Non-Monotonic Reads Example

```
Timeline:
T0: value=0 (in all regions)
T1: Write value=100 (Region A)
T2: Read from Region A → value=100 ✓
T3: Read from Region B → value=0 ⚠️ (went backwards!)
T4: All regions caught up → value=100
```

### Use Cases (Когда использовать Eventual consistency)

✅ **Аналитика**  
— агрегации, подсчёты (допустима eventual-точность)

✅ **Социальные счётчики**  
— лайки, просмотры, репосты

✅ **Некритичные метрики**  
— количество загрузок, просмотры страниц

✅ **Кэширование**  
— read-heavy сценарии с редкими обновлениями

✅ **IoT telemetry**  
— данные датчиков (точность отдельных значений не критична)

---

### Anti-Patterns (Когда НЕ использовать)

❌ Пользовательские данные, требующие строгой консистентности  
❌ Финансовые операции  
❌ Управление складскими остатками  
❌ Сценарии, где требуется read-your-writes

---

### Общий принцип

Используйте **Eventual consistency**, когда:

- Производительность важнее точности
- Допустима временная рассинхронизация данных
- Ошибка в несколько секунд или версий не критична

---

> 🎯 Экзаменационный момент AZ-204:
> Eventual — максимальная производительность,  
> но нет гарантий порядка и актуальности данных.


### Configuration

```bash
# Set eventual consistency
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --default-consistency-level Eventual
```

## Consistency Level Scope

### Region-Agnostic

**All consistency levels work globally**:

```
Account with 5 regions:
├── Region 1 (East US)
├── Region 2 (West US)
├── Region 3 (Europe)
├── Region 4 (Asia)
└── Region 5 (Australia)

Consistency level applies uniformly across ALL regions
```

### Guaranteed Regardless Of (Гарантируется независимо от)

✅ **Региона обслуживания чтения/записи**  
— Работает в любом регионе

✅ **Количество регионов**  
— 1 регион или 50 регионов

✅ **Конфигурации записи**  
— Single-write или multi-write

---

## Partition-Scoped Operations
(Ограничения области консистентности)

### Read consistency применяется к:

- 🔹 Одной операции чтения
- 🔹 В пределах одного диапазона partition key
- 🔹 Или в пределах одной logical partition

---

### Важно понимать

- Гарантии консистентности действуют **на уровне операции**
- Транзакции ограничены одной logical partition
- Для максимальной производительности запросы должны быть partition-aware

---

> 🎯 Экзаменационный момент AZ-204:
> Гарантии консистентности применяются  
> к отдельной операции чтения и в пределах partition.


```csharp
// Consistency applies to this specific read
var item = await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics")  // Scoped to this partition
);

// Different partition = independent consistency view
var item2 = await container.ReadItemAsync<Product>(
    "product-2",
    new PartitionKey("books")  // Different consistency view
);
```

## Configuring Consistency

### Account-Level Default

```bash
# Set default consistency for entire account
az cosmosdb update \
  --name mycosmosaccount \
  --resource-group myResourceGroup \
  --default-consistency-level Session

# Applies to:
# - All databases
# - All containers
# - All operations (unless overridden)
```

### Per-Request Override

**Can only relax consistency, not strengthen**:

```csharp
// Account default: Session

// ✅ Can relax to Eventual
var response = await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics"),
    new ItemRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Eventual  // Weaker OK
    }
);

// ❌ Cannot strengthen to Strong (if account default is Session)
var response2 = await container.ReadItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics"),
    new ItemRequestOptions
    {
        ConsistencyLevel = ConsistencyLevel.Strong  // Error!
    }
);
```

### Правила переопределения уровней согласованности (Override Rules)

| Уровень учетной записи | Можно понизить до |
|------------------------|-------------------|
| **Strong (Строгая согласованность)** | Нет (уже самый высокий уровень) |
| **Bounded Staleness (Ограниченная устаревание)** | Session, Consistent Prefix, Eventual |
| **Session (Сеансовая)** | Consistent Prefix, Eventual |
| **Consistent Prefix (Согласованный префикс)** | Eventual |
| **Eventual (В конечном итоге)** | Нет (уже самый слабый уровень) |

---

### Дополнительные пояснения

- Переопределение возможно **только в сторону ослабления согласованности**.
- Нельзя повысить уровень согласованности в рамках запроса выше уровня, установленного на уровне учетной записи.
- Более высокий уровень согласованности = выше гарантия целостности данных, но потенциально выше задержка.
- Более низкий уровень согласованности = выше производительность и масштабируемость, но меньше гарантий актуальности данных.

### Практический совет для AZ-204

На экзамене важно помнить:
- **Strong** обеспечивает линейную согласованность (чтение всегда возвращает последнюю запись).
- **Session** — наиболее часто используемый баланс между производительностью и гарантией чтения своих записей (read-your-own-writes).
- **Eventual** — лучший выбор для глобально распределённых систем с высокой нагрузкой, где допустима временная несогласованность.


## Performance & Cost Impact

### Read Latency

```
Strong:             ~20-50ms (cross-region quorum)
Bounded Staleness:  ~10-20ms (within bounds)
Session:            ~5-10ms (local + session token)
Consistent Prefix:  ~5-10ms (local read)
Eventual:           ~5ms (local read)
```

### RU Consumption

**Relative RU cost** for same operation:

```
Strong:             2x RU (cross-region coordination)
Bounded Staleness:  1.5x RU (monitoring lag)
Session:            1x RU (baseline)
Consistent Prefix:  1x RU
Eventual:           1x RU
```

### Throughput Impact

```
Account: 10,000 RU/s provisioned

Strong consistency:
Effective: ~5,000 ops/sec

Eventual consistency:
Effective: ~10,000 ops/sec

Reason: Strong requires cross-region coordination
```

## Choosing the Right Level

### Decision Tree

```
Do you need linearizability (banking, inventory)?
├── Yes → Strong
└── No ↓

Do you need bounded staleness (stock quotes, monitoring)?
├── Yes → Bounded Staleness (configure K and T)
└── No ↓

Do you need read-your-writes (user-facing apps)?
├── Yes → Session (DEFAULT - best for most apps)
└── No ↓

Do you need ordered reads (feeds, logs)?
├── Yes → Consistent Prefix
└── No ↓

Analytics, counters, non-critical?
└── Yes → Eventual
```

### Типовые сценарии использования (Common Scenarios)

| Сценарий | Рекомендуемый уровень | Причина |
|-----------|----------------------|----------|
| **Корзина интернет-магазина** | Session | Пользователь видит собственные действия (read-your-writes) |
| **Банковские операции** | Strong | Критическая важность корректности денежных данных |
| **Лента социальной сети** | Consistent Prefix | Сохранение хронологического порядка событий |
| **Счётчик просмотров страницы** | Eventual | Абсолютная точность не критична в реальном времени |
| **Котировки акций** | Bounded Staleness | Допустима небольшая задержка обновления |
| **Профиль пользователя** | Session | Пользователь видит внесённые изменения |
| **Аналитическая панель (Dashboard)** | Eventual | Агрегированные данные, допустима eventual-точность |

---

## Важные замечания (Critical Notes)

- 💡 **Пять уровней согласованности** — Strong, Bounded Staleness, Session, Consistent Prefix, Eventual
- 🎯 **Session — уровень по умолчанию** — оптимальный баланс для большинства приложений
- ✅ **Независимость от региона (Region-agnostic)** — применяется глобально во всех регионах
- ⚠️ **Правило переопределения** — можно только ослабить согласованность, усилить нельзя
- 🔄 **Компромисс** — Consistency ↔ Availability + Latency + Throughput
- 📊 **Strong** — максимальная согласованность, высокая стоимость RU, возможное снижение доступности
- 💡 **Session** — гарантия "read-your-writes" в рамках клиентской сессии
- ✅ **Bounded Staleness** — настраиваемая задержка (K версий или T времени)
- ⚠️ **Eventual** — нет гарантии порядка, максимальная производительность и минимальная стоимость
- 🔒 **100% гарантия SLA** — все уровни согласованности поддерживаются SLA

Дополнение:
- Чем выше согласованность, тем больше потребление RU (Request Units).
- При глобальной репликации Strong может увеличить задержки между регионами.
- В большинстве реальных cloud-сценариев используется **Session**.

---

## Подсказки для экзамена AZ-204 (Exam Tips)

- Уровень по умолчанию: **Session** (read-your-writes в рамках сессии)
- Самый строгий: **Strong** (линеаризуемость, всегда последнее записанное значение)
- Самый слабый: **Eventual** (нет гарантии порядка, максимальная доступность)
- **Bounded Staleness**: задержка ограничена K версиями ИЛИ T временем (что наступит раньше)
- Ограничения Bounded Staleness:
    - K: от 1 до 1 000 000 версий
    - T: от 5 секунд до 24 часов
- Гарантии Session:
    - Read-your-writes
    - Monotonic reads
    - Monotonic writes
- **Consistent Prefix** — упорядоченная eventual-согласованность (никогда не будет "перепутанного" порядка)
- Правило override — можно только ослаблять уровень
- Стоимость Strong по RU — примерно в 2 раза выше, чем Session/Eventual
- Region-agnostic — согласованность применяется ко всем регионам
- Bounded Staleness в одном регионе — по поведению близок к Session + Eventual
- Session token — автоматически управляется SDK
- 100% гарантия — все чтения соответствуют SLA выбранного уровня
- Область действия — внутри диапазона partition key
- Account-level — уровень по умолчанию задаётся на уровне аккаунта, можно переопределить в запросе

---

### Типовые use cases

- **Strong** — банковские операции, инвентаризация, голосования
- **Session** — корзины, профили пользователей (большинство приложений)
- **Eventual** — аналитика, счётчики, некритичные метрики

---

### Спектр согласованности

Strong → Bounded Staleness → Session → Consistent Prefix → Eventual

---

### Связь с CAP-теоремой

- **Strong** → C + P (жертвуем доступностью при сбоях)
- **Session** → баланс
- **Eventual** → A + P (жертвуем строгой согласованностью ради доступности)

[Learn More](https://learn.microsoft.com/en-us/training/modules/explore-azure-cosmos-db/4-cosmos-db-consistency-levels-overview)
