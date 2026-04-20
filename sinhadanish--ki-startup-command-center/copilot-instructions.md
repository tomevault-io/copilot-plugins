## ki-startup-command-center

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Ki's startup command center - a comprehensive development environment for building the world's first Human-AI-Human relationship intelligence platform. This is a lean 3-person startup focused on transforming how couples navigate relationships through AI.

## Development Commands

### Essential Commands (Makefile)
```bash
# Development Commands
make setup     # Complete setup (submodules + Ki platform + automation)
make dev       # Start complete development environment
make ki-setup  # Setup Ki platform specifically  
make stop      # Stop all services
make n8n       # Open n8n automation dashboard (admin/ki2024)
make backup    # Backup everything
make clean     # Clean up everything

# Git Automation Commands (⭐ Claude Code Compatible)
make git-status  # Check status of all submodules and main repository
make git-push    # Push changes in all submodules + main repository
make git-pull    # Pull latest changes from all repositories
```

### Git Automation System
Ki includes comprehensive git automation that handles all submodule repositories automatically. This system is **fully compatible with Claude Code** and eliminates the need for manual submodule management.

**Key Benefits:**
- **Single Command Operations**: One command handles all repositories
- **Claude Code Integration**: Can be executed directly by AI assistants
- **Automated Commit Messages**: Includes timestamps and proper attribution
- **Intelligent Status Reporting**: Shows detailed status across all repositories
- **Error Handling**: Gracefully handles missing submodules and connection issues

### MCP Server Configuration
This project includes comprehensive MCP (Model Context Protocol) server configuration in `.mcp.json` for enhanced Claude Code capabilities:

#### Available MCP Servers
```bash
# View all configured MCP servers for this project
claude mcp list

# Project-scoped MCP servers include:
# - puppeteer: Web automation for competitive analysis and testing
# - filesystem: Enhanced file operations for data room management  
# - github: Repository management across ki-platform, ki-business, ki-automation
# - postgres: Database operations for user analytics and metrics
# - gdrive: Cloud storage integration for investor documents
# - slack: Team communication automation
# - calendar: Meeting scheduling and milestone tracking
# - analytics: Business intelligence and user behavior analysis
```

#### MCP Server Usage
MCP servers are automatically available when working in this project directory and provide:
- **Web Automation**: Competitive analysis, market research, automated testing
- **Database Operations**: User data analysis, metrics tracking, business intelligence
- **File Management**: Advanced document processing, data room organization
- **Team Collaboration**: Automated notifications, meeting scheduling, document sharing
- **Repository Management**: Multi-repo coordination, issue tracking, release management

### Service URLs (when running `make dev`)
- **Ki App**: http://localhost:3001 - Main relationship intelligence platform
- **AI Engine API**: http://localhost:8000 - FastAPI with LangGraph backend
- **Marketing Site**: http://localhost:3000 - Next.js website
- **n8n Automation**: http://localhost:5678 - Business automation workflows

### Testing & Quality (Next Forge Commands)
```bash
# Navigate to Ki platform (Next Forge monorepo)
cd submodules/product/ki-platform

# Core development commands
pnpm dev           # Start all Next.js apps in development mode
pnpm build         # Build all applications for production
pnpm test          # Run Vitest tests across all packages
pnpm lint          # Ultracite linting (Biome-based, faster than ESLint)
pnpm format        # Format code with Ultracite
pnpm typecheck     # TypeScript checking across monorepo
pnpm analyze       # Bundle analysis for optimization
pnpm translate     # Internationalization workflow
pnpm boundaries    # Enforce package boundary rules

# Database operations
pnpm migrate       # Prisma format + generate + push to database

# Package management
pnpm install       # Install dependencies (preferred package manager)
pnpm bump-deps     # Update all dependencies
pnpm bump-ui       # Update shadcn/ui components
pnpm clean         # Clean node_modules across workspace

# Backend (LangGraph AI Engine)
cd apps/langgraph-backend
pytest tests/
python -m pytest tests/ -v
```

### Next Forge Workflow Patterns
```bash
# 1. Start development environment
make dev                    # Starts all services including Next.js apps

# 2. Work on specific applications
cd submodules/product/ki-platform
pnpm dev                   # Development mode for all apps
# Apps run on:
# - Web (marketing): http://localhost:3000
# - App (platform): http://localhost:3001  
# - Docs: http://localhost:3002
# - Storybook: http://localhost:6006

# 3. Add new features following Next Forge patterns
# - Use workspace packages (@repo/*) for shared logic
# - Follow environment variable patterns with keys.ts
# - Implement UI with design-system components
# - Add comprehensive tests with Vitest

# 4. Quality assurance
pnpm lint          # Fast linting with Ultracite
pnpm typecheck     # Ensure type safety
pnpm test          # Run test suite
pnpm analyze       # Check bundle sizes

# 5. Database changes
pnpm migrate       # Handle schema updates safely
```

## Architecture Overview

### Core Innovation: Human-AI-Human Framework
Ki's breakthrough architecture processes both partners simultaneously while maintaining privacy:

```
Partner A → Private Channel → Ki AI Core ← Private Channel ← Partner B
                              ↓
                         Empathy AI Layer
                              ↓
                    Personalized Dual Responses
```

### LangGraph AI Engine
Core relationship intelligence workflow in `submodules/product/ki-platform/apps/langgraph-backend/src/graphs/ki_relationship_graph.py`:

```python
# Processing flow
intake → safety_check → emotional_analysis → conflict_detection 
→ anxiety_detection → empathy_processing → pattern_recognition 
→ response_generation → memory_update
```

**Key Components:**
- **Dual-partner message processing** with separate private channels
- **Emotional intelligence** with empathy AI layer
- **Conflict detection** using Thomas-Kilmann styles
- **Anxiety detection** and safety monitoring
- **Pattern recognition** for relationship dynamics
- **Personalized responses** for each partner

### Technology Stack (Next Forge Architecture)

**Core Framework**: Built on [Next Forge](https://www.next-forge.com/docs) - A production-ready Next.js boilerplate with enterprise-grade tooling

#### Frontend & UI Stack
- **Framework**: Next.js 15 with App Router (React 19.1.0)
- **UI Components**: Radix UI primitives with shadcn/ui design system
- **Styling**: Tailwind CSS 4.1.7 with class-variance-authority
- **Icons**: Lucide React + Radix Icons
- **Forms**: React Hook Form with Hookform Resolvers + Zod validation
- **Typography**: Geist font family
- **Theme**: next-themes for dark/light mode
- **Charts**: Recharts for data visualization
- **Command Menu**: CMDK for search interfaces

#### Backend & Infrastructure
- **AI Engine**: LangGraph + OpenAI GPT-4 + Anthropic Claude
- **Backend API**: FastAPI (Python 3.11+) + Next.js API routes
- **Database**: PostgreSQL with Prisma ORM + Neon serverless
- **Cache/Real-time**: Redis + WebSocket support
- **Authentication**: Clerk with custom themes
- **File Storage**: Integrated cloud storage solutions
- **Rate Limiting**: Built-in rate limiting package

#### Development & DevOps
- **Monorepo**: Turborepo for build orchestration
- **Package Manager**: pnpm 10.11.0
- **TypeScript**: 5.8.3 with strict configuration
- **Linting**: Ultracite (Biome-based) + ESLint
- **Testing**: Vitest for unit/integration tests
- **Environment**: @t3-oss/env-nextjs for type-safe env vars
- **Automation**: n8n workflows for business processes
- **Orchestration**: Docker Compose

#### Production & Monitoring
- **Deployment**: Vercel with edge functions
- **Analytics**: PostHog + Vercel Analytics + Google Analytics
- **Error Tracking**: Sentry with performance monitoring
- **Logging**: Logtail for structured logging
- **Email**: Resend with React Email templates
- **Payments**: Stripe with agent toolkit integration
- **Security**: Arcjet for protection and rate limiting

## Repository Structure

### GitHub Repository Setup
**Main Repository**: [ki-startup-command-center](https://github.com/sinhadanish/ki-startup-command-center)
- Master repository containing data room, planning, operations, and configuration
- Includes submodule configuration for modular development

**Submodule Repositories**:
- **[ki-platform](https://github.com/sinhadanish/ki-platform)** - Main Ki platform (monorepo)
- **[ki-business](https://github.com/sinhadanish/ki-business)** - Customer development & fundraising
- **[ki-automation](https://github.com/sinhadanish/ki-automation)** - Business automation workflows

### Git Commands for Repository Management

#### Automated Git Operations (Recommended)
```bash
# Check status across all repositories
make git-status

# Push all changes (submodules + main repo)
make git-push

# Pull latest from all repositories  
make git-pull
```

#### Manual Git Commands
```bash
# Clone with all submodules
git clone --recurse-submodules https://github.com/sinhadanish/ki-startup-command-center.git

# Initialize submodules in existing clone
git submodule update --init --recursive

# Update all submodules to latest
git submodule update --remote

# Push changes to main repository
git add . && git commit -m "Update description" && git push origin main
```

#### Direct Script Execution (Claude Code Compatible)
```bash
# For Claude Code or direct execution
./scripts/git-submodule-status.sh   # Check all repository status
./scripts/git-submodule-push.sh     # Push all changes
./scripts/git-submodule-pull.sh     # Pull all updates
```

#### Git Automation Features
**✅ Comprehensive Automation:**
- Handles all 3 submodule repositories + main repository
- Automated commit messages with timestamps and Claude Code attribution
- Colorized output for easy status reading
- Error handling for missing submodules
- Status reporting (changes, commits ahead/behind, branch info)

**✅ Claude Code Integration:**
Claude Code can execute these commands directly:
```bash
cd "/path/to/startup-command-center" && make git-push
cd "/path/to/startup-command-center" && ./scripts/git-submodule-push.sh
```

**✅ Eliminates Manual Submodule Management:**
- ❌ Old way: 3 separate git commands per submodule
- ✅ New way: Single `make git-push` command
- Automatically handles submodule reference updates in main repository

**✅ Smart Status Checking:**
- Shows working tree status across all repositories
- Displays branch information and remote sync status
- Lists recent commits and change summaries
- Identifies which repositories need attention

### Submodules Architecture
```
submodules/
├── product/ki-platform/          # Main Ki platform (Next.js Turborepo monorepo)
│   ├── apps/web/                 # Marketing website (Next.js 15)
│   ├── apps/app/                 # Ki relationship app (Next.js 15)
│   ├── apps/docs/                # Documentation site (Next.js 15)
│   ├── apps/langgraph-backend/   # LangGraph AI engine (Python + FastAPI)
│   ├── packages/ui/              # Shared React component library
│   ├── packages/eslint-config/   # Shared ESLint configurations
│   └── packages/typescript-config/ # Shared TypeScript configurations
├── business/                     # Complete business operations
│   ├── data-room/               # Investor documentation for $1.5M pre-seed
│   ├── planning/                # MVP roadmap and weekly goals
│   ├── operations/              # Team sync, metrics, automation
│   ├── growth/                  # Customer development, market research
│   ├── customer-research/       # User interviews and validation
│   └── fundraising-prep/        # Investment materials and prep
└── automation/n8n-workflows/    # Business automation workflows
```

### Key Directories
- **`submodules/business/data-room/`**: Complete investor documentation for $1.5M pre-seed
- **`submodules/business/planning/`**: MVP roadmap and weekly goals
- **`submodules/business/operations/`**: Team sync, metrics, automation
- **`submodules/business/growth/`**: Customer development, market research
- **`config/`**: Environment and service configuration
- **`scripts/`**: Development and deployment scripts

## Development Guidelines

### Next Forge Architecture Patterns

#### Environment Variables (Type-Safe)
- **Composable System**: Each package defines environment variables in `keys.ts` files
- **Type Safety**: Uses @t3-oss/env-nextjs for runtime validation and autocompletion
- **Structure**: Server/client separation with Zod validation schemas
- **Files to Update**: When adding env vars, update both `.env.local` files and relevant `keys.ts`

```typescript
// Example: packages/auth/keys.ts
export const keys = () => createEnv({
  server: {
    CLERK_SECRET_KEY: z.string().min(1),
  },
  client: {
    NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY: z.string().min(1),
  },
  runtimeEnv: process.env,
});
```

#### Package Architecture
- **Modular Design**: Each feature is a workspace package (e.g., `@repo/auth`, `@repo/database`)
- **Shared Dependencies**: Common TypeScript configs, UI components, and utilities
- **Clean Imports**: Use workspace aliases (`@repo/*`) for internal packages
- **Separation of Concerns**: Auth, database, email, payments, etc. are isolated packages

#### UI Component Development
- **Design System**: Use `@repo/design-system` with Radix UI primitives
- **Styling**: Tailwind CSS with class-variance-authority for component variants
- **Theming**: Built-in dark/light mode support with next-themes
- **Accessibility**: Radix UI ensures WCAG compliance out of the box
- **Typography**: Geist font family for consistent branding

#### Data Layer Best Practices
- **ORM**: Prisma with type-safe database operations
- **Validation**: Zod schemas for runtime type checking
- **Migrations**: Use `pnpm migrate` for database schema updates
- **Connection**: Neon serverless PostgreSQL with connection pooling

### Privacy-First Development
- All relationship data is highly sensitive and encrypted
- Maintain strict separation between partners' private data channels
- Follow HIPAA compliance patterns for healthcare-grade security
- Never log or expose personal relationship content
- **Security Package**: Use `@repo/security` for consistent protection patterns

### AI/ML Best Practices
- **Emotional Intelligence**: UI adapts to detected emotional states
- **Pattern Recognition**: Identify dynamics without clinical labeling
- **Voice-First**: Seamless voice-visual interaction for intimate conversations
- **Real-time Processing**: <2 second response times for relationship support
- **Error Handling**: Comprehensive Sentry integration for AI pipeline monitoring

### Code Quality Standards
- **Testing**: Vitest for unit/integration tests covering relationship scenarios
- **Privacy**: Verify partner data separation in all features
- **Performance**: Voice response times under 100ms with performance monitoring
- **Empathy**: Validate insight delivery and emotional responsiveness
- **Linting**: Ultracite (Biome-based) for fast, consistent code formatting
- **Type Safety**: Strict TypeScript configuration across all packages

## Business Context

### Stage: Pre-seed startup building relationship intelligence platform
- **Market**: $50B relationship support market
- **Innovation**: First Human-AI-Human framework with 2-3 year technical moat
- **Target**: Therapy-priced-out couples (25M US couples, $30-60K income)

### Current Development Phase (Weeks 1-8)
- LangGraph AI engine with empathy processing ✅
- Conversational interface with breathing design ✅
- Pattern recognition for relationship dynamics (in progress)
- Partner integration for shared conversations (next)

## Key Files to Understand

### Core AI Implementation
- `submodules/product/ki-platform/apps/langgraph-backend/src/graphs/ki_relationship_graph.py` - Main relationship intelligence workflow
- `submodules/product/ki-platform/apps/langgraph-backend/src/main.py` - FastAPI application entry
- `docker-compose.yml` - Complete service orchestration

### Product Documentation
- `submodules/business/data-room/01-product-intelligence/` - Complete product intelligence documentation
- `submodules/business/data-room/01-product-intelligence/human-ai-human-framework.md` - Core technical innovation
- `submodules/business/data-room/01-product-intelligence/ki-conversational-system.md` - AI conversation framework

### Business Intelligence
- `submodules/business/data-room/00-executive-summary/ki-executive-summary.md` - Investment thesis
- `submodules/business/planning/mvp-roadmap.md` - 20-week development roadmap
- `submodules/business/operations/` - Team workflows and metrics

## n8n Business Automation

Ki uses n8n extensively for business process automation:
- **Customer Interview Pipeline**: Typeform → Airtable → Calendar → Slack
- **User Onboarding**: App signup → Database → Welcome sequence
- **Team Metrics**: Auto-collect data → Generate insights → Team reports

Access n8n at http://localhost:5678 (admin/ki2024) after running `make dev`.

## Data Privacy & Ethics

### Critical Privacy Requirements
- End-to-end encryption for all partner communications
- Separate private channels with secure shared context only by consent
- Zero-knowledge architecture where possible
- HIPAA-compliant data handling

### Ethical AI Guidelines
- No psychological diagnosis or clinical labeling
- Strengths-based pattern recognition, not deficit-focused
- Professional therapy referral for serious safety concerns
- Transparent AI decision-making with user control

## Git Automation Examples

### For Claude Code Usage
Claude Code can execute these commands directly in any conversation:

```bash
# Check status of all repositories
cd "/Users/snapsprint/Documents/Ki Master Folder/startup-command-center" && make git-status

# Push all changes across repositories  
cd "/Users/snapsprint/Documents/Ki Master Folder/startup-command-center" && make git-push

# Pull latest updates from all repositories
cd "/Users/snapsprint/Documents/Ki Master Folder/startup-command-center" && make git-pull
```

### Typical Development Workflow
```bash
# 1. Start development work
make dev

# 2. Make changes across multiple repositories
# ... development work ...

# 3. Check what's changed across all repos
make git-status

# 4. Push all changes with one command
make git-push

# 5. Pull latest team updates
make git-pull
```

### Script Output Examples
The automation provides detailed, colorized output:
- 📊 **Status Reports**: Shows branch, changes, commits ahead/behind
- ⬆️ **Push Operations**: Handles each submodule + main repo automatically  
- ⬇️ **Pull Operations**: Updates all repositories with latest changes
- 🎉 **Success Summaries**: Confirms all operations completed successfully

## Next Forge Implementation Guidelines

### Key Next Forge Principles for Development

#### 1. Environment Variable Management
Always use the composable environment system:
```bash
# When adding new environment variables:
# 1. Add to relevant .env.local files
# 2. Update corresponding keys.ts file with Zod validation
# 3. Import and extend in app's env.ts file
```

#### 2. Package-First Development
- Create new features as workspace packages when they're reusable
- Use `@repo/*` imports for internal packages
- Follow the established patterns in `packages/` directory
- Keep apps thin, packages thick

#### 3. UI Development Best Practices
```typescript
// Use design system components
import { Button } from '@repo/design-system/button'
import { Card } from '@repo/design-system/card'

// Follow variant patterns with class-variance-authority
const buttonVariants = cva('base-styles', {
  variants: {
    variant: { default: '...', destructive: '...' },
    size: { default: '...', sm: '...' },
  }
})
```

#### 4. Data Layer Patterns
```typescript
// Use Prisma with type safety
import { db } from '@repo/database'
import { z } from 'zod'

// Define schemas with Zod
const userSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
})

// Type-safe database operations
const users = await db.user.findMany({
  where: { active: true }
})
```

#### 5. Testing Strategy
- Use Vitest for fast unit and integration tests
- Test component behavior, not implementation
- Mock external services and APIs
- Focus on user interactions and data flows

#### 6. Performance Optimization
- Use `pnpm analyze` to monitor bundle sizes
- Implement proper code splitting
- Optimize images and assets
- Monitor Core Web Vitals with Vercel Analytics

This command center enables rapid iteration by a small team through extensive automation, sophisticated AI capabilities, comprehensive git management, complete business documentation for fundraising, and enterprise-grade Next Forge architecture patterns.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sinhadanish) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
