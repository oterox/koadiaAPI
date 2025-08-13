## 📁 API_SPECIFICATION.md

# API Specification Document
# Laravel Secure API - Endpoints & Contracts

## 📋 API Overview

**Base URL:** `http://localhost:7001/api`  
**API Version:** v1  
**Authentication:** Bearer Token (Laravel Sanctum)  
**Content-Type:** `application/json`  
**Rate Limiting:** 60 requests per minute per user

## 🔐 Authentication Flow

### Register User
```http
POST /api/auth/register
Content-Type: application/json

{
  "name": "John Doe",
  "email": "test@test.com", 
  "password": "1234",
  "password_confirmation": "1234"
}

Response 201:
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "id": 1,
      "name": "John Doe",
      "email": "test@test.com",
      "email_verified_at": null,
      "created_at": "2024-01-15T10:30:00.000000Z"
    },
    "token": "1|abc123def456..."
  }
}
```

### Login User
```http
POST /api/auth/login
Content-Type: application/json

{
  "email": "test@test.com",
  "password": "1234"
}

Response 200:
{
  "success": true,
  "message": "Login successful",
  "data": {
    "user": { /* user object */ },
    "token": "2|xyz789abc123...",
    "expires_at": "2024-01-22T10:30:00.000000Z"
  }
}
```

### Get Current User
```http
GET /api/auth/me
Authorization: Bearer {token}

Response 200:
{
  "success": true,
  "data": {
    "id": 1,
    "name": "John Doe",
    "email": "test@test.com",
    "roles": ["user"],
    "permissions": ["test-records.read", "test-records.create"]
  }
}
```

### Logout
```http
POST /api/auth/logout
Authorization: Bearer {token}

Response 200:
{
  "success": true,
  "message": "Logged out successfully"
}
```

## 👥 User Management

### List Users (Admin/Manager only)
```http
GET /api/users?page=1&per_page=15&search=john&role=user
Authorization: Bearer {token}

Response 200:
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "John Doe",
      "email": "test@test.com",
      "roles": ["user"],
      "created_at": "2024-01-15T10:30:00.000000Z"
    }
  ],
  "meta": {
    "current_page": 1,
    "per_page": 15,
    "total": 25,
    "last_page": 2,
    "from": 1,
    "to": 15
  }
}
```

### Create User (Admin only)
```http
POST /api/users
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Jane Smith",
  "email": "jane@example.com",
  "password": "1234",
  "roles": ["manager"]
}

Response 201:
{
  "success": true,
  "message": "User created successfully",
  "data": {
    "id": 2,
    "name": "Jane Smith",
    "email": "jane@example.com",
    "roles": ["manager"],
    "created_at": "2024-01-15T11:00:00.000000Z"
  }
}
```

### Update User
```http
PUT /api/users/{id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "Jane Doe Smith",
  "email": "jane.smith@example.com"
}

Response 200:
{
  "success": true,
  "message": "User updated successfully",
  "data": { /* updated user object */ }
}
```

### Delete User (Admin only)
```http
DELETE /api/users/{id}
Authorization: Bearer {token}

Response 204: (No Content)
```

## 📝 Test Records CRUD

### List Test Records
```http
GET /api/test-records?page=1&status=active&priority=3&user_id=1&sort=created_at&direction=desc
Authorization: Bearer {token}

Response 200:
{
  "success": true,
  "data": [
    {
      "id": 1,
      "title": "Sample Test Record",
      "description": "This is a test description",
      "status": "active",
      "priority": 3,
      "due_date": "2024-02-01",
      "is_featured": false,
      "metadata": {
        "tags": ["important", "backend"],
        "category": "testing"
      },
      "user": {
        "id": 1,
        "name": "John Doe",
        "email": "test@test.com"
      },
      "created_at": "2024-01-15T10:30:00.000000Z",
      "updated_at": "2024-01-15T10:30:00.000000Z"
    }
  ],
  "meta": {
    "pagination": { /* pagination info */ },
    "filters_applied": {
      "status": "active",
      "priority": 3,
      "user_id": 1
    }
  }
}
```

### Create Test Record
```http
POST /api/test-records
Authorization: Bearer {token}
Content-Type: application/json

{
  "title": "New Test Record",
  "description": "Detailed description here",
  "status": "pending",
  "priority": 2,
  "due_date": "2024-02-15",
  "is_featured": true,
  "metadata": {
    "tags": ["new", "feature"],
    "estimated_hours": 8
  }
}

Response 201:
{
  "success": true,
  "message": "Test record created successfully",
  "data": { /* created record object */ }
}
```

### Show Test Record
```http
GET /api/test-records/{id}
Authorization: Bearer {token}

Response 200:
{
  "success": true,
  "data": { /* full test record object with user relationship */ }
}
```

### Update Test Record
```http
PUT /api/test-records/{id}
Authorization: Bearer {token}
Content-Type: application/json

{
  "title": "Updated Title",
  "status": "completed",
  "priority": 1
}

Response 200:
{
  "success": true,
  "message": "Test record updated successfully",
  "data": { /* updated record object */ }
}
```

### Delete Test Record
```http
DELETE /api/test-records/{id}
Authorization: Bearer {token}

Response 204: (No Content)
```

## 🛡️ Roles & Permissions (Admin only)

### List Roles
```http
GET /api/roles
Authorization: Bearer {token}

Response 200:
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "admin",
      "guard_name": "web",
      "permissions": [
        "users.create", "users.read", "users.update", "users.delete",
        "test-records.create", "test-records.read", "test-records.update", "test-records.delete",
        "roles.manage"
      ],
      "users_count": 2
    },
    {
      "id": 2,
      "name": "manager", 
      "guard_name": "web",
      "permissions": [
        "users.read", "users.update",
        "test-records.create", "test-records.read", "test-records.update", "test-records.delete"
      ],
      "users_count": 5
    }
  ]
}
```

### Create Role
```http
POST /api/roles
Authorization: Bearer {token}
Content-Type: application/json

{
  "name": "editor",
  "permissions": [
    "test-records.create",
    "test-records.read", 
    "test-records.update"
  ]
}

Response 201:
{
  "success": true,
  "message": "Role created successfully",
  "data": { /* created role object */ }
}
```

### Assign Role to User
```http
POST /api/users/{user_id}/roles
Authorization: Bearer {token}
Content-Type: application/json

{
  "roles": ["manager", "editor"]
}

Response 200:
{
  "success": true,
  "message": "Roles assigned successfully",
  "data": {
    "user": { /* user object with updated roles */ }
  }
}
```

### List Available Permissions
```http
GET /api/permissions
Authorization: Bearer {token}

Response 200:
{
  "success": true,
  "data": [
    {
      "id": 1,
      "name": "users.create",
      "guard_name": "web"
    },
    {
      "id": 2,
      "name": "users.read", 
      "guard_name": "web"
    }
    // ... more permissions
  ]
}
```

## ❌ Error Responses

### Validation Error (422)
```json
{
  "success": false,
  "message": "The given data was invalid.",
  "errors": {
    "email": [
      "The email field is required."
    ],
    "password": [
      "The password must be at least 8 characters."
    ]
  }
}
```

### Authentication Error (401)
```json
{
  "success": false,
  "message": "Unauthenticated."
}
```

### Authorization Error (403)
```json
{
  "success": false,
  "message": "This action is unauthorized."
}
```

### Not Found Error (404)
```json
{
  "success": false,
  "message": "Resource not found."
}
```

### Rate Limit Error (429)
```json
{
  "success": false,
  "message": "Too Many Attempts.",
  "retry_after": 60
}
```

### Server Error (500)
```json
{
  "success": false,
  "message": "Server Error. Please try again later."
}
```

## 📊 Query Parameters

### Pagination
- `page`: Page number (default: 1)
- `per_page`: Items per page (default: 15, max: 100)

### Filtering (Test Records)
- `status`: Filter by status (active, inactive, pending)
- `priority`: Filter by priority (1-5)
- `user_id`: Filter by user ID
- `is_featured`: Filter featured records (true/false)
- `due_date_from`: Filter by due date range start
- `due_date_to`: Filter by due date range end

### Sorting
- `sort`: Field to sort by (created_at, updated_at, title, priority)
- `direction`: Sort direction (asc, desc)

### Search
- `search`: Search in title and description fields

## 🔒 Permission Matrix

| Endpoint | Admin | Manager | User |
|----------|-------|---------|------|
| GET /users | ✅ | ✅ | ❌ |
| POST /users | ✅ | ❌ | ❌ |
| PUT /users/{id} | ✅ | ✅ (own) | ✅ (own) |
| DELETE /users/{id} | ✅ | ❌ | ❌ |
| GET /test-records | ✅ | ✅ | ✅ (own) |
| POST /test-records | ✅ | ✅ | ✅ |
| PUT /test-records/{id} | ✅ | ✅ | ✅ (own) |
| DELETE /test-records/{id} | ✅ | ✅ | ✅ (own) |
| GET /roles | ✅ | ❌ | ❌ |
| POST /roles | ✅ | ❌ | ❌ |

## 📝 Request/Response Examples

### Complex Test Record with Metadata
```json
{
  "title": "Complex Feature Implementation",
  "description": "Implementing advanced search with filters and pagination",
  "status": "active",
  "priority": 4,
  "due_date": "2024-03-01",
  "is_featured": true,
  "metadata": {
    "tags": ["feature", "search", "backend"],
    "estimated_hours": 40,
    "complexity": "high",
    "assigned_team": "backend",
    "dependencies": ["user-management", "authentication"],
    "technical_notes": {
      "database_changes": true,
      "api_changes": true,
      "breaking_changes": false
    }
  }
}
```

### Bulk Operations (Future Enhancement)
```http
POST /api/test-records/bulk
Authorization: Bearer {token}
Content-Type: application/json

{
  "action": "update_status",
  "ids": [1, 2, 3, 4],
  "data": {
    "status": "completed"
  }
}