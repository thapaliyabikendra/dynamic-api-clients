# Error Handling

## Overview

Robust error handling is essential for reliable API integrations. The Dynamic API Client system provides structured error responses, error mapping, and strategies for handling various error scenarios, including authentication failures, validation errors, and transformation issues.

---

## 1. Error Response Structure

All API endpoints return a consistent error response format:

```json
{
  "success": false,
  "code": 400,  // or 401, 403, 404, 500
  "message": "Error description",
  "error": {
    // Mapped error data from ErrorResponseConfig (if configured)
  }
}
```

- `success`: Always `false` for errors
- `code`: HTTP status code (400, 401, 403, 404, 500, etc.)
- `message`: Human-readable error message
- `error`: Optional, detailed error object (from ErrorResponseConfig)

---

## 2. Error Types

- **400 Bad Request**: Validation errors, missing/invalid input, configuration issues
- **401 Unauthorized**: Missing or invalid authentication (e.g., `x-api-key` or `Authorization` header)
- **403 Forbidden**: User lacks required permissions
- **404 Not Found**: API client or resource not found
- **500 Internal Server Error**: Unexpected server-side errors

---

## 3. Error Mapping (ErrorResponseConfig)

You can define custom error mappings for API-specific error formats using `ErrorResponseConfig`.

**Syntax:**
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

**Example:**
```json
{
  "ApiClient": {
    "ResponseConfig": {
      "data": { "source": "result" }
    },
    "ErrorResponseConfig": {
      "errorCode": { "source": "error.code" },
      "errorMessage": { "source": "error.description" },
      "errorDetails": { "source": "error.details" }
    }
  }
}
```

**Mapped Error Response:**
```json
{
  "success": false,
  "code": 400,
  "message": "Validation failed",
  "error": {
    "errorCode": "E1001",
    "errorMessage": "Missing required field: orderId",
    "errorDetails": "The 'orderId' field is required."
  }
}
```

---

## 4. Error Handling Strategies

- **Missing Credentials**: Returns empty string or null for missing/inactive credentials; logs a warning
- **Invalid Paths**: Missing source paths result in null/empty fields; arrays return empty arrays
- **Transformation Errors**: Invalid casts or transformation failures return the original value or null
- **Signature Generation Errors**: Missing/invalid private key causes request failure; logs error
- **OAuth Token Failures**: Returns empty token or error; logs details
- **Proxy Authentication**: Missing/invalid `x-api-key` returns 401 Unauthorized
- **Permission Denied**: User without required permission receives 403 Forbidden
- **Malformed JSON**: Returns 400 Bad Request with parsing error details

---

## 5. Error Handling Examples

### Example 1: Validation Error
```json
{
  "success": false,
  "code": 400,
  "message": "Validation failed: 'amount' must be a number"
}
```

### Example 2: Unauthorized (Missing API Key)
```json
{
  "success": false,
  "code": 401,
  "message": "Unauthorized: Missing or invalid x-api-key"
}
```

### Example 3: Forbidden (Permission Denied)
```json
{
  "success": false,
  "code": 403,
  "message": "User is not authorized to access this API client."
}
```

### Example 4: Not Found (API Client)
```json
{
  "success": false,
  "code": 404,
  "message": "API Client 'PAYMENT_API' not found."
}
```

### Example 5: Internal Server Error
```json
{
  "success": false,
  "code": 500,
  "message": "An unexpected error occurred. Please try again later."
}
```

---

## 6. Logging and Debugging

### Log Entry Structure

The service provides comprehensive logging for error tracking:

```json
{
  "timestamp": "2024-01-15T10:30:45.123Z",
  "level": "ERROR",
  "component": "JsonMapperService",
  "operation": "ProcessMapping",
  "apiClient": "PAYMENT_GATEWAY",
  "userId": "usr_123",
  "requestId": "req_456",
  "error": {
    "type": "MappingError",
    "message": "Failed to process array mapping",
    "details": {
      "sourcePath": "items",
      "sourceValue": "not_an_array",
      "expectedType": "array"
    }
  },
  "context": {
    "inputData": "truncated_input_json",
    "mappingConfig": "truncated_mapping_config"
  }
}
```

### Log Categories

#### Authentication Errors
```json
{
  "level": "WARNING",
  "component": "AuthenticationHandler",
  "message": "Credential not found or inactive",
  "details": {
    "credentialSystemName": "MISSING_API_KEY",
    "credentialType": "ApiKey",
    "isFound": false,
    "isActive": null
  }
}
```

#### Transformation Errors
```json
{
  "level": "WARNING",
  "component": "TransformationEngine",
  "message": "Type cast failed, returning original value",
  "details": {
    "sourcePath": "status",
    "sourceValue": "invalid_boolean",
    "targetType": "bool",
    "operation": "cast"
  }
}
```

#### HTTP Request Errors
```json
{
  "level": "ERROR",
  "component": "HttpClient",
  "message": "HTTP request failed",
  "details": {
    "url": "https://api.service.com/endpoint",
    "method": "POST",
    "statusCode": 500,
    "responseBody": "truncated_response",
    "requestBody": "truncated_request"
  }
}
```

### Debug Information

Debug logs include truncated request/response bodies (max 2000 characters):

```json
{
  "level": "DEBUG",
  "component": "JsonMapperService",
  "message": "Request mapping completed",
  "details": {
    "inputDataPreview": "{\"user\":{\"name\":\"John\",\"email\":\"john@...",
    "outputDataPreview": "{\"customer\":{\"fullName\":\"John Doe\",\"...",
    "mappingRulesApplied": 15,
    "processingTimeMs": 45
  }
}
```

## 7. Best Practices

- Always check the `success` and `code` fields in responses
- Handle all error codes and messages in your client logic
- Use `ErrorResponseConfig` for custom error mapping
- Log error details for troubleshooting and auditing
- Validate all input before sending requests
- Use descriptive error messages for easier debugging
- Test error handling with missing fields, invalid credentials, and transformation failures

---

## 8. Troubleshooting Guide

### Common Issues and Solutions

1. **Empty Authorization Header**
   - Check credential system name exists and matches exactly
   - Verify credential is active
   - Ensure correct authentication type

2. **Signature Generation Fails**
   - Verify PEM certificate system name
   - Check certificate is properly formatted
   - Ensure private key is not corrupted

3. **Array Mapping Returns Empty**
   - Confirm source path points to actual array
   - Check array element structure matches item rules
   - Verify transform type is "map"

4. **Type Casting Issues**
   - Ensure source value can be converted to target type
   - Use string intermediate type for complex conversions
   - Handle null values explicitly

5. **OAuth Token Not Retrieved**
   - Verify API client configuration
   - Check OAuth credentials are correct
   - Ensure token endpoint is accessible

6. **Malformed JSON or Parsing Errors**
   - Validate all input and output JSON
   - Use JSON validators and schema checks

---
