## construct

> Tool-agnostic guidance for any agent (Claude, Codex, Gemini, Grok, …) editing

# AGENTS.md — Construct skill catalogue

Tool-agnostic guidance for any agent (Claude, Codex, Gemini, Grok, …) editing
skills in this repo. The full, version-controlled workflow lives in
[`CONTRIBUTING.md`](CONTRIBUTING.md); this file is the committed digest of the
rules that bite. (A maintainer-local `CLAUDE.md` overlay adds host-specific notes
— Home Manager fan-out, local config — and is gitignored.)

## Hard rules

- **`SKILL.md` frontmatter `description` — hard cap 1000 characters.** The loader's
  absolute limit is 1024 (`field 'description' in SKILL.md must be at most 1024
  characters`), but this repo caps the *rendered* description at **1000** — it
  MUST NOT exceed 1000 chars. Folded `description: >` blocks render by joining
  lines with spaces (blank lines → newlines, plus a trailing newline); that
  rendered length is what counts, not the raw line count. Re-check after any edit.
- **Rebuild BOTH bundles after any skill-dir edit**, in the same commit:
  `<name>.zip` (`zip -qr`, keeps dir entries) and `<name>.skill` (`zip -qrD`,
  drops them). A bundle that lags its `SKILL.md`/`references/` ships broken
  content to every consumer.
- **Stage explicitly by name — never `git add -A` / `git add .`.** Other root
  `.skill` files carry unrelated uncommitted changes that must not be swept in.
- **Commit in UTC, signed** (signing is global, no flag needed); assistant
  commits add a `Co-Authored-By: Claude …` trailer.
- **Keep the README §2 catalogue row in sync** when adding, removing, or
  re-scoping a skill.

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the bundling commands, the drift
sweep, and the push procedure. The maintainer-local `CLAUDE.md` adds the Home
Manager local fan-out (host-specific).

---
> Source: [Spacecraft-Software/Construct](https://github.com/Spacecraft-Software/Construct) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
