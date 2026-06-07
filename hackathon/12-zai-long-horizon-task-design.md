# Z.AI | Web3 x Long-Horizon Task Track
## AI Agent Complex Task Decomposition, Tool Calling and Web3 Workflow Delivery

> Updated: 2026-06-08
> Project: Cat-Agent
> Sponsor Track: Z.AI | Web3 x Long-Horizon Task

---

## 1. Core Problem: Why Cat-Agent Needs Long-Horizon Task Capability

Cat-Agent's payment task seems simple (confirm intent -> pay) but is actually a multi-step, multi-tool, multi-condition complex workflow:

```
Cat says "I want fish" (simple voice input)
  ↓
Decomposed into:
  1. Understand intent (multimodal AI inference)
  2. Check product inventory and price (external API)
  3. Check budget and permissions (CAW Warrant check)
  4. Construct payment transaction (Safe{Wallet})
  5. Broadcast to chain (Ethereum)
  6. Verify transaction confirmation (RPC query)
  7. Trigger delivery (3rd party API)
  8. Record audit log (IPFS + chain Event)
  9. Update RLPF weights (local learning)
  10. Notify owner (Telegram)

Without task decomposition and self-repair, any single failure crashes the entire workflow.
```

Z.AI's Long-Horizon Task capability enables Cat-Agent to: transform vague requests into executable plans, call tools continuously, self-repair on errors, and complete end-to-end Web3 workflow delivery.

---

## 2. Cat-Agent Workflow Architecture: Three-Layer Structure

### 2.1 Task Layers

```
Strategy Layer (Z.AI LLM Agent)
  └── Understand cat intent → Decompose task plan → Manage execution state

Tool Layer (MCP Tools)
  └── Z.AI Multimodal API / Alchemy RPC / Cobo CAW API / Safe API

Execution Layer (On-chain / Off-chain services)
  └── Ethereum / IPFS / Telegram / Delivery Platform API
```

### 2.2 Data Flow Between Layers

```
User/Cat Input
        ↓
Strategy Layer (Z.AI LLM):
  Understand input → Decompose into subtasks → Judge success/failure → Decide retry/skip/abort
        ↓
Tool Layer (MCP):
  Strategy calls specific tool → Tool returns result → Strategy judges result
        ↓
Execution Layer (On-chain/Off-chain):
  Actual executor of tool calls → Returns raw results
```

---

## 3. Task Decomposition: ReAct Pattern in Cat-Agent

### 3.1 Cat-Agent ReAct Loop

ReAct = Reasoning + Acting. Every Cat-Agent payment task follows this loop:

```
Thought:
  "Cat intent confidence is 0.94, above 0.90 threshold. Cat wants tuna can (2 USDC).
   I need to check budget, construct transaction, broadcast to chain, wait for confirmation."

Action:
  call_policy_engine(warrant_id="cat-agent-001", amount=2000000)

Observation:
  policy_engine returns {"passed": true, "daily_remaining": "8000000"}

Thought:
  "Policy check passed, 8 USDC remaining in daily budget. Now construct and sign transaction."

Action:
  call_safe_wallet(safe_address="0xCatSafeForMimi", to="0xMerchant", amount=2000000)

Observation:
  safe_wallet returns {"tx_hash": "0xabc123...", "status": "pending"}

Thought:
  "Transaction broadcast, waiting for confirmation. Check status in 15 seconds."

Action:
  wait_for_confirmation(tx_hash="0xabc123...", timeout=30)

Observation:
  confirmation returns {"block_number": 12345678, "status": "confirmed"}

Thought:
  "Transaction confirmed. Now log audit trail and notify owner."

Action:
  call_audit_log(ipfs_data=..., tx_hash="0xabc123...")
  call_telegram_notify(message="Mimi bought tuna can (2 USDC)")

Observation:
  audit_log returns {"ipfs_cid": "QmXyz..."}
  telegram returns {"message_id": 12345}

Final Answer:
  "Task complete. Mimi's tuna can arriving in ~30 minutes."
```

### 3.2 Task State Machine

```
PENDING → RUNNING → WAITING_CONFIRMATION → COMPLETED
              ↓              ↓
              ERROR ──────────▶ RETRY
              ↓
           FAILED (retries exhausted or non-recoverable)
```

---

## 4. Tool Calling: Cat-Agent MCP Toolset

### 4.1 Tool Inventory

| Tool Name | Provider | Function | Latency |
|-----------|----------|----------|---------|
| multimodal_intent | Z.AI | Image + Audio → Intent + Confidence | ~1.2s |
| policy_check | Self-built Policy Engine | Budget + Scope + Time Window triple check | ~50ms |
| safe_execute_payment | Safe{Wallet} API | Session Key sign + broadcast USDC transfer | ~5s |
| wait_for_confirmation | Alchemy RPC | Wait for chain transaction confirmation | ~15s |
| log_to_ipfs | IPFS API | Upload audit log, return CID | ~2s |
| emit_chain_event | Safe{Wallet} | On-chain Event anchoring (optional) | ~15s |
| send_telegram | Telegram Bot API | Notify owner of transaction result | ~1s |
| update_rlpf | Self-built RLPF | Update preference weights | ~100ms |
| query_product | Merchant API | Query product inventory and price | ~500ms |
| trigger_delivery | Delivery Platform API | Trigger delivery | ~3s |

### 4.2 Tool Call Dependency Graph

```
multimodal_intent (start)
        ↓
policy_check
        │ passed
        ↓
query_product
        │ in_stock
        ↓
safe_execute_payment
        │ tx_hash
        ↓
wait_for_confirmation
        │ confirmed
        ├──────────────────┐
        ↓                  ↓
log_to_ipfs            send_telegram
        ↓                  ↓
emit_chain_event      update_rlpf
        ↓                  ↓
     COMPLETED         COMPLETED
```

---

## 5. Iterative Repair: Cat-Agent Error Handling

### 5.1 Error Classification and Handling Strategy

| Error Type | Example | Strategy | Max Retries |
|-----------|---------|---------|------------|
| Network timeout | Z.AI API timeout | Exponential backoff retry (2s → 4s → 8s) | 3 |
| API rate limit | Z.AI rate limit | Wait 30s then retry | 3 |
| Chain tx failure | Gas insufficient | Auto retry with higher gas | 2 |
| Tx unconfirmed | Timeout no confirmation | RPC query retry confirmation | 3 |
| Budget insufficient | Daily budget exhausted | Terminate workflow, notify owner | 0 |
| Scope violation | Non-whitelist call | Log and terminate, notify owner | 0 |
| Warrant expired | valid_until passed | Terminate workflow, prompt renewal | 0 |
| Product out of stock | query_product returns out_of_stock | Notify owner, terminate workflow | 0 |

### 5.2 Retry Loop Example

```python
# src/agent/retry_loop.py

async def execute_with_retry(tool_call: ToolCall, max_retries: int = 3) -> ToolResult:
    """
    Retry loop with exponential backoff.
    """
    backoff = 2  # seconds
    last_error = None

    for attempt in range(max_retries + 1):
        try:
            result = await tool_call.execute()

            # Business errors (budget_exceeded) do not retry, return directly
            if result.is_business_error:
                return result

            return result

        except RetryableError as e:
            last_error = e
            if attempt < max_retries:
                await asyncio.sleep(backoff)
                backoff *= 2  # Exponential backoff
            continue

        except NonRetryableError as e:
            return ToolResult(error=str(e), retryable=False)

    # Retries exhausted
    return ToolResult(
        error=f"Max retries ({max_retries}) exhausted: {last_error}",
        retryable=False,
        final_attempt=True
    )
```

### 5.3 Self-Repair Example

**Scenario: Transaction broadcast times out without confirmation**

```
Thought:
  "Tx 0xabc123 broadcast 30s ago without confirmation. Could be low gas or network congestion.
   Need to check actual chain status."

Action:
  call_alchemy_get_transaction_receipt(tx_hash="0xabc123")

Observation:
  receipt returns null (tx still in mempool)

Thought:
  "Tx not confirmed. Need to re-query or increase gas priority."

Action (repair):
  call_eth_gas_price()  # Get current gas price
  → Found current gas 50% higher than at broadcast time

Action (repair):
  Rebroadcast same tx with higher gas price (same nonce)
  → Miner prioritizes new tx, old tx gets replaced

Observation:
  15s later, new tx_hash = 0xdef456, block_number = 12345700

Final:
  "Transaction confirmed (original tx replaced)."
```

---

## 6. Complete Web3 Workflow: From Request to Delivery

### 6.1 Workflow Phases

```
Phase 1: Requirement Understanding (~3s)
  Input: Cat camera + microphone raw data
  Process: Z.AI multimodal inference → intent + confidence
  Output: { intent, confidence_score, target_item, action }

Phase 2: Task Planning (~200ms)
  Input: Intent analysis result
  Process: Z.AI LLM decomposes subtasks, generates execution plan
  Output: [ { tool: "policy_check", args: {...} }, { tool: "safe_execute", args: {...} }, ... ]

Phase 3: Execution Check (~25s)
  Input: Execution plan
  Process: Call tools in order, check each result
  Output: tx_hash + confirmation

Phase 4: Delivery Confirmation (~5s)
  Input: Chain confirmation result
  Process: IPFS archive + Telegram notify + RLPF update
  Output: Completion message + delivery ETA

Phase 5: Post-Processing (~10s)
  Trigger delivery
  Update product inventory
  Record pet behavior feedback (RLPF)
```

### 6.2 Complete Sequence Diagram

```
Cat-Agent Strategy Layer (Z.AI LLM)
        │
        │ 1. multimodal_intent(camera_frame, audio_data)
        ↓
Z.AI Multimodal API
        │
        │ Returns: { intent: "buy_snack", confidence: 0.94, target: "tuna" }
        ↓
Cat-Agent Strategy Layer
        │
        │ 2. policy_check(warrant_id, amount=2000000)
        ↓
Policy Engine
        │
        │ Returns: { passed: true, daily_remaining: 8000000 }
        ↓
Cat-Agent Strategy Layer
        │
        │ 3. query_product(item_id="tuna-can")
        ↓
Merchant API
        │
        │ Returns: { in_stock: true, price: 2 USDC, delivery_time: 30min }
        ↓
Cat-Agent Strategy Layer
        │
        │ 4. safe_execute_payment(to="0xMerchant", amount=2000000)
        ↓
Safe{Wallet} (Session Key signing)
        │
        │ Returns: { tx_hash: "0xabc...", status: "pending" }
        ↓
Cat-Agent Strategy Layer
        │
        │ 5. wait_for_confirmation(tx_hash="0xabc...", timeout=30)
        ↓
Alchemy RPC
        │
        │ Returns: { block_number: 12345678, status: "confirmed" }
        ↓
Cat-Agent Strategy Layer (parallel execution)
        │
        ├── 6a. log_to_ipfs(intent_log, tx_receipt)
        │            ↓
        │    IPFS
        │            │
        │            │ Returns: { ipfs_cid: "QmXyz..." }
        │
        ├── 6b. send_telegram("Mimi bought tuna can (2 USDC), arriving ~30min")
        │            ↓
        │    Telegram Bot
        │            │
        │            │ Returns: { ok: true, message_id: 12345 }
        │
        └── 6c. update_rlpf(cat_id="mimi-001", item_id="tuna-can", feedback="ate_all")
                     ↓
              RLPF System
                     ↓
              Cat-Agent Strategy Layer
                     ↓
              Final Output: "Task complete. Delivery in progress, RLPF weights updated."
```

---

## 7. Long-Horizon Capability Details

### 7.1 What is Long-Horizon Task

Long-Horizon Task requires multiple rounds of reasoning, multiple tool calls, potentially spanning minutes to hours. Challenges vs single tool call:

1. **State Maintenance**: Intermediate results need persistence to resume after interruption
2. **Error Accumulation**: Any single step failure may invalidate all previous work
3. **Resource Management**: Many API calls = high cost, need to control
4. **Explainability**: Every decision step needs logging for audit

### 7.2 How Cat-Agent Implements Long-Horizon

**1. Task State Persistence**

```python
# Every task has unique task_id, all intermediate state stored to IPFS
class TaskState:
    task_id: str
    cat_id: str
    phase: str  # PENDING / RUNNING / COMPLETED / FAILED
    steps: list[StepRecord]  # Input/output/error for each step
    created_at: timestamp
    updated_at: timestamp

class StepRecord:
    step_id: int
    thought: str
    action: str
    observation: str
    result: ToolResult
    error: str | None
    retry_count: int
```

**2. Resume from Checkpoint**

```
Workflow execution interrupted (network disconnect / Agent restart)
        ↓
Check TaskState on IPFS
        ↓
Find last completed Step, continue from next Step
        ↓
No need to restart entire workflow
```

**3. Cost Control**

```python
# Z.AI API cost = tokens x unit price
# Cat-Agent controls cost via:

1. Intent inference optimization: low temperature (0.1), reduces random retries
2. Tool-first: use API calls when possible instead of LLM reasoning
3. Result caching: same input Z.AI response cached for 5 minutes
4. Degradation strategy: fall back to rule engine (zero cost) when Z.AI unavailable
```

**4. Explainability**

Every step's `thought` and `observation` fully recorded, owner can see what Cat-Agent is "thinking":

```
Step 5: wait_for_confirmation
Thought: "Tx broadcast 20s ago, need to confirm if on chain"
Action: call_alchemy_get_transaction_receipt(tx_hash="0xabc...")
Observation: "receipt returns null, tx still in mempool"
Thought: "Tx not confirmed, may need to wait longer or increase gas"
Action: (wait 10s then requery)
...
```

### 7.3 Multi-Session Tasks

Some tasks span multiple days, e.g., cat "Monthly Health Report":

```
Day 1-30:
  Cat-Agent records each purchase to local DB
        ↓
Day 30:
  Owner triggers "Generate monthly report"
        ↓
Cat-Agent Strategy Layer:
  1. Query 30-day purchase records (local DB)
  2. Call Z.AI to analyze diet structure changes (multimodal)
  3. Generate PDF report
  4. Upload to IPFS
  5. ERC-712 sign report summary
  6. Notify owner report is ready
```

---

## 8. Deep Z.AI Integration

### 8.1 Z.AI Role in Cat-Agent Workflow

```
Z.AI provides:
  1. Multimodal inference (camera + microphone → intent)
  2. LLM Strategy Layer (task decomposition, ReAct loop, error reasoning)
  3. Tool call orchestration (decide which tool to call, in what order)
  4. Long-term state management (multi-step task state persistence)
```

### 8.2 Z.AI Long-Horizon Capabilities in Cat-Agent

| Z.AI Capability | Cat-Agent Application Scenario |
|----------------|-------------------------------|
| Multi-turn context maintenance | ReAct loop chain reasoning |
| Tool call orchestration | Call policy_check → safe_execute → wait_confirmation in order |
| Error retry reasoning | Judge if error is retryable, choose repair strategy |
| State persistence | TaskState stored to IPFS, resume from checkpoint |
| Cost optimization | Low temperature + result caching + degradation strategy |
| Explainable output | Complete thought/observation chain for every step |

### 8.3 Z.AI Unique Value for Cat-Agent

1. **Real-time performance**: ~1.2s multimodal inference latency meets cat interaction real-time requirements
2. **Native multimodal**: Process image + audio simultaneously, no need to stitch multiple single-modal APIs
3. **Native tool calling**: Z.AI supports MCP protocol natively, Cat-Agent tools can connect via standard interface
4. **Long context**: Cat-Agent ReAct loop may produce 20+ reasoning steps, Z.AI context window is sufficient
5. **x402 payment**: Z.AI inference API can be protected by x402, enabling pay-per-call

---

## 9. Summary: Cat-Agent Web3 Workflow Delivery Capability

```
Cat-Agent is not a "one-click order" simple script
It is an AI Agent with Long-Horizon capability

It can:
  - Understand vague requests (cat meowing = wants something)
  - Decompose complex tasks (10+ step subtask decomposition)
  - Call tools continuously (Z.AI / Safe / Alchemy / IPFS / Telegram)
  - Iteratively repair errors (exponential backoff retry + self-repair)
  - Maintain execution state (TaskState + IPFS persistence)
  - Complete end-to-end delivery (request → chain tx → delivery notify → feedback learning)

Key guarantees:
  - Every step has thought/observation record (auditable)
  - Every step has error handling and retry strategy (recoverable)
  - Chain transactions have SpendingLimitModule hard cap (secure)
  - Full workflow has IPFS + chain Event dual archive (verifiable)
```