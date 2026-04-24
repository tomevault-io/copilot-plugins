## pawsitive

> pawsitive is a web application designed to help pet owners manage and enhance the well-being of their pets. The app offers features for tracking pet health, scheduling reminders, and accessing helpful resources for pet care.

# Pawsitive

## Project Overview
pawsitive is a web application designed to help pet owners manage and enhance the well-being of their pets. The app offers features for tracking pet health, scheduling reminders, and accessing helpful resources for pet care.

## Technology Stack
- **Framework**: Vite
- **Language**: TypeScript
- **Runtime**: React 18.3.1
- **Styling**: Tailwind CSS
- **UI Components**: Radix UI + shadcn/ui
- **State Management**: TanStack Query
- **Routing**: React Router
- **Form Management**: React Hook Form + Zod validation
- **Build Tool**: Vite
- **Deployment**: GitHub Pages

## Key Features
- Modern responsive web application
- TypeScript for type safety
- Accessible UI components with Radix UI
- Advanced form handling and validation
- Efficient data fetching and caching
- Interactive data visualization
- Dark/light theme switching

## Core Dependencies
- **UI Framework**: Radix UI component suite
- **Styling**: class-variance-authority, clsx, tailwind-merge
- **Forms**: react-hook-form with Zod validation
- **Data Fetching**: TanStack Query for server state
- **Recharts**: Data visualization and charts
- **Date Fns**: Date manipulation utilities
- **Lucide React**: Icon library
- **Next Themes**: Theme management
- **Sonner**: Toast notifications
- **Cmdk**: Command palette
- **Vaul**: Mobile-optimized modals

## Development Commands
```bash
# Development server
npm run dev

# Production build
npm run build

# Lint code
npm run lint

# Preview production build
npm run preview

# Deploy to GitHub Pages
npm run deploy

```

## Project Structure
```
pawsitive/
├── src/                     # Source code
├── public/                  # Static assets
├── vite.config.ts           # Configuration
├── tailwind.config.ts       # Configuration
├── tsconfig.json            # Configuration
└── package.json             # Dependencies
```

## Deployment
- **Platform**: GitHub Pages
- **URL**: 
- **Build**: Static site generation
- **CI/CD**: Automated deployment via gh-pages

## Development Notes
- Uses Vite for fast development and optimized builds
- Implements comprehensive form validation with Zod
- Includes accessibility features through Radix UI
- Supports dark/light theme switching

## Vite Features
- **Fast HMR**: Hot module replacement
- **Lightning Dev Server**: Instant startup
- **Optimized Builds**: Production optimization
- **Plugin Ecosystem**: Rich plugin support
- **Modern JavaScript**: ES modules support

## Testing & Quality
- TypeScript for type safety
- ESLint for code quality

## Future Enhancements
- Performance optimizations
- Advanced analytics integration
- Enhanced user experience features
- API integrations
- Mobile app development



## 🧭 Claude Code Navigation

### Quick Commands
**Development Scripts:**
- `dev`: vite
- `build`: vite build
- `lint`: eslint .
- `deploy`: gh-pages -d dist

**Key Files:**
- `package.json` - Dependencies and scripts configuration
- `README.md` - Project documentation and setup guide
- `CLAUDE.md` - Comprehensive development guide for Claude
- `vite.config.ts` - Vite build tool configuration
- `tailwind.config.ts` - Tailwind CSS styling configuration
- `tsconfig.json` - TypeScript compiler configuration
- `components.json` - shadcn/ui components configuration

**Key Directories:**
- `src/` - Source code and main application logic
- `public/` - Static assets (images, icons, etc.)

**Claude Code Files:**
- `.claude/project-context.md` - Project overview and structure
- `.claude/coding-standards.md` - Development guidelines and patterns
- `.claude/commands/` - Custom Claude commands for common tasks
- `.claude/context/` - Domain-specific development context


### Quick Reference

**Common Tasks:**
- Start development: `npm run dev` or `bun dev`
- Build project: `npm run build` or `bun build`
- Lint code: `npm run lint` or `bun lint`
- Deploy: `npm run deploy` or `bun deploy`

**File Patterns:**
- Components: `components/**/*.tsx`
- Pages: `app/**/*.tsx` or `pages/**/*.tsx`
- Utilities: `lib/**/*.ts`
- Styles: `**/*.css` or use Tailwind classes
- Tests: `**/*.test.ts` or `**/*.spec.ts`

**Development Tips:**
- Use TypeScript for type safety
- Follow existing component patterns
- Utilize shadcn/ui components
- Implement responsive design with Tailwind
- Test changes before committing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/igor-kan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
