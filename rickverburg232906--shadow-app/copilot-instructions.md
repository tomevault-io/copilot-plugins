## shadow-app

> Guidance for AI coding agents working in this repo. Read this before making changes.

# AGENTS.md

Guidance for AI coding agents working in this repo. Read this before making changes.

## Working rules (read first)

**At the start of every reply in this project, output this line verbatim
as the first thing:**

```
✓ AGENTS.md
```

This lets the user verify you've loaded these rules. When you break a
rule, say so in the reply — "I used a hex instead of a var, fixing" beats
pretending. When a choice is shaped by a rule, cite the section briefly
(e.g. "per Hard nos, no refactor here").

### Before writing code

- **Grep, don't read.** `inschrijftafel.js`, `member.js`, `data.js` are
  60–100KB. Use `Select-String` to locate the section first.
- **Check `style.css` first** for existing patterns: `--blue`,
  `--yellow`, `.surface-card`, `.page-blocker-overlay`,
  `.manual-locked-overlay`. Don't invent parallel styles.
- **Firestore casing is exact:** `Naam`, `Voor naam` (with space),
  `Tussenvoegsel`, `Jaarhanger`, `ScanDatums`, `lunchDeelname`,
  `lunchKeuze`, `lunchExpires`. Wrong case = silent duplicate field.

### CSS → always in `style.css`

- Never inject CSS via `<style>` from JS.
- Use `var(--blue)`, `var(--yellow)`, `var(--bg-light)` — not hex.
- `font-family: inherit` — Plus Jakarta Sans cascades from body.
- Z-index ladder (don't invent new numbers): nav `40`, footer `50`,
  admin bottom-nav `60`, page blocker `9999`, modal backdrop `99998`,
  modal content `99999`.
- Dutch for all user-facing strings. Comments can be English.

### JS (match existing style, don't modernize)

- Keep `try { ... } catch(_){}` defensive wrappers — iOS WebKit needs
  them. Don't "clean up" to async/await with proper error handling
  unless asked.
- All Firestore access goes through `src/firebase.js`. No direct
  `firebase/firestore` imports elsewhere.
- Reuse existing helpers from `src/utils.js`: `tsToMillis`, `tsToIso`,
  `toYMDString`, `todayYMD` (timestamp/datum normalisatie),
  `formatMemberName`, `getLidnr`, `getMemberInitials` (member helpers).
  Other in-file helpers: `normalizeYesNo`, `escapeHtml`.
- Guard member objects:
  `Object.assign({ lidnummer: knownId }, memberDoc || {})`.

### Comments — kort

- **1 regel waar mogelijk.** Max 3 regels voor non-obvious WHY (bv.
  een Firestore-quirk, race condition, of waarom we het *niet* op de
  voor de hand liggende manier doen).
- **Geen narratie.** Niet uitleggen wát de code doet als de code dat
  zelf laat zien. Niet samenvatten van een functie in proza.
- **Geen historie in comments.** "Pass 12 (date): X verving Y" hoort
  in de git-commit, niet in de code. Verwijder zulke regels bij
  aanrakingen.
- **Section dividers (`// === foo ===`) en TODO/FIXME blijven.**
- **Geen defensieve "wat als..." uitleg** rond een try/catch — de
  catch zelf is de uitleg.

Slecht (narratie, herhaalt wat de code doet):
```js
// Tellen per keuze uit huidige _currentExtras. We bouwen een Map omdat
// we straks per keuze een lookup willen doen, en we filteren lege keuzes
// weg zodat de UI geen ghost-rijen toont.
const counts = new Map();
for (const item of _currentExtras) { ... }
```

Goed (alleen WHY, en alleen het non-obvious deel):
```js
// Map ipv object: behoud insertion order voor consistente render.
const counts = new Map();
for (const item of _currentExtras) { ... }
```

Of nog beter: helemaal geen comment, want de code spreekt voor zich.

### After every change

- Run `node --check src/<file>.js`. This is the only safety net — no
  linter, no tests.
- Grep for the pattern you replaced — expect 0 matches.
- Flow/routing changes: test both `npm run dev` AND `npm run preview`.
- Bij een afgeronde taak in `IMPROVEMENTS.md`: verwijder de hele entry
  (heading + body + trailing `---`). De git-commit message is het
  record. Laat IMPROVEMENTS.md alleen openstaande items bevatten.

### Hard nos

- No new npm deps (Firebase is gstatic CDN).
- No TypeScript, ESLint, Tailwind, framework.
- No editing `assets/firebase-runtime-config.js` (auto-generated).
- No unrequested features (progress bars, shortcuts, sounds,
  animations).
- No refactor mixed with feature work — separate passes.

### Ambiguity

When the spec is ambiguous (e.g. "5 seconds" could mean auto-dismiss
or click-to-dismiss), **ask before building**. Don't guess.

---
> Source: [RickVerburg232906/Shadow-App](https://github.com/RickVerburg232906/Shadow-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
