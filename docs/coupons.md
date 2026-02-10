# Marcode Coupon Management API

## Overview

The Coupon Management API provides three capabilities for managing and querying coupon codes associated with a brand:

- **Coupon Patterns** - Rules for grouping and filtering coupon codes by pattern (prefix, suffix, contains, exact match, or regex). Used for categorisation and reporting.
- **Monitored Coupons** - Explicit coupon codes to actively search for and receive alerts when usage is detected. Always matched exactly.
- **Coupon Data** - Retrieve and filter observed coupon data using pattern-based or text-based filters.

## Authentication

All API endpoints require authentication using Bearer tokens.

Include the token in the Authorization header:
```
Authorization: Bearer <your_token>
```

All endpoints also require `edit` access to the brand.

## Data Models

### Coupon Pattern Object

```json
{
  "coupon_brand_pattern_id": "uuid",
  "brand_id": 123,
  "pattern_type": "prefix" | "suffix" | "contains" | "exact" | "regex",
  "pattern": "string",
  "name": "string"
}
```

#### Field Descriptions

- `coupon_brand_pattern_id`: Unique identifier (UUID) for the pattern
- `brand_id`: ID of the brand this pattern belongs to
- `pattern_type`: How the pattern should be matched against coupon codes
  - `prefix`: Matches coupons starting with the pattern
  - `suffix`: Matches coupons ending with the pattern
  - `contains`: Matches coupons containing the pattern
  - `exact`: Matches the coupon code exactly
  - `regex`: Matches coupons against a regular expression
- `pattern`: The pattern string to match against
- `name`: A human-readable label for the pattern

### Monitored Coupon Object

```json
{
  "coupon_brand_pattern_id": "uuid",
  "brand_id": 123,
  "coupon_code": "string"
}
```

#### Field Descriptions

- `coupon_brand_pattern_id`: Unique identifier (UUID) for the monitored coupon
- `brand_id`: ID of the brand this monitored coupon belongs to
- `coupon_code`: The exact coupon code to search for

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

### Coupon Patterns

#### List Coupon Patterns

```http
GET /api/brand/{brandId}/coupons/patterns
```

Returns all coupon patterns (non-monitored) for the specified brand.

**Path Parameters:**
- `brandId`: ID of the brand

**Response Example:**
```json
[
  {
    "coupon_brand_pattern_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "brand_id": 1,
    "pattern_type": "prefix",
    "pattern": "BRAND",
    "name": "Brand prefix codes"
  }
]
```

#### Sync Coupon Patterns (Full Replace)

```http
POST /api/brand/{brandId}/coupons/patterns
```

Replaces the full set of coupon patterns for the brand. Patterns included in the request with a `coupon_brand_pattern_id` are updated; new patterns (without an ID) are created; existing patterns not included in the request are deleted.

**Path Parameters:**
- `brandId`: ID of the brand

**Request Body:**
```json
{
  "couponPatterns": [
    {
      "coupon_brand_pattern_id": "a1b2c3d4-...",  // Optional: include to update existing
      "pattern_type": "prefix",                    // Required: "prefix", "suffix", "contains", "exact", "regex"
      "pattern": "BRAND",                          // Required: the pattern string
      "name": "Brand prefix codes"                 // Optional: human-readable label
    }
  ]
}
```

#### Add Coupon Patterns

```http
PATCH /api/brand/{brandId}/coupons/patterns
```

Adds one or more new coupon patterns to the brand without affecting existing patterns.

**Path Parameters:**
- `brandId`: ID of the brand

**Request Body:**
```json
{
  "couponPatterns": [
    {
      "pattern_type": "contains",       // Required: "prefix", "suffix", "contains", "exact", "regex"
      "pattern": "SUMMER",              // Required: the pattern string
      "name": "Summer campaign codes"   // Optional: human-readable label
    },
    {
      "pattern_type": "prefix",
      "pattern": "VIP",
      "name": "VIP codes"
    }
  ]
}
```

**Response:**
```json
{
  "success": true
}
```

#### Delete a Single Coupon Pattern

```http
DELETE /api/brand/{brandId}/coupons/patterns/{couponBrandPatternId}
```

Removes a single coupon pattern by its ID.

**Path Parameters:**
- `brandId`: ID of the brand
- `couponBrandPatternId`: UUID of the pattern to delete

**Response:**
```json
{
  "success": true,
  "deleted": 1
}
```

#### Delete Multiple Coupon Patterns

```http
DELETE /api/brand/{brandId}/coupons/patterns
```

Removes multiple coupon patterns by their IDs.

**Path Parameters:**
- `brandId`: ID of the brand

**Request Body:**
```json
{
  "couponBrandPatternIds": [
    "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "b2c3d4e5-f6a7-8901-bcde-f12345678901"
  ]
}
```

**Response:**
```json
{
  "success": true,
  "deleted": 2
}
```

#### Suggest Coupon Patterns

```http
GET /api/brand/{brandId}/coupons/patterns/suggest
```

Analyses the brand's observed coupon codes and suggests new patterns using AI. Takes into account existing patterns to avoid duplicates.

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

---

### Monitored Coupon Codes

#### List Monitored Coupon Codes

```http
GET /api/brand/{brandId}/coupons/monitored
```

Returns all monitored coupons for the specified brand.

**Path Parameters:**
- `brandId`: ID of the brand

**Response Example:**
```json
[
  {
    "coupon_brand_pattern_id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
    "brand_id": 1,
    "coupon_code": "EXCLUSIVE20"
  }
]
```

#### Add Monitored Coupons

```http
PATCH /api/brand/{brandId}/coupons/monitored
```

Adds one or more coupon codes to be monitored. Monitored coupons are always matched exactly and are case insensitive.

**Path Parameters:**
- `brandId`: ID of the brand

**Request Body:**
```json
{
  "monitoredCoupons": [
    {
      "coupon_code": "EXCLUSIVE20"           // Required: the exact coupon code to monitor
    },
    {
      "coupon_code": "VIP50"
    }
  ]
}
```

**Response:**
```json
{
  "success": true
}
```

#### Delete a Single Monitored Coupon

```http
DELETE /api/brand/{brandId}/coupons/monitored/{couponBrandPatternId}
```

Removes a single monitored coupon by its ID.

**Path Parameters:**
- `brandId`: ID of the brand
- `couponBrandPatternId`: UUID of the monitored coupon to delete

**Response:**
```json
{
  "success": true,
  "deleted": 1
}
```

#### Delete Multiple Monitored Coupons

```http
DELETE /api/brand/{brandId}/coupons/monitored
```

Removes multiple monitored coupons by their IDs.

**Path Parameters:**
- `brandId`: ID of the brand

**Request Body:**
```json
{
  "couponBrandPatternIds": [
    "c3d4e5f6-a7b8-9012-cdef-123456789012",
    "d4e5f6a7-b8c9-0123-defa-234567890123"
  ]
}
```

**Response:**
```json
{
  "success": true,
  "deleted": 2
}
```

---

### Coupon Data Retrieval

#### Get Coupon List with Filtering

```http
GET /api/data/brandId/{brandId}/analysisCode/couponsList?filter={base64EncodedFilter}&export={format}
```

Retrieves coupon data for a brand with optional filtering.

**Path Parameters:**
- `brandId`: ID of the brand

**Query Parameters:**
- `filter`: Base64-encoded JSON filter object (optional)
- `export`: Export format (optional). One of: `"csv"`, `"json"`, `"parquet"`

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

#### Example 2: Filter by Multiple Patterns
```json
{
  "field": "coupon_pattern_id",
  "predicate": "IN",
  "values": [
    "550e8400-e29b-41d4-a716-446655440001",
    "550e8400-e29b-41d4-a716-446655440002"
  ]
}
```

**Result:** Returns coupons that match either the first OR second pattern.

#### Example 3: Filter by Coupon Code Text
```json
{
  "field": "coupon_code",
  "predicate": "CONTAINS",
  "value": "DISCOUNT"
}
```

**Result:** Returns coupons with "DISCOUNT" anywhere in the code.

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

**Result:** Coupons matching the pattern AND containing "DISCOUNT".

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

**Result:** Returns coupons that do NOT match the specified pattern.

### Base64 Encoding

Filters must be base64-encoded when passed as query parameters:

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

## Notes

1. **Patterns vs Monitored Coupons**:
   - **Patterns** are used for grouping and filtering coupon codes in reports. They support flexible matching (prefix, suffix, contains, exact, regex).
   - **Monitored coupons** are used for explicit searching and alerting. They are always matched exactly and are case insensitive. Use these when you want to be notified when a specific coupon code is found in the wild.

2. **POST vs PATCH for Patterns**:
   - `POST` performs a full sync: it updates existing patterns, creates new ones, and deletes any patterns not included in the request. Use this when you want to replace the entire pattern set.
   - `PATCH` is additive only: it creates new patterns without modifying or removing existing ones. Use this when you want to append patterns to the existing list.

3. **Bulk Deletes**:
   - Both patterns and monitored coupons support single delete (via URL parameter) and bulk delete (via request body array). The bulk delete endpoint returns the count of records actually deleted.

4. **Pattern-Based Filtering**:
   - When filtering coupon data by `coupon_pattern_id`, the system resolves the pattern and applies the appropriate matching logic (prefix, suffix, etc.) to return matching coupon codes. This allows you to define patterns once and reuse them for filtering across data queries.
