## kathacut

> Read docs/PRODUCT.md, docs/ARCHITECTURE.md and docs/ROADMAP.md before implementation. Update docs/STATUS.md after each meaningful completed slice with changes, verification, limitations and next task.

# Project instructions

Read docs/PRODUCT.md, docs/ARCHITECTURE.md and docs/ROADMAP.md before implementation. Update docs/STATUS.md after each meaningful completed slice with changes, verification, limitations and next task.

## Product constraints
- Core workflow is local and works offline after explicit model downloads. No accounts, cloud inference, telemetry, media uploads or required subscription.
- Automatic transcription from the video's audio, timed subtitles and timeline editing are first-release requirements.
- Preserve Malayalam shaping and mixed Malayalam/English text. Never animate raw Unicode code units or split vowel marks from their grapheme clusters.
- SRT import preserves original text and cue timings unless the user explicitly edits or requests transformation. Estimated word timing must never masquerade as aligned timing.
- Treat user corrections as authoritative. Retranscription must not silently overwrite them.
- Preview and export share caption layout and time-driven animation logic.
- Keep macOS and Windows paths and worker integration portable. Do not assume CUDA on a Mac.

## Engineering
- Proposed stack: Electron, React, TypeScript; separate native/media worker processes. Verify maintained versions and licensing before selecting dependencies; commit the lockfile.
- Renderer has no Node integration. Use context isolation and a narrow validated preload/IPC bridge. Spawn tools with argument arrays, not interpolated shell commands.
- Use source-media timestamps as canonical time; seconds are for playback adapters only. Avoid accumulating frame rounding errors.
- Never block the UI thread with transcription, waveform extraction or export.
- Never overwrite input media. Use atomic project saves, recovery copies and relinking for missing media.
- Keep actual models, recordings, cache files, generated exports and credentials out of git.
- Prefer small working vertical slices. Do not present mock transcription, fake progress or nonfunctional export controls as implemented features.
- Test high-risk timing, SRT round trips, edits/undo, migrations and export parity. Report platforms actually tested; do not claim Mac/Windows validation from a Linux-only environment.
- No Remotion commitment: evaluate redistribution/license suitability before adopting it. Track FFmpeg build configuration, models and fonts in the dependency license inventory.

---
> Source: [vxlabs/KathaCut](https://github.com/vxlabs/KathaCut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
