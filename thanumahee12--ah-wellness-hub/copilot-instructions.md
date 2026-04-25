## ah-wellness-hub

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AH Wellness Hub Blood Lab Manager is a complete Point of Sale (POS) system for blood testing laboratories. It provides comprehensive management of patients, blood tests, checkups/billing, medicines, user management with activity tracking, and generates professional PDF invoices and prescriptions. The application features comprehensive role-based access control (RBAC) with superadmin, admin, maintainer, editor, and user roles.

## Development Commands

- **Start dev server**: `npm run dev` - Starts Vite dev server with HMR at http://localhost:5173
- **Build**: `npm run build` - Production build to `dist/` directory
- **Lint**: `npm run lint` - Run ESLint on all files
- **Preview build**: `npm run preview` - Preview production build locally

## Tech Stack

- **Framework**: React 19.1.1
- **Build Tool**: Vite 7.1.7
- **State Management**: Redux Toolkit (@reduxjs/toolkit 2.9.0)
- **Routing**: React Router DOM 7.9.4
- **UI Framework**: React Bootstrap 2.10.10 + Bootstrap 5.3.8
- **Icons**: React Icons 5.5.0
- **PDF Generation**: jsPDF 3.0.3 + html2canvas 1.4.1
- **Charts**: Recharts 2.15.0
- **Backend**: Firebase (Authentication, Firestore, Hosting)
- **Language**: JavaScript (JSX)

## Project Structure

### Core Files
- `src/main.jsx` - Entry point with Redux Provider, Router, and Bootstrap CSS
- `src/App.jsx` - Main app with routing and protected routes
- `index.html` - HTML shell

### State Management (Redux)
- `src/store/store.js` - Redux store configuration
- `src/store/authSlice.js` - Authentication state with Firebase integration
- `src/store/testsSlice.js` - Blood tests management with Firestore sync
- `src/store/patientsSlice.js` - Patients management with Firestore sync
- `src/store/checkupsSlice.js` - Checkups/billing management with Firestore sync
- `src/store/usersSlice.js` - Users management with Firestore sync
- `src/store/medicinesSlice.js` - Medicines management with Firestore sync
- `src/store/settingsSlice.js` - Dynamic settings (forms, tables, pages, permissions) with Firestore sync

### Components
- `src/components/Navbar.jsx` - Top navigation bar with user info and logout
- `src/components/Sidebar.jsx` - Side navigation with role-based menu items
- `src/components/ProtectedRoute.jsx` - Route protection with role-based access control
- `src/components/crud/` - Reusable CRUD components (CRUDTable, CRUDModal, EnhancedCRUDTable)
- `src/components/ui/` - UI components (PageHeader, FormField, RichTextEditor)

### Pages
- `src/pages/Home.jsx` - Landing page with features showcase (content driven by settings)
- `src/pages/Login.jsx` - Firebase authentication page
- `src/pages/Dashboard.jsx` - Analytics dashboard with charts, statistics, and time-range filters
- `src/pages/Patients.jsx` - Patient CRUD with enhanced table
- `src/pages/PatientDetail.jsx` - Individual patient details and history
- `src/pages/Checkups.jsx` - Checkup/billing with multi-step form
- `src/pages/CheckupDetail.jsx` - Detailed checkup view with invoice and prescription PDF generation
- `src/pages/Tests.jsx` - Blood tests CRUD (role-restricted)
- `src/pages/Medicines.jsx` - Medicine inventory management (role-restricted)
- `src/pages/UserManagement.jsx` - Consolidated user management with tabs:
  - `src/pages/tabs/UsersTab.jsx` - User CRUD operations
  - `src/pages/tabs/UserActivityTab.jsx` - Activity tracking and analytics (superadmin only)
  - `src/pages/tabs/UserRequestsTab.jsx` - User role change requests (superadmin only)
- `src/pages/Settings.jsx` - Settings page with dynamic tabs
  - `src/pages/tabs/TablesSettingsTab.jsx` - Table column config (visibility, roles, searchable)
  - `src/pages/tabs/PagesSettingsTab.jsx` - Page access and sidebar config
  - `src/pages/tabs/PublicPageTab.jsx` - Home page content editor (hero, blogs, CTA)
- `src/pages/ColumnSettingDetail.jsx` - Column detail editor (label, visible, roles, searchable, field type, etc.)
- `src/pages/AdminSetup.jsx` - Initial admin setup page

### Services
- `src/services/authService.js` - Firebase Authentication service
- `src/services/firestoreService.js` - Firestore database operations
- `src/services/activityService.js` - User activity tracking
- `src/services/roleRequestService.js` - Role change request management

### Hooks
- `src/hooks/useCRUD.js` - Reusable CRUD operations hook
- `src/hooks/usePermission.js` - Permission checking hook
- `src/hooks/useSettings.js` - Settings access hook (filter fields/columns by visibility, role, searchable; page access checks; permission checks)

### Constants
- `src/constants/defaultSettings.js` - Default settings (forms, tables, pages, permissions); single source of truth merged with Firestore
- `src/constants/roles.js` - Role hierarchy and permission helpers

### Context
- `src/context/NotificationContext.jsx` - Toast notifications provider

## Application Features

### Authentication & Authorization
- Firebase Authentication integration
- Comprehensive role-based access control (RBAC)
- Fine-grained permissions system (read, create, edit, delete per resource)
- User activity tracking for audit trails
- Role change request workflow
- Protected routes with role verification

### Data Models

**User**: `{ id, username, email, mobile, role, permissions, createdAt, lastLogin }`
**Patient**: `{ id, name, age, gender, mobile, address, email, bloodGroup }`
**Test**: `{ id, code, name, price, percentage, details, rules }`
**Medicine**: `{ id, code, name, brand, dosage, unit, description }`
**Checkup**: `{ id, billNo, patientId, tests[], medicines[], total, commission, discount, notes, timestamp }`
**UserActivity**: `{ id, userId, username, action, resource, metadata, timestamp }`
**RoleRequest**: `{ id, userId, username, currentRole, requestedRole, status, requestedAt, processedAt, processedBy }`

### Dynamic Settings System
Settings are stored in Firestore (`settings` collection) and deep-merged on top of `DEFAULT_SETTINGS` in `src/constants/defaultSettings.js`.

**Forms settings** (`settings.forms.[entity].fields`):
- Per-field: `visible`, `required`, `label`, `type`, `colSize`, `placeholder`, `rows`

**Tables settings** (`settings.tables.[entity]`):
- `itemsPerPage`: Rows per page for the table
- Per-column: `visible`, `label`, `roles` (array of roles that can see the column), `searchable` (whether included in search)

**Pages settings** (`settings.pages.[pageKey]`):
- `label`, `icon`, `path`, `order`, `roles`, `sidebar`
- `tabs`: per-tab `label` and `roles`
- `content`: home page hero/blog content

**Permissions settings** (`settings.permissions.[resource].[action]`):
- Array of roles allowed for each action (view, create, edit, delete)

### User Roles & Permissions

**Superadmin** - Full system access:
- Manage all resources (CRUD)
- View activity logs and analytics
- Approve/reject role change requests
- Access all dashboard features

**Admin** - Administrative access:
- Manage users (CRUD)
- Manage tests and medicines
- All maintainer permissions

**Maintainer** - Content management:
- Manage tests and medicines
- View user requests
- All editor permissions

**Editor** - Data entry:
- Manage patients (CRUD)
- Create/edit checkups
- Manage medicines
- All user permissions

**User** - Basic access:
- View patients
- View checkups
- Generate PDF bills and prescriptions

### Firebase Setup
First-time setup requires creating a superadmin account via AdminSetup page (`/admin-setup`).

## Design System

### Theme
- Color scheme: Cyan/Teal professional theme
  - Primary: #0891B2 (Cyan 600) to #06B6D4 (Cyan 500)
  - Secondary: #14B8A6 (Teal 500)
  - Accents: #F59E0B (Amber 500), #22D3EE (Cyan 400)
  - Backgrounds: White (#fff) with gradient headers
  - Text: Dark (#333) on light backgrounds
- Modern card-based layouts with shadows
- Gradient backgrounds for headers and buttons

### Mobile-First Responsive Design
The application is **highly optimized for mobile devices** with responsive breakpoints and touch-friendly interactions:

- **Breakpoints**: xs (<576px), sm (≥576px), md (≥768px), lg (≥992px), xl (≥1200px)
- **Sidebar**:
  - Desktop: Fixed left sidebar (250px width)
  - Mobile: Hamburger toggle button, slides in from right
  - Touch-friendly 48px minimum button size
- **Tables**: Fully mobile-responsive with horizontal scrolling
  - Adaptive column widths: Narrower on mobile (120-150px) vs desktop (200-300px)
  - Data-label attributes for mobile card view fallback
  - Action buttons with appropriate spacing
- **Forms**:
  - Modals use `fullscreen="md-down"` for mobile full-screen experience
  - Form fields stack vertically on mobile (col-12)
  - Buttons full-width on mobile, auto-width on desktop
- **Charts**:
  - Responsive containers with adjusted margins for mobile
  - Smaller fonts and reduced label distances on mobile
  - Button groups stack and expand to full width on mobile
- **Typography**:
  - Extensive use of `clamp()` for responsive font sizing
  - Example: `clamp(0.75rem, 2vw, 1rem)` scales from 12px to 16px
- **Touch Targets**:
  - Minimum 44px×44px for all interactive elements
  - Increased padding on mobile for easier tapping
- **Navbar**:
  - Responsive user info with abbreviated text on mobile
  - Logout button maintains 44px minimum touch target

### Mobile-Specific Optimizations
- Dashboard chart button labels shortened ("7 Days" → "7d") on mobile
- Table column widths dynamically adjust based on `window.innerWidth < 768`
- PieChart margins and label positioning responsive to screen size
- All modals adapt to mobile with proper scrolling and padding

## Code Conventions

- **State Management**: Redux Toolkit with Firebase Firestore sync
  - Use async thunks for all CRUD operations
  - Handle loading, success, and error states
- **Routing**: ProtectedRoute component with role-based access
- **Forms**: Controlled components using useCRUD hook
- **Modals**: Bootstrap modals with `fullscreen="md-down"` for mobile
- **PDF Generation**: html2canvas + jsPDF for invoice and prescription generation
- **Icons**: React Icons (Font Awesome)
- **Styling**:
  - Bootstrap 5 utility classes preferred
  - Inline styles for dynamic/responsive values
  - Use `clamp()` for responsive typography
  - Mobile-first approach with responsive breakpoints
- **Responsive Design**:
  - Tables: `window.innerWidth < 768` for mobile detection
  - Modals: `fullscreen="md-down"` attribute
  - Buttons: `w-100 w-sm-auto` for mobile-first full width
  - Touch targets: Minimum 44px×44px for interactive elements
- **Color Scheme**: Cyan/Teal gradient theme (#0891B2, #06B6D4, #14B8A6)

## Git Commit Guidelines

- **DO NOT** include Claude Code signature in commit messages
- Keep commit messages concise and descriptive
- Format: `type: description` (e.g., `feat: add responsive tables`, `fix: resolve mobile layout issues`)

## Key Workflows

### Creating a Checkup (Multi-Step Process)
1. **Step 1**: Select or create new patient
   - Choose existing patient from dropdown
   - Or create new patient inline
2. **Step 2**: Configure checkup details
   - Select multiple tests (with prices)
   - Add medicines with dosages
   - Apply discount percentage
   - Auto-calculate commission (test percentage)
   - Add optional notes
3. Save creates checkup with auto-generated bill number
4. Navigate to checkup detail page
5. Generate PDF invoice or prescription

### PDF Generation
Both invoice and prescription share the same branded template (header with dual logos, company name, bill #, date/time).

**Invoice Format** (Bill Tab):
- Branded header with AWH + ASIRI logos
- Compact patient info row
- Tests table with prices and total
- Green PAID stamp in footer
- Contact info footer (Mobile, Email, IG, FB)
- "Thank you" banner with ASIRI logo

**Prescription Format** (Prescription Tab):
- Same branded header as invoice
- Compact patient info row
- 70/30 body split: medicines table + instructions (left 70%), patient vitals H/W (right 30%)
- Date line (left) and Signature line (right) in footer
- Same contact info footer and "Thank you" banner

**Page Size Configuration**:
- Default: A5 (148 x 210 mm)
- Supported: A4, A5, Letter, Thermal 80mm, Thermal 58mm, Custom
- Clone width and html2canvas windowWidth dynamically match selected page size
- Smaller formats (<100mm) auto-adjust padding and font size

### User Activity Tracking
- Automatic logging of all CRUD operations
- Tracks: user, action, resource, timestamp, metadata
- Viewable by superadmin in Activity tab
- Provides audit trail for compliance

### Role Change Request Workflow
1. User requests role change via profile
2. Request stored in Firestore with "pending" status
3. Superadmin views requests in Requests tab
4. Superadmin approves or rejects
5. User role updated, activity logged
6. Notification sent to user

## Development Notes

- **Backend**: Firebase Firestore for data persistence
- **Auth**: Firebase Authentication with role-based claims
- **State**: Redux syncs with Firestore in real-time
- **IDs**: Firestore auto-generated document IDs
- **Timestamps**: Server timestamps for consistency
- **PDF**: Client-side generation, browser downloads
- **Activity Logging**: Firestore collection with TTL (optional)
- **Print Styles**: Custom CSS (`src/styles/print.css`) for PDF printing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThanuMahee12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
