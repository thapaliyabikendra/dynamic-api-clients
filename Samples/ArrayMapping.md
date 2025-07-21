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

## Error Handling

- **Empty Arrays**:  
  - `List`: Returns `[]`  
  - `First` / `Last`: Returns `null`

- **Missing Array Source**:  
  All result types return an empty array or `null`.

- **Invalid Items**:  
  Items that aren't objects are skipped.

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

- Use **Descriptive Result Types**:
```json
{
  "latestOrder": {
    "source": "orders",
    "resultType": "Last",
    "transform": "map",
    "item": { ... }
  }
}
```

- Handle **Empty Arrays**:
```json
{
  "firstProduct": {
    "source": "products",
    "resultType": "First",
    "transform": "map",
    "item": { ... }
  }
}
```

- **Nested Arrays**:
```json
{
  "orderItems": {
    "source": "orders[0].items",
    "transform": "map",
    "resultType": "List",
    "item": { ... }
  }
}
```

- Apply **Complex Transformations**:
```json
{
  "products": {
    "source": "rawProducts",
    "transform": "map",
    "resultType": "List",
    "item": {
      "name": {
        "source": "productName",
        "transform": "uppercase"
      }
    }
  }
}
```

---

## Common Use Cases

- API Response Processing  
- Data Aggregation  
- Report Generation  
- UI Data Preparation  
- Batch Processing  
