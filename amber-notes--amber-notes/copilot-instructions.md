## amber-notes

> Rules for coding agents (and people) changing this repo. The README explains the architecture and how to build and test.

# Working on Amber Notes

Rules for coding agents (and people) changing this repo. The README explains the architecture and how to build and test.

- **Apple Notes is the behaviour reference.** Copy what Notes does for editing, lists, checklists, tables, selection and navigation. Follow Apple's Human Interface Guidelines for layout, type, colour, controls and accessibility, in light and dark mode.
- **Every fix gets a regression test.** Editor and UI behaviour goes in the offscreen harness under `PaneTests/Harness` and `PaneTests/Interaction`; run `scripts/qa-test.sh`. Server changes get Deno tests; end-to-end tests run against the local stack only (`supabase start`, then `scripts/mcp-e2e.sh`).
- **Never post global input events** (synthetic clicks or keystrokes) on a developer's Mac, and never leave windows or Dock icons behind. Kill only processes you started, by exact PID.
- **No secrets, ever.** Don't commit or print keys, tokens or passwords. They live in `.env`, `.secrets/`, `Config/Backend.local.xcconfig` and GitHub Secrets.
- **Never test against production data.** Load, abuse and end-to-end tests run on the local stack.
- **Migrations are additive** with a timestamp newer than every existing one. Never set `[auth.email] enable_signup = false` in `supabase/config.toml`: it turns off email sign-in entirely.
- **Installing on a developer's devices:** `scripts/install-mac.sh` (team-signed; never copy an ad-hoc build over the installed app) and `scripts/install-phone.sh`.
- **Releases** are tag-based and run in GitHub Actions; see `docs/RELEASING.md`. Never submit an app for App Store review; that stays with the maintainer.
- Write plainly in UI copy and docs: sentence case, short sentences, no hype.

---
> Source: [amber-notes/amber-notes](https://github.com/amber-notes/amber-notes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
