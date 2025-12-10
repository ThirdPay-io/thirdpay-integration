# ThirdPay API - Payin Transaction Documentation

## Base URL
```
https://api.thirdpay.io/widgets/
```

## Authentication
All API requests require a Bearer token in the Authorization header:
```
Authorization: Bearer YOUR_API_TOKEN
```

The API token will automatically provide the company context and user permissions needed for the requests.

---

## 1. Create Transaction

### Create Payment Transaction
Create a new payment transaction (one-time or subscription) and receive a redirect URL to direct the customer to for payment processing.

**Endpoint:** `POST /create-transaction`

**Headers:**
```
Authorization: Bearer YOUR_API_TOKEN
Content-Type: application/json
```

**Request Body (One-time Payment):**
```json
{
  "external_reference": "TXN-2024-001",
  "site_url": "https://example.com",
  "email": "customer@example.com",
  "amount": 100.50,
  "currency": "USD",
  "success_url": "https://example.com/payment/success",
  "fail_url": "https://example.com/payment/failed",
  "notify_url": "https://example.com/webhooks/payment",
  "name": "John Doe",
  "telephone_number": "+1234567890",
  "identification_number": "ID123456789",
  "address": "123 Main St, New York, NY 10001"
}
```

**Request Body (Subscription Payment):**
```json
{
  "external_reference": "SUB-2024-001",
  "site_url": "https://example.com",
  "email": "customer@example.com",
  "amount": 29.99,
  "cycles": 3,
  "currency": "USD",
  "success_url": "https://example.com/payment/success",
  "fail_url": "https://example.com/payment/failed",
  "notify_url": "https://example.com/webhooks/payment",
  "name": "John Doe",
  "telephone_number": "+1234567890",
  "identification_number": "ID123456789",
  "address": "123 Main St, New York, NY 10001"
}
```

**Required Fields:**
- `external_reference`: Unique reference from your system for internal tracking (required)
- `site_url`: The site URL you want to associate this transaction with (required)
- `email`: Email address of the user you want to collect payment from (required)
- `amount`: Amount in USD (or specified currency) to collect (required)
- `currency`: Currency code (defaults to "USD" if not provided, can be something like "CAD") (optional)
- `success_url`: URL to redirect the user to when the transaction completes successfully (required)
- `fail_url`: URL to redirect the user to when the transaction fails (required)
- `notify_url`: URL to send the webhook POST request to when the transaction is successfully completed (required)

**Optional Fields (for Subscriptions):**
- `cycles`: Subscription billing cycle (optional, defaults to 0 for one-time payment)

**Optional Fields (for smoother KYC journey):**
- `name`: Full name of the customer (optional, but recommended for smoother KYC process)
- `telephone_number`: Phone number of the customer (optional, but recommended for smoother KYC process)
- `identification_number`: Government-issued identification number (optional, but recommended for smoother KYC process)
- `address`: Full address of the customer (optional, but recommended for smoother KYC process)

**Field Details:**

**external_reference:**
- A unique identifier from your system that you can use to track this transaction internally
- This reference will be included in webhook notifications for easy reconciliation
- Recommended format: Use your internal transaction ID or order number

**site_url:**
- The website or application URL associated with this transaction
- Used for tracking and analytics purposes
- Should be a valid URL format

**email:**
- The email address of the customer making the payment
- Must be a valid email format
- This email will be used for transaction notifications and account creation if needed

**amount:**
- The payment amount to collect from the customer
- Should be a positive number
- Amount is in the currency specified by the `currency` field (defaults to USD)
- For subscriptions, this is the amount charged per billing cycle

**cycles (optional):**
- Subscription billing cycle frequency
- Determines how often the customer will be charged the specified `amount`
- If not provided or set to `0`, the transaction is a one-time payment
- Valid values:
  - `0` - **Instant one-time charge**: Single payment, no recurring charges (default)
  - `1` - **Daily charge**: Customer is charged every day at the same time
  - `2` - **Weekly charge**: Customer is charged once per week (every 7 days)
  - `3` - **Monthly charge**: Customer is charged once per month (every 30 days)
  - `4` - **Quarterly charge**: Customer is charged every 3 months (every 90 days)
  - `5` - **Half-yearly charge**: Customer is charged every 6 months (every 180 days)
  - `6` - **Annual charge**: Customer is charged once per year (every 365 days)
- When a subscription is created, the first charge occurs immediately upon successful payment
- Subsequent charges will be automatically processed at the specified interval
- Each recurring charge will trigger a webhook notification to your `notify_url`
- The `external_reference` will be included in all webhook notifications for subscription charges to help you track recurring payments

**currency:**
- The currency code for the transaction (e.g., "USD", "CAD", "EUR")
- Defaults to "USD" if not provided
- Must be a valid ISO currency code supported by ThirdPay

**success_url:**
- The URL where the customer will be redirected after a successful payment
- Should be a fully qualified URL (e.g., "https://example.com/payment/success")
- The customer will be automatically redirected here upon successful transaction completion

**fail_url:**
- The URL where the customer will be redirected if the payment fails
- Should be a fully qualified URL (e.g., "https://example.com/payment/failed")
- The customer will be automatically redirected here if the transaction fails or is cancelled

**notify_url:**
- The webhook endpoint URL where ThirdPay will send transaction completion notifications
- Should be a fully qualified URL that accepts POST requests
- A POST request with transaction details will be sent to this URL when the transaction is successfully completed
- Your webhook endpoint should return a 200 status code to acknowledge receipt

**name (optional):**
- The full name of the customer
- Providing this information can help streamline the KYC (Know Your Customer) verification process
- Format: "First Last" or "Full Name"

**telephone_number (optional):**
- The customer's phone number
- Should include country code (e.g., "+1234567890")
- Providing this can help with identity verification and communication during the payment process

**identification_number (optional):**
- Government-issued identification number (e.g., passport number, national ID, driver's license)
- Helps with KYC compliance and verification
- Format depends on the country and ID type

**address (optional):**
- The customer's full address
- Can include street address, city, state, postal code
- Providing this information can help with address verification and compliance requirements

**Response:**
```json
{
  "success": true,
  "redirect_url": "https://thirdpay.io/payment/transaction-uuid-1234",
  "transaction_id": "transaction-uuid-1234",
  "external_reference": "TXN-2024-001",
  "status": "pending"
}
```

**Response Fields:**
- `success`: Boolean indicating if the transaction was created successfully
- `redirect_url`: The URL you should redirect the customer to for payment processing
- `transaction_id`: Internal ThirdPay transaction identifier
- `external_reference`: Echo of the external reference you provided
- `status`: Initial transaction status (typically "pending")

**Status Codes:**
- `200` - Success
- `400` - Bad Request (validation errors, missing required fields)
- `401` - Unauthorized (invalid or missing token)
- `500` - Internal Server Error

---

## Webhook Notification

### Transaction Completion Webhook
When a transaction is successfully completed, ThirdPay will send a POST request to your `notify_url` with the transaction details.

**Webhook Request:**
```
POST {notify_url}
Content-Type: application/json
```

**Webhook Payload (One-time Payment):**
```json
{
  "transaction_id": "transaction-uuid-1234",
  "external_reference": "TXN-2024-001",
  "status": "completed",
  "amount": 100.50,
  "currency": "USD",
  "email": "customer@example.com",
  "completed_at": "2024-01-01T10:30:00Z",
  "payment_method": "card",
  "site_url": "https://example.com",
  "cycles": 0
}
```

**Webhook Payload (Subscription Payment):**
```json
{
  "transaction_id": "transaction-uuid-1234",
  "external_reference": "SUB-2024-001",
  "status": "completed",
  "amount": 29.99,
  "currency": "USD",
  "email": "customer@example.com",
  "completed_at": "2024-01-01T10:30:00Z",
  "payment_method": "card",
  "site_url": "https://example.com",
  "cycles": 3,
  "subscription_id": "sub-uuid-1234",
  "charge_number": 1,
  "next_charge_date": "2024-02-01T10:30:00Z"
}
```

**Subscription Webhook Notes:**
- For subscription payments, each recurring charge will trigger a separate webhook notification
- The `charge_number` field indicates which charge in the subscription cycle this is (1 for initial, 2 for first renewal, etc.)
- The `next_charge_date` field indicates when the next automatic charge will occur
- The `subscription_id` field allows you to track all charges related to the same subscription
- Use the `external_reference` to correlate subscription charges with your internal records

**Webhook Response:**
Your webhook endpoint should return a `200 OK` status code to acknowledge receipt of the notification. If ThirdPay does not receive a successful response, it may retry the webhook delivery.

**Important Notes:**
- Webhooks are sent via POST requests
- Always verify the webhook signature if provided by ThirdPay
- Implement idempotency checks to handle duplicate webhook deliveries
- Process webhooks asynchronously to avoid timeout issues
- Log all webhook requests for debugging and audit purposes
- For subscriptions, you will receive a webhook for each recurring charge
- Handle subscription webhooks to update your system with recurring payment status

---

## Error Handling

### Common Error Responses

**400 Bad Request - Missing Required Fields:**
```json
{
  "error": "Missing required fields: external_reference, email, amount"
}
```

**400 Bad Request - Invalid Email Format:**
```json
{
  "error": "Invalid email format"
}
```

**400 Bad Request - Invalid URL Format:**
```json
{
  "error": "Invalid URL format for success_url"
}
```

**400 Bad Request - Invalid Amount:**
```json
{
  "error": "Amount must be a positive number"
}
```

**400 Bad Request - Invalid Currency:**
```json
{
  "error": "Unsupported currency code. Supported currencies: USD, CAD, EUR"
}
```

**400 Bad Request - Invalid Cycles Value:**
```json
{
  "error": "Invalid cycles value. Must be between 0 and 6"
}
```

**401 Unauthorized:**
```json
{
  "error": "No authorization token provided"
}
```

**401 Unauthorized - Invalid Token:**
```json
{
  "error": "Invalid or expired API token"
}
```

**500 Internal Server Error:**
```json
{
  "error": "Internal server error. Please try again later."
}
```

---

## Rate Limits & Constraints

### API Rate Limits
- Standard API rate limits apply per Bearer token
- Contact support for higher rate limits if needed

### Transaction Constraints
- **Minimum Amount**: Varies by currency (typically $1.00 USD equivalent)
- **Maximum Amount**: Contact support for transaction limits
- **Currency Support**: USD (default), CAD, and other supported currencies
- **External Reference**: Maximum 255 characters
- **URL Length**: All URL fields must be valid and accessible
- **Cycles**: Must be an integer between 0 and 6 (inclusive)
  - `0` = One-time payment
  - `1` = Daily subscription
  - `2` = Weekly subscription
  - `3` = Monthly subscription
  - `4` = Quarterly subscription
  - `5` = Half-yearly subscription
  - `6` = Annual subscription

### Processing Times
- **Transaction Creation**: Immediate via API
- **Payment Processing**: Real-time through ThirdPay payment gateway
- **Webhook Delivery**: Typically within seconds of transaction completion
- **Redirect URLs**: Available immediately upon transaction creation

---

## Integration Workflow

### Recommended Integration Flow

**For One-time Payments:**
1. **Create Transaction**: Call `POST /create-transaction` with required fields (omit `cycles` or set to `0`)
2. **Receive Redirect URL**: Extract `redirect_url` from the response
3. **Redirect Customer**: Direct the customer to the `redirect_url` for payment
4. **Customer Completes Payment**: Customer completes payment on ThirdPay platform
5. **Receive Webhook**: Your `notify_url` receives POST request with transaction details
6. **Update Your System**: Process the webhook and update your internal records
7. **Customer Redirected**: Customer is redirected to `success_url` or `fail_url` based on outcome

**For Subscription Payments:**
1. **Create Subscription**: Call `POST /create-transaction` with required fields and `cycles` value (1-6)
2. **Receive Redirect URL**: Extract `redirect_url` from the response
3. **Redirect Customer**: Direct the customer to the `redirect_url` for payment
4. **Customer Completes Payment**: Customer completes initial payment on ThirdPay platform
5. **Receive Initial Webhook**: Your `notify_url` receives POST request for the first charge
6. **Update Your System**: Process the webhook, create subscription record, and activate customer access
7. **Customer Redirected**: Customer is redirected to `success_url` or `fail_url` based on outcome
8. **Handle Recurring Charges**: Receive webhook notifications for each subsequent billing cycle
9. **Monitor Subscriptions**: Track subscription status and handle failed recurring payments

### Best Practices

- **Store Transaction References**: Keep track of `external_reference` and `transaction_id` for reconciliation
- **Handle Webhooks Securely**: Verify webhook authenticity and implement proper security measures
- **Provide Optional Fields**: Include optional KYC fields when available to streamline the payment process
- **Test Webhook Endpoints**: Ensure your webhook endpoint is accessible and returns proper responses
- **Monitor Transaction Status**: Use the `transaction_id` to check transaction status if needed
- **Error Handling**: Implement proper error handling for failed transactions and webhook delivery failures
- **Subscription Management**: 
  - Store `subscription_id` from webhook responses to track recurring payments
  - Use `charge_number` to identify which billing cycle each webhook represents
  - Monitor `next_charge_date` to anticipate upcoming charges
  - Implement logic to handle failed recurring payments and subscription cancellations
  - Keep subscription status synchronized with your internal records

---

