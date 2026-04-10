## fileorganizer

> File Organizer is a show-case application whose sole purpose is to demonstrate Apple Intelligence (on-device Foundation Models) in a real-world workflow.

File Organizer is a show-case application whose sole purpose is to demonstrate Apple Intelligence (on-device Foundation Models) in a real-world workflow.
• The app will not fall back to rule-based heuristics if the model is unavailable.
• Any feature or code path that assumes “AI disabled” should be removed.
• Future work (Fast/Thorough speed modes, richer metadata extraction, etc.) must keep this AI-only contract intact.


No rule-based fallback; extension logic removed as of PR #6.

NOTE: Tutorial screen will explain how to enable Apple Intelligence.
For now this is a placeholder; app may sit idle on incompatible Macs.

TODO (next PR): add SpeedMode (thorough, fast) and token-budget guard
– thorough: roll-up at 3 500 tokens
– fast    : hard reset at 2 000 tokens (env SPEED_MODE=fast)


As of PR #5 the Date-based sorting mode has been removed; the only supported mode is full Apple Intelligence analysis.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brookcs3)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/brookcs3)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
