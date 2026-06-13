# Cat-Agent（喵星智能体）

> AI Agent 自主消费 · Cobo Agentic Wallet 授权管理 · Safe{Wallet} 链上预算管控
> An AI Agent that helps pets make autonomous purchases — with on-chain warrant controls and wallet authorization

[![Hackathon Cohort 0](https://img.shields.io/badge/Hackathon-AI%20x%20Web3%20School%20Cohort%200-blue)](https://github.com/Minami-Bein/ai-web3-school-cohort-0)
[![Status](https://img.shields.io/badge/status-Week%204%20Sprint-orange)](./hackathon/04-cat-agent-demo-day.md)
[![Demo Day](https://img.shields.io/badge/Demo-Day%20%E2%9C%85-brightgreen)](./hackathon/03-demo-story.md)

---

## Project Overview

Cat-Agent 是一个**宠物自主消费 AI Agent**。当主人不在家时，猫咪可以通过 AI 感知 + 链上授权，自主发起购买请求——买零食、买玩具。主人拥有完整的钱包控制权，所有交易受日限额 / 单笔限额双重保护。

**Demo Day 状态：** 最小闭环已在本地 Anvil 测试网跑通，完整交易可查证。

- 🐱 **角色：** 宠物（Cat Safe 钱包持有者）
- 🤖 **引擎：** AI 感知（意图识别）→ 规则引擎（Budget 检查）→ 链上执行（SpendingLimitModule）
- 🔐 **钱包：** Safe{Wallet} + SpendingLimitModule（Session Key 签名）
- ⛓️ **链：** Anvil 本地测试网（Sepolia 部署待完成）

---

## Problem

猫咪饿了会叫，但主人无法随时响应。现有宠物食品订购需要人类全程介入：收到信号 → 打开 App → 搜索 → 确认支付。

**核心矛盾：** 宠物有急切的消费需求，但没有金融自主权，无法直接使用支付系统。

更深层的问题是**信任**：主人如何放心让 AI Agent 持有钱包、替宠物花钱，而不担心超支或滥用？

---

## Why AI

AI 是唯一能够弥合"宠物意图"与"金融操作"之间鸿沟的技术：

- **感知层：** AI 多模态模型（摄像头 + 麦克风）理解猫咪当前行为——是在讨食、在玩耍、还是无聊？
- **推理层：** AI Agent 在预算约束内做决策（买什么、多少钱、是否合规），无需人类实时批准
- **执行层：** AI 自动协调链上交易、监控余额、记录日志

没有 AI，这套系统就无法理解"猫咪这个动作意味着它想吃金枪鱼"——它只是一台等待人类指令的机器。

---

## Why Web3

传统方案（App Store 礼品卡）让主人控制权过度集中；现有加密支付需要人类私钥签名。Web3 解锁的是**自主权下放**：

- **Safe{Wallet} + Session Key：** Agent 持有有限权限的会话密钥，主人随时可撤销——Agent 永远不接触主私钥
- **SpendingLimitModule：** 链上硬编码日限额 / 单笔限额，超出则交易被合约直接拦截，无法被覆盖
- **不可篡改的授权记录：** 每笔交易带上意图 + 签名 + 结果，全部上链可查

这不是"把信用卡放到链上"，而是**重新定义宠物与支付系统的关系**——Agent 在主人设定的边界内自主决策，边界本身由合约强制执行。

---

## How It Works

```
[猫咪行为] ──camera + mic──▶ [AI 感知层]
                                  │
                          intent: "buy_snack"
                          confidence: 0.94
                                  │
                                  ▼
                         [规则引擎]
                    Warrant 检查（日限额/单笔限额）
                                  │
                            ✅ 通过
                                  │
                                  ▼
                         [Safe{Wallet}]
                    Session Key 签名 USDC 转账
                                  │
                                  ▼
                    [SpendingLimitModule 合约]
                    ┌─ 日限额拦截（>10 USDC/天）
                    └─ 单笔限额拦截（>10 USDC/笔）
                                  │
                                  ▼
                         [链上交易广播]
                                  │
                                  ▼
                         [Telegram 通知主人]
                    "Mimi买了金枪鱼罐头（2 USDC），
                     日预算剩余：8 USDC"
```

**关键参数：**
- 日限额：10 USDC
- 单笔限额：10 USDC
- Cat Safe：`0x70997970C51812dc3A010C7d01b50e0d17dc79C8`
- 商家：`0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC`

---

## Demo

> 📖 完整 Demo 故事（含分镜脚本）：[hackathon/03-demo-story.md](./hackathon/03-demo-story.md)
> ⚠️ 本次 Demo 基于 Anvil 本地测试网；Z.AI 感知层、Cobo CAW 为 Mock

### 3 分钟 Demo 主流程

**场景：** 主人外出，猫咪 Mimi 想吃金枪鱼罐头

```
[00:00] 画面：Mimi 对着摄像头叫，走到食物盆旁边
         ┌─────────────────────────────────────────┐
         │  AI 感知：检测到猫咪叫声 + 靠近食物盆      │
         │  intent: buy_snack  |  confidence: 94%  │
         └─────────────────────────────────────────┘
              ↓
[00:45] 画面：规则引擎日志滚动
         ┌─────────────────────────────────────────┐
         │  [Policy Engine] Warrant check           │
         │  日限额：10 USDC ✓                       │
         │  单笔限额：10 USDC ✓                     │
         │  商户白名单：通过 ✓                       │
         │  结论：APPROVED                          │
         └─────────────────────────────────────────┘
              ↓
[01:15] 画面：链上交易签名 + 广播
         ┌─────────────────────────────────────────┐
         │  Safe{Wallet} Session Key signs TX       │
         │  Function: transferUSDC(address,uint256) │
         │  Amount: 2 USDC → Merchant              │
         │  TX Hash: 0x70d3...accaa                │
         └─────────────────────────────────────────┘
              ↓
[01:45] 画面：Etherscan 截图 / cast receipt 输出
         ┌─────────────────────────────────────────┐
         │  Status: ✅ 1 (success)                 │
         │  Gas Used: 58,194                       │
         │  Logs: 2 events                         │
         │  Cat Safe USDC: 94 | Merchant: 6        │
         └─────────────────────────────────────────┘
              ↓
[02:30] 画面：Telegram 通知弹出
         ┌─────────────────────────────────────────┐
         │  🐱 Mimi 买了金枪鱼罐头                  │
         │  💸 2 USDC | 日预算剩余：8 USDC          │
         │  🔗 tx hash                             │
         └─────────────────────────────────────────┘
              ↓
[03:00] 画面：防御测试——超额交易被链上拦截
         ┌─────────────────────────────────────────┐
         │  尝试支付 11 USDC（> 单笔限额）           │
         │  Status: ❌ REVERTED                    │
         │  Reason: per_tx_limit exceeded          │
         │  日限额 / 单笔限额双重拦截验证 ✅         │
         └─────────────────────────────────────────┘
```

**Demo 中 Mock / 未完成的部分：**

| 模块 | 状态 | 说明 |
|------|------|------|
| AI 感知层（Z.AI） | 🔶 Mock | 意图通过脚本模拟，confidence=0.94 |
| Cobo CAW Warrant | 🔶 Mock | Warrant 规则为本地 JSON，未接入 CAW SDK |
| Telegram 通知 | 🔶 部分完成 | 代码存在，Demo 中以日志替代 |
| Sepolia 部署 | 🔴 未完成 | 当前在 Anvil 本地测试网运行 |

---

## Validation

所有验证材料可在 [hackathon/04-cat-agent-demo-day.md](./hackathon/04-cat-agent-demo-day.md) 复查。

### 可查证的链上证据

| 验证项 | 证据 |
|--------|------|
| **成功交易** | Tx: `0x70d3db2fac2ced63a6ba8263b0fa290dd1d9ef5b4778e7a68014768d0c0accaa` — status=1, Gas=58194, 2 event logs |
| **Cat Safe USDC 余额** | 94 USDC（转账 3×2 USDC 后）|
| **商家 USDC 余额** | 6 USDC（收款 3×2 USDC）|
| **日限额状态** | `getDailyRemaining()` → 8 / 10 USDC |
| **SLM 存储 slot0** | USDC 地址正确写入 |
| **SLM 存储 slot1** | dailySpent = 4e6（= 4 USDC，已花）|
| **单笔超额拦截** | 11 USDC 交易 revert: `per_tx_limit exceeded` ✅ |
| **日限额超额拦截** | 逻辑存在，待单独测试验证 |

**验证命令（Anvil 本地）：**
```bash
# 交易回执
cast receipt 0x70d3db2fac2ced63a6ba8263b0fa290dd1d9ef5b4778e7a68014768d0c0accaa --rpc-url http://127.0.0.1:8545

# 日限额剩余
cast call 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0 \
  "getDailyRemaining()(uint256)" \
  --from 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 \
  --rpc-url http://127.0.0.1:8545
```

### 技术关键踩坑

⚠️ **`eth_sendTransaction` Anvil bug：** Anvil unlocked 模式下自动签名的交易有缺陷（status=0x0，gas 不够）。解决：改用 `web3.eth.account.sign_transaction` 本地签名 + `eth_sendRawTransaction`。

⚠️ **函数选择器：** 手动拼接 hex 时应使用 `web3.contract.functions.transferUSDC(...).build_transaction()` 自动编码，避免选择器错误。

---

## Risks

| 风险 | 可能性 | 影响 | 缓解措施 |
|------|--------|------|----------|
| **Cobo Agentic Wallet 签名方案未适配** | 🔴 高 | Agent 无法在真实 Safe{Wallet} 发起交易 | P0：验证 `agentkit` 是否支持 `eth_sendRawTransaction` |
| **Sepolia 真实 USDC 获取** | 🟡 中 | 无法在主网完成端到端 Demo | 已准备多水龙头备选 |
| **Z.AI API Key 未及时获批** | 🟡 中 | 感知层降级至 OpenAI Fallback | Policy Engine 已解耦，切换不影响链上逻辑 |
| **日限额重置依赖 block.timestamp** | 🟡 中 | 验证者可 manipulation 时间戳 | 生产环境应使用 Chainlink Time Oracle |
| **Session Key 撤销机制未测试** | 🟡 中 | Agent 权限无法及时回收 | 待接入 Safe{Wallet} SDK 后验证 |

详细风险分析：[13-project-prerequisites-risk-fallback.md](./hackathon/13-project-prerequisites-risk-fallback.md)

---

## Next Steps

| 优先级 | 任务 | 依赖 |
|--------|------|------|
| **P0** | 集成 Cobo Agentic Wallet 签名 API（`agentkit`）| Cobo agentkit docs |
| **P0** | Sepolia 测试网部署（真实 USDC）| Sepolia USDC 水龙头 |
| **P1** | Safe{Wallet} 1-of-1 创建 + Session Key 绑定 | Safe{Wallet} SDK |
| **P2** | 竞品对标（Dogo、CatFi）| Token Terminal |
| **P2** | Pitch Deck / Demo Video | 设计资源 |

---

## Team

- **Minami-Bein** — AI × Web3 School Cohort 0

---

## License

MIT