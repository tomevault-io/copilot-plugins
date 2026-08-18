## complex-platform

> - Preserve fail-closed behavior. Integration failures must remain visible failures.

# Project operating rules

- Preserve fail-closed behavior. Integration failures must remain visible failures.
- Do not fake Product Design, Browser, ImageGen, Codex, or AI-DLC success.
- Do not expand confirmed requirements without explicit human approval.
- Keep the Requirement and Design stage boundaries explicit. Do not enter Construction, Testing, or Deployment from these agents.
- Design Agent V2 uses AI-DLC for Application Design and Product Design as the only GUI design/prototype capability. Do not add Penpot or a frontend fallback.
- Run the relevant offline tests after changes; keep real external smoke tests separate.
- Do not commit secrets, credentials, private requirement documents, or runtime session data.
- Prefer minimal, reviewable changes over new frameworks or unnecessary abstractions.

---
> Source: [CabbageFans/complex_platform](https://github.com/CabbageFans/complex_platform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
