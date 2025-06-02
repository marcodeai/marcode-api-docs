# Marcode Brand Management API

## Overview

The Brand Management API allows you to create, read, update, and delete brands in the system. A brand represents an entity that you want to track and manage, including its keywords, domains, and alerts.

## Authentication

All API endpoints require authentication using Bearer tokens. 

Include the token in the Authorization header:
```
Authorization: Bearer <your_token>
```

## Data Models

### Brand Object

```json
{
  "brandId": "string",
  "brandName": "string",
  "reservedKeywords": [
    {
      "brkId": "string",
      "keyword": "string",
      "keyword_type": "reserved" | "negative"
    }
  ],
  "keywords": [
    {
      "bkrId": "string",
      "keyword": "string",
      "region_id": 1,
      "region_name": "string",
      "device": "string",
      "engine": "string"
    }
  ],
  "domains": [
    {
      "brandAllowedDomainId": "string",
      "domain": "string",
      "advertiserarid": "string",
      "advertiser_name": "string"
    }
  ]
}
```

#### Field Descriptions

- `brandId`: Unique identifier for the brand
- `brandName`: Name of the brand
- `reservedKeywords`: List of reserved keywords for the brand
  - `brkId`: Unique identifier for the reserved keyword
  - `keyword`: The reserved keyword text
  - `keyword_type`: Type of the reserved keyword
    - `reserved`: Matches keyword in title/description (i.e. Trademark - "Jigsaw")
    - `negative`: Excludes matches when found in title/description (i.e. "Jigsaw Puzzle")
- `keywords`: List of keywords and their regions
  - `bkrId`: Unique identifier for the keyword-region association
  - `keyword`: The keyword text
  - `region_id`: ID of the region
  - `region_name`: Name of the region
  - `device`: Name of the device (desktop or mobile)
  - `engine`: Name of the search engine (google, bing, yahoo, aol)
- `domains`: List of domains associated with the brand
  - `brandAllowedDomainId`: Unique identifier for the allowed domain
  - `domain`: Domain name (www. prefix automatically removed)
  - `advertiserarid`: Advertiser ID associated with the domain
  - `advertiser_name`: Name of the advertiser
- `hasEditAccess`: Whether the current user has edit access to the brand

## API Endpoints

### List Available Brands

```http
GET /api/brand
```

Returns all brands available to the authenticated user, along with user information.

**Response Example:**
```json
{
  "me": {
    "user_id": 1,
    "email": "seeduser@marcode.ai",
    "first_name": "Seed",
    "last_name": "User"
  },
  "brands": [
    {
      "brand_id": 1,
      "enabled": true,
      "brand_name": "Seed Brand"
    },
  ]
}
```

### Create a New Brand

```http
POST /api/org/{orgId}/brand
```

Creates a new brand under the specified organization.

**Path Parameters:**
- `orgId`: ID of the organization to create the brand under

**Request Body:**
```json
{
  "brandName": "New Brand Name",         // Required
  "reservedKeywords": [
    {
      "keyword": "Jigsaw",              // Required
      "keyword_type": "reserved"        // Required: "reserved" or "negative"
    },
    {
      "keyword": "Puzzle",        
      "keyword_type": "negative"        
    }
  ],
  "keywords": [
    {
      "keyword": "keyword1",            // Required
      "regionId": 1,                    // Required
      "engine": "google",               // Required: "google", "bing", "yahoo", "aol"
      "device": "mobile"                // Required: "desktop", "mobile"
    }
  ],
  "domains": [
    "example.com"                       // www. prefix will be automatically removed
  ]
}
```

### Get Brand Information

```http
GET /api/brand/{brandId}
```

Retrieves detailed information about a specific brand.

**Path Parameters:**
- `brandId`: ID of the brand to retrieve

### Update Brand Information

```http
PATCH /api/brand/{brandId}
```

Updates the specified brand's information.  For arrays, items will be appended to exiting values.

**Path Parameters:**
- `brandId`: ID of the brand to update

**Request Body:**
```json
{
  "brandName": "Updated Brand Name",
  "reservedKeywords": [                 // Optional
    {
      "keyword": "Jigsaw",              // Required
      "keyword_type": "reserved"        // Required: "reserved" or "negative"
    }
  ],
  "keywords": [                         // Optional
    {
      "keyword": "keyword1",            // Required
      "regionId": 1,                    // Required
      "engine": "google",               // Required: "google", "bing", "yahoo", "aol"
      "device": "mobile"                // Required: "desktop", "mobile"
    }
  ],
  "domains": [                          // Optional
    "example.com"                       // Required: www. prefix will be automatically removed
  ]
}
```

### Remove Domain from Brand

```http
DELETE /api/brand/{brandId}/domain/{brandAllowedDomainId}
```

Removes a specific domain from a brand.

**Path Parameters:**
- `brandId`: ID of the brand
- `brandAllowedDomainId`: ID of the domain to remove

### Remove Keyword from Brand

```http
DELETE /api/brand/{brandId}/keyword/{bkrId}
```

Removes a specific keyword from a brand.

**Path Parameters:**
- `brandId`: ID of the brand
- `bkrId`: ID of the keyword-region association to remove

### Remove Reserved Keyword from Brand

```http
DELETE /api/brand/{brandId}/reservedkeyword/{brkId}
```

Removes a specific reserved keyword from a brand.

**Path Parameters:**
- `brandId`: ID of the brand
- `brkId`: ID of the reserved keyword to remove

### Enable / Disable Brand

```http
PUT /api/brand/{brandId}/enabled
```

Sets the brand enabled status.

**Path Parameters:**
- `brandId`: ID of the brand

**Request Body:**
```json
{
  "enabled": true                // Required: true, false
}
```

### Assign Brand to a Package

```http
PUT /api/brand/{brandId}/package
```

Assigns the Brand to a package.

**Path Parameters:**
- `brandId`: ID of the brand

**Request Body:**
```json
{
  "packageId": 1                // Required
}
```

## Error Responses

All endpoints can return the following error responses:

- `400 Bad Request`: Invalid input parameters
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

1. **Reserved Keywords**:
   - Use `reserved` type for trademarks or brand terms you want to match
   - Use `negative` type for terms that should exclude matches when found
   - Both types match against title and description fields

2. **Keywords**:
   - Each keyword must be associated with a region
   - Region IDs are predefined in the system

3. **Domains**:
   - The `www.` prefix is automatically removed from domains
   - Domains are associated with advertisers in the system 