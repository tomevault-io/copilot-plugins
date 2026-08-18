## please-eject

> Handoff notes for any AI agent (or human) picking this project up later.

# AGENTS.md — please_eject

Handoff notes for any AI agent (or human) picking this project up later.
Read this before making changes so you don't relearn everything from scratch.

## What this is

A macOS CLI + TUI that frees a **stuck external volume** so it can eject. The
typical pain point: ejecting an external drive fails
with *"one or more programs may be using it"*, forcing an unsafe force-eject.

`please_eject`:
1. Resolves the volume by name.
2. Finds every process holding it open (via `lsof`).
3. Shows an interactive TUI listing those programs.
4. Lets the user stop them — **all user programs at once** (with confirmation)
   or **one-by-one** with the arrow keys.
5. Optionally **nudges Spotlight** to stop indexing (releases `mds`/`mdworker`).
6. **Rescans automatically** after every action and, when clear, offers to eject.

Target OS: **macOS only** (relies on `diskutil`, `lsof`, `ps`, `mdutil`).
Go version at time of writing: **go1.26.0 darwin/arm64**.

## Run / build / test

```sh
go build -o please_eject .            # build binary
./please_eject "My Passport for Mac"         # run (volume name is the arg)
go test ./...                          # unit tests (all in internal/tui)
go vet ./... && gofmt -l .             # keep both clean before finishing
```

There is **no `go.sum` drama** — deps are Charm's bubbletea/bubbles/lipgloss.
Run `go mod tidy` if you add imports.

## Architecture

```
main.go                    arg parsing, volume resolution, launches TUI
internal/volume/volume.go  resolve mount point + eject + Spotlight control (diskutil/mdutil)
internal/scan/scan.go      lsof parsing -> []Process, ps enrichment, system-proc detection
internal/kill/kill.go      SIGTERM -> grace -> SIGKILL escalation, errno-aware messages
internal/tui/model.go      Bubble Tea model: state machine, Update, commands
internal/tui/view.go       Bubble Tea View: rendering + lipgloss styles
internal/tui/*_test.go     state-machine + render tests (drive Update() directly)
```

Dependency direction: `main` -> `tui` -> {`scan`, `kill`, `volume`}. Keep it
that way; `scan`/`kill`/`volume` must not import `tui`.

### How process discovery works (important detail)

Passing a **mount point** to `lsof` makes it list every open file on that
*filesystem*, at any directory depth — not just files literally under that path
string. The command used is:

```
lsof -w -F pcuLtn -- <mountPoint>
```

`-F` is machine-readable: lines are `<tag><value>`. A `p` line starts a process
block; `f` starts a file block within it. See `scan.parse`. Command names from
lsof are short/truncated, so `scan.enrich` runs one batched `ps -o pid=,comm=`
to get full executable paths.

`lsof` exits non-zero when it simply finds nothing — treat empty output as "no
processes", not an error (already handled in `scan.Scan`).

### System-process safety

`scan.isSystem` flags OS-owned/critical processes (Spotlight `mds*`, Quick Look,
Time Machine `backupd`, Finder, `fseventsd`, `diskarbitrationd`, root-owned,
etc.) via the `systemCommands` map + UID==0 check. These are:
- sorted to the bottom of the list,
- **excluded from "stop all" (`a`)** — the user must stop them individually
  with `x`. Don't change this without good reason; force-killing daemons is how
  you cause data-loss surprises.

### Kill strategy

`kill.Graceful` sends SIGTERM, waits `gracePeriod` (4s, defined in tui/model.go),
then escalates to SIGKILL only if still alive. `alive()` uses `syscall.Kill(pid, 0)`
and treats EPERM as "alive but not ours". EPERM on the real kill is surfaced as
"permission denied (try running with sudo)". There's also `kill.Force` (immediate
SIGKILL) — currently unused by the TUI but kept for future use.

Note: an app that lives *on the volume itself* can't release until the process
fully dies — expect SIGKILL escalation in that case.

### TUI state machine (tui/model.go)

States: `stateScanning`, `stateList`, `stateConfirmKillAll`, `stateKilling`,
`stateEjecting`, `stateClear`, `stateEjected`, `stateFatal`.

Messages: `scanDoneMsg{procs, spotlight, err}`, `killDoneMsg{results}`,
`ejectDoneMsg{err}`, `spotlightDoneMsg{err}`.

Flow: scan -> list (or clear if empty) -> user acts -> kill/spotlight/eject ->
**always rescan** to reflect real state -> loop. After a kill or a Spotlight
nudge the model returns to `stateScanning` and re-runs `scanVolCmd`.

Keybindings (list view): `↑/↓`/`j`/`k` move, `space` mark, `x`/`enter` stop
marked-or-current, `a` stop all user progs (confirm), `s` stop Spotlight,
`r` rescan, `q`/`ctrl+c`/`esc` quit. Clear view: `e`/`enter` eject, `s`, `r`, `q`.

The program runs **inline (no alt-screen)** — `tea.NewProgram(tui.New(vol))`
with no `WithAltScreen()`. That's intentional; keep it inline.

### Spotlight nudge (tui + volume)

`s` runs `sudo mdutil -i off <mount>` through **`tea.ExecProcess`**, which
releases the terminal so the user can type their sudo password, then resumes and
rescans. This is the documented Bubble Tea pattern for interactive subprocesses.

`volume.SpotlightStatus()` reads `mdutil -s <mount>` and returns
`"on"|"off"|"not indexed"|""`. `model.nudgeSpotlight()` **short-circuits** when
status is `off` or `not indexed` (shows a message instead of pointlessly
prompting for sudo). The status is fetched inside `scanVolCmd` (off the Update
goroutine) and shown in the header.

## Known limitations / UNVERIFIED paths

Be honest about these; don't claim they're tested.

- **The interactive TUI cannot be driven from piped stdin.** Attempts to feed
  keystrokes via `script`/pipes hang because bubbletea owns the input loop. All
  TUI logic is therefore tested by calling `model.Update()` directly with
  synthetic messages (see `internal/tui/model_test.go`). If you need to verify
  interactive behavior, ask the user to run it in a real terminal.
- **`sudo mdutil -i off` end-to-end is UNVERIFIED.** When a drive reports
  `not indexed`, the nudge correctly short-circuits and the actual sudo path
  never runs. The `ExecProcess` wiring follows the docs but hasn't been
  exercised on an indexed external volume in CI. Don't test this by disabling
  indexing on the boot volume.
- **Eject path partially verified.** `diskutil eject` wiring is straightforward
  but was not run to completion against a live external drive during
  development.
- `scan`/`kill`/`volume` packages have **no unit tests** yet — they were
  validated manually on real hardware. Consider adding table tests for
  `scan.parse` (feed canned `lsof -F` output) if you touch it.

## Repo notes

- The built `please_eject` binary is a build artifact (listed in `.gitignore`);
  do not commit it — Go embeds local source paths in the binary.

## Reasonable future work (only if asked)

- Add `mdutil -i on` re-enable command / toggle.
- Table tests for `scan.parse` and `volume.parseDiskutil`.
- Optional `--json` / non-interactive "list only" mode for scripting.
- Handle volumes with identical names (diskutil picks one; could disambiguate by
  device node).
- A `--force`/`--all` flag to run headless without the TUI.
- Scrolling/pagination in the list if a volume has many holders (currently
  renders all rows; fine for typical cases).

---
> Source: [missingno15/please_eject](https://github.com/missingno15/please_eject) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
