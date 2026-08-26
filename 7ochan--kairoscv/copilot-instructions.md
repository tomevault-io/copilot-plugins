## kairoscv

> **Project Name:** KairosCV - AI-Powered Resume Optimization Platform

# KairosCV - AI Agent Performance Optimization Guide

## 🎯 Project Overview

**Project Name:** KairosCV - AI-Powered Resume Optimization Platform
**Current Status:** Transition to HTML-to-PDF approach (Branch: bharath-013)
**Goal:** Build an MVP that converts any resume format into ATS-optimized PDFs using AI enhancement

**CRITICAL:** This project uses **Next.js 16 + TypeScript**, NOT FastAPI/Python. The stack has evolved from the original design.

---

## 📐 Current Architecture (ACCURATE AS OF NOV 2025)

### Actual Technology Stack
```
Framework:     Next.js 16 (App Router)
Language:      TypeScript
Runtime:       Node.js 18+
Package Mgr:   pnpm
UI:            React 19 + Radix UI + Tailwind CSS
AI:            Google Gemini 1.5 Flash API
Parsers:       pdf-parse, mammoth, pdf-lib
PDF Gen:       Puppeteer (NEW - replacing pdf-lib)
Templates:     Handlebars (NEW)
Validation:    Zod (NEW)
Hosting:       Render.com
```

### Actual File Structure
```
kairosCV/
├── app/
│   ├── api/
│   │   ├── upload/route.ts          # File upload endpoint
│   │   ├── stream/[fileId]/route.ts # SSE progress streaming
│   │   ├── download/[fileId]/route.ts # PDF download
│   │   └── health/route.ts          # Health check
│   ├── page.tsx                      # Main UI
│   └── layout.tsx                    # App layout
├── lib/
│   ├── resume-processor.ts          # Main processing pipeline
│   ├── file-storage.ts              # File I/O utilities
│   ├── parsers/                     # NEW - Enhanced parsers
│   ├── ai/                          # NEW - Gemini integration
│   ├── templates/                   # NEW - HTML templates
│   ├── pdf/                         # NEW - Puppeteer generator
│   └── schemas/                     # NEW - Zod schemas
├── components/                       # React UI components
├── public/                          # Static assets
└── uploads/                         # Temporary file storage
```

### Data Flow (CURRENT)
```
User Upload → Next.js API Route → File Validation →
Parse (PDF/DOCX/TXT) → Extract Sections →
Gemini AI Enhancement → HTML Template Population →
Puppeteer PDF Generation → Download Link (SSE Progress Updates)
```

---

## 🚀 AI Agent Performance Optimization Rules

### Rule 1: Always Read Files Before Editing
**WHY:** Next.js routing and file structure must be preserved exactly
**HOW:** Always use the Read tool before Edit or Write tools
**EXAMPLE:**
```typescript
// ❌ BAD: Writing without reading first
Write("app/api/new-endpoint/route.ts", content)

// ✅ GOOD: Read existing patterns, then create similar
Read("app/api/upload/route.ts")
// ... then create new endpoint following the same pattern
```

### Rule 2: Use Type-Safe Patterns
**WHY:** TypeScript catches errors at compile time, not runtime
**HOW:** Always define types/interfaces before implementation
**EXAMPLE:**
```typescript
// ✅ Define schema first with Zod
export const ResumeDataSchema = z.object({
  contact: ContactSchema,
  experience: z.array(ExperienceSchema),
  // ...
})

export type ResumeData = z.infer<typeof ResumeDataSchema>

// Then use in functions
function processResume(data: ResumeData): Promise<Buffer> {
  // TypeScript will validate all property access
}
```

### Rule 3: Follow Next.js App Router Conventions
**WHY:** Incorrect routing breaks the entire application
**HOW:** Use exact Next.js 16 patterns for API routes and pages

**API Route Pattern:**
```typescript
// app/api/endpoint/route.ts
import { NextRequest, NextResponse } from 'next/server'

export async function POST(request: NextRequest) {
  // Set runtime to nodejs for server-side features
  export const runtime = 'nodejs'

  try {
    const data = await request.json()
    // ... process
    return NextResponse.json({ success: true })
  } catch (error) {
    return NextResponse.json(
      { error: 'Error message' },
      { status: 500 }
    )
  }
}
```

**Dynamic Route Pattern:**
```typescript
// app/api/download/[fileId]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: { fileId: string } }
) {
  const { fileId } = params
  // ... process
}
```

### Rule 4: Implement Progressive Enhancement
**WHY:** Users should see progress, not blank screens
**HOW:** Use Server-Sent Events (SSE) for real-time updates

**SSE Pattern (already implemented):**
```typescript
// Create ReadableStream for SSE
const stream = new ReadableStream({
  async start(controller) {
    const send = (data: object) => {
      const message = `data: ${JSON.stringify(data)}\n\n`
      controller.enqueue(encoder.encode(message))
    }

    // Send progress updates
    send({ stage: 'parsing', progress: 20, message: 'Parsing resume...' })
    // ... process
    send({ stage: 'complete', progress: 100, download_url: '/api/download/...' })

    controller.close()
  }
})

return new Response(stream, {
  headers: {
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive'
  }
})
```

### Rule 5: Validate All User Input
**WHY:** Security and data integrity are critical
**HOW:** Use Zod schemas for validation, sanitize all text

**Validation Pattern:**
```typescript
// Define schema
const UploadSchema = z.object({
  file: z.instanceof(File),
  // ...
})

// Validate in API route
const result = UploadSchema.safeParse(data)
if (!result.success) {
  return NextResponse.json(
    { error: result.error.issues },
    { status: 400 }
  )
}
```

### Rule 6: Handle Async Operations Properly
**WHY:** Prevents race conditions and unhandled rejections
**HOW:** Use async/await with proper error handling

**Pattern:**
```typescript
export async function* processResume(
  fileId: string
): AsyncGenerator<Progress, ResumeData, unknown> {
  try {
    // Stage 1
    yield { stage: 'parsing', progress: 20, message: '...' }
    const data = await parseResume(fileId)

    // Stage 2
    yield { stage: 'enhancing', progress: 50, message: '...' }
    const enhanced = await enhanceWithAI(data)

    // Stage 3
    yield { stage: 'generating', progress: 80, message: '...' }
    const pdf = await generatePDF(enhanced)

    yield { stage: 'complete', progress: 100, message: 'Done!' }
    return enhanced
  } catch (error) {
    const message = error instanceof Error ? error.message : 'Unknown error'
    throw new Error(`Processing failed: ${message}`)
  }
}
```

### Rule 7: Optimize for Render.com Deployment
**WHY:** Free tier has limitations (512MB RAM, cold starts)
**HOW:** Minimize memory usage, optimize cold start time

**Optimization Checklist:**
- [ ] Use browser pooling for Puppeteer (reuse instances)
- [ ] Set `PUPPETEER_EXECUTABLE_PATH` correctly
- [ ] Implement graceful shutdown (SIGTERM handling)
- [ ] Add health check endpoint
- [ ] Cache compiled templates
- [ ] Clean up temp files regularly

### Rule 8: Implement Graceful Degradation
**WHY:** AI APIs can fail; parsing isn't perfect
**HOW:** Always have fallback strategies

**Pattern:**
```typescript
async function enhanceWithAI(data: ResumeData): Promise<ResumeData> {
  try {
    const enhanced = await geminiService.enhance(data)
    return enhanced
  } catch (error) {
    console.error('AI enhancement failed, using original:', error)
    // Return original data if AI fails
    return data
  }
}
```

### Rule 9: Never Block on Long Operations
**WHY:** Next.js API routes have timeouts
**HOW:** Use streaming responses or background jobs

**Current Implementation:** SSE streaming (good!)
**Future:** Consider background queue for >60s operations

### Rule 10: Write Tests for Critical Paths
**WHY:** Prevents regressions, enables confident refactoring
**HOW:** Use Jest for unit tests, Playwright for E2E

**Test Pattern:**
```typescript
// __tests__/parsers/pdf-parser.test.ts
describe('PDF Parser', () => {
  it('should extract text from standard PDF', async () => {
    const text = await parsePDF('path/to/sample.pdf')
    expect(text).toContain('experience')
  })

  it('should handle malformed PDF gracefully', async () => {
    await expect(parsePDF('path/to/bad.pdf')).resolves.toBeDefined()
  })
})
```

---

## 🎯 Project-Specific Instructions

### When Implementing the HTML-to-PDF Transition

**Phase 1: Install Dependencies**
```bash
pnpm add puppeteer handlebars zod @google/generative-ai
pnpm add -D @types/puppeteer @types/handlebars
```

**Phase 2: Create Zod Schemas**
1. Read the implementation plan: `HTML_TO_PDF_IMPLEMENTATION_PLAN.md`
2. Create `lib/schemas/resume-schema.ts` exactly as specified
3. Export all types for use across the codebase

**Phase 3: Enhance Parser**
1. Read current `lib/resume-processor.ts`
2. Create `lib/parsers/` directory with new enhanced parsers
3. Implement section detection with regex patterns
4. Test with diverse resume samples

**Phase 4: Gemini Integration**
1. Get API key from Google AI Studio
2. Create `lib/ai/gemini-service.ts`
3. Implement bullet enhancement with rate limiting
4. Add caching for common phrases

**Phase 5: HTML Templates**
1. Create `lib/templates/jakes-resume.html`
2. Style to match Jake's Resume LaTeX design
3. Set up Handlebars engine with helpers
4. Test rendering with sample data

**Phase 6: Puppeteer PDF Generation**
1. Create `lib/pdf/puppeteer-generator.ts`
2. Optimize Puppeteer args for memory
3. Implement browser pooling
4. Add timeout and error handling

**Phase 7: Integration**
1. Update `lib/resume-processor.ts` to use new pipeline
2. Test full flow: upload → parse → enhance → PDF
3. Verify SSE progress updates work
4. Test error scenarios

**Phase 8: Deployment**
1. Update `render.yaml` with Chromium dependencies
2. Set `GOOGLE_GEMINI_API_KEY` in Render dashboard
3. Test build locally: `pnpm build`
4. Deploy and monitor logs

---

## 🔧 Common Pitfalls & Solutions

### Pitfall 1: "Cannot find module" errors
**Cause:** Incorrect import paths in Next.js
**Solution:** Use `@/` alias for imports
```typescript
// ❌ BAD
import { parseResume } from '../../../lib/resume-processor'

// ✅ GOOD
import { parseResume } from '@/lib/resume-processor'
```

### Pitfall 2: API route returns HTML instead of JSON
**Cause:** Missing `export const runtime = 'nodejs'`
**Solution:** Add runtime export to server-side routes
```typescript
export const runtime = 'nodejs' // Forces server-side execution
export const dynamic = 'force-dynamic' // Disables caching
```

### Pitfall 3: Puppeteer fails to launch
**Cause:** Missing Chromium or incorrect path
**Solution:** Set executable path correctly
```typescript
const browser = await puppeteer.launch({
  headless: true,
  executablePath: process.env.PUPPETEER_EXECUTABLE_PATH ||
    '/opt/render/.cache/puppeteer/chrome/linux-*/chrome',
  args: ['--no-sandbox', '--disable-setuid-sandbox']
})
```

### Pitfall 4: Memory leaks in Puppeteer
**Cause:** Not closing browser instances
**Solution:** Use browser pooling and proper cleanup
```typescript
// Singleton browser instance
let browserInstance: Browser | null = null

async function getBrowser(): Promise<Browser> {
  if (!browserInstance || !browserInstance.isConnected()) {
    browserInstance = await puppeteer.launch(config)
  }
  return browserInstance
}

// Clean up on shutdown
process.on('SIGTERM', async () => {
  if (browserInstance) {
    await browserInstance.close()
  }
})
```

### Pitfall 5: Gemini API rate limit errors
**Cause:** Too many concurrent requests
**Solution:** Implement rate limiting with delays
```typescript
async function enhanceAllBullets(bullets: string[]): Promise<string[]> {
  const enhanced = []

  for (const bullet of bullets) {
    const result = await enhanceBullet(bullet)
    enhanced.push(result)

    // Rate limiting: 1 request per second
    await new Promise(resolve => setTimeout(resolve, 1000))
  }

  return enhanced
}
```

---

## 📊 Performance Targets

| Metric | Target | Current | Status |
|--------|--------|---------|--------|
| File Upload | <2s | ✓ | ✅ |
| Parsing (PDF) | <5s | ✓ | ✅ |
| AI Enhancement | <30s | TBD | ⏳ |
| PDF Generation | <10s | TBD | ⏳ |
| Total Time | <60s | TBD | ⏳ |
| Memory Usage | <400MB | TBD | ⏳ |

---

## 🧪 Testing Strategy

### Unit Tests (Priority: HIGH)
- [ ] Parser functions (PDF, DOCX, TXT)
- [ ] Section detection regex patterns
- [ ] Zod schema validation
- [ ] Template rendering
- [ ] Gemini API service (mocked)

### Integration Tests (Priority: MEDIUM)
- [ ] Upload → Parse pipeline
- [ ] Parse → Enhance pipeline
- [ ] Enhance → PDF pipeline
- [ ] SSE progress updates

### End-to-End Tests (Priority: HIGH)
- [ ] Complete flow with sample resumes
- [ ] Error scenarios (invalid files, API failures)
- [ ] Download and verify PDF quality

### Test Command
```bash
# Run all tests
pnpm test

# Run specific test file
pnpm test lib/parsers/pdf-parser.test.ts

# Run with coverage
pnpm test --coverage
```

---

## 🚨 Critical Errors to Avoid

1. **NEVER commit sensitive data**
   - API keys go in `.env.local` (gitignored)
   - Never hardcode secrets in source code

2. **NEVER skip input validation**
   - All user uploads must be validated
   - Check file type, size, magic numbers
   - Sanitize all text before HTML rendering

3. **NEVER ignore error handling**
   - Wrap all async operations in try-catch
   - Log errors with context
   - Return user-friendly error messages

4. **NEVER mutate function arguments**
   - Use spread operator or Object.assign
   - Return new objects instead

5. **NEVER use `any` type**
   - Define proper TypeScript interfaces
   - Use `unknown` if type is truly unknown

---

## 💡 When AI Agent Should Ask Questions

### Ask Before Implementation If:
- Multiple approaches are valid (e.g., "Should I use Playwright or Puppeteer?")
- Requirements are ambiguous (e.g., "Should I enhance all bullets or just poor ones?")
- Architecture changes needed (e.g., "Should I refactor file-storage.ts?")
- Trade-offs exist (e.g., "Speed vs. quality for AI enhancement?")

### Ask After Implementation If:
- Tests fail unexpectedly
- Performance is worse than targets
- New errors appear in production
- User feedback indicates issues

### Never Ask About:
- Syntax questions (look up documentation)
- Standard patterns (follow existing code)
- Simple bugs (debug and fix)
- Obvious improvements (just do them)

---

## 🎓 Learning Resources

**Next.js 16 App Router:**
- https://nextjs.org/docs/app

**Puppeteer:**
- https://pptr.dev/

**Gemini API:**
- https://ai.google.dev/docs

**Zod Validation:**
- https://zod.dev/

**Jake's Resume (inspiration):**
- https://github.com/jakegut/resume

---

## 🔄 Collaboration Protocol (3-Person Team)

### Developer Roles
1. **Primary Developer (You)** - Active development, main branch management
2. **Contributor 1** - Feature branches, code review
3. **Contributor 2** - Testing, documentation, deployment

### Branch Naming Convention (NEW)
```
main               - Production-ready code
feature/*          - New features (e.g., feature/gemini-integration)
bugfix/*           - Bug fixes (e.g., bugfix/parsing-error)
experiment/*       - Experimental work (e.g., experiment/latex-alternative)
release/*          - Release preparation (e.g., release/v1.0)
```

### Workflow
1. Create feature branch from `main`
2. Implement feature with tests
3. Submit PR with description
4. Code review by another team member
5. Merge to `main` after approval
6. Deploy to staging → production

### Commit Message Format
```
<type>(<scope>): <subject>

<body>

<footer>

Types: feat, fix, docs, style, refactor, test, chore
Scopes: parser, ai, pdf, ui, api, deploy

Example:
feat(ai): integrate Gemini API for bullet enhancement

- Add GeminiService class with rate limiting
- Implement bullet enhancement with retry logic
- Add caching for common phrases
- Test with 20 sample resumes

Closes #42
```

---

## 📈 Success Metrics

### Technical Quality
- [ ] 80%+ code coverage with tests
- [ ] Zero critical security vulnerabilities
- [ ] <5% error rate in production
- [ ] <15s average processing time
- [ ] 99%+ uptime

### User Experience
- [ ] 70%+ completion rate (upload → download)
- [ ] 4/5 average satisfaction score
- [ ] <3% abandon rate during processing
- [ ] Clear error messages for all failures

### Business Viability
- [ ] 100 users in first month
- [ ] <$50/month infrastructure costs
- [ ] 10+ positive testimonials
- [ ] 60%+ would recommend to others

---

## 🎯 Current Sprint Goals (Branch: bharath-013)

### Week 1: Foundation
- [x] Create implementation plan ✅
- [x] Create risk analysis ✅
- [ ] Install dependencies
- [ ] Set up Zod schemas
- [ ] Test Puppeteer locally

### Week 2: Core Features
- [ ] Enhance resume parser
- [ ] Integrate Gemini API
- [ ] Test AI enhancement quality
- [ ] Implement caching

### Week 3: PDF Generation
- [ ] Create HTML template
- [ ] Set up Puppeteer generator
- [ ] Test PDF output quality
- [ ] Optimize memory usage

### Week 4: Launch
- [ ] Write comprehensive tests
- [ ] Deploy to production
- [ ] Monitor performance
- [ ] Gather user feedback

---

## 🚀 Quick Start Commands

```bash
# Install dependencies
pnpm install

# Run development server
pnpm dev

# Build for production
pnpm build

# Start production server
pnpm start

# Run tests
pnpm test

# Type checking
pnpm type-check

# Lint code
pnpm lint
```

---

## 📞 Getting Help

### When Stuck:
1. Check existing code patterns in the codebase
2. Review implementation plan documents
3. Search Next.js/Puppeteer documentation
4. Ask specific questions with context
5. Share error messages and stack traces

### Escalation Path:
1. Try to debug yourself (15 min)
2. Ask AI assistant for help (30 min)
3. Discuss with team member (1 hour)
4. Post in community forum (4 hours)

---

**Document Version:** 2.0
**Last Updated:** November 10, 2025
**Branch:** bharath-013
**Status:** Active Development
**Next Review:** After Phase 1 completion

---

## 🎯 AI Agent Final Checklist

Before responding to any request, ensure you:
- [ ] Read relevant existing files first
- [ ] Understand current architecture (Next.js, not FastAPI)
- [ ] Follow TypeScript patterns strictly
- [ ] Implement proper error handling
- [ ] Add progress updates for long operations
- [ ] Write or update tests
- [ ] Check for security vulnerabilities
- [ ] Optimize for performance
- [ ] Document complex logic
- [ ] Follow commit message format

**REMEMBER:** This is a production application that will handle real users' sensitive resume data. Code quality, security, and reliability are paramount.

---
> Source: [7ochan/KairosCV](https://github.com/7ochan/KairosCV) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
