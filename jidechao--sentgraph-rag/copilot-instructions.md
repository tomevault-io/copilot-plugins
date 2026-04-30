## sentgraph-rag

> 本文档为 AI 编程助手提供 SentGraph 项目的开发规范、构建命令和代码风格指南。

# AGENTS.md - SentGraph 代码库开发指南

本文档为 AI 编程助手提供 SentGraph 项目的开发规范、构建命令和代码风格指南。

## 构建和测试命令

### 安装依赖

```bash
# 安装生产依赖
pip install -r requirements.txt

# 安装开发依赖
pip install -r requirements-dev.txt

# 或使用 setuptools 安装（包含开发依赖）
pip install -e ".[dev]"
```

### 运行测试

```bash
# 运行所有测试
pytest

# 运行单个测试文件
pytest tests/test_graph_builder.py

# 运行单个测试函数
pytest tests/test_graph_builder.py::test_graph_builder_initialization

# 运行测试并显示覆盖率
pytest --cov=sentgraph --cov-report=html

# 运行测试（详细输出）
pytest -v

# 运行测试（显示简短回溯）
pytest --tb=short
```

### 代码格式化

```bash
# 使用 Black 格式化代码
black sentgraph/ tests/ cli/ api/

# 检查代码格式（不修改）
black --check sentgraph/ tests/ cli/ api/

# 使用 isort 排序导入
isort sentgraph/ tests/ cli/ api/

# 检查导入顺序（不修改）
isort --check-only sentgraph/ tests/ cli/ api/
```

### 代码检查

```bash
# 使用 ruff 进行 linting
ruff check sentgraph/ tests/ cli/ api/

# 自动修复 ruff 问题
ruff check --fix sentgraph/ tests/ cli/ api/

# 使用 mypy 进行类型检查
mypy sentgraph/

# 运行所有检查（格式化 + lint + 类型检查）
black --check . && isort --check-only . && ruff check . && mypy sentgraph/
```

### 构建项目

```bash
# 构建分发包
python setup.py sdist bdist_wheel

# 安装项目
pip install -e .
```

## 代码风格指南

### Python 版本要求

- **最低版本**: Python 3.9+
- **目标版本**: Python 3.9, 3.10, 3.11

### 导入规范

1. **导入顺序**（使用 isort，profile=black）：
   - 标准库导入
   - 第三方库导入
   - 本地应用/库导入

2. **示例**：
```python
import json
import logging
from typing import Any, Dict, List, Optional

import numpy as np
from pydantic import Field

from sentgraph.config import settings
from sentgraph.exceptions import GraphConstructionError
```

3. **多行导入**：使用括号包裹，每行一个导入，末尾逗号
```python
from sentgraph.prompts import (
    get_cross_doc_bridge_prompt,
    get_nn_relation_prompt,
    get_ns_relation_prompt,
)
```

### 格式化规范

- **行长度**: 88 字符（Black 默认）
- **缩进**: 4 个空格
- **引号**: 使用双引号（Black 默认）
- **尾随逗号**: 多行结构末尾保留逗号

### 类型注解

1. **函数签名必须包含类型注解**：
```python
def process_documents(
    documents: List[str],
    document_ids: Optional[List[str]] = None,
) -> Dict[str, Any]:
    """处理文档。"""
    pass
```

2. **使用 typing 模块的类型**：
   - `List[T]`, `Dict[K, V]`, `Set[T]`, `Tuple[T, ...]`
   - `Optional[T]` 等价于 `Union[T, None]`
   - `Any` 用于动态类型

3. **mypy 配置**：
   - `disallow_untyped_defs = false`（允许未注解函数）
   - `ignore_missing_imports = true`（忽略缺失导入）

### 命名约定

1. **模块和包**: 小写，下划线分隔（`graph_builder.py`）
2. **类名**: 驼峰命名（`GraphBuilder`, `Neo4jClient`）
3. **函数和变量**: 小写，下划线分隔（`build_hierarchical_graph`）
4. **常量**: 全大写，下划线分隔（`MAX_DEPTH`）
5. **私有方法**: 单下划线前缀（`_split_sentences`）
6. **枚举**: 驼峰命名，值为小写字符串（`LLMProvider.OPENAI`）

### 文档字符串

1. **使用中文文档字符串**（项目统一使用中文）：
```python
def build_hierarchical_graph(
    self,
    documents: List[str],
    document_ids: Optional[List[str]] = None,
) -> Dict[str, Any]:
    """构建分层句子图。

    Args:
        documents: 文档列表。
        document_ids: 文档 ID 列表（可选）。

    Returns:
        构建统计信息字典。

    Raises:
        GraphConstructionError: 图构建失败时。
    """
    pass
```

2. **类文档字符串**：
```python
class GraphBuilder:
    """分层句子图构建器。"""
```

### 错误处理

1. **自定义异常类**：继承自 `SentGraphError`
```python
from sentgraph.exceptions import GraphConstructionError

if not documents:
    raise GraphConstructionError("文档列表不能为空")
```

2. **异常类定义**：
```python
class GraphConstructionError(SentGraphError):
    """图构建过程中的错误。"""
    pass
```

3. **异常处理**：捕获具体异常，提供有意义的错误信息

### 日志记录

1. **使用标准 logging 模块**：
```python
import logging

logger = logging.getLogger(__name__)

logger.info("开始构建图")
logger.warning("未找到配置，使用默认值")
logger.error("图构建失败", exc_info=True)
```

2. **日志级别**: DEBUG, INFO, WARNING, ERROR, CRITICAL

### 配置管理

1. **使用 Pydantic Settings**：
   - 从环境变量和 `.env` 文件读取配置
   - 使用 `BaseSettings` 和 `Field` 定义配置项
   - 使用枚举类定义选项值（如 `LLMProvider`）

2. **配置访问**：
```python
from sentgraph.config import settings

model = settings.llm_model
```

### 测试规范

1. **测试文件命名**: `test_*.py`
2. **测试函数命名**: `test_*`
3. **测试类命名**: `Test*`
4. **使用 pytest fixtures**：
```python
@pytest.fixture
def sample_documents():
    """示例文档。"""
    return ["文档1", "文档2"]
```

5. **跳过需要外部依赖的测试**：
```python
@pytest.mark.skip(reason="需要 Neo4j 数据库连接")
def test_integration():
    pass
```

### 其他规范

1. **模块级文档字符串**: 每个模块文件开头使用中文文档字符串
2. **空行**: 类和函数定义前后各一个空行
3. **注释**: 使用中文注释，解释"为什么"而非"是什么"
4. **常量定义**: 模块级常量使用全大写
5. **避免魔法数字**: 使用命名常量替代

## 项目结构

- `sentgraph/`: 核心库代码
- `tests/`: 测试代码
- `cli/`: 命令行工具
- `api/`: REST API 服务
- `docs/`: 文档

## 开发工作流

1. 创建功能分支
2. 编写代码（遵循上述规范）
3. 运行格式化: `black . && isort .`
4. 运行 lint: `ruff check .`
5. 运行类型检查: `mypy sentgraph/`
6. 编写测试
7. 运行测试: `pytest`
8. 提交代码

## 注意事项

- 所有文档字符串和注释使用中文
- 代码中的字符串可以使用中文（如错误消息）
- 保持与现有代码风格一致
- 在修改代码前先运行测试确保没有破坏现有功能

---
> Source: [jidechao/SentGraph-RAG](https://github.com/jidechao/SentGraph-RAG) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
