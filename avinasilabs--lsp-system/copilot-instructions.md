## lsp-system

> - HealthKit数据使用timestamp with time zone存储

# LSP积分系统项目规则

## 项目特定模式

### 数据处理
- HealthKit数据使用timestamp with time zone存储
- 所有健康数据的value字段统一存储为字符串，计算时再转换
- 批量数据导入使用1000条为一批，避免内存溢出

### 积分计算规则
- 积分计算必须是幂等的，相同输入产生相同输出
- 使用配置文件定义规则，不要硬编码积分值
- 每个维度的计算器都继承自DimensionCalculator基类

### 数据库操作
- 使用连接池，不要直接创建连接
- 所有数据库操作都要有适当的错误处理
- 使用参数化查询防止SQL注入

## 用户偏好

### 代码风格
- 使用类型注解提高代码可读性
- 函数和类都需要docstring
- 使用中文注释解释业务逻辑

### 项目结构
- 业务逻辑放在src/core/
- API相关代码放在src/api/
- 数据库模型放在src/models/
- 独立脚本放在scripts/

### 开发流程
- 先更新Memory Bank文档再开始编码
- 重要变更需要更新activeContext.md
- 完成功能后更新progress.md

## 关键实现路径

### 添加新的维度计算器
1. 在src/core/calculators/创建新文件
2. 继承DimensionCalculator基类
3. 实现calculate()和get_rules()方法
4. 在配置文件中添加规则定义
5. 更新测试用例

### 处理新的HealthKit数据类型
1. 检查数据库中是否已有该类型
2. 在数据需求文档中更新
3. 创建对应的计算逻辑
4. 添加数据验证

### API端点开发
1. 在src/api/v1/创建端点文件
2. 使用FastAPI的依赖注入
3. 添加请求/响应模型
4. 实现适当的错误处理
5. 更新API文档

## 已知挑战

### 数据一致性
- 问题：用户可能多次导入相同数据
- 解决：使用upsert操作，以start_date和type为唯一键

### 性能优化
- 问题：历史数据计算耗时
- 解决：使用批量计算、缓存中间结果、异步处理

### 规则复杂性
- 问题：连锁反应规则难以实现
- 解决：使用事件驱动架构，规则引擎处理复杂逻辑

## 工具使用模式

### 数据库查询
```python
# 使用连接池
from src.db.postgresql import POSTGRES_POOL

# 查询示例
data = POSTGRES_POOL.select_data(
    table_name="health_metric",
    conditions="type = %s AND start_date >= %s",
    params=(data_type, start_date)
)
```

### 配置管理
```python
# 使用pydantic管理配置
from pydantic import BaseSettings

class Config(BaseSettings):
    class Config:
        env_file = ".env"
```

### 错误处理
```python
# 统一的错误处理模式
try:
    result = calculate_score(data)
except DataValidationError as e:
    logger.error(f"数据验证失败: {e}")
    return {"error": "invalid_data"}
except Exception as e:
    logger.exception("未知错误")
    return {"error": "internal_error"}
```

## 项目演进记录

### 2025-07-18
- 项目初始化，建立基础架构
- 导入HealthKit测试数据
- 创建Memory Bank文档体系
- 实现第一版积分计算系统
- 添加多用户支持（user_id）
- 实现FastAPI服务器
- 创建API测试脚本
- 实现可选认证系统（JWT）
- 支持认证/非认证双模式

### 2025-07-19
- 待实现：真实用户认证（密码验证）
- 待实现：积分持久化存储
- 待实现：更多维度计算器
- 待实现：连锁反应机制

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AvinasiLabs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
