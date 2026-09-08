## eventyay-interpretation

> > **Primary context file for coding agents.** Read this entire file before making changes.

# Eventyay Interpretation — Agent Guide

> **Primary context file for coding agents.** Read this entire file before making changes.
> All implementation details are derived from the live codebase. Constraints below are non-negotiable.

---

## Quick-Start Context

Read these files in order before any task:

1. **This file** — guardrails, invariants, ownership
2. [`.agents/context/REPOSITORY_CONTEXT.md`](.agents/context/REPOSITORY_CONTEXT.md) — stack, auth, Eventyay core plugin architecture
3. [`.agents/context/CHANGE_IMPACT_MAP.md`](.agents/context/CHANGE_IMPACT_MAP.md) — which files to touch for your specific task
4. [`.agents/context/ROUTE_MAP.md`](.agents/context/ROUTE_MAP.md) — Django URLs, OAuth endpoints, Webhook receivers
5. [`.agents/context/DATABASE_MAP.md`](.agents/context/DATABASE_MAP.md) — Django ORM models (VoxbentoOAuthGrant, RoomInterpretation)

### Skills (use for specialised tasks)

| Task | Skill |
|---|---|
| Navigate codebase | [`.agents/skills/repo-navigation/SKILL.md`](.agents/skills/repo-navigation/SKILL.md) |
| Architecture review | [`.agents/skills/architecture-review/SKILL.md`](.agents/skills/architecture-review/SKILL.md) |
| PR review | [`.agents/skills/pr-review/SKILL.md`](.agents/skills/pr-review/SKILL.md) |
| Security audit | [`.agents/skills/security-audit/SKILL.md`](.agents/skills/security-audit/SKILL.md) |

---

## Product Intent

`eventyay-interpretation` is a **Django plugin for Eventyay** (formerly Pretix). 
It bridges Eventyay's video rooms with the **VoxBento** live interpretation platform.

- Organizers connect their Eventyay event to VoxBento via OAuth 2.0.
- Webhooks keep VoxBento's state synchronized with Eventyay.
- Attendees in Eventyay's video player can select interpretation languages via VoxBento's WHEP streams.

**Stack:** Python 3.12+, Django 6.1+, Celery (for async tasks), pytest-django.
**Plugin Architecture:** It hooks into Eventyay via `pretix.plugin` entry points.

---

## Module Ownership

| File / Directory | Owns |
|---|---|
| `interpretation/signals.py` | Eventyay plugin hooks (nav menus, page injection) |
| `interpretation/views.py` | Dashboard views and forms for organizers |
| `interpretation/views_oauth.py` | OAuth 2.0 PKCE flow (authorize & callback) |
| `interpretation/views_webhooks.py` | Secure webhook receiver (`X-VoxBento-Signature` verification) |
| `interpretation/models.py` | Database models (`VoxbentoOAuthGrant`, `RoomInterpretation`) |
| `interpretation/backends/voxbento_api.py` | API client interacting with VoxBento |
| `interpretation/tasks.py` | Celery background tasks (e.g. `sync_voxbento_connection`) |
| `tests/` | pytest-django test suite |

---

## Core Invariants (Non-Negotiable)

1. **Security First:** Webhooks must verify `X-VoxBento-Signature` using `hmac.compare_digest`. OAuth state must prevent CSRF. Tokens must be stored securely (using `EncryptedTextField`).
2. **Plugin Sandbox:** This is a plugin. It must not crash the core Eventyay application. If VoxBento API is down, tasks must gracefully degrade or retry via Celery.
3. **No Direct Eventyay Core Modifications:** We integrate via hooks and signals. Do not attempt to modify `eventyay-host` core directly unless explicitly coordinating a Phase 4 frontend change.
4. **`uv.lock` is the dependency source of truth.** Never modify without running `uv lock --upgrade` and confirming tests pass.
5. **Code Style:** `ruff` is strictly enforced with a `120` line length limit. Use `ruff format .` before committing.

---

## Validation Before Submitting Changes

```bash
uvx ruff check . --fix
uvx ruff format .
pytest tests/ -v
```

---
> Source: [fossasia/eventyay-interpretation](https://github.com/fossasia/eventyay-interpretation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
