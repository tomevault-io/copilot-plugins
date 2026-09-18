## setup-migration

> Create an automated setup process that handles all configuration:

# Setup & Migration Guide for Zen Flow UI

## One-Command Setup

### Quick Start Script
Create an automated setup process that handles all configuration:

```bash
# Single command to set up zen-flow-ui in any React project
npx @gurusharan3107/zen-flow-ui init
```

This command should:
1. Install all required dependencies
2. Configure Tailwind CSS with zen-flow design tokens
3. Set up the utility functions
4. Create example components
5. Add TypeScript configuration if needed

### CLI Tool Enhancement
Enhance the existing [cli/index.cjs](mdc:cli/index.cjs) to support full project initialization:

```javascript
#!/usr/bin/env node

const commands = {
  init: initProject,
  add: addComponent,
  update: updateComponents,
  migrate: runMigration,
  doctor: runDiagnostics
};

async function initProject(options = {}) {
  // Detect project type (Vite, Next.js, CRA, etc.)
  // Install dependencies
  // Configure files
  // Setup examples
}
```

## Installation Methods

### Method 1: Automated Setup (Recommended)
```bash
# For new projects
npx create-react-app my-app --template typescript
cd my-app
npx @gurusharan3107/zen-flow-ui init

# For existing projects
npx @gurusharan3107/zen-flow-ui init
```

### Method 2: Manual Installation
```bash
# Install core package
npm install @gurusharan3107/zen-flow-ui

# Install peer dependencies
npm install react react-dom class-variance-authority clsx tailwind-merge gsap

# Install dev dependencies for TypeScript projects
npm install -D @types/react @types/react-dom @types/gsap
```

### Method 3: Yarn/PNPM Support
```bash
# Yarn
yarn add @gurusharan3107/zen-flow-ui
yarn add react react-dom class-variance-authority clsx tailwind-merge gsap

# PNPM
pnpm add @gurusharan3107/zen-flow-ui
pnpm add react react-dom class-variance-authority clsx tailwind-merge gsap
```

## Automated Configuration

### Package.json Scripts
The CLI tool should add helpful scripts to package.json:
```json
{
  "scripts": {
    "zen:add": "npx @gurusharan3107/zen-flow-ui add",
    "zen:update": "npx @gurusharan3107/zen-flow-ui update",
    "zen:doctor": "npx @gurusharan3107/zen-flow-ui doctor"
  }
}
```

### Tailwind CSS Auto-Configuration
Create [scripts/setup-tailwind.js](mdc:scripts/setup-tailwind.js) that automatically configures Tailwind:

```javascript
export const setupTailwind = async (projectRoot) => {
  const tailwindConfig = `
module.exports = {
  content: [
    "./src/**/*.{js,ts,jsx,tsx}",
    "./node_modules/@gurusharan3107/zen-flow-ui/dist/**/*.js"
  ],
  theme: {
    extend: {
      colors: {
        'zen-void': '#0a0a0a',
        'zen-ink': '#1a1a1a',
        'zen-shadow': '#2a2a2a',
        'zen-stone': '#4a4a4a',
        'zen-mist': '#8a8a8a',
        'zen-cloud': '#dadada',
        'zen-paper': '#fafafa',
        'zen-light': '#ffffff',
        'zen-accent': '#ff4757',
        'zen-water': '#3742fa',
        'zen-leaf': '#26de81',
        'zen-sun': '#fed330',
      },
      spacing: {
        'zen-xs': '4px',
        'zen-sm': '8px',
        'zen-md': '16px',
        'zen-lg': '24px',
        'zen-xl': '32px',
        'zen-2xl': '48px',
        'zen-3xl': '64px',
      },
      borderRadius: {
        'zen-sm': '4px',
        'zen-md': '8px',
        'zen-lg': '12px',
        'zen-xl': '16px',
      },
      animation: {
        'zen-shimmer': 'shimmer 2s linear infinite',
        'zen-accordion-down': 'accordion-down 0.2s ease-out',
        'zen-accordion-up': 'accordion-up 0.2s ease-out',
        'zen-fade-in': 'fade-in 0.3s ease-out',
        'zen-slide-up': 'slide-up 0.3s ease-out',
      },
      keyframes: {
        'shimmer': {
          '0%': { transform: 'translateX(-100%)' },
          '100%': { transform: 'translateX(100%)' },
        },
        'accordion-down': {
          from: { height: 0 },
          to: { height: 'var(--radix-accordion-content-height)' },
        },
        'accordion-up': {
          from: { height: 'var(--radix-accordion-content-height)' },
          to: { height: 0 },
        },
        'fade-in': {
          from: { opacity: 0 },
          to: { opacity: 1 },
        },
        'slide-up': {
          from: { opacity: 0, transform: 'translateY(10px)' },
          to: { opacity: 1, transform: 'translateY(0)' },
        },
      },
    },
  },
  plugins: [],
};`;

  await writeFile(path.join(projectRoot, 'tailwind.config.js'), tailwindConfig);
};
```

### CSS Setup Automation
Auto-create [src/lib/utils.ts](mdc:src/lib/utils.ts):
```typescript
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

// Utility for reduced motion
export function useReducedMotion() {
  if (typeof window === 'undefined') return false;
  return window.matchMedia('(prefers-reduced-motion: reduce)').matches;
}

// Theme utilities
export const zenColors = {
  void: '#0a0a0a',
  ink: '#1a1a1a',
  shadow: '#2a2a2a',
  stone: '#4a4a4a',
  mist: '#8a8a8a',
  cloud: '#dadada',
  paper: '#fafafa',
  light: '#ffffff',
  accent: '#ff4757',
  water: '#3742fa',
  leaf: '#26de81',
  sun: '#fed330',
};
```

## Project Type Detection

### Framework-Specific Setup
Detect and configure for different React frameworks:

```javascript
// scripts/detect-framework.js
export const detectFramework = () => {
  const packageJson = JSON.parse(fs.readFileSync('package.json', 'utf8'));
  
  if (packageJson.dependencies?.['next']) return 'nextjs';
  if (packageJson.dependencies?.['vite']) return 'vite';
  if (packageJson.dependencies?.['react-scripts']) return 'cra';
  if (packageJson.dependencies?.['gatsby']) return 'gatsby';
  
  return 'custom';
};
```

### Next.js Configuration
```javascript
// For Next.js projects
const nextConfigPath = path.join(projectRoot, 'next.config.js');
const nextConfig = `
/** @type {import('next').NextConfig} */
const nextConfig = {
  experimental: {
    appDir: true,
  },
  transpilePackages: ['@gurusharan3107/zen-flow-ui'],
};

module.exports = nextConfig;
`;
```

### Vite Configuration
```javascript
// For Vite projects
const viteConfigPath = path.join(projectRoot, 'vite.config.ts');
const viteConfig = `
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
    },
  },
})
`;
```

## Migration System

### Version Migration Scripts
Create migration scripts for major version updates:

```javascript
// migrations/v2-to-v3.js
export const migrateV2ToV3 = async (projectRoot) => {
  const migrations = [
    {
      name: 'Update import paths',
      run: updateImportPaths,
    },
    {
      name: 'Migrate Framer Motion to GSAP',
      run: migrateAnimations,
    },
    {
      name: 'Update component props',
      run: updateComponentProps,
    },
  ];
  
  for (const migration of migrations) {
    console.log(`Running: ${migration.name}`);
    await migration.run(projectRoot);
  }
};
```

### Automatic Code Transformation
Use AST transformation for automatic migration:

```javascript
import { transform } from 'jscodeshift';

const transformImports = (source) => {
  return transform(source, {
    // Transform old imports to new structure
    'ImportDeclaration[source.value="zen-flow-ui"]': (path) => {
      path.get('source').replaceWith(
        j.literal('@gurusharan3107/zen-flow-ui')
      );
    },
  });
};
```

### Dependency Updates
```javascript
export const updateDependencies = async (projectRoot) => {
  const packageJsonPath = path.join(projectRoot, 'package.json');
  const packageJson = JSON.parse(fs.readFileSync(packageJsonPath, 'utf8'));
  
  // Remove old dependencies
  delete packageJson.dependencies['framer-motion'];
  
  // Add new dependencies
  packageJson.dependencies['gsap'] = '^3.12.2';
  
  // Update zen-flow-ui to latest
  packageJson.dependencies['@gurusharan3107/zen-flow-ui'] = '^3.0.0';
  
  fs.writeFileSync(packageJsonPath, JSON.stringify(packageJson, null, 2));
};
```

## Health Check System

### Project Diagnostics
Create a health check command that verifies setup:

```javascript
// scripts/doctor.js
export const runDiagnostics = async () => {
  const checks = [
    checkDependencies,
    checkTailwindConfig,
    checkUtilsFile,
    checkImportPaths,
    checkComponentUsage,
    checkAccessibility,
    checkPerformance,
  ];
  
  const results = [];
  
  for (const check of checks) {
    const result = await check();
    results.push(result);
    
    if (result.status === 'error') {
      console.error(`❌ ${result.name}: ${result.message}`);
      if (result.fix) {
        console.log(`💡 Fix: ${result.fix}`);
      }
    } else if (result.status === 'warning') {
      console.warn(`⚠️  ${result.name}: ${result.message}`);
    } else {
      console.log(`✅ ${result.name}: OK`);
    }
  }
  
  return results;
};
```

### Automated Fixes
```javascript
const checkTailwindConfig = async () => {
  const configPath = 'tailwind.config.js';
  
  if (!fs.existsSync(configPath)) {
    return {
      name: 'Tailwind Configuration',
      status: 'error',
      message: 'Tailwind config not found',
      fix: 'Run: npx @gurusharan3107/zen-flow-ui init --fix-tailwind',
      autoFix: () => setupTailwind(process.cwd())
    };
  }
  
  const config = fs.readFileSync(configPath, 'utf8');
  if (!config.includes('zen-flow-ui')) {
    return {
      name: 'Tailwind Configuration',
      status: 'warning',
      message: 'Zen Flow UI not configured in Tailwind',
      fix: 'Add zen-flow-ui to content array',
      autoFix: () => updateTailwindConfig(configPath)
    };
  }
  
  return {
    name: 'Tailwind Configuration',
    status: 'success',
    message: 'Properly configured'
  };
};
```

## IDE Integration

### VSCode Extensions
Create a VSCode extension that provides:
- Component snippets
- Auto-completion for zen-flow classes
- Real-time design token preview
- Accessibility linting

### TypeScript Support
```json
// .vscode/settings.json (auto-generated)
{
  "typescript.preferences.includePackageJsonAutoImports": "on",
  "editor.quickSuggestions": {
    "strings": true
  },
  "tailwindCSS.classAttributes": [
    "class",
    "className",
    "ngClass"
  ],
  "tailwindCSS.experimental.classRegex": [
    ["cn\\(([^)]*)\\)", "'([^']*)'"],
    ["cva\\(([^)]*)\\)", "'([^']*)'"]
  ]
}
```

### IntelliSense Configuration
```typescript
// types/zen-flow-ui.d.ts (auto-generated)
declare module '@gurusharan3107/zen-flow-ui' {
  export interface ZenFlowTheme {
    colors: {
      'zen-void': string;
      'zen-ink': string;
      // ... all theme colors
    };
    spacing: {
      'zen-xs': string;
      'zen-sm': string;
      // ... all spacing values
    };
  }
}
```

## Example Generation

### Starter Templates
Create different starter templates based on use case:

```javascript
const templates = {
  'dashboard': {
    components: ['DataTable', 'Card', 'Button', 'Select', 'Modal'],
    pages: ['Dashboard', 'Analytics', 'Settings'],
    features: ['Dark mode', 'Responsive design', 'Data visualization']
  },
  'landing-page': {
    components: ['Button', 'Card', 'Accordion', 'Modal'],
    pages: ['Home', 'About', 'Contact'],
    features: ['Smooth scrolling', 'Animations', 'Contact forms']
  },
  'admin-panel': {
    components: ['DataTable', 'Form', 'Navigation', 'Breadcrumb'],
    pages: ['Users', 'Content', 'Reports', 'Settings'],
    features: ['CRUD operations', 'Permissions', 'Bulk actions']
  }
};
```

### Component Playground
Generate an interactive playground:

```javascript
// scripts/generate-playground.js
export const generatePlayground = async () => {
  const playgroundPath = 'src/zen-flow-playground';
  
  // Create playground directory
  await fs.mkdir(playgroundPath, { recursive: true });
  
  // Generate component examples
  const components = await getAvailableComponents();
  
  for (const component of components) {
    await generateComponentExample(component, playgroundPath);
  }
  
  // Generate index page
  await generatePlaygroundIndex(components, playgroundPath);
};
```

## Migration Checklist

### Pre-Migration Assessment
```javascript
export const assessProject = async () => {
  return {
    framework: detectFramework(),
    zenFlowVersion: getCurrentVersion(),
    dependencies: getDependencies(),
    components: getUsedComponents(),
    customizations: getCustomizations(),
    risks: assessMigrationRisks(),
  };
};
```

### Migration Steps
1. **Backup Project**
   ```bash
   git commit -am "Pre-zen-flow-ui migration backup"
   git tag "pre-migration-$(date +%Y%m%d)"
   ```

2. **Run Assessment**
   ```bash
   npx @gurusharan3107/zen-flow-ui migrate --assess
   ```

3. **Execute Migration**
   ```bash
   npx @gurusharan3107/zen-flow-ui migrate --from=v2 --to=v3
   ```

4. **Verify & Test**
   ```bash
   npx @gurusharan3107/zen-flow-ui doctor
   npm test
   ```

### Rollback Strategy
```javascript
export const rollback = async (backupTag) => {
  console.log(`Rolling back to ${backupTag}...`);
  
  // Restore git state
  execSync(`git reset --hard ${backupTag}`);
  
  // Restore package.json
  execSync('npm install');
  
  console.log('Rollback completed successfully');
};
```

## Documentation Generation

### Auto-Generated Setup Docs
Create project-specific documentation:

```javascript
export const generateDocs = async (projectRoot) => {
  const config = analyzeProject(projectRoot);
  
  const readme = `
# ${config.projectName} - Zen Flow UI Setup

## Installation Completed ✅

Your project has been configured with Zen Flow UI v${config.version}.

### Available Components
${config.components.map(c => `- ${c.name}: ${c.description}`).join('\n')}

### Getting Started
\`\`\`tsx
import { Button, Card } from '@gurusharan3107/zen-flow-ui';

function App() {
  return (
    <Card>
      <Button variant="primary">Hello Zen Flow UI!</Button>
    </Card>
  );
}
\`\`\`

### Next Steps
1. Explore the component playground: \`npm run zen:playground\`
2. Run health check: \`npm run zen:doctor\`
3. Read the full documentation: [zen-flow-ui docs](https://github.com/ingpoc/zen-flow-ui)
`;

  await writeFile(path.join(projectRoot, 'ZEN_FLOW_SETUP.md'), readme);
};
```

## Setup Command Examples

### Basic Setup
```bash
# Initialize zen-flow-ui in current project
npx @gurusharan3107/zen-flow-ui init

# Initialize with specific template
npx @gurusharan3107/zen-flow-ui init --template=dashboard

# Initialize with custom configuration
npx @gurusharan3107/zen-flow-ui init --config=./zen-flow.config.js
```

### Advanced Setup
```bash
# Setup with TypeScript support
npx @gurusharan3107/zen-flow-ui init --typescript

# Setup with custom theme
npx @gurusharan3107/zen-flow-ui init --theme=custom

# Setup with specific components only
npx @gurusharan3107/zen-flow-ui init --components=Button,Card,Input

# Setup with GSAP animations
npx @gurusharan3107/zen-flow-ui init --animations=gsap
```

This comprehensive setup system ensures that developers can get started with zen-flow-ui in seconds, regardless of their project structure or experience level.

---
> Source: [ingpoc/zen-flow-ui](https://github.com/ingpoc/zen-flow-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
