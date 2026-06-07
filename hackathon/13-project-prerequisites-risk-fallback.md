# 项目成立前提、失败风险与 Week 4 Fallback Plan

> Updated: 2026-06-08
> Project: Cat-Agent

---

## 一、项目成立依赖的 8 个前提

Cat-Agent 不是一个可以在真空中成立的项目。它的每一个核心模块都依赖特定的外部条件。以下是项目成立的 8 个必要前提，按依赖程度排序：

### P0（必须满足，缺一不可）

**前提 1：Safe{Wallet} 能在 Sepolia 测试网正常部署和运行**
- 依赖内容：Safe 合约部署、Session Key 生成、SpendingLimitModule 安装、USDC 转账
- 验证方式：在 Sepolia 部署一个测试 Safe，完成 USDC 转账，Etherscan 可查
- 若不满足：整个链上交易层无法工作，项目无法演示任何 Web3 能力
- 责任方：Safe{Wallet} 技术文档已完善，可自行完成（预计 2h）

**前提 2：Z.AI 多模态推理 API 能正常调用（或有可替换方案）**
- 依赖内容：摄像头+麦克风 → 意图+置信度的实时推理
- 验证方式：调用 API，返回结构包含 intent / confidence_score / action
- 若不满足：感知层失效，Cat-Agent 无法知道猫咪想要什么
- 风险：Z.AI API Key 申请可能需要 Sponsor 审批（不确定周期）
- 备选：OpenAI GPT-4o + Whisper（费用更高，但文档完整）

**前提 3：宠物主人（Alice）愿意参与测试**
- 依赖内容：真实猫咪在摄像头前产生可识别的行为（叫、靠近食盆等）
- 验证方式：Alice 录一段猫咪叫声 + 视频，测试多模态推理
- 若不满足：只能做模拟测试，无法展示真实用户场景
- 注意：猫咪不一定配合，需要多次测试

### P1（重要，但有替代方案）

**前提 4：Cobo CAW SDK 或等效 Warrant 机制可用**
- 依赖内容：链上 Budget/Scope/Time Window 三重权限校验
- 验证方式：读取 Warrant 配置，执行一次超出预算的交易，被链上模块拦截
- 若不满足：使用本地 JSON 配置替代（Policy Engine 逻辑不变）
- 影响：没有链上 Warrant，权限控制只能依赖 Safe 模块（安全性降低但仍可用）

**前提 5：Sepolia 测试网 USDC 可获取**
- 依赖内容：部署 Safe 和发起交易需要 Sepolia USDC
- 验证方式：钱包里有 >20 USDC 的 Sepolia 测试币
- 若不满足：无法完成链上交易演示
- 获取方式：Sepolia 水龙头（https://faucet.sepolia.dev/）或联系 Sponsor

**前提 6：外卖平台 API（或 Mock）可接入**
- 依赖内容：猫咪下单后，商品能被真正配送（或至少模拟配送流程）
- 验证方式：调用 API 返回 in_stock + delivery_time
- 若不满足：用 Mock API 替代（返回固定商品数据）
- 影响：不影响链上流程，只影响"配送"这一后置步骤

### P2（锦上添花，有则更好）

**前提 7：IPFS 存储服务可用（Pinata / Infura IPFS）**
- 依赖内容：审计日志存档（intent_log + tx_receipt）
- 验证方式：上传一段 JSON，返回可访问的 CID
- 若不满足：审计日志仅存本地（可审计性降低，但不影响核心功能）
- 获取：Pinata 有免费额度，Infura IPFS 需要注册

**前提 8：Telegram Bot 已创建且 Webhook 可达**
- 依赖内容：向 Alice 推送交易通知
- 验证方式：发送测试消息，Alice 在 Telegram 收到
- 若不满足：无法实时通知，但交易仍可在 Etherscan 查询
- 创建：@BotFather 获取 Token，配置 Webhook 或使用轮询

---

## 二、最可能失败的 5 个地方

按概率从高到低排列：

### 失败点 1：Z.AI API Key 申请不下来（P0，概率 40%）

**问题描述：**
Sponsor 的 Z.AI API 需要申请和审批，审批周期不确定。如果 6/20 前拿不到 API Key，整个感知层开发被迫使用 OpenAI Fallback，增加成本且失去"宠物 AI 感知"的独特故事点。

**影响范围：**
- 感知层无法真实运行
- 多模态推理 demo 无法展示
- 项目从"AI 感知宠物"退化为"规则触发器"

**预防措施：**
- 6/18 前主动联系 Sponsor 询问审批进度
- 同时准备 OpenAI Fallback，Z.AI 到达后仅替换 API Key
- 如果 6/22 仍未拿到，果断切换 OpenAI，继续推进其他模块

### 失败点 2：猫咪行为数据不规律，难以触发 AI 推理（P0，概率 35%）

**问题描述：**
猫咪不像人类会主动发出清晰指令。猫咪可能在凌晨 3 点叫、在镜头外叫、或根本不配合。真实场景下多模态推理的置信度可能长期低于 0.90 阈值，导致 Cat-Agent 持续返回"意图不明"。

**影响范围：**
- Demo 可能需要大量人工模拟（猫咪不听指挥）
- 真实场景下触发频率远低于预期

**预防措施：**
- 提前录制多种猫咪行为数据，建立测试数据集
- 降低置信度阈值到 0.80，或增加多帧融合逻辑
- 设计"手动触发"模式作为保底（主人按按钮 = 猫咪想要某物）

### 失败点 3：Safe SpendingLimitModule 配置错误（P1，概率 25%）

**问题描述：**
Safe 模块配置有学习成本。SpendingLimitModule 的日预算和单笔限额需要正确设置，否则可能出现：限额设置过小导致正常交易被拦截，或限额设置过大导致失去保护意义。

**影响范围：**
- 链上权限控制失效（全部交易通过或全部拦截）
- Demo 演示时出现尴尬的技术失误

**预防措施：**
- 在测试网先部署一个独立 Safe，完整测试限额逻辑
- 所有模块配置用 checklist 逐项验证（P-01 ~ P-08）
- 准备截图证据：配置页 + Etherscan 拦截记录

### 失败点 4：Cobo CAW SDK 文档不足或 Access 未授权（P1，概率 25%）

**问题描述：**
Cobo CAW 是较新的产品，SDK 文档可能不完整，审批流程不确定。如果 SDK 无法按期接入，Policy Engine 只能用本地 JSON 模拟 Warrant 数据。

**影响范围：**
- 链上 Warrant 标准演示效果降低（本地 JSON vs 真实链上 Warrant）
- 可能错过与 Sponsor 深度合作的故事点

**预防措施：**
- 6/18 前主动联系 Sponsor 获取 SDK 进度
- Policy Engine 完全与 CAW SDK 解耦（纯接口设计），切换到真实 SDK 无需改业务逻辑
- 如果 6/22 仍未获批，用本地 JSON + Mock Warrant 继续开发

### 失败点 5：测试网 USDC 获取失败（P2，概率 15%）

**问题描述：**
Sepolia 水龙头有频率限制，可能短时间内无法获取足够 USDC。

**影响范围：**
- 链上交易演示数量受限
- 无法测试日预算耗尽场景

**预防措施：**
- 同时准备多个水龙头（Alchemy、QuickNode、Chainlink）
- 联系 Sponsor 是否有专用测试币渠道

---

## 三、Week 4 Fallback Plan（6/23 - 6/27）

Week 4 是黑客松提交周。如果以下模块在 Week 3 结束时仍未完成，按此 Plan 降级：

### 降级原则

```
能跑 > 完美
链上 > 模拟
截图 > 口述
部分功能 > 零功能
```

### Fallback 矩阵

| 模块 | 正常目标 | Fallback 1 | Fallback 2 | Fallback 3 |
|------|---------|-----------|-----------|-----------|
| Z.AI 多模态推理 | 真实 API | OpenAI GPT-4o + Whisper | 规则引擎 + 固定意图 | 预录制视频演示 |
| Cobo CAW | 链上 Warrant | 本地 JSON Warrant | Mock Warrant + 截图 | 文档说明 + 代码注释 |
| Safe{Wallet} | 真实链上交易 | 测试网完整交易 | Hardhat 模拟交易 | 代码 + 截图 |
| 外卖配送 | 真实 API | Mock API（固定返回）| 手动填单截图 | 说明文档 |
| IPFS 存档 | 真实 IPFS CID | 本地文件存档 | 不存档（记录到 DB）| 说明文档 |
| Telegram 通知 | 实时推送 | 手动截图 | Etherscan 链接替代 | 说明文档 |

### 具体 Fallback 场景

**场景 A：Z.AI API Key 6/23 前未到达**
```
立即切换 OpenAI Fallback：
1. 在 .env 中添加 OPENAI_API_KEY
2. 将 src/perception/zai_client.py 标记为 USE_ZAI=false
3. 使用 gpt-4o + whisper-1 组合
4. 继续开发，不等待
5. 如果 Z.AI 后续到达，仅修改 API Key 配置即可
```

**场景 B：Safe 部署遇到技术问题（6/24 前无法解决）**
```
使用 Hardhat 本地模拟：
1. 启动 Hardhat node（npx hardhat node）
2. 使用预编译的 Safe 合约（Hardhat console 中已有）
3. 在 MockSafeWallet 中实现 SpendingLimitModule 逻辑
4. 录制 Hardhat console 输出作为演示证据
5. 在 README 中注明"可在 Sepolia 部署完整版"
```

**场景 C：猫咪完全不配合（无法录制真实 Demo）**
```
使用预录制数据：
1. 提前录制 5 种猫咪行为视频（叫、靠近食盆、慢眨眼等）
2. 每次 demo 播放对应视频作为触发输入
3. 同时保留"实时摄像头"作为可选项
4. 截图 + 录屏作为提交证据
```

**场景 D：链上交易全部失败（Gas 问题 / 网络问题）**
```
切换到"代码 + 文档"模式：
1. 完整代码上传 GitHub（可审核）
2. 测试网交易用 tx hash + Etherscan 截图（不需要实时）
3. 每个 Checkpoint 用代码注释 + 运行结果说明
4. 最终提交："功能已实现，测试网部署视频见 XXX"
```

### 最小可提交版本（Minimum Viable Demo）

只要满足以下条件，项目即可提交：

```
P0（必须）：
  - Cat-Agent 核心代码在 GitHub 可查
  - README 说明架构和实现方式
  - 至少 1 个 Checkpoint 有测试网交易记录（tx hash 可查）

P1（应该有）：
  - Safe{Wallet} 部署截图
  - 意图推理演示（哪怕是 GPT-4o Fallback）
  - 权限拦截演示（哪怕是本地 Mock）

P2（锦上添花）：
  - IPFS 存档 CID
  - Telegram 通知截图
  - 完整的 Checkpoint 验证矩阵
```

---

## 四、风险监控 Checkpoint

| 时间 | 检查项 | 通过标准 | 未通过则 |
|------|-------|---------|---------|
| 6/18 上午 | Safe Sepolia 部署 | Etherscan 可查 Safe 地址 | 使用 Hardhat 模拟 |
| 6/18 下午 | Z.AI API Key 状态 | 收到 Key 或明确审批周期 | 切换 OpenAI Fallback |
| 6/19 上午 | SpendingLimitModule | 发送 11 USDC 被拦截 | 检查 Module 配置 |
| 6/20 上午 | 多模态推理 | Z.AI 或 OpenAI 返回有效结果 | 使用规则引擎 |
| 6/21 上午 | Cobo CAW 状态 | SDK 可用或 Access 已授权 | 使用本地 JSON |
| 6/22 上午 | 全链路测试 | 5 次完整工作流全部成功 | 定位失败点并降级 |
| 6/23 上午 | Demo 准备 | 有可演示的截图/录屏 | 使用预录制数据 |
| 6/24 下午 | 提交材料 | Checkpoint 矩阵完成度 >= 80% | 降级目标至 P0 |

---

## 五、一句话总结

> **Cat-Agent 的最大风险不是技术复杂度，而是外部依赖（Z.AI API Key + Cobo Access）的审批时间。建议 6/18 前主动推进 Sponsor 审批，6/20 前完成 Fallback 切换准备。核心原则：永远有降级方案，永远不让外部依赖阻塞开发进度。**