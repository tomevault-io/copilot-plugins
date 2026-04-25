## a16z-makeathon

> **Vision:** Transform daily standups into living documentation through real-time AI transcription and intelligent document updates.

# StandupSync - AI-Native Documentation Assistant

**Vision:** Transform daily standups into living documentation through real-time AI transcription and intelligent document updates.

## Tech Stack

### Frontend
- **Next.js 15** (App Router) - Primary framework
- **TypeScript** - Strict mode enabled
- **Tailwind CSS** - Utility-first styling
- **shadcn/ui** - Component library (from v0)
- **Vercel AI SDK** - Streaming UI updates, real-time transcription display

### Backend & APIs
- **Next.js API Routes** - Serverless functions
- **OpenAI API** - GPT-4o for audio transcription + structured generation
- **Instructor** - Type-safe LLM outputs
- **Voyage AI** - Text embeddings

### Data & Storage
- **Supabase** - PostgreSQL database, authentication, real-time subscriptions
- **Qdrant Cloud** - Vector database for semantic search

### Deployment
- **Vercel** - Frontend and API hosting

## Project Structure

```
/app
  /api
    /transcribe       # GPT-4o audio transcription endpoint
    /analyze          # Process transcript chunks
    /docs             # CRUD operations for documents
  /dashboard          # Main UI page
  /docs               # Document viewer/editor
/components
  /ui                 # shadcn components from v0
  /audio              # Recording controls, live transcription display
  /documents          # Doc list, doc viewer, update indicators
/lib
  /openai            # GPT-4o transcription + generation utilities
  /ai                # LLM structured generation (instructor)
  /embeddings        # Voyage AI embedding functions
  /vectordb          # Qdrant client and search
  /supabase          # Database client and queries
/instructions        # Reference docs and code samples
/types               # TypeScript definitions
```

## Development Guidelines

### Code Patterns

1. **Server Components by Default**
   - Use `'use client'` only when needed (interactivity, hooks)
   - Keep AI logic server-side only

2. **Type Safety**
   - All API responses must have TypeScript interfaces
   - Use Zod for runtime validation
   - Instructor ensures structured LLM outputs

3. **Streaming Responses**
   - Use Vercel AI SDK's `streamText` for real-time updates
   - Server-Sent Events for live transcription
   - Show loading states immediately

4. **Error Handling**
   - Try/catch all async operations
   - Return user-friendly error messages
   - Log detailed errors server-side only
   - Never expose API keys or internal errors

5. **Security**
   - All secrets in `.env.local` (never commit!)
   - Validate all user inputs server-side
   - Use Supabase RLS for data access control
   - Rate limit API routes

### Component Conventions

- **Naming:** PascalCase for components, kebab-case for files
- **Structure:** One component per file
- **Props:** Define TypeScript interface for all props
- **Reusability:** Extract common patterns into `/components/ui`

### Styling

- **Tailwind only** - No custom CSS unless absolutely necessary
- **Responsive first** - Mobile → Desktop
- **Dark mode** - Use `dark:` variants from shadcn/ui
- **Consistent spacing** - Use Tailwind spacing scale (4, 8, 12, 16...)

### Database

**Tables:**
- `documents` - id, user_id, title, content (markdown), created_at, updated_at
- `standups` - id, user_id, transcript, audio_url, created_at
- `doc_updates` - id, standup_id, document_id, section, new_content, created_at

**Supabase RLS:**
- Users can only access their own documents and standups
- Enable RLS on all tables

### API Routes

**Pattern:**
```typescript
// app/api/example/route.ts
export async function POST(req: Request) {
  try {
    const body = await req.json()
    // Validate with Zod
    // Process
    return Response.json({ success: true, data })
  } catch (error) {
    console.error('Detailed error:', error)
    return Response.json(
      { error: 'User-friendly message' },
      { status: 500 }
    )
  }
}
```

### AI Generation Rules

1. **Use instructor for structured outputs** - Never parse raw LLM text
2. **Streaming for UX** - Show progress for operations >2 seconds
3. **Fallbacks** - If AI fails, don't crash, show graceful error
4. **Context management** - Only send relevant docs to LLM (use embeddings to filter)

### Environment Variables

Required in `.env.local`:
```
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
OPENAI_API_KEY=
VOYAGE_API_KEY=
QDRANT_URL=
QDRANT_API_KEY=
```

## Common Workflows

### Adding a New Feature
1. Break into phases (don't do everything at once)
2. Start with types/interfaces
3. Build API route
4. Add UI component
5. Test each phase before moving on
6. Commit after each working feature

### Debugging
1. Check browser console for client errors
2. Check Vercel logs for server errors
3. Verify environment variables loaded
4. Test API routes independently (curl/Postman)
5. If stuck after 3 attempts, start new chat with detailed context

### Git Workflow
- Commit after each completed feature
- Descriptive commit messages: "feat: add live transcription display"
- Never commit `.env.local` or `node_modules`

## DO NOT

- Do not change files unless explicitly asked
- Do not add features not in the current prompt
- Do not use external CSS files
- Do not hardcode API keys
- Do not skip error handling
- Do not create one-off test files (use proper test structure)
- Do not make assumptions about user intent - ask if unclear

## Testing Before Demo

1. **Happy path:** Record → transcribe → doc update works
2. **Error handling:** What if API fails? Show user-friendly error
3. **Empty states:** No docs yet, no standups yet
4. **Loading states:** Show spinners during async operations
5. **Mobile responsive:** Test on small screen

## Demo Script

1. Show empty state
2. Create a sample document (e.g., "Sprint 1 Status")
3. Click "Record Standup"
4. Speak: "Yesterday I fixed the Redis timeout bug. Today I'm working on the new API endpoint. No blockers."
5. Show live transcription appearing
6. Show AI analyzing and deciding to update document
7. Show live update in the document (seamless insertion)
8. Stop recording
9. Show final updated document with new content integrated

---

**Remember:** This is a 3-hour hackathon project. Prioritize working features over perfect code. Focus on the demo flow above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thenerd31) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
