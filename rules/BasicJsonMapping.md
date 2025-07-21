# Basic JSON Mapping

## Overview

The basic JSON mapping feature allows you to transform JSON objects by copying values from source paths to destination fields using simple rule definitions.

## Core Concept

Map values from a source JSON object to a new destination JSON object using dot notation paths and transformation rules.

## Simple Field Mapping

### Direct Value Assignment

Set constant values directly in the destination:

```json
{
  "staticField": "constant value",
  "version": "1.0",
  "enabled": true,
  "maxRetries": 3
}
```

## Source Path Mapping

Copy values from source object using dot notation:

```json
{
  "destinationField": {
    "source": "sourceField"
  },
  "userName": {
    "source": "user.profile.name"
  },
  "email": {
    "source": "contact.email"
  }
}
```

## Path Syntax

### Dot Notation

Access nested properties using dots:

- `"user.name"` → source.user.name
- `"profile.settings.theme"` → source.profile.settings.theme

### Array Indexing

Access array elements using square brackets:

- `"items[0]"` → First item in items array
- `"users[2].name"` → Name of third user
- `"data.results[0].id"` → ID of first result

### Combined Paths

Mix dot notation and array indexing:

- `"response.data.users[0].profile.email"`
- `"orders[1].items[0].product.name"`

## Examples

### Example 1: Simple User Profile Mapping

#### Source JSON:

```json
{
  "userData": {
    "firstName": "John",
    "lastName": "Doe", 
    "contactInfo": {
      "email": "john.doe@example.com",
      "phone": "+1-555-0123"
    }
  },
  "accountStatus": "active"
}
```

#### Mapping Rules:

```json
{
  "fullName": { "source": "userData.firstName" },
  "email": { "source": "userData.contactInfo.email" },
  "phone": { "source": "userData.contactInfo.phone" },
  "status": { "source": "accountStatus" },
  "createdAt": "2024-01-15"
}
```

#### Result

```json
{
  "fullName": "John",
  "email": "john.doe@example.com", 
  "phone": "+1-555-0123",
  "status": "active",
  "createdAt": "2024-01-15"
}
```

### Example 2: Array Element Access

#### Source JSON:

```json
{
  "products": [
    {
      "id": 1,
      "name": "Laptop",
      "price": 999.99
    },
    {
      "id": 2, 
      "name": "Mouse",
      "price": 29.99
    }
  ]
}
```

### Mapping Rules:

```json
{
  "firstProductId": { "source": "products[0].id" },
  "firstProductName": { "source": "products[0].name" },
  "secondProductPrice": { "source": "products[1].price" }
}
```

#### Result:

```json
{
  "firstProductId": 1,
  "firstProductName": "Laptop",
  "secondProductPrice": 29.99
}
```

### Example 3: Nested Object Creation

#### Source JSON:

```json
{
  "firstName": "Jane",
  "lastName": "Smith",
  "email": "jane@example.com",
  "age": 30,
  "city": "New York"
}
```

#### Mapping Rules:

```json
{
  "user": {
    "personal": {
      "name": { "source": "firstName" },
      "surname": { "source": "lastName" },
      "age": { "source": "age" }
    },
    "contact": {
      "email": { "source": "email" },
      "location": { "source": "city" }
    }
  }
}
```

#### Result:

```json
{
  "user": {
    "personal": {
      "name": "Jane",
      "surname": "Smith", 
      "age": 30
    },
    "contact": {
      "email": "jane@example.com",
      "location": "New York"
    }
  }
}
```

## Error Handling

### Missing Source Paths

When a source path doesn't exist, the field is set to null:

```json
// If "user.middleName" doesn't exist in source
{
  "middleName": { "source": "user.middleName" }
}
// Results in: { "middleName": null }
```

### Invalid Array Indices

Accessing non-existent array indices returns null:
```json
// If array has only 2 items
{
  "thirdItem": { "source": "items[2].name" }  
}
// Results in: { "thirdItem": null }
```

## Best Practices

### Use Clear Field Names

Make destination field names descriptive:

```json
{
  "customerEmail": { "source": "user.email" }  // Good
  "e": { "source": "user.email" }             // Avoid
}
```

### Handle Optional Fields

Plan for missing source data:

```json
{
  "optionalField": { "source": "data.optional" },
  "fallbackValue": "default"
}
```

### Validate Source Structure

Ensure source paths exist before mapping:

```json
{
  "safeField": { "source": "confirmed.existing.path" }
}
```

### Group Related Fields

Organize related mappings in nested objects:

```json
{
  "userInfo": {
    "name": { "source": "firstName" },
    "email": { "source": "email" }
  }
}
```

## Advanced Features

### 1. URL Configuration

#### Path Parameters
Replace placeholders in URLs with actual values:

```json
{
  "url": "https://api.service.com/v2/customers/{customerId}/orders/{orderId}",
  "pathParams": {
    "customerId": {
      "source": "customer.id"
    },
    "orderId": {
      "source": "order.number"
    }
  }
}
```

#### Query Parameters
Add query string parameters:

```json
{
  "url": "https://api.service.com/v2/products",
  "queryParams": {
    "category": {
      "source": "filters.category"
    },
    "page": 1,
    "limit": 20
  }
}
```

#### Environment Placeholders
Use environment-specific URLs:

```json
{
  "url": "{ACMS_API_BASE_URL}/api/v1/users",
  "httpMethod": "GET"
}
```

Supported placeholders:
- `{PREFIX_API_BASE_URL}` → Maps to `App:SelfUrl` configuration
- `{PREFIX_AUTH_BASE_URL}` → Maps to `AuthServer:Authority` configuration
- `{PREFIX_PM_BASE_URL}` → Maps to `ProcessMakerAPI:BaseURL` configuration

### 2. Content Type Handling

#### Form URL Encoded
For form data submissions:

```json
{
  "contentType": "application/x-www-form-urlencoded",
  "body": {
    "grant_type": "client_credentials",
    "client_id": {
      "source": "oauth.clientId"
    },
    "client_secret": {
      "source": "oauth.clientSecret"
    }
  }
}
```

#### Multipart Form Data
For file uploads and mixed content:

```json
{
  "contentType": "multipart/form-data",
  "body": {
    "file": {
      "source": "document.content"
    },
    "metadata": {
      "source": "document.info"
    }
  }
}
```

### 3. Advanced Error Handling

#### Service-Level Error Responses
The service provides structured error responses:

```json
{
  "success": false,
  "code": 500,  // HTTP status code
  "message": "Error description"
}
```

Error codes:
- `400`: Bad Request (validation errors)
- `401`: Unauthorized (invalid API key for proxy)
- `403`: Forbidden (missing permissions)
- `404`: API Client not found
- `500`: Internal Server Error

#### Error Response Mapping
When HTTP calls fail, the service uses `ErrorResponseConfig`:

```json
{
  "ApiClient": {
    "SystemName": "PAYMENT_API",
    "ResponseConfig": {
      // Success response mapping
    },
    "ErrorResponseConfig": {
      "errorCode": {
        "source": "error.code"
      },
      "errorMessage": {
        "source": "error.description"
      },
      "errorDetails": {
        "source": "error.details"
      }
    }
  }
}
```

#### Empty Response Body Handling
```json
{
  "success": true,  // or false based on HTTP status
  "code": 200       // HTTP status code
}
```

#### Non-Mapped Response
If no ResponseConfig is provided, returns the parsed response as-is.

## Extended Best Practices

### 1. URL Configuration
- Use environment placeholders instead of hardcoded URLs
- Properly encode path and query parameters
- Handle optional query parameters gracefully

```json
// Good
{
  "url": "{SERVICE_API_BASE_URL}/users",
  "queryParams": {
    "active": {
      "source": "filters.activeOnly",
      "optional": true
    }
  }
}

// Avoid
{
  "url": "https://hardcoded.domain.com/api/users"
}
```

### 2. Content Type Selection
- Choose appropriate content type for the data being sent
- Use `application/json` for JSON payloads
- Use `multipart/form-data` for file uploads
- Use `application/x-www-form-urlencoded` for simple form submissions

### 3. Error Handling Strategy
- Define both success and error response mappings
- Handle empty responses gracefully
- Include proper error details in the response
- Log relevant error information for debugging

### 4. Security Considerations
- Never include sensitive data in URLs
- Use proper content types for secure data transmission
- Validate and sanitize path parameters
- Use HTTPS for all external URLs

## Common Use Cases

### 1. RESTful API Integration
```json
{
  "url": "{API_BASE_URL}/v1/resources/{resourceId}",
  "pathParams": {
    "resourceId": {
      "source": "id"
    }
  },
  "queryParams": {
    "include": "details,metadata",
    "version": {
      "source": "requestedVersion"
    }
  }
}
```

### 2. Form Submission
```json
{
  "contentType": "application/x-www-form-urlencoded",
  "body": {
    "username": {
      "source": "user.name"
    },
    "password": {
      "source": "user.password"
    },
    "remember": true
  }
}
```

### 3. File Upload
```json
{
  "contentType": "multipart/form-data",
  "body": {
    "document": {
      "source": "uploadedFile"
    },
    "description": {
      "source": "fileMetadata.description"
    },
    "tags": {
      "source": "fileMetadata.tags"
    }
  }
}
```

### 4. Error Response Handling
```json
{
  "ResponseConfig": {
    "data": {
      "source": "result"
    }
  },
  "ErrorResponseConfig": {
    "error": {
      "code": {
        "source": "errorCode"
      },
      "message": {
        "source": "errorMessage"
      },
      "details": {
        "source": "errorDetails"
      }
    }
  }
}
```

