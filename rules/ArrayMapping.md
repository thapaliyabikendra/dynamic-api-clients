# Array Mapping

## Overview
Array mapping allows you to transform arrays by applying mapping rules to each item and controlling the result format. You can process entire arrays, get specific items, or transform individual array elements.

---

## Basic Array Mapping Syntax

```json
{
  "destinationArray": {
    "source": "sourceArrayPath",
    "transform": "map",
    "resultType": "List|First|Last",
    "item": {
      // Mapping rules for each array item
    }
  }
}
```

---

## Result Types

### List (Default)
Returns all mapped items as an array:

```json
{
  "allItems": {
    "source": "products",
    "transform": "map",
    "resultType": "List",
    "item": {
      "id": { "source": "productId" },
      "name": { "source": "productName" }
    }
  }
}
```

### First
Returns only the first mapped item (or null if array is empty):

```json
{
  "firstItem": {
    "source": "products",
    "transform": "map",
    "resultType": "First",
    "item": {
      "id": { "source": "productId" },
      "name": { "source": "productName" }
    }
  }
}
```

### Last
Returns only the last mapped item (or null if array is empty):

```json
{
  "lastItem": {
    "source": "products",
    "transform": "map",
    "resultType": "Last",
    "item": {
      "id": { "source": "productId" },
      "name": { "source": "productName" }
    }
  }
}
```

---

## Examples

### Example 1: Product List Transformation

**Source JSON**:
```json
{
  "products": [
    {
      "productId": "P001",
      "productName": "Laptop",
      "price": 999.99,
      "category": "Electronics"
    },
    {
      "productId": "P002",
      "productName": "Mouse",
      "price": 29.99,
      "category": "Electronics"
    },
    {
      "productId": "P003",
      "productName": "Desk",
      "price": 199.99,
      "category": "Furniture"
    }
  ]
}
```

**Mapping Rules**:
```json
{
  "items": {
    "source": "products",
    "transform": "map",
    "resultType": "List",
    "item": {
      "id": { "source": "productId" },
      "name": { "source": "productName" },
      "cost": { "source": "price" }
    }
  }
}
```

**Result**:
```json
{
  "items": [
    { "id": "P001", "name": "Laptop", "cost": 999.99 },
    { "id": "P002", "name": "Mouse", "cost": 29.99 },
    { "id": "P003", "name": "Desk", "cost": 199.99 }
  ]
}
```

---

### Example 2: Get First Item Only

```json
{
  "featuredProduct": {
    "source": "products",
    "transform": "map",
    "resultType": "First",
    "item": {
      "id": { "source": "productId" },
      "name": { "source": "productName" },
      "cost": { "source": "price" }
    }
  }
}
```

**Result**:
```json
{
  "featuredProduct": {
    "id": "P001",
    "name": "Laptop",
    "cost": 999.99
  }
}
```

---

### Example 3: Get Last Item Only

```json
{
  "newestProduct": {
    "source": "products",
    "transform": "map",
    "resultType": "Last",
    "item": {
      "id": { "source": "productId" },
      "name": { "source": "productName" },
      "cost": { "source": "price" }
    }
  }
}
```

**Result**:
```json
{
  "newestProduct": {
    "id": "P003",
    "name": "Desk",
    "cost": 199.99
  }
}
```

---

### Example 4: Complex Nested Array Mapping

**Source JSON**:
```json
{
  "orders": [
    {
      "orderId": "ORD001",
      "customer": {
        "name": "John Doe",
        "email": "john@example.com"
      },
      "items": [
        { "product": "Laptop", "quantity": 1 },
        { "product": "Mouse", "quantity": 2 }
      ]
    },
    {
      "orderId": "ORD002",
      "customer": {
        "name": "Jane Smith",
        "email": "jane@example.com"
      },
      "items": [
        { "product": "Desk", "quantity": 1 }
      ]
    }
  ]
}
```

**Mapping Rules**:
```json
{
  "processedOrders": {
    "source": "orders",
    "transform": "map",
    "resultType": "List",
    "item": {
      "orderNumber": { "source": "orderId" },
      "customerName": { "source": "customer.name" },
      "customerEmail": { "source": "customer.email" },
      "itemCount": { "source": "items" },
      "firstItem": { "source": "items[0].product" }
    }
  }
}
```

**Result**:
```json
{
  "processedOrders": [
    {
      "orderNumber": "ORD001",
      "customerName": "John Doe",
      "customerEmail": "john@example.com",
      "itemCount": [
        { "product": "Laptop", "quantity": 1 },
        { "product": "Mouse", "quantity": 2 }
      ],
      "firstItem": "Laptop"
    },
    {
      "orderNumber": "ORD002",
      "customerName": "Jane Smith",
      "customerEmail": "jane@example.com",
      "itemCount": [
        { "product": "Desk", "quantity": 1 }
      ],
      "firstItem": "Desk"
    }
  ]
}
```

---

## Advanced Scenarios

### 1. Handling Paginated Responses
For APIs that return data in pages, you can map the items list and pagination details into a structured object.

**Source JSON**:
```json
{
  "page": 2,
  "totalPages": 10,
  "hasMore": true,
  "links": {
    "next": "https://api.service.com/items?page=3"
  },
  "results": [
    { "id": "item101", "name": "Item 101" },
    { "id": "item102", "name": "Item 102" }
  ]
}
```

**Mapping Rules**:
```json
{
  "items": {
    "source": "results",
    "transform": "map",
    "item": {
      "id": { "source": "id" },
      "name": { "source": "name" }
    }
  },
  "pagination": {
    "currentPage": {
      "source": "page"
    },
    "totalPages": {
      "source": "totalPages"
    },
    "hasNext": {
      "source": "hasMore",
      "transform": "cast",
      "castTo": "bool"
    },
    "nextPageUrl": {
      "source": "links.next"
    }
  }
}
```

**Result**:
```json
{
  "items": [
    { "id": "item101", "name": "Item 101" },
    { "id": "item102", "name": "Item 102" }
  ],
  "pagination": {
    "currentPage": 2,
    "totalPages": 10,
    "hasNext": true,
    "nextPageUrl": "https://api.service.com/items?page=3"
  }
}
```

### 2. Batch Request Creation
Array mapping can be used to construct a request body for batch processing endpoints by transforming a source array into a structured array of operations.

**Source JSON**:
```json
{
  "items": [
    { "id": "A1", "newStatus": "approved" },
    { "id": "B2", "newStatus": "completed" }
  ],
  "currentUserDetail": {
    "id": "user-123"
  }
}
```

**Mapping Rules (within `RequestConfig`)**:
```json
{
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
          }
        }
      }
    }
  }
}
```

**Result (Request Body)**:
```json
{
  "requests": [
    {
      "requestId": "A1",
      "operation": "UPDATE",
      "data": {
        "status": "approved",
        "updatedBy": "user-123"
      }
    },
    {
      "requestId": "B2",
      "operation": "UPDATE",
      "data": {
        "status": "completed",
        "updatedBy": "user-123"
      }
    }
  ]
}
```

### 3. Complex Array Sources

Combine multiple source arrays into a single destination property using transformations like `concat`.

**Source JSON**:
```json
{
  "users": {
    "active": ["user1", "user2"],
    "pending": ["user3"],
    "inactive": ["user4"]
  }
}
```

**Mapping Rules**:
```json
{
  "allUserIds": {
    "source": ["users.active", "users.pending", "users.inactive"],
    "transform": "concat"
  }
}
```

**Result**:
```json
{
  "allUserIds": ["user1", "user2", "user3", "user4"]
}
```

---

## Error Handling

- **Empty Arrays**:  
  - `List`: Returns `[]`  
  - `First` / `Last`: Returns `null`

- **Missing Array Source**:  
  - All result types return an empty array or `null`.
  - The system logs a warning, but processing continues.

- **Invalid Items**:  
  - Items that aren't objects or don't match the expected structure are skipped. The mapping process will continue with the remaining valid items.

- **Transformation Errors**:
  - If a transformation within an item mapping fails (e.g., a `cast` on an invalid value), it may result in a `null` value for that specific field, but it will not stop the entire array mapping process.

---

## Multiple Arrays in Rules

```json
{
  "multipleItems": [
    {
      "name": { "source": "firstName" },
      "type": "primary"
    },
    {
      "name": { "source": "lastName" },
      "type": "secondary"
    }
  ]
}
```

---

## Best Practices

- **Performance Optimization**: For large arrays where you only need the first item, always use `resultType: "First"` to avoid unnecessary processing. Avoid deep nesting (more than 5 levels) inside an array map if possible.

- **Handle Empty & Null Cases**: When mapping an array, anticipate that the source could be empty or null. Use `resultType: "First"` or `Last` carefully, as they can return `null`, which may need to be handled by downstream logic.

- **Use Descriptive Destination Properties**:

  ```json
  // Good
  "latestOrder": {
    "source": "orders",
    "resultType": "Last",
    "transform": "map",
    "item": { ... }
  }
  ```

- **Isolate Nested Array Logic**: When dealing with nested arrays, map the outer array first, and then map the inner array from the result of the first transformation if necessary.
  ```json
  {
    "orderItems": {
      "source": "orders[0].items",
      "transform": "map",
      "item": { ... }
    }
  }
  ```

- **Combine with Other Transformations**: Apply transformations within the `item` definition to clean or format data efficiently.

  ```json
  {
    "products": {
      "source": "rawProducts",
      "transform": "map",
      "item": {
        "name": {
          "source": "productName",
          "transform": "uppercase"
        }
      }
    }
  }
  ```

- **Testing**:
  - Test with empty source arrays (`[]`) and `null` source arrays.
  - Verify `First`, `Last`, and `List` result types behave as expected.
  - Test scenarios where some items in the array are missing properties.
  - Validate transformations within the array mapping work correctly.

---

## Common Use Cases

- **API Response Processing**: Standardizing and cleaning array data from external APIs.
- **Batch Request Creation**: Preparing payloads for APIs that accept multiple operations in a single call.
- **Data Aggregation**: Extracting and collecting specific fields from a list of objects.
- **Report Generation**: Transforming raw data arrays into a format suitable for reports.
- **UI Data Preparation**: Shaping data to fit the requirements of a user interface component.
- **Handling Paginated Data**: Extracting both the data and the pagination context from an API response.
