# 个人A股量化交易系统

## 项目简介

基于Python开发的个人A股量化交易系统，支持策略开发、回测、实盘交易和风险管理。

**核心特点**：
- 🎯 针对中低频交易（日线/小时线级别）
- 🛡️ 完善的风险控制机制（支持20-40%回撤容忍度）
- 🔌 集成华泰证券QMT/MiniQMT接口
- 📊 完整的回测和性能评估体系
- 🔄 自动化交易执行与人工干预相结合

## 技术架构

详细架构设计请参考：[ARCHITECTURE.md](./ARCHITECTURE.md)

### 系统模块

1. **数据服务层** - 历史数据获取、实时行情订阅
2. **策略引擎层** - 策略开发框架、技术指标库
3. **回测引擎层** - 历史回测、性能评估
4. **风险管理层** - 仓位管理、止损止盈、风险监控
5. **交易执行层** - QMT接口封装、订单管理
6. **监控告警层** - 实时监控、告警机制
7. **策略调度层** - 任务调度、流程编排

## 快速开始

### 1. 环境准备

```bash
# 创建虚拟环境
conda create -n quant_trading_venv python=3.12
conda activate quant_trading_venv
# 或 venv\Scripts\activate  # Windows

# 安装依赖
pip install -r requirements.txt
```

### 2. 配置系统

```bash
# 复制配置文件
cp config/config.example.yaml config/config.yaml

# 编辑配置文件，填写必要信息
# - 数据源配置（akshare/tushare）
# - 数据库配置
# - QMT账户信息（实盘交易时）
# - 风险参数
```

### 3. 获取数据

```bash
# 获取单只股票数据
python scripts/fetch_data.py --code 000001 --start-date 20230101

# 获取多只股票数据
python scripts/fetch_data.py --code 000001,600000 --start-date 20230101
```

### 4. 运行回测

```bash
# 运行策略回测
python scripts/run_backtest.py --strategy ma_cross --code 000001 --start-date 20230101 --output results/
```

## 项目结构

```
quant_trading/
├── config/              # 配置文件
│   ├── config.example.yaml
│   └── config.yaml      # 实际配置（不提交git）
├── src/                 # 源代码
│   ├── data/           # 数据服务层
│   ├── strategy/       # 策略引擎层
│   ├── backtest/       # 回测引擎层
│   ├── risk/           # 风险管理层
│   ├── trade/          # 交易执行层
│   ├── monitor/        # 监控告警层
│   ├── scheduler/      # 策略调度层
│   └── utils/          # 工具函数
├── strategies/          # 策略文件
├── scripts/             # 脚本工具
├── data/                # 数据目录
│   ├── raw/            # 原始数据
│   ├── processed/      # 处理后的数据
│   └── database/       # 数据库文件
├── logs/                # 日志文件
├── results/             # 回测结果
├── tests/               # 测试文件
├── ARCHITECTURE.md      # 架构设计文档
└── requirements.txt     # 依赖包
```

## 开发路径

详细开发指南请参考：[DEVELOPMENT_GUIDE.md](./DEVELOPMENT_GUIDE.md)

### 阶段概览

1. **基础建设**（2-3周）- 数据获取、存储、基础框架
2. **策略开发**（2-3周）- 策略开发、回测验证
3. **风险控制**（1-2周）- 风险管理模块
4. **交易接口**（2-3周）- QMT集成、实盘测试
5. **监控调度**（1-2周）- 任务调度、监控告警
6. **优化迭代**（持续）- 策略优化、系统完善

## 风险提示

⚠️ **重要提示**：

1. **投资有风险**：量化交易不能保证盈利，可能面临本金损失
2. **充分测试**：实盘交易前必须充分回测和模拟交易
3. **风险控制**：严格遵守风险控制规则，设置止损
4. **合规交易**：遵守相关法律法规和券商API使用规范
5. **个人责任**：本系统仅供学习研究，实盘交易风险自担

## 许可证

MIT License

## 联系方式

如有问题或建议，欢迎提交Issue。
