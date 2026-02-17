# Azure Container Registry Tasks (ACR Tasks)

## Ключевые понятия (Key Concepts)

- **ACR Tasks** — сборка контейнерных образов в облаке Azure
- **Quick task** — разовая сборка образа по требованию (`az acr build`)
- **Automated triggers** — автоматический запуск по Git-коммиту, обновлению base image или расписанию
- **Multi-step tasks** — сложные workflow из нескольких шагов

---

# Что такое ACR Tasks?

**Облачная сборка контейнерных образов** без использования локального Docker.

- Сборка образов напрямую в Azure
- Не требуется локальный Docker Engine
- Поддержка автоматизированных build-пайплайнов
- Поддержка разных платформ (Linux, Windows, ARM)
- Интеграция с CI/CD процессами

---

## Преимущества

✅ **Без локального Docker** — сборка полностью в облаке  
✅ **Автоматизация** — автоматический запуск сборки  
✅ **CI/CD интеграция** — часть DevOps-процесса  
✅ **Поддержка платформ** — Linux, Windows, ARM  
✅ **Многошаговые сценарии** — build → test → push

---

## Когда использовать ACR Tasks

- Нужна облачная сборка без локальной инфраструктуры
- Требуется автоматическая пересборка при изменении base image
- Необходимо встроить сборку в Azure-native workflow
- Нужна поддержка multi-platform build

---

## Экзаменационный акцент (AZ-204)

- ACR Tasks позволяют собирать образы без локального Docker
- Поддерживают автоматические триггеры
- Интегрируются с CI/CD
- Поддерживают multi-step workflow
- Используются вместе с Azure Container Registry

## Task Scenarios

### 1. Quick Task
**On-demand build and push** without local Docker:

```bash
# Build image from Dockerfile and push to registry
az acr build \
  --registry myregistry \
  --image myapp:v1.0 \
  --file Dockerfile \
  .

# Think: "docker build + docker push" in the cloud
```

## Use Cases (Сценарии использования ACR Tasks)

- Быстрая сборка образов без локального Docker
- Сборка образов в рамках CI/CD pipeline
- Проверка Dockerfile перед коммитом
- Inner-loop разработка (быстрая итерация и тестирование изменений)

---

## Когда это особенно полезно

- Разработчики работают без установленного Docker
- Нужно быстро протестировать изменения в Dockerfile
- Требуется централизованная сборка в облаке
- Необходимо автоматизировать процесс build → test → push

---

## Экзаменационный акцент (AZ-204)

Если в вопросе говорится:
- «собрать образ без локального Docker»
- «автоматическая сборка по Git-коммиту»
- «пересборка при обновлении base image»

→ правильный ответ: **ACR Tasks**.


### 2. Trigger on Source Code Update
**Automatic builds on Git commit**:

```bash
# Create task triggered by Git commit
az acr task create \
  --registry myregistry \
  --name build-on-commit \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/myorg/myrepo.git \
  --file Dockerfile \
  --git-access-token $(cat token.txt) \
  --commit-trigger-enabled true

# Now: Every commit triggers a build
```
## Triggers (Триггеры ACR Tasks)

### Поддерживаемые события запуска

- Коммит кода в ветку
- Создание или обновление Pull Request
- Совпадение с конкретной веткой или шаблоном тега

---

## Trigger on Base Image Update

**Автоматическая пересборка приложения при обновлении базового образа**

- Отслеживание изменений в base image
- Автоматический запуск сборки при появлении новой версии
- Обновление зависимых образов без ручного вмешательства
- Повышение безопасности (получение последних патчей)

---

## Зачем это нужно

- Автоматическое применение security-патчей
- Минимизация уязвимостей
- Поддержание актуальности зависимостей
- Упрощение DevSecOps-процессов

---

## Экзаменационный акцент (AZ-204)

Если требуется:
- «пересобрать образ при обновлении базового образа»
- «автоматически применять security-обновления»

→ правильный ответ: **ACR Task с триггером на base image update**.


```bash
# Create task that watches base image
az acr task create \
  --registry myregistry \
  --name rebuild-on-base \
  --image myapp:latest \
  --context https://github.com/myorg/myrepo.git \
  --file Dockerfile \
  --base-image-trigger-enabled true \
  --base-image-trigger-name mybaseimage

# Automatic rebuild when:
# - Base image updated in ACR
# - Base image updated in Docker Hub
```

**Scenario**:
```dockerfile
# Your Dockerfile
FROM myregistry.azurecr.io/baseimage:latest
COPY . /app
CMD ["./app"]

# When baseimage:latest is updated → automatic rebuild
```

### 4. Schedule a Task
**Run builds on a schedule**:

```bash
# Create scheduled task (cron format)
az acr task create \
  --registry myregistry \
  --name nightly-build \
  --image myapp:nightly-{{.Run.Date}} \
  --context https://github.com/myorg/myrepo.git \
  --file Dockerfile \
  --schedule "0 2 * * *"  # 2 AM daily

# Examples:
# "0 2 * * *"      - Daily at 2 AM
# "0 2 * * 0"      - Weekly on Sunday at 2 AM
# "0 */4 * * *"    - Every 4 hours
# "0 2 1 * *"      - Monthly on 1st at 2 AM
```

## Use Cases (Сценарии использования по расписанию)

- Ночные сборки (nightly builds)
- Регулярные задачи обслуживания
- Плановое сканирование образов
- Периодические тестовые прогоны

---

## Multi-Step Tasks

### Сложные workflow с несколькими этапами

Поддержка последовательных шагов:

- Сборка образа
- Запуск тестов
- Проверка безопасности
- Публикация (push)
- Развёртывание

---

## Что это даёт

- Полноценный CI/CD процесс внутри ACR
- Автоматизация цепочки build → test → push → deploy
- Уменьшение зависимости от внешних pipeline-систем
- Возможность реализовать сложные DevOps-сценарии

---

## Экзаменационный акцент (AZ-204)

- Multi-step tasks используются для сложных workflow
- Поддерживают несколько последовательных действий
- Подходят для автоматизации end-to-end процесса сборки и доставки

```yaml
# acr-task.yaml
version: v1.1.0
steps:
  # Step 1: Build web app image
  - build: -t {{.Run.Registry}}/webapp:{{.Run.ID}} -f Dockerfile .
  
  # Step 2: Run web app container
  - cmd: {{.Run.Registry}}/webapp:{{.Run.ID}}
    id: web
    detach: true
    ports:
      - 8080:80
  
  # Step 3: Build test image
  - build: -t {{.Run.Registry}}/tests:{{.Run.ID}} -f Dockerfile.test .
  
  # Step 4: Run tests against web app
  - cmd: {{.Run.Registry}}/tests:{{.Run.ID}}
    env:
      - WEB_URL=http://web:80
  
  # Step 5: Push if tests pass
  - push:
      - {{.Run.Registry}}/webapp:{{.Run.ID}}
      - {{.Run.Registry}}/webapp:latest
```

```bash
# Create multi-step task
az acr task create \
  --registry myregistry \
  --name multi-step-build \
  --context https://github.com/myorg/myrepo.git \
  --file acr-task.yaml \
  --git-access-token $(cat token.txt)

# Run task manually
az acr task run --registry myregistry --name multi-step-build
```

## Quick Task Examples

### Build from Local Context
```bash
# Build from current directory
az acr build --registry myregistry --image myapp:v1.0 .

# Build with specific Dockerfile
az acr build \
  --registry myregistry \
  --image myapp:v1.0 \
  --file Dockerfile.production \
  .
```

### Build from Git Repository
```bash
# Build from GitHub repo
az acr build \
  --registry myregistry \
  --image myapp:v1.0 \
  https://github.com/myorg/myrepo.git

# Build specific branch
az acr build \
  --registry myregistry \
  --image myapp:v1.0 \
  https://github.com/myorg/myrepo.git#develop

# Build with build args
az acr build \
  --registry myregistry \
  --image myapp:v1.0 \
  --build-arg VERSION=1.0.0 \
  .
```

### Build for Multiple Platforms
```bash
# Build for Linux ARM64
az acr build \
  --registry myregistry \
  --image myapp:v1.0-arm64 \
  --platform Linux/arm64 \
  .

# Build for Windows
az acr build \
  --registry myregistry \
  --image myapp:v1.0-windows \
  --platform Windows/amd64 \
  .
```

## Task Management

### Create Task
```bash
# Basic task
az acr task create \
  --registry myregistry \
  --name build-task \
  --image myapp:{{.Run.ID}} \
  --context https://github.com/myorg/myrepo.git \
  --file Dockerfile \
  --git-access-token <token>
```

### List Tasks
```bash
# List all tasks
az acr task list --registry myregistry --output table

# Show task details
az acr task show \
  --registry myregistry \
  --name build-task \
  --output json
```

### Run Task
```bash
# Run task manually
az acr task run --registry myregistry --name build-task

# Run task with overrides
az acr task run \
  --registry myregistry \
  --name build-task \
  --set VERSION=2.0.0
```

### View Task Runs
```bash
# List task runs
az acr task list-runs \
  --registry myregistry \
  --output table

# Show specific run
az acr task show-run \
  --registry myregistry \
  --run-id <run-id>

# Get run logs
az acr task logs \
  --registry myregistry \
  --run-id <run-id>
```

### Update Task
```bash
# Update task image
az acr task update \
  --registry myregistry \
  --name build-task \
  --image myapp:{{.Run.Date}}-{{.Run.ID}}

# Update trigger settings
az acr task update \
  --registry myregistry \
  --name build-task \
  --commit-trigger-enabled false
```

### Delete Task
```bash
# Delete task
az acr task delete \
  --registry myregistry \
  --name build-task
```

## Use Cases (Сценарии использования по расписанию)

- Ночные сборки (nightly builds)
- Регулярные задачи обслуживания
- Плановое сканирование образов
- Периодические тестовые прогоны

---

## Multi-Step Tasks

### Сложные workflow с несколькими этапами

Поддержка последовательных шагов:

- Сборка образа
- Запуск тестов
- Проверка безопасности
- Публикация (push)
- Развёртывание

---

## Что это даёт

- Полноценный CI/CD процесс внутри ACR
- Автоматизация цепочки build → test → push → deploy
- Уменьшение зависимости от внешних pipeline-систем
- Возможность реализовать сложные DevOps-сценарии

---

## Экзаменационный акцент (AZ-204)

- Multi-step tasks используются для сложных workflow
- Поддерживают несколько последовательных действий
- Подходят для автоматизации end-to-end процесса сборки и доставки


### Platform Specification
```bash
# Linux AMD64 (default)
--platform Linux/amd64

# Linux ARM
--platform Linux/arm

# Linux ARM64 with variant
--platform Linux/arm64/v8

# Windows
--platform Windows/amd64
```

# Task Variables (Переменные задач ACR)

## Встроенные переменные (Built-in Variables)

| Переменная | Описание | Пример |
|------------|----------|--------|
| `{{.Run.ID}}` | Уникальный идентификатор запуска | `ca1` |
| `{{.Run.Date}}` | Дата запуска (YYYYMMDD) | `20260103` |
| `{{.Run.Registry}}` | Имя реестра | `myregistry.azurecr.io` |
| `{{.Run.Commit}}` | SHA коммита Git | `abc123...` |
| `{{.Run.Branch}}` | Ветка Git | `main` |

---

## Для чего используются переменные

- Динамическое формирование тегов образов
- Версионирование сборок
- Автоматическое добавление метаданных
- Связь образа с конкретным коммитом или веткой

---

## Практическое применение

- Генерация уникальных тегов
- Трассировка сборок
- Поддержка CI/CD-процессов
- Улучшение audit и debugging

---

## Экзаменационный акцент (AZ-204)

- Переменные используются в шаблонах задач
- Позволяют автоматически формировать теги
- Связывают сборку с Git-коммитом или веткой
- Упрощают автоматизацию версионирования


### Usage in Tasks
```bash
# Use variables in image tags
az acr task create \
  --registry myregistry \
  --name build-task \
  --image myapp:{{.Run.Date}}-{{.Run.ID}} \
  --context https://github.com/myorg/myrepo.git
```

## Integration Examples

### Azure DevOps Pipeline
```yaml
# azure-pipelines.yml
trigger:
  - main

pool:
  vmImage: 'ubuntu-latest'

steps:
- task: AzureCLI@2
  inputs:
    azureSubscription: 'MyAzureSubscription'
    scriptType: 'bash'
    scriptLocation: 'inlineScript'
    inlineScript: |
      az acr build \
        --registry myregistry \
        --image myapp:$(Build.BuildId) \
        .
```

### GitHub Actions
```yaml
# .github/workflows/build.yml
name: Build Container Image
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      
      - name: Azure Login
        uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}
      
      - name: Build and Push
        run: |
          az acr build \
            --registry myregistry \
            --image myapp:${{ github.sha }} \
            .
```

# Critical Notes — ACR Tasks

- 💡 **Docker не требуется** — сборка полностью выполняется в Azure
- ⚠️ **Quick task** — `az acr build` = cloud-версия docker build + push
- 🎯 **Автоматические триггеры** — Git-коммит, обновление base image, расписание
- ✅ **Multi-step tasks** — YAML workflow (build → test → push)
- 📊 **Поддержка платформ** — Linux (amd64, arm, arm64), Windows (amd64)
- 🔄 **CI/CD интеграция** — Azure Pipelines, GitHub Actions, Jenkins
- 🔒 **Отслеживание base image** — автоматическая пересборка при обновлении
- ⏱️ **Scheduled tasks** — запуск по cron-расписанию

---

# Exam Tips (AZ-204)

## Основы

- ACR Tasks — сборка контейнерных образов в Azure без локального Docker
- Quick task — `az acr build` (одиночная сборка и публикация)

---

## Сценарии задач

- Quick build
- Сборка по Git-коммиту
- Пересборка при обновлении base image
- Запуск по расписанию
- Multi-step workflow

---

## Триггеры

- Коммит исходного кода
- Обновление base image
- Расписание (cron-формат)

---

## Multi-step tasks

- Описываются в YAML
- Поддерживают последовательность шагов: build → test → push

---

## Поддержка платформ

- Linux: amd64, arm, arm64, 386
- Windows: amd64
- Синтаксис: `OS/architecture` или `OS/architecture/variant`

---

## Переменные задач

- `{{.Run.ID}}`
- `{{.Run.Date}}`
- `{{.Run.Registry}}`
- и другие встроенные переменные

---

## Base image tracking

- Автоматическая пересборка при обновлении образа в ACR или Docker Hub
- Улучшение безопасности за счёт актуальных патчей

---

## Расписание

- Используется cron-синтаксис
- Пример: `0 2 * * *` — ежедневно в 02:00

---

## Управление задачами

- create
- list
- run
- show
- update
- delete
- logs

---

## Частые экзаменационные ловушки

- «Сборка без локального Docker» → ACR Tasks
- «Автоматическая пересборка при обновлении base image» → Base image trigger
- «Ночные сборки» → Scheduled task
- «Сложный workflow» → Multi-step task


[Learn More](https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/4-azure-container-registry-tasks)
