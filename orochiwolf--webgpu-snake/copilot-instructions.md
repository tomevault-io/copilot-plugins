## webgpu-snake

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Stack

- **Rendering**: WebGPU API + WGSL shaders
- **Language**: Vanilla JavaScript (ES2020+), HTML, CSS
- **Storage**: localStorage (leaderboard)
- **Build tools**: None — single `index.html`, zero dependencies

## Running

```bash
# Option 1: open directly
open index.html

# Option 2: local server
python3 -m http.server
```

Requires a WebGPU-capable browser (Chrome 113+, Edge 113+, Firefox Nightly).

## Related Docs

- [ARCHITECTURE.md](ARCHITECTURE.md) — rendering pipeline, game systems, food type registry
- [STATUS.md](STATUS.md) — current sprint and priorities
- [SESSIONS.md](SESSIONS.md) — history of past work sessions
- [DECISIONS.md](DECISIONS.md) — design discussions and architecture decisions

---
> Source: [orochiwolf/WebGPU-Snake](https://github.com/orochiwolf/WebGPU-Snake) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
