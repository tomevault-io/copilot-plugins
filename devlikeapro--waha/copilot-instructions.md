## waha

> Here’s a complete `AGENTS.md` draft for your i18n workflow 👇

Here’s a complete `AGENTS.md` draft for your i18n workflow 👇

---

# 🌐 AGENTS.md — Locale Maintenance Guide

This document defines how the **i18n Agent** manages, updates, and synchronizes
locale files (e.g. `en-US.yaml`, `pt-BR.yaml`, etc.) across all supported
languages.

---

## 🧩 Overview

Each locale file (e.g. `en-US.yaml`) contains translation keys and values used
throughout the system. The **English (en-US)** file is the **source of truth** —
every other language must mirror its structure, keys, emojis, and formatting.

---

## 🪄 Core Commands

### 1. `update all i18n`

**Purpose:** Ensures all locales are synchronized with the current `en-US.yaml`.

**Steps:**

1. **Start with `en-US.yaml`.**

   - Scan all code and templates to verify each key is used.
   - Ignore usage warnings for `datetime.*`, flat keys without a dot, and any
     `locale.*` entries — they are considered always in use.
   - Propagate every key change (add, edit, remove) from `en-US.yaml` to all
     other locale files with the corresponding translations so structures stay
     aligned.
   - Preserve `datetime.locales` in every non-English locale; it pins the
     language hint for formatting and must never be dropped.
   - If any keys appear **unused**, list them and **ask whether to remove or
     keep**.

2. **Check for “ChatWoot” mentions.**

   - Any occurrence must be reviewed — the system should not reference
     _ChatWoot_ directly.
   - Ask:

     > “Found ‘ChatWoot’ in key `X.Y.Z`. Should we reword this? If yes, how?”

3. **Validate formatting and structure.**

   - Ensure all keys use consistent YAML indentation and ordering.
   - Emojis, punctuation, and tags (`{{variable}}`, `**bold**`, etc.) must stay
     exactly as in `en-US`.

4. **Synchronize all target languages.**

   - For each locale (`es-ES.yaml`, `fr-FR.yaml`, etc.):

     - Copy the structure and keys from `en-US.yaml`.
     - Translate the values from English → target language.
     - **Ignore any manual changes in target files** — always use the English
       source.
     - Preserve all markdown, emojis, punctuation, and placeholders.

5. **Report summary.**

   - Added / updated / removed keys count per language.

---

### 2. `update key.* for i18n`

**Purpose:** Update or add translations for specific keys only.

**Steps:**

1. Match all keys provided in the command (supports wildcard like `key.auth.*`).
2. Update those keys in `en-US.yaml` first.
3. Propagate and translate the updated keys to all other locale files.
4. Do **not** touch unrelated keys or structure.

---

## ⚙️ Translation Rules

| Aspect         | Rule                                                                              |
| -------------- | --------------------------------------------------------------------------------- |
| **Structure**  | Must always follow `en-US.yaml`.                                                  |
| **Formatting** | Keep emojis, punctuation, capitalization, and markdown exactly as in English.     |
| **Tags**       | Preserve placeholders (`{{variable}}`) and code blocks.                           |
| **Priority**   | English (`en-US.yaml`) is always the canonical version.                           |
| **Comments**   | If a translation key needs explanation, add a YAML comment above it (not inline). |

---

## 🚫 ChatWoot Mentions

- The word **ChatWoot** must never appear in translations.
- When found, the agent must pause and ask:

  > “ChatWoot found in key `X.Y.Z`: `{{string}}`. Should we rephrase it (e.g.,
  > ‘Inbox’, ‘Messaging Platform’, or something else)?”

---

## 📘 Terms (reserved)

> This section will define key terminology and translation conventions (e.g.,
> “Contact” vs “Customer”, “Session” vs “Chat”), once finalized.

---

## ⚠️ Exceptions

- Treat `datetime.*` keys as standard infrastructure strings; do not flag them
  as unused.
- Always keep `datetime.locales` aligned with the target locale code (e.g.,
  `es-ES`); if it is missing, restore it during the sync.
- Keys without a dot (e.g., `UNKNOWN`) and any under `locale.*` are canonical
  metadata — skip unused checks for them.

---

## 🧾 Example Workflow

```bash
# Full synchronization
> update all i18n
→ Checking unused keys...
→ Found 3 unused keys: session.old, inbox.archive, legacy.test
Keep or remove? [k/r]
→ Found "ChatWoot" in en-US.yaml: "ChatWoot inbox connected"
Replace? [yes/no]

→ Translating en-US → es-ES, fr-FR, pt-BR...
✅ All i18n files updated and aligned with en-US.yaml
```

---

Would you like me to add an example **YAML structure** and show how the agent
should align keys (e.g., diff between en-US and pt-BR)? It helps clarify exactly
how updates propagate.

---
> Source: [devlikeapro/waha](https://github.com/devlikeapro/waha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-21 -->
