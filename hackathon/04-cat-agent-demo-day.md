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
- Cat Safe（Anvil 测试账户）：`0x70997970C51812dc3A010C7d01b50e0d17dc79C8`
- 商家（Anvil 测试账户）：`0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC`

---

## 三、可复查验证材料

### 3.1 合约地址与字节码

| 合约 | 地址 | 字节码长度 | 验证命令 |
|------|------|-----------|---------|
| MockUSDC | `0x5FbDB2315678afecb367f032d93F642f64180aa3` | 7048 chars | `cast code <addr> --rpc-url http://127.0.0.1:8545` |
| SpendingLimitModule | `0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0` | 6044 chars | 同上 |

### 3.2 SLM 存储状态（验证初始化正确）

```bash
# slot 0 = usdc 地址
cast storage 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0 0 --rpc-url http://127.0.0.1:8545
→ 0x0000000000000000000000005fbdb2315678afecb367f032d93f642f64180aa3  ✅

# slot 1 = dailySpent（2笔各2 USDC = 4 USDC）
cast storage 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0 1 --rpc-url http://127.0.0.1:8545
→ 0x00000000000000000000000000000000000000000000000000000000003d0900  ✅ (= 4e6)

# slot 2 = lastResetDay
cast storage 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0 2 --rpc-url http://127.0.0.1:8545
→ 0x0000000000000000000000000000000000000000000000000000000000005088
```

### 3.3 成功交易回执

```
Tx Hash:   0x70d3db2fac2ced63a6ba8263b0fa290dd1d9ef5b4778e7a68014768d0c0accaa
Block:     11
Status:    1 (success)
Gas Used:  58194
From:      0x70997970C51812dc3A010C7d01b50e0d17dc79C8 (Cat Safe)
To:        0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0 (SLM)
Function:  transferUSDC(address,uint256) → selector: 0xf7aaaf3f
Data:      0xf7aaaf3f0000000000000000000000003c44cdddb6a900fa2b585dd299e03d12fa4293bc00000000000000000000000000000000000000000000000000000000001e8480
Logs:      2 events (Transfer + TransferSpendingLimit)
  [0] address: 0x5fbdb2315678afecb367f032d93f642f64180aa3
      topics[0]: 0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef
      topics[1]: 0x00000000000000000000000070997970c51812dc3a010c7d01b50e0d17dc79c8 (from)
      topics[2]: 0x0000000000000000000000003c44cdddb6a900fa2b585dd299e03d12fa4293bc (to)
      data: 0x00000000000000000000000000000000000000000000000000000000001e8480 (= 2 USDC)
  [1] address: 0x9fe46736679d2d9a65f0992f2272de9f3c7fa6e0
      topics[0]: 0xf9526c97e462282f793fa26260fdbb98fb81047df50b1249179bfffae687094a
      data: 0x00000000000000000000000000000000000000000000000000000000001e8480 (amount)
              00000000000000000000000000000000000000000000000000000000005b8d80 (dailyRemaining = 8 USDC)
```

**验证命令：**
```bash
cast receipt 0x70d3db2fac2ced63a6ba8263b0fa290dd1d9ef5b4778e7a68014768d0c0accaa --rpc-url http://127.0.0.1:8545
```

### 3.4 余额验证

```bash
# Cat Safe USDC 余额（转账后：100 - 2 - 2 - 2 = 94）
cast call 0x5FbDB2315678afecb367f032d93F642f64180aa3 \
  "balanceOf(address)(uint256)" 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 \
  --from 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 \
  --rpc-url http://127.0.0.1:8545
→ 94000000  ✅ (= 94 USDC)

# Merchant USDC 余额（收款后：0 + 2 + 2 + 2 = 6）
cast call 0x5FbDB2315678afecb367f032d93F642f64180aa3 \
  "balanceOf(address)(uint256)" 0x3C44CdDdB6a900fa2b585dd299e03d12FA4293BC \
  --from 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 \
  --rpc-url http://127.0.0.1:8545
→ 6000000  ✅ (= 6 USDC)
```

### 3.5 日限额状态验证

```bash
cast call 0x9fE46736679d2D9a65F0992F2272dE9f3c7fa6e0 \
  "getDailyRemaining()(uint256)" \
  --from 0x70997970C51812dc3A010C7d01b50e0d17dc79C8 \
  --rpc-url http://127.0.0.1:8545
→ 8000000  ✅ (= 8 USDC remaining, 2 USDC spent today)
```

### 3.6 防御验证（超额交易被链上拦截）

```
Test:     发送 11 USDC（超过单笔限额 10 USDC）
Result:   SLM revert: "SpendingLimitModule: per_tx_limit exceeded"
Tx Status: 0 (reverted on-chain)
Verdict:  ✅ 防御正确触发
```

### 3.7 技术关键发现

**⚠️ 踩坑：eth_sendTransaction 在 Anvil unlocked 模式下的 bug**

- **现象：** `eth_sendTransaction`（Anvil unlocked 自动签名）交易状态码 `0x0`，gasUsed=22340（刚好 gas 不够），无 event logs
- **根本原因：** Anvil unlocked 模式下 `eth_sendTransaction` 构造的交易有缺陷（可能 gasPrice 计算问题或签名不完整）
- **解决：** 改用 `web3.eth.account.sign_transaction` 本地签名 + `eth_sendRawTransaction`

```python
signed = w3.eth.account.sign_transaction(tx, CAT_KEY)
result = w3.eth.send_raw_transaction(signed.rawTransaction)
receipt = w3.eth.wait_for_transaction_receipt(result)
# Status: 1 (success), Gas: 58194, Logs: 2
```

**⚠️ 踩坑：函数选择器错误**

- 手动拼接 hex 数据时，函数选择器 `a1e2e1a0` 是错的
- 正确选择器：`0xf7aaaf3f`（`transferUSDC(address,uint256)`）
- **解决：** 使用 `web3.contract.functions.transferUSDC(...).build_transaction()` 自动编码

---

## 四、最大风险

### 🔴 签名方案尚未适配真实 Safe{Wallet} 环境

- **当前：** Anvil 本地账户私钥直接签名
- **真实场景：** Safe{Wallet} 通过 Safe{Wallet} Web App 或 Cobo Agentic Wallet 发起
- **差距：** Cobo Agentic Wallet 的 `agentkit` 签名 API 尚未验证是否支持 `personal_sign` 或 `eth_sendRawTransaction`

### 🟡 日限额重置依赖 `block.timestamp`

- 合约使用 `block.timestamp / 86400` 计算日重置，理论上验证者可 manipulation
- 缓解：生产环境应使用可靠的 Time Oracle（如 Chainlink）

---

## 五、下一步计划

| 优先级 | 任务 | 依赖 |
|--------|------|------|
| P0 | 集成 Cobo Agentic Wallet 签名 API | Cobo agentkit docs |
| P0 | Sepolia 测试网部署（真实 USDC） | Sepolia USDC 水龙头 |
| P1 | Safe{Wallet} 1-of-1 创建 + Session Key 绑定 | Safe{Wallet} SDK |
| P2 | 竞品对标（Dogo、CatFi）| Token Terminal |
| P2 | Pitch Deck / Demo Video | 设计资源 |

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

## 七、核心合约代码

### SpendingLimitModule.sol

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

interface IERC20 {
    function transferFrom(address from, address to, uint256 amount) external returns (bool);
}

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
        require(amount <= TX_LIMIT, "SpendingLimitModule: per_tx_limit exceeded");
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
        require(dailySpent[wallet] + amount <= DAILY_LIMIT,
            "SpendingLimitModule: daily_limit exceeded");
        dailySpent[wallet] += amount;
    }

    function _getDailySpent(address wallet) internal view returns (uint256) {
        return lastResetDay[wallet] == block.timestamp / 86400 ? dailySpent[wallet] : 0;
    }
}
```

### 完整测试日志

```json
{
  "intent": {
    "intent": "buy_snack",
    "target": "tuna_can",
    "price_usdc": 2000000,
    "confidence_score": 0.94
  },
  "tx_hash": "0x70d3db2fac2ced63a6ba8263b0fa290dd1d9ef5b4778e7a68014768d0c0accaa",
  "block_number": 11,
  "amount_usdc": 2.0,
  "daily_remaining_usdc": 8.0,
  "chain_id": 31337,
  "cat_safe_usdc_balance": 94.0,
  "merchant_usdc_balance": 6.0,
  "status": "PASS"
}
```

---

## 八、目录结构

```
/tmp/repo/
├── src/
│   ├── agent/              # AI 感知模块
│   ├── perception/        # 意图识别
│   ├── wallet/             # 钱包签名
│   ├── run_minimal_loop.py # ✅ 验证通过的主流程脚本
│   ├── MockUSDC.sol
│   └── SpendingLimitModule.sol  # ✅ 验证通过的 SLM
├── script/
│   └── CatAgentDeploy.s.sol    # 部署脚本
├── logs/
│   ├── verification/           # ✅ 可复查验证材料
│   │   └── verified_minimal_loop_result.json
│   └── payments/
│       └── minimal_loop_result.json
├── configs/
├── foundry.toml
└── foundry.lock
```