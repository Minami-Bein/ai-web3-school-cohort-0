# 阅读摘要：Cat-Agent 相关标准 / 协议 / SDK

> 更新日期：2026-06-08
> 围绕主方向：AI Agent 自主支付（宠物消费场景）

---

## 摘要 1：x402 — 机器支付协议

### 解决什么问题

x402 是"可编程支付"协议，解决机器与机器之间的 HTTP API 调用如何自动完成支付的问题。

传统 HTTP 认证是"你是谁"（API Key / OAuth），x402 的核心创新是"你愿意付多少"——它把支付能力嵌入 HTTP 协议层（`WWW-Authenticate` 响应头），让支付成为 API 调用本身的自然组成部分，而不是跳到外部支付页面。

典型场景：
- AI Agent 需要调用付费 LLM 推理 API
- IoT 设备需要按调用量支付云服务
- Cat-Agent 的 AI 推理层需要为每次猫咪意图分析付费

### 边界是什么

**x402 能做的：**
- 单次 HTTP 请求级别的支付扣款（USDC 为主）
- 收据携带重试（`Receipt: <base64>` 响应头，客户端自动凭收据重试原请求）
- 支付方可以是 EOA、Safe、合约钱包
- 收款方无需 KYC（链上结算）

**x402 不能做的（边界）：**
- 不提供身份认证（需要配合 API Key 或其他认证层）
- 不托管资金（收据是一次性的、链上原子结算）
- 收据是链下承诺，如果 Server 故意不交付资源，收据无法强制 Server 履约（需要信任 Server）
- 不支持订阅制（每次独立支付，适合按次计费，不适合包月）
- 支付金额必须提前约定（Server 声明价格，Client 无定价权）

### 还缺什么

1. **收据防伪造验证缺乏链上机制**：收据是 Server 签名的，Client 侧无法独立验证收据真伪——需要 Server 把收据的 tx_hash 锚定到链上才算"不可伪造"
2. **隐私问题**：收据包含 Client 钱包地址，Server 可追踪 Client 行为画像
3. **多币种支持**：当前规范以 USDC 为主，其他 ERC-20 需要额外适配
4. **离线/异步场景**：收据重试机制不适合耗时任务（如 AI 视频生成），需要扩展 `202 Accepted` 模式
5. **Cat-Agent 适配点**：AI 推理层应该用 x402 保护，但收据的 tx_hash 验证需要在 Cat-Agent Server 侧实现，否则 Agent 可能被伪造收据欺骗

---

## 摘要 2：CAW — Custom Agent Wallet（Cobo）

### 解决什么问题

CAW 是 Cobo 提供的"Agent 专用钱包"解决方案，解决 AI Agent 在执行链上操作时如何安全授权、限制权限、防止被盗的问题。

传统 EOA 钱包（私钥控制一切）对 Agent 来说风险极大——私钥一旦泄露，Agent 可以做任何操作。CAW 的核心思路是把 EOA 的"万能钥匙"拆成多个受限的"能力卡片"（Warrant），每个 Warrant 有独立的 Budget、Scope 和 Time Window。

Cat-Agent 的应用：
- 为每只猫部署一个 CAW Warrant
- Warrant 设定：日预算 10 USDC、单笔上限 10 USDC、仅允许调用白名单商户 API
- Agent（猫的 AI 大脑）只持有 Session Key，只能在 Warrant 授权范围内操作

### 边界是什么

**CAW 能做的：**
- Warrant 级别的链上权限控制（不可绕过，因为 SpendingLimitModule 在合约层硬限制）
- 多 Agent 多钱包管理（一个主人 Safe 管辖多个 Agent 钱包）
- 权限的细粒度控制（Budget + Scope + Time 三元组）
- Session Key 临时签名（使用后销毁，减少私钥暴露时间窗口）

**CAW 不能做的（边界）：**
- 不提供 AI 推理能力（需要结合 Z.AI 或 OpenAI）
- Warrant 授权后，Agent 在授权范围内可以自主执行，无法实时中止（需要等待链上确认延迟）
- CAW Warrant 的"条件触发"能力有限，无法表达"只有当 AI 置信度 >90% 时才授权"这种业务层条件——这类逻辑需要在 CAW 上层（Policy Engine）实现
- Session Key 泄露风险仍存在（SpendingLimitModule 可以限制损失上限，但无法防止恶意操作立即发生）
- 跨链支持：主要面向 EVM 链，非 EVM 链需要额外适配

### 还缺什么

1. **条件型 Warrant**：现有 Warrant 是静态的（固定预算/时间），Cat-Agent 需要"动态条件触发"型 Warrant——这需要在上层 Policy Engine 实现，CAW 本身不支持
2. **多 Agent 协调**：两只猫同时有购买意图时，如何协调优先级？CAW 没有多 Agent 调度机制
3. **链上声誉系统**：ERC-8004 链上身份注册与 CAW Warrant 之间的关联还没有标准实践
4. **审计日志标准**：CAW Warrant 的所有使用记录需要统一的链上 Event 标准，当前各家实现不一致
5. **Cat-Agent 适配点**：CAW 是底层钱包基础设施，Cat-Agent 需要在 CAW 之上构建 Policy Engine（业务层权限逻辑）和 RLPF（学习反馈层），这两层 CAW 不提供

---

## 摘要 3：MCP — Model Context Protocol（Anthropic）

### 解决什么问题

MCP 是 Anthropic 提出的 AI Agent 与外部工具/数据源交互的协议标准，解决"AI 模型如何安全、可扩展地调用外部工具"的问题。

在 Cat-Agent 场景中，MCP 的作用是让 AI Agent（猫的意图分析大脑）能够标准地调用：
- 摄像头/麦克风（IoT 设备 MCP Server）
- Z.AI 多模态推理 API（MCP Server）
- 链上数据查询（Alchemy/QuickNode MCP Server）
- CAW Warrant 状态查询（MCP Server）

MCP 解决的是 AI 工具调用的"插拔问题"——不同供应商的工具可以用统一协议接入 Agent，无需为每个工具单独写集成代码。

### 边界是什么

**MCP 能做的：**
- 标准化的工具发现（`tools/list` 方法返回可用工具清单）
- 统一工具调用格式（`tools/call` 方法，参数 JSON 化）
- 工具权限控制（Host 可以控制 Agent 能调用哪些工具）
- 多工具协调（Agent 可以自主决定调用顺序）

**MCP 不能做的（边界）：**
- 不提供身份认证（谁在调用 MCP Server 靠 Host 层解决）
- 不处理支付（MCP 是纯工具调用协议，支付由 x402 等上层协议处理）
- 不保证工具实现的正确性（工具返回假数据时 MCP 本身无法识别）
- 不适合实时流式数据（当前版本是请求-响应模式，不适合音视频流）
- 状态管理依赖 Host（MCP Server 无状态，状态在 Agent Host 中管理）

### 还缺什么

1. **MCP 安全标准**：当前 MCP 没有强制工具权限分级标准，容易出现工具越权（如恶意工具伪装成只读工具实际执行写入操作）
2. **多 Agent 共享 MCP Server**：多个 Agent（如 Cat-Agent + Dog-Agent）共享同一个 MCP Server 时，资源隔离和计费没有标准
3. **MCP 工具的 x402 保护**：如果 MCP Server 需要按次付费（如 Z.AI 推理 API），MCP 协议层不处理支付，需要在工具实现层集成 x402
4. **离线工具发现**：当 MCP Server 不可达时，没有标准的降级机制
5. **Cat-Agent 适配点**：Cat-Agent 的 IoT 设备层（摄像头/麦克风）应该实现为 MCP Server，这样 Z.AI 推理层可以标准调用，切换推理底座时不需要改业务逻辑

---

## 三者关系总结

```
Cat-Agent 应用层
       │
       ├── RLPF（宠物反馈强化学习）  ← 自行开发
       │
       ├── Policy Engine（业务权限逻辑）  ← 自行开发
       │        ▲
       │        │ 调用
       │        │
       ▼        │
   x402 ───────┘  ← 机器支付协议
       │
       ▼
   MCP ────────────  ← 工具调用协议
       │
       ▼
   CAW + Safe{Wallet}  ← 链上钱包基础设施
       │
       ▼
   Ethereum (Sepolia)
```

| 标准/项目 | 角色 | Cat-Agent 需要做什么 |
|-----------|------|-------------------|
| x402 | 机器微支付协议 | 实现 x402 Client，验证收据 tx_hash 链上锚定 |
| CAW + Safe | Agent 钱包 + 权限控制 | 在 CAW 上封装 Policy Engine（业务条件）|
| MCP | AI 工具调用协议 | 实现 IoT MCP Server，Z.AI 作为 MCP 工具接入 |
