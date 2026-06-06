## claude-farsi-rtl

> Tiny Chrome MV3 extension. No build step, no dependencies, no network calls.

# CLAUDE.md — claude_extention

Tiny Chrome MV3 extension. No build step, no dependencies, no network calls.
The font is bundled. After any change, the user reloads via
`chrome://extensions` and refreshes the host tab.

## Scope

Runs on three hosts (declared in `manifest.json`):

- `https://claude.ai/*`
- `https://chatgpt.com/*`
- `https://chat.openai.com/*` (legacy domain — redirects to chatgpt.com, kept for safety)

The extension is still named **"Claude.ai Farsi RTL"** even though it covers
ChatGPT too. Do not rename the extension or files unless the user asks —
they explicitly deferred rebranding.

## Features (all in one content script)

1. **Per-block Farsi RTL** — `MutationObserver` over the page; for each prose
   block (`<p>`, `<li>`, headings, etc., *not* `<div>`), sample ~50 chars and
   flip to `dir="rtl" lang="fa"` if Persian letters exceed `THRESHOLD` (0.4).
   Nested English blocks inside a Farsi container get flipped back to LTR.
   `<pre>` / `<code>` are skipped.

2. **Font picker** — popup writes `farsiFont` / `englishFont` to
   `chrome.storage.local`; content script mirrors them onto
   `documentElement` as `--farsi-font` / `--english-font` CSS variables that
   `styles.css` reads.
   - Font lists live in `FARSI_FONTS` / `ENGLISH_FONTS` arrays at the top of
     `popup.js` — keep the defaults aligned with `DEFAULT_FARSI_FONT` /
     `DEFAULT_ENGLISH_FONT` in `content.js`.
   - Custom values from older installs are preserved by `fill()` — it
     appends a "Custom (…)" option when the stored value isn't in the list.
   - The English font is only visible when an English block sits inside a
     Farsi-marked container (`[data-farsi-rtl="0"]`). In English-only
     conversations nothing gets tagged and the dropdown appears inert —
     this is by design. Don't "fix" it by applying the font globally; that
     would override the host site's typography.
   - `applyPreview()` updates the in-popup preview elements; live propagation
     to the chat tab is via the content script's `chrome.storage.onChanged`
     listener — no tab message needed.

3. **Synced prompt library** — popup stores prompts in
   `chrome.storage.sync` (Chrome syncs them across browsers / laptops
   signed into the same Google account — no network code on our side,
   no new permissions) and sends `{type: 'cfr-insert', body, position}`
   via `chrome.tabs.sendMessage` to the active supported tab.
   - Schema: `{id: string, title: string, body: string}` array under key
     `prompts`. New entries are unshifted (newest first). `id` from
     `crypto.randomUUID()` with a `p_<ts>_<rand>` fallback.
   - **One-time migration:** `migratePromptsToSync()` in `popup.js` runs
     before the first render. If `local.prompts` exists and `sync.prompts`
     is empty, it copies up; then it sets `local._promptsMigrated: true`
     and removes `local.prompts`. The marker is idempotent — once set,
     migration never runs again, so a user who later empties their synced
     list on a fresh device won't get it repopulated from a stale local
     copy. Don't remove the marker check.
   - **Sync limits to remember:** `chrome.storage.sync` caps total at
     ~100KB, per-item at ~8KB, and ~120 writes/min. Plenty for a prompt
     library, but a single 9KB prompt body will silently fail to save —
     if users start hitting this, surface the error rather than swallowing.
   - Title uniqueness is enforced case-insensitively in `popup.js` — don't
     remove that check; the list reads as an index, not a log. The inline
     edit form skips self in the duplicate-title check so the user can keep
     the existing title while editing only the body.
   - Cards show a truncated body preview (`truncateBody()` in `popup.js`)
     capped at `previewChars` chars, configurable from Settings under
     `promptPreviewChars` (clamped to `[20, 2000]`, default 100). The full
     body is always what gets inserted and what stays in storage.
   - Each card has a ✎ (inline edit) and × (delete) button on one row;
     ✎ swaps the card into an edit form (`buildPromptEditForm`) tracked by
     module-level `editingId` so a `storage.onChanged` re-render doesn't
     clobber an in-progress edit.
   - `position` is `'top' | 'cursor' | 'end'`. `'cursor'` requires that the
     user clicked into the composer before opening the popup — the popup
     steals focus, so `content.js` tracks `lastEditor` / `lastRange` via a
     `selectionchange` listener. Don't try to read the live selection from
     the popup-spawned context — it'll be empty.
   - `findComposer()` picks the bottom-most `contenteditable="true"` /
     `div[role="textbox"]` wider than 100px. This works on both Claude and
     ChatGPT (both ProseMirror). If a site lands with a different composer
     shape, prefer extending this heuristic over hardcoding a selector.
   - Insertion path: `execCommand('insertText')` first, then a
     `beforeinput` `InputEvent`, then clipboard fallback. All three return
     a structured `{ok, error}` to the popup so `flashStatus()` can show
     the right message.
   - The popup auto-closes ~250ms after a successful insert so focus
     returns to the composer.

4. **Feature toggles (Settings tab)** — master `enabled` plus per-feature
   `rtlEnabled`, `fontsEnabled`, `promptsEnabled`, plus a storage-area
   switch `syncPromptsEnabled`. All booleans in `chrome.storage.local`,
   all default `true`. The list lives in `TOGGLE_KEYS` in `popup.js`;
   the master-gated subset lives in `MASTER_GATED_TOGGLES`.
   - When master is off in the popup, the master-gated per-feature
     switches go visually disabled but keep their stored value
     (re-enabling master restores prior layout). `syncPromptsEnabled`
     is **not** master-gated — storage choice is orthogonal to whether
     the extension is currently active, and disabling it when master is
     off would prevent a user from changing their sync preference while
     the extension is paused.
   - content.js applies them live via `chrome.storage.onChanged`:
     RTL off → `observer.disconnect()` + strip `data-farsi-rtl` / `dir` /
     `lang`. Fonts off → remove `--farsi-font` / `--english-font` CSS vars.
     Prompts off → the `cfr-insert` handler returns
     `{ok:false, error:'Prompt insertion is disabled in Settings.'}`.
   - `syncPromptsEnabled` is read only by `popup.js` — content.js never
     touches the prompt list, so it doesn't care about the storage area.
     When the toggle flips, `migratePromptsOnToggle()` copies the current
     visible list from the OLD area into the NEW area so the user's list
     doesn't change underneath them. **The source is intentionally not
     deleted** — flipping back restores that snapshot. The cached
     `syncPromptsEnabled` mirror in `popup.js` is updated both on local
     flip and on `storage.onChanged` (so a second popup instance stays
     in sync with the flip).
   - The Settings tab also holds the `promptPreviewChars` number input.
     Don't move it into the Prompts tab — keep all knobs in Settings.

## Conventions

- **Site-agnostic content script.** Don't add per-site DOM selectors —
  the script keys off generic tags (`<p>`, `contenteditable`, `role="textbox"`)
  so it works on both Claude and ChatGPT (and survives their DOM changes).
- **Three URL lists must stay in sync** in `manifest.json`:
  `host_permissions`, `content_scripts[0].matches`,
  `web_accessible_resources[0].matches`. And `SUPPORTED_URLS` in `popup.js`.
- **No new permissions** beyond `storage` + the three host matches. The
  extension's pitch is "no network calls" — keep it that way.
- **Storage keys in use:**
  - `chrome.storage.sync`: `prompts` *only when `syncPromptsEnabled` is true*
    (so the list follows the user across devices). When the toggle is
    false, the list lives in `chrome.storage.local` under the same key.
    Both areas may hold a snapshot — the "live" one is whichever the
    current toggle points at; the inactive one is a snapshot from the
    last time the toggle pointed there.
  - `chrome.storage.local`: `farsiFont`, `englishFont`, `enabled`,
    `rtlEnabled`, `fontsEnabled`, `promptsEnabled`, `syncPromptsEnabled`,
    `promptPreviewChars`, `_promptsMigrated` (the one-time sync-migration
    marker), and a copy of `prompts` whenever sync is off (or as a
    snapshot from a previous off-period). Fonts and toggles stay local
    by design — fonts are device-dependent (different OS → different
    installed fonts) and toggles are typically per-browser preferences.
- **Composer message protocol:** `{type: 'cfr-insert', body: string, position: 'top' | 'cursor' | 'end'}`.

## Testing

No automated tests. Manual flow:

1. Reload extension at `chrome://extensions`.
2. Open / refresh a claude.ai *and* a chatgpt.com tab.
3. Paste mixed Farsi/English text — verify per-block RTL.
4. Open popup → Fonts → change font → see live preview + chat updates.
5. Open popup → Prompts → save one → click Top / Here / End → verify insert.
6. Open popup → Prompts → ✎ on a card → edit body → Save → verify update.
7. Open popup → Settings → toggle master off → verify RTL reverts, font
   vars cleared, prompt insert returns the disabled error. Toggle each
   per-feature switch independently and confirm only that feature stops.
8. Open popup → Settings → change "Prompt preview length" → switch to
   Prompts → verify card truncation respects the new value.

## Files

- `manifest.json` — MV3 manifest, three host patterns.
- `content.js` — RTL detector, font CSS-var wiring, prompt insertion handler.
- `styles.css` — `@font-face` for bundled Vazirmatn + RTL rules keyed off `[data-farsi-rtl="1"]`.
- `popup.html` / `popup.css` / `popup.js` — tabbed popup (Prompts + Fonts + Settings).
- `fonts/Vazirmatn-{Regular,Bold}.woff2` — bundled webfont (OFL).
- `icons/` — extension icons.

---
> Source: [alirahmani93/claude-farsi-rtl](https://github.com/alirahmani93/claude-farsi-rtl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
