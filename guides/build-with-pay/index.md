---
title: "Build with Pay — Production Guide for Ᵽay Integrations"
description: "Seven-page guide on shipping a Ᵽay integration: choosing an integration layer, wallet key management, tab lifecycle, spending controls, error handling, production checklist, and settlement mode."
---

# Build with Pay

A production-oriented guide for developers (and AI coding agents building apps for developers) wiring Ᵽay into a real application. Each page is one topic you need to understand before launch, and each builds on the ones before it.

For the API surface of any individual SDK or middleware package, see the per-framework pages under [Application Middleware](/middleware/) or the [fetch() Wrapper](/sdk/fetch). This guide sits one layer above those: it is about **how the pieces fit together in production**, not about the packages themselves.

## The Seven Pages

1. **[Choosing Your Integration](/guides/build-with-pay/choosing)** — Before you pick a middleware package, decide whether you need middleware at all. Five common scenarios and the right integration layer for each: CLI, `createPayFetch`, consumer middleware, provider middleware, or pay-gate.

2. **[Wallet Key Management in Production](/guides/build-with-pay/key-management)** — Where the wallet private key comes from in each environment: OS keychain for local dev, `PAYSKILL_KEY` env var for Docker, CI, Kubernetes, Vercel, Lambda, Cloud Run. Key rotation walkthrough and the anti-patterns list.

3. **[Tab Lifecycle in Long-Running Services](/guides/build-with-pay/tab-lifecycle)** — Two patterns for tab management (let the SDK auto-open, or persist tab IDs to a database and top up on `tab.low_balance`). Tab sizing by call volume and the shutdown decision.

4. **[Spending Controls & Budgeting](/guides/build-with-pay/spending-controls)** — Three layers of spend safety: `maxPerRequest`, `maxTotal`, and application-level per-provider caps via the `onPayment` callback. Monitoring spend via the audit trail, `wallet.status()`, and webhooks.

5. **[Error Handling for x402](/guides/build-with-pay/error-handling)** — Every error class the SDK raises, what causes it, and the recovery pattern that fits. Full catch-all handler in TypeScript and Python.

6. **[Going to Production](/guides/build-with-pay/production)** — Seven-item pre-launch checklist tying the earlier pages together. Latency budget table and a six-step pre-launch dry run against a small real-USDC staging wallet.

7. **[Choosing Settlement Mode](/guides/build-with-pay/settlement-mode)** — Tab vs. direct settlement as a provider-side decision. Three worked cost comparisons. When to mix modes per route.

## Who This Guide Is For

- **Application developers** shipping a service that calls paid APIs (consumer side) or charges for paid APIs (provider side)
- **AI coding agents** generating code for developers who have never touched x402 before
- **Operators** taking over a Ᵽay-enabled service and needing the production-concerns cheat sheet

It is not for:

- Agent-with-wallet use cases that want to call APIs from a shell or an MCP tool — see the [CLI reference](/cli/) and the [Claude Desktop quickstart](/quickstart/claude-desktop) instead
- One-off scripts that call one paid API and exit — see the [fetch() Wrapper](/sdk/fetch) directly
- Providers who just want a reverse proxy in front of an existing API — see [pay-gate](/gate/)

Each of those has a shorter path to "working." This guide is for the cases where the short path is not enough.

## Conventions Across the Guide

Every page uses the same conventions so you can skim them predictably:

- **Mainnet is the default** in every example. Testnet is for internal development and is not advertised here.
- **`Wallet.create()` (OS keychain) is the recommended local-dev setup.** Production examples use `Wallet.fromEnv()` / `Wallet.from_env()` to fail loud if `PAYSKILL_KEY` is missing.
- **TypeScript and Python code is shown side-by-side** via `::: code-group` blocks where both languages apply.
- **No silent fallbacks.** Every recovery pattern either succeeds visibly or fails visibly. Swallowing a payment error to return default data is called out as an anti-pattern on multiple pages.
- **Cross-links between pages are one level deep.** Each page links back to the guide index and forward to the next deep dive, but no page assumes you have read every page before it.

## Where to Start

If you are starting from scratch, read [Choosing Your Integration](/guides/build-with-pay/choosing) first to figure out which layer you need. Most readers only end up touching a subset of the pages — for example, a consumer-only Next.js app can skip the provider-facing parts of [Settlement Mode](/guides/build-with-pay/settlement-mode) and the provider middleware on the overview page.

If you already have a working integration and want to ship it, jump to [Going to Production](/guides/build-with-pay/production) and work backwards through the links to the deep-dive pages it references.

If you are debugging a running integration, the two pages that cover the "why did this error happen" questions are [Error Handling for x402](/guides/build-with-pay/error-handling) and [Tab Lifecycle](/guides/build-with-pay/tab-lifecycle).
