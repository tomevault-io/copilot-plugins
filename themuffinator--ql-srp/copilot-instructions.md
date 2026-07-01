## ql-srp

> This repository exists to faithfully reconstruct the Quake Live engine and game source using the Binary Ninja HLIL references as an accurate guide to the retail code base, with the committed Ghidra corpus acting as a structured companion evidence base; every change should focus on rebuilding the Quake Live codebase piece by piece on top of the retail Quake III Arena GPL source.

# Agent Instructions

This repository exists to faithfully reconstruct the Quake Live engine and game source using the Binary Ninja HLIL references as an accurate guide to the retail code base, with the committed Ghidra corpus acting as a structured companion evidence base; every change should focus on rebuilding the Quake Live codebase piece by piece on top of the retail Quake III Arena GPL source.

This repository currently has the following rules for agents:

- Obey system, developer, and user instructions, and ensure AGENTS.md scope rules are followed.
- In C/C++ code, indent with tabs, and include the required commented header format above every function definition.
- Avoid creating binary files (including standard images, models, and similar assets).
- Avoid creating new source files for trivially small code additions unless they are expected to grow.
- When tasks are large, break them into smaller tasks automatically.
- Do not make significant decisions based on assumptions; ask questions if needed.
- Treat Quake Live-only online services (advert fetching, Awesomium/web menu fetching, and Steam integration) as an explicit divergence from the repository's accuracy-prioritizing reverse-engineering goal: keep them behind `QL_BUILD_ONLINE_SERVICES`, default that setting to disabled, and prefer elegant stubs or fallbacks over live service usage until a documented open replacement path exists.
- Prefer `rg` instead of `ls -R` or `grep -R` for repository searches.
- Never add VS Code `preLaunchTask` entries to `.vscode/launch.json` or any other launch configuration; keep build and launch steps explicit and separate.
- Never launch the game in fullscreen; always use `+set r_fullscreen 0` for every automated or manual launch command.
- Only launch the game when there is a credible investigative or testing need that cannot be resolved by static analysis, unit/integration tests, log inspection, or other lower-cost evidence. Prefer the cheapest runtime mode that answers the question: use dedicated or otherwise headless-style probes for qagame/server-only work when possible, prefer reduced-render client probes such as `+set r_norefresh 1 +set s_initsound 0` when visuals are not under test, and reserve full visual client launches for renderer, UI, input, cgame, or rendered-output validation.
- After committing changes, generate a pull request message using the `make_pr` tool.
- For each task completion, estimate before and after parity percentages versus the retail Quake Live source base outlined in the Binary Ninja HLIL references.
- **Read-only access to the `assets/` and `src/ui/` directory trees.**

## Reverse-Engineering Evidence Workflow

Use the committed references before making new assumptions:

- Canonical parity evidence:
  - retail Quake Live binaries in `assets/quakelive/`
  - Binary Ninja HLIL dumps in `references/hlil/`
- Structured companion corpus:
  - `references/reverse-engineering/ghidra/quakelive_steam/`
  - `references/reverse-engineering/ghidra/cgamex86/`
  - `references/reverse-engineering/ghidra/qagamex86/`
  - `references/reverse-engineering/ghidra/uix86/`
- Symbol/name support:
  - `references/symbol-maps/`
  - `references/analysis/quakelive_symbol_aliases.json`

When analyzing a subsystem:

1. Pick the owning retail binary first.
2. Start with `metadata.txt`, `imports.txt`, `exports.txt`, and `functions.csv`.
3. Use `analysis_symbols.txt` for promoted analyst names.
4. Treat `decompile_top_functions.c` as a hint set, not ground truth.
5. Cross-check behavior claims against HLIL when control flow or ownership is ambiguous.

Inference guardrails imported from the OpenAlice workflow:

- Build claims from at least two signals when possible:
  - call relationships and symbol context
  - strings, imports, exports, or constants
  - repeated offsets and access patterns
- Separate observed facts from inferred meaning in notes and reviews.
- Track confidence and open questions instead of forcing unstable renames.
- Optional live MCP analysis is advisory until revalidated against the committed corpus and HLIL.

Reference tooling:

- `scripts/ghidra/run_quakelive_reference.ps1`
- `scripts/ghidra/ExportQuakeLiveReference.java`
- `scripts/ghidra/setup_ghidrassist_mcp.ps1`
- `docs/reverse-engineering/ghidra-reference-workflow.md`
- `docs/reverse-engineering/ghidrassist-mcp.md`

## Automatic Debugging Process (Windows)

Use this process only when code changes create a credible startup/runtime question that static evidence and automated tests do not settle. Pure mapping, naming, or source-reconstruction work does not require a launch unless the reconstruction itself needs runtime confirmation.

1. Build `Debug|x86` and ensure the binary is up to date.
2. Choose the lowest-cost runtime probe that can answer the question:
   - Prefer dedicated or otherwise headless-style execution for qagame/server-only investigations when the client and renderer are not part of the hypothesis under test.
   - If a client process is still needed but rendered output is not, prefer reduced-render probes such as `+set r_norefresh 1 +set s_initsound 0`.
   - Escalate to a normal visual client launch only when UI, cgame, renderer, input, screenshot evidence, or other rendered-output behavior is part of the investigation.
3. When a visual client launch is required, run a normal launch pass with logging enabled:
   - Use `+set developer 1 +set logfile 2 +set g_logfile 1`.
   - Always force windowed mode with `+set r_fullscreen 0`. Fullscreen launches are prohibited.
   - Set `+set fs_basepath C:\\Program Files (x86)\\Steam\\steamapps\\common\\Quake Live` (or equivalent retail install containing `baseq3\\pak00.pk3`).
   - Set `+set fs_cdpath ${workspaceFolder}\\assets\\quakelive` and `+set fs_homepath ${workspaceFolder}\\build\\win32\\Debug\\bin`.
   - Use `cwd=${workspaceFolder}` to keep path resolution deterministic.
   - Capture at least one runtime screenshot for this pass and store it under `build\\win32\\Debug\\dumps\\screenshots\\`.
   - Use only the engine-generated screenshot commands (`screenshot` or `screenshotJPEG`; prefer `screenshotJPEG` for tooling compatibility).
   - Do not use OS-level print-screen, desktop capture, or process-bound window-capture workflows as runtime evidence.
4. Read the newest `build\\win32\\Debug\\bin\\baseq3\\qconsole.log` and classify result:
   - Success path: confirm non-zero pk3 mounts, preflight checks, UI init completion, and clean shutdown sequence.
   - Crash/fatal path: capture exact failing subsystem and message from log.
5. Validate dump generation on crash paths only when crash handling, dump generation, or crash-path behavior is part of the investigation:
   - Set `QLR_DUMP_PATH=${workspaceFolder}\\build\\win32\\Debug\\dumps`.
   - Trigger/observe crash and verify a fresh `quakelive_steam_*.dmp` file appears.
   - Capture an engine screenshot near the crash moment (or immediately after relaunch if the process exits too fast).
   - Use the engine screenshot artifact as the authoritative rendered-output record for crash-path investigation.
6. Iterate fix -> rebuild -> relaunch:
   - If crash occurred, inspect dump/log evidence, patch root cause, then rerun the minimum probe needed to retest the hypothesis.
   - Do not add a forced-crash pass by default; only run `+crash` when dump-pipeline validation or crash-path behavior is itself under test.
7. Report outcomes with artifact evidence:
   - Log snippets (startup and failure/success markers).
   - Dump filename/timestamp/size when crashes are tested.
   - Screenshot filenames/timestamps for each visual pass.
   - Whether post-fix relaunch is stable.

# Work Queue

Agents should refer to `IMPLEMENTATION_PLAN.md` for the current prioritized list of reconstruction tasks. The primary goal is to close the parity gaps identified in `AUDIT.md`.

*   **Priority 1**: Implement PQL/CPMA Air Control logic in `bg_pmove.c` (Physics).
*   **Priority 2**: Verify UI bridge and fallback mechanisms.
*   **Priority 3**: Validate Race and Gametype logic.

Tick off items in `IMPLEMENTATION_PLAN.md` as they are completed.

---
> Source: [themuffinator/QL-SRP](https://github.com/themuffinator/QL-SRP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
