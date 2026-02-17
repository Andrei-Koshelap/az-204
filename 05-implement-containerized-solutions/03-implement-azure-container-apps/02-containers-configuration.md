# Containers в Azure Container Apps

## Ключевые понятия (Key Concepts)

- **Linux-контейнеры** — поддерживается только архитектура linux/amd64
- **Несколько контейнеров** — поддержка sidecar-паттерна
- **Конфигурация** — ARM templates, переменные окружения, health probes
- **Без привилегированного режима** — контейнеры не запускаются в privileged mode

---

# Поддержка контейнеров

## Поддерживаемые типы

✅ **Linux x86-64** — архитектура linux/amd64  
✅ **Любой runtime** — Node.js, Python, Java, .NET, Go и др.  
✅ **Любой registry** — Docker Hub, ACR, приватные registry  
✅ **Несколько контейнеров** — поддержка sidecar-паттерна

---

## Ограничения

❌ **Нет privileged-контейнеров** — процессы, требующие повышенных прав, не поддерживаются  
❌ **Windows не поддерживается** — только Linux  
❌ **Нет ARM-архитектуры** — только x86-64 (amd64)

---

## Поддержка возможностей

| Возможность | Поддерживается |
|--------------|----------------|
| Linux-контейнеры | ✅ Да |
| Windows-контейнеры | ❌ Нет |
| x86-64 (amd64) | ✅ Да |
| ARM-архитектура | ❌ Нет |
| Privileged mode | ❌ Нет |
| Root-доступ | ❌ Приводит к ошибке выполнения |

---

## Что важно понимать

- Azure Container Apps работает только с Linux amd64-образами
- Образы для ARM (например, для Apple Silicon) не поддерживаются
- Privileged mode запрещён по соображениям безопасности
- Sidecar-контейнеры позволяют реализовать паттерны логирования, прокси, Dapr

---

## Экзаменационный акцент (AZ-204)

- Только Linux amd64
- Windows и ARM не поддерживаются
- Нет privileged mode
- Поддерживается multi-container (sidecar)
- Можно использовать любой runtime и любой registry


## Container Configuration

### ARM Template Example
```json
{
  "containers": [
    {
      "name": "main",
      "image": "myregistry.azurecr.io/myapp:v1.0",
      "env": [
        {
          "name": "HTTP_PORT",
          "value": "80"
        },
        {
          "name": "SECRET_VAL",
          "secretRef": "mysecret"
        }
      ],
      "resources": {
        "cpu": 0.5,
        "memory": "1Gi"
      },
      "volumeMounts": [
        {
          "mountPath": "/myfiles",
          "volumeName": "azure-files-volume"
        }
      ],
      "probes": [
        {
          "type": "liveness",
          "httpGet": {
            "path": "/health",
            "port": 8080
          },
          "initialDelaySeconds": 7,
          "periodSeconds": 3
        }
      ]
    }
  ]
}
```

# Configuration Options — Containers в Azure Container Apps

| Параметр | Описание | Пример |
|------------|------------|-----------|
| **name** | Имя контейнера | "main" |
| **image** | Образ контейнера | "nginx:alpine" |
| **env** | Переменные окружения | [{"name": "PORT", "value": "80"}] |
| **resources** | CPU и память | {"cpu": 0.5, "memory": "1Gi"} |
| **volumeMounts** | Подключение томов | [{"mountPath": "/data", "volumeName": "vol"}] |
| **probes** | Проверки здоровья | Liveness, Readiness, Startup |

---

## Что важно понимать

- **name** — уникальное имя контейнера внутри приложения
- **image** — может быть из Docker Hub, ACR или приватного registry
- **env** — используется для динамической конфигурации
- **resources** — задаёт лимиты CPU и памяти
- **volumeMounts** — подключает определённый volume к пути внутри контейнера
- **probes** — контролируют состояние и готовность контейнера

---

## Health Probes

- **Liveness probe** — перезапускает контейнер при сбое
- **Readiness probe** — определяет готовность принимать трафик
- **Startup probe** — проверка успешного старта приложения

---

## Экзаменационный акцент (AZ-204)

- CPU и память задаются явно
- Поддерживаются health probes (liveness, readiness, startup)
- Переменные окружения используются для конфигурации
- Volume подключается через volumeMounts
- Образ может быть из любого поддерживаемого registry

## Environment Variables

### Standard Variables
```json
{
  "env": [
    {
      "name": "API_URL",
      "value": "https://api.example.com"
    },
    {
      "name": "PORT",
      "value": "8080"
    }
  ]
}
```

### Secret References
```json
{
  "env": [
    {
      "name": "CONNECTION_STRING",
      "secretRef": "db-connection"
    },
    {
      "name": "API_KEY",
      "secretRef": "api-key-secret"
    }
  ]
}
```

## Resource Allocation

### CPU and Memory Limits
```json
{
  "resources": {
    "cpu": 0.5,        // 0.5 vCPU
    "memory": "1Gi"    // 1 GB memory
  }
}
```

# Common Configurations (Типовые конфигурации ресурсов)

| Тип приложения | CPU | Память |
|----------------|-----|--------|
| **Лёгкий API** | 0.25 | 0.5 Gi |
| **Стандартное приложение** | 0.5 | 1 Gi |
| **Интенсивная обработка** | 1.0 | 2 Gi |
| **Крупная нагрузка** | 2.0 | 4 Gi |

---

## Что это означает

- CPU указывается в ядрах (можно дробные значения)
- Память указывается в Gi (гибибайтах)
- Ресурсы влияют на стоимость и масштабирование
- Недостаток памяти может привести к перезапуску контейнера

---

# Health Probes (Проверки состояния)

## Liveness Probe

**Проверяет, жив ли контейнер.**

- Если probe не проходит — контейнер перезапускается
- Используется для обнаружения зависаний
- Помогает автоматически восстановить приложение

---

## Когда использовать Liveness Probe

- Приложение может зависнуть, но процесс остаётся активным
- Требуется автоматический перезапуск при сбое
- В микросервисной архитектуре для self-healing

---

## Экзаменационный акцент (AZ-204)

- Liveness → перезапуск контейнера при сбое
- Недостаток памяти может вызвать restart
- CPU и память задаются явно в конфигурации
- Ресурсы влияют на стоимость


```json
{
  "type": "liveness",
  "httpGet": {
    "path": "/health",
    "port": 8080,
    "httpHeaders": [
      {
        "name": "Custom-Header",
        "value": "liveness probe"
      }
    ]
  },
  "initialDelaySeconds": 7,
  "periodSeconds": 3
}
```

### Readiness Probe
**Checks if container is ready to serve traffic**:

```json
{
  "type": "readiness",
  "httpGet": {
    "path": "/ready",
    "port": 8080
  },
  "initialDelaySeconds": 5,
  "periodSeconds": 5
}
```

### Startup Probe
**Checks if container has started**:

```json
{
  "type": "startup",
  "httpGet": {
    "path": "/startup",
    "port": 8080
  },
  "initialDelaySeconds": 0,
  "periodSeconds": 3,
  "failureThreshold": 30
}
```

# Multiple Containers (Sidecar Pattern)

## Когда использовать Sidecar

Sidecar — это вспомогательный контейнер, который работает рядом с основным контейнером и расширяет его функциональность.

✅ **Log collector** — сбор и отправка логов основного приложения  
✅ **Cache refresher** — обновление кэша в общем томе (shared volume)  
✅ **Proxy** — service mesh, аутентификация, TLS-терминация  
✅ **Monitoring** — health-check, сбор метрик

---

## Что важно понимать

- Все контейнеры в Container App работают в одном revision
- Они разделяют сетевое пространство (localhost)
- Могут использовать общие volume
- Масштабируются вместе как единое приложение

---

## Когда НЕ использовать sidecar

- Если сервис должен масштабироваться независимо
- Если требуется отдельный lifecycle
- Если нагрузка сильно различается

---

## Экзаменационный акцент (AZ-204)

- Sidecar используется для расширения функциональности основного контейнера
- Контейнеры внутри приложения масштабируются вместе
- Общая сеть и общие volume доступны всем контейнерам
- Типовые сценарии: логирование, прокси, мониторинг


### Example: App + Log Collector
```json
{
  "containers": [
    {
      "name": "webapp",
      "image": "webapp:latest",
      "resources": {
        "cpu": 0.5,
        "memory": "1Gi"
      },
      "volumeMounts": [
        {
          "mountPath": "/var/log/app",
          "volumeName": "logs"
        }
      ]
    },
    {
      "name": "log-collector",
      "image": "fluentd:latest",
      "resources": {
        "cpu": 0.25,
        "memory": "0.5Gi"
      },
      "volumeMounts": [
        {
          "mountPath": "/logs",
          "volumeName": "logs"
        }
      ]
    }
  ]
}
```

## Sidecar Communication (Взаимодействие контейнеров в sidecar-паттерне)

- **Общие тома (shared volumes)** — обмен файлами между контейнерами
- **Localhost** — сетевое взаимодействие внутри одного приложения
- **Общий жизненный цикл** — запускаются и останавливаются вместе

---

## Что это означает

- Контейнеры могут передавать данные через файловую систему
- Вызовы между контейнерами выполняются через `localhost:<port>`
- Нельзя перезапустить или масштабировать один контейнер отдельно
- Все контейнеры входят в одну revision

---

⚠️ **Важно**: Если сервисы должны масштабироваться или версионироваться независимо — используйте отдельные Container Apps, а не sidecar.

---

## Экзаменационный акцент (AZ-204)

- Sidecar → общая сеть и lifecycle
- Взаимодействие через localhost
- Независимые микросервисы → отдельные Container Apps
- Sidecar подходит для вспомогательных задач (логирование, прокси, мониторинг)


## Container Registries

### Azure Container Registry
```json
{
  "registries": [
    {
      "server": "myregistry.azurecr.io",
      "username": "myregistry",
      "passwordSecretRef": "registry-password"
    }
  ]
}
```

### Docker Hub (Private)
```json
{
  "registries": [
    {
      "server": "docker.io",
      "username": "my-docker-username",
      "passwordSecretRef": "docker-password"
    }
  ]
}
```

### Using Managed Identity (ACR)
```bash
# Create container app with managed identity for ACR
az containerapp create \
  --name myapp \
  --resource-group myResourceGroup \
  --environment myenvironment \
  --image myregistry.azurecr.io/myapp:latest \
  --registry-server myregistry.azurecr.io \
  --registry-identity system
```

## Volume Mounts

### Azure Files Volume
```json
{
  "volumeMounts": [
    {
      "mountPath": "/data",
      "volumeName": "azure-files-volume"
    }
  ],
  "volumes": [
    {
      "name": "azure-files-volume",
      "storageType": "AzureFile",
      "storageName": "myfilestorage"
    }
  ]
}
```

### EmptyDir Volume (Temporary)
```json
{
  "volumeMounts": [
    {
      "mountPath": "/tmp",
      "volumeName": "temp-storage"
    }
  ],
  "volumes": [
    {
      "name": "temp-storage",
      "storageType": "EmptyDir"
    }
  ]
}
```

# Automatic Restart в Azure Container Apps

## Перезапуск при сбое

✅ **Автоматически** — контейнер перезапускается при crash  
✅ **Health probes** — перезапуск при сбое liveness probe  
✅ **Без ручного вмешательства** — self-healing поведение

---

# Critical Notes

- 💡 **Только Linux** — поддерживаются контейнеры Linux x86-64 (amd64)
- ⚠️ **Нет privileged mode** — ошибка выполнения при необходимости root-доступа
- 🎯 **Несколько контейнеров** — поддержка sidecar-паттерна
- ✅ **Health probes** — liveness, readiness, startup
- 📊 **Лимиты ресурсов** — CPU (vCPU) и память (Gi)
- 🔄 **Автоперезапуск** — при crash или сбое liveness probe
- 🔒 **Secret references** — использование `secretRef` в переменных окружения
- ⚠️ **Volume mounts** — Azure Files (персистентный) или EmptyDir (временный)

---

# Exam Tips (AZ-204)

## Контейнерная поддержка

- Только Linux (Windows не поддерживается)
- Архитектура — x86-64 (amd64)
- Privileged-контейнеры не поддерживаются

---

## Несколько контейнеров

- Sidecar-паттерн
- Общий lifecycle
- Общая сеть (localhost)
- Общие volume

---

## Конфигурация

- Любое изменение конфигурации создаёт новую revision
- Переменные окружения:
    - `value` — обычная
    - `secretRef` — ссылка на секрет

---

## Ресурсы

- CPU указывается в vCPU
- Память указывается в Gi

---

## Health Probes

- **Liveness** — контейнер жив?
- **Readiness** — готов принимать трафик?
- **Startup** — успешно стартовал?

Типы probe:

- `httpGet`
- `tcpSocket`
- `exec`

---

## Реестры контейнеров

- Публичные
- ACR (рекомендуется Managed Identity)
- Приватные (с credentials)

---

## Volume

- Azure Files — персистентное
- EmptyDir — временное

---

## Автоматический перезапуск

- При crash
- При сбое liveness probe

---

## Типовые sidecar-сценарии

- Логирование
- Кэширование
- Мониторинг
- Прокси


[Learn More](https://learn.microsoft.com/en-us/training/modules/implement-azure-container-apps/4-container-apps-containers)
