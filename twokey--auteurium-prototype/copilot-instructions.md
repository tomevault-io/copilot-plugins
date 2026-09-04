## auteurium-prototype

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Never mention claude code in git commit messages.
Never try to create git commit without me asking so.

## Project Overview

**Auteurium** is a web application for creating text snippets and organizing them visually on a canvas with logical connections. The application is built with a complete AWS serverless architecture and React frontend, ready for development and testing.

### Core Concept
- Users create text snippets within projects
- Snippets are positioned on a visual canvas (similar to Miro/FigJam)
- Snippets can be connected with directional, many-to-many relationships
- Each snippet has a primary text field (textField1) and a title field, plus tags/categories
- Inline editing of title and text directly on canvas nodes (click to edit, Cmd/Ctrl+Enter to save)
- Canvas supports zooming, panning, and drag-and-drop positioning

## Architecture

**Monorepo Structure**: All components (frontend, backend, infrastructure) will be organized in a single GitHub repository

**Technology Stack**:
- **Frontend**: React with React Flow for canvas interactions, Tailwind CSS for styling, AWS Amplify v6
- **Backend**: AWS Lambda (Node.js with TypeScript), GraphQL resolvers
- **API Layer**: GraphQL with AWS AppSync
- **Authentication**: AWS Cognito with JWT validation
- **Storage**: DynamoDB with Global Secondary Indexes (designed for Neptune migration)
- **Infrastructure**: AWS CDK for Infrastructure as Code
- **Hosting**: Amazon S3 + CloudFront (disabled for development security)

## Key Application Features

### User Management
- Self-registration with email/password
- Admin users with system analytics access (cannot view snippet content)
- All user data is private and scoped to individual users

### Project Organization
- Users can create multiple projects
- Snippets, tags, and categories are project-specific
- Project deletion cascades to all contained snippets

### Snippet Management
- Each snippet has a title field and primary text field (textField1), no length limits (expected up to page length)
- Unique alphanumeric IDs displayed on canvas (shortened to 8 characters)
- Large snippets (>100 words) shown minimized with expand modal
- Version history with revert capability
- Positioned freely on infinite canvas
- **Inline Editing**: Click title or text field directly on canvas node to edit, Cmd/Ctrl+Enter to save, Esc to cancel
- **Connected Content Aggregation**: Snippets automatically aggregate content from upstream connections
- **Prompt Designer**: Interactive prompt builder for AI generation with connected content preview
- **AI-Powered Image Generation**: Multiple models (Google Imagen 4 Fast, Gemini 2.5 Flash Image with multimodal support for up to 3 connected images)
- **Text-to-Text Generation**: LLM-powered snippet creation with model selection
- **Model Selection**: Choose text, image, or video models directly on snippet nodes
- **Optimistic Updates**: Instant UI feedback with background synchronization

### Connection System
- Directional relationships between snippets (A depends on B)
- Many-to-many connections supported
- Connection labels/tags for relationship types
- Connections created by entering target snippet ID
- Visual lines drawn between connected snippets

### Canvas Interface
- Infinite scrollable workspace per project
- Pan and zoom controls
- Drag-and-drop snippet positioning
- Context menus for snippet operations
- Modal dialogs for editing and connection management

## Development Commands

**Prerequisites**: Node.js 22+ (LTS), npm 10+, TypeScript 5.9+, AWS CLI configured (for deployment)

### Setup
- `npm install` - Install all workspace dependencies

### Development
- `npm run dev` - Start web app (localhost:3000) and API service concurrently
- `npm run dev:web` - Start only web app with Vite dev server
- `npm run dev:api` - Start only API service in watch mode

### Building
- `npm run build` - Build entire monorepo using automated script
- `cd apps/web && npm run build` - Build React web application
- `cd services/api && npm run build` - Build API Lambda functions
- `cd infrastructure/aws-cdk && npm run build` - Build CDK infrastructure
- `npm run generate` - Generate GraphQL types and schemas
- Build dependencies: shared-types → validation → graphql-schema → api → web → infrastructure

**Build Scripts**:
- `tools/scripts/build-all.sh` - Automated build process for all components
- `tools/scripts/generate-graphql.sh` - GraphQL schema and type generation
- `tools/scripts/setup-dev.sh` - Development environment setup
- `tools/scripts/deploy-stack.sh` - Deployment automation

### Code Quality
- `npm run lint` - Run linting for all workspaces (web + API build check)
- `npm run typecheck` - Run TypeScript type checking for all workspaces
- `cd apps/web && npm run lint` - ESLint for React frontend only
- `cd apps/web && npm run typecheck` - TypeScript type checking for frontend only

### Testing
- `cd services/api && npm run test` - Run Jest unit tests for API resolvers
- `npm run test` - Run all tests (API + integration tests from root)
- `npm run test:e2e` - Run Playwright end-to-end tests across all browsers
- `npm run test:e2e:headed` - Run E2E tests with browser UI visible
- `npm run test:e2e:debug` - Run E2E tests in debug mode
- `npm run test:e2e:ui` - Run E2E tests with Playwright UI
- `npm run test:e2e:report` - Show E2E test results report
- Tests include database operations, cascade deletes, authentication, validation
- Test files: `src/__tests__/` with setup, middleware, integration, and database tests

**E2E Test Setup**:
- Copy `.env.e2e.example` to `.env.e2e` and configure test environment
- E2E tests run against localhost:3000 with automatic dev server startup
- Tests configured for Chromium, Firefox, and WebKit browsers

### Infrastructure Testing
**CDK TDD Framework** (from infrastructure/aws-cdk/):
- `npm run test` - Run all infrastructure tests with comprehensive coverage
- `npm run test:unit` - Fast unit tests for individual stacks
- `npm run test:integration` - Cross-stack dependency tests
- `npm run test:validation` - Runtime validation against deployed infrastructure
- `npm run test:watch` - Watch mode for development
- `npm run test:coverage` - Generate coverage reports

**Test Organization**:
- **Unit Tests**: `test/unit/stacks/` - Auth, Database, API, and Media stack tests
- **Integration Tests**: `test/integration/cross-stack/` - Complete infrastructure synthesis
- **Validation Tests**: `test/validation/deployment/` - Runtime version and security validation
- **Critical Protection**: Node.js 22.x runtime enforcement, IAM permissions, security compliance

**Running Targeted Tests**:
- `cd infrastructure/aws-cdk && npm test -- auth-stack.test.ts` - Test specific stack
- `cd infrastructure/aws-cdk && npm test -- --testNamePattern="Node.js"` - Test with pattern matching
- `cd infrastructure/aws-cdk && npm test -- --changedSince=origin/main` - Test only changed files

### Deployment
**CDK Commands** (from infrastructure/aws-cdk/):
- `npm run synth` - Synthesize CloudFormation templates
- `npm run deploy` - Deploy all AWS stacks
- `npm run destroy` - Destroy all AWS stacks
- `npm run diff` - Show differences between deployed and local stacks

**Environment Setup**:
- Copy `apps/web/.env.example` to `apps/web/.env.local`
- Update environment variables with actual AWS resource IDs after deployment
- Variables needed: USER_POOL_ID, USER_POOL_CLIENT_ID, GRAPHQL_ENDPOINT


## Development Workflow

**Quick Start**:
1. `npm run setup` - Initialize development environment (runs setup-dev.sh)
2. `npm run dev` - Start both web app and API in development mode
3. `npm run test:e2e` - Run full end-to-end test suite

**GraphQL Development**:
- Schema definitions in `packages/graphql-schema/`
- Run `npm run generate` after schema changes to regenerate TypeScript types
- API resolvers automatically get updated type definitions

**Environment Configuration**:
- **Web App**: Copy `apps/web/.env.example` → `apps/web/.env.local`
- **E2E Tests**: Copy `.env.e2e.example` → `.env.e2e`
- Configure AWS resource IDs after initial deployment

## Monorepo Structure

**Workspaces Configuration**: Uses npm workspaces with apps/*, packages/*, services/*, infrastructure/aws-cdk

- **apps/web** - React frontend with React Flow, Vite, Tailwind CSS, AWS Amplify v6
- **packages/shared-types** - TypeScript definitions shared across monorepo
- **packages/validation** - Zod schemas for request validation
- **packages/graphql-schema** - GraphQL schema definitions and code generation
- **services/api** - AWS Lambda GraphQL resolvers with PowerTools and comprehensive tests
- **services/genai-orchestrator** - GenAI orchestration layer for LLM providers (Gemini, OpenAI)
- **services/media** - Media upload and processing handlers
- **services/genai** - GenAI-specific resolvers and business logic
- **infrastructure/aws-cdk** - Infrastructure as Code with CDK stacks for 6 AWS services
- **e2e/** - Playwright end-to-end tests
- **tools/scripts/** - Build, deployment, and development automation scripts

## Architecture Notes

**Data Model**:
- Users → Projects → Snippets (1:many relationships)
- Snippets have directional many-to-many connections with optional labels
- Project-scoped tags/categories
- Snippet versioning with revert capability
- One primary text field per snippet, positioned on infinite canvas

**Frontend Architecture**:
- React 19.2 with React Flow for canvas interactions, React Router for navigation
- AWS Amplify v6 for authentication integration with Cognito and GraphQL operations with AppSync (recently migrated from Apollo Client)
- **Feature-based Folder Structure**: Organized by domain (features/canvas/, features/projects/, shared/)
- **State Management**: Zustand stores for canvas, modals, prompts, optimistic updates, and toasts
- Optimistic UI updates for improved user experience
- Tailwind CSS for styling
- Authentication context with useAuth hook
- **Shared Component Library**: Reusable UI components (Button, Modal, Card, Input, Textarea, Toast, LoadingSpinner)
- **Custom Hooks Library**: useLocalStorage, useDebounce, useMediaQuery, useKeyPress, useGraphQLQueryWithCache
- **Centralized Constants**: Canvas config, validation rules, UI settings in shared/constants/
- **Error Boundaries**: Global and feature-level error handling with fallback UI

**Backend Architecture**:
- GraphQL API via AWS AppSync with Lambda resolvers organized by domain
- Resolver organization: Each domain (project, snippet, connection, user, genai) has separate mutations.ts and queries.ts
- DynamoDB with 6 tables (users, projects, snippets, connections, versions, generations) and Global Secondary Indexes
- AWS Cognito for authentication with JWT validation using aws-jwt-verify
- User data isolation: users can only access their own projects/snippets
- S3 for media uploads with pre-signed URLs (private bucket) and upload completion handlers
- GenAI integration with multiple LLM providers (Google Gemini, OpenAI) for image and text generation via genai-orchestrator service
- Secrets Manager for secure LLM API key storage
- CloudWatch for logging (monitoring stack disabled for cost savings)

**Critical Implementation Areas**:
- **Authentication**: AWS Cognito integration, admin users cannot access snippet content
- **Cascade Deletes**: Project deletion removes all snippets and connections (complete cascade deletion implemented with fix 1bc0934)
- **Media Handling**: Direct S3 uploads via pre-signed URLs (private bucket), backend registration on completion
- **Image Generation**: Multi-model support including Google Imagen 4 Fast, Gemini 2.5 Flash Image with multimodal capabilities using genai-orchestrator
- **Text Generation**: LLM-powered text-to-text generation for snippet creation using configurable models
- **Data Validation**: Zod schemas in packages/validation for consistent validation
- **Error Handling**: Consistent API error format, comprehensive CloudWatch logging
- **Infrastructure**: CDK stacks with app-specific resource naming for multi-app AWS accounts
- **Cost Optimization**: Monitoring stack disabled during development (~$5-7/month savings)
- **Resolver Pattern**: Domain-based organization with mutations.ts and queries.ts per feature area

**Frontend State Management (Zustand Stores)**:
- **canvasStore** (`features/canvas/store/canvasStore.ts`) - Canvas viewport persistence, zoom/pan state
- **promptDesignerStore** (`features/canvas/store/promptDesignerStore.ts`) - AI prompt builder UI state, connected content preview
- **optimisticUpdatesStore** (`features/canvas/store/optimisticUpdatesStore.ts`) - Optimistic UI updates, tracks pending/deleting/real snippets
- **modalStore** (`shared/store/modalStore.ts`) - Centralized modal state management
- **toastStore** (`shared/store/toastStore.ts`) - Toast notifications with queue management
- **projectStore** (`features/projects/store/projectStore.ts`) - Client-side project cache

**Shared Components & Utilities**:
- **UI Components** (`shared/components/ui/`): Button, Modal, Card, Input, Textarea, Toast, LoadingSpinner
- **Forms** (`shared/components/forms/`): FormField, FormSection, BaseFormModal, ConfirmationModal
- **Error Handling** (`shared/components/`): ErrorBoundary, ErrorFallback
- **Custom Hooks** (`shared/hooks/`): useLocalStorage, useDebounce, useMediaQuery, useKeyPress, useGraphQLQueryWithCache, useEntityForm, useModalForm
- **Utilities** (`shared/utils/`): textUtils (word counting, truncation), dateFormatters, errorLogger, formHelpers, validation
- **Constants** (`shared/constants/`): CANVAS_CONSTANTS, VALIDATION, UI, GRAPHQL, IMAGE_GENERATION, ERROR_MESSAGES

**Canvas Hooks** (feature-based organization):
- **useCanvasData** (`features/canvas/hooks/useCanvasData.ts`) - Data fetching, snippet/connection aggregation, memoization
- **useCanvasHandlers** (`features/canvas/hooks/useCanvasHandlers.ts`) - Event handlers for canvas operations
- **useFlowNodes** - Transform snippets to ReactFlow nodes with connected content analysis
- **useFlowEdges** - Transform connections to ReactFlow edges

**AWS Infrastructure** (6 deployed stacks):
1. **Auteurium-Auth-dev** - Cognito User Pool with email/password authentication
2. **Auteurium-Database-dev** - 6 DynamoDB tables: users, projects, snippets, connections, versions, generations
3. **Auteurium-Api-dev** - AppSync GraphQL API with Lambda resolvers for all CRUD operations
4. **Auteurium-Media-dev** - S3 private bucket with presigned URL functions for media uploads
5. **Auteurium-GenAI-dev** - GenAI integration with Secrets Manager for API keys, Lambda resolvers for image generation and content generation
6. **Auteurium-Web-dev** - S3 + CloudFront for hosting (CloudFront disabled for development security)

**Build Dependencies**:
1. packages/shared-types → packages/validation → packages/graphql-schema
2. services/genai-orchestrator (depends on shared-types, validation)
3. services/api (depends on packages + genai-orchestrator)
4. apps/web (depends on packages)
5. infrastructure/aws-cdk (independent, can be built separately)

## Infrastructure Testing Framework

**TDD Approach**: The CDK infrastructure uses Test-Driven Development with comprehensive test coverage to prevent regressions and ensure consistency across all AWS stacks.

**Key Testing Commands** (from infrastructure/aws-cdk/):
- `npm test -- auth-stack.test.ts` - Test specific stack
- `npm test -- --testNamePattern="Node.js"` - Test with pattern matching
- `npm test -- --changedSince=origin/main` - Test only changed files
- `export RUN_DEPLOYMENT_TESTS=true && npm run test:validation` - Validate deployed infrastructure

**Critical Test Areas**:
- **Runtime Version Safety**: Prevents regression to older Node.js versions (enforces 22.x)
- **Security Compliance**: IAM permissions, CORS configuration, authentication setup
- **Resource Integrity**: Naming conventions, cross-stack references, environment variables
- **Template Validation**: CDK Template assertions for all AWS resources

**Test Configuration**: Tests use Jest with ts-jest preset, separate tsconfig in test/tsconfig.json

## Monitoring Configuration

**Current Status**: CloudWatch monitoring stack is DISABLED for development cost savings (~$5-7/month)

**Re-enabling Monitoring** (when ready for production):
1. Rename `infrastructure/aws-cdk/lib/stacks/auteurium-monitoring-stack.ts.disabled` to `.ts`
2. Uncomment monitoring lines in `infrastructure/aws-cdk/lib/auteurium-app.ts`
3. Run: `cdk deploy Auteurium-Monitoring-dev`

**Monitoring Features** (when enabled):
- Custom CloudWatch dashboard with API and CloudFront metrics
- Automated alerts for API errors (>5 errors) and high latency (>5 seconds)
- SNS notifications for production issues
- Centralized logging with configurable retention

## Documentation References

**Architectural Documentation** (docs/ directory):
- `docs/ARCHITECTURE.md` - System architecture and design decisions
- `docs/ARCHITECTURE_DECISIONS.md` - ADRs and architectural choices
- `docs/application-requirements.md` - Complete feature specifications
- `docs/canvas-design.md` - UI/UX design and interaction patterns
- `docs/aws-well-architecture-analysis-results.md` - AWS Well-Architected Framework analysis

**Development Plans** (.cursor/plans/):
- `react-architecture-0483ac5b.plan.md` - React architecture refactoring plan (completed implementation includes feature-based structure, Zustand stores, shared components, error boundaries, toast notifications, and custom hooks)

**Current Architecture Status**:
- ✅ Feature-based folder structure implemented
- ✅ Zustand state management fully integrated
- ✅ Shared component library created
- ✅ Custom hooks library in use
- ✅ Error boundaries deployed
- ✅ Toast notification system replacing alerts
- ✅ Centralized constants and utilities
- ✅ Optimistic UI updates pattern
- ✅ Inline editing on canvas nodes
- ✅ Prompt designer for AI generation

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation files (*.md) or README files. Only create documentation files if explicitly requested by the User.
- Never try to commit until I ask you so

---
> Source: [twokey/auteurium-prototype](https://github.com/twokey/auteurium-prototype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-04 -->
