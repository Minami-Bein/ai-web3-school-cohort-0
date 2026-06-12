# Cat-Agent 项目复盘 — AI × Web3 School Cohort 0

> Demo Day / 最终复盘文档
> 更新日期：2026-06-12
> 项目阶段：Sprint 4 启动前，Week 4 冲刺倒计时

---

## 一、项目解决的问题

**核心问题：宠物有消费需求，但没有任何数字身份和支付能力。**

现有智能喂食器只能"定时定量"，无法感知宠物的即时情绪（饿了/想玩/寂寞）。宠物主人必须全程人工介入：接收信号 → 打开 App → 搜索商品 → 确认支付。

**我们解决的三个具体问题：**

**问题 1 — 意图感知缺失**
猫咪通过叫声/肢体语言表达需求，但现有系统无法理解这些信号。
→ 解决方案：多模态 AI（音频频率分析 + 视觉姿态识别）实时推理宠物意图，置信度必须 >90% 才触发购买，连续两轮交互确认真实需求。

**问题 2 — 消费边界缺失**
没有机制限制宠物（或熊孩子）乱买的上限。
→ 解决方案：CAW Warrant（链上授权）定义 Budget / Scope / Time Window 三重边界，Safe{Wallet} SpendingLimitModule 链上硬拦截，日累计 10 USDC 硬上限。

**问题 3 — 数字身份缺失**
宠物在数字世界中没有独立账户，无法拥有自己的钱包和消费记录。
→ 解决方案：每只宠物拥有独立的 Safe{Wallet} + Session Key，Agent 在 Warrant 授权范围内代理执行支付，每笔交易带审计日志。

---

## 二、已跑通的主流程（Minimal Viable Loop）

```
[猫咪行为输入]
  摄像头 + 麦克风 采集音频 + 图像
        ↓
[意图推理引擎]
  规则引擎（本地，0 外部依赖）
  输出：intent + confidence_score
  阈值：≥ 90% 才继续，否则拒绝 + 通知主人
        ↓
[CAW Policy Engine 三重校验]
  Budget（日限额 10 USDC）
  Scope（白名单商品）
  Time Window（Warrant 有效期）
        ↓
[Safe{Wallet} 执行]
  Session Key 签名
  USDC Transfer → Sepolia 测试网
        ↓
[链上记录 + 通知]
  Etherscan 可查 tx_hash
  Telegram 推送主人（含金额 + 商品名）
        ↓
[审计日志存档]
  本地 JSON（logs/），IPFS-ready
```

**MVP 成功标准：一笔完整交易，Etherscan 可查 + Telegram 通知到账。**

---

## 三、最重要的验证材料

### P0 必查材料（黑客松评委必看）

| # | 材料 | 位置 | 说明 |
|---|------|------|------|
| 1 | Etherscan 成功交易截图 | `docs/demo/screenshots/testnet-tx.png` | tx_hash 可查，日限额 10 USDC 内任意金额 |
| 2 | SpendingLimitModule 拦截截图 | `docs/demo/screenshots/limit-intercept.png` | 发送 11 USDC 被链上拒绝 |
| 3 | 意图推理日志样例 | `logs/intent/trace_*.json` | 含 intent + confidence_score + timestamp |
| 4 | README + 架构图 | `README.md` | 评委直接可读，60 秒理解项目 |
| 5 | Demo 录屏 | `demo/cat-agent-demo.mp4` | 完整闭环：猫叫 → 推理 → 支付 → 通知 |

### P1 重要材料

| # | 材料 | 说明 |
|---|------|------|
| 6 | Policy Engine 8 边界测试 | `logs/policy/p-*-results.json`，Budget/Scope/Time 全拦截 |
| 7 | Telegram 通知截图 | 主人收到含 tx_hash 的支付结果消息 |
| 8 | CAW Warrant 本地 JSON | `configs/caaw_templates.json`，逻辑与链上一致 |
| 9 | Sponsor Integration Plan | `10-sponsor-sdk-integration-plan.md` |

### P2 加分材料

| # | 材料 | 说明 |
|---|------|------|
| 10 | Agent Trace 完整链路 | `logs/intent/` 含 audio_features + visual_features |
| 11 | 3 次稳定交易记录 | 3 个不同 tx_hash，验证系统稳定性 |
| 12 | IPFS 审计日志存档 | `docs/ipfs-cids.md`，CID + 验证链接 |

---

## 四、最大风险

### 风险 1（高优先级）：外部 API 依赖未解决

**Z.AI 多模态 API Key 未获批**
- 概率：40%
- 影响：感知层降级为规则引擎，无法展示真实 AI 意图识别能力
- 当前预案：使用 OpenAI GPT-4o + Whisper 替代
- 残余风险：Demo 效果与真实 AI 感知有差距

**Cobo CAW Access 未获批**
- 概率：25%
- 影响：Policy Engine 使用本地 JSON，链上 Warrant 授权无法展示
- 当前预案：Policy Engine 完全与 CAW SDK 解耦，本地 JSON 仍可验证业务逻辑
- 残余风险：评委可能看不到链上 CAW Warrant 配置界面

### 风险 2（中优先级）：意图识别准确率

**猫咪行为数据不稳定，置信度频繁低于 90%**
- 概率：35%（猫咪行为本来就随机）
- 影响：系统频繁返回"intent unknown"，Demo 体验断断续续
- 当前预案：保守阈值 >90%，多帧融合 + 手动触发 fallback
- 残余风险：Demo 需要多次重拍

### 风险 3（低优先级）：Safe{Wallet} 配置错误

**SpendingLimitModule 配置错误**
- 概率：25%
- 影响：所有交易被拦截（配置过严）或全部放行（配置过松）
- 当前预案：先在测试网部署，运行 P-01~P-08 边界测试用例
- 残余风险：需要在 6/23 前完成测试验证

---

## 五、下一步计划

### 立即行动（6/12 ~ 6/22）

| 日期 | 任务 | 目标 |
|------|------|------|
| 6/13 | 联系 Z.AI 确认 API Key 申请状态 | 明确是否能在 6/23 前接入 |
| 6/14 | 联系 Cobo 确认 CAW Access 审批进度 | 明确链上 Warrant 是否可在 Sprint 期间使用 |
| 6/15~6/20 | Safe{Wallet} Sepolia 部署 + SpendingLimitModule 安装 | 6/23 前 Etherscan 有 Safe 地址 |
| 6/20 | 完成 CAW Policy Engine 本地 JSON 实现 | 8 个边界测试用例通过 |
| 6/22 | 完整流程 Mock 测试 | Sprint 开始前全链路可跑 |

### Sprint 4 冲刺（6/23 ~ 6/27）

```
Day 1（6/23）：全链路跑通 → 1 笔 Etherscan 成功交易
Day 2（6/24）：Checkpoint 矩阵补齐 → 5 个 P0 有截图
Day 3（6/25）：Fallback 冻结 → 无任务因外部依赖阻塞
Day 4（6/26）：Demo 材料 → 录屏 + 截图 + README
Day 5（6/27）：最终提交 → 黑客松平台提交成功
```

### Demo Day 后（7 月）

| 方向 | 计划 |
|------|------|
| Z.AI 接入 | API Key 到达后切换，展示真实多模态 AI 感知 |
| Cobo CAW 链上 | Access 批准后部署链上 Warrant，替换本地 JSON |
| 硬件 | 树莓派 + 麦克风阵列，实现真实猫咪行为采集 |
| 产品化 | 宠物食品供应商合作，接入真实商品目录 |

---

## 六、希望获得的反馈

### Mentor 反馈（最希望回答的问题）

**1. 产品定位**
"宠物自主消费"这个场景，评委可能会质疑是伪需求。你怎么看待这个风险？有没有更好的切入点（如：老年人/儿童辅助决策，而非直接让宠物做主）？

**2. 安全假设**
我们的安全模型基于"Agent 不持有主私钥，只有 Session Key + Warrant 授权"。但如果 Agent 被攻破，攻击者能否通过反复触发猫咪意图来耗尽每日预算？

**3. 商业模式**
MVP 阶段只验证技术可行性（< 10 USDC/天）。但评委通常关心：长期怎么赚钱？你怎么想？

### Sponsor 反馈（技术接入相关）

**Z.AI**
- API Key 申请流程和预期时间线？
- 是否有测试额度（test credit）可用？
- 多模态推理的 P99 延迟是多少？

**Cobo**
- CAW SDK Python 版文档在哪里？
- SpendingLimitModule + AllowlistGuard 的配置 JSON 格式？
- Warrant 撤销后 Session Key 立即失效，还是有冷却期？

**Safe**
- SpendingLimitModule（日限额 10 USDC，单笔 10 USDC）的正确配置方式？
- 如何在不重新部署 Safe 的前提下更新限额？

### 同伴反馈

- **Cat-Agent 的 Demo 演示方式**：你会在 Demo Day 用什么形式展示？视频？实时演示？截图？
- **Fallback 策略**：当外部 API 未接入时，你认为纯 Mock 的 Demo 可以接受吗？还是有更好的展示方式？
- **Sprint 4 时间管理**：Day 1 跑通全链路 vs. 提前做好所有 Mock，你倾向于哪种策略？

---

## 七、项目当前状态总览

```
[✅ 已完成]
  项目方向确定（Cat-Agent，AI × Web3）
  团队搭建（Minami-Bein solo）
  GitHub 仓库初始化（16 个 hackathon 文档）
  Safe{Wallet} Sepolia 测试网计划（待部署）
  CAW Policy Engine 设计（本地 JSON，待实现）
  意图推理规则引擎（本地 Mock，待接入 Z.AI）
  Telegram Bot 通知（待测试）
  MVP Sprint Plan（6/23 ~ 6/27）

[⏳ 进行中]
  Safe{Wallet} 部署（SpendingLimitModule 待安装）
  Session Key 生成
  Policy Engine 8 边界测试

[🔲 待启动]
  Z.AI API Key（等待审批）
  Cobo CAW Access（等待审批）
  完整支付流程 Mock 测试
  Demo 录屏

[❌ 已砍除 / 延后]
  Z.AI 真实 API（等待 Key）
  Cobo CAW 链上 SDK（等待 Access）
  x402 支付 Client（外部依赖，Mock 替代）
  多宠物意图区分
  IPFS 审计日志上链
  RLPF 机器学习权重更新
```

---

## 八、文件索引

| 文件 | 说明 |
|------|------|
| `01-direction-card.md` | 项目方向卡 |
| `02-proposal-memo.md` | 项目提案备忘录 |
| `04-sprint-plan.md` | 4 个 Sprint 的完整计划 |
| `05-risk-memo.md` | 风险分析备忘录 |
| `07-week4-verification.md` | Week 4 详细验证清单（40+ 验证点） |
| `08-architecture-minimal-loop.md` | 最小闭环架构图 |
| `10-sponsor-sdk-integration-plan.md` | Sponsor SDK 接入计划 |
| `11-cobo-agentic-wallet-design.md` | Cobo Agentic Wallet 设计文档 |
| `13-project-prerequisites-risk-fallback.md` | 前置条件 + 风险 + Fallback 决策树 |
| `14-questions-for-sponsors-mentors.md` | 向 Sponsor / Mentor 请教的问题清单 |
| `15-scope-boundary.md` | Scope 边界定义（砍掉哪些） |
| `16-week4-daily-sprint.md` | Week 4 每日冲刺计划 |
| `17-week4-mvp-verification.md` | MVP 验证看板（M-01~M-08 + S/N/X 任务） |

---

*本项目为 Minami-Bein 原创，AI × Web3 School Cohort 0 黑客松作品。*
*MIT License — 欢迎 Star 和 Fork。*
