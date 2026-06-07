# Week 4 关键技术验证清单

> 基于 Cat-Agent 项目 Sprint 3 末尾阶段，面向黑客松提交的验证记录
> 更新日期：2026-06-08

---

## 验证总览

Week 4 是 Sprint 4（产品+演示冲刺）的核心验证周，目标是确认 Cat-Agent 全链路技术可行性。所有验证结果需截图/录屏存档，作为黑客松提交的技术证明材料。

---

## 1. Agent Trace（多模态 AI 意图链路追踪）

### 验证目标
确认 Cat-Agent 的"感知 → 意图 → 确认"链路可追溯、可复盘。

### 验证点

| ID | 验证点 | 具体内容 | 通过标准 |
|----|--------|---------|---------|
| T-01 | 音频输入追溯 | 猫咪叫声频率/时长波形图存档 | 每帧原始音频带时间戳存储 |
| T-02 | 视觉输入追溯 | 摄像头截图 + 检测框存档 | 每轮交互的猫咪姿态图可查 |
| T-03 | 意图推理记录 | LLM 调用输入/输出完整记录（含置信度分数） | trace.json 每条记录含 audio_features + visual_features + intent + confidence_score |
| T-04 | 两轮交互状态机 | Round 1 → Physical Feedback → Round 2 → Decision 状态转换 | 状态机流转图可导出 |
| T-05 | 拒绝路径记录 | 置信度 <90% 时的拒绝理由存档 | action: decline 记录含 reason 字段 |

### 验证方式
触发 3 次不同场景（要零食 / 要玩 / 拒绝），检查 trace 日志完整性。

### 产出
- `logs/intent/trace_YYYYMMDD_HHMMSS.json` — 每轮交互完整记录
- `logs/intent/screenshots/` — 对应截图存档

---

## 2. SDK 调用（第三方服务集成）

### 验证目标
确认 Z.AI、Cobo CAW、Safe Wallet SDK 调用正常、错误可处理。

### 验证点

| ID | 验证点 | 调用链 | 通过标准 |
|----|--------|--------|---------|
| S-01 | Z.AI 多模态推理 | audio_analyzer.py → Z.AI API → 意图+置信度 | 响应延迟 <2s，P99 <3s |
| S-02 | Z.AI API 错误处理 | 网络超时 / API 限流时的降级逻辑 | 触发降级时日志记录 + 用户通知 |
| S-03 | Cobo CAW Warrant 读取 | caaw_warrant.py → 链上 Warrant → Policy 检查结果 | Warrant 数据解析正确，链上数据与本地配置一致 |
| S-04 | Safe Wallet Session Key | safe_wallet.py → Session Key 签名 → 交易哈希 | 签名成功且 Etherscan 可验证 |
| S-05 | x402 支付 Client | 401 检测 → 支付 → 收据携带重试 | 全流程自动完成，receipt tx_hash 非空 |
| S-06 | Z.AI API 响应格式 | 验证 intent + confidence_score JSON 字段解析正确 | 无字段缺失，无类型错误 |

### 验证方式
Mock 测试 + 实际 API 调用对照，验证响应格式解析正确。

### 产出
- `logs/sdk/s-01_zai_response.json` — Z.AI API 响应样例
- `logs/sdk/s-03_caaw_warrant.json` — CAW Warrant 链上数据解析结果

---

## 3. 测试网交易（链上交易验证）

### 验证目标
确认每笔宠物消费交易在测试网可查、可验证。

### 验证点

| ID | 验证点 | 测试网络 | 通过标准 |
|----|--------|---------|---------|
| N-01 | USDC 转账交易 | Sepolia | Etherscan 显示 from=Safe地址 to=商户地址 value=正确金额 |
| N-02 | 交易哈希唯一性 | 每笔交易 | 无哈希重复（防重放验证） |
| N-03 | 交易失败处理 | 余额不足 / 超出限额 | 链上回退 + 本地错误日志 |
| N-04 | 交易批量查询 | 过去 10 笔宠物消费 | 所有记录可按时间排序查询 |
| N-05 | Gas 费用记录 | 每笔交易 | 记录 gas_used 和 gas_price，计算美元价值 |

### 验证方式
在 Etherscan Sepolia 浏览器逐笔核对，截图存档。

### 产出
- `docs/testnet-tx-log.md` — 所有测试网交易记录 + Etherscan 截图链接
- `logs/blockchain/tx_hashes.json` — 交易哈希汇总

---

## 4. 合约交互（Smart Contract）

### 验证目标
确认 CAW Warrant 链上配置正确，SpendingLimitModule 强制执行。

### 验证点

| ID | 验证点 | 合约 | 通过标准 |
|----|--------|------|---------|
| C-01 | CAW Warrant 注册 | 链上 Warrant Contract | warrantId + budget + scope + validUntil 在链上可读 |
| C-02 | Warrant 签名验证 | 主人主私钥签名 | 合约验签通过，issuer 地址正确 |
| C-03 | SpendingLimitModule 硬拦截 | Safe Module | 单笔 12 USDC 转账被链上回退（>10 USDC 硬限制） |
| C-04 | AllowlistGuard 白名单拦截 | Safe Module | 调用非白名单合约地址被拦截，事件日志记录 |
| C-05 | Warrant 撤销 | 主人调用 revoke() | 撤销后 Agent 无法继续执行支付，链上确认 |
| C-06 | 时间窗口过期 | validUntil 到期后 | 交易被合约拒绝，错误信息含 warrant_expired |

### 验证方式
调用合约方法 + 测试各种边界条件，截图合约返回结果。

### 产出
- `docs/contracts/caaw-warrant-address.txt` — Warrant 合约地址
- `docs/contracts/test-results.md` — 6 个合约测试用例结果

---

## 5. 权限控制（Permission Control）

### 验证目标
确认 CAW Policy Engine 的 Budget / Scope / Time Window 三重检查生效。

### 验证点

| ID | 验证点 | 测试场景 | 通过标准 |
|----|--------|---------|---------|
| P-01 | 日预算拦截 | 当日已消费 45 USDC，再次触发 8 USDC | 引擎返回 budget_exceeded，交易不发送 |
| P-02 | 单笔上限拦截 | 尝试单笔 15 USDC（>10 USDC 限制） | Safe 链上回退 + Policy 层拒绝日志 |
| P-03 | 白名单拦截 | 尝试调用 0xATTACKER（不在白名单） | scope_violation 错误，AllowlistGuard 拦截 |
| P-04 | 时间窗口外拦截 | Warrant 过期后尝试执行 | warrant_expired 错误，交易不广播 |
| P-05 | 自动执行阈值 | 2 USDC 支付（≤3 USDC） | 无需人工确认，自动执行，5 秒内完成 |
| P-06 | 需确认阈值 | 5 USDC 支付（>3 USDC） | 弹出确认 UI，主人确认后才执行 |
| P-07 | 冷却期拦截 | 购买后 23h 内再次购买（<24h 冷却） | cooldown_active 错误，拒绝交易 |
| P-08 | 健康红线拦截 | 连续 3 天只买零食，第 4 次非主粮请求 | health_redline 错误，健康策略强制执行 |

### 验证方式
构造 8 个边界测试用例，100% 被正确拦截/放行。

### 产出
- `logs/policy/p-01_to_p-08_results.json` — 8 个测试用例结果
- `logs/alerts/` — 所有拦截事件的告警日志

---

## 6. 日志记录（Audit Logging）

### 验证目标
确认所有关键操作有完整审计日志，可追溯、可验证。

### 验证点

| ID | 验证点 | 存储位置 | 通过标准 |
|----|--------|---------|---------|
| A-01 | 意图推理日志 | logs/intent/ + IPFS | 每轮交互的 intent_confidence_{timestamp}.json 上传 IPFS |
| A-02 | 支付交易日志 | logs/payments/ + 链上 Event | 每笔支付的 tx_hash + amount + recipient + timestamp |
| A-03 | 审计事件上链 | ERC-8183 Escrow Event 或自定义 Event | 事件日志含 event_type + ipfs_cid，Etherscan 可读 |
| A-04 | RLPF 反馈日志 | logs/rlpf/ | 宠物进食行为记录含 item_id + feedback + weight_update |
| A-05 | 异常告警日志 | logs/alerts/ + Telegram 推送 | 预算超 80% / 异常频率 / 拦截事件 3 分钟内告警 |
| A-06 | 日志防篡改 | IPFS | 日志哈希在链上 Event 中锚定，下载后可验证完整性 |

### 验证方式
触发一次完整购买流程，检查所有日志文件生成 + IPFS 上传 + 链上 Event。

### 产出
- `logs/audit/audit_YYYYMMDD.json` — 完整审计日志
- `docs/ipfs-cids.md` — IPFS CID 汇总 + 验证链接

---

## 7. Demo 证据（截图 / 录屏）

### 验证目标
为黑客松提交准备可截图展示的技术证明材料。

### 证据清单

| ID | 证据类型 | 内容 | 存放位置 |
|----|---------|------|---------|
| D-01 | 意图识别截图 | 猫咪照片 + AI 推理结果面板（意图+置信度+两轮状态） | docs/demo/screenshots/intent-detection.png |
| D-02 | CAW Warrant 配置截图 | Safe App 中 Warrant 可视化配置界面 | docs/demo/screenshots/caaw-config.png |
| D-03 | 链上交易截图 | Etherscan Sepolia 交易详情页（含 tx hash） | docs/demo/screenshots/testnet-tx.png |
| D-04 | 支付确认截图 | 手机 App 弹出支付确认 + 交易结果 | docs/demo/screenshots/payment-confirm.png |
| D-05 | 审计日志截图 | IPFS 存储页面 + 链上 Event 日志 | docs/demo/screenshots/audit-log.png |
| D-06 | 全流程录屏 | 60 秒内展示完整购买闭环（猫咪需求 → AI 确认 → 链上支付 → 交付） | demo/cat-agent-demo.mp4 |
| D-07 | README 封面图 | 项目架构图 + 核心价值主张 | docs/architecture.png |
| D-08 | Sprint 回顾截图 | GitHub Projects 或 Notion 看板（展示 4 个 Sprint 进度） | docs/demo/screenshots/sprint-board.png |

### 验证方式
所有截图上传到 GitHub 仓库 docs/demo/ 目录，可直接用于黑客松提交页面。

---

## Checkpoint 总览

### P0 — 必须通过

- [ ] T-01~T-05：Agent Trace 完整性，每轮交互有完整输入/输出记录
- [ ] N-01：测试网 USDC 转账成功，Etherscan 可查
- [ ] C-01：CAW Warrant 链上注册成功，权限三元组（B/S/T）可读
- [ ] C-03：SpendingLimitModule 硬拦截（单笔 >10 USDC）验证通过
- [ ] D-06：Demo 录屏完成上传（YouTube/Vimeo 链接在 README）

### P1 — 重要

- [ ] S-01~S-05：所有 SDK 调用正常，x402 支付全流程自动完成
- [ ] A-01~A-06：审计日志完整，IPFS + 链上 Event 双重锚定
- [ ] P-01~P-08：8 个权限控制测试用例 100% 通过
- [ ] C-05：Warrant 撤销后 Agent 无法执行支付验证通过
- [ ] P-05：80% 预算告警触发（模拟或实际）
- [ ] D-01~D-08：所有 Demo 证据截图完成存档

### P2 — 加分

- [ ] 多宠物场景（2只猫）意图区分验证
- [ ] 跨链资产（ETH + USDC）混合支付验证
- [ ] RLPF 权重更新后推荐结果变化验证
- [ ] 演示视频添加字幕（中文 + 英文）

---

## 验证时间安排

| 日期 | 验证重点 | 预计时长 |
|------|---------|---------|
| 6/23（周一） | Agent Trace（T-01~T-05）+ SDK 调用（S-01~S-06） | 3h |
| 6/24（周二） | 合约交互（C-01~C-06）+ 权限控制（P-01~P-08） | 3h |
| 6/25（周三） | 测试网交易（N-01~N-05）+ 日志审计（A-01~A-06） | 2h |
| 6/26（周四） | Demo 截图（D-01~D-08）+ 录屏（D-06） | 2h |
| 6/27（周五） | 查漏补缺 + 提交黑客松 | 2h |

> 注：端午节假期（6/22-6/24）期间，Sprint 4 照常进行，可远程验证。
