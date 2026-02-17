# Mount Azure File Share в Azure Container Instances

## Ключевые понятия (Key Concepts)

- **Persistent storage** — данные сохраняются после перезапуска контейнера
- **Azure Files** — полностью управляемые SMB-файловые шары
- **Volume mount** — подключение хранилища к пути внутри контейнера
- **Только Linux** — монтирование Azure Files поддерживается только для Linux-контейнеров

---

# Зачем монтировать Azure Files?

## Сохранение данных вне жизненного цикла контейнера

- Локальное хранилище контейнера временное (теряется при перезапуске)
- Возможность разделять данные между контейнерами
- Хранение логов, конфигураций, пользовательских данных
- Поддержка сценариев резервного копирования
- Обмен файлами между несколькими Container Instances

---

## Что это даёт

- Персистентность данных
- Централизованное файловое хранилище
- Совместный доступ к данным
- Упрощение stateful-сценариев в ACI

---

## Когда использовать

- Приложение должно сохранять данные
- Нужно хранить логи
- Требуется общий доступ к файлам
- Необходима долговременная персистентность

---

## Экзаменационный акцент (AZ-204)

- Контейнерное хранилище по умолчанию — временное
- Для постоянных данных → Azure Files
- Поддерживается только для Linux-контейнеров
- Монтирование происходит через volume в Container Group

### Problem Without Persistent Storage
```
Container created → Data written → Container stops → Data LOST
```

### Solution With Azure Files
```
Container created → Data written to Azure Files → Container stops → Data PERSISTS
```

# Azure Files — Обзор

## Что такое Azure Files?

**Полностью управляемые файловые шары** в облаке.

- **SMB-протокол** — стандарт CIFS
- **Доступность** — можно подключать из VM, контейнеров и on-premises
- **Совместный доступ** — несколько контейнеров могут монтировать одну шару
- **Персистентность** — данные сохраняются после перезапуска контейнера

---

# Ограничения

## Важные ограничения

❌ **Только Linux** — монтирование не поддерживается для Windows-контейнеров  
❌ **Требуется root** — Linux-контейнер должен запускаться от root  
❌ **Только CIFS (SMB)** — поддержка NFS отсутствует  
❌ **Одна шара на контейнер** — но можно подключать разные шары к разным контейнерам в группе

---

## Поддержка функций

| Возможность | Поддерживается |
|-------------|----------------|
| **Linux-контейнеры** | ✅ Да |
| **Windows-контейнеры** | ❌ Нет |
| **Root-пользователь** | ✅ Обязателен |
| **CIFS/SMB** | ✅ Да |
| **NFS** | ❌ Нет |

---

## Что важно учитывать

- Контейнер должен работать от root для монтирования Azure Files.
- Windows ACI не поддерживает подключение Azure Files.
- Azure Files подходит для stateful-сценариев в Linux ACI.

---

## Экзаменационный акцент (AZ-204)

- Azure Files поддерживается только для Linux ACI
- Требуется root-доступ
- Используется SMB (CIFS)
- NFS не поддерживается

## Prerequisites

### Create Storage Account and File Share
```bash
# Create storage account
az storage account create \
  --resource-group myResourceGroup \
  --name mystorageaccount \
  --location eastus \
  --sku Standard_LRS

# Get storage account key
STORAGE_KEY=$(az storage account keys list \
  --resource-group myResourceGroup \
  --account-name mystorageaccount \
  --query "[0].value" \
  --output tsv)

# Create file share
az storage share create \
  --name myshare \
  --account-name mystorageaccount \
  --account-key $STORAGE_KEY
```

## Mount File Share (Azure CLI)

### Basic Mount
```bash
# Create container with mounted Azure Files
az container create \
  --resource-group myResourceGroup \
  --name mycontainer \
  --image nginx \
  --azure-file-volume-account-name mystorageaccount \
  --azure-file-volume-account-key $STORAGE_KEY \
  --azure-file-volume-share-name myshare \
  --azure-file-volume-mount-path /data

# Files in /data persist across container restarts
```

### Full Example
```bash
# Variables
RESOURCE_GROUP="myResourceGroup"
STORAGE_ACCOUNT="mystorageaccount"
SHARE_NAME="myshare"
STORAGE_KEY=$(az storage account keys list \
  --resource-group $RESOURCE_GROUP \
  --account-name $STORAGE_ACCOUNT \
  --query "[0].value" --output tsv)

# Create container with Azure Files
az container create \
  --resource-group $RESOURCE_GROUP \
  --name webapp \
  --image webapp:latest \
  --dns-name-label mywebapp \
  --ports 80 \
  --azure-file-volume-account-name $STORAGE_ACCOUNT \
  --azure-file-volume-account-key $STORAGE_KEY \
  --azure-file-volume-share-name $SHARE_NAME \
  --azure-file-volume-mount-path /app/data

# Application writes to /app/data
# Data persists in Azure Files myshare
```

## Mount File Share (YAML)

### Single Volume
```yaml
apiVersion: '2021-09-01'
location: eastus
name: file-share-demo
properties:
  containers:
  - name: webapp
    properties:
      image: nginx
      ports:
      - port: 80
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      - name: filesharevolume
        mountPath: /data
  
  osType: Linux
  restartPolicy: Always
  
  ipAddress:
    type: Public
    ports:
    - port: 80
    dnsNameLabel: webapp-demo
  
  volumes:
  - name: filesharevolume
    azureFile:
      shareName: myshare
      storageAccountName: mystorageaccount
      storageAccountKey: <storage-account-key>
```

### Deploy YAML
```bash
# Deploy container group with YAML
az container create \
  --resource-group myResourceGroup \
  --file container-with-volume.yaml
```

## Mount Multiple Volumes

### Multiple File Shares to Same Container
```yaml
apiVersion: '2021-09-01'
location: eastus
name: multi-volume-demo
properties:
  containers:
  - name: myapp
    properties:
      image: myapp:latest
      resources:
        requests:
          cpu: 1.0
          memoryInGB: 1.5
      volumeMounts:
      # Mount share1 to /app/data
      - name: datavolume
        mountPath: /app/data
      # Mount share2 to /app/logs
      - name: logvolume
        mountPath: /app/logs
  
  osType: Linux
  
  volumes:
  # First volume
  - name: datavolume
    azureFile:
      shareName: share1
      storageAccountName: mystorageaccount
      storageAccountKey: <key>
  # Second volume
  - name: logvolume
    azureFile:
      shareName: share2
      storageAccountName: mystorageaccount
      storageAccountKey: <key>
```

### Multiple Containers with Different Mounts
```yaml
apiVersion: '2021-09-01'
location: eastus
name: multi-container-volumes
properties:
  containers:
  # Container 1: Write data
  - name: writer
    properties:
      image: writer:latest
      volumeMounts:
      - name: shareddata
        mountPath: /output
      resources:
        requests:
          cpu: 1
          memoryInGB: 1
  
  # Container 2: Read data
  - name: reader
    properties:
      image: reader:latest
      volumeMounts:
      - name: shareddata
        mountPath: /input
      resources:
        requests:
          cpu: 1
          memoryInGB: 1
  
  osType: Linux
  
  volumes:
  - name: shareddata
    azureFile:
      shareName: shared
      storageAccountName: mystorageaccount
      storageAccountKey: <key>
```

## Practical Examples

### Example 1: Web App with Persistent Uploads
```bash
# Web app that stores uploaded files
az container create \
  --resource-group myResourceGroup \
  --name webapp \
  --image webapp:latest \
  --dns-name-label mywebapp \
  --ports 80 443 \
  --azure-file-volume-account-name mystorageaccount \
  --azure-file-volume-account-key $STORAGE_KEY \
  --azure-file-volume-share-name uploads \
  --azure-file-volume-mount-path /app/uploads \
  --environment-variables \
    'UPLOAD_DIR'='/app/uploads'

# Users upload files → Stored in Azure Files
# Container restarts → Files still available
```

### Example 2: Log Collection
```yaml
apiVersion: '2021-09-01'
location: eastus
name: app-with-logging
properties:
  containers:
  # Application
  - name: webapp
    properties:
      image: webapp:latest
      ports:
      - port: 80
      volumeMounts:
      - name: logs
        mountPath: /var/log/app
      resources:
        requests:
          cpu: 1
          memoryInGB: 1.5
  
  # Log collector sidecar
  - name: log-collector
    properties:
      image: fluentd:latest
      volumeMounts:
      - name: logs
        mountPath: /logs
      resources:
        requests:
          cpu: 0.5
          memoryInGB: 0.5
  
  osType: Linux
  
  volumes:
  - name: logs
    azureFile:
      shareName: applogs
      storageAccountName: mystorageaccount
      storageAccountKey: <key>
```

### Example 3: Configuration Files
```bash
# Upload config file to file share first
az storage file upload \
  --account-name mystorageaccount \
  --account-key $STORAGE_KEY \
  --share-name config \
  --source config.json \
  --path config.json

# Container reads config from mounted share
az container create \
  --resource-group myResourceGroup \
  --name myapp \
  --image myapp:latest \
  --azure-file-volume-account-name mystorageaccount \
  --azure-file-volume-account-key $STORAGE_KEY \
  --azure-file-volume-share-name config \
  --azure-file-volume-mount-path /etc/config

# Application reads /etc/config/config.json
```

### Example 4: Data Processing Pipeline
```yaml
apiVersion: '2021-09-01'
location: eastus
name: data-pipeline
properties:
  containers:
  - name: processor
    properties:
      image: data-processor:latest
      volumeMounts:
      - name: input
        mountPath: /input
      - name: output
        mountPath: /output
      resources:
        requests:
          cpu: 2
          memoryInGB: 4
  
  osType: Linux
  restartPolicy: OnFailure
  
  volumes:
  # Input data from file share
  - name: input
    azureFile:
      shareName: input-data
      storageAccountName: mystorageaccount
      storageAccountKey: <key>
  # Write results to file share
  - name: output
    azureFile:
      shareName: output-data
      storageAccountName: mystorageaccount
      storageAccountKey: <key>
```

## Accessing Files

### From Container
```bash
# Inside container
ls /data
cat /data/file.txt
echo "Hello" > /data/newfile.txt
```

### From Azure Portal
1. Go to Storage Account
2. Navigate to File shares
3. Select your share
4. Browse/upload/download files

### Using Azure CLI
```bash
# List files
az storage file list \
  --account-name mystorageaccount \
  --account-key $STORAGE_KEY \
  --share-name myshare \
  --output table

# Upload file
az storage file upload \
  --account-name mystorageaccount \
  --account-key $STORAGE_KEY \
  --share-name myshare \
  --source localfile.txt \
  --path remotefile.txt

# Download file
az storage file download \
  --account-name mystorageaccount \
  --account-key $STORAGE_KEY \
  --share-name myshare \
  --path remotefile.txt \
  --dest localfile.txt
```

## Using Storage Explorer

- Установите **Azure Storage Explorer**
- Подключитесь к нужному Storage Account
- Откройте раздел File Shares
- Просматривайте, загружайте и скачивайте файлы

---

# Сравнение типов томов (Volume Types Comparison)

| Тип тома | Персистентность | Сценарий использования |
|-----------|-----------------|------------------------|
| **Azure Files** | Постоянное | Данные приложения, логи, загрузки |
| **emptyDir** | Временное | Временные файлы, кэш |
| **gitRepo** | Только чтение | Исходный код, конфигурация |
| **secret** | Чувствительные данные | Пароли, сертификаты |

---

## Что важно понимать

- **Azure Files** — сохраняет данные после перезапуска контейнера
- **emptyDir** — существует только в рамках жизненного цикла Container Group
- **gitRepo** — автоматически клонируется при старте, только для чтения
- **secret** — хранится в памяти, не записывается на диск

---

## Экзаменационный акцент (AZ-204)

- Для постоянных данных → Azure Files
- Для временных данных → emptyDir
- Для секретов → secret
- Для автоматической загрузки исходников → gitRepo


### Azure Files Volume
```yaml
volumes:
- name: persistent
  azureFile:
    shareName: myshare
    storageAccountName: mystorageaccount
    storageAccountKey: <key>
```

### Empty Directory (Temporary)
```yaml
volumes:
- name: scratch
  emptyDir: {}
```

### Git Repo
```yaml
volumes:
- name: code
  gitRepo:
    repository: https://github.com/user/repo.git
    directory: .
```

### Secret
```yaml
volumes:
- name: secrets
  secret:
    secretName: my-secret
```

## Security Considerations

### Storage Account Key
⚠️ **Storage account key** is highly sensitive:

```yaml
# ❌ BAD: Hardcoded in YAML
storageAccountKey: "abcd1234..."  # Don't commit to source control

# ✅ BETTER: Use parameter or CI/CD variable
storageAccountKey: ${STORAGE_KEY}

# ✅ BEST: Use managed identity (not yet supported for ACI file shares)
```

### Read-Only Mounts
```yaml
# Mount as read-only (prevents writes)
volumeMounts:
- name: configvolume
  mountPath: /etc/config
  readOnly: true
```

### Access Control
```bash
# Limit access with SAS token (future)
# Currently requires storage account key
```

## Troubleshooting

### Common Issues

#### Mount Fails
```bash
# Check storage account key
az storage account keys list \
  --resource-group myResourceGroup \
  --account-name mystorageaccount

# Verify file share exists
az storage share show \
  --name myshare \
  --account-name mystorageaccount \
  --account-key $STORAGE_KEY
```

#### Permission Denied
```yaml
# Ensure container runs as root
securityContext:
  runAsUser: 0  # Root user
```

#### Windows Container Error
```
# Error: Volume mount not supported for Windows containers
# Solution: Use Linux containers
osType: Linux  # Not Windows
```

## Best Practices

### 1. Separate Shares for Different Purposes
```bash
# Separate shares
az storage share create --name app-data
az storage share create --name app-logs
az storage share create --name app-uploads
```

### 2. Use Descriptive Mount Paths
```yaml
volumeMounts:
- name: datavolume
  mountPath: /app/data  # Clear purpose
- name: logvolume
  mountPath: /var/log/app  # Standard location
```

### 3. Protect Storage Keys
```bash
# Store in Key Vault
az keyvault secret set \
  --vault-name myvault \
  --name storage-key \
  --value $STORAGE_KEY

# Reference in CI/CD (don't hardcode)
```

### 4. Monitor Storage Usage
```bash
# Check file share usage
az storage share stats \
  --name myshare \
  --account-name mystorageaccount \
  --account-key $STORAGE_KEY
```

### 5. Backup Important Data
```bash
# Snapshot file share
az storage share snapshot \
  --name myshare \
  --account-name mystorageaccount \
  --account-key $STORAGE_KEY
```

# Critical Notes — Azure Files в ACI

- 💡 **Persistent storage** — данные в Azure Files сохраняются после перезапуска контейнера
- ⚠️ **Только Linux** — монтирование не поддерживается для Windows-контейнеров
- 🎯 **Требуется root** — контейнер должен запускаться от root
- ✅ **SMB/CIFS** — используется стандартный протокол SMB (не NFS)
- 📊 **Несколько монтирований** — можно подключать разные шары к разным путям
- 🔄 **Общее хранилище** — несколько контейнеров могут монтировать одну и ту же шару
- 🔒 **Storage key** — требуется ключ аккаунта хранения (чувствительные данные)
- ⚠️ **Read-only режим** — можно подключить с `readOnly: true`

---

# Exam Tips (AZ-204)

## Основы

- Azure Files — персистентное хранилище для контейнеров (SMB/CIFS)
- Поддержка только для Linux-контейнеров
- Контейнер должен работать от root

---

## Настройка

- CLI → `--azure-file-volume-*` параметры
- YAML:
    - `volumes` — определение томов
    - `volumeMounts` — подключение к контейнеру

- Можно определить несколько томов в массиве `volumes`

---

## Безопасность

- Требуется ключ storage account
- Ключ является чувствительным — хранить безопасно
- Лучше использовать отдельные шары для разных типов данных

---

## Использование

- Путь монтирования определяет, где том доступен в файловой системе контейнера
- Несколько контейнеров могут использовать одну шару
- Можно подключить том в режиме только для чтения (`readOnly: true`)

---

## Сценарии

- Данные приложения
- Логи
- Загрузки
- Конфигурация
- Совместное хранение данных

---

## Альтернативные тома

- emptyDir — временное хранилище
- gitRepo — read-only исходный код
- secret — чувствительные данные

---

## Troubleshooting

- Проверить корректность storage key
- Убедиться, что file share существует
- Контейнер должен быть Linux
- Контейнер должен запускаться от root

---

## Частые экзаменационные ловушки

- Windows ACI не поддерживает Azure Files
- Требуется root для монтирования
- Нужен storage account key
- Для постоянных данных → Azure Files


[Learn More](https://learn.microsoft.com/en-us/training/modules/create-run-container-images-azure-container-instances/6-mount-azure-file-share-azure-container-instances)
