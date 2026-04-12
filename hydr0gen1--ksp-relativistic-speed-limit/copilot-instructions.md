## ksp-relativistic-speed-limit

> KSP 1.12.x mod that enforces a relativistic speed cap on all vessels using

# CLAUDE.md — Relativistic Speed Limit (root)

## Project overview
KSP 1.12.x mod that enforces a relativistic speed cap on all vessels using
rapidity mapping: `v_rel = c · tanh(|v_newton| / c)`. Implemented as a
`VesselModule` so KSP attaches it automatically to every vessel.

## Repository layout
```
RelativisticSpeedLimit/          # C# source + .csproj
  RelativisticSpeedLimit.cs      # VesselModule — core logic
  RSLSettings.cs                 # ConfigNode load/save
  RelativisticSpeedLimit.csproj  # net472, no external deps
  GameData/
    RelativisticSpeedLimit/
      Settings.cfg               # User-editable config (ship with the DLL)
CLAUDE.md                        # ← you are here
RelativisticSpeedLimit/CLAUDE.md # Source-level context
RelativisticSpeedLimit/GameData/RelativisticSpeedLimit/CLAUDE.md  # Config context
README.md
```

## Active development branch
`claude/ksp-relativistic-speed-mod-6lGZC` — all changes go here.
Never push directly to `main` without a PR.

## Build
```bash
dotnet build RelativisticSpeedLimit/RelativisticSpeedLimit.csproj \
    /p:KSP_DIR="/path/to/KSP"
```
Output goes to `<KSP_DIR>/GameData/RelativisticSpeedLimit/` when `KSP_DIR`
is set, or `RelativisticSpeedLimit/GameData/RelativisticSpeedLimit/` otherwise.

## Constraints (never violate these)
- Target: KSP 1.12.x, .NET Framework 4.7.2, C# 7.3
- No Harmony — zero method patches, pure VesselModule only
- No Unity UI (no ScreenMessages, no toolbar, no GUI windows)
- No new NuGet dependencies; only KSP/Unity assemblies from `lib/` or `KSP_DIR`
- FixedUpdate hot path must be O(1) and allocation-free
- `Settings.cfg` format must remain a single `RSL_SETTINGS {}` ConfigNode block

## Out of scope (do not add)
- Time dilation
- On-rails velocity correction
- GUI / toolbar integration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hydr0gen1)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Hydr0gen1)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
