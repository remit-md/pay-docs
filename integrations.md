---
title: "AI Framework Integrations — Claude, LangChain, CrewAI, LlamaIndex"
description: "Connect Ᵽay to Claude Desktop, Cursor, LangChain, CrewAI, LlamaIndex, Vercel AI SDK, OpenAI, and Semantic Kernel via MCP."
---

# Framework Integrations

All frameworks below connect to the same MCP server: `@pay-skill/mcp`. Install once, use from any AI framework that supports the Model Context Protocol.

## Claude Desktop

Add to your Claude Desktop config file (`~/Library/Application Support/Claude/claude_desktop_config.json` on macOS, `%APPDATA%\Claude\claude_desktop_config.json` on Windows):

```json
{
  "mcpServers": {
    "pay": {
      "command": "npx",
      "args": ["-y", "@pay-skill/mcp"]
    }
  }
}
```

Restart Claude Desktop after saving. See the [Claude Desktop quickstart](/quickstart/claude-desktop) for testnet config and wallet import.

## Claude Code

```bash
claude mcp add pay -- npx -y @pay-skill/mcp
```

## Cursor

Add to `.cursor/mcp.json` in your project root:

```json
{
  "mcpServers": {
    "pay": {
      "command": "npx",
      "args": ["-y", "@pay-skill/mcp"]
    }
  }
}
```

## Windsurf

Add to `~/.codeium/windsurf/mcp_config.json`:

```json
{
  "mcpServers": {
    "pay": {
      "command": "npx",
      "args": ["-y", "@pay-skill/mcp"]
    }
  }
}
```

---

## LangChain

```bash
pip install langchain-mcp-adapters
```

```python
from langchain_mcp_adapters.client import MultiServerMCPClient

client = MultiServerMCPClient(
    {
        "pay": {
            "transport": "stdio",
            "command": "npx",
            "args": ["-y", "@pay-skill/mcp"],
        }
    }
)

tools = await client.get_tools()
```

## CrewAI

```bash
pip install crewai crewai-tools
```

```python
from crewai import Agent
from crewai_tools import MCPServerAdapter
from mcp import StdioServerParameters

server_params = StdioServerParameters(
    command="npx",
    args=["-y", "@pay-skill/mcp"],
)

with MCPServerAdapter(server_params) as tools:
    agent = Agent(
        role="Payment Agent",
        goal="Handle USDC payments on Base.",
        backstory="I can send payments, open tabs, and call paid APIs.",
        tools=tools,
    )
```

## LlamaIndex

```bash
pip install llama-index-tools-mcp
```

```python
from llama_index.tools.mcp import BasicMCPClient, McpToolSpec

client = BasicMCPClient("npx", args=["-y", "@pay-skill/mcp"])
tools = await McpToolSpec(client=client).to_tool_list_async()
```

## Vercel AI SDK

```bash
npm install @ai-sdk/mcp @modelcontextprotocol/sdk
```

```typescript
import { createMCPClient } from '@ai-sdk/mcp';
import { StdioClientTransport } from '@modelcontextprotocol/sdk/client/stdio.js';

const pay = await createMCPClient({
  transport: new StdioClientTransport({
    command: 'npx',
    args: ['-y', '@pay-skill/mcp'],
  }),
});

const tools = await pay.tools();
```

## OpenAI Agents SDK

```bash
pip install openai-agents
```

```python
from agents import Agent, Runner
from agents.mcp import MCPServerStdio

async with MCPServerStdio(
    name="pay",
    params={
        "command": "npx",
        "args": ["-y", "@pay-skill/mcp"],
    },
) as pay:
    agent = Agent(
        name="Payment Agent",
        instructions="Use Pay tools to handle USDC payments.",
        mcp_servers=[pay],
    )
    result = await Runner.run(agent, "Check my balance")
    print(result.final_output)
```

## Semantic Kernel

```bash
pip install semantic-kernel[mcp]
```

```python
from semantic_kernel import Kernel
from semantic_kernel.connectors.mcp import MCPStdioPlugin

async with MCPStdioPlugin(
    name="pay",
    description="USDC payments on Base",
    command="npx",
    args=["-y", "@pay-skill/mcp"],
) as pay_plugin:
    kernel = Kernel()
    kernel.add_plugin(pay_plugin)
```

---

## Docker

Run the MCP server as a container:

```bash
docker run -i ghcr.io/pay-skill/mcp:latest
```

Pass environment variables for testnet or wallet import:

```bash
docker run -i \
  -e PAY_NETWORK=testnet \
  ghcr.io/pay-skill/mcp:latest
```

Use in any framework that supports stdio by replacing the `npx` command with `docker run -i ghcr.io/pay-skill/mcp:latest`.

## Environment Variables

All configurations above accept environment variables:

| Variable | Description | Default |
|----------|-------------|---------|
| `PAY_NETWORK` | `mainnet` or `testnet` | `mainnet` |
| `PAYSKILL_SIGNER_KEY` | Hex private key (skip auto-generation) | Auto-generated |

## Available Tools

Once connected, your agent has access to 15 tools:

| Tool | Description |
|------|-------------|
| `pay_status` | Check wallet balance and open tabs |
| `pay_direct` | Send USDC to an address ($1 min) |
| `pay_request` | Hit a paywalled API (handles x402 automatically) |
| `pay_discover` | Search for paid services by keyword |
| `pay_tab_open` | Open a pre-funded metered tab |
| `pay_tab_charge` | Charge against an open tab |
| `pay_tab_close` | Close a tab and settle funds |
| `pay_tab_topup` | Add funds to an open tab |
| `pay_tab_list` | List all open tabs |
| `pay_send` | Send a one-shot payment to an address |
| `pay_fund` | Generate a funding link |
| `pay_withdraw` | Generate a withdrawal link |
| `pay_mint` | Mint testnet USDC (testnet only) |
| `pay_webhook_register` | Register a webhook |
| `pay_webhook_list` | List registered webhooks |
