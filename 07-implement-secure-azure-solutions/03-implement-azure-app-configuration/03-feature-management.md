# Управление функциями приложения (Manage Application Features)

## Что такое Feature Management?

**Feature management** — это современный подход к разработке, который отделяет выпуск функциональности от деплоя кода.

Он позволяет включать или отключать функции по требованию без изменения или повторного развертывания приложения.

---

## Другие названия

- Feature flags
- Feature toggles
- Feature switches
- Dark launches

---

## Зачем это нужно

- 🚀 Постепенный запуск новых функций
- 🎯 Таргетирование определённых групп пользователей
- 🧪 A/B тестирование
- 🔄 Быстрое отключение проблемной функции без rollback
- ⚡ Управление релизами без простоя

---

## Важно для AZ-204

- Feature flags — встроенная возможность Azure App Configuration.
- Позволяют управлять функциональностью без redeploy.
- Частый экзаменационный сценарий:
  > Нужно включить новую функцию для части пользователей  
  → использовать Feature Flags.


---

## Почему использовать Feature Management?

### Проблема традиционного деплоя

**Без Feature Flags**:

```
Изменение кода → Сборка → Тестирование → Деплой → Функция доступна всем
```

### Проблемы

- Функция становится доступной сразу после деплоя
- Нельзя включить только для части пользователей
- Сложно откатить без повторного деплоя
- Риск, что незавершённая функциональность повлияет на всех пользователей

---

## Современный подход с Feature Flags

**С Feature Flags**:

```
Изменение кода (с флагом) → Сборка → Деплой → Управление функцией через конфигурацию
```


### Преимущества

- ✅ Код можно задеплоить с отключённой функцией
- ✅ Можно включать для отдельных пользователей или групп
- ✅ Постепенный rollout (10% → 50% → 100%)
- ✅ Мгновенный rollback без redeploy
- ✅ A/B тестирование
- ✅ Разделение деплоя и релиза

---

# Базовые понятия

## 1. Feature Flag

**Feature Flag** — это переменная с бинарным состоянием (включено / выключено), которая определяет, будет ли выполняться определённый участок кода.

---

### Что важно для AZ-204

- Feature flags позволяют управлять релизом без изменения кода.
- Поддерживаются нативно в Azure App Configuration.
- Частый сценарий:
  > Нужно включить функцию только для части пользователей  
  → использовать Feature Flag с фильтром.


**Simple Example**:
```csharp
if (featureFlag)
{
    // Run the following code
}
```
### Характеристики Feature Flag

- Бинарное состояние: `true` (включено) или `false` (выключено)
- Связан с конкретным блоком кода
- Состояние определяет, будет ли выполнен код
- Может быть статическим или динамическим

---

## 2. Feature Manager

**Feature Manager** — это библиотека или компонент приложения, который управляет жизненным циклом всех feature flags.

### Основные обязанности

- Загружает feature flags из источника конфигурации
- Определяет текущее состояние флага
- Кэширует значения для повышения производительности
- Обновляет состояния динамически
- Применяет фильтры и правила

---

### Популярные библиотеки

- **.NET**: `Microsoft.FeatureManagement`
- **Java Spring**: Spring Cloud Feature Management
- **JavaScript**: кастомные или сторонние решения
- **Python**: чаще всего кастомная реализация

---

## 3. Filter

**Filter** — это правило, определяющее, включён ли feature flag для конкретного запроса или пользователя.

---

### Типы фильтров

- **Percentage** — включение для X% пользователей
- **Targeting** — включение для конкретных пользователей или групп
- **Time window** — включение в определённый период времени
- **Geographic** — включение для конкретных регионов
- **Browser / Device** — включение для определённых устройств или браузеров
- **Custom** — собственная бизнес-логика

---

## Важно для AZ-204

- Feature Flag = бинарный переключатель.
- Feature Manager отвечает за загрузку и оценку флагов.
- Percentage и Targeting — самые часто встречающиеся фильтры в экзаменационных вопросах.
- Feature flags позволяют управлять релизом без redeploy.

## How Feature Flags Work

### Component Interaction

```
┌──────────────────────────────────────────────────────────┐
│  Application Code                                         │
│  ┌────────────────────────────────────────────────┐      │
│  │  if (await featureManager.IsEnabledAsync(      │      │
│  │         "NewCheckout"))                        │      │
│  │  {                                              │      │
│  │      // New checkout process                   │      │
│  │  }                                              │      │
│  │  else                                           │      │
│  │  {                                              │      │
│  │      // Old checkout process                   │      │
│  │  }                                              │      │
│  └─────────────────┬──────────────────────────────┘      │
│                    │                                      │
│                    ▼                                      │
│  ┌────────────────────────────────────────────────┐      │
│  │  Feature Manager                               │      │
│  │  • Loads flags from configuration              │      │
│  │  • Evaluates filters                           │      │
│  │  • Caches results                              │      │
│  │  • Returns true/false                          │      │
│  └─────────────────┬──────────────────────────────┘      │
└────────────────────┼───────────────────────────────────────┘
                     │
                     ▼
   ┌─────────────────────────────────────────┐
   │  Configuration Repository               │
   │  (Azure App Configuration)              │
   │  ┌──────────────────────────────────┐   │
   │  │  FeatureManagement:              │   │
   │  │    NewCheckout:                  │   │
   │  │      EnabledFor:                 │   │
   │  │        - Percentage: 25%         │   │
   │  │    BetaFeatures:                 │   │
   │  │      EnabledFor:                 │   │
   │  │        - Targeting: BetaUsers    │   │
   │  └──────────────────────────────────┘   │
   └─────────────────────────────────────────┘
```

---

## Feature Flag Usage in Code

### 1. Static Boolean Flag

**Simplest form**:
```csharp
bool featureFlag = true;

if (featureFlag)
{
    // Run the following code
}
```

**Use case**: Quick on/off toggle during development

### 2. Rule-Based Evaluation

**Evaluate based on logic**:
```csharp
bool featureFlag = isBetaUser();

if (featureFlag)
{
    // Beta feature code
}
```

### 3. Conditional Branching

**Different behavior based on state**:
```csharp
if (featureFlag)
{
    // This code runs if featureFlag is true
}
else
{
    // This code runs if featureFlag is false
}
```

### 4. .NET Feature Manager (Recommended)

**ASP.NET Core example**:
```csharp
using Microsoft.FeatureManagement;

public class CheckoutController : Controller
{
    private readonly IFeatureManager _featureManager;

    public CheckoutController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }

    public async Task<IActionResult> Process()
    {
        if (await _featureManager.IsEnabledAsync("NewCheckout"))
        {
            return await ProcessNewCheckout();
        }
        else
        {
            return await ProcessOldCheckout();
        }
    }
}
```

### 5. Razor View Example

**Toggle UI elements**:
```html
@inject IFeatureManager FeatureManager

@if (await FeatureManager.IsEnabledAsync("NewUI"))
{
    <div class="new-ui">
        <!-- New UI components -->
    </div>
}
else
{
    <div class="old-ui">
        <!-- Old UI components -->
    </div>
}
```

### 6. Feature Gate Attribute

**Controller-level gating**:
```csharp
[FeatureGate("BetaFeatures")]
public class BetaController : Controller
{
    // Entire controller only accessible if BetaFeatures is enabled
    
    public IActionResult Index()
    {
        return View();
    }
}
```

---

## Feature Flag Declaration

### Configuration Structure

Feature flags have two parts:
1. **Name**: Unique identifier
2. **Filters**: List of rules to evaluate state

### JSON Configuration Format

#### Simple On/Off Flags

```json
{
  "FeatureManagement": {
    "FeatureA": true,      // Always on
    "FeatureB": false      // Always off
  }
}
```

#### Percentage Rollout

```json
{
  "FeatureManagement": {
    "NewCheckout": {
      "EnabledFor": [
        {
          "Name": "Percentage",
          "Parameters": {
            "Value": 50    // Enable for 50% of users
          }
        }
      ]
    }
  }
}
```

#### Time Window Filter

```json
{
  "FeatureManagement": {
    "ChristmasSale": {
      "EnabledFor": [
        {
          "Name": "TimeWindow",
          "Parameters": {
            "Start": "2024-12-20T00:00:00Z",
            "End": "2024-12-26T23:59:59Z"
          }
        }
      ]
    }
  }
}
```

#### Targeting Filter (Specific Users/Groups)

```json
{
  "FeatureManagement": {
    "PremiumFeatures": {
      "EnabledFor": [
        {
          "Name": "Targeting",
          "Parameters": {
            "Audience": {
              "Users": [
                "user1@contoso.com",
                "user2@contoso.com"
              ],
              "Groups": [
                {
                  "Name": "PremiumUsers",
                  "RolloutPercentage": 100
                },
                {
                  "Name": "BetaTesters",
                  "RolloutPercentage": 50
                }
              ],
              "DefaultRolloutPercentage": 0
            }
          }
        }
      ]
    }
  }
}
```

#### Multiple Filters (OR Logic)

When multiple filters are present, they are evaluated in order. **First filter that returns true enables the feature**.

```json
{
  "FeatureManagement": {
    "NewFeature": {
      "EnabledFor": [
        {
          "Name": "Targeting",
          "Parameters": {
            "Audience": {
              "Users": ["admin@contoso.com"]
            }
          }
        },
        {
          "Name": "Percentage",
          "Parameters": {
            "Value": 25
          }
        }
      ]
    }
  }
}
```
### Логика оценки (Evaluation Logic)

1. Проверить, является ли пользователь `admin@contoso.com` → если да, включить (остановить проверку)
2. Проверить, попадает ли пользователь в 25% rollout → если да, включить (остановить проверку)
3. В остальных случаях — отключить

---

# Feature Flag Repository

## Почему нужно выносить Feature Flags во внешний источник?

### Проблемы «захардкоженных» флагов

- ❌ Требуют изменения кода
- ❌ Требуют redeploy
- ❌ Нельзя изменить во время работы приложения
- ❌ Нет централизованного управления

---

## Преимущества внешнего репозитория

- ✅ Изменение состояния без redeploy
- ✅ Управление функциями в реальном времени
- ✅ Централизованный контроль
- ✅ Аудит изменений
- ✅ Разные состояния для разных сред

---

# Azure App Configuration как репозиторий Feature Flags

Azure App Configuration **специально разработан** для управления feature flags.

### Возможности

- Централизованное хранение всех feature flags
- Отдельный UI для управления функциями
- Изменение состояния в реальном времени
- Поддержка различных типов фильтров
- Разделение по средам (через labels)
- Интеграция с .NET, Java Spring и другими платформами
- REST API для кастомных решений

---

## Важно для AZ-204

- Feature flags не должны быть в коде.
- App Configuration — правильный сервис для управления флагами.
- Частый сценарий:
  > Нужно управлять включением функции без redeploy  
  → использовать Azure App Configuration.

**Setup in App Configuration**:
```bash
# Create feature flag
az appconfig feature set \
  --name myappconfig \
  --feature NewCheckout \
  --label Production

# Enable feature flag
az appconfig feature enable \
  --name myappconfig \
  --feature NewCheckout \
  --label Production

# Disable feature flag
az appconfig feature disable \
  --name myappconfig \
  --feature NewCheckout \
  --label Production

# Set percentage filter
az appconfig feature filter add \
  --name myappconfig \
  --feature NewCheckout \
  --filter-name Microsoft.Percentage \
  --filter-parameters Value=50
```

---

## Practical Implementation

### Complete .NET Example

#### 1. Install NuGet Packages

```bash
dotnet add package Microsoft.FeatureManagement.AspNetCore
dotnet add package Microsoft.Extensions.Configuration.AzureAppConfiguration
```

#### 2. Configure Application

**Program.cs**:
```csharp
using Microsoft.FeatureManagement;

var builder = WebApplication.CreateBuilder(args);

// Add Azure App Configuration
builder.Configuration.AddAzureAppConfiguration(options =>
{
    options.Connect(Environment.GetEnvironmentVariable("APP_CONFIG_CONNECTION_STRING"))
           .UseFeatureFlags(featureFlagOptions =>
           {
               featureFlagOptions.CacheExpirationInterval = TimeSpan.FromMinutes(5);
           });
});

// Add feature management
builder.Services.AddFeatureManagement();

var app = builder.Build();

// Use Azure App Configuration middleware (for dynamic refresh)
app.UseAzureAppConfiguration();

app.MapGet("/checkout", async (IFeatureManager featureManager) =>
{
    if (await featureManager.IsEnabledAsync("NewCheckout"))
    {
        return Results.Ok(new { message = "New checkout process" });
    }
    else
    {
        return Results.Ok(new { message = "Old checkout process" });
    }
});

app.Run();
```

#### 3. Configure App Configuration

**Add to App Configuration**:
```json
{
  "FeatureManagement": {
    "NewCheckout": {
      "EnabledFor": [
        {
          "Name": "Microsoft.Percentage",
          "Parameters": {
            "Value": 25
          }
        }
      ]
    },
    "BetaFeatures": true,
    "MaintenanceMode": false
  }
}
```

#### 4. Use in Controllers

```csharp
[ApiController]
[Route("api/[controller]")]
public class FeaturesController : ControllerBase
{
    private readonly IFeatureManager _featureManager;

    public FeaturesController(IFeatureManager featureManager)
    {
        _featureManager = featureManager;
    }

    [HttpGet("check/{featureName}")]
    public async Task<IActionResult> CheckFeature(string featureName)
    {
        bool isEnabled = await _featureManager.IsEnabledAsync(featureName);
        return Ok(new { feature = featureName, enabled = isEnabled });
    }

    [HttpGet("beta")]
    [FeatureGate("BetaFeatures")]
    public IActionResult BetaEndpoint()
    {
        return Ok(new { message = "This is a beta feature" });
    }
}
```

---

## Common Use Cases

### 1. Gradual Rollout (Canary Release)

**Scenario**: New payment processing feature

**Implementation**:
```json
{
  "FeatureManagement": {
    "NewPaymentProcessor": {
      "EnabledFor": [
        {
          "Name": "Percentage",
          "Parameters": { "Value": 10 }
        }
      ]
    }
  }
}
```

**Rollout Plan**:
- Day 1: 10% of users
- Day 2: Monitor error rates and performance
- Day 3: Increase to 50% if metrics are good
- Day 5: Increase to 100%

**Code**:
```csharp
if (await _featureManager.IsEnabledAsync("NewPaymentProcessor"))
{
    await _newPaymentService.ProcessPayment(order);
}
else
{
    await _legacyPaymentService.ProcessPayment(order);
}
```

### 2. Beta Testing / Early Access

**Scenario**: Premium users get early access to features

**Implementation**:
```json
{
  "FeatureManagement": {
    "AdvancedReporting": {
      "EnabledFor": [
        {
          "Name": "Targeting",
          "Parameters": {
            "Audience": {
              "Groups": [
                {
                  "Name": "PremiumUsers",
                  "RolloutPercentage": 100
                }
              ]
            }
          }
        }
      ]
    }
  }
}
```

### 3. A/B Testing

**Scenario**: Test two different UI layouts

**Implementation**:
```json
{
  "FeatureManagement": {
    "LayoutVariantA": {
      "EnabledFor": [
        {
          "Name": "Percentage",
          "Parameters": { "Value": 50 }
        }
      ]
    }
  }
}
```

**Code**:
```csharp
string layout = await _featureManager.IsEnabledAsync("LayoutVariantA") 
    ? "variant-a" 
    : "variant-b";

// Track which variant user sees
_analytics.TrackVariant(userId, layout);
```

### 4. Kill Switch / Circuit Breaker

**Scenario**: Instantly disable problematic feature

**Implementation**:
```json
{
  "FeatureManagement": {
    "ExternalApiIntegration": true
  }
}
```

**Usage**:
```csharp
if (await _featureManager.IsEnabledAsync("ExternalApiIntegration"))
{
    try
    {
        await _externalApi.CallAsync();
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "External API failed");
        // If errors spike, disable feature in App Configuration
        // No code deployment needed
    }
}
else
{
    // Fallback behavior
    await _fallbackService.HandleAsync();
}
```

**In case of emergency**:
```bash
# Instantly disable feature
az appconfig feature disable --name myappconfig --feature ExternalApiIntegration
```

### 5. Scheduled Feature Release

**Scenario**: Feature goes live at specific time

**Implementation**:
```json
{
  "FeatureManagement": {
    "BlackFridaySale": {
      "EnabledFor": [
        {
          "Name": "TimeWindow",
          "Parameters": {
            "Start": "2024-11-29T00:00:00Z",
            "End": "2024-11-30T23:59:59Z"
          }
        }
      ]
    }
  }
}
```

---

## Best Practices

### 1. **Name Feature Flags Descriptively**

✅ **Good**:
```
NewCheckoutFlow
EnhancedSearchAlgorithm
PremiumReporting
```

❌ **Bad**:
```
Feature1
NewFeature
Test
```

### 2. **Start with Small Percentage**

```
10% → 25% → 50% → 100%
```

Monitor metrics at each stage before increasing.

### 3. **Remove Old Flags**

Feature flags are **temporary**. Remove them after:
- Feature is fully rolled out
- Old code path is removed
- No longer needed

**Technical debt**: Too many feature flags make code hard to maintain.

### 4. **Use Targeting for Internal Testing**

```json
{
  "FeatureManagement": {
    "ExperimentalFeature": {
      "EnabledFor": [
        {
          "Name": "Targeting",
          "Parameters": {
            "Audience": {
              "Users": ["dev-team@contoso.com"],
              "DefaultRolloutPercentage": 0
            }
          }
        }
      ]
    }
  }
}
```

### 5. **Log Feature Flag Decisions**

```csharp
bool isEnabled = await _featureManager.IsEnabledAsync("NewFeature");
_logger.LogInformation("Feature {FeatureName} evaluated to {IsEnabled} for user {UserId}",
    "NewFeature", isEnabled, userId);
```

### 6. **Environment-Specific Flags**

Use labels in App Configuration:

```bash
# Development: Enable all features
az appconfig feature enable --name myappconfig --feature NewFeature --label Development

# Production: Gradual rollout
az appconfig feature filter add \
  --name myappconfig \
  --feature NewFeature \
  --label Production \
  --filter-name Microsoft.Percentage \
  --filter-parameters Value=10
```

---

# Exam Tips — AZ-204 (Feature Management)

## Ключевые концепции

1. **Feature flags отделяют деплой от релиза**  
   Код можно задеплоить с отключённой функцией.

2. **Feature Manager**  
   Управляет жизненным циклом флагов: загрузка, проверка, кэширование, обновление.

3. **Фильтры (Filters)**  
   Определяют, включена ли функция:
    - Percentage
    - Targeting
    - Time Window

4. **Azure App Configuration**  
   Централизованный репозиторий для feature flags.

5. **Несколько фильтров = логика OR**  
   Если первый фильтр возвращает `true`, функция включается.

6. **Percentage filter**  
   Постепенный rollout для X% пользователей.

7. **Targeting filter**  
   Включение для конкретных пользователей или групп.

8. **Time Window filter**  
   Включение в определённый период времени.

9. **Feature flags временные**  
   После полного rollout их рекомендуется удалить.

10. **Dynamic refresh**  
    Приложение может применять изменения без перезапуска.

11. **.NET библиотека**  
    `Microsoft.FeatureManagement.AspNetCore`

12. **Атрибут FeatureGate**  
    Позволяет применить флаг к контроллеру или конкретному action.

---

# Частые экзаменационные сценарии

### Сценарий 1
> Выпустить новую функцию для 10% пользователей

→ Использовать Feature Flag + Percentage filter (10)

---

### Сценарий 2
> Включить функцию только для beta-тестеров

→ Использовать Targeting filter (конкретные пользователи или группы)

---

### Сценарий 3
> Нужно мгновенно отключить проблемную функцию

→ Использовать feature flag как kill switch и выключить его в App Configuration

---

### Сценарий 4
> Протестировать два варианта checkout

→ Использовать Percentage filter (50%) для A/B тестирования

---

### Сценарий 5
> Включить акцию только на Black Friday

→ Использовать Time Window filter с датами начала и окончания

---

## Главное для запоминания

- Feature flags = управление релизом без redeploy.
- Фильтры работают по логике OR.
- Azure App Configuration — основной сервис для управления флагами.
- Percentage и Targeting — самые часто встречающиеся в вопросах.

## Quick Reference

### Azure CLI Commands

```bash
# Create feature flag
az appconfig feature set --name <appconfig-name> --feature <feature-name>

# Enable feature
az appconfig feature enable --name <appconfig-name> --feature <feature-name>

# Disable feature
az appconfig feature disable --name <appconfig-name> --feature <feature-name>

# Add percentage filter
az appconfig feature filter add \
  --name <appconfig-name> \
  --feature <feature-name> \
  --filter-name Microsoft.Percentage \
  --filter-parameters Value=50

# List all features
az appconfig feature list --name <appconfig-name>

# Delete feature
az appconfig feature delete --name <appconfig-name> --feature <feature-name>
```

### .NET Code Snippets

```csharp
// Check if feature is enabled
bool isEnabled = await _featureManager.IsEnabledAsync("FeatureName");

// Feature gate attribute
[FeatureGate("FeatureName")]
public IActionResult MyAction() { }

// Conditional execution
if (await _featureManager.IsEnabledAsync("NewFeature"))
{
    // New code
}
else
{
    // Old code
}

// In Razor view
@inject IFeatureManager FeatureManager
@if (await FeatureManager.IsEnabledAsync("NewUI"))
{
    <!-- New UI -->
}
```

---

## Learn More

- [Feature Management Overview](https://docs.microsoft.com/azure/azure-app-configuration/concept-feature-management)
- [Use feature flags in ASP.NET Core](https://docs.microsoft.com/azure/azure-app-configuration/use-feature-flags-dotnet-core)
- [Feature Management Best Practices](https://docs.microsoft.com/azure/azure-app-configuration/howto-best-practices#feature-flag-best-practices)
