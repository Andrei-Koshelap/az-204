# Azure Container Apps — Обзор

## Ключевые понятия (Key Concepts)

- **Container Apps** — serverless-платформа для микросервисов и контейнеров
- **Построена на AKS** — работает поверх Azure Kubernetes Service
- **Container Apps environment** — изолированная среда для размещения приложений
- **Интеграция с Dapr** — нативная поддержка распределённых приложений

---

# Что такое Azure Container Apps?

**Serverless-платформа** для запуска микросервисов и контейнеризированных приложений.

- Работает поверх Azure Kubernetes Service (AKS)
- Не требует знаний Kubernetes
- Полностью управляемая оркестрация
- Автоматическое масштабирование (включая scale to zero)
- Встроенный load balancing и распределение трафика

---

# Сравнение с другими сервисами контейнеров Azure

| Сервис | Сценарий | Оркестрация | Масштабирование |
|----------|------------|---------------|------------------|
| **Container Apps** | Микросервисы, API, event-driven | Управляемая (скрыта) | Авто (включая до нуля) |
| **Container Instances** | Простые контейнеры, batch | Отсутствует | Ручное |
| **AKS** | Полный контроль Kubernetes | Самостоятельное управление | Manual / HPA |
| **App Service** | Web-приложения, API | Встроенная | Авто |

---

# Типовые сценарии использования

✅ API endpoints — REST, GraphQL  
✅ Background processing — обработчики очередей, планировщики  
✅ Event-driven обработка — реакция на события Event Grid, Service Bus  
✅ Микросервисы — распределённые системы с service discovery

---

## Примеры сценариев

- Обработка сообщений из очереди с масштабированием через KEDA
- Развёртывание нескольких версий API (A/B тестирование)
- Batch-задачи с масштабированием до нуля при простое
- Event-driven workflow с использованием Dapr

---

# Ключевые возможности

## Динамическое масштабирование

Масштабирование на основе различных триггеров:

| Триггер | Описание |
|----------|------------|
| **HTTP-трафик** | Масштабирование по количеству запросов |
| **CPU/Memory** | Масштабирование по нагрузке |
| **Event-driven** | KEDA-scalers (очереди, топики и т.д.) |
| **Custom metrics** | Любой scaler, поддерживаемый KEDA |

⚠️ Приложения, масштабируемые по CPU/Memory, **не могут масштабироваться до нуля**

---

## Экзаменационный акцент (AZ-204)

- Container Apps — serverless поверх AKS
- Не требует управления Kubernetes
- Поддерживает auto-scaling и scale to zero
- Подходит для микросервисов и event-driven архитектуры
- CPU-based scaling ≠ scale to zero


### Application Lifecycle Management
```
Create → Deploy → Update → Create Revision → Traffic Split → Deactivate
```

## Revisions (Версионирование приложений)

- **Multiple revisions** — можно держать несколько версий приложения одновременно
- **Traffic splitting** — распределение трафика (Blue/Green, A/B тестирование)
- **Rollback** — активация предыдущей версии
- **Revision history** — история всех развертываний

---

## HTTPS Ingress

✅ **Без настройки инфраструктуры** — автоматический HTTPS endpoint  
✅ **Custom domains** — подключение собственного домена  
✅ **Certificates** — управляемые или собственные сертификаты  
✅ **Internal / External** — публичные или приватные endpoints

---

## Что это означает

- Можно безопасно деплоить новую версию без остановки старой
- Возможна постепенная подача трафика на новую ревизию
- Поддержка production-grade HTTPS из коробки
- Подходит для микросервисной архитектуры

---

## Экзаменационный акцент (AZ-204)

- Container Apps поддерживает revision-based deployment
- Поддержка Blue/Green и A/B тестирования
- Встроенный HTTPS ingress
- Возможность rollback без повторного деплоя


### Traffic Management
```
Production Traffic
├── 80% → Revision v1.2 (stable)
└── 20% → Revision v2.0 (canary)
```

## Use Cases (Сценарии использования ревизий)

- Blue/Green deployment
- A/B тестирование
- Canary-релизы
- Постепенный rollout

---

# Service Discovery

✅ **Встроенный DNS** — внутренние вызовы между сервисами  
✅ **Интеграция с Dapr** — service invocation с mTLS  
✅ **Без внешнего IP** — можно создавать только внутренние endpoints

---

# Поддержка микросервисов

- **Независимое масштабирование** — каждый сервис масштабируется отдельно
- **Независимое версионирование** — обновления деплоятся независимо
- **Service discovery** — автоматическое обнаружение сервисов
- **Интеграция с Dapr** — API для state management, pub/sub, bindings

---

# Мониторинг и логирование

✅ **Azure Log Analytics** — централизованный сбор логов  
✅ **Application Insights** — распределённая трассировка  
✅ **Console logs** — просмотр вывода контейнера  
✅ **Метрики** — CPU, память, количество запросов

---

# Container Apps Environments

## Что такое Environment?

**Изолированная безопасная граница** для группы Container Apps.

- Развёртывается в одной виртуальной сети
- Использует общий Log Analytics workspace
- Изолирован от других environments
- Поддерживает кастомную VNet

---

## Что важно понимать

- Environment — логическая граница безопасности
- Несколько приложений могут работать в одном environment
- Environment определяет сетевую конфигурацию
- Подходит для разделения dev/test/prod

---

## Экзаменационный акцент (AZ-204)

- Container Apps поддерживает service discovery
- Dapr обеспечивает mTLS и межсервисное взаимодействие
- Каждое приложение масштабируется независимо
- Environment = безопасная граница для группы приложений
- Log Analytics подключается на уровне environment


### Environment Architecture
```
Container Apps Environment
├── VNet: 10.0.0.0/16
├── Log Analytics: shared-workspace
├── Container App 1: frontend
├── Container App 2: backend-api
└── Container App 3: queue-worker
```

## Когда использовать один и тот же Environment

Развёртывайте приложения в **одном environment**, если:

✅ Управляете связанными сервисами (например, frontend + backend)  
✅ Сервисы должны взаимодействовать через Dapr  
✅ Требуется общая виртуальная сеть  
✅ Нужна общая конфигурация Dapr  
✅ Используется один Log Analytics workspace

---

## Когда использовать разные Environments

Развёртывайте приложения в **разных environments**, если:

❌ Сервисы не должны разделять вычислительные ресурсы  
❌ Нужно изолировать production от development  
❌ Требуется запретить Dapr service invocation между приложениями  
❌ Различаются требования к сетевой конфигурации

---

## Что важно понимать

- Environment — это граница изоляции и сетевой конфигурации
- Внутри одного environment приложения могут взаимодействовать
- Разные environments обеспечивают более строгую изоляцию

---

## Экзаменационный акцент (AZ-204)

- Связанные микросервисы → один environment
- Dev/Test/Prod → разные environments
- Требуется полная изоляция → разные environments
- Dapr service-to-service работает внутри одного environment


### Example: Multi-Tier App (Same Environment)
```yaml
Environment: production-env
├── frontend-app (external ingress)
├── api-app (internal ingress)
├── worker-app (no ingress)
└── Shared: VNet, logging, Dapr
```

### Example: Environment Separation
```yaml
Environment: production-env
├── prod-frontend
└── prod-api

Environment: staging-env
├── staging-frontend
└── staging-api

Environment: development-env
├── dev-frontend
└── dev-api
```

# Microservices with Azure Container Apps

## Возможности для микросервисной архитектуры

| Возможность | Преимущество |
|--------------|--------------|
| **Независимое масштабирование** | Каждый сервис масштабируется отдельно |
| **Независимое версионирование** | Обновление сервисов без влияния на другие |
| **Service discovery** | Обнаружение сервисов через встроенный DNS |
| **Интеграция с Dapr** | Service invocation, pub/sub, управление состоянием |

---

## Что это означает на практике

- Можно масштабировать только нагруженный сервис
- Разные версии сервисов могут работать параллельно
- Сервисы находят друг друга без ручной настройки IP
- Dapr упрощает построение распределённых систем

---

## Экзаменационный акцент (AZ-204)

- Container Apps подходит для микросервисов
- Каждый сервис масштабируется независимо
- Встроенный DNS обеспечивает service discovery
- Dapr добавляет pub/sub, state management и mTLS


### Microservices Architecture Example
```
Container Apps Environment
├── API Gateway (external, Dapr-enabled)
│   └── Routes to internal services
├── Order Service (internal, Dapr-enabled)
│   └── Listens to orders queue
├── Payment Service (internal, Dapr-enabled)
│   └── Calls external payment API
├── Notification Service (internal, Dapr-enabled)
│   └── Sends emails via Dapr binding
└── Shared: Service discovery, pub/sub, state store
```

# Dapr Integration в Azure Container Apps

## Что такое Dapr?

**Distributed Application Runtime (Dapr)** — фреймворк для построения микросервисных систем.

- Проект Cloud Native Computing Foundation (CNCF)
- Предоставляет готовые building blocks для распределённых приложений
- Использует sidecar-архитектуру
- Языконезависимые API

---

## Зачем использовать Dapr с Container Apps?

✅ **Управляемый сервис** — Azure автоматически управляет установкой и обновлениями  
✅ **Упрощение** — не требуется самостоятельная установка Dapr  
✅ **Интеграция** — включается простой настройкой  
✅ **Безопасность** — автоматический mTLS между сервисами

---

## Возможности Dapr

- **Service invocation** — вызов сервисов с mTLS и retry
- **State management** — распределённое хранение состояния с поддержкой транзакций
- **Pub/Sub** — публикация и подписка на события
- **Bindings** — интеграция с внешними системами (очереди, cron, события)
- **Observability** — распределённая трассировка
- **Secrets** — безопасное управление секретами

---

## Что важно понимать

- Dapr работает как sidecar рядом с приложением
- Позволяет писать код без привязки к конкретной инфраструктуре
- Упрощает реализацию event-driven архитектуры
- Поддерживает микросервисные паттерны без сложной настройки Kubernetes

---

## Экзаменационный акцент (AZ-204)

- Dapr — CNCF-проект для распределённых приложений
- В Container Apps Dapr управляется Azure
- Поддержка mTLS между сервисами
- Предоставляет building blocks: service invocation, pub/sub, state
- Работает по sidecar-модели

### Example: Dapr Service Invocation
```
Frontend App → Dapr sidecar → HTTP/gRPC → Dapr sidecar → Backend App
             (mTLS, retries, tracing)
```

# Container Registry Support в Azure Container Apps

## Поддержка реестров контейнеров

✅ **Публичные реестры** — Docker Hub, Microsoft Container Registry (MCR)  
✅ **Azure Container Registry (ACR)** — поддержка Managed Identity  
✅ **Приватные реестры** — любой registry с учётными данными

---

## Что важно понимать

- Container Apps может получать образы из публичных и приватных реестров
- Для ACR рекомендуется использовать **Managed Identity**, а не логин/пароль
- Для сторонних приватных реестров требуется указать credentials
- Интеграция с ACR наиболее безопасна и удобна в Azure-сценариях

---

## Экзаменационный акцент (AZ-204)

- Поддерживаются Docker Hub и MCR
- Для ACR → предпочтительно Managed Identity
- Для приватных registry → требуется аутентификация
- Безопасность > хранение логина и пароля

## CLI Commands

### Create Environment
```bash
# Create Container Apps environment
az containerapp env create \
  --name myenvironment \
  --resource-group myResourceGroup \
  --location eastus
```

### Create Container App
```bash
# Create container app
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myregistry.azurecr.io/myapp:v1.0 \
  --target-port 80 \
  --ingress external \
  --min-replicas 0 \
  --max-replicas 10

# App URL: https://myapp.{env-id}.{region}.azurecontainerapps.io
```

### Enable Dapr
```bash
# Create app with Dapr enabled
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myapp:latest \
  --enable-dapr \
  --dapr-app-id myapp \
  --dapr-app-port 3000
```

### List Container Apps
```bash
# List apps in environment
az containerapp list \
  --environment myenvironment \
  --resource-group myResourceGroup \
  --output table
```

### View Logs
```bash
# Stream logs
az containerapp logs show \
  --name myapp \
  --resource-group myResourceGroup \
  --follow

# View specific revision logs
az containerapp revision show \
  --name myapp \
  --resource-group myResourceGroup \
  --revision myapp--revision1
```

## Scaling Configuration

### HTTP Scaling
```bash
# Scale based on HTTP requests
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myapp:latest \
  --min-replicas 1 \
  --max-replicas 10 \
  --scale-rule-name http-rule \
  --scale-rule-http-concurrency 100
```

### KEDA Scaling (Azure Queue)
```bash
# Scale based on queue length
az containerapp create \
  --name queue-processor \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image processor:latest \
  --min-replicas 0 \
  --max-replicas 30 \
  --scale-rule-name queue-rule \
  --scale-rule-type azure-queue \
  --scale-rule-metadata \
    "queueName=messages" \
    "queueLength=10" \
  --scale-rule-auth connection=queue-connection

# Scales to 0 when queue is empty
```

## Ingress Configuration

### External Ingress (Public)
```bash
# Public endpoint
az containerapp ingress enable \
  --name myapp \
  --resource-group myResourceGroup \
  --type external \
  --target-port 80 \
  --transport http

# Public URL created
```

### Internal Ingress (Private)
```bash
# Internal-only endpoint
az containerapp ingress enable \
  --name internal-api \
  --resource-group myResourceGroup \
  --type internal \
  --target-port 8080 \
  --transport http

# Only accessible within environment
```

## Virtual Network Integration

### Custom VNet
```bash
# Create environment with custom VNet
az containerapp env create \
  --name myenvironment \
  --resource-group myResourceGroup \
  --location eastus \
  --infrastructure-subnet-resource-id /subscriptions/{sub-id}/resourceGroups/{rg}/providers/Microsoft.Network/virtualNetworks/{vnet}/subnets/{subnet}
```

# Critical Notes — Azure Container Apps

- 💡 **Serverless** — работает поверх AKS, без управления кластером
- ⚠️ **Environment** — изолированная граница для связанных Container Apps
- 🎯 **Масштабирование** — auto-scale до нуля (кроме CPU/Memory-based)
- ✅ **Revisions** — неизменяемые снимки для версионирования
- 📊 **Traffic splitting** — Blue/Green, A/B, Canary
- 🔄 **Dapr** — управляемая интеграция для микросервисов
- 🔒 **Ingress** — внешний (public) или внутренний (private)
- ⚠️ **KEDA** — поддержка любых KEDA-scalers для event-driven масштабирования

---

# Exam Tips (AZ-204)

## Основы

- Container Apps — serverless-платформа на базе AKS
- Подходит для: API, микросервисов, event-driven задач, background processing

---

## Environment

- Безопасная граница изоляции
- Общая VNet
- Общий Log Analytics
- Общая конфигурация Dapr

### Один environment
- Связанные сервисы
- Общая сеть
- Service-to-service вызовы
- Общая телеметрия

### Разные environments
- Изоляция dev/test/prod
- Нет общих ресурсов
- Разные сетевые требования

---

## Масштабирование

- HTTP-трафик
- CPU/Memory
- Event-driven (через KEDA)

⚠️ Scale to zero поддерживается для всех триггеров, кроме CPU/Memory

---

## Revisions

- Immutable snapshots
- Traffic splitting
- Rollback
- Поддержка Blue/Green, A/B, Canary

---

## Ingress

- External — публичный доступ
- Internal — только внутри environment

---

## Service Discovery

- Встроенный DNS
- Dapr service invocation

---

## Dapr

- Управляется Azure
- Включается флагом
- Sidecar-архитектура
- Pub/Sub, state management, mTLS

---

## Monitoring

- Log Analytics
- Application Insights
- Console logs

---

## Container Registry

- Публичные registry
- Azure Container Registry (рекомендуется Managed Identity)
- Приватные registry (с учётными данными)

---

## Сеть

- Можно подключить кастомную VNet к environment

---

## Частые экзаменационные ловушки

- Нужен scale to zero → Container Apps (не CPU-based scaling)
- Нужна полноценная Kubernetes-оркестрация → AKS
- Нужен serverless для микросервисов → Container Apps
- Нужна изоляция → разные environments


az aks get-credentials --resource-group <rg> --name <cluster>

az aks get-credentials
kubectl apply

[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/2-explore-azure-container-apps)

1️⃣ Log Streaming
📌 Что это
Просмотр логов контейнера в реальном времени.
📌 Что показывает
stdout
stderr
системные сообщения
логи приложения

📌 Когда использовать
Проверка после деплоя
Debug “прямо сейчас”
Проверка, стартовал ли контейнер

az containerapp logs show --follow

2️⃣ Log Analytics (Azure Monitor Logs)
📌 Что это
Централизованное хранилище логов с KQL-запросами.
📌 Что позволяет
Фильтрацию
Агрегацию
Исторический анализ
Поиск по времени
Создание алертов

📌 Когда использовать
Анализ за прошлые часы/дни
Поиск ошибок
Метрики и отчёты
👉 Это не live streaming.

3️⃣ Container Console
📌 Что это
Интерактивный shell в контейнере.
📌 Что можно делать
Запустить bash
Проверить файлы
Выполнить команды
📌 Это НЕ
Не лог-сервис
Не мниторинг
👉 Это для ручной диагностики.

4️⃣ Diagnostic Logs
📌 Что это
Механизм отправки логов в:
Log Analytics
Storage Account
Event Hub
📌 Это
Конфигурация маршрутизации логов
Не инструмент просмотра