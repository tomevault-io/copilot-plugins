## aeo-audit

> A multi-agent rig that generates AEO (Answer Engine Optimization) and SEO audit reports for businesses. Runs in two modes:

# AEO & SEO Audit -- AI Search & SEO Audit Generator Rig

## What This Is

A multi-agent rig that generates AEO (Answer Engine Optimization) and SEO audit reports for businesses. Runs in two modes:

- **Light mode**: Quick research + AEO snapshot + cold outreach email. Costs fewer tokens, produces a cold-outreach asset for initial prospecting. Run this first to see if the prospect bites.
- **Full mode**: Complete 25-35 page branded report, executive summary, cover letter, and follow-up email sequence. Run this when a prospect responds to a light-mode outreach, or when you want the full deliverable package upfront.

A member provides a business URL, competitor URLs (or auto-find), and target location. The rig crawls the site, calls free APIs for real metrics (PageSpeed, TLS/SSL, W3C, WHOIS, Wayback), performs a deep AEO analysis, compares competitors, and produces the selected deliverable package.

**AEO is the lead, not a section.** The AEO assessment is the first thing the prospect sees, the executive summary leads with it, and the cover letter opens with it. This is what makes the audit stand out in a world of commodity SEO audits.

Members use the output to close $2,000-$3,500 deals with the "did the work first" motion: run the audit without asking, send the executive summary cold, close when they ask "can you fix this?"

## Quick Start

- `/setup` -- Configure your branding, niche, pricing, and optional PageSpeed API key (run once)
- `/run-audit` -- Generate an AEO & SEO audit (you'll choose light or full mode)

## The Agent Team

| Agent | Role | When to Use |
|-------|------|-------------|
| **Web Crawler** | Crawls target URL + key pages via WebFetch for HTML analysis, meta tags, schema, headings | Phase 2 of /run-audit |
| **API Caller** | Calls 5 free APIs (PageSpeed, SSL Labs, W3C, Wayback, WHOIS) via Bash curl for real metrics | Phase 2 of /run-audit (parallel with Web Crawler) |
| **Competitor Researcher** | Finds and analyzes 2-3 competitors | Phase 2 of /run-audit (parallel with Web Crawler) |
| **AEO Analyst** | Deep-dive AI search readiness assessment -- the star analysis | Phase 2.5 of /run-audit |
| **SEO Strategist** | Synthesizes all research into scoring framework, keyword gaps, action plan | Phase 3 of /run-audit |
| **Report Writer** | Writes assigned report section (spawned 5x for 5 sections) | Phase 4 of /run-audit |
| **Report Designer** | Creates visual design brief for branded PDF report | Phase 4.5 of /run-audit |
| **Report Builder** | Builds report.html, executive-summary.html, follow-up emails | Phase 5 of /run-audit |
| **Design Reviewer** | Reviews HTML deliverables for layout quality, responsiveness, typography, print rendering | Phase 5.7 of /run-audit |
| **Cover Letter Writer** | Personalized outreach letter leading with AEO findings | Phase 5.5 of /run-audit |
| **Quality Reviewer** | **GATE**: Validates technical accuracy, API data usage, AEO depth, completeness, specificity | Phase 6 of /run-audit (mandatory) |

## Context Engineering (CRITICAL)

### Rules for the Orchestrator (YOU)

1. **NEVER read skill files yourself.** Skills are for agents. They are at `.claude/skills/*/SKILL.md`.
2. **NEVER read agent definition files yourself.** Pass the agent file path in the Task prompt so the agent reads its own instructions. They are at `.claude/agents/*.md`.
3. **Pass file PATHS, not file CONTENTS** to agents. Tell the agent: "Read the file at `<path>`."
4. **Agents write directly to disk.** You do NOT receive generated content back.
5. **Track status, not content.** After an agent finishes, you need: (a) status, (b) file paths, (c) issues. NOT file contents.
6. **Quality Reviewer reads from disk.** Pass it the output directory path.
7. **For report writing, spawn ONE Report Writer per section.** Each spawn writes one file.
8. **Use max_turns on Task calls.** Web Crawler: 18, API Caller: 15, Competitor Researcher: 12, AEO Analyst: 15, SEO Strategist: 15, Report Writers: 20, Report Designer: 15, Report Builder: 30, Design Reviewer: 15, Cover Letter Writer: 10, Quality Reviewer: 25.

### How to Spawn Agents (Context-Safe Pattern)

CORRECT -- agent reads its own files, writes to disk:
```
Task tool:
  subagent_type: "general-purpose"
  prompt: |
    You are the [Agent Name] for the AEO & SEO Audit.
    Read your full instructions at: .claude/agents/<agent>.md

    ## Your Task
    [Brief task description]

    ## Input Files (read these yourself)
    - Plan: output/<client-slug>/plan.md
    - [Other inputs]: <path>

    ## Output Files (write these yourself)
    - <path-to-output>

    ## Return Format
    Return ONLY:
    - Status: SUCCESS or FAILED
    - Files created: [list of paths]
    - Issues: [any problems encountered]
```

WRONG -- orchestrator reads everything and pastes it in:
```
Read .claude/agents/<agent>.md          <- wastes orchestrator context
Read .claude/skills/<skill>/SKILL.md    <- wastes orchestrator context
Task tool with all that content pasted  <- doubled context usage
```

## Workflow: /run-audit

### Phase 1: Input & Planning (MANDATORY)
1. Read `config/member-profile.md` for member branding/niche and optional API key
2. Collect: business URL, competitor URLs (or "auto-find"), target location, prospect logo URL (optional)
3. **Ask: "Light mode or full mode?"**
   - **Light**: Quick research + AEO snapshot + cold email. Fewer tokens, fast outreach asset.
   - **Full**: Complete 25-35 page report + executive summary + cover letter + emails.
4. Create client slug, output directory with subdirs: `research/`, `strategy/`, `report/`, `design/`, `sales/`, `deliverables/`
5. Present audit plan (including mode), wait for user approval
6. Save plan to `output/<client-slug>/plan.md` (include `mode: light` or `mode: full`)

### Phase 2: Research (all 3 in parallel)
6. Spawn **Web Crawler** (max_turns: 18) -- crawls URL via WebFetch -> site-crawl.md
7. Spawn **API Caller** (max_turns: 15) -- calls 5 APIs via Bash curl -> tool-data.md
8. Spawn **Competitor Researcher** in parallel (max_turns: 12) -> competitor-analysis.md
9. Verify `research/site-crawl.md`, `research/tool-data.md`, `research/competitor-analysis.md` exist
10. For any missing research file: re-spawn the responsible agent with max_turns: 10 and focused prompt. If still missing after retry: create a minimal placeholder noting the gap and proceed.

### Phase 2.5: AEO Analysis
9. Spawn **AEO Analyst** (max_turns: 15) -- deep-dive AI search readiness
10. Verify `research/aeo-analysis.md` exists

### Mode Branch
- If `mode: light` → skip to **Phase L: Light Output**
- If `mode: full` → continue to Phase 3

### Phase 3: Strategy
11. Spawn **SEO Strategist** (max_turns: 15) -- synthesizes all research
12. Verify `strategy/audit-framework.md` exists

### Phase 4: Report Writing
**Group A (parallel):**
13. Spawn **Report Writer** for `report/technical-audit.md` (max_turns: 20)
14. Spawn **Report Writer** for `report/content-analysis.md` (max_turns: 20)
15. Verify Group A files exist

**Group B (parallel, after A):**
16. Spawn **Report Writer** for `report/aeo-assessment.md` (max_turns: 20)
17. Spawn **Report Writer** for `report/local-seo-scorecard.md` (max_turns: 20)
18. Verify Group B files exist

**Group C (after A+B):**
19. Spawn **Report Writer** for `report/competitor-comparison.md` (max_turns: 20)
20. Verify all 5 report files exist

### Phase 4.5: Design
21. Spawn **Report Designer** (max_turns: 15)
22. Verify `design/report-brief.md` exists

### Phase 5: Report Build
23. Spawn **Report Builder** (max_turns: 30) -- builds follow-up-emails.md, executive-summary.html, report.html (smallest first)
24. Verify all deliverable files exist

### Phase 5.5: Cover Letter
25. Spawn **Cover Letter Writer** (max_turns: 10)
26. Verify `sales/cover-letter.md` exists

### Phase 5.7: Design Review
27. Spawn **Design Reviewer** (max_turns: 15)
28. If NEEDS FIXES: re-spawn **Report Builder** with design feedback, max 1 iteration

### Phase 6: Quality Gate (MANDATORY)
29. Spawn **Quality Reviewer** (model: opus, max_turns: 25)
30. If APPROVED: proceed
31. If BLOCKED: re-spawn relevant agent(s) with QAS feedback, max 2 iterations

### Phase L: Light Output (light mode only)

Skip here from Mode Branch after Phase 2.5. Produces 2 files for quick cold outreach.

L1. Spawn a **general-purpose** agent (max_turns: 15) to write `sales/aeo-snapshot.md`:
```
Task tool:
  description: "Write AEO snapshot for light mode"
  subagent_type: "general-purpose"
  max_turns: 15
  prompt: |
    You are writing a concise AEO snapshot for cold outreach.

    ## Your Task
    Read the research files and produce a punchy 800-word-max AEO snapshot summarizing:
    - AI Search Readiness Score (1-10) with brief justification
    - Top 3 AEO findings (most attention-grabbing)
    - Top 3 quick wins the business could implement
    - One competitor comparison highlight

    ## Input Files (read these yourself)
    - Plan: output/<client-slug>/plan.md
    - Site Crawl: output/<client-slug>/research/site-crawl.md
    - Tool Data: output/<client-slug>/research/tool-data.md
    - Competitor Analysis: output/<client-slug>/research/competitor-analysis.md
    - AEO Analysis: output/<client-slug>/research/aeo-analysis.md

    ## Output Files (write these yourself)
    - output/<client-slug>/sales/aeo-snapshot.md

    ## Format
    Use this structure:
    # AEO Snapshot: [Business Name]
    ## AI Search Readiness: [Score]/10
    [2-3 sentence summary]
    ## Top Findings
    1. [Finding with specific evidence]
    2. [Finding with specific evidence]
    3. [Finding with specific evidence]
    ## Quick Wins
    1. [Actionable recommendation]
    2. [Actionable recommendation]
    3. [Actionable recommendation]
    ## vs. Competitors
    [One key competitive insight]
```

L2. Spawn **Cover Letter Writer** (max_turns: 10) to write `sales/cold-email.md`:
```
Task tool:
  description: "Write cold outreach email"
  subagent_type: "general-purpose"
  max_turns: 10
  prompt: |
    You are the Cover Letter Writer for the AEO & SEO Audit (light mode).
    Read your full instructions at: .claude/agents/cover-letter-writer.md

    ## Your Task
    Write a cold outreach email (150-250 words) with 3 subject lines. Lead with the single strongest AEO finding.
    This is for light mode -- the email references the AEO snapshot, not a full report.

    ## IMPORTANT DIFFERENCE FROM FULL MODE
    - Instead of "I put together a complete audit", say "I ran a quick AEO check on your business"
    - Instead of referencing a 25-page report, reference the snapshot
    - CTA: "I can put together a full analysis if you're interested"

    ## Input Files (read these yourself)
    - Member Profile: config/member-profile.md
    - Plan: output/<client-slug>/plan.md
    - AEO Analysis: output/<client-slug>/research/aeo-analysis.md

    ## Output Files (write these yourself)
    - output/<client-slug>/sales/cold-email.md
```

L3. Verify `sales/aeo-snapshot.md` and `sales/cold-email.md` exist.

L4. Generate `output/<client-slug>/README.md`:
```
Light Mode Audit: [Business Name]
Files: sales/aeo-snapshot.md, sales/cold-email.md
Research data saved in research/ for full mode upgrade.

Next steps:
1. Review the cold email and AEO snapshot
2. Send the cold email to the prospect
3. If they respond, run /run-audit in full mode -- research data will be reused
```

L5. Report results and stop (do NOT continue to Phase 3+).

### Phase 7: Output
32. Verify all files exist
33. Generate `output/<client-slug>/README.md`
34. Report results with next steps

## Quality Gate Checklist (Quality Reviewer Owns This)

### AEO Assessment Quality (Priority Check)
- [ ] AI search readiness score is present and justified
- [ ] Structured data assessment matches actual schema found on site
- [ ] Answer-ready content evaluation references actual page content
- [ ] Entity authority assessment is evidence-based
- [ ] AEO recommendations are specific, actionable, and prioritized
- [ ] AEO section is substantial (6-8 pages)

### Technical Accuracy (API Data)
- [ ] PageSpeed scores match actual API response in tool-data.md
- [ ] SSL grade matches SSL Labs response
- [ ] W3C validation results reference actual errors
- [ ] WHOIS/domain data matches actual lookup
- [ ] tool-data.md contains raw API data supporting claims

### Completeness
- [ ] All sections present: AEO, technical, content, local, competitor, action plan
- [ ] Executive summary leads with AEO score
- [ ] Action plan has 10+ recommendations, AEO actions highlighted
- [ ] Scores consistent across all files

### Specificity
- [ ] Every finding references the specific client website
- [ ] No section could be copy-pasted to a different business
- [ ] Follow-up emails reference specific audit data
- [ ] Cover letter leads with specific AEO findings

### Deliverables
- [ ] report.html renders correctly, print-optimized
- [ ] executive-summary.html fits one page, leads with AEO
- [ ] Member branding applied
- [ ] Cover letter is 150-250 words with 3 subject lines, single finding focus, no pricing

## Output Structure

```
output/<client-slug>/
  README.md                          # Table of contents + delivery checklist
  plan.md                            # Approved audit plan
  research/
    site-crawl.md                    # Manual crawl analysis
    tool-data.md                     # Raw API data
    competitor-analysis.md           # Competitor research
    aeo-analysis.md                  # AEO deep-dive
  strategy/
    audit-framework.md               # Scores, gaps, action plan
  report/
    aeo-assessment.md                # AEO Assessment (THE LEAD)
    technical-audit.md               # Technical SEO
    content-analysis.md              # On-Page Content
    local-seo-scorecard.md           # Local SEO
    competitor-comparison.md         # Competitor Comparison
  design/
    report-brief.md                  # Visual design brief
  sales/
    cover-letter.md                  # Outreach letter (AEO-led)
    follow-up-emails.md              # 3-email sequence
  deliverables/
    report.html                      # Full branded report (print > PDF)
    executive-summary.html           # One-page summary (print > PDF)
```

### Light Mode Output
```
output/<client-slug>/
  plan.md                            # Audit plan (mode: light)
  research/
    site-crawl.md                    # Manual crawl analysis
    tool-data.md                     # Raw API data
    competitor-analysis.md           # Competitor research
    aeo-analysis.md                  # AEO deep-dive
  sales/
    aeo-snapshot.md                  # 800-word AEO summary for outreach
    cold-email.md                    # Cold outreach email + 3 subject lines
```

## Skills (Domain Knowledge)

| Skill | Location | Used By |
|-------|----------|---------|
| Technical SEO | `.claude/skills/technical-seo/SKILL.md` | Web Crawler, API Caller, Report Writer |
| Content SEO | `.claude/skills/content-seo/SKILL.md` | Report Writer, SEO Strategist |
| AEO Optimization | `.claude/skills/aeo-optimization/SKILL.md` | AEO Analyst, Report Writer, SEO Strategist |
| Local SEO | `.claude/skills/local-seo/SKILL.md` | Report Writer, Competitor Researcher |
| SEO Report Structure | `.claude/skills/seo-report-structure/SKILL.md` | Report Writer, Report Designer, Report Builder, SEO Strategist |
| Audit Outreach | `.claude/skills/audit-outreach/SKILL.md` | Cover Letter Writer, Report Builder |
| QAS Checklist | `.claude/skills/qas-checklist/SKILL.md` | Quality Reviewer (ONLY skill the QAS reads) |

## File Reference

| What | Where |
|------|-------|
| This file (orchestration hub) | `CLAUDE.md` |
| Commands | `.claude/commands/run-audit.md`, `setup.md` |
| Agents | `.claude/agents/web-crawler.md`, `api-caller.md`, `competitor-researcher.md`, `aeo-analyst.md`, `seo-strategist.md`, `report-writer.md`, `report-designer.md`, `report-builder.md`, `design-reviewer.md`, `cover-letter-writer.md`, `quality-reviewer.md` |
| Skills | `.claude/skills/technical-seo/`, `content-seo/`, `aeo-optimization/`, `local-seo/`, `seo-report-structure/`, `audit-outreach/`, `qas-checklist/` |
| Member config | `config/member-profile.md` |
| Output | `output/<client-slug>/` |

---
> Source: [AI-Captains-Academy/aeo-audit](https://github.com/AI-Captains-Academy/aeo-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
