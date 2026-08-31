## qwaude

> `qwaude` runs Claude Code against a local Qwen 3.8 27B on Apple Silicon. It is a

# qwaude — agent context

`qwaude` runs Claude Code against a local Qwen 3.8 27B on Apple Silicon. It is a
single self-contained bash script; the interesting engineering is in the choices
below, most of which were arrived at by measurement, not preference. Read this
before changing anything — several "obvious improvements" were tried and lost.

## Hard constraints (don't break these)

- **One file, stock `/bin/bash` 3.2.** A bootstrapper runs before its
  dependencies exist; a fresh Mac has no Python, Ruby, jq, uv. No associative
  arrays, no `mapfile`, no `${var,,}`, no `[[ -v ]]`; empty arrays under
  `set -u` need `${arr[@]+"${arr[@]}"}`. Test under `/bin/bash`, never
  Homebrew bash. After oMLX is installed, its bundled Python
  (`$(brew --prefix)/opt/omlx/libexec/bin/python`) may be used for JSON/Jinja.
  Perl is also stock on macOS. `timeout` is NOT (use background + poll + kill).
- **Prerequisites stop-and-instruct, never install:** Apple Silicon, Homebrew,
  Claude Code. Missing → print the official install command, exit 1.
- **Flags are exactly** `--speed 4bit|8bit`, `--effort low|medium|xhigh|off`,
  `--uninstall`, `-y`, `-h/--help`. Everything else passes through to
  `claude` (`-r`, `-c`, …).
- **`--uninstall` is provenance-aware.** Setup appends to a manifest
  (`~/.qwaude/installed`) only for components qwaude itself installed;
  uninstall offers those (default Y), skips ones that pre-existed ("already on
  your machine before qwaude"), and with no manifest treats provenance as
  unknown (defaults N for uv/oMLX/LiteLLM, Y for the models). Homebrew and
  Claude Code are never touched. It refuses to run while a qwaude server is
  up. Keep the manifest honest when adding install steps.
- **`--uninstall` has to leave nothing behind** — the README promises "remove
  all traces", so every new artefact needs an offer. Order: oMLX / LiteLLM /
  uv / both model dirs → `~/.omlx/{cache,logs}` → the rest of `~/.omlx` →
  `brew autoremove` → `~/.qwaude` + /tmp state → the script itself. Three
  things that are easy to get wrong here:
  - **Show the real size, not a constant.** `dir_size_h` (`du -sh`) goes in
    the cache/logs/`~/.omlx` prompts. The oMLX cache reaches tens of GB (57 GB
    on the owner's machine) and is invisible otherwise — the number is the
    whole argument for the prompt existing.
  - **Two prompts deliberately default N**: `brew autoremove` (brew-wide; it
    prints the exact formula list from `--dry-run` first, and is skipped
    entirely when that list is empty) and deleting the script. `-y` accepts
    defaults, so `-y` does neither. That is the intended reading of `-y`.
  - **`~/.omlx` wholesale** is offered only when `models/` is empty *and*
    `omlx_settings_only_ours` confirms `model_settings.json` names no model
    but ours (no Python to read it with ⇒ can't tell ⇒ don't offer). The
    script is found via `command -v`, verified by grepping `^QWAUDE_VERSION=`
    in it before any `rm` — never trust the name alone — and a symlink is
    removed as a symlink, never followed into the user's checkout.
- **One hidden knob only:** `QWAUDE_INITIAL_DRY_RUN=1` replays the full first
  run (fake timed downloads, prompts, install steps) writing nothing, and ends
  exactly where a real first run ends — "Installation is complete! Now run
  qwaude", exit 0, no servers, no session. The owner explicitly rejected any
  other env switches and `NO_COLOR`; plain output is automatic when stdout
  isn't a TTY. Don't add knobs.
- **Setup never rolls into a session.** `run_setup` does not return: it prints
  `✓ Setup complete.`, a blank line, then `Installation is complete! Now run
  qwaude` and exits 0. That applies to any run where `setup_needed` was true,
  including one where only the quiet steps (template, settings, proxy config,
  Tavily key) had work to do. Only a launch with nothing to set up reaches
  `start_server`. The self-check that used to run in `main` after setup now
  lives at the end of `run_setup`, before those lines.
- First run downloads **both** quants (4-bit 16.6 GB, 8-bit 29.3 GB) so
  `--speed` switches instantly later. Everything is idempotent, and the
  checklist reflects it: components already present are listed as ✓ under
  "Checking for prerequisites"; "We are about to set up" shows only real work.
- **The weights come down over `curl`, not oMLX's bundled `hf`.** `hf` only
  exists after `brew install omlx` has built 3.2 GB, so on a genuinely fresh
  Mac the download could not begin until minutes after "Proceed? y" and the
  progress bar under the Tavily prompt never appeared — it only ever worked in
  demo mode or on a machine that already had oMLX. Nothing in the download
  path may depend on an installed component again.
- The first-run UI wording (banner, "First run, initializing...", "Checking for
  prerequisites", "We are about to set up", the "Quick start" paragraph, the
  unindented "Download total" / "Proceed? [Y/n]" lines, the Tavily steps and
  paste prompt) is owner-specified verbatim. Don't reword it.

## Architecture

```
Claude Code ──ANTHROPIC_BASE_URL──▶ LiteLLM proxy :4002 ──▶ oMLX server :8092
              (effort router + Tavily          (paged KV, MTP k=3,
               web_search interception)         both quants resident)
```
- **oMLX** (`brew tap jundot/omlx`, HEAD build with `--with-custom-kernel`)
  serves both models from `~/.omlx/models`; they lazy-load on first request
  and stay resident. State in `/tmp/qwaude-8092`; refcounted via client PID
  files — first qwaude starts it, last one out stops it.
- **LiteLLM** (`uv tool install "litellm[proxy]"`) config and the effort router
  are embedded heredocs written to `~/.qwaude/`. Refcounted the same way
  (`/tmp/qwaude-litellm`). Started in parallel with the model server's warm-up.
- **Effort router** (`~/.qwaude/effort_router.py`, a LiteLLM pre-request hook):
  Claude Code always sends `output_config.effort`; the router maps
  low/medium/xhigh/max(→xhigh) as pegs and treats **`high` as the AUTO
  sentinel**: tool-result turns → `low`, user text containing "think hard" →
  `xhigh`, else `medium`. It injects `chat_template_kwargs.reasoning_effort`
  (oMLX honors it per request), strips `output_config`/`thinking`/
  `context_management`, runs Claude Code's session-namer call with thinking
  off, canonicalizes tool schemas (see debugging), and appends decisions to
  `~/.qwaude/effort.log`. The hook finds the system prompt in
  `kwargs["proxy_server_request"]["body"]["system"]`, not `kwargs["system"]`.
- **Model download** (`dl_*` in the script): the file list and exact byte sizes
  come from `https://huggingface.co/api/models/<repo>/tree/main?recursive=true`,
  parsed by perl (strip the nested `"lfs"` objects first, or its `size` key is
  mistaken for the file's). That listing is written into the model dir as
  `.qwaude-files` *before* the first byte lands, which is what makes
  `model_present` honest: a directory counts as present only when every listed
  file is there at its listed length (a dir with no `.qwaude-files` is a
  hand-made/legacy one and falls back to "config.json exists"). Each file is
  fetched with `curl -fL -C - --retry 5 --retry-delay 3`, three at a time, so
  an interrupted run resumes; curl is re-invoked if it returns short, and a
  file longer than advertised is deleted rather than resumed into. The bar's
  denominator is the API's byte total, and progress is `du -sk` of the dir in
  flight plus the models already finished. Ordering in `run_setup`: download
  starts immediately after `confirm` → Tavily prompt with the bar anchored
  beneath it → brew/uv/LiteLLM install *while the download continues* (the bar
  stands down rather than fight brew for the terminal) → `dl_wait` → template
  patch + settings per model. The download never waits on the runtimes.
- **Chat template**: froggeric's Qwen-Fixed-Chat-Templates + our "tail patch"
  that moves the reasoning-effort instruction from the system head to just
  before the generation prompt. Result: effort changes never invalidate the
  prompt-cache prefix. Stock template backed up as `chat_template.jinja.stock`.
- **Claude Code launch hardening** (all measured to matter):
  `CLAUDE_CODE_ATTRIBUTION_HEADER=0` (header changes per request → cache
  miss every turn), `--exclude-dynamic-system-prompt-sections`,
  `CLAUDE_CODE_MAX_CONTEXT_TOKENS=262144` + `AUTO_COMPACT_WINDOW`,
  `--settings '{"awaySummaryEnabled":false,"effortLevel":"high"}'` (recaps
  burned ~8 s per idle; effortLevel forces the auto sentinel even if the
  user's settings pin something else), `--effort auto`,
  `unset CLAUDE_CODE_EFFORT_LEVEL`, `API_TIMEOUT_MS=3000000`, and
  `--disallowedTools` for a lean set (Workflow, ScheduleWakeup, Cron*,
  NotebookEdit, ReportFindings, TaskCreate/Update/Get/List — never Agent,
  SendMessage, worktrees, TaskOutput/TaskStop: the owner delegates to
  subagents). WebSearch is disallowed only when no Tavily key exists.

## Decisions and why (measured on an M5 Max, 128 GB)

- **oMLX over MTPLX.** MTPLX has faster short-context decode but an
  irreducible 5–16 s/round session-state tax at 32K+ context (KV snapshot/
  restore) plus serial scheduling that 503s Claude Code's side-calls. Same
  tool-heavy question: 25 s on oMLX vs 76–125 s on MTPLX.
- **ANE prefill is OFF** (`qwen35_ane_prefill_enabled: false`). The
  community's +57% was M3 Max; on M5 Max it measured *slower* (357 vs 626
  tok/s GPU-only). Kernel is built anyway for future oMLX versions.
- **4-bit default.** Benchmarks tie the "e" quants on quality; 8-bit costs
  ~2–3× on decode and on every cache miss. Users judge quality themselves.
- **Responses-API lane** (LiteLLM's default for OpenAI-style backends) vs
  chat-completions: measured tie; kept default. Escape hatch:
  `litellm_settings: use_chat_completions_url_for_anthropic_messages: true`.
- **No server linger.** Owner prefers servers stop when the last session
  exits; oMLX's SSD-paged KV cache survives restarts, so a same-prefix
  session re-prefills in seconds anyway.
- **Physics you can't tune away:** decode is memory-bandwidth-bound; ~30–40
  tok/s is the M5 Max ceiling for a dense 27B (20–31 at 32K context). Only
  MoE models beat it locally.

## Expected performance (so you know what "slow" means)

Cold start: server ~3–6 s + model load ~3–7 s + first prefill. A ~30K-token
Claude Code prompt prefills at ~600 tok/s (4-bit) → ~50 s **only when the
cache misses**; with a warm SSD cache the first reply lands in ~10 s. Warm
turns 5–15 s; tool rounds add ~1–3 s each. Decode 30–42 tok/s short context,
20–31 at 32K. If numbers are far off these, debug — don't tune blindly.

## Debugging "it's slow" — the playbook that found every real problem

1. **Where did the first prompt's time go?** oMLX logs tokens actually
   prefilled per request:
   `grep "primed=" ~/.omlx/logs/server.log | tail`
   `primed≈30000` = cache miss (full prefill); `≈100–2000` = cache hit;
   mid-size (e.g. 12K) = the prompt *diverged* partway — something before
   that point changed between sessions.
2. **Per-request timing/decode:** `grep -E "Responses API|Chat completion|MTP\[" ~/.omlx/logs/server.log | tail` — tok/s, MTP acceptance (~80% is normal;
   drops on prose), and whether requests queued behind each other.
3. **Is auto-effort doing what you think?** `tail -f ~/.qwaude/effort.log`
   shows `effort_in=high -> low|medium|xhigh|off` per request. `effort_in`
   other than `high` means the client pegged it. No lines = router not loaded
   (proxy config drift, or the proxy needs a restart to pick up router edits —
   it's loaded at start). Test effort with a hard multi-step question; trivial
   prompts produce noise-sized thinking and will mislead you.
4. **Prefix divergence hunt.** If `primed` is mid-size on back-to-back
   sessions: capture two sessions' first requests and `cmp` them, then map the
   first differing byte to system / tools / messages. Known culprits found
   this way: an MCP server (chrome-debug) emitting tool `input_schema`
   properties in a different key order each launch (fixed generically by the
   router's tool-schema canonicalization); Claude Code auto-updates rewriting
   the system prompt/tool text (one full prefill per update — unavoidable);
   the attribution header (now off). A zero-GPU way to capture a real Claude
   Code request body: point a throwaway LiteLLM at a dead backend port and
   run `claude -p "hi"` through it with the same env — the hook fires before
   the backend call fails.
5. **Startup chain:** the proxy starts in parallel with the server wait, and
   an eager 1-token request triggers model load during Claude Code's own
   boot. If a start is slow, check `/tmp/qwaude-8092/server.log` timestamps
   from "Server initialized" to the first `MTP path activated`.
6. **Wedged state:** `rm -rf /tmp/qwaude-8092 /tmp/qwaude-litellm`, and check
   `lsof -nP -iTCP:8092 -sTCP:LISTEN` for an orphaned `omlx-server`.
7. **Don't disturb a live session:** a user may have qwaude running; never
   send generation requests to its ports or restart its servers while testing.
   Use `QWAUDE_INITIAL_DRY_RUN=1` for UI work; for backend tests start your
   own server on a different port.

## Things that bit us (bash 3.2 specifics)

- `local a="$1" b="$a/x"` fails under `set -u` — `local` expands all args
  before assigning any. Use separate statements.
- `trap f EXIT INT TERM` re-runs `f` on EXIT after the INT handler; guard
  handlers for idempotence.
- LiteLLM resolves callback modules relative to the config file's directory;
  keep `effort_router.py` beside `litellm.yaml`.
- The demo mode must never write: it reuses the real installer code paths
  behind a `SIM` guard — keep new install steps behind that guard too. Its
  setup scratch is `$SETUP_SCRATCH` (`/tmp/qwaude-demo-$$` under `SIM`, the
  real `$STATE_DIR` otherwise) because `dl_queue_reset` does `rm -rf` on it:
  running the demo while a real first run is downloading in another terminal
  used to wipe that run's progress state and freeze its bar at 0%.
- **Nothing setup runs may ask the user a question.** Since Homebrew 5,
  `brew install` prints its plan and prompts "Do you want to proceed with the
  installation? [y/n]" whenever dependencies are involved — that stopped a
  real first run dead. `brew_run` sets `HOMEBREW_NO_ASK` (the documented off
  switch), plus `NO_AUTO_UPDATE`, `NO_ENV_HINTS`, `NO_INSTALL_UPGRADE`,
  `NO_COLOR`; `install_watch` additionally redirects both streams to
  `$INSTALL_LOG` and feeds stdin from `yes` (Homebrew also skips the prompt
  when stdin/stdout isn't a TTY, so that alone would do it — belt and
  braces). Do NOT set `HOMEBREW_NO_REQUIRE_TAP_TRUST`; it isn't needed,
  because `brew install jundot/omlx/omlx` is fully qualified and Homebrew
  trusts such a formula on the spot. Tap-trust warnings about the user's
  other taps go to the log, unread.
- **The install phase owns two rows**, drawn by the foreground (no background
  renderer): the component's status line — `▸ Installing oMLX runtime ⠹ 2m10s
  (<last "==>" line from the log>)` — and the download bar underneath it, via
  `\n`, bar, `ESC[1A`. Neither may wrap or the cursor arithmetic breaks, which
  is why `BAR_W` shrinks when `tput cols` is under 84. `dl_wait` re-prints the
  "Downloading both model builds …" headline when it takes the bottom row back
  so the bar is never left bare.

---
> Source: [keithschacht/qwaude](https://github.com/keithschacht/qwaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
