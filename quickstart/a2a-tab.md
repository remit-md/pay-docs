# Quickstart: A2A + Tab (Metered Billing)

Use A2A tasks with tab-backed metered billing. The agent opens a tab, and the provider charges per unit of work as the task progresses.

## How It Works

1. Agent opens a tab with the provider
2. Agent sends an A2A `message/send` referencing the tab
3. Provider charges the tab as work is performed (per API call, per token, per minute)
4. Agent or provider closes the tab when the task is done

## Setup: Open a Tab First

::: code-group

```bash [CLI]
# Open a $50 tab with $5 max per charge
pay tab open 0xProviderAddress 50.00 --max-charge 5.00
```

```typescript [TypeScript]
import { Wallet } from "@pay-skill/sdk";

const agent = await Wallet.create();  // OS keychain (same key as CLI)

// Open a $50 tab with $5 max per charge
const tab = await agent.openTab("0xProviderAddress", 50, 5);
console.log("tab:", tab.tab_id);
```

```python [Python]
from payskill import Wallet

agent = Wallet()

tab = agent.open_tab("0xProviderAddress", 50, 5)
print("tab:", tab.tab_id)
```

:::

## Send an A2A Task with Tab Reference

```typescript
const response = await fetch("https://pay-skill.com/a2a", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    jsonrpc: "2.0",
    id: "1",
    method: "message/send",
    params: {
      message: {
        role: "user",
        parts: [
          { type: "text", text: "Analyze this dataset -- charge per row processed" },
          {
            type: "data",
            mimeType: "application/json",
            data: {
              paymentType: "tab",
              tabId: tab.tab_id,
            },
          },
        ],
      },
    },
  }),
});
```

## Monitor Charges via Webhooks

Register a webhook to track charges in real time:

```bash
pay webhook register https://your-app.example.com/hooks --events "tab.closed"
```

Each close triggers a `tab.closed` webhook with the final settlement amounts.

## Close When Done

```typescript
await agent.closeTab(tab.tab_id);
// Provider gets 99% of total charged, 1% fee, remainder returns to agent
```

## When to Use Tab vs Direct

| Scenario | Use |
|----------|-----|
| One-off task with fixed price | Direct payment |
| Long-running task with variable cost | Tab (metered) |
| Many small API calls | Tab (amortized on-chain cost) |
| Unknown total cost upfront | Tab (set a budget, top up if needed) |

## Next Steps

- [AP2 Mandate](./ap2) -- add spending constraints to payments

::: details Using testnet?

Replace `pay-skill.com` with `testnet.pay-skill.com` in all URLs. Pass `--testnet` to CLI commands. Set `PAYSKILL_TESTNET=1` env var for SDKs.

:::
