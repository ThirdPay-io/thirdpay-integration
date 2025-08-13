# ThirdPay API - User Management Documentation

## Base URL
```
https://api.thirdpay.io/v1/
```

## Authentication
All API requests require a Bearer token in the Authorization header:
```
Authorization: Bearer YOUR_API_TOKEN
```

The API token will automatically provide the company context and user permissions needed for the requests.

## 🔒 API vs Platform Responsibilities

### What You Can Do via API:
- ✅ **Create users** via bulk import (up to 100 per request)
- ✅ **View user details** and lists with pagination/filtering
- ✅ **Validate email availability** before user creation
- ✅ **Update organization roles** (with proper permissions)
- ✅ **Manage user data** for your organization

### What Requires ThirdPay Platform:
- 🔒 **User onboarding flow** - Email invitations and activation
- 🔒 **User authentication** - Login and password management
- 🔒 **Advanced user management** - Detailed profile editing
- 🔒 **Compliance verification** - KYC and document uploads
- 🔒 **User wallet setup** - Wallet creation and management

### Integration Workflow:
1. **API**: Create users and manage basic data
2. **Platform**: Users complete onboarding at https://thirdpay.io
3. **Platform**: Users access wallet and payment features
4. **API**: Monitor user status and manage organization

---

## Create Users via API

### Bulk Import Members/Users
Create multiple users in a single request for your organization.

**Endpoint:** `POST /members/import`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
Content-Type: application/json
```

**Request Body:**
```json
{
  "members": [
    {
      "emailService": "user@example.com",
      "firstName": "John",
      "lastName": "Doe", 
      "phoneNumber": "+1234567890",
      "country": "United States",
      "address": "123 Main St",
      "city": "New York",
      "state": "NY",
      "zipCode": "10001"
    },
    {
      "emailService": "user2@example.com",
      "firstName": "Jane",
      "lastName": "Smith",
      "phoneNumber": "+1987654321", 
      "country": "United States",
      "address": "456 Oak Ave",
      "city": "Los Angeles",
      "state": "CA",
      "zipCode": "90210"
    }
  ]
}
```

**Field Requirements:**
- `emailService`: Valid email address (required)
- `firstName`: User's first name (required)
- `lastName`: User's last name (required)
- `phoneNumber`: Phone number with country code (required)
- `country`: Full country name (required)
- `address`: Street address (required)
- `city`: City name (required)
- `state`: State/Province (required)
- `zipCode`: Postal/ZIP code (required)

**Response:**
```json
{
  "success": true,
  "summary": {
    "success": true,
    "import_results": {
      "successful": [
        {
          "email": "user@example.com",
          "user_id": "uuid-1234",
          "status": "created"
        }
      ],
      "errors": [
        {
          "email": "invalid@email",
          "error": "Invalid email format",
          "status": "failed"
        }
      ]
    },
    "total_processed": 2
  }
}
```

**Status Codes:**
- `200` - Success
- `400` - Bad Request (missing required fields)
- `401` - Unauthorized (invalid or missing token)
- `500` - Internal Server Error

---

## Get Users

### Get User by ID
Retrieve details for a specific user.

**Endpoint:** `GET /users/{user_id}`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "user-uuid",
    "email": "user@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "phone_number": "+1234567890",
    "country": "United States",
    "address": "123 Main St",
    "city": "New York", 
    "state": "NY",
    "zip_code": "10001",
    "created_at": "2024-01-01T00:00:00Z",
    "updated_at": "2024-01-01T00:00:00Z"
  }
}
```

### Get All Members
Retrieve all members for your organization with pagination and filtering.

**Endpoint:** `GET /members`

**Query Parameters:**
- `search`: Search term for filtering users (optional)
- `status`: Filter by user status (optional)
- `source`: Filter by user source (optional)
- `page`: Page number for pagination (default: 1)
- `limit`: Number of results per page (default: 50, max: 100)

**Example Request:**
```
GET /members?search=john&page=1&limit=20
```

**Response:**
```json
{
  "members": [
    {
      "id": "user-uuid",
      "email": "john@example.com",
      "first_name": "John",
      "last_name": "Doe",
      "status": "active",
      "created_at": "2024-01-01T00:00:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "totalPages": 8
  }
}
```

---

## Validate Users

### Check Email Availability
Validate if an email address is available for registration.

**Endpoint:** `GET /users/validate/email/{email}`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
```

**Example Request:**
```
GET /users/validate/email/newuser@example.com
```

**Response:**
```json
{
  "success": true  // true if email is available, false if taken
}
```

---



## Error Handling

### Common Error Responses

**400 Bad Request:**
```json
{
  "error": "Missing required fields: firstName, lastName, emailService"
}
```

**401 Unauthorized:**
```json
{
  "error": "No authorization token provided"
}
```

**403 Forbidden:**
```json
{
  "error": "Insufficient permissions"
}
```

**404 Not Found:**
```json
{
  "error": "User not found"
}
```

**500 Internal Server Error:**
```json
{
  "error": "Internal server error"
}
```

---

