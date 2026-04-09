## watchtower

> // app/dashboard/[projectId]/page.tsx

# Frontend Development Rules

## Component Architecture

### Server Components (DEFAULT)

```typescript
// app/dashboard/[projectId]/page.tsx
import { getCampaigns } from '@/app/actions/happiness/getCampaigns'
import { CampaignList } from './CampaignList'

export default async function CampaignsPage({ 
  params 
}: { 
  params: { projectId: string } 
}) {
  // Fetch data in Server Component
  const result = await getCampaigns(params.projectId)
  
  if (!result.success) {
    return <ErrorState message={result.error} />
  }

  // Pass DTOs to Client Component
  return <CampaignList campaigns={result.data} />
}
```

### Client Components (When Needed)

```typescript
'use client'

import { useState } from 'react'
import { submitAnswer } from '@/app/actions/happiness/submitAnswer'
import { toast } from 'sonner'
import type { HappinessCampaignDTO } from '@/types/happiness'

interface Props {
  campaign: HappinessCampaignDTO
}

export function AnswerForm({ campaign }: Props) {
  const [isSubmitting, setIsSubmitting] = useState(false)

  async function handleSubmit(formData: FormData) {
    setIsSubmitting(true)
    
    const result = await submitAnswer({
      campaignId: campaign.id,
      answers: extractAnswers(formData)
    })

    setIsSubmitting(false)

    if (!result.success) {
      toast.error(result.error)
      return
    }

    toast.success('Answer submitted')
    // Handle success (e.g., redirect)
  }

  return (
    <form action={handleSubmit}>
      {/* Form fields */}
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  )
}
```

## Component Organization

### File Structure

```
components/
├─ ui/                    # shadcn/ui or base components
│  ├─ button.tsx
│  ├─ card.tsx
│  ├─ dialog.tsx
│  └─ form.tsx
├─ happiness/             # Domain-specific components
│  ├─ CampaignCard.tsx
│  ├─ AnswerForm.tsx
│  ├─ ResultsChart.tsx
│  └─ AIAnalysisView.tsx
├─ risks/
│  ├─ RiskList.tsx
│  ├─ RiskForm.tsx
│  └─ RiskCard.tsx
└─ layout/               # Layout components
   ├─ Header.tsx
   ├─ Sidebar.tsx
   └─ Footer.tsx
```

### Component Naming

- **PascalCase** for component files: `CampaignCard.tsx`
- **Descriptive names**: `CreateCampaignDialog.tsx` not `Dialog.tsx`
- **Domain prefix** when needed: `HappinessCampaignList.tsx`

## Props & Types

### Always Use Typed Props

```typescript
// ✅ CORRECT - Explicit type
interface CampaignCardProps {
  campaign: HappinessCampaignDTO
  onActivate?: (id: string) => void
  showActions?: boolean
}

export function CampaignCard({ 
  campaign, 
  onActivate,
  showActions = true 
}: CampaignCardProps) {
  // Component logic
}

// ❌ WRONG - No types
export function CampaignCard({ campaign, onActivate }) {
  // TypeScript can't help you here
}
```

### Use DTOs, Not Prisma Types

```typescript
// ❌ WRONG
import type { HappinessCampaign } from '@prisma/client'
interface Props {
  campaign: HappinessCampaign
}

// ✅ CORRECT
import type { HappinessCampaignDTO } from '@/types/happiness'
interface Props {
  campaign: HappinessCampaignDTO
}
```

## Forms & Server Actions

### Form Handling with Server Actions

```typescript
'use client'

import { useFormState, useFormStatus } from 'react-dom'
import { createRisk } from '@/app/actions/risks/createRisk'

export function CreateRiskForm({ projectId }: { projectId: string }) {
  const [state, formAction] = useFormState(createRisk, null)

  return (
    <form action={formAction}>
      <input type="hidden" name="projectId" value={projectId} />
      
      <div>
        <label htmlFor="title">Title</label>
        <input 
          id="title" 
          name="title" 
          required 
          minLength={3}
          maxLength={200}
        />
      </div>

      <div>
        <label htmlFor="severity">Severity</label>
        <select id="severity" name="severity" required>
          <option value="LOW">Low</option>
          <option value="MEDIUM">Medium</option>
          <option value="HIGH">High</option>
          <option value="CRITICAL">Critical</option>
        </select>
      </div>

      {state?.error && (
        <div className="text-red-500">{state.error}</div>
      )}

      <SubmitButton />
    </form>
  )
}

function SubmitButton() {
  const { pending } = useFormStatus()
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Creating...' : 'Create Risk'}
    </button>
  )
}
```

### Progressive Enhancement

```typescript
// Form works without JavaScript
export async function createRisk(
  prevState: any,
  formData: FormData
): Promise<{ success?: boolean; error?: string }> {
  const data = {
    projectId: formData.get('projectId') as string,
    title: formData.get('title') as string,
    severity: formData.get('severity') as string
  }

  const result = await createRiskAction(data)
  
  if (!result.success) {
    return { error: result.error }
  }

  redirect(`/dashboard/${data.projectId}/risks`)
}
```

## State Management

### Local State (useState)

```typescript
'use client'

export function CampaignFilters() {
  const [status, setStatus] = useState<'ALL' | 'ACTIVE' | 'CLOSED'>('ALL')
  const [search, setSearch] = useState('')

  return (
    <div>
      <input
        value={search}
        onChange={(e) => setSearch(e.target.value)}
        placeholder="Search campaigns..."
      />
      <select value={status} onChange={(e) => setStatus(e.target.value)}>
        <option value="ALL">All</option>
        <option value="ACTIVE">Active</option>
        <option value="CLOSED">Closed</option>
      </select>
    </div>
  )
}
```

### URL State (useSearchParams)

```typescript
'use client'

import { useSearchParams, useRouter } from 'next/navigation'

export function CampaignFilters() {
  const router = useRouter()
  const searchParams = useSearchParams()
  
  const status = searchParams.get('status') || 'ALL'

  function setStatus(newStatus: string) {
    const params = new URLSearchParams(searchParams)
    params.set('status', newStatus)
    router.push(`?${params.toString()}`)
  }

  return (
    <select value={status} onChange={(e) => setStatus(e.target.value)}>
      <option value="ALL">All</option>
      <option value="ACTIVE">Active</option>
      <option value="CLOSED">Closed</option>
    </select>
  )
}
```

### Server State (Server Components)

```typescript
// Use searchParams in Server Components
export default async function CampaignsPage({
  params,
  searchParams
}: {
  params: { projectId: string }
  searchParams: { status?: string; search?: string }
}) {
  const result = await getCampaigns(
    params.projectId,
    searchParams.status,
    searchParams.search
  )

  return <CampaignList campaigns={result.data} />
}
```

## Data Fetching

### Server Components (Preferred)

```typescript
// ✅ Fetch in Server Component
export default async function ProjectPage({ 
  params 
}: { 
  params: { id: string } 
}) {
  const project = await getProject(params.id)
  return <ProjectView project={project} />
}
```

### Client Components (When Needed)

```typescript
'use client'

import { useEffect, useState } from 'react'

export function LiveCampaignStatus({ campaignId }: { campaignId: string }) {
  const [status, setStatus] = useState<CampaignStatusDTO | null>(null)

  useEffect(() => {
    const interval = setInterval(async () => {
      const result = await getCampaignStatus(campaignId)
      if (result.success) {
        setStatus(result.data)
      }
    }, 5000) // Poll every 5 seconds

    return () => clearInterval(interval)
  }, [campaignId])

  return <StatusBadge status={status} />
}
```

## Styling

### TailwindCSS Best Practices

```typescript
// ✅ Use semantic class names
export function CampaignCard({ campaign }: Props) {
  return (
    <div className="rounded-lg border bg-card p-6 shadow-sm">
      <h3 className="text-xl font-semibold">{campaign.title}</h3>
      <p className="mt-2 text-muted-foreground">{campaign.description}</p>
      
      <div className="mt-4 flex items-center gap-2">
        <StatusBadge status={campaign.status} />
        <span className="text-sm text-muted-foreground">
          {campaign.responseCount} responses
        </span>
      </div>
    </div>
  )
}

// ✅ Extract repeated patterns
const statusStyles = {
  DRAFT: 'bg-gray-100 text-gray-800',
  ACTIVE: 'bg-green-100 text-green-800',
  CLOSED: 'bg-blue-100 text-blue-800'
} as const

export function StatusBadge({ status }: { status: keyof typeof statusStyles }) {
  return (
    <span className={`rounded-full px-2 py-1 text-xs font-medium ${statusStyles[status]}`}>
      {status}
    </span>
  )
}
```

### CSS Modules (Alternative)

```typescript
// components/CampaignCard.module.css
.card {
  border-radius: 0.5rem;
  border: 1px solid var(--border);
  padding: 1.5rem;
}

// components/CampaignCard.tsx
import styles from './CampaignCard.module.css'

export function CampaignCard({ campaign }: Props) {
  return <div className={styles.card}>{/* ... */}</div>
}
```

## Error Handling

### Error Boundaries

```typescript
// app/error.tsx
'use client'

export default function Error({
  error,
  reset
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <div className="text-center">
        <h2 className="text-2xl font-bold">Something went wrong!</h2>
        <p className="mt-2 text-muted-foreground">{error.message}</p>
        <button
          onClick={reset}
          className="mt-4 rounded bg-primary px-4 py-2 text-primary-foreground"
        >
          Try again
        </button>
      </div>
    </div>
  )
}
```

### Loading States

```typescript
// app/dashboard/[projectId]/loading.tsx
export default function Loading() {
  return (
    <div className="space-y-4">
      <div className="h-8 w-48 animate-pulse rounded bg-muted" />
      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-3">
        {[...Array(6)].map((_, i) => (
          <div key={i} className="h-32 animate-pulse rounded bg-muted" />
        ))}
      </div>
    </div>
  )
}
```

### Toast Notifications

```typescript
'use client'

import { toast } from 'sonner'

export function DeleteRiskButton({ riskId }: { riskId: string }) {
  async function handleDelete() {
    const result = await deleteRisk(riskId)
    
    if (!result.success) {
      toast.error(result.error)
      return
    }

    toast.success('Risk deleted successfully')
    router.refresh() // Refresh Server Component data
  }

  return (
    <button onClick={handleDelete} className="text-destructive">
      Delete
    </button>
  )
}
```

## Accessibility

### Semantic HTML

```typescript
// ✅ CORRECT - Semantic elements
export function CampaignList({ campaigns }: Props) {
  return (
    <section aria-labelledby="campaigns-heading">
      <h2 id="campaigns-heading" className="sr-only">
        Happiness Campaigns
      </h2>
      <ul className="space-y-4">
        {campaigns.map((campaign) => (
          <li key={campaign.id}>
            <CampaignCard campaign={campaign} />
          </li>
        ))}
      </ul>
    </section>
  )
}

// ❌ WRONG - Divs everywhere
export function CampaignList({ campaigns }: Props) {
  return (
    <div>
      <div>Happiness Campaigns</div>
      <div>
        {campaigns.map((campaign) => (
          <div key={campaign.id}>
            <CampaignCard campaign={campaign} />
          </div>
        ))}
      </div>
    </div>
  )
}
```

### Form Labels & ARIA

```typescript
export function AnswerForm() {
  return (
    <form>
      {/* Always associate labels */}
      <div>
        <label htmlFor="happiness-rating">
          How happy are you with the project?
        </label>
        <input
          id="happiness-rating"
          type="range"
          min="1"
          max="5"
          aria-describedby="rating-description"
        />
        <p id="rating-description" className="text-sm text-muted-foreground">
          1 = Very unhappy, 5 = Very happy
        </p>
      </div>

      {/* Accessible buttons */}
      <button
        type="submit"
        aria-label="Submit happiness survey"
      >
        Submit
      </button>
    </form>
  )
}
```

### Keyboard Navigation

```typescript
'use client'

export function CampaignCard({ campaign, onSelect }: Props) {
  return (
    <div
      role="button"
      tabIndex={0}
      onClick={() => onSelect(campaign.id)}
      onKeyDown={(e) => {
        if (e.key === 'Enter' || e.key === ' ') {
          e.preventDefault()
          onSelect(campaign.id)
        }
      }}
      className="cursor-pointer rounded-lg border p-4 hover:bg-accent focus:outline-none focus:ring-2 focus:ring-primary"
    >
      {campaign.title}
    </div>
  )
}
```

## Performance Optimization

### Image Optimization

```typescript
import Image from 'next/image'

export function UserAvatar({ user }: { user: { image: string; name: string } }) {
  return (
    <Image
      src={user.image}
      alt={`${user.name}'s avatar`}
      width={40}
      height={40}
      className="rounded-full"
      priority={false} // Don't prioritize unless above fold
    />
  )
}
```

### Dynamic Imports

```typescript
// Lazy load heavy components
import dynamic from 'next/dynamic'

const AIAnalysisChart = dynamic(
  () => import('./AIAnalysisChart'),
  {
    loading: () => <ChartSkeleton />,
    ssr: false // Don't render on server
  }
)

export function CampaignResults({ campaign }: Props) {
  return (
    <div>
      <ResultsSummary campaign={campaign} />
      {campaign.aiStatus === 'COMPLETED' && (
        <AIAnalysisChart analysis={campaign.aiSummary} />
      )}
    </div>
  )
}
```

### Memoization

```typescript
'use client'

import { useMemo } from 'react'

export function CampaignStats({ answers }: { answers: AnswerDTO[] }) {
  // Expensive calculation - memoize it
  const stats = useMemo(() => {
    return {
      average: answers.reduce((sum, a) => sum + a.value, 0) / answers.length,
      median: calculateMedian(answers),
      distribution: calculateDistribution(answers)
    }
  }, [answers])

  return <StatsDisplay stats={stats} />
}
```

## Data Visualization

### Chart Libraries

```typescript
'use client'

import { Bar } from 'react-chartjs-2'
import type { HappinessSummaryDTO } from '@/types/happiness'

export function HappinessChart({ summary }: { summary: HappinessSummaryDTO }) {
  const data = {
    labels: ['Very Unhappy', 'Unhappy', 'Neutral', 'Happy', 'Very Happy'],
    datasets: [{
      label: 'Responses',
      data: summary.distribution,
      backgroundColor: 'rgba(59, 130, 246, 0.5)'
    }]
  }

  return (
    <div className="h-64">
      <Bar data={data} options={{ maintainAspectRatio: false }} />
    </div>
  )
}
```

## Testing

### Component Tests

```typescript
// __tests__/components/CampaignCard.test.tsx
import { render, screen } from '@testing-library/react'
import { CampaignCard } from '@/components/happiness/CampaignCard'

describe('CampaignCard', () => {
  const mockCampaign = {
    id: '1',
    title: 'Q1 Happiness Survey',
    status: 'ACTIVE' as const,
    responseCount: 15
  }

  it('renders campaign title', () => {
    render(<CampaignCard campaign={mockCampaign} />)
    expect(screen.getByText('Q1 Happiness Survey')).toBeInTheDocument()
  })

  it('displays response count', () => {
    render(<CampaignCard campaign={mockCampaign} />)
    expect(screen.getByText(/15 responses/i)).toBeInTheDocument()
  })

  it('shows correct status badge', () => {
    render(<CampaignCard campaign={mockCampaign} />)
    expect(screen.getByText('ACTIVE')).toBeInTheDocument()
  })
})
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zacol)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/zacol)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
