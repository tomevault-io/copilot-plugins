## harness-engineering-from-cc-to-ai-coding

> 本项目基于 Claude Code v2.1.88 的逆向工程源码，编写一本关于 AI Agent 驾驭工程（Harness Engineering）的技术书籍（29 章 + 4 附录）。

# 驾驭工程 — Claude Code 源码分析书籍项目

本项目基于 Claude Code v2.1.88 的逆向工程源码，编写一本关于 AI Agent 驾驭工程（Harness Engineering）的技术书籍（29 章 + 4 附录）。

## Specs & Plan

任务契约和实施计划在 `specs/` 目录：

- `specs/project.spec.md` — 项目级约束、决策、边界（被所有任务继承）
- `specs/part1-architecture.spec.md` — 第一篇：架构（ch01-ch04）
- `specs/part2-prompt-engineering.spec.md` — 第二篇：提示工程（ch05-ch08）
- `specs/part3-context-management.spec.md` — 第三篇：上下文管理（ch09-ch12）
- `specs/part4-prompt-cache.spec.md` — 第四篇：提示词缓存（ch13-ch15）
- `specs/part5-safety-permissions.spec.md` — 第五篇：安全与权限（ch16-ch19）
- `specs/part6-advanced-subsystems.spec.md` — 第六篇：高级子系统（ch20-ch24）
- `specs/part7-lessons.spec.md` — 第七篇：经验教训（ch25-ch29 + 附录）
- `specs/writing-plan.md` — 写作执行计划，含依赖图和批次安排

编写任何章节前，先读对应的 spec 文件。大纲在 `docs/book-outline.md`。

## 项目结构

```
restored-src/src/    # Claude Code v2.1.88 还原的 TypeScript 源码（只读，不修改）
book/                # mdbook 书籍项目
  book.toml          # mdbook 配置
  src/
    SUMMARY.md       # 目录结构（mdbook 必需）
    preface.md       # 前言
    part1/ch01-04.md # 第一篇：架构
    part2/ch05-08.md # 第二篇：提示工程
    part3/ch09-12.md # 第三篇：上下文管理
    part4/ch13-15.md # 第四篇：提示词缓存
    part5/ch16-19.md # 第五篇：安全与权限
    part6/ch20-24.md # 第六篇：高级子系统
    part7/ch25-29.md # 第七篇：经验教训
    appendix/        # 附录 A-D
docs/
  book-outline.md    # 书籍大纲（28 章 + 4 附录）
specs/               # agent-spec 任务契约
```

## 构建和预览

```bash
cd book && mdbook build    # 构建 HTML 到 book/book/
cd book && mdbook serve    # 本地预览 http://localhost:3000
```

章节文件写在 `book/src/partN/chNN.md`，mdbook 通过 `SUMMARY.md` 自动组织目录。

## 必须使用的 Skills

编写本书时，AI agent 必须加载以下 skill：

### 写作 Skill（最重要）

| Skill | 何时加载 |
|-------|---------|
| `tech-writer` | **编写任何章节前必须加载。** 遵循三阶段流程：Research → Write（含预算声明）→ Review（3-agent 并行审阅） |

### Spec 管理 Skills

| Skill | 何时加载 |
|-------|---------|
| `agent-spec-authoring` | 编辑 `.spec.md` 文件时 |
| `agent-spec-tool-first` | 根据 spec 验收标准验证章节完成度时 |

### 章节写作流程

编写一章的标准流程：

1. **读 Spec** — 先读 `specs/` 下对应的 spec 文件，确认验收标准和边界
2. **读 Outline** — 读 `docs/book-outline.md`，确认本章在全书中的位置、与前后章的依赖关系
3. **加载 tech-writer** — 按 tech-writer skill 的 Book Writing Mode 执行 Pre-Flight 检查
4. **声明预算** — 从 spec 的约束中提取字数和深度层级，向用户确认
5. **Research** — 验证所有源码引用，确认 `restored-src/` 路径和行号（tech-writer Phase 1）
6. **Write** — 按章节结构模板写作（tech-writer Phase 2）
7. **Review** — 启动 3-agent 并行审阅（tech-writer Phase 3）
8. **Verify** — 用 `agent-spec-tool-first` 验证 spec 中的验收标准全部通过

## 写作规范

### 预算制度

每章开始写作前，必须声明预算（tech-writer skill 的 Step 0）：

```
Type: book-chapter
Depth: 3 layers (What → Why → How to prevent)
Word budget: 5,000-8,000 words per chapter
```

如果某个论点需要第 4 层深度（production 约束讨论），在预算中显式标注。

### 章节结构

每章必须包含以下部分：

```markdown
# 第N章：标题

## 为什么这很重要
  动机说明（2-3 段）

## 源码分析
  核心分析，代码片段标注 `file:line`
  流程图使用 Mermaid 格式

## 模式提炼
  从分析中提取可复用的模式
  每个模式：名称、解决的问题、代码模板、前置条件

## 用户能做什么
  读者可操作的建议（如适用）
```

### 源码引用规范

- 所有技术论断必须有源码证据，格式：`restored-src/src/path/to/file.ts:行号`
- 代码片段使用实际源码，不使用伪代码
- 代码块标注语言和源文件：

```typescript
// restored-src/src/services/compact/autoCompact.ts:72-91
export function getAutoCompactThreshold(model: string): number {
  ...
}
```

### 每章图表要求

| 图表类型 | 要求 |
|---------|------|
| 流程图 | 每章至少 1 个 Mermaid 流程图展示核心流程 |
| 对比表 | 涉及多个选项/模式时必须有对比表 |
| 架构图 | 第一篇各章需要架构层级图 |
| 数据表 | 涉及具体数值（token 数、阈值）时用表格汇总 |

### 跨章引用

- 同一段源码不在两个章节中重复出现超过 3 行
- 跨章引用使用 "详见第N章" 格式
- 第3章（Agent Loop）是全书的锚点，其他章节引用它来定位各自在循环中的位置

### 语言规范

- 中文写作，技术术语首次出现时附英文原文：如"压缩（Compaction）"
- 之后使用中文简称即可
- 不对 Anthropic 做道德评判，聚焦于工程实践本身
- 不在书中包含安全凭证、API key、内部 Slack channel ID

## 源码目录（只读）

源码在 `restored-src/src/` 下，关键文件索引见 `docs/book-outline.md` 附录 A。

**绝不修改 `restored-src/` 下的任何文件。**

## Review 流程

每章写完后，按 tech-writer skill 的 Phase 3 执行 3-agent 并行 review：

1. **fact-checker**：验证源码引用是否指向真实存在的文件和行号
2. **tech-reviewer**：检查技术公平性和代码示例的 production 可行性
3. **structure-editor**：检查结构一致性、跨章重复、措辞问题

三个 agent 必须在**单条消息中并行启动**。

---
> Source: [ZhangHanDong/harness-engineering-from-cc-to-ai-coding](https://github.com/ZhangHanDong/harness-engineering-from-cc-to-ai-coding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
