# Request Configuration Documentation

## Overview

Request configuration allows you to transform and structure data for API requests using JSON mapping rules. This system supports field mapping, constant values, array transformations, nested object construction, and secure request signing.

## Configuration Structure

All request configurations follow this basic structure:

```json
{
  "body": {
    // Configuration rules go here
  }
}
```

## Configuration Types

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
- When a source path doesn't exist, the field is set to `null`
- Arrays with missing sources return empty arrays `[]`

### Invalid Data Types
- Non-object items in arrays are skipped during mapping
- Invalid source paths are handled gracefully

### Signature Generation Errors
- Missing private key configuration will cause request to fail
- Invalid key formats will be logged and handled appropriately

## Security Considerations

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
  "body": {
    "requiredField": { "source": "guaranteed.field" },
    "optionalField": { "source": "might.not.exist" }
  }
}
```

### Array Processing
```json
// Process arrays efficiently
{
  "body": {
    "items": {
      "source": "orderItems",
      "transform": "map",
      "item": {
        "id": { "source": "productId" },
        "name": { "source": "productName" }
      }
    }
  }
}
```

## Common Use Cases

- **Payment Processing**: Transform order data for payment gateway APIs
- **Third-Party Integration**: Map internal data models to external API formats
- **Webhook Payloads**: Structure data for webhook notifications
- **Microservice Communication**: Transform data between service boundaries
- **Legacy System Integration**: Adapt modern data formats to legacy API requirements