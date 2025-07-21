# Response Configuration Documentation

## Overview

Response configuration allows you to transform and structure API response data using JSON mapping rules. This system enables you to extract specific fields, transform arrays, and reshape response data to match your application's requirements.

## Configuration Structure

Response configurations use mapping rules to define how external API responses should be transformed:

```json
{
  "destinationField": {
    "source": "responseField"
  }
}
```

## Configuration Types

### 1. Simple Response Mapping

Map individual fields from the API response to your desired output structure.

**Syntax:**
```json
{
  "outputField": {
    "source": "response.path.to.field"
  }
}
```

**Example:**
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

**API Response:**
```json
{
  "StatusCode": 200,
  "StatusMessage": "Transaction processed successfully",
  "Data": {
    "TransactionReference": "TXN-2024-001234",
    "ProcessingTime": "2024-01-15T10:30:45.123Z"
  }
}
```

**Transformed Output:**
```json
{
  "status": 200,
  "message": "Transaction processed successfully",
  "transactionId": "TXN-2024-001234"
}
```

### 2. Array Response Mapping

Transform arrays from API responses by applying mapping rules to each item.

**Syntax:**
```json
{
  "outputArray": {
    "source": "response.array.path",
    "transform": "map",
    "item": {
      "outputField": {
        "source": "sourceField"
      }
    }
  }
}
```

**Example:**
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

**API Response:**
```json
{
  "Data": {
    "totalCount": 3,
    "QueryResult": [
      {
        "HREmployeeId": "EMP001",
        "EmployeeName": "John Doe",
        "DepartmentName": "Engineering",
        "Salary": 75000
      },
      {
        "HREmployeeId": "EMP002",
        "EmployeeName": "Jane Smith", 
        "DepartmentName": "Marketing",
        "Salary": 65000
      },
      {
        "HREmployeeId": "EMP003",
        "EmployeeName": "Bob Johnson",
        "DepartmentName": "Sales",
        "Salary": 70000
      }
    ]
  }
}
```

**Transformed Output:**
```json
{
  "data": {
    "totalCount": 3,
    "items": [
      {
        "id": "EMP001",
        "name": "John Doe",
        "department": "Engineering"
      },
      {
        "id": "EMP002", 
        "name": "Jane Smith",
        "department": "Marketing"
      },
      {
        "id": "EMP003",
        "name": "Bob Johnson",
        "department": "Sales"
      }
    ]
  }
}
```

### 3. Result Type Options

Control how arrays are processed by specifying result types: `First`, `Last`, or `List` (default).

**Syntax:**
```json
{
  "outputField": {
    "source": "response.array.path",
    "transform": "map",
    "resultType": "First|Last|List",
    "item": {
      "field": { "source": "sourceField" }
    }
  }
}
```

**Example:**
```json
{
  "firstItem": {
    "source": "Data.Results",
    "transform": "map",
    "resultType": "First",
    "item": {
      "id": { "source": "Id" },
      "name": { "source": "Name" }
    }
  },
  "lastItem": {
    "source": "Data.Results",
    "transform": "map",
    "resultType": "Last",
    "item": {
      "id": { "source": "Id" },
      "name": { "source": "Name" }
    }
  },
  "allItems": {
    "source": "Data.Results",
    "transform": "map",
    "resultType": "List",
    "item": {
      "id": { "source": "Id" },
      "name": { "source": "Name" }
    }
  }
}
```

**API Response:**
```json
{
  "Data": {
    "Results": [
      {
        "Id": "ITEM001",
        "Name": "First Item",
        "Priority": 1
      },
      {
        "Id": "ITEM002",
        "Name": "Second Item", 
        "Priority": 2
      },
      {
        "Id": "ITEM003",
        "Name": "Third Item",
        "Priority": 3
      }
    ]
  }
}
```

**Transformed Output:**
```json
{
  "firstItem": {
    "id": "ITEM001",
    "name": "First Item"
  },
  "lastItem": {
    "id": "ITEM003",
    "name": "Third Item"
  },
  "allItems": [
    {
      "id": "ITEM001",
      "name": "First Item"
    },
    {
      "id": "ITEM002",
      "name": "Second Item"
    },
    {
      "id": "ITEM003",
      "name": "Third Item"
    }
  ]
}
```

## Result Type Details

### List (Default)
- Returns all mapped items as an array
- Use when you need to process all items from the source array
- Returns empty array `[]` if source is empty

### First
- Returns only the first mapped item as a single object
- Use when you only need the first item from a collection
- Returns `null` if source array is empty

### Last
- Returns only the last mapped item as a single object  
- Use when you need the most recent or final item
- Returns `null` if source array is empty

## Advanced Examples

### Complex Nested Response Mapping

**Configuration:**
```json
{
  "order": {
    "id": {
      "source": "OrderData.OrderId"
    },
    "customer": {
      "name": {
        "source": "OrderData.Customer.FullName"
      },
      "email": {
        "source": "OrderData.Customer.ContactInfo.Email"
      }
    },
    "items": {
      "source": "OrderData.LineItems",
      "transform": "map",
      "item": {
        "productId": {
          "source": "Product.SKU"
        },
        "productName": {
          "source": "Product.DisplayName"
        },
        "quantity": {
          "source": "OrderedQuantity"
        },
        "unitPrice": {
          "source": "Product.Price.Amount"
        }
      }
    },
    "summary": {
      "itemCount": {
        "source": "OrderData.TotalItems"
      },
      "totalAmount": {
        "source": "OrderData.OrderTotal.GrandTotal"
      }
    }
  }
}
```

### Conditional Field Mapping

**Configuration:**
```json
{
  "user": {
    "id": {
      "source": "UserProfile.UserId"
    },
    "displayName": {
      "source": "UserProfile.DisplayName"
    },
    "avatar": {
      "source": "UserProfile.ProfileImage.Url"
    },
    "preferences": {
      "source": "UserProfile.Settings",
      "transform": "map",
      "resultType": "First",
      "item": {
        "theme": {
          "source": "UITheme"
        },
        "language": {
          "source": "PreferredLanguage"
        }
      }
    }
  }
}
```

## Path Syntax

### Dot Notation
Access nested properties using dots:
- `"Data.User.Name"` → response.Data.User.Name
- `"Result.Items.Product.Price"` → response.Result.Items.Product.Price

### Array Indexing
Access specific array elements:
- `"Items[0].Name"` → First item's name
- `"Results[2].Id"` → Third result's ID

### Combined Paths
Mix dot notation and array indexing:
- `"Data.Orders[0].Customer.Email"`
- `"Response.Results[1].Product.Details.Description"`

## Error Handling

### Missing Fields
- When a source path doesn't exist in the response, the output field is set to `null`
- Missing nested paths are handled gracefully

### Empty Arrays
- **List**: Returns empty array `[]`
- **First/Last**: Returns `null`

### Invalid Response Structure
- Malformed JSON responses are handled with appropriate error logging
- Non-array sources with array transforms return empty results

## Best Practices

### Use Descriptive Field Names
```json
// Good: Clear and meaningful
{
  "customerId": { "source": "Data.Customer.Id" },
  "customerEmail": { "source": "Data.Customer.Email" }
}

// Avoid: Unclear abbreviations
{
  "cId": { "source": "Data.Customer.Id" },
  "cEmail": { "source": "Data.Customer.Email" }
}
```

### Handle Optional Data
```json
{
  "requiredField": { "source": "Data.Required" },
  "optionalField": { "source": "Data.Optional" },
  "defaultValue": "fallback_value"
}
```

### Organize Complex Mappings
```json
{
  "metadata": {
    "timestamp": { "source": "ResponseTime" },
    "version": { "source": "ApiVersion" }
  },
  "payload": {
    "data": { "source": "ActualData" }
  }
}
```

### Optimize Array Processing
```json
// Use appropriate result types
{
  "latestOrder": {
    "source": "Orders",
    "transform": "map", 
    "resultType": "Last",
    "item": { "id": { "source": "OrderId" } }
  },
  "firstProduct": {
    "source": "Products",
    "transform": "map",
    "resultType": "First", 
    "item": { "name": { "source": "ProductName" } }
  }
}
```

## Common Use Cases

- **API Response Normalization**: Transform external API responses to match internal data models
- **Data Aggregation**: Extract and combine data from complex nested responses
- **Frontend Data Preparation**: Shape API responses for UI consumption
- **Webhook Processing**: Transform incoming webhook payloads
- **Legacy System Integration**: Adapt old API response formats to modern applications
- **Multi-Source Data Combination**: Merge data from multiple API responses
- **Error Response Handling**: Extract error information from failed API calls

## Performance Considerations

### Large Arrays
- Consider using `First` or `Last` result types when processing large arrays if you don't need all items
- Implement pagination for very large datasets

### Deep Nesting
- Deeply nested source paths may impact performance
- Consider flattening complex structures when possible

### Memory Usage
- Large response transformations consume memory
- Monitor memory usage with high-volume API processing