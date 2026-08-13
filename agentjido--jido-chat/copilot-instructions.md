## jido-chat

> `jido_chat` owns the core adapter contract and typed chat data model for the

# AGENTS.md - Jido Chat Development Guide

`jido_chat` owns the core adapter contract and typed chat data model for the
Jido chat ecosystem.

## Commands

- `mix setup` - Fetch dependencies.
- `mix test` - Run the default test suite.
- `mix quality` - Run the Jido package quality gate.
- `mix coveralls` - Run coverage.
- `mix install_hooks` - Explicitly install local git hooks.

## Rules

- Module namespace for this repository is `Jido.Chat`.
- Public modules should be defined under `Jido.Chat.*`.
- Runtime process trees, bridge supervision, queues, retries, and delivery orchestration belong in `jido_messaging`.
- Prefer Zoi-backed structs and Splode errors for new core data and errors.
- Keep public APIs documented and typed.

## Release Hygiene

- Do not modify `CHANGELOG.md`; release notes are generated from Git history during release, so keep changes focused on proper Conventional Commits.

---
> Source: [agentjido/jido_chat](https://github.com/agentjido/jido_chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
