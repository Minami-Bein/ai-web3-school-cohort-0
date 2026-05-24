# AI × Web3 项目拆解：Bittensor & Ritual (Infernet)

> 来源：AI × Web3 School — 个人学习笔记
> 日期：2026-05-25
> 目的：拆解两个真实 AI × Web3 项目，训练识别问题、技术路径和 proof-of-work 的能力

---

## 项目一：Bittensor (TAO)

> 定位：去中心化 AI 生产力与激励层（Substrate-based 区块链）
> 官网：https://bittensor.com/
> GitHub：https://github.com/opentensor

---

### 1. 它在解决什么问题

**中心化 AI 市场的两个困境：**
- **供给侧**：个人 AI 开发者 / 小团队有好的模型，但没有分发渠道和变现方式
- **需求侧**：需要 AI 能力的企业或个人，只能依赖 OpenAI、Google 等闭源平台，无法议价、无法定制

**Bittensor 的核心解法：**
把 AI 模型变成一个**去中心化商品市场**——任何人可以贡献 AI 能力（运行模型节点），任何人可以消费 AI 能力（发起请求并支付），价格由市场机制决定而非平台定价。

---

### 2. AI 部分

**Miner（算力提供者）：**
- 运行自己的 AI 模型（文本、图像、代码等各类模型）
- 接收来自验证者的请求，执行推理并返回结果
- 模型类型不受限制——可以用任何架构、任何数据集
- 类似于"把 AI 模型变成 API 服务商"

**Validator（验证者）：**
- 运行"裁判节点"，对 Miner 返回的结果质量打分
- 不运行自己的模型，只做评判
- 评分权重由持有的 TAO 代币量决定（权益证明）
- 类似于"AI 能力的评级机构"

**激励机制：**
- Validator 评分高 → Miner 获得更多 TAO 奖励
- Validator 评分被其他Validator 认可 → Validator 获得奖励
- 形成一个**基于质量的激励循环**，而非简单算力竞争

**关键创新点：**
- 模型多样性：不同 Miner 可以提供完全不同类型的模型
- 无需数据共享：各节点的私有数据不被要求公开
- 蒸馏层（Distillation）：用网络蒸馏技术让模型之间可以互相学习

---

### 3. Web3 部分

**Subtensor（区块链层）：**
- 基于 Substrate 框架（Polkadot 生态）
- 存储 Miner 的权重、Validator 的评分、质押信息
- 区块确认 + TAO 转账

**TAO 代币经济：**
- 总供应量：21,000,000 TAO（类似 BTC 的通缩机制）
- 用途：
  - 质押给 Validator（影响评分权重）
  - 支付给 Miner（使用 AI 服务）
  - 通胀激励（年化约 5%）
- 与 BTC 的区别：通胀部分不是给矿工，而是给 Validator 和 Miner 的贡献者

**Subnet（子网）机制：**
- 网络支持多个并行的子网，每个子网专注不同类型的 AI 任务
- 子网 1：通用文本模型
- 子网 X：图像模型、代码模型等
- 可以自由创建新子网，让市场决定哪些类型的 AI 最有价值

**链上数据（可验证）：**
- TAO 持有量 / 质押量：链上可查
- Miner 评分历史：链上记录
- 各子网的活跃度和收益：链上可查
- 合约调用：不需要智能合约（Substrate 链原生存量）

---

### 4. 可验证材料

| 材料类型 | 链接 / 说明 |
|----------|-------------|
| 白皮书 | https://bittensor.com/whitepaper |
| 官方文档 | https://docs.bittensor.io/ |
| GitHub（核心） | https://github.com/opentensor/subtensor（区块链层，344 stars） |
| GitHub（开发文档） | https://github.com/opentensor/developer-docs（43 stars） |
| Python SDK | `pip install bittensor`（bittensor Python 包） |
| 区块浏览器 | https://taostats.io/（非官方，但数据全面） |
| 代币合约 | TAO 是 Substrate 原生代币，非 ERC-20 |
| 链上记录 | subtensor 区块链，opentensor 组织维护 |

**Torch2BT（相关有趣项目）：**
- GitHub：Shindora/torch2bt（17 stars）
- 功能：将 PyTorch 模型一键打包为 Bittensor Miner，自动生成 synapse、Dockerfile

---

### 5. 我的判断、启发与疑问

**启发：**
- Bittensor 解决的不是"AI 太贵"，而是"AI 市场太中心化"——用代币激励创造了一个真正竞争的质量市场
- 质押权重决定评分影响力——这与 Delegated Proof of Stake 类似，防止女巫攻击
- 子网设计允许市场自然分化不同任务类型，而不是用一个通用模型解决所有问题

**疑问：**
- Validator 的评分标准是什么？是否存在 Validator 串通给特定 Miner 打高分的行为（类似矿池串通）？
- 当模型质量难以客观量化时（比如创意写作）， Validator 的评分机制如何保证公平？
- TAO 的价格波动如何影响 AI 服务的定价稳定性？

**我的判断：**
Bittensor 是一种**基础设施赌注**——它假设去中心化 AI 市场会爆发，且市场会选择质量而非单纯价格。如果 AI 模型继续集中于大公司，Bittensor 的价值主张会更强；如果 AI 成本持续下降且趋于商品化，Bittensor 的竞争优势会减弱。

---

## 项目二：Ritual (Infernet)

> 定位：AI 推理共处理器 / 链上 AI 接入层（将 AI 能力引入智能合约）
> 官网：https://ritual.net/
> GitHub：https://github.com/Ritual-Net（组织层面，无公开 repo）

---

### 1. 它在解决什么问题

**智能合约的一个根本限制：**
- 合约只能执行确定性逻辑——相同输入必须产生相同输出
- AI 模型的本质是概率性的——同样输入可能产生不同输出
- 合约无法直接调用外部 AI（HTTP 请求不是原子的，且无法验证结果真实性）

**当前解决方案的缺陷：**
- 依赖第三方预言机（Chainlink 等）提供链下数据，但不支持复杂 AI 推理
- 如果合约自己运行 AI 模型，成本极高且无法保证隐私
- 现有的"链上 AI"多为链下计算 + 链上验证的简单模式，无法满足复杂任务

**Ritual 的核心解法：**
创建一个**专门为 AI 推理设计的链下计算层（Infernet）**，让智能合约可以请求任意 AI 模型执行，并将结果以可验证的方式返回链上。

---

### 2. AI 部分

**Inference Nodes（推理节点）：**
- 运行 AI 模型的服务器节点网络
- 模型类型不受限制（LLM、图像模型、语音模型等）
- 节点通过激励机制参与，提供算力换取奖励

**与 Bittensor 的关键区别：**
| 对比维度 | Bittensor | Ritual (Infernet) |
|----------|-----------|-------------------|
| **核心功能** | AI 模型质量的市场化 | 智能合约调用 AI 的基础设施 |
| **用户角色** | Miner（模型提供者）/ Validator（评分者） | Inference Node（算力提供者）|
| **激励机制** | 基于质量评分的代币激励 | 基于计算任务完成的代币支付 |
| **调用方式** | P2P 网络内部请求 | 智能合约通过 SDK 调用 |
| **输出类型** | 模型推理结果 | 带验证的推理结果（附证明） |

**支持隐私计算（理论方向）：**
- Ritual 文档提到"隐私计算"和"TEE（可信执行环境）"
- 意味着某些敏感数据可以在加密环境下被 AI 处理，不会被节点看到原文
- 这是 Ritual 区别于普通"AI API"的核心差异点

---

### 3. Web3 部分

**链无关架构（Chain-agnostic）：**
- Ritual 被设计为可以在以太坊、Solana、多条 EVM 链上运行
- 不绑定单一 L1，而是作为多链的 AI 接入层

**智能合约调用流程：**
```
1. DApp 开发者（合约）调用 Ritual SDK
2. 请求被发送至 Ritual 网络
3. Inference Node 执行 AI 推理
4. 结果被附上证明（proof）返回链上
5. 合约收到结果，继续执行
```

**Infernet 合约：**
- 核心合约负责请求路由、节点选择、结果验证
- 开发者通过 SDK 发起请求，不需要自己运行节点
- 类似于"AI 领域的 Chainlink"，但专门处理 AI 推理而非数据

**代币机制（推测）：**
- Ritual 网络需要支付费用才能调用 AI 推理
- 节点通过提供服务获得代币奖励
- 细节待官方文档明确（文档域名 infernet.ml 尚不可访问）

---

### 4. 可验证材料

| 材料类型 | 链接 / 说明 |
|----------|-------------|
| 官网 | https://ritual.net/ |
| GitHub 组织 | https://github.com/Ritual-Net（无公开 repo，内容待发布） |
| 社区节点指南 | https://github.com/arcxteam/ritual-node（5 stars，非官方） |
| 开发者讨论 | https://github.com/search?q=ritual+infernet（多个社区贡献的节点运行教程） |
| X / Twitter | @ritualnet（官方） |
| Discord | Ritual 官方 Discord（社区支持） |

**现状说明：**
- Ritual 的核心代码库未公开（2026 年 5 月时）
- 公开可见的主要是社区贡献的节点运行指南和 SDK 使用文档
- 文档域名 infernet.ml 无法解析，可能还在建设中

**社区节点实现（非官方）：**
```
arcxteam/ritual-node — 5 stars，AI provider guide
kphed/infernet-node-typescript — 2 stars，TypeScript 实现
Mayankgg01/Ritual_Infernet_Node_Guide — 27 stars，完整指南
```
注意：这些是社区贡献，非官方发布。

---

### 5. 我的判断、启发与疑问

**启发：**
- Ritual 的价值主张在**扩展智能合约的能力边界**——合约不再只能做确定性计算，可以请求 AI 进行概率性推理
- "链上 AI"的核心问题不是"能不能调用 AI"，而是"调用的 AI 结果是否可信"—— Ritual 的证明机制可能是关键
- 如果隐私计算（TEE）真的实现，Ritual 可以解决"AI 需要数据但不能暴露数据"的核心矛盾

**疑问：**
- Ritual 的"证明"机制具体是什么？零知识证明？TEE 远程证明？如果是 TEE，硬件信任假设是否足够安全？
- 链上 AI 请求的延迟如何？智能合约的原子性要求是否与 AI 推理的异步性冲突？
- 如果节点故意返回错误的推理结果，是否有惩罚机制？作恶成本有多高？
- 在 OpenAI API 已经很便宜的情况下，Ritual 的定价优势在哪里？

**我的判断：**
Ritual 是一个**解决智能合约能力上限**的项目——它的目标不是替代 Bittensor 的"AI 市场"，而是成为 AI 与区块链之间的**协议层**。成功的关键在于：证明机制是否可靠、延迟是否可接受、定价是否有竞争力。

目前公开材料有限，核心代码未开源，生态成熟度待观察。

---

## 横向对比：两个项目的本质差异

| 维度 | Bittensor (TAO) | Ritual (Infernet) |
|------|-----------------|-------------------|
| **解决的问题** | AI 能力分散、变现困难、质量难评估 | 智能合约无法调用 AI、AI 结果无法验证 |
| **AI 角色** | 模型即产品，Miner 卖模型能力 | AI 即服务，Node 卖算力执行 |
| **Web3 角色** | 去中心化市场 + 激励层 | 链上 AI 接入层 + 验证层 |
| **激励机制** | 基于质量评分的代币通胀 | 基于任务完成的费用支付 |
| **目标用户** | AI 开发者（卖模型）、消费者（买 AI） | DApp 开发者（需要 AI 增强合约） |
| **生态成熟度** | 高（subtensor 主网运行，52 个 repo） | 低（核心代码未公开，文档在建） |
| **可验证性** | 高（链上数据全透明） | 中低（架构清晰但实现细节未公开） |

---

## 从这两个项目中识别到的通用模式

**模式 1：AI × Web3 项目的两种切入角度**
- **Bittensor 模式**：用 Web3 机制改善 AI 市场的激励问题（代币激励 + 质量评分）
- **Ritual 模式**：用 AI 能力扩展 Web3 的计算边界（智能合约 + AI 推理）

**模式 2：Proof-of-Work 的变形**
- Bittensor 用"模型质量贡献"替代传统 PoW 的算力消耗
- 验证者通过主观判断（但有经济利益约束）而非客观计算来确定质量

**模式 3：去中心化 AI 的核心挑战**
- 如何在不泄露数据隐私的前提下验证 AI 执行？
- 如何衡量 AI 质量（主观任务 vs 客观任务）？
- 链上 AI 请求的延迟和成本是否可控？

---

## 相关链接

**Bittensor**
- 官网：https://bittensor.com/
- 白皮书：https://bittensor.com/whitepaper
- GitHub：https://github.com/opentensor
- 区块数据：https://taostats.io/

**Ritual**
- 官网：https://ritual.net/
- GitHub：https://github.com/Ritual-Net
- 社区节点指南：https://github.com/arcxteam/ritual-node
- 节点 TypeScript 实现：https://github.com/kphed/infernet-node-typescript

**工具类**
- torch2bt（PyTorch → Bittensor Miner）：https://github.com/Shindora/torch2bt