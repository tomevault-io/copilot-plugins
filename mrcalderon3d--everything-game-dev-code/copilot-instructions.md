## 01-engine-isolation

> Prevent Unity, Unreal, and Godot guidance from contaminating each other.


Engine isolation is mandatory.

- Use `rules/common/` first.
- Then use exactly one engine layer for implementation: `rules/unity/`, `rules/unreal/`, or `rules/godot/`.
- Only compare engines in research contexts. Do not mix their implementation advice in production tasks.

---
> Source: [MRCalderon3D/everything-game-dev-code](https://github.com/MRCalderon3D/everything-game-dev-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
