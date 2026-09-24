## vex-sensory

> *Rules for Gemini-CLI when working on Vex Sensory*

# GEMINI.md - Antigravity Guidelines

*Rules for Gemini-CLI when working on Vex Sensory*

---

## Process (Get Approval At Each Step)

1. **Read Spec** → Read the feature spec file
2. **Clarify** → Ask questions if unclear (WAIT)
3. **Plan** → Propose implementation plan (WAIT for approval)
4. **Implement** → Write code
5. **Build & Test** → Run: `cargo build`, `cargo test`, `cargo fmt`, `cargo clippy`
6. **Code Review** → Show files changed, wait for review approval
7. **Commit** → Only commit AFTER approval
8. **Report** → Send Discord embed summary

**NEVER skip steps. NEVER commit without approval.**

---

## Coding Standards

- Follow Rust idioms
- Use proper error handling (`Result`, `?`)
- Write unit tests for core functions
- Document public APIs
- Run `cargo fmt` before committing
- Fix clippy warnings

---

## Security

- Never hardcode secrets (API keys, webhooks)
- Use environment variables
- Add secrets to `.gitignore`

---

## Reporting

After each feature:

1. Create Discord webhook
2. Store URL in `DISCORD_WEBHOOK_URL` env var
3. Send embed report with curl

---

## Project Location

Specs: `vex-sensory/`

---

*Last updated: 2026-02-26*

---
> Source: [levia17/vex-sensory](https://github.com/levia17/vex-sensory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
