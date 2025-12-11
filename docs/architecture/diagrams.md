# AgentGatePay Architecture Diagrams

**Public-facing conceptual diagrams** for understanding how AgentGatePay works.

**Current Version:** Beta v0.9.0 | **Last Updated:** December 2025

**Note:** These diagrams show high-level concepts and user-facing flows only. Internal implementation details are not shown for security reasons.

---

## 1. Payment Flow Overview

### Agent-to-Agent Payment (x402 Protocol)

```mermaid
sequenceDiagram
    actor Agent as AI Agent (Buyer)
    participant API as AgentGatePay API
    participant BC as Blockchain
    participant Seller as Service Provider

    Note over Agent,Seller: Complete Payment Flow

    Agent->>API: 1. Issue Mandate
    Note over Agent,API: Budget: $100, Scope: "research"
    API-->>Agent: Mandate Token

    Agent->>Seller: 2. Request Resource
    Seller-->>Agent: 402 Payment Required
    Note over Seller,Agent: Price: $0.01, Wallet: 0xABC...

    Agent->>BC: 3. Send USDC Transaction
    Note over Agent,BC: Sign & Submit Payment
    BC-->>Agent: Transaction Hash

    Agent->>API: 4. Submit tx_hash
    Note over Agent,API: Headers: x-mandate, x-payment
    API->>BC: Verify Transaction
    BC-->>API: Confirmed ✓
    API-->>Agent: 200 OK + Resource Access

    API->>Seller: 5. Webhook Notification
    Note over API,Seller: Payment received
```

---

## 2. Multi-Chain Architecture

### How Chain Selection Works

```mermaid
graph TD
    A[AI Agent] -->|Request Payment| B{AgentGatePay API}
    B -->|Specify Chain| C[Base]
    B -->|Specify Chain| D[Ethereum]
    B -->|Specify Chain| E[Polygon]
    B -->|Specify Chain| F[Arbitrum]

    C -->|2-5 sec| G[USDC/DAI]
    D -->|15-60 sec| H[USDC/USDT/DAI]
    E -->|3-8 sec| I[USDC/USDT/DAI]
    F -->|3-8 sec| J[USDC/USDT/DAI]

    G --> K[Settlement]
    H --> K
    I --> K
    J --> K

    K --> L[Verification]
    L --> M[Agent Receives Resource]

    style C fill:#0052FF
    style D fill:#627EEA
    style E fill:#8247E5
    style F fill:#28A0F0
```

**Key Points:**
- Agents choose chain via query parameters: `?chain=base&token=USDC`
- Different chains have different settlement speeds
- AgentGatePay verifies transactions on-chain for all chains
- Multi-token support varies by chain

---

## 3. Security Layers (AIF)

### Agent Interaction Firewall Protection

```mermaid
graph TB
    subgraph "AI Agent Request"
        A[Agent Makes Request]
    end

    subgraph "AIF Security Layers"
        B[1. Rate Limiting]
        C[2. Replay Protection]
        D[3. Mandate Verification]
        E[4. Reputation Check]
        F[5. Pattern Detection]
    end

    subgraph "Blockchain Verification"
        G[6. On-Chain Verification]
        H[7. Commission Enforcement]
    end

    subgraph "Final Decision"
        I{Allow Payment?}
    end

    A --> B
    B --> C
    C --> D
    D --> E
    E --> F
    F --> G
    G --> H
    H --> I

    I -->|Yes ✓| J[Grant Access]
    I -->|No ✗| K[Reject & Log]

    style B fill:#FFE5E5
    style C fill:#FFE5E5
    style D fill:#FFE5E5
    style E fill:#FFE5E5
    style F fill:#FFE5E5
    style G fill:#E5F5FF
    style H fill:#E5F5FF
    style J fill:#E5FFE5
    style K fill:#FFE5E5
```

**Security Features:**
1. **Rate Limiting** - Prevents abuse (20-100 req/min depending on endpoint)
2. **Replay Protection** - Each transaction can only be used once
3. **Mandate Verification** - Budget and scope enforcement
4. **Reputation Check** - Agent behavior scoring (0-200)
5. **Pattern Detection** - Suspicious activity monitoring
6. **On-Chain Verification** - Blockchain transaction confirmation
7. **Commission Enforcement** - Automatic 0.5% fee collection

---

## 4. Integration Options

### How Developers Connect to AgentGatePay

```mermaid
graph LR
    subgraph "Developer's Choice"
        A[Choose Integration Method]
    end

    subgraph "Option 1: SDKs"
        B[JavaScript/TypeScript SDK]
        C[Python SDK]
    end

    subgraph "Option 2: REST API"
        D[Direct HTTP Calls]
    end

    subgraph "Option 3: MCP Tools"
        E[Claude Desktop]
        F[OpenAI Agents]
        G[Custom MCP Clients]
    end

    subgraph "AgentGatePay Platform"
        H[Payment Processing]
        I[Verification]
        J[Analytics]
    end

    A --> B
    A --> C
    A --> D
    A --> E
    A --> F
    A --> G

    B --> H
    C --> H
    D --> H
    E --> H
    F --> H
    G --> H

    H --> I
    I --> J
    J --> K[AI Agent Application]

    style B fill:#F0DB4F
    style C fill:#3776AB
    style D fill:#FF6B6B
    style E fill:#9B59B6
    style F fill:#2ECC71
    style G fill:#E74C3C
```

**Integration Comparison:**

| Method | Setup Time | Best For | Language Support |
|--------|------------|----------|------------------|
| **SDK** | 10 minutes | Production apps, type safety | JavaScript, Python |
| **REST API** | 15 minutes | Any language, full control | Any |
| **MCP Tools** | 5 minutes | Claude/OpenAI agents | Framework-agnostic |

---

## 5. Mandate (Budget Control) Flow

### How AP2 Mandates Work

```mermaid
stateDiagram-v2
    [*] --> Issued: Agent Issues Mandate

    Issued --> Active: Budget: $100

    state Active {
        [*] --> Checking: Payment Request
        Checking --> Deduct: Budget Available
        Checking --> Reject: Insufficient Budget
        Deduct --> Checking: Continue Spending
    }

    Active --> Expired: TTL Reached
    Active --> Depleted: Budget = $0

    Expired --> [*]
    Depleted --> [*]
    Reject --> [*]: Request Denied

    note right of Issued
        Mandate Parameters:
        - Budget: USD amount
        - Scope: "research", "compute", etc.
        - TTL: Time to live (minutes)
        - Subject: Agent identifier
    end note

    note right of Active
        Budget Tracking:
        - Real-time deduction
        - Automatic enforcement
        - Cannot exceed limit
    end note
```

**Key Concepts:**
- **Mandate** = Pre-authorized spending limit for agents
- **Budget** = Maximum USD amount agent can spend
- **Scope** = What agent is allowed to purchase
- **TTL** = How long mandate remains valid
- **Automatic Enforcement** = Gateway rejects payments exceeding budget

---

## 6. Transaction Signing Options

### Client-Side vs. Server-Side Signing

```mermaid
graph TD
    subgraph "Option 1: Client-Side Signing"
        A1[AI Agent] -->|Has Private Key| B1[Sign Transaction]
        B1 --> C1[Submit to Blockchain]
        C1 --> D1[Get tx_hash]
        D1 --> E1[Send to AgentGatePay]
        E1 --> F1[Verification Only]
    end

    subgraph "Option 2: TX Signing Service"
        A2[AI Agent] -->|API Call| B2[TX Service]
        B2 -->|API Key Check| C2[Authorized?]
        C2 -->|Yes| D2[Sign Transaction]
        D2 --> E2[Submit to Blockchain]
        E2 --> F2[Return tx_hash]
        F2 --> G2[Send to AgentGatePay]
        G2 --> H2[Verification]
    end

    style A1 fill:#E3F2FD
    style B1 fill:#E3F2FD
    style C1 fill:#E3F2FD
    style D1 fill:#E3F2FD
    style E1 fill:#E3F2FD
    style F1 fill:#E3F2FD

    style A2 fill:#FFF3E0
    style B2 fill:#FFF3E0
    style C2 fill:#FFF3E0
    style D2 fill:#FFF3E0
    style E2 fill:#FFF3E0
    style F2 fill:#FFF3E0
    style G2 fill:#FFF3E0
    style H2 fill:#FFF3E0
```

**Comparison:**

| Aspect | Client-Side | TX Service |
|--------|-------------|------------|
| **Security** | Keys in client code (risk) | Keys isolated on server (safer) |
| **Complexity** | Manage wallets per agent | Simple API call |
| **Commission** | Trust client enforcement | Server-enforced (guaranteed) |
| **Best For** | Trusted environments | Production deployments |

---

## 7. Webhook Notification Flow

### How Sellers Get Notified

```mermaid
sequenceDiagram
    actor Buyer as AI Agent (Buyer)
    participant BC as Blockchain
    participant API as AgentGatePay
    participant Seller as Merchant Server

    Note over Buyer,Seller: Payment Notification Flow

    Buyer->>BC: 1. Send Payment Transaction
    BC-->>Buyer: tx_hash

    Buyer->>API: 2. Submit tx_hash
    API->>BC: 3. Verify Transaction
    BC-->>API: Confirmed ✓

    API->>Seller: 4. Webhook POST
    Note over API,Seller: JSON payload with payment details
    Seller-->>API: 200 OK

    API-->>Buyer: 5. Resource Access Granted

    alt Webhook Failed
        API->>API: Increment failure_count
        Note over API: Manual review required<br/>(no automatic retry)
    end

    Note over Seller: Optional: Seller can verify<br/>payment manually via API
```

**Webhook Benefits:**
- Real-time payment notifications
- Automatic seller notification (no polling required)
- Failure tracking for monitoring
- Signature verification for security

---

## 8. Audit & Analytics Flow

### Transaction Tracking

```mermaid
graph LR
    subgraph "Payment Events"
        A[Mandate Issued]
        B[Payment Request]
        C[Transaction Submitted]
        D[Payment Verified]
        E[Resource Delivered]
    end

    subgraph "Logging System"
        F[Audit Logs]
    end

    subgraph "Analytics Engine"
        G[Real-Time Metrics]
        H[Historical Data]
        I[Budget Tracking]
    end

    subgraph "User Dashboards"
        J[Buyer Dashboard]
        K[Seller Dashboard]
        L[Admin Dashboard]
    end

    A --> F
    B --> F
    C --> F
    D --> F
    E --> F

    F --> G
    F --> H
    F --> I

    G --> J
    G --> K
    G --> L

    H --> J
    H --> K
    H --> L

    I --> J
    I --> K

    style F fill:#FFE5E5
    style G fill:#E5F5FF
    style H fill:#E5F5FF
    style I fill:#E5F5FF
    style J fill:#E5FFE5
    style K fill:#E5FFE5
    style L fill:#E5FFE5
```

**What's Tracked:**
- Every mandate issuance
- All payment requests
- Transaction submissions
- Verification outcomes
- Budget deductions
- Commission collection
- Webhook deliveries

**Access Levels:**
- **Buyer** - See spending, mandate budget remaining, payment history
- **Seller** - See revenue, payment confirmations, webhook logs
- **Admin** - See system-wide metrics, security events, performance

---

## 9. Reputation System (AIF)

### Agent Behavior Scoring

```mermaid
graph TD
    A[Agent Makes Payment] --> B{Check Events}

    B -->|Normal Payment| C[+5 to +30 Points]
    B -->|Failed Transaction| D[-3 to -15 Points]
    B -->|Replay Attempt| E[-20 Points]
    B -->|Rate Limit Hit| F[-10 Points]
    B -->|Spam Detected| G[-30 Points]

    C --> H[Update Score]
    D --> H
    E --> H
    F --> H
    G --> H

    H --> I{Current Score?}

    I -->|< 30| J[Block Agent]
    I -->|< 60| K[Warning State]
    I -->|>= 60| L[Allow Payment]

    J --> M[Reject Request]
    K --> N[Allow with Monitoring]
    L --> O[Normal Processing]

    style C fill:#E5FFE5
    style D fill:#FFE5E5
    style E fill:#FFE5E5
    style F fill:#FFE5E5
    style G fill:#FFE5E5
    style J fill:#FF0000,color:#FFFFFF
    style K fill:#FFA500
    style L fill:#00FF00
```

**Reputation Mechanics:**
- **Starting Score:** 100 (neutral)
- **Maximum Score:** 200 (excellent)
- **Minimum Score:** 0 (blocked)
- **Decay:** Scores naturally increase over time if no violations
- **Manual Reset:** Admin can reset false positives

**Point Distribution:**
- **Normal Payment:** +5 to +30 points (based on transaction amount)
- **Failed Transaction:** -3 to -15 points (based on chain/token)
- **Replay Attempt:** -20 points (security violation)
- **Rate Limit Hit:** -10 points (abuse prevention)
- **Spam Detected:** -30 points (severe violation)

**Agent States:**
- **Blocked (< 30)** - Cannot make payments, requires manual review
- **Warning (< 60)** - Extra monitoring, stricter limits
- **Good (61-100)** - Normal operation
- **Excellent (101-200)** - Trusted agent, potential perks

---

## 10. System Components Overview

### High-Level Architecture

```mermaid
graph TB
    subgraph "User Layer"
        A[AI Agents]
        B[Developers]
        C[Merchants]
    end

    subgraph "Integration Layer"
        D[SDKs]
        E[REST API]
        F[MCP Tools]
    end

    subgraph "AgentGatePay Platform"
        G[API Gateway]
        H[Payment Processing]
        I[Security AIF]
        J[Analytics]
    end

    subgraph "Blockchain Layer"
        K[Ethereum]
        L[Base]
        M[Polygon]
        N[Arbitrum]
    end

    subgraph "Data Layer"
        O[Transaction History]
        P[Audit Logs]
        Q[Mandate Storage]
    end

    A --> D
    A --> E
    A --> F
    B --> D
    B --> E
    B --> F
    C --> E

    D --> G
    E --> G
    F --> G

    G --> H
    G --> I
    G --> J

    H --> K
    H --> L
    H --> M
    H --> N

    H --> O
    I --> P
    J --> Q

    style G fill:#E3F2FD
    style H fill:#E3F2FD
    style I fill:#FFE5E5
    style J fill:#E5FFE5
```

**Component Responsibilities:**

| Component | Purpose |
|-----------|---------|
| **API Gateway** | Entry point for all requests |
| **Payment Processing** | Transaction verification & settlement |
| **Security (AIF)** | Rate limiting, replay protection, reputation |
| **Analytics** | Real-time metrics & historical tracking |
| **Blockchain Layer** | Multi-chain transaction submission & verification |
| **Data Layer** | Persistent storage for mandates, logs, history |

---

## Additional Resources

- **[API Reference](../api/endpoints-reference.md)** - Complete endpoint documentation
- **[Buyer Quickstart](../quickstart/buyer-guide.md)** - Getting started with payments
- **[Seller Quickstart](../quickstart/merchant-guide.md)** - Getting started with receiving
- **[MCP Tools](../mcp/tools-reference.md)** - AI agent integration

---

**Note:** These diagrams show conceptual flows and public-facing components only. Internal infrastructure details (databases, servers, code modules) are not shown for security reasons.

**Last Updated:** December 2025
**Diagrams Version:** 1.0.0
**Format:** Mermaid.js (GitHub-rendered)
