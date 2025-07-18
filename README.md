# dynamic-api-clients

# JsonMapperService Documentation for Dynamic API Client

## Table of Contents
1. [Overview](#overview)
2. [Core Concepts](#core-concepts)
3. [Authentication Methods](#authentication-methods)
4. [Request Configuration](#request-configuration)
5. [Response Configuration](#response-configuration)
6. [Transformation Functions](#transformation-functions)
7. [Special Features](#special-features)
8. [Complete Examples](#complete-examples)
9. [Error Handling](#error-handling)
10. [Best Practices](#best-practices)

## Overview

The `JsonMapperService` is a powerful JSON transformation engine designed for the Dynamic API Client system. It enables flexible mapping between different JSON structures, handles various authentication methods, and supports complex data transformations.

### Key Features
- Dynamic JSON-to-JSON mapping using rule-based configurations
- Multiple authentication methods (OAuth, Basic Auth, API Key, PEM certificates)
- Data transformation capabilities (encoding, encryption, signatures)
- Support for nested objects and array manipulations
- Integration with Rules Engine for conditional logic
- Current user context awareness

### Basic Usage Flow
```
Source JSON + Mapping Rules → JsonMapperService → Transformed JSON
```

## Core Concepts

### 1. Source Path Navigation
The service uses dot notation to navigate through JSON structures:
- Simple property: `"propertyName"`
- Nested property: `"parent.child.property"`
- Array element: `"items[0].name"`
- Deep nesting: `"data.results[2].details.info"`

### 2. Mapping Rule Structure
Rules are defined as JSON objects where:
- **Key**: Destination property name
- **Value**: Rule definition (constant, source mapping, or transformation)

```json
{
  "destinationProperty": {
    "source": "sourceProperty",
    "transform": "transformationType"
  }
}
```

## Authentication Methods

### 1. API Key Authentication
```json
{
  "headers": {
    "x-api-key": "API_KEY_CREDENTIAL_SYSTEM_NAME"
  }
}
```

**Example:**
```json
{
  "headers": {
    "x-api-key": "PAYMENT_GATEWAY_API_KEY"
  }
}
```

The service will:
1. Look up the credential by system name
2. Decrypt the stored API key
3. Set it in the header

### 2. Basic Authentication
```json
{
  "headers": {
    "Authorization": "Basic CREDENTIAL_SYSTEM_NAME"
  }
}
```

**Example:**
```json
{
  "headers": {
    "Authorization": "Basic MBL_BASIC_AUTH_CREDENTIAL"
  }
}
```

The service will:
1. Retrieve username and password from credential store
2. Decrypt the password
3. Create Base64 encoded "username:password"
4. Set "Basic {encoded_credentials}" in Authorization header

### 3. OAuth Bearer Token
```json
{
  "headers": {
    "Authorization": "Bearer API_CLIENT_SYSTEM_NAME"
  }
}
```

**Special Cases:**
- `"Bearer CURRENT_USER_TOKEN"` - Uses the current user's authentication token
- `"Bearer EXTERNAL_API_CLIENT"` - Fetches OAuth token for external API

**Example:**
```json
{
  "headers": {
    "Authorization": "Bearer PAYMENT_SERVICE_OAUTH"
  }
}
```

### 4. OAuth Client Credentials
```json
{
  "body": {
    "source": "ApiClientCredential OAUTH_CREDENTIAL_NAME"
  }
}
```

**Example:**
```json
{
  "body": {
    "source": "ApiClientCredential AZURE_AD_CLIENT_CREDENTIALS"
  }
}
```

This will inject OAuth credentials (client_id, client_secret, grant_type) into the request body.

## Request Configuration

### 1. Simple Field Mapping
```json
{
  "body": {
    "transactionId": {
      "source": "orderId"
    },
    "amount": {
      "source": "payment.totalAmount"
    }
  }
}
```

### 2. Constant Values
```json
{
  "body": {
    "apiVersion": "v2",
    "channel": "WEB"
  }
}
```

### 3. Array Mapping
```json
{
  "body": {
    "items": {
      "source": "orderItems",
      "transform": "map",
      "item": {
        "productId": {
          "source": "sku"
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
}
```

### 4. Nested Object Construction
```json
{
  "body": {
    "customer": {
      "personalInfo": {
        "firstName": {
          "source": "user.fname"
        },
        "lastName": {
          "source": "user.lname"
        }
      },
      "contact": {
        "email": {
          "source": "user.email"
        },
        "phone": {
          "source": "user.mobile"
        }
      }
    }
  }
}
```

### 5. Request Model with Signature
```json
{
  "body": {
    "requestModel": {
      "encodeBase64": true,
      "generateSignature": true,
      "signaturePrivateKeySystemName": "MBL_PRIVATE_KEY",
      "TransactionId": {
        "source": "transactionId"
      },
      "Amount": {
        "source": "amount"
      }
    }
  }
}
```

This will:
1. Create the request model object
2. Encode it as Base64 (if `encodeBase64: true`)
3. Generate RSA signature using the specified private key
4. Structure the output as:
```json
{
  "Data": "base64_encoded_request_model",
  "Signature": "rsa_signature",
  "TimeStamp": "2024-01-15T10:30:45.123"
}
```

## Response Configuration

### 1. Simple Response Mapping
```json
{
  "status": {
    "source": "StatusCode"
  },
  "message": {
    "source": "StatusMessage"
  },
  "transactionId": {
    "source": "Data.TransactionReference"
  }
}
```

### 2. Array Response Mapping
```json
{
  "data": {
    "totalCount": {
      "source": "Data.totalCount"
    },
    "items": {
      "source": "Data.QueryResult",
      "transform": "map",
      "item": {
        "id": {
          "source": "HREmployeeId"
        },
        "name": {
          "source": "EmployeeName"
        },
        "department": {
          "source": "DepartmentName"
        }
      }
    }
  }
}
```

### 3. Result Type Options
```json
{
  "firstItem": {
    "source": "Data.Results",
    "transform": "map",
    "resultType": "First",
    "item": {
      "id": { "source": "Id" }
    }
  },
  "lastItem": {
    "source": "Data.Results",
    "transform": "map",
    "resultType": "Last",
    "item": {
      "id": { "source": "Id" }
    }
  },
  "allItems": {
    "source": "Data.Results",
    "transform": "map",
    "resultType": "List",
    "item": {
      "id": { "source": "Id" }
    }
  }
}
```

## Transformation Functions

### 1. String Transformations

#### Uppercase
```json
{
  "countryCode": {
    "source": "country",
    "transform": "uppercase"
  }
}
```

#### Lowercase
```json
{
  "email": {
    "source": "Email",
    "transform": "lowercase"
  }
}
```

### 2. Concatenation
```json
{
  "fullName": {
    "source": ["firstName", "middleName", "lastName"],
    "transform": "concat",
    "separator": " "
  }
}
```

### 3. Type Casting
```json
{
  "isActive": {
    "source": "Status",
    "transform": "cast",
    "castTo": "bool"
  },
  "statusCode": {
    "source": "ResponseCode",
    "transform": "cast",
    "castTo": "int"
  },
  "isFlagged": {
    "source": "FlagStatus",
    "transform": "cast",
    "castTo": "bit"
  }
}
```

Supported cast types:
- `"string"` - Converts to string
- `"int"` or `"integer"` - Converts to integer
- `"bool"` or `"boolean"` - Converts to boolean
- `"bit"` - Converts boolean to 1/0

### 4. String Replacement

#### Single Replacement
```json
{
  "formattedPhone": {
    "source": "phoneNumber",
    "transform": "replace",
    "replaceFrom": "-",
    "replaceTo": ""
  }
}
```

#### Multiple Replacements
```json
{
  "cleanedText": {
    "source": "description",
    "transform": "replace",
    "replaceList": [
      { "from": "<br>", "to": "\n" },
      { "from": "&amp;", "to": "&" },
      { "from": "&lt;", "to": "<" }
    ]
  }
}
```

## Special Features

### 1. Current User Details
Access current user information using special placeholders:

```json
{
  "body": {
    "userId": {
      "source": "currentUserDetail.id"
    },
    "userName": {
      "source": "currentUserDetail.username"
    },
    "userEmail": {
      "source": "currentUserDetail.email"
    },
    "fullName": {
      "source": "currentUserDetail.fullname"
    }
  }
}
```

Available properties:
- `currentUserDetail.id`
- `currentUserDetail.username`
- `currentUserDetail.email`
- `currentUserDetail.name`
- `currentUserDetail.surname`
- `currentUserDetail.fullname`
- `currentUserDetail.isauthenticated`

### 2. Rules Engine Integration
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

### 3. Complex Array Source
```json
{
  "combinedData": {
    "source": ["users.active", "users.pending", "users.inactive"],
    "transform": "concat",
    "separator": ","
  }
}
```

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

## Error Handling

### 1. Missing Credentials
When a credential system name is not found:
- API Key: Sets empty string in header
- Basic Auth: Sets empty string in Authorization
- OAuth: Returns empty Bearer token

### 2. Inactive Credentials
The service checks `IsActive` flag and returns empty values for inactive credentials.

### 3. Invalid Paths
When a source path doesn't exist:
- Simple mappings: Property is not included in output
- Array mappings: Returns empty array
- Nested objects: Creates empty object structure

### 4. Transformation Errors
- Invalid cast operations: Returns original value
- Missing transformation parameters: Uses defaults (e.g., space for concat separator)
- Array operations on non-arrays: Returns empty array

## Best Practices

### 1. Credential Management
- Use uppercase for credential system names: `PAYMENT_API_KEY`
- Implement credential rotation without changing configurations
- Store sensitive data encrypted in the credential store

### 2. Performance Optimization
- Avoid deep nesting beyond 5 levels
- Use `resultType: "First"` when only needing one item from arrays
- Cache OAuth tokens to minimize token requests

### 3. Security Considerations
- Always use HTTPS for external APIs
- Implement request signing for sensitive operations
- Use appropriate authentication method for each API
- Never log decrypted credentials or tokens

### 4. Configuration Organization
- Group related mappings together
- Use consistent naming conventions
- Comment complex transformations
- Validate configurations before deployment

### 5. Error Handling Strategy
```json
{
  "responseConfig": {
    "success": {
      "source": "status",
      "transform": "cast",
      "castTo": "bool"
    },
    "errorCode": {
      "source": "error.code"
    },
    "errorMessage": {
      "source": "error.message"
    },
    "data": {
      "source": "data",
      "condition": "success == true"
    }
  }
}
```

### 6. Testing Recommendations
- Test with missing source properties
- Verify authentication credential handling
- Test array transformations with empty arrays
- Validate signature generation with test keys
- Check response mapping with various response structures

## Advanced Scenarios

### 1. Conditional Mapping with Rules Engine
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
    }
  }
}
```

### 2. Multi-Step Authentication
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
      "Authorization": "Bearer {tokenFromStep1}"
    }
  }
}
```

### 3. Dynamic URL Construction
```json
{
  "url": "https://api.service.com/v2/customers/{customerId}/orders",
  "urlParameters": {
    "customerId": {
      "source": "customer.id"
    }
  }
}
```

## Troubleshooting Guide

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

This documentation provides a comprehensive guide to using the JsonMapperService for dynamic API integrations. The service's flexibility allows for complex data transformations while maintaining security through proper credential management and authentication handling.