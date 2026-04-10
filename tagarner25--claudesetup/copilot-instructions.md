## claudesetup

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This is a **reference and resource repository** for Claude AI tooling — not a software project. It contains no build system, tests, or runnable code. It is a knowledge base synced to Obsidian (note the `[[wikilink]]` syntax throughout) and organized around the Claude ecosystem.

## Repository Structure

```
ClaudeSetup/
├── README.md                          ← Master index (Obsidian MOC)
├── Cheat Sheets/                      ← Quick-reference guides
├── General Overview/                  ← Beginner-to-power-user setup & platform guides
│   ├── 1_Setup/                       ← Installation & configuration guides
│   ├── 2_Anthropic/                   ← Claude product deep-dives (models, tools, features)
│   │   ├── Claude Chat/
│   │   ├── Claude Code/
│   │   ├── Claude Computer Use/
│   │   ├── Claude Cowork/
│   │   ├── Claude Desktop/
│   │   ├── Claude For Excel/
│   │   ├── Claude For PowerPoint/
│   │   └── Claude Projects/
│   └── 3_Google/                      ← Google AI product overview (Gemini, etc.)
├── Hooks/                             ← Claude Code hook examples
│   └── Examples/
├── Plugins/                           ← Claude Code plugin examples
│   └── Examples/
├── Prompts/                           ← Prompt templates by domain
│   ├── Data Analysis/
│   ├── Design/                        ← Diagram generation, visual prompts
│   ├── Document Generation/
│   ├── Excel/                         ← Financial modelling prompts
│   ├── Finance/                       ← 20 institutional finance prompts (.txt)
│   └── PowerPoint/
├── Resources/                         ← Curated external link collections (by platform)
│   ├── Claude/
│   ├── Flow/
│   ├── Gemini/
│   ├── Obsidian/
│   └── Stitch/
├── Skills/                            ← Skill reference docs & prompt series
│   ├── 01_Document_Creation/ through 20_Plugin_Management/  ← 20 skill categories
│   ├── 21_D3js_Skills/                ← D3.js visualization skill (third-party)
│   ├── 22_EricksonGroup_BrandGuidelines/
│   ├── 23_GoogleStitch_Skills/        ← Google Stitch MCP skills
│   ├── Overview/                      ← What skills are & how they work
│   └── XX_ZipFolders/                 ← Packaged/installable skill bundles
└── Use Cases/                         ← Domain-specific workflow examples
    ├── Design/
    ├── Finance/
    ├── Operations/
    ├── Project Management/
    └── Software Development/
```

## Content Conventions

- **Obsidian wikilinks** (`[[filename]]`) are used throughout for internal navigation — these are not broken links, they're intentional Obsidian syntax.
- **YAML frontmatter** — all `.md` files use frontmatter with `title`, `description`, `created`, `updated`, and `topics` fields.
- **New files** — when creating content, always include YAML frontmatter. Use `README.md` as the index file for any new directory. Place prompts in `Prompts/<Domain>/`, use cases in `Use Cases/<Domain>/`, and skill docs in the appropriate `Skills/##_*` folder.
- **Skill categories** (`Skills/01_*` through `Skills/20_*`) are reference documentation only — they describe what installed skills do. Actual skill installation happens via Claude's plugin/Cowork interfaces.
- **Prompts** use `[BRACKETED FIELDS]` as fill-in-the-blank placeholders the user replaces before submitting to Claude.
- Date format in frontmatter: `MM/DD/YYYY` or `YYYY-MM-DD` (both are used; prefer `YYYY-MM-DD` for new content).

## Prompt Series

**Series A — Investment Analysis** (`Skills/skill_investment_analysis.md`): Goldman Sachs Stock Screener, DCF Valuation, Risk Analysis, Earnings Breakdown, Portfolio Construction, Technical Analysis, Dividend Strategy, Competitive Advantage, Pattern Recognition, Macro Impact.

**Series B — Personal Finance** (`Skills/skill_personal_finance.md`): Wealth Diagnostic, Retirement Planning, Portfolio Architecture, Tax Optimization, Debt Elimination, Emergency Fund/Cash Strategy, Insurance Audit, College Savings, Estate Planning, Real Estate Analysis.

Individual `.txt` versions of these prompts live in `Prompts/Finance/`.

## Skill Categories (23 total)

Categories 01–12 cover general business (Document Creation, Sales, Product Management, Legal, Operations, Brand Voice, Finance/Accounting, Data Analytics, Marketing, Productivity, Engineering, Design). Categories 13–20 are finance-specific (Equity Research, Financial Analysis, Investment Banking, Private Equity, Wealth Management, LSEG Fixed Income, S&P Global Market Intelligence, Plugin Management). Categories 21–23 are third-party integrations (D3.js, Erickson Group Brand Guidelines, Google Stitch).

## General Overview Content

`General Overview/2_Anthropic/README.md` is the main Anthropic Claude reference (models, features, comparisons). Each subdirectory under `2_Anthropic/` has a dedicated guide per Claude product.

## Environment

- **Platform**: Windows 11 — Claude Code uses bash shell (Unix syntax), but native tools use backslash paths.
- **No runnable code** — no `npm`, `pip`, or build commands apply. All work is content authoring/editing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TAGarner25)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/TAGarner25)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
