# Configure Path Mappings (Настройка сопоставлений путей)

## Key Concepts (Ключевые понятия)

- **Handler mappings (Сопоставления обработчиков)** — настройка пользовательских обработчиков для определённых расширений файлов
- **Virtual applications (Виртуальные приложения)** — сопоставление URL с различными физическими директориями
- **Custom storage (Пользовательское хранилище)** — подключение Azure Storage к контейнеризованным приложениям

> 💡 В контексте AZ-204 это относится к Azure App Service и управлению файловой системой веб-приложения.

---

## Windows Apps (Без контейнеров)

### Handler Mappings (Сопоставления обработчиков)

Позволяют добавить собственный обработчик для определённого расширения файла (если платформа не поддерживается по умолчанию).

### Параметры конфигурации:

- **Extension (Расширение)** — например:  
  `*.php`, `handler.fcgi`
- **Script processor (Обработчик)** — абсолютный путь к исполняемому файлу  
  Для корня приложения используется:  
  `D:\home\site\wwwroot`
- **Arguments (Аргументы)** — необязательные параметры командной строки

> ⚠️ `D:\home` — персистентное хранилище App Service. Всё вне этой директории может быть перезаписано при деплое.

---

### Virtual Applications & Directories (Виртуальные приложения и директории)

- **Путь по умолчанию**:  
  `/` → `D:\home\site\wwwroot`
- Можно сопоставить виртуальные пути с физическими директориями относительно `D:\home`
- Если снять флажок **"Directory"**, элемент становится **веб-приложением**, а не просто папкой

### Пример конфигурации

| Virtual Path | Physical Path        | Is Application |
|--------------|----------------------|----------------|
| `/`          | `site\wwwroot`       | ✅ Yes |
| `/api`       | `site\wwwroot\api`   | ✅ Yes |
| `/images`    | `site\assets\images` | ❌ No (directory) |

> 📝 `/api` может быть отдельным приложением (например, ASP.NET Core),  
> а `/images` — каталог статических ресурсов.

---

## Linux and Containerized Apps

### Custom Storage Mount (Монтирование хранилища)

Позволяет подключить Azure Storage (Blob или File) внутрь контейнера.

> ⚠️ Файловая система контейнера по умолчанию неперсистентная.

---

### Параметры конфигурации

| Setting | Description | Options |
|---------|-------------|---------|
| **Name** | Отображаемое имя | Произвольное |
| **Configuration** | Basic или Advanced | Basic: стандартное хранилище<br>Advanced: service endpoints, private endpoints, Key Vault |
| **Storage account** | Аккаунт Azure Storage | Выбор из подписки |
| **Storage type** | Blob или File | Blobs: только чтение<br>Files: чтение/запись |
| **Storage container** | Контейнер (Basic) | Существующий container |
| **Share name** | File share (Advanced) | Существующий share |
| **Access key** | Ключ доступа (Advanced) | Из storage account |
| **Mount path** | Путь внутри контейнера | Абсолютный путь (например, `/data`) |
| **Deployment slot setting** | Привязка к слоту | ✅/❌ |

---

### Storage Type Restrictions (Ограничения)

- **Windows containers** → поддерживается только Azure Files
- **Linux containers** → Azure Blobs (read-only) или Azure Files
- **Azure Blobs** → доступ только для чтения

> 🎯 Частый экзаменационный вопрос.

---

## Дополнительно (Важно для AZ-204)

### Файловая система App Service

- `D:\home` (Windows) или `/home` (Linux) — персистентная
- Остальная часть файловой системы может быть сброшена при рестарте

### Когда использовать Azure Files

- Нужно чтение/запись
- Нужно разделяемое хранилище между несколькими инстансами

### Когда использовать Blob

- Статические данные
- Только чтение
- Интеграция с CDN
- Более дешёвое хранение

---

## Quick Reference (Краткая выжимка)

| Сценарий | Решение |
|-----------|----------|
| Кастомный runtime | Handler mappings |
| Разделение приложения по URL | Virtual applications |
| Персистентное хранилище в контейнере | Azure Files |
| Только чтение в Linux контейнере | Azure Blob |
| Общий доступ между инстансами | Azure Files |


```bash
# Configure handler mapping (via portal only)
# Configuration > Path mappings > Handler mappings

# Configure virtual application (via portal only)
# Configuration > Path mappings > Virtual applications and directories

# Add Azure Storage mount (Linux/Container)
az webapp config storage-account add \
  --resource-group <rg-name> \
  --name <app-name> \
  --custom-id <mount-name> \
  --storage-type AzureBlob \
  --account-name <storage-account> \
  --share-name <container-name> \
  --access-key <storage-key> \
  --mount-path /data

# List storage mounts
az webapp config storage-account list \
  --resource-group <rg-name> \
  --name <app-name>

# Remove storage mount
az webapp config storage-account delete \
  --resource-group <rg-name> \
  --name <app-name> \
  --custom-id <mount-name>
```

## Use Cases (Сценарии использования)

| Scenario | Solution |
|----------|----------|
| Multiple apps in one deployment | Virtual applications |
| Custom PHP/Python handler | Handler mappings |
| Shared file storage across instances | Azure Files mount |
| Static assets from Blob Storage | Azure Blobs mount (read-only) |
| Separate app in subdirectory | Virtual application |

---

## Critical Notes (Критически важные моменты)

- 💡 **Windows containers** поддерживают только Azure Files (Blobs не поддерживаются)
- ⚠️ **Azure Blobs** при монтировании доступны только для чтения
- 🎯 **Корневой путь по умолчанию**: `D:\home\site\wwwroot` (Windows)
- 📝 **Virtual applications** могут работать как отдельные веб-приложения
- 🔐 Для private endpoints и service endpoints требуется **Advanced configuration**
- ⚠️ **Mount path** должен быть абсолютным (например, `/data`, а не `data`)

---

## Exam Tips (Советы для экзамена)

- Handler mappings применяются только к Windows App Service (не к Linux)
- Чётко понимать разницу между Virtual application и Directory
- Azure Blobs при монтировании всегда read-only
- Windows containers могут использовать только Azure Files
- Custom storage нужен при масштабировании (shared storage между инстансами)
- Настройки storage можно привязывать к deployment slot (slot setting)

[Learn More](https://learn.microsoft.com/en-us/training/modules/configure-web-app-settings/4-configure-path-mappings)
