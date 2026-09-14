## agent-operations

> description: Agent 执行与环境操作规范（venv、config、pip、scripts）

﻿---
description: Agent 执行与环境操作规范（venv、config、pip、scripts）
alwaysApply: true
---

# Agent 操作规范

完整说明见 `docs/prompts/agent-guidelines.md`。

## 最小执行

- 只运行验证**当前改动**所必需的命令；禁止无关训练、全量下载或批量数据处理。
- 优先阅读代码与静态分析；确需运行时再执行最小命令集。

## 目录扫描

- **禁止**对项目根无差别 `**/*`、全量 Glob、递归 `list_dir`。
- **默认排除**：`.venv/`、`.cache/`、`outputs/`、`__pycache__/`、`node_modules/`、`.git/`（无需读取其内容）。
- 用**有目标**路径/Glob 探索（如 `image/**/*.py`）；已知路径直接读文件。

## 虚拟环境

执行 `python` / `pip` 前先激活项目根目录 `.venv`：

- PowerShell: `.\.venv\Scripts\Activate.ps1`
- Linux/macOS: `source .venv/bin/activate`

## config.py

- 经 `main.py` 入口：由入口调用 `config.setup()`。
- 不经 `main.py` 的脚本或测试：在 import `matplotlib` / `torch` / `transformers` / `huggingface_hub` **之前**执行：

```python
import config
config.setup()
```

## 依赖安装

1. 新包先写入根目录 `requirements.txt`，再考虑安装。
2. `pip install -r requirements.txt` 前检查 `TMP`、`HF_HOME`、`PIP_CACHE_DIR`、`MPLCONFIGDIR` 等是否仍指向 C 盘且 `config.py` 未覆盖。
3. 若环境未就绪：**不要**执行 pip install；提示用户先在 `config.py` 配置路径后自行安装。

## scripts/

可复用批处理任务放入 `./scripts/`，仅用 Python 标准库，跨平台可运行。

## 其他

- 最小 diff，不擅自 git commit 或新建无关文档（除非用户要求）。
- 与用户交流使用简体中文。

---
> Source: [zhitian111/VistionTextCrossModelRetrivelByGAT](https://github.com/zhitian111/VistionTextCrossModelRetrivelByGAT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
