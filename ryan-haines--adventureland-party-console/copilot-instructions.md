## adventureland-party-console

> - Right-click context menus, all their submenus, and embedded upgrade previews use opaque WHITE backgrounds with BLACK text and light-gray hover states. This is an explicit exception to the dark dashboard control guidance.

# Repository UI guidance

- Right-click context menus, all their submenus, and embedded upgrade previews use opaque WHITE backgrounds with BLACK text and light-gray hover states. This is an explicit exception to the dark dashboard control guidance.


- Keep dashboard controls on dark, opaque backgrounds with explicit high-contrast text and borders.
- Do not rely on the default outline-button colors; they can render as light gray on white in this dashboard.
- For outline, icon, cancel, and secondary-action buttons, set background, text, border, and hover colors explicitly and verify readability against the surrounding panel.

# Coordinator development

- Read [the coordinator README](runtime/coordinator/README.md) before changing coordinator behavior or build/deployment wiring. It maps domains and documents manual validation and activation.
- Edit maintained TypeScript under `runtime/coordinator/`; do not edit generated `.build/runtime/coordinator-*.cjs` bundles or the installed caracAL launcher. The launcher template lives under `tools/caracal/`.
- Building and activating are different operations. Follow the README's supported restart workflow; do not claim a build has reloaded the live coordinator without verifying it. Preserve current character assets when deploying coordinator-only changes.

# Game TypeScript types

- Prefer `typed-adventureland` types when describing the same game object. Use `Pick`, `Partial`, or indexed access for subsets instead of repeating compatible fields.
- Extend upstream types for additional fields. When field semantics differ, use `Omit` plus explicit replacements; document why broader IDs, optional fields, or normalized values are needed.
- Keep application protocols, jobs, and intentionally different external payloads local. Do not force partial wire data into complete game-object types.
- Document verified upstream gaps alongside compatibility extensions and review them when upgrading the package. Do not add casts solely to hide incompatible contracts.

# Testing

- Never write unit tests after you write code.
- Highly prefer E2E tests as the sole testing mechanism. Use them to verify complex features work. At the end of E2E tests, produce a verifiable and repeatable artifact.
- If you must test a system in isolation, first write down all the ways it could fail, then write the code.

Run `npm test` for real-game and console E2E and inspect `.build/e2e-report/` and `.build/e2e-results/`.
See [the testing guide](docs/testing.md) for setup, artifacts, fixture boundaries, and
the retained isolated regression exceptions. Do not replace observable behavior
checks with source-text, callback-order, or CSS-class assertions.

---
> Source: [Ryan-Haines/adventureland-party-console](https://github.com/Ryan-Haines/adventureland-party-console) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
