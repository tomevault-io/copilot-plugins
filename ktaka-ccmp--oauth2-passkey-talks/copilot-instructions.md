## oauth2-passkey-talks

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Presentation repository for `oauth2-passkey` — a Passkey (WebAuthn) + OAuth2 authentication library for Rust/Axum. This repo holds slides, posters, and planning docs for multiple events.

- **This repo**: presentations (slides, posters, diagrams, planning docs)
- **Source repo**: `~/GitHub/oauth2-passkey` (the actual Rust library)
- **Languages**: content may be in Japanese or English

## Directory Structure

```
├── 2026-03-31-tokyo-rust/       # Per-event directory
│   ├── talk/                    # Marp slides
│   │   ├── slides.md            # v1 (original)
│   │   └── slides-v2.md         # v2 (current, restructured flow)
│   ├── poster/                  # Typst A0 poster
│   │   └── poster.typ
│   ├── diagrams/                # Event-specific diagrams (Mermaid SVGs)
│   │   ├── *.mmd               # Mermaid source files
│   │   └── *.svg                # Generated SVGs
│   ├── TALKING_POINTS.md        # Topic list with priorities
│   └── Passkey_OAuth2-Osaki-rs-20250325__1_.pdf  # Previous LT reference
├── shared/                      # Cross-event assets
│   ├── qr-demo.svg              # passkey-demo.ccmp.jp
│   ├── qr-github.svg            # github.com/ktaka-ccmp/oauth2-passkey
│   └── qr-contact.svg           # ktaka.blog.ccmp.jp/p/p.html
├── CONTEXT.md                   # Event info, outlines, demo plan
└── CLAUDE.md                    # This file
```

New events: create `YYYY-MM-DD-event-name/` with `talk/`, `poster/`, `diagrams/` subdirectories.

## Tools

### Marp (Slides)

```bash
# Preview with live reload
npx @marp-team/marp-cli --preview 2026-03-31-tokyo-rust/talk/slides-v2.md

# Export to PDF (--allow-local-files required for local images)
npx @marp-team/marp-cli 2026-03-31-tokyo-rust/talk/slides-v2.md --pdf --allow-local-files -o slides-v2.pdf
```

### Typst (Poster)

```bash
# Compile A0 poster (--root . needed for cross-directory image paths)
typst compile 2026-03-31-tokyo-rust/poster/poster.typ --root .
```

### Mermaid (Diagrams)

```bash
# Generate SVG from .mmd source
mmdc -i diagrams/foo.mmd -o diagrams/foo.svg -b transparent
```

### QR Codes

```bash
# Generate QR code SVG
qrencode -o shared/qr-foo.svg -t SVG -l M "https://example.com"
```

## Slide Design Notes (Marp)

- Marp default theme overrides `display` and `padding` on section via high-specificity selectors. Use `!important` on `display: flex`, `flex-flow`, `justify-content`, and `padding`.
- `bg right` for images stretches to full height. Use `class="with-qr"` (custom CSS grid) for QR code placement instead.
- 2-column layouts: use `class="columns"`, `class="columns-60-40"`, `class="columns-40-60"` defined in the slide CSS.
- Image paths: event diagrams use `../diagrams/`, shared assets use `../../shared/`.

## Workflow Tools

### Available Commands

| Command | Description |
|---------|-------------|
| `/snapshot` | Create a session snapshot for context transfer between machines |
| `/issue` | Create or update an issue for task/bug tracking |
| `/backlog` | View all open issues |

### Issue Tracking (`.claude/issues/`)

- **Filename**: `YYYYMMDD-HHMM-<short-slug>.md`
- **Status**: `open`, `completed`, `wontfix`, `deferred`
- **Priority**: `high`, `medium`, `low`
- **Format & Rules**: See `.claude/issues/README.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ktaka-ccmp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
