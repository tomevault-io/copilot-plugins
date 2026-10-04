## compass

> A public map of fields/problem-spaces across tech/AI, plus a lighter "entities" layer (companies, academic labs, OSS projects, research orgs) cross-linked into it. Static site (Quartz), deployed to GitHub Pages on every push to `main`. Live: https://hnxnq7.github.io/compass/. No manual deploy step — just push.

# Compass

A public map of fields/problem-spaces across tech/AI, plus a lighter "entities" layer (companies, academic labs, OSS projects, research orgs) cross-linked into it. Static site (Quartz), deployed to GitHub Pages on every push to `main`. Live: https://hnxnq7.github.io/compass/. No manual deploy step — just push.

Read `content/index.md` first — it's the actual homepage and documents the content model, category legend, and update workflow in more depth than this file does. `content/Field Backlog.md` has known taxonomy gaps flagged but not yet built. `content/Live Feed Vision.md` has the (not-yet-built) plan for daily auto-updates.

## Content model

- **Fields** (`content/<Category>/*.md`) — problem spaces, not products. 12 root categories (tagged `field-root`, no `parent`), each with child field notes. Every note follows a fixed "Problem Space" template (see `_templates/Field Template.md`): what's genuinely hard, why unsolved, fake vs. inevitable solutions, `## Notable tools / instances` (concrete products/papers as **plain text, not graph nodes**), `## Key players` (entities, wikilinked), `## Watch list`, `## Connections` (static — see below).
- **Entities** (`content/Entities/{Companies,Academic Labs,Open-Source Projects,Research Orgs}/*.md`) — organizations/projects/labs with *sustained identity*. The bar: would this still be worth tracking in six months, independent of the specific thing that surfaced it? A landmark paper or a CLI tool stays plain text in a field's `Notable tools / instances`; the lab/company behind it gets an entity page. Template: `_templates/Entity Template.md`.
- Graph edges: `parent` = tree edge (fields only). `depends_on`/`enables`/`related` (fields) and `works_in` (entities) = the actual graph, cross-cutting the tree. **These are static, not live-queried** (no Dataview in Quartz) — a link added to frontmatter must also be mirrored in the body's `## Connections` section (fields) or the target field's `## Key players` (entities), or it won't show up in the rendered page/graph even though the frontmatter is "correct."
- Root notes' `## Children` lists and `content/index.md`'s nested tree are *also* hand-maintained — adding a new field note means updating both, not just creating the file.

## Gotchas that have already cost real debugging time

- **Filenames can't contain `/`.** A field whose display name needs a slash (e.g. "Real-World / Deployment Evals") gets a filename with `_` instead, linked via Obsidian alias syntax: `[[Real-World _ Deployment Evals|Real-World / Deployment Evals]]`.
- **Two or more literal `$amounts` on one line trigger the LaTeX plugin.** `"$200M round at a $1.6B valuation"` gets auto-detected as inline math by KaTeX and breaks. Escape as `\$200M`. This has bitten multiple population-sweep passes (funding-amount prose is everywhere in entity bios) — check for it (`grep -rn '\$[0-9].*\$[0-9]' content/`) after any bulk content addition.
- **The graph plugin (`plugins/graph/`) is a local fork**, not the upstream `@quartz-community/graph` package — Quartz's stock graph needed real code changes (landmark labels, meta-page exclusion, entities toggle, radius capping) that no config option covers. Its compiled `dist/` **is committed** (not gitignored), because the lockfile-driven `npx quartz plugin install` step silently no-ops for a local source with no lockfile entry — CI will build from a stale/missing `dist/` otherwise, despite printing a misleading success log. After any edit to `plugins/graph/src/`, run `npm run build` inside `plugins/graph/` and commit the resulting `dist/` changes.
- **Verify plugin/config changes with a true clean-room build before considering them shipped** — a local dev-server success has been misleading twice (once from a stale npm-registry package silently shadowing the local fork, once from the lockfile issue above). The real check: `rm -rf node_modules .quartz public plugins/graph/{node_modules,dist} && npm ci && (cd plugins/graph && npm install && npm run build) && npx quartz build`, and ideally diff the built JS against what's actually live (`curl` the deployed bundle) rather than trusting CI's green checkmark alone.
- Filenames/paths in this repo contain non-ASCII characters (emoji-free here, but `&` and parens are common in entity names) — always read a file before editing it, don't assume a glob match is the file you think it is.

## Skills

Two Claude Code skills, committed into this repo at `.claude/skills/` (project-scoped — picked up automatically by any Claude Code session working in this repo, on any machine, no manual setup):
- **`update-compass`** — lightweight, reactive. Fires on one new fact (a launch, a paper, something mentioned in conversation or turned up by a search). One or two small edits, then commits and pushes itself.
- **`populate-compass`** — deliberate, research-heavy. For auditing/enriching a note or a whole batch: WebSearch per note, decide entity-vs-plain-text, create entity pages when warranted, flag (don't auto-create) new field/subfield gaps into `Field Backlog.md`.

(A copy of both also lives at `~/.claude/skills/` on the machine they were authored on, as user-level skills available across any project there — that copy predates and is independent of the project-scoped one here; if you edit one, consider whether the other should match.)

## Bundled tooling

`~/.claude/skills/update-compass/scripts/check_links.py` — verifies every `[[wikilink]]` in `content/` resolves to a real filename. Run before every commit (`python3 ~/.claude/skills/update-compass/scripts/check_links.py content` from the repo root); Quartz resolves links by filename and fails silently/at-build-time otherwise.

---
> Source: [hnxnq7/compass](https://github.com/hnxnq7/compass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
