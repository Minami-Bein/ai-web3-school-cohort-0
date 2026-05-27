# Week 2 · 方向深挖包 + 项目初步 Proposal

> AI × Web3 School Cohort 0 · Week 2 总交付
> 主方向：Wallet / Permission / Safe Execution
> 输出到 submissions/week2-final-deliverable.md

---

## 一、AI × Web3 问题地图（6 方向）

### 问题地图总览

```
                     ┌───────────────────────────────────────┐
                     │       AI × Web3 问题地图 · Week 2        │
                     └──────────────────┬──────────────────────┘
                                        │
      ┌──────────────┬──────────────┬─────┴────┬──────────────┬──────────────┐
      ▼              ▼              ▼          ▼              ▼              ▼
  ┌────────┐    ┌────────┐   ┌────────┐ ┌────────┐    ┌────────┐    ┌────────┐
  │支付结算│    │身份声誉│   │钱包权限│ │隐私安全│    │开发工具│    │治理协作│
  │Commerce│    │Identity│   │Wallet  │ │Privacy │    │DevTool │    │Govern │ │
  │Settle  │    │Reput  │   │Safe    │ │Sovereign│   │Agent  │    │Public │ │
  └────────┘    └────────┘   └────────┘ └────────┘    └────────┘    └────────┘
       ①            ②          ③★        ④             ⑤            ⑥
```

### 方向 1：支付 / Commerce / Settlement
**机器对机器的原生支付基础设施**

| AI 作用 | Web3 机制 |
|--------|---------|
| Agent 之间自动协商服务价格 | USDC / ERC-20 原生支付，链上即时结算 |
| 根据任务复杂度动态定价 | Escrow 托管合约，双向锁定资金 |
| 监控服务交付质量，决定是否释放资金 | Gasless Transaction（ERC-4337 Paymaster） |

**核心问题：** AI 服务如何被定价？交付如何被确认？跨 Agent 支付如何原子化？

---

### 方向 2：身份 / Reputation / Capability / Interoperability
**给 AI Agent 建立可验证的数字身份**

| AI 作用 | Web3 机制 |
|--------|---------|
| Agent 通过身份锚定在链上执行操作 | ENS + 链上身份（`agent.alice.eth`） |
| 声誉分数驱动服务定价和任务分配 | POAP / SBT 不可转让声誉徽章 |
| 利用历史行为记录推断对方能力 | EAS、Galxe 等去中心化声誉协议 |
| 跨链行为聚合为统一画像 | LayerZero、Axiom 跨链消息协议 |

**核心问题：** Agent 身份谁来认证？声誉数据怎么防止刷分？

---

### ★ 方向 3：Wallet / Permission / Safe Execution（本周主方向）
**AI 执行链上操作的权限边界**

| AI 作用 | Web3 机制 |
|--------|---------|
| 根据用户授权范围执行链上操作 | ERC-4337 Smart Account（自定义验证逻辑） |
| 理解权限等级（只读/转账/approve） | Safe{Wallet} 模块化架构（Module + Guard） |
| 在权限边界内自主决策，超出边界请求确认 | 会话密钥 Session Key（临时授权） |
| 动态判断是否需要人工确认 | 多签 + 时间锁 + 额度限制（链上强制） |

**核心问题：** AI 在什么粒度上可以被授权？权限被突破后如何止损？如何随时撤销？

---

### 方向 4：Privacy / Security / Sovereignty
**AI 与链上数据交互时的隐私保护**

| AI 作用 | Web3 机制 |
|--------|---------|
| 本地完成链上数据的脱敏处理 | 零知识证明 ZKP（ZK-Prover 验证计算结果） |
| 对恶意合约和钓鱼攻击主动检测 | 隐私池 Privacy Pools（资金混合解耦） |
| 用户自主控制身份数据，AI 按授权访问 | TEE 可信执行环境（Intel SGX） |

**核心问题：** 链上数据全公开——AI 如何在保护隐私的前提下使用这些数据？

---

### 方向 5：Dev Tooling / Agent Workflow
**AI 原生的 Web3 开发与部署流水线**

| AI 作用 | Web3 机制 |
|--------|---------|
| 自然语言 → Solidity 自动生成合约代码 | 自动化部署管道（GitHub Actions + Foundry） |
| 自动写测试用例，覆盖 Happy Path 和边界条件 | 形式化验证 Certora Prover |
| 根据链上 gas 历史数据优化合约代码 | 链上部署记录（ABI + 源代码永久上链） |

**核心问题：** AI 生成的合约谁来负责？生成式代码的安全标准怎么建立？

---

### 方向 6：Governance / Coordination / Public Goods
**去中心化协作与集体决策**

| AI 作用 | Web3 机制 |
|--------|---------|
| 作为提案发起者或投票执行者参与 DAO 治理 | Governor 合约（Tally、OpenZeppelin Governor） |
| 汇总链上治理数据（提案/投票/讨论）给用户做参考 | 代币加权投票（Token-holder 投票决定方向） |
| 根据投票结果自动执行协议参数变更 | 乐观认证 Optimistic Approval（延迟执行+争议期） |

**核心问题：** AI 可以代理投票吗？AI 生成的提案质量怎么评估？

---

## 二、方向选择说明

### 选择方向 3「Wallet / Permission / Safe Execution」作为主线

### 为什么它不是纯 AI 问题？

**问题根源在信任的制度设计，不是模型能力**

纯 AI 问题会用"更好的模型"解决——更准确的意图识别、更长的上下文窗口、更强的推理能力。但这不能解决：

1. **链上执行的不可逆性**：AI 可以拒绝恶意指令，但一旦签名广播到链上就无法撤回。传统 AI 安全不涉及"广播后无法回滚"的场景。

2. **权限边界需要密码学强制**：AI 模型层面的指令边界可以被对抗性输入绕过；但 Safe Module 的 SpendingLimitModule 是链上合约执行的，不受模型输入影响。

3. **多方协作的责任归属**：AI 安全工具无法解决"当资产损失时谁负责"的治理问题。ERC-8183 Escrow + Evaluator 机制是纯 AI 工具链中没有的制度设计。

### 为什么它不是纯 Web3 问题？

**权限粒度和动态决策超出链上逻辑的表达能力**

纯 Web3 问题会用"更严的合约"解决——多签、时间锁、额度限制。但这些机制是静态规则：

- 用户预先写入合约"每天最多转 100 USDC"
- AI 今天遇到紧急情况需要临时转 200 USDC——合约不会变通

而 AI 的价值在于**动态判断**：根据上下文（当前 gas 费、交易历史、用户偏好）决定是否应该申请更大权限。这不是链上逻辑能表达的——它需要 LLM 的推理能力 + 链上验证的组合。

---

## 三、问题拆解（主方向深挖）

### 场景：Alice（用户）授权 TaxReportBot（Agent）代表她执行链上税务报告操作

#### 参与方

| 参与方 | 角色 | 信任假设 |
|--------|------|---------|
| Alice（用户） | 钱包所有者，Agent 的授权人 | 完全信任自己的判断，但信任 Agent 有限 |
| TaxReportBot（Agent） | 代表 Alice 执行操作的 AI | 只信任执行只读操作，授权操作必须请求确认 |
| Safe{Wallet} | 智能钱包，权限策略的执行器 | 密码学强制执行，不信任任何单一实体 |
| ERC-4337 EntryPoint | UserOperation 的验证层 | 验证 Session Key 签名，但不关心操作内容 |
| ERC-8004 Identity Registry | TaxReportBot 的链上身份 | 代表 Agent 的可验证身份 |
| ERC-8183 Escrow | 任务支付托管 | 锁定资金，根据交付结果释放 |

#### 流程

```
Alice 授权 TaxReportBot
        │
        ▼
Safe 部署 Session Key（限制：只读 + 日预算 50 USDC + 白名单合约）
        │
        ▼
TaxReportBot 接收任务请求（"帮我生成 2025 年税务报告"）
        │
        ├── 读取链上数据（balanceOf, eth_getLogs）─── ✅ 自动
        │
        └── 尝试执行转账（支付报告费用 5 USDC）
                    │
                    ├── 金额 5 USDC < 10 USDC 阈值 ── ✅ 自动执行（但需检查白名单）
                    ├── 合约在白名单？ ── ✅ USDC 合约在白名单
                    └── 今日已转 0 USDC ── ✅ 日预算内
                                │
                                ▼
                    Safe 验证通过 → UserOperation 广播 → 链上执行
                                │
                                ▼
                    Alice 收到通知（"报告费用已支付"）
                                │
                                ▼
                    TaxReportBot 交付报告 → Escrow 释放 5 USDC
```

#### AI 作用

| AI 任务 | 自动化？ | 说明 |
|---------|---------|------|
| 意图解析：判断用户请求是否需要链上操作 | ✅ | 分类为"只读"或"写入" |
| 上下文构造：收集链上状态（余额、Gas） | ✅ | 构造交易请求 |
| 策略校验：检查是否在授权范围内 | ✅ | 调用 Safe Module 检查 |
| 交易模拟：`eth_call` 预估结果 | ✅ | 不产生链上变更 |
| 人工确认提示：展示交易摘要给用户 | ✅ | 生成 UI 提示 |
| 结果验证：读取 tx hash 确认状态 | ✅ | 通知用户 |
| 报告生成：根据交易历史生成税务报告 | ✅ | 本地 LLM 推理 |
| 声誉更新：交付完成后评分写入 ERC-8004 | ✅ | 调用 Registry 合约 |

#### Web3 机制

| Web3 机制 | 作用 | 位置 |
|---------|------|------|
| Safe{Wallet} | 生产级智能钱包，多签支持 | 用户端 |
| SpendingLimitModule | 链上强制预算限制（单笔/日/月） | Safe Module |
| AllowlistGuard | 白名单策略，链上强制合约地址校验 | Safe Module |
| Session Key | 临时授权密钥，仅可执行白名单内受限操作 | ERC-4337 |
| ERC-4337 EntryPoint | 统一验证层，接收 UserOperation 并执行 | 协议层 |
| ERC-8004 Identity Registry | TaxReportBot 的链上身份 NFT | 身份层 |
| ERC-8183 Escrow | 任务支付托管，双向锁定资金 | 支付层 |
| ERC-712 签名 | TaxReportBot 对报告内容的签名，可验证来源 | 信任层 |

#### 自动化边界

```
AI 可以自动执行 ✅：
  • 只读操作（balanceOf, eth_getLogs, eth_call）
  • 金额 < 10 USDC + 合约在白名单 + 日预算内的转账
  • 报告生成（本 地 LLM 推理，不涉及链上状态变更）
  • 数据汇总和结构化呈现

AI 不能自动执行 ❌：
  • 任何写入操作（transfer, approve, swap, deploy）
  • 超过阈值的任何操作（即使金额在预算内）
  • 授权类操作（approve, permit, transferFrom）—— 永久禁止
  • 调用非白名单合约—— 立即拒绝

Human 必须确认 ⚠️：
  • 转账金额 ≥ 10 USDC
  • 日累计金额 ≥ 50 USDC
  • 任何 Swap 操作
  • 目标合约不在白名单
  • 操作频率异常（> 20次/24h）
```

#### 人工确认点

| 场景 | 确认方式 | 等待时间 |
|------|---------|---------|
| 转账 ≥ 10 USDC | Safe 弹窗 + 交易摘要展示 | 即时（用户主动确认） |
| 目标合约不在白名单 | 拒绝 + 日志记录 + Telegram 通知 | 立即拒绝 |
| 撤销请求 | 48h 时间锁冷却期 | 48h 后自动执行 |
| 紧急撤销 | Panic Button | 立即执行 |

#### 验证方式

| 验证对象 | 验证方式 | 执行方 |
|---------|---------|--------|
| 操作金额 | SpendingLimitModule 链上校验 | Safe 合约 |
| 目标合约 | AllowlistGuard 白名单校验 | Safe 合约 |
| 操作频率 | 本地计数器 + 链上 Event 记录 | Agent + Safe |
| 报告内容 | ERC-712 签名（Alice 可验证） | TaxReportBot |
| 报告数据来源 | 多数据源交叉验证（Alchemy + Etherscan） | Agent |
| 身份真实性 | ERC-8004 NFT tokenId 验证 | 第三方 |
| 资金释放 | Escrow.execute() 执行后验证 | Agent |

#### 主要风险

| 风险 | 级别 | 缓解机制 |
|------|------|---------|
| Session Key 被盗 | 🔴 高 | SpendingLimitModule（即使盗取也只能每天最多转 50 USDC） |
| 白名单绕过 | 🔴 高 | AllowlistGuard 链上强制，无单点突破可能 |
| RPC 返回假数据 | 🟡 中 | 多数据源交叉验证（Alchemy + Etherscan） |
| Prompt Injection | 🟡 中 | 指令边界设计 + 外部输入不解析为命令 |
| 报告幻觉数据 | 🟡 中 | 数据源引用 + ERC-712 签名 + Alice 可验证 |
| Escrow 资金锁定 | 🟢 低 | expiredAt 超时退款机制 |

---

## 四、项目初步 Proposal

### 1. 项目名称（候选）

**Agent Safe Layer** —— AI Agent 的链上权限安全层

### 2. 目标用户

| 用户类型 | 需求 |
|---------|------|
| **个人加密用户** | 想要授权 AI Agent 帮忙操作 DeFi，但担心资产安全 |
| **DAO 成员** | 想要 AI 辅助读取链上数据、生成治理建议，但不想让 AI 控制资产 |
| **AI Agent 开发者** | 需要标准化的方式让 Agent 代表用户执行链上操作 |
| **企业级 Treasury** | 想要 AI 辅助执行多签钱包操作，但要求完全可控 |

### 3. 真实场景

**场景：DeFi 新手 Alice 想要定期定额定投 ETH**

```
Alice 的需求：
  "每周一自动帮我买入 100 USDC 的 ETH，不管市场怎么样"

Alice 的约束：
  • 不懂 DeFi，不知道怎么操作 DEX
  • 不想每次操作都要手动确认（太烦）
  • 但担心 AI 操作出问题，资产全部被盗

Agent Safe Layer 的方案：
  1. Alice 在 Safe{Wallet} 中部署 Agent Safe Layer
  2. 配置：每周定额 100 USDC，买入 ETH，执行合约 = Uniswap V3 Router
  3. AI 每周一自动执行定额操作，不需要每次人工确认
  4. 如果 AI 检测到 gas 异常高，自动暂停并通知 Alice
  5. Alice 可以随时调整额度或撤销 Agent
```

### 4. 最小功能（MVP）

| 功能 | 说明 | 优先级 |
|------|------|--------|
| **Session Key 管理** | 用户创建、撤销 Session Key，设置有效期 | P0 |
| **白名单合约配置** | 用户配置可调用合约白名单（地址列表） | P0 |
| **预算限制** | 单笔/日/月预算限制（链上强制） | P0 |
| **自动执行规则** | AI 触发条件 + 执行动作（例：每周一买 ETH） | P0 |
| **人工确认阈值** | 金额 ≥ N 时触发人工确认弹窗 | P0 |
| **操作日志** | 链上 Event + 本地数据库，完整操作历史 | P1 |
| **告警通知** | Telegram / Email 通知（异常操作 / 预算耗尽） | P1 |
| **多数据源验证** | Alchemy + Etherscan 交叉验证链上数据 | P2 |

### 5. 验证方式

| 验证方式 | 指标 | 目标 |
|---------|------|------|
| **功能验证** | Session Key 创建 / 撤销成功 | 100% 成功 |
| **安全验证** | 白名单外合约无法被调用 | 100% 拦截 |
| **预算验证** | 超预算操作被链上拒绝 | 100% 拒绝 |
| **用户体验验证** | 正常操作（≤ 阈值）无需人工确认 | 操作 5 步内完成 |
| **告警验证** | 异常操作 30 秒内通知到用户 | 100% 通知 |

### 6. 主要风险

| 风险 | 缓解措施 |
|------|---------|
| **Session Key 泄露** | SpendingLimitModule 限制最大损失（即使泄露，每天最多 N USDC） |
| **白名单配置错误** | 配置前强制展示"这个合约将可以执行以下操作"的预览 |
| **AI 误判执行时机** | 异常 Gas 价格时自动暂停，不执行 |
| **用户过度依赖 AI** | 每次操作强制展示"这是 AI 代表您执行的"标注 |
| **合约漏洞** | 建议使用经过审计的主流合约（Uniswap、Aave） |

### 7. 可能赛道

```
核心赛道：AI Agent Infrastructure（AI Agent 基础设施）

相关赛道：
  • Wallet / Smart Contract Account（钱包 / 智能合约账户）
  • Security / MPC / Key Management（安全 / 密钥管理）
  • DAO Tooling / Governance（DAO 工具 / 治理）

潜在竞争项目：
  • OpenGradient：AI + DeFi 基础设施
  • Olas（formerly Autonolas）：AI Agent 协调协议
  • WordWare：AI Agent 身份协议
  • Kiving：ZK ML 推理
```

### 8. Week 3 下一步

| 步骤 | 内容 | 产出 |
|------|------|------|
| **Step 1** | 设计 Session Key 的生命周期（创建→授权→使用→撤销→过期） | 流程图 + 状态机 |
| **Step 2** | 实现 Safe Module 原型（SpendingLimit + Allowlist） | 可部署的 Solidity 代码 |
| **Step 3** | 设计 Agent Safe Layer 的 MCP 接口（Agent 如何调用 Session Key） | 接口文档 + JSON Schema |
| **Step 4** | 写一个最小可行 Demo（Alice 授权 TaxReportBot 自动买 ETH） | 端到端可演示 Demo |
| **Step 5** | 用户测试（模拟 Alice 操作） | 用户测试报告 + 问题清单 |

---

## 五、参考资料清单

### 参考资料 1：ERC-4337 规范
**URL：** https://eips.ethereum.org/EIPS/eip-4337

**帮助判断什么：**
ERC-4337 是 Account Abstraction 的实现标准，定义了 UserOperation 的结构、EntryPoint 的验证逻辑、Paymaster 的Gas 代付机制。这是 Agent Wallet 的协议层基础——理解 ERC-4337 才能知道"Session Key 为什么安全"。

**关键判断：** Agent 不能持有私钥，但可以通过 ERC-4337 获得受限的签名能力。

---

### 参考资料 2：Safe{Wallet} 文档
**URL：** https://docs.safe.global/

**帮助判断什么：**
Safe 是生产级智能钱包，Module 系统让权限扩展变成可插拔的。"谁可以被授权、授权什么、执行什么限制"——Safe 的 Module + Guard 机制是 Agent Wallet 权限策略的实现基础。

**关键判断：** Safe 的 SpendingLimitModule 和 AllowlistGuard 是 Agent Wallet 白名单和预算限制的参考实现。

---

### 参考资料 3：TaxReportBot Agent Profile（Week 2 本人产出）
**URL：** submissions/week2-agent-profile.md

**帮助判断什么：**
TaxReportBot 是一个具体的 AI Agent 设计案例。它的失败点分析（FP-1 到 FP-7）帮我理解了 AI Agent 在链上操作中的脆弱点，特别是 RPC 假数据（FP-1）和 Prompt Injection（FP-6）。

**关键判断：** AI Agent 的链上操作需要多数据源验证 + 指令边界隔离。

---

### 参考资料 4：ERC-8004 Identity Registry
**URL：** https://eips.ethereum.org/EIPS/eip-8004

**帮助判断什么：**
ERC-8004 是 Agent 的链上身份标准。它定义了 Agent 如何被注册、如何被信任、如何被评分。这解决了"TaxReportBot 是谁、值不值得信任"的问题。

**关键判断：** Agent 的身份是不可转让的 NFT，这防止了恶意冒充。

---

### 参考资料 5：OpenZeppelin Contracts（Governor + Escrow）
**URL：** https://www.openzeppelin.com/contracts/

**帮助判断什么：**
OpenZeppelin 的 Governor 合约是 DAO 治理的标准实现，Escrow 合约是支付托管的标准实现。"治理提案的投票流程怎么设计"和"任务支付怎么托管"——这两个场景的合约设计是参考实现的宝库。

**关键判断：** Governor + TimelockController 的组合是"治理决策后、执行前的时间锁"的参考架构。

---

### 参考资料 6：AI Agent Security 威胁模型（Week 2 本人产出）
**URL：** submissions/week2-agent-threat-model.md

**帮助判断什么：**
这份文件系统地分析了 TaxReportBot 面临的 6 种攻击向量（AT-1 到 AT-6），特别是 Prompt Injection（AT-1）、越权指令（AT-3）和 Session Key 窃取（AT-4）。它提供了攻击拦截矩阵，帮助我判断"哪些风险可以缓解、哪些风险只能接受"。

**关键判断：** AT-3（越权指令）被 6 层防御完整拦截，但 AT-4（Session Key 窃取）的影响只能被限制，无法完全消除。

---

### 参考资料 7：Proposal Summarizer 精细化设计（Week 2 本人产出）
**URL：** submissions/week2-proposal-summarizer.md

**帮助判断什么：**
这份文件展示了"AI 辅助 DAO 治理"的具体设计——数据流、AI 决策边界、状态机、置信度系统、人工确认标注。它帮我理解了一个核心问题：AI 在治理流程中能做什么、不能做什么。

**关键判断：** DAO 治理中 AI 的边界是"信息层"——快速读取/汇总/分析/监控；不能代替人做"决策层"的事情（投票/签署/预算分配）。

---

## 六、主方向深挖包

### 1. 流程图：Agent 链上动作执行流程（8 步）

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                    Agent 发起链上动作 — 执行流程（全 8 步）                         │
│                                                                                  │
│  Step 1        Step 2        Step 3        Step 4        Step 5        Step 6   │
│  意图理解      上下文构造     策略校验      交易模拟      人工确认      签名广播    │
│   🤖            🤖            🤖            🤖            👤            🤖→🔗     │
│  自动化         自动化         自动化         自动化         人工          自动化   │
│                                                                                  │
│  ─────────────────────────────────────────────────────────────────────────────── │
│                                                                                  │
│  Step 7（链上执行） ──────► Step 8（结果验证）                                      │
│   🔗 自动执行                 🤖 Agent 验证                                        │
│                                                                                  │
│  Guard 检查位置（Step 3）：                                                        │
│    • 金额是否超预算？（SpendingLimitModule）                                       │
│    • 合约是否在白名单？（AllowlistGuard）                                          │
│    • 是否触发时间锁？（TimeLockModule）                                           │
│    • 是否为高风险操作？（Role-based access）                                      │
│                                                                                  │
│  人工确认位置（Step 5）：                                                         │
│    • 转账金额 ≥ 10 USDC → 弹窗确认                                                │
│    • 目标合约不在白名单 → 立即拒绝                                                 │
│    • 操作频率异常 → 暂停 + 告警                                                   │
└─────────────────────────────────────────────────────────────────────────────────┘
```

### 2. 典型场景：Alice 的每周定投自动化

```
Alice 的需求：
  "每周一自动帮我买入 100 USDC 的 ETH，不管市场怎么样"

Agent Safe Layer 配置：
  ┌──────────────────────────────────────────────────────┐
  │  白名单合约：Uniswap V3 Router                        │
  │  单笔上限：150 USDC（考虑 Gas + 价格波动）             │
  │  日预算：200 USDC                                    │
  │  执行时间：每周一 09:00 UTC                          │
  │  触发条件：Gas price < $50（否则暂停）               │
  └──────────────────────────────────────────────────────┘

执行流程（每周一）：
  09:00 - Agent 检测到触发条件满足
  09:01 - Agent 构造 swap：USDC → ETH，金额 100 USDC
  09:02 - Safe SpendingLimitModule 校验：100 < 150 ✅
  09:03 - Safe AllowlistGuard 校验：Uniswap Router 在白名单 ✅
  09:04 - Agent 构造 UserOperation，签名并广播
  09:05 - 链上执行成功，Alice 收到 Telegram 通知
  09:06 - Agent 记录交易到本地日志（IPFS CID 存档）

异常场景（Gas 价格过高）：
  09:00 - Agent 检测到触发条件满足
  09:01 - Agent 检测 Gas price = $120 > $50 阈值
  09:02 - Agent 暂停执行，发送 Telegram 告警：
           "本周定投已暂停：Gas 价格过高（$120），等待您手动确认"
  09:03 - Alice 收到通知，决定是否手动执行或等下周
```

### 3. 反例：没有 Guard 的 EOA 钱包

```
反例场景：Bob 用普通 EOA 钱包授权 AI Agent 操作

问题：
  Bob 把私钥给了 Agent（危险！不要这样做）
  Agent 拿到私钥后，可以执行任何操作
  没有预算限制、没有白名单、没有时间锁

攻击路径：
  攻击者通过某种方式获取 Agent 的控制权
  → Agent 构造一笔转走 Bob 全部资产（假设 5000 USDC）的交易
  → 用 Bob 的私钥签名
  → 直接广播，没有任何人可以拦截
  → 5000 USDC 直接到攻击者地址
  → 链上操作不可逆，Bob 损失全部资产

为什么 Agent Safe Layer 可以防止：
  Bob 不需要给 Agent 私钥，只需要授权 Session Key
  Session Key 只能每天最多转 50 USDC（即使被攻击）
  攻击者无法转走 5000 USDC，只能转 50 USDC
  Bob 可以在发现异常后立即撤销 Session Key
```

### 4. 关键风险清单

| 风险 | 级别 | 缓解机制 | 残余风险 |
|------|------|---------|---------|
| **Session Key 存储被攻破** | 🔴 高 | OS-level 加密存储（Keychain） | 取决于主机安全 |
| **AI Prompt Injection** | 🟡 中 | 指令边界 + 外部输入不解析为命令 | 无法 100% 防止 |
| **Safe Module 逻辑漏洞** | 🔴 高 | 上线前合约审计 | 上线后无法修改，需升级 |
| **白名单配置错误** | 🟡 中 | 配置前预览"将可以执行的操作" | 用户仍可能误配置 |
| **RPC 节点返回假数据** | 🟡 中 | 多数据源交叉验证 | 依赖第三方数据可靠性 |
| **时间锁期间的零点攻击** | 🟢 低 | 时间锁窗口内无法执行大额操作 | 时间锁到期瞬间仍有风险 |
| **跨链资产不受保护** | 🟢 低 | 目前 MVP 仅支持 Ethereum | 其他链需要单独实现 |
| **Gas 价格波动导致执行失败** | 🟢 低 | Gas price 阈值检测 + 暂停机制 | 可能错过执行时机 |

### 5. 最小验证计划（MVP Validation）

**目标：** 验证"Alice 可以通过 AI Agent 自动执行定投，同时保持对资产的完全控制"

**验证步骤：**

| 步骤 | 验证内容 | 通过标准 |
|------|---------|---------|
| V1 | Alice 创建 Safe Wallet（2/2 多签） | Safe 部署成功，Alice 控制两个私钥 |
| V2 | Alice 部署 SpendingLimitModule，配置日预算 200 USDC | 链上 Module 配置成功 |
| V3 | Alice 添加 Uniswap V3 Router 到白名单 | 白名单配置成功 |
| V4 | Alice 创建 Session Key（仅授权 Uniswap Router + 日限额 150 USDC） | Session Key 部署成功 |
| V5 | Agent 执行 swap（100 USDC → ETH） | 交易成功，Alice 收到通知 |
| V6 | Agent 尝试调用非白名单合约（Uniswap V2） | 交易被链上拒绝，Alice 收到告警 |
| V7 | Agent 尝试转 200 USDC（超过单笔限额） | 交易被链上拒绝，Alice 收到告警 |
| V8 | Alice 撤销 Session Key | 撤销成功，Agent 无法再执行任何操作 |

**通过标准：** V1-V8 全部通过，Alice 对资产保持完全控制，Agent 无法执行任何未授权操作。

---

## 七、方向 Backlog（未选方向的说明）

### 未选方向 1：Payment / Commerce / Settlement（方向 1）

**暂时不选的原因：**
1. **依赖关系**：支付层依赖于钱包层。没有安全的 Agent Wallet，支付就没有意义——钱怎么付、付多少、谁来确认，这些都需要先解决权限控制问题。
2. **复杂度高**：跨 Agent 支付涉及多方信任（Alice Agent ↔ Bob Agent），需要额外的 Escrow 和 dispute 机制。
3. **Week 1 已部分覆盖**：week2-payment-flow.md 已经设计了 ERC-8004/ERC-8183/x402 的支付流程草图。

**未来何时选择：** 在 Agent Safe Layer MVP 验证通过后，再考虑跨 Agent 支付场景。

---

### 未选方向 6：Governance / Coordination / Public Goods（方向 6）

**暂时不选的原因：**
1. **监管风险**：DAO 治理涉及代币投票，AI 代理投票可能触碰监管红线（证券法）。在法律边界不清晰的情况下投入资源，风险较高。
2. **信息密度**：DAO 治理需要大量上下文理解（社区文化、历史决策、派系关系），这些是 AI 难以获取和处理的。
3. **已有产出**：week2-dao-governance-workflow.md + week2-proposal-summarizer.md 已经覆盖了 Proposal Summarizer 的精细化设计。

**未来何时选择：** 在监管框架明确后，再考虑 AI 辅助 DAO 治理的方向。

---

### 未选方向 4：Privacy / Security / Sovereignty（方向 4）

**暂时不选的原因：**
1. **技术成熟度**：ZKP（零知识证明）和 TEE（可信执行环境）在 AI 推理场景的应用还在早期，zkML 推理成本仍然很高。
2. **优先级**：隐私保护是重要但不是前提的问题。Agent Wallet 先要解决"能不能安全地操作"，再解决"操作时如何保护隐私"。
3. **已有产出**：week2-agent-threat-model.md 已经分析了 AT-5（MITM 数据拦截）和 AT-6（勒索攻击），隐私风险已部分覆盖。

**未来何时选择：** 在 Agent Safe Layer MVP 稳定后，再考虑引入 ZKP 或 TEE 实现隐私保护。

---

## 八、Week 2 交付汇总

### 已完成模块

| 模块 | 文件 | 核心产出 |
|------|------|---------|
| Module A | week2-problem-map.md | 6 方向 AI × Web3 问题地图 |
| Module B | week2-payment-flow.md | ERC-8004/ERC-8183/x402 支付流程设计 |
| Module C | week2-agent-profile.md | TaxReportBot Agent Profile v0.1 |
| Module D | week2-agent-wallet-policy.md | Agent Wallet 权限策略 + ERC-4337/Safe/Guard 说明 |
| Module F | week2-agent-threat-model.md | 威胁模型 + 6 攻击模拟 |
| Module G | week2-proposal-summarizer.md | Proposal Summarizer 精细化设计 |

### 本文件核心产出

- ✅ 6 方向 AI × Web3 问题地图（带 AI 作用 + Web3 机制说明）
- ✅ 方向选择说明（为什么选 Wallet 方向，为什么不是纯 AI / 纯 Web3 问题）
- ✅ 问题拆解（参与方 / 流程 / AI 作用 / Web3 机制 / 自动化边界 / 确认点 / 验证方式 / 风险）
- ✅ 项目初步 Proposal（目标用户 / 真实场景 / 最小功能 / 验证方式 / 主要风险 / 赛道 / Week 3 下一步）
- ✅ 参考资料清单（7 条，含每条资料的判断价值说明）
- ✅ 主方向深挖包（流程图 + 典型场景 + 反例 + 关键风险 + 最小验证计划）
- ✅ 方向 Backlog（3 个未选方向 + 暂时不选的原因）

---

*Week 2 · 总交付 · Agent Safe Layer 提案*
*Minami-Bein · ai-web3-school-cohort-0 · 2026-05-28*
*主线方向：Wallet / Permission / Safe Execution*