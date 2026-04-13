## js-and-ts-study

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 回复风格
- 始终保持简洁，去掉冗余解释

## 命令

```bash
# 运行 JS 文件
node 01-js-basics/01-variables-and-types/examples.js

# 运行 TS 文件
npx ts-node 03-ts-basics/01-type-annotations/examples.ts

# TS 类型检查（不生成文件）
npx tsc --noEmit
```

## 项目结构

4 个学习阶段，每个主题目录包含三个固定文件：`examples.js/.ts`（示例）、`exercise.js/.ts`（练习）、`notes.md`（C++/Python 对比笔记）。

| 目录 | 内容 |
|------|------|
| `01-js-basics/` | 变量、函数、闭包、对象、数组、类、模块（10 个主题） |
| `02-js-intermediate/` | Promise、async/await、事件循环、迭代器、Proxy（8 个主题） |
| `03-ts-basics/` | 类型注解、接口、泛型、联合类型、类型守卫（7 个主题） |
| `04-ts-intermediate/` | 工具类型、条件类型、映射类型、声明文件（6 个主题） |
| `mini-projects/` | CLI Todo App、异步文件处理器、类型安全 API 客户端 |

## TS 配置要点

- `"type": "module"` — 项目使用 ES Module
- `tsconfig.json` 的 `include` 仅覆盖 `03-ts-basics/`、`04-ts-intermediate/`、`mini-projects/`，不包含 JS 文件
- target: ES2022，strict 模式开启

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zenyarn)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zenyarn)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
