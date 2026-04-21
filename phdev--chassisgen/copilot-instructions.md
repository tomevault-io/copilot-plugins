## chassisgen

> ChassisForge is an AI-powered parametric robot chassis design tool for mechanical engineers. It accelerates the early conceptual design phase by generating structured design briefs, enabling rapid exploration of parametric design spaces, and providing real-time physics-based scoring — all backed by a curated database of real, purchasable components.

# ChassisForge — AI-Powered Robot Chassis Design Explorer

## Project Overview

ChassisForge is an AI-powered parametric robot chassis design tool for mechanical engineers. It accelerates the early conceptual design phase by generating structured design briefs, enabling rapid exploration of parametric design spaces, and providing real-time physics-based scoring — all backed by a curated database of real, purchasable components.

This is NOT a chassis generator that replaces MEs. It's a "first draft" tool that does the tedious constraint-checking homework so MEs can start CAD work with validated configurations instead of blank screens.

### Core Value Proposition

- Describe a task → get a structured design brief with real component recommendations
- Interactively explore parametric chassis configurations with sliders
- See real-time physics scores (stability, runtime, turning radius, payload capacity) update as you adjust
- Export a validated starting point: STEP file, BOM, URDF, design brief PDF

### Target User

Mechanical engineers, robotics startup teams, and advanced hobbyists designing wheeled/tracked mobile robot platforms for indoor logistics, inspection, delivery, agriculture, and similar applications.

-----

## Architecture

### System Diagram

```
┌─────────────────────────────────────────────────────┐
│                    Frontend (React)                  │
│  ┌─────────────┐ ┌──────────────┐ ┌──────────────┐  │
│  │ Task Intake  │ │  Parametric  │ │  Scoring     │  │
│  │ (Chat/Form) │ │  Explorer    │ │  Dashboard   │  │
│  │             │ │  (Three.js)  │ │  (Recharts)  │  │
│  └──────┬──────┘ └──────┬───────┘ └──────┬───────┘  │
│         │               │                │           │
│         └───────────────┼────────────────┘           │
│                         │ REST API                   │
└─────────────────────────┼───────────────────────────┘
                          │
┌─────────────────────────┼───────────────────────────┐
│                  Backend (FastAPI)                    │
│  ┌──────────────┐ ┌────────────┐ ┌───────────────┐  │
│  │ Brief Gen    │ │ Parametric │ │ Simulation    │  │
│  │ (LLM + DB)  │ │ Geometry   │ │ Engine        │  │
│  │             │ │ (CadQuery) │ │ (analytic +   │  │
│  │             │ │            │ │  PyBullet)    │  │
│  └──────┬──────┘ └─────┬──────┘ └──────┬────────┘  │
│         │              │               │             │
│         └──────────────┼───────────────┘             │
│                        │                             │
│              ┌─────────┴─────────┐                   │
│              │ Component Database│                   │
│              │ (SQLite)          │                   │
│              └───────────────────┘                   │
└─────────────────────────────────────────────────────┘
```

### Tech Stack

- **Frontend**: React 18 + TypeScript, Three.js (via @react-three/fiber + @react-three/drei), Recharts for scoring dashboard, Tailwind CSS
- **Backend**: Python 3.11+, FastAPI, CadQuery for parametric CAD, PyBullet for advanced sim (Phase 4)
- **Database**: SQLite for component catalog
- **LLM**: Anthropic Claude API for design brief generation (Phase 2+)
- **Export**: CadQuery → STEP/STL, ReportLab or WeasyPrint → PDF, custom → URDF
- **Deployment**: Vercel (all phases)

### Deployment (Vercel)

Deploy to Vercel from day one. This covers all phases without migration.

**Phase 1 (client-side only)**:

- Vite React app deployed as static site to Vercel
- No backend needed — all physics runs in the browser
- Repo structure: monorepo with `frontend/` directory
- `vercel.json` at repo root points to `frontend/` as the build target
- Auto-deploys on push to `main`, preview deploys on PRs

**Phase 2+ (frontend + backend)**:

- Frontend remains on Vercel as before
- Backend (FastAPI) deployed as Vercel Serverless Functions via `api/` directory, OR as a separate service on Railway/Fly.io if serverless doesn't fit (CadQuery and PyBullet have heavy dependencies)
- If using separate backend host: configure CORS and set `NEXT_PUBLIC_API_URL` env var in Vercel
- SQLite works for dev; for production with a separate backend, use Turso (libSQL) or Postgres on Railway

**Repo structure**:

```
chassisforge/
├── vercel.json              # Vercel project config
├── CLAUDE.md                # This file
├── frontend/
│   ├── package.json
│   ├── vite.config.ts
│   ├── tsconfig.json
│   ├── index.html
│   └── src/
│       └── ...
├── backend/                 # Phase 2+
│   ├── requirements.txt
│   ├── main.py
│   └── ...
└── api/                     # Vercel serverless functions (Phase 2+ alternative)
    └── ...
```

**`vercel.json`** (Phase 1):

```json
{
  "buildCommand": "cd frontend && npm run build",
  "outputDirectory": "frontend/dist",
  "framework": "vite"
}
```

**GitHub Actions CI** (optional but recommended):

- Lint + type-check on every PR
- Run physics engine unit tests
- Preview deploy URL posted as PR comment (Vercel does this automatically)

-----

## Component Database Schema

The component database is the backbone of the tool. Every recommendation must reference real, purchasable parts.

### Tables

```sql
-- Motors: DC gear motors, brushless, steppers
CREATE TABLE motors (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    type TEXT NOT NULL,  -- 'dc_gear', 'brushless', 'stepper'
    voltage_v REAL,
    stall_torque_nm REAL,
    no_load_rpm REAL,
    stall_current_a REAL,
    weight_g REAL,
    shaft_diameter_mm REAL,
    body_length_mm REAL,
    body_diameter_mm REAL,
    has_encoder BOOLEAN,
    encoder_cpr INTEGER,
    price_usd REAL,
    vendor TEXT,
    vendor_url TEXT,
    datasheet_url TEXT
);

-- Batteries
CREATE TABLE batteries (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    chemistry TEXT NOT NULL,  -- 'lipo', 'liion', 'lifepo4', 'nimh'
    voltage_v REAL,
    capacity_mah REAL,
    max_discharge_c REAL,
    weight_g REAL,
    length_mm REAL,
    width_mm REAL,
    height_mm REAL,
    price_usd REAL,
    vendor TEXT,
    vendor_url TEXT
);

-- Compute boards
CREATE TABLE compute_boards (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    type TEXT NOT NULL,  -- 'sbc', 'mcu', 'gpu_module'
    power_draw_w REAL,
    idle_power_w REAL,
    length_mm REAL,
    width_mm REAL,
    height_mm REAL,
    weight_g REAL,
    has_gpio BOOLEAN,
    has_usb3 BOOLEAN,
    has_csi BOOLEAN,
    gpu_tflops REAL,
    price_usd REAL,
    vendor TEXT,
    vendor_url TEXT
);

-- Sensors
CREATE TABLE sensors (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    type TEXT NOT NULL,  -- 'depth_camera', 'lidar_2d', 'lidar_3d', 'imu', 'ultrasonic', 'ir_tof', 'encoder'
    fov_h_deg REAL,
    fov_v_deg REAL,
    range_min_m REAL,
    range_max_m REAL,
    weight_g REAL,
    length_mm REAL,
    width_mm REAL,
    height_mm REAL,
    power_draw_w REAL,
    interface TEXT,  -- 'usb', 'i2c', 'spi', 'uart', 'ethernet'
    price_usd REAL,
    vendor TEXT,
    vendor_url TEXT
);

-- Wheels
CREATE TABLE wheels (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    type TEXT NOT NULL,  -- 'rubber', 'mecanum', 'omni', 'pneumatic', 'track'
    diameter_mm REAL,
    width_mm REAL,
    hub_bore_mm REAL,
    load_rating_kg REAL,
    weight_g REAL,
    price_usd REAL,
    vendor TEXT,
    vendor_url TEXT
);

-- Motor drivers
CREATE TABLE motor_drivers (
    id TEXT PRIMARY KEY,
    name TEXT NOT NULL,
    channels INTEGER,
    max_voltage_v REAL,
    max_current_a REAL,
    has_current_sense BOOLEAN,
    interface TEXT,  -- 'pwm', 'i2c', 'serial', 'canbus'
    length_mm REAL,
    width_mm REAL,
    height_mm REAL,
    weight_g REAL,
    price_usd REAL,
    vendor TEXT,
    vendor_url TEXT
);
```

### Seeding Strategy

Start with 50-100 components covering the most common configurations:

- **Motors**: 5-8 DC gear motors (Pololu 37D, N20 series, JGB37), 3-4 brushless (T-Motor, Turnigy)
- **Batteries**: 5-6 LiPo packs (2S-6S, 2200-10000mAh), 2-3 LiFePO4
- **Compute**: RPi 5, Jetson Orin Nano, ESP32, Arduino Mega, Khadas VIM4
- **Sensors**: RealSense D435, OAK-D Lite, RPLidar A1, RPLidar A2M12, MPU6050, VL53L1X
- **Wheels**: 5-8 common sizes (65mm-150mm rubber, mecanum sets, omni wheels)
- **Drivers**: TB6612FNG, L298N, Cytron MDD10A, ODrive

-----

## Parametric Design Space

### Chassis Parameters (Phase 1 — Differential Drive Indoor Mobile Base)

```typescript
interface ChassisParams {
  // Frame dimensions
  frameLength_mm: number;      // 150-800, step 5
  frameWidth_mm: number;       // 100-600, step 5
  frameHeight_mm: number;      // 30-200, step 5
  groundClearance_mm: number;  // 10-150, step 5
  frameThickness_mm: number;   // 2-10, step 0.5
  frameMaterial: 'pla' | 'petg' | 'aluminum_6061' | 'steel' | 'carbon_fiber';

  // Drivetrain
  drivetrainType: 'differential' | 'ackermann' | 'omni_4' | 'mecanum_4';
  wheelDiameter_mm: number;    // from selected wheel component
  wheelWidth_mm: number;       // from selected wheel component
  motorMountInset_mm: number;  // how far inboard the motors sit

  // Component placements (X, Y relative to chassis center, Z relative to chassis floor)
  batteryPosition: { x: number; y: number; z: number };
  computePosition: { x: number; y: number; z: number };
  sensorMounts: Array<{
    sensorId: string;
    position: { x: number; y: number; z: number };
    rotation: { pitch: number; yaw: number };  // degrees
  }>;

  // Payload
  payloadMountHeight_mm: number;
  maxPayload_kg: number;
}
```

### Simulation Scores (computed analytically, update on every param change)

```typescript
interface SimulationScores {
  // Static stability
  cgPosition: { x: number; y: number; z: number };  // mm
  stabilityMargin_pct: number;     // 0-100, distance from CG to nearest tip edge
  tipAngle_deg: number;            // slope angle at which robot tips (front/rear/side)

  // Mobility
  turningRadius_mm: number;        // 0 for diff drive pivot, >0 for ackermann
  maxSpeed_mps: number;            // from motor RPM and wheel diameter
  maxGradeability_deg: number;     // steepest slope it can climb under load
  stepClearance_mm: number;        // max obstacle height it can roll over

  // Power
  estimatedRuntime_hrs: number;    // at nominal speed + sensor + compute draw
  totalPowerDraw_w: number;        // all systems
  motorPowerDraw_w: number;
  electronicsPowerDraw_w: number;

  // Mass
  totalMass_kg: number;
  massBudget: Record<string, number>;  // subsystem -> kg
  payloadCapacity_kg: number;          // before stability degrades below threshold

  // Fit checks
  allComponentsFit: boolean;
  interferenceWarnings: string[];      // "Battery overlaps motor driver by 12mm"
  doorwayFit: { standard_762mm: boolean; wide_914mm: boolean };
}
```

-----

## Phase Plan

### Phase 1: Parametric Explorer MVP

**Goal**: Interactive 3D chassis explorer with real-time analytic scoring for a single archetype (differential drive indoor mobile base).

**Scope**:

- React frontend with Three.js 3D viewport showing parametric chassis
- Slider panel for all chassis parameters
- Real-time scoring dashboard (stability, runtime, turning, mass budget, fit checks)
- Hardcoded component set (no database yet — just 3-5 motors, 2-3 batteries, 1-2 compute boards, a few wheels and sensors embedded in code)
- Components rendered as color-coded bounding boxes in the 3D viewport
- Chassis frame rendered as a translucent plate structure
- No backend — everything runs client-side (physics calcs in JS/TS)

**Key files**:

```
frontend/
├── src/
│   ├── App.tsx                    # Main layout: viewport + sidebar + dashboard
│   ├── components/
│   │   ├── Viewport.tsx           # Three.js canvas with chassis model
│   │   ├── ChassisModel.tsx       # Parametric 3D chassis geometry
│   │   ├── ComponentBlock.tsx     # Colored bounding box for a component
│   │   ├── WheelModel.tsx         # Wheel geometry with motor housing
│   │   ├── SliderPanel.tsx        # All parameter controls
│   │   ├── ComponentSelector.tsx  # Dropdowns for motor/battery/etc selection
│   │   ├── ScoringDashboard.tsx   # Gauges and charts for sim scores
│   │   ├── MassBudgetChart.tsx    # Stacked bar or pie for mass breakdown
│   │   └── StabilityIndicator.tsx # Visual CG position + tip margin overlay
│   ├── engine/
│   │   ├── physics.ts             # All analytic simulation calculations
│   │   ├── cgCalculator.ts        # Center of gravity computation
│   │   ├── stabilityAnalysis.ts   # Tip angle, stability margin
│   │   ├── powerBudget.ts         # Runtime and power draw estimation
│   │   ├── mobilityCalcs.ts       # Speed, turning radius, gradeability
│   │   └── interferenceCheck.ts   # Bounding box overlap detection
│   ├── data/
│   │   ├── components.ts          # Hardcoded component catalog
│   │   └── defaults.ts            # Default chassis parameters per archetype
│   └── types/
│       ├── chassis.ts             # ChassisParams, SimulationScores types
│       └── components.ts          # Motor, Battery, Sensor, etc. types
```

**Acceptance criteria**:

- [ ] User can adjust frame dimensions and see 3D model update in real time
- [ ] User can select from hardcoded components (motor, battery, compute, wheels)
- [ ] Components appear as labeled colored blocks in 3D view at their positions
- [ ] CG is visualized as a marker in the 3D view
- [ ] Stability margin, runtime, max speed, mass budget update live on slider change
- [ ] Interference warnings appear when components overlap
- [ ] Doorway fit check shows pass/fail for standard sizes

### Phase 2: Design Brief Generator

**Goal**: LLM-powered task intake that produces a structured design brief with component recommendations from a real database.

**Scope**:

- Python FastAPI backend
- SQLite component database with 50-100 real parts (seeded from curated JSON)
- Conversational task intake UI (natural language or structured form)
- LLM (Claude API) decomposes task into constraints, queries component DB, generates design brief
- Design brief output: architecture recommendation, component selections with rationale, mass/power budgets, flagged risks
- Brief auto-populates the parametric explorer from Phase 1
- Design brief exportable as PDF

**Key files**:

```
backend/
├── main.py                      # FastAPI app
├── routers/
│   ├── brief.py                 # POST /brief/generate — task → design brief
│   ├── components.py            # GET /components/{type} — query catalog
│   └── export.py                # POST /export/pdf, /export/step, etc.
├── services/
│   ├── brief_generator.py       # LLM orchestration for brief generation
│   ├── component_selector.py    # Constraint-based component filtering + ranking
│   └── prompt_templates.py      # System prompts for brief generation
├── models/
│   ├── brief.py                 # DesignBrief pydantic model
│   ├── task.py                  # TaskConstraints pydantic model
│   └── components.py            # DB models for all component types
├── db/
│   ├── init_db.py               # Create tables and seed data
│   ├── seed_data/               # JSON files with component specs
│   │   ├── motors.json
│   │   ├── batteries.json
│   │   ├── compute_boards.json
│   │   ├── sensors.json
│   │   ├── wheels.json
│   │   └── motor_drivers.json
│   └── queries.py               # DB query helpers
└── tests/
    ├── test_brief_generator.py
    ├── test_component_selector.py
    └── test_physics.py
```

**Acceptance criteria**:

- [ ] User describes a task in natural language, gets a structured brief in <30 seconds
- [ ] Brief includes specific part numbers, prices, and datasheet links
- [ ] Brief includes mass budget, power budget, and flagged risks
- [ ] Clicking "Load into Explorer" populates all Phase 1 sliders with brief's recommendations
- [ ] Component database has at least 50 real parts with accurate specs
- [ ] Brief exportable as formatted PDF

### Phase 3: Export Pipeline

**Goal**: Generate fabrication-ready and sim-ready output files from the configured chassis.

**Scope**:

- CadQuery backend generates parametric STEP and STL files
- BOM export as CSV and formatted PDF
- URDF export for ROS/Gazebo integration
- DXF export for laser-cut flat chassis plates (for aluminum/acrylic fabrication)
- Wiring diagram suggestion (which pins, which connectors)

**Key files**:

```
backend/
├── services/
│   ├── cad_generator.py         # CadQuery parametric chassis model
│   ├── urdf_generator.py        # URDF XML generation with collision/visual meshes
│   ├── bom_generator.py         # BOM from selected components
│   ├── dxf_generator.py         # 2D plate outlines for laser cutting
│   └── wiring_diagram.py        # Pin assignment + connector recommendations
```

**Acceptance criteria**:

- [ ] STEP file opens correctly in FreeCAD/Fusion 360 with accurate dimensions
- [ ] STL file is manifold and 3D-printable without repair
- [ ] URDF loads in RViz and Gazebo with correct mass/inertia properties
- [ ] BOM PDF includes all components with quantities, prices, and vendor links
- [ ] DXF export produces cuttable plate outlines for the frame

### Phase 4: Advanced Simulation

**Goal**: Physics-engine-backed simulation for dynamic scenarios beyond analytic estimates.

**Scope**:

- PyBullet integration for rigid-body dynamics simulation
- Terrain traversal test: flat, ramp, steps, rough surface
- Dynamic stability: acceleration, braking, turning at speed
- Payload shift simulation: how does a shifting load affect stability?
- Simulation replay viewable in the frontend (recorded trajectory playback)
- Scored comparison: run same terrain course with different configurations

**Key files**:

```
backend/
├── services/
│   ├── simulation/
│   │   ├── pybullet_runner.py       # Simulation setup and execution
│   │   ├── terrain_generator.py     # Procedural test terrain generation
│   │   ├── chassis_to_urdf.py       # Convert live config to sim-ready URDF
│   │   ├── trajectory_recorder.py   # Record positions/orientations per timestep
│   │   └── scoring.py               # Score simulation outcomes
frontend/
├── src/
│   ├── components/
│   │   ├── SimulationPanel.tsx       # Trigger sim, show progress
│   │   ├── TrajectoryPlayer.tsx      # Replay recorded sim in Three.js
│   │   └── ComparisonView.tsx        # Side-by-side config scoring
```

**Acceptance criteria**:

- [ ] User clicks "Run Terrain Test" and sees simulation complete in <30 seconds
- [ ] Sim results include: completed course (y/n), time, max tilt angle, energy consumed
- [ ] Trajectory replay shows the robot navigating the terrain in 3D
- [ ] User can compare 2+ configurations on the same terrain side-by-side
- [ ] Sim scores correlate reasonably with analytic estimates (validation)

### Phase 5: Additional Archetypes (Future)

**Goal**: Expand beyond differential drive indoor base to other common robot form factors.

**Planned archetypes**:

- **Ackermann steering outdoor rover** — 4-wheel, front-steer, for outdoor terrain (agriculture, inspection)
- **Mecanum/omni holonomic base** — 4-wheel omnidirectional for tight indoor spaces (warehouse, lab)
- **Tracked platform** — continuous tracks for rough terrain, stairs (mining, search & rescue)
- **6-wheel rocker-bogie** — Mars rover-style suspension for uneven terrain
- **Arm-on-base composite** — mobile base + mounted robot arm, with reachability analysis

Each archetype adds:

- New parametric space (e.g., ackermann adds steering angle, kingpin offset; tracks add track width, sprocket size, idler position)
- Archetype-specific simulation scoring (e.g., rocker-bogie gets articulation angle scoring)
- Archetype-specific CadQuery geometry generator
- Archetype-specific URDF template

### Phase 6: Community & Iteration (Future)

**Goal**: Close the design-to-reality feedback loop.

**Planned features**:

- Save/load/share configurations via URL or file
- Community gallery of validated configurations with real-world test results
- "I built this" workflow: upload photos/video of physical build, tag the configuration
- Real-world test data feedback: actual vs. predicted runtime, actual vs. predicted stability
- Component database contributions from community

-----

## Coding Standards

### General

- TypeScript strict mode throughout the frontend
- Python type hints everywhere in the backend
- All physics calculations must include units in variable names (e.g., `mass_kg`, `torque_nm`, `speed_mps`)
- All physics functions must have docstrings explaining the formula used and its assumptions
- No magic numbers — all constants defined in a constants file with source citations

### Frontend

- Use @react-three/fiber and @react-three/drei for Three.js integration — do not use raw Three.js
- All 3D dimensions in millimeters internally, converted to Three.js units (meters) at the rendering layer only
- Scoring dashboard updates must feel instant (<16ms) — all analytic physics runs client-side, no API calls for slider changes
- Use Zustand for state management (single store for chassis params + component selections + scores)
- Responsive layout: viewport takes 60% width, slider panel 20%, scoring dashboard 20%

### Backend

- FastAPI with async endpoints
- Pydantic models for all API request/response schemas
- CadQuery operations must be idempotent — same params always produce same geometry
- Component database queries must support filtering by any physical constraint (e.g., "motors with stall torque > 0.5 Nm and body diameter < 30mm")

### Testing

- Physics engine functions: unit tests with hand-calculated expected values
- CG calculation: test with known symmetric and asymmetric mass distributions
- Component database: test constraint queries return correct results
- Frontend: visual regression tests are nice-to-have but not required for MVP

-----

## Key Engineering Decisions

### Why client-side physics for Phase 1?

The analytic calculations (CG, stability margin, power budget, turning radius) are simple arithmetic — they don't need a server. Running them client-side means the scoring dashboard updates at 60fps as the user drags sliders, which is the core UX differentiator. Server-side physics is only introduced in Phase 4 for PyBullet simulations that actually need a physics engine.

### Why CadQuery over OpenSCAD?

CadQuery produces native STEP files (industry-standard CAD interchange format) directly, while OpenSCAD only exports STL/AMF. MEs importing into Fusion 360 or SolidWorks want STEP. CadQuery also has a Python API, which integrates cleanly with the FastAPI backend.

### Why start with differential drive?

It's the most common indoor mobile robot architecture, has the simplest parametric space (no steering geometry), and covers the highest-value use case (warehouse/logistics bots). Ackermann and omni add complexity that's not needed to validate the core UX.

### Why a curated component database instead of scraping?

Accuracy matters more than breadth. An ME will immediately distrust the tool if a motor's specs are wrong. 50 hand-verified components beat 5,000 scraped-and-maybe-correct ones. Expansion is additive — each new component just needs a JSON entry with verified specs.

-----

## References and Resources

### Simulation & Physics

- [EvoGym](https://github.com/EvolutionGym/evogym) — soft robot co-design benchmark (inspiration for parametric design space concept)
- [PyBullet quickstart](https://pybullet.org/) — Phase 4 simulation engine
- [MuJoCo](https://mujoco.org/) — alternative sim engine if PyBullet proves limiting
- [URDF spec](http://wiki.ros.org/urdf/XML) — for Phase 3 export

### CAD Generation

- [CadQuery docs](https://cadquery.readthedocs.io/) — parametric CAD in Python
- [CadQuery examples](https://github.com/CadQuery/cadquery) — reference for chassis plate modeling

### Prior Art in AI Robot Design

- LASeR (ICLR 2025) — LLM-aided evolutionary search for robot design
- RoboMoRe (2025) — joint morphology + reward optimization
- MIT CSAIL diffusion robot design (ICRA 2025) — generative AI for physical robot geometry
- RoboCrafter-QA — benchmark for LLM robot design reasoning

### Frontend

- [@react-three/fiber](https://docs.pmnd.rs/react-three-fiber) — React renderer for Three.js
- [@react-three/drei](https://github.com/pmndrs/drei) — helpers for R3F (OrbitControls, Grid, etc.)
- [Recharts](https://recharts.org/) — React charting library for scoring dashboard
- [Zustand](https://github.com/pmndrs/zustand) — lightweight state management

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
