# pay-docs

Documentation site for Pay -- the complete x402 payment stack for AI agents. Served at [pay-skill.com/docs/](https://pay-skill.com/docs/).

## Development

VitePress site. The `base` is set to `/docs/` in `.vitepress/config.ts`.

```bash
npm install
npm run dev
```

## Structure

```
docs/
  index.md              -- landing page
  architecture.md       -- system overview + component diagram
  contracts.md          -- network addresses
  integrations.md       -- framework integration guide
  provider-guide.md     -- how providers earn
  webhooks.md           -- event notifications
  troubleshooting.md    -- common errors + debugging
  api-reference.md      -- REST API reference
  cli/                  -- CLI command reference
  gate/                 -- pay-gate docs (overview, quickstart, config, guide)
  sdk/                  -- TypeScript + Python SDK reference
  quickstart/           -- 8 scenario-specific quickstarts
  skills/               -- Claude Code skill references
```

## Part of Pay

Pay is the complete x402 payment stack -- gateway, facilitator, SDKs, CLI, and MCP server -- that lets AI agents pay for APIs with USDC on Base.

- [SDK](https://github.com/pay-skill/pay-sdk) -- Python + TypeScript
- [CLI](https://github.com/pay-skill/pay-cli) -- Command-line tool
- [pay-gate](https://github.com/pay-skill/gate) -- x402 payment gateway
- [MCP Server](https://github.com/pay-skill/mcp) -- Claude Desktop / Cursor / VS Code
- [Protocol](https://github.com/pay-skill/pay-protocol) -- Smart contracts
