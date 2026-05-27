# Week 2 Module D · Agent 链上动作执行流程 + Agent Wallet 权限策略

> 基于 Week 2 Module D：Agent Wallet / Permission / Safe Execution
> 结合 ERC-4337、Safe{Wallet}、guard / policy 机制
> 输出到 submissions/week2-agent-wallet-policy.md

---

## Part 1：Agent 发起链上动作的执行流程图

### 执行流程（8 步）

```
┌─────────────────────────────────────────────────────────────────────────────┐
│               Agent 发起链上动作 — 执行流程图                                  │
│                                                                             │
│  Step 1       Step 2       Step 3       Step 4       Step 5       Step 6   │
│  意图理解     上下文构造    策略校验     交易模拟     人工确认     签名广播  │
│   🤖           🤖           🤖           🤖           👤           🤖→🔗    │
│  自动化        自动化        自动化        自动化        人工        自动化  │
└─────────────────────────────────────────────────────────────────────────────┘
                                          │
                                          ▼
                            Step 7（链上执行）→ Step 8（结果验证）
                              🔗 网络执行      🤖 Agent 验证
                              自动执行         自动化
```

### 每一步详解

| Step | 执行方 | 自动化？ | 说明 |
|------|--------|---------|------|
| **1. 意图理解** | Agent（LLM） | ✅ | 解析用户自然语言请求，判断是否需要链上操作 |
| **2. 上下文构造** | Agent | ✅ | 收集链上状态（余额、Gas price、合约数据），构造交易请求 |
| **3. 策略校验** | Agent + Smart Wallet | ✅ | 用预算/权限策略校验交易是否在允许范围内（ERC-4337 validation） |
| **4. 交易模拟** | Agent + RPC | ✅ | `eth_call` 模拟交易，预估结果（不产生链上变更） |
| **5. 人工确认** | 用户（👤） | ❌ | **必须由人类确认**：展示交易摘要，用户主动点击确认 |
| **6. 签名广播** | User Wallet → Bundler | ✅ | 用户签名后，Transaction / UserOperation 广播到网络 |
| **7. 链上执行** | EVM 网络 | 自动 | 交易被打包进区块，状态变更，不可逆 |
| **8. 结果验证** | Agent | ✅ | 读取 tx hash 验证状态，更新本地记录，通知用户 |

### 关键原则

**哪些可以自动化：**
- ✅ 只读操作（`eth_call`、读取余额/历史）
- ✅ 交易模拟和 Gas 估算
- ✅ 上下文收集和意图解析
- ✅ 链上状态监控和结果验证

**哪些必须人工确认：**
- ❌ 任何写入操作（`eth_sendTransaction`）
- ❌ 签署授权（`approve`、`permit`）
- ❌ 调用合约（即使是安全的合约）
- ❌ 部署合约

### Guard / Policy 在流程中的位置

```
意图 → 上下文 → [策略校验 Step 3] → 模拟 → [人工确认 Step 5] → 签名 → 执行
              ↑
         Guard 检查：
         • 金额是否超预算？
         • 合约是否在白名单？
         • 是否触发时间锁？
         • 是否为高风险操作？
```

---

## Part 2：Agent Wallet 权限策略设计

### 场景设定

Alice（用户）授权 TaxReportBot（Agent）代表她执行某些链上操作，但设置了严格的权限边界。

### 策略组件

```
┌─────────────────────────────────────────────────────────────────┐
│              TaxReportBot Agent Wallet 权限策略                   │
│                                                                 │
│  ┌─────────┐  ┌──────────────┐  ┌─────────────┐  ┌─────────┐  │
│  │ 预算    │  │ 可调用合约   │  │ 执行动作    │  │ 人工确认│  │
│  │Budget   │  │ Allowlist   │  │ Actions    │  │阈值     │  │
│  └─────────┘  └──────────────┘  └─────────────┘  └─────────┘  │
│                                                                 │
│  ┌─────────┐  ┌──────────────┐  ┌─────────────────────────┐   │
│  │ 撤销   │  │  日志记录    │  │   失败处理               │   │
│  │Revoke  │  │  Logging    │  │   Failure Handling      │   │
│  └─────────┘  └──────────────┘  └─────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
```

---

### 1. 预算（Budget）

| 参数 | 值 | 说明 |
|------|---|------|
| **日预算** | 50 USDC | 24小时内 Agent 可转出的最大金额 |
| **单笔上限** | 10 USDC | 单次交易不得超过 10 USDC |
| **月预算** | 500 USDC | 30天内 Agent 可转出的最大金额（硬上限） |
| **Gas 预算** | 0.1 ETH/月 | Agent 执行链上操作的总 Gas 上限 |

**实现机制：**
- Safe{Wallet} 的 ` SpendingLimitModule`：在模块级别设置每日/单笔限额
- 或 ERC-4337 Paymaster：为 UserOperation 提供 Gas 代付，超额则拒绝

**超预算时的行为：**
```
金额 10 USDC → ✅ 通过
金额 15 USDC → ❌ 拒绝（单笔超限）
今日已转 45 USDC，再转 10 USDC → ❌ 拒绝（日预算耗尽）
```

---

### 2. 可调用合约白名单（Allowed Contracts）

| 类别 | 合约 | 原因 |
|------|------|------|
| **ERC-20 读取** | ERC-20 代币合约（USDC、DAI） | 只读 balanceOf |
| **RPC** | Alchemy / Infura | 链上数据读取 |
| **IPFS** | ipfs.io API | 报告文件上传 |
| **Indexing** | The Graph（Subgraph） | 交易历史查询 |
| **❌ 禁止** | 任何未经白名单的合约 | 默认拒绝 |

**实现机制：**
- Safe{Wallet} 的 `AllowanceModule` + 地址校验
- ERC-4337 `ValidationModule`：验证 `UserOperation.target` 是否在白名单

**动态更新：**
- 用户可以添加新合约到白名单（需要主钱包签名确认）
- 用户可以随时移除合约（立即生效）

---

### 3. 可执行动作（Actions）

| Action | 权限 | 说明 |
|--------|------|------|
| `balanceOf` | ✅ 自动 | 读取任意钱包余额 |
| `transfer`（< 10 USDC） | ✅ 自动 | 日预算内自动转账 |
| `transfer`（≥ 10 USDC） | ⚠️ 人工确认 | 超过单笔上限 |
| `approve` | ❌ 禁止 | 永远不允许 Agent 代表用户授权 |
| `swap`（Uniswap V3） | ⚠️ 人工确认 | 任何 Swap 操作 |
| `permit` | ❌ 禁止 | 永远不允许签名授权 |
| `deploy` | ❌ 禁止 | 永远不允许部署合约 |

**核心原则：**
- 最小权限：Agent 只能执行白名单内的动作
- 授权类操作（approve/permit）：**永久禁止 Agent 执行**，即使金额很小

---

### 4. 人工确认阈值（Human-in-the-Loop Thresholds）

| 条件 | 行为 |
|------|------|
| 转账金额 ≥ 10 USDC | 暂停 → 展示摘要 → 等待用户确认 |
| 转账金额 ≥ 500 USDC | 暂停 → 通知备用联系人 → 额外延迟确认 |
| 目标合约不在白名单 | 立即拒绝 + 日志记录 |
| 24小时内操作次数 ≥ 20 次 | 暂停 + 告警（可能是攻击行为） |
| 触发时间锁（> 7天未操作） | 自动解除时间锁冷却期 |

**人工确认的 UI 提示（示例）：**
```
⚠️ Agent 操作需要确认

交易摘要：
  从：0x742d...e7595f
  到：Uniswap V3 Router
  金额：15 USDC
  操作：USDC → ETH Swap

风险提示：
  ⚠️ 超过单笔自动执行限额（10 USDC）
  ⚠️ 此操作涉及代币交换，不可逆

请在钱包内确认上述信息。
```

---

### 5. 撤销方式（Revocation）

| 撤销方式 | 操作方法 | 生效时间 |
|---------|---------|---------|
| **即时撤销** | 用户在 Safe{Wallet} UI 中禁用 Agent Module | 立即生效 |
| **时间锁撤销** | 用户发起撤销请求，等待 48h 冷却期 | 48h 后自动执行 |
| **紧急撤销** | 触发 Panic Button（预设的硬件钱包按钮或独立私钥） | 立即生效 |
| **自动撤销** | 月预算耗尽或日预算连续 3 天达到上限 | 自动触发通知 |
| **Session Key 过期** | Session Key 设置有效期限（如 7 天） | 到期自动失效 |

**撤销后行为：**
- Agent 尝试执行任何操作 → 链上验证失败（Module 被禁用）
- Agent 的 Session Key 签名不再被 EntryPoint 接受
- 用户收到通知（Email / Telegram）

---

### 6. 日志记录（Logging）

| 事件类型 | 记录内容 | 存储位置 |
|---------|---------|---------|
| **每次操作** | timestamp, action, amount, target, result | 链上（Event）+ 本地数据库 |
| **权限变更** | 预算调整、白名单更新、时间锁变更 | 链上（Event） |
| **人工确认** | 用户确认的操作摘要、确认时间 | 链上（Event） |
| **异常事件** | FP-1 到 FP-7 触发记录 | 链上 + 告警系统 |
| **撤销事件** | 撤销原因、撤销时间、生效时间 | 链上（Event） |

**链上 Event 示例（Solidity）：**
```solidity
event AgentOperation(
    address indexed agent,
    address indexed target,
    uint256 amount,
    bytes32 actionHash,
    uint256 timestamp,
    bool success
);

event AgentRevoked(
    address indexed agent,
    uint256 timestamp,
    string reason
);
```

**日志可查性：**
- 任何时候可以通过 `eth_getLogs` 查询 Agent 的所有操作历史
- 日志作为仲裁证据（ERC-8183 Evaluator 可以引用链上日志判断争议）

---

### 7. 失败处理（Failure Handling）

| 失败场景 | 检测方式 | 处理方式 |
|---------|---------|---------|
| **RPC 返回假数据** | 多数据源交叉验证 | 切换备用 RPC，拒绝单一来源数据 |
| **超预算** | 本地预算计数器 | 拒绝操作 + 记录 FP-1 日志 + 通知用户 |
| **合约不在白名单** | 白名单校验（链上 + 本地） | 立即拒绝 + 记录异常日志 |
| **Gas 估算失败** | `eth_estimateGas` 返回 error | 拒绝操作 + 提示用户检查网络状态 |
| **EntryPoint 验证失败** | ERC-4337 ValidationPhase 失败 | 拒绝签名 + 展示错误原因给用户 |
| **时间锁未满足** | 区块时间戳 vs 时间锁设置 | 拒绝操作 + 显示等待时间 |
| **Session Key 被盗用** | 异常频率检测（> 20次/24h） | 暂停 Agent + 告警 + 建议撤销 |

---

## Part 3：ERC-4337、Safe、Guard/Policy 机制为什么重要

### ERC-4337：Account Abstraction 的实现层

**解决什么问题：**
EOA（外部拥有账户）的验证逻辑是固定的（ECDSA 签名），无法表达复杂权限。
ERC-4337 允许 Smart Contract Account 自定义验证逻辑，同时通过 EntryPoint 合约统一处理所有 UserOperation。

**为什么对 Agent Wallet 重要：**

| 传统 EOA | ERC-4337 Smart Account |
|---------|------------------------|
| 验证逻辑固定（私钥签名） | **验证逻辑可编程**（自定义规则） |
| 无法设置预算限制 | **可以内嵌 SpendingLimit** |
| 无法设置时间锁 | **可以内嵌 TimeLock** |
| Agent 需要持有私钥 | **Agent 只有 Session Key（临时授权签名）** |
| 无法暂停/撤销 | **可以通过 Module 禁用立即撤销** |

**核心价值：**
ERC-4337 把"谁可以签"和"签的是什么"解耦——Agent 拿到的 Session Key 只能执行特定受限操作，不是一条完整私钥。

---

### Safe{Wallet}：生产级智能钱包

**解决什么问题：**
Safe 是 ERC-4337 兼容的生产级智能钱包实现，提供多签、模块化架构、Guard 机制。

**为什么对 Agent Wallet 重要：**

Safe 的模块化架构让权限控制变成可插拔的组件：
- **Module（模块）**：扩展钱包功能（预算限制、社交恢复、时间锁）
- **Guard（守卫）**：在交易执行前/后进行检查（白名单、预算校验）
- **Fallback Handler**：处理特殊调用（如 ERC-4337 UserOperation）

Safe 解决了：
1. **多签恢复**：Agent 被攻击后，用户可以通过多签恢复钱包控制权
2. **模块化权限**：不需要写合约，只需要配置 Module
3. **生产安全**：经过大量资产验证，代码经过审计

---

### Guard / Policy 机制：权限控制的执行层

**Guard 是什么：**
Guard 是部署在 Safe 上的检查合约，在 `preCheck` 和 `postCheck` 阶段拦截交易。

**Policy 是什么：**
Policy 是 Guard 的规则定义——"什么条件下允许通过"。

**实际执行流程：**
```
UserOperation 发送到 EntryPoint
        ↓
EntryPoint 调用 Safe 的 validateUserOp()
        ↓
Safe 调用 SpendingLimitModule（检查预算）
        ↓
Safe 调用 AllowlistGuard（检查目标合约是否在白名单）
        ↓
如果任一检查失败 → 拒绝 UserOperation
        ↓
验证通过 → 执行 UserOperation.callData
        ↓
Guard.postCheck → 检查执行结果是否异常
```

**三者的关系：**

```
ERC-4337    = 协议层：定义 UserOperation 如何被验证和执行
Safe        = 实现层：生产级 Smart Account，支持模块化
Guard/Policy = 规则层：具体的权限规则（预算、白名单、时间锁）
```

**为什么它们组合在一起很重要：**

| 风险 | 没有这些机制时 | 有这些机制时 |
|------|-------------|------------|
| Agent 被盗后转走全部资产 | 攻击者用私钥直接转走 | Guard 检查金额是否超预算，阻止超限转账 |
| Agent 调用恶意合约 | 合约执行任意操作 | 白名单 Policy 拒绝非白名单合约 |
| 攻击者修改 Agent 权限 | 无法察觉 | Guard 记录所有权限变更事件 |
| 用户失去钱包控制权 | 无法恢复 | Safe 多签 + 社交恢复 Module |
| Agent 执行了未授权操作 | 无法撤销 | 时间锁延迟执行，用户有时间介入 |

---

## 三者组合：最小可行的 Agent Wallet 架构

```
用户（Alice）
   │
   ├── Smart Wallet（Safe）
   │      ├── SpendingLimitModule（预算限制）
   │      ├── AllowlistGuard（白名单策略）
   │      ├── TimeLockModule（时间锁）
   │      └── ERC-4337 EntryPoint（兼容）
   │
   └── Agent（TaxReportBot）
          ├── Session Key（临时授权，仅可执行白名单内操作）
          └── MCP/RPC（读取链上数据）
```

**安全保证：**
- Agent 没有完整私钥，只有 Session Key
- Session Key 只能执行白名单内的受限操作
- 所有操作都有预算上限和时间锁
- 用户可以随时撤销，撤销后 Session Key 立即失效

---

*Week 2 · Module D · Agent Wallet Policy · Minami-Bein · 2026-05-28*