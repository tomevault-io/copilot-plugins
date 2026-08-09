## mindflow

> Whenever introducing a new string literal union value (e.g., adding a new status state or category) to a React component or utility, you MUST search for and update the corresponding central TypeScript type definitions (e.g., in `src/types/index.ts` or `src/types.ts`) before committing. Never rely solely on inline component updates.

# AI Assistant Global Rules

## TypeScript Type Synchronization
Whenever introducing a new string literal union value (e.g., adding a new status state or category) to a React component or utility, you MUST search for and update the corresponding central TypeScript type definitions (e.g., in `src/types/index.ts` or `src/types.ts`) before committing. Never rely solely on inline component updates.

## Draggable Containers vs. Inputs
When applying `draggable={true}` to a parent container (like a Card) for HTML5 drag-and-drop, you must attach `draggable={true}` and `onDragStart={(e) => { e.preventDefault(); e.stopPropagation(); }}` to any child `<input>` or `<textarea>` elements. This is a strict requirement to allow users to highlight and select text inside inputs without accidentally triggering the parent's drag event.

---
> Source: [ThunderD9/mindflow](https://github.com/ThunderD9/mindflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
