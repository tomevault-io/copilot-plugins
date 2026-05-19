## ytoledo

> This is a full-stack application with:

# AGENTS.md - OpenCode Agent Guidelines

## Project Overview

This is a full-stack application with:
- **Backend**: Node.js 20.11.0 + Express + TypeScript + Sequelize (MySQL)
- **Frontend**: Angular 17.1.0 + PrimeNG + Socket.io
- **Communication**: Real-time via Socket.io

Project root: `/home/jmolero/Repository/base/1_Project/1_Sources/`

Agent context docs:
- `1_Project/2_Docs/agent-context/architecture.md`
- `1_Project/2_Docs/agent-context/security-hardening.md`
- `1_Project/2_Docs/agent-context/project-base-template.md`
- `1_Project/2_Docs/agent-context/active-tasks.md`

---

## Build / Lint / Test Commands

### Backend (Node.js + Express + TypeScript)

```bash
# Navigate to backend directory
cd 1_Project/1_Sources/backend

# Install dependencies
npm install

# Compile TypeScript to JavaScript
tsc

# Compile and watch for changes
tsc -w

# Start development server (uses nodemon on dist/)
npm start

# Run all tests
npm test

# Run all tests with force exit
npm run test:run
```

**Running a single test file:**
```bash
# Run specific test file
npx jest tests/property/userModel.property.test.ts --runInBand

# Run specific test by name
npx jest --runInBand -t "Propiedad 15"
```

### Frontend (Angular 17)

```bash
# Navigate to frontend directory
cd 1_Project/1_Sources/frontend

# Install dependencies
npm install

# Start development server (opens browser)
npm start
# or: npm run ng serve -o

# Build application
npx ng build

# Run tests (Karma)
npm test

# Run linting (TSLint)
npm run lint

# Run e2e tests
npm run e2e
```

**Running a single Angular test:**
```bash
# In karma.conf.js or angular.json, specify single file
# Or use ng test with --include flag
ng test --include="**/user.service.spec.ts"
```

---

## Code Style Guidelines

### Editor Configuration
- **Indentation**: 2 spaces
- **Charset**: UTF-8
- **Line endings**: LF (Unix)
- **Quotes**: Single quotes for TypeScript files
- **Final newline**: Yes
- **Trailing whitespace**: Trimmed

### TypeScript Conventions (Both Frontend & Backend)

**Types:**
- Enable strict mode in tsconfig.json
- Avoid `any` type - use proper types or `unknown`
- Use interfaces for object shapes
- Use type aliases for unions/intersections

**Naming:**
- Classes: PascalCase (e.g., `UsersController`, `AuthService`)
- Functions/Methods: camelCase (e.g., `getUsers()`, `login()`)
- Variables: camelCase (e.g., `userId`, `tokenDecoded`)
- Constants: UPPER_SNAKE_CASE for config values
- Files: kebab-case (e.g., `user.controller.ts`, `auth.bll.ts`)

**Import Style:**
- Use relative paths for local modules
- Group imports: external libs, then internal modules
- Use named exports (preferred) or default exports consistently

**Classes & Methods:**
- Use access modifiers: `private`, `public`, `protected`
- Use dependency injection via constructor
- Keep methods focused and small

### Backend Specific Patterns

**Error Handling:**
- Use `ControlException` class for business logic errors
- Structure: `throw new ControlException(message, code)`
- Controller catches errors and emits to socket:
```typescript
try {
    // business logic
} catch(error) {
    if (error instanceof ControlException) {
        socket.emit("error_message", { message: error.message, code: error.code });
    } else {
        socket.emit("error_message", { message: "Error no controlado" });
    }
}
```

**Database:**
- Use Sequelize ORM with transactions for write operations
- Wrap operations in `sequelize.transaction()`
- Always commit on success, rollback on failure

**Service Layer:**
- DAL (Data Access Layer): database queries
- BLL (Business Logic Layer): business rules, validation

**Testing:**
- Use Jest with ts-jest preset
- Property-based tests with fast-check
- Test files location: `tests/property/*.test.ts`
- Test naming: descriptive, in Spanish if needed

### Frontend Specific Patterns

**Components:**
- Follow Angular 17 patterns
- Use services for business logic (via dependency injection)
- Use guards for route protection
- Use pipes for data transformation

**State Management:**
- Services with BehaviorSubject for local state
- Socket.io for real-time updates

**UI:**
- PrimeNG components
- PrimeFlex for utilities
- W3.CSS for basic styling

**Testing:**
- Karma for unit tests
- TSLint for code quality (configured in tslint.json)

---

## Project Structure

```
1_Project/1_Sources/
├── backend/
│   ├── controllers/ws/     # Socket.io controllers
│   ├── services/           # Business logic (bll) + DAL
│   ├── models/             # Sequelize models
│   ├── routes/ws/          # Route definitions
│   ├── server/             # Server config, middlewares
│   ├── utils/              # Utilities (logger, security)
│   ├── tests/property/     # Jest property-based tests
│   ├── config/             # Environment config
│   └── dist/               # Compiled output
│
├── frontend/
│   ├── src/
│   │   ├── app/
│   │   │   ├── components/ # Angular components
│   │   │   ├── services/   # Business services
│   │   │   ├── models/     # Data models
│   │   │   ├── guards/     # Route guards
│   │   │   └── pipes/      # Angular pipes
│   │   ├── assets/         # Static assets
│   │   └── environments/   # Environment configs
│   └── dist/               # Production build
│
└── nginx/
    └── nginx.conf          # Nginx configuration
```

---

## Environment Setup

1. **MySQL Database**: Run Docker container or local MySQL
   - Default credentials in config files
   - Run database script from `4_Database/` folder

2. **Environment Variables**:
   - Backend: `.env` file (see config/config.ts)
   - Frontend: `src/environments/environment.ts`

3. **Default Admin User**:
   - Username: `admin`
   - Password: `123qwe`

---

## Common Tasks

**Adding a new backend endpoint:**
1. Create/update controller in `controllers/ws/`
2. Define route in `routes/ws/`
3. Add business logic in `services/`
4. Add DAL method if needed in `services/*/`
5. Add tests in `tests/property/`

**Adding a new frontend feature:**
1. Create component in `src/app/components/`
2. Add service if needed in `src/app/services/`
3. Add model if needed in `src/app/models/`
4. Configure routing in `app-routing.module.ts`
5. Add tests if needed

---

## Notes for Agents

- This codebase uses Socket.io for real-time communication (not REST)
- Backend uses JWT tokens for authentication
- Frontend uses Angular 17 with standalone-ish modules
- TSLint is deprecated but still used in this project
- When editing, respect existing code patterns and conventions

---
> Source: [jmddesarrollo/ytoledo](https://github.com/jmddesarrollo/ytoledo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
