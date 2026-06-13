# Cat-Agent Demo Story

> 3 分钟 Demo 脚本 · Demo Day 展示用
> AI × Web3 School Cohort 0 · Week 4 Hackathon
> 日期：2026-06-13

---

## Demo 目标

展示一条完整的"猫咪自主购买"最小闭环——从 AI 感知到链上交易确认。

**核心信息：** 这是一个可以在 3 分钟内讲完、每个技术人都能验证的 Demo。

---

## Demo 舞台布置

**设备：** 电脑屏幕 + 投屏（Chrome → 全屏）
**提前打开：**
- [ ] Anvil 终端（已启动 local node + 已部署合约）
- [ ] `cast receipt` 验证命令准备好
- [ ] `cast call` 日限额查询准备好
- [ ] `cast storage` SLM 存储读取准备好
- [ ] Telegram Bot 测试窗口
- [ ] [Demo Day 复盘文档](./04-cat-agent-demo-day.md) 备查

---

## 3 分钟 Demo 脚本

### 第 1 分钟：问题（60 秒）

**[画面：黑色背景 + 标题]**

> "今天主人不在家。猫咪 Mimi 饿了。"
> "它走到食物盆旁边，对着摄像头叫了一声。"

**[画面切换：展示猫咪叫的示意 + 意图识别日志]**

```
[AI Perception]
Cat behavior: meowing + approaching food bowl
Intent: buy_snack
Confidence: 94%
Product: tuna_can (2 USDC)
```

> "猫咪无法使用信用卡。现有方案——App Store 礼品卡——权力过度集中在主人手里。"
> "Crypto 支付？需要人类私钥签名。"
> "猫咪需要自己的金融代理。"

**[切到核心观点画面]**
> "Cat-Agent：让 AI Agent 在主人设定的链上边界内，替猫咪自主做购买决策。"

---

### 第 2 分钟：流程演示（90 秒）

**[画面：架构图 + 实时日志滚动]**

> "我们跑通了一条最小闭环。"

```
🐱 猫咪意图（AI感知）
    ↓ 置信度 ≥ 90%
📋 规则引擎 Budget 检查（日限额 / 单笔限额）
    ↓ 通过
⛓️ 链上交易（Safe{Wallet} Session Key 签名）
    ↓
💸 SpendingLimitModule 转账 USDC 给商家
    ↓
🛡️ 链上防御（日限额 / 单笔限额双重拦截超出交易）
```

**[画面：终端滚动日志——规则引擎检查通过]**
```
[Policy Engine] Warrant check
  日限额：10 USDC ✓
  单笔限额：10 USDC ✓
  商户白名单：通过 ✓
  结论：APPROVED → sign_transaction()
```

**[画面：交易广播]**
```
TX Hash: 0x70d3db2fac2ced63a6ba8263b0fa290dd1d9ef5b4778e7a68014768d0c0accaa
Status: 1 (success)
Gas Used: 58,194
Events: 2 (Transfer + TransferSpendingLimit)
```

---

### 第 3 分钟：验证 + 防御测试（60 秒）

**[画面：cast 命令输出]**

> "每一步都可以在链上验证。"

```bash
# 余额验证
Cat Safe USDC: 94 USDC ✅
Merchant USDC: 6 USDC ✅

# 日限额
Daily Remaining: 8 / 10 USDC ✅

# SLM 存储
slot0: USDC address ✓
slot1: dailySpent = 4e6 (4 USDC spent today) ✓
```

**[画面：防御测试——超额交易被拦截]**

> "更重要的是：边界是链上强制的，不是应用层说说而已。"

```bash
# 尝试支付 11 USDC（超过单笔限额 10 USDC）
TX Status: 0 (REVERTED)
Revert Reason: "SpendingLimitModule: per_tx_limit exceeded"
```

> "11 USDC 被链上直接拦截，合约级别的防御——应用层无法覆盖。"
> "这就是 Web3 带来的核心价值：规则写在合约里，不可绕过。"

**[收尾画面]**
```
Cat-Agent 最小闭环 ✅
下一步：Sepolia + Cobo Agentic Wallet 真实签名
```

---

## Demo 中 Mock / 未完成的部分

| 模块 | 状态 | 说明 |
|------|------|------|
| **AI 感知层（Z.AI）** | 🔶 Mock | 意图通过 Python 脚本模拟输出，confidence=0.94 |
| **Cobo CAW Warrant** | 🔶 Mock | Warrant 为本地 JSON 规则，未接入 CAW SDK |
| **Telegram 通知** | 🔶 部分完成 | 代码存在（`telegram_bot.py`），Demo 中以终端日志替代 |
| **Sepolia 部署** | 🔴 未完成 | 当前在 Anvil 本地测试网运行，合约字节码已验证 |

> **诚实说明：** 本次 Demo 展示的是本地测试网闭环。Z.AI 感知层和 Cobo CAW SDK 因外部依赖（API Key、Access）尚未到位，使用 Mock。核心链上逻辑（SpendingLimitModule + Safe{Wallet} Session Key）已真实跑通。

---

## 附录：完整验证命令

```bash
# 启动 Anvil
anvil --chain-id 31337

# 部署合约（Foundry 脚本）
cd src && forge script script/CatAgentDeploy.s.sol:CatAgentDeploy --rpc-url http://127.0.0.1:8545 --broadcast

# 运行最小闭环
python run_minimal_loop.py

# 交易回执
cast receipt 0x70d3db2fac2ced63a6ba8263b0fa290dd1d9ef5b4778e7a68014768d0c0accaa --rpc-url http://127.0.0.1:8545

# 日限额查询
cast call 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0 \
  "getDailyRemaining()(uint256)" \
  --from 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 \
  --rpc-url http://127.0.0.1:8545

# SLM 存储读取
cast storage 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0 1 --rpc-url http://127.0.0.1:8545
```

---

## 关键设计决策备注（Q&A）

**Q: 为什么用 Safe{Wallet} 而不是普通 EOA？**
A: Safe{Wallet} 支持 Session Key（会话密钥）——Agent 持有的是受限权限的子密钥，主人可随时撤销。这比 EOA 更适合"委托权限给 AI Agent"的场景。

**Q: SpendingLimitModule 为什么用 `transferFrom` 而不是 `transfer`？**
A: Cat Safe 的 USDC 通过 `approve + transferFrom` 授权给 SLM 合约。这样 SLM 可以控制转账逻辑（检查限额），而 Cat Safe 无需将 USDC"存入" SLM。

**Q: 日限额重置为什么用 `block.timestamp / 86400`？**
A: 简化实现。最简单的方案是"按自然日重置"。生产环境应使用 Chainlink Time Oracle 防止验证者 manipulation。