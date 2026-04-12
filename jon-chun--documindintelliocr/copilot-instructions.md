## documindintelliocr

> **Context**: This is a Next.js project.

# Environment Rule: Always Use pnpm

**Context**: This is a Next.js project.

**Rule**: For all package management operations (installing, adding, removing, updating dependencies), you **MUST** use `pnpm`. Do not use `npm` or `yarn`.

**Example Commands**:
*   To install dependencies: `pnpm install`
*   To add a new package: `pnpm add [package-name]`
*   To add a dev dependency: `pnpm add -D [package-name]`
*   To remove a package: `pnpm remove [package-name]`
*   To run a script: `pnpm [script-name]`

**Rationale**: This project is configured to use `pnpm`, and consistent use of the correct package manager prevents dependency conflicts and ensures reliable builds.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jon-chun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
