## tokenscope

> CLI that profiles AI session logs for token waste. "Chrome DevTools for your

\# tokenscope



CLI that profiles AI session logs for token waste. "Chrome DevTools for your

context window" — shows where tokens went, flags waste, suggests fixes.



\## Principles (do not violate)

\- 100% offline: no API calls, no API keys, no telemetry. Everything runs locally.

\- Dependency-light: currently only `js-tiktoken`. Justify any new dependency.

\- Zero config: `npx tokenscope <path>` must work with no setup.

\- Findings are linter-style: every finding has a code (W00x), a detail, an

&#x20; estimated `wastedTokens`, and a concrete `fix`.



\## Stack

\- Node 18+, ESM (`"type": "module"`), plain JS, no build step, no TypeScript.

\- Entry point: `src/cli.js` (also the npm `bin`).



\## Architecture (data flows in this order)

1\. `src/parser/claudeCode.js` — parses Claude Code JSONL into the unified

&#x20;  session model. New adapters for other tools go in `src/parser/` and MUST

&#x20;  output the same model.

2\. `src/tokenizer.js` — offline token counting (o200k via js-tiktoken).

&#x20;  Counts are estimates (\~±5%); this is documented and acceptable.

3\. `src/analyzers/index.js` — category breakdown + waste detectors.

&#x20;  W001 = repeated content (chunk-level fingerprinting, 20-line windows).

&#x20;  W002 = bloated tool results.

4\. `src/report/terminal.js` — ANSI report. `src/report/html.js` — single

&#x20;  self-contained HTML file, zero external requests (hand-rolled treemap,

&#x20;  no CDN imports — keep it that way).



\## Unified session model

{ source, adapter, messages: \[{ index, role, tokens,

&#x20; blocks: \[{ kind, label, text, tokens }] }], totalTokens }

Block kinds: system | user\_text | assistant\_text | tool\_use | tool\_result |

attachment | other. Do not add kinds without updating analyzers and both reports.



\## Testing

\- Fixture: `fixtures/sample-session.jsonl` (contains deliberate waste:

&#x20; a file re-sent 3x and one bloated tool result).

\- Smoke test after any change:

&#x20; `node src/cli.js fixtures/sample-session.jsonl --html report.html`

&#x20; Expected: W001 and W002 both fire, total 33,821 tokens.



\## Conventions

\- Keep parsers tolerant: skip unparseable lines, never crash on weird logs.

\- No emojis in terminal output. Sentence case everywhere.

\- Roadmap order: v0.2 proxy mode + dollar estimates → v0.3 OpenAI/generic

&#x20; JSONL input → v0.4 prompt linting + GitHub Action.

---
> Source: [yaswanthme007/tokenscope](https://github.com/yaswanthme007/tokenscope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-03 -->
