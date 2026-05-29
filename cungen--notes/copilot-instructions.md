## notes

> <!-- Generated: 2026-04-29 | Updated: 2026-04-29 -->

<!-- Generated: 2026-04-29 | Updated: 2026-04-29 -->

# obsidian-workspace

## Purpose
这是一个以 Obsidian 为编辑入口、以 Quartz v5 为发布引擎的个人知识库。LLM 在这里的职责不是“重写一切”，而是按分层工作流维护知识：保留原始来源，持续整理中间知识层，谨慎沉淀高价值卡片。

## Key Files
| File | Description |
|------|-------------|
| `README.md` | Quartz 本地开发、构建与检查命令 |
| `package.json` | 脚本入口，含 `dev`、`dev:full`、`check`、`test` |
| `quartz.config.yaml` | 站点配置 |
| `content/index.md` | 三层内容入口页 |
| `content/log.md` | ingest/query/lint 追加式日志 |
| `content/Cards/统一LLM-Wiki工作流方案.md` | 当前仓库采用的分层 LLM-Wiki 方案 |
| `content/Cards/Tags标签使用指南.md` | `Cards` 与 `Memos` 的标签约定 |

## Subdirectories
| Directory | Purpose |
|-----------|---------|
| `content/raw/` | 原始来源层；人类优先，只做轻量整理，不改写原意 |
| `content/wiki/` | LLM 可维护的综合层；放概念、实体、综合、来源摘要 |
| `content/Cards/` | 对外输出与行动层；放结论、决策、方法、稳定卡片 |
| `content/Memos/` | 快速捕获与临时记录；默认人类优先，不做激进改写 |
| `content/templates/` | 笔记模板 |
| `quartz/` | Quartz 构建、渲染与插件代码 |
| `public/` | 构建产物；非构建任务不要手改 |
| `docs/` | 规格、计划与过程文档 |

## Working Model
- 采用三层模型：`raw -> wiki -> cards`。
- 采用三类操作：`ingest`、`query`、`lint`。
- 默认原则：人类定方向，LLM 做整理、链接、归纳、维护。
- 内部链接优先使用 `[[wikilink]]`，不要把仓库内笔记改成相对 Markdown 链接。
- 优先增量更新，不做整页重写；优先编辑已有页面，不重复造新页。

## Frontmatter Contract
对 `content/raw/`、`content/wiki/`、`content/Cards/`、`content/Memos/` 下的笔记，新增或重写时尽量补齐这些字段：

- 必填优先：`title`、`tags`、`date`、`kind`、`managedBy`、`status`
- 推荐补充：`draft`、`updated`、`layer`、`confidence`、`sourceCount`、`sources`
- `layer` 只用：`raw`、`wiki`、`cards`
- `managedBy` 只用：`human`、`llm`、`hybrid`
- 若页面基于来源或下游综合生成，补 `sources: []`

如果文件已有旧 frontmatter，不要为追求“完美一致”做无关大迁移；只在本次修改触达的文件内顺手校正。

## Tag Rules
- 英文标签统一小写；Quartz 会对 frontmatter `tags` 做 slug 规范化，大小写混用会造成展示与索引分裂。
- 对 `Cards` 与 `Memos`，主轴标签只选一个：`area/*`、`resource/*`、`project/*`。
- 再补一个 `kind/*`，再补一个 `state/*`，未整理完成时可加 `inbox`。
- 不使用裸标签，如 `project`、`obsidian`、`share`。
- 不在 frontmatter 里写 `#tag`。
- 不要为这次任务批量重写全库旧标签；仅在相关文件内向新规范收敛。

可参考 [Tags标签使用指南](./content/Cards/Tags标签使用指南.md)

## Layer Rules

### `content/raw/`
- 这里存原始资料、网页摘录、论文、纪要、导出文本。
- 可以整理文件名、补元数据、补来源信息；不要改写正文事实含义。
- 原始资料的总结应写去 `content/wiki/`，不是写回 `raw/` 正文。

### `content/wiki/`
- 这里存 LLM 维护的知识页：概念、实体、综合、比较、来源摘要。
- 新建综合页时，优先沿用 `Summary`、`Evidence`、`Conflicts`、`Open Questions`、`Next Actions` 这类结构。
- 更新冲突时不要直接删旧结论；保留冲突信息，并把状态调到争议态。
- 若已有下游卡片，补回链；可用 `Derived Cards` 或等价区块。

### `content/Cards/`
- 这里只放高价值沉淀：结论、决策、方法、行动、成熟知识卡。
- 不要把长摘录、低加工资料直接倾倒到这里。
- 由 `raw/` 或 `wiki/` 派生的卡片，至少给出一个来源引用：正文 `Sources` 区或 frontmatter `sources`。
- 卡片以“人能直接使用”为先，不以“收集得越全越好”为目标。

### `content/Memos/`
- 这里是速记、草稿、半成品、暂存区。
- 默认保留作者语气与原始思路，不做大幅改写。
- LLM 可做的事：补标题、补标签、补最小 frontmatter、整理结构、提炼可升级内容。
- 若内容已经稳定，优先“提炼并迁移”到 `wiki/` 或 `Cards/`，而不是把 memo 直接改造成正式卡片。

## Ingest / Query / Lint

### Ingest
1. 新资料先进入 `content/raw/`。
2. 再在 `content/wiki/` 生成或更新概念页、综合页、来源摘要页。
3. 只有高价值结论才沉淀到 `content/Cards/`。
4. 涉及跨层新增或重要更新时，追加记录到 `content/log.md`。

### Query
1. 先看 `content/index.md` 与已有相关页面。
2. 回答问题时，优先读 `wiki/` 做综合，再落到 `Cards/` 给出可执行结论。
3. 若本次问答产生了可复用知识，再沉淀为卡片并回链来源。

### Lint
重点检查：
- 孤儿页与断链
- 低价值重复页
- `state/*` / `status` 长期失修页
- tag 漂移与同义重复
- `raw -> wiki -> cards` 链路是否断裂

## For AI Agents

### Before Creating Anything
- 先搜同主题现有笔记，能补就补，不能补再新建。
- 新建页面前先决定它属于哪一层，不要把 `raw`、`wiki`、`cards` 职责混在一起。
- 不要为了“更整齐”随意改文件名、批量搬家、批量改标签。

### Writing Style
- 默认写简洁中文；专有名词保留业界通用英文。
- 保留作者已有术语与命名，不要无故统一成另一套词汇。
- 先给结论，再给证据，再给待确认项；避免空泛套话。

### Source Traceability
- 任何综合、判断、决策，尽量能追到 `raw` 或上游 `wiki` 页面。
- 有冲突时并列记录，不偷删旧说法。
- 删除来源相关内容前，先确认是否还有别的页面依赖它。

### Scope Control
- 笔记任务通常不需要改 `quartz/`、`public/`、`.obsidian/`。
- 站点或插件任务才改 `quartz/`、`package.json`、`quartz.config.yaml`。
- 非构建任务不要手改 `public/`。

## Validation
- 纯笔记改动：检查 frontmatter、标签前缀、小写英文标签、`[[wikilink]]`、来源回链是否合理。
- 改 `quartz/`、配置或脚本后：运行 `npm run check`。
- 改渲染表现后：运行 `npm run dev` 或 `npm run dev:full`，在浏览器确认相关页面。

## Dependencies

### Internal
- `content/` 提供知识内容与治理页。
- `quartz/` 负责解析 frontmatter、标签与渲染。
- `docs/` 提供规范与计划，不是站点内容层。

### External
- Obsidian：主要编辑入口。
- Quartz v5：静态站点构建器。
- Node.js 22 + npm 10：本地运行环境。

<!-- MANUAL: Any manually added notes below this line are preserved on regeneration -->

---
> Source: [cungen/notes](https://github.com/cungen/notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
