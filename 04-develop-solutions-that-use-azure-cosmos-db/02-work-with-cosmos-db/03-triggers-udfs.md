# Azure Cosmos DB Triggers and User-Defined Functions

## Ключевые понятия (Key Concepts)

- **Pre-triggers** — выполняются до операции
- **Post-triggers** — выполняются после операции
- **UDFs** — пользовательские функции для запросов
- **JavaScript** — реализуются на JavaScript

---

# Обзор

## Серверная программируемость (Server-side programmability)

| Механизм | Назначение | Когда выполняется |
|-----------|------------|------------------|
| **Pre-triggers** | Валидация / изменение входных данных | До операции |
| **Post-triggers** | Обновление метаданных / логирование | После операции |
| **UDFs** | Пользовательская логика в запросах | Во время запроса |

---

# Pre-Triggers

## Что такое Pre-Triggers?

**Выполняются ПЕРЕД операциями create / update / delete**

- **Назначение** — проверка или изменение входных данных
- **Момент выполнения** — до выполнения основной операции
- **Доступ** — могут читать и изменять тело запроса
- **Обязательность** — не обязательны, указываются явно в запросе
- **Rollback** — при выбросе ошибки операция отменяется

---

## Когда использовать Pre-Triggers

- Валидация бизнес-правил
- Установка значений по умолчанию
- Автоматическое добавление метаданных
- Контроль целостности данных до сохранения

---

## Ограничения

- Работают только в пределах одного контейнера
- Не выполняются автоматически — нужно указать при вызове
- Ограничены временем выполнения
- Не поддерживают cross-partition операции

### Basic Pre-Trigger Example

```javascript
function validateDocumentContents() {
    var context = getContext();
    var request = context.getRequest();
    
    // Get the document being created/updated
    var document = request.getBody();
    
    // Validation logic
    if (!document.id) {
        throw new Error("Document must have an id property");
    }
    
    if (!document.category) {
        throw new Error("Document must have a category");
    }
    
    // Modify document before creation
    document.createdAt = new Date().toISOString();
    
    // Update the request
    request.setBody(document);
}
```

**Register and use from C#**:

```csharp
// 1. Register the pre-trigger
var trigger = new TriggerProperties
{
    Id = "validateDocumentContents",
    Body = File.ReadAllText("validateDocument.js"),
    TriggerOperation = TriggerOperation.Create,  // or Update, Delete, All
    TriggerType = TriggerType.Pre
};

await container.Scripts.CreateTriggerAsync(trigger);

// 2. Execute with operation
var newProduct = new
{
    id = "product-1",
    category = "electronics",
    name = "Laptop",
    price = 999.99
};

ItemRequestOptions requestOptions = new ItemRequestOptions
{
    PreTriggers = new List<string> { "validateDocumentContents" }
};

ItemResponse<dynamic> response = await container.CreateItemAsync(
    newProduct,
    new PartitionKey("electronics"),
    requestOptions
);
```

### Timestamp Pre-Trigger

```javascript
function addTimestamp() {
    var context = getContext();
    var request = context.getRequest();
    var document = request.getBody();
    
    var timestamp = new Date().toISOString();
    
    // Add timestamp based on operation
    if (request.getOperationType() === "Create") {
        document.createdAt = timestamp;
        document.updatedAt = timestamp;
    } else if (request.getOperationType() === "Replace") {
        document.updatedAt = timestamp;
    }
    
    request.setBody(document);
}
```

### Validation Pre-Trigger

```javascript
function validateProduct() {
    var context = getContext();
    var request = context.getRequest();
    var document = request.getBody();
    
    // Required fields
    var requiredFields = ['id', 'name', 'category', 'price'];
    
    requiredFields.forEach(function(field) {
        if (!document[field]) {
            throw new Error(`Field '${field}' is required`);
        }
    });
    
    // Price validation
    if (typeof document.price !== 'number' || document.price < 0) {
        throw new Error("Price must be a positive number");
    }
    
    // Category validation
    var validCategories = ['electronics', 'clothing', 'food', 'books'];
    if (validCategories.indexOf(document.category) === -1) {
        throw new Error("Invalid category");
    }
    
    request.setBody(document);
}
```

### Data Enrichment Pre-Trigger

```javascript
function enrichDocument() {
    var context = getContext();
    var request = context.getRequest();
    var document = request.getBody();
    
    // Add computed fields
    if (document.price && document.quantity) {
        document.totalValue = document.price * document.quantity;
    }
    
    // Add metadata
    document.version = 1;
    document.status = 'active';
    
    // Generate unique identifier
    if (!document.uniqueId) {
        document.uniqueId = document.id + '-' + Date.now();
    }
    
    request.setBody(document);
}
```
## Post-Triggers

### Что такое Post-Triggers?

**Выполняются ПОСЛЕ операций create / update / delete**

- **Назначение** — обновление метаданных, логирование, вычисление производных данных
- **Момент выполнения** — после успешного завершения основной операции
- **Доступ** — могут читать результат выполненной операции
- **Транзакционность** — выполняются в рамках той же транзакции (в пределах одной партиции)
- **Откат** — основная операция уже зафиксирована

---

## Когда использовать Post-Triggers

- Автоматическое обновление агрегированных значений
- Логирование изменений
- Обновление связанных элементов внутри той же партиции
- Поддержка audit-полей (modifiedAt, modifiedBy)

---

## Ограничения

- Работают только в пределах одного partition key
- Должны быть явно указаны при выполнении операции
- Ограничены временем выполнения
- Не могут выполнять cross-partition операции


### Basic Post-Trigger Example

```javascript
function updateMetadata() {
    var context = getContext();
    var request = context.getRequest();
    var response = context.getResponse();
    var collection = context.getCollection();
    
    // Get the created/updated document
    var createdDocument = response.getBody();
    
    // Create metadata document
    var metadataDoc = {
        id: 'metadata-' + createdDocument.id,
        documentId: createdDocument.id,
        operation: request.getOperationType(),
        timestamp: new Date().toISOString()
    };
    
    // Create metadata document
    var accepted = collection.createDocument(
        collection.getSelfLink(),
        metadataDoc,
        function(err, docCreated) {
            if (err) throw new Error('Failed to create metadata: ' + err.message);
        }
    );
    
    if (!accepted) {
        throw new Error('Metadata creation not accepted');
    }
}
```

**Register and use from C#**:

```csharp
// 1. Register the post-trigger
var trigger = new TriggerProperties
{
    Id = "updateMetadata",
    Body = File.ReadAllText("updateMetadata.js"),
    TriggerOperation = TriggerOperation.Create,
    TriggerType = TriggerType.Post
};

await container.Scripts.CreateTriggerAsync(trigger);

// 2. Execute with operation
ItemRequestOptions requestOptions = new ItemRequestOptions
{
    PostTriggers = new List<string> { "updateMetadata" }
};

ItemResponse<Product> response = await container.CreateItemAsync(
    newProduct,
    new PartitionKey("electronics"),
    requestOptions
);
```

### Audit Log Post-Trigger

```javascript
function auditLog() {
    var context = getContext();
    var request = context.getRequest();
    var response = context.getResponse();
    var collection = context.getCollection();
    
    var document = response.getBody();
    
    // Create audit entry
    var auditEntry = {
        id: 'audit-' + document.id + '-' + Date.now(),
        documentId: document.id,
        operation: request.getOperationType(),
        timestamp: new Date().toISOString(),
        data: document
    };
    
    collection.createDocument(
        collection.getSelfLink(),
        auditEntry,
        function(err, created) {
            if (err) throw new Error('Audit log failed: ' + err.message);
        }
    );
}
```

### Count Update Post-Trigger

```javascript
function updateDocumentCount() {
    var context = getContext();
    var collection = context.getCollection();
    var response = context.getResponse();
    
    // Query for counter document
    var counterQuery = 'SELECT * FROM c WHERE c.id = "documentCounter"';
    
    var accepted = collection.queryDocuments(
        collection.getSelfLink(),
        counterQuery,
        function(err, documents) {
            if (err) throw new Error('Query failed: ' + err.message);
            
            if (documents.length > 0) {
                // Update existing counter
                var counter = documents[0];
                counter.count = (counter.count || 0) + 1;
                counter.lastUpdated = new Date().toISOString();
                
                collection.replaceDocument(
                    counter._self,
                    counter,
                    function(err, replaced) {
                        if (err) throw new Error('Update failed: ' + err.message);
                    }
                );
            } else {
                // Create new counter
                var newCounter = {
                    id: 'documentCounter',
                    count: 1,
                    lastUpdated: new Date().toISOString()
                };
                
                collection.createDocument(
                    collection.getSelfLink(),
                    newCounter,
                    function(err, created) {
                        if (err) throw new Error('Create failed: ' + err.message);
                    }
                );
            }
        }
    );
    
    if (!accepted) throw new Error('Query not accepted');
}
```

## User-Defined Functions (UDF)

### Что такое UDF?

**Пользовательские функции для использования в запросах**

- **Назначение** — расширение возможностей запросов собственной логикой
- **Когда выполняются** — во время выполнения запроса
- **Область применения** — только в выражениях запроса (SELECT, WHERE, ORDER BY)
- **Только чтение** — не могут изменять данные
- **Язык реализации** — JavaScript

---

## Когда использовать UDF

- Кастомные вычисления в SELECT
- Сложные фильтры в WHERE
- Логика форматирования или преобразования данных
- Повторно используемые вычисления в разных запросах

---

## Ограничения

- Работают только в контексте запроса
- Не могут выполнять операции записи
- Могут увеличивать потребление RU
- Ограничены временем выполнения

---

## Экзаменационный акцент (AZ-204)

- UDF используются **только в запросах**
- Не поддерживают операции изменения данных
- Пишутся на JavaScript
- Подходят для кастомной логики в SELECT и WHERE


### Basic UDF Example

```javascript
function tax(income) {
    if (income < 10000) {
        return income * 0.1;
    } else if (income < 50000) {
        return income * 0.2;
    } else {
        return income * 0.3;
    }
}
```

**Register and use from C#**:

```csharp
// 1. Register the UDF
var udf = new UserDefinedFunctionProperties
{
    Id = "tax",
    Body = @"
        function tax(income) {
            if (income < 10000) return income * 0.1;
            else if (income < 50000) return income * 0.2;
            else return income * 0.3;
        }"
};

await container.Scripts.CreateUserDefinedFunctionAsync(udf);

// 2. Use in query
var query = new QueryDefinition(
    "SELECT c.id, c.income, udf.tax(c.income) AS taxAmount FROM c"
);

FeedIterator<dynamic> iterator = container.GetItemQueryIterator<dynamic>(query);

while (iterator.HasMoreResults)
{
    FeedResponse<dynamic> results = await iterator.ReadNextAsync();
    
    foreach (var result in results)
    {
        Console.WriteLine($"ID: {result.id}, Income: {result.income}, Tax: {result.taxAmount}");
    }
}
```

### String Manipulation UDF

```javascript
function formatName(firstName, lastName) {
    if (!firstName || !lastName) {
        return '';
    }
    return lastName.toUpperCase() + ', ' + firstName;
}
```

**Use in query**:

```csharp
var query = new QueryDefinition(
    "SELECT udf.formatName(c.firstName, c.lastName) AS fullName FROM c"
);
```

### Calculation UDF

```javascript
function calculateDiscount(price, discountPercent, membershipLevel) {
    var baseDiscount = price * (discountPercent / 100);
    
    // Additional discount for membership
    var membershipMultiplier = 1.0;
    if (membershipLevel === 'gold') {
        membershipMultiplier = 1.2;
    } else if (membershipLevel === 'platinum') {
        membershipMultiplier = 1.5;
    }
    
    return baseDiscount * membershipMultiplier;
}
```

**Use in query**:

```sql
SELECT 
    c.id, 
    c.price,
    udf.calculateDiscount(c.price, 10, c.membershipLevel) AS discount,
    c.price - udf.calculateDiscount(c.price, 10, c.membershipLevel) AS finalPrice
FROM c
WHERE c.category = 'electronics'
```

### Date/Time UDF

```javascript
function daysSince(dateString) {
    if (!dateString) return null;
    
    var date = new Date(dateString);
    var now = new Date();
    var diffMs = now - date;
    var diffDays = Math.floor(diffMs / (1000 * 60 * 60 * 24));
    
    return diffDays;
}
```

**Use in query**:

```csharp
var query = new QueryDefinition(@"
    SELECT 
        c.id, 
        c.createdAt,
        udf.daysSince(c.createdAt) AS daysOld
    FROM c
    WHERE udf.daysSince(c.createdAt) < 30
    ORDER BY udf.daysSince(c.createdAt) DESC
");
```

### Complex UDF with Multiple Parameters

```javascript
function calculateShipping(weight, distance, expedited) {
    var baseRate = 5.0;
    var weightRate = weight * 0.5;
    var distanceRate = distance * 0.1;
    var total = baseRate + weightRate + distanceRate;
    
    if (expedited) {
        total *= 1.5;
    }
    
    return Math.round(total * 100) / 100;  // Round to 2 decimals
}
```

**Use in query**:

```sql
SELECT 
    c.id,
    c.weight,
    c.distance,
    udf.calculateShipping(c.weight, c.distance, true) AS expeditedShipping,
    udf.calculateShipping(c.weight, c.distance, false) AS standardShipping
FROM c
```

### Conditional UDF

```javascript
function getStatus(stock, reorderLevel) {
    if (stock === 0) {
        return 'OUT_OF_STOCK';
    } else if (stock <= reorderLevel) {
        return 'LOW_STOCK';
    } else if (stock <= reorderLevel * 2) {
        return 'NORMAL';
    } else {
        return 'OVERSTOCKED';
    }
}
```

## Registering Triggers and UDFs

### Register Pre-Trigger

```csharp
var preTrigger = new TriggerProperties
{
    Id = "validateProduct",
    Body = File.ReadAllText("validateProduct.js"),
    TriggerOperation = TriggerOperation.Create,  // Create, Update, Delete, All
    TriggerType = TriggerType.Pre
};

TriggerResponse response = await container.Scripts.CreateTriggerAsync(preTrigger);
Console.WriteLine($"Created trigger: {response.Resource.Id}");
```

### Register Post-Trigger

```csharp
var postTrigger = new TriggerProperties
{
    Id = "auditLog",
    Body = File.ReadAllText("auditLog.js"),
    TriggerOperation = TriggerOperation.All,  // Trigger on all operations
    TriggerType = TriggerType.Post
};

await container.Scripts.CreateTriggerAsync(postTrigger);
```

### Register UDF

```csharp
var udf = new UserDefinedFunctionProperties
{
    Id = "calculateTax",
    Body = @"
        function calculateTax(amount, rate) {
            return amount * rate;
        }"
};

await container.Scripts.CreateUserDefinedFunctionAsync(udf);
```

### List All Triggers

```csharp
FeedIterator<TriggerProperties> iterator = 
    container.Scripts.GetTriggerQueryIterator<TriggerProperties>();

while (iterator.HasMoreResults)
{
    FeedResponse<TriggerProperties> triggers = await iterator.ReadNextAsync();
    
    foreach (var trigger in triggers)
    {
        Console.WriteLine($"Trigger: {trigger.Id}");
        Console.WriteLine($"  Type: {trigger.TriggerType}");
        Console.WriteLine($"  Operation: {trigger.TriggerOperation}");
    }
}
```

### List All UDFs

```csharp
FeedIterator<UserDefinedFunctionProperties> iterator = 
    container.Scripts.GetUserDefinedFunctionQueryIterator<UserDefinedFunctionProperties>();

while (iterator.HasMoreResults)
{
    FeedResponse<UserDefinedFunctionProperties> udfs = await iterator.ReadNextAsync();
    
    foreach (var udf in udfs)
    {
        Console.WriteLine($"UDF: {udf.Id}");
    }
}
```

### Update Trigger

```csharp
// Read existing trigger
var existingTrigger = await container.Scripts.ReadTriggerAsync("validateProduct");

// Update body
existingTrigger.Resource.Body = File.ReadAllText("validateProductV2.js");

// Replace
await container.Scripts.ReplaceTriggerAsync(existingTrigger.Resource);
```

### Delete Trigger or UDF

```csharp
// Delete trigger
await container.Scripts.DeleteTriggerAsync("validateProduct");

// Delete UDF
await container.Scripts.DeleteUserDefinedFunctionAsync("calculateTax");
```

## Using Triggers in Operations

### With Create Operation

```csharp
ItemRequestOptions options = new ItemRequestOptions
{
    PreTriggers = new List<string> { "validateDocument", "addTimestamp" },
    PostTriggers = new List<string> { "auditLog" }
};

ItemResponse<Product> response = await container.CreateItemAsync(
    newProduct,
    new PartitionKey("electronics"),
    options
);
```

### With Update Operation

```csharp
ItemRequestOptions options = new ItemRequestOptions
{
    PreTriggers = new List<string> { "updateTimestamp" },
    PostTriggers = new List<string> { "logChange" }
};

ItemResponse<Product> response = await container.ReplaceItemAsync(
    updatedProduct,
    updatedProduct.id,
    new PartitionKey("electronics"),
    options
);
```

### With Delete Operation

```csharp
ItemRequestOptions options = new ItemRequestOptions
{
    PostTriggers = new List<string> { "archiveDeleted" }
};

await container.DeleteItemAsync<Product>(
    "product-1",
    new PartitionKey("electronics"),
    options
);
```

### Multiple Triggers

```csharp
// Execute multiple triggers in order
ItemRequestOptions options = new ItemRequestOptions
{
    PreTriggers = new List<string> 
    { 
        "validateSchema",      // 1. Validate first
        "addTimestamp",        // 2. Then add timestamp
        "enrichData"           // 3. Finally enrich
    },
    PostTriggers = new List<string> 
    { 
        "updateMetadata",      // 1. Update metadata
        "auditLog",            // 2. Then log
        "notifySubscribers"    // 3. Finally notify
    }
};
```

## Trigger Operations Enum

```csharp
public enum TriggerOperation
{
    All,      // Trigger on all operations
    Create,   // Trigger only on create
    Update,   // Trigger only on update
    Delete,   // Trigger only on delete
    Replace   // Trigger only on replace
}

public enum TriggerType
{
    Pre,   // Execute before operation
    Post   // Execute after operation
}
```

## Best Practices

### 1. Keep Triggers Lightweight

```javascript
// ✅ Good: Simple, focused logic
function addTimestamp() {
    var context = getContext();
    var request = context.getRequest();
    var document = request.getBody();
    document.timestamp = new Date().toISOString();
    request.setBody(document);
}

// ❌ Bad: Complex, heavy logic
function complexTrigger() {
    // Multiple queries, complex calculations
    // Risk of timeout and performance issues
}
```

### 2. Use Pre-Triggers for Validation

```javascript
// ✅ Good: Validate input
function validateInput() {
    var document = getContext().getRequest().getBody();
    if (!document.requiredField) {
        throw new Error("Missing required field");
    }
}
```

### 3. Use Post-Triggers for Metadata

```javascript
// ✅ Good: Update related data after operation succeeds
function updateMetadata() {
    // Create audit log, update counters, etc.
}
```

### 4. Error Handling in Triggers

```javascript
function safeTrigger() {
    try {
        var context = getContext();
        var request = context.getRequest();
        var document = request.getBody();
        
        // Validation logic
        if (!document.id) {
            throw new Error("Document must have an id");
        }
        
        request.setBody(document);
    } catch (error) {
        // Log error and throw
        throw new Error('Trigger failed: ' + error.message);
    }
}
```

### 5. UDF Naming Conventions

```javascript
// ✅ Good: Descriptive names
function calculateTax(income) { }
function formatCurrency(amount) { }
function daysBetween(date1, date2) { }

// ❌ Bad: Generic names
function calc(x) { }
function process(data) { }
```

### 6. UDF Performance

```javascript
// ✅ Good: Simple calculations
function calculateTotal(price, quantity) {
    return price * quantity;
}

// ❌ Bad: Complex operations
function complexCalculation(data) {
    // Multiple loops, heavy processing
    // Slows down query execution
}
```

## Common Patterns

### Pattern 1: Timestamp Management

```javascript
// Pre-trigger for automatic timestamps
function manageTimestamps() {
    var context = getContext();
    var request = context.getRequest();
    var document = request.getBody();
    var operation = request.getOperationType();
    
    var now = new Date().toISOString();
    
    if (operation === "Create") {
        document.createdAt = now;
        document.updatedAt = now;
    } else if (operation === "Replace") {
        document.updatedAt = now;
    }
    
    request.setBody(document);
}
```

### Pattern 2: Versioning

```javascript
// Pre-trigger for version management
function incrementVersion() {
    var context = getContext();
    var request = context.getRequest();
    var document = request.getBody();
    
    if (request.getOperationType() === "Replace") {
        document.version = (document.version || 0) + 1;
    } else {
        document.version = 1;
    }
    
    request.setBody(document);
}
```

### Pattern 3: Audit Trail

```javascript
// Post-trigger for audit logging
function createAuditTrail() {
    var context = getContext();
    var request = context.getRequest();
    var response = context.getResponse();
    var collection = context.getCollection();
    
    var auditDoc = {
        id: 'audit-' + response.getBody().id + '-' + Date.now(),
        documentId: response.getBody().id,
        operation: request.getOperationType(),
        timestamp: new Date().toISOString(),
        snapshot: response.getBody()
    };
    
    collection.createDocument(
        collection.getSelfLink(),
        auditDoc,
        function(err) {
            if (err) throw new Error('Audit failed: ' + err.message);
        }
    );
}
```

# Critical Notes — Triggers и UDF

- 💡 **Pre-triggers** — выполняются до операции, могут изменять входные данные
- 🎯 **Post-triggers** — выполняются после операции, могут создавать связанные документы
- ✅ **UDF** — пользовательские функции для запросов (SELECT, WHERE, ORDER BY)
- ⚠️ **Опциональные** — триггеры должны быть явно указаны в каждом запросе
- 🔄 **JavaScript** — всё реализуется на JavaScript
- 📊 **Валидация** — pre-triggers подходят для проверки входных данных
- 💡 **Метаданные** — post-triggers подходят для audit-логов, счётчиков
- ✅ **Только чтение** — UDF не могут изменять данные
- ⚠️ **Транзакционность** — триггеры выполняются в рамках той же транзакции
- 🔒 **Обработка ошибок** — выброс ошибки в триггере приводит к откату операции
- 🎯 **Регистрация** — необходимо зарегистрировать перед использованием
- 💡 **Request options** — триггеры указываются в ItemRequestOptions
- ⚠️ **Несколько триггеров** — можно указать несколько pre/post триггеров
- ✅ **TriggerOperation** — All, Create, Update, Delete, Replace
- 🔄 **Context objects** — getContext(), getRequest(), getResponse(), getCollection()

---

# Exam Tips (AZ-204)

- Pre-triggers — выполняются до операций (Create, Update, Delete)
- Post-triggers — выполняются после успешного завершения операций
- UDF — используются только в запросах
- Триггеры опциональны — нужно явно указать в ItemRequestOptions
- Язык — JavaScript (как и Stored Procedures)

---

## Работа с объектами

- Request object — getRequest().getBody() для чтения/изменения входных данных
- Response object — getResponse().getBody() для получения результата

---

## Назначение

- Валидация — pre-triggers
- Метаданные, audit, счётчики — post-triggers

---

## Регистрация

- Триггеры — Scripts.CreateTriggerAsync()
- UDF — Scripts.CreateUserDefinedFunctionAsync()

---

## Перечисления

- TriggerOperation — All, Create, Update, Delete, Replace
- TriggerType — Pre, Post

---

## Использование UDF в запросе

- Используются в SELECT, WHERE, ORDER BY
- Только для чтения
- Не могут изменять документы

---

## Дополнительно

- PreTriggers — задаются в ItemRequestOptions
- PostTriggers — задаются в ItemRequestOptions
- Можно указать массив идентификаторов триггеров
- Выполняются в рамках одной транзакции
- Поддерживаются только внутри одного partition key
- Триггеры должны быть лёгкими по логике (чтобы избежать timeout)


[Learn More](https://learn.microsoft.com/en-us/training/modules/work-with-cosmos-db/5-cosmos-db-triggers-user-defined-functions)
