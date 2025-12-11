# AgentGatePay Postman Collection

Complete API testing collection for AgentGatePay with all endpoints and example requests.

---

## Quick Start

### 1. Import Collection

**Download:** [AgentGatePay.postman_collection.json](AgentGatePay.postman_collection.json)

**Import to Postman:**
1. Open Postman
2. Click "Import" button
3. Drag and drop the JSON file
4. Collection appears in left sidebar

---

### 2. Configure Environment Variables

Create a new environment with these variables:

| Variable | Description | Example Value |
|----------|-------------|---------------|
| `api_url` | Base API URL | `https://api.agentgatepay.com` |
| `api_key` | Your API key | `pk_live_abc123...` |
| `mandate_token` | Current mandate token | `eyJhbGc...` |
| `wallet_address` | Your wallet address | `0xYour...` |

**Setup:**
1. Click "Environments" in Postman
2. Click "+" to create new environment
3. Add variables above
4. Click "Save"
5. Select environment from dropdown

---

### 3. Run Collection

**Quick Test:**
1. Select "AgentGatePay" collection
2. Click "Run" button
3. Select requests to test
4. Click "Run AgentGatePay"

**Manual Testing:**
1. Open any request in collection
2. Update variables if needed
3. Click "Send"
4. View response

---

## Collection Structure

### Core Endpoints
- `GET /health` - System health check
- `GET /x402/resource` - Protected resource access

### Mandates
- `POST /mandates/issue` - Issue AP2 mandate
- `POST /mandates/verify` - Verify mandate token

### User Management
- `POST /v1/users/signup` - Create account
- `GET /v1/users/me` - Get profile
- `POST /v1/users/wallets/add` - Add wallet

### API Keys
- `POST /v1/api-keys/create` - Create API key
- `GET /v1/api-keys/list` - List API keys
- `POST /v1/api-keys/revoke` - Revoke API key

### Payments
- `GET /v1/payments/verify/{tx_hash}` - Verify payment
- `GET /v1/payments/status/{tx_hash}` - Payment status
- `GET /v1/payments/list` - Payment history

### Webhooks
- `POST /v1/webhooks/configure` - Configure webhook
- `GET /v1/webhooks/list` - List webhooks
- `POST /v1/webhooks/test` - Test webhook
- `DELETE /v1/webhooks/{webhook_id}` - Delete webhook

### Analytics
- `GET /v1/analytics/public` - Public analytics
- `GET /v1/analytics/me` - User analytics
- `GET /v1/merchant/revenue` - Revenue analytics

### Audit Logs
- `GET /audit/logs` - List audit logs
- `GET /audit/stats` - Audit statistics
- `GET /audit/logs/transaction/{tx_hash}` - Transaction logs

### MCP (Model Context Protocol)
- `POST https://mcp.agentgatepay.com/` - Unified MCP endpoint

---

## Usage Guide

### Complete Payment Flow

**Step 1: Create Account**
```
Request: POST /v1/users/signup
Body: {
  "email": "test@example.com",
  "password": "SecurePass123",
  "account_type": "agent"
}
Response: Save api_key to environment variable
```

**Step 2: Issue Mandate**
```
Request: POST /mandates/issue
Headers: x-api-key: {{api_key}}
Body: {
  "subject": "{{wallet_address}}",
  "budget_usd": "10.00",
  "scope": "research",
  "ttl_minutes": 1440
}
Response: Save mandate_token to environment variable
```

**Step 3: Verify Mandate**
```
Request: POST /mandates/verify
Body: {
  "mandate_token": "{{mandate_token}}"
}
Response: Check budget_remaining
```

**Step 4: Request Resource (Get Payment Details)**
```
Request: GET /x402/resource
Headers:
  x-agent-id: test-agent
  x-mandate: {{mandate_token}}
Query: ?chain=base&token=USDC
Response: Save receiver_address, amount_tokens
```

**Step 5: Submit Payment**
```
(Send blockchain transaction first using ethers.js/web3.py)

Request: GET /x402/resource
Headers:
  x-agent-id: test-agent
  x-mandate: {{mandate_token}}
  x-payment: {"tx_hash":"0xYourTxHash","chain":"base","token":"USDC"}
Response: Access granted with resource data
```

---

## Pre-request Scripts

Some requests have pre-request scripts that:
- Generate timestamps
- Create signatures
- Format payloads
- Update environment variables

**Example - Auto-generate email:**
```javascript
pm.environment.set("test_email", `test-${Date.now()}@example.com`);
```

---

## Tests

Each request includes tests to validate responses:

```javascript
// Status code test
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

// Response body test
pm.test("Response has mandate_token", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('mandate_token');
});

// Save to environment
pm.test("Save mandate_token", function () {
    var jsonData = pm.response.json();
    pm.environment.set("mandate_token", jsonData.mandate_token);
});
```

**View test results:**
- Click "Test Results" tab after running request
- Green checkmarks = passed tests
- Red X = failed tests

---

## Collection Runner

Run entire collection or specific folders:

**Setup:**
1. Click "Runner" button (top right)
2. Select "AgentGatePay" collection
3. Select environment
4. Configure iterations (default: 1)
5. Add delay between requests (optional)

**Run specific flow:**
1. Select folder (e.g., "Core Endpoints")
2. Click "Run"
3. View summary report

**Export results:**
1. After run completes
2. Click "Export Results"
3. Save JSON report

---

## Common Issues

### Issue: "api_key is undefined"

**Cause:** Environment variable not set
**Solution:**
1. Run `/v1/users/signup` request
2. Copy `api_key` from response
3. Add to environment variables
4. Rerun request

### Issue: "mandate_token is undefined"

**Cause:** Haven't issued mandate yet
**Solution:**
1. Run `/mandates/issue` request
2. Copy `mandate_token` from response
3. Auto-saved to environment variable
4. Rerun request

### Issue: "401 Unauthorized"

**Cause:** Missing or invalid API key
**Solution:**
1. Check `api_key` environment variable
2. Ensure it starts with `pk_live_`
3. Run signup if you don't have a key

### Issue: "429 Too Many Requests"

**Cause:** Rate limit exceeded
**Solution:**
1. Wait 60 seconds
2. Reduce request frequency
3. Create account for higher limits (100/min vs 20/min)

---

## Advanced Features

### Variables in Requests

Use `{{variable_name}}` syntax:
```
URL: {{api_url}}/v1/users/me
Header: x-api-key: {{api_key}}
Body: {"mandate_token": "{{mandate_token}}"}
```

### Dynamic Data Generation

Use JavaScript in pre-request scripts:
```javascript
// Generate random email
pm.environment.set("random_email", `user-${Math.random().toString(36).substring(7)}@example.com`);

// Generate timestamp
pm.environment.set("timestamp", Math.floor(Date.now() / 1000));

// Generate UUID
pm.environment.set("request_id", pm.variables.replaceIn('{{$guid}}'));
```

### Chain Requests

Use tests to save data for next request:
```javascript
pm.test("Save values for next request", function () {
    var jsonData = pm.response.json();

    // Save mandate token
    pm.environment.set("mandate_token", jsonData.mandate_token);

    // Save mandate ID
    pm.environment.set("mandate_id", jsonData.mandate_id);
});
```

---

## Rate Limits

Rate limits are enforced per-endpoint (all users get the same limits):

| Endpoint               | Rate Limit  |
|------------------------|-------------|
| `/mandates/issue`      | 20 req/min  |
| `/mandates/verify`     | 100 req/min |
| `/x402/resource`       | 60 req/min  |
| Other endpoints        | 60 req/min  |

**Check remaining:**
View response headers:
```
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1702345678
```

---

## Support

- 📖 **API Documentation**: [../api/endpoints-reference.md](../api/endpoints-reference.md)
- 💬 **GitHub Issues**: [Report bugs](mailto:support@agentgatepay.com)
- 📧 **Email**: support@agentgatepay.com

---

**Last Updated:** December 2025
**Collection Version:** 1.0.0
**Total Requests:** 25+
