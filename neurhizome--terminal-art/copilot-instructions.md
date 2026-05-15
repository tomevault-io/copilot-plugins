## terminal-art

> This file documents the codebase structure, development conventions, and workflows for AI assistants working in this repository.

# CLAUDE.md — terminal-art (asciicology)

This file documents the codebase structure, development conventions, and workflows for AI assistants working in this repository.

---

## Spirit of the Work

This project is a record of emergence documented from within the process itself. Before reading API signatures or directory trees, understand the methodology:

**Run it. Watch what happens. Write down what surprised you.**

The experiments don't have predetermined outcomes. They are genuine encounters with complexity — patterns that form without being designed, boundaries that emerge from math no walker planned. The sessions in `docs/_posts/` are not tutorials. They are evidence that certain behaviors are possible. Running the same code with different initial conditions will produce different results, and that is correct.

Several constants throughout the codebase are intentionally imprecise — diffusion coefficients that average to almost-right but not quite right. If you correct them, the system dies: equilibrium, gray, nothing to see. The living behavior exists in the gap between correct and almost-correct. **Do not fix these unless you have confirmed they are actual bugs.** Many are the design principle.

The order in which components are composed matters. The initialization history is present in every subsequent state. You cannot cleanly separate the path from the destination.

---

## Self-Documenting Knowledge Graph

The most important automation in this repo: the knowledge graph rebuilds itself.

`tools/graph_viz.py` parses YAML frontmatter from every post and concept file, builds a graph of nodes and edges from the `related:` links, and renders a 96×N ANSI canvas to `docs/assets/captures/knowledge-graph.ans`. This file is displayed live on the `/graph/` page of the blog.

**The graph runs automatically:**

- **Local commits**: `.git/hooks/post-commit` detects changes to `docs/_posts/` or `docs/concepts/` and runs `graph_viz.py`, then stages the updated `.ans` for the next commit.
- **Merge to main**: `.github/workflows/pages.yml` runs `graph_viz.py` before Jekyll builds the site, so the deployed graph always reflects the current state.

The graph is not a map of finished ideas. It is a record of introductions — what met what, in which order. The nodes matter less than the edges. The edges matter less than the sequence.

**To run manually:**
```bash
python3 tools/graph_viz.py
```

**To install the local hook** (if working in a fresh clone):
```bash
cp scripts/hooks/post-commit .git/hooks/post-commit
chmod +x .git/hooks/post-commit
```

**Layout is fully dynamic.** The graph positions every node automatically based on type and date — no hardcoded coordinates. New posts appear in the left (chronological) column; new concepts or aesthetic posts appear in the right (concept space) column. Add a post, commit it, and the graph updates.

---

## Repository Structure

```
terminal-art/
├── src/                          # Core modular toolkit (~3,073 LOC)
│   ├── automata/                 # Walker entities and population management
│   │   ├── walker.py             # Base Walker class (position + genome + age)
│   │   ├── spawner.py            # Population manager (add/remove/breed walkers)
│   │   └── behaviors.py         # Pluggable movement strategies
│   ├── genetics/                 # Memetic trait system
│   │   ├── genome.py             # HSV color genome with inheritance
│   │   ├── inheritance.py        # Parent→child trait flow
│   │   └── speciation.py         # Reproductive barriers (hue distance)
│   ├── fields/                   # 2D grid-based spatial systems
│   │   ├── base.py               # Abstract Field interface
│   │   ├── diffusion.py          # Scent trails / chemical diffusion
│   │   ├── territory.py          # Chunked ownership tracking
│   │   └── energy.py             # Excitable medium with cascade dynamics
│   ├── events/                   # Temporal perturbation system
│   │   ├── event.py              # Base Event class
│   │   ├── catalog.py            # Pre-built event library (AESTHETIC_POOL, CHAOS_POOL)
│   │   └── scheduler.py          # Event timing and triggering
│   ├── glyphs/                   # Probabilistic Unicode character selection
│   │   ├── picker.py             # GlyphPicker (load DB, query by direction/style)
│   │   └── direction.py          # Direction enum (N, NE, E, SE, S, SW, W, NW, NONE)
│   ├── renderers/                # Terminal display layer
│   │   └── terminal_stage.py     # TerminalStage: double-buffered full-screen canvas
│   └── utils/
│       └── colors.py             # HSV/RGB conversion, circular hue mean
├── experiments/                  # Composable experiment scripts (reference implementations)
│   ├── simple_walkers.py         # Minimal example: walkers + colors (~30 lines)
│   ├── memetic_territories.py    # Full-featured: genetics + fields + events
│   ├── color_speciation.py       # Reproductive barriers creating color species
│   ├── gradient_flow.py          # Aesthetic mode: flowing color gradients
│   ├── predator_prey.py          # Lotka-Volterra dynamics (green prey vs red predators)
│   └── README.md                 # Experiment composition guide
├── demos/                        # Standalone pre-modular demonstration scripts (20+)
│   ├── ascii_waves.py            # Cellular automata waves (most feature-rich demo)
│   └── ...                       # Other standalone visualizations
├── tools/                        # Glyph database builders, Unicode scanners, graph renderer
│   ├── graph_viz.py              # Knowledge graph renderer (dynamic layout, auto-run on commit)
│   ├── build_comprehensive_db.py # Generates 1,742-glyph full database
│   ├── build_optimized_db.py     # Mobile-optimized 720-glyph subset
│   └── unicode_scanner.py        # Scans Unicode ranges for character properties
├── docs/                         # Jekyll blog: posts, concepts, graph, gallery
│   ├── _posts/                   # Session exploration blog posts
│   ├── concepts/                 # stigmergy.md, diffusion-memory.md
│   ├── assets/captures/          # ANSI art files (including knowledge-graph.ans)
│   └── graph.md                  # Live knowledge graph page
├── scripts/
│   ├── hooks/post-commit         # Tracked copy of the post-commit hook (install manually)
│   └── speciation_capture.py     # Records animations to ANSI format
├── .github/workflows/pages.yml   # Jekyll deploy — also runs graph_viz.py before build
├── sketches/                     # Quick experiment templates
├── museum/                       # Captured ANSI animation outputs
├── glyph_database.json           # Essential glyph set (~9.6 KB)
├── glyph_database_optimized.json # Mobile-optimized (~136 KB, 720 glyphs)
├── glyph_database_full.json      # Full database (~194 KB, 1,742 glyphs)
└── requirements.txt              # Runtime deps (wcwidth only)
```

---

## Core Abstractions

The toolkit is built on five composable abstractions. Understanding these is essential before making any changes.

### 1. Walker (`src/automata/walker.py`)

An entity with **position + genetic traits**. Behavior is injected, not hardcoded.

```python
class Walker:
    x, y        # Terminal position
    genome      # Genetic traits (color_h, vigor, etc.)
    age         # Tick counter
    vigor       # Fitness weight (affects inheritance)

    def move(self, dx, dy, width, height, wrap=True)
    def deposit(self, field)           # Leave scent/energy in a field
    def sense(self, field)             # Read field values
    def reproduce_with(self, other)    # Returns child Walker
    def can_breed_with(self, other)    # Checks reproductive barrier
```

### 2. Field (`src/fields/base.py`)

An **abstract 2D grid** that stores and evolves values.

```python
class Field(ABC):
    def get(self, x, y) -> value
    def set(self, x, y, value)
    def update()           # Apply dynamics (diffusion, decay, cascade)
    def render()           # Returns grid of (char, fg_color, bg_color)
```

**Concrete implementations:**
- `DiffusionField` — values spread to neighbors and decay each tick
- `TerritoryField` — chunked ownership tracking (vigor-weighted)
- `EnergyField` — excitable medium with cascade dynamics
- `ConnectionField` — NESW bitmasks for connector-walker rendering

### 3. Genome (`src/genetics/genome.py`)

A **memetic trait container** where color is the primary (visible) genetic marker.

```python
class Genome:
    color_h    # Hue [0, 1) — the visible phenotype
    vigor      # Fitness weight — affects inheritance dominance
    traits     # Extensible dict for custom properties

    def reproduce_with(self, other, mutation_rate=0.03) -> Genome
    def distance_to(self, other) -> float   # Circular hue distance [0, 0.5]
```

Reproduction uses **vigor-weighted circular mean** of hue + Gaussian drift. The `distance_to` method enables speciation (reproductive barriers).

### 4. Event (`src/events/event.py`)

**Perturbative dynamics** that temporarily modify system parameters.

```python
class Event:
    duration, strength, target_param
    elapsed

    def apply(self, system)      # Modifies system dict in-place
    def is_finished() -> bool
```

Pre-built events in `catalog.py`: `SpawnRateBurst`, `GlobalColorShift`, `VigorWave`, `Extinction`, and more. Use `AESTHETIC_POOL` for calm events, `CHAOS_POOL` for dramatic ones.

### 5. TerminalStage (`src/renderers/terminal_stage.py`)

A **double-buffered full-screen canvas** for flicker-free rendering.

```python
stage = TerminalStage()
stage.width, stage.height    # Terminal dimensions
stage.set_cell(x, y, char, fg_color, bg_color)
stage.render_field(field)    # Composites a Field onto the canvas
stage.render_walkers(walkers)
stage.flush()                # Writes only changed cells to terminal
```

Uses 24-bit truecolor ANSI escape codes and handles terminal resize.

---

## Composition Patterns

New experiments are built by composing the five abstractions. Reference `experiments/` for working examples.

### Minimal (walkers only)
```python
from src.automata import Spawner
from src.genetics import Genome

spawner = Spawner(max_walkers=500, width=80, height=24)
spawner.spawn_random(genome=Genome(color_h=0.5))

for walker in spawner.walkers:
    dx, dy = behavior.get_move(walker.x, walker.y)
    walker.move(dx, dy, 80, 24, wrap=True)
```

### Full composition (events + genetics + fields)
```python
from src.automata import Spawner
from src.genetics import Genome
from src.fields import DiffusionField, TerritoryField
from src.events import EventScheduler, AESTHETIC_POOL
from src.renderers import TerminalStage

stage = TerminalStage()
spawner = Spawner(max_walkers=300, width=stage.width, height=stage.height)
scent = DiffusionField(stage.width, stage.height)
territory = TerritoryField(stage.width, stage.height, chunk_size=8)
events = EventScheduler()

while True:
    events.update({'spawner': spawner, 'field': scent})
    for walker in spawner.walkers:
        scent.deposit(walker.x, walker.y, walker.vigor)
        territory.claim(walker)
        walker.move(dx, dy, stage.width, stage.height, wrap=True)
    scent.update()
    territory.update()
    stage.render_field(territory)
    stage.render_walkers(spawner.walkers)
    stage.flush()
```

**Data flow:**
```
Walkers → deposit() → Fields → update() → render() → Stage → Display
   ↑                                         ↓
   └──────────── sense() ←──────────────────┘
```

---

## Code Conventions

### Language & Style
- **Python 3.7+** with type hints throughout
- **Snake_case** for functions, variables, module names; **PascalCase** for classes
- **Dataclasses** for immutable state containers (`WalkerState`, `GlyphInfo`)
- **ABC** (Abstract Base Classes) for extensible interfaces
- Comprehensive docstrings at module, class, and method level

### Module Exports
Always import from the package, not sub-modules directly:
```python
from src.automata import Walker, Spawner          # correct
from src.automata.walker import Walker            # avoid in experiments
```

Each subpackage uses explicit `__init__.py` exports via `__all__`.

### Behavior Injection (not inheritance)
Behaviors are **always passed in** — never subclass Walker to add movement:
```python
# Correct: inject behavior
behavior = GradientFollow('scent', attraction=True)
dx, dy = behavior.get_move(walker.x, walker.y, field=scent_field)

# Wrong: subclass for behavior
class ChemotaxisWalker(Walker): ...
```

### On Imprecision
Some numerical constants in the codebase are intentionally not exact. If you encounter a constant like `3.984` where `4.0` seems correct, investigate before changing it. The living dynamics of these systems often depend on the gap between almost-right and right. Correcting to exact values can cause the system to converge to gray equilibrium — which is the death of the pattern, not a bug fix.

### Colors
Colors are represented as **HSV tuples `(h, s, v)`** internally, converted to RGB for ANSI output via `src/utils/colors.py`. Hue `color_h` is a float in `[0, 1)` using circular arithmetic.

### Graceful Degradation
Components fall back gracefully when optional deps are absent (e.g., missing glyphs fall back to space character). Do not add hard runtime requirements.

---

## The Neurhizome Cycle

When working as the pseudonymous author of this blog, follow the **Neurhizome Cycle** documented in `CYCLE.md`. The cycle alternates between directed phases and undirected exploration interludes, with a compression pass (the sleep cycle) that synthesizes discoveries into parables, music prompts, image prompts, and structural metaphors.

The short version: **run it, watch what happens, write down what surprised you, then compress.**

Key artifacts the cycle produces:
- Session posts in `docs/_posts/`
- Trajectories log at `docs/_drafts/trajectories.md`
- Concept pages in `docs/concepts/` (when recurring ideas surface)
- An updated knowledge graph (rebuilt automatically on commit)

See `CYCLE.md` for the full workflow.

---

## Development Workflow

### Environment Setup
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt   # just wcwidth
```

### Running Experiments
```bash
# Minimal — walkers + colors
python3 experiments/simple_walkers.py --walkers 100

# Full-featured with events
python3 experiments/memetic_territories.py --initial-walkers 20 --max-walkers 300 --events

# Color speciation
python3 experiments/color_speciation.py

# Predator-prey dynamics
python3 experiments/predator_prey.py

# Standalone demo
python3 demos/ascii_waves.py --rows 200 --delay 0.01 --style heavy
```

### Writing a Session Post
1. Run an experiment; capture interesting moments to `museum/`
2. Create `docs/_posts/YYYY-MM-DD-title.md` with YAML frontmatter
3. Add `related:` links to connect it to prior sessions and concepts
4. Copy `.ans` captures to `docs/assets/captures/`
5. Commit — the post-commit hook rebuilds the knowledge graph automatically

Post frontmatter shape:
```yaml
---
layout: post
title: "Session N: What I Observed"
date: YYYY-MM-DD
tags: [session, tag1, tag2]
related:
  - title: "Session N-1: Previous"
    url: /YYYY/MM/DD/previous-slug.html
  - title: "Concept: Relevant Concept"
    url: /concepts/concept-slug/
captures:
  - file: my-capture.ans
    title: "Short title"
    description: "Why this was interesting"
    seed: 42
    tick: 1500
    params: "walkers=250, diffusion=0.15"
---
```

### Building Glyph Databases
```bash
python3 tools/build_comprehensive_db.py --all-ranges -o glyph_database_full.json
python3 tools/build_optimized_db.py -o glyph_database_optimized.json
python3 tools/unicode_scanner.py --start 0x2500 --end 0x259F --outfile box_drawing.json
```

### Capturing Animations
```bash
python3 scripts/speciation_capture.py   # Saves ANSI files to museum/
```

### Validation (the methodology)
There is no automated test runner. Validation is watching:
1. Run the experiment. Watch what emerges.
2. If the output is static or gray, something has converged — consider reverting recent changes to numerical constants.
3. Check `experiments/README.md` for described behaviors.
4. Use `PLAYGROUND.md` for guided parameter exploration.

When adding new modules, write a minimal experiment in `experiments/` or `sketches/` that demonstrates the behavior you expect to see.

---

## Knowledge Graph Workflow

The graph (`/graph/` on the blog) is the live topology of what-has-influenced-what.

| Trigger | What runs | Result |
|---------|-----------|--------|
| `git commit` touching any post/concept `.md` | `.git/hooks/post-commit` → `tools/graph_viz.py` | Updated `.ans` staged for next commit |
| Push/merge to `main` | `.github/workflows/pages.yml` → `python3 tools/graph_viz.py` → Jekyll build | Deployed site has fresh graph |
| Manual | `python3 tools/graph_viz.py` | Updated `.ans` written locally |

**How `graph_viz.py` works:**
1. **Parse** — reads YAML frontmatter from every post and concept. Extracts `title`, `date`, `tags`, `related`.
2. **Build** — constructs the node graph. Edges come from `related:` URL lists.
3. **Layout** — assigns positions dynamically: posts sorted by date in the left column; concepts and aesthetic posts sorted by label in the right column. Canvas height grows automatically.
4. **Render** — draws a 96×N ANSI canvas: `double` boxes for sessions, `heavy` for beginning, `single` for concepts, `round` for gradient/aesthetic posts. Edges route as horizontal or L-shaped connectors.
5. **Write** — outputs `docs/assets/captures/knowledge-graph.ans`.

The "Sleep Cycle" compression pass — folding dense clusters of tightly-linked nodes into topological knots — is designed but not yet implemented. The infrastructure is in place.

---

## Performance Guidelines

| Scale | Walkers | Fields | Target FPS |
|-------|---------|--------|-----------|
| Small | < 100 | none | 60+ FPS |
| Medium | 100–500 | 1–2 | 20–30 FPS |
| Large | 500+ | multiple | 10–20 FPS |

Key rules:
- `TerminalStage.flush()` only writes **changed cells** — avoid forcing full redraws
- `TerritoryField` uses **chunked** ownership (8×8 default) to avoid per-cell tracking
- `DiffusionField.update()` uses NumPy-style array ops when available, pure Python fallback
- Use `glyph_database_optimized.json` (720 glyphs) on mobile/low-power terminals
- Prefer `wrap=True` over edge-bounce for lower branch cost in the walker loop

---

## Glyph System

Probabilistic Unicode character selection across 1,742 glyphs.

```python
from src.glyphs import GlyphPicker, Direction

picker = GlyphPicker.from_json("glyph_database_full.json")
char   = picker.get(direction=Direction.E, intensity=0.7)   # varies each call
arrow  = picker.get(direction=Direction.NE, style="arrow")
braille = picker.get(direction=Direction.S, style="braille", intensity=0.3)
```

**Database files:** `glyph_database.json` (essential), `glyph_database_optimized.json` (720, mobile), `glyph_database_full.json` (1,742, all ranges).

**Direction enum:** `N, NE, E, SE, S, SW, W, NW, NONE`

---

## iSH (iOS Alpine Linux) Setup

```bash
apk update && apk add python3 py3-pip ncurses git
python3 -m venv venv && source venv/bin/activate
export TERM=xterm-256color
export COLORTERM=truecolor
export PYTHONIOENCODING=utf-8
pip install wcwidth
python3 demos/ascii_waves.py --rows 300 --delay 0.015 --style light --bg-set dots
```

Use `glyph_database_optimized.json` on iOS.

---

## Key Files for AI Assistants

| File | Purpose |
|------|---------|
| `src/automata/walker.py` | Walker entity — start here for movement/genetics questions |
| `src/automata/spawner.py` | Population management — add/remove/breed walkers |
| `src/automata/behaviors.py` | All movement strategies (RandomWalk, LevyFlight, GradientFollow, etc.) |
| `src/genetics/genome.py` | Color genome and reproduction logic |
| `src/fields/base.py` | Abstract Field interface — all field types extend this |
| `src/fields/diffusion.py` | Most-used field: scent/chemical diffusion |
| `src/events/catalog.py` | Pre-built events and event pools |
| `src/renderers/terminal_stage.py` | Rendering pipeline and display layer |
| `tools/graph_viz.py` | Knowledge graph renderer — dynamic layout, runs on every doc commit |
| `scripts/hooks/post-commit` | Tracked hook source (copy to `.git/hooks/` to activate) |
| `src/utils/colors.py` | Color math (HSV↔RGB, circular hue mean) |
| `experiments/memetic_territories.py` | Best reference for full-stack composition |
| `experiments/simple_walkers.py` | Best reference for minimal composition |
| `ARCHITECTURE.md` | Design patterns and module contracts |
| `docs/graph.md` | Knowledge graph page — topology and self-documentation system |

---

## Common Tasks

### Add a new movement behavior
1. Open `src/automata/behaviors.py`
2. Subclass the behavior base class; implement `get_move(x, y, **kwargs) -> (dx, dy)`
3. Export from `src/automata/__init__.py`
4. Demonstrate in a sketch or experiment

### Add a new field type
1. Subclass `Field` from `src/fields/base.py`
2. Implement `get`, `set`, `update`, and optionally `render`
3. Export from `src/fields/__init__.py`

### Add a new event
1. Subclass `Event` from `src/events/event.py`
2. Implement `apply(self, system)` — `system` is a dict with keys `'spawner'`, `'field'`, `'config'`
3. Add to `catalog.py` and optionally to `AESTHETIC_POOL` or `CHAOS_POOL`

### Create a new experiment
1. Copy `experiments/simple_walkers.py` as starting point
2. Import only what you need; keep total length under 100 lines
3. Add CLI args with `argparse` following the pattern in existing experiments

### Add a new blog post
1. Write `docs/_posts/YYYY-MM-DD-title.md` with `related:` links to prior posts and concepts
2. Commit — the hook rebuilds the graph automatically
3. The `/graph/` page updates on next deploy to main

---

## Documentation Files

| File | Contents |
|------|---------|
| `README.md` | Quick start, full API reference, CLI options |
| `ARCHITECTURE.md` | Design patterns, module contracts, composition examples |
| `OPTIMIZATION.md` | Performance tuning and benchmarks |
| `PLAYGROUND.md` | Creative exploration guide, parameter tuning suggestions |
| `BLOG_GUIDE.md` | How to write session blog posts in `docs/_posts/` |
| `docs/concepts/` | Conceptual articles (stigmergy, diffusion-memory) |
| `docs/graph.md` | Knowledge graph page — the self-documenting topology |
| `experiments/README.md` | Experiment composition patterns and performance table |

---
> Source: [neurhizome/terminal-art](https://github.com/neurhizome/terminal-art) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
