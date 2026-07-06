## yh-cheat

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Deploy

**Prerequisites:** Android NDK (set `ANDROID_NDK_PATH`), `ccache`, `mold` linker, `adb` with rooted device, external `injector` binary at `/data/adb/injector`.

**Build only (no deploy):**
```bash
source scripts/build.sh
```

**Build + deploy + inject:**
```bash
source scripts/run.sh
```

Both scripts set up NDK toolchain env vars, `ANDROID_API_LEVEL=29`, and `RUSTFLAGS` (mold linker, NEON, gc-sections, icf). Primary target is `aarch64-linux-android`. `run.sh` pushes both `libRS.so` and `yh-cheat` to `/data/adb/` on device, injects the lib, then launches the overlay.

**Output artifacts:**
- `target/aarch64-linux-android/release/libRS.so` — injected library (`cdylib`)
- `target/aarch64-linux-android/release/yh-cheat` — overlay binary

**No test suite exists.** The project is tested manually on device.

## Architecture

Two-process design targeting `com.hottagames.yh.laohu` (Tower of Fantasy, UE5):

1. **Injected library (`libRS.so`)** — loaded into game process via `JNI_OnLoad` (`src/lib.rs`). Waits for `libUnreal.so` to appear in `/proc/maps`, then injects a fake `FTickableGameObject` into UE5's main-thread TArray to get per-frame callbacks. Each tick reads game state through the SDK crate and serializes it to WuWa shared memory (`rkyv` zero-copy, 32MB).

2. **Overlay binary (`yh-cheat`)** — separate process (`src/main.rs`). Reads injection metadata from a TOML file (`--injection`, default `/data/adb/injection_result.toml`), connects to WuWa shared memory via `read_shm_id_from_process`, creates a native window overlay, and renders a player list using ImGui foreground draw list. `--bp` enables cgroup bypass.

### IPC Mechanism

**Shared memory:** Uses WuWa kernel driver — injected lib calls `driver.shm_create(32MB)` and publishes the `shm_id` (u64) by overwriting the ELF header at offset 0 via `mprotect + write_volatile`. Overlay reads the `shm_id` via `process_vm_readv` polling, then maps the region via `driver.shm_map(shm_id)`.

**`poll_remote_header` retries:** retries on both `ESRCH` (process not yet started) and `EFAULT` (address not yet mapped — happens when library just injected but ELF sections not yet loaded). Max 120 attempts × 500ms = 60s timeout.

**Protocol** (`bridge/src/protocol.rs`): 64-byte header + rkyv payload. **Seqlock** for lock-free consistency. States: `Ignore → WaitingForTick → WaitingForMatch → InMatch`.

**`MatchData`** contains: `players: Vec<Player>` with position, health, is_ai, team_id, name.

### Ticker Injection (`src/ticker.rs`)

No WuWa observer/breakpoint. Instead, directly injects a fake object into `FTickableStatics` main-thread TArray (identified by IDA analysis of `sub_57E6DB8` = `FTickableGameObject::TickObjects`):

- **mutex**: `libUnreal.so + 0xD6B6608` (`stru_D6B6608`)
- **TArray.Data**: `libUnreal.so + 0xD6B6630`
- **TArray.Num**: `libUnreal.so + 0xD6B6638` (low 32 bits)
- **TArray.Max**: `libUnreal.so + 0xD6B663C`
- Entry stride: 16 bytes `{FTickableGameObject* (8B), bTickEvenWhenPaused (1B), pad (7B)}`

Vtable layout (from decompile): `[0]=Tick(float)`, `[2]=IsTickableWhenPaused→false`, `[3]=IsTickable→true`, `[9]=GetTickableGameObjectWorld→nullptr`.

Inject sequence: lock mutex → grow TArray if needed (realloc) → write fake entry → increment Num → unlock.

**Non-main-thread TArray** (`0xD6B6640/0xD6B6668/0xD6B6670`) is intentionally untouched.

### SDK (`sdk/`)

UE5 SDK from Dumper-7 dump (`5.6.1-0+UE5-HT`, 1215 files in `sdk/cpp/SDK/`). Build only compiles `Basic.cpp` + `bridge.cpp` — the rest of the SDK `.cpp` files are ProcessEvent implementations not needed. Headers included for type definitions.

**bridge.cpp key class mappings (HT → UE5):**
- All characters: `UGameplayStatics::GetAllActorsOfClass(w, AHTAbilityCharacter::StaticClass(), &actors)`
- Health: `AHTAbilityCharacter::GetHP()`, `GetHPMax(false)`
- Alive: `AHTAbilityCharacter::CharacterIsAlive()`
- Is AI: `AHTAbilityCharacter::IsAICharacter()`
- Team: `*(uint8_t*)(character + 0x0990)` (FGenericTeamId)
- Player name: `*(FString*)(player_state + 0x0440)` (`m_strRoleName` in AHottaPlayerState)
- Mesh: `*(USkeletalMeshComponent**)(character + 0x0348)`

**SDK compile quirks fixed:**
- `PATH_MAX` system macro conflicts with `HTGame_structs.hpp` enum — renamed to `PATH_MAX_VAL`
- `FUtf8String` forward-declared as `using FUtf8String = FString` in `Basic.hpp`
- `TPersistentObjectPtr::Get()` qualified with full template type
- `FTransform::translation` → `FTransform::Translation` (UE5 capitalization)
- Narrowing warnings suppressed with `-Wno-c++11-narrowing`

## Crate Structure

- **Root crate (`yh-cheat`)** — Rust edition 2024. `cdylib` (libRS.so) + `bin` (overlay). `src/collector/character.rs` collects alive characters each tick. `src/lib.rs` owns JNI_OnLoad, waiter, ticker injection, SHM setup.
- **`esp/`** — graphics, windowing, ImGui, WuWa driver SDK.
- **`sdk/`** — UE5 C++ SDK bridge via cxx. Only `Basic.cpp` + `bridge.cpp` compiled.
- **`bridge/`** — SHM protocol, seqlock writer/reader, InjectionResult parsing.

## Key Patterns

- **No spoof call** — target game has no call stack detection, direct vtable calls used.
- **libUnreal.so not libUE4.so** — UE5 ships as `libUnreal.so`.
- **String obfuscation:** `obfstr!()` / `obfstring!()` for all sensitive strings.
- **Game offsets:** `src/constants.rs` — `android_app_ptr` and ticker TArray offsets. All others are TODO (need reversing for each game update).
- **Rust 2024:** unsafe operations inside `unsafe fn` require explicit `unsafe {}` blocks.

## Code Style

- Rust: Edition 2024, `max_width=120`, trailing commas in match, field init shorthand.
- C++: C++17, exceptions disabled (`-fno-exceptions`).
- Release: `opt-level="z"`, LTO, `panic="abort"`, stripped, single codegen unit.
- CMakeLists.txt is for **IDE autocomplete only**.
- Comments and log messages in Chinese.

---
> Source: [fuqiuluo/yh-cheat](https://github.com/fuqiuluo/yh-cheat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-06 -->
