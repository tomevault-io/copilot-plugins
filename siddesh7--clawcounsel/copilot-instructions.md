## clawcounsel

> Terminal-style UI conventions for ClawCounsel frontend


# Terminal UI Aesthetic

All pages use a retro terminal look: green text on black, monospace fonts, scanlines.

## CSS Variables (from globals.css)

- `--term-bg: #030303` — background
- `--term-green: #00ff41` — primary text
- `--term-green-mid` — secondary text
- `--term-green-dim` — muted text
- `--term-amber: #ffb000` — warnings
- `--term-border`, `--term-surface` — borders/cards

## Utility Classes

- `.term-glow` / `.term-glow-static` — text glow effects
- `.term-btn` — terminal-style buttons with hover
- `.term-box` / `.term-box-glow` — card containers
- `.term-input` — input fields
- `.term-statusbar` — top status bar
- `.cursor-blink` — blinking cursor animation
- `.font-display` — VT323 retro font

## Rules

- Use inline styles with CSS variables (not Tailwind classes for theming)
- Keep component files under 120 lines
- No emojis in UI — use ASCII symbols (▸, ●, ○, ←, →)
- Status indicators: `#00ff41` for good, `var(--term-amber)` for warning, `#ff4444` for critical
- Fonts: VT323 for headings, IBM Plex Mono for body
- All caps with letter-spacing for labels (e.g. `letterSpacing: "0.2em"`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Siddesh7)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Siddesh7)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
