## mcp-mssql

> Rules AI agents must follow when working in this repository.

# Agent Instructions

Rules AI agents must follow when working in this repository.

---

## Commit messages

Use **Conventional Commits**.

### Header

* Format: `<type>(optional scope): summary`
* Use lowercase types (`feat`, `fix`, `ci`, `chore`, `docs`)
* Use scopes when relevant
* Write summaries in lowercase, imperative mood

### Body

* Leave a blank line after the header
* Explain **why**, not what
* Use imperative, present tense
* Wrap lines at ~72 characters

The body is optional for trivial changes.

---

## Release tags

* Use the bare version as the tag name — **no `v` prefix** (e.g. `1.0.0-beta.3`, not `v1.0.0-beta.3`)
* Tags must be annotated (`git tag -a`) with a structured release-notes message

---

## Commits (automation)

When generating commits via a shell:

* Do **not** pass generated messages directly to `git commit -m`
* Write the commit message to a file or standard input
* Use `git commit -F <file>` or `git commit -F -`
* Disable shell expansion when writing commit messages

This avoids issues with backticks, quotes, and other shell-expanded
characters in generated commit messages.

---

## Attribution

Every AI-assisted commit, tag, PR, comment, reply, or message an agent writes
for someone must carry an `Assisted-by` trailer:

```
Assisted-by: AGENT_NAME:MODEL_VERSION [TOOL1] [TOOL2]
```

| Field             | Description                                               |
|-------------------|-----------------------------------------------------------|
| `AGENT_NAME`      | AI tool or framework (e.g. `Claude`, `Cursor`, `Copilot`) |
| `MODEL_VERSION`   | Specific model (e.g. `claude-opus-4-6`)                   |
| `[TOOL1] [TOOL2]` | Optional specialized analysis tools; omit everyday tools  |

* Place it at the **end**, after a blank line: a git trailer in commits, the last line of the body everywhere else.
* Skip it only for text the user dictates verbatim.
* Use only `Assisted-by` — no `Co-Authored-By`, no `Made with …`, no hand-written `Sent using …`, no other footers.

Example:

```
Assisted-by: Claude:claude-opus-4-6 coccinelle sparse
```

---

## Code style

Follow existing project conventions.

* Match formatting, naming, and file structure already in use
* Do not reformat unrelated code
* Prefer small, focused changes
* Avoid introducing new patterns without clear benefit

### Language-specific rules

* If a formatter or linter exists, follow it
* Respect `.editorconfig` when present
* Do not disable lint rules without justification
* Prefer explicit, readable code over clever abstractions

### C#

* Use trailing commas in multi-line collections, object initializers, and enums

---

## MCP metadata (normative)

Normative, high-density metadata: enough for correct tool and parameter selection, minimal to reduce token cost.

- **The `DescriptionAttribute` statement MUST start with [MSSQL], followed by a Verb-Object fragment, e.g., `[MSSQL] Execute Read-only T-SQL`.**
- **Use tag-based lineage (Src: <Entity>) for parameters that refer to server or database entities** (profile → Src: profiles, catalog → Src: sys.databases, plan id → Src: analyze_query). Prefer a catalog view to a tool name.

---
> Source: [alyiox/mcp-mssql](https://github.com/alyiox/mcp-mssql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
