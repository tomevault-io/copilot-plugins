## mcpwebbridge

> Scope: Operating rules and guardrails for the project, copied verbatim where possible from the redesign plan. These govern architecture, permissions, DOM handling, and development practices.

# RULES

Scope: Operating rules and guardrails for the project, copied verbatim where possible from the redesign plan. These govern architecture, permissions, DOM handling, and development practices.
Last updated: 2025-09-03 (UTC)

## System Rules & Project Guidelines

- Decide Late for Unproven Choices: Defer irreversible decisions until we have more data. Use feature flags or toggles for experimental features and have a rollback plan. Start behind a flag or in dev mode until proven.
- Lock Down Core Structure Early: The content/background split and message schema are fundamental – solidify these now (with TypeScript types or exhaustive documentation) so they don’t keep changing. Define Message types centrally and require justification and testing for any changes to this core.
- Consistent Messaging Contract: All message types and data structures must be clearly documented and remain consistent. Avoid vague payloads. Each handler returns a well-structured success response or a detailed error. No silent failures – log and propagate errors for visibility.
- Minimal & Explicit Permissions: Principle of least privilege. Only request needed permissions (storage and limited host permissions). Avoid any user data telemetry or external calls from the extension itself. All network calls are user-initiated and to MCP servers the user configures.
- Robust DOM Handling:
  - Use stable selectors whenever possible (prefer data-* attributes or roles over brittle classnames).
  - Implement a MutationObserver to detect when critical DOM elements change and re-inject or reposition the UI accordingly. Handle window resize events to adjust positioning.
  - Use safe fallback positions if dynamic anchoring fails so the MCP control remains visible.
  - No interference with the page: Shadow DOM ensures styles don’t leak. Avoid injecting globals that conflict with the host page.
- Cursor/AI Development Practices:
  - Provide the latest context (this blueprint and updated design docs) to prevent reintroducing solved issues. Clearly state current state and next goal.
  - Iterative development: implement and verify in small steps (storage, messaging, UI, integration).
  - Ask for alternatives & justify for tricky decisions; weigh pros/cons.
  - Verify environment assumptions; use empirical debugging with DevTools if behavior diverges from expectations.
  - Keep tone focused and professional; prefer clear, directive prompts.
- Prevent Regression of Known Issues: Document known problems and instruct not to repeat them. Enforce central message schema, stable positioning (avoid overlap with chat input), and design rules to prevent brittle behavior.

## Canonical terms (must be used consistently)

- Content Script = UI only
- Background = logic only
- Shadow DOM
- typed message schema
- storage: sync vs local
- MutationObserver with fallback
- minimal permissions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jasonshaw0) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
