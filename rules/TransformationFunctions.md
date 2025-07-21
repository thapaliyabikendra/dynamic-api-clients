# Transformation Functions

## Overview

Transformation functions allow you to modify, format, and process data as it is mapped from the source to the destination. They can be applied to fields, arrays, or entire objects within mapping rules for both requests and responses.

## Supported Transformation Types & Examples

### 1. String Transformations

#### Uppercase
Converts a string to uppercase.
```json
{
  "countryCode": {
    "source": "country",
    "transform": "uppercase"
  }
}
```
*Input:* `{ "country": "np" }` → *Output:* `{ "countryCode": "NP" }`

#### Lowercase
Converts a string to lowercase.
```json
{
  "email": {
    "source": "Email",
    "transform": "lowercase"
  }
}
```
*Input:* `{ "Email": "USER@EXAMPLE.COM" }` → *Output:* `{ "email": "user@example.com" }`

### 2. Concatenation
Joins multiple fields or values into a single string, with an optional separator.
```json
{
  "fullName": {
    "source": ["firstName", "middleName", "lastName"],
    "transform": "concat",
    "separator": " "
  }
}
```
*Input:* `{ "firstName": "Jane", "middleName": "A.", "lastName": "Smith" }` → *Output:* `{ "fullName": "Jane A. Smith" }`

### 3. Type Casting
Casts a value to a specific type.
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
*Input:* `{ "Status": "true", "ResponseCode": "200", "FlagStatus": false }` → *Output:* `{ "isActive": true, "statusCode": 200, "isFlagged": 0 }`

Supported `castTo` types:
- `string` – Converts to string
- `int` or `integer` – Converts to integer
- `bool` or `boolean` – Converts to boolean
- `bit` – Converts boolean to 1/0

### 4. String Replacement

#### Single Replacement
Replaces all occurrences of a substring with another value.
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
*Input:* `{ "phoneNumber": "555-123-4567" }` → *Output:* `{ "formattedPhone": "5551234567" }`

#### Multiple Replacements
Performs multiple replacements in sequence.
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
*Input:* `{ "description": "Hello<br>World &amp; Friends &lt;3" }` → *Output:* `{ "cleanedText": "Hello\nWorld & Friends <3" }`

### 5. Array Transformations

#### Map
Applies a mapping rule to each item in an array.
```json
{
  "items": {
    "source": "orderItems",
    "transform": "map",
    "item": {
      "id": { "source": "sku" },
      "qty": { "source": "quantity" }
    }
  }
}
```
*Input:* `{ "orderItems": [ { "sku": "A1", "quantity": 2 }, { "sku": "B2", "quantity": 1 } ] }` → *Output:* `{ "items": [ { "id": "A1", "qty": 2 }, { "id": "B2", "qty": 1 } ] }`

#### Concat (Arrays)
Concatenates multiple arrays or values into a single array or string.
```json
{
  "combinedData": {
    "source": ["users.active", "users.pending", "users.inactive"],
    "transform": "concat",
    "separator": ","
  }
}
```
*Input:* `{ "users": { "active": ["a"], "pending": ["b"], "inactive": ["c"] } }` → *Output:* `{ "combinedData": ["a", "b", "c"] }`

### 6. Advanced Transformation Chaining
You can chain multiple transformations for complex scenarios by using an array of transformation steps.
```json
{
  "data": {
    "source": "result",
    "transform": [
      { "type": "map", "item": { "id": { "source": "identifier" } } },
      { "type": "filter", "condition": "status === 'active'" },
      { "type": "sort", "field": "createdAt", "order": "desc" }
    ]
  }
}
```
*Input:* `{ "result": [ { "identifier": 1, "status": "active", "createdAt": 2 }, { "identifier": 2, "status": "inactive", "createdAt": 1 } ] }` → *Output:* `{ "data": [ { "id": 1, "status": "active", "createdAt": 2 } ] }`

### 7. Rules Engine Integration
Apply business rules to a field using the rules engine.
```json
{
  "eligibilityStatus": {
    "source": "applicationData",
    "rulesEngine": "LOAN_ELIGIBILITY_RULES"
  }
}
```
*Input:* `{ "applicationData": { ... } }` → *Output:* `{ "eligibilityStatus": "Eligible" }` *(assuming rules engine returns this)*

## Error Handling

- **Invalid Source Path**: If the source path does not exist, the field is set to `null` or omitted.
- **Invalid Cast**: If a value cannot be cast to the requested type, the original value is returned.
- **Replacement Errors**: If `replaceFrom` is not found, the original string is returned unchanged.
- **Array Transformations**: If the source is not an array, returns an empty array or `null` for `First`/`Last` result types.
- **Transformation Chain Failures**: If a step in a chain fails, subsequent steps are skipped for that item.
- **Rules Engine Errors**: If the rules engine fails, the field is set to `null` and an error may be logged.

## Advanced Features

- **Transformation Chaining**: Combine multiple transformations for powerful data processing.
- **Conditional Transformations**: Use the rules engine or custom logic for conditional field values.
- **Array Filtering and Sorting**: Filter and sort arrays as part of transformation chains.
- **Custom Transform Functions**: Extend with custom logic if supported by your platform.

## Best Practices
- Use transformations to standardize and clean data before sending or after receiving.
- Chain transformations for complex data processing.
- Use type casting to ensure data types match API requirements.
- Use the rules engine for conditional or computed fields.
- Document custom transformation logic for maintainability.
- Test transformations with edge cases (empty, null, invalid data).
- Prefer simple, readable transformation chains for maintainability.

## Common Use Cases
- Formatting user input (e.g., emails, phone numbers)
- Aggregating or splitting data fields
- Cleaning up HTML or encoded text
- Converting types for API compatibility
- Mapping and filtering arrays
- Applying business rules to data
