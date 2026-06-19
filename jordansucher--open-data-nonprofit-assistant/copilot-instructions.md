## open-data-nonprofit-assistant

> >


# Nonprofit Data Strategy Skill

This skill guides user and Claude through a structured, interactive process: research the real org → map what they actually do → identify and search for relevant datasets (both top-down and bottom-up) → build a gap analysis → draft and optionally submit formal data requests → build the artifacts the user actually wants.

**The skill is a conversation, not a batch job.** After every phase, stop and share what you've produced, ask for feedback, and wait for the user to confirm before moving on. Do not proceed to the next phase without an explicit green light.

The skill has **two modes**:
- **Quick mode** (15-20 min): Phases 0–2 only. Good for workshops or an initial conversation. Produces a grounded picture of the org's work and a preliminary dataset map.
- **Deep mode** (1-2 hours): All phases. Includes live dataset searching, bottom-up portal discovery, gap analysis, data request drafting and submission, and artifact creation.

---

## Upfront: Orient, Then Ask Two Questions

Start by giving the user a brief overview of what this skill does and how it works — don't just jump into questions. Something like:

*"This skill walks through a structured process for building a data strategy for your nonprofit using NYC open data and other public sources. Here's the full arc:*

| Phase | What happens |
|-------|-------------|
| **0 — Research** | I research your org (website, 990, recent news) before we discuss anything, so the analysis is grounded in what you actually do |
| **1 — Operational map** | We identify your org's branches of work and the data question embedded in each — then you choose which areas to focus on |
| **2 — Dataset brainstorm** | For each area, I surface what datasets would be useful, whether they're public, whether they're on the NYC Open Data portal, and how feasible it would be to request anything that isn't |
| **3 — Live search + bottom-up discovery** | I actually search for each dataset, verify it exists, and browse the portal for datasets we didn't think to look for |
| **3.5 — Data quality audit** | I flag quality issues across the datasets found: gaps, staleness, methodological bias, proxy limitations |
| **4 — Gap analysis** | For each area: what can we build today, and what data is missing and should be sourced |
| **4.5 — Data requests** | I check whether missing datasets have already been requested from the NYC Open Data portal, draft new requests, and can submit them via the portal on your behalf |
| **5 — Artifact drafting** | I build the actual tools — targeting scorecards, advocacy maps, briefs, funder materials — saved as real files |
| **5.5 — Credibility review** | For each artifact, a memo surfacing where data quality issues or missing datasets could affect the conclusions |

*The full process takes 1-2 hours. A quick version (phases 0–2 only) takes about 15 minutes and gives you the operational map and dataset landscape — you can always continue from there."*

Then ask two questions:

**1. Which mode?**
Quick (phases 0–2 only, ~15 min) or deep (all phases, ~1-2 hours)?

**2. Is the Claude in Chrome browser extension set up?**
Several steps in this skill work much better with browser access — specifically:
- Actually retrieving datasets from the NYC Open Data portal to verify they exist and check their fields
- Discovering datasets by browsing the portal directly
- Submitting formal data requests via the NYC Open Data engage form

If the user isn't sure or says no, say: *"You can enable it in Claude settings → Desktop app → Computer use. If you'd rather skip it, that's fine — I'll note where functionality will be limited."* If they decline, proceed anyway and flag those steps as browser-dependent.

---

## Phase 0: Research the Organization

Before asking the user anything about their work, research the org yourself.

**Search for**:
- Their website (homepage, "about," "our work," "programs")
- Their most recent Form 990 via ProPublica Nonprofit Explorer (projects.propublica.org/nonprofits) — this often reveals programs and scale that aren't on the website
- Any recent news, reports, or publications

**Present your findings** in a short summary covering:
- Mission (quote their own language where possible)
- Main programs and branches of work — what they *actually do*, not just what category they fit
- Rough scale: budget, staff, geography served, populations served
- Any mentions of data, research, or metrics they already track

Then stop: *"Here's what I found about [org name]. Does this accurately reflect your work? Anything I've missed or gotten wrong?"*

Wait for confirmation before proceeding.

**Why this step matters**: Skipping it leads to generic operational maps that don't reflect how the org actually works. A housing org that only does legal services has completely different data needs than one that also does community organizing or manages affordable housing units. The 990 in particular reveals programs invisible on the website.

---

## Phase 1: Map Operational Areas, Then Scope

Based on your research (not generic categories), draft a specific map of what this org actually does. Use the org's own language for their programs and activities.

For each operational area, write one sentence describing the embedded data question — what decision or insight would better data unlock? This is more useful than just naming areas.

Common categories to draw from (adapt — don't copy verbatim):
- Program delivery / direct services
- Targeting & prioritization (who to focus on, where to direct resources)
- Advocacy & policy
- Research & evidence-building
- Grant writing & funder relations
- Communications & public narrative
- Partnerships & coalition-building
- Evaluation & impact measurement

Present the list and stop: *"Here are the operational areas I've mapped for [org]. Do these feel right? Anything to add, remove, or reframe?"*

Once the user confirms the list, don't just ask which areas to focus on — make a recommendation first. Based on what you learned in Phase 0 (the org's stated priorities, where their work seems most data-hungry, what their 990 suggests they're investing in), suggest 2-3 areas where data exploration is likely to have the most impact, and briefly explain why each one. Then ask: **"I'd suggest starting with these — but you know your org better than I do. Want to go with this, adjust the selection, or explore all areas?"**

Good reasons to prioritize an area: it drives decisions that affect resource allocation; it's something the org does at scale or wants to grow; it involves external audiences (funders, policymakers, community members) who would benefit from a data-backed argument; or the data landscape for it seems particularly rich based on what you already know.

Carry only the selected areas forward. Note any excluded areas briefly in case the user wants to revisit later.

---

## Phase 2: Dataset Brainstorm (Top-Down)

For each selected operational area, generate a list of datasets that would be useful. Cast a wide net — include datasets even if you're unsure they exist publicly. The goal is to surface the full universe before narrowing.

For each dataset, produce a table with these columns:

| Dataset | What it contains | Producer | Relevant areas | Publicly available? | On NYC Open Data portal? | If not public: feasibility of requesting |
|---------|-----------------|----------|---------------|--------------------|-----------------------|------------------------------------------|

- **Publicly available?**: ✓ Yes | ✗ No / restricted | ? Unclear
- **On NYC Open Data portal?**: ✓ Yes | ✗ No (public elsewhere) | ? Unclear — these are separate questions. A dataset can be public but hosted on a federal site, an agency page, or a third-party repository rather than data.cityofnewyork.us.
- **If not public: feasibility of requesting**: Only fill this in for datasets marked ✗ or ? on availability. Use one of: *Easy — request via NYC Open Data portal* | *Possible — FOIL request to [agency]* | *Hard — no clear public path; would require legislation, partnership, or data-sharing agreement* | *Unlikely — proprietary or sensitive data*. Add a brief note explaining why (e.g., "Agency collects this internally; precedent for release exists" or "Involves personally identifiable health data").

Save the table as `[org-slug]-dataset-brainstorm.md` in the user's working directory (or session directory if not in Cowork). Use `present_files` to surface it as a clickable file, then stop: *"Here's the dataset brainstorm — I've saved it as a file you can keep. Anything obviously missing? Anything irrelevant to cut?"*

**If in quick mode**, after the user confirms the dataset brainstorm, this is the natural stopping point. Close out with a summary of what continuing would look like, and offer to proceed:

*"That wraps up the quick version — you now have a map of [org name]'s operational areas and a preliminary view of what data exists. Here's what the deeper process would add if you wanted to continue:*

- *Phase 3 (~20 min): I'd actually search for each of these datasets, verify they exist, check their fields and freshness, and do a bottom-up browse of the NYC Open Data portal to surface anything we didn't think to look for.*
- *Phase 3.5 (~10 min): A quality audit flagging missing values, staleness, methodological bias, and join issues across the datasets I find.*
- *Phase 4 (~15 min): A gap analysis — for each operational area, what can be built with data that exists today, and what's missing in priority order.*
- *Phase 4.5 (~15 min): I'd check whether anyone has already requested the missing datasets from the NYC Open Data portal, draft formal requests for any that haven't been, and — if you have the browser extension set up — submit them on your behalf.*
- *Phase 5 (~30 min+): Drafting actual artifacts — targeting scorecards, advocacy maps, school briefs, funder materials — for whichever ones you want.*
- *Phase 5.5: A credibility memo for each artifact, surfacing where the data limitations could affect your conclusions.*

*Want to keep going now, or save the deeper dive for another session?"*

### NYC data sources to draw from

**NYC Open Data portal (data.cityofnewyork.us)**
- DOE: school demographics, enrollment, chronic absenteeism, Economic Need Index, school locations, capital plans, building space usage
- DOHMH: neighborhood health indicators, asthma ER rates, community air quality (NYCCAS), restaurant inspections, vital stats, lead poisoning
- NYPD: complaints, arrests, stop-and-frisk, summonses
- 311: service requests by type, geography, time
- DOB: building permits, violations, construction filings
- HPD: housing violations, landlord registration, alternative enforcement, eviction filings
- DCP: zoning, PLUTO (land use + building age + owner), census geographies
- Parks: inspections, maintenance, permits
- HRA/DSS: public assistance caseloads, shelter census
- DCAS: city-owned properties
- SCA: school construction projects, capital schedules
- NYC Checkbook: city contracts and expenditures

**NYC government (not on the Open Data portal)**
- DOE Space & Facilities Reports: lead paint inspections, building space usage
- DOE Building Ventilation Status: per-school pages, no bulk download
- BCAS (SCA Building Condition Surveys): per-school PDF assessments at survey.nycsca.org/bcas/
- NYC Housing Connect: affordable housing applications and lotteries

**Federal & state**
- ACS (American Community Survey): demographics, income, housing by census tract/block group
- HUD: fair market rents, LIHTC affordable housing properties, voucher data
- EPA EJScreen: environmental justice indices by census tract
- NYSED: school performance, graduation rates, assessments
- DOL / BLS: labor market, employment, wages by industry and geography
- CMS: Medicare/Medicaid enrollment and outcomes
- HMDA: home mortgage data by neighborhood
- ICE: enforcement data (aggregate only at field-office level; neighborhood detail mostly unavailable)

**Other useful sources**
- ProPublica Nonprofit Explorer: 990s for any US nonprofit
- PurpleAir: community air quality sensors (API available, free tier)
- NYC Court records / NYCDB (housingdatanyc.org): housing court filings, eviction proceedings
- OpenStreetMap: building footprints, points of interest

---

## Phase 3: Live Dataset Search + Bottom-Up Portal Discovery

*(Deep mode only)*

Run two parallel searches:

### 3A: Top-Down — Search for the Specific Datasets Identified in Phase 2

For each dataset in the Phase 2 table, search for it:
1. **NYC Open Data portal**: Search data.cityofnewyork.us by keyword. Note dataset name, ID, last updated, whether it has an API endpoint.
2. **Agency websites**: Check the relevant city or state agency directly.
3. **Federal portals**: census.gov, data.gov, epa.gov, bls.gov, hud.gov, etc.
4. **Other sources**: academic repositories, NGO data projects, APIs with free tiers.

For each, record: Found / Not found | URL | Dataset ID (if on Open Data) | Last updated | Format | Geographic granularity | Notes on access

### 3B: Bottom-Up — Browse the NYC Open Data Portal for Thematically Relevant Datasets

Don't just look for what you already imagined. Search the NYC Open Data catalog using keywords related to the org's topic area — you're looking for datasets that weren't in the Phase 2 brainstorm but that turn out to be relevant.

Try at least 3-5 different search terms related to the org's issue area. Browse category pages (e.g., "Health," "Housing & Development," "Education"). Look for datasets with recent update dates and meaningful geographic granularity.

For each unexpected dataset you find, note: what it contains, why it might be relevant, and which operational area it could serve.

Save the full results as `[org-slug]-dataset-inventory.md` in the user's working directory, organized by operational area with a separate section for "Unexpected finds from portal browsing." Use `present_files` to surface it, then stop: *"Here's the dataset inventory — saved as a file. Any surprises? Anything you want me to dig into further?"*

---

## Phase 3.5: Data Quality Audit

*(Deep mode only)*

For each dataset you found in Phase 3, surface any quality issues that could affect how much weight someone should put on it. This isn't about discarding datasets — it's about being honest with the user so they don't build arguments on shaky foundations.

For each dataset, flag any of the following that apply:

**Coverage gaps**
- Missing rows or time periods (e.g., dataset covers 2018–2021 but not 2022–2024)
- Geographic gaps (e.g., only covers certain boroughs, or uses a geography that doesn't match the org's service area)
- Population gaps (e.g., data only covers public schools, excluding charters or private schools)

**Staleness**
- When was it last updated? For time-sensitive work (air quality, housing market, crime), data more than 2-3 years old may tell a different story than current conditions
- Is there a regular update cadence, or is this a one-time dataset?

**Known methodological issues**
- Datasets that rely on self-reporting or complaint-driven collection are systematically biased toward places with higher reporting capacity (e.g., 311 data reflects where residents call 311, not necessarily where problems are worst)
- Administrative datasets (arrests, violations, housing court filings) reflect enforcement patterns as much as underlying conditions
- Survey data with small sample sizes at fine geographic levels will have high variance — confidence intervals matter

**Proxy reliability**
- If a dataset is being used as a proxy for something else (e.g., LL84 year_built as a proxy for HVAC condition), flag the assumptions embedded in that proxy and where they might break down

**Join and linkage issues**
- If combining two datasets requires a geographic or ID join, flag cases where the join will be incomplete or imprecise (e.g., school address matching to neighborhood boundary, BBL matching to school DBN)

Present a **Data Quality Summary** table:

| Dataset | Issue type | Description | Severity (Low/Med/High) | Implication for use |
|---------|-----------|-------------|------------------------|---------------------|

Keep flags honest but proportionate — note minor issues briefly, spend more time on anything that could materially affect conclusions. Append this table to `[org-slug]-dataset-inventory.md` under a "Data Quality Audit" heading, or save it separately as `[org-slug]-data-quality.md`. Use `present_files` to surface the updated file, then stop: *"Here are the quality flags — added to the saved file. Any of these surprises? Anything you want to investigate further?"*

---

## Phase 4: Gap Analysis

*(Deep mode only)*

For each selected operational area, produce two clearly separated lists:

**What we can build now** — specific, named artifacts using currently available data. Not "a dashboard" but "a ranked list of schools by composite need score using NYCCAS PM2.5, ENI, and LL84 building age" or "a council-district map showing number of affected students per district." Be concrete enough that the user can picture the output.

**What's missing** (in priority order) — data that doesn't exist publicly or isn't accessible, and would meaningfully improve the work. For each:
- What the dataset would contain
- Why it matters (what decision or artifact it unlocks)
- Whether it should come from NYC Open Data portal, another government process, FOIL, or a data partnership
- Rough effort: Low / Medium / High

Save as `[org-slug]-gap-analysis.md` in the user's working directory. Use `present_files` to surface it, then stop: *"Here's the gap analysis — saved as a file. Do the priorities feel right? Anything you'd reorder?"*

---

## Phase 4.5: Data Requests

*(Deep mode only)*

### Step 1: Check What's Already Been Requested

Before drafting new requests, search the NYC Open Data public requests log to see if anyone has already asked for similar data. The requests are publicly available as a dataset on the portal:

- **Dataset**: "NYC Open Data: Public Dataset Requests" (dataset ID: `63us-eqtq`)
- Search it via the Socrata API: `https://data.cityofnewyork.us/resource/63us-eqtq.json?$q=[search term]`

For each priority-1 missing dataset, search the requests log for relevant terms. Report back: *"Someone already requested [X] in [year] — here's the status"* or *"No existing request found for [X] — this would be new."*

This prevents duplicate requests and helps the user understand the landscape. Existing requests that haven't been fulfilled are worth noting — the user may want to add support to them rather than file separately.

### Step 2: Draft New Requests

For each high-priority missing dataset with no existing request (or where an existing request stalled), draft a formal request ready to submit:

```
DATASET REQUEST: [Dataset name]

Agency: [Which NYC agency owns or would own this data]
Dataset description: [What fields it would contain, geographic granularity, update frequency]
Why it's in the public interest: [Who would use it, what decisions it would inform]
Why this agency: [Why this agency should publish it, not another]
Existing partial sources: [What currently exists and why it's insufficient]
Requester context: [One sentence about the org and why they need this]
```

Keep each under 200 words. These are meant to be submittable, not just aspirational.

### Step 3: Offer to Submit via the Portal

Present the draft requests and ask: *"Would you like me to submit these to the NYC Open Data portal on your behalf? I can navigate to the engage form and fill them in — I'll show you each one before submitting."*

If yes, navigate to **https://opendata.cityofnewyork.us/engage/** using the browser extension and fill out and submit the form for each approved request. Show the user the completed form before hitting submit. Note: this requires the browser extension to be active.

Save the draft requests as `[org-slug]-data-requests.md` in the user's working directory and use `present_files` to surface it before offering to submit. If the browser extension isn't available, the saved file gives the user copy-paste-ready text to submit manually.

---

## Phase 5: Artifact Drafting

*(Deep mode only)*

After the gap analysis and data requests, present the full list of "buildable now" artifacts across all operational areas and ask explicitly:

*"Here's everything we could build with currently available data. Which of these would you like me to draft? You can name specific ones, say 'all,' or say 'none for now.'"*

Build only what the user selects. Work through them one at a time, checking in after each.

### Always create real files — never just paste into the response

Every artifact must be saved as an actual file, not rendered inline in the conversation. The user should be able to open, share, and reuse what's produced.

**Where to save:**
- If the user is in Cowork with a working directory mounted, save directly to that folder. Use the `present_files` tool to surface each file as a clickable card after saving.
- Otherwise, save to the session working directory and tell the user where the file is.

**What format to use:**
- Interactive visualizations, maps, scorecards → `.html` (self-contained, single file with inline CSS/JS)
- Briefs, memos, narrative documents → `.docx` using the `docx` skill for proper formatting, or `.md` if the user prefers
- Spreadsheet data, ranked lists → `.xlsx` using the `xlsx` skill
- Data request drafts → `.md` is fine

**If another skill handles the format better, invoke it.** The `docx` skill produces better Word documents than raw XML. The `xlsx` skill produces better spreadsheets. Don't reinvent what those skills already do well — invoke them for the actual file creation, then return here.

**What artifact drafting looks like:**
- **Targeting scorecard or ranked list**: pull real data from the Open Data API where possible; produce a sorted `.xlsx` or `.html` table with the top entries
- **Advocacy map or visualization**: a self-contained `.html` file with embedded data, charts, or a Leaflet map
- **School, neighborhood, or community brief**: a filled-in `.docx` for one real entity using their actual data — not a template with blanks
- **Grant application data section**: a `.docx` with the data narrative written out, ready to paste
- **Policy brief or one-pager**: a `.docx` with the argument made, citations included

Use real data where possible. A partial artifact with actual numbers for one school or neighborhood is more useful than a polished empty shell.

After saving each artifact and presenting the file link: *"Here's [artifact name] — does this feel useful? Anything to adjust?"* Then immediately run Phase 5.5 before moving to the next artifact.

---

## Phase 5.5: Credibility Gap Review

*(Deep mode only — run after each artifact, not just at the end)*

For each artifact drafted, produce a short **Credibility Memo** that surfaces how data limitations might affect the conclusions a reader would draw from it. This is not a disclaimer — it's a tool for the user to understand where their argument is strong and where it's vulnerable.

Structure each memo around two questions:

**1. Does the data used have quality issues that affect this artifact's conclusions?**

Pull from the Phase 3.5 quality audit. For each flag that's relevant to this specific artifact, explain concretely how it could affect the result:
- *"The ranking uses 2021 air quality data (NYCCAS). If pollution levels have shifted since then — which is plausible given post-COVID traffic changes — some schools' relative rankings could change."*
- *"The building age proxy (LL84 year_built) may misclassify schools that had major renovations after their original construction date, so some 'old' buildings may have newer HVAC systems than the score implies."*

**2. If the missing datasets had been available, could the conclusions be materially different?**

For each gap identified in Phase 4 that's relevant to this artifact, reason through the counterfactual:
- *"If per-school indoor air quality readings existed, some schools ranked low here (due to favorable outdoor air quality) might rank high because their indoor air is actually poor due to HVAC failures."*
- *"If school-level asthma outcomes were available, the targeting score might weight health burden more heavily, which could elevate schools in neighborhoods with older populations or more outdoor air pollution but lower economic need."*

Be direct about severity. Use language like "minor caveat," "meaningful limitation," or "could materially change conclusions" — not just "data limitations exist."

**End each memo with a recommended disclosure** — one or two sentences the user can include in the artifact itself or in any presentation of it to funders, policymakers, or the public. Good disclosures are specific, not boilerplate.

After each memo: *"Here's the credibility memo for [artifact name]. Anything here that changes how you want to frame or use the artifact?"*

---

## Output Documents

Every output is a real saved file — never just text in the response. In Cowork, save to the user's mounted working directory and use `present_files` to surface each one. Outside Cowork, save to the session directory and share the path.

Use a consistent naming convention so files are easy to find:

| Phase | File | Format |
|-------|------|--------|
| 2 | `[org-slug]-dataset-brainstorm.md` | Markdown table with all seven columns |
| 3 | `[org-slug]-dataset-inventory.md` | Markdown table, one section per operational area + unexpected finds |
| 3.5 | Appended to dataset inventory under "Data Quality Audit", or `[org-slug]-data-quality.md` | Markdown table |
| 4 | `[org-slug]-gap-analysis.md` | Markdown, buildable now / missing by area |
| 4.5 | `[org-slug]-data-requests.md` | Markdown, formatted for copy-paste submission |
| 5 | One file per artifact — see format guide in Phase 5 | `.html`, `.docx`, `.xlsx`, or `.md` |
| 5.5 | Appended to each artifact file, or `[artifact-slug]-credibility-memo.md` | Markdown |

The `[org-slug]` is a short lowercase hyphenated version of the org name (e.g., `clean-air-schools`, `make-the-road-ny`).

---

## Reference Example

`references/cas-example.md` shows all five phases applied to Clean Air for Schools Fund — real datasets found and not found in NYC, a gap analysis, drafted data requests, and a school-targeting scorecard. Read it when:
- The user wants to see what the output looks like
- The org works in air quality, school health, or environmental justice
- You need to calibrate the depth and specificity expected in each phase

---

## The Most Important Things

**Ground everything in the real org.** The research step is not optional. Operational maps built from website research and 990s are worth 10x those built from generic assumptions. If the user doesn't name their org, ask.

**Turn-taking is the point.** The reason this is a skill and not a prompt is that a human needs to catch errors, redirect effort, and make judgment calls at each step. A skill that barrels through all phases autonomously produces polished but wrong output — which is worse than no output.

**The bottom-up portal search often produces the best finds.** NYC Open Data has thousands of datasets. The ones that turn out to be most useful are often not the ones you'd think to search for. Always do the bottom-up browse.

**The data request step is uniquely valuable.** Most nonprofits have never thought systematically about what data *should* exist. Helping them articulate that clearly — and actually submit a request — is something few consultants ever do for them.

**The quality audit and credibility memos protect the user.** Nonprofits sometimes use data to make funding or policy arguments, and conclusions built on shaky data can backfire badly — especially in adversarial contexts like council testimony or grant competition. The point of Phase 3.5 and 5.5 is not to undermine the work but to help the user know exactly where their argument is strong and where they need to hedge or caveat. An artifact with an honest credibility memo is more trustworthy, not less.

---
> Source: [JordanSucher/open-data-nonprofit-assistant](https://github.com/JordanSucher/open-data-nonprofit-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
