## vision-only-execution

> Tool execution acts at the model's grounded point — no label-text DOM lookup


# Vision-only execution (act at the grounded point)

Grounding gives a point; **execution must use that point.** Every tool/action that touches the live page (click, type, select, toggle, clear, focus, blur) resolves its element via **`elementFromPoint` at the model's grounded `[x, y]`** — the `controlAtNorm` / `triggerActionAtNorm` path in `src/browser-tools/dom-actions.ts`.

## Required

- One ShowUI inference → capture-norm point → element at that point → act on it.
- A label→control hop is fine **only** when the point itself lands on the field's `<label>` (point-derived), as is `closest('input,textarea,select,[contenteditable]')`.
- If the grounded point doesn't resolve to a suitable control, the tool **fails honestly** (`ok: false`). The user re-captures or rephrases.

## Forbidden (product code)

- Finding controls by **label text, placeholder, aria-label, visible text, or selectors** in the live DOM (`findControl`-style helpers, `querySelector('[placeholder=…]')`, label-text scans).
- Applying a value via DOM lookup while the grounded point is only used for the marker/cursor — the vision coordinate must be **load-bearing**, never decorative.
- Label-lookup **fallbacks** when grounding misses.

Capture sizing exceptions (`offsetWidth`/`offsetHeight` for SnapDOM resolution) per `no-dom-grounding.mdc` remain allowed.

## Related

- `no-dom-grounding.mdc` — coords come only from the VLA on the screenshot
- `wllama-only.mdc`, `client-side-only.mdc`

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
