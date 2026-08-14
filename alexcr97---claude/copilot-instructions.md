## claude

> In Windows prefer using PowerShell. If a command cannot be executed fallback to the OS's configured shell, e.g. bash.

# Global Claude Code Instructions

## Command Line

In Windows prefer using PowerShell. If a command cannot be executed fallback to the OS's configured shell, e.g. bash.

## JavaScript Package Manager

Always use `pnpm` instead of `npm`. This applies to all commands:

- Install dependencies: `pnpm install` (not `npm install`)
- Add packages: `pnpm add <pkg>` (not `npm install <pkg>`)
- Run scripts: `pnpm <script>` (not `npm run <script>`)
- Execute binaries: `pnpm dlx <pkg>` (not `npx <pkg>`)

---
> Source: [AlexCR97/.claude](https://github.com/AlexCR97/.claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
