## bdd-workflow

> These rules apply to agents working in THIS repository only. They supplement the

# Agent Rules for the bdd-workflow Repo

These rules apply to agents working in THIS repository only. They supplement the
bdd-workflow skill.

## Template-first editing rule

Every framework Markdown file exists in two places:

- **Template** (source of truth): `src/scaffold/templates/.opencode/...`
- **Live** (in use by this project): `.opencode/...`

**Always edit the template file. Never edit the live `.opencode/` file directly.**

After editing templates, run:

    npm run build && npx bdd-workflow update

The build step is required because `bdd-workflow update` reads compiled templates from
`dist/`, not from `src/scaffold/templates/` directly.

`bdd-workflow update` performs a three-way diff and safely overwrites unmodified live
files while preserving any local customisations.

## Markdown testing criteria

In general BDD tests which are "markdown file contains <specific word>" are  pretty
useless.  Of course there's then no way to verify agent performance - but that's ok.
We can add that in the future (though it'll be expensive).  For now it's enough to
"file exists and is not empty" (not necessarily for every file).  Markdown files are
constantly changing and we don't need to change words in tests for that it's silly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dougyfresh42) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
