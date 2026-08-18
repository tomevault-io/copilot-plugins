## syyyy618-expense-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

CLI / GUI / Web 三模式记账/支出追踪工具。数据层共享同一 SQLite 数据库。

## 安装与启动

```bash
pip install -e .

# CLI 模式
expense add 35.5 -c 餐饮 -d "午餐"
expense ls -m 2026-08
expense sum

# Web 模式（推荐）
expense-web
# 浏览器访问 http://localhost:8020

# 原生 GUI 模式
expense-gui
```

## 架构

```
expense_tracker/
├── __init__.py       # 版本号
├── __main__.py       # python -m 入口 → CLI
├── db.py             # 数据层：SQLite CRUD，所有 SQL 集中于此
├── cli.py            # CLI：argparse 子命令 → cmd_* 函数
├── gui.py            # 原生 GUI：tkinter 侧边栏 App
├── web_server.py     # Web 后端：HTTP API 服务器 (端口 8020)
└── index.html        # Web 前端：Tailwind CSS + Vanilla JS SPA
```

### 分层

- **`db.py`** — 数据层。`get_connection()` 是唯一建连入口。函数返回 `list[dict]` 或基本类型。
- **`cli.py`** / **`gui.py`** / **`web_server.py`** — 三个平行的表示层，均调用 `db.*`。
- **`index.html`** — 纯前端 SPA，通过 `fetch()` 调用 web_server 的 JSON API。

### Web API

| Method | Path | 说明 |
|--------|------|------|
| GET | `/api/expenses?month=&category=` | 列出支出 |
| POST | `/api/expenses` | 添加支出 `{amount, category, date, description}` |
| PUT | `/api/expenses/<id>` | 更新支出 |
| DELETE | `/api/expenses/<id>` | 删除支出 |
| GET | `/api/summary?month=` | 分类汇总 + 总额 |
| GET | `/api/monthly` | 月度趋势 |
| GET | `/api/budgets?month=` | 预算执行状态 |
| POST | `/api/budgets` | 设置预算 `{category, month, amount}` |
| GET | `/api/categories` | 已有分类列表 |

### 前端组件 (Vanilla JS)

- **`CategorySelector`** — 分类下拉组件，点击展开预设分类列表，支持 emoji 图标，选中回调
- **`AmountInput`** — 金额输入组件，¥ 前缀 + 快捷金额按钮 + 表单验证（红色错误提示）
- **`App`** — 主应用类，管理侧边栏导航、页面切换、API 调用

### 数据库

- 路径: `$EXPENSE_TRACKER_HOME/expenses.db`，默认 `~/.expense-tracker/expenses.db`
- `expenses`: id, amount, category, description, date, created_at
- `budgets`: category (NULL=总预算), amount, month, UNIQUE(category, month)
- WAL 模式，外键开启

### 关键约定

- 金额单位元，`FMT(v)` / `_fmt_money()` 统一格式 `¥X,XXX.XX`
- 日期 `YYYY-MM-DD`，月份 `YYYY-MM`
- 零外部依赖（除 Web 前端的 Tailwind CDN）
- 数据按 date DESC, id DESC 排列
- Windows 控制台需 `sys.stdout.reconfigure(encoding="utf-8")` 避免 GBK 错误

---
> Source: [syyyy618/syyyy618-expense-tracker](https://github.com/syyyy618/syyyy618-expense-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
