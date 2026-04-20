# [Product Name] - API Documentation

## Document Information
- **Version**: 1.0
- **Last Updated**: [Date]
- **Base URL**: [e.g., https://api.example.com/v1]

## 1. Global Specifications

### 1.1 Authentication
[How to authenticate - API keys, JWT, OAuth, etc.]

**Example:**
```
Authorization: Bearer <token>
```

### 1.2 Request Format
- **Content-Type**: application/json
- **Character Encoding**: UTF-8

### 1.3 Response Format

All API responses follow a unified `code` / `message` / `data` structure:

**Success Response:**
```json
{
  "code": 200,
  "message": "success",
  "data": { ... }
}
```

**Error Response:**
```json
{
  "code": 400,
  "message": "Human-readable error description",
  "data": null
}
```

| Field | Type | Description |
|-------|------|-------------|
| `code` | `number` | Business status code. `200` means success; other values indicate specific errors (e.g., `400`, `401`, `404`, `500`) |
| `message` | `string` | Human-readable result description |
| `data` | `any` \| `null` | Response payload on success; `null` on error |

### 1.4 HTTP Status Codes
- `200 OK` - Success
- `201 Created` - Resource created
- `400 Bad Request` - Invalid input
- `401 Unauthorized` - Authentication required
- `403 Forbidden` - Insufficient permissions
- `404 Not Found` - Resource doesn't exist
- `500 Internal Server Error` - Server error

### 1.5 Rate Limiting
[Limits and headers]

### 1.6 Pagination
[How paginated responses work]

## 2. API Endpoints

[Group by module/resource]

### 2.1 [Module Name] (e.g., User Management)

#### Endpoint: [Action Description]

**Request:**
```
[METHOD] /path/to/endpoint
```

**Headers:**
```
Authorization: Bearer <token>
Content-Type: application/json
```

**Path Parameters:**
- `param1` (type, required/optional) - Description

**Query Parameters:**
- `param1` (type, required/optional) - Description

**Request Body:**
```json
{
  "field1": "value",
  "field2": 123
}
```

**Response (200 OK):**
```json
{
  "code": 200,
  "message": "success",
  "data": {
    "field1": "value",
    "field2": 123
  }
}
```

**Error Responses:**
- `400` - [When this happens]
- `401` - [When this happens]

**Example:**
```bash
curl -X POST https://api.example.com/v1/endpoint \
  -H "Authorization: Bearer token123" \
  -H "Content-Type: application/json" \
  -d '{"field1": "value"}'
```

[Repeat for each endpoint]

## 3. Data Models

[Define common data structures]

### Model: [Name]
```json
{
  "id": "string (UUID)",
  "name": "string",
  "created_at": "string (ISO 8601 datetime)",
  "updated_at": "string (ISO 8601 datetime)"
}
```

## 4. Webhooks (if applicable)

[How webhooks work, what events trigger them]

## 5. SDK/Client Libraries (if applicable)

[Links or examples for different languages]
