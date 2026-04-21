## gas-tracker

> This file provides guidance to Claude Code when working with this React web application.

# CLAUDE.md - gas-tracker

This file provides guidance to Claude Code when working with this React web application.

## LEGACY STATUS

> **This app is being deprecated.** All features will migrate to **termoapp** (React Native).
>
> **Do NOT add new features to this app.** New development should go to termoapp.
>
> **Features to migrate:**
> - Tank Management (Boyas)
> - Gas Discharge Tracking (DescargasDeGas)
> - Tank Transfers (TanquesEnviados)
> - Spare Parts Management (Refacciones)

## Project Overview

**Purpose**: Web application for tracking refrigerant gas tanks (boyas), gas discharges, and spare parts management.

**Stack**: React 18 + TypeScript + Chakra UI + Firebase Auth + REST API

**Deployment**: Netlify

## Architecture

```
gas-tracker/
├── src/
│   ├── components/          # Reusable UI components
│   │   ├── elements/        # Layout, BackButton, Menus
│   │   ├── forms/           # Form components (7 forms)
│   │   ├── inputs/          # Form input fields
│   │   └── libs/            # Table components
│   ├── containers/          # Page components (15 pages)
│   ├── hooks/               # Custom React hooks (7 hooks)
│   ├── helpers/             # Utility functions & interfaces
│   ├── App.tsx              # Router setup
│   └── index.tsx            # Entry point + Firebase config
├── public/                  # Static assets
├── package.json
├── tsconfig.json
└── netlify.toml             # Deployment config
```

## Commands

```bash
npm install          # Install dependencies
npm start            # Start dev server (port 3000)
npm run build        # Production build
npm test             # Run tests
```

## Key Features

### Tank Management (Boyas)
- Register new tanks with ID, refrigerant type, weight
- View tank details and discharge history
- Transfer tanks between users (invite system)
- Track weight changes after gas discharges
- Auto-mark tanks inactive when below threshold

### Gas Discharge Tracking
- Record discharge events with folio, store, equipment
- Capture weight before/after, timestamps, photos
- Validate weight against refrigerant-specific thresholds
- Comments and metadata tracking

### Spare Parts Management (Refacciones)
- Register spare parts with lifecycle tracking
- Upload service sheet images
- Track status: request → authorization → reception → installation
- Color-coded display for delayed/expiring items

### Tank Transfer System
- Request ownership transfer to other users
- Accept/reject invitations with password verification
- Status tracking: pending → accepted/rejected

## Tech Stack

| Category | Technology |
|----------|------------|
| Framework | React 18.1.0 |
| Language | TypeScript 4.6.3 |
| UI | Chakra UI 2.2.1 |
| Forms | Formik 2.2.9 + Yup |
| Auth | Firebase 9.9.0 + ReactFire |
| Routing | React Router DOM 6 |
| Build | Create React App |

## Routes

| Route | Component | Purpose |
|-------|-----------|---------|
| `/` | HomePage | Dashboard - tank list, invites |
| `/login` | LoginPage | Authentication |
| `/create-tank` | CreateTankPage | Register new tank |
| `/tank-info` | TankInfoPage | Tank details + discharge history |
| `/transfer-tank` | TransferTankPage | Request tank transfer |
| `/create-gas-discharge` | CreateGasDischargePage | Record gas discharge |
| `/spares` | SparesInfoPage | Spare parts management |
| `/update-spare` | UpdateSparePage | Edit spare part |

## Custom Hooks

| Hook | Purpose |
|------|---------|
| `useTanks()` | Tank CRUD via REST API |
| `useGasDischarges()` | Discharge records per tank |
| `useSpares()` | Spare parts with file upload |
| `useSendedTanks()` | Tank transfer requests |
| `useUsers()` | User list with name deduplication |
| `useStores()` | Store list with localStorage caching |
| `useFirebaseLogin()` | Firebase auth (login/logout) |

## Data Models

### ITank
```typescript
{
  id: string;                    // Tank serial/ID
  owner_id: number;              // Owner user ID
  owner_name: string;            // Owner display name
  refrigerant: string;           // R22, R134, R404, R410, R449
  tank_weight: number;           // Current weight (kg)
  initial_weight: number;        // Original weight
  status: string;                // InTransit, InDelivery, Inactive
  registered_at: string;
}
```

### IGasDischarge
```typescript
{
  id: string;
  tank_id: string;
  folio: string;
  store: string;
  unit: string;
  owner_name: string;
  actual_tank_weight: number;    // Weight after discharge
  timedate_of_start: string;
  timedate: string;
  register_of_photo: string;
  comments: string;
}
```

### ISpare
```typescript
{
  id: string;
  registrant_name: string;
  name: string;                  // Part name
  folio: string;
  store: string;
  unit: string;
  is_delayed: boolean;
  created_at: string;
  request_date?: string;
  authorization_date?: string;
  reception_date?: string;
  installation_date?: string;
  service_sheet?: string | Blob;
  supplier?: string;
  store_manager?: string;
}
```

## API Integration

**Backend**: Connects to REST API via `API_URL` environment variable

### Required Endpoints (served by termoBackend)
```
GET/POST   /tanks                         # Tank CRUD
GET/POST   /tanks/:id/gas-discharge       # Discharge records
GET/POST   /sended-tanks                  # Transfer requests
PATCH      /sended-tanks/:id              # Accept/reject transfer
GET/POST   /spares                        # Spare parts
PATCH      /spares/:id                    # Update spare
POST       /spares/:id/service-sheet      # Upload service sheet
GET        /spares/:id/service-sheet      # Download service sheet
GET        /users                         # User list
GET        /stores                        # Store list
```

## Firebase Configuration

**Project**: `termoconfort-5d960` (same as other projects)

**Services Used**:
- **Authentication**: Email/password login
- **No direct Firestore**: All data via REST API to backend

## Business Logic

### Refrigerant Weight Thresholds
Tanks are marked "Inactive" when remaining gas drops below threshold:

| Refrigerant | Liquid Weight (kg) |
|-------------|-------------------|
| R22 | 13.6 |
| R134 | 13.62 |
| R404 | 10.89 |
| R410 | 11.35 |
| R449 | 11.35 |

### Tank Registration Permissions
Only specific users can register tanks (UIDs: 1, 9, 13)

### Store Caching
Stores are cached in localStorage with monthly refresh for performance.

## Environment Variables

```bash
API_URL=https://your-backend-url  # REST API endpoint
```

## Relationship to Other Projects

| Project | Relationship |
|---------|--------------|
| **termoBackend** | Provides REST API for tanks/spares (current) |
| **termo-backend-go** | Will provide REST API after migration |
| **termoapp** | Mobile app for thermal measurements (different domain) |
| **Firebase** | Shared auth + database (via backend) |

## Code Style

- **TypeScript**: Strict mode enabled
- **Formatting**: Prettier (80 char, single quotes, 2 spaces)
- **Linting**: ESLint with Airbnb config
- **Forms**: Formik + Yup with Spanish validation messages
- **Components**: Functional components with hooks

## Common Tasks

### Adding a New Form
1. Create folder in `src/components/forms/{FormName}/`
2. Add `{FormName}.tsx` with Formik form
3. Create `helpers/form-helpers.ts` with Yup schema
4. Export via `index.ts`

### Adding a New Hook
1. Create in `src/hooks/use{Resource}.ts`
2. Follow pattern: state + fetch/post/patch methods
3. Export from `src/hooks/index.ts`

### Adding a New Route
1. Add route in `src/App.tsx`
2. Create page component in `src/containers/{PageName}.tsx`
3. Add to Layout navigation if needed

## Notes

- UI language: Spanish ("Trackeador de Boyas")
- Firebase config is embedded in `src/index.tsx`
- Some orphaned JSX files in root (legacy, not imported)
- Password verification required for accepting tank transfers

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EnriqueePerez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
