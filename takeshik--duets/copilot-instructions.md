## duets

> Guidelines for coding agents when working on the Duets repository.

# AGENTS.md

Guidelines for coding agents when working on the Duets repository.

> **Note:** `CLAUDE.md` is a symlink to this file (`AGENTS.md`). Edits via
> either path modify `AGENTS.md` on disk. When staging changes, always use
> `git add AGENTS.md` explicitly — `git add CLAUDE.md` will not work because
> Git tracks the symlink target, not the symlink itself.

> **IMPORTANT — Language policy (read before responding):**
> - **Assistant prose outside repository content** (chat responses, reviews,
    > explanations, summaries, plans, status updates, scratch notes, handoff notes,
    > and design discussion notes): write in **the same language the user used**.
    > Never default to English.
> - **Repository content** (source code, comments, commits, docs, ADRs):
    > always write in **English only**.

## Build & Run

```bash
# Build the entire solution
dotnet build

# Run the sample application
dotnet run --project src/Duets.Sandbox

# Run tests
dotnet test
```

The solution targets **.NET 10**. The SDK version may be pinned via `mise.toml`.

## Project Structure

- `Duets.slnx` — Solution file (XML-based slnx format)
- `Directory.Build.props` — Shared build properties (TFM, nullable, etc.) applied to all projects
- `src/`
  - `Duets/` — Core library (public API): session, declarations, transpiler interface
    - `Resources/language-service.js` — Embedded TypeScript language service script loaded server-side by `TypeScriptService` for completions
  - `Duets.Pad/` — DuetsPad browser debug pad package; depends on `Duets` and `HttpHarker` (see `src/Duets.Pad/README.md`)
    - `Resources/StaticFiles/` — Embedded web assets compiled as `EmbeddedResource` and served by `DuetsPadService` at runtime
  - `Duets.Jint/` — [Jint](https://github.com/sebastienros/jint) backend package: `JintScriptEngine`,
    `BabelTranspiler`, `TypeScriptService`, `ExtensionMethodRegistry`, `DuetsSessionConfigurationExtensions`
  - `HttpHarker/` — Standalone lightweight HTTP server library (may be extracted to its own repo)
  - `Duets.Sandbox/` — Multi-mode debugging CLI (batch, repl, serve, complete); not part of the public API
  - `shared/` — `internal` utility code shared across all projects via `<Compile Include>` (not a separate assembly);
    place cross-project internal helpers here
- `samples/` — Runnable file-based app examples, grouped per package (run with `dotnet run samples/<package>/<file>.cs`)
- `docs/`
  - `architecture.md` — Architecture overview (current snapshot)
  - `decisions/` — Architecture Decision Records (ADRs)
- `tests/`
  - `Duets.Tests/` — Unit tests (xUnit v3)
  - `Duets.Pad.Tests/` — Unit tests for `Duets.Pad`
  - `HttpHarker.Tests/` — Unit tests for `HttpHarker`
  - `shared/` — test-support sources shared across test projects via `<Compile Include>`

## Architecture & Design

- [docs/architecture.md](docs/architecture.md) — Current architecture snapshot. Read this before making structural
  changes or answering any design or feasibility question.
- [docs/decisions/index.md](docs/decisions/index.md) — ADR index: Title, Keywords, and Abstract for all ADRs. Read this
  to identify relevant decisions before reading full ADRs.
- [docs/decisions/](docs/decisions/) — Architecture Decision Records (ADRs). ADR-N is at `docs/decisions/<N>_*.md`.

When a session involves a design decision (new component, technology choice, API design trade-off, etc.), draft an ADR
in `docs/decisions/` at the end of the session. If the decision affects the overall architecture, update
`docs/architecture.md` to reflect the new state.

## Committing

Use the `/commit` skill to commit changes. It handles commit granularity, code style, pre-commit checks, and message
authoring.

## Language

There are two distinct contexts with different language rules:

**Repository content** — source code, comments, commit messages, documentation, ADRs, and any other checked-in text
files — **must be in English**.

**Assistant prose outside repository content** — chat responses, reviews, explanations, summaries, plans, status
updates, scratch notes, handoff notes, design discussion notes, and other non-committed working prose — **must be in the
same language the user used**. Do not default to English. These are conversational or working outputs, not repository
content, and the distinction must be respected even when the subject matter is code.

## Code Style

Use `scripts/format.cs` as the repository formatting entry point. It is okay to run this script manually, even though
Agent Stop hooks also run it automatically. Generated code must still follow the code style and rules defined in
`.editorconfig`.

Format/lint suggestions must be resolved, not merely observed. `scripts/format.cs` auto-applies only safe fixes; tools
such as biome additionally *report* unsafe fixes (e.g. `useOptionalChain`) without applying them — deliberately, because
they may change behavior and require judgment. The script is not changed to blanket-apply unsafe fixes; exercising that
judgment is the agent's job. Every reported suggestion must be triaged to a decision before a session ends: review each
one and either apply it (when it is behavior-preserving and correct) or reject it deliberately (suppress via config or an
inline ignore, with a reason). Never leave a reported suggestion dangling as a mere observation.

### Comments

These rules govern the *form* of comments, never their information. No explanatory content may be deleted in the
name of tidiness. "Remove" always means remove styling or redundancy only; any genuine explanation in the affected
text must be preserved — relocated into a documentation comment or kept as a plain in-body comment, whichever fits.

- **No decorative dividers.** Do not write banner or divider lines built from repeated glyphs (`----`, `====`,
  `****`, `####`, box-drawing `──`), in any language (C# `//`, JS/TS `//` and `/* */`, CSS `/* */`, HTML
  `<!-- -->`). Delete the glyphs. If a divider wrapped a label, keep the label as a single plain comment
  (`// URL helpers`, `/* Base */`, `<!-- Top bar -->`).
- **Section labels, sparingly.** A single short label comment grouping related members within a type or file is
  acceptable. When a type accumulates many such labels (roughly four or more), treat it as a signal to split the
  type rather than to label harder.
- **Document the public surface.** Public and protected C# types and members carry `///` XML doc comments stating
  intent and contract; this applies especially to packaged libraries (e.g. HttpHarker). In JS/TS, intent and
  contract prose on a function, class, or exported/global symbol uses a JSDoc block (`/** … */`) above the
  declaration, matching the existing convention in `ScriptEngineInit.d.ts`. Trivial C# overrides (`ToString`,
  `Equals`, `GetHashCode`), operators, and accessors may be left undocumented when the type-level doc already
  covers them.
- **Intent goes in doc comments; rationale stays inline.** A comment describing *what a member is for or
  guarantees* belongs in a `///`/JSDoc doc comment on that member. A comment explaining *why* a particular
  statement or block does what it does stays an in-body `//`. Do not promote local rationale to a doc comment, and
  do not demote contract prose to an inline comment. Private-field invariants and concurrency notes stay as `//`
  comments above the field (the codebase does not use `///` on private members).
- **No commented-out code.** Delete it; rely on version control.
- **Markers.** `TODO`, `HACK`, and `FIXME` are permitted when paired with a concrete, actionable note.
- **Preserve load-bearing prose.** Security invariants (e.g. a single-point-of-use contract for `innerHTML`),
  unit/why rationale, and similar explanations must survive any reformatting verbatim.

These rules apply equally to test code.

## Testing

```bash
dotnet test
```

Tests use [xUnit v3](https://xunit.net/) and live in `tests/Duets.Tests/`. xUnit v3 runs on Microsoft.Testing.Platform;
use `--filter-class`/`--filter-method` instead of `--filter`.

## End-to-end verification with Duets.Sandbox

`Duets.Sandbox` provides a JSONL batch mode for agent-friendly end-to-end verification of the full stack (transpilation,
completions, type registration, web server). Use this to validate changes without writing test code.

```bash
# Pipe JSONL operations to stdin; one JSON result per line is written to stdout.
echo '{"op":"eval","code":"1 + 2"}' | dotnet run --project src/Duets.Sandbox -- batch

# Multiple operations in one session (variables persist across eval calls):
printf '{"op":"eval","code":"const xs = [1,2,3]"}\n{"op":"eval","code":"xs.length"}\n' \
  | dotnet run --project src/Duets.Sandbox -- batch
```

Send `{"op":"help"}` to get the full list of supported operations and their fields:

```bash
echo '{"op":"help"}' | dotnet run --project src/Duets.Sandbox -- batch
```

Diagnostic output (initialization messages) goes to stderr; stdout contains only JSONL results, making it
straightforward to parse with standard tools.

---
> Source: [takeshik/Duets](https://github.com/takeshik/Duets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
