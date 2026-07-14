## mazepro

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

A Qt Widgets (C++17) desktop application that visualizes maze generation and solving algorithms as an "algorithm-driven maze adventure game". Source comments and all UI strings are in Chinese; match that convention when editing.

## Build & Run

This is a `qmake`-based Qt project (`MazeGame.pro`). There is no test suite, linter, or CI configured.

```bash
# Out-of-source build (this repo's convention — see untracked deck_build/, report_build/)
mkdir -p build && cd build
qmake ../MazeGame.pro        # generates the Makefile
mingw32-make                 # Windows/MinGW; use `nmake` for MSVC, `make` on unix
                             # add `-j` for parallel builds on make/mingw32-make
```

qmake auto-generates `moc_*.cpp`, `ui_*.h`, and `qrc_*.cpp` from `Q_OBJECT` classes and `.ui` forms — these are build artifacts, never edit them. After changing which source/header/form files exist, update the `SOURCES`/`HEADERS`/`FORMS` lists in `MazeGame.pro`, then re-run `qmake` before building.

## Architecture

Two layers: `core/` (algorithms + game logic, pure logic aside from `QObject`) and `ui/` (Qt widgets). UI depends on core; core never depends on UI.

### Global state: `MazeManager` singleton
`core/maze_manager.h` is a `QObject` singleton (`MazeManager::instance()`, created in `main.cpp`) that owns all cross-page state: the four generated mazes (`m_mazeList[4]`, indexed `0=divide-conquer, 1=greedy-MST, 2=backtrack, 3=branch-bound`), the selected "best" maze, cached `DPResult` and `BossResult`, and shared player state (HP/coins/position). Pages communicate through it and react to its Qt signals (`mazeGenerated`, `bestMazeChanged`, `dpCompleted`, `bossCalculated`) rather than calling each other directly. `mazeGeneration()` is a monotonic counter pages use to detect stale data.

### Maze algorithms: `Maze` base class + 4 subclasses
`core/maze_base.h` defines the `Maze` abstract base — the grid (`QVector<QVector<CellType>>`), start/end/coins/traps, and shared analysis methods (`isConnected`, `hasUniquePath`, `computePathLength`, `getPath` via BFS, `countBranches`). Every generator produces a **perfect maze** (a tree: fully connected, no cycles, unique path between any two cells) — later stages rely on this tree property. Subclasses implement `generate()` and `algorithmName()`:
- `MazeDivideConquer` — recursive wall division
- `MazeGreedyMST` — randomized Kruskal MST
- `MazeBacktrack` — randomized DFS backtracking
- `MazeBranchBound` — randomized growing-tree (Prim-like BFS)

Each `generate()` records a `QVector<Step>` of grid snapshots via `addStep()` so the UI can replay the generation frame-by-frame.

### Downstream algorithms
- `core/dp_resource.h` — `calculateOptimalPath()` runs tree DP over the maze. Because the maze is a tree, the start→end path is forced; the DP decides for each branch off that path whether the side-branch's net coin/trap value is positive (enter) or not (skip).
- `core/boss_battle.h` — `calculateOptimalStrategy()` uses branch-and-bound to find the minimum-round skill sequence to defeat a series of bosses (`bArray` HPs) given skills with damage + cooldown.

### UI: `MainWindow` + 6 stacked pages
`ui/mainwindow.h` hosts a left nav list + `QStackedWidget` of six `QWidget` pages (`0=home, 1=maze-generate, 2=maze-compare, 3=dp-resource, 4=boss-battle, 5=play-test`). Pages emit a `navigateTo(int)` signal that `MainWindow::switchPage` handles. Each page pairs a `.h/.cpp` with a Qt Designer `.ui` form.

- `ui/widgets/maze_grid_widget.h` — the core custom-painted maze renderer. One widget serves all views via `DrawMode` (`NormalMode`, `DPMode` with value overlay + optimal path, `PlayMode` with player + fog-of-war, `ThumbnailMode` for the compare page). Set the mode before use.
- `ui/widgets/boss_battle_widget.h` — boss battle visualization.

### Serialization
`Maze::toString`/`fromString` handle in-app maze persistence. `core/json_writer.h` is a small header-only pretty-printer that writes maze/boss config as ordered, human-readable JSON (see `maze_15_15.json` for the exported format: `maze` grid of `#/ /S/E/G/T/B` glyphs, `B` boss HPs, `PlayerSkills` as `[damage,cooldown]` pairs).

---
> Source: [ro-fee/mazepro](https://github.com/ro-fee/mazepro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-14 -->
