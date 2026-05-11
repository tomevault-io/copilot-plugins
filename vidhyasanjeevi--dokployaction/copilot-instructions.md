## dokployaction

> **Always prefix shell/terminal commands with `rtk`** (located at `C:\rtk\rtk.exe`, also on PATH).

# RTK (Rust Token Killer) - Token Compression

**Always prefix shell/terminal commands with `rtk`** (located at `C:\rtk\rtk.exe`, also on PATH).

RTK is a transparent command wrapper that compresses verbose CLI output to save tokens. If RTK has a dedicated filter for a command, it compresses the output (60-99% savings). If not, it passes through unchanged. **RTK is always safe to use.**

## Usage Rules

1. **Every command gets `rtk` prefix**, including chained commands:
   ```bash
   # Correct
   rtk git status && rtk git add . && rtk git commit -m "msg"
   rtk cargo build && rtk cargo test
   rtk pnpm install && rtk pnpm run build

   # Wrong - never omit rtk
   git status && git add . && git commit -m "msg"
   ```

2. **Key commands and savings:**
   - Build: `rtk cargo build`, `rtk tsc`, `rtk next build`, `rtk lint` (70-90%)
   - Test: `rtk cargo test`, `rtk vitest run`, `rtk playwright test` (90-99%)
   - Git: `rtk git status/log/diff/add/commit/push/pull` (59-80%)
   - GitHub: `rtk gh pr view/checks`, `rtk gh run list` (26-87%)
   - JS/TS: `rtk pnpm install/list/outdated`, `rtk npm run`, `rtk npx` (70-90%)
   - Files: `rtk ls`, `rtk grep`, `rtk find` (60-75%)
   - Infra: `rtk docker ps/logs`, `rtk kubectl get/logs` (85%)
   - Debug: `rtk err <cmd>`, `rtk log <file>`, `rtk summary <cmd>`

3. **All git subcommands are supported** - even those not listed above pass through RTK.

---
> Source: [VidhyaSanjeevi/dokployaction](https://github.com/VidhyaSanjeevi/dokployaction) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
