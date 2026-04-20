## claude-code-synopsis

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a **product research workspace** for running structured, data-driven research projects. Claude operates here as a stateful research and synthesis engine, not a chatbot.

## Key Workflows

### Starting a New Project (`/project` action)

1. Ask for a kebab-case project name (e.g., `reduce-churn`, `new-pricing-eval`)
2. Create the folder structure:
   ```
   /projects/<project-name>/
     project.md
     /data
     /artifacts
   ```
3. Run a discovery interview covering:
   - Problem/opportunity definition
   - Products in scope
   - Success metrics
   - Constraints (time, people, tech, risk)
   - Stakeholders and target audience
4. Execute research phase:
   - Read all `/context` materials first
   - Query **Synopsis MCP server** for [UPDATE WITH YOUR SYSTEMS]
   - Cross-validate insights and flag data gaps
5. Write durable artifacts to disk

## Repository Structure

- `/context/` - Authoritative, slow-changing context (product overviews, strategic goals, user segments). Read this **before** starting any project.
- `/projects/` - Individual research initiatives, each with `project.md`, `/data`, and `/artifacts`

## Data Sources via Synopsis MCP

- UPDATE THESE SOURCES TO MATCH YOUR SYSTEMS

## Output Standards

`project.md` must include:
- Problem statement and why it matters now
- Evidence summary with Synopsis data
- Insights and opportunity framing
- Recommendations with tradeoffs/risks
- Open questions and data gaps

`/artifacts/` contains decision-ready outputs: PRDs, opportunity briefs, roadmap proposals, executive updates.

## Operating Principles

- Clarify intent before researching
- Ground claims in Synopsis data; label hypotheses explicitly
- Separate facts, interpretations, and recommendations
- Write artifacts as durable documents, not ephemeral answers
- Think in terms of tradeoffs, sequencing, and organizational reality

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/synopsis-ai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
