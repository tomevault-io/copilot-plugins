## framework-north-star

> Geometra framework north-star coding focus


# Geometra North Star

Keep this priority order front-and-center when writing code:

1. Protect the core pipeline: `Tree -> Yoga WASM -> Geometry -> Pixels`.
2. Maintain one declarative model across Canvas, Terminal, and server/client protocol.
3. Prioritize correctness in pointer/keyboard/focus/selection/text input/IME flows.
4. Preserve Bun-first, reliable CI behavior.
5. Improve performance in hot paths without sacrificing correctness.
6. Keep docs and exports aligned with real behavior.

# Merge Readiness Checklist

- Changes respect DOM-free invariants.
- Protocol behavior is version-safe and explicit.
- Interaction/input changes include targeted tests where practical.
- No avoidable performance regressions in hit-test, text measurement, or repaint loops.
- README/API documentation remains accurate for user-facing features.

---
> Source: [Agent-Pattern-Labs/geometra](https://github.com/Agent-Pattern-Labs/geometra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
