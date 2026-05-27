## ijfw

> This file follows the open AGENTS.md spec (https://agents.md/) and is the

# AGENTS.md

This file follows the open AGENTS.md spec (https://agents.md/) and is the
canonical agent-instructions surface for this project. Platform-specific
files (CLAUDE.md, GEMINI.md, WAYLAND.md, codex/AGENTS.md, .cursorrules,
.windsurfrules, copilot-instructions.md) are thin adapters that point here.

Four IJFW-managed regions live in this file. Content outside the markers is
yours -- IJFW will never touch it.

<!-- IJFW-MEMORY-START -->
Project memory at .ijfw/memory/. Call `ijfw_memory_prelude` for full context.

Last handoff: IJFW 1.3.2 RC audit/verify pass is complete locally. The 1.3.1 release caution still applies: user explicitly said, "When windows completes and verifies we go forward. Remember that". Do NOT create/push tags or attempt npm publish until Windows CI/release clearance is explicitly given by the user. Current 1.3.2 verification has passed 880/880 Node tests, 158/158 focused regression tests, package dry-runs, temp-project swarm E2E, cross-audit reconciliation, platform drift, diff hygiene, and preflight blocking gates. Findings fixed during audit: hermetic update-confirm npm test PATH, Trident EPIPE hardening, workflow bootstrap/fallback guidance, team/AGENTS mirroring specificity, Codex dispatch command order, and memory/release-gate consistency. Advisory warnings only: eslint-security non-literal fs warnings in installer paths and PSScriptAnalyzer skipped because pwsh unavailable locally. No tag or publish performed.
<!-- IJFW-MEMORY-END -->

<!-- IJFW-ROUTING-START -->
<!-- IJFW-ROUTING-END -->

<!-- IJFW-AGENTS-START -->
No project agents yet. Run `ijfw team` to set them up.
<!-- IJFW-AGENTS-END -->

<!-- IJFW-BLACKBOARD-START -->
<!-- Reserved for Pillar B multi-CLI orchestration. Empty in alpha. -->
<!-- IJFW-BLACKBOARD-END -->

---
> Source: [FerroxLabs/ijfw](https://github.com/FerroxLabs/ijfw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
