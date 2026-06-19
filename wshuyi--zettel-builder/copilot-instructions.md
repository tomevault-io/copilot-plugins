## zettel-builder

> |


# zettel-builder

让卡片**自己长出来**:素材进 → 切卡 → 连边 → 巡检 → 拼文。用户在每个分叉点保留决策权,但不必逐张手动建卡。

## 核心理念(凡修改先读一遍)

1. **原子化是为了可重组,不是为了字数小**。判断一张卡是否合格的硬标准:**脱离上下文,半年后的你能不能独立读懂?** 能,就是合格,几百字也行。不能,再短也是垃圾。
2. **"用自己的话重述"不是同义词替换**。必须产生新判断:这个观点和我已知的什么冲突?边界条件是什么?我相不相信?没有这一步,重述只是更慢的复制。
3. **链接的价值在于**写出*为什么连*。盲连等于不连。每条 link 必须带 `reason` 字段(一行话)。
4. **文章不会"自己长出来"——文章是从已有写作方向上的稠密簇中浮现的**。本 skill 帮你更高效地组织既定方向的材料,不替你决定写什么。这是诚实的边界。
5. **触发分两层**:mechanical scan 由 systemd 离线跑(纯 Python,不切卡只列待办);agent scan 由用户或 `/schedule` 触发(真正切卡 + 价值观重述 + 连边)。

参见 `references/philosophy.md`(为什么这样设计 + 与 topic-inspiration 的区别)。

## 路径默认

| 参数 | 默认 | 说明 |
|---|---|---|
| `zettel_root` | `~/zettel/` | 卡片仓根目录,独立 git 仓 |
| `wiki_root` | `~/wiki/published-articles/` | 上游素材来源 |
| `raw_articles` | `${wiki_root}/raw/articles/` | 已发表文章源(241 篇+) |
| `raw_getnote` | `${wiki_root}/raw/getnote/` | GetNote 笔记同步(1974+) |
| `published_source` | `~/Dropbox/cn_articles_published/` | 已发表文章上游(去重对照) |

## 卡片格式硬规则

完整规范见 `references/card-format.md`。一行话总结:

```yaml
---
id: YYYYMMDD-HHMM-<kebab-标题>
status: fleeting | literature | permanent
created: <ISO8601>
source: {type: ..., ref: ...}
tags: [...]
entities: [...]
links: [{id: ..., reason: ...}]
voice_passed: true|false
---

# 标题(直接陈述一个判断,不是名词短语)

正文(150-800 字,可超,以脱离上下文可读为准)。
```

**永远不要**:
- 同时往一张卡塞多个独立判断(切两张)
- 标题写成名词("原子化原则")而不是判断("原子化的目的是可重组")
- 链接只列 id 不写 reason
- 把 raw 原文复制进卡(literature note 只引路径 + 短摘录)

## 四种模式

### Mode 1: ingest — 单次喂料

用法:`/zettel ingest <来源>`。来源可以是:
- 一段贴在对话里的文本
- 一个文件路径(`/zettel ingest /path/to/note.md`)
- 一个 URL(走 markdown-proxy 先抓正文)
- 本次对话上方的若干消息("把上面那段对话切卡")

**步骤**:

1. **来源归一**:把素材转为 Markdown 正文 + 来源元数据(type/ref/timestamp)。
2. **原子切分(v1.13.0 加强:长文必须多卡)**:
   - 识别**所有独立判断**,每个一张卡。判断标志:"X 不是 Y,而是 Z"、"X 的真正原因是…"、"X 和 Y 的边界是…"、"X 表面 ... 实际 ..."
   - **wiki-article 长文必须通读全文找全 candidates**,典型字数对应产卡数:短文 2-3 张 / 中长 3-6 张 / 长文 6-10 张 / 巨长 10-15 张
   - 若 > 1500 字的 article 只产 1 张卡,**必须**在 `processed/<date>.jsonl` 写明理由(否则视为信息压缩失败)
   - 详细判读流程见 `references/voice-alignment.md` Step B.2
   - **列举不算判断**(三条原则、四种类型),除非每条独立成段并含论证
3. **价值观重述**:对每张卡,调 `references/voice-alignment.md` 的协议——读 `references/voice-snapshot.md`(你自己的价值观/文风快照)的核心条款,重写卡的正文。**关键**:不是改文风,是产生新判断。如果原文是引述他人,重述时必须加上"我同意/不同意/部分同意,因为…"。
4. **链接候选**:跑 `scripts/link_candidates.py --card-id <new-id>`,得到 top-10 候选卡。对每个候选,agent 判定是否真连 + 写一行 reason。无连不强求。
5. **去重 + 链接(v1.6.0 agent-judged)**:嵌入只做**召回**,**最终判断必须 agent 读全文**。
   - **召回阶段**(嵌入):算候选卡嵌入,跟 `_index/embeddings.npy` cosine,取:
     - 近邻 top-20(sim 倒序)
     - **远距 top-5**(从 sim<0.5 的卡里随机抽,关键 — 防语义局部扎堆)
   - **判断阶段**(必须 Read 候选卡全文):
     - 对近邻候选:Oven 读全文,判定"这是否真的同一判断"。**不**基于 sim 自动跳过。
       - 真同一判断 → dedup-skip,记 `{action:"dedup-skip", existing:<id>, agent_reason:"..."}`
       - 表面相似但实质不同 → 写新卡,但**必须自动 link** 到那张近似卡,relation + reason **说清差异**
     - 对远距候选:Oven 读全文,判定"有没有非显然的跨域关联"。命中即建立 link,relation 通常 `extend` / `counter`,reason 说明跨域桥梁。
   - **绝不**直接用 sim ≥ 0.85 自动跳过 — 必须 agent 看完判定
   - **绝不**只从近邻候选选 link — 至少考察 5 张远距,防扎堆
5.5. **filename + id 生成(v1.9.0,关键)**:写盘前确定 filename 和 frontmatter:
   - `id`(frontmatter)= 13 字符 `YYYYMMDD-HHMM` 纯时间戳。**不**含标题文字
   - `aliases`(frontmatter)= 完整 H1 标题作为第一条(Obsidian 双链友好,改标题不破链)
   - `filename` = `<short-id>-<主语-主结论>.md`,slug ≤ 20 中文字符,首词主语,英文小写连字符
   - 完整规则见 `references/card-naming.md`
   - 链接(frontmatter `links:` + 正文 `[[]]`)永远引用短 id
6. **图片搬运(v1.4.0)**:若 source 是含 `![](...)` 的 raw markdown,先跑 `python3 scripts/migrate_images.py --card-id <id> --source <raw-path>` 把图复制到 `assets/<id>/`,记录路径 mapping。后续正文用新路径引用关键图(支撑本卡判断的;不必全搬)。
7. **写盘(v1.6.0 self-contained + 实质化引用 + Obsidian 双链)**:`cards/<id>.md`,正文结构:
   - `## 原文要点` 节(blockquote 完整搬运 200-500 字原文,含图)
   - `## 我的判断` 节(选边表态 + 边界 + **必须自然引述 ≥1 张相关卡**)
   - `## 相关卡片` 节(Obsidian 双链索引)

   **关键(v1.6.0):实质化引用**。`## 我的判断` 不能只是孤立陈述,必须在正文里**自然引述**至少一张相关卡:
   - 错(只有 ## 相关卡片 节列条目):`## 相关卡片\n- [[X]] — support: ...`
   - 对(## 我的判断 正文里融入):`...如 [[X]] 所论 A 边界在 B,但本卡聚焦的是 C。[[X]] 强调测量,本卡关心如何处置——这是同一光谱上的两端。`
   
   引述必须言之有物(说出"为什么相关"+"差异点"),不是机械加 `[[]]`。validator 检查 `## 我的判断` 节里至少有 1 个 `[[]]`,否则 voice_passed=false。

   `## 相关卡片` 节继续做 index(给 cluster_inspect 等脚本用),validator 校验两边数量一致。
8. **git push 硬条款**(v1.2.0):写完所有卡片后**必须**执行 `cd ~/zettel && git add -A && git commit -m "feat(ingest): <一句话摘要 N 张卡>" && git pull --rebase origin main && git push origin main`。不依赖 systemd timer 兜底。push 失败必须显式报错(常见原因:无网、远程有冲突)。
9. **回报**:列出新建卡片清单 + 每张的链接情况 + dedup-skip 数量 + push 是否成功。

### Mode 2: scan — 消化 mechanical queue

用法:`/zettel scan`(或由 OpenClaw cron / event-driven trigger)。

**前置**:`scan_mechanical.py` 由 systemd 每小时跑,把 `raw/` 新增文件写进 `_queue/*.json`。Agent scan 消化这些 queue 项。

**步骤**:

1. **状态检查**:读 `_queue/` 当前积压;读 `_state/last_scan.json`;**读 `_state/mode.json` 判定当前模式**(`bootstrap` 或 `steady`)。
2. **批量限额**(v1.3.0 mode-aware):
   - `bootstrap` 模式:N=20(库存高速消化期,挑战长 session token budget)
   - `steady` 模式:N=5(日常消化,event-driven 触发)
   - 读 `_state/mode.json` 中 `thresholds.bootstrap_batch_size` / `steady_batch_size`,出错时 fallback 默认值。
3. **对每个 queue 项**:
   - 读 source 原文(literature note 只引路径,不复制原文)
   - 走 ingest 的 Step 2-5(切卡 + 重述 + 链接 + 写盘)
   - 处理完把 queue 项移到 `_state/processed/<date>.jsonl`
4. **顺手 inspect**:跑一次 Mode 3 inspect 的轻量版,若发现文章就绪簇,发 Telegram 提醒。
5. **更新** `_state/processed/<date>.jsonl`(只动 agent scan 自己的进度)。**绝不**修改 `_state/last_scan.json` — 那是 mechanical scan 的 raw 游标,agent scan 写它会造成新增 raw 文件被跳过(详见 `references/scan-mechanical.md` 游标所有权章节)。
6. **git push 硬条款**(v1.2.0):`cd ~/zettel && git add -A && git commit -m "feat(scan): <N 张新卡 + Y 项处理>" && git pull --rebase origin main && git push origin main`。跨机器同步零延迟。push 失败必须显式报错。

### Mode 3: inspect — 巡检卡片池

用法:`/zettel inspect`(或 scan 末尾自动跑轻量版)。

**目的**:找出三类信号——文章就绪簇 / 孤儿卡 / 缺口。**不自动出文**,只给建议。

**步骤**:

1. **跑** `scripts/build_index.py`:重建 tag/entity/link 索引。
2. **跑** `scripts/embed_lite.py --include-published`:同时刷新 cards/ 和 published-articles 的嵌入,供 overlap 检测用。
3. **跑** `scripts/cluster_inspect.py`:输出 JSON 报告,每个簇含:
   - 成员 id 列表 + 密度 + voice_ratio
   - **gaps**:counter_gap / case_gap / evidence_gap / time_gap(基于簇内 link relation 枚举判定)
   - **published_overlap**(v1.3.0 新增):跟已发表文章的语义重叠 — top-3 候选 + `overlap_class`(`covered` ≥0.75 / `adjacent` 0.6-0.75 / `clean` <0.6)
   - **kind**:`core`(densest 子图)或 `whole`(整连通分量)
4. **LLM 命题**:对每个新就绪簇,把 `suggested_title` 占位符替换为陈述性判断句(不要名词短语)。
5. **去重**:对照 `_state/cluster_history.json`,过滤已建议过的簇(除非密度显著上升)。
6. **回报**(v1.3.0 协议):Telegram 摘要 + 每簇含 `overlap_class` 标记:
   - `covered`:⚠️ 红色 — "主题已被 <已发文章标题> 覆盖,建议跳过或明确新角度"
   - `adjacent`:橙色 — "主题相邻 <已发文章标题>,需要说明差异"
   - `clean`:绿色 — "干净新主题,可推荐 /zettel write"

簇就绪判定细则见 `references/cluster-detection.md`。published-overlap 算法和阈值理由见 `references/published-overlap.md`。

### Mode 4: write — 把簇拼成文章

用法:`/zettel write <cluster-id>` 或 `/zettel write` 后选最就绪簇。

**步骤**:

1. **读簇**:从 `_state/pending_articles.json` 取 cluster-id,加载成员卡。
2. **缺口检查**:若簇含 `gap` 标记,询问用户是否先调 deep-research 补卡(调 `/deep-research` 走 Skill 路由,主题 = 缺口描述,返回后回流为新卡 + 重跑簇就绪判定)。
3. **大纲合成**:按卡片的链接关系排序成大纲(不是按时间)。
4. **移交写作 Skill**:调你自己的写作 Skill(例如 `/your-writer`),输入 = 大纲 + 成员卡正文 + 已链接的相关 wiki concepts/。**关键**:本 skill 不自己写,只把材料组好。
5. **回流**:文章发表后(用户手动通知或 wiki-sync 检测到 published-source 新增),把对应卡的 `cluster_history` 标为 `published`,卡的 frontmatter 加 `produced_article: <wiki-path>`。
6. **git push 硬条款**(v1.2.0):若 cluster_history 或卡 frontmatter 有改动,`cd ~/zettel && git add -A && git commit -m "chore(write): <cluster-id> handed off" && git pull --rebase && git push`。

## 调度集成(双层)

### Layer 1: systemd(无 LLM,hourly)

`scripts/zettel-hourly-sync.sh` 由 `zettel-sync.service`(systemd user unit)hourly 触发,链在 `wiki-sync.service` 之后(`After=` + `Wants=`),保证 raw/ 已刷新。

只做无 LLM 的工作:
1. `build_index.py` 重建 tag/entity/link/term 索引
2. `embed_lite.py` 增量刷新 BGE 嵌入
3. `scan_mechanical.py` 检测 raw/ 新增 → 写 `_queue/`
4. `cluster_inspect.py` 重算 ready/near_ready
5. git pull/push 兜底(以防 LLM session 漏 push)

### Layer 2: OpenClaw cron(LLM session,daily/weekly)

`~/.openclaw/cron/jobs.json` 中两条 job,target=Oven(workspace-openai,GPT-5.5,@your-openclaw-bot):

| Job | 频率 | 干什么 |
|---|---|---|
| `zettel-daily-scan` | `0 9 * * *` | 消化 `_queue/` 中 mechanical scan 攒下的待办,Mode 2 跑一遍,Telegram 摘要 |
| `zettel-weekly-inspect` | `0 10 * * 0` | 周日深巡:LLM 给就绪簇命题、解读 gap,Telegram 推荐 `/zettel write` 候选 |

为什么是 OpenClaw 不是 Hermes:OpenClaw cron 原生支持 `payload.kind: agentTurn`(唤起 LLM 跑一段对话),Hermes cron 只是 script-driven(跑 Python 脚本)。zettel 的 scan/inspect 需要 LLM session,**架构上只有 OpenClaw 能直接表达**。

详见 `references/scan-mechanical.md`(systemd 层)+ `references/openclaw-cron.md`(Oven 层)。

### Mode 5: propose — 命题作文(v1.12.0)

用法:
- 命令式:`/zettel propose <seed-text>`
- 自然语言(给 Oven):"围绕 X 写一篇"、"用 X 这个想法启发,从我笔记里找点料"

**与 Mode 4 write 的区别**:Mode 4 是自下而上(就绪簇 → 文章);Mode 5 是自顶向下(user seed → 从网络挑材料组簇 → 文章)。

**步骤**:

1. **Python 召回**:`python3 scripts/propose_seed.py --seed "<text>"`
   - 算 seed 的 BGE 嵌入
   - 跟 cards/ 嵌入做 cosine,取 top-30 候选
   - 写 `_state/proposal_<ts>.json`(含 seed + candidates + 元数据)

2. **Agent 判定 + 分角色**(必须 Read 每张候选卡正文):
   - **support**:本卡可作论据支撑 seed
   - **counter**:本卡反对 / 限定 seed
   - **case**:本卡是 seed 的具体案例
   - **context**:本卡提供背景 / 前史
   - **unrelated**:看似相关其实不是,丢弃
   不是 sim 高就纳入 — 必须看实际内容判断。

3. **写 proposal 文档**:`~/zettel/_drafts/proposal_<ts>.md`
   - Seed 原文(引言)
   - 分角色卡片清单(每条 `[[<card-id>]] — <role>: <一句话相关性说明>`)
   - 拟标题(基于 seed + 候选卡综合命题,陈述性判断,不要名词短语)
   - 建议大纲(把 seed 作为论点,把候选卡组织成承接它的论证链)
   - 缺口标注(若主要 support 链上有空洞,可触发 deep-research 补卡)

4. **回报**:
   - Telegram 简报 + proposal_id
   - 用户审视 `_drafts/proposal_<ts>.md` 后,可:
     - `/zettel write proposal_<ts>`:移交你自己的写作 Skill 出稿
     - `/zettel propose ...`:换 seed 重来
     - 手动编辑 proposal:增删卡 / 改大纲

5. **git push 硬条款**:proposal 文档写完后 cd ~/zettel && git add/commit/push,跨机器可见

**触发关键词**(Oven 在 Telegram 接到这些自动走 Mode 5):
- `/zettel propose <seed>`
- "围绕 X 写一篇" / "用 X 这个想法启发" / "命题作文 X" / "基于这个想法写"

## 与其他 skill 的关系

| Skill | 关系 |
|---|---|
| **你的写作 Skill** | Mode 4 移交;Mode 1 重述阶段复用其 values + style-guide(在 `references/voice-snapshot.md` 维护快照) |
| **选题/topic-inspiration 类 Skill** | **互补**:那个是看缺口选题(自顶向下),本 skill 是已有素材长卡(自底向上)。两者可以串:选题输出 → 转 deep-research → 卡入 zettel |
| **deep-research** | Mode 4 缺口补漏;调用方式见 `references/deep-research-handoff.md` |
| **你的笔记/收藏源**(如 GetNote、Readwise) | 上游源(通过 wiki-sync 等方式落到 raw/) |
| **闲聊/想法延伸 Skill**(如 getseed) | **不重叠**:那类是把闲聊延伸成线性产物;本 skill 是积累卡片网络。可串接:线性产物输出后转 ingest |

## 不做的事(防边界滑坡)

- **不自动发文**:就绪簇只发 Telegram 建议,等用户 `/zettel write`
- **不改你的写作风格规则**:重述时直接读 `references/voice-snapshot.md`(你预先填好的价值观/文风快照),不在本 skill 复制业务文风
- **不当 GetNote 镜像**:literature note 引用 raw/ 路径,不复制原文
- **不解决"该写什么"**:本 skill 帮你在既定方向上更高效组织,不替你决定方向(这是诚实的边界)
- **本地嵌入做语义召回(v1.1.0)**:BGE-small-zh-v1.5 via `fastembed`,~90MB 缓存在 `~/.cache/fastembed/`(不进 git);TF-IDF 作为降级路径。最终是否真连仍由 agent 判断

## 失败时

| 现象 | 排查 |
|---|---|
| ingest 找不到判断,只能列举 | 素材本身是清单/事实集,不适合做 permanent note;考虑只入 fleeting/ |
| 链接候选全是低相关度 | 检查 `_index/` 是否过期,跑 `scripts/build_index.py --rebuild` |
| scan 反复处理同一文件 | 检查 `_state/last_scan.json` 的 mtime 是否被 git 重置;`processed/<date>.jsonl` 是否漏写 |
| 簇就绪却没收到 Telegram | 检查 `_state/cluster_history.json` 是否已把该簇标过(去重逻辑) |
| 价值观重述变成换词游戏 | agent 重读 `references/voice-alignment.md`,该协议明确要求产生新判断;否则不算 voice_passed |

## 版本

- **v1.4.0 (2026-05-20)** — **Self-contained 协议**:正文必须含 `## 原文要点` 块(blockquote 完整搬运 200-500 字原文,不是 30 字摘要)+ `## 我的判断` 节。**图片搬运**:原素材含 `![](../images/...)` 时,`scripts/migrate_images.py` 复制图到 `~/zettel/assets/<card-id>/`,卡正文改用新路径引用。脱离 source.ref 仍可读为硬测试。`scan_mechanical.py` queue item 增加 `source_images` 字段。旧 raw-source 卡(v1.3 及之前)归档到 `cards/_archived_v13/`,raw 重新入队 Oven 按新协议重做。
- **v1.13.0 (2026-05-20)** — **wiki-article 多卡切分**。用户发现 v1.12 时 82 篇文章只产 83 张卡(平均 1.01),信息压缩过严。新协议:长文必须通读 + 找全 atomic 判断 + 按字数对应产卡数(短 2-3 / 中长 3-6 / 长 6-10 / 巨长 10-15)。1500+ 字的文章只产 1 张需在 processed jsonl 写明理由。规则见 `references/voice-alignment.md` Step B.2。已切 1 张的 82 篇暂不强制重做,bootstrap 跑完后由用户决定是否回填。
- **v1.16.1 (2026-05-21)** — **Deep-research 后台补漏 worker**。新 cron `zettel-deep-research-worker`(disabled 默认,手动触发或 cron schedule)。流程:读 `_state/deep_research_pending.json` 下一个 pending → 跑 `/deep-research` → 存 `~/zettel/raw/deep_research/<proposal_id>__<gap_type>/` → Mode 1 ingest 切多卡(`source.type=deep-research`)→ 更新 `top3_outlines.json` 的 `deep_research_status` → render HTML → push → Telegram。触发词 "/zettel deepresearch" / "补一个 gap" 等。
- **v1.16.0 (2026-05-21)** — **图文 TOP3 + GitHub Pages 部署**。`render_top3_html.py` 生成图文 HTML(SVG 关系图 + LLM outline + gap badges + 推荐动作 + GitHub blob 卡片链接)。Pages: https://your-github-user.github.io/zettel/。Oven 写 `top3_outlines.json` + `deep_research_pending.json`,Python 渲染 HTML。
- **v1.15.0 (2026-05-21)** — **Multi-view cluster detection**。详上。
- **v1.12.0 (2026-05-20)** — **Mode 5 命题作文 + 证据图优先**。
  - **Mode 5 propose**:用户给 seed(案例/论点/问题),Python 召回 top-30 候选卡 → agent 读卡判定相关性+分角色(support/counter/case/context)→ 写 `_drafts/proposal_<ts>.md` → 用户审视后 `/zettel write` 移交写作 Skill。自顶向下"命题作文",跟 Mode 4 (自下而上从就绪簇出文) 互补。
  - **证据图协议**:wiki-article 源默认 `migrate_images.py --skip-first --include-remote`。第一张图通常是题图(封面/装饰),跳;evidence 图(idx 1+)是 R2 远程 URL,卡正文直接用 URL,不复制本地。选图标准:证据图(截图/数据图/对比图)优先,题图不要。
- **v1.11.0 (2026-05-20)** — **冷启动来源门禁 + Obsidian 归档隐藏 + YAML 双引号修复**。bootstrap 阶段 source.type 只接受 wiki-getnote / wiki-article(其他源走手动 Mode 1 ingest)。归档 5 张 chat-source 历史卡到 `cards/_archived_bootstrap_chat_v1_11/`(它们是 zettel-builder 自身做 smoke test 时的 demo 卡,不符合冷启动协议)。`.obsidian/app.json` 改用 prefix `cards/_archived` 一键隐藏所有归档目录,Obsidian 不再展示 broken / archived 卡。修复 1504 卡的双引号嵌套 YAML(单引号外包)。
- **v1.10.0 (2026-05-20)** — **时效性过滤 + 穿越时间原理提取**。判定 source.created_at 距今是否 > 12 个月。老素材只切原理/规则/方法论层判断,**不切**绑定具体型号/工具/价格/功能的判断。已加抽象化技巧(把"GPT-4 写代码"提到"高能力模型扩展开发者产出方式"层)。完整规则在 `references/voice-alignment.md` Step B.4。AI 迭代极快,这条防止 zettel 网络被"几年前如此"的死信息稀释。
- **v1.9.2 (2026-05-20)** — **回退到 id = filename stem 单一形式**(用户反馈 v1.9.0/v1.9.1 alias 解析机制不稳 + 加认知负担)。aliases 字段删除。wikilink 直接用完整 filename stem,Obsidian 原生解析。`references/card-naming.md` 完全重写。
- v1.9.1 (2026-05-20) — 短 id 也作为 alias 修复 Obsidian dangling link。**已被 v1.9.2 取代**。
- v1.9.0 (2026-05-20) — 试图 filename / id 解耦,引入短 id + aliases。**已被 v1.9.2 取代**(过度设计)。
- **v1.8.0 (2026-05-20)** — **session-review 过滤器**。session-review 类素材(tag/文件名/结构判定)进卡时,**只切用户层**判断(需求/价值观/偏好/纠正/边界),**不切技术层**(问题描述/解决方案/工具细节/Codex 修复)。规则在 `references/voice-alignment.md` Step B.5。同步加 cleanup cron `zettel-tech-cleanup-once`,Oven 巡现有 cards/ 归档技术细节卡到 `cards/_archived_technical_v18/`。
- **v1.7.0 (2026-05-20)** — **原文时间戳**。frontmatter `source.created_at` 必填(chat 可缺);正文 `## 原文要点` 节首行 italic 标注 `*原文时间:<ts>*`。来源:wiki-getnote 读其 frontmatter `created_at`,wiki-article 查 `raw/metadata/article-dates.json`。`backfill_source_timestamp.py` 已对所有现有 54 张非 chat 卡补齐(20 wiki-article + 34 wiki-getnote)。
- **v1.6.0 (2026-05-20)** — **Agent-judged 链接 + Serendipity 涌现机制**。变更:
  - dedup/link 判断改为 agent 读全文,**不再**信余弦自动跳过。召回扩大到 top-20 近邻 + top-5 远距(sim<0.5 随机),Oven 看完才决定。
  - `## 我的判断` 正文必须自然引述至少 1 张相关卡(言之有物的 `[[]]` 内嵌),不只是 `## 相关卡片` 节列条目。validator 校验。
  - **新 cron job `zettel-serendipity-daily`**:每天 3 次(08:00 / 14:00 / 20:00)随机抽 20 张互不相连的卡,Oven 读全文找潜在关联,命中即建立 link(`serendipity` tag),reason 说明跨域桥梁。
  - **Serendipity → cluster 涌现联动**:每次 serendipity 跑完立刻调 `cluster_inspect`。新就绪簇(尤其含 serendipity 链接的)立即 Telegram 高优先级推送"⚡ 涌现:一个新主题快速饱和"。
- **v1.5.0 (2026-05-20)** — **库存全量消化模式**(替代之前的"标记 already-scanned + 仅处理增量"误用)。变更:
  - `scan_mechanical.py` 现在默认入队 **raw/getnote + raw/articles** 全部(241 已发文章不再 skip;已发文章里的判断也是宝贵原料,经去重后回流网络)
  - **三档语义去重**(协议见 Mode 1 Step 5):写盘前嵌入比对,max_sim ≥0.85 跳过,0.7-0.85 强制 link,<0.7 新建。一篇 source 可对应零张或多张卡
  - **bootstrap_runner.sh 守护进程**:持续从 queue 取批喂 Oven,不再受 cron 间隔限速;遇 quota/rate-limit 用指数退避(2/4/8/16/32 min);queue 持续 1 hour 空后自动切 steady(disable bootstrap cron + enable event-driven + 更新 mode.json),发 Telegram"bootstrap 完成"
- **v1.4.0 (2026-05-20)** — Self-contained 协议 + 图片搬运 + voice_passed validator(详上)
- **v1.3.0 (2026-05-20)** — 双态调度(bootstrap → steady)、published-articles 嵌入去重、event-driven trigger。
  - **双态**:`_state/mode.json` 记录当前态。bootstrap N=20,steady N=5。`zettel-bootstrap-hourly` cron 每小时跑直到 queue 排空;手动 `openclaw cron disable <id>` 切 steady。
  - **published-overlap**:`embed_lite.py --include-published` 把 `~/Dropbox/cn_articles_published/all/*.md` 也嵌入。`cluster_inspect.py` 对每个就绪簇算 centroid → cosine match → top-3 候选 + `overlap_class`。Mode 3 LLM 命题时**必读** overlap,Telegram 摘要用红/橙/绿三色标记。
  - **event-driven**:`zettel-hourly-sync.sh` Step 6:steady 态下,wiki 出现新内容(`.new-content-trigger`)时立即 `openclaw cron run <event-driven-scan-id>`,无需等下一轮 cron。
- v1.2.0 (2026-05-20) — Mode 1/2/4 写卡后显式 `git add + commit + push`(不靠 systemd 兜底)。新增 OpenClaw cron 编排:Oven (workspace-openai, GPT-5.5) 接管 daily scan + weekly inspect,Telegram 通过 @your-openclaw-bot 原生发送。
- v1.1.0 (2026-05-20) — 链接候选改 BGE-small-zh-v1.5 嵌入(语义,主)+ tag/entity/链邻居结构信号(辅);TF-IDF 降级路径保留。mechanical scan 不再预算 candidate_links,该计算移入 agent 路径。`scripts/inspect.py` → `cluster_inspect.py`(避免 stdlib 撞名)。
- v1.0.0 (2026-05-20) — 初版。TF-IDF + 4 因子打分。

---
> Source: [wshuyi/zettel-builder](https://github.com/wshuyi/zettel-builder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
