---
title: "Smart Contract Addresses on Base — Mainnet and Testnet"
description: "Deployed Ᵽay contract addresses on Base mainnet and Sepolia testnet. PayRouter, PayDirect, PayTab, and USDC. Always fetch at runtime."
---

# Contracts & Networks

Pay deploys smart contracts on Base. **Always fetch addresses at runtime** from the `/api/v1/contracts` endpoint -- never hardcode them.

## Fetch Contract Addresses

```bash
curl https://pay-skill.com/api/v1/contracts
```

Response:

```json
{
  "chain_id": 8453,
  "router": "0x...",
  "direct": "0x...",
  "tab": "0x...",
  "tab_v2": "0x...",
  "usdc": "0x..."
}
```

## Networks

| Network | Chain | Chain ID | API URL |
|---------|-------|----------|---------|
| **Mainnet** | Base | 8453 | `https://pay-skill.com/api/v1` |
| Testnet | Base Sepolia | 84532 | `https://testnet.pay-skill.com/api/v1` |

The CLI and SDKs default to mainnet. No network configuration needed for production.

## Contract Roles

| Contract | Role |
|----------|------|
| **PayRouter** (`router`) | Entry point for x402 settlement. Receives EIP-3009 authorizations, splits payment between provider and fee wallet. |
| **PayDirect** (`direct`) | One-shot USDC transfers. Agent permits, server calls `payDirectFor`. |
| **PayTab** (`tab`) | Pre-funded metered accounts (v1). Agent locks USDC, provider charges per use. |
| **PayTabV2** (`tab_v2`) | Metered accounts with batch settlement. Charges are buffered off-chain, settled in batches. |
| **PayFee** | Fee calculation and volume tracking. Cliff-based tiers: 1% below $50k/month, 0.75% at/above. Not returned by `/contracts` -- called internally by other contracts. |
| **USDC** (`usdc`) | Circle's USDC stablecoin on Base. ERC-20 with EIP-2612 permit and EIP-3009 transferWithAuthorization. |

## Using Contracts in Code

### TypeScript

```typescript
import { Wallet } from "@pay-skill/sdk";

// Wallet.create() auto-fetches contracts and caches them internally
const wallet = await Wallet.create();  // OS keychain (same key as CLI)
await wallet.send("0xProvider...", 5);
```

### Python

```python
from payskill import Wallet

# Wallet() auto-fetches contracts and caches them internally
wallet = Wallet()  # OS keychain (same key as CLI)
wallet.send("0xProvider...", 5)
```

### CLI

The CLI fetches contract addresses automatically during `pay init`. No manual configuration needed.

## x402 Provider Setup

When building x402 payment requirements, use the addresses from `/contracts`:

```javascript
const contracts = await fetch("https://pay-skill.com/api/v1/contracts")
  .then(r => r.json());

const paymentRequired = {
  x402Version: 2,
  accepts: [{
    scheme: "exact",
    network: `eip155:${contracts.chain_id}`,
    amount: "1000000",
    asset: contracts.usdc,        // USDC address from /contracts
    payTo: "0xYourProviderWallet",
    maxTimeoutSeconds: 60,
    extra: {
      name: "USDC",
      version: "2",
      facilitator: "https://pay-skill.com/x402",
      settlement: "direct",
    },
  }],
};
```

::: details Using testnet?

```bash
# Testnet contracts
curl https://testnet.pay-skill.com/api/v1/contracts

# CLI
pay network testnet
pay network mainnet      # switch back
```

SDK: set `PAYSKILL_TESTNET=1` env var, or pass `testnet: true` to the Wallet constructor.

:::
