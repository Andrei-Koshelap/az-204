# Включение диагностического логирования

## Ключевые понятия

- **Встроенная диагностика** для отладки и мониторинга
- **Несколько типов логов** для разных задач
- **Гибкие варианты хранения** — файловая система или Azure Storage
- **Доступ в реальном времени** и возможность просмотра истории

---

## Обзор типов логов

| Тип лога | Платформа | Варианты хранения | Назначение |
|------------|------------|------------------|------------|
| **Application Logging** | Windows, Linux | Файловая система, Blob Storage (Windows) | Сообщения из кода приложения |
| **Web Server Logging** | Windows | Файловая система, Blob Storage | «Сырые» HTTP-запросы (формат W3C) |
| **Detailed Error Messages** | Windows | Файловая система | HTML-страницы ошибок (HTTP 400+) |
| **Failed Request Tracing** | Windows | Файловая система | Трассировка запросов IIS |
| **Deployment Logging** | Windows, Linux | Файловая система | Включается автоматически, не настраивается |

---

## Application Logging

### Приложения Windows

#### Уровни логирования

| Уровень | Включает |
|----------|----------|
| **Disabled** | Логирование отключено |
| **Error** | Error, Critical |
| **Warning** | Warning, Error, Critical |
| **Information** | Info, Warning, Error, Critical |
| **Verbose** | Trace, Debug, Info, Warning, Error, Critical |

---

### Варианты хранения

- **File System (Файловая система)**  
  Используется для временной отладки  
  ⚠️ Автоматически отключается через 12 часов

- **Blob Storage**  
  Подходит для долгосрочного хранения логов  
  Требуется предварительно созданный контейнер в Azure Storage

---

### Важно для AZ-204

- File System подходит только для краткосрочной диагностики
- Для production-сценариев рекомендуется Blob Storage
- Уровень Verbose используется для глубокой отладки
- Web Server Logging и Failed Request Tracing доступны только на Windows

Часто в экзаменационных вопросах требуется определить:
- какой тип логирования включить
- где хранить логи
- как обеспечить долгосрочный доступ к данным диагностики


```bash
# Enable app logging (File System)
az webapp log config \
  --name <app-name> \
  --resource-group <rg-name> \
  --application-logging filesystem \
  --level information

# Enable app logging (Blob Storage)
az webapp log config \
  --name <app-name> \
  --resource-group <rg-name> \
  --application-logging azureblobstorage \
  --level verbose \
  --docker-container-logging filesystem
```

⚠️ **Regenerate storage keys**: Must reconfigure logging after key rotation

### Linux/Container Apps

```bash
# Enable app logging (Linux)
az webapp log config \
  --name <app-name> \
  --resource-group <rg-name> \
  --application-logging true \
  --docker-container-logging filesystem \
  --level information
```

#### Linux-Specific Settings
- **Quota (MB)**: Disk quota for logs
- **Retention Period (Days)**: How long to keep logs

## Web Server Logging (Windows Only)

### Configuration

```bash
# Enable web server logging (File System)
az webapp log config \
  --name <app-name> \
  --resource-group <rg-name> \
  --web-server-logging filesystem

# Enable web server logging (Blob Storage)
az webapp log config \
  --name <app-name> \
  --resource-group <rg-name> \
  --web-server-logging storage
```

### W3C Extended Log Format
Includes:
- HTTP method (GET, POST, etc.)
- Resource URI
- Client IP and port
- User agent
- Response code
- Time taken

## Add Log Messages in Code

### ASP.NET

```csharp
// ASP.NET - System.Diagnostics.Trace
System.Diagnostics.Trace.TraceError("Something bad happened");
System.Diagnostics.Trace.TraceWarning("Warning message");
System.Diagnostics.Trace.TraceInformation("Info message");
```

### ASP.NET Core

```csharp
// ASP.NET Core - ILogger
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;
    
    public HomeController(ILogger<HomeController> logger)
    {
        _logger = logger;
    }
    
    public IActionResult Index()
    {
        _logger.LogInformation("Home page visited");
        _logger.LogWarning("This is a warning");
        _logger.LogError("An error occurred");
        return View();
    }
}
```

### Node.js

```javascript
// Node.js - console
console.log('Information message');
console.warn('Warning message');
console.error('Error message');
```

### Python

```python
# Python - OpenCensus package
import logging
from opencensus.ext.azure.log_exporter import AzureLogHandler

logger = logging.getLogger(__name__)
logger.addHandler(AzureLogHandler())

logger.info("Information message")
logger.warning("Warning message")
logger.error("Error message")
```

## Stream Logs

### Azure Portal
- Navigate to app → **Log stream**
- Real-time log viewing

### Azure CLI

```bash
# Stream logs in Cloud Shell
az webapp log tail \
  --name <app-name> \
  --resource-group <rg-name>

# Stream logs locally
az webapp log tail \
  --name <app-name> \
  --resource-group <rg-name> \
  --provider application
```

### Log Location
- Logs stored in: `/LogFiles` directory (`d:/home/logfiles`)
- Files ending in: `.txt`, `.log`, `.htm`

⚠️ **Out-of-order events**: Buffered writes may cause sequence issues

## Access Log Files

### Download ZIP Archive

| App Type | URL |
|----------|-----|
| **Linux/Container** | `https://<app-name>.scm.azurewebsites.net/api/logs/docker/zip` |
| **Windows** | `https://<app-name>.scm.azurewebsites.net/api/dump` |

```bash
# Download logs (CLI)
az webapp log download \
  --name <app-name> \
  --resource-group <rg-name> \
  --log-file logs.zip
```

### Blob Storage Logs
- Use **Storage Explorer** or Azure portal
- Requires blob storage client tool

## Log File Structure

### Linux/Container Apps
```
/home/LogFiles/
├── docker/          # Docker container logs
├── Application/     # Application logs
└── kudu/           # Deployment logs
```

### Windows Apps
```
d:/home/logfiles/
├── Application/     # Application logs
├── http/           # Web server logs
├── DetailedErrors/ # Detailed error pages
└── W3SVC*/         # Failed request traces
```

## Configuration Quick Reference

```bash
# Complete logging setup
az webapp log config \
  --name <app-name> \
  --resource-group <rg-name> \
  --application-logging filesystem \
  --detailed-error-messages true \
  --failed-request-tracing true \
  --web-server-logging filesystem \
  --level verbose
```

## Хранение логов (Log Retention)

| Хранилище | Срок хранения по умолчанию | Настраивается |
|-------------|-----------------------------|---------------|
| **File System** | Зависит от тарифного плана | ✅ Да |
| **Blob Storage** | Без ограничений (хранятся постоянно) | ✅ Да (через lifecycle policies) |

---

## Важные замечания

- 💡 **Логирование в File System автоматически отключается через 12 часов**  
  (для Application Logging на Windows)

- 🎯 **Для долгосрочного хранения используйте Blob Storage**

- ⚠️ **При регенерации ключей хранилища** необходимо заново настроить логирование

- 📊 **Deployment Logging включён всегда** — дополнительная настройка не требуется

- 🔄 Использование буфера может привести к **нарушению порядка событий** при потоковом просмотре

- 🐧 На Linux доступны настройки **квот и срока хранения** (для File System)

- ⏰ Потоковый просмотр логов возможен только из каталога **/LogFiles**

---

### Что важно для AZ-204

- File System — временное решение для диагностики
- Blob Storage — production-подход
- При изменении ключей Storage логирование перестаёт работать
- Поток логов не гарантирует строгий порядок сообщений

В экзаменационных вопросах часто проверяют понимание различий между временным и постоянным хранением логов.


## Best Practices

### Development
```bash
# Enable verbose logging
az webapp log config \
  --name dev-app \
  --resource-group dev-rg \
  --application-logging filesystem \
  --level verbose
```

### Production
```bash
# Enable error logging to Blob Storage
az webapp log config \
  --name prod-app \
  --resource-group prod-rg \
  --application-logging azureblobstorage \
  --level error \
  --web-server-logging storage
```

## Exam Tips
- Know the difference between File System (temp, 12hr) and Blob Storage (long-term)
- Understand log levels: Disabled < Error < Warning < Information < Verbose
- Remember application logging available on Windows and Linux
- Web server logging, detailed errors, failed request tracing: Windows only
- Deployment logging is automatic and always enabled
- Know how to access logs via URL pattern (`scm.azurewebsites.net`)
- 
  | Тип логирования                   | Windows | Linux        | Что логирует                             | Когда использовать   |
  | --------------------------------- | ------- | ------------ | ---------------------------------------- | -------------------- |
  | **Application logging**           | ✅       | ✅            | Логи из кода (ILogger, console, log4net) | Debug приложения     |
  | **Web server logging**            | ✅       | ❌ (IIS only) | Все HTTP-запросы (status codes)          | Анализ трафика       |
  | **Detailed error logging**        | ✅       | ❌            | Подробные HTTP 4xx/5xx                   | Debug ошибок         |
  | **Failed request tracing (FREB)** | ✅       | ❌            | Только failed requests (400+)            | Troubleshooting      |
  | **Container logging**             | ❌       | ✅            | stdout/stderr контейнера                 | Linux container apps |
  | **Diagnostic logs to Blob**       | ✅       | ✅            | Централизованные логи                    | Продакшен            |


Если вопрос про:
deployment failure
web app deployment logs
Ответ обычно:

App Service filesystem (через Kudu)

| Нужно узнать       | Где смотреть    |
| ------------------ | --------------- |
| Кто изменил ресурс | Activity Log    |
| HTTP 500           | Diagnostic Logs |
| Почему деплой упал | Deployment Logs |
| Посмотреть файлы   | Kudu            |
| Ошибки контейнера  | Container logs  |


[Learn More](https://learn.microsoft.com/en-us/training/modules/configure-web-app-settings/5-enable-diagnostic-logging)
