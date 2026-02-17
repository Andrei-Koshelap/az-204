# Dockerfile — Основные элементы

## Ключевые понятия (Key Concepts)

- **Dockerfile** — файл с инструкциями для сборки Docker-образа
- **Слоистая архитектура** — каждая инструкция создаёт новый слой
- **Base image** — базовый образ (инструкция FROM)
- **Build context** — файлы, передаваемые Docker-демону для сборки

---

# Что такое Dockerfile?

**Текстовый файл с инструкциями** для сборки контейнерного образа.

- Содержит последовательность команд для создания образа
- Определяет базовый образ, зависимости и код приложения
- Каждая инструкция создаёт новый слой
- Слои кэшируются для ускорения повторной сборки

---

# Основные инструкции Dockerfile

## Базовые инструкции

| Инструкция | Назначение | Пример |
|------------|------------|--------|
| `FROM` | Указать базовый образ | `FROM node:18` |
| `WORKDIR` | Установить рабочую директорию | `WORKDIR /app` |
| `COPY` | Копировать файлы в образ | `COPY . /app` |
| `ADD` | Копировать файлы + распаковывать архивы | `ADD app.tar.gz /app` |
| `RUN` | Выполнить команду при сборке | `RUN npm install` |
| `CMD` | Команда по умолчанию при запуске | `CMD ["node", "app.js"]` |
| `ENTRYPOINT` | Основной исполняемый файл | `ENTRYPOINT ["python"]` |
| `EXPOSE` | Задокументировать порт | `EXPOSE 8080` |
| `ENV` | Задать переменную окружения | `ENV NODE_ENV=production` |
| `ARG` | Переменная на этапе сборки | `ARG VERSION=1.0` |
| `LABEL` | Добавить метаданные | `LABEL version="1.0"` |
| `USER` | Указать пользователя для RUN/CMD | `USER appuser` |
| `VOLUME` | Создать точку монтирования | `VOLUME /data` |
| `HEALTHCHECK` | Проверка состояния контейнера | `HEALTHCHECK CMD curl` |

---

## Важно понимать

- `FROM` — всегда первая инструкция (за исключением ARG перед FROM).
- `RUN` создаёт новый слой при каждой инструкции.
- `COPY` предпочтительнее `ADD`, если не требуется распаковка.
- `CMD` можно переопределить при запуске контейнера.
- `ENTRYPOINT` задаёт основной процесс контейнера.

---

## Экзаменационный акцент (AZ-204)

- Каждая инструкция = новый слой.
- Слои кэшируются для ускорения сборки.
- `COPY` и `RUN` влияют на кэш.
- `CMD` и `ENTRYPOINT` определяют поведение контейнера при запуске.
- `ARG` используется только во время сборки, `ENV` — во время выполнения.


## Example Dockerfile (.NET)

### Basic .NET Application
```dockerfile
# Use the .NET 6 runtime as a base image
FROM mcr.microsoft.com/dotnet/runtime:6.0

# Set the working directory to /app
WORKDIR /app

# Copy the contents of the published app to the container's /app directory
COPY bin/Release/net6.0/publish/ .

# Document that the application listens on port 80 (does not publish it)
EXPOSE 80

# Set the command to run when the container starts
CMD ["dotnet", "MyApp.dll"]
```
## Пояснение построчно (Line-by-line explanation)

1. `FROM mcr.microsoft.com/dotnet/runtime:6.0`  
   Используется базовый образ с .NET 6 Runtime в качестве основы для контейнера.

2. `WORKDIR /app`  
   Создаётся и устанавливается рабочая директория `/app`.

3. `COPY bin/Release/net6.0/publish/ .`  
   Скопированы опубликованные файлы приложения в директорию `/app`.

4. `EXPOSE 80`  
   Указывается, что приложение слушает порт 80 (документирование порта).

5. `CMD ["dotnet", "MyApp.dll"]`  
   Команда, которая запускается при старте контейнера.

---

⚠️ **Важно:**  
`EXPOSE` не публикует порт наружу.  
Для проброса порта используется параметр `-p` при запуске контейнера.

## Detailed Instruction Examples

### FROM - Base Image
```dockerfile
# Official image from Docker Hub
FROM node:18

# Microsoft image from MCR
FROM mcr.microsoft.com/dotnet/aspnet:6.0

# Multi-stage build (multiple FROM)
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
# ...
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS runtime
```

### WORKDIR - Set Working Directory
```dockerfile
# Create and switch to /app
WORKDIR /app

# Subsequent commands run from /app
COPY package.json .
RUN npm install

# Equivalent to: COPY package.json /app
```

### COPY vs ADD
```dockerfile
# COPY - Simple file copy (preferred)
COPY package.json /app/
COPY src/ /app/src/

# ADD - Copy + auto-extract tar files
ADD archive.tar.gz /app/

# ADD - Download from URL (not recommended)
ADD https://example.com/file.txt /app/
```

💡 **Best Practice**: Use `COPY` unless you need `ADD` features

### RUN - Execute Commands
```dockerfile
# Install packages (Linux)
RUN apt-get update && apt-get install -y curl

# Install dependencies (Node.js)
RUN npm install

# Multiple commands in one layer
RUN apt-get update && \
    apt-get install -y curl git && \
    apt-get clean

# Create user
RUN adduser --disabled-password --gecos '' appuser
```

### CMD vs ENTRYPOINT

#### CMD - Default Command
```dockerfile
# Exec form (preferred)
CMD ["node", "server.js"]

# Can be overridden
docker run myimage python app.py  # Runs python instead
```

#### ENTRYPOINT - Fixed Executable
```dockerfile
# Fixed entry point
ENTRYPOINT ["python", "app.py"]

# Cannot be overridden (only args can)
docker run myimage --debug  # Runs: python app.py --debug
```

#### Combined ENTRYPOINT + CMD
```dockerfile
# Fixed command with default args
ENTRYPOINT ["python", "app.py"]
CMD ["--port", "8080"]

# Default: python app.py --port 8080
# Override args: docker run myimage --port 9000
```

### ENV - Environment Variables
```dockerfile
# Set environment variables
ENV NODE_ENV=production
ENV PORT=8080
ENV DATABASE_URL=postgres://localhost/db

# Use in subsequent commands
RUN echo "Environment: $NODE_ENV"
```

### ARG - Build Arguments
```dockerfile
# Define build argument with default
ARG VERSION=1.0.0
ARG BUILD_DATE

# Use in Dockerfile
LABEL version=$VERSION
RUN echo "Building version $VERSION"

# Set at build time
docker build --build-arg VERSION=2.0.0 .
```

## Разница: ARG vs ENV

- `ARG` — доступна только во время сборки образа
- `ENV` — доступна во время сборки и при запуске контейнера

---

## Что это означает

### ARG
- Используется для параметризации сборки
- Значение можно передать при build
- Не сохраняется в финальном контейнере

### ENV
- Устанавливает переменную окружения
- Доступна приложению во время выполнения
- Сохраняется в образе

---

## Экзаменационный акцент (AZ-204)

- Нужно значение только при сборке → **ARG**
- Нужно значение во время выполнения → **ENV**
- ARG не доступен после запуска контейнера
- ENV остаётся в runtime-среде


### EXPOSE - Document Ports
```dockerfile
# Single port
EXPOSE 8080

# Multiple ports
EXPOSE 8080 8443

# Port + protocol
EXPOSE 3000/tcp
EXPOSE 53/udp
```

⚠️ **Important**: Does not actually publish ports - only documentation

```bash
# Publish port at runtime
docker run -p 8080:8080 myimage
docker run -p 9000:8080 myimage  # Host 9000 → Container 8080
```

### USER - Set User
```dockerfile
# Run as non-root user
RUN adduser --disabled-password appuser
USER appuser

# All subsequent commands run as appuser
RUN whoami  # Returns: appuser
CMD ["./app"]  # Runs as appuser
```

### VOLUME - Mount Points
```dockerfile
# Create mount point for persistent data
VOLUME /data

# Multiple volumes
VOLUME ["/data", "/logs"]

# Use at runtime
docker run -v /host/path:/data myimage
```

# Multi-Stage Builds

## Зачем использовать Multi-Stage?

✅ **Меньший размер образа** — инструменты сборки не попадают в финальный образ  
✅ **Разделение сборки и запуска** — сборка в SDK-образе, запуск в runtime-образе  
✅ **Безопасность** — меньше установленных инструментов → меньше поверхность атаки

---

## Что это даёт на практике

- Финальный образ содержит только необходимые runtime-зависимости
- Уменьшается размер скачивания и время запуска
- Снижается количество уязвимостей
- Улучшается производительность CI/CD

---

## Типичный подход

1. Первый этап — build (SDK, компиляция, тесты)
2. Второй этап — runtime (только скомпилированное приложение)

---

## Экзаменационный акцент (AZ-204)

- Multi-stage → уменьшение размера образа
- Инструменты сборки не входят в финальный контейнер
- Улучшение безопасности и производительности


### Example: .NET Multi-Stage Build
```dockerfile
# Stage 1: Build
FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["MyApp.csproj", "./"]
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Stage 2: Runtime
FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS runtime
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

**Result**: Final image contains only runtime + published app (no SDK)

### Example: Node.js Multi-Stage Build
```dockerfile
# Stage 1: Build
FROM node:18 AS build
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production
FROM node:18-alpine AS production
WORKDIR /app
COPY --from=build /app/dist ./dist
COPY --from=build /app/node_modules ./node_modules
EXPOSE 3000
CMD ["node", "dist/server.js"]
```

# Layer Caching

## Как работает кэширование слоёв

1. Docker кэширует каждый слой образа
2. Если инструкция не изменилась — используется кэшированный слой
3. Первая изменённая инструкция инвалидирует кэш для всех последующих слоёв

---

## Что это означает на практике

- Порядок инструкций влияет на скорость сборки
- Часто изменяемые шаги лучше размещать ниже
- Зависимости (например, установка пакетов) — выше, если они редко меняются
- Изменение одного файла может пересобрать половину образа

---

## Практические рекомендации

- Копируйте сначала файлы зависимостей, затем остальной код
- Объединяйте команды RUN, чтобы уменьшить количество слоёв
- Используйте `.dockerignore`, чтобы не отправлять лишние файлы

---

## Экзаменационный акцент (AZ-204)

- Каждая инструкция = отдельный слой
- Изменение инструкции → сброс кэша ниже по Dockerfile
- Правильный порядок инструкций ускоряет сборку


### Optimization: Order Instructions by Change Frequency
```dockerfile
# ❌ BAD - Changes to source invalidate npm install cache
FROM node:18
WORKDIR /app
COPY . .              # Changes frequently
RUN npm install       # Must rebuild every time

# ✅ GOOD - Cache npm install layer
FROM node:18
WORKDIR /app
COPY package*.json ./ # Changes rarely
RUN npm install       # Cached unless package.json changes
COPY . .              # Changes frequently
```

## Best Practices

### 1. Use Specific Base Image Tags
```dockerfile
# ❌ Avoid: Latest tag
FROM node:latest

# ✅ Use: Specific version
FROM node:18.15.0-alpine
```

### 2. Minimize Layers
```dockerfile
# ❌ Multiple layers
RUN apt-get update
RUN apt-get install -y curl
RUN apt-get clean

# ✅ Single layer
RUN apt-get update && \
    apt-get install -y curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*
```

### 3. Use .dockerignore
```
# .dockerignore
node_modules
.git
*.log
.env
.vscode
```

### 4. Run as Non-Root User
```dockerfile
RUN adduser --disabled-password appuser
USER appuser
```

### 5. Multi-Stage Builds
```dockerfile
# Build stage with all tools
FROM node:18 AS build
# ...

# Production stage with minimal footprint
FROM node:18-alpine AS production
COPY --from=build /app/dist .
```

### 6. Use Alpine Images
```dockerfile
# Standard: ~900 MB
FROM node:18

# Alpine: ~100 MB
FROM node:18-alpine
```

### 7. Label Images
```dockerfile
LABEL maintainer="team@example.com"
LABEL version="1.0.0"
LABEL description="My application"
```

## Common Dockerfile Patterns

### Node.js Application
```dockerfile
FROM node:18-alpine
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production
COPY . .
EXPOSE 3000
USER node
CMD ["node", "server.js"]
```

### Python Application
```dockerfile
FROM python:3.11-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
USER nobody
CMD ["python", "app.py"]
```

### Java Spring Boot
```dockerfile
FROM eclipse-temurin:17-jre-alpine
WORKDIR /app
COPY target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Build Commands

### Basic Build
```bash
# Build image from Dockerfile
docker build -t myapp:v1.0 .

# Build with specific Dockerfile
docker build -t myapp:v1.0 -f Dockerfile.prod .

# Build with build args
docker build --build-arg VERSION=1.0.0 -t myapp:v1.0 .
```

### ACR Build
```bash
# Build in Azure
az acr build \
  --registry myregistry \
  --image myapp:v1.0 \
  --file Dockerfile \
  .
```

# Critical Notes — Dockerfile

- 💡 **Dockerfile** — текстовый скрипт с инструкциями для сборки образа
- ⚠️ **Layers** — каждая инструкция создаёт слой (кэшируется для ускорения сборки)
- 🎯 **FROM** — обязательная первая инструкция (задаёт базовый образ)
- ✅ **Multi-stage** — сборка в SDK, запуск в runtime (меньший размер образа)
- 📊 **Порядок важен** — инструкции размещаются с учётом частоты изменений
- 🔄 **COPY предпочтительнее** — используйте COPY вместо ADD (если не нужна распаковка)
- 🔒 **Non-root user** — запуск от непривилегированного пользователя повышает безопасность
- ⚠️ **EXPOSE** — только документирует порт, не публикует его

---

# Exam Tips (AZ-204)

## Основные инструкции

- Dockerfile — скрипт для сборки контейнерного образа
- FROM — первая инструкция, задаёт базовый образ
- WORKDIR — устанавливает рабочую директорию (создаёт её при отсутствии)
- COPY — копирование файлов (предпочтительнее ADD)
- ADD — копирование + автоматическая распаковка tar-архивов
- RUN — выполнение команды во время сборки (каждый RUN создаёт слой)
- CMD — команда по умолчанию (можно переопределить при запуске)
- ENTRYPOINT — основной исполняемый файл (аргументы можно переопределить)
- EXPOSE — документирует порт (публикация через `docker run -p`)
- ENV — переменная окружения (доступна при сборке и runtime)
- ARG — переменная только для этапа сборки
- USER — задаёт пользователя для последующих RUN/CMD

---

## Архитектура сборки

- Multi-stage builds — разделение этапов сборки и запуска
- Уменьшение размера и повышение безопасности
- Layer caching — порядок инструкций влияет на скорость сборки

---

## Best Practices

- Использовать конкретные теги образов (не `latest`)
- Минимизировать количество слоёв
- Использовать `.dockerignore`
- Запускать контейнер не от root
- Использовать lightweight-образы (например, alpine)

---

## Команды сборки

- Локальная сборка:  
  `docker build -t <name>:<tag> .`

- Сборка через ACR:  
  `az acr build --registry <name> --image <image> .`

---

## Частые экзаменационные ловушки

- EXPOSE не публикует порт
- ARG недоступен во время выполнения
- Изменение инструкции сбрасывает кэш ниже
- Multi-stage уменьшает размер образа
- COPY предпочтительнее ADD для обычного копирования


[Learn More](https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/5-dockerfile-components)
