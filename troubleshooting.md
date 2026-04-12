# Troubleshooting

## Common Errors

| Error | Meaning | Fix |
|-------|---------|-----|
| `INSUFFICIENT_BALANCE` | Wallet or tab balance too low | Run `pay fund` to add USDC, or `pay tab topup <id> <amount>` |
| `BELOW_MINIMUM` | Direct payment < $1.00 or tab open < $5.00 | Increase the amount. For sub-$1 payments, use tab settlement via x402 |
| `TAB_DEPLETED` | Tab balance is zero | `pay tab topup <id> <amount>` then retry |
| `NONCE_REUSED` | Duplicate payment nonce | CLI handles this on retry. If using SDK, generate a new nonce |
| `RATE_LIMITED` | Too many requests in the rate window | Wait the `retry_after` period. Do not loop |
| `INVALID_ADDRESS` | Malformed recipient address | Must be 42 characters starting with `0x`, valid hex |
| `SELF_PAYMENT` | Sending to your own address | Use a different recipient |
| `TAB_NOT_FOUND` | Tab ID doesn't exist | Check `pay tab list` for valid IDs |
| `UNAUTHORIZED` | Auth failure or wrong wallet | Run `pay address` to verify your wallet. May need `pay init` |
| `PROVIDER_ONLY` | Non-provider tried to charge a tab | Only the tab's provider address can charge it |
| `MANDATE_VIOLATION` | AP2 mandate bounds exceeded | Payment exceeds mandate constraints. Renegotiate the mandate |

## Debugging a Failed Payment

### Step 1: Check the error

```bash
# Verbose output shows full request/response headers
pay request -v https://api.example.com/data
```

The `-v` flag shows:
- The 402 response with payment requirements
- The payment proof your agent signed
- The facilitator's verify response
- The final API response (or error)

### Step 2: Check your wallet

```bash
pay status
```

Verify you have sufficient USDC balance and check your open tabs.

### Step 3: Common scenarios

**"I get 402 but no payment happens"**

Your SDK or CLI isn't handling the x402 flow. Check:
- Is the SDK installed? (`npm list @pay-skill/sdk` or `pip show payskill`)
- Are you using `wallet.request()` (not raw `fetch`/`requests`)?
- Is your wallet initialized? (`pay address` should return an address)
- Does your wallet have USDC? (`pay status`)

**"Payment signed but provider rejects it"**

The facilitator returned `is_valid: false`. Check:
- Is the payment amount correct? (matches the 402 requirement)
- Is your wallet funded? (balance >= payment amount)
- For tab settlement: is the tab still open? (`pay tab list`)
- For tab settlement: does the charge exceed `maxChargePerCall`?

**"Tab charge rejected"**

Possible causes:
- Tab is closed (check `pay tab list`)
- Tab balance exhausted (check remaining balance, top up with `pay tab topup`)
- Charge exceeds `maxChargePerCall` (set at tab open, enforced on-chain)
- Wrong provider (only the tab's provider can charge it)

**"Insufficient funds"**

```bash
# Check balance
pay status

# Fund via Coinbase Onramp (zero fee)
pay fund

# Or top up an existing tab
pay tab topup <tab_id> 10.00
```

**"Wrong network"**

If you initialized on testnet but are trying to reach a mainnet API (or vice versa):

```bash
# Check current network
pay network

# CLI defaults to mainnet. Use --testnet for testnet commands.
pay status --testnet
```

The SDK defaults to mainnet. Testnet requires explicit configuration.

## Rate Limits

| Action | Limit |
|--------|-------|
| Tab opens | 10/minute per wallet |
| Direct payments | 120/minute per wallet |
| Tab charges | No limit (bounded by balance + maxChargePerCall) |
| Webhook registrations | 10/hour per wallet |
| Discovery search | 60/minute per IP |

If rate limited, wait the `retry_after` period. Do not retry in a loop.

## Getting Help

- [API Reference](/api-reference) -- endpoint details, request/response formats
- [CLI Reference](/cli/) -- all commands and flags
- [GitHub Issues](https://github.com/pay-skill/pay-cli/issues) -- bug reports
