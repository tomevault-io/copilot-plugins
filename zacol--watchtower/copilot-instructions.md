## watchtower

> - **ALWAYS** use JWT sessions (`strategy: "jwt"`)

# Security Rules

## Authentication

### Session Management

- **ALWAYS** use JWT sessions (`strategy: "jwt"`)
- Check session in every Server Action
- Check session in every API route
- Never trust client-side session data

```typescript
// ✅ CORRECT - Verify session server-side
export async function deleteProject(projectId: string) {
  const session = await getAuthSession()
  if (!session?.user?.id) {
    return { error: 'Unauthorized' }
  }
  // Continue...
}
```

### Session Validation Pattern

```typescript
// lib/auth.ts
export async function requireAuth() {
  const session = await getAuthSession()
  if (!session?.user?.id) {
    redirect('/api/auth/signin')
  }
  return session
}

// Usage in Server Actions
const session = await requireAuth()
```

## Authorization

### ALWAYS Check Project Membership

```typescript
// ✅ CORRECT - Verify user is project member
export async function createRisk(projectId: string, data: RiskInput) {
  const session = await requireAuth()
  
  // Check membership
  const member = await db.projectMember.findUnique({
    where: {
      projectId_userId: {
        projectId,
        userId: session.user.id
      }
    }
  })
  
  if (!member) {
    return { error: 'Forbidden' }
  }
  
  // Check role if needed
  if (requiresPM && member.role !== 'PM') {
    return { error: 'Requires PM role' }
  }
  
  // Continue...
}
```

### Permission Checks (MANDATORY)

Every operation must verify:
1. **Authentication** - Is user logged in?
2. **Project Access** - Is user member of this project?
3. **Role Permission** - Does user role allow this action?

### Authorization Matrix

| Action | Global Role | Project Role | Additional Checks |
|--------|-------------|--------------|-------------------|
| Create Project | USER | - | - |
| Add Member | - | PM | Project exists |
| Create Risk | - | MEMBER or PM | Project active |
| Create Campaign | - | PM | Project active |
| Activate Campaign | - | PM | Campaign is DRAFT |
| Submit Answer | - | MEMBER or PM | Campaign ACTIVE, not expired, one per user |
| View Results | - | PM | Campaign CLOSED |
| Delete Project | - | PM | - |

## Input Validation

### Use Zod for Validation

```typescript
import { z } from 'zod'

const createRiskSchema = z.object({
  title: z.string().min(3).max(200),
  description: z.string().max(2000),
  severity: z.enum(['LOW', 'MEDIUM', 'HIGH', 'CRITICAL']),
  projectId: z.string().uuid(),
})

export async function createRisk(input: unknown) {
  // Validate first
  const validated = createRiskSchema.safeParse(input)
  if (!validated.success) {
    return { error: 'Invalid input', details: validated.error.flatten() }
  }
  
  const data = validated.data
  // Continue with validated data...
}
```

### Validation Rules

- **Validate ALL user input** - Never trust client data
- **Sanitize text** - Prevent XSS via proper escaping (React does this by default)
- **Limit lengths** - Prevent DoS via large payloads
- **Validate UUIDs** - Ensure proper format for IDs
- **Validate enums** - Only allow defined values

## SQL Injection Prevention

### Use Prisma Parameterized Queries (DEFAULT)

```typescript
// ✅ Prisma automatically parameterizes
const user = await db.user.findUnique({
  where: { email: userInput }  // Safe - parameterized
})

// ❌ NEVER use raw SQL with user input
const users = await db.$queryRaw`SELECT * FROM users WHERE email = ${userInput}`
// Only use $queryRaw with hardcoded queries or properly sanitized inputs
```

## Anonymization & Privacy

### Critical Privacy Rules

1. **NEVER expose `userId` in responses/reports**
2. **NEVER send identifying data to AI**
3. **Always use `anonymousHash` internally**
4. **Hide open text if team size < threshold**
5. **Aggregate data server-side only**

### Anonymization Implementation

```typescript
// ✅ CORRECT - Generate anonymous hash
import { generateAnonymousHash } from '@/lib/hashing'

const anonymousHash = generateAnonymousHash(userId, campaignId, secret)

// Store with hash, not userId
await db.happinessAnswer.create({
  data: {
    campaignId,
    questionId,
    textValue: answer,
    anonymousHash,  // ✅ Hash stored
    // userId NOT stored in answers table
  }
})
```

### Text Response Privacy

```typescript
// domain/happiness/openTextPolicy.ts
export function canShowOpenText(teamSize: number, minThreshold = 3): boolean {
  return teamSize >= minThreshold
}

// Usage in aggregation
if (!canShowOpenText(memberCount)) {
  return {
    ...summary,
    openTextResponses: [],
    notice: 'Text responses hidden due to small team size'
  }
}
```

## Campaign Expiry Validation

### Prevent Late Submissions

```typescript
export async function submitAnswer(campaignId: string, answers: Answer[]) {
  const campaign = await db.happinessCampaign.findUnique({
    where: { id: campaignId }
  })
  
  // Check status
  if (campaign.status !== 'ACTIVE') {
    return { error: 'Campaign not active' }
  }
  
  // Check expiry
  if (campaign.endsAt && new Date() > campaign.endsAt) {
    return { error: 'Campaign has ended' }
  }
  
  // Continue...
}
```

## Rate Limiting (RECOMMENDED)

### Prevent Answer Spam

```typescript
// Check one answer per user per campaign
const existingAnswer = await db.happinessAnswer.findFirst({
  where: {
    campaignId,
    anonymousHash: generateAnonymousHash(userId, campaignId, secret)
  }
})

if (existingAnswer) {
  return { error: 'Already submitted' }
}
```

## Secrets Management

### Environment Variables

```typescript
// ✅ CORRECT - Use env vars
const openAiKey = process.env.OPENAI_API_KEY
const hashSecret = process.env.ANONYMOUS_HASH_SECRET

// ❌ NEVER hardcode secrets
const openAiKey = 'sk-...'  // NEVER DO THIS
```

### Secret Rotation Support

```typescript
// lib/hashing.ts
const HASH_SECRETS = [
  process.env.ANONYMOUS_HASH_SECRET!,
  process.env.ANONYMOUS_HASH_SECRET_OLD // For rotation
].filter(Boolean)

export function generateAnonymousHash(
  userId: string,
  campaignId: string
): string {
  const secret = HASH_SECRETS[0]  // Use current secret
  return crypto
    .createHmac('sha256', secret)
    .update(`${userId}:${campaignId}`)
    .digest('hex')
}
```

## Data Exposure Prevention

### DTO Pattern (MANDATORY)

```typescript
// ❌ WRONG - Exposes internal fields
export async function getCampaign(id: string) {
  const campaign = await db.happinessCampaign.findUnique({
    where: { id },
    include: { answers: true }  // Includes anonymousHash!
  })
  return campaign  // Exposes sensitive data
}

// ✅ CORRECT - Use DTO mapper
export async function getCampaign(id: string) {
  const campaign = await db.happinessCampaign.findUnique({
    where: { id },
    include: { answers: true }
  })
  return campaignToDTO(campaign)  // Filters sensitive fields
}
```

## AI Safety

### Never Send PII to AI

```typescript
// ✅ CORRECT - Remove identifying info
const aiInput = {
  responses: answers.map(a => ({
    text: a.textValue,
    rating: a.numericValue,
    // NO userId, NO anonymousHash, NO names
  }))
}

const analysis = await openai.chat.completions.create({
  messages: [{
    role: 'system',
    content: 'Analyze team happiness. Do not identify individuals.'
  }, {
    role: 'user',
    content: JSON.stringify(aiInput)
  }]
})
```

### Validate AI Responses

- Don't trust AI output blindly
- Validate JSON structure
- Sanitize before storing
- Handle errors gracefully

## CORS & API Security

### API Routes Protection

```typescript
// app/api/cron/close-campaigns/route.ts
export async function GET(request: Request) {
  // Verify cron secret
  const authHeader = request.headers.get('authorization')
  if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
    return new Response('Unauthorized', { status: 401 })
  }
  
  // Continue...
}
```

## Audit Logging (RECOMMENDED)

### Log Critical Actions

- User login/logout
- Campaign activation
- Campaign closure
- AI analysis triggers
- Permission changes
- Project deletions

```typescript
// Example audit log
await db.auditLog.create({
  data: {
    action: 'CAMPAIGN_CLOSED',
    userId: session.user.id,
    projectId,
    campaignId,
    timestamp: new Date(),
    metadata: { reason: 'auto' }
  }
})
```

## Error Messages

### Safe Error Handling

```typescript
// ❌ WRONG - Exposes internal details
catch (error) {
  return { error: error.message }  // May leak DB structure
}

// ✅ CORRECT - Generic message
catch (error) {
  console.error('Failed to create risk:', error)  // Log server-side
  return { error: 'Failed to create risk' }  // Generic to client
}
```

## Dependencies

### Regular Updates
- Keep dependencies updated
- Monitor for security advisories
- Use `npm audit` regularly
- Pin versions in production

### Minimal Dependencies
- Only add necessary packages
- Audit package permissions
- Prefer maintained packages
- Avoid deprecated libraries

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zacol) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
