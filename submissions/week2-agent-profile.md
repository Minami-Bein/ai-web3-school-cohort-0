# Week 2 Module C · Agent Profile 草图

> Agent 设计：TaxReportBot（链上税务报告机器人）
> 基于 Week 1 实验"受限 Web3 助手"和 Week 2 Module B 的任务市场流程
> 输出到 submissions/week2-agent-profile.md

---

## Agent 选择理由

我选择之前在实验文件（`experiments/restricted-web3-assistant-design.md`）中设计过的 **TaxReportBot**——一个帮用户生成链上税务报告的 AI Agent。

选择原因：
- 有完整的工作流（输入 → 分析 → 报告生成 → 用户验收）
- 涉及多种 Web3 交互（读取多链数据、处理交易历史、生成结构化报告）
- 天然需要身份验证（报告代表用户真实财务数据）
- 是 Week 2 Module B 场景（Agent 完成任务并收款）的具体落地案例

---

## Agent Profile 草图

### 基础信息

| 字段 | 内容 |
|------|------|
| **名称** | TaxReportBot |
| **类型** | Personal AI Agent（个人财务助手） |
| **版本** | v0.1（draft） |
| **一句话描述** | 读取用户指定钱包的链上交易历史，自动生成税务报告，并可通过链上 Escrow 收款 |
| **由谁维护** | 个人用户本地运行（self-hosted）+ 可选社区维护的云端版本 |
| **所属组织** | 个人（非企业） |
| **官网/文档** | （待补充） |

---

## Identity（身份）

### 链上身份（ERC-8004 Identity Registry）

```
namespace: eip155:1                    # Ethereum Mainnet
chainId: 1
identityRegistry: 0x742d...            # ERC-8004 Identity Registry 地址
agentId: 17                            # TaxReportBot 的链上 NFT tokenId
agentURI: ipfs://bafybeiag...           # 元数据文件（C见下方）
```

### AgentURI 元数据文件（ipfs://.../agent-metadata.json）

```json
{
  "type": "https://eips.ethereum.org/EIPS/eip-8004#registration-v1",
  "name": "TaxReportBot",
  "description": "链上税务报告生成机器人。读取钱包交易历史，生成ERC-20转账、Gas消耗、收益/亏损报告，支持年度报表和自定义周期。",
  "image": "https://example.com/taxreportbot.png",
  "services": [
    {
      "name": "A2A",
      "endpoint": "https://taxreportbot.example/.well-known/agent-card.json",
      "version": "0.3.0"
    },
    {
      "name": "MCP",
      "endpoint": "https://mcp.taxreportbot.eth/",
      "version": "2025-01-01"
    }
  ],
  "capabilities": [
    "read_onchain_history",
    "generate_tax_report",
    "calculate_gas_spent",
    "calculate_token_gains",
    "export_pdf",
    "export_csv"
  ],
  "pricing": {
    "per_report": "5 USDC",
    "monthly_subscription": "15 USDC",
    "currency": "USDC"
  },
  "trust_model": "reputation_score",
  "reputation_score": 4.7
}
```

### 身份说明

- TaxReportBot 不需要企业认证，个人开发者可将其部署并注册到 ERC-8004 Identity Registry
- `agentId` 是 NFT tokenId，代表 TaxReportBot 在链上的唯一身份
- 身份不可转让（NFT 特性），防止恶意冒充

---

## Capability（能力）

### 核心能力清单

| 能力 | 说明 | 是否需要链上签名 |
|------|------|-----------------|
| `read_onchain_history` | 读取指定钱包在指定时间范围内的所有交易历史（ERC-20 转账、Swap、借贷利息等） | ❌ 只读 |
| `generate_tax_report` | 根据交易历史生成结构化税务报告（包含：总收益、总亏损、Gas 消耗、净收益） | ❌ 只读 |
| `calculate_gas_spent` | 计算指定钱包在周期内的 Gas 消耗总额（折算 ETH 和 USD） | ❌ 只读 |
| `calculate_token_gains` | 统计每个 ERC-20 代币的买入量、卖出量、当前持仓，计算实现收益 | ❌ 只读 |
| `export_pdf` | 将税务报告导出为 PDF 文件（IPFS 存储，返回 CID） | ❌ 只读 |
| `export_csv` | 将税务报告导出为 CSV 文件（用于导入 Excel） | ❌ 只读 |

### 边界限制（Guardrails）

```
✅ 可以做（只读，自动执行）：
  - 读取公开链上数据（任何钱包地址的交易历史）
  - 生成报告摘要（不产生链上交易）
  - 计算 Gas 消耗（基于历史交易数据）
  - 导出报告文件（IPFS 上传）

❌ 不能做（写入，必须人工确认）：
  - 代表用户签署任何交易
  - 代表用户调用合约
  - 代表用户授权任何代币
  - 修改任何链上状态
```

---

## 输入（Input）

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| `wallet_address` | string（地址） | ✅ | 要生成报告的钱包地址 |
| `start_date` | string（ISO 日期） | ✅ | 报告周期起始日 |
| `end_date` | string（ISO 日期） | ✅ | 报告周期截止日 |
| `report_type` | enum | ✅ | `annual`（年度） / `quarterly`（季度） / `custom`（自定义） |
| `base_currency` | string | ❌（默认 USD） | 计价货币（USD / EUR / CNY） |
| `include_gas` | boolean | ❌（默认 true） | 是否包含 Gas 消耗统计 |
| `tax_regime` | enum | ❌（默认 unspecified） | 税务管辖区（`us` / `de` / `cn` 等，影响计算规则） |

**输入示例：**
```json
{
  "wallet_address": "0x742d35Cc6634C0532925a3b844Bc9e7595f",
  "start_date": "2025-01-01",
  "end_date": "2025-12-31",
  "report_type": "annual",
  "base_currency": "USD",
  "include_gas": true,
  "tax_regime": "us"
}
```

---

## 输出（Output）

| 字段 | 类型 | 说明 |
|------|------|------|
| `report_cid` | string（IPFS CID） | 完整报告文件在 IPFS 上的地址 |
| `summary` | object | 报告摘要（见下方） |
| `generated_at` | string（ISO 时间戳） | 生成时间 |
| `agent_id` | string | TaxReportBot 的链上身份（ERC-8004） |
| `signature` | string | TaxReportBot 对报告内容的签名（ERC-712） |

**summary 示例：**
```json
{
  "wallet": "0x742d...e7595f",
  "period": "2025-01-01 ~ 2025-12-31",
  "total_income": "+12500.00 USDC",
  "total_expenses": "-8320.00 USDC",
  "net_gain": "+4180.00 USDC",
  "gas_spent": "0.85 ETH ($2125)",
  "realized_gains_by_token": {
    "USDC": "+3000.00",
    "ETH": "-$820",
    "DAI": "+2000.00"
  },
  "unrealized_gains": "+$1200 ( holdings at year-end)",
  "tax_liability_estimate": "$850 (us, short-term)"
}
```

---

## 协作对象（Collaborators）

### 与 TaxReportBot 交互的所有外部对象

```
┌─────────────────────────────────────────────────────────────────┐
│                         TaxReportBot                             │
│                                                                  │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐    │
│  │ RPC Node    │ ◄─ │ Alchemy /    │ ◄─ │ 多链数据源   │    │
│  │（链上原始数据）│  │ Infura       │     │ Indexer      │    │
│  └──────────────┘     └──────────────┘     └──────────────┘    │
│                                                                  │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐    │
│  │ IPFS        │ ◄─ │ 报告存储     │     │ PDF生成服务  │    │
│  │（输出存储）  │     │              │     │              │    │
│  └──────────────┘     └──────────────┘     └──────────────┘    │
│                                                                  │
│  ┌──────────────┐     ┌──────────────┐     ┌──────────────┐    │
│  │ ERC-8004    │ ◄─ │ 身份/声誉     │     │ ERC-8183     │    │
│  │ Registry    │     │              │     │ Escrow       │    │
│  └──────────────┘     └──────────────┘     └──────────────┘    │
│                                                                  │
│  ┌──────────────┐                                               │
│  │ User        │ ◄── 任务发起方 + 报告接收方 + 付款方           │
│  │（Alice）    │                                               │
│  └──────────────┘                                               │
└─────────────────────────────────────────────────────────────────┘
```

### 协作关系表

| 协作对象 | 角色 | 交互协议 |
|---------|------|---------|
| Alice（User） | 任务发起方 + 付款方 | A2A / MCP |
| Alchemy / Infura | RPC 数据源 | HTTP/RPC |
| IPFS | 报告存储 | IPFS API |
| ERC-8004 Registry | 身份 + 声誉锚定 | ERC-721 |
| ERC-8183 Escrow | 收款托管 | Smart Contract |
| TaxReportBot Operator | 维护者 | 链下协议 |

---

## 失败点分析（Failure Points）

### 失败场景清单

| # | 失败场景 | 影响 | 防御机制 |
|---|---------|------|---------|
| **FP-1** | RPC 节点返回假数据（交易历史被篡改） | 税务报告数字错误，可能导致用户税务申报失误 | 使用多个 RPC 源交叉验证（Alchemy + Infura + 区块浏览器） |
| **FP-2** | IPFS 上传失败（报告未存储） | 用户无法获取完整报告 | 本地缓存 + 重试机制 + 备用云存储 |
| **FP-3** | Token 价格数据错误（使用错误的 oracle 价格） | 收益计算偏差 | 标注数据来源 + 警告用户此为估算值 |
| **FP-4** | Agent 被恶意调用（恶意方伪装 Alice 请求报告） | 用户隐私泄露 | MCP 认证 + 任务授权确认（需 Alice 签名授权读取自己的钱包） |
| **FP-5** | Escrow 资金锁定后 Evaluator 失联（无法验收） | 超时退款触发 | `expiredAt` 设置足够长 + 备用 Evaluator 地址 |
| **FP-6** | Prompt Injection（恶意指令注入请求） | Agent 执行非预期操作 | 指令边界设计，外部输入不能覆盖核心指令 |
| **FP-7** | 模型生成错误报告（幻觉数据） | 用户基于错误数据做税务决策 | 结果需附带数据源引用 + ERC-712 签名，用户可验证 |

### 最危险的失败点

**FP-4（隐私泄露）和 FP-3（价格错误）** 是最高风险：

- FP-4：恶意调用 → 需要 MCP 级别的认证和授权确认
- FP-3：价格错误 → 税务报告的数值依赖 oracle 价格，价格错则报告错

---

## 收费机制（Payment）

### 收费模式

| 模式 | 价格 | 说明 |
|------|------|------|
| 单次报告 | 5 USDC | 按次计费，适合偶尔需要报告的用户 |
| 月度订阅 | 15 USDC/月 | 无限次报告，适合高频用户 |
| 年度订阅 | 120 USDC/年 | 相当于两个月免费 |

### 支付流程（结合 ERC-8183 Escrow）

```
1. Alice 通过 MCP 调用 TaxReportBot，提出报告需求
2. TaxReportBot 返回报价（5 USDC）和交付标准
3. Alice 授权 ERC-8183 Escrow 合约锁定 5 USDC（Funded 状态）
4. TaxReportBot 执行任务，调用 Escrow.submit(jobId, reportCid) 交付
5. Evaluator（= Alice）验收，调用 Escrow.complete(jobId)
6. Escrow 自动将 5 USDC 转给 TaxReportBot 的钱包
7. x402 生成 payment receipt（税务凭证）
```

### ERC-8004 声誉绑定

- 每次服务完成后，Alice 对 TaxReportBot 评分（1-5 星）
- 评分写入 ERC-8004 Reputation Registry
- 声誉分数（4.7 → 4.8）影响 TaxReportBot 在任务市场的排名

---

## 验证机制（Verification）

### 三层验证

**第一层：数据源验证**
- 交易历史取自至少两个独立 RPC 源（Alchemy + Etherscan API）
- 差异超过 1% 时自动告警，不生成报告，提示用户核查

**第二层：输出签名验证**
- TaxReportBot 使用 ERC-712 对报告摘要签名
- Alice 可以在 Etherscan 上验证签名，确认报告来自 TaxReportBot 且未被篡改

**第三层：IPFS 不可篡改性**
- 报告存储在 IPFS，CID 是内容的密码学哈希
- 任何人下载报告后可用相同哈希验证文件未被修改

---

## 失败处理（Failure Handling）

| 状态 | 处理方式 |
|------|---------|
| **FP-1** RPC 假数据 | 切换备用 RPC + 提示用户数据不一致 |
| **FP-2** IPFS 上传失败 | 本地缓存 + 邮件通知用户 + 72h 内重试 |
| **FP-3** 价格数据错误 | 标注数据来源 + 免责声明，此估算值不构成税务建议 |
| **FP-4** 恶意调用 | 请求签名验证 + 任务授权确认（需 Alice 主动确认） |
| **FP-5** Evaluator 失联 | `claimRefund()` 超时触发退款 |
| **FP-6** Prompt Injection | 拒绝执行 + 记录异常日志 |
| **FP-7** 幻觉数据 | 数据源引用缺失时拒绝生成 + 错误内容可提交 ERC-8004 Validation 争议 |

---

## Protocol 对比加分：MCP vs ERC-8004

| 维度 | **MCP** | **ERC-8004** |
|------|---------|--------------|
| **核心解决** | 接口协议：模型 ↔ 工具怎么连接 | 身份协议：Agent 怎么被找到、被信任 |
| **适用场景** | TaxReportBot 读取 Alchemy RPC、调用 IPFS 上传——这些工具连接用 MCP | Alice 在任务市场发现 TaxReportBot、查看其声誉——这些用 ERC-8004 |
| **支付支持** | ❌ 不涉及支付 | ❌ 不涉及支付 |
| **协作覆盖** | ✅ **工具调用层**（TaxReportBot 的能力接入） | ✅ **发现与信任层**（TaxReportBot 的身份锚定） |
| **组合关系** | MCP 负责"TaxReportBot 能做什么工具" | ERC-8004 负责"TaxReportBot 是谁、值不值得信任" |

**MCP 是 TaxReportBot 的能力接入层**（读取链上数据、输出报告文件），**ERC-8004 是 TaxReportBot 的身份与声誉层**（被发现、被信任、被评分）。两者作用于 Agent 的不同生命周期阶段，不竞争而是互补。

---

## A2A vs MCP（补充对比）

| 维度 | **A2A** | **MCP** |
|------|---------|---------|
| **核心解决** | Agent ↔ Agent 通信：任务协作、代理投票、子任务分包 | 模型 ↔ 工具通信：RPC 调用、文件读写、API 请求 |
| **典型场景** | TaxReportBot 发现一个专门计算 DeFi 收益的子 Agent，把部分任务分包给它 → A2A | TaxReportBot 调用 Alchemy RPC 读取交易历史 → MCP |
| **协作粒度** | 任务级别（"帮我完成这份税务报告"） | 工具级别（"调用这个函数读取数据"） |
| **状态管理** | 有（完整的任务状态机：pending → in_progress → completed） | 无（MCP 是无状态的工具调用协议） |
| **谁用** | Agent 与 Agent | 模型（LLM）与工具 |

A2A 和 MCP 不是竞争关系，而是不同层次的协议：
- **MCP** 是模型调用工具的接口（细粒度）
- **A2A** 是 Agent 协作任务的协议（粗粒度）

一个复杂的 TaxReportBot 会在内部用 MCP 调用数据工具，同时通过 A2A 与其他专业 Agent（如专门算 DeFi 收益的子 Agent）协作。

---

*Week 2 · Module C · Agent Profile · TaxReportBot v0.1 · Minami-Bein · 2026-05-28*