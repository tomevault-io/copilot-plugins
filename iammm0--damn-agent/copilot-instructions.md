## damn-agent

> 本文件是 Damn Agent 项目协作者在编码时使用的编码代理、编程助手和自动化执行工具需要遵从的协作约定，适用于 Claude Code、Codex、Cursor、Cline、Aider、Continue 等协助读写代码、运行命令、提交改动或创建 PR 的工具。

# AGENTS.md

本文件是 Damn Agent 项目协作者在编码时使用的编码代理、编程助手和自动化执行工具需要遵从的协作约定，适用于 Claude Code、Codex、Cursor、Cline、Aider、Continue 等协助读写代码、运行命令、提交改动或创建 PR 的工具。

优先级低于用户在当前对话中的明确指令，高于代理自己的默认习惯。协作者在使用上述工具参与本项目开发时，应把本文件作为项目级编码约定提供给对应代理。

## 适用对象

- 使用 Claude Code、Codex、Cursor 等编码代理进行需求拆解、代码修改、文档编写、测试验证、提交和 PR 的协作者。
- 被协作者授权进入本仓库执行命令、读取文件、编辑文件、操作 Git、调用浏览器或其他本地工具的编程助手。
- 围绕本项目生成代码、修改文档、维护排期、整理发布记录的自动化脚本或 agent workflow。

## 项目定位

- 本项目是面向中文开发者的 AI Agent 技术学习文档站，核心目标是把 Agent 从概念、论文、框架宣传拆成可学习、可实现、可调试、可评测、可维护的工程知识。
- 内容应服务真实学习和工程落地，不写空泛宣传文案。
- 页面和文档默认使用中文；专业术语第一次出现时建议给出英文原词。
- 文档需要兼顾基础读者和工程实践读者：先讲清定义、边界和心智模型，再给流程、代码、检查清单、案例和延伸阅读。

## 代码与内容规范

- 先使用 `rg` 或 `rg --files` 查找文件和内容，再按现有结构修改。
- 文档内容位于 `content/docs`；新增 MDX 页面时同步检查对应目录的 `meta.json`。
- 更新 MDX 文档后，页面会自动读取 Git 最近一次提交的作者与时间，并在文末展示“最后编辑 / 编辑者”。
- 如需手动覆盖 Git 记录，可在 frontmatter 中补充 `lastEdited`（ISO 8601）与 `editor`（GitHub ID）；手动字段优先级高于 Git。
- 协作者更新文档时应使用已绑定 GitHub 的 Git 身份提交，确保编辑者显示为正确的 GitHub ID。
- 根目录 `CHANGESLOG.md` 是项目排期者维护的总进度文件；每次新版本发布时更新协作者进度、内容更新、工程更新、风险和下一步计划。
- 根目录 `AGENTS.md` 是代理协作约定；修改项目协作规则时同步更新本文件。
- 不要随意移动已有内容结构，不要做与任务无关的大规模重构。
- 引用书籍、论文、文章、仓库或外部资料时，尽量保留来源、页码、章节、链接或 commit 线索，方便复核。

## 本地验证

常用命令：

```bash
pnpm lint
pnpm exec tsc --noEmit
pnpm build
```

维护者页依赖 GitHub collaborators 接口，构建或本地预览前请配置 `GITHUB_TOKEN`（可用 `gh auth token` 获取）。

修改文档或页面后，至少运行与改动风险匹配的检查。只改 Markdown/MDX 文案时可以优先运行 `pnpm lint`；改到路由、组件、构建配置时运行 TypeScript 和 build。

## Git 约定

- 修改代码或文档后必须提交一次 commit。
- 一次性改动较多时，按功能点拆分为多个 commit，不要把不相关改动混在一个提交里。
- 提交前只暂存本次任务目标文件，执行 `git diff --cached --name-status` 核对暂存范围。
- 提交信息使用中文，遵循 `feat:`、`fix:`、`docs:`、`chore:`、`refactor:` 等前缀。
- 为避免中文乱码，可以先将提交信息写入 UTF-8 文本文件，再使用 `git commit -F <file>`。
- 提交后按用户要求推送；如果推送失败，报告失败原因和当前 commit hash。
- 不要回滚、删除或覆盖用户未要求处理的改动。

## 发布与排期

- 新版本发布前，项目排期者需要更新 `CHANGESLOG.md`。
- 每个协作者的工作应能追踪到 GitHub ID、微信号、负责板块、todo 编号、PR 或文档链接。
- 版本记录中应写明已完成、进行中、阻塞、延期和下个版本计划，不只写笼统总结。
- 第一阶段学习排期和认领规则以 `.github/ISSUE_DRAFTS/phase-1-agent-learning-plan.md` 审核通过后的 GitHub issue 为准。

---
> Source: [iammm0/damn-agent](https://github.com/iammm0/damn-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
