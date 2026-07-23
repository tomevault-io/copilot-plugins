## argosytides

> **Argosy Tides** is a maritime trading simulation built with Unreal Engine 5.4+.

# Copilot Instructions for Argosy Tides

## Project Context

**Argosy Tides** is a maritime trading simulation built with Unreal Engine 5.4+.

### Tech Stack
- **C++20** - Core game systems (UE5 native)
- **Blueprint** - UI and gameplay scripting
- **Rust 1.78+** - Data pipeline (real-world APIs)
- **Python 3.11+** - Prototyping scripts
- **SQLite** - Game state and data storage

### Key Directories
```
Source/ArgosyTides/     - C++ game code
Content/                - UE5 assets (Blueprints, Maps, etc.)
Tools/DataPipeline/     - Rust data fetcher
Config/                 - UE5 configuration
docs/                   - Design documents
```

## Coding Standards

### C++ (UE5)
- Use `UPROPERTY()` for all UObject references
- Use `UFUNCTION()` for Blueprint-exposed functions
- Prefix classes: `U` (UObject), `A` (Actor), `F` (Struct)
- Prefix bools with `b` (e.g., `bIsPlayerOwned`)
- Use `FString`, `TArray`, `TMap` (not std::)
- Log with `UE_LOG(LogTemp, Level, TEXT("message"))`

### Rust
- Use `anyhow::Result` for error handling
- Use `tracing` for logging (info/warn/error)
- Use `?` operator for error propagation
- Rate limit API calls (500ms between requests)

### Blueprint
- Prefix with `BP_` (e.g., `BP_Ship`)
- Start from C++ base classes
- Add comments to complex logic

## Common Patterns

### Data Manager Pattern
```cpp
UCLASS()
class UDataManager : public UObject {
    UFUNCTION(BlueprintCallable)
    FCommodityPrice GetCommodityPrice(FString Id);
    
    UPROPERTY()
    TMap<FString, FCommodityPrice> Cache;
};
```

### Calibration Pattern (5% drift)
```cpp
GamePrice = FMath::Lerp(GamePrice, RealPrice, 0.05f);
```

## Build Commands

```powershell
# Build UE5 project
.\build.bat

# Build Rust pipeline
cd Tools\DataPipeline
cargo build --release

# Test Python prototype
python Tools/fetch_data_prototype.py --all
```

## Architecture Principles

1. **Real data = calibration only** (not direct gameplay)
2. **Local simulation = actual prices**
3. **C++ for performance, Blueprint for iteration**
4. **Rust for data pipeline (compiled, no runtime deps)**

## Phase 1 (MVP) Scope

Focus on:
- ✅ One port (Oslo)
- ✅ One ship (Panamax container)
- ✅ One commodity (grain)
- ✅ Basic UI screens
- ✅ Save/load system

Avoid:
- ❌ Multiple ports (yet)
- ❌ Weather system (yet)
- ❌ Historical eras (Phase 4)
- ❌ Multiplayer (Phase 5)

## Documentation

- **GDD:** `docs/GDD.md` - Game design
- **TAD:** `docs/TAD.md` - Technical architecture
- **SETUP:** `SETUP.md` - Installation guide
- **AI_INSTRUCTIONS:** `.github/AI_INSTRUCTIONS.md` - Detailed guide

## Gotchas

1. **Always compile C++** before testing in Blueprint
2. **Use UPROPERTY()** for garbage collection
3. **Rust needs Visual Studio C++ tools** for MSVC linker
4. **Git LFS** for binary assets (`.uasset`, `.umap`)
5. **GitHub CLI authentication** - Use `GITHUB_PAT_TOKEN` from `.env` file:
   ```powershell
   $env:GH_TOKEN = $env:GITHUB_PAT_TOKEN
   gh auth status
   ```   Ensure `.env` is ignored by Git and not committed to the repository.
## When in Doubt

1. Check existing code patterns in `Source/ArgosyTides/`
2. Consult `docs/TAD.md` for architecture
3. Review `PHASE0_STATUS.md` for current state
4. Ask about UE5 best practices

---
> Source: [egkristi/ArgosyTides](https://github.com/egkristi/ArgosyTides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
