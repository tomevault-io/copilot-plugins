## autolive2d

> Start with `README.md`, then use the document that matches the task:

# AutoLive2d Agent Guide

Start with `README.md`, then use the document that matches the task:

- New machine or deployment: `docs/deployment-guide.md`
- Adapt a split PSD: `docs/psd-adaptation-guide.md`
- Start from one screenshot: `docs/screenshot-to-live2d-workflow.md`
- Change deformation/tracking math: `docs/project-algorithms-and-workflow.md`
- Find code ownership: `docs/project-structure.md`
- Validate a change: `docs/rig-validation-checklist.md`

Core invariants:

- Draw `z` controls visual order; pseudo-Z only controls projection.
- Body motion carries neck and head. Do not introduce independent head translation that can separate the neck.
- Eye sockets stay attached to the face. Iris overflow must be clipped, not painted white.
- Mouth opening scales around the saved mouth center and must not drift.
- Hair/accessory roots remain attached; inertia increases toward tips.
- Runtime popup and editor must produce the same animation for the same project and tracking parameter stream.
- Keep `u3` as the first regression preset and verify older presets when adding optional fields.

Repository boundaries:

- `third_party/see-through` is vendored Apache-2.0 code with local modifications. Preserve its `LICENSE` and update `LOCAL_CHANGES.md` when changing the fork.
- Never commit `.venv`, model weights, `workspace`, generated outputs, camera captures, API keys, access tokens, or local validation screenshots.
- Built-in PSD/image samples are test assets and are not covered automatically by the root source-code license.

Minimum checks for editor changes:

```powershell
npm ci
npm run build
npm run inspect:sample
```

For visible rig/deformation changes, also start the dev server and run:

```powershell
npm run validate:ui -- --preset u3
```

Then manually inspect neutral pose, all expression presets, eye extremes, mouth extremes, head X/Y/Z extremes, body/neck/head chaining, arm pivots, dynamics, and proxy-head draw order.

---
> Source: [Fenglin-Maple/AutoLive2d](https://github.com/Fenglin-Maple/AutoLive2d) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
