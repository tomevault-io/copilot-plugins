## hr-skillforge

> HR SkillForge / HR methodology-to-deliverable distiller. Distill public evidence from consulting firms, benchmark companies, and HR software companies into installable company-specific HR methodology Skills. Emphasizes English target normalization, full-strategy distillation by default, anysearch-enhanced discovery, public video/audio/transcript handling, source cards, evidence scoring, method-to-deliverable logic, eight research layers, quality gates, and explicit evidence boundaries.


# HR SkillForge / HR 方法论 Skill 锻造器

This Skill forges company-specific HR methodology Skills from public evidence about consulting firms, benchmark companies, and HR software companies. It does not write company profiles; it generates reusable `SKILL.md` files that preserve the source organization's methodology and convert that methodology into evidence-bounded consulting deliverables for real HR, organization, talent, performance, rewards, leadership, culture, people analytics, and change-management problems.

本 Skill 用于把咨询公司、标杆公司或 HR 软件公司的公开资料，蒸馏成可安装、可调用的 company-specific HR 方法论 Skill。目标不是写公司介绍，而是生成既保留来源组织方法论、又能把该方法论转化为证据边界内咨询交付物的 `SKILL.md`，用于解决真实 HR / 组织 / 人才 / 绩效 / 薪酬激励 / 领导力 / 变革问题。

## Operating Defaults / 运行默认规则

- Default to full-strategy distillation, not MVP or fast mode. Use the richest practical search depth, full source-card coverage, video/audio expansion, quality gates, and a practical installable `SKILL.md` unless the user explicitly asks for a quick probe.
- Before searching, normalize the distillation target into an English search name. If the user provides Chinese, translated, abbreviated, or ambiguous names, remind the user that the target is being searched as its English/public name and include the Chinese name as an alias rather than the primary search key.
- For international targets, do not rely on Chinese target names as the primary query string. Chinese names can create encoding issues, irrelevant search results, and weak official-source recall.
- Examples: `麦肯锡` -> primary search target `McKinsey`; aliases `麦肯锡`, `McKinsey & Company`, `McKinsey People & Organizational Performance`. `美世` -> `Mercer`. `光辉合益` / `Hay` -> `Korn Ferry Hay Group` or the best public English brand.
- When running scripts, prefer `--mode full --depth standard` as the default. Escalate to `--depth deep` when the target is important, official sources are sparse, or the user asks for a complete playbook.
- Default all user-facing outputs to bilingual Chinese and English. This includes generated `SKILL.md`, distillation reports, evidence summaries, quality notes, warnings, usage instructions, and final answers unless the user explicitly requests one language only.
- Default to method-to-deliverable distillation. A deliverable is valid only when it is traceable to the distilled methodology; a methodology is usable only when it can guide concrete deliverables.

中文规则：默认做“全攻略蒸馏”，不要默认做 fast/MVP。蒸馏对象必须先确认英文公开名称；如果用户输入中文公司名或中文方法论名，要提醒使用者“将按英文公开名称搜索，中文作为别名补充”，再开始搜索和生成。所有面向使用者的输出默认中英文双语，除非用户明确要求只用一种语言。默认执行“方法论到交付物”的蒸馏：只有能追溯到蒸馏方法论的交付物才是有效交付物；只有能指导具体交付物的方法论才是可用方法论。

## Core Principles / 核心原则

- Input is a company, institution, product, or method source; output is an installable Skill, not a research report.
- The output Skill must preserve the source methodology's logic and convert that logic into concrete deliverable patterns.
- Do not create generic HR deliverables disconnected from the source methodology.
- Do not summarize methodology without showing how it drives diagnosis, interventions, and deliverables.
- Use public sources, user-authorized sources, and compliant video/audio/subtitle/transcript sources only.
- Video, webinar, interview, podcast, and transcript sources are high-weight and must not be silently skipped.
- Video/audio discovery is mandatory, but subtitle capture must be time-boxed and non-blocking. Failed subtitles become explicit evidence gaps, not silent blockers.
- Distillation information volume still determines success or failure. Do not let non-blocking subtitle rules inflate quality: if usable evidence is thin, downgrade to `Usable Candidate` or `Light Version`.
- Every source should become a source card; every core judgment should be traceable to evidence or explicitly marked as professional inference.
- If evidence is insufficient, generate a `Light Version`; do not fabricate a complete methodology.
- Do not bypass login, CAPTCHA, membership, private-video restrictions, paywalls, or platform controls.
- For current facts, regulations, labor-market data, compensation data, product details, or recent news, refresh public sources before answering.

## Core Distillation Chain / 核心蒸馏链条

Every distillation must follow this chain:

```text
Source Evidence
-> Method Principle
-> Operating Logic
-> Diagnostic Lens
-> Intervention Pattern
-> Deliverable Pattern
-> Evidence Boundary
```

每次蒸馏都必须遵循以下链条：

```text
来源证据
-> 方法原则
-> 运作逻辑
-> 诊断镜头
-> 干预模式
-> 交付物模式
-> 证据边界
```

Definitions:

- `Method Principle`: What the source organization appears to optimize for, reject, or repeatedly emphasize.
- `Operating Logic`: How the principle works as a decision rule or management mechanism.
- `Diagnostic Lens`: The questions and signals used to interpret a user's problem through that methodology.
- `Intervention Pattern`: The type of management action implied by the method.
- `Deliverable Pattern`: The concrete consulting artifact that should be produced from the intervention.
- `Evidence Boundary`: What can be claimed from public evidence, what is inference, and what requires user data.

中文定义：

- `方法原则`：来源组织反复强调、优化或反对的核心方法论主张。
- `运作逻辑`：该原则如何作为决策规则或管理机制运转。
- `诊断镜头`：用该方法论理解用户问题时应提出的问题和观察信号。
- `干预模式`：该方法论自然推出的管理动作类型。
- `交付物模式`：由干预模式生成的具体咨询交付物。
- `证据边界`：哪些可由公开证据支持，哪些属于推断，哪些需要用户数据。

## Default Workflow / 默认工作流

```text
target_name + target_type
-> normalize to English/public search name and aliases
-> init target directory
-> create or read sources.csv
-> check search provider availability, especially anysearch
-> generate query matrix, source-card search plan, and video/audio search plan
-> run source-card-driven search, with anysearch preferred when available
-> fetch web pages, PDFs, transcripts, video/audio pages, and manual files
-> capture subtitles/transcripts only when compliant and authorized
-> clean transcripts
-> create source cards with evidence score
-> extract method principles, operating logic, diagnostic lenses, intervention patterns, and deliverable patterns
-> merge into eight research files
-> build company-specific SKILL.md
-> quality_check -> quality-report.md
-> if possible, run a second distillation pass to improve practicality and bilingual usability
```

## Eight Research Agents / 八类研究文件

| Agent | Output | Focus |
|---|---|---|
| Source Evidence Map / 来源证据地图 | `01-source-evidence-map.md` | Source strength, usable claims, method signals, evidence limits |
| Method Principles / 方法原则 | `02-method-principles.md` | Core principles, what they optimize for, what they reject, confidence |
| Operating Logic / 运作逻辑 | `03-operating-logic.md` | Logic chains, decision rules, required roles, required data |
| Diagnostic Lenses / 诊断镜头 | `04-diagnostic-lenses.md` | Trigger problems, diagnostic questions, signals, anti-patterns |
| Intervention Patterns / 干预模式 | `05-intervention-patterns.md` | Linked principles, actions, owners, governance, risks |
| Deliverable Patterns / 交付物模式 | `06-deliverable-patterns.md` | Linked deliverables, required sections, tables, artifacts, minimum detail |
| Application Protocol / 调用协议 | `07-application-protocol.md` | Problem types, method lenses, reasoning paths, deliverables, assumptions |
| Localization and Limits / 本地化与边界 | `08-localization-and-limits.md` | China/context conversion, local management language, unsupported claims, refresh needs |

Older seven-file outputs may still exist for legacy runs, but new distillations should explicitly synthesize the eight-layer method-to-deliverable chain.

旧版七类研究文件可继续作为历史产物存在，但新蒸馏必须明确综合八层“方法论到交付物”链条。

## Source-Card-Driven Search / 证据卡驱动搜索

Before searching broadly, define the evidence cards the final Skill needs. Search by card type, not only by generic keywords.

Core source-card types:

- `official_methodology`: official methodologies, model definitions, white papers, capability pages.
- `method_principle`: recurring principles, decision beliefs, management philosophy, value equations.
- `operating_logic`: logic chains, operating mechanisms, governance rules, decision flows.
- `diagnostic_question`: diagnostic questions, maturity models, assessment logic.
- `tool_template`: scorecards, checklists, templates, toolkits.
- `intervention_pattern`: transformation actions, operating model changes, capability-building moves, governance changes.
- `deliverable_pattern`: consulting deliverables, artifact structures, required fields, tables, dashboards, roadmaps.
- `case_practice`: client cases, implementation paths, management actions.
- `video_transcript`: webinars, podcasts, interviews, subtitles, transcripts.
- `video_metadata`: video/podcast/webinar title, description, publisher, date, official page, playlist title, tags, chapters, and search snippets when transcript text is unavailable.
- `external_boundary`: external views, controversy, limitations, failure conditions.
- `china_localization`: China cases, Chinese wording, localization conversion.

If a card type has fewer than three usable sources, expand with target aliases, named methodologies, consultant names, report titles, conference names, client cases, and Chinese/English synonyms.

Every deliverable pattern must be linked to at least one method principle and one intervention pattern. Do not collect generic templates that cannot be traced back to the source methodology.

每个交付物模式必须绑定至少一个方法原则和一个干预模式。不要收集无法追溯到来源方法论的通用模板。

## Search Provider Rules / 搜索工具规则

- Prefer `anysearch` when installed. Use it for broad web search, batch query expansion, full-text extraction, video/audio page discovery, PDF discovery, and source export.
- anysearch is a discovery and extraction layer; it does not replace source cards, evidence scoring, quality gates, or video/subtitle rules.
- If anysearch returns Markdown search results, parse result titles, URLs, snippets, dates, and source types into `sources.csv`.
- If anysearch is unavailable, rate-limited, or returns low-quality results, fall back to browser/web search, official site checks, video platform search, and manual sources.

## Video / Audio / Subtitle Rules / 视频音频与字幕规则

Video and audio are not auxiliary. Formal distillation must process at least one round of video/audio/subtitle or transcript source discovery.

The principle is strict:

```text
video discovery is mandatory
subtitle capture is time-boxed
failed subtitles become video_metadata evidence
video_metadata supports topic discovery, not content claims
information volume still decides Skill quality
```

原则必须严格：

```text
视频发现必须做
字幕抓取限时做
字幕失败后降级为 video_metadata 证据
video_metadata 只支持主题发现，不支持内容主张
蒸馏信息量仍然决定 Skill 质量
```

Priority:

```text
official transcript / podcast transcript
-> official webinar or podcast page
-> YouTube/Bilibili manual subtitles
-> YouTube/Bilibili auto subtitles
-> user-authorized local audio/video transcription
-> video/audio page metadata
-> no subtitles: record the failure and evidence gap
```

When using `yt-dlp`, download subtitles only, not video files.

```powershell
yt-dlp --list-subs "VIDEO_URL"
yt-dlp --skip-download --write-subs --write-auto-subs --sub-langs "en-orig,en,en-US,en-GB,zh-Hans,zh-Hant,zh,zh-CN,zh-TW" --sub-format "srt/vtt/best" "VIDEO_URL"
```

If YouTube requires login or triggers anti-bot restrictions, stop automatic capture, record the item in `pending_transcription.md`, and ask the user for an authorized transcript, exported cookies file, or local file. Do not bypass platform controls.

Subtitle capture must not block the full pipeline. Recommended limits:

```text
single video subtitle attempt: 60 seconds
playlist subtitle attempt: 90 seconds
entire subtitle stage: 8-10 minutes
```

字幕抓取不得阻塞完整流水线。建议限制：

```text
单个视频字幕尝试：60 秒
播放列表字幕尝试：90 秒
整个字幕阶段：8-10 分钟
```

Playlist handling:

```text
If URL is a YouTube/Bilibili playlist:
-> do not fetch the entire playlist by default
-> record the playlist as video_metadata
-> sample only the first 3-5 highly relevant items when possible
-> if sampling is slow, stop and log transcript_gap
```

播放列表处理：

```text
如果 URL 是 YouTube/Bilibili 播放列表：
-> 默认不要抓取完整播放列表
-> 将播放列表记录为 video_metadata
-> 可行时最多抽取前 3-5 个高相关视频
-> 如果抽取变慢，停止并记录 transcript_gap
```

When subtitles/transcripts fail, keep the source card but downgrade it:

```text
video candidate
-> video_metadata source card
-> topic signal
-> search expansion clue
-> transcript_gap
-> confidence downgrade
```

当字幕/transcript 失败时，仍保留证据卡，但降级：

```text
视频候选
-> video_metadata 证据卡
-> 主题信号
-> 扩展搜索线索
-> transcript_gap
-> 信心降级
```

`video_metadata` may be used only for:

- topic discovery
- official communication signal
- search expansion
- evidence gap logging
- low-confidence source-card coverage

`video_metadata` must not be used for:

- direct quotation
- detailed argument extraction
- consultant wording
- high-confidence methodology claims
- detailed deliverable logic

`video_metadata` 只能用于：

- 主题发现
- 官方传播信号
- 扩展搜索
- 记录证据缺口
- 低信心证据卡覆盖

`video_metadata` 不得用于：

- 直接引用
- 详细论点提炼
- 顾问表达还原
- 高信心方法论判断
- 详细交付物逻辑

If enough official/web/PDF evidence exists, continue generating the Skill and mark the transcript gap. If the transcript gap leaves the overall evidence base thin, downgrade quality or produce a `Light Version`.

如果已有足够官方网页、报告、PDF 证据，可以继续生成 Skill，并标注 transcript 缺口。如果 transcript 缺口导致整体信息量不足，必须降级质量或生成 `Light Version`。

After YouTube fails, immediately expand to:

- Official transcript / webinar / podcast pages
- Bilibili, Vimeo, LinkedIn Events
- Apple Podcasts / Spotify / Omny / podcast indexes
- SHRM, Gartner HR, HR Tech, WorldatWork, CIPD, ATD
- Thinkers50, TED/TEDx, business school public lectures

For user visibility, each run should maintain a progress log or `progress.json` with:

```json
{
  "stage": "video_subtitle_fetch",
  "current": 12,
  "total": 62,
  "source_id": "src_0030",
  "success": 1,
  "failed": 8,
  "timeout": 3,
  "metadata_only": 50,
  "next_action": "continue_after_timeout"
}
```

为了让用户知道没有卡死，每次运行应维护进度日志或 `progress.json`，显示当前阶段、处理数量、成功、失败、超时、metadata-only 数量和下一步动作。

## Quality Gates / 质量门槛

Use three levels:

- `Full Version`: strong source base, broad coverage, enough official/recent/video or audio evidence.
- `Usable Candidate`: enough high-signal public sources for analysis and draft solution design, with confidence labels.
- `Light Version`: sparse or weak evidence; generate candidate hypotheses and research summary only.

Video evidence quality levels:

| Level | Evidence | Allowed Use |
|---|---|---|
| High | official transcript or authorized transcript | method details, wording, sequence, examples |
| Medium | subtitles or auto subtitles | topic and argument extraction with caveats |
| Low | official video/podcast page without transcript | official communication signal and topic discovery |
| Very low | title/search snippet only | search expansion clue only |

视频证据质量等级：

| 等级 | 证据 | 可用范围 |
|---|---|---|
| 高 | 官方 transcript 或授权 transcript | 方法细节、表达、顺序、案例 |
| 中 | 字幕或自动字幕 | 带边界地提炼主题和观点 |
| 低 | 无 transcript 的官方视频/播客页面 | 官方传播信号和主题发现 |
| 很低 | 只有标题/搜索摘要 | 只能作为扩展搜索线索 |

For international consulting companies, aim for:

```text
Full Version: 50 total sources, 8 video/audio/transcript candidates, 25 official sources, 10 recent sources.
Usable Candidate: 20 total sources, 5 video/audio/transcript candidates, 10 official sources, 5 recent sources.
```

Even when quantitative gates are met, mark transcript gaps if subtitles or full transcripts were not actually captured.

Do not count `video_metadata` as equivalent to transcript evidence. It can help satisfy video discovery coverage, but it cannot raise content confidence by itself. If method principles or deliverable patterns rely mainly on video_metadata, downgrade confidence and continue searching for official text/PDF evidence.

不要把 `video_metadata` 等同于 transcript 证据。它可以帮助满足“视频发现覆盖”，但不能单独提升内容信心。如果方法原则或交付物模式主要依赖 video_metadata，必须降低信心并继续寻找官方文字/PDF 证据。

Add a method-deliverable integrity check:

```text
Full Version requires:
- core method principles are explicit
- each core principle has operating logic
- operating logic produces diagnostic lenses
- diagnostic lenses connect to intervention patterns
- intervention patterns connect to deliverable patterns
- deliverables state method source, assumptions, and evidence boundary

Downgrade quality if:
- methodology is summarized but not operationalized
- deliverables are generic and not traceable to the distilled methodology
- the Skill can answer with frameworks only
- the Skill can produce deliverables without explaining the method logic
```

增加“方法论-交付物整合度”检查：

```text
完整版必须满足：
- 核心方法原则清晰
- 每个核心原则都有运作逻辑
- 运作逻辑能形成诊断镜头
- 诊断镜头能连接干预模式
- 干预模式能连接交付物模式
- 交付物必须说明方法来源、假设和证据边界

以下情况应降低质量等级：
- 只总结方法论但没有操作化
- 交付物是通用模板，无法追溯到蒸馏方法论
- Skill 可以只讲框架而不交付
- Skill 可以直接给交付物但不说明方法逻辑
```

## Scripts / 脚本默认用法

Default project root:

```text
PROJECT_ROOT/
```

Recommended full-strategy command:

```powershell
python scripts/run_pipeline.py --target "McKinsey" --type consulting_company --mode full --depth standard
```

Deep distillation when the user asks for a complete playbook:

```powershell
python scripts/run_pipeline.py --target "McKinsey" --type consulting_company --mode full --depth deep
```

Avoid using Chinese target names as the primary target for international companies. Use the English public name as `--target`, and record Chinese names as aliases or source-query variants.

## Final Skill Requirements / 最终产物要求

The generated `SKILL.md` should include:

- Bilingual Chinese and English throughout, especially headings, core conclusions, workflows, diagnostic questions, evidence boundaries, and usage instructions.
- Evidence status and quality boundary.
- Honest non-affiliation boundary.
- Core method principles, not generic HR advice.
- Operating logic for each core principle.
- Diagnostic lenses linked to method principles.
- Intervention patterns linked to diagnostic lenses.
- Method-to-deliverable map.
- Application protocol that forces method-to-deliverable reasoning.
- Deliverable standards and default deliverable templates.
- Diagnostic question sets.
- Answer workflow.
- Source-card recall rules.
- Confidence labels.
- Default output templates.
- Anti-patterns.
- China localization notes when relevant.
- Refresh rules for current data and recent trends.

## Method-to-Deliverable Map Requirement / 方法论到交付物映射要求

Every generated Skill must include a table like this:

| Method principle | Operating logic | Diagnostic lens | Intervention pattern | Deliverable pattern |
|---|---|---|---|---|

每个生成后的 Skill 必须包含类似下表的映射：

| 方法原则 | 运作逻辑 | 诊断镜头 | 干预模式 | 交付物模式 |
|---|---|---|---|---|

The map is mandatory because it prevents two failures:

1. `Framework-only failure`: explaining methods without producing usable outputs.
2. `Deliverable-without-method failure`: producing generic deliverables disconnected from the source methodology.

该映射是必需项，用于避免两种失败：

1. `只有框架，没有交付物`：只解释方法论，不生成可用结果。
2. `只有交付物，没有方法论依据`：产出通用交付物，但与来源方法论脱节。

## Generated Skill Application Protocol / 生成后 Skill 调用协议

When solving a user problem, the generated Skill must not jump directly from request to deliverable. It must follow:

```text
1. Identify the relevant method principle.
2. Explain the operating logic in one short paragraph.
3. Translate the user's context through the diagnostic lens.
4. Select the linked intervention pattern.
5. Generate the linked deliverable pattern with concrete details.
6. Mark evidence boundary, assumptions, and data needed for improvement.
```

当生成后的 Skill 解决用户问题时，不得从用户请求直接跳到交付物，必须遵循：

```text
1. 识别相关方法原则。
2. 用一小段说明运作逻辑。
3. 用诊断镜头转译用户场景。
4. 选择对应干预模式。
5. 生成与该干预绑定的交付物，并填充具体内容。
6. 标注证据边界、假设和进一步优化所需数据。
```

Each deliverable must state:

- Which method principle it comes from.
- Which diagnostic lens triggered it.
- Which intervention pattern it supports.
- Which assumptions were made.
- Which user data would improve it.

每个交付物必须说明：

- 来自哪个方法原则。
- 由哪个诊断镜头触发。
- 支持哪个干预模式。
- 使用了哪些假设。
- 哪些用户数据可以进一步增强。

## When To Read Reference Files / 何时读取参考文件

- Search strategy and query matrix: `references/research-framework.md`
- anysearch installation and usage: `references/anysearch-search-provider.md`
- Source-card-driven search: `references/source-card-search-strategy.md`
- Evidence scoring: `references/evidence-scoring.md`
- Video subtitle strategy: `references/video-source-strategy.md`
- Extraction fields and source cards: `references/extraction-framework.md`
- Quality gates: `references/quality-standard.md`

---
> Source: [wangjile200888-eng/hr-skillforge](https://github.com/wangjile200888-eng/hr-skillforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
