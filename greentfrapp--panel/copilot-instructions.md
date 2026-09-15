## panel

> An agentic research workspace: a dockable-pane UI (Vue) over a Python sidecar that drives

# Panel — working in this repo

An agentic research workspace: a dockable-pane UI (Vue) over a Python sidecar that drives
agent backends and runs Modules. See [README.md](./README.md) for the vision and
[docs/](./docs/) for the design.

## Two runtimes

This is a polyglot monorepo. **The frontend and the backend are separate processes and
separate package managers.** Getting this wrong is the most common mistake here.

| | Path | Manager | Never use |
|---|---|---|---|
| Web app | `apps/web` | `pnpm` | `npm`, `yarn` |
| Sidecar | `apps/server` | `uv` | `pip`, `poetry`, a manually-activated venv |

```sh
pnpm install                 # workspace root — installs apps/web
uv sync                      # workspace root — installs apps/server + dev group
pnpm dev                     # runs both: Vite on :5173, sidecar on :8787
```

`pnpm dev` is `scripts/dev.mjs`, which runs both and **reaps the whole process tree on
exit**. It replaced `run-p` for that second half: `npm-run-all2` tree-kills only on task
failure and installs no signal handlers, so on Windows a Ctrl-C left twelve descendants
running. To run one side alone:

```sh
pnpm dev:web
pnpm dev:api
# if `uv run --directory` misbehaves on your platform:
cd apps/server && uv run uvicorn panel_server.app:app --port 8787
```

Vite proxies `/api` to `127.0.0.1:8787`, so the browser only ever talks to :5173.

**`pnpm dev` offers dev Modules; `pnpm start` does not.** `dev:api` loads
`apps/server/dev.env`, which sets `PANEL_DEV_MODULES=1` and so puts `tally` in the
catalogue. `pnpm start` builds the web app, serves it with `vite preview` on :4173 (proxying
`/api` the same way), and starts the sidecar without that file. It is what a tester runs.
The test suite sets the flag itself in `conftest.py`, because `MODULES` is built at import.

**The sidecar runs without `--reload`, and on Windows it must.** uvicorn picks its event
loop with `asyncio_loop_factory(use_subprocess)`, where `use_subprocess` means uvicorn's
own reload supervisor — and on Windows that choice costs *the application* the loop that
can spawn subprocesses:

| | loop | can a backend spawn a CLI? |
|---|---|---|
| `--reload` | `SelectorEventLoop` | no |
| plain | `ProactorEventLoop` | yes |

So `pnpm dev` with `--reload` could never run Claude Code, and would fail Codex the same
way. The symptom names neither the loop nor the reason: `CLIConnectionError: Failed to
start Claude Code:` with **nothing after the colon**, because the message is
`f"...: {e}"` and a bare `NotImplementedError` stringifies to `""`. An empty tail there is
the signature — the CLI is installed and fine.

**A jupyter kernel is a subprocess too, so it depends on the same loop** — and it also wants
something `ProactorEventLoop` does not have, `add_reader`, which zmq uses. Measured rather
than assumed: pyzmq registers a selector thread through tornado, prints a `RuntimeWarning`
saying so, and works. That warning in the sidecar's output is expected and is not the thing
to chase.

Editing Python therefore needs a manual restart. An external watcher would restore that,
but the one tried (`watchfiles` wrapping uvicorn) killed the whole console twice while
handling a change, taking the terminal with it, so it is not in place. Anything attempted
here should be started in its own console and proven not to signal the one it came from.

Verify both are up: `curl http://localhost:5173/api/health`.

### When the app looks broken before you have changed anything

```sh
pnpm dev:doctor
```

Reports each dev port as `free`, `healthy`, or `STALE`, and prints the command to clear a
stale one. The third state is the one worth knowing about: kill `pnpm dev` in a way that
skips the reaping — Task Manager, a force-stop, a closed terminal on some shells — and
`uvicorn --reload`'s supervisor can outlive its worker while keeping :8787 bound. The port
accepts connections and answers nothing, so the next run presents as a broken server
rather than as a leftover. `pnpm dev:doctor` tells the two apart in one command; a bare
`netstat` cannot.

Note Vite binds **IPv6** loopback. `http://127.0.0.1:5173` is refused while the server is
perfectly healthy, and `netstat -ano -p TCP` hides it — that flag means IPv4 only.

A fourth state exists that `dev:doctor` reports as `healthy`: a **surviving worker whose
supervisor was killed**. Kill the supervisor's pid alone and its `multiprocessing` child
lives on holding the inherited socket, so the port answers correctly while running
whatever code the dead tree started with. The recorded socket owner is then a pid that no
longer exists, which is the giveaway — and the worker is hard to find by name, since its
command line is `spawn_main(parent_pid=...)` and mentions neither `panel` nor the port.
`taskkill /T` on the supervisor avoids creating it; `Get-NetTCPConnection -LocalPort 8787`
plus a `tasklist` on the owner it names finds it afterwards.

## Layout

```
apps/web        Vue 3 + Vite + dockview + Tailwind + Pinia
apps/server     FastAPI sidecar — Runs, backends, Modules, DAL, custom Panes
packages/       not yet created; see docs/repo_structure.md §5 for the trigger
docs/           architecture and design; read before changing structure

~/Panel/        the user's own — panel.db beside workspaces/, outside the checkout
```

**Nothing durable is written inside the checkout.** The database defaults to
`~/Panel/panel.db` and Workspaces to `~/Panel/workspaces`, both home-relative so
the answer does not depend on where the sidecar was started — `./panel.db` gave one
database only because every documented launch command happens to land in `apps/server`.

Every checkout therefore shares one database. `schema.prepare` refuses to open one whose
`core.sql` has drifted, so a branch that changes the schema fails loudly rather than
corrupting it; set `PANEL_DB_PATH=./panel.db` in `apps/server/.env` (gitignored) to
keep a local one while working on that branch.

## Conventions

- **Vue SFC block order is `<template>`, then `<style>` if any, then `<script>`** — markup
  first, behaviour last, with no blank line between the top-level blocks.
- **Always `<script setup lang="ts">`.** No Options API, no untyped `<script setup>`.
- **Colour comes from semantic tokens, never from Tailwind's palette.** `bg-canvas`,
  `text-muted`, `border-warn-line` — not `bg-neutral-900` or `text-amber-400`. The tokens live
  in `apps/web/src/styles/styles.css`, which is the only file in the app that knows a hex code,
  and they flip between light and dark on their own. `docs/ui_design.md` says what each one
  means and what it may sit on. A misspelled utility emits no CSS and no error, so it is
  invisible until someone looks at the screen.
- **Serif is opt-in, and never below `text-sm`.** `--font-sans` is the document default, so
  chrome needs no class; `font-serif` marks prose — a model's reply, a Module's reported values.
  At 12px and below a text serif turns to mush.
- **The type scale is shifted one step and lives in `@theme`**, beside the colour tokens:
  `text-xs` is 14px, `text-sm` 16px. So the floor above is 16px, and the chrome — `text-xs`
  and `text-[12px]` — sits under it. Change a size there, never by renaming classes, and
  remember what the tokens cannot reach: every `leading-*`, the two `--dv-*-font-size`
  values, `lib/editor-theme.ts`, and a custom Pane's own scoped CSS. dockview's tab strip is
  deliberately left off the scale at 12px. `docs/ui_design.md` §3 has the list and why each
  is separate.
- **Read `docs/chat_architecture.md` before touching `apps/server`.** Decisions D1–D9 in §2
  record what was chosen and why; several look arbitrary without the rationale.
- **`panel_server.events` is the contract.** Backends normalize into it; nothing outside
  `panel_server/backends/*` may import a vendor SDK. Adding a field there affects every client.
- **A Run is created by its first message, and named by it.** Opening a tab makes a
  draft Pane with no `runId`; `Chat.vue` calls `POST /runs` when there is something to
  send. Nothing persists a Run before that — not the new-tab button, not startup. The
  title is derived server-side in `projections.py` so every client agrees on one rule,
  which is why `''` means "not named yet" and a blank title is a 422.
- **Python is `src`-layout.** Import as `from panel_server... import`, never by relative path
  from the working directory.
- **The sidecar binds `127.0.0.1`.** It holds agent credentials. Do not change this without
  reading §15. Outbound, it talks to backends and to exactly one other host: the model
  catalogue that tells `openai-api` which of its models answer chat, since `/v1/models` does
  not say. `openai_model_catalog_url` turns that off. Anything else reaching off-box is new.
- **Single process.** `mem://` and the in-process MCP server assume `--workers 1`.
- **Node is required even though the backend is Python** — the `claude` CLI is a Node
  program, and the same toolchain compiles custom Panes.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/) for the subject line:

```
<type>(<scope>): <imperative summary>
```

`type` is one of `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `build`, `chore`.
`scope` is the workspace the change lands in — `web`, `server`, `scripts`, `docs`, or
omitted when it spans them. Deliberately coarse: the workspace boundary is what predicts
which suite has to run, and finer scopes drift into inconsistency without earning
anything.

**Only the subject line is constrained.** Bodies here carry the reasoning — what was
measured, what was rejected, what a comment cannot hold — and that should not change. If a
commit needs no body it is usually a commit that needed no thought.

Commits before `7c9ebdf` predate this and are not conventional. Left alone: `git log`
reads fine, and rewriting history costs more than the inconsistency does.

## Testing

```sh
pnpm test                    # lint, then pytest, then vitest — stops at the first failure
pnpm lint                    # ruff check + ruff format --check
pnpm format                  # ruff format + prettier, both sides
pnpm test:api                # pytest alone, across every core
pnpm test:api:live           # only the tests that call a real model, and are billed
pnpm test:web                # vitest, once
pnpm type-check              # vue-tsc; deliberately not part of `pnpm test`
```

**`pnpm test` lints first.** Formatting drift is cheap to catch and annoying to review, so
it gates the suite rather than accumulating. If it fails, `pnpm format` fixes it — that is
the whole reason `format` covers Python as well as web.

`type-check` stays out because `vue-tsc --build` is slow and `vitest` does not typecheck
specs; run it before you commit web changes.

**A test marked `live` is deselected by default**, via `addopts` in
`apps/server/pyproject.toml`. These are the ones that prove a claim nothing else can — that
a real harness discovers a Module and asks before calling it — and they cost a model
request and the wait for one every time the suite runs. Deselected is not skipped for
missing tooling: `needs_cli` already answers whether the harness is installed, and a
machine can satisfy that and still not want to spend a request on every commit. Run them
before touching a backend adapter, since they are what would notice a harness changing
underneath one.

Web tests are `vitest` + `@vue/test-utils` on `happy-dom`, configured by the `test` key in
`apps/web/vite.config.ts` — they share the build's `@` alias rather than restating it. Specs
live beside the code as `src/**/*.spec.ts`.

`pnpm --filter @panel/web test:watch` reruns on save.

### Give a command a deadline it can miss

**`pnpm test:api` takes about a minute, vitest about twenty seconds.** It runs `pytest -n auto`;
a bare `pytest` is serial and takes four minutes, which is what to use when reading a failure.
Anything running materially longer than that has stopped, and the deadline set for it is what
decides how long that takes to notice. A generous timeout does not make a hang succeed; it
converts a failure that would have been visible immediately into several minutes of
apparent work.

So size it to what the thing actually takes, not to the worst case imaginable. A suite that
hangs is the *common* outcome here rather than an exotic one, because a parked permission
request waits for an answer that a test harness with nobody behind it never sends — and a
turn parked that way produces no output at all, which reads exactly like a slow one.

Two habits follow.

**Report progress to a file, not through a pipe.** `Select-Object -Last n` and friends
buffer until the command ends, so a run that never ends shows nothing whatsoever — the same
blank screen for "hung on the first test" and "almost finished". `pytest -v` written to a
file names the last test that passed, which is the one before the one that hung.

**A killed parent leaves its children running.** Stopping a wrapper shell does not stop the
interpreter beneath it, and the orphan keeps holding whatever it held — the same shape as
the reaping `pnpm dev` does and the worker that outlives its supervisor. Kill the tree
(`taskkill /T`), then confirm nothing survived: an orphaned suite competing for the same
files is indistinguishable from one that is merely slow.

Be careful what a process filter matches. A shell whose own command line contains the word
being searched for is itself a match, so a filter written to find stray test processes will
happily kill the shell running it.

### Anything that edits the tree while it runs

Scripts that mutate sources to check whether the tests notice are worth running, and worth
running carefully. They edit real files and restore them afterwards, which makes them the
one kind of tooling here that can lose work.

- **Foreground only.** A background job's lifetime is invisible, and one killed mid-edit
  leaves the mutation applied.
- **One at a time.** Two runs racing over the same files each restore a snapshot taken
  while the other had a file mutated, and both verdicts are then worthless. A lock file is
  enough.
- **Restore in an outer `finally`, then verify byte-for-byte** rather than trusting that it
  happened.
- **Check the result with `git diff`, not by searching for what was inserted.** A
  fingerprint search finds only substitutions; a mutation that *deletes* a line leaves
  nothing to match, so the check passes over a file that is still broken.

Not every survivor is a missing test. One sometimes shows a test passing for a reason other
than the one it claims — an assertion satisfied by a default elsewhere, where the framework
or the browser supplies the expected value whatever the code does.

## Status

**Milestones 1–10.** 1–9 are complete: the Run skeleton, the backend contract with two
unrelated implementers, Modules, the DAL, permissions, custom Panes, file and notebook Panes,
background jobs. Milestone 10 is the extension system, and only half of it is built — a Pane
can be authored, compiled and bound, and the three-tier *Module* resolver is not written.

What each milestone found — the bugs, the measurements, the diagnoses that were wrong first —
is in [docs/history.md](./docs/history.md). What is decided and unbuilt is §16 of
`docs/chat_architecture.md`; what is undecided is §18. One list each, beside the roadmap that
says why they were deferred, because two lists means one of them is wrong.

## What keeps going wrong here

Ten milestones of write-ups came down to about twenty mistakes, each made between three and
eleven times. They are worth reading before starting rather than after finishing; the case
behind each one is in `docs/history.md`.

**Half a mechanism passes its own tests.** A reader with no writer, eleven times so far —
`attached` set and never cleared, `external_session_id` recorded and never read,
`capabilities.interrupt` served and never called, `KernelStatus.detail` rendered nowhere, a
Shut down route with no button. The default that hides it is `allow`. Before believing a
feature exists, find both halves.

**A field that both routes and displays comes apart quietly, and the display side loses.** The
tell is a value compared in one place and rendered in another: a Module Run's title, a Pane's
`placeholder`, a dockview tab stamped at open.

**Ask of any "is it still going?" flag what writes the ending, and what happens when that
writer never runs.** A tool card spun for ever, a Run stayed `running` after its backend was
closed, a job's `stopping` was never cleared, a kernel never said `idle` again. A state derived
from the absence of an event never arrives.

**A default nobody chose is a decision nobody made.** `Store.history` defaulted to the earliest
thousand events and four callers wanted something else.

**Two values that have always been equal answer two different questions.** Ask which. A save's
`ETag` against a poll's `seen`; a Pane's `source_hash` against its `content_hash`.

**A sentence in a doc is not a mechanism.** Nine times a document described behaviour the code
did not have — layout scoped per Workspace, a Pane registry gated on a hash, dockview offering
no cancellable close. Read the code the sentence describes.

**Advice is not a gate**, recorded six times. A tool description, a diagnostic, a better-worded
refusal: an agent reads all of them and does what it was going to do. A Module adjudicates; a
message does not. Enforce where the tool is dispatched, not where it is asked about.

**When one of two implementations is right, the disagreement is the finding.** Four times a
bug was one of a pair, with the working half already pointing at the answer.

**Fixing a branch and leaving its neighbour**, four times. Interrupt and close, `_drive` and
`restart`, `read_object` and `read_artifact`.

**Driving the app finds what a green suite cannot.** Every milestone here ends with the same
sentence — nine hundred tests green, three defects found by opening the thing. Re-reading the
code you just wrote has never once found one of them.

**Audit a roadmap row before believing it**, and ask whether it names a capability or a
feature. "Change its model" sat in the list for two milestones with no control in the app.

**A pane rendering something proves the store can fold it, not that any backend produces it.**

**Ask what a test would prove, not whether it passes.** The vacuous family, all found here: a
spec asserting a value the framework was going to write anyway; a state already in the state
under test; a detached mount, where `.focus()` is a silent no-op; an `indexOf` assertion
satisfied by `-1`; a `<select>` whose value matches no option showing its first one; a fixture
whose payload failed to decode for an unrelated reason; a DOM read inside a mock, before Vue
has applied anything. A control that does not make the world change proves nothing.

**A survivor is a question, not a verdict.** Sometimes the answer is a test, sometimes the code
is dead and should be deleted, and seven times the mutation was a no-op that could not have
changed an answer. Read it before writing anything.

**A probe cannot tell absence from emptiness**, and a probe that cannot show its subject ran is
evidence of nothing. Run the obvious control first: four attempts to capture reasoning failed
before the variable turned out to be the model.

**An agent's report of a UI is a report of its own tool result.** It saw the payload, not the
screen.

**A killed parent leaves its children running.** `pnpm dev` reaps for this reason; a killed
uvicorn supervisor leaves a worker holding the port; two orphaned mutation runs raced over the
same files. Kill the tree, then confirm nothing survived.

**Check the staged diff for the hunk you are about to claim.** A commit once took the tests for
a fix and not the fix. And `git checkout --` is not an undo — nothing in the working tree comes
back.

**Colour, contrast and motion are only ever settled by looking.** No test here can see a screen.
A token that inverts with the theme is wrong for a scrim; a line colour chosen against one
surface is not chosen; `leading-*` on an inline element does nothing; a Tailwind class assembled
at runtime emits no CSS and no error.

## Before touching the Claude Code backend

**It asks, and the question is answerable** — `can_use_tool` parks the turn on
`agent.permission.request`, the pane renders it, the route resolves it, and an unanswered one
expires. Both backends reach the same events by different routes: the harness asks us, and the
API loop asks on its own behalf.

**Six things silence that callback, and none of them fails loudly.** The harness auto-approves
a read-only command inside its working directory; an `allowed_tools` entry; an allow rule in a
settings file; a deny rule, in the refusing direction; a `PreToolUse` hook; and *Allow for this
session*, which for a file write is always `setMode: acceptEdits` rather than a rule about that
file. All six were measured, and in each case the callback fires zero times while the turn
proceeds as though nothing had been decided. **So a permission log showing nothing for a tool is
not evidence the tool was never called.** §6 and §18 of `docs/chat_architecture.md` carry each
one, and `RunFacts.silenced_tools` reports the ones we cause.

**A Run a Module opened settles rather than asks**, because nobody is watching it: a tool the
Module handed over is allowed and anything else is denied, both appended to the log. Everything
else — the shell, the file primitives, nearly every native harness tool — still asks.

**`AskUserQuestion` is deliberately not a permission.** `_asker` intercepts it by name before a
`PermissionRequest` exists, so what the user is asked is the answer rather than whether the
agent may ask. Nothing may pre-answer one — no standing decision, no policy, no session grant.

---
> Source: [greentfrapp/panel](https://github.com/greentfrapp/panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-15 -->
