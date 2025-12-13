# AgentGatePay

**Payment Gateway for the Agent Economy**

AgentGatePay enables AI agents to autonomously send and receive payments using blockchain technology. Designed for agent-to-agent and agent-to-merchant transactions with enterprise-grade security.

**Beta Release** | **Multi-Chain** (Ethereum, Base, Polygon, Arbitrum) | **Multi-Token** (USDC, USDT, DAI)

> **📊 New to AgentGatePay?** Start with our **[Visual Introduction (PDF)](docs/overview/AgentGatePay-Overview-Presentation.pdf)** - a 9-slide visual guide explaining everything in minutes!

---

## ⚠️ IMPORTANT DISCLAIMER

**AgentGatePay is currently in BETA.** By using this service, you acknowledge and accept:

- **Service Availability:** The service may be unavailable, suspended, or permanently shut down at any time without prior notice. No SLA or uptime guarantees.
- **Data Loss Risk:** All data may be lost at any time without recovery. Users are solely responsible for maintaining independent backups of transaction records.
- **No Liability:** AgentGatePay and its operators are NOT LIABLE for any direct, indirect, incidental, or consequential damages, including but not limited to: lost cryptocurrency, failed transactions, service interruptions, or loss of revenue.
- **Financial Risk:** Blockchain transactions are irreversible. Use at your own risk. Users are solely responsible for securing their private keys, API keys, and compliance with applicable laws.
- **No Warranty:** This service is provided "AS IS" without warranties of any kind.

**📄 Read the full [DISCLAIMER.md](DISCLAIMER.md) before using AgentGatePay.**

**BY USING THIS SERVICE, YOU AGREE TO THESE TERMS. IF YOU DO NOT AGREE, DO NOT USE AGENTGATEPAY.**

---

## Quick Links

- **[📊 Visual Introduction (PDF)](docs/overview/AgentGatePay-Overview-Presentation.pdf)** - 9-slide visual overview (start here!)
- [Documentation](docs/) | [API Reference](docs/api/endpoints-reference.md)
- [Buyer Quickstart](docs/quickstart/buyer-guide.md) | [Seller Quickstart](docs/quickstart/merchant-guide.md)
- [Examples](https://github.com/AgentGatePay/agentgatepay-examples) | [SDKs](https://github.com/AgentGatePay/agentgatepay-sdks)
- [Transaction Signing Service](https://github.com/AgentGatePay/TX)

---

## What is AgentGatePay?

AgentGatePay is a **payment infrastructure layer** for AI agents. It enables autonomous economic interactions where agents can:

- **Send payments** to access services and resources
- **Receive payments** for providing services
- **Manage budgets** with cryptographic mandates
- **Track spending** with real-time analytics
- **Verify transactions** instantly across 4 blockchains

Built on **blockchain technology** for permissionless, programmable, and global payments.

---

## The Problem We're Solving

### The Agent Economy is Stuck

AI agents are becoming increasingly capable of autonomous decision-making, but there's a critical bottleneck: **they can't pay for things**.

Today's AI agents face a fundamental problem:

**Agents need to buy services autonomously, but traditional payment systems won't let them.**

### Why Traditional Payments Don't Work for Agents

**Credit Cards & Traditional Payment Processors:**
- Require human identity verification (SSN, passport, bank account)
- Need credit history and credit checks
- Designed for humans, not autonomous software
- Slow approval processes (days to weeks)
- High fees for small transactions ($0.30 + 2.9%)
- No programmable budget controls

**Bank Transfers & Wire Payments:**
- Require bank accounts (agents can't open bank accounts)
- Slow settlement (1-3 business days)
- High minimum amounts (usually $10+)
- Cross-border transfers are expensive and slow
- No real-time verification

**The Result:** AI agents can't be truly autonomous because they need humans to handle every payment.

### What This Blocks

Without payment infrastructure, the agent economy can't exist:

1. **No Autonomous Agent Services**
   - Agents can't buy API access, data, or compute without human approval
   - Every transaction requires manual intervention
   - Agents can't operate 24/7 independently

2. **No Agent-to-Agent Commerce**
   - Agents can't pay other agents for services
   - No marketplace for agent capabilities
   - No specialization or division of labor among agents

3. **No Micropayments**
   - Traditional payments cost $0.30 minimum per transaction
   - Impossible to charge $0.01 for a single API call
   - Pay-per-use models don't work economically

4. **No Budget Control**
   - No way to give an agent a spending limit
   - All-or-nothing: either full access or no access
   - No scope restrictions (what can be purchased)
   - No automatic tracking of agent spending

5. **No Global Access**
   - Cross-border payments take days
   - High currency conversion fees
   - Geographic restrictions block many agents
   - Different banking systems don't interoperate

**The fundamental issue:** Traditional finance was built for humans with identities, not for autonomous software agents.

---

## How AgentGatePay Solves This

We built the **first payment gateway specifically designed for AI agents**, addressing each problem directly:

### 1. Permissionless Access (No Identity Required)

**Blockchain-based payments** eliminate identity requirements:
- No SSN, passport, or credit check needed
- No bank account required
- Just an Ethereum wallet address (generated in seconds)
- Works for any agent, anywhere in the world
- Instant activation

### 2. Autonomous Operation

**AP2 Mandate Protocol** enables true agent autonomy:
- Set spending budgets in advance ($10, $100, $1000)
- Define scope (what agent can purchase: "research", "compute", "api_access")
- Time-bound authorizations (expire after 24h, 7d, 30d)
- Cryptographic signatures (Ed25519) - no human in the loop
- Automatic budget tracking and deduction

**Example:** Give agent $100 budget for "research" valid for 7 days → agent operates autonomously within limits.

### 3. Micropayments That Actually Work

**Blockchain settlement** makes $0.01 payments economically viable:
- No minimum transaction amount
- Settlement cost: ~$0.0001 to $0.002 (depending on chain)
- Pay $0.01 for a single API call - actually profitable
- Enables pay-per-use pricing models

**Comparison:**
- Traditional processors: $0.30 + 2.9% = $0.30 minimum cost → $0.01 payment loses $0.29
- AgentGatePay: $0.0001 fee → $0.01 payment keeps $0.0099 (99% margin)

### 4. Instant Global Settlement

**Multi-chain blockchain support:**
- **Base**: 2-5 second confirmation
- **Polygon**: 3-8 seconds
- **Arbitrum**: 3-8 seconds
- **Ethereum**: 15-60 seconds (for larger amounts)
- Works across all countries instantly
- No currency conversion needed (USD stablecoins)
- 24/7 operation (no banking hours)

### 5. Enterprise-Grade Security for Agents

**AIF (Agent Interaction Firewall)** - the first security system built for AI agents:
- **Rate limiting**: Prevent abuse (distributed across all servers)
- **Replay protection**: Each transaction can only be used once
- **Reputation scoring**: Track agent behavior (0-200 score)
- **Pattern detection**: Identify suspicious spending patterns
- **Budget enforcement**: Hard limits prevent overspending

**Traditional systems** protect against human fraud. **AIF protects agents from malicious agents.**

### 6. Two Novel Protocols

We didn't just build a payment gateway - we created two new protocols for agent payments:

**AP2 (Agent Payment Protocol v2):**
- Cryptographic budget mandates
- Scope-based authorization
- Automatic spending tracking
- Time-bound permissions

**x402 (HTTP 402 Payment Required):**
- Standardized payment-required responses
- Multi-chain payment requests
- On-chain verification
- Atomic settlement

These protocols are **open** and can be implemented by anyone building agent infrastructure.

---

## Why AgentGatePay

### What Makes Us Different

**1. Built Specifically for Agents**
- Not a modified human payment system
- Designed from scratch for autonomous software
- APIs optimized for agent workflows
- MCP tools for Claude, OpenAI, and other AI platforms

**2. Production-Ready Today**
- Real transactions happening now on mainnet
- Multi-chain support (Ethereum, Base, Polygon, Arbitrum)
- Multi-token support (USDC, USDT, DAI)
- Enterprise security (rate limiting, replay protection, reputation)
- Real-time analytics and webhooks
- Complete SDKs (JavaScript, Python) and examples

**3. Developer-First**
- 5-minute integration (seriously - check the quickstart guides)
- Comprehensive documentation with real examples
- SDKs for popular languages
- MCP integration for AI platforms
- Active support and rapid iteration

**4. Our Effort**
We've spent months building:
- Two novel protocols (AP2, x402)
- Multi-chain blockchain integration
- Security infrastructure for agent protection
- Complete API with 24+ endpoints
- 15 MCP tools for AI platforms
- Python and JavaScript SDKs
- 10+ production-ready examples
- Comprehensive documentation

**Beta status** means we're actively improving based on real-world usage. We're responsive, we iterate fast, and we're committed to solving this problem.

---

## How This Changes the Future

### The Agent Economy We're Enabling

**Today (without AgentGatePay):**
```
Human → Manually pay for API
     → Agent uses API
     → Agent completes task
     → Human pays again for next task
```

**Tomorrow (with AgentGatePay):**
```
Human → Give agent $100 budget
     → Agent autonomously:
        - Buys API access ($0.50)
        - Purchases data ($2.00)
        - Pays for compute ($5.00)
        - Uses specialized AI service ($1.50)
        - Completes entire project
     → Human reviews results
     → Budget remaining: $91.00
```

### New Possibilities

**1. Autonomous Agent Services**
- Agents buy what they need, when they need it
- No human approval required for each transaction
- 24/7 operation without intervention
- Real-time adaptation to changing requirements

**2. Agent Marketplaces**
- Specialized agents sell capabilities to other agents
- Instant payment settlement
- Reputation-based pricing
- Emergent agent economy

**3. Micropayment Business Models**
- $0.001 per API call becomes profitable
- Pay-per-use instead of monthly subscriptions
- Fair pricing based on actual usage
- Lower barrier to entry for new services

**4. Multi-Agent Collaboration**
- Project manager agent delegates to specialist agents
- Each specialist bills for their work
- Automatic budget tracking and reconciliation
- Complex workflows with many participants

**5. Global Agent Workforce**
- Agents from anywhere can participate
- Instant cross-border payments
- No geographic restrictions
- No currency conversion needed

### The Economic Impact

**We believe the agent economy will be massive:**
- Billions of microtransactions per day
- Agents as first-class economic participants
- New markets that don't exist today
- Economic activity at machine speed (not human speed)

**Traditional payment systems can't support this.** They're too slow, too expensive, too restricted to humans.

**AgentGatePay is the infrastructure layer that makes it possible.**

---

## Integration Guide

### I Want to BUY Services (Agent/Buyer)

**What you need:**
- Ethereum wallet with USDC/USDT/DAI
- Wallet private key (for signing blockchain transactions)
- AgentGatePay account (create via `/v1/users/signup`)

**Choose your integration method:**

| Method | Best For | Setup Time |
|--------|----------|------------|
| **SDK** (Recommended) | Production apps, type safety | 10 minutes |
| **REST API** | Any language, full control | 15 minutes |
| **MCP Tools** | Claude Desktop, OpenAI agents | 5 minutes |

**Core Payment Flow:**

```
1. Create Mandate   → POST /mandates/issue → get budget authorization
2. Request Resource → GET /x402/resource → receives 402 + payment details
3. Sign Transaction → Send USDC to seller's wallet → get tx_hash
4. Submit Proof     → GET /x402/resource (with tx_hash) → get resource
5. Track Spending   → GET /v1/analytics/me → view budget remaining
```

**Endpoints you'll use:**

**Setup (one-time):**
1. `POST /v1/users/signup` - Create account → get API key
2. `POST /mandates/issue` - Set budget ($10) → get mandate token

**For each purchase:**
3. `GET /x402/resource` - Request resource → receive price & wallet address
4. Send blockchain transaction → USDC transfer to seller
5. `GET /x402/resource` - Submit tx_hash → receive resource instantly

**Monitoring:**
6. `GET /v1/analytics/me` - Check spending & remaining budget

**Quick links:**
- [Buyer Guide](docs/quickstart/buyer-guide.md) - Step-by-step tutorial
- [Python SDK](https://github.com/AgentGatePay/agentgatepay-sdks/tree/main/python) - `pip install agentgatepay-sdk`
- [JavaScript SDK](https://github.com/AgentGatePay/agentgatepay-sdks/tree/main/javascript) - `npm install agentgatepay-sdk`
- [Python Examples](https://github.com/AgentGatePay/agentgatepay-examples/tree/main/python/langchain-payment-agent) - 10 complete examples

---

### I Want to SELL Services (Agent/Merchant)

**What you need:**
- Ethereum wallet address (to receive payments)
- AgentGatePay account (create via `/v1/users/signup`)
- **No webhooks required** - verify payments with simple API calls

**Choose your integration method:**

| Method | Best For | Setup Time |
|--------|----------|------------|
| **SDK** (Recommended) | Production apps, type safety | 10 minutes |
| **REST API** | Any language, full control | 15 minutes |
| **MCP Tools** | Claude Desktop, OpenAI agents | 5 minutes |

**Core Payment Flow (No webhooks needed):**

```
1. Buyer Request    → GET /resource?resource_id=X
2. Return 402       → Your API: "Price is $0.01, send to wallet 0xABC..."
3. Buyer Pays       → Blockchain transaction happens
4. Buyer Submits    → GET /resource with x-payment header (tx_hash)
5. You Verify       → Call: GET /v1/payments/verify/{tx_hash}
6. Deliver Resource → Return resource data if verified
```

**Endpoints you'll use:**

**Setup (one-time):**
1. `POST /v1/users/signup` - Create account → get API key
2. `POST /v1/users/wallets/add` - Register your wallet → start receiving

**For each sale:**
3. `GET /v1/payments/verify/{tx_hash}` - Verify buyer paid → returns true/false
4. Return resource data if verified → sale complete

**Optional (for automation):**
5. `POST /v1/webhooks/configure` - Auto-notify on payments (production optimization)
6. `GET /v1/payments/list` - Batch check multiple payments
7. `GET /v1/merchant/revenue` - View earnings dashboard

**Quick links:**
- [Seller Guide](docs/quickstart/merchant-guide.md) - Step-by-step tutorial
- [Python SDK](https://github.com/AgentGatePay/agentgatepay-sdks/tree/main/python) - `pip install agentgatepay-sdk`
- [JavaScript SDK](https://github.com/AgentGatePay/agentgatepay-sdks/tree/main/javascript) - `npm install agentgatepay-sdk`
- [n8n Workflows](https://github.com/AgentGatePay/agentgatepay-examples/tree/main/n8n) - Zero-code automation

---

### Integration Decision Tree

```
┌─ I'm building an AI agent ─────────────────────────┐
│                                                     │
│  ├─ Using Claude Desktop? ───→ Use MCP Tools       │
│  │                             (5 min setup)        │
│  │                                                  │
│  ├─ Using LangChain/AutoGPT? ──→ Use SDK           │
│  │                              (Python/JS)         │
│  │                                                  │
│  └─ Custom framework? ─────────→ Use REST API      │
│                                  (Any language)     │
└─────────────────────────────────────────────────────┘

┌─ I'm building a web app ───────────────────────────┐
│                                                     │
│  ├─ Node.js/TypeScript? ───────→ Use JS SDK        │
│  │                              (npm install)       │
│  │                                                  │
│  ├─ Python/Django/Flask? ───────→ Use Python SDK   │
│  │                              (pip install)       │
│  │                                                  │
│  └─ Other language? ────────────→ Use REST API     │
│                                  (curl/http client) │
└─────────────────────────────────────────────────────┘

┌─ I want zero-code automation ──────────────────────┐
│                                                     │
│  └─ Use n8n workflows ──────────→ Import JSON      │
│                                   (drag & drop)     │
└─────────────────────────────────────────────────────┘
```

---

## Key Features

### AI-Native Design
- **REST API** + **MCP Tools** (15 tools for Claude, OpenAI, and more)
- Designed for autonomous agent workflows
- Framework integrations: LangChain, AutoGPT, n8n, CrewAI

### Budget Mandates (AP2)
- **Spending limits** - Set maximum budgets in USD
- **Scope control** - Define what agents can pay for
- **Cryptographic authorization** - Ed25519 signatures
- **Automatic tracking** - Real-time budget deduction

### Security & Protection (AIF)
- **Rate limiting** - Distributed protection against abuse
- **Replay protection** - Nonce-based duplicate prevention
- **Reputation scoring** - Agent behavior tracking (0-200)
- **Pattern detection** - Fraud prevention

### Multi-Chain & Multi-Token
- **Chains**: Ethereum, Base, Polygon, Arbitrum
- **Tokens**: USDC, USDT, DAI
- **Dynamic routing** - Query parameters for chain/token selection
- **Fast settlement** - 3-56 seconds depending on chain/amount

### Analytics & Monitoring
- **Real-time dashboards** - Spending and revenue tracking
- **Webhooks** - Instant payment notifications
- **Audit logs** - Complete transaction history
- **Revenue analytics** - Merchant earnings reports

### Enterprise-Grade Security
- **Multi-layer security** - Multiple protection layers
- **Global availability** - Distributed edge network
- **High uptime** - Health monitoring and redundancy
- **Secure by design** - Built for production workloads

---

## How It Works

### Agent-to-Agent Payment Flow

```
1. Issue Mandate     → Agent creates budget authorization
2. Request Resource  → Agent requests paid content (402 Payment Required)
3. Submit Payment    → Agent sends blockchain transaction
4. Access Resource   → Gateway verifies & grants access (200 OK)
```

### Agent-to-Merchant Payment Flow

```
1. Agent Sends Payment    → Direct blockchain transaction
2. Gateway Verifies       → On-chain verification
3. Webhook Notification   → Merchant receives instant alert
4. Merchant Delivers      → Service/resource provided to agent
```

---

## Legal & Compliance

### AgentGatePay is a Non-Custodial Service

**You maintain full control at all times:**
- **Your wallet keys** - You control your Ethereum wallet private keys (we never have access)
- **Your API keys** - You manage your AgentGatePay API keys (for authentication only)
- **Your funds** - All cryptocurrency remains in your wallet until you choose to send it
- **Your transactions** - You sign and submit all blockchain transactions directly

**What AgentGatePay provides:**
- **Transaction verification** - We verify on-chain that payments occurred correctly
- **Security layer** - Rate limiting, replay protection, reputation scoring (AIF)
- **Budget tracking** - We track mandate spending limits (AP2 protocol)
- **Audit logging** - Complete transaction history and analytics

**AgentGatePay is the middle man that:**
1. Verifies blockchain transactions between buyers and sellers
2. Provides security infrastructure (anti-fraud, rate limits)
3. Offers audit logs and analytics dashboards
4. **Never holds, controls, or has access to your funds**

### Compliance Responsibility

**You are responsible for:**
- Maintaining security of your wallet private keys
- Complying with applicable laws in your jurisdiction
- Tax reporting for cryptocurrency transactions (if required)
- Understanding regulations that may apply to your use case

**AgentGatePay does not:**
- Hold or custody funds (non-custodial)
- Provide investment, legal, tax, or financial advice
- Control or have access to your wallet or keys
- Act as a money transmitter (you control all transactions)

**For enterprise compliance questions**, consult with your legal and compliance teams about blockchain payment regulations in your jurisdiction.

**This service is legal** because it's non-custodial - users control their own keys, wallets, and transactions. We simply verify and provide security infrastructure.

---

## Quick Start

### For Buyers (Send Payments)

**5-minute setup** to send your first payment:

```bash
# 1. Create account
curl -X POST https://api.agentgatepay.com/v1/users/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "your-email@example.com",
    "password": "secure-password",
    "account_type": "agent"
  }'

# 2. Issue mandate (use API key from signup response)
curl -X POST https://api.agentgatepay.com/mandates/issue \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "subject": "your-wallet-address",
    "budget_usd": "10.00",
    "scope": "research",
    "ttl_minutes": 1440
  }'

# 3. Make payment (see full example in buyer guide)
```

**[Complete Buyer Guide](docs/quickstart/buyer-guide.md)**

### For Sellers (Receive Payments)

**For agents or merchants receiving payments - 5-minute setup:**

```bash
# 1. Create merchant account
curl -X POST https://api.agentgatepay.com/v1/users/signup \
  -H "Content-Type: application/json" \
  -d '{
    "email": "merchant@example.com",
    "password": "secure-password",
    "account_type": "merchant"
  }'

# 2. Add your wallet address
curl -X POST https://api.agentgatepay.com/v1/users/wallets/add \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"wallet_address": "0xYourEthereumAddress"}'

# 3. Configure webhook (optional)
curl -X POST https://api.agentgatepay.com/v1/webhooks/configure \
  -H "x-api-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "merchant_wallet": "0xYourEthereumAddress",
    "webhook_url": "https://your-server.com/webhook"
  }'

# 4. Verify payments
curl https://api.agentgatepay.com/v1/payments/verify/TX_HASH \
  -H "x-api-key: YOUR_API_KEY"
```

**[Complete Seller Guide](docs/quickstart/merchant-guide.md)** (for agents & merchants)

### For AI Agents (MCP Integration)

**Connect Claude Desktop, OpenAI, or any MCP-compatible platform:**

```bash
# For Claude Desktop - add to config:
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

# For OpenAI - use unified JSON-RPC endpoint:
POST https://mcp.agentgatepay.com/
```

See **[MCP Tools Reference](docs/mcp/tools-reference.md)** for complete integration guide.

---

## Use Cases

### Autonomous AI Agents
AI agents that purchase API access, data, or compute resources without human intervention.

**Example**: Research agent with $100 budget, autonomously purchases data from multiple providers.

### Agent Marketplaces
Platforms where agents buy and sell services with instant settlement.

**Example**: AI service marketplace with automatic payment verification and instant access.

### Micropayments for AI Services
Pay-per-use pricing for AI services with blockchain settlement.

**Example**: $0.01 per query to a specialized AI model, settled in seconds.

### Multi-Agent Systems
Teams of agents collaborating and transacting with budget controls.

**Example**: Project management agent delegates tasks to specialist agents, tracking budget in real-time.

---

## Core Protocols

AgentGatePay is built on two innovative payment protocols:

### AP2 (Agent Payment Protocol v2)
Cryptographic authorization protocol for agent payments with budget controls. Enables agents to operate with pre-authorized spending limits using Ed25519 signatures and JWT-like tokens.

**Key capabilities:**
- Budget mandates with USD tracking
- Scope-based permissions
- Time-bound authorizations (TTL)
- Automatic budget deduction

### x402 (HTTP 402 Payment Required)
Extension of the HTTP protocol for payment-required resources. Implements the standardHTTP 402 status code for blockchain-based microtransactions.

**Key capabilities:**
- Multi-chain payment requests
- On-chain verification
- Atomic payment settlement
- Resource access control

---

## Integration Options

### SDKs (Recommended)
Fast integration with type-safe libraries:

```bash
# JavaScript/TypeScript
npm install agentgatepay-sdk

# Python
pip install agentgatepay-sdk
```

**[SDK Documentation](https://github.com/AgentGatePay/agentgatepay-sdks)**

### REST API
Direct HTTP integration with any language:

**[API Reference](docs/api/endpoints-reference.md)** | **[Postman Collection](docs/testing/postman/)**

### MCP Tools
For Claude Desktop, OpenAI, and MCP-compatible platforms:

**[15 Tools Reference](docs/mcp/tools-reference.md)**

---

## Examples

Full production-ready examples in the **[Examples Repository](https://github.com/AgentGatePay/agentgatepay-examples)**:

### Python + LangChain (Available Now)
- Basic payment flow
- Buyer/seller marketplace
- MCP tool integration
- Complete feature demo (11 features)
- Production TX signing
- Monitoring dashboards

### n8n Workflows (Available Now)
- Autonomous buyer agent
- Seller API server
- Buyer monitoring dashboard
- Seller revenue analytics

### JavaScript + LangChain.js (Available Now)
- Complete payment flow
- Buyer/seller marketplace
- MCP tool integration
- Complete feature demo
- Production TX signing
- Monitoring dashboards

### Need Examples for Your Framework?

**We create framework-specific examples based on user demand.**

Currently available:
- ✅ **Python + LangChain** - 10 complete examples
- ✅ **n8n Workflows** - 4 production workflows
- ✅ **JavaScript + LangChain.js** - 10 complete examples

**Request examples for your framework:**
- AutoGPT
- CrewAI
- Semantic Kernel
- Vercel AI SDK
- AutoGen
- Other AI frameworks

**How to request:**
Email **support@agentgatepay.com** with:
1. Framework name and version
2. Your use case (buyer, seller, or both)
3. Preferred programming language

We prioritize examples based on demand. If multiple users request the same framework, we'll build and publish the example within days.

---

## Testing

### Postman Collection
Import the complete API collection for testing:

**[Download Postman Collection](docs/testing/postman/AgentGatePay.postman_collection.json)**

See **[Postman Guide](docs/testing/postman/README.md)** for setup instructions.

### Curl Examples
Manual testing with curl:

**[Curl Examples](docs/testing/curl-examples.md)** - 30+ ready-to-run commands.

---

## Supported Chains & Tokens

| Chain      | Chain ID | USDC | USDT | DAI |
|------------|----------|------|------|-----|
| Ethereum   | 1        | Yes  | Yes  | Yes |
| Base       | 8453     | Yes  | Yes  | Yes |
| Polygon    | 137      | Yes  | Yes  | Yes |
| Arbitrum   | 42161    | Yes  | Yes  | Yes |

---

## Rate Limits

Rate limits are enforced per-endpoint to prevent abuse:

| Endpoint               | Rate Limit      |
|------------------------|-----------------|
| `/mandates/issue`      | 20 req/min      |
| `/mandates/verify`     | 100 req/min     |
| `/x402/resource`       | 60 req/min      |
| Other endpoints        | 60 req/min      |

**Rate limit headers** included in all responses:
- `X-RateLimit-Limit`: Maximum requests per window
- `X-RateLimit-Remaining`: Requests remaining in window
- `X-RateLimit-Reset`: Unix timestamp when window resets

---

## Beta Status & Feedback

AgentGatePay is currently in **beta**.

**We're actively seeking feedback!** This is a new and rapidly evolving space. We're building payment infrastructure for the agent economy and want to hear from developers, agent builders and early adopters about what works and what needs improvement.

**Share your feedback:**
- **Email**: support@agentgatepay.com - We read every message!
- Try the platform and let us know your experience
- Your input directly shapes our roadmap and priorities

### Current Capabilities (Beta)
- Multi-chain blockchain payments (Ethereum, Base, Polygon, Arbitrum)
- Multi-token support (USDC, USDT, DAI)
- AP2 mandates with cryptographic budget controls
- Agent reputation system (AIF security)
- Real-time analytics and webhooks
- MCP integration (15 tools for Claude, OpenAI)
- Production-ready security features
- REST API + Python/JavaScript SDKs

### Known Limitations & Upcoming Features

**Current Limitations:**
- Mainnet only (no testnet support yet)
- Manual wallet funding required
- Command-line/API access only (no web UI)
- Crypto payments only (blockchain required)
- Limited error recovery for failed transactions

**Coming Soon (High Priority):**
- **🎨 Web Dashboard UI** - Visual interface for managing mandates, viewing analytics and monitoring transactions without code. Making AgentGatePay accessible to non-technical users. (In active development)
- **💳 Traditional Payment Methods** - Credit card support, ACH bank transfers, and other conventional payment options. Bridging crypto and traditional finance for broader adoption. (Roadmap: Q1-Q2 2026)
- **🧪 Testnet Support** - Safe testing environment with test tokens and faucets for risk-free experimentation
- **📧 Email Notifications** - Automated alerts for payments, mandate expirations and important events
- **🔄 Enhanced Error Recovery** - Improved handling of failed transactions with automatic retry logic
- **📱 Mobile SDKs** - Native iOS and Android libraries for mobile agent applications

### Roadmap

**Your feedback shapes our roadmap.** We prioritize features based on real-world developer needs and use cases. Email us at support@agentgatepay.com with feature requests or suggestions.

---

## Documentation

### Quickstart Guides
- [Buyer Guide](docs/quickstart/buyer-guide.md) - Send payments in 5 minutes
- [Seller Guide](docs/quickstart/merchant-guide.md) - Receive payments in 5 minutes

### API Reference
- [Complete Endpoints Reference](docs/api/endpoints-reference.md) - 24 endpoints with examples
- [Postman Collection](docs/testing/postman/) - Import & test API
- [Curl Examples](docs/testing/curl-examples.md) - 30+ curl commands

### MCP Integration
- [15 Tools Reference](docs/mcp/tools-reference.md) - Complete MCP tool documentation

### Full Documentation
See [docs/README.md](docs/README.md) for complete documentation index.

---

## Support

### Get Help
- **Documentation**: [docs/](docs/)
- **Email Support**: support@agentgatepay.com - Bug reports, feature requests, integration help
- **Examples**: [GitHub Examples Repo](https://github.com/AgentGatePay/agentgatepay-examples)

### Contributing
AgentGatePay is currently in beta. We welcome feedback and bug reports via email at support@agentgatepay.com.

---

## License

Copyright (c) 2025 AgentGatePay. All Rights Reserved.

This documentation is proprietary to AgentGatePay. See [LICENSE](LICENSE) for full terms.

### Licensing Strategy

AgentGatePay uses a **dual licensing approach** designed to protect our core infrastructure while enabling developers to freely use and modify integration examples:

| Repository | License | Purpose |
|------------|---------|---------|
| [agentgatepay](https://github.com/AgentGatePay/agentgatepay) | Proprietary | Documentation & specifications |
| [agentgatepay-sdks](https://github.com/AgentGatePay/agentgatepay-sdks) | **MIT License** | Official Python & JavaScript libraries |
| [TX](https://github.com/AgentGatePay/TX) | Proprietary | Transaction signing service |
| [agentgatepay-examples](https://github.com/AgentGatePay/agentgatepay-examples) | **MIT License** | Integration examples & workflows |

**Why This Approach?**

**Infrastructure Protection (Proprietary):**
- Core platform documentation and TX signing service remain proprietary
- Protects our infrastructure intellectual property
- Ensures quality control and security standards
- Allows us to build a sustainable business

**Developer Freedom (MIT License):**
- SDKs and integration examples are **MIT-licensed** (most permissive open source)
- Developers can copy, modify, distribute, and sell code freely
- No restrictions on commercial use
- Fork and customize for your projects without attribution required

**What This Means for You:**
- ✅ Use AgentGatePay SDKs in your commercial projects (MIT licensed)
- ✅ Copy, modify, and redistribute SDK code freely
- ✅ Build and sell applications using AgentGatePay
- ✅ No licensing fees for using the platform or SDKs
- ✅ Fork and customize SDKs for your specific needs

**Questions?** Email support@agentgatepay.com for licensing inquiries.

---

**AgentGatePay** - Payment infrastructure for the agent economy. **Currently in beta** - we're actively improving based on real-world usage and developer feedback. Your feedback helps our roadmap.
