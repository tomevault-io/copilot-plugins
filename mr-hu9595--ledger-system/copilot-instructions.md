## ledger-system

> 工地材料设备进出库台账管理系统，支持自然语言录入、文档 OCR 解析、库存查询、报表导出。

# 台账管理系统 (Ledger System)

## 项目概述

工地材料设备进出库台账管理系统，支持自然语言录入、文档 OCR 解析、库存查询、报表导出。

## 技术栈

- **AI**: MiniMax API (understand_image + NLP)
- **数据库**: PostgreSQL
- **语言**: Python 3.9+
- **CLI**: 命令行界面

## 三层架构

```
program/     → CLI 入口、命令解析
business/    → AI服务、规则引擎、学习引擎、文档处理
data/        → PostgreSQL、数据模型、仓储
```

## 目录结构

```
ledger_system/
├── program/           # 程序层 (CLI)
│   ├── cli.py
│   └── commands/      # add, query, process, export
├── business/          # 业务层
│   ├── nlp/           # AI服务 + 规则引擎
│   ├── learning/      # 反馈学习 + 差异日志
│   └── document/      # OCR + 解析 + 文件监控
├── data/              # 数据层
│   ├── models/        # SQLAlchemy 模型
│   ├── repository/    # 数据仓储
│   └── database.py
├── config/
│   └── settings.yaml  # 配置文件
└── main.py
```

## 设计文档

详细设计见: `docs/superpowers/specs/2026-04-30-ledger-design.md`

## 工作流程

1. 自然语言录入 → AI 提取 → 规则兜底 → 写入台账
2. 文档/单据 → OCR → AI 解析 → 生成记录
3. 查询命令 → 展示库存/记录
4. 导出命令 → 生成 Excel 报表

## 常用命令

```bash
# 录入
python main.py add "今天进了3吨钢筋 from 杭州建材"

# 查询
python main.py status --material 钢筋
python main.py history --material 钢筋 --days 30

# 处理文档
python main.py process --file <path>

# 导出
python main.py export --type inventory --format xlsx --output 台账.xlsx
```

## 注意事项

- 数据库结构后续详细讨论
- 业务逻辑通过测试逐步完善
- AI 使用 MiniMax API，规则引擎兜底
- 支持规则学习进化（反馈 + 差异记录）

---
> Source: [Mr-Hu9595/ledger-system](https://github.com/Mr-Hu9595/ledger-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
