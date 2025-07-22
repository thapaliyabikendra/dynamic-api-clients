# Best Practices

## Overview

This document provides a comprehensive set of best practices for configuring, securing, and maintaining API integrations using the Dynamic API Client system. Adhering to these guidelines ensures robust, performant, and maintainable integrations.

---

## 1. API Client Configuration

### 1.1 Naming Conventions
- **Use UPPERCASE for System Names:** Always use uppercase for `ApiClient` system names (e.g., `PAYMENT_GATEWAY_API`, `MBL_STAFF_DETAIL_INFO`). This is a system convention and ensures consistency.
- **Be Descriptive:** Choose names that clearly indicate the service being integrated (e.g., `SAP_INVOICE_CREATE`).
- **Use Prefixes for Grouping:** Group related APIs with common prefixes (e.g., `PM_TASK_CREATE`, `PM_TASK_UPDATE` for ProcessMaker tasks).

### 1.2 Configuration Management
- **Store Config in Database:** Keep `RequestConfig`, `ResponseConfig`, and `ErrorResponseConfig` in the `ApiClient` entity in the database as the source of truth.
- **Use Overrides for Exceptions:** Only use request-time overrides for temporary or highly specific, one-off scenarios.
- **Document API Clients:** Maintain a separate registry or documentation explaining the purpose, owner, and dependencies of each `ApiClient`.

**Usage Example (ApiClient Structure):**
```json
{
  "SystemName": "PAYMENT_GATEWAY_API",
  "RequestConfig": { "..." },
  "ResponseConfig": { "..." },
  "ErrorResponseConfig": { "..." },
  "IsProxyable": false,
  "Permissions": ["DynamicApiClients.Default"]
}
```

---

## 2. Credential Management

### 2.1 Naming and Security
- **Use UPPERCASE for Credentials:** Credential system names should also be in uppercase (e.g., `PAYMENT_API_KEY`).
- **Store Securely:** All sensitive data (passwords, secrets, keys) must be stored encrypted in the credential store. The system handles decryption automatically.
- **Never Log Sensitive Data:** Ensure that decrypted credentials, tokens, or private keys are never logged.

### 2.2 Lifecycle Management
- **Implement Credential Rotation:** Regularly rotate credentials. This can be done by updating the credential value in the store without changing any `ApiClient` configurations.
- **Use the `IsActive` Flag:** To temporarily disable a credential without deleting it, set its `IsActive` flag to `false`.

**Usage Example (Inactive Credential):**
```json
{
  "SystemName": "OLD_SYSTEM_API_KEY",
  "Type": "ApiKey",
  "Value": "encrypted:...",
  "IsActive": false
}
```

---

## 3. Performance Optimization

### 3.1 Mapping and Transformation
- **Avoid Deep Nesting:** Try to avoid nesting mappings beyond 5 levels, as it can impact performance.
- **Use `resultType: "First"`:** When mapping an array where you only need the first item, use `resultType: "First"` to prevent unnecessary processing of the entire array.

**Syntax:**
```json
{
  "firstItem": {
    "source": "largeArray",
    "transform": "map",
    "resultType": "First",
    "item": { "id": { "source": "Id" } }
  }
}
```

### 3.2 Authentication
- **Cache OAuth Tokens:** For services that use OAuth, ensure token caching is enabled to minimize redundant token acquisition requests.

### 3.3 URL Configuration
- **Use Environment Placeholders:** Always use environment placeholders (e.g., `{ACMS_API_BASE_URL}`) for base URLs. This avoids hardcoding and makes configurations portable across different environments (dev, test, prod).

**Syntax:**
```json
{
  "url": "{ACMS_API_BASE_URL}/v1/some-service"
}
```

---

## 4. Security Considerations

### 4.1 Transport and Data
- **Always Use HTTPS:** All external API endpoints should be configured with `https://`.
- **Implement Request Signing:** For sensitive operations, use the built-in request signing feature (`generateSignature: true`) to ensure message integrity and non-repudiation.
- **Validate Certificates:** Ensure server certificate validation is enabled for all external communications.

**Usage Example (Secure Request):**
```json
{
  "url": "https://secure-api.com/v1/transaction",
  "body": {
    "requestModel": {
      "generateSignature": true,
      "signaturePrivateKeySystemName": "TRANSACTION_SIGNING_KEY",
      "data": { "source": "payload" }
    }
  }
}
```

### 4.2 Access Control
- **Use Appropriate Authentication:** Select the right authentication method (API Key, Basic, OAuth, PEM) for each integration.
- **Configure `IsProxyable` Carefully:** Only set `IsProxyable` to `true` for APIs that are explicitly designed to be exposed externally. Secure them with a strong, unique `x-api-key`.
- **Apply Principle of Least Privilege:** Use the permission system to ensure that internal users can only access the API clients relevant to their roles.

**Usage Example (Proxyable Client):**
```json
{
  "SystemName": "EXTERNAL_WEBHOOK_HANDLER",
  "IsProxyable": true,
  "Permissions": [] // No internal permissions needed
}
```

---

## 5. Error Handling Strategy

### 5.1 Comprehensive Error Mapping
- **Always Define `ErrorResponseConfig`:** For every `ApiClient`, define an `ErrorResponseConfig` to standardize the error format you receive from the external API. This makes your client-side error handling much more predictable.

**Usage Example:**
```json
{
  "ApiClient": {
    "ResponseConfig": {
      "success": { 
        "source": "status", 
        "transform": "cast", 
        "castTo": "bool" 
        },
      "data": { 
        "source": "data" 
        }
    },
    "ErrorResponseConfig": {
      "errorCode": { "source": "error.code" },
      "errorMessage": { "source": "error.message" }
    }
  }
}
```

### 5.2 Client-Side Handling
- **Check `success` and `code`:** Your application logic should always check the `success` field (true/false) and the `code` (HTTP status) to determine how to proceed.

---

## 6. URL Configuration Best Practices

### 6.1 Use Placeholders
- **Good:** `{ACMS_API_BASE_URL}/api/v1/resource`
- **Bad:** `https://hardcoded.domain.com/api/v1/resource`

### 6.2 Path vs. Query Parameters
- **Use Path Parameters for Resource Identifiers:** Use path parameters for unique identifiers of a resource.
  
  **Usage Example:**
  ```json
  {
    "url": "/api/users/{userId}/orders/{orderId}"
  }
  ```
- **Use Query Parameters for Filtering and Options:** Use query parameters for sorting, filtering, and pagination.
  
  **Usage Example:**
  ```json
  {
    "queryParams": { 
    "status": "active", 
    "page": 1 }
  }
  ```

---

## 7. Testing Recommendations

- **Source Data Variations:** Test with missing or null source properties to see how mappings behave.
- **Authentication:** Verify credential handling for both active and inactive credentials, and for correct and incorrect keys/tokens.
- **Array Transformations:** Test with empty arrays, arrays with one item, and arrays with many items.
- **Signature Generation:** Validate signature generation and verification with test keys.
- **Response Mapping:** Test both success (`ResponseConfig`) and error (`ErrorResponseConfig`) mappings with various response structures from the target API.
- **Proxy Access:** If `IsProxyable` is true, test access both with and without the correct `x-api-key`.

---

## 8. Monitoring and Maintenance

- **Monitor Failures:** Set up monitoring for failed API calls, especially for repeated authentication failures or 5xx errors from external systems.
- **Set Up Alerts:** Configure alerts for critical failures to enable rapid response.
- **Regularly Review Clients:** Periodically review and clean up unused or obsolete `ApiClient` configurations.
- **Maintain a Registry:** Keep a central document or registry that details the purpose, owner, and business impact of each `ApiClient` integration.
