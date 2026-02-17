# Azure Container Registry (ACR) — Обзор

## Ключевые понятия (Key Concepts)

- **ACR** — управляемый сервис Docker Registry 2.0 в Azure
- **Private registry** — приватное хранилище контейнерных образов
- **Интеграция** — работает с существующими CI/CD-пайплайнами
- **ACR Tasks** — сборка образов в Azure без локального Docker

---

# Что такое Azure Container Registry?

**Управляемый сервис реестра контейнеров** для хранения и управления образами.

- Основан на open-source **Docker Registry 2.0**
- Приватный registry, размещённый в Azure
- Хранит контейнерные образы и связанные артефакты
- Интегрируется с Azure-сервисами и оркестраторами
- Не требует локального Docker Engine (при использовании ACR Tasks)

---

# Сценарии использования

## Цели развертывания (Pull образов)

| Цель | Описание |
|------|----------|
| **Kubernetes** | AKS, DC/OS, Docker Swarm |
| **Azure App Service** | Развёртывание контейнеризированных веб-приложений |
| **Azure Batch** | Запуск batch-нагрузок |
| **Service Fabric** | Платформа для микросервисов |

---

## Процесс разработки (Push образов)

Интеграция с CI/CD:

- Azure Pipelines
- Jenkins
- GitHub Actions
- GitLab CI/CD

---

## Автоматизация

- Пересборка образов при обновлении base image
- Сборка образов при Git-коммите
- Многошаговые процессы: build → test → patch

---

# Тарифные планы (Service Tiers)

| Tier | Хранилище | Пропускная способность | Сценарий |
|------|------------|------------------------|----------|
| **Basic** | 10 GB | Низкая | Обучение, dev-среда |
| **Standard** | 100 GB | Средняя | Большинство production-сценариев |
| **Premium** | 500 GB | Высокая | Высокая нагрузка, geo-replication |

---

# Возможности по уровням

## Доступно во всех планах

✅ Аутентификация через Microsoft Entra ID  
✅ Удаление образов  
✅ Webhooks  
✅ Полный программный API

---

## Только в Premium

🎯 **Geo-replication** — один registry в нескольких регионах  
🎯 **Content trust** — подпись тегов образов  
🎯 **Private Link** — приватные endpoint’ы  
🎯 **Zone redundancy** — поддержка availability zones

---

# Поддерживаемый контент

## Типы образов

- **Docker-контейнеры** (Windows и Linux)
- **Helm charts**
- **OCI-образы** (Open Container Initiative)
- **Связанные артефакты** (конфигурации, шаблоны и т.д.)

---

## Экзаменационный акцент (AZ-204)

- ACR — приватный registry в Azure
- Основан на Docker Registry 2.0
- Интеграция с AKS и CI/CD
- ACR Tasks — сборка без локального Docker
- Geo-replication и Private Link — только Premium
- Используется для хранения и распространения контейнерных образов

### Repository Organization
```
myregistry.azurecr.io/
├── webapp/frontend:v1.0
├── webapp/backend:v1.0
├── webapp/backend:v1.1
├── api/orders:latest
└── api/payments:stable
```

## CLI Commands

### Create Registry
```bash
# Create resource group
az group create --name myResourceGroup --location eastus

# Create ACR (Basic tier)
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Basic

# Create Premium registry (for geo-replication)
az acr create \
  --resource-group myResourceGroup \
  --name mypremiumregistry \
  --sku Premium
```

### Login to Registry
```bash
# Login with Azure CLI
az acr login --name myregistry

# Get admin credentials (not recommended for production)
az acr credential show --name myregistry

# Docker login using admin credentials
docker login myregistry.azurecr.io
```

### Push/Pull Images
```bash
# Tag local image
docker tag myapp:latest myregistry.azurecr.io/myapp:v1.0

# Push to ACR
docker push myregistry.azurecr.io/myapp:v1.0

# Pull from ACR
docker pull myregistry.azurecr.io/myapp:v1.0
```

### List Images
```bash
# List repositories
az acr repository list --name myregistry --output table

# List tags for a repository
az acr repository show-tags \
  --name myregistry \
  --repository myapp \
  --output table

# Show image manifest
az acr repository show \
  --name myregistry \
  --image myapp:v1.0
```

### Delete Images
```bash
# Delete image tag
az acr repository delete \
  --name myregistry \
  --image myapp:v1.0

# Delete repository
az acr repository delete \
  --name myregistry \
  --repository myapp
```

## Authentication Methods

### Azure CLI (Recommended)
```bash
az acr login --name myregistry
# Token valid for 3 hours
```

### Service Principal
```bash
# Create service principal
az ad sp create-for-rbac \
  --name acr-service-principal \
  --scopes /subscriptions/<subscription-id>/resourceGroups/<rg>/providers/Microsoft.ContainerRegistry/registries/<registry> \
  --role acrpull

# Use in Docker login
docker login myregistry.azurecr.io \
  --username <appId> \
  --password <password>
```

### Managed Identity
```bash
# Assign AcrPull role to managed identity
az role assignment create \
  --assignee <managed-identity-id> \
  --scope /subscriptions/<subscription-id>/resourceGroups/<rg>/providers/Microsoft.ContainerRegistry/registries/<registry> \
  --role AcrPull
```

### Admin Account (Not Recommended)
```bash
# Enable admin account
az acr update --name myregistry --admin-enabled true

# Get credentials
az acr credential show --name myregistry
```

⚠️ **Production**: Используйте service principal или managed identity, а не admin-аккаунт

---

# RBAC-роли в Azure Container Registry

| Роль | Описание | Разрешения |
|------|-----------|------------|
| **AcrPull** | Получение (pull) образов | Только чтение |
| **AcrPush** | Получение и публикация (pull + push) | Чтение и запись образов |
| **AcrDelete** | Удаление образов | Удаление образов и репозиториев |
| **Owner** | Полный доступ | Все операции |

---

## Практические рекомендации

- Для продакшена:
    - Используйте **Managed Identity** (для Azure сервисов)
    - Или **Service Principal** (для CI/CD)
- Не используйте admin account в production.
- Применяйте принцип **минимально необходимых прав (least privilege)**.

---

## Типовые сценарии

- AKS или App Service → **AcrPull**
- CI/CD pipeline → **AcrPush**
- Очистка старых образов → **AcrDelete**
- Полный контроль → **Owner**

---

## Экзаменационный акцент (AZ-204)

- Для production → Managed Identity или Service Principal
- AcrPull → только загрузка образов
- AcrPush → загрузка и публикация
- AcrDelete → удаление
- Admin account → не рекомендуется для production


## Integration with Azure Services

### Azure Kubernetes Service (AKS)
```bash
# Attach ACR to AKS
az aks update \
  --name myAKSCluster \
  --resource-group myResourceGroup \
  --attach-acr myregistry

# Deploy from ACR in Kubernetes manifest
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
spec:
  template:
    spec:
      containers:
      - name: myapp
        image: myregistry.azurecr.io/myapp:v1.0
```

### Azure App Service
```bash
# Create web app with container
az webapp create \
  --resource-group myResourceGroup \
  --plan myAppServicePlan \
  --name mywebapp \
  --deployment-container-image-name myregistry.azurecr.io/myapp:v1.0
```

### Azure Container Instances
```bash
# Create container instance
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image myregistry.azurecr.io/myapp:v1.0 \
  --registry-login-server myregistry.azurecr.io \
  --registry-username <username> \
  --registry-password <password>
```

# Critical Notes — Azure Container Registry

- 💡 **Private registry** — безопасная альтернатива Docker Hub
- ⚠️ **Admin account** — не рекомендуется для production
- 🎯 **Service tiers** — выбираются исходя из объёма хранения, пропускной способности и нужных функций
- ✅ **Premium** — обязателен для geo-replication, content trust и private link
- 🔒 **Аутентификация** — используйте service principal или managed identity
- 📊 **Интеграция** — работает с AKS, App Service, ACI
- 🔄 **ACR Tasks** — сборка образов в облаке без локального Docker Engine

---

# Exam Tips (AZ-204)

- ACR — управляемый Docker Registry 2.0 сервис
- Три уровня:
    - **Basic** — dev
    - **Standard** — production
    - **Premium** — расширенные возможности

---

## Premium-функции

- Geo-replication
- Content trust
- Private Link
- Zone redundancy

---

## Доступно во всех планах

- Аутентификация через Microsoft Entra ID
- Удаление образов
- Webhooks

---

## Аутентификация

- Azure CLI (временный токен ~3 часа)
- Service Principal
- Managed Identity
- Admin account (не использовать в production)

---

## RBAC-роли

- **AcrPull** — чтение
- **AcrPush** — чтение и запись
- **AcrDelete** — удаление
- **Owner** — полный доступ

---

## Интеграция

- AKS
- App Service
- Azure Container Instances (ACI)
- Azure Batch
- Service Fabric

---

## ACR Tasks

- Сборка образов в облаке
- Не требуется локальный Docker
- Поддержка автоматических триггеров

---

## Формат имени репозитория
    registry.azurecr.io/repository:tag
---

## Частые экзаменационные ловушки

- «Нужна geo-replication» → Premium
- «Production authentication» → Managed Identity или Service Principal
- «Private registry в Azure» → ACR
- «Сборка без локального Docker» → ACR Tasks


[Learn More](https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/2-azure-container-registry-overview)
