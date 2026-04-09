## water-tracker-v2

> 1. [Project Overview](#project-overview)

# CLAUDE.md - AI Assistant Guide for Water Tracker PWA v2.0

## Table of Contents
1. [Project Overview](#project-overview)
2. [Codebase Structure](#codebase-structure)
3. [Architecture & Design Patterns](#architecture--design-patterns)
4. [Key Conventions](#key-conventions)
5. [Development Workflows](#development-workflows)
6. [Testing & Quality Assurance](#testing--quality-assurance)
7. [Deployment](#deployment)
8. [AI Assistant Guidelines](#ai-assistant-guidelines)

---

## Project Overview

### What is Water Tracker PWA?
A Progressive Web App (PWA) for tracking daily water intake with native installation capabilities on iOS and Android devices. The app works offline, stores data locally, and provides a native app-like experience.

### Technology Stack
- **Frontend**: Vanilla JavaScript (ES6+ Class-based)
- **Styling**: CSS3 with CSS Custom Properties
- **PWA**: Service Worker with smart caching strategies
- **Storage**: localStorage for all data persistence
- **Build**: None - pure static files served directly
- **Deployment**: GitHub Pages

### Key Features
- Daily water intake tracking (8-10 cups goal)
- Quick-add buttons for 1 or 2 cups
- Visual progress indicators with color-coded feedback
- History views: Week, Month, Year
- Data export/import (JSON format)
- Offline-first functionality
- URL shortcuts for PWA home screen
- Service worker auto-updates with user notification
- Comprehensive accessibility (ARIA labels, keyboard navigation)

---

## Codebase Structure

### File Organization
```
water-tracker-v2/
├── index.html              # Main HTML with ARIA labels and CSP
├── app.js                  # Main application logic (WaterTracker class)
├── styles.css              # Styles with CSS custom properties
├── sw.js                   # Service Worker with caching strategies
├── manifest.json           # PWA manifest with shortcuts
├── icons/                  # PWA icons (72x72 to 512x512)
│   └── icon-*.png
├── README.md               # User-facing documentation
├── DEVELOPMENT_HISTORY.md  # Development session log
├── CLAUDE.md              # This file - AI assistant guide
└── .gitignore             # Git ignore rules
```

### File Responsibilities

#### `index.html` (12,417 bytes)
**Purpose**: Main HTML structure with accessibility and security enhancements

**Key Features**:
- Content Security Policy (CSP) meta tag
- ARIA labels on all interactive elements
- Semantic HTML with proper roles
- iOS PWA meta tags
- Service worker registration with auto-update logic
- Update notification banner system

**Important Sections**:
- Lines 8-13: CSP configuration
- Lines 20-24: iOS PWA support
- Lines 65-78: Quick-add buttons with ARIA labels
- Lines 92-96: History tabs with ARIA roles
- Lines 130-164: Settings modal
- Lines 169-298: Service worker registration and update handling

#### `app.js` (32,803 bytes)
**Purpose**: Main application logic using class-based architecture

**Class Structure**:
```javascript
class WaterTracker {
    static CONSTANTS = {...}  // Lines 8-18: Centralized constants

    constructor()             // Lines 20-38: Initialize app
    initializeApp()          // Lines 40-86: DOM element validation
    setupEventListeners()    // Lines 88-188: Event binding

    // Data Management
    loadData()               // Lines 351-387: Load from localStorage
    saveData()               // Lines 389-405: Save to localStorage
    saveToHistory()          // Lines 328-346: Store daily data

    // Display Updates
    updateDisplay()          // Lines 243-287: Update UI with current data
    updateDate()             // Lines 290-302: Update date display
    updateHistory()          // Lines 429-444: Update history view

    // User Actions
    addGlasses(amount)       // Lines 209-230: Add water intake
    resetDay()               // Lines 233-240: Reset today's data

    // History Management
    switchPeriod(period)     // Lines 408-425: Switch week/month/year
    navigatePeriod(direction) // Lines 600-612: Navigate time periods
    getHistoryDays()         // Lines 446-483: Calculate history data
    groupDataByWeeks(days)   // Lines 615-642: Group for month view
    groupDataByMonths(days)  // Lines 644-676: Group for year view

    // Settings & Data
    openSettings()           // Lines 678-693: Open modal with escape handler
    closeSettings()          // Lines 695-708: Close modal
    exportData()             // Lines 710-738: Export JSON backup
    importData(file)         // Lines 740-799: Import JSON backup
    resetAllData()           // Lines 801-822: Reset all data

    // Utilities
    handleURLParameters()    // Lines 190-206: Process URL shortcuts
    animateAddition(amount)  // Lines 305-325: Show popup animation
    showMessage(msg, type)   // Lines 824-855: User notifications
    checkForUpdates()        // Lines 857-956: Manual update check
}
```

**Critical Constants** (Lines 8-18):
```javascript
static CONSTANTS = {
    MIN_DAILY_GOAL: 8,           // Minimum daily goal
    MAX_DAILY_GOAL: 10,          // Maximum daily goal (enforced)
    ANIMATION_DURATION: 1200,     // Popup animation duration (ms)
    MESSAGE_DURATION: 3000,       // Toast message duration (ms)
    UPDATE_CHECK_INTERVAL: 60000, // SW update check interval (ms)
    WEEK_DAYS: 7,                // Days in week view
    MONTH_DAYS: 30,              // Days in month view
    YEAR_DAYS: 365,              // Days in year view
    APP_VERSION: '2.0.0'         // Current app version
};
```

#### `styles.css` (15,339 bytes)
**Purpose**: All styling with CSS custom properties for theming

**CSS Variable Structure** (Lines 1-44):
```css
:root {
    /* Colors */
    --color-background: #1a1a1a;
    --color-card-bg: #2a2a2a;
    --color-text-primary: #ffffff;
    --color-text-secondary: #888;

    /* Brand Colors */
    --color-primary: #4a9eff;
    --color-success: #28a745;
    --color-danger: #dc3545;

    /* Spacing */
    --spacing-xs: 4px;
    --spacing-sm: 8px;
    --spacing-md: 15px;
    --spacing-lg: 20px;
    --spacing-xl: 25px;

    /* Border Radius */
    --border-radius-sm: 4px;
    --border-radius-lg: 15px;
    --border-radius-xl: 20px;
    --border-radius-xxl: 25px;

    /* Shadows */
    --shadow-sm: 0 2px 10px rgba(0, 0, 0, 0.3);

    /* Transitions */
    --transition-fast: 0.2s ease;
    --transition-normal: 0.3s ease;
}
```

**Key Sections**:
- Lines 1-44: CSS custom properties
- Lines 83-88: Card base styles
- Lines 213-243: Button styles with focus states
- Lines 328-353: Tab component styles
- Lines 522-697: Modal styles
- Lines 699-719: Animation keyframes
- Lines 738-759: Responsive breakpoints

#### `sw.js` (4,914 bytes)
**Purpose**: Service Worker for offline functionality and caching

**Caching Strategy**:
- **Network-first**: HTML documents (lines 28-48)
- **Cache-first**: All other resources (lines 50-81)
- **Cache version**: `hydrateme-v2000000` (line 1)

**Event Handlers**:
- Lines 12-20: `install` - Cache initial resources
- Lines 23-81: `fetch` - Smart caching with fallbacks
- Lines 84-102: `activate` - Clean old caches, claim clients
- Lines 105-109: `message` - Handle skip waiting
- Lines 112-118: `notificationclick` - Open app
- Lines 121-148: `push` - Push notifications (future)
- Lines 151-162: `sync` - Background sync (future)

#### `manifest.json` (3,042 bytes)
**Purpose**: PWA configuration

**Key Configuration**:
- Lines 2-4: App name and description
- Lines 5-10: Display mode and theme
- Lines 11-29: Screenshots metadata
- Lines 31-80: Icons (8 sizes: 72x72 to 512x512)
- Lines 81-108: Shortcuts for quick water logging

**URL Shortcuts**:
```json
{
  "name": "Add 1 Cup",
  "url": "./index.html?action=add&amount=250"
}
```
Handled by `handleURLParameters()` in app.js:193-206

---

## Architecture & Design Patterns

### Design Patterns Used

#### 1. **Single Class Architecture**
- Entire app logic encapsulated in `WaterTracker` class
- Single instance created on `DOMContentLoaded`
- No frameworks or external dependencies

#### 2. **Observer Pattern (Implicit)**
- Event listeners for all user interactions
- Service worker message passing for updates
- DOM updates trigger from state changes

#### 3. **Module Pattern**
- Static CONSTANTS object for configuration
- Private state via class properties
- Public methods for user actions

#### 4. **Strategy Pattern**
- Different caching strategies in service worker
- Different rendering strategies for week/month/year views
- Different grouping algorithms for history data

### Data Flow

```
User Action
    ↓
Event Listener (setupEventListeners)
    ↓
Action Method (addGlasses, switchPeriod, etc.)
    ↓
Update State (this.currentGlasses, this.historicalData, etc.)
    ↓
Save to localStorage (saveData, saveToHistory)
    ↓
Update Display (updateDisplay, updateHistory)
    ↓
Render to DOM
```

### State Management

**State Location**: All state stored in `WaterTracker` class instance

**State Properties**:
```javascript
{
    currentGlasses: 0,           // Today's water intake
    dailyGoal: 8,                // Daily minimum goal
    maxGoal: 10,                 // Daily maximum goal
    historicalData: {},          // All historical data {date: {glasses, goal, ...}}
    currentPeriod: 'week',       // Current view: 'week'|'month'|'year'
    currentOffset: 0,            // Offset for date navigation
    lastHistoryHash: null,       // For optimized rendering
    animationStyleInjected: false, // Animation style injection flag
    escapeHandler: null          // Escape key handler reference
}
```

**Storage Keys**:
- `waterTrackerHistory`: Historical data object
- `waterTrackerSettings`: User settings (goals, preferences)

**Data Schema**:
```javascript
historicalData = {
    "Wed Nov 20 2025": {
        glasses: 8,
        goal: 8,
        goalMax: 10,
        date: "Wed Nov 20 2025",
        timestamp: 1732060800000
    },
    // ... more dates
}
```

### Error Handling Strategy

**All localStorage operations wrapped in try-catch** (app.js):
- Lines 340-346: Save to history with error handling
- Lines 352-387: Load data with error handling
- Lines 392-405: Save data with error handling
- Lines 810-815: Clear storage with error handling

**User-facing error messages**:
- Storage full: "Failed to save data. Storage may be full."
- Load failure: "Failed to load saved data."
- Import failure: "Failed to import data. Please check the file format."

**DOM validation**:
- Lines 73-84: Validates all required DOM elements exist
- Logs errors for missing elements but continues execution

---

## Key Conventions

### JavaScript Conventions

#### 1. **Naming Conventions**
- **Classes**: PascalCase (`WaterTracker`)
- **Methods**: camelCase (`updateDisplay`, `addGlasses`)
- **Constants**: UPPER_SNAKE_CASE (`MIN_DAILY_GOAL`)
- **Private handlers**: camelCase with descriptive names (`escapeHandler`)
- **DOM elements**: Stored in `this.elements` object

#### 2. **Code Organization**
- **JSDoc comments** for all public methods
- **Grouped methods** by functionality (Data, Display, History, Settings)
- **No magic numbers** - all constants in CONSTANTS object
- **Template literals** for all string concatenation

#### 3. **Error Handling**
- **Always wrap** localStorage operations in try-catch
- **User-friendly messages** via `showMessage()`
- **Console errors** for debugging information
- **Graceful degradation** when DOM elements missing

#### 4. **Performance Optimizations**
- **Change detection** for history rendering (app.js:434-437)
- **CSS animations** instead of dynamic style injection
- **Single event listener** delegation where possible
- **Cleanup handlers** for escape key listener (app.js:703-707)

### CSS Conventions

#### 1. **CSS Custom Properties**
- **All colors** defined as variables
- **All spacing** uses spacing scale
- **All shadows** predefined
- **All transitions** standardized

#### 2. **Naming Conventions**
- **BEM-like** structure: `.history-day`, `.day-info`, `.day-name`
- **Modifier classes**: `.active`, `.completed`, `.visible`
- **State classes**: `.toast-goal`, `.toast-perfect`

#### 3. **Responsive Design**
- **Mobile-first** approach
- **Single breakpoint** at 480px (lines 738-759)
- **Fluid sizing** with percentages and viewport units

#### 4. **Accessibility**
- **Focus states** for all interactive elements
- **Outline offsets** for better visibility
- **High contrast** colors for readability
- **Disabled states** clearly indicated

### HTML Conventions

#### 1. **Semantic HTML**
- `<button>` for all interactive elements
- `role` attributes for ARIA (tablist, tab, dialog)
- `aria-label` on all buttons
- `aria-selected` for tab states

#### 2. **Structure**
- **Single page** application structure
- **Cards** for content organization
- **Modals** for settings/dialogs
- **Progressive enhancement** approach

#### 3. **Security**
- **Content Security Policy** (CSP) defined in meta tag
- **No inline event handlers** (all addEventListener)
- **Safe data handling** for imports

---

## Development Workflows

### Local Development

#### 1. **Starting a Local Server**
```bash
# Option 1: Python
python3 -m http.server 8000

# Option 2: Node.js
npx serve .

# Option 3: PHP
php -S localhost:8000
```

Access at: `http://localhost:8000`

#### 2. **Development Process**
1. Make changes to files
2. Refresh browser (service worker may need clearing)
3. Test functionality manually
4. Check browser console for errors
5. Validate accessibility with screen reader/keyboard

#### 3. **Service Worker Development**
**Important**: Service workers cache aggressively

**To test SW changes**:
1. Update `CACHE_NAME` version in sw.js (line 1)
2. Update cache busting params in index.html (lines 18, 167)
3. In browser DevTools → Application → Service Workers
4. Click "Unregister" to clear old SW
5. Hard refresh (Cmd+Shift+R or Ctrl+Shift+R)

### Making Changes

#### 1. **Adding a New Feature**
**Example: Add custom cup sizes**

Steps:
1. Update CONSTANTS if needed (app.js:8-18)
2. Add UI elements to index.html with ARIA labels
3. Add styles to styles.css using CSS variables
4. Add event listeners in setupEventListeners()
5. Create handler method with JSDoc comment
6. Update saveData/loadData for persistence
7. Add error handling with try-catch
8. Test manually and check console

#### 2. **Modifying Styles**
**Always use CSS variables**:

```css
/* ✅ GOOD */
.my-element {
    color: var(--color-primary);
    padding: var(--spacing-md);
    border-radius: var(--border-radius-lg);
}

/* ❌ BAD */
.my-element {
    color: #4a9eff;
    padding: 15px;
    border-radius: 15px;
}
```

#### 3. **Updating Constants**
**All magic numbers go in CONSTANTS**:

```javascript
// ✅ GOOD
setTimeout(() => {
    // ...
}, WaterTracker.CONSTANTS.MESSAGE_DURATION);

// ❌ BAD
setTimeout(() => {
    // ...
}, 3000);
```

#### 4. **Adding Event Listeners**
**Always add in setupEventListeners()**:

```javascript
// ✅ GOOD - in setupEventListeners()
if (this.elements.myButton) {
    this.elements.myButton.addEventListener('click', () => {
        this.handleMyAction();
    });
}

// ❌ BAD - inline or scattered
myButton.onclick = () => { ... }
```

### Git Workflow

#### 1. **Branch Naming**
Current branch: `claude/claude-md-miduon1mn2kht5iv-01FVdGaSEPnrizEFsX3acFfh`

**Convention**: `claude/<session-id>` for AI-assisted development

#### 2. **Commit Messages**
Follow existing pattern from DEVELOPMENT_HISTORY.md:
- Clear, descriptive messages
- Focus on "why" not "what"
- Examples:
  - "Add development history documentation"
  - "Initial commit: Water Tracker v2.0 with all improvements"

#### 3. **Push Process**
```bash
# Always push to the designated branch
git add .
git commit -m "Your descriptive message"
git push -u origin claude/your-branch-name
```

**Retry logic**: If push fails due to network, retry up to 4 times with exponential backoff (2s, 4s, 8s, 16s)

---

## Testing & Quality Assurance

### Manual Testing Checklist

#### Core Functionality
- [ ] Add 1 cup of water - progress updates
- [ ] Add 2 cups of water - progress updates
- [ ] Reach daily goal (8 cups) - goal toast appears
- [ ] Reach perfect goal (10 cups) - perfect toast appears
- [ ] Reset day - clears current progress
- [ ] Data persists after page reload

#### History Views
- [ ] Switch to Week view - shows last 7 days
- [ ] Switch to Month view - shows 4 weeks
- [ ] Switch to Year view - shows 12 months
- [ ] Navigate to previous period - loads older data
- [ ] Navigate to next period - returns to current
- [ ] Next button disabled when at current period

#### Settings & Data
- [ ] Open settings modal - displays correctly
- [ ] Close with X button - modal closes
- [ ] Close with Escape key - modal closes
- [ ] Click outside modal - modal closes
- [ ] Export data - downloads JSON file
- [ ] Import data - restores from JSON file
- [ ] Reset all data - clears everything

#### PWA Features
- [ ] Install as PWA - app installs
- [ ] URL shortcuts work - quick add from home screen
- [ ] Offline mode - works without internet
- [ ] Service worker updates - notification appears
- [ ] Update app - new version loads

#### Accessibility
- [ ] Tab navigation - all elements reachable
- [ ] Focus indicators - visible on all elements
- [ ] ARIA labels - present on all buttons
- [ ] Keyboard shortcuts - Escape closes modal
- [ ] Screen reader - announces elements correctly

### Browser Testing

**Required Browsers**:
- Chrome/Edge (Desktop & Mobile)
- Safari (Desktop & iOS)
- Firefox (Desktop)

**PWA Installation Testing**:
- iOS Safari: Share → Add to Home Screen
- Android Chrome: Menu → Install app
- Desktop Chrome: Install icon in address bar

### Performance Considerations

#### Optimizations Implemented
1. **History rendering** only when data changes (app.js:434-437)
2. **CSS animations** instead of JS animations
3. **Cache-first** strategy for static assets
4. **Lazy DOM updates** via change detection

#### Performance Metrics to Monitor
- Time to Interactive (TTI): < 2s on 3G
- First Contentful Paint (FCP): < 1s
- localStorage size: Monitor for quota issues

### Security Testing

#### Content Security Policy Validation
```html
<!-- CSP defined in index.html:8-13 -->
Content-Security-Policy:
  - default-src 'self'
  - style-src 'self' 'unsafe-inline' https://fonts.googleapis.com
  - font-src 'self' https://fonts.gstatic.com
  - img-src 'self' data:
  - script-src 'self' 'unsafe-inline'
```

**Note**: `'unsafe-inline'` required for inline scripts. Consider moving to external files for stricter CSP.

#### Security Checklist
- [ ] No XSS vulnerabilities in user inputs
- [ ] Import data validates JSON structure
- [ ] No sensitive data in localStorage
- [ ] HTTPS enforced (via GitHub Pages)
- [ ] No external script dependencies

---

## Deployment

### GitHub Pages Setup

#### Current Configuration
- **Repository**: `chiefhoser/water-tracker-v2`
- **Branch**: `main`
- **Directory**: `/` (root)
- **Custom Domain**: `www.dannycecil.net`
- **Live URL**: https://www.dannycecil.net/water-tracker-v2/

#### Deployment Process

**Option 1: Push to Main**
```bash
git add .
git commit -m "Your message"
git push origin main
```
GitHub Pages automatically deploys from main branch.

**Option 2: From Feature Branch**
```bash
# Create PR to main
gh pr create --title "Your PR title" --body "Description"

# After review, merge to main
gh pr merge <pr-number>
```

#### Cache Busting Strategy

**When deploying updates**:
1. Update cache version in `sw.js`:
   ```javascript
   const CACHE_NAME = 'hydrateme-v2000001'; // Increment version
   ```

2. Update cache busting params in `index.html`:
   ```html
   <link rel="stylesheet" href="styles.css?v=2000001">
   <script src="app.js?v=2000001"></script>
   ```

3. Service worker will detect new version
4. Users see update notification banner
5. Click "Update Now" to reload with new version

### Deployment Checklist

Before pushing to main:
- [ ] Update cache version in sw.js
- [ ] Update query params in index.html
- [ ] Test locally on http://localhost:8000
- [ ] Clear service worker and test fresh install
- [ ] Test PWA installation
- [ ] Verify all links and resources load
- [ ] Check console for errors
- [ ] Test on mobile device if possible
- [ ] Update version in app.js CONSTANTS if major release
- [ ] Update README.md if new features added

---

## AI Assistant Guidelines

### When Modifying This Codebase

#### ✅ DO:

1. **Read First, Change Later**
   - Always read the entire file before making changes
   - Understand the existing patterns and conventions
   - Check DEVELOPMENT_HISTORY.md for context

2. **Follow Established Patterns**
   - Use the same coding style as existing code
   - Add JSDoc comments for new methods
   - Use CSS variables for all styling values
   - Put all constants in CONSTANTS object

3. **Maintain Code Quality**
   - Wrap localStorage operations in try-catch
   - Validate DOM elements before use
   - Add ARIA labels to new interactive elements
   - Include focus styles for keyboard navigation

4. **Test Changes**
   - Suggest manual testing steps
   - Consider edge cases (empty data, full storage, etc.)
   - Think about mobile/desktop differences
   - Verify PWA functionality still works

5. **Document Changes**
   - Update comments when changing behavior
   - Update README.md for user-facing changes
   - Consider updating this CLAUDE.md if architecture changes
   - Keep DEVELOPMENT_HISTORY.md pattern for major work

#### ❌ DON'T:

1. **Don't Break Conventions**
   - Don't add magic numbers - use CONSTANTS
   - Don't use inline styles - use CSS classes
   - Don't add inline event handlers - use addEventListener
   - Don't hardcode colors - use CSS variables

2. **Don't Over-Engineer**
   - Don't add external dependencies/frameworks
   - Don't create unnecessary abstractions
   - Don't add features that weren't requested
   - Don't refactor working code without good reason

3. **Don't Skip Security**
   - Don't remove CSP protections
   - Don't disable HTTPS
   - Don't trust imported data without validation
   - Don't expose sensitive information in console logs

4. **Don't Break PWA**
   - Don't remove service worker functionality
   - Don't break offline mode
   - Don't remove manifest.json features
   - Don't forget to test installation

### Specific Scenarios

#### Scenario 1: User Asks to Add a New Feature

**Example**: "Add ability to track water temperature (hot/cold)"

**Response Pattern**:
1. ✅ Read relevant files (app.js, index.html, styles.css)
2. ✅ Explain where changes would go:
   - Constants in app.js CONSTANTS
   - UI in index.html with ARIA labels
   - Styles in styles.css with CSS variables
   - Data structure in historicalData schema
3. ✅ Show code following existing patterns
4. ✅ Wrap new localStorage code in try-catch
5. ✅ Add JSDoc comments
6. ✅ Suggest testing steps

#### Scenario 2: User Reports a Bug

**Example**: "Export button not working on iOS Safari"

**Response Pattern**:
1. ✅ Read the relevant code (app.js exportData method)
2. ✅ Identify potential issues (blob handling on iOS)
3. ✅ Propose fix following existing error handling patterns
4. ✅ Test on similar browsers if possible
5. ✅ Suggest user testing steps

#### Scenario 3: User Wants to Change Styling

**Example**: "Make buttons bigger and change color scheme"

**Response Pattern**:
1. ✅ Read styles.css to understand CSS variable system
2. ✅ Identify variables to change (--color-primary, --spacing-md, etc.)
3. ✅ Show changes to CSS variables only
4. ✅ Don't hardcode values - update variables
5. ✅ Consider accessibility (contrast ratios)
6. ✅ Test on mobile viewports

#### Scenario 4: User Asks About Architecture

**Example**: "Why is everything in one class? Should we split it up?"

**Response Pattern**:
1. ✅ Explain the single-class pattern is intentional
2. ✅ No external dependencies keeps it simple
3. ✅ Point to this CLAUDE.md for architecture decisions
4. ✅ Only suggest refactoring if there's clear benefit
5. ✅ Don't over-engineer for theoretical future needs

### Code Review Checklist

When making changes, ensure:

**JavaScript**:
- [ ] All new methods have JSDoc comments
- [ ] Constants added to CONSTANTS object
- [ ] localStorage wrapped in try-catch
- [ ] Error messages user-friendly via showMessage()
- [ ] DOM elements validated before use
- [ ] Event listeners added in setupEventListeners()
- [ ] State updates trigger display updates
- [ ] No memory leaks (cleanup handlers removed)

**CSS**:
- [ ] All values use CSS variables
- [ ] Follow existing naming conventions
- [ ] Include focus states for interactive elements
- [ ] Test responsive design at 480px breakpoint
- [ ] High contrast for accessibility

**HTML**:
- [ ] ARIA labels on interactive elements
- [ ] Semantic HTML tags used
- [ ] role attributes for complex components
- [ ] No inline event handlers
- [ ] Proper heading hierarchy

**Security**:
- [ ] CSP not weakened
- [ ] User inputs validated
- [ ] No XSS vulnerabilities
- [ ] Import data validated
- [ ] No sensitive data exposed

**PWA**:
- [ ] Service worker still functions
- [ ] Offline mode works
- [ ] Cache version updated if needed
- [ ] Manifest.json updated if needed
- [ ] URL shortcuts still work

### Understanding the Codebase

#### Key Files by Use Case

**Want to understand how data is stored?**
- Read: app.js lines 351-405 (loadData, saveData, saveToHistory)
- Storage format: Object with date keys

**Want to add a new button?**
- Read: index.html for structure with ARIA
- Read: styles.css lines 213-243 for button styles
- Read: app.js lines 88-188 for event listener pattern

**Want to change the UI?**
- Read: styles.css lines 1-44 for CSS variables
- Change variables, not individual properties
- Test responsive at 480px

**Want to modify history views?**
- Read: app.js lines 408-676 (switchPeriod, grouping functions)
- Different logic for week/month/year
- Uses change detection for performance

**Want to understand PWA features?**
- Read: sw.js for caching strategies
- Read: manifest.json for PWA config
- Read: index.html lines 169-298 for update handling

#### Common Patterns to Follow

**Pattern 1: Adding Data to State**
```javascript
// 1. Update instance property
this.newProperty = value;

// 2. Save to localStorage (with error handling)
try {
    localStorage.setItem('key', JSON.stringify(data));
} catch (error) {
    console.error('Failed to save:', error);
    this.showMessage('Failed to save data.', 'error');
}

// 3. Update display
this.updateDisplay();
```

**Pattern 2: Adding UI Element**
```html
<!-- 1. HTML with ARIA -->
<button
    id="my-button"
    class="my-btn"
    aria-label="Descriptive label for screen readers">
    Button Text
</button>
```

```css
/* 2. CSS with variables */
.my-btn {
    background: var(--color-primary);
    padding: var(--spacing-md);
    border-radius: var(--border-radius-lg);
    transition: all var(--transition-normal);
}

.my-btn:focus-visible {
    outline: 2px solid var(--color-primary);
    outline-offset: 2px;
}
```

```javascript
// 3. JavaScript event listener
initializeApp() {
    this.elements = {
        myButton: document.getElementById('my-button'),
        // ...
    };
}

setupEventListeners() {
    if (this.elements.myButton) {
        this.elements.myButton.addEventListener('click', () => {
            this.handleMyAction();
        });
    }
}

/**
 * Handle my action
 */
handleMyAction() {
    // Implementation
}
```

**Pattern 3: Adding a Constant**
```javascript
// app.js lines 8-18
static CONSTANTS = {
    // Existing constants...
    MY_NEW_CONSTANT: 42,  // Description of what this is
};

// Usage
const value = WaterTracker.CONSTANTS.MY_NEW_CONSTANT;
```

### Final Notes for AI Assistants

1. **This is production code** - It's already deployed and working. Changes should be carefully considered.

2. **Simplicity is a feature** - The codebase intentionally has no build process, no dependencies, no frameworks. Keep it that way.

3. **Users rely on this** - Data is stored locally. Don't break data compatibility. Migrations need careful planning.

4. **PWA is critical** - Many users have this installed as an app. Service worker must work.

5. **Accessibility matters** - Users with disabilities depend on ARIA labels and keyboard navigation.

6. **When in doubt, ask** - Better to clarify requirements than make assumptions.

7. **Read DEVELOPMENT_HISTORY.md** - It provides valuable context about past decisions and improvements.

8. **Test on mobile** - This is primarily a mobile app. Desktop is secondary.

---

## Document History

**Created**: 2025-11-25
**Version**: 1.0.0
**Author**: AI Assistant (Claude)
**Purpose**: Comprehensive guide for AI assistants working on Water Tracker PWA v2.0

**Last Updated**: 2025-11-25
**Update Notes**: Initial creation based on codebase analysis

---

**For questions or clarifications about this codebase, refer to**:
- This document (CLAUDE.md) for architecture and conventions
- README.md for user-facing features and installation
- DEVELOPMENT_HISTORY.md for development context and decisions
- Code comments (JSDoc) for method-level documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chiefhoser)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/chiefhoser)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
