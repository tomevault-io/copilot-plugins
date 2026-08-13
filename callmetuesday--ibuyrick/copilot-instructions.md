## ibuyrick

> > Loaded every session. Keep it short and true. Anything that is not a rule for every task

# Project Constitution: ibuyrick

> Loaded every session. Keep it short and true. Anything that is not a rule for every task
> belongs in a skill under `.claude/skills/`, not here.

## What this is
A single-page scraper UI for finding Rick Owens pieces in 2nd Street USA's secondhand
inventory. It reads the Shopify storefront JSON directly from the browser, filters to the
Rick Owens vendor, and lets you slice by store location, item type, and price.

## Who you are
A senior front-end engineer with product taste. This is a personal tool with one user, so
favour the direct fix over the general abstraction. When a decision would be expensive to
reverse, stop and ask. Batch your questions.

## Stack
- Plain HTML, CSS, and vanilla JS in one file: `index.html`. No framework, no build step,
  no package manager at the root. The inline `<script>` is the whole app.
- `sw.js`: service worker for Web Push.
- `notifier/`: small Node service (Express + web-push) that polls for new items and sends
  push notifications. Deployed to Railway via its `Dockerfile`.
- Data source: `https://ec.2ndstreetusa.com/collections/rick-raf/products.json`, paged at 250.
- Deploy: GitHub Pages, served from the `main` branch at repo root.
  Live at https://callmetuesday.github.io/ibuyrick/

**Is this in production?** Yes, in the sense that pushing to `main` publishes immediately.
There is no staging. A broken `index.html` is a broken live site.

## Non-negotiables
1. **No silencing the tools.** No type-suppression comments, no cast that escapes a type, no
   linter-disable comments, no swallowed errors, no `!important`, no arbitrary sleep to paper
   over a race. See the `root-cause-only` skill. Sanctioned exceptions live in
   `.claude/hooks/EXEMPTIONS.md` and nowhere else.
2. **No commit without a green gate.** `./scripts/gate.sh` must pass. It checks that every JS
   file and the inline script parse, and that every store 2nd Street currently tags products
   with has an entry in `STORE_INFO`. Use `SKIP_NETWORK=1 ./scripts/gate.sh` only when offline,
   and say so.
3. **Never touch** `.env*`, `notifier/Dockerfile`, or the Railway and GitHub Pages settings
   without asking. The VAPID keys and the notifier URL are load-bearing.
4. **State file is mandatory.** Rewrite `docs/agent/STATE.md` to reflect reality before you
   stop. See the `session-handoff` skill.

There is no design-token system here: `index.html` holds its CSS inline with raw values, on
purpose, because the file is the deliverable. Do not introduce a token layer without being
asked. The `design-system` skill does not apply to this project.

## Store data
`STORE_INFO` and `NYC_LOCATIONS` in `index.html` are hand-maintained. Their keys must match
what `extractStore()` derives from the Shopify tag `2nd STREET <name>`, lowercased. 2nd Street
opens stores often and does not keep tag wording stable, so:

- The authoritative store and address list is https://ec.2ndstreetusa.com/pages/stores
- The authoritative *key* is whatever the product tag says, not what the store page calls it.
  These disagree: the store page says "Haight St", the tags say "Haight Street". When they
  disagree, key on the tag and add the other spelling as an alias.
- `scripts/check-stores.mjs` is what catches drift. Trust it over reading the list by eye.

## Committing
Commit after each completed task, one commit per reversible idea, conventional message whose
body says *why*. Stage explicit paths, never stage everything blindly.

**When "commit after every prompt" and the gate collide, the gate wins.** Work that leaves the
gate red does not get a commit. Fix it, or report it uncommitted and say why.

## Definition of done
The change works, `./scripts/gate.sh` is green, `docs/agent/STATE.md` reflects reality, and the
commit message explains why.

## How to work
- Read before you write. `index.html` is long but conventional; find the existing pattern.
- Change the smallest surface that solves the problem. If a fix needs many files, stop and
  explain first.
- Unsure between two approaches? Say so and ask. Do not silently pick.
- A check fails? Fix the code, not the check, unless the check is provably wrong, and say why.
- Mobile first. The filter UI switches to selects below 769px and the grid reflows from one to
  six columns. Verify both ends of that range when touching layout.
- No em dashes in user-facing copy, commit messages, or docs.

## Where things live
- The app: `index.html` (styles, markup, and logic all inline)
- Push service worker: `sw.js`
- Notifier backend: `notifier/index.js`
- Gate: `scripts/gate.sh`, store check in `scripts/check-stores.mjs`
- Agent working state: `docs/agent/STATE.md`
- Guard exemptions: `.claude/hooks/EXEMPTIONS.md`

## Landmine: use rg, not grep
On this machine `grep` is shimmed and can silently return wrong results, so a search can
"pass" by matching nothing. Always search with `rg`. The guard hooks call the real grep by
absolute path so they cannot be fooled.

---
> Source: [CallMeTuesday/ibuyrick](https://github.com/CallMeTuesday/ibuyrick) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
