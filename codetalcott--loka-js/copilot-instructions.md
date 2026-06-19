## loka-js

> Work on loka-js (the hook-contract localization pattern for the fixiproject family — fixi, moxi, ssexi, paxi, rexi) or apply the same pattern to another minimalist library. Use when editing this repo's patched libraries, the orchestrator, locale vocab, demos, tutorials, or tests — or when a user asks how to add a new locale, add a new library, or contribute the patches upstream. Encodes the v1 coverage matrix, the bounded-surface principle that distinguishes loka from a hyperscript-style preprocessor, the load-order constraints, and the gotchas we hit in v1.


# loka-js: localizing the fixi-family

## Quick orientation

loka-js makes it possible for non-English-native authors to write fixiproject-family code in their own language without the libraries being aware of any language. Five libraries (fixi, moxi, ssexi, paxi, rexi); three are patched with tiny hook contracts; two need no patch and are localized from outside.

This is **not** a general i18n system — it localizes *authoring vocabulary* (attribute names, modifiers, global function names, event names), not user-visible content. The audience is beginner web developers whose native language isn't English.

The project follows the fixiproject convention: flat root, no build step, single-file sources, no bundler. Don't introduce dist/, package.json scripts beyond test+gen, or compilation.

## Coverage matrix

| Library | Patched? | Hooks | Localization surface |
|---|---|---|---|
| fixi  | Yes  | `name`, `event`, `sel`, `ignoreSel` | attribute names + trigger event values |
| moxi  | Yes  | `name`, `event`, `modifier`, `ignoreSel`, `xpath` | `live`/`on-`/`mx-ignore` attrs + dotted modifiers + globals |
| paxi  | Yes  | `isSwap` | the `morph` swap value + the `window.morph` global |
| ssexi | **No** | — | event names re-fired by orchestrator listener |
| rexi  | **No** | — | global verb names aliased via `loka.alias` |

**The decision to NOT patch ssexi and rexi is principled, not lazy.** ssexi fires events on the DOM that bubble; the orchestrator can re-fire localized variants from outside. rexi is pure JS with no DOM attributes; localization is global aliasing. Patching them would add surface for no gain. Apply the same reasoning if you extend the pattern to a new library.

## The bounded-surface principle

The translation surface for each library is **finite and flat**: a known vocabulary of attribute names, modifier names, event names, or function names. **Never** translate inside an arbitrary expression language (JS bodies in `on-*` / `live`, the q-DSL keywords like `next/closest/in`, hyperscript). That path requires a parser and inherits all the brittleness of [hyperfixi's hyperscript localization](../hyperfixi/) — which is why we don't do it.

If a new library's translation surface requires parsing, the answer is no. Document the line you didn't cross and move on.

## Load order (matters and is fragile)

```html
<script src="./orchestrator.js"></script>   <!-- pre-installs hooks on every lib namespace -->
<script src="./locales/{code}.js"></script>  <!-- one or more; calls window.loka.register -->
<script src="./moxi.js"></script>            <!-- moxi before fixi (fixiproject convention) -->
<script src="./ssexi.js"></script>           <!-- any order before fixi -->
<script src="./paxi.js"></script>            <!-- any order before fixi -->
<script src="./rexi.js"></script>            <!-- any order; rexi is standalone -->
<script src="./fixi.js"></script>            <!-- LAST among fixi-family -->
```

- **orchestrator must precede every patched library.** If a patched lib loads first, its `??=` defaults take effect and the orchestrator's hooks are silently ignored. The orchestrator emits a console warning for the fixi case (`document.__fixi_mo` already set) but not for all libs.
- **moxi before fixi** is upstream's rule — moxi needs to register `fx:init`/`fx:process` listeners before fixi dispatches them on DOMContentLoaded.
- **rexi is standalone** — no DOM lifecycle coupling.

## Per-element language resolution

[orchestrator.js](orchestrator.js) `langOf(elt)` checks in order:
1. `data-loka-lang` on element or ancestor (explicit override)
2. `lang` on element or ancestor (HTML standard — `<html lang>`, `<section lang>`)
3. Falls back to `"en"`

This is the capability that distinguishes loka from preprocessor-style approaches generally: loka resolves at attribute-read time per element, so a single page can mix `<section lang="es">` and `<section lang="ja">` without rewriting markup.

## Vocab pipeline

Single source of truth: [scripts/fx-vocab.mjs](scripts/fx-vocab.mjs) exports `LOCALES`, a table keyed by locale code. Each entry has per-library sub-objects:

```js
es: {
  profile: 'spanish',           // basename of @lokascript/semantic profile (sibling-repo)
  name: 'Spanish',
  reviewed: true,                // fixi attrs reviewed by native speaker
  globalsOptIn: true,            // emit window.loka.alias calls (off by default)
  fixi:  { attrs, events },
  moxi:  { attrs, modifiers, globals, reviewed },
  ssexi: { events, reviewed },
  paxi:  { swaps, globals, reviewed },
  rexi:  { globals, reviewed },
  props,                          // for psatina-modular, not fixi
}
```

[scripts/gen-locales.mjs](scripts/gen-locales.mjs) reads this plus the `@lokascript/semantic` profile from `~/projects/hyperfixi` (sibling checkout) and emits two files per locale:

- [locales/{code}.js](locales/) — script-tag loaded, calls `window.loka.register(code, data)`
- [dom-vocab/{code}.js](dom-vocab/) — ES module of shared `{events, props}` for non-fixi consumers (psatina-modular, etc.)

**Locale files are auto-generated. Never hand-edit them.** To change vocab, edit `scripts/fx-vocab.mjs` and run `npm run gen` (or `node scripts/gen-locales.mjs --locale=es` for one locale).

## How to: add a new locale

1. Add an entry to `LOCALES` in [scripts/fx-vocab.mjs](scripts/fx-vocab.mjs) with at least `profile` (matching a file in `~/projects/hyperfixi/packages/semantic/src/generators/profiles/`), `name`, `reviewed: false`, and a `fixi.attrs` map.
2. Run `node scripts/gen-locales.mjs --locale=<code>` and inspect the output.
3. If `globalsOptIn` should be true (Spanish-style locale that wants `obtener=get` aliasing), add per-library `globals` blocks too.
4. Run the test suite — Phase A's hardcoded locale loop (`['en','es','ja','ar']`) won't pick up new locales, but the locale generation must succeed without errors.

## How to: add a new library to the pattern

**Fork first: is this a fixi-family library or a library with its own native registry?**

- **Fixi-family library** (no native registry; needs hooks via `??=` slots): follow the recipe below. The orchestrator installs hooks on `window.<libname>` and collects per-locale vocab from `data.<libname>` blocks. Current fixi-family members: fixi, moxi, paxi, ssexi, rexi.
- **Library with its own native registry** (exposes a Map / `addX` method / `register` function / etc.): **don't extend the orchestrator.** Write a small locale module that fills the native registry directly and imports `dom-vocab/<lang>.js` for shared DOM-keyword vocab. See [experiments/psatina-study/](experiments/psatina-study/) as the worked example — psatina exports its `templateDirectives` Map; psatina-study's 40-line [`lib/locales/es.js`](experiments/psatina-study/lib/locales/es.js) does `templateDirectives.set('si', ...)` etc., importing `{events, props}` from this repo's `dom-vocab/`. The orchestrator never knows about psatina.

The rest of this section is the **fixi-family recipe**.

Before patching anything, decide: does the library require a patch, or can it be localized from outside (like ssexi/rexi)?

- **No patch needed** if: the library fires DOM events that bubble (listener re-fire works) OR exposes only JS APIs with no DOM attributes (global aliasing works).
- **Patch needed** if: the library reads attributes or attribute values directly (then it needs hooks to redirect through localized vocabulary).

If patching:

1. Identify the translation surface — attribute name literals, magic string values, modifier names. List them.
2. Design the minimal hook contract — usually 1–5 hooks, defaulted via `??=` to preserve upstream behavior.
3. Write the patched library file at repo root, matching upstream code style (single-IIFE, short var names, no extra whitespace).
4. Build the patch artifact at `reference-patches/<lib>.patch` in `git format-patch` shape — this is documentation of the fork's diff, not a PR submission. Placeholder SHA `0000…` is fine for the repo artifact; only regenerate via real `git format-patch` from a clone if you're hand-delivering the patch to a maintainer who specifically asked for it.
5. Extend the [orchestrator.js](orchestrator.js) `register()` flow to collect per-locale vocab for this library and update the relevant hooks. See [the four orchestrator touch-points](#orchestrator-the-four-places-to-touch) below.
6. Extend the LocaleSpec typedef + a per-library section in [scripts/gen-locales.mjs](scripts/gen-locales.mjs) `renderLocaleFile`.
7. Add a test phase to [test/loka-js.spec.mjs](test/loka-js.spec.mjs) and a behavior-preservation case to [test/preservation.mjs](test/preservation.mjs).
8. Write a demo under `demo/<lib>/` and a tutorial page at `tutorial/es/<lib>.html`.

### Orchestrator: the four places to touch

When you reach step 5 above, the orchestrator needs changes in four places. The current code (paxi, moxi, ssexi, rexi) shows all four patterns; copy the closest match for your new library.

1. **Per-library state**, at the top of the file in a `// ── <libname> ──` section. Examples:
   - `let MORPH_NAMES = new Set(['morph'])` (paxi — Set of localized swap-strategy aliases)
   - `let LIVE_NAMES = new Set(['live']); let ON_PREFIXES = new Set(['on-']); let MX_MODIFIERS = {}` (moxi — three slots)
   - `let SSE_REFIRES = new Set()` (ssexi — set of already-wired re-fire event names)
2. **Hook installation** — `let X = window.<libname> ??= {}` then `X.<hookName> = (args) => ...`. The hook closes over the per-library state so updates are picked up live (the library captures the function by reference; the function reads state at call time).
3. **Collector function** — `let collect<Lib> = (data, code) => { ... }`. Reads `data?.<libname>?.<vocabKey>` from the locale block and updates the per-library state. Pattern: iterate `Object.entries(...)` of the localized-to-canonical map, push localized values into the state slot.
4. **Wire into `register()`** — add a single call `collect<Lib>(data, c)` in the chain (currently after `collectMoxi`). If your library has `globals` to alias onto `globalThis`, **also add the library name to the hardcoded `['paxi', 'rexi', 'moxi']` list in `collectAliases`** — this is the one piece of duplication left in the orchestrator (see the design note below).

### Design note: why no LIBRARIES table

We considered a data-driven `LIBRARIES` table (each entry has `name` + `collect` fields; `register()` and `collectAliases` iterate it). **Skipped on 2026-05-25** because:

- hyperfixi has no analogous pattern to align with (`DomainRegistry` exists but solves a different problem — heavyweight DSL/MCP plumbing, not thin per-library hook installation).
- Net win was ~5 lines saved with the same legibility.
- Per-library state slots (`MORPH_NAMES`, `LIVE_NAMES`, etc.) are heterogeneous in shape and don't tabulate cleanly without losing the per-library readability.

**Architectural boundary.** The orchestrator's responsibility is bounded to **fixi-family hook installation** — patched libraries whose `??=` slots need filling at load time. Libraries with their own native registries (e.g. psatina) don't go through the orchestrator at all; they import `dom-vocab/<lang>.js` directly and call their own native `register` / `set` method. Adding "library N+1 to the loka ecosystem" usually doesn't mean N+1 collectors in the orchestrator — it often means a new entry in `dom-vocab/` only (or nothing, if the library shares vocabulary that's already there). [experiments/psatina-study/](experiments/psatina-study/) is the existence proof.

**Revisit the LIBRARIES-table refactor if:**

- The fixi-family itself grows past ~8 libraries (currently 5: fixi, moxi, paxi, ssexi, rexi).
- A new library that genuinely needs `??=` hook installation (no native registry, like fixi-family) joins the orchestrator.

**Do NOT revisit just because** a non-fixi-family library starts using loka vocab. That's the dom-vocab path — already worked out via psatina-study and doesn't pressure the orchestrator's shape.

## Footguns we hit in v1

These will bite again if you don't watch for them.

- **Duplicate-id after a fixi swap.** `document.querySelector("#x")` returns the first match in document order. If a fixi swap loads a fragment containing `<div id="list">` into `#fixi-out`, and another `#list` exists later in the page, subsequent `fx-objetivo="#list"` resolves to the wrong element. Fix: unique IDs across the document, or scope your queries.

- **paxi's `morph` strips attributes** (including `id`) from descendants that don't match the new tree's attribute set. If you preserve an element by id in the parent but the inner `<input>` has no id and the new fragment's input doesn't have it either, the original input's `id` is removed during attribute reconciliation. Don't rely on inner-element ids surviving a morph unless those ids exist on both sides.

- **HTML `value` attribute vs `.value` property** diverge after user input. `morph` sets the attribute; the dirty property is unchanged. This is *why* morph preserves typed content — but it means your test/assertion must use `inputValue()` not `getAttribute('value')`.

- **`ignoreSel` is captured at library load.** Both fixi and moxi cache `igSl = h.ignoreSel ??= "[fx-ignore]"` as a string at IIFE-evaluation time. `register()` updates `window.<lib>.ignoreSel` but the captured local stays stale. In practice locales load before the libraries so first-load is fine; mid-page registration of a localized `fx-ignore` won't take effect. Don't depend on post-load ignoreSel changes.

- **moxi's handler-scope `q` differs from `window.q`.** Handler-scope is `mkq(elt)` (element context for `next/closest/in this` DSL). The global is `mkq(documentElement)`. Aliasing `consulta = window.q` means `consulta('next btn in this')` from inside a handler resolves `this` to documentElement, not the element. Simple CSS selectors are unaffected. Document this if a Spanish-speaking user complains.

- **Per-element-lang means mixed-prefix on the same element fails silently.** `<section lang="es"><button al-clic="..." on-click="...">` — only `al-clic` is processed because the element's lang resolves to `es` and the prefix is `al-`. The English form on a Spanish-context element is invisible. Acceptable; document if it surprises someone.

- **ssexi event re-fire is cancellation-independent.** Calling `e.preventDefault()` on the canonical `fx:sse:open` does **not** cancel the re-fired `fx:sse:abrir`. SSE events are mostly notifications so this is OK, but don't rely on cross-language cancellation propagation.

## Patch discipline (we are a fork, not a PR queue)

loka-js is an **alternate distribution** of the fixiproject family. We ship patched copies of fixi/moxi/paxi; we do not pursue upstream merges. The rationale: fixiproject's minimalism is a deliberate position (every byte matters in 1.8–3.5 KB libraries; the hook overhead benefits ~5% of users), and asking Carson to absorb that cost on behalf of our audience is the wrong frame. Localized authoring vocabulary is a parallel-distribution concern, not a core-library concern.

The patches in [reference-patches/](reference-patches/) are **documentary artifacts**, not PR submissions. They show the diff between our patched copies and upstream HEAD at pinned commits. Useful for:

- Tracking drift when upstream changes (re-port the patch against new HEAD)
- Explaining to a curious maintainer or contributor what we changed and why
- Hand-delivering to a maintainer in the unlikely event they want to consider the hooks

Keep edits to the patched library files:

- **Bit-identical to upstream when no orchestrator is loaded** — verified by [test/preservation.mjs](test/preservation.mjs). This contract lets loka-js users who don't load a locale get unchanged upstream behavior, and lets us re-port cleanly.
- **Faithful to upstream code style** — single IIFE, `??=` defaults, short var names (`nm`, `ev`, `sl`, `mxh`, `igSl`), no extra whitespace. Not because we're trying to look mergeable, but because diff minimality reduces re-port friction when upstream changes.
- **Hook surface as small as it can be** — every hook is more code to re-port when upstream changes the surrounding logic.

**Do not open upstream PRs in an automated session.** If a maintainer ever solicits the patches, that's a human-judgment moment requiring a clone, the upstream test suite to pass, and a PR description tuned to that maintainer's framing.

What IS worth filing upstream (when it comes up): documentation issues that benefit English users too — e.g., "paxi's `morph` strips id attrs from non-matching descendants" or "ssexi's `preventDefault` semantics across lifecycle events." Those are bug reports, not feature requests; the audience is fixiproject's existing users.

## Audience and decision framing

When evaluating design choices in this repo:

- The user is a **beginner web developer whose native language isn't English** — not an experienced English-fluent dev who'll graduate to upstream docs anyway.
- **Don't import "no major framework does this"** as a default critique. Majority frameworks chose a different audience.
- The reviewed-vs-best-effort split exists because bad translations hurt this audience more than experienced devs (who can shrug them off). When in doubt, mark `reviewed: false` and add a banner.

## Translation policy: open questions

We made judgment calls in v1 without a written rulebook. Before more locales get reviewed passes:

- **HTTP verbs** (head→cabecera, put→poner, patch→parchear) — protocol terms or natural words? We translated. Defensible either way.
- **moxi `.cc` modifier** (kebab-to-camelCase) — kept English (technical mechanism). Where's the line vs. `.prevent`?
- **rexi chained methods** (`.json()`, `.text()`) — kept English. So `obtener(...).text()` reads mixed. Acceptable?
- **Prefix translation depth** — `al-clic` (translate both "on" and "click") vs `on-clic` (translate only the event noun) vs `al-click` (translate only the prefix). We chose `al-clic`. A reviewer might disagree.

If you're adding a new locale's vocabulary, document the choices you make in a header comment so the next reviewer can match the style.

## Testing

```bash
# Acceptance suite (needs static server on :3002)
npx http-server . -p 3002 -c-1 -s &
npm test                                      # node test/loka-js.spec.mjs

# Behavior-preservation harness (needs static server on :3001)
npx http-server . -p 3001 -c-1 -s &
node test/preservation.mjs
```

Both scripts honor `LOKA_BASE_URL` env var if you need alternate ports.

When a test fails, prefer fixing the test or the demo over loosening assertions — every spurious-looking failure in v1 turned out to be a real bug (duplicate ids, attribute stripping, wrong selector). Trust the harness.

---
> Source: [codetalcott/loka-js](https://github.com/codetalcott/loka-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
