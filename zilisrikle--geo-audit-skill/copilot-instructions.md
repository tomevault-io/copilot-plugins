## geo-audit-skill

> A multi-agent GEO + SEO audit framework for Claude Code. Combines:

# geo-audit-skill

## What This Is

A multi-agent GEO + SEO audit framework for Claude Code. Combines:
- **Traditional SEO** — on-page signals, schema validation, PageSpeed (via Python scripts)
- **GEO** (Generative Engine Optimization) — AI citability scoring, AI crawler access, question gap mining, content engineering for AI extraction

Works for any website. Brand context comes from `brand.md` in the project root (create one per client or project).

## Agent Team

**You (the main Claude session) are the orchestrator.** Analyze tasks, delegate to specialists, ensure quality.

### Routing Table

| Task | Agent | When |
|------|-------|------|
| AI visibility, citability score, GEO audit | `geo-auditor` | Checking AI platform citations, scoring 6-dimension citability |
| Traditional SEO audit | `seo-auditor` | On-page SEO, crawlability, meta tags, PageSpeed |
| Question gap mining | `question-miner` | Finding questions AI answers poorly; competitor FAQ analysis |
| Content writing / rewriting for GEO | `content-engineer` | CITE framework content, FAQ sections, content briefs |
| Schema markup, JSON-LD | `schema-architect` | HowTo/FAQPage/Article/Organization schema generation |
| Quality review | `code-reviewer` | All content and markup before completion |

### Workflow Chains

- **SEO Audit**: you → `seo-auditor` → you (prioritize) → `schema-architect` or `content-engineer`
- **GEO Audit**: you → `geo-auditor` → you → `content-engineer` or `schema-architect`
- **Question Mining**: you → `question-miner` → `content-engineer` → `schema-architect` → `code-reviewer`
- **Full Sprint** (GEO + SEO): you → `seo-auditor` (baseline) + `geo-auditor` (parallel) → `content-engineer` + `schema-architect` → `code-reviewer`

### Orchestration Rules

1. Always delegate to the most specific agent — don't do specialist work yourself
2. Provide handoff context: (a) objective, (b) file paths or URLs, (c) acceptance criteria
3. Max 2 agents in parallel to avoid conflicts
4. `code-reviewer` is the quality gate for all content and schema changes

## Python Scripts

All deterministic checks run via scripts in `scripts/`. Run before any semantic analysis.

```bash
python3 scripts/check-site.py <url>        # robots.txt, sitemap, AI crawlers, llms.txt
python3 scripts/check-page.py <url>        # title, meta, H1, canonical, GEO signals
python3 scripts/check-schema.py <url>      # JSON-LD validation
python3 scripts/check-pagespeed.py <url>   # PageSpeed Insights API
python3 scripts/check-geo.py <url>         # 0-30 citability score (6 dimensions)
python3 scripts/fetch-page.py <url>        # raw HTML, redirect chain, SSRF protection
```

## Brand Context

Create a `brand.md` in the project root for each client/project. All agents will read it.

If no `brand.md` exists, agents infer context from the target site directly.

## GEO Strategy Framework

### Seven Pillars (Princeton GEO Research)

1. **Direct Response** — Lead every section with a clear, extractable answer
2. **Numerical Data** — Specific statistics, durations, percentages
3. **Authoritative Quotations** — Peer-reviewed studies, named experts
4. **Extractable Structure** — Clean H2/H3 hierarchy, FAQ sections, lists
5. **Original Expertise** — Proprietary data, unique frameworks, primary research
6. **Technical Infrastructure** — Schema markup, AI crawler access, llms.txt
7. **Content Freshness** — Regular updates with timestamps

### Citability Scoring (0-30)

| Dimension | Abbrev | What it measures |
|-----------|--------|-----------------|
| Direct Answer | DA | Section openers are standalone extractable answers |
| Data Density | DD | Specific numbers, durations, study results per page |
| Authority Signals | AS | Named researchers, institutions, citations |
| Structure Quality | SQ | Heading hierarchy, FAQ, lists, tables |
| Freshness | FR | datePublished, dateModified, visible dates |
| Uniqueness | UQ | Proprietary data, original frameworks, brand terms |

Grades: A (27-30), B (22-26), C (17-21), D (12-16), F (<12)

## File Organization

```
brand.md               # Brand context (create per project)
scripts/               # Python deterministic checks
.claude/agents/        # Specialist agents
.claude/skills/        # User-invocable skills
audits/                # Audit reports (auto-saved by agents)
content/               # GEO-optimized content drafts
schema/                # JSON-LD markup files
references/            # SEO/GEO field definitions and scoring rubrics
data/                  # AI crawler list, research data
```

## Skills

- `/seo-audit <url>` — Traditional SEO audit (on-page, technical, schema, pagespeed)
- `/geo-audit <url>` — AI visibility + citability audit
- `/content-optimizer` — Optimize content for AI citability using CITE framework
- `/schema-generator` — Generate JSON-LD for any page type

## Standards

### Content Files
- Markdown with YAML frontmatter (title, description, keywords, schema_type, last_updated)
- Max 150 words per section before a subheading
- Every section must start with a direct answer sentence

### Schema Markup
- JSON-LD format only — no Microdata or RDFa
- Validate with `python3 -m json.tool`
- All URLs absolute, all dates ISO 8601

### Naming
- Content: `kebab-case.md`
- Schema: `[page-slug]-schema.json`
- Audits: `geo-audit-[domain]-YYYY-MM-DD.md`, `seo-audit-[domain]-YYYY-MM-DD.md`

---
> Source: [zilisrikle/geo-audit-skill](https://github.com/zilisrikle/geo-audit-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
