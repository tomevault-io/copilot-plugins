## intentflow-ios

> This repository uses IntentFlow architecture. Treat `.intentflow.yaml` manifests as source-of-truth contracts for AI-mode features.

# IntentFlow Copilot Instructions

This repository uses IntentFlow architecture. Treat `.intentflow.yaml` manifests as source-of-truth contracts for AI-mode features.

When generating or editing code:

- Read `.ai/agent-context.md` and the relevant manifest before touching implementation files.
- Model product behavior before UI.
- Add or update `State`, `Intent`, `Event`, `Effect`, `Output`, and `Route` before writing adapters.
- Keep reducers pure. Do not call network, database, analytics, storage, timers, or UI APIs from reducers.
- Put async work in `FlowEffectHandler`.
- Use `EffectID` for long-running, repeatable, or replaceable work.
- Represent navigation as `Route`.
- Represent parent communication as `Output`.
- Keep SwiftUI views and UIKit view controllers as adapters.
- Put display formatting in `FlowProjection`.
- Add transition tests for every new workflow branch.
- For AI mode, update the `.intentflow.yaml` manifest before changing the Swift contract.
- Do not add hidden callbacks when a typed output or route would express the behavior.
- Prefer compact context from `swift run intentflow ai-context <manifest> --tool copilot`.
- Apply `.github/instructions/*.instructions.md` for path-specific Swift and documentation rules.

Generated code is incomplete unless it includes tests for the transition it introduces.

---
> Source: [emrecanozturk/intentflow-ios](https://github.com/emrecanozturk/intentflow-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
