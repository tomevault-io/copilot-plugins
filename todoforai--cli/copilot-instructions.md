## cli

> When a command is added or changed, update ALL of these in the same change — agents only learn a

# tfa-cli

When a command is added or changed, update ALL of these in the same change — agents only learn a
capability from what they are shown:

1. `src/args.ts` — `printUsage()` line, `HELP_SUBCOMMANDS`, and any new flags in `parseCliArgs`.
2. The command's own `--help` text.
3. `README.md`.
4. `../packages/shared-fbe/src/tool_catalog.json` → `tfa-cli.description` (what agents see in their tool list),
   then `../sandbox-manager/scripts/sync-vendor.sh catalog` so `sandbox-manager/assets/tool_catalog.json` matches.

---
> Source: [todoforai/cli](https://github.com/todoforai/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
