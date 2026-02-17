# Azure Container Registry — Возможности хранения (Storage Capabilities)

## Ключевые понятия (Key Concepts)

- **Encryption-at-rest** — все образы автоматически шифруются при хранении
- **Regional storage** — данные хранятся в регионе, где создан registry
- **Geo-replication** — многорегиональный registry (только Premium)
- **Zone redundancy** — поддержка availability zones (только Premium)
- **Scalable storage** — масштабируемое хранилище (в пределах лимитов тарифа)

---

# Функции хранения

## Encryption-at-Rest (Шифрование при хранении)

**Автоматическое шифрование** всего содержимого registry:

- Образы шифруются перед записью в хранилище
- При pull выполняется автоматическая расшифровка
- Используются ключи, управляемые Azure
- Опционально: поддержка customer-managed keys (CMK)

---

## Что это означает

- Не требует дополнительной настройки для базовой защиты
- Соответствует требованиям безопасности enterprise-уровня
- Поддерживает сценарии compliance
- CMK позволяет контролировать жизненный цикл ключей

---

## Экзаменационный акцент (AZ-204)

- Шифрование включено по умолчанию
- Ключи управляются Azure
- CMK — дополнительная опция
- Geo-replication и zone redundancy доступны только в Premium


```bash
# Enable customer-managed key (Premium only)
az acr encryption set \
  --resource-group myResourceGroup \
  --name myregistry \
  --key-encryption-key <key-identifier> \
  --identity <managed-identity-id>
```

## Regional Storage (Региональное хранение)

**Data residency для соответствия требованиям compliance**

| Регион | Место хранения | Парный регион |
|---------|----------------|----------------|
| **Большинство регионов** | Основной + парный регион | Да |
| **Brazil South** | Только Brazil South | Нет |
| **Southeast Asia** | Только Southeast Asia | Нет |

⚠️ **Сбой региона**  
При региональном отказе данные могут стать недоступными и не восстанавливаются автоматически.

---

## Geo-Replication (только Premium)

### Преимущества

✅ **Высокая доступность** — защита от региональных сбоев  
✅ **Близость к сети** — более быстрый push/pull за счёт размещения рядом с потребителями  
✅ **Единый registry** — управление одним реестром в нескольких регионах  
✅ **Региональное развертывание** — деплой ближе к пользователям

---

## Что важно понимать

- Geo-replication снижает задержку при загрузке образов.
- Подходит для глобальных production-систем.
- Требует тариф Premium.
- Улучшает устойчивость к сбоям региона.

---

## Экзаменационный акцент (AZ-204)

- Нужна защита от регионального сбоя → Geo-replication (Premium).
- Требуется соблюдение data residency → учитывать регион хранения.
- Regional storage ≠ автоматическое восстановление при сбое.

### How It Works
```
Primary Registry (East US)
├── Replica (West Europe)
├── Replica (Southeast Asia)
└── Replica (Australia East)

- Single registry name: myregistry.azurecr.io
- Automatic image replication
- Regional endpoints for fast pulls
- Centralized management
```

### CLI Commands
```bash
# Enable geo-replication (Premium required)
az acr replication create \
  --resource-group myResourceGroup \
  --registry myregistry \
  --location westeurope

# List replicas
az acr replication list \
  --registry myregistry \
  --output table

# Delete replica
az acr replication delete \
  --resource-group myResourceGroup \
  --registry myregistry \
  --name westeurope
```

### Replication Workflow
```bash
# 1. Push to primary region
docker push myregistry.azurecr.io/myapp:v1.0

# 2. Automatic replication to all replicas
# myregistry-eastus.azurecr.io/myapp:v1.0
# myregistry-westeurope.azurecr.io/myapp:v1.0
# myregistry-southeastasia.azurecr.io/myapp:v1.0

# 3. Pull from nearest region automatically
docker pull myregistry.azurecr.io/myapp:v1.0
```

## Zone Redundancy (только Premium)

### Availability Zones

**Репликация registry внутри одного региона**:

- Минимум 3 изолированные зоны доступности в регионе
- Защита от отказа отдельной зоны
- Доступно только в поддерживаемых регионах

---

## Что это даёт

- Повышенную устойчивость внутри региона
- Защиту от сбоя дата-центра
- Более высокий уровень доступности для production-нагрузок

---

## Важно различать

- **Zone Redundancy** → защита внутри одного региона
- **Geo-Replication** → защита между регионами

---

## Экзаменационный акцент (AZ-204)

- Защита от сбоя зоны → Zone Redundancy (Premium)
- Защита от сбоя региона → Geo-replication (Premium)
- Обе функции доступны только в тарифе Premium


```bash
# Enable zone redundancy at creation
az acr create \
  --resource-group myResourceGroup \
  --name myregistry \
  --sku Premium \
  --zone-redundancy enabled

# Enable on existing registry
az acr update \
  --resource-group myResourceGroup \
  --name myregistry \
  --zone-redundancy enabled
```

# Scalable Storage (Масштабируемое хранилище)

## Лимиты хранения по тарифам

| Tier | Включённое хранилище | Максимум | Пропускная способность |
|------|----------------------|-----------|------------------------|
| **Basic** | 10 GB | 2 TB | Ограниченная |
| **Standard** | 100 GB | 2 TB | Средняя |
| **Premium** | 500 GB | 2 TB | Высокая |

---

## Неограниченные ресурсы

✅ **Repositories** — можно создавать неограниченное количество  
✅ **Images** — нет лимита по количеству образов  
✅ **Layers** — поддержка всех слоёв образов  
✅ **Tags** — неограниченное количество тегов в репозитории

---

## Важно учитывать

⚠️ Большое количество репозиториев и тегов может влиять на производительность:

- Замедление операций list
- Увеличение времени очистки (cleanup)
- Повышенная нагрузка на метаданные

---

## Практические рекомендации

- Используйте стратегию lifecycle management
- Удаляйте старые и неиспользуемые теги
- Автоматизируйте очистку через ACR Tasks или скрипты
- Продумывайте структуру именования репозиториев

---

## Экзаменационный акцент (AZ-204)

- Максимальный объём хранения — 2 TB
- Premium — самая высокая пропускная способность
- Количество репозиториев и тегов не ограничено
- Производительность может снижаться при большом числе тегов


### Storage Management
```bash
# Check registry usage
az acr show-usage --name myregistry --output table

# Example output:
# NAME                CURRENT    LIMIT
# ------------------  ---------  -------
# Size                5.2 GB     10 GB
# Webhooks            2          10
```

## Maintenance Best Practices

### Regular Cleanup
```bash
# Delete unused images (older than 30 days)
az acr run \
  --registry myregistry \
  --cmd "acr purge --filter 'myrepo:.*' --ago 30d" \
  /dev/null

# Delete untagged manifests
az acr manifest delete \
  --name myregistry \
  --repository myrepo \
  --manifest <manifest-digest>
```

### Retention Policies (Preview)
```bash
# Set retention policy
az acr config retention update \
  --registry myregistry \
  --status enabled \
  --days 30 \
  --type UntaggedManifests
```

### Lifecycle Management
```bash
# Create task to clean old images weekly
az acr task create \
  --registry myregistry \
  --name weekly-cleanup \
  --cmd "acr purge --filter 'myrepo:.*' --ago 30d" \
  --schedule "0 2 * * 0" \
  --context /dev/null
```

## Data Recovery

### Soft Delete (Preview)
```bash
# Enable soft delete (Premium)
az acr config soft-delete update \
  --registry myregistry \
  --status enabled \
  --days 7

# List deleted artifacts
az acr manifest list-deleted \
  --registry myregistry \
  --repository myrepo

# Restore deleted artifact
az acr manifest restore \
  --registry myregistry \
  --repository myrepo \
  --manifest <manifest-digest>
```

⚠️ **Important**: Deleted resources cannot be recovered without soft delete enabled

## Performance Optimization

### Network Proximity
**Use geo-replication** for global deployments:

```bash
# Replicate to regions where you deploy
az acr replication create --registry myregistry --location westus2
az acr replication create --registry myregistry --location eastasia
az acr replication create --registry myregistry --location northeurope

# Benefit: Automatic routing to nearest replica
```

# Throughput Limits (Ограничения пропускной способности)

| Tier | Параллельные операции | ReadOps/сек | WriteOps/сек |
|------|------------------------|-------------|--------------|
| **Basic** | 10 | 300 | 100 |
| **Standard** | 20 | 600 | 200 |
| **Premium** | 500 | 10 000 | 2 000 |

---

## Что это означает

- **Concurrent operations** — максимальное количество одновременных push/pull операций
- **ReadOps/sec** — количество операций чтения в секунду
- **WriteOps/sec** — количество операций записи в секунду
- Premium значительно превосходит Basic и Standard по производительности

---

# Optimization Tips (Рекомендации по оптимизации)

✅ **Layer caching** — используйте multi-stage builds для уменьшения количества слоёв  
✅ **Parallel pulls** — Premium поддерживает до 500 параллельных операций  
✅ **Сжатие** — Docker автоматически сжимает слои  
✅ **Региональные реплики** — размещайте registry ближе к вычислительным ресурсам

---

## Практические рекомендации

- Для high-scale production выбирайте Premium
- Минимизируйте размер образов
- Используйте общие base images
- Настраивайте geo-replication для глобальных систем

---

## Экзаменационный акцент (AZ-204)

- Premium → высокая производительность и 500 concurrent operations
- Basic → подходит для dev
- Standard → большинство production-сценариев
- Высокая нагрузка или глобальный деплой → Premium


## Cost Management

### Storage Costs
```
Basic:    $0.167/day (~$5/month) + $0.10/GB/month
Standard: $0.667/day (~$20/month) + $0.10/GB/month
Premium:  $1.667/day (~$50/month) + $0.10/GB/month

Geo-replication: Additional $0.10/GB/month per replica
```

# Cost Optimization (Оптимизация затрат)

✅ **Удаляйте неиспользуемые образы** — снижает расходы на хранение  
✅ **Используйте Basic для dev/test** — минимальная стоимость  
✅ **Оптимизируйте размер образов** — применяйте multi-stage builds  
✅ **Мониторинг использования** — команда `az acr show-usage`  
✅ **Retention policies** — автоматическое удаление старых образов

---

# Critical Notes (Критически важные моменты)

- 💡 **Encryption-at-rest** — автоматическое шифрование всех образов
- ⚠️ **Региональный сбой** — данные могут стать недоступными и не восстанавливаются автоматически
- 🎯 **Geo-replication** — только Premium, высокая доступность между регионами
- ✅ **Zone redundancy** — только Premium, минимум 3 зоны в регионе
- 📊 **Лимит хранения** — максимум 2 TB на тариф
- 🔄 **Очистка обязательна** — удаляйте старые образы для производительности
- ⚠️ **Удаление необратимо** — используйте soft delete (Preview) для восстановления
- 🔒 **CMK** — customer-managed keys для дополнительного уровня шифрования

---

# Exam Tips (AZ-204)

## Общие моменты

- Все тарифы → encryption-at-rest включено по умолчанию
- Используются ключи, управляемые Azure
- Данные хранятся в регионе registry
- В большинстве регионов есть парный регион

---

## Особые регионы

- Brazil South → данные только в одном регионе
- Southeast Asia → данные только в одном регионе

---

## Premium-функции

- Geo-replication → multi-region high availability
- Zone redundancy → минимум 3 зоны
- Soft delete → восстановление удалённых артефактов (Preview)

---

## Лимиты хранения

- Basic → 10 GB включено
- Standard → 100 GB включено
- Premium → 500 GB включено
- Максимум → 2 TB на тариф

---

## Неограниченные ресурсы (в пределах хранилища)

- Репозитории
- Образы
- Слои
- Теги

⚠️ Большое количество репозиториев и тегов может влиять на производительность.

---

## Очистка и управление

- `az acr purge`
- Retention policies
- Плановые задачи (scheduled tasks)

---

## Стоимость

- Ежедневная ставка тарифа
- ~$0.10 за GB хранения
- Дополнительная стоимость за geo-replication

---

## Throughput

- Premium → 500 параллельных операций
- Standard → 20
- Basic → 10

---

## Частые экзаменационные ловушки

- «Нужна multi-region HA» → Premium
- «Минимальная стоимость dev» → Basic
- «Защита от отказа зоны» → Zone Redundancy
- «Восстановление удалённых образов» → Soft delete (Premium, Preview)


[Learn More](https://learn.microsoft.com/en-us/training/modules/publish-container-image-to-azure-container-registry/3-azure-container-registry-storage)
