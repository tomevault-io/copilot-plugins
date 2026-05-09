## project-agent-workflow

> > esp-rainmaker-home: before substantive work, follow the same routing model as cursor/agents/project-agent.md (layers, questions, change kind, output shape).


# Project agent workflow (esp-rainmaker-home)

Aligned with **`.cursor/agents/project-agent.md`**. For **non-trivial** work (code changes, multi-file debug, new behavior, “where should this live?”), **before** editing:

1. **Questions (if needed)** — If goal, SDK, scope, or layer is unclear, ask **2–5 short questions** (or state assumptions). If blocked, output questions first; optional provisional route.
2. **Change kind** — One primary: **modify** · **add-files** (same domain/module) · **new-module** · **new-feature** (often `features.config` + surfaces + maybe `store` / `sdk-adaptors`).
3. **Classify** — Intent: add \| update \| fix \| refactor \| configure. Map to **Category**: Config \| Integrations \| SDKAdaptors \| NativeAdaptors \| Features \| Context \| Shared \| Store \| CrossCutting.

**Startup flow (for orientation):** `integrations` → `runtime.config` → `sdk-adaptors` → `initCDF` (`store`) → active SDK via `sdk.config` + `sdk.identifiers`. `sdk.config` injects `native-adaptors` into `@espressif/*-sdk`. `features.config` ↔ `sdk.config`. `params.config` → `shared` ParamControls.

**Layer map (same as project-agent):**

- **Config** — `config/` (URLs, identifiers, runtime, flags, devices, params, agent)
- **Integrations** — `src/integrations/` (`initializeApp`, adaptor registry, `initCDF`)
- **SDK adaptors** — `src/sdk-adaptors/` (`ESPSDKAdaptor`, transformers → ESPCDF)
- **Native adaptors** — `src/native-adaptors/` (RN/native bridges for SDK config)
- **Features** — `src/features/<domain>/` (screens, components, hooks, utils, theme per domain)
- **Context** — `src/context/`: app-wide `store`, `appRestart`; **flow contexts only** for multi-screen coordination (schedule, automation, scene)—**do not** add `*.context.tsx` per feature; prefer hooks + `store`
- **Shared** — `src/shared/` (cross-feature UI, hooks, utils)
- **Store** — `src/store/` (ESPCDF, MobX, sync; no UI)
- Also **`src/tasks/`** or other paths when relevant; devices: `config/devices.config.ts` + feature UI.

**Cross-cuts:** Features vs shared—domain code stays in `features/<domain>/`; lift to `shared/` only when reused across domains. Context—only multi-screen flows; else `store` + hooks. SDK API changes often touch **sdk-adaptors** + **store** + **sdk.config** / identifiers. Param UI: **params.config** + **shared/ParamControls**. Bad startup/SDK: **integrations** + **runtime.config** + **sdk.config**.

**Mini output (before coding):** Category · Change kind · Intent · primary + related paths · what to add/change/remove at file/module level · why (1–2 sentences) · risks if any.

Skip for pure Q&A with no code impact, or when the user wants a one-liner only.

**Subagent:** For read-only triage only, delegate via Task with subagent **`project-agent`** (see `.cursor/agents/project-agent.md`).

---
> Source: [espressif/esp-rainmaker-home](https://github.com/espressif/esp-rainmaker-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
