## kounta

> This document provides instructions for the Gemini AI agent to assist in the

# Gemini Instructions: Kounta

This document provides instructions for the Gemini AI agent to assist in the
development of the Kounta mobile application.

## 1. Project Overview

**Kounta** is a local-first, offline-capable personal financial management
mobile app for Android and iOS. It allows a single user to manage their finances
effectively, with a focus on providing a secure, scalable, and user-friendly
experience that aligns with the established brand and architectural vision.

## 2. Core Architectural Principles

- **Local-First:** All data is stored and managed on the user's device. The app
  must be 100% functional without an internet connection (except for initial
  authentication and backup/restore).
- **Repository Pattern:** All business logic must interact with a "Repository"
  layer, which in turn uses WatermelonDB to execute data operations. This
  isolates the data source and allows for a seamless transition to a backend API
  in the future.
- **Reactive UI:** The UI should be reactive, updating in real-time as the
  underlying data changes.

## 3. Technology Stack

- **Framework:** React Native with Expo
- **UI Components:** React Native Paper (for a complete Material Design
  component library)
- **ORM & Database:** WatermelonDB with a SQLite adapter
- **Navigation:** React Navigation
- **Authentication:** `expo-auth-session` / `expo-google-app-auth` for Google
  Sign-In
- **Cloud Backup:** `expo-google-drive-api-upload-and-download` (or a similar
  library) for Google Drive file operations
- **Icons:** `react-native-vector-icons/MaterialCommunityIcons` (included with
  React Native Paper)

## 4. Brand and Styling

- **Brand Philosophy:** Sophistication, growth, and reassurance.
- **Color Palette:**
  - **Primary:** Green (`#005248`)
  - **Secondary:** Amber (`#B57B00`)
- **Typography:**
  - **Headings:** Poppins
  - **Body:** Inter
- **Theme:** A custom theme file defines the color palettes for both light and
  dark modes. The device's theme should be respected automatically.
- **Icons:** Use Material Community Icons for all icons, ensuring consistency
  with the Material Design guidelines.
- **Brand Assets:** Refer to `docs/brand-guideline.pdf` for detailed brand
  guidelines, including logo usage, color codes, and typography specifications.

## 5. Database Schema & Models

The database schema is defined as WatermelonDB Models. The ORM is responsible
for creating and migrating the underlying SQLite tables.

- **Account Model:** `name` (string), `type` (string), `currency` (string),
  `opening_balance` (number), `created_at` (number, timestamp)
- **Category Model:** `name` (string), `type` (string: "Income" or "Expense"),
  `created_at` (number, timestamp)
- **Asset Model:** `name` (string), `type` (string), `currency` (string),
  `value` (number), `created_at` (number, timestamp)
- **Liability Model:** `name` (string), `type` (string), `currency` (string),
  `total_amount` (number), `current_balance` (number), `created_at` (number,
  timestamp)
- **Transaction Model:** `description` (string), `amount` (number), `type`
  (string: "Income" or "Expense"), `date` (number, timestamp), and relations to
  `accounts`, `categories`, `assets` (optional), and `liabilities` (optional).

## 6. Feature Implementation Guide

- **Onboarding & Setup:** Guide the user through setting up their primary
  currency on first launch. Pre-populate the database with default Category
  models using a WatermelonDB "batch" operation.
- **Authentication & Backup:** Implement Google Sign-In, requesting the
  `https://www.googleapis.com/auth/drive.file` scope. The backup function must
  upload the `kounta.db` file (and related `-wal`, `-shm` files) to Google
  Drive. The restore function must download the database files, replace the
  local ones, and restart the app.
- **CRUD Operations:** All Create, Read, Update, and Delete operations must be
  performed via WatermelonDB model methods. UI components displaying lists of
  data should use the `@watermelondb/withObservables` HOC or related hooks to be
  reactive.
- **Transaction Management:** The "Add Transaction" form will create a new
  Transaction model instance. Dropdowns for relations will be populated by
  querying the respective model collections. All database changes for a single
  transaction must be wrapped in a single `database.write()` block to ensure
  atomicity.
- **Dashboard & Reporting:** Dashboard widgets will be powered by WatermelonDB
  queries. Due to WatermelonDB's reactive nature, the dashboard figures should
  update in real-time as new transactions are added.

## 7. Folder Structure

### Current Workspace Structure

```
/ (root)
├── apps/
│   └── mobile/   # React Native + Expo mobile app
├── docs/
│   ├── brand-guideline.pdf
│   ├── project-description.pdf
│   └── technical-guidelines.pdf
├── scripts/
├── README.md
├── GEMINI.md
```

### Recommended Mobile App Structure

```
apps/mobile/
├── assets/           # Images, fonts, icons
├── components/       # Stateless UI components (React Native Paper)
├── hooks/            # Business logic hooks (e.g., useAccounts.hook.js)
├── models/           # WatermelonDB models (e.g., account.model.js)
├── repositories/     # Data access layer (e.g., account.repository.js)
├── navigation/       # React Navigation setup
├── screens/          # Screen components (stateless, UI only)
├── theme/            # Custom theme, color palettes, typography
├── utils/            # Utility functions
├── App.js            # Entry point
├── database.js       # WatermelonDB setup
├── .env              # Environment variables
└── ...
```

> Refer to docs/brand-guideline.pdf, docs/project-description.pdf, and
> docs/technical-guidelines.pdf for further details and standards.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Francois-MUGOROZI)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Francois-MUGOROZI)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
