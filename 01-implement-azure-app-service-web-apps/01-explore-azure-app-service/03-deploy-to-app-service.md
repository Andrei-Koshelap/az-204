# Развертывание в Azure App Service

## Ключевые концепции

- **Автоматизированное развертывание** — использование CI/CD пайплайна (рекомендуется)
- **Ручное развертывание** — прямой push кода
- **Deployment Slots** — развертывание в staging перед production
- **Kudu** — внутренний движок развертывания для Git и ZIP Deploy

---

## Варианты автоматического развертывания

| Источник | Описание | Подходит для |
|-----------|------------|----------------|
| **Azure DevOps** | Полный CI/CD: сборка, тестирование, релиз | Enterprise-процессы |
| **GitHub** | Авто-деплой из определённой ветки | Современные проекты |
| **Bitbucket** | Аналог GitHub (реже используется) | Команды на Bitbucket |

### Преимущества автоматизации

- ✅ Быстрое и повторяемое развертывание
- ✅ Минимальный downtime
- ✅ Автоматическое тестирование
- ✅ Возможность rollback

---

## Варианты ручного развертывания

| Метод | Команда / Инструмент | Сценарий |
|--------|----------------------|------------|
| **Git** | `git push azure main` | Локальная разработка |
| **Azure CLI** | `az webapp up` | Быстрое развертывание (может создать App Service автоматически) |
| **ZIP Deploy** | `curl` или REST API | Развертывание архива |
| **FTP/S** | FTP-клиент | Legacy-сценарии |

---

### Essential Commands

```bash
# Deploy using Azure CLI (creates app if needed)
az webapp up \
  --name <app-name> \
  --resource-group <rg-name> \
  --runtime "NODE:18-lts"

# Deploy ZIP file
az webapp deploy \
  --name <app-name> \
  --resource-group <rg-name> \
  --src-path app.zip

# Set up Git deployment
az webapp deployment source config-local-git \
  --name <app-name> \
  --resource-group <rg-name>
```

## Deployment Slots

### Основные преимущества

- ✅ Развертывание без downtime
- ✅ Прогрев (warm-up) перед swap
- ✅ Быстрый rollback
- ✅ Тестирование в production-подобной среде

---

### Deployment Slot Workflow

```bash
# Create deployment slot
az webapp deployment slot create \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging

# Deploy to staging slot
az webapp deployment source config \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --repo-url <github-url> \
  --branch main

# Swap staging to production
az webapp deployment slot swap \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --target-slot production
```

## Лучшие практики Continuous Deployment

### Для развертывания кода

1. **Стратегия веток**  
   Сопоставьте ветки со слотами:
   - `main` → production
   - `develop` → staging

2. **Автоматическое развертывание**  
   Включите CI/CD напрямую из репозитория.

3. **Тестирование стейкхолдерами**  
   Используйте deployment slots для QA и предварительной проверки.

4. **Swap в production**  
   Выполняйте swap только после успешной валидации.

---

### Для развертывания контейнеров

1. **Используйте версионированные теги**  
   Применяйте commit ID или timestamp вместо `latest`.


   ```bash
   docker tag myapp:latest myregistry.azurecr.io/myapp:v1.2.3-abc123
   ```
2. **Push to registry** - Azure Container Registry or Docker Hub
3. **Update slot** - Point to new image tag
4. **Auto-restart** - App pulls new image and restarts

```bash
# Update container image
az webapp config container set \
  --name <app-name> \
  --resource-group <rg-name> \
  --slot staging \
  --docker-custom-image-name myregistry.azurecr.io/myapp:v1.2.3
```

## Sidecar Containers

### Основные концепции

- До 9 sidecar-контейнеров
- Только для Linux custom containers
- Используются для вспомогательных сервисов

### Типовые сценарии

| Тип | Назначение | Пример |
|------|------------|--------|
| Monitoring | Метрики | Prometheus exporter |
| Logging | Централизованный логинг | Fluentd |
| Configuration | Динамическая конфигурация | Consul |
| Networking | Service mesh | Envoy |

Sidecar-контейнеры:
- управляются через Deployment Center
- имеют общую сеть с основным контейнером

---

## Сравнение способов развертывания

| Метод | Время настройки | Автоматизация | Подходит для |
|--------|----------------|----------------|----------------|
| Azure DevOps | Высокое | ✅ Полная | Enterprise CI/CD |
| GitHub Actions | Среднее | ✅ Полная | Современные проекты |
| `az webapp up` | Низкое | ❌ Ручное | Быстрые тесты |
| ZIP Deploy | Низкое | ⚠️ Частично | Пакетное развертывание |
| Git Push | Низкое | ⚠️ Частично | Разработчики |
| FTP | Низкое | ❌ Ручное | Legacy |

---

## Критические замечания

- 🎯 Используйте Deployment Slots для production (Standard+)
- ⚠️ Не используйте `latest` для контейнеров
- 💡 Kudu отвечает за Git и ZIP Deploy
- 🚀 Swap выполняет прогрев инстансов
- 📦 `az webapp up` может создать приложение автоматически
- 🔄 Rollback выполняется через обратный swap

## Экзаменационные советы

- Различайте автоматическое и ручное развертывание
- Помните, что deployment slots требуют Standard tier+
- Kudu — движок Git и ZIP deploy
- Контейнеры должны использовать версионированные теги

---

# Экзаменационные ловушки AZ-204 по Deployment

## 1️⃣ Deployment Slots требуют Standard+

Если в задаче:
- zero-downtime deployment
- staging environment
- swap перед production

Ответ → **Standard tier или выше**

---

## 2️⃣ Swap не переносит slot settings

Настройки, помеченные как *slot setting*, не участвуют в swap.

⚠️ Частая ловушка на экзамене.

---

## 3️⃣ `latest` — плохая практика

Если в вопросе про контейнеры:

> "Ensure predictable deployments and easy rollback"

Правильный ответ → использовать версионированные теги, не `latest`.

---

## 4️⃣ Kudu = Git/ZIP Deploy Engine

Kudu:
- управляет Git deploy
- выполняет ZIP Deploy
- синхронизирует файлы

Если в вопросе Git или ZIP — за этим стоит Kudu.

---

## 5️⃣ `az webapp up` может создать приложение

CLI-команда может:
- создать App Service Plan
- создать Web App
- задеплоить приложение

⚠️ Экзамен проверяет понимание этого упрощённого способа.

---

## 6️⃣ Rollback через swap

Rollback не означает redeploy старой версии.

Правильный ответ → выполнить обратный swap слотов.

---

## 7️⃣ Manual vs Automated

Если в вопросе:
- требуется повторяемость
- аудит
- тестирование перед production

Правильный ответ → CI/CD pipeline.

---

## 8️⃣ Sidecar Containers работают только на Linux

Если в вопросе:
- несколько контейнеров
- sidecar pattern
- monitoring container

Ответ → Linux custom container.

---

## 9️⃣ Warm-up перед swap

Swap прогревает инстансы.

⚠️ Это предотвращает cold start.

---

## 🔟 CI/CD + Slots = best practice

Если нужно:
- минимальный downtime
- контроль качества
- безопасный релиз

Правильная комбинация:
CI/CD → staging slot → тестирование → swap


🔷 Как работает Python в Azure App Service?
Когда ты деплоишь Python-приложение:
Azure использует Oryx для авто-детекции
Если это Flask или Django — Azure знает, как запустить
Но для FastAPI (ASGI framework) нужен сервер вроде:
uvicorn
🔥 Если НЕ указать startup command?
Azure не знает:
какой файл запускать
какой ASGI сервер использовать
какой объект приложения стартовать
В итоге:
контейнер запускается
но приложение не стартует
получаешь runtime error / 500

[Learn More](https://learn.microsoft.com/en-us/training/modules/introduction-to-azure-app-service/4-deploy-code-to-app-service)
