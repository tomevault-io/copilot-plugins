## prebidjs-light

> This document contains critical information for working with the pbjs_engine Prebid wrapper platform. Follow these guidelines precisely.

# Development Guidelines for pbjs_engine

This document contains critical information for working with the pbjs_engine Prebid wrapper platform. Follow these guidelines precisely.

## Project Overview

pbjs_engine is a Prebid.js wrapper platform that provides server-managed configurations for publishers. The platform enables publishers to embed a lightweight JavaScript wrapper on their sites that automatically loads Prebid.js with centrally managed bidder configurations.

### Technology Stack

**Backend (API Server):**
- Node.js 20 LTS with TypeScript
- Fastify web framework
- Drizzle ORM for type-safe database access
- SQLite for data storage (PostgreSQL-compatible schema)
- JWT authentication with bcrypt password hashing

**Frontend (Admin Portal):**
- React 18 with TypeScript
- Vite for build tooling
- Tailwind CSS for styling
- Zustand for state management
- React Router for navigation

**Publisher Wrapper:**
- TypeScript compiled to ES2015
- Webpack with Terser for minification
- Lightweight (<2KB gzipped) JavaScript library
- CDN-ready with cache control headers

### Architecture

```
pbjs_engine/
├── apps/
│   ├── api/              # Fastify backend server
│   │   ├── src/
│   │   │   ├── db/       # Database schema and migrations
│   │   │   ├── routes/   # API route handlers
│   │   │   └── index.ts  # Server entry point
│   │   └── data/         # SQLite database files
│   ├── admin/            # React admin portal
│   │   ├── src/
│   │   │   ├── components/  # UI components
│   │   │   ├── pages/       # Page components
│   │   │   ├── stores/      # Zustand state stores
│   │   │   └── App.tsx
│   │   └── dist/         # Production build output
│   └── wrapper/          # Publisher wrapper script
│       ├── src/
│       │   └── pb.ts     # Wrapper source code
│       └── dist/         # Minified production build
└── docs/                 # Documentation
```

## Core Development Rules

### 1. Language & Framework Standards

**Backend:**
- **Language**: TypeScript 5.3+ with strict mode enabled
- **Runtime**: Node.js 20 LTS
- **Web Framework**: Fastify for high-performance HTTP handling
- **Code Style**: Prettier for formatting, ESLint for linting
- **Error Handling**: Use try-catch with proper error responses
- **Testing**: Vitest for unit and integration tests
- **Naming**: Follow conventions in `NAMING_CONVENTIONS.md`

**Frontend:**
- **Language**: TypeScript with React JSX
- **Framework**: React 18 with functional components and hooks
- **Styling**: Tailwind CSS utility classes
- **State**: Zustand for global state, React hooks for local state
- **Forms**: Controlled components with validation
- **Naming**: Follow conventions in `NAMING_CONVENTIONS.md`

### 2. Database Guidelines

**Technology**: SQLite with Drizzle ORM

**Schema Design:**
- Use proper foreign key constraints
- Implement soft deletes with `deleted_at` timestamp
- Use UUIDs for primary keys
- Maintain audit fields: `created_at`, `updated_at`, `deleted_at`

**Data Hierarchy:**
```
Publisher → Website → Ad Unit
          ↓
       Bidder Configuration
```

**Migrations:**
```bash
# Create migration script in apps/api/src/db/
# Run with: npm run migrate
```

**Query Patterns:**
```typescript
// Use Drizzle ORM for type safety
import { db } from '../db';
import { publishers, websites, adUnits } from '../db/schema';
import { eq, and, isNull } from 'drizzle-orm';

// Fetch with soft delete filter
const activePublishers = db.select()
  .from(publishers)
  .where(isNull(publishers.deletedAt))
  .all();

// Join query
const websiteWithAdUnits = db.select()
  .from(websites)
  .leftJoin(adUnits, eq(websites.id, adUnits.websiteId))
  .where(eq(websites.publisherId, publisherId))
  .all();
```

### 3. API Design

**RESTful Conventions:**
- Use plural nouns for resources (`/publishers`, `/websites`)
- Hierarchical routes for relationships (`/websites/:id/ad-units`)
- Proper HTTP methods (GET, POST, PUT, DELETE)
- Use HTTP status codes correctly (200, 201, 400, 401, 404, 500)

**Authentication:**
```typescript
// JWT-based authentication
// Token required in Authorization header: Bearer <token>
// Role-based access: super_admin, admin, publisher
```

**Route Structure:**
```
/api/auth/login              # Authentication
/api/auth/forgot-password    # Password reset
/api/publishers              # Publisher CRUD
/api/publishers/:id          # Single publisher
/api/websites/:id/ad-units   # Hierarchical ad units
/api/system/health           # System monitoring
/pb.min.js                   # Wrapper script (no /api prefix)
/c/:publisherId              # Public config endpoint
```

**Response Format:**
```typescript
// Success
{ data: T, message?: string }

// Error
{ error: string, details?: any }

// Pagination
{ data: T[], total: number, page: number, limit: number }
```

### 4. Frontend Patterns

**Component Structure:**
```typescript
// Functional component with TypeScript
interface Props {
  id: string;
  onSave: (data: FormData) => void;
}

export function ComponentName({ id, onSave }: Props) {
  const [state, setState] = useState<StateType>(initialState);

  useEffect(() => {
    // Side effects
  }, [dependencies]);

  return (
    <div className="tailwind-classes">
      {/* JSX */}
    </div>
  );
}
```

**State Management:**
```typescript
// Zustand store (apps/admin/src/stores/)
import { create } from 'zustand';

interface AuthStore {
  user: User | null;
  token: string | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
}

export const useAuthStore = create<AuthStore>((set) => ({
  user: null,
  token: localStorage.getItem('token'),
  login: async (email, password) => { /* ... */ },
  logout: () => { /* ... */ },
}));
```

**Data Fetching:**
```typescript
// Use fetch with proper error handling
const fetchData = async () => {
  try {
    const response = await fetch('/api/resource', {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });

    if (!response.ok) {
      throw new Error('Failed to fetch');
    }

    const data = await response.json();
    setData(data);
  } catch (err) {
    console.error(err);
    setError(err.message);
  }
};
```

### 5. Wrapper Development

**Purpose**: Lightweight JavaScript library that publishers embed to load Prebid.js with server-managed configuration.

**Build Process:**
```bash
cd apps/wrapper
npm install
npm run build      # Production build (minified)
npm run build:dev  # Development build (readable)
npm run dev        # Watch mode
```

**Integration:**
```html
<!-- Publisher embeds this script -->
<script src="https://cdn.example.com/pb.min.js" async></script>
<script>
  window.pb = window.pb || { que: [] };
  window.pb.que.push(function() {
    pb.init().then(() => console.log('Prebid ready!'));
  });
</script>
```

**API Surface:**
- `pb.init()` - Initialize with server config
- `pb.refresh([codes])` - Refresh ad units
- `pb.getConfig()` - Get current configuration
- `pb.setConfig(config)` - Update configuration
- `pb.on(event, callback)` - Subscribe to events
- `pb.off(event, callback)` - Unsubscribe from events

### 6. Security Standards

**Authentication:**
- JWT tokens with expiration
- Bcrypt password hashing (salt rounds: 10)
- Secure password reset flow with expiring tokens

**CORS:**
- Configured for cross-origin requests
- Allows credentials for cookie-based sessions

**Input Validation:**
- Validate all user inputs on backend
- Sanitize data before database insertion
- Use TypeScript types for compile-time safety

**Secrets Management:**
- Store secrets in environment variables
- Never commit `.env` files
- Use different secrets for dev/prod

**API Security:**
- Rate limiting (100 requests per minute)
- JWT token validation on protected routes
- Role-based access control (RBAC)

### 7. Naming Conventions

**IMPORTANT**: Follow the comprehensive naming standards documented in `NAMING_CONVENTIONS.md`.

**Quick Reference:**

**Variables:**
- Use **camelCase** for standard variables: `userId`, `apiEndpoint`
- Prefix booleans with `is`, `has`, `should`, or `can`: `isActive`, `hasPermission`
- Use **SCREAMING_SNAKE_CASE** for constants: `API_BASE_URL`, `MAX_RETRY_ATTEMPTS`

**Functions:**
- Use **camelCase** with verb-based names: `fetchPublishers()`, `validateInput()`
- Prefix async functions with action verbs: `fetchUserData()`, `loadConfiguration()`
- Prefix boolean-returning functions: `isValidUUID()`, `hasPermission()`
- Prefix event handlers: `handleClick()`, `onSubmit()`

**Classes & Interfaces:**
- Use **PascalCase** for classes: `UserService`, `DataRepository`
- Use **PascalCase** for interfaces (no `I` prefix): `User`, `PublisherConfig`
- Use **PascalCase** for type aliases: `UserId`, `PublisherStatus`

**Files:**
- Use **kebab-case** for utilities: `safe-json.ts`, `error-handler.ts`
- Use **PascalCase** for React components: `PublisherCard.tsx`, `WebsiteModal.tsx`
- Use **kebab-case** or singular for routes: `auth.ts`, `publishers.ts`

**Database:**
- Use **snake_case** for table names (plural): `publishers`, `ad_units`
- Use **snake_case** for column names: `user_id`, `created_at`

**API Routes:**
- Use **kebab-case** and plurals: `/api/publishers`, `/api/ad-units`
- Use **camelCase** for query params: `?page=1&sortBy=name`

**React:**
- Use **PascalCase** for components: `PublisherCard`, `WebsiteModal`
- Suffix props interfaces with `Props`: `PublisherCardProps`
- Prefix hooks with `use`: `useAuth()`, `usePublishers()`

See `NAMING_CONVENTIONS.md` for complete details and examples.

### 8. Testing

**Backend Tests:**
```bash
cd apps/api
npm test              # Run all tests
npm run test:watch    # Watch mode
npm run test:coverage # Coverage report
```

**Frontend Tests:**
```bash
cd apps/admin
npm test
```

**Integration Tests:**
- Test API endpoints with actual database
- Use test database instance
- Clean up test data after each run

### 8. Performance Optimization

**Backend:**
- Use connection pooling for database
- Implement caching for frequently accessed data
- Optimize database queries with proper indexes
- Use async/await for non-blocking operations

**Frontend:**
- Code splitting with React.lazy()
- Optimize bundle size with Vite
- Use React.memo for expensive components
- Debounce user inputs for search/filter

**Wrapper:**
- Keep bundle size minimal (<2KB gzipped)
- Use ES2015 target for modern browsers
- Minify with Terser
- Enable gzip compression on CDN

### 9. Deployment

**Development:**
```bash
# Run all services
npm run dev

# Backend only
cd apps/api && npm run dev

# Frontend only
cd apps/admin && npm run dev
```

**Production Build:**
```bash
# Backend
cd apps/api && npm run build

# Frontend
cd apps/admin && npm run build

# Wrapper
cd apps/wrapper && npm run build
```

**Environment Variables:**
```bash
# Backend (.env in apps/api/)
NODE_ENV=production
API_PORT=3001
JWT_SECRET=<secure-random-secret>
COOKIE_SECRET=<secure-random-secret>
DATABASE_URL=file:./data/pbjs_engine.db

# Frontend (.env in apps/admin/)
VITE_API_URL=https://api.example.com
```

**Health Checks:**
- `/health` - Basic API health
- `/api/system/health` - Detailed system status

## Common Workflows

### Adding a New Feature

1. **Create database migration** (if needed)
   ```typescript
   // apps/api/src/db/migrations/add-feature.ts
   ```

2. **Update schema**
   ```typescript
   // apps/api/src/db/schema.ts
   export const newTable = sqliteTable('new_table', { ... });
   ```

3. **Create API routes**
   ```typescript
   // apps/api/src/routes/feature.ts
   export default async function featureRoutes(fastify: FastifyInstance) {
     fastify.get('/feature', handler);
   }
   ```

4. **Register routes**
   ```typescript
   // apps/api/src/index.ts
   import featureRoutes from './routes/feature';
   app.register(featureRoutes, { prefix: '/api/feature' });
   ```

5. **Create frontend page**
   ```typescript
   // apps/admin/src/pages/admin/FeaturePage.tsx
   export function FeaturePage() { ... }
   ```

6. **Add route**
   ```typescript
   // apps/admin/src/App.tsx
   <Route path="feature" element={<FeaturePage />} />
   ```

### Debugging

**Backend Logging:**
```typescript
fastify.log.info('Info message');
fastify.log.error('Error message', err);
```

**Frontend Console:**
```typescript
console.log('Debug info:', data);
console.error('Error:', err);
```

**Database Inspection:**
```bash
cd apps/api/data
sqlite3 pbjs_engine.db
.schema          # Show schema
.tables          # List tables
SELECT * FROM publishers;
```

### Common Issues

1. **CORS errors**: Check CORS configuration in apps/api/src/index.ts
2. **Auth failures**: Verify JWT_SECRET matches between client/server
3. **Database locked**: SQLite allows only one writer - check for concurrent writes
4. **Missing dependencies**: Run `npm install` in each app directory

## Best Practices

1. **Type Safety**: Use TypeScript strictly, avoid `any` types
2. **Error Handling**: Always handle errors gracefully with user-friendly messages
3. **Code Reuse**: Extract common logic into shared utilities
4. **Documentation**: Comment complex logic, keep README files updated
5. **Git Commits**: Use conventional commit messages (feat, fix, docs, etc.)
6. **Code Reviews**: Review all changes before merging
7. **Testing**: Write tests for critical business logic
8. **Performance**: Profile and optimize hot paths

## Project-Specific Notes

### Data Taxonomy

The system uses a three-tier hierarchy:
- **Publisher**: Top-level account (e.g., "News Corp")
- **Website**: Domain under publisher (e.g., "news.com")
- **Ad Unit**: Specific ad placement on website (e.g., "header-banner")

### Soft Deletes

All main tables use soft deletes:
```typescript
deletedAt: text('deleted_at') // NULL = active, timestamp = deleted
```

Always filter by `deleted_at IS NULL` when querying active records.

### UUID Generation

```typescript
import { v4 as uuidv4 } from 'uuid';
const id = uuidv4();
```

### Wrapper Distribution

The wrapper is served at:
- `/pb.min.js` - Generic wrapper
- `/pb/:apiKey.js` - Publisher-specific wrapper
- `/pb/info` - Build information

Configuration endpoint:
- `/c/:publisherId` - Public config (cached 5 minutes)

Remember: This is a management platform for Prebid.js configurations, not a real-time bidding exchange. Focus on configuration management, not auction performance.

## Working with AI Assistants (Claude)

### Documentation First

When working with Claude or other AI assistants:

1. **Always Update Documentation**: After implementing features or fixing bugs, update relevant documentation files immediately
2. **Learn from Mistakes**: Document incorrect assumptions or architecture mismatches to prevent future confusion
3. **Create Clarity**: If specs from other projects get mixed in, add clear warnings at the top of files
4. **Architecture Notes**: Maintain ARCHITECTURE_NOTES.md for project-specific decisions and clarifications

### Using Agents Effectively

**When to Use Task Agents:**
- Complex multi-step implementations requiring >5 tool calls
- Research tasks that need exploration across multiple files
- Specialized tasks (testing, security audits, optimization)

**Agent Types Available:**
- `Explore` - For codebase exploration and research
- `Plan` - For implementation strategy design
- `general-purpose` - For complex multi-step tasks

**Example:**
```
Instead of manually searching 20+ files, use:
Task tool with subagent_type=Explore to find all error handling patterns
```

### Continuous Improvement

**After Every Feature:**
1. ✅ Update CLAUDE.md with new patterns/conventions
2. ✅ Document any architecture decisions
3. ✅ Add examples to help future development
4. ✅ Update README.md if user-facing changes
5. ✅ Create/update migration guides if needed

**After Every Bug Fix:**
1. ✅ Document root cause in commit message
2. ✅ Update relevant documentation to prevent recurrence
3. ✅ Add test cases to catch similar issues
4. ✅ Share learnings in team docs

**Learning from Mistakes:**
- If incorrect specs are used → Add warnings to prevent confusion
- If architecture mismatch occurs → Document the correct stack
- If dependencies break → Document version requirements
- If deployment fails → Update deployment documentation

### Documentation Standards

**Keep These Files Updated:**
- `CLAUDE.md` - Development guidelines (this file)
- `README.md` - Project overview and setup
- `NAMING_CONVENTIONS.md` - Naming standards and conventions
- `ARCHITECTURE_NOTES.md` - Architecture decisions and clarifications
- Feature-specific `.md` files - Implementation details

**When Adding Features:**
Create documentation that includes:
- Purpose and use cases
- Technical implementation details
- API endpoints (if applicable)
- Frontend components (if applicable)
- Example usage
- Testing approach
- Known limitations

**Template for Feature Documentation:**
```markdown
# Feature Name

## Purpose
What problem does this solve?

## Architecture
How is it built? (Backend + Frontend)

## Implementation
Key files and their roles

## Usage
How do users interact with it?

## API Endpoints
List all relevant endpoints

## Testing
How to test this feature

## Future Enhancements
What could be added later?
```

### Proactive Improvements

Encourage Claude to:
- ✅ Suggest better patterns when seeing repetitive code
- ✅ Identify missing error handling
- ✅ Recommend performance optimizations
- ✅ Point out security concerns
- ✅ Suggest documentation improvements
- ✅ Create specialized agents for complex tasks
- ✅ Update documentation without being asked

**Claude Should:**
- Ask clarifying questions before implementing ambiguous requirements
- Verify architecture assumptions before major changes
- Suggest creating agents for exploration or complex tasks
- Update documentation as part of every implementation
- Learn from past mistakes documented in the codebase

## Phase 2 Features

Phase 2 adds advanced configuration management capabilities. See `PHASE2_FEATURES.md` for detailed documentation.

**Implemented:**
- ✅ **Parameter Configuration (Feature 1)** - Dynamic forms for configuring bidder/module/analytics parameters
  - Backend: `/apps/api/src/routes/component-parameters.ts`
  - Frontend: `/apps/admin/src/components/ComponentConfigModal.tsx`, `DynamicParameterForm.tsx`, `ParameterField.tsx`
  - Database: `component_parameters`, `component_parameter_values` tables
  - Schemas: `/apps/api/src/utils/prebid-markdown-parser.ts`

**Planned:**
- ⏳ **Enhanced Analytics (Feature 2)** - Rich dashboards with trends, heatmaps, geo analytics
- ⏳ **Prebid.js Build System (Feature 3)** - Custom bundle generation with selected components
- ⏳ **Bulk Operations & Templates (Feature 4)** - Templates, bulk operations, import/export

### Parameter Configuration Patterns

**Adding New Parameter Schemas:**

1. Edit `/apps/api/src/utils/prebid-markdown-parser.ts`
2. Add to BIDDER_SCHEMAS, MODULE_SCHEMAS, or ANALYTICS_SCHEMAS:

```typescript
rubicon: [
  {
    name: 'accountId',
    type: 'number',
    required: true,
    description: 'The publisher account ID',
    validation: { min: 1 },
  },
  // ... more parameters
],
```

3. Restart server - schemas auto-seed on startup

**Using Configuration Modal in Pages:**

```typescript
import ComponentConfigModal from '../../components/ComponentConfigModal';

// Add state
const [configModalOpen, setConfigModalOpen] = useState(false);
const [selectedComponent, setSelectedComponent] = useState<ComponentType | null>(null);

// Add button
<button onClick={() => {
  setSelectedComponent(component);
  setConfigModalOpen(true);
}}>
  Configure
</button>

// Add modal
<ComponentConfigModal
  isOpen={configModalOpen}
  onClose={() => setConfigModalOpen(false)}
  componentType="bidder" // or "module" or "analytics"
  componentCode={selectedComponent.code}
  componentName={selectedComponent.name}
  publisherId={publisherId}
/>
```

---
> Source: [q9void/prebidjs-light](https://github.com/q9void/prebidjs-light) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
