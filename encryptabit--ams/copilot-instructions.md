## ams

> - In AMS GSD auto-mode, before broad `rg`/`find` exploration for C# symbols or subsystem mapping, call `mcp_servers` and use the Serena MCP server for code intelligence when available.

# AMS Project Memory

## Immediate GSD Auto-Mode Rule: Use Serena First for C#

- In AMS GSD auto-mode, before broad `rg`/`find` exploration for C# symbols or subsystem mapping, call `mcp_servers` and use the Serena MCP server for code intelligence when available.
- Prefer Serena over the built-in `lsp` tool for C# navigation in this repo. The built-in OmniSharp adapter can partially index the solution and miss Core/Workstation symbols, while Serena has successfully resolved AMS symbols such as `DocumentSlot`, `PickupEdlModels`, `PickupMatchingService`, `CrxPickupTarget`, and `AsrResponse`.
- Use `rg` only for literal text searches, file inventory, or when Serena is unavailable/insufficient. If Serena is unavailable, say so briefly before falling back.

## Project Overview
Audio Management System (AMS) - CLI and core library for audio processing, ASR, forced alignment, and audiobook mastering.

## GSD Auto-Mode Guardrails

- In GSD auto-mode, treat any message explicitly marked as a `system notification` / `not user input` / `do not treat this as a human message` as non-actionable unless the notification changes local execution state in a way that requires tool use.
- Do **not** emit a user-visible reply to `async_job_result` notifications. Consume the result silently if needed; otherwise ignore it.
- After the required terminal line for a task or slice (for example `Task T01 complete.`), emit no further assistant text unless a new human message arrives.
- Near the end of a GSD auto task, prefer blocking verification commands over background jobs so completion is not followed by stray async notifications.

## Application Layer & CLI Host

- `host/Ams.Core/Application` now exposes use-case commands (`GenerateTranscript`, `ComputeAnchors`, `BuildTranscriptIndex`, `HydrateTranscript`, `RunMfa`, `MergeTimings`) plus orchestration services (`PipelineService`, `ValidationService`). Each command implements a single `ExecuteAsync(ChapterContext, Options, CancellationToken)` entry point so hosts (CLI today, future UI/daemon later) can reuse identical business logic.
- Workspaces now own `BookContext`/`ChapterManager` lifetimes. Each host (CLI REPL, Web UI, future daemons) exposes an `IWorkspace` implementation that opens `ChapterContext` handles via `BookContext.Chapters.CreateContext(...)`, so there is no standalone context factory.
- Pipeline coordination (build index → ASR → anchors → transcript → hydrate → MFA merge) moved out of `Ams.Cli/Commands/PipelineCommand.cs` into `PipelineService`. CLI code now just parses options, resolves a pipeline run (single chapter or batch) and forwards `PipelineRunOptions` + concurrency limits.
- CLI bootstrapping (see `host/Ams.Cli/Program.cs`) registers all application commands/services through DI. Non-trivial orchestration that previously lived under `Ams.Cli/Services` (e.g., ASR/MFA supervisors) has been relocated to Core for consistent behavior across future hosts, while the REPL now surfaces an `IWorkspace` for chapter access.
- When authoring new commands, prefer adding a use-case class inside `Ams.Core.Application.Commands` and wiring it through DI. CLI verbs should stay “thin”—parse arguments, resolve the active workspace (`IWorkspace`), open the needed chapter via `workspace.OpenChapter(...)`, invoke the command/service, then persist artifacts.

## Current Pipeline (MFA-based)

The pipeline uses **Whisper-family ASR** (in-process Whisper.NET or spawned WhisperX) for initial speech recognition and **Montreal Forced Aligner (MFA)** for precise word/phone-level timing refinement.

### Pipeline Stages

1. **ASR Stage** (`host/Ams.Cli/Commands/AsrCommand.cs`)
   - Uses Whisper.NET or WhisperX to generate initial word-level timings
   - Outputs: `{chapter}.asr.json` with token timings

2. **Alignment Stage** (`host/Ams.Cli/Commands/AlignCommand.cs`)
   - Anchor selection: identifies reliable sync points between book text and ASR output
   - Transcript indexing: matches book text to ASR tokens
   - Hydration: creates fully timed transcript from book source
   - Outputs: `{chapter}.align.anchors.json`, `{chapter}.align.tx.json`, `{chapter}.align.hydrate.json`

3. **MFA Forced Alignment** (`host/Ams.Cli/Services/MfaWorkflow.cs`)
   - Validates corpus and identifies out-of-vocabulary (OOV) words
   - Generates pronunciations for OOV words using G2P (grapheme-to-phoneme)
   - Creates custom dictionary with OOV pronunciations
   - Runs forced alignment to produce TextGrid with precise word/phone boundaries
   - Outputs: `{chapter}.TextGrid`, alignment analysis, and supporting artifacts

4. **Timing Merge** (`host/Ams.Core/Alignment/Mfa/MfaTimingMerger.cs`)
   - Merges MFA TextGrid timings into hydrate and tx JSON files
   - Replaces initial ASR timings with more accurate forced-alignment timings

5. **Treated Copy**
   - Pipeline currently copies the source chapter WAV to `{chapter}.treated.wav` for downstream verification/staging.

### Key Models & Services

- **Whisper.NET**: default in-process backend using GGML models
- **WhisperX**: optional spawned backend for alternative word timing behavior
- **MFA Models**:
  - Dictionary: `english_us_arpa`
  - Acoustic: `english_us_arpa`
  - G2P: `english_us_arpa`

### Usage (PowerShell)

```powershell
# Run full pipeline (ASR → Alignment → MFA)
dotnet .\host\Ams.Cli\bin\Debug\net9.0\Ams.Cli.dll pipeline run `
  --book "C:\Books\MyBook.md" `
  --audio "C:\Audio\Chapter01.wav" `
  --work-dir "C:\Work" `
  --chapter "Chapter 01"

# Individual stages
# 1. ASR only
dotnet .\host\Ams.Cli\bin\Debug\net9.0\Ams.Cli.dll asr run `
  --audio "C:\Audio\Chapter01.wav" `
  --output "C:\Work\Chapter01.asr.json"

# 2. Alignment (requires book index and ASR output)
dotnet .\host\Ams.Cli\bin\Debug\net9.0\Ams.Cli.dll align hydrate `
  --book-index "C:\Work\book-index.json" `
  --asr "C:\Work\Chapter01.asr.json" `
  --transcript "C:\Work\Chapter01.align.tx.json" `
  --output "C:\Work\Chapter01.align.hydrate.json"

# 3. (Treated audio is currently a direct copy of the source WAV after alignment/MFA)
```

### Notes
- MFA runs in a persistent PowerShell session with conda environment activated (via `MfaProcessSupervisor`)
- TextGrid files provide phone-level timing precision for improved gap detection
- Roomtone-specific calibration is temporarily disabled while audio processing is simplified

---
> Source: [Encryptabit/AMS](https://github.com/Encryptabit/AMS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
