# ThirdPay API - Payout Management Documentation

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
- ✅ **Create payout batches** with up to 100 recipients
- ✅ **Add recipients** to existing batches (until 100 limit)
- ✅ **View batch details** and recipient lists
- ✅ **Check batch status** (draft, ready, processing, completed)
- ✅ **Create multiple batches** when first batch reaches capacity

### What Requires ThirdPay Platform:
- 🔒 **Execute payouts** - All payout execution requires platform login
- 🔒 **2FA verification** - Security verification handled on platform
- 🔒 **Final approval** - Batch review and approval on platform
- 🔒 **Transaction monitoring** - Real-time payout tracking on platform

### Integration Workflow:
1. **API**: Create and manage payout batches
2. **Platform**: Log in to https://thirdpay.io to execute payouts
3. **API**: Monitor batch status after execution

---

## 1. Initiate Bulk Payout

### Create Payout Batch
Create a new bulk payout batch with multiple recipients.

**Endpoint:** `POST /payouts/batch`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
Content-Type: application/json
```

**Request Body:**
```json
{
  "payment_reference": "PAY-2024-001",
  "payees": [
    {
      "email": "recipient1@example.com",
      "amount": 100.50,
      "memo": "Payment for services",
      "status": "pending"           
    },
    {
      "email": "recipient2@example.com", 
      "amount": 250.00,
      "memo": "Monthly bonus",
      "status": "pending"
    },
    {
      "email": "invalid@email",
      "amount": 50.00,
      "memo": "Refund",
      "status": "cancelled"         
    }
  ]
}
```

**Field Requirements:**
- `payment_reference`: Unique reference for this batch (required)
- `payees`: Array of recipients (optional, max 100 per batch)
  - `email`: Valid email address (required)
  - `amount`: Payment amount in USD (required, min: $3.00)
  - `memo`: Payment description (optional, max 200 characters)
  - `status`: "pending" for valid payees, "cancelled" for invalid emails

**Response:**
```json
{
  "batch": {
    "id": "batch-uuid-1234",
    "reference": "PAY-2024-001", 
    "status": "pending",
    "total_amount": 350.50,
    "total_recipients": 3,
    "valid_recipients": 2,
    "cancelled_recipients": 1,
    "created_at": "2024-01-01T00:00:00Z"
  },
  "total_records": 3,
  "payees": [
    {
      "id": "payee-uuid-1",
      "email": "recipient1@example.com",
      "amount": 100.50,
      "status": "pending"
    },
    {
      "id": "payee-uuid-2", 
      "email": "recipient2@example.com",
      "amount": 250.00,
      "status": "pending"
    },
    {
      "id": "payee-uuid-3",
      "email": "invalid@email",
      "amount": 50.00,
      "status": "cancelled"
    }
  ],
  "failed_total": 1
}
```

**Status Codes:**
- `200` - Success
- `400` - Bad Request (validation errors)
- `401` - Unauthorized
- `403` - Insufficient wallet balance
- `500` - Internal Server Error

---

## 2. Get Payout Batch Details

### Get Existing Payout Batch
Retrieve details of an existing payout batch to review recipients and status.

**Endpoint:** `GET /payouts/batches/{batch_id}`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
```

**Response:**
```json
{
  "success": true,
  "batch": {
    "id": "batch-uuid-1234",
    "reference": "PAY-2024-001",
    "status": "draft",
    "total_amount": 350.50,
    "total_recipients": 3,
    "remaining_capacity": 97,
    "created_at": "2024-01-01T00:00:00Z"
  },
  "recipients": [
    {
      "id": "payee-uuid-1",
      "email": "recipient1@example.com", 
      "amount": 100.50,
      "status": "pending",
      "memo": "Payment for services"
    }
  ],
  "statistics": {
    "total_recipients": 3,
    "pending_recipients": 2,
    "completed_recipients": 0,
    "cancelled_recipients": 1,
    "failed_recipients": 0,
    "total_amount": 350.50,
    "active_amount": 350.50
  }
}
```

**Note:** Use this endpoint to review batch details before adding more recipients or proceeding to the ThirdPay platform for payout execution.

---

## 3. Check Status of Batch Payout

### Get Payout Batch Status
Check the current status and progress of a payout batch.

**Endpoint:** `GET /payouts/batches/{batch_id}/status`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
```

**Response:**
```json
{
  "success": true,
  "batch_id": "batch-uuid-1234",
  "batch_status": "draft",
  "total_recipients": 2,
  "total_amount": 350.50,
  "created_at": "2024-01-01T00:00:00Z",
  "updated_at": "2024-01-01T10:30:00Z"
}
```

**Batch Status Values:**
- `draft`: Batch created via API, ready for review on ThirdPay platform
- `ready`: Batch finalized and ready for payout execution on ThirdPay platform
- `processing`: Payments are being sent (managed through ThirdPay platform)
- `completed`: All payments processed successfully
- `failed`: Batch processing failed
- `cancelled`: Batch was cancelled

**🔒 Important:** Batch execution (processing payments) can only be initiated through the ThirdPay platform at https://thirdpay.io. API clients can create and manage batches, but payout execution requires platform authentication and approval.

---

## Adding Recipients to Existing Batches

### Add Recipients to a Batch
You can add more recipients to an existing payout batch until it reaches the 100 recipient limit.

**Endpoint:** `POST /payouts/batches/{batch_id}/recipients`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
Content-Type: application/json
```

**Request Body:**
```json
{
  "payees": [
    {
      "email": "newrecipient@example.com",
      "amount": 75.00,
      "memo": "Additional payment",
      "status": "pending"
    }
  ]
}
```

**Response:**
```json
{
  "success": true,
  "batch_id": "batch-uuid-1234",
  "added_payees": 1,
  "total_payees": 3,
  "remaining_capacity": 97,
  "new_total_amount": 425.50
}
```

**🚨 Important Limits:**
- **Maximum 100 payees per batch**
- **Once a batch reaches 100 payees, you must create a new batch**

### Batch Management Rules
1. **Create batches via API** - Use `POST /payouts/batch` 
2. **Add payees** - Use `POST /payouts/batches/{batch_id}/recipients` until 100 limit
3. **Multiple batches** - Create additional batches when first batch is full
4. **Execution** - All payout execution must be done through ThirdPay platform
5. **Platform review** - Log in to https://thirdpay.io to review and execute payouts

---

## Get All Payout Batches

### List Payout Batches
Retrieve all payout batches for your organization with pagination.

**Endpoint:** `GET /payouts`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
```

**Query Parameters:**
- `page`: Page number (default: 1)
- `limit`: Results per page (default: 50, max: 100)

**Example Request:**
```
GET /payouts?page=1&limit=20
```

**Response:**
```json
{
  "payouts": [
    {
      "id": "batch-uuid-1234",
      "reference": "PAY-2024-001",
      "status": "completed",
      "total_recipients": 3,
      "valid_recipients": 2,
      "total_amount": 350.50,
      "created_at": "2024-01-01T00:00:00Z",
      "completed_at": "2024-01-01T10:30:00Z"
    },
    {
      "id": "batch-uuid-5678", 
      "reference": "PAY-2024-002",
      "status": "pending",
      "total_recipients": 5,
      "valid_recipients": 5,
      "total_amount": 1000.00,
      "created_at": "2024-01-02T00:00:00Z",
      "completed_at": null
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 25,
    "totalPages": 2
  }
}
```

---

## Error Handling

### Common Error Responses

**400 Bad Request - Validation Error:**
```json
{
  "error": "Missing required fields: payment_reference, payees"
}
```

**400 Bad Request - Invalid Email:**
```json
{
  "error": "Invalid email format in payees array"
}
```

**401 Unauthorized:**
```json
{
  "error": "No authorization token provided"
}
```

**403 Forbidden - Insufficient Balance:**
```json
{
  "error": "Insufficient wallet balance. Required: $1000.00, Available: $500.00"
}
```

**403 Forbidden - Invalid 2FA:**
```json
{
  "success": false,
  "message": "Invalid or expired 2FA code"
}
```

**404 Not Found:**
```json
{
  "error": "Batch not found"
}
```

**429 Too Many Requests:**
```json
{
  "error": "Rate limit exceeded. Please try again in 60 seconds."
}
```

---

## Rate Limits & Constraints

### API Rate Limits
- Standard API rate limits apply per Bearer token

### Batch Constraints
- **Maximum Payees**: 100 per batch (API limitation)
- **Minimum Amount**: $3.00 per payee
- **Reference Length**: 50 characters maximum
- **Memo Length**: 200 characters maximum
- **Execution**: All payouts must be executed through ThirdPay platform

### Processing Times
- **Batch Creation**: Immediate via API
- **Batch Review**: Manual review on ThirdPay platform
- **Payout Execution**: Depends on batch size (platform only)
- **Status Updates**: Real-time via API

---

## Email Verification

### Verify Email Address
Verify if an email address is valid and exists on the ThirdPay platform.

**Endpoint:** `GET /users/validate/email/{email}`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
```

**Example Request:**
```
GET /users/validate/email/user@example.com
```

**Response:**
```json
{
  "success": true  // true if email exists on platform, false if not found
}
```

**Use Case:** Verify recipient emails before adding them to payout batches to reduce cancelled payees.

---
