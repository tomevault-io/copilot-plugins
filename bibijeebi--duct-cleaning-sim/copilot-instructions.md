## duct-cleaning-sim

> A browser-playable 3D first-person training simulator for commercial duct cleaning technicians. Built for Carolina Quality Air. The goal: someone plays through this game thoroughly, shows up to a real job site, and fails at nothing.

# Duct Cleaning Simulator

## What This Is

A browser-playable 3D first-person training simulator for commercial duct cleaning technicians. Built for Carolina Quality Air. The goal: someone plays through this game thoroughly, shows up to a real job site, and fails at nothing.

## Tech Stack

- **Engine**: Babylon.js (latest v7+) — NOT Three.js. Babylon has built-in collision detection, gravity, FreeCamera (FPS-style WASD+mouse), physics plugins, action manager, and audio. Do not use Three.js.
- **Build**: Vite + vanilla TypeScript. No React, no framework. Babylon manages its own canvas and DOM.
- **Package manager**: npm
- **Deployment**: Static build → Cloudflare Pages
- **Style**: Industrial training aesthetic. Safety yellow (#FFD700) accents, dark grays (#1a1a1a, #2d2d2d), blueprint-style diagrams. Functional, utilitarian, not pretty. Think OSHA manual meets video game.

## Architecture

```
src/
  main.ts              # Entry point, engine init, scene loader
  scenes/
    MainMenu.ts        # Title screen, scenario select
    BuildingScene.ts   # The main 3D walkable environment
    DuctCrossSection.ts # 2D overlay for inside-duct cleaning view
  systems/
    PlayerController.ts # FPS movement, pointer lock, interaction raycasting
    GameState.ts       # Phase tracking, scoring, task completion
    EquipmentSystem.ts # Inventory, tool selection, equipment interaction
    HVACSystem.ts      # Duct network model, airflow direction, debris state
    ScoringSystem.ts   # Points, deductions, bonuses, final grade
    PatchingSystem.ts  # Access hole cutting, patching to code
    PressureWashSystem.ts # Grill cleaning sub-task
  ui/
    HUD.ts             # Babylon GUI overlay — task list, score, current tool
    EquipmentSelect.ts # Van/trailer loadout screen
    PhaseOverlay.ts    # Phase transition screens
    ScoreCard.ts       # End-of-job report
  models/
    BuildingGenerator.ts # Procedural commercial building floor
    DuctNetwork.ts     # Duct layout generation (trunk, branches, VAVs)
    Equipment.ts       # 3D representations of tools/equipment
  data/
    scenarios.ts       # Building configs, difficulty, system types
    equipment.ts       # Tool definitions, compatibility rules
    problems.ts        # Random event definitions
    scoring.ts         # Point values, deduction rules
  utils/
    constants.ts       # Colors, sizes, physics params
    audio.ts           # Sound effects manager
```

## Game Design

### Core Loop

Player receives a job ticket → loads equipment from van/trailer → enters building → assesses HVAC system → sets up (plastic, negative air, tubing) → cleans ducts (returns first, then supply) → patches access holes → pressure washes grills → cleans coils → reinstalls everything → cleanup → scored

### The 6 Phases

1. **Pre-Job**: Read job ticket. Select equipment loadout from van/trailer inventory. Vehicle check (random: low fuel, flat tire, check engine). Forgetting items = time penalty later.

2. **Arrival & Assessment**: Enter building. Find air handler. Identify system type (split, RTU, PTAC, VAV). Count registers (supply vs return). Plan access points. Lay plastic sheeting under work areas.

3. **Setup**: Connect tubing from negative air machine (squirrel cage) to trunk line. Run compressor hose for agitation wand. Quick-connect sections of 8-10" tubing (duct tape joints). Position portable vacuums at access points.

4. **Execution**: Clean returns first (upstream), then supply. Snake agitation wand with compressed air heads (forward/reverse). Cut access holes where needed (every 12ft or at turns). Use correct technique per material:
   - Rigid metal: aggressive agitation OK
   - Flex duct: gentle only (aggressive = collapse, -25 points)
   - Ductboard: air wash only (contact = fiber release, -20 points)
   
5. **Completion**: Patch all access holes to code:
   - 1" holes → pop plugs
   - 8" holes → sheet metal square patch, screws at corners and sides, mastic around seam, mastic on patch face, roll insulation back, seal with FSK tape (not duct tape)
   - Pressure wash all grills/registers (tarp down, garden hose to pressure washer, scrub stubborn ones)
   - Clean coils in air handler (coil cleaner spray + degreaser/Simple Green diluted)
   - Replace filters
   - Reinstall all registers/grills
   
6. **Cleanup**: Pull plastic sheeting. Sweep/dustpan debris. Pack equipment. Final walkthrough.

### HVAC System Model

The simulator must accurately model:

- **Air Handler**: Main unit that conditions air. Contains coils (evaporator), blower, filter rack.
- **Trunk Line**: Main duct leaving the air handler. Carries conditioned air.
- **Branch Ducts**: Smaller ducts splitting off trunk to individual zones/rooms.
- **VAV Boxes** (Variable Air Volume): Smart damper boxes between trunk and branch ducts. Control airflow per zone via thermostat. Have internal damper, possibly reheat coil. Player cleans these.
- **Supply Registers**: Where conditioned air enters rooms. Smaller, more numerous, may have directional louvers. Should be relatively clean unless mold/filter neglect.
- **Return Grills**: Where room air returns to air handler. Larger, fewer. Always grosser (unfiltered room air).
- **Airflow Direction**: Supply = FROM air handler TO rooms. Return = FROM rooms TO air handler. Clean returns FIRST (upstream) to avoid recontaminating cleaned supply ducts.

### Equipment

Three tiers of vacuum/air equipment:

1. **Truck-mounted compressors**: Small air hose → feeds snake wand agitation heads. The offense — blasts debris loose. Forward head sprays air ahead (pushes debris downstream). Reverse head sprays backward (propels wand forward through duct).

2. **Gas-powered squirrel cage units** (portable negative air machines): Big 8-10" tubing connection. The defense — sucks everything out. Portable so they can go upstairs. Require gas can with primer bulb.

3. **Portable vacuums**: Small units for catching debris at access points during agitation.

Other equipment: drop cloths/plastic sheeting, tin snips/hole cutters, sheet metal patches, pop plugs, screws, screw gun, mastic/putty, FSK tape (foil-faced, looks like insulation — NOT duct tape), pressure washer, garden hose, coil cleaner spray, degreaser, brushes, brooms, dustpans, masks/PPE.

### Duct Cross-Section View

When player cuts an access hole and inserts the wand, camera switches to a **2D side-view** showing:
- Duct interior cross-section
- Debris particles on duct walls
- The wand snaking through
- Airflow direction arrows (toward negative air machine)
- Particle effects as debris gets knocked loose and flows toward suction
- Visual feedback on technique (too aggressive on flex = duct walls deforming)

This avoids the impossible problem of rendering inside ductwork in 3D.

### Problem Injection (Random Events)

Each scenario randomly includes 2-3 of:
- Painted-over screws on registers (need different removal approach)
- Stripped screw heads
- Registers caulked to ceiling (need scoring knife)
- Brittle/old registers that crack during removal
- Breaker trips when running equipment (find panel, reset)
- Wrong electrical adapter needed
- Mold discovery → STOP WORK, notify supervisor
- Asbestos-era building indicators → STOP WORK, notify supervisor
- Dead animal in ductwork → special removal protocol
- Collapsed flex duct section
- Fire damper in duct run (do not disturb)
- Customer/building manager interruption (dialogue tree)
- Missing tool that was left in the van

### Scoring

Start at 100 points.

**Deductions:**
- Wrong cleaning order (supply before return): -15
- Forgot plastic sheeting: -10
- Wrong tool for material type: -10
- Aggressive on flex duct (collapse): -25
- Contact on ductboard (fiber release): -20
- Missed register: -5
- Bad patch (wrong screws, no mastic, wrong tape): -10 per violation
- Used duct tape instead of FSK tape: -10
- Didn't clean coils: -15
- Didn't replace filters: -10
- Ignored hazard (mold/asbestos): -30
- Forgot equipment in van (time penalty): -5

**Bonuses:**
- Correct hazard protocol: +5
- All registers identified on first survey: +5
- Perfect patch (all elements correct): +5
- Under par time: +10
- Clean final walkthrough: +5

### Scenarios (MVP: implement scenario 1 first)

1. **Commercial Office** (Difficulty: 1): Single-floor office. Split system with air handler in mechanical room. 8 supply registers, 3 return grills. Rigid metal ductwork. Truck-mounted compressor accessible.

2. **Multi-Story Courthouse** (Difficulty: 2): PTAC/fan coil wall units. Short duct runs. Portable equipment required (can't park truck on 3rd floor). Multiple units to clean. Based on Durham County Courthouse experience.

3. **Industrial/Institutional** (Difficulty: 3): Large RTU on roof. VAV boxes throughout. Mix of rigid, flex, and ductboard. Long trunk runs. Fire dampers present. Potential hazard encounters.

## Coding Standards

- TypeScript strict mode
- No `any` types
- Babylon.js imports from `@babylonjs/core`, `@babylonjs/gui`, `@babylonjs/loaders`
- All game state in GameState singleton — no scattered global state
- Systems communicate through events (Babylon's Observable pattern)
- Constants in `data/` files, not magic numbers in logic
- Every interactive object gets a descriptive `name` property for raycasting identification
- Meshes that need collision get `checkCollisions = true`
- Camera uses `camera.applyGravity = true` and `camera.checkCollisions = true`

## Build & Run

```bash
npm install
npm run dev      # Vite dev server at localhost:5173
npm run build    # Production build to dist/
npm run preview  # Preview production build
```

## Testing

Manual testing primary. Automated tests for:
- Scoring calculation correctness
- Phase state machine transitions
- Equipment compatibility matrix (tool + material → result)
- Duct network generation validity

## Commit Convention

- `feat/`: New features (`feat/building-layout`, `feat/equipment-system`)
- `fix/`: Bug fixes
- `refactor/`: Code restructuring
- `docs/`: Documentation only

## What NOT To Do

- Do NOT use React or any UI framework. Babylon GUI handles all in-game UI.
- Do NOT use Three.js. Period.
- Do NOT use localStorage (won't work in some deployment contexts). Use in-memory state.
- Do NOT make the building a single huge mesh. Use modular room/wall/ceiling pieces for collision.
- Do NOT try to render the inside of ducts in 3D. Use the 2D cross-section overlay.
- Do NOT hardcode building layouts. Use the procedural generator with scenario configs.
- Do NOT skip collision detection on walls/floors/ceilings. Player must not clip through geometry.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bibijeebi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
