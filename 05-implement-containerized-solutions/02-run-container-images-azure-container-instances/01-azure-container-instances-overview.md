# Azure Container Instances (ACI) — Обзор

## Ключевые понятия (Key Concepts)

- **ACI** — самый быстрый способ запустить контейнеры в Azure
- **Container groups** — группа контейнеров на одном хосте (аналог Kubernetes pod)
- **Изолированные контейнеры** — не требуется оркестратор
- **Оплата посекундно** — платите только за фактически использованное время

---

# Что такое Azure Container Instances?

**Самый быстрый и простой способ** запустить контейнер в Azure.

- Не требуется управление виртуальными машинами
- Запуск контейнеров за секунды
- Поддержка одиночных контейнеров и container groups
- Подходит для изолированных нагрузок
- Безопасность на уровне гипервизора

---

## Когда использовать ACI

✅ Простые приложения — один контейнер, без оркестрации  
✅ Автоматизация задач — batch-задачи  
✅ Build-задачи — этапы CI/CD  
✅ Dev/test — быстрое тестирование контейнеров  
✅ Event-driven нагрузки — обработка сообщений и событий

---

## Когда лучше выбрать AKS

❌ Нужна полноценная оркестрация (service discovery, auto-scaling)  
❌ Сложные приложения с несколькими зависимыми сервисами  
❌ Долгоживущие сервисы с высокой доступностью

---

# Преимущества ACI

| Возможность | Описание |
|-------------|----------|
| **Быстрый запуск** | Контейнеры стартуют за секунды |
| **Public IP + FQDN** | Публичный IP и DNS-имя |
| **Безопасность гипервизора** | Полная изоляция (как у VM) |
| **Гибкие размеры** | Точное указание CPU и памяти |
| **Постоянное хранилище** | Подключение Azure Files |
| **Linux и Windows** | Поддержка обеих ОС |
| **Без инфраструктуры** | Нет VM для управления |

---

# Container Groups

## Что такое Container Group?

**Группа контейнеров**, запущенных на одном хосте.

- Аналог Kubernetes pod
- Общий жизненный цикл
- Общие ресурсы (CPU, память)
- Общая сеть
- Общее хранилище
- Является основным ресурсом в ACI

---

## Когда использовать Container Groups

- Sidecar-контейнеры
- Логирование и мониторинг рядом с основным контейнером
- Контейнеры, которым нужно тесное взаимодействие
- Общий сетевой namespace

---

## Экзаменационный акцент (AZ-204)

- ACI — самый быстрый способ запустить контейнер
- Оплата посекундно
- Не требует VM или Kubernetes
- Container Group = аналог pod
- Подходит для простых и изолированных сценариев
- Для сложной оркестрации → AKS


### Container Group Architecture
```
Container Group (pod-like unit)
├── Container 1 (nginx)
│   └── Port 80 exposed
├── Container 2 (log collector)
│   └── Port 5000 exposed
├── Shared IP: 40.112.23.145
├── DNS: myapp.eastus.azurecontainer.io
├── Volume 1: Azure Files share → Container 1
└── Volume 2: Azure Files share → Container 2
```

## Key Characteristics (Основные характеристики Container Group)

✅ **Один хост** — все контейнеры размещаются на одной машине  
✅ **Общий IP** — один публичный IP для всей группы  
✅ **Общий жизненный цикл** — запускаются и останавливаются вместе  
✅ **Общая сеть** — контейнеры взаимодействуют через localhost  
✅ **Общие тома** — можно подключать volume к отдельным контейнерам

⚠️ **Только Linux** — multi-container группы поддерживаются только для Linux  
(Windows поддерживает только один контейнер в группе)

---

## Что это означает

- Контейнеры внутри группы тесно связаны
- Подходит для паттерна sidecar
- Невозможно масштабировать контейнеры внутри группы независимо
- В Windows ACI — только single-container сценарии

---

## Экзаменационный акцент (AZ-204)

- Container Group = аналог Kubernetes pod
- Один IP на группу
- Контейнеры общаются через localhost
- Multi-container поддерживается только для Linux


## Deployment Methods

### Azure CLI
```bash
# Single container
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image nginx \
  --ports 80 \
  --dns-name-label myapp \
  --location eastus
```

### Resource Manager Template
```json
{
  "type": "Microsoft.ContainerInstance/containerGroups",
  "apiVersion": "2021-09-01",
  "name": "mycontainergroup",
  "location": "eastus",
  "properties": {
    "containers": [...],
    "osType": "Linux"
  }
}
```

### YAML File (Recommended for Multi-Container)
```yaml
apiVersion: '2021-09-01'
location: eastus
name: mycontainergroup
properties:
  containers:
  - name: nginx
    properties:
      image: nginx
      ports:
      - port: 80
  osType: Linux
```

## Recommendation (Рекомендации по развертыванию)

- **YAML** — подходит для развёртывания только контейнеров
- **ARM template** — использовать, если вместе с контейнерами развёртываются другие ресурсы Azure

---

# Resource Allocation (Выделение ресурсов)

## Как распределяются ресурсы

Ресурсы рассчитываются как **сумма всех запросов контейнеров** в группе.

- CPU суммируется по всем контейнерам
- Память суммируется по всем контейнерам
- Оплата рассчитывается исходя из общего объёма ресурсов

---

## Что это означает

- Container Group получает общий пул CPU и памяти
- Нельзя задать разные VM-уровни внутри одной группы
- Если один контейнер использует много ресурсов — это влияет на всю группу
- Планируйте ресурсы с учётом всех контейнеров

---

## Экзаменационный акцент (AZ-204)

- Ресурсы выделяются на уровне Container Group
- Общие CPU и память = сумма всех контейнеров
- YAML — для контейнеров
- ARM — для комплексных инфраструктурных развертываний


```
Container Group CPU/Memory = Sum of all containers

Example:
- Container 1: 1 CPU, 1.5 GB memory
- Container 2: 0.5 CPU, 1 GB memory
- Total allocated: 1.5 CPU, 2.5 GB memory
```

### CPU and Memory Specs
```bash
# Specify resources
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp \
  --cpu 2 \
  --memory 4

# 2 CPU cores, 4 GB memory
```

### GPU Support (Preview)
```bash
# GPU-enabled container
az container create \
  --resource-group myResourceGroup \
  --name gpu-container \
  --image tensorflow/tensorflow:latest-gpu \
  --gpu-count 1 \
  --gpu-sku K80
```

# Networking в Azure Container Instances

## IP-адрес и порты

- **Один IP-адрес** — общий для всех контейнеров в группе
- **Общее пространство портов** — порты разделяются между контейнерами
- **Внешний доступ** — публикация портов через общий IP-адрес
- **Внутреннее взаимодействие** — через localhost

---

## Что это означает

- Контейнеры внутри группы могут обращаться друг к другу по `localhost:<port>`
- Нельзя использовать один и тот же порт в двух контейнерах группы
- Для внешнего доступа необходимо явно указать порт
- Публичный IP и FQDN назначаются на уровне всей группы

---

## Практические моменты

- Если контейнеры используют одинаковый порт — потребуется изменить конфигурацию
- Для внутренних sidecar-сценариев внешний IP может не требоваться
- Можно использовать Private IP в VNet-сценариях

---

## Экзаменационный акцент (AZ-204)

- Один IP на Container Group
- Общее пространство портов
- Внутренняя коммуникация через localhost
- Порты должны быть явно опубликованы для внешнего доступа


### Port Configuration
```bash
# Expose multiple ports
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp \
  --ports 80 443 8080
```

### DNS Name Label
```bash
# Get FQDN
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image nginx \
  --dns-name-label myapp \
  --location eastus

# FQDN: myapp.eastus.azurecontainer.io
```

⚠️ **Port mapping not supported** - Can't map host port 8080 to container port 80

### Localhost Communication
```yaml
# Container 1 can access Container 2 via localhost
apiVersion: '2021-09-01'
name: multi-container
properties:
  containers:
  - name: frontend
    properties:
      image: nginx
      ports:
      - port: 80
  - name: backend
    properties:
      image: api
      ports:
      - port: 5000  # Not exposed externally
      environmentVariables:
      - name: FRONTEND_URL
        value: http://localhost:80
  osType: Linux
```

# Storage Volumes в Azure Container Instances

## Поддерживаемые типы томов

| Тип тома | Описание | Сценарий использования |
|-----------|------------|------------------------|
| **Azure Files** | SMB-файловая шара | Постоянное хранение данных |
| **Secret** | Чувствительные данные | Пароли, ключи, токены |
| **Empty directory** | Временное хранилище | Scratch space, временные файлы |
| **Git repo** | Клонирование репозитория | Исходный код приложения |

---

## Что важно понимать

- Volume подключается к конкретному контейнеру в группе.
- Azure Files обеспечивает персистентность данных.
- Secret volume хранится в памяти (не сохраняется на диск).
- Empty directory существует только в рамках жизненного цикла группы.
- Git repo автоматически клонируется при старте контейнера.

---

## Когда использовать

- **Azure Files** → данные должны сохраниться после перезапуска
- **Secret** → конфиденциальные параметры
- **Empty directory** → временные вычисления
- **Git repo** → быстрый запуск контейнера с исходным кодом

---

## Экзаменационный акцент (AZ-204)

- Для постоянного хранения → Azure Files
- Для секретов → Secret volume
- Для временных данных → Empty directory
- Volume настраивается на уровне Container Group


### Volume Example
```bash
# Mount Azure Files share
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp \
  --azure-file-volume-account-name mystorageaccount \
  --azure-file-volume-account-key <key> \
  --azure-file-volume-share-name myshare \
  --azure-file-volume-mount-path /data
```

## Common Scenarios

### 1. Web App + Content Puller
```
- Container 1: Nginx (serve web app)
- Container 2: Git sync (pull latest content)
```

### 2. App + Logging Sidecar
```
- Container 1: Application (write logs)
- Container 2: Log collector (ship logs to Azure Monitor)
```

### 3. App + Monitoring Sidecar
```
- Container 1: Application
- Container 2: Health checker (monitor and alert)
```

### 4. Front-End + Back-End
```
- Container 1: Web UI (public port 80)
- Container 2: API service (localhost port 5000)
```

## CLI Commands

### Create Container
```bash
# Basic container
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image nginx \
  --ports 80

# With DNS and environment variables
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myapp:v1.0 \
  --dns-name-label myapp \
  --ports 80 443 \
  --environment-variables 'API_URL'='https://api.example.com' \
  --cpu 2 \
  --memory 4
```

### List Containers
```bash
# List container groups
az container list \
  --resource-group myResourceGroup \
  --output table

# Show container details
az container show \
  --resource-group myResourceGroup \
  --name mycontainer \
  --output json
```

### Container Logs
```bash
# View logs
az container logs \
  --resource-group myResourceGroup \
  --name mycontainer

# Follow logs
az container attach \
  --resource-group myResourceGroup \
  --name mycontainer
```

### Container State
```bash
# Start container
az container start \
  --resource-group myResourceGroup \
  --name mycontainer

# Stop container
az container stop \
  --resource-group myResourceGroup \
  --name mycontainer

# Restart container
az container restart \
  --resource-group myResourceGroup \
  --name mycontainer
```

### Delete Container
```bash
# Delete container group
az container delete \
  --resource-group myResourceGroup \
  --name mycontainer \
  --yes
```

### Exec Into Container
```bash
# Run command in container
az container exec \
  --resource-group myResourceGroup \
  --name mycontainer \
  --exec-command "/bin/bash"
```

## Pricing

### Billing Model
**Per-second billing** for CPU and memory:

```
Cost = (CPU cores × CPU price × seconds) + (GB memory × memory price × seconds)

Example (East US):
- CPU: $0.0000012/vCPU/second
- Memory: $0.0000001/GB/second

2 vCPU, 4 GB, 1 hour:
= (2 × $0.0000012 × 3600) + (4 × $0.0000001 × 3600)
= $0.01 per hour
```

# Cost Optimization (Оптимизация затрат)

✅ **Останавливайте контейнеры, когда они не нужны** — оплата только во время работы  
✅ **Правильно подбирайте ресурсы** — не выделяйте лишние CPU и память  
✅ **Restart policy** — используйте `OnFailure` или `Never` для batch-задач  
✅ **Spot containers (Preview)** — сниженная стоимость для прерываемых нагрузок

---

# Critical Notes (Критически важные моменты)

- 💡 **Самый быстрый запуск** — контейнеры стартуют за секунды без VM
- ⚠️ **Container groups** — аналог Kubernetes pod (общий хост, сеть, жизненный цикл)
- 🎯 **Multi-container группы** — только Linux (Windows поддерживает только один контейнер)
- ✅ **Оплата посекундно** — платите только за время работы
- 📊 **Распределение ресурсов** — сумма CPU и памяти всех контейнеров
- 🔄 **Общее пространство портов** — нет port mapping внутри группы
- 🔒 **Связь через localhost** — контейнеры взаимодействуют через localhost
- ⚠️ **Для оркестрации используйте AKS** — ACI подходит для простых, изолированных сценариев

---

# Exam Tips (AZ-204)

- ACI — самый быстрый способ запустить контейнер в Azure (секунды, без VM)
- Container Group — основной ресурс (аналог pod)
- Multi-container → только Linux
- Windows → только один контейнер

---

## Особенности Container Group

- Один IP-адрес
- Общий жизненный цикл
- Общая сеть
- Общие тома хранения

---

## Ресурсы

- CPU и память = сумма всех контейнеров
- Оплата посекундная за CPU и память

---

## Сеть

- Один IP на группу
- Общее пространство портов
- Взаимодействие через localhost

---

## Хранилище

- Azure Files — постоянное
- Secret — конфиденциальные данные
- Empty directory — временное
- Git repo — клонирование репозитория

---

## Развертывание

- CLI
- ARM template
- YAML (для container-only сценариев)

---

## Когда использовать ACI

- Простые приложения
- Batch-задачи
- Build-задачи
- Dev/test

---

## Когда использовать AKS

- Полноценная оркестрация
- Service discovery
- Auto-scaling
- Сложные микросервисные системы

---

## DNS

- Параметр `--dns-name-label` создаёт FQDN вида:  
  `name.region.azurecontainer.io`

---

## Частые экзаменационные ловушки

- «Нужен быстрый запуск без VM» → ACI
- «Нужна оркестрация и auto-scaling» → AKS
- «Несколько контейнеров вместе» → Container Group (Linux)
- «Оплата только за время выполнения» → ACI


[Learn More](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/2-azure-container-instances-overview)
