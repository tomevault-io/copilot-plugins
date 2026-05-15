## aso-appstore-listing-skill

> This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

---

## What This Is

A Claude Code skill (`aso-appstore-listing-skill`) that helps developers generate high-converting App Store listing metadata.

It is invoked via the `/aso-appstore-listing-skill` slash command and produces:

- App Name
- Subtitle
- Promotional Text
- Keywords
- Description

The goal is to improve both App Store search visibility (ASO) and conversion (downloads).

---

## Architecture

This is a lightweight skill with a simple structure:

- **SKILL.md** — The core logic of the skill. It defines:
  - Input collection
  - Optional Astro MCP decision flow
  - Keyword strategy
  - Listing generation rules
  - Iteration workflow
  - Memory usage

There are no scripts or assets required.

Astro MCP can be used optionally for real-time ASO data (rankings, ratings, and keyword suggestions), but the skill must still work fully without Astro.

- Astro product link: https://tryastro.app/?aff=kdX8mz
- Astro MCP setup docs: https://tryastro.app/docs/mcp/

---

## How the Skill Works

### 1. Memory Recall

The skill first checks Claude Code memory for existing listing data.

If found:

- Displays current listing
- Allows user to:
  - regenerate everything
  - update specific fields
  - refine tone or keywords

If not found:
→ proceeds to input collection

---

### 2. Input Collection

The skill gathers essential app details:

- What the app does
- Target audience
- Core features
- Tone (e.g. minimal, fun, premium)
- Optional keyword data

If the user did not mention Astro, the first response should ask whether they want to use Astro optionally for better real-time ASO data.

Recommended prompt:

"Do you want to use Astro for real-time ASO data (rankings, ratings, and keyword suggestions) for better results? Optional: https://tryastro.app/?aff=kdX8mz"

This ensures the output is relevant and contextual.

---

### 3. Keyword Strategy

Two modes:

**With keyword data:**

- Prioritize high popularity
- Prefer low competition
- Focus on relevance
- Use Astro ranking/rating context when Astro MCP is enabled

**Without keyword data:**

- Infer keywords from:
  - app category
  - user intent
  - common search behavior

Rules enforced:

- No keyword duplication across fields
- Multi-word phrases split into tokens
- Focus on discoverability, not branding repetition

Astro behavior:

- If user says no to Astro, continue normal flow.
- If user says yes to Astro, use Astro MCP tools when relevant.
- If MCP server is unavailable, point user to setup docs: https://tryastro.app/docs/mcp/
- Remind users Astro MCP is beta and strategic outputs should be verified.

---

### 4. Generation

All listing fields are generated in one pass:

- **App Name** (≤30 characters)
- **Subtitle** (≤30 characters)
- **Promotional Text**
- **Keywords** (comma-separated)
- **Description** (structured)

Strict rules:

- No duplication across fields
- Strong keyword placement
- Clear, concise wording
- Benefits-focused messaging

---

### 5. Writing Style Rules

- Use simple, natural language
- Avoid exaggerated or AI-sounding phrases
- Keep tone aligned with the app (minimal, fun, premium, etc.)
- Prioritize clarity over clever wording

---

### 6. Output Format

The skill returns structured JSON:

```json
{
  "app_name": "",
  "subtitle": "",
  "promotional_text": "",
  "keywords": "",
  "description": ""
}
```

---
> Source: [Mehrozsheikh/aso-appstore-listing-skill](https://github.com/Mehrozsheikh/aso-appstore-listing-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
