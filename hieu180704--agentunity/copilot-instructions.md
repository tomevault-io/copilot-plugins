## agentunity

> - **NEVER generate code or scripts that directly modify `.prefab`, `.unity`, `.asset`, or `.meta` files as raw text/YAML.** These files are managed by the Unity Engine; manual text modifications break GUIDs and FileIDs causing silent runtime reference failures.

# GitHub Copilot Instructions for Unity & C# (AgentUnity Framework)

## Core Guidelines & Safety
- **NEVER generate code or scripts that directly modify `.prefab`, `.unity`, `.asset`, or `.meta` files as raw text/YAML.** These files are managed by the Unity Engine; manual text modifications break GUIDs and FileIDs causing silent runtime reference failures.
- When suggesting Unity modifications, provide C# Editor scripts, MCP commands, or clear Inspector step-by-step instructions.

## C# Code & Performance Conventions
1. **Zero-GC in Hot Paths (`Update`, `LateUpdate`, `FixedUpdate`):**
   - Do NOT allocate objects (`new Class()`), closures, LINQ, or boxing operations in per-frame methods.
   - Cache `GetComponent`, `Camera.main`, and hash strings (`Shader.PropertyToID`, `Animator.StringToHash`) in `Awake()` / `Initialize()`.
   - Use NonAlloc physics queries (`Physics.RaycastNonAlloc`, `Physics.OverlapSphereNonAlloc`).
2. **Naming Conventions:**
   - Types, Methods, Properties, Public Fields, Enums: `PascalCase`.
   - Private/Protected Fields: `_camelCase` or `m_camelCase`.
   - Interfaces: `I` prefix (e.g., `IDamageable`, `IInteractable`).
   - Constants: `UPPER_SNAKE_CASE` or `PascalCase`.
3. **Architecture & Modularity:**
   - Use Assembly Definitions (`.asmdef`) for modular compilation (target compile time < 1s).
   - Use Type-Safe Event Bus with `readonly struct` payloads for decoupled communication.
   - Use Object Pooling for frequently spawned items (bullets, effects, floating texts).
4. **DOTween Best Practices:**
   - Always call `.DOKill()` before restarting tweens on the same target.
   - Set `.SetUpdate(true)` for UI animations that run during pause (`Time.timeScale == 0`).

## Living Docs Reference
- Game Design & Specs: `Docs/SourceOfTruth/`
- Architecture Decision Records: `Docs/Decisions/`
- Worklog & Handoffs: `Docs/Handoffs/` and `Docs/Done/`

---
> Source: [hieu180704/AgentUnity](https://github.com/hieu180704/AgentUnity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
