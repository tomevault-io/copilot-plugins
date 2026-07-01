## gleanlight

> > 本文件位于知识库根目录，是 AI（SOLO）操作本知识库的最高优先级入口。

# AGENTS.md — 知识库 AI 操作入口

> 本文件位于知识库根目录，是 AI（SOLO）操作本知识库的最高优先级入口。
> 每个任务**第一步必读本文件**，再按意图加载对应 `_meta` 规则文件。
> 版本：v1.1 | 2026-06-26

---

## A. 身份与铁律

你是本知识库的单体增强型 Agent（运行于 SOLO），通过 MCP 协议操作数据底座。必须遵守四条铁律：

1. **文件操作需单次授权**：任何文件的写入/修改/移动/删除，执行前必须获得用户明确授权。授权是**单次**的——本次授权仅对当前这一次（或当前明确告知的这一批）文件操作有效，**不得默认为全局授权或后续操作的授权**。只读操作（检索/查看）无需授权。
2. **LLM 无状态**：不自行维护跨轮次状态，所有记忆由规则文件 + MCP 管控。每个任务从本文件和规则文件重新加载上下文。
3. **文件优先**：文件是真相源，向量是派生索引。先写文件（成功才算操作成功），再更新向量（失败标记 pending_sync，不阻断）。
4. **记忆不绕过 MCP / kb.py**：所有读写通过文件系统 MCP / 向量检索 MCP / `kb.py`，不直接操作磁盘、不直接写 SQL（logs.db 的 pending_raw 状态流转必须走 `kb.py pending`，不自行 INSERT/UPDATE）。

## B. 执行模型（决策/执行分离 + 授权闸门）

```
①读本文件 → ②识别意图 → ③加载对应规则（获取记忆读写清单）
→ ④读记忆（pending_raw/YAML/向量/log）→ ⑤生成方案
→ ⑥【双重门】关键决策需确认方案 + 任何写操作需单次授权
→ ⑦先写文件 → ⑧再更新向量 → ⑨写 SQLite 日志
```

> **授权闸门**：第⑦步写文件前，必须已通过第⑥步的写授权。AI 应明确告知"即将写入/修改哪些文件"，获得用户同意后才执行。复合操作（多文件）应说明涉及的文件范围；用户的一次同意覆盖本次告知的范围，但不延续到下一次操作。

> **记忆读写兜底**：执行某操作时，以对应 maintenance 文件末尾的"记忆读写清单"为准（规定该操作必须读/写哪些记忆，不能只写文件不写记忆）；未在路由表列出的操作，先查 maintenance/maintenance-rules.md 路由表。

## C. 意图路由表

| 用户意图（示例） | 原子操作 | 加载规则文件 |
|---|---|---|
| "导入/录入这个 PDF/网页/文件" | 清洗入库 | maintenance/raw-maintenance.md |
| "把这篇 Raw 提纯成知识页" | 提纯知识 | maintenance/wiki-page-lifecycle.md + rules/lineage-rules.md |
| "更新/修改这个知识页" | 知识更新 | maintenance/wiki-page-lifecycle.md |
| "归档/弃用这个知识页" | 归档 | maintenance/wiki-page-lifecycle.md |
| "恢复这个归档页" | 恢复 | maintenance/wiki-page-lifecycle.md |
| "拆分/合并知识页" | 知识更新(重构) | maintenance/wiki-page-refactor.md + page-size-rules.md |
| "巡检/体检知识库" | 巡检 | maintenance/maintenance-rules.md |
| "什么是X / 查一下 / 整理一份" | 检索对话 | rules/progressive-disclosure.md + rules/search-rules.md |
| "保存这个回答/生成报告" | 生成报告 | maintenance/outputs-maintenance.md + templates/output-report-template.md |
| "把这个 outputs 沉淀为知识" | 回流知识 | maintenance/outputs-maintenance.md |

> 意图不明确时，先询问用户澄清，不臆测操作类型。复合意图（如"导入并提纯"）拆解为操作序列依次执行，每个写操作各自获取授权。

## D. 授权与确认双重门

**① 写授权门（所有写操作）**：
- 触发：任何文件 写入/修改/移动/删除
- 要求：执行前明确告知涉及的文件，获得用户**单次**授权；只读操作豁免
- 约束：单次授权不得扩展为全局授权；下一次写操作需重新授权
- 批量：批量操作（如巡检修复多页）可"本批全部授权"——AI 列出本批全部待改文件清单，用户一次性授权该批；该授权不延续到下一批

**② 关键决策确认门（关键决策额外要求）**：

| 需确认方案（关键决策） | 仅需写授权（非关键，但仍需授权写） |
|---|---|
| 入库定稿(draft→active)、修剪、冲突解决、归档、恢复、拆分/合并、冷热分离、规则更新 | 清洗入库、生成报告、质量度量/maturity更新、向量补同步 |

> 关键决策：先确认方案 → 再写授权 → 执行。非关键写操作：写授权 → 执行。纯检索：直接执行（只读豁免）。

## E. 规则加载策略

- **每个操作必读**：本文件 + 对应意图的主规则文件（C 表）
- **按需追加**：
  - 涉及血缘 → rules/lineage-rules.md
  - 涉及分节/表达 → rules/writing-rules.md
  - 涉及质量度量 → rules/quality-rules.md
  - 涉及检索来源标注 → rules/source-of-truth.md
  - 涉及边界判断 → rules/hard-boundaries.md
  - 涉及原子化/类型 → rules/single-responsibility.md、rules/source-types.md
  - 涉及检索维度/混合检索/tags → rules/search-rules.md
  - 涉及 INDEX/MOC → maintenance/navigation-maintenance.md
  - 涉及附件 → maintenance/asset-maintenance.md
  - 涉及冲突 → maintenance/conflict-resolution.md
  - 涉及影响范围 → maintenance/impact-analysis.md
  - 涉及成熟度可视化/发布 → rules/maturity-display.md、rules/publish-rules.md
  - 生成知识页 → templates/wiki-page-template.md
- **缓存依赖**：规则文件缓存依赖 SOLO 的上下文管理能力，单任务内不重复加载

## F. 容错总则

| 场景 | 降级 |
|---|---|
| 向量 MCP 不可用 | 降级全文搜索（filesystem 遍历 + YAML 解析） |
| 文件写成功、向量失败 | 标记 pending_sync，下次巡检补偿 |
| 文件写失败 | 整体失败，记日志，不更新向量 |
| LLM 输出违规 | 标记 draft，不回滚已写资产，提示复核 |
| 检索空/低质量 | 全文搜索补充；仍无→明确告知"未找到" |

**核心**：不回滚已写入的知识资产；无依据不臆测；未授权不写文件。

## G. 向量操作调用规范（scripts/kb.py）

向量索引的增删改查由 `scripts/kb.py` 统一执行（职责分离：**kb.py 只管向量+SQLite日志，md 文件读写仍走文件系统 MCP + 授权门**）。
AI 在各工作流的"⑧更新向量"步骤，调用对应子命令。

**调用前缀**（用知识库内 venv 的 python，路径基于知识库根）：
```
<KB>/.venv/Scripts/python.exe <KB>/scripts/kb.py <子命令> [参数]
```

| 工作流（C表意图） | 时机 | 命令 | 写授权 |
|---|---|---|---|
| 检索对话 | 检索时（只读） | `kb.py search "<查询>" [--topk N] [--status active] [--type/--domain]` | 免（只读）|
| 提纯-ID检查 | 提纯前**强制**（只读）| `kb.py check-ids`（扫描已有 kb_id，报告冲突/缺口/下一个可用 ID）| 免（只读）|
| 提纯-查重 | 提纯前判重（只读） | `kb.py dedup --text "<摘要>"` 或 `--file <md> [--exclude <kb_id>]` | 免（只读）|
| 提纯-检索已有知识 | 提纯前**强制**（只读）| **两级检索**：主题域预检索 `kb.py search "<核心概念>" --topk 10`（每批 3-5 次）+ 每页按需补充（见 wiki-page-lifecycle §2.1）| 免（只读）|
| 提纯/知识更新 | md 写盘**成功后** | `kb.py index <md路径>`（或 `--all` 全量重建）| 需（属写操作）|
| inbox→sources 转换 | inbox 原始文件（.md/.ipynb等）→ 标准 Raw | `ingest_inbox.py`（批量）或手动转换单个文件 | 需（写 sources + pending_raw）|
| 清洗入库（向量化）| Raw 写盘成功后 | `kb.py ingest-source <raw_md路径>`（单文件）或 `batch-ingest`（批量）| 需 |
| 提纯状态流转 | 提纯开始/定稿/失败时 | `kb.py pending <raw_id> --status <processing\|done\|failed>` | 需（写 pending_raw + log_entries）|
| 归档 | md 移入 03-archive 后 | `kb.py archive <kb_id> [--original-path <路径>]` | 需（关键决策）|
| 恢复 | md 移回 02-knowledge 后 | `kb.py restore <kb_id> --file <md路径>` | 需（关键决策）|
| 巡检 | 体检时（只读） | `kb.py audit`（检出：isolated/related_only/no_source_refs/**yaml_filename_format**/**wikilink_to_raw**）、`kb.py status` | 免（只读）|

> ⚠️ **提纯语言要求**：知识页（02-knowledge）正文主体**必须为中文**。英文 raw 提纯时翻译为中文正文，英文专有概念保留原词并附中文说明，代码/命令保留原文。详见 `writing-rules.md §1` 语言行 + `wiki-page-lifecycle.md §2` 定稿第 5 条。

> ⚠️ **outputs 生成要求**：生成 outputs 前必读 `templates/output-report-template.md`。文件**必须**满足：① frontmatter 4 字段（generated / query / source_refs / format）齐全；② 文件名 `output-<YYYYMMDDTHHMMSS>-<slug>.md`；③ 末尾来源标注用 Obsidian 双链 `[[kb-id_标题|标题]]` + `> 来源：` + `> 生成时间：`。详见 `outputs-maintenance.md §1`。

**批量场景（>10 文件，必用）**：大批量处理**禁止逐文件手动转换+向量化**，改用两步批处理：

**第一步：批量转换（秒级，无外部依赖）**

| 场景 | 命令 | 说明 |
|---|---|---|
| inbox→sources 转换 | `python scripts/ingest_inbox.py <inbox目录> [--sources-dir <目录>] [--db <logs.db>] [--dry-run]` | 自动处理 .md/.zh.md（去SVG噪音+换frontmatter）、.ipynb（跳outputs转md）；PDF/图片等路由到外部工具（预留） |

**第二步：批量向量化（~15分钟/200文件）**

| 场景 | 命令 | 说明 |
|---|---|---|
| 批量清洗入库 | `kb.py batch-ingest <sources目录> [--resume] [--force] [--batch-size 64]` | 增量检测（默认）：只处理新增+内容变更文件；`--force` 全量重建 |
| 批量提纯/重索引 | `kb.py batch-index <knowledge目录> [--resume] [--force] [--batch-size 64]` | 增量检测（默认）：只处理新增+内容变更文件；`--force` 全量重建 |
| 清理孤儿向量 | `kb.py cleanup-vectors --collection <名称>` | 删除文件已不存在但向量残留的 kb_id |

- **增量检测（默认）**：基于 `_meta/file_manifest.json` 的 mtime+hash 两级检测，只对新增+内容变更文件 embedding；改 YAML 关联字段不触发重建（hash 只算正文 body）
- **`--force`**：强制全量重建（忽略 manifest），模型升级/chunk 策略变更时用
- **`--resume`**：断点续传，跳过 log_entries 中已成功的 kb_id（中途失败重跑用）；和增量检测可共存
- **`--dry-run`**（ingest_inbox）：只预览不写文件
- 自动补全 `collected`/`language`（无需 LLM）；输出含 `failed` 清单
- 实测：95篇增量检测<1秒（全量重建~15分钟）；219文件转换<30秒

**关键约束**：
1. **文件优先**：先经授权门写/移 md 文件，**文件成功后**才调 kb.py 同步向量（对齐铁律3）。
2. **幂等**：`index`/`ingest-source` 按 `kb_id` 先删旧 chunk 再写，重复执行安全。
3. **分 chunk**：知识页正文按 `## 小节` 切分向量，`search` 返回命中的具体小节（section 字段）。
4. **输出 JSON**：所有命令输出 JSON，AI 解析后据此回答/决策。
5. **日志自动写**：写类命令自动写 `log_entries`，`search` 自动写 `search_log`，无需 AI 额外操作。
6. **向量失败不阻断**：kb.py 报错时，按 F 容错——文件已成功则标记 pending_sync，下次巡检补偿。
7. **嵌入式单连接**：kb.py 每次调用独立开关 Qdrant，AI 不要并发多个 kb.py 写同一库。

### 批量提纯加速指引（>20 篇时必读）⭐

大批量提纯时，按以下策略优化执行，**不缩减质量**：

| 策略 | 做法 | 效果 |
|---|---|---|
| **①合并主题域** | 相关小主题合并为大主题域一次提纯（如 语义搜索+高级检索+高级RAG → "检索与RAG系统"）| 提纯批次 11→4，减少启动/协调开销 |
| **②两级检索** | 主题域开始时 search 3-5 个核心概念生成"关联地图"共享给子代理；每页按需补充 search（见 wiki-page-lifecycle §2.1）| search 次数 95→~25，减少 70% |
| **③子代理并行** | 4-5 个子代理并行，每个处理 5-6 页；子代理只写 md 不调 kb.py（避免锁冲突）| 并行度 2-3→4-5，耗时减半 |
| **④统一向量化** | 所有知识页写完后一次性 `batch-index 02-knowledge`（增量检测只处理新文件）| 向量化 4→1 次 |
| **⑤批量 pending** | `kb.py pending <raw_id> --status done --batch "raw-001,raw-002,raw-003"` 一次标记多个 | 省去逐个调用的进程启动开销 |

**执行顺序**：
```
1. 主题域预检索 → 生成关联地图
2. 子代理并行生成知识页（md 写盘）
3. 所有 md 写完 → batch-index 02-knowledge（一次性向量化）
4. batch pending --status done（一次性标记）
5. batch pending --status processing（下一批开始时）
```


## 目录结构速查

```
KnowledgeBase/
├── AGENTS.md        本文件（根目录）
├── 00-inbox/        临时接收
├── 01-sources/      Raw（含 _assets/ 附件）
├── 02-knowledge/    知识页（active 主体）+ INDEX.md + <domain> MOC
├── 03-archive/      归档
├── 04-outputs/      AI 产出
├── _meta/
│   ├── logs.db          SQLite（日志+pending_raw+log_archive+search_log）
│   ├── rules/           核心规则（声明性）
│   ├── maintenance/     维护流程（操作性）
│   └── templates/       模板
└── .qdrant/         向量库（knowledge/sources/archive/tags）
```

---
> Source: [dengpeng1985-maker/Gleanlight](https://github.com/dengpeng1985-maker/Gleanlight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-30 -->
