# Week 1 学习总结 — 发布草稿

> 发布平台：X / Twitter
> 发布状态：待发布（请手动复制到 X 发布）

---

## Week 1 学习总结：AI 基础 × Web3 扫盲

完成了 @aiweb3school Cohort 0 第一周学习，从零开始补 Web3 基础，同步深化 AI 概念理解。

**Repo**: https://github.com/Minami-Bein/ai-web3-school-cohort-0

---

### 🧠 重新理解的 AI 概念：Agent 与 Workflow 的本质区别

以前以为"加了 tool use 就是 Agent"，学完才明白：

**Workflow** = 固定流程，LLM 按顺序执行每一步，人在关键节点可介入
**Agent** = 自主决策系统，感知→思考→行动→观察结果，循环直到任务完成

关键区别在于**自主性**：Workflow 的路径是预设的，Agent 的下一步由它自己决定。

举例：一个 Web3 研究 Agent 发现地址交互了 Aave，不是按预设步骤走，而是自己决定下一步查清算记录。这才是 Agent。

---

### ⛓️ 重新理解的 Web3 概念：钱包签名 ≠ 交易确认

最颠覆认知的一点：

- **签名（Sign）** = 证明"我同意"，不付 gas，不改变链上状态
- **交易确认（Transaction）** = 真正执行操作，消耗 gas，状态不可逆

这两个概念太容易混用了。Agent 做 Web3 操作时，必须分清楚"我只是在签名"还是"我要发起交易"。钓鱼攻击正是利用这个混淆——让你签一个恶意消息，你以为只是登录，实际已经授权了转账。

---

### 🔗 AI × Web3 交叉问题：哪些操作必须人工确认

Agent 执行 Web3 操作时，核心安全原则：

**只读操作**（读链上数据、查余额、看合约源码）→ 可自动执行
**写入操作**（转账、approve 授权、与新合约交互、投票）→ 必须用户明确确认

这个原则也映射到 guardrails 设计：
- 转账超过阈值 → 人工确认
- 调用未经白名单的合约 → 人工确认
- 授权第三方动用资产 → 明确告知授权范围和额度

---

### ✅ 本周 Proof of Work

**1. GitHub Repo 建立 + 每日笔记**
https://github.com/Minami-Bein/ai-web3-school-cohort-0
- 每日学习笔记：Day 1-5（5月18-23日）
- 包含 WCB 课程跟进、Handbook 阅读笔记

**2. AI 基础概念速查表**（10 个核心概念）
- LLM、Prompt、Context Window、Workflow、Agent、Tool Use、AI Coding、Guardrails、Human-in-the-Loop、Tracing
- 链接：experiments/2026-05-24-ai-basics-glossary.md

**3. Web3 基础概念速查表**（11 个核心概念）
- Address、Wallet、Seed Phrase、Private Key、Signature、Transaction、Gas、Smart Contract、Testnet、EOA vs Smart Account、Block Explorer
- 重点标注：approve() 授权是最常见资产被盗原因
- 链接：experiments/2026-05-24-web3-basics-glossary.md

---

### ❓ 还没解决的问题 & 下周方向

**未解决**：智能合约到底怎么部署？Foundry/Hardhat 的开发流程是什么样的？看了概念但还没实际跑过一个部署命令。

**下周计划**：
- 领取 Sepolia 测试网 ETH，实际操作一笔测试网交易
- 用 Foundry 部署一个最简单的合约（Counter）
- 理解 Etherscan 上怎么看合约源码和交易详情
- 探索 AI Agent × Web3 的具体切入点：链上数据查询方向

---

Tagging: @LXDAO_Official @ETHPanda_Org @aiweb3school @Zai_org @web3careerbuild

#AIxWeb3School

---

## 发布说明

此文件为 Week 1 学习总结草稿，已验证：
- ✅ 不含私钥、助记词、API Key、token、.env 等敏感信息
- ✅ 所有引用为公开资料和真实学习笔记
- ✅ Repo 为公开仓库，内容可核实
