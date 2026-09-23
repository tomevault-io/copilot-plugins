## nitra-fe-event-registration-assignment

> Event Registration Wizard — Vue 3.5.17 + Quasar 2.18.5 + UnoCSS. Interview assignment.

# CLAUDE.md

Event Registration Wizard — Vue 3.5.17 + Quasar 2.18.5 + UnoCSS. Interview assignment.
Narrative rationale lives in `PLAN.md`; this file is operational rules only.

**Where the work stands: `PLAN.md` §1, phase table and progress log.** Update it when a phase
completes — status and the delivering commits, not elapsed hours. Deliberately not mirrored here;
duplicated progress goes stale on one side and then misleads.

## Commands

```bash
nvm use 22.17.0   # package.json pins this; do not develop on another major
yarn dev          # http://localhost:9001 (note: not 9000, see quasar.config.js)
yarn build
```

Run `yarn lint && yarn format` before each commit.

## Source precedence

When sources conflict: **official assignment doc > repo README > Figma mockup**.
The Figma file is internally inconsistent in places — see `PLAN.md` §3 before treating a frame as
specification.

## Hard rules

- **No hardcoded hex.** Use UnoCSS semantic shortcuts (`text-neutral`, `bg-surface-l1`,
  `border-danger-emphasis`). Full list in `src/unocss/semantic.js`. This is an explicit grading
  criterion.
- **No arbitrary radius values.** Use the theme radius scale (2 / 6 / 10 / 12 / full). Do not write
  `rounded-[10px]`.
- **All date handling is UTC.** Group, compare, and display with
  `Intl.DateTimeFormat(..., { timeZone: 'UTC' })`. Never `getDate()`/`getHours()` — mock timestamps
  are `Z` and local-time conversion shifts `ws2` across a day boundary.
- **Money is integer cents.** Format only at the display boundary via `utils/currency.js`.
- **Derived state is `computed`, never `watch`.** There is exactly **one watcher**: the
  `watchEffect` in `src/boot/i18n.js` syncing `<html lang>` to the active locale — a DOM attribute,
  so genuinely external state, and annotated in place. Any further watcher must clear the same bar.
  The other candidate (URL step sync) was cut because that state was derivable.
- **No user-facing string literals.** All copy goes through `vue-i18n` keys (`en`, `zh-TW`).
- **JSDoc on every exported function** — params, return, and non-obvious behaviour.
- Render all event copy from `src/mocks/event.js`. Never hardcode the event name (the design's
  "WebDev Summit 2025" / "TechConf 2025" strings are stale; data says 2028). The static
  `<title>` in `index.html` is the deliberate exception — document metadata, not rendered copy,
  and deriving it in JS would only trade duplication for a title flash. If a title ever needs to
  vary by step or locale, use Quasar's Meta plugin rather than assigning `document.title`.

## Gotchas

- **The primary CTA is orange, not brand teal.** Use `bg-accent-emphasis-rest` (`#FB7429`).
  `--q-primary` in `src/css/colors.scss` is bound to brand teal, so Quasar's `color="primary"`
  renders the wrong button.
- **Use `text-accent-default` / `text-info-default` / `text-warning-default`, never the bare
  `text-accent` / `text-info` / `text-warning`.** The `-default` forms are the design system's
  canonical names — Figma's token reference frame lists `text/warning/default` and
  `text/info/default`, matching the CSS variables. The bare names are shorthand, and Quasar
  defines those three itself with `!important`, so the shorthand silently renders Quasar's
  palette (`text-warning` gives `#FADD00`, not the design's `#918108`). These are the only three
  colour collisions: every other semantic shortcut carries a suffix Quasar does not define, and its
  palette stops at `-14`.
- **The typography shortcuts collide too, but resolve our way.** Quasar also defines `.text-h1`
  through `.text-h6`, `.text-subtitle1` and `.text-subtitle2`. Unlike the three colour classes it
  declares these _without_ `!important`, and the UnoCSS sheet loads later, so ours win on cascade
  order alone — verified: `text-subtitle1` renders 16/20 at weight 610, not Quasar's 16/28 at 400.
  Nothing to fix, but do not reorder the stylesheets, and be aware a typography class built
  dynamically (which UnoCSS cannot scan) would fall through to Quasar's much larger scale.
- **Translate Figma tokens by value, never by name.** The brand ramp is shifted one step between
  the two: Figma's `bg/brand/muted/rest` is `#EEF6F7`, but the starter's `--bg-brand-muted-rest`
  is `#CBE5E6` — `#EEF6F7` lives in `bg-brand-subtle-rest`. `text/brand/default` and
  `text/brand/emphasis` disagree the same way. Copying the name out of Figma's generated CSS
  silently produces the wrong colour; look the hex up in `src/css/colors.scss` and use whichever
  token holds it. When no semantic token holds the value, use the **palette utility** for that hex
  (`text-teal-500` for `#3A7679`, `text-teal-700` for `#264D4F`, `text-orange-700` for `#A13B02`) —
  still theme-backed, and honest about the value. Do not snap to the nearest semantic token.
- **Inter Variable must be loaded.** The starter ships no webfont; the design's weight tokens
  (630/610/570/485) are meaningless without it.
- **Never use `hidden`.** Quasar declares `.hidden { display: none !important }`, which beats any
  responsive utility, so `hidden tablet:inline` stays invisible at every width. Use `lt-tablet:hidden`
  and friends — a class name Quasar does not define. Audited: of Quasar's 625 `!important` utility
  classes, three overlap this app's markup — `hidden`, `block` and `overflow-hidden` — and only
  `hidden` does harm, since the other two happen to declare what we already want.
- **Quasar's `.flex` sets `flex-wrap: wrap`**, UnoCSS's sets `display` only, so before the
  preflight in `uno.config.js` every `flex` in this app wrapped. That preflight restores
  `flex-wrap: nowrap` for `.flex`; do not remove it, and keep using `flex-wrap` where wrapping is
  intended (utilities are emitted after preflights and still win). `.row` and `.column` carry the
  same rule — avoid those class names entirely.
- **Figma's surface ramp is inverted relative to the starter's.** Figma's `bg/surface/l0` is the
  grey base (`#F4F5F6`), which the starter holds in `bg-surface-l1`; the starter's `l0` is white.
  Same trap as the brand ramp: look the hex up, never copy the name.
- The Figma variable `bg/disable` returns the string `"50"`, not a colour. Use the starter's
  `bg-disable` token.
- Capacity applies to add-ons as well as sessions (`ws2` is 25/25 and sold out), though the README
  only mentions sessions.

## Architecture

State is a `createRegistrationState()` factory provided at the wizard root under a `Symbol` key —
not a module-level singleton. Consume via `useRegistration()`.

```
src/
  composables/   useRegistration useStepper useSessions useAddons usePricing useValidation
  utils/         time.js currency.js validators.js      # pure, JSDoc'd, unit-testable
  components/    steps/ ui/
  i18n/          en.js zh-TW.js
```

Validation is one declarative rule array (`VALIDATION_RULES` in `useValidation.js`) reduced into
the views the UI needs. Stepper badges, the error banner, danger card borders, `— (required)`
review placeholders and Step 1's inline field errors all derive from it — never maintain those
separately. `validate` takes a flat snapshot, not the reactive state, so rules stay pure. A rule
that can fail repeatedly interpolates the offenders into one message rather than becoming a
factory.

Errors are gated behind `submitAttempted`, except `isValid`, which submit must read before any
attempt. The review's `— (required)` text always shows; only its danger colour is gated.

**Composables reading registration state take an optional `{ registration }`**, because the wizard
root provides that state and so cannot inject it. `useStepper` and `useValidation` both do this;
follow the pattern rather than rediscovering the throw.

## Design reference

Figma file key `E60tE1WSZbpG4m8dO4qvfG` (duplicate — the original grants view access only, which the
MCP server rejects). Frame node IDs:

| Frame                          | Node       |
| ------------------------------ | ---------- |
| Step 1 — Attendee Info         | `1069:968` |
| Step 2 — Session Selection     | `1072:912` |
| Step 3 — Add-ons               | `1073:899` |
| Step 3 — Add-ons (Merchandise) | `1149:565` |
| Step 4 — Review & Submit       | `1074:897` |
| Success State                  | `1075:903` |
| Validation Error State         | `1076:904` |
| Design Token Reference         | `1077:896` |
| Shipping Address — States      | `1203:587` |

Load the `figma:figma-implement-design` skill before calling `get_design_context`.

---
> Source: [ellenchialin/nitra-fe-event-registration-assignment](https://github.com/ellenchialin/nitra-fe-event-registration-assignment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
