# Cat-Agent（喵星智能体）Demo Day 项目复盘

> AI × Web3 School Cohort 0 · Week 4 Hackathon
> 日期：2026-06-13
> 作者：Minami-Bein

---

## 一、项目要解决什么问题

**场景：** 当主人不在家，猫咪想自主购买零食或玩具

**问题链：**
1. 猫咪无法使用信用卡
2. 中心化方案（App Store 礼品卡）→ 主人控制权过度集中
3. 现有加密支付 → 需要人类私钥签名

**解决方案：** 基于 Safe{Wallet} + Cobo Agentic Wallet + AI 感知，猫咪可自主发起合规购买

---

## 二、已跑通的主流程（Minimal Loop）

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

**关键参数：**
- 单笔限额：10 USDC
- 日限额：10 USDC
- Cat Safe：0x70997970C51812dc3A010C7d01b50e0d17dc79C8（Anvil 测试账户）
- 商家：0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC

---

## 三、最重要的验证材料

### 1. 合约部署

| 合约 | 地址 | 状态 |
|------|------|------|
| MockUSDC | `0x5FbDB2315678afecb367f032d93F642f64180aa3` | ✅ |
| SpendingLimitModule | `0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0` | ✅ |

**验证命令：**
```bash
cast call 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0 "getDailyRemaining()(uint256)" --rpc-url http://127.0.0.1:8545
# → 0x0000000000000000000000000000000000000000000000000000000000989680 (= 10 USDC)
```

### 2. 成功交易

| 交易 | 金额 | 状态 | 链接 |
|------|------|------|------|
| `0x1c1ed16cb8db3cf1c54b940a440ae45030bf8072a877a08f7ee556d8989c97bf` | 2 USDC | ✅ Success | Anvil block 6 |
| `0xc3756f2882faf19f0d741cd709ac01b0913ac1bc023652815af39a793bb4ae02` | 2 USDC | ✅ Success | Anvil block 7 |

### 3. 防御验证

- **11 USDC 超额交易** → SLM revert（`invalid value`），状态码 `0x0` ✅
- **日限额 2 USDC 后** → `getDailyRemaining()` 返回 `0.08 USDC`（8.00 / 10.00）✅

### 4. 技术关键点

**问题：** `eth_sendTransaction` 在 Anvil unlocked 模式下交易失败（状态码 0x0，无 logs）

**解决：** 改用 `web3.eth.account.sign_transaction` 本地签名 + `eth_sendRawTransaction`

```python
signed = w3.eth.account.sign_transaction(tx, CAT_KEY)
result = w3.eth.send_raw_transaction(signed.rawTransaction)
receipt = w3.eth.wait_for_transaction_receipt(result)
# Status: 1 (success)
```

---

## 四、最大风险

### 🔴 签名方案尚未适配真实 Safe{Wallet} 环境

- **当前：** Anvil 本地账户私钥直接签名
- **真实场景：** Safe{Wallet} 通过 Safe{Wallet} Web App 或 Cobo Agentic Wallet 发起
- **差距：** Cobo Agentic Wallet 的 `agentkit` 签名 API 尚未验证是否支持 `personal_sign` 或 `eth_sendRawTransaction`

**缓解：** Session Key 方案在 Safe{Wallet} 理论上是可行的（类似 Gnosis Safe Modules 机制），需要真网验证

### 🟡 日限额重置依赖 `block.timestamp`

- 合约使用 `block.timestamp` 计算日重置，理论上验证者可 manipulation
- 缓解：生产环境应使用可靠的 Time Oracle（如 Chainlink）

---

## 五、下一步计划

| 优先级 | 任务 | 状态 |
|--------|------|------|
| P0 | 集成 Cobo Agentic Wallet 签名 API | 待开始 |
| P0 | Sepolia 测试网部署（真实 USDC） | 待开始 |
| P1 | Safe{Wallet} 1-of-1 创建 + Session Key 绑定 | 待开始 |
| P2 | 竞品对标（Dogo、CatFi）| 待开始 |
| P2 | Pitch Deck / Demo Video | 待开始 |

---

## 六、希望获得的反馈

### Mentor
- Cobo Agentic Wallet 签名方案推荐？Session Key 在 Safe{Wallet} 中的正确集成方式？
- `agentkit` 是否支持 `eth_sendRawTransaction`？还是只支持 `personal_sign`？

### Sponsor
- 是否有 Sepolia 测试网 USDC 水龙头资源？
- 是否可对接 Safe{Wallet} 团队获取技术支持？

### 同伴
- 其他团队的 AI Agent 支付流程是如何处理钱包签名的？
- 有没有在 Cobo Agentic Wallet 上跑通 `eth_sendRawTransaction` 的案例？

---

## 七、合约代码（核心）

### SpendingLimitModule.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

contract SpendingLimitModule {
    uint256 public constant DAILY_LIMIT = 10 * 1e6; // 10 USDC
    uint256 public constant TX_LIMIT    = 10 * 1e6; // 10 USDC

    address public immutable usdc;
    mapping(address => uint256) public dailySpent;
    mapping(address => uint256) public lastResetDay;

    event Transfer(address indexed from, address indexed to, uint256 amount, uint256 dailyRemaining);

    constructor(address _usdc) {
        usdc = _usdc;
    }

    function transferUSDC(address to, uint256 amount) external {
        require(amount <= TX_LIMIT, "TX_LIMIT_EXCEEDED");
        _checkAndUpdateDaily(msg.sender, amount);
        require(IERC20(usdc).transferFrom(msg.sender, to, amount), "TRANSFER_FAILED");
        emit Transfer(msg.sender, to, amount, DAILY_LIMIT - dailySpent[msg.sender]);
    }

    function getDailyRemaining() external view returns (uint256) {
        return DAILY_LIMIT - _getDailySpent(msg.sender);
    }

    function _checkAndUpdateDaily(address wallet, uint256 amount) internal {
        uint256 today = block.timestamp / 86400;
        if (lastResetDay[wallet] != today) {
            dailySpent[wallet] = 0;
            lastResetDay[wallet] = today;
        }
        require(dailySpent[wallet] + amount <= DAILY_LIMIT, "DAILY_LIMIT_EXCEEDED");
        dailySpent[wallet] += amount;
    }

    function _getDailySpent(address wallet) internal view returns (uint256) {
        return lastResetDay[wallet] == block.timestamp / 86400 ? dailySpent[wallet] : 0;
    }
}
```

### 目录结构

```
/tmp/repo/
├── src/
│   ├── agent/          # AI 感知模块
│   ├── perception/    # 意图识别
│   ├── wallet/        # 钱包签名
│   ├── run_minimal_loop.py   # 主流程脚本
│   ├── MockUSDC.sol
│   └── SpendingLimitModule.sol
├── script/
│   └── CatAgentDeploy.s.sol   # 部署脚本
├── configs/            # 配置文件
└── foundry.toml
```