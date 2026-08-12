## harness-v2

> Project knowledge for coding agents. See [README.md](README.md) for what the project is

# harness-v2

Project knowledge for coding agents. See [README.md](README.md) for what the project is
and how to run it.

Keep this file short. Add a rule only after it earns its place — that is, after a wrong
assumption costs real time. An empty section is better than a guess.

## Rules

**A throwaway probe that touches main-process state must be a `.ts` file, not `.mts`.**
The main sources are CJS. Under `tsx`, a `.mts` file loads them as a second module
instance, so `initWorkspace()` called from the probe stays invisible to `supervisor.ts`,
and `getWorkspace()` throws "workspace has not been initialised". Write the probe as
`scripts/tmp-*.ts` with an `async main()`, and run it with
`npx tsx --tsconfig tsconfig.node.json`. Delete it before you commit.

---
> Source: [szymon-pitula/harness-v2](https://github.com/szymon-pitula/harness-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
