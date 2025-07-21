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

