## cumulus

> - Workspace packages are consumed as **TypeScript source** (`main`/`exports` point at `./src/index.ts`, no build step), so importing `@cumulus/shared-types` typechecks without ever running `pnpm build`.

# Cumulus — notes for Claude

- Workspace packages are consumed as **TypeScript source** (`main`/`exports` point at `./src/index.ts`, no build step), so importing `@cumulus/shared-types` typechecks without ever running `pnpm build`.
- `pnpm lint` fails on `main` for reasons unrelated to your change: `apps/dashboard`'s `next lint` is deprecated and prompts interactively for an ESLint config, so it exits 1 in any non-TTY. Gate on `pnpm typecheck` + `pnpm test` instead.
- `SshDriver.runRemote` refuses any single command matching `ufw … deny … (22|ssh)`, so remote firewall work must be issued **one command per call** — chaining `ufw default deny incoming && ufw allow 22/tcp` into one string trips the lockout guard even though the sequence is safe.
- The bootstrap dry-run test asserts no attempted command matches `/apt|docker run|tailscale up|reboot|systemctl (start|enable)/`, so a new **read-only** probe in `apps/cli/src` must avoid those substrings *anywhere* in the string — including inside an `echo "key=…"` field name (a reachability key called `apt` fails the suite).

---
> Source: [wirkling/cumulus](https://github.com/wirkling/cumulus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
