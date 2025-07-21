# Authentication Integration

## Overview
The JsonMapperService provides built-in support for various authentication methods including API Keys, Basic Authentication, Bearer Tokens, and OAuth. Authentication credentials are securely stored in the database and automatically decrypted during mapping.

## Authentication Types

#### 1. API Key Authentication
Use stored API keys for request authentication.

**Syntax:**
```json
{
  "headers": {
    "x-api-key": "API_KEY_SYSTEM_NAME"
  }
}
```

**Example:**
```json
{
  "headers": {
    "x-api-key": "WEATHER_SERVICE_API_KEY",
    "Content-Type": "application/json"
  },
  "body": {
    "location": { "source": "city" }
  }
}
```

**Process:**
1. Service looks up `WEATHER_SERVICE_API_KEY` in database
2. Validates credential is active and type is `ApiKey`
3. Decrypts the stored API key
4. Replaces system name with actual key value

**Result:**
```json
{
  "headers": {
    "x-api-key": "actual-decrypted-api-key-value",
    "Content-Type": "application/json"
  },
  "body": {
    "location": "New York"
  }
}
```

#### 2. Basic Authentication

Use username/password credentials encoded as Base64.

**Syntax:**

```json
{
  "headers": {
    "Authorization": "Basic BASIC_AUTH_SYSTEM_NAME"
  }
}
```

**Example:**

```json
{
  "headers": {
    "Authorization": "Basic DATABASE_SERVICE_AUTH",
    "Content-Type": "application/json"
  }
}
```

**Process:**

1. Service looks up `DATABASE_SERVICE_AUTH` credentials
2. Validates credential is active and type is `BasicAuth`
3. Decrypts stored username and password
4. Encodes as Base64: `username:password`
5. Creates proper Authorization header

**Result:**

```json
{
  "headers": {
    "Authorization": "Basic dXNlcm5hbWU6cGFzc3dvcmQ=",
    "Content-Type": "application/json"
  }
}
```

#### 3. Bearer Token Authentication

**Static Bearer Token**
Use a pre-configured bearer token:

```json
{
  "headers": {
    "Authorization": "Bearer OAUTH_SYSTEM_NAME"
  }
}
```

**Current User Token**
Use the current authenticated user's token:

```json
{
  "headers": {
    "Authorization": "Bearer CURRENT_USER_TOKEN"
  }
}
```

**OAuth Client Credentials**
Use OAuth client credentials flow:

```json
{
  "headers": {
    "Authorization": "Bearer API_CLIENT_SYSTEM_NAME"
  }
}
```

**Process for OAuth:**

1. Service looks up API client configuration
2. Executes OAuth flow (client credentials grant)
3. Retrieves access token
4. Creates Bearer authorization header

#### 4. API Client Credentials Access

**Direct Credential Access**
Access stored OAuth credentials directly in your mapping:

**Syntax:**

```json
{
  "authData": {
    "source": "ApiClientCredential OAUTH_SYSTEM_NAME"
  }
}
```

**Example:**

```json
{
  "oauthRequest": {
    "grantType": { "source": "ApiClientCredential PAYMENT_SERVICE.grant_type" },
    "clientId": { "source": "ApiClientCredential PAYMENT_SERVICE.client_id" },
    "clientSecret": { "source": "ApiClientCredential PAYMENT_SERVICE.client_secret" }
  }
}
```

**Supported Grant Types:**

- `password`: Username/password flow (password auto-decrypted)
- `client_credentials`: Client credentials flow (client_secret auto-decrypted)

### Complete Examples

#### Example 1: API Key Integration

**Source JSON:**

```json
{
  "weatherQuery": {
    "city": "London",
    "units": "metric"
  }
}
```

**Mapping Rules:**

```json
{
  "method": "GET",
  "url": "https://api.weather.com/v1/current",
  "headers": {
    "x-api-key": "WEATHER_API_KEY",
    "Accept": "application/json"
  },
  "params": {
    "q": { "source": "weatherQuery.city" },
    "units": { "source": "weatherQuery.units" }
  }
}
```

**Result:**

```json
{
  "method": "GET", 
  "url": "https://api.weather.com/v1/current",
  "headers": {
    "x-api-key": "sk-1234567890abcdef",
    "Accept": "application/json"
  },
  "params": {
    "q": "London",
    "units": "metric"
  }
}
```

#### Example 2: Basic Auth with Current User Context

**Source JSON:**

```json
{
  "report": {
    "type": "monthly",
    "department": "sales"
  }
}
```

**Mapping Rules:**

```json
{
  "method": "POST",
  "url": "https://internal-api.company.com/reports",
  "headers": {
    "Authorization": "Basic INTERNAL_API_AUTH",
    "Content-Type": "application/json"
  },
  "body": {
    "reportType": { "source": "report.type" },
    "department": { "source": "report.department" },
    "requestedBy": { "source": "currentUserDetail.UserName" },
    "requestedByEmail": { "source": "currentUserDetail.Email" }
  }
}
```

**Result:**

```json
{
  "method": "POST",
  "url": "https://internal-api.company.com/reports", 
  "headers": {
    "Authorization": "Basic YWRtaW46c2VjcmV0MTIz",
    "Content-Type": "application/json"
  },
  "body": {
    "reportType": "monthly",
    "department": "sales", 
    "requestedBy": "john.doe",
    "requestedByEmail": "john.doe@company.com"
  }
}
```

#### Example 3: OAuth Client Credentials Flow

**Source JSON:**

```json
{
  "payment": {
    "amount": 100.50,
    "currency": "USD",
    "customerId": "CUST-12345"
  }
}
```

**Mapping Rules:**

```json
{
  "method": "POST",
  "url": "https://api.paymentgateway.com/v2/payments",
  "headers": {
    "Authorization": "Bearer PAYMENT_GATEWAY_CLIENT",
    "Content-Type": "application/json"
  },
  "body": {
    "amount": { "source": "payment.amount" },
    "currency": { "source": "payment.currency" },
    "customer_id": { "source": "payment.customerId" },
    "merchant_id": { "source": "currentUserDetail.Id" }
  }
}
```

**Result:**

```json
{
  "method": "POST",
  "url": "https://api.paymentgateway.com/v2/payments",
  "headers": {
    "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "Content-Type": "application/json"
  },
  "body": {
    "amount": 100.50,
    "currency": "USD",
    "customer_id": "CUST-12345",
    "merchant_id": "usr_789"
  }
}
```

#### Example 4: Direct OAuth Credential Access

**Source JSON:**

```json
{
  "tokenRequest": {
    "scope": "read_users write_users"
  }
}
```

**Mapping Rules:**

```json
{
  "method": "POST",
  "url": "https://oauth.service.com/token",
  "headers": {
    "Content-Type": "application/x-www-form-urlencoded"
  },
  "body": {
    "grant_type": { "source": "ApiClientCredential CRM_SERVICE.grant_type" },
    "client_id": { "source": "ApiClientCredential CRM_SERVICE.client_id" },
    "client_secret": { "source": "ApiClientCredential CRM_SERVICE.client_secret" },
    "scope": { "source": "tokenRequest.scope" }
  }
}
```

**Result:**

```json
{
  "method": "POST",
  "url": "https://oauth.service.com/token",
  "headers": {
    "Content-Type": "application/x-www-form-urlencoded"
  },
  "body": {
    "grant_type": "client_credentials",
    "client_id": "crm_client_12345",
    "client_secret": "decrypted_secret_value",
    "scope": "read_users write_users"
  }
}
```

#### Example 5: Multiple Authentication Methods

**Source JSON:**

```json
{
  "syncRequest": {
    "sourceSystem": "CRM",
    "targetSystem": "ERP",
    "dataType": "customers"
  }
}
```

**Mapping Rules:**

```json
{
  "sourceApi": {
    "method": "GET",
    "url": "https://crm.company.com/api/customers",
    "headers": {
      "Authorization": "Bearer CRM_OAUTH_CLIENT",
      "Accept": "application/json"
    }
  },
  "targetApi": {
    "method": "POST", 
    "url": "https://erp.company.com/api/import/customers",
    "headers": {
      "x-api-key": "ERP_API_KEY",
      "Content-Type": "application/json"
    }
  },
  "auditInfo": {
    "requestedBy": { "source": "currentUserDetail.UserName" },
    "timestamp": { "source": "currentUserDetail.Id" }
  }
}
```

**Result:**

```json
{
  "sourceApi": {
    "method": "GET",
    "url": "https://crm.company.com/api/customers",
    "headers": {
      "Authorization": "Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
      "Accept": "application/json"
    }
  },
  "targetApi": {
    "method": "POST",
    "url": "https://erp.company.com/api/import/customers", 
    "headers": {
      "x-api-key": "ak_1234567890abcdef",
      "Content-Type": "application/json"
    }
  },
  "auditInfo": {
    "requestedBy": "integration.user",
    "timestamp": "usr_456"
  }
}
```

### Authentication Security Features

#### Automatic Encryption/Decryption

- All sensitive credential data is automatically encrypted in storage
- Decryption happens transparently during mapping
- No plaintext credentials are ever exposed in logs or responses

#### Credential Validation

- Active status checking for all credentials
- Authentication type validation
- Automatic fallback to empty values for invalid/inactive credentials

#### Current User Integration

- Seamless integration with current authenticated user context
- Support for accessing user properties in mappings
- Automatic token extraction from HTTP context

### Error Handling

#### Invalid Credentials

```json
{
  "headers": {
    "x-api-key": "",  // Empty if credential not found or inactive
    "Content-Type": "application/json"
  }
}
```

#### Missing Authentication Type

- Service validates authentication type matches expected format
- Falls back to empty values rather than throwing exceptions
- Logs appropriate warnings for debugging

#### OAuth Token Retrieval Failures

- Graceful handling of OAuth service unavailability
- Empty Authorization header if token cannot be obtained
- Maintains request structure for upstream error handling
