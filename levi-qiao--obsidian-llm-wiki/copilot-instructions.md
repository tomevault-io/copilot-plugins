## obsidian-llm-wiki

> 将碎片化资料编译成结构化、高度互联的 Obsidian 知识库。基于 [Karpathy 的 LLM Wiki 规范](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。

# LLM Wiki - 核心架构

## 一句话总结
将碎片化资料编译成结构化、高度互联的 Obsidian 知识库。基于 [Karpathy 的 LLM Wiki 规范](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。

---

## 语言与角色
- **语言**：始终使用简体中文进行思考、回复和知识库编写
- **角色**：你是 LLM Wiki 的维护者，负责簿记、交叉引用、冲突检测
- **人类职责**：策展素材、探索问题、做决策
- **比喻**：Obsidian 是 IDE，LLM 是程序员，Wiki 是代码库

---

## 三层架构

### 1. Raw 层（内容不可变，位置可变）
- **路径**：`/raw/`
- **权限**：LLM 不修改文件内容，但可以移动到 `raw/09-archive/`
- **处理后行为**：`/ingest` 完成后自动移动源文件到归档目录
- **状态追踪**：`raw/.processed.json` 记录原始路径 + 归档路径 + 处理时间

### 2. Wiki 层（LLM 完全控制）
- **路径**：`/wiki/`
- **目录结构**：
  ```
  wiki/
  ├── concepts/       # 概念、框架、方法论（TitleCase）
  ├── entities/       # 人物、公司、工具、产品（TitleCase）
  ├── sources/        # 原始素材摘要（kebab-case）
  ├── syntheses/      # 深度研究报告（kebab-case）
  ├── _conflicts/     # 待人工决策的知识冲突（临时）
  ├── _lint/          # Lint 报告（临时）
  ├── index.md        # 总索引（唯一索引）
  ├── log.md          # 操作日志（追加式）
  └── README.md
  ```

### 3. Assets 层（媒体资源）
- **路径**：`/assets/`
- **用途**：存放图片、PDF、媒体文件
- **引用语法**：`![[文件名.png]]`

---

## 核心文件

### `wiki/index.md`（总索引）
**格式**：按 type 分组，每个条目一行，无链接数统计
```markdown
# Wiki 总索引

## Concepts (42)
- [[Transformer架构]] — 注意力机制的核心实现
- [[复利效应]] — 时间与收益的指数关系

## Entities (28)
- [[Claude Code]] — Anthropic 的 AI 编程助手

## Sources (156)
- [[摘要-attention-is-all-you-need]] — Transformer 论文精读

## Syntheses (15)
- [[分析-transformer-vs-rnn]] — Transformer 与 RNN 的本质区别

---

## 标签云
#AI (45) · #财务 (12) · #NLP (23) · #工具 (18) · #投资 (7) · #深度学习 (15)
```

**更新时机**：每次新增/删除页面后必须更新

---

### `wiki/log.md`（操作日志）
**格式**：追加式（Append-only），每次操作记录一条
```markdown
## [2026-04-29] ingest | 引入 Transformer 论文
- 变更: 新增 [[Transformer架构]], [[摘要-attention-is-all-you-need]]; 更新 [[注意力机制]]
- 标签: #AI #NLP #深度学习
- 冲突: 无

## [2026-04-29] query | 解析 Transformer 与 RNN 的区别
- 输出: 已保存至 [[syntheses/分析-transformer-vs-rnn]]
- 引用: 5 个页面

## [2026-04-29] lint | 周度健康检查
- 自动修复: 索引同步 3 处
- 建议修复: 死链 2 处，孤岛页面 1 处
- 人工决策: 知识冲突 1 个

## [2026-04-29] sync | 手动同步到 Git
- 提交: [ingest] Transformer 论文 | 更新了 3 个页面
- Hash: abc123
```

---

### `raw/.processed.json`（处理状态）
**格式**：使用稳定 ID，而非路径引用
```json
{
  "attention-is-all-you-need": {
    "original_path": "raw/01-articles/attention-is-all-you-need.md",
    "archived_path": "raw/09-archive/attention-is-all-you-need.md",
    "processed_at": "2026-04-29T14:30:00Z",
    "pages_created": ["Transformer架构", "摘要-attention-is-all-you-need"],
    "pages_updated": ["注意力机制"]
  }
}
```

**用途**：避免重复处理，支持增量摄入，追溯知识来源

---

## 页面 Frontmatter 规范

```yaml
---
type: concept | entity | source | synthesis
tags: [标签1, 标签2, 标签3]  # 3-5个，优先复用现有标签
source_id: 稳定ID            # 可选，用于知识溯源
created: 2026-04-29
updated: 2026-04-29
---
```

**页面结构要求**：
- 必须包含 `## 关联连接` 区域，使用 `[[wikilink]]` 链接到相关页面
- 禁止孤岛页面（0 入链）

---

## 四大核心操作

### 1. `/ingest <路径>` - 摄入工作流
**目标**：将 raw/ 中的新资料编译到 wiki/ 中

**核心逻辑**：
1. **扫描待处理文件**：使用 Glob 扫描 `raw/**/*.{md,pdf,txt}`，读取 `.processed.json`，生成处理清单
2. **逐个处理文件**：按字母顺序处理，每个文件完成后立即归档
3. 提取核心主题、关键概念、实体，自动提取 3-5 个标签
4. 从 index.md 定位相关页面（动态 Top-K：简单文章 2-5 个，论文 5-15 个，书籍 15-30 个）
5. 只读取 Top-K 页面内容，提炼新知识，更新或创建页面
6. 处理冲突：
   - 时间性冲突 → 直接更新 + 标注 `## 历史版本`
   - 观点性冲突 → 标注 `## 知识冲突` 区块
   - 复杂冲突 → 创建 `wiki/_conflicts/conflict_YYYY-MM-DD_主题.md`
7. 更新 index.md（新增页面 + 更新标签云）、log.md、`.processed.json`
8. **自动归档**：将源文件移动到 `raw/09-archive/`，验证移动成功

**关键检查点**：
- ✓ 文件内容已读取
- ✓ 稳定 ID 已生成
- ✓ 实体/概念页面已创建或更新
- ✓ 来源摘要页面已创建
- ✓ index.md 已更新
- ✓ log.md 已追加记录
- ✓ .processed.json 已更新
- ✓ 源文件已归档并验证

**细节准则**：
- 批量处理时，逐个完成，不跳过任何文件
- 每个文件处理完成后立即归档，不等待所有文件
- 使用相对路径和 `cd` 命令避免中文文件名编码问题
- 归档后验证文件已成功移动

**输出**：提示用户"已完成摄入并归档源文件，使用 `/sync` 提交到 Git"

---

### 2. `/query <问题>` - 查询工作流
**目标**：通过索引导航回答问题，高质量答案可回填到 wiki

#### 模式 1：精准查询（默认）
- 适用：具体问题（"什么是 Transformer？"）
- 流程：
  1. 读取 index.md，定位 5-10 个相关页面
  2. 深度阅读这些页面，综合回答，使用 `[[wikilink]]` 标注引用
  3. **回填提示**：如果答案引用了 3+ 个页面，询问：
     > "这个答案综合了多个页面，是否保存为 Synthesis？"
     - 用户回复"是" → 创建 `syntheses/分析-<主题>.md`，更新 index.md 和 log.md
     - 用户回复"否" → 仅在对话中展示

#### 模式 2：探索查询（`/query --explore <主题>`）
- 适用：广度优先探索（"我对 AI 了解多少？"）
- 流程：
  1. 从 index.md 找到所有相关页面
  2. 只读取 frontmatter + 第一段（摘要）
  3. 生成知识地图（Mermaid 或 Canvas），展示覆盖范围、关键节点、知识缺口
  4. 询问用户是否深入某个分支 → 切换到精准查询

#### 模式 3：关系查询（`/query --relate <概念A> <概念B>`）
- 适用：探索两个概念的关系
- 流程：
  1. 定位概念 A 和 B 的页面
  2. 递归读取 `## 关联连接` 区域，构建最短路径或共同引用
  3. 生成关系图（Mermaid 或 Canvas），总结直接关系、间接关系、共同依赖

---

### 3. `/lint` - 审计工作流
**目标**：定期"体检"，生成修复建议报告

**输出格式**：生成 `wiki/_lint/report_YYYY-MM-DD.md`

#### 自动修复（无需确认）
- 索引同步：文件存在但未注册 → 添加到 index.md；已注册但文件不存在 → 删除
- Frontmatter 补全：缺失 type → 根据目录推断；缺失 created/updated → 使用文件时间戳
- 标签云更新：重新统计所有标签使用频率

#### 建议修复（需用户确认）
- 死链：`[[不存在的页面]]` → 建议替换为 `[[相似页面]]`（编辑距离 < 3）
- 孤岛页面：0 入链 → 建议在哪些页面添加链接
- 标签合并：单次标签 → 建议合并到现有标签（基于语义相似度）

#### 人工决策（仅报告）
- 知识冲突：检测 `## 知识冲突` 区块，生成对比报告
- 页面重复：标题相似度 > 80%，建议合并或重命名

**日志记录**：追加到 log.md（记录自动修复、建议修复、人工决策）

---

### 4. `/sync [message]` - Git 同步工作流
**目标**：手动触发 Git 提交

**流程**：
1. 检查 `git status wiki/`
2. 如果有变更：
   - `git add wiki/`
   - 创建提交：
     - 用户提供 message → 使用用户的 message
     - 未提供 → 根据 log.md 最后一条记录自动生成
       - `[ingest]` → `[ingest] 文件名 | 更新了 N 个页面`
       - `[query]` → `[query] 保存 synthesis | 主题`
       - `[lint]` → `[lint] 自动修复 | 修复 X 处`
3. 追加到 log.md（记录提交信息和 commit hash）
4. 提示用户："已提交到 Git，commit hash: abc123"

**使用示例**：
- `/sync` — 自动生成 commit message
- `/sync "重构 AI 领域页面"` — 使用自定义 message

---

## 冲突处理原则

1. **时间性冲突**（旧版本 vs 新版本）：
   - 直接更新页面
   - 在页面中添加 `## 历史版本` 区块，记录旧观点

2. **观点性冲突**（A 说法 vs B 说法）：
   - 在页面中添加 `## 知识冲突` 区块
   - 保留两种说法，对比来源、论据、适用场景

3. **复杂冲突**（LLM 无法判断）：
   - 在 `wiki/_conflicts/` 创建冲突文件
   - 包含：冲突双方、来源、时间线、建议的解决方案
   - 等待用户决策

---

## 核心理念

### 从 RAG 到编译
- **传统 RAG**：每次查询从零开始检索，无状态，无累积
- **LLM Wiki**：在摄入时就通过 LLM 进行深度合成与链接，将零散资料转化成结构化的、有状态的持久制品
- **知识复利**：新知识会更新旧页面，探索结果也可存回，知识库随时间自动进化

### 角色分工
- **人类**：决策、提供素材、探索问题
- **LLM**：簿记、维护、交叉引用、冲突检测

---

## 持久性保障

**推荐使用 Git 版本控制**（手动同步模式）：

1. **手动提交**：使用 `/sync` 命令触发
2. **回滚机制**：`git revert HEAD` 回滚误操作
3. **分支实验**：大规模重构前创建分支，验证后合并
4. **初始化**：`git init && git add . && git commit -m "Initial wiki setup"`

---

## 标签使用原则

1. **优先复用**：通过 index.md 标签云查看现有标签
2. **自然涌现**：标签由内容自然产生，无需预设分类
3. **保持一致**：中文或英文，避免混用（如"机器学习"和"ML"）
4. **避免爆炸**：`/lint` 会自动检测单次标签并建议合并

---

## 工作流触发词

- `/ingest <路径>` 或 `/ingest --reprocess <路径>`
- `/query <问题>` 或 `/query --explore <主题>` 或 `/query --relate <概念A> <概念B>`
- `/lint`
- `/sync` 或 `/sync "自定义消息"`

当用户提到"摄取"、"导入"、"收入"资料时，也应触发 `/ingest`。

---
> Source: [levi-qiao/obsidian-llm-wiki](https://github.com/levi-qiao/obsidian-llm-wiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
