## mapleclaude

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**MapleClaude** is a brand-new, modernized 64-bit MapleStory v95 game client written in C# 13 / .NET 10. It is greenfield — not a fork of any existing client — and is built to connect to the Kinoko Java server (v95, locale 8, patch `"1"`). The client speaks the standard v95 wire protocol (Shanda + Maple-AES + IGCipher IV rotation) and reads standard GMS v95 WZ files for visuals.

The codebase is organized for long-term, multi-phase extension. Phase 1 delivers the full pre-game flow: launch → title → login → server/world select → character list → character create → PIN entry → channel migrate handoff. Phase 2 picks up at field load. See `docs/roadmap.md` for the full roadmap.

## Build & Run

```powershell
dotnet restore
dotnet build -c Debug
dotnet test
dotnet run --project src/MapleClaude
```

Or open `MapleClaude.slnx` in Visual Studio 2026.

Runtime requires a running Kinoko login server (defaults to `127.0.0.1:8484`) and a directory of standard v95 WZ files. See `README.md` for the environment variables.

### Hot-reload dev loop (no close/build/deploy/reopen)

```powershell
.\watch.ps1
```

Runs the client under `dotnet watch run` (from `bin/`, not the deployed single-file exe). On save,
**method-body edits** (Draw/Update/layout logic) apply live via .NET Hot Reload; **structural edits**
(new/changed fields, field initializers, signatures, types) are "rude edits" → `dotnet watch`
auto-rebuilds and relaunches. Either way there's no manual cycle. The script finds the WZ folder
from `MAPLECLAUDE_WZ_DIR`, then `MAPLECLAUDE_DEPLOY_DIR`, then `.deploy.local` (the deploy folder
*is* the WZ folder) and passes `-p:NoAutoPublish=true -p:NoAutoDeploy=true` so each rebuild skips
the single-file publish.

`watch.ps1` also sets `MAPLECLAUDE_DEBUG=1`, which opens the **live layout overlay**: tick "drag"
and drag a registered position knob (e.g. the `CharCreate` panels/fields) to tune layout with
**zero rebuild**, then bake the value into the default. Any non-empty `MAPLECLAUDE_DEBUG` enables it.

### Single-file `.exe` is the default build output

**Every `dotnet build` (and every F5 in Visual Studio) auto-produces a single self-contained `MapleClaude.exe`** at:

```
artifacts/<Configuration>/MapleClaude.exe
```

The .NET runtime, MonoGame, and all native dependencies are bundled. Drop the `.exe` next to your `UI.wz` / `Map.wz` and run it.

Size: **~73 MB** for both Debug and Release. The single-file bundler's compression and `IncludeAllContentForSelfExtract` options are intentionally **off** — both trigger `STATUS_STACK_BUFFER_OVERRUN (0xc0000409)` on launch when combined with MonoGame's native deps under .NET 10. Do not re-enable them without a verified fix upstream.

First launch takes 30–90 s while Windows Defender scans the binary; subsequent launches are sub-second. Add the deploy folder to Defender exclusions to skip the scan.

The MSBuild target `AutoPublishSingleFile` in `src/MapleClaude/MapleClaude.csproj` hooks `AfterTargets="Build"` and invokes the standard `Publish` target. It auto-skips for:
- Visual Studio design-time builds (IntelliSense passes).
- `dotnet test` (the test projects don't reference `MapleClaude.csproj`, so the target never fires).
- Anything passing `-p:NoAutoPublish=true`.

Escape hatch for the fastest possible iteration:

```powershell
dotnet build -c Debug -p:NoAutoPublish=true      # ~3 s; skips publish, multi-file output only
```

`PublishTrimmed` and `PublishAot` are explicitly disabled because MonoGame relies on reflection (content pipeline, type lookup) that breaks under trimming.

### Auto-deploy on every build (MapleStory folder)

If you want every `dotnet build` (and every Rebuild Solution in VS2026) to also **drop `MapleClaude.exe` into your MapleStory folder**, configure a deploy directory once. The MSBuild target `AutoDeploySingleFile` runs right after `AutoPublishSingleFile` and copies the freshly built exe.

Resolution order:

1. **Env var** `MAPLECLAUDE_DEPLOY_DIR` (preferred — visible to any tool, persists across projects).
   ```powershell
   [Environment]::SetEnvironmentVariable('MAPLECLAUDE_DEPLOY_DIR', 'X:\path\to\maplestory', 'User')
   ```
   Then **restart Visual Studio** so the new VS process inherits the env var.

2. **`.deploy.local` file** at the repo root containing the path on a single line (fallback — no restart needed, easier per-machine setup).
   ```powershell
   "X:\path\to\maplestory" | Set-Content .deploy.local -Encoding ascii
   ```

Both options are gitignored. If neither is set, the deploy step prints a skip hint and continues. To disable for one build: `dotnet build -p:NoAutoDeploy=true`.

`publish.ps1` is still around for explicit Release + deploy from the CLI, but it's no longer required since `dotnet build` does the same work automatically.

## Repository Layout

```
src/
  MapleClaude/         main executable (MonoGame entry, App/, Stages/, UI/, Platform/)
  MapleClaude.Net/     Crypto/, Packet/, Session/, Handlers/
  MapleClaude.Wz/      WzPackage, WzReader, WzImage, WzCanvas, WzCrypto
  MapleClaude.Render/  MonoGame integration (SpriteAtlas, WzTextureLoader, Camera2D)
  MapleClaude.Domain/  POD state types — no dependencies
tests/
  MapleClaude.Net.Tests/
  MapleClaude.Wz.Tests/
tools/
  gen-opcodes/         Kinoko In/OutHeader.java → OpCodes.cs codegen
  pcap-replay/         offline packet replay through PacketCipher
docs/
  protocol.md          wire protocol, header XOR, cipher composition
  wz-format.md         GMS v95 PKG1 reader notes
  architecture.md      module dependency graph
  roadmap.md           phases 1..N
.claude/
  skills/              auto-trigger skills (see table below)
  agents/              named agents (see table below)
```

## Architecture

### Application loop & stages

`MapleClaudeGame` (a `Microsoft.Xna.Framework.Game` subclass) owns the MonoGame loop. It holds an `IStageDirector` (stack-of-stages) and pushes the initial `TitleStage`. Stages implement `OnEnter / OnExit / Update / Draw / OnPacket` and request transitions via the director. Stage transitions cross-fade through the renderer.

### Network pipeline

```
Socket -> PipeReader -> framed [header(4) | body] -> PacketCipher.Decrypt -> InPacket -> PacketRouter -> IPacketHandler
                                                                  ^
                                                          IgCipher.InnoHash(iv)   (per packet, both directions)

IPacketHandler / ClientSession.Send -> OutPacket -> PacketCipher.Encrypt -> PipeWriter -> Socket
```

The session begins with one **unencrypted** handshake packet from the server: `short version=95`, length-prefixed `string patch="1"`, `byte[4] sendIv`, `byte[4] recvIv`, `byte locale=8`. From there every packet is wrapped:

- Header (4 bytes): `rawSeq = (iv[2] | iv[3]<<8) ^ (0xFFFF - 95)` (LE short); `dataLen = payloadLen ^ rawSeq` (LE short).
- Body **encrypt**: Shanda → MapleCrypto (AES-128 ECB with expanded IV; 1456-byte block boundaries).
- Body **decrypt**: MapleCrypto → Shanda.
- After every packet: `IgCipher.InnoHash(iv)` rotates the appropriate IV.

### Rendering pipeline

MonoGame `GraphicsDevice` (DirectX 11 backend) drives the loop. `SpriteAtlas` packs WZ canvas BGRA buffers into `Texture2D` atlases; `WzTextureLoader` caches lazily with an LRU. Stages compose by drawing widgets through a `Renderer` facade.

### WZ asset pipeline

`WzPackage.Open(path)` memory-maps a `.wz` file, verifies the PKG1 header and version hash, and exposes a node tree (`WzDirectory` → `WzImage` → `WzProperty` / `WzCanvas` / `WzSound` / `WzUol`). Paths resolve like `Login.img/Title/Logo`. Strings are AES-decrypted via `WzCrypto`; canvases are decompressed (LZ4 / Deflate depending on type) then AES-decrypted into raw BGRA.

## Server protocol

See `docs/protocol.md` for the full reference. The authoritative source is the upstream Kinoko server repository at <https://github.com/iw2d/kinoko>. Relative paths below are inside that repo:

- Opcodes: `src/main/java/kinoko/server/header/InHeader.java` and `OutHeader.java`.
- Login flow: `src/main/java/kinoko/handler/stage/LoginHandler.java` + `src/main/java/kinoko/packet/stage/LoginPacket.java`.
- Migration: `src/main/java/kinoko/handler/stage/MigrationHandler.java`.
- Cipher: `src/main/java/kinoko/util/crypto/{MapleCrypto,ShandaCrypto,IGCipher}.java`.
- Channel handshake: `src/main/java/kinoko/server/netty/{PacketChannelInitializer,PacketEncoder,PacketDecoder}.java`.

## Cipher cheat-sheet

```
header (4 bytes, LE):
    rawSeq  = (iv[2] | (iv[3] << 8)) ^ (0xFFFF - GAME_VERSION)
    dataLen = payloadLen ^ rawSeq

encrypt(body, iv):
    body = ShandaCrypto.Encrypt(body)
    body = MapleCrypto.Crypt(body, iv)   // AES-128 ECB with expanded IV per 1456 bytes
    iv   = IgCipher.InnoHash(iv)

decrypt(body, iv):
    body = MapleCrypto.Crypt(body, iv)
    body = ShandaCrypto.Decrypt(body)
    iv   = IgCipher.InnoHash(iv)
```

`GAME_VERSION = 95`, `LOCALE = 8`, `PATCH = "1"`.

## Opcode table

`src/MapleClaude.Net/Packet/OpCodes.cs` is generated by `tools/gen-opcodes` from the Kinoko `InHeader.java` / `OutHeader.java`. Re-run after pulling Kinoko changes.

A few opcodes that turn up often (v95 values, authoritative — the source of truth is the upstream Kinoko `InHeader.java` / `OutHeader.java`):

| Opcode | Value | Direction |
| ------ | ----- | --------- |
| `CheckPassword` | 1 | C→S |
| `WorldInfoRequest` | 4 | C→S |
| `SelectWorld` | 5 | C→S |
| `WorldRequest` | 11 | C→S |
| `SelectCharacter` | 19 | C→S |
| `MigrateIn` | 20 | C→S |
| `CheckDuplicatedID` | 21 | C→S |
| `CreateNewCharacter` | 22 | C→S |
| `DeleteCharacter` | 24 | C→S |
| `AliveAck` | 25 | C→S |
| `UserMove` | **44 (0x2C)** | C→S |
| `UserMeleeAttack` | 47 | C→S |
| `CheckPasswordResult` | 0 | S→C |
| `CheckPinCodeResult` | 6 | S→C |
| `WorldInformation` | 10 | S→C |
| `SelectWorldResult` | 11 | S→C |
| `SelectCharacterResult` | 12 | S→C |
| `CheckDuplicatedIDResult` | 13 | S→C |
| `CreateNewCharacterResult` | 14 | S→C |
| `MigrateCommand` | 16 | S→C |
| `AliveReq` | 17 | S→C |
| `SetField` | **141 (0x8D)** | S→C |
| `UserEnterField` | 179 | S→C |
| `UserMove` (remote) | 210 | S→C |

All Maple wire strings are length-prefixed LE-short + US-ASCII (not UTF-16LE). All multi-byte primitives are little-endian.

## Adding a new packet handler

Use the `server-packet-mirror` skill for server→client opcodes and the `client-packet-author` skill for client→server. The skills walk through:

1. Locate the opcode definition in the Kinoko header file.
2. Find the matching packet builder / request handler in Kinoko.
3. Generate a C# decoder/encoder skeleton in `MapleClaude.Net/Handlers/` mirroring the field order.
4. Wire it into `PacketRouter` (and, for outgoing, into the originating `Stage`).
5. Write a round-trip test with a captured byte fixture.

## Coding style

- File-scoped namespaces (see `.editorconfig`).
- Nullable enabled, treated-as-errors.
- `var` for built-ins and apparent types; explicit type otherwise.
- Private fields are `_camelCase`.
- No exceptions on hot paths (cipher, packet framing). Use `bool TryParseXxx`.
- `[StructLayout(LayoutKind.Sequential, Pack = 1)]` only for documented wire structs.
- `System.IO.Pipelines` for streaming reads/writes; never block the MonoGame Update thread on I/O.

## Testing

`dotnet test` from the repo root. Cipher tests use known vectors so they catch byte-level regressions early. Add a fixture in `tests/MapleClaude.Net.Tests/Fixtures/` whenever you add a new packet handler.

## Git workflow

**Never commit on `master`.** All work happens on `phase-N/<slug>` branches; master only receives merge commits from completed phase PRs.

**Always ask the user for permission before each commit.** Auto-committing is forbidden. When a cohesive unit is ready, propose the staged files and the commit message and wait for explicit approval before running `git commit`.

- Cut a branch from master at session start (`phase-1/cipher`, `phase-1/login-stage`, etc.).
- When a cohesive unit lands (file compiles, test passes, opcode decodes, stage renders): propose the commit; **wait for user approval**; then commit.
- Commit message: `phase-N(scope): imperative subject` plus a body explaining the *why*.
- When the phase reaches its definition-of-done, propose opening a PR to master via `gh pr create` with sections: **Summary**, **Test plan**, **Risk assessment** (protocol / cipher / asset / future-compat), **Recommendation** (`ACCEPT` / `ACCEPT WITH CHANGES` / `HOLD`). Wait for approval before opening.

The Phase 1 definition-of-done: log `MigrateIn ACK observed — Phase 2 boundary reached` after the channel server's first decrypted packet.

## Privacy & local references

**Never** reference machine-local absolute paths, machine-specific identifiers, or any private reference material in committed files (source, docs, skills, agents, commit messages, PR bodies). Anything machine-local goes in `CLAUDE.local.md` (gitignored). Before any commit, run `git diff --cached` and visually scan for absolute paths, drive letters, user folders, or private project names; abort the commit if any are present.

Public references that ARE allowed in committed files:
- Public GitHub repository URLs (e.g., <https://github.com/iw2d/kinoko>).
- Relative paths inside a public repository (e.g., `src/main/java/kinoko/util/crypto/IGCipher.java`).
- Environment variable names that the user sets locally (e.g., `MAPLECLAUDE_WZ_DIR`).

## Phase roadmap

See `docs/roadmap.md` and `README.md` for the table.

## Claude skills

| Skill | Triggers on | Owns |
| ----- | ----------- | ---- |
| `server-packet-mirror` | Adding a server→client opcode | Locate Kinoko `OutHeader` entry + builder, generate C# decoder skeleton |
| `client-packet-author` | Adding a client→server opcode | Locate Kinoko `InHeader` entry + handler, generate C# encoder skeleton |
| `wz-reader` | Edits under `src/MapleClaude.Wz/` or any GMS WZ-format question | Cross-reference against `kinoko/provider/wz/*.java`; UOL, version-hash, AES string crypt |
| `opcode-sync` | Kinoko `In/OutHeader.java` changed | Run `tools/gen-opcodes`, report any renames/insertions |
| `crypto-validator` | Edits under `src/MapleClaude.Net/Crypto/` | Ensure C# pipeline is byte-identical to Java reference via known-vector tests |
| `disasm-lookup` | Question about an original v95 client function | Resolve via local-only paths from `CLAUDE.local.md`; never print the path |
| `idb-bind` | Need the live `ida-pro/idalib` MCP but it may be busy/disconnected | Bind the `.i64`, background-wait when locked, copy-to-temp fallback, alignment check |
| `wz-subsystem-research` | "Research Item.wz / Skill.wz / Map.wz / Quest.wz / etc." — full subsystem sweep (20+ fields) | Kinoko `*Template.java` + IDB `CXxx*` cross-reference, structured report → typed C# model + `docs/<x>-wz.md` |
| `ida-lookup` | "Look into the IDB", "reference the IDB", "look up CXxx", "decompile CUI[X]" — single-symbol IDB query | `idb-bind` → `lookup_funcs`/`decompile`/`xrefs_to` → paraphrased answer, no `.i64` path leak |
| `authentic-ui-rebuild` | "Not the custom overlay, the authentic v95 X", "rebuild [UI] from WZ+IDB" | WZ subtree + `CUI*` decompile + coordinate extraction; escalates to `v95-ui-rebuilder` / `ui-origin-finder` agents |
| `ingame-feature` | "Implement [combat / death / drop / pickup / NPC click / emotion / ...]", "mobs should X" — full feature end-to-end | Coordinates packets + IDB animation + WZ assets + runtime; delegates to packet/WZ/audio skills |
| `wz-audio-bind` | "Play [X] sound", "BGM from Sound.wz", "mob sounds on hit" | Locate Sound.wz node → wire through `src/MapleClaude/Audio/` MonoGame XAudio2 service |
| `ship-pr` | "Commit, push, PR, merge", "open a PR", "ship it" | Sequenced commit → push → PR → merge with privacy guard + mandatory per-commit approval |
| `autonomous-phase` | "Continue with the next phase", "auto-implement the rest", "im going to bed" | Read `docs/roadmap.md` → plan → implement → ship via `ship-pr` → loop (commit-ask still required) |
| `layout-tune` | "Move it 3px down", "icon too high", "position is wrong" | Redirect to live overlay (`watch.ps1` + `MAPLECLAUDE_DEBUG=1`); else single-constant code edit |

## Claude agents

| Agent | When to launch | Loads |
| ----- | -------------- | ----- |
| `maple-cipher-expert` | Deep dive on cipher or desync bug | `kinoko/util/crypto/*.java`, `PacketEncoder.java`, `PacketDecoder.java`, `src/MapleClaude.Net/Crypto/**` |
| `kinoko-protocol-mirror` | Implementing a new flow end-to-end | The `OutHeader` entry, packet builder, in-game handler, plus a trace through `kinoko/world/**` |
| `monogame-renderer` | Anything in `src/MapleClaude.Render/**` or shaders | `MapleClaudeGame`, `SpriteAtlas`, `WzTextureLoader`, MonoGame docs |
| `v95-ui-rebuilder` | Reconstructing a UI screen | The relevant `UI/Login.img` subtree + reference flow via `disasm-lookup` |
| `migration-debugger` | Channel handshake breaks | `MigrationHandler.java`, `LoginPacket.selectCharacterResultSuccess`, `ClientSession`, `PacketCipher`, `MigrationCoordinator` |

---
> Source: [MujyKun/MapleClaude](https://github.com/MujyKun/MapleClaude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
