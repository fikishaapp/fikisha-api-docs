# Fikisha Vendor API Documentation

## Overview

The Fikisha Vendor API allows third-party applications to integrate with Fikisha's services programmatically. This API enables vendors to perform operations such as airtime purchases, data bundle sales, wallet management, and more through HTTP requests.

## API Endpoints

All production HTTP API endpoints are accessed via the following base URL:

```plaintext
https://api.fikisha.app/v1/
```

## Authentication

All Fikisha Vendor API endpoints require authentication using your Vendor ID and API Key passed in the HTTP headers:

```http
X-Vendor-ID: YOUR_VENDOR_ID_HERE
X-API-Key: YOUR_SECRET_API_KEY_HERE
Content-Type: application/json
Accept: application/json
```

Your Vendor ID and API Key can be obtained from the [Vendor Portal Dashboard](https://vendor.fikisha.com/dashboard).

## Asynchronous Operations

Many operations in the Fikisha API are asynchronous, particularly those involving external providers:

1. You send a request to an endpoint (e.g., `POST /v1/sales/initiate`)
2. The API validates your request and responds with a status code (typically HTTP `202 Accepted`) and a `requestId`
3. The final result is communicated via one of two methods:
   - **Webhooks** (recommended): Real-time notifications sent to your configured endpoint
   - **Status Polling**: Checking the status endpoint periodically using the `requestId`

## API Endpoints

### 1. Sales Operations

#### Initiate Sale Request
`POST /v1/sales/initiate`

This endpoint initiates various types of sales (airtime, data bundles, physical goods). It performs initial validation, deducts the cost from your wallet, and queues the request for asynchronous processing.

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| saleType | string | Yes | Type of sale: `direct_airtime`, `pin_airtime`, `data_bundle`, or `physical_good` |
| totalCost | number | Yes | Total cost to be deducted from wallet (must match amount for airtime) |
| phoneNumber | string | Yes\* | Required for airtime/data sales: Recipient's phone number |
| amount | number | Yes\* | Required for airtime: Amount of airtime |
| provider | string | Yes\* | Required for airtime/data: Provider code (e.g., `safaricom`) |
| bundleId | string | Yes\* | Required for data: Bundle ID from the data bundles list |
| productId | string | Yes\* | Required for physical goods: Product ID |
| quantity | integer | Yes\* | Required for physical goods: Quantity (must be >= 1) |
| clientRef | string | No | Your reference for this transaction |

\* Required based on saleType

**Example Request (Direct Airtime):**

```json
{
  "saleType": "direct_airtime",
  "phoneNumber": "+254712345678",
  "amount": 100,
  "totalCost": 100,
  "provider": "safaricom",
  "clientRef": "my-order-123"
}
```

**Example Response (202 Accepted):**

```json
{
  "success": true,
  "requestId": "sale_req_abc123def456",
  "message": "Sale request accepted and queued.",
  "clientRef": "my-order-123"
}
```

#### Check Sale Request Status
`GET /v1/sales/status/{requestId}`

Retrieves the current status of a previously initiated sale request.

**Response Example (Completed):**

```json
{
  "success": true,
  "requestId": "sale_req_abc123",
  "status": "completed",
  "saleType": "direct_airtime",
  "totalCost": 100,
  "amount": 100,
  "phoneNumber": "+254712345678",
  "provider": "safaricom",
  "clientRef": "my-order-123",
  "createdAt": "2023-08-15T10:25:00Z",
  "updatedAt": "2023-08-15T10:27:30Z",
  "completedAt": "2023-08-15T10:27:30Z"
}
```

#### Get Sales Transaction History
`GET /v1/sales/history`

Retrieves a paginated history of sales transactions.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| limit | integer | No | Maximum records to return (default: 20, max: 100) |
| startAfter | string | No | Request ID of the last transaction from previous page |
| saleType | string | No | Filter by sale type |
| status | string | No | Filter by status (`pending`, `completed`, `failed`, etc.) |
| provider | string | No | Filter by provider |
| phoneNumber | string | No | Filter by recipient phone number |
| startDate | string | No | Filter from this date (ISO 8601) |
| endDate | string | No | Filter to this date (ISO 8601) |

**Response Example:**

```json
{
  "success": true,
  "transactions": [
    {
      "requestId": "sale_req_abc123",
      "saleType": "direct_airtime",
      "status": "completed",
      "totalCost": 100,
      "amount": 100,
      "phoneNumber": "+254712345678",
      "provider": "safaricom",
      "clientRef": "my-reference-123",
      "createdAt": "2023-08-15T10:25:00Z",
      "updatedAt": "2023-08-15T10:27:30Z",
      "completedAt": "2023-08-15T10:27:30Z"
    }
  ],
  "hasMore": true,
  "lastVisibleId": "sale_req_def456"
}
```

### 2. Airtime & Data Services

#### Bulk Request Airtime
`POST /v1/airtime/bulk`

Process multiple airtime requests in a single call.

**Request Example:**

```json
{
  "airtimeRequests": [
    {
      "saleType": "direct_airtime",
      "phoneNumber": "+254711111111",
      "amount": 50,
      "provider": "safaricom",
      "clientRef": "bulk-item-1"
    },
    {
      "saleType": "pin_airtime",
      "phoneNumber": "+254722222222",
      "amount": 100,
      "provider": "airtel",
      "clientRef": "bulk-item-2"
    }
  ]
}
```

**Response Example (202 Accepted):**

```json
{
  "success": true,
  "message": "Bulk request processed. 2 requests queued, 0 failed validation.",
  "requestIds": [ "sale_req_abc123", "sale_req_def456" ],
  "validationErrors": null
}
```

#### List Data Bundles
`GET /v1/data/bundles?provider={provider}`

Lists available data bundles for the specified provider.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| provider | string | Yes | Provider code (e.g., `safaricom`, `airtel`) |

**Response Example:**

```json
{
  "success": true,
  "bundles": [
    {
      "id": "saf_daily_500mb",
      "name": "Daily 500MB + 500 SMS",
      "data": "500MB",
      "price": 50,
      "validity": "24 Hours",
      "provider": "safaricom",
      "description": "Enjoy 500MB data and 500 SMS valid for 24 hours.",
      "isActive": true,
      "ussdCode": "*544*1#",
      "lastUpdated": "2024-08-10T12:00:00Z"
    }
  ]
}
```

#### Purchase Data Bundle
`POST /v1/sales/initiate`

To purchase a data bundle, use the main sales initiate endpoint with `saleType` set to `data_bundle`.

**Request Example:**

```json
{
  "saleType": "data_bundle",
  "phoneNumber": "+254712345678",
  "bundleId": "saf_weekly_2gb",
  "provider": "safaricom",
  "totalCost": 250,
  "bundleName": "Weekly 2GB",
  "clientRef": "data-req-001"
}
```

### 3. Wallet Management

#### Get Wallet Balance
`GET /v1/wallet/balance`

Retrieves the current wallet balance and pending top-up amount.

**Response Example:**

```json
{
  "success": true,
  "balance": 4950.50,
  "pendingTopup": 0,
  "currency": "KES",
  "lastUpdated": "2024-08-15T10:30:05Z"
}
```

#### Get Wallet Transactions
`GET /v1/wallet/transactions`

Retrieves a paginated list of wallet transactions.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| limit | integer | No | Maximum records to return (default: 20, max: 100) |
| startAfter | string | No | ID of the last transaction from previous page |
| type | string | No | Filter by transaction type |
| status | string | No | Filter by status |
| startDate | string | No | Filter from this date (ISO 8601) |
| endDate | string | No | Filter to this date (ISO 8601) |

**Response Example:**

```json
{
  "success": true,
  "transactions": [
    {
      "id": "wtx_commission_abc...",
      "reference": "commission_txn_abc...",
      "amount": 5.00,
      "type": "commission",
      "status": "completed",
      "description": "Commission for airtime sale",
      "relatedTransactionId": "api_txn_xyz...",
      "timestamp": "2024-08-15T10:30:00Z"
    }
  ],
  "hasMore": true,
  "lastVisibleId": "wtx_topup_def..."
}
```

#### Initiate Wallet Top-Up
`POST /v1/wallet/topup`

Initiates a wallet top-up transaction with integrated payment processing.

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| amount | number | Yes | Amount to add to wallet (min. 100 KES) |
| paymentMethod | string | Yes | `mobile_money`, `card`, or `bank` |
| phoneNumber | string | Yes\* | Required for mobile_money |
| metadata | object | No | Additional metadata for this transaction |

\* Required based on paymentMethod

**Example Request (Mobile Money):**

```json
{
  "amount": 1000,
  "paymentMethod": "mobile_money",
  "phoneNumber": "+254712345678"
}
```

**Response Example (202 Accepted):**

```json
{
  "success": true,
  "reference": "topup_abc123def456",
  "paymentDetails": {
    "status": "processing",
    "requestId": "mpesa_xyz789",
    "phoneNumber": "+254712345678",
    "instructions": "You will receive an M-PESA prompt on your phone. Enter your PIN to complete the payment."
  }
}
```

#### Withdraw Commission
`POST /v1/wallet/withdraw`

Requests a withdrawal of accrued commissions to your preferred payment method.

**Request Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| amount | number | Yes | Amount to withdraw (min. 500 KES) |
| withdrawalMethod | string | Yes | `mobile_money` or `bank_transfer` |
| withdrawalDetails | object | Yes | Details for the selected withdrawal method |
| description | string | No | Optional notes for this withdrawal |

**Example Request (Mobile Money):**

```json
{
  "amount": 1000,
  "withdrawalMethod": "mobile_money",
  "withdrawalDetails": {
    "phoneNumber": "+254712345678"
  },
  "description": "August commission withdrawal"
}
```

**Response Example (202 Accepted):**

```json
{
  "success": true,
  "withdrawalId": "withdrawal_abc123def456",
  "message": "Withdrawal of KES 1,000.00 initiated successfully. Reference ID: withdrawal_abc123def456"
}
```

### 4. Product Management

#### Get Available Products
`GET /v1/products`

Retrieves a list of available products that can be sold through your vendor account.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| category | string | No | Filter by category |
| stockStatus | string | No | Filter by stock status (`available`, `low`, `out_of_stock`) |
| minPrice | number | No | Filter products with price >= this value |
| maxPrice | number | No | Filter products with price <= this value |
| limit | integer | No | Maximum products to return (default: 50, max: 100) |

**Response Example:**

```json
{
  "success": true,
  "products": [
    {
      "id": "prod_12345",
      "name": "Premium Smartphone",
      "description": "High-end smartphone with advanced features",
      "category": "electronics",
      "price": 25000,
      "hasVariableAmount": false,
      "provider": "TechBrand",
      "imageUrl": "https://example.com/images/smartphone.jpg",
      "stockStatus": "available",
      "isActive": true,
      "currentStock": 15,
      "commissionRate": 5.0,
      "availableRegions": ["KE", "UG", "TZ"]
    }
  ]
}
```

#### Get Product Categories
`GET /v1/products/categories`

Retrieves a list of available product categories.

**Response Example:**

```json
{
  "success": true,
  "categories": [
    {
      "id": "cat_electronics",
      "name": "Electronics",
      "description": "Electronic devices and accessories",
      "iconName": "devices",
      "isActive": true,
      "sortOrder": 1
    }
  ]
}
```

### 5. Webhook Management

Webhooks allow your server to receive real-time notifications about the final status of asynchronous operations. You can configure and manage your webhook settings in the Vendor Portal.

#### Webhook Events

| Event Type | Description |
|------------|-------------|
| `sale.request.completed` | Sale request successfully completed |
| `sale.request.failed` | Sale request failed |
| `withdrawal.completed` | Withdrawal request completed |
| `withdrawal.failed` | Withdrawal request failed |
| `topup.completed` | Wallet top-up completed |
| `topup.failed` | Wallet top-up failed |

#### Webhook Payload Structure

```json
{
  "eventType": "sale.request.completed",
  "eventId": "evt_123456789abcdef",
  "timestamp": "2023-08-15T10:30:00Z",
  "data": {
    "requestId": "sale_req_abc123",
    "transactionId": "api_txn_xyz789",
    "clientRef": "your-reference-123",
    "status": "completed",
    "type": "direct_airtime",
    "amount": 100,
    "phoneNumber": "+254712345678",
    "provider": "safaricom"
  }
}
```

## Error Handling

The API uses standard HTTP status codes and a consistent JSON error format:

```json
{
  "success": false,
  "error": "A human-readable description of the error.",
  "code": "specific_error_code"
}
```

Common error codes include:

| HTTP Code | Error Code | Description |
|-----------|------------|-------------|
| 400 | invalid-argument | Missing or invalid parameters |
| 401 | unauthenticated | Invalid API credentials |
| 402 | insufficient-funds | Wallet balance too low for operation |
| 403 | permission-denied | Insufficient permissions |
| 404 | not-found | Requested resource not found |
| 429 | rate-limit-exceeded | Too many requests |
| 500 | internal | Server-side error |

## Testing

Currently, Fikisha does not provide a dedicated sandbox environment. When testing:

* Use your live credentials but with minimal amounts (e.g., 10 KES)
* Use test phone numbers that you own
* Always include a unique `clientRef` to identify test transactions
* Maintain a minimal wallet balance during testing

## Best Practices

1. **Idempotency**: Always use unique client references (`clientRef`) to avoid duplicate transactions
2. **Webhooks**: Implement webhook handling for real-time updates
3. **Error Handling**: Implement proper error handling with retries for transient errors
4. **Security**: Secure your API keys and webhook endpoints
5. **Monitoring**: Log and monitor your API usage and responses

## Support

For API support or questions, contact us at [support@fikisha.app](mailto:support@fikisha.app).