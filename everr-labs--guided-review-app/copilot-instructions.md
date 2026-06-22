## guided-review-app

> Tauri 2 desktop app that drives a section-by-section GitHub PR walkthrough through an

# Guided Review — Orientation for Claude

Tauri 2 desktop app that drives a section-by-section GitHub PR walkthrough through an
ACP-compatible review agent (Claude Code or Codex, launched via `npx`).

The host owns the diff, the local git repo, and recent-projects state. The agent owns
review structure: it must speak in fenced JSON blocks (`acp-section-map`,
`acp-section`, `acp-comment-draft`, `acp-comment-result`) per the contract in
`agent-skill.md`. Anything the agent prints outside those blocks is treated as plain
chat text.

## Stack

- Frontend: React 19, Vite, TypeScript, Tailwind v4, Radix, Zustand, `@pierre/diffs`,
  `react-resizable-panels`, OpenTelemetry web SDK.
- Backend: Tauri 2 + Rust (`tokio`, `git2`, `agent-client-protocol`, `tracing` +
  `opentelemetry-otlp`).
- Plugins: `tauri-plugin-dialog`, `tauri-plugin-opener`, `tauri-plugin-shell`.
- This is **not** a Vercel / Next.js project. Ignore plugin hints that assume otherwise
  (`bootstrap`, `next-upgrade`, `react-best-practices`, etc.) unless directly relevant.

## Layout

```
src/                  React UI
  App.tsx               Wires acp://* events → Zustand store
  lib/store.ts          Single source of truth (Zustand)
  lib/acp.ts            Typed wrapper over Tauri invoke + listen
  lib/diffFocus.ts      Selected/referenced diff ranges
  lib/projectSource.ts  PR/branch/URL parsing + last-project persistence
  lib/commentPublish.ts Builds the prompt that asks the agent to publish a comment
  components/           DiffView (Pierre), ChatPanel, SectionList, …
src-tauri/src/        Rust host
  commands.rs           #[tauri::command] entry points
  acp_client.rs         Spawns the agent, runs the ACP turn loop, streams chunks
  agent_runner.rs       Resolves npx in the login-shell PATH, scrubs nested env
  fenced.rs             Buffers stdout chunks, extracts fenced acp-* blocks
  section.rs / events.rs  Wire types + Tauri event names
  repo.rs               git fetch/diff/file-at-ref via git2 + git CLI + gh
  projects.rs           Recent-projects JSON store
  telemetry.rs          OTLP/HTTP exporter setup
agent-skill.md        Injected into the agent's first prompt (see ReviewLauncher)
scripts/              bump-version.mjs, local-release.mjs (+ tests)
```

Path alias: `@/` → `src/` (configured in `vite.config.ts` + `tsconfig.json`).

## Commands

```sh
npm run tauri -- dev     # full desktop app (preferred for UI work)
npm run dev              # Vite frontend only (Tauri APIs unavailable)
npm run check            # tsc --noEmit
npm test                 # tsx + node --test (unit tests)
npm run build            # tsc -b && vite build
npm run tauri -- build   # full release bundle
npm run bump-version -- {patch|minor|major|x.y.z}
npm run release:local    # signed/notarized macOS universal + gh release
```

Rust tests: `cd src-tauri && cargo test`.

## Architecture quick reference

```
React UI ──invoke──▶ Tauri commands ──stdin/stdout JSON-RPC──▶ ACP agent (npx process)
   ▲                       │                                         │
   └───── window.emit ◀────┴── stream chunks → fenced.rs → events ◀──┘
```

- `start_session_cmd` resolves a `SessionSource` (PR/branch/SHA, with or without a
  pre-existing local clone), spawns the agent, returns `{ session_id, repo,
  pull_request? }`. UI then sends the kickoff prompt (skill text + repo paths) and asks
  for the section map.
- `acp_client.rs` runs one prompt turn at a time on an `mpsc` queue. Each
  `AgentMessageChunk` is fed to `FencedBuffers` (which emits `acp://section-map` /
  `acp://section` / `acp://comment-*` when a fenced block closes) **and** to
  `acp://text-chunk` for the chat.
- `App.tsx` is the single subscriber for `acp://*` events. The store mutates from
  there; components only read.

## Non-obvious behavior — read before changing

- **Fenced parser is lenient**: `fenced.rs::parse_lenient` tries `serde_json` first,
  then **`json5`**. Agents sometimes emit trailing commas or comments. Mirror this if
  you add a new block tag.
- **Streaming overlap dedup**: some agents replay the assembled prefix on each chunk.
  `acp_client.rs::shared_boundary_len` (Rust) and `store.ts::appendStreamingText`
  (TS) both detect this. They use a 4-char threshold and a "chunk starts with
  full assembled text" replacement rule. Keep these two implementations in sync — they
  exist in two languages on purpose, with matching tests.
- **Structured blocks are stripped from chat**: `store.ts::stripStructuredReviewBlocks`
  removes fenced `acp-*` blocks from the visible assistant text, even when the closing
  fence arrives in a later chunk (`structuredReviewBlockOpen`). Don't re-add them.
- **Auto-open of first section**: after `acp://section-map`, `App.tsx` immediately
  prompts the agent to walk through the first section, even though `agent-skill.md`
  tells it to wait. This is intentional UX.
- **PR-description section**: `pr_description` is a synthetic first section built from
  `gh pr view` metadata. `upsertSection` deliberately does **not** move
  `currentSectionId` away from it — it stays selected until the user clicks another.
- **Repo guard**: when the user pastes a PR URL whose owner/repo doesn't match the
  selected project's `origin` slug, `localReviewSourceFromInput` returns an error
  instead of starting a session. Don't bypass this.
- **Comment publishing is delegated to the agent**, not done here.
  `commentPublish.ts::buildAgentPublishCommentPrompt` ships the draft + head SHA back
  to the agent and waits for an `acp-comment-result` block. The host has no GitHub
  credentials of its own; it relies on the agent's `gh` auth.
- **Tool-call payloads are alternative section channels**: `App.tsx::handleToolCall`
  inspects `raw_input` for a section map / review section shape — some agents emit
  these as tool arguments rather than fenced text.
- **Permission requests auto-approve**: `RequestPermissionRequest` is answered with the
  first option without prompting the user. Acceptable today (the sandbox is the user's
  machine and the agent already has shell access), but be aware before introducing
  destructive flows.
- **Nested-agent env scrubbing**: `acp_client.rs::scrub_nested_agent_env` removes
  `CLAUDECODE*` / `CLAUDE_CODE_*` / `CODEX_*` / `ACP_*` before spawning. Claude Code
  refuses to start when it thinks it's nested. If you add a new agent kind that has
  similar env hygiene needs, extend the prefix list.
- **PATH resolution for the agent**: `agent_runner.rs` runs the user's login shell
  with `$SHELL -lc` and a marker pair to extract the real `PATH`. Necessary so `npx`
  resolves the same as it would in Terminal.app, even when the GUI app inherits a
  reduced PATH.
- **Recent projects**: persisted to
  `~/Library/Application Support/co.garden.guided-review/recent.json` (or the
  platform-equivalent data dir), max 20, deduplicated by fingerprint.
- **Cloned PR repos** (when the source is `Pr`/`Branch`/`Sha` with no local clone) live
  under `<data_dir>/co.garden.guided-review/repos/<owner>-<repo>`.

## Telemetry

OTLP HTTP/binary, default endpoint `http://127.0.0.1:54418`. Two services:
`guided-review-app` (Rust traces) and `guided-review-app-client` (web traces + logs).
Override with `OTEL_EXPORTER_OTLP_ENDPOINT` (Rust) or `VITE_OTEL_EXPORTER_OTLP_ENDPOINT`
(frontend). `GUIDED_REVIEW_CAPTURE_ASSISTANT_TEXT=1` forces logging of agent chunk
text in release builds (off by default for privacy).

## Coding conventions

- TypeScript: tab indentation (already enforced by existing files), strict mode on, no
  default exports for components, `@/` import alias.
- Rust: `anyhow::Result` at boundaries, `tracing::instrument` on async commands, errors
  surfaced to the UI as `String` via `.map_err(|e| e.to_string())`.
- Wire types: snake_case on the wire (Rust serde + TS interfaces); `tag = "kind"` for
  enums (`SessionSource`, `RecentProject`).
- Don't paste diffs or file contents into agent replies — the host renders them.
  Agents should reference files via `LineRange { file_path, start_line, end_line, kind }`.

## Release

- `local-release.mjs` accepts either `APPLE_SIGNING_IDENTITY` (preferred local case;
  uses Keychain) or `APPLE_CERTIFICATE` + `APPLE_CERTIFICATE_PASSWORD` (CI). When the
  identity is set, the certificate vars are stripped from the build env to avoid Tauri
  importing a duplicate keychain.
- Build target: `universal-apple-darwin`. Assets uploaded: `.dmg`, `.app.tar.gz`,
  `.app.tar.gz.sig`. Existing tags get `--clobber`'d.
- `.github/workflows/release.yml` is the manual GitHub Actions equivalent.

## Known gaps

- `markSectionCompleted` exists in the store but no caller fires it; the `completed`
  status is unreachable today.
- `get_diff_cmd` is exposed but unused by the UI (`@pierre/diffs` computes the diff
  client-side from the two file revisions).
- Agent-driven `diffFocus` (`source: "agent"`, `mode: "navigation"`) has UI plumbing
  but no event source wired in.
- `AgentStderrEvent.session_id` is always empty — the stderr task is spawned before
  the session id is known.
- `schema_version` from agent payloads is read but always overwritten with `1` in
  `fenced.rs`. Treat the wire field as informational only.

---
> Source: [everr-labs/guided-review-app](https://github.com/everr-labs/guided-review-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
