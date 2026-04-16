## foodie

> **Project**: Foodie - Comprehensive Meal Planning Web Application

# CLAUDE.md - Foodie Meal Planning PWA

**Project**: Foodie - Comprehensive Meal Planning Web Application
**Version**: 1.0.0
**Last Updated**: 2025-01-10
**Tech Stack**: React 18, TypeScript, Vite 5, Tailwind CSS, Firebase (optional), GitHub Pages

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Architecture](#architecture)
3. [Tech Stack Decisions](#tech-stack-decisions)
4. [Project Structure](#project-structure)
5. [Key Features](#key-features)
6. [Data Model](#data-model)
7. [State Management](#state-management)
8. [Internationalization](#internationalization)
9. [PWA Implementation](#pwa-implementation)
10. [GitHub Integration](#github-integration)
11. [Development Workflow](#development-workflow)
12. [Deployment](#deployment)
13. [Testing Strategy](#testing-strategy)
14. [Performance Optimization](#performance-optimization)
15. [Accessibility](#accessibility)
16. [Future Enhancements](#future-enhancements)
17. [Troubleshooting](#troubleshooting)
18. [Contributing](#contributing)

---

## Project Overview

Foodie is a Progressive Web Application (PWA) designed to help users plan meals, discover recipes, generate shopping lists, and manage their pantry. The app supports multiple languages (English, Spanish, French) and multiple cuisines, making it truly global.

### Core Objectives

- **Flexibility**: Support any cuisine, dietary restriction, or meal planning style
- **Accessibility**: WCAG 2.1 AA compliant, works offline, mobile-first
- **Community-Driven**: GitHub-integrated recipe contribution system
- **Performant**: Fast loading, optimized bundles, PWA capabilities
- **Multilingual**: Full i18n support with easy expansion to more languages

---

## Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    User Interface (React)                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌─────────┐│
│  │  Pages   │  │Components│  │ Layouts  │  │ Routing ││
│  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬────┘│
└───────┼────────────┼──────────────┼──────────────┼─────┘
        │            │              │              │
┌───────▼────────────▼──────────────▼──────────────▼─────┐
│              Context API (State Management)             │
│  ┌──────┐ ┌──────┐ ┌─────────┐ ┌──────┐ ┌───────────┐ │
│  │Recipe│ │Planner│ │Shopping│ │ Auth │ │ Language  │ │
│  └──────┘ └──────┘ └─────────┘ └──────┘ └───────────┘ │
└────────┬────────────────────────────────────────┬──────┘
         │                                        │
┌────────▼────────────────────────────────────────▼──────┐
│                     Services Layer                      │
│  ┌──────────┐  ┌──────────┐  ┌───────────┐  ┌───────┐│
│  │ Recipe   │  │Firebase  │  │  GitHub   │  │ Utils ││
│  │ Service  │  │  Service │  │   API     │  │       ││
│  └──────────┘  └──────────┘  └───────────┘  └───────┘│
└────────┬────────────────────────────────────────┬──────┘
         │                                        │
┌────────▼────────────────────────────────────────▼──────┐
│                      Data Layer                         │
│  ┌──────────┐  ┌──────────────┐  ┌──────────────────┐│
│  │JSON Files│  │ LocalStorage │  │ Firebase/Firestore││
│  │(Static)  │  │  (Offline)   │  │  (Cloud Sync)     ││
│  └──────────┘  └──────────────┘  └──────────────────┘│
└─────────────────────────────────────────────────────────┘
```

### Design Patterns

1. **Context + Hooks Pattern**: Global state managed through React Context API with custom hooks
2. **Container/Presentational**: Separation of logic (containers) and UI (presentational components)
3. **Composition**: Small, reusable components composed to build complex UIs
4. **Service Layer**: Business logic abstracted into service modules
5. **Progressive Enhancement**: Core functionality works without JavaScript, enhanced with React

---

## Tech Stack Decisions

### Why These Technologies?

#### React 18 + TypeScript
- **React**: Industry standard, excellent ecosystem, great performance with concurrent features
- **TypeScript**: Type safety reduces bugs, better IDE support, self-documenting code
- **Hooks**: Modern React patterns, easier state management and side effects

#### Vite 5
- **Fast HMR**: Lightning-fast hot module replacement for better DX
- **ES Modules**: Native ES module support, no bundling in dev
- **Optimized Build**: Rollup-based production builds with automatic code splitting
- **Plugin Ecosystem**: Easy PWA integration with vite-plugin-pwa

#### Tailwind CSS
- **Utility-First**: Rapid development with utility classes
- **Customization**: Easy to customize theme and extend
- **Performance**: PurgeCSS removes unused styles automatically
- **Dark Mode**: Built-in dark mode support with `class` strategy

#### React Context API (vs Redux/Zustand)
- **Built-in**: No external dependencies for state management
- **Sufficient**: App complexity doesn't require heavy state management
- **Simple**: Easier for contributors to understand and extend
- **Performant**: With proper memoization and context splitting

#### i18next
- **Mature**: Battle-tested i18n solution
- **React Integration**: Official react-i18next package
- **Language Detection**: Automatic language detection from browser
- **Async Loading**: Load translations on-demand

#### Firebase (Optional)
- **Authentication**: Easy OAuth integration (Google, GitHub)
- **Firestore**: Real-time database for user data, meal plans
- **Storage**: Cloud storage for user-uploaded recipe images
- **Free Tier**: Generous free tier for small to medium apps

#### GitHub Pages
- **Free**: Free hosting for public repositories
- **CI/CD**: Integrated with GitHub Actions
- **Custom Domains**: Support for custom domains
- **HTTPS**: Automatic HTTPS support

---

## Project Structure

```
foodie/
├── public/                     # Static assets
│   ├── images/                 # Image assets
│   │   ├── recipes/            # Recipe photos
│   │   ├── ingredients/        # Ingredient photos
│   │   ├── icons/              # PWA icons
│   │   └── placeholders/       # Placeholder images
│   ├── locales/                # Translation files
│   │   ├── en/translation.json
│   │   ├── es/translation.json
│   │   └── fr/translation.json
│   └── manifest.webmanifest    # PWA manifest
│
├── src/
│   ├── components/             # React components
│   │   ├── common/             # Reusable UI components
│   │   ├── layout/             # Layout components (Header, Footer)
│   │   ├── recipe/             # Recipe-related components
│   │   ├── planner/            # Meal planner components
│   │   ├── shopping/           # Shopping list components
│   │   ├── contribute/         # Recipe contribution wizard
│   │   ├── auth/               # Authentication components
│   │   └── pantry/             # Pantry management components
│   │
│   ├── contexts/               # React Context providers
│   │   ├── AppContext.tsx      # App-wide settings
│   │   ├── ThemeContext.tsx    # Dark/light theme
│   │   ├── LanguageContext.tsx # i18n language state
│   │   ├── AuthContext.tsx     # User authentication
│   │   ├── RecipeContext.tsx   # Recipe data & filters
│   │   ├── PlannerContext.tsx  # Meal planning state
│   │   └── ShoppingContext.tsx # Shopping list state
│   │
│   ├── hooks/                  # Custom React hooks
│   │   ├── useRecipes.ts       # Recipe operations
│   │   ├── useMealPlans.ts     # Meal plan operations
│   │   ├── useLocalStorage.ts  # LocalStorage wrapper
│   │   ├── useDebounce.ts      # Debounce hook
│   │   └── useMediaQuery.ts    # Responsive design hook
│   │
│   ├── utils/                  # Utility functions
│   │   ├── calculations.ts     # Nutrition, cost calculations
│   │   ├── unitConversions.ts  # Unit conversion logic
│   │   ├── nutritionEstimator.ts # Nutrition estimation
│   │   ├── githubAPI.ts        # GitHub API wrapper
│   │   ├── firebaseAPI.ts      # Firebase operations
│   │   ├── storage.ts          # LocalStorage utilities
│   │   ├── validation.ts       # Form & data validation
│   │   └── formatting.ts       # Date, number formatting
│   │
│   ├── services/               # Business logic services
│   │   ├── recipeService.ts    # Recipe CRUD operations
│   │   ├── ingredientService.ts # Ingredient operations
│   │   ├── plannerService.ts   # Meal planning logic
│   │   └── shoppingService.ts  # Shopping list generation
│   │
│   ├── data/                   # Static data (JSON)
│   │   ├── recipes.json        # Recipe database
│   │   ├── ingredients.json    # Ingredient database
│   │   ├── categories.json     # Categories & taxonomies
│   │   └── config.json         # App configuration
│   │
│   ├── schemas/                # JSON schemas for validation
│   │   ├── recipe.schema.json
│   │   ├── ingredient.schema.json
│   │   └── mealPlan.schema.json
│   │
│   ├── pages/                  # Page components
│   │   ├── HomePage.tsx
│   │   ├── RecipesPage.tsx
│   │   ├── RecipeDetailPage.tsx
│   │   ├── PlannerPage.tsx
│   │   ├── ShoppingListPage.tsx
│   │   ├── ContributePage.tsx
│   │   ├── PantryPage.tsx
│   │   ├── ProfilePage.tsx
│   │   └── NotFoundPage.tsx
│   │
│   ├── types/                  # TypeScript type definitions
│   │   └── index.ts            # All type definitions
│   │
│   ├── styles/                 # Global styles
│   │   └── globals.css         # Tailwind + custom styles
│   │
│   ├── App.tsx                 # Root component
│   ├── main.tsx                # Entry point
│   └── i18n.ts                 # i18next configuration
│
├── .github/
│   └── workflows/              # GitHub Actions
│       ├── deploy.yml          # Deploy to GitHub Pages
│       ├── validate-recipe-pr.yml # Validate recipe PRs
│       └── lighthouse-ci.yml   # Performance testing
│
├── scripts/                    # Build & utility scripts
│   ├── validateJSON.js         # JSON schema validation
│   └── generateSitemap.js      # Sitemap generation
│
├── tests/                      # Test files
│   ├── unit/                   # Unit tests
│   ├── integration/            # Integration tests
│   └── e2e/                    # End-to-end tests
│
├── package.json
├── tsconfig.json
├── vite.config.ts
├── tailwind.config.js
├── .env.example
├── README.md
└── CLAUDE.md                   # This file
```

---

## Key Features

### 1. Recipe Browser
- **Search**: Full-text search across recipes in all languages
- **Filters**: By meal type, cuisine, dietary tags, prep/cook time
- **Sorting**: By rating, time, cost, popularity, name
- **Detail View**: Full recipe with scaling, nutrition facts, timer
- **Favorites**: Save favorite recipes (localStorage or Firebase)

### 2. Meal Planner
- **Calendar Views**: Week and month views
- **Drag & Drop**: Intuitive drag-and-drop recipe assignment
- **Servings Adjustment**: Global and per-meal servings control
- **Plan Templates**: Save and reuse meal plans
- **Sharing**: Generate shareable links for meal plans

### 3. Shopping List
- **Auto-Generation**: Generate from meal plan
- **Smart Consolidation**: Combine same ingredients
- **Category Grouping**: Organize by ingredient category
- **Interactive**: Check off items, add notes
- **Export**: Print, copy, WhatsApp, CSV

### 4. Recipe Contribution
- **Multi-Step Wizard**: 7-step guided recipe submission
- **GitHub Integration**: Automatic PR creation for new recipes
- **Validation**: Client-side and server-side validation
- **Preview**: Preview recipe before submission

### 5. Pantry Management
- **Inventory**: Track ingredients at home
- **Expiration Tracking**: See what's expiring soon
- **Recipe Suggestions**: Suggest recipes based on pantry items

### 6. User System
- **Authentication**: Email/password, Google OAuth
- **Preferences**: Language, theme, dietary restrictions
- **Cloud Sync**: Sync favorites, plans across devices (Firebase)
- **Guest Mode**: Use app without account (localStorage only)

### 7. PWA Features
- **Offline Support**: Service Worker caches recipes and app shell
- **Installable**: Add to home screen prompt
- **Fast Loading**: Optimized bundle, lazy loading
- **Responsive**: Mobile-first design, works on all devices

---

## Data Model

### Recipe

```typescript
interface Recipe {
  id: string;
  name: MultiLangText;               // EN, ES, FR
  description: MultiLangText;
  type: string;                      // breakfast, lunch, dinner, snack, dessert
  cuisine: string[];                 // mediterranean, mexican, etc.
  prepTime: number;                  // minutes
  cookTime: number;
  totalTime: number;
  servings: number;
  difficulty: 'easy' | 'medium' | 'hard';
  tags: string[];                    // gluten-free, vegan, etc.
  dietaryLabels: DietaryLabels;
  nutrition: NutritionInfo;
  ingredients: RecipeIngredient[];
  instructions: RecipeInstruction[];
  tips?: MultiLangText;
  equipment: string[];
  imageUrl?: string;
  author?: string;
  dateAdded: string;
  rating: number;
  reviewCount: number;
}
```

### Ingredient

```typescript
interface Ingredient {
  id: string;
  name: MultiLangText;
  category: string;                  // protein, vegetables, etc.
  unit: string;                      // piece, cup, lb, etc.
  avgPrice: number;
  currency: string;
  region: string;
  tags: {
    glutenFree: boolean;
    vegan: boolean;
    vegetarian: boolean;
    dairyFree: boolean;
    nutFree: boolean;
    kosher: boolean;
    halal: boolean;
  };
  alternatives: string[];
  seasonality: string[];
  storageInstructions: MultiLangText;
}
```

### Meal Plan

```typescript
interface MealPlan {
  id: string;
  name: MultiLangText;
  description: MultiLangText;
  servings: number;
  dietaryRestrictions: string[];
  difficulty: string;
  estimatedCost: number;
  currency: string;
  days: PlanDay[];                   // 7 days typically
  tags: string[];
  isPublic: boolean;
  shareToken?: string;
}
```

---

## State Management

### Context Architecture

We use **React Context API** with separate contexts for different concerns:

1. **AppContext**: App-wide config, online status, PWA install prompt
2. **ThemeContext**: Dark/light theme toggle
3. **LanguageContext**: Current language, translation helpers
4. **AuthContext**: User auth state, sign in/out methods
5. **RecipeContext**: Recipe data, filters, favorites
6. **PlannerContext**: Current meal plan, CRUD operations
7. **ShoppingContext**: Shopping list items, CRUD operations

### Why Not Redux?

- **Simpler**: Fewer concepts, easier for contributors
- **Built-in**: No external dependencies
- **Sufficient**: App complexity doesn't require Redux
- **Performance**: Proper context splitting prevents unnecessary re-renders

### State Persistence

- **LocalStorage**: Meal plans, favorites, shopping lists (offline-first)
- **Firebase**: Optional cloud sync for authenticated users
- **Service Worker**: Cache static assets and API responses

---

## Internationalization

### Implementation

- **Library**: i18next + react-i18next
- **Languages**: English (en), Spanish (es), French (fr)
- **Detection**: Automatic browser language detection
- **Fallback**: English as fallback language
- **Async Loading**: Translation files loaded on-demand

### Translation Files

Located in `public/locales/{lang}/translation.json`:

```json
{
  "app": {
    "name": "Foodie",
    "tagline": "Your Personal Meal Planning Assistant"
  },
  "nav": {
    "home": "Home",
    "recipes": "Recipes",
    ...
  }
}
```

### Usage in Components

```typescript
import { useTranslation } from 'react-i18next';

function MyComponent() {
  const { t } = useTranslation();
  return <h1>{t('app.name')}</h1>;
}
```

### Multilingual Data

Recipes, ingredients, and categories use `MultiLangText`:

```typescript
interface MultiLangText {
  en: string;
  es: string;
  fr: string;
}

// Usage
const recipeName: MultiLangText = {
  en: "Scrambled Eggs",
  es: "Huevos Revueltos",
  fr: "Œufs Brouillés"
};
```

Use `LanguageContext.getTranslated(text)` to get current language version.

---

## PWA Implementation

### Service Worker

Configured via `vite-plugin-pwa`:

```typescript
// vite.config.ts
VitePWA({
  registerType: 'autoUpdate',
  workbox: {
    globPatterns: ['**/*.{js,css,html,ico,png,svg,json}'],
    runtimeCaching: [
      {
        urlPattern: /^https:\/\/fonts\.googleapis\.com\/.*/i,
        handler: 'CacheFirst',
      },
      {
        urlPattern: /^https:\/\/firebasestorage\.googleapis\.com\/.*/i,
        handler: 'CacheFirst',
      }
    ]
  }
})
```

### Manifest

`public/manifest.webmanifest`:

```json
{
  "name": "Foodie - Meal Planner",
  "short_name": "Foodie",
  "theme_color": "#10b981",
  "display": "standalone",
  "start_url": "/",
  "icons": [...]
}
```

### Offline Strategy

- **App Shell**: Cached on first visit
- **Recipes**: Cached after viewing
- **Images**: Cached with CacheFirst strategy
- **API Calls**: Network-first, cache fallback

---

## GitHub Integration

### Recipe Contribution Workflow

1. **User**: Fills out 7-step recipe wizard
2. **App**: Generates JSON matching schema
3. **App**: Uses Octokit to:
   - Fork repository
   - Create feature branch
   - Commit JSON files
   - Create Pull Request
4. **GitHub Actions**: Validates JSON schema, checks duplicates
5. **Maintainer**: Reviews and merges PR
6. **Auto-Deploy**: Merged changes trigger deployment

### OAuth Setup

```typescript
// In contribution wizard
import { Octokit } from '@octokit/rest';

const octokit = new Octokit({
  auth: userGitHubToken
});

// Fork repo
const fork = await octokit.repos.createFork({
  owner: 'your-org',
  repo: 'foodie'
});

// Create branch, commit, PR...
```

### PR Template

```markdown
## New Recipe Submission

**Recipe**: {{name}}
**Submitted by**: @{{username}}

### Checklist
- [ ] JSON schema valid
- [ ] All required fields present
- [ ] Nutrition info reasonable
- [ ] Instructions clear
- [ ] Image appropriate
- [ ] No duplicate recipe
```

---

## Development Workflow

### Getting Started

```bash
# Clone repository
git clone https://github.com/artemiopadilla/foodie.git
cd foodie

# Install dependencies
npm install

# Copy environment variables
cp .env.example .env

# Start development server
npm run dev
```

### Available Scripts

```json
{
  "dev": "vite",                      // Start dev server
  "build": "tsc && vite build",       // Production build
  "preview": "vite preview",          // Preview production build
  "lint": "eslint .",                 // Run ESLint
  "test": "vitest",                   // Run unit tests
  "test:e2e": "playwright test",      // Run E2E tests
  "validate:json": "node scripts/validateJSON.js" // Validate JSON files
}
```

### Code Style

- **Linting**: ESLint with TypeScript rules
- **Formatting**: Prettier (2 spaces, single quotes)
- **Naming**:
  - Components: PascalCase (e.g., `RecipeCard.tsx`)
  - Functions: camelCase (e.g., `calculateTotalCost`)
  - Constants: UPPER_SNAKE_CASE (e.g., `MAX_SERVINGS`)
- **Files**:
  - Components: `.tsx`
  - Utilities: `.ts`
  - Styles: `.css`

---

## Deployment

### GitHub Pages

Deployed automatically via GitHub Actions:

```yaml
# .github/workflows/deploy.yml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
      - run: npm ci
      - run: npm run build
      - uses: actions/deploy-pages@v4
```

### Base Path

Configured in `vite.config.ts`:

```typescript
export default defineConfig({
  base: '/foodie/',  // Replace with your repo name
  ...
});
```

### Custom Domain (Optional)

1. Add `CNAME` file to `public/` directory
2. Configure DNS settings at your domain provider
3. Enable HTTPS in GitHub Pages settings

---

## Testing Strategy

### Unit Tests (Vitest)

Test utilities, hooks, and pure functions:

```typescript
// utils/unitConversions.test.ts
import { convertUnit } from './unitConversions';

describe('convertUnit', () => {
  it('converts cups to ml', () => {
    expect(convertUnit(1, 'cup', 'ml')).toBe(240);
  });
});
```

### Integration Tests

Test component interactions:

```typescript
// RecipeCard.test.tsx
import { render, screen } from '@testing-library/react';
import RecipeCard from './RecipeCard';

test('displays recipe name', () => {
  const recipe = { name: { en: 'Test Recipe' }, ... };
  render(<RecipeCard recipe={recipe} />);
  expect(screen.getByText('Test Recipe')).toBeInTheDocument();
});
```

### E2E Tests (Playwright)

Test critical user flows:

```typescript
// tests/e2e/meal-planner.spec.ts
test('create meal plan', async ({ page }) => {
  await page.goto('/planner');
  await page.click('text=Create New Plan');
  await page.fill('input[name="planName"]', 'My Plan');
  await page.click('text=Save');
  await expect(page.locator('text=My Plan')).toBeVisible();
});
```

---

## Performance Optimization

### Bundle Optimization

- **Code Splitting**: Route-based lazy loading
- **Tree Shaking**: Remove unused code
- **Minification**: Terser for JS, cssnano for CSS
- **Compression**: gzip/brotli enabled

### Image Optimization

- **Format**: WebP with JPEG fallback
- **Lazy Loading**: Images below fold lazy loaded
- **Responsive**: srcset for different screen sizes
- **Compression**: All images optimized

### Caching Strategy

- **Static Assets**: Cache-First (1 year)
- **API Calls**: Network-First with cache fallback
- **Images**: Cache-First (30 days)

### Lighthouse Scores

Target scores:
- Performance: >90
- Accessibility: 100
- Best Practices: 100
- SEO: 100

---

## Accessibility

### WCAG 2.1 AA Compliance

- **Semantic HTML**: Proper use of headings, landmarks
- **Keyboard Navigation**: All interactive elements keyboard accessible
- **ARIA Labels**: Screen reader support
- **Color Contrast**: 4.5:1 for normal text, 3:1 for large text
- **Focus Indicators**: Visible focus states
- **Alt Text**: All images have descriptive alt text
- **Forms**: Proper labels, error messages

### Testing

- **Automated**: Lighthouse, axe-core
- **Manual**: Keyboard navigation, screen reader testing
- **Tools**: WAVE, axe DevTools

---

## Future Enhancements

### Phase 2
- Native mobile apps (React Native)
- Voice commands (Alexa, Google Assistant)
- Barcode scanning for pantry management
- AI recipe suggestions
- Social features (follow users, share plans)

### Phase 3
- Grocery delivery integration (Instacart, Amazon Fresh)
- Meal prep video tutorials
- Nutritionist consultation booking
- Integration with fitness trackers
- Smart appliance integration (IoT)

---

## Troubleshooting

### Common Issues

#### 1. Build Fails with TypeScript Errors

```bash
# Clear cache and reinstall
rm -rf node_modules package-lock.json
npm install
npm run build
```

#### 2. PWA Not Updating

```bash
# Clear service worker cache
# In browser DevTools: Application > Service Workers > Unregister
# Then hard refresh: Ctrl+Shift+R (Windows) or Cmd+Shift+R (Mac)
```

#### 3. Images Not Loading

Check `vite.config.ts` base path matches your deployment URL:

```typescript
export default defineConfig({
  base: '/foodie/',  // Must match GitHub repo name
});
```

#### 4. i18n Translations Not Loading

Ensure translation files are in `public/locales/{lang}/translation.json`, not in `src/`.

---

## Contributing

### How to Contribute

1. **Fork** the repository
2. **Create** a feature branch: `git checkout -b feature/amazing-feature`
3. **Commit** your changes: `git commit -m 'Add amazing feature'`
4. **Push** to the branch: `git push origin feature/amazing-feature`
5. **Open** a Pull Request

### Contribution Guidelines

- Follow existing code style
- Add tests for new features
- Update documentation
- Ensure all tests pass
- Use semantic commit messages

### Adding Recipes

Use the in-app recipe contribution wizard for the best experience. It will:
- Validate your recipe
- Generate proper JSON
- Create a PR automatically

---

## Maintainer Notes

### Adding a New Language

1. Add language code to `src/i18n.ts`:
   ```typescript
   supportedLngs: ['en', 'es', 'fr', 'de'],
   ```

2. Create translation file: `public/locales/de/translation.json`

3. Update `MultiLangText` interface in `src/types/index.ts`:
   ```typescript
   interface MultiLangText {
     en: string;
     es: string;
     fr: string;
     de: string;
   }
   ```

4. Add recipes/ingredients in new language

### Reviewing Recipe PRs

Check:
- [ ] JSON schema valid (automated)
- [ ] Nutrition values reasonable
- [ ] Instructions clear and numbered
- [ ] Image appropriate (no copyrighted content)
- [ ] No duplicate recipe
- [ ] All 3 languages present and accurate

### Deployment

Push to `main` branch triggers automatic deployment via GitHub Actions. No manual steps required.

---

## License

MIT License - see LICENSE file for details.

---

## Contact

- **Issues**: https://github.com/artemiopadilla/foodie/issues
- **Discussions**: https://github.com/artemiopadilla/foodie/discussions
- **Email**: foodie@example.com

---

**Generated with Claude Code** - Anthropic's AI-powered coding assistant
Last updated: 2025-01-10

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArtemioPadilla) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
