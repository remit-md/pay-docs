# Architecture

Pay is an end-to-end x402 payment system. Every component -- from the smart contracts to the CLI -- ships as part of one integrated stack.

## System Overview

```
                         Agent                              Provider
                    (SDK / CLI / MCP)                     (any HTTP API)
                           |                                    ^
                           |  1. GET /api/data                  |
                           |  2. 402 Payment Required           |
                           |  3. Sign payment proof             |
                           |  4. Retry with PAYMENT-SIGNATURE   |
                           v                                    |
                      +-----------+                             |
                      | pay-gate  |-------- 5. /verify -------->|
                      | (reverse  |<------- is_valid: true -----|
                      |  proxy)   |-------- 6. proxy request -->|
                      +-----------+                             |
                           |                                    |
                     7. /settle (async)                         |
                           |                                    |
                           v                                    |
                    +-------------+                             |
                    | Facilitator |                              |
                    | pay-skill.  |                              |
                    | com/x402    |                              |
                    +-------------+                              |
                           |                                    |
                    8. On-chain settlement                      |
                           |                                    |
                           v                                    |
                    +-------------+                             |
                    |    Base     |                              |
                    | USDC Smart  |--- provider receives USDC --|
                    | Contracts   |
                    +-------------+
```

**The agent never talks to the blockchain directly.** The SDK signs permits and authorizations off-chain. The facilitator handles on-chain settlement. The agent holds USDC, never ETH. Gas is paid by the protocol.

## Components

### pay-gate (Provider Side)

Stateless reverse proxy that sits in front of any HTTP API. Intercepts unpaid requests, returns `402 Payment Required` with pricing, verifies payment proofs via the facilitator, and proxies paid requests with `X-Pay-*` headers. Deploy as a Cloudflare Worker, Rust binary, Docker container, or sidecar (nginx, Traefik, Caddy, Envoy).

- [Overview](/gate/)
- [Quick Start](/gate/quickstart)
- [Configuration Reference](/gate/config)
- [GitHub](https://github.com/pay-skill/gate)

### Facilitator (Protocol Side)

The x402 facilitator at `pay-skill.com/x402`. Two endpoints:

- **POST /x402/verify** -- Validates payment proofs. Off-chain, instant, no side effects. Called at request time.
- **POST /x402/settle** -- Executes on-chain settlement. Async, happens after the API response. Direct payments settle immediately. Tab charges are batched.

The verify/settle split means API responses are never blocked on blockchain confirmation.

### SDKs (Agent Side)

Python and TypeScript client libraries with identical API surfaces. Handle the full x402 flow transparently: parse 402 responses, sign payment proofs, retry with payment, manage tabs. Default: reads key from OS keychain (same key as `pay` CLI).

- [TypeScript SDK](/sdk/typescript) -- `npm install @pay-skill/sdk`
- [Python SDK](/sdk/python) -- `pip install payskill`

### CLI (Agent Side)

Command-line tool designed for programmatic use by AI agents. JSON output by default, mainnet by default. Agents can discover services, make x402 requests, manage tabs, and send direct payments.

- [CLI Reference](/cli/)
- [GitHub](https://github.com/pay-skill/pay-cli)

### MCP Server (Agent Side)

Model Context Protocol server with 15 tools for Claude Desktop, VS Code, Cursor, and any MCP-compatible client. Wraps the full SDK -- Claude can pay for APIs, open tabs, discover services, and manage wallets from the chat interface.

- [Claude Desktop Setup](/quickstart/claude-desktop)
- [GitHub](https://github.com/pay-skill/mcp)

### Framework Integrations (Agent Side)

Dedicated packages for 7 AI frameworks: Claude (MCP), Vercel AI SDK, OpenAI function calling, LangChain, CrewAI, LlamaIndex, and Semantic Kernel. Each provides tool definitions that map to Pay operations.

- [Integration Guide](/integrations)

### Smart Contracts (Settlement Layer)

Four Solidity contracts on Base:

| Contract | Role | Upgradeability |
|----------|------|---------------|
| **PayDirect** | One-shot USDC transfers with fee deduction | Immutable |
| **PayTab** | Tab lifecycle -- open, charge, settle, close, top-up | Immutable |
| **PayFee** | Fee calculation and volume tracking | UUPS upgradeable |
| **PayRouter** | Entry point for x402 settlement | UUPS upgradeable |

Fund-holding contracts (PayDirect, PayTab) are immutable -- no proxy, no admin key, no upgrade path.

- [Contracts & Networks](/contracts)
- [GitHub](https://github.com/pay-skill/pay-protocol)

### Dashboard (Funding)

Web interface for wallet funding (Coinbase Onramp) and USDC withdrawal. Accessed via `pay fund` and `pay withdraw` CLI commands which generate time-limited links.

## How x402 Works End-to-End

1. **Agent sends request** to a Pay-gated API (via SDK, CLI, or MCP).
2. **pay-gate returns 402** with `PAYMENT-REQUIRED` header containing price, accepted payment schemes, and settlement mode (direct or tab).
3. **Agent SDK parses 402.** For tab settlement, it checks for an existing tab with this provider. If none, it auto-opens one (min $5, sized at 10x the per-call price).
4. **Agent signs payment proof.** For direct: EIP-3009 `transferWithAuthorization`. For tab: submits charge against the tab and includes the charge ID.
5. **Agent retries** with `PAYMENT-SIGNATURE` header containing the base64-encoded proof.
6. **pay-gate forwards proof** to the facilitator's `/verify` endpoint.
7. **Facilitator validates** the proof (signature recovery, balance check, charge verification). Off-chain, instant.
8. **pay-gate proxies** the request to the provider's API with `X-Pay-Verified`, `X-Pay-From`, `X-Pay-Amount`, `X-Pay-Settlement`, and `X-Pay-Tab` headers.
9. **Provider serves the response.** The agent gets the API data.
10. **Facilitator settles asynchronously.** Direct: on-chain transfer (~2s). Tab: charge buffered, settled in batches.

Steps 2-9 happen transparently when using `wallet.request()` or `pay request`.

## Settlement Modes

### Direct Settlement

One on-chain USDC transfer per request. For payments of $1.00 or more.

- Agent signs EIP-3009 `transferWithAuthorization` (off-chain)
- Facilitator submits the transfer on-chain (~2 seconds on Base)
- Provider receives amount minus 1% fee
- Works with any x402-compatible agent SDK

### Tab Settlement

Pre-funded metered account. For micropayments at any price point.

- Agent opens a tab (locks USDC in the smart contract, $5 minimum)
- Provider charges the tab per API call (no minimum per charge)
- Charges are verified instantly, buffered, and settled on-chain in batches
- Provider payouts processed twice daily (5 AM / 5 PM UTC) once the minimum threshold ($0.10) is reached
- Either party can close the tab at any time -- unilateral, non-blocking
- Agent's risk is bounded by the tab balance and `maxChargePerCall` (enforced on-chain)

### When to Use Which

| Scenario | Settlement | Why |
|----------|-----------|-----|
| One-off task payment ($5+) | Direct | Simple, immediate, one transaction |
| Metered API access | Tab | Sub-cent charges, amortized gas |
| x402 paywall (provider chooses) | Either | Provider declares mode in 402 response |
| High-frequency agent-to-agent | Tab | Hundreds of charges, one settlement |
| A2A task with defined price | Direct | Known amount, immediate confirmation |

## Fee Summary

| Fee | Who Pays | Amount |
|-----|----------|--------|
| Direct payment processing | Provider | 1% (0.75% above $50k/month) |
| Tab activation | Agent | max($0.10, 1% of tab amount) |
| Tab charge processing | Provider | max($0.002, 1%) per charge |
| Tab top-up | Nobody | Free |
| Gas (all operations) | Protocol | Covered |

Fees are deducted automatically. The agent pays the listed price with no surprises. The provider receives revenue minus fees.
