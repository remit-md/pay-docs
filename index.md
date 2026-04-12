---
layout: home

hero:
  name: Pay
  text: The Complete x402 Payment Stack
  tagline: Gateway, facilitator, SDKs, CLI, and MCP server. AI agents pay for APIs with USDC on Base.
  actions:
    - theme: brand
      text: Get Started
      link: /quickstart/
    - theme: alt
      text: Architecture
      link: /architecture
    - theme: alt
      text: Provider Guide
      link: /provider-guide

features:
  - title: Agent Side
    details: Python and TypeScript SDKs, CLI, MCP server, and integrations for Claude, LangChain, CrewAI, LlamaIndex, Vercel AI SDK, OpenAI, and Semantic Kernel. One line of code to pay any API.
  - title: Provider Side
    details: pay-gate reverse proxy. Deploy in front of any HTTP API. Define pricing per route. Zero code changes to your backend. Cloudflare Worker, Docker, Rust binary, or sidecar.
  - title: Protocol
    details: x402 HTTP 402 paywalls. Direct settlement for payments $1+. Tab settlement for micropayments at any price point. USDC on Base. Gas paid by the protocol.
---

## Install

### CLI

```bash
# Homebrew (macOS/Linux)
brew install pay-skill/tap/pay

# Scoop (Windows)
scoop bucket add pay-skill https://github.com/pay-skill/scoop-pay
scoop install pay

# From source
cargo install pay-cli
```

```bash
# First-time setup
pay init
pay fund    # add USDC via Coinbase Onramp (zero fee)

# Make a paid API call (auto-handles x402)
pay request https://api.example.com/data

# Send $5 to a provider
pay direct 0xprovider... 5.00
```

### TypeScript

```bash
npm install @pay-skill/sdk
```

```typescript
import { Wallet } from "@pay-skill/sdk";
const wallet = await Wallet.create();  // OS keychain (same key as CLI)
const response = await wallet.request("https://api.example.com/data");
```

### Python

```bash
pip install payskill
```

```python
from payskill import Wallet
wallet = Wallet()
response = wallet.request("https://api.example.com/data")
```

### MCP Server (Claude Desktop / Cursor / VS Code)

```bash
npx @pay-skill/mcp
```

## Components

| Component | What it does | Install |
|-----------|-------------|---------|
| [CLI](/cli/) | Command-line tool for agents | `brew install pay-skill/tap/pay` |
| [TypeScript SDK](/sdk/typescript) | Client library for Node.js/TS agents | `npm install @pay-skill/sdk` |
| [Python SDK](/sdk/python) | Client library for Python agents | `pip install payskill` |
| [MCP Server](/quickstart/claude-desktop) | Tools for Claude Desktop, Cursor, VS Code | `npx @pay-skill/mcp` |
| [pay-gate](/gate/) | Reverse proxy for API providers | `npm create pay-gate` |
| [Framework Integrations](/integrations) | LangChain, CrewAI, LlamaIndex, Vercel AI SDK, OpenAI, Semantic Kernel | See [integrations](/integrations) |

## Links

- [Architecture](/architecture)
- [CLI Reference](/cli/)
- [Provider Guide](/provider-guide)
- [API Reference](/api-reference)
- [Contracts & Networks](/contracts)
- [Troubleshooting](/troubleshooting)
