## null-signal

> This repo's current agent handoff is `CONTEXT.md`. Read that first.

# CLAUDE.md

This repo's current agent handoff is `CONTEXT.md`. Read that first.

High-level facts:
- `null_signal` is a live VJ instrument for minimal techno.
- Runtime is native ES modules, p5.js, Web Audio, Electron, and vanilla DOM/CSS.
- The app has a two-window Electron flow: control window plus output renderer.
- There are 46 visual modes, including local-LLM modes ORACLE and INTERPRETER.
- Local LLM defaults point at `~/llm-server`.
- There is no renderer build step, no linter, and no test suite.

Useful commands:

```sh
npm run electron
npm stop
npm run serve
npm run dist
npm run dist:mac
```

Source of truth:
- `README.md` for operator-facing usage.
- `CONTEXT.md` for architecture and handoff details.
- Source files when docs and code disagree.

---
> Source: [cyberpatrolunit/null-signal](https://github.com/cyberpatrolunit/null-signal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
