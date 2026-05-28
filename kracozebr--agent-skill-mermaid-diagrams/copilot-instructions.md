## agent-skill-mermaid-diagrams

> This repository packages a portable Agent Skill for turning plain text into Mermaid diagrams and inserting them into Markdown files.

# Repository Instructions

This repository packages a portable Agent Skill for turning plain text into Mermaid diagrams and inserting them into Markdown files.

## Development Rules

- Keep `skills/insert-mermaid-diagrams/SKILL.md` portable. Its frontmatter should contain only `name` and `description`.
- Keep OpenAI/Codex-specific UI metadata in `skills/insert-mermaid-diagrams/agents/openai.yaml`.
- Keep helper scripts dependency-free unless there is a concrete reason to add packaging.
- Preserve cross-agent install paths for Codex, Claude Code, Cursor, and generic `.agents/skills` users.
- Before finishing changes, run:

```powershell
python -m py_compile .\skills\insert-mermaid-diagrams\scripts\update_mermaid_block.py
python -m unittest discover -s tests
python C:\Users\RND\.codex\skills\.system\skill-creator\scripts\quick_validate.py .\skills\insert-mermaid-diagrams
```

---
> Source: [Kracozebr/agent-skill-mermaid-diagrams](https://github.com/Kracozebr/agent-skill-mermaid-diagrams) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
