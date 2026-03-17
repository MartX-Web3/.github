# Echo Protocol Lab

**The permission layer for AI agents on Ethereum.**

AI agents are actively managing on-chain assets right now. The infrastructure to do this safely does not exist at the protocol level. Echo builds it.

---

## What we build

Echo Protocol is an on-chain permission layer that lets users delegate bounded authority to AI agents — without handing over private keys, without manual approval for every transaction, and without custodial risk.

Users define a MetaPolicy once: per-token limits, daily caps, exploration budgets, allowed protocols. AI agents operate freely within those boundaries. Everything outside the policy is rejected on-chain, enforced by code, by the `EchoPolicyValidator` module — not by trust in Echo, the agent, or any intermediary.

```
User defines:  WETH maxPerOp 100 USDC · maxPerDay 500 USDC · explorationBudget 50 USDC
Agent does:    "buy 80 USDC of ETH"   →  validates  →  swap executes
Agent tries:   "buy 200 USDC of ETH"  →  rejected on-chain  →  nothing moves
```

---

## Repositories

| Repo | Description |
|---|---|
| [`echo-contracts`](https://github.com/echo-protocol-lab/echo-contracts) | Solidity smart contracts — PolicyRegistry, EchoPolicyValidator (ERC-7579), IntentRegistry, EchoAccount, EchoAccountFactory |
| [`echo-gateway`](https://github.com/echo-protocol-lab/echo-gateway) | Local execution gateway — MCP server, Proxy Adapter, Action Builder, Key Store |
| [`echo-sdk`](https://github.com/echo-protocol-lab/echo-sdk) | TypeScript SDK — deploy accounts, manage policies, create sessions, read activity |
| [`echo-dashboard`](https://github.com/echo-protocol-lab/echo-dashboard) | React dashboard — onboarding, policy management, session monitoring, activity feed |

---

## Architecture

```
OpenClaw / LangChain / any agent framework
         │
         │  MCP tools: echo_submit_intent, echo_create_session, ...
         ▼
   Echo Gateway  ←── runs locally on user's machine
   ├── MCP Server          agent control plane
   ├── Action Builder      intent → protocol calldata (recipient hardcoded)
   ├── Pre-validator       two-stage checks before UserOp is built
   └── Proxy Adapter       intercepts RPC, submits to Pimlico
         │
         │  ERC-4337 UserOperation
         ▼
   EchoPolicyValidator     on-chain final enforcer (ERC-7579 module)
   ├── Real-time mode      [0x01][executeKey]              — user present
   └── Session mode        [0x02][sessionId][sessionKey]   — autonomous task
         │
         │  SIG_VALIDATION_SUCCESS
         ▼
   Uniswap V3 / Aave / Curve  (Phase 3+)
```

The gateway is **headless, local-first, and agent-framework agnostic**. Any agent that can make HTTP calls can use Echo. Existing skills require zero code changes.

---

## Security model

Four non-negotiable on-chain properties, enforced by `EchoPolicyValidator`:

**S1 —** No operation outside MetaPolicy or SessionPolicy can execute.

**S2 —** Swap `recipient` must equal the user's `AccountERC7579`. Assets cannot go to third-party addresses under any circumstances. The Action Builder hardcodes this; the Validator independently re-verifies.

**S3 —** `globalTotalSpent` is append-only. Spend counters never decrease. Only `recordSpend()` (callable only by the Validator) writes to them.

**S4 —** Only the instance owner (user EOA) can modify a PolicyInstance. Agents cannot expand their own permissions.

**Session subset re-verification** — At validation time, `SessionPolicy ⊆ PolicyInstance` is re-verified against the current instance state. A session cannot execute operations that would exceed limits the user has since lowered.

---

## Deployment status

| Network | Status |
|---|---|
| Sepolia testnet | Live — Phase 1 demo contracts deployed |
| Mainnet | Requires formal third-party audit before deployment |

Demo: [echo-demo-pi.vercel.app](https://echo-demo-pi.vercel.app) · Website: [echopay.ai](https://echopay.ai)

---

## Development status

**Phase 1 (complete)** — PolicyRegistry, EchoPolicyValidator, Execute Key mechanism, Dashboard demo. 133 unit and integration tests passing.

**Phase 2 (in progress)** — EchoAccount (OZ AccountERC7579), EchoAccountFactory, dual-mode Validator with per-token limits and session support, Echo Gateway (MCP + Proxy), SDK, Dashboard onboarding, OpenClaw integration.

**Phase 3** — Additional protocols (Aave, Curve, GMX), Policy Marketplace, multi-chain deployment (Base, Arbitrum), formal audit, mainnet.

**Phase 4** — Remote Proxy with TEE isolation. Same security guarantees without local deployment.

---

## Team

**Sunny Shen** — CEO. BSc Computing, HKPolyU. President, NTU CCTF Alumni Association. 1st Prize Asia AI Startup Competition. AWS Most Commercially Valuable AI Award. ETHGlobal Bangkok & Singapore.

**Dr. Jeff Ma** — CTO. PhD candidate, NTU Centre in Computational Technologies for Finance. ETH Foundation PhD Fellowship proposal. Expert in ERC-4337, EIP-712, ERC-7579. Internal security review lead.

**Kaid Yin** — CMO. Web3 growth and ecosystem.

---

## Contributing

Echo is in active development toward a Sepolia public launch. If you are building AI agent infrastructure, DeFi tooling, or ERC-7579 modules and want to collaborate, reach out.

[echopay.ai](https://echopay.ai)
