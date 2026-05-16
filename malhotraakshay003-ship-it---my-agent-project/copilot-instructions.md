## my-agent-project

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

This is a **Web Research Agent**. Its job is to:
1. Accept a research topic or question
2. Search the web for relevant information
3. Summarize findings into a structured markdown file
4. Self-verify the output for factual consistency and accuracy before finalizing

## Agent Behavior

### Research Flow
1. **Search** — use web search tools to gather sources on the topic
2. **Summarize** — synthesize findings into a markdown report saved to `output/`
3. **Verify** — re-read the output and cross-check key claims against the source material; flag or correct anything inconsistent

### Output Format
Research reports go in `output/<topic-slug>.md`. Each report should include:
- A `## Sources` section listing URLs used
- A `## Summary` section with the synthesized findings
- A `## Confidence` section noting any claims that could not be corroborated

### Self-Check Step
After writing the report, the agent must re-read it and compare key facts against retrieved source content. If a claim cannot be verified against a source, mark it with `> ⚠️ Unverified` in the report rather than silently including it.

## Tools Expected

- Web search (e.g., `WebSearch`) for querying the internet
- Web fetch (e.g., `WebFetch`) for reading page contents
- File write for saving reports to `output/`

## Key Constraints

- Do not fabricate sources — only cite URLs actually retrieved during the session
- If search results are thin or contradictory, say so explicitly in the `## Confidence` section rather than papering over it
- Keep summaries factual and neutral; avoid editorializing

---
> Source: [malhotraakshay003-ship-it/-my-agent-project](https://github.com/malhotraakshay003-ship-it/-my-agent-project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
