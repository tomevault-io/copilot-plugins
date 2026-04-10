## nestly

> **Family Planner** is a comprehensive React-based family planning application designed for Surface tablets. It runs entirely client-side with no backend, integrating with Microsoft services and Azure OpenAI for intelligent features.

# AGENTS.md - Developer & AI Agent Guide

## Project Overview

**Family Planner** is a comprehensive React-based family planning application designed for Surface tablets. It runs entirely client-side with no backend, integrating with Microsoft services and Azure OpenAI for intelligent features.

### Key Features
- Multi-account Microsoft calendar synchronization
- OneDrive photo slideshow
- AI-powered meal planning with Azure OpenAI
- Microsoft To Do task management
- Touch-optimized tablet interface
- Screen wake lock for always-on display
- Offline support with LocalStorage caching

---

## Development Setup & Port Configuration

### Fixed Port Strategy

**Port:** Always uses port **5173** (standard Vite default)

**Configuration:** `vite.config.ts`
```typescript
server: {
  port: 5173,
  strictPort: true, // Fails if port is in use instead of trying alternatives
  host: true,       // Listens on all addresses for network access
}
```

**Why Fixed Port?**
- Consistent OAuth redirect URI (`http://localhost:5173/auth/callback`)
- Easier debugging (no need to update port in Azure AD app registration)
- Predictable development environment
- Simpler documentation and setup instructions

**If Port is in Use:**
1. Kill existing processes: `lsof -ti:5173 | xargs kill -9`
2. Or change the port in both `vite.config.ts` and Azure AD redirect URI

### Running the Application

```bash
# Development server
npm run dev

# Production build
npm run build

# Preview production build
npm run preview

# Lint
npm run lint
```

**Access URLs:**
- Local: http://localhost:5173
- Network: http://[your-ip]:5173 (for testing on actual Surface tablet)

### Playwright MCP Server for Visual Verification

**⭐ BEST PRACTICE:** This project uses the Playwright MCP server to enable AI agents (like Claude Code) to visually inspect and interact with the running application.

**Why This Matters:**
- **See What You Build**: AI agents can take screenshots, navigate pages, and verify UI components
- **Fast Feedback Loop**: Instantly verify that UI changes look correct without manual testing
- **Touch Target Verification**: Programmatically check that buttons meet the 44x44px requirement
- **Accessibility Testing**: Inspect the accessibility tree to ensure proper semantic structure
- **Automated Visual Regression**: Compare UI snapshots to detect unintended changes

**Setup (Already Configured):**
```bash
# The Playwright MCP server is already configured for this project
# It was added using:
claude mcp add playwright npx @playwright/mcp@latest
```

**How AI Agents Use It:**
When implementing or modifying UI features, AI agents should:

1. **Start the dev server**: `npm run dev`
2. **Navigate to the page**: Use MCP `browser_navigate` tool to open http://localhost:5173
3. **Take screenshots**: Use `browser_take_screenshot` to capture visual state
4. **Inspect accessibility**: Use `browser_snapshot` to get structured page content
5. **Verify interactions**: Use `browser_click`, `browser_type`, etc. to test user flows
6. **Check touch targets**: Use `browser_evaluate` to measure button sizes programmatically

**Example AI Agent Workflow:**
```typescript
// 1. Navigate to the app
await browser_navigate({ url: "http://localhost:5173" });

// 2. Take a screenshot to see the current state
await browser_take_screenshot({ filename: "homepage.png" });

// 3. Get accessibility snapshot to understand structure
await browser_snapshot();

// 4. Click the FAB menu button
await browser_click({
  element: "FAB menu button",
  ref: "[data-testid='fab-menu-button']"
});

// 5. Verify the menu opened
await browser_take_screenshot({ filename: "fab-menu-open.png" });

// 6. Check button sizes meet touch requirements
await browser_evaluate({
  function: `() => {
    const buttons = document.querySelectorAll('button');
    return Array.from(buttons).map(btn => {
      const rect = btn.getBoundingBox();
      return {
        text: btn.textContent,
        width: rect.width,
        height: rect.height,
        meetsTouchTarget: rect.width >= 44 && rect.height >= 44
      };
    });
  }`
});
```

**Benefits for This Project:**
- **Touch Optimization**: Verify all buttons meet 44x44px minimum for Surface tablet
- **FAB Menu**: Confirm the floating action button is positioned correctly
- **Calendar Views**: Visually verify week/day/month view layouts
- **Modal Dialogs**: Check that modals display properly and are centered
- **Responsive Design**: Test layouts at different viewport sizes (desktop, tablet)
- **Color Coding**: Verify calendar events display with correct colors

**MCP Server Configuration:**
The Playwright MCP server runs with default settings:
- Headed browser mode (you can see what the AI agent is doing)
- Persistent browser profile (login state preserved)
- Full access to browser automation tools
- Screenshot and accessibility snapshot capabilities

**When to Use:**
- Implementing new UI components
- Modifying existing layouts
- Debugging visual issues
- Verifying responsive behavior
- Checking touch target sizes
- Validating accessibility
- Testing user interaction flows

---

## Tech Stack

### Core Framework
- **React 18.2+** - UI framework with concurrent features
- **TypeScript 5.9** - Type safety and developer experience
- **Vite 7.2+** - Build tool and development server (fast HMR)

### Styling
- **Tailwind CSS 3.4** - Utility-first CSS framework
- **PostCSS 8.4** - CSS processing
- **Autoprefixer 10.4** - Vendor prefix automation

### Routing & State
- **React Router v7** - Client-side routing
- **React Context API** - Global state management (no Redux)
- **React Hooks** - All components are functional with hooks

### Authentication & APIs
- **@azure/msal-browser 4.27+** - Microsoft OAuth 2.0 authentication
- **Microsoft Graph API** - Calendar, OneDrive, To Do integration
- **Azure OpenAI** - AI recipe generation via REST API
- **date-fns 4.1+** - Date manipulation and formatting

### UI Components
- **lucide-react 0.562+** - Icon library (modern, tree-shakeable)
- Custom components (no UI library like Material-UI or shadcn/ui)

### Browser APIs
- **Screen Wake Lock API** - Prevent tablet sleep
- **LocalStorage API** - Offline data persistence
- **Fetch API** - HTTP requests to Microsoft Graph and Azure OpenAI

---

## Core Principles & Architecture Decisions

### 1. Client-Side Only Architecture

**No Backend Server**
- Application runs entirely in the browser
- All data stored in LocalStorage or fetched from Microsoft/Azure APIs
- OAuth tokens managed client-side (acceptable for localhost-only app)

**Implications:**
- Cannot use client secrets (must use PKCE flow for OAuth)
- API keys exposed in browser (Azure OpenAI key is necessary evil)
- No server-side rendering or API routes
- No database - LocalStorage is the persistence layer

### 2. Multi-Account First

**Design Philosophy:**
- Support multiple Microsoft accounts simultaneously (e.g., husband + wife)
- Each account has independent authentication state
- Calendars, tasks, and files from all accounts merged into unified views
- Color-coding distinguishes content from different accounts/calendars

**Implementation:**
- Array of `AuthAccount` objects in AuthContext
- Each account has separate access/refresh tokens
- Services accept `accountId` parameter to identify which account to use
- Independent token refresh logic per account

### 3. Touch-First Design

**Tablet Optimization:**
- Minimum touch target size: **44x44 pixels** (Apple/W3C guideline)
- No hover-dependent interactions
- Large, finger-friendly buttons and controls
- FAB (Floating Action Button) menu for thumb-zone navigation
- Swipe gestures where appropriate (future enhancement)

**CSS Classes:**
```css
.touch-target {
  min-width: 44px;
  min-height: 44px;
}

.btn-icon {
  min-width: 44px;
  min-height: 44px;
}
```

### 4. Always Maintain Playwright Tests

**CRITICAL BEST PRACTICE:** All UI features MUST have corresponding Playwright tests that are kept in sync.

**Why This Matters:**
- Prevents regressions when adding new features
- Documents expected behavior
- Validates touch target requirements automatically
- Ensures responsive design works across viewports
- Catches console errors and accessibility issues

**When Implementing UI Features:**
1. **Add tests IMMEDIATELY** after implementing the feature
2. **Update existing tests** when modifying UI components
3. **Run tests** before considering work complete
4. **Keep test coverage high** - aim for all pages and major interactions

**Test Organization:**
```
e2e/
  ui-validation.spec.ts       # Core UI validation suite
  app.spec.ts                  # Basic smoke tests
  [feature].spec.ts            # Feature-specific tests
```

**Required Test Coverage:**
- ✅ All pages load and render
- ✅ Touch targets meet 44x44px minimum
- ✅ Responsive design (tablet + desktop)
- ✅ No critical console errors
- ✅ Key user interactions work
- ✅ Navigation flows complete successfully

**Example Test Pattern:**
```typescript
test('Feature X works correctly', async ({ page }) => {
  await page.goto('/feature');
  await page.waitForLoadState('networkidle');

  // Verify UI elements
  await expect(page.locator('.key-element')).toBeVisible();

  // Test interaction
  await page.click('.action-button');

  // Verify result
  await expect(page.locator('.result')).toContainText('Expected');

  // Screenshot for visual verification
  await page.screenshot({ path: 'test-screenshots/feature-x.png' });
});
```

**Running Tests:**
```bash
# Run all tests
npm test

# Run in UI mode (recommended for development)
npm run test:ui

# Run with browser visible
npm run test:headed

# Debug specific test
npm run test:debug
```

**When to Update Tests:**
- ✅ After adding new UI components
- ✅ After modifying existing layouts
- ✅ After changing navigation structure
- ✅ After updating button sizes or styles
- ✅ After adding new pages or routes
- ✅ After fixing bugs (add regression test)

**Test Failure Policy:**
- ❌ **DO NOT** ignore failing tests
- ❌ **DO NOT** delete tests to make them pass
- ✅ **DO** fix the code or update the test appropriately
- ✅ **DO** investigate why tests fail before proceeding

### 5. Offline-First with Caching

**LocalStorage Strategy:**
- Cache all fetched data with timestamps
- Display cached data immediately on load
- Sync in background and update UI
- Queue write operations when offline (future enhancement)

**Cache Keys:**
```typescript
STORAGE_KEYS = {
  AUTH_ACCOUNTS: 'planner_auth_accounts',
  CALENDAR_CACHE: 'planner_calendar_cache',
  PHOTO_FOLDER: 'planner_photo_folder',
  FRIDGE_INVENTORY: 'planner_fridge_inventory',
  SAVED_RECIPES: 'planner_saved_recipes',
  SETTINGS: 'planner_settings',
}
```

### 6. Service Layer Abstraction

**Pattern:**
- Separate API logic from UI components
- Services handle all external API calls
- Components consume services via Context providers
- Type-safe interfaces for all API responses

**Structure:**
```
src/services/
  auth.service.ts       → MSAL OAuth operations
  graph.service.ts      → Generic Microsoft Graph client
  calendar.service.ts   → Calendar-specific operations
  onedrive.service.ts   → OneDrive file operations
  openai.service.ts     → Azure OpenAI recipe generation
  todo.service.ts       → Microsoft To Do operations
  storage.service.ts    → LocalStorage wrapper
```

### 7. Context-Based State Management

**No Redux - Just React Context + Hooks**

**Why?**
- Simpler mental model for small-to-medium apps
- Built-in React feature (no extra dependency)
- Sufficient for this application's complexity
- Hooks (useCallback, useMemo) prevent re-renders

**Context Providers:**
```typescript
<AuthProvider>
  <CalendarProvider>
    <PhotoProvider>
      <MealProvider>
        <TaskProvider>
          <App />
        </TaskProvider>
      </MealProvider>
    </PhotoProvider>
  </CalendarProvider>
</AuthProvider>
```

**Pattern:**
```typescript
// Context provides state + operations
interface CalendarContextType {
  // State
  calendars: Calendar[];
  events: CalendarEvent[];
  loading: boolean;
  error: string | null;

  // Operations
  syncCalendars: () => Promise<void>;
  createEvent: (input: CreateEventInput) => Promise<void>;
  deleteEvent: (eventId: string, accountId: string) => Promise<void>;
}
```

### 8. Optimistic UI Updates

**Pattern:**
- Update UI immediately on user action
- Send API request in background
- Rollback if API call fails

**Example (Event Creation):**
```typescript
const createEvent = async (input: CreateEventInput) => {
  // 1. Optimistically add to local state
  const tempEvent = { id: 'temp-' + Date.now(), ...input };
  setEvents(prev => [...prev, tempEvent]);

  // 2. Call API
  try {
    const realEvent = await calendarService.createEvent(token, input);
    // 3. Replace temp with real
    setEvents(prev => prev.map(e => e.id === tempEvent.id ? realEvent : e));
  } catch (error) {
    // 4. Rollback on failure
    setEvents(prev => prev.filter(e => e.id !== tempEvent.id));
    showError('Failed to create event');
  }
};
```

---

## Project Structure

```
planner/
├── src/
│   ├── components/           # React components (all functional)
│   │   ├── auth/            # Authentication UI
│   │   │   ├── LoginButton.tsx
│   │   │   ├── AccountManager.tsx
│   │   │   └── AuthCallback.tsx
│   │   ├── calendar/        # Calendar views and event management
│   │   │   ├── WeekView.tsx
│   │   │   ├── DayView.tsx
│   │   │   ├── MonthView.tsx
│   │   │   ├── EventCard.tsx
│   │   │   ├── CreateEventModal.tsx
│   │   │   └── ViewSwitcher.tsx
│   │   ├── photos/          # OneDrive photo slideshow
│   │   │   ├── PhotoSlideshow.tsx
│   │   │   └── FolderPicker.tsx
│   │   ├── meals/           # Meal planning and recipes
│   │   │   ├── FridgeInventory.tsx
│   │   │   └── RecipeGenerator.tsx
│   │   ├── tasks/           # Microsoft To Do integration
│   │   │   ├── TaskList.tsx
│   │   │   └── CreateTaskModal.tsx
│   │   └── layout/          # Layout and navigation
│   │       ├── FABMenu.tsx
│   │       ├── Header.tsx
│   │       └── MainLayout.tsx
│   │
│   ├── contexts/            # React Context providers
│   │   ├── AuthContext.tsx         # Multi-account auth state
│   │   ├── CalendarContext.tsx     # Calendar data and sync
│   │   ├── PhotoContext.tsx        # Photo slideshow state
│   │   ├── MealContext.tsx         # Meal planning state
│   │   └── TaskContext.tsx         # To Do tasks state
│   │
│   ├── services/            # API and business logic
│   │   ├── auth.service.ts         # MSAL OAuth implementation
│   │   ├── graph.service.ts        # Microsoft Graph API client
│   │   ├── calendar.service.ts     # Calendar CRUD operations
│   │   ├── onedrive.service.ts     # OneDrive file access
│   │   ├── openai.service.ts       # Azure OpenAI integration
│   │   ├── todo.service.ts         # Microsoft To Do operations
│   │   └── storage.service.ts      # LocalStorage wrapper
│   │
│   ├── types/               # TypeScript type definitions
│   │   ├── auth.types.ts
│   │   ├── calendar.types.ts
│   │   ├── meal.types.ts
│   │   ├── photo.types.ts
│   │   └── task.types.ts
│   │
│   ├── hooks/               # Custom React hooks
│   │   ├── useWakeLock.ts         # Screen wake lock management
│   │   └── (future hooks here)
│   │
│   ├── utils/               # Utility functions
│   │   └── dateHelpers.ts         # Date formatting with date-fns
│   │
│   ├── config/              # Application configuration
│   │   └── app.config.ts          # Centralized config (API scopes, intervals, etc.)
│   │
│   ├── pages/               # Page-level components
│   │   ├── CalendarPage.tsx
│   │   ├── PhotosPage.tsx
│   │   ├── MealsPage.tsx
│   │   ├── TasksPage.tsx
│   │   └── SettingsPage.tsx
│   │
│   ├── App.tsx              # Root component with router
│   ├── main.tsx             # Entry point (ReactDOM.render)
│   └── index.css            # Global styles (Tailwind + custom)
│
├── public/                  # Static assets
├── e2e/                     # End-to-end tests (Playwright)
│   ├── ui-validation.spec.ts    # Core UI validation suite
│   ├── app.spec.ts              # Basic smoke tests
│   ├── README.md                # Test documentation
│   └── MCP_VISUAL_TESTING.md    # MCP testing guide
│
├── .reports/                # All reports and summaries (YYYY-MM-DD_NAME.md format)
│   ├── README.md                # Report conventions and guidelines
│   └── 2026-01-05_UI_TEST_RESULTS.md   # Example: UI testing report
│
├── .env.example             # Environment variables template
├── .env                     # Environment variables (gitignored)
├── .gitignore
├── package.json
├── package-lock.json
├── tsconfig.json            # TypeScript configuration
├── vite.config.ts           # Vite configuration (port 5173)
├── playwright.config.ts     # Playwright test configuration
├── tailwind.config.js       # Tailwind theme and plugins
├── postcss.config.js        # PostCSS configuration
├── README.md                # User-facing documentation
└── AGENTS.md                # This file - Developer/AI documentation
```

---

## Authentication Flow

### OAuth 2.0 with MSAL (PKCE Flow)

**Flow:**
1. User clicks "Add Account" button
2. MSAL opens Microsoft login popup
3. User authenticates and consents to permissions
4. Microsoft redirects to `/auth/callback` with auth code
5. MSAL exchanges code for access token + refresh token (PKCE)
6. Store tokens and profile in LocalStorage
7. User can repeat to add multiple accounts

**Required Microsoft Graph Scopes:**
```typescript
const SCOPES = [
  'User.Read',                    // Basic profile info
  'Calendars.ReadWrite',          // Read/write user calendars
  'Calendars.ReadWrite.Shared',   // Access shared calendars
  'Files.Read',                   // Read OneDrive files
  'Tasks.ReadWrite',              // Read/write Microsoft To Do tasks
  'offline_access',               // Refresh tokens for long-lived sessions
];
```

**Token Refresh Strategy:**
```typescript
// Check expiry before each API call (5-minute buffer)
const getAccessToken = async (accountId: string) => {
  const account = accounts.find(a => a.homeAccountId === accountId);
  const expiresIn = account.expiresOn - Date.now();

  if (expiresIn < 300000) { // 5 minutes
    return refreshToken(accountId); // Silent refresh via MSAL
  }

  return account.accessToken;
};
```

**Security Considerations:**
- Tokens stored in LocalStorage (acceptable for localhost-only app)
- No client secret (public client with PKCE)
- HTTPS not required for localhost (OAuth2 exception)
- Azure OpenAI key exposed in browser (unavoidable without backend)

---

## State Management Patterns

### Context Provider Pattern

**Structure:**
```typescript
// 1. Define state interface
interface CalendarState {
  calendars: Calendar[];
  events: CalendarEvent[];
  loading: boolean;
  error: string | null;
}

// 2. Define context type (state + operations)
interface CalendarContextType extends CalendarState {
  syncCalendars: () => Promise<void>;
  createEvent: (input: CreateEventInput) => Promise<void>;
  // ... other operations
}

// 3. Create context with undefined default
const CalendarContext = createContext<CalendarContextType | undefined>(undefined);

// 4. Provider component with useState
export const CalendarProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const [calendars, setCalendars] = useState<Calendar[]>([]);
  const [events, setEvents] = useState<CalendarEvent[]>([]);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  // Operations with useCallback to prevent re-renders
  const syncCalendars = useCallback(async () => {
    // ... implementation
  }, [dependencies]);

  return (
    <CalendarContext.Provider value={{ calendars, events, loading, error, syncCalendars, ... }}>
      {children}
    </CalendarContext.Provider>
  );
};

// 5. Custom hook for consuming context
export const useCalendar = () => {
  const context = useContext(CalendarContext);
  if (!context) {
    throw new Error('useCalendar must be used within CalendarProvider');
  }
  return context;
};
```

### Auto-Sync Pattern

**Calendar Context Auto-Sync:**
```typescript
// Sync on mount and every 5 minutes
useEffect(() => {
  syncCalendars(); // Initial sync

  const interval = setInterval(() => {
    syncCalendars(); // Periodic sync
  }, appConfig.calendar.syncIntervalMs); // 300000ms = 5 minutes

  return () => clearInterval(interval);
}, [syncCalendars]);
```

### Cross-Context Dependencies

**Example: Calendar needs Auth**
```typescript
// CalendarProvider depends on AuthContext
export const CalendarProvider: React.FC<{ children: ReactNode }> = ({ children }) => {
  const { accounts, getAccessToken } = useAuth(); // Access auth state

  const syncCalendars = useCallback(async () => {
    const allCalendars: Calendar[] = [];

    // Fetch calendars for all authenticated accounts
    for (const account of accounts) {
      const accessToken = await getAccessToken(account.homeAccountId);
      const accountCalendars = await calendarService.getCalendars(accessToken, account.homeAccountId);
      allCalendars.push(...accountCalendars);
    }

    setCalendars(allCalendars);
  }, [accounts, getAccessToken]);

  // ...
};
```

---

## Service Layer Architecture

### Generic Graph API Client

**`graph.service.ts` - Base client for all Graph API calls**

```typescript
class GraphService {
  private readonly baseUrl = 'https://graph.microsoft.com/v1.0';

  // Generic request method
  private async request<T>(
    endpoint: string,
    method: string,
    accessToken: string,
    body?: any
  ): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      method,
      headers: {
        'Authorization': `Bearer ${accessToken}`,
        'Content-Type': 'application/json',
      },
      body: body ? JSON.stringify(body) : undefined,
    });

    if (!response.ok) {
      throw new Error(`Graph API error: ${response.statusText}`);
    }

    return response.json();
  }

  // Convenience methods
  async get<T>(endpoint: string, accessToken: string): Promise<T> {
    return this.request<T>(endpoint, 'GET', accessToken);
  }

  async post<T>(endpoint: string, accessToken: string, body: any): Promise<T> {
    return this.request<T>(endpoint, 'POST', accessToken, body);
  }

  // ... patch, delete
}
```

### Specialized Services

**Pattern: Service methods take `accessToken` and return typed data**

```typescript
// calendar.service.ts
class CalendarService {
  async getCalendars(accessToken: string, accountId: string): Promise<Calendar[]> {
    const response: any = await graphService.get('/me/calendars', accessToken);
    return response.value.map(cal => this.mapGraphCalendar(cal, accountId));
  }

  async createEvent(accessToken: string, input: CreateEventInput): Promise<CalendarEvent> {
    const body = {
      subject: input.subject,
      start: { dateTime: input.start, timeZone: 'UTC' },
      end: { dateTime: input.end, timeZone: 'UTC' },
      // ... map input to Graph API format
    };

    const response: any = await graphService.post(
      `/me/calendars/${input.calendarId}/events`,
      accessToken,
      body
    );

    return this.mapGraphEventToCalendarEvent(response, input.calendarId, input.accountId);
  }

  // Private mapper methods
  private mapGraphCalendar(graphCal: any, accountId: string): Calendar {
    return {
      id: graphCal.id,
      accountId,
      name: graphCal.name,
      color: this.generateColorFromId(graphCal.id),
      isDefault: graphCal.isDefaultCalendar,
    };
  }
}
```

### Azure OpenAI Integration

**`openai.service.ts` - Recipe generation**

```typescript
class OpenAIService {
  async generateRecipes(ingredients: string[]): Promise<Recipe[]> {
    const prompt = `You are a helpful cooking assistant. Based on the following ingredients, suggest 3 creative and practical recipes...
Format your response as valid JSON array with this structure:
[{ "title": "Recipe Name", "description": "...", "ingredients": [...], "instructions": [...], "prepTime": "...", "cookTime": "...", "servings": 4 }]

Ingredients: ${ingredients.join(', ')}`;

    const response = await fetch(
      `${appConfig.azureOpenAI.endpoint}/openai/deployments/${appConfig.azureOpenAI.deployment}/chat/completions?api-version=2024-02-15-preview`,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
          'api-key': appConfig.azureOpenAI.apiKey,
        },
        body: JSON.stringify({
          messages: [
            { role: 'system', content: 'You are a helpful cooking assistant.' },
            { role: 'user', content: prompt },
          ],
          max_tokens: 2000,
          temperature: 0.7,
        }),
      }
    );

    const data = await response.json();
    const content = data.choices[0].message.content;

    // Extract JSON from markdown code blocks if present
    const jsonMatch = content.match(/```json\n([\s\S]*?)\n```/) || content.match(/\[[\s\S]*\]/);
    const jsonStr = jsonMatch ? jsonMatch[1] || jsonMatch[0] : content;

    return JSON.parse(jsonStr);
  }
}
```

---

## Component Design Patterns

### Functional Components Only

**No class components - use hooks for all logic**

```typescript
// Pattern: FC with explicit props interface
interface WeekViewProps {
  onCreateEvent: (date: Date, hour: number) => void;
}

export const WeekView: React.FC<WeekViewProps> = ({ onCreateEvent }) => {
  // State hooks
  const [currentWeek, setCurrentWeek] = useState(new Date());

  // Context hooks
  const { events, calendars } = useCalendar();

  // Memoized values
  const weekDays = useMemo(() => dateHelpers.getWeekDays(currentWeek), [currentWeek]);
  const weekEvents = useMemo(() =>
    events.filter(event => /* ... */),
    [events, currentWeek]
  );

  // Callbacks
  const handleSlotClick = useCallback((day: Date, hour: number) => {
    onCreateEvent(day, hour);
  }, [onCreateEvent]);

  return (
    <div className="week-view">
      {/* JSX */}
    </div>
  );
};
```

### Modal Pattern

**Controlled modal with isOpen prop**

```typescript
interface CreateEventModalProps {
  isOpen: boolean;
  onClose: () => void;
  initialDate?: Date;
  initialHour?: number;
}

export const CreateEventModal: React.FC<CreateEventModalProps> = ({
  isOpen,
  onClose,
  initialDate,
  initialHour = 9,
}) => {
  const [formData, setFormData] = useState<CreateEventInput>({
    subject: '',
    start: initialDate ? dateHelpers.setHour(initialDate, initialHour) : new Date(),
    // ...
  });

  const { createEvent } = useCalendar();

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    await createEvent(formData);
    onClose();
  };

  if (!isOpen) return null;

  return (
    <div className="modal-backdrop" onClick={onClose}>
      <div className="modal-content" onClick={e => e.stopPropagation()}>
        <form onSubmit={handleSubmit}>
          {/* Form fields */}
        </form>
      </div>
    </div>
  );
};
```

### FAB Menu Pattern

**Floating Action Button with expandable menu**

```typescript
export const FABMenu: React.FC = () => {
  const [isOpen, setIsOpen] = useState(false);
  const navigate = useNavigate();

  const menuItems: MenuItem[] = [
    { path: '/calendar', icon: <Calendar />, label: 'Calendar', color: 'bg-primary-600' },
    { path: '/photos', icon: <Image />, label: 'Photos', color: 'bg-purple-600' },
    // ...
  ];

  return (
    <>
      {/* Backdrop */}
      {isOpen && (
        <div className="fixed inset-0 bg-black/30 z-40" onClick={() => setIsOpen(false)} />
      )}

      {/* Menu items (appear above backdrop) */}
      <div className="fixed bottom-24 right-6 z-50 flex flex-col-reverse gap-3">
        {isOpen && menuItems.map((item, index) => (
          <button
            key={item.path}
            onClick={() => { navigate(item.path); setIsOpen(false); }}
            className={`${item.color} text-white rounded-full p-4 shadow-lg`}
            style={{ animation: `fadeIn 0.2s ease-out ${index * 0.05}s backwards` }}
          >
            {item.icon}
            <span className="ml-3">{item.label}</span>
          </button>
        ))}
      </div>

      {/* Main FAB button */}
      <button
        onClick={() => setIsOpen(!isOpen)}
        className="fixed bottom-6 right-6 z-50 bg-primary-600 text-white rounded-full p-5 shadow-xl"
      >
        {isOpen ? <X size={28} /> : <Menu size={28} />}
      </button>
    </>
  );
};
```

---

## Styling Approach

### Tailwind CSS Utility-First

**Custom Theme (`tailwind.config.js`):**
```javascript
module.exports = {
  theme: {
    extend: {
      colors: {
        primary: { 500: '#0ea5e9', 600: '#0284c7', 700: '#0369a1' },
        secondary: { 500: '#a855f7', 600: '#9333ea', 700: '#7e22ce' },
        accent: { 500: '#fbbf24', 600: '#f59e0b', 700: '#d97706' },
      },
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        display: ['Poppins', 'sans-serif'],
      },
      boxShadow: {
        soft: '0 2px 15px rgba(0,0,0,0.1)',
        card: '0 4px 20px rgba(0,0,0,0.08)',
      },
    },
  },
};
```

### Custom Component Classes (`index.css`)

```css
@layer components {
  .btn-primary {
    @apply bg-primary-600 hover:bg-primary-700 text-white font-medium py-2 px-4 rounded-lg
           transition-colors duration-200 shadow-sm hover:shadow-md active:scale-95
           disabled:opacity-50 disabled:cursor-not-allowed;
  }

  .card {
    @apply bg-white rounded-xl shadow-card p-4 transition-shadow duration-200 hover:shadow-lg;
  }

  .touch-target {
    min-width: 44px;
    min-height: 44px;
    @apply flex items-center justify-center;
  }
}
```

### Responsive Design

**Mobile-first approach with tablet breakpoints:**
```tsx
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
  {/* Responsive grid */}
</div>
```

**Common breakpoints:**
- `sm:` - 640px (small tablets)
- `md:` - 768px (tablets)
- `lg:` - 1024px (large tablets/laptops)

---

## Common Development Tasks

### Adding a New Feature

1. **Define types** in `src/types/`
2. **Create service** in `src/services/` if API integration needed
3. **Create context** in `src/contexts/` for state management
4. **Build components** in `src/components/[feature]/`
5. **Create page** in `src/pages/`
6. **Add route** in `src/App.tsx`
7. **Add to FAB menu** in `src/components/layout/FABMenu.tsx`

### Adding a New Microsoft Graph API Endpoint

```typescript
// 1. Add to graph.service.ts
async getNewResource(accessToken: string): Promise<any> {
  return this.get('/me/newresource', accessToken);
}

// 2. Create specialized service
class NewResourceService {
  async fetchResources(accessToken: string, accountId: string): Promise<Resource[]> {
    const response = await graphService.getNewResource(accessToken);
    return response.value.map(item => this.mapToResource(item, accountId));
  }
}

// 3. Add to context
export const NewResourceProvider: React.FC = ({ children }) => {
  const { accounts, getAccessToken } = useAuth();
  const [resources, setResources] = useState<Resource[]>([]);

  const syncResources = useCallback(async () => {
    const all: Resource[] = [];
    for (const account of accounts) {
      const token = await getAccessToken(account.homeAccountId);
      const items = await newResourceService.fetchResources(token, account.homeAccountId);
      all.push(...items);
    }
    setResources(all);
  }, [accounts, getAccessToken]);

  return (
    <NewResourceContext.Provider value={{ resources, syncResources }}>
      {children}
    </NewResourceContext.Provider>
  );
};
```

### Debugging Authentication Issues

**Check token validity:**
```typescript
// In browser console
const accounts = JSON.parse(localStorage.getItem('planner_auth_accounts') || '[]');
console.log(accounts);

// Check expiry
accounts.forEach(acc => {
  const expiresIn = acc.expiresOn - Date.now();
  console.log(`${acc.email}: expires in ${Math.floor(expiresIn / 60000)} minutes`);
});
```

**Test token manually:**
```bash
curl -H "Authorization: Bearer YOUR_TOKEN" https://graph.microsoft.com/v1.0/me
```

### Adding a New Calendar View

1. Create component: `src/components/calendar/NewView.tsx`
2. Add to CalendarPage view switcher
3. Implement event rendering logic
4. Use `useCalendar()` hook for data

```typescript
export const CustomView: React.FC<CustomViewProps> = ({ onCreateEvent }) => {
  const { events, calendars, selectedCalendars } = useCalendar();

  const filteredEvents = useMemo(() =>
    events.filter(event => selectedCalendars.includes(event.calendarId)),
    [events, selectedCalendars]
  );

  return (
    <div className="custom-view">
      {/* Custom rendering logic */}
    </div>
  );
};
```

---

## Troubleshooting

### Port Already in Use

**Problem:** `Port 5173 is already in use`

**Solutions:**
```bash
# Option 1: Kill process on port 5173
lsof -ti:5173 | xargs kill -9

# Option 2: Find and kill manually
lsof -i:5173  # Find PID
kill -9 <PID>

# Option 3: Change port (requires updating Azure AD redirect URI)
# Edit vite.config.ts, change port to 3000 or other
```

### Build Errors

**Problem:** `Module not found` or build failures

**Solutions:**
```bash
# Clear node_modules and reinstall
rm -rf node_modules package-lock.json
npm install

# Clear Vite cache
rm -rf node_modules/.vite
npm run dev
```

### Tailwind Classes Not Working

**Problem:** Tailwind classes not applying or build errors

**Checklist:**
- Verify `tailwind.config.js` content paths include all source files
- Ensure `@tailwind` directives present in `index.css`
- Check for typos in class names
- Use Tailwind 3.4 (not v4 - PostCSS compatibility issue)

```javascript
// tailwind.config.js
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",  // Important: include all source files
  ],
  // ...
};
```

### Authentication Failures

**Problem:** "Failed to sign in" or "Authentication initialization failed"

**Checklist:**
1. Verify Azure AD client ID in `.env` is correct
2. Check redirect URI matches in both:
   - `.env` → `VITE_MICROSOFT_REDIRECT_URI`
   - Azure AD app registration → Redirect URIs
3. Ensure all required API permissions granted and consented
4. Clear LocalStorage: `localStorage.clear()` in browser console
5. Check browser console for detailed MSAL errors

### Microsoft Graph API Errors

**401 Unauthorized:**
- Token expired → Sign out and back in
- Insufficient permissions → Check Azure AD app permissions

**403 Forbidden:**
- Missing consent → Grant admin consent in Azure AD
- Scope not included in token → Verify scope in `app.config.ts`

**429 Too Many Requests:**
- Rate limited → Wait and implement exponential backoff

**404 Not Found:**
- Resource doesn't exist or wrong endpoint
- Check accountId/calendarId is valid

### Azure OpenAI Errors

**Problem:** Recipe generation failing

**Checklist:**
1. Verify endpoint, key, and deployment name in `.env`
2. Check Azure OpenAI resource is deployed and active
3. Ensure deployment model supports chat completions (GPT-4, GPT-3.5-turbo)
4. Check Azure OpenAI quota and limits
5. Test endpoint with curl:
```bash
curl -X POST "YOUR_ENDPOINT/openai/deployments/YOUR_DEPLOYMENT/chat/completions?api-version=2024-02-15-preview" \
  -H "Content-Type: application/json" \
  -H "api-key: YOUR_KEY" \
  -d '{"messages":[{"role":"user","content":"Hello"}]}'
```

### Wake Lock Not Working

**Problem:** Screen still sleeps on tablet

**Possible causes:**
- Browser doesn't support Wake Lock API (check compatibility)
- Page not in foreground (wake lock only works when visible)
- User hasn't interacted with page yet (security restriction)

**Check support:**
```javascript
// In browser console
console.log('Wake Lock API:', 'wakeLock' in navigator);
```

---

## Testing Checklist

### Manual Testing

**Authentication:**
- [ ] Sign in with first account
- [ ] Sign in with second account
- [ ] View both accounts in settings
- [ ] Sign out one account
- [ ] Token refresh after 55+ minutes

**Calendar:**
- [ ] View week/day/month views
- [ ] Create event in different calendars
- [ ] Events display with correct colors
- [ ] Calendar filter toggles work
- [ ] Sync button refreshes data
- [ ] Events from both accounts visible

**Photos:**
- [ ] Browse OneDrive folder tree
- [ ] Select folder with images
- [ ] Slideshow auto-advances
- [ ] Pause/play controls work
- [ ] Navigate previous/next
- [ ] Fullscreen mode

**Meals:**
- [ ] Add ingredients to fridge
- [ ] Remove ingredients
- [ ] Generate recipes with AI
- [ ] Favorite recipes
- [ ] View saved recipes

**Tasks:**
- [ ] View To Do lists from both accounts
- [ ] Create new task
- [ ] Toggle task completion
- [ ] Delete task
- [ ] Filter by active/completed

**Tablet:**
- [ ] All touch targets minimum 44px
- [ ] FAB menu accessible with thumb
- [ ] Wake lock prevents sleep
- [ ] Responsive on tablet screen size
- [ ] Network access on actual device

### Edge Cases

- [ ] No internet connection (offline mode)
- [ ] Token refresh during active request
- [ ] Very long event titles
- [ ] All-day events
- [ ] Events spanning multiple days
- [ ] Multiple accounts with duplicate calendar names
- [ ] Large photo folders (1000+ images)
- [ ] Empty fridge inventory (recipe generation)

---

## Performance Optimization

### Best Practices

**Memoization:**
```typescript
// Expensive calculations
const weekDays = useMemo(() => dateHelpers.getWeekDays(currentWeek), [currentWeek]);

// Callbacks to prevent re-renders
const handleClick = useCallback(() => { /* ... */ }, [dependencies]);
```

**Lazy Loading:**
```typescript
// Future enhancement: lazy load calendar views
const WeekView = lazy(() => import('./components/calendar/WeekView'));
```

**Debouncing:**
```typescript
// Debounce search/filter inputs
const debouncedSearch = useMemo(
  () => debounce((query: string) => { /* search logic */ }, 300),
  []
);
```

**Pagination:**
- Limit initial calendar event fetch to visible date range
- Load older/future events on demand
- Paginate OneDrive folder contents (50 items at a time)

---

## Environment Variables

**Required Configuration (`.env`):**

```bash
# Microsoft OAuth (Azure AD App Registration)
VITE_MICROSOFT_CLIENT_ID=your_azure_ad_client_id_here
VITE_MICROSOFT_REDIRECT_URI=http://localhost:5173/auth/callback

# Azure OpenAI (for meal planning AI)
VITE_AZURE_OPENAI_ENDPOINT=https://your-resource.openai.azure.com
VITE_AZURE_OPENAI_KEY=your_api_key_here
VITE_AZURE_OPENAI_DEPLOYMENT=your_deployment_name_here
```

**Setup Steps:**
1. Copy `.env.example` to `.env`
2. Fill in your Azure AD client ID
3. Fill in your Azure OpenAI credentials
4. Restart dev server (`npm run dev`)

---

## Future Enhancements

**Potential features (not yet implemented):**

- [ ] Push notifications for upcoming events
- [ ] Grocery list generation from recipes
- [ ] Recipe search/database integration
- [ ] Calendar event sharing links
- [ ] Export meal plan to PDF
- [ ] Voice commands via Web Speech API
- [ ] Smart home integration (lights, displays)
- [ ] Recurring event support (Graph API supports this)
- [ ] Event reminders
- [ ] Dark mode theme toggle
- [ ] Gesture-based navigation (swipe between views)
- [ ] Offline queue for write operations
- [ ] Service worker for true PWA offline support

---

## Key Files Reference

**Must-read files for understanding the codebase:**

1. `src/config/app.config.ts` - All configuration constants
2. `src/contexts/AuthContext.tsx` - Multi-account authentication
3. `src/services/graph.service.ts` - Microsoft Graph API client
4. `src/components/layout/FABMenu.tsx` - Navigation pattern
5. `src/App.tsx` - Routing and context providers
6. `vite.config.ts` - Build configuration
7. `tailwind.config.js` - Theme configuration

---

## AI Agent Development Best Practices

### Critical Git Rules

1. **NEVER `git push` without explicit user approval.** Always commit locally and inform the user. Only push when the user explicitly says to push.
2. **ALWAYS run `npm run build` before committing.** This runs `tsc -b && vite build` and catches TypeScript errors that `tsc --noEmit` may miss. Do not commit if the build fails.
3. **ALWAYS stop the dev server before building, then restart it after.** Stop the running `npm run dev` process, run `npm run build`, then start `npm run dev` again. This ensures a clean build and a fresh dev server.
4. **ALWAYS clear the Vite cache before starting the dev server.** Run `rm -rf node_modules/.vite` before `npm run dev` to avoid stale HMR cache errors (e.g., missing exports).

### Workflow for Implementing UI Features

When working on this project as an AI agent, follow this standard workflow:

#### 1. **Always Start the Dev Server**
```bash
npm run dev
```
The app will be available at http://localhost:5173 (fixed port).

#### 2. **Use Playwright MCP to Visually Verify**

After making UI changes, immediately verify them visually:

```typescript
// Navigate to the page
await browser_navigate({ url: "http://localhost:5173" });

// Take a screenshot to see current state
await browser_take_screenshot({ filename: "before-change.png" });

// Make your code changes...

// Refresh and verify
await browser_navigate({ url: "http://localhost:5173" });
await browser_take_screenshot({ filename: "after-change.png" });

// Get accessibility snapshot to verify structure
await browser_snapshot();
```

#### 3. **Verify Touch Targets**

For any new buttons or interactive elements:

```typescript
await browser_evaluate({
  function: `() => {
    const newButton = document.querySelector('[data-testid="new-button"]');
    const rect = newButton.getBoundingClientRect();
    return {
      width: rect.width,
      height: rect.height,
      meetsTouchRequirement: rect.width >= 44 && rect.height >= 44
    };
  }`
});
```

#### 4. **Test User Flows**

Don't just verify static UI - test interactions:

```typescript
// Example: Testing FAB menu interaction
await browser_navigate({ url: "http://localhost:5173" });
await browser_take_screenshot({ filename: "1-homepage.png" });

// Click FAB button
await browser_click({ element: "FAB menu button", ref: "..." });
await browser_take_screenshot({ filename: "2-menu-open.png" });

// Click calendar option
await browser_click({ element: "Calendar menu item", ref: "..." });
await browser_take_screenshot({ filename: "3-calendar-page.png" });

// Verify calendar loaded
await browser_snapshot();
```

#### 5. **Check Responsive Behavior**

Test at tablet viewport size:

```typescript
// Resize to Surface Pro dimensions
await browser_resize({ width: 1280, height: 800 });
await browser_take_screenshot({ filename: "tablet-view.png" });

// Check desktop view
await browser_resize({ width: 1920, height: 1080 });
await browser_take_screenshot({ filename: "desktop-view.png" });
```

### Common MCP Commands for This Project

**Navigation:**
```typescript
browser_navigate({ url: "http://localhost:5173" })
browser_navigate({ url: "http://localhost:5173/calendar" })
browser_navigate({ url: "http://localhost:5173/settings" })
```

**Screenshots:**
```typescript
// Full page
browser_take_screenshot({ filename: "full-page.png", fullPage: true })

// Viewport only
browser_take_screenshot({ filename: "viewport.png" })

// Specific element
browser_take_screenshot({
  filename: "fab-menu.png",
  element: "FAB menu",
  ref: "..."
})
```

**Accessibility Snapshots:**
```typescript
// Get full page structure
browser_snapshot()

// Save to file for reference
browser_snapshot({ filename: "page-structure.md" })
```

**Interactions:**
```typescript
// Click
browser_click({ element: "Button name", ref: "[data-testid='button-id']" })

// Type text
browser_type({
  element: "Input field",
  ref: "[name='email']",
  text: "test@example.com"
})

// Evaluate JavaScript
browser_evaluate({
  function: `() => { return document.title; }`
})
```

### When to Use Visual Verification

**ALWAYS verify visually when:**
- Creating new components
- Modifying layouts or styles
- Adding new pages or routes
- Changing responsive breakpoints
- Updating the FAB menu
- Modifying calendar views
- Working on modals or overlays
- Adjusting touch target sizes
- Implementing new color schemes
- Debugging CSS issues

**Quick verification is acceptable for:**
- Small text changes
- Code refactoring without UI changes
- Adding types or interfaces
- Updating service logic
- Fixing TypeScript errors

### Integration with Development Workflow

**Standard Pattern:**

1. Read relevant files to understand current implementation
2. Make code changes
3. **Start dev server** (`npm run dev`)
4. **Navigate to page** with Playwright MCP
5. **Take screenshots** before and after
6. **Verify touch targets** if adding interactive elements
7. **Test user flows** for new features
8. Document changes

**Example: Adding a New Button**

```bash
# 1. Make code changes
# Edit src/components/example/NewFeature.tsx

# 2. Verify visually
npm run dev

# 3. Use MCP to verify
# - Navigate to http://localhost:5173
# - Take screenshot
# - Click the new button
# - Verify it works
# - Check button size >= 44x44px
```

### Tips for Effective Visual Verification

1. **Take screenshots at key steps** - Don't just test the final state, capture the journey
2. **Name screenshots descriptively** - `fab-menu-open.png` not `screenshot1.png`
3. **Use accessibility snapshots** - They provide structured data that's easier to verify than pixels
4. **Test multiple viewports** - Desktop, tablet, and mobile sizes
5. **Verify error states** - Not just happy paths
6. **Check loading states** - Ensure spinners and skeletons display correctly
7. **Test with real data** - Don't just test with empty states

### Debugging UI Issues with MCP

When a UI issue is reported:

```typescript
// 1. Navigate to the problematic page
await browser_navigate({ url: "http://localhost:5173/problem-page" });

// 2. Take a screenshot to see the issue
await browser_take_screenshot({ filename: "issue-screenshot.png" });

// 3. Get the accessibility snapshot
await browser_snapshot({ filename: "page-structure.md" });

// 4. Inspect specific elements
await browser_evaluate({
  function: `() => {
    const element = document.querySelector('.problematic-element');
    const styles = window.getComputedStyle(element);
    return {
      display: styles.display,
      position: styles.position,
      width: styles.width,
      height: styles.height,
      // ... other relevant styles
    };
  }`
});

// 5. Check console for errors
await browser_console_messages({ level: "error" });
```

### Best Practices Summary

✅ **DO:**
- Start dev server before implementing UI changes
- Use Playwright MCP to verify all UI changes visually
- Take screenshots at multiple steps of user flows
- Verify touch target sizes programmatically
- Test at multiple viewport sizes
- Check accessibility snapshots for semantic correctness
- Document visual verification steps in commit messages

❌ **DON'T:**
- Make UI changes without visual verification
- Assume UI looks correct based on code alone
- Skip testing on tablet viewport
- Forget to verify touch target sizes
- Only test happy paths - test error states too
- Leave debug screenshots in the codebase

---

## Reporting Standards

### Report Directory Structure

All reports, summaries, and analysis documents MUST be stored in the `.reports/` directory with a standardized naming convention.

### Naming Convention

**Format:** `YYYY-MM-DD_{REPORT_NAME}.md`

**Rules:**
- Always use ISO date format (YYYY-MM-DD)
- Use underscores for spaces in report name
- Use UPPERCASE_WITH_UNDERSCORES for report name
- Always use `.md` extension (Markdown)

**Examples:**
```
.reports/2026-01-05_UI_TEST_RESULTS.md
.reports/2026-01-10_PERFORMANCE_ANALYSIS.md
.reports/2026-01-15_SECURITY_AUDIT.md
.reports/2026-02-01_FEATURE_IMPLEMENTATION_SUMMARY.md
.reports/2026-03-15_BUG_INVESTIGATION_REPORT.md
```

### When to Create Reports

Create a report in `.reports/` when:

✅ **Testing & Validation**
- Completing comprehensive UI testing
- Running performance benchmarks
- Conducting security audits
- Performing accessibility validation
- Executing integration tests

✅ **Implementation & Development**
- Finishing significant features
- Completing major refactors
- Making architectural decisions
- Migrating to new technologies
- Implementing breaking changes

✅ **Analysis & Audits**
- Code quality analysis
- Dependency audits
- Bundle size analysis
- Database optimization
- API performance review

✅ **Incidents & Issues**
- Investigating complex bugs
- Post-mortem analysis
- Root cause investigations
- Critical incident reports

✅ **AI Agent Work Sessions**
- Major feature implementations by AI
- Significant debugging sessions
- Architecture design sessions
- Large-scale refactoring work

### Report Structure Template

```markdown
# [Report Title]

## Date
YYYY-MM-DD

## Summary
Brief 2-3 sentence overview of the report.

## Details

### Section 1: [Topic]
Detailed information about the first major topic...

### Section 2: [Topic]
More detailed information...

## Findings
- ✅ Key finding 1
- ✅ Key finding 2
- ⚠️  Warning/concern 1
- ❌ Issue found 1

## Action Items
- [ ] Action 1 - Description
- [ ] Action 2 - Description
- [ ] Action 3 - Description

## Recommendations
1. Recommendation 1 with rationale
2. Recommendation 2 with rationale
3. Recommendation 3 with rationale

## Conclusion
Final assessment, overall status, and next steps.

---

**Reported By:** [Name/AI Agent/Tool]
**Report Type:** [Test/Implementation/Analysis/Incident]
**Status:** [Complete/In Progress/Requires Follow-up]
```

### Report Categories

**Test Reports:**
- UI validation results
- Integration test results
- Performance test results
- Accessibility audit results
- Security test results

**Implementation Reports:**
- Feature completion summaries
- Architecture decision records (ADR)
- Migration completion reports
- Refactoring summaries
- Deployment reports

**Analysis Reports:**
- Code quality analysis
- Performance profiling
- Security audit results
- Dependency audit reports
- Bundle size analysis

**Incident Reports:**
- Bug investigation reports
- Post-mortem analysis
- Root cause analysis
- Incident response reports

### Report Best Practices

✅ **DO:**
- Use the standardized naming convention
- Include date in both filename and content
- Write clear, concise summaries
- Document all findings and action items
- Include screenshots/evidence when relevant
- Tag reports with status (Complete/In Progress)
- Reference related files with line numbers
- Keep reports in version control

❌ **DON'T:**
- Create reports outside `.reports/` directory
- Use inconsistent naming conventions
- Skip the date in filename
- Write vague or ambiguous summaries
- Omit action items or next steps
- Delete old reports (they're historical record)
- Leave reports as drafts indefinitely

### Viewing Reports

**List all reports:**
```bash
ls -lh .reports/
```

**Search reports by keyword:**
```bash
grep -r "keyword" .reports/
```

**View recent reports:**
```bash
ls -lt .reports/ | head -10
```

### Report Retention

- ✅ Keep all reports in version control (Git)
- ✅ Reports serve as project history
- ✅ Old reports remain valuable for reference
- ✅ Use reports to track progress over time
- ❌ Do not delete reports unless truly obsolete

### Integration with Development Workflow

**After Major Work:**
1. Complete the implementation/testing
2. Create a report in `.reports/` with proper naming
3. Document findings, issues, and resolutions
4. Include action items for follow-up work
5. Commit the report with descriptive message

**Example Workflow:**
```bash
# After completing UI testing
touch .reports/2026-01-05_UI_TEST_RESULTS.md
# Write the report...
git add .reports/2026-01-05_UI_TEST_RESULTS.md
git commit -m "Add UI test results report for 2026-01-05"
```

---

## Contact & Support

**For issues or questions:**
1. Check this AGENTS.md file first
2. Review README.md for user-facing documentation
3. Check browser console for error messages
4. Verify Azure AD app configuration
5. Test API endpoints with curl/Postman

---

## Version History

- **v0.0.0** (Current) - Initial implementation with all core features
  - Multi-account calendar sync
  - OneDrive photo slideshow
  - AI meal planning
  - Microsoft To Do integration
  - Fixed port configuration (5173)
  - **Playwright MCP server integration for visual verification**
  - **Comprehensive Playwright test suite with automated UI validation**
  - **Standardized reporting system (.reports/ directory with YYYY-MM-DD naming)**
  - Comprehensive documentation with AI agent best practices

---

**Last Updated:** 2026-01-05

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sebastienlevert)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/sebastienlevert)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
