## fwhagent

> - **变量与函数**：统一使用小写字母加下划线（snake_case）。例如：`user_data`, `get_response()`。

# Python 核心开发规范 (v1.0)

## 1. 命名风格 (Naming Conventions)

- **变量与函数**：统一使用小写字母加下划线（snake_case）。例如：`user_data`, `get_response()`。
- **类名**：使用大驼峰命名（PascalCase）。例如：`DatabaseManager`。
- **常量**：全大写加下划线。例如：`MAX_RETRY_COUNT = 5`。
- **私有成员**：以单下划线开头。例如：`_internal_method()`。

## 2. 代码结构与逻辑

- **类型提示 (Type Hinting)**：所有函数参数和返回值必须明确类型。
  - 正确示例：`def add_user(user_id: int, name: str) -> bool:`
- **文档字符串 (Docstrings)**：复杂的公共函数必须包含三引号文档注释，说明功能、参数及返回值。
- **显式优于隐式**：避免使用 `from module import *`，应明确导入具体名称。
- **错误处理**：使用具体的异常类（如 `ValueError`），禁止使用宽泛的 `except Exception:`。

## 3. 性能与安全性

- **列表推导式**：优先使用列表推导式处理简单循环，但嵌套超过两层时应改回常规 loop。
- **路径处理**：严禁直接拼接字符串，必须使用 `pathlib.Path` 处理路径。
- **上下文管理**：处理文件或数据库连接必须使用 `with` 语句确保安全关闭。

## 4. 给 Codex 的交互要求

- **重构优先**：在修改现有代码前，先分析当前代码的性能瓶颈。
- **伪代码确认**：对于逻辑复杂的算法，先输出逻辑说明，得到我的确认后再写具体代码。
- **单单元测试**：生成函数后，默认附带一个基于 `pytest` 或 `unittest` 的测试用例。

## 5. 项目背景 (Context)

-运行环境：Python 3.12.3+。

---
> Source: [fanweihao12-debug/fwhAgent](https://github.com/fanweihao12-debug/fwhAgent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-28 -->
