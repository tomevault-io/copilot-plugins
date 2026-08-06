## greenfield

> You are in **GreenField**, a prescriptive build manual with copy-ready templates for shipping a

# AGENTS.md — GreenField

You are in **GreenField**, a prescriptive build manual with copy-ready templates for shipping a
Next.js web app (which is also the only backend) plus a native Kotlin and Compose Android client,
on two Firebase projects — staging and production — with Web and Android sharing the project for
their environment, in a pnpm and Turborepo monorepo.

**This repository is not a product.** Nothing here is deployed. The deliverable is the manual in
`vault/` and the templates in `vault/90-Templates/`, which are copied into new product repositories.

> This file governs work on **the manual itself**. It is not the `AGENTS.md` that products get —
> that one is a template at `vault/90-Templates/files/root/AGENTS.md.template` and describes a
> product's rules, not these.

## Repository map

```text
greenfield/
├── AGENTS.md            ← you are here
├── README.md            ← the public entry point
├── SECURITY.md          ← disclosure policy; a template flaw affects every product built from it
├── vault/               ← THE MANUAL. An Obsidian vault, 104 notes
│   ├── Home.md            index
│   ├── 00-…12-           standards, decisions
│   ├── 90-Templates/      manifest.json plus the literal files
│   └── 91-Runbooks/       ordered procedures
├── skills/              ← agent skills for operating on GreenField itself
└── tools/check-vault.mjs  integrity checks, also run in CI
```

Only `vault/` is copied into a product. Everything else belongs to this repository.

---

## If the user wants to start a new product

This is the most common request. Do this, in order:

1. Read `vault/00-Start-Here/Bootstrap-A-New-Product.md` — the master ordered recipe, which cites
   the exact note for every step.
2. Read `vault/91-Runbooks/Runbook-Bootstrap-New-Product.md` — the command-level procedure,
   including the `product.json` answers file.
3. Follow them. Do not improvise a step a note already specifies, and do not reconstruct the
   procedure from this file.

### Where the product goes

The product is **never** created inside this repository. There are two layouts, and which one
applies depends on how you got here:

| You are working in | The product goes | Then |
| --- | --- | --- |
| A clone made *inside* the product's own directory, i.e. `my-product/greenfield/` | `my-product/`, the parent | **Delete this clone** before `git init`. It is scaffolding |
| A permanent clone, e.g. `~/src/greenfield` | A sibling directory | Leave this clone alone |

The first is the normal case and the one the runbook documents. The clone is a delivery mechanism,
like a downloaded template: the manual is copied to `my-product/docs/` and the clone is discarded
before the product's history begins, so the nesting never reaches a commit.

Nothing is lost by discarding it. `docs/` holds the whole manual, and the product's own `AGENTS.md`
and `.cursor/skills/` are materialized from templates.

### The five things that go wrong here

| Mistake | Why it matters |
| --- | --- |
| `git init` in the product before removing the clone | A repository inside a repository is a rejected pattern (ADR-0001) and forces a gitignore workaround. Order: `rm -rf greenfield`, then `git init` |
| Copying more than `vault/` | This repository's README, licence, CI, `skills/` and `tools/` are the manual's, not the product's. Its CI checks paths that will not exist there and will fail |
| Losing the provenance record | Write `.greenfield-origin` with this clone's commit **before** deleting it. A GreenField advisory cannot reach a copy; that file is what tells the product whether one applies |
| Choosing the Android application id casually | Once an artifact is uploaded to Play under an id, that id can never be changed or reused |
| Skipping `--dry-run` on the materialize script | It reports unresolved placeholders before writing about fifty files |

### Human-only steps

Billing account linkage, Firebase terms acceptance, Play Console and payment provider account
creation, and the first manual bundle upload cannot be scripted. The runbook marks each one. Stop
and hand over rather than pretending they succeeded.

---

## If the user wants to change the manual

### The rules

- **A note and its template must agree.** A standard saying one thing while the template ships
  another is the worst defect this repository can have: the builder follows the template and
  believes the note. Fix both in one commit.
- **A note may describe a capability, never a product.** "A structured-output endpoint" is correct;
  naming a real product is a leak. No product names, domains, project ids or application ids.
- **Reversing a locked decision means superseding its ADR**, never editing an accepted one in
  place. Then update `vault/00-Start-Here/Decision-Register.md`.
- **Add every new note to the link graph.** An unlinked note is invisible, and the checks fail on
  orphans.
- **Versions live in exactly two notes**, the web and Android stack baselines, plus the templates
  that literally contain them. Do not repeat a version anywhere else.
- Bump `updated:` in the frontmatter of every note you touch.

### Adding a template

1. Put the file under `vault/90-Templates/files/**` with a `.template` suffix.
2. Add an entry to `vault/90-Templates/manifest.json` with its destination, and a `when` condition
   if it is module-specific.
3. Use only the placeholders in `vault/00-Start-Here/Placeholder-Conventions.md`. An angle-bracket
   token that is not a placeholder belongs in the `NOT_PLACEHOLDERS` set in the materialize script,
   with a reason.
4. Document it in `vault/90-Templates/Templates-Index.md`.

### Verify

```bash
node tools/check-vault.mjs
```

Link integrity, orphans, frontmatter, code-fence balance, manifest agreement in both directions,
product-name leakage, and that the bootstrap script still parses. It runs in CI on every pull
request and it must pass before you commit.

---

## Security

A weak default in a template becomes a weakness in every product built from it, and those products
have no channel through which a fix reaches them. So:

- Templates are more conservative than product code would be. When in doubt, ship the stricter
  default and let the builder loosen it deliberately.
- `vault/01-Foundations/Threat-Model.md` states what the architecture defends against **and what it
  does not**. If you add a control, move the row. If you decline to add one, put it in the second
  table with the consequence — an omission that is written down is a decision; an omission that is
  silent is a trap.
- Never claim a control that does not ship. "An ESLint rule blocks this" is false until the rule is
  in the template.

## Quality bar

- Small changes. A note and its template, together.
- Prose in English, in the imperative. Standards, not option lists.
- No emojis in notes, templates or commits.
- Conventional Commits: `docs(vault): …`, `feat(templates): …`, `fix(templates): …`.

## When you do not know what to do next

Read `vault/Home.md`, then `vault/00-Start-Here/How-To-Use-This-Vault.md`. Do not invent direction
for the manual silently — a wrong standard propagates into every product that copies it.

---
> Source: [diegoalvarezmgl/greenfield](https://github.com/diegoalvarezmgl/greenfield) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
