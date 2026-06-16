## promptito

> Each prompt is a Markdown file with YAML frontmatter.

# SKILL.md Format

Each prompt is a Markdown file with YAML frontmatter.

## Required Fields

```yaml
---
name: Prompt Name
description: One sentence description
version: 1.0.0
---
```

## Optional Fields

```yaml
---
category: engineering    # engineering, writing, design, data, other
tags: [code, review]    # Searchable tags
sfia:
  level: 3              # 1-5 (competency)
framework:
  type: risen           # risen, costar, or custom
qualityMetrics:
  accuracy: 0.9         # 0-1 score
guardrails:
  do: [allowed uses]
  dont: [disallowed uses]
ethics:
  humanAgency: "Human must approve"
author: Your Name
---
```

## Content

```markdown
---
name: Code Reviewer
---

# Role
You are a senior code reviewer...

# Instructions
- Check for bugs
- Verify tests pass
```

[Example](./public/prompts/code-reviewer/SKILL.md)

---
> Source: [jacotoledo/Promptito](https://github.com/jacotoledo/Promptito) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
