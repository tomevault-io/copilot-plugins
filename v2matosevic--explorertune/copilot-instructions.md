## explorertune

> generates ~130 rules for 13 names. Do not "simplify" them.

# ExplorerTune

PowerShell toolkit that tunes the in-box Windows File Explorer. Writes to HKLM,
HKCR and the Windows Search crawl scope on a live machine.

This file is for coding agents. Humans want [README.md](README.md) and
[CONTRIBUTING.md](CONTRIBUTING.md).

## Invariants

Break one of these and you break the product, not just the style.

- **Never write without `-Apply`.** Every mutating script is dry-run by default
  and prints a plan. Do not add a script that acts on bare invocation. CI fails
  the build if a bare run creates `backups/`.
- **Back up before you write.** `Backup-RegKey` for every registry root you are
  about to change, into a `New-BackupDir` folder, and print the
  `Restore-Explorer.ps1 -From ...` line at the end. A change with no export is a
  bug.
- **Block shell extensions, do not delete them.** Add the CLSID to
  `Shell Extensions\Blocked` in BOTH registry views. A deleted vendor key comes
  back with their next update; a Blocked value does not, and undo is one value.
  The only exception is a registration whose CLSID resolves to no COM server —
  those are dead and get deleted.
- **Static verbs, never new `shellex` handlers.** A `shell\<verb>\command` string
  costs nothing at runtime. A `shellex` handler is a DLL loaded into
  `explorer.exe` on every right-click. This project exists to remove those.
- **Count real successes.** Never print "wrote N" from the length of the input
  list. An early version reported "wrote 82 rules" while all 82 writes were
  denied by an ACL. Increment inside the `try`.
- **Verify against the system, not against your own write.** Confirm a blocked
  DLL via `explorer.exe`'s module list. Confirm search scope via
  `IncludedInCrawlScope`. Re-reading a registry value you just set proves
  nothing.
- **Nothing machine-specific in a commit.** No usernames, drive layouts, volume
  GUIDs or SIDs. Paths go in `explorertune.config.psd1` (gitignored) or the
  example config as `%USERPROFILE%`-style strings. CI scans for this.
- **Nothing is blocked by default.** `Handlers.Block` ships empty. Blocking is
  the user's decision, per catalog id.

## Search scope: read before touching

- **The crawl scope CANNOT be set from the registry.** `UserScopeRules` denies
  `CreateSubKey` to an elevated Administrator; `DefaultRules` gives
  `BUILTIN\Administrators` only `ReadKey`. `NT SERVICE\WSearch` owns both and
  regenerates `DefaultRules` on every service start. Use the COM path in
  `lib/SearchApi.ps1`. Deletions of non-default rules do stick, which is all
  `Clear-SearchRuleJunk.ps1` claims.
- **Never diagnose scope from the registry.** Measured: `DefaultRules` said
  `include=1` for two whole drives while `IncludedInCrawlScope` reported both
  entirely out of scope. Ask the API.
- **The bracketed GUID in a crawl rule is not the NTFS volume GUID.** Different
  identifier entirely for the same volume. A rule built from `mountvol` never
  matches and never errors. Do not construct these strings; pass plain paths to
  the API.
- **PowerShell cannot call these COM interfaces.** It dispatches through
  `IDispatch`, which they do not implement, so a `[ComImport]` cast arrives as a
  bare `System.__ComObject`. Every call lives inside the C# in
  `lib/SearchApi.ps1`.
- **The published IIDs are not always right.** `ISearchCatalogManager` answered
  to `...EF50` rather than the widely quoted `...EF61`/`...EF62` on build 26100.
  Hence runtime discovery by QueryInterface sweep. Do not hardcode them back.
- **Never call a hand-declared vtable without a two-sided oracle.**
  `ISearchCatalogManager::Reset()` is at slot 5 and `RevertToDefaultScopes()` at
  slot 13, so an off-by-one index is data loss, not a crash. Require a read-only
  method to return the RIGHT answer for a case that must be true AND one that
  must be false before enabling any mutating call. One-sided checks pass by
  accident; two-sided ones do not. `Validated = false` must keep every mutating
  method throwing.

## Non-obvious why

- **Windows Search has no recursive name exclusion.** A `*` matches exactly ONE
  path segment; the shipped defaults prove it (`file:///*\$RECYCLE.BIN\`). So
  each noise directory name needs one rule per depth, which is why the plan
  generates ~130 rules for 13 names. Do not "simplify" them.
- **`WorkingSetRules` is a derived cache**, normalised from `DefaultRules` plus
  the user scope on startup. Safe to wipe. `DefaultRules` is partly
  regenerated: the service restores one whole-drive rule per present fixed
  volume; anything else stays deleted.
- **`SetupCompletedSuccessfully=0` triggers a rebuild but the service sets the
  flag back to 1 itself.** Never read it back as confirmation. Use
  `GetCatalogStatus` / `NumberOfItems`.
- **Icon overlays are capped at 15 by sort order.** Count before cutting:
  removing dead entries can fix starvation without touching a working app.
- **Never match an overlay key name un-normalised.** Vendors pad names to jump
  the queue — some with leading spaces, at least one with **U+0001**.
  `String.Trim()` strips Unicode whitespace but NOT control characters, so
  `Trim()` plus `^MEGA` finds zero of six such keys. Strip `^[\s\p{C}]+`.
- **`h` is a built-in alias for `Get-History`, and aliases beat functions** in
  PowerShell's command resolution. A helper named `H` silently does nothing.
  Cost the audit every section header for several runs.
- **Walk the CLSID hive with the .NET registry API, not the PowerShell
  provider.** `Get-ChildItem HKCR:\CLSID` over ~20k keys took over ten minutes;
  `[Microsoft.Win32.RegistryKey]::OpenBaseKey` takes two seconds. See
  `Get-ETClsidByDll`.
- **Check both registry views for anything CLSID-related.** A 32-bit handler is
  invisible from the 64-bit view alone.
- **Never launch `explorer.exe` from an elevated process.** It inherits the
  elevated token: drag-and-drop from normal apps breaks and its children run as
  administrator. `Restart-Explorer.ps1` detects and refuses.
- **The shell-vs-filesystem timing ratio is noisy** (25x, 65x and 155x on the
  same folder in one session). Report it as an order of magnitude. The
  attributable measure of a blocking change is the module count and the
  third-party DLL list.
- **The watchdog holds off on purpose.** Thresholds alone are not enough since a
  restart closes windows, so it also needs no open windows, user idle, and a
  cooldown. Do not loosen those to make it fire more often.
- **The index store dir is ACL'd to SYSTEM**, so an unelevated `Test-Path` on
  `Windows.edb` returns false whether or not it exists. Do not report that as
  missing.

## Layout

```
lib/Common.ps1      backup, logging, elevation, vitals, CLSID resolution
lib/Config.ps1      defaults + explorertune.config.psd1 merge
lib/SearchApi.ps1   the search COM interop; all C#, discovery + validation
handlers/catalog.psd1   community list of known shell extensions (data only)
```

## Commands

| Command | Elevated | Notes |
|---|---|---|
| `.\Audit-Explorer.ps1` | no | read-only; write it to a file for before/after |
| `.\Optimize-Explorer.ps1 -List` | no | catalog vs what is installed here |
| `.\Optimize-Explorer.ps1 [-Apply] [-Block <ids>]` | to apply | restarts Explorer unless `-NoRestart` |
| `.\Set-SearchScope.ps1 [-Apply\|-Test <paths>\|-Status]` | no | `-Test` is the only honest scope query |
| `.\Clear-SearchRuleJunk.ps1 [-Apply] [-Rebuild]` | to apply | cleanup only; cannot set scope |
| `.\Setup-Everything.ps1 [-Apply]` | to apply | winget install + static verbs |
| `.\Restore-Explorer.ps1 [-List] [-From <dir>]` | to restore | newest backup if `-From` omitted |
| `.\watchdog\Install-Watchdog.ps1 [-Uninstall]` | maybe | scheduled task, user context |

## Read order

1. `README.md` — what it does and why.
2. `docs/search-scope.md` — almost nothing about Windows Search is what you
   would expect; read this before changing anything search-related.
3. `docs/how-it-works.md` — the shell-extension mechanisms.
4. `docs/case-study.md` — the measured audit this was built from, including the
   three things that turned out to be wrong.

---
> Source: [v2matosevic/ExplorerTune](https://github.com/v2matosevic/ExplorerTune) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
