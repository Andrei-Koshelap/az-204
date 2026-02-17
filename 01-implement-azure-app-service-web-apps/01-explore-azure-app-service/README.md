# Azure App Service — Полный модуль

## 📚 Быстрая навигация

| № | Раздел | Ключевые темы для экзамена | Статус |
|----|------------------------------|--------------------------------------|--------|
| 1 | [Обзор Azure App Service](./01-examine-azure-app-service.md) | Возможности PaaS, Linux, ASE | ✅ |
| 2 | [App Service Plans](./02-app-service-plans.md) | Ценовые уровни, стратегии масштабирования | ✅ |
| 3 | [Развертывание в App Service](./03-deploy-to-app-service.md) | CI/CD, deployment slots, контейнеры | ✅ |
| 4 | [Аутентификация и авторизация](./04-authentication-authorization.md) | Встроенная аутентификация, identity-провайдеры | ✅ |
| 5 | [Сетевые возможности](./05-networking-features.md) | VNet Integration, outbound IP, Private Endpoint | ✅ |
| 6 | [Настройка App Settings](./06-configure-app-settings.md) | Переменные окружения, slot settings | ✅ |
| 7 | [Настройка General Settings](./07-configure-general-settings.md) | Always On, ARR Affinity, HTTPS, TLS | ✅ |
| 8 | [Диагностика и логирование](./08-diagnostic-logging.md) | Типы логов, streaming, хранение | ✅ |

---

## 🎯 Как использовать этот модуль

1. Изучайте разделы по порядку.
2. Обращайте внимание на блоки:
    - Экзаменационные ловушки
    - Критические замечания
    - Best Practices
3. После прохождения всех разделов повторите:
    - Networking vs Security
    - Deployment vs Configuration
    - Plan vs App масштабирование

---

## 🧠 Фокус для AZ-204

Особое внимание уделите:

- Разнице между inbound и outbound networking
- Deployment slots и swap
- Always On и ARR Affinity
- Managed Identity vs Easy Auth
- Private Endpoint vs Service Endpoint
- Slot settings vs обычные настройки

---

## 📌 Итог

Этот модуль покрывает:

- Архитектуру App Service
- Масштабирование
- Безопасность
- Сетевые возможности
- Конфигурацию
- Развертывание
- Диагностику

Достаточно для уверенной подготовки к разделу App Service на AZ-204.


## 🎯 Exam Focus Areas

### Critical Commands to Memorize
```bash
# Create web app with deployment
az webapp up --name <app> --runtime "NODE:18-lts"

# Deployment slot operations
az webapp deployment slot create --name <app> --slot staging
az webapp deployment slot swap --name <app> --slot staging

# Configuration
az webapp config appsettings set --name <app> --settings KEY=value
az webapp config set --name <app> --always-on true

# Logging
az webapp log tail --name <app>
az webapp log download --name <app> --log-file logs.zip
```

## 📌 Требования к Tier — Быстрая справка

| Функция | Минимальный Tier |
|------------|------------------|
| Custom Domains | Basic |
| SSL/TLS | Basic |
| Deployment Slots | Standard |
| Always On | Basic |
| VNet Integration | Standard |
| Auto Scale | Standard |
| Managed Identity | Все tier |

💡 Если в задаче требуется deployment slots или auto scale — минимум Standard.

---

# 📖 Типовые экзаменационные сценарии

## Сценарий 1: Zero-Downtime Deployment

**Ответ:** Использовать Deployment Slots (Standard tier+)

1. Развернуть в staging
2. Протестировать
3. Выполнить swap в production

---

## Сценарий 2: Конфигурация для разных сред

**Ответ:** Использовать Slot Settings

- Отметить как "Deployment slot setting"
- Не участвуют в swap

---

## Сценарий 3: Приложение должно работать постоянно

**Ответ:** Включить Always On (Basic tier+)

- Обязательно для WebJobs
- Предотвращает 20-минутную выгрузку

---

## Сценарий 4: Stateless API и производительность

**Ответ:** Отключить ARR Affinity

- Улучшает балансировку нагрузки
- Нет необходимости в sticky sessions

---

## Сценарий 5: Безопасное production-приложение

**Ответ:** Комбинация настроек

- HTTPS Only = true
- Minimum TLS = 1.2
- Использовать Key Vault для секретов

---

# 📊 Decision Matrices

## Какой Pricing Tier выбрать?

| Требование | Tier |
|-------------|------|
| Только тестирование | Free / Shared |
| Базовый production | Basic |
| Нужны deployment slots | Standard+ |
| Высокая производительность | PremiumV3 |
| Сетевая изоляция | IsolatedV2 |

---

## Какой способ развертывания выбрать?

| Сценарий | Метод |
|-----------|--------|
| Современный CI/CD | GitHub Actions, Azure DevOps |
| Быстрый тест | `az webapp up` |
| Контейнерное приложение | Azure Container Registry + Slots |
| Legacy-система | FTP/S (не рекомендуется) |

---

## Какой тип логирования выбрать?

| Цель | Тип | Хранение |
|-------|------|------------|
| Временная отладка | Application (File System) | Автоудаление через 12 часов |
| Долгосрочное хранение | Application (Blob) | Постоянное |
| Логи HTTP-запросов | Web Server | File или Blob |
| Production-ошибки | Application (Blob, уровень Error) | Blob Storage |

---

# 🔍 Search Patterns (Экзаменационная логика)

Если в вопросе встречается:

- "zero downtime" → Deployment Slots
- "environment-specific configuration" → Slot Settings
- "app goes idle" → Always On
- "stateless API" → ARR Off
- "secure production" → HTTPS + TLS 1.2 + Key Vault
- "private access only" → Private Endpoint
- "fixed outbound IP" → NAT Gateway
- "access on-prem" → Hybrid Connections
- "scaling strategy" → App Service Plan

---

# 🎯 Финальный вывод

Чтобы правильно ответить на вопросы AZ-204 по App Service:

1. Определите: inbound или outbound задача.
2. Определите: stateless или stateful архитектура.
3. Определите: требуется ли production-grade безопасность.
4. Проверьте минимальный необходимый tier.
5. Выберите наиболее безопасное и современное решение.

App Service — это конфигурация + архитектурный выбор.
Экзамен проверяет именно архитектурное мышление.

### Find Commands
```bash
# Find all CLI commands
grep -r "az webapp" .

# Find specific settings
grep -r "always-on" .

# Find connection strings
grep -r "SQLCONNSTR" .
```

### Find Topics
```bash
# Find deployment info
grep -r "deployment slot" .

# Find authentication info
grep -r "/.auth/login" .

# Find networking info
grep -r "VNet" .
```

# 💡 Pro Tips Summary

1. **Всегда используйте deployment slots** для production (требуется Standard+).
2. **Помечайте конфигурации среды** как slot settings.
3. **Отключайте ARR Affinity** для stateless-приложений.
4. **Включайте Always On** для WebJobs и production-приложений.
5. **Используйте Blob Storage** для долгосрочного хранения логов.
6. **Принудительно включайте HTTPS и TLS 1.2+** в production.
7. **Используйте Key Vault references** вместо хранения секретов.
8. **Версионируйте контейнерные образы**, не используйте `latest`.
9. **Outbound IP-адреса могут изменяться** при смене tier.
10. **Connection strings имеют префиксы** (`SQLCONNSTR_`, `MYSQLCONNSTR_` и т.д.).

---

# 📝 Важные нюансы (Gotchas)

- ⚠️ Free/Shared tiers предназначены только для dev/test.
- ⚠️ File System logging автоматически отключается через 12 часов.
- ⚠️ Remote debugging автоматически отключается через 48 часов.
- ⚠️ Always On недоступен в Free tier.
- ⚠️ Deployment slots требуют Standard tier или выше.
- ⚠️ В Linux для вложенных JSON-ключей используется `__` вместо `:`.
- ⚠️ Connection strings предназначены в основном для .NET.
- ⚠️ Outbound IP-адреса меняются при переходе между VM families.

---

# 🎓 Рекомендации по изучению

## 🔹 Pass 1: Быстрый обзор (30 минут)

- Прочитать разделы "Key Concepts".
- Просмотреть все таблицы "Quick Reference".
- Отметить темы, требующие углубления.

---

## 🔹 Pass 2: Углублённое изучение (2 часа)

- Подробно изучить слабые темы.
- Попробовать команды Azure CLI.
- Разобрать примеры кода.

---

## 🔹 Pass 3: Практика (1 час)

- Создать тестовое приложение в Azure.
- Настроить deployment slots.
- Попрактиковаться с конфигурацией и логированием.
- Настроить аутентификацию.

---

## 🔹 Pass 4: Финальное повторение (30 минут)

- Повторить разделы "Critical Notes".
- Повторить "Exam Tips".
- Проверить себя по decision matrices.

---

# ✅ Модуль завершён

- Пройдено разделов: **8 из 8**
- Оценочное время изучения: **~4 часа**
- Рекомендуемая практика: **1–2 часа hands-on**

---

# 🎯 Финальный совет

AZ-204 проверяет не запоминание кнопок в портале,  
а умение:

- выбрать правильный tier,
- определить inbound vs outbound проблему,
- обеспечить безопасность,
- настроить масштабирование,
- выбрать корректную стратегию deployment.

Если вы понимаете архитектурную логику — раздел App Service будет одним из самых лёгких на экзамене.


[← Back to Main Index](../../README.md) | [Next: Azure Functions →](../02-azure-functions/)
