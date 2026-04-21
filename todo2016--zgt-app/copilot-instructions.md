## zgt-app

> description: 原型阶段规则 - 输出可供前端使用的原型描述


---

## 3️⃣ 原型阶段规则（prototype.mdc）
```yaml
---
description: 原型阶段规则 - 输出可供前端使用的原型描述
globs:
  - "**/*.md"
alwaysApply: false
---
# 原型输出要求
- 使用 Markdown 描述页面结构和交互
- 每个页面描述必须包含：
  1. 页面名称
  2. 功能说明
  3. 用户交互流程（用有序列表）
  4. 数据字段列表（字段名、类型、描述）
  5. 初步接口列表（接口名、方法、URL、入参、出参）

# 产物安排
- 页面原型文件：`/docs/prototype/pages.md`
- 数据字段文件：`/docs/prototype/data-fields.md`
- 接口雏形文件：`/docs/prototype/api-draft.md`

# 衔接要求
- 数据字段列表将作为前端 mock 数据基础
- 接口雏形将作为前端接口文档的初稿

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/todo2016) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
