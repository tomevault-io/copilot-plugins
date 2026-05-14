## openstory

> AI-powered video sequence platform built with TanStack Start, optimized for edge deployment.

# CLAUDE.md

AI-powered video sequence platform built with TanStack Start, optimized for edge deployment.

## Architecture Overview

**Tech Stack:**

- **Runtime**: Bun (not Node.js)
- **Framework**: TanStack Start + TanStack Router + Vite
- **Database**: Turso (libSQL/SQLite) + Drizzle ORM
- **Workflows**: QStash (durable execution for AI tasks)
- **Storage**: Cloudflare R2 (S3-compatible)
- **Auth**: Better Auth
- **Styling**: Tailwind v4 + shadcn/ui
- **Testing**: Bun test

**Core Principles:**

- Database access ONLY in server handlers (never in components)
- Anonymous-first → upgrade to save work
- Team-based resources (sequences, styles, characters)
- Script-driven generation for consistency

**Data Model:**

```
teams
  ├── users (members)
  ├── sequences (videos)
  │   └── frames (scenes with metadata)
  └── libraries (styles, characters, vfx, audio)
```

---

## Setup

```bash
bun install
bun setup                          # Auto-configure local dev (SQLite + QStash)
bun db:setup                       # Migrate + seed database
```

**Daily workflow (2 terminals):**

- Terminal 1: `bun qstash:dev` (async job processing)
- Terminal 2: `bun dev`

**Before commit:** Lefthook auto-checks quality. Branch `123-feature` → commits tagged `#123`.

---

## Server Handler Pattern

All API routes use TanStack Start server handlers:

```typescript
// src/routes/api/example/$id.ts
import { createFileRoute } from '@tanstack/react-router';
import { json } from '@tanstack/react-start';
import { requireUser } from '@/lib/auth/action-utils';
import { handleApiError } from '@/lib/errors';

export const Route = createFileRoute('/api/example/$id')({
  server: {
    handlers: {
      POST: async ({ params, request }) => {
        try {
          // 1. Validate input
          const input = schema.parse(await request.json());

          // 2. Check auth/team permissions
          const user = await requireUser();

          // 3. Execute business logic (DB operations ONLY here)
          const record = await db.insert(table).values({
            ...input,
            teamId: user.teamId,
          });

          // 4. Trigger workflows for async AI tasks
          const { messageId } = await qstash.publishJSON({
            url: `${getQStashWebhookUrl()}/workflows/image`,
            body: { userId: user.id, teamId: user.teamId, ...input },
          });

          // 5. Return standardized response
          return json({ id: record.id, workflowRunId: messageId });
        } catch (error) {
          const handledError = handleApiError(error);
          return json(
            { success: false, error: handledError.toJSON() },
            { status: handledError.statusCode }
          );
        }
      },
    },
  },
});
```

---

## Workflow Pattern

**Triggering workflows (from server handlers):**

```typescript
// ❌ WRONG - Direct fetch() calls don't include QStash signatures
await fetch('/api/workflows/image', {
  method: 'POST',
  body: JSON.stringify(data),
});

// ✅ CORRECT - Use qstash.publishJSON() for proper signatures
const qstash = getQStashClient();
const { messageId } = await qstash.publishJSON({
  url: `${getQStashWebhookUrl()}/workflows/image`, // External URL QStash can reach
  body: { userId, teamId, prompt, ...params },
});
const workflowRunId = messageId;
```

**Implementing workflows (TanStack Start + serveMany):**

```typescript
// src/routes/api/workflows/$.ts - Register with serveMany
import { createFileRoute } from '@tanstack/react-router';
import { serveMany } from '@upstash/workflow/tanstack';

const handler = serveMany({
  image: generateImageWorkflow,
  motion: generateMotionWorkflow,
  storyboard: generateStoryboardWorkflow,
});

export const Route = createFileRoute('/api/workflows/$')({
  server: {
    handlers: {
      POST: async ({ request }) => {
        return handler.POST({ request });
      },
    },
  },
});

// Individual workflow (src/lib/workflows/image-workflow.ts)
export const generateImageWorkflow = async (
  context: WorkflowContext<ImageWorkflowInput>
) => {
  const input = context.requestPayload;
  validateWorkflowAuth(input); // Check userId/teamId passed through context

  const result = await context.run('generate-image', async () => {
    // Step logic - automatically retried on failure
    const image = await generateImage(input.prompt);

    // Update database directly
    await db
      .update(frames)
      .set({ thumbnailUrl: image.url })
      .where(eq(frames.id, input.frameId));

    return { imageUrl: image.url };
  });

  return result;
};
```

**Key principles:**

- Workflows handle their own state (no DB job tracking needed)
- Pass auth (userId/teamId) through workflow context
- Steps are durable - execution continues even if server restarts
- Update DB records directly in workflow steps

---

## Frame System

Frames are the core content unit - each represents one scene from script analysis.

**Frame Structure:**

- `thumbnailUrl` - Generated image
- `videoUrl` - Motion video (image-to-video)
- `metadata` - Complete `Scene` object (typed JSONB)

**Frame.metadata IS the Scene object** (no wrapper):

```typescript
// src/lib/ai/frame.schema.ts
frame.metadata = {
  sceneId: string,
  sceneNumber: number,
  originalScript: { extract, lineNumber, dialogue },
  metadata: { title, durationSeconds, location, timeOfDay, storyBeat },
  variants: { cameraAngles, movementStyles, moodTreatments }, // A/B/C options
  selectedVariant: { cameraAngle, movementStyle, moodTreatment, rationale },
  prompts: {
    visual: { fullPrompt, negativePrompt, components, parameters },
    motion: { fullPrompt, components, parameters },
  },
  continuity: { characterTags, environmentTag, colorPalette, lightingSetup },
  musicDesign: { presence, style, mood, atmosphere },
};
```

**Working with frames:**

```typescript
import { frameService } from '@/lib/services/frame.service';

// Get scene data (metadata IS the scene)
const scene = frameService.getSceneData(frame);

// Get prompts for regeneration
const visualPrompt = frameService.getVisualPrompt(frame);
const motionPrompt = frameService.getMotionPrompt(frame);

// Access directly (fully typed!)
const sceneTitle = frame.metadata.metadata.title;
const fullPrompt = frame.metadata.prompts.visual.fullPrompt;
```

**Benefits:**

- Complete scene data enables regeneration without re-analyzing script
- Variants preserved for trying different creative options
- Type-safe with Drizzle's typed JSONB

---

## Fal.ai Integration

**Always verify API specs before updating models:**

```bash
# Get authoritative parameter specifications
https://fal.ai/models/{model-path}/llms.txt

# Examples
https://fal.ai/models/fal-ai/kling-video/v2.5-turbo/pro/image-to-video/llms.txt
https://fal.ai/models/fal-ai/fast-svd-lcm/llms.txt
```

**Why `/llms.txt`?**

- Machine-readable, authoritative specs
- Includes all parameters with types, defaults, constraints
- More reliable than HTML docs
- Essential for accurate `src/lib/ai/models.ts` definitions

**Motion generation status checking:**

```typescript
// Generate motion
const result = await generateMotionForFrame({
  imageUrl: 'https://example.com/image.jpg',
  prompt: 'Camera pan left',
  model: 'wan_i2v',
});
// result.requestId, result.statusUrl, result.responseUrl, result.cancelUrl

// Check status
import {
  checkMotionStatus,
  getMotionResult,
  cancelMotionGeneration,
} from '@/lib/services/motion.service';

const status = await checkMotionStatus(result.statusUrl);
// status.status: "IN_QUEUE" | "IN_PROGRESS" | "COMPLETED"

const video = await getMotionResult(result.responseUrl);
await cancelMotionGeneration(result.cancelUrl);
```

**CLI status checking:**

```bash
bun scripts/check-motion-status.ts <status-url>
bun scripts/check-motion-status.ts --result <response-url>
bun scripts/check-motion-status.ts --cancel <cancel-url>
```

---

## Database Patterns

**Schema management:**

```bash
bun db:generate  # Generate migrations from schema changes
bun db:migrate   # Apply migrations to local.db
```

**Key conventions:**

- Schema in `src/lib/db/schema/` (Drizzle auto-infers types)
- **NEVER** manually write migration SQL files
- **ULID** primary keys (not UUID)
- **Typed JSONB**: `frame.metadata` typed as `Scene`
- **DB access ONLY in server handlers** (never in components)

---

## React Patterns

### Data Fetching

```tsx
// ❌ BAD - useState + useEffect
import { useEffect, useState } from 'react';

export default function UserProfile({ userId }: { userId: string }) {
  const [user, setUser] = useState(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    fetch(`/api/users/${userId}`)
      .then((r) => r.json())
      .then((data) => {
        setUser(data);
        setIsLoading(false);
      });
  }, [userId]);

  if (isLoading) return <div>Loading...</div>;
  return <div style={{ fontSize: 16, color: '#333' }}>{user.name}</div>;
}

// ✅ GOOD - TanStack Query + Suspense + vanilla TS logic
import { Suspense } from 'react';
import { useQuery } from '@tanstack/react-query';
import { Skeleton } from '@/components/ui/skeleton';
import { formatUserName } from '@/lib/users/format'; // vanilla TS function

const UserProfileContent: React.FC<{ userId: string }> = ({ userId }) => {
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId), // vanilla TS function
    suspense: true, // No isLoading checks needed
  });

  return <div className="text-base">{formatUserName(user)}</div>;
};

export const UserProfile: React.FC<{ userId: string }> = ({ userId }) => {
  return (
    <Suspense fallback={<Skeleton className="h-6 w-32" />}>
      <UserProfileContent userId={userId} />
    </Suspense>
  );
};
```

### Styling

```tsx
// ❌ BAD - Excessive inline Tailwind, hard-coded colors
export default function FrameCard({ frame, onSelect }) {
  return (
    <div
      className="w-[300px] h-[200px] m-4 p-6 bg-white dark:bg-slate-900 text-slate-900 dark:text-white rounded-xl shadow-lg hover:shadow-xl transition-shadow border border-slate-200 dark:border-slate-700"
      onClick={onSelect}
    >
      <h3 className="text-xl font-bold mb-2 text-slate-900 dark:text-white">
        {frame.title}
      </h3>
      <p className="text-sm text-slate-600 dark:text-slate-400">
        {frame.description}
      </p>
    </div>
  );
}

// ✅ GOOD - shadcn/ui base components + layout-only Tailwind
import {
  Card,
  CardHeader,
  CardTitle,
  CardDescription,
} from '@/components/ui/card';

type FrameCardProps = {
  frame: Frame;
  onSelect?: () => void;
};

export const FrameCard: React.FC<FrameCardProps> = ({ frame, onSelect }) => {
  return (
    <Card onClick={onSelect} className="cursor-pointer">
      <CardHeader>
        <CardTitle>{frame.title}</CardTitle>
        <CardDescription>{frame.description}</CardDescription>
      </CardHeader>
    </Card>
  );
};

// Views use Tailwind ONLY for layout
export const FrameGrid: React.FC<{ frames: Frame[] }> = ({ frames }) => {
  return (
    <div className="grid grid-cols-3 gap-4">
      {frames.map((frame) => (
        <FrameCard key={frame.id} frame={frame} />
      ))}
    </div>
  );
};
```

**Why this is better:**

- Base `Card` component handles theming, colors, shadows automatically
- Tailwind used ONLY for layout (grid, gap, flex)
- Dark mode, hover states come from theme
- Easy to maintain - change theme, not every component

### Complex State Management

```tsx
// ❌ BAD - Multiple useState, scattered logic
import { useEffect, useState } from 'react';

function FrameEditor({ frameId }: { frameId: string }) {
  const [frame, setFrame] = useState(null);
  const [isLoading, setIsLoading] = useState(true);
  const [isEditing, setIsEditing] = useState(false);
  const [prompt, setPrompt] = useState('');
  const [style, setStyle] = useState('default');

  useEffect(() => {
    setIsLoading(true);
    fetch(`/api/frames/${frameId}`)
      .then((r) => r.json())
      .then((data) => {
        setFrame(data);
        setIsLoading(false);
      });
  }, [frameId]);

  if (isLoading) return <div>Loading...</div>;
  // ... complex update logic scattered across handlers
}

// ✅ GOOD - Suspense + reducer for complex state
import { Suspense, useReducer } from 'react';
import { useQuery } from '@tanstack/react-query';
import { frameReducer, initialState } from './frame-editor.reducer'; // vanilla TS
import { Skeleton } from '@/components/ui/skeleton';

type FrameEditorProps = {
  frameId: string;
  onUpdate?: (state: EditorState) => void;
};

const FrameEditorContent: React.FC<FrameEditorProps> = ({
  frameId,
  onUpdate,
}) => {
  const { data: frame } = useQuery({
    queryKey: ['frame', frameId],
    queryFn: () => fetchFrame(frameId),
    suspense: true,
  });

  const [state, dispatch] = useReducer(frameReducer, initialState);

  return (
    <div className="flex flex-col gap-4">
      <h2>{frame.title}</h2>
      {/* ... */}
    </div>
  );
};

export const FrameEditor: React.FC<FrameEditorProps> = (props) => {
  return (
    <Suspense fallback={<Skeleton className="h-96 w-full" />}>
      <FrameEditorContent {...props} />
    </Suspense>
  );
};
```

**Reducer pattern (vanilla TypeScript):**

```typescript
// frame-editor.reducer.ts - Pure vanilla TypeScript
export type EditorState = {
  isEditing: boolean;
  prompt: string;
  style: string;
  isDirty: boolean;
};

export type EditorAction =
  | { type: 'START_EDIT' }
  | { type: 'UPDATE_PROMPT'; payload: string }
  | { type: 'CHANGE_STYLE'; payload: string }
  | { type: 'RESET' };

export const initialState: EditorState = {
  isEditing: false,
  prompt: '',
  style: 'default',
  isDirty: false,
};

export function frameReducer(
  state: EditorState,
  action: EditorAction
): EditorState {
  switch (action.type) {
    case 'START_EDIT':
      return { ...state, isEditing: true };
    case 'UPDATE_PROMPT':
      return { ...state, prompt: action.payload, isDirty: true };
    case 'CHANGE_STYLE':
      return { ...state, style: action.payload, isDirty: true };
    case 'RESET':
      return initialState;
    default:
      return state;
  }
}
```

### Forms

```tsx
// ❌ BAD - Controlled inputs everywhere, manual validation
import { useState } from 'react';

function ScriptForm() {
  const [title, setTitle] = useState('');
  const [content, setContent] = useState('');
  const [errors, setErrors] = useState({});

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (!title) {
      setErrors({ title: 'Required' });
      return;
    }
    await fetch('/api/scripts', {
      method: 'POST',
      body: JSON.stringify({ title, content }),
    });
  };

  return (
    <form onSubmit={handleSubmit}>
      <input value={title} onChange={(e) => setTitle(e.target.value)} />
      {errors.title && <span>{errors.title}</span>}
    </form>
  );
}

// ✅ GOOD - TanStack Query mutation + Zod validation
import { useMutation } from '@tanstack/react-query';
import { Button } from '@/components/ui/button';
import { Input } from '@/components/ui/input';
import { createScript } from '@/lib/api/scripts'; // vanilla TS function
import { scriptSchema } from '@/lib/schemas/script'; // Zod schema

export const ScriptForm: React.FC = () => {
  const mutation = useMutation({
    mutationFn: createScript,
  });

  const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const result = scriptSchema.safeParse(Object.fromEntries(formData));

    if (!result.success) {
      // Handle validation errors
      return;
    }

    mutation.mutate(result.data);
  };

  return (
    <form onSubmit={handleSubmit} className="flex flex-col gap-4">
      <div className="flex flex-col gap-2">
        <Input
          name="title"
          placeholder="Script title…"
          autoComplete="off"
          required
        />
      </div>

      <div className="flex flex-col gap-2">
        <textarea
          name="content"
          placeholder="Write your script…"
          className="min-h-[200px] resize-y"
          required
        />
      </div>

      <Button type="submit" disabled={mutation.isPending}>
        {mutation.isPending ? 'Creating…' : 'Create Script'}
      </Button>
    </form>
  );
};
```

### Loading States

```tsx
// ❌ BAD - Separate skeleton component, conditional rendering causes layout shift
function FrameCardSkeleton() {
  return (
    <div className="p-4 border rounded">
      <div className="h-4 bg-gray-200 rounded w-3/4" />
      <div className="h-3 bg-gray-200 rounded w-1/2 mt-2" />
    </div>
  );
}

function FrameCard({ frame, showDetails }: Props) {
  if (!frame) return <FrameCardSkeleton />;

  return (
    <div className="p-4 border rounded">
      <h3>{frame.title}</h3>
      {showDetails && <DetailPanel frame={frame} />} {/* Layout shift */}
    </div>
  );
}

// ✅ GOOD - Inline skeleton, CSS display for visibility
import { Skeleton } from '@/components/ui/skeleton';

type FrameCardProps = {
  frame?: Frame;
  showDetails?: boolean;
};

export const FrameCard: React.FC<FrameCardProps> = ({
  frame,
  showDetails = false,
}) => {
  return (
    <div className="flex flex-col gap-3 p-4 border rounded">
      {frame ? (
        <h3 className="text-lg font-semibold">{frame.title}</h3>
      ) : (
        <Skeleton className="h-6 w-3/4" />
      )}

      {/* Pre-renders but hidden - no layout shift */}
      <div className={showDetails ? 'block' : 'hidden'}>
        {frame ? (
          <DetailPanel frame={frame} />
        ) : (
          <Skeleton className="h-20 w-full" />
        )}
      </div>
    </div>
  );
};
```

### File Organization

```tsx
// ❌ BAD
// File: UserProfile.tsx (PascalCase causes git issues)
export default function Component({ id }) {
  const user = useGlobalUser(); // global state
  // No URL state - can't deep link
}

// ✅ GOOD
// File: src/components/user-profile.tsx
import { formatUserName } from '@/lib/users/format-user-name'; // vanilla TS

type UserProfileProps = {
  userId: string; // from URL params
};

export const UserProfile: React.FC<UserProfileProps> = ({ userId }) => {
  const { data: user } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetchUser(userId),
  });

  return (
    <div className="flex flex-col gap-4">
      <h1>{user ? formatUserName(user) : <Skeleton className="h-8 w-48" />}</h1>
    </div>
  );
};

// File: src/routes/_protected/users/$userId.tsx (TanStack Router)
import { createFileRoute } from '@tanstack/react-router';
import { UserProfile } from '@/components/user-profile';

export const Route = createFileRoute('/_protected/users/$userId')({
  component: RouteComponent,
});

function RouteComponent() {
  const { userId } = Route.useParams();
  return <UserProfile userId={userId} />;
}
```

### Quick Reference

- **State**: TanStack Query for server data, reducers ONLY for complex UI state
- **Loading**: Suspense instead of isLoading checks, inline `<Skeleton />` fallbacks
- **Styling**: shadcn/ui base components + layout-only Tailwind (flex, gap, grid)
- **Layout**: Flexbox + gap (never margin on components), CSS display for show/hide
- **Files**: kebab-case.tsx, named exports, vanilla TS for logic
- **Forms**: TanStack Query mutations + Zod validation
- **Routing**: TanStack Router with `createFileRoute`, params via `Route.useParams()`
- **Imports**: Direct (useState not React.useState), @ alias, no default exports

---

## UI/UX Rules (MUST/SHOULD/NEVER)

**Keyboard & Focus:**

- MUST: Full keyboard support per [WAI-ARIA APG](https://www.w3.org/WAI/ARIA/apg/patterns/)
- MUST: Visible focus rings (`:focus-visible`), focus management (trap/return)

**Inputs & Forms:**

- MUST: Hit targets ≥24px (mobile ≥44px), font-size ≥16px to prevent zoom
- MUST: Hydration-safe inputs, allow paste, trim values
- MUST: Enter submits text input; Ctrl/Cmd+Enter submits textarea
- MUST: Errors inline next to fields; focus first error on submit
- MUST: `autocomplete` + meaningful `name`; correct `type` and `inputmode`
- SHOULD: Placeholders end with `…`, disable spellcheck for codes/emails
- NEVER: Block paste, disable browser zoom

**State & Navigation:**

- MUST: URL reflects state (filters/tabs/pagination) - use TanStack Router search params
- MUST: Back/Forward restores scroll
- MUST: Links use TanStack Router `<Link>` (supports Cmd/Ctrl/middle-click)

**Feedback:**

- SHOULD: Optimistic UI; reconcile on response; rollback or Undo on failure
- MUST: Confirm destructive actions or provide Undo window
- MUST: Use polite `aria-live` for toasts/inline validation
- SHOULD: Ellipsis (`…`) for loading states ("Loading…", "Saving…")

**Animation:**

- MUST: Honor `prefers-reduced-motion` (provide reduced variant)
- MUST: Animate compositor-friendly props (`transform`, `opacity`)
- MUST: Animations are interruptible and input-driven (avoid autoplay)
- SHOULD: Prefer CSS > Web Animations API > JS libraries

**Accessibility:**

- MUST: Redundant status cues (not color-only), icons have text labels
- MUST: `aria-label` for icon-only buttons
- MUST: Tabular numbers (`font-variant-numeric: tabular-nums`) for comparisons
- MUST: Non-breaking spaces: `10&nbsp;MB`, `Cmd&nbsp;+&nbsp;K`
- MUST: Prefer native semantics (`button`, `a`, `label`) before ARIA
- MUST: Skeletons mirror final content to avoid layout shift

**Performance:**

- MUST: Virtualize long lists (use `virtua`)
- MUST: Prevent CLS from images (explicit dimensions or reserved space)
- MUST: Track re-renders (React DevTools/React Scan)
- MUST: Mutations (`POST/PATCH/DELETE`) target <500ms
- SHOULD: Prefer uncontrolled inputs; make controlled loops cheap

**Layout:**

- MUST: Verify mobile, laptop, ultra-wide (simulate at 50% zoom)
- MUST: Respect safe areas (`env(safe-area-inset-*)`)
- MUST: Deliberate alignment to grid/baseline/edges - no accidental placement
- SHOULD: Optical alignment; adjust by ±1px when perception beats geometry

---

## Testing

**Framework:** `bun:test` (migrated from Vitest)

**Test location:**

- Server handlers: `__tests__/` directories alongside routes
- Services/utils: Same directory as module (`service.test.ts`)

**Focus:** Business logic (not React components)

**Bun mock pattern** (avoid shared state):

```typescript
const mockDb = mock(() => ({
  /* mock implementation */
}));

mock.module('@/lib/db/client', () => ({
  db: mockDb,
}));

beforeEach(async () => {
  mockDb.mockClear(); // Clear call history
  const module = await import('./module-under-test');
});
```

**Database testing:**

- Mock Drizzle/Turso clients (not real connections)
- Types auto-inferred from `src/lib/db/schema/`
- ULID primary keys (not UUID)

**Workflow testing:**

- Mock workflow context + AI service calls
- Test: step execution → state management → error handling
- Pass auth (userId/teamId) through context

**Commands:**

```bash
bun test                           # Run all tests
bun test --watch                   # Watch mode
bun test path/to/file.test.ts      # Single file
bun test --coverage                # With coverage
```

---

## Platform & Deployment

**Automatic platform detection:**

```typescript
// src/lib/utils/environment.ts
const platform = getDeploymentPlatform(); // cloudflare | vercel | railway | local
const appUrl = getAppUrl(); // Auto-resolves CF_PAGES_URL/VERCEL_URL/etc.
```

**Supported platforms:**

- **Cloudflare Pages** (recommended) - Edge runtime, R2 native, global CDN
- **Vercel** - Auto-scaling, edge functions
- **Railway** - Simple deploys, preview environments

**CI/CD** (`.github/workflows/`):

- Auto-deploys on push to main
- PR preview deployments
- Unique Turso database per PR

**Environment variables:**

- See `.env.example` for required vars
- Run `bun setup` for local dev defaults
- Production: Set via platform dashboard

---

## Global Rules

From `~/.claude/CLAUDE.md` (applies to all projects):

- Don't cast to `any`
- Don't create fallback code - display errors to user
- Look for repeated code or existing implementations
- Review code to make it concise and less repetitive
- Inline skeleton states (no separate skeleton components)
- Don't generate excessive tests - cover critical paths only
- **Database migrations**: Use Drizzle Kit (`bun db:generate`), never manually write SQL
- Use `type` instead of `interface`
- Throw errors instead of returning success boolean

<!-- intent-skills:start -->

# Skill mappings - when working in these areas, load the linked skill file into context.

skills:

- task: "setting up or modifying TanStack Start project structure"
  load: "node_modules/@tanstack/start-client-core/skills/start-core/SKILL.md"
- task: "deploying to Cloudflare Workers or other platforms"
  load: "node_modules/@tanstack/start-client-core/skills/start-core/deployment/SKILL.md"
- task: "working with server functions (createServerFn)"
  load: "node_modules/@tanstack/start-client-core/skills/start-core/server-functions/SKILL.md"
- task: "creating or modifying API server routes"
  load: "node_modules/@tanstack/start-client-core/skills/start-core/server-routes/SKILL.md"
- task: "implementing middleware for server functions"
  load: "node_modules/@tanstack/start-client-core/skills/start-core/middleware/SKILL.md"
- task: "understanding client/server execution boundaries"
  load: "node_modules/@tanstack/start-client-core/skills/start-core/execution-model/SKILL.md"
- task: "configuring TanStack Router (route trees, createRouter)"
  load: "node_modules/@tanstack/router-core/skills/router-core/SKILL.md"
- task: "working with route data loading and loaders"
  load: "node_modules/@tanstack/router-core/skills/router-core/data-loading/SKILL.md"
- task: "implementing navigation, links, and preloading"
  load: "node_modules/@tanstack/router-core/skills/router-core/navigation/SKILL.md"
- task: "working with search params and URL state"
  load: "node_modules/@tanstack/router-core/skills/router-core/search-params/SKILL.md"
- task: "route protection and auth guards"
  load: "node_modules/@tanstack/router-core/skills/router-core/auth-and-guards/SKILL.md"
- task: "code splitting routes"
  load: "node_modules/@tanstack/router-core/skills/router-core/code-splitting/SKILL.md"
- task: "handling not-found and error states in routes"
  load: "node_modules/@tanstack/router-core/skills/router-core/not-found-and-errors/SKILL.md"
- task: "working with path params ($paramName)"
  load: "node_modules/@tanstack/router-core/skills/router-core/path-params/SKILL.md"
- task: "type-safe routing patterns"
  load: "node_modules/@tanstack/router-core/skills/router-core/type-safety/SKILL.md"
- task: "SSR and streaming"
  load: "node_modules/@tanstack/router-core/skills/router-core/ssr/SKILL.md"
- task: "configuring the router Vite plugin"
  load: "node_modules/@tanstack/router-plugin/skills/router-plugin/SKILL.md"
- task: "setting up TanStack Devtools"
load: "node_modules/@tanstack/devtools/skills/devtools-app-setup/SKILL.md"
<!-- intent-skills:end -->

---
> Source: [openstory-so/openstory](https://github.com/openstory-so/openstory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
