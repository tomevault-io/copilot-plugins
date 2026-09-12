## veyra-interactive-car

> Read README.md, docs/ARCHITECTURE.md and docs/VERIFICATION.md before editing.

# Working on VEYRA

Read README.md, docs/ARCHITECTURE.md and docs/VERIFICATION.md before editing.

For an exact reproduction, use tag v1.0.0 and docs/RECREATE.md. The release is the source of truth. Do not redesign while restoring it.

- Keep `src/HoverVideo.tsx`, media files, timing and state contracts unchanged for presentation-only work.
- Do not convert the scene into a scrolling landing page, substitute the assets, autoplay looping video or add a backend to playback.
- Match Space Grotesk 400, the blue/white/electric-green palette and the framed composition.
- Preserve the neutral-pose gate, mutually exclusive appearance modes, closing locks, decoded-frame gate, endpoint queue, frozen detail handoff, Escape and reduced motion.
- Use the existing dependency versions. The Windows-specific direct esbuild dependency was removed for portable installation; esbuild selects its own platform package.
- Run `pnpm test` and `pnpm build`. For reference restoration also run `pnpm verify:reference`. The reference check deliberately rejects changes to approved media/source; do not regenerate the manifest to conceal a mismatch.
- Inspect the browser for visual changes. Test at 1440x900, 1280x720 and 390x844, including both detail views and both menus.
- LTX is optional and offline from the page. Keep `LTX_API_KEY` in the local environment or `.env.ltx.local`, never in `src/`, `public/`, a VITE_ variable, screenshots or logs. Use the dry run before a paid request; obtain the user's authorization for generation cost and input upload.
- Keep new outputs in `generated/` until approved. Never overwrite approved files during generation.
- Do not deploy, publish a fork or commit credentials unless the user has authorized the applicable action. Local edits and checks are normal work.
- Record known failures honestly. The v1.0.0 appearance close/focus race is documented; reproducing the release does not imply it has been fixed.

This repository is self-contained. It does not require the author's private notes, local paths, installed skills or conversation history.

---
> Source: [amirmushichge/veyra-interactive-car](https://github.com/amirmushichge/veyra-interactive-car) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-12 -->
