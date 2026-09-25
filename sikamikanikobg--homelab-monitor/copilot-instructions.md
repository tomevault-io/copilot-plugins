## homelab-monitor

> Project-specific instructions for AI agents working in this repo. See

# CLAUDE.md

Project-specific instructions for AI agents working in this repo. See
[`CONTRIBUTING.md`](CONTRIBUTING.md) for the human-facing contributor guide
(branching, the "add a monitor" pattern, style) — it applies here too.

## Conventions

- **Animations**: see [`docs/animations.md`](docs/animations.md) before
  adding any transition, `@keyframes`, or JS-driven value tween. CSS-based
  animations are covered automatically by the reduced-motion kill switch;
  JS number tweens must go through `mcCountUp()`, never a new
  `requestAnimationFrame` loop.

---
> Source: [SikamikanikoBG/homelab-monitor](https://github.com/SikamikanikoBG/homelab-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-25 -->
