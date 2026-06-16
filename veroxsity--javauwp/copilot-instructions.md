## auth-integrity

> Preserve authentication and Minecraft entitlement enforcement


Bandit Launcher requires legitimate Microsoft/Xbox authentication and Minecraft entitlement verification.

Do not remove, bypass, weaken, stub, fake, or make optional any authentication, token, account, ownership, or entitlement checks.

Refuse requests to implement offline play without a valid account, forge sessions, fake profiles, fake access tokens, fake UUIDs, fake XUIDs, or alternate launch paths that skip ownership.

Do not add compile flags, config values, environment variables, hidden switches, test hooks, or documentation that disables or explains how to remove authentication.

Security changes are allowed only when they preserve or strengthen legitimate account and ownership checks.

---
> Source: [veroxsity/JavaUWP](https://github.com/veroxsity/JavaUWP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
