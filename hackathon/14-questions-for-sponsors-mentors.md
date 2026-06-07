# 向 Sponsor / Mentor 请教的问题

> Updated: 2026-06-08
> Project: Cat-Agent

---

## 问题 1：Z.AI 多模态推理 API 的接入流程与 Timeline

**背景：**
Cat-Agent 的感知层依赖 Z.AI 的多模态推理 API（摄像头+麦克风 → 猫咪意图+置信度），这是项目最核心的差异化能力。项目的感知层开发进度直接取决于 Z.AI API 是否可用。

**具体问题：**
1. Z.AI 多模态推理 API 的申请流程是什么？需要哪些材料？
2. 从提交申请到拿到可用 API Key 的typical timeline 是多久？
3. 是否有测试额度（test credits）或沙盒环境可以提前接入开发？
4. 如果 Z.AI API 在 6/20 前无法到达，是否有其他 Partner 渠道可以加速？

**期望答案：**
明确的 API Key 到达时间，或测试额度获取方式，以便安排开发优先级（先接 Z.AI 还是先做 OpenAI Fallback）。

---

## 问题 2：Cobo CAW SDK 的文档可用性和 Access 申请流程

**背景：**
Cat-Agent 的权限控制层依赖 Cobo CAW Warrant 实现链上的 Budget/Scope/Time 三重校验。如果 CAW SDK 文档不完整或 Access 审批周期长，整个 Policy Engine 开发将被迫使用本地 Mock，影响项目展示效果。

**具体问题：**
1. Cobo CAW SDK 目前的文档完善程度如何？是否有 Python SDK 或 REST API 示例？
2. CAW Warrant 的链上配置（budget/scope/time_window）是否有参考实现可以先行研究？
3. CAW Access（开发者权限）的申请流程和审批周期是？
4. 如果 CAW SDK 在 6/22 前无法接入，Cat-Agent 的 Policy Engine 可以参考哪些现有的链上权限标准（如 ERC-7734、Safe Modules 文档）先行开发？

**期望答案：**
SDK 文档链接或 Reference 实现地址，以及明确的 Access 申请 Timeline。如果 CAW SDK 不确定可用，期望得到其他链上权限标准的建议作为备选。

---

## 问题 3：Safe{Wallet} SpendingLimitModule 的正确配置方式

**背景：**
Cat-Agent 使用 Safe{Wallet} 的 SpendingLimitModule 实现日预算和单笔限额硬限制。在测试网部署时，需要正确配置 Module 参数（日限额 10 USDC、单笔限额 10 USDC）。Safe 模块的配置有多个参数组合，如果配置错误可能导致：限额形同虚设，或正常交易被意外拦截，影响 Demo 演示。

**具体问题：**
1. Safe SpendingLimitModule 的日限额（dailyLimit）和单笔限额（txLimit）在部署时的配置顺序是什么？是否有标准 JSON 配置模板？
2. SpendingLimitModule 和 AllowlistGuard 同时启用时，两者的拦截优先级是怎样的？（比如：交易超过日限额但目标在白名单内，会被哪个模块拦截？）
3. Session Key 配合 SpendingLimitModule 使用时，Session Key 的权限是否需要特别配置（如 onlyAllowDelegateCall 等）？
4. 在 Sepolia 测试网上，SpendingLimitModule 的日限额是按区块时间重置还是按链上时间戳重置？

**期望答案：**
 SpendingLimitModule 的标准配置 JSON 示例，以及 AllowlistGuard 同时启用时的注意事项，避免部署时踩坑。

---

## 问题使用说明

| 问题 | 问谁 | 优先级 |
|------|------|-------|
| Z.AI API 接入流程 | Sponsor（Z.AI 联系人）| P0 |
| Cobo CAW SDK 文档和 Access | Sponsor（Cobo 联系人）| P0 |
| Safe SpendingLimitModule 配置 | Mentor / TA / Safe 文档 | P1 |

**提问时机：**
- 问题 1 和 2：尽快联系（6/18 前），这两个决定项目的 Fallback 策略
- 问题 3：可以在 Workshop 或群聊中提出，技术细节不影响整体架构