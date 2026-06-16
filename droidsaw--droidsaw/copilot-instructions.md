## droidsaw

> The operator + AI-driver guide for **droidsaw**: how to drive it and read what it tells you. It's

# Using droidsaw

The operator + AI-driver guide for **droidsaw**: how to drive it and read what it tells you. It's
written for the AI assistant driving the tool and for anyone who wants the lean operational reference.

The user-facing guide — install, [per-audience playbooks](README.md#playbooks),
[MCP wiring](README.md#mcp-server), the architecture, and the full [correctness story](README.md#correctness)
— lives in [`README.md`](README.md). This file is the operational layer on top of it.

> droidsaw is **static-only**. It reads an app as bytes, parses the container, decompiles the
> bytecode, and runs detectors entirely in memory. It never executes the sample and never makes a
> network request. That property is what makes it safe to point at hostile input.

---

## For AI assistants

If you are driving droidsaw:

- **Prefer the MCP tools** when the `droidsaw` server is active — it's active when `mcp__droidsaw__*`
  tools appear in your tool list. `load` once, then operate on the session. Otherwise use the CLI
  (`droidsaw <cmd> … | jq`).
- **Call `load` before any other MCP tool** — every other tool errors until the session is loaded
  (see the MCP section below).
- **Default to `audit --mode=basic`, explicitly** — fast, hermetic, no external tools. The binary
  default is `full`, which silently skips Semgrep/TruffleHog when they aren't installed; only reach
  for `full` when the user wants those and the binaries are present. (Via MCP the `audit` tool is
  classed `spawns-subprocess` for its worst-case mode, so even `basic` needs that class granted — that
  is a per-tool capability label, not a claim that basic spawns anything.)
- **`taint` and `query` are not CLI commands.** Taint comes from `audit`; query an exported DB with
  `sqlite3` (or the MCP `query` tool, which is `SELECT`-only).
- **stdout is always JSON.** Parse it; don't scrape human text. Errors are JSON envelopes with a
  non-zero exit code.
- **Resource caps exist** (`--budget-mem`, `--budget-time`) — use them on untrusted or large input in
  a constrained context.
- **Report honestly.** *Unrecognized* output and flagged-but-untraced flows are real signal, not
  failure — surface them as-is rather than papering over them.

---

## Build & run

```sh
cargo install droidsaw                 # the CLI
cargo install droidsaw --features mcp  # also installs the droidsaw-mcp server
cargo auditable install --locked droidsaw  # fleet/CI: binary embeds its dependency inventory
                                           # (verify later: cargo audit bin $(which droidsaw))
```

`cargo` pulls the `droidsaw-*` library crates from crates.io and compiles locally. Prereqs: a Rust
toolchain ([rustup](https://rustup.rs)) and a C compiler (`cc`/`clang`). `semgrep` and `trufflehog`
are optional (used by `audit --mode=full`, `--mode=semgrep`, `--mode=trufflehog`); YARA is built in. No
Java or Android SDK is needed. Install + build-from-source detail is in the [README](README.md#install).

**Building from a source checkout** (not needed to *use* droidsaw — `cargo install` above is the
normal path; PRs aren't currently solicited). The workspace wires its sibling library crates via a
`[patch.crates-io]` block that points at `../droidsaw-common`, `../droidsaw-cli-contract`,
`../droidsaw-dex`, `../droidsaw-hermes`, `../droidsaw-apk`. A bare clone of just this repo therefore
**fails** to build with a missing path-dependency error. Clone all six repos as siblings under one
parent directory, then build `droidsaw/`:

```sh
for r in droidsaw droidsaw-common droidsaw-cli-contract droidsaw-dex droidsaw-hermes droidsaw-apk; do
  git clone https://github.com/droidsaw/$r
done
cd droidsaw && cargo build   # the [patch.crates-io] block resolves the five siblings from disk
```

If you (an AI assistant) are asked to build a checkout and hit `error: failed to load source for
dependency ... ../droidsaw-<crate>`, the cause is missing sibling repos — clone them as above; do
not switch the user to `cargo install` (that installs the published crate, ignoring their checkout).

---

## I/O contract

- **Input** is an `.apk`, `.xapk`, `.hbc` (Hermes bundle), or `.dex`. DEX and Hermes layers are
  extracted from an APK automatically. Co-located `split_config.*.apk` siblings are auto-merged
  (`--no-auto-splits` to disable).
- **Output** is JSON on **stdout** — one object, one array, or an NDJSON stream. Nothing else.
  The documented plain-text exemptions: `--version` and `--help` (clap convention),
  `hbc disassemble`, `scan trufflehog`, and `decompile --all --js`.
- **Progress** goes to **stderr**, prefixed `droidsaw: `.
- **Exit code** is `0` on success, `2` on failure, and `1` is reserved for the opt-in
  `audit --fail-on=<critical|high|medium|low|info>` gate: the audit completed, stdout carries the
  normal audit output, and at least one emitted finding is at or above the threshold — gate a CI
  pipeline on the exit code alone. Every failure (including argv errors) is a typed JSON error
  envelope on stdout: `{ "error": { "code": "USER_INPUT|PERMISSION|TRANSIENT|CONFIGURATION|INTERNAL", ... } }`.
- **Output is deterministic** — same input, byte-identical output across runs.

Because stdout is always JSON, every command composes with [`jq`](https://jqlang.github.io/jq/):

```sh
droidsaw manifest app.apk | jq '.manifest.exported_components'
```

---

## Command surface

Every command takes a path and writes JSON. The verified top-level surface:

| Command | What it does |
|---|---|
| `info <path>` | One-shot layer summary: bytecode layers + manifest + signing. The first thing to run. |
| `audit <path>` | Full security audit: findings + cross-layer taint + (optionally) Semgrep/TruffleHog. See [§ audit](#audit-the-workhorse). |
| `decompile <path> [target]` | DEX class → Java, Hermes function → JS. **Target shape picks the layer:** a dotted class name (`com.example.Foo`) → DEX; a bare integer (`42`) → Hermes function index. `--all`, `--js`, `--search <re>`, `--out <dir>`. |
| `strings <path>` | Search strings across all layers. `--layer dex\|hbc\|native`, `--search <re>`. |
| `xrefs <path> --search <re>` | Per matched string, the functions that reference it (DEX + HBC) → `{xrefs:[{layer, string, functions:["name(#id)"]}]}`; `--limit <n>` caps results. "Who touches this key?" without grep — then `decompile` a referencing function to see how it's used. |
| `manifest <path>` | AndroidManifest.xml: permissions, exported components (with `exported_reason`), intent filters, findings. |
| `signing <path>` | v1/v2/v3/v4 signing blocks, certificate details, and crypto-attack results (ROCA, Fermat, Wiener, batch-GCD). |
| `frida <path> --search <re>` | Generate Frida hook stubs for functions that touch matching strings. |
| `diff <old> <new>` | Structural diff of two Hermes bundles. Takes APK or `.hbc` (HBC is extracted from an APK automatically). |
| `deobf-strings <path>` | Emulate a DEX method over argument sets to recover obfuscated strings. `--class`, `--method`, `--int-range`/`--args-json`. |

**Umbrella commands** group nested subcommands:

| Umbrella | Subcommands |
|---|---|
| `hbc <sub>` | `info` · `functions` · `decompile` · `strings` · `disassemble` |
| `dex <sub>` | `classes` · `methods` (`--implementations`) · `strings` |
| `inspect <sub>` | `entries` (ZIP + anomalies) · `elf` (native lib metadata) · `resources` (resources.arsc) · `webview` (assets/www/) |
| `scan <sub>` | `yara` · `sbom` · `trufflehog` · `semgrep` · `export` (→ SQLite) |
| `corpus <sub>` | `ingest` (→ SQLite corpus DB) · `scan` (batch NDJSON) |
| `triage <sub>` | `promote` (crash-bundle → adversarial fixture) |

Run `droidsaw <cmd> --help` or `droidsaw <umbrella> <sub> --help` for exact flags.

### Two commands you might expect but that are **not** on the CLI

- **`taint`** is *not* a standalone command. Cross-layer taint runs as part of `audit`; results
  appear in the audit JSON (finding IDs `HBC_TAINT_FLOW`, `DEX_TAINT_FLOW`, `BRIDGE_TAINT_FLOW`) and
  in the `taint_flows` SQLite table. (A `taint` *tool* exists on the MCP surface.)
- **`query`** is *not* a CLI command. Query an exported SQLite database with `sqlite3` directly, or
  use the MCP `query` tool. (See [§ Getting findings into SQLite](#getting-findings-into-sqlite).)

### Global flags (resource caps for CI)

- `--budget-mem <bytes>` — memory ceiling per parse (default 4 GiB).
- `--budget-time <secs>` — wall-clock ceiling per parse (default: none).
- `--single-thread` — deterministic serial execution (~2× wall; same findings).
- `--no-auto-splits` — analyze a single APK without merging split siblings.
- `--permissive-recovery <opt,…>` — tolerate malformed AXML (`multi_root`, `unclosed_elements`,
  `orphan_end_element`, `all`); each recovery emits an `AXML_*` finding. Over-reports, never
  under-reports runtime-visible content.

---

## `audit`: the workhorse

`audit` parses every layer, runs the detector suite, and traces cross-layer taint in one pass.

### Modes (`--mode`)

| Mode | What runs | When to use |
|---|---|---|
| `basic` | Parser-side findings + built-in YARA + cross-layer taint. **No subprocesses.** ~10–30 s on most APKs. | **CI and agent loops.** Fast, hermetic, no external tools. |
| `full` *(binary default)* | `basic` + Semgrep + TruffleHog. | Deep one-off review (needs `semgrep` + `trufflehog` on `PATH`). |
| `semgrep` | `basic` + Semgrep only. | Code-pattern review with your own rules. |
| `trufflehog` | `basic` + TruffleHog only. | Credential sweep. |

> **For agent loops and CI, pass `--mode=basic` explicitly.** The binary defaults to `full`, which
> silently skips Semgrep/TruffleHog when those binaries aren't on `PATH` (the `detectors` field shows
> the skip) — easy to misread as a clean scan.
>
> **droidsaw ships no Semgrep rules.** For `semgrep`/`full`, supply your own with `--rules <path>`
> (repeatable) or the `DROIDSAW_SEMGREP_RULES` env var.

### Output format (`--format`)

- *(default)* — JSON audit object on stdout.
- `--format sarif` — SARIF 2.1.0, ready for GitHub code scanning / any SARIF viewer.
- `--format unsigned-evidence --output <dir>` — a canonical evidence bundle:
  `envelope.json` + `findings.ndjson` + a `findings.db` SQLite file. Good for case files.

**STIX IOC matching** — also pass `--stix-feed <path>` (repeatable; local file, no network) to match
a STIX 2.1 bundle against the parsed APK content.

### Cross-layer taint, briefly

`audit` runs three taint passes (full mechanics in [README § Cross-layer taint](README.md#cross-layer-taint)):
an **HBC pass** (Hermes `DirectEval` / tainted NativeModule args), a **DEX pass** (interprocedural
depth 4 via a cross-DEX class-hierarchy index), and a **bridge pass** that stitches a JS-side source
through a React Native `@ReactMethod` into a Java-side sink as one finding. The library defines 15
source kinds × 15 sink kinds; taint is traced across the DEX and Hermes layers (Dart/Flutter and
IL2CPP are not traced).

> **JNI boundary.** A tainted value crossing into a JNI-native method is flagged
> (`JNI_TAINTED_NATIVE_CALL`) but not traced into native code — surface it as a boundary finding, not
> a gap in the analysis.

### Getting findings into SQLite

The CLI's primary output is JSON on stdout — pipe it to `jq`. When you want a relational store:

```sh
# Evidence bundle (includes a canonical findings.db):
droidsaw audit app.apk --mode=basic --format unsigned-evidence --output ./case/
sqlite3 ./case/findings.db "SELECT severity, id_tag, detail FROM findings ORDER BY severity"

# Or export parsed layers (strings/functions/classes/edges + strings_fts) for ad-hoc query:
droidsaw scan export app.apk --output app.db
sqlite3 app.db "SELECT * FROM strings_fts WHERE strings_fts MATCH 'http'"
```

The two databases carry different schemas. The **findings DB** — from `audit --format
unsigned-evidence` (or the session DB an MCP `audit` writes) — is at schema revision **6**; tables/
views include `findings` (+ `findings_fts` full-text), `taint_flows`, `credentials`,
`cross_layer_taint_critical`, `actionable_findings`, `semgrep_hits`. The **`scan export` DB** holds
the parsed layers: `strings`, `functions`, `classes`, `edges`, `strings_fts`.

---

## MCP server (for AI agents)

droidsaw exposes its analysis surface over MCP so an agent can load an APK, audit it, query findings,
decompile a class, and diff bundles in one session. The server is the standalone `droidsaw-mcp`
binary; **transport is stdio only** — an MCP client spawns it as a child process (no network
listener).

Two dials plus a sandbox set the posture: **`DROIDSAW_MCP_ROOT`** confines every path the server reads
or writes; **`--allowed-tool-classes`** (default `read-only,writes-tempfile`) gates what it may do;
and **`--tool-tier`** (`basic` = 12-tool core workflow, `full` *(default)* = every tool) gates what
the model sees.
The class gate is per-tool — note `audit` is classed `spawns-subprocess` even in `basic` mode, and
`triage` needs `manages-state`. There is no built-in authentication (`DROIDSAW_MCP_ROOT` is the only
confinement boundary), and large tool outputs stream to a tempfile rather than the context window.

Session shape: **`load` first** (every other tool errors until then) → `audit` → `query` (read-only
SELECT) → `investigate(rowid=…)` (enriches one finding) → `decompile` → `triage`.

**The full `.mcp.json` wiring and security model are in the [README](README.md#mcp-server).**

---

## Interpreting findings

- **Severity** ranks `Critical > High > Medium > Low > Info`. Each finding carries a stable id, a
  human-readable `detail`, the `layer` it came from, and a CWE where one applies.
- **`detectors`** in the audit JSON lists which passes ran vs were skipped (binary-not-on-PATH,
  mode-gated, or subprocess error) — so a quiet result from a skipped pass is never mistaken for a
  clean result.
- **Taint flows** appear both as findings (`*_TAINT_FLOW` ids) and in the `taint_flows` table with
  `source_type` / `sink_type` / `severity` / `cwe`.
- **Honest gaps are labeled, not hidden.** Where the decompiler cannot recognize a region it emits an
  explicit *Unrecognized* marker rather than guessing; where a tainted value crosses into native code
  the flow is flagged but not followed. Trust the labels.

---

## Why the output is trustworthy

droidsaw's findings build on a parse checked against the original bytes, not a best-effort heuristic
decode: parse → re-emit → compare byte-for-byte, so a misunderstood byte produces a divergence the
test localizes precisely. The evidence — **5,767 DEX files** from F-Droid recovered bit-identically
(preservation mode) and Hermes round-trip on v84/v96/v98/v99 (parser covers v40–v100) — plus fuzz, cross-tool differential vs
`dexdump`/`hbcdump`, and Kani + Lean proofs, are the five correctness gates detailed in
[README § Correctness](README.md#correctness).

Underpinning all of it: **no panic on adversarial input.** Every parser/CFG/SSA/region/emit path is
compiled under a lint that rejects `unwrap`/`expect`/`panic`/unchecked arithmetic in non-test code,
so malformed input yields a typed error, never a crash. That is why it is safe to point droidsaw at a
malware sample or a fuzzed file.

---

## What droidsaw deliberately does not do

- **No execution, no network.** Static analysis only; STIX feeds load from a local file path.
- **No native disassembly.** ELF analysis reports hardening flags, JNI exports, and relocation
  counts — it does not lift native code.
- **Dart (Flutter) and IL2CPP (Unity) are not currently supported.** An app whose logic is only in
  those layers shows no decompile/taint output for them.
- **Obfuscation resilience is bounded** by the fixture matrix. Heavily obfuscated input may surface
  as *Unrecognized* rather than a plausible-but-wrong guess — by design.
- **No whole-program reflection / self-modifying-code modeling.**

---

## Reporting issues

Found a bug? File a [GitHub issue](https://github.com/droidsaw/droidsaw/issues) with a reproduction
(search first). A crash → the triggering input, or a `droidsaw triage promote <bundle>` fixture; a
decompilation gap or a wrong/missing finding → the input plus got-vs-expected. Full guidance is in
[README § Reporting issues](README.md#reporting-issues). Code PRs aren't currently solicited.

---

*License: BSD-3-Clause. Art by [pmjv_prahou](https://analognowhere.com/about).*

---
> Source: [droidsaw/droidsaw](https://github.com/droidsaw/droidsaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
