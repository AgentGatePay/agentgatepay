# Curl Examples

Ready-to-run curl commands for testing AgentGatePay API.

---

## Prerequisites

Set environment variables:

```bash
export API_URL="https://api.agentgatepay.com"
export AGENT_API_KEY="pk_live_your_key_here"  # Get from signup
export MANDATE_TOKEN=""  # Will be set after issuing mandate
```

---

## User Management

### Create Account

```bash
curl -X POST "$API_URL/v1/users/signup" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "your-email@example.com",
    "password": "SecurePassword123",
    "account_type": "agent"
  }'
```

**Save the `api_key` from response!**

### Get User Profile

```bash
curl "$API_URL/v1/users/me" \
  -H "x-api-key: $AGENT_API_KEY"
```

### Add Wallet Address

```bash
curl -X POST "$API_URL/v1/users/wallets/add" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $AGENT_API_KEY" \
  -d '{
    "wallet_address": "0xYourEthereumAddress",
    "label": "Primary Wallet"
  }'
```

---

## Mandates

### Issue Mandate

```bash
curl -X POST "$API_URL/mandates/issue" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $AGENT_API_KEY" \
  -d '{
    "subject": "0xYourWalletAddress",
    "budget_usd": "10.00",
    "scope": "research",
    "ttl_minutes": 1440
  }'
```

**Save the `mandate_token` from response:**
```bash
export MANDATE_TOKEN="eyJhbGc..."  # Copy full token
```

### Verify Mandate

```bash
curl -X POST "$API_URL/mandates/verify" \
  -H "Content-Type: application/json" \
  -d "{\"mandate_token\": \"$MANDATE_TOKEN\"}"
```

---

## Payments (x402 Protocol)

### Request Resource (Get Payment Details)

```bash
curl -X GET "$API_URL/x402/resource?chain=base&token=USDC" \
  -H "x-agent-id: my-agent" \
  -H "x-mandate: $MANDATE_TOKEN"
```

**Response:** 402 Payment Required with payment details

### Submit Payment

First, send blockchain transaction using ethers.js/web3.py, then:

```bash
curl -X GET "$API_URL/x402/resource" \
  -H "x-agent-id: my-agent" \
  -H "x-mandate: $MANDATE_TOKEN" \
  -H 'x-payment: {"tx_hash":"0xYourTransactionHash","chain":"base","token":"USDC"}'
```

**Response:** 200 OK with resource access

---

## Payment Verification

### Verify Payment by TX Hash

```bash
curl "$API_URL/v1/payments/verify/0xTransactionHash" \
  -H "x-api-key: $AGENT_API_KEY"
```

### Get Payment Status

```bash
curl "$API_URL/v1/payments/status/0xTransactionHash" \
  -H "x-api-key: $AGENT_API_KEY"
```

### List Payment History

```bash
curl "$API_URL/v1/payments/list?wallet=0xYourAddress&limit=10" \
  -H "x-api-key: $AGENT_API_KEY"
```

---

## Webhooks

### Configure Webhook

```bash
curl -X POST "$API_URL/v1/webhooks/configure" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $AGENT_API_KEY" \
  -d '{
    "merchant_wallet": "0xYourWallet",
    "webhook_url": "https://your-server.com/webhook",
    "events": ["payment.received", "payment.confirmed"]
  }'
```

**Save the `webhook_secret` from response!**

### List Webhooks

```bash
curl "$API_URL/v1/webhooks/list" \
  -H "x-api-key: $AGENT_API_KEY"
```

### Test Webhook

```bash
curl -X POST "$API_URL/v1/webhooks/test" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $AGENT_API_KEY" \
  -d '{"webhook_id": "wh_abc123..."}'
```

### Delete Webhook

```bash
curl -X DELETE "$API_URL/v1/webhooks/wh_abc123..." \
  -H "x-api-key: $AGENT_API_KEY"
```

---

## Analytics

### Public Analytics (No Auth Required)

```bash
curl "$API_URL/v1/analytics/public"
```

### User-Specific Analytics

```bash
curl "$API_URL/v1/analytics/me" \
  -H "x-api-key: $AGENT_API_KEY"
```

### Merchant Revenue

```bash
curl "$API_URL/v1/merchant/revenue?period=week" \
  -H "x-api-key: $AGENT_API_KEY"
```

---

## API Keys

### Create API Key

```bash
curl -X POST "$API_URL/v1/api-keys/create" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $AGENT_API_KEY" \
  -d '{"name": "Production Server"}'
```

### List API Keys

```bash
curl "$API_URL/v1/api-keys/list" \
  -H "x-api-key: $AGENT_API_KEY"
```

### Revoke API Key

```bash
curl -X POST "$API_URL/v1/api-keys/revoke" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $AGENT_API_KEY" \
  -d '{"key_id": "key_abc123..."}'
```

---

## Audit Logs

### List Audit Logs

```bash
curl "$API_URL/audit/logs?event_type=x402_payment_settled&limit=10" \
  -H "x-api-key: $AGENT_API_KEY"
```

### Get Audit Statistics

```bash
curl "$API_URL/audit/stats?hours=24" \
  -H "x-api-key: $AGENT_API_KEY"
```

### Get Transaction Logs

```bash
curl "$API_URL/audit/logs/transaction/0xTransactionHash" \
  -H "x-api-key: $AGENT_API_KEY"
```

---

## System Health

### Health Check

```bash
curl "$API_URL/health"
```

---

## MCP (Model Context Protocol)

### List MCP Tools

```bash
curl -X POST "https://mcp.agentgatepay.com/" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/list",
    "params": {}
  }'
```

### Call MCP Tool

```bash
curl -X POST "https://mcp.agentgatepay.com/" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $AGENT_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "tools/call",
    "params": {
      "name": "agentpay_get_system_health",
      "arguments": {}
    }
  }'
```

---

## Complete Payment Flow Script

Save as `test_payment.sh`:

```bash
#!/bin/bash

export API_URL="https://api.agentgatepay.com"

echo "==================================="
echo "AgentGatePay - Complete Test Flow"
echo "==================================="

# 1. Health Check
echo -e "\n1. Health Check"
curl -s "$API_URL/health" | jq '.'

# 2. Create Account
echo -e "\n2. Create Account"
SIGNUP_RESPONSE=$(curl -s -X POST "$API_URL/v1/users/signup" \
  -H "Content-Type: application/json" \
  -d '{
    "email": "test-'$(date +%s)'@example.com",
    "password": "SecurePass123",
    "account_type": "agent"
  }')

echo "$SIGNUP_RESPONSE" | jq '.'
export AGENT_API_KEY=$(echo "$SIGNUP_RESPONSE" | jq -r '.api_key')
echo "API Key: $AGENT_API_KEY"

# 3. Issue Mandate
echo -e "\n3. Issue Mandate"
MANDATE_RESPONSE=$(curl -s -X POST "$API_URL/mandates/issue" \
  -H "Content-Type: application/json" \
  -H "x-api-key: $AGENT_API_KEY" \
  -d '{
    "subject": "test-agent-'$(date +%s)'",
    "budget_usd": "10.00",
    "scope": "research",
    "ttl_minutes": 60
  }')

echo "$MANDATE_RESPONSE" | jq '.'
export MANDATE_TOKEN=$(echo "$MANDATE_RESPONSE" | jq -r '.mandate_token')
echo "Mandate Token: ${MANDATE_TOKEN:0:50}..."

# 4. Verify Mandate
echo -e "\n4. Verify Mandate"
curl -s -X POST "$API_URL/mandates/verify" \
  -H "Content-Type: application/json" \
  -d "{\"mandate_token\": \"$MANDATE_TOKEN\"}" | jq '.'

# 5. Request Resource (402)
echo -e "\n5. Request Resource (Expect 402)"
curl -s -X GET "$API_URL/x402/resource?chain=base&token=USDC" \
  -H "x-agent-id: test-agent" \
  -H "x-mandate: $MANDATE_TOKEN" | jq '.'

# 6. Get Analytics
echo -e "\n6. Get User Analytics"
curl -s "$API_URL/v1/analytics/me" \
  -H "x-api-key: $AGENT_API_KEY" | jq '.'

echo -e "\n==================================="
echo "Test Complete!"
echo "==================================="
```

**Run:**
```bash
chmod +x test_payment.sh
./test_payment.sh
```

---

## Multi-Chain Examples

### Ethereum

```bash
curl -X GET "$API_URL/x402/resource?chain=ethereum&token=USDC" \
  -H "x-agent-id: my-agent" \
  -H "x-mandate: $MANDATE_TOKEN"
```

### Base

```bash
curl -X GET "$API_URL/x402/resource?chain=base&token=USDC" \
  -H "x-agent-id: my-agent" \
  -H "x-mandate: $MANDATE_TOKEN"
```

### Polygon

```bash
curl -X GET "$API_URL/x402/resource?chain=polygon&token=USDC" \
  -H "x-agent-id: my-agent" \
  -H "x-mandate: $MANDATE_TOKEN"
```

### Arbitrum

```bash
curl -X GET "$API_URL/x402/resource?chain=arbitrum&token=USDC" \
  -H "x-agent-id: my-agent" \
  -H "x-mandate: $MANDATE_TOKEN"
```

---

## Troubleshooting

### Pretty Print JSON

Add `| jq '.'` to any curl command:

```bash
curl "$API_URL/health" | jq '.'
```

### See HTTP Headers

Add `-i` flag:

```bash
curl -i "$API_URL/health"
```

### Verbose Output

Add `-v` flag for debugging:

```bash
curl -v "$API_URL/health"
```

### Save Response

Use `-o` flag:

```bash
curl "$API_URL/health" -o health.json
```

---

## Rate Limits

Check rate limit headers in response:

```bash
curl -i "$API_URL/health" | grep "X-RateLimit"
```

**Headers:**
- `X-RateLimit-Limit: 100`
- `X-RateLimit-Remaining: 95`
- `X-RateLimit-Reset: 1702345678`

---

## Support

- 📖 **API Documentation**: [../api/endpoints-reference.md](../api/endpoints-reference.md)
- 💬 **GitHub Issues**: [Report bugs](mailto:support@agentgatepay.com)
- 📧 **Email**: support@agentgatepay.com

---

**Last Updated:** December 2025
**Examples:** 30+
