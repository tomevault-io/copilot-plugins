## testing-and-qa

> Use Python Playwright (`test-visual.py`) for visual QA.

# Visual Testing & QA

## Playwright Setup
Use Python Playwright (`test-visual.py`) for visual QA.
Viewport should mimic Cursor's bottom panel: wide + short (e.g., 900x220).

## Click Coordinate Math (CRITICAL)
Playwright clicks are in CSS pixel space, but the canvas uses internal coordinates with offset.
To click on object at grid position (col, row):

```python
x = offset_x + col * TILE_SIZE * scale + hitbox_center_x * scale
y = offset_y + row * TILE_SIZE * scale + hitbox_center_y * scale
```

The `offset_x` and `offset_y` are calculated from viewport size - scene size.
If clicks don't register, the offset math is wrong. This was the most common bug during testing.

## Build + Test Loop
```bash
cd cursor-office && npm run build
open dev/playground.html
```
Then open `dev/playground.html` in browser to preview, or package as VSIX with `npx vsce package --no-dependencies`.

## dev/playground.html — Visual Sandbox
`dev/playground.html` stubs the VS Code API and loads `dist/webview.js`.
Has **control buttons** at the bottom to simulate activity states:
- Idle, Working, Phone Call, Celebrate, Error
- Active button gets a white outline indicator.
- These call `window.postMessage({ type: 'agentStatus', activity })` to trigger state changes.
- Use this to test all state transitions visually without needing a real Cursor agent.
- Plugin authors: load your plugin script after `webview.js` to test custom objects.

## What to QA After Any Visual Change
1. Character sits correctly at desk (rear-facing, hands at keyboard level)
2. Character phones correctly (rear-facing, left arm raised with handset, near phone on desk)
3. Character walks to EACH waypoint and faces the right direction
4. Objects are clickable (especially cat, arcade, plant — small hitboxes)
5. Speech bubbles don't clip at canvas edges
6. Z-order: character behind desk when sitting, in front of rug when walking
7. "Working..." dots animate while at desk, "Delegating..." while phoning
8. Phone on desk vibrates during phoning, stops when activity changes
9. Celebration auto-expires after ~4 seconds
10. Error shows ⁉️ bubble then auto-transitions to idle
11. Lamp on/off toggles dimmed overlay
12. Window shows correct time-of-day cycle
13. Arcade cycles mini-game animations on click

## Screenshot-to-GIF Pipeline
Take individual screenshots with Playwright, then combine:
```bash
magick -delay 200 -loop 0 img1.png img2.png img3.png demo.gif
```
Output goes to `assets/demo.gif` for the README.

---
> Source: [ofershap/cursor-office](https://github.com/ofershap/cursor-office) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
