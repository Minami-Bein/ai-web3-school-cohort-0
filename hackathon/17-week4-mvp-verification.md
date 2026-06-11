# Week 4 MVP 验证看板

> 项目：Cat-Agent（喵星智能体）
> 版本：v1.0 — 最小可验证主流程
> 更新日期：2026-06-11

---

## 一、最小可验证主流程（Minimal Viable Loop）

```
猫咪行为输入（音频/摄像头）
        │
        ▼
意图推理（AI 或规则引擎）
  confidence_score ≥ 90%？ ──否──▶ 拒绝，通知主人
        │是
        ▼
CAW Policy Engine（三重校验）
  Budget（日限额10 USDC）+ Scope（白名单）+ Time Window
        │PASS
        ▼
Safe{Wallet} 执行
  Session Key 签名 ──▶ USDC Transfer ──▶ tx_hash
        │
        ▼
通知主人（Telegram）+ 审计日志存档
```

**MVP 成功标准：一笔完整交易，Etherscan 可查 + Telegram 通知到账。**

---

## 二、任务看板

### Must-have（6/27黑客松提交前必须完成）

| # | 任务 | Owner | Deadline | 验证方式 |
|---|------|-------|----------|---------|
| M-01 | Safe{Wallet} Sepolia 部署 + SpendingLimitModule 安装 | Bein | 6/23 | Etherscan 截图：发送 11 USDC 被拦截 |
| M-02 | Session Key 生成并配置到 Agent | Bein | 6/23 | `configs/session_key.json` 存在且地址非零 |
| M-03 | CAW Policy Engine（B/S/T 三重校验）实现 | Bein | 6/24 | 8 个边界测试用例全部通过（logs/policy/） |
| M-04 | 意图推理（规则引擎或 AI） + 置信度日志 | Bein | 6/24 | `logs/intent/trace_*.json` 每轮有 intent + confidence_score |
| M-05 | 完整支付流程：意图 → Policy → Safe → tx_hash | Bein | 6/25 | Etherscan 有至少 1 笔 USDC 转账记录，tx hash 可查 |
| M-06 | Telegram 通知（Alice 收到支付结果） | Bein | 6/25 | Alice 手机收到含 tx hash 的 Telegram 消息 |
| M-07 | README 更新（架构图 + 运行说明 + 截图） | Bein | 6/26 | GitHub README评委可直接阅读并运行 |
| M-08 | 黑客松平台提交 | Bein | 6/27 | 提交成功，有确认回执 |

### Should-have（有时间再做，没时间砍掉）

| # | 任务 | 验证方式 |
|---|------|---------|
| S-01 | Agent Trace 完整链路日志（T-01~T-05） | `logs/intent/trace_*.json` 含 audio_features + visual_features |
| S-02 | Demo 录屏（60 秒完整闭环） | YouTube/Vimeo 链接写入 README |
| S-03 | Sponsor SDK Integration Plan 上传 | `10-sponsor-sdk-integration-plan.md` 在 GitHub |
| S-04 | Etherscan 多笔交易截图（稳定性验证） | 3 次完整流程，3 个不同 tx hash |
| S-05 | Warrant 本地 JSON Mock 配置完整 | `configs/caaw_templates.json` 可读，逻辑与链上一致 |

### Nice-to-have（加分项，砍掉不影响提交）

| # | 任务 | 验证方式 |
|---|------|---------|
| N-01 | 两轮交互状态机可视化 | 状态流转图导出为 PNG |
| N-02 | 健康红线拦截（连续 3 天只吃零食） | 日志有 health_redline 拒绝记录 |
| N-03 | IPFS 审计日志存档 | `docs/ipfs-cids.md` 有 CID + 验证链接 |
| N-04 | RLPF 权重更新记录 | `logs/rlpf/weights_*.json` |
| N-05 | Demo 截图包（8 张截图规范命名） | `docs/demo/screenshots/` 目录完整 |

### Cut（本期砍掉，MVP 范围外）

| # | 任务 | 砍掉原因 |
|---|------|---------|
| X-01 | Z.AI 多模态推理（真实 API） | 等 API Key，时间不确定 |
| X-02 | Cobo CAW 链上 Warrant SDK | 等 Access 审批，时间不确定 |
| X-03 | x402 支付 Client | 外部依赖，MVP 用 Mock |
| X-04 | 跨链资产混合支付（ETH + USDC） | 超出 MVP 范围 |
| X-05 | 多宠物意图区分（2 只猫） | 单宠 MVP 足够 |
| X-06 | 完整 IPFS 审计日志上链 | 本地 JSON 可接受 |
| X-07 | RLPF 机器学习权重更新 | MVP 期只记录日志，不做更新 |

### Mock / Fallback 状态

| 模块 | 决策 | 说明 |
|------|------|------|
| 意图推理 | **规则引擎** | 本地 `src/perception/rule_engine.py`，0外部依赖 |
| CAW Warrant | **本地 JSON** | `configs/caaw_templates.json`，逻辑与链上一致 |
| 商品/外卖平台 | **Mock** | `configs/products.json`，不动 |
| IPFS 存档 | **本地 JSON** | `logs/` 目录，不上传 IPFS |
| Safe{Wallet} | **真实（Sepolia）** | 必须真实，无 Mock，核心链路 |

---

## 三、Must-have 每日计划

### 6/23（周一）—链上基础搭建
- [ ] M-01：Safe{Wallet} Sepolia 部署 + SpendingLimitModule 安装
- [ ] M-02：Session Key 生成
- 交付：Etherscan 有 Safe 地址 + SpendingLimitModule 安装截图

### 6/24（周二）— Policy Engine + 意图推理
- [ ] M-03：CAW Policy Engine 实现 + 8 个测试用例
- [ ] M-04：规则引擎 + 置信度日志
- 交付：`logs/policy/p-*-results.json`（8/8 通过）+ `logs/intent/trace_*.json`

### 6/25（周三）—完整支付流程
- [ ] M-05：完整流程跑通（意图 → Policy → Safe → tx_hash）
- [ ] M-06：Telegram 通知
- 交付：Etherscan 有 tx hash + Alice收到 Telegram 消息

### 6/26（周四）— 材料准备
- [ ] M-07：README 更新
- [ ] S-02：Demo 录屏（如时间允许）
- 交付：GitHub 所有文件就绪，评委可直接阅读

### 6/27（周五）— 最终提交
- [ ] M-08：黑客松平台提交
- 交付：提交成功确认回执

---

## 四、MVP Checkpoint 总览

```
[P0 必须通过]
  ✅ M-01: Safe{Wallet} 部署 + SpendingLimitModule 硬拦截
  ✅ M-02: Session Key 配置
  ✅ M-03: Policy Engine 8/8 测试通过
  ✅ M-04: 意图推理 + 置信度日志
  ✅ M-05: 完整支付流程 tx_hash（Etherscan 可查）
  ✅ M-06: Telegram 通知
  ✅ M-07: README 更新
  ✅ M-08: 黑客松提交

[P1 Should-have - 尽量完成]
  ⬜ S-01: Agent Trace 完整链路
  ⬜ S-02: Demo 录屏
  ⬜ S-03: Sponsor Integration Plan
  ⬜ S-04: 3 次稳定交易验证
  ⬜ S-05: Warrant 本地 JSON Mock

[P2 Nice-to-have - 有时间加分]
  ⬜ N-01~N-05: 状态机/健康红线/IPFS/RLPF/截图包

[Cut 本期不碰]
  X-01 ~ X-07: Z.AI真实 API / CAW SDK / x402 / 跨链 / 多宠 / IPFS 上链 / RLPF 更新
```

---

## 五、Fallback 决策树

```
意图推理：
  有 Z.AI API Key？ → 用 Z.AI
  无 → 有 OpenAI Key？ → 用 GPT-4o + Whisper
  无 → 用规则引擎（当前决策）

CAW Warrant：
  有 Cobo CAW Access？ → 用链上 Warrant SDK
  无 → 用本地 JSON（configs/caaw_templates.json）

Safe{Wallet}：
  必须用真实 Sepolia 网络，无 Mock

外卖/商品平台：
  全程 Mock（configs/products.json）
```