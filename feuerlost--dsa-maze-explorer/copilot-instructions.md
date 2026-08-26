## dsa-maze-explorer

> This is a Data Structures and Algorithms course design project.

# AGENTS.md

## Project Overview

This is a Data Structures and Algorithms course design project.

Topic: Maze Solving / 走迷宫

The project should implement the following features:

1. Rectangular maze representation, generation, and path search.
2. DFS for finding any valid path.
3. BFS for finding the shortest path in an unweighted maze.
4. Pygame GUI for displaying the maze, the search process, and the final path.
5. Mondrian artwork-to-maze conversion:
   - Do not hard-code a fixed Mondrian layout.
   - Implement image processing scripts to detect black thick lines and colored rectangular regions from a Mondrian reference image.
   - Generate `configs/mondrian_layout.json`.
6. Circular non-rectangular maze generation and path search.
7. Pytest tests.
8. README, report draft, and demo video script.

The main design idea is to model every maze as a graph. Nodes represent walkable cells or regions, and edges represent valid movement between adjacent positions. Concrete maze types are responsible for generating nodes and neighbor relationships, while DFS, BFS, and constrained BFS should be implemented in a shared algorithm layer.

## Development Environment

The project is developed in WSL.

Use conda for the Python environment and pip for dependencies.

```bash
conda create -n dsa_maze python=3.10 -y
conda activate dsa_maze
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Main dependencies:

```text
pygame
pytest
numpy
opencv-python
pillow
```

The recommended `requirements.txt` is:

```text
pygame>=2.5.0
pytest>=8.0.0
numpy>=1.24.0
opencv-python>=4.8.0
pillow>=10.0.0
```

## Architecture Requirements

Use a unified `MazeBase` abstraction.

Search algorithms must be decoupled from concrete maze types.

Expected project structure:

```text
dsa-maze-explorer/
├── AGENTS.md
├── main.py
├── requirements.txt
├── README.md
├── logs/
│   └── codex_log.md
├── assets/
│   └── mondrian/
│       ├── README.md
│       └── reference.png
├── configs/
│   ├── rect_default.json
│   ├── circular_default.json
│   └── mondrian_layout.json
├── src/
│   ├── core/
│   │   ├── __init__.py
│   │   ├── maze_base.py
│   │   ├── search.py
│   │   └── path_result.py
│   ├── mazes/
│   │   ├── __init__.py
│   │   ├── rectangular.py
│   │   ├── mondrian.py
│   │   └── circular.py
│   ├── gui/
│   │   ├── __init__.py
│   │   ├── app.py
│   │   ├── draw_rectangular.py
│   │   ├── draw_mondrian.py
│   │   └── draw_circular.py
│   └── utils/
│       ├── __init__.py
│       └── config.py
├── tools/
│   ├── extract_mondrian_layout.py
│   ├── validate_mondrian_layout.py
│   └── preview_mondrian_layout.py
├── tests/
│   ├── test_search.py
│   ├── test_rectangular.py
│   ├── test_mondrian.py
│   └── test_circular.py
└── report/
    ├── report.md
    ├── video_script.md
    └── images/
```

Each maze type should implement the following interface through `MazeBase`:

```python
nodes()
neighbors(node)
start
goal
```

DFS, BFS, and Mondrian room-constrained BFS should live in `src/core/search.py`.

## Core Modules

### `src/core/maze_base.py`

Define the abstract base class for all mazes.

Expected responsibilities:

- Provide `start` and `goal`.
- Provide `nodes()`.
- Provide `neighbors(node)`.
- Keep the interface independent of specific maze shapes.

### `src/core/path_result.py`

Define a `PathResult` data class.

Expected fields:

```python
path: list
visited_order: list
found: bool
algorithm: str
extra: dict
```

### `src/core/search.py`

Implement shared graph search algorithms:

- `dfs_find_path(maze)` for finding any valid path.
- `bfs_shortest_path(maze)` for finding the shortest path in an unweighted maze.
- `bfs_with_room_constraint(maze, min_rooms_required=3)` for Mondrian maze search.

Search functions should return `PathResult`.

## Maze Modules

### Rectangular Maze

File: `src/mazes/rectangular.py`

Requirements:

- Use a `rows x cols` grid.
- Node format: `(row, col)`.
- Each cell has four walls: up, right, down, and left.
- Use randomized DFS/backtracking to generate a perfect maze.
- The entrance should be on the left boundary.
- The exit should be on the opposite right boundary.
- `neighbors(node)` should only return adjacent cells with no wall between them.
- Support a random seed for reproducible results.

### Mondrian Maze

File: `src/mazes/mondrian.py`

Requirements:

- Read layout from `configs/mondrian_layout.json`.
- Treat black thick lines in the original artwork as walls.
- Treat colored rectangular regions as rooms.
- Use doors to connect neighboring rooms.
- Discretize each room into small grid nodes.
- Recommended node format: `(room_id, local_row, local_col)`.
- Implement `room_of(node)`.
- Implement path search with the requirement that the final path passes through at least three rooms.

### Circular Maze

File: `src/mazes/circular.py`

Requirements:

- Use a polar-coordinate grid.
- Node format: `(ring, sector)`.
- Use a configurable number of rings and sectors.
- Each node may connect to:
  - left sector in the same ring,
  - right sector in the same ring,
  - inner ring,
  - outer ring.
- Use randomized DFS/backtracking to generate the maze.
- Put the entrance on the outer boundary.
- Put the exit inside the circular maze.
- Reuse the shared BFS algorithm for shortest path search.

## GUI Requirements

Use pygame for GUI.

GUI code should stay separate from algorithm code.

Files:

- `src/gui/app.py`
- `src/gui/draw_rectangular.py`
- `src/gui/draw_mondrian.py`
- `src/gui/draw_circular.py`

The GUI should support:

- Drawing the maze.
- Marking start and goal.
- Animating visited nodes.
- Drawing the final path.
- Displaying path length and visited count.

Recommended keys:

```text
R: regenerate or reload
B: run BFS
D: run DFS
Space: pause or resume animation
Q or Esc: quit
```

For the Mondrian GUI, also display:

- visited rooms count,
- minimum required rooms,
- doors,
- colored rooms,
- thick black borders.

## Mondrian Image Processing Toolchain

The Mondrian maze should not be based on a fully hard-coded layout.

Implement scripts that convert a reference image into a maze layout.

### `tools/extract_mondrian_layout.py`

Command:

```bash
python tools/extract_mondrian_layout.py assets/mondrian/reference.png --out configs/mondrian_layout.json --preview report/images/mondrian_detected.png
```

Responsibilities:

- Read the reference image using OpenCV, NumPy, or Pillow.
- Detect black thick lines as walls.
- Use connected-component analysis on non-black regions.
- Filter out small noisy regions.
- Generate room data:
  - `id`
  - `name`
  - `rect`
  - `color`
  - `area`
- Infer neighboring rooms from their bounding boxes.
- Automatically generate doors between neighboring rooms.
- Ensure the room graph is connected.
- Choose `start_room` as the leftmost suitable room.
- Choose `goal_room` as the rightmost suitable room.
- Set `min_rooms_required` to 3 by default.
- Output `configs/mondrian_layout.json`.
- Output a preview image with room IDs, room boxes, doors, start, and goal.

### `tools/validate_mondrian_layout.py`

Command:

```bash
python tools/validate_mondrian_layout.py configs/mondrian_layout.json
```

Responsibilities:

- Validate JSON field completeness.
- Check that the number of rooms is at least 3.
- Check that door endpoints are valid room IDs.
- Check that `start_room` can reach `goal_room`.
- Check that there exists a room-level path passing through at least `min_rooms_required` rooms.
- Print clear validation results.

### `tools/preview_mondrian_layout.py`

Command:

```bash
python tools/preview_mondrian_layout.py configs/mondrian_layout.json --out report/images/mondrian_layout_preview.png
```

Responsibilities:

- Redraw the Mondrian maze from JSON.
- Fill rooms using configured colors.
- Draw thick black borders.
- Draw doors using green markers or gaps.
- Mark start and goal.
- Draw a room-level path that satisfies the minimum room constraint if such a path exists.

## Main Commands

Non-GUI commands:

```bash
python main.py --mode rect --rows 10 --cols 10
python main.py --mode mondrian
python main.py --mode circular --rings 6 --sectors 32
```

GUI commands:

```bash
python main.py --mode rect --rows 25 --cols 25 --gui
python main.py --mode mondrian --gui
python main.py --mode circular --rings 6 --sectors 32 --gui
```

Test command:

```bash
pytest -q
```

## Development Rules for Codex

Follow these rules when modifying the project:

1. Do not rewrite the whole project unless explicitly asked.
2. Modify only files relevant to the current phase.
3. Keep code simple and readable for a course design report.
4. Avoid unnecessary dependencies.
5. Prefer standard library, pygame, pytest, numpy, opencv-python, and pillow.
6. Keep GUI code separate from algorithm code.
7. Do not duplicate DFS or BFS logic inside GUI files.
8. After each change, explain which commands should be run to test it.
9. Do not invent screenshots, benchmark numbers, or fake results in the report.
10. If a feature is incomplete, state it clearly.
11. Keep generated files deterministic when a seed is provided.
12. Add or update tests when changing algorithmic behavior.

## Testing Requirements

Tests should not require a GUI window.

Required test coverage:

### `tests/test_search.py`

- DFS can find a valid path.
- BFS can find the shortest path.
- Unreachable graph returns `found=False`.

### `tests/test_rectangular.py`

- Generated rectangular maze is reachable from start to goal.
- BFS path length is no longer than DFS path length.
- Multiple seeds generate reachable mazes.
- `neighbors(node)` never returns out-of-bound nodes.

### `tests/test_mondrian.py`

- `configs/mondrian_layout.json` has required fields.
- Number of rooms is at least 3.
- Door endpoints are valid room IDs.
- `MondrianMaze` can find a path from start to goal.
- Room-constrained BFS returns a path passing through at least `min_rooms_required` rooms.

### `tests/test_circular.py`

- Circular maze start and goal are reachable.
- Multiple seeds generate reachable circular mazes.
- `neighbors(node)` never returns an invalid ring or sector.
- BFS path length is no longer than DFS path length.

## Report Requirements

The final course design report should include:

1. Problem analysis.
2. Data structure design and implementation.
3. Algorithm design.
4. Complexity analysis.
5. Result analysis.
6. Member contribution.
7. AI-assisted development description.

The report should emphasize:

- Unified graph abstraction.
- DFS for valid path search.
- BFS for shortest path in an unweighted maze.
- Randomized DFS/backtracking for maze generation.
- Mondrian artwork-to-maze conversion using image processing.
- Mondrian room-constrained BFS.
- Circular maze based on a polar-coordinate grid.
- Separation between data structure, algorithm, and GUI layers.

Do not fabricate screenshots or results. Use placeholders until real screenshots are produced.

## Suggested Development Phases

1. Project skeleton.
2. `MazeBase`, `PathResult`, DFS, and BFS.
3. Rectangular maze generation and solving.
4. Rectangular maze GUI.
5. Mondrian image layout extraction tools.
6. Mondrian maze and room-constrained BFS.
7. Mondrian GUI.
8. Circular maze generation and solving.
9. Circular maze GUI.
10. Unified CLI and GUI workflow.
11. Complete tests.
12. README, report draft, and video script.

---
> Source: [feuerlost/dsa-maze-explorer](https://github.com/feuerlost/dsa-maze-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
