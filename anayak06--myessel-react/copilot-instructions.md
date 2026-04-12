## myessel-react

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm start        # Start development server (localhost:3000)
npm run build    # Production build
npm test         # Run tests (react-scripts test)
npm test -- --testPathPattern=<filename>  # Run a single test file
```

The backend API runs separately at `http://localhost:57771/api` (configured in `src/config/apiConfig.js`). There is a commented-out production URL pointing to `https://myesselapi.esselprojects.com/api`.

## Architecture Overview

This is a Create React App (React 19) ERP frontend for "RAPP-SLAPP", an enterprise resource planning system for a construction/project company (Essel Projects). It uses Redux Toolkit for state management, Axios for API calls, Tailwind CSS (dark mode via `class` strategy), Formik + Yup for forms, and react-toastify for notifications.

### Routing

There are three top-level routes (`App.js`):
- `/` — Login page (employee credential entry)
- `/login-options` — Secondary login screen after employee validation
- `/employee-dashboard` — Employee self-service dashboard
- `/role-dashboard` — Main ERP application (`RoleBasedApplication`)

All non-public routes are wrapped in `ProtectedRoute`. `SessionProvider` handles session timeout warnings.

### Application Shell: RoleBasedApplication

`src/pages/Dashboard/RoleBasedApplication.jsx` is the main application container. It manages `currentPage` state and renders different page components based on navigation selections from `TopNavbarLayout`. Navigation does **not** use URL routes — all page switching is handled via `onNavigate` prop callbacks and the `currentPage` state string.

`TopNavbarLayout` fetches the user's role-based menu from the backend and renders it. Menu link click frequency is tracked in `sessionStorage` for "recent pages" functionality.

### Inbox / Verification Workflow

`src/components/Inbox/InboxRouter.jsx` is the approval inbox. All verification/approval pages are routed through it based on `inboxType` strings. When a user clicks an item in the inbox, `InboxRouter` renders the appropriate verification component. The inbox sub-components (`LeftPanel`, `RightDetailPanel`, `ActionButtons`, `RemarksHistory`, etc.) are shared across all verification flows.

### State Management Pattern

Every feature follows the same three-layer pattern:
1. **API layer** (`src/api/<Domain>/<featureName>API.js`) — raw Axios calls, no Redux dependency
2. **Slice layer** (`src/slices/<DomainSlice>/<featureName>Slice.js`) — `createAsyncThunk` + `createSlice` with `pending/fulfilled/rejected` states
3. **Page layer** (`src/pages/<Domain>/<FeaturePage>.jsx`) — `useDispatch`/`useSelector` consuming the slice

All slices are registered in `src/store/store.js`.

### Domain Areas

| Domain | Pages | Slices |
|---|---|---|
| HR (Operations) | `src/pages/HR/` | `src/slices/HRSlice/` |
| HR Reports | `src/pages/HRReports/` | `src/slices/HrReportSlice/` |
| Finance Reports | `src/pages/FinancialReports/` | `src/slices/financialReportSlice/` |
| Stock | `src/pages/Stock/` | `src/slices/stockSlice/` |
| Supplier PO | `src/pages/SupplierPO/` | `src/slices/supplierPOSlice/` |
| SPPO | `src/pages/SPPO/` | `src/slices/spPOSlice/` |
| Capital/Shares | `src/pages/shares/` | `src/slices/capitalSlice/` |
| Budget | `src/pages/Budget/` | `src/slices/budgetSlice/` |

### Authentication

`src/slices/auth/authSlice.js` holds `isAuthenticated`, `loginType` (`"employee"` or `"role"`), `roleData`, `employeeId`, `roleId`, and `employeeData`. Auth uses `SessionManager` (`src/utilities/SessionManager.js`) for session persistence. Login flow: employee credentials → `/Security/GetValidEmployee` → optionally select role → `/Security/GetValidUser` → fetch menu via `/Accounts/GetMenu?roleId=`.

### Shared Utilities

- `src/utilities/toastUtilities.js` — `showToast(type, message)` wrapper for react-toastify
- `src/utilities/dailogUtilities.js` — exports `cn()` (clsx + tailwind-merge utility)
- `src/utilities/dateUtils.js` — date formatting helpers
- `src/components/CustomDatePicker.jsx` — custom date picker with portal rendering, used project-wide (accepts `format`, `showTime`, `minDate`, `maxDate` props)

### PDF Generation

Reports use either `@react-pdf/renderer` (for structured PDFs) or `jspdf` + `html2canvas` (for screenshot-to-PDF). PaySlip modal (`src/pages/HRReports/PaySlipModal.jsx`) is a notable example of the html2canvas approach.

### Dark Mode

Tailwind dark mode uses the `class` strategy. `ThemeContext` (`src/contexts/ThemeContext.js`) and `ThemeToggle` component manage the `dark` class on the root element.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ANAYAK06)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ANAYAK06)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
