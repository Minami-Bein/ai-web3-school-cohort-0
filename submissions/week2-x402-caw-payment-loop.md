# Week 2 Module B · 进阶任务
# x402 Paywall + Cobo CAW Agent 自主支付闭环设计

> 目标：展示服务提供方的 x402 保护 API + Agent 的自主支付 + CAW/Pact 预算控制 + 可审计记录
> 输出到 submissions/week2-x402-caw-payment-loop.md

---

## 一、系统架构总览

### 1.1 完整闭环架构图

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                        x402 Paywall + CAW Agent 自主支付闭环                              │
│                                                                                         │
│  ┌───────────────────────────────────────────────────────────────────────────────────┐   │
│  │                         服务提供方（Service Provider）                             │   │
│  │                                                                                   │   │
│  │   ┌─────────────────┐      ┌──────────────────┐      ┌─────────────────────┐   │   │
│  │   │  x402 Server    │      │  Payment Receipt │      │  Audit Logger       │   │   │
│  │   │  (受保护 API)   │─────►│  (链上验证)       │─────►│  (可审计记录)        │   │   │
│  │   │  Port: 8080    │      │  USDC 转账        │      │  IPFS + Events      │   │   │
│  │   └─────────────────┘      └──────────────────┘      └─────────────────────┘   │   │
│  │          │                                                                  │   │
│  │          │ x402: 401 + WWW-Authenticate: prepayment=x402                      │   │
│  └──────────┼──────────────────────────────────────────────────────────────────┘   │
│             │  HTTP Request + Response                                                  │
│  ┌──────────┴──────────────────────────────────────────────────────────────────┐       │
│  │                              Cobo CAW Agent（消费方）                             │       │
│  │  ┌────────────┐  ┌────────────┐  ┌────────────┐  ┌────────────┐          │       │
│  │  │  Intent    │  │  Policy    │  │  Pact      │  │  CAW       │          │       │
│  │  │  Resolver  │  │  Engine    │  │  Executor  │  │  Warrant   │          │       │
│  │  │  (意图解析) │  │  (预算/范围│  │  (支付执行) │  │  (操作授权) │          │       │
│  │  │             │  │  时间窗口) │  │            │  │            │          │       │
│  │  └────────────┘  └────────────┘  └────────────┘  └────────────┘          │       │
│  │        │                │               │                │                 │       │
│  │        ▼                ▼               ▼                ▼                 │       │
│  │  ┌────────────────────────────────────────────────────────────────────┐   │       │
│  │  │                      Safe{Wallet} with SpendingLimitModule          │   │       │
│  │  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                │   │       │
│  │  │  │ 日预算 50   │  │ 白名单:     │  │ 时间窗口:   │                │   │       │
│  │  │  │ USDC        │  │ x402 Server │  │ 7 天        │                │   │       │
│  │  │  │ 单笔 ≤ 10   │  │ 合约白名单  │  │ 自动过期    │                │   │
│  │  │  └─────────────┘  └─────────────┘  └─────────────┘                │   │       │
│  │  └────────────────────────────────────────────────────────────────────┘   │       │
│  └───────────────────────────────────────────────────────────────────────────────────┘       │
│                                                                                         │
│  ┌───────────────────────────────────────────────────────────────────────────────────┐   │
│  │                              基础设施层（Infrastructure）                            │   │
│  │                                                                                   │   │
│  │   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │   │
│  │   │  Escrow      │  │  ERC-8004   │  │  IPFS        │  │  RPC        │        │   │
│  │   │  (托管/结算) │  │  Identity    │  │  (证据存储)  │  │  (数据)      │        │   │
│  │   └──────────────┘  └──────────────┘  └──────────────┘  └──────────────┘        │   │
│  └───────────────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 交互流程时序图

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                            CAW Agent 自主支付完整时序                                     │
│                                                                                         │
│  Agent (CAW)              x402 Server          Safe{Wallet}         Escrow/链上         │
│     │                         │                      │                    │            │
│     │  1. API 请求            │                      │                    │            │
│     │────────────────────────►│                      │                    │            │
│     │                         │                      │                    │            │
│     │  2. 401 + WWW-Auth      │                      │                    │            │
│     │  + x402 amount          │                      │                    │            │
│     │◄────────────────────────│                      │                    │            │
│     │                         │                      │                    │            │
│     │  3. 解析 x402 header    │                      │                    │            │
│     │  识别付款金额 + 要求     │                      │                    │            │
│     │                         │                      │                    │            │
│     │  4. CAW Policy 检查      │                      │                    │            │
│     │     金额 < 日预算？      │                      │                    │            │
│     │     合约在白名单？       │                      │                    │            │
│     │     在时间窗口内？       │                      │                    │            │
│     │                         │                      │                    │            │
│     │  5. [通过]              │                      │                    │            │
│     │     调用 Pact.executor   │                      │                    │            │
│     │─────────────────────────│─────────────────────►│                    │            │
│     │                         │                      │                    │            │
│     │  6. 签名并广播           │                      │                    │            │
│     │     USDC transfer       │                      │                    │            │
│     │───────────────────────────────────────────────►│                    │            │
│     │                         │                      │                    │            │
│     │  7. 链上确认 + receipt  │                      │                    │            │
│     │◄───────────────────────────────────────────────│                    │            │
│     │                         │                      │                    │            │
│     │  8. Authorization: x402  │                      │                    │            │
│     │     <receipt>            │                      │                    │            │
│     │─────────────────────────►│                      │                    │            │
│     │                         │                      │                    │            │
│     │  9. 验证收据            │                      │                    │            │
│     │     链上检查 receipt    │                      │                    │            │
│     │                         │                      │                    │            │
│     │  10. 返回受保护资源      │                      │                    │            │
│     │◄────────────────────────│                      │                    │            │
│     │                         │                      │                    │            │
│     │  11. 记录审计日志       │                      │                    │            │
│     │     (IPFS + Events)     │                      │                    │            │
│     │                         │                      │                    │            │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 二、x402 Paywall 服务端设计

### 2.1 x402 协议核心机制

**x402 工作原理：**

```
HTTP Request Flow (x402 Payment Flow)

Client                              Server
  │                                    │
  │──── GET /api/resource ────────────►│
  │                                    │
  │◄─── 401 Unauthorized ──────────────│
  │      WWW-Authenticate: x402         │
  │      x402-amount: 5000000          │  ← 5 USDC (6 decimals)
  │      x402-address: 0xServer...      │  ← 收款方地址
  │      x402-data: {plan: "basic"}    │  ← 附加数据
  │
  │  [Client 检测到 x402 要求]
  │
  │──── POST /x402/pay ────────────────►│
  │      {amount, recipient, data}      │
  │                                    │
  │◄─── {receipt: "0x..."} ────────────│  ← 链上收据
  │
  │──── GET /api/resource ────────────►│
  │      Authorization: x402 <receipt>  │  ← 携带收据重试
  │                                    │
  │◄─── 200 OK + Resource ─────────────│
```

### 2.2 x402 Server 实现（Python/Flask 伪代码）

```python
# ============================================================
# x402 Server - 受保护的 AI 推理 API
# ============================================================

from flask import Flask, request, jsonify, abort
from web3 import Web3
import hashlib, json, time

app = Flask(__name__)
w3 = Web3(Web3.HTTPProvider("https://eth-mainnet.alchemyapi.io/v2/..."))

# ============================================================
# 智能合约配置
# ============================================================

USDC_ADDRESS  = "0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48"   # USDC Contract
ESCROW_ADDRESS = "0x..."  # ERC-8183 Escrow 合约地址

# USDC ABI (仅需要 transfer 和 balanceOf)
USDC_ABI = [
    {
        "inputs": [
            {"name": "to", "type": "address"},
            {"name": "amount", "type": "uint256"}
        ],
        "name": "transfer",
        "outputs": [{"type": "bool"}],
        "stateMutability": "nonpayable",
        "type": "function"
    },
    {
        "inputs": [{"name": "account", "type": "address"}],
        "name": "balanceOf",
        "outputs": [{"name": "", "type": "uint256"}],
        "stateMutability": "view",
        "type": "function"
    }
]

usdc = w3.eth.contract(address=USDC_ADDRESS, abi=USDC_ABI)
escrow = w3.eth.contract(address=ESCROW_ADDRESS, abi=ERC8183_ABI)

# ============================================================
# 服务定价表（x402 amount，单位：USDC 的最小单位 = 1e-6）
# ============================================================

PRICING = {
    "/api/ai/inference": {
        "base":  5000000,   # 5 USDC   （基础推理）
        "premium": 15000000, # 15 USDC  （高精度推理）
    },
    "/api/ai/report": {
        "single":  2000000,  # 2 USDC   （单份报告）
        "batch":  10000000,  # 10 USDC  （批量报告）
    }
}

# 服务端钱包（x402-address）
SERVER_WALLET = "0x742d35Cc6634C0532925a3b844Bc9e7595f"  # 服务商收款地址

# ============================================================
# 核心端点：受保护的 AI 推理 API
# ============================================================

@app.route("/api/ai/inference", methods=["GET", "POST"])
def ai_inference():
    """受 x402 保护的 AI 推理端点"""

    # ---- 步骤 1：检查 Authorization header ----
    auth_header = request.headers.get("Authorization", "")

    if auth_header.startswith("x402 "):
        # ---- 已有收据，验证并返回资源 ----
        receipt = auth_header[4:].strip()  # "0x..."
        if verify_receipt(receipt):
            return jsonify({
                "status": "success",
                "result": ai_inference_model(request.json),
                "receipt_used": receipt
            })
        else:
            abort(401, description="Invalid or expired receipt")
    else:
        # ---- 无收据，返回 401 + x402 要求 ----
        pricing_key = "premium" if request.args.get("quality") == "high" else "base"
        amount = PRICING["/api/ai/inference"][pricing_key]

        response = jsonify({
            "error": "payment_required",
            "message": "This endpoint requires x402 payment"
        })
        response.headers["WWW-Authenticate"] = \
            f'x402 x402-amount="{amount}", x402-address="{SERVER_WALLET}", x402-data="{{\\"endpoint\\":\\"/api/ai/inference\\"}}"'  # noqa
        response.status_code = 401
        return response


def verify_receipt(receipt: str) -> bool:
    """
    验证 x402 收据
    1. 解析收据哈希
    2. 在链上检查交易哈希是否存在
    3. 检查收款方 + 金额 + 时间戳
    """
    try:
        tx_hash, payment_amount, payer, payee = parse_receipt(receipt)

        # 链上验证：交易哈希存在
        tx = w3.eth.get_transaction(tx_hash)

        # 验证：收款方是服务端地址
        assert tx["to"] == SERVER_WALLET, "Wrong recipient"

        # 验证：金额足够支付
        assert payment_amount >= required_amount, "Insufficient payment"

        # 验证：交易未被使用过（防重放）
        assert not is_receipt_used(receipt), "Receipt already used"

        # 标记收据为已使用
        mark_receipt_used(receipt)

        return True

    except Exception as e:
        log_error(f"Receipt verification failed: {e}")
        return False


# ============================================================
# x402 支付端点
# ============================================================

@app.route("/x402/pay", methods=["POST"])
def x402_pay():
    """
    x402 支付端点：客户端调用此接口完成支付
    返回 payment receipt（链上交易哈希）
    """
    data = request.json
    required_amount = int(data["amount"])      # 单位：USDC smallest unit
    recipient       = data["recipient"]        # 服务端地址
    endpoint_data   = json.loads(data.get("data", "{}"))

    # ---- 步骤 1：验证参数 ----
    assert recipient == SERVER_WALLET, "Wrong recipient address"
    assert required_amount >= MINIMUM_PAYMENT, "Amount too low"

    # ---- 步骤 2：通过 Pact 执行支付 ----
    #   注意：这里实际是通过 Agent 的 Safe{Wallet} + Pact 执行
    #   Agent 调用此端点，传入 payer 参数，由服务端构造交易
    receipt = execute_x402_payment(
        payer=Agent_WALLET,       # Agent 钱包地址
        recipient=recipient,      # 服务端地址
        amount=required_amount,    # 金额
        data=endpoint_data
    )

    # ---- 步骤 3：记录审计日志 ----
    log_audit_event(
        event_type="x402_payment",
        receipt=receipt,
        payer=Agent_WALLET,
        recipient=recipient,
        amount=required_amount,
        endpoint=endpoint_data.get("endpoint"),
        timestamp=int(time.time())
    )

    return jsonify({
        "status": "success",
        "receipt": receipt,       # 链上交易哈希
        "tx_hash": receipt         # 交易哈希 = 收据
    })


# ============================================================
# 审计日志记录（Event + IPFS）
# ============================================================

def log_audit_event(**kwargs):
    """记录可审计事件到链上 + IPFS"""

    # 1. 写入 IPFS（完整事件数据）
    ipfs_cid = upload_to_ipfs(json.dumps(kwargs))

    # 2. 发送链上 Event（事件哈希）
    event_tx = escrow.functions.recordAuditLog(
        kwargs["event_type"],
        kwargs["receipt"],
        ipfs_cid,
        kwargs["timestamp"]
    ).transact({"from": SERVER_WALLET})

    # 3. 本地日志（降级备选）
    with open("/var/log/x402_audit.jsonl", "a") as f:
        f.write(json.dumps(kwargs) + "\n")

    return ipfs_cid
```

---

## 三、Cobo CAW Agent 消费方设计

### 3.1 Agent 架构

```
┌─────────────────────────────────────────────────────────────────┐
│                    Cobo CAW Agent 消费方架构                       │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                        User Request                        │  │
│  │                    "帮我分析钱包税务"                        │  │
│  └──────────────────────────┬────────────────────────────────┘  │
│                             │                                   │
│                             ▼                                   │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   Intent Resolver                          │  │
│  │           理解用户请求 → 判断需要哪些 API                     │  │
│  └──────────────────────────┬────────────────────────────────┘  │
│                             │                                   │
│            ┌────────────────┼────────────────┐                  │
│            ▼                ▼                ▼                  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐       │
│  │ 本地数据     │  │ x402 受保护   │  │ 无需付费的    │       │
│  │ 查询         │  │ API           │  │ 公开 API       │       │
│  └──────────────┘  └──────┬───────┘  └──────────────┘       │
│                            │                                    │
│                            ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              Payment Decision Engine                       │  │
│  │                                                               │  │
│  │   检测到 WWW-Authenticate: x402 ？                          │  │
│  │          │                                                  │  │
│  │          ├─ YES ──► CAW Policy 检查 ──► Pact 执行支付      │  │
│  │          │                                                  │  │
│  │          └─ NO  ──► 直接使用资源                            │  │
│  └──────────────────────────┬────────────────────────────────┘  │
│                             │                                    │
│                             ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                   CAW Warrant Engine                        │  │
│  │                                                               │  │
│  │   ┌─────────────┐  ┌─────────────┐  ┌─────────────┐      │  │
│  │   │  Budget     │  │  Scope       │  │  Time       │      │  │
│  │   │  日预算 50   │  │ 操作范围     │  │ 窗口 7 天   │      │  │
│  │   │  USDC       │  │ 白名单合约   │  │ 自动过期    │      │  │
│  │   └─────────────┘  └─────────────┘  └─────────────┘      │  │
│  └───────────────────────────────────────────────────────────┘  │
│                             │                                    │
│                             ▼                                    │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │              Safe{Wallet} (智能钱包)                        │  │
│  │                                                               │  │
│  │   Session Key → SpendingLimitModule → AllowlistGuard       │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### 3.2 Agent 核心伪代码

```python
# ============================================================
# Cobo CAW Agent - 自主支付消费方
# ============================================================

import httpx
import json
import time
from web3 import Web3
from typing import Optional

# ============================================================
# CAW Warrant 配置（用户在 Safe 中预设的权限）
# ============================================================

CAW_WARRANT = {
    "agent_id": "cobo_caw_agent_001",       # Agent 链上身份（ERC-8004）
    "payer": "0xAgentWallet...",            # Safe 钱包地址

    # Budget 限制
    "budget": {
        "daily_limit": 50_000_000,          # 50 USDC（单位：1e-6）
        "per_tx_limit": 10_000_000,          # 单笔上限 10 USDC
        "monthly_limit": 500_000_000,       # 月度硬上限
        "gas_limit": 0.1,                   # ETH 用于 gas
    },

    # 操作范围（白名单）
    "allowed_endpoints": [
        "https://api.taxservice.com/api/ai/inference",
        "https://api.taxservice.com/api/ai/report",
    ],

    # 时间窗口
    "validity": {
        "start": int(time.time()),
        "end": int(time.time()) + 7 * 24 * 3600,   # 7 天后过期
    },

    # 豁免确认的阈值（金额 <= 此值，自动执行）
    "auto_confirm_threshold": 3_000_000,    # 3 USDC 以下自动执行
}

# ============================================================
# Pact Executor - 支付执行器
# ============================================================

class PactExecutor:
    """通过 Safe{Wallet} 执行 x402 支付"""

    def __init__(self, warrant: dict):
        self.warrant = warrant
        self.w3 = Web3(Web3.HTTPProvider("https://eth-mainnet.alchemyapi.io/v2/..."))
        self.usdc = self.w3.eth.contract(USDC_ADDRESS, abi=USDC_ABI)

        # 统计计数器
        self.today_sent = 0
        self.tx_count = 0
        self.last_reset_date = time.strftime("%Y-%m-%d")

    def reset_if_new_day(self):
        """每天重置计数器"""
        today = time.strftime("%Y-%m-%d")
        if today != self.last_reset_date:
            self.today_sent = 0
            self.last_reset_date = today

    def check_budget(self, amount: int) -> bool:
        """检查是否在预算内"""
        self.reset_if_new_day()

        if self.today_sent + amount > self.warrant["budget"]["daily_limit"]:
            return False
        if amount > self.warrant["budget"]["per_tx_limit"]:
            return False
        return True

    def execute_payment(
        self,
        recipient: str,
        amount: int,
        endpoint: str
    ) -> dict:
        """
        执行 x402 支付
        1. 预算检查
        2. 白名单检查
        3. 时间窗口检查
        4. 链上转账
        5. 返回 receipt
        """

        # ---- 检查 1：预算限制 ----
        if not self.check_budget(amount):
            raise PaymentError(
                f"Budget exceeded: today {self.today_sent}, "
                f"limit {self.warrant['budget']['daily_limit']}"
            )

        # ---- 检查 2：白名单 ----
        if endpoint not in self.warrant["allowed_endpoints"]:
            raise PaymentError(f"Endpoint not in allowlist: {endpoint}")

        # ---- 检查 3：时间窗口 ----
        now = int(time.time())
        if not (self.warrant["validity"]["start"] <= now <= self.warrant["validity"]["end"]):
            raise PaymentError("Warrant expired or not yet valid")

        # ---- 检查 4：Gas 预算 ----
        gas_estimate = self.estimate_gas()
        if gas_estimate > self.warrant["budget"]["gas_limit"]:
            raise PaymentError(f"Gas estimate {gas_estimate} exceeds limit")

        # ---- 执行支付：USDC transfer ----
        # 使用 Safe{Wallet} 的 Session Key 签名（不是私钥）
        tx_hash = self.execute_usdc_transfer(recipient, amount)

        # 更新计数器
        self.today_sent += amount
        self.tx_count += 1

        # ---- 记录审计日志 ----
        self.log_payment(tx_hash, recipient, amount, endpoint)

        return {
            "receipt": tx_hash,      # 链上交易哈希 = 收据
            "amount": amount,
            "recipient": recipient,
            "tx_hash": tx_hash,
            "timestamp": int(time.time()),
            "agent_id": self.warrant["agent_id"],
        }

    def execute_usdc_transfer(self, to: str, amount: int) -> str:
        """通过 Safe{Wallet} 执行 USDC 转账（模拟）"""
        # 注意：实际使用 Safe 的 Session Key + ERC-4337 UserOperation
        # 这里简化为直接 sendTransaction（演示用）

        nonce = self.w3.eth.get_transaction_count(self.warrant["payer"])
        gas_price = self.w3.eth.gas_price

        tx = self.usdc.functions.transfer(
            to,
            amount
        ).build_transaction({
            "from": self.warrant["payer"],
            "nonce": nonce,
            "gas": 100000,
            "gasPrice": gas_price,
        })

        # 用 Session Key 签名（不是主私钥）
        signed_tx = self.sign_with_session_key(tx)

        tx_hash = self.w3.eth.send_raw_transaction(signed_tx)
        self.w3.eth.wait_for_transaction_receipt(tx_hash)

        return tx_hash.hex()

    def log_payment(self, tx_hash, recipient, amount, endpoint):
        """记录支付审计日志"""
        audit_record = {
            "type": "x402_payment",
            "agent_id": self.warrant["agent_id"],
            "tx_hash": tx_hash,
            "recipient": recipient,
            "amount_usdc": amount / 1_000_000,
            "endpoint": endpoint,
            "timestamp": int(time.time()),
            "daily_total": self.today_sent / 1_000_000,
            "tx_count": self.tx_count,
        }

        # 写入本地审计日志（降级方案）
        with open("/var/log/caw_audit.jsonl", "a") as f:
            f.write(json.dumps(audit_record) + "\n")

        # TODO: 写入 IPFS + 链上 Event


# ============================================================
# HTTP Client - 带 x402 感知能力的 httpx wrapper
# ============================================================

class X402AwareClient:
    """自动处理 x402 支付的 HTTP 客户端"""

    def __init__(self, pact_executor: PactExecutor):
        self.pact = pact_executor
        self.client = httpx.Client(timeout=30.0)
        self.used_receipts = set()     # 防重放

    def request(self, method: str, url: str, **kwargs) -> httpx.Response:
        """自动处理 x402 支付流程的请求方法"""

        # ---- 第一次请求（不带收据）----
        response = self.client.request(method, url, **kwargs)

        if response.status_code == 401:
            # ---- 检测到 x402 要求 ----
            authenticate = response.headers.get("WWW-Authenticate", "")
            if not authenticate.startswith("x402 "):
                raise PaymentError("Unexpected 401 without x402")

            # 解析 x402 header
            x402_params = self.parse_x402_header(authenticate)
            amount = int(x402_params["x402-amount"])
            recipient = x402_params["x402-address"]
            endpoint_data = json.loads(x402_params.get("x402-data", "{}"))

            # ---- CAW Policy 检查 ----
            self.validate_caw_policy(amount, url)

            # ---- 通过 Pact 执行支付 ----
            receipt_data = self.pact.execute_payment(
                recipient=recipient,
                amount=amount,
                endpoint=url
            )

            receipt = receipt_data["receipt"]
            self.used_receipts.add(receipt)

            # ---- 携带收据重试请求 ----
            kwargs["headers"] = kwargs.get("headers", {})
            kwargs["headers"]["Authorization"] = f"x402 {receipt}"

            response = self.client.request(method, url, **kwargs)

        return response

    def parse_x402_header(self, header: str) -> dict:
        """解析 WWW-Authenticate: x402 ... header"""
        params = {}
        # 格式：x402 x402-amount="5000000", x402-address="0x...", x402-data="{...}"
        for part in header.split(","):
            key, _, value = part.strip().partition("=")
            if value:
                params[key.strip()] = json.loads(value.strip('"'))
        return params

    def validate_caw_policy(self, amount: int, url: str):
        """验证 CAW Policy"""
        warrant = self.pact.warrant

        # 预算检查
        if not self.pact.check_budget(amount):
            raise PaymentError(
                f"Payment amount {amount} exceeds CAW daily budget "
                f"({warrant['budget']['daily_limit']})"
            )

        # 白名单检查
        if url not in warrant["allowed_endpoints"]:
            raise PaymentError(f"URL not in CAW allowlist: {url}")

        # 时间窗口检查
        now = int(time.time())
        if not (warrant["validity"]["start"] <= now <= warrant["validity"]["end"]):
            raise PaymentError("CAW Warrant expired or not yet valid")


# ============================================================
# TaxReportBot - 具体使用场景
# ============================================================

class TaxReportBot:
    """
    具体场景：TaxReportBot 需要调用受 x402 保护的 AI 推理 API
    来生成钱包税务报告
    """

    def __init__(self):
        self.pact = PactExecutor(CAW_WARRANT)
        self.client = X402AwareClient(self.pact)

    def generate_tax_report(self, wallet_address: str, year: int) -> dict:
        """
        生成钱包税务报告
        1. 调用公开 API 获取交易历史（免费）
        2. 调用受保护的 AI 推理 API 生成报告（x402 付费）
        """

        # ---- 第一步：获取交易历史（免费 API）----
        tx_history = self.fetch_transaction_history(wallet_address, year)

        # ---- 第二步：调用受保护的 AI 推理 API（x402 付费）----
        # X402AwareClient 自动处理支付流程
        ai_response = self.client.request(
            method="POST",
            url="https://api.taxservice.com/api/ai/report",
            json={
                "wallet": wallet_address,
                "year": year,
                "transactions": tx_history,
                "quality": "standard"
            }
        )

        return {
            "report": ai_response.json()["result"],
            "payment_receipt": self.client.used_receipts.pop(),
            "wallet": wallet_address,
            "year": year
        }

    def fetch_transaction_history(self, wallet: str, year: int) -> list:
        """获取钱包交易历史（免费 API）"""
        # 使用公开 RPC 或 The Graph（无需支付）
        response = self.client.request(
            method="GET",
            url=f"https://api.taxservice.com/api/tx/history?wallet={wallet}&year={year}"
        )
        return response.json()["transactions"]
```

---

## 四、CAW Warrant 机制详解

### 4.1 CAW Warrant 结构

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                         CAW Warrant（操作授权书）                                  │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐   │
│  │  Warrant ID: warrant_caw_001                                              │   │
│  │  Agent ID:  cobo_caw_agent_001 (ERC-8004 链上身份)                        │   │
│  │  Payer:    0xAgent...Safe{Wallet}                                        │   │
│  │  Issuer:   0xUser... (Alice 的主钱包签名授权)                              │   │
│  │  Created:  2026-05-28 00:00:00 UTC                                       │   │
│  │  Expires:  2026-06-04 00:00:00 UTC（7 天后自动过期）                      │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐   │
│  │                          Budget 限制                                      │   │
│  │                                                                          │   │
│  │  日预算：50 USDC     （单笔上限：10 USDC）                                 │   │
│  │  月预算：500 USDC   （硬上限，超出后自动暂停）                             │   │
│  │  Gas 预算：0.1 ETH  （每月）                                              │   │
│  │                                                                          │   │
│  │  ⚠️ 触发条件：日预算的 80%（40 USDC）时告警                               │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐   │
│  │                          Scope 限制（白名单）                              │   │
│  │                                                                          │   │
│  │  ✅ 允许调用的端点：                                                      │   │
│  │     • https://api.taxservice.com/api/ai/inference                        │   │
│  │     • https://api.taxservice.com/api/ai/report                           │   │
│  │                                                                          │   │
│  │  ❌ 禁止调用的端点：                                                      │   │
│  │     • 任何其他 API（默认拒绝）                                            │   │
│  │     • 任何 swap/transfer/approve 操作（除非通过 Escrow）                   │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐   │
│  │                          操作类型限制                                      │   │
│  │                                                                          │   │
│  │  ✅ Agent 可以自动执行：                                                  │   │
│  │     • 只读操作（balanceOf, eth_getLogs, API GET）                        │   │
│  │     • 金额 <= 3 USDC 的 x402 支付（自动确认）                             │   │
│  │                                                                          │   │
│  │  ⚠️ Agent 必须请求确认：                                                  │   │
│  │     • 金额 > 3 USDC 的 x402 支付（展示摘要，等待用户确认）                │   │
│  │     • 新合约地址（不在白名单）                                            │   │
│  │                                                                          │   │
│  │  ❌ Agent 永远不能执行：                                                   │   │
│  │     • approve / permit / transferFrom（授权类操作）                      │   │
│  │     • 合约部署                                                           │   │
│  │     • 任何非白名单端点的调用                                              │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
│                                                                                 │
│  ┌──────────────────────────────────────────────────────────────────────────┐   │
│  │                          时间窗口                                          │   │
│  │                                                                          │   │
│  │  有效时间：2026-05-28 00:00 UTC → 2026-06-04 00:00 UTC（7 天）            │   │
│  │                                                                          │   │
│  │  自动过期：当 timestamp > expires 时，所有授权自动失效                     │   │
│  │  提前撤销：Alice 可以随时在 Safe UI 中撤销                                │   │
│  └──────────────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 4.2 CAW Policy Engine 决策流程

```
用户请求 → Intent 解析 → 判断是否需要链上操作
                              │
                              ├── 只读（无需支付）→ 直接执行 → 返回结果
                              │
                              └── 写入 / 付费
                                      │
                                      ▼
                              检测 x402 要求
                              （解析 WWW-Authenticate header）
                                      │
                                      ▼
                              CAW Policy 检查
                                      │
                              ┌──────┴──────┐
                              │              │
                              ▼              ▼
                          通过              不通过
                              │              │
              ┌───────────────┼──────────────┘
              │               │
              ▼               ▼
          金额 <= 3 USDC?    金额 > 3 USDC?
              │               │
              ▼               ▼
          自动执行          展示摘要
          签名并广播        等待用户确认
              │               │
              │               ▼
              │           用户确认
              │               │
              ▼               ▼
          收据写入          签名并广播
          携带收据重试请求
              │
              ▼
          返回受保护资源
```

---

## 五、关键接口说明

### 5.1 x402 相关接口

| 接口 | 方法 | 方向 | 说明 |
|------|------|------|------|
| `/api/ai/inference` | GET/POST | Server → Agent | 受保护端点，返回 401 + x402 header |
| `/x402/pay` | POST | Agent → Server | 执行支付，返回 receipt |
| `Authorization: x402 <receipt>` | Header | Agent → Server | 携带收据重试请求 |

### 5.2 Agent 内部接口

| 接口 | 说明 |
|------|------|
| `PactExecutor.execute_payment()` | 验证 CAW Policy，执行链上 USDC 转账 |
| `X402AwareClient.request()` | 自动处理 x402 支付流程的 HTTP wrapper |
| `CAWPolicyEngine.validate()` | 验证 Budget / Scope / Time Window |
| `Safe{Wallet}.transfer()` | 通过 Session Key 执行受限的链上操作 |

### 5.3 链上合约接口

| 合约 | 接口 | 说明 |
|------|------|------|
| **USDC** | `transfer(to, amount)` | 转账 USDC |
| **ERC-8183 Escrow** | `createJob()`, `fund()`, `submit()`, `complete()` | 任务托管 |
| **ERC-8004 Identity** | `register()`, `updateReputation()` | Agent 身份 |
| **Safe Module** | `SpendingLimitModule.checkAndExecute()` | 预算强制校验 |

---

## 六、支付闭环完整流程

### 6.1 端到端场景：TaxReportBot 生成税务报告

```
场景：Alice 授权 TaxReportBot 生成 2025 年钱包税务报告
      报告生成需要调用受 x402 保护的 AI 推理 API（5 USDC）

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

Step 1: Alice 部署 CAW Warrant
  ├── 在 Safe{Wallet} 中配置 CAW Warrant
  │      • Agent: TaxReportBot（ERC-8004 身份）
  │      • 日预算: 50 USDC
  │      • 白名单: api.taxservice.com
  │      • 时间窗口: 7 天
  └── 签署授权，Session Key 部署到链上

Step 2: Agent 发起 API 请求
  └── TaxReportBot 收到 Alice 的请求："生成 2025 年税务报告"

Step 3: Server 返回 401 + x402 要求
  ├── GET /api/ai/report
  ├── Server 检测无 Authorization header
  └── 返回 401 + WWW-Authenticate: x402 x402-amount="5000000"

Step 4: Agent 检测 x402 要求
  ├── X402AwareClient 解析 header
  ├── 提取 amount=5 USDC, recipient=0xServer...
  └── 调用 CAW Policy Engine 检查

Step 5: CAW Policy 检查通过
  ├── 金额 5 USDC < 日预算 50 USDC ✓
  ├── 端点在白名单 ✓
  ├── 在时间窗口内 ✓
  └── 5 USDC > 自动确认阈值 3 USDC → 展示摘要给 Alice 确认

Step 6: Alice 确认（自动/手动）
  ├── Alice 看到交易摘要：支付 5 USDC 给 api.taxservice.com
  ├── 如果金额 <= 3 USDC → 自动执行（无需确认）
  └── 如果金额 > 3 USDC → Alice 手动确认

Step 7: Pact Executor 执行支付
  ├── Safe{Wallet} 构造 USDC transfer 交易
  ├── Session Key 签名（不是主私钥）
  ├── 广播到网络，链上确认
  └── 返回 tx_hash（= receipt）

Step 8: 携带收据重试请求
  ├── Authorization: x402 0xtxhash...
  ├── Server 验证收据（链上检查 tx_hash）
  └── Server 返回受保护的 AI 推理结果

Step 9: 审计记录
  ├── Server 记录 payment receipt + 审计日志
  ├── Agent 记录 tx_hash + 使用量到本地日志
  └── Alice 收到通知："税务报告已生成，费用 5 USDC"

Step 10: 交付结果
  └── TaxReportBot 将报告发给 Alice
       报告附带 x402 receipt（可作为税务凭证）
```

---

## 七、风险边界

### 7.1 风险矩阵

| 风险 | 级别 | 缓解机制 | 残余风险 |
|------|------|---------|---------|
| **收据伪造** | 🔴 高 | Server 链上验证 tx_hash（不能伪造） | — |
| **收据重放** | 🟡 中 | Server 维护 used_receipts 集合 | Server 单点依赖 |
| **预算超支** | 🟡 中 | CAW 日预算硬限制，链上 SpendingLimitModule 强制 | 边缘情况（Gas 波动） |
| **端点伪造** | 🔴 高 | CAW 白名单检查，Signature 验证 | 白名单维护成本 |
| **Session Key 被盗** | 🔴 高 | SpendingLimitModule 限制最大损失（每天最多 50 USDC） | 主机安全依赖 |
| **Warrant 超时未清理** | 🟢 低 | 时间窗口自动过期 + Safe 撤销机制 | — |
| **Gas 价格波动** | 🟢 低 | 预估 + 阈值，超过则暂停 | 可能错过执行时机 |
| **Server 不返回资源** | 🟡 中 | Escrow 托管（先付到 Escrow，满意后释放） | 增加复杂性 |
| **x402 header 解析错误** | 🟡 中 | 严格格式校验 + 降级处理 | — |
| **跨链支付** | 🟡 中 | 目前仅支持 Ethereum USDC | 其他链需单独实现 |

### 7.2 防御层级

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                             防御层级图                                            │
│                                                                                 │
│  Layer 1：协议层（x402）                                                          │
│    └── Server 强制验证收据（链上 tx_hash 可信）                                   │
│                                                                                 │
│  Layer 2：钱包层（Safe）                                                          │
│    └── SpendingLimitModule 链上强制预算限制                                       │
│    └── AllowlistGuard 链上强制白名单校验                                          │
│                                                                                 │
│  Layer 3：Warrant 层（CAW）                                                       │
│    └── Budget / Scope / Time Window 三重检查                                      │
│    └── 自动确认阈值（≤ 3 USDC 自动，> 3 USDC 人工）                              │
│                                                                                 │
│  Layer 4：Agent 层                                                                │
│    └── X402AwareClient 自动处理支付流程                                          │
│    └── 防重放（used_receipts 集合）                                              │
│                                                                                 │
│  Layer 5：监控层                                                                  │
│    └── 80% 预算告警（每日）                                                       │
│    └── 异常频率检测（> 20次/24h 告警）                                            │
│    └── Telegram 实时通知                                                          │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 7.3 关键设计原则

**1. 支付授权的层级分离**

```
用户主私钥（绝对控制）
  └── 签署 CAW Warrant
        └── 授权 Session Key（受限执行）
              └── 每日预算限制（链上强制）
                    └── 白名单限制（链上强制）
```

**2. 收据的可验证性**

```
Receipt = tx_hash（区块链上不可伪造）
    ├── Server 验证：tx 存在 + to=服务端地址 + amount>=要求
    ├── Agent 验证：tx 存在 + from=Agent 钱包 + 收据未被使用
    └── Alice 验证：在 Etherscan 查看交易记录
```

**3. 预算控制的不可绕过性**

```
Safe{SpendingLimitModule} 部署在链上
    └── 即使 Agent 被完全控制，攻击者每天最多转走 50 USDC
    └── 月预算 500 USDC 硬上限，超出后所有操作暂停
```

---

## 八、与 Week 2 Module B 原始设计的关联

### 8.1 本设计是 Module B 的进阶版

| 维度 | Module B（原始） | 本进阶任务 |
|------|----------------|-----------|
| **支付触发** | Escrow 托管（任务完成后释放） | x402 实时支付（请求时即支付） |
| **授权机制** | Escrow 锁定资金 | CAW Warrant 限制操作范围/预算 |
| **控制粒度** | 粗粒度（任务维度） | 细粒度（API 调用维度） |
| **Agent 自主性** | 低（需要人工触发验收） | 高（自动识别 + 执行支付） |
| **实时性** | 低（有托管延迟） | 高（毫秒级支付确认） |

### 8.2 关键差异

```
Module B（Escrow）：
  Alice 锁定资金 → Agent 执行 → 提交交付 → Evaluator 验收 → 释放资金
  时间延迟：可能数小时到数天
  Agent 自主性：低（需要多方交互）

本进阶任务（x402 + CAW）：
  Agent 发起请求 → Server 返回 401+x402 → Agent 自动支付 → 获取资源
  时间延迟：几秒内完成
  Agent 自主性：高（在 CAW 限制内完全自动）
```

### 8.3 组合使用

```
实际系统应该组合使用：
  • x402：处理实时 API 访问的微支付（每次 2-5 USDC）
  • Escrow（ERC-8183）：处理复杂任务的托管（一次性 100+ USDC）
  • CAW：限制 Agent 的整体操作范围和预算
  • ERC-8004：提供 Agent 身份和声誉锚定

x402 和 Escrow 不是互斥的，是互补的：
  • 小额高频支付 → x402（实时）
  • 大额低频任务 → Escrow（托管+验收）
```

---

## 九、架构图（Mermaid）

### 9.1 系统架构 Mermaid

```
graph TB
    subgraph User["用户层"]
        Alice[Alice<br/>主钱包]
    end

    subgraph Agent["Cobo CAW Agent"]
        Intent[Intent Resolver]
        Policy[CAW Policy Engine]
        Pact[Pact Executor]
        Client[X402AwareClient]
    end

    subgraph Safe["Safe{Wallet}"]
        SessionKey[Session Key]
        SpendingLimit[SpendingLimitModule]
        Allowlist[AllowlistGuard]
    end

    subgraph Server["服务提供方"]
        x402API[x402 Server<br/>/api/ai/inference]
        PayEndpoint[x402 Pay Endpoint<br/>/x402/pay]
        AuditLog[Audit Logger]
    end

    subgraph Blockchain["链上"]
        USDC[USDC Contract]
        Escrow[ERC-8183 Escrow]
        Identity[ERC-8004 Identity]
        Events[Events]
    end

    Alice --> Intent
    Intent --> Policy
    Policy --> Pact
    Pact --> Client

    Client -->|1. API Request|x402API
    x402API -->|2. 401 + x402|Client
    Client -->|3. CAW Check|Policy
    Policy -->|4. Approve|Pact
    Pact -->|5. USDC Transfer|USDC
    USDC -->|6. Receipt tx_hash|Pact
    Pact -->|7. Authorization: x402|PayEndpoint
    PayEndpoint -->|8. Receipt|Pact
    Client -->|9. Authorized Request|x402API
    x402API -->|10. Protected Resource|Client

    Safe --> SessionKey
    SessionKey --> SpendingLimit
    SessionKey --> Allowlist
    SpendingLimit -->|Budget Limit|USDC
    Allowlist -->|Scope Check|Client

    AuditLog -->|Events + IPFS|Events
    Pact -->|Audit Record|AuditLog
```

---

*Week 2 · Module B 进阶任务 · x402 Paywall + CAW Agent 自主支付闭环*
*Minami-Bein · 2026-05-28*
*重点：明确授权 + 预算控制 + 可审计记录*