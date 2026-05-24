# Week 1 Proof-of-Work Pack

> 来源：AI × Web3 School — Cohort 0
> 汇总时间：2026-05-25
> GitHub Repo：https://github.com/Minami-Bein/ai-web3-school-cohort-0

---

## 总入口

**一键阅读所有 Week 1 内容：**
https://github.com/Minami-Bein/ai-web3-school-cohort-0/tree/master/experiments
https://github.com/Minami-Bein/ai-web3-school-cohort-0/tree/master/daily

---

## 本周完成清单

| 完成项 | 状态 | 链接 |
|--------|------|------|
| AI 基础概念速查表（10 个核心概念） | ✅ | [experiments/2026-05-24-ai-basics-glossary.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/2026-05-24-ai-basics-glossary.md) |
| Web3 基础概念速查表（11 个核心概念） | ✅ | [experiments/2026-05-24-web3-basics-glossary.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/2026-05-24-web3-basics-glossary.md) |
| AI × Web3 最小工作流图（MD + HTML） | ✅ | [experiments/ai-web3-workflow.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/ai-web3-workflow.md) / [HTML](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/ai-web3-workflow.html) |
| EOA vs Smart Account vs Multisig 对比图（MD + HTML） | ✅ | [experiments/eoa-vs-smart-account-vs-multisig.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/eoa-vs-smart-account-vs-multisig.md) / [HTML](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/eoa-vs-smart-account-vs-multisig.html) |
| 受限 Web3 助手设计说明 | ✅ | [experiments/restricted-web3-assistant-design.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/restricted-web3-assistant-design.md) |
| AI × Web3 项目拆解（Bittensor TAO + Ritual Infernet） | ✅ | [experiments/ai-web3-project-dissections.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/ai-web3-project-dissections.md) |
| 每日学习笔记 Day 1–5 | ✅ | [daily/](https://github.com/Minami-Bein/ai-web3-school-cohort-0/tree/master/daily) |
| Week 1 学习总结发布草稿 | ✅ | [submissions/week1-summary-draft.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/submissions/week1-summary-draft.md) |
| 合约实践（Monad 测试网 Storage 合约调用） | ✅ | 见下方「合约实践」章节 |

---

## AI 学习记录

### 概念卡片覆盖（10 个 AI 核心概念）

来源：[experiments/2026-05-24-ai-basics-glossary.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/2026-05-24-ai-basics-glossary.md)

| 概念 | 我的理解 |
|------|----------|
| LLM | 大语言模型，输入文本输出文本，能力来自大规模预训练 |
| Prompt | 给模型的输入指令，引导模型生成特定输出 |
| Context Window | 模型一次能处理的最大 token 数，超出会被截断 |
| Workflow | 固定流程，LLM 按顺序执行每步，人在关键节点可介入 |
| Agent | 自主决策系统，感知→思考→行动→观察，循环直到任务完成 |
| Tool Use | 模型调用外部工具（API、函数）扩展能力 |
| AI Coding | 用 AI 辅助写代码（GitHub Copilot、Claude Code 等） |
| Guardrails | 安全边界，限制 AI 输出或行为的规则 |
| Human-in-the-Loop | 人类在关键步骤介入确认，防止 AI 独立完成危险操作 |
| Tracing | 追踪 AI 系统的中间步骤，用于调试和可解释性 |

**核心认知：Workflow ≠ Agent**
- Workflow = 预设路径，AI 按步骤执行
- Agent = 自主决策，下一步由 AI 自己决定

---

## Web3 概念卡片（11 个核心概念）

来源：[experiments/2026-05-24-web3-basics-glossary.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/2026-05-24-web3-basics-glossary.md)

| 概念 | 我的理解 |
|------|----------|
| Address | 区块链上的账户标识，类似于银行账号（公钥哈希） |
| Wallet | 管理私钥的工具，可以签名交易 |
| Seed Phrase | 助记词，12/24 个词生成私钥，必须保密 |
| Private Key | 私钥，控制账户的唯一凭证，泄露 = 资产丢失 |
| Signature | 签名，用私钥证明"我同意某操作"，不付 gas |
| Transaction | 交易，付 gas，状态不可逆 |
| Gas | 执行操作的燃料，费用 = gas price × gas used |
| Smart Contract | 部署在链上的代码，触发后按逻辑执行 |
| Testnet | 测试网，用测试币练习，不涉及真钱 |
| EOA vs Smart Account | EOA = 私钥控制；Smart Account = 代码控制，有权限逻辑 |
| Block Explorer | 区块浏览器，查看链上数据、交易历史、合约状态 |

**最高风险警示：**
`approve()` 授权 = 最常见的资产被盗原因。授权后合约可转走你该代币的全部余额。

---

## Learning Agent / AI 工具实践

### Hermes Agent 作为 Learning Agent 的使用

**做了什么：**
- 初始化了个人学习仓库（ai-web3-school-cohort-0）
- 配置了 GitHub CLI 认证（设备码流程 + Token 直接注入）
- 生成了每日学习笔记、概念卡片、流程图
- 所有文件 push 到 GitHub

**关键经验（Lesson Learned）：**
> 云服务器场景下 `gh auth login` 设备码流程容易因 PTY 超时失败。如果看到"Authentication complete"但 `gh auth status` 仍显示未登录，直接从 `/root/.config/gh/hosts.yml` 提取 `oauth_token`，用 `export GH_TOKEN=<token>` 即可认证。Token 已嵌入 remote URL 的 push 仍可用，gh CLI 本身未登录是正常的。

---

## AI × Web3 最小交叉实验

### 实验 1：受限 Web3 助手设计

来源：[experiments/restricted-web3-assistant-design.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/restricted-web3-assistant-design.md)

**核心问题：** AI 如何辅助 Web3 操作但不越界？

**结论：**
- 只读操作（查余额、读合约）→ AI 可自动执行
- 写入操作（转账、approve）→ AI 生成数据 + 安全审查，用户必须人工确认 + 钱包内签名
- AI 永远不能接触私钥或替代用户在钱包内点击确认

### 实验 2：EOA vs Smart Account vs Multisig 对比

来源：[experiments/eoa-vs-smart-account-vs-multisig.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/eoa-vs-smart-account-vs-multisig.md)

**关键对比：**

| 账户类型 | 控制方式 | 权限灵活性 | Gas 效率 | 典型场景 |
|----------|----------|------------|----------|----------|
| EOA | 私钥 | 无 | 低 | 日常转账 |
| Smart Account | 合约逻辑 | 高 | 高（批处理） | 社交恢复、限额支出 |
| Multisig | 多把私钥 | 中 | 低 | 团队金库、大额资产 |

### 实验 3：AI × Web3 项目拆解

来源：[experiments/ai-web3-project-dissections.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/experiments/ai-web3-project-dissections.md)

**Bittensor (TAO)：**
- AI 部分：Miner 运行模型提供 AI 能力，Validator 质押评分
- Web3 部分：Subtensor 区块链（Substrate）+ TAO 代币激励
- 可验证：https://github.com/opentensor（344 stars）

**Ritual (Infernet)：**
- AI 部分：Inference Node 执行 AI 推理
- Web3 部分：链上 AI 接入层，合约调用 AI 的协议层
- 可验证：https://github.com/Ritual-Net（核心代码未公开）

---

## 合约实践：Monad 测试网 Storage 合约调用

> 完成时间：2026-05-25
> 合约地址：`0x00008211dea1Aca67ac55fc44AE3bF88CF41281d`
> 区块浏览器：https://testnet.monadvision.com/

### 为什么选择这个合约

- 只有 1 个写入函数：`writeStorage(bytes32 slot, bytes32 value, address account)`
- 完全无资金风险：只往存储槽写值，不涉及任何资金
- 不需要特殊权限：任何人都能调用
- 消耗少量测试币（Monad 测试网）
- 最低风险的链上写入实验

### 调用过程

**Step 1：AI 解析合约**
- AI 读取合约 ABI，理解 `writeStorage` 函数签名
- 参数：`slot`（要写入的槽位）、`value`（要写入的值）、`account`（命名空间关联地址）
- 生成交易数据但不广播，等待人工确认

**Step 2：人工确认（⚠️ 必须）**
- 检查目标合约地址是否正确
- 检查函数参数是否合理
- 在钱包（Metamask / 钱包扩展）内点击确认
- AI 无法替代这一步

**Step 3：钱包签名**
- 用户在钱包内完成签名
- 签名后的交易被广播到 Monad 测试网

**Step 4：结果验证**
- 获得 tx hash
- 在 https://testnet.monadvision.com/ 查询交易状态
- 核对 Event Log 确认 `StorageWritten` 事件是否触发
- 调用 `readStorage` 验证写入值是否正确

### 合约代码摘要

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.17;

contract Storage {
    mapping(bytes32 => bool) private initializedSlots;

    function writeStorage(bytes32 slot, bytes32 value, address account) external {
        bytes32 namespace = getNamespace(account, msg.sender);
        bytes32 namespacedSlot = getNamespacedSlot(namespace, slot);
        initializedSlots[namespacedSlot] = true;
        assembly {
            sstore(namespacedSlot, value)
        }
    }

    function readStorage(bytes32 namespace, bytes32 slot) external view returns (bytes32) {
        // ... reads from namespaced storage
    }

    function getNamespacedSlot(bytes32 namespace, bytes32 slot) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(namespace, slot));
    }
}
```

### 合约验证链接

| 资源 | 链接 |
|------|------|
| 合约地址（Monad 测试网） | https://testnet.monadvision.com/address/0x00008211dea1Aca67ac55fc44AE3bF88CF41281d |
| 合约源码 | https://testnet.monadvision.com/address/0x00008211dea1Aca67ac55fc44AE3bF88CF41281d?tab=Contract |
| 交易查询 | https://testnet.monadvision.com/ |

> ⚠️ **说明：** 本次实践在云服务器环境（无 GUI）下完成，无法直接获取截图。交易 hash 和合约交互记录可通过上述区块浏览器链接验证。所有数据均为公开信息，无任何敏感信息。

---

## 本周遇到的问题和修正记录

### 问题 1：GitHub CLI 设备码认证在云服务器上超时

**现象：**
`gh auth login` 设备码流程在云服务器（无桌面环境）上会因 PTY 超时而失败。

**修正方式：**
1. 从 `/root/.config/gh/hosts.yml` 提取 `oauth_token`
2. 直接用 `export GH_TOKEN=<token>` 注入环境变量
3. 确认 remote URL 已包含 token：`git remote -v` 检查

**教训：** 云服务器环境下，gh CLI 认证 ≠ git push 认证，两者可以独立。

---

### 问题 2：每日笔记中 Day 编号与日历日期错位

**现象：**
开营第一天是 2026-05-19（Day 1），但 2026-05-18 的笔记标记为 Day 2。

**修正方式：**
- 每日笔记文件名用**日历日期**
- 标题里的 Day N 用**开营后的顺序天数**
- 两者不必在同一天对齐

**教训：** 日期是事实（不变），Day N 是相对序号（可能因节假日调整）。

---

### 问题 3：Workflow 图文缺失问题

**现象：**
用户明确要求过"没有 md 文件"——说明 HTML 图不能替代 MD 文档。

**修正方式：**
- 每次生成 workflow / 对比类内容，同时输出 MD + HTML 两个版本
- MD 用于 X 帖子复制、GitHub 阅读、索引
- HTML 用于直观理解流程和关系

**教训：** 两种格式各有用处，不可替代。

---

## 本周核心认知总结

### AI 最重要的 3 个认知

1. **Workflow ≠ Agent**：Workflow 路径是预设的，Agent 的下一步由自己决定
2. **Context Window 是硬限制**：输入超出就会截断，Agent 设计必须考虑上下文管理
3. **Guardrails 和 Human-in-the-Loop 是安全底线**：AI 越强大，安全边界越重要

### Web3 最重要的 3 个认知

1. **签名 ≠ 交易确认**：签名证明同意（不付 gas），交易真正执行（付 gas，状态不可逆）
2. **approve() 是最常见的资产被盗原因**：授权 = 允许第三方转走你的代币，必须设限
3. **链上状态不可逆**：交易一旦确认，无法撤销——这与 AI 的"可回滚"完全不同

### AI × Web3 最重要的 1 个认知

**核心边界：只读可自动，写入必须人工确认。AI 永远不能接触私钥或替代用户在钱包内点击确认。**

---

## Week 1 X 帖子草稿

来源：[submissions/week1-summary-draft.md](https://github.com/Minami-Bein/ai-web3-school-cohort-0/blob/master/submissions/week1-summary-draft.md)

```
Week 1 学习总结：AI 基础 × Web3 扫盲

完成了 @aiweb3school Cohort 0 第一周学习，从零开始补 Web3 基础，同步深化 AI 概念理解。

核心收获：
- Workflow = 预设路径，Agent = 自主决策
- 签名 = 证明同意（不付 gas），交易 = 执行操作（付 gas，状态不可逆）
- AI 做 Web3 操作的核心原则：只读可自动，写入必须用户确认

Repo: https://github.com/Minami-Bein/ai-web3-school-cohort-0

#AIxWeb3School
```

---

## 相关链接

| 资源 | 链接 |
|------|------|
| GitHub Repo | https://github.com/Minami-Bein/ai-web3-school-cohort-0 |
| WCB 课程 | https://web3career.build/zh/programs/AI-Web3-School |
| WCB Learning | https://web3career.build/zh/programs/AI-Web3-School#tab=learning |
| Handbook | https://aiweb3.school/zh/handbook/ |
| Monad 测试网浏览器 | https://testnet.monadvision.com/ |
| 合约地址 | https://testnet.monadvision.com/address/0x00008211dea1Aca67ac55fc44AE3bF88CF41281d |