## vim-cmake-naive

> The nearest ancestor directory (from current working directory upward) that

# Concepts

## Root Directory

The nearest ancestor directory (from current working directory upward) that
contains `CMakeLists.txt`. Many commands fail immediately if this directory cannot
be found.

## Local Configuration

`.vim-cmake-naive-config.json` file at the [Root Directory](#root-directory).

Keys:

1. `output` - build output root (default fallback: `build`)
2. `preset` - CMake preset name (empty string means "no preset")
3. `build` - CMake build type (default fallback: `Debug`)
4. `target` - active selected target (missing means `all`)

## Local Cache

`.vim-cmake-naive-cache.json` file at the [Root Directory](#root-directory).

Keys:

1. `targets` - list of discovered target names

## Build Directory

Resolved from [Local Configuration](#local-configuration):

1. If `preset` is empty: `<root>/<output>`
2. If `preset` is set: `<root>/<output>/<preset>`

## Integration State Files

These files are maintained in the [Root Directory](#root-directory):

1. `.vim-cmake-naive-target` - current target name (is empty when target key is
   missing)
2. `.vim-cmake-naive-output` - current build directory path relative to root
   (`<output>` or `<output>/<preset>`)

They are updated whenever local configuration is written.

## Active and Target Compile Commands

1. Active file: `<output>/compile_commands.json` (used as the main compile
   database)
2. Target-local files:
   `<build-directory>/**/CMakeFiles/<target>.dir/compile_commands.json`

## Command Lock

All public `:CMake*` commands run through a single lock.  If one command is
already running, a new command is rejected with: `CMake: another command
<CommandName> is already running`.

## Terminal Reuse

The following commands are run asynchronously and share the same Vim terminal
window:

1. `CMakeBuild`
2. `CMakeTest`
3. `CMakeRun`

# Commands

## CMakeConfig

1. Resolves [Root Directory](#root-directory) from current working directory.
2. Resolves [Local Configuration](#local-configuration) at that root.
3. Creates `.vim-cmake-naive-config.json` only when it does not already exist.
4. Initial payload is an empty JSON object (`{}`), not default values.
5. Updates integration state files (`.vim-cmake-naive-target`,
   `.vim-cmake-naive-output`) immediately after writing config.
6. If config already exists, command does not overwrite it.

## CMakeConfigDefault

1. Resolves [Root Directory](#root-directory).
2. Resolves [Local Configuration](#local-configuration) at that root.
3. Creates config when missing, or updates existing config.
4. Writes default keys:
   1. `output = "build"`
   2. `preset = ""`
   3. `build = "Debug"`
5. Preserves unrelated custom keys in existing config.
6. Updates integration state files.

## CMakeConfigSetOutput `<value>`

1. Requires local configuration to exist.
2. Validates that `<value>` is non-empty.
3. Resolves [Local Configuration](#local-configuration) in current directory or
   parent directories.
4. Sets `output` to the exact provided string.
5. Updates integration state files, including recalculated
   `.vim-cmake-naive-output` based on current `preset`.

## CMakeSwitchPreset

1. Requires local configuration to exist.
2. Resolves [Root Directory](#root-directory), then reads `CMakePresets.json`.
3. Resolves [Local Configuration](#local-configuration).
4. Builds selectable preset list from `configurePresets`:
   1. skips hidden presets
   2. evaluates supported preset conditions (`const`, `equals`, `notequals`,
      `inList`, `notInList`, `matches`, `notMatches`, `anyOf`, `allOf`, `not`)
   3. resolves inheritance and rejects cycles/missing parents
5. Prepends synthetic option `(none)`.
6. Shows [Preset Selection Popup](#preset-selection-popup) when popup support
   exists; otherwise uses menu/inputlist fallback.
7. On selection:
   1. `(none)` removes `preset`
   2. any preset name sets `preset` and removes `build`

## CMakeSwitchBuild

1. Requires local configuration to exist.
2. Resolves [Local Configuration](#local-configuration).
3. Uses current [Local Configuration](#local-configuration) to resolve current
   build value.
4. Presents options: `(none)`, `Debug`, `Release`, `RelWithDebInfo`, `MinSizeRel`.
5. Shows [Build Selection Popup](#build-selection-popup) when popup support
   exists; otherwise uses menu/inputlist fallback.
6. On selection:
   1. `(none)` removes `build`
   2. build name sets `build` and removes `preset`

## CMakeSwitchTarget

1. Requires local configuration to exist.
2. Requires [Local Cache](#local-cache) with `targets` array (usually produced by
   `:CMakeGenerate`).
3. If cache file is missing, reports built-in error: `No cache found. Please run
   CMakeGenerate command first.`
4. Resolves [Local Configuration](#local-configuration).
5. Resolves [Build Directory](#build-directory) and [Scan
   Directory](#scan-directory) from config.
6. Prepends synthetic option `(all)` to cached targets.
7. Shows [Target Selection Popup](#target-selection-popup) when popup support
   exists; otherwise inputlist fallback.
8. On selection:
   1. `(all)`:
      1. copies root `<scan-directory>/compile_commands.json` to
         `<output>/compile_commands.json`
      2. removes `target` key from config
   2. specific target:
      1. resolves `.../CMakeFiles/<target>.dir`
      2. copies that target-local `compile_commands.json` to
         `<output>/compile_commands.json`
      3. sets `target` key in config
9. Updates integration state files through config writes.

## CMakeGenerate

1. Resolves [Root Directory](#root-directory).
2. Resolves [Local Configuration](#local-configuration), creating default config if
   missing.
3. Computes output directories from `output` and optional `preset`.
4. Ensures output directories exist.
5. Starts asynchronous terminal command:
   1. `cmake -S <root> -B <generation-dir> --fresh -DCMAKE_BUILD_TYPE=<build>`
   2. adds `--preset <preset>` when preset is non-empty
6. Uses plugin terminal split/reuse system (horizontal split, reusable terminal
   window, command status naming).
7. On successful completion:
   1. reads root `compile_commands.json` from scan directory
   2. discovers available targets
   3. updates `.vim-cmake-naive-cache.json` key `targets`
   4. splits root compile database into target-local compile databases

## CMakeBuild

1. Resolves [Root Directory](#root-directory).
2. Resolves [Local Configuration](#local-configuration), creating default config if
   missing.
3. Resolves target build directory from `output` and optional `preset`.
4. Detects parallelism:
   1. `$NUMBER_OF_PROCESSORS` first
   2. then platform commands (`sysctl` / `nproc` / `getconf`)
   3. fallback `1`
5. Starts asynchronous terminal command:
   1. `cmake --build <dir> --parallel <N>`
   2. adds `--preset <preset>` when preset is set
   3. adds `--target <target>` when target is set
6. Uses plugin terminal split/reuse system.

## CMakeTest

1. Resolves [Root Directory](#root-directory).
2. Resolves [Local Configuration](#local-configuration), creating default config if
   missing.
3. Resolves test working directory from `output` and optional `preset`.
4. Ensures test directory exists.
5. Starts asynchronous terminal command in that directory:
   1. `ctest --parallel <N>`
6. Uses plugin terminal split/reuse system.

## CMakeRun

1. Requires non-empty `target` in config; otherwise reports: `No target selected.
   Please use CMakeSwitchTarget command first.`
2. Resolves [Root Directory](#root-directory).
3. Resolves [Local Configuration](#local-configuration), creating default config if
   missing.
4. Resolves run directory from `output` and optional `preset`.
5. Resolves executable path for target:
   1. checks direct candidate names (`<target>`, and on Windows also
      `.exe/.bat/.cmd`)
   2. checks `<run-dir>/<build>/<candidate>` when `build` value is set
   3. falls back to recursive search under run directory
   4. excludes files under `CMakeFiles/`
   5. requires exactly one executable match
6. Starts asynchronous terminal command for discovered executable in run
   directory.
7. Uses plugin terminal split/reuse system.

## CMakeClose

1. Closes all visible plugin-managed terminal windows.
2. Wipes hidden plugin-managed terminal buffers.
3. Resets remembered reusable terminal state.
4. Clears pending terminal success callbacks.

## CMakeInfo

1. Resolves [Local Configuration](#local-configuration).
2. Reads nearest existing [Local Configuration](#local-configuration).
3. If config exists:
   1. sorts keys
   2. renders aligned `key | value` table
   3. title includes config filename: `CMake info [.vim-cmake-naive-config.json]`
4. If config is missing, displays one-line guidance: `No configuration, please use
   CMakeConfigDefault to get started`
5. Opens [Information Popup](#information-popup).

## CMakeMenu

1. Opens command menu for compact set:
   1. `CMakeBuild`
   2. `CMakeRun`
   3. `CMakeTest`
   4. `CMakeSwitchTarget`
2. Uses [Command Menu Popup](#command-menu-popup) when popup support exists;
   otherwise list/menu fallback.
3. Runs selected command with `silent`.

## CMakeMenuFull

1. Opens command menu for full command set:
   1. `CMakeConfig`
   2. `CMakeConfigDefault`
   3. `CMakeSwitchPreset`
   4. `CMakeSwitchBuild`
   5. `CMakeSwitchTarget`
   6. `CMakeGenerate`
   7. `CMakeBuild`
   8. `CMakeTest`
   9. `CMakeRun`
   10. `CMakeClose`
   11. `CMakeInfo`
   12. `CMakeMenu`
   13. `CMakeMenuFull`
   14. `CMakeConfigSetOutput`
2. Only commands that currently exist in Vim are shown.
3. For commands requiring args (`CMakeConfigSetOutput`), prompts: `Arguments for
   <Command>: `.
4. Empty args cancel execution for that command.
5. Runs selected command with `silent`.


# Popups

## Shared Selection Popup Behavior Used by preset/build/target/menu popups.

1. Visual style:
   1. fixed width: `30`
   2. dynamic height: `1..10` lines (with scrollbar)
   3. highlight: `Pmenu`
   4. border highlight: `Pmenu`
   5. single-line rounded border style
2. Item rendering:
   1. all rows are numbered (`1.`, `2.`, ...)
   2. preset/build/target popups show `*` marker on current item
   3. menu popups intentionally do not show `*`
3. Navigation keys:
   1. `j` or `Down` - move down
   2. `k` or `Up` - move up
   3. `b`, `Enter`, or `CR` - confirm selection
   4. `x` or `Esc` - close/cancel
4. Search mode:
   1. `Ctrl+I` toggles search mode on/off
   2. while search mode is on, title ends with `(Insert)`
   3. with query text, title becomes `<Prompt> [<query>] (Insert)`
   4. search is case-insensitive substring matching
   5. query updates after each typed character
   6. `Backspace`, `Ctrl+H`, `Del`, and `kDel` remove one character
   7. `Ctrl+U` clears query
   8. leaving search mode keeps current query/filter
5. Important key precedence detail:
   1. `x`, `Esc`, `j`, `k`, `b`, and Enter remain command keys even when search
      mode is active
   2. only other printable characters are inserted into search query
6. Empty filter result is rendered as one row: `1.   no matches`.

## Information Popup Created by `:CMakeInfo` via `popup_create`.

1. Visual style:
   1. width is dynamic and clamped to `10..30`
   2. width is computed from max(title length, content width)
   3. height is dynamic and clamped to `1..10`
   4. highlight: `Pmenu`
   5. border highlight: `Pmenu`
   6. single-line rounded border style
   7. padding: top/right/bottom/left = `0,1,0,1`
2. Content:
   1. config table lines (`key | value`) when config exists
   2. one guidance line when config is missing
3. Uses `popup_filter_menu` filter behavior.

## Preset Selection Popup Created by `:CMakeSwitchPreset`.

1. Title starts as `Select CMake preset`.
2. `(none)` is displayed as the first option.
3. Current preset is marked with `*`.
4. On confirm, applies preset selection logic (set/remove keys as described in
   [CMakeSwitchPreset](#cmakeswitchpreset)).

## Build Selection Popup Created by `:CMakeSwitchBuild`.

1. Title starts as `Select CMake build`.
2. `(none)` plus default build types are displayed.
3. Current build is marked with `*`.
4. On confirm, applies build selection logic (set/remove keys as described in
   [CMakeSwitchBuild](#cmakeswitchbuild)).

## Target Selection Popup Created by `:CMakeSwitchTarget`.

1. Title starts as `Select CMake target`.
2. `(all)` is displayed as the first option.
3. Current target is marked with `*` (or `(all)` when target key is missing).
4. On confirm, applies target selection logic (copy compile commands and
   set/remove `target` as described in [CMakeSwitchTarget](#cmakeswitchtarget)).

## Command Menu Popup Created by `:CMakeMenu` and `:CMakeMenuFull`.

1. Title starts as `Select CMake command`.
2. Rows are numbered command names with no current-item marker.
3. `:CMakeMenu` shows compact command set.
4. `:CMakeMenuFull` shows full command set.
5. On confirm, executes selected command immediately (`silent` execution).


# Plug Mappings

The plugin registers `<Plug>(...)` mappings for all public commands:

1. `<Plug>(CMakeConfig)`
2. `<Plug>(CMakeConfigDefault)`
3. `<Plug>(CMakeSwitchPreset)`
4. `<Plug>(CMakeSwitchBuild)`
5. `<Plug>(CMakeSwitchTarget)`
6. `<Plug>(CMakeGenerate)`
7. `<Plug>(CMakeBuild)`
8. `<Plug>(CMakeTest)`
9. `<Plug>(CMakeRun)`
10. `<Plug>(CMakeClose)`
11. `<Plug>(CMakeInfo)`
12. `<Plug>(CMakeMenu)`
13. `<Plug>(CMakeMenuFull)`
14. `<Plug>(CMakeConfigSetOutput)` (opens command-line with trailing space for
    argument entry)


# Additional Public Vimscript Functions (No Ex command wrapper)

## `vim_cmake_naive#split(<build-directory>, [...options])`

1. Splits a root `compile_commands.json` into target-local files.
2. Supports:
   1. `-i`, `--input`, `--input=<path>`
   2. `-o`, `--output-name`, `--output-name=<name>`
   3. `--dry-run`
3. Output file name must be a file name only (no path separators).

## `vim_cmake_naive#switch(<build-directory>, <target>, [...options])`

1. Copies target-local `compile_commands.json` into active output location.
2. Supports output override:
   1. `-o <path>`
   2. `--output <path>`
   3. `--output=<path>`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OlegKarasik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
