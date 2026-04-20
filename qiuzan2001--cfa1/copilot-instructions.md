## cfa1

> > 简洁的 CFA Level 1 学习 vault。用户按教材顺序 (V1→V10) 阅读 PDF，Agent 辅助生成笔记和追踪习题。

# CFA L1 Vault — Claude Code 规则

> 简洁的 CFA Level 1 学习 vault。用户按教材顺序 (V1→V10) 阅读 PDF，Agent 辅助生成笔记和追踪习题。

核心循环：**阅读教材 → 宽笔记 → 遇到问题 → 详细笔记 → 做题 → 追踪**

---

## 1. 设计原则

1. **教材为主**: 用户的主要学习时间在阅读 PDF 教材上，Agent 是辅助
2. **按需展开**: 只在用户开始某周学习时生成宽笔记，只在用户提问时生成详细笔记
3. **严格贴合教材**: 宽笔记和详细笔记必须严格跟随教材结构和内容
4. **Obsidian 原生**: 充分利用 Obsidian 原生功能——Bases、Callouts、Properties、Wikilinks、Embeds、Canvas、Mermaid
5. **学习收尾要可审计**: 当用户说今天学完了，应优先基于 Git 变化而不是模糊记忆来判断今天实际学了什么

---

## 2. Vault 结构

```
Home.md              — Dashboard（含 callouts、embeds）
Plan.md              — 21 周计划（带 frontmatter）
Glossary/            — 术语库
  Glossary.base      — 数据库视图（table/cards，按科目分组）
  README.md          — 说明文档
  Library/
    *.md             — 每个术语一个笔记（tag: term）
Formulas/            — 公式库
  Formulas.base      — 数据库视图（table/cards，按科目分组）
  README.md          — 说明文档
  Library/
    *.md             — 每个公式一个笔记（tag: formula）
Practice/            — 习题追踪
  Practice.base      — 数据库视图（错题本/全部/标星/按错误类型）
  README.md          — 说明文档
  V{n}-{Subject}/Week{X}/
    Week-{XX}-LM{nn}-Practice.md  — LM 习题集合 note
    Week-{XX}-LM{nn}-Q{qq}.md     — 每道题一个笔记（tag: question）
V1-Quant/ ... V10-Ethics/  — 各 Volume 笔记
  Week1/ Week2/ ...        — 每周子文件夹
    Week-{XX}.md           — week root
    Week-{XX}-LM{nn}-*.md  — LM 子笔记
    {知识点}.md            — 仅在需要时创建的详细笔记
PDFs/                — 教材 PDF
scripts/             — 工具文件
  agent-log.md       — 学习日志
  session.ctx        — 当前学习状态
```

---

## 3. 快速工作流路由

| 用户说 | 执行 |
|--------|------|
| `今天学什么` / `开始第 X 周` / `本周学什么` | 查 `Plan.md` → 创建 week root + LM 子笔记 → 参考 `.claude/vault-skills/start-week-note/SKILL.md` |
| `开始LM01` / `开始 LM1` | 创建/更新 LM 子笔记 + 同步收集教材 Practice Problems → 参考 `.claude/vault-skills/build-lm-note/SKILL.md` + `collect-lm-practice/SKILL.md` |
| `补全 LM3` / `生成 LM3 笔记` | 创建/更新 LM 子笔记 → 参考 `build-lm-note/SKILL.md` |
| `检查这周有没有漏内容` | 对照 Plan.md + PDF TOC 审计 → 参考 `audit-week-coverage/SKILL.md` |
| 问某个概念 / `展开这个知识点` | 创建详细笔记 → 参考 `expand-topic-note/SKILL.md` |
| `这题不懂` / `这道题我还是不理解` | 先在 question note 写完整解析，必要时补充详细笔记 → 参考 `expand-topic-note/SKILL.md` |
| `润色这篇笔记` | 保持范围，提升质量 → 参考 `polish-cfa-note/SKILL.md` |
| `添加术语` / `提取术语` | 创建 `Glossary/Library/{Term}.md` → 参考 `extract-glossary-terms/SKILL.md` |
| `添加公式` / `提取公式` | 创建 `Formulas/Library/{Formula}.md` → 参考 `extract-formulas/SKILL.md` |
| `润色这个术语` / `润色这个公式` | 润色库条目 → 参考 `polish-memory-note/SKILL.md` |
| `结束今天学习` / `今天学完了` | 基于 git 变化做学习收尾 → 参考 `close-study-session/SKILL.md` |
| `复习一下` / `今天复习什么` | 间隔复习 → 参考 `spaced-review/SKILL.md` |
| `更新进度` / `同步进度` | 扫描 vault 更新 Home/Plan 进度 → 参考 `sync-progress/SKILL.md` |
| `我哪里最弱` / `分析错题` | 分析薄弱点 → 参考 `weak-spot-analyzer/SKILL.md` |
| `开始学习` / `热身` | 推送热身卡 → 参考 `daily-warmup/SKILL.md` |

---

## 4. Skill 调用速查

### Vault Local Skills（`.claude/vault-skills/`）

| 需要做什么 | Skill 路径 |
|-----------|-----------|
| 开始一周学习 | `.claude/vault-skills/start-week-note/SKILL.md` |
| 创建/补全某个 LM 笔记 | `.claude/vault-skills/build-lm-note/SKILL.md` |
| 收集某个 LM 的教材习题 | `.claude/vault-skills/collect-lm-practice/SKILL.md` |
| 审计某周覆盖完整性 | `.claude/vault-skills/audit-week-coverage/SKILL.md` |
| 展开一个知识点为详细笔记 | `.claude/vault-skills/expand-topic-note/SKILL.md` |
| 润色现有笔记 | `.claude/vault-skills/polish-cfa-note/SKILL.md` |
| 从笔记提取术语 | `.claude/vault-skills/extract-glossary-terms/SKILL.md` |
| 从笔记提取核心公式 | `.claude/vault-skills/extract-formulas/SKILL.md` |
| 润色术语/公式库条目 | `.claude/vault-skills/polish-memory-note/SKILL.md` |
| 结束当天学习并总结 | `.claude/vault-skills/close-study-session/SKILL.md` |
| 间隔复习术语/公式/错题 | `.claude/vault-skills/spaced-review/SKILL.md` |
| 同步 Home/Plan 进度 | `.claude/vault-skills/sync-progress/SKILL.md` |
| 分析薄弱点和错题模式 | `.claude/vault-skills/weak-spot-analyzer/SKILL.md` |
| 每日学习前热身 | `.claude/vault-skills/daily-warmup/SKILL.md` |

### Obsidian Skills（`.claude/obsidian-skills/skills/`）

| 需要做什么 | Skill 路径 |
|-----------|-----------|
| 写 `.md` 笔记（任何类型） | `.claude/obsidian-skills/skills/obsidian-markdown/SKILL.md` |
| 修改 `.base` 数据库视图 | `.claude/obsidian-skills/skills/obsidian-bases/SKILL.md` |
| 创建 `.canvas` 知识图谱 | `.claude/obsidian-skills/skills/json-canvas/SKILL.md` |
| Obsidian CLI 批量操作 | `.claude/obsidian-skills/skills/obsidian-cli/SKILL.md` |
| 提取网页内容 | `.claude/obsidian-skills/skills/defuddle/SKILL.md` |

---

## 5. 宽笔记 (Wide Notes)

**触发**: 用户说 "今天学什么"、"开始第 X 周"、"本周学什么" 或 "本周学 V1 LM1-6"

宽笔记分为两层：

1. **Week Root**
   - 文件：`V{n}-{Subject}/Week{X}/Week-{XX}.md`
2. **LM Child Note**
   - 文件：`V{n}-{Subject}/Week{X}/Week-{XX}-LM{nn}-{English Slug}.md`

**Week Root Frontmatter**:
```yaml
---
tags: [week-root, wide-note, V1, quant]
volume: V1
subject: Quant
week: 1
lm_range: "LM1–6"
note_role: week-root
---
```

**LM Child Frontmatter**:
```yaml
---
tags: [wide-note, lm-note, V1, quant]
volume: V1
subject: Quant
week: 1
lm: 1
lm_title: "Rates and Returns"
page_range: "3-44"
parent: "[[Week-01]]"
source_pdf: "PDFs/cfa-program2026L1V1.PDF"
---
```

**Week Root 内容要求**:
- week root 负责范围、顺序、LM 出链、公式索引、问题索引
- 它可以有高层摘要，但不应承担全部 LM 的完整正文覆盖
- 必须链接该 week 的每个 LM 子笔记
- 不要预先生成详细笔记链接列表

**LM Child 内容要求**:
- 严格按教材章节结构，覆盖该 LM 的所有知识点
- 先根据 PDF TOC 生成 `## Coverage Checklist`
- 每个 checklist item 都要在正文中被覆盖，或在 `## End Matter` 中明确承认
- 每个知识点 1-3 句话概括，公式用 LaTeX `$$...$$` 列出
- 使用 Obsidian callouts 标注重要内容：
  - `> [!formula]` — 核心公式
  - `> [!warning]` — 易错点 / 常见陷阱
  - `> [!tip]` — 考试技巧 / 记忆窍门
  - `> [!example]` — 关键例题
  - `> [!info]` — 背景知识
- 中文为主，术语保留英文，首次出现格式：**中文 (English Term)**
- 教材页码标注在每个 section 旁边
- LM section / subsection 标题文本本身应直接链接到对应 PDF 页
- 如果用户说的是 "今天学什么"，先根据 `2026-03-25` 这个开始日期和当前日期推算当前计划周，再落到对应的 week note
- 完成后应用 `audit-week-coverage` 检查遗漏
- 不要在 LM note 中预先生成 placeholder detail links；只有用户提问时才出链到详细笔记
- 如果用户是在"开始某个 LM"，还要同步收集该 LM 的教材 Practice Problems 与 Solutions
- 题目链接应尽量挂在对应知识点的小节下面，而不是只在 LM note 底部集中列一个总表

---

## 6. 详细笔记 (Detailed Notes)

**触发**: 用户在学习中提问

**文件**: `V{n}-{Subject}/Week{X}/{知识点名}.md`

**Frontmatter**:
```yaml
---
tags: [detail, V1, quant]
volume: V1
subject: Quant
week: 1
lm: 3
parent: "[[Week-01-LM03-Statistical-Measures-of-Asset-Returns]]"
---
```

**内容要求**:
- 只在用户遇到问题、明确要求展开时创建
- 从对应 LM 子笔记出链（LM note 中添加 `[[知识点名]]` wikilink）
- 每个详细笔记都必须能从对应宽笔记的相关 section 直接点到
- 先阅读相关教材页，再针对该问题写一读就懂的详细讲解
- 若触发来自某道 practice question，题目专属的逐步解析和个人错误记录应优先保留在 question note
- 详细笔记必须独立可读，不应把"这道题 / 某题在问什么"当作主体结构
- 使用 callouts: `[!formula]` `[!warning]` `[!tip]` `[!example]` `[!question]`
- 对比内容用表格，流程用 Mermaid 图
- 顶部要有 `Source` 区块，给出相关 PDF 页链接与父知识点链接
- 公式类知识点要同时解释"符号是什么意思"和"实际做题时怎么一步步拆"
- 在不失焦的前提下，补上合理的外链：相邻知识点、父 LM、最近的相关习题或题集
- 复杂主题可拆分多个 md，用 wikilinks 建立网络

---

## 7. 公式笔记

**触发**: "添加公式: XXX"、"提取公式" 或从某篇笔记中沉淀高价值公式

**文件**: `Formulas/Library/{English Formula Name}.md`

**Frontmatter**:
```yaml
---
tags: [formula, new]
chinese: "中文名称"
subject: Quant
volume: V1
lm: 1
importance: high  # high / medium / low
mastery: new      # new / reviewing / solid
formula_family: "return-measure"
parent: "[[Week-01-LM01-Rates-and-Returns]]"
latex: "EAR = (1 + r_s/m)^m - 1"
---
```

**正文要求**:
- 一个公式 note 只服务一个可复用的公式或公式族
- 默认新抽取条目使用 `mastery: new`
- 顶部先给渲染后的公式本体，再解释变量含义
- 明确写清楚 `什么时候用 / 什么时候别用`
- 若存在常见变体，应在同一 note 中并列说明
- 至少补上一个最小例子或和相近公式的区别
- 关联父 LM、相邻知识点、相关题目或详细笔记
- 优先更新已有公式 note，避免重复创建

Formulas.base 自动从 tag:formula 的笔记构建数据库视图。

---

## 8. 术语笔记

**触发**: "添加术语: XXX" 或生成笔记时遇到重要术语

**文件**: `Glossary/Library/{English Term}.md`

**Frontmatter**:
```yaml
---
tags: [term, new]
chinese: "中文释义"
subject: Quant
volume: V1
lm: 3
importance: high  # high / medium / low
mastery: new      # new / reviewing / solid
---
```

**正文**: 详细释义、用法、例句、易混淆对比。使用 wikilinks 链接相关术语和知识笔记。默认新抽取条目使用 `mastery: new`。

Glossary.base 自动从 tag:term 的笔记构建数据库视图。

---

## 9. 题目笔记

**触发**: 用户开始某个 LM、报告做题结果或问某道题

分两层：

1. **LM Practice Set**
   - 文件：`Practice/V{n}-{Subject}/Week{X}/Week-{XX}-LM{nn}-Practice.md`
2. **Question Note**
   - 文件：`Practice/V{n}-{Subject}/Week{X}/Week-{XX}-LM{nn}-Q{qq}.md`

**Question Note Frontmatter**:
```yaml
---
tags: [question, textbook-practice, unattempted]
volume: V1
week: 1
lm: 3
number: 5
subject: Quant
result: unattempted            # unattempted / wrong / correct
source_type: textbook_practice
question_page: 120
solution_page: 124
parent_week: "[[Week-01]]"
parent_lm: "[[Week-01-LM03-...]]"
practice_set: "[[Week-01-LM03-Practice]]"
linked:
  - "[[Week-01-LM03-...#Some Heading]]"
---
```

如果用户明确说"这道题不懂 / 还是不理解"，在该题的 tags 中追加 `unclear`。

**Question Note 正文**: 必须保留教材中的英文原题，并记录题目页/解答页 PDF 跳转、中文提示、正确答案、解题逻辑、关联知识点、作答结果。其中提示、答案、知识链接应使用默认折叠的 callout。

**错误分类**: concept_misunderstanding, formula_confusion, calculation_error, wording_trap, comparison_confusion, rushed_guess, memory_failure

Practice.base 自动从 tag:question 的笔记构建错题本。

---

## 10. 学习收尾规范

**触发**: 用户说 "今天学完了"、"完成学习"、"结束今天学习"、"总结今天的单词和公式"

**要求**:
- 优先根据 Git 变化确认今天改过哪些学习内容
- 先整理今天改过的 week / LM / detail / practice / glossary / formula 笔记
- 补 weak links、重复条目、遗漏的库沉淀
- 对今天新增的 glossary / formula 条目默认使用 `mastery: new`
- 输出一份简短复习清单，聚焦 `new / reviewing`
- 完成后做基本 git 检查，并在 clean 状态下 commit / push

---

## 11. Obsidian 功能使用规范

### Callouts

```markdown
> [!formula] 核心公式
> $$PV = \frac{FV}{(1+r)^n}$$

> [!warning] 易错点
> LIFO 在 IFRS 下不允许使用

> [!tip] 考试技巧
> Duration 近似公式记忆窍门...

> [!example] 例题
> 某债券面值 1000...

> [!question] 理解检查
> 为什么 YTM 上升时债券价格下降？
```

### Wikilinks
- 笔记间链接用 `[[Note Name]]`
- 指向特定 heading 用 `[[Note#Heading]]`
- 详细笔记只在用户提问后，从 LM 子笔记出链

### Embeds
- 在笔记中嵌入其他笔记内容 `![[Note Name]]`
- 嵌入特定 section `![[Note#Section]]`
- 嵌入 Base 视图 `![[Practice.base#错题本]]`

### Properties (Frontmatter)
- 所有笔记都必须有 frontmatter
- tags 用于 Base 筛选
- subject, volume, lm 用于分类

### Mermaid
- 流程和决策用 Mermaid 图嵌入笔记
- 复杂多节点网络用 Canvas

### Canvas
- 如需可视化某科目的知识结构，可创建 `.canvas` 文件
- Canvas 只做导航，不存储知识内容

---

## 12. 语言规范

- 正文中文，所有 CFA 术语保留英文原文
- 首次出现：**中文名 (English Term)**
- 公式 LaTeX: `$$...$$` 块级，`$...$` 行内
- 确保足够英文量（考试是英文）

---

## 13. 考试信息

- 考试：CFA Level 1
- 目标日期：2026年8月
- 开始日期：2026-03-25
- 每周可用时间：~20小时
- 教材：CFA Program 2026 L1 V1-V10 (PDF)
- 总计：10 科目，94 Learning Modules，21 周计划

---

## 14. 间隔复习规范

### Frontmatter 字段

`Glossary/Library/*.md` 和 `Formulas/Library/*.md` 新增：

```yaml
last_reviewed: 2026-04-06
```

### Mastery 转换规则

| 当前 | 用户说"掌握了" | 用户说"还在复习" | 用户说"还不太稳" |
|------|--------------|-----------------|-----------------|
| `new` | → `reviewing` | → `reviewing` | 保持 `new` |
| `reviewing` | → `solid` | 保持 `reviewing` | → `new` |
| `solid` | 保持 `solid` | → `reviewing` | → `reviewing` |

### 间隔算法

| Mastery | 间隔 | 条件 |
|---------|------|------|
| `new` | 0 天 | 始终包含 |
| `reviewing` | 3 天 | last_reviewed 3天前或缺失 |
| `solid` | 14 天 | last_reviewed 14天前或缺失 |

详细算法参见：`.claude/vault-skills/spaced-review/references/review-algorithm.md`

### Tag 同步

mastery 变更时，同步更新 frontmatter 中的 tag。

---

## 15. 进度同步规范

### LM 状态映射

| 条件 | 状态 |
|------|------|
| LM note 不存在 | ⬜ |
| LM note 存在，practice 未收集 | 🔄 |
| LM note 存在，practice 已收集 | ✅ |

### 周状态映射（Home.md）

| 条件 | 状态 |
|------|------|
| 无 LM note | ⬜ |
| 部分 LM 有笔记 | 🟡 |
| 全部 LM 有笔记且 practice 已收集 | ✅ |

### 检测方法

- LM note: 检查 `V{n}-{Subject}/Week{X}/Week-{XX}-LM{nn}-*.md` 是否存在
- Practice set: 检查 `Practice/V{n}-*/Week{X}/*-LM{nn}-Practice.md` 是否存在
- 只修改状态 emoji，不改变表格其他内容

---

## 16. Skill 参考文档索引

执行具体工作流时，查阅以下 SKILL.md 获取详细步骤：

- **Vault 工作流**：`.claude/vault-skills/{skill-name}/SKILL.md`
- **Obsidian 文件操作**：`.claude/obsidian-skills/skills/{skill-name}/SKILL.md`
- **间隔复习算法**：`.claude/vault-skills/spaced-review/references/review-algorithm.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qiuzan2001) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
