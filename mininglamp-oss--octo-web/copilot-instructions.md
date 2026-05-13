## octo-web

> > 通用工作习惯在 Agent 自身配置里。

# AGENTS.md — DMWork 项目工作约定

> 只写这个项目特有的约定。
> 通用工作习惯在 Agent 自身配置里。
> 技术规范在 **DEVELOPMENT.md**。

---

## 开始任务前

读 `DEVELOPMENT.md` — 按顶部"快速查阅"找对应章节，不需要全读。

在分配的 worktree 里工作，不要动主仓库目录。

---

## 新建 UI 组件：必须先写 Story 再接业务

```
1. 建组件文件（index.tsx + index.css）
2. 写 Story（ComponentName.stories.tsx）
3. Storybook 里验证通过（light + dark 都看）
4. 再接入业务代码
```

顺序不能颠倒。CI 会检查 story 覆盖，没有 story 的新组件 PR 不能合并。

Story 写法见 DEVELOPMENT.md 章节四、六。

---

## 禁止事项

详细规范见 DEVELOPMENT.md 对应章节，以下为核心约束：

- **硬编码颜色/间距/圆角** → 章节二
- **`!important`** → 章节十三
- **直接覆盖 Semi class** → 章节十三
- **在组件里创建新颜色变量** → 章节十三
- **`@media (prefers-color-scheme: dark)`** → 章节十三

---

## UI/数据分离架构

本项目遵循 UI/数据分离开发规范（skill: `ui-data-separation`）。

**三层结构：**
- `ui/` — 纯 UI 组件，无 WKSDK/WKApp，由 agent 维护
- `bridge/` — 数据桥接层（types.ts + use*.ts），由工程师维护
- `Components/` / `Messages/` — 旧组件，迁移中，**禁止修改**

**三条核心规则：**
1. `ui/` 下禁止 import `wukongimjssdk`、`WKApp`、`Service/`
2. 组件 props 类型只用 `bridge/types.ts` 里的类型
3. 遇到越界需求：输出工单，不自行处理

**详细流程：** 读 skill `ui-data-separation` 的对应 reference
**项目路径配置：** `AGENTS.config.json`（根目录）

---
> Source: [Mininglamp-OSS/octo-web](https://github.com/Mininglamp-OSS/octo-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
