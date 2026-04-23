## pscue

> PowerShell completion module combining Tab completion (NativeAOT) + inline predictions (managed DLL) with generic learning and cross-session persistence. See README.md for features, installation, configuration, and usage.

# PSCue - Quick Reference for AI Agents

PowerShell completion module combining Tab completion (NativeAOT) + inline predictions (managed DLL) with generic learning and cross-session persistence. See README.md for features, installation, configuration, and usage.

## Architecture Internals
- **ArgumentCompleter** (`pscue-completer.exe`): NativeAOT exe, <10ms startup, computes completions locally with full dynamic arguments support
- **Module** (`PSCue.Module.dll`): Long-lived, implements `ICommandPredictor` + `IFeedbackProvider` (7.4+), provides PowerShell module functions
- **Learning System**:
  - **CommandHistory**: Ring buffer tracking last 100 commands
  - **CommandParser**: Parses commands into typed arguments (Verb, Flag, Parameter, ParameterValue, Standalone)
  - **ArgumentGraph**: Knowledge graph of command → arguments with frequency + recency scoring
    - **ArgumentSequences**: Tracks consecutive argument pairs for multi-word suggestions (up to 50 per command)
    - **ParameterStats**: Tracks parameters and their known values
    - **ParameterValuePairs**: Tracks bound parameter-value pairs
  - **ContextAnalyzer**: Detects command sequences and workflow patterns
  - **SequencePredictor**: ML-based n-gram prediction for next commands
  - **WorkflowLearner**: Learns command → next command transitions with timing data
  - **GenericPredictor**: Generates context-aware suggestions (values only after parameters, flags otherwise)
  - **Hybrid CommandPredictor**: Blends known completions + generic learning + ML predictions + workflow patterns
  - **PcdSubsequenceScorer**: fzf-style subsequence matching with boundary bonuses and match position output for highlighting (adapted from Wade's FuzzyScorer)
  - **PcdCompletionEngine**: Enhanced directory navigation with subsequence matching, frecency scoring, distance awareness
- **Persistence**:
  - **PersistenceManager**: SQLite-based cross-session storage
  - **Tables**: commands, arguments, co_occurrences, flag_combinations, argument_sequences, command_history, command_sequences, workflow_transitions, parameters, parameter_values
  - **Auto-save**: Every 5 minutes + on module unload
  - **Concurrent Access**: SQLite WAL mode handles multiple PowerShell sessions safely
  - **Additive Merging**: Frequencies summed, timestamps use max (most recent)

## Common Tasks
```bash
# Build (requires .NET 10.0 SDK — ArgumentCompleter targets net10.0, Module targets net9.0)
dotnet build src/PSCue.Module/ -c Release -f net9.0
dotnet publish src/PSCue.ArgumentCompleter/ -c Release -r win-x64

# Test
dotnet test test/PSCue.ArgumentCompleter.Tests/
dotnet test test/PSCue.Module.Tests/

# Run specific test groups
dotnet test --filter "FullyQualifiedName~Persistence"
dotnet test --filter "FullyQualifiedName~FeedbackProvider"
dotnet test --filter "FullyQualifiedName~CommandPredictor"
dotnet test --filter "FullyQualifiedName~SequencePredictor"
dotnet test --filter "FullyQualifiedName~WorkflowLearner"

# Install locally
./install-local.ps1

# Dev testing (isolated from production install)
./install-local.ps1 -Force -InstallPath D:\temp\PSCue-dev
# Then in a new PowerShell session:
#   $env:PSCUE_DATA_DIR = "D:\temp\PSCue-dev\data"
#   Import-Module "D:\temp\PSCue-dev\PSCue.psd1"
# Cleanup: Remove-Item -Recurse -Force D:\temp\PSCue-dev
```

## Key Technical Decisions
1. **NativeAOT for ArgumentCompleter**: <10ms startup required for Tab responsiveness
2. **Shared logic in PSCue.Shared**: NativeAOT exe can't be referenced by Module.dll at runtime
3. **ArgumentCompleter computes locally**: Tab completion always computes locally with full dynamic arguments. Fast enough (<50ms) and simpler.
4. **NestedModules in manifest**: Required for `IModuleAssemblyInitializer` to trigger
5. **Concurrent logging**: `FileShare.ReadWrite` + `AutoFlush` for multi-process debug logging
6. **PowerShell module functions**: Direct in-process access, no IPC overhead

## Performance Targets
- ArgumentCompleter startup: <10ms
- Total Tab completion: <50ms
- Module function calls: <5ms
- Database queries: <10ms
- PCD tab completion: <10ms
- PCD best-match navigation: <50ms

## When Adding Features
- Put shared completion logic in `PSCue.Shared`
- DynamicArguments (git branches, scoop packages) are computed locally by ArgumentCompleter
- Learning happens automatically via `FeedbackProvider` - no manual tracking needed
- When adding support for new commands, add the completer registration in `module/PSCue.psm1` as well
- **Command aliases**: Use `Alias` property on `Command` class, include in tooltip like `"Create a new tab (alias: nt)"`
- **Parameter aliases**: Use `Alias` property on `CommandParameter` class, include in tooltip like `"Only list directories (-d)"`
  - **IMPORTANT**: Do NOT create separate parameter entries for short and long forms (e.g., `-d` and `--diff` as separate parameters)
  - Instead, define the long form with the short form as an alias: `new("--diff", "Compare files (-d)") { Alias = "-d" }`
  - This prevents duplicate suggestions and keeps the completion list clean

## Testing Patterns
```csharp
// Test module functions directly using PSCueModule static properties
[Fact]
public void TestLearningAccess()
{
    var graph = PSCueModule.KnowledgeGraph;
    Assert.NotNull(graph);

    // Test learning operations
    graph.RecordUsage("git", new[] { "status" }, null);
    var suggestions = graph.GetSuggestions("git", Array.Empty<string>());
    Assert.Contains(suggestions, s => s.Argument == "status");
}
```

## Common Pitfalls
1. **ArgumentCompleter slow**: DynamicArguments (git branches, scoop packages) are computed on every Tab press. This is expected and fast (<50ms).
2. **NativeAOT reference errors**: Put shared code in PSCue.Shared, not ArgumentCompleter
3. **Module functions return null**: Module may not be fully initialized. Check PSCueModule.KnowledgeGraph != null before use.
4. **Corrupted database prevents initialization**: Use `Clear-PSCueLearning -Force` to delete database files without requiring module initialization.
5. **Partial word completion**: When implementing predictor features, always check if the command line ends with a space. If not, the last word is being completed and suggestions should be filtered by `StartsWith(wordToComplete)`.
6. **Initialize methods must record baseline**: When adding new `Initialize*` methods in ArgumentGraph (used by PersistenceManager during load), always record the loaded values in `_baseline` dictionary. Without baseline tracking, delta calculations will return full counts, causing values to be added again on save, leading to exponential growth and eventual overflow.
7. **PCD exact match not first**: When frecency scoring dominates match quality, use a boost multiplier for exact matches. Small match score components (0.1) can be overwhelmed by frequency/recency scores. Solution: Apply 100× boost for exact matches (matchScore >= 1.0) to ensure they always rank first.
8. **Inconsistent trailing separators**: Directory completions must have trailing separators in BOTH tab completion (ArgumentCompleter) and inline predictions (ICommandPredictor). Check both code paths when modifying directory suggestion logic.
9. **Path normalization requires workingDirectory**: When calling `ArgumentGraph.RecordUsage()` for navigation commands (cd, sl, chdir), MUST provide the `workingDirectory` parameter. If null/empty, path normalization (including symlink resolution) is skipped. This causes duplicate entries for symlinked paths. Always pass a valid working directory for proper deduplication.
10. **PCD best-match returns 0 suggestions**: If `GetSuggestions()` returns no matches, check that `GetLearnedDirectories()` is requesting enough paths from ArgumentGraph. The default pool size should be 200+ to ensure less-frequently-used directories are searchable. Also verify `CalculateMatchScore()` checks BOTH full paths and directory names.
11. **PCD attempts Set-Location on non-existent path**: Always loop through ALL suggestions and verify existence before navigation. Never call `Set-Location` on paths that don't exist - show helpful error messages instead. This handles race conditions and stale database entries gracefully.
12. **PCD tab completion behavior**: CompletionText must match native cd exactly - use .\ prefix for child directories, ..\ for siblings, and single quotes for spaces. ListItemText should be clean (no prefixes/separators/quotes). See module/Functions/PCD.ps1 for implementation.
13. **Testing with non-existent paths**: Use `skipExistenceCheck: true` parameter in `PcdCompletionEngine.GetSuggestions()` when testing with mock/non-existent paths. Production code filters non-existent paths by default.
14. **Release builds missing dependencies**: The release workflow (.github/workflows/release.yml) MUST use `dotnet publish` (not `dotnet build`) for PSCue.Module to include all dependencies. `dotnet build` only outputs primary assemblies, while `dotnet publish` creates a complete deployable package. The release workflow flattens the platform-specific native SQLite library (e.g., `e_sqlite3.dll`, `libe_sqlite3.so`) to the package root and removes the `runtimes/` directory, so release packages have a flat layout.
15. **install-local.ps1 dependency list**: The local install script has a hardcoded `$Dependencies` array listing DLLs to copy from the `publish/` directory. When adding new NuGet packages, you MUST add the DLL to this list or the module will fail at runtime with assembly-not-found errors. Consider replacing this with a bulk copy approach (like `install-remote.ps1` does) to avoid this pitfall.
16. **PCD interactive selector excludes current dir and `..`**: `PcdInteractiveSelector` explicitly filters out the current directory and the `..` parent shortcut. This is intentional — navigating to where you already are is useless, and `..` is always available via `pcd ..` directly. When testing, ensure the learned paths are not just the current directory or its parent.
17. **FeedbackProvider uses PowerShell `$PWD` for path normalization**: The `FeedbackProvider` uses the PowerShell `$PWD` variable (not `System.Environment.CurrentDirectory`) to get the current working directory for path normalization. This is important because `Set-Location` in PowerShell does not update the process CWD. Always use `PSCmdlet.SessionState.Path.CurrentLocation` or invoke `$PWD` via PowerShell when you need the true PowerShell working directory.
18. **Navigation timestamp tracking**: For navigation commands (cd, Set-Location, sl, chdir), the FeedbackProvider records the absolute destination path (from context.CurrentLocation after navigation) with trailing separator, not the relative path typed. The `pcd` function manually records navigations under the 'cd' command since FeedbackProvider only sees the 'pcd' command, not the internal Set-Location calls. Without this, pcd navigations wouldn't update learned directory timestamps.

## Documentation
- Active work: `TODO.md` | Completed work: `docs/COMPLETED.md`
- Database functions: `docs/DATABASE-FUNCTIONS.md`
- Troubleshooting: `docs/TROUBLESHOOTING.md`

# Misc
- When running ./install-local.ps1, always use -Force
- Don't reference phases in code, e.g. "// Phase 20: Parse command to understand parameter-value context" or "// Add multi-word suggestions (Phase 17.4)"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucaspimentel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
