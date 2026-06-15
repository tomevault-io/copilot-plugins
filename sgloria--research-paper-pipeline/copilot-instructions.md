## research-paper-pipeline

> |


# Research Paper Writing Skill

7-Agent 管线，将一个粗糙的研究想法变成一篇投稿就绪的顶会论文。

## 快速开始

用户可能给出以下任意一种输入：
- 一个研究方向/关键词（如 "BLE wearable + mental health"）
- 一个具体的 idea 描述
- 一份已有的实验数据/代码
- 一篇写了一半的论文草稿
- 只想做某个阶段（如 "帮我做 literature review"、"帮我模拟审稿"）

**判断入口点**：根据用户输入判断应该从哪个阶段开始，不必每次都从头跑完整管线。

| 用户意图 | 入口 Agent | 说明 |
|---------|-----------|------|
| 给了方向/关键词 | Idea Agent | 全流程 |
| 给了具体 idea | Literature Agent | 跳过选题 |
| 给了 idea + 文献 | Method Agent | 跳过前两步 |
| 给了方法 + 实验设计 | Writing Agent | 直接写 |
| 给了完整论文草稿 | Review Agent | 只做审稿 |
| 给了论文 + reviews | Rebuttal Agent | 只做 rebuttal |

## 工作目录初始化

确认研究主题后，第一件事是创建工作目录：

```bash
mkdir -p "[项目名称]"/{sections,figures/svg_output,figures/output,data,output}
```

目录结构：
```
[项目名称]/
  research_state.json      # 全局状态文件（断点续跑）
  idea_output.json         # Idea Agent 输出
  literature_pool.json     # 文献池
  references.bib           # BibTeX 文件
  sections/                # 各 section 的 Markdown 草稿
    abstract.md
    introduction.md
    related_work.md
    method.md
    experiments.md
    conclusion.md
  figures/                 # 图表文件
    figure_list.json       # 图表清单（id/type/description/status）
    svg_output/            # SVG 源文件
    output/                # 导出产物
      paper_figures.pptx   # 所有 SVG 汇总的 PPTX
  data/                    # 实验数据
  output/                  # 最终产出
    paper.md               # 完整 Markdown 论文
    paper.tex              # LaTeX 版本
    paper.pdf              # 编译后 PDF
    rebuttal.md            # Rebuttal letter
```

## 状态管理

用 `research_state.json` 驱动整个管线，支持断点续跑。每完成一个阶段就更新状态文件。

初始化模板：
```json
{
  "project_name": "",
  "target_venue": "",
  "current_phase": "idea",
  "selected_idea_index": null,
  "phases": {
    "idea": {"status": "pending", "output_path": null},
    "literature": {"status": "pending", "output_path": null},
    "method": {"status": "pending", "output_path": null},
    "experiment": {"status": "pending", "output_path": null},
    "writing": {"status": "pending", "output_path": null},
    "review": {"status": "pending", "output_path": null},
    "rebuttal": {"status": "pending", "output_path": null}
  },
  "created_at": "",
  "updated_at": ""
}
```

每个阶段完成后：
1. 将 status 更新为 `"completed"`
2. 填入 `output_path`
3. 将 `current_phase` 推进到下一阶段
4. 更新 `updated_at` 时间戳

如果用户中途回来继续，读取 `research_state.json` 判断从哪里接续。

## 7-Agent 管线

按顺序执行以下阶段。每个阶段的详细指令在对应的 reference 文件中。

### Phase 1: Idea Agent（选题）

**何时执行**：用户给了一个方向/关键词但没有具体 idea。

**读取详细指令**：[idea-agent.md](references/idea-agent.md)

**摘要**：
1. 搜索领域最新趋势（arXiv API + WebSearch）
2. 识别 Research Gap
3. 生成 3-5 个候选 Idea（含 novelty、contributions、difficulty）
4. 用 Semantic Scholar 做 Novelty Check
5. 让用户选择一个 Idea
6. 输出 `idea_output.json`

### Phase 2: Literature Agent（文献）

**何时执行**：有了具体 idea，需要做文献调研。

**读取详细指令**：[literature-agent.md](references/literature-agent.md)

**摘要**：
1. 多源检索（arXiv + Semantic Scholar + WebSearch）
2. 去重 + 透明排名
3. 引用追踪（Snowball）
4. 分类聚类
5. 撰写 Related Work
6. 4 层引用验证
7. 输出 `literature_pool.json` + `references.bib` + `sections/related_work.md`

### Phase 3: Method Agent（方法设计）

**何时执行**：有了 idea 和文献，需要设计方法。

**读取详细指令**：[method-agent.md](references/method-agent.md)

**摘要**：
1. 分析现有方法的技术路线
2. 设计方法结构 + 框架图
3. 选定 Baseline
4. 撰写算法伪代码
5. 输出 `sections/method.md`

### Phase 4: Experiment Agent（实验）

**何时执行**：方法设计完成，需要规划实验。

**读取详细指令**：[experiment-agent.md](references/experiment-agent.md)

**摘要**：
1. 数据集选择 + 评估指标设计
2. 生成 Evaluation Protocol
3. 设计 Ablation Study
4. 生成结果表格模板
5. 如有真实数据，接入并生成实验描述
6. 输出 `sections/experiments.md`

### Phase 5: Writing Agent（写论文）

**何时执行**：前 4 个阶段完成，组装完整论文。

**读取详细指令**：[writing-agent.md](references/writing-agent.md)

**核心原则**：Writing Agent 是组装者 + 润色者，不是重写者。已有 section 原样复用 + 定点修补，不重新生成。

**摘要**：
1. **新写** Abstract（~150 词）→ `sections/abstract.md`
2. **新写** Introduction（漏斗结构，~1.5 页）→ `sections/introduction.md`
3. **轻量润色** Related Work / Method / Experiments（StrReplace 定点修改，不重写）
4. **新写** Conclusion + Limitations → `sections/conclusion.md`
5. **纯文件拼接** 组装 `output/paper.md`（不经过 LLM）
6. 一致性检查（术语 / 符号 / 引用 / 图表）

### Phase 5.5: Figures Agent（论文绘图）

**何时执行**：Writing Agent 完成全文组装后，将 `[FIGURE: ...]` 占位符转化为实际图表。也可独立触发。

**读取详细指令**：[figures-agent.md](references/figures-agent.md)

**核心流程**：先生成 SVG 矢量图，再统一嵌入到一个 PPTX 文件中。

**摘要**：
1. 扫描论文中所有 `[FIGURE: ...]` 占位符，生成图表清单
2. 按类型分类（架构图 / 流程图 / 柱状图 / 折线图 / 可视化）
3. 逐图生成 SVG（遵循学术配色 + PPTX 兼容约束）
4. SVG 质量自检（XML 合法性 / 禁用特性 / 可读性）
5. 运行 `figure_to_pptx.py` 将所有 SVG 汇总到单个 PPTX
6. 更新论文中的占位符为 LaTeX figure 环境

**输出**：
- `figures/svg_output/*.svg` — 各图 SVG 源文件
- `figures/output/paper_figures.pptx` — 所有图汇总的 PPTX（可在 PPT 中编辑）
- `figures/figure_list.json` — 图表清单

### Phase 6: Review Agent（模拟审稿）

**何时执行**：完整论文草稿就绪。

**读取详细指令**：[review-agent.md](references/review-agent.md)

**摘要**：
1. 模拟 3 个 Reviewer（方法论/实验/写作 各一个）
2. 锚定审稿（用已知分数论文做参照）
3. 生成 Meta-Review + Accept/Reject 预测
4. 输出优先修改清单

### Phase 7: Rebuttal Agent（反驳）

**何时执行**：收到 review（模拟的或真实的）。

**读取详细指令**：[rebuttal-agent.md](references/rebuttal-agent.md)

**摘要**：
1. 逐条分析 Weakness/Question
2. 分类（可回复/需补实验/需改写/无法解决）
3. 生成 Rebuttal Letter
4. 生成论文修改建议
5. 如需大改，回到 Writing Agent

## 论文绘图：SVG → PPTX

论文中的图表通过 Figures Agent 生成。核心流程：**先写 SVG → 再汇总到 PPTX**。

**绘图命令**：
```bash
python3 [skill目录]/scripts/figure_to_pptx.py "[项目路径]/figures"
```

**输出**：`figures/output/paper_figures.pptx`（一个 PPTX 包含所有论文图表，每图一页 slide）

**PPTX 用途**：
- 用户在 PowerPoint 中精调图表细节（对齐、颜色、标签位置）
- 导出为 PDF/PNG 用于 LaTeX 插图
- 直接复制到答辩 PPT

**SVG 技术约束**（确保 PPTX 兼容）：
- 禁用 `<style>`/`class`/`mask`/`<foreignObject>`/`<animate>`
- 颜色用 HEX + `fill-opacity`，不用 `rgba()`
- 字体用系统预装字体（Arial / Times New Roman / Microsoft YaHei）
- 详见 [figures-agent.md](references/figures-agent.md)

## 最终输出：Markdown -> LaTeX -> PDF

论文写完后，转换为目标会议的 LaTeX 格式并编译 PDF。

**会议模板规范**：[venue-templates.md](references/venue-templates.md)

**转换流程**：
1. 运行 `scripts/md_to_latex.py` 将 Markdown 转为 LaTeX
2. 运行 `scripts/build_paper.sh` 编译 PDF

```bash
python [skill目录]/scripts/md_to_latex.py \
  "output/paper.md" \
  "output/paper.tex" \
  --venue <venue_name> \
  --bib "references.bib"

bash [skill目录]/scripts/build_paper.sh "output/paper.tex"
```

支持的 venue 列表：`neurips_2026`、`icml_2026`、`iclr_2026`、`cvpr_2026`、`acl_2026`、`iccse_2026`

## 引用验证

在 Literature Agent 和 Writing Agent 阶段，都需要验证引用的真实性。

**详细协议**：[citation-verification.md](references/citation-verification.md)

**快速验证**：
```bash
python [skill目录]/scripts/verify_citations.py "references.bib"
```

## 联网工具使用

整个管线需要大量联网搜索。使用以下工具：

- **WebSearch**：搜索论文趋势、最新工作、特定主题
- **WebFetch**：获取具体页面内容（arXiv 论文页、GitHub README 等）
- **arXiv API**：`curl -s "https://export.arxiv.org/api/query?search_query=all:关键词1+AND+all:关键词2&max_results=20&sortBy=submittedDate&sortOrder=descending"`
- **Semantic Scholar API**：`curl -s "https://api.semanticscholar.org/graph/v1/paper/search?query=关键词&limit=20&fields=title,authors,year,citationCount,venue,externalIds,abstract"`

搜索策略：先用 WebSearch 发现方向，再用 arXiv API / Semantic Scholar API 做精确检索。多次搜索、多个关键词组合，不要只搜一次。

## 使用子 Agent 加速

对于耗时较长的阶段（尤其是 Literature Agent），建议使用子 Agent 并行：
- 子 Agent 1：arXiv 检索 + 最新 preprint 扫描
- 子 Agent 2：Semantic Scholar 检索 + citation chasing
- 子 Agent 3：Google Scholar 补充搜索

每个子 Agent 的 prompt 中包含联网工具使用指引（见上文）。

## 质量红线

以下问题出现任何一条，论文不可提交：
- 引用了不存在的论文（幻觉引用）
- 实验数据/结果是编造的
- 方法描述与实际实现不一致
- Related Work 遗漏了该领域的关键工作
- 论文存在明显的逻辑矛盾

发现上述问题时，必须停下来修复，不可跳过。

---
> Source: [SGloria/research-paper-pipeline](https://github.com/SGloria/research-paper-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
