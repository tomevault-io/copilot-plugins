## llm-wiki-skill

> 用 Codex 直接维护客制化 LLM Wiki / Markdown 知识库。Use when the user wants Codex to ingest sources, query with citations, lint, analyze knowledge graph, manage review/deep-research queues, or emulate nashsu/llm_wiki without an app, API key, or local model server.


# LLM Wiki Skill

这个 skill 用来让 Codex 直接维护一个可长期积累的 Markdown wiki：原始资料保持不可变，Codex 增量生成和更新 wiki，Obsidian 只是可选的浏览器。

## 核心约定

- 保持三层结构：`raw/` 原始资料、生成的 `wiki/`、控制文档 `purpose.md` 和 `schema.md`。
- Codex 负责生成和维护 wiki 页面；人类负责选择资料、设定优先级、做关键判断。
- 优先把有长期价值的回答写回 wiki，而不是只留在聊天里。重要回答进入 `wiki/queries/` 或 `wiki/synthesis/`。
- 使用 `[[wikilinks]]`、YAML frontmatter、来源引用、`wiki/index.md`、`wiki/overview.md` 和只追加的 `wiki/log.md`。
- 除非用户明确要求导入、移动或删除 source 文件，否则不要修改 `raw/sources/` 下的原始资料。
- 不把整个 Obsidian vault 自动纳入 wiki。`2-Learnings/EmbodiedWorld/`、`2-Learnings/Courses/`、`0-Daily/`、`1-Meetings/` 是宽输入池；`3-Projects/` 是围绕特定项目组织学习、方案和工作推进的项目层；`4-Research-Wiki/raw/` 是已经决定进入该专题 wiki 的精选证据库；`4-Research-Wiki/wiki/` 是 Codex 编译后的理解层。
- 判断资料是否进入 `raw/`，看它是否值得成为该专题的长期证据，而不是看它有没有 property/frontmatter。

## 原子笔记原则

Research-Wiki 的长期理解层遵循 Zettelkasten / 原子笔记理念：它不是资料堆放处，也不是把 source 改写成另一篇长文，而是把可复用的想法编译成彼此连接的小卡片。

- 一条稳定笔记只承载一个核心想法：一个 concept、claim、question、method、research-practice 或清晰的关系判断。
- `source` 页面可以稍长，用来保存来源上下文；`concept`、`claim`、`question`、`research-practice` 应尽量原子化，不写成无边界综述。
- 写笔记时先问：未来的我会在什么情境下重新需要这个想法？它应该和哪些旧笔记互相发现？
- 不做 source 原文搬运。用自己的话重写理解，同时保留来源引用、PDF/arXiv/网页链接和必要的原句定位。
- 优先创建或更新多条互相链接的小笔记，而不是把多个可独立复用的想法塞进一个大 synthesis。
- 每条 durable note 至少说明：单一想法是什么、来自哪里、连接到哪些旧笔记、还存在哪些待验证问题。
- 每条 durable note 必须能通过 `wiki/index.md`、`wiki/overview.md` 或一条清晰的 wikilink 路径重新找到；孤立笔记不算完成。
- 如果一条内容没有未来复用途径、没有可连接对象，也不能形成概念/主张/问题/方法论，就暂时留在 Daily、Projects 或 raw，不进入 wiki 理解层。

## 输入层边界

推荐关系：

```text
2-Learnings/EmbodiedWorld / 2-Learnings/Courses / 0-Daily / 1-Meetings
= 未筛选或半筛选输入池

3-Projects
= 项目级学习、方案、推进记录和待验证判断

4-Research-Wiki/raw
= 已决定进入世界模型研究语境的证据库

4-Research-Wiki/wiki
= Codex 编译后的概念、主张、问题、综合与方法论
```

操作原则：

- 外层资料池可以宽，`4-Research-Wiki/raw` 必须精选，`4-Research-Wiki/wiki` 必须编译。
- `2-Learnings/EmbodiedWorld/` 不放进 `4-Research-Wiki/` 内部；它是世界模型专题学习层、paper cards 和候选池，不再包一层 `Papers/`。
- `2-Learnings/Courses/` 单独保存课程资料、课件和课程笔记；课程中的稳定概念或方法论应先蒸馏为 study note，再进入 Research-Wiki。
- `0-Daily/` 和 `1-Meetings/` 不全量进入 `raw/`；只把蒸馏后的高价值片段放入 `raw/sources/daily-distillations/` 或 `raw/sources/meeting-extracts/`。
- `3-Projects/` 不保存实验代码、训练日志和论文写作工程；它保存项目目标、方案推理、定向学习、任务拆解、风险判断和阶段总结。
- Projects 中稳定、可复用、跨项目成立的概念、claim、open question 或 research-practice，才进一步蒸馏进入 `4-Research-Wiki/`。
- README 只保留在 vault 根目录下的一级模块中：`0-Daily/README.md`、`1-Meetings/README.md`、`2-Learnings/README.md`、`3-Projects/README.md`、`4-Research-Wiki/README.md`。不要在二级或更深层目录继续散落 README；需要说明时合并进对应一级 README。
- 一份材料进入 `raw/` 前，先问：它是否值得成为 Research-Wiki 的证据？
- 如果不能生成或更新 concept、claim、question、synthesis 或 research-practice，就暂时不要进入 `4-Research-Wiki/wiki/`。
- 进入 `4-Research-Wiki/wiki/` 时，先判断是否应拆成原子笔记：source summary 负责记录来源，长期理解优先拆成可链接的 concept、claim、question 或 research-practice。
- 原始输入文件可以没有 property/frontmatter；生成的 wiki 页面必须有 frontmatter。
- 多机器同步时，skill 源文件保存在 vault 内的 `.tools/skills/llm-wiki-skill/`；每台 Mac 通过 `~/.codex/skills/llm-wiki` 软链接到该目录。新机器按 `Vault Setup.md` 运行 `.tools/setup-vault-mac.sh`，不要手动复制整份 `~/.codex/`。

Daily 输入层包括：

- 文本形式的日常记录：想法、问题、反思、灵感、即时理解。默认写入 `0-Daily/Diary/YYYY-MM-DD.md`；历史研究日记和已整理文本记录保存在 `0-Daily/Diary/Research Diary.md`。注意目录名是大写 `Diary`，不要使用 `0-Daily/diary/` 或旧路径 `0-Daily/Research Diary.md`。
- AI 聊天记录：GPT、Gemini、NotebookLM 等对话。它们是思考痕迹和候选观点，不是默认可靠来源；不需要每天处理，应周期性批量提取核心知识。
- 手机截图：主要来自微信/公众号和小红书。它们默认上下文不完整、来源质量不稳定，需要更强筛选。截图原图放入 `0-Daily/Screenshots/assets/`；整理稿（`0-Daily/Screenshots/YYYY-MM-DD-简短主题.md`）**不放内嵌原图**，结构固定为：**上半「总结」**（主题、要点、与 vault 的衔接、待核验），**下半「全部原文（OCR）」**——将截图中**可见文字尽数转写**（含标题、副标题、段落、列表、小标题、按钮文案、评论等版面文字），并标 `ocr.status`；不是「摘录」或大幅删节。**写作前须校验原图**：若宽高异常（例如宽度仅数十像素的长条），说明导出损坏或误选文件，须请用户重导原图后再 OCR，**禁止**仅凭多模态「读图描述」编造正文。论文线索与判断写在「总结」中即可。B 站内容通常不走截图主流程，更适合以链接或剪藏进入 `0-Daily/Clippings/`。
- Clippings / 剪藏：网页文章、公众号网页、访谈、博客、平台内容等网页剪藏。它们属于 Daily 日常摄取，不等同于已进入 Research-Wiki 的证据；不要把截图整理稿默认放进 `0-Daily/Clippings/`。

处理 Daily 时：

- 不直接把 daily 全量摄入 wiki。
- 每日原始日记主入口是 `0-Daily/Diary/YYYY-MM-DD.md`。如果用户说“写进今天的日记”或“记录到 Daily”，优先写入当天日期文件，而不是写入 `0-Daily/Diary/Research Diary.md`；只有历史整理、跨日合并或用户明确要求时，才维护 `0-Daily/Diary/Research Diary.md`。
- 先生成 `raw/sources/daily-distillations/YYYY-MM-DD.md` 这类蒸馏稿。
- 只有能转化为 concept、claim、question、synthesis 或 research-practice 的部分，才进入 wiki。
- AI 聊天应低频、批量、少手动地处理。不要要求用户每天导出或整理；当用户提供一批 ChatGPT/Gemini/NotebookLM 记录时，只提取核心知识。
- AI 聊天中的事实主张需要追溯到论文、原文、实验或可靠资料；如果暂时无法验证，放入 `reviews.md`。
- 从 AI 聊天中只保留：用户反复追问的问题、被用户认可的判断、概念澄清、研究路线、阅读路线、实验想法、待验证 claim。忽略寒暄、工具性问答、泛泛建议和重复解释。
- 推荐批处理路径：`0-Daily/ai-chats/batches/` 存放定期导出的原始聊天记录，`raw/sources/daily-distillations/YYYY-MM-DD-ai-chats.md` 存放蒸馏后的核心知识。
- 截图内容可以保留启发价值，但事实可靠性需要二次验证。**整理稿**须含可复制检索的 **全文 OCR**（版面文字尽量不漏），且**不嵌原图**，避免体积与检索问题。
- Clippings 先提取正文、元数据、论文线索、核心观点、启发和待验证事实；只有能转化为 concept、claim、question、synthesis 或 research-practice 的内容，才进入 wiki。
- 从 Clippings 提炼长期知识时，不按原文段落机械摘要；先识别可复用的单一想法，再写成原子笔记或更新已有笔记。

截图处理分流：

- 如果截图中识别到论文线索，例如标题、作者、arXiv ID、DOI、会议名、论文页面、论文 PDF 或论文总结，应自动进入论文追踪流程：联网搜索对应论文资源，优先找到 arXiv、DOI、PDF、OpenReview、会议页面、项目页和代码仓库；总结论文内容，并判断它与世界模型研究的相关性。截图整理稿默认写入 `0-Daily/Screenshots/YYYY-MM-DD-简短主题.md`：**不得**用 `![[...]]` 嵌入原图；须按 **总结 / 全部原文（OCR）** 两截书写，原文部分**尽录可见文字**。原图只保留在 `0-Daily/Screenshots/assets/`（整理稿中可用 YAML `source` 指路径，不嵌图）。
- 如果截图是公众号、小红书等普通文章或观点内容，同样：**先**在「全部原文（OCR）」中转写全文，**再**在顶部「总结」中写启发、问题、类比、方法论反思和待验证事实。
- 普通平台内容的事实主张不能直接写成 stable claim；未验证时进入 `reviews.md` 或以 `status: review` 保存。
- 论文资源总结可进入 `raw/sources/papers/`；普通文章截图的蒸馏结果进入 `raw/sources/daily-distillations/`。

论文卡片格式：

```markdown
#### 论文英文名
论文中文名或简译｜**核心创新短语**
录用信息｜机构
PDF 链接 或 w/o. PDF
Project page 链接 或 w/o. project page
Code 链接 或 w/o. code
![流程图或关键图](assets/日期/xxx.png)
Figure N: caption 第一段（如有）
- 问题：现有方法的问题、作者核心洞察/分析/假设。
- 方法：精简总结方法。
  - 核心创新 1：本质性创新点。
  - 核心创新 2：本质性创新点。
- 实现：数据集、评估指标、训练/推理 GPU 时或运行开销。
- 边界：依赖假设、可能失效场景、未解决问题。
- 启发：对 world model 研究的启发。
```

放在 **`3-Projects/<项目名>/`** 的 paper cards 与 `2-Learnings/EmbodiedWorld/` 使用**同一套**字段顺序、链接行、`w/o.` 规则、五条 bullet、排序与截图标准。区别在资产位置：配图放在该项目下的 **`assets/YYYY-MM-DD/`**，文件名为 **`YYYY-MM-DD-简短主题-图像用途.扩展名`**，Markdown 内用**相对于该 `.md` 文件**的路径；`alt` 用与文件名一致的**短语义名**。需要完整图注时，优先保存**带 caption 的裁图**或从 PDF/HTML 导出时保留 figure+figcaption；避免仅用裸图又在 `alt` 里粘贴长段 caption。

无论论文来自 `0-Daily` 截图、clipping、`1-Meetings` 链接还是 Zotero，都优先使用这个紧凑 **普通 Markdown** paper card 格式（`####` 四级标题 + 元数据行 + 配图 + 五条 bullet），**不使用** Obsidian `[!paper]` callout。

每张正式 paper card 必须包含一张本地图片；缺图的 card 只能临时标记为候选，不算整理完成。图片行引用本地语义化命名的流程图或关键图。卡片内部尽量不要空行。核心创新短语放在中文标题右侧，用 `｜**核心创新短语**` 连接；它必须是一个可扫读的短语，不写成长句，作用是极简标明论文的核心创新、核心抓手或最值得记住的机制。录用信息和机构写在同一行，用 `｜` 分隔，例如 `CVPR 2025｜清华大学 上海交通大学`；如果没有正式录用信息，写 `arXiv 2025｜机构` 或 `未找到｜机构`。PDF、project page、code 链接没有找到时分别写 `w/o. PDF`、`w/o. project page`、`w/o. code`，不要留空，也不要写泛化的 `未找到`。流程图优先选择 overview、pipeline、architecture、method、framework、score computation 等能概括方法流程的图；如果没有标准流程图，选择最能表达方法结构、benchmark 设计、系统架构或能力分解的关键图。截图只保留图本体和图注，不要截入上下文正文。图注应尽量完整，至少包含 figure number 和 caption 第一段。bullet 总结固定 5 条，语义槽位固定如下：1. 问题，写现有方法的问题、作者核心洞察/分析/假设；2. 方法，精简总结方法并用两条子 bullet 写核心创新点，尤其是本质性创新；3. 实现，写数据集、评估指标、训练/推理 GPU 时或运行开销，不展开过细 baseline 和实验结果；4. 边界，写依赖假设、可能失效场景、未解决问题；5. 启发，必须写对 world model 研究的启发，并作为最后一条。启发应面向世界模型的表示、预测、规划、因果、泛化、数据、评估或科研方法论，不默认绑定 avatar / 3DGS。

论文卡片在同一文档或同一小节中，必须按日期从新到旧排列。优先使用正式录用年份和会议时间；没有会议信息时使用 arXiv 首发/最新版本年份；如果同年有多篇，按月份或 arXiv 编号近似排序。综述、路线图或总览性质条目只有在小节名明确为 `Survey`、`Overview` 或 `Roadmap` 时才可以放在该小节顶部；常规论文 card 必须保持新到旧。整理完成前必须检查排序，不能只依赖写作时的主观顺序。

Meeting 论文处理完成后，`1-Meetings/assets/` 只保留最终会被 meeting 文档引用的流程图或必要图像。下载的 PDF、整页渲染截图、临时裁图和其他中间文件必须删除。图片文件名必须可读且稳定，不使用 `image.png`、`image 1.png`、`figure.png` 这类无意义名称；统一使用 `YYYY-MM-DD-简短主题-图像用途.png`，例如 `2025-10-18-coadaptation-score-flow-with-caption.png`、`2025-11-11-genpriv-st-vae-overview.png`。Markdown 引用和 alt text 也要同步使用语义化名称。

论文截图标准：

- 每张正式 paper card 通常保留一张主图，自动整理默认只放一张；必要时最多可以放两张图。第二张图只在它补充了主图无法表达的关键信息时使用，例如核心 benchmark construction、重要结果矩阵、状态/数据流和方法图分离、或论文核心贡献本来由两张图共同表达。
- 主图优先级为：方法 overview / pipeline / architecture / framework / score computation / benchmark construction；没有这些时，再选择最能表达核心机制的关键图。第二张图优先选择与主图互补的 benchmark / data construction / result schema / task setup，不重复展示相同信息。
- paper card 配图优先级：1. arXiv HTML / 论文 HTML / project page 中的原始 figure 图片，并在 card 内紧跟一行 caption；2. MinerU / Docling 等结构化解析输出的 referenced image 与 figure caption；3. 只有前两者不可用时，才从 PDF 手工裁图。不要在可直接提取 HTML figure 的情况下默认截图。
- 截图必须包含图本体和对应图注，至少保留 figure number 和 caption 第一段；图注太长时可保留第一段和定义关键符号的句子。
- 截图边界要紧：不能截入大段正文、页眉页脚、参考文献、无关段落或相邻图；上下只保留必要留白和图注。
- 不优先使用 teaser、纯效果图、结果拼图或定性案例，除非论文没有方法图，或该图正是论文核心贡献。
- 如果自动截图包含过多正文、图注缺失、图太小、图和 caption 不对应，不能算完成，必须重新裁切或换图。
- 如果论文没有 PDF，但项目页有官方 overview 图，可以使用项目页图；若只有视频，则截取能代表方法或系统的关键帧，并在 alt text 中标明是 key visual。
- `1-Meetings/assets/YYYY-MM-DD/` 只保存最终图；PDF、整页截图、临时裁切、中间渲染图必须清理。`2-Learnings/EmbodiedWorld/assets/` 同理只保存最终卡片图和必要长期资产。

论文处理分层：

- Paper card 阶段是轻量索引：只需要元信息、1-2 张关键图、核心创新短语和 5 条固定 bullet。目标是快速建立可扫读入口，不展开全文结构。
- Dive into paper / 深入论文阶段才进行 PDF-to-Markdown 解析。当用户说“深入读这篇论文”“详细解析”“dive into”“转成 md”“逐节分析”“做深读笔记”时，优先把 PDF 转为结构化 Markdown，再基于 Markdown 深读。
- PDF-to-Markdown 优先考虑 MinerU、Docling、Marker、Unstructured、PDFFigures2 这类结构化解析工具；若本机未安装，先用轻量 PyMuPDF / pdfplumber 作为 fallback，不把缺工具作为停止理由。
- 当前 vault **默认**本地主工具为 **MinerU**：`.tools/mineru-env/bin/mineru`，便捷脚本 **`.tools/mineru-md.sh INPUT.pdf OUTPUT_DIR`**（默认 `pipeline` 后端，纯 CPU 可跑）。做深读解析时优先使用该脚本；输出目录结构与 MinerU 版本相关，以生成物为准。若仍需 Docling，使用 `.tools/setup-docling-mac.sh` 与 `.tools/docling-md.sh`。
- PDF-to-Markdown 结果是深读中间层，不等同于 Research-Wiki。不要把论文全文 Markdown 全量塞进 `4-Research-Wiki/wiki/`；只把经过提炼的 concept、claim、method、evidence、limitation、open question 或 research-practice 写入 wiki。
- 深读产物放在 `2-Learnings/EmbodiedWorld/World Model Papers/deep-dive/`：`{slug}.pdf`（与 `assets/.paper-pdfs/` 硬链接，不重复占空间）、`{slug}-en.md` / `{slug}-zh.md` 成对；MinerU 英文稿可留在 `assets/mineru-out/` 并复制或链接到 `deep-dive`。正文插图统一用 `../../assets/mineru-out/{slug}/.../auto/images/{hash}.jpg`；文首 callout 链到同目录 `{slug}.pdf`。paper card 用 `Deep read EN/ZH` 指向 `deep-dive/`。新深读可用 `.tools/link-deep-dive-pdf.sh SLUG PDF_BASENAME`。只有当内容被判断为世界模型专题长期证据时，再蒸馏进入 `4-Research-Wiki/raw/sources/papers/` 和 `4-Research-Wiki/wiki/`。
- 结构化解析时保留章节层级、公式/表格占位、figure caption、引用线索和关键术语；如果解析质量差，需要回到 PDF 原文核验，不得只依赖损坏的 Markdown。

Deep dive 标准流程：

- **先补外部上下文**：在正式写作前必须联网检索论文的最新状态和高度相关工作，优先查 arXiv、OpenReview、会议页、项目页、GitHub、Hugging Face、作者主页、follow-up 论文、同期批评论文、复现实验、官方回应和后续版本。若发现最新高度相关论文（例如同团队后续、直接批评、benchmark shortcut、pose/action/reward 等互补工作），必须在深读笔记中单独设“相关最新进展 / 后续线索 / 批评与争议”小节。不要只依赖用户给的论文 PDF。
- **先解析，后深读**：正式 deep dive 必须优先用 MinerU 将 PDF 转为结构化 Markdown；只有 MinerU 明确失败或论文没有 PDF 时，才使用 Docling / Marker / PyMuPDF / pdfplumber fallback。不要直接只用 PDF 抽文本或网页摘要写深读，除非用户明确要求快速摘要。
- **产物必须拆分并分层存放**：正式 deep dive 至少保留三份稳定内容：中文深度笔记 `deep-dive/{slug}-zh-notes.md`、中文原文译文 `deep-dive/{slug}-zh-translation.md`、结构化英文 Markdown `deep-dive/assets/{slug}-en.md`。对应 PDF 放在 `deep-dive/assets/{slug}.pdf`；中间 raw txt、临时解析稿、整页截图等也放在 `deep-dive/assets/` 或更上层约定资产目录，不放在 `deep-dive/` 根目录。`deep-dive/` 根目录只放中文笔记和中文译文类 Markdown，加一个 `assets/` 目录；不要把英文稿、PDF、raw txt 直接留在根目录。
- **图片目录约定**：`deep-dive/images/` 不再使用；若需要本地图片目录，统一放在 `deep-dive/assets/images/`。如果图片来自 MinerU 输出，优先直接引用 `../../assets/mineru-out/.../auto/images/...`；移动英文稿到 `deep-dive/assets/` 后，必须同步修正英文稿里的图片相对路径。任何移动文件后都要检查 Markdown 图片链接不失效。
- **跳转链接要求**：每份中文笔记和中文译文开头必须有可点击的 PDF 与英文 Markdown 链接，例如 `PDF：[PDF](assets/{slug}.pdf) · 英文 Markdown：[EN](assets/{slug}-en.md)`，方便在 Obsidian 中从中文稿直接跳转到原始 PDF 和结构化英文稿。
- **中文笔记结构**：`{slug}-zh-notes.md` 是学习型深度笔记，至少包括：一句话结论、论文定位、核心问题、方法拆解、关键实验、最重要图表解释、与 world model / 当前专题的关系、局限与反驳、可迁移启发、可做实验或后续阅读。所有“章节级中文总结”“每节要点”“作者在本节做了什么”都必须融合到这份笔记中，不放进译文稿。
- **中文译文结构**：`{slug}-zh-translation.md` 用 `# 原文中文翻译` 或 `# 原文中文译读` 开头，目标是按论文原章节顺序做逐段中文翻译/译读，而不是章节级中文摘要。每个原文段落都应有对应中文段落；保留原文的章节层级、段落顺序、图表位置、引用编号和公式位置。若版权、篇幅或上下文限制导致不能输出整篇逐字翻译，必须明确标注为“逐段中文译读/学习性释译”，并写清已覆盖章节和未覆盖章节；不得把章节级摘要伪装成全文翻译。
- **翻译规则**：公式、变量、损失函数、算法伪代码、表格数值、引用编号、图号表号、模型名、数据集名、benchmark 名、仓库名、方法缩写和专有术语不要意译或改写；数学表达保留 LaTeX 原样。术语首次出现用“中文（English term）”，后文保持一致。不要把 `latent`、`world model`、`rollout`、`policy`、`reward`、`pose`、`occupancy`、`token` 等核心术语翻成不稳定的口语词。
- **图表处理**：优先复用 MinerU 提取的 figure 和 caption；中文笔记只放最关键的 3-6 张图并解释其作用，中文译文稿按原文位置保留图号、caption 翻译和图片引用。若 MinerU 漏图或错图，再从 arXiv HTML / 项目页 / PDF 裁图补齐。
- **质量门槛**：完成前必须检查：英文结构化稿、中文笔记稿、中文译文稿均存在；中文笔记不混入长篇原文译文；中文译文稿不是章节级摘要；公式未被翻译破坏；图片路径存在；最新相关工作已检索并记录；临时 PDF/整页截图/中间文件已清理或放入约定资产目录。若因论文过长无法一次翻完，必须在中文译文稿中明确标注翻译进度和剩余章节，而不是伪装完成。

Meeting 输入层包括：

- 豆包总结记录：会议总结、转写、要点提炼。它们是快速入口，不是最终证据。
- 拍摄图像：slides、白板、论文截图、方法图、实验表格等。需要先识别文字和结构。
- 论文链接信息：arXiv、DOI、OpenReview、会议页、代码仓库、项目页等。

Meeting 总结默认包含两层：**保守纪要** 和 **会后头脑风暴**。保守纪要只写可由转写、图像、论文链接或原文校验的信息；会后头脑风暴明确标注为研究发散，用来把当天论文、组内项目和更大的研究主线连接起来。

Meeting 默认按日期文档组织：

```text
1-Meetings/YYYY-MM-DD-topic.md
```

一次组会、一次文献整理、一次导师讨论或一次科研交流，对应一个 Markdown 文档。文档里保留日期、简述、关键论文/链接/图片路径和待处理事项即可。文件名使用 `YYYY-MM-DD-简短主题.md`，主题应根据当日核心内容命名，不使用无信息量的固定尾缀。日期文档是输入池，不代表已纳入 Research-Wiki。如果材料来自 Zotero collection，而 collection 本身跨越多个日期，应优先按日期子 collection 名或明确记录中的日期拆成多个日期文档；没有日期子 collection 时，再参考条目的 `dateAdded`。不要把导出日期当成组会日期。

处理 Meeting 时：

- 不直接把整场会议记录编译进 wiki，只提取高价值科研片段。
- 先生成 `raw/sources/meeting-extracts/YYYY-MM-DD.md`。
- 豆包总结中的关键信息需要和图像、论文链接或原论文互相校验。
- 如果图像或链接中识别到论文线索，自动进入论文追踪流程：搜索论文资源，总结论文，判断与世界模型研究的相关性；正式 meeting 文档中的论文应优先补成 paper card，含本地关键图、元信息、链接和五条固定 bullet。
- 与世界模型高度相关的论文资源总结可进入 `raw/sources/papers/`；组会启发、科研通法、问题意识进入 `raw/sources/meeting-extracts/`。
- 未验证事实主张进入 `reviews.md` 或以 `status: review` 保存。
- 推荐 meeting 文档结构：frontmatter、简述、论文列表、Paper Cards、保守纪要/讨论脉络、横向结论、会后头脑风暴、待办。
- `会后头脑风暴` 必须与保守纪要分开写，避免把发散判断伪装成已发生讨论或论文事实。开头写清楚“这一节不是保守纪要，而是研究发散”。
- 头脑风暴不只服务 JEPA / world model；它也应主动连接组内其他课题，例如动作理解、长尾识别、3DGS/PBR、PEFT/LoRA/Adapter、机器人策略、多模态融合、实验设计和论文写作方法论。也可以另设“个人兴趣连接”，把音乐生成、折纸艺术等用户长期兴趣作为灵感来源，但不要误写成组内课题。
- 头脑风暴可固定回答这些问题：这篇论文预测的是 pixel、token、latent、object state、reward 还是 action？它的状态表示是隐式还是显式？prediction 是训练期辅助任务还是测试时决策模块？如果改成 JEPA/world-model/PEFT/3DGS/PBR/长尾识别等组内范式，context、target、loss、模块、数据和指标应如何改？它能否转化为组内可做实验？是否能和个人兴趣如音乐生成、折纸艺术形成概念类比或创作灵感？限制来自数据、任务定义、物理假设、评价指标、baseline 还是计算资源？
- 对头脑风暴中的想法分级：`可立即做实验`、`需要补文献`、`概念启发`、`高风险猜想`。只有经过后续验证、可复用且跨项目成立的想法，才进一步蒸馏进 Research-Wiki 的 concept、claim、question 或 research-practice。

## Wiki 目录结构

默认项目根目录：

```text
<project>/
  purpose.md
  schema.md
  raw/sources/
  raw/assets/
  wiki/index.md
  wiki/log.md
  wiki/overview.md
  wiki/entities/
  wiki/concepts/
  wiki/sources/
  wiki/claims/
  wiki/questions/
  wiki/queries/
  wiki/synthesis/
  wiki/comparisons/
  wiki/reviews.md
  wiki/research.md
  .llm-wiki/manifest.json
  .llm-wiki/queue.json
  .llm-wiki/graph.json
  .llm-wiki/lint.json
```

用下面命令创建结构：

```bash
scripts/llm_wiki_tools.py init <project>
```

## 页面 Frontmatter

每个生成的 wiki 页面都必须用下面的 YAML frontmatter 开头：

```yaml
---
title: "Readable title"
type: "source|entity|concept|claim|question|query|synthesis|comparison|overview|index"
status: "seed|draft|review|stable|deprecated"
created: "YYYY-MM-DD"
updated: "YYYY-MM-DD"
sources:
  - "raw/sources/example.md"
tags: []
---
```

说明：

- `sources` 使用相对路径。
- 生成页面文件名使用稳定的小写连字符格式，例如 `large-language-model.md`。
- 长期 wiki 页面正文优先使用中文；字段名和枚举值保持英文，方便脚本和未来工具读取。

## 页面写作规范

- 正文主体用中文。重要英文术语首次出现时写成 `中文（English term）`，后文可按语境使用中文或缩写。
- 数学符号、变量、概率表达式和结构方程必须使用 LaTeX：行内用 `$...$`，块级用 `$$...$$`。
- 不要用反引号包裹数学符号或公式，例如不要写 `` `PA_i` ``、`` `P(Y | X=x)` ``；应写成 $\mathrm{PA}_i$、$P(Y \mid X=x)$。
- 反引号只用于文件路径、命令、代码标识符、字段名、枚举值、脚本参数和确实属于代码的片段。
- 论文标题、模型名、数据集名、仓库名和链接保持原文；必要时在中文解释后保留英文括注。

## 工作流

### 初始化

1. 只有当项目目录不明确时才询问用户。
2. 运行 `scripts/llm_wiki_tools.py init <project>`。
3. 如果用户给了领域目标，按领域改写 `purpose.md`。
4. 把 `schema.md` 当作操作契约；当目录、页面类型、引用规则变化时同步更新。

### 摄入资料

采用 `nashsu/llm_wiki` 启发的两步摄入法：

摄入前先确认资料是否应该进入该专题 wiki：

- **一定纳入**：与世界模型直接相关的论文、综述、课程笔记、学习总结；会长期复用的概念；能形成明确 claim 的内容；改变已有理解的材料。
- **选择性纳入**：组会中的高价值片段；daily 中经过蒸馏的科研想法；AI 聊天中被用户认可且可追溯/可验证的观点；截图触发的科研类比、问题意识或方法论启发；低度相关但高启发的材料；可复用科研通法。
- **不纳入**：临时 todo、行政信息、日常流水、只对当天有意义的记录、没有形成问题/概念/claim/方法论启发的摘抄、实验代码、实验日志、论文正文工程、单纯“以后可能有用”的剪藏。

1. **分析**：阅读 source、`purpose.md`、`schema.md`、`wiki/index.md`、`wiki/overview.md` 和相关旧页面。提取实体、概念、主张、证据、矛盾、可能要创建/更新的页面、review 项、deep research 线索。
2. **生成**：创建或更新 source 摘要页，然后把长期价值拆成受影响的 entity/concept/claim/question/research-practice/synthesis 页面。能原子化的内容优先进入小笔记；只有跨多条笔记形成叙事、比较或阶段性判断时，才写 synthesis。更新 `index.md`、`overview.md`、`reviews.md`、`research.md`，并向 `log.md` 追加记录。

摄入前先运行：

```bash
scripts/llm_wiki_tools.py manifest <project>
```

如果 source 没变化，默认跳过，除非用户要求重新摄入。摄入文件夹时保留相对路径，并把文件夹路径当作分类上下文。

Source 摘要页放在 `wiki/sources/`，必须包含：

- 这份资料实际说了什么，而不是泛泛背景介绍
- source 元数据和路径
- 关键主张、证据、结论
- 指向相关页面的 `[[wikilinks]]`
- 开放问题和需要人工判断的 review 项

### 查询

1. 读取 `purpose.md`、`schema.md`、`wiki/index.md` 和 `wiki/overview.md`。
2. 运行 `scripts/llm_wiki_tools.py search <project> "<query>"`。
3. 阅读搜索结果中的高相关页面；必要时再读图谱邻居页面。
4. 回答时引用 wiki 页面和 source 路径。
5. 如果答案有长期价值，把它创建或更新到 `wiki/queries/`、`wiki/synthesis/` 或 `wiki/comparisons/`，然后更新 `index.md` 和 `log.md`。

### Lint / 健康检查

运行：

```bash
scripts/llm_wiki_tools.py lint <project>
```

然后修复或记录：

- 缺失 frontmatter
- 断掉的 `[[wikilinks]]`
- 孤立页面或弱连接页面
- `sources` 过期或缺失
- 重复实体/重复概念
- 与更新或更强来源冲突的 claim
- 被反复提到但没有独立页面的重要概念

Lint 完成后向 `wiki/log.md` 追加记录；不能自动判断的事项放到 `wiki/reviews.md`。

### 图谱和洞察

运行：

```bash
scripts/llm_wiki_tools.py graph <project>
```

使用 `.llm-wiki/graph.json` 识别：

- 直接 `[[wikilinks]]`
- 共享 source 的页面
- Adamic-Adar 风格的共同邻居关联
- 同类型页面关联
- 用 connected components 作为无依赖版 Louvain 替代
- 孤立页面、稀疏社区、桥接页面、意外跨类型连接

如果用户要“图谱视图”，不要假装有 App UI；用 Markdown 表格和 Mermaid 图替代。

### Deep Research / 深度研究

如果需要当前网页信息，并且用户允许或明确要求联网，使用 Codex 浏览网页做研究。否则在 `wiki/research.md` 创建研究任务，包含：

- topic
- 为什么它对 `purpose.md` 重要
- 建议搜索查询
- 相关页面
- 预期输出页面

研究完成后，把结果保存为 `wiki/synthesis/` 或 `wiki/sources/` 页面，并按正常摄入流程更新链接、索引和日志。

### Review Queue / 人工审核队列

用 `wiki/reviews.md` 记录异步人工判断。每个 item 应包含：

- id、date、status
- source/path
- issue
- recommended actions：create page、merge pages、mark conflict、deep research、skip
- 如果需要研究，提前写好 search queries

不要因为主观判断阻塞摄入；保守记录，继续处理确定部分。

## 功能等价说明

这个 skill 可以通过 Codex + 文件系统模拟 `nashsu/llm_wiki` 的大部分信息效果。它不能原生提供桌面 UI、后台文件夹监听、流式聊天持久化、Chrome 扩展、LanceDB 向量搜索、Tauri 系统集成或真正的 Louvain 可视化布局。

替代方案：

- App UI → Markdown 文件、Codex 摘要、Mermaid、JSON 报告。
- 后台 watcher → 每次会话前或用户要求同步时运行 `manifest`。
- 持久队列 → `.llm-wiki/queue.json` 加 `wiki/log.md`。
- 向量搜索 → 关键词搜索加图谱扩展；以后可接入 qmd 或 embedding 工具。
- Louvain → `graph` 里的 connected components 和稀疏社区启发式。
- 网页剪藏 → 把剪藏后的 Markdown 放进 `raw/sources/`，或让 Codex 浏览网页后归档 source 页面。
- 多模态图片/PDF → 能提取文本就先提取；图片只有在作为本地文件或对话附件可见时再用 Codex 视觉能力处理。

## 参考文件

- 当需要判断 `nashsu/llm_wiki` 的某个功能在无 App 工作流里怎么替代时，读取 `references/feature-map.md`。
- 需要确定性维护命令时，运行 `scripts/llm_wiki_tools.py --help`。

---
> Source: [WangPu1999/llm-wiki-skill](https://github.com/WangPu1999/llm-wiki-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
