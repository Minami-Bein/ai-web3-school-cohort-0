# Week 2 Module B · Payment / Commerce Flow

> 场景：AI Agent 帮人完成任务并收款
> 基于 ERC-8004（身份/声誉）、ERC-8183（任务托管）、x402（支付）
> 输出到 submissions/week2-payment-flow.md

---

## 场景选择

**场景：链上任务市场（On-chain Task Marketplace）**

Alice（需求方）委托一个 AI Agent（服务提供方）完成一项链上任务——
例如："分析 Wallet ABC 的所有 USDC 跨年流水并生成税务报告"。

Alice 下单 → Agent 接单执行 → 链上托管资金 → 交付结果 → 验收评分 → 结算

---

## 角色拆解

| 角色 | 说明 |
|------|------|
| **Client（Alice）** | 任务发起方，下单、付款、验收 |
| **Provider（Agent）** | 任务执行方，接单、执行、交付 |
| **Evaluator（仲裁者）** | 独立第三方，判断交付是否符合要求（可设定为合约或人） |
| **Facilitator（x402）** | 支付中间层，负责资金划转和异常处理 |
| **Registry（ERC-8004）** | 链上身份和声誉记录，提供 Agent 发现和信任锚定 |

---

## 完整流程（7 步）

```
Step 1        Step 2        Step 3        Step 4        Step 5        Step 6        Step 7
 报价         预算授权       执行         交付          验收          付款/退款     记录证明
  │             │            │            │             │             │             │
  ▼             ▼            ▼            ▼             ▼             ▼             ▼
Alice ─────► Agent       Escrow      Agent ───► Client   Evaluator    Escrow       On-chain
提出需求     接受/还价     合约锁定     提交交付物     验收通过/拒绝   释放或退款    证据上链
```

---

### Step 1：报价（Quotation）

**谁做：** Alice（Client）提出任务需求，Agent 返回报价

**发生什么：**
- Alice 描述任务（"生成 Wallet ABC 的税务报告"）
- Agent 查询 ERC-8004 Identity Registry，确认 Alice 的链上身份
- Agent 根据任务复杂度生成报价（USDC 计价）
- Alice 可以还价，双方通过链下消息协商后达成一致价格 P

**Web3 机制：**
- Agent 的 identity 通过 ERC-8004 Identity Registry 可验证（ERC-721 NFT）
- 报价agreement不需要上链，链上只记录最终确认的价格区间

**记录内容：**
```
任务描述 | 报价价格 P | 交付标准（简述）
```

---

### Step 2：预算授权（Budget Authorization）

**谁做：** Alice（Client）将资金锁定到 Escrow 合约

**发生什么：**
- Alice 授权 Escrow 合约从她的钱包提取预算（B ≥ P）
- 资金进入托管状态，Alice 无法单方撤回
- 同时设定 `expiredAt`（超时时间），超时后自动退款

**Web3 机制：**
- **ERC-8183 Escrow 合约**：`createJob()` → `setBudget()` → `fund()`
- ERC-20（USDC）直接转入合约，无需中间件
- 资金锁定期望：Alice 授权额度 < 实际任务价值，留有争议缓冲

**关键状态：**
```
Job 状态：Open → Funded
Alice: budget locked in Escrow
Agent: 确认接单，开始执行
```

**记录内容：**
```
Escrow 合约地址 | 锁定金额 B | 过期时间 expiredAt
```

---

### Step 3：执行（Execution）

**谁做：** Agent 独立执行任务

**发生什么：**
- Agent 使用链上数据（RPC 调用）和自己的推理能力完成任务
- 所有中间步骤可选择性地记录到链上（作为证据）
- Agent 不得动用 Escrow 中的资金

**AI 侧需要处理：**
- 理解任务范围：读取钱包历史 → 分析交易 → 生成报告
- 上下文管理：判断哪些数据相关、哪些需要聚合
- 执行节奏：任务较大时分段提交（避免 gas 超限）

**记录内容（可选）：**
```
执行开始时间 | 执行的工具调用记录（哈希）| 阶段性状态
```

---

### Step 4：交付（Delivery）

**谁做：** Agent 提交交付物

**发生什么：**
- Agent 调用 `submit(jobId, deliverable)`，附带交付物哈希（IPFS CID 或摘要）
- Escrow 合约状态更新：`Funded → Submitted`
- Alice 和 Evaluator 收到链上事件通知（`WorkSubmitted`）

**Web3 机制：**
- `deliverable` 参数可以是：
  - IPFS CID（完整文件存 IPFS）
  - 链上哈希（摘要，由 Evaluator 自行验证）
  - 任意字符串（取决于任务类型）

**记录内容：**
```
交付物哈希 | 提交时间戳 | Job 状态：Submitted
```

---

### Step 5：验收（Acceptance / Arbitration）

**谁做：** Evaluator 验收，触发结算或争议

**三种可能路径：**

**路径 A：验收通过**
- Evaluator（可以是 Alice 本人）调用 `complete(jobId, reason?)`
- Escrow 合约将资金从 `Completed` 状态释放给 Agent

**路径 B：部分争议**
- Evaluator 调用 `reject(jobId, reason?)`，说明拒绝原因
- Alice 可以与 Agent 协商部分退款（双方链下达成一致）
- 或者提交至更高级别的仲裁（人工事务或链上仲裁协议）

**路径 C：超时未验收**
- `block.timestamp >= expiredAt` 时，任何人可以调用 `claimRefund(jobId)`
- Escrow 自动将资金退还 Alice（`Expired` 状态）

**关键点：Evaluator 的独立性**
- Evaluator 可以是：Alice 本人（`evaluator = client`）、第三方专业仲裁合约、或 DAO 投票合约
- Evaluator 的判断是**唯一的**触发 `complete` 或 `reject` 的通道（Submitted 状态下）
- 这防止了 Alice 和 Agent 在资金锁定后任何一方单方操控

**记录内容：**
```
验收决策 | 原因（可选）| 最终状态 | 结算时间戳
```

---

### Step 6：付款 / 退款 / 争议（settlement）

**谁做：** Escrow 合约自动执行（或 Evaluator 触发）

**发生什么：**

| 状态 | 资金去向 | 调用者 |
|------|----------|--------|
| `Completed` | B → Provider（Agent） | Escrow 自动 |
| `Rejected` | B → Client（Alice） | Evaluator 触发 |
| `Expired` | B → Client（Alice） | 任何人触发 |

**可选手续费：**
- 如果有平台方，Escrow 合约支持扣留 `platformFee` 后释放给 Agent
- `complete(jobId, reason?)` 中附带 reason 可被 Reputation Registry 引用

**记录内容：**
```
实际支付金额 |手续费| 最终状态 | 链上交易哈希
```

---

### Step 7：记录证明（Attestation & Records）

**谁做：** 各方将结果写入链上记录

**发生什么：**

**ERC-8004 Reputation Registry 写入：**
- Alice 对 Agent 留下评分 + 评价（feedback signal）
- Evaluator 将验收结果哈希写入 Validation Registry
- Agent 的声誉分数更新（off-chain 算法 + on-chain 锚定）

**x402 支付凭证：**
- 每次资金释放会生成标准的 x402 收据（payment receipt）
- 收据包含：payer、payee、amount、jobId、escrowContract、timestamp
- 收据作为 Alice 的税务凭证（可导出）

**链上证据永久留存：**
- Escrow 合约的交易历史（链上）
- IPFS 上的原始交付物
- Evaluator 的决策理由（可选，存储在事件中）

**记录内容：**
```
Job 完成证明 | 评分/评价 | x402 收据 | 声誉分数更新
```

---

## Protocol 对比：x402 vs ERC-8004 vs ERC-8183

| 维度 | x402 | ERC-8004 | ERC-8183 |
|------|------|----------|----------|
| **核心问题** | 支付：如何让机器/服务收到钱 | 身份：如何找到可信 Agent | 结算：任务完成后钱怎么分 |
| **解决哪一段** | **支付层**：服务付费、micropayment、跨网络原生支付 | **发现与信任层**：Agent 身份注册、声誉信号、验证钩子 | **结算层**：托管锁定、验收触发、资金释放 |
| **是否覆盖支付** | ✅ 原生支付协议（HTTP-native） | ❌ "Payments are orthogonal to this protocol" | ❌ 只处理 Escrow，不处理支付接入 |
| **验证机制** | 支付凭证（receipt），可导出 | Validation Registry（zkML/TEE/staker 重跑） | Evaluator 单一授权触发 |
| **身份机制** | 无（基于钱包地址） | ERC-721 Identity Registry（链上可发现） | 无（基于 Job 的 client/provider/evaluator 地址） |
| **仲裁机制** | 无 | Validation Registry（外部验证钩子） | Evaluator 唯一判断权 |
| **状态机** | 无 | 无 | ✅ Open → Funded → Submitted → Completed/Rejected/Expired |
| **与本场景关系** | 付款触发层（本场景 Step 6 之后） | Agent 发现 + 声誉锚定（本场景 Step 1 之前） | 托管 + 结算核心（本场景 Step 2/5/6） |

### 组合建议

```
Alice 发现 Agent
    │
    ▼
ERC-8004 Identity Registry ──► 确认 Agent 身份 + 声誉
    │
    ▼
双方议价（链下）
    │
    ▼
Alice 锁定资金到 Escrow
（ERC-8183）
    │
    ▼
Agent 执行任务
    │
    ▼
Agent 提交交付（IPFS）
    │
    ▼
Evaluator 验收
    │
    ├─► 合格 ──► Escrow 释放资金 ──► x402 生成收据
    │
    └─► 不合格 ──► 争议/退款 ──► ERC-8004 Reputation 更新
```

**x402** 不替代 Escrow，它是 Escrow 释放后的**支付凭证层**——把钱从合约转到 Agent 钱包后，x402 记录这笔支付的完整轨迹，供税务和审计使用。

**ERC-8004** 不解决支付，它解决的是"我怎么知道这个 Agent 是否可信"——在报价之前就已经介入（发现 + 声誉锚定）。

**ERC-8183** 是本场景的结算核心，但它需要 x402 来处理跨链/跨网络的实际资金转移，需要 ERC-8004 来提供 Agent 可信度参考。

---

## 最小可运行 Flow（文字版）

```
1. Client 在链下告诉 Agent 任务需求
2. Agent 查询 ERC-8004 确认 Client 身份和历史评分
3. 双方议价确认后，Client 调用 Escrow.fund(jobId) 锁定 USDC
4. Agent 执行任务，完成后调用 Escrow.submit(jobId, ipfsCid)
5. Evaluator（= Client）调用 Escrow.complete(jobId) 验收通过
6. Escrow 自动将 USDC 转给 Agent
7. x402 生成 payment receipt，双方声誉更新到 ERC-8004
```

---

*Week 2 · Module B · Payment Commerce Flow · Minami-Bein · 2026-05-28*