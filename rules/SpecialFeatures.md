# Special Features

## Overview

Special features extend the flexibility and power of the mapping system, enabling advanced scenarios such as user context injection, rules engine integration, dynamic URL construction, environment placeholders, multi-step authentication flows, and more.

---

## 1. Current User Context & Token

Inject details about the currently authenticated user and their token into your mappings using special placeholders.

**User Details Example:**
```json
{
  "body": {
    "userId": { "source": "currentUserDetail.id" },
    "userName": { "source": "currentUserDetail.username" },
    "userEmail": { "source": "currentUserDetail.email" },
    "fullName": { "source": "currentUserDetail.fullname" }
  }
}
```
*Input:* `{ "currentUserDetail": { "id": "u1", "username": "jdoe", "email": "jdoe@example.com", "fullname": "John Doe" } }`
*Output:* `{ "body": { "userId": "u1", "userName": "jdoe", "userEmail": "jdoe@example.com", "fullName": "John Doe" } }`

**Current User Token Example:**
```json
{
  "headers": {
    "Authorization": "Bearer CURRENT_USER_TOKEN"
  }
}
```
*Result:* The user's current authentication token is injected at runtime.

**Error Handling:** If a user property or token is missing, the field is set to `null` or the header is omitted.

---

## 2. Rules Engine Integration (Advanced)

Compute field values based on business logic using the rules engine. You can use it in both request and response mappings, and for conditional logic.

**Request Example:**
```json
{
  "body": {
    "isEligible": {
      "source": "applicationData",
      "rulesEngine": "LOAN_ELIGIBILITY_RULES"
    }
  }
}
```
**Response Example:**
```json
{
  "ResponseConfig": {
    "eligibilityStatus": {
      "source": "applicationData",
      "rulesEngine": "LOAN_ELIGIBILITY"
    },
    "riskCategory": {
      "source": "customerProfile",
      "rulesEngine": "RISK_ASSESSMENT"
    }
  }
}
```
*Input:* `{ "applicationData": { ... } }`
*Output:* `{ "body": { "isEligible": "Eligible" } }` *(assuming rules engine returns this)*

**Error Handling:** If the rules engine fails, the field is set to `null` and an error is logged.

---

## 3. URL Configuration

Build URLs dynamically using path parameters, query parameters, and environment placeholders.

**Path Parameters Example:**
```json
{
  "url": "https://api.service.com/v2/customers/{customerId}/orders/{orderId}",
  "pathParams": {
    "customerId": { "source": "customer.id" },
    "orderId": { "source": "order.number" }
  }
}
```
*Input:* `{ "customer": { "id": "C123" }, "order": { "number": "O456" } }`
*Output URL:* `https://api.service.com/v2/customers/C123/orders/O456`

**Query Parameters Example:**
```json
{
  "url": "https://api.service.com/v2/products",
  "queryParams": {
    "category": { "source": "filters.category" },
    "page": 1,
    "limit": 20
  }
}
```
*Input:* `{ "filters": { "category": "electronics" } }`
*Output URL:* `https://api.service.com/v2/products?category=electronics&page=1&limit=20`

**Environment Placeholders Example:**
```json
{
  "url": "{ACMS_API_BASE_URL}/api/v1/users"
}
```
*Resolved at runtime based on environment configuration.*

**Error Handling:** If a path parameter is missing, the request will fail. Missing query parameters are omitted. If the environment placeholder is not configured, the request will fail with a configuration error.

---

## 4. Content-Type Handling

Specify the content type for requests, including support for form data and file uploads.

**Form URL Encoded Example:**
```json
{
  "contentType": "application/x-www-form-urlencoded",
  "body": {
    "grant_type": "client_credentials",
    "client_id": { "source": "oauth.clientId" },
    "client_secret": { "source": "oauth.clientSecret" }
  }
}
```
*Input:* `{ "oauth": { "clientId": "abc", "clientSecret": "xyz" } }`
*Output:* Properly formatted form data body.

**Multipart Form Data Example:**
```json
{
  "contentType": "multipart/form-data",
  "body": {
    "file": { "source": "document.content" },
    "metadata": { "source": "document.info" }
  }
}
```
*Input:* `{ "document": { "content": "...", "info": { "name": "file.txt" } } }`
*Output:* Properly formatted multipart request.

**Error Handling:** If the content type is not supported, the request will fail.

---

## 5. Complex Array Source (Concatenation)

Combine multiple arrays or values into a single destination property using the `concat` transformation.

**Example:**
```json
{
  "combinedData": {
    "source": ["users.active", "users.pending", "users.inactive"],
    "transform": "concat"
  }
}
```
*Input:* `{ "users": { "active": ["a"], "pending": ["b"], "inactive": ["c"] } }`
*Output:* `{ "combinedData": ["a", "b", "c"] }`

**Error Handling:** If any source array is missing, it is treated as empty.

---

## 6. Multi-Step Authentication / API Orchestration

Support for workflows that require multiple API calls in sequence.

**Example:**
```json
{
  "step1_getToken": {
    "url": "https://auth.api.com/token",
    "body": {
      "source": "ApiClientCredential OAUTH_CREDS"
    }
  },
  "step2_apiCall": {
    "url": "https://api.service.com/data",
    "headers": {
      "Authorization": "Bearer {step1_getToken.access_token}"
    }
  }
}
```
*Input:* Credentials and data for both steps.
*Output:* Step 2 uses the token from Step 1.

**Error Handling:** If any step fails, subsequent steps are not executed.

---

## 7. Request Signing and Encryption

Support for secure request models with Base64 encoding and digital signatures.

**Example:**
```json
{
  "body": {
    "requestModel": {
      "encodeBase64": true,
      "generateSignature": true,
      "signaturePrivateKeySystemName": "MBL_PRIVATE_KEY",
      "TransactionId": { "source": "transactionId" },
      "Amount": { "source": "amount" }
    }
  }
}
```
*Input:* `{ "transactionId": "T1", "amount": 100 }`
*Output:* `{ "Data": "...base64...", "Signature": "...", "TimeStamp": "..." }`

**Error Handling:** If the private key is missing or invalid, signature generation fails and the request is not sent.

---

## 8. Advanced Error Handling

Define separate mappings for error responses using `ErrorResponseConfig`.

**Example:**
```json
{
  "ApiClient": {
    "ResponseConfig": {
      "data": { "source": "result" }
    },
    "ErrorResponseConfig": {
      "errorCode": { "source": "error.code" },
      "errorMessage": { "source": "error.description" }
    }
  }
}
```
*Input:* Error response from API.
*Output:* `{ "errorCode": "401", "errorMessage": "Unauthorized" }`

**Error Handling:** If error fields are missing, they are set to `null`.

---

## 9. Validation and Conditional Logic

Validate responses and apply conditional logic using validation rules and the rules engine.

**Example:**
```json
{
  "ResponseConfig": {
    "validation": {
      "required": ["transactionId", "status"],
      "custom": "response.amount > 0"
    },
    "data": {
      "transactionId": { "source": "txn.id" },
      "status": { "source": "state" },
      "amount": { "source": "total" }
    }
  }
}
```
*Input:* `{ "txn": { "id": "T1" }, "state": "success", "total": 100 }`
*Output:* `{ "transactionId": "T1", "status": "success", "amount": 100 }`

**Error Handling:** If required fields are missing or validation fails, an error is returned or the response is flagged as invalid.

---

## Advanced Features
- **Multi-step API orchestration**
- **Dynamic environment-based configuration**
- **Conditional mapping with rules engine**
- **Batch processing with array inputs**
- **Pagination handling in responses**
- **Proxy authentication and permission-based access**

---

## Best Practices
- Use environment placeholders for all base URLs.
- Leverage current user context for audit and personalization fields.
- Use the rules engine for complex business logic and eligibility checks.
- Use multi-step flows for APIs requiring token exchange or orchestration.
- Always define error response mappings for robust error handling.
- Validate responses and requests to ensure data integrity.
- Test advanced features with both success and error scenarios.
- Document complex flows and custom logic for maintainability.

---

## Common Use Cases
- Injecting current user info and token into requests
- Orchestrating multi-step API workflows
- Building dynamic URLs for RESTful APIs
- Handling file uploads and form submissions
- Securing requests with signatures and encryption
- Standardizing error responses
- Validating and transforming complex API responses
- Batch processing and pagination
