## home-assignment-source

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Medication Validation Application** — a React-based QA home assignment platform that validates medication-related data (medication names, dosages, frequencies, dates of birth, and patient weight). The app includes form field detection, comprehensive validators, and a sophisticated system for detecting user testing patterns (detector/accomplishment system).

Key context: The application is designed for QA candidates to complete timed assignments with limited access windows. It monitors user interactions to detect different testing approaches and patterns.

## Build, Lint, and Test Commands

```bash
# Development server (runs on http://localhost:5173)
npm run dev

# Build for production (outputs to dist/)
npm run build

# Run linter to check code quality
npm run lint

# Run all tests
npm test

# Run a specific test file
npm test -- src/__tests__/integratedDetectorValidator.test.js

# Run tests in watch mode
npm test -- --watch
```

## Code Architecture

### Core Data Flow

1. **Access Control** (App.jsx):
   - Validates time-limited access tokens from URL parameters
   - Uses CET-based timezone handling for consistent scheduling across regions
   - Tokens encode email, start datetime, and duration (e.g., "3h")
   - Automatically redirects unencoded parameters (`?email=...&date=...&time=...&duration=...`) to encoded token format
   - Checks access validity every 5 seconds; shows "expires in X minutes" countdown timer

2. **Form Management** (ValidationForm.jsx):
   - Maintains form state for: medication, dateOfBirth, weight, dosage, frequency
   - Real-time field validation using dedicated validators
   - **Detector/Accomplishment System**: On form submission, calls detector functions for each field to identify testing patterns
   - Synced session storage with checksums to detect tampering and concurrent sessions
   - Autosaves to Google Forms every 1 minute (if changes detected)

3. **Validation & Detection Pattern**:
   - Each field has paired **detector** and **validator** modules:
     - `*Detector`: Identifies user testing patterns (e.g., boundary testing, edge cases)
     - `*Validator`: Checks if field value is valid according to rules
   - Detectors run **only on form submission**, returning an array of detection tags
   - Field-level detections feed into **Form-level detector** which detects cross-field patterns (e.g., "nominal_form" if all fields have nominal values)
   - All detections are stored in session storage as "accomplishments"

### Directory Structure

```
src/
├── App.jsx                          # Main app: access token validation, timers, header rendering
├── components/
│   ├── ValidationForm.jsx           # Form submission, field management, detector/accomplishment tracking
│   ├── Timer.jsx                    # Countdown timer showing time remaining for form access
│   └── Badges.jsx                   # UI components for displaying detected patterns
├── detectors/                       # Pattern detection functions (called on form submission)
│   ├── fieldDetector.js             # Generic field-level pattern detection
│   ├── formDetector.js              # Form-level patterns (e.g., "nominal_form", "enter_submit")
│   ├── medicationDetector.js
│   ├── dosageDetector.js
│   ├── frequencyDetector.js
│   ├── dateOfBirthDetector.js
│   └── weightDetector.js
├── validators/                      # Validation rules (return error messages or null)
│   ├── medicationValidator.js       # Validates against: placebo, aspirin, ibuprofen, paracetamol, naproxen
│   ├── dosageValidator.js
│   ├── frequencyValidator.js
│   ├── dateOfBirthValidator.js
│   └── weightValidator.js
├── hooks/                           # Custom React hooks
│   ├── useSessionStorage.js         # Basic session storage hook
│   ├── useProtectedSessionStorage.js # Session storage with checksum validation (detects tampering)
│   └── useSyncedSessionStorage.js   # Multi-window session storage sync with tampering detection
├── utils/
│   ├── accessTokenManager.js        # Token encoding/decoding, access window validation (CET timezone)
│   ├── googleFormsSubmission.js     # Submits accomplishments and form data to Google Forms
│   ├── storageChangeDetector.js     # Monitors sessionStorage for tampering and cross-window changes
│   ├── checksumUtils.js             # Checksum package creation/verification for storage integrity
│   ├── ageCalculator.js             # Age calculation from date of birth
│   └── readOnlyState.js             # deepFreeze() for immutable state
├── config/
│   ├── validationRules.js           # Centralized error messages for all validators
│   ├── badgeIcons.js                # Maps detection tags to UI icons/emojis
│   ├── badgeSortOrder.js            # Defines display order for badges
│   └── detectorCoverageData.js      # Detector coverage metadata
├── __tests__/
│   ├── integratedDetectorValidator.test.js  # Integration tests using fixtures
│   └── fixtures/                    # JSON test case files (dynamically loaded with Vite's import.meta.glob())
├── index.css                        # Global styles
├── main.jsx                         # React entry point
└── testConsole.js                   # Test console utility
```

### Key Components & Modules

**ValidationForm.jsx** — The heart of the app:
- Manages form state and field errors
- Calls validators on change, detectors on submit
- Tracks "accomplishments" (detected testing patterns) in session storage
- Monitors storage for tampering (attempted manual edits) or concurrent session access
- Autosaves every minute via Google Forms API
- Clears form after successful submission

**Timer.jsx** — Countdown timer display:
- Shows remaining time until access window expires
- Integrated with access token validation from App.jsx
- Updates every second to reflect time remaining

**Detector Functions** — Pattern detection (run on submit):
- Return array of detection tags (e.g., `['boundary_testing', 'nominal_value']`)
- Examples: `boundary_testing`, `nominal_value`, `edge_case_testing`, `enter_submit`, `nominal_form`
- Stored in accomplishments object keyed by field name

**Validators** — Form validation (run on change):
- Return array of error message strings: `Array<string>` (empty array means valid, non-empty means invalid)
- Checked against `validationRules.js` for user-facing messages

**Storage System** (`useProtectedSessionStorage` + `storageChangeDetector`):
- All state persisted to sessionStorage with checksums to prevent tampering
- Detects two types of violations:
  1. **Tampering**: Local script modifications to stored data (checksum mismatch)
  2. **Concurrent Session**: Another window/tab modifying the same storage key
- Flags set when violations detected; submitted with accomplishments
- Automatically restored from in-memory backup if tampering detected

**Access Token Manager** (`accessTokenManager.js`):
- Encodes: `email|iso8601DateTime|duration` + checksum → Base64
- Decodes and validates token format and checksum
- Interprets dates/times as **CET** (Central European Time, UTC+1)
- Returns validation object with `isValid`, `message`, `timeRemaining`, and `timeRemainingSeconds`

### Testing Strategy

Tests use **fixtures** (JSON files in `__tests__/fixtures/`) containing test cases:
- Fixture files: `aspirin.json`, `ibuprofen.json`, `naproxen.json`, `paracetamol.json`, `default.json`
- Each fixture defines `values` (form inputs) and `expectations` (expected detector outputs and validation errors)
- Date offsets `[-X, 0, 0]` in `dateOfBirth` are converted to relative dates (e.g., age calculation tests)
- Tests dynamically load all fixture files using Vite's `import.meta.glob()` and run integrated detector + validator checks

Run single test: `npm test -- integratedDetectorValidator.test.js`

## Important Patterns & Constraints

### Detector Detection Tags

Common tags returned by detectors:
- `nominal_value` — Standard/typical input
- `boundary_testing` — Edge values (min/max)
- `edge_case_testing` — Invalid/unusual values
- `enter_submit` — Form submitted via Enter key (not button click)
- `nominal_form` — Form-level: all fields contain nominal values
- `storage_tampering` — Session storage was modified manually
- `concurrent_session` — Another tab/window accessed storage

### Storage & Tampering Detection

- Use `useProtectedSessionStorage` for any data you want protected from tampering
- The hook automatically validates checksums on read and flags tampering
- Never directly call `window.sessionStorage.setItem()` — always use the hooks
- In-memory backups in `storageChangeDetector.js` persist across tampering attempts

### Access Token Format

When generating test links, use one of two formats:
1. **Encoded (preferred)**: `?<token>` — single Base64 string
2. **Unencoded (for testing)**: `?email=user@example.com&date=2026-02-06&time=14:30&duration=3h` — automatically redirects to encoded

Tokens must pass checksum validation; tampering the URL parameters will break the checksum.

### React Hooks & State Management

- Use `useSyncedSessionStorage` for form data that needs cross-tab sync and tampering detection
- Use `useProtectedSessionStorage` for sensitive data (accomplishments, email)
- Use standard `useState` for transient UI state (errors, loading)
- State values returned by protected storage hooks are **frozen (immutable)**; don't mutate them

### Validation Rules

All validation error messages are defined in `config/validationRules.js`:
- Medication: required, maxLength (20), must be one of the known medications
- DateOfBirth: required (also validates age ranges if needed)
- Weight: required (validates unit support)
- Dosage: required (validates numeric format and units)
- Frequency: required (validates frequency patterns)

When adding validators, always add corresponding rules to `validationRules.js`.

## Development Notes

### Vite Configuration
- Base is set to `./` for relative paths
- Uses Terser minification with aggressive settings (drop_console, mangle enabled)
- Comment stripping and CSS minification enabled
- No manual chunk splitting (single bundle)

### ESLint Rules
- Extends: `@eslint/js`, `react-hooks`, `react-refresh/vite`
- Custom rule: Variables starting with uppercase or underscore are allowed as unused; function arguments starting with underscore are allowed as unused
- Run `npm run lint` before committing

### Google Forms Integration
- `googleFormsSubmission.js` handles form submission
- Submits: form values, detected patterns (accomplishments), tampering flags, timestamp, candidate email
- Ensure environment/config has correct Google Forms URL

### Timezone Handling (Important!)
- All date/time handling uses **CET (UTC+1)** as the canonical timezone
- Access tokens store times in ISO 8601 format with `+01:00` offset
- When parsing, JavaScript's `Date` constructor correctly handles timezone offsets
- Never assume UTC; always use CET for schedule-related logic

### Performance Considerations
- Storage change detection polls every 500ms (not instant)
- Access validation checks expire every 5 seconds (not 1 second) to reduce overhead
- Autosave interval is 1 minute for testing (change to 10 minutes for production)
- Form is cleared after successful submission to free memory

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zs-varga) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
