# Data Transformations

## Overview

Data transformations allow you to modify values during the mapping process. You can change text case, concatenate arrays, cast data types, and perform string replacements.

## Basic Transformation Syntax

```json
{
  "destinationField": {
    "source": "sourceField",
    "transform": "transformationType",
    // Additional transformation parameters
  }
}
```

## Available Transformations

### 1. Copy (Default)

Simply copies the value without modification:

```json
{
  "copiedField": {
    "source": "originalField",
    "transform": "copy"
  }
}
```

### 2. Uppercase

Converts string values to uppercase:

```json
{
  "upperName": {
    "source": "name",
    "transform": "uppercase"
  }
}
```

**Example:**
- Input: `"john doe"`
- Output: `"JOHN DOE"`

### 3. Lowercase

Converts string values to lowercase:

```json
{
  "lowerEmail": {
    "source": "email",
    "transform": "lowercase" 
  }
}
```

**Example:**
- Input: `"JOHN.DOE@EXAMPLE.COM"`
- Output: `"john.doe@example.com"`

### 4. Concatenation

Joins array elements into a single string:

```json
{
  "fullAddress": {
    "source": ["street", "city", "country"],
    "transform": "concat",
    "separator": ", "
  }
}
```

**Example:**
- Input: `["123 Main St", "New York", "USA"]`
- Output: `"123 Main St, New York, USA"`

**Default separator**: Single space `" "`

### 5. Type Casting

Converts values between different data types:

```json
{
  "convertedValue": {
    "source": "originalValue",
    "transform": "cast",
    "castTo": "targetType"
  }
}
```

#### Cast Types:

##### To Bit (Integer 0/1)
```json
{
  "isActiveFlag": {
    "source": "isActive", 
    "transform": "cast",
    "castTo": "bit"
  }
}
```
- `true` → `1`
- `false` → `0`

##### To Boolean
```json
{
  "boolValue": {
    "source": "stringFlag",
    "transform": "cast", 
    "castTo": "bool"
  }
}
```
- `"true"` → `true`
- `"false"` → `false`
- `1` → `true`
- `0` → `false`

##### To String
```json
{
  "stringValue": {
    "source": "numericValue",
    "transform": "cast",
    "castTo": "string"
  }
}
```
- `42` → `"42"`
- `true` → `"true"`

##### To Integer
```json
{
  "intValue": {
    "source": "stringNumber", 
    "transform": "cast",
    "castTo": "integer"
  }
}
```
- `"42"` → `42`
- `true` → `1`
- `false` → `0`

### 6. String Replacement

#### Single Replacement
Replace one substring with another:

```json
{
  "cleanedText": {
    "source": "originalText",
    "transform": "replace",
    "replaceFrom": "old_value",
    "replaceTo": "new_value"
  }
}
```

#### Multiple Replacements
Apply multiple replacements in sequence:

```json
{
  "normalizedText": {
    "source": "originalText",
    "transform": "replace", 
    "replaceList": [
      { "from": "\\n", "to": " " },
      { "from": "\\t", "to": " " },
      { "from": "  ", "to": " " }
    ]
  }
}
```

## Detailed Examples

### Example 1: User Profile Transformation

**Source JSON:**
```json
{
  "firstName": "john",
  "lastName": "doe", 
  "email": "JOHN.DOE@EXAMPLE.COM",
  "isActive": true,
  "roles": ["admin", "user", "moderator"]
}
```

**Mapping Rules:**
```json
{
  "profile": {
    "fullName": {
      "source": ["firstName", "lastName"],
      "transform": "concat",
      "separator": " "
    },
    "displayName": {
      "source": "firstName", 
      "transform": "uppercase"
    },
    "email": {
      "source": "email",
      "transform": "lowercase"
    },
    "statusFlag": {
      "source": "isActive",
      "transform": "cast", 
      "castTo": "bit"
    },
    "roleList": {
      "source": "roles",
      "transform": "concat",
      "separator": ", "
    }
  }
}
```

**Result:**
```json
{
  "profile": {
    "fullName": "john doe",
    "displayName": "JOHN",
    "email": "john.doe@example.com", 
    "statusFlag": 1,
    "roleList": "admin, user, moderator"
  }
}
```

### Example 2: Text Cleaning and Normalization

**Source JSON:**
```json
{
  "description": "This\tis\na\n\nmulti-line\ttext\nwith\ttabs",
  "status": "ACTIVE_USER",
  "score": "85.5"
}
```

**Mapping Rules:**
```json
{
  "cleanDescription": {
    "source": "description",
    "transform": "replace",
    "replaceList": [
      { "from": "\\n", "to": " " },
      { "from": "\\t", "to": " " }, 
      { "from": "  ", "to": " " }
    ]
  },
  "normalizedStatus": {
    "source": "status",
    "transform": "replace",
    "replaceFrom": "_",
    "replaceTo": " "
  },
  "numericScore": {
    "source": "score",
    "transform": "cast",
    "castTo": "integer"
  }
}
```

**Result:**
```json
{
  "cleanDescription": "This is a multi-line text with tabs",
  "normalizedStatus": "ACTIVE USER", 
  "numericScore": 85
}
```

### Example 3: API Response Standardization

**Source JSON:**
```json
{
  "user_name": "JohnDoe",
  "user_email": "john@EXAMPLE.com",
  "is_verified": "true",
  "account_type": "premium_user",
  "tags": ["VIP", "GOLD", "SUBSCRIBER"]
}
```

**Mapping Rules:**
```json
{
  "user": {
    "username": {
      "source": "user_name", 
      "transform": "lowercase"
    },
    "email": {
      "source": "user_email",
      "transform": "lowercase"
    },
    "verified": {
      "source": "is_verified",
      "transform": "cast",
      "castTo": "bool"
    },
    "accountType": {
      "source": "account_type",
      "transform": "replace", 
      "replaceFrom": "_",
      "replaceTo": " "
    },
    "tagString": {
      "source": "tags",
      "transform": "concat", 
      "separator": " | "
    }
  }
}
```

**Result:**
```json
{
  "user": {
    "username": "johndoe",
    "email": "john@example.com",
    "verified": true,
    "accountType": "premium user",
    "tagString": "VIP | GOLD | SUBSCRIBER"
  }
}
```

## Transformation Chaining

Transformations are applied in the order specified. Currently, only one transformation per field is supported, but you can achieve complex transformations through multiple mapping steps.

## Error Handling

### Invalid Cast Operations
- If casting fails, the original value is returned
- Non-numeric strings to integer return original string
- Invalid boolean strings return original string

### Missing Values
- Transformations on `null` or missing values return `null`
- Empty arrays in concat return empty string

### Type Mismatches
- Applying string transformations to non-strings may produce unexpected results
- Always ensure source data types match expected transformation input

## Best Practices

1. **Choose Appropriate Transformations**:
   ```json
   // Good: Use lowercase for email normalization
   {
     "email": { "source": "userEmail", "transform": "lowercase" }
   }
   
   // Good: Use bit casting for database flags
   {
     "isActive": { "source": "active", "transform": "cast", "castTo": "bit" }
   }
   ```

2. **Handle Arrays for Concatenation**:
   ```json
   // Ensure source is array for concat
   {
     "fullName": {
       "source": ["firstName", "lastName"], 
       "transform": "concat",
       "separator": " "
     }
   }
   ```

3. **Validate Cast Operations**:
   ```json
   // Be careful with casting - ensure source data is compatible
   {
     "age": { "source": "ageString", "transform": "cast", "castTo": "integer" }
   }
   ```

4. **Use Meaningful Separators**:
   ```json
   // Choose separators that make sense for your data
   {
     "address": { 
       "source": ["street", "city"], 
       "transform": "concat", 
       "separator": ", " 
     }
   }
   ```

## Common Use Cases

- **Data Standardization**: Normalize text case, formats
- **Type Conversion**: Convert strings to numbers, booleans to bits
- **Text Processing**: Clean up whitespace, remove unwanted characters
- **Display Formatting**: Concatenate fields for user-friendly display
- **API Integration**: Transform data formats between systems