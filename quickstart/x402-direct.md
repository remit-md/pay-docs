# Quickstart: x402 Direct Settlement

Pay for HTTP API calls automatically using the x402 protocol. When a server returns `402 Payment Required`, the SDK pays and retries — no manual payment logic needed.

## How It Works

1. Agent requests `GET /api/data` from a provider
2. Provider returns `402` with base64-encoded v2 payment requirements in `PAYMENT-REQUIRED` header
3. SDK decodes requirements, reads `accepts[0].extra.settlement === "direct"`, pays via `payDirect`
4. SDK retries with `PAYMENT-SIGNATURE: base64(v2 PaymentPayload)` — provider verifies and returns data

## Provider Setup

Add a facilitator URL to your server. When a request lacks payment proof, return 402:

```javascript
// Express example — or use pay-gate for zero-code setup
app.get("/api/data", (req, res) => {
  if (!req.headers["payment-signature"]) {
    const paymentRequired = {
      x402Version: 2,
      resource: { url: `https://${req.hostname}${req.path}`, mimeType: "application/json" },
      accepts: [{
        scheme: "exact",
        network: "eip155:8453",
        amount: "1000000",
        asset: "0x833589fCD6eDb6E08f4c7C32D4f71b54bdA02913",
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
  // Payment verified — serve content
  res.json({ data: "premium content" });
});
```

## Agent: Pay Automatically

::: code-group

```typescript [TypeScript]
import { PayClient } from "@pay-skill/sdk";

const client = new PayClient({
  apiUrl: "https://testnet.pay-skill.com/api/v1",
  privateKey: process.env.PAYSKILL_KEY!,
  chainId: 84532,
  routerAddress: "0x24F26eCb1f46451994c59585817e87896749935D",
});

// One line — payment is automatic
const response = await client.request("https://provider.example.com/api/data");
const data = await response.json();
console.log(data); // { data: "premium content" }
```

```python [Python]
from payskill import PayClient

client = PayClient(
    api_url="https://testnet.pay-skill.com/api/v1",
    signer="raw",
    private_key="0xYOUR_KEY",
    chain_id=84532,
    router_address="0x24F26eCb1f46451994c59585817e87896749935D",
)

# One line — payment is automatic
response = client.request("https://provider.example.com/api/data")
print(response.json())  # { "data": "premium content" }
```

```bash [CLI]
# GET
pay request https://provider.example.com/api/data
# => [200] {"data": "premium content"}

# POST with body
pay request -X POST -d '{"query":"test"}' https://provider.example.com/api/search
# => [200] {"results": [...]}
```

:::

## What Happened

1. SDK sent `GET /api/data` — got `402` with `PAYMENT-REQUIRED` header
2. SDK decoded base64 v2 requirements, read `accepts[0].extra.settlement === "direct"`
3. SDK called `payDirect` to send $1.00 to the provider
4. SDK retried with `PAYMENT-SIGNATURE: base64(v2 PaymentPayload)` containing the transaction proof
5. Provider verified payment and returned the content

## Next Steps

- [x402 Tab Settlement](./x402-tab) — use tabs for repeated micropayments (cheaper per call)
