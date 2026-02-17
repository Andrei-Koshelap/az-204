# Container Restart Policies в Azure Container Instances

## Ключевые понятия (Key Concepts)

- **Restart policy** — определяет поведение контейнера после завершения работы
- **Always** — перезапуск при любом завершении (по умолчанию)
- **Never** — не перезапускать (однократный запуск)
- **OnFailure** — перезапуск только при ошибке (код выхода ≠ 0)

---

# Обзор Restart Policy

Определяет, что произойдёт после завершения контейнера:

- Нужно ли автоматически перезапускать контейнер
- Влияет на стоимость (остановленные контейнеры не тарифицируются)
- Подходит для batch-задач и одноразовых процессов
- Оплата посекундная — платите только во время выполнения

---

# Доступные политики перезапуска

| Политика | Поведение | Сценарий | Статус после завершения |
|-----------|------------|-----------|--------------------------|
| **Always** | Всегда перезапускать | Долгоживущие сервисы, веб-приложения | Running |
| **Never** | Никогда не перезапускать | Одноразовые задачи, batch | Terminated |
| **OnFailure** | Перезапуск при ошибке | Задачи с возможностью повторной попытки | Running или Terminated |

---

## Когда какую использовать

- **Always** → API, web-сервис, постоянная служба
- **Never** → миграции БД, одноразовые скрипты
- **OnFailure** → retry-логика для batch-процессов

---

## Экзаменационный акцент (AZ-204)

- По умолчанию используется **Always**
- Для одноразовой задачи → **Never**
- Для retry при ошибке → **OnFailure**
- Остановленный контейнер не тарифицируется


### Always (Default)
```bash
# Containers always restart (even on success)
az container create \
  --resource-group myResourceGroup \
  --name web-app \
  --image nginx \
  --restart-policy Always

# Use for:
# - Web servers
# - APIs
# - Long-running services
# - Continuous monitoring
```

## Behavior (Политика Always — по умолчанию)

- Код выхода 0 (успешное завершение) → контейнер перезапускается
- Код выхода ≠ 0 (ошибка) → контейнер перезапускается
- Используется по умолчанию, если политика не указана

---

## Что это означает

- Контейнер будет постоянно поддерживаться в состоянии Running
- Подходит для долгоживущих сервисов
- Не подходит для одноразовых задач (они будут перезапускаться снова и снова)

---

## Экзаменационный акцент (AZ-204)

- Если restart policy не указана → используется **Always**
- Always → перезапуск при любом коде выхода
- Для batch-задач нужно явно указывать `Never` или `OnFailure`

### Never
```bash
# Container never restarts
az container create \
  --resource-group myResourceGroup \
  --name batch-job \
  --image myapp \
  --restart-policy Never

# Use for:
# - One-time batch jobs
# - Data processing tasks
# - Image rendering
# - Build jobs
# - Database migrations
```

## Behavior (Политика Always — по умолчанию)

- Код выхода 0 (успешное завершение) → контейнер перезапускается
- Код выхода ≠ 0 (ошибка) → контейнер перезапускается
- Используется по умолчанию, если политика не указана

---

## Что это означает

- Контейнер будет постоянно поддерживаться в состоянии Running
- Подходит для долгоживущих сервисов
- Не подходит для одноразовых задач (они будут перезапускаться снова и снова)

---

## Экзаменационный акцент (AZ-204)

- Если restart policy не указана → используется **Always**
- Always → перезапуск при любом коде выхода
- Для batch-задач нужно явно указывать `Never` или `OnFailure`

### OnFailure
```bash
# Container restarts only on failure
az container create \
  --resource-group myResourceGroup \
  --name retry-task \
  --image myapp \
  --restart-policy OnFailure

# Use for:
# - Tasks that may fail temporarily
# - Network-dependent operations
# - API calls with retry logic
# - Data sync tasks
```

## Behavior (Политика OnFailure)

- Код выхода 0 (успешное завершение) → контейнер переходит в состояние Terminated
- Код выхода ≠ 0 (ошибка) → контейнер перезапускается и выполняет повторную попытку
- Контейнер гарантированно запускается минимум один раз

---

# Run-to-Completion Tasks

## Идеально для batch-задач

Подходит для сценариев, где контейнер:

- Выполняет работу
- Завершается
- Не должен работать постоянно

---

## Преимущества

- Быстрый старт (секунды)
- Выполнение задачи до завершения
- Оплата только за фактическое время работы
- Автоматическое завершение после выполнения

---

## Типичные сценарии

- Обработка файлов
- Миграции базы данных
- Генерация отчётов
- Data processing jobs
- CI/CD вспомогательные задачи

---

## Экзаменационный акцент (AZ-204)

- Run-once задача → `Never`
- Retry при ошибке → `OnFailure`
- Долгоживущий сервис → `Always`
- Batch + оплата только за выполнение → ACI + `OnFailure` или `Never`


### Example: Data Processing
```bash
# Process data file and exit
az container create \
  --resource-group myResourceGroup \
  --name data-processor \
  --image myprocessor:v1.0 \
  --restart-policy Never \
  --environment-variables \
    'INPUT_FILE'='data.csv' \
    'OUTPUT_FILE'='results.json' \
  --azure-file-volume-account-name mystorage \
  --azure-file-volume-account-key <key> \
  --azure-file-volume-share-name data \
  --azure-file-volume-mount-path /data

# Container runs once and stops
# Status becomes "Terminated"
```

### Example: Build Job
```bash
# Build container image
az container create \
  --resource-group myResourceGroup \
  --name build-job \
  --image docker:dind \
  --restart-policy OnFailure \
  --command-line "docker build -t myapp:latest ." \
  --azure-file-volume-mount-path /workspace

# If build fails → Restart
# If build succeeds → Terminate
```

## Container Status

### Status Lifecycle

```
Create → Waiting → Running → Succeeded/Failed → Terminated
                              ↓ (OnFailure)
                            Restart
```

### Check Status
```bash
# View container status
az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --query instanceView.state

# Possible states:
# - Waiting
# - Running
# - Succeeded (exit code 0)
# - Failed (exit code != 0)
# - Terminated
```

### View Logs After Termination
```bash
# Even after container stops, logs are available
az container logs \
  --resource-group myResourceGroup \
  --name mycontainer

# View last exit code
az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --query "instanceView.currentState.exitCode"
```

## Practical Examples

### Example 1: Web Server (Always)
```bash
# NGINX web server - always running
az container create \
  --resource-group myResourceGroup \
  --name nginx-server \
  --image nginx:alpine \
  --dns-name-label mywebsite \
  --ports 80 \
  --restart-policy Always

# Stays running indefinitely
# Restarts if crashes
```

### Example 2: Nightly Backup (Never)
```bash
# Backup job runs once per schedule
az container create \
  --resource-group myResourceGroup \
  --name backup-job \
  --image backup-tool:latest \
  --restart-policy Never \
  --environment-variables \
    'SOURCE'='/data' \
    'DESTINATION'='https://backup.blob.core.windows.net' \
  --azure-file-volume-mount-path /data

# Runs once, then stops
# Triggered by external scheduler (Logic Apps, Azure Automation)
```

### Example 3: Message Processor (OnFailure)
```bash
# Process queue messages with retry
az container create \
  --resource-group myResourceGroup \
  --name message-processor \
  --image processor:v1.0 \
  --restart-policy OnFailure \
  --environment-variables \
    'QUEUE_NAME'='messages' \
    'CONNECTION_STRING'='...'

# Success: Process messages → Exit 0 → Terminate
# Failure: Network error → Exit 1 → Restart
```

### Example 4: Image Rendering (OnFailure)
```bash
# Render video frame
az container create \
  --resource-group myResourceGroup \
  --name renderer \
  --image renderer:latest \
  --restart-policy OnFailure \
  --gpu-count 1 \
  --gpu-sku K80 \
  --environment-variables \
    'FRAME'='42' \
    'SCENE'='explosion'

# Render success → Exit 0 → Terminate
# Render failure → Exit 1 → Restart
```

## YAML Configuration

### Always
```yaml
apiVersion: '2021-09-01'
location: eastus
name: always-running
properties:
  containers:
  - name: webapp
    properties:
      image: nginx
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1
          memoryInGB: 1.5
  osType: Linux
  restartPolicy: Always
```

### Never
```yaml
apiVersion: '2021-09-01'
location: eastus
name: one-time-task
properties:
  containers:
  - name: batch-job
    properties:
      image: batch-processor:latest
      resources:
        requests:
          cpu: 2
          memoryInGB: 4
  osType: Linux
  restartPolicy: Never
```

### OnFailure
```yaml
apiVersion: '2021-09-01'
location: eastus
name: retry-on-failure
properties:
  containers:
  - name: data-sync
    properties:
      image: sync-tool:v1.0
      environmentVariables:
      - name: RETRY_COUNT
        value: '3'
      resources:
        requests:
          cpu: 1
          memoryInGB: 2
  osType: Linux
  restartPolicy: OnFailure
```

# Exit Codes (Коды завершения контейнера)

## Стандартные коды выхода

| Exit Code | Значение | Перезапуск при OnFailure? |
|------------|-----------|----------------------------|
| **0** | Успешное завершение | Нет (Terminated) |
| **1** | Общая ошибка | Да |
| **2** | Неверное использование команды | Да |
| **126** | Невозможно выполнить | Да |
| **127** | Команда не найдена | Да |
| **137** | SIGKILL (обычно OOM — нехватка памяти) | Да |
| **139** | Segmentation fault | Да |

---

## Что важно понимать

- Код выхода **0** означает успешное завершение задачи.
- Любой код ≠ 0 считается ошибкой.
- При политике `OnFailure` контейнер будет перезапущен при ошибке.
- Код 137 часто указывает на нехватку памяти (Out Of Memory).

---

## Экзаменационный акцент (AZ-204)

- Exit code 0 → успех, без перезапуска при `OnFailure`
- Exit code ≠ 0 → перезапуск при `OnFailure`
- OOM (137) → признак нехватки памяти
- Для batch-задач важно правильно выбрать restart policy


### Setting Exit Codes in Your App
```bash
# Shell script
#!/bin/bash
if [ -f "/data/input.txt" ]; then
    # Process file
    process_file
    exit 0  # Success
else
    echo "Input file not found"
    exit 1  # Failure - will trigger restart with OnFailure
fi
```

```python
# Python
import sys

try:
    process_data()
    sys.exit(0)  # Success
except Exception as e:
    print(f"Error: {e}")
    sys.exit(1)  # Failure - will trigger restart with OnFailure
```

# Cost Implications (Влияние политики перезапуска на стоимость)

## Тарификация в зависимости от Restart Policy

| Политика | Время работы | Модель затрат |
|-----------|--------------|---------------|
| **Always** | Непрерывно | Постоянные расходы (пока не удалён ресурс) |
| **Never** | Один запуск | Разовая стоимость выполнения |
| **OnFailure** | До успешного завершения | Переменные расходы (зависят от числа ошибок) |

---

## Что это означает

- **Always** — контейнер постоянно работает → постоянная тарификация
- **Never** — контейнер завершился → оплата прекращается
- **OnFailure** — каждый перезапуск увеличивает общее время работы и стоимость

---

## Практические рекомендации

- Для batch-задач → `Never` или `OnFailure`
- Для долгоживущих сервисов → `Always`
- Контролируйте exit codes, чтобы избежать бесконечных перезапусков
- Мониторьте использование CPU и памяти

---

## Экзаменационный акцент (AZ-204)

- Оплата в ACI идёт только за время работы контейнера
- Неправильный restart policy может увеличить расходы
- Batch-задачи + минимальная стоимость → `Never`
- Retry-логика → `OnFailure`
- Web/API сервис → `Always`

### Example Cost Calculation
```
Container: 1 CPU, 2 GB memory
Region: East US
CPU: $0.0000012/core/second
Memory: $0.0000001/GB/second

Always (24 hours):
= 86,400 seconds
= (1 × $0.0000012 × 86,400) + (2 × $0.0000001 × 86,400)
= $0.12/day

Never (5 minutes):
= 300 seconds
= (1 × $0.0000012 × 300) + (2 × $0.0000001 × 300)
= $0.0004 per run

OnFailure (3 retries, 2 min each):
= 360 seconds total
= (1 × $0.0000012 × 360) + (2 × $0.0000001 × 360)
= $0.0005 per task
```

## Best Practices

### 1. Choose Right Policy
```bash
# Long-running services
--restart-policy Always

# Batch jobs, migrations
--restart-policy Never

# Tasks with transient failures
--restart-policy OnFailure
```

### 2. Implement Proper Exit Codes
```python
# Return correct exit codes
sys.exit(0)  # Success
sys.exit(1)  # Failure
```

### 3. Monitor Container Status
```bash
# Check final status
az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --query "instanceView.state"
```

### 4. Use Logs for Debugging
```bash
# View logs even after termination
az container logs \
  --resource-group myResourceGroup \
  --name mycontainer
```

### 5. Set Timeouts
```bash
# Prevent infinite retries with OnFailure
# Implement timeout logic in your application
```

# Critical Notes — Container Restart Policies

- 💡 **Политика по умолчанию** — Always (перезапуск при любом завершении)
- ⚠️ **Never** — однократный запуск, затем завершение (идеально для batch-задач)
- 🎯 **OnFailure** — перезапуск только при ошибке (exit code ≠ 0)
- ✅ **Оплата посекундно** — остановленные контейнеры не тарифицируются
- 📊 **Exit code 0** — успешное завершение (без перезапуска при OnFailure)
- 🔄 **Exit code ≠ 0** — ошибка (перезапуск при OnFailure)
- 🔒 **Состояние Terminated** — контейнер остановлен, логи доступны
- ⏱️ **Run-to-completion** — идеально для batch-задач и обработки данных

---

# Exam Tips (AZ-204)

## Политики перезапуска

- **Always** — по умолчанию, перезапуск при любом завершении  
  (подходит для долгоживущих сервисов)

- **Never** — запуск один раз, без перезапуска  
  (batch-задачи, миграции, одноразовые скрипты)

- **OnFailure** — перезапуск только при ошибке  
  (exit code ≠ 0)

---

## Коды завершения

- Exit code 0 → успех → OnFailure не перезапускает
- Exit code ≠ 0 → ошибка → OnFailure перезапускает

---

## Поведение после завершения

- При `Never` или успешном `OnFailure` → статус **Terminated**
- Логи доступны даже после завершения контейнера

---

## Настройка

- CLI:  
  `--restart-policy Always|Never|OnFailure`

- YAML:  
  `restartPolicy: Always|Never|OnFailure`

---

## Биллинг

- Оплата только во время работы контейнера
- Run-to-completion — оптимально для batch-задач
- OnFailure гарантирует минимум один запуск

---

## Частые экзаменационные ловушки

- Если политика не указана → Always
- Batch-задача без перезапуска → Never
- Retry при ошибке → OnFailure
- Логи доступны после завершения


[Learn More](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/4-run-containerized-tasks-restart-policies)
