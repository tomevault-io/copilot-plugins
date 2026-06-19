## vc-analyst-skill

> Deep venture capital due diligence analysis using a multi-agent architecture. Use this skill whenever the user wants to analyze, evaluate, research, or diligence a startup, project, protocol, token, or company from an investor perspective. Triggers on phrases like 'analyze this project', 'do DD on', 'VC analysis', 'evaluate this startup', 'investment memo', 'diligence this', 'should I invest in', 'what do you think about [project]', 'analyze these founders', 'competitive analysis of', or any request to evaluate projects or founders for investment potential. Also triggers when users provide a list of project URLs, Twitter handles, or company names and want them assessed. Covers both crypto/web3 and traditional tech projects. This skill dispatches parallel sub-agents for product analysis, competitive intelligence, traction metrics, founder background, social influence, and wealth/track-record assessment — each in isolated context windows — then synthesizes into a structured investment memo.


# VC Analyst Skill

Perform institutional-grade venture capital due diligence using a **multi-agent architecture**. The main agent acts as a thin orchestrator (like a managing partner) while specialized sub-agents do deep research in isolated context windows (like junior analysts), returning structured JSON summaries.

## Why Sub-Agents?

VC due diligence is context-expensive. A single project analysis can require crawling websites, scraping social metrics, fetching on-chain data, reading GitHub repos, and cross-referencing competitor landscapes. Running this in the main context window degrades quality fast — especially for batch analysis of multiple projects. Sub-agents solve this:

- Each sub-agent works in its own fresh context window
- Only structured summaries return to the main agent (~300-500 tokens per sub-agent vs ~15,000+ tokens of raw web content)
- Multiple projects can be analyzed in parallel
- Multiple dimensions of the SAME project are analyzed in parallel
- If one source blocks or errors, it doesn't pollute the main context

## Architecture Overview

```
Main Agent (Managing Partner / Orchestrator)
│
├─ Phase 1: Input Parsing (inline, lightweight)
│   └─ Parse project list → structured entries
│   └─ Identify: name, URL, socials, founder names, chain/sector
│
├─ Phase 2: Per-Project Dispatch (parallel per project)
│   │
│   │  ┌─ Project Alpha ──────────────────────────────────┐
│   │  │                                                   │
│   │  │  PROJECT COMPONENT (parallel sub-agents):         │
│   │  │  ├─ Product Scout ─────── product, roadmap,       │
│   │  │  │                        philosophy, UX, docs    │
│   │  │  ├─ Competitive Intel ─── moats, competitors,     │
│   │  │  │                        differentiation         │
│   │  │  ├─ Traction Analyzer ─── social metrics,         │
│   │  │  │                        on-chain data, revenue  │
│   │  │  └─ Tech Deep Dive ────── GitHub, tech stack,     │
│   │  │                           code quality, activity  │
│   │  │                                                   │
│   │  │  FOUNDER COMPONENT (parallel sub-agents):         │
│   │  │  ├─ Background Scout ──── education, career,      │
│   │  │  │                        credentials, fit        │
│   │  │  ├─ Social Influence ──── Twitter, LinkedIn,      │
│   │  │  │   Scanner              audience quality,       │
│   │  │  │                        thought leadership      │
│   │  │  └─ Track Record & ────── past projects, exits,   │
│   │  │     Wealth Analyzer       funding raised,         │
│   │  │                           wealth signals          │
│   │  └───────────────────────────────────────────────────┘
│   │
│   │  ┌─ Project Beta (same structure, parallel) ─────────┐
│   ├──│  ...                                               │
│   │  └────────────────────────────────────────────────────┘
│   └─ ... (one pipeline per project)
│
├─ Phase 3: Cross-Project Comparison (inline, if multiple projects)
│   └─ Relative strengths, positioning matrix, ranking
│
└─ Phase 4: Investment Memo Synthesis (inline)
    └─ Compile structured memo with scores and recommendation
```

## ZERO HALLUCINATION POLICY

**This is the single most important rule in this skill. Violating it destroys the entire value of the analysis.**

Every sub-agent and the orchestrator MUST follow these rules:

1. **If you didn't find it, say "Not found" or "Insufficient data."** Never infer, guess, or fill in plausible-sounding details. A blank field with "Not found — searched [sources]" is infinitely more valuable than a fabricated answer.

2. **Distinguish between CONFIRMED, ESTIMATED, and NOT FOUND:**
   - ✅ **Confirmed** — you found this specific data point from a named, citable source
   - 📐 **Estimated** — you derived this from indirect evidence (state your reasoning and evidence)
   - ❌ **Not found** — you searched and came up empty (list what you searched)
   
3. **Every claim needs a source.** If a sub-agent returns a metric or fact, it MUST include where it came from. "Revenue: $5M ARR" is useless. "Revenue: ~$5M ARR (📐 estimated from 500 enterprise customers × ~$10k/yr pricing page)" is useful. "Revenue: Not found — checked Crunchbase, press releases, Token Terminal, pricing page" is also useful.

4. **Confidence tagging is mandatory.** Every sub-agent output includes a `confidence` field. This is NOT optional decoration — it directly informs how the orchestrator weighs the data in the final memo:
   - **High** — multiple corroborating sources, recent data, official sources
   - **Medium** — single source, or data >6 months old, or estimated from indirect evidence
   - **Low** — sparse data, only social media mentions, or significant gaps in what you'd expect to find

5. **The absence of information IS information.** A project with no press coverage, no Crunchbase profile, no GitHub, and a 2-week-old Twitter account is telling you something. Report the absence explicitly — it's a signal, not a failure.

6. **Never round up ambiguity into confidence.** If you're not sure whether a project has 1,000 or 10,000 users, say "User count unclear — Twitter bio claims '10k+ users' but no independent verification found" rather than picking a number.

---

## Stage Detection Framework

**Before dispatching sub-agents, the orchestrator must first assess the project's likely stage.** This determines which sub-agents are dispatched, what data is expected vs. aspirational, and how to calibrate scoring.

### Stage Classification

| Stage | Signals | Expected Data Availability |
|---|---|---|
| **💡 Idea / Pre-product** | Landing page only, "coming soon", waitlist, no product screenshots, Twitter <1 month old | Almost nothing. Founder analysis is 90% of the value. |
| **🔧 Building / Pre-launch** | GitHub activity but no live product, testnet/devnet, "alpha" or "beta" labels, small Discord of early testers | Some tech data, minimal traction, no revenue |
| **🌱 Launched / Early** | Live product <12 months, <1000 users/addresses, first press mentions, seed round | Some traction, early metrics, founder story emerging |
| **📈 Growth** | 12+ months live, meaningful metrics (>10k users or >$1M TVL), Series A+, regular press | Full data expected across all dimensions |
| **🏢 Mature / Established** | Multi-year track record, institutional investors, significant revenue/TVL, market leader or strong challenger | Comprehensive data available; gaps are red flags |

### How Stage Affects Analysis

**For 💡 Idea / Pre-product projects:**
- Skip Traction Analyzer (there's nothing to measure — don't fabricate metrics)
- Skip Tech Deep Dive if no GitHub exists
- HEAVILY weight Founder Component (team is almost the entire investment thesis at this stage)
- Product Scout focuses on vision/whitepaper/thesis quality rather than UX/features
- Competitive Intel is still valuable (understanding the space they want to enter)
- In the memo, explicitly state: "This is a pre-product assessment. The investment thesis rests almost entirely on team conviction and market opportunity."

**For 🔧 Building / Pre-launch projects:**
- Traction Analyzer runs but expects near-zero metrics — frame output as "baseline" not "weak"
- Tech Deep Dive is HIGH value (code is the product at this stage)
- Product Scout evaluates vision + technical design docs rather than live UX
- In the memo, explicitly frame scores against stage-appropriate benchmarks

**For 🌱 Launched / Early projects:**
- All sub-agents run, but with stage-calibrated expectations
- Small numbers are expected and should be flagged as "stage-appropriate" not "concerning"
- Growth RATE matters more than absolute numbers at this stage

**For 📈 Growth and 🏢 Mature:**
- All sub-agents run with full expectations
- Gaps in data at these stages ARE red flags and should be called out
- Absolute numbers and unit economics matter more than growth rates

### Stage-Calibrated Scoring

**CRITICAL: The scorecard in the investment memo MUST be calibrated to stage.**

A pre-product project cannot score 1/10 on traction — it should show "N/A (pre-product)" or be benchmarked against other pre-product companies. A growth-stage project with no public GitHub should score lower on tech than a pre-launch project with no public GitHub (because at growth stage, this is a choice to hide; at pre-launch, it's normal).

Include a `stage_context` note on every scorecard dimension:
- "Traction: 3/10 — low for a growth-stage project with 18 months of runway burned"
- "Traction: N/A — pre-product, no metrics expected yet"
- "Tech: 7/10 — strong for seed stage; active GitHub with 4 contributors and weekly releases"

---

## Search Strategy

**CRITICAL: Use all available search tools in parallel for maximum coverage.**

For EVERY sub-agent dispatch, instruct the sub-agent to use these tools simultaneously when available:
- `web_search` (Anthropic's built-in search)
- `exa:web_search_exa` (Exa semantic search — better for finding company pages, LinkedIn profiles, research papers)
- `exa:company_research_exa` (Exa company-specific research — use for any named company/project)
- `brave-search:brave_web_search` (Brave search — good for recent/real-time content)
- `web_fetch` (for crawling specific URLs the user provides or that search surfaces)

Always run web_search, exa, and brave in parallel for each query. Then use web_fetch to deep-dive the most promising URLs.

---

## Phase 1: Input Parsing & Stage Detection (Main Agent, Inline)

Before dispatching any sub-agents, parse the user's input into structured project entries AND determine the project stage.

### Step 1A: Parse Inputs

For each project, extract or ask for:
- **Project name**
- **Website URL** (if provided)
- **Social handles** (Twitter/X, Discord, Telegram)
- **Founder name(s)** (if provided)
- **Chain/sector** (e.g., Ethereum L2, DeFi, AI infra, SaaS)
- **Stage** (if user specifies, otherwise auto-detect in Step 1B)
- **Any additional context** the user provides

If the user provides bare URLs or names without context, that's fine — the sub-agents will discover the rest. At minimum you need a project name or URL.

**Batch detection:** If the user provides multiple projects, parse each into a separate entry. Each project gets its own independent pipeline in Phase 2.

### Step 1B: Quick Stage Detection

Before dispatching the full sub-agent pipeline, do a FAST initial probe to determine the project's stage. This takes 1-2 search calls and determines which sub-agents to dispatch and how to calibrate expectations.

**Quick probe queries (run in parallel):**
- `web_search`: "[project_name]" (check what exists)
- `exa:company_research_exa`: "[project_name]" (check for company data)

**From the results, classify the stage** using the Stage Detection Framework above.

**Then adapt the sub-agent dispatch plan:**
- For 💡 Idea/Pre-product: Skip Traction Analyzer, conditionally skip Tech Deep Dive
- For 🔧 Building/Pre-launch: Run all but with adjusted expectations passed to each sub-agent
- For 🌱+ stages: Run all sub-agents normally

**Pass the detected stage to every sub-agent** as an input parameter so they can calibrate their research depth and expectations accordingly.

---

## Phase 2: Per-Project Pipeline

For each project, dispatch the following sub-agents. If there are multiple projects, dispatch their pipelines in parallel — each project's pipeline is fully independent.

### PROJECT COMPONENT

#### Sub-Agent 2A: Product Scout

Dispatch a sub-agent to deeply analyze the product itself.

**Sub-agent prompt:** Read `./prompts/product-scout.md` and pass it:
- Project name, URL, any known product details
- Instructions to crawl the website, docs, blog, and changelog

**What it returns (structured JSON):**
- Product description (what it does, for whom)
- Product philosophy and strategic vision
- Current features vs. roadmap / what's planned
- UX/design quality assessment (1-10)
- Documentation quality (1-10)
- Revenue model / monetization strategy
- Pricing (if visible)
- Key differentiators from user's perspective

**Tool access:** web_search, exa, brave-search, web_fetch

#### Sub-Agent 2B: Competitive Intelligence

Dispatch a sub-agent to map the competitive landscape.

**Sub-agent prompt:** Read `./prompts/competitive-intel.md` and pass it:
- Project name, sector, brief description from Phase 1 parsing
- Any competitors the user already mentioned

**What it returns (structured JSON):**
- Top 3-5 direct competitors with brief descriptions
- Competitive advantages / moats for the target project
- Key differences in product approach (technical, UX, go-to-market)
- Key differences in business model
- Market positioning (leader / challenger / niche / emerging)
- Barriers to entry for new entrants
- Network effects or switching costs analysis

**Tool access:** web_search, exa, brave-search, web_fetch

#### Sub-Agent 2C: Traction Analyzer

Dispatch a sub-agent to gather quantitative traction data.

**Sub-agent prompt:** Read `./prompts/traction-analyzer.md` and pass it:
- Project name, URL, social handles, chain (if crypto)
- Whether to look for on-chain OR off-chain metrics (or both)

**What it returns (structured JSON):**
- **Social metrics:** Twitter followers, engagement rate, Discord members, Telegram size, growth trends
- **On-chain data** (if applicable): TVL, daily active users, transaction volume, token holders, protocol revenue, fees generated. Sources: DefiLlama, Dune Analytics, Token Terminal, DappRadar
- **Off-chain data** (if applicable): web traffic (SimilarWeb estimates), app store rankings, G2/ProductHunt reviews, Crunchbase funding data
- **Revenue:** known ARR, MRR, or protocol revenue
- **Growth trajectory:** month-over-month or quarter-over-quarter trends
- **Red flags:** declining metrics, bot activity, fake followers

**Tool access:** web_search, exa, brave-search, web_fetch, bash (for API calls if needed)

#### Sub-Agent 2D: Tech Deep Dive

Dispatch a sub-agent to assess technical quality and activity.

**Sub-agent prompt:** Read `./prompts/tech-deep-dive.md` and pass it:
- Project name, GitHub org/repo URL (if known), tech stack hints

**What it returns (structured JSON):**
- GitHub activity: stars, forks, contributors, commit frequency, last commit date
- Code quality signals: test coverage, CI/CD, documentation, code review practices
- Tech stack and architecture choices
- Open source vs. closed source assessment
- Security: audits completed, bug bounties, known vulnerabilities
- Smart contract risk (if applicable): audit firms used, TVL at risk
- Developer ecosystem: SDKs, APIs, integrations, developer adoption

**Tool access:** web_search, exa, brave-search, web_fetch, bash (for GitHub API)

---

### FOUNDER COMPONENT

For each named founder (or key team member), dispatch the following sub-agents. If multiple founders are named, you may batch them into the same sub-agent call or dispatch separate ones.

#### Sub-Agent 2E: Background Scout

Dispatch a sub-agent to research founder credentials and background.

**Sub-agent prompt:** Read `./prompts/background-scout.md` and pass it:
- Founder name(s), company/project name
- Any known LinkedIn URL, personal site, or bio

**What it returns (structured JSON):**
- **Education:** university, degree, graduation year, notable programs/honors
- **Age estimate:** based on graduation year and career timeline (approximate, framed as career stage)
- **Career history:** previous roles, companies, duration at each
- **Credentials and fit:** how well their background maps to what they're building
- **Domain expertise:** years in the relevant industry/tech area
- **Notable achievements:** awards, publications, patents, speaking engagements
- **Previous projects / startups:** what they built before, outcomes
- **Red flags:** very short tenures, gaps, misrepresentations, controversies

**Tool access:** web_search, exa (with `category: "people"` for LinkedIn-style results), brave-search, web_fetch

#### Sub-Agent 2F: Social Influence Scanner

Dispatch a sub-agent to assess founder's public presence and influence.

**Sub-agent prompt:** Read `./prompts/social-influence.md` and pass it:
- Founder name(s), known Twitter handle, LinkedIn URL
- Project/company name for context

**What it returns (structured JSON):**
- **Twitter/X:** follower count, avg engagement, posting frequency, audience quality (are followers real? relevant?), notable followers/mutuals, content themes
- **LinkedIn:** connection count, post engagement, endorsements, recommendation quality
- **Other platforms:** YouTube, Substack, podcast appearances, conference talks
- **Thought leadership score (1-10):** are they seen as an authority in their space?
- **Community building ability:** do they rally people? quality of discourse in replies
- **Network quality:** who engages with them? other founders? VCs? developers?
- **Platform influence assessment:** could they drive distribution for the product?

**Tool access:** web_search, exa, brave-search, web_fetch

#### Sub-Agent 2G: Track Record & Wealth Analyzer

Dispatch a sub-agent to assess founder's past exits and financial position.

**Sub-agent prompt:** Read `./prompts/track-record-wealth.md` and pass it:
- Founder name(s), known companies
- Any Crunchbase or PitchBook data if available

**What it returns (structured JSON):**
- **Past exits:** companies founded → acquired/IPO'd, outcome size
- **Funding raised previously:** total capital raised across past ventures
- **Investor quality:** who backed them before? Tier 1 VCs? Angels?
- **Wealth signals** (publicly observable):
  - Serial entrepreneur with exits → likely has capital reserves
  - First-time founder with no prior exits → likely more hungry, higher hustle
  - Background at high-comp companies (FAANG, top finance) → savings buffer
  - Public angel investments or LP commitments → demonstrates wealth
  - Lifestyle signals from public social media (travel, real estate, etc.)
- **Propensity to grind assessment:**
  - First-time founder: ★★★★★ (high hunger, everything to prove)
  - Post-exit founder: ★★★☆☆ (proven but may lack edge, depends on exit size)
  - Corporate refugee: ★★★★☆ (high motivation to escape, may lack startup grit)
  - Repeat founder (no exit): ★★★★★ (persistent, battle-tested)
- **Financial runway:** can they self-fund? or completely dependent on raise?

**Tool access:** web_search, exa, brave-search, web_fetch

---

## Phase 3: Cross-Project Comparison (Main Agent, Inline)

**Only if multiple projects are being analyzed.**

After all per-project pipelines complete, the main agent synthesizes a comparison:

1. **Positioning Matrix:** Map all projects on key dimensions (traction, team, tech, moat)
2. **Relative Ranking:** Stack-rank projects by overall investment attractiveness
3. **Portfolio Fit Notes:** Which projects complement vs. cannibalize each other
4. **Conviction Tiers:**
   - 🟢 **High Conviction** — strong across multiple dimensions
   - 🟡 **Interesting / Monitor** — promising but gaps or early
   - 🔴 **Pass** — fundamental concerns

---

## Phase 4: Investment Memo Synthesis (Main Agent, Inline)

Compile everything into a structured investment memo for EACH project.

### Memo Format

```
═══════════════════════════════════════════════════════
INVESTMENT MEMO: [Project Name]
Prepared: [Date] | Analyst: Claude VC Analyst
═══════════════════════════════════════════════════════

▸ VERDICT: [🟢 INVEST / 🟡 MONITOR / 🔴 PASS]
▸ CONVICTION: [1-10]
▸ SECTOR: [e.g., DeFi, AI Infra, SaaS]
▸ STAGE: [💡 Idea / 🔧 Building / 🌱 Launched / 📈 Growth / 🏢 Mature]
▸ DATA QUALITY: [🟢 Rich / 🟡 Moderate / 🔴 Sparse]

⚠️ DATA COVERAGE NOTE:
[Explicitly state what data WAS and WAS NOT found. Example:]
[✅ Found: website, Twitter, GitHub, 2 press articles, Crunchbase profile]
[❌ Not found: revenue data, on-chain metrics, founder LinkedIn]
[📐 Estimated: user count (from social signals), team size (from job postings)]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
1. EXECUTIVE SUMMARY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[2-3 sentence thesis on why this is/isn't interesting]
[For early stage: "This is a [stage] assessment. Limited public data exists.
The thesis rests primarily on [what the thesis actually rests on]."]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
2. PRODUCT & STRATEGY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
What they build:
Product philosophy:
Roadmap:
Revenue model:
UX/Docs quality:
[For pre-product: replace UX/Docs with "Vision clarity" and "Whitepaper/thesis quality"]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
3. COMPETITIVE LANDSCAPE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Key competitors:
Competitive advantages:
Moat assessment:
Business model differentiation:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
4. TRACTION & METRICS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[For pre-product: "No traction data expected at this stage.
Early signals only: waitlist size, Twitter following, community interest."]
Social footprint: [✅ / 📐 / ❌ tag each metric]
On-chain metrics (if applicable): [✅ / 📐 / ❌ tag each metric]
Revenue / ARR:
Growth trajectory:
Red flags:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
5. TECHNOLOGY
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Tech stack:
GitHub activity:
Code quality:
Security posture:
Developer ecosystem:
[For pre-product with no GitHub: "No public code found. This is normal
for [stage]. Technical assessment deferred to data room / founder call."]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
6. FOUNDER & TEAM
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Background & credentials:
Founder-market fit:
Social influence & network:
Track record:
Wealth & hustle assessment:

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
7. RISKS & CONCERNS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Top 3-5 risks ranked by severity]
[ALWAYS include "Limited public information" as a risk for early-stage]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
8. BULL CASE / BEAR CASE
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
🐂 Bull: [Best case scenario and what drives it]
🐻 Bear: [Worst case scenario and what drives it]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
9. SCORECARD (calibrated to stage: [STAGE])
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Product Quality:     [█████░░░░░] 5/10  ← [stage context note]
Competitive Moat:    [███░░░░░░░] 3/10  ← [stage context note]
Traction:            [N/A — pre-product] OR [███████░░░] 7/10  ← [note]
Team & Founders:     [████████░░] 8/10  ← [stage context note]
Tech & Execution:    [██████░░░░] 6/10  ← [stage context note]
Market Timing:       [█████████░] 9/10  ← [stage context note]
Data Completeness:   [████░░░░░░] 4/10  ← [how much we actually know]
───────────────────────────────────
OVERALL:             [██████░░░░] 6.3/10

Note: Scores benchmarked against [stage] projects. Data
completeness of [X/10] means [explanation of gaps].

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
10. RECOMMENDATION & NEXT STEPS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
[Actionable next steps: schedule call, request data room, pass, etc.]
[Key questions to ask in founder meeting]
[For early stage: "Due to limited public data, the following
items should be verified in a founder call or data room:"]
[List specific unknowns that need verification]

═══════════════════════════════════════════════════════
```

---

## Sub-Agent Dispatch Guidelines

When dispatching sub-agents, follow these principles:

1. **Structured returns only.** Every sub-agent prompt ends with an explicit instruction to return JSON. The prompt files enforce this.

2. **Use all search tools in parallel.** Each sub-agent should fire web_search, exa, and brave-search simultaneously for each query to maximize coverage and minimize blind spots.

3. **Minimal tool access.** Sub-agents get read-only tools. No Write or Edit tools needed.

4. **Fail gracefully.** If a sub-agent encounters a 403, timeout, or block, it notes the failure in its return JSON and moves on. The main agent treats missing data as "source checked, nothing found" — not as an error.

5. **No nesting.** Sub-agents do not dispatch further sub-agents. They return structured JSON to the main agent.

6. **Context isolation.** Never pass the full conversation history to a sub-agent. Pass only the specific inputs it needs (project name, URL, founder name).

7. **Parallelism budget.** For a single project, dispatch up to 7 sub-agents in parallel (4 project + 3 founder). For batch requests, dispatch up to 2-3 project pipelines in parallel.

8. **Source attribution.** Every claim in the final memo should trace back to a specific sub-agent finding. Include source URLs where available.

---

## Ethical Considerations

- **Wealth assessment** is based only on publicly observable signals (past exits, public investments, career trajectory at known-compensation companies). Never speculate beyond what's publicly available.
- **Age estimation** is approximate and based on career timeline / graduation year. Frame as "career stage" rather than specific age.
- **Propensity to work hard** is a heuristic based on founder archetype patterns observed in VC. It's a signal, not a judgment — always caveat this.
- **Social media analysis** uses only public profiles. Do not attempt to access private accounts.
- Respect when information isn't findable — report what was searched and what came up empty.

---

## Tips for Edge Cases

- **Stealth-mode projects:** Limited public info. Lean heavily on founder analysis and whatever breadcrumbs exist (job postings, GitHub activity, trademark filings).
- **Pre-product projects:** No traction data. Focus on team, vision, and competitive landscape.
- **Anonymous/pseudonymous founders (crypto):** Skip personal background, focus on on-chain reputation, GitHub contributions, and community standing.
- **Non-English projects:** Sub-agents should include local-language search queries.
- **Multiple co-founders:** Dispatch Background Scout and Social Influence Scanner for each founder separately.

---

## Output Delivery

After synthesis, deliver the memo in TWO formats:
1. **In-chat:** Present the full memo directly in conversation using the format above
2. **As a file:** If the user wants a downloadable version, create a `.md` file in `/mnt/user-data/outputs/` and present it via `present_files`

For batch analysis of 3+ projects, always create a file — the output will be too long for comfortable in-chat reading.

---

## Error Handling

- If a URL times out or returns 403 → sub-agent notes it in return JSON, main agent marks source as "checked, blocked"
- If LinkedIn blocks access → social scanner falls back to search engine snippets about the person
- If no GitHub exists → skip tech deep dive sub-agent for code quality, focus on product/docs
- If all sources exhausted with no result for a dimension → report clearly, mark as "Insufficient Data"
- If a sub-agent hangs or returns malformed output → main agent skips it and proceeds with other results, noting the gap

---
> Source: [yoitsyoung/vc-analyst-skill](https://github.com/yoitsyoung/vc-analyst-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
