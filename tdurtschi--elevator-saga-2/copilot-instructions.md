## elevator-saga-2

> **For UI/rendering changes:** use Playwright specs asserting on DOM structure/behavior.

# Claude Code Guidelines

## Workflow

### Test framework guidance
**For UI/rendering changes:** use Playwright specs asserting on DOM structure/behavior.
**For logic/pure function changes:** use Jasmine unit tests.

Tests also serve as regression guards during refactors — write characterization tests first if the behavior isn't already covered.

Run `npm run ci` — lint and full test suite must pass before committing.

### GitHub PR workflow
Use this workflow whenever the user asks you to use PRs or work unsupervised.
Always work on a feature branch — never commit directly to `master`. Before starting any work:
1. Create a branch: `git checkout -b fix/<issue-number>-short-description` or `feat/<issue-number>-short-description`
2. Do the work and commit on that branch
3. Run `npm run ci` — lint and full test suite must pass before opening a PR
4. Push the branch and open a PR targeting `master`, linking the relevant issue

Note: When running this workflow, take care to avoid getting stuck. It is preferable to provide an update or ask a question directly in the github issue and exit.

### Headless runner
`headless-runner.js` runs challenges without a browser. Two modes:
- `node headless-runner.js --challenge 1 example-solution.js` — run a single challenge, outputs one line of JSON
- `node headless-runner.js example-solution.js` — campaign mode, runs challenges 1, 2, 3… stopping on first failure

Useful for quickly validating elevator solutions or testing simulation changes.

## Running tests

```
npm run test
```

Runs both Jasmine unit tests and Playwright e2e tests. Playwright uses the `list` reporter so results print to stdout.

## Architecture direction

### Completed work
- **Riot.js fully removed:** `riot.render()` and `<script type="text/template">` replaced with vanilla JS template literals in `presenters.js`; `riot.observable()` replaced with `window.unobservable.observable()`; `riot.route()` replaced with a native `hashchange` handler; `libs/riot.js` deleted
- **Monaco CDN removed:** `monaco-editor` npm package installed; `editor.js` extracted from `app.js`; Vite worker config added; CDN script tag removed from `index.html`
- **Playwright tests added** covering floor, elevator, challenge, feedback, and user templates; hash routing; and editor behavior (error display, persistence, reset)

### In progress
- **Editor abstraction:** `editor.js` is extracted but still tightly coupled (jQuery, persistence, AI all imported directly) — next step is cleaning up those dependencies
- **Split `app.js`** god class

### General
- Prefer vanilla JS / direct DOM over jQuery where possible when writing new code
- The long-term goal is a reactive UI layer driven by a `Ticker` abstraction — see `TODO.md` for the full roadmap

## Game mechanics reference

See `docs/PLAYER_API.md` for a full description of the elevator/floor API and how the simulation works (boarding logic, direction indicators, floor button firing behaviour). Useful context when working on anything that touches the game's user-facing behaviour.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tdurtschi)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tdurtschi)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
