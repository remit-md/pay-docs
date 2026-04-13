---
title: "Quickstart: x402 Direct Settlement — HTTP 402 Payments"
description: "Automatically pay for HTTP API calls using x402. When a server returns 402 Payment Required, the agent pays and retries with no manual logic."
---

# Quickstart: x402 Direct Settlement

Pay for HTTP API calls automatically using the x402 protocol. When a server returns `402 Payment Required`, the client pays and retries -- no manual payment logic needed.

## How It Works

1. Agent requests `GET /api/data` from a provider
2. Provider returns `402` with base64-encoded v2 payment requirements in `PAYMENT-REQUIRED` header
3. Client decodes requirements, reads `accepts[0].extra.settlement === "direct"`, pays via `send`
4. Client retries with `PAYMENT-SIGNATURE: base64(v2 PaymentPayload)` -- provider verifies and returns data

## Provider Setup

Add a facilitator URL to your server. When a request lacks payment proof, return 402:

```javascript
// Express example -- or use pay-gate for zero-code setup
app.get("/api/data", (req, res) => {
  if (!req.headers["payment-signature"]) {
    const paymentRequired = {
      x402Version: 2,
      resource: { url: `https://${req.hostname}${req.path}`, mimeType: "application/json" },
      accepts: [{
        scheme: "exact",
        network: "eip155:8453",
        amount: "1000000",
        asset: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",  // USDC on Base
        payTo: "0xYourProviderWallet",
        maxTimeoutSeconds: 60,
        extra: { name: "USDC", version: "2", facilitator: "https://pay-skill.com/x402", settlement: "direct" },
      }],
      extensions: {},
    };
    const encoded = Buffer.from(JSON.stringify(paymentRequired)).toString("base64");
    res.set("PAYMENT-REQUIRED", encoded);
    return res.status(402).json({ error: "payment_required", message: "$1.00 per request" });
  }
  // Payment verified -- serve content
  res.json({ data: "premium content" });
});
```

## Agent: Pay Automatically

::: code-group

```bash [CLI]
# GET
pay request https://provider.example.com/api/data
# => [200] {"data": "premium content"}

# POST with body
pay request -X POST -d '{"query":"test"}' https://provider.example.com/api/search
# => [200] {"results": [...]}
```

```typescript [TypeScript]
import { Wallet } from "@pay-skill/sdk";

const wallet = await Wallet.create();  // OS keychain (same key as CLI)

// One line -- payment is automatic
const response = await wallet.request("https://provider.example.com/api/data");
const data = await response.json();
console.log(data); // { data: "premium content" }
```

```python [Python]
from payskill import Wallet

wallet = Wallet()

# One line -- payment is automatic
response = wallet.request("https://provider.example.com/api/data")
print(response.json())  # { "data": "premium content" }
```

:::

## What Happened

1. Client sent `GET /api/data` -- got `402` with `PAYMENT-REQUIRED` header
2. Client decoded base64 v2 requirements, read `accepts[0].extra.settlement === "direct"`
3. Client signed an EIP-3009 `transferWithAuthorization` for $1.00 to the provider
4. Client retried with `PAYMENT-SIGNATURE: base64(v2 PaymentPayload)` containing the signed authorization
5. Provider (or facilitator) submitted the authorization on-chain and returned the content

## Next Steps

- [x402 Tab Settlement](./x402-tab) -- use tabs for repeated micropayments (cheaper per call)

::: details Using testnet?

Set `PAYSKILL_TESTNET=1` env var, or pass `--testnet` to CLI commands. Use `pay mint 100` to get testnet USDC.

:::
