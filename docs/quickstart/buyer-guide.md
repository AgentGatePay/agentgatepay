# Buyer Quickstart Guide

**Complete this guide in 5 minutes** to send your first payment using AgentGatePay.

This guide is for **agents** (buyers) who want to pay for services or resources. If you're a merchant receiving payments, see the [Merchant Quickstart Guide](merchant-guide.md).

**Beta Version** - AgentGatePay is actively in development. We welcome your feedback!

---

## What You'll Need

- Email address
- Ethereum wallet with USDC/USDT/DAI (on Ethereum, Base, Polygon, or Arbitrum)
- Your wallet's private key (for signing transactions)
- ~5 minutes

---

## Step 1: Create Your Account (30 seconds)

Create an account to get higher rate limits and access to analytics:

```bash
curl -X POST https://api.agentgatepay.com/v1/users/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "your-email@example.com",
    "password": "your-secure-password",
    "account_type": "agent"
  }'
```

**Response:**
```json
{
  "user_id": "usr_abc123...",
  "email": "your-email@example.com",
  "account_type": "agent",
  "api_key": "pk_live_xyz789...",
  "created_at": "2025-12-11T15:30:00Z"
}
```

**⚠️ IMPORTANT**: Save your `api_key`! You'll need it for all subsequent requests. It's only shown once.

---

## Step 2: Issue a Mandate (1 minute)

**What is a mandate?**

A mandate is a cryptographic authorization that:
- Sets a spending budget (e.g., $10)
- Defines what the agent can pay for (scope)
- Has an expiration time (TTL)
- Tracks spending automatically

Think of it as a "prepaid card" for your AI agent with built-in budget controls.

**Issue your first mandate:**

```bash
curl -X POST https://api.agentgatepay.com/mandates/issue \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "0xYourWalletAddress",
    "budget_usd": "10.00",
    "scope": "research",
    "ttl_minutes": 1440
  }'
```

**Parameters:**
- `subject`: Your wallet address (the payer)
- `budget_usd`: Maximum spending limit in USD (e.g., "10.00")
- `scope`: What the agent can pay for (e.g., "research", "api_access", "compute")
- `ttl_minutes`: Time to live in minutes (1440 = 24 hours, 43200 = 30 days)

**Response:**
```json
{
  "mandate_token": "eyJhbGc...long_token_here",
  "mandate_id": "mnd_abc123...",
  "subject": "0xYourWalletAddress",
  "budget_usd": "10.00",
  "scope": "research",
  "expires_at": "2025-12-12T15:30:00Z",
  "status": "active"
}
```

**Save your `mandate_token`!** You'll include this in payment requests.

---

## Step 3: Make Your First Payment (2 minutes)

Now you're ready to pay for a protected resource. AgentGatePay uses the **x402 protocol** (HTTP 402 Payment Required).

### 3.1: Request the Resource (Expect 402)

First, try accessing the resource without payment:

```bash
curl -i https://api.agentgatepay.com/x402/resource \
  -H "x-agent-id: 0xYourWalletAddress" \
  -H "x-mandate: YOUR_MANDATE_TOKEN"
```

**Response (402 Payment Required):**
```
HTTP/1.1 402 Payment Required

{
  "status": "payment_required",
  "message": "Payment of 0.01 USD required to access this resource",
  "price_usd": "0.01",
  "receiver_address": "0xMerchantWalletAddress",
  "supported_chains": ["ethereum", "base", "polygon", "arbitrum"],
  "supported_tokens": ["USDC", "USDT", "DAI"]
}
```

### 3.2: Send Blockchain Transaction

The gateway tells you:
- How much to pay (`price_usd`)
- Where to send it (`receiver_address`)
- Which chains/tokens are supported

Now send the transaction on your chosen chain (Base recommended for speed):

```python
# Example using ethers (JavaScript) or web3.py (Python)
from web3 import Web3
from eth_account import Account

# Connect to blockchain
w3 = Web3(Web3.HTTPProvider('https://mainnet.base.org'))

# Your wallet
private_key = 'YOUR_PRIVATE_KEY'
account = Account.from_key(private_key)

# USDC contract on Base
usdc_address = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913'
usdc_abi = [...] # ERC20 ABI

usdc = w3.eth.contract(address=usdc_address, abi=usdc_abi)

# Send 0.01 USD = 10000 USDC tokens (6 decimals)
amount = 10000  # 0.01 USDC

tx = usdc.functions.transfer(
    '0xMerchantWalletAddress',  # receiver
    amount
).build_transaction({
    'from': account.address,
    'nonce': w3.eth.get_transaction_count(account.address),
    'gas': 100000,
    'gasPrice': w3.eth.gas_price
})

signed_tx = account.sign_transaction(tx)
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)

print(f"Transaction hash: {tx_hash.hex()}")
```

**⚠️ Important: Decimal Handling**
- **USDC & USDT**: 6 decimals → $1.00 = 1,000,000 tokens
- **DAI**: 18 decimals → $1.00 = 1,000,000,000,000,000,000 tokens

### 3.3: Submit Payment Proof

Once your transaction is sent, submit the tx_hash to the gateway:

```bash
curl -i https://api.agentgatepay.com/x402/resource \
  -H "x-agent-id: 0xYourWalletAddress" \
  -H "x-mandate: YOUR_MANDATE_TOKEN" \
  -H "x-payment: {\"tx_hash\":\"0xYourTransactionHash\",\"chain\":\"base\",\"token\":\"USDC\"}"
```

**Response (200 OK - Access Granted):**
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

**Success!** You've accessed the resource and your mandate budget was automatically decreased.

---

## Step 4: Monitor Your Spending (1 minute)

View your spending analytics:

```bash
curl https://api.agentgatepay.com/v1/analytics/me \
  -H "x-api-key: YOUR_API_KEY"
```

**Response:**
```json
{
  "role": "agent",
  "total_spent_usd": "0.01",
  "total_transactions": 1,
  "average_transaction_usd": "0.01",
  "active_mandates": 1,
  "recent_payments": [
    {
      "charge_id": "chg_abc123...",
      "amount_usd": "0.01",
      "receiver": "0xMerchantWalletAddress",
      "paid_at": "2025-12-11T15:32:00Z",
      "tx_hash": "0xYourTransactionHash",
      "chain": "base"
    }
  ]
}
```

---

## Complete Python Example

Here's a complete working example using the **AgentGatePay Python SDK**:

```python
from agentgatepay_sdk import AgentGatePayClient
from agentgatepay_sdk.mandates import issue_mandate
from agentgatepay_sdk.payments import request_resource, submit_payment
from web3 import Web3
from eth_account import Account

# Initialize client
client = AgentGatePayClient(
    api_key="YOUR_API_KEY",
    base_url="https://api.agentgatepay.com"
)

# Your wallet
private_key = "YOUR_PRIVATE_KEY"
account = Account.from_key(private_key)
wallet_address = account.address

# Step 1: Issue mandate
mandate = issue_mandate(
    client=client,
    subject=wallet_address,
    budget_usd="10.00",
    scope="research",
    ttl_minutes=1440
)
print(f"Mandate issued: {mandate['mandate_token']}")

# Step 2: Request resource (get payment details)
payment_required = request_resource(
    client=client,
    agent_id=wallet_address,
    mandate_token=mandate['mandate_token']
)
print(f"Payment required: ${payment_required['price_usd']} to {payment_required['receiver_address']}")

# Step 3: Send blockchain transaction (Base USDC)
w3 = Web3(Web3.HTTPProvider('https://mainnet.base.org'))
usdc_address = '0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913'

# Calculate amount (USDC has 6 decimals)
amount_usd = float(payment_required['price_usd'])
amount_tokens = int(amount_usd * 1_000_000)  # 6 decimals

# Send transaction
usdc = w3.eth.contract(address=usdc_address, abi=ERC20_ABI)
tx = usdc.functions.transfer(
    payment_required['receiver_address'],
    amount_tokens
).build_transaction({
    'from': account.address,
    'nonce': w3.eth.get_transaction_count(account.address),
    'gas': 100000,
    'gasPrice': w3.eth.gas_price
})

signed_tx = account.sign_transaction(tx)
tx_hash = w3.eth.send_raw_transaction(signed_tx.rawTransaction)
print(f"Transaction sent: {tx_hash.hex()}")

# Step 4: Submit payment proof
result = submit_payment(
    client=client,
    agent_id=wallet_address,
    mandate_token=mandate['mandate_token'],
    tx_hash=tx_hash.hex(),
    chain="base",
    token="USDC"
)
print(f"Access granted! Resource: {result['resource_data']}")
print(f"Remaining budget: ${result['mandate_remaining_budget']}")
```

**Install SDK:**
```bash
pip install agentgatepay-sdk
```

---

## Chain & Token Selection

AgentGatePay supports **4 chains** and **3 tokens**. Choose based on your needs:

| Chain      | Settlement Speed | Best For                |
|------------|------------------|-------------------------|
| **Base**   | 2-5 seconds      | ✅ **Recommended** (fast + low cost) |
| Polygon    | 3-8 seconds      | Balance of speed/cost   |
| Arbitrum   | 3-8 seconds      | Ethereum L2 ecosystem   |
| Ethereum   | 15-60 seconds    | Only if required        |

**Note:** Gas costs vary based on network congestion. L2 chains (Base, Polygon, Arbitrum) typically have significantly lower fees than Ethereum mainnet.

**Tokens:**
- **USDC**: Most widely supported
- **USDT**: Alternative stablecoin
- **DAI**: Decentralized stablecoin (18 decimals!)

**Default**: If you don't specify, the gateway uses Base + USDC.

**Specify chain/token:**
```bash
curl https://api.agentgatepay.com/x402/resource?chain=polygon&token=USDT \
  -H "x-agent-id: 0xYour Address" \
  -H "x-mandate: YOUR_MANDATE"
```

---

## SDK Integration

### JavaScript/TypeScript

```bash
npm install agentgatepay-sdk
```

```typescript
import { AgentGatePayClient, issueMand ate, submitPayment } from 'agentgatepay-sdk';

const client = new AgentGatePayClient({
  apiKey: 'YOUR_API_KEY',
  baseUrl: 'https://api.agentgatepay.com'
});

// Issue mandate
const mandate = await issueMandate(client, {
  subject: walletAddress,
  budgetUsd: '10.00',
  scope: 'research',
  ttlMinutes: 1440
});

// Make payment (handles blockchain transaction internally)
const result = await submitPayment(client, {
  agentId: walletAddress,
  mandateToken: mandate.mandate_token,
  // ... transaction details
});
```

See [SDK Documentation](https://github.com/AgentGatePay/agentgatepay-sdks) for full API.

---

## MCP Integration (For AI Agents)

If you're using **Claude Desktop, OpenAI, or any MCP-compatible platform**, you can use AgentGatePay's MCP tools instead of the REST API:

```javascript
// Claude Desktop config
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

Then your AI agent can use natural language:
```
"Issue a mandate with $10 budget for research, then purchase data from provider X"
```

See [MCP Tools Reference](../mcp/tools-reference.md) for details.

---

## Troubleshooting

### Error: "Mandate token invalid or expired"
- **Solution**: Issue a new mandate. Mandates expire based on `ttl_minutes`.
- Check mandate status: `GET /mandates/verify` with your token.

### Error: "Insufficient mandate budget"
- **Solution**: Issue a new mandate with a higher budget or check remaining budget:
  ```bash
  curl -X POST https://api.agentgatepay.com/mandates/verify \
    -H "Content-Type: application/json" \
    -d '{"mandate_token": "YOUR_TOKEN"}'
  ```

### Error: "Transaction verification failed"
- **Causes**: Wrong amount, wrong recipient, transaction pending
- **Solution**:
  - Check tx_hash on blockchain explorer (Etherscan, Basescan)
  - Ensure amount matches exactly (including decimals!)
  - Wait for transaction confirmation (can take 15-60 sec on Ethereum)

### Error: "Nonce already used (replay attack blocked)"
- **Cause**: You tried to submit the same transaction twice
- **Solution**: This is security working correctly. Each transaction can only be used once.

### Payment Stuck in "Pending"
- **Ethereum**: Can take 15-60 seconds
- **Other chains**: Usually 3-8 seconds
- **Solution**: Wait for blockchain confirmation, then resubmit payment proof

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

## Next Steps

You've successfully:
- Created an account
- Issued a mandate
- Made a payment
- Accessed a protected resource

**Now explore:**
- [Architecture Diagrams](../architecture/diagrams.md) - Understand payment flows and security
- [MCP Integration](../mcp/tools-reference.md) - Connect your AI agents with 15 tools
- [Examples Repository](https://github.com/AgentGatePay/agentgatepay-examples) - Production-ready code
- [API Reference](../api/endpoints-reference.md) - Complete endpoint documentation
- [Official SDKs](https://github.com/AgentGatePay/agentgatepay-sdks) - JavaScript & Python libraries

---

## Support

Need help?
- [Complete Documentation](../)
- **Email Support**: support@agentgatepay.com - Bug reports, feature requests, integration help
- [Examples](https://github.com/AgentGatePay/agentgatepay-examples)

---

**Happy building!**

**AgentGatePay is in beta** - we're actively improving based on developer feedback. Your input directly shapes our roadmap and priorities. Thank you for being an early adopter!
