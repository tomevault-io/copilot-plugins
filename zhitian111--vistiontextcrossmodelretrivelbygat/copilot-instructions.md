## python-standards

> description: Python 编码规范（注释、全局变量、项目风格）

﻿---
description: Python 编码规范（注释、全局变量、项目风格）
globs: "**/*.py"
alwaysApply: false
---

# Python 编码规范

## 注释与文档

- **模块**：顶部 docstring 说明职责、用法、与 `config.py` / `main.py` 的关系。
- **类**：docstring 写清权责边界（负责什么、不负责什么）。
- **函数**：docstring 说明输入输出、副作用；非平凡函数写 Args / Returns。
- **函数内部**：对算法步骤、边界条件、非显而易见逻辑加注释；避免废话注释。

## 全局变量

- 不在非 setup 阶段修改模块级可变状态或 `os.environ`。
- 环境初始化集中在 `config.setup()` 或模块 `setup()` 中；业务逻辑通过参数传递。
- 模块级标志（如 `_setup_done`）仅在 `setup()` 内写入一次，保证幂等。

## 风格

- 使用 `from __future__ import annotations`、类型注解、`pathlib.Path`。
- 匹配现有代码风格；不重构或格式化无关文件。

## 示例

```python
def build_spatial_graph(patch_coords, k):
    """根据 patch 坐标构建空间 kNN 图。

    职责：几何邻接构图。不负责语义特征或动态图更新。
    """
    # 1. 计算 pairwise 距离
    ...
    # 2. 取 k 近邻并构造对称边
    ...
```

---
> Source: [zhitian111/VistionTextCrossModelRetrivelByGAT](https://github.com/zhitian111/VistionTextCrossModelRetrivelByGAT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
