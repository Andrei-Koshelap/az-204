# Разбор Azure App Service

## Обзор

Azure App Service — это полностью управляемая платформа (PaaS) для разработки и размещения:

- веб-приложений
- REST API
- мобильных backend-сервисов
- контейнерных приложений

Главная идея: разработчик фокусируется на коде, а инфраструктурой (VM, ОС, патчи, масштабирование, балансировка нагрузки) управляет Azure.

---

## Ключевые концепции

### Полностью управляемый PaaS
- Не требуется администрирование серверов
- Нет необходимости устанавливать обновления ОС
- Масштабирование и балансировка нагрузки встроены

💡 **Комментарий:**  
App Service — хороший выбор, когда не требуется контроль на уровне Kubernetes или VM, но нужен быстрый запуск и масштабируемость.

---

### Поддержка нескольких языков
Поддерживаемые технологии:

- .NET
- Java (Java SE, Tomcat, JBoss)
- Node.js
- Python
- PHP

---

### Поддержка Windows и Linux
Можно развернуть приложение:

- в Windows-среде
- в Linux-среде

⚠️ Выбор ОС влияет на доступные функции и поддержку рантаймов.

---

### Поддержка контейнеров
Можно развернуть:

- кастомный Docker-контейнер
- образ из Azure Container Registry (ACR)
- образ из Docker Hub

💡 Это удобно, если нужен специфический runtime или кастомная конфигурация окружения.

---

# Основные возможности

## Автоматическое масштабирование (Auto Scale)

### Масштабирование вверх/вниз (Vertical Scaling)
- Изменение SKU
- Увеличение CPU и RAM

### Масштабирование наружу/внутрь (Horizontal Scaling)
- Увеличение количества инстансов
- Уменьшение количества инстансов

Основано на:
- метриках CPU
- использовании памяти
- HTTP-запросах
- пользовательских правилах

💡 Важно помнить: масштабируется **App Service Plan**, а не отдельное приложение.

---

## CI/CD интеграция

Поддерживается интеграция с:

- Azure DevOps
- GitHub (автоматический деплой из ветки)
- Bitbucket
- Local Git
- Azure Container Registry

💡 Это позволяет реализовать полноценный DevOps-процесс без дополнительной инфраструктуры.

---

## Deployment Slots

Доступны начиная с **Standard tier**.

Позволяют:

- развернуть версию в staging
- протестировать перед production
- выполнить swap
- обеспечить zero-downtime deployment

Каждый слот:
- имеет собственный hostname
- имеет собственные настройки

⚠️ Настройки, помеченные как *slot setting*, не обмениваются при swap.

---

# App Service на Linux

## Просмотр доступных runtime

```bash
az webapp list-runtimes --os-type linux

az webapp list-runtimes --linux -o tsv

```
Она выводит список доступных runtime-стеков, которые можно использовать в Azure App Service.
Например:
NODE|18-lts
PYTHON|3.11
DOTNETCORE|8.0
PHP|8.2

Что означает -o tsv?
Формат вывода:
tsv = tab-separated values
(удобно для скриптов)
## Ограничения App Service (Linux)

- ❌ **Не поддерживается в Shared tier**
- ❌ **Повышенная задержка диска для встроенных образов** (используется Azure Storage)
- ⚠️ В портале отображаются только функции, совместимые с Linux
- ✅ Можно использовать **кастомные контейнеры** для нестандартных рантаймов

💡 Если требуется полный контроль над окружением, версиями библиотек и зависимостями — рекомендуется использовать контейнер.

---

## App Service Environment (ASE)

**App Service Environment (ASE)** — это:

- полностью изолированная среда
- выделенные вычислительные ресурсы
- высокая масштабируемость
- изоляция на уровне сети (VNet)

### Используется для:

- Enterprise-сценариев
- требований повышенной безопасности
- compliance-задач
- внутренних корпоративных приложений

⚠️ Стоимость значительно выше стандартного App Service.

---

## Управление трафиком

### Входящий трафик (Inbound)

#### Access Restrictions
Ограничение доступа по IP-адресам (allow/deny).

#### Private Endpoint
Делает Web App доступным только из виртуальной сети (VNet).

#### Service Endpoint
Ограничивает доступ из определённых подсетей VNet.

#### Custom Domains + TLS
Подключение собственного домена и обязательного HTTPS.

---

### Исходящий трафик (Outbound)

По умолчанию исходящий трафик идёт через публичный интернет.

Чтобы ограничить или контролировать его:

#### VNet Integration
Позволяет Web App обращаться к ресурсам внутри виртуальной сети.

#### NAT Gateway
Обеспечивает фиксированный outbound IP-адрес.

#### Private DNS + Private Link
Позволяет безопасно подключаться к другим Azure-ресурсам без выхода в публичный интернет.

💡 На экзамене часто задают вопрос:  
**"Как обеспечить фиксированный исходящий IP?"**  
Ответ: **NAT Gateway**.

---

## Развертывание контейнера

### Создание Web App с контейнером

```bash
az webapp create \
  --resource-group myRG \
  --plan myPlan \
  --name myApp \
  --deployment-container-image-name myimage:latest
```
```bash
az webapp config container set \
  --name myApp \
  --resource-group myRG \
  --docker-custom-image-name myacr.azurecr.io/myimage:latest \
  --docker-registry-server-url https://myacr.azurecr.io

```

## Рекомендация по работе с ACR

Для подключения к Azure Container Registry (ACR) рекомендуется использовать **Managed Identity** вместо хранения логина и пароля (credentials).

Преимущества:
- отсутствие секретов в конфигурации
- централизованное управление доступом через RBAC
- повышение уровня безопасности
- соответствие best practices Azure

---

## Таблица доступности функций

| Функция | Доступность |
|----------|------------|
| Auto Scale | Все tier (с ограничениями) |
| Deployment Slots | Standard+ |
| Custom Domains | Basic+ |
| SSL/TLS | Basic+ |
| VNet Integration | Standard+ |
| Linux Support | Basic+ (не Shared) |

---

## Критические замечания

- Используйте **Deployment Slots** для production-деплоя без downtime.
- Free и Shared tier не подходят для production-нагрузки.
- Масштабирование выполняется на уровне **App Service Plan**, а не отдельного приложения.
- **Managed Identity** предпочтительнее SAS или connection string.
- **Private Endpoint** обеспечивает более высокий уровень безопасности по сравнению с Access Restrictions.

---

## Итог

**Azure App Service** — это:

- быстрый запуск приложений
- минимальное управление инфраструктурой
- встроенные механизмы безопасности
- интеграция с DevOps-процессами
- поддержка контейнеров

Подходит для большинства веб-приложений и REST API.

Если требуется сложная оркестрация микросервисов, управление pod-ами, авто-восстановление и гибкая маршрутизация — стоит рассмотреть использование Azure Kubernetes Service (AKS).


Когда открываешь Web App в Azure Portal, ты попадаешь на:
👉 Overview blade
URL приложения
Кнопка Browse
Статус
Restart / Stop
Metrics

B. A Webhook created by ACR triggers a task, which can be a multi-step process.
Что происходит при коммите в связанный Git-репозиторий
Если Azure Container Registry (ACR) настроен с ACR Task и подключён к Git-репозиторию (GitHub, Azure Repos и т.п.), то:
В репозитории происходит commit.
Git отправляет webhook.
ACR получает событие.
Запускается ACR Task.
Выполняется сборка образа (и при необходимости multi-step pipeline).
Новый контейнерный образ пушится в ACR.
Это автоматический CI-процесс внутри ACR.

| Если задача             | Ответ            |
| ----------------------- | ---------------- |
| ZIP deploy в production | Risky            |
| Safe deployment         | Deployment slots |
| Zero downtime           | Slots + Swap     |


free managed certificate:
Бесплатный
Автоматически продлевается
НО ❗
Работает только с одним App Service
Нельзя использовать в нескольких apps
Ограничения по wildcard и private domains

Import from Key Vault — нет?
Нужно самому управлять сертификатом
Нужно обеспечить обновление в Key Vault
Не минимальный overhead

Purchase an App Service certificate — правильно?
App Service Certificate:
Автоматически продлевается
Интегрируется с Key Vault
Можно использовать в нескольких App Services
Минимальный management

| Условие                    | Ответ                    |
| -------------------------- | ------------------------ |
| Auto renew + multiple apps | App Service Certificate  |
| Simple + single app        | Free managed certificate |
| External enterprise cert   | Key Vault import         |
