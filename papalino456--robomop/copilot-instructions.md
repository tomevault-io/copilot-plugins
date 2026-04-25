## robomop

> Auto-generated from all feature plans. Last updated: 2025-10-01

# RoboMop Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-10-01

## Active Technologies

- **Robot**: Python 3.11+ (001-the-robomop-project)
  - RPLidar SDK, numpy, scipy, asyncio, python-socketio
  - Testing: pytest, pytest-asyncio, unittest.mock
  - Tools: black, pylint, mypy

- **Backend**: Node.js 20.x LTS / TypeScript 5.x (001-the-robomop-project)
  - Express 4.x, Socket.io, TypeORM
  - Testing: Jest, supertest, Socket.io test utilities
  - Tools: ESLint, Prettier

- **Frontend**: TypeScript 5.x / React 18.x / Vite 5.x (001-the-robomop-project)
  - Socket.io-client, Canvas API for rendering
  - Testing: Vitest, React Testing Library, Playwright
  - Tools: ESLint, Prettier

- **Database**: PostgreSQL (production), SQLite (development) (001-the-robomop-project)
  - TypeORM for migrations and entities
  - JSONB for flexible metadata

## Project Structure
```
robot/
├── src/
│   ├── slam/            # Grid SLAM implementation
│   ├── navigation/      # Path planning (A*, coverage)
│   ├── sensors/         # LIDAR, IMU, encoders, ultrasonic, water level
│   ├── control/         # Motor control, PID loops, safety
│   ├── communication/   # WebSocket client, state publishing
│   └── main.py          # Entry point
└── tests/
    ├── contract/        # API/WebSocket contract tests
    ├── integration/     # End-to-end robot tests
    └── unit/            # Module unit tests

backend/
├── src/
│   ├── models/          # TypeORM entities
│   ├── services/        # Business logic
│   ├── api/             # Express REST routes
│   ├── websocket/       # Socket.io handlers
│   └── server.ts        # Entry point
└── tests/
    ├── contract/        # API schema validation
    ├── integration/     # API integration tests
    └── unit/            # Service unit tests

frontend/
├── src/
│   ├── components/      # React components
│   │   ├── MapViewer/   # Canvas map rendering
│   │   ├── Controls/    # Jogging, E-stop, mode selector
│   │   └── StatusPanel/ # Robot status, water, battery
│   ├── services/        # API & WebSocket clients
│   ├── hooks/           # Custom React hooks
│   └── App.tsx          # Entry point
└── tests/
    ├── unit/            # Component tests
    └── e2e/             # Playwright E2E tests
```

## Commands

**Robot (Python)**:
```bash
cd robot
pytest                          # Run all tests
pytest --cov=src --cov-report=html  # Coverage report
black src tests                 # Format code
pylint src                      # Lint code
mypy src                        # Type check
```

**Backend (Node.js/TypeScript)**:
```bash
cd backend
npm test                        # Run all tests
npm run test:coverage           # Coverage report
npm run lint                    # ESLint
npm run format                  # Prettier
npm run typecheck               # TypeScript check
npm run dev                     # Start dev server
```

**Frontend (React/TypeScript)**:
```bash
cd frontend
npm test                        # Vitest unit tests
npm run test:e2e                # Playwright E2E tests
npm run lint                    # ESLint
npm run format                  # Prettier
npm run dev                     # Start dev server (Vite)
npm run build                   # Production build
```

## Code Style

**Python (PEP 8)**:
- snake_case for functions, variables, modules
- PascalCase for classes
- UPPER_CASE for constants
- Docstrings: Google style
- Type hints: Required for all functions
- Max line length: 100 characters

**TypeScript**:
- camelCase for variables, functions
- PascalCase for types, interfaces, classes, React components
- kebab-case for file names
- JSDoc comments for all exported functions
- Strict mode enabled
- Max line length: 100 characters

**API Conventions**:
- REST endpoints: kebab-case (e.g., `/api/v1/robot-state`)
- JSON fields: camelCase (e.g., `batteryLevel`)
- WebSocket events: kebab-case with namespace (e.g., `robot:map-update`)

## Architecture Principles

**Modular Design**:
- Each module has single, clear purpose
- Modules communicate via documented contracts only
- No dependencies on implementation details
- Robot, Backend, Frontend are independent deployable units

**Testing Strategy (TDD)**:
- Write tests FIRST, watch them FAIL
- Then implement to make tests PASS
- Coverage target: 90%+ (critical safety: 100%)
- Test types: Unit, Integration, Contract, E2E

**Communication Protocol**:
- WebSocket (Socket.io) for real-time bidirectional data
- REST API for CRUD operations
- Robot publishes: state (10 Hz), map (5 Hz), sensors
- Backend commands robot via WebSocket events
- Frontend subscribes to robot state via WebSocket

**Performance Requirements**:
- Position accuracy: ±5 cm
- Map update latency: <1 second end-to-end
- Emergency stop: <500ms response
- UI rendering: 60 FPS
- Position updates: 10 Hz
- Map transmission: 5 Hz

## Recent Changes

- 001-the-robomop-project: Initial RoboMop autonomous cleaning robot system with Python robot, Node.js backend, React frontend

## Key Technical Decisions

**SLAM**: Occupancy Grid SLAM + Particle Filter (100-500 particles, 5cm resolution)
**Navigation**: A* pathfinding for exploration, Coverage path planning (boustrophedon) for cleaning
**Sensor Fusion**: EKF combining LIDAR, IMU (complementary filter), and encoders
**Motor Control**: PID loops (100 Hz) for velocity control
**Map Format**: 2D occupancy grid (0=unknown, 1-127=free, 128-255=occupied)
**Map Rendering**: HTML5 Canvas with delta updates for 60 FPS performance

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END --># RoboMop Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-10-01

## Active Technologies

- **Robot**: Python 3.11+ (001-the-robomop-project)
  - RPLidar SDK, numpy, scipy, asyncio, python-socketio
  - Testing: pytest, pytest-asyncio, unittest.mock
  - Tools: black, pylint, mypy

- **Backend**: Node.js 20.x LTS / TypeScript 5.x (001-the-robomop-project)
  - Express 4.x, Socket.io, TypeORM
  - Testing: Jest, supertest, Socket.io test utilities
  - Tools: ESLint, Prettier

- **Frontend**: TypeScript 5.x / React 18.x / Vite 5.x (001-the-robomop-project)
  - Socket.io-client, Canvas API for rendering
  - Testing: Vitest, React Testing Library, Playwright
  - Tools: ESLint, Prettier

- **Database**: PostgreSQL (production), SQLite (development) (001-the-robomop-project)
  - TypeORM for migrations and entities
  - JSONB for flexible metadata

## Project Structure
```
robot/
├── src/
│   ├── slam/            # Grid SLAM implementation
│   ├── navigation/      # Path planning (A*, coverage)
│   ├── sensors/         # LIDAR, IMU, encoders, ultrasonic, water level
│   ├── control/         # Motor control, PID loops, safety
│   ├── communication/   # WebSocket client, state publishing
│   └── main.py          # Entry point
└── tests/
    ├── contract/        # API/WebSocket contract tests
    ├── integration/     # End-to-end robot tests
    └── unit/            # Module unit tests

backend/
├── src/
│   ├── models/          # TypeORM entities
│   ├── services/        # Business logic
│   ├── api/             # Express REST routes
│   ├── websocket/       # Socket.io handlers
│   └── server.ts        # Entry point
└── tests/
    ├── contract/        # API schema validation
    ├── integration/     # API integration tests
    └── unit/            # Service unit tests

frontend/
├── src/
│   ├── components/      # React components
│   │   ├── MapViewer/   # Canvas map rendering
│   │   ├── Controls/    # Jogging, E-stop, mode selector
│   │   └── StatusPanel/ # Robot status, water, battery
│   ├── services/        # API & WebSocket clients
│   ├── hooks/           # Custom React hooks
│   └── App.tsx          # Entry point
└── tests/
    ├── unit/            # Component tests
    └── e2e/             # Playwright E2E tests
```

## Commands

**Robot (Python)**:
```bash
cd robot
pytest                          # Run all tests
pytest --cov=src --cov-report=html  # Coverage report
black src tests                 # Format code
pylint src                      # Lint code
mypy src                        # Type check
```

**Backend (Node.js/TypeScript)**:
```bash
cd backend
npm test                        # Run all tests
npm run test:coverage           # Coverage report
npm run lint                    # ESLint
npm run format                  # Prettier
npm run typecheck               # TypeScript check
npm run dev                     # Start dev server
```

**Frontend (React/TypeScript)**:
```bash
cd frontend
npm test                        # Vitest unit tests
npm run test:e2e                # Playwright E2E tests
npm run lint                    # ESLint
npm run format                  # Prettier
npm run dev                     # Start dev server (Vite)
npm run build                   # Production build
```

## Code Style

**Python (PEP 8)**:
- snake_case for functions, variables, modules
- PascalCase for classes
- UPPER_CASE for constants
- Docstrings: Google style
- Type hints: Required for all functions
- Max line length: 100 characters

**TypeScript**:
- camelCase for variables, functions
- PascalCase for types, interfaces, classes, React components
- kebab-case for file names
- JSDoc comments for all exported functions
- Strict mode enabled
- Max line length: 100 characters

**API Conventions**:
- REST endpoints: kebab-case (e.g., `/api/v1/robot-state`)
- JSON fields: camelCase (e.g., `batteryLevel`)
- WebSocket events: kebab-case with namespace (e.g., `robot:map-update`)

## Architecture Principles

**Modular Design**:
- Each module has single, clear purpose
- Modules communicate via documented contracts only
- No dependencies on implementation details
- Robot, Backend, Frontend are independent deployable units

**Testing Strategy (TDD)**:
- Write tests FIRST, watch them FAIL
- Then implement to make tests PASS
- Coverage target: 90%+ (critical safety: 100%)
- Test types: Unit, Integration, Contract, E2E

**Communication Protocol**:
- WebSocket (Socket.io) for real-time bidirectional data
- REST API for CRUD operations
- Robot publishes: state (10 Hz), map (5 Hz), sensors
- Backend commands robot via WebSocket events
- Frontend subscribes to robot state via WebSocket

**Performance Requirements**:
- Position accuracy: ±5 cm
- Map update latency: <1 second end-to-end
- Emergency stop: <500ms response
- UI rendering: 60 FPS
- Position updates: 10 Hz
- Map transmission: 5 Hz

## Recent Changes

- 001-the-robomop-project: Initial RoboMop autonomous cleaning robot system with Python robot, Node.js backend, React frontend

## Key Technical Decisions

**SLAM**: Occupancy Grid SLAM + Particle Filter (100-500 particles, 5cm resolution)
**Navigation**: A* pathfinding for exploration, Coverage path planning (boustrophedon) for cleaning
**Sensor Fusion**: EKF combining LIDAR, IMU (complementary filter), and encoders
**Motor Control**: PID loops (100 Hz) for velocity control
**Map Format**: 2D occupancy grid (0=unknown, 1-127=free, 128-255=occupied)
**Map Rendering**: HTML5 Canvas with delta updates for 60 FPS performance

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/papalino456) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
