## synadia-agents-labs

> Orientation for Claude Code (and any new contributor) working in this repo.

# CLAUDE.md

Orientation for Claude Code (and any new contributor) working in this repo.
Read this first; it assumes you know nothing about the project.

## What this repo is

**`synadia-agents-labs`** is standalone **courseware** — small, runnable,
heavily-commented examples for the **Synadia Agent Protocol for NATS** SDKs. It
accompanies Synadia workshops and trainings, and doubles as a self-study ground.

It is deliberately **independent** of the SDK *source*: every example depends on
the **published** packages (`@synadia-ai/agents` + `@synadia-ai/agent-service`
from npm; `synadia-ai-agents` + `synadia-ai-agent-service` from PyPI). There is
**no link back** to the SDK monorepo — no `file:` deps, no submodules, no
relative imports into another repo. You can develop here without the SDK source
checked out.

- **User-facing overview:** [`README.md`](README.md) — layout, conventions, the
  environment-variable reference, quickstart. Keep it in sync when you change
  user-visible surface.
- **The protocol spec** (source of truth for wire shape) is **not vendored**:
  <https://github.com/synadia-ai/synadia-agent-sdk-docs>. Link to it; don't copy
  it.
- **The SDK source** lives in the separate `synadia-ai/synadia-agents` monorepo.
  It's a *reference*, not a dependency. (On this machine it's typically a sibling
  dir: `../synadia-agents`.)

## The protocol in 30 seconds

An **agent** is a NATS micro-service identified by a triple
`<agent>/<owner>/<session>` (e.g. `echo/alice/main`) — the SDK calls the 5th
subject token `name` in TypeScript and `session_name` in Python. From it the SDK
derives subjects:

- `agents.prompt.<agent>.<owner>.<name>` — send a prompt, stream a reply
- `agents.status.<agent>.<owner>.<name>` — status endpoint
- `agents.hb.<agent>.<owner>.<name>` — heartbeats (liveness)

A reply is a stream of typed frames: `{"type":"status","data":"ack"}` →
`{"type":"response","data":"…"}` (one or many) → an **empty terminator**.
**Callers** (`@synadia-ai/agents`) discover agents and prompt them; **agents**
(`@synadia-ai/agent-service`) host the endpoints. Wire **protocol version is
`0.3`**; the published package version is currently `0.5.2` (different axes —
don't conflate).

## Layout

```
typescript/
  client/   01-discover 02-prompt-text 03-prompt-attachment 04-query-reply 05-liveness 06-chat
  agent/    01-echo 02-llm-ollama 03-llm-openrouter 04-llm-combined 05-system-prompt 06-chat-agent reference-agent
python/     parity with the TypeScript examples through agent 04 (agent 05–06 not yet ported)
projects/
  tool-calling/  typescript/ + python/ agents (01-single-tool … 04-memory) +
                 responders/ (calculator·time·temperature·inventory·memory, polyglot;
                 the calculators join queue group "calculators" → fleet demo)
  human-tool/      TS agent whose ask_expert tool is answered by a HUMAN (`nats reply tools.expert.<owner>`)
  crowd-poll/      TS agent whose ask_the_room tool scatter-gathers `room.poll` via requestMany
  agent-of-agents/ TS "reporter" agent whose tools are the caller SDK (discover + prompt other agents)
  edge-sensor/   cross-domain capstone (TS agent + Go bridge + Go mock sensor)
labs/       guided workshop track — numbered lab sheets (00-setup … 09-agent-of-agents),
            plus `-python` twins (00/02/04/05) where the repo has Python parity.
            TRACKED courseware: attendees follow these from their checkout; secrets
            (creds, API keys) are never in them — those go via workshop chat
tests/      verification harness — runs every example, writes a transcript (tests/README.md)
infra/      dev container + docker-compose (nats-server + Node·Bun·Python·uv·Go + nats CLI; no LLM bundled) — see infra/README.md
presenter/  gitignored, presenter-only: live runbook + workshop structure (not courseware)
docs/       (planned — protocol primer & deeper guides)
```

Two tiers: the **core examples** (small, single-language lessons under
`typescript/` & `python/`, split into `client/` callers and `agent/` hosts) and
**`projects/`** (bigger, self-contained demos, sometimes multi-language — e.g.
tool-calling with polyglot responders, the edge-sensor capstone).
Terminology: call the numbered sequences the **client/agent examples** — the
word "ladder" is banned in all text (maintainer preference).

## Conventions — treat these as rules

1. **Every example is self-contained.** Its own `package.json` (TS) /
   `pyproject.toml` (Py), its own `tsconfig.json`, its own README, all of its
   code in the folder. You can copy a single example folder out of the repo and
   it still runs. We **accept duplication** as the price of clarity — a learner
   should see *exactly* what an example uses without chasing imports.
2. **Pinned, published SDKs.** Depend on the published packages by semver, never
   a path back to the SDK source: npm `@synadia-ai/agents` + `@synadia-ai/agent-service`
   (`^0.5.2`), PyPI `synadia-ai-agents` (`>=0.7`) + `synadia-ai-agent-service`
   (`>=0.4`). To bump, edit each example's `package.json` / `pyproject.toml`
   (there is no shared manifest by design).
3. **Role-based filenames.** The runnable file is named for its protocol role:
   `agent.{ts,py}` (a host), `client.{ts,py}` (a caller), `responder.{ts,py}` (a
   plain-NATS tool backend, in projects). Shared-but-copied helpers get a plain
   name: `llm.ts` / `llm.py` (and `llm-multitool.ts` / `llm_multitool.py`).
4. **Reuse is copy-forward, not import.** When a later example builds on an
   earlier one, it carries its **own copy** of the shared file with a comment
   pointing back to the original. The keystone case: `agent/04-llm-combined/llm.ts`
   (a backend-agnostic chat client) is the base the tool-calling project copies
   and extends. Locality beats DRY in a teaching repo.
5. **Clients and agents are decoupled.** Any client works against any agent —
   that's what discovery is for. Don't wire them together in code; the READMEs
   narrate "run agent X in one terminal, client Y in another."
6. **Uniform connection resolution**, inlined in every example (it's just SDK
   calls — keep it visible, don't extract):
   `NATS_CONTEXT` → `NATS_URL` → `nats://127.0.0.1:4222`.

## Environment variables

Full table in [`README.md`](README.md#environment-variables). The scheme:

- **Our** connection/agent config carries a **`NATS_` prefix**: `NATS_CONTEXT`,
  `NATS_URL`, `NATS_AGENT_OWNER` (default `$USER`), `NATS_AGENT_NAME` (default
  `main`), `NATS_AGENT_HEARTBEAT_INTERVAL` (seconds).
- **Third-party tool** variables keep their **conventional, un-prefixed names**
  so they interoperate with what users already have: `OLLAMA_URL`,
  `OLLAMA_MODEL`, `OPENROUTER_API_KEY`, `OPENROUTER_MODEL`.
- Agent examples honor `NATS_AGENT_OWNER`/`NATS_AGENT_NAME` so many people can
  run the same example against one shared server (e.g. synadia-cloud) without
  subject collisions. The `<agent>` token is fixed per example.

## Running an example

Each folder is independent:

```sh
cd typescript/agent/01-echo
npm install        # or: bun install
npm start          # or: bun agent.ts   (scripts use tsx; Node 20+ or Bun)
npm run typecheck  # tsc --noEmit
```

The Python examples are identical, with **[uv](https://docs.astral.sh/uv/)**:

```sh
cd python/agent/01-echo
uv run python agent.py     # uv builds the per-folder venv from pyproject.toml
```

Agents and clients pair across two terminals — e.g. start `agent/01-echo`, then
run `client/01-discover`. LLM agents (`02-llm-*`) need either a local
[Ollama](https://ollama.com) or an `OPENROUTER_API_KEY`; `04-llm-combined`
auto-selects (key present → OpenRouter, else Ollama) and logs the choice.

## Verifying changes

- **Run the harness** — the fastest way to confirm everything still works:
  `bash tests/run-all.sh` boots an isolated NATS server, runs every example in
  both languages, and writes a transcript to `tests/logs/`. `--core` skips the
  LLM tier (no model needed); `--tools` needs Ollama. See
  [`tests/README.md`](tests/README.md). CI (`.github/workflows/ci.yml`) runs the
  static checks + the `--core` smoke on every push/PR.
- **Typecheck** is the cheap gate: `npm run typecheck` in the changed folder.
  Examples are written against the *published* SDK, so typecheck catches API
  drift a local SDK checkout would hide.
- **Runtime smoke test** needs a NATS server. Pattern that works headless:
  start `nats-server` on an **isolated port** (e.g. `-p 4299`) so you never
  touch a real server, wait for readiness by **grepping the server log** for
  `"Server is ready"` (don't rely on `/dev/tcp` — the dev shell is zsh, which
  lacks it), run the example via its local
  `./node_modules/.bin/tsx <abs-path>`, then kill everything. Foreground
  `sleep` may be unavailable; use `perl -e 'select(undef,undef,undef,0.1)'` for
  short waits.
- **Heads-up:** the production `AgentService` heartbeat defaults to **30s** (only
  the testing `ReferenceAgent` uses a shorter default). So `05-liveness` shows
  `no heartbeat yet` for up to 30s — that's expected. Use
  `NATS_AGENT_HEARTBEAT_INTERVAL=2` for a livelier demo.
- **Gotchas the harness surfaced (keep them fixed):**
  - **Python signal handling** must use `loop.add_signal_handler(sig, stop.set)`,
    *not* `signal.signal(...)` + `Event.set()` — the latter doesn't wake
    `await stop.wait()`, so the agent ignores SIGTERM (won't stop on `kill`/Ctrl-C).
  - **Local LLM tool-routing is probabilistic** — a small model sometimes answers
    without calling a tool, or omits the number. The harness retries the LLM tier;
    don't treat a single chatty miss as a regression. No-argument tools
    (`list_memories`, `get_current_time`) need a capable model (`gpt-oss`/OpenRouter).
  - The dev shell is **zsh** (no `/dev/tcp`); foreground `sleep` may be blocked
    (use `perl -e 'select(undef,undef,undef,0.1)'`); `tsx`/`uv` spawn a child the
    wrapper PID may not reap (kill by file pattern or use the harness helpers).
  - **The harness exercises only `NATS_URL`** — the `NATS_CONTEXT` (+ creds)
    path needs its own smoke test. Two traps found this way (June 2026):
    responders used to skip `NATS_CONTEXT` entirely (fixed — they now resolve
    `NATS_CONTEXT → NATS_URL → localhost` like every example; the SDK helper /
    Go context reader is used only to *read* the context), and **nats-py needs
    the `nkeys` extra** to use a creds-bearing context — every Python example
    pins `nats-py[nkeys]>=2.6`; keep that extra when adding new ones. Also:
    `nats context save` copies unset fields (including **creds**) from the
    currently selected context — isolate test contexts deliberately.
  - **Synadia Cloud accounts require `max_bytes` on stream/KV creation** — a
    bare `kvm.create(bucket)` works locally but is rejected on NGS ("account
    requires a stream config to have max bytes set"). The memory responder
    passes `{ max_bytes: 1024 * 1024 }`; cap any new JetStream resource.

## Adding a new example

1. Copy the nearest existing example folder (don't scaffold from scratch).
2. Keep it self-contained: own `package.json`/`tsconfig`, pin the published SDK,
   role-based filename, inline the connection resolution.
3. If it builds on an earlier example, **copy** the shared file in and add a
   comment pointing to the origin (copy-forward).
4. Write a README with the standard sections: *What this shows · Prerequisites ·
   Configuration · Run · How it works · Next*.
5. Update [`README.md`](README.md): the relevant *What's inside* table.
6. Verify: `npm run typecheck`, then a runtime smoke test if feasible.

## Status & roadmap

**Done** — TypeScript + Python at one-for-one parity (except where noted), all
core tiers verified by `tests/`:

- **Client examples** `01-discover … 06-chat` and **agent examples**
  `01-echo … 04-llm-combined` + `reference-agent`, in both languages.
- **Agent examples `05-system-prompt` + `06-chat-agent`** (TS only so far —
  Python port pending): 05 = 04 plus a persona (`SYSTEM_PROMPT` constant /
  `AGENT_SYSTEM_PROMPT` env); 06 = 05 plus an in-memory accumulating
  conversation (one shared history per process, bounded by `MAX_EXCHANGES`).
- **Workshop trio under `projects/`** (TS only, built for shared-server
  sessions): **`human-tool`** (ask_expert → `tools.expert.<owner>`, a human
  answers via `nats reply`, 15s timeout via `EXPERT_TIMEOUT_S`),
  **`crowd-poll`** (ask_the_room → `requestMany("room.poll")`, 10s window via
  `POLL_WINDOW_S`, strategy `"timer"`), **`agent-of-agents`** (reporter with
  `list_agents`/`ask_agent` tools wrapping the caller SDK; refuses to interview
  itself; copies the 03-tool-loop llm-multitool with `MAX_STEPS=10`). The
  calculator responders additionally joined queue group `calculators` and log
  `served: …` per request (fleet demo).
- **`labs/`** — the guided workshop/self-study track (lab sheets `00 … 09`
  sequencing the core examples + projects). Conventions: one goal line, paste-ready
  commands, an *Expected* block, a *Self-study* note; no secrets ever (keys and
  creds travel via workshop chat). Lab-relevant gotcha: plain `nats reply`
  joins a default queue group — crowd-style labs need `--queue ''`.
- **`projects/tool-calling`** — `01-single-tool … 04-memory` agents in both
  languages, each tool body a single `nc.request(subject)` answered by **polyglot
  responders** (calculator/time/temperature in TS·Go·Python; inventory + a
  JetStream-KV memory in TS). `04-memory` persists to a KV bucket. Agents copy
  `04-llm-combined/llm.{ts,py}` forward and add tools.
- **`projects/edge-sensor`** — cross-domain capstone (TS agent + Go bridge + Go
  mock sensor).
- **`tests/`** verification harness + **`.github/workflows/ci.yml`** (static
  checks + a no-LLM protocol smoke).
- **`infra/`** — a one-size-fits-all dev container + docker-compose (a `nats`
  service with JetStream, plus the full **Node·Bun·Python·uv·Go·NATS** toolchain).
  Deliberately **no LLM bundled** — the LLM examples reach OpenRouter or a host
  Ollama. Repo is bind-mounted (edit on host, run inside); a VS Code
  `.devcontainer/` reuses the same compose. Verified end-to-end (TS + Python
  agents built/discovered cross-language, Go build). See `infra/README.md`.

**Still planned:**

- **Python parity** for agent `05-system-prompt` / `06-chat-agent` and the
  workshop trio (`human-tool`, `crowd-poll`, `agent-of-agents`).
- **`docs/`** — a protocol primer & deeper guides.
- **Exercises** — fill-in-the-blank starters + solutions, so the repo works for
  self-study after a workshop.

## Conventions for commits & PRs

- Prefer PRs once a remote exists; keep commits scoped and descriptive.
- Don't commit `node_modules` or lockfiles (see `.gitignore`; lockfiles are
  intentionally ignored — each example installs independently).
- **`memory/`** is a gitignored local dev journal (cross-session notes for Claude
  Code); the curated, durable project knowledge lives in *this* file. `tests/logs/`
  and **`presenter/`** (presenter-only workshop runbook) are gitignored too.
- License is Apache-2.0.

---
> Source: [synadia-ai/synadia-agents-labs](https://github.com/synadia-ai/synadia-agents-labs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
