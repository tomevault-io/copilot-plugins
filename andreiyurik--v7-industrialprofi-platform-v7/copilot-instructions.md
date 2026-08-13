## v7-industrialprofi-platform-v7

> - **Backend:** Ruby on Rails 8.0 (Modern Monolith, Propshaft)

# Project Guide: IndustrialProfi — B2B SaaS Platform

## Tech Stack

### Active
- **Backend:** Ruby on Rails 8.0 (Modern Monolith, Propshaft)
- **Frontend:** React 18 + TypeScript 5.7 (strict) via Inertia.js 2.0
- **Build Tool:** Vite 5 + vite-plugin-ruby
- **Database:** PostgreSQL 16
- **Styling:** Tailwind CSS 3.4 + shadcn/ui (Radix primitives) + Headless UI
- **Graphs:** @xyflow/react 12.3 (React Flow)
- **Icons:** Lucide React
- **i18n:** react-i18next (ru + en)
- **Dates:** date-fns 4

### Planned
- **Components:** Magic UI (Marketing), Aceternity UI (Hero effects)
- **Typography:** Geist Sans (Vercel)

### Backend Stack
- **Auth:** Custom session-based (bcrypt) + Pundit (authorization policies)
- **Multi-tenancy:** ActsAsTenant (organization isolation)
- **Audit:** PaperTrail (compliance versioning)
- **Jobs:** Solid Queue (no Redis)
- **Cache:** Solid Cache
- **WebSockets:** Solid Cable
- **Email:** ActionMailer + letter_opener (dev)
- **Pagination:** Pagy
- **Deployment:** Kamal 2 + Docker + Thruster

### Runtime Versions
- **Ruby:** 3.4.1
- **Node:** not pinned (Vite 5, works with Node 18+)

## Domain Model

```
Organization (tenant root)
├── has_many Users (employee | manager | owner)
├── has_many JobTitles
├── has_many RoadmapAssignments
│
User
├── belongs_to Organization (acts_as_tenant)
├── has_many RoadmapAssignments → assigned_roadmaps (through)
├── has_many UserProgresses (skill completion tracking)
├── has_many RoadmapFavorites → favorite_roadmaps (through)
├── has_many authored_roadmaps (Roadmap.author_id)
├── has_many verified_progresses (UserProgress.verified_by_id)
│
Roadmap (organization_id nullable — nil = global public template)
├── belongs_to Organization (optional, acts_as_tenant has_global_records: true)
├── belongs_to author: User (optional)
├── belongs_to forked_from: Roadmap (self-referential fork chain)
├── has_many Skills (counter_cache: skills_count)
├── has_many RoadmapAssignments → assigned_users (through)
├── has_many RoadmapFavorites (counter_cache: favorites_count)
│
Skill
├── belongs_to Roadmap
├── belongs_to PermitTemplate (optional — auto-sets skill_type to "permit")
├── has_many SkillDependencies (from/to — directed graph edges)
├── types: skill | permit | milestone
├── stage (integer) — visual grouping for React Flow layout
│
UserProgress (core training record: user × skill)
├── belongs_to User
├── belongs_to Skill
├── belongs_to verified_by: User (optional — manager who approved)
├── status flow: todo → in_progress → pending_review → completed
├── permit fields: certificate_number, issued_at, expires_at, issuing_authority
│
PermitTemplate (global, no org — shared permit definitions)
├── code (unique, e.g. "ELECTRICAL_1"), expiration_months, country_code
```

Key relationships: A manager assigns a User to a Roadmap (via RoadmapAssignment). The user progresses through each Skill in that Roadmap (via UserProgress). For permit-type skills, a manager verifies completion and the system tracks certificate expiry.

## Directory Structure
- `app/frontend/pages/` — Inertia page components, grouped by domain (Auth, Dashboard, Landing, Roadmaps, Org)
- `app/frontend/components/ui/` — shadcn/Radix base components
- `app/frontend/components/roadmap/` — XyFlow custom nodes and graph visualization
- `app/frontend/components/dashboard/` — Dashboard-specific widgets
- `app/frontend/components/landing/` — Landing page marketing sections
- `app/frontend/components/layout/` — Layout primitives (Sidebar)
- `app/frontend/components/shared/` — Reusable components (LanguageSwitcher, FlashMessages, TagsInput)
- `app/frontend/layouts/` — MainLayout (nav + sidebar), AuthLayout (centered card)
- `app/frontend/lib/` — Utilities: route helpers, i18n config, graph layout, cn()
- `app/frontend/types/` — TypeScript interfaces mirroring Rails serializer output
- `app/frontend/locales/` — Translation JSON files (en.json, ru.json)
- `app/frontend/entrypoints/` — Vite entry points (inertia.tsx, application.css)
- `app/controllers/` — Rails controllers with Inertia renders
- `app/policies/` — Pundit authorization policies
- `app/mailers/` — UserMailer, ManagerMailer
- `app/jobs/` — ActiveJob tasks (Solid Queue)

## Coding Standards & Rules
- **TypeScript Strict:** All frontend code in TypeScript strict mode. Define interfaces in `types/`.
- **i18n First:** All user-facing strings via `t()` from react-i18next. No hardcoded text.
- **Centralized Routes:** All URLs via `routes.*` helpers from `lib/routes.ts`. No hardcoded paths.
- **Performance First:**
  - Prioritize CSS animations (`tailwindcss-animate`) over JS-based ones.
  - Use `React.lazy` for components below the fold.
- **Inertia.js:** Always prefer SSR-friendly patterns. Use `router.*` and `<Link>` for navigation.
- **Styling:** Tailwind classes only. Global variables in `application.css`. Dark mode first aesthetic.
- **Multi-tenancy:** Always scope queries through `current_organization`. Never bypass tenant isolation.
- **Authorization:** Every controller action must be authorized via Pundit (`verify_authorized`).
- **React Flow:** Use XyFlow v12+ patterns. `nodeTypes` declared outside components. Custom nodes wrapped in `memo()`.
- **No Hardcoded Secrets:** Use Rails credentials or environment variables.
- **shadcn/ui via MCP:** Always use the shadcn MCP server tools when working with shadcn/ui components. See [shadcn/ui Workflow](#shadcnui-workflow) section below.

## i18n Key Naming Convention

### Frontend (`locales/en.json`, `locales/ru.json`)

Flat-ish structure: `{page_or_domain}.{descriptive_key}`, all `snake_case`, max 2 levels deep.

Top-level namespaces match pages/features:
- `common.*` — shared buttons/labels (`common.save`, `common.cancel`, `common.delete`)
- `auth.*` — login/register page (`auth.login_title`, `auth.submit_login`)
- `nav.*` — sidebar/header navigation (`nav.catalog`, `nav.create_roadmap`)
- `landing.*` — public landing page
- `dashboard.*` — dashboard page (`dashboard.title`, `dashboard.my_roadmaps`)
- `roadmaps.*` — catalog and roadmap pages (`roadmaps.catalog_title`, `roadmaps.fork`)
- `sidebar.*` — skill detail sidebar (`sidebar.status_todo`, `sidebar.type_permit`)
- `org.*` — organization management (`org.employees_title`, `org.invite_employee`)
- `skills_matrix.*` — matrix page
- `job_titles.*` — job titles CRUD
- `profile.*` — user profile
- `quick_create.*` — modal dialog

Sub-forms nest one level deeper: `roadmaps.form.title`, `sidebar.permit_form.certificate_number`.

Key naming patterns:
```
{domain}.title                    — page heading (dashboard.title)
{domain}.{entity}_{action}        — buttons (org.invite_employee, org.assign_roadmap)
{domain}.no_{entity}              — empty states (dashboard.no_assigned_roadmaps, org.no_employees)
{domain}.col_{field}              — table column headers (dashboard.permit_col_employee)
{domain}.filter_{field}           — filter labels (roadmaps.filter_locale)
{domain}.sort_{type}              — sort options (roadmaps.sort_popular)
{domain}.status_{value}           — status labels (sidebar.status_in_progress)
{domain}.{action}ing              — loading states (roadmaps.form.creating, auth.logging_in)
{domain}.submit_{action}          — submit buttons (auth.submit_login, roadmaps.form.submit_create)
```

Interpolation uses `{{variable}}`: `t('dashboard.skills_short', { completed: 3, total: 10 })` → "3/10 skills".

### Backend (Rails `I18n.t`)

Flash messages use a flat `flash.*` namespace with `{entity}_{past_tense}` pattern:
```
flash.roadmap_created             — success after create
flash.roadmap_updated             — success after update
flash.roadmap_deleted             — success after destroy
flash.employee_invited            — success after invite
flash.profile_updated             — success after profile save
flash.permit_submitted            — permit sent for review
flash.permit_confirmed            — manager approved permit
flash.permit_rejected             — manager rejected permit
flash.login_success / flash.logout_success / flash.register_success
flash.not_authorized              — Pundit denial
flash.invalid_credentials         — login failure
flash.employee_limit_reached      — plan limit hit
```

## Anti-patterns — Do NOT

### Frontend
- **No `useEffect` for navigation.** Use `router.get()` / `router.post()` from `@inertiajs/react`. Never `window.location.href = ...` for internal links.
- **No `<a href>` for internal links.** Always `<Link href={routes.path()}>` — Inertia manages SPA navigation and preserves state.
- **No `fetch()` for mutations that change pages.** Use `router.post` / `router.put` / `router.delete`. Reserve `fetch()` only for background JSON calls (favorites toggle, autocomplete) where you handle optimistic UI yourself.
- **No hardcoded URL strings.** Always `routes.roadmap(slug)`, never `` `/roadmaps/${slug}` ``.
- **No hardcoded user-facing text.** Always `t('key')`, even for button labels and placeholders.
- **No `nodeTypes` inside render.** Declare XyFlow `nodeTypes` / `edgeTypes` as module-level constants — otherwise React Flow re-registers them on every render.
- **No `.as_json` / `.to_json` on models.** Serialize explicitly via private `*_json` helper methods in controllers.

### Backend
- **No raw SQL that bypasses tenant scope.** `ActsAsTenant` scopes queries automatically — never use `unscoped`, `Model.where(organization_id: ...)` manually, or raw SQL that skips it.
- **No controller action without `authorize`.** `after_action :verify_authorized` will raise if you forget. For public endpoints, explicitly `skip_after_action :verify_authorized`.
- **No `redirect_to` with hardcoded strings for flash.** Always `I18n.t("flash.key")`.
- **No N+1 queries.** Bulk-load with `.includes()` or pre-fetch into a hash (see Skills Matrix pattern below).

## Code Patterns

### Inertia Controller (typical auth'd action)

```ruby
class DashboardController < ApplicationController
  def index
    authorize :dashboard, :index?                    # 1. Authorize first

    props = {
      stats:            build_stats,                 # 2. Build props via private methods
      assignedRoadmaps: load_assigned_roadmaps,
      isManager:        current_user.manager? || current_user.owner?
    }

    if current_user.manager? || current_user.owner?  # 3. Conditional props by role
      props[:permitAlerts] = load_permit_alerts
    end

    render inertia: "Dashboard/Index", props: props  # 4. render inertia: "Dir/Component"
  end

  private

  def build_stats                                    # 5. Private builder methods
    { roadmapsCount: assignments.count, ... }
  end
end
```

Key: `authorize` → build props → `render inertia:`. On validation failure: re-render same page with `errors: @record.errors.messages`. On success: `redirect_to path, notice: I18n.t("flash.key")`.

### Inertia Shared Props (available in every page)

Defined once in `ApplicationController`:

```ruby
inertia_share do
  {
    auth: {
      user:         current_user ? auth_user_json(current_user) : nil,
      organization: current_user ? auth_org_json(current_user.organization) : nil
    },
    flash:    { notice: flash.notice, alert: flash.alert },
    features: { can_create_roadmaps: can_manage, can_manage_employees: can_manage, ... }
  }
end
```

### TypeScript: Shared Props Type

```typescript
// types/index.ts — extends Inertia's PageProps
export interface SharedProps extends PageProps {
  auth: { user: User | null; organization: Organization | null };
  flash: { notice?: string; alert?: string };
  features: Features;
}

// Usage in any component:
const { auth, features, flash } = usePage<SharedProps>().props;
```

### TypeScript: Page Props Interface

Each page defines its props inline, mirroring the controller's props hash:

```typescript
interface DashboardProps {
  stats: { roadmapsCount: number; completedSkills: number; overdueCount: number };
  assignedRoadmaps: AssignedRoadmap[];
  isManager?: boolean;           // optional = only present for some roles
  permitAlerts?: PermitAlert[];
}

export default function Dashboard({ stats, assignedRoadmaps, isManager }: DashboardProps) {
  const { t } = useTranslation();
  const { features } = usePage<SharedProps>().props;

  return (
    <MainLayout>
      <h1>{t('dashboard.title')}</h1>
      {isManager && <ManagerSection ... />}
      <Link href={routes.roadmaps()}>{t('dashboard.browse_catalog')}</Link>
    </MainLayout>
  );
}
```

### Centralized Routes

```typescript
// lib/routes.ts — all paths in one place, typed arguments
export const routes = {
  dashboard:           () => "/dashboard",
  roadmaps:            () => "/roadmaps",
  roadmap:         (slug: string) => `/roadmaps/${slug}`,
  editRoadmap:     (slug: string) => `/roadmaps/${slug}/edit`,
  orgEmployees:        () => "/org/employees",
  orgEmployee:      (id: number) => `/org/employees/${id}`,
  // ...
} as const;
```

### Optimistic UI Pattern (favorites, progress toggles)

```typescript
const handleToggleFavorite = useCallback(async () => {
  setIsFavorited(!wasFavorited);                         // 1. Optimistic update
  try {
    const res = await fetch(routes.roadmapFavorite(slug), {
      method: 'POST',
      headers: { 'X-CSRF-Token': csrfToken, 'Accept': 'application/json' },
    });
    const data = await res.json();
    setIsFavorited(data.favorited);                      // 2. Reconcile with server
  } catch {
    setIsFavorited(wasFavorited);                         // 3. Rollback on failure
  }
}, [isFavorited, slug]);
```

### N+1-free Bulk Loading (Skills Matrix pattern)

```ruby
# Load all data in one query, build a lookup hash in Ruby — O(n), not N+1
progresses = UserProgress.where(user_id: employee_ids, skill_id: skill_ids)
matrix = {}
progresses.each do |p|
  matrix["#{p.user_id}:#{p.skill_id}"] = { status: p.status, ... }
end
render inertia: "Org/SkillsMatrix/Index", props: { matrix: matrix, ... }
```

### XyFlow Custom Node

```typescript
// Declared at module level — never inside a component
const nodeTypes = { skillNode: SkillNode, stageHeader: StageHeaderNode };

// Node component wrapped in memo()
function SkillNode({ data }: NodeProps) {
  return (
    <div className="px-4 py-3 rounded-xl border-2 shadow-sm w-[220px]">
      <Handle type="target" position={Position.Top} />
      <span className="text-sm font-medium">{data.label}</span>
      <Handle type="source" position={Position.Bottom} />
    </div>
  );
}
export default memo(SkillNode);
```

## Testing

### Stack
- **Backend:** RSpec 7 + FactoryBot + Shoulda Matchers + Pundit Matchers + SimpleCov
- **Frontend:** Vitest + @testing-library/react + jsdom

### RSpec: Model Spec Pattern (shoulda + FactoryBot + PaperTrail)

```ruby
RSpec.describe User, type: :model do
  describe 'associations' do
    it { should belong_to(:organization) }
    it { should have_many(:roadmap_assignments).dependent(:destroy) }
  end

  describe 'validations' do
    subject { build(:user) }
    it { should validate_presence_of(:email) }
    it { should validate_uniqueness_of(:email).ignoring_case_sensitivity }
    it { should define_enum_for(:role).with_values(employee: 'employee', manager: 'manager', owner: 'owner').backed_by_column_of_type(:string) }
  end

  describe 'audit trail', versioning: true do
    it 'tracks changes to critical fields' do
      user = create(:user, full_name: 'Original Name')
      expect { user.update(full_name: 'New Name') }.to change { user.versions.count }.by(1)
    end
  end

  describe '#can_edit_roadmaps?' do
    it 'returns true for manager' do
      expect(build(:user, :manager).can_edit_roadmaps?).to be true
    end
    it 'returns false for employee' do
      expect(build(:user, role: 'employee').can_edit_roadmaps?).to be false
    end
  end
end
```

### RSpec: Policy Spec Pattern (Pundit Matchers)

```ruby
RSpec.describe RoadmapPolicy, type: :policy do
  let(:organization) { create(:organization) }
  let(:author) { create(:user, organization: organization) }
  let(:roadmap) { create(:roadmap, organization: organization, author: author) }

  describe '#update?' do
    context 'when user is the author' do
      subject { described_class.new(author, roadmap) }
      it { is_expected.to permit_action(:update) }
    end

    context 'when user is a manager in same org' do
      let(:manager) { create(:user, :manager, organization: organization) }
      subject { described_class.new(manager, roadmap) }
      it { is_expected.to permit_action(:update) }
    end

    context 'when user is an employee (not author)' do
      let(:employee) { create(:user, organization: organization, role: 'employee') }
      subject { described_class.new(employee, roadmap) }
      it { is_expected.not_to permit_action(:update) }
    end
  end
end
```

### Factory Conventions

```ruby
# Traits for roles: build(:user, :manager), build(:user, :owner)
# Trait with children: create(:roadmap, :with_skills, skills_count: 5)
# Public template: create(:roadmap, :public)  # org_id=nil, visibility="public"
# Permit skill: create(:skill, :permit)       # auto-associates permit_template
```

## Core Commands
- **Dev Server:** `bin/dev` (Rails + Vite via Procfile.dev)
- **Add UI Component:** Use shadcn MCP tools (see shadcn/ui workflow below)
- **Migrations:** `bin/rails db:migrate`
- **Tests (backend):** `bundle exec rspec`
- **Tests (frontend):** `npx vitest`
- **Type Check:** `npx tsc --noEmit`

## shadcn/ui Workflow

This project has a **shadcn MCP server** configured. Always use it instead of manual CLI commands or guessing component APIs.

### When you need a shadcn/ui component, follow this order:

1. **Search** — find the component by name:
   ```
   mcp__shadcn__search_items_in_registries(registries: ["@shadcn"], query: "dialog")
   ```

2. **View** — read its source, props, and dependencies:
   ```
   mcp__shadcn__view_items_in_registries(items: ["@shadcn/dialog"])
   ```

3. **Examples** — see real usage patterns and demos:
   ```
   mcp__shadcn__get_item_examples_from_registries(registries: ["@shadcn"], query: "dialog-demo")
   ```

4. **Install** — get the correct CLI command and run it:
   ```
   mcp__shadcn__get_add_command_for_items(items: ["@shadcn/dialog"])
   ```

5. **Audit** — after adding components, run the checklist:
   ```
   mcp__shadcn__get_audit_checklist()
   ```

### Rules:
- **Never guess** shadcn component APIs or props — always `view_items` first.
- **Never run** `npx shadcn@latest add ...` manually — use `get_add_command_for_items` to get the correct command.
- **Check examples** before writing complex component compositions (forms, dialogs, popovers).
- **Use `search_items`** when unsure which component fits the need (e.g., "dropdown" → DropdownMenu vs Select vs Combobox).
- Components install to `app/frontend/components/ui/` — this is configured in `components.json`.

## UI Style Rules (Nova Core)

These rules codify the visual design system. Follow them for every new component or page.

### Border Radius
- **Cards, panels, modals, dropdowns:** `rounded-xl` (12px)
- **Buttons, inputs, selects, textareas, badges:** `rounded-lg` (8px)
- **Avatars, progress bars, pills:** `rounded-full`
- **Never use `rounded-md` on new components** — it was the old default, now deprecated.

### Spacing Scale
- **Page content max-width:** `max-w-screen-xl mx-auto` — always wrap page content, never full-bleed.
- **Section gaps on a page:** `mb-6` between sections (not `mb-8`).
- **Grid gaps:** `gap-6` everywhere (not `gap-4` or mixed values).
- **Card internal padding:** `p-4` via `CardHeader` / `CardContent` — do not override unless justified.
- **List item rows inside a card:** `space-y-3` or `space-y-4`, never `space-y-2` (too tight).

### Card Component
The `Card` uses a ring instead of a border+shadow. Do not add `shadow-sm` or `border` back on top.

```tsx
// Correct — ring already applied by Card base
<Card>
  <CardHeader>
    <CardTitle>Section title</CardTitle>          {/* text-base by default — do not override */}
    <CardDescription>Subtitle</CardDescription>
  </CardHeader>
  <CardContent>...</CardContent>
</Card>

// Wrong
<Card className="border shadow-sm rounded-lg">   {/* re-adding old style */}
  <CardHeader>
    <CardTitle className="text-base">...</CardTitle>  {/* redundant — text-base is default */}
  </CardHeader>
</Card>
```

### Typography Hierarchy
| Element | Class |
|---------|-------|
| Page title (`<h1>`) | `text-2xl font-bold` |
| Page subtitle | `text-sm text-muted-foreground mt-1` |
| Card / section title (`CardTitle`) | `text-base font-semibold` — **this is the default, do not add `className="text-base"`** |
| Stat card label | `<CardTitle className="text-sm font-medium">` |
| Body text | `text-sm` |
| Secondary / meta text | `text-xs text-muted-foreground` |

### Color Rules — Status & Severity

Use semantic CSS variables where available. For colors without a semantic variable, always include a `dark:` variant.

| Meaning | Class |
|---------|-------|
| Error / danger / overdue | `text-destructive` |
| Success / compliant / active | `text-emerald-600 dark:text-emerald-400` |
| Warning / expiring / medium risk | `text-amber-600 dark:text-amber-400` |
| Low severity | `text-yellow-600 dark:text-yellow-400` |
| Trend up (positive) | `text-emerald-600 dark:text-emerald-400` |
| Trend down (negative) | `text-destructive` |
| Neutral / info | `text-muted-foreground` |

**Never write `text-red-500`, `text-orange-500`, `text-green-600` without a `dark:` pair.** If unsure, use `text-destructive` (red) or `text-muted-foreground` (neutral).

### Status Icons
Always pair icon colors with the same semantic color as text:

```tsx
<AlertTriangle className="w-5 h-5 text-amber-500 dark:text-amber-400" />  // warning
<ShieldCheck   className="w-5 h-5 text-emerald-600 dark:text-emerald-400" /> // success
<ShieldX       className="w-5 h-5 text-destructive" />                      // error
```

### Progress Bar Colors
Use `indicatorClassName` prop, **never `[&>div]:bg-*`** arbitrary selectors.

```tsx
// Correct
<Progress value={pct} size="sm" indicatorClassName={complianceIndicatorClass(pct)} />

// Wrong
<Progress value={pct} size="sm" className={`[&>div]:bg-emerald-500`} />
```

The `complianceIndicatorClass(percent)` helper lives in `components/dashboard/complianceColor.ts` and returns the right `indicatorClassName` for green/amber/red thresholds.

### Component Sizing
Button, Input, Select, Textarea — all standardized to height `h-9` (not `h-10`).

```tsx
<Button size="default">  // h-9  — default
<Button size="sm">       // h-8
<Button size="lg">       // h-10
<Button size="icon">     // h-9 w-9
```

### Page Layout Pattern

Every authenticated page must follow this structure:

```tsx
<MainLayout>
  {/* 1. Page header */}
  <div className="mb-6">
    <h1 className="text-2xl font-bold">{t('section.title')}</h1>
    <p className="text-sm text-muted-foreground mt-1">{t('section.subtitle')}</p>
  </div>

  {/* 2. Primary KPIs / action items */}
  <div className="grid gap-6 md:grid-cols-2 lg:grid-cols-4 mb-6">
    ...
  </div>

  {/* 3. Main content */}
  <div className="grid gap-6 lg:grid-cols-2 mb-6">
    ...
  </div>

  {/* 4. Secondary / analytics (manager-only or advanced) */}
  ...
</MainLayout>
```

### Anti-patterns — UI Style

- **No `rounded-md`** on new components. Use `rounded-xl` (panels) or `rounded-lg` (controls).
- **No `shadow-sm` on `<Card>`** — the ring replaces the shadow.
- **No `className="text-base"` on `<CardTitle>`** — it is the default. Override only for `text-sm` (stat card labels).
- **No hardcoded colors without dark: variant** — `text-red-500` alone breaks dark mode.
- **No `[&>div]:bg-*`** — use `indicatorClassName` on `<Progress>` instead.
- **No `mb-8` for section spacing** — use `mb-6` consistently.
- **No `gap-4` in page-level grids** — use `gap-6`.

## Documentation
- Project docs in `/docs/` — start with `docs/README.md` for navigation.

---
> Source: [andreiyurik/v7-industrialprofi-platform-v7](https://github.com/andreiyurik/v7-industrialprofi-platform-v7) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
