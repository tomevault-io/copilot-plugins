## codex-paper-summerize-to-notion

> When the user asks to set up `paper-summarize-to-notion` in this project, follow `SETUP_AGENT.md`.

# Codex Setup Instructions

When the user asks to set up `paper-summarize-to-notion` in this project, follow `SETUP_AGENT.md`.

Expected trigger phrases include:

- `paper-summarize-to-notion のセットアップを始めて`
- `このPJTで論文要約Notion登録のセットアップをして`
- `このプロジェクトに論文PDF要約Skillを導入して`

During setup:

- Install the bundled skill from `skills/paper-summarize-to-notion` into `$CODEX_HOME/skills/paper-summarize-to-notion` if it is not already installed.
- Guide the user through GitHub account/repository preparation.
- Create or guide creation of the GitHub repository when tooling allows.
- Require Notion MCP and direct the user to connect `https://mcp.notion.com/sse`.
- Create or guide creation of the Notion database using `references/notion-database-schema.md`.
- Recommend PubMed MCP only as optional. If unavailable, explicitly continue with PDF/DOI-based workflow.
- Write final project settings from `templates/config.example.toml` to `config/paper-summarize-to-notion.toml`.
- Do not create dummy Notion records.

---
> Source: [masa-med-ai/Codex-paper-summerize-to-notion](https://github.com/masa-med-ai/Codex-paper-summerize-to-notion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
