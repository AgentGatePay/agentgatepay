# MCP Tools Reference

**15 MCP tools for AI agent integration with AgentGatePay**

Model Context Protocol (MCP) provides a standardized way for AI agents (Claude, ChatGPT, custom agents) to interact with AgentGatePay's payment infrastructure.

---

## Quick Navigation

- [Core Payment Flow](#core-payment-flow) - 4 tools
- [Payment History & Analytics](#payment-history--analytics) - 2 tools
- [Payment Verification](#payment-verification) - 1 tool
- [User Management](#user-management) - 3 tools
- [API Key Management](#api-key-management) - 3 tools
- [Audit & Monitoring](#audit--monitoring) - 2 tools

---

## Integration Options

### Option 1: JSON-RPC (Recommended - OpenAI, Custom Agents)

**Endpoint:** `POST https://mcp.agentgatepay.com/`

This is the primary MCP interface. All 15 tools are available through a single unified endpoint.

```bash
curl -X POST https://mcp.agentgatepay.com/ \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "agentpay_issue_mandate",
      "arguments": {
        "subject": "agent-wallet-address",
        "budget_usd": 10.0,
        "scope": "research",
        "ttl_minutes": 1440
      }
    }
  }'
```

### Option 2: Claude Desktop (Stdio Bridge)

**Server:** `/mcp-server/agentpay-mcp-server.js`

Add to Claude Desktop config (`~/Library/Application Support/Claude/claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "agentgatepay": {
      "command": "node",
      "args": ["/path/to/agentpay-mcp-server.js"],
      "env": {
        "API_KEY": "your-api-key",
        "WALLET_PRIVATE_KEY": "your-private-key"
      }
    }
  }
}
```

---

## Core Payment Flow

### 1. agentpay_issue_mandate

Issue an AP2 budget mandate for spending control.

**Authentication:** Recommended (tracks mandate usage)

**Arguments:**
```json
{
  "subject": "agent-wallet-address-or-id",
  "budget_usd": 10.0,
  "scope": "research,api_access",
  "ttl_minutes": 1440
}
```

**Parameters:**
- `subject` (required): Agent identifier or wallet address
- `budget_usd` (required): Maximum spending limit in USD
- `scope` (required): Comma-separated permissions
- `ttl_minutes` (required): Time to live (max: 43200 = 30 days)

**Returns:**
```json
{
  "mandate_id": "mnd_abc123...",
  "mandate_token": "eyJhbGciOiJFZERTQSIsInR5cCI6IkFQMi1WREMifQ...",
  "subject": "agent-wallet-address",
  "budget_usd": "10.00",
  "scope": "research",
  "issued_at": "2025-12-11T15:30:00Z",
  "expires_at": "2025-12-12T15:30:00Z",
  "status": "active"
}
```

**⚠️ Save the `mandate_token` - required for all payments!**

---

### 2. agentpay_verify_mandate

Verify mandate validity and check remaining budget.

**Authentication:** Not required

**Arguments:**
```json
{
  "mandate_token": "eyJhbGciOiJFZERTQSIsInR5cCI6IkFQMi1WREMifQ..."
}
```

**Returns:**
```json
{
  "valid": true,
  "mandate_id": "mnd_abc123...",
  "subject": "agent-wallet-address",
  "budget_usd": "10.00",
  "budget_remaining": "9.99",
  "scope": "research",
  "expires_at": "2025-12-12T15:30:00Z",
  "status": "active"
}
```

---

### 3. agentpay_create_payment

Create x402 payment requirement (get payment details).

**Authentication:** Requires mandate token

**Arguments:**
```json
{
  "amount_usd": "0.01",
  "resource_path": "/api/premium-content",
  "chain": "base",
  "token": "USDC"
}
```

**Parameters:**
- `amount_usd` (required): Payment amount in USD
- `resource_path` (optional): Resource identifier
- `chain` (optional): `ethereum`, `base`, `polygon`, `arbitrum` (default: `base`)
- `token` (optional): `USDC`, `USDT`, `DAI` (default: `USDC`)

**Returns:**
```json
{
  "status": "payment_required",
  "price_usd": "0.01",
  "receiver_address": "0xReceiverWallet...",
  "chain": "base",
  "token": "USDC",
  "amount_tokens": 10000,
  "payment_instructions": "Send 10000 USDC (6 decimals) to receiver address"
}
```

---

### 4. agentpay_submit_payment

Submit payment and access protected resource.

**Authentication:** Requires API key + mandate token

**Arguments:**
```json
{
  "tx_hash": "0xYourBlockchainTransactionHash",
  "mandate_token": "eyJhbGciOiJFZERTQSIsInR5cCI6IkFQMi1WREMifQ...",
  "chain": "base",
  "token": "USDC",
  "price_usd": "0.01"
}
```

**Parameters:**
- `tx_hash` (required): Blockchain transaction hash
- `mandate_token` (required): AP2 mandate token
- `chain` (required): Blockchain used for payment
- `token` (required): Token used for payment
- `price_usd` (required): Payment amount

**Returns:**
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

## Payment History & Analytics

### 5. agentpay_get_payment_history

Get payment history for authenticated user.

**Authentication:** Required (API key)

**Arguments:**
```json
{
  "limit": 50,
  "offset": 0,
  "start_time": 1699564800,
  "end_time": 1699651200
}
```

**Parameters:**
- `limit` (optional): Results per page (default: 20, max: 100)
- `offset` (optional): Pagination offset
- `start_time` (optional): Unix timestamp
- `end_time` (optional): Unix timestamp

**Returns:**
```json
{
  "total": 42,
  "count": 10,
  "payments": [
    {
      "charge_id": "chg_abc123...",
      "tx_hash": "0xTransaction...",
      "amount_usd": "0.01",
      "chain": "base",
      "token": "USDC",
      "paid_at": "2025-12-11T15:34:50Z",
      "direction": "sent"
    }
  ]
}
```

---

### 6. agentpay_get_analytics

Get spending or revenue analytics for authenticated user.

**Authentication:** Required (API key)

**Arguments:**
```json
{
  "period": "30d"
}
```

**Parameters:**
- `period` (optional): `24h`, `7d`, `30d`, `all` (default: `all`)

**Returns (Agent):**
```json
{
  "role": "agent",
  "total_spent_usd": "5.00",
  "total_transactions": 5,
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

**Returns (Merchant):**
```json
{
  "role": "merchant",
  "total_revenue_usd": "12.50",
  "total_transactions": 12,
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

## Payment Verification

### 7. agentpay_verify_payment

Verify a blockchain payment transaction.

**Authentication:** Required (API key)

**Arguments:**
```json
{
  "tx_hash": "0xBlockchainTransactionHash"
}
```

**Returns:**
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

## User Management

### 8. agentpay_signup

Create a new user account.

**Authentication:** Not required

**Arguments:**
```json
{
  "email": "user@example.com",
  "password": "SecurePassword123",
  "user_type": "agent"
}
```

**Parameters:**
- `email` (required): Valid email address
- `password` (required): Minimum 8 characters
- `user_type` (required): `"agent"`, `"merchant"`, or `"both"`

**Returns:**
```json
{
  "user_id": "usr_abc123...",
  "email": "user@example.com",
  "account_type": "agent",
  "api_key": "pk_live_GFnXNA4o3Tbboj2Ca03uTwqk...",
  "created_at": "2025-12-11T15:30:00Z",
  "reputation_score": 100
}
```

**⚠️ Save the `api_key` - it's only shown once!**

---

### 9. agentpay_get_user_info

Get current user profile information.

**Authentication:** Required (API key)

**Arguments:**
```json
{}
```

**Returns:**
```json
{
  "user_id": "usr_abc123...",
  "email": "user@example.com",
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

### 10. agentpay_add_wallet

Add a blockchain wallet address to your account.

**Authentication:** Required (API key)

**Arguments:**
```json
{
  "chain": "base",
  "address": "0xYourEthereumAddress",
  "label": "Primary Wallet"
}
```

**Parameters:**
- `chain` (required): `ethereum`, `base`, `polygon`, `arbitrum`
- `address` (required): Wallet address (0x...)
- `label` (optional): Friendly name

**Returns:**
```json
{
  "user_id": "usr_abc123...",
  "wallet_address": "0xYourEthereumAddress",
  "chain": "base",
  "label": "Primary Wallet",
  "added_at": "2025-12-11T15:31:00Z"
}
```

---

## API Key Management

### 11. agentpay_create_api_key

Create a new API key for programmatic access.

**Authentication:** Required (API key)

**Arguments:**
```json
{
  "name": "Production Server",
  "permissions": ["read", "write"]
}
```

**Parameters:**
- `name` (required): Friendly name for the key
- `permissions` (optional): Array of permissions

**Returns:**
```json
{
  "key_id": "key_abc123...",
  "api_key": "pk_live_NewGeneratedKey...",
  "key_prefix": "pk_live_NewG",
  "name": "Production Server",
  "created_at": "2025-12-11T15:35:00Z"
}
```

**⚠️ Save the `api_key` - it's only shown once!**

**Rate Limit:** 10 keys per day per user

---

### 12. agentpay_list_api_keys

List all API keys for your account.

**Authentication:** Required (API key)

**Arguments:**
```json
{}
```

**Returns:**
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
    }
  ],
  "count": 1
}
```

---

### 13. agentpay_revoke_api_key

Revoke an API key (cannot be undone).

**Authentication:** Required (API key)

**Arguments:**
```json
{
  "key_id": "key_abc123..."
}
```

**Returns:**
```json
{
  "success": true,
  "key_id": "key_abc123...",
  "key_prefix": "pk_live_GFnX",
  "revoked_at": "2025-12-11T16:50:00Z",
  "message": "API key revoked successfully"
}
```

---

## Audit & Monitoring

### 14. agentpay_get_audit_logs

Get audit logs for current user (authentication-scoped).

**Authentication:** Required (API key)

**Arguments:**
```json
{
  "event_type": "x402_payment_settled",
  "limit": 50,
  "start_time": 1699564800,
  "end_time": 1699651200
}
```

**Parameters:**
- `event_type` (optional): Filter by event type
- `limit` (optional): Results per page (default: 20, max: 100)
- `start_time` (optional): Unix timestamp
- `end_time` (optional): Unix timestamp

**Returns:**
```json
{
  "logs": [
    {
      "id": "log_abc123...",
      "timestamp": "2025-12-11T15:34:50Z",
      "event_type": "x402_payment_settled",
      "details": {
        "tx_hash": "0xTransaction...",
        "amount_usd": "0.01",
        "chain": "base"
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
- `aif_check_passed` - Security check passed

---

### 15. agentpay_get_system_health

Get system health and component status.

**Authentication:** Not required

**Arguments:**
```json
{}
```

**Returns:**
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

## Complete Payment Flow Example

Here's a complete example showing all 4 payment flow tools:

```bash
# 1. Issue mandate
curl -X POST https://mcp.agentgatepay.com/ \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "agentpay_issue_mandate",
      "arguments": {
        "subject": "0xYourWalletAddress",
        "budget_usd": 10.0,
        "scope": "research",
        "ttl_minutes": 1440
      }
    }
  }'

# Save mandate_token from response

# 2. Verify mandate
curl -X POST https://mcp.agentgatepay.com/ \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "agentpay_verify_mandate",
      "arguments": {
        "mandate_token": "YOUR_MANDATE_TOKEN"
      }
    }
  }'

# 3. Create payment (get payment details)
curl -X POST https://mcp.agentgatepay.com/ \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 3,
    "method": "tools/call",
    "params": {
      "name": "agentpay_create_payment",
      "arguments": {
        "amount_usd": "0.01",
        "chain": "base",
        "token": "USDC"
      }
    }
  }'

# 4. Send blockchain transaction (use ethers.js/web3.py)
# ... send USDC to receiver_address from step 3 ...

# 5. Submit payment proof
curl -X POST https://mcp.agentgatepay.com/ \
  -H "Content-Type: application/json" \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 4,
    "method": "tools/call",
    "params": {
      "name": "agentpay_submit_payment",
      "arguments": {
        "tx_hash": "0xYourTransactionHash",
        "mandate_token": "YOUR_MANDATE_TOKEN",
        "chain": "base",
        "token": "USDC",
        "price_usd": "0.01"
      }
    }
  }'
```

---

## Python SDK Integration

Use the official SDK for easier integration:

```bash
pip install agentgatepay-sdk
```

```python
from agentgatepay_sdk import AgentGatePayMCPClient

# Initialize client
client = AgentGatePayMCPClient(
    api_url="https://mcp.agentgatepay.com/",
    api_key="YOUR_API_KEY"
)

# Issue mandate
mandate = client.call_tool("agentpay_issue_mandate", {
    "subject": "0xYourWalletAddress",
    "budget_usd": 10.0,
    "scope": "research",
    "ttl_minutes": 1440
})

# Verify mandate
verification = client.call_tool("agentpay_verify_mandate", {
    "mandate_token": mandate['mandate_token']
})

# Get analytics
analytics = client.call_tool("agentpay_get_analytics", {
    "period": "30d"
})

print(f"Total spent: ${analytics['total_spent_usd']}")
```

---

## Error Handling

All MCP tools return structured error responses:

```json
{
  "success": false,
  "error": "Mandate token invalid or expired",
  "code": "INVALID_MANDATE",
  "timestamp": 1702345678
}
```

**Always check `success` field before proceeding:**

```python
result = client.call_tool("agentpay_issue_mandate", {...})

if not result.get('success', True):
    print(f"Error: {result['error']}")
    return

# Proceed with mandate_token
mandate_token = result['mandate_token']
```

**Common Errors:**
- `INVALID_MANDATE` - Mandate expired or invalid
- `INSUFFICIENT_BUDGET` - Mandate budget too low
- `NONCE_ALREADY_USED` - Replay attack blocked (tx_hash submitted twice)
- `PAYMENT_VERIFICATION_FAILED` - Blockchain transaction invalid
- `RATE_LIMIT_EXCEEDED` - Too many requests

---

## Rate Limits

Rate limits are enforced per-endpoint to prevent abuse:

| Endpoint               | Rate Limit      |
|------------------------|-----------------|
| `/mandates/issue`      | 20 req/min      |
| `/mandates/verify`     | 100 req/min     |
| `/x402/resource`       | 60 req/min      |
| Other endpoints        | 60 req/min      |

MCP tools count toward endpoint rate limits.

---

## Need Help?

- **Documentation**: [docs/](../)
- **GitHub Issues**: [Report bugs](mailto:support@agentgatepay.com)
- **Email**: support@agentgatepay.com
- **Examples**: [Example Code](https://github.com/AgentGatePay/agentgatepay-examples)
- **SDKs**: [Official SDKs](https://github.com/AgentGatePay/agentgatepay-sdks)

---

**Last Updated:** December 2025
**MCP Version:** 1.0
**Tools:** 15 (100% REST API parity)
