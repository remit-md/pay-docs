# Quickstart: Direct Payment

Send a one-shot USDC payment from an agent to a provider. This is the simplest Pay flow.

## Prerequisites

- A wallet with USDC. Run `pay init` to create one, then `pay fund` to add USDC via Coinbase Onramp.

## 1. Send a Direct Payment

::: code-group

```bash [CLI]
pay direct 0xProviderAddress 5.00 --memo "invoice-42"
# => {"tx_hash":"0xabc...","status":"confirmed"}
```

```typescript [TypeScript]
import { Wallet } from "@pay-skill/sdk";

const wallet = await Wallet.create();  // OS keychain (same key as CLI)

const result = await wallet.send(
  "0xProviderAddress",  // recipient
  5,                    // $5.00
  "invoice-42",         // memo
);
console.log("tx:", result.tx_hash);
```

```python [Python]
from payskill import Wallet

wallet = Wallet()

result = wallet.send(
    to="0xProviderAddress",
    amount=5,               # $5.00
    memo="invoice-42",
)
print("tx:", result.tx_hash)
```

:::

## 2. Verify

```bash
pay status
```

## What Happened

1. The SDK signed an EIP-2612 **permit** granting the PayDirect contract approval to transfer USDC
2. The server submitted the permit on-chain, then called `payDirectFor`
3. PayDirect transferred `$5.00 * 0.99 = $4.95` to the provider and `$0.05` to the fee wallet
4. The server returned the transaction hash

## Next Steps

- [Tab Lifecycle](./tab) -- pre-funded metered billing
- [x402 Direct Settlement](./x402-direct) -- automatic HTTP paywall payments

::: details Using testnet?

Switch to Base Sepolia for development:

```bash
pay network testnet
pay mint 100.00
pay direct 0xProviderAddress 5.00 --memo "invoice-42" --testnet
```

SDK: set `PAYSKILL_TESTNET=1` environment variable.

:::
