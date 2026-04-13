## lead-research-agent

> An AI agent that automatically finds, qualifies, and drafts personalized outreach for ideal client prospects. Built as a portfolio project to demonstrate AI automation on Upwork.

# Lead Research Agent

An AI agent that automatically finds, qualifies, and drafts personalized outreach for ideal client prospects. Built as a portfolio project to demonstrate AI automation on Upwork.

## What This Agent Does

1. **Find** — Pull a list of target companies or contacts from a data source (e.g. Apollo.io free tier, Google search scrape, or a manual CSV)
2. **Qualify** — Score each lead against an ICP (Ideal Client Profile) using Claude
3. **Draft** — Generate personalized outreach emails/DMs per lead

## Stack

- **Python** — all tools live in `tools/`
- **Claude API** (Anthropic) — qualification scoring + draft generation (use `claude-haiku-4-5-20251001` for bulk ops, Sonnet for drafts)
- **Google Sheets** — lead storage and output CRM (via `gspread`)
- **`.env`** — all credentials (ANTHROPIC_API_KEY, Google OAuth, etc.)

## Project Structure

```
tools/          # Python scripts — one file per task
workflows/      # Markdown SOPs — one file per workflow
.env            # API keys (never commit)
credentials.json, token.json  # Google OAuth (gitignored)
output/         # Generated drafts and qualified lead exports
```

## ICP (Ideal Client Profile)

This agent is built to find clients for an **AI agent developer**. Default ICP:
- Small business owners or operators (10-100 employees)
- Using manual processes that can be automated (lead gen, onboarding, reporting)
- Industries: SaaS, agencies, e-commerce, professional services
- Budget signals: hiring for ops/admin roles, posting on Upwork

Adjust ICP in `workflows/qualify.md` as the use case evolves.

## WAT Architecture

Follow the same pattern as the parent project:
- **Workflows** — define the SOP in Markdown before writing any code
- **Tools** — one Python script per discrete task (find, qualify, draft)
- **Agent (Claude)** — reads workflows, sequences tools, handles errors

## Build Order

1. `workflows/find.md` + `tools/find_leads.py` — pull raw leads
2. `workflows/qualify.md` + `tools/qualify_leads.py` — score against ICP
3. `workflows/draft.md` + `tools/draft_outreach.py` — generate outreach copy
4. `workflows/pipeline.md` — end-to-end orchestration SOP

## Output

- Qualified leads written to a Google Sheet (columns: name, company, score, reasoning, draft)
- Drafts ready to send or review before sending

## Decision Log

Log key decisions in `decisions/log.md`.
Format: `[YYYY-MM-DD] DECISION: ... | REASONING: ... | CONTEXT: ...`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/theatosam)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/theatosam)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
