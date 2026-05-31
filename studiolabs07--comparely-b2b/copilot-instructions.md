## core

> ✅ Ideal LOC (Lines of Code) Guidelines for React Files

✅ Ideal LOC (Lines of Code) Guidelines for React Files
File Type	Recommended LOC	Notes
Component files (UI)	100–200 lines	Keep logic focused and UI clean. Split into smaller components if it grows.
Hooks (custom)	50–150 lines	Logic-focused, should do one thing well. Split into multiple hooks if too long.
Redux slice / state mgmt	100–300 lines	Includes actions, reducers, and types. Keep it modular.
Utility/helper functions	< 100 lines	Focused functions only; split per domain/use.
Page-level files (Next.js)	150–300 lines	These can be a bit larger as they compose many components but should not hold heavy logic.

🧠 Good Practices to Keep Files Maintainable
Single Responsibility Principle (SRP):

Each file/component should do one thing well.

Break down complex components:

Extract parts into smaller components or submodules.

Use folders, not fat files:

Organize components into folders (Header/, Sidebar/) with index.tsx, styles.ts, etc.

Separate concerns:

Logic: in hooks (useUser.ts)

UI: in components

Data/API: in services

Avoid giant index.tsx or App.tsx files:

Keep them minimal—just layout and routing.

🔥 Red Flags
Files with 500+ lines (unless it's a config or mock data).

Too many useEffect, useState, or props in a single component.

Deeply nested JSX (over 3–4 levels).

Mixing data-fetching, UI, and logic in one place.

📌 Rule of Thumb
“If you scroll more than twice in a file and still don’t see the bottom, it's probably time to refactor.”

---
> Source: [studiolabs07/comparely-b2b](https://github.com/studiolabs07/comparely-b2b) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
