## coverme

> This playbook guides AI coding agents working on **cover.me**, an AI tool that generates tailored cover letters from a user’s background, resume, and job description. It keeps patterns consistent, enforces security and accessibility, and aligns the product toward high conversion and quality.

## Introduction

This playbook guides AI coding agents working on **cover.me**, an AI tool that generates tailored cover letters from a user’s background, resume, and job description. It keeps patterns consistent, enforces security and accessibility, and aligns the product toward high conversion and quality.

---

## Quick Reference Card

```
PROJECT: cover.me — AI cover letter writer
STACK: SvelteKit + TypeScript + Tailwind + shadcn-svelte
DEPLOYMENT: Cloudflare Pages
AUTH: Supabase Auth
DATABASE: Supabase (user data) + Cloudflare D1 (cache of model outputs)
STORAGE: Cloudflare R2 (resume uploads, exports)
PACKAGE MANAGER: Bun
```

---

## 📋 CRITICAL: Development Process Requirements

**BEFORE ANY IMPLEMENTATION, YOU MUST:**

1. **Read the canonical documentation in this exact order:**
  
  1. devdocs/prd.md
  2. devdocs/planning/development-roadmap.md
  3. AGENTS.md

2. **Follow the Stage Implementation Protocol:**
   - Review current stage requirements from `devdocs/planning/development-roadmap.md`
   - Create implementation plan with testable success criteria
   - Implement ONLY current stage features (no premature implementation)
   - Validate against documented success criteria before claiming completion

3. **Stage Completion Requirements:**
   - All stage requirements from roadmap must be implemented
   - Success criteria must be met and testable
   - Code must follow AGENTS.md principles
   - Documentation must be updated

---

## Core Principles

1) **Mobile-first**: design for small screens first, then add breakpoints.
2) **Accessibility**: every interactive element must be keyboard and screen-reader friendly (ARIA labels, focus states).
3) **Type safety**: strict TypeScript; no `any`. Model all payloads (job, resume, letter).
4) **Server-side validation**: validate/sanitize job text, resumes, and preferences on the server.
5) **Privacy & PII**: never log raw resumes or job descriptions; strip PII from telemetry.
6) **ATS-friendly output**: clean text structure, single column, no decorative images in exports.
7) **Conversion mindset**: propose A/B tests when UX changes could affect signup, generation, or upgrade funnels.
8) **Documentation First**: Review product requirements in `devdocs/prd.md` and reference feature-specific documents in `devdocs` before planning and implementing any new feature.

### Mobile-First Example
```svelte
<!-- ✅ Correct -->
<div class="flex flex-col gap-4 md:flex-row md:gap-6 lg:gap-8">
```

### Accessible Button Example
```svelte
<Button on:click={generate} aria-label="Generate cover letter" class="focus:ring-2 focus:ring-primary-500">
  Generate
</Button>
```

### Typed Payload Example
```typescript
interface CoverLetterPayload {
  userId: string;
  jobTitle: string;
  company: string;
  jobDescription: string;
  resumeUrl?: string;
  tone: 'concise' | 'enthusiastic' | 'formal';
  language?: string;
}
```

---

## Project Structure Guide

- Components: `PascalCase.svelte` (e.g., `LetterPreview.svelte`, `ResumeUpload.svelte`)
- Routes: lowercase `+page.svelte`, `+page.server.ts`, `+layout.svelte`
- Utils: `kebab-case.ts` (e.g., `job-parser.ts`, `resume-reader.ts`)
- Types: `PascalCase.ts` (e.g., `CoverLetter.ts`, `UserProfile.ts`)
- API routes: `+server.ts`

### Import Order
```typescript
// 1. External deps
import { z } from 'zod';
// 2. SvelteKit
import { error, redirect } from '@sveltejs/kit';
// 3. Internal absolute
import { createCoverLetter } from '$lib/server/ai/letters';
// 4. Relative
import Form from './Form.svelte';
// 5. Types-only
import type { PageServerLoad } from './$types';
```

---

## Component Patterns

### Standard Svelte Component
```svelte
<script lang="ts">
  import { Button } from '$lib/components/ui/button';
  import type { CoverLetterDraft } from '$lib/types/CoverLetter';

  export let draft: CoverLetterDraft | null = null;
  let loading = false;

  async function regenerate() {
    loading = true;
    await triggerRegeneration();
    loading = false;
  }
</script>

<section class="space-y-4">
  <header class="flex items-center justify-between gap-3">
    <h2 class="text-xl font-semibold text-neutral-900">Preview</h2>
    <Button size="sm" on:click={regenerate} disabled={loading} aria-label="Regenerate cover letter">
      {loading ? 'Working…' : 'Regenerate'}
    </Button>
  </header>

  {#if draft}
    <article class="prose max-w-none bg-white p-4 rounded-lg border border-neutral-200">
      {@html draft.html}
    </article>
  {:else}
    <p class="text-sm text-neutral-600">No draft yet. Add a job description to start.</p>
  {/if}
</section>
```

### Protected Layout Example
```typescript
// src/routes/app/+layout.server.ts
import { redirect } from '@sveltejs/kit';
import type { LayoutServerLoad } from './$types';

export const load: LayoutServerLoad = async ({ locals }) => {
  const { user } = await locals.safeGetSession();
  if (!user) throw redirect(302, '/auth/login');
  return { user };
};
```

### Dashboard Loader Example
```typescript
// src/routes/app/+page.server.ts
import { getLatestLetters } from '$lib/server/db/letters';
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ locals }) => {
  const { user } = await locals.safeGetSession();
  if (!user) throw redirect(302, '/auth/login');
  const letters = await getLatestLetters(user.id);
  return { letters, user };
};
```

---

## Data & Database Patterns

### Supabase Clients
```typescript
// Server-side
export const supabaseAdmin = createClient(
  process.env.PUBLIC_SUPABASE_URL!,
  process.env.SUPABASE_SERVICE_ROLE_KEY!
);

// Client-side
export const supabase = createClient(
  process.env.PUBLIC_SUPABASE_URL!,
  process.env.PUBLIC_SUPABASE_ANON_KEY!
);
```

### Key Tables
- `profiles`: name, headline, links, location, skills
- `resumes`: file metadata + signed R2 URL, mime, size
- `letters`: title, company, job_title, tone, body, html, version, created_at
- `jobs`: parsed job postings (title, company, location, description, source_url)
- `experiments`: A/B assignments and metrics

### Query Pattern
```typescript
const { data, error } = await supabase
  .from('letters')
  .select('id, title, company, job_title, created_at')
  .eq('user_id', userId)
  .order('created_at', { ascending: false })
  .limit(20);
if (error) throw error;
```
Always filter by `user_id`. Keep RLS-friendly queries; never fetch-all-and-filter in memory.

### Caching
- Use D1 to cache AI outputs keyed by normalized job text hash + resume hash + tone.
- TTL the cache; invalidate when resume/profile changes.

---

## Authentication Patterns

- Protect app routes with `+layout.server.ts` redirects when no session.
- Use `locals.safeGetSession()`; never trust client state.
- On logout, clear client stores and invalidate SvelteKit data.

Client store:
```typescript
export const user = writable<User | null>(null);
export const isAuthenticated = derived(user, ($user) => !!$user);
```

---

## Content & Template Management

- Store template metadata in Supabase (`templates`: name, tone, language, structure, enabled).
- Keep template bodies in R2 as markdown with placeholders (`{{candidateName}}`, `{{jobTitle}}`).
- Render markdown server-side, sanitize HTML, and keep exports ATS-friendly (single column, no images).

```typescript
import { marked } from 'marked';
import DOMPurify from 'dompurify';
import { JSDOM } from 'jsdom';

const purify = DOMPurify(new JSDOM('').window);
export async function renderTemplate(md: string) {
  const html = await marked.parse(md);
  return purify.sanitize(html);
}
```

---

## Styling Patterns

- Use Tailwind; avoid custom CSS unless necessary.
- Build mobile-first grids and flex layouts; expand progressively.
- Use design tokens (`primary`, `accent`, `muted`); avoid arbitrary hex codes.

---

## API Route Patterns

### Standard Request Flow
```typescript
import { json, error } from '@sveltejs/kit';
import { z } from 'zod';
import type { RequestHandler } from './$types';

const schema = z.object({
  jobTitle: z.string().min(2),
  company: z.string().min(1),
  jobDescription: z.string().min(30),
  tone: z.enum(['concise', 'enthusiastic', 'formal']).default('concise'),
  resumeUrl: z.string().url().optional(),
  language: z.string().min(2).max(10).optional()
});

export const POST: RequestHandler = async ({ request, locals }) => {
  const { user } = await locals.safeGetSession();
  if (!user) throw error(401, 'Unauthorized');

  const payload = schema.parse(await request.json());
  const letter = await createCoverLetter({ ...payload, userId: user.id });
  return json({ success: true, data: letter });
};
```

### Error Helper
```typescript
export function apiError(status: number, message: string, details?: any) {
  return new Response(
    JSON.stringify({ success: false, error: { message, details } }),
    { status, headers: { 'Content-Type': 'application/json' } }
  );
}
```

---

## Form Handling Patterns

### Client Validation (job intake)
```svelte
<script lang="ts">
  import { z } from 'zod';
  const schema = z.object({
    jobTitle: z.string().min(2, 'Add a job title'),
    company: z.string().min(1, 'Add a company'),
    jobDescription: z.string().min(30, 'Paste the job description'),
    tone: z.enum(['concise', 'enthusiastic', 'formal'])
  });

  let jobTitle = '';
  let company = '';
  let jobDescription = '';
  let tone: 'concise' | 'enthusiastic' | 'formal' = 'concise';
  let errors: Record<string, string> = {};

  function validate() {
    try {
      schema.parse({ jobTitle, company, jobDescription, tone });
      errors = {};
      return true;
    } catch (err) {
      if (err instanceof z.ZodError) {
        errors = Object.fromEntries(
          err.errors.map((e) => [e.path[0] as string, e.message])
        );
      }
      return false;
    }
  }
</script>
```

### Server Actions (resume upload)
```typescript
export const actions: Actions = {
  uploadResume: async ({ request, locals }) => {
    const { user } = await locals.safeGetSession();
    if (!user) return fail(401, { message: 'Unauthorized' });

    const form = await request.formData();
    const file = form.get('resume') as File | null;
    if (!file) return fail(400, { message: 'File required' });

    const errorText = validateResume(file);
    if (errorText) return fail(400, { message: errorText });

    const url = await saveResumeToR2(user.id, file);
    return { success: true, url };
  }
};
```

---

## Testing Patterns

- Use Vitest + Testing Library for components.
- Mock AI calls, Supabase, and storage; never hit real services in tests.

```typescript
import { render, screen } from '@testing-library/svelte';
import LetterPreview from './LetterPreview.svelte';

test('renders generated letter content', () => {
  render(LetterPreview, { draft: { html: '<p>Hello</p>' } });
  expect(screen.getByText('Hello')).toBeInTheDocument();
});
```

---

## Performance & Cost

- Cache AI outputs in D1; normalize keys (job hash + resume hash + tone).
- Stream AI responses for faster perceived speed.
- Lazy-load heavy editors/PDF modules.
- Optimize images on marketing pages (WebP, `loading="lazy"`).

---

## Error Handling

Centralized user-friendly messages:
```typescript
export const ERROR_MESSAGES = {
  AUTH_REQUIRED: 'Please sign in to continue.',
  GENERATION_FAILED: 'We could not generate your letter. Please retry.',
  UPLOAD_FAILED: 'Upload failed. Use PDF or DOCX up to 5MB.',
  JOB_TOO_SHORT: 'Job description is too short. Paste the full text.',
  SERVER_ERROR: 'Something went wrong. Try again shortly.'
} as const;
```

- Show errors as inline hints near fields; use toasts for global failures.
- Do not expose stack traces or raw request bodies.

---

## Common Pitfalls

- ❌ Using `localStorage`/`window` during SSR without `browser` guard.
- ❌ Fetching protected data in client components instead of `+page.server.ts`.
- ❌ Logging PII (job text, resume). Log hashes/ids only.
- ❌ Hardcoding URLs; use `$app/paths` base.

---

## Stage-Specific Guidance

### 1) Marketing Site
- Goal: drive signups; optimize LCP and SEO (meta tags, OG/Twitter).
- Clear CTA ("Generate my cover letter"), social proof, and pricing clarity.
- Track `page_view`, `cta_click`, `signup_start`.

### 2) Onboarding
- Collect name, role, experience, top skills, target roles.
- Offer resume upload early; validate type/size; show saving states.
- Provide skip options to reduce drop-off.

### 3) Drafting Flow
- Inputs: job description paste or URL, role/company, tone, language, resume selection.
- Show streaming preview and version history; allow inline edits with autosave.
- Quick toggles: tone, length, keywords to include.
- Keep output ATS-friendly (single column, semantic headings, no images).

### 4) Exports & Delivery
- Export to PDF/DOCX and copy-to-clipboard. Safe filenames: `{company}-{role}-cover-letter.pdf`.
- Email sharing: require verified email; rate-limit sends.

### 5) Analytics & Experiments
- Events: `letter_generated`, `letter_regenerated`, `export_pdf`, `copy_clipboard`, `resume_uploaded`, `job_parsed`.
- Propose A/B tests for CTA copy, default tone, onboarding flow, template ordering.
- Respect consent before loading marketing pixels.

---

## Debugging Tips

- Dev logging: `VITE_LOG_LEVEL=debug`.
- Log hashes, not raw text, for resumes/jobs.
- For Supabase permission errors, confirm `user_id` filters and RLS policies.
- For AI issues, log prompt ids and model response ids (no PII) into D1.

---

## Quick Command Reference

```bash
bun run dev       # Start dev server
bun run build     # Production build
bun run preview   # Preview build
bun run check     # Type + Svelte check
bun run lint      # ESLint
bun run test      # Tests
```

---

## Checklist Before Commit

- [ ] `bun run check` passes
- [ ] No console errors
- [ ] Mobile-first, responsive verified
- [ ] ARIA labels and keyboard nav verified
- [ ] Loading + error states covered
- [ ] No PII in logs; inputs validated server-side
- [ ] Queries filtered by `user_id`
- [ ] ATS-friendly output preserved
- [ ] Formatted with Prettier

---

## Example: End-to-End Letter Generation

### 1) Save to DB
```typescript
export async function saveLetter(input: CoverLetterPayload, body: string, html: string) {
  const { data, error } = await supabaseAdmin
    .from('letters')
    .insert({
      user_id: input.userId,
      job_title: input.jobTitle,
      company: input.company,
      tone: input.tone,
      body,
      html
    })
    .select()
    .single();
  if (error) throw error;
  return data;
}
```

### 2) API Route
```typescript
// src/routes/api/letters/+server.ts
export const POST: RequestHandler = async ({ request, locals }) => {
  const { user } = await locals.safeGetSession();
  if (!user) throw error(401, 'Unauthorized');

  const payload = schema.parse(await request.json());
  const cached = await getCachedLetter(payload);
  if (cached) return json({ success: true, data: cached, cached: true });

  const ai = await createCoverLetter({ ...payload, userId: user.id });
  const saved = await saveLetter({ ...payload, userId: user.id }, ai.body, ai.html);
  await cacheLetter(payload, saved);

  return json({ success: true, data: saved, cached: false });
};
```

### 3) Client UI
```svelte
<script lang="ts">
  import { Button } from '$lib/components/ui/button';
  import { toast } from '$lib/stores/toast';
  import { invalidateAll } from '$app/navigation';
  let loading = false;

  async function generate() {
    loading = true;
    try {
      const res = await fetch('/api/letters', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(formValues)
      });
      if (!res.ok) throw new Error('Generation failed');
      await invalidateAll();
      toast.success('Cover letter generated');
    } catch (e) {
      toast.error('Could not generate. Please retry.');
    } finally {
      loading = false;
    }
  }
</script>

<Button on:click={generate} disabled={loading} aria-label="Generate cover letter">
  {loading ? 'Generating…' : 'Generate'}
</Button>
```

---

## Advanced Patterns

- **Job URL ingestion**: fetch server-side, extract main content, strip scripts/ads before prompting.
- **Resume validation**: PDF/DOCX only, max 5MB; reject images. Extract text server-side.
- **Debounced parsing**: debounce job-text parsing to avoid extra requests; show inline validation when text is short.
- **Versioning**: store letter versions; allow quick revert.
- **Localization**: tone + language settings; default to browser language for marketing pages.
- **Rate limiting**: per-user generation limits; surface remaining quota in UI.

---

## Types

```typescript
export interface CoverLetterDraft {
  id: string;
  userId: string;
  title: string;
  company: string;
  jobTitle: string;
  tone: 'concise' | 'enthusiastic' | 'formal';
  body: string;
  html: string;
  createdAt: string;
}

export interface UserProfile {
  id: string;
  fullName: string;
  headline?: string;
  location?: string;
  links?: string[];
  skills?: string[];
}
```

---

## Analytics & Consent

- Initialize analytics only after consent (`consent` store).
- Track conversion-critical events: `signup_start`, `signup_complete`, `letter_generated`, `export_pdf`, `copy_clipboard`, `upgrade_clicked`, `checkout_success`.
- Do not send PII; prefer anon ids or hashed ids.

---

## Privacy Best Practices

- Check consent before loading GA/Meta/TikTok.
- Strip PII from logs and telemetry; log ids/hashes only.
- Respect Do-Not-Track where feasible.
- Require explicit user action for exports/sharing; never auto-email.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markplusgood) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
