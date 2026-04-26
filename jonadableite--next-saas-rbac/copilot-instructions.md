## next-saas-rbac

> This document serves as the central repository for documentation on all API endpoints within the SaaS RBAC application. It provides details on endpoint paths, HTTP methods, purpose, request/response formats, and authentication requirements.

## Purpose & Scope

This document serves as the central repository for documentation on all API endpoints within the SaaS RBAC application. It provides details on endpoint paths, HTTP methods, purpose, request/response formats, and authentication requirements.

**MUST** be consulted and updated whenever API endpoints are created, modified, or deleted, as mandated by `@memory.mdc`.

---

## Authentication

All protected endpoints require authentication via Bearer token in the Authorization header:

```
Authorization: Bearer <token>
```

---

## API Endpoints

### Authentication Endpoints

#### POST /api/auth/register
**Purpose**: Register a new user account

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "securePassword123",
  "name": "User Name"
}
```

**Response** (201 Created):
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "User Name",
    "role": "USER"
  },
  "token": "jwt_token"
}
```

**Errors**:
- 400: Validation error
- 409: Email already exists

---

#### POST /api/auth/login
**Purpose**: Authenticate user and return access token

**Request Body**:
```json
{
  "email": "user@example.com",
  "password": "securePassword123"
}
```

**Response** (200 OK):
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "name": "User Name",
    "role": "USER"
  },
  "token": "jwt_token"
}
```

**Errors**:
- 401: Invalid credentials
- 400: Validation error

---

#### POST /api/auth/logout
**Purpose**: Invalidate current session

**Headers**: `Authorization: Bearer <token>`

**Response** (200 OK):
```json
{
  "message": "Logged out successfully"
}
```

---

#### GET /api/auth/me
**Purpose**: Get current authenticated user information

**Headers**: `Authorization: Bearer <token>`

**Response** (200 OK):
```json
{
  "id": "uuid",
  "email": "user@example.com",
  "name": "User Name",
  "role": "USER",
  "createdAt": "2024-01-01T00:00:00Z"
}
```

**Errors**:
- 401: Unauthorized

---

### User Management Endpoints

#### GET /api/users
**Purpose**: List all users (Admin only)

**Headers**: `Authorization: Bearer <token>`

**Query Parameters**:
- `page` (optional): Page number (default: 1)
- `limit` (optional): Items per page (default: 10)
- `role` (optional): Filter by role
- `search` (optional): Search by name or email

**Response** (200 OK):
```json
{
  "users": [
    {
      "id": "uuid",
      "email": "user@example.com",
      "name": "User Name",
      "role": "USER",
      "createdAt": "2024-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100,
    "totalPages": 10
  }
}
```

**Errors**:
- 401: Unauthorized
- 403: Forbidden (not admin)

---

#### GET /api/users/:id
**Purpose**: Get user by ID

**Headers**: `Authorization: Bearer <token>`

**Response** (200 OK):
```json
{
  "id": "uuid",
  "email": "user@example.com",
  "name": "User Name",
  "role": "USER",
  "createdAt": "2024-01-01T00:00:00Z"
}
```

**Errors**:
- 401: Unauthorized
- 404: User not found

---

#### PATCH /api/users/:id
**Purpose**: Update user information (Admin or self)

**Headers**: `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "name": "Updated Name",
  "role": "ADMIN" // Admin only
}
```

**Response** (200 OK):
```json
{
  "id": "uuid",
  "email": "user@example.com",
  "name": "Updated Name",
  "role": "ADMIN",
  "updatedAt": "2024-01-01T00:00:00Z"
}
```

**Errors**:
- 401: Unauthorized
- 403: Forbidden (cannot change role unless admin)
- 404: User not found

---

#### DELETE /api/users/:id
**Purpose**: Delete user (Admin only)

**Headers**: `Authorization: Bearer <token>`

**Response** (200 OK):
```json
{
  "message": "User deleted successfully"
}
```

**Errors**:
- 401: Unauthorized
- 403: Forbidden (not admin)
- 404: User not found

---

### Role Management Endpoints

#### GET /api/roles
**Purpose**: List all available roles

**Headers**: `Authorization: Bearer <token>`

**Response** (200 OK):
```json
{
  "roles": [
    {
      "id": "uuid",
      "name": "ADMIN",
      "description": "Administrator with full access",
      "permissions": ["*"]
    },
    {
      "id": "uuid",
      "name": "USER",
      "description": "Standard user",
      "permissions": ["read:own", "write:own"]
    }
  ]
}
```

---

#### POST /api/roles
**Purpose**: Create new role (Admin only)

**Headers**: `Authorization: Bearer <token>`

**Request Body**:
```json
{
  "name": "MODERATOR",
  "description": "Moderator role",
  "permissions": ["read:all", "write:content", "delete:content"]
}
```

**Response** (201 Created):
```json
{
  "id": "uuid",
  "name": "MODERATOR",
  "description": "Moderator role",
  "permissions": ["read:all", "write:content", "delete:content"],
  "createdAt": "2024-01-01T00:00:00Z"
}
```

**Errors**:
- 401: Unauthorized
- 403: Forbidden (not admin)
- 409: Role name already exists

---

### Permission Management Endpoints

#### GET /api/permissions
**Purpose**: List all available permissions

**Headers**: `Authorization: Bearer <token>`

**Response** (200 OK):
```json
{
  "permissions": [
    {
      "id": "uuid",
      "resource": "users",
      "action": "read",
      "description": "Read users"
    },
    {
      "id": "uuid",
      "resource": "users",
      "action": "write",
      "description": "Create/update users"
    }
  ]
}
```

---

## Error Response Format

All error responses follow this format:

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {} // Optional additional details
  }
}
```

**Common HTTP Status Codes**:
- 200: Success
- 201: Created
- 400: Bad Request (validation errors)
- 401: Unauthorized (missing or invalid token)
- 403: Forbidden (insufficient permissions)
- 404: Not Found
- 409: Conflict (duplicate resource)
- 500: Internal Server Error

---

## Rate Limiting

API endpoints are rate-limited to prevent abuse:
- Authentication endpoints: 5 requests per minute per IP
- Other endpoints: 100 requests per minute per authenticated user

Rate limit headers are included in responses:
```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1640995200
```

---

## Versioning

Current API version: `v1`

Version is specified in the URL path:
- `/api/v1/users` (future)
- Currently using `/api/users` (defaults to v1)

---

## Notes

- All timestamps are in ISO 8601 format (UTC)
- All UUIDs follow RFC 4122 standard
- Pagination defaults: page=1, limit=10
- Maximum page size: 100 items

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonadableite) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
