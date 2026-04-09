## portalznackare

> 1. **NEVER RUN DESTRUCTIVE DATABASE COMMANDS**

# AI Context - Portál značkaře

## 🚨 ABSOLUTELY CRITICAL - DATA SAFETY - HIGHEST PRIORITY 🚨

### NEVER DO WITHOUT EXPLICIT PERMISSION:
1. **NEVER RUN DESTRUCTIVE DATABASE COMMANDS**
   - ❌ NO `doctrine:schema:drop`
   - ❌ NO `DROP TABLE` or `DROP DATABASE`
   - ❌ NO `TRUNCATE`
   - ❌ NO `DELETE FROM`
   - ❌ NO database recreation/reset
   - ✅ ALWAYS ASK: "This will DELETE ALL DATA. Do you explicitly want me to proceed?"
   - ✅ WAIT for clear "YES, delete the database" before proceeding

2. **NEVER DELETE OR OVERWRITE FILES**
   - ❌ NO `rm -rf` or any file deletion
   - ❌ NO overwriting critical files
   - ✅ ALWAYS warn before any destructive operation

3. **PRODUCTION DATA IS SACRED**
   - This is a PRODUCTION environment unless explicitly stated otherwise
   - Data loss = catastrophic failure
   - When in doubt, DON'T DO IT

## CRITICAL RULES - NEVER BREAK
1. **Documentation = Definition of Done** - No code without updating docs/features/
2. **Hybrid Architecture ONLY** - Twig pages + React micro-apps, NO full SPA
3. **Material UI ONLY for tables** - Everything else uses Tailwind + BEM
4. **Dark mode mandatory** - Every component must support light/dark
5. **Czech project** - UI text, comments, and user-facing content in Czech
6. **Communication language** - Always respond to user in Czech, internal AI context can be any language
7. **Snake_Case Czech parameters ONLY** - All data fields, form properties, and calculations in Czech Snake_Case format for INSYZ consistency
8. **No spontaneous changes** - Any change must be discussed and approved by the user.
9. **KISS principle** - Keep It Simple, Stupid. Always choose the simplest solution that works. No overengineering.

## Memory Guidelines
- **Critical Security Rule:** Uvěřuj si veškeré informace buď v kódu a nebo u uživatele, nepracuj s výmysli a doměnkami a nezapisuj je do dokumentace

## Architecture Pattern
```
Twig Template → React App Mount → Symfony API → MockMSSQLService (dev)
              ↘ Tailwind+BEM    ↘ PostgreSQL (prod data)
```

**Key constraint:** React apps are micro-frontends mounted in Twig pages, not route-based SPA.

## Tech Stack
- **Backend:** Symfony 6.4 LTS + PHP 8.3 + PostgreSQL + MSSQL mock
- **Frontend:** Twig templating + React 18 micro-apps + Tailwind CSS + BEM methodology
- **React UI:** Material React Table ONLY, rest is Tailwind
- **Dev:** DDEV (https://portalznackare.ddev.site)

## File Locations (Critical Paths)
```
assets/js/apps/           → React micro-apps (App.jsx + index.jsx)
assets/css/components/    → BEM components with Tailwind @apply
templates/components/     → Reusable Twig components  
templates/pages/          → Page templates with React mount points
src/Controller/Api/       → InsysController (MSSQL) + PortalController (PostgreSQL)
src/Service/              → Business logic services
docs/features/            → MAIN functional documentation
```

## Decision Trees for Common Tasks

### New Feature Implementation
```
1. Create docs/features/feature-name.md FIRST (before any code)
2. Identify if needs React app → create assets/js/apps/app-name/
3. Create Twig template with data-app mount point
4. Add webpack entry in webpack.config.js
5. Create BEM component in assets/css/components/ with dark mode
6. Update docs/overview.md with cross-links
7. Test both light/dark modes
```

### React App Creation
```
MUST HAVE:
- App.jsx (main component)  
- index.jsx (mount logic with document.querySelector('[data-app="name"]'))
- Dark mode state sync with document.documentElement.classList
- Material UI ONLY if using Material React Table
- All other UI uses Tailwind classes

MOUNT PATTERN:
const container = document.querySelector('[data-app="app-name"]');
if (container) {
    const root = createRoot(container);
    root.render(<App />);
}
```

### API Endpoint Creation
```
Controller choice:
- InsysController → MSSQL mock data (příkazy, users from KČT system)
- PortalController → PostgreSQL data (new app-specific data)

MUST UPDATE:
- docs/api/endpoint-name.md
- Cross-link from relevant docs/features/ document
```

### Styling Rules
```scss
// BEM + Tailwind pattern (MANDATORY)
.component-name {
    @apply base-tailwind-classes;
    @apply dark:dark-variant-classes; // Dark mode REQUIRED
    
    &__element {
        @apply element-classes;
        @apply dark:dark-element-classes;
    }
    
    &--modifier {
        @apply modifier-classes;
    }
}

// KČT Brand Colors (CSS Variables)
:root {
    --color-kct-green: #009C00;   // Zelená značka
    --color-kct-blue: #1a6dff;    // Modrá značka  
    --color-kct-yellow: #ffdd00;  // Žlutá značka
    --color-kct-red: #e50313;     // Červená značka
}

// Badge Component Examples
.badge {
    @apply inline-flex items-center px-2 py-1 rounded-md text-xs font-medium;
    
    // KČT trail colors use CSS variables
    &--kct-ce { background-color: var(--color-kct-red); color: white; }
    &--kct-mo { background-color: var(--color-kct-blue); color: white; }
    
    // Semantic colors use Tailwind
    &--primary { @apply bg-blue-600 text-white dark:bg-blue-700; }
    &--danger { @apply bg-red-600 text-white dark:bg-red-700; }
}
```

### Component Styling Decision Tree
```
Styling komponentu?
├── Standardní pattern? → BEM komponenta (.badge, .card, .btn)
├── KČT brand prvek? → CSS variables (var(--color-kct-*))
├── Specifická úprava? → BEM struktura + Tailwind třídy
└── Jinak → Čistě Tailwind utility třídy

Badge použití:
- KČT barvy: className={`badge badge--kct-${kod.toLowerCase()}`}
- Semantic: className="badge badge--success"  
- Custom: className="badge badge--md bg-indigo-100 text-indigo-800"
```

## Tabler Icons System

### SVG Sprite Implementation
Používá optimalizovaný SVG sprite systém pro výkon a čistotu kódu.

**React aplikace:** Import z `@tabler/icons-react`
```javascript
import { 
    IconArrowLeft, 
    IconTool, 
    IconDownload,
    IconEdit,
    IconCheck,
    IconAlertTriangle 
} from '@tabler/icons-react';

// Základní použití
<IconArrowLeft size={16} />
<IconTool size={20} className="text-blue-600" />
<IconCheck size={14} className="mr-1" />
```

**Twig templates:** Funkce `tabler_icon()`
```twig
{# Základní použití #}
{{ tabler_icon('home') }}
{{ tabler_icon('user', 20) }}
{{ tabler_icon('settings', 24, 'icon--lg') }}

{# V tlačítkách #}
<button class="btn btn--primary">
    {{ tabler_icon('plus', 16) }}
    Přidat
</button>

{# S CSS třídami pro barvy a velikosti #}
{{ tabler_icon('check', 20, 'text-green-600 dark:text-green-400') }}
{{ tabler_icon('alert-triangle', 24, 'icon--warning') }}
{{ tabler_icon('download', 16, 'icon--sm icon--primary') }}

{# S dodatečnými HTML atributy #}
{{ tabler_icon('download', 20, 'icon--primary', {
    'data-action': 'download',
    'aria-label': 'Stáhnout soubor'
}) }}

{# V navigaci #}
<a href="/prikazy" class="nav__link">
    {{ tabler_icon('clipboard-list', 20) }}
    Příkazy
</a>
```

### CSS Icon Classes
```scss
// Automaticky generované třídy
.icon {
    @apply inline-block; // Base icon styling
}

.icon-tabler {
    stroke: currentColor; // Respektuje text color
}

.icon-tabler-home { /* specifická ikona */ }

// Utility třídy pro velikosti
.icon--xs { @apply w-4 h-4; }    // 16px
.icon--sm { @apply w-5 h-5; }    // 20px  
.icon--md { @apply w-6 h-6; }    // 24px (default)
.icon--lg { @apply w-7 h-7; }    // 28px
.icon--xl { @apply w-8 h-8; }    // 32px

// Utility třídy pro barvy
.icon--primary { @apply text-blue-600 dark:text-blue-400; }
.icon--success { @apply text-green-600 dark:text-green-400; }
.icon--warning { @apply text-orange-600 dark:text-orange-400; }
.icon--danger { @apply text-red-600 dark:text-red-400; }
.icon--muted { @apply text-gray-500 dark:text-gray-400; }
```

### Výhody Sprite Systému
- **Výkon:** Jeden HTTP request pro všechny ikony
- **Čistota:** Žádné inline SVG v HTML
- **Bezpečnost:** XSS ochrana s `htmlspecialchars()`
- **Cache:** Sprite se načte jednou a cache-uje
- **Flexibilita:** CSS styling, dark mode support
- **Udržovatelnost:** Auto-sync s Tabler Icons updates

### Technická implementace
```php
// src/Twig/TablerIconExtension.php
public function renderIcon(string $name, int $size = 24, string $class = '', array $attributes = []): string 
{
    return sprintf(
        '<svg%s><use href="/images/tabler-sprite.svg#tabler-%s"></use></svg>',
        $attributeString,
        htmlspecialchars($name) // XSS ochrana
    );
}
```

**Sprite lokace:** `/public/images/tabler-sprite.svg` (statická cesta)
**Použitelné ikony:** Celá Tabler Icons kolekce (4000+ ikon)

## Frontend Architecture Rules (MANDATORY)

### Component Structure
```
Component/
├── ComponentName.jsx          // Pure UI render + minimal logic
├── utils/
│   ├── componentLogic.js      // Business logic, data processing
│   ├── componentValidation.js // Form validation, data validation
│   └── componentHelpers.js    // Helper functions, formatters
└── hooks/
    └── useComponentState.js   // Custom hooks for state management
```

### Debug System (MANDATORY)
**Environment Variables:**
- `DEBUG_PHP=true/false` - Backend debugging
- `DEBUG_LOG=true/false` - Backend detailed logging
- `DEBUG_APPS=true/false` - Frontend console logging

**Usage in React Components:**
```javascript
import { createDebugLogger } from '../../utils/debug';

const logger = createDebugLogger('ComponentName');

// Lifecycle events
logger.lifecycle('Component mounted', { props });

// API calls
logger.api('POST', '/api/endpoint', requestData, responseData);

// State changes
logger.state('formData', oldValue, newValue);

// Errors (always logged)
logger.error('Operation failed', error);

// Performance
logger.performance('Operation', 150); // ms

// Custom logging
logger.custom('Custom event', data);
```

### Component Separation Rules
1. **UI Components (JSX files):**
   - Only JSX render logic
   - Event handlers that call utils functions
   - Basic state for UI-only concerns (modals, dropdowns)
   - React.memo for optimization

2. **Utils Files:**
   - Data transformation logic
   - Business rules and calculations
   - Form validation functions
   - API data preparation
   - Complex computations

3. **Custom Hooks:**
   - State management logic
   - Effect lifecycle management
   - Shared stateful logic between components

### File Naming Convention
```
utils/
├── apiService.js           // API calls and data fetching
├── formUtils.js           // Form helpers, validation
├── dataProcessing.js      // Data transformation
├── businessLogic.js       // Domain-specific logic
└── helpers.js             // Generic helper functions
```

### Debug Logger Configuration
**Set via Twig template:**
```twig
<div id="app-root" data-debug="{{ app.debug ? 'true' : 'false' }}"></div>
```

**Logger Types:**
- `logger.lifecycle()` - Component lifecycle events
- `logger.render()` - Component renders
- `logger.api()` - API requests/responses
- `logger.state()` - State changes
- `logger.error()` - Errors (always shown)
- `logger.performance()` - Performance metrics
- `logger.data()` - Data processing

## Parameter Naming Convention (MANDATORY)

### **Snake_Case Czech Parameters ONLY**
All data fields, form properties, calculations, and user data MUST use Czech Snake_Case format for INSYS consistency.

**Hierarchy:** část → oblast → vlastnost

```javascript
// ✅ CORRECT - Czech Snake_Case
const formData = {
    Datum_Provedeni: new Date(),
    Skupiny_Cest: [...],
    Hlavni_Ridic: "...",
    SPZ: "...",
    Cast_A_Dokoncena: false,
    Trasa_Poznamka: "...",
    Zaplatil: "..."
};

// ❌ WRONG - English or camelCase
const formData = {
    executionDate: new Date(),
    travelGroups: [...],
    primaryDriver: "...",
    vehicleRegistration: "...",
    partACompleted: false,
    routeNotes: "...",
    paidByMember: "..."
};
```

**Segment Properties (Travel Data):**
```javascript
const segment = {
    Cas_Odjezdu: "",
    Cas_Prijezdu: "",
    Misto_Odjezdu: "",
    Misto_Prijezdu: "",
    Druh_Dopravy: "AUV",
    Kilometry: 0,
    Naklady: 0,
    Prilohy: []
};
```

**Accommodation/Expenses:**
```javascript
const accommodation = {
    Zarizeni: "",
    Misto: "",
    Castka: 0,
    Zaplatil: ""
};
```

### Parameter Audit Checklist
- [ ] All form fields in Czech Snake_Case
- [ ] All calculation properties in Czech
- [ ] All DTO properties match frontend
- [ ] All database JSON paths use Czech names
- [ ] No English parameter names in user data

## Documentation Update Triggers
- New PHP service → Update docs/features/ + configuration/services.md
- New API endpoint → Create docs/api/endpoint-name.md
- New React app → Update docs/features/ + frontend/architecture.md
- Security changes → Update configuration/security.md
- Environment changes → Update configuration/environment.md
- UI component → Update relevant feature doc + frontend/components.md
- Debug system changes → Update docs/development/debugging.md

## Test Credentials & Debugging
- Login: `test` / `test`
- DDEV URL: https://portalznackare.ddev.site
- Mock data: src/Service/MockMSSQLService.php
- User: `$this->getUser()` returns User entity with getJmeno(), getIntAdr()

## Red Flags (Auto-Reject These Patterns)
- Material UI outside of Material React Table usage
- React routing/navigation (use Twig routing only)
- CSS-in-JS or styled-components (use BEM + Tailwind)
- Missing dark mode variants
- New code without corresponding docs/features/ update
- API changes without docs/api/ update
- **Business logic mixed with UI components**
- **Missing debug logging in new components**
- **Components without utils separation**
- **Console.log instead of debug logger**
- **Heavy computation in render methods**
- **Missing React.memo optimization**
- **English parameter names in user data** (must be Czech Snake_Case)
- **camelCase parameters** (must be Snake_Case)
- **Mixed naming conventions** (frontend vs backend inconsistency)

## Quick Reference Commands
```bash
# Development
ddev start && ddev npm run watch

# Check docs consistency  
grep -r "TODO" docs/
find docs/ -name "*.md" -mtime -1
```

## Development Workflow
1. **Create component structure** with separated utils
2. **Add debug logging** to all functions
3. **Implement React.memo** for optimization
4. **Test with DEBUG_APPS=true** for proper logging
5. **Update documentation** in docs/features/
6. **Verify dark mode compatibility**
7. **Audit parameter names** for Czech Snake_Case consistency
## Memory Guidelines
- **Critical Security Rule:** Uvěřuj si veškeré informace buď v kódu a nebo u uživatele, nepracuj s výmysli a doměnkami a nezapisuj je do dokumentace

---
**This context is optimized for AI decision-making efficiency.**
**Human documentation is in docs/ directory.**
**Last updated:** 2025-08-03 - Added Parameter Naming Convention (Czech Snake_Case MANDATORY)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martin-svoboda)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/martin-svoboda)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
