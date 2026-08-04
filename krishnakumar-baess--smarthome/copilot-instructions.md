## smarthome

> This file is the **single source of truth** for any AI coding assistant working in

# AGENTS.md — AI Assistant Guide

This file is the **single source of truth** for any AI coding assistant working in
this repository. Read it before making any changes.

## Repo purpose

`SmartHome` is an umbrella monorepo for home automation and home documentation.
Subsystems are self-contained top-level directories. Shared meta lives at the root.

## Subsystem map

| Directory    | Status      | Notes                                                |
|--------------|-------------|------------------------------------------------------|
| `security/`  | Placeholder | KumarSec polished stack — https://github.com/BAESolutions/KumarSec |
| `home-docs/` | Placeholder | Home documentation system, design pending            |
| `platforms/` | Reserved    | One subdirectory per future automation platform      |

## Conventions

1. **Each subsystem is self-contained.** It owns its own `apps/`, `services/`,
   `infra/`, and `docs/` as needed. Do not reach across subsystem boundaries.
2. **Secrets are never committed.** `.env` files are gitignored. Commit
   `.env.example` templates instead. See [`docs/conventions.md`](docs/conventions.md).
3. **Follow existing patterns.** Match the naming, structure, and tooling already
   present in the subsystem you are modifying. Introduce new tooling only when
   there is a clear reason.
4. **Directory names are kebab-case.** File names in documentation are kebab-case.
   Code follows the conventions of the language/framework in use.

## Changelog update protocol

The repo keeps a single root [`CHANGELOG.md`](CHANGELOG.md) in
[Keep a Changelog 1.1.0](https://keepachangelog.com/en/1.1.0/) format.

- All notable changes go under the **`[Unreleased]`** section, grouped by
  `Added` / `Changed` / `Deprecated` / `Removed` / `Fixed` / `Security`.
- **When to update:** any change that adds or removes a subsystem, changes
  structure or conventions, or is otherwise user-visible. Routine doc typo fixes
  do not require an entry.
- Each entry is one line, imperative mood, and names the subsystem it touches
  when applicable (e.g. `Added security/ placeholder reserving the slot for the
  polished KumarSec stack`).
- Versioned releases (cutting `[Unreleased]` to a dated version) are manual and
  intentional. Assistants append to `[Unreleased]`; they do not cut releases
  unless explicitly asked.

## KumarSec reference

The `security/` subsystem will eventually host a polished version of KumarSec:
<https://github.com/BAESolutions/KumarSec>. The stack includes RTSP cameras,
MediaMTX media server, a YOLO-based AI worker, a FastAPI REST API, a Next.js web
front-end, a Kotlin Android app, and a backup service. No code lives here yet —
migration is deliberate and pending.

---
> Source: [KrishnaKumar-BAESS/SmartHome](https://github.com/KrishnaKumar-BAESS/SmartHome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-02 -->
