## golemcore-bot

> This module contains shared contracts for GolemCore Bot.

# CLAUDE.md — GolemCore Bot Contracts Module

This module contains shared contracts for GolemCore Bot.

## What belongs here

- port interfaces
- shared domain models used across modules
- contract-style component interfaces
- lightweight shared view models
- other stable boundary types needed by multiple modules

## Rules

1. New shared contracts must go into this module.
2. Do not put runtime-heavy Spring services, adapters, controllers, or tool implementations here.
3. Keep this module lightweight and dependency-minimal.
4. Prefer stable, reusable boundary types over feature-specific implementation details.

---
> Source: [alexk-dev/golemcore-bot](https://github.com/alexk-dev/golemcore-bot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
