## python-specific

> Python语言特定规则和最佳实践


# Python语言特定规则

基于awesome-cursorrules的Python最佳实践，专门针对Django项目优化。

## 核心原则

- **类型注解**: 为所有函数和类添加类型注解，包括明确的返回类型
- **文档字符串**: 遵循PEP 257规范，为所有函数和类添加详细文档字符串
- **模块化设计**: 清晰的目录结构，分离模型、服务、控制器和工具函数
- **错误处理**: 健壮的错误处理和日志记录，包括上下文捕获
- **测试覆盖**: 使用pytest进行综合测试
- **代码风格**: 使用Ruff保持代码风格一致性

## 类型注解要求

```python
from typing import List, Dict, Optional, Union, Any
from django.db.models import QuerySet
from django.http import HttpRequest, HttpResponse

def example_function(
    param1: str, 
    param2: Optional[int] = None
) -> Dict[str, Any]:
    """
    示例函数的文档字符串。
    
    Args:
        param1: 字符串参数描述
        param2: 可选的整数参数描述
        
    Returns:
        包含结果的字典
        
    Raises:
        ValueError: 当参数无效时
    """
    pass
```

## 测试要求

- 使用pytest或pytest插件（不使用unittest）
- 所有测试必须有类型注解
- 测试文件放在`./tests`目录下
- 测试函数必须包含文档字符串

```python
from typing import TYPE_CHECKING
import pytest
from django.test import TestCase

if TYPE_CHECKING:
    from _pytest.capture import CaptureFixture
    from _pytest.fixtures import FixtureRequest
    from _pytest.logging import LogCaptureFixture
    from _pytest.monkeypatch import MonkeyPatch
    from pytest_mock.plugin import MockerFixture

def test_example_function() -> None:
    """测试示例函数的功能。"""
    pass
```

## Django特定规则

- 使用Django的最新稳定版本
- 遵循Django的最佳实践和约定
- 使用Django的ORM进行数据库操作
- 实现适当的权限检查和认证
- 使用Django的中间件和信号系统

## 代码组织

```
apps/
  tools/
    models/
      __init__.py
      base_models.py
      chat_models.py
      # ... 其他模型文件
    views/
      __init__.py
      # ... 视图文件
    services/
      __init__.py
      # ... 服务文件
    utils/
      __init__.py
      # ... 工具文件
tests/
  unit/
  integration/
  e2e/
```

## 环境变量管理

```python
import os
from typing import Optional

def get_env_variable(key: str, default: Optional[str] = None) -> Optional[str]:
    """安全获取环境变量。"""
    value = os.getenv(key, default)
    if not value and default is None:
        raise ValueError(f"环境变量 {key} 未设置")
    return value
```

## 错误处理模式

```python
import logging
from typing import Optional

logger = logging.getLogger(__name__)

def safe_operation() -> Optional[Dict[str, Any]]:
    """安全执行操作的示例。"""
    try:
        # 执行操作
        result = perform_operation()
        return result
    except SpecificException as e:
        logger.error(f"操作失败: {e}", exc_info=True)
        return None
    except Exception as e:
        logger.critical(f"意外错误: {e}", exc_info=True)
        raise
```

## 性能优化

- 使用`select_related`和`prefetch_related`优化数据库查询
- 实现适当的缓存策略
- 使用异步操作处理长时间运行的任务
- 监控和记录性能指标

## 安全最佳实践

- 使用环境变量管理敏感信息
- 实现适当的输入验证和清理
- 使用Django的安全中间件
- 定期更新依赖包

---
> Source: [shinytsing/modeshift_django](https://github.com/shinytsing/modeshift_django) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
