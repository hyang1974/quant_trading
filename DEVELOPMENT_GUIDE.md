# 开发实践指南

## 一、开发环境搭建

### 1.1 Python环境

```bash
# 推荐使用Python 3.9+
python3 --version

# 创建虚拟环境
python3 -m venv venv
source venv/bin/activate  # Linux/Mac
# 或 venv\Scripts\activate  # Windows

# 安装依赖
pip install -r requirements.txt
```

### 1.2 QMT环境配置

1. **下载安装QMT客户端**
   - 访问华泰证券官网下载QMT或MiniQMT
   - 安装并登录账户

2. **配置Python环境**
   - QMT通常自带Python环境
   - 或配置系统Python环境指向QMT的Python

3. **获取API文档**
   - 联系华泰证券获取QMT Python API文档
   - 了解接口调用方式

### 1.3 数据源配置

**akshare（推荐新手）**：
- 免费使用，无需配置
- 数据质量较好，适合回测

**tushare pro（推荐生产）**：
- 需要注册：https://tushare.pro/register
- 获取token后配置到config.yaml
- 数据质量高，更新及时

## 二、开发阶段规划

### Phase 1: 基础建设（2-3周）

#### 目标
建立数据获取、存储和基础框架

#### 任务清单
- [ ] 实现数据获取模块（akshare/tushare）
- [ ] 实现数据存储模块（SQLite/PostgreSQL）
- [ ] 实现基础策略框架
- [ ] 实现简单回测引擎
- [ ] 编写数据获取脚本

#### 验收标准
- 能够成功获取并存储A股历史数据
- 能够运行简单的策略回测
- 回测结果可视化正常

---

### Phase 2: 策略开发（2-3周）

#### 目标
开发2-3个基础策略，完善回测功能

#### 任务清单
- [ ] 实现技术指标库（MA、MACD、RSI等）
- [ ] 开发双均线交叉策略
- [ ] 开发RSI超买超卖策略
- [ ] 开发多因子选股策略（可选）
- [ ] 实现参数优化功能
- [ ] 完善回测报告（可视化、指标计算）

#### 策略开发示例

```python
# strategies/ma_cross_strategy.py
from src.strategy.base_strategy import BaseStrategy
from src.strategy.indicators import calculate_ma
import pandas as pd

class MACrossStrategy(BaseStrategy):
    def __init__(self, short_period=5, long_period=20):
        params = {'short_period': short_period, 'long_period': long_period}
        super().__init__(name="双均线交叉", params=params)
    
    def generate_signals(self, data: pd.DataFrame) -> pd.DataFrame:
        # 计算均线
        data['ma_short'] = calculate_ma(data, self.params['short_period'])
        data['ma_long'] = calculate_ma(data, self.params['long_period'])
        
        # 生成信号
        data['signal'] = 0
        # 金叉买入
        buy_condition = (
            (data['ma_short'].shift(1) <= data['ma_long'].shift(1)) &
            (data['ma_short'] > data['ma_long'])
        )
        data.loc[buy_condition, 'signal'] = 1
        
        # 死叉卖出
        sell_condition = (
            (data['ma_short'].shift(1) >= data['ma_long'].shift(1)) &
            (data['ma_short'] < data['ma_long'])
        )
        data.loc[sell_condition, 'signal'] = -1
        
        return data
```

#### 验收标准
- 至少2个策略能够正常运行回测
- 回测指标计算正确（收益率、回撤、夏普比率等）
- 能够进行参数优化

---

### Phase 3: 风险控制（1-2周）

#### 目标
实现完善的风险管理模块

#### 任务清单
- [ ] 实现仓位管理（单股、总仓位、行业分散）
- [ ] 实现止损止盈机制（固定、移动止损）
- [ ] 实现风险监控（实时风险度计算）
- [ ] 实现异常处理（停牌、退市、异常波动）
- [ ] 集成到回测引擎

#### 风险控制示例

```python
# src/risk/risk_manager.py
class RiskManager:
    def __init__(self, config):
        self.max_single_position = config['max_single_position']
        self.max_total_position = config['max_total_position']
        self.stop_loss_pct = config['stop_loss_pct']
        # ...
    
    def check_position_limit(self, symbol, quantity, price, current_positions):
        """检查仓位限制"""
        new_position_value = quantity * price
        total_value = sum(p['value'] for p in current_positions.values())
        
        # 检查单股仓位
        if new_position_value / total_value > self.max_single_position:
            return False, "超过单股最大仓位限制"
        
        # 检查总仓位
        if (total_value + new_position_value) / total_value > self.max_total_position:
            return False, "超过总仓位限制"
        
        return True, "通过"
    
    def check_stop_loss(self, position, current_price):
        """检查止损"""
        cost_price = position['cost_price']
        loss_pct = (current_price - cost_price) / cost_price
        
        if loss_pct <= self.stop_loss_pct:
            return True, "触发止损"
        
        return False, "未触发"
```

#### 验收标准
- 所有风险规则能够正确执行
- 回测中能够正确应用风险控制
- 风险监控实时计算准确

---

### Phase 4: 交易接口（2-3周）

#### 目标
集成QMT接口，实现实盘交易

#### 任务清单
- [ ] 研究QMT Python API文档
- [ ] 实现QMT接口封装
- [ ] 实现订单管理系统
- [ ] 实现模拟交易功能
- [ ] 小资金实盘测试（建议<1万）

#### QMT接口示例结构

```python
# src/trade/qmt_trader.py
class QMTTrader:
    """
    华泰证券QMT交易接口封装
    注意：具体实现需要参考QMT官方API文档
    """
    def __init__(self, account_id, password, client_path):
        # 连接QMT
        # 具体实现根据QMT API文档
        pass
    
    def connect(self):
        """连接QMT"""
        pass
    
    def get_balance(self):
        """获取资金"""
        # 调用QMT API
        pass
    
    def get_position(self):
        """获取持仓"""
        pass
    
    def buy(self, symbol, price, quantity):
        """买入"""
        # 1. 风险检查
        # 2. 调用QMT下单接口
        # 3. 记录订单
        pass
    
    def sell(self, symbol, price, quantity):
        """卖出"""
        pass
    
    def cancel_order(self, order_id):
        """撤单"""
        pass
```

#### 验收标准
- 能够成功连接QMT
- 能够查询资金和持仓
- 模拟交易功能正常
- 小资金实盘测试通过

---

### Phase 5: 监控调度（1-2周）

#### 目标
实现任务调度和监控告警

#### 任务清单
- [ ] 实现任务调度系统（APScheduler）
- [ ] 实现监控模块（账户、持仓、风险）
- [ ] 实现告警机制（邮件/微信）
- [ ] 实现日志系统
- [ ] 系统集成测试

#### 任务调度示例

```python
# src/scheduler/task_scheduler.py
from apscheduler.schedulers.blocking import BlockingScheduler

class TaskScheduler:
    def __init__(self):
        self.scheduler = BlockingScheduler()
    
    def schedule_data_update(self):
        """调度数据更新任务"""
        self.scheduler.add_job(
            func=self.update_data,
            trigger='cron',
            hour=18,
            minute=0,
            id='data_update'
        )
    
    def schedule_strategy_calculate(self):
        """调度策略计算任务"""
        self.scheduler.add_job(
            func=self.calculate_strategy,
            trigger='cron',
            hour=9,
            minute=0,
            day_of_week='mon-fri',
            id='strategy_calculate'
        )
```

#### 验收标准
- 定时任务能够正常执行
- 监控数据准确
- 告警能够正常触发
- 日志记录完整

---

### Phase 6: 优化迭代（持续）

#### 目标
持续优化策略和系统

#### 任务清单
- [ ] 策略参数优化
- [ ] 策略失效检测
- [ ] 系统性能优化
- [ ] 功能完善
- [ ] Bug修复

## 三、关键开发要点

### 3.1 数据质量

- **数据清洗**：处理停牌、复权、异常值
- **数据验证**：检查数据完整性、准确性
- **数据备份**：定期备份重要数据

### 3.2 策略开发

- **避免未来函数**：不使用未来数据
- **考虑滑点和手续费**：回测中要真实模拟
- **参数过拟合**：避免过度优化参数
- **样本外测试**：保留部分数据用于验证

### 3.3 风险控制

- **严格执行**：风险规则必须严格执行
- **实时监控**：实时计算风险指标
- **异常处理**：完善的异常情况处理

### 3.4 实盘交易

- **充分测试**：回测+模拟交易至少1个月
- **小资金开始**：实盘初期使用小资金
- **逐步增加**：验证稳定后逐步增加资金
- **人工监控**：初期需要人工密切监控

## 四、测试建议

### 4.1 单元测试

```bash
# 运行测试
pytest tests/

# 测试覆盖率
pytest --cov=src tests/
```

### 4.2 回测验证

- **历史数据回测**：至少1-2年历史数据
- **不同市场环境**：牛市、熊市、震荡市
- **参数稳定性**：参数微调后结果是否稳定

### 4.3 模拟交易

- **至少1个月**：充分验证系统稳定性
- **真实市场环境**：使用真实行情数据
- **完整流程**：数据→策略→风控→交易

## 五、常见问题

### Q1: QMT接口如何获取？
A: 联系华泰证券客服，通常需要满足资金门槛（如50万）才能申请。

### Q2: 策略回测很好但实盘表现差？
A: 可能原因：
- 回测存在未来函数
- 滑点和手续费估计不足
- 市场环境变化
- 数据质量问题

### Q3: 如何选择股票池？
A: 可以基于：
- 市值筛选（如>50亿）
- 行业筛选
- 技术指标筛选
- 基本面筛选（PE、PB等）

### Q4: 如何应对策略失效？
A: 
- 定期回测验证策略有效性
- 设置策略失效检测机制
- 准备多个策略组合
- 根据市场环境调整策略

## 六、学习资源

### 书籍推荐
- 《量化交易：如何建立自己的算法交易》
- 《Python金融大数据分析》
- 《量化投资：策略与技术》

### 在线资源
- 聚宽量化平台（学习策略开发）
- 米筐量化平台
- QuantConnect
- 华泰证券QMT文档

### 技术指标学习
- 技术分析基础
- 各种技术指标原理和应用

---

**祝开发顺利！记住：量化交易是概率游戏，风险控制永远是第一位的。**
