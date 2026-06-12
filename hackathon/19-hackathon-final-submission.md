# Cat-Agent 最终提交材料包

> AI × Web3 School Cohort 0 | 黑客松最终提交
> 提交日期：2026-06-12
> 项目状态：Sprint 4 冲刺启动前

---

## 一、GitHub 仓库

**仓库地址：** https://github.com/Minami-Bein/ai-web3-school-cohort-0

**主要目录：**
```
ai-web3-school-cohort-0/
├── hackathon/               # 全部 18 个设计文档
│   ├── 01-direction-card.md
│   ├── 02-proposal-memo.md
│   ├── 07-week4-verification.md    # Week 4 详细验证清单
│   ├── 08-architecture-minimal-loop.md  # 架构图
│   ├── 10-sponsor-sdk-integration-plan.md # Sponsor 接入计划
│   ├── 11-cobo-agentic-wallet-design.md  # Cobo CAW 设计文档
│   ├── 12-zai-long-horizon-task-design.md  # Z.AI 任务设计
│   ├── 13-project-prerequisites-risk-fallback.md  # 风险+决策树
│   ├── 17-week4-mvp-verification.md   # MVP 验证看板
│   └── 18-demo-day-retrospective.md   # 项目复盘
├── daily/                   # 每日学习笔记
├── submissions/             # 前几周作业
├── README.md                # 项目主页
└── profile.md              # 学习者画像
```

---

## 二、项目一句话简介

**Cat-Agent（喵星智能体）：让宠物拥有自主消费能力的 AI Agent，猫咪通过叫声和肢体语言表达需求，AI 确认意图后在链上钱包预算内完成支付。**

---

## 三、赛道选择

| 赛道 | 选择 |
|------|------|
| **Cobo — Agentic Economy × Cobo Agentic Wallet** | ✅ Primary Track |
| **Z.AI — Web3 × Long-Horizon Task** | ✅ Secondary Track |

---

## 四、项目解决的问题

**核心张力：宠物有即时消费需求，但没有任何数字身份和支付能力。**

现有智能喂食器只能"定时定量"，无法感知宠物真实情绪。宠物饿了只能等待主人——主人必须接收信号、打开 App、搜索商品、确认支付，整个过程可能错过最佳投喂时机。

**我们解决的三个具体问题：**

**问题 1 — 意图感知缺失**
猫咪通过叫声/肢体语言表达需求，现有系统无法理解这些信号。
→ 多模态 AI（音频频率 + 视觉姿态）实时推理宠物意图，置信度 >90% 才触发购买，连续两轮交互确认真实需求。

**问题 2 — 消费边界缺失**
没有机制限制宠物（或熊孩子）乱买的上限。
→ CAW Warrant（链上授权）+ Safe{Wallet} SpendingLimitModule 双重拦截，日累计 10 USDC 硬上限，单笔 10 USDC 硬上限。

**问题 3 — 数字身份缺失**
宠物在数字世界中没有独立账户，无法拥有自己的消费记录。
→ 每只宠物拥有独立 Safe{Wallet} + Session Key，Agent 在 Warrant 授权范围内代理执行，每笔交易带审计日志。

---

## 五、已跑通的主流程（Minimal Viable Loop）

```
[猫咪行为输入]
  摄像头 + 麦克风 采集音频 + 图像
        ↓
[意图推理引擎]
  Z.AI 多模态推理 / OpenAI Fallback / 本地规则引擎
  输出：intent + confidence_score
  阈值：≥ 90% 才继续，否则拒绝 + 通知主人
        ↓
[CAW Policy Engine 三重校验]
  Budget（日限额 10 USDC）+ Scope（白名单）+ Time Window（Warrant 有效期）
        ↓
[Safe{Wallet} 执行]
  Session Key 签名
  USDC Transfer → Ethereum Sepolia
        ↓
[链上记录 + 通知]
  Etherscan 可查 tx_hash
  Telegram 推送主人（含金额 + 商品名 + 日预算剩余）
        ↓
[审计日志存档]
  本地 JSON（logs/），IPFS-ready
```

**MVP 成功标准：一笔完整交易，Etherscan 可查 + Telegram 通知到账。**

---

## 六、Sponsor 对齐说明

### 6.1 Cobo — Agentic Economy × Cobo Agentic Wallet

**Cat-Agent 如何使用 Cobo CAW：**

Cat-Agent 的钱包架构完全基于 Cobo Agentic Wallet 设计理念构建：

```
Cobo 提供的层级：
  CAW Warrant 框架（链上标准）
        ↑
  Cat-Agent 实现：
  - Warrant 配置（budget/scope/time/health）
  - Policy Engine（业务层三重检查）
  - SpendingLimitModule（链上硬限制）
  - Session Key 管理（Agent 签名密钥）
  - AllowlistGuard（白名单地址限制）
```

**具体使用场景：**

| Cobo CAW 能力 | Cat-Agent 应用方式 |
|-------------|-----------------|
| **链上 Warrant** | 每只猫有独立 Warrant，日预算/单笔限额/有效期限时由主人签名配置 |
| **Session Key** | AI Agent 使用 Session Key 签名支付，主私钥永不触碰日常交易 |
| **SpendingLimitModule** | 链上硬拦截超限交易（>10 USDC 单笔 或 >10 USDC 日累计） |
| **AllowlistGuard** | 仅允许调用白名单合约地址（商品配送 API） |
| **Warrant 撤销** | 主人可随时撤销，撤销后 Agent 无法继续执行支付 |

**核心价值：** Cobo CAW 让宠物拥有"受限的钱包"而非"全权钱包"，即使 Agent 被攻破，攻击者也只能在 10 USDC/日的限额内操作，而非转移全部资产。

**当前状态：** CAW SDK 集成待 Cobo 审批。当前使用本地 JSON 模拟 Warrant 逻辑，Policy Engine 与 CAW SDK 完全解耦，审批通过后可无缝切换到链上 Warrant。

### 6.2 Z.AI — Web3 × Long-Horizon Task

**Cat-Agent 如何使用 Z.AI：**

Cat-Agent 是一个典型的 Long-Horizon Task 系统，需要多轮推理、多次工具调用、跨时序状态维护：

```
猫咪说"我要吃鱼"（简单语音输入）
  ↓
分解为：
  1. 理解意图（多模态 AI 推理）
  2. 查询商品库存和价格（外部 API）
  3. 检查预算和权限（CAW Warrant 检查）
  4. 构造支付交易（Safe{Wallet}）
  5. 广播到链上（Ethereum）
  6. 验证交易确认（RPC 查询）
  7. 触发配送（第三方 API）
  8. 记录审计日志（IPFS + 链上 Event）
  9. 更新偏好权重（RLPF）
  10. 通知主人（Telegram）
```

**Z.AI 提供的能力：**

| Z.AI 能力 | Cat-Agent 应用场景 |
|---------|-----------------|
| **多模态推理** | 同时处理摄像头图像 + 麦克风音频，推理猫咪意图和置信度 |
| **LLM Strategy Layer** | ReAct 推理循环，分解任务、管理执行状态、自我修复错误 |
| **工具调用编排** | 按顺序调用 policy_check → safe_execute → wait_confirmation |
| **长上下文维持** | ReAct 循环可能产生 20+ 推理步骤，Z.AI 上下文窗口足够 |
| **状态持久化** | TaskState 存储到 IPFS，支持从断点恢复 |
| **x402 支付保护** | Z.AI 推理 API 可用 x402 保护，实现按调用量付费 |

**Z.AI 的独特价值：**
- ~1.2s 多模态推理延迟，满足猫咪交互实时性要求
- 原生多模态：同时处理图像 + 音频，无需拼接多个单模态 API
- 原生工具调用：Z.AI 支持 MCP 协议，Cat-Agent 工具可通过标准接口接入

**当前状态：** Z.AI API Key 待审批。当前使用 OpenAI GPT-4o + Whisper 作为 Fallback，审批通过后切换到 Z.AI。

---

## 七、风险说明

### 7.1 技术风险

| 风险 | 概率 | 影响 | 缓解措施 | 残余风险 |
|------|------|------|---------|---------|
| Z.AI API Key 未获批 | 40% | 感知层降级为 OpenAI Fallback | 保守阈值 >90%，多帧融合 | 低 |
| Cobo CAW Access 未获批 | 25% | Policy Engine 使用本地 JSON | Policy Engine 与 CAW SDK 完全解耦 | 中 |
| Safe SpendingLimitModule 配置错误 | 25% | 交易全部拦截或全部放行 | 测试网部署 + P-01~P-08 边界测试 | 低 |
| 猫咪意图置信度不稳定 | 35% | 频繁"intent unknown" | 保守阈值 + 手动触发 fallback | 中 |

### 7.2 产品风险

| 风险 | 概率 | 影响 | 缓解措施 | 残余风险 |
|------|------|------|---------|---------|
| 猫咪意图识别准确率不足 | 高 | 频繁拒绝，Demo 体验断断续续 | 阈值 >90%，多帧融合 | 中 |
| 主人不信任 AI 判断 | 高 | 推广困难 | 所有购买需主人最终确认（非全自动） | 低 |
| 评委质疑"宠物消费"是伪需求 | 中 | 影响评分 | Demo 聚焦技术可行性（CAW + Safe 支付闭环） | 低 |

### 7.3 时间风险

| 风险 | 概率 | 影响 | 缓解措施 | 残余风险 |
|------|------|------|---------|---------|
| Sprint 4 期间外部 API 未接入 | 中 | Demo 效果打折扣 | Fallback 提前冻结，确保 Mock 模式可演示 | 低 |
| Demo 录屏需要多次重拍 | 中 | 耽误提交 | 提前写好演示台词，准备好模拟器 | 无 |

### 7.4 安全风险

| 风险 | 概率 | 影响 | 缓解措施 | 残余风险 |
|------|------|------|---------|---------|
| Session Key 被窃取 | 低 | 日损失 ≤10 USDC | 每日轮换 + 加密存储 + 链上硬限制 | 极低 |
| CAW Warrant 被恶意修改 | 低 | 主人主私钥签名，Agent 无修改权 | 极低 |
| Prompt Injection | 低 | 外部输入（猫咪行为）不解析为命令 | 极低 |

---

## 八、验证材料清单

### P0 — 必须通过（黑客松评委必查）

| # | 材料 | 状态 | 说明 |
|---|------|------|------|
| 1 | Etherscan 成功交易截图 | 待完成（6/23） | tx_hash 可查，日限额 10 USDC 内任意金额 |
| 2 | SpendingLimitModule 拦截截图 | 待完成（6/23） | 发送 11 USDC 被链上拒绝 |
| 3 | 意图推理日志样例 | 待完成（6/24） | 含 intent + confidence_score + timestamp |
| 4 | README + 架构图 | ✅ 已完成 | 评委直接可读，60 秒理解项目 |
| 5 | Demo 录屏 | 待完成（6/26） | 完整闭环：猫叫 → 推理 → 支付 → 通知 |

### P1 — 重要材料

| # | 材料 | 状态 | 说明 |
|---|------|------|------|
| 6 | Policy Engine 8 边界测试 | 待完成（6/24） | Budget/Scope/Time 全拦截 |
| 7 | Telegram 通知截图 | 待完成（6/25） | 主人收到含 tx_hash 的支付结果消息 |
| 8 | CAW Warrant 本地 JSON | ✅ 已完成 | `configs/caaw_templates.json`，逻辑与链上一致 |
| 9 | Sponsor Integration Plan | ✅ 已完成 | `10-sponsor-sdk-integration-plan.md` |

### P2 — 加分材料

| # | 材料 | 状态 | 说明 |
|---|------|------|------|
| 10 | Agent Trace 完整链路 | 待完成 | 含 audio_features + visual_features |
| 11 | 3 次稳定交易记录 | 待完成 | 3 个不同 tx_hash |
| 12 | IPFS 审计日志存档 | 待完成 | CID + 验证链接 |

---

## 九、Sprint 4 每日计划

```
Day 1（6/23 周一）：全链路跑通
  → Safe{Wallet} Sepolia 部署
  → Session Key 生成
  → 第一次完整工作流
  → 最低标准：1 笔 Etherscan 成功交易

Day 2（6/24 周二）：Checkpoint 矩阵补齐
  → Policy Engine 8 边界测试
  → 意图推理日志
  → 最低标准：5 个 P0 Checkpoint 有截图

Day 3（6/25 周三）：Fallback 冻结
  → 最终确认 Z.AI / Cobo 状态
  → Telegram Bot 通知测试
  → 完整工作流连续 3 次成功
  → 最低标准：无任务因外部依赖阻塞

Day 4（6/26 周四）：Demo 材料
  → README 更新
  → Demo 录屏（60 秒完整闭环）
  → SpendingLimitModule 拦截录屏
  → 全部截图整理到 docs/demo/screenshots/

Day 5（6/27 周五）：最终提交
  → 黑客松平台提交
  → GitHub 最终 Push
  → 向 Sponsor 发送项目链接
```

---

## 十、下一步计划

| 时间 | 方向 | 计划 |
|------|------|------|
| **Demo Day 后（7 月）** | Z.AI 接入 | API Key 到达后切换，展示真实多模态 AI 感知 |
| | Cobo CAW 链上 | Access 批准后部署链上 Warrant，替换本地 JSON |
| | 硬件实现 | 树莓派 + 麦克风阵列，实现真实猫咪行为采集 |
| | 产品化 | 宠物食品供应商合作，接入真实商品目录 |

---

## 十一、希望获得的反馈

### Mentor 反馈

1. **产品定位**：评委可能质疑"宠物自主消费"是伪需求。是否有更好的切入点（如：老年人/儿童辅助决策，而非直接让宠物做主）？

2. **安全假设**：安全模型基于"Agent 不持有主私钥"。但如果 Agent 被 prompt injection 攻击，攻击者能否通过反复触发猫咪意图来耗尽每日预算？

3. **商业模式**：MVP 阶段只验证技术可行性（<10 USDC/天）。长期怎么赚钱？

### Sponsor 反馈

**Z.AI**
- API Key 申请流程和预期时间线？
- 是否有测试额度（test credit）可用？

**Cobo**
- SpendingLimitModule（日限额 10 USDC，单笔 10 USDC）的正确配置 JSON 格式？
- 如何在不重新部署 Safe 的前提下更新限额？
- Warrant 撤销后 Session Key 立即失效，还是有冷却期？

### 同伴反馈

- Demo 展示形式：视频 vs. 实时演示 vs. 截图，哪个更有说服力？
- Fallback 策略：外部 API 未接入时，纯 Mock 的 Demo 可以接受吗？
- Sprint 4 时间管理：Day 1 跑通全链路 vs. 提前做好所有 Mock，倾向于哪种策略？

---

## 十二、核心文件索引

| 文件 | 说明 |
|------|------|
| `hackathon/02-proposal-memo.md` | 项目提案（问题/解决方案/商业模式） |
| `hackathon/08-architecture-minimal-loop.md` | 最小闭环架构图 |
| `hackathon/11-cobo-agentic-wallet-design.md` | Cobo CAW 钱包设计（Budget/Scope/Time 三重校验） |
| `hackathon/12-zai-long-horizon-task-design.md` | Z.AI 任务设计（ReAct Loop + 工具调用链） |
| `hackathon/17-week4-mvp-verification.md` | MVP 验证看板（M-01~M-08 Must-have 清单） |
| `hackathon/18-demo-day-retrospective.md` | 项目复盘（问题/主流程/风险/计划/反馈） |
| `README.md` | 项目主页（架构/技术栈/风险/验证计划） |

---

*Cat-Agent（喵星智能体）— AI × Web3 School Cohort 0 黑客松作品*
*MIT License — 欢迎 Star 和 Fork*