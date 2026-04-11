## vanguard

> - App type: Tauri 2 desktop app.

# AGENTS.md

## Project Snapshot
- App type: Tauri 2 desktop app.
- Backend: Rust in `src-tauri/src`.
- Frontend: React/TypeScript in `src/`.
- Core domains: auth, vault sync, vault unlock/biometric.

## Frontend Package Manager
- Use `pnpm` for frontend/package-manager commands in this repository.
- Prefer `pnpm <command>` over `npm <command>` when installing dependencies, running scripts, or invoking frontend tooling.

## Frontend Component Guidelines
- Prefer shadcn components for UI implementation.
- Reuse existing components in `src/components/ui` before adding new ones.
- When installing a new component, use: `pnpm dlx shadcn@latest add <component-name>`.
- Do not introduce other UI component libraries unless explicitly required.
- Keep custom UI wrappers thin; compose from shadcn primitives first.

## AI Response Style
- Keep AI responses concise and to the point.

## Troubleshooting Guidelines
- Do not make blind code changes when issues are unclear; investigate and confirm the likely root cause first.
- If root cause cannot be confirmed, add minimal, temporary diagnostic logs (with sensitive data redacted) to narrow scope.
- Ask the user to help reproduce and collect the requested logs, then apply targeted fixes based on evidence.

## DDD Architecture Constraints
- Enforce dependency direction: `interfaces -> application -> domain`.
- `infrastructure` only implements application ports; do not place business rules there.
- `domain` must stay pure: no Tauri, HTTP, SQLite, or OS APIs.
- Tauri commands are interface adapters only:
  - validate/request mapping,
  - call application services/use-cases,
  - map errors/DTOs.
- Do not bypass application layer from interfaces to infrastructure.

## Security Baseline
- Never log secrets: `access_token`, `refresh_token`, `password`, `master_password`, key material.
- Keep redaction on all user-visible and loggable error messages.
- Do not persist plaintext sensitive data; use encrypted persistence/keychain only.
- Treat auth/session initialization as fail-closed on critical errors.
- Keep `allow_invalid_certs = false` by default; only allow in explicit local dev scenarios.
- Validate all external input (Tauri command args, remote payload assumptions).

## Quality Gate (Rust)
- Before merge, run in `src-tauri/`:
  - `cargo check`
  - `cargo test`
  - `cargo clippy --all-targets --all-features -- -D warnings`

## Commit Messages (Conventional Commits)
- Format: `<type>(<scope>): <subject>`
- Types: `feat`, `fix`, `refactor`, `perf`, `test`, `docs`, `chore`, `build`, `ci`.
- Subject rules:
  - imperative mood,
  - lowercase start,
  - concise, no trailing period.
- Recommended scope examples: `auth`, `sync`, `vault`, `desktop`, `infra`, `bootstrap`, `docs`.
- Examples:
  - `fix(sync): avoid debounce pollution when sync slot acquisition fails`
  - `refactor(vault): move unlock flow from tauri command to application use case`
  - `docs(agents): add ddd and security constraints for codex`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ryuyb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ryuyb)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
