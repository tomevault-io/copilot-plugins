## ai-website-audit-cli

> Guidance for Codex and other AI coding agents working in this repository.

# AGENTS.md

Guidance for Codex and other AI coding agents working in this repository.

## Project intent

`ai-website-audit-cli` is a small, inspectable, MIT-licensed CLI for website audits. It combines deterministic extraction/scoring with OpenAI-powered report generation through the Responses API.

## Contribution rules

- Keep deterministic checks transparent and testable.
- Do not hide network calls in helper functions without clear naming.
- Do not send secrets, `.env`, cookies, or private headers to OpenAI.
- Keep OpenAI calls in `src/ai_website_audit/openai_client.py`.
- Keep prompts in `prompts/` and prompt assembly in `src/ai_website_audit/prompts.py`.
- Add or update tests for parsing, scoring, CLI behavior, and report generation.
- Prefer small focused PRs.

## Commands

```bash
python -m venv .venv
source .venv/bin/activate
pip install -e . pytest
pytest
ai-website-audit inspect https://example.com --html
ai-website-audit audit https://example.com --language en --html
```

## Safety and privacy

- The tool only fetches public URLs provided by the user.
- The AI prompt contains extracted public page data and deterministic audit output.
- The default setting is `OPENAI_STORE_RESPONSES=false`.
- Never commit API keys or generated reports containing sensitive private websites.

---
> Source: [Tools2U/AI-Website-Audit-CLI](https://github.com/Tools2U/AI-Website-Audit-CLI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
