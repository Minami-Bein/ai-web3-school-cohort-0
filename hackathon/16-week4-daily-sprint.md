# Week 4 每日冲刺计划

> 2026-06-23（周一）~ 2026-06-27（周五，黑客松截止日）
> Project: Cat-Agent
> 目标：黑客松提交，核心工作流可演示可截图

---

## 冲刺总原则

```
Day 1-2（周一二）：基础搭建日 —— 跑通全链路
Day 3（周三）：   补缺日 —— 补齐 Checkpoint 矩阵
Day 4（周四）：   证据日 —— 录屏截图，准备 Demo 材料
Day 5（周五）：   提交日 —— 最终提交 + 备用材料
```

---

## Day 1（6/23 周一）：全链路跑通

### 目标
Safe{Wallet} + 意图推理 + Policy Engine + 交易广播，完整链路跑通至少 1 次。

### 任务清单

| # | 任务 | 类型 | 验收标准 |
|---|------|------|---------|
| 1 | 确认 Z.AI API Key 状态，决定用 Z.AI / OpenAI / Mock | 决策 | 有明确结论并记录到 .env |
| 2 | 确认 Cobo CAW Access 状态，决定用链上 / 本地 JSON | 决策 | 有明确结论并记录到 configs/ |
| 3 | Safe{Wallet} Sepolia 部署完成，Etherscan 可查 | 真实实现 | Etherscan 截图，Safe 地址在 README |
| 4 | SpendingLimitModule（日限额 10 USDC）安装完成 | 真实实现 | 发送 11 USDC 被拦截的 Etherscan 截图 |
| 5 | Session Key 生成并配置 | 真实实现 | Session Key 地址在 configs/ |
| 6 | Cat-Agent 核心代码上传 GitHub | 真实实现 | src/ 目录结构清晰，README 可读 |
| 7 | 第一次完整工作流（意图→检查→支付→确认）| 真实实现 | tx hash 可在 Etherscan 查询 |

### 当日完成标识
- [ ] Etherscan 有至少 1 笔成功交易记录
- [ ] GitHub 有可运行的代码仓库
- [ ] README 包含架构图和运行说明

---

## Day 2（6/24 周二）：Checkpoint 矩阵补齐

### 目标
补齐 Week 2 验证计划（`07-week4-verification.md`）中的所有 Checkpoint，留下可截图的证据。

### 任务清单

| # | 任务 | 类型 | 验收标准 |
|---|------|------|---------|
| 1 | T-01~T-05（Agent Trace 验证）| 真实/Mock | 截图展示每步推理记录 |
| 2 | S-01~S-06（SDK 调用验证）| 真实/Mock | 代码注释 + 实际调用输出 |
| 3 | N-01~N-05（测试网交易验证）| 真实实现 | 每笔交易 Etherscan 截图 |
| 4 | C-01~C-06（合约交互验证）| 真实/Mock | SpendingLimitModule 拦截场景截图 |
| 5 | P-01~P-08（权限控制验证）| 真实/Mock | 8 个拦截场景有截图或日志 |
| 6 | A-01~A-06（日志审计验证）| Mock | 本地 JSON 日志文件，非 IPFS |
| 7 | D-01~D-08（Demo 证据）| 真实/Mock | 每个 Checkpoint 有对应截图 |

### 优先顺序
```
真实 > Mock：
  链上交易（SpendingLimit 拦截）→ 必须真实
  意图推理（Z.AI/OpenAI/Mock） → 取决于 API Key
  合约调用（Safe API）          → 必须真实
  审计日志（IPFS/本地）        → 本地可接受
  Demo 截图                    → 真实优先，录屏备选
```

### 当日完成标识
- [ ] 至少 5 个 P0 Checkpoint 有 Etherscan 截图
- [ ] Checkpoint 矩阵完成度 >= 70%

---

## Day 3（6/25 周三）：Mock / Fallback 切换确认

### 目标
根据 Day 1-2 的实际情况，完成所有 Mock 和 Fallback 的最终切换，确保没有"卡在外部依赖"的任务。

### 任务清单

| # | 任务 | 决策依据 | 完成标准 |
|---|------|---------|---------|
| 1 | 最终确认 Z.AI 状态 | Day 1 决策结论 | 如果 API Key 到达则切换回 Z.AI，否则保持 OpenAI/Mock |
| 2 | 最终确认 Cobo CAW 状态 | Day 1 决策结论 | 如果 Access 到达则用链上 Warrant，否则本地 JSON |
| 3 | 编写 Fallback 代码（如有切换）| 上述决策 | 代码中 USE_ZAI / USE_CAW 等 flag 已配置 |
| 4 | 端到端 Mock 测试（如果所有外部 API 都未到达）| 全部 Mock | 纯本地完整工作流可运行，结果符合预期 |
| 5 | Telegram Bot 通知测试 | 真实实现 | Alice 在手机上收到测试消息 |
| 6 | Mock 商品目录配置 | Mock | configs/products.json 配置完整 |
| 7 | 完整工作流再跑 3 次（稳定性验证）| 真实/Mock | 3 次全部成功，记录 tx hash |

### 当日完成标识
- [ ] 没有任务因为"等外部 API"而阻塞
- [ ] Telegram 通知 Alice 能收到
- [ ] 完整工作流连续 3 次成功

---

## Day 4（6/26 周四）：Demo 材料准备

### 目标
准备好黑客松提交的所有材料：代码、截图、录屏、README、architecture 图。

### 任务清单

| # | 任务 | 类型 | 验收标准 |
|---|------|------|---------|
| 1 | README 更新（架构图 + 运行说明 + Checkpoint 截图）| 真实 | GitHub README 评委可直接阅读 |
| 2 | 架构图（最小闭环）上传 GitHub | 真实 | 08-architecture-minimal-loop.md 已有，可补充截图 |
| 3 | 核心 Demo 录屏（60 秒内）| 真实 | 展示：猫叫 → 意图推理 → 链上交易 → Telegram 通知 |
| 4 | SpendingLimitModule 拦截录屏 | 真实 | 展示发送 11 USDC 被拒绝 |
| 5 | 全部 Etherscan 截图整理到 screenshots/ | 真实 | 每张截图命名规范（日期-功能.png）|
| 6 | Project Proposal 更新（目标用户/场景/功能/验证/风险/赛道）| 真实 | 黑客松需要 1 页 PDF 或 README |
| 7 | Sponsor Integration Plan 确认（已上传）| 真实 | 10-sponsor-sdk-integration-plan.md |
| 8 | Scope Boundary 确认（已上传）| 真实 | 15-scope-boundary.md |
| 9 | 提交前最后一次代码检查 | 真实 | 无明显错误，README 可运行 |

### 当日完成标识
- [ ] GitHub 所有文件已上传，结构清晰
- [ ] 有至少 1 个 Demo 录屏（哪怕是 Mock 模式）
- [ ] README 评委可直接理解项目做什么

---

## Day 5（6/27 周五）：最终提交

### 目标
完成黑客松平台最终提交，所有备用材料本地留存。

### 任务清单

| # | 任务 | 类型 | 验收标准 |
|---|------|------|---------|
| 1 | 黑客松平台提交 | 必须 | 提交成功，收到确认 |
| 2 | GitHub 最终 Push | 必须 | 所有文件已上传 |
| 3 | 备用录屏保存到本地 | 建议 | 以防平台音视频格式问题 |
| 4 | README 确认 README 链接可用 | 必须 | 无断链，无 404 |
| 5 | 向 Sponsor 发送项目链接 | 建议 | 让他们知道你的进展 |
| 6 | （可选）发 LinkedIn / X 公开项目 | 可选 | 有利于个人品牌 |

### 提交 Checklist

```
必填项：
  [ ] GitHub 仓库链接已提交到黑客松平台
  [ ] README 包含：项目简介、架构图、运行步骤、Checkpoint 截图
  [ ] 至少 1 个 Etherscan tx hash 可查
  [ ] 至少 1 个 Demo 截图/录屏

加分项（如有时间）：
  [ ] Sponsor Integration Plan
  [ ] Scope Boundary 说明
  [ ] Risk Analysis 和 Fallback Plan
  [ ] 向 Sponsor / Mentor 请教的问题清单
```

---

## 每日 Mock/Fallback 状态看板

```
状态看板（每天更新）

  Z.AI 多模态推理：
    Day 1：待定（根据 API Key 状态）
    Day 2：按 Day 1 决策执行
    Day 3：如未接入，切换 OpenAI/Mock
    Day 4：最终确认，冻结配置
    Day 5：维持不变

  Cobo CAW：
    Day 1：待定（根据 Access 状态）
    Day 2：按 Day 1 决策执行
    Day 3：如未获批，使用本地 JSON
    Day 4：Policy Engine 逻辑不变，仅数据源切换
    Day 5：维持不变

  外卖平台：
    全周：Mock（configs/products.json）
    不变

  IPFS：
    全周：本地 JSON（日志存 logs/）
    不变
```

---

## 一句话总结

| 日期 | 核心任务 | 最低完成标准 |
|------|---------|------------|
| 6/23（一）| 全链路跑通 | 1 笔成功交易，Etherscan 可查 |
| 6/24（二）| Checkpoint 矩阵补齐 | 5 个 P0 Checkpoint 有截图 |
| 6/25（三）| Fallback 切换确认 | 无任务因外部依赖阻塞 |
| 6/26（四）| Demo 材料准备 | 录屏 + 截图 + README |
| 6/27（五）| 最终提交 | 黑客松平台提交成功 |

**Day 1 的成功 = Day 5 的自由。Day 1 跑通全链路，后面 4 天都是补证据和做材料。**