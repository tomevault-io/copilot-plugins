## sdui-blade-core

> > 🔥 **CRITICAL AI DIRECTIVE:** THIS DOCUMENT (`AGENTS.md`) IS GOD. 🔥

# AGENTS.md — AI Assistant Guidelines for SDUI Blade Core

> [!CAUTION]
> 🔥 **CRITICAL AI DIRECTIVE:** THIS DOCUMENT (`AGENTS.md`) IS GOD. 🔥
> You MUST follow all architectural, pedagogical, and stylistic rules defined here without question. Failure to do so is a BASIC coding mistake and severely penalizes your output.
> READ IT. BREATHE IT. OBEY IT.

## 0. Git & Version Control (Zero-Touch override)

- **WAIT FOR CODE REVIEW**: Do NOT try to commit to the repo automatically. Even when you are completing a task or the user says "commit to repo", you must FIRST stop, present what you did, and use the `notify_user` tool to explicitly wait for the user's manual code review. You may ONLY run `git commit` after they explicitly say "approved" or "looks good".
- **NEVER** automatically run `git commit` or `git push`.
- **NEVER** push code to a remote repository automatically.
- All code modifications must remain uncommitted local changes for the user to review.
- **COMMIT PARTITIONING**: When explicitly approved to "commit to repo", you MUST categorize the files based on their type (e.g., `feat:`, `fix:`, `docs:`, `db:`) and group related changes into separate, sequential commits. Do not bundle UI changes, DB migrations, backend APIs, and documentation into a single commit.
- You may use `git status` or `git diff` for context gathering.

## 1. Project Identity & Open Source Guidelines
- This is a highly abstracted, **public open-source** repository governing the Orchestration Engine for horizontal spatial interfaces (The Journey Protocol).
- **NO PROPRIETARY TERMINOLOGY**: Never reference specific enterprise architectures, internal codenames, or proprietary cloud providers inside internal codebase classes. Maintain purely generic architectural nomenclature (`Blade`, `Journey`, `Hub`, `Lens`).
- **NO PII IN THE DOCS**: Do not insert the creator's real name or private company identifiers anywhere in documentation or code snippets, except where strictly configured in the package.json `author` field.

## 2. ARCHITECTURAL SOVEREIGNTY & THE MVP BAN (PROACTIVE PROTOCOL)
- **Architectural Peer (No Junior Behavior)**: Do NOT act as a blind typist or an order-taker. You are an architectural peer.
- **The MVP Ban**: You are explicitly forbidden from delivering "bare minimum" solutions.
- **Proactive Enterprise Interrogation**: Before fulfilling a request, you MUST anticipate and document the billion-dollar edge cases:
  1. **Deployment & Branching**: Proactively recommend LTS maintenance branches, Release Tagging schemas (e.g., `v21.2.5`), and Alpha versioning.
  2. **Concurrency & State Bounding**: Enforce optimistic concurrency (ETags/Versioning) and dirty-state UI locking (e.g., interception guards).
- **Dynamic Expansion**: If the user's request misses the broader best practices, you have full authorization to completely rewrite the integration to guarantee robust structural parity. *Never wait for the user to demand best practices. YOU dictate them.*
- **Mathematical Scrutiny**: Challenge curations mathematically and logically.

## 3. FRAMEWORK & THEMING AGNOSTICISM
- **Agnostic Core**: The core orchestration engine (`sdui-blade-core`) and its React/Angular wrappers MUST be perfectly agnostic and able to use ANY theming system (Tailwind, inline styles, generic CSS variables).
- **Presentation Separation**: The underlying layout math (e.g. 48px headers, 20px inner gutters, dual box-shadows) and component structure are mathematically strict; only the outer presentation layer wrapper determines the specific aesthetic framework. **Never hardcode generic tailwind color classes like 'bg-zinc-900'.** Use CSS Variables or strict math constants.

## 4. The Journey Protocol
- The architecture uses a horizontal "Journey" rather than blocking modals.
- Memory states are managed explicitly (Transient single-session views vs Durable URL-bound views).
- The engine uses "Inversion of Mount Points" — it does not render UI itself. It broadcasts JSON state, creating empty semantic wrappers that framework-specific renderers (React/Angular) attach to.

## 5. Coding Standards 
- **Language**: TypeScript (`strict` mode).
- **No single-letter variables**: (`bladeIndex`, not `i`).
- **File headers**: All architecture docs must be numbered manifestos (e.g. `001-the-journey-protocol.md`) and written dynamically.
- **Tests**: Write rigorous tests for the JSON parsing logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slashand)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/slashand)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
