## nt-mt5-grpc

> Goal: Give AI agents the minimum, project-specific context to work productively in this repo.

# Copilot Instructions for NT-MT5 gRPC

Goal: Give AI agents the minimum, project-specific context to work productively in this repo.

## Architecture
- NT Addon ↔ BridgeApp (Go/Wails, pure gRPC) ↔ MT5 EA (MQL5 via DLL)
- Flow: NT trade → Bridge queue → MT5 hedge; MT5 hedge close → Bridge → NT close original
- Proto source of truth: `BridgeApp/proto/trading.proto` (synced to NT and MT5 folders)

## Key dirs/files
- Bridge server: `BridgeApp/internal/grpc/server.go` (+ `conversions.go`)
- Bridge app state/queue: `BridgeApp/app.go` (env `BRIDGE_GRPC_PORT`, default 50051)
- NT Addon gRPC client: `MultiStratManagerRepo/External/NTGrpcClient/` (public `TradingGrpcClient`)
- MT5 EA: `MT5/ACHedgeMaster_gRPC.mq5`; gRPC DLLs in `MT5/MT5GrpcClient/` and `MT5/cpp-grpc-client/`

## Contracts (examples)
- SubmitTrade: JSON → Trade proto with keys like `id, base_id, timestamp, action, quantity, price, total_quantity, contract_num, order_type, instrument, account_name, nt_*`
- Stream trades: Bridge polls queue every ~200ms and streams to MT5 (GetTrades)
- MT5 → Bridge: `MT5TradeResult`, `HedgeCloseNotification`
- Health: `HealthRequest{source}` → `HealthResponse{status, queue_size, net_position, hedge_size}`

## Conventions
- Bridge runs gRPC-only by default; no HTTP unless explicitly re-enabled elsewhere
- Edges use JSON; Bridge converts JSON↔proto (see `conversions.go` and JSON marshal in `app.go`)
- NT often keeps `useGrpc` feature flag with HTTP fallback still wired
- Queue is buffered (~100); stream buffers may drop on overflow (logged)

## Build/run (PowerShell)
- Bridge: from `BridgeApp/` → `make install-tools` (first time); `make proto`; `wails dev`; override port with `$env:BRIDGE_GRPC_PORT="50052"; wails dev`
  - **After making changes to Bridge code**: Use `./run-wails-dev-with-logs.ps1` from repo root for development with enhanced logging
  - **Build standalone Bridge server**: `cd 'c:\Documents\Dev\OfficialFuturesHedgebotv2\BridgeApp'; go build -o bridge-server.exe .`
- NT: build/use `External/NTGrpcClient.dll`; ensure .NET 4.8 and add `netstandard.dll` ref if CS0012 occurs (see gRPC progress docs)
- MT5: Build the native C++ gRPC client using your native dev prompt only. After any MT5-related changes, you MUST run `MT5/cpp-grpc-client/build_cpp_grpc.bat` to rebuild and deploy the DLLs to `MQL5\Libraries`. Do NOT use the PowerShell `build.ps1` for MT5 builds.

### NinjaTrader deploy script (recommended)
- Use `scripts/deploy-nt-to-ninjatrader.ps1` to build the NT gRPC client DLLs and deploy both DLLs and NinjaScript source `.cs` files for the MultiStratManager addon to their correct NinjaTrader folders.
- What the script does:
	- Builds `External/NTGrpcClient` for net48 by default.
	- Copies runtime DLLs (`NTGrpcClient.dll`, `Grpc.Core*.dll`, `Google.Protobuf.dll`, `System.Text.Json.dll`, etc.) into NinjaTrader roots under `bin/Custom` and `AddOns/MultiStratManager/External`.
	- Deploys ONLY top-level NinjaScript sources from `MultiStratManagerRepo/*.cs` into `...\NinjaTrader 8\bin\Custom\AddOns\MultiStratManager`. Subfolders (e.g., `External/`) are treated as DLL project code and are not deployed as NinjaScript sources.
	- Purges any DLL-only `.cs` accidentally placed under `bin/Custom` to prevent NinjaScript compile errors.
- When to close NinjaTrader:
	- Editing DLL project code (the C# files that build the .NET DLLs): Close NinjaTrader first, then run the script to update DLLs (avoids file locks). Reopen NT and press F5 to rebuild NinjaScript.
	- Editing NinjaScript source `.cs` (files in `MultiStratManagerRepo` root): You do NOT need to close NinjaTrader. Run the script to sync sources, then press F5 in NinjaTrader to compile.
	- If the script reports locked DLLs in `bin/Custom`, close NinjaTrader and rerun.
 - Typical usage:
	 - For DLL + sources (NT closed): run the script from repo root; reopen NT → F5.
	 - For sources only (NT can stay open): run the script from repo root → F5.

## File sync to live environments
- Source of truth is THIS repo. Make edits here only; then copy the updated files to the live platform folders.
- MT5 (after editing files under `MT5/`): find the exact file in
	`C:\Users\marth\AppData\Roaming\MetaQuotes\Terminal\7BC3F33EDFDBDBDBADB45838B9A2D03F\MQL5` and copy over.
	- Example: for includes like `ACFunctions_gRPC.mqh`, target is
		`...\MQL5\Include\gRPC\ACFunctions_gRPC.mqh` (preserve folder structure).
	- Recompile the EA in MetaEditor after copying. If native DLLs are involved, rebuild them via `MT5/cpp-grpc-client/build_cpp_grpc.bat` first.
- NinjaTrader MultiStratManager: copy changes from `MultiStratManagerRepo/` to
	`C:\Users\marth\OneDrive\Desktop\OneDrive\Old video editing files\NinjaTrader 8\bin\Custom\AddOns\MultiStratManager`, then rebuild inside NinjaTrader.

### NinjaTrader: exact copy targets for addon and gRPC client
- Source of truth (repo):
	- DLL build output: `MultiStratManagerRepo/External/NTGrpcClient/bin/Release/net48/`
	- Addon sources: `MultiStratManagerRepo/*.cs` (e.g., `MultiStratManager.cs`, `IndicatorCalculator.cs`, `SQLiteHelper.cs`)

- Copy these addon source files into the live NinjaTrader addon folder:
	- To: `...\NinjaTrader 8\bin\Custom\AddOns\MultiStratManager\`
	- Files: `MultiStratManager.cs`, `IndicatorCalculator.cs`, `SQLiteHelper.cs`
	- Then rebuild from NinjaTrader's NinjaScript Editor (F5).

- Copy the NT gRPC client DLL and dependencies to folders where NT already loads them (keep duplicates in sync):
	- Primary: `...\NinjaTrader 8\bin\Custom\AddOns\MultiStratManager\External\`
	- Also update if present:
		- `...\NinjaTrader 8\bin\Custom\External\`
		- `...\NinjaTrader 8\External\`
		- And any existing older placements NinjaTrader may probe:
			- `...\NinjaTrader 8\bin\Custom\AddOns\MultiStratManager\`
			- `...\NinjaTrader 8\bin\Custom\AddOns\MultiStratManager\bin\External\`
			- `...\NinjaTrader 8\bin\External\`
			- `...\NinjaTrader 8\bin\Custom\`

- Files to copy from the repo build output (`net48`) into each target folder:
	- `NTGrpcClient.dll`
	- `Grpc.Core.dll`, `Grpc.Core.Api.dll`
	- `Google.Protobuf.dll`
	- `System.Text.Json.dll`
	- `grpc_csharp_ext.x64.dll` (x64 NinjaTrader)
	- Support libs often required on .NET Framework: `Microsoft.Bcl.AsyncInterfaces.dll`, `System.Buffers.dll`, `System.Memory.dll`, `System.Numerics.Vectors.dll`, `System.Runtime.CompilerServices.Unsafe.dll`, `System.Threading.Tasks.Extensions.dll`, `System.ValueTuple.dll`, `System.Text.Encodings.Web.dll`

- Notes:
	- Close NinjaTrader only when updating DLLs (to release `bin/Custom` locks). For NinjaScript source-only changes, NT can remain open.
	- MultiStratManager.cs now forwards logs to the Bridge via `TradingGrpcClient.Log*` and also prints to NinjaScript Output.
	- IndicatorCalculator.cs and SQLiteHelper.cs route their logging through `MultiStratManager.Instance`, so forwarding applies automatically.
	- If NinjaScript compile errors mention missing assemblies, ensure the above DLLs exist next to the addon under `External` (and mirrored dupe paths if used previously). The deploy script handles this automatically.
	- The addon's project references `NTGrpcClient.dll` from `MultiStratManagerRepo\NTGrpcClient.dll` in-repo; at runtime, NinjaTrader resolves from the folders above.

## Debugging
- Bridge logs: startup, health, queue size, stream sends (server.go/app.go)
- Health `source` values update status: `addon`, `hedgebot`, `nt_addon_init`
- NT: NinjaScript Output shows gRPC vs HTTP fallback
- MT5: Experts tab; `GrpcInitialize` and streaming stats; ensure native C++ client rebuilt via `MT5/cpp-grpc-client/build_cpp_grpc.bat` before testing.

## When changing protocol/schema
- Edit `BridgeApp/proto/trading.proto` → run `make proto` in `BridgeApp/` → update JSON/proto conversions in Go + NT + MT5
- For MT5: rebuild the native client via `MT5/cpp-grpc-client/build_cpp_grpc.bat` from your native dev prompt so the deployed DLLs are refreshed.

## Gotchas
- NinjaTrader targets .NET 4.8: missing `netstandard.dll` causes CS0012
- Open firewall for gRPC port (default 50051)
- Match MT5 DLL import signatures to the active client (C# vs C++)

Questions or gaps (e.g., exact NT paths, flags, fallback state)? Ask and we'll refine this file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Galaxybigbird) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
