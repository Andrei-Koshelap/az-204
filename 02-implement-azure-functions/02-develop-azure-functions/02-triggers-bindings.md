# Triggers and Bindings (Триггеры и биндинги)

## Key Concepts (Ключевые понятия)

- **Trigger** — событие, которое запускает выполнение функции  
  (ровно **один trigger на функцию**)
- **Binding** — декларативное подключение к источнику или получателю данных
- **Input binding** — чтение данных в функцию
- **Output binding** — запись данных из функции
- **Direction** — `in`, `out` или `inout`

---

# Triggers (Триггеры)

## Definition (Определение)

Trigger — это **событие, которое вызывает выполнение функции**.

📌 У каждой функции должен быть **ровно один trigger**.

---

## Trigger Properties (Свойства триггера)

- **Type** — тип события (`http`, `timer`, `queue` и др.)
- **Data** — данные события (передаются как параметр функции)
- **Direction** — всегда `in`

---

## Common Triggers (Часто используемые триггеры)

| Trigger | Event | Use Case |
|----------|--------|------------|
| **HTTP** | HTTP-запрос | REST API, webhooks |
| **Timer** | Расписание (cron) | Периодические задачи |
| **Queue** | Сообщение в очереди | Асинхронная обработка |
| **Blob** | Добавление/изменение файла | Обработка файлов |
| **Event Hub** | Поток событий | IoT, телеметрия |
| **Event Grid** | Событие Azure | Реакция на изменения ресурсов |
| **Service Bus** | Корпоративное сообщение | Надёжный messaging |
| **Cosmos DB** | Изменение документа | Синхронизация данных |

---

# Bindings (Биндинги)

## Definition (Определение)

Binding — это **декларативный способ** подключения к внешним сервисам.

Не нужно писать код подключения — runtime делает это автоматически.

---

## Binding Types (Типы биндингов)

1️⃣ **Input binding** — получение данных  
2️⃣ **Output binding** — отправка данных  
3️⃣ **Bidirectional (inout)** — чтение и запись

---

## Benefits (Преимущества)

✅ Нет хардкодинга строк подключения  
✅ Упрощённый код  
✅ Легче тестировать  
✅ Повторное использование конфигурации

---

## Binding Properties (Свойства биндинга)

- **Type** — источник данных (`blob`, `table`, `queue` и т.д.)
- **Direction** — `in`, `out`, `inout`
- **Name** — имя параметра в коде
- **Connection** — имя app setting с connection string

---

## Важно для AZ-204

- Один trigger на функцию
- Несколько bindings допустимы
- Trigger всегда `direction = in`
- Bindings конфигурируются декларативно
- Connection strings хранятся в Application Settings

## Configuration Methods

### Method 1: function.json (JavaScript/Python/PowerShell)
```json
{
  "disabled": false,
  "bindings": [
    {
      "type": "queueTrigger",
      "direction": "in",
      "name": "myQueueItem",
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting"
    },
    {
      "type": "table",
      "direction": "out",
      "name": "tableBinding",
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

**Properties**:
- `type` - Trigger/binding type
- `direction` - Data flow direction
- `name` - Function parameter name
- `queueName` / `tableName` - Resource name
- `connection` - App setting name (NOT connection string itself)

### Method 2: Attributes (C#)
```csharp
[FunctionName("QueueTriggerTableOutput")]
[return: Table("outTable", Connection = "MY_TABLE_STORAGE_ACCT_APP_SETTING")]
public static Person Run(
    [QueueTrigger("myqueue-items", Connection = "MY_STORAGE_ACCT_APP_SETTING")] JObject order,
    ILogger log)
{
    return new Person() {
        PartitionKey = "Orders",
        RowKey = Guid.NewGuid().ToString(),
        Name = order["Name"].ToString(),
        MobileNumber = order["MobileNumber"].ToString()
    };
}
```

**Attributes provide**:
- Trigger definition (`QueueTrigger`)
- Input/output bindings (`Table`)
- Connection settings
- Resource names

### Method 3: Annotations (Java)
```java
@FunctionName("QueueTriggerTableOutput")
@TableOutput(name = "tableBinding", tableName = "Person", connection = "MyStorageConnection")
public Person run(
    @QueueTrigger(name = "myQueueItem", queueName = "myqueue-items", connection = "MyStorageConnection") String order,
    final ExecutionContext context) {
    
    Person person = new Person();
    person.setPartitionKey("Orders");
    person.setRowKey(UUID.randomUUID().toString());
    return person;
}
```
# Binding Direction (Направление биндингов)

## Trigger

- **Direction**: всегда `in`
- **Count**: ровно 1 на функцию
- **Purpose**: запуск выполнения функции

---

## Input Binding

- **Direction**: `in`
- **Count**: 0 или больше
- **Purpose**: получение данных в функцию

---

## Output Binding

- **Direction**: `out`
- **Count**: 0 или больше
- **Purpose**: запись данных из функции

---

## Bidirectional Binding

- **Direction**: `inout`
- **Count**: 0 или больше
- **Purpose**: чтение и запись одного и того же ресурса
- **Portal**: требуется Advanced editor

> 💡 Используется, например, для обновления документа в базе.

---

## Direction Summary (Сводка)

| Binding | Direction | Count | Example |
|----------|------------|--------|----------|
| **Trigger** | `in` | 1 (обязателен) | Сообщение в очереди |
| **Input** | `in` | 0+ | Чтение blob, запрос к таблице |
| **Output** | `out` | 0+ | Запись blob, вставка строки |
| **Bidirectional** | `inout` | 0+ | Обновление документа |

---

# Complete Example: Queue → Table

## Scenario (Сценарий)

Новое сообщение в очереди → записать строку в таблицу.

### Логика:

1. Сообщение поступает в **Storage Queue**
2. Срабатывает **Queue trigger**
3. Функция получает сообщение
4. Через **Output binding** добавляет строку в Azure Table Storage

---

## Пример (JavaScript / function.json)

```json
{
  "bindings": [
    {
      "name": "myQueueItem",
      "type": "queueTrigger",
      "direction": "in",
      "queueName": "orders",
      "connection": "AzureWebJobsStorage"
    },
    {
      "name": "outputTable",
      "type": "table",
      "direction": "out",
      "tableName": "Orders",
      "connection": "AzureWebJobsStorage"
    }
  ]
}
```

### function.json (JavaScript)
```json
{
  "disabled": false,
  "bindings": [
    {
      "type": "queueTrigger",
      "direction": "in",
      "name": "myQueueItem",
      "queueName": "myqueue-items",
      "connection": "MyStorageConnectionAppSetting"
    },
    {
      "type": "table",
      "direction": "out",
      "name": "tableBinding",
      "tableName": "Person",
      "connection": "MyStorageConnectionAppSetting"
    }
  ]
}
```

### index.js
```javascript
module.exports = async function (context, myQueueItem) {
    context.log('Processing queue message:', myQueueItem);
    
    // Output to table via binding
    context.bindings.tableBinding = {
        PartitionKey: "Orders",
        RowKey: context.bindingData.id,
        Name: myQueueItem.name,
        MobileNumber: myQueueItem.mobile
    };
};
```


Важно для AZ-204
Trigger всегда один
Bindings могут быть множественными
Output binding автоматически записывает данные
Connection указывается через имя App Setting
Код не содержит строк подключения

### C# Equivalent
```csharp
[FunctionName("QueueTriggerTableOutput")]
[return: Table("Person", Connection = "MyStorageConnectionAppSetting")]
public static Person Run(
    [QueueTrigger("myqueue-items", Connection = "MyStorageConnectionAppSetting")] Order order,
    ILogger log)
{
    log.LogInformation($"Processing order: {order.Name}");
    
    return new Person
    {
        PartitionKey = "Orders",
        RowKey = Guid.NewGuid().ToString(),
        Name = order.Name,
        MobileNumber = order.MobileNumber
    };
}
```

## Data Types

### JavaScript/Python (Dynamically Typed)
Use `dataType` property in function.json:

```json
{
    "type": "httpTrigger",
    "name": "req",
    "direction": "in",
    "dataType": "binary"
}
```

**Options**:
- `binary` - Byte array
- `stream` - Stream data
- `string` - String (default)

### C#/.NET (Statically Typed)
Type inferred from parameter:

```csharp
// String
[HttpTrigger] string req

// Binary
[HttpTrigger] byte[] req

// Stream
[HttpTrigger] Stream req

// Object (JSON deserialization)
[HttpTrigger] MyCustomType req
```

## Multiple Bindings Example

### Scenario
HTTP trigger → Read from Blob → Write to Queue and Table

### function.json
```json
{
  "bindings": [
    {
      "type": "httpTrigger",
      "direction": "in",
      "name": "req",
      "methods": ["post"]
    },
    {
      "type": "blob",
      "direction": "in",
      "name": "inputBlob",
      "path": "input/{filename}",
      "connection": "AzureWebJobsStorage"
    },
    {
      "type": "queue",
      "direction": "out",
      "name": "outputQueue",
      "queueName": "processed-items",
      "connection": "AzureWebJobsStorage"
    },
    {
      "type": "table",
      "direction": "out",
      "name": "outputTable",
      "tableName": "ProcessedFiles",
      "connection": "AzureWebJobsStorage"
    },
    {
      "type": "http",
      "direction": "out",
      "name": "res"
    }
  ]
}
```

### index.js
```javascript
module.exports = async function (context, req, inputBlob) {
    context.log('Processing file:', req.query.filename);
    
    // Read from blob (input binding)
    const fileContent = inputBlob.toString();
    
    // Write to queue (output binding)
    context.bindings.outputQueue = {
        filename: req.query.filename,
        processed: true
    };
    
    // Write to table (output binding)
    context.bindings.outputTable = {
        PartitionKey: "Files",
        RowKey: new Date().toISOString(),
        Filename: req.query.filename,
        Size: inputBlob.length
    };
    
    // HTTP response (output binding)
    context.res = {
        status: 200,
        body: "File processed successfully"
    };
};
```

## Common Binding Patterns

### Pattern 1: Trigger → Process → Output
```
Queue Trigger → Function Logic → Blob Output
Timer Trigger → Function Logic → Table Output
HTTP Trigger → Function Logic → Queue Output
```

### Pattern 2: Trigger → Read → Process → Write
```
HTTP Trigger → Blob Input → Process → Table Output
Queue Trigger → Cosmos Input → Process → Queue Output
```

### Pattern 3: Fan-Out
```
Single Trigger → Multiple Outputs
Queue message → Write to Blob + Table + Queue
```

### Pattern 4: Chain
```
HTTP → Queue (output)
Queue (trigger) → Process → Blob (output)
Blob (trigger) → Process → Table (output)
```

## Connection Configuration

### App Settings (Recommended)
```json
// local.settings.json
{
  "Values": {
    "MyStorageConnectionAppSetting": "DefaultEndpointsProtocol=https;AccountName=mystorage;..."
  }
}
```

### Reference in Binding
```json
{
  "type": "queueTrigger",
  "connection": "MyStorageConnectionAppSetting"
}
```

⚠️ **Security**: Never hardcode connection strings in code

### Azure App Settings
```bash
# Set connection string
az functionapp config appsettings set \
  --name <function-app-name> \
  --resource-group <rg-name> \
  --settings "MyStorageConnectionAppSetting=DefaultEndpointsProtocol=https;..."
```

## Portal Binding Configuration

### For JavaScript/Python/PowerShell:
1. Navigate to function in portal
2. Click "Integration" tab
3. Click "+ Add input" or "+ Add output"
4. Select binding type
5. Configure properties
6. Save

### For C#:
❌ Cannot add bindings in portal - Use attributes in code

## Testing Bindings Locally

### Option 1: Live Azure Services
```json
// local.settings.json - Point to live Azure
{
  "Values": {
    "AzureWebJobsStorage": "DefaultEndpointsProtocol=https;AccountName=..."
  }
}
```

⚠️ **Caution**: Uses real data, can incur costs

### Option 2: Azurite Emulator
```bash
# Install and start Azurite
npm install -g azurite
azurite --silent --location c:\azurite

# In local.settings.json
"AzureWebJobsStorage": "UseDevelopmentStorage=true"
```

✅ **Recommended**: Safe, no costs, offline capable

### Option 3: Manual Admin Endpoint
```bash
# Trigger non-HTTP functions manually
curl -X POST http://localhost:7071/admin/functions/MyQueueFunction \
  -H "Content-Type: application/json" \
  -d '{"input": "test message"}'
```

## Binding Expressions

### Automatic Values
Use binding expressions in `path` or other properties:

```json
{
  "type": "blob",
  "direction": "out",
  "name": "outputBlob",
  "path": "output/{rand-guid}.txt",
  "connection": "AzureWebJobsStorage"
}
```

**Available expressions**:
- `{rand-guid}` - Random GUID
- `{datetime}` - Current timestamp
- `{name}` - From trigger data
- `{queueTrigger}` - Queue message content

### Example
```json
{
  "bindings": [
    {
      "type": "queueTrigger",
      "name": "order",
      "queueName": "orders"
    },
    {
      "type": "blob",
      "direction": "out",
      "name": "receipt",
      "path": "receipts/{id}-{datetime}.json"
    }
  ]
}
```
## Critical Notes (Критически важные моменты)

- 💡 **Один trigger** — строго один на функцию (обязателен)
- ⚠️ **Несколько bindings** — можно 0 или больше input/output биндингов
- 🎯 **Direction**:
    - Trigger всегда `in`
    - Bindings — `in`, `out`, `inout`
- 📊 **Декларативная модель** — конфигурация, а не код
- ✅ **Connection** — указывает на имя App Setting, а не на строку подключения
- 🔄 **C# использует атрибуты**, другие языки — `function.json`
- ⏱️ **Azurite** — эмулятор Azure Storage для локального тестирования
- 🔒 Никогда не хардкодьте секреты — используйте App Settings

---

## Exam Tips (Советы для экзамена)

- **Trigger** = событие запуска функции (РОВНО один)
- **Binding** = подключение к источнику/назначению данных (0 или больше)
- Direction:
    - Trigger → всегда `in`
    - Bindings → `in` / `out` / `inout`

### Языковые различия

- **C# (compiled)** → атрибуты (`[QueueTrigger]`, `[BlobOutput]` и т.д.)
- **JavaScript / Python / PowerShell** → `function.json`
- **Java** → аннотации (`@QueueTrigger`, `@TableOutput`, и др.)

---

### Важные детали

- Свойство `connection` ссылается на **имя настройки**, а не сам connection string
- `dataType` используется в динамически типизированных языках (`binary`, `stream`, `string`)
- В C# тип определяется типом параметра функции
- В портале вкладка **Integration** доступна только для `function.json` языков
- Для C# bindings настраиваются только через атрибуты (не через портал)
- Azurite позволяет тестировать Storage bindings локально
- Admin endpoint для ручного запуска:
       http://localhost:7071/admin/functions/{name}


---

### Binding Expressions (Выражения биндингов)

Можно использовать специальные выражения:

- `{rand-guid}` — случайный GUID
- `{datetime}` — текущая дата/время
- `{propertyName}` — значение свойства из триггера

> 💡 Часто используется для динамического именования blob-файлов и строк таблиц.

[Learn More](https://learn.microsoft.com/en-us/training/modules/develop-azure-functions/3-create-triggers-bindings)
