## study-abroad-planner

> >-


# Study Abroad Planner

A privacy-first planner that turns a user's profile into a structured study
abroad + immigration pathway. The skill is **country-agnostic** by design;
it ships with Australia as a worked example and stub references for the UK,
NZ, and Canada. Add new countries by dropping a `references/<country>.md`.

## Privacy contract (READ FIRST)

This skill MUST NOT contain personal data. Follow these rules every time:

| Rule | Why |
|---|---|
| Never bake a real name, age, score, university, or salary into the skill | The skill may be published; PII must not leak |
| Use `<placeholder>` syntax in all examples and references (`<user_age>`, `<english_score>`, `<target_country>`) | Keeps examples copy-paste safe |
| Never read a `user-profile.json` from disk unless the user explicitly passes a path | Privacy by default |
| Never echo back profile data in skill logs or telemetry | Audit hygiene |
| Before any push to git, run the privacy audit in §"Pre-release checks" | Last line of defence |

If the user wants their saved profile used, ask them to paste a path or
inline a JSON object — never read silently.

## When to read which reference

| Country the user mentions | Read this reference first |
|---|---|
| Australia / 澳洲 / AU / 189 / 190 / 491 / EOI | `references/australia.md` |
| UK / 英国 / UCAS / Russell Group | `references/countries.md` → UK section |
| New Zealand / 新西兰 / NZ / SMC | `references/countries.md` → NZ section |
| Canada / 加拿大 / IRCC / PNP | `references/countries.md` → Canada section |
| Anything else | Ask the user to clarify; do not guess |

If the user mentions a country you have no reference for, be honest:
"sorry, this skill does not cover <country> yet — here is the data-source
template you can use to research it yourself" (see `references/data-sources.md`).

## Conversation flow

Ask the user for profile data in this order. Each question is required;
do not skip. If the user already answered some of these in earlier turns
of the same conversation, reuse the values and confirm.

| # | Question | Why we need it | Example valid answer |
|---|---|---|---|
| 1 | What is your current age (or age at planned graduation)? | Drives EOI points and visa age cut-offs | "27" or "30" |
| 2 | What is your English test score? Format: `<test> <overall> (<listening>, <reading>, <writing>, <speaking>)` | Entry requirement + visa points | "PTE <65> (<60>, <65>, <60>, <65>)" or "IELTS <7.0> (<6.5>, <7.0>, <6.5>, <7.0>)" |
| 3 | What is your highest education level + GPA / average? | Entry requirement + visa points | "Bachelor in <Engineering>, GPA <3.5>/4.0" |
| 4 | What is your total budget in your home currency (including tuition + 2 years living)? | Filters affordable schools | "500000 RMB" or "AUD 100000" |
| 5 | Target country + target field of study? | Routes to the right reference + visa subclass | "Australia, Master of Occupational Therapy" |
| 6 (optional) | Any constraints? (e.g. regional preference, family in destination, work rights) | Lets the plan bias toward 491 / 190 over 189 | "Prefer regional area for PR points" |

After collecting the profile, **confirm it back to the user in a single
short table** and ask "looks right, or want to correct anything?" before
proceeding. This is the single biggest source of mistakes.

## Output template

Produce ONE markdown report with the following sections in this order.
Keep it scannable — the user is choosing a multi-year path, not reading
a thesis.

```markdown
# 🛫 Study Abroad Pathway — <target_country> / <target_field>

> Generated: <YYYY-MM-DD> · Inputs: <one-line summary of profile>

## 1. Profile snapshot
| Field | Value |
|---|---|
| Age at graduation | <value> |
| English | <test> <overall> (<sub-scores>) |
| Education | <degree> in <major>, GPA <value> |
| Budget | <value> <currency> (tuition + 2yr living) |
| Target | <country>, <field> |
| Constraints | <list or "none"> |

## 2. Shortlist (3-5 schools)
For each school, give: name, location, tuition/year, duration, entry
requirements, ranking signal (QS / THE / local), and one-line "why it
fits you". Sort by fit, not by prestige.

## 3. Visa pathway
Visa subclass(es) the user is eligible for, with EOI / points score
breakdown, expected invitation round competitiveness, and timeline
(sketch milestones: enrolment → graduation → skills assessment → EOI
→ invitation → grant).

## 4. Cost & cashflow
Tuition per year + living per week + total. Show the gap to the user's
budget explicitly. Flag any "hidden" costs (skills assessment fee,
English retake, registration with professional body).

## 5. Risk register
3-5 risks specific to this profile, with severity (low/med/high) and a
mitigation. Common risks:
- English score below entry requirement
- Occupation not on the relevant skilled list
- Quota changes mid-application cycle
- Skills assessment body requires additional supervised practice
- PR points threshold rises faster than expected

## 6. Recommended next actions
A numbered list of 3-7 concrete next steps the user can do THIS WEEK.
Order by leverage. Each action should have a verifiable outcome
("book PTE exam by <date>", not "study harder").
```

## Country routing rules

- If the user mentions a country twice in the same conversation, assume
  the first one and confirm.
- If the user names a specific school (e.g. "Charles Darwin University"),
  still produce the full pathway — the school is a strong signal of
  preference but not the whole plan.
- If the user asks only about one slice (e.g. "EOI 打分怎么算"), answer
  that slice and offer the full plan as a follow-up. Do not force the
  full report on a focused question.

## Data sources (no scraping — just point the user)

Always link to authoritative public sources, never to scraped copies.
See `references/data-sources.md` for the canonical list per country.

| Need | Source |
|---|---|
| AU occupation list (MLTSSL / STSOL) | https://immi.homeaffairs.gov.au/visas/working-in-australia/skill-occupation-list |
| AU EOI / SkillSelect | https://immi.homeaffairs.gov.au/visas/getting-a-visa/visa-listing/skilled-independent-189 |
| AU QILT (course quality + employment) | https://www.qilt.edu.au/ |
| UK UCAS tariff | https://www.ucas.com/undergraduate/applying-to-university/entry-requirements |
| NZ Skilled Migrant Category | https://www.immigration.govt.nz/work-visas/skilled-migrant-category |
| Canada Express Entry | https://www.canada.ca/en/immigration-refugees-citizenship/services/immigrate-canada/express-entry.html |

## Pre-release checks (run before publishing this skill)

This is the publish gate. Run all four before any `git push` or share.

1. **PII grep** — search the skill directory for common PII patterns. Use
   generic patterns, never user-specific values:
   ```bash
   # Generic check — replace with patterns matching your own PII
   grep -rE "<suspicious-pattern>" \
     ~/.claude/skills/study-abroad-planner/ || echo "OK no PII"
   ```
   If anything matches, scrub it. The examples in the skill use
   generic `<placeholder>` syntax — no real scores, no real names.

2. **Placeholder audit** — every example value must be wrapped in
   `<angle_brackets>`. Real values are forbidden in examples.

3. **Frontmatter check** — `name` matches the directory name;
   `description` is "pushy" enough that it would trigger on realistic
   user queries (see skill-creator guidance).

4. **No fabricated URLs** — every link in this skill must point to a
   public authoritative source, not a placeholder. If you are unsure
   a link resolves, drop it.

5. **Human review gate** — show the publish checklist to the user
   and wait for explicit "push" / "发" / "好的" before any push.

## What this skill does NOT do

- It does not file visa applications, draft statements, or contact
  universities on the user's behalf.
- It does not guarantee admission or visa grant.
- It does not replace a licensed migration agent. If the user's case
  is non-trivial (e.g. dependent family, prior refusals, complex work
  history), recommend they consult a MARA-registered agent (AU) /
  OISC-regulated adviser (UK) / ICCRC consultant (CA) for the legal
  step. Planning is free; the legal step is not.

## Reference index

- `references/australia.md` — full AU 189 / 190 / 491 + EOI breakdown
- `references/countries.md` — UK / NZ / Canada short pathways
- `references/data-sources.md` — canonical public sources per country
- `examples/sample-output.md` — sample report with placeholder values

## 🆕 v0.2.1 — Per-course data source tracking (2026-06-09)

**v0.2.0 shipped the JSON dataset. v0.2.1 makes every output row
traceable to the exact URL it came from:**

| Change | What it does | Why it matters |
|---|---|---|
| Added `source` field to every course in `australian-courses.json` | Each course carries `{course_page, retrieved, verified_by}` (plus optional `verify_flag`) | When a university changes a requirement, the user can check retrieval date and decide whether to re-verify before relying on the output |
| New `--show-source` flag in `aus-course-filter.mjs` | Prints the source block under each course in text output | Visible audit trail in the terminal — no need to inspect the JSON |
| JSON output now includes `data_source` block | Top-level `data_source: {schema_version, last_updated, schema_note}` | Downstream consumers (other CLIs, dashboards) get provenance metadata |
| 4 courses flagged with `verify_flag` | UTAS / Flinders (PTE vs IELTS policy), UniMelb / USyd (generic URL) | Known gaps surfaced explicitly so the user re-verifies before applying |

**Backward compatible**: existing `aus-course-filter` invocations without
`--show-source` still produce the same output as v0.2.0. The `source`
field is additive — old consumers that ignore unknown fields keep working.

**Use it**:

```bash
# Show source for every course
node ~/.claude/skills/study-abroad-planner/bin/aus-course-filter.mjs \
  --pte 70 --pte-each-band 65 --profession OT --show-source

# JSON consumers get data_source + per-course source block
node ~/.claude/skills/study-abroad-planner/bin/aus-course-filter.mjs \
  --json --pte 70 ... | jq '.data_source, .results[0].source'
```

**When to re-pull data** (per `trigger_conditions_for_adding_more_courses`):
- Annual tuition refresh (January) — every school's URL stays the same
- QILT update (February) — employment % shifts
- Any university page URL change — re-retrieve the course_page URL

The schema now makes the freshness signal explicit: a 2024 retrieval
date tells the user "this is old, re-check before relying on it".

### 🆕 v0.2.1 — SkillSelect OT data (additional)

`data/skillselect-ot-invitations.json` ships alongside the courses
dataset — 8 historical invitation rounds for OT (ANZSCO 252411) covering
2023-05 to 2026-03, with FY2025-26 pool metrics. Read this when the user
asks about PR pathway timing, EOI score targets, or competitive context.

### 🆕 v0.2.1 — AHPRA/OTBA/OTC pathway data (additional)

`data/ahpra-ot-registration-pathway.json` documents the 2-track OT
registration + migration skills assessment pathway:

| Track | Owner | Purpose | Fee | When |
|---|---|---|---|---|
| AHPRA registration | AHPRA / OTBA | To legally practice as OT in AU | varies (verify at portal) | Post-graduation |
| OTC migration skills | OTC | Required for EOI lodgement | A$800 desktop | Post-graduation, 6mo before EOI |

**Critical 27 Oct 2025 change**: OTC stopped assessing qualifications for
REGISTRATION (moved to AHPRA/OTBA's 3 new streamlined pathways), but
OTC continues to assess for MIGRATION. Both tracks still required, separately.

### 🆕 v0.2.1 — 7-school PR pathway comparison (additional)

`data/seven-school-pr-pathway-comparison.json` ranks 7 Australian OT
schools by PR pathway structural advantage using 4 proxy signals:

| Signal | Why it matters |
|---|---|
| QILT FT employment % | Higher = better graduate outcomes |
| Regional bonus eligibility | +5 EOI points for 491 |
| State nomination availability | NT/TAS = easier than NSW/VIC |
| State OT workforce density | Smaller pool = easier nomination |

**Headline ranking**: CDU (Darwin NT) > UTAS (TAS) > La Trobe (Bendigo)
> Flinders (Adelaide) > UniSA > UniMelb > USyd. Detailed PR conversion
estimates in the data file (NOT real grant rates — AHPRA does not
publish school-of-graduation; estimates are best-effort directional).

**CRITICAL caveat**: Real PR grant rates by school are NOT publicly
available. AHPRA does not track school of graduation in published
statistics. Universities do not publish PR grant data. This is a
structural ranking, not an actual outcome report.

---

## 🆕 v0.3.0 — 核心:把功能串成一条咬合的流水线 (Pipeline)

v0.2.1 的能力是并列的;v0.3.0 要求**按固定顺序咬合执行**,信任在每一步被强制检查:

```
① validate  →  ② gate  →  ③ analyse  →  ④ synthesise(按来源等级加权)  →  ⑤ honesty self-check  →  输出
```

1. **validate** — 先跑 `node bin/source-validator.mjs`。有 ❌(缺源/社媒)直接停;有 ⚠️(过期/聚合器/待核)先标记,带着标记往下走,**不在过期数据上直接出结论**。
2. **gate** — 资格门(见下),先抛致命硬伤 + 替代方案。
3. **analyse** — 现有模块(course-filter / eoi-calculator / 各 JSON)。
4. **synthesise** — 7 校排名等结论**按 `data/source-tiers.json` 的 tier 加权**:重大推荐不得仅靠 tier≥4。
5. **honesty self-check** — 出报告前的强制自检(见下),让"诚实优先"有牙。

---

## 🆕 v0.3.0 — Eligibility Gate(资格门 · 含 OT 专属雷)

报告**开头**先跑这一段,1 段话给出"有没有致命硬伤 + 替代方案",别让用户读完整篇才发现走不通。
通用项(职业是否在列表、年龄是否过加分高峰、预算缺口、CRICOS 注册)照旧,**OT 专属四雷必须显式检查**:

1. **三道英语门**(读 `data/ahpra-english-standard.json`)
   - gate1 入学 / gate2 **AHPRA ELS 注册** / gate3 EOI 加分,三者独立,**真正约束 = 三者最高**。
   - ★ **ELS 教育豁免默认按"不成立"处理**:国内非英语本科 + 单个澳洲 2 年硕士大概率不满足豁免年限,
     仍需考试达 ELS 线。除非 AHPRA self-assessment tool 确认,否则把"考到 ELS 分"排进时间表。
2. **临床实习合规** — OT 学位有强制 fieldwork:实习小时数、可能的乡村实习,以及 pre-placement 合规
   (无犯罪记录证明、Working With Children Check、疫苗接种、急救/CPR,部分要 NDIS Worker Screening)。
   这些是 OT 专属的**时间+钱+可行性**约束,需提示并尽量量化(各校官网/手册,标来源)。
3. **临时 vs 完全注册** — 27 Oct 2025 改革后的 streamlined pathway:确认是直接 general registration,
   还是带 supervised practice 条件的 provisional;影响"何时能执业/起算工作经验"。
4. **州担保真实标准 ≠ 池子大小** — 7 校排名用"州 OT 注册人数"作易州担保代理(NT 最小→CDU 第一),
   但 190/491 有**明文标准**(职业是否在该州当期列表、是否要求 state study pathway、工作承诺、
   有时 job offer)。对推荐州(如 NT)必须核**真实提名标准**(tier 2 州政府页),不能只靠代理信号。

> 任一雷亮红灯 → 先说清、给替代路径,再继续;判断都登记进 passport(带 tier + retrieved)。

---

## 🆕 v0.3.0 — 来源等级 & 数据护照 (Source-Tier Passport)

- 每个数据点记:`{ value, source, tier, retrieved, status }`(tier/规则见 `data/source-tiers.json`)。
- **状态标签会随时间自动降级**:validator 扫到超保鲜期 → ✅ 自动变 ⚠️;tier≥4 → ⚠️ 需佐证;缺源 → ❌。
- **输出纪律**:报告里出现的 ✅ **必须紧跟 retrieved 日期**(无日期的 ✅ 视为未校验);
  重大推荐(选校 / PR 可行性)**不得仅由 tier≥4 聚合器支撑**——这正是修 SkillSelect 数据来自
  Visa Sidekick/Immitrend 的根问题:要么补 tier1 的 DHA 官方交叉佐证,要么标"待官方确认"。

---

## 🆕 v0.3.0 — 概率改"分级",禁止裸百分比

- 7 校 PR 列**禁止**写 `85% / 75%`(AHPRA 不按学校追踪 PR,精确百分比=假精确)。
- 改为 **Tier 排名 + 定性分级(高/中/低)+ confidence(high/medium/low)+ 所依据的代理信号**。
  例:`CDU — PR 前景:高(confidence: medium;依据:偏远+NT 最小 OT 池子+96.4% FT 就业;
  真实 grant-by-school 数据不公开)`。

---

## 🆕 v0.3.0 — 诚实自检 (Honesty Self-Check · 出报告前强制)

把"诚实优先于安慰"从一句口号变成**出报告前必过的步骤**(让机制逼出诚实,不靠自觉):
- [ ] 每个代理/估算数字都带了 caveat 与 confidence?没有裸百分比?
- [ ] 每个 ✅ 都紧跟 retrieved 日期?validator 的 ⚠️/❌ 都在报告里如实反映?
- [ ] 重大推荐是否仅靠 tier≥4?若是,已标"待官方确认"?
- [ ] 档案弱处(英语/经验/预算)是否如实说明 + 给了 B 计划,而非美化?
- [ ] 三道英语门、ELS 豁免风险是否显式呈现,未被乐观默认掩盖?

---

## 🆕 v0.3.0 — Re-run 与 validator 咬合

- **Re-run 前先自动跑 validator**(`--strict`):关键数据若过期/降级,先拦下重查,再增量重算,
  避免"重算建在一堆过期数据上"。
- 增量规则不变:只重答变化项(如 PTE 重考 79 → 只重算 EOI/签证段),其余段保留;
  并指出"这次相比上次,什么变了"。

---

## 🆕 v0.3.0 — 未来预案 (Pessimistic Branches · 针对 2030)

被动 Re-run 之外,预先写好**主动分支 + 触发器**(分数线 4 年后高度不确定):
- 触发器示例:`若 FY27 R1 OT 189 分数线 ≥ 90 → 启动 B 计划`:
  - 转 491 偏远地区(NT/TAS 等)+ 州担保提名;
  - 配偶英语/技能加分;
  - 评估 Professional Year / NAATI 等可加分项;
  - 必要时延后 EOI、补澳洲工作经验。
- 每次刷新(validator 报告)后,对照触发器复核当前选路是否仍最优。

---

## 🆕 v0.3.1 — 高频变动项机制 (Freshness Policy · 政策类数据自动 hook)

**问题**:v0.3.0 之前的 validator 用单一 180 天阈值检查所有数据。但政策类(英语线、EOI 分数线、配额)实际**比 180 天短得多就过时**,course facts 反而**比 180 天长得多也不影响**。两类数据共用同一阈值 = 错配。

**修法**:把"保鲜期"从"写死的 180 天"升级成"按 `data_type` 分级的政策表"。

### 工作流

1. **数据文件加 `data_type` 字段**:每个 `source` 对象声明所属的政策类型:
   ```json
   "source": {
     "course_page": "...",
     "retrieved": "2026-06-09",
     "data_type": ["course_facts", "english_standard", "tuition_fee"]
   }
   ```

2. **`data/freshness-policy.json` 集中管理阈值**:
   | data_type | warn_days | error_days | 含义 |
   |-----------|----------:|-----------:|------|
   | `english_standard` | 30 | 60 | AHPRA ELS / PTE 等价表 / DHA 积分换算 |
   | `eoi_threshold` | 7 | 30 | SkillSelect 邀请轮 |
   | `visa_allocation` | 7 | 14 | 州担保配额 |
   | `skills_assessment_process` | 30 | 90 | OTC / AHPRA 评估流程 |
   | `policy_decision` | 14 | 30 | 近期改革(27 Oct 2025 等)|
   | `occupation_list` | 30 | 90 | MLTSSL / SOL 列表 |
   | `course_facts` | 180 | 365 | duration, intake, prereq, scholarship |
   | `tuition_fee` | 365 | 730 | 年度学费 |
   | `register_data_static` | 730 | 1825 | QILT / AHPRA 慢变数据 |

3. **`bin/source-validator.mjs` 自动按 policy 检查**:每个 retrieved 日期若超 warn → 标 ⚠️;超 error → 标 ❌(block synthesise)。

4. **`bin/release-precheck.mjs` 一键跑 PII + validator**:pre-push 必跑。
   ```bash
   node bin/release-precheck.mjs              # exit 0/1/2
   node bin/release-precheck.mjs --strict     # warnings 也 block
   node bin/release-precheck.mjs --force      # 跳 PII (已知会过)
   ```

5. **新增 `data_type` 类型时**:先在 `freshness-policy.json` 加规则,再在数据文件用——不要写死数字。

### Synthesise Gate 规则

合成 6 段报告前**必须**先跑 `release-precheck`:
- **exit 0** → clean,照常 synthesise
- **exit 2** → 有 freshness warnings,允许 synthesise 但报告**必须**加 `data_staleness` footer 列出哪些字段过期
- **exit 1** → 有 PII / tier_5 / 严重过期,**拒绝** synthesise,先修

### 杠杆:为什么这是"修一处,治一类"

下次 AHPRA ELS 标准变更(可能是 76 speaking,可能别的)——
- 旧机制:你得在 `data/ahpra-english-standard.json` 改 `pte_each_band_min: 65` → 新数字,commit,推
- **新机制**:数据本身 `retrieved` 日期一过 30 天 warn / 60 天 error,validator 自动报警——
  **不用碰数字**,只需要去 AHPRA 官方页拉一次,把 `retrieved` 改成今天

English 不是唯一的高频项。`visa_allocation`(配额 7 天)、`eoi_threshold`(7 天)、`policy_decision`(14 天)也都在这个机制保护下。任何政策类数据自动 hook,不需要每次新增类型都改 validator 代码。

---
> Source: [wujiajun4/study-abroad-planner](https://github.com/wujiajun4/study-abroad-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
