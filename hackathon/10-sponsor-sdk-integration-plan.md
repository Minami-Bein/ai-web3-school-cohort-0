# Week 3 | Sponsor Workshop | SDK / API Integration Plan

> Updated: 2026-06-08
> Project: Cat-Agent
> Status: Sprint 3 In Progress

---

## Integration Overview

| Sponsor | SDK / API | Purpose | Priority | Deadline |
|---------|-----------|---------|----------|----------|
| Z.AI | Multimodal Inference API (Vision + Audio + Text) | Cat intent analysis | P0 | Week 4 (6/25) |
| Cobo | CAW Warrant SDK + Safe{Wallet} API | Pet wallet auth + on-chain payment | P0 | Week 4 (6/25) |
| Safe | Safe{Wallet} API + SpendingLimitModule | Daily budget hard cap | P0 | Week 4 (6/25) |

---

## 1. Z.AI Multimodal Inference API

### What to Integrate

Use Z.AI multimodal inference to replace OpenAI Vision + Whisper for analyzing cat visual behavior (camera frames) and audio behavior (microphone).

**API Endpoints:**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| Multimodal inference | `POST /v1/multimodal` | Image + Audio -> Intent + Confidence |
| Streaming output | `WebSocket /v1/stream` | Real-time inference progress (optional) |
| Model list | `GET /v1/models` | Check account model permissions |
| Usage query | `GET /v1/usage` | Check API call volume and costs |

**Expected Request Format:**

```python
response = zai_client.multimodal(
    model="cat-intent-v1",
    inputs={
        "image": base64.b64encode(camera_frame).decode(),
        "audio": base64.b64encode(audio_wav).decode(),
    },
    parameters={"temperature": 0.1, "max_tokens": 512}
)

# Expected Response
{
    "intent": "buy_snack",
    "confidence_score": 0.94,
    "target_item": "tuna_can",
    "action": "buy",
    "reason": "High-freq meow + slow blink + near bowl",
    "processing_time_ms": 1240
}
```

### How to Integrate

```
Step 1: Get Z.AI API Key (contact Sponsor)
Step 2: Set ZAI_API_KEY in .env
Step 3: Install SDK or use REST API directly
Step 4: Wrap in src/perception/zai_client.py
Step 5: Test with mock data
Step 6: Replace with real Z.AI API calls
Step 7: Verify latency <2s, P99 <3s
```

### Week 4 Completion?

**Time Estimate: 9 hours total**

| Task | Hours | Latest Start | Dependency |
|------|-------|-------------|------------|
| Get API Key | 1h (waiting for approval) | 6/20 | Sponsor response |
| SDK integration (mock mode) | 2h | 6/21 | - |
| Response parsing + unit tests | 2h | 6/22 | API Key |
| Real API integration | 3h | 6/23 | API Key |
| Latency + error handling | 1h | 6/24 | Integration done |
| **Total** | **9h** | - | - |

**Conclusion: YES, completable by 6/24**

### Fallback (If Z.AI Unavailable)

**Primary Fallback: OpenAI GPT-4o + Whisper**

```python
# Fallback 1: OpenAI Vision + Whisper
# Pros: Stable API, complete docs, no application needed
# Cons: Higher cost (~USD 0.01/call vs USD 0.003/call with Z.AI)

vision_response = openai.Image.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": [
        {"type": "image", "data": base64_img, "format": "base64"},
        {"type": "text", "text": "Analyze cat behavior and emotion, output confidence score (0-100)"}
    ]}]
)
audio_text = whisper.transcribe(audio_bytes)
intent_response = openai.Chat.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": f"Cat said: {audio_text}, analyze intent"}]
)
```

**Fallback 2: Rule-Based Engine (No AI)**

```python
# When AI is completely unavailable, use rule-based fallback
# Pros: Zero cost, stable response
# Cons: Cannot handle complex scenarios, only simple triggers

def rule_based_intent(audio_freq, audio_duration, cat_pose):
    if audio_freq > 800 and audio_duration > 3 and cat_pose == "near_bowl":
        return {"intent": "buy_snack", "confidence_score": 0.85, "action": "buy"}
    elif audio_freq > 600 and cat_pose == "near_door":
        return {"intent": "go_out", "confidence_score": 0.75, "action": "buy"}
    return {"intent": "unknown", "confidence_score": 0.3, "action": "decline"}
```

**Decision Tree:**

```
Z.AI API available?
  YES -> Use Z.AI (lowest cost)
  NO (timeout/rate limit)
      OpenAI API available?
        YES -> Use GPT-4o + Whisper (degraded mode)
        NO -> Use rule-based engine (minimum fidelity)
  NO (auth failure)
      Use rule-based engine + notify admin
```

---

## 2. Cobo CAW Warrant SDK

### What to Integrate

Use Cobo CAW SDK to read on-chain Warrant configuration and execute Policy checks (Budget / Scope / Time Window triple verification).

**API Endpoints:**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| Warrant read | `GET /v1/warrants/{warrant_id}` | Get auth config from chain |
| Warrant verify | `POST /v1/warrants/verify` | Verify if payment is within auth scope |
| Warrant register | `POST /v1/warrants` | Write new Warrant to chain (init) |
| Warrant revoke | `POST /v1/warrants/{warrant_id}/revoke` | Owner revokes auth |

**Cat-Agent Warrant Config:**

```json
{
  "warrant_id": "cat-agent-001",
  "issuer": "0xOwnerWalletAddress",
  "agent": "0xSafeWalletAddress",
  "budget": {
    "daily_limit": "10000000",
    "per_tx_limit": "10000000",
    "currency": "USDC"
  },
  "scope": {
    "allowed_endpoints": ["https://api.z.ai/v1/multimodal", "https://api.cobo.com/v1/delivery"],
    "allowed_contracts": ["0xMerchantContractAddress"],
    "blocked_actions": ["approve", "permit", "transferFrom"]
  },
  "time_window": {
    "valid_from": 1757280000,
    "valid_until": 1767926400,
    "cooldown_seconds": 86400
  },
  "health_rules": {
    "max_consecutive_snack_days": 3,
    "required_main_food_ratio": 0.5
  }
}
```

### How to Integrate

```
Step 1: Apply for Cobo CAW Developer Access (contact Sponsor)
Step 2: Set COBO_CAW_KEY + COBO_API_ENDPOINT in .env
Step 3: Install Cobo CAW SDK or use REST API directly
Step 4: Implement Warrant read and verify in src/wallet/caaw_warrant.py
Step 5: Implement Policy Engine (Budget + Scope + Time Window checks)
Step 6: Deploy on testnet (Sepolia) and verify Warrant config
Step 7: Test all 8 interception scenarios (P-01 ~ P-08)
```

### Week 4 Completion?

**Time Estimate: 16 hours total**

| Task | Hours | Latest Start | Dependency |
|------|-------|-------------|------------|
| Get Cobo CAW Access | 2h (contact + waiting) | 6/18 | Sponsor response |
| CAW SDK integration (mock) | 3h | 6/19 | - |
| Policy Engine development | 4h | 6/20 | CAW SDK |
| Testnet Warrant deployment | 2h | 6/21 | Testnet USDC |
| 8 interception scenario tests | 3h | 6/22 | Warrant deployed |
| Integration + bug fixes | 2h | 6/23 | Full chain |
| **Total** | **16h** | - | - |

**Conclusion: RISKY but achievable**
- SDK + Policy Engine logic can be done by 6/22
- Main risk: Cobo approval wait time is uncertain
- Mitigation: Use local JSON config to simulate Warrant if SDK is delayed

### Fallback (If Cobo CAW Unavailable)

**Primary Fallback: Local JSON Config File**

```python
# Fallback: Use local JSON config to simulate on-chain CAW Warrant
# Pros: Independent of Cobo SDK, can develop and test standalone
# Cons: No on-chain immutability, for dev/demo only

class LocalWarrantStore:
    def __init__(self, config_path: str = "configs/caaw_templates.json"):
        with open(config_path) as f:
            self.warrants = json.load(f)

    def get_warrant(self, warrant_id: str) -> dict:
        return self.warrants.get(warrant_id)

    def verify_payment(self, warrant_id: str, payment: dict) -> dict:
        warrant = self.get_warrant(warrant_id)
        # Same logic as PolicyEngine.check()
        ...
```

**Decision Tree:**

```
Cobo CAW SDK available?
  YES -> Use Cobo official SDK (on-chain Warrant)
  NO (SDK not published / docs insufficient)
      Refer to Cobo GitHub reference implementation
      If REST calls still fail -> Use local JSON config
  NO (approval not granted)
      Use local JSON config + note in README "upgrade when Cobo support is available"
```

**Critical:** Even with Fallback, Policy Engine business logic (P-01 ~ P-08 interception scenarios) must be fully implemented to ensure zero changes when switching to real CAW SDK.

---

## 3. Safe{Wallet} API + SpendingLimitModule

### What to Integrate

Use Safe{Wallet} API to manage Session Keys and SpendingLimitModule for enforcing daily budget and per-transaction limits.

**API Endpoints:**

| Endpoint | Method | Purpose |
|----------|--------|---------|
| Create Session Key | `POST /safes/{safe_address}/session-keys` | Generate temp signing key for Agent |
| Get Safe info | `GET /safes/{safe_address}` | Query Safe balance + installed modules |
| Execute transaction | `POST /safes/{safe_address}/transactions` | Sign and execute USDC transfer with Session Key |
| Query tx history | `GET /safes/{safe_address}/transactions` | Get pet spending records |
| Estimate gas | `POST /v2/gas-station/{chain_id}/suggested-params` | Estimate transaction gas |

**Cat-Agent Safe Config:**

```
Safe Address: 0xCatSafeForAlice
Master Key Holder: Alice (only for init and revoke)
Session Key: CatAgentSessionKey (used by AI Agent, signing-only)
Installed Modules:
  SpendingLimitModule (daily budget 10 USDC, per-tx 10 USDC)
  AllowlistGuard (whitelist-only contract addresses)
```

### How to Integrate

```
Step 1: Deploy Safe{Wallet} on Sepolia testnet
        (use https://app.safe.global or Safe{Core} SDK)
Step 2: Install SpendingLimitModule + AllowlistGuard via Safe App
Step 3: Install Safe Python SDK:
        pip install safe-eth-py[safe-wallet-api]
        or use Safe Transaction Service API directly
Step 4: Implement Session Key management in src/wallet/safe_wallet.py
Step 5: Implement USDC transfer method
Step 6: Test SpendingLimitModule hard cap (single tx >10 USDC)
Step 7: Test AllowlistGuard interception (non-whitelist contract)
```

### Week 4 Completion?

**Time Estimate: 10 hours total**

| Task | Hours | Latest Start | Dependency |
|------|-------|-------------|------------|
| Safe deployment on Sepolia | 1h | 6/18 | Testnet ETH + Sepolia |
| Install SpendingLimitModule + AllowlistGuard | 1h | 6/19 | Safe deployed |
| Safe Python SDK integration | 2h | 6/19 | - |
| Session Key generation + management | 2h | 6/20 | SDK ready |
| USDC transfer + Etherscan verification | 2h | 6/21 | Session Key ready |
| SpendingLimitModule interception test | 2h | 6/22 | USDC transfer |
| **Total** | **10h** | - | - |

**Conclusion: YES, completable by 6/22**

### Fallback (If Safe{Wallet} API Unavailable)

**Primary Fallback: Hardhat / Foundry Local Testnet Simulation**

```python
# Fallback: Use local testnet (Hardhat node) to simulate Safe{Wallet} behavior
# Pros: Fully local, controllable, no external dependencies
# Cons: Addresses are not real Ethereum, for dev/test only

class MockSafeWallet:
    def __init__(self, initial_usdc: int = 100_000_000):  # 100 USDC
        self.usdc_balance = initial_usdc
        self.transactions = []
        self.spending_limit = 10_000_000   # 10 USDC
        self.daily_limit = 10_000_000      # 10 USDC
        self.daily_spent = 0
        self.whitelist = set()

    def execute_payment(self, to_address: str, amount_usdc: int) -> dict:
        if amount_usdc > self.spending_limit:
            raise PermissionError(f"SpendingLimitModule: per_tx_limit exceeded")
        if self.daily_spent + amount_usdc > self.daily_limit:
            raise PermissionError(f"SpendingLimitModule: daily limit exceeded")
        if to_address not in self.whitelist and self.whitelist:
            raise PermissionError(f"AllowlistGuard: address not in whitelist")

        self.usdc_balance -= amount_usdc
        self.daily_spent += amount_usdc
        tx_hash = f"mock_tx_{len(self.transactions)}"
        self.transactions.append({
            "hash": tx_hash, "to": to_address, "amount": amount_usdc, "status": "confirmed"
        })
        return {"hash": tx_hash, "status": "confirmed"}

    def add_to_whitelist(self, address: str):
        self.whitelist.add(address)
```

**Decision Tree:**

```
Safe{Wallet} API available?
  YES -> Use Safe official SDK (real on-chain transactions)
  NO (API rate limit/service down)
      Use MockSafeWallet to continue dev testing
      (不影响 Policy Engine and AI layer development)
  NO (Safe deployment issues)
      Use pre-deployed standard Safe on Sepolia
      Reference: https://docs.safe.global/getting-started/testnets
```

---

## Integration Timeline Overview

```
Week 3 (6/18-6/22) -- Integration Development
───────────────────────────────────────────────
6/18  Z.AI API Key apply  +  Safe Sepolia deploy
6/19  Z.AI SDK mock        +  SpendingLimitModule install
6/20  CAW Warrant apply    +  Safe Session Key generate
6/21  Z.AI real integration +  CAW Policy Engine dev
6/22  Full-chain mock test  +  Fallback switch verification

Week 4 (6/23-6/30) -- Verification Sprint
───────────────────────────────────────────────
6/23  Agent Trace verification  +  SDK call verification
6/24  Contract interaction     +  Permission control tests
6/25  Testnet transaction      +  Audit logging verification
6/26  Demo screenshots         +  Recording prep
6/27  Hackathon submission
```

---

## One-Sentence Summary

| SDK / API | Difficulty | Main Risk | Fallback | Week 4 Completion |
|-----------|-----------|-----------|---------|-------------------|
| Z.AI Multimodal | Low (REST API) | API approval time | OpenAI GPT-4o + Whisper | 95% |
| Cobo CAW | High (SDK docs dependent) | SDK not published / slow approval | Local JSON config | 80% |
| Safe{Wallet} | Medium (standard API) | Testnet deployment | Hardhat simulation | 95% |

**Highest Risk: Cobo CAW SDK docs and access approval. Recommend proactively contacting Sponsor by 6/18 for status update. Z.AI and Safe risks are manageable with mature fallbacks.**
