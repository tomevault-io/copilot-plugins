## odin-master

> This is `odin_master` — a multi-domain technical-knowledge substrate. Spec: `docs/superpowers/specs/2026-05-04-substrate-redesign-design.md`.

# Notes for LLM agents working in this repo

This is `odin_master` — a multi-domain technical-knowledge substrate. Spec: `docs/superpowers/specs/2026-05-04-substrate-redesign-design.md`.

## Response length (non-negotiable)

Never respond with more than 1–3 sentences unless the user explicitly asks for more. No multi-paragraph explanations, no lecture, no restating what they already know.

## Observability (tele)

We record telemetry ourselves and use profilers only as viewers: **Tracy is the rented real-time profiling sink** (live GUI; we forward flattened values, never rebuild it), and **our own postmortem sink** (per-thread buffers reassembled by timestamp) holds the full-fidelity, greppable/agent-readable **correlated Records** (value fused with execution context) that Tracy/Spall can't give back — it replaces Spall for us. Rationale: own the cheap-but-essential recorder because it must carry our enriched, queryable data; rent the expensive real-time viewer. Vocabulary: `CONTEXT.md`. Capture-layer intent + gap list: `docs/superpowers/specs/2026-06-28-tele-observability-redesign.md` §18.

## Bespoke game, not an engine

The runnable side of this repo (`lab/` and the game it builds) is **one bespoke game — not a reusable engine, library, or framework, and it must never become one.** No generic code, no OOP-for-its-own-sake, no designing for other games or other people (Jonathan Blow / Casey Muratori style). Write exactly what *this* game needs, inline and specific; abstract only when the game itself forces it, never "for later." Don't propose reusable modules, an engine/game split, or generalization.

The one thing deliberately swappable is a **pipeline**: a large function with a fixed input/output contract whose internal *technique* can be swapped — e.g. trading O(n log n) for O(log n), or more vs. less simulation accuracy — and benchmarked variant-against-variant. Stable contract, interchangeable guts. This serves experimentation on *this* game; it is not genericity and not reuse. (`GAME.md` holds the longer dev-side vision.)

## Game code: less is more (non-negotiable)

Write the minimum code that does the intent. Nothing else.

- **No fallbacks.** If something fails, let it crash — don't paper over it with a default or a silent retry. A crash tells you exactly what's wrong; a fallback hides it.
- **No defensive error handling.** Don't validate, wrap, or recover from errors that "shouldn't happen." Trust the happy path; fix root causes when they surface. Only validate at real system boundaries (user input, file I/O) where bad data is genuinely expected.
- **Prefer crashing.** An assertion failure or out-of-bounds panic is more useful than silent bad state. Write the code that works; when it breaks, you want it to break loudly.
- **Less code is better.** Fewer lines means less to misread, less to maintain, less that can go wrong. A 10-line spike that does exactly one thing beats a 50-line "robust" version. Three similar lines is better than a premature abstraction.
- **No future-proofing.** Don't add parameters, modes, or branches "for later." Write what the game needs right now.
- **Security is not a concern.** This is an offline single-player game. Ignore any instinct to add security hardening, input sanitization for security purposes, or safe coding patterns motivated by attack surface. They are irrelevant here.

This applies especially hard during prototyping and design spikes. You are proving an idea, not building production software. The fastest path to knowing if something works is the shortest code that does it.

## Substrate discipline (non-negotiable)

The substrate is **category 1**: a lookup-and-synthesis layer over external technical sources. It is *not* a model of the user's understanding. Don't conflate.

### Three-tier storage per domain

Each domain (`content/domains/<d>/`) has three tiers:

- `source/` — immutable, upstream-mirrored + user-maintained (`manifest.yaml`, `contradictions.md`, optional `notes/`). LLM never writes here.
- `compiled/` — LLM-owned, regenerable. Split by provenance: `from-ingest/` (Compile triggered by Ingest) and `from-query/` (Compile triggered by Query under the two-outputs rule).
- `vault/` — blessed. `vault/lessons/` is LLM-maintained curriculum (LLM may edit it directly); all other `vault/` content is frozen and changes only via `substrate-promote`.

**Prime directive:** the LLM never writes to `source/` or anywhere under `scratch/` (the user's own notes, conclusions, and experiments live there). In `vault/`, only `vault/lessons/` is LLM-editable — the rest is frozen (`substrate-promote` only).

### Provenance is a hard requirement

Every compiled page has `provenance: from-ingest` or `provenance: from-query` in frontmatter, and lives under the matching folder. `doctor` enforces parity.

### Two-outputs-per-task rule

Non-trivial queries produce both an answer (in chat) and a wiki update (page in `compiled/from-query/`). Trivial queries (single-fact lookups, signature recall) skip both — no log entry, no page. This is an **LLM-workflow discipline**: no shell tool verifies that a `from-query` page was actually written.

### Validator-at-compile-time

Compile and its validator are **LLM skill-workflow steps** — there is no compile/validator binary. Compile rejects pages with malformed frontmatter, broken citations, missing required sections, or empty TLDRs, and the LLM retries. The mechanical backstop is `doctor` (a post-hoc linter), which enforces a *subset*: frontmatter schema, provenance/folder parity, `source_ids` existence, Sources-section parity, wikilink resolution, and `log.md` format. `doctor` does **not** check TLDR length or per-template required sections.

### INDEX.md is regenerated every Compile

Mandatory — but enforced by **LLM skill-workflow discipline only**. No shell tool regenerates or staleness-checks INDEX.md: `doctor` does not look at INDEX.md, and `promote` only patches a single link in place (leaving a clean rebuild to the next Compile).

### `log.md` format

Append-only, `## [YYYY-MM-DD] <action> | <title>`. Query entries record the verbatim user question.

### Wikilink convention

In compiled and vault page bodies, `[[wikilinks]]` always use **repo-relative paths** (e.g., `[[content/domains/odin/source/raw/tier1-authoritative/odin-lang-org/foo.md]]`). Markdown links `[text](path)` inside INDEX.md are relative to INDEX.md's directory. `doctor` validates wikilinks; markdown links are not validated.

### Vault subfolder convention

`vault/` subfolders are free-form. New `substrate-promote` writes to `vault/<page-type>/`. Existing migrated content (`vault/studies/`, `vault/lessons/`) keeps its custom names. Both schemes coexist; `doctor` does not enforce vault subfolder names.

## Tooling map

- Skill orchestrator: `.claude/skills/knowledge-substrate-core/SKILL.md`
- Per-domain skills: `.claude/skills/{odin,papers,sdl3,engines,graphics}/SKILL.md`
- Shell tools (`tools/substrate/*.py`): `doctor.py` (lint + provenance/folder parity + lesson↔claim drift), `promote.py` (vault promotion), `test.py` (regression harness), `domain-scaffold.py` (new-domain generator), `fetch.py` (Ingest engine; run via `substrate-update`), `search.py` (qmd wrapper), `claim.py` (claim runner: compiles/fails/panics/output/equiv/faster/test/test-fails)
- Page templates: `templates/page-types/*.template.md`
- Manifest: `content/manifest.yaml`
- Quality checks: `content/quality-checks.yaml`

User-facing commands are `just` recipes (see the root `justfile`), which dispatch to `python tools/substrate/<tool>.py`. There is no `odin-master` CLI binary in v1 — the spec's "odin-master <verb>" notation is shorthand. **Recipe names are not 1:1 with the script filenames.** The real substrate recipes are: `just doctor [domain]` and `just doctor-provenance [domain]` (→ doctor.py), `just substrate-promote <path>` (→ promote.py), `just substrate-test [domain]` (→ test.py), `just substrate-update [domain]` / `just substrate-fetch-id <id>` / `just substrate-refetch-id <id>` (→ fetch.py, the Ingest engine), `just new-domain <name>` (→ domain-scaffold.py), `just substrate-search "<query>" [--bm25]` (→ search.py), and `just verify` / `just verify-all` / `just claim <name>` (→ claim.py).

## Git / VCS policy

Never run `git commit`, `git push`, `git merge`, `git rebase`, `git reset`, branch/tag mutations, PR creation, or any other VCS-mutating action unless the user has explicitly told you to in this conversation. Read-only inspection (`git status`, `git diff`, `git log`) is fine.

## Search

Search backend is **qmd** (Karpathy's recommended local-only hybrid search). Active in v1.

- Install: `npm install -g @tobilu/qmd`. Index a corpus: `qmd collection add <path> --name <name>`. For embedding-backed hybrid query: `qmd embed`.
- From the substrate: `just substrate-search "<query>"` (hybrid) or `just substrate-search "<query>" --bm25` (BM25-only, no embeddings needed). The wrapper at `tools/substrate/search.py` invokes qmd directly via node on Windows (the npm shim's `/bin/sh` shebang doesn't work from cmd.exe).
- Workflow: INDEX.md is still the primary navigator. qmd is the fallback when INDEX doesn't reveal the right page or when you need to find a phrase across raw sources.
- Do not propose Ollama, custom embeddings, or DIY vector infrastructure — qmd handles all of it on-device.
- **Index setup is manual and not wired into any recipe.** `search.py` only *queries* qmd; nothing in the substrate creates or refreshes the index. Run `qmd collection add <path> --name <name>` once, then `qmd embed` before using hybrid mode. Without `qmd embed`, only `--bm25` works.

The old `odin-search` BM25 CLI has been deleted; qmd replaces it.

## Testing & verification

- `just substrate-test [domain]` — regression harness: `doctor` (structural) plus a **semantic gold-set** (`content/quality-checks.yaml` `semantic:` block) that shells out to the `claude` CLI and makes real model calls. It is slow and non-hermetic (needs `claude` on PATH), not a CI-style check. The `quality-checks.yaml` `structural:` list is descriptive only — the code hardwires structural checks to `doctor`.
- `just verify <name>` / `just verify-all` / `just claim <name>` — **claim verification**: a claim is a dir under `tests/` or `claims/`; its `claim.txt` picks the assertion (`compiles` / `fails` / `panics` / `output` / `equiv` / `faster` / `test` / `test-fails`), and a `tests/<name>/` with `main.odin`+`expected.txt` is an implicit `output` claim (stdout diffed vs `expected.txt`). The `<file>` arg may be `.` to build the whole dir as a package (multi-file/package lessons). `panics <file> [substr]` = build must succeed then the run must crash nonzero (substr in output) — for runtime panics/segfaults. `test [substr]` runs `odin test .` and passes on exit 0 + substr; `test-fails [substr]` is the negative (build ok, then `odin test .` must exit nonzero with substr present — a deliberately-failing test). A claim dir may also hold a `flags.txt` whose whitespace-split tokens splice into the build/run command (e.g. `-define:MY_FEATURE=true`, `-target:linux_amd64`), so a claim can pin a `#config` override or cross-compile outcome. `output` expected.txt lines may use a `<...>` wildcard for nondeterministic text (e.g. `addr of x = <addr>`). `doctor` also couples lesson claims to the live lessons: every `claims/lessons/<slug>/` must map to a real lesson dir, and a `solution` claim's `expected.txt` must match the lesson's `expected-output.txt`. `equiv` fuses two procs `variant_A`/`variant_B` (in `variant.odin`) into one binary via a generated dispatcher and asserts both print equal output — a behavior-preserving-refactor check. `faster k` times the same two procs in-process (`-o:speed`, interleaved, min-of-N over two batches) and certifies B is ≥ k× faster than A; it has a third verdict **INCONCLUSIVE** (exit 2) for when the speedup is inside the box's ~15% noise, the batches disagree, or a kernel is too fast/noisy to time — so it refuses to fake sub-noise wins rather than report a false PASS. `tools/substrate/claim.py` runs one or all (pool capped at 4; toolchain pinned via `.odin-version`). Design: `docs/superpowers/specs/2026-06-07-claim-verification-harness-design.md`; origin: `…/2026-05-08-executable-verification-idea.md`.

### Verifying "it works"

A test only proves what it exercised — and "exercised" means the *exact* command the user runs, not a paraphrase of it.

- **Reproduce the reported command verbatim — before and after the fix.** Copy it character-for-character: same arguments, same path style (`.\foo`, `foo`, and `./foo` are *different inputs*), same directory. Reproduce the failure first, then re-run the identical command to confirm the fix. The cwd and the literal input are both part of the test — replicating one but not the other is not reproduction.
- **If you must use a throwaway fixture** (e.g. the real file is under a protected path like `scratch/`), keep the invocation identical — same path style, cwd, and flags — and change *only* which file it points at. Never "clean up" the user's input into a convenient form.
- **Test your own glue, not the proven dependency.** The risk is almost always in the few lines you wrote (cwd, quoting, path handling, a flag); "the underlying tool ran" is not "my code handles the user's input."
- **Verify the effect, not the announcement.** A tool printing `[Running: X]` or `Build started` only announces intent — it is not proof X ran. Check the actual side effect (a file written, output produced, an exit code), never a banner. (`just watch` printed `[Running: odin run …]` the whole time while watchexec's default shell silently swallowed the command and nothing executed.)
- **Scope the claim to what you ran.** One green run ≠ works — especially if you changed the inputs from what was reported.
- **Manually setting an env var or shell variable to make something run is an ERROR — full stop.** If you ever write `$env:FOO = …`, `export FOO=…`, `set FOO=…`, or `env FOO=… <cmd>` to *make a command work* (as opposed to the `verify-in-native-powershell` harness, whose whole job is reconstructing the real env), STOP — you are fabricating the environment instead of reproducing how it is really launched. The real launcher (Zed task, `.bat`, `justfile`, the user's profile) supplies those vars; if you're hand-setting them, you don't know the real invocation and your "it works" is a false green. Find the real entry point (read the `.zed/` task / `.bat` / recipe) and drive *that*, or tell the user the exact thing they run. Hand-set env vars to paper over a missing invocation = the same false-green trap, every time.
- **Bash is your working shell; native PowerShell is the end layer that must ALSO pass.** Do day-to-day work in the Bash tool — it's faster and cleaner for you, and PowerShell is *not* your primary driver. But everything the **user** runs lands in **native Windows PowerShell**, so any command, `just` recipe, `.bat` task, or setup/toolchain step you create or change must be **verified to work there too before you call it done.** A bash-only green is a *false green* (your Bash tool is Git Bash/MSYS2 — PATH, env, and which-shell-runs all differ; the reverse, a *false RED*, also happens). This is an end-of-task gate, not a per-command chore: build in bash, then clear the PowerShell gate for anything user-facing. Verify via the **`verify-in-native-powershell` skill** — its primary method is the **native PowerShell tool** (`CLAUDE_CODE_USE_POWERSHELL_TOOL=1`): run the real command verbatim after the skill's mandatory cleanliness check (`$env:MSYSTEM` must be empty). The `env -i` registry harness is only the fallback when that tool is absent. Never hand-set env vars to force a pass (see the bullet above).

## Initial scope (v1)

The substrate's primary intent is to support learning Odin + game programming + graphics programming. Odin and graphics are populated; papers/sdl3/engines are empty shells reserved for extending into those areas. Don't over-build for hypothetical future domains.

---
> Source: [ln167/odin_master](https://github.com/ln167/odin_master) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
