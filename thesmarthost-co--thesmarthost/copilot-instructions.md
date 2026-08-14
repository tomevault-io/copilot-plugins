## thesmarthost

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

> **HostMetrics Frontend** - Property management reporting platform built with Next.js, TypeScript, and Tailwind CSS

**Last Updated:** January 22, 2026

---

## Quick Reference

### Development Commands

```bash
# Start development server (with Turbopack)
npm run dev
# → Runs on http://localhost:3000

# Build for production (ALWAYS run before pushing)
npm run build

# Start production server
npm start
```

**Important:** Always run `npm run build` after changes before pushing to verify no TypeScript or build errors.

---

## Project Context

### What is HostMetrics?

Property management reporting platform for short-term rental managers.

- **Client:** TheSmartHost Co. Inc (Luis Torres)
- **Team:** Mark Cena (Calgary) + Hussein Saab (Toronto)
- **Timeline:** Oct 7 - Dec 20, 2025 (10 weeks)
- **Status:** Feature Complete - All core features implemented

**Goal:** Automate monthly financial reports for property owners (reduce from 2-4 hours → 10 minutes per client)

---

## Tech Stack

| Technology | Version | Purpose |
|------------|---------|---------|
| Next.js | 16.1.0 | App Router with Turbopack |
| React | 19.2.3 | UI Framework |
| TypeScript | 5.x | Type Safety (strict mode) |
| Tailwind CSS | 4.x | Styling |
| Zustand | 5.0.8 | State Management |
| Recharts | 3.6.0 | Analytics Charts |
| Framer Motion | 12.x | Animations |
| Supabase SSR | 0.7.0 | Authentication |
| Heroicons | 2.2.0 | Icons |
| React Icons | 5.5.0 | Additional Icons |
| React Markdown | 10.1.0 | Markdown Rendering (AI Insights) |

**Backend:** Express.js + PostgreSQL (separate repository: `thesmarthost-backend`)

---

## Architecture Patterns

### 1. Route Groups for Access Control

```
app/
├── (prelogin)/     # Public pages (login, signup, about, contact)
├── (user)/         # Protected pages
│   └── property-manager/
│       ├── analytics/
│       ├── bookings/
│       ├── clients/
│       ├── dashboard/
│       ├── expenses/
│       ├── incoming-bookings/
│       ├── properties/
│       ├── reports/
│       ├── settings/
│       └── upload-bookings/
├── api/auth/       # Auth callback routes
├── forbidden/      # Access denied page
├── error.tsx       # Error boundary
└── not-found.tsx   # 404 page
```

**Layout inheritance:**
- `(prelogin)` → uses PreNavbar
- `(user)` → uses UserNavbar
- `property-manager` → adds ManagerSidebar

### 2. Component Organization by Resource + Action

```
components/
├── [resource]/
│   ├── create/
│   │   └── create[Resource]Modal.tsx
│   ├── update/
│   │   └── update[Resource]Modal.tsx
│   ├── delete/
│   │   └── delete[Resource]Modal.tsx
│   ├── preview/
│   │   └── preview[Resource]Modal.tsx
│   └── import/
│       ├── bulk[Resource]Modal.tsx
│       └── steps/
└── shared/
    ├── modal.tsx
    ├── notification.tsx
    ├── TableActionsDropdown.tsx
    ├── FloatingActionButton.tsx
    └── LogoutModal.tsx
```

**Rules:**
- Folders use lowercase-hyphenated names (`client-agreement/`)
- Files use PascalCase (`createPropertyModal.tsx`)
- All interactive components must have `'use client'` directive

### 3. Service Layer Architecture

```
services/
├── apiClient.ts              # Fetch wrapper with credentials
├── [resource]Service.ts      # API functions (GET, POST, PUT, DELETE)
└── types/
    └── [resource].ts         # TypeScript interfaces
```

### 4. API Response Contract

All backend endpoints follow this format:

```typescript
// Success
{ status: 'success', data: { ... } }

// Error
{ status: 'failed', message: 'Human-readable error' }

// Frontend handling
const res = await createResource(payload)
if (res.status === 'success') {
  showNotification('Success', 'success')
} else {
  showNotification(res.message || 'Failed', 'error')
}
```

### 5. State Management Strategy

**Zustand Stores (3 total):**
- `useUserStore` - User profile and auth state (persisted to localStorage)
- `useNotificationStore` - Toast notifications (not persisted)
- `useAnalyticsStore` - Analytics filters and data (not persisted)

**Local State (`useState`):** Form inputs, modals, fetched data

---

## Complete Component Inventory (98 components)

### Analytics Components (8 files)
```
components/analytics/
├── AnalyticsWidget.tsx           # Main composite component (full & compact modes)
├── DrillDownModal.tsx            # Modal for viewing filtered bookings
└── shared/
    ├── AIInsightsCard.tsx        # AI-generated weekly insights (markdown)
    ├── AnalyticsFilters.tsx      # Date range presets + property/channel filters
    ├── BreakdownTabs.tsx         # Property/Channel tabs with table/chart views
    ├── KPICard.tsx               # Individual metric card with delta
    ├── KPIGrid.tsx               # Grid of 7 KPI cards
    ├── TimelineChart.tsx         # Recharts line/area/bar chart
    └── index.ts                  # Barrel exports
```

### Booking Components (5 files)
```
components/booking/
├── create/createBookingModal.tsx
├── delete/deleteBookingModal.tsx
├── import/ImportHostawayBookingsModal.tsx
├── preview/previewBookingModal.tsx
└── update/updateBookingModal.tsx
```

### Client Components (10 files)
```
components/client/
├── create/createClientModal.tsx
├── delete/deleteClientModal.tsx
├── preview/previewClientModal.tsx
├── quick-create/QuickCreateClientModal.tsx
├── update/updateClientModal.tsx
└── import/
    ├── bulkImportClientModal.tsx
    └── steps/
        ├── MapFieldsStep.tsx
        ├── PreviewStep.tsx
        └── UploadStep.tsx
```

### Client-Related Components (3 files)
```
components/client-agreement/clientAgreementModal.tsx
components/client-note/clientNoteModal.tsx
components/status/statusCodeManagementModal.tsx
```

### Connection Components (2 files)
```
components/connection/
├── guesty/GuestyConnectionModal.tsx
└── hostaway/HostawayConnectionModal.tsx
```

### Dashboard Components (13 files)
```
components/dashboard/
├── ActionBar/
│   └── ActionBar.tsx             # Sticky quick-action buttons
├── AlertsZone/
│   ├── AlertCard.tsx             # Individual alert card
│   ├── AlertsZone.tsx            # Alert container
│   └── AllClearState.tsx         # Green checkmark when no alerts
├── MetricsZone/
│   ├── ActivityFeed.tsx          # Recent activity timeline
│   ├── ActivityItem.tsx          # Individual activity item
│   ├── InsightCard.tsx           # Performance insight card
│   ├── InsightsSection.tsx       # Insights container
│   ├── MetricCard.tsx            # Health metric card
│   └── MetricsGrid.tsx           # 6-card metrics grid
└── shared/
    ├── Sparkline.tsx             # Mini trend charts
    ├── TimeAgo.tsx               # Relative time display
    └── TrendIndicator.tsx        # Up/down trend arrows
```

### Expense Components (4 files)
```
components/expenses/
├── ExpenseViewerModal.tsx
├── categories/ExpenseCategoriesModal.tsx
├── create/CreateExpenseModal.tsx
└── scan/ScanReceiptModal.tsx
```

### Property Components (14 files)
```
components/property/
├── create/createPropertyModal.tsx
├── delete/deletePropertyModal.tsx
├── preview/previewPropertyModal.tsx
├── update/updatePropertyModal.tsx
├── channels/
│   ├── channelForm.tsx
│   ├── channelIconRow.tsx
│   └── channelList.tsx
└── import/
    ├── bulkImportPropertyModal.tsx
    └── steps/
        ├── AssignClientsStep.tsx
        ├── MapFieldsStep.tsx
        ├── PreviewStep.tsx
        └── UploadStep.tsx
```

### Property-Related Components (5 files)
```
components/property-channel/propertyChannelModal.tsx
components/property-field-mapping/
├── FieldMappingEditor.tsx
└── propertyFieldMappingModal.tsx
components/property-license/propertyLicenseModal.tsx
components/property-owners/propertyOwnersModal.tsx
```

### Report Components (2 files)
```
components/report/
├── generate/generateReportModal.tsx
└── view/viewReportModal.tsx
```

### Session Components (2 files)
```
components/session/
├── SessionExpiredModal.tsx
└── SessionWarningModal.tsx
```

### Upload Wizard Components (15 files)
```
components/upload-wizard/
├── UploadWizard.tsx              # Main wizard container
├── ResumeDraftModal.tsx          # Draft resume/discard prompt
├── shared/
│   ├── StepIndicator.tsx
│   ├── ColumnMapper.tsx
│   ├── DataPreviewTable.tsx
│   └── ValidationWarning.tsx
├── steps/
│   ├── CompleteStep.tsx
│   ├── FieldMappingStep.tsx
│   ├── PreviewStep.tsx
│   ├── ProcessStep.tsx
│   ├── PropertyIdentificationStep.tsx
│   ├── PropertyMappingStep.tsx
│   ├── UploadStep.tsx
│   └── ValidateStep.tsx
└── types/wizard.ts
```

### Other Components (12 files)
```
components/calculation-rules/calculationRuleModal.tsx
components/csv-mapping/FieldMappingForm.tsx
components/field-value-changed/EditFieldModal.tsx
components/footer/Footer.tsx
components/global-field-mapping/GlobalFieldMappingModal.tsx
components/incoming-bookings/ReviewIncomingBookingsModal.tsx
components/navbar/
├── ManagerSidebar.tsx
├── PreNavbar.tsx
└── UserNavbar.tsx
components/pms-credential/pmsCredentialModal.tsx
components/webhook-mapping/
├── SearchableSelect.tsx
└── WebhookFieldMappingForm.tsx
```

### Shared Components (7 files)
```
components/shared/
├── DualModalContainer.tsx
├── FloatingActionButton.tsx
├── LogoutModal.tsx
├── SearchableSelect.tsx
├── TableActionsDropdown.tsx
├── modal.tsx
└── notification.tsx
```

---

## Complete Service Inventory (25 services)

### Core Services
| Service | File | Purpose |
|---------|------|---------|
| API Client | `apiClient.ts` | HTTP fetch wrapper with credentials |
| Auth | `authService.ts` | Login, signup, password reset |
| Profile | `profileService.ts` | User profile CRUD |

### Resource Services
| Service | File | Purpose |
|---------|------|---------|
| Client | `clientService.ts` | Client CRUD + bulk import |
| Client Agreement | `clientAgreementService.ts` | Agreement file management |
| Client Code | `clientCodeService.ts` | Status code management |
| Client Note | `clientNoteService.ts` | Client notes CRUD |
| Property | `propertyService.ts` | Property CRUD + search + stats |
| Property Channel | `propertyChannelService.ts` | Distribution channels |
| Property License | `propertyLicenseService.ts` | License file management |
| Property Field Mapping | `propertyFieldMappingService.ts` | CSV field mapping templates |
| Property Webhook Mapping | `propertyWebhookMappingService.ts` | Webhook field mappings |
| Booking | `bookingService.ts` | Booking CRUD + search |
| Report | `reportService.ts` | Report generation + file management |
| Expense | `expenseService.ts` | Expense CRUD |
| Expense Categories | `expenseCategoriesService.ts` | Expense category management |

### Specialized Services
| Service | File | Purpose |
|---------|------|---------|
| CSV Upload | `csvUploadService.ts` | Multi-property CSV import |
| Analytics | `analyticsService.ts` | Analytics API + date helpers |
| Dashboard | `dashboardService.ts` | Dashboard metrics + activity |
| PMS Credential | `pmsCredentialService.ts` | PMS login storage |
| Guesty Connection | `guestyConnectionService.ts` | Guesty PMS integration |
| Hostaway Connection | `hostawayConnectionService.ts` | Hostaway PMS integration |
| Incoming Booking | `incomingBookingService.ts` | Webhook booking processing |
| Calculation Rule | `calculationRuleService.ts` | Formula rules for calculations |
| Field Values Changed | `fieldValuesChangedService.ts` | Booking field edit history |

---

## Complete Type Definitions (24 files)

```
services/types/
├── analytics.ts           # AnalyticsData, PortfolioData, TimelinePoint
├── auth.ts                # LoginPayload, SignupPayload, AuthResponse
├── booking.ts             # Booking, BookingPayload, BookingsResponse
├── calculationRule.ts     # CalculationRule, RuleTemplate
├── client.ts              # Client, ClientPayload, ClientsResponse
├── clientAgreement.ts     # ClientAgreement, AgreementPayload
├── clientCode.ts          # StatusCode, StatusCodePayload
├── clientNote.ts          # ClientNote, NotePayload
├── csvMapping.ts          # FieldMapping, MappingConfig
├── dashboard.ts           # DashboardMetrics, DashboardActivity
├── expense.ts             # Expense, ExpensePayload
├── expenseCategories.ts   # ExpenseCategory, CategoryPayload
├── fieldValueChanged.ts   # FieldChange, ChangeHistory
├── guestyConnection.ts    # GuestyConnection, ConnectionPayload
├── hostawayConnection.ts  # HostawayConnection, ConnectionPayload
├── incomingBooking.ts     # IncomingBooking, WebhookData
├── pmsCredential.ts       # PMSCredential, CredentialPayload
├── profile.ts             # UserProfile, ProfilePayload
├── property.ts            # Property, PropertyPayload, Owner
├── propertyChannel.ts     # PropertyChannel, ChannelPayload
├── propertyFieldMapping.ts # FieldMappingTemplate, MappingPayload
├── propertyLicense.ts     # PropertyLicense, LicensePayload
├── propertyWebhookMapping.ts # WebhookMapping, MappingConfig
└── report.ts              # Report, ReportFile, ReportPayload
```

---

## State Management (3 Stores)

### useUserStore (`src/store/useUserStore.ts`)
```typescript
interface UserState {
  user: UserProfile | null
  setUser: (user: UserProfile | null) => void
  clearUser: () => void
}
// Persisted to localStorage
```

### useNotificationStore (`src/store/useNotificationStore.ts`)
```typescript
interface NotificationState {
  notifications: Notification[]
  showNotification: (message: string, type: 'success' | 'error' | 'info') => void
  removeNotification: (id: string) => void
}
// Not persisted
```

### useAnalyticsStore (`src/store/useAnalyticsStore.ts`)
```typescript
interface AnalyticsState {
  filters: AnalyticsFilters
  granularity: 'daily' | 'weekly' | 'monthly'
  analyticsData: AnalyticsData | null
  bookingsData: BookingsData | null
  aiInsights: AIInsights | null
  drillDown: DrillDownContext | null
  // Actions...
}
// Not persisted (fresh data each session)
```

---

## Custom Hooks (2 files)

### useSessionMonitor (`src/hooks/useSessionMonitor.ts`)
```typescript
// Monitors Supabase session for expiration
// Shows warning modal 5 minutes before expiry
// Shows expired modal and redirects to login on expiry
```

### useWizardDraft (`src/hooks/useWizardDraft.ts`)
```typescript
// Manages Upload Wizard draft save/restore to localStorage
// - Auto-saves on step transitions (not every interaction)
// - Manual save via "Save Draft" button in step footers
// - Shows ResumeDraftModal on page load if draft exists
// - Stores raw CSV text (not File object) + wizard state
// - localStorage key: 'upload-wizard-draft'
// - Clears draft on wizard completion or reset
```

---

## Utilities (8 files)

### Supabase Utilities
```
utils/supabase/
├── api.ts              # Server-side Supabase client
├── component.ts        # Client component Supabase client
├── server-props.ts     # getServerSideProps helper
└── static-props.ts     # getStaticProps helper
```

### General Utilities
```
utils/
├── csvParser.ts            # CSV parsing and validation
├── webhookFieldProcessor.ts # Webhook data transformation
└── logoutState.ts          # Cross-tab logout coordination
```

### UI Utilities (in services/)
```
services/
└── channelUtils.tsx        # Channel icons and display name React helpers
```

---

## Backend API Routes

### Core Resources
| Resource | Endpoints | Status |
|----------|-----------|--------|
| Properties | GET, GET/:id, GET/search, GET/stats, POST, PUT, DELETE, PATCH/:id/status | ✅ |
| Property Owners | GET/:propertyId, POST, PUT/:id, DELETE/:id | ✅ |
| Property Channels | GET/:propertyId, POST, PUT/:id, DELETE/:id | ✅ |
| Property Licenses | GET/:propertyId, POST, PUT/:id, DELETE/:id | ✅ |
| Property Field Mappings | GET/:propertyId, POST, PUT/:id, DELETE/:id | ✅ |
| Property Webhook Mappings | GET/:propertyId, POST, PUT/:id, DELETE/:id | ✅ |
| Clients | GET, POST, PUT, DELETE | ✅ |
| Client Status Codes | GET, POST, PUT, DELETE | ✅ |
| Client Agreements | GET, POST, PUT, DELETE | ✅ |
| Client Notes | GET, POST, PUT, DELETE | ✅ |
| PMS Credentials | GET, POST, PUT, DELETE | ✅ |
| Profiles | GET, POST, PUT | ✅ |

### Bookings & Reports
| Resource | Endpoints | Status |
|----------|-----------|--------|
| Bookings | GET, GET/:id, POST, PUT, DELETE | ✅ |
| Reports | GET, GET/:id, POST, DELETE | ✅ |
| Report Preview | POST /preview | ✅ |
| Report Generate | POST /generate | ✅ |
| Report Files | DELETE /files/:fileId | ✅ |
| Logos | GET, POST /upload-logo | ✅ |

### Analytics & Dashboard
| Resource | Endpoints | Status |
|----------|-----------|--------|
| Analytics | POST / (main data) | ✅ |
| Analytics Bookings | POST /bookings (drill-down) | ✅ |
| AI Insights | GET /ai-insights | ✅ |
| Dashboard Alerts | GET /alerts | ✅ |
| Dashboard Metrics | GET /metrics | ✅ |
| Dashboard Insights | GET /insights | ✅ |
| Dashboard Activity | GET /activity | ✅ |

### CSV & Integrations
| Resource | Endpoints | Status |
|----------|-----------|--------|
| CSV Upload | POST /upload, POST /process | ✅ |
| Calculation Rules | GET, POST, PUT, DELETE | ✅ |
| Incoming Bookings | GET, POST /review, POST /import | ✅ |
| Hostaway Connection | GET, POST, PUT, DELETE | ✅ |
| Guesty Connection | GET, POST, PUT, DELETE | ✅ |
| Expenses | GET, POST, PUT, DELETE | ✅ |
| Expense Categories | GET, POST, PUT, DELETE | ✅ |

---

## Analytics API Details

### POST /api/analytics - Main Dashboard Data
```typescript
// Request
{
  dateRange: { startDate: string, endDate: string },
  propertyIds: string[],      // [] = all properties
  channels: string[],         // [] = all channels
  comparison: boolean,        // Include previous period
  granularity: 'daily' | 'weekly' | 'monthly'
}

// Response
{
  portfolio: { current: KPIs, previous: KPIs, delta: Deltas },
  byProperty: PropertyBreakdown[],
  byChannel: ChannelBreakdown[],
  timeline: TimelinePoint[]
}
```

### POST /api/analytics/bookings - Drill-Down
```typescript
// Request (filter by property/channel/date)
{
  dateRange: { startDate: string, endDate: string },
  propertyIds: string[],
  channels: string[],
  page: number,
  limit: number
}

// Response: Paginated booking list
```

### GET /api/analytics/ai-insights - AI Summary
```typescript
// Uses last complete calendar week automatically
// Response: { available: boolean, markdown: string, period: {...} }
```

---

## Implemented Features (Complete)

### Authentication System
- Complete auth flow (signup/login/logout)
- Email verification with Supabase
- Password reset functionality
- Session monitoring with warning/expiry modals
- Role-based redirects

### Client Management
- Full CRUD (Create/Read/Update/Delete)
- Bulk import from CSV
- Status code system (custom statuses)
- Client agreements file management
- Client notes
- PMS credentials storage
- Client preview with all details

### Property Management
- Full CRUD with multi-owner support
- Property search and filtering
- Stats dashboard (active/inactive counts)
- **Owners Management:**
  - Add/remove co-owners
  - Primary owner designation
  - Commission rate override per owner
- **Channels Management:**
  - Multiple distribution channels (Airbnb, VRBO, Booking.com, Google, Direct, Expedia)
  - Channel icons and display names
  - Active/Inactive status toggle
- **Licenses Management:**
  - File upload and storage
  - License title and notes
- **Field Mapping Templates:**
  - Reusable CSV field mappings per property
  - Default template auto-loading

### Reports Management
- **Multi-Format Generation:**
  - PDF with custom logo
  - CSV export
  - Excel export
- **Report Preview:**
  - PDF preview in new tab (base64)
  - CSV/Excel data table preview
- **Financial Summary:**
  - Revenue breakdown (room, extra fees, cleaning)
  - Tax calculations (GST, QST, lodging, sales)
  - Platform fees (channel, Stripe, management)
  - Net earnings and totals
- **File Management:**
  - Version history per format
  - Individual file download
  - File deletion with confirmation

### CSV Upload Wizard (6 Steps)
1. **Upload** - File selection and parsing
2. **Property Identification** - Map CSV listings to properties
3. **Field Mapping** - Global or per-property column mapping
4. **Preview** - Review before import
5. **Process** - Import with progress tracking
6. **Complete** - Summary and next steps

**Draft Save/Resume Feature:**
- Auto-saves to localStorage on step transitions
- Manual "Save Draft" button in step footers (steps 2-4)
- `ResumeDraftModal` prompts to resume or start fresh on page load
- Stores raw CSV text + wizard state (File objects can't be serialized)
- Draft cleared on successful completion or "Upload Another"
- Key files: `useWizardDraft.ts`, `ResumeDraftModal.tsx`

### Dashboard (Operational Hub)
- **Action Bar (Sticky):**
  - Upload CSV, Generate Report, New Client, New Property
  - Amber gradient primary actions
- **Alerts Zone:**
  - Properties missing bookings
  - Properties without reports
  - "All Clear" state
- **Metrics Zone (6 cards):**
  - Properties (active/total/inactive)
  - Clients (total with breakdown)
  - CSV Uploads (month comparison)
  - Reports Generated (month comparison)
  - Bookings (month comparison)
  - Revenue (month comparison)
- **Performance Insights:**
  - Properties with >15% change
  - Sparkline charts
  - Quick actions
- **Activity Feed:**
  - Timeline design
  - Color-coded activity types
  - Hover-reveal actions

### Analytics System
- **KPI Grid (7 metrics):**
  - Total Payout
  - Net Earnings
  - Management Fee
  - Occupancy Rate
  - Total Bookings
  - Total Nights
  - Average Nightly Rate
- **Timeline Chart:**
  - Line/Area/Bar modes
  - Multiple metric selection
  - Granularity: daily/weekly/monthly
- **Breakdowns:**
  - By Property (table/bar/pie)
  - By Channel (table/bar/pie)
- **Drill-Down Modal:**
  - Paginated booking list
  - Filtered by property/channel
- **AI Insights:**
  - Weekly AI-generated summary
  - Markdown rendering

### Bookings Management
- Full CRUD operations
- Field edit history tracking
- Incoming bookings from webhooks
- Review and import workflow
- **Booking Financial Fields:**
  - Base: `nightlyRate`, `extraGuestFees`, `cleaningFee`, `bedLinenFee`
  - Taxes: `gst`, `qst`, `lodgingTax`, `salesTax`
  - Fees: `channelFee`, `stripeFee`, `mgmtFee`, `cohostFee`
  - Totals: `totalPayout`, `netEarnings`
  - Collected: `rentCollected`, `taxesCollected`
- **New Fields (Jan 2026):** `rentCollected`, `taxesCollected`, `cohostFee`
  - User-defined via CSV field mapping formulas
  - Calculated at import time (multi-pass formula evaluation)
  - Available in booking create/update/preview modals

### Expense Management
- Full CRUD operations
- Category management
- Receipt file upload
- Reimbursable/Tax deductible flags

### PMS Integrations
- **Hostaway:**
  - API connection
  - Webhook setup
  - Auto-import toggle
- **Guesty:**
  - API connection
  - Webhook setup
  - Auto-import toggle

### Settings
- Profile management
- Status code customization
- Calculation rules configuration

---

## Database Schema (Key Tables)

| Table | Purpose |
|-------|---------|
| profiles | User accounts |
| clients | Property owners (clients of the property manager) |
| client_status_codes | Custom client status labels |
| client_agreements | Agreement file metadata |
| client_notes | Notes about clients |
| client_pms_credentials | PMS login storage |
| properties | Rental properties |
| client_properties | Many-to-many: owners ↔ properties |
| property_channels | Distribution channels per property |
| property_licenses | License files per property |
| property_field_mappings | CSV field mapping templates |
| property_webhook_mappings | Webhook field mappings |
| bookings | Individual reservations |
| csv_uploads | CSV upload metadata |
| reports | Report metadata |
| report_files | Generated report files |
| report_properties | Many-to-many: reports ↔ properties |
| expenses | Property expenses |
| expense_categories | User-defined expense categories |
| calculation_rules | Formula rules for calculations |
| calculation_rule_templates | Reusable rule sets |
| field_values_changed | Booking field edit history |
| incoming_bookings | Webhook bookings pending review |
| hostaway_connections | Hostaway API credentials |
| guesty_connections | Guesty API credentials |
| logos | Uploaded logos for reports |

---

## Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Components | PascalCase | `CreateClientModal` |
| Component files | PascalCase | `createPropertyModal.tsx` |
| Folders | lowercase-hyphenated | `client-agreement/` |
| Service files | camelCase | `clientService.ts` |
| Functions | camelCase | `getClients()` |
| Hooks | camelCase with use | `useUserStore()` |
| Types/Interfaces | PascalCase | `Client`, `UserProfile` |
| Constants | UPPER_SNAKE_CASE | `API_BASE_URL` |

---

## Code Patterns

### Modal Component Pattern
```typescript
'use client'

interface Create[Resource]ModalProps {
  isOpen: boolean
  onClose: () => void
  onAdd: (new[Resource]: [Resource]) => void
}

const Create[Resource]Modal: React.FC<Create[Resource]ModalProps> = ({ ... }) => {
  // 1. Form state (useState)
  // 2. Global state (Zustand stores)
  // 3. useEffect to reset form on open
  // 4. handleSubmit with validation → API call → notification
  // 5. Return Modal wrapper with form
}
```

### Service Function Pattern
```typescript
export async function getResources(userId: string): Promise<ResourcesResponse> {
  return apiClient<ResourcesResponse>(`/resources?userId=${userId}`)
}

export async function createResource(data: CreatePayload): Promise<ResourceResponse> {
  return apiClient<ResourceResponse, CreatePayload>('/resources', {
    method: 'POST',
    body: data,
  })
}
```

### Error Handling Pattern
```typescript
try {
  const res = await apiCall(data)
  if (res.status === 'success') {
    showNotification('Success', 'success')
    onClose()
  } else {
    showNotification(res.message || 'Failed', 'error')
  }
} catch (err) {
  console.error('Error:', err)
  showNotification(err instanceof Error ? err.message : 'Network error', 'error')
}
```

---

## Environment Variables

```bash
# .env.local
NEXT_PUBLIC_BASE_URL=http://localhost:4000
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key
```

---

## Key Files to Reference

### Services
- `src/services/apiClient.ts` - HTTP client with credentials
- `src/services/propertyService.ts` - Complete CRUD example
- `src/services/analyticsService.ts` - Analytics API + date helpers
- `src/services/reportService.ts` - Multi-format report generation

### Components
- `src/components/property/create/createPropertyModal.tsx` - Modal pattern
- `src/components/analytics/AnalyticsWidget.tsx` - Composite widget
- `src/components/dashboard/MetricsZone/MetricsGrid.tsx` - Dashboard metrics
- `src/components/upload-wizard/UploadWizard.tsx` - Multi-step wizard

### Pages
- `src/app/(user)/property-manager/properties/page.tsx` - List page pattern
- `src/app/(user)/property-manager/analytics/page.tsx` - Analytics page
- `src/app/(user)/property-manager/dashboard/page.tsx` - Dashboard

### State
- `src/store/useUserStore.ts` - Persisted auth state
- `src/store/useAnalyticsStore.ts` - Analytics state management

---

## Claude Code Skills

Custom skills in `.claude/skills/`:

1. **`thesmarthost-context`** - Business context and backend API reference
2. **`add-frontend-feature`** - UI component patterns and styling
3. **`connect-to-backend-api`** - Service layer patterns

---

**Project:** HostMetrics for TheSmartHost Co. Inc
**Team:** Mark Cena (Calgary) + Hussein Saab (Toronto)
**Contact:** husseinsaab14@gmail.com, markjpcena@gmail.com

---

**Last Updated:** January 22, 2026

---
> Source: [TheSmartHost-Co/thesmarthost](https://github.com/TheSmartHost-Co/thesmarthost) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
