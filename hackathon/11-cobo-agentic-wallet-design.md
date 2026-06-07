# Cobo | Agentic Economy Track
## AI Agent 钱包管控与风险边界设计

> Updated: 2026-06-08
> Project: Cat-Agent（喵星智能体）
> Sponsor Track: Cobo | Agentic Economy x Cobo Agentic Wallet

---

## 一、核心问题：AI Agent 为何需要专属钱包？

传统 EOA（外部拥有账户）钱包由私钥完全控制，任何获取私钥的程序可以做任何操作（transfer、approve、合约调用等）。AI Agent 的威胁模型是：**如果 Agent 被 prompt injection 攻击或私钥泄露，攻击者可以转移全部资产、授权恶意合约、无限approve**。

Cat-Agent 的答案是：**让 Agent 持有受限的钱包，而不是全权钱包**。

```
传统方案：Agent 持有主私钥 → 任何操作无限制 → 风险极高
Cat-Agent：Agent 持有 Session Key → 只能在 Warrant 授权范围内操作 → 风险可控
```

---

## 二、Cat-Agent 钱包架构

### 2.1 钱包层级关系

```
Alice（主人，主私钥持有者）
  └── Safe{Wallet}（主钱包合约，0xCatSafeForAlice）
        ├── 主模块：Safe Core（管理多签/主私钥）
        ├── SpendingLimitModule（日预算 10 USDC，单笔 10 USDC）
        ├── AllowlistGuard（仅允许白名单合约地址）
        │
        └── Session Key（CatAgentSessionKey，由 AI Agent 使用）
              ├── 权限：仅签名（Signing Only）
              ├── 用途：为猫咪消费交易签名
              ├── 有效期：每日自动轮换
              └── 持有上限：10 USDC/笔

每只猫有独立 Warrant：
  └── Warrant: cat-agent-001（Alice 的猫 Mimi）
        ├── issuer: Alice（主人）
        ├── agent: Safe{Wallet}（猫的钱包）
        ├── budget: daily_limit=10 USDC, per_tx_limit=10 USDC
        ├── scope: 仅允许调用 z.ai API 和外卖 API
        └── time_window: 7 天有效，24h 冷却期
```

### 2.2 钱包持有者的角色划分

| 角色 | 持有什么 | 能做什么 | 不能做什么 |
|------|---------|---------|-----------|
| Alice（主人）| 主私钥 | 部署 Safe、安装模块、撤销 Warrant、修改预算 | 日常操作（不直接触碰交易）|
| AI Agent（Cat-Agent）| Session Key | 为授权范围内的交易签名 | 跨出授权范围（链上硬限制）|
| 合约（Safe 模块）| 无私钥 | 强制执行预算限制、拦截未授权交易 | — |

---

## 三、Warrant（授权书）设计

### 3.1 什么是 CAW Warrant

Warrant 是链上可验证的"能力授权书"，定义 Agent 可以做什么、在什么条件下做、做到什么时候。

类比现实世界：
- EOA 私钥 = 持有者拥有无限权力（类似"万能钥匙"）
- CAW Warrant = 持有者只能在授权范围内行动（类似"酒店房卡"，只能在授权楼层和时间开门）

### 3.2 Cat-Agent Warrant 配置

```json
{
  "warrant_id": "cat-agent-001",
  "issuer": "0xAliceMainWallet",
  "agent": "0xCatSafeForMimi",
  "description": "Mimi 的自主消费授权书",
  "budget": {
    "daily_limit": "10000000",
    "per_tx_limit": "10000000",
    "currency": "USDC",
    "reset_interval": "86400"
  },
  "scope": {
    "allowed_endpoints": [
      "https://api.z.ai/v1/multimodal",
      "https://api.delivery.example.com/orders"
    ],
    "allowed_contracts": [
      "0xUSDCContract",
      "0xMerchantContract"
    ],
    "blocked_actions": [
      "approve",
      "permit",
      "transferFrom",
      "increaseAllowance"
    ]
  },
  "time_window": {
    "valid_from": 1757280000,
    "valid_until": 1767926400,
    "cooldown_seconds": 86400,
    "max_transactions_per_day": 5
  },
  "health_rules": {
    "max_consecutive_snack_days": 3,
    "required_main_food_ratio": 0.5,
    "alert_threshold": 0.8
  }
}
```

### 3.3 三重权限校验机制

```
AI Agent 发起支付请求
        │
        ▼
┌─────────────────────────┐
│  Layer 1: Budget Check  │
│  日预算 10 USDC？        │
│  单笔 10 USDC？          │
└────────────┬────────────┘
             │ PASS
             ▼
┌─────────────────────────┐
│  Layer 2: Scope Check   │
│  目标在白名单？          │
│  操作类型被禁止？        │
└────────────┬────────────┘
             │ PASS
             ▼
┌─────────────────────────┐
│  Layer 3: Time Window   │
│  Warrant 有效期内？      │
│  冷却期已过？            │
└────────────┬────────────┘
             │ PASS
             ▼
      Safe{Wallet} 执行
      Session Key 签名广播
```

三层任一失败 → 交易被拦截 → 记录拒绝日志 → 通知主人

---

## 四、AI Agent 执行支付全流程

### 4.1 完整时序

```
1. 猫咪 Mimi 对着摄像头叫
        │
        ▼
2. Cat-Agent 感知层采集：
   - 摄像头截图（猫咪姿态：靠近食盆）
   - 麦克风音频（高频叫声，持续 4 秒）
        │
        ▼
3. Z.AI 多模态推理：
   - 意图：buy_snack（金枪鱼罐头）
   - 置信度：0.94（>0.90 阈值）
   - 目标商品：金枪鱼罐头，2 USDC
        │
        ▼
4. Cat-Agent 业务层 Policy Engine 检查：
   - Warrant 日预算：已用 3 USDC，剩余 7 USDC ✅
   - 单笔限额：2 USDC <= 10 USDC ✅
   - Scope 白名单：api.delivery.example.com 在列表 ✅
   - 冷却期：上次购买是 26 小时前 ✅
   - 健康规则：连续 2 天买零食，第 3 天允许 ✅
        │
        ▼
5. Cat-Agent 构造交易：
   - from: 0xCatSafeForMimi
   - to: 0xMerchantContract
   - value: 2000000（2 USDC，6 位精度）
   - data: USDC transfer method
        │
        ▼
6. Session Key 签名：
   - Cat-Agent 使用手中持有的 Session Key 对交易签名
   - 签名不暴露主私钥
        │
        ▼
7. Safe SpendingLimitModule 再次校验（链上）：
   - 模块独立检查日预算和单笔限额
   - 即使 Cat-Agent 上层检查通过，链上模块也会拦截超限交易
        │
        ▼
8. 交易广播到 Sepolia 网络
        │
        ▼
9. 交易确认（~15 秒），tx_hash 生成
        │
        ▼
10. 审计日志记录：
    - IPFS：完整意图推理记录（含 Z.AI 响应）
    - 链上 Event：tx_hash + amount + timestamp
    - 本地日志：SpendingLimitModule 检查通过
        │
        ▼
11. Telegram 通知 Alice：
    "Mimi 购买了金枪鱼罐头（2 USDC），当前日预算剩余 5 USDC"
```

### 4.2 关键代码路径

```python
# src/wallet/cat_agent_wallet.py

class CatAgentWallet:
    def __init__(self, safe_address: str, session_key: str, warrant_id: str):
        self.safe_address = safe_address
        self.session_key = session_key
        self.warrant_id = warrant_id
        self.policy_engine = PolicyEngine(warrant_id)
        self.safe_wallet = SafeWallet(safe_address)

    async def execute_cat_payment(self, payment_request: PaymentRequest) -> PaymentResult:
        # Step 1: Policy Engine 三重检查
        check_result = await self.policy_engine.check(payment_request)
        if check_result.rejected:
            await self._log_rejection(payment_request, check_result.reason)
            await self._notify_owner(payment_request, check_result.reason)
            return PaymentResult(rejected=True, reason=check_result.reason)

        # Step 2: Session Key 签名
        tx = self._build_transaction(payment_request)
        signed_tx = self._sign_with_session_key(tx)

        # Step 3: 广播交易
        tx_hash = await self.safe_wallet.broadcast(signed_tx)

        # Step 4: 等待确认
        receipt = await self.safe_wallet.wait_for_confirmation(tx_hash)

        # Step 5: 审计日志
        await self._log_payment(payment_request, tx_hash, receipt)

        return PaymentResult(rejected=False, tx_hash=tx_hash)

    def _sign_with_session_key(self, tx: Transaction) -> SignedTransaction:
        # 使用 Session Key（不是主私钥）签名
        return eth_account.sign_transaction(tx, self.session_key)
```

---

## 五、风险边界记录

### 5.1 风险边界矩阵

| 风险类型 | 边界定义 | 触发条件 | 响应机制 |
|---------|---------|---------|---------|
| 预算超支 | 日累计 <= 10 USDC | 当日已消费 >= 10 USDC | SpendingLimitModule 链上拦截，Policy 返回 budget_exceeded |
| 单笔超限 | 单笔 <= 10 USDC | 支付金额 > 10 USDC | SpendingLimitModule 链上拦截，Policy 返回 per_tx_limit_exceeded |
| 合约越权 | 仅白名单合约 | 目标地址不在白名单 | AllowlistGuard 拦截，Policy 返回 scope_violation |
| Warrant 过期 | 7 天有效期 | 当前时间 > valid_until | Policy 返回 warrant_expired，交易不发送 |
| 冷却期未过 | 24h 间隔 | 距上次购买 < 24h | Policy 返回 cooldown_active |
| 健康红线 | 连续 3 天买零食 | 第 4 天仍买非主粮 | Policy 返回 health_redline |
| Session Key 泄露 | 仅签名权限 | Key 被恶意获取 | Safe SpendingLimit 限制日损失 <= 10 USDC（主私钥未被暴露）|
| Prompt Injection | 外部输入不解析为命令 | 恶意指令注入猫咪行为数据 | 输入不进入 command shell，仅进入意图分析 LLM |

### 5.2 链上拦截记录（链上 Event 示例）

```
每笔交易（无论成功还是被拦截）都会在链上留下 Event：

// 交易执行 Event
event CatPaymentExecuted(
    address indexed cat,
    address indexed merchant,
    uint256 amount,
    bytes32 txHash,
    uint256 dailyBudgetRemaining
);

// 交易拦截 Event（关键！用于审计和告警）
event CatPaymentIntercepted(
    address indexed cat,
    string reason,          // "budget_exceeded" | "scope_violation" | "warrant_expired" 等
    uint256 attemptedAmount,
    uint256 currentDailySpent,
    uint256 timestamp
);
```

### 5.3 审计日志结构

```json
{
  "log_id": "audit_20260608_143022",
  "type": "payment_attempt",
  "cat_id": "mimi-001",
  "sequence": {
    "intent_analysis": {
      "timestamp": "2026-06-08T14:30:22Z",
      "audio_features": { "frequency": 842, "duration": 4.2, "pitch": "high" },
      "visual_features": { "pose": "near_bowl", "eye_contact": true, "tail": "up" },
      "z_ai_response": {
        "intent": "buy_snack",
        "confidence_score": 0.94,
        "target_item": "tuna_can",
        "action": "buy"
      },
      "ipfs_cid": "QmXyz...abc"
    },
    "policy_check": {
      "warrant_id": "cat-agent-001",
      "budget_check": { "passed": true, "daily_remaining": "8000000" },
      "scope_check": { "passed": true, "endpoint": "api.delivery.example.com" },
      "time_check": { "passed": true, "cooldown_remaining_seconds": 0 },
      "health_check": { "passed": true, "consecutive_snack_days": 2 }
    },
    "chain_result": {
      "tx_hash": "0xabc123...",
      "block_number": 12345678,
      "status": "confirmed",
      "gas_used": 52000,
      "daily_budget_remaining": "6000000"
    }
  },
  "ipfs_cid": "QmAudit...xyz",
  "on_chain_event_hash": "0xdef456..."
}
```

### 5.4 告警阈值

| 告警级别 | 触发条件 | 通知方式 | 动作 |
|---------|---------|---------|------|
| YELLOW（预算 80%）| 当日消费 >= 8 USDC | Telegram | 提醒主人关注猫咪消费情况 |
| ORANGE（预算 100%）| 当日消费达到 10 USDC | Telegram + SMS | 自动拒绝后续交易，主人确认后恢复 |
| RED（异常频率）| 1 小时内 >= 5 次购买尝试 | Telegram | 检查是否被攻击或误触发，临时冻结 Warrant |
| RED（健康红线）| 连续 3 天只买零食 | Telegram | 自动拒绝第 4 天的零食请求，强制主粮优先 |

---

## 六、与 Cobo CAW 的关系

### 6.1 Cat-Agent 如何使用 CAW

```
Cobo 提供的层级：
  CAW Warrant 框架（链上标准）
        ↑
  Cat-Agent 实现：
  - Warrant 配置（budget/scope/time/health）
  - Policy Engine（业务层三重检查）
  - SpendingLimitModule（链上硬限制）
  - Session Key 管理（Agent 签名密钥）

Cobo CAW 的价值：
  - 链上 Warrant 标准（跨应用可复用）
  - Safe 模块集成（SpendingLimit / AllowlistGuard）
  - Session Key 生命周期管理
  - Warrant 撤销的链上确认
```

### 6.2 Cobo CAW 对 Cat-Agent 的独特价值

1. **宠物专属身份**：每只猫有独立 Warrant（Warrant ID = cat-agent-001），猫与猫之间完全隔离，一只猫超支不影响另一只
2. **无需 KYC**：Warrant 由主人主私钥签名即可，宠物无需身份验证
3. **可继承/可撤销**：主人可以随时撤销 Warrant，撤销操作链上即时生效
4. **与 Safe 无缝集成**：CAW Warrant 天然配合 Safe{Wallet}，不需要额外开发多签逻辑

### 6.3 Cobo CAW 当前限制（Cat-Agent 需要自行补充的部分）

| CAW 提供的能力 | Cat-Agent 需要额外实现的 |
|--------------|------------------------|
| Warrant 基础配置（Budget/Scope/Time）| 健康规则（连续零食天数、主粮比例）|
| SpendingLimitModule 拦截 | RLPF 反馈权重更新 |
| Session Key 签名 | 多宠物场景协调（2 只猫同时购买）|
| Warrant 撤销 | Prompt Injection 防护 |
| — | Telegram 通知和用户交互层 |

---

## 七、总结：Cat-Agent 的安全边界设计原则

```
原则 1：最小权限原则
  Agent 永远不持有主私钥，只持有 Session Key
  Session Key 只能在 Warrant 授权范围内操作

原则 2：纵深防御
  应用层 Policy Engine + 链上 SpendingLimitModule 双重检查
  任一层失败，交易不发送

原则 3：可审计
  所有操作（成功/失败）都有完整日志
  IPFS 存档 + 链上 Event 锚定
  主人可随时查看猫咪消费记录

原则 4：可恢复
  Warrant 可随时撤销
  Session Key 可随时轮换
  主私钥永不触碰日常交易

原则 5：业务层边界
  健康规则（主粮优先）
  冷却期（防止冲动消费）
  告警阈值（主人知情权）
```
