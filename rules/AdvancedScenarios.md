# Advanced Scenarios for Dynamic API Client

## Conditional Mapping with Rules Engine

### Basic Rules Engine Integration
Use the Rules Engine to apply conditional logic during data transformation:

```json
{
  "body": {
    "eligibilityStatus": {
      "source": "applicationData",
      "rulesEngine": "LOAN_ELIGIBILITY"
    },
    "riskCategory": {
      "source": "customerProfile",
      "rulesEngine": "RISK_ASSESSMENT"
    },
    "approvalLevel": {
      "source": "loanRequest",
      "rulesEngine": "APPROVAL_WORKFLOW"
    }
  }
}
```

### Complex Conditional Logic
Combine multiple data sources with rules engine:

```json
{
  "body": {
    "decision": {
      "source": {
        "customerData": "customer.profile",
        "creditScore": "credit.rating",
        "loanAmount": "request.amount",
        "collateral": "assets.value"
      },
      "rulesEngine": "LOAN_DECISION_ENGINE"
    }
  }
}
```

## Multi-Step API Orchestration

### OAuth Token Exchange Pattern
For APIs requiring OAuth authentication, implement a two-step process:

```csharp
// Step 1: Get OAuth Token
var tokenRequest = new CreateHttpClientDto
{
    ApiClientName = "OAUTH_TOKEN_PROVIDER",
    Request = JObject.Parse("{\"scope\": \"api.read\"}")
};
var tokenResponse = await dynamicHttpClientService.CreateHttpClientAsync(tokenRequest);

// Step 2: Use token in subsequent call
var apiRequest = new CreateHttpClientDto
{
    ApiClientName = "PROTECTED_API",
    Request = JObject.Parse($"{{\"token\": \"{tokenResponse["accessToken"]}\"}}"),
    RequestConfig = @"{
        ""headers"": {
            ""Authorization"": ""Bearer {token}""
        }
    }"
};
var apiResponse = await dynamicHttpClientService.CreateHttpClientAsync(apiRequest);
```

### Multi-Step Configuration
Define multiple steps in a single configuration:

```json
{
  "step1_getToken": {
    "url": "https://auth.api.com/token",
    "httpMethod": "POST",
    "contentType": "application/x-www-form-urlencoded",
    "body": {
      "source": "ApiClientCredential OAUTH_CREDS"
    }
  },
  "step2_apiCall": {
    "url": "https://api.service.com/data",
    "httpMethod": "GET",
    "headers": {
      "Authorization": "Bearer {tokenFromStep1}"
    }
  }
}
```

## Dynamic URL Construction

### Path Parameters
Replace placeholders in URLs with dynamic values:

```json
{
  "url": "https://api.service.com/v2/{region}/customers/{customerId}/orders/{orderId}",
  "pathParams": {
    "region": {
      "source": "user.region",
      "transform": "lowercase"
    },
    "customerId": {
      "source": "customer.id"
    },
    "orderId": {
      "source": "order.number"
    }
  }
}
```

### Query Parameters
Add dynamic query string parameters:

```json
{
  "url": "https://api.service.com/v2/products",
  "queryParams": {
    "category": {
      "source": "filters.category"
    },
    "status": {
      "source": "filters.status",
      "transform": "uppercase"
    },
    "page": {
      "source": "pagination.page"
    },
    "limit": {
      "source": "pagination.size"
    },
    "sortBy": "createdDate",
    "sortOrder": "desc"
  }
}
```

### Environment-Specific URLs
Use environment placeholders for flexible deployment:

```json
{
  "url": "{ACMS_API_BASE_URL}/api/v1/users",
  "httpMethod": "GET"
}
```

Supported placeholders:
- `{PREFIX_API_BASE_URL}` → Maps to `App:SelfUrl`
- `{PREFIX_AUTH_BASE_URL}` → Maps to `AuthServer:Authority`
- `{PREFIX_PM_BASE_URL}` → Maps to `ProcessMakerAPI:BaseURL`

## Handling Paginated Responses

### Standard Pagination
Handle APIs that return paginated data:

```json
{
  "ResponseConfig": {
    "items": {
      "source": "data.results",
      "transform": "map",
      "item": {
        "id": { "source": "id" },
        "name": { "source": "name" },
        "status": { "source": "status" }
      }
    },
    "pagination": {
      "currentPage": {
        "source": "data.page"
      },
      "totalPages": {
        "source": "data.totalPages"
      },
      "totalRecords": {
        "source": "data.totalCount"
      },
      "hasNext": {
        "source": "data.hasMore",
        "transform": "cast",
        "castTo": "bool"
      },
      "hasPrevious": {
        "source": "data.hasPrevious",
        "transform": "cast",
        "castTo": "bool"
      },
      "nextPageUrl": {
        "source": "data.links.next"
      },
      "previousPageUrl": {
        "source": "data.links.previous"
      }
    }
  }
}
```

### Cursor-Based Pagination
Handle APIs using cursor-based pagination:

```json
{
  "ResponseConfig": {
    "items": {
      "source": "data.items",
      "transform": "map",
      "item": {
        "id": { "source": "id" },
        "name": { "source": "name" }
      }
    },
    "pagination": {
      "nextCursor": {
        "source": "data.nextCursor"
      },
      "hasMore": {
        "source": "data.hasMore",
        "transform": "cast",
        "castTo": "bool"
      }
    }
  }
}
```

## Batch Processing with Array Inputs

### Simple Batch Processing
Process multiple items in a single request:

```json
{
  "RequestConfig": {
    "url": "https://api.service.com/batch/process",
    "httpMethod": "POST",
    "body": {
      "requests": {
        "source": "items",
        "transform": "map",
        "item": {
          "requestId": {
            "source": "id"
          },
          "operation": "UPDATE",
          "data": {
            "status": {
              "source": "newStatus"
            },
            "updatedBy": {
              "source": "currentUserDetail.id"
            },
            "updatedAt": {
              "source": "currentUserDetail.timestamp"
            }
          }
        }
      }
    }
  }
}
```

## Complex Data Transformations

### Nested Object and Array Mapping
Transform deeply nested objects and arrays with custom rules:

```json
{
  "body": {
    "customer": {
      "personalInfo": {
        "firstName": { "source": "user.fname" },
        "lastName": { "source": "user.lname" }
      },
      "contact": {
        "email": { "source": "user.email" },
        "phone": { "source": "user.mobile" }
      }
    },
    "orders": {
      "source": "orderList",
      "transform": "map",
      "item": {
        "orderId": { "source": "id" },
        "amount": { "source": "total", "transform": "cast", "castTo": "string" },
        "items": {
          "source": "items",
          "transform": "map",
          "item": {
            "sku": { "source": "sku" },
            "qty": { "source": "quantity" }
          }
        }
      }
    }
  }
}
```

### Data Encoding and Signature Generation
Encode request models and generate digital signatures:

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
This produces:
```json
{
  "Data": "base64_encoded_request_model",
  "Signature": "rsa_signature",
  "TimeStamp": "2024-01-15T10:30:45.123"
}
```

### Array Concatenation and Merging
Combine multiple arrays or values:

```json
{
  "combinedData": {
    "source": ["users.active", "users.pending", "users.inactive"],
    "transform": "concat",
    "separator": ","
  }
}
```

## Advanced Authentication Flows

### API Key, Basic Auth, and OAuth
Inject credentials dynamically:

```json
{
  "headers": {
    "x-api-key": "PAYMENT_GATEWAY_API_KEY",
    "Authorization": "Basic MBL_BASIC_AUTH_CREDENTIAL",
    "Custom-Token": "Bearer EXTERNAL_API_CLIENT"
  }
}
```

### OAuth Client Credentials in Body
```json
{
  "body": {
    "source": "ApiClientCredential OAUTH_CREDENTIAL_NAME"
  }
}
```

## Error Handling Strategies

- Use `ErrorResponseConfig` to map error responses:
  ```json
  {
    "errorCode": { "source": "error.code" },
    "errorMessage": { "source": "error.description" },
    "errorDetails": { "source": "error.details" }
  }
  ```
- If a source path is missing, the property is omitted or set to a default.
- Invalid transformations (e.g., failed casts) return the original value.
- Array mappings on non-arrays return empty arrays.

## Performance Optimization

- Use `resultType: "First"` or `"Last"` for large arrays when only one item is needed.
- Avoid deep nesting beyond 5 levels for maintainability.
- Cache OAuth tokens to reduce authentication overhead.
- Use environment placeholders for URLs to simplify deployment.

## Security Considerations

- Always use HTTPS for external API calls.
- Store credentials encrypted and rotate them regularly.
- Never log decrypted credentials or tokens.
- Use permission-based access control for sensitive APIs.
- Set `IsProxyable` carefully to control external access.

