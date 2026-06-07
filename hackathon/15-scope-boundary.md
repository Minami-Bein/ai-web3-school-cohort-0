# 项目 Scope 边界：暂时不做什么

> Updated: 2026-06-08
> Project: Cat-Agent

---

## Scope 决策原则

**"能跑 > 能完美跑"**

Week 4 是黑客松提交周，不是产品发布。在有限时间内，把 1 个核心流程跑通比把 10 个功能做一半更有价值。

以下功能被明确砍掉、延后或改为 Mock，以确保 Week 4 交付质量。

---

## 砍掉的功能（Cut）

### 1. 多宠物并发管理（Multi-Pet Coordination）

**原计划：**
支持 2 只及以上的猫咪同时发起购买请求，系统排队处理，避免日预算冲突。

**砍掉原因：**
- 实现复杂度高：需要引入分布式锁或消息队列协调
- 测试难度高：需要 2 只猫同时配合（猫不听指挥）
- 核心价值低：单宠物场景已完整展示 Warrant 机制

**替代方案：**
保持单宠物（Mimi）场景。所有文档和代码注释标注"支持多宠物扩展"，但不实现。

---

### 2. RLPF 反馈权重自动更新（RLPF Weight Auto-Update）

**原计划：**
猫咪吃完某商品后，通过摄像头判断"是否吃光"，自动更新该商品的 RLPF 权重。

**砍掉原因：**
- 需要二次 AI 推理（判断猫咪是否吃光）
- 反馈回路的准确性难以验证
- 不影响核心支付流程展示

**替代方案：**
RLPF 权重初始值固定为预分配（如 tuna=0.6, salmon=0.4）。Feedback 留空，后续迭代。

---

### 3. 外卖平台真实 API 对接（Real Delivery Platform Integration）

**原计划：**
真实调用外卖平台 API，触发商品配送，获取实时配送状态。

**砍掉原因：**
- 外卖平台 API 需要商户入驻和资质审核
- 真实配送涉及真实金钱（哪怕是测试环境也有合规风险）
- 不影响链上支付流程的核心演示

**替代方案：**
Mock API 返回固定 JSON：`{ "order_id": "mock-order-001", "delivery_time": "30min", "status": "dispatched" }`。

---

## 延后的功能（Deferred）

### 4. IPFS 审计日志存档（IPFS Audit Log Archiving）

**原计划：**
每次交易后，将 intent_log + tx_receipt 完整存档到 IPFS，返回可验证的 CID。

**延后原因：**
- IPFS 依赖 Pinata/Infura 账号配置，增加环境变量
- IPFS CID 不影响黑客松评分（评委看代码和演示截图）
- 可以在 Week 4 后单独迭代

**替代方案：**
审计日志仅存本地 JSON 文件（`logs/audit_20260608.json`）。代码注释标注"可扩展为 IPFS"。

---

### 5. ERC-712 签名报告（ERC-712 Signed Report）

**原计划：**
月度报告用 ERC-712 签名，主人可以验证报告内容未被篡改。

**延后原因：**
- 需要额外实现 signer/verifier 逻辑
- 月度报告本身是 Week 4 后才有的功能（当前无数据）
- 不影响核心支付流程

**替代方案：**
月度报告为普通 JSON，注释标注"可升级为 ERC-712 签名"。

---

### 6. Telegram 实时通知优化（Telegram Rich Notifications）

**原计划：**
Telegram 推送使用 Inline Keyboard（确认/取消按钮）、配送状态卡片、带图片的消息。

**延后原因：**
- Telegram Bot Inline Keyboard 需要额外的 Callback 处理逻辑
- 普通文本消息已足够传达"猫咪买了什么、花了多少钱"
- 简化通知逻辑降低集成复杂度

**替代方案：**
Telegram 发送纯文本消息：`"Mimi 购买了金枪鱼罐头（2 USDC），日预算剩余 8 USDC。"`

---

## 改为 Mock 的功能（Mock）

### 7. Z.AI 多模态推理（Mock Mode）

**正常目标：**
Z.AI 真实 API 调用。

**改为 Mock 条件：**
如果 6/22 前 Z.AI API Key 未到达。

**Mock 实现：**
```python
# src/perception/mock_intent.py
INTENT_POOL = [
    {"intent": "buy_snack", "confidence_score": 0.94, "target_item": "tuna_can"},
    {"intent": "buy_snack", "confidence_score": 0.89, "target_item": "salmon_treat"},
    {"intent": "go_out",    "confidence_score": 0.76, "target_item": None},
]

def mock_multimodal_intent(camera_frame: bytes, audio_data: bytes) -> dict:
    return random.choice(INTENT_POOL)
```

---

### 8. Cobo CAW Warrant 链上验证（Local JSON Mock）

**正常目标：**
从 Cobo CAW SDK 读取链上 Warrant 配置。

**改为 Mock 条件：**
如果 6/22 前 CAW Access 未授权或 SDK 不可用。

**Mock 实现：**
```json
// configs/warrants/mimi-001.json
{
  "warrant_id": "cat-agent-001",
  "issuer": "0xAliceMainWallet",
  "agent": "0xCatSafeForMimi",
  "budget": { "daily_limit": "10000000", "per_tx_limit": "10000000" },
  "scope": { "allowed_endpoints": ["https://api.z.ai/v1/multimodal"] },
  "time_window": { "valid_until": 1767926400, "cooldown_seconds": 86400 }
}
```

---

### 9. 商品库存查询（Mock Delivery API）

**正常目标：**
调用外卖平台真实 API 查询商品库存和配送时间。

**改为 Mock 条件：**
始终使用 Mock（不依赖真实外卖平台 API）。

**Mock 实现：**
```python
# src/integration/mock_delivery.py
PRODUCT_CATALOG = {
    "tuna_can":    {"price_usdc": 2_000_000, "delivery_min": 25, "delivery_max": 35},
    "salmon_treat":{"price_usdc": 3_000_000, "delivery_min": 20, "delivery_max": 30},
    "chicken_snack":{"price_usdc": 1_500_000, "delivery_min": 30, "delivery_max": 45},
}

def mock_query_product(item_id: str) -> dict:
    if item_id not in PRODUCT_CATALOG:
        return {"in_stock": False}
    return {**PRODUCT_CATALOG[item_id], "in_stock": True}
```

---

## Scope 边界总结

| 功能 | 状态 | 理由 |
|------|------|------|
| 多宠物并发管理 | 砍掉 | Week 4 核心是单宠物支付全流程 |
| RLPF 权重自动更新 | 砍掉 | 反馈回路验证成本高，不影响展示 |
| 真实外卖 API | 砍掉 | 商户入驻耗时，不影响链上流程 |
| IPFS 存档 | 延后 | 本地 JSON 已满足审计需求 |
| ERC-712 报告签名 | 延后 | 报告功能本身是 Week 4 后才有 |
| Telegram 富文本通知 | 延后 | 纯文本已足够传达核心信息 |
| Z.AI 多模态推理 | Mock | 依赖 API Key 到达时间 |
| CAW Warrant 链上验证 | Mock | 依赖 Cobo Access 审批 |
| 商品库存查询 | Mock | 不依赖外部 API |

---

## Week 4 实际交付范围

```
必须交付（Yes）：
  - Safe{Wallet} 部署 + USDC 转账
  - Z.AI 或 OpenAI 意图推理（哪怕 Mock）
  - Policy Engine 三重检查（Benchmark 演示）
  - SpendingLimitModule 链上拦截截图
  - 完整 ReAct 工作流（代码可查）
  - Telegram 文本通知
  - README 架构说明

不交付（No）：
  - 多宠物支持
  - RLPF 自动更新
  - 真实外卖配送
  - IPFS 存档
  - ERC-712 签名
  - 富文本 Telegram 消息
```

**核心原则：把 1 个猫咪购买工作流做透，不做 10 个半成品功能。**