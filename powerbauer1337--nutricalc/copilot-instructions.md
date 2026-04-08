## nutricalc

> NutriCalc ist ein fortschrittlicher Nährstoffrechner für Pflanzen (speziell Cannabis) als React SPA mit TypeScript, Vite, und Tailwind CSS. Die Anwendung bietet präzise Nährstoffberechnungen, AI-gestützte Beratung über Google Gemini API, und umfassende Dünger- und Wassermanagement-Features.

# NutriCalc - Claude Configuration

## Project Overview
NutriCalc ist ein fortschrittlicher Nährstoffrechner für Pflanzen (speziell Cannabis) als React SPA mit TypeScript, Vite, und Tailwind CSS. Die Anwendung bietet präzise Nährstoffberechnungen, AI-gestützte Beratung über Google Gemini API, und umfassende Dünger- und Wassermanagement-Features.

## Development Rules & Guidelines

### Code Quality Standards
- **TypeScript**: Strict mode enabled, no `any` types, proper interfaces
- **Components**: Functional components with hooks, no class components
- **Naming**: PascalCase for components, camelCase for functions/variables, UPPER_SNAKE_CASE for constants
- **File Organization**: Components in `components/`, utilities in `utils/`, types in `types/`
- **Imports**: Absolute imports using `@/` prefix, group imports (React, libraries, local)
- **Error Handling**: Always handle errors gracefully with user feedback
- **Accessibility**: ARIA labels, keyboard navigation, focus management, semantic HTML

### Performance Guidelines
- Use React.memo for expensive components
- Implement useCallback for event handlers passed to children
- Use useMemo for expensive calculations
- Lazy load components where appropriate
- Optimize images and bundle size

### Testing Requirements
- Unit tests for all utility functions
- Component tests for critical user flows
- E2E tests for main application features
- Test coverage should be >80% for utilities
- Use descriptive test names and arrange-act-assert pattern

### Security Best Practices
- Never expose API keys in client code
- Validate all user inputs
- Sanitize data before rendering
- Use HTTPS for all external API calls
- Implement proper CORS handling

### Git Workflow
- Use conventional commits (feat:, fix:, docs:, style:, refactor:, test:)
- Create feature branches from main
- Squash commits before merging
- Include co-author attribution for AI assistance
- Never commit secrets, node_modules, or build artifacts

## Tech Stack
- **Frontend**: React 18 + TypeScript + Tailwind CSS
- **Build Tool**: Vite 4.x
- **Testing**: Vitest + Playwright + Testing Library  
- **Linting**: ESLint + Prettier
- **API Integration**: Google Generative AI (@google/generative-ai)
- **Icons**: Lucide React
- **Package Manager**: npm

## Project Structure
```
src/
├── components/         # React components (tabs, UI elements)
├── contexts/          # React contexts (Theme, Toast, Water, DataPersistence)
├── hooks/             # Custom hooks (useApiKey, useAppSettings, etc.)
├── types/             # TypeScript type definitions
├── utils/             # Utility functions (calculateNutrients, etc.)
├── constants/         # Configuration constants and data
└── styles/            # CSS styles

docs/                  # Project documentation
├── prd.md            # Product Requirements Document
├── workflow-rules.md # Development workflow guidelines
└── feature-improvement-ideas.md

tests/                # Playwright e2e tests
xnotes/              # Development notes and planning
```

## Development Commands
```bash
# Development
npm start              # Start dev server (Vite) on port 3002
npm run build          # Production build
npm run preview        # Preview production build

# Testing
npm test              # Run unit tests (Vitest)
npm run test:ui       # Run tests with UI
npm run test:coverage # Test coverage report
npx playwright test   # Run e2e tests

# Code Quality
npm run lint          # ESLint check
npm run lint:fix      # ESLint auto-fix
npm run format        # Prettier formatting
npm run type-check    # TypeScript type checking
```

## Development Tools & Helpers

### Quick Analysis Commands
```bash
# Project structure analysis
find src -name "*.tsx" -o -name "*.ts" | head -20
wc -l src/**/*.{ts,tsx}

# Find components without proper TypeScript types
grep -r "any\|unknown" src/ --include="*.ts" --include="*.tsx"

# Check for accessibility issues
grep -r "onClick.*div\|onClick.*span" src/ --include="*.tsx"

# Find hardcoded colors (should use design system)
grep -r "#[0-9a-fA-F]\{6\}\|rgb(\|rgba(" src/ --include="*.tsx" --include="*.css"

# Check for console.log statements (should be removed in production)
grep -r "console\." src/ --include="*.ts" --include="*.tsx"
```

### Component Generation Template
```bash
# Create new component structure
mkdir src/components/NewComponent
cat > src/components/NewComponent/index.tsx << 'EOF'
import React from 'react';
import { NewComponentProps } from './types';

const NewComponent: React.FC<NewComponentProps> = ({ 
  ...props 
}) => {
  return (
    <div className="new-component">
      {/* Component content */}
    </div>
  );
};

export default NewComponent;
EOF
```

### Useful Development Snippets
```typescript
// Error boundary wrapper
const withErrorBoundary = (Component: React.ComponentType) => {
  return (props: any) => (
    <ErrorBoundary>
      <Component {...props} />
    </ErrorBoundary>
  );
};

// Custom hook template
const useCustomHook = (dependency: any) => {
  const [state, setState] = useState(null);
  
  useEffect(() => {
    // Effect logic
  }, [dependency]);
  
  return { state, setState };
};

// API error handling
const handleApiError = (error: Error, showToast: (msg: string) => void) => {
  console.error('API Error:', error);
  showToast(error.message || 'Ein Fehler ist aufgetreten');
};
```

## Key Features & Components

### Core Components
- **SetupTab**: Wassermenge, Wachstumsphase, Wassertyp-Konfiguration
- **FertilizerTab**: Düngemittel-Datenbank-Management (CRUD)
- **AnalysisTab**: Nährstoff-Visualisierung und Optimierungsempfehlungen
- **DetailsTab**: Mikronährstoff-Details anzeigen
- **MixingAssistant**: Interaktiver Misch-Assistent mit Live-Berechnungen
- **WateringScheduler**: Gießplan-Management
- **ChatBar**: AI-Helfer Integration (Google Gemini)

### State Management
- **ThemeContext**: Dark/Light Mode
- **ToastContext**: Benachrichtigungen
- **WaterContext**: Wasser-Mischungen management
- **DataPersistenceContext**: LocalStorage data persistence

### Key Utilities
- **calculateNutrients.ts**: Hauptberechungslogik für NPK, EC, pH
- **nutrientUtils.ts**: Hilfsfunktionen für Nährstoff-Berechnungen
- **memoization.ts**: Performance-Optimierung für Berechnungen

## Important Files
- `package.json`: Dependencies und Scripts
- `vite.config.js`: Vite-Konfiguration (PWA, Server-Settings)
- `tailwind.config.js`: Tailwind CSS-Konfiguration (Dark Mode)
- `tsconfig.json`: TypeScript-Konfiguration
- `eslint.config.js`: ESLint-Regeln
- `playwright.config.ts`: E2E-Test-Konfiguration

## Code Quality Standards
- **TypeScript**: Strict mode, no unused locals/parameters
- **ESLint**: React hooks rules, TypeScript integration
- **Prettier**: Consistent code formatting
- **Component Structure**: Functional components with hooks
- **Naming Conventions**: PascalCase für Components, camelCase für functions/variables

## API Integration
- **Google Gemini API**: AI chat assistant mit contextual data
- **LocalStorage**: User data persistence (custom fertilizers, settings)
- **JSON Import/Export**: Recipe und setup management

## Development Notes
- **Port**: Development server läuft auf Port 3002
- **Responsive**: Mobile-first approach mit Tailwind breakpoints
- **Accessibility**: ARIA labels, keyboard navigation support
- **PWA**: Progressive Web App features konfiguriert
- **Testing**: Unit tests für utilities, E2E tests für user flows

## Current Development Phase
**UI Redesign**: 6-phase plan zur Verbesserung von UX/Accessibility
- Phase 1: Foundation (color palette, component consistency) - **IN PROGRESS**
- Phase 2: Navigation redesign
- Phase 3: Micro-interactions & feedback
- Phase 4: Accessibility enhancements
- Phase 5: Help & documentation
- Phase 6: Testing & rollout

## Common Tasks
- **Adding new fertilizers**: Update `constants/index.ts` BASE_FERTILIZER_DATABASE
- **New components**: Follow existing patterns in `components/` folder
- **State updates**: Use appropriate context or local state
- **Testing**: Add unit tests for utilities, update E2E tests for new features
- **Styling**: Use Tailwind classes, follow existing responsive patterns

## Code Quality Checks

### Pre-Development Checklist
- [ ] Run `npm run type-check` - no TypeScript errors
- [ ] Run `npm run lint` - no ESLint warnings/errors  
- [ ] Run `npm test` - all tests passing
- [ ] Check `grep -r "console\." src/` - no console.log statements
- [ ] Check accessibility with screen reader testing

### Pre-Commit Checklist  
- [ ] All new components have proper TypeScript interfaces
- [ ] New utility functions have unit tests
- [ ] No hardcoded colors (use design system)
- [ ] Proper error handling implemented
- [ ] ARIA labels on interactive elements
- [ ] No security issues (API keys, XSS vulnerabilities)

## Performance Monitoring

### Key Metrics to Track
- Bundle size: `npm run build && ls -la dist/assets/`
- Lighthouse scores: Accessibility, Performance, SEO
- Core Web Vitals: LCP, FID, CLS
- Memory usage in Chrome DevTools

### Optimization Strategies
```bash
# Bundle analysis
npm install --save-dev rollup-plugin-visualizer
# Add to vite.config.js for bundle analysis

# Check for large dependencies
npx depcheck --detailed

# Performance profiling in dev
npm start -- --profile
```

## Testing Strategy

### Test Coverage Requirements
- Utilities: >90% coverage
- Components: >80% coverage  
- E2E: Critical user flows covered
- Accessibility: All interactive elements tested

### Test Commands
```bash
# Run specific test file
npm test -- Button.test.tsx

# Watch mode for development
npm test -- --watch

# Coverage report
npm run test:coverage

# E2E tests in headed mode  
npx playwright test --headed

# Debug specific E2E test
npx playwright test --debug basic.spec.ts
```

## Debugging & Troubleshooting

### Common Issues
- **Dev Server Issues**: Check Node.js version (requires 20.19.0+ for Vite 7.x)
- **Build Errors**: Run `npm run type-check` für TypeScript issues
- **Dependency Problems**: Clear `node_modules` and `package-lock.json`, reinstall
- **Test Failures**: Check `test-results/` folder für detailed error context
- **Slow Performance**: Check React DevTools Profiler, reduce re-renders

### Debug Commands
```bash
# Detailed dependency tree
npm ls --depth=2

# Check for duplicate packages
npm ls --depth=0 | grep -E "WARN|ERROR"

# Bundle analysis
npm run build && npx vite-bundle-analyzer dist

# Type checking with verbose output
npx tsc --noEmit --pretty

# ESLint with specific rules
npx eslint src/ --ext .ts,.tsx --rule "no-console: error"
```

## Security Considerations
- **API Keys**: Google Gemini API key management via settings, never in code
- **LocalStorage**: Keine sensitive Daten client-side speichern
- **CORS**: Vite dev server configured für external API calls
- **Input Validation**: All user inputs validated and sanitized
- **XSS Prevention**: Use proper React patterns, avoid dangerouslySetInnerHTML
- **Dependency Security**: Run `npm audit` regularly, update dependencies

## Deployment Checklist
- [ ] All tests passing (`npm test && npx playwright test`)
- [ ] Type checking clean (`npm run type-check`)
- [ ] Linting clean (`npm run lint`)
- [ ] No console.log statements in production code
- [ ] Environment variables properly configured
- [ ] Bundle size optimized (check dist/ folder size)
- [ ] Performance audit with Lighthouse
- [ ] Security audit (`npm audit`)
- [ ] Accessibility testing completed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/powerbauer1337)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/powerbauer1337)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
