## core

> - Every time you do a discovery on something unexpected, something that you couldn't guess, save it in the MEMORY.md document. Don't just write a story of what happened, write decisive info that you could not find anywhere else.

- Every time you do a discovery on something unexpected, something that you couldn't guess, save it in the MEMORY.md document. Don't just write a story of what happened, write decisive info that you could not find anywhere else.
- Read from MEMORY.md when you encounter a unknow decision or are you in doubt of something.
- Run the full test suite only when you modify or add code in the main classes; if you find any errors fix them.
- Changes not related to the main classes (docs, comments, metadata, or other non-core edits) do not require retesting.
- Don't break any feature when you implement something new or you change something already there.
- Avoid adding new external dependencies.
- Maximize the use of the already present classes in the Core collection.
- Follow the existing code-style.
- Start files with `<?php` followed by a blank line.
- Never close `?>` php files unless they are used as templates.
- Do not add `declare(strict_types=1);` (not used in this codebase).
- Class files typically include a file-level docblock with: class name, short description, `@package core`, `@author`, and `@copyright`.
- Global classes are in the root namespace; subfolders (e.g. `classes/Email`, `classes/FileSystem`, `classes/Cache`) use a matching `namespace`.
- Multiple classes can live in a single file when they are tightly related.
- All public methods must have docblocks.
- Class names are `PascalCase`; acronyms remain uppercase (`HTTP`, `URL`, `CLI`).
- Methods are `camelCase`.
- Properties and internal variables use the existing local conventions; do not rename or normalize casing.
- Short array syntax `[]` is used for arrays.
- Braces follow K&R style (`class X {`, `function y() {`, `if (...) {`), with single-line `if` statements permitted when used in-file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coesion) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
