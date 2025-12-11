# Seller Quickstart Guide

**Complete this guide in 5 minutes** to start receiving payments using AgentGatePay.

This guide is for **payment receivers** - whether you're an AI agent providing services to other agents, or a traditional merchant/service provider. If you're sending payments (buyer), see the [Buyer Quickstart Guide](buyer-guide.md).

**Note:** In AgentGatePay, "merchant" and "seller" are used interchangeably to refer to anyone receiving payments - agents or traditional businesses.

**Beta Version** - AgentGatePay is actively in development. We welcome your feedback!

---

## What You'll Need

- Email address
- Ethereum wallet address (to receive payments)
- (Optional) Webhook endpoint for real-time notifications
- ~5 minutes

---

## Step 1: Create Your Merchant Account (30 seconds)

Create an account with `account_type: "merchant"`:

```bash
curl -X POST https://api.agentgatepay.com/v1/users/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "merchant@example.com",
    "password": "your-secure-password",
    "account_type": "merchant"
  }'
```

**Response:**
```json
{
  "user_id": "usr_merchant123...",
  "email": "merchant@example.com",
  "account_type": "merchant",
  "api_key": "pk_live_merchant789...",
  "created_at": "2025-12-11T15:30:00Z"
}
```

**⚠️ IMPORTANT**: Save your `api_key`! You'll need it for all subsequent requests. It's only shown once.

---

## Step 2: Add Your Wallet Address (30 seconds)

Tell AgentGatePay where to send payments:

```bash
curl -X POST https://api.agentgatepay.com/v1/users/wallets/add \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "wallet_address": "0xYourEthereumAddress",
    "label": "Primary Revenue Wallet"
  }'
```

**Response:**
```json
{
  "user_id": "usr_merchant123...",
  "wallet_address": "0xYourEthereumAddress",
  "label": "Primary Revenue Wallet",
  "added_at": "2025-12-11T15:31:00Z"
}
```

**Notes:**
- You can add multiple wallets for different purposes
- Payments will be sent directly to your wallet on-chain
- No KYC required - just provide an Ethereum address

---

## Step 3: Configure Webhooks (Optional, 1 minute)

Get real-time notifications when payments arrive:

```bash
curl -X POST https://api.agentgatepay.com/v1/webhooks/configure \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "merchant_wallet": "0xYourEthereumAddress",
    "webhook_url": "https://your-server.com/webhook/agentgatepay",
    "events": ["payment.received", "payment.confirmed"]
  }'
```

**Response:**
```json
{
  "webhook_id": "wh_abc123...",
  "merchant_wallet": "0xYourEthereumAddress",
  "webhook_url": "https://your-server.com/webhook/agentgatepay",
  "events": ["payment.received", "payment.confirmed"],
  "secret": "whsec_xyz789...",
  "status": "active",
  "created_at": "2025-12-11T15:32:00Z"
}
```

**Save the `secret`!** Use it to verify webhook signatures.

### Webhook Payload Example

When a payment is received, you'll get a POST request:

```json
{
  "event": "payment.received",
  "timestamp": "2025-12-11T15:35:00Z",
  "data": {
    "charge_id": "chg_abc123...",
    "tx_hash": "0xTransactionHash",
    "amount_usd": "0.01",
    "receiver_address": "0xYourEthereumAddress",
    "payer_address": "0xBuyerWalletAddress",
    "chain": "base",
    "token": "USDC",
    "paid_at": "2025-12-11T15:34:50Z",
    "block_number": 12345678,
    "confirmations": 3
  },
  "signature": "sha256_hmac_signature..."
}
```

### Verify Webhook Signature

```python
import hmac
import hashlib
import json

def verify_webhook(payload_body, signature, secret):
    """Verify AgentGatePay webhook signature"""
    expected_sig = hmac.new(
        secret.encode(),
        payload_body.encode(),
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(expected_sig, signature)

# In your webhook handler
@app.route('/webhook/agentgatepay', methods=['POST'])
def handle_webhook():
    signature = request.headers.get('X-Webhook-Signature')
    payload = request.get_data(as_text=True)

    if not verify_webhook(payload, signature, WEBHOOK_SECRET):
        return 'Invalid signature', 403

    event = json.loads(payload)

    if event['event'] == 'payment.received':
        # Process payment
        charge_id = event['data']['charge_id']
        amount = event['data']['amount_usd']
        tx_hash = event['data']['tx_hash']

        print(f"Received ${amount} payment: {tx_hash}")
        # Grant access to service, deliver product, etc.

    return 'OK', 200
```

---

## Step 4: Verify Payments (1 minute)

When you receive a payment (via webhook or API polling), verify it:

### Option A: Verify by Transaction Hash

```bash
curl https://api.agentgatepay.com/v1/payments/verify/0xTransactionHash \
  -H "x-api-key: YOUR_API_KEY"
```

**Response:**
```json
{
  "verified": true,
  "payment": {
    "tx_hash": "0xTransactionHash",
    "amount_usd": "0.01",
    "receiver_address": "0xYourEthereumAddress",
    "payer_address": "0xBuyerWalletAddress",
    "chain": "base",
    "token": "USDC",
    "paid_at": "2025-12-11T15:34:50Z",
    "block_number": 12345678,
    "confirmations": 12,
    "status": "confirmed"
  }
}
```

### Option B: List All Payments to Your Wallet

```bash
curl "https://api.agentgatepay.com/v1/payments/list?wallet=0xYourAddress&limit=10" \
  -H "x-api-key: YOUR_API_KEY"
```

**Response:**
```json
{
  "payments": [
    {
      "charge_id": "chg_abc123...",
      "tx_hash": "0xTransactionHash",
      "amount_usd": "0.01",
      "payer": "0xBuyerWalletAddress",
      "paid_at": "2025-12-11T15:34:50Z",
      "chain": "base",
      "token": "USDC"
    },
    // ... more payments
  ],
  "total": 42,
  "page": 1,
  "limit": 10
}
```

---

## Step 5: Monitor Revenue (1 minute)

View your revenue analytics:

```bash
curl https://api.agentgatepay.com/v1/merchant/revenue \
  -H "x-api-key: YOUR_API_KEY"
```

**Response:**
```json
{
  "merchant_wallet": "0xYourEthereumAddress",
  "total_revenue_usd": "127.50",
  "total_transactions": 450,
  "average_transaction_usd": "0.28",
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
  "recent_payments": [
    // ... last 10 payments
  ]
}
```

### Analytics Dashboard

Visit the web dashboard for visual analytics:
```
https://api.agentgatepay.com/dashboard?wallet=0xYourAddress
```

Features:
- Revenue charts (daily, weekly, monthly)
- Transaction volume graphs
- Chain/token distribution
- Top payers
- Real-time updates

---

## Complete Integration Example

Here's a complete Flask API server that accepts AgentGatePay payments:

```python
from flask import Flask, request, jsonify
from agentgatepay_sdk import AgentGatePayClient, verify_payment
import hmac
import hashlib

app = Flask(__name__)

# Initialize AgentGatePay client
agp_client = AgentGatePayClient(
    api_key="YOUR_API_KEY",
    base_url="https://api.agentgatepay.com"
)

MERCHANT_WALLET = "0xYourEthereumAddress"
WEBHOOK_SECRET = "whsec_xyz789..."
SERVICE_PRICE_USD = "0.01"

# In-memory storage (use database in production)
paid_users = set()

@app.route('/api/resource', methods=['GET'])
def get_resource():
    """Protected resource endpoint"""
    tx_hash = request.headers.get('X-Payment-TxHash')

    if not tx_hash:
        return jsonify({
            "error": "Payment required",
            "price_usd": SERVICE_PRICE_USD,
            "merchant_wallet": MERCHANT_WALLET,
            "chains": ["ethereum", "base", "polygon", "arbitrum"],
            "tokens": ["USDC", "USDT", "DAI"]
        }), 402  # HTTP 402 Payment Required

    # Check if already paid
    if tx_hash in paid_users:
        return jsonify({
            "status": "success",
            "resource": "Your premium content here..."
        })

    # Verify payment
    try:
        result = verify_payment(agp_client, tx_hash)

        if (result['verified'] and
            result['payment']['receiver_address'].lower() == MERCHANT_WALLET.lower() and
            float(result['payment']['amount_usd']) >= float(SERVICE_PRICE_USD)):

            # Payment confirmed! Grant access
            paid_users.add(tx_hash)

            return jsonify({
                "status": "success",
                "resource": "Your premium content here...",
                "payment_confirmed": True,
                "tx_hash": tx_hash
            })
        else:
            return jsonify({
                "error": "Payment verification failed"
            }), 402

    except Exception as e:
        return jsonify({
            "error": f"Verification error: {str(e)}"
        }), 500

@app.route('/webhook/agentgatepay', methods=['POST'])
def handle_webhook():
    """Webhook endpoint for real-time payment notifications"""
    signature = request.headers.get('X-Webhook-Signature')
    payload = request.get_data(as_text=True)

    # Verify signature
    expected_sig = hmac.new(
        WEBHOOK_SECRET.encode(),
        payload.encode(),
        hashlib.sha256
    ).hexdigest()

    if not hmac.compare_digest(expected_sig, signature):
        return 'Invalid signature', 403

    event = request.json

    if event['event'] == 'payment.received':
        charge_id = event['data']['charge_id']
        tx_hash = event['data']['tx_hash']
        amount = event['data']['amount_usd']

        print(f"Payment received: ${amount} (tx: {tx_hash})")

        # Add to paid users
        paid_users.add(tx_hash)

        # You could also:
        # - Send confirmation email
        # - Update database
        # - Trigger service delivery
        # - Log to analytics

    return 'OK', 200

@app.route('/revenue', methods=['GET'])
def get_revenue():
    """Get revenue analytics"""
    from agentgatepay_sdk.analytics import get_merchant_revenue

    revenue = get_merchant_revenue(agp_client, MERCHANT_WALLET)
    return jsonify(revenue)

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5000)
```

**Install dependencies:**
```bash
pip install flask agentgatepay-sdk
```

**Run server:**
```bash
python merchant_api.py
```

**Test it:**
```bash
# Try without payment (expect 402)
curl http://localhost:5000/api/resource

# After buyer sends payment, include tx_hash
curl http://localhost:5000/api/resource \
  -H "X-Payment-TxHash: 0xTransactionHash"
```

---

## SDK Integration

### Python

```bash
pip install agentgatepay-sdk
```

```python
from agentgatepay_sdk import AgentGatePayClient
from agentgatepay_sdk.payments import verify_payment, list_payments
from agentgatepay_sdk.analytics import get_merchant_revenue

client = AgentGatePayClient(
    api_key='YOUR_API_KEY',
    base_url='https://api.agentgatepay.com'
)

# Verify payment
payment = verify_payment(client, tx_hash='0xTransactionHash')
if payment['verified']:
    print(f"Payment confirmed: ${payment['payment']['amount_usd']}")

# List all payments to your wallet
payments = list_payments(client, wallet='0xYourAddress', limit=100)
print(f"Total payments: {payments['total']}")

# Get revenue analytics
revenue = get_merchant_revenue(client, wallet='0xYourAddress')
print(f"Total revenue: ${revenue['total_revenue_usd']}")
```

### JavaScript/TypeScript

```bash
npm install agentgatepay-sdk
```

```typescript
import { AgentGatePayClient, verifyPayment, listPayments, getMerchantRevenue } from 'agentgatepay-sdk';

const client = new AgentGatePayClient({
  apiKey: 'YOUR_API_KEY',
  baseUrl: 'https://api.agentgatepay.com'
});

// Verify payment
const payment = await verifyPayment(client, '0xTransactionHash');
if (payment.verified) {
  console.log(`Payment confirmed: $${payment.payment.amount_usd}`);
}

// List payments
const payments = await listPayments(client, {
  wallet: '0xYourAddress',
  limit: 100
});
console.log(`Total payments: ${payments.total}`);

// Get revenue
const revenue = await getMerchantRevenue(client, '0xYourAddress');
console.log(`Total revenue: $${revenue.total_revenue_usd}`);
```

See [SDK Documentation](https://github.com/AgentGatePay/agentgatepay-sdks) for full API.

---

## Integration Patterns

### Pattern 1: Webhook-Driven (Recommended)

**Best for**: Real-time applications, instant access

```
1. User sends payment on blockchain
2. AgentGatePay detects payment → sends webhook
3. Your server receives webhook → verifies signature
4. Grant access immediately
```

**Pros**: Instant, no polling, scalable
**Cons**: Requires public endpoint

### Pattern 2: On-Demand Verification

**Best for**: APIs, stateless services

```
1. User includes tx_hash in request header
2. Your server calls AgentGatePay verify API
3. If verified → grant access
```

**Pros**: Simple, no webhook setup
**Cons**: Adds latency (API call per request)

### Pattern 3: Batch Polling

**Best for**: Background processing, batch jobs

```
1. Every X minutes, poll /v1/payments/list
2. Process new payments since last check
3. Update database, grant access
```

**Pros**: No webhook endpoint needed
**Cons**: Not real-time, polling overhead

---

## Multi-Chain Support

AgentGatePay supports **4 blockchains**. You automatically receive payments on all of them to the same wallet address:

| Chain      | Settlement Speed | Notes                |
|------------|------------------|----------------------|
| Base       | 2-5 seconds      | ✅ Most popular choice |
| Polygon    | 3-8 seconds      | Fast and reliable    |
| Arbitrum   | 3-8 seconds      | Ethereum L2          |
| Ethereum   | 15-60 seconds    | Slower confirmations |

**Note:** Gas costs vary based on network congestion. Buyers typically prefer L2 chains (Base, Polygon, Arbitrum) due to lower transaction fees.

**You don't need to do anything** - just provide one Ethereum address and you'll receive payments on all chains.

**Tokens:**
- USDC (most common)
- USDT (alternative)
- DAI (decentralized)

All settle to your wallet in the native token on the chosen chain.

---

## Testing

### Test Webhook Locally

Use **ngrok** to expose your local webhook endpoint:

```bash
# Install ngrok
npm install -g ngrok

# Expose port 5000
ngrok http 5000

# Use the ngrok URL in webhook config
curl -X POST https://api.agentgatepay.com/v1/webhooks/configure \
  -H "x-api-key: YOUR_API_KEY" \
  -d '{
    "merchant_wallet": "0xYourAddress",
    "webhook_url": "https://abc123.ngrok.io/webhook/agentgatepay"
  }'
```

### Test Webhook Endpoint

AgentGatePay provides a test endpoint:

```bash
curl -X POST https://api.agentgatepay.com/v1/webhooks/test \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"webhook_id": "wh_abc123..."}'
```

This sends a test payment event to your webhook.

---

## Troubleshooting

### Webhook not receiving events

1. **Check webhook status:**
   ```bash
   curl https://api.agentgatepay.com/v1/webhooks/list \
     -H "x-api-key: YOUR_API_KEY"
   ```

2. **Test webhook:**
   ```bash
   curl -X POST https://api.agentgatepay.com/v1/webhooks/test \
     -H "x-api-key: YOUR_API_KEY" \
     -d '{"webhook_id": "wh_abc123..."}'
   ```

3. **Check firewall**: Ensure your server accepts POST requests from AgentGatePay IPs

4. **Return 200 OK**: Your webhook must return HTTP 200, or we'll retry (up to 3 times)

### Payment verification fails

- **Check wallet address**: Must match exactly (case-insensitive)
- **Check amount**: Must be >= expected price
- **Check token decimals**: USDC/USDT = 6, DAI = 18
- **Wait for confirmations**: Can take 3-60 seconds depending on chain

### Revenue doesn't match

- **Commission deducted**: AgentGatePay takes 0.5% commission
- **Multiple wallets**: Check all wallet addresses in your account
- **Blockchain explorer**: Verify on Etherscan/Basescan

---

## Rate Limits

Rate limits are enforced per-endpoint:
- `/mandates/issue`: **20 requests/minute**
- `/x402/resource`: **60 requests/minute**
- Most endpoints: **60 requests/minute**

Headers show limits:
```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 55
X-RateLimit-Reset: 1702345678
```

---

## Security Best Practices

1. **Verify webhook signatures** - Always check `X-Webhook-Signature` header
2. **Use HTTPS** - Never expose webhook endpoints over HTTP
3. **Rate limit your API** - Prevent abuse of your endpoints
4. **Double-check amounts** - Verify payment amount >= expected price
5. **Store API keys securely** - Use environment variables, never commit to git
6. **Monitor failed webhooks** - Set up alerts for delivery failures
7. **Log all payments** - Keep audit trail for compliance

---

## Next Steps

You've successfully:
- Created a merchant account
- Added your wallet address
- Configured webhooks
- Verified payments
- Checked revenue analytics

**Now explore:**
- [Architecture Diagrams](../architecture/diagrams.md) - Understand payment flows and security
- [Examples Repository](https://github.com/AgentGatePay/agentgatepay-examples) - Production code (n8n workflows)
- [API Reference](../api/endpoints-reference.md) - Complete documentation
- [Official SDKs](https://github.com/AgentGatePay/agentgatepay-sdks) - JavaScript & Python libraries
- [MCP Tools](../mcp/tools-reference.md) - AI agent integration

---

## Support

Need help?
- [Complete Documentation](../)
- **Email Support**: support@agentgatepay.com - Bug reports, feature requests, integration help
- [Examples](https://github.com/AgentGatePay/agentgatepay-examples)

---

**Happy selling!**

**AgentGatePay is in beta** - we're actively improving based on developer feedback. Your input directly shapes our roadmap and priorities. Thank you for being an early adopter!
