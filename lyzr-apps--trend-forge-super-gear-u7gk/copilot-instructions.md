## trend-forge-super-gear-u7gk

> For FRESH BUILDS: Do NOT list directories or browse the file structure. Read ONLY `response_schemas/*.json` and `workflow.json` — then immediately start writing `app/page.tsx`. No exploring, no reading other files.

# Next.js React Frontend

## DO NOT EXPLORE — START BUILDING IMMEDIATELY

For FRESH BUILDS: Do NOT list directories or browse the file structure. Read ONLY `response_schemas/*.json` and `workflow.json` — then immediately start writing `app/page.tsx`. No exploring, no reading other files.

For ITERATIONS: Read `app/page.tsx` first to understand current state, then `response_schemas/*.json` for any updated response shapes.

---

## Import Rules

**Icons:** `lucide-react` ONLY (never react-icons)
```tsx
import { Loader2, Send, X } from 'lucide-react'
```

**Components:** `@/components/ui/*` (shadcn only)
```tsx
import { Button } from '@/components/ui/button'
import { Card, CardHeader, CardContent } from '@/components/ui/card'
import { Input } from '@/components/ui/input'
```

**Agent Calls:** `@/lib/aiAgent` (client-side, calls pre-built `/api/agent` route — NEVER create new routes)
```tsx
import { callAIAgent } from '@/lib/aiAgent'
// callAIAgent(message, agent_id) — ONLY way to call agents. NEVER custom fetch.
```

---

## COMPONENT WHITELIST (CLOSED SET — nothing else exists)

**ONLY these components may be used in `app/page.tsx`.** If a name is not listed here, it does NOT exist. Define custom components inline as functions in page.tsx.

### shadcn/ui Components (import from `@/components/ui/<file>`)

| File | Exports |
|------|---------|
| accordion | `Accordion`, `AccordionItem`, `AccordionTrigger`, `AccordionContent` |
| alert | `Alert`, `AlertTitle`, `AlertDescription` |
| alert-dialog | `AlertDialog`, `AlertDialogTrigger`, `AlertDialogContent`, `AlertDialogHeader`, `AlertDialogFooter`, `AlertDialogTitle`, `AlertDialogDescription`, `AlertDialogAction`, `AlertDialogCancel` |
| aspect-ratio | `AspectRatio` |
| avatar | `Avatar`, `AvatarImage`, `AvatarFallback` |
| badge | `Badge` |
| breadcrumb | `Breadcrumb`, `BreadcrumbList`, `BreadcrumbItem`, `BreadcrumbLink`, `BreadcrumbPage`, `BreadcrumbSeparator` |
| button | `Button` |
| calendar | `Calendar` |
| card | `Card`, `CardHeader`, `CardFooter`, `CardTitle`, `CardDescription`, `CardContent` |
| carousel | `Carousel`, `CarouselContent`, `CarouselItem`, `CarouselPrevious`, `CarouselNext` |
| chart | `ChartContainer`, `ChartTooltip`, `ChartTooltipContent`, `ChartLegend`, `ChartLegendContent` |
| checkbox | `Checkbox` |
| collapsible | `Collapsible`, `CollapsibleTrigger`, `CollapsibleContent` |
| command | `Command`, `CommandDialog`, `CommandInput`, `CommandList`, `CommandEmpty`, `CommandGroup`, `CommandItem`, `CommandShortcut`, `CommandSeparator` |
| context-menu | `ContextMenu`, `ContextMenuTrigger`, `ContextMenuContent`, `ContextMenuItem`, `ContextMenuLabel`, `ContextMenuSeparator` |
| dialog | `Dialog`, `DialogTrigger`, `DialogContent`, `DialogHeader`, `DialogFooter`, `DialogTitle`, `DialogDescription`, `DialogClose` |
| drawer | `Drawer`, `DrawerTrigger`, `DrawerContent`, `DrawerHeader`, `DrawerFooter`, `DrawerTitle`, `DrawerDescription`, `DrawerClose` |
| dropdown-menu | `DropdownMenu`, `DropdownMenuTrigger`, `DropdownMenuContent`, `DropdownMenuItem`, `DropdownMenuLabel`, `DropdownMenuSeparator`, `DropdownMenuGroup` |
| empty | `Empty`, `EmptyHeader`, `EmptyTitle`, `EmptyDescription`, `EmptyContent`, `EmptyMedia` |
| form | `Form`, `FormItem`, `FormLabel`, `FormControl`, `FormDescription`, `FormMessage`, `FormField` |
| hover-card | `HoverCard`, `HoverCardTrigger`, `HoverCardContent` |
| input | `Input` |
| input-group | `InputGroup`, `InputGroupAddon`, `InputGroupButton`, `InputGroupText` |
| input-otp | `InputOTP`, `InputOTPGroup`, `InputOTPSlot`, `InputOTPSeparator` |
| label | `Label` |
| menubar | `Menubar`, `MenubarMenu`, `MenubarTrigger`, `MenubarContent`, `MenubarItem`, `MenubarSeparator` |
| navigation-menu | `NavigationMenu`, `NavigationMenuList`, `NavigationMenuItem`, `NavigationMenuContent`, `NavigationMenuTrigger`, `NavigationMenuLink` |
| pagination | `Pagination`, `PaginationContent`, `PaginationItem`, `PaginationLink`, `PaginationNext`, `PaginationPrevious` |
| popover | `Popover`, `PopoverTrigger`, `PopoverContent` |
| progress | `Progress` |
| radio-group | `RadioGroup`, `RadioGroupItem` |
| resizable | `ResizablePanelGroup`, `ResizablePanel`, `ResizableHandle` |
| scroll-area | `ScrollArea`, `ScrollBar` |
| select | `Select`, `SelectTrigger`, `SelectContent`, `SelectItem`, `SelectValue`, `SelectGroup`, `SelectLabel` |
| separator | `Separator` |
| sheet | `Sheet`, `SheetTrigger`, `SheetContent`, `SheetHeader`, `SheetFooter`, `SheetTitle`, `SheetDescription`, `SheetClose` |
| sidebar | `Sidebar`, `SidebarContent`, `SidebarFooter`, `SidebarGroup`, `SidebarGroupContent`, `SidebarGroupLabel`, `SidebarHeader`, `SidebarMenu`, `SidebarMenuButton`, `SidebarMenuItem`, `SidebarProvider`, `SidebarTrigger` |
| skeleton | `Skeleton` |
| slider | `Slider` |
| sonner | `Toaster` |
| spinner | `Spinner` |
| switch | `Switch` |
| table | `Table`, `TableHeader`, `TableBody`, `TableFooter`, `TableHead`, `TableRow`, `TableCell`, `TableCaption` |
| tabs | `Tabs`, `TabsList`, `TabsTrigger`, `TabsContent` |
| textarea | `Textarea` |
| toggle | `Toggle` |
| toggle-group | `ToggleGroup`, `ToggleGroupItem` |
| tooltip | `Tooltip`, `TooltipTrigger`, `TooltipContent`, `TooltipProvider` |

### lucide-react Icons (commonly used)

```
Search, Send, X, Plus, Minus, Check, ChevronDown, ChevronUp, ChevronLeft, ChevronRight,
ArrowLeft, ArrowRight, ArrowUp, ArrowDown, Loader2, Menu, Settings, User, Users,
Home, Mail, Phone, Calendar, Clock, Star, Heart, Trash2, Edit, Copy, Download,
Upload, File, FileText, Image, Camera, Mic, MicOff, Volume2, VolumeX, Play, Pause,
Square, Circle, AlertCircle, AlertTriangle, Info, HelpCircle, ExternalLink, Link,
Eye, EyeOff, Lock, Unlock, Shield, Zap, RefreshCw, RotateCcw, Filter, SortAsc,
MapPin, Globe, Bookmark, Tag, Hash, AtSign, MessageSquare, MessageCircle, Bot,
Sparkles, Wand2, Palette, Layout, Grid, List, BarChart3, PieChart, TrendingUp
```

Any lucide-react icon name is valid — check https://lucide.dev/icons for the full set. The list above is for convenience.

### HALLUCINATION BLACKLIST (these DO NOT exist — never use them)

| Hallucinated Name | Use Instead |
|---|---|
| `SectionCard` | `Card` + `CardHeader` + `CardContent` |
| `DiscoverScreen` | Plain `<div>` or `Card` |
| `FeatureCard` | `Card` + `CardContent` — or define inline: `function FeatureCard(...)` |
| `ActionButton` | `Button` |
| `IconButton` | `Button` with `variant="ghost" size="icon"` |
| `TextInput` | `Input` |
| `SearchInput` | `Input` with a Search icon |
| `NavBar` / `Navbar` | `<nav>` with Tailwind — or define inline |
| `Container` | `<div className="max-w-7xl mx-auto px-4">` |
| `Hero` / `HeroSection` | Plain `<section>` with Tailwind |
| `Footer` | `<footer>` with Tailwind |
| `Modal` | `Dialog` |
| `Dropdown` | `DropdownMenu` or `Select` |
| `Toast` | `Toaster` from sonner |
| `Chip` | `Badge` |
| `Tag` | `Badge` |
| `Spinner` (from wrong path) | `Spinner` from `@/components/ui/spinner` or `Loader2` with `animate-spin` |
| `Box` / `Stack` / `Flex` | Plain `<div>` with Tailwind flex/grid classes |

**Rule:** If you need a component that isn't in the whitelist above, **define it as a function** in `page.tsx`:

```tsx
// CORRECT — define custom components inline
function FeatureCard({ title, desc }: { title: string; desc: string }) {
  return (
    <Card>
      <CardHeader><CardTitle>{title}</CardTitle></CardHeader>
      <CardContent><p>{desc}</p></CardContent>
    </Card>
  )
}
```

---

## callAIAgent Response (GUARANTEED)

```tsx
const result = await callAIAgent(message, AGENT_ID)

// Structure is ALWAYS:
result.success          // boolean - API call succeeded?
result.response.status  // "success" | "error" - agent status
result.response.result  // { ...agent data } - YOUR FIELDS HERE (preserves agent schema exactly)
result.response.message // string | undefined - display text extracted from result
result.module_outputs   // { artifact_files?: [{ file_url, name, format_type }] } - images/files from agent
```

### Response Field Preservation

`result` preserves the agent's schema fields exactly as returned. If the agent's `response_format` schema defines `{"result": {"response": "text"}}`, then `result.response.result.response` will contain the text. Common field names inside `result`: `text`, `response`, `message`, `answer`, `summary`, `content`.

The `message` field is auto-extracted from the first matching text field inside `result` for display convenience (via `extractText()`).

### Agent Response Parsing (CRITICAL)

1. READ `response_schemas/<agent_name>.json` to find EXACT field names
2. The schema defines `result: { field1, field2, ... }` — use THOSE fields
3. Access: `result.response.result.<schema_field>`
4. Multi-path fallback for robustness:

```tsx
const parsed = result?.response?.result ?? result?.response ?? null;
if (typeof parsed === 'string') {
  setMessage(parsed);
} else if (parsed && typeof parsed === 'object') {
  // Use fields from response_schema — NOT hardcoded guesses
  const text = parsed.summary ?? parsed.message ?? parsed.response
    ?? parsed.text ?? parsed.content ?? JSON.stringify(parsed, null, 2);
  setMessage(text);
}
```

**NEVER use static fallback strings like `'Analysis complete.'`** — if response is empty, show an error state instead.

### Complete Usage Example
```tsx
'use client'
import { useState } from 'react'
import { callAIAgent } from '@/lib/aiAgent'

export default function MyPage() {
  const [loading, setLoading] = useState(false)
  const [data, setData] = useState<any>(null)
  const [error, setError] = useState<string | null>(null)

  const handleSubmit = async () => {
    setLoading(true)
    setError(null)

    const result = await callAIAgent(userMessage, AGENT_ID)

    if (result.success && result.response.status === 'success') {
      setData(result.response.result)
    } else {
      setError(result.response.message || 'Request failed')
    }

    setLoading(false)
  }

  // ... rest of component
}
```

---

## CRITICAL: EVERYTHING IS PRE-BUILT — NEVER RECREATE

This template is a complete, wired-up app. NEVER recreate, rewrite, or duplicate any existing file.

### API Routes (NEVER create new `route.ts` files — exception: Database apps create auth + data routes, see Database & Auth section)

| Route | Method | Purpose |
|-------|--------|---------|
| `/api/agent` | POST | Call AI agent |
| `/api/upload` | POST | Upload files for AI analysis |
| `/api/rag` | POST/PATCH/DELETE | RAG knowledge base operations |
| `/api/scheduler` | GET/POST/DELETE | Schedule management operations |
| `/api/lyzr-config` | GET | Get API key (utility) |

### Client Libraries (NEVER write custom `fetch()` calls)

| Library | Import | Use for |
|---------|--------|---------|
| `@/lib/aiAgent` | `callAIAgent`, `uploadFiles`, `useAIAgent`, `useFileUpload`, `extractText` | Agent calls, file uploads |
| `@/lib/ragKnowledgeBase` | `getDocuments`, `uploadAndTrainDocument`, `deleteDocuments`, `crawlWebsite`, `validateFile`, `isFileTypeSupported`, `useRAGKnowledgeBase` | RAG/knowledge base operations |
| `@/lib/scheduler` | `listSchedules`, `getSchedule`, `getSchedulesForAgent`, `getScheduleLogs`, `getRecentExecutions`, `createSchedule`, `pauseSchedule`, `resumeSchedule`, `triggerScheduleNow`, `deleteSchedule`, `cronToHuman`, `useScheduler` | Schedule management |
| `@/lib/clipboard` | `copyToClipboard`, `useCopyToClipboard` | Clipboard operations |
| `@/lib/jsonParser` | `parseLLMJson` | JSON parsing |
| `@/lib/utils` | `cn` | Class name merging |

### Tool Parameter Forms (for agents with tools)

**If your agent uses external tools (Gmail, Slack, Calendar, etc.), create input forms to collect required parameters BEFORE calling the agent.**

Example for Gmail send email:
```tsx
const [emailForm, setEmailForm] = useState({ recipient: '', subject: '', body: '' })

<Input 
  placeholder="Recipient" 
  value={emailForm.recipient}
  onChange={(e) => setEmailForm(prev => ({ ...prev, recipient: e.target.value }))}
/>
<Button 
  onClick={() => callAIAgent(`Send email to ${emailForm.recipient}...`, AGENT_ID)}
  disabled={!emailForm.recipient || !emailForm.subject}
>
  Send Email
</Button>
```

See `TOOL_PARAMETER_FORMS_GUIDE.md` for complete patterns and examples.

### Components (NEVER create files in `components/`)

All shadcn/ui components are installed at `@/components/ui/*`. See COMPONENT WHITELIST above. Define custom components inline in `page.tsx`.

### Providers (already in `layout.tsx` — NEVER recreate)

`ErrorBoundary` · `AgentInterceptorProvider` · `IframeLoggerInit` · `KnowledgeBaseUpload`

**If it exists in the template, import it. Do not rebuild it.**

---

## UI Code Location

**CRITICAL:**
- ALL UI code goes in `app/page.tsx`
- Define components inline or import from `@/components/ui/`
- NEVER create files in `components/` (reserved for shadcn/ui)

```tsx
// app/page.tsx

// Define inline components
const ChatMessage = ({ message }: { message: string }) => (
  <div className="p-4 bg-muted rounded-lg">{message}</div>
)

// Main page component
export default function Page() {
  return (
    <div>
      <ChatMessage message="Hello" />
    </div>
  )
}
```

---

## Sections (for parallel builds)

- `app/sections/*.tsx` — max 4 files, page-level sections only
- Each: `'use client'`, one `export default function`, typed props
- Same import rules as page.tsx (shadcn/ui, lucide-react, @/lib/*)
- `components/` still reserved for shadcn/ui — NEVER create files there
- Simple apps (1 agent, no special features): skip sections, use monolithic page.tsx

## Write Tool Fallback

If the Write tool fails for any file (`app/page.tsx` or `app/sections/*.tsx`):
1. Write a minimal version first (imports + one component)
2. Use Edit to append remaining components one at a time
3. Never retry the same failed Write — always switch to incremental Edit

---

## File Upload with AI Analysis

```tsx
'use client'
import { uploadFiles, callAIAgent } from '@/lib/aiAgent'

const handleFileUpload = async (file: File) => {
  // 1. Upload file
  const uploadResult = await uploadFiles(file)

  if (uploadResult.success) {
    // 2. Call agent with asset IDs
    const result = await callAIAgent('Analyze this document', AGENT_ID, {
      assets: uploadResult.asset_ids
    })
  }
}
```

---

## RAG Knowledge Base

```tsx
'use client'
import {
  getDocuments,
  uploadAndTrainDocument,
  deleteDocuments,
  useRAGKnowledgeBase
} from '@/lib/ragKnowledgeBase'

// Using hook
const { documents, loading, fetchDocuments, uploadDocument, removeDocuments } = useRAGKnowledgeBase()

// Or direct functions
const docs = await getDocuments('rag-id')
await uploadAndTrainDocument('rag-id', file)
await deleteDocuments('rag-id', ['doc.pdf'])
```

---

## Environment Variables

**Server-side (in `.env.local`):**
```
LYZR_API_KEY=your-api-key
```

**Client-side access (if needed):**
```tsx
// Only NEXT_PUBLIC_ prefixed vars are exposed to client
const publicVar = process.env.NEXT_PUBLIC_SOME_VAR
```

---

## Fast Development

The project is configured for fast dev with Turbopack:
```bash
npm run dev  # Uses --turbo flag
```

---

## Available shadcn/ui Components (All Prebuilt)

All these components are prebuilt in `@/components/ui/` — import directly, no installation needed:

```
accordion, alert, alert-dialog, aspect-ratio, avatar, badge, breadcrumb,
button, button-group, calendar, card, carousel, chart, checkbox, collapsible,
command, context-menu, dialog, drawer, dropdown-menu, empty, field, form,
hover-card, input, input-group, input-otp, item, kbd, label, menubar,
navigation-menu, pagination, popover, progress, radio-group, resizable,
scroll-area, select, separator, sheet, sidebar, skeleton, slider, sonner,
spinner, switch, table, tabs, textarea, toggle, toggle-group, tooltip
```

---

## IFRAME-BLOCKED APIs (CRITICAL!)

This app runs in an iframe. These browser APIs are BLOCKED and will throw errors:

### Clipboard - USE UTILITY!
```tsx
// BANNED - Will throw NotAllowedError:
navigator.clipboard.writeText(text)  // BLOCKED!

// CORRECT - Use safe utility:
import { copyToClipboard } from '@/lib/clipboard'

const handleCopy = async () => {
  const success = await copyToClipboard(text)
  if (success) setCopied(true)
}
```

### Other Blocked APIs
- `navigator.geolocation` - blocked
- `navigator.share()` - blocked
- `window.open()` - may be blocked

---

## Database & Auth — `lyzr-architect` Package

**See the database skill for full provisioning flow, model templates, auth routes, RLS rules, and anti-localStorage rules.**

When the app has a database (`DATABASE_URL` env var is set), use the **`lyzr-architect`** package for ALL database and auth operations. It is pre-installed — do NOT install it manually.

**CRITICAL RULES:**
- **NEVER use `localStorage` or `sessionStorage` for application data** (tasks, notes, records, user data). localStorage is ONLY for UI preferences (theme, sidebar). All persistent data MUST go through API routes backed by MongoDB.
- NEVER write custom JWT, bcrypt, session, or cookie auth code
- NEVER manually filter queries by user ID — the RLS plugin does it automatically
- NEVER access the `_users` collection directly — use the auth API
- NEVER use raw `mongodb` driver or `mongoose` directly — use `lyzr-architect` wrappers

### 1. Database Connection

Call `initDB()` once. It connects to MongoDB via `DATABASE_URL` env var and registers the RLS plugin.

```ts
// In any API route or layout:
import { initDB } from 'lyzr-architect'

await initDB()  // Safe to call multiple times (idempotent)
```

### 2. Data Models — Lazy Getter Pattern (CRITICAL)

**NEVER call `createModel()` at module top level** — it runs before `initDB()` registers RLS, breaking security. Always use the lazy getter pattern:

```ts
// models/task.ts
import { initDB, createModel } from 'lyzr-architect'

// createModel auto-adds: owner_user_id (for RLS), created_at, updated_at
let _model: any = null
export default async function getTaskModel() {
  if (!_model) {
    await initDB()
    _model = createModel('Task', {
      title: { type: String, required: true },
      description: { type: String, default: '' },
      status: { type: String, enum: ['pending', 'done'], default: 'pending' },
      priority: { type: Number, default: 0 },
    })
  }
  return _model
}
```

Usage in API routes:
```ts
const Task = await getTaskModel()
const tasks = await Task.find({})                    // Only returns current user's tasks
await Task.findByIdAndUpdate(id, { status: 'done' })  // Only updates if owned by user
await Task.findByIdAndDelete(id)                       // Only deletes if owned by user
```

### 3. Auth Routes — Pre-built Handlers (4 individual files)

```ts
// app/api/auth/register/route.ts
import { handleRegister } from 'lyzr-architect'
export const POST = handleRegister  // Body: { email, password, name? }

// app/api/auth/login/route.ts
import { handleLogin } from 'lyzr-architect'
export const POST = handleLogin  // Body: { email, password }

// app/api/auth/logout/route.ts
import { handleLogout } from 'lyzr-architect'
export const POST = handleLogout

// app/api/auth/me/route.ts
import { handleMe } from 'lyzr-architect'
export const dynamic = 'force-dynamic'
export const GET = handleMe  // Returns { user } or { user: null }
```

### 4. Protected API Routes — `authMiddleware()`

All API routes MUST use `try/catch` and return `{ success, data/error }` format. Use the lazy model getter — never import a top-level model.

```ts
import { authMiddleware } from 'lyzr-architect'
import getTaskModel from '@/models/task'
import { NextRequest, NextResponse } from 'next/server'

// authMiddleware: extracts JWT, sets RLS context, returns 401 if not logged in
// All queries inside are auto-scoped to the logged-in user
export const GET = authMiddleware(async (req: NextRequest) => {
  try {
    const Task = await getTaskModel()
    const tasks = await Task.find({}).sort({ created_at: -1 })
    return NextResponse.json({ success: true, data: tasks })
  } catch (err: any) {
    console.error('[API] GET /api/tasks error:', err)
    return NextResponse.json({ success: false, error: err.message }, { status: 400 })
  }
})

export const POST = authMiddleware(async (req: NextRequest) => {
  try {
    const Task = await getTaskModel()
    const body = await req.json()
    const { getCurrentUserId } = await import('lyzr-architect')
    const task = await Task.create({ ...body, owner_user_id: getCurrentUserId() })
    return NextResponse.json({ success: true, data: task }, { status: 201 })
  } catch (err: any) {
    console.error('[API] POST /api/tasks error:', err)
    return NextResponse.json({ success: false, error: err.message }, { status: 400 })
  }
})
```

### 5. Client Components — Auth UI

```tsx
'use client'
import { AuthProvider, LoginForm, RegisterForm, UserMenu, ProtectedRoute } from 'lyzr-architect/client'

// Auth screen with login/register toggle
function AuthScreen() {
  const [isLogin, setIsLogin] = useState(true)
  return isLogin
    ? <LoginForm onSwitchToRegister={() => setIsLogin(false)} />
    : <RegisterForm onSwitchToLogin={() => setIsLogin(true)} />
}

// Wrap app in AuthProvider, use ProtectedRoute with unauthenticatedFallback
<AuthProvider>
  <ProtectedRoute unauthenticatedFallback={<AuthScreen />}>
    <header><UserMenu /></header>
    <Dashboard />
  </ProtectedRoute>
</AuthProvider>

// Do NOT manually check `if (!user)` — ProtectedRoute handles it
// Do NOT use window.location.reload() — AuthProvider auto-updates state
```

### 6. How RLS Works

- `authMiddleware(handler)` extracts JWT → sets RLS context with `userId`
- The global RLS plugin auto-adds `{ owner_user_id: userId }` to ALL queries
- On `create`/`save`, `owner_user_id` is force-set to the logged-in user (prevents spoofing)
- No context = query blocked (deny-by-default)
- System collections (`_users`, `_sessions`) are blocked from direct access
- **For queries (find, update, delete), RLS auto-scopes — no code needed.**
- **For `.create()` calls, pass `owner_user_id: getCurrentUserId()` as a safety fallback** (belt-and-suspenders with the RLS plugin).

### 7. Env Vars (Auto-set, DO NOT hardcode)

```
DATABASE_URL=mongodb://...        # Connection string (set by platform)
APP_JWT_SECRET=...                # JWT signing secret (set by platform)
```

---

## Anti-Hallucination Checklist

Before writing UI code:
- [ ] Read workflow.json for agent_ids?
- [ ] Read response_schemas/*.json for field names?
- [ ] Interfaces match schema exactly?
- [ ] Using optional chaining (?.)?
- [ ] Loading/error states handled?
- [ ] Only lucide-react icons?
- [ ] Only shadcn/ui components?
- [ ] 'use client' directive for client components?
- [ ] No navigator.clipboard? (use @/lib/clipboard)
- [ ] Database code uses `lyzr-architect` (not raw mongodb/mongoose)?
- [ ] Auth uses pre-built handlers (not custom JWT/bcrypt)?
- [ ] Models use lazy getter pattern (`async function getTaskModel()`) — NOT top-level `createModel()`?
- [ ] API routes wrapped with `authMiddleware()`?
- [ ] All API routes wrapped in `try/catch`?
- [ ] Error responses use `{ success: false, error: message }` format?
- [ ] UI shows API errors (not silent failures)?
- [ ] POST routes include `owner_user_id: getCurrentUserId()` safety fallback?
- [ ] ZERO localStorage/sessionStorage for application data? (only UI preferences like theme)
- [ ] Data fetched from API routes (not from localStorage)?

---
> Source: [Lyzr-Apps/trend-forge-super-gear-u7gk](https://github.com/Lyzr-Apps/trend-forge-super-gear-u7gk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
