# Marcode Coupon Patterns & Data API

## Overview

The Coupon Patterns & Data API allows you to manage coupon patterns for intelligent matching and retrieve filtered coupon data. This system enables pattern-based filtering where you can define patterns (like "SAVE*", "*10", etc.) and filter coupon data based on these patterns rather than exact matches.

## Authentication

All API endpoints require authentication using Bearer tokens. 

Include the token in the Authorization header:
```
Authorization: Bearer <your_token>
```

## Data Models

### Coupon Pattern Object

```json
{
  "coupon_brand_pattern_id": "string",
  "brand_id": 1,
  "pattern_type": "prefix" | "suffix" | "contains" | "exact" | "regex",
  "pattern": "string",
  "name": "string"
}
```

#### Field Descriptions

- `coupon_brand_pattern_id`: Unique identifier for the coupon pattern (UUID)
- `brand_id`: ID of the brand this pattern belongs to
- `pattern_type`: Type of pattern matching
  - `prefix`: Matches coupon codes that start with the pattern
  - `suffix`: Matches coupon codes that end with the pattern
  - `contains`: Matches coupon codes that contain the pattern anywhere
  - `exact`: Matches coupon codes that exactly equal the pattern
  - `regex`: Matches coupon codes using regular expressions
- `pattern`: The pattern string to match against
- `name`: Human-readable name for the pattern

### Coupon Data Object

```json
{
  "coupon_id": "string",
  "coupon_code": "string",
  "coupon_title": "string",
  "coupon_description": "string",
  "coupon_first_seen": "2024-01-01T00:00:00Z",
  "coupon_last_seen": "2024-01-01T00:00:00Z",
  "coupon_expires": "2024-12-31",
  "domain": "string",
  "coupon_site_name": "string",
  "coupon_site_url": "string",
  "coupon_site_extension_id": "string",
  "coupon_site_metadata": "object",
  "coupon_site_code": "string"
}
```

### Filter Object Structure

Filters use a hierarchical structure with operators and operands:

```json
{
  "operator": "AND" | "OR",
  "operands": [
    {
      "field": "string",
      "predicate": "string",
      "value": "string",           // For single values
      "values": ["string"],        // For multiple values
      "value1": "string",          // For BETWEEN predicates
      "value2": "string"           // For BETWEEN predicates
    }
  ]
}
```

## API Endpoints

### Pattern Management

#### Get Coupon Patterns

```http
GET /api/brand/{brandId}/coupons/patterns
```

Retrieves all coupon patterns for a specific brand.

**Path Parameters:**
- `brandId`: ID of the brand

**Response Example:**
```json
[
  {
    "coupon_brand_pattern_id": "550e8400-e29b-41d4-a716-446655440001",
    "brand_id": 1,
    "pattern_type": "prefix",
    "pattern": "SAVE",
    "name": "Save Prefix Pattern"
  },
  {
    "coupon_brand_pattern_id": "550e8400-e29b-41d4-a716-446655440002",
    "brand_id": 1,
    "pattern_type": "suffix", 
    "pattern": "10",
    "name": "Ten Suffix Pattern"
  }
]
```

#### Save/Update Coupon Patterns

```http
POST /api/brand/{brandId}/coupons/patterns
```

Creates, updates, or deletes coupon patterns for a brand. This endpoint performs a complete replacement of the brand's patterns.

**Path Parameters:**
- `brandId`: ID of the brand

**Request Body:**
```json
{
  "couponPatterns": [
    {
      "coupon_brand_pattern_id": "550e8400-e29b-41d4-a716-446655440001", // Optional: Include for updates
      "pattern_type": "prefix",        // Required
      "pattern": "SAVE",              // Required
      "name": "Save Prefix Pattern"   // Optional
    },
    {
      // New pattern (no ID)
      "pattern_type": "contains",     // Required
      "pattern": "OFF",               // Required
      "name": "Off Contains Pattern"  // Optional
    }
  ]
}
```

**Behavior:**
- **Create**: Patterns without `coupon_brand_pattern_id` are created as new patterns
- **Update**: Patterns with existing `coupon_brand_pattern_id` are updated
- **Delete**: Any existing patterns not included in the request are deleted

**Response:**
```json
{
  "success": true
}
```

#### AI-Powered Pattern Suggestions

```http
GET /api/brand/{brandId}/coupons/patterns/suggest
```

Uses AI to analyze existing coupon codes and suggest patterns.

**Path Parameters:**
- `brandId`: ID of the brand

**Response Example:**
```json
[
  {
    "pattern": "SAVE",
    "type": "prefix",
    "name": "Save Codes",
    "confidence": 0.85,
    "examples": ["SAVE10", "SAVE20", "SAVE50"]
  },
  {
    "pattern": "OFF",
    "type": "contains",
    "name": "Off Codes",
    "confidence": 0.72,
    "examples": ["10OFF", "TAKEOFF", "OFFNOW"]
  }
]
```

### Data Retrieval

#### Get Coupon List with Filtering

```http
GET /api/data/brandId/{brandId}/analysisCode/couponsList?filter={base64EncodedFilter}&export={format}
```

Retrieves coupon data for a brand with optional filtering.

**Path Parameters:**
- `brandId`: ID of the brand

**Query Parameters:**
- `filter`: Base64-encoded JSON filter object (optional)
- `export`: Export the data to the provided format (optional, default false, options "csv", "json", "parquet")

**Response Example:**
```json
{
  "couponsList": [
    {
      "coupon_id": "12345",
      "coupon_code": "SAVE20",
      "coupon_title": "Save 20% on Everything",
      "coupon_description": "Get 20% off your entire order",
      "domain": "example.com",
      "coupon_site_name": "Example Deals",
      "coupon_site_url": "https://deals.example.com"
    }
  ]
}
```

## Filter Construction

### Available Filter Fields

#### Coupon-Specific Fields
- **`coupon_pattern_id`**: Filter by coupon pattern ID (uses intelligent pattern matching)
- **`coupon_code`**: Filter by exact coupon code text
- **`coupon_site_id`**: Filter by coupon site ID

### Available Predicates

#### For String Fields
- **`EQUALS`**: Exact match
- **`NOT_EQUALS`**: Not equal to
- **`CONTAINS`**: Contains substring (case-insensitive)
- **`NOT_CONTAINS`**: Does not contain substring
- **`STARTS_WITH`**: Starts with substring
- **`ENDS_WITH`**: Ends with substring

#### For Multiple Values
- **`IN`**: Matches any of the provided values
- **`NOT_IN`**: Does not match any of the provided values

#### For Dates/Numbers
- **`GREATER_THAN`**: Greater than value
- **`LESS_THAN`**: Less than value
- **`GREATER_THAN_OR_EQUAL`**: Greater than or equal to value
- **`LESS_THAN_OR_EQUAL`**: Less than or equal to value
- **`BETWEEN`**: Between two values (inclusive)

#### For Booleans
- **`TRUE`**: Is true
- **`FALSE`**: Is false

### Filter Examples

#### Example 1: Filter by Single Coupon Pattern
```json
{
  "field": "coupon_pattern_id",
  "predicate": "EQUALS",
  "value": "550e8400-e29b-41d4-a716-446655440001"
}
```

**What happens:**
1. System looks up pattern ID `550e8400-e29b-41d4-a716-446655440001`
2. Finds pattern: `SAVE` with type `prefix`
3. Returns all coupons starting with "SAVE"

**Base64 encoded:**
```
eyJmaWVsZCI6ImNvdXBvbl9wYXR0ZXJuX2lkIiwicHJlZGljYXRlIjoiRVFVQUxTIiwidmFsdWUiOiI1NTBlODQwMC1lMjliLTQxZDQtYTcxNi00NDY2NTU0NDAwMDEifQ==
```

#### Example 2: Filter by Multiple Patterns
```json
{
  "field": "coupon_pattern_id",
  "predicate": "IN",
  "values": [
    "550e8400-e29b-41d4-a716-446655440001", // SAVE prefix
    "550e8400-e29b-41d4-a716-446655440002"  // 10 suffix
  ]
}
```

**Result:** Returns coupons that either start with "SAVE" OR end with "10"

#### Example 3: Filter by Coupon Code Text
```json
{
  "field": "coupon_code",
  "predicate": "CONTAINS",
  "value": "DISCOUNT"
}
```

**Result:** Returns coupons with "DISCOUNT" anywhere in the code

#### Example 4: Complex Filter with Multiple Conditions
```json
{
  "operator": "AND",
  "operands": [
    {
      "field": "coupon_pattern_id",
      "predicate": "EQUALS",
      "value": "550e8400-e29b-41d4-a716-446655440001"
    },
    {
        "field": "coupon_code",
        "predicate": "CONTAINS",
        "value": "DISCOUNT"
    }
  ]
}
```

**Result:** Coupons matching the "SAVE" pattern AND contain the string "DISCOUNT"

#### Example 5: Exclude Certain Patterns
```json
{
  "field": "coupon_pattern_id",
  "predicate": "NOT_IN",
  "values": [
    "550e8400-e29b-41d4-a716-446655440001"
  ]
}
```

**Result:** Returns coupons that do NOT match the "SAVE" prefix pattern

### Base64 Encoding

Filters must be base64-encoded when passed as URL parameters. Here's how to encode a filter:

**JavaScript:**
```javascript
const filter = {
  "field": "coupon_pattern_id",
  "predicate": "EQUALS",
  "value": "550e8400-e29b-41d4-a716-446655440001"
};

const base64Filter = btoa(JSON.stringify(filter));
// Use in URL: ?filter=eyJmaWVsZCI6ImNvdXBvbl9wYXR0ZXJuX2lkIiw...
```

## Pattern Types in Detail

### 1. Prefix Patterns (`prefix`)
Matches coupon codes that start with the pattern.
- **Pattern**: `SAVE`
- **Matches**: `SAVE10`, `SAVE20`, `SAVEBIG`
- **Use Case**: Brand-specific prefixes, promotional prefixes

### 2. Suffix Patterns (`suffix`)
Matches coupon codes that end with the pattern.
- **Pattern**: `10`
- **Matches**: `SAVE10`, `GET10`, `DISCOUNT10`
- **Use Case**: Discount amounts, percentage indicators

### 3. Contains Patterns (`contains`)
Matches coupon codes that contain the pattern anywhere.
- **Pattern**: `OFF`
- **Matches**: `10OFF`, `OFFNOW`, `TAKEOFF`
- **Use Case**: Common discount words, brand terms

### 4. Exact Patterns (`exact`)
Matches coupon codes that exactly equal the pattern.
- **Pattern**: `FREESHIP`
- **Matches**: `FREESHIP` (only)
- **Use Case**: Specific promotional codes, standard offers

### 5. Regex Patterns (`regex`)
Matches coupon codes using regular expressions.
- **Pattern**: `^SAVE[0-9]+$`
- **Matches**: `SAVE10`, `SAVE25`, `SAVE100`
- **Use Case**: Complex patterns, variable components

## Error Responses

All endpoints can return the following error responses:

- `400 Bad Request`: Invalid input parameters or malformed filter
- `401 Unauthorized`: Invalid or missing authentication token
- `403 Forbidden`: User does not have required permissions
- `404 Not Found`: Requested resource does not exist

Error responses include a description of what went wrong:
```json
{
  "error": "Description of what went wrong"
}
```