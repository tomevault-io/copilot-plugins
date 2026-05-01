## mermaid-compat

> Mermaid diagram compatibility guidelines for docs


# Mermaid compatibility (docs)

Different Mermaid renderers (GitHub, VS Code previews, docs sites) can be stricter than Mermaid itself. To avoid parse/render failures in project docs:

- Prefer ASCII in node labels (avoid curly quotes, arrows like `→`, ellipses like `…`, and emoji prefixes).
- Avoid Markdown formatting inside labels (especially backticks).
- Prefer `direction TB` inside `subgraph` blocks.
- Keep labels short; move detail to surrounding Markdown.

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
