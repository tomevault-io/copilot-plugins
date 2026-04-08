## agent-cms

> - This repository acts as a content management system (CMS) for agents.

# Gemini Agents CMS

- This repository acts as a content management system (CMS) for agents.

## Directory Overview

- `.cms/doctypes/*`: Definitions for the different types of documents.
- `.cms/scripts/create`: Must be used when creating documents.
- `.cms/scripts/find`: Must be used instead of `ls` to locate documents.
- `.cms/scripts/move`: Must be used instead of `mv` when moving documents.
- `.cms/scripts/delete`: Must be used when deleting documents.

---

# CMS Procedures

- These procedures allow you to capture ideas into a _well-structured document schema_.
- _Doctypes_ define the _classes_ of possible documents.
- _Documents_ are _instances_ of doctypes.

## CMS Workflow

0. Start by reading all the doctypes.
1. You will chat with the user about a topic.
2. The user will ask you to capture specific ideas.
3. You will find the appropriate _doctype_ to capture that idea (e.g.: task, note, etc).
4. You will manage document files using the guidance in the _doctype_.
5. Repeat...

## Mandates

- _NEVER_ trust your memory. The filesystem may have changed.
- The `find` script is the only way to know if a document exists.

## Decision Loop

Follow this list of instructions When creating or updating documents:

1. Use the `find` script to determine if a similar task already exists.
   1. _ALWAYS_ trust the results of `find`. The filesystem is constantly changing, so you must not trust your memory.
1. Identify the relevant doctype(s).
   1. If you can't infer a doctype, ask the user for clarification.
1. Use a script to determine the filename for the document:
   1. When creating, use the `create` script.
   2. When updating, use the `find` script.
1. Read the doctype for the document.
1. **Apply changes to the document according to the instrctions in the doctype.**
1. Report actions taken and explicitly note any assumptions or guesses.
1. STOP. Perform no other document management/verification steps.

## Doctypes

- Doctypes are like OOP classes for documents.
- Doctypes use plain text to describe the format, content, and actions that can be performed on a document.
- _DO NOT_ make up doctypes.

## Documents

- Documents are instances of a doctype.
- _ALWAYS_ refer to a document's doctype instructions before taking any CRUD action.
- _NEVER_ invent metdata for file (e.g.: lastModified, ID, etc).

## Naming Conventions

- _ALWAYS_ use _short_ names unless given a _specific_ filename to use (e.g.: `three-or-four-words.md`).
- _ALWAYS_ use kebab-case unless asked to change _this format_ specifically.
- _ALWAYS_ use include the current working directory `./` for relative filenames (e.g.: `./note/archived/ideas.md`).

## Scripts

- Scripts are shell commands that perform all deterministic work that needs to be predictable.
- Located at `.cms/scripts`.
- _ALWAYS_ delegate to scripts when possible.
- The agent must not re-implement script behavior.

### `create`

- **Usage**: `create <document-filename> --type=<doctype> --status=<status>`
- **Purpose**: Creates a new, zero-byte file in the correct folder based on the provided type.
- **Output**: The relative path to the new file.

### `find`

- **Usage:** `find <document-type> [--status=<status>]`
- **Purpose:** Finds documents in the filesystem based on type, and optional status.
- **Output:** A list of documents in CSV format: `document-type, status, relative-document-filename`
- Use this instead of `ls` to locate files.
- _ALWAYS_ consider the entire CSV when making a desicion.
- _NEVER_ `grep` the output.
- The `find` script is _definitive_. If it returns nothing, then nothing exists.

#### `move`

- **Usage:** `move <relative-document-filename> --type=<document-type> --status=<new-status>`
- **Purpose:** Changes a document's status by moving it to a new folder.
- **Output:** The relative path to the moved file.
- Use this instead of `mv`.

### `delete`

- **Usage**: `delete <relative-document-filename>`
- **Purpose**: Soft-deletes a document to a `_trash_` folder.
- **Output**: The relative path to the soft-deleted file.
- Use this instead of `rm`.

### System Commands

- You may use shell-specific built-in commands for updating file contents.
- _ALWAYS_ use a path provided by `create` or `find`.

### Error Handling

- If you find errors, let the user know and **stop**.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/waiteb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/waiteb)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
