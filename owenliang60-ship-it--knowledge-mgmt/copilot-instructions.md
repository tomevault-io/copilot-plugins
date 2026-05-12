## knowledge-mgmt

> > **Install**: 把本文件复制到你的 Obsidian vault 根目录。


# AGENTS.md — Rules for AI Agents Operating the Vault

> **Install**: 把本文件复制到你的 Obsidian vault 根目录。
>
> **对齐 Karpathy LLM Wiki**: 这是 gist 里提到的 schema/agent rules layer ——
> 对 Claude Code 就是 `CLAUDE.md`，对 Codex 就是 `AGENTS.md`。这份文件对两者都有效。
>
> **Scope**: 本文件约束 AI agent（Claude Code / Codex / 其他）**操作 vault** 时的行为。适用的 agent 范围由用户自己指定（例如你可以在 `CLAUDE.md` 里写 "本 vault 的规则见 AGENTS.md"）。
>
> **与 SCHEMA.md 的关系**: `SCHEMA.md` 定义**什么是什么**（Card 类型、标签体系、权限矩阵），`AGENTS.md` 定义 agent **怎么操作**（CLI 命令、铁律、写入纪律）。两者互补，缺一不可。

---

## Schema 优先

操作 vault 前先读取 vault 根目录的 `SCHEMA.md`（`obsidian read path="SCHEMA.md"`），了解当前三层架构、权限矩阵和写入规则。

---

## 工具：Obsidian CLI（默认）+ MCP（回退）

**默认使用 CLI**（通过 Bash 工具调用 `obsidian` 命令）。仅当 Obsidian 应用未运行且 CLI 报 connection refused 时，回退使用 MCP（`mcp__obsidian__*`）。

> **架构**：CLI 是 IPC 遥控器，通过 socket 与运行中的 Obsidian GUI 通信。v1.12.4 起官方内置，100+ 命令。优势：移动文件自动更新 wikilinks、访问 backlinks/插件/同步状态。

### CLI 能力速查

| 操作 | CLI 命令 | MCP 回退 |
|------|----------|----------|
| 创建笔记 | `obsidian create path="..." content="..."` | `write_note` |
| 追加内容 | `obsidian append path="..." content="..."` | `write_note` (mode=append) |
| 前插内容 | `obsidian prepend path="..." content="..."` | `write_note` (mode=prepend) |
| 读取笔记 | `obsidian read path="..."` | `read_note` |
| 搜索 | `obsidian search query="..."` | `search_notes` |
| 带上下文搜索 | `obsidian search:context query="..."` | `search_notes` |
| 设置属性 | `obsidian property:set path="..." name="..." value="..." type=list` | `update_frontmatter` |
| 读取属性 | `obsidian property:read path="..." name="..."` | `get_frontmatter` |
| 标签列表 | `obsidian tags path="..."` | `manage_tags` |
| 移动/重命名 | `obsidian move path="..." to="..."` | `move_note` |
| 反向链接 | `obsidian backlinks path="..."` | _(MCP 无此功能)_ |
| 目录浏览 | `obsidian files folder="..."` / `obsidian folders` | `list_directory` |
| Vault 统计 | `obsidian vault` | `get_vault_stats` |
| 删除笔记 | `obsidian delete path="..."` | `delete_note` |

### 批量读取

CLI 无原生批量读取命令，使用 for 循环：

```bash
for card in $(obsidian search query="type/atomic"); do
  obsidian read path="$card"
done
```

> 85 张卡片 ~15 秒全读完，比 MCP 分批（每批 ≤10，需多轮 tool call）更高效。

---

## 铁律（FATAL）— 违反等于不可接受的灾难

> AI 最危险的时刻，不是它犯错的时候。是它觉得自己在帮你的时候。

### FATAL-001：禁止擅自重组文件结构

AI 天生有整理强迫症。它看到文件夹结构"不够整齐"，就浑身难受，恨不得帮你归归类、建建层级。但用户的文件结构是用户心智模型的外化。用户觉得它乱，其实什么都找得到。AI 觉得它整齐了，但用户什么都找不到了。

**绝对禁止**：移动文件夹、重命名目录、批量迁移文件。除非用户明确指示。

### FATAL-002：删除用户内容必须归档，禁止直接删

信息不可逆。删除是单向门——推开了就回不来。归档是暂停键，东西还在，只是换了个位置。所有不要的内容，归档，不删除。这条规则的本质是敬畏。对信息的敬畏，对用户记忆的敬畏。

### FATAL-003：禁止强加分类体系

用户自己的心智模型，比世界上任何分类体系都重要。你的笔记你知道放哪里。你的逻辑你自己清楚。AI 替用户决定分类，就像一个新来的实习生帮老板重新布置办公桌——你觉得你在帮忙，用户觉得你在添乱。

### FATAL-004：禁止破坏现有链接和标签

每个 `[[wikilink]]` 都是一条连接。每个 tag 都是一个维度。这些不是格式，是资产。是用户一条一条手动建立的关系网络，日积月累，越来越值钱。断链等于资产贬值。一次鲁莽的文件移动，可能让几十个链接同时失效。知识图谱瞬间变成孤岛群。

### FATAL-005：结构性变更必须走审批流

任何涉及文件移动、目录调整、批量重命名的操作，必须走流程：
1. **SCAN** — 只读模式，扫描现状
2. **PLAN** — 提出最小化变更方案，附安全叙事（哪些文件会动、怎么回滚、有没有链接会断）
3. **CONFIRM** — 展示 before/after 对照，等用户确认
4. **EXECUTE** — 审批通过才动手
5. **VERIFY** — 检查链接完整性，确认没有断链

质量标准三个字：**最少步骤**。不是帮你做最多的事，是做最少但最有效的事。

---

## 写入质量（SEVERE）— 违反需修正

### 笔记必须有 frontmatter

没有 frontmatter 的笔记是孤岛。搜不到，分不了类，进不了知识图谱。一个 vault 里有上千个 .md 文件，如果其中两百个没有 tags，你的知识图谱就缺了两百个节点。不是内容不存在，是系统看不见它。

最低要求：
- `tags` 字段（遵循四维标签体系：`type/` + `domain/` + `category/`（可选）+ `mastery/`（可选））
- `date` 或 `created` 字段

### 先搜再写，禁止创建冗余笔记

写入新笔记前，先搜索 vault 中是否已有同主题内容。你不需要三份内容一样的笔记。能追加就追加，能链接就链接，确认没有才新建。

### 不破坏 frontmatter 格式

元数据是 AI 可读性的基础，格式一乱，所有自动化全崩。修改笔记时必须保持现有 frontmatter 结构完整。

### 文件夹嵌套不超过 3 层

三层以上的目录结构，人类自己都记不住路径。

---

## 行为纪律

### 大批量操作必须先出 plan

涉及 5 个以上文件的操作（批量打标签、批量移动、批量修改 frontmatter），必须先列出完整方案，等用户确认再执行。**AI 提议，人类拍板。**

### 不确定归哪就放 inbox，不硬塞

如果不确定一篇笔记该归到哪个目录，不要猜。放进 `Inbox/`，生成选项让用户选。**不确定就说不确定。** 宁可多问一句，不可乱做一步。

### 移动文件前必须检查反向链接

移动任何 .md 文件前，先用 `obsidian backlinks path="..."` 检查有无其他笔记链接到它。有链接的文件移动后必须更新所有引用（CLI `move` 会自动更新 wikilinks），或告知用户影响范围。

---

## 八荣八耻

> 承认不知道，比假装懂了有用一万倍。

| 荣 | 耻 |
|---|---|
| 深入查阅现有笔记和结构 | 猜测 vault 目录和文件位置 |
| 主动确认再动手 | 模糊理解就执行 |
| 交给用户验证 | 假设用户的组织意图 |
| 复用已有笔记和 `[[链接]]` | 凭空新建重复内容 |

---

> **Customization**: 这份文件定义的是**作者本人使用这套系统时的纪律**。你 fork 时，可以根据你的领域和风格调整 —— 但请保留 FATAL 五条铁律。它们不是个人偏好，是防止 AI 毁掉知识资产的底线。

---
> Source: [owenliang60-ship-it/knowledge-mgmt](https://github.com/owenliang60-ship-it/knowledge-mgmt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
