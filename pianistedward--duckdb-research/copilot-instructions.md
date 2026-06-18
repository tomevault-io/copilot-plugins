## duckdb-research

> 本文件用于指导 Claude Code 在本仓库中的工作方式与关注点。

# CLAUDE.md

本文件用于指导 Claude Code 在本仓库中的工作方式与关注点。

## 项目概况

- 版本：DuckDB v1.5.0（源码研究）
- 目标：CSV 稀疏存储特性（从解析 → 表函数 → 存储落盘的链路）

参考文档：
- 在线文档 https://duckdb.org/docs/current/
- 离线文档 docs/duckdb-docs-v1.5.0.pdf

## 目录结构

- `docs`：项目文档
- `source_code`：项目源码
- `python`：生成的python脚本位置

## 文档

- **稀疏存储研究**：`docs/sparse-storage-research.md`
- **Python 模块 API**：`docs/python-module.md`

## Python 模块

`python/sparse_csv.py` 提供稀疏 CSV 处理功能，详见 `docs/python-module.md`

---
> Source: [PianistEdward/duckdb-research](https://github.com/PianistEdward/duckdb-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
