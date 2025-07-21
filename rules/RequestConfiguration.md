# Request Configuration Documentation

## Overview

Request configuration allows you to define and structure the entire outgoing API request, including the URL, HTTP method, headers, and body. It uses JSON mapping rules to transform source data into a complete and valid request.

## Complete Request Configuration Structure

A full request configuration object can include the following top-level properties:

```json
{
  "url": "https://api.example.com/v1/resource/{id}",
  "httpMethod": "POST",
  "contentType": "application/json",
  "headers": {
    "Accept": "application/json",
    "x-custom-header": "value"
  },
  "pathParams": {
    "id": { "source": "resourceId" }
  },
  "queryParams": {
    "filter": "active"
  },
  "body": {
    // Body mapping rules go here
  }
}
```

| Property      | Optional | Description                                                       |
| ------------- | -------- | ----------------------------------------------------------------- |
| `url`         | No       | The target URL for the API request. Can contain placeholders.     |
| `httpMethod`  | Yes      | The HTTP method (e.g., "GET", "POST", "PUT"). Defaults to "POST". |
| `contentType` | Yes      | The `Content-Type` of the request body. Defaults to `application/json`. |
| `headers`     | Yes      | An object defining static or dynamic request headers.             |
| `pathParams`  | Yes      | An object for replacing URL path placeholders (e.g., `{id}`).     |
| `queryParams` | Yes      | An object for adding query string parameters to the URL.          |
| `body`        | Yes      | An object containing the mapping rules for the request body.      |

---

## URL Configuration

### 1. Path Parameters

Dynamically replace placeholders in the `url` path.

```json
{
  "url": "https://api.service.com/users/{userId}/posts/{postId}",
  "pathParams": {
    "userId": { "source": "userInfo.id" },
    "postId": { "source": "post.identifier" }
  }
}
```

### 2. Query Parameters

Add static or dynamic query parameters to the URL.

```json
{
  "url": "https://api.service.com/search",
  "queryParams": {
    "query": { "source": "searchQuery" },
    "limit": 25,
    "sort": "desc"
  }
}
```

### 3. Environment Placeholders

Use placeholders for environment-specific base URLs to avoid hardcoding.

```json
{
  "url": "{ACMS_API_BASE_URL}/api/v1/users"
}
```

*Supported placeholders are configured system-wide.*

---

## Headers and Content Type

### 1. Headers Configuration

Define static or dynamic headers for the request.

```json
{
  "headers": {
    "Accept": "application/json",
    "x-request-id": { "source": "transactionId" },
    "Authorization": "Bearer YOUR_STATIC_TOKEN"
  }
}
```

### 2. Content-Type Handling

The `contentType` property determines how the `body` is formatted.

#### application/x-www-form-urlencoded

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

#### multipart/form-data

This is typically used for file uploads and is handled by the service when it detects file-like objects in the source.

```json
{
  "contentType": "multipart/form-data",
  "body": {
    "file": { "source": "document.content" },
    "metadata": { "source": "document.info" }
  }
}
```

---

## Body Configuration

The `body` object contains the rules for constructing the request payload.

### 1. Simple Field Mapping

Map values from source data to request fields using dot notation paths.

**Syntax:**
```json
{
  "body": {
    "destinationField": {
      "source": "sourceField"
    }
  }
}
```

**Example:**
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

**Input Data:**
```json
{
  "orderId": "TXN123456",
  "payment": {
    "totalAmount": 299.99
  }
}
```

**Generated Request:**
```json
{
  "transactionId": "TXN123456",
  "amount": 299.99
}
```

### 2. Constant Values

Set static values directly in the request configuration.

**Syntax:**
```json
{
  "body": {
    "fieldName": "constant_value"
  }
}
```

**Example:**
```json
{
  "body": {
    "apiVersion": "v2",
    "channel": "WEB"
  }
}
```

**Generated Request:**
```json
{
  "apiVersion": "v2",
  "channel": "WEB"
}
```

### 3. Array Mapping

Transform arrays by applying mapping rules to each item in the source array.

**Syntax:**
```json
{
  "body": {
    "destinationArray": {
      "source": "sourceArray",
      "transform": "map",
      "item": {
        "destinationField": {
          "source": "sourceField"
        }
      }
    }
  }
}
```

**Example:**
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

**Input Data:**
```json
{
  "orderItems": [
    {
      "sku": "PROD001",
      "qty": 2,
      "unitPrice": 49.99
    },
    {
      "sku": "PROD002", 
      "qty": 1,
      "unitPrice": 19.99
    }
  ]
}
```

**Generated Request:**
```json
{
  "items": [
    {
      "productId": "PROD001",
      "quantity": 2,
      "price": 49.99
    },
    {
      "productId": "PROD002",
      "quantity": 1,
      "price": 19.99
    }
  ]
}
```

### 4. Nested Object Construction

Create complex nested structures by organizing mapping rules hierarchically.

**Syntax:**
```json
{
  "body": {
    "parentObject": {
      "childObject": {
        "field": {
          "source": "sourceField"
        }
      }
    }
  }
}
```

**Example:**
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

**Input Data:**
```json
{
  "user": {
    "fname": "John",
    "lname": "Doe",
    "email": "john.doe@example.com",
    "mobile": "+1-555-0123"
  }
}
```

**Generated Request:**
```json
{
  "customer": {
    "personalInfo": {
      "firstName": "John",
      "lastName": "Doe"
    },
    "contact": {
      "email": "john.doe@example.com",
      "phone": "+1-555-0123"
    }
  }
}
```

### 5. Request Model with Signature

Create secure requests with Base64 encoding and RSA signature generation.

**Syntax:**
```json
{
  "body": {
    "requestModel": {
      "encodeBase64": true,
      "generateSignature": true,
      "signaturePrivateKeySystemName": "PRIVATE_KEY_NAME",
      "field1": {
        "source": "sourceField1"
      },
      "field2": {
        "source": "sourceField2"
      }
    }
  }
}
```

**Example:**
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

**Input Data:**
```json
{
  "transactionId": "TXN789012",
  "amount": 150.00
}
```

**Processing Steps:**
1. Creates request model object:
   ```json
   {
     "TransactionId": "TXN789012",
     "Amount": 150.00
   }
   ```

2. Encodes as Base64 (if `encodeBase64: true`)
3. Generates RSA signature using specified private key
4. Creates final structured output

**Generated Request:**
```json
{
  "Data": "eyJUcmFuc2FjdGlvbklkIjoiVFhONzg5MDEyIiwiQW1vdW50IjoxNTAuMDB9",
  "Signature": "rsa_signature_hash_here",
  "TimeStamp": "2024-01-15T10:30:45.123"
}
```

### 6. Rules Engine Integration

Execute a pre-defined set of business rules from the Rules Engine and use the result as the value for a field.

**Syntax:**
```json
{
  "body": {
    "fieldName": {
      "source": "sourceObjectForRules",
      "rulesEngine": "RULE_SET_NAME"
    }
  }
}
```

**Example:**
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

## Configuration Options

### Request Model Options

| Option | Type | Description |
|--------|------|-------------|
| `encodeBase64` | Boolean | Encode the request model as Base64 |
| `generateSignature` | Boolean | Generate RSA signature for the request |
| `signaturePrivateKeySystemName` | String | System name of the private key to use for signing |

### Transform Types

| Transform | Description |
|-----------|-------------|
| `map` | Apply mapping rules to each item in an array |

## Path Syntax

### Dot Notation
Access nested properties using dots:
- `"user.name"` → source.user.name
- `"payment.details.amount"` → source.payment.details.amount

### Array Indexing
Access specific array elements:
- `"items[0].price"` → First item's price
- `"users[2].email"` → Third user's email

### Combined Paths
Mix dot notation and array indexing:
- `"orders[0].customer.email"`
- `"response.data.items[1].product.name"`

## Error Handling

### Missing Source Fields
- When a source path for a `body`, `pathParams`, or `queryParams` field doesn't exist, it is set to `null` or omitted.
- An undefined `url` will result in a configuration error.
- Arrays with missing sources return empty arrays `[]`.

### Invalid Data Types
- Non-object items in arrays are skipped during mapping
- Invalid source paths are handled gracefully

### Signature Generation Errors
- Missing private key configuration will cause the request to fail.
- Invalid key formats will be logged and handled appropriately.

## Security Considerations

### URL and Parameter Security
- Never include sensitive data like tokens or passwords directly in the URL. Use the `headers` section for authentication.
- Be cautious with dynamic `pathParams` and `queryParams` to prevent injection vulnerabilities if the source data is user-provided. Sanitize inputs where possible.

### Private Key Management
- Private keys should be stored securely in the system configuration
- Use descriptive but non-revealing key names (e.g., `MBL_PRIVATE_KEY`)
- Keys should be rotated regularly according to security policies

### Data Encoding
- Base64 encoding helps ensure data integrity during transmission
- Always validate decoded data on the receiving end

### Signature Verification
- Recipients should verify signatures using the corresponding public key
- Include timestamp validation to prevent replay attacks

## Best Practices

### Configuration Organization
```json
// Good: Group related fields
{
  "body": {
    "transaction": {
      "id": { "source": "transactionId" },
      "amount": { "source": "amount" }
    },
    "customer": {
      "email": { "source": "user.email" }
    }
  }
}
```

### Error Prevention
```json
// Handle optional fields gracefully
{
  "queryParams": {
    "requiredField": { "source": "guaranteed.field" },
    "optionalField": { "source": "might.not.exist" }
  },
  "body": {
    "requiredField": { "source": "guaranteed.field" },
    "optionalField": { "source": "might.not.exist" }
  }
}
```

### Array Processing

When building a request body, you often need to transform an array from your source data into a different array structure for the destination API. This is where array processing comes in.

In the `RequestConfiguration.md` file, this is covered under the **"Body Configuration" -> "3. Array Mapping"** section.

**Example from the documentation:**

```json
{
  "body": {
    "items": {
      "source": "orderItems",
      "transform": "map",
      "item": {
        "productId": { "source": "sku" },
        "quantity": { "source": "qty" },
        "price": { "source": "unitPrice" }
      }
    }
  }
}
```

**What this does:**

1.  It looks for the `orderItems` array in your source data.
2.  It iterates through each object in that array.
3.  For each object, it applies the rules inside the `item` block, mapping `sku` to `productId`, `qty` to `quantity`, and so on.
4.  It constructs a new `items` array in the final request body with the transformed objects.

### Relationship to `ArrayMapping.md`

The functionality itself (`"transform": "map"`) is the core concept of array mapping.

*   `RequestConfiguration.md` shows you **where** and **how** to use array mapping *within the specific context of building a request body*.
*   `ArrayMapping.md` is the **detailed, comprehensive documentation** for the array mapping feature itself, including all its advanced capabilities like `resultType`, pagination handling, and batch processing.

Think of it this way: `RequestConfiguration.md` tells you "You can process arrays here", while `ArrayMapping.md` tells you "Here is everything you can possibly do when you process an array".

The "Array Processing" part of `RequestConfiguration.md` is sufficient as is, because it demonstrates the most common use case within a request. For more advanced array operations, a user would refer to the dedicated `ArrayMapping.md` document we've already improved.