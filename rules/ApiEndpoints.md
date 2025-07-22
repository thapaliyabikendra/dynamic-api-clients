# API Endpoints

## Overview

API endpoints define how clients interact with the Dynamic API Client system. The system supports both legacy and RESTful endpoint styles, permission-based access, proxy support, robust error handling, and advanced configuration overrides. This documentation provides a complete reference for automated systems and developers.

---

## 1. Endpoint Types

### 1.1 Legacy Endpoint (POST)

**Syntax:**
```http
POST /api/dynamic-http-client
Content-Type: application/json

{
  "apiClientName": "MBL_STAFF_DETAIL_INFO",
  "request": "{\"transactionId\": \"12345\"}"
}
```

**Full API Request:**
```http
POST /api/dynamic-http-client HTTP/1.1
Host: your-api.com
Content-Type: application/json
Authorization: Bearer <user_token>

{
  "apiClientName": "PAYMENT_GATEWAY_INTEGRATION",
  "request": "{\"orderId\": \"ORD-123\", \"totalAmount\": 99.99}"
}
```

**API Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Success",
  "data": {
    "transactionReference": "pg_ref_XYZ123",
    "paymentUrl": "https://payment.gateway.com/pay/XYZ123"
  }
}
```

**API Error Response (404 Not Found):**
```json
{
  "success": false,
  "code": 404,
  "message": "API Client 'INVALID_CLIENT_NAME' not found."
}
```

---

### 1.2 RESTful Endpoint (POST)

**Syntax:**
```http
POST /api/dynamic-http-client/{apiClientName}
Content-Type: application/json

{
  "request": "{\"transactionId\": \"12345\"}"
}
```

**Full API Request:**
```http
POST /api/dynamic-http-client/STAFF_INFO_API HTTP/1.1
Host: your-api.com
Content-Type: application/json
Authorization: Bearer <user_token>

{
  "request": {
    "branchCode": "001",
    "includeInactive": false
  }
}
```

**API Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Success",
  "data": {
    "totalCount": 1,
    "items": [
      {
        "employeeId": "EMP001",
        "fullName": "John Doe",
        "department": {
          "name": "Engineering"
        }
      }
    ]
  }
}
```

**API Error Response (403 Forbidden):**
```json
{
  "success": false,
  "code": 403,
  "message": "User is not authorized to access this API client."
}
```

---

## 2. Request Structure

- `apiClientName` (string): The system name of the API client configuration to use. **(Required)**
- `request` (string or object): The request payload, either as a JSON string or object. **(Required)**
- `requestConfig` (optional): Override the default request configuration (JSON string/object).
- `responseConfig` (optional): Override the default response configuration (JSON string/object).
- `headers` (optional): Additional headers to include in the request.
- `queryParams` (optional): Query string parameters for the endpoint.
- `pathParams` (optional): Path parameters for dynamic URL construction.

**Example with overrides:**
```json
{
  "apiClientName": "PAYMENT_API",
  "request": { "orderId": "ORD-001" },
  "requestConfig": { "body": { "currency": "USD" } },
  "responseConfig": { "data": { "source": "result" } },
  "headers": { "x-trace-id": "abc123" },
  "queryParams": { "debug": true },
  "pathParams": { "customerId": "C123" }
}
```

---

## 3. API Response Structure

**Success Response:**
```json
{
  "success": true,
  "code": 200,
  "message": "Success",
  "data": {
    // Mapped response data
  }
}
```

**Error Response:**
```json
{
  "success": false,
  "code": 400,  // or 401, 403, 404, 500
  "message": "Error description",
  "error": {
    // Mapped error data from ErrorResponseConfig
  }
}
```

**Versioning:**
- The API is versioned via the URL path or a custom header (e.g., `X-Api-Version`). Always check the documentation for the supported version.

---

## 4. Permission Requirements & Examples

Access to API endpoints is controlled through two main mechanisms: user permissions for regular internal APIs and API keys for external (proxyable) APIs.

### 4.1 Regular API (Internal Access)

- **Permission Required:** `AccessControlManagementSystemPermissions.DynamicApiClients.Default`
- **Header:** `Authorization: Bearer <user_token_with_required_permission>`

**Request Example:**
```http
POST /api/dynamic-http-client/STAFF_INFO_API HTTP/1.1
Host: your-api.com
Content-Type: application/json
Authorization: Bearer <user_token_with_required_permission>

{
  "request": { "branchCode": "001" }
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Success",
  "data": { "totalCount": 1, "items": [ ... ] }
}
```

**Error Response (403 Forbidden):**
```json
{
  "success": false,
  "code": 403,
  "message": "User is not authorized to access this API client."
}
```

### 4.2 Proxyable API (External Access)

- **Permission Required:** A valid `x-api-key` header matching the `Webhook:ApiKey` value in the server's configuration. The `ApiClient` must also be marked as `IsProxyable: true`.
- **Header:** `x-api-key: <valid-api-key>`

**Request Example:**
```http
POST /api/dynamic-http-client/EXTERNAL_WEBHOOK_HANDLER HTTP/1.1
Host: your-api.com
Content-Type: application/json
x-api-key: a-valid-and-configured-secret-key

{
  "request": { "payload": { "event": "item.created" } }
}
```

**Success Response (200 OK):**
```json
{
  "success": true,
  "code": 200,
  "message": "Success",
  "data": { "status": "received" }
}
```

**Error Response (401 Unauthorized):**
```json
{
  "success": false,
  "code": 401,
  "message": "Unauthorized"
}
```

---

## 5. Headers, Query Parameters, and Path Parameters

- **Headers:** You can include custom headers in your request. Some headers (like `Authorization` or `x-api-key`) are required for authentication.
- **Query Parameters:** Add query string parameters for filtering, pagination, or debugging.
- **Path Parameters:** Use path parameters for dynamic URL segments (e.g., `/api/dynamic-http-client/{apiClientName}`).

**Example:**
```json
{
  "headers": { "x-trace-id": "abc123" },
  "queryParams": { "page": 2, "limit": 10 },
  "pathParams": { "customerId": "C123" }
}
```

---

## 6. Error Handling

- **Missing API Client:** Returns 404 with a descriptive message.
- **Unauthorized (Proxy):** Returns 401 if `x-api-key` is missing or invalid.
- **Forbidden:** Returns 403 if the user lacks required permissions.
- **Validation Errors:** Returns 400 for invalid input or configuration.
- **Internal Errors:** Returns 500 for unexpected server errors.
- **ErrorResponseConfig:** Use custom error mapping for API-specific error formats.

**Error Response Example:**
```json
{
  "success": false,
  "code": 404,
  "message": "API Client not found"
}
```

---

## 7. Extensibility & Advanced Usage

- **Request/Response Overrides:** You can override the default request/response mapping for a single call.
- **Environment Placeholders:** Use placeholders in URLs (e.g., `{ACMS_API_BASE_URL}`) for environment-specific routing.
- **Multi-Step Orchestration:** Chain multiple API calls by using the output of one as input to another.
- **Batch Processing:** Send arrays of requests for bulk operations.
- **Pagination:** Use query parameters to paginate large result sets.
- **Versioning:** Specify API version in the URL or headers for backward compatibility.

**Multi-Step Example:**
```json
{
  "step1_getToken": {
    "url": "https://auth.api.com/token",
    "body": { "source": "ApiClientCredential OAUTH_CREDS" }
  },
  "step2_apiCall": {
    "url": "https://api.service.com/data",
    "headers": { "Authorization": "Bearer {step1_getToken.access_token}" }
  }
}
```

---

## 8. Best Practices

- Always use uppercase for `apiClientName`.
- Use the RESTful endpoint style for new integrations.
- Provide `x-api-key` for proxyable APIs.
- Use request/response config overrides for custom mapping needs.
- Handle all error response fields in your client.
- Validate your request payloads before sending.
- Monitor API usage and error rates for troubleshooting.
- Use environment placeholders for flexible deployments.
- Document custom flows and overrides for maintainability.

---

## 9. Common Use Cases

- Integrating with payment gateways, staff info, analytics, and more.
- Proxying requests to external APIs with secure API key validation.
- Standardizing API responses for frontend and downstream services.
- Supporting legacy and modern API integration patterns.
- Bulk/batch processing and pagination.
- Multi-step API orchestration and chaining.
- Environment-specific routing and configuration.

## Complete Examples

### Example 1: Payment Gateway Integration
```json
{
  "requestConfig": {
    "url": "https://payment.gateway.com/api/v2/process",
    "httpMethod": "POST",
    "contentType": "application/json",
    "headers": {
      "Accept": "application/json",
      "x-api-key": "PAYMENT_GATEWAY_API_KEY",
      "x-merchant-id": "MERCHANT_12345"
    },
    "body": {
      "transactionId": {
        "source": "orderId"
      },
      "amount": {
        "source": "totalAmount",
        "transform": "cast",
        "castTo": "string"
      },
      "currency": "USD",
      "customer": {
        "email": {
          "source": "customerEmail",
          "transform": "lowercase"
        },
        "name": {
          "source": ["firstName", "lastName"],
          "transform": "concat",
          "separator": " "
        }
      },
      "items": {
        "source": "orderItems",
        "transform": "map",
        "item": {
          "sku": {
            "source": "productId"
          },
          "name": {
            "source": "productName"
          },
          "quantity": {
            "source": "qty"
          },
          "price": {
            "source": "unitPrice"
          }
        }
      }
    }
  },
  "responseConfig": {
    "success": {
      "source": "status",
      "transform": "cast",
      "castTo": "bool"
    },
    "transactionReference": {
      "source": "data.reference"
    },
    "message": {
      "source": "message"
    },
    "paymentUrl": {
      "source": "data.redirectUrl"
    }
  }
}
```

### Example 2: Staff Information API with Signature
```json
{
  "requestConfig": {
    "url": "http://mblcmapi.iis/api/v1/connect",
    "httpMethod": "POST",
    "contentType": "application/json",
    "headers": {
      "Accept": "application/json",
      "Authorization": "Basic MBL_BASIC_AUTH_CREDENTIAL"
    },
    "body": {
      "FunctionName": "MBLStaffDetailInfo",
      "requestModel": {
        "encodeBase64": true,
        "generateSignature": true,
        "signaturePrivateKeySystemName": "MBL_PRIVATE_KEY",
        "TransactionId": {
          "source": "transactionId"
        },
        "BranchCode": {
          "source": "branchCode"
        },
        "IncludeInactive": false
      }
    }
  },
  "responseConfig": {
    "code": {
      "source": "Code"
    },
    "message": {
      "source": "Message"
    },
    "data": {
      "totalCount": {
        "source": "Data.totalCount"
      },
      "items": {
        "source": "Data.QueryResult",
        "transform": "map",
        "item": {
          "employeeId": {
            "source": "HREmployeeId"
          },
          "staffId": {
            "source": "StaffId"
          },
          "fullName": {
            "source": ["Title", "FirstName", "MiddleName", "LastName"],
            "transform": "concat",
            "separator": " "
          },
          "branch": {
            "id": {
              "source": "BranchId"
            },
            "name": {
              "source": "BranchName"
            },
            "code": {
              "source": "BranchCode"
            }
          },
          "department": {
            "id": {
              "source": "DepartmentId"
            },
            "name": {
              "source": "DepartmentName"
            },
            "code": {
              "source": "DepartmentCode"
            }
          },
          "position": {
            "title": {
              "source": "FunctionalTitle"
            },
            "designation": {
              "source": "Designation"
            },
            "level": {
              "source": "Level"
            }
          },
          "contact": {
            "email": {
              "source": "Email",
              "transform": "lowercase"
            },
            "mobile": {
              "source": "CIMobileNo"
            },
            "address": {
              "source": "Address"
            }
          },
          "status": {
            "isActive": {
              "source": "Status",
              "transform": "cast",
              "castTo": "bool"
            },
            "isRetired": {
              "source": "IsRetired"
            },
            "isResigned": {
              "source": "IsResigned"
            },
            "isHead": {
              "source": "IsHead"
            }
          },
          "dates": {
            "dateOfBirth": {
              "source": "DOBinAD"
            },
            "joiningDate": {
              "source": "JoiningDateEng"
            }
          }
        }
      }
    }
  }
}
```

### Example 3: OAuth Token Exchange
```json
{
  "requestConfig": {
    "url": "https://auth.provider.com/oauth/token",
    "httpMethod": "POST",
    "contentType": "application/x-www-form-urlencoded",
    "body": {
      "source": "ApiClientCredential OAUTH_CLIENT_CREDENTIALS"
    }
  },
  "responseConfig": {
    "accessToken": {
      "source": "access_token"
    },
    "tokenType": {
      "source": "token_type"
    },
    "expiresIn": {
      "source": "expires_in"
    },
    "scope": {
      "source": "scope"
    }
  }
}
```

### Example 4: Complex Data Transformation
```json
{
  "requestConfig": {
    "url": "https://api.analytics.com/v1/report",
    "httpMethod": "POST",
    "headers": {
      "Authorization": "Bearer ANALYTICS_API_TOKEN"
    },
    "body": {
      "reportType": "SALES_ANALYSIS",
      "dateRange": {
        "from": {
          "source": "startDate"
        },
        "to": {
          "source": "endDate"
        }
      },
      "filters": {
        "branches": {
          "source": "selectedBranches",
          "transform": "map",
          "item": {
            "code": {
              "source": "branchCode",
              "transform": "uppercase"
            },
            "includeSubBranches": true
          }
        },
        "productCategories": {
          "source": ["electronics", "appliances", "furniture"]
        }
      },
      "userId": {
        "source": "currentUserDetail.id"
      }
    }
  },
  "responseConfig": {
    "summary": {
      "totalSales": {
        "source": "data.aggregate.totalRevenue",
        "transform": "cast",
        "castTo": "string"
      },
      "transactionCount": {
        "source": "data.aggregate.count"
      },
      "averageTicket": {
        "source": "data.aggregate.avgTicketSize"
      }
    },
    "topProducts": {
      "source": "data.products",
      "transform": "map",
      "resultType": "List",
      "item": {
        "productId": {
          "source": "id"
        },
        "productName": {
          "source": "name"
        },
        "sales": {
          "source": "revenue"
        },
        "rank": {
          "source": "rank"
        }
      }
    },
    "branchPerformance": {
      "source": "data.branches",
      "transform": "map",
      "item": {
        "branchCode": {
          "source": "code"
        },
        "branchName": {
          "source": "name"
        },
        "metrics": {
          "revenue": {
            "source": "totalRevenue"
          },
          "growth": {
            "source": "growthRate",
            "transform": "cast",
            "castTo": "string"
          },
          "target": {
            "achievement": {
              "source": "targetAchievement"
            },
            "status": {
              "source": "targetMet",
              "transform": "cast",
              "castTo": "bool"
            }
          }
        }
      }
    }
  }
}
```
