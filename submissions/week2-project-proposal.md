# AI × Web3 School · Week 2 · Project Proposal
**TaxReportBot + x402 自主支付闭环**
Minami-Bein · 2026-05-28

---

## 目标用户

**个人加密交易者**（美国/德国/中国）
- 痛点：链上活动复杂，手动算税极易出错
- 需求：自动化生成合规税务报告

**Web3 项目财务人员**
- 痛点：多钱包、多链收支汇总困难
- 需求：一键导出全链路财务记录

**DAO 金库管理员**
- 痛点：链上治理支出缺乏结构化报告
- 需求：生成可审计的国库支出报告

---

## 真实场景

Alice（个人交易者）需要 2025 年度税务报告

流程：
1. Alice发送请求：帮我生成 2025 年税务报告
2. TaxReportBot 通过 MCP 调用 Alchemy RPC 读取钱包历史（只读）
3. TaxReportBot 通过 Etherscan API 交叉验证数据（只读）
4. 若 AI 推理 API 受 x402 保护：CAW Warrant 检查后自动完成 USDC 支付
5. 获取 AI 推理结果，生成结构化税务报告
6. PDF 导出并上传 IPFS，返回 CID
7. TaxReportBot 使用 ERC-712 签名报告摘要，Alice 可链上验证

关键价值：Alice 无需手动整理交易记录，Agent 在 CAW Warrant 授权范围内自主完成全流程，每笔支付可审计。

---

## 最小功能（MVP）

P0（必须交付）：
1. 读取单钱包、指定周期的链上交易历史（MCP + Alchemy RPC，只读）
2. 生成结构化税务报告：收益/亏损/Gas/净收益（AI 推理，可 x402 保护）
3. PDF 导出 + IPFS 存储 + ERC-712 签名报告摘要
4. x402 感知 HTTP Client：401 检测 → 自动支付 → 收据携带重试
5. CAW Warrant 预算引擎：日预算50 USDC / 单笔上限10 USDC / 白名单 / 时间窗口7天

P1（第二阶段）：
6. 多数据源交叉验证（Alchemy + Etherscan，差异超过1%自动告警）
7. ERC-8004 链上身份注册 + 声誉评分

P2（第三阶段）：
8. 月度订阅 + ERC-8183 Escrow 托管支付

MVP 交付物：可运行的 TaxReportBot，支持单一钱包年/季度报告生成，支持 x402 付费 API 调用，每笔支付带 CAW Policy 校验。

---

## 验证方式

数据层验证：Alchemy + Etherscan 双源交叉，差异超过1%告警，数据一致性目标99%
签名验证：Alice 在 Etherscan 验证 ERC-712 签名，确认报告来自 TaxReportBot，签名可验证率目标 100%
支付可审计性：每笔 x402 支付生成 tx_hash 收据，Alice 可在链上查验，支付记录可查率目标 100%
预算强制：Safe SpendingLimitModule 链上硬限制，超出则交易回退，预算超支事件目标 0
白名单校验：AllowlistGuard 拦截所有非白名单合约调用，拦截率目标 100%
产品验证：向 5 名加密用户演示，收集准确率反馈，NPS 目标 7/10

---

## 风险边界

高风险：
x402 收据伪造 -防御：Server 链上验证 tx_hash（区块链不可伪造）
Session Key 被盗 - 防御：SpendingLimitModule 限制最大日损失 50 USDC

中风险：
收据重放攻击 - 防御：Server 维护 used_receipts 集合防重放，残余风险：Server 单点依赖
Prompt Injection - 防御：指令边界隔离，外部输入不解析为命令，残余风险：低
RPC 假数据 - 防御：多源交叉验证加差异告警，残余风险：依赖用户主动核查
报告幻觉数字 - 防御：报告须附带数据源引用，缺失数据则拒绝生成，残余风险：低

Agent 行为红线（永远不能执行）：
approve / permit / transferFrom（任何授权类操作）
调用非白名单合约
单笔转账大于等于 10 USDC
日操作超过 20 次

---

## 可能赛道

税务合规（Tax Compliance）：直接落地，链上税务报告是强需求，x402 按次付费模式天然匹配，契合度最高
Agent 基础设施（Agent Infrastructure）：TaxReportBot 是 AI Agent 商业化具体案例，验证 x402 加 CAW 授权机制，契合度高
DeFi 数据分析（DeFi Data / Analytics）：交易历史读取加结构化报告属于链上数据分析范畴，契合度中
钱包即服务（Wallet-as-a-Service）：Safe 加 Session Key 加 CAW Warrant 是 WaaS 核心功能集，契合度中
支付与 x402（Payments / x402）：TaxReportBot 是 x402 协议消费者，验证支付闭环，契合度中

最快落地赛道：税务合规（目标用户明确、付费意愿强、x402 按次计费模式天然匹配）

---

## 项目信息

项目名称：TaxReportBot
类型：Personal AI Agent（个人财务助手）
核心协议：MCP（能力接入）加 x402（支付）加 ERC-8004（身份）加 ERC-8183（托管）
定价：单次报告 5 USDC，月度订阅 15 USDC
技术栈：Python Agent 加 Alchemy RPC 加 Safe Wallet 加 x402 加 IPFS 加 ERC-712
风险评级：中（Agent 只读操作，无资产直接损失风险）
