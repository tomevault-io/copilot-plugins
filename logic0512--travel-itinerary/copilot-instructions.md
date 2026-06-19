## travel-itinerary

> Travel assistant for generating source-checked, visually polished travel guides. Handles three user states: no clear idea yet, partial plan needing completion, and fixed places needing itinerary cleanup. Includes path triage, artifact intake, active research, fact ledger, image asset collection, geography/time validation, destination-specific visual theming, a required HTML/CSS print source, browser-exported editorial PDF output, a trip-wide constraints layer (budget tier, companion structure, theme/pilgrimage), multi-city and cross-border skeletons, weather planning, and solo-traveler safety. Use when the user asks for travel ideas, travel guides, itinerary planning, trip route cleanup, or a good-looking file to carry while traveling.


# 旅游攻略 Skill · travel-itinerary

## 一、触发条件

用户提出旅行相关任务时触发，包括：

- 「不知道去哪，帮我推荐/做攻略」
- 「想去 X，但还没定路线/天数/酒店」
- 「这些地点我想去，帮我排成行程」
- 「我已有每天大概安排，帮我整理成能旅行时看的文件」

**触发后第一件事**：执行 Stage 0.0 情况评估（见下方），确定走哪条流程路径。

## 二、流程路径（四界面管道 + 三条成熟度分支）

**核心方案（v3.0）：围绕「地图 + 点位」做规划布局，再一键导出渲染成攻略。问卷表格方案（Q1/Q2）已彻底退役，归档在 `assets/_deprecated/`，不再使用。**

四个界面顺序串联，形成「发现 → 规划 → 补充 → 出图」的数据管道：

```
discovery     planner             supplement        trip-print-v2
发现选点   →  地图排点位+分天  →  勾选备选       →  8模板渲染出图
（勾必去/  →  （Leaflet拖拽   →  （备选景点/    →  （导出 PDF）
 感兴趣）      排序+锁定）         美食/特产）
```

**多城（≥2 城）走同一条管道，不另造分支**：开局先在 Stage 0 定下「城市顺序 + 每城天数 + 城际交通段」骨架（`meta.city_plan`，见第六章）。planner 地图按当前天的点位自动 fit，按天切换自然聚焦当天的城——所以**多城用单个 planner 文件即可，不拆文件、不分图、不分次跑**。跨城游产出粒度就是「天」：每天归属它所在的城，城际那天出 T2 转场页，由现有「按天出页」原生覆盖。

### 路径决定走哪几个界面（由 checklist 9.0 路径分诊门判定主路径）

| 路径 | 触发 | 走哪几个界面 |
|---|---|---|
| **A · Full** | 没想法 / 认知低 | discovery → planner → supplement → trip-print-v2（全程） |
| **B · Constraint** | 有部分计划 | 跳过 discovery；planner 直接载入「用户已有点 + Claude 补研究的点」→ supplement → trip-print-v2 |
| **C · Express** | 行程已定 | 跳过 discovery + supplement；planner 仅核实/排序确认 → trip-print-v2 |

分段成熟度仍生效（见 9.0.2）：`locked` 段只核实、`open` 段主动研究——路径只决定界面起点，研究力度按段走。

### 三个数据交接断裂点（铁律 · 必须 Claude 手工转换）

界面间用 JSON 导出/导入衔接，但有三处对不齐，Claude 必须补：

- **断裂 A（discovery → planner）**：discovery 导出的选择列表**无坐标**。进 planner 前，Claude 必须为每个点补 `coords:[lat,lng]`（web_search 地图/高德/OSM 查），注入 planner 的 PLACES 池（注入时按实际给 planner type，方言对照见 `references/type-vocabulary.md`）。**没坐标 planner 地图无法标点。**
- **断裂 B（supplement 独立）**：supplement 与 planner 行程**完全解耦**，只导出补充项的 id+name。最终在构建 TRIP 时**并行合并**进 `extras`，不从 planner 拼接；**合并前按 POI id/name 去重——已进 planner timeline 的点不再重复进 extras**（见 checklist I-1③）。
- **断裂 C（planner+supplement → trip-print-v2）**：两者导出 ≠ TRIP 结构，**不能直接拼**。先按 `references/type-vocabulary.md`（链路通用语言总表）做 type 归一化+细化再组装。Claude 必须：① **type 归一化**——planner/discovery 方言全部翻成标准 type，`spot/experience` 粗分按实际细化（自然=sight_nature·寺庙=sight_cultural·徒步=experience_outdoor）；② **合成转场段**——跨城/跨岛那天造 `{type:'transit',intent:'intercity',from,to,coords,duration}`（planner 不产出转场段，漏了选不出 T2）；**城际段对账**：按 `meta.city_plan.intercity` 逐段核对——骨架里有 N 段城际交通，最终 timeline 就必须有 N 条 `{type:'transit',intent:'intercity'}`，逐段比对 from/to，缺一段即不通过（防三城两段漏造中间一段）；段数 + from/to 对账之外，**还要过一遍距离常识**——反常识超长段（跨大区/跨气候带，如"吉林→腾冲 3000km"）回 9.0.0 事实预检消歧，不静默通过；③ **标徒步段**——整天/半天徒步主段标 `experience_outdoor`+必填 `duration`，视情填 `on_trail_food`（否则选不出 T3）；③.5 **自驾沿途景观点不丢**（病灶⑮）——自驾 / 自驾环线转场日，沿途经过的景观点（如五彩滩、魔鬼城）**必须造成正常的 `sight` / `sight_nature` 点位**，**且整体排在该日 timeline 的 transit 段之后**（先把当天所有 intercity transit 段连排，再排沿途点 + 落地活动），**绝不把景观点夹在多段 transit 之间**。原因：选页器只提取「末段跨城 transit 之后」的落地点（`landing = timeline.slice(lastCrossIdx+1)`），夹在多段 transit 中间的景观点会被 slice **彻底切掉、静默消失**；排在 transit 段之后才保得住。**已知代价（L1 · 校准措辞）**：当沿途点 ≥3 个时，它们会被选页器**独立渲染成到达城的「城内游」页、地图归到达城名下**，可能与点的实际位置有偏差（不只是「被当成到达点的活动」，偏差比这更明显）；**点不丢**（满足止血目标），但**归属语义有偏**——这是 L1 已知且接受的妥协（不丢内容优先于语义精确），根治留待 L2。④ **回填 maturity**——每个 day 按 9.0.2 标（A 路径缺省 open·B/C 缺省 partial）；⑤ `planner.days[].places[]` 转成 `TRIP.days[].timeline[]`，补 `meta`（日期/人数/预算/交通/酒店）、`global`（紧急/清单/票据）、`days[].meals`/`reminders`/`weekday`/`theme`；⑥ 丰富每个 card 的 `emoji/intro/ph/price`（intro 2-4 段，按 cards schema）；⑦ 跑 `selectPagesForDay` 拆页，**验收：点位无消失·徒步日出 T3·转场日出 T2·每天 maturity 已落·`meta.themes[].anchors` 每个（已过 9.0.0④ 核查、且在计划城市内的）点都在某天 timeline 找得到对应 card（anchor 保入对账，防主题点被稀释丢失，见 checklist 9.0.5⑥②）**；⑧ **约束统一出口**——`meta.constraints` **全部由「出行须知」区渲染**（含 `scope` 为 region slug / `'day:N'` 的约束，渲染层自动在文字前标注适用范围：`day:N`→`【第N天】`、region→`【region名】`，无中文名映射时降级显示 slug 原文）。region/day 级约束**可选**额外落进对应 `day.reminders` 作当天上下文增强，但「出行须知」区才是保证不丢的**唯一主出口**——别再依赖各页型渲染 reminders 来承载约束。**跨境行程额外**：把 `meta.city_plan.entry_exit.border_notes` 每条转写成 `meta.constraints`（`type:'transport'`、`scope:'trip'`）再走上述出口，`inbound/outbound` 的口岸→市区交通落进抵达/离境日 T2；`entry_exit` 自身不出图（病灶⑫：填了不转写=不出图=白填）。**主题行程额外**：把 `meta.themes[]` 每条转写成 `meta.constraints`（`type:'theme'`、scope 沿用，text 点明「本趟贯穿 X 主题、已优先安排取景/朝圣点」）再走上述「出行须知」出口；themes 自身不出图，只填不转写=用户看不到主题=病灶⑬没修（与 `border_notes` 同一转写模式）。⑨ **跨夜长途转场**（夜船/红眼/夜卧跨午夜）按 9.0.6 分支②单独造一个 `day`（timeline = 该 transit 段 ＋ 一个 `type:'free'` 锚点；纯 transit 无 free 的日会被 `selectPagesForDay` 判成 T8 休整页、出不了 T2），让 `selectPagesForDay` 出 T2，到达城首日从次日另起。

### 文档职责边界（维护规则）

- `SKILL.md` 只放触发条件、路径选择、资产导航和最高优先级原则。
- `references/v2.4-checklist.md` 虽保留旧文件名，但内容按当前版本维护；所有执行细则、自检项、事实核查规则以它为准。
- `references/fact_ledger.template.md` 是事实台账模板；新 case 优先复制它，不要每次重新发明格式。
- `references/asset_ledger.template.md` 是图片资产台账模板；PDF 配图必须用它追踪来源和状态。
- `references/pdf-output-rules.md` 是最终画册交付规则；它规定 HTML/CSS 是唯一视觉排版源，PDF 必须由浏览器从 HTML 导出。
- `references/design-rules.md` 只管视觉方向、色板和 CSS 变量映射。
- 若同一条执行规则在 `SKILL.md` 和 checklist 同时出现，以 checklist 为准，并在下次维护时删掉 `SKILL.md` 中的重复细节。

### Stage 0.A · 输入物识别（用户给了文件/截图时必做）

如果用户提供图片、PDF、地图、备忘录截图、票据截图、聊天记录或散乱文档，先判断输入物类型，再选流程。不要默认 PDF 一定有文字，也不要默认截图只是参考图。

| 输入物 | 典型信息 | 默认路径 | 处理方式 |
|---|---|---|---|
| 手机备忘录/便签截图 | Day、食物/地点清单、少量顺序 | B · Constraint Flow | 先 OCR/视觉抽取成 `raw_user_intent`，再补店铺实体、时间、地址、交通 |
| 手绘路线图 PDF/图片 | Day、路线箭头、时间、交通方式、票据截图、待确认点 | C-lite · Express Rebuild | 保留用户路线框架，但逐点核实并重建结构化 timeline |
| 地图收藏/POI 清单 | 大量地点、少量优先级 | B · Constraint Flow | 先分类、去重、按片区聚类，再让用户确认必去/可选 |
| 票据/订单截图 | 航班、火车、酒店、门票、租车 | B/C 锁定项 | 作为 `fixed_constraints`，先提取时间和地点，再核实衔接 |
| 纯文字攻略/聊天记录 | 候选方案、朋友建议、碎片提醒 | B · Constraint Flow | 拆成 confirmed / maybe / needs_recheck 三类 |

**输入物处理原则**：

1. 先输出一段「我从文件里识别到的已定信息 / 待确认信息 / 风险点」，让用户知道 skill 理解对了。
2. 手写图、截图、图片型 PDF 不能只靠 `pdftotext`；若抽不到文字，走视觉识别/人工转录摘要。
3. 用户手绘路线图通常代表「路线框架已定」，不要推翻重做；应该做事实核查、交通校正、时间缓冲和视觉化交付。
4. 手机备忘录截图通常代表「偏好和半成品清单」，不要直接排满 timeline；按 checklist **I-1 已有偏好流转**走：消歧（口语条目→真实 POI，一对多回钩用户确认）→ 预填（已点名的预勾选 ★必去）→ 去重。
5. 所有从图片/PDF 识别出的内容先标记为 `source=user_artifact`，进入 fact ledger 后再升级置信度。

### Stage 0.0 · 情况评估（必做，用 5 项判断分支）

先用用户已给信息做判断，不足时最多问 3 个关键问题。不要一上来把整套四界面流程或一堆表单抛给用户。

判断 6 项：

| 项 | 判断 |
|---|---|
| destination | 目的地是否确定？若没有，进入 A0 目的地发现 |
| **experience_knowledge** | **用户对目的地的认知深度——「知道要去哪」≠「知道去那里该怎么玩」** |
| dates | 日期/天数是否确定？若没有，只能先做方向和候选，不锁死逐日排程 |
| fixed_constraints | 机票/酒店/门票/城市顺序等不可改约束有多少 |
| must_places | 用户明确必去地点/体验有多少 |
| route_maturity | 空白 / 有核心但未成行 / 每天框架已定 |

**多城开局（≥2 城）必做城际骨架**：评估完上表后，若目的地为 2 城及以上，Stage 0 还要多产一步——固化 ① 城市先后顺序 ② 每城待几天 ③ 城市之间的城际交通段，写进 `meta.city_plan`（结构见第六章 Schema）。一天只在一个城活动，城际移动是「某一天」（转场日）的事；多城用单个 planner 文件，按天切换自然聚焦当天的城，**不拆文件、不分图、不分次跑**。

根据回答进入对应路径：

```
                  ┌─── A · Full Flow（从零开始 OR 对目的地缺乏认知）
                  │    destination 未定
                  │    OR experience_knowledge 低（虽然知道去哪，但不知道该玩什么）
用户说"做攻略" →─┼─── B · Constraint Flow（有部分计划）
                  │    destination 定 + experience_knowledge 中等 + 有固定约束或必去点
                  └─── C · Express Flow（地点/每日框架已定）
                       destination 定 + experience_knowledge 高 + 每天框架基本成型
```

**experience_knowledge 判断信号**（仅作快速直觉参考；与 9.0 占比法冲突时，一律以 checklist 9.0.1 占比法为准）：

| 信号 | 判断 |
|---|---|
| 「没去过」「不了解」「你帮我定」「随便」「不知道有什么好玩的」 | 低 → 走 A |
| 只给了目的地 + 天数，没有任何景点/体验偏好 | 低 → 走 A |
| 给了 1-3 个笼统方向（「想吃好吃的」「想看古建」），但没有具体点 | 低偏中 → 走 A，研究后让用户确认方向 |
| 给了几个具体必去点，但其余天空白 | 中 → 走 B |
| 给了每天大致安排 | 高 → 走 C |

**路径选择防错规则**：

- **A 路径的核心价值是「主动研究 + 带用户发现」，不是「问完 3 个问题就排行程」**。只要 experience_knowledge 低，不管用户说了多少词，都应该先给用户看研究成果再推进，而不是让用户自己填。
- destination 未定，不要进入 B/C；先做 A0 目的地发现。
- 用户只给了目的地+天数（如「我要去东京 7 天」），不能判断成 B——这是 experience_knowledge 低的典型信号，走 A。
- 用户给了 5 个以上地点但没有日期/城市顺序，通常是 B，不是 C。
- 用户给了每天安排、酒店、交通票，通常是 C；但仍必须逐点核实。
- 用户给了手绘路线图/行程地图 PDF，通常是 C-lite：保留大框架，但不直接相信时间、交通、营业状态。
- 用户给了手机备忘录截图，通常是 B：它表达偏好，不等于可执行行程。
- 用户说「随便帮我安排」**且没有**已订票/酒店约束，走 A（experience_knowledge 低的典型表达）。
- 判断不确定时，先输出一句：`我先按 X 路径做，原因是...；如果你其实已经锁定每天安排，我会切到 C。`

**★ 强制门（v2.9）**：完成上述判断后，进入任何 Stage 0 之前，**必须先输出 `references/v2.4-checklist.md` 9.0.1 的【路径分诊】声明**，并按 9.0.2 标出分段成熟度（locked/partial/open）、过 9.0.3 防误判 3 问。未输出分诊声明就开始排程或研究 = 流程违规。一趟行程的成熟度按段分配研究力度：`locked` 段只核实、`open` 段必须主动调研——主路径只决定**走哪几个界面**（A 从 discovery 起 / B 跳 discovery / C 跳 discovery+supplement），不代表整趟用同一种力度。**离开 Stage 0 triage、进研究/排点前，还要过 9.0.7 整趟可行性摊牌门**（把地理×天数 / 预算×诉求 / 节奏×数量 / 关键岔路四轴摆一桌，命中冲突列出来 + 给 2 个取舍方案回钩用户裁决，不闷头硬排、不静默砍诉求）。**涉及自驾 / 跨境 / 特殊活动（潜水 / 入山徒步等）时，必过 9.0.8 出行资格准入门**（查"人有没有资格用这种方式出行"——自驾查驾照承认 / 认证翻译件、跨境查签证及免签有效期 / 护照 6 个月、特殊活动查执照许可，命中门槛提前告知 + 给补办路径或替代方案，结果落 `meta.constraints`）。

### Stage 0.1 · 事实可信度总规则（所有路径必做）

1. 复制 `references/fact_ledger.template.md` 建立 `fact_ledger.md`，或在分析报告中加入同格式「事实核查表」：每条关键事实记录 `事实 / 来源 / 查询日期 / 置信度 / 对行程影响`。
2. 关键事实包括：营业时间、定休日、预约规则、票价、交通班次/末班、跨城耗时、地址坐标、关闭/搬迁状态。
3. 来源冲突时，不要硬编结论。按优先级处理：官网/官方地图 > 官方社媒 > 大型平台 > 博客攻略；给出采用理由。
4. 无法确认的事实必须标为 `needs_recheck`，并把它从不可改安排降级为备选或提醒，不得写成确定结论。
5. 用户提供的信息必须二次核实；发现不可行要直接指出，并给 1-2 个替代方案。
6. **海外信息不好找时，优先用 YouTube 搜实拍视频**——图文攻略稀少/语言不通的海外目的地，YouTube 上的实拍 vlog 往往是最可靠来源。**尤其海外徒步**：YouTube 视频常含完整徒步路线、实拍路况、爬升/耗时、以及**线路上有没有补给/进食点**——这些正是渲染 `on_trail_food` 字段所需（线路有吃的→徒步页内含餐饮；没有→正餐放配套下一页）。查询用英文关键词 + `trail` / `hike` / `route` / `full walkthrough`。

---

### A · Full Flow（从零开始 / 对目的地缺乏认知）

```
Stage A0 · Idea Discovery
           触发条件：destination 未定 OR experience_knowledge 低
           ——————————————————————————————————
           [目的地未定时] 先输出 2-4 个目的地/路线方向，
             每个方向含：适合人群、季节、预算区间、核心体验、风险、推荐天数
             让用户选定方向后再进入下一步
           ——————————————————————————————————
           [目的地已定但 experience_knowledge 低时]
             ★ 这是 A 路径的核心价值，不要跳过 ★
             主动做目的地研究，输出：
             · 目的地核心特色（这个地方独有的、值得专程去的体验是什么）
             · 片区/城市结构梳理（哪几个区值得花时间，各有什么侧重）
             · 2-4 个旅行类型方向（文化沉浸 / 美食扫街 / 自然徒步 / 都市体验…）
               每个方向含代表行程点和适合人群
             · 季节/时间提醒、必提前预约的项目
             让用户确认偏好方向 + 基本诉求后，再进入 Stage 0
           ——————————————————————————————————
           禁止：问完 2-3 个问题就开始排行程——这不是「一起做攻略」，
             这是让用户在没有信息的情况下做决定。
Stage 0  · Intake + 批量 web_search（目的地/季节/交通/候选实体/餐饮）+ fact ledger
           ★ 多城骨架（≥2 城时必做，单城跳过）：开局先定 ① 城市先后顺序 ② 每城待几天
             ③ 城市之间的城际交通段（from/to/mode/约耗时），产出一张「城市分配表」写进
             `meta.city_plan`（结构见第六章 Schema）。全程当规划骨架，不是界面字段。
             多城仍用单个 planner 文件，按天聚焦当前城，不拆文件不分图。
Stage 1  · ① discovery 发现手册（注入研究出的景点/美食/体验候选 → 用户勾选必去/感兴趣）
★ Phase 1 终点：分析报告（路线/住宿区/机票时段/预算）→ 用户订机票+酒店
Stage 2  · 断裂 A：Claude 为 discovery 选出的点补坐标
           → ② planner 地图规划（用户拖拽排点位 + 分天 + 锁定）
           → ③ supplement 补充确认（勾选备选景点/美食/特产）
Stage 3  · 断裂 C：Claude 合并 planner+supplement 导出 → 组装 TRIP 结构
           → 读 pdf-output-rules.md + design-rules.md + 收集 image assets
           → ④ trip-print-v2 渲染（8 模板自动选） → 浏览器导出 trip-guide.pdf
★ Phase 2 终点：trip-guide.pdf + trip-print-v2 成品 html + fact_ledger.md + asset_ledger.md
```

适合：完全没有计划，或知道目的地但不了解当地特色，需要 skill 主动做调研 + 带用户发现 + 决策 + 排程。

---

### B · Constraint Flow（有部分计划）

```
Stage 0  · 快速约束收集（对话）：哪些固定？哪些开放？核心点有哪些？（跳过 discovery）
Stage 0b · 针对「开放」部分批量 web_search（补充研究）
           + 验证所有「固定」核心点的可行性（时间/开放状态/交通衔接）+ fact ledger
Stage 1.5· 矛盾消化对话（发现问题直接指出，给替代方案）
★ Phase 1 终点：约束清单 + 分析报告（只锁定未定部分）
Stage 2  · 断裂 A：Claude 为「用户已有点 + 补研究的点」补坐标
           → ② planner 地图规划（已锁定项标 locked 只核实、开放点用户拖拽编排）
           → ③ supplement 补充确认
Stage 3  · 断裂 C：合并导出 → 组装 TRIP → ④ trip-print-v2 渲染 → 导出 PDF
```

适合：有几个必去地点和固定约束（已订机票、景点门票），需要 skill 研究填补空缺 + 核实固定计划的可行性。

---

### C · Express Flow（行程已定）

```
Stage 0  · 行程收集（对话 · 直接让用户列出每天的安排）（跳过 discovery + supplement）
Stage 0c · web_search 核实每个地点（营业时间/定休日/坐标/到达方式）
           [每个地点都必查，不接受用户信息不经核实；生成 fact ledger]
Stage 0d · 若来源是手绘路线图/地图 PDF，重建结构化 timeline：
           原图内容 → day / time / place / transport / locked? / note / needs_recheck
Stage 2  · 断裂 A：Claude 补坐标 → ② planner 仅作核实/排序确认（用户微调动线即可）
Stage 3  · 断裂 C：planner 导出 → 组装 TRIP → ④ trip-print-v2 渲染 → 导出 PDF
★ Phase 2 终点：trip-guide.pdf + trip-print-v2 成品 html + fact_ledger.md + asset_ledger.md
```

适合：行程已基本定好，只需要把散乱的计划整合成一份好用的可交互行程单，并补充每个地点的实用信息。

## 三、资产

```
travel-itinerary/
├── SKILL.md（本文件）
├── assets/                          · 四界面管道（按数据流顺序）
│   ├── discovery.template.html      · ① 发现手册（注入候选 → 用户勾选必去/感兴趣）
│   ├── planner.template.html        · ② 行程规划（Leaflet 地图 + 拖拽排点位 + 分天）
│   ├── supplement.template.html     · ③ 补充确认（勾选备选景点/美食/特产）
│   ├── trip-print-v2.template.html  · ④ 最终渲染（8 模板系统 · Stage 3 出图源）
│   ├── theme.template.css           · 视觉骨架（结构层·不直接改）
│   └── _deprecated/                 · 已退役（Q1/Q2 问卷 + build_q2 + 旧 trip-print/trip 渲染模板，仅供参考字段设计）
└── references/
    ├── v2.4-checklist.md          · ⭐ 作业清单（含 9.0 路径分诊门，旧文件名保留）
    ├── page-template-system.md    · ⭐ 8 模板系统设计规格（图片/地图形状铁律 + 选模板逻辑）
    ├── stage4-selector.js         · 选模板逻辑（typeToCategory / selectPagesForDay，断裂 C 用）
    ├── fact_ledger.template.md    · 事实台账模板
    ├── asset_ledger.template.md   · 图片资产台账模板
    ├── pdf-output-rules.md        · HTML-first / browser-export PDF 最终交付规则
    └── design-rules.md            · 目的地视觉规则（Stage 3 生成 theme.css 的依据）
```

**新 case 启动顺序**：
1. 执行 **Stage 0.0 情况评估**，确定 A/B/C 路径
2. 读 [references/v2.4-checklist.md](references/v2.4-checklist.md)
3. 按对应路径动手

**依赖说明（v2.5 起）**：
- **模型**：不指定，无硬绑定。
- **外部 skill**：无。主题生成用内置 `references/design-rules.md`，无需调用 `frontend-design` 或任何设计 skill。
- **所需工具**：`web_search`（信息核实与图片来源）+ `Read/Write/Edit/Bash`（构建与自检）。如果环境支持浏览器截图/PDF 渲染，Stage 3 必须逐页截图验收。

**资产文件（本地）**：
```
assets/theme.template.css     → 基础骨架（结构层，不改）
references/design-rules.md    → 目的地配色规则（皮肤层，Stage 3 读这里生成 theme.css）
```

### 四界面数据注入说明（v3.0）

每个界面模板按 `{{占位符}}` / inline 数据块注入后生成成品。注入与导出契约：

| 界面 | 注入（Claude 填） | 用户操作 | 导出 |
|---|---|---|---|
| ① discovery | 目的地 + 城市列表 + 景点/美食/体验候选（研究出的） | 三态标记 ★必去/♡感兴趣/—跳过 | JSON：`{must_visit:[{id,name,type,city}], interested:[...]}`（**无坐标**） |
| ② planner | PLACES 池 `{id:{name,type,region,coords:[lat,lng],desc}}`（**坐标必填**）+ dayPlans | 地图拖拽排点位、分天、锁定 | JSON：`{days:[{day,date,region,places:[{id,name,type,order,coords}]}]}` |
| ③ supplement | 备选景点/美食/特产卡片库 | 勾选要加入攻略的 | JSON：`{supplement_selections:{backup_sights:[{id,name}],...}}` |
| ④ trip-print-v2 | 完整 TRIP 对象（见第六章 Schema） | —（最终成品） | 浏览器导出 PDF |

**注入铁律**：
- planner 的每个点位**必须带 coords**——这是地图标点的命脉，断裂 A 的 Claude 补坐标动作不可省。
- ④ 的 TRIP 对象不是任何单一界面的导出，是 Claude 按断裂 C 规则合并 planner+supplement+新增字段拼出来的。
- 三个界面都内联数据、双击即用，不依赖服务器（遵守 W-3）。

> 退役方案的字段设计（Q2 四维过滤 / build_q2 守卫校验 / 餐饮密度字段）仍有参考价值，需要时翻 `assets/_deprecated/`。

## 四、v2.3 核心功能

### 三大主线
1. **链接策略重写** — 国内：高德 POI 优先 → 官网 → 携程 → 黑珍珠等评级页，大众点评/美团降为最末 fallback（标"需登录"）
2. **过滤升四维** — 时间轴下拉从 `(type, intent)` 升为 `(type, intent, region)`，跨片区可切换
3. **餐饮密度强制** — 新增 `daily_meal_pool` 按片区组织便餐；Phase 1 前密度检查：medium ≤20% 占位

### 新增功能
- **天气感知** — 出发前 7 天检测；雨天态度偏好（Stage 0 询问）；planner 标室外点；成品页天气栏 + 室外标签 + 室内备选
- **solo_traveler 模式** — 人数=1 时展开独行偏好 + 成品页夜间安全提醒
- **预算实时追踪** — 成品页速查卡，LocalStorage 记录每日支出，余额变红提醒
- **entity_type 四态** — storefront / brand / aggregate / changed_or_gone
- **HTML-first PDF 交付** — 先生成 `trip-print-v2` 成品作为唯一视觉源，再用浏览器导出 `trip-guide.pdf`
- **强制配图** — PDF 中出现的非 transit 行程点必须有 image asset，并记录到 `asset_ledger.md`
- **界面一键导出** — discovery/planner/supplement 各自导出 JSON，Claude 合并成 TRIP 喂给 trip-print-v2

## 四点五、质量优先级（v2.6）

当时间不足或用户要求快速产出时，按以下顺序保质量：

1. **事实准确 > 行程完整**：不确定的信息宁可标注和降级，也不要写成确定。
2. **地理顺路 > 网红丰富**：每天尽量围绕同一片区或清晰转场，不堆点。
3. **用户约束 > AI 推荐**：已订票/已订酒店先保，AI 推荐只填开放部分。
4. **HTML 版式源稳定 > 直接 PDF 生成**：主交付仍是 PDF，但必须先做可目检、可截图、可修补的 HTML/CSS 画册源。
5. **目的地气质 > 固定皮肤**：每个目的地都要生成一句 visual brief，再做 theme.css。

### Stage 3 输出模式（v2.8）

默认输出分两层，但生成顺序不可颠倒：

1. `trip-print-v2.html`：唯一视觉排版源，8 模板按「当天行程形状」选页，图片先行，可直接用浏览器打开验收。
2. `trip-guide.pdf`：主成果，必须由浏览器从 HTML/CSS 打印导出，适合分享/携带/打印。

**禁止直接写 PDF**：不得用 ReportLab、FPDF、PDFKit 手工绘制最终 PDF；不得用渐变色块/占位图冒充图片资产。没有浏览器导出能力时，交付 HTML 源并说明 PDF 导出受阻，不要伪造低质量 PDF。

### Stage 3 视觉 brief（必做）

生成 `theme.css` 前，先写 visual_brief——格式和 destination_mood 选项见 **[references/design-rules.md](references/design-rules.md) 第二章**。写完 brief 再查对应色板，不要跳过直接套色。

## 五、Archetype 档位

| 档位 | 场景 | 说明 |
|---|---|---|
| `short` | 1–3 天 · 单城 | §0 LOCKS 默认折叠；备选截断 |
| `medium-single` | 4–6 天 · 单城 | 标准流程 |
| `medium-multi` | 4–7 天 · 跨城 | **片区锁定 + 跨城交通锁定 + 每城独立美食池**（见下方 medium-multi 特殊要求） |
| `long` | 7 天以上 | 完整流程；todo 5 阶段 |

### medium-multi 跨城行程特殊要求（v2.5 新增）

> **边界**：城市顺序 / region / 每城天数 以 9.0.6 的 `meta.city_plan` 为准，本节只管每城内部研究质量（美食池 / alt 候选 / 地理验证 / 转场日活动分配）。

**每进入一个新城市，视为独立的 sub-itinerary，必须分别做：**

1. **城市片区锁定** — 确定每个城市的主 region slug，day.region 必须与其一致
2. **独立美食研究** — 每个城市独立做 web_search，不能用另一个城市的餐厅顶数
3. **独立 alt_candidates** — 每城市备选只含本城市或当天可达范围内的地点
4. **Timeline 地理验证** — 写完每天 timeline 后，检查：所有 non-transit card 的 `card.region` = `day.region`；发现跨城混入立即纠正
5. **跨城转场当天** — 含转场的日子，到达城市的活动只放到达后可合理完成的内容；出发城市的活动只放在搭车前能完成的内容

## 六、Schema 关键字段（v2.3 新增）

| 字段 | 用途 |
|---|---|
| `meta.weather_tolerance` | 雨天态度（high/medium/low/flex） |
| `cards[].outdoor` | 是否室外景点（bool） |
| `cards[].indoor_alt` | 雨天室内替换（card_id 或字符串） |
| `days[].weather.*` | 天气数据（condition/temp_high/temp_low/rain_probability/advice） |
| `cards[].region` | 片区 slug（四维过滤必填） |
| `daily_meal_pool` | 日常便餐池（按片区 slug 组织） |
| `meta.route_path` | 主路径判定（'A'/'B'/'C'，见 9.0 路径分诊门） |
| `days[].maturity` | 分段成熟度（'locked'/'partial'/'open'），驱动研究力度与渲染，缺省 partial |

### 贯穿约束承载位（v3.x 新增）

整趟旅行的硬规矩（高原反应 / 独行安全 / 梅雨备雨 / 包车 vs 班车 / 预算档位）不是用户在界面填的字段，而是 Claude 全程把关的横切关注点。在 Stage 0 固化进 `meta`，全程作为筛选 lens，最终渲染进「出行须知」区。

| 字段 | 用途 |
|---|---|
| `meta.budget_tier` | 预算**主导**档位 `'backpacker'`/`'standard'`/`'comfort'`，驱动 Claude 选餐饮/住宿/交通的研究方向（省钱→平价/大众优先）。与展示字符串 `meta.budget` 分离。**病灶⑩**：拿到金额先厘清口径（全程总额/人均/某项专项·币种·含不含机酒）再判档，专项金额不并进档位判定；混合（一半青旅）/分项（化妆品专项5000）/弹性（必要时降档）等非单一档语义，落 `meta.constraints`（`type:'budget'`）承载。**用户全程没提预算则默认 `standard`、不强制追问**（标一句默认假设即可）。详见 checklist 9.0.5③ |
| `meta.budget_tracking` | 是否渲染预算追踪卡（布尔，独立字段，旧名 `meta.budget.enabled` 已废）。**⚠ 当前 trip-print-v2 无预算追踪卡消费者（`renderQuickCards` 是旧模板遗留），此字段暂 inert——L2 补卡后才生效，见 checklist S-5 注** |
| `meta.constraints[]` | 整趟约束清单。每条 `{type, text, scope}` |
| `meta.constraints[].type` | `health`🏥 / `safety`🌙 / `budget`💰 / `season`🌦 / `transport`🚐 / `family`👨‍👩‍👧 / `theme`🎬 / `other`📌 ——决定研究筛选方向与渲染分组。`family`=同行结构（带娃/适老/多代），驱动 open 段降强度筛选（候选偏亲子低强度、planner 降单日强度、缩跨城、长车程拆缓），详见 checklist 9.0.5⑤。`theme`=主题/朝圣**加分** lens 的渲染载体（由 `meta.themes[]` 转写而来、非用户原始输入；正向驱动逻辑在 `meta.themes`，见下行 + checklist 9.0.5⑥）。`family`/`theme` 已有**专属分组**（trip-print-v2 `CSTR_TYPE`+`CSTR_ORDER` 已补 `family`👨‍👩‍👧 / `theme`🎬，render 实测在「出行须知」独立成组、不再挤 other）；仅 selectPagesForDay 真正感知同行结构去选页仍属缓做（要碰 selector） |
| `meta.constraints[].text` | 一句话——既是 Claude 规划时的筛选依据，也是给用户的提醒 |
| `meta.constraints[].scope` | 默认 `'trip'`（整趟）；也可填 region slug（如 `'hualien'`）或 `'day:3'`。**全部 scope 统一进「出行须知」区**，region/day 级自动标注适用范围（`【第3天】`/`【hualien】`，无中文名映射降级显示 slug）。落 `day.reminders` 仅为可选当天增强，非唯一出口 |
| `meta.themes[]` | 整趟主题 / 朝圣**加分** lens 清单（影视打卡 / 圣地巡礼 / 朝圣 / 追星等），用户有贯穿性主题主线时才有。每条 `{name, kind, anchors[], scope}`，`kind` ∈ film/anime/literary/history/music/sport/other。与 `meta.constraints` **语义相反**：constraints 是**减分**把关、themes 是**加分**驱动——主题相关候选顶到候选前排、`anchors` 当优先锚点保入，但**绝不筛掉非主题的吃住行候选**。断裂 C 须把每个 theme 转写成一条 `meta.constraints`（`type:'theme'`）走「出行须知」出图（themes 自身不出图）。详见 checklist 9.0.5⑥ |

### 多城城际骨架承载位（病灶⑤ · 多城才有）

`meta.city_plan` 是 Claude 在 Stage 0 固化的**多城规划骨架**（照搬 `meta.constraints` 的「Claude 消费的数据容器」模式，不是界面字段）：固定「城市顺序 + 每城天数 + 城际交通段」，给下游 planner 注入分天、断裂 C 合成转场段当锚。多城（≥2 城）才有；单城可省略。**例外：单城但跨境（出发国≠目的国 / 经第三地中转）也要建，仅承载 `entry_exit` 入境段，见 checklist 9.0.6①跨境额外触发。** **多城用单个 planner 文件、按天聚焦当前城，不拆文件、不分图、不分次跑。**

```
meta.city_plan: {          // 多城（≥2城）才有；单城可省略。Claude 在 Stage 0 固化、全程当规划骨架，不是界面字段
  "cities": [               // 按游览先后顺序排列
    { "name": "曼谷", "region": "bangkok", "days": 3 },
    { "name": "清迈", "region": "chiangmai", "days": 2 },
    { "name": "普吉", "region": "phuket", "days": 2 }
  ],
  "intercity": [            // 城际交通段，cities 相邻两城之间各一段
    { "from": "曼谷", "to": "清迈", "mode": "flight", "duration": "约1.5h" },
    { "from": "清迈", "to": "普吉", "mode": "flight", "duration": "约2h" }
  ],
  "entry_exit": {           // 仅跨境行程才有：出发国→目的国的国际出入境段（非 city、非 intercity）。病灶⑫修复
    "inbound":  { "from": "深圳", "gateway": "香港", "to": "台中", "mode": "flight", "note": "深圳经港中转入境" },
    "outbound": { "from": "台北", "gateway": "香港", "to": "深圳", "mode": "flight" },
    "border_notes": [ "入台证/签注正本随身+电子档备份", "海关申报与禁带物", "落地电话卡/网卡", "口岸·机场→市区交通", "经第三地中转：行李是否直挂、是否需过境签" ]
  }
}
```

| 字段 | 用途 |
|---|---|
| `meta.city_plan.cities[]` | 城市分配表，按游览先后排序；每条 `{name, region, days}`。各城 `days` 之和（含转场日归属）= 总天数。**过夜驻点即使是景区村 / 自然驻点（布尔津 / 禾木 / 贾登峪等）而非行政市，也必须计入 `cities[]`，否则那几天无归属、对账对不上**（详见 checklist 9.0.6 ③） |
| `meta.city_plan.intercity[]` | 城际交通段，相邻两城之间各一段；每条 `{from, to, mode, duration}`。断裂 C 按它逐段合成 `{type:'transit',intent:'intercity'}` |
| `meta.city_plan.entry_exit` | **仅跨境行程才有**（出发国≠目的国 / 经第三地中转）：装出发国→目的国的国际出入境段 + 跨境操作提醒（`border_notes`）。中转口岸 `gateway` 只作途经点、**不建 region / 不占游览天**。与 9.0.8 分工：9.0.8 管「有没有资格跨境」（签证/护照），`entry_exit` 管「怎么落地」（海关/电话卡/口岸交通/中转行李）。详见 checklist 9.0.6 |

> **转场日天数归属口径（两种分支）：**
> **① 常规当日转场**：转场日**归入出发城**——上午出发城活动、下午搭车赶路，主活动归出发城；到达城当天傍晚才到、不单独占一天，从次日起算。各城 `days` 严格不重叠，之和 = 总天数。（与 medium-multi⑤ / 9.1 自洽：出发城活动只放赶车前、到达城活动只放抵达后。）
> **② 跨夜长途转场**（夜船 / 红眼航班 / 夜行卧铺，整段跨午夜，病灶⑫）：这一「在途夜」**单独占一天、作独立 T2 赶路日，既不归出发城也不归到达城**。到达城从抵达**次日**起算。天数对账：`各城 days 之和 + 独立跨夜赶路日数 = 总天数`。「归出发城」口径在此场景失效，不可套用。详见 checklist 9.0.6 ③。

完整 TRIP Schema（meta/global/extras/days/timeline/card）见 `assets/trip-print-v2.template.html` 顶部数据注释；planner 的 PLACES/dayPlans 结构见 `assets/planner.template.html`。

## 七、测试历程

| 版本 | case | 结果 |
|---|---|---|
| v1 | 京都 | 已废（暴露根本问题） |
| v2.1 | 泰国 5/1-5/8 | 跑通（14 条补丁） |
| v2.1 | 柳州 5/2-5/4 | 跑通（24 条 → v2.2 修订源头） |
| v2.2 | 上海 5/1-5/5 | ✅ 跑通（36 条 → v2.3 修订源头） |
| v2.3 | 罗马（进行中） | 第四轮验证 · 国际化场景 |
| **v2.4** | **京都 4/30-5/4 4 人** | **✅ 跑通·暴露 14 条 → v2.4 修订源头** |
| **v2.5** | **台湾 6/17-6/22 solo** | **暴露 3 类系统性问题 → v2.5 修订源头** |
| **v3.0** | **架构级重构** | **问卷 Q1/Q2 退役；改为四界面地图管道 discovery→planner→supplement→trip-print-v2；新增 9.0 路径分诊门** |

## 八 · 九、历次改进清单（已迁移至 checklist）

> **所有执行细则均以 [references/v2.4-checklist.md](references/v2.4-checklist.md) 为准。**
> 本章只保留分类索引，方便快速定位。

| 类别 | checklist 章节 | 内容 |
|---|---|---|
| cards schema | 第一章 | 餐厅 / 景点 / 转场 / 酒店四类完整字段 |
| 字体样式 | 第二章 S-1~S-5 | 字体堆栈、timeline 布局、链接按钮风格 |
| 工程流程 | 第三章 W-1~W-4 | 交付节奏、渲染函数、inline JSON 注入 |
| 事实核查 | 第四章 F-0~F-3 | 台账格式、餐厅必查、动线核实 |
| Stage 3 执行 | 第五章 | 渲染步骤顺序 |
| 链接策略 | 第七章 | 禁用站点、推荐链接库 |
| Stage 3 对账 | 第八章 8.1~8.4 | 渲染自检、链接验证、数据完整性 |
| 路径自检 | 第九章 9.0 | A/B/C 路径判断自检 |
| 跨城锁定 | 第九章 9.1 | G 规则（region 验证） |
| 坐标密度 | 第九章 9.2 | M 规则（地图点位） |
| 主动研究 | 第九章 9.3 | A 规则（禁止甩给用户） |
| 美食密度 | 第九章 9.4 | D 规则（三餐具名） |
| 备选质量 | 第九章 9.5 | B 规则（地理匹配） |

---

---
> Source: [logic0512/travel-itinerary](https://github.com/logic0512/travel-itinerary) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
