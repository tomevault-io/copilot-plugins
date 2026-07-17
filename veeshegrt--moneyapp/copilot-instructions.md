## moneyapp

> - **平台**：Windows / macOS（Electron 桌面应用）

# 记账APP - 产品文档

## 产品概述
- **名称**：记账APP
- **定位**：轻量级个人记账工具
- **平台**：Windows / macOS（Electron 桌面应用）
- **目标用户**：需要记录日常收支的个人用户

## 核心功能

### 1. 收支记录（核心）
- **收入记录**：金额、来源（工资/兼职/理财/其他）、日期、备注
- **支出记录**：金额、类别（餐饮/交通/购物/住房/娱乐/医疗/教育/其他）、日期、备注
- 支持快速添加（最少只需输入金额即可保存）

### 2. 数据展示
- **首页仪表盘**：本月总收入、总支出、净收入、近7天趋势图
- **流水列表**：按时间倒序展示所有记录，支持按月筛选
- **分类统计**：饼图展示各类别占比

### 3. 数据管理
- 本地 SQLite 存储，数据不离开设备
- 支持按月份查看历史记录
- 支持编辑和删除记录

## 技术架构

```
┌─────────────────────────────────────┐
│           Electron Shell            │
│  ┌───────────────────────────────┐  │
│  │        React Frontend         │  │
│  │  (Vite + TypeScript)          │  │
│  │  - 首页仪表盘                 │  │
│  │  - 记录列表                   │  │
│  │  - 添加/编辑表单              │  │
│  │  - 统计图表                   │  │
│  └───────────┬───────────────────┘  │
│              │ IPC                   │
│  ┌───────────▼───────────────────┐  │
│  │     Main Process (Node)       │  │
│  │  - SQLite 数据库操作           │  │
│  │  - 文件系统访问               │  │
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**技术选型**：
- **前端**：React 18 + TypeScript + Vite
- **UI 库**：Ant Design（中文友好，组件丰富）
- **图表**：ECharts（轻量，中文支持好）
- **数据库**：better-sqlite3（同步、轻量、无需服务）
- **打包**：electron-builder（支持 Win/Mac 双平台）

## 数据模型

```sql
-- 交易记录表
CREATE TABLE transactions (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  type TEXT NOT NULL,          -- 'income' | 'expense'
  amount REAL NOT NULL,        -- 金额
  category TEXT NOT NULL,      -- 分类
  note TEXT DEFAULT '',        -- 备注
  date TEXT NOT NULL,          -- 日期 YYYY-MM-DD
  created_at TEXT DEFAULT (datetime('now'))
);
```

## 页面结构（3个页面）

1. **首页**：仪表盘 + 快速记账入口
2. **流水**：所有记录列表，支持月份切换
3. **统计**：收支趋势图 + 分类饼图

## 分类定义

### 收入分类
- 工资、兼职、理财、红包、其他

### 支出分类
- 餐饮、交通、购物、住房、娱乐、医疗、教育、其他

---
> Source: [veeshegrt/moneyApp](https://github.com/veeshegrt/moneyApp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
