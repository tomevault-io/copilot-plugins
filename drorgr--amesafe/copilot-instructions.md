## amesafe

> **Essential Info**: Angular 20.2.1 frontend → AWS S3 + CloudFront | Project: `demo` | Build: `dist/demo/browser/`

# AmesaBase Project - Frontend - Cursor Rules

## 🚀 Quick Reference (TL;DR)

**Essential Info**: Angular 20.2.1 frontend → AWS S3 + CloudFront | Project: `demo` | Build: `dist/demo/browser/`

| Item | Value |
|------|-------|
| **Angular Project** | `demo` (in angular.json) |
| **Build Output** | `dist/demo/browser/` |
| **Angular Version** | 20.2.1 |
| **TypeScript** | 5.9.2 |
| **CloudFront ID** | E3GU3QXUR43ZOH |
| **Production URL** | https://dpqbvdgnenckf.cloudfront.net |
| **Local Dev** | http://localhost:4200 |
| **Backend API** | http://localhost:5000 (dev) / amesa-backend-alb-509078867.eu-north-1.elb.amazonaws.com (prod) |
| **Status** | ✅ Production operational, 100% complete |

**⚠️ CRITICAL**: Never pipe build commands directly - always use file-based output capture (see Agent Instructions)

---

## Project Overview
AmesaBase is a lottery management system with Angular frontend, .NET backend, and AWS infrastructure. The application is a property lottery platform featuring a "4Wins Model" where profits support community causes.

**⚠️ This repository is part of the AmesaBase-Monorepo workspace**
- **Monorepo Root**: `C:\Users\dror0\Curser-Repos\AmesaBase-Monorepo\`
- **This Repository (FE/)**: Angular frontend → https://github.com/DrorGr/amesaFE
- **Backend (BE/)**: .NET 8.0 backend → https://github.com/DrorGr/amesaBE
- **MetaData/**: Cross-cutting docs, scripts, and configs

## Technology Stack

| Category | Technology |
|----------|-----------|
| **Frontend Framework** | Angular 20.2.1, TypeScript 5.9.2, Tailwind CSS 3.4.3 |
| **Build System** | Angular CLI (`@angular/build`) |
| **Backend** | .NET 8.0 (Docker + ECS) |
| **Database** | Aurora PostgreSQL Serverless v2 |
| **Infrastructure** | AWS (S3, CloudFront, ECS, ALB) |
| **CI/CD** | GitHub Actions |
| **Package Manager** | npm |

## Frontend Project Configuration

**Project**: `demo` | **Build Output**: `dist/demo/browser/` | **CloudFront Origin Path**: `/browser`

| Configuration | Details |
|--------------|---------|
| **Build Configs** | `development` (source maps), `production` (optimized, both staging & prod) |
| **Environment Files** | `environment.ts` (base), `environment.dev.ts`, `environment.stage.ts`, `environment.prod.ts` |
| **Location** | `src/environments/` |
| **Contains** | `apiUrl`, `frontendUrl`, `production` flag, service-specific URLs |

## External Service Integrations

| Service | Package/Provider | Purpose |
|---------|-----------------|---------|
| **Stripe** | `@stripe/stripe-js` v8.5.3 | Payment processing |
| **SignalR** | `@microsoft/signalr` v9.0.6 | Real-time communication |
| **QR Codes** | `angularx-qrcode` v20.0.0, `html5-qrcode` v2.3.8 | QR generation & scanning |
| **OAuth** | Google, Meta (Facebook) | Authentication |
| **AWS Rekognition** | (via backend) | ID verification |
| **AWS SES** | (via backend) | Email delivery |
| **AWS SNS** | (via backend) | SMS/notifications |
| **reCAPTCHA Enterprise** | (via backend) | Bot protection |
| **Telegram Bot API** | (via backend) | Notification channel |
| **WebPush API** | (native) | Browser push notifications |

## Current Status (2025-01-25)

| Area | Status |
|------|--------|
| **Production Environment** | ✅ Fully operational |
| **Backend Admin Panel** | ✅ Deployed and working |
| **CloudFront Config** | ✅ API routing configured |
| **System Enhancement Plan** | ✅ **100% COMPLETE** - All phases done, gaps fixed |
| **Locale Formatting** | ✅ **100%** - 10 components, 3 services use LocaleService |
| **Accessibility** | ✅ **100%** - All components have ARIA labels & keyboard nav |
| **Dark Mode** | ✅ **100%** - All 57 components support dark mode |
| **Git Status** | ✅ Clean, all changes committed (branch: main) |

## Environment URLs

| Environment | Frontend | Backend API | Admin Panel |
|------------|----------|-------------|-------------|
| **Production** | https://dpqbvdgnenckf.cloudfront.net | amesa-backend-alb-509078867.eu-north-1.elb.amazonaws.com | http://amesa-backend-alb-509078867.eu-north-1.elb.amazonaws.com/admin ✅ |
| **Local Dev** | http://localhost:4200 | http://localhost:5000 | http://localhost:5000/admin |

**Note**: See root `.cursorrules` for complete environment URLs table and backend API endpoints.

## Coding Standards

### Angular/TypeScript
- Use standalone components architecture
- Follow Angular 20.2.1 best practices
- Implement lazy loading with custom preloading strategy
- Use TypeScript strict mode
- Follow Tailwind CSS utility-first approach
- Implement proper error handling and logging

### Environment Configuration
- Use external configuration injection
- Never hardcode environment-specific values
- Maintain separate configs for local development and production
- Use GitHub secrets for sensitive data

### AWS Infrastructure
- Follow serverless architecture patterns
- Use proper CloudFront origin path configurations
- Implement proper cache invalidation strategies
- Maintain environment isolation
- **Region**: eu-north-1
- **Account**: 129394705401
- **S3 Buckets**: amesa-frontend-prod
- **CloudFront Distributions**: Production distribution with API routing configured
- **ECS Cluster**: "Amesa" with Fargate launch type
- **Aurora PostgreSQL**: Serverless v2 production cluster
- **API Routing**: CloudFront `/api/*` routes proxy to backend ALB

### Code Patterns and Conventions
- **Component Naming**: PascalCase for class names, kebab-case for selectors (e.g., `HouseCarouselComponent` → `<app-house-carousel>`)
- **Service Naming**: PascalCase with "Service" suffix (e.g., `AuthService`, `PaymentService`)
- **File Naming**: Match class name (e.g., `house-carousel.component.ts` for `HouseCarouselComponent`)
- **Directory Structure**: Organize by feature (e.g., `src/components/payment/`, `src/services/payment/`)
- **Imports**: Group imports (Angular, third-party, local) with blank lines between groups
- **RxJS**: Always unsubscribe in `ngOnDestroy` or use `takeUntil` pattern
- **Error Handling**: Use try-catch blocks, proper error logging, user-friendly error messages
- **Type Safety**: Use TypeScript strict mode, avoid `any`, use proper interfaces/types
- **Accessibility**: All interactive elements must have ARIA labels and keyboard navigation
- **Responsive Design**: Mobile-first approach with Tailwind CSS breakpoints
- **Dark Mode**: All components must support dark mode via Tailwind dark: classes

## Key Configuration Files

| File | Purpose | Key Details |
|------|---------|-------------|
| **`angular.json`** | Angular project config | Project: `demo`, Output: `dist/demo/browser/`, Builds: `development`, `production` |
| **`package.json`** | Dependencies & scripts | Angular 20.2.1, TS 5.9.2, Tailwind 3.4.3, Scripts: `ng serve/build/test` |
| **`tsconfig.json`** | TypeScript config | Strict mode, ES2022 target |
| **`tailwind.config.js`** | Tailwind CSS | Custom palette, dark mode, spacing, typography |
| **`playwright.config.ts`** | E2E tests | 5 browsers, 2 retries, Base: localhost:4200 |
| **`karma.conf.js`** | Unit tests | Jasmine, ChromeHeadless (CI), Coverage disabled in CI |
| **`eslint.config.js`** | Linting | TypeScript + Angular ESLint, template linting, accessibility |
| **`proxy.conf.json`** | Dev proxy | `/api/v1/*` → ALB, `/ws/*` → SignalR, CORS configured |
| **`src/environments/*.ts`** | Environment configs | Base, dev, stage, prod (each: `apiUrl`, `frontendUrl`, `production`) |

## File Structure Guidelines
- Keep context files updated with current status
- Maintain comprehensive documentation
- Use consistent naming conventions
- Organize components by feature

## Documentation Requirements
- Update context files when making significant changes
- Document all infrastructure changes
- Maintain troubleshooting guides
- Keep deployment status current

## Important Configuration Notes

- **Project Name**: `demo` (defined in `angular.json`)
- **Build Output**: `dist/demo/browser/` → CloudFront origin path `/browser`
- **Environment Files**: `src/environments/` (base, dev, stage, prod)
- **API Routing**: CloudFront `/api/*` → Backend ALB (automatic proxy)
- **Local Dev**: Uses `proxy.conf.json` → Backend at http://localhost:5000
- **Production**: Uses CloudFront → Production ALB

## Context Files to Maintain

| Location | File/Path | Purpose |
|----------|-----------|---------|
| **FE/** | `.cursorrules` | Main context file (this file) |
| **MetaData/Documentation/** | `Infrastructure/DEPLOYMENT_GUIDE.md` | Deployment guide (includes status) |
| **MetaData/Documentation/** | `AWS_INFRASTRUCTURE_DETAILS.md` | Infrastructure details |
| **MetaData/Documentation/** | `TROUBLESHOOTING.md` | Common issues and solutions |
| **MetaData/Documentation/** | `ENVIRONMENT_URLS_REFERENCE.md` | Comprehensive URLs |
| **MetaData/Reference/** | `ENVIRONMENT_URLS_GRID.csv` | CSV for team sharing |
| **MetaData/Reference/** | `AmesaEnvironmentGrid.gs` | Google Apps Script |
| **MetaData/Scripts/** | `*.sql`, `*.ps1`, `*.sh` | Deployment and utility scripts |
| **MetaData/Configs/** | CloudFront, S3, ECS configs | Infrastructure configurations |

**Note**: All paths relative to FE/ directory. Use `../MetaData/...` to access cross-cutting docs.

## Agent Instructions
- Always check context files for current project status
- Update relevant context files when making changes
- Follow the established patterns and conventions
- Maintain environment consistency for local development and production
- Document any infrastructure or configuration changes
- **CRITICAL**: If any changes relate to project structure, environment configuration, infrastructure, or deployment processes, update the appropriate context files and this .cursorrules file

## Pieces MCP Integration

### Overview
Pieces MCP provides access to Long-Term Memory (LTM-2.7) engine, allowing agents to query workflow context, code snippets, notes, and activity history. This significantly improves agent effectiveness by providing access to your workflow history and saved context.

**Key Benefits:**
- **Context Continuity**: Agents understand recent work and decisions across sessions
- **Pattern Reuse**: Discover existing implementations and proven solutions
- **Decision Tracking**: Understand project evolution and documented challenges
- **Resource Discovery**: Access saved code snippets, bookmarks, and notes
- **Workflow Efficiency**: Faster problem-solving with historical context

### Prerequisites (Optional)
- **PiecesOS**: Optional - Can be installed and running (check system tray/processes) if you want to use Pieces MCP
- **Long-Term Memory Engine (LTM-2.7)**: Optional - Can be enabled in PiecesOS if you want LTM features
- **MCP Configuration**: Optional - Pieces MCP server can be configured in Cursor Settings if desired
- **Tool Availability**: `ask_pieces_ltm` tool is automatically available in **ALL modes** (Auto, Agent, Manual) when MCP server is configured, but it's **optional to use**

**Setup Documentation**: See `MetaData/Documentation/Development/PIECES_MCP_INTEGRATION.md` for complete setup instructions.

**Note**: Pieces MCP works seamlessly in **Auto Mode** - no special mode selection required. The tool is automatically available when the MCP server is configured and running.

### Usage Guidelines for Frontend Development

#### When to Use Pieces MCP

**1. Context Retrieval** - Understand previous frontend work:
- "What was I working on in the frontend yesterday?"
- "Show me recent Angular component changes"
- "What frontend components did I modify last week?"
- "What was I doing with this component file yesterday?"

**2. Code Pattern Discovery** - Find existing Angular/TypeScript implementations:
- "Show examples of Angular Signals usage in this project"
- "What was my last implementation of error handling in components?"
- "Have I previously implemented similar UI patterns?"
- "Show me how I implemented authentication guards"
- "What patterns did I use for state management?"

**3. Decision Tracking** - Understand frontend architecture decisions:
- "Track the evolution of the payment flow component"
- "Review documented challenges with SignalR integration"
- "Show the decisions made around routing structure"
- "What architectural decisions were made for component organization?"

**4. Resource Discovery** - Find saved frontend resources:
- "Find recent bookmarks about Angular 20 features"
- "What resources did I save recently related to Tailwind CSS patterns?"
- "Show notes taken about TypeScript best practices"
- "What documentation did I bookmark about Angular Signals?"

**5. Code Review Context** - Understand frontend feedback:
- "Show code review comments related to component refactoring"
- "Did we finalize naming conventions for Angular components?"
- "What feedback did I leave on recent frontend PRs?"
- "What issues were identified in the accessibility audit?"

#### Frontend-Specific Query Examples

**Component Implementation Patterns:**
- "How did I implement the payment consolidated step component?"
- "Show me examples of Angular Signals patterns I've used"
- "What was my approach to implementing dark mode support?"
- "How did I structure reusable UI components?"

**State Management Patterns:**
- "Show me examples of Angular Signals usage in components"
- "What was my approach to managing component state?"
- "How did I implement reactive state with RxJS?"
- "Show me patterns for localStorage caching I've used"

**Service & API Patterns:**
- "How did I structure the API service for HTTP requests?"
- "Show me examples of error handling patterns in services"
- "What was my approach to implementing retry logic?"
- "How did I handle SignalR connection management?"

**UI/UX Patterns:**
- "Show me examples of accessibility implementations"
- "What patterns did I use for responsive design?"
- "How did I implement theme switching?"
- "Show me examples of loading states and error handling"

**Testing Patterns:**
- "How did I structure component unit tests?"
- "Show me examples of Playwright E2E test patterns"
- "What was my approach to mocking services in tests?"
- "How did I implement test data builders?"

### Agent Workflow Integration

#### Automatic Usage in All Modes
**Note**: Pieces MCP works in **ALL modes** (Auto, Agent, Manual). The `ask_pieces_ltm` tool is automatically available when MCP server is configured, but using it is **optional**.

**Best Practice**: Pieces MCP can be helpful for context retrieval, but agents should use standard workflow if Pieces MCP is not available or not desired.

#### On Agent Creation - Optional Enhancement
**Note**: When a new agent session starts, you **can optionally** use Pieces MCP for additional context:

1. **Optional: Check Pieces MCP Availability**:
   - **Can attempt to use `ask_pieces_ltm` tool** if you want additional context from workflow history
   - If tool is available and you think it would be helpful, you can use it
   - If tool is not available or you prefer not to use it, continue with standard workflow
   - **Pieces MCP is optional** - standard workflow works fine without it

2. **Optional: Query Recent Frontend Context** (if tool available and desired):
   - **Can query**: "What was I working on in the frontend in the last 24 hours?" (if helpful)
   - **Can query**: "What recent changes were made to Angular components?" (if helpful)
   - **Can query**: "Show me recent work on the payment flow" (if helpful)
   - **Can query**: "What frontend issues or bugs were recently fixed?" (if helpful)
   - **Can query**: "What UI/UX improvements were recently implemented?" (if helpful)
   - Use these queries only if you think they would provide useful context

3. **Optional: Understand Current Frontend State** (if helpful):
   - "What is the current deployment status of the frontend?" (if Pieces MCP might have this info)
   - "What frontend features are in progress?" (if Pieces MCP might have this info)
   - "What technical debt or known issues exist in the frontend?" (if Pieces MCP might have this info)
   - "What recent accessibility or performance improvements were made?" (if Pieces MCP might have this info)

#### Automatic Usage Triggers for Frontend

**PRIMARY USE CASES - Use Pieces MCP (`ask_pieces_ltm` tool) when:**

1. ✅ **When Stuck on Frontend Issues** - Query for similar problems and solutions:
   - Encountering an Angular error or build issue
   - Facing a UI/UX problem without clear solution
   - Need to understand why a component isn't working
   - Looking for troubleshooting approaches for frontend issues

2. ✅ **Need Frontend Patterns** - Query for existing implementations:
   - Looking for Angular patterns or architectural patterns
   - Need examples of similar component implementations
   - Want to follow established frontend conventions
   - Seeking proven solutions for frontend problems

3. ✅ **Frontend Task Could Benefit from Past Work** - Query for related context:
   - Starting a new frontend task or feature (query for related previous work)
   - Need to understand frontend history and evolution
   - Understanding frontend architectural decisions
   - Finding related frontend code or documentation
   - Need context about why frontend decisions were made

4. ✅ **Work Completed Earlier in the Day** - Query for recent frontend work:
   - User asking about frontend work completed earlier today
   - Need to understand what frontend changes were made recently
   - Looking for context about today's frontend activities

5. ✅ **References to People, Applications, or Research** - Query for specific context:
   - User references specific people by name (e.g., in code reviews, discussions)
   - User mentions specific applications (e.g., VS Code, Chrome DevTools, GitHub)
   - User asks about frontend research activities
   - Need context about conversations or interactions related to frontend work

#### User Rules for `ask_pieces_ltm` Tool (Optional)

**Note**: These are guidelines for when you choose to use the `ask_pieces_ltm` tool (it's optional):

1. **When to Consider Using**:
   - ✅ User is asking about frontend work completed earlier in the day (Pieces MCP might have this context)
   - ✅ User references names of specific people (Pieces MCP might have conversation history)
   - ✅ User mentions specific applications (Pieces MCP might have activity from those apps)
   - ✅ User asks about frontend research activities (Pieces MCP might have saved research)

2. **Query Best Practices** (if using the tool):
   - **Should specify time ranges** in queries for better results (e.g., "today", "earlier today", "this morning", "last 24 hours")
   - **Can include other suggested queries** when appropriate (e.g., related frontend topics, follow-up questions)

3. **Source Assumptions**:
   - **If keyword 'researching' is used**: Assume primary application source is **Google Chrome**
   - **If a person is referenced**: Assume source is **Google Chrome** or **Google Chat**

4. **Tool Usage**:
   - When using `ask_pieces_ltm`, you are free to make multiple calls to the tool
   - Consider making follow-up queries based on initial results
   - Use time ranges and source hints to improve query accuracy

#### Using `create_pieces_memory` Tool

**Purpose**: Create persistent memories in Pieces LTM for important frontend decisions, patterns, and context that future agent sessions should know.

**When to Use `create_pieces_memory`**:
- ✅ **After Completing Significant Frontend Work** - Document important component implementations, decisions, or solutions
- ✅ **After Solving Complex Frontend Problems** - Save troubleshooting approaches and solutions
- ✅ **After Making Frontend Architectural Decisions** - Document why decisions were made (e.g., component structure, state management, routing)
- ✅ **After Discovering Frontend Patterns** - Save proven Angular patterns, TypeScript patterns, and best practices
- ✅ **After Code Reviews or Feedback** - Document important frontend feedback and changes
- ✅ **After Major Frontend Refactoring** - Document structural changes and rationale
- ✅ **After UI/UX Improvements** - Document design decisions and user experience changes
- ✅ **After Performance Optimizations** - Document optimization approaches and results

**What to Include in Frontend Memories**:
- **Summary**: Concise title describing the frontend work (1-2 sentences)
- **Detailed Narrative**: Complete story with background, thought process, what worked/didn't work, decisions made
- **Files**: List all relevant frontend files involved (absolute paths, e.g., `FE/src/components/payment/payment-consolidated-step.component.ts`)
- **External Links**: GitHub URLs, Angular documentation, TypeScript documentation, articles consulted
- **Project Path**: Absolute path to project root (`C:\Users\dror0\Curser-Repos\AmesaBase-Monorepo`)
- **Context**: Why this memory is important for future frontend sessions

**Frontend-Specific Memory Examples**:
- After implementing a new component: Document the component structure, state management, and integration approach
- After fixing a critical frontend bug: Document the root cause, fix, and prevention strategies
- After routing changes: Document the routing structure and navigation flow
- After state management changes: Document the state management approach and rationale
- After performance optimizations: Document the optimization approach and results
- After accessibility improvements: Document the accessibility enhancements and impact

**Best Practices**:
- Create memories for frontend work that future agents will benefit from understanding
- Include enough context that someone reading it later understands the full picture
- Link to relevant frontend files and external resources
- Use clear, descriptive summaries
- Document both successes and lessons learned
- Include component names, service names, and routing paths when relevant

**Additional Triggers:**
- ✅ **Starting a new frontend task or feature** - Query for related previous work
- ✅ **Encountering a frontend error or issue** - Query for similar problems and solutions
- ✅ **Need to understand frontend history** - Query for evolution and decisions
- ✅ **Looking for existing frontend patterns or solutions** - Query for implementations
- ✅ **Understanding frontend architectural decisions** - Query for documented choices
- ✅ **Finding related frontend code or documentation** - Query for saved resources

**Optional Decision Flow** (consider Pieces MCP if you think it would help):
1. Is `ask_pieces_ltm` tool available? → **YES**: Consider using it if helpful → **NO**: Continue with standard workflow
2. **Am I stuck on a frontend issue?** → **YES**: Can query LTM for solutions (optional) → **NO**: Continue to next check
3. **Do I need frontend patterns?** → **YES**: Can query LTM for implementations (optional) → **NO**: Continue to next check
4. **Could this frontend task benefit from understanding past work?** → **YES**: Can query LTM for related context (optional) → **NO**: Continue
5. Starting new frontend task? → **YES**: Can query LTM for related work (optional) → **NO**: Continue

#### During Frontend Task Execution

**Before Starting Work** (Optional Enhancement):
1. **Optional: Query LTM for related previous frontend work** (if Pieces MCP is available and you think it would help):
   - "Have I worked on similar frontend features/components before?" (if helpful)
   - "What patterns or solutions did I use for [similar frontend task]?" (if helpful)
   - "Show me previous implementations of [component type]" (if helpful)
   - "How did I handle [similar requirement] in other components?" (if helpful)
   - "What was my approach to [similar frontend feature]?" (if helpful)

2. **Optional: Check for Existing Frontend Solutions** (if Pieces MCP is available):
   - "How did I solve [similar frontend problem] before?" (if helpful)
   - "What approaches have been tried for [frontend issue type]?" (if helpful)
   - "Are there existing patterns I should follow for [frontend feature]?" (if helpful)
   - "How did I implement [similar component/service] before?" (if helpful)

**When Stuck on Frontend Issues** (Optional Enhancement):
**Note**: When stuck on a frontend issue, you **can optionally** query Pieces MCP for solutions, but standard troubleshooting approaches work fine too.

1. **Optional: Query LTM for frontend solutions** (if Pieces MCP is available and you think it would help):
   - "How did I solve [similar frontend problem] before?" (if helpful)
   - "Show me examples of [frontend pattern/technique] I've used" (if helpful)
   - "What resources did I save about [frontend topic]?" (if helpful)
   - "How did I handle [similar build/deployment issue]?" (if helpful)
   - "What approaches have been tried for [frontend issue type]?" (if helpful)
   - "What was my troubleshooting approach for [similar error]?" (if helpful)

2. **Optional: Find Related Frontend Context** (if Pieces MCP is available):
   - "What documentation exists about [frontend topic]?" (if helpful)
   - "Show me code snippets related to [frontend feature]" (if helpful)
   - "What decisions were made about [frontend architectural choice]?" (if helpful)
   - "How did I configure [Angular feature/Tailwind pattern] before?" (if helpful)

**When Needing Frontend Patterns** (Optional Enhancement):
**Note**: When you need frontend patterns or examples, you **can optionally** query Pieces MCP, but codebase search and standard approaches work fine too.

1. **Query for Frontend Implementation Patterns**:
   - "Show me examples of [Angular pattern/technique] I've used in this project"
   - "What patterns did I use for [similar frontend requirement]?"
   - "How did I implement [similar frontend feature] before?"
   - "What architectural patterns have I used for [component/service type]?"
   - "Show me examples of [Angular Signals/RxJS/Tailwind] patterns I've used"

2. **Query for Frontend Code Examples**:
   - "Show me code snippets related to [frontend feature/pattern]"
   - "What was my implementation approach for [similar frontend task]?"
   - "How did I structure [similar component/service]?"
   - "Show me examples of [component/service/guard] implementations"

**When Frontend Task Could Benefit from Past Work** (Optional Enhancement):
**Note**: Before starting any frontend task, you **can optionally** query Pieces MCP if you think understanding past work would help, but it's not required.

1. **Optional: Query for Related Previous Frontend Work** (if Pieces MCP is available and helpful):
   - "Have I worked on similar [frontend feature/component] before?" (if helpful)
   - "What was my approach to [similar frontend task]?" (if helpful)
   - "What decisions were made about [related frontend feature]?" (if helpful)
   - "How did I handle [similar requirement] in other components?" (if helpful)
   - "What was my implementation strategy for [similar frontend feature]?" (if helpful)

2. **Optional: Query for Frontend Context and History** (if Pieces MCP is available):
   - "What is the history of [frontend feature/component]?" (if helpful)
   - "Why was [frontend decision/approach] chosen?" (if helpful)
   - "What challenges were encountered with [related frontend feature]?" (if helpful)
   - "What lessons were learned from [previous frontend implementation]?" (if helpful)
   - "What architectural decisions were made for [frontend component]?" (if helpful)

**After Completing Frontend Work** (Optional Enhancement):
**Note**: After completing significant frontend work, you **can optionally** use `create_pieces_memory` to document important context for future reference, but it's not required.

1. **Optional: Evaluate if Memory Would Be Helpful**:
   - Was this a significant frontend implementation or decision? (if so, can document)
   - Will future agents benefit from understanding this frontend work? (if so, can document)
   - Does this solve a frontend problem that might recur? (if so, can document)
   - Is this a frontend pattern or approach worth preserving? (if so, can document)

2. **Optional: Create Memory if Desired**:
   - Can use `create_pieces_memory` tool to document (optional):
     - What frontend work was implemented/fixed/decided
     - Why frontend decisions were made
     - What worked and what didn't
     - Relevant frontend files and resources
     - Lessons learned
   - This can help future agent sessions understand frontend context, but it's optional

3. **Memory Content Guidelines**:
   - **Summary**: Clear, concise title (1-2 sentences)
   - **Detailed Narrative**: Complete story with full frontend context
   - **Files**: All relevant frontend files (absolute paths, e.g., `FE/src/components/payment/payment-consolidated-step.component.ts`)
   - **External Links**: Documentation, GitHub URLs, Angular docs, TypeScript docs, articles
   - **Project Path**: Absolute path to project root (`C:\Users\dror0\Curser-Repos\AmesaBase-Monorepo`)

#### Optimized Frontend Query Templates

**Session Start Queries** (Use these immediately):
- "What was I working on in the frontend in the last 24 hours?"
- "What recent changes were made to [specific Angular component/service]?"
- "What frontend issues or bugs were recently fixed?"
- "What is the current deployment status of the frontend?"

**Task Start Queries** (Use before starting any frontend task):
- "Have I worked on similar [frontend feature/component] before?"
- "What patterns or solutions did I use for [similar frontend requirement]?"
- "Show me previous implementations of [component/service type]"
- "How did I handle [similar requirement] in other components?"

**Problem Solving Queries** (Use when stuck on frontend issues):
- "How did I solve [similar frontend problem] before?"
- "What approaches have been tried for [frontend issue type]?"
- "Show me examples of [frontend pattern/technique] I've used"
- "What resources did I save about [frontend topic]?"

### Troubleshooting

**MCP Server Not Running**:
- Verify PiecesOS is running (check system tray/processes)
- Check MCP server status in Cursor settings (should show "running")
- Restart MCP server if needed
- Verify LTM engine is enabled in PiecesOS

**Tool Not Available**:
- **Verify MCP Connection**: Check green dot in MCP settings (should show "running")
- **Check Mode**: Tool works in ALL modes (Auto, Agent, Manual) - no special mode required
- **Restart Cursor**: Sometimes requires IDE restart after MCP configuration
- **Verify PiecesOS**: Ensure PiecesOS is running and LTM is enabled
- **Note**: In Auto Mode, tool should be automatically available - if not, check MCP server status

**No Results from Queries**:
- Verify LTM is enabled
- Check PiecesOS is capturing your workflow
- Wait for indexing (LTM may need time to index recent activity)
- Check date ranges in queries reference timeframes with activity

**Documentation**: See `MetaData/Documentation/Development/PIECES_MCP_INTEGRATION.md` for complete troubleshooting guide.

### ⚠️ CRITICAL: Build Command Safety Rules
**NEVER** run build commands that pipe directly to the agent output. Large build outputs can crash the agent and disconnect the chat session.

**❌ FORBIDDEN Commands:**
- `npm run build 2>&1 | Select-Object -First 100` (crashes agent)
- `npm run build | head -n 100` (crashes agent)
- Any command that streams build output directly to stdout without file buffering

**✅ REQUIRED: Use File-Based Output Capture**
When running build commands, ALWAYS capture output to a file first, then read portions of the file:

```powershell
# CORRECT: Capture to file, then read
cd FE
npm run build 2>&1 | Out-File -FilePath build-output.log -Encoding utf8
# Then read only what's needed
Get-Content build-output.log -Tail 50
# Or filter for errors only
Select-String -Path build-output.log -Pattern "ERROR|Error|error|Failed|failed" | Select-Object -First 20
```

**Alternative Safe Patterns:**
```powershell
# Check exit code first, then read file
$output = npm run build 2>&1 | Out-File -FilePath build-output.log -Encoding utf8
if ($LASTEXITCODE -eq 0) {
    Write-Host "Build succeeded"
    Get-Content build-output.log -Tail 20
} else {
    Select-String -Path build-output.log -Pattern "ERROR|Error|error|Failed|failed" | Select-Object -First 30
}
```

**Why This Matters:**
- Angular build outputs can be extremely verbose (1000+ lines)
- Streaming large output directly crashes the agent
- File-based capture isolates the agent from output streams
- Reading file portions allows selective, safe output display

**Exception**: Only use direct piping for commands with guaranteed small output (< 20 lines), and even then prefer file capture for consistency.

## File Path Reference

| Path Pattern | Location | Description |
|--------------|----------|-------------|
| `src/components/{feature}/` | Components | Standalone components, organized by feature |
| `src/services/{feature}.service.ts` | Services | Business logic services |
| `src/guards/{feature}.guard.ts` | Guards | Route guards (e.g., `auth.guard.ts`) |
| `src/environments/*.ts` | Environment configs | Base, dev, stage, prod configurations |
| `src/app.preloading-strategy.ts` | Routing | Custom preloading strategy |
| `src/main.ts` | Entry point | Application bootstrap |
| `src/styles.css`, `src/global_styles.css` | Styles | Global stylesheets |
| `src/assets/` | Static assets | Images, fonts, etc. |
| `dist/demo/browser/` | Build output | Production build destination |
| `angular.json`, `package.json`, `tsconfig.json` | Root config | Project configuration files |
| `*.spec.ts` | Tests | Unit test files (alongside source) |
| `e2e/` | E2E tests | Playwright end-to-end tests |

**Note**: All paths relative to `FE/` directory. Use `../MetaData/...` for cross-cutting documentation.

---

## Frontend Application Structure

### Routes (20+ Routes)
All routes use lazy loading with `loadComponent()`:
- `/` - Home page
- `/about` - About page
- `/sponsorship` - Sponsorship page
- `/faq` - FAQ page
- `/help` - Help center
- `/register` - User registration
- `/member-settings` - Member settings (AuthGuard protected)
- `/partners` - Partners page
- `/promotions` - Promotions page
- `/responsible-gambling` - Responsible gambling info
- `/how-it-works` - How it works page
- `/lottery-results` - Lottery results listing
- `/lottery-result/:id` - Lottery result detail
- `/lottery/dashboard` - Lottery dashboard
- `/lottery/favorites` - Lottery favorites (AuthGuard protected)
- `/lottery/entries/active` - Active entries (AuthGuard protected)
- `/lottery/entries/history` - Entry history (AuthGuard protected)
- `/house/:id` - House detail page
- `/houses/:id` - Alternative house detail route
- `/search` - Search page
- `/auth/callback` - OAuth callback handler
- `/verify-email` - Email verification
- `/settings/sessions` - Session management (AuthGuard protected)
- `/products` - Product selector
- `/payment/checkout` - Payment checkout (AuthGuard protected)
- `/payment/stripe` - Stripe payment (AuthGuard protected)
- `/payment/crypto` - Crypto payment (AuthGuard protected)
- `/payment/methods` - Payment methods (AuthGuard protected)

### Services (50+ Services)

| Category | Count | Key Services |
|----------|-------|--------------|
| **Core** | 5 | `api`, `auth`, `translation`, `locale`, `theme` |
| **Feature** | 11 | `lottery`, `lottery-results`, `payment`, `stripe`, `crypto-payment`, `reservation`, `product`, `notification`, `realtime`, `web-push`, `telegram-link` |
| **User** | 5 | `user`, `user-preferences`, `user-preferences-form`, `identity-verification`, `member-settings` |
| **UI** | 7 | `auth-modal`, `cookie-consent`, `toast`, `payment-panel`, `accessibility`, `mobile-detection`, `navigation` |
| **Utility** | 18 | `error-handling`, `error-mapping`, `error-message`, `retry`, `validation`, `logging`, `performance`, `memory-monitor`, `offline`, `security`, `security-audit`, `file`, `content`, `promotion`, `promotions-menu`, `captcha`, `analytics`, etc. |
| **Component-Specific** | 7 | `house-card`, `house-carousel`, `registration-form`, `route-loading`, `route-performance`, `focus-trap`, `heart-animation` |

**Location**: `src/services/` | **Pattern**: `{feature}.service.ts`

### Components (50+ Components)

| Category | Count | Key Components |
|----------|-------|----------------|
| **Page** | 10 | `home`, `about-page`, `sponsorship-page`, `faq-page`, `help-center-page`, `partners-page`, `promotions-page`, `responsible-gambling-page`, `how-it-works-page`, `search-page` |
| **Lottery** | 13 | `lottery-dashboard`, `lottery-favorites`, `lottery-results-page`, `lottery-result-detail`, `active-entries`, `entry-history`, `house-card`, `house-carousel`, `house-detail`, `house-grid`, `participant-stats`, etc. |
| **Payment** | 13 | `payment-checkout`, `payment-modal`, `responsive-payment-panel`, `payment-consolidated-step` ✅ (NEW), `stripe-payment`, `crypto-payment`, `payment-methods`, legacy multi-step components, etc. |
| **User** | 8 | `registration-page`, `member-settings-page`, `user-menu`, `user-preferences`, `sessions-management`, `verify-email`, `oauth-callback`, `verification-gate` |
| **UI** | 16 | `topbar`, `hero-section`, `toast`, `auth-modal`, `cookie-consent`, `language-switcher`, `theme-toggle`, `accessibility-menu`, `translation-loader`, `skip-links`, `chatbot`, `stats-section`, `countdown-timer`, `live-inventory`, `reservation-countdown`, `reservation-status` |
| **Notification** | 4 | `notification-sidebar`, `notification-preferences`, `web-push-permission`, `telegram-link` |
| **Utility** | 2 | `password-reset-modal`, `promotions-sliding-menu` |

**Location**: `src/components/` | **Pattern**: `{feature}.component.ts` | **All components**: Standalone, support dark mode, accessibility compliant

## Architecture Overview

### GitHub Repository Structure
- **amesaFE**: https://github.com/DrorGr/amesaFE (Frontend)
- **amesaBE**: https://github.com/DrorGr/amesaBE (Backend)
- **amesaDevOps**: Infrastructure as Code repository

### AWS Architecture
```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Frontend      │    │     Backend      │    │    Database     │
│                 │    │                  │    │                 │
│ Angular 20.2.1  │───▶│ .NET 8.0 + Docker│───▶│ Aurora PostgreSQL│
│ S3 + CloudFront │    │ ECS Fargate      │    │ Serverless v2   │
└─────────────────┘    └──────────────────┘    └─────────────────┘
```

### Deployment Flow
1. **Production**: Push to main branch → Auto-deploy to S3/CloudFront
2. **Local Development**: Run `ng serve` for local testing

### SignalR Hub Details
- **LotteryHub** (`/ws/lottery`):
  - **Service**: `amesa-lottery-service`
  - **Events Broadcasted**:
    - `OnLotteryUpdate` - House status, ticket sales, participation updates
    - `OnFavoriteUpdate` - Watchlist changes (added/removed)
    - `OnEntryStatusChange` - Ticket entry status changes
    - `OnDrawReminder` - Lottery draw countdown reminders
    - `OnRecommendation` - House recommendations
    - `InventoryUpdated` - Real-time ticket inventory updates
    - `TicketPurchased` - Ticket purchase notifications
    - `CountdownUpdated` - Lottery countdown updates
    - `LotteryDrawStarted` - Draw start notifications
    - `LotteryDrawCompleted` - Draw completion notifications
  - **User Groups**: Automatically managed in `OnConnectedAsync()` (no manual join/leave needed)
  - **Authorization**: JWT Bearer token required
  
- **NotificationHub** (`/ws/notifications`):
  - **Service**: `amesa-notification-service`
  - **Events Broadcasted**:
    - `OnNotification` - User notifications (email, SMS, WebPush, Telegram)
    - `OnUserStatusUpdate` - User online/offline status
  - **User Groups**: Automatically managed in `OnConnectedAsync()` (no manual join/leave needed)
  - **Authorization**: JWT Bearer token required

## Cross-Cutting Concerns

### HTTP Request/Response Patterns
- **No HTTP Interceptors**: All HTTP logic centralized in `ApiService`
- **Token Management**: Automatic token refresh via `ApiService.token$` observable
- **Request Retry**: Automatic retry with exponential backoff for network errors
- **Error Handling**: Centralized error handling via `ErrorHandlingService`
- **Offline Support**: `OfflineService` queues requests when offline, retries when back online
- **Request Format**: All requests use `ApiResponse<T>` wrapper format
- **Base URL**: Environment-specific (`/api/v1` for local, CloudFront URL for production)

### SignalR Real-Time Communication
- **Hubs**: 
  - `/ws/lottery` - Lottery updates (LotteryHub)
  - `/ws/notifications` - Notifications (NotificationHub)
- **Transport**: LongPolling (required for CloudFront compatibility, WebSocket not supported)
- **Connection Pattern**:
  - Automatic reconnection with exponential backoff (max 5 retries)
  - Connection state tracked with signals (`isConnected`, `connectionState`)
  - Token-based authentication (JWT sent in connection headers)
- **Event Subscriptions**:
  - `LotteryUpdateEvent` - House status, ticket sales, participation updates
  - `NotificationEvent` - User notifications (email, SMS, WebPush, Telegram)
  - `UserStatusEvent` - User online/offline status
  - `FavoriteUpdateEvent` - Watchlist changes (added/removed)
  - `EntryStatusChangeEvent` - Ticket entry status changes
  - `DrawReminderEvent` - Lottery draw countdown reminders
  - `RecommendationEvent` - House recommendations
  - `InventoryUpdateEvent` - Real-time ticket inventory updates
  - `CountdownUpdateEvent` - Lottery countdown updates
  - `ReservationStatusUpdateEvent` - Ticket reservation status
- **Reconnection Strategy**:
  - Exponential backoff: 1s, 2s, 4s, 8s, 16s
  - Max retries: 5
  - Automatic reconnection on connection loss
  - User group management handled automatically by backend in `OnConnectedAsync()`
- **Service**: `RealtimeService` manages all SignalR connections and event subscriptions

### Error Handling Patterns
- **Error Handling Service**: `ErrorHandlingService` centralizes all error handling
- **Error Severity Levels**: `low`, `medium`, `high`, `critical`
- **Error Response Format**:
  ```typescript
  {
    success: false,
    error: {
      code: "ERROR_CODE",
      message: "Human-readable message",
      details?: any
    }
  }
  ```
- **Error Code Mapping**: Frontend maps backend error codes to user-friendly messages
- **Toast Notifications**: Critical/high/medium errors show toast notifications
- **Error Logging**: All errors logged with context (userId, request details)
- **401 Handling**: Automatic token refresh attempt on 401 errors
- **Network Errors**: Automatic retry with exponential backoff

### Caching Strategies
- **In-Memory Cache**: Service-level caching (5 min TTL for houses list)
- **localStorage Cache**: Persistent cache for houses, favorites, active entries, user stats
  - Houses list: 30 min TTL
  - Individual houses: 1 hour TTL
  - User favorites: 15 min TTL
  - Active entries: 15 min TTL
  - User stats: 15 min TTL
- **sessionStorage Cache**: Temporary cache for dashboard data (5 min TTL)
- **Cache Invalidation**:
  - Time-based: TTL expiration
  - Event-based: SignalR events invalidate relevant caches
  - User action-based: Add/remove favorite, purchase ticket
  - Authentication-based: Login/logout clears user-specific caches
- **Cache Keys**:
  - Houses list: `houses_list`
  - Individual house: `house:{houseId}`
  - User favorites: `user_favorites:{userId}`
  - Active entries: `user_active_entries:{userId}`
  - User stats: `user_stats:{userId}`
- **Real-Time Data Protection**: Methods like `getAvailableTickets()` are never cached (always fetch fresh)

### Offline Handling
- **Offline Detection**: `OfflineService` monitors network status
- **Request Queuing**: Failed requests queued when offline
- **Queue Persistence**: Queued requests stored in memory (cleared on page refresh)
- **Automatic Retry**: Queued requests automatically retried when network restored
- **Queue Size**: No explicit limit (monitor memory usage)
- **Integration**: `ApiService` uses `OfflineService` for queuing failed requests

### Performance Monitoring
- **Performance Service**: `PerformanceService` tracks application performance metrics
- **Route Performance**: `RoutePerformanceService` tracks route load times
- **Memory Monitoring**: `MemoryMonitorService` tracks memory usage
- **Metrics Collected**:
  - Route load times
  - API call durations
  - Memory usage
  - Component render times

### State Management Patterns
- **Angular Signals** (Primary State Management):
  - **Signal Creation**: `signal<T>(initialValue)` for reactive state
  - **Computed Signals**: `computed(() => derivedValue)` for derived state
  - **Effects**: `effect(() => { sideEffects })` for side effects based on signal changes
  - **Usage Examples**:
    - `showPassword = signal(false)` - UI state (e.g., password visibility)
    - `isVerified = signal(false)` - Verification status
    - `isLoading = signal(true)` - Loading states
    - `effect(() => { /* react to signal changes */ })` - Side effects (e.g., favorites updates)
  - **Location**: Used throughout components and services (e.g., `registration-page.component.ts`, `verification-gate.component.ts`, `lottery-favorites.component.ts`)
- **RxJS Observables** (Async Operations & Streams):
  - **Observable**: For async operations (HTTP calls, timers, event streams)
  - **Subject**: For multicasting values to multiple subscribers
  - **BehaviorSubject**: For state that needs initial value and current value tracking
  - **Operators**: `map`, `filter`, `switchMap`, `catchError`, `retry`, `takeUntil`, etc.
  - **Usage Examples**:
    - HTTP calls return `Observable<T>` (e.g., `getHouseById(id): Observable<HouseDto>`)
    - `Subject<void>` for cleanup/unsubscribe patterns (e.g., `cleanup$ = new Subject<void>()`)
    - `firstValueFrom()` for converting Observable to Promise (e.g., in SignalR service)
  - **Location**: Used in services (e.g., `lottery.service.ts`, `realtime.service.ts`, `api.service.ts`)
- **State Storage**:
  - **localStorage**: Persistent state (user preferences, theme, language)
  - **sessionStorage**: Session-scoped state (temporary data)
  - **In-Memory**: Component/service-level state (signals, observables)
- **State Synchronization**:
  - **Real-time Updates**: SignalR events update local state (e.g., lottery updates, notifications)
  - **Cache Invalidation**: State updates trigger cache invalidation
  - **Cross-Component Communication**: Services with signals/observables shared across components

### Routing Patterns
- **Lazy Loading**:
  - **Method**: `loadComponent()` for route-based code splitting
  - **Benefits**: Reduces initial bundle size, improves load time
  - **Configuration**: Routes configured with `loadComponent` in route definitions
- **Route Guards**:
  - **AuthGuard**: `canActivate()` checks authentication and email verification
    - Checks: `user.isAuthenticated` and `userDto.emailVerified`
    - Redirects: `/` if not authenticated, `/verify-email` if email not verified
    - Toast Notifications: Shows warning/error messages
    - Location: `src/guards/auth.guard.ts`
- **Custom Preloading Strategy**:
  - **Class**: `CustomPreloadingStrategy` implements `PreloadingStrategy`
  - **Priority-Based Delays**:
    - **High Priority** (0ms delay): `about`, `faq`, `help`
    - **Medium Priority** (2s delay): `sponsorship`, `partners`, `how-it-works`
    - **Low Priority** (5s delay): `register`, `member-settings`, `promotions`, `responsible-gambling`
    - **Default** (3s delay): Unknown routes
  - **Purpose**: Preload routes after initial load without blocking critical resources
  - **Location**: `src/app.preloading-strategy.ts`
  - **Registration**: Configured in `main.ts` via `withPreloading(CustomPreloadingStrategy)`
- **Route Configuration**:
  - **Wildcard Routes**: `**` for catch-all (e.g., 404 page)
  - **Route Parameters**: `:id` for dynamic segments (e.g., `/house/:id`)
  - **Query Parameters**: Accessible via `ActivatedRoute.queryParams`
  - **Route Data**: Static data passed via `data` property in route config

## Testing Infrastructure
- **E2E Testing**: Playwright configured with 5 browser projects
  - Chromium (Desktop Chrome)
  - Firefox (Desktop Firefox)
  - WebKit (Desktop Safari)
  - Mobile Chrome (Pixel 5)
  - Mobile Safari (iPhone 12)
- **Unit Testing**: Karma/Jasmine with ChromeHeadless
- **Test Configuration**: 
  - CI test script: `ng test demo --browsers=ChromeHeadless --watch=false --code-coverage=false`
  - Playwright base URL: `http://localhost:4200` (configurable via `BASE_URL` env var)
  - Retry strategy: 2 retries on CI, 0 locally
  - Trace collection: On first retry
  - Screenshots: Only on failure
  - Video: Retain on failure

### Testing Patterns
- **Unit Testing** (Jasmine/Karma):
  - **Framework**: Jasmine for test syntax, Karma for test runner
  - **Test Location**: `*.spec.ts` files alongside source files
  - **Mocking**: 
    - Angular testing utilities (`TestBed`, `ComponentFixture`)
    - Service mocks using `jasmine.createSpy()` or `jasmine.createSpyObj()`
    - HTTP client mocking via `HttpClientTestingModule`
  - **Test Structure**: `describe()` blocks for test suites, `it()` for test cases
  - **Angular Testing Utilities**:
    - `TestBed.configureTestingModule()` for test module setup
    - `ComponentFixture` for component testing
    - `DebugElement` for DOM querying
  - **Coverage**: Target 80%+ code coverage for services and components
- **E2E Testing** (Playwright):
  - **Framework**: Playwright with TypeScript
  - **Page Object Model**: Reusable page objects for test maintainability
  - **Test Data**: 
    - Test fixtures for consistent test data
    - Isolated test data per test to avoid dependencies
  - **Browser Projects**: 5 projects (Chromium, Firefox, WebKit, Mobile Chrome, Mobile Safari)
  - **Test Configuration**:
    - Base URL: `http://localhost:4200` (configurable via `BASE_URL` env var)
    - Retry strategy: 2 retries on CI, 0 locally
    - Trace collection: On first retry
    - Screenshots: Only on failure
    - Video: Retain on failure
  - **Test Organization**: Tests organized by feature/page in `e2e/` directory
  - **Best Practices**:
    - Use data-testid attributes for reliable element selection
    - Wait for network requests to complete
    - Clean up test data after tests

### Application Initialization Patterns
- **Bootstrap Method**: `bootstrapApplication()` from `@angular/platform-browser`
- **Entry Point**: `src/main.ts`
- **Provider Configuration**:
  - **Services**: All core services registered as providers (TranslationService, ThemeService, AuthService, etc.)
  - **Error Handler**: Custom `ErrorHandlingService` via `{ provide: ErrorHandler, useClass: ErrorHandlingService }`
  - **HTTP Client**: `provideHttpClient()` for HTTP requests
  - **Router**: `provideRouter()` with routes, preloading strategy, and in-memory scrolling
  - **Animations**: `provideAnimations()` for Angular animations
- **APP_INITIALIZER Pattern** (Non-blocking startup):
  - **Purpose**: Initialize services before app starts without blocking bootstrap
  - **Translation Initialization** (`initializeTranslations`):
    - Priority: localStorage → browser language → default ('en')
    - Non-blocking: Loads translations in background, app starts immediately
    - Location: `src/main.ts`
  - **Service Initialization** (`initializeServices`):
    - Sets up cross-service dependencies (e.g., `userPreferencesService.setThemeService()`)
    - Loads theme and preferences from localStorage (synchronous, no network)
    - Applies accessibility settings from preferences
    - Non-blocking: All operations synchronous, resolves immediately
  - **Multi-provider**: `multi: true` allows multiple `APP_INITIALIZER` functions
- **Router Configuration**:
  - **Preloading**: `CustomPreloadingStrategy` with priority-based delays
  - **Scrolling**: `withInMemoryScrolling({ scrollPositionRestoration: 'top', anchorScrolling: 'enabled' })`
  - **Routes**: Lazy-loaded routes with `loadComponent()`
- **Console Override**: `./console-override` removes console logs in production
- **Best Practices**:
  - Keep initialization non-blocking (no network requests in APP_INITIALIZER)
  - Use localStorage for fast initial state
  - Network requests happen via service subscriptions after bootstrap
  - Resolve APP_INITIALIZER promises immediately for synchronous operations

### Build & Packaging Patterns

| Aspect | Configuration |
|--------|---------------|
| **Build Config** | `angular.json` (project: `demo`) |
| **Builder** | `@angular/build:application` |
| **Output Path** | `dist/demo/browser/` |
| **Development Build** | Source maps ✅, Named chunks ✅, No optimization, No license extraction |
| **Production Build** | AOT ✅, Full optimization ✅, Output hashing: "all", Source maps ❌, License extraction ✅, `environment.ts` → `environment.prod.ts` |
| **Assets** | `src/assets/` (static), `sw.js` (service worker to root) |
| **Styles** | `src/styles.css`, `src/global_styles.css` |
| **Polyfills** | `zone.js` (Angular change detection) |
| **TypeScript** | `tsconfig.app.json` (strict mode, ES2022) |
| **Bundle Strategy** | Code splitting (lazy loading), Vendor chunks, Common chunks |
| **Deployment** | S3 bucket → CloudFront distribution (cache invalidation required) |

## Common Commands
```bash
# Frontend development
npm install
ng serve                    # Start dev server (http://localhost:4200)
ng build                    # Build for production
ng build --configuration=production
ng test                     # Run unit tests
ng test demo --browsers=ChromeHeadless --watch=false --code-coverage=false  # CI tests
npm run test:ci             # Run CI test script
npx playwright test        # Run E2E tests

# Build output
# Production build outputs to: dist/demo/browser/
# CloudFront origin path configured for: /browser

# AWS operations
aws cloudfront list-distributions
aws s3 ls s3://amesa-frontend-prod/ --recursive
aws cloudfront create-invalidation --distribution-id E3GU3QXUR43ZOH --paths "/*"

# ECS operations
aws ecs list-services --cluster Amesa
aws ecs describe-services --cluster Amesa --services amesa-auth-service

# Database operations
aws rds describe-db-clusters --db-cluster-identifier amesadbmain

# Git operations
git status
git log --oneline -5
git checkout main
```

## Business Context
- **4Wins Model**: Winner gets property, community gets support, company gets sustainable business, society gets social impact
- **Legal Partners**: Zeiba & Partners (legal), PiK Podatki (accounting)
- **Responsible Gaming**: Built-in responsible gambling features
- **Multi-language Support**: Internationalization with translation service
- **Theme Support**: Dark/light mode switching
- **Real-time Features**: SignalR integration for live updates

## Cursor Cost Optimization - Frontend Component Refactoring Status

**Phase 4 Progress**: ~60% Complete (6/10+ large components refactored)

### ✅ Completed Component Refactorings:
1. **HouseCarouselComponent** - 924 → 819 lines (11% reduction)
   - Created `HouseCarouselService` for carousel state and image loading
2. **RegistrationPageComponent** - 867 → 651 lines (25% reduction)
   - Created `RegistrationFormService` for form validation and state management
3. **MemberSettingsPageComponent** - 818 → 721 lines (12% reduction)
   - Created `MemberSettingsService` for settings management
4. **HouseCardComponent** - 706 → 587 lines (17% reduction)
   - Created `HouseCardService` for formatting and purchase logic
5. **UserPreferencesComponent** - 688 → 515 lines (25% reduction)
   - Created `UserPreferencesFormService` for form logic
6. **AccessibilityWidgetComponent** - 662 → 442 lines (33% reduction)
   - Created `AccessibilityWidgetService` for state and settings application

### ⚪ Remaining Components:
- Other large components (hero-section, topbar, etc.) - Not yet refactored

**Expected Savings**: $12-18/month achieved, $8-12/month remaining potential

---

## Recent Changes Summary

| Change | Status | Date | Key Points |
|--------|--------|------|------------|
| **Notification Bell** | ✅ Complete | 2025-01-25 | Sidebar component, ALB routing configured, SignalR integration |
| **Translation Performance** | ✅ Fixed | 2025-01-25 | Removed blocking delay, fixed race condition, ~1s faster startup |
| **Image Gallery** | ✅ Fixed | 2025-01-25 | Thumbnails/dots navigation, ARIA labels added |
| **System Enhancement** | ✅ 100% Complete | 2025-01-25 | Locale formatting (10 components, 3 services), Accessibility fixes |
| **Payment UI** | ✅ A+ Quality | 2025-12-20 | 8 components, 3 services, all issues resolved, production-ready |
| **Payment Consolidated Step** | ✅ Production Ready | 2025-12-20 | Audit complete, all critical/high/medium issues fixed, 99.9% complete |

**Detailed Documentation**: See `../MetaData/Documentation/` for complete audit reports and session documentation.

**Last Updated**: 2025-12-20
**Context Version**: 2.5.0 (Payment Consolidated Step Component audit added)
**Note**: This file should be updated whenever project structure, environment configuration, infrastructure, or deployment processes change.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DrorGr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
