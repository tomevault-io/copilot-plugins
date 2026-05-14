## flatboard

> TypeScript-based parametric design system for generating 3D-printable keyboard cases (split, unibody, macropads). Uses scad-js to transpile TypeScript to OpenSCAD, enabling programmatic CAD modeling with modern language features.

# flatboard: Parametric Keyboard Case Generator

## Project Overview

TypeScript-based parametric design system for generating 3D-printable keyboard cases (split, unibody, macropads). Uses scad-js to transpile TypeScript to OpenSCAD, enabling programmatic CAD modeling with modern language features.

## Core Technology Stack

- **TypeScript**: Configuration and geometry logic
- **scad-js v0.6.9**: TypeScript-to-OpenSCAD transpiler (also renders STL directly)
- **fp-ts**: Functional programming patterns (pipe, Option, Either, Array utilities)
- **Bun**: Runtime and build system with Bun.Glob for profile discovery
- **Biome**: Linting and formatting (120 line width, single quotes, trailing commas)

## Project Structure

```
src/
├── index.ts                  # CLI with object literal command routing
├── build.ts                  # Build orchestration with 3 modes
├── config.ts                 # Configuration factory with explicit construction
├── profile-loader.ts         # Bridge between profiles/ and src/
├── interfaces.ts             # TypeScript type definitions (250 lines)
├── switches.ts               # Switch specifications (Choc, MX)
├── connector-specs.ts        # Connector definitions (USB-C, TRRS, power button)
├── layout.ts                 # Layout geometry calculations (split + unibody mirroring)
├── switch-sockets.ts         # Switch cutout generation
├── connector.ts              # Connector system with face mapping
├── top.ts                    # Top plate assembly (fp-ts pipe)
├── top-mcu-pocket.ts         # MCU pocket with pin access and USB port cutout
├── bottom.ts                 # Bottom case assembly (fp-ts pipe, 8-step pipeline)
├── bottom-pads-sockets.ts    # Silicon pad socket structures (pure functional)
├── bottom-magsafe-ring.ts    # MagSafe ring for magnetic tenting
├── bottom-patterns.ts        # Bottom patterns (honeycomb, circles, square)
├── switch-visualization.ts   # Switch and keycap visualization for preview
├── utils.ts                  # Core utilities (constants, anchor positioning, deepMerge, rotatePoint, createRoundedSquare)
└── assets/
    ├── cherry-mx.ts          # 3D Cherry MX switch model for visualization
    ├── choc.ts               # Choc keycap model with configurable parameters
    └── keycap-generator.ts   # DSA/XDA keycap generator (layered polygon approach)

profiles/
├── index.ts                  # Auto-discovery with Bun.Glob
├── split-36.ts               # 36-key split (MX, MCU pocket, magsafe, honeycomb pattern)
├── corne.ts                  # 42-key split (MX, USB-C, circles pattern, magsafe)
├── sweep.ts                  # 34-key split (Choc, USB-C, honeycomb pattern, magsafe)
├── planck.ts                 # 48-key unibody ortholinear (MX, USB-C)
├── unibody-36.ts             # 36-key unibody (Choc, USB-C, power button)
├── macropad-3x3.ts           # 9-key macropad (MX, MCU pocket, honeycomb pattern)
├── test-single-choc.ts       # Single Choc key test
└── test-single-mx.ts         # Single MX key test

dist/                         # Generated output
├── <profile>-<hash>/         # Production builds (timestamped)
│   ├── top.scad / top.stl
│   ├── bottom.scad / bottom.stl
│   └── complete.scad / complete.stl
└── (dev mode writes directly to dist/)
```

## Configuration System

### Profile Separation

Keyboard configurations are separated from the codebase in the `profiles/` directory:

- **User space**: `profiles/` contains keyboard definitions (data only)
- **Code space**: `src/` contains the parametric generator logic
- **Auto-discovery**: `profiles/index.ts` uses Bun.Glob to scan and dynamically import all profile files

```typescript
// profiles/index.ts
const glob = new Glob('*.ts');
const profileFiles = Array.from(glob.scanSync({ cwd: __dirname }))
  .filter((file) => file !== 'index.ts');

for (const file of profileFiles) {
  const filePath = join(__dirname, file);
  const module = await import(filePath);
  const profileName = file.replace(/\.ts$/, '');

  if (module.profile) {
    profileEntries.push([profileName, module.profile]);
  }
}
```

### Configuration Flow

```
profiles/*.ts → PROFILES → profile-loader.ts → config.ts → SWITCH_SPECS → KeyboardConfig
```

1. Profile files export a `profile` constant of type `ParameterProfile`
2. `profiles/index.ts` dynamically imports all profiles using Bun.Glob
3. `profile-loader.ts` provides type-safe access to profiles (thin bridge: `getProfileNames()`, `profileExists()`, `getProfile()`)
4. `config.ts` constructs `KeyboardConfig` explicitly from profile + switch specs, using fp-ts Option for null-safe profile lookup
5. `normalizeEdgeMargin()` converts `number | EdgeMargin | undefined` → `EdgeMargin` object
6. Switch spec `spacingX`/`spacingY` used as defaults if not set in profile
7. Final `KeyboardConfig` contains all required parameters with proper defaults

### Row Layout System

Each row defined by parameters:
```typescript
rowLayout: [
  { start: 0, length: 3, offset: 5, thumbAnchor: 2 },  // Row 0: 3 keys, 5mm stagger, thumb anchored to key 2
  { start: -1, length: 4, offset: 0 },                  // Row 1: starts at column -1
  { start: 1, length: 3, offset: 2 }                    // Row 2: starts at column 1
]
```

- `start`: Starting column (can be negative)
- `length`: Number of keys
- `offset`: Column stagger in millimeters
- `thumbAnchor`: Optional anchor key index for thumb cluster positioning

### Switch Support

Automatic parameter inheritance based on switch type. Specs are flat objects in `switches.ts`:

**Choc**: `innerWidth: 13.8`, `outerWidth: 15.0`, `spacingX: 17.7`, `spacingY: 16.6`, `depth: 4.35`, `ledgeHeight: 2.2`
**MX**: `innerWidth: 13.9`, `outerWidth: 15.6`, `spacingX: 18.6`, `spacingY: 18.6`, `depth: 4.0`, `ledgeHeight: 4.0`

Both use `as const satisfies Record<string, SwitchSpec>` for type-safe const narrowing.

### Connector System

Connectors can be placed on any face (top/bottom/left/right) with 0-1 positioning along that edge.

Connectors use direct face mapping — the face name matches the physical edge:
- `top`: High Y edge (away from user) → rotation 180°
- `right`: High X edge → rotation 90°
- `bottom`: Low Y edge (near user) → rotation 0°
- `left`: Low X edge → rotation -90°
**Geometry types** (defined in `connector-specs.ts`):
- `pill`: Two cylinders connected with hull (USB-C: circleRadius 1.55, centerDistance 5.8)
- `circle`: Single circular cutout (TRRS: radius 2.45)
- `square`: Rectangular cutout (power button: width 10.5, height 5.6)

## Code Patterns

### Functional Programming with fp-ts

The codebase uses fp-ts for functional composition:

```typescript
// Pipe for sequential operations (from bottom-pads-sockets.ts)
return pipe(
  config.enclosure.bottomPadsSockets ?? [],
  O.fromPredicate(sockets => sockets.length > 0),
  O.map(sockets => processSocketStructures(sockets)),
  O.getOrElse(() => ({ reinforcements: null, cutouts: null }))
);
```

**Common patterns:**
- `pipe()`: Sequential transformations — used in every geometry module
- `O.fromNullable()`: Convert nullable to Option — optional features (magsafe, MCU pocket, patterns)
- `O.fromPredicate()`: Convert based on condition — empty array checks
- `O.map()` → `O.toNullable()`: Transform optional geometry, return null if absent
- `O.fold()`: Branch on Some/None — config.ts profile lookup
- `O.getOrElse()`: Provide default for None
- `E.Either`: Error handling with Left/Right — `deepMerge`, `validateRoundedSquareParams`
- `A.traverse(E.Applicative)`: Traverse array collecting errors — deepMerge implementation
- `A.map()`, `A.filter()`, `A.chain()`, `A.reduce()`, `A.concat()`, `A.zipWith()`: Array operations
- `A.mapWithIndex()`, `A.compact()`, `A.makeBy()`: Layout generation

### Object Literal Pattern

Replaced switch statements and if-else chains with object literals:

```typescript
const positionCalculators = {
  top: () => ({ x: wallThickness, y: calculateY(), z, rotation: [0, 0, -90] }),
  right: () => ({ x: calculateX(), y: wallThickness, z, rotation: [0, 0, 0] }),
  bottom: () => ({ x: plateWidth + wallThickness, y: calculateY(), z, rotation: [0, 0, 90] }),
  left: () => ({ x: calculateX(), y: plateHeight + wallThickness, z, rotation: [0, 0, 180] }),
};

return positionCalculators[face]();
```

This pattern appears in:
- `connector.ts`: `positionCalculators`, `geometryCreators`
- `utils.ts`: `anchorCalculators` (shared anchor positioning)
- `bottom-pads-sockets.ts`: `boundaryCalculators`, `shapeCreators`
- `top-mcu-pocket.ts`: `positionCalculators` (USB port)
- `bottom-patterns.ts`: `patternGenerators`
- `bottom-magsafe-ring.ts`: `placementZOffsets`
- `index.ts`: `commands` (CLI routing)

### Pure Functions

Modules use pure functional patterns:
- No mutations (eliminated `forEach`, `push`, mutable arrays) — except `switch-visualization.ts` and `build.ts` which use imperative style
- Extracted pure helper functions
- Functional composition with `pipe`, `A.map`, `O.fromPredicate`

Example from `layout.ts`:
```typescript
const createMatrixKey = (
  rowIndex: number,
  keyIndex: number,
  row: { start: number; length: number; offset?: number },
  spacingX: number,
  spacingY: number,
): KeyPlacement => ({
  pos: {
    x: (row.start + keyIndex) * spacingX + (row.offset ?? 0),
    y: rowIndex * spacingY,
  },
  rot: 0,
});
```

## Key Modules

### `profiles/index.ts`
Auto-discovery system using Bun.Glob. Scans all `*.ts` files in `profiles/` directory and dynamically imports them. Filters out `index.ts` itself. Exports `PROFILES` as `Record<string, ParameterProfile>`.

### `profile-loader.ts`
Bridge between `profiles/` and `src/`. Provides:
- `getProfileNames()`: List all available profiles
- `profileExists()`: Check if profile exists
- `getProfile()`: Get profile by name
- Re-exports `PROFILES` as `KEYBOARD_PROFILES`

### `config.ts`
Configuration factory that constructs `KeyboardConfig` explicitly from profile parameters and switch specifications:
```typescript
const createConfigFromProfile = (params: ParameterProfile): KeyboardConfig => {
  const switchType = params.switch?.type ?? 'mx';
  const switchSpec = SWITCH_SPECS[switchType as keyof typeof SWITCH_SPECS];

  const switchConfig: SwitchConfig = {
    type: switchType,
    outerWidth: switchSpec?.outerWidth ?? 15.6,
    innerWidth: switchSpec?.innerWidth ?? 13.9,
    // ... all switch dimensions with spec defaults
  };

  return {
    layout: {
      matrix: { rowLayout: params.layout?.matrix?.rowLayout ?? [], spacingX, spacingY },
      edgeMargin: normalizeEdgeMargin(params.layout?.edgeMargin),
      baseDegrees: params.layout?.baseDegrees ?? 0,
      // ... all sections with explicit defaults
    },
    switch: switchConfig,
    // ... enclosure, connectors, output
  };
};
```

No unsafe casts — every field is explicitly constructed with proper defaults. Switch spec lookup happens inside `createConfigFromProfile` in a single pass.

Also exports everything from `connector-specs.ts`, `interfaces.ts`, `profile-loader.ts`, and `switches.ts`.

### `layout.ts`
Mathematical layout engine with rotation-aware calculations:
- `buildLayout()`: Creates matrix keys via `A.mapWithIndex` → `A.compact` → `A.chain`, then thumb cluster via `A.makeBy` → `A.map`
- `applyGlobalRotation()`: Rotates entire layout by `baseDegrees`
- `mirrorLayout()`: For unibody mode — creates mirrored left/right halves separated by `centerGap` (mirrors along X axis)
- `getLayout()`: Calculates final positions with edge margins and wall thickness offsets
- `calculatePlateDimensions()`: Computes plate size from key bounds + edge margins
- `calculateKeyBounds()`: Rotation-aware bounding box using `calculateAbsoluteCosineSine`

### `connector.ts`
Generic connector system with type-safe specifications. Uses direct face mapping — face names match physical edges. Uses object literals for position and geometry calculations.

**Geometry creators:**
- `pill`: Two cylinders connected with hull (USB-C)
- `circle`: Single circular cutout (TRRS)
- `square`: Rectangular cube cutout (power button)

### `top.ts`
Top plate assembly using fp-ts `pipe` for 6-step conditional geometry composition:
```typescript
return pipe(
  union(wallBox, sockets),
  (geometry) => mcuReinforcement ? union(geometry, mcuReinforcement) : geometry,
  (geometry) => difference(geometry, socketCutouts),
  (geometry) => connectorCutouts ? difference(geometry, connectorCutouts) : geometry,
  (geometry) => mcuPocket ? difference(geometry, mcuPocket) : geometry,
  (geometry) => mcuUSBPort ? difference(geometry, mcuUSBPort) : geometry,
);
```

Creates:
- Outer wall box (rounded square extruded, with inner cavity)
- Switch sockets (walls + inner mounting ledges)
- MCU pocket reinforcement (additive, optional)
- Switch socket cutouts (subtractive)
- Connector cutouts (subtractive, optional)
- MCU pocket cavity and USB port cutout (subtractive, optional)

### `top-mcu-pocket.ts`
MCU (microcontroller) pocket system for the top plate. Returns `{reinforcement, pocket, usbPortCutout}`.

**Features:**
- Anchor-based positioning (`top-left`, `top-right`, `bottom-left`, `bottom-right`, `center`) via shared `calculateAnchorPosition()`
- Configurable pocket dimensions (width, height, depth)
- Pin access cutouts — center area left open for pin through-holes, side areas closed
- USB port cutout — rectangular hole through pocket wall for cable access (configurable edge: top/bottom/left/right)
- Rotation support for angled placement
- Reinforcement walls around the pocket with configurable thickness and height
- Exports `calculatePocketBoundary()` and `calculateMCUPocketAnchor()` for shared use by `bottom.ts`

### `bottom.ts`
Bottom case assembly using fp-ts `pipe` for 8-step conditional pipeline:
```typescript
return pipe(
  baseGeometry,                                                                    // floor plate + inner walls
  (geometry) => socketStructures.reinforcements ? union(geometry, ...) : geometry,  // + pad socket reinforcements
  (geometry) => magsafeRingStructure ? union(geometry, ...) : geometry,             // + magsafe ring structure
  (geometry) => patternCutout ? difference(geometry, ...) : geometry,               // - bottom pattern (with exclusion zones)
  (geometry) => connectorCutouts ? difference(geometry, ...) : geometry,            // - connector cutouts
  (geometry) => mcuPocketCutout ? difference(geometry, ...) : geometry,             // - MCU pocket passthrough
  (geometry) => socketStructures.cutouts ? difference(geometry, ...) : geometry,    // - pad socket holes
  (geometry) => magsafeRingCutout ? difference(geometry, ...) : geometry,           // - magsafe ring groove
);
```

Creates:
- Base floor plate (rounded square extruded to bottomThickness)
- Inner walls extending upward (outer square minus inner square)
- Silicon pad socket reinforcements (additive, optional)
- MagSafe ring structure (additive, optional)
- Bottom pattern cutout — honeycomb/circles/squares with exclusion zones around sockets and magsafe (subtractive, optional)
- Connector cutouts hulled between bottom and top positions (subtractive, optional)
- MCU pocket passthrough from bottom into top plate cavity (subtractive, optional)
- Silicon pad socket holes (subtractive, optional)
- MagSafe ring groove (subtractive, optional)

### `bottom-pads-sockets.ts`
Socket structure generation using pure functional patterns. Creates reinforcements and cutouts for silicon pad sockets with anchor-based positioning.

**Features:**
- Anchor-based positioning via shared `calculateAnchorPosition()` from `utils.ts` (wrapped in `calculateSocketAnchor()`)
- Size-aware boundary calculations including reinforcement and wall thickness
- Additive reinforcement (thickness and height added to socket dimensions)
- Support for round and square socket shapes
- Exclusion zone generation for bottom pattern system

Object literal pattern for:
- `calculateSocketBoundary()`: Boundary calculations
- `createSocketShapes()`: Shape creation

### `bottom-magsafe-ring.ts`
Magnetic mounting for tenting using MagSafe ring socket with phone holders. Center-positioned with configurable X/Y offset.

**Standard MagSafe dimensions** (constants in code):
- Outer diameter: 56mm
- Inner diameter: 44mm
- Depth: 0.6mm

Three functions for three concerns:
- `createMagSafeRingStructure()`: Reinforcement ring (additive)
- `createMagSafeRingCutout()`: Ring groove for magnet (subtractive), supports `external` and `embedded` placement
- `createMagSafeRingExclusionZone()`: 2D footprint for bottom pattern exclusion

**Configuration:**
```typescript
magsafeRing: {
  clearance: 0.2,              // Fit adjustment (positive = looser)
  reinforcement: {
    outer: 2.0,                // Thickness around outer diameter
    inner: 2.0,                // Grip margin on inner diameter
    height: 0.5,               // Additional height for ring
  },
  position: {
    offset: { x: 0, y: 0 },    // Offset from keyboard center
    placement: 'embedded',      // 'external' or 'embedded'
  },
}
```

### `bottom-patterns.ts`
Bottom plate pattern generation for weight reduction and aesthetics. Returns `O.Option<ScadObject>`.

Three pattern types:
- `honeycomb`: Hexagonal grid using `circle($fn: 6)` with staggered rows
- `circles`: Circular holes with optional row staggering
- `square`: Rounded squares in staggered grid

Grid generation uses functional patterns: `makeSymmetricRange()` helper with `A.chain()`, `A.mapWithIndex()`, and `A.flatten()` for declarative grid construction (no imperative loops or mutation).

All patterns are generated as 2D, bounded by `intersection(pattern, boundary_square)`, then passed through exclusion zones (socket + magsafe footprints are subtracted) before being extruded and subtracted from the bottom plate.

**Configuration:**
```typescript
bottomPattern: {
  type: 'honeycomb',    // 'honeycomb' | 'circles' | 'square'
  cellSize: 14,         // Size of each cell
  wallThickness: 4,     // Wall between cells
  margin: 5,            // Inset from case edges
}
```

### `switch-sockets.ts`
Switch socket and cutout generation using `SocketOptions` parameter object pattern. Two-layer socket structure per key:

- **Socket wall**: `createRectangularFrame(outer + 2*wallThickness, outer, depth)` — extends down from top surface
- **Inner ledge**: `createRectangularFrame(outer, inner, ledgeHeight)` — lip at socket bottom to hold switch
- **Cutouts**: Outer cutout above ledge + inner cutout through ledge — subtractive geometry

All functions accept `SocketOptions` (`{ dimensions: SwitchDimensions; totalHeight: number }`) instead of 8-11 positional parameters. `SwitchDimensions` is `Omit<SwitchConfig, 'type'>`.

### `switch-visualization.ts`
Creates 3D switch and keycap visualizations for the `complete.scad` preview file.

- Positions Cherry MX switch bodies at each key placement
- Positions keycaps (DSA, XDA, or Choc profile) at each key placement
- Configurable keycap colors
- Only used in `buildCompleteEnclosure()` — not in top/bottom plate output

### `assets/cherry-mx.ts`
Detailed 3D Cherry MX switch model with colored components: stem (brown), top housing (gray), bottom housing (green), guides (dark green), pins (orange). Built from cubes, cylinders, and hulls.

### `assets/choc.ts`
Kailh Choc keycap model with configurable parameters (width, depth, height, corner radius, skirt, bump). Uses `minkowski` for smooth rounding. Includes stem legs for mounting.

### `assets/keycap-generator.ts`
DSA and XDA profile keycap generator (406 lines — largest file). Builds keycaps layer by layer using 10 polygon layers with `hull()` between adjacent layers for smooth tapering. Includes Cherry MX cross stem, spherical dish cutout, interior hollow, and stem walls.

### `utils.ts`
Core utilities:
- `CSG_OVERLAP`: Constant (`0.1`) ensuring clean CSG boolean operations (prevents coincident face artifacts in OpenSCAD)
- `DEFAULT_CORNER_RADIUS`: Constant (`0.5`) for enclosure rounded squares
- `calculateAnchorPosition()`: Shared anchor-based positioning for MCU pocket, silicon pad sockets, and bottom case cutouts. Uses `AnchorPosition` type and object literal pattern for 5-point anchoring.
- `deepMerge<T>()`: Configuration merging returning `Either<string, T>`, uses `A.traverse(E.Applicative)` for error collection
- `isPlainObject()`: Type predicate using fp-ts Option
- `convertDegreesToRadians()`: Angle conversion
- `calculateAbsoluteCosineSine()`: `|cos| + |sin|` for rotation-aware bounding boxes
- `calculateHalfIndex()`: `(n-1)/2` for centering
- `rotatePoint()`: Point rotation with fp-ts `O.fromPredicate(deg !== 0)` to skip identity
- `createRoundedSquare()`: Hull-based rounded rectangles with `Either` validation

All utilities use fp-ts patterns. No backward compatibility wrappers.

### `build.ts`
Build orchestration with three modes:

**Production mode** (default):
- Outputs to `dist/<profile>-<hash>/`
- Timestamp hash using base36: `now.getTime().toString(36).slice(-6)`
- Preserves build history
- SCAD files only

**Development mode** (`build:dev`):
- Outputs to `dist/` (overwrites)
- Watch mode support via `bun run --watch`
- Open `dist/complete.scad` in OpenSCAD for live preview

**STL mode** (`build:stl`):
- Outputs to `dist/<profile>-<hash>/`
- Generates both SCAD and STL files using `modelGeometry.render()`
- Ready for 3D printing

`listProfiles()` displays all profiles with computed metadata: key counts, layout patterns, switch types, layout modes.

Output includes file tree with sizes:
```
dist/split-36-w92ivk/
├── bottom.scad (7.9K)
├── bottom.stl (52.8K)
├── complete.scad (5.9K)
├── complete.stl (49.6K)
├── top.scad (3.5K)
└── top.stl (50.5K)
```

### `index.ts`
CLI with object literal command routing:
```typescript
const commands: Record<string, () => void> = {
  list: () => listProfiles(),
  build: () => handleBuild(commandArgs[0], false, false),
  'build:dev': () => handleBuild(commandArgs[0], true, false),
  'build:stl': () => handleBuild(commandArgs[0], false, true),
  help: () => console.log('...'),
  undefined: () => { /* handle no command */ },
};

const executeCommand = commands[command ?? 'undefined'];
if (executeCommand) {
  executeCommand();
}
```

Error handling shows available profiles when profile not found.

## scad-js Patterns

```typescript
// 2D to 3D extrusion
const shape2D = createRoundedSquare(width, height);
const shape3D = shape2D.linear_extrude(thickness);

// Boolean operations
const result = difference(
  union(base, feature),
  cutout
);

// Transformations
const positioned = geometry
  .rotate([0, 0, 90])
  .translate([x, y, z]);

// Hull for organic shapes
const rounded = hull(
  circle.translate([x1, y1, 0]),
  circle.translate([x2, y2, 0]),
);

// Minkowski for rounding (keycap generation)
const rounded3D = minkowski(innerCube, roundingCylinder);

// Intersection for bounding (pattern generation)
const bounded = intersection(union(...hexagons), patternBoundary);

// Serialize to OpenSCAD
writeFileSync('output.scad', result.serialize({$fn: 64}));

// Render to STL (async)
const stlData = await result.render({$fn: 64});
writeFileSync('output.stl', stlData);

// Color for visualization
geometry.color('#b54c9e', 0.8);
```

## Output Files

### `top.scad` / `top.stl`
Top plate with:
- Switch sockets (walls + inner mounting ledges)
- Switch socket cutouts (outer + inner holes)
- MCU pocket with reinforcement, pin access, and USB port cutout (optional)
- Connector cutouts
- Walls extending downward from switch plate

### `bottom.scad` / `bottom.stl`
Bottom case with:
- Base floor plate
- Inner walls extending upward
- Silicon pad socket reinforcements and cutouts (optional)
- MagSafe ring structure and groove (optional)
- Bottom pattern (honeycomb/circles/square with exclusion zones, optional)
- Connector cutouts (hulled between bottom and top positions)
- MCU pocket passthrough (optional)

### `complete.scad` / `complete.stl`
Assembly preview showing both parts together with configurable colors, optional switch bodies and keycaps. Top plate translated vertically by `bottomThickness` to show proper assembly.

## Split Wall Architecture

- Top plate: walls extending downward from switch plate
- Bottom case: walls extending upward from floor plate
- Assembly: Walls meet for snap-fit enclosure
- No screws required
- Designed for FDM printing (supports needed for switch cutouts; optional socket structures may also need supports)

## Layout Modes

- **Split** (`mode: 'split'`): Single half generated — user prints two (mirrored manually or by slicer)
- **Unibody** (`mode: 'unibody'`): Both halves generated in one piece, separated by `centerGap`, with automatic Y-axis mirroring

## Usage

```bash
# List available profiles
bun run list

# Build SCAD files (production)
bun run build -- <profile>

# Build with watch mode (development)
bun run build:dev -- <profile>

# Build SCAD + STL files (production)
bun run build:stl -- <profile>

# Show help
bun run help

# Remove generated files
bun run clean

# Lint and format
bun run check
```

## Design Principles

1. **Everything calculated from configuration**: No hardcoded positions
2. **Modular architecture**: Clear separation of concerns (profiles vs code)
3. **Type-safe**: Full TypeScript interfaces, `as const satisfies` for specs, strict tsconfig
4. **Functional patterns**: fp-ts for maintainability and composability
5. **Manufacturing constraints**: Built into geometry generation
6. **Single source of truth**: All dimensions derived from configuration
7. **Auto-discovery**: No manual registration of profiles
8. **Pure functions**: Predictable, testable, composable
9. **Additive then subtractive**: Consistent CSG pattern — build up geometry, then cut away

---

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.

---
> Source: [20lives/flatboard](https://github.com/20lives/flatboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
