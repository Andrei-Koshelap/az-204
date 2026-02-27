# Выбор и настройка Availability Tests

## Обзор

Availability tests позволяют проактивно отслеживать доступность и отзывчивость вашего приложения из разных географических регионов.

Система регулярно отправляет synthetic-запросы к вашим endpoint’ам, что позволяет обнаружить проблему **до того, как её заметят реальные пользователи**.

---

## Типы Availability Tests

Application Insights поддерживает три типа тестов доступности.

---

### 1. Standard Test (Рекомендуется)

Современная замена URL Ping Test с расширенными возможностями.

### Возможности:

- Один HTTP/HTTPS-запрос
- Проверка TLS/SSL-сертификата
- Проактивная проверка срока действия сертификата
- Поддержка HTTP-методов (GET, HEAD, POST)
- Кастомные заголовки и аутентификация
- Передача данных в теле запроса
- Проверка содержимого ответа
- Запуск из нескольких регионов
- Настраиваемая частота (каждые 5–15 минут)

---

### Когда использовать:

✅ Публичные endpoint’ы  
✅ Проверка здоровья API  
✅ Мониторинг срока действия сертификата  
✅ Простые сценарии с одним запросом

📌 Это основной вариант, который следует выбирать по умолчанию (важно для AZ-204).

---

### 2. Custom TrackAvailability Test

Позволяет писать собственный код для сложных сценариев тестирования.

### Сценарии использования:

- Многошаговые workflow
- Процессы аутентификации (OAuth, SAML)
- Сложная бизнес-логика
- Внутренние endpoint’ы (недоступные публично)

Обычно реализуется через:
- Azure Functions
- WebJobs
- Фоновый сервис

---

## Ключевая идея

- Standard Test → проще, быстрее, чаще всего правильный выбор
- Custom Test → когда требуется сложная логика или доступ к внутренним ресурсам

В следующем разделе обычно рассматривается настройка тестов и конфигурация алёртов.

**Implementation:**
```csharp
using Microsoft.ApplicationInsights;
using Azure.Functions.Worker;

public class CustomAvailabilityTest
{
    private readonly TelemetryClient _telemetryClient;

    [Function("AvailabilityTest")]
    public async Task Run([TimerTrigger("0 */5 * * * *")] TimerInfo timer)
    {
        var availability = new AvailabilityTelemetry
        {
            Name = "Custom Checkout Flow Test",
            RunLocation = "EastUS-Function",
            Success = false
        };

        var stopwatch = Stopwatch.StartNew();

        try
        {
            // Step 1: Login
            var loginResponse = await LoginAsync();
            
            // Step 2: Add to cart
            var cartResponse = await AddToCartAsync(loginResponse.Token);
            
            // Step 3: Checkout
            var checkoutResponse = await CheckoutAsync(cartResponse.CartId);

            availability.Success = checkoutResponse.IsSuccessStatusCode;
        }
        catch (Exception ex)
        {
            availability.Message = ex.Message;
        }
        finally
        {
            stopwatch.Stop();
            availability.Duration = stopwatch.Elapsed;
            availability.Timestamp = DateTimeOffset.UtcNow;
            
            _telemetryClient.TrackAvailability(availability);
        }
    }
}
```

### 3. URL Ping Test (Classic) - Retiring Sept 2026

**⚠️ Deprecated**: Migrate to Standard tests before September 30, 2026.

## Creating Standard Availability Tests

### Azure Portal

```
1. Navigate to Application Insights resource
2. Select "Availability" in left menu
3. Click "+ Add Standard test"
4. Configure test:
   • Test name: "Homepage Availability"
   • URL: https://www.contoso.com
   • Test frequency: 5 minutes
   • Test locations: Select 5+ locations
   • Success criteria: HTTP 200, Response time < 5s
   • Alerts: Enable
5. Click "Create"
```

### Azure CLI

```bash
# Create standard availability test
az monitor app-insights web-test create \
  --resource-group MyResourceGroup \
  --name "Homepage-Test" \
  --location "eastus" \
  --web-test-name "homepage-standard-test" \
  --web-test-kind "standard" \
  --locations \
    "us-ca-sjc-azr" \
    "us-va-ash-azr" \
    "emea-nl-ams-azr" \
    "apac-sg-sin-azr" \
    "apac-hk-hkn-azr" \
  --frequency 300 \
  --timeout 30 \
  --enabled true \
  --synthetic-monitor-id "homepage-availability" \
  --request-url "https://www.contoso.com" \
  --expected-http-status-code 200 \
  --ssl-check true \
  --ssl-cert-remaining-lifetime-check 7 \
  --defined-tags "Environment=Production" "Owner=DevOps"
```

### Параметры конфигурации

| Параметр | Описание | Рекомендуемое значение |
|-----------|------------|------------------------|
| **Test Frequency** | Как часто запускать тест | 5 минут (production) |
| **Test Locations** | Географические точки запуска | 5+ регионов |
| **Success Criteria** | Условия успешного прохождения | HTTP 200, < 5 секунд |
| **Alerts** | Включить оповещения | Да (если < 3 регионов дали сбой) |
| **Timeout** | Таймаут запроса | 30 секунд |
| **Parse dependent requests** | Загружать ресурсы страницы | Нет (быстрее и дешевле) |
| **Enable retries** | Повтор при ошибке | Да (уменьшает ложные срабатывания) |

---

## Локации тестирования

Application Insights предоставляет глобальные точки запуска тестов.

### Северная Америка:
- us-ca-sjc-azr (West US — Калифорния)
- us-va-ash-azr (East US — Вирджиния)
- us-tx-sn1-azr (South Central US — Техас)
- us-il-ch1-azr (Central US — Иллинойс)
- us-fl-mia-azr (East US 2 — Флорида)

### Европа:
- emea-nl-ams-azr (West Europe — Нидерланды)
- emea-gb-db3-azr (UK South — Лондон)
- emea-fr-pra-azr (France Central — Париж)
- emea-ch-zrh-azr (Switzerland North — Цюрих)

### Азиатско-Тихоокеанский регион:
- apac-sg-sin-azr (Southeast Asia — Сингапур)
- apac-hk-hkn-azr (East Asia — Гонконг)
- apac-jp-kaw-azr (Japan East — Токио)
- apac-au-syd-azr (Australia East — Сидней)

---

## Рекомендация

Выбирайте 5 и более локаций из разных регионов для:

- Проверки глобальной доступности
- Исключения локальных сетевых проблем
- Повышения точности SLA-мониторинга

---

## Экзаменационный акцент (AZ-204)

Если требуется:
- Минимизировать ложные алёрты → включить retries
- Глобальный сервис → выбрать регионы из разных частей мира
- Быстрое обнаружение проблем → частота 5 минут
- Контроль частичных сбоев → алёрт при падении из нескольких регионов

Главная идея: Availability Tests должны быть геораспределёнными и настроены с разумными порогами.

## Advanced Configuration

### Custom Headers

```bash
# Add authentication header
az monitor app-insights web-test create \
  --name "API-Test-Auth" \
  --request-url "https://api.contoso.com/health" \
  --request-headers "Authorization=Bearer <token>" \
  --request-headers "X-API-Key=<api-key>" \
  ...
```

### Content Validation

```bash
# Validate response contains specific text
az monitor app-insights web-test create \
  --name "Content-Validation-Test" \
  --request-url "https://api.contoso.com/status" \
  --content-validation "Status\":\"Healthy" \
  --content-match-must-be-present true \
  ...
```

### POST Request with Body

```bash
# Send POST with JSON body
az monitor app-insights web-test create \
  --name "API-POST-Test" \
  --request-url "https://api.contoso.com/validate" \
  --http-verb "POST" \
  --request-body '{"check":"health"}' \
  --request-headers "Content-Type=application/json" \
  ...
```

## Alert Configuration

### Create Availability Alert

```bash
# Create alert rule
az monitor metrics alert create \
  --name "Low Availability Alert" \
  --resource-group MyResourceGroup \
  --scopes "/subscriptions/{sub-id}/resourceGroups/MyRG/providers/Microsoft.Insights/webtests/Homepage-Test" \
  --condition "avg availabilityResults/availabilityPercentage < 80" \
  --description "Alert when availability drops below 80%" \
  --window-size 5m \
  --evaluation-frequency 1m \
  --severity 2 \
  --action-group-ids "/subscriptions/{sub-id}/resourceGroups/MyRG/providers/microsoft.insights/actionGroups/EmailAdmins"
```

### Alert Thresholds

**Recommended Settings:**
```
Critical (Sev 0): < 2 locations passing (immediate escalation)
Warning (Sev 2):  < 3 locations passing (investigate)
Info (Sev 4):     < 4 locations passing (monitor)
```

### Action Groups

```bash
# Create action group with email and SMS
az monitor action-group create \
  --name "EmailAdmins" \
  --resource-group MyResourceGroup \
  --short-name "EmailDev" \
  --email-receiver \
    name="DevOps Team" \
    email-address="devops@contoso.com" \
    use-common-alert-schema=true \
  --sms-receiver \
    name="On-Call Engineer" \
    country-code="1" \
    phone-number="5551234567" \
  --webhook-receiver \
    name="Slack Webhook" \
    service-uri="https://hooks.slack.com/services/..." \
    use-common-alert-schema=true
```

## Monitoring Availability Results

### Availability Dashboard

```kusto
// Query availability results
availabilityResults
| where timestamp > ago(24h)
| summarize 
    AvailabilityPct = 100.0 * count(success == true) / count(),
    AvgDuration = avg(duration),
    FailureCount = countif(success == false)
    by name, location
| order by AvailabilityPct asc
```

**Example Output:**
```
Test Name         Location        Availability  AvgDuration  Failures
Homepage-Test     us-ca-sjc-azr   100%          245ms        0
Homepage-Test     emea-nl-ams-azr 98.6%         890ms        4
Homepage-Test     apac-sg-sin-azr 95.2%         1240ms       14
API-Health-Test   us-va-ash-azr   87.3%         520ms        37
```

### Failure Analysis

```kusto
// Analyze failures
availabilityResults
| where timestamp > ago(7d)
| where success == false
| summarize 
    FailureCount = count(),
    AvgDuration = avg(duration),
    sample_message = any(message)
    by name, location, resultCode
| order by FailureCount desc
```

## Лучшие практики

✅ **Частота тестирования**:
- 5 минут — для production
- 15 минут — для некритичных систем

✅ **Несколько регионов**:  
Используйте 5+ локаций, чтобы снизить риск ложных срабатываний.

✅ **Настройка алёртов**:  
Триггерить алёрт, если успешно проходит менее 3 локаций (а не при единичном сбое).

✅ **Timeout**:  
Устанавливайте реалистичный таймаут (5–30 секунд в зависимости от endpoint’а).

✅ **Проверка SSL**:  
Включайте проверку сертификата для мониторинга срока его действия.

✅ **Retries**:  
Включайте повторные попытки, чтобы уменьшить ложные срабатывания.

✅ **Проверка содержимого ответа**:  
Убедитесь, что ответ содержит ожидаемые данные (не только HTTP 200).

✅ **Не парсить зависимые ресурсы**:  
Отключайте загрузку зависимых ресурсов (быстрее и дешевле).

---

## Основные выводы

✅ **Standard tests** — рекомендуемый тип (URL Ping выводится из эксплуатации в 2026 году)

✅ **5+ регионов** обеспечивают надёжное покрытие и снижают ложные срабатывания

✅ **Частота 5 минут** — оптимальный баланс между стоимостью и скоростью реакции

✅ **Custom TrackAvailability** — для сложных многошаговых сценариев

✅ **Порог алёрта**: менее 3 успешных локаций — признак реальной проблемы

✅ **Проверка SSL** помогает отслеживать истечение сертификата

---

## Советы для экзамена AZ-204

💡 **Standard Test — почти всегда правильный ответ**  
URL Ping считается устаревшим.

💡 **Несколько регионов обязательны**  
Один регион может давать локальные сбои.

💡 **Custom TrackAvailability**  
Выбирается, если требуется аутентификация или многошаговый сценарий.

💡 **Частота 5 минут**  
Стандартное и рекомендуемое значение для production.

💡 **Алёрты должны учитывать несколько регионов**  
Не настраивайте алёрт на единичный сбой — это частая экзаменационная ловушка.

---

### Экзаменационный акцент

Availability Tests — это synthetic monitoring.  
Главная цель — обнаружить проблему **до** того, как её увидят пользователи.

В вопросах выбирайте решение, которое:
- геораспределено
- устойчиво к ложным срабатываниям
- минимально усложняет архитектуру
- использует Standard Test по умолчанию
---

**📚 Further Reading:**
- [Availability tests overview](https://learn.microsoft.com/en-us/azure/azure-monitor/app/availability-overview)
- [Standard tests](https://learn.microsoft.com/en-us/azure/azure-monitor/app/availability-standard-tests)
- [Custom TrackAvailability](https://learn.microsoft.com/en-us/azure/azure-monitor/app/availability-azure-functions)
