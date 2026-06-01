## talks-formatting

> When working with files in the `docs/talks/` directory, follow these formatting standards:

# Talks Formatting Standards

When working with files in the `docs/talks/` directory, follow these formatting standards:

## Title Format
All talk titles must follow a **catchy, conversational format** with these principles:
- Use conversational tone with "I", "You", "Why", "How" to make it personal
- Include specific benefits, numbers, or percentages when possible
- Reference the company/organization for credibility
- Use controversial hooks that challenge conventional wisdom
- Suggest actionable implications for readers

### Title Pattern Examples:
- "Why I Stopped Using RAG for Coding Agents (And You Should Too)"
- "The RAG Mistakes That Are Killing Your AI (Lessons from Google & LinkedIn)"
- "Stop Trusting MTEB Rankings - Here's How Chroma Actually Tests Embeddings"
- "The 12% RAG Performance Boost You're Missing (LanceDB's Re-ranking Secrets)"

## YAML Frontmatter Requirements
```yaml
---
title: [Catchy conversational title]
speaker: [Name, Company]
cohort: [Number]
description: [Brief description of key insights]
tags: [relevant, tags, including, company, technology]
date: [YYYY-MM-DD]
---
```

## Content Structure
- **H1 title**: Must match YAML frontmatter title exactly
- **Main sections**: Use `##` for primary sections
- **Subsections**: Use `###` for subsections
- **Key takeaways**: Use `**Key Takeaway:**` format for main insights
- **Quotes**: Use `>` blockquotes for speaker quotes
- **Lists**: Use `-` with **bold** labels for bullet points
- **Company attribution**: Always include company names in titles and content

## Writing Style
- 9th-grade reading level for accessibility
- Technical depth with practical examples
- Actionable insights over theoretical concepts
- Conversational tone that matches catchy titles
- Specific metrics and performance improvements noted

Reference [AGENTS.md](mdc:docs/talks/AGENTS.md) for complete formatting guidelines.

---
> Source: [jxnl/systematically-improving-rag](https://github.com/jxnl/systematically-improving-rag) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
