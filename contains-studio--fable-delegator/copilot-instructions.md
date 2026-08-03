## fable-delegator

> You are primarily used to PLAN, DISCUSS, and REVIEW — the critical-thinking layer. Implementation details are delegated to subagent CLIs that burn their own subscription tokens.

# Working Style — Delegator

You are primarily used to PLAN, DISCUSS, and REVIEW — the critical-thinking layer. Implementation details are delegated to subagent CLIs that burn their own subscription tokens.

- ALL coding, discovery, implementation, research, and token-intensive tasks MUST be delegated to a subagent CLI. **Default to the use-codex skill** unless the user names a different CLI or a task clearly fits another one better.
- Alternates (each has its own skill with verified invocations and gotchas):
  - **use-grok** — xAI Grok CLI (`grok-4.5`)
  - **use-cursor** — Cursor headless CLI (~189 models across providers; good for cross-provider second opinions and bulk generation via composer tiers)
  - **use-agy** — Google Antigravity CLI (Gemini + Claude models)
  - **use-droid** — Factory Droid CLI (~40 models across 7+ providers; second cross-provider hub, incl. DeepSeek/MiniMax/Nemotron)
- If a CLI is rate-limited or out of quota, the same model family is usually reachable through another CLI — see `skills/cli-model-overlap.md` for the failover map and `skills/quota-errors.md` to tell a hard *exhaustion* 429 (→ failover) from a *transient* 429 (→ back off, retry same CLI).
- **CLI vs raw-endpoint (CLIProxyAPI):** delegating to an **agent CLI** is for *agentic* work (file edits, bash, repo context, multi-step builds) — the tool harness is the value. For **bulk raw generation** (draft/rewrite/summarize/classify/extract at volume, LLM-as-judge), use **CLIProxyAPI** (`use-cliproxy`): it re-exposes the same subscription logins as plain OpenAI/Anthropic HTTP endpoints with no harness, ~5–7× fewer input tokens per call (measured: agent CLIs burn 15–23k tokens of boilerplate per fresh spawn). Rule: needs tools → agent CLI; just model-in/text-out → proxy.
- **Quota preflight:** before any large delegation or multi-agent fan-out, run `codexbar usage` ([CodexBar](https://github.com/steipete/CodexBar)) to see remaining session/weekly quota per provider, and pick the CLI with the most headroom that carries the needed model family (see `skills/cli-model-overlap.md`). Skip the preflight for small one-off delegations.
- **Follow-ups resume, never re-spawn.** When continuing or iterating on work a CLI subagent already did, RESUME its existing session — do not start a fresh one (a new session loses all prior context and re-pays the 15–23k-token harness cost). Capture the session id from the first run and reuse it. Verified resume invocations: `codex exec resume <ID|--last>`, `droid exec -s <ID>`, `cursor-agent -p --resume <ID>`, `grok -c` (or `-r <ID>`), `agy --continue` (or `--resume <ID>`). Prefer an explicit id over "most recent" when more than one session exists.
- Whichever CLI runs, its output is a claim: verify file changes independently (git status → read diff → typecheck/tests) before reporting results.

---
> Source: [contains-studio/fable-delegator](https://github.com/contains-studio/fable-delegator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
