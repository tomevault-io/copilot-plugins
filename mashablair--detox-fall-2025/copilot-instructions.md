## detox-fall-2025

> This is a 30-day Detox Fall 2025 wellness tracking app for Russian-speaking users. It helps users track their daily progress through a detox protocol with meal logging, supplement tracking, and progress visualization.

# Fall 2025 Detox Tracker - Cursor AI Rules

## Project Overview

This is a 30-day Detox Fall 2025 wellness tracking app for Russian-speaking users. It helps users track their daily progress through a detox protocol with meal logging, supplement tracking, and progress visualization.

## Tech Stack

- **Frontend Framework**: Angular 20.3.6 (standalone components)
- **Styling**: Tailwind CSS 4.x with custom cream/brand color palette
- **UI Components**: @tailwindplus/elements for dropdowns, modals, etc.
- **Backend**: Firebase
  - **Authentication**: Firebase Auth (email/password)
  - **Database**: Firestore (region: nam5 - Iowa)
  - **Image Storage**: Cloudinary (not Firebase Storage)
- **Charts**: Chart.js with ng2-charts
- **Icons**: ng-icons with Heroicons (outline style)
- **Language**: Russian UI (Cyrillic text)

## Project Structure

```
src/app/
├── core/
│   ├── config/          # App configuration (icons.config.ts)
│   ├── guards/          # Auth guards (authGuard, publicGuard)
│   ├── models/          # TypeScript interfaces (UserProfile, DailyLog, Product)
│   ├── services/        # Business logic services
│   └── data/            # Static data (products catalog, nutrition data)
├── features/            # Feature modules (auth, home, daily-log, profile, etc.)
├── layout/              # Shell components (sidebar, top-bar, dashboard-shell)
└── app.routes.ts        # Route configuration
```

## Architecture Patterns

### Authentication Flow

1. **Firebase Authentication** handles login/signup
2. **UserService** manages user profiles in Firestore
3. Auth state changes trigger profile load/clear automatically
4. Guards protect routes: `authGuard` for authenticated routes, `publicGuard` for public routes

### User Profile

- **Model**: `UserProfile` interface with firstName, lastName, country, email, startDate, products
- **Storage**: Firestore collection `userProfiles` (document ID = Firebase Auth UID)
- **Cache**: localStorage cache for instant page loads (key: `detox_user_profile`)
- **Service**: `UserService` handles all profile CRUD operations
- **Cache-First Strategy**: 
  1. Load from localStorage immediately (instant display)
  2. Fetch from Firestore in background to verify/update cache
  3. All profile updates save to both Firestore and localStorage
  4. Logout clears both Firestore cache and localStorage
- **Auto-sync**: Profile loads on login with cache-first strategy, clears on logout via callback pattern
- **Products Field**: Array of product IDs user selected in onboarding (e.g., ['vmg-plus', 'eo-mega'])
- **Cross-device sync**: When user logs in on different device, Firestore provides latest data and updates local cache

### Data Services

- Use Angular signals for reactive state (`signal<T>()`)
- Async operations return Promises
- Services are `@Injectable({ providedIn: 'root' })`
- Firestore integration uses `@angular/fire/firestore`
- **StorageService**: Wrapper for localStorage operations (setItem, getItem, removeItem) with error handling
- **Caching Strategy**: User profiles cached in localStorage for instant page loads, with background sync from Firestore

### Key Services

**UserService** (`core/services/user.service.ts`):
- **userProfile signal**: Reactive state for current user's profile
- **loadUserProfileWithCache(userId)**: Private method using cache-first strategy
  - Step 1: Load from localStorage (instant)
  - Step 2: Fetch from Firestore in background (sync)
- **setUserProfile(profile, userId?)**: Save profile to Firestore + localStorage
- **clearUserProfile()**: Clear profile from signal + localStorage (logout)
- **deleteUserProfile()**: Delete from Firestore + localStorage (account deletion)
- **getStartDate()**: Returns user's detox start date
- **Cache Key**: `detox_user_profile`

**StorageService** (`core/services/storage.service.ts`):
- **setItem<T>(key, value)**: Save to localStorage with JSON serialization
- **getItem<T>(key)**: Retrieve from localStorage with JSON parsing
- **removeItem(key)**: Remove from localStorage
- Error handling for all operations (catches localStorage exceptions)

**ProtocolService** (`core/services/protocol.service.ts`):
- Uses computed signals to access user's startDate and products from UserService
- **getDayNumber(date)**: Returns current day 1-30 in program
- **getWeekNumber(date)**: Returns current week 1-4
- **getCurrentPhase(date)**: Returns phase metadata with Russian descriptions
- **getSupplementsByTiming(date)**: Returns supplements grouped by morning/lunch/dinner (filtered by user's products)
- **isCurrentWeekLocked(date)**: Checks if schedule is available

**DailyLogService** (`core/services/daily-log.service.ts`):
- Local storage management for daily logs
- Save and retrieve daily tracking data (habits, symptoms, supplements, notes)

**ImageService** (`core/services/image.service.ts`):
- Cloudinary image URL generation with transformations
- Product images, profile avatars (future), general assets
- Fallback to local assets if Cloudinary not configured

**AuthService** (`core/services/auth.service.ts`):
- Firebase Authentication wrapper
- Login, signup, logout, getCurrentUser
- **onUserAuthStateChanged callback**: Triggers UserService profile load/clear

### Product Catalog System

**Architecture:** Separation of types/interfaces from static data

- **Models** (`core/models/products.model.ts`):
  - `Product` interface - complete product schema
  - Type definitions: `RegionCode`, `ProductType`, `ProgramLevel`
  - `TakeInstruction` interface - structured dosage/timing info
  - `RegionVariant` interface - regional names/substitutes
- **Data** (`core/data/products.data.ts`):
  - `INITIAL_PRODUCTS` array - 8 core catalog products
  - Helper functions: `byRegion()`
  - Products stored as static data (no database needed for core catalog)

### Protocol System (30-Day Detox)

**Architecture:** 4-week program with phase-based supplement schedules

- **Models** (`core/models/protocol.model.ts`):

  - `SupplementTiming` - Type: 'morning' | 'lunch' | 'dinner'
  - `ProtocolSupplement` - Individual supplement entry (productId, timing, amount, notes)
  - `WeekSchedule` - Weekly supplement schedule with locked status
  - `ProtocolPhase` - Phase metadata (week, title, subtitle, description, topics in Russian)

- **Data Files**:

  - `protocol-phases.data.ts` - 4 weeks of phase descriptions in Russian
  - `protocol-schedule.data.ts` - Weekly supplement schedules (Week 1 complete, Weeks 2-4 locked/placeholder)

- **ProtocolService** (`core/services/protocol.service.ts`):
  - `getDayNumber(date)` - Returns current day 1-30
  - `getWeekNumber(date)` - Returns current week 1-4
  - `getCurrentPhase(date)` - Returns ProtocolPhase with Russian metadata
  - `getSupplementsByTiming(date)` - Returns supplements grouped by morning/lunch/dinner
  - `isCurrentWeekLocked(date)` - Checks if schedule is available
  - **Filtering**: Only returns supplements user selected in onboarding

**4-Week Program Structure:**

- **Week 1** (Days 1-7): "Запуск систем детокса и молодости" - Complete schedule
- **Week 2** (Days 8-14): "Пищеварение и митохондрии" - Placeholder (coming soon)
- **Week 3** (Days 15-21): "Лимфа, иммунитет и теломеры" - Placeholder (coming soon)
- **Week 4** (Days 22-30): "Жизнь после детокса" - Placeholder (coming soon)

**Protocol Data Pattern:**

```typescript
{
  week: 1,
  locked: false,
  supplements: [
    {
      productId: 'vmg-plus',
      timing: 'morning',
      amount: '1 порция',
      notes: 'Смешать с водой'
    },
    // ... more supplements
  ]
}
```

### Nutrition Tracking System

**Architecture:** Habit-based scoring with contextual tips

- **Models** (`core/models/nutrition.model.ts`):
  - `HabitDef` - Defines trackable habits (boolean or counter type)
  - `TipRule` - Contextual tips triggered by user behavior
  - `NutritionPrinciple`, `FoodGuideline`, `HydrationGuideline`, `PracticeGuideline` - Reference data
  - `MealLog`, `DayLog`, `NutritionScore` - Future tracking models
- **Data** (`core/data/nutrition.data.ts`):
  - `PRINCIPLES` - Core nutrition principles for detox
  - `FOODS` - Foods to avoid during detox
  - `HYDRATION` - Water and herbal tea guidelines
  - `PRACTICES` - Supportive practices (IF, mindful eating, seasonal foods)
  - `HABITS` - 13 trackable habits (9 daily + 4 weekly) with weights for scoring
  - `TIPS` - 6 contextual tips that trigger based on missing/incomplete habits

**Habits System:**

```typescript
interface HabitDef {
  id: string; // kebab-case habit ID
  title: string; // Russian display title
  period: 'daily' | 'weekly';
  type: 'boolean' | 'counter'; // checkbox or numeric counter
  target?: number; // for counter types (e.g., 5 colors, 8 glasses)
  weight: number; // scoring weight (6-16 points)
}
```

**Daily Habits (9 total, 94 points):**

1. ≥80% натуральных продуктов (10 points)
2. Цветная тарелка: ≥5 цветов (10 points, counter)
3. Клетчатка 25–30 г (12 points)
4. Белок в каждом приёме пищи (12 points)
5. Полезные жиры в рационе (8 points)
6. Гидратация: 8–10 стаканов (10 points, counter)
7. Без сахара и UPF (14 points)
8. 12–14 ч ночное окно (10 points)
9. Осознанное питание (8 points)

**Scoring Algorithm:**

- Boolean habits: 100% weight if checked, 0% if unchecked
- Counter habits: Proportional weight based on progress (value/target \* weight)
- Total score = (sum of earned weights / sum of all weights) \* 100
- Score displayed as 0-100 integer

**Contextual Tips:**

- Tips automatically display based on missing habits
- Two trigger types: `missingHabit` (boolean not checked) or `counterBelow` (counter below threshold)
- Max 3 tips shown at once
- Real-time updates as user checks habits

**Product Schema:**

```typescript
interface Product {
  id: string; // kebab-case slug
  name: string; // English canonical name
  description?: string;
  takes?: TakeInstruction[]; // dosage/timing instructions
  altName?: string;
  level?: ProgramLevel; // base | advanced
  regions?: Partial<Record<RegionCode, RegionVariant>>;
  region?: RegionCode[]; // availability by region
  type: ProductType; // supplement | essential_oil
  imageURL?: string; // filename only (ImageService builds full URL)
  optional?: boolean;
  source?: 'catalog' | 'user'; // catalog = immutable, user = editable
  userId?: string; // for user-created products
}
```

**Product Source Types:**

- `catalog` - Immutable core products from static data, read-only, no database storage
- `user` - Custom user products stored in Firestore (future feature), editable

**Catalog Products (8 total):**

1. VMG+™ (base) - vmg-plus.png
2. EO Mega™ (base) - eo-mega.png
3. PB Restore® (base) - pb-restore.png
4. TerraZyme™ (base) - terrazyme.png
5. GX Assist® (advanced) - gx-assist.png
6. DDR Prime® (advanced) - ddr-prime.png
7. RevitaZen™ Detoxification Blend (advanced) - revitazen-detox-blend.png
8. RevitaZen™ Advanced Organ Support (advanced) - revitazen-advanced-organ.png

## Firestore Structure

```
Firestore Database (nam5 region)
├── userProfiles/
│   └── {userId}/          # Document ID = Firebase Auth UID
│       ├── firstName: string
│       ├── lastName: string
│       ├── country: string
│       ├── email: string
│       ├── startDate: string (ISO 8601: YYYY-MM-DD)
│       └── products: string[] (product IDs: ['vmg-plus', 'eo-mega', 'pb-restore', ...])
└── (future collections for daily logs, etc.)
```

## Firestore Security Rules

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /userProfiles/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

## Styling Conventions

### Color Palette (Tailwind)

- **Primary Brand**: `brand-*` (green/sage tones for CTAs and accents)
- **Background**: `cream-*` (soft neutral palette - 50, 100, 200, etc.)
- **Text**: `cream-900` for dark text, `cream-700` for secondary text
- **Icons**:
  - Primary icons: `cream-900` (soft alternative to black)
  - Secondary icons: `cream-800` (less prominent)
  - Tertiary/disabled: `cream-700` (placeholders, disabled states)
  - Accent icons: `brand-600` or `brand-700` (featured actions)
- **Fonts**:
  - Display/headings: `font-display`
  - Body text: `font-body`

### Component Structure

- Use standalone components (no modules)
- Import `CommonModule`, `FormsModule`, `RouterLink` as needed
- Use new Angular control flow: `@if`, `@for` (not *ngIf, *ngFor)
- Use signals for reactive state: `signal<T>()`, `.set()`, `()`

### Form Patterns

- Template-driven forms with `[(ngModel)]`
- Validation messages with signals: `errorMessage = signal<string>('')`
- Loading states: `isLoading = signal<boolean>(false)`
- Russian placeholder text and labels

## Routing Structure

```
/ (landing page - public, accessible to all)
/login (public with publicGuard)
/signup (public with publicGuard)
/onboarding (authenticated, separate from shell)
/dashboard (authenticated, inside dashboard-shell - uses HomeComponent)
/log-daily (authenticated, inside dashboard-shell)
/progress (authenticated, inside dashboard-shell)
/nutrition-guide (authenticated, inside dashboard-shell)
/profile (authenticated, inside dashboard-shell)
```

**Home Component** (`/dashboard` route):

- Displays personalized welcome with user's first name
- Shows progress through 30-day program with visual progress bar
- Quick stats: days logged, current streak, completion percentage
- Phase indicator with Russian translations
- Quick action cards linking to daily log, progress, and profile
- Daily wellness tip
- All UI in Russian with cream/brand color palette

**Nutrition Guide Component** (`/nutrition-guide` route):

- Reference page with four scrolling cards (no tabs)
- **Principles Card**: Core nutrition principles with checkmark bullets
- **Foods to Avoid Card**: List with red X icons and warning callout
- **Hydration Card**: Water and herbal tea guidelines with tip callout
- **Practices Card**: IF, mindful eating, seasonal foods with tip callout
- All data sourced from `nutrition.data.ts`
- Beautiful icons (heroSparkles, heroXMark, heroBeaker, heroLightBulb, heroCheckCircle)
- Cream/brand color palette with red accents for "avoid" section

**Daily Log Component** (`/log-daily` route - Enhanced):

- **Header**: Shows live nutrition score (0-100) with sparkles icon
- **Nutrition Habits Section**:
  - 7 boolean habit checkboxes with weights displayed
  - 2 counter habits (colors, water) with +/- buttons and progress bars
  - Visual feedback: counters show value/target and progress bar
- **Contextual Tips Card**: Max 3 tips displayed based on missing habits (auto-updates)
- **Symptoms Section**: 5 sliders for digestion, bloating, energy, sleep, puffiness
- **Supplements Section**: Protocol-based, grouped by timing
  - **Morning** (☀️): Supplements to take with breakfast
  - **Lunch** (🔥): Supplements to take with lunch
  - **Dinner** (🌙): Supplements to take with dinner
  - Each supplement shows: Product image, name, dosage, notes
  - Only displays products user selected in onboarding
  - Empty state for locked weeks
- **Notes Section**: Free-form text area
- Real-time score calculation as user interacts with habits
- All form data saved to DailyLogService (local storage for now)

## Navigation Components

- **Sidebar**: Logo (links to /), navigation items with visual separator
  - Main section: Dashboard, Daily Log, Progress
  - Separator: Horizontal line with `border-cream-500`
  - Reference section: Nutrition Guide (with heroBookOpen icon)
  - Profile link in top-bar dropdown
- **Top-bar**: User dropdown with profile link and logout
- **Logo behavior**: Clicking logo navigates to landing page (allowed for authenticated users)
- **Active state**: Cream-200 background, brand-700 text for current route

## User Interface Language

- **ALL UI text in Russian (Cyrillic)**
- Form labels, buttons, error messages, navigation - everything in Russian
- Use professional, friendly tone appropriate for wellness app

## Firebase Configuration

- Project ID: `detox-fall-2025`
- Environment config in: `src/environments/environment.ts`
- Firestore region: `nam5` (us-central1 - Iowa)
- Auth provider: Email/Password (enabled)

## Icons System (ng-icons)

**Configuration:** Centralized icon management using ng-icons library

**Setup:**

- **Package**: `@ng-icons/core` + `@ng-icons/heroicons`
- **Icon Set**: Heroicons (outline style only)
- **Config File**: `core/config/icons.config.ts`
- **Registration**: Icons registered globally in `app.config.ts` via `provideIcons(appIcons)`

**Icon Configuration File** (`core/config/icons.config.ts`):

```typescript
import {
  heroHome,
  heroCalendar,
  // ... other icons
} from '@ng-icons/heroicons/outline';

export const appIcons = {
  // Organized by category
  heroHome,
  heroCalendar,
  // ... all app icons
};
```

**Usage in Components:**

1. **Import NgIcon component:**

```typescript
import { NgIcon } from '@ng-icons/core';

@Component({
  imports: [CommonModule, NgIcon, ...],
})
```

2. **Use in template:**

```html
<!-- Static color with CSS variable -->
<ng-icon name="heroHome" size="24" style="color: var(--color-brand-600)" />

<!-- Static color with RGB -->
<ng-icon name="heroFire" size="24" style="color: rgb(234 88 12)" />

<!-- Dynamic color with currentColor (inherits from parent) -->
<span class="text-cream-600 hover:text-brand-600">
  <ng-icon name="heroUser" size="24" style="color: currentColor" />
</span>
```

**Color Styling Best Practices:**

- **IMPORTANT**: Always use inline `style="color: ..."` for icon colors
- Tailwind color classes (like `class="text-brand-600"`) don't work reliably on ng-icon
- Use `style="color: currentColor"` to inherit color from parent element (for hover states)
- Use `style="color: var(--color-brand-600)"` for CSS custom properties
- Use `style="color: rgb(234 88 12)"` for direct RGB values

**Adding New Icons:**

1. Import icon in `core/config/icons.config.ts`
2. Add to `appIcons` object (organized by category)
3. Use in components with `<ng-icon name="iconName" />`

**Icon Categories:**

- Stats & Progress (heroCheckCircle, heroFire, heroArrowTrendingUp)
- Actions (heroPlus, heroChevronRight, heroChevronDown)
- Navigation (heroHome, heroCalendar, heroChartPie, heroChartBar, heroUser, heroBookOpen)
- UI Elements (heroLightBulb, heroBell, heroCog6Tooth, heroXMark, heroSparkles, heroBeaker, heroSun, heroMoon, heroInformationCircle)

## Cloudinary Configuration

**Service:** Image delivery and transformations (product images, future user uploads)

**Configuration:**

- Cloud Name: `djnvzdffx`
- Base Folder: `detox-fall`
- Products Folder: `detox-fall/products/`
- Environment files: `src/environments/environment.ts` and `environment.prod.ts`

**ImageService** (`core/services/image.service.ts`):

- Central service for all image URL generation
- Handles Cloudinary transformations (width, height, quality, format)
- Supports local assets fallback if Cloudinary not configured
- Methods:
  - `getProductImage(filename, options?)` - Product images with transformations
  - `getProfileImage(filename, size?)` - User avatars (future)
  - `getAssetImage(filename)` - General assets
  - `getPlaceholder(type)` - Placeholder images

**Image URL Pattern:**

```
https://res.cloudinary.com/{cloudName}/image/upload/{transformations}/{folder}/products/{filename}
```

**Transformations:**

```typescript
imageService.getProductImage('vmg-plus.png', {
  width: 300, // resize width
  height: 300, // resize height
  quality: 80, // compression quality
  format: 'webp', // force format conversion
});
```

**Current Format:** PNG (all 8 product images uploaded as PNG)
**Future Optimization:** Can use `f_auto` for automatic WebP/PNG delivery based on browser

**Folder Structure in Cloudinary:**

```
detox-fall/
├── products/         # 8 catalog product images (PNG format)
├── profiles/         # Future: user avatars
└── assets/           # Future: misc images
```

**Important Notes:**

- Product images stored in Cloudinary (not Firebase Storage)
- ImageService automatically falls back to `/assets/` if Cloudinary not configured
- User-uploaded images (future) will use full URLs, service passes them through
- Always use ImageService, never hardcode Cloudinary URLs

## Chart.js Configuration

**Important:** Chart.js v3+ requires explicit component registration

```typescript
import {
  Chart,
  LineController,
  LineElement,
  PointElement,
  LinearScale,
  CategoryScale,
  Title,
  Tooltip,
  Legend,
  Filler,
} from 'chart.js';

// Register all Chart.js components before using
Chart.register(
  LineController,
  LineElement,
  PointElement,
  LinearScale,
  CategoryScale,
  Title,
  Tooltip,
  Legend,
  Filler
);
```

**Required for:** Progress component with ng2-charts

## Code Style Preferences

1. **TypeScript**: Use strict typing, interfaces for models
2. **Async/Await**: Prefer async/await over promises chains
3. **Error Handling**: Try-catch blocks with user-friendly error messages in Russian
4. **Console Logging**: Use console.log/error for debugging, include descriptive messages
5. **Comments**: Add JSDoc comments for service methods
6. **File Naming**: kebab-case for files (e.g., `user.service.ts`, `edit-profile.component.ts`)
7. **PrettierIgnores**: Use `<!-- prettier-ignore -->` for Angular control flow blocks (`@if`, `@for`, `@let`) when Prettier has parsing issues

## Common Patterns

### Service Method Pattern

```typescript
async methodName(param: Type): Promise<ReturnType> {
  try {
    // Validation
    if (!param) {
      console.error('Descriptive error message');
      throw new Error('User-friendly message in Russian');
    }

    // Business logic
    const result = await someAsyncOperation();

    // Update state
    this.stateSignal.set(result);

    return result;
  } catch (error) {
    console.error('Error message:', error);
    throw error;
  }
}
```

### Component Pattern

```typescript
@Component({
  selector: 'app-component-name',
  standalone: true,
  imports: [CommonModule, FormsModule, RouterLink],
  templateUrl: './component-name.component.html',
  styles: [],
})
export class ComponentNameComponent {
  // Signals for reactive state
  isLoading = signal<boolean>(false);
  errorMessage = signal<string>('');

  constructor(private service: Service, private router: Router) {}

  async onSubmit(): Promise<void> {
    // Implementation
  }
}
```

## Important Notes

- **Profile Caching**: User profiles use cache-first strategy (localStorage + Firestore) for instant page loads
  - localStorage provides immediate display
  - Firestore background sync ensures data freshness
  - Works offline with cached data
  - Cross-device sync via Firestore on login
- **Auth Timing**: Pass userId directly to avoid timing issues with getCurrentUser()
- **Disabled Fields**: Email field in profile is disabled (tied to Firebase Auth)
- **Form Validation**: Email not required in validation if it's disabled
- **Navigation**: Landing page accessible to everyone (modified publicGuard)
- **Protocol Data**: Static TypeScript files, not Firestore (easier to update schedules)
- **Product Filtering**: Daily log only shows supplements user selected in onboarding
- **Week Schedules**: Weeks 2-4 are locked placeholders - add schedules to `protocol-schedule.data.ts` when ready
- **Form Control Keys**: Supplements use `productId-timing-index` pattern for unique form control names

## Implemented Features

✅ **Profile Caching System**:

- Cache-first loading strategy for instant page loads
- localStorage cache with Firestore background sync
- Automatic sync on login (updates stale cache)
- All profile operations update both localStorage and Firestore
- Graceful offline support (uses cache if Firestore unavailable)
- Cross-device data consistency via Firestore

✅ **Protocol System (30-Day Program)**:

- 4-week program structure with phase descriptions
- Protocol schedules stored in static TypeScript files
- Week 1 complete with timing-based supplement schedule
- Weeks 2-4 placeholder structure (locked until schedules added)
- ProtocolService calculates current day/week and filters by user's products
- Phase metadata with Russian titles and descriptions

✅ **User Onboarding**:

- Product selection by region (US, EU, Russia)
- Products grouped by level (base/advanced)
- Selected products saved to Firestore userProfiles
- Region auto-detection based on user's country

✅ **Nutrition Tracking**:

- Nutrition Guide reference page with principles, foods, hydration, practices
- Enhanced Daily Log with habit tracking (9 daily boolean + 2 daily counters)
- Real-time nutrition scoring (0-100 based on weighted habits)
- Contextual tips that appear based on user's missing/incomplete habits
- Beautiful UI with counters (+/- buttons, progress bars)

✅ **Daily Logging**:

- Symptom tracking (5 sliders: digestion, bloating, energy, sleep, puffiness)
- Supplement tracking by timing (Morning, Lunch, Dinner)
  - Shows product images from Cloudinary
  - Displays dosage and notes
  - Only shows user's selected products
  - Visual timing indicators (sun, fire, moon icons)
- Notes section
- Local storage persistence

## Future Features to Consider

- **Meal Journal**: Time-stamped meals with tags (protein, veggies, fiber, etc.)
- **Weekly Habits**: Track 4 weekly goals (14 plants, 3 herbal teas, IF 3+ days, 1 seasonal dish)
- **Progress Charts**: Nutrition score trends, habit heatmap, weekly breakdown
- **Firestore Integration**: Save daily logs to Firestore (currently local storage)
- **Product Recommendations**: Link supplements to habits with Cloudinary images
- **Weekly Reports**: PDF/email summaries with insights
- **Community Features**: Coach integration or group support

## Dependencies

Key packages:

- `@angular/fire` (20.0.1) - Firebase integration
- `@tailwindcss/postcss` (4.1.13) - Styling
- `@tailwindplus/elements` (1.0.14) - UI components
- `@ng-icons/core` + `@ng-icons/heroicons` - Icon system
- `chart.js` (4.5.0) + `ng2-charts` (8.0.0) - Charts
- `firebase` (11.10.0) - Firebase SDK

## Development Commands

- `npm start` - Start dev server
- `npm run build` - Production build
- `npm run watch` - Build with watch mode

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mashablair) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
