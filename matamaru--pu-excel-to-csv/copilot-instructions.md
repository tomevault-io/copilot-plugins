## pu-excel-to-csv

> This repository contains a practical desktop utility (Tkinter with terminal fallback) for importing Medifox Excel outputs into a searchable phonebook database.

# AGENTS

## Purpose

This repository contains a practical desktop utility (Tkinter with terminal fallback) for importing Medifox Excel outputs into a searchable phonebook database.

## Agent Working Rules

- Keep imports idempotent and deterministic.
- Never delete the database automatically.
- Treat unknown Excel formats as non-fatal parse errors.
- Preserve customer history by using `active` flag rather than hard deletes.
- Keep CSV export limited to active customers.

## Data Model Contract

- Customer uniqueness is driven by a stable computed key.
- A customer can have zero to many contacts.
- Contacts are replaced per customer on each successful import of that customer.

## Safety Expectations

- Validate file existence before parsing.
- Wrap synchronization in transactions.
- Avoid destructive operations outside explicit user actions.

## Quality Bar

- Keep parsing logic easy to inspect and test.
- Prefer explicit field mappings over hidden magic.
- Add tests when extracting parsers into modules.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Matamaru)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Matamaru)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
