# API Endpoints Reference

Complete API reference for integrating AgentGatePay into your application.

**Base URL:** `https://api.agentgatepay.com`

**Authentication:** Optional for most endpoints. Include `x-api-key` header for enhanced features.

---

## Quick Navigation

- [Core Endpoints](#core-endpoints) - Health, x402 resource
- [Mandates](#mandates) - Issue, verify budget authorizations
- [User Management](#user-management) - Signup, profile, wallets
- [API Keys](#api-keys) - Create, list, revoke
- [Payments](#payments) - Verify, status, history
- [Webhooks](#webhooks) - Configure, test, list, delete
- [Analytics](#analytics) - Public stats, user-specific data
- [Merchant](#merchant) - Revenue analytics
- [Audit Logs](#audit-logs) - Transaction history, stats

---

## Authentication

### API Key (Optional)

Include in header for enhanced rate limits and features:

```bash
-H "x-api-key: pk_live_your_key_here"
```

**Get API Key:** Sign up at `/v1/users/signup` (returns API key in response)

Rate limits are per-endpoint (see Rate Limits section below).

---

## Core Endpoints

### GET /health

Check system health and component status.

**Authentication:** Not required

**Request:**
```bash
curl https://api.agentgatepay.com/health
```

**Response (200 OK):**
```json
{
  "status": "healthy",
  "version": "v1.0.0",
  "timestamp": 1702345678,
  "components": {
    "gateway": "ok",
    "payments": "ok",
    "aif": "ok",
    "mandates": "ok",
    "mcp": "ok"
  }
}
```

---

### GET /x402/resource

Access protected resource using HTTP 402 payment protocol.

**Authentication:** Requires `x-mandate` header

**Query Parameters:**
- `chain` (optional): `ethereum`, `base`, `polygon`, `arbitrum` (default: `base`)
- `token` (optional): `USDC`, `USDT`, `DAI` (default: `USDC`)
- `price_usd` (optional): Override default price

**Request (Without Payment - Returns 402):**
```bash
curl -X GET "https://api.agentgatepay.com/x402/resource?chain=base&token=USDC" \
  -H "x-agent-id: your-agent-id" \
  -H "x-mandate: YOUR_MANDATE_TOKEN"
```

**Response (402 Payment Required):**
```json
{
  "status": "payment_required",
  "message": "Payment of 0.01 USD required",
  "price_usd": "0.01",
  "receiver_address": "0xReceiverWallet...",
  "chain": "base",
  "token": "USDC",
  "amount_tokens": 10000,
  "supported_chains": ["ethereum", "base", "polygon", "arbitrum"],
  "supported_tokens": ["USDC", "USDT", "DAI"]
}
```

**Request (With Payment - Returns Resource):**
```bash
curl -X GET "https://api.agentgatepay.com/x402/resource" \
  -H "x-agent-id: your-agent-id" \
  -H "x-mandate: YOUR_MANDATE_TOKEN" \
  -H "x-payment: {\"tx_hash\":\"0xYourTransactionHash\",\"chain\":\"base\",\"token\":\"USDC\"}"
```

**Response (200 OK):**
```json
{
  "status": "success",
  "message": "Payment verified successfully",
  "resource_data": "Protected content here...",
  "charge_id": "chg_abc123...",
  "payment_details": {
    "tx_hash": "0xYourTransactionHash",
    "amount_usd": "0.01",
    "paid_at": "2025-12-11T15:32:00Z",
    "block_number": 12345678,
    "confirmations": 3
  },
  "mandate_remaining_budget": "9.99"
}
```

---

## Mandates

### POST /mandates/issue

Issue an AP2 budget mandate for agent spending control.

**Authentication:** Recommended (API key for tracking)

**Request Body:**
```json
{
  "subject": "agent-wallet-address-or-id",
  "budget_usd": "10.00",
  "scope": "research,api_access",
  "ttl_minutes": 1440
}
```

**Parameters:**
- `subject` (required): Agent identifier or wallet address
- `budget_usd` (required): Maximum spending limit in USD
- `scope` (required): Comma-separated permissions
- `ttl_minutes` (required): Time to live in minutes (max: 43200 = 30 days)

**Request:**
```bash
curl -X POST https://api.agentgatepay.com/mandates/issue \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "subject": "0xYourWalletAddress",
    "budget_usd": "10.00",
    "scope": "research",
    "ttl_minutes": 1440
  }'
```

**Response (200 OK):**
```json
{
  "mandate_token": "eyJhbGc...long_jwt_token",
  "mandate_id": "mnd_abc123...",
  "subject": "0xYourWalletAddress",
  "budget_usd": "10.00",
  "scope": "research",
  "expires_at": "2025-12-12T15:30:00Z",
  "status": "active"
}
```

**⚠️ IMPORTANT:** Save the `mandate_token` - it's required for all payments.

---

### POST /mandates/verify

Verify a mandate token's validity and check remaining budget.

**Authentication:** Not required

**Request Body:**
```json
{
  "mandate_token": "eyJhbGc..."
}
```

**Request:**
```bash
curl -X POST https://api.agentgatepay.com/mandates/verify \
  -H "Content-Type: application/json" \
  -d '{"mandate_token": "YOUR_MANDATE_TOKEN"}'
```

**Response (200 OK):**
```json
{
  "valid": true,
  "mandate_id": "mnd_abc123...",
  "subject": "0xYourWalletAddress",
  "budget_usd": "10.00",
  "budget_remaining": "9.99",
  "scope": "research",
  "expires_at": "2025-12-12T15:30:00Z",
  "status": "active"
}
```

**Response (400 Bad Request - Invalid):**
```json
{
  "valid": false,
  "error": "Mandate expired",
  "expired_at": "2025-12-11T15:30:00Z"
}
```

---

## User Management

### POST /v1/users/signup

Create a new user account (agent, merchant, or both).

**Authentication:** Not required

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123",
  "account_type": "agent"
}
```

**Parameters:**
- `email` (required): Valid email address
- `password` (required): Minimum 8 characters
- `account_type` (required): `"agent"`, `"merchant"`, or `"both"`

**Account Types:**
- **agent**: For AI agents/buyers paying for services
- **merchant**: For service providers receiving payments
- **both**: For platforms that send AND receive payments

**Request:**
```bash
curl -X POST https://api.agentgatepay.com/v1/users/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "agent@example.com",
    "password": "SecurePass123",
    "account_type": "agent"
  }'
```

**Response (200 OK):**
```json
{
  "user_id": "usr_abc123...",
  "email": "agent@example.com",
  "account_type": "agent",
  "api_key": "pk_live_GFnXNA4o3Tbboj2Ca03uTwqk...",
  "created_at": "2025-12-11T15:30:00Z",
  "reputation_score": 100
}
```

**⚠️ IMPORTANT:** Save the `api_key` - it's only shown once!

**Rate Limit:** 5 signups per hour per IP address

---

### GET /v1/users/me

Get current user profile information.

**Authentication:** Required (API key)

**Request:**
```bash
curl https://api.agentgatepay.com/v1/users/me \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "user_id": "usr_abc123...",
  "email": "agent@example.com",
  "account_type": "agent",
  "reputation_score": 100,
  "created_at": "2025-12-11T15:30:00Z",
  "wallets": [
    {
      "address": "0xYourWallet...",
      "label": "Primary Wallet",
      "added_at": "2025-12-11T15:31:00Z"
    }
  ]
}
```

---

### POST /v1/users/wallets/add

Add a wallet address to your account.

**Authentication:** Required (API key)

**Request Body:**
```json
{
  "wallet_address": "0xYourEthereumAddress",
  "label": "Primary Wallet"
}
```

**Request:**
```bash
curl -X POST https://api.agentgatepay.com/v1/users/wallets/add \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "wallet_address": "0xYourEthereumAddress",
    "label": "Primary Revenue Wallet"
  }'
```

**Response (200 OK):**
```json
{
  "user_id": "usr_abc123...",
  "wallet_address": "0xYourEthereumAddress",
  "label": "Primary Revenue Wallet",
  "added_at": "2025-12-11T15:31:00Z"
}
```

---

## API Keys

### POST /v1/api-keys/create

Create a new API key for programmatic access.

**Authentication:** Required (API key)

**Request Body:**
```json
{
  "name": "Production Server",
  "permissions": ["read", "write"]
}
```

**Request:**
```bash
curl -X POST https://api.agentgatepay.com/v1/api-keys/create \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"name": "Production Server"}'
```

**Response (200 OK):**
```json
{
  "key_id": "key_abc123...",
  "api_key": "pk_live_NewGeneratedKey...",
  "key_prefix": "pk_live_NewG",
  "name": "Production Server",
  "created_at": "2025-12-11T15:35:00Z",
  "message": "⚠️ SAVE THIS KEY - You won't see it again!"
}
```

**Rate Limit:** 10 keys per day per user

---

### GET /v1/api-keys/list

List all API keys for your account.

**Authentication:** Required (API key)

**Request:**
```bash
curl https://api.agentgatepay.com/v1/api-keys/list \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "api_keys": [
    {
      "key_id": "key_abc123...",
      "key_prefix": "pk_live_GFnX",
      "name": "Primary Key",
      "created_at": "2025-12-11T15:30:00Z",
      "last_used": "2025-12-11T16:45:00Z",
      "status": "active"
    },
    {
      "key_id": "key_def456...",
      "key_prefix": "pk_live_Tbb",
      "name": "Production Server",
      "created_at": "2025-12-11T15:35:00Z",
      "last_used": null,
      "status": "active"
    }
  ],
  "count": 2
}
```

---

### POST /v1/api-keys/revoke

Revoke an API key (cannot be undone).

**Authentication:** Required (API key)

**Request Body:**
```json
{
  "key_id": "key_abc123..."
}
```

**Request:**
```bash
curl -X POST https://api.agentgatepay.com/v1/api-keys/revoke \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"key_id": "key_def456..."}'
```

**Response (200 OK):**
```json
{
  "success": true,
  "key_id": "key_def456...",
  "key_prefix": "pk_live_Tbb",
  "revoked_at": "2025-12-11T16:50:00Z",
  "message": "API key revoked successfully"
}
```

---

## Payments

### GET /v1/payments/verify/{tx_hash}

Verify a blockchain payment transaction.

**Authentication:** Required (API key)

**Path Parameters:**
- `tx_hash`: Blockchain transaction hash (0x...)

**Request:**
```bash
curl "https://api.agentgatepay.com/v1/payments/verify/0xTransactionHash" \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "verified": true,
  "payment": {
    "tx_hash": "0xTransactionHash",
    "amount_usd": "0.01",
    "receiver_address": "0xMerchantWallet...",
    "payer_address": "0xBuyerWallet...",
    "chain": "base",
    "token": "USDC",
    "paid_at": "2025-12-11T15:34:50Z",
    "block_number": 12345678,
    "confirmations": 12,
    "status": "confirmed"
  }
}
```

---

### GET /v1/payments/status/{tx_hash}

Quick payment status check (lighter response).

**Authentication:** Required (API key)

**Request:**
```bash
curl "https://api.agentgatepay.com/v1/payments/status/0xTransactionHash" \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "tx_hash": "0xTransactionHash",
  "status": "confirmed",
  "verified": true
}
```

---

### GET /v1/payments/list

List payment history for a wallet (sent + received).

**Authentication:** Required (API key)

**Query Parameters:**
- `wallet` (optional): Wallet address to filter by
- `limit` (optional): Results per page (default: 20, max: 100)
- `offset` (optional): Pagination offset

**Request:**
```bash
curl "https://api.agentgatepay.com/v1/payments/list?wallet=0xYourAddress&limit=10" \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "total": 42,
  "count": 10,
  "revenue": 12.50,
  "spending": 5.00,
  "is_registered_wallet": true,
  "payments": [
    {
      "charge_id": "chg_abc123...",
      "tx_hash": "0xTransaction1...",
      "from_address": "0xBuyer...",
      "to_address": "0xYourAddress",
      "amount_usd": "0.01",
      "chain": "base",
      "token": "USDC",
      "paid_at": "2025-12-11T15:34:50Z",
      "direction": "received"
    },
    {
      "charge_id": "chg_def456...",
      "tx_hash": "0xTransaction2...",
      "from_address": "0xYourAddress",
      "to_address": "0xSeller...",
      "amount_usd": "0.02",
      "chain": "ethereum",
      "token": "USDT",
      "paid_at": "2025-12-11T14:20:30Z",
      "direction": "sent"
    }
  ]
}
```

**Response Fields:**
- `revenue`: Total received (`direction` = "received")
- `spending`: Total sent (`direction` = "sent")
- `direction`: Either `"sent"` (payer) or `"received"` (receiver)

---

## Webhooks

### POST /v1/webhooks/configure

Configure webhook for real-time payment notifications.

**Authentication:** Required (API key)

**Request Body:**
```json
{
  "merchant_wallet": "0xYourWallet...",
  "webhook_url": "https://your-server.com/webhook",
  "events": ["payment.received", "payment.confirmed"]
}
```

**Request:**
```bash
curl -X POST https://api.agentgatepay.com/v1/webhooks/configure \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "merchant_wallet": "0xYourWallet...",
    "webhook_url": "https://your-server.com/webhook",
    "events": ["payment.received", "payment.confirmed"]
  }'
```

**Response (200 OK):**
```json
{
  "webhook_id": "wh_abc123...",
  "merchant_wallet": "0xYourWallet...",
  "webhook_url": "https://your-server.com/webhook",
  "webhook_secret": "whsec_GFnXNA4o3Tbboj2Ca03uTwqk...",
  "events": ["payment.received", "payment.confirmed"],
  "status": "active",
  "created_at": "2025-12-11T15:32:00Z"
}
```

**⚠️ IMPORTANT:** Save the `webhook_secret` - use it to verify webhook signatures!

**Webhook Payload Example:**
```json
{
  "event": "payment.received",
  "timestamp": "2025-12-11T15:35:00Z",
  "data": {
    "charge_id": "chg_abc123...",
    "tx_hash": "0xTransactionHash",
    "amount_usd": "0.01",
    "receiver_address": "0xYourWallet...",
    "payer_address": "0xBuyerWallet...",
    "chain": "base",
    "token": "USDC",
    "paid_at": "2025-12-11T15:34:50Z",
    "block_number": 12345678,
    "confirmations": 3
  },
  "signature": "sha256_hmac_signature..."
}
```

**Verify Signature (Python):**
```python
import hmac
import hashlib

def verify_webhook(payload, signature, secret):
    expected = hmac.new(
        secret.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected, signature)
```

---

### GET /v1/webhooks/list

List all configured webhooks.

**Authentication:** Required (API key)

**Request:**
```bash
curl https://api.agentgatepay.com/v1/webhooks/list \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "webhooks": [
    {
      "webhook_id": "wh_abc123...",
      "merchant_wallet": "0xYourWallet...",
      "webhook_url": "https://your-server.com/webhook",
      "events": ["payment.received"],
      "status": "active",
      "delivery_count": 42,
      "failure_count": 0,
      "last_triggered": "2025-12-11T16:30:00Z",
      "created_at": "2025-12-11T15:32:00Z"
    }
  ],
  "count": 1
}
```

---

### POST /v1/webhooks/test

Test webhook delivery.

**Authentication:** Required (API key)

**Request Body:**
```json
{
  "webhook_id": "wh_abc123..."
}
```

**Request:**
```bash
curl -X POST https://api.agentgatepay.com/v1/webhooks/test \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{"webhook_id": "wh_abc123..."}'
```

**Response (200 OK):**
```json
{
  "success": true,
  "webhook_id": "wh_abc123...",
  "test_delivery": {
    "status_code": 200,
    "response_time_ms": 145,
    "delivered_at": "2025-12-11T16:45:00Z"
  },
  "message": "Test webhook delivered successfully"
}
```

---

### DELETE /v1/webhooks/{webhook_id}

Delete a configured webhook.

**Authentication:** Required (API key)

**Path Parameters:**
- `webhook_id`: Webhook identifier

**Request:**
```bash
curl -X DELETE https://api.agentgatepay.com/v1/webhooks/wh_abc123... \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "success": true,
  "webhook_id": "wh_abc123...",
  "deleted_at": "2025-12-11T16:50:00Z",
  "message": "Webhook deleted successfully"
}
```

---

## Analytics

### GET /v1/analytics/public

Get public aggregate platform analytics (no authentication required).

**Authentication:** Not required

**Request:**
```bash
curl https://api.agentgatepay.com/v1/analytics/public
```

**Response (200 OK):**
```json
{
  "total_volume_usd": 12450.75,
  "total_transactions": 342,
  "active_users_approx": 20,
  "success_rate": 98.5,
  "avg_transaction_usd": 36.43,
  "chains": {
    "base": 180,
    "ethereum": 95,
    "polygon": 42,
    "arbitrum": 25
  },
  "tokens": {
    "USDC": 275,
    "USDT": 45,
    "DAI": 22
  },
  "timestamp": 1702345678,
  "cache_ttl_seconds": 900
}
```

**Cache:** 15 minutes

---

### GET /v1/analytics/me

Get user-specific analytics (agent spending or merchant revenue).

**Authentication:** Required (API key)

**Request:**
```bash
curl https://api.agentgatepay.com/v1/analytics/me \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK - Agent):**
```json
{
  "role": "agent",
  "total_spent_usd": "5.00",
  "total_transactions": 5,
  "average_transaction_usd": "1.00",
  "active_mandates": 1,
  "by_chain": {
    "base": "3.00",
    "ethereum": "2.00"
  },
  "by_token": {
    "USDC": "4.00",
    "USDT": "1.00"
  },
  "reputation_score": 100
}
```

**Response (200 OK - Merchant):**
```json
{
  "role": "merchant",
  "total_revenue_usd": "12.50",
  "total_transactions": 12,
  "average_transaction_usd": "1.04",
  "by_chain": {
    "base": "8.00",
    "ethereum": "4.50"
  },
  "by_token": {
    "USDC": "10.00",
    "USDT": "2.50"
  },
  "commission_paid_usd": "0.06",
  "reputation_score": 100
}
```

---

## Merchant

### GET /v1/merchant/revenue

Get detailed revenue analytics for merchant wallet.

**Authentication:** Required (API key)

**Query Parameters:**
- `wallet` (optional): Merchant wallet address (uses account default if not provided)
- `period` (optional): `today`, `week`, `month`, `all_time`, `custom` (default: `all_time`)
- `start_date` (optional): For `custom` period (YYYY-MM-DD)
- `end_date` (optional): For `custom` period (YYYY-MM-DD)

**Request:**
```bash
curl "https://api.agentgatepay.com/v1/merchant/revenue?period=week" \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "merchant_wallet": "0xYourWallet...",
  "total_revenue_usd": "127.50",
  "total_transactions": 45,
  "average_transaction_usd": "2.83",
  "revenue_by_chain": {
    "base": "98.20",
    "ethereum": "15.30",
    "polygon": "10.00",
    "arbitrum": "4.00"
  },
  "revenue_by_token": {
    "USDC": "100.00",
    "USDT": "20.00",
    "DAI": "7.50"
  },
  "period": "week",
  "start_date": "2025-12-04",
  "end_date": "2025-12-11"
}
```

---

## Audit Logs

### GET /audit/logs

Get audit logs with filtering.

**Authentication:** Required (API key) - Users see only their own logs

**Query Parameters:**
- `event_type` (optional): Filter by event type
- `limit` (optional): Results per page (default: 20, max: 100)
- `offset` (optional): Pagination offset
- `start_time` (optional): Unix timestamp
- `end_time` (optional): Unix timestamp

**Request:**
```bash
curl "https://api.agentgatepay.com/audit/logs?event_type=x402_payment_settled&limit=10" \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "logs": [
    {
      "id": "log_abc123...",
      "timestamp": "2025-12-11T15:34:50Z",
      "event_type": "x402_payment_settled",
      "client_id": "0xYourWallet...",
      "details": {
        "tx_hash": "0xTransaction...",
        "amount_usd": "0.01",
        "chain": "base",
        "mandate_budget_deducted": "0.01"
      }
    }
  ],
  "count": 10,
  "has_more": true
}
```

**Event Types:**
- `mandate_issued` - Mandate created
- `mandate_verified` - Mandate validation
- `x402_payment_settled` - Payment completed
- `reputation_updated` - Reputation score changed
- `health_check` - System health check
- `aif_check_passed` - Security check passed
- `aif_check_blocked` - Security check failed

---

### GET /audit/stats

Get audit log statistics.

**Authentication:** Required (API key)

**Query Parameters:**
- `hours` (optional): Time range in hours (default: 24)

**Request:**
```bash
curl "https://api.agentgatepay.com/audit/stats?hours=24" \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "total_events": 42,
  "time_range_hours": 24,
  "event_type_counts": {
    "mandate_issued": 10,
    "mandate_verified": 8,
    "x402_payment_settled": 5,
    "aif_check_passed": 19
  }
}
```

---

### GET /audit/logs/transaction/{tx_hash}

Get all audit logs for a specific blockchain transaction.

**Authentication:** Required (API key)

**Path Parameters:**
- `tx_hash`: Blockchain transaction hash

**Request:**
```bash
curl "https://api.agentgatepay.com/audit/logs/transaction/0xTransactionHash" \
  -H "x-api-key: YOUR_API_KEY"
```

**Response (200 OK):**
```json
{
  "logs": [
    {
      "event_type": "x402_payment_settled",
      "details": {
        "tx_hash": "0xTransactionHash",
        "amount_usd": "0.01",
        "chain": "base",
        "status": "completed"
      },
      "timestamp": "2025-12-11T15:34:50Z"
    },
    {
      "event_type": "reputation_updated",
      "details": {
        "score_delta": 5,
        "old_score": 100,
        "new_score": 105
      },
      "timestamp": "2025-12-11T15:34:50Z"
    }
  ],
  "count": 2,
  "tx_hash": "0xTransactionHash"
}
```

---

## Rate Limits

Rate limits are enforced per-endpoint to prevent abuse:

| Endpoint               | Rate Limit      |
|------------------------|-----------------|
| `/mandates/issue`      | 20 req/min      |
| `/mandates/verify`     | 100 req/min     |
| `/x402/resource`       | 60 req/min      |
| Other endpoints        | 60 req/min      |

**Rate Limit Headers (included in all responses):**
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 55
X-RateLimit-Reset: 1702345678
```

---

## Error Codes

| Status Code | Meaning                  | Description                                  |
|-------------|--------------------------|----------------------------------------------|
| 200         | OK                       | Request successful                           |
| 400         | Bad Request              | Invalid request parameters                   |
| 401         | Unauthorized             | Missing or invalid API key                   |
| 402         | Payment Required         | Payment needed to access resource (x402)     |
| 403         | Forbidden                | Insufficient permissions                     |
| 404         | Not Found                | Resource not found                           |
| 429         | Too Many Requests        | Rate limit exceeded                          |
| 500         | Internal Server Error    | Server error (contact support)               |
| 503         | Service Unavailable      | System maintenance or temporary issue        |

**Error Response Format:**
```json
{
  "error": "Error title",
  "message": "Detailed error description",
  "code": "ERROR_CODE",
  "timestamp": 1702345678
}
```

---

## Need Help?

- **Documentation**: [docs/](../)
- **GitHub Issues**: [Report bugs](mailto:support@agentgatepay.com)
- **Email**: support@agentgatepay.com
- **Examples**: [Example Code](https://github.com/AgentGatePay/agentgatepay-examples)

---

**Last Updated:** December 2025
**API Version:** v1.0.0
