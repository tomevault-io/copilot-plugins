## triaevum

> Read `docs/OOT3D_NRI_FIDELITY_EXTENSION_ARCHITECTURE.md` before changing the renderer, presentation pipeline, UI composition, or extension settings.

# OOT3D Recomp Engineering Rules

Read `docs/OOT3D_NRI_FIDELITY_EXTENSION_ARCHITECTURE.md` before changing the renderer, presentation pipeline, UI composition, or extension settings.

- Native OOT3D assets, commands, PICA state, and typed composition metadata are authoritative. Never repair a scene with asset-specific runtime corrections.
- The compiled effect graph owns pass order, resources, barriers, and lifetime. Effects may use only declared inputs at declared native composition anchors.
- World, transparent, atmosphere, UI, and presentation are separate domains. Scene effects must complete before UI composition; HUD and menus must never become scene-effect inputs.
- Preserve native draw order. A transparent draw may cover a scene effect only through its real native ordering and visibility contract.
- Keep title-specific addresses and gameplay semantics in the OOT3D adapter. Keep reusable PICA/NRI mechanisms in the shared 3DS renderer.
- Give each setting one owner and one F1 control surface. Input owns device routing; TopScreen owns TopScreen behavior and layout; the window host owns output mode and extent; the renderer owns internal resolution.
- For F1 changes, run the actual-widget smoke test described in `docs/TRIAEVUM_F1_MENU_REVIEW.md`. Preserve capability-safe Off controls, explicit persistence errors, and display confirmation outside individual tabs.
- Optional effects must be behaviorally inert when disabled and must not alter canonical shaders, depth/stencil/blend state, draw order, targets, or transfers.
- Validate renderer changes with deterministic framebuffer capture and structural diagnostics, never Windows screenshots alone.
- Treat external decompilation repositories as read-only evidence unless the user explicitly requests synchronization or decompilation work.
- Keep changes modular, tested at their owning boundary, and free of hidden state, string-patched scheduling, and per-scene exceptions.

## Public release invariant

- Public TriAevum packages are allowlist-built and must pass the release audit.
- The adopted release model ships explicitly catalogued precompiled title logic
  and its corresponding translated source, like static Xbox 360 recomp ports.
  Read `docs/TRIAEVUM_PRECOMPILED_RELEASE.md` before changing this boundary.
- Never package ROM images, keys, extracted assets, `code.bin`, save states,
  captures, mod payloads, object caches, or SDKs. Do not disguise translated
  title code as title-neutral code in release metadata.
- Forge's user installation path must not generate IR, compile, link, or acquire
  an SDK. A missing/incompatible title module is a package error, not permission
  to fall back to compilation. Developer build commands remain separate.
- The distributed runtime and Forge are separate from user ROM-derived data.
  Game-derived `game.tam`, `content.tap`, and shader-cache files stay on the
  user's machine and must be reproducible from user-provided inputs.
- Do not copy legacy development directories into public packages wholesale.
- Preserve all donor notices and keep proprietary SDKs or DLLs outside the
  default public package unless a separate license review explicitly permits
  redistribution.

---
> Source: [coccofresco/TriAevum](https://github.com/coccofresco/TriAevum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
