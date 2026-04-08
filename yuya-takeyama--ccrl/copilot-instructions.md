## ccrl

> A Slack bot that launches Claude Code remote sessions from Slack. Users run `/ccrl` in a channel, select a repository, and receive a Remote Control URL in the thread — no terminal required.

# ccrl - Claude Code Remote Launcher

A Slack bot that launches Claude Code remote sessions from Slack. Users run `/ccrl` in a channel, select a repository, and receive a Remote Control URL in the thread — no terminal required.

## Tech Stack

- **Runtime**: Node.js 24+
- **Language**: TypeScript (strict mode)
- **Slack framework**: @slack/bolt (Socket Mode)
- **Test runner**: Vitest
- **Linter/Formatter**: Biome
- **Package manager**: pnpm

## Project Structure

```
src/
  index.ts       # Slack app entry point (slash command & view handlers)
  config.ts      # Config loader (ccrl.config.json or CCRL_DIRS env var)
  launcher.ts    # Git worktree creation & claude remote-control spawner
  modal.ts       # Slack modal UI builder
  config.test.ts # Unit tests for config loader
```

## Development

```bash
pnpm dev        # Run with hot reload (tsx watch)
pnpm build      # Compile TypeScript → dist/
pnpm start      # Run compiled output
pnpm test       # Run unit tests
pnpm lint       # Biome lint check
pnpm typecheck  # Type check without emitting
pnpm all        # Run lint + typecheck + test (use before committing)
```

## Implementation Guidelines

- After making any code changes, run `pnpm all` and ensure it passes before committing.
- `pnpm all` runs `lint`, `typecheck`, and `test` in sequence — all three must pass.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuya-takeyama)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/yuya-takeyama)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
