## ai-arena-cpp

> This is the native Windows C++23 application, version 0.0.5. Read README.md, BUILD.md and the relevant source before changing a feature. The public repository is https://github.com/neeveew/AI-Arena-CPP; the website is https://aiarena.me.

# Working on AI Arena C++

This is the native Windows C++23 application, version 0.0.5. Read README.md, BUILD.md and the relevant source before changing a feature. The public repository is https://github.com/neeveew/AI-Arena-CPP; the website is https://aiarena.me.

## Architecture map

| Location | Responsibility |
| --- | --- |
| src/app | Native Windows UI, retained controls, settings, transcript presentation and application composition. WindowsApp.cpp connects the surfaces; its WindowsModelMemory*.inl files implement model workspace panels. |
| src/core | Conversation and turn logic, context planning primitives, provider interfaces, transcript classification and SQLite storage. Public headers are under src/core/include/aiarena. |
| src/services | Auto Chat execution, context and memory policies, model lifecycle and resource planning. |
| src/worker | Isolated inference worker, its transport and managed llama.cpp backend. |
| src/operations | Durable operation identities, receipts, event history, control contracts and operational storage. |
| protocols | Authoritative protobuf schemas for Arena control and connected local apps. |
| src/app/LocalApp* | App discovery, registration, handshake and model-directed tool exchanges. Keep this generic; Auralis is a peripheral integration. |
| src/cli, src/extensions, src/tools | Command-line entry point, extension loading and maintenance utilities. |
| cmake, CMakeLists.txt | Build targets, dependency pins and runtime staging. |
| assets, tests, docs | Application assets, focused tests and public documentation. |

## Implementation rules

- Put scratch output, build trees and disposable test profiles under .workspace. Keep generated output, caches, credentials, model weights and personal data out of source changes.
- Do not edit generated protobuf bindings or files under a build directory. Change the schemas or generator configuration and regenerate through CMake.
- Keep the conversation and legacy layouts consistent in behavior. Presentation changes should use the existing shared control and transcript helpers where possible.
- Preserve participant identity, message roles, cancellation behavior and model load/unload ownership. Do not block the UI thread with provider calls, inference, filesystem scans or long database operations.
- Keep public transcript text separate from actual provider reasoning and private tool traces. Do not fabricate reasoning or turn diagnostic prompts into public replies.
- Connected apps advertise tools; the model chooses whether to call them. Do not add an app-specific persona, keyword router or forced tool use to ordinary conversation.
- Preserve protobuf field compatibility, authentication and operation idempotency when changing local endpoints.
- Keep the pinned llama.cpp adapter/runtime ABI pairing. Runtime redistribution must follow the reviewed provenance recipe, including the LLVM 21.1.8 OpenMP build and notices; do not copy an unreviewed upstream DLL into a release.
- Respect LICENSE. Private modifications and permission to redistribute modified versions are different matters.

## Validation and handoff

Build the affected targets and run focused tests that verify the changed behavior. BUILD.md includes a small model-free example. Historical release-qualification tests and private machine fixtures are not the default development gate.

Do not load a user's GGUF files, modify their conversation database, run a repair utility or start a live app integration solely to test a change unless that use is requested. Prefer temporary profiles and synthetic fixtures. Read the parameters of scripts/ai-arena-control.ps1, scripts/Repair-AIArenaOperationalStore.ps1 and scripts/verify-release.ps1 before using them; repair and release containment can affect running processes or stored state.

Report what changed, what was tested and any remaining limitation. Keep changes scoped and reviewable. Do not commit build output or publish a release unless the task authorizes it.

---
> Source: [neeveew/AI-Arena-CPP](https://github.com/neeveew/AI-Arena-CPP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
