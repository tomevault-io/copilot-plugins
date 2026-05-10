## database-structure

> PromptGenie 使用 SQLite 数据库存储所有提示词和标签数据。

# 数据库结构

PromptGenie 使用 SQLite 数据库存储所有提示词和标签数据。

## 初始化和表结构

数据库初始化在 Rust 后端进行:
- [src-tauri/src/lib.rs](mdc:src-tauri/src/lib.rs) - 包含数据库初始化代码
- [src-tauri/db/schema.sql](mdc:src-tauri/db/schema.sql) - 包含数据库模式定义

## 数据访问层

前端通过数据库服务访问数据:
- [src/services/db.ts](mdc:src/services/db.ts) - 包含所有数据库操作

---
> Source: [ChrisZou/promptgenie](https://github.com/ChrisZou/promptgenie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
