## fm-viewer

> Instructions for AI agents (Claude Code, Codex, etc.) picking up this project.

# FM Viewer — Agent Notes

Instructions for AI agents (Claude Code, Codex, etc.) picking up this project.

## What this is

A static single-page app that visualizes Apple Intelligence Report JSON exports (Siri / Apple Foundation Models request traces, on-device + Private Cloud Compute). No build step, no framework, no dependencies. A handful of files do everything:

| File | Role |
|------|------|
| `generate-data.mjs` | Node script. Parses raw report JSON from `reports/` into a normalized bundle and writes `data.js` (`window.FM_REPORTS_DATA`). All parsing/classification logic lives here. |
| `blob-decode.mjs` | Dependency-free, side-effect-free decoders for the opaque base64 blobs (PCC attestation bundles → protobuf/DER/CBOR/JSON tree; `thought_signature` → described, not decoded). Imported by the generator. |
| `blob-decode.test.mjs` | `node --test` suite for the decoders, using synthetic fixtures (no personal data). |
| `app.js` | Vanilla JS UI. Renders from `window.FM_REPORTS_DATA`, holds all state in a single `state` object, re-renders via innerHTML. No virtual DOM, no libs. |
| `index.html` + `styles.css` | Static shell and styling. `app.js` targets elements by ID (`#list`, `#inspector`, `#categoryRail`, …) — keep IDs stable or update both sides. |

## Data flow

```
reports/*.json  →  node generate-data.mjs  →  data.js  →  index.html (script tag)  →  app.js renders
```

- `data.js` and `reports/*.json` are **gitignored** — they contain the owner's personal Siri requests. Never commit them, never paste their contents into PRs/issues.
- The generator is idempotent; regenerate any time with `node generate-data.mjs`.
- The bundle global is `window.FM_REPORTS_DATA` (`{ generatedAt, reports: [...] }`). `window.SIRI_REPORTS_DATA` / `window.SIRI_REPORT_DATA` are legacy aliases kept for old bundles.

## Key invariants (don't break these)

1. **Scopes**: Prompts / Thread / Decoded Strings / Tools / PCC — driven by `state.scope` and `getDataset()` in `app.js`.
2. **Report picker**: multiple reports per bundle, switching resets filters (`switchReport`).
3. **Category rail**: counts recomputed per scope/filter, colors from the generator's `categoryColor` map.
4. **Filters**: model, use case, role, dedupe (fingerprint-based, entries scope only), search across title/content/tags/tools, four sort modes, Reset.
5. **Rows**: click selects + toggles inline expansion (full content with Copy/Collapse); thread rows expand into per-role `<details>` nodes plus matched PCC runs.
6. **Inspector**: metadata grid, tags, decoded fragments, line-numbered code view with light syntax highlight, Copy / Copy JSON / Wrap toggle.
7. **Keyboard**: `/` focuses search, `Escape` blurs, Enter/Space activates rows.
8. **Responsive**: ≤1180px the inspector becomes a slide-over; ≤760px single-column with sticky header. Test both — iPhone/iPad use is a first-class scenario.
9. **No-data state**: opening `index.html` without `data.js` must show onboarding instructions, not a blank page or console crash.

## Verifying changes

```bash
node --test                            # decoder unit tests (blob-decode.test.mjs)
node generate-data.mjs                 # must print "Wrote N report(s) to .../data.js"
python3 -m http.server 8080            # then check http://localhost:8080
```

Check: all five scopes render, search narrows results, a row expands, the inspector populates, and the mobile layout (≤760px) works. The decoders have unit tests; the UI's test is the browser.

The blob decoders are heuristic and order-sensitive (text → DER → CBOR → protobuf → raw, because each format's leading bytes can masquerade as the next). If you touch `blob-decode.mjs`, run `node --test` and re-check a PCC node in the viewer. When Apple changes the bundle format, expect new top-level shapes; the decoder degrades gracefully to a hex+strings preview rather than throwing.

## Context

- Origin: built June 2026 to inspect iOS 27 Siri traces.
- Report exports come from Settings → Privacy & Security → Apple Intelligence Report on device.
- Apple changes the prompt format between OS builds. If new reports parse weirdly, the regexes at the top of `generate-data.mjs` (`roleTokenRe`, `systemToolsRe`, `toolCallRe`) are where to look first.

---
> Source: [viticci/fm-viewer](https://github.com/viticci/fm-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
