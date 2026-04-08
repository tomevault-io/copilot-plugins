## adi-modoc-amplify-vite-react

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Modoc Clinician Dashboard - A React + Vite + TypeScript web application for monitoring patient heart rate data from connected medical devices. Integrated with AWS Amplify for authentication and API Gateway for device data.

## Common Commands

```bash
# Start development server
npm run dev

# Build for production
npm run build

# Run ESLint
npm run lint

# Preview production build locally
npm run preview
```

## Architecture

### Tech Stack
- **React 18** with TypeScript
- **Vite** for build tooling
- **AWS Amplify v6** for authentication (Cognito) and backend
- **@nivo/line** for heart rate chart visualization
- **AWS API Gateway** for device data fetching

### Data Flow

1. **Authentication**: Users authenticate via AWS Cognito through the `<Authenticator>` component from `@aws-amplify/ui-react`. Sign-up is disabled (`hideSignUp` prop).

2. **Device Data Fetching** (`App.tsx:92-161`):
   - Primary data source: AWS API Gateway endpoint (`hwm6t7hyy1.execute-api.us-east-1.amazonaws.com/dev/dashboard-devices`)
   - Authenticated requests include `Authorization: Bearer ${idToken}` header
   - On CORS/auth failure, falls back to hardcoded `DeviceDefaults` from `DeviceDefaults.tsx`
   - Fake device data (`FakeDeviceData`) is concatenated to the real response for testing

3. **Heart Rate Data Structure**:
   - `Device.hrValues`: Array of `{x: number, y: number}` points (time in seconds vs BPM)
   - `Device.zones`: Heart rate zone thresholds (min/max HR with colors)
   - `Device.targets`: Prescription targets (zone name + duration)
   - `Device.pastMeasurements`: Historical S3 keys for retrieving previous sessions

4. **Admin Privileges** (`App.tsx:81-90`):
   - Admin status determined by hardcoded Cognito user IDs
   - Two user pools are supported with specific user IDs checked
   - Admin users see "Devices" and "Administration" navigation items

### Component Patterns

- **State Management**: React useState/useEffect at App level, props drilled to children
- **Modal Pattern**: Modals rendered via conditional overlay in `App.tsx` (not a portal)
- **Data Passing**: Child components receive setter functions to trigger modal state changes

### API Endpoints

```typescript
// List all devices
GET https://hwm6t7hyy1.execute-api.us-east-1.amazonaws.com/dev/dashboard-devices
Authorization: Bearer ${idToken}

// Get specific measurement from S3
GET ${AWS_DEVICES_ENDPOINT}/${deviceId}?s3key=${encodeURIComponent(s3Path)}

// Update device prescription (zones)
PATCH ${AWS_DEVICES_ENDPOINT}/${thingName}
Body: { zones: prescription }

// Update device profile
PATCH ${AWS_DEVICES_ENDPOINT}/${deviceId}
Body: zones array
```

### Key Files

- `Modoc.types.ts`: Core TypeScript interfaces (`Device`, `HrZone`, `Kit`)
- `DeviceDefaults.tsx`: Hardcoded fallback device data and default zone configurations
- `MeasurementScreen.tsx`: Heart rate chart using Nivo with zone duration calculations
- `PrescriptionModal.tsx`: Configure zone targets per device
- `amplify/auth/resource.ts`: Cognito configuration with email login
- `amplify.yml`: Amplify CI/CD build specification

### Device Thing Name Mapping

Device IDs are mapped to AWS IoT thing names in `App.tsx:49-55` via `deviceThingNames` object. When updating prescriptions, the thing name is resolved from this mapping before API calls.

### Zone Calculation Logic

Zone durations are calculated client-side in `MeasurementScreen.tsx:30-53` by iterating through `hrValues` and counting data points that fall within each zone's min/max HR range. Each data point represents 2 seconds of activity.

### Environment Notes

- No test runner configured (no Jest/Vitest/Playwright)
- ESLint configured with React Hooks and TypeScript rules
- AWS Amplify backend only includes auth (no API/database configured in `amplify/`)
- Production build outputs to `dist/` directory

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wired247)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wired247)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
