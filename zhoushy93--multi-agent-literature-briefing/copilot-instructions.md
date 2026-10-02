## multi-agent-literature-briefing

> 本文件是本仓库**最高优先级的项目约定**。所有 AI 编码代理（Codex/Claude/其他）与人类协作者都必须遵守。

# AGENTS.md

本文件是本仓库**最高优先级的项目约定**。所有 AI 编码代理（Codex/Claude/其他）与人类协作者都必须遵守。

- 若与其他文档、注释、口头约定冲突，**以本文件为准**。
- 若任务需要改变本文件中标注为「架构级决策」的内容，**必须在同一次改动中先更新本文件**，再改代码。
- 本文件描述「项目要建成什么 + 怎么建」，不是当前进度快照。进度与待办写在 `docs/SPEC.md` 与 issue 中。

---

## 1. 项目目标

构建一个**基于 DeepSeek V4 的 literature briefing 多 Agent 系统**：

> 输入一个研究 topic，检索 3–5 篇相关论文，逐篇分析，最终生成一份可引用的 PDF 报告。

主入口（目标形态，实现时不得随意改名）：

```bash
briefing run --topic "diffusion models for weather forecasting" --out outputs/2026-09-17-weather
```

### 1.1 产品级完成标准（Definition of Done）

一次运行结束后，`<out>/` 目录必须同时存在：

| 文件 | 内容 |
| --- | --- |
| `report.pdf` | 最终交付物，见 §8 报告规范 |
| `briefing.md` | 报告的可编辑版本。它与 PDF **同源于同一个通过校验的 `Briefing` 对象**（而非 PDF 的输入），因此"两者内容一致"由构造保证 |
| `briefing.json` | 结构化简报（章节、结论、对比表） |
| `papers.json` | 入选论文的完整元数据 + 逐篇分析结果 |
| `manifest.json` | 运行审计：模型 id、prompt 版本、参数、token 用量、耗时、检索来源、缓存命中情况、renderer |

硬性验收条件：

1. 论文数量 **3 ≤ n ≤ 5**；不足 3 篇时**明确失败并说明原因**，绝不允许凑数或编造。
2. 每篇论文都具备可解析的：标题、作者列表、年份、来源（venue / 预印本平台）、URL，以及 DOI 或 arXiv ID（至少其一，确实不存在时显式标注 `null` 并说明）。
3. `report.pdf` 中**每一条引用都能追溯到 `papers.json`**；零虚构引用（详见 §3 Verifier）。
4. PDF 中英文混排可正常阅读，字体内嵌，文本可选中、可搜索，页码与目录正确。
5. 单次运行有可审计的调用次数与 token 上限（§11）。
6. `manifest.json` 能回答「这份报告是用哪个模型、哪版 prompt、什么参数生成的」。

### 1.2 明确非目标（v0 不做）

- 不做论文全文数据库、不做 PDF 全文抓取（只处理开放获取可合法获取的文本）。
- 不做引用网络/引文图分析、不做系统性综述（PRISMA）级别的方法学。
- 不做网页 UI、不做多用户/账号体系；v0 只有 CLI。
- 不为了「更全」而接入需要付费或违反 tos 的数据源。

---

## 2. 架构级决策（改动需谨慎）

以下决策属于架构级，修改必须在同一次变更中更新本文件并记录到 `docs/decisions/`。

1. **编排形态**：确定性流水线（pipeline）+ 分析阶段 fan-out 并行。不引入通用 agent 框架（LangGraph/AutoGen 等）作为 v0 依赖；直接用一个薄编排器（`orchestrator.py`）实现，便于测试与审计。
2. **模型供应商**：DeepSeek，OpenAI 兼容接口。业务代码只依赖内部 `LLMClient` 协议，不直接依赖任何 SDK。
3. **结构化契约**：Agent 之间只传递 `src/briefing/schemas.py` 中的 pydantic 模型，**禁止裸 `dict` 跨 Agent 传递**。
4. **可复现**：所有外部 HTTP 响应可缓存；缓存命中必须体现在 `manifest.json` 中。
5. **引用可验证**：引文只能来自检索阶段真实取回的元数据，LLM 不得生成参考文献条目。

---

## 3. 多 Agent 设计

每个 Agent 是一个可独立测试的纯函数/类：`async def run(input_model) -> output_model`。

| Agent | 职责 | 输入 | 输出 | LLM? |
| --- | --- | --- | --- | --- |
| `Planner` | 把 topic 拆成查询词、同义词、时间窗、纳入/排除标准 | `TopicRequest` | `SearchPlan` | 是 |
| `Retriever` | 调外部数据源检索候选（目标 20–40 篇） | `SearchPlan` | `CandidateList` | 否 |
| `Screener` | 去重、过滤、按相关性排序，选出 3–5 篇 | `CandidateList` | `SelectedPapers` | 是 |
| `Analyzer` | **每篇并行**：抽取问题/方法/数据/结论/局限/可复用点 | `Paper` | `PaperAnalysis` | 是 |
| `Synthesizer` | 横向对比、共性、分歧、研究空白、开放问题 | `SelectedPapers` + `[PaperAnalysis]` | `Briefing` | 是 |
| `Verifier` | 校验每条引用存在于 `papers.json`，标注无支撑断言 | `Briefing` + `SelectedPapers` | `VerificationReport` | 是（可选规则） |
| `Renderer` | `Briefing` → HTML → PDF | `Briefing` | `report.pdf` | 否 |

编排规则：

- `Analyzer` 阶段并行执行，并发数受 `MAX_CONCURRENCY`（默认 4）限制。
- 单个论文分析失败：重试 ≤2 次；仍失败则**整次运行失败**，不得静默丢弃该论文（否则会出现「3–5 篇」中实际只有 2 篇被分析的情况）。
- `Verifier` 发现无支撑引用：先让 `Synthesizer` 修订一次；仍不通过则失败并输出 `verification_failed`，保留中间产物。
- 每个 Agent 的 prompt 独立存放于 `src/briefing/prompts/<agent>_v<N>.md`，**prompt 文件名带版本号**，版本写入 `manifest.json`。

---

## 4. 技术栈与依赖约束

- Python **3.11+**，依赖与虚拟环境统一用 `uv` 管理（`pyproject.toml` + `uv.lock`，lock 文件必须提交）。
- 运行期依赖限定在：`httpx`、`pydantic`(v2)、`pydantic-settings`、`tenacity`、`jinja2`、`weasyprint`、`pypdf`。新增运行期依赖需在 `docs/decisions/` 里说明理由。
- 开发依赖：`pytest`、`pytest-asyncio`、`respx`、`ruff`、`mypy`。
- 全链路 `async` I/O；禁止在异步代码里用阻塞式 `requests`/`time.sleep`。
- 除 PDF 渲染外不得使用线程池/多进程。

---

## 5. 目录结构（约定，不得随意散落文件）

```
.
├── AGENTS.md                  # 本文件
├── README.md                  # 面向使用者的快速上手
├── pyproject.toml
├── uv.lock
├── .env.example               # 与 .env 保持同步，只含占位值
├── .githooks/pre-commit       # 拒绝提交密钥（每个 clone 启用一次）
├── TASK.md                    # 分步执行计划
├── docs/
│   ├── SPEC.md                # 当前范围、进度、待办
│   ├── architecture.md        # 多 Agent 架构、契约、状态机、不变量
│   └── decisions/NNN-*.md     # 关键决策记录
├── src/briefing/
│   ├── cli.py                 # 唯一 CLI 入口（stdlib argparse）
│   ├── orchestrator.py        # 流水线编排
│   ├── config.py              # pydantic-settings 配置
│   ├── schemas.py             # 所有 Agent 契约模型
│   ├── manifest.py            # 运行审计契约 + 阶段检查点/指纹
│   ├── evidence.py            # 引文溯源规则（无 LLM 依赖，供分析与验证共用）
│   ├── rendering.py           # 论文目录渲染（合成与审计共用同一份材料）
│   ├── agents/                # planner/retriever/normalizer/screener/analyzer/synthesizer
│   ├── llm/                   # deepseek_client.py / stub_client.py / budget.py + 协议
│   ├── sources/               # arxiv.py / replay.py / cache.py + Source 协议
│   ├── verify/                # deterministic.py(L1) / semantic.py(L2) / report.py
│   ├── report/                # references.py / markdown.py / briefing_md.py / render_pdf.py / templates
│   └── prompts/               # 带版本号的 prompt
├── tests/
│   ├── unit/                  # 全离线
│   ├── integration/           # 离线端到端 / PDF 渲染 / live 冒烟
│   └── fixtures/              # 录制的 HTTP 响应与模型回复
├── examples/                  # 人工确认过的样例产物
├── data/cache/                # 原始响应缓存（gitignore）
└── outputs/                   # 运行产物（gitignore；样例放 examples/）
```

根目录**不新增**临时脚本、notebook、草稿文件；一次性实验放在 `scratch/`（gitignore）或删除。

---

## 6. DeepSeek 接入规则

- 只通过 `src/briefing/llm/deepseek_client.py` 调用；接口为 OpenAI 兼容的 `POST {base_url}/chat/completions`。
- 模型 id **必须来自配置**：`BRIEFING_DEEPSEEK_MODEL`（默认 `deepseek-v4-pro`），base url 为 `BRIEFING_DEEPSEEK_BASE_URL`（默认 `https://api.deepseek.com`）。凭证用提供商原生的 `DEEPSEEK_API_KEY`（不加前缀）。**禁止把模型名硬编码进业务代码或 prompt 文本**。
  - 「DeepSeek V4」在实现时以官方文档公布的确切模型 id 与上下文长度为准；不要凭记忆填写，先用一次真实调用验证可用性并记录到 `docs/decisions/`。
- 需要推理链的任务（Screener/Synthesizer）可用 reasoner 类模型，并通过 `BRIEFING_DEEPSEEK_MODEL_REASONING` 单独配置；是否拆分由配置控制，不改代码分支。
- 结构化输出：优先 JSON 输出 + `pydantic` 校验。校验失败时携带**校验错误原文**重试，最多 2 次；仍失败则该 Agent 失败。
- 采样参数：抽取/校验 `temperature=0`；逐篇分析 `0.2`；综合 `0.3`。temperature 与 `top_p` 写入 manifest。
- 成本纪律：不把论文全文塞进 prompt。只使用摘要 + 元数据 + 合法获取的开放全文片段，并在 `PaperAnalysis.evidence` 中标注来源段落。
- 重试：对 429/5xx/超时使用指数退避 + 抖动，最多 5 次；4xx（除 429）直接失败并给出可读错误。
- 记录每次调用的 usage（prompt/completion tokens）并汇总进 `manifest.json`。

---

## 7. 检索与选文规则

- 数据源：**arXiv Atom API**（主，无需 key）+ **OpenAlex**（备，无需 key）。二者默认启用；Crossref、Semantic Scholar 仍是可选项（需要 key 时通过环境变量注入，缺失则跳过而非报错）。
- **任何单一数据源都不得拖垮整次运行**：某查询在自己的数据源上失败时，检索器必须把该查询以纯文本形式重试到其它已配置数据源，并记账（`dropped["source_fallback"]`，查询列表标注 `(fallback)`）。这是 arXiv 按 IP 限流（HTTP 406）时的正常降级路径，不是异常。
- 候选量：先取 20–40 篇，再排序筛选出 3–5 篇；摘要少于 300 字符的候选降权。
- 去重优先级：DOI → arXiv ID → 归一化标题（小写、去标点、压缩空白）。
- 检索参数（查询式、时间窗、排序依据）必须完整写入 `manifest.json`，使结果可复现。
- 不足 3 篇时：抛出明确错误，附「实际找到的候选数 + 所用查询 + 建议的放宽策略」，不允许降低标准凑数。
- 缓存：原始响应存 `data/cache/<source>/<sha256(query+params)>.json`，默认 30 天内复用；`--no-cache` 可禁用。
- 速率与合规：遵守各 API 的 rate limit 与 tos，请求带 `User-Agent: briefing/0.1 (+repo url)`。
- **arXiv 的 406 是限流**（不是请求格式错误）：它针对来源 IP 按滚动窗口计算。因此同一 source 的请求必须**串行且至少间隔 3 秒**（`BRIEFING_SOURCE_MIN_INTERVAL_S`），收到 406/429 时进入**共享冷却**（20→120 秒，所有并发查询一起等待，不得各自重试把封禁续命），并有界重试（`BRIEFING_SOURCE_MAX_ATTEMPTS`，默认 4，`Retry-After` 优先）。详见 `docs/decisions/004-arxiv-rate-limits.md`。
- **限流要熔断，不要逐查询重试**：整轮共享一个很小的限流重试预算（`RATE_LIMIT_ATTEMPTS`，默认 2 次请求）。用尽即把该 source 在本轮标记为不可用（`rate_limited`），后续查询**一次请求都不再发**，直接交给备用源。逐查询重试会白等十几分钟并把封禁续命——这是实测过的真实缺陷，别再改回去。
- **重复运行靠缓存，不要习惯性加 `--no-cache`**：默认缓存下重跑不再请求 arXiv；`--no-cache` 每次都会真实抓取全部查询，是触发限流的主要途径。

---

## 8. 报告规范（PDF）

渲染链：`Briefing` → Jinja2 HTML →（主）WeasyPrint ；（降级）headless Chromium。使用降级渲染器时必须在 `manifest.json` 标注 `renderer`。

章节顺序固定：

1. 标题页：topic、生成日期、模型 id、论文数量
2. 一句话结论（Executive summary，≤120 字；模型超出时由 S6 按句子边界确定性收敛到上限，L1 仍作为兜底校验）
3. **主题速览（面向快速阅读）**：一张 topic overview 图（主题 → 各论文的 hub-and-spoke）＋一张速览表（Ref / 论文 / 方法族 / 主要发现 / 与主题的关系）
4. 主题背景与本次检索范围
5. 检索方法与纳入/排除标准（可复现）
6. 逐篇论文卡片：问题 / 方法 / 数据与实验 / 主要结论 / 局限 / **与主题的关系（Relevance）** / 可复用点 / 原文链接。卡片用两列表格排版，标签列固定，便于逐篇对照扫读
7. 横向对比表（维度固定：任务、方法族、数据、评估指标、主要结论）
8. 研究空白与开放问题
9. 后续阅读建议
10. 参考文献（编号与正文引用一致）
11. 附录：生成参数摘要（模型、prompt 版本、时间、token 用量）

速览章节的两个元素都由**确定性代码**生成，不额外调用模型：速览表取自各篇分析与对比表，总览图是代码拼出的内联 SVG（`src/briefing/report/figures.py`，不引入任何图表依赖）。SVG 文本必须转义，且因为 WeasyPrint 用自己的 SVG 引擎、不套用文档 CSS，形状样式要写成 SVG 表现属性而不是 class。

排版硬要求：

- CJK 字体必须内嵌（Noto Sans CJK / Source Han Sans，或平台上等价的系统 CJK 字体；本机为 `STHeiti`，见 `docs/decisions/001-runtime-and-renderer.md`），HTML 中显式指定 font-family 回退链。
- 文本可选中、可搜索；**禁止把整页渲染成图片**。
- 正文引用使用稳定编号 `[1]`、`[2]`…，与参考文献一一对应；同一条引用不得指向两篇不同论文。
- 引用格式：`作者. 标题. 来源, 年份. DOI/URL`。
- 报告语言默认与 topic 语言一致，`--lang zh|en` 可覆盖。

---

## 9. 编码与工程约定

- 全量类型注解；`mypy` 对 `src/` 至少达到 `--strict` 可运行（新增代码不得引入新错误）。
- 代码标识符、prompt 模板、注释用英文；面向用户的 CLI 输出与文档可中文。
- 日志：结构化 JSON（标准 `logging`，`--verbose` 控制级别）。**禁止 `print` 调试**。
- 错误：统一异常基类 `BriefingError`；CLI 捕获后输出可读信息并以非零码退出（参数错误 2、运行失败 1）。
- 配置：全部经 `config.py`（`pydantic-settings`），env 前缀 `BRIEFING_`；新增配置项必须同步 `.env.example` 与 README。
- 禁止提交任何密钥、`.env`、缓存与运行产物；`.gitignore` 覆盖 `data/cache/`、`outputs/`、`.env` 及 `.env.*`（仅 `.env.example` 例外）、`scratch/`、`*.pem`、`secrets/`。
- 密钥防护有两层，改动敏感配置时都要保持有效：提交前钩子 `.githooks/pre-commit`（每个 clone 执行一次 `git config core.hooksPath .githooks` 启用），以及作为 Gate 一部分运行的 `tests/unit/test_no_secrets.py`——它扫描的就是"会被上传的那些文件"，任何密钥形状的内容都会让它失败。
- 依赖版本固定；升级依赖需单独提交并跑通全部测试。

---

## 10. 测试与验收

默认全套测试**离线**：HTTP 用 `respx` 或 `tests/fixtures/` 录制样本 mock，LLM 调用用 stub client 替换。

必须覆盖：

- 去重逻辑（DOI / arXiv ID / 标题归一化三种路径）
- 排序与选文（含「不足 3 篇 → 报错」分支）
- 每个 schema 的校验与非法输入拒绝
- 重试与退避（429、5xx、超时）
- 缓存命中与 `--no-cache`
- PDF 生成：文件存在、页数 > 0、能被 `pypdf` 提取到正文文本（用 fixtures 驱动，不依赖网络）
- 引用一致性：正文引用编号 ⊆ 参考文献 ⊆ `papers.json`

真实联网 + 真实 LLM 的端到端测试仅在 `RUN_LIVE_TESTS=1` 且存在 `DEEPSEEK_API_KEY` 时运行，默认跳过。

提交前必跑（必须全绿）：

```bash
uv run ruff check . && uv run ruff format --check . && uv run mypy src && uv run pytest -q
```

---

## 11. 成本与安全护栏

- 每次运行上限：`MAX_LLM_CALLS`（默认 40）、`MAX_TOKENS_BUDGET`（默认 400k）。超限立即中止，保留已完成产物并在 manifest 标记 `budget_exceeded`。
- 论文上限 5 篇、下限 3 篇；选文阶段不得额外调用 LLM 超过 3 次。
- 不下载、不传播非开放获取的全文；不绕过付费墙；不把论文原文存入仓库。
- 日志与产物中不得出现 API key、Cookie、代理凭证。
- 默认不联网发送除 topic 与论文摘要之外的任何仓库内容。

---

## 12. 协作与提交

- Commit message 用 Conventional Commits（`feat:`/`fix:`/`docs:`/`refactor:`/`test:`/`chore:`），一次提交只做一件事。
- 行为变更必须同 PR 更新：测试 + `docs/SPEC.md` +（若涉及契约）`schemas.py` 与 `docs/architecture.md`。
- 架构级决策记录到 `docs/decisions/NNN-标题.md`：背景 / 选项 / 决定 / 后果。
- 不提交 `outputs/` 与 `data/cache/`；若需示例，把一份人工确认过的报告放到 `examples/<date>-<topic-slug>/` 并注明是样例。

---

## 13. 给 AI 编码代理的硬规则

1. 动手前先读本文件、`docs/SPEC.md`、`docs/architecture.md`。
2. 任务闭环 = 定位改动点 → 实现 → 跑 §10 的验证命令 → 用证据（命令输出、diff）汇报。**没跑验证不算完成**。
3. 不擅自扩大范围：不新增付费数据源、不换模型供应商、不引入新框架、不重构无关模块。
4. 需要真实联网/付费调用时，先说明预计调用次数与成本，再执行。
5. 不编造 API 行为、模型名、论文元数据。不确定就查官方文档，或写成配置项 + 待确认项。
6. 不为了让测试通过而削弱断言、放宽 schema 或跳过失败分支。
7. 发现本文件与实际需求冲突时：**先停手说明冲突**，给出选项，再按确认结果更新本文件与代码。

---

## 14. 待确认项（实现时逐条落实，不要凭记忆填）

- [x] DeepSeek 可用模型 id → 已实测：账号下为 `deepseek-v4-pro` 与 `deepseek-flash`，**不存在** `deepseek-chat`；默认值已改为 `deepseek-v4-pro`（见 `docs/decisions/003-deepseek-model-ids.md`）。上下文长度与 JSON 输出支持情况见同一记录。
- [ ] 是否需要 Semantic Scholar API key（影响候选召回质量）—— **仍未接入**：`Source` 协议已就绪，加一个实现即可；当前仅 arXiv，召回靠多查询覆盖
- [x] `weasyprint` 在目标机器（macOS）的依赖可用性 → 已确认可用：`brew install pango` + 在 import 前设置 `DYLD_FALLBACK_LIBRARY_PATH=/opt/homebrew/lib`，WeasyPrint 保持主渲染器，Chrome 仍为降级路径（见 `docs/decisions/001-runtime-and-renderer.md`）
- [x] 报告页数与单篇卡片上限 → 已用真实运行定标：11 页 / 34,962 prompt + 7,037 completion token / 10 次调用（见 `docs/SPEC.md` §4）。字段级上限由 schema 强制（每篇 ≤6 条结论、≤5 条局限、摘要 ≤120 字），候选池由 `BRIEFING_MAX_CANDIDATES=40` 界定。

> 实现过程中发现的**已知缺口**（与文档不一致或尚未实现的部分）统一记在 `docs/SPEC.md` §3，改动相关代码时请一并核对。

---
> Source: [zhoushy93/multi-agent-literature-briefing](https://github.com/zhoushy93/multi-agent-literature-briefing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-01 -->
