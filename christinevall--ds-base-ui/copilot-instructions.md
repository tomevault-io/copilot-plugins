## ds-base-ui

> A small, real design system used to test and teach **how designers and AI work

# Working in this repo

## What this project is

A small, real design system used to test and teach **how designers and AI work
together on a real codebase without drift**. Not a product — every decision
optimises for legible and demonstrable over scale.

**If the task is writing, explaining, planning or discussing this project
rather than changing code, read `docs/decisions.md` first.** It carries what
the project is for, the thesis it tests, and every decision with its reason.
Keep it current when a decision changes. Notes, findings and articles about the
project are kept outside this repo.

## If someone is running this for the first time

Many people opening this repo are designers, not developers. Before running any
npm command, check the runtime and say what is wrong in plain language:

- `node -v` must print v20.19+ or v22.12+ (see `engines` in `package.json`).
  If the command is not found, the fix is: download the LTS installer from
  <https://nodejs.org>, run it, reopen the terminal. Say that — do not paste a
  stack trace.
- If `node_modules/` is missing, run `npm install` yourself rather than telling
  them to. It takes a few minutes on a first run; say so before starting.
- "Show me Storybook" means: install if needed, `npm run storybook` (port 6001),
  then open it in the Browser pane. Do not ask them to open a URL by hand.

## STOP — ground yourself first

**Before writing or changing any UI in this repo, check what actually exists. This session. Not from memory, not from an earlier session.**

Three sources of truth, in this order:

| Question | Where the answer is |
| --- | --- |
| What components exist, with what props, defaults and stories? | `storybook-static/manifests/components.json` — run `npm run build-storybook` if absent |
| What tokens exist? | `tokens/tier-1-definitions/` and `tokens/tier-2-usage/` |
| How is a component actually used? | its `*.stories.tsx`, and `src/patterns/` for full screens |

The manifest is generated from source, so it cannot be out of date. It is the answer to "does this component have a `variant` prop" — never guess, and never infer a prop from another component that looks similar.

If you cannot read those files, say so and stop. Do not fall back to writing from memory. A wrong answer that looks confident is the failure mode this file exists to prevent.

**One caveat the manifest will not tell you.** It documents *our* additions. Most components are thin wrappers around Base UI, and Base UI's own props come from `node_modules/@base-ui/react/<part>/index.d.ts`. If a component's manifest entry lists few or no props, that is not a component without an API — read the Base UI types too.

## The rules

1. **Semantic tokens only.** Components may use `--sds-color-background-*`, `--sds-color-content-*`, `--sds-color-border-*`, `--sds-space-*`, `--sds-radius-*`, `--sds-typography-*`, `--sds-elevation-*`, `--sds-duration-*`, `--sds-easing-*`. Never a primitive (`--sds-color-brand-indigo-600`, `--sds-shadow-*`), never a raw hex, never a magic pixel where a token exists.
   **Type comes from a text style.** Set all six `--sds-typography-<style>-*` properties (family, size, weight, line-height, letter-spacing, text-transform) from one style in `tokens/tier-2-usage/text-style.json` — a style is applied whole, never in part. Never set `--sds-font-size-*`, `--sds-line-height-*` or `--sds-letter-spacing-*` by hand; `--sds-font-weight-*` alone is allowed only to emphasise inherited text. If no style fits, that is a design decision — raise it, do not invent a combination.
2. **Wrap, do not rebuild.** If Base UI ships a primitive, wrap it. Never reimplement focus management, keyboard handling or ARIA.
3. **Compose downward.** Reach for a pattern in `src/patterns/` first, then a component, then a primitive. Building a card out of divs when `Card` exists is the most common failure here.
4. **Never edit generated files.** `src/tokens/primitives.css`, `src/tokens/semantic.css` and `src/tokens/breakpoints.ts` are build output. Edit `tokens/**/*.json` and run `npm run build:tokens`. Each generated file says so in its header.
5. **Do not add dependencies.** No component library, no icon package, no CSS framework. Icons are inline SVG.

## Two tiers of component

The distinction matters when deciding what to change:

- **Base UI wrappers** — most of the 42. Behaviour belongs to Base UI; we own styling and the token contract.
- **Composed** — `Card`, `Table`, `Badge`, `Alert`, `Spinner`, `Breadcrumb`, `IconButton`. No Base UI primitive to wrap because they carry no interaction logic. We own these outright.

## After you change UI

Check the rendered result, not just the code:

```bash
npm run build:tokens     # if you touched tokens/
npx tsc -b --noEmit      # must be zero errors
npm run lint
npm run validate         # checks this file's rules; warn-only
npm run build-storybook  # also regenerates the manifest
```

`npm run validate` is the net under the rules above. It checks that every token
referenced exists, that no component reaches past the semantic layer, that no
raw colour is hard-coded, and that every component still resolves in the
manifest. It warns and exits 0 so it never blocks a prototype; `-- --strict`
makes it fail, which is what CI would use.

Then look at the story in a browser. Markup that compiles and renders nothing still counts as broken.

## Branching

`docs/branching.md` is authoritative. Short version: `feature/*` off `main`, merged back into `main` by pull request with CI green. There is no `develop`. `design` is the designer playground and is a source of decisions, not a source of merges — accepted prototypes get rebuilt on a `feature/*` branch.

## Figma

The Figma library mirrors the code; it is never the source. To add or change
anything in it — a component, a variable, a text style — use the
`figma-mirror` skill (`.claude/skills/figma-mirror/`). `figma/manifest.json`
records what the library contains and `npm run validate` checks it against the
tokens and the components, so a Figma name that drifts from its token or its
prop fails CI. What cannot be mirrored exactly, and why, is in `figma/GAPS.md`.

## Further reading

- `docs/decisions.md` — what this project is for, the thesis, and every decision with its reason
- `docs/architecture.md` — why the repo is shaped this way
- `docs/conventions.md` — how to add a component
- `docs/branching.md` — the branch model
- `figma/GAPS.md` — where Figma cannot match the code, and why

---
> Source: [christinevall/ds-base-ui](https://github.com/christinevall/ds-base-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
