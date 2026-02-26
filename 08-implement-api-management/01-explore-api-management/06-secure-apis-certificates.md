# Защита API с использованием сертификатов

## Обзор

**Аутентификация на основе сертификатов** (также известная как **TLS mutual authentication** или **mTLS**) обеспечивает высокий уровень безопасности, требуя от клиента предоставления действительного цифрового сертификата при обращении к API.

**Назначение**: защита API с использованием криптографических сертификатов в сценариях с повышенным уровнем доверия.

---

## Что такое mTLS?

В стандартном HTTPS:

- Клиент проверяет сертификат сервера
- Сервер не проверяет сертификат клиента

В **mTLS (mutual TLS)**:

- Клиент проверяет сертификат сервера
- Сервер проверяет сертификат клиента

Обе стороны аутентифицируют друг друга.

---

## Когда используется certificate-based authentication

- B2B-интеграции
- Внутренние корпоративные API
- Банковские и финансовые системы
- Государственные сервисы
- Высокозащищённые microservice-to-microservice коммуникации

---

## Почему это безопаснее, чем subscription key

- Используется криптография и PKI
- Сертификат сложно подделать
- Нет передачи секретного ключа в заголовках
- Поддерживается аппаратное хранение ключей (HSM)
- Можно проверять issuer, subject, thumbprint

---

## Архитектурное значение

mTLS обеспечивает:

- сильную аутентификацию клиента;
- защиту от MITM-атак;
- централизованное управление доверием через CA;
- возможность ограничения доступа по сертификату.

В Azure API Management проверка клиентского сертификата выполняется на уровне gateway до передачи запроса в backend.

---

## Важно для AZ-204

Если в вопросе говорится о:
- взаимной аутентификации (mutual authentication),
- проверке клиентского сертификата,
- повышенных требованиях безопасности,
- B2B или high-trust интеграциях,

— правильное решение связано с использованием certificate-based authentication (mTLS).

---

## What is TLS Mutual Authentication?

In standard TLS (HTTPS), only the **server** presents a certificate to the client. In **mutual TLS**, both the **client and server** exchange certificates.

### Standard TLS (One-Way)

```
┌────────┐                    ┌────────┐
│ Client │                    │ Server │
└───┬────┘                    └───┬────┘
    │                             │
    │  1. ClientHello             │
    │─────────────────────────────>│
    │                             │
    │  2. ServerHello             │
    │  + Server Certificate       │
    │<─────────────────────────────│
    │                             │
    │  3. Client verifies cert    │
    │                             │
    │  4. Encrypted communication │
    │<────────────────────────────>│
```

### Mutual TLS (Two-Way)

```
┌────────┐                    ┌────────┐
│ Client │                    │ Server │
└───┬────┘                    └───┬────┘
    │                             │
    │  1. ClientHello             │
    │─────────────────────────────>│
    │                             │
    │  2. ServerHello             │
    │  + Server Certificate       │
    │  + Request Client Cert      │
    │<─────────────────────────────│
    │                             │
    │  3. Client Certificate      │
    │─────────────────────────────>│
    │                             │
    │  4. Server verifies cert    │
    │                             │
    │  5. Encrypted communication │
    │<────────────────────────────>│
```

## Преимущества

- ✅ **Сильная аутентификация** (криптографическое подтверждение личности)
- ✅ **Non-repudiation** (клиент не может отрицать факт запроса)
- ✅ Поддержка **отзыва сертификатов** (CRL / OCSP)
- ✅ Отсутствие управления паролями
- ✅ Подходит для **machine-to-machine (M2M)** взаимодействия

---

### Пояснение

**Сильная аутентификация**  
Подтверждение личности происходит с использованием закрытого ключа клиента. Это значительно безопаснее, 
чем передача секретов в заголовках.

**Non-repudiation**  
Так как запрос подписывается криптографически, клиент не может отрицать факт взаимодействия.

**Отзыв сертификатов**  
При компрометации сертификат можно отозвать через CA без изменения backend-кода.

**Нет паролей**  
Исключаются риски, связанные с хранением, утечкой и повторным использованием паролей.

**M2M-сценарии**  
Идеально подходит для автоматизированных интеграций между сервисами, где нет пользователя, но требуется высокий уровень доверия.

---

### Архитектурное значение

Certificate-based authentication особенно эффективна в:

- B2B-интеграциях
- Закрытых корпоративных сетях
- Высоконагруженных системах
- Микросервисных архитектурах с повышенными требованиями к безопасности

---

### Важно для AZ-204

Если в вопросе упоминается:
- mutual TLS,
- подтверждение личности через сертификат,
- высокозащищённые интеграции,
- M2M-коммуникации,

— правильным решением будет использование аутентификации на основе сертификатов (mTLS).

---

## Certificate Properties

APIM validates certificates using these properties:

### 1. **Certificate Authority (CA)**

The **CA** is the entity that issued the certificate.

**Validation**:
- Check if certificate is signed by trusted CA
- Verify chain of trust to root CA

**Example**:
```
Root CA: DigiCert Global Root CA
Intermediate CA: DigiCert SHA2 Secure Server CA
Certificate: api-client.contoso.com
```

### 2. **Thumbprint (Fingerprint)**

**Thumbprint** — это SHA-1 хэш сертификата (уникальный идентификатор).

**Формат**:  
40-символьная шестнадцатеричная строка

**Пример**:  
`A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0`

**Сценарий использования**:  
Whitelist конкретных сертификатов (разрешение доступа только определённым клиентам).

---

### 3. **Subject**

**Subject** — это информация о владельце сертификата.

**Формат**: Distinguished Name (DN)

**Пример**:  
`CN=api-client.contoso.com, O=Contoso, C=US`

**Компоненты**:

- **CN (Common Name)** — идентификатор клиента
- **O (Organization)** — название организации
- **C (Country)** — код страны
- **OU (Organizational Unit)** — подразделение

Subject часто используется для:

- проверки принадлежности сертификата конкретной организации;
- реализации условной логики на основе DN;
- дополнительной валидации клиента.

---

### 4. **Expiration Date**

Сертификаты имеют срок действия:

- **Not Before** — дата начала действия
- **Not After** — дата окончания действия

**Валидация**: текущая дата должна находиться в пределах периода действия.

Истёкший сертификат автоматически считается недействительным.

---

## Включение Client Certificates

### Тарифы Consumption и Developer

В большинстве тарифов поддержка клиентских сертификатов **включена по умолчанию**.

---

### Только для Consumption Tier

В тарифе **Consumption** необходимо явно включить переговоры клиентского сертификата.

**Через Azure Portal**:

1. Перейти в экземпляр API Management
2. Открыть **Settings** → **Custom domains**
3. Включить опцию **Negotiate client certificate**

---

### Архитектурный момент

Если negotiation не включён, gateway не запросит клиентский сертификат, даже если политика настроена.

---

### Важно для AZ-204

Если в вопросе говорится о:
- mutual TLS в Consumption tier,
- необходимости включить клиентские сертификаты,
- настройке certificate negotiation,

— требуется включить **Negotiate client certificate** в настройках APIM.
**Azure CLI**:
```bash
az apim update \
  --name apim-instance \
  --resource-group rg-apim \
  --enable-client-certificate true
```

**ARM Template**:
```json
{
  "type": "Microsoft.ApiManagement/service",
  "apiVersion": "2021-08-01",
  "name": "apim-instance",
  "properties": {
    "hostnameConfigurations": [
      {
        "type": "Proxy",
        "negotiateClientCertificate": true
      }
    ]
  }
}
```

---

## Certificate Validation in Policies

### 1. Check Certificate Thumbprint

Validate that the client certificate matches a specific thumbprint.

**Policy**:
```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Thumbprint != "desired-thumbprint-value")">
        <return-response>
          <set-status code="403" reason="Invalid client certificate" />
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

**Example with Specific Thumbprint**:
```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Thumbprint != "A1B2C3D4E5F6G7H8I9J0K1L2M3N4O5P6Q7R8S9T0")">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>{"error": "Invalid or missing client certificate"}</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### 2. Check Against Uploaded Certificates

Validate that the client certificate matches one of the certificates uploaded to APIM.

**Upload Certificate to APIM**:
```bash
# Azure Portal: API Management → Certificates → Add
# Or use Azure CLI:
az apim certificate create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --certificate-id client-cert \
  --data @certificate.pfx \
  --password "cert-password"
```

**Policy**:
```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Certificate == null || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>{"error": "Certificate not recognized"}</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

**Explanation**:
- `context.Request.Certificate`: Client certificate from request
- `context.Deployment.Certificates`: Certificates uploaded to APIM
- Checks if client cert thumbprint matches any uploaded cert

### 3. Check Certificate Issuer and Subject

Validate specific certificate properties.

**Policy**:
```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Issuer != "CN=My CA, O=Contoso, C=US" || context.Request.Certificate.SubjectName.Name != "CN=api-client.contoso.com, O=Contoso, C=US")">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>{"error": "Certificate validation failed"}</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### 4. Check Certificate Expiration

Ensure certificate is not expired.

**Policy**:
```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Certificate == null)">
        <return-response>
          <set-status code="403" reason="Certificate required" />
          <set-body>{"error": "Client certificate is required"}</set-body>
        </return-response>
      </when>
      <when condition="@(context.Request.Certificate.NotBefore > DateTime.UtcNow || context.Request.Certificate.NotAfter < DateTime.UtcNow)">
        <return-response>
          <set-status code="403" reason="Certificate expired or not yet valid" />
          <set-body>{"error": "Certificate is expired or not yet valid"}</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### 5. Validate Certificate Chain (CA Trust)

Check if certificate is signed by a trusted CA.

**Policy**:
```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Verify())">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>{"error": "Certificate validation failed - not trusted"}</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

**Note**: `context.Request.Certificate.Verify()` validates the certificate chain against the system's trusted root CAs.

---

## Complete Certificate Validation Policy

Here's a comprehensive policy combining multiple validations:

```xml
<policies>
  <inbound>
    <!-- 1. Check if certificate is present -->
    <choose>
      <when condition="@(context.Request.Certificate == null)">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-header name="Content-Type" exists-action="override">
            <value>application/json</value>
          </set-header>
          <set-body>@{
            return new JObject(
              new JProperty("error", "Client certificate required"),
              new JProperty("message", "Please provide a valid client certificate")
            ).ToString();
          }</set-body>
        </return-response>
      </when>
    </choose>
    
    <!-- 2. Validate certificate chain (CA trust) -->
    <choose>
      <when condition="@(!context.Request.Certificate.Verify())">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>@{
            return new JObject(
              new JProperty("error", "Certificate validation failed"),
              new JProperty("message", "Certificate is not trusted")
            ).ToString();
          }</set-body>
        </return-response>
      </when>
    </choose>
    
    <!-- 3. Check certificate expiration -->
    <choose>
      <when condition="@(context.Request.Certificate.NotBefore > DateTime.UtcNow || context.Request.Certificate.NotAfter < DateTime.UtcNow)">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>@{
            return new JObject(
              new JProperty("error", "Certificate expired"),
              new JProperty("notBefore", context.Request.Certificate.NotBefore),
              new JProperty("notAfter", context.Request.Certificate.NotAfter),
              new JProperty("currentTime", DateTime.UtcNow)
            ).ToString();
          }</set-body>
        </return-response>
      </when>
    </choose>
    
    <!-- 4. Validate thumbprint against uploaded certificates -->
    <choose>
      <when condition="@(!context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>@{
            return new JObject(
              new JProperty("error", "Certificate not recognized"),
              new JProperty("thumbprint", context.Request.Certificate.Thumbprint)
            ).ToString();
          }</set-body>
        </return-response>
      </when>
    </choose>
    
    <!-- 5. Log certificate details -->
    <trace source="certificate-auth" severity="information">
      @{
        return new JObject(
          new JProperty("thumbprint", context.Request.Certificate.Thumbprint),
          new JProperty("subject", context.Request.Certificate.SubjectName.Name),
          new JProperty("issuer", context.Request.Certificate.Issuer),
          new JProperty("notBefore", context.Request.Certificate.NotBefore),
          new JProperty("notAfter", context.Request.Certificate.NotAfter)
        ).ToString();
      }
    </trace>
    
    <!-- 6. Set custom header with certificate info -->
    <set-header name="X-Client-Certificate-Subject" exists-action="override">
      <value>@(context.Request.Certificate.SubjectName.Name)</value>
    </set-header>
    
    <base />
  </inbound>
</policies>
```

---

## Свойства сертификата в Context

Доступ к данным клиентского сертификата осуществляется через:

`context.Request.Certificate`

Это позволяет выполнять дополнительную валидацию и реализовывать условную логику в policies.

---

| Свойство | Описание | Пример |
|------------|------------|---------|
| `Thumbprint` | SHA-1 хэш сертификата | `A1B2C3...` |
| `Subject` | Subject сертификата (DN) | `CN=client.contoso.com` |
| `SubjectName.Name` | Полное имя субъекта | `CN=client, O=Contoso, C=US` |
| `Issuer` | Издатель сертификата | `CN=My CA, O=Contoso` |
| `NotBefore` | Дата начала действия | `DateTime` |
| `NotAfter` | Дата окончания действия | `DateTime` |
| `SignatureAlgorithm` | Алгоритм подписи | `sha256RSA` |
| `Version` | Версия сертификата | `3` |
| `SerialNumber` | Серийный номер сертификата | `01:23:45:67:89:AB` |
| `Verify()` | Проверка цепочки доверия | `bool` |

---

### Что важно понимать

С помощью `context.Request.Certificate` можно:

- Проверять конкретный `Thumbprint` (whitelisting)
- Сравнивать `Subject` или `Issuer`
- Проверять срок действия (`NotBefore`, `NotAfter`)
- Убедиться, что сертификат подписан доверенным CA (`Verify()`)

Если сертификат отсутствует, объект будет `null`.

---

### Архитектурное значение

Использование certificate context позволяет:

- реализовать строгую аутентификацию;
- ограничивать доступ по конкретным сертификатам;
- применять разные правила для разных клиентов;
- усиливать безопасность без изменения backend.

---

### Важно для AZ-204

Если в вопросе говорится о:
- проверке thumbprint,
- валидации issuer,
- проверке срока действия,
- использовании `context.Request.Certificate`,

— решение связано с проверкой свойств клиентского сертификата в policy.
### Example: Log Certificate Details

```xml
<policies>
  <inbound>
    <trace source="cert-info">
      @{
        var cert = context.Request.Certificate;
        return new JObject(
          new JProperty("thumbprint", cert?.Thumbprint ?? "none"),
          new JProperty("subject", cert?.SubjectName.Name ?? "none"),
          new JProperty("issuer", cert?.Issuer ?? "none"),
          new JProperty("notBefore", cert?.NotBefore.ToString() ?? "none"),
          new JProperty("notAfter", cert?.NotAfter.ToString() ?? "none"),
          new JProperty("serialNumber", cert?.SerialNumber ?? "none")
        ).ToString();
      }
    </trace>
    <base />
  </inbound>
</policies>
```

---

## Client Certificate Usage Examples

### cURL

```bash
curl -X GET https://apim-instance.azure-api.net/api/users \
  --cert client-cert.pem \
  --key client-key.pem
```

**With PFX/PKCS12**:
```bash
curl -X GET https://apim-instance.azure-api.net/api/users \
  --cert client-cert.pfx:password
```

### C# (HttpClient)

```csharp
using System.Net.Http;
using System.Security.Cryptography.X509Certificates;

var handler = new HttpClientHandler();
handler.ClientCertificates.Add(new X509Certificate2("client-cert.pfx", "password"));

using var client = new HttpClient(handler);
var response = await client.GetAsync("https://apim-instance.azure-api.net/api/users");
var content = await response.Content.ReadAsStringAsync();
```

### Python (requests)

```python
import requests

response = requests.get(
    'https://apim-instance.azure-api.net/api/users',
    cert=('client-cert.pem', 'client-key.pem')
)
print(response.json())
```

### JavaScript (Node.js with https)

```javascript
const https = require('https');
const fs = require('fs');

const options = {
  hostname: 'apim-instance.azure-api.net',
  port: 443,
  path: '/api/users',
  method: 'GET',
  cert: fs.readFileSync('client-cert.pem'),
  key: fs.readFileSync('client-key.pem')
};

const req = https.request(options, (res) => {
  res.on('data', (d) => {
    process.stdout.write(d);
  });
});

req.end();
```

### PowerShell

```powershell
$cert = Get-PfxCertificate -FilePath "client-cert.pfx"

Invoke-RestMethod -Uri "https://apim-instance.azure-api.net/api/users" `
  -Method GET `
  -Certificate $cert
```

---

## Управление сертификатами

### Загрузка сертификата в APIM

**Через Azure Portal**:

1. Перейти в экземпляр API Management
2. Открыть раздел **Certificates**
3. Нажать **+ Add**
4. Загрузить файл формата **PFX** и указать пароль

---

### Что важно понимать

- Загружается именно файл **PFX**, так как он содержит приватный ключ.
- Пароль защищает приватный ключ внутри файла.
- После загрузки сертификат можно использовать:
    - для mTLS-аутентификации клиентов;
    - для исходящих вызовов к backend (client certificate authentication);
    - для настройки custom domain (TLS).

---

### Архитектурный аспект

В Azure API Management сертификаты могут использоваться для:

- входящей аутентификации (client certificates);
- исходящей аутентификации к backend;
- защиты пользовательских доменов;
- интеграции с Key Vault (рекомендуемый способ в production).

Для production-сценариев предпочтительно хранить сертификаты в **Azure Key Vault**, а не загружать их вручную.

---

### Важно для AZ-204

Если в вопросе говорится о:
- загрузке клиентского сертификата,
- использовании PFX-файла,
- необходимости указать пароль,
- интеграции с Key Vault,

— речь идёт о разделе **Certificates** в Azure API Management.

**Azure CLI**:
```bash
az apim certificate create \
  --resource-group rg-apim \
  --service-name apim-instance \
  --certificate-id my-client-cert \
  --data @client-cert.pfx \
  --password "cert-password"
```

### List Certificates

```bash
az apim certificate list \
  --resource-group rg-apim \
  --service-name apim-instance
```

### Show Certificate

```bash
az apim certificate show \
  --resource-group rg-apim \
  --service-name apim-instance \
  --certificate-id my-client-cert
```

### Delete Certificate

```bash
az apim certificate delete \
  --resource-group rg-apim \
  --service-name apim-instance \
  --certificate-id my-client-cert
```

---

## Common Scenarios

### Scenario 1: Partner Integration (Single Certificate)

**Requirement**: Partner system uses one certificate

**Solution**:
```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Certificate == null || context.Request.Certificate.Thumbprint != "A1B2C3...")">
        <return-response>
          <set-status code="403" reason="Forbidden" />
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### Scenario 2: Multiple Partners (Whitelist)

**Requirement**: Multiple partners, each with own certificate

**Solution**:
1. Upload all partner certificates to APIM
2. Use policy to check against uploaded certificates

```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Certificate == null || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))">
        <return-response>
          <set-status code="403" reason="Forbidden" />
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### Scenario 3: Corporate CA

**Requirement**: Accept certificates from corporate CA only

**Solution**:
```xml
<policies>
  <inbound>
    <choose>
      <when condition="@(context.Request.Certificate == null || !context.Request.Certificate.Issuer.Contains("CN=Contoso Corporate CA"))">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>{"error": "Certificate must be issued by Contoso Corporate CA"}</set-body>
        </return-response>
      </when>
      <when condition="@(!context.Request.Certificate.Verify())">
        <return-response>
          <set-status code="403" reason="Forbidden" />
          <set-body>{"error": "Certificate validation failed"}</set-body>
        </return-response>
      </when>
    </choose>
    <base />
  </inbound>
</policies>
```

### Scenario 4: Hybrid Authentication (Cert + Subscription Key)

**Requirement**: Require both certificate AND subscription key

**Solution**:
```xml
<policies>
  <inbound>
    <!-- Check certificate -->
    <choose>
      <when condition="@(context.Request.Certificate == null || !context.Deployment.Certificates.Any(c => c.Value.Thumbprint == context.Request.Certificate.Thumbprint))">
        <return-response>
          <set-status code="403" reason="Invalid certificate" />
        </return-response>
      </when>
    </choose>
    
    <!-- Check subscription key -->
    <check-header name="Ocp-Apim-Subscription-Key" failed-check-httpcode="401" />
    
    <base />
  </inbound>
</policies>
```

---

## Best Practices

### 1. **Enable Client Certificates in Consumption Tier**

✅ **Do**: Explicitly enable in Consumption tier
```bash
az apim update --enable-client-certificate true
```

### 2. **Upload Certificates to APIM**

✅ **Do**: Upload trusted certificates to APIM for validation
```bash
az apim certificate create --data @cert.pfx
```

### 3. **Validate Certificate Chain**

✅ **Do**: Use `context.Request.Certificate.Verify()`
```xml
<when condition="@(!context.Request.Certificate.Verify())">
  <return-response>...</return-response>
</when>
```

### 4. **Check Expiration**

✅ **Do**: Validate NotBefore and NotAfter
```xml
<when condition="@(cert.NotAfter < DateTime.UtcNow)">
```

### 5. **Log Certificate Details**

✅ **Do**: Log for auditing
```xml
<trace source="cert-auth">
  @(context.Request.Certificate.Thumbprint)
</trace>
```

### 6. **Use Thumbprint Validation**

✅ **Do**: Validate specific certificates by thumbprint
```xml
<when condition="@(cert.Thumbprint != "expected-thumbprint")">
```

### 7. **Return Meaningful Error Messages**

✅ **Do**: Provide clear error messages
```json
{
  "error": "Certificate validation failed",
  "reason": "Certificate not recognized",
  "thumbprint": "A1B2C3..."
}
```

---

## Советы к экзамену

### Ключевые концепции для AZ-204

1. **Mutual TLS (mTLS)**  
   И клиент, и сервер предъявляют сертификаты для взаимной аутентификации.

2. **Свойства сертификата**  
   Проверяются: CA (Issuer), Thumbprint, Subject, срок действия (Expiration).

3. **Consumption tier**  
   Требуется явно включить negotiation клиентских сертификатов.

4. **Загрузка сертификатов**  
   Доверенные сертификаты необходимо загрузить в APIM (или подключить через Key Vault).

5. **Валидация в политиках**  
   Используется `context.Request.Certificate` в policy-выражениях.

6. **Проверка thumbprint**  
   Можно сравнивать с конкретным thumbprint или с загруженными доверенными сертификатами.

7. **Проверка цепочки доверия**  
   Используется метод `Verify()` для валидации сертификата.

8. **Раздел inbound**  
   Проверка сертификата выполняется в `inbound` policies до вызова backend.

9. **HTTP 403**  
   Если сертификат недействителен — возвращается `403 Forbidden`.

---

### Финальный акцент для AZ-204

- mTLS применяется в high-trust и B2B-сценариях.
- В Consumption tier необходимо вручную включить поддержку клиентских сертификатов.
- Проверка сертификата выполняется через `context.Request.Certificate` в разделе `inbound`.
- Невалидный сертификат → `403 Forbidden`, запрос не передаётся в backend.
- Для продакшена предпочтительно хранить сертификаты в Key Vault.
10. **Context properties**:
    - `context.Request.Certificate.Thumbprint`
    - `context.Request.Certificate.Issuer`
    - `context.Request.Certificate.SubjectName.Name`
    - `context.Request.Certificate.NotBefore`
    - `context.Request.Certificate.NotAfter`
    - `context.Request.Certificate.Verify()`

### Частые экзаменационные сценарии

**Сценарий 1**:  
"Защитить API с помощью аутентификации на основе сертификатов"  
→ **Ответ**: Включить client certificates и использовать policy для проверки  
`context.Request.Certificate.Thumbprint`

---

**Сценарий 2**:  
"Разрешить доступ только сертификатам доверенных партнёров"  
→ **Ответ**: Загрузить сертификаты партнёров в APIM и выполнить проверку через  
`context.Deployment.Certificates`

---

**Сценарий 3**:  
"Проверить, что сертификат выпущен корпоративным CA"  
→ **Ответ**: Проверить `context.Request.Certificate.Issuer` и использовать метод `Verify()`

---

**Сценарий 4**:  
"Аутентификация по сертификату не работает в Consumption tier"  
→ **Ответ**: Включить **Negotiate client certificate** в настройках APIM

---

**Сценарий 5**:  
"Проверить, не истёк ли срок действия сертификата"  
→ **Ответ**: Проверить `context.Request.Certificate.NotBefore` и `NotAfter` относительно `DateTime.UtcNow`

---

### Экзаменационный фокус

- Проверка сертификата выполняется в разделе `inbound`.
- Для невалидного сертификата возвращается `403 Forbidden`.
- Thumbprint используется для точного whitelist-контроля.
- `Verify()` проверяет цепочку доверия.
- В Consumption tier требуется дополнительная настройка.
## Learn More

- [Mutual TLS Authentication](https://docs.microsoft.com/azure/api-management/api-management-howto-mutual-certificates)
- [Certificate Policies](https://docs.microsoft.com/azure/api-management/api-management-access-restriction-policies#ValidateClientCertificate)
- [Client Certificates Overview](https://docs.microsoft.com/azure/api-management/api-management-howto-mutual-certificates-for-clients)
- [X.509 Certificates](https://docs.microsoft.com/azure/security/fundamentals/encryption-overview#x509-certificates)
