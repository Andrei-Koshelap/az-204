# Event Hubs Capture

## Что такое Event Hubs Capture?

**Event Hubs Capture** — встроенная функция, которая автоматически сохраняет потоковые данные из Event Hubs в:

- **Azure Blob Storage**
- **Azure Data Lake Storage**

Данные сохраняются в формате **Apache Avro**.

Capture работает без написания кода — достаточно включить его в настройках.

---

# Ключевые преимущества

- **Автоматическая работа**  
  Включается через Azure Portal или CLI

- **Масштабируемость**  
  Поддерживает миллионы событий в секунду

- **Экономичность**  
  Использует внутреннее хранилище Event Hubs  
  (не расходует TU egress)

- **Триггер по времени или размеру**  
  Захват по таймеру или при достижении объёма данных

- **Надёжность**  
  Подходит для долгосрочного хранения

- **Гибкость**  
  Файлы можно обрабатывать любыми Avro-совместимыми инструментами

---

# Сценарии использования

| Сценарий | Описание | Пример |
|------------|-----------|---------|
| **Data Lake** | Построение аналитического хранилища | Хранение IoT-телеметрии для ML |
| **Compliance** | Долгосрочное хранение | Финансовые транзакции (7 лет) |
| **Batch Processing** | Периодическая аналитика | Ежедневная агрегация через Spark |
| **Backup** | Архивация потоков | Disaster recovery |
| **Cold Storage** | Дешёвое долгосрочное хранение | Перенос в Cool/Archive tier |
| **Hybrid Processing** | Real-time + batch | Мгновенные алерты + ежедневные отчёты |

---

## Архитектурный акцент

Event Hubs Capture используется в:

- Lambda architecture
- Streaming + Data Lake pipeline
- Архивировании потоковых данных
- Replay-сценариях

Это способ превратить поток в файловое хранилище.

---

## Важно для AZ-204

Нужно помнить:

- Capture сохраняет данные в Avro
- Работает с Blob Storage и Data Lake
- Не требует кода
- Не расходует egress TU
- Может запускаться по времени или размеру

Если в вопросе говорится об автоматической архивации потоков — правильный ответ — Event Hubs Capture.

## How Event Hubs Capture Works

### Capture Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      EVENT PRODUCERS                         │
│   IoT Devices, Applications, Services, Kafka Clients        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ▼
        ┌────────────────────────────────────┐
        │      EVENT HUBS NAMESPACE          │
        │  ┌──────────────────────────────┐  │
        │  │   Event Hub: "telemetry"     │  │
        │  │  ┌────┬────┬────┬────┐      │  │
        │  │  │ P0 │ P1 │ P2 │ P3 │      │  │
        │  │  └────┴────┴────┴────┘      │  │
        │  │                              │  │
        │  │  Capture Enabled:            │  │
        │  │  • Time: 5 minutes           │  │
        │  │  • Size: 100 MB              │  │
        │  └──────────────────────────────┘  │
        └──────────────┬─────────────────────┘
                       │
                       │ (Automatic Capture)
                       │
                       ▼
        ┌────────────────────────────────────┐
        │  AZURE BLOB STORAGE / DATA LAKE    │
        │                                     │
        │  Container: mycaptures              │
        │  ├── telemetry/                     │
        │  │   ├── 0/                         │
        │  │   │   ├── 2024/01/15/10/        │
        │  │   │   │   ├── 00.avro           │
        │  │   │   │   ├── 05.avro           │
        │  │   │   │   └── 10.avro           │
        │  │   ├── 1/                         │
        │  │   │   └── 2024/01/15/10/        │
        │  │   ├── 2/                         │
        │  │   └── 3/                         │
        └────────────────────────────────────┘
                       │
                       ▼
        ┌────────────────────────────────────┐
        │     BATCH PROCESSING               │
        │  • Azure Databricks                │
        │  • Azure Synapse Analytics         │
        │  • Azure Data Factory              │
        │  • HDInsight (Spark, Hive)         │
        └────────────────────────────────────┘
```

# Процесс работы Event Hubs Capture

## Flow Capture

1️⃣ **События поступают**  
Producers отправляют события в Event Hubs.

2️⃣ **Внутреннее хранение**  
События сохраняются во внутреннем time-retention store.

3️⃣ **Триггер Capture**  
Срабатывает при:
- достижении временного окна  
  или
- достижении порога по размеру  
  (что наступит раньше)

4️⃣ **Создание файла**  
События записываются в файл формата Avro.

5️⃣ **Загрузка**  
Файл отправляется в:
- Azure Blob Storage  
  или
- Azure Data Lake Storage

6️⃣ **Пустые файлы**  
Если в течение временного окна не было событий, создаётся пустой файл.

---

## Важные замечания

- ⚠️ Capture **не расходует Throughput Units (egress)**
- ✅ Работает напрямую из внутреннего хранилища
- ✅ Не влияет на real-time consumers
- ✅ Выполняется **независимо для каждой partition**

---

# Формат Apache Avro

**Apache Avro** — компактный бинарный формат сериализации данных со встроенной схемой.

---

## Почему используется Avro?

| Свойство | Преимущество |
|------------|--------------|
| **Компактность** | Бинарный формат меньше JSON/XML |
| **Производительность** | Быстрая сериализация и десериализация |
| **Эволюция схемы** | Можно добавлять/удалять поля без нарушения совместимости |
| **Self-Describing** | Схема встроена в файл |
| **Кросс-платформенность** | Поддержка C#, Java, Python и др. |
| **Splittable** | Подходит для MapReduce и Spark |

---

## Архитектурный смысл

Avro + Capture позволяет:

- Строить Data Lake
- Поддерживать replay сценарии
- Реализовать batch analytics
- Обеспечить compliance-хранение

---

## Важно для AZ-204

Нужно помнить:

- Capture создаёт Avro-файлы
- Работает по времени или размеру
- Не использует egress TU
- Выполняется по partition
- Создаёт пустые файлы при отсутствии событий

Если в вопросе речь об автоматическом архивировании потоков в Data Lake — это Event Hubs Capture.

### Avro File Structure

```
┌──────────────────────────────────┐
│         Avro File Header         │
│  • Magic: "Obj1"                 │
│  • Schema (JSON)                 │
│  • Sync marker (16 bytes)        │
├──────────────────────────────────┤
│         Data Block 1             │
│  • Count: Number of events       │
│  • Size: Block size in bytes     │
│  • Events (binary)               │
│  • Sync marker                   │
├──────────────────────────────────┤
│         Data Block 2             │
│  • Events...                     │
│  • Sync marker                   │
├──────────────────────────────────┤
│            ...                   │
└──────────────────────────────────┘
```

### Event Hubs Avro Schema

**Schema for captured events:**

```json
{
  "type": "record",
  "name": "EventData",
  "namespace": "Microsoft.ServiceBus.Messaging",
  "fields": [
    {
      "name": "SequenceNumber",
      "type": "long"
    },
    {
      "name": "Offset",
      "type": "string"
    },
    {
      "name": "EnqueuedTimeUtc",
      "type": "string"
    },
    {
      "name": "SystemProperties",
      "type": {
        "type": "map",
        "values": ["long", "double", "string", "bytes"]
      }
    },
    {
      "name": "Properties",
      "type": {
        "type": "map",
        "values": ["long", "double", "string", "bytes", "null"]
      }
    },
    {
      "name": "Body",
      "type": ["null", "bytes"]
    }
  ]
}
```

# Описание полей Avro-файла Capture

| Поле | Тип | Описание |
|------|------|-----------|
| `SequenceNumber` | long | Уникальный порядковый номер события внутри partition |
| `Offset` | string | Смещение события в журнале partition |
| `EnqueuedTimeUtc` | string | Время добавления события (UTC) |
| `SystemProperties` | map | Системные свойства, назначенные Event Hubs |
| `Properties` | map | Пользовательские свойства приложения |
| `Body` | bytes | Тело события (payload) |

---

## Что важно понимать

- `SequenceNumber` и `Offset` используются для отслеживания позиции
- `EnqueuedTimeUtc` важен для аналитики по времени
- `SystemProperties` содержит служебную информацию
- `Properties` — custom metadata
- `Body` — фактические данные события

---

# Настройка Capture

## Windowing (Окна захвата)

Capture использует принцип **first wins policy**:

Файл создаётся при наступлении первого из условий:

- **Time Window**  
  Захват через X минут  
  (от 1 до 15 минут)

- **Size Window**  
  Захват при достижении X MB  
  (от 10 до 500 MB)

---

## Архитектурный смысл

- Маленькое time window → больше файлов
- Большое size window → меньше файлов, выше задержка
- Выбор зависит от аналитических требований

---

## Важно для AZ-204

Нужно помнить:

- Capture работает по принципу «что наступит раньше»
- Time window: 1–15 минут
- Size window: 10–500 MB
- Capture выполняется отдельно для каждой partition
- Формат хранения — Avro

Если в вопросе говорится о периодическом создании файлов из потока — это Capture + windowing.

**Example Scenarios:**

```
Scenario 1: Time=5 min, Size=100 MB
- High traffic: File created every ~2 minutes (100 MB reached)
- Low traffic: File created every 5 minutes (time reached)

Scenario 2: Time=15 min, Size=10 MB
- Steady traffic: File created every ~1 minute (10 MB reached)
- Very low traffic: File created every 15 minutes

Scenario 3: No events
- Empty file created every time window (maintains predictable cadence)
```

### Naming Convention

**File naming pattern:**
```
{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}
```

**Example file paths:**

```
mystorage.blob.core.windows.net/captures/
├── mynamespace/
│   ├── telemetry/
│   │   ├── 0/
│   │   │   ├── 2024/
│   │   │   │   ├── 01/
│   │   │   │   │   ├── 15/
│   │   │   │   │   │   ├── 10/
│   │   │   │   │   │   │   ├── 00/
│   │   │   │   │   │   │   │   └── 17.avro  ← File created at 10:00:17
│   │   │   │   │   │   │   ├── 05/
│   │   │   │   │   │   │   │   └── 42.avro  ← File created at 10:05:42
│   │   ├── 1/
│   │   │   └── 2024/01/15/10/00/23.avro
│   │   ├── 2/
│   │   └── 3/
```

**Custom Naming (Optional):**
You can add a custom prefix:
```
{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}
→ mydata/{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}
```

### Параметры конфигурации (Event Hubs Capture)

| Параметр | Описание | Диапазон | Рекомендация |
|-----------|----------|-----------|---------------|
| **Time Window** | Интервал времени между операциями захвата (capture) | 1–15 минут | 5–10 минут (баланс между задержкой и количеством файлов) |
| **Size Window** | Размер данных (в МБ), после которого выполняется захват | 10–500 МБ | 100–300 МБ (оптимально для последующей обработки) |
| **Skip Empty** | Создавать ли пустые файлы при отсутствии событий | true / false | false (предсказуемая структура и отсутствие лишних файлов) |

---

## Включение Capture

Event Hubs Capture позволяет автоматически сохранять поток событий в:

- Azure Storage Blob
- Azure Data Lake

Это часто используется для:
- аналитики
- архивирования
- последующей batch-обработки
- интеграции с системами обработки больших данных

---

### Настройка через Azure Portal

1. Перейдите в **Event Hubs namespace**
2. Выберите нужный **Event Hub**
3. В разделе **Settings** выберите **Capture**
4. Включите переключатель **On**
5. Настройте параметры:

    - **Time window**: 5 минут (пример)
    - **Size window**: 100 МБ (пример)
    - **Capture provider**: Azure Storage Blob или Data Lake
    - **Storage account**: выбрать существующий или создать новый
    - **Container**: указать имя контейнера
    - **Naming format**: использовать стандартный или задать пользовательский

6. Нажмите **Save**

---

## Практические рекомендации для AZ-204

- Capture работает на уровне Event Hub (не namespace).
- Файл создаётся при достижении **Time Window ИЛИ Size Window** (что произойдёт раньше).
- Включённый Skip Empty = false помогает поддерживать регулярный график файлов.
- Capture используется для интеграции с аналитическими сервисами (например, Spark, Synapse).

---

## Что важно запомнить

- Capture — это автоматический механизм сохранения потока событий.
- Поддерживается Blob Storage и Data Lake.
- Конфигурация включает временное окно и размер окна.
- Файлы формируются по принципу "что наступит раньше — время или размер".

Эта тема часто встречается в вопросах про потоковую обработку и интеграцию Event Hubs с аналитическими системами.

### Azure CLI

**Create Event Hub with Capture:**

```bash
# Variables
RG="rg-eventhubs"
NAMESPACE="myeventhubns"
EVENTHUB="myeventhub"
STORAGE_ACCOUNT="mystorageaccount"
CONTAINER="captures"

# Create storage account (if not exists)
az storage account create \
  --name $STORAGE_ACCOUNT \
  --resource-group $RG \
  --location eastus \
  --sku Standard_LRS

# Get storage account resource ID
STORAGE_ID=$(az storage account show \
  --name $STORAGE_ACCOUNT \
  --resource-group $RG \
  --query id \
  --output tsv)

# Create container
az storage container create \
  --name $CONTAINER \
  --account-name $STORAGE_ACCOUNT

# Create Event Hub with Capture enabled
az eventhubs eventhub create \
  --name $EVENTHUB \
  --namespace-name $NAMESPACE \
  --resource-group $RG \
  --partition-count 4 \
  --message-retention 7 \
  --enable-capture true \
  --capture-interval 300 \
  --capture-size-limit 104857600 \
  --destination-name EventHubArchive.AzureBlockBlob \
  --storage-account $STORAGE_ID \
  --blob-container $CONTAINER \
  --archive-name-format "{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}"
```

**Enable Capture on Existing Event Hub:**

```bash
az eventhubs eventhub update \
  --name $EVENTHUB \
  --namespace-name $NAMESPACE \
  --resource-group $RG \
  --enable-capture true \
  --capture-interval 300 \
  --capture-size-limit 104857600 \
  --destination-name EventHubArchive.AzureBlockBlob \
  --storage-account $STORAGE_ID \
  --blob-container $CONTAINER
```

**Capture Parameters Explained:**

| Parameter | Value | Description |
|-----------|-------|-------------|
| `--enable-capture` | true | Enable Capture feature |
| `--capture-interval` | 300 | Time window in seconds (5 minutes) |
| `--capture-size-limit` | 104857600 | Size window in bytes (100 MB) |
| `--destination-name` | EventHubArchive.AzureBlockBlob | Capture destination type |
| `--storage-account` | Resource ID | Target storage account |
| `--blob-container` | captures | Container name |
| `--archive-name-format` | Pattern | File naming pattern |

### ARM Template

```json
{
  "type": "Microsoft.EventHub/namespaces/eventhubs",
  "apiVersion": "2021-11-01",
  "name": "[concat(parameters('namespaceName'), '/', parameters('eventHubName'))]",
  "properties": {
    "messageRetentionInDays": 7,
    "partitionCount": 4,
    "captureDescription": {
      "enabled": true,
      "skipEmptyArchives": false,
      "encoding": "Avro",
      "intervalInSeconds": 300,
      "sizeLimitInBytes": 104857600,
      "destination": {
        "name": "EventHubArchive.AzureBlockBlob",
        "properties": {
          "storageAccountResourceId": "[resourceId('Microsoft.Storage/storageAccounts', parameters('storageAccountName'))]",
          "blobContainer": "captures",
          "archiveNameFormat": "{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}"
        }
      }
    }
  }
}
```

---

## Reading Captured Avro Files

### Using Python (Avro Library)

```python
from avro.datafile import DataFileReader
from avro.io import DatumReader
from azure.storage.blob import BlobServiceClient
import io

# Download Avro file from Blob Storage
connection_string = "<storage-connection-string>"
container_name = "captures"
blob_name = "mynamespace/myeventhub/0/2024/01/15/10/00/17.avro"

blob_service_client = BlobServiceClient.from_connection_string(connection_string)
blob_client = blob_service_client.get_blob_client(container_name, blob_name)

# Download to memory
blob_data = blob_client.download_blob().readall()
avro_stream = io.BytesIO(blob_data)

# Read Avro file
reader = DataFileReader(avro_stream, DatumReader())

for event in reader:
    sequence_number = event['SequenceNumber']
    offset = event['Offset']
    enqueued_time = event['EnqueuedTimeUtc']
    body = event['Body']
    
    # Decode body (UTF-8 string)
    body_str = body.decode('utf-8')
    
    print(f"Sequence: {sequence_number}")
    print(f"Offset: {offset}")
    print(f"Time: {enqueued_time}")
    print(f"Body: {body_str}")
    print("---")

reader.close()
```

### Using C# (Avro Library)

```csharp
using Avro.File;
using Avro.Generic;
using Azure.Storage.Blobs;

string connectionString = "<storage-connection-string>";
string containerName = "captures";
string blobName = "mynamespace/myeventhub/0/2024/01/15/10/00/17.avro";

// Download Avro file
var blobClient = new BlobClient(connectionString, containerName, blobName);
using var stream = new MemoryStream();
await blobClient.DownloadToAsync(stream);
stream.Position = 0;

// Read Avro file
using var reader = DataFileReader<GenericRecord>.OpenReader(stream);

foreach (var record in reader.NextEntries)
{
    long sequenceNumber = (long)record["SequenceNumber"];
    string offset = (string)record["Offset"];
    string enqueuedTime = (string)record["EnqueuedTimeUtc"];
    byte[] body = (byte[])record["Body"];
    
    // Decode body
    string bodyStr = Encoding.UTF8.GetString(body);
    
    Console.WriteLine($"Sequence: {sequenceNumber}");
    Console.WriteLine($"Offset: {offset}");
    Console.WriteLine($"Time: {enqueuedTime}");
    Console.WriteLine($"Body: {bodyStr}");
    Console.WriteLine("---");
}
```

### Using Azure Databricks (PySpark)

```python
# Mount storage account (one-time setup)
dbutils.fs.mount(
    source = "wasbs://captures@mystorageaccount.blob.core.windows.net",
    mount_point = "/mnt/captures",
    extra_configs = {
        "fs.azure.account.key.mystorageaccount.blob.core.windows.net": "<storage-key>"
    }
)

# Read Avro files with Spark
df = spark.read.format("avro").load("/mnt/captures/mynamespace/myeventhub/*/*/*/*/*/*/*/*.avro")

# Show schema
df.printSchema()

# Query events
df.select("SequenceNumber", "EnqueuedTimeUtc", "Body").show()

# Decode body and convert to JSON
from pyspark.sql.functions import col, decode

df_decoded = df.withColumn("BodyString", decode(col("Body"), "UTF-8"))

# Parse JSON body
from pyspark.sql.functions import from_json, schema_of_json

# Infer schema from sample
sample_json = df_decoded.select("BodyString").first()[0]
json_schema = schema_of_json(sample_json)

# Parse all events
df_parsed = df_decoded.withColumn("BodyJson", from_json(col("BodyString"), json_schema))

# Query specific fields
df_parsed.select("EnqueuedTimeUtc", "BodyJson.*").show()
```

### Using Azure Synapse Analytics

```sql
-- Create external data source
CREATE EXTERNAL DATA SOURCE CapturedEvents
WITH (
    TYPE = HADOOP,
    LOCATION = 'wasbs://captures@mystorageaccount.blob.core.windows.net',
    CREDENTIAL = AzureStorageCredential
);

-- Query Avro files
SELECT 
    SequenceNumber,
    Offset,
    EnqueuedTimeUtc,
    CAST(Body AS VARCHAR(MAX)) AS BodyText
FROM OPENROWSET(
    BULK '/mynamespace/myeventhub/*/*/*/*/*/*/*/*.avro',
    DATA_SOURCE = 'CapturedEvents',
    FORMAT = 'PARQUET'
) AS [Events];
```

---

## Throughput and Capacity

### Throughput Units and Capture

**Key Points:**
- ✅ **Capture does NOT consume egress TU quota**
- ✅ Operates directly from Event Hubs internal storage
- ✅ No impact on real-time consumers
- ✅ No additional TU cost for Capture operation

**Throughput Unit Reminder:**
- 1 TU = 1 MB/s ingress, 2 MB/s egress
- Standard tier: 1-40 TUs
- Capture bypasses egress quota

**Example Calculation:**

```
Scenario: 10 MB/s ingress, real-time consumers reading 5 MB/s

WITHOUT Capture:
- Ingress: 10 TUs (10 MB/s ÷ 1 MB/s per TU)
- Egress: 3 TUs (5 MB/s ÷ 2 MB/s per TU)
- Required: 10 TUs

WITH Capture (10 MB/s also captured):
- Ingress: 10 TUs (10 MB/s ÷ 1 MB/s per TU)
- Egress: 3 TUs (5 MB/s ÷ 2 MB/s per TU)
- Capture: 0 TUs (bypasses egress quota)
- Required: Still 10 TUs!

Capture is effectively FREE from TU perspective!
```

### Storage Costs

**Storage Tiers:**

| Tier | Cost (per GB/month) | Use Case |
|------|---------------------|----------|
| **Hot** | ~$0.02 | Recent data, frequent access |
| **Cool** | ~$0.01 | 30-90 day retention, infrequent access |
| **Archive** | ~$0.002 | Long-term compliance (>90 days) |

**Lifecycle Management:**
```bash
# Move to Cool after 30 days, Archive after 90 days
az storage account management-policy create \
  --account-name mystorageaccount \
  --resource-group myResourceGroup \
  --policy @lifecycle-policy.json
```

**lifecycle-policy.json:**
```json
{
  "rules": [
    {
      "enabled": true,
      "name": "MoveToArchive",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90
            }
          }
        },
        "filters": {
          "blobTypes": ["blockBlob"],
          "prefixMatch": ["captures/"]
        }
      }
    }
  ]
}
```

---

## Monitoring Capture

### Azure Portal Metrics

Navigate to: **Event Hub → Metrics**

**Key Metrics:**

| Metric | Description | Alert Threshold |
|--------|-------------|----------------|
| **Captured Bytes** | Total bytes captured | Monitor for gaps |
| **Captured Messages** | Total messages captured | Compare with ingress |
| **Capture Backlog** | Events waiting to be captured | > 0 (should be low) |
| **Capture Errors** | Failed capture attempts | > 0 |

### Azure CLI Monitoring

```bash
# View capture metrics
az monitor metrics list \
  --resource /subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{ns}/eventhubs/{eh} \
  --metric "CapturedMessages" \
  --start-time 2024-01-15T00:00:00Z \
  --end-time 2024-01-15T23:59:59Z \
  --interval PT1H
```

### Diagnostic Logs

**Enable diagnostic logs:**

```bash
az monitor diagnostic-settings create \
  --name CaptureLogging \
  --resource /subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.EventHub/namespaces/{ns}/eventhubs/{eh} \
  --logs '[{"category":"ArchiveLogs","enabled":true}]' \
  --workspace /subscriptions/{subscription}/resourceGroups/{rg}/providers/Microsoft.OperationalInsights/workspaces/{workspace}
```

**Query logs (KQL):**
```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.EVENTHUB"
| where Category == "ArchiveLogs"
| where OperationName == "Archive"
| project TimeGenerated, Status, Message, PartitionId
| order by TimeGenerated desc
```

---

## Лучшие практики (Event Hubs Capture)

### Конфигурация

1. **Time Window (временное окно)**
    - Короткое (1–5 мин): ниже задержка, больше файлов
    - Длинное (10–15 мин): меньше файлов, выше задержка
    - **Рекомендация**: 5 минут для большинства сценариев

2. **Size Window (размер окна)**
    - Малый размер (10–50 МБ): больше файлов, быстрее старт обработки
    - Большой размер (100–500 МБ): меньше файлов, выше эффективность batch-обработки
    - **Рекомендация**: 100–300 МБ для Spark / Databricks

3. **Пустые файлы (Empty Files)**
    - Рекомендуется оставить создание пустых файлов включённым (`skipEmptyArchives = false`)
    - Обеспечивает предсказуемую периодичность
    - Упрощает мониторинг и автоматизацию

4. **Партиции (Partitions)**
    - Каждая партиция создаёт отдельные файлы
    - Больше партиций → больше файлов
    - Важно балансировать количество партиций и уровень параллелизма обработки

---

## Обработка данных (Processing)

1. **Инкрементальная обработка**
    - Отслеживайте timestamp последнего обработанного файла
    - Обрабатывайте только новые файлы
    - Используйте watermarking в Spark Structured Streaming

2. **Параллельная обработка**
    - Обрабатывайте партиции параллельно
    - Используйте Spark / Databricks для масштабируемости
    - Оптимально — один executor на партицию

3. **Обработка ошибок**
    - Обрабатывайте повреждённые или неполные файлы
    - Реализуйте retry-логику
    - Логируйте проблемные файлы для ручного анализа

4. **Эволюция схемы (Schema Evolution)**
    - Используйте возможности Avro для эволюции схем
    - Обеспечивайте backward / forward compatibility
    - Версионируйте схемы

---

## Оптимизация затрат (Cost Optimization)

1. **Storage Tier**
    - Используйте lifecycle management
    - Cool tier — для хранения 30–90 дней
    - Archive tier — для длительного хранения (> 90 дней, compliance)

2. **Сжатие (Compression)**
    - Avro уже использует встроенное сжатие
    - Можно дополнительно применять gzip
    - Балансируйте экономию хранения и нагрузку на CPU

3. **Retention**
    - Удаляйте старые файлы при отсутствии требований к хранению
    - Балансируйте требования по хранению и стоимость

4. **Именование файлов (Naming Convention)**
    - Используйте единый стандарт именования
    - Добавляйте метаданные в структуру папок
    - Оптимизируйте структуру под шаблоны запросов (например, партиционирование по дате)

---

## Устранение проблем (Troubleshooting)

### Распространённые проблемы

---

### Проблема 1: Файлы не создаются

**Симптомы:**
- Capture включён
- В хранилище отсутствуют файлы

**Возможные причины:**

- В Event Hub не публикуются события
- Проблемы подключения к Storage Account
- Недостаточные права доступа

**Что проверить:**

- Поступают ли события в Event Hub (метрики Incoming Messages)
- Корректность connection string / managed identity
- Наличие прав на запись в контейнер
- Не превышены ли квоты Storage

---

## Что важно для AZ-204

- Capture работает на уровне Event Hub
- Файлы создаются по принципу: **Time Window ИЛИ Size Window (что раньше)**
- Количество партиций влияет на количество создаваемых файлов
- Capture часто используется для интеграции с аналитическими системами

Понимание конфигурации Capture и её влияния на производительность и стоимость — важный аспект потоковой архитектуры в Azure.
**Resolution:**
```bash
# Check Event Hub metrics
az monitor metrics list \
  --resource <event-hub-resource-id> \
  --metric "IncomingMessages"

# Verify storage account connection
az storage container show \
  --name captures \
  --account-name mystorageaccount

# Check Event Hub capture configuration
az eventhubs eventhub show \
  --name myeventhub \
  --namespace-name myeventhubns \
  --resource-group myResourceGroup \
  --query captureDescription
```

---

### Проблема 2: Задержка Capture (Capture Lag)

**Симптомы:**
- События сохраняются в хранилище с заметной задержкой
- Файлы появляются позже ожидаемого времени

**Возможные причины:**

- Слишком большое значение **Time Window**
- Слишком большой **Size Window**
- Низкая пропускная способность (low throughput) — данные медленно накапливаются

**Решение:**

- Уменьшить временное окно (например, с 5 мин → до 2 мин)
- Уменьшить размер окна (например, с 500 МБ → до 100 МБ)
- Отслеживать метрику **CaptureBacklog** в Azure Monitor
- Проверить метрики Incoming Messages и Throughput Units

📌 Важно понимать: файл создаётся только при достижении Time Window или Size Window.  
Если поток событий небольшой, Size Window может долго не достигаться.

---

### Проблема 3: Невозможно прочитать Avro-файлы

**Симптомы:**
- Ошибки при чтении Avro-файлов
- Ошибки десериализации
- Исключения при загрузке в Spark / Databricks

**Возможные причины:**

- Используется несовместимая версия Avro-библиотеки
- Файл ещё не полностью записан (процесс Capture не завершён)
- Файл повреждён

**Решение:**

- Проверить совместимость версии Avro (особенно при использовании schema evolution)
- Убедиться, что файл полностью сформирован перед чтением
- Проверить размер файла (не нулевой ли он)
- Перепроверить корректность lifecycle и прав доступа
- Использовать инструменты проверки Avro (например, avro-tools)

📌 Для production-сценариев рекомендуется:
- Реализовать retry при чтении
- Игнорировать частично записанные файлы
- Логировать проблемные файлы для последующего анализа

---

## Что важно для AZ-204

- Capture может создавать задержку при больших окнах
- Метрики (особенно CaptureBacklog) помогают диагностировать проблему
- Avro используется по умолчанию для хранения событий
- Нужно учитывать совместимость схем и корректность обработки файлов

Понимание причин задержек и проблем с Avro-файлами — важная часть диагностики потоковых решений в Azure.
**Resolution:**
```python
# Check file size (incomplete files may be 0 bytes)
from azure.storage.blob import BlobServiceClient

blob_client = BlobServiceClient.from_connection_string(conn_str)
blob = blob_client.get_blob_client("captures", blob_name)
properties = blob.get_blob_properties()
print(f"Size: {properties.size} bytes")

# Try reading with error handling
try:
    reader = DataFileReader(stream, DatumReader())
    for record in reader:
        print(record)
except Exception as e:
    print(f"Error reading file: {e}")
```

---

## Советы к экзамену AZ-204 (Event Hubs Capture)

## Ключевые концепции

1. **Capture** — автоматическое сохранение событий в хранилище  
   (не требует написания кода)

2. **Формат** — Apache Avro
    - компактный
    - быстрый
    - бинарный
    - содержит схему (self-describing)

3. **Оконная модель (Windowing)** —  
   используется правило **Time OR Size (что наступит раньше)**

4. **Нет дополнительных TU-затрат** —  
   Capture не расходует egress-квоту Throughput Units

5. **Назначение (Destinations)** —
    - Azure Blob Storage
    - Azure Data Lake Storage

6. **Формат именования файлов** —  
   `{Namespace}/{EventHub}/{PartitionId}/{Year}/{Month}/{Day}/{Hour}/{Minute}/{Second}`

7. **Пустые файлы** —  
   создаются даже при отсутствии событий (для предсказуемой периодичности)

---

## Типовые экзаменационные сценарии

### Сценарий 1: Долгосрочное хранение (compliance)

✔ Включить Event Hubs Capture  
✔ Настроить retention-политику  
✔ Использовать lifecycle management (Cool / Archive tiers)

---

### Сценарий 2: Построение Data Lake для аналитики

✔ Включить Capture в Data Lake Storage  
✔ Обрабатывать данные через Databricks / Synapse  
✔ Выполнять запросы через Spark SQL

---

### Сценарий 3: Lambda-архитектура (real-time + batch)

✔ Реалтайм-консьюмеры для немедленной обработки  
✔ Capture для batch-аналитики  
✔ Оба механизма читают одни и те же события независимо

Важно: Capture не мешает обычным consumer group.

---

### Сценарий 4: Оптимизация затрат

✔ Capture не потребляет TU egress-квоту  
✔ Использовать lifecycle management для хранения  
✔ Балансировать Time Window и Size Window для контроля количества файлов

---

## Что обязательно помнить на экзамене

- **Avro** — компактный, быстрый, со встроенной схемой
- **Time Window**: 1–15 минут
- **Size Window**: 10–500 МБ
- Принцип: **Time ИЛИ Size (что раньше)**
- Capture не влияет на Throughput Units
- Каждая партиция создаёт отдельные файлы
- Пустые файлы обеспечивают предсказуемую периодичность
- Поддерживаемые назначения: Blob Storage или Data Lake

---

## Экзаменационный лайфхак

Если в вопросе говорится о:

- долгосрочном хранении
- построении Data Lake
- batch-аналитике
- compliance
- необходимости сохранять все события без написания кода

— почти всегда правильный ответ связан с **Event Hubs Capture**.

Понимание разницы между real-time consumer и Capture — ключевой момент для AZ-204.
### Quick Command Reference

```bash
# Enable capture on new Event Hub
az eventhubs eventhub create \
  --name <eh> \
  --namespace-name <ns> \
  --resource-group <rg> \
  --enable-capture true \
  --capture-interval 300 \
  --capture-size-limit 104857600 \
  --storage-account <storage-id> \
  --blob-container <container>

# Enable capture on existing Event Hub
az eventhubs eventhub update \
  --name <eh> \
  --namespace-name <ns> \
  --resource-group <rg> \
  --enable-capture true

# Check capture status
az eventhubs eventhub show \
  --name <eh> \
  --namespace-name <ns> \
  --resource-group <rg> \
  --query captureDescription
```

---

## Итог

**Event Hubs Capture** автоматически сохраняет потоковые данные в хранилище для долгосрочного хранения и batch-аналитики.

---

## Ключевые возможности

- Автоматический захват событий (без написания кода)
- Формат Apache Avro
- Оконная модель: **Time ИЛИ Size (что раньше)**
- Нет затрат на TU egress
- Назначение: Blob Storage или Data Lake Storage

---

## Конфигурация

- **Time window**: 1–15 минут
- **Size window**: 10–500 МБ
- **Именование**: стандартный или пользовательский формат
- **Пустые файлы**: обеспечивают предсказуемую периодичность

---

## Основные сценарии использования

- Построение Data Lake для batch-аналитики
- Долгосрочное хранение и соответствие требованиям compliance
- Lambda-архитектура (real-time + batch)
- Повторное воспроизведение и повторная обработка данных

---

## Преимущества

- Экономичность (не расходует TU egress-квоту)
- Масштабируемость (поддержка миллионов событий в секунду)
- Надёжность (долговременное хранение)
- Гибкость (обработка любыми инструментами с поддержкой Avro)

---

## Главное, что нужно помнить для AZ-204

Если требуется:

- сохранять все события автоматически
- построить аналитическое хранилище
- обеспечить долгосрочное хранение
- реализовать batch-обработку

— правильным решением почти всегда будет **Event Hubs Capture**.