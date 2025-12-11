# AgentGatePay Documentation

Complete documentation for integrating AgentGatePay into your application.

**Current Version:** Beta | [Visual Introduction (PDF)](overview/AgentGatePay-Overview-Presentation.pdf)

---

## Quick Links

- [Main README](../) - Repository landing page
- **[📊 Visual Introduction (PDF)](overview/AgentGatePay-Overview-Presentation.pdf)** - 9-slide visual overview (start here!)
- [Buyer Quickstart](quickstart/buyer-guide.md) - Send your first payment in 5 minutes
- [Seller Quickstart](quickstart/merchant-guide.md) - Receive your first payment in 5 minutes
- [API Reference](api/endpoints-reference.md) - Complete endpoint documentation
- [MCP Tools](mcp/tools-reference.md) - AI agent integration

---

## Getting Started

### I want to send payments (Buyer/Agent)

Start here: **[Buyer Quickstart Guide](quickstart/buyer-guide.md)**

You'll learn how to:
1. Create an account (30 seconds)
2. Issue a budget mandate (1 minute)
3. Make your first payment (2 minutes)
4. Monitor spending (1 minute)

**Total time: 5 minutes**

### I want to receive payments (Seller/Merchant)

Start here: **[Seller Quickstart Guide](quickstart/merchant-guide.md)**

You'll learn how to:
1. Create a merchant account (30 seconds)
2. Add your wallet address (30 seconds)
3. Configure webhooks (1 minute)
4. Verify payments (1 minute)
5. Track revenue (1 minute)

**Total time: 5 minutes**

### I'm building an AI agent

Start here: **[MCP Tools Reference](mcp/tools-reference.md)**

Options:
- **Claude Desktop**: Stdio bridge integration
- **OpenAI/Custom**: JSON-RPC unified endpoint
- **SDKs**: Official JavaScript & Python libraries ([see examples repo](https://github.com/AgentGatePay/agentgatepay-sdks))

---

## Documentation Structure

### Overview
- **[Visual Introduction (PDF)](overview/AgentGatePay-Overview-Presentation.pdf)** - 9-slide visual guide for everyone
- [Architecture Diagrams](architecture/diagrams.md) - System flows and security layers
- [Diagram Verification Report](architecture/DIAGRAM_VERIFICATION_REPORT.md) - Code accuracy validation

### Quickstart Guides
- [Buyer Guide](quickstart/buyer-guide.md) - Send payments (5 minutes)
- [Seller Guide](quickstart/merchant-guide.md) - Receive payments (5 minutes)

### API Reference
- [Endpoints Reference](api/endpoints-reference.md) - Complete API documentation with examples

### MCP Integration
- [15 Tools Reference](mcp/tools-reference.md) - Complete tool documentation for AI agents

### Testing
- [Postman Collection](testing/postman/) - Import & run API tests
- [Curl Examples](testing/curl-examples.md) - Command-line testing

### External Resources
- **[Official SDKs](https://github.com/AgentGatePay/agentgatepay-sdks)** - JavaScript & Python libraries
- **[Examples Repository](https://github.com/AgentGatePay/agentgatepay-examples)** - LangChain, n8n workflows
- **[TX Signing Service](https://github.com/AgentGatePay/TX)** - External transaction signing

---

## Popular Topics

### How do I...

**...create an account?**
→ See "Step 1: Create Your Account" in [Buyer Quickstart](quickstart/buyer-guide.md) or [Seller Quickstart](quickstart/merchant-guide.md)

**...issue a budget mandate?**
→ See "Step 2: Issue a Mandate" in [Buyer Quickstart](quickstart/buyer-guide.md)

**...send my first payment?**
→ [Buyer Quickstart](quickstart/buyer-guide.md)

**...receive payments?**
→ [Seller Quickstart](quickstart/merchant-guide.md)

**...configure webhooks?**
→ See "Step 3: Configure Webhooks" in [Seller Quickstart](quickstart/merchant-guide.md)

**...integrate with my AI agent?**
→ [MCP Tools Reference](mcp/tools-reference.md)

**...use the SDKs?**
→ [Official SDKs Repository](https://github.com/AgentGatePay/agentgatepay-sdks)

**...test the API?**
→ [Curl Examples](testing/curl-examples.md) | [Postman Collection](testing/postman/)

**...see payment history?**
→ See "Analytics" endpoints in [API Reference](api/endpoints-reference.md)

**...verify a payment?**
→ See "Payment Verification" section in [API Reference](api/endpoints-reference.md)

---

## Code Examples

### JavaScript/TypeScript
```bash
npm install agentgatepay-sdk
```
[SDK Documentation](https://github.com/AgentGatePay/agentgatepay-sdks)

### Python
```bash
pip install agentgatepay-sdk
```
[SDK Documentation](https://github.com/AgentGatePay/agentgatepay-sdks)

### Complete Examples
[Examples Repository](https://github.com/AgentGatePay/agentgatepay-examples)
- Python + LangChain (10 examples) ✅
- n8n Workflows (4 workflows) ✅
- JavaScript + LangChain.js (10 examples) ✅

**Need examples for your framework?** (AutoGPT, CrewAI, Semantic Kernel, Vercel AI SDK, AutoGen)
Email **support@agentgatepay.com** - We create framework-specific examples based on user demand.

---

## Support

### Get Help
- **Documentation**: You're reading it!
- **GitHub Issues**: [Report bugs or request features](mailto:support@agentgatepay.com)
- **Email**: support@agentgatepay.com
- **Examples**: [GitHub Examples Repo](https://github.com/AgentGatePay/agentgatepay-examples)

### Contributing
AgentGatePay is currently in beta. We welcome:
- Bug reports
- Feature requests
- Documentation improvements
- Integration examples

Submit via **Email**: support@agentgatepay.com.

---

## Status

**Current Version:** Beta

**What's Working:**
- Multi-chain payments (4 chains, 3 tokens)
- AP2 mandates with budget tracking
- Agent reputation system
- Real-time analytics and webhooks
- MCP integration (15 tools)
- Production-ready security (AIF)

**Known Limitations:**
- Mainnet only (no testnet support yet)
- Manual wallet funding required
- Email notifications not yet implemented

**Planned Features:**
- Testnet support for development
- Automated wallet funding
- Email notifications for payments
- Additional payment methods (credit cards, ACH)

---

**Last Updated:** December 2025
**Documentation Version:** 1.0.0
