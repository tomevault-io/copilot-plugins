## vibe-solo-kit

> 把本仓库维护成一个无需读取聊天历史，也能被全新 Agent 会话快速恢复的产品工程。

# Agent 协作协议

## 项目使命

把本仓库维护成一个无需读取聊天历史，也能被全新 Agent 会话快速恢复的产品工程。

## 两套上下文的职责边界

- `shaping/`：由 PM-Make 维护的**产品定型基线**，回答为谁做、解决什么问题、MVP 做什么、有哪些页面流程和业务规则。
- `context/`：开发过程的**运行上下文**，回答当前做到哪、这一轮改什么、采用了什么工程决策、如何验证、发布后从哪里继续。

不得在 `context/` 中复制产品定位、JTBD、MVP 范围、页面清单或业务流程。需要改变产品基线时，回到 PM-Make 流程更新 `shaping/`，经用户确认后向 `shaping/06-shaped-brief.md` 追加新的定型摘要。

## 会话启动

修改代码前必须：

1. 阅读 `context/INDEX.md` 和 `context/current/NOW.md`。
2. 阅读 `shaping/06-shaped-brief.md` 中最新一版确认摘要，以及 `shaping/05-open-questions.md`。
3. 按 INDEX 的路由表，只加载当前任务需要的定型文档、增量规格和工程约束。
4. 复述定型基线、当前状态、本轮增量、关键约束和验证计划。
5. 如果不存在已确认的定型摘要，先使用 PM-Make 完成产品定型，不得直接开始产品代码开发。

## 作业过程中

- 把 `shaping/` 视为产品要求的唯一事实来源，把当前增量规格的验收标准视为本轮交付契约。
- 产品定位、范围、页面流程或业务规则发生变化时，回到 PM-Make 更新产品基线。
- 存在多个合理工程方案，或选择会形成长期技术约束时，在 `context/decisions/` 记录决策。
- 修改组件规范、API 契约、数据结构、权限实现或运行环境时，同步更新 `context/interfaces/constraints.md`。
- 不得把聊天记录作为长期事实的唯一存放位置。
- 保留用户已有改动，避免顺手进行无关重构。

## 完成定义

只有同时满足以下条件，任务才算完成：

- 实现没有偏离最新产品定型基线；
- 当前增量规格的验收标准已经满足；
- 相关自动化检查通过；
- 已完成人工验收，或明确标注为待验收；
- `context/current/NOW.md` 已反映最新状态；
- 相关工程决策和验证证据已经记录；
- 若产生了可运行版本，已经创建发布快照并注明对应的定型基线。

把 `skills/start-task/SKILL.md`、`skills/finish-task/SKILL.md` 和 `skills/release-version/SKILL.md` 作为标准作业流程。

---
> Source: [comeonzhj/vibe-solo-kit](https://github.com/comeonzhj/vibe-solo-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
