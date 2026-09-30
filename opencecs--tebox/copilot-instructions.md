## gki

> Project source layout, ARM64 build and Android GPU validation


Read `AGENTS.md` for this workspace's source boundaries and boot invariants.
Skills are shared through `.cursor/skills/` → `.ai/skills/`; prompts are in
`.ai/prompts/`. Use `scripts/env.sh` for paths, and `.ci/README.md` /
`.ai/README.md` for build/CI and agent wiring.
Preserve current GPU rendering and user data when changing build configuration.
Track `qemu/` + `thirdparty/` sources and this variant’s `*.img`; ignore only
compile products under `out/` / `prebuilts/` / `dist/`.

---
> Source: [opencecs/tebox](https://github.com/opencecs/tebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
