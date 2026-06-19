## deeppaperreader4chinese

> >-


# Paper Reader

Use this skill to read an academic PDF in either full mode or fast-report mode.
The goal is not to translate every sentence. Reconstruct the authors' reasoning,
identify the core problem, explain why prior approaches are insufficient, and
extract the key insights.

- **Full mode**: produce two Chinese Markdown files: one concise translated
  logic reconstruction and one research-level analysis.
- **Fast-report mode**: produce one Chinese Markdown file focused on task
  definition, method details, experiments, first-principles reconstruction, and
  literature positioning.

## Inputs and Outputs

- If the user did not provide a PDF path, ask for one concise question.
- Resolve the PDF path before working. Put all outputs in the same directory as
  the PDF.
- Use fast-report mode when the user asks for "fast", "quick", "快速",
  "速读", "快读", "快速报告", "fast report", or requests a single compact
  paper report. Otherwise use full mode.
- In full mode, create exactly these two files:
  - `<original_filename>_translation.md`
  - `<original_filename>_analysis.md`
- In fast-report mode, create exactly this one file:
  - `<original_filename>_fastreport.md`
- All output files must be written entirely in Chinese. Section titles, table
  headers, explanations, and commentary must be Chinese. Mathematical formulas,
  variable names, method names, benchmark names, model names, code identifiers,
  and paper-internal identifiers may remain in their original form.

## Evidence Rules

1. Do not hallucinate. Every factual claim must be traceable to the paper.
2. Separate paper facts from your interpretations.
3. When evidence is insufficient, write: `论文未提供足够证据。`
4. Prefer citing the paper location when possible: section, page, figure, table,
   appendix, equation, or algorithm.
5. Preserve uncertainty. Do not convert author speculation into established
   fact.
6. Do not use external sources unless the user asks, the paper explicitly relies
   on an external artifact, or the task cannot be completed without checking a
   referenced repository/document. If external sources are used, clearly label
   them separately from paper evidence.

## Markdown and LaTeX Rules

1. Write math in Markdown-compatible LaTeX. Use `$...$` for inline math and
   `$$...$$` for display math. Do not use `\(...\)` or `\[...\]` delimiters.
2. Preserve LaTeX commands with literal backslashes in the final Markdown file.
   For example, write `$4.22 \times 10^{20}$`, not `\(4.22\times10^{20}\)`,
   `4.22 times 10^20`, or `4.22 x 10^20`.
3. Reconstruct malformed extracted formulas into valid LaTeX when the PDF text
   extraction drops symbols, spacing, superscripts, subscripts, or Greek letters.
   If the formula cannot be recovered from the paper, write
   `论文未提供足够证据。`
4. Keep formulas in math mode even inside Chinese sentences and tables. In
   Markdown tables, avoid raw `|` inside formulas; use `\mid`, `\vert`,
   `\lVert...\rVert`, or move the formula outside the table.
5. Do not wrap formulas in code backticks unless explicitly discussing source
   code. Do not translate LaTeX operators or variables into Chinese inside math.

## Reading Workflow

1. Extract readable text from the PDF. Prefer structured extraction when
   available, such as `pdftotext -layout`; if extraction is poor, try another
   available local tool or inspect relevant pages visually.
2. Build an internal evidence map before writing:
   - title, authors, venue/date if present
   - abstract and stated contributions
   - problem setup and assumptions
   - method components and information flow
   - datasets, baselines, metrics, tables, figures, ablations, appendices
   - stated limitations and unstated failure modes
3. Read figures, tables, equations, algorithms, and appendices, but extract them
   at different granularity:
   - Core methods, algorithms, definitions, equations, architectural diagrams,
     main results, key ablations, and central claims must be preserved.
   - Repetitive background, rhetorical setup, duplicated claims, routine dataset
     descriptions, and boilerplate experimental details may be merged or omitted.
   - Appendices and supplementary experiments should usually be summarized by
     their purpose, evidence value, and conclusion. Only expand appendix content
     when it contains essential proofs, implementation details needed to
     understand the method, or results that materially change the interpretation.
   - Reproduce a table only when exact numbers are central to the paper's
     argument. Otherwise summarize the trend and cite the table/figure location.
4. For long papers, process in chunks but keep a global outline so later
   sections do not contradict earlier evidence.
5. In full mode, generate the translation first, then the analysis. Do not stop
   after only one file. In fast-report mode, generate only the fast report.

## Fast Report File

Create `<original_filename>_fastreport.md`.

This file is a compact Chinese research report for quickly understanding a paper
without producing the full translation and analysis pair. It must still be
grounded in paper evidence and preserve enough technical detail for a researcher
to explain, compare, and potentially reproduce the method.

Use this structure:

### 1. 论文定位与一句话贡献

- 论文解决的核心问题是什么？
- 一句话说明最关键贡献。
- 说明它属于哪个研究方向或技术路线。

### 2. 任务定义

- **输入**：模型、算法或系统接收什么？
- **输出**：它要预测、生成、选择或优化什么？
- **约束**：训练数据、监督信号、计算、交互、部署或任务假设是什么？
- **优化目标**：列出主要 loss、reward、likelihood、matching objective 或评估目标；公式存在时保留公式。

### 3. 实际方法细节

不要只写模块名称。必须解释信息流、算法步骤、训练方式和计算细节。

- **整体架构**：组件如何连接？每个组件输入/输出是什么？
- **核心算法**：按步骤写清算法流程；论文有算法框、伪代码或公式时保留关键步骤。
- **训练方式**：数据来源、采样方式、阶段训练、预训练/微调、损失项、优化器、batch、序列长度、模型规模等；论文未给出的项目写 `论文未提供足够证据。`
- **推理方式**：测试或部署时如何从输入得到输出？是否有采样、搜索、规划、解码、后处理？
- **关键实现细节**：tokenizer、action space、memory、masking、attention、scheduler、augmentation、normalization 等影响复现或理解的细节。

### 4. 实验分析

- **实验设置**：数据集、任务、指标、baseline、评估协议。
- **主结果**：最关键表格/图证明了什么？哪些比较真正支撑主张？
- **消融实验**：哪些组件被验证为必要？结果说明了什么机制？
- **失败与边界**：论文报告的失败、limitations，以及从结果中能看出的未验证声明。

### 5. 第一性原理重构

用连续问题链重构方法如何自然出现：

> Q1：已有方法的默认假设是什么？
> ↓
> Q2：该假设在本文任务中为什么失败？
> ↓
> Q3：真正缺失的信息、结构或监督信号是什么？
> ↓
> Q4：作者如何把这个缺失项变成可学习对象？
> ↓
> Q5：为什么最终结构能解决这个问题？

### 6. 文献定位

将论文放入相关研究谱系，至少覆盖经典方法、强基线和近期相近工作。每个对比说明：

- 相似点
- 差异点
- 本文优势
- 本文劣势或未覆盖场景

### 7. 速读结论

- **最值得记住的创新点**：3-5 条。
- **复现或迁移时最关键的技术细节**：3-5 条。
- **后续研究切入点**：2-4 条，必须从论文证据或缺口推出。

## Translation File

Create `<original_filename>_translation.md`.

This file is a Chinese translated reading note, not a sentence-by-sentence or
paragraph-by-paragraph translation. It should let a reader who has not seen the
original paper understand the author's argument and the complete core technical
content while avoiding unnecessary bulk.

- Preserve the original document hierarchy at the section/subsection level, but
  merge paragraphs when they play the same rhetorical role.
- For each section, reconstruct the author's local logic: what question they are
  answering, what evidence or design they introduce, and how the section advances
  the paper's main argument.
- Keep all core technical details: problem definition, assumptions, algorithmic
  steps, formulas, architecture, training/inference flow, datasets, main metrics,
  primary results, key ablations, and limitations.
- Omit or compress nonessential content: generic motivation, repeated related
  work descriptions, verbose dataset administration, implementation boilerplate,
  and repeated conclusion phrases.
- Handle appendices with a separate short section named `附录与补充材料概览`.
  Describe the spirit and evidential role of each appendix. Supplemental
  experiments can be one sentence or one bullet each unless their result changes
  the interpretation of the main method.
- Do not add independent critique or interpretation in the translation file.
- Do not omit core algorithms, core ideas, central evidence, or result trends.
- Keep formulas unchanged when translating them would reduce clarity.
- When an English technical term is important, include it once in parentheses
  after the Chinese term.

## Analysis File

Create `<original_filename>_analysis.md`.

The analysis must use the structure below. Keep headings in Chinese.

### 1. 任务定义

#### 问题形式化

- 输入是什么？
- 输出是什么？
- 约束是什么？
- 优化目标是什么？
- 尽可能用形式化方式表达任务。

#### 任务价值

- 解决该任务能解锁什么实际能力？
- 它属于哪个更大的研究方向？

### 2. 核心挑战

解释为什么问题困难。每个挑战使用以下格式：

**挑战 X**

- **已有假设**：先前方法默认了什么？
- **失效原因**：这个假设为什么会失败？
- **后果**：会出现什么失败模式？
- **论文证据**：论文中哪些实验、图表、段落或现象支持这一点？

### 3. 洞察与创新

这是最重要的部分。重构作者的思考过程。

#### 3.1 灵感来源

说明论文的灵感可能来自哪些观察、现象、先前工作或直觉。每一项都说明来源、观察和暴露出的限制。

#### 3.2 核心洞察

每个洞察回答：

- 洞察是什么？
- 它解决了哪个挑战？
- 从第一性原理看为什么合理？
- 为什么它可能有效？
- 论文证据是什么？

#### 3.3 创新点拆解

每个创新点使用以下格式：

**创新点 X**

- **问题**：它解决的具体问题是什么？
- **洞察**：它由哪个洞察驱动？
- **设计**：作者设计了什么？
- **实现**：如何实现？
- **机制**：为什么这个设计应该有效？
- **预期效果**：理论上应带来什么改善？
- **观察效果**：论文实际观察到了什么改善？

### 4. 方法拆解

按组件分析。每个组件使用以下格式：

**组件名称**

- **目的**：为什么需要它？
- **输入/输出**：输入和输出分别是什么？
- **内部机制**：组件内部如何工作？
- **与其他组件的交互**：信息如何传递？
- **失败模式**：在什么情况下可能失败？
- **消融证据**：论文是否验证了它的重要性？

然后补充：

#### 信息流

逐步描述信息如何从输入流向输出。

#### 训练与推理流程

- 数据
- 目标函数
- 优化方式
- 损失项
- 课程学习或阶段训练（如有）
- 推理过程

### 5. 实验分析

不要只复述作者结论，要解释实验真正证明了什么。

#### 主结果

主表格展示了什么？哪些比较最关键？

#### 消融实验

哪些组件最重要？为什么？

#### 规模变化

性能如何随数据量、模型规模、计算量、环境复杂度或任务难度变化？

#### 隐含发现

论文没有明确强调，但可以从结果中提取出什么结论？

### 6. 潜在弱点

独立批判分析，不要只重复作者的 limitations。

- **场景限制**：在哪些环境或假设下可能失败？
- **数据限制**：分布偏移、噪声、稀疏性、长尾事件、缺失模态、非平稳性、对抗样本。
- **结构限制**：瓶颈、硬编码假设、可扩展性问题。
- **评估限制**：缺失实验、未验证声明、不充分指标。

### 7. 未来研究方向与方法优化

包含两个层次：A. 基于论文缺口的未来方向；B. 对模型结构或方法的深层优化。

#### A. 未来研究方向

每个方向使用以下格式：

**方向 X**

- **对应限制**：它解决哪个缺口？
- **核心想法**：关键洞察是什么？
- **潜在方法**：如何实现？
- **预期收益**：可能改善什么？
- **论文潜力**：Workshop / CCF-C / CCF-B / CCF-A / 顶会潜力。说明理由。

#### B. 深层结构与方法优化

至少从以下四个维度中的三个提出具体优化想法：

- **B1. 从实验结果反推优化**：性能瓶颈、消融信号、天花板效应、规模趋势。
- **B2. 从能力短板反推优化**：归纳偏置不匹配、信息瓶颈、目标函数错配、泛化边界。
- **B3. 从结构/机制约束反推优化**：模块交互刚性、表征粒度不匹配、时序或结构建模深度不足、训练-推理不一致。
- **B4. 跨领域迁移启发**：NLP、CV、RL、图学习、优化理论等领域中可迁移的类似解决方案。

每个优化想法使用以下格式：

**优化想法 X**

- **分析维度**：属于 B1/B2/B3/B4 中哪一类？
- **观察/推理依据**：论文中的哪些结果、结构特征或理论分析支持这个想法？
- **具体改进方案**：如何修改模型结构或方法？要具体到可实现的设计。
- **预期改善**：会在哪些指标或场景上提升？为什么？
- **潜在成本**：计算量、训练复杂度、稳定性或工程成本。
- **可行性**：低 / 中 / 高，并解释判断。

### 8. 第一性原理重构

假设这篇论文不存在，只从问题本身出发，重构一个强研究者如何自然想到该方法。使用连续问题链：

> Q1：已有方法做了什么？它们的根本假设是什么？
> ↓
> Q2：这个假设什么时候会失败？
> ↓
> Q3：真正缺失的信息是什么？
> ↓
> Q4：能否显式建模这些信息？
> ↓
> Q5：什么结构自然支持这种建模？
> ↓
> 持续推进，直到推导出论文的最终方案。

### 9. 文献定位

将论文与经典方法、强基线和近期 SOTA 对比。每个对比说明：

- 相似点
- 差异点
- 优势
- 劣势

### 10. 研究者要点

- **30 秒总结**：给第一次听说这篇论文的人。
- **5 分钟总结**：给刚进入该领域的研究生。
- **研究思维总结**：研究者应该学到的思维方式，不只是方法本身。

## Final Self-Check

Before reporting completion, verify:

- In full mode, both required files exist in the PDF directory.
- In fast-report mode, the single required fast report exists in the PDF
  directory and no translation/analysis pair is required unless the user also
  requested full mode.
- All output files are entirely Chinese except formulas, identifiers, method names,
  dataset names, benchmark names, and code-like tokens.
- In full mode, the translation file does not contain independent critique.
- In full mode, the translation file reconstructs section-level and
  paragraph-group-level logic instead of translating sentence by sentence.
- In full mode, the analysis file separates paper evidence from interpretation.
- In fast-report mode, the report contains task definition, actual method
  details including algorithms/training/computation details, experiment
  analysis, first-principles reconstruction, and literature positioning.
- Core figures, tables, equations, and algorithms were preserved or summarized
  with cited locations; appendices were reviewed and summarized at appropriate
  granularity.
- Claims with insufficient evidence explicitly say `论文未提供足够证据。`
- Formula delimiters are Markdown-compatible: no `\(...\)` or `\[...\]`
  delimiters remain; inline formulas use `$...$`, display formulas use
  `$$...$$`, and LaTeX commands such as `\times`, `\sum`, `\mathbb`, `\le`,
  subscripts, and superscripts are preserved correctly.

After the required file or files are written, report the file paths and a
concise Chinese summary of the most important insights found.

---
> Source: [V1staz/DeepPaperReader4Chinese](https://github.com/V1staz/DeepPaperReader4Chinese) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
