# Cat-Agent Repo Skeleton

```
cat-agent/
├── README.md
├── LICENSE
├── .github/
│   └── workflows/
│       └── ci.yml
├── docs/
│   ├── architecture.png
│   ├── cat-agent-flow.excalidraw
│   ├── caaw-warrant-design.md
│   └── x402-payment-flow.md
├── src/
│   ├── __init__.py
│   ├── main.py                    # Agent 入口
│   ├── perception/
│   │   ├── __init__.py
│   │   ├── audio_analyzer.py       # 音频分析（频率/时长）
│   │   ├── vision_analyzer.py      # 视觉分析（姿态/表情）
│   │   └── multimodal_fusion.py    # 多模态融合 → 意图置信度
│   ├── intent/
│   │   ├── __init__.py
│   │   ├── intent_classifier.py    # 意图分类（要零食/要玩/要出门）
│   │   ├── confidence_engine.py   # 置信度计算（两轮交互）
│   │   └── physical_feedback.py     # 物理反馈（激光笔/扬声器）
│   ├── wallet/
│   │   ├── __init__.py
│   │   ├── caaw_warrant.py         # CAW Warrant 读取与验证
│   │   ├── policy_engine.py        # Budget/Scope/Time Window 检查
│   │   ├── safe_wallet.py          # Safe{Wallet} Session Key 管理
│   │   └── x402_client.py          # x402 感知 HTTP Client
│   ├── blockchain/
│   │   ├── __init__.py
│   │   ├── usdc_transfer.py        # USDC 转账
│   │   └── tx_logger.py            # 链上交易记录
│   ├── learning/
│   │   ├── __init__.py
│   │   ├── rlpf.py                 # 宠物反馈强化学习
│   │   └── preference_tracker.py   # 口味偏好追踪
│   └── api/
│       ├── __init__.py
│       ├── app.py                  # FastAPI 主服务
│       ├── ws_endpoint.py          # WebSocket 实时推送
│       └── admin_panel.py          # 管理后台
├── hardware/
│   ├── cat_collar.py               # 宠物项圈固件（可选）
│   ├── laser_pointer.py            # 激光笔控制（树莓派）
│   └── speaker.py                  # 扬声器反馈
├── tests/
│   ├── __init__.py
│   ├── test_intent_classifier.py
│   ├── test_confidence_engine.py
│   ├── test_caaw_warrant.py
│   ├── test_policy_engine.py
│   └── test_x402_client.py
├── demo/
│   ├── cat_agent_demo.mp4          # 演示视频
│   ├── demo_script.md              # 演示台词
│   └── mock_iot.py                 # 模拟 IoT 输入（无硬件时用）
├── scripts/
│   ├── deploy_safe.py              # 部署 Safe{Wallet}
│   ├── register_caw.py             # 注册 CAW Warrant
│   └── setup_x402.py               # 配置 x402 受保护 API
├── notebooks/
│   ├── cat_behavior_analysis.ipynb  # 猫咪行为数据分析
│   └── caaw_policy_design.ipynb    # CAW 策略设计工具
├── configs/
│   ├── cat_configs.json            # 猫咪配置文件
│   ├── caaw_templates.json         # CAW Warrant 模板
│   └── product_catalog.json        # 商品目录
├── requirements.txt
├── pyproject.toml
└── .env.example
```

## 快速启动

```bash
# 克隆
git clone https://github.com/Minami-Bein/cat-agent.git
cd cat-agent

# 安装依赖
pip install -r requirements.txt

# 配置环境变量
cp .env.example .env
# 填入 Z.AI_API_KEY / COBO_CAW_KEY / SAFE_WALLET_ADDRESS

# 启动 Demo（模拟 IoT 输入，无需硬件）
python demo/mock_iot.py

# 启动 API
python src/api/app.py
```
