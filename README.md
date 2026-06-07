# Cat-Agent（喵星智能体）

> AI Agent 自主消费 + Cobo Agentic Wallet 授权管理
> A pet's autonomous spending agent powered by AI perception and on-chain warrant controls

[![Hackathon Cohort 0](https://img.shields.io/badge/Hackathon-AI%20x%20Web3%20School%20Cohort%200-blue)](https://github.com/Minami-Bein/ai-web3-school-cohort-0)
[![Status](https://img.shields.io/badge/status-Week%204%20Sprint-orange)](./hackathon/16-week4-daily-sprint.md)

---

## Problem

Pets cannot express spending needs. When a cat is hungry, it meows — but the owner cannot always respond immediately. Existing pet food ordering requires human intervention: the owner must receive the signal, open an app, search, confirm payment.

**Core tension:** Pets have urgent consumption needs, but they lack financial agency and cannot directly access payment systems. This is an AI Agent problem at its core — how to delegate financial authority to an AI that can perceive the world and act within controlled boundaries.

The deeper problem is **trust**: How do owners let an AI Agent hold a wallet and spend money on behalf of their pet, without risking runaway spending or malicious use?

---

## Track

**Primary:** Cobo — Agentic Economy × Cobo Agentic Wallet

**Secondary:** Z.AI — Web3 × Long-Horizon Task

> Cat-Agent uses Cobo's Agentic Wallet concept as the financial control layer. Each pet has its own Warrant (authorization scope) that defines what the Agent can pay for, within what budget, and for how long. The Z.AI multimodal inference layer provides the perceptual "eyes and ears" for understanding what the pet actually wants.

---

## MVP Flow

```
[Cat Mimi] --meows + approaches food bowl-->
[Cat-Agent Perception] --camera + microphone-->
[Z.AI Multimodal Inference] --intent: "buy_snack", confidence: 0.94-->
[Policy Engine] --Warrant check: Budget + Scope + Time Window-->
[Safe{Wallet}] --Session Key signs USDC transfer (2 USDC)-->
[Ethereum Sepolia] --tx broadcast + confirmation-->
[Telegram] --notify owner: "Mimi bought tuna can (2 USDC), daily budget remaining: 8 USDC"-->
[Audit Log] --intent_log + tx_hash saved to logs/-->
```

### Minimal闭环

```
Input: Cat behavior (audio + video)
  ↓
AI Perception: Z.AI / OpenAI multimodal intent inference
  ↓
Authorization: Cobo CAW Warrant / local JSON (Budget + Scope + Time)
  ↓
Wallet: Safe{Wallet} with SpendingLimitModule (daily: 10 USDC, per-tx: 10 USDC)
  ↓
Chain: Ethereum Sepolia (USDC transfer)
  ↓
Notification: Telegram text to owner
  ↓
Audit: Local JSON log (IPFS-ready)
```

Full architecture diagram: [08-architecture-minimal-loop.md](./hackathon/08-architecture-minimal-loop.md)

---

## Tech Stack

### Core Components

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Perception** | Z.AI Multimodal API / OpenAI GPT-4o + Whisper | Cat intent inference from camera + microphone |
| **Strategy** | Z.AI LLM Agent (ReAct loop) | Task decomposition, tool orchestration, self-repair |
| **Authorization** | Cobo CAW Warrant SDK / local JSON | On-chain budget/scope/time triple-check |
| **Wallet** | Safe{Wallet} + SpendingLimitModule + Session Key | On-chain wallet with hard spending limits |
| **Chain** | Ethereum Sepolia + USDC | Transaction execution and confirmation |
| **Notification** | Telegram Bot API | Real-time push to owner |
| **Audit** | Local JSON logs (IPFS-ready) | Intent + transaction record archive |
| **Fallback** | Rule-based intent engine / Hardhat mock | Degradation when external APIs unavailable |

### Project Structure

```
cat-agent/
├── src/
│   ├── perception/
│   │   ├── zai_client.py       # Z.AI API wrapper
│   │   ├── openai_fallback.py  # OpenAI GPT-4o fallback
│   │   └── mock_intent.py      # Rule-based mock intent
│   ├── agent/
│   │   ├── react_loop.py       # ReAct reasoning loop
│   │   └── task_state.py       # Task state persistence
│   ├── wallet/
│   │   ├── safe_wallet.py      # Safe{Wallet} + Session Key
│   │   ├── caaw_warrant.py     # Cobo CAW Warrant client
│   │   ├── policy_engine.py    # Budget + Scope + Time checks
│   │   └── mock_safe.py        # Hardhat-style mock wallet
│   ├── tools/
│   │   ├── mcp_tools.py        # MCP tool definitions
│   │   └── retry_loop.py       # Exponential backoff retry
│   ├── audit/
│   │   └── logger.py           # Audit log writer (local JSON)
│   └── notify/
│       └── telegram_bot.py     # Telegram notification
├── configs/
│   ├── warrants/               # Local Warrant JSON (CAW fallback)
│   └── products.json            # Mock product catalog
├── logs/                        # Audit logs (gitignored)
├── hackathon/                   # All design docs
│   ├── 07-week4-verification.md
│   ├── 10-sponsor-sdk-integration-plan.md
│   ├── 11-cobo-agentic-wallet-design.md
│   ├── 12-zai-long-horizon-task-design.md
│   ├── 13-project-prerequisites-risk-fallback.md
│   ├── 15-scope-boundary.md
│   └── 16-week4-daily-sprint.md
├── .env.example
└── README.md
```

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|-----------|
| Z.AI API Key not approved in time | 40% | Perception layer fails | Switch to OpenAI Fallback immediately |
| Cat behavior data is erratic / confidence below threshold | 35% | Frequent "intent unknown" responses | Lower threshold to 0.80, add multi-frame fusion, manual trigger fallback |
| Safe SpendingLimitModule misconfiguration | 25% | All transactions blocked or none blocked | Deploy on testnet first, run P-01~P-08 checklist |
| Cobo CAW Access not granted | 25% | Policy Engine uses local JSON only | Policy Engine fully decoupled from CAW SDK; local JSON still validates business logic |
| Sepolia testnet USDC faucet rate-limited | 15% | Insufficient test funds | Multiple faucets备选; contact sponsor for dedicated test tokens |

Full risk analysis: [13-project-prerequisites-risk-fallback.md](./hackathon/13-project-prerequisites-risk-fallback.md)

---

## Validation Plan

### Checkpoint Matrix (P0 = Must Have, P1 = Should Have)

| Category | Checkpoint | Type | Evidence |
|----------|-----------|------|---------|
| **Agent Trace** | T-01~T-05 | P0 | Screenshot of each reasoning step |
| **SDK Calls** | S-01~S-06 | P1 | Code comments + actual output |
| **Testnet TX** | N-01~N-05 | P0 | Etherscan tx hash + block confirmation |
| **Contract Interaction** | C-01~C-06 | P0 | SpendingLimitModule interception screenshots |
| **Permission Control** | P-01~P-08 | P1 | 8 interception scenarios documented |
| **Audit Logging** | A-01~A-06 | P1 | Local JSON log files |
| **Demo Evidence** | D-01~D-08 | P0 | Screenshots / screencast |

Full Checkpoint spec: [07-week4-verification.md](./hackathon/07-week4-verification.md)

### Week 4 Daily Progress

| Date | Core Task | Minimum Success Criteria |
|------|-----------|--------------------------|
| 6/23 (Mon) | Full chain run-through | 1 successful tx on Etherscan |
| 6/24 (Tue) | Checkpoint matrix completion | 5 P0 Checkpoints with screenshots |
| 6/25 (Wed) | Fallback freeze | No task blocked by external dependency |
| 6/26 (Thu) | Demo materials | Screencast + screenshots + README |
| 6/27 (Fri) | Final submission | Hackathon platform submission confirmed |

Full sprint plan: [16-week4-daily-sprint.md](./hackathon/16-week4-daily-sprint.md)

---

## Design Principles

```
1. Minimum Privilege: Agent never holds master key; only Session Key with warrant-scoped permissions
2. Defense in Depth: Policy Engine (app-layer) + SpendingLimitModule (chain-layer) dual checks
3. Auditable: Every operation (success/failure) logged with thought + observation chain
4. Recoverable: All external dependencies have fallback; task state persisted for resume
5. Scope Discipline: 3 items cut/deferred/mock for Week 4 to avoid scope creep
   - Cut: Multi-pet coordination, RLPF auto-update, real delivery API
   - Deferred: IPFS archiving, ERC-712 report signing, Telegram rich notifications
   - Mock: Z.AI (until API Key arrives), Cobo CAW (until Access granted), product catalog
```

Full scope boundary: [15-scope-boundary.md](./hackathon/15-scope-boundary.md)

---

## Sponsor Integration

| Sponsor | SDK / API | Purpose | Fallback |
|---------|-----------|---------|---------|
| Z.AI | Multimodal Inference API | Cat intent analysis (camera + audio) | OpenAI GPT-4o + Whisper |
| Cobo | CAW Warrant SDK | On-chain budget/scope/time authorization | Local JSON Warrant |
| Safe | Safe{Wallet} + SpendingLimitModule | Daily budget hard cap (10 USDC) | Hardhat mock |

Full integration plan: [10-sponsor-sdk-integration-plan.md](./hackathon/10-sponsor-sdk-integration-plan.md)

---

## Questions for Sponsor / Mentor

1. **Z.AI**: What is the application process and typical timeline for API Key? Is there a test credit or sandbox available?
2. **Cobo**: How mature is the CAW SDK documentation? Is there a Python SDK or REST API reference? What is the Access approval process?
3. **Safe**: What is the correct configuration JSON for SpendingLimitModule (daily 10 USDC, per-tx 10 USDC)? How does it interact with AllowlistGuard?

Full Q&A: [14-questions-for-sponsors-mentors.md](./hackathon/14-questions-for-sponsors-mentors.md)

---

## Team

- **Minami-Bein** — AI × Web3 School Cohort 0

## License

MIT