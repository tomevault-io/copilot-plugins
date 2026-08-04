## pixel-art-lab

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A local web app for benchmarking **any** OpenRouter model at agentic pixel art creation. It drives
[`pixel-mcp`](https://github.com/willibrandon/pixel-mcp) — an MCP server scripting a real Aseprite
install — through a tool-calling loop so the model draws, renders a preview, *looks at it*, and fixes
what's wrong.

Because the point is to *compare* models, anything that makes a model's job harder for reasons
unrelated to drawing (huge tool menus, undocumented API quirks, a two-step canvas creation dance) is
smoothed over by the server rather than left for each model to rediscover. That principle is why the
synthetic tools and the curated toolset exist; keep it in mind before exposing more raw surface.

## Commands

npm workspaces (`server`, `web`) at the root:

```sh
npm run dev          # server on :8787 + web on :5273 (vite proxies /api → 8787)
npm test             # vitest run in both workspaces
npm run typecheck    # tsc --noEmit in both
npm run build        # tsc -b (server) then tsc -b && vite build (web)
```

Single workspace / single test:

```sh
npm run test --workspace server
npx vitest run src/mcp/paths.test.ts --workspace server   # or cd server && npx vitest run paths
npx vitest run -t "rejects traversal" --workspace server  # by test name
```

Read back what a model actually did on a run (see "Run transcripts" below):

```sh
npm run transcript                    # the most recent run
npm run transcript -- <runId> --full  # full tool args/results; --reasoning adds reasoning text
npm run transcript -- --list          # runs, newest first
```

End-to-end smoke test against a real model (needs the server already running, costs money):

```sh
node server/scripts/smoke.mjs [model] [prompt]
# env knobs: PAL_BASE W H TURNS ITERS MAX_COST EFFORT
```

Check the MCP dependency is healthy before debugging a failing run:

```sh
./pixel-mcp/bin/pixel-mcp --health   # or wherever PIXEL_MCP_BIN points
```

## Requirements

Node 20+, Aseprite plus a built `pixel-mcp` binary with `~/.config/pixel-mcp/config.json` pointing at
the Aseprite executable, and `OPENROUTER_API_KEY` set. The key is read server-side only
(`server/src/config.ts`) and never reaches the browser — models are proxied through `/api/models` for
that reason. All tunables (budget ceilings, preview sizing, context-pressure thresholds, timeouts)
live in `config.ts`; prefer adding one there over hardcoding.

The server binds `127.0.0.1` explicitly, and that is not incidental: there is no authentication
anywhere, and the app proxies a key that spends money, starts runs on request and serves files out of
the run workspaces. On the default `0.0.0.0` all three belong to whoever shares the network. Making
it reachable from another machine means adding auth first, not widening the bind. Relatedly,
`/api/health` reports `ok` from whether the pixel-mcp binary is actually executable rather than from
the process being up — the first thing a fresh clone gets wrong is that path, and a spawn `ENOENT`
surfaced as run-end error text reads like the model broke rather than like setup is incomplete.

Environment comes from `.env` at the repo root (`cp .env.example .env`), loaded by the server's own
npm scripts via `--env-file-if-exists` — *if exists*, so a shell-exported key still works and a
missing file fails as `assertConfigured`'s clear message rather than an ENOENT. Node's parser does no
tilde expansion, so `~/...` in `.env` is a literal path; `PIXEL_MCP_BIN`, `RUNS_DIR` and `GALLERY_DIR`
are each resolved against the project root, so a relative value there means what it looks like
regardless of where the server was started from.

## Architecture

Request flow: browser posts a multipart form to `POST /api/runs` → `createRun` makes an isolated
workspace → the response returns `{runId}` **immediately** and `runAgent` runs detached → the browser
subscribes to `GET /api/runs/:id/events` (SSE). Events are buffered on the `Run` and replayed on
connect, so a late or reconnecting browser still sees the whole run.

### Run transcripts

`emitTo` appends every `RunEvent` to `runs/<runId>/events.jsonl` (preceded by a `run.request` header
line with the model and form settings), so a run outlives the browser tab and the server process —
the in-memory `Run.events` buffer only serves SSE replay. Each line carries `atMs`, elapsed run time;
`tool.result`'s own `ms` is the call duration, which is why the two keys differ.
`server/scripts/transcript.mjs` folds that file into a readable transcript. Alongside it,
`runs/<runId>/mcp.log` is pixel-mcp's own stderr — the place to look when a tool fails for reasons
the tool result does not explain.

### The benchmark gallery (`server/src/gallery.ts`, `web/src/components/Gallery.tsx`)

Saving an image **copies** it — plus the sprite, the whole `RunRequest`, and the run's tallies — into
`gallery/<entryId>/{image.png, sprite.aseprite, meta.json}`. It has to be a copy: `pruneOldRuns`
deletes the workspace it came from, and the point of a benchmark record is that it outlives the run.
`config.galleryDir` is deliberately a sibling of `runsDir`, not a child.

`summarizeEvents` folds the event stream into the `RunStats` the entry keeps. It prefers `run.end`'s
own tallies and falls back to the accumulated `usage` totals, because an image can be saved mid-run
from a preview. The events come from `snapshotRun` when the process still holds the run and from
`readRunSnapshot` (parsing `events.jsonl`) when it does not — the same reason transcripts exist.
Runs predating `events.jsonl` cannot be saved; the POST answers 404.

Both a run log and a saved entry outlive the code that wrote them, so both are read through a
compatibility shim: runs recorded before turns and iterations were separate counts emit
`iteration.start` per round-trip and a `run.end.iterations` that tallies round-trips (`fromLegacy`),
and entries saved then keep the turn tally in `stats.iterations`, the preview tally in
`stats.previews`, and the turn cap in `request.maxIterations` (`migrateEntry`, keyed off the absence
of `stats.turns`, applied on read so the file itself stays as the run produced it).

The prompt is the grouping key (`groupByPrompt`, case- and whitespace-insensitive), so two models given
the same words line up automatically and no one has to name a benchmark. "Run again" prefills
everything *except* the model — carrying the canvas and budget over is what makes the second answer
comparable, and changing the model is the entire point. `leaders` marks ties rather than breaking
them; claiming a single winner between two runs that both cost $0.004 would misreport the measurement.

Saved images are served from `/api/gallery/:id/image`, which does **not** need the run in memory —
unlike `/api/runs/:id/files/*`, which 404s for any run this process did not start.

### The agent loop (`server/src/agent/loop.ts`)

The core file. One **turn** = one model round-trip. One **iteration** = one `render_preview` the
model got an image back from — the look-and-fix cycle, and the unit of progress worth measuring;
a turn is only the round-trip that carries it. Keep the two words apart everywhere: the transcript,
the events and the form all distinguish them, and conflating them is the confusion this vocabulary
was introduced to end. Tool calls within a turn are executed **strictly sequentially** — pixel-mcp
offers no concurrency guarantees on a single file. Every failure mode is converted into a tool
*result* the model can read and self-correct from (bad JSON args, unknown tool name, path violation,
MCP error); only fatal OpenRouter errors and cancellation end a run early.

Termination: a turn with no tool calls means done, except that a vision model that never previewed
gets **one** nudge to look at its work first. A turn carrying *nothing* — no tool calls, no text, no
reasoning — is a dropped response wearing that same costume, so `isEmptyResponse` splits it out and
the turn is retried, ending as `end: empty_response` rather than a `done` that would score a broken
turn as a clean finish (this is how `e32b48be` came to report success on a blank canvas). What ends
the run is `config.emptyResponseLimit` blanks **consecutively** — `state.emptyStreak` is cleared by
any turn that carries something, because blanks scattered across a long run are hiccups the retry
already recovered from, not an endpoint that stopped answering. Each retry waits first
(`config.emptyResponseDelaysMs`, indexed by the streak), since re-asking instantly tends to land in
the same transient state that produced the blank. The retry loop sits **inside** the turn, and
`state.turns` is only advanced once a response actually arrives: a provider that drops a response is
not the model spending a turn, and billing the retries to the turn budget would silently shorten the
run the form asked for. So the attempts all report under one `turn.start` (its `n` is the turn the
answer will become), and their `usage` events still land, because every attempt is really charged.
Otherwise the run ends at whichever budget lands first.
`resolveLimits` reads all three off the form: `maxTurns` (defaulted from `defaultMaxTurns`, capped by
`turnCeiling`) plus the two opt-in ones, `maxIterations` and `maxCostUsd` — blank there means *no cap
of that kind* rather than a default, so a budget is always deliberate and two runs that asked for
none stay comparable. `toolCallCeiling` is an independent guard on top.

Budgets are checked **between turns**, never mid-turn, so a turn's edits all land together. Hitting
the turn or iteration limit buys one final tools-less turn to sign off; hitting the cost limit does
not, because that turn would spend past the number the limit named.

What the model knows of all this is one `budget` line (`budgetStatus`) appended to every successful
`render_preview` result — the system prompt states the totals once and then goes stale, and the
model cannot recover the count from its own history because pruning rewrites it. Previews carry it
because the look-and-fix cycle is the thing being paced, and because a per-turn reminder would move
the rolling cache breakpoint every turn (see "Prompt caching"). Only caps the run actually has are
named, for the same reason `budgetDirective` omits them. A failed render gets no line and is not an
iteration; the line is built *after* the tally and *before* the `tool.result` event, so the
transcript shows what the model read.

### Sandboxing (`server/src/mcp/paths.ts`)

Each run gets `runs/<runId>/` with `sprites/ exports/ reference/ tmp/`. `PATH_PARAMS` lists which
tool arguments carry paths and how each is used; `sandboxArgs` rewrites every one of them before it
reaches MCP. Absolute paths outside the workspace are re-homed by basename into the right subdir
(that is usually what the model meant), traversal and symlink escapes are rejected as `PathViolation`,
and extensions are validated per kind. **Adding a pixel-mcp tool with a new path parameter means
adding it to `PATH_PARAMS`** — unlisted args pass through untouched. `/api/runs/:id/files/*` applies
the same `isInside` check when serving.

### Synthetic tools

Four tools in the model's menu are implemented here, not by pixel-mcp:

- **`new_sprite`** (`mcp/newSprite.ts`) — fuses `create_canvas` + `save_as`. `create_canvas` takes no
  path and writes to a temp dir that gets wiped; it is in `ALWAYS_HIDDEN` so no model ever sees that
  dance.
- **`render_preview`** (`mcp/preview.ts`) — the verify loop. Copies the sprite (`scale_sprite` mutates
  in place and would destroy the master), scales the copy nearest-neighbour toward `previewTargetPx`,
  exports a PNG, and injects it as an image. `buildToolset` withholds it from a model without vision:
  the injected `image_url` part is a 400 from a text-only endpoint, so leaving the tool in the menu
  hands the model a way to kill its own run. A blind model gets no preview tool, no iteration cap in
  its prompt (it could never reach one) and is pointed at `get_pixels` instead.
- **`undo`** (`mcp/checkpoints.ts`) — restores the sprite to the state of a preview. See below.
- **`draw_pixels`** (`mcp/drawPixels.ts`) — keeps pixel-mcp's name and schema, but the call is
  wrapped rather than forwarded. See below.

**Undo is a file copy, and the preview is the checkpoint.** pixel-mcp is stateless — every drawing
tool ends in `spr:saveAs(spr.filename)` — so the `.aseprite` on disk is the entire state and copying
it captures pixels, layers, frames, palette and tags at once. There is no Aseprite undo history to
drive and nothing to keep in step. `Checkpoints.capture` snapshots into `tmp/checkpoints/` from the
`render_preview` branch of the loop, once per preview the model actually got an image back from,
labelled with that iteration number; `undo` copies one back.

The preview is the checkpoint because it is the only moment the model has *judged* the work — "put it
back how it was" is only meaningful about a state someone looked at — and because the image
describing the restored state is already in context, so the tool result needs only name the iteration.
That is also why `buildToolset` withholds `undo` from a blind model alongside `render_preview`: with
no previews it would take no checkpoints and could only ever answer "nothing to undo".

Two behaviours in `restore` are load-bearing. It compares the live file's hash against the newest
checkpoint and **steps one further back when they match**, because the newest checkpoint *is* the
current state whenever nothing has been drawn since — restoring it would report success and change
nothing, and a second `undo` has to keep walking backwards. And it **truncates** the stack at the
restored point, deleting the snapshots after it: those describe work that was just thrown away, so
leaving them would let a later undo land in a state the model had already rejected. The revert was
confirmed against a real Aseprite install (pixels, layers and palette all came back).

A restore also has to reach *context*, or the model is left looking at a picture of a canvas that no
longer exists — see `dropUndonePreviews` under "Context management". The `undo` event carries
`restoredTo` so the browser can mark those frames too; the iteration tally deliberately does not go
back, because an undone preview was still looked at and still paid for.

**The cel trap and why `draw_pixels` is wrapped.** pixel-mcp's `draw_pixels` addresses the layer's
*cel*, not the canvas: coordinates are cel-relative, anything outside the cel's bounding box is
discarded, and `pixels_drawn` counts it anyway. Every other drawing tool is canvas-accurate, so it
fails silently in both directions. This used to be documented in the system prompt with a
pin-the-origin workaround, which turned out to be worse than nothing — pinning `#00000001` at (0,0)
on an *empty* layer yields a 1x1 cel, so everything is clipped. Run `e32b48be` followed that
instruction exactly, drew 340 pixels into a 1x1 cel, rendered a blank canvas and gave up; run
`8b7f8a4c` only survived because it blocked in with rectangles first, which had already grown the
cel. The models that read the docs most literally were the ones it broke.

The wrapper pins **both** corners — origin and far corner, which is what actually makes the cel span
the canvas — forwards the call, then removes the pins. Removing them matters: `apply_outline` treats
an alpha-1 pixel as opaque and paints a blob around it, and a pin left behind would follow the
sprite into the gallery. A corner that already holds art is left alone, and a corner the model drew
on itself is not cleaned up. Off-canvas pixels come back as `skipped_off_canvas` plus a note rather
than vanishing. `drawPixels.test.ts` fakes the cel-clipping behaviour, with one test asserting the
fake still reproduces the bug so the rest cannot pass for the wrong reason.

`mcp/toolset.ts` also curates the menu: `NON_CORE` (selection/clipboard, destructive, low-value
analysis tools) is dropped in the default `core` toolset because ~50 schemas is a per-turn token tax
that measurably degrades weaker models. `matchToolName` recovers names emitted with wrong case,
a namespace prefix, or dashes for underscores.

### Context management (`server/src/agent/history.ts`)

Preview images are injected as **user** messages after all tool results, never inside tool results —
a tool message must immediately follow the assistant's `tool_calls`, and image-in-tool-result support
varies by provider. Older previews decay to one-line text placeholders (`keptPreviews`, tightened
when `promptTokens / contextLength` crosses `contextPressureThreshold`). User-supplied reference
images are never pruned.

A preview image can go for two reasons that mean opposite things, so `pushPreview` stores a
*description* and `collapse` supplies the reason. `prune` drops images for age. `dropUndonePreviews`
drops the ones an `undo` invalidated: the model has just restored an older state, and the newest
image it can otherwise see is the state it asked to be rid of. A note in the tool result does not
compete with an image — same lesson as the cel trap — so the image goes and the line of text stays,
because a mistake the model cannot recall is one it draws again.

Both ends of that range matter, and both are about a single turn's ordering. The loop applies it
*after* the turn's pending previews are pushed, because a turn that previews and *then* undoes still
has the discarded image sitting in `pendingPreviews` — the one picture most likely to mislead the
next turn. And the range is closed at the top (`state.iterations` as the call ran), because a turn
that undoes and *then* previews to confirm has produced the only accurate image in the conversation,
which an open-ended drop would take. Ranges are kept separate rather than merged for the same
reason: a preview between two undos survives both. And the drop is scoped to the undone sprite —
iterations are numbered globally but an undo rewinds one file, so another sprite's preview inside
the range is still true of its canvas (the browser fold filters on the event's `spritePath` for the
same reason).

`toArray(cacheBreakpoints)` applies two **static** cache breakpoints to a shallow copy — the stored
messages stay unmarked so marks cannot accumulate or reach the transcript. The first is the system
prompt (fixed for the run; tools serialise ahead of it, so one marker covers the whole tool menu).
The second is the *pruning frontier*, and it exists because collapsing rewrites previews **in
place**: any edit invalidates every breakpoint at or after it, so on a turn that collapsed anything,
the rolling tail marker is dead. What survives is the end of the **contiguous** run of collapsed
previews — nothing before it can change again. The contiguity is load-bearing and easy to lose:
age-pruning alone only walks forward, which made "the newest collapsed index" an equivalent and
simpler answer, but `dropUndonePreviews` collapses the *newest* previews while older ones are still
live images, and that answer would put the breakpoint past previews `prune` has yet to rewrite. If
you make anything else rewrite messages, `cacheFrontier` is the invariant you are breaking.

### System prompt (`server/src/agent/prompt.ts`)

Built per run from the request. The "rules that will bite you" section documents real pixel-mcp
quirks: 1-based frames, `layer_name` being mandatory, `export_sprite`'s `frame_number: 0`. It also
branches on `hasVision`: a blind model is told to use `get_pixels` instead.

If you learn a new quirk from a failing run, first ask whether the server can absorb it — a quirk
documented here only helps models that follow instructions correctly, and the cel trap that used to
live in this section is the case study for why that is the weaker fix. Document it only when it
cannot be handled in `mcp/`, and describe the behaviour the model will actually observe.

### OpenRouter (`server/src/openrouter/`)

`models.ts` reduces the catalogue to the flags this app decides on — `supportsTools` is a hard filter
(a model that can't call tools can't drive Aseprite), `supportsVision` gates the preview loop and the
nudge, `supportsReasoning` gates sending the `reasoning` field at all (others 400 on it). `stream.ts`
does one streamed round-trip with backoff retry; 400/401/402/404 are `FatalOpenRouterError` and are
not retried. Tool-call argument fragments arrive keyed by `index`, not `id`, and several calls can be
open at once — hence the accumulate-then-assemble in `consume()`.

**Prompt caching.** `supportsCaching` / `needsCacheBreakpoints` are derived from *pricing*, not from
a feature list: a quoted `input_cache_read` means the provider discounts cached input, and a quoted
`input_cache_write` means it caches **only** where you mark it (Anthropic, Alibaba, Google) rather
than automatically (OpenAI, DeepSeek, Groq, xAI). A run resends an ever-growing prompt a hundred
times, so this is most of the input bill for the first group. When `config.promptCaching` and
`needsCacheBreakpoints` both hold, the request carries a top-level `cache_control` — OpenRouter reads
that as "breakpoint on the last cacheable block", a rolling tail marker renewed each turn — on top of
the static markers `History` places. Measured on claude-haiku-4.5 at a 19k-token prompt: $0.0240
cold, $0.0022 warm. `PAL_PROMPT_CACHE=0` disables it to measure the uncached baseline, which matters
because cost is a benchmark axis and a cached run is not comparable to an uncached one. `usage`
events carry `cachedTokens` / `cacheWriteTokens`; the status bar shows the hit rate and
`npm run transcript` prints both.

### Web (`web/src/`)

React + Tailwind, no router or state library. `App.tsx` holds the raw `RunEvent[]`; `lib/events.ts`
folds it into a `Timeline` (merging consecutive deltas, joining `tool.result` back onto its
`tool.call`) so components stay dumb and the merge logic stays unit-testable.

`web/src/types.ts` **mirrors `server/src/types.ts` by hand** — there is no shared package. Change one
and change the other, or the SSE contract silently drifts.

## Tests

Vitest, colocated `*.test.ts`, covering the pure logic: path sandboxing, toolset filtering/name
matching, the `draw_pixels` cel wrapper against a fake that models the clipping, history pruning,
SSE chunk parsing, the browser-side timeline fold, the run-stats summary and prompt grouping. The
`undo` checkpoints are covered against real files in a temp dir — sprite contents are plain strings
there, which is enough because the revert is a byte-for-byte copy either way.
Anything requiring a live model is exercised by `smoke.mjs` instead.

Aseprite-dependent behaviour has no standing harness, so when you change something in `mcp/` that
depends on how Aseprite actually behaves, drive it directly: build, then import `PixelMcp` and the
function under test from `dist/` in a throwaway script and assert with `get_pixels`. That is how the
cel-clipping fix was confirmed, and it costs nothing.

The UI can be exercised without spending anything by replaying a past run: serve a finished
`events.jsonl` as SSE on `:8787` (and answer `POST /api/runs` with its runId), run the real server on
another port behind it, and drive the browser at vite as usual.

---
> Source: [nbrown725/pixel-art-lab](https://github.com/nbrown725/pixel-art-lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
