# Sprint Plan（黑客松冲刺计划）

## 总时间线
- **开始：** 2026-06-08
- **结束：** 2026-06-30（黑客松提交截止）
- **总天数：** 23 天（含端午节假期）

---

## Sprint 1：基础设施搭建（6/8 - 6/12，5天）

### 目标
完成项目初始化、仓库结构、CI/CD、以及 CAW Warrant + Safe{Wallet} 集成准备。

| 任务 | 子任务 | 负责 | 截止 | 状态 |
|------|--------|------|------|------|
| 创建 GitHub 仓库 | init, license, README, gitflow | Minami-Bein | 6/8 | ⬜ |
| CI/CD 配置 | GitHub Actions + pytest + flake8 | Minami-Bein | 6/9 | ⬜ |
| Safe{Wallet} 部署 | 测试网 Safe 部署 + Session Key 生成 | Minami-Bein | 6/10 | ⬜ |
| CAW Warrant 注册 | 写入链上 Warrant 配置 | Minami-Bein | 6/11 | ⬜ |
| x402 Server 搭建 | 本地 x402 受保护 API 模拟 | Minami-Bein | 6/12 | ⬜ |

### 交付物
- 公开可访问的 GitHub 仓库
- 测试网 Safe{Wallet} 地址（可查看）
- CAW Warrant 链上记录

---

## Sprint 2：感知 + 意图层（6/13 - 6/17，5天）

### 目标
完成多模态 AI 推理（音频+视觉）和意图置信度引擎。

| 任务 | 子任务 | 负责 | 截止 | 状态 |
|------|--------|------|------|------|
| 音频分析模块 | 频率/时长/音调提取 | Minami-Bein | 6/14 | ⬜ |
| 视觉分析模块 | 猫咪姿态/表情/眼神识别 | Minami-Bein | 6/14 | ⬜ |
| Z.AI 集成 | 多模态推理 API 接入 | Minami-Bein | 6/15 | ⬜ |
| 意图分类器 | 零食/玩耍/出门/拒绝 四分类 | Minami-Bein | 6/16 | ⬜ |
| 置信度引擎 | 两轮交互 + 90% 阈值触发 | Minami-Bein | 6/17 | ⬜ |
| 物理反馈 | 激光笔 + 扬声器驱动（或模拟器）| Minami-Bein | 6/17 | ⬜ |

### 交付物
- 意图分类 API（输入：音频+图像，输出：意图+置信度）
- Demo：猫咪行为 → 置信度分数实时展示

---

## Sprint 3：支付 + 钱包层（6/18 - 6/22，5天）

### 目标
完成 CAW Warrant 检查、x402 支付闭环、Safe{Wallet} 执行。

| 任务 | 子任务 | 负责 | 截止 | 状态 |
|------|--------|------|------|------|
| CAW Policy Engine | Budget/Scope/Time Window 三重检查 | Minami-Bein | 6/19 | ⬜ |
| x402 感知 Client | 401 检测 → 自动支付 → 收据携带重试 | Minami-Bein | 6/19 | ⬜ |
| USDC 转账集成 | Safe{Wallet} Session Key 签名 | Minami-Bein | 6/20 | ⬜ |
| 交易记录 | 链上事件 + IPFS 审计日志 | Minami-Bein | 6/21 | ⬜ |
| ERC-712 签名 | 报告摘要签名（可选） | Minami-Bein | 6/22 | ⬜ |
| RLPF 基础版 | 进食反馈权重更新 | Minami-Bein | 6/22 | ⬜ |

### 交付物
- 完整支付闭环 Demo（模拟猫咪购买商品）
- 链上交易记录可查

---

## Sprint 4：产品 + 演示（6/23 - 6/29，7天）

### 目标
完成演示视频、README完善、产品文案、黑客松提交。

| 任务 | 子任务 | 负责 | 截止 | 状态 |
|------|--------|------|------|------|
| 演示视频录制 | 全流程 Demo（60秒内） | Minami-Bein | 6/25 | ⬜ |
| 黑客松提交页面 | 填写项目介绍 + 上传 Demo | Minami-Bein | 6/27 | ⬜ |
| README 完善 | 架构图 + 快速启动 + 截图 | Minami-Bein | 6/27 | ⬜ |
| 追加 Sponsor 问题 | 向 Cobo/Z.AI 提问获取支持 | Minami-Bein | 6/26 | ⬜ |
| 最终测试 | 全流程回归测试 | Minami-Bein | 6/28 | ⬜ |
| 提交确认 | 确认提交成功 | Minami-Bein | 6/29 | ⬜ |
| 备用缓冲 | Bug 修复 / 演示优化 | Minami-Bein | 6/30 | ⬜ |

### 交付物
- 演示视频（上传 YouTube/Vimeo，链接在 README）
- 黑客松提交页面完成
- 项目文档完整可读

---

## 每日站会模板

```markdown
## 2026-06-XX 日站会

### 昨日完成
- [ ] ...

### 今日计划
- [ ] ...

### 卡点
- [ ] ...

### 明日计划
- [ ] ...
```

---

## 关键依赖

| 依赖项 | 预计获取时间 | 风险预案 |
|--------|------------|---------|
| Z.AI 多模态 API | 6/13 | 用 OpenAI Vision API 替代 |
| Cobo CAW 文档 | 6/10 | 参考 GitHub 参考实现 |
| 测试网 USDC 水龙头 | 6/10 | 用 Goerli Sepolia 官方水龙头 |
| 测试网 ETH | 6/10 | Alchemy 免费测试网 RPC |

---

## 成功标准

- [ ] 演示视频展示完整流程（猫咪需求 → AI 确认 → 链上支付 → 交付）
- [ ] 每笔链上交易在 Etherscan 可查
- [ ] CAW Warrant 策略可配置、可验证
- [ ] README 包含快速启动指南
- [ ] 代码有基本测试覆盖（>60%）
