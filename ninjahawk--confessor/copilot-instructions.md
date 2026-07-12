## confessor

> Confessor is a 100% local CLI that (1) reconstructs what an AI coding agent

# Confessor — notes for Claude Code sessions

Confessor is a 100% local CLI that (1) reconstructs what an AI coding agent
(Claude Code today) did on your machine from its own logs, and (2) scans AI
chat exports for secrets/PII/sensitive disclosures. One self-contained offline
HTML report. Zero runtime deps, zero network calls. Goal: a viral open-source
repo — the headline feature is the **agent forensics**, chat scanning is
secondary.

## The invariants (do not break these — CI enforces them)

1. **Zero network calls.** Nothing in `src/` may import `http`/`https`/`net`/
   `tls`/`dgram`/`dns`/`http2` or call `fetch`/`XMLHttpRequest`/`WebSocket`.
   Enforced by `scripts/no-network-check.mjs` (runs in CI + `prepublishOnly`).
2. **Zero runtime dependencies.** `package.json` deps must stay empty. Need a
   capability? Write it from scratch (that's why `src/zip.ts` exists).
3. **No raw secret in any output ever.** Every detection rule has a `redact()`;
   previews are built by `src/detect/engine.ts` `buildPreview`/`scrubRange`,
   which scrub against *all* findings in the text. Command previews and tool
   results in the agent path also go through `redactSecrets()`. Tests assert no
   planted value from `test/helpers/plants.ts` (`MUST_NOT_LEAK`) appears in the
   report or the agent audit.
4. **The report is one offline file** — inline CSS/JS/SVG, a strict CSP, no CDN,
   no fonts, no fetches.

## Commands

```bash
npm test                 # build src + tests, run node:test (73 tests)
npm run build            # tsc → dist/
npm run check:no-network # the zero-network guarantee
node dist/cli.js scan <paths...> --out docs/index.html --no-open --quiet
```

There is no lint step. Match the existing style: small pure functions, rules as
data, comments only where the code can't say it.

## Architecture

- `src/adapters/*` — turn each source into `SourceMessage[]`. `claude-code.ts`
  parses `~/.claude/projects/**/*.jsonl` (skips `isMeta`, pulls a `summary`
  title). ChatGPT/Claude/Gemini/generic + `zip.ts` (from-scratch `node:zlib`).
- `src/detect/*` — three layers of plain-data rules: `secrets.ts` (30 rules),
  `pii.ts` (13), `categories.ts` + `lexicons.ts` (7 topic lexicons).
  `engine.ts` runs rules, resolves overlaps by priority, builds redacted
  previews. `validators.ts` = Luhn/entropy/structure.
- `src/agent/*` — **the headline feature.** `activity.ts` replays the Claude
  Code JSONL as ordered tool calls (preserving tool *names*, which the message
  adapter discards) and produces an `AgentAudit`: files opened (classified by
  `classify.ts`), secrets that entered context (detection engine over tool
  *results*), network sinks, and **exposure paths**. `classify.ts` = path
  sensitivity + command/host classification.
- `src/audit.ts` (`Auditor`) — aggregates chat findings; `src/index.ts`
  (`audit()`) wires it together and attaches `result.agent` when a Claude Code
  projects dir is among the targets.
- `src/report/html.ts` — the whole report as one string. `src/cli.ts` — arg
  parsing + terminal summary.

### Exposure paths (the core novel idea — keep it precise)

An exposure path = a sensitive read (a `secret`-class file, or a
*high-confidence* secret in a tool result — NOT generic `x=y`/password rules)
followed **within ~6 tool-calls and ~5 minutes** by a **data-carrying** sink
(`curl`/`wget`/`scp`/`nc`/`rsync`/`WebFetch`/MCP) to an **external, non-benign**
host. WebSearch, `git push`, `npm publish`, and benign hosts
(github/npm/pypi/anthropic…) never trigger one. Each source links to at most
one sink; pairs are deduped. This precision was tuned against 219 real sessions
(loose linking gave 25 false positives → tightened to 2 real ones). The
regression tests in `test/agent.test.ts` (the `PRECISION:` ones) exist to keep
it honest — **do not loosen without re-validating on real logs.** An exposure
path is framed as a *lead, not proof*; keep that framing in all copy.

## Design language (report)

Light, Apple-style: `#f5f5f7` canvas, white cards (20px radius, soft shadow),
system fonts, iOS palette (`#ff3b30`/`#ff9500`/`#007aff`/`#34c759`), iOS
segmented-control filters. Plain non-technical language: "passwords & keys" not
"credentials", "personal details" not "PII", "private topics" not
"disclosures". The report must read for a 40-year-old who knows nothing about
AI. (See the private memory `apple-design-taste` for the origin of this bar.)

## Workflow gotchas (Windows / this machine)

- **Never commit with a Claude co-author.** The user's global settings disable
  it (`includeCoAuthoredBy:false`, empty `attribution`); don't re-add it.
- **git push + fixtures = GitHub push protection.** Fixtures contain
  realistic-looking fake keys. Account-level push protection blocks the push
  even with the repo setting off. Workaround: flip repo private → push → public,
  then resolve the secret-scanning alerts as `used_in_tests`. **Flipping private
  deletes GitHub Pages** — re-enable it after (`gh api -X POST repos/.../pages
  -f "source[branch]=main" -f "source[path]=/docs"`) and wait ~60s between
  visibility changes (else HTTP 422). Details in memory `github-push-protection-fixtures`.
- **PowerShell + git commit messages:** here-strings with inline quotes/parens
  break arg parsing — write the message to a temp file and use `git commit -F`.
- **Regenerating the sample report / screenshots:** rebuild, run the CLI to
  `docs/index.html`, then inject the sample banner via
  `scratchpad/add-sample-banner.mjs`. Screenshots are captured with headless
  Edge (`--headless=new --screenshot`); to isolate a section, append
  `<style>section:not(#id){display:none!important}</style>` to the HTML. Read
  PNGs back with the Read tool to verify before committing.
- `SPEC.pdf` is gitignored (private). `LAUNCH.md` is gitignored (launch drafts —
  deliberately not in the public repo).

## State / open items (as of 2026-07-10)

- Live sample report: https://ninjahawk.github.io/Confessor/ (Pages from `docs/`).
- **Not yet published to npm** — `npx confessor` won't work for others until the
  user runs `npm publish`. This gates the launch.
- Launch collateral drafted in the gitignored `LAUNCH.md`.
- Next big lever: **Cursor/Cline log support** (widens audience a lot) — needs
  real log samples to build without guessing the format.

---
> Source: [ninjahawk/Confessor](https://github.com/ninjahawk/Confessor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
