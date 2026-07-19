## scp-sl-community-edition

> <!-- Companion docs — read these when working on the relevant system: -->

<!-- Companion docs — read these when working on the relevant system: -->
<!-- CLAUDE-systems.md       — detailed subsystem descriptions, role tables, ScriptableObjects -->
<!-- CLAUDE-firearm.md       — full weapon system: architecture, all bug fixes (sessions 2-6), animation, invariants -->
<!-- CLAUDE-scenes.md        — all 6 scenes: root hierarchies, Facility GameManager/camera stack (ViewmodelCamera!), HUD canvases -->
<!-- CLAUDE-prefabs-items.md — item/weapon prefabs: Defined Items (Resources), viewmodel/pickup/thirdperson triplets -->
<!-- CLAUDE-prefabs-roles.md — Player.prefab (ReferenceHub + all status effects), Defined Roles, models, ragdolls, SCP HUDs -->
<!-- CLAUDE-prefabs-world.md — doors, structures, lockers, elevators, targets, UI element prefabs, network-spawnable list -->
<!-- CLAUDE-binary-recovery.md — HOW TO READ GROUND TRUTH from the original build: field offsets (*_metadata.txt), ISIL decoding, Tools/ReadGameAssembly.ps1 for binary constants. READ BEFORE restoring any decompiled method — never guess constants/logic. -->
<!-- CLAUDE-workflow.md    — MANDATORY working method: the debugging loop, catalog of 18 known rip failure patterns with symptom heuristics, editing/verification rules. READ FIRST when fixing any bug in this project. -->
# CLAUDE.md — SCP: Secret Laboratory 12.0.2 Restoration Project

## Project Overview

**Product Name:** SCPSL (SCP: Secret Laboratory)
**Company:** Northwood (original game developer)
**Project Type:** Multiplayer horror/action game — restoration/reverse-engineering of SCP:SL v12.0.2
**Unity Version:** 6000.3.10f1 (Unity 6)
**Game Version:** 12.0.2 (Development build, versioned as `12.0.2-12.0.1-rc-2298ba84`)
**Target Platform:** PC (Windows primary, Linux server support)
**Default Resolution:** 1920x1080 (Full HD)

This is an open educational project that reverse-engineers and rebuilds SCP: Secret Laboratory version 12.0.2 for study purposes. The game is a multiplayer asymmetric horror game set in an SCP containment facility, where players take on roles as SCPs, D-Class personnel, scientists, MTF, or Chaos Insurgency.

---

## Development Philosophy (IMPORTANT — READ BEFORE MAKING CHANGES)

**Exact restoration is NOT required.** The goal is a working, maintainable, and enjoyable game — not a byte-perfect replica of v12.0.2.

### Allowed and encouraged:
- **Use v13 code (`CODES/SCP13.0/`) freely** for logic, architecture, and patterns — it is the same codebase evolved. Better code from v13 is preferred over broken stubs from v12 decompilation.
- **Improve readability and maintainability** — cleaner code is a better goal than matching the decompiled v12 style.
- **Add null guards, safety checks, and convenience helpers** that were missing in v12. Stability beats purity.
- **Adopt v13 patterns** (additive Weight instead of Lerp, proper OnActivated/OnShutdown transitions, etc.) even when v12 had simpler or buggier versions.
- **Keep the old PostProcessing stack** (`com.unity.postprocessing 3.5.4`, `PostProcessVolume`, `PostProcessProfile`) — the project does NOT use URP/HDRP. v13 uses `UnityEngine.Rendering.Volume`; adapt its logic but keep the old API types.
- **Future-proof the code** for modding and extension where it costs nothing.

### NOT allowed:
- Do not add gameplay features, balance changes, or new content that did not exist in v12.
- Do not break the Mirror networking contract (SyncVars, Commands, RPCs).
- Do not introduce Unity package upgrades without explicit user approval.

---

## Repository Structure

```
StabelProjectSCPSL12/
├── Assets/
│   ├── Scripts/
│   │   ├── Assembly-CSharp/      # Main game scripts (all gameplay code)
│   │   └── Utf8Json/             # Utf8Json serialization library (source)
│   ├── Mirror/                   # Mirror networking framework (full source)
│   ├── ParrelSync/               # Multi-instance Unity editor tool
│   ├── Plugins/                  # Native DLLs and precompiled assemblies
│   ├── SC Post Effects/          # Post-processing visual effects package
│   ├── UBER/                     # UBER shader system
│   ├── Standard Assets/          # Unity standard assets
│   ├── _Scenes/                  # Game scenes (menu, facility)
│   ├── Resources/                # Runtime-loadable assets
│   └── StreamingAssets/          # Streamed data
├── CODES/
│   ├── ClientCode/               # Reference decompiled client code
│   ├── ServerCode/               # Reference decompiled server code
│   └── SCP13.0/                  # Reference code from version 13.0
├── Packages/                     # Unity Package Manager manifest
├── ProjectSettings/              # Unity project settings
├── Translations/                 # Localization files
└── README.md
```

---

## Unity Packages (manifest.json)

| Package | Version |
|---|---|
| `com.unity.ai.navigation` | 2.0.12 |
| `com.unity.nuget.newtonsoft-json` | 3.2.2 |
| `com.unity.postprocessing` | 3.5.4 |
| `com.unity.ugui` | 2.0.0 |
| `com.unity.multiplayer.center` | 1.0.1 |
| `com.unity.ide.visualstudio` | 2.0.27 |

See CLAUDE-systems.md for package descriptions.

---

## Assembly Definitions (.asmdef)

All gameplay code compiles into the default `Assembly-CSharp` assembly (no custom .asmdef for game scripts).

| Assembly | Location |
|---|---|
| `Mirror` | `Assets/Mirror/Core/` |
| `Mirror.Transports` | `Assets/Mirror/Transports/` |
| `Mirror.Components` | `Assets/Mirror/Components/` |
| `Mirror.Editor` | `Assets/Mirror/Editor/` |
| `Unity.Mirror.CodeGen` | `Assets/Mirror/Editor/Weaver/` |
| `Mirror.CompilerSymbols` | `Assets/Mirror/CompilerSymbols/` |
| `Mirror.Examples` | — |
| `KCP` | `Assets/Mirror/Transports/KCP/kcp2k/` |
| `Telepathy` | `Assets/Mirror/Transports/Telepathy/` |
| `SimpleWebTransport` | `Assets/Mirror/Transports/SimpleWeb/` |
| `Edgegap` | `Assets/Mirror/Hosting/Edgegap/` |
| `EncryptionTransportEditor` | `Assets/Mirror/Transports/Encryption/Editor/` |
| `sc.posteffects.runtime` | `Assets/SC Post Effects/Runtime/` |
| `sc.posteffects.editor` | `Assets/SC Post Effects/Editor/` |
| `projectCloner` | `Assets/ParrelSync/` |

**Note:** Mirror requires `allowUnsafeCode: true` for performance-critical networking code.

See CLAUDE-systems.md for assembly descriptions.

---

## Third-Party Libraries & Plugins

### DLLs in `Assets/Plugins/`
| Library | Purpose |
|---|---|
| `AmplifyBloom.dll` | Bloom post-processing |
| `BouncyCastle.Crypto.dll` | RSA/ECDSA/AES/ECDH cryptography |
| `Caress.dll` | Unknown utility |
| `CommandSystem.Core.dll` | Command system core |
| `Facepunch.Steamworks.Win64.dll` | Steam lobby/auth/tickets |
| `NorthwoodLib.dll` | Northwood utility library |
| `YamlDotNet.dll` | YAML config parsing |
| `zxing.unity.dll` | QR code generation |

### Networking Stack
- **Mirror** — Primary networking (NetworkBehaviour, SyncVar, Commands, Rpcs)
- **LiteNetLib4Mirror** — Custom UDP transport wrapper
- **KCP2K** — KCP protocol transport
- **Telepathy** — TCP transport (fallback)
- **SimpleWebTransport** — WebSocket transport

### In-Source Libraries
- **MEC** — `Assets/Plugins/Assembly-CSharp-firstpass/MEC/` — coroutines (`Timing.RunCoroutine`)
- **Utf8Json** — `Assets/Scripts/Utf8Json/` — high-performance JSON
- **Discord SDK** — `Assets/Plugins/Assembly-CSharp-firstpass/Discord/` — Discord integration
- **CameraFilterPack** — `CODES/ClientCode/` — camera shader/filter collection
- **UBER Shader** — `Assets/UBER/` — advanced surface shaders

---

## Core Architecture

### Networking Pattern
The game uses **Mirror networking** with a **client-host or dedicated server** architecture:
- Server is authoritative for all game state
- `NetworkBehaviour` subclasses use `[SyncVar]` for state sync and `[Command]`/`[ClientRpc]` for RPC calls
- Custom transport layer: `CustomLiteNetLib4MirrorTransport` extends LiteNetLib4Mirror with rate limiting, challenge-response auth, IP passthrough, and ban handling

### Player Representation
Every connected player is represented by a **`ReferenceHub`** — the central aggregator component:

```csharp
public sealed class ReferenceHub : NetworkBehaviour
```

`ReferenceHub` holds references to all per-player subsystem components and provides static lookup methods:
- `ReferenceHub.AllHubs` — all connected players
- `ReferenceHub.LocalHub` — the local player
- `ReferenceHub.HostHub` — the server/host player
- `ReferenceHub.GetHub(GameObject)`, `TryGetHub(int playerId, ...)`, `TryGetHubNetID(uint, ...)`

### Scene Structure
- **NewMainMenu** — Main menu scene (server browser, settings, credits)
- **Facility** — Gameplay scene (the SCP containment facility)

---

## Core Systems

For detailed sub-bullets, SCP role class tables, and subsystem breakdowns, see CLAUDE-systems.md.

1. **Network Manager** — `CustomNetworkManager` (`CustomNetworkManager.cs`): server lifecycle, rate limiting, config loading, fast restart, reconnect. Singleton: `TypedSingleton`.
2. **Player Hub** — `ReferenceHub` (`ReferenceHub.cs`): central per-player aggregator; holds all subsystem component references (`CharacterClassManager`, `PlayerRoleManager`, `PlayerStats`, `Inventory`, etc.).
3. **Role System** — `PlayerRoleManager`/`PlayerRoleBase` (`PlayerRoles/`): role assignment, SCP/human role states, `RoleTypeId` enum, `OnServerRoleSet`/`RoleChanged` events.
4. **Stats System** — `PlayerStats` (`PlayerStatsSystem/`): modular HP/AHP/stamina/hume shield with per-damage-type handlers; `OnAnyPlayerDamaged`/`OnAnyPlayerDied` events.
5. **Inventory System** — `Inventory` (`InventorySystem/`): per-player items — firearms, armor, keycards, usables, throwables, MicroHID, radio; `ItemBase`/`ItemType` enum.
6. **Authentication** — `CentralAuthManager`/`CharacterClassManager` (`CentralAuth*.cs`): ECDSA/RSA token auth with Northwood central server on background thread (`_authThread`).
7. **Map Generation** — `SeedSynchronizer` (`MapGeneration/`): syncs `[SyncVar] int _syncSeed`, drives LCZ/HCZ/EZ zone generation, `RoomIdentifier` tagging, `OnMapGenerated` event.
8. **Round Management** — `RoundStart` (lobby timer, `RoundStart.singleton`) + `RoundSummary` (win condition, `LeadingTeam` enum): both NetworkBehaviours.
9. **Respawn System** — `RespawnManager` (`Respawning/`): token-driven wave respawn, phases `RespawnCooldown → SelectingTeam → PlayingEntryAnimations → SpawningSelectedTeam`.
10. **Alpha Warhead** — `AlphaWarheadController` (`AlphaWarheadController.cs`): NetworkBehaviour; nuke arm/countdown/detonate via `AlphaWarheadSyncInfo` SyncVar.
11. **Decontamination** — `DecontaminationController` (`LightContainmentZoneDecontamination/`): phase-based LCZ gas/door sequence with client timers.
12. **SCP-914** — `Scp914Controller` (`Scp914/`): NetworkBehaviour; item upgrade machine with knob settings and per-item-type upgrade processors.
13. **Interactables/Doors** — (`Interactables/`): `IInteractable`/`IClientInteractable`/`IServerInteractable` interfaces; door types (Basic/Breakable/Checkpoint/Elevator/Pryable); elevator system.
14. **Player Effects** — `PlayerEffectsController` (`CustomPlayerEffects/`): status effects — `Bleeding`, `Blinded`, `Burned`, `Corroding`, `Decontaminating`, `Asphyxiated`, `CardiacArrest`, `AmnesiaVision`, etc.
15. **Voice Chat** — (`VoiceChat/`): `VoiceChatMicCapture`, channel types (proximity/intercom/SCP/radio), `VcMuteFlags`/`VcPrivacyFlags`, codec/playback.
16. **Remote Admin** — (`RemoteAdmin/`): `QueryProcessor` per-player handler, `CommandProcessor`, encrypted comm via `RemoteAdminCryptographicManager`.
17. **Command System** — (`CommandSystem/`): `ClientCommandHandler`, `RemoteAdminCommandHandler`, `GameConsoleCommandHandler`; commands in `CommandSystem/Commands/`.
18. **Config System** — `ConfigFile`/`YamlConfig` (`GameCore/`): YAML config via YamlDotNet; files `config_gameplay.txt`, `config_remoteadmin.txt`, `hoster_policy.txt`.
19. **Security/Rate Limiting** — (`Security/`): `RateLimit`/`ServerRateLimit`/`PlayerRateLimitHandler` for connection and command rate limiting.
20. **Cryptography** — (`Cryptography/`): BouncyCastle wrappers — `AES`, `ECDH`, `ECDSA`, `RSA`, `Sha`, `Md`, `PBKDF2`.
21. **Friendly Fire** — `FriendlyFireHandler`/`FriendlyFireDetector` subclasses: per-player FF tracking, incident logging, `FriendlyFireConfig`.
22. **Hazards** — (`Hazards/`): `EnvironmentalHazard` base; `TantrumEnvironmentalHazard` (SCP-173), `SinkholeEnvironmentalHazard` (SCP-106).
23. **GameObjectPools** — (`GameObjectPools/`): `PoolManager`/`Pool<T>` generic pool, `IPoolResettable`/`IPoolSpawnable` interfaces.
24. **Server Console** — `ServerConsole` (`ServerConsole.cs`): singleton; server registration, public key fetch/cache, log output, whitelist, FF config flags.

---

## Key MonoBehaviour Classes

| Class | Type |
|---|---|
| `CustomNetworkManager` | NetworkManager |
| `ReferenceHub` | NetworkBehaviour |
| `CharacterClassManager` | NetworkBehaviour |
| `PlayerRoleManager` | NetworkBehaviour |
| `PlayerStats` | NetworkBehaviour |
| `Inventory` | NetworkBehaviour |
| `RoundStart` | NetworkBehaviour |
| `RoundSummary` | NetworkBehaviour |
| `AlphaWarheadController` | NetworkBehaviour |
| `RespawnManager` | MonoBehaviour |
| `DecontaminationController` | NetworkBehaviour |
| `Scp914Controller` | NetworkBehaviour |
| `TeslaGateController` | NetworkBehaviour |
| `SeedSynchronizer` | NetworkBehaviour |
| `ServerConsole` | MonoBehaviour |
| `SteamLobby` | MonoBehaviour |

See CLAUDE-systems.md for role descriptions.

---

## Code Conventions and Patterns

### Singleton Pattern
Most managers use either:
- Static `singleton` field set in `Awake()` (e.g., `RespawnManager.Singleton`, `RoundStart.singleton`, `ServerConsole.singleton`)
- `TypedSingleton` property casting Mirror's `NetworkManager.singleton` (e.g., `CustomNetworkManager.TypedSingleton`)

### Coroutines
MEC (More Effective Coroutines) is used almost exclusively over Unity coroutines:
```csharp
Timing.RunCoroutine(_MyCoroutine());
Timing.CallDelayed(seconds, () => { ... });
IEnumerator<float> _MyCoroutine() { yield return Timing.WaitForSeconds(1f); }
```

### Network Sync
- `[SyncVar]` for server-to-all-clients state
- `[SyncVar(hook = nameof(MyHook))]` for change callbacks
- `[Command]` for client-to-server RPCs
- `[ClientRpc]` for server-to-all-clients RPCs

### Events
Static C# events are preferred for cross-system communication:
```csharp
public static event Action<ReferenceHub> OnPlayerAdded;
public static event Action<SpawnableTeamType, List<ReferenceHub>> ServerOnRespawned;
```

### Configuration Access
```csharp
ConfigFile.ServerConfig.GetBool("key", defaultValue);
ConfigFile.ServerConfig.GetInt("key", defaultValue);
ConfigFile.ServerConfig.GetString("key", defaultValue);
```

### Player Lookup
```csharp
ReferenceHub.TryGetHub(gameObject, out ReferenceHub hub);
ReferenceHub.TryGetHub(playerId, out ReferenceHub hub);
ReferenceHub.TryGetHubNetID(netId, out ReferenceHub hub);
```

### Extension Methods
`Utils.NonAllocLINQ` — non-allocating LINQ alternatives used throughout to avoid GC pressure.
`NorthwoodLib.Pools` — object pooling from NorthwoodLib.

---

## Critical Files

| File | Importance |
|---|---|
| `Assets/Scripts/Assembly-CSharp/ReferenceHub.cs` | Central player component — understand this first |
| `Assets/Scripts/Assembly-CSharp/CustomNetworkManager.cs` | Server/client lifecycle, connection handling |
| `Assets/Scripts/Assembly-CSharp/CharacterClassManager.cs` | Authentication, user IDs, instance modes |
| `Assets/Scripts/Assembly-CSharp/GameCore/Version.cs` | Game version constants (Major=12, Minor=0, Revision=2) |
| `Assets/Scripts/Assembly-CSharp/GameCore/ConfigFile.cs` | Config loading entry point |
| `Assets/Scripts/Assembly-CSharp/PlayerRoles/PlayerRoleManager.cs` | Role system |
| `Assets/Scripts/Assembly-CSharp/PlayerRoles/RoleTypeId.cs` | All role identifiers |
| `Assets/Scripts/Assembly-CSharp/PlayerStatsSystem/PlayerStats.cs` | Stats/damage system |
| `Assets/Scripts/Assembly-CSharp/Respawning/RespawnManager.cs` | Wave respawn logic |
| `Assets/Scripts/Assembly-CSharp/MapGeneration/SeedSynchronizer.cs` | Map seed sync |
| `Assets/Scripts/Assembly-CSharp/LiteNetLib4Mirror/CustomLiteNetLib4MirrorTransport.cs` | Custom transport with auth/rate limiting |
| `Assets/Mirror/Core/Mirror.asmdef` | Mirror assembly root |
| `Packages/manifest.json` | Package dependencies |

---

## Warnings and Non-Obvious Setup

### Authentication System
- The game requires a **Northwood Central Server** public key to start in online mode. `ServerConsole.RunRefreshPublicKey()` must complete before `StartHost()` is called.
- The preauth challenge system (`CustomLiteNetLib4MirrorTransport.Challenges`) uses ECDSA-signed tokens. Without the correct keys, connections will fail silently.
- `CharacterClassManager.OnlineMode` must be `true` for Central Auth to be active. Offline mode skips user ID validation.

### Config Files
Server reads YAML configs from the app folder (`FileManager.GetAppFolder()`). Required files:
- `config_gameplay.txt` — main server config
- `config_remoteadmin.txt` — Remote Admin permissions (auto-created from `ConfigTemplates/config_remoteadmin.template.txt`)
- Optional: `hoster_policy.txt` — hosting provider policy

### GameFilesVersion Check
`CustomNetworkManager.GameFilesVersion` must equal `4` (the `_expectedGameFilesVersion` constant) or the server will refuse to start. This field must be set correctly in the prefab/inspector.

### MEC Coroutines
The project uses MEC exclusively. Standard Unity `StartCoroutine` / `IEnumerator` patterns are **not** used for main game logic. Use `Timing.RunCoroutine()` for new coroutines.

### Transport Layer
The active transport is `LiteNetLib4Mirror`. `CustomLiteNetLib4MirrorTransport` overrides it and handles:
- Rate limiting (IP and UserID windows)
- Preauth challenge-response
- IP passthrough for proxy servers
- Rejection suppression during attacks

### Steam Integration
`SteamManager` and `SteamLobby` handle Steamworks integration. `Facepunch.Steamworks.Win64.dll` is Windows-only. Linux dedicated server mode bypasses Steam.

### ParrelSync
`ParrelSync` is included for running multiple Unity Editor instances simultaneously (useful for testing client-server locally without building).

### CODES Directory
`CODES/ClientCode/` and `CODES/ServerCode/` contain decompiled reference code (via cpp2il/Il2CppDumper), not compiled into the project. This is reference material only.

**IMPORTANT — ClientCode format:** All files in `CODES/ClientCode/` are plain-text `.txt` files (not `.cs`). When comparing a restored script against references, **always check `CODES/ClientCode/` first** — it contains the decompiled client-side source for v12.0.2 and is the ground truth for client behaviour. `CODES/SCP13.0/` (ISIL) is the secondary reference. Use both together: ClientCode for exact v12 behaviour, ISIL for cleaner architecture and patterns to follow.

**IMPORTANT — beyond the .txt dumps, the ORIGINAL BUILD itself is readable.** When the pseudocode is corrupt or a constant/field mapping is in doubt, do NOT guess — read the real thing. Full recipe in **`CLAUDE-binary-recovery.md`**: `cpp2il_out/<Type>_metadata.txt` gives exact field offsets (decodes ISIL `[reg+N]` reads) and method File/Ram offsets; `CPP2IL_ISIL_Client/IsilDump/<Type>.txt` gives per-instruction logic; `Tools/ReadGameAssembly.ps1` reads any constant straight out of `D:\AssetToBaseStart12.0.2\Client\GameAssembly.dll` by the VA that appears in ISIL.

### Namespaces
The majority of game code is in the **global namespace** (no namespace declared). Key exceptions:
- `GameCore` — version, config, console, round start
- `PlayerRoles` — role system and all SCP/human role classes
- `PlayerStatsSystem` — stats and damage handlers
- `InventorySystem` — inventory and items
- `Respawning` — respawn management
- `MapGeneration` — map generation
- `RemoteAdmin` — admin interface
- `Security` — rate limiting
- `Cryptography` — crypto wrappers
- `VoiceChat` — voice system
- `Interactables` — interaction system
- `Mirror` — networking framework
- `Utils.NonAllocLINQ`, `Utils.Networking` — utilities

### Linear Color Space
Project uses **Linear color space** (`m_ActiveColorSpace: 1`), which affects shader and texture import settings. All textures must be imported with the correct color space setting.

---

## Event Flows

### (a) Player Death → Round Summary → Respawn Tokens

1. **Damage entry point** — any system calls `PlayerStats.DealDamage(DamageHandlerBase handler)` on the victim's `PlayerStats` component.
2. **God-mode guard** — if `CharacterClassManager.GodMode` is true, `DealDamage` returns false immediately.
3. **Role pre-processing** — if the current role implements `IDamageHandlerProcessingRole`, `ProcessDamageHandler(handler)` is called to allow the role to modify or replace the handler.
4. **Stat deduction** — `handler.ApplyDamage(ReferenceHub ply)` is called. In `StandardDamageHandler.ApplyDamage` the damage is absorbed in order: `HumeShieldStat` → `AhpStat` → `HealthStat`. Each stat's `CurValue` is decremented directly.
5. **Damage events fire** — `PlayerStats.OnAnyPlayerDamaged` (static) and `PlayerStats.OnThisPlayerDamaged` (instance) are raised.
6. **Death check** — if `ApplyDamage` returns `HandlerOutput.Death`, `PlayerStats.OnAnyPlayerDied` and `PlayerStats.OnThisPlayerDied` are raised, then `PlayerStats.KillPlayer(handler)` is called.
7. **KillPlayer side effects** — `RagdollManager.ServerSpawnRagdoll(_hub, handler)` spawns a ragdoll; `_hub.inventory.ServerDropEverything()` drops all items; `_hub.roleManager.ServerSetRole(RoleTypeId.Spectator, RoleChangeReason.Died)` transitions the player to `SpectatorRole`; the spectator role receives the handler via `spectatorRole.ServerSetData(handler)`.
8. **Round summary tracking** — `RoundSummary` subscribes to `PlayerStats.OnAnyPlayerDied` via `OnAnyPlayerDied`. Each death increments `RoundSummary.Kills`. If the killer is an SCP (detected via `AttackerDamageHandler.Attacker.Role` team check) or the cause is pocket decay (`DeathTranslations.PocketDecay`), `KilledBySCPs` is also incremented.
9. **Round end detection** — `RoundSummary._ProcessServerSideCode()` (MEC coroutine, `FixedUpdate` segment) polls every 2.5 seconds. When only one faction remains alive (`facilityForces`, `chaosInsurgency`, or `anomalies` count drops to ≤1), `_roundEnded = true`.
10. **Summary broadcast** — `RoundSummary.RpcShowRoundSummary(classlistStart, newList, leadingTeam, ...)` is sent to all clients via `[ClientRpc]`. The `LeadingTeam` enum (`FacilityForces`, `ChaosInsurgency`, `Anomalies`, `Draw`) is resolved from surviving player counts and escape stats.
11. **Round restart** — after the round-restart delay (`auto_round_restart_time` config, clamped 5–1000 s), `RpcDimScreen()` fades the screen, then `RoundRestarting.RoundRestart.InitiateRoundRestart()` is called.
12. **Respawn token adjustment on death** — `HumanTerminationTokens` subscribes to `PlayerStats.OnAnyPlayerDied` at startup (`[RuntimeInitializeOnLoadMethod]`). On each death, `HandleHomocide` or `HandleOtherMilitantDeath` calls `RespawnTokensManager.GrantTokens(SpawnableTeamType, float)` or `RespawnTokensManager.RemoveTokens(SpawnableTeamType, float)`. Example: killing a CI militant grants 1.5 tokens to `SpawnableTeamType.NineTailedFox`; a CI militant dying from non-enemy fire removes tokens scaled by `MilitantDiedPenalty (1.2) × numberOfRespawns`.
13. **Token-driven respawn team selection** — `RespawnManager.Update()` (server-only, guarded by `NetworkServer.active`) advances through `RespawnSequencePhase`: `RespawnCooldown → SelectingTeam → PlayingEntryAnimations → SpawningSelectedTeam`. At `SelectingTeam`, `RespawnTokensManager.DominatingTeam` returns whichever `SpawnableTeamType` holds the most tokens; `RespawnManager.Spawn()` then calls `PlayerRoleManager.ServerSetRole(newRole, RoleChangeReason.Respawn)` for each eligible spectator and raises `RespawnManager.ServerOnRespawned`.

---

### (b) Round Start → Map Generation → Decontamination

1. **Lobby countdown** — `RoundStart` (NetworkBehaviour, singleton `RoundStart.singleton`) initialises in `Awake`. On the server, `CharacterClassManager` runs the `_WaitForPlayers` MEC coroutine which counts down `Timer` (`[SyncVar]`) from the configured `lobby_waiting_time` (default 20 s).
2. **Round forced** — when the countdown reaches -1 or a player clicks the force-start button (calls `CmdForceRoundStart()` → `CharacterClassManager.ForceRoundStart()`): `CharacterClassManager.RoundStarted = true` (SyncVar), `CharacterClassManager.OnRoundStarted` event fires, and `RpcRoundStarted()` fires on all clients so they also raise `OnRoundStarted`.
3. **Role assignment** — `RoleAssigner` listens to the round-start event and distributes SCP and human roles via `ScpSpawner` and `HumanSpawner`, calling `PlayerRoleManager.ServerSetRole(...)` with `RoleChangeReason.RoundStart`.
4. **Seed sync** — `SeedSynchronizer` (NetworkBehaviour) sets `[SyncVar] int _syncSeed` on the server in `Start()` — either from the `map_seed` config key or from `UnityEngine.Random.Range(1, int.MaxValue)`. The SyncVar propagates to all clients automatically via Mirror.
5. **Map generation** — once `Seed > 0` and `!MapGenerated`, `SeedSynchronizer.Update()` calls `GenerateLevel()`. This iterates `ImageGenerator.ZoneGenerators` (LCZ, HCZ, EZ) and calls `generator.GenerateMap(_syncSeed - i, zoneAlias, out blackbox)` for each zone. After all zones generate, `DoorSpawnpoint.SetupAllDoors()` places doors (server-only).
6. **Room ID assignment** — after generation, all `RoomIdentifier` components in the scene have `TryAssignId()` called; any that fail are removed from `RoomIdentifier.AllRoomIdentifiers`.
7. **OnMapGenerated event** — `SeedSynchronizer.OnMapGenerated` (static event, `Action`) fires once; `MapGenerated = true` and the stopwatch starts for `TimeSinceMapGeneration`.
8. **Decontamination timer activation** — `DecontaminationController.ServersideSetup()` is called each `Update()`. It waits until `RoundStart.singleton.Timer == -1` (round is started), then sets `NetworkRoundStartTime = NetworkTime.time` (a `[SyncVar]`, propagated to clients). The timer offset formula is `GetServerTime = NetworkTime.time - RoundStartTime + TimeOffset`.
9. **Phase progression** — `DecontaminationController.UpdateTime()` runs each frame. It checks `serverTime > DecontaminationPhases[_nextPhase].TimeTrigger` (phases are configured in the Inspector in minutes, converted to seconds on the server in `Start()`). Phases advance through announcements: `DecontaminationStart`, `DecontaminationMinutes (10 min)`, `DecontaminationMinutes (5 min)`, `Decontamination1Minute`, `DecontaminationCountdown`.
10. **Checkpoint doors open** — a phase with `PhaseFunction.OpenCheckpoints` triggers `DoorEventOpenerExtension.TriggerAction(OpenerEventType.DeconEvac)`.
11. **Final decontamination** — when the `PhaseFunction.Final` phase is reached, `FinishDecontamination()` is called: `DoorEventOpenerExtension.TriggerAction(OpenerEventType.DeconFinish)` opens/seals LCZ doors; `DisableElevators()` locks all `ElevatorDoor` components in LCZ zone; `DecontaminationGas.TurnedOn = true`; and the `KillPlayers()` MEC coroutine starts, applying the `Decontaminating` status effect (via `PlayerEffectsController.EnableEffect<Decontaminating>()`) to any alive player whose Y position is between -200 and 200 (LCZ bounds).
12. **Status effect damage** — the `Decontaminating` effect (in `CustomPlayerEffects/`) periodically calls `PlayerStats.DealDamage` on the affected player, completing the kill loop back through the death chain.

---

### (c) SCP Ability Activation → Damage → Stats Update (SCP-096 example)

1. **Key detection (client)** — `Scp096AttackAbility` extends `ScpKeySubroutine<Scp096Role>` with `TargetKey = ActionName.Shoot`. In `OnKeyDown()`, if `AttackPossible` (SCP is in `Scp096RageState.Enraged` and `Scp096AbilityState.None`) and `_clientAttackCooldown.IsReady`, the attack proceeds.
2. **Backtracker payload** — client gathers nearby human players within `BacktrackingDisSqr` (3 units radius), packages their positions as `RelativePosition` values, and calls `ClientSendCmd()` which serialises everything via `ClientWriteCmd(NetworkWriter)`. The `[Command]` sends the data to the server.
3. **Server validation** — `ServerProcessCmd(NetworkReader reader)` checks `_serverAttackCooldown.TolerantIsReady` and `AttackPossible`. It re-creates `FpcBacktracker` objects for the SCP and each nearby player to rewind positions for lag compensation.
4. **ServerAttack** — `ServerAttack()` sets `Scp096AbilityState.Attacking`, creates a `Scp096HitHandler` for the chosen hand (left/right), and calls `scp096HitHandler.DamageSphere(position, radius)` using a sphere overlap at `PlayerCameraReference.position + forward * _sphereHitboxOffset`.
5. **Hit processing** — `Scp096HitHandler.DamageSphere` calls `Physics.OverlapSphereNonAlloc` on the `Hitbox/Door/Glass` layer mask. For each `IDestructible` hit, a line-of-sight check (`Physics.Linecast` on solid layers) filters occluded targets. For human hitboxes (`HitboxIdentity`), damage is `85f` for targets tracked by `Scp096TargetsTracker`, or `0f` for non-targets.
6. **DamageHandler construction** — `Scp096HitHandler.DealDamage` creates `new Scp096DamageHandler(scpRole, damage, attackType)` where `Scp096DamageHandler` extends `AttackerDamageHandler`. It stores an `Attacker` `Footprint` (the SCP's `ReferenceHub`).
7. **IDestructible.Damage call** — `target.Damage(dmg, handler, centerOfMass)` is called. For `HitboxIdentity`, this routes to `PlayerStats.DealDamage(handler)` on the victim hub.
8. **Stat deduction** — `Scp096DamageHandler.ApplyDamage` calls `base.ApplyDamage` (`StandardDamageHandler.ApplyDamage`): damage first absorbed by `HumeShieldStat.CurValue`, then `AhpStat.CurValue`, then `HealthStat.CurValue`. Each stat's `CurValue` is a property on `SyncedStatBase`; changes are flagged dirty and replicated to the local player and spectators via `SyncedStatMessages`.
9. **Knockback** — after calling base, `Scp096DamageHandler.ApplyDamage` sets `StartVelocity` (direction × 7, Y = 3.5 for slap; forward × 10, Y = -10 for charge). This is serialised into the ragdoll on death.
10. **Death or damage events** — if `HealthStat.CurValue <= 0`, `HandlerOutput.Death` propagates back to `PlayerStats.DealDamage`, fires `OnAnyPlayerDied`, calls `KillPlayer`, spawns a ragdoll, and transitions the victim to `SpectatorRole` (see chain (a) above).
11. **Hit result sync** — `_hitResult` (a `Scp096HitResult` flags enum) is written by `ServerWriteRpc` and read by `ClientProcessRpc` on all clients. Clients play the appropriate audio (`_killSound`, `_hitClipsHuman`, `_hitClipsObjects`) and show the hitmarker via `Hitmarker.PlayHitmarker(1f)`.
12. **Cooldown reset** — `_serverAttackCooldown.Trigger(DefaultAttackCooldown)` (0.5 s) is started. Once elapsed, `Scp096StateController.ResetAbilityState()` clears `Scp096AbilityState.Attacking` so the next slap can occur.

---

## Asset-Driven Data

The project does **not** use ScriptableObjects for core gameplay data; all per-role and per-weapon numeric values are hardcoded. Several ScriptableObjects drive visual/audio/config concerns (`FirearmGlobalSettingsPreset`, `ModelSharedSettings`, `DoorAudioSettings`, `HeadlessRuntime`, etc.); YAML config drives all gameplay tuning at runtime. See CLAUDE-systems.md for the full ScriptableObject table and non-SO data-driven system details.

---

## Platform & Build

### Conditional Compilation

There are **no** `#if UNITY_SERVER`, `#if DEDICATED_SERVER`, or `#if CLIENT` / `#if !UNITY_SERVER` preprocessor directives in `Assets/Scripts/Assembly-CSharp/`. All server/client separation is done at **runtime** using Mirror's API and the custom `ServerStatic.IsDedicated` flag.

### Runtime Server/Client Branching

The project uses two primary runtime guards:

**`NetworkServer.active`** — guards all server-authoritative logic. Examples:
- `SeedSynchronizer.Start()` — only generates/sets seed on the server.
- `DecontaminationController.ServersideSetup()` / `FinishDecontamination()` — phase advancement and gas/door control are server-only.
- `RoundSummary._ProcessServerSideCode()` — win condition checks and `RpcShowRoundSummary` only called from server.
- `RespawnManager.Update()` — entire respawn state machine gated by `NetworkServer.active`.
- `AlphaWarheadController` — all arm/detonate/auto-detonate paths gated by `NetworkServer.active`.
- `CharacterClassManager.ForceRoundStart()` — returns false immediately if not server.
- `PlayerStats.DealDamage` — called on the server; damage events fire server-side only (clients receive effects via SyncVars/RPCs).

**`NetworkClient.active`** — guards client-only visual/audio logic. Examples:
- `BloodEffectsSystem` — all blood particle spawning gated by `NetworkClient.active` (no blood on headless server).

**`isLocalPlayer`** — guards local-player-only UI and input:
- `PlayerStats.Start()` — subscribes to `PlayerRoleManager.OnRoleChanged` only for the local player.
- `RoundStart.Update()` — skips all UI update if `ServerStatic.IsDedicated`.
- `CharacterClassManager.Start()` — auth token sending, local console setup, Steam/Discord init gated by `isLocalPlayer`.
- `Broadcast.Awake()` — overlay UI activation gated by `isLocalPlayer`.
- `AspectRatioSync` — aspect ratio command only sent if `isLocalPlayer`.

**`isServer`** (Mirror NetworkBehaviour property):
- `AdminToyBase.Start()` — spawn registration only on server.
- `AmbientSoundPlayer.Start()` — dual-role host check uses `isLocalPlayer && isServer`.
- `CharacterClassManager.Start()` — `CentralAuthInterface` construction passes `isServer` as the server-side flag.

**`ServerStatic.IsDedicated`** — used to distinguish a headless dedicated server from a listen-server/client:
- `RoundStart.Update()` — skips the entire lobby UI update block when `IsDedicated`.
- `CharacterClassManager.Start()` — skips Steam/Discord init on dedicated server.
- `CharacterClassManager` sets `UserId = "ID_Dedicated"` for the dedicated server's own hub, or `"ID_Host"` for listen-server.
- `SoftRestartCommand` / `StopNextRoundCommand` — refuse to run if not dedicated.
- `CentralServer` — registration with Northwood master server only runs on dedicated.
- `CentralServerKeyCache` — public key caching path differs for dedicated vs host.

### Server-Only Systems

The following classes perform meaningful work only when `NetworkServer.active` or `ServerStatic.IsDedicated` is true and are effectively no-ops on pure clients:

| System | Class(es) |
|---|---|
| Round management | `RoundSummary._ProcessServerSideCode`, `CharacterClassManager._WaitForPlayers`, `CharacterClassManager.ForceRoundStart` |
| Respawn | `RespawnManager.Update`, `RespawnManager.Spawn`, `RespawnTokensManager.ModifyTokens` |
| Damage application | `PlayerStats.DealDamage`, `PlayerStats.KillPlayer`, `StandardDamageHandler.ApplyDamage` |
| Map generation (seed) | `SeedSynchronizer.Start` (seed assignment), `SeedSynchronizer.GenerateLevel → DoorSpawnpoint.SetupAllDoors` |
| Decontamination | `DecontaminationController.ServersideSetup`, `FinishDecontamination`, `KillPlayers` |
| Player effects (server tick) | `Bleeding.Update`, `Asphyxiated.Update`, `Corroding.Update`, `Decontaminating` tick |
| Authentication | `CentralAuthInterface` (server-side token verification path), `CustomLiteNetLib4MirrorTransport` preauth |
| Server registration | `CentralServer`, `ServerConsole.RunRefreshPublicKey` |
| AFK enforcement | `AFKManager` — kick logic gated by `NetworkServer.active` |
| Achievements | `AchievementHandlerBase` — award logic gated by `NetworkServer.active` |

### Client-Only Systems

| System | Class(es) |
|---|---|
| Blood effects | `BloodEffectsSystem` — all particle spawning gated by `NetworkClient.active` |
| Death animations | `BlurBlackDeathAnimation`, `HeadSpin`, `FirstpersonDeathAnimation` — triggered client-side on `isLocalPlayer` or spectating |
| HUD / UI | `RoundStart` lobby UI, `RoundSummary._ShowRoundSummary`, `UserMainInterface`, `HintDisplay`, `HideHUDController` |
| Voice capture | `VoiceChatMicCapture` — microphone capture only runs on the local player |
| Hitmarker | `Hitmarker.PlayHitmarker` — only called when `isLocalPlayer` or spectating via `SpectatorNetworking.IsLocallySpectated` |

### Dedicated Server Build Pipeline

There is **no custom `BuildPlayerOptions` script** or dedicated build pipeline in `Assets/Scripts/Assembly-CSharp/`. The dedicated server is built and run in Unity's standard **batch mode** (`-batchmode -nographics`). The `HeadlessRuntime` ScriptableObject (`HeadlessRuntime.cs`) provides a profile asset with framerate limit and camera/console flags for headless operation. `ServerStatic.IsDedicated` is set at runtime by detecting batch mode or a command-line flag rather than a compile-time symbol. Linux dedicated server support is noted in the project overview and bypasses Steam (Steamworks integration is Windows-only via `Facepunch.Steamworks.Win64.dll`).

---
> Source: [Siteforge-Collective/SCP-SL-Community-Edition](https://github.com/Siteforge-Collective/SCP-SL-Community-Edition) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
