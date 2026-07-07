## rusty-shed

> - **Framework**: Tauri 2.0 + Svelte 5 (**Runes only**, strictly no legacy stores).

# 🧬 Project

- **Framework**: Tauri 2.0 + Svelte 5 (**Runes only**, strictly no legacy stores).
- **UI Architecture**: `shadcn-svelte` (bits-ui) + Tailwind CSS.
  - **Rule**: All components must use `$props()` and `$state()`. Refactor any `export let` or `$:` from shadcn CLI imports immediately.
- **Backend**: Clean Architecture in `src-tauri/`.
  - **Layers**: Domain (Logic/Traits), Application (Services), Infrastructure (Adapters/Tauri Commands).
- **Localization (i18n)**:
  - **No Hardcoded Strings**: All user-facing text must use **Paraglide-JS**.
  - **Dual-Language**: Every new key **must** be defined in both English (`en`) and Italian (`it`) before a task is considered complete.
- **Error Handling**: No `unwrap()` in Rust. Use `Result` mapping with custom serializable error enums for the frontend.

# 🛑 Hard Constraints

- **Package Manager**: Always use `pnpm`. Never use `npm` or `yarn`.
- **Type Safety**:
  - Use `tauri-specta` bindings; **never** redefine types manually on the frontend.
  - **Strict TypeScript**: No `any`. Use `unknown` or specific interfaces. Use `as const` for literals.
- **Architecture**: Consult `docs/adr/` before changes; adhere to established decisions.
- **Testing Integrity**: Never skip, disable, or delete a failing test. Fix the logic, not the check.
- **Dependencies**: Never update or add packages without explicit user confirmation.

# 🔄 Workflow Protocol

A task is **Complete** only when this sequence passes with **zero** errors/warnings:

1. **Plan**: Describe logic and architectural impact. Identify the layer affected.
2. **Implement**:
   - **Backend**: Logic in Domain/Application before Infrastructure.
   - **Frontend**: Modular structure. Ensure shadcn components are Runes-compliant.
   - **i18n**: Add keys to messages/en.json and messages/it.json.
3. **Sync Bindings**: If Rust types or commands change, run `pnpm specta:generate` and verify the `.ts` binding updates.
4. **Format & Lint**: Run project formatters. **Clippy warnings are errors** (`cargo clippy -- -D warnings`).
5. **Verify**: Run `pnpm svelte-check`, Vitest, and Cargo tests.

# 📝 Commit Convention

- Use **Conventional Commits** (e.g., `feat:`, `fix:`, `refactor:`, `chore:`).
- Only commit once the full **Workflow Protocol** has been satisfied.

---
> Source: [CarloMicieli/rusty-shed](https://github.com/CarloMicieli/rusty-shed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
