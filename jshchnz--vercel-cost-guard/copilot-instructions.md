## vercel-cost-guard

> >


# Vercel Cost Guard

Audit a Next.js/Vercel project for patterns that cause high hosting costs — bandwidth, compute, and invocation charges.

Vercel's pay-per-use pricing means code patterns directly affect your hosting bill. A missing `preload="none"`, an unscoped middleware, or a single `cookies()` call can silently multiply costs. This audit catches these patterns before they show up on an invoice.

## How to Run the Audit

Perform the following 22 checks in order using your built-in tools (Glob, Grep, Read). After completing all checks, compile findings into the report format at the bottom.

**Report generation rules:**
- For every CRITICAL finding, include a copy-paste-ready fix (shell command, code snippet, or config change) drawn from reference/optimization-guide.md. Substitute the project's actual file paths into the commands.
- Estimate dollar impact where possible. Use the cost formulas from reference/vercel-pricing.md. Assume 50K monthly visitors as a baseline unless the project's traffic is known. State your assumption.
- Track positive findings ("GOOD" patterns from Checks 2, 8, and 11) — these are used in the report when the project has no critical or warning issues.

**Important:** Search `.ts` and `.js` files in addition to `.tsx` and `.jsx` — blog content and HTML strings are often defined in TypeScript template literals, not just JSX files.

---

## Check 1: Public Directory Assets

Use Glob to find all files in `public/`. For each media file, note its size.

**CRITICAL — Files > 10 MB:**
Any file over 10 MB in `public/` will cause significant bandwidth costs under traffic. Each visitor downloads it fully.
- Fix: Compress the file, convert to a smaller format, or move to an external CDN (Cloudflare R2, AWS S3).

**WARNING — Files 5-10 MB:**
Large but not extreme. Should be compressed before any launch or promotion.

**CRITICAL — Uncompressed formats:**
Flag any files with these extensions — they have compressed alternatives that are 2-20x smaller:
- `.wav`, `.flac` → convert to `.mp3` or `.m4a`
- `.mov`, `.avi` → convert to `.mp4` (H.264)
- `.bmp`, `.tiff`, `.tif`, `.raw`, `.psd` → convert to `.webp` or `.avif`

Calculate the total size of all media files in `public/` and list the top 5 largest files.

For detailed compression commands, see [reference/optimization-guide.md](reference/optimization-guide.md).

---

## Check 2: Media Preload Attributes

Use Grep to search all `.tsx`, `.jsx`, `.ts`, `.js`, and `.html` files (excluding `node_modules/` and `.next/`) for `<audio` and `<video` elements.

**CRITICAL — `preload="auto"`:**
Pattern: `<(audio|video)[^>]*preload\s*=\s*["']auto["']`
The browser downloads the ENTIRE file on page load. This is the most expensive setting.
- Fix: Change to `preload="none"`.

**WARNING — `preload="metadata"`:**
Pattern: `<(audio|video)[^>]*preload\s*=\s*["']metadata["']`
The browser downloads file headers to read duration/codec info. For large files, this can be several MB per element.
- Fix: Change to `preload="none"` for cost safety.

**CRITICAL — Missing preload attribute:**
Find `<audio` and `<video` tags that do NOT have a `preload` attribute. Many browsers default to `preload="auto"` when the attribute is missing, which downloads the entire file.
- Fix: Add `preload="none"` explicitly.

**GOOD — `preload="none"`:**
Note any elements correctly using `preload="none"` as positive findings.

---

## Check 3: Next.js Config

Use Glob to find `next.config.ts`, `next.config.mjs`, or `next.config.js`. Read the file.

**WARNING — Missing `images.minimumCacheTTL` (Next.js 14/15):**
Without this setting, optimized images expire quickly and must be re-optimized on the next request (costs compute + bandwidth). Next.js 16+ increased the default from 60s to 14,400s (4 hours), so this is less critical on v16+ but still important on v14/v15 where the default is only 60s.
- Fix: Add `minimumCacheTTL: 2592000` (30 days) to the `images` config.

**WARNING — Low cache TTL (< 86400 seconds / 1 day):**
Even with the setting present, a value under 1 day causes frequent re-optimization.
- Fix: Increase to at least `2592000` (30 days) for static images.

**INFO — Missing AVIF format:**
If `images.formats` does not include `'image/avif'`, images are 20-50% larger than they could be.
- Fix: Add `formats: ['image/avif', 'image/webp']` to images config.

**WARNING — Missing custom Cache-Control headers:**
Check if the config has a `headers()` function with `Cache-Control` for static assets. Without custom headers, static assets may not be cached effectively at the CDN edge.
- Fix: Add long-lived cache headers for static file extensions. See [reference/optimization-guide.md](reference/optimization-guide.md).

**WARNING — Overly broad `remotePatterns`:**
Check the `images.remotePatterns` array. Flag if any entry uses `hostname: '**'` or a very permissive wildcard (e.g., `hostname: '*.com'`). Broad remote patterns allow any external source to trigger image optimization (billed per transformation — see [reference/vercel-pricing.md](reference/vercel-pricing.md)). Wildcard hostnames also create SSRF/DoS risk against the image optimizer (CVE-2025-59471).
- Fix: Restrict `remotePatterns` to specific, known hostnames your app actually uses.

---

## Check 4: Image Optimization

**WARNING — Raw `<img>` tags instead of next/image:**
Use Grep to find `<img` tags in `app/`, `components/`, and `pages/` directories (excluding `node_modules/`). The Next.js `<Image>` component provides automatic optimization, lazy loading, and modern format conversion.
- Fix: Replace `<img>` with `<Image>` from `next/image`.

**WARNING — GIF files with Image component:**
Grep for Image component usage with `.gif` sources. Next.js does NOT optimize GIFs — they pass through at full size.
- Fix: Convert GIFs to `.mp4`/`.webm` video, or use a static image.

**INFO — Missing `quality` prop on Image components:**
Grep for `<Image` usage without a `quality` prop. The default is 75, which is usually fine, but explicitly setting it to 75 or lower can reduce sizes further.

**WARNING — Large PNGs in public/:**
Use Glob to find `.png` files > 500 KB in `public/`. PNGs are lossless and often 2-4x larger than WebP/AVIF.
- Fix: Convert to WebP (25-34% smaller) or AVIF (up to 50% smaller).

---

## Check 5: API Route Cache Headers

Find API route files in `app/api/` or `pages/api/`.

**WARNING — GET routes missing Cache-Control:**
Grep for files that export a `GET` handler but don't contain `Cache-Control`. Every uncached GET request invokes the serverless function, costing compute time and bandwidth.
- Fix: Add `Cache-Control: public, s-maxage=60, stale-while-revalidate=300` (adjust TTL to your needs).

---

## Check 6: Anti-Patterns

**WARNING — Server components calling own API routes:**
Grep for `fetch('/api/` or `fetch("/api/` in files that do NOT contain `'use client'` or `"use client"`. Server components calling their own API routes cause double invocations (the server component invocation + the API route invocation).
- Fix: Import the API logic directly instead of fetching via HTTP.

**INFO — Excessive Link prefetching:**
Grep for `<Link` with `prefetch={true}` or `prefetch=\{true\}`. In Next.js 15+, `prefetch={true}` is deprecated — the default behavior (`"auto"`) already prefetches visible links in the viewport. Explicit `prefetch={true}` is redundant and may increase bandwidth by fully prefetching dynamic routes.
- Fix: Remove `prefetch={true}` (use the default). Use `prefetch={false}` for infrequently visited pages.

**INFO — Client-side fetching without caching library:**
Check if `package.json` includes `swr` or `@tanstack/react-query`. If neither is present and client components use `fetch()`, data may be re-fetched unnecessarily on every render.
- Fix: Add SWR or React Query for automatic caching and request deduplication.

---

## Check 7: Vercel Spend Management

This check is **always included** in the report, regardless of other findings.

Vercel's Spend Management sets an on-demand budget with alerts at 50%, 75%, and 100%. It is enabled by default on Pro plans. Check the current default limit at the [Spend Management docs](https://vercel.com/docs/spend-management) — do not assume a specific dollar amount, as defaults change.

Remind the user to:
1. Go to Vercel Dashboard → Settings → Billing → Spend Management
2. Review the current limit and lower it if appropriate (e.g., $20-50 for hobby projects)
3. Consider enabling **"Pause production deployment"** at 100% for a hard spending cap
4. Configure alert notifications for usage spikes

For pricing details, see [reference/vercel-pricing.md](reference/vercel-pricing.md).

---

## Check 8: Middleware Scope

Use Glob to find `middleware.ts`, `middleware.js`, `proxy.ts`, or `proxy.js` at the project root (or `src/`). Next.js 16 renamed `middleware.ts` to `proxy.ts` — check for both.

If no middleware/proxy file exists, skip this check.

If one exists, Read the file and check for a `matcher` config.

**WARNING — No `matcher` config:**
Without a `matcher`, middleware runs on **every request** — including `_next/static`, `_next/image`, and other asset requests. This can multiply edge invocations by 10x or more.
- Fix: Add a `config.matcher` that excludes static assets. See [reference/optimization-guide.md](reference/optimization-guide.md).

**WARNING — `matcher` does not exclude `_next/static` or `_next/image`:**
Even with a matcher, if it doesn't exclude Next.js internal asset paths, middleware still runs on those requests unnecessarily.
- Fix: Use a negative lookahead pattern: `'/((?!_next/static|_next/image|favicon.ico).*)'`

**GOOD — Properly scoped middleware:**
Note if middleware has a well-scoped matcher that excludes static assets.

---

## Check 9: Accidental Dynamic Rendering

Use Grep to search `page.tsx` and `page.ts` files in the `app/` directory for patterns that force dynamic rendering.

> **Note:** In Next.js 15+, `cookies()`, `headers()`, and `searchParams` are **async** and must be `await`ed. Grep patterns should catch both sync and async usage.

**WARNING — `cookies()` usage from `next/headers`:**
Pattern: `(await\s+)?cookies\(\)` in files that import from `next/headers`
A single `cookies()` call forces the entire page to render on every request instead of being statically cached.
- Fix: Move cookie-dependent logic to a client component, or handle in middleware.

**WARNING — `headers()` usage from `next/headers`:**
Pattern: `(await\s+)?headers\(\)` in files that import from `next/headers`
Same dynamic opt-in as `cookies()`.
- Fix: Extract needed header values in middleware and pass via rewrites/cookies.

**WARNING — `searchParams` in page component signature:**
Pattern: `searchParams` in the page component's props type or destructuring (in Next.js 15+, this is a `Promise` that must be `await`ed)
Accessing `searchParams` in a server component forces dynamic rendering for every unique URL.
- Fix: Use `generateStaticParams` for known parameter values, or move filtering logic to a client component.

**INFO — `connection()` or `unstable_noStore()` usage:**
Also check for `connection()` from `next/server` (Next.js 15+) or the legacy `unstable_noStore()` — both explicitly opt out of static rendering.
- Fix: Use `revalidate` instead for periodic freshness where possible.

For detailed before/after examples, see [reference/optimization-guide.md](reference/optimization-guide.md).

---

## Check 10: Sequential Async in Serverless

Use Grep to search API route files (`app/api/**/route.ts`, `app/api/**/route.js`) and server action files for functions with 3 or more sequential `await` statements.

**INFO — 3+ sequential `await` in a single function:**
Each sequential `await` keeps the serverless function running (and billing) while waiting on I/O. Three sequential 200ms operations take 600ms; parallelized they take 200ms — a 3x reduction in billed duration.
- Fix: Wrap independent operations in `Promise.all()`. See [reference/optimization-guide.md](reference/optimization-guide.md).

Note: Only flag `await` statements that appear to be independent (not where one result feeds into the next). Use judgment — `const user = await getUser(); const posts = await getPostsByUser(user.id);` is necessarily sequential.

**INFO — Missing `maxDuration` route segment config:**
If no `export const maxDuration = N` is set on API routes or pages with Server Actions, functions run up to the platform default (which can be long on Pro plans). Adding `maxDuration` caps execution time and prevents runaway billing from slow upstream services.
- Fix: Add `export const maxDuration = 10` (or appropriate value) to API routes and pages with Server Actions.

---

## Check 11: Missing ISR / generateStaticParams

Use Glob to find dynamic route directories in `app/` — directories containing `[param]`, `[...param]`, or `[[...param]]` segments.

For each dynamic route, Read the `page.tsx`/`page.ts` file and check for:

**WARNING — Dynamic route without `generateStaticParams` or `revalidate`:**
If a dynamic route page has neither `export async function generateStaticParams` nor `export const revalidate`, every visit triggers a serverless function invocation.
- Fix: Add `generateStaticParams` to pre-render known pages at build time, or add `export const revalidate = 3600` for ISR.

**GOOD — Has `generateStaticParams`:**
Note pages that correctly pre-render dynamic routes.

**GOOD — Has `revalidate` export:**
Note pages using ISR with a reasonable revalidation interval.

---

## Check 12: AI Agent & Streaming Response Patterns

Use Grep to search API routes (`app/api/**/route.ts`, `app/api/**/route.js`) and server action files for AI API usage patterns.

**CRITICAL — Long-running AI streaming without timeout caps:**
Grep for AI provider imports: `openai`, `anthropic`, `@anthropic-ai/sdk`, `@ai-sdk`, keywords like `chat/completions`, `stream: true`, `messages.create`.

For any file using AI APIs, Read the file and check:
- Is there an `export const maxDuration = N` statement?
- If streaming (`stream: true`), is there an AbortSignal or timeout control?

Without `maxDuration`, functions can run up to 300s on Pro (costs add up fast). Without timeout caps, long-running AI requests can cause bills to spike from $20 to $1000+/month.

- Fix: Add `export const maxDuration = 30` (or appropriate value based on model). Add AbortSignal with timeout:

```typescript
export const maxDuration = 30;

export async function POST(req: Request) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), 25000);

  try {
    const completion = await openai.chat.completions.create(
      { model: 'gpt-4', messages: [...], stream: true },
      { signal: controller.signal }
    );
    clearTimeout(timeoutId);
    return new StreamingTextResponse(OpenAIStream(completion));
  } catch (err) {
    clearTimeout(timeoutId);
    if (err.name === 'AbortError') {
      return Response.json({ error: 'Request timeout' }, { status: 504 });
    }
    throw err;
  }
}
```

**WARNING — Streaming responses without proper chunking:**
Check if files with `stream: true` return a `ReadableStream` or `StreamingTextResponse`. If they collect chunks into an array or string before returning, they're buffering in memory (defeats streaming benefits and can cause memory issues).

- Fix: Use `StreamingTextResponse` from `ai` package or return a proper `ReadableStream`.

**WARNING — AI responses in middleware:**
Grep for AI API calls (`openai`, `anthropic`, AI-related imports) in `middleware.ts`, `middleware.js`, `proxy.ts`, or `proxy.js`.

Middleware runs on Edge runtime with a 30s timeout limit. AI calls often take 10-30+ seconds and are unpredictable. This causes frequent 504 timeouts.

- Fix: Move AI logic to API routes or Server Actions. Use middleware only for lightweight checks (e.g., cache lookups).

**WARNING — Missing maxDuration on AI routes:**
For any route with AI API calls that doesn't have `export const maxDuration`, flag it. On Pro plans, the default is 300s. On Hobby, it's 60s. Without an explicit cap, you're vulnerable to runaway billing.

- Fix: Add `export const maxDuration = N` where N is appropriate for the AI model being used (GPT-3.5: 15s, GPT-4: 30s, Claude Opus: 60s).

For detailed AI cost optimization patterns, see [reference/ai-costs.md](reference/ai-costs.md).

---

## Check 13: Request/Response Payload Limits

Vercel has a **4.5 MB limit** on request and response payloads for standard functions. Exceeding this causes `FUNCTION_PAYLOAD_TOO_LARGE` errors (413).

**CRITICAL — Large JSON payloads in API routes:**
Use Grep to find API routes that accept file uploads or large data:
- Pattern: `await req.json()`, `await req.formData()`, file upload patterns

For routes that accept uploads or large POST bodies, check if they handle files > 4.5 MB or use Base64 encoding (which inflates size by 33%).

Calculate potential payload size:
- Image files: 3.5 MB image → 4.67 MB as Base64 (exceeds limit!)
- Form data with multiple files
- Large JSON arrays

- Fix: Use streaming functions for payloads > 4.5 MB:

```typescript
export async function POST(req: Request) {
  // Don't call req.json() — work with raw stream
  const stream = req.body;

  await uploadToStorage(stream); // Stream directly to S3/R2
  return Response.json({ success: true });
}
```

**WARNING — Base64 file encoding in API routes:**
Grep for Base64 encoding patterns: `btoa(`, `atob(`, `base64`, `data:image`, `Buffer.from(.*base64)`.

Base64 increases file size by ~33%. A 3.5 MB file becomes 4.67 MB (exceeds limit).

- Fix: Use FormData with binary files instead of Base64, or stream directly to external storage from the client:

```typescript
// Bad: Base64 (inflates size)
const base64 = await fileToBase64(image);
fetch('/api/upload', { body: JSON.stringify({ image: base64 }) });

// Good: FormData (raw binary)
const formData = new FormData();
formData.append('image', file);
fetch('/api/upload', { body: formData });

// Better: Direct upload to S3/R2 (bypasses Vercel entirely)
const { uploadUrl } = await fetch('/api/upload-url').then(r => r.json());
await fetch(uploadUrl, { method: 'PUT', body: file });
```

**INFO — Missing streaming function detection:**
Check if the project has any routes using streaming for large responses. If not, and there are API routes returning large data, suggest considering streaming.

- Fix: For responses > 4.5 MB, use streaming:

```typescript
export async function GET() {
  const largeData = await fetchLargeDataset();

  const stream = new ReadableStream({
    start(controller) {
      controller.enqueue(JSON.stringify(largeData));
      controller.close();
    },
  });

  return new Response(stream, {
    headers: { 'Content-Type': 'application/json' },
  });
}
```

---

## Check 14: Database N+1 Query Patterns

Use Grep to search for common N+1 query patterns in API routes, Server Actions, and server components.

**WARNING — Loop with await inside:**
Pattern: `for.*await`, `\.map.*async.*=>.*await`, `.forEach.*await`

These patterns often indicate sequential database queries inside a loop — each iteration waits for a query result before starting the next.

Example of bad pattern:
```typescript
for (const post of posts) {
  post.author = await db.findUser(post.authorId); // N queries
}
```

- Fix: Use `Promise.all()` for independent queries, or use a single query with JOIN/IN clause:

```typescript
// Option 1: Promise.all for independent queries
const authors = await Promise.all(
  posts.map(post => db.findUser(post.authorId))
);

// Option 2: Single query with IN clause (better)
const authorIds = [...new Set(posts.map(p => p.authorId))];
const authors = await db.findMany({ where: { id: { in: authorIds } } });
```

**WARNING — Multiple sequential ORM find() calls:**
Grep for multiple sequential database query calls in the same function:
- Prisma: multiple `prisma.*.findMany()` or `prisma.*.findUnique()` in sequence
- Drizzle: multiple `db.select().from()` in sequence
- Supabase: multiple `supabase.from().select()` in sequence

Each query is a network round-trip (20-50ms same-region, 100-200ms cross-region). Sequential queries multiply function duration.

- Fix: Use ORM relation loading or parallel queries:

```typescript
// Bad: Sequential (300ms total if 3 × 100ms per query)
const user = await prisma.user.findUnique({ where: { id } });
const posts = await prisma.post.findMany({ where: { authorId: id } });
const comments = await prisma.comment.findMany({ where: { authorId: id } });

// Good: Parallel (100ms total, all run concurrently)
const [user, posts, comments] = await Promise.all([
  prisma.user.findUnique({ where: { id } }),
  prisma.post.findMany({ where: { authorId: id } }),
  prisma.comment.findMany({ where: { authorId: id } }),
]);

// Better: Single query with relations
const user = await prisma.user.findUnique({
  where: { id },
  include: { posts: true, comments: true },
});
```

**INFO — Missing connection pooling:**
Check `package.json` for database client packages: `@prisma/client`, `drizzle-orm`, `pg`, `postgres`, `mysql2`, `@supabase/supabase-js`.

Then check:
- If using Prisma: Look for `@prisma/extension-accelerate` (Prisma Accelerate provides connection pooling)
- If using Postgres: Check for PgBouncer or Supabase Pooler config in env vars
- If using PlanetScale/Neon: Pooling is built-in (note this as a positive finding)

Without connection pooling, every function invocation creates a new database connection (100-500ms overhead on cold starts).

- Fix: Enable connection pooling:
  - Prisma: Add Prisma Accelerate
  - Supabase: Enable Pooler in dashboard and use pooler connection string
  - Self-hosted Postgres: Add PgBouncer
  - PlanetScale/Neon: No action needed (built-in)

For detailed database optimization patterns, see [reference/database-optimization.md](reference/database-optimization.md).

---

## Check 15: Missing Compression Headers

Vercel automatically compresses responses > 1 KB, but explicit configuration ensures optimal compression for all content types.

**WARNING — API routes with large JSON responses lacking compression:**
Find API routes (`app/api/**/route.ts`) that return large datasets (arrays, objects).

While Vercel auto-compresses JSON > 1 KB, responses near the 4.5 MB limit should explicitly hint compression. Gzip/Brotli can reduce JSON by 70-90%.

- Fix: Add explicit content headers (Vercel handles the actual compression):

```typescript
export async function GET() {
  const largeData = await getLargeDataset();

  return new Response(JSON.stringify(largeData), {
    headers: {
      'Content-Type': 'application/json',
      'Cache-Control': 'public, s-maxage=60',
    },
  });
}
```

**WARNING — Middleware doubling Fast Data Transfer:**
Check if `middleware.ts`/`proxy.ts` reads or modifies request/response bodies (pattern: `await req.json()`, `await req.text()`, `NextResponse.json(modifiedBody)`).

When middleware reads/modifies bodies, data is transferred multiple times:
1. Client → Edge (counted as FDT)
2. Edge → Middleware (processes body) → Edge (counted again)
3. Edge → Origin → Edge → Client (counted again)

Result: Same data counted 2-3x in bandwidth billing.

- Fix: Don't process bodies in middleware. Use middleware only for headers, redirects, and rewrites:

```typescript
// Bad: Reading/modifying body in middleware
export async function middleware(req: NextRequest) {
  const body = await req.json(); // Triggers FDT
  const modified = transformBody(body);
  return NextResponse.json(modified); // Triggers FDT again
}

// Good: Only inspect headers
export async function middleware(req: NextRequest) {
  const authHeader = req.headers.get('authorization');
  if (!authHeader) {
    return new Response('Unauthorized', { status: 401 });
  }
  return NextResponse.next();
}
```

**INFO — SVG files without compression:**
Use Glob to find `.svg` files in `public/`. SVG files are XML-based text and compress 60-80% with gzip, but are often served uncompressed.

- Fix: Ensure `next.config.js` has compression headers for SVG mime type, or pre-compress SVGs at build time:

```javascript
// next.config.js
async headers() {
  return [
    {
      source: '/:all*(svg)',
      headers: [
        {
          key: 'Content-Type',
          value: 'image/svg+xml',
        },
        {
          key: 'Cache-Control',
          value: 'public, max-age=31536000, immutable',
        },
      ],
    },
  ];
}
```

---

## Check 16: ISR Read/Write Cost Patterns

Use Grep to find pages with ISR configuration (`export const revalidate`).

**WARNING — ISR with very short revalidate (< 60s):**
Grep for `export const revalidate\s*=\s*\d+` and extract the value.

If `revalidate` is less than 60 seconds, the page revalidates very frequently. Each revalidation triggers:
- 1 function invocation
- Compute time to render the page
- Fast Origin Transfer (page HTML from origin to edge)

For a page revalidating every 30 seconds under constant traffic:
```
Revalidations per hour: 120
Revalidations per day: 2,880
Revalidations per month: ~86,400 function invocations
```

- Fix: Increase to at least 300 seconds (5 minutes) unless real-time freshness is critical:

```typescript
// Bad: Revalidates every 30 seconds
export const revalidate = 30;

// Good: Revalidates every 5 minutes
export const revalidate = 300;

// Better: Revalidates every hour
export const revalidate = 3600;
```

**INFO — ISR without on-demand revalidation:**
For pages with `export const revalidate`, check if the project has any usage of `revalidatePath()` or `revalidateTag()` (Grep for these patterns in API routes or Server Actions).

If not found, the project is only using time-based revalidation. This means content can't be updated immediately when changed — you have to wait until the revalidation period expires.

- Fix: Add on-demand revalidation for time-sensitive content updates:

```typescript
// app/api/revalidate/route.ts
import { revalidatePath } from 'next/cache';

export async function POST(req: Request) {
  const { path } = await req.json();

  // Trigger immediate revalidation when content changes
  revalidatePath(path);

  return Response.json({ revalidated: true, now: Date.now() });
}

// Hybrid approach: time-based fallback + on-demand for immediate updates
export const revalidate = 3600; // Fallback every hour

// In page.tsx or CMS webhook: call revalidatePath when content changes
```

---

## Check 17: Vercel Fluid Compute Detection

This check reminds the user to enable Vercel Fluid Compute if not already enabled.

**INFO — Fluid Compute eligibility check:**
Check if the project is deployed on Vercel (look for `vercel.json`, `.vercel/` directory, or `VERCEL` in environment variables).

If the project is on Vercel and not on the Hobby plan, Fluid Compute can reduce function costs by 20-30% for I/O-heavy workloads (database queries, API calls, AI workloads).

Fluid Compute uses Active CPU pricing:
- Active CPU time billed at standard rate
- I/O wait time (database queries, API calls) billed at lower rate

For functions with database queries or AI API calls, most of the time is I/O wait, making Fluid Compute very cost-effective.

- Fix: Enable Fluid Compute in the Vercel Dashboard:
  1. Go to **Dashboard → [Project] → Settings → Functions**
  2. Scroll to **"Function Duration and Fluid Compute"**
  3. Toggle **"Enable Fluid Compute"** to ON
  4. Deploy the project (applies to new deployments)

**INFO — I/O-heavy functions without optimization:**
For functions with database queries or external API calls, remind the user that Fluid Compute bills active CPU at a higher rate than I/O wait time. Parallelizing I/O operations reduces total function time (and thus total cost).

- Fix: Use `Promise.all()` to parallelize independent I/O operations (see Check 10 and Check 14).

---

## Check 18: Function Timeout Patterns

Use Grep to find routes with long-running operations that may need timeout configuration.

**WARNING — AI/streaming routes without maxDuration on Hobby:**
For routes with AI API calls (detected in Check 12), check if there's an `export const maxDuration` statement.

If not, and the project appears to be on the Hobby plan (no Pro plan indicators like `vercel.json` with custom regions or enterprise features), warn that the Hobby plan has a 60s hard timeout.

AI models like GPT-4 streaming can take 30-60 seconds. Without explicit timeout configuration, requests may hit the 60s limit and fail with 504 errors.

- Fix: Add `export const maxDuration = 60` (or lower) for Hobby plan, or upgrade to Pro for longer timeouts:

```typescript
// For Hobby plan (60s max)
export const maxDuration = 60;

// For Pro plan (300s default, up to 900s max)
export const maxDuration = 300;
```

**WARNING — Database operations without timeout:**
For routes with database queries, check if the database client has timeout configuration.

Long-running queries (e.g., full table scans, complex joins on large datasets) can max out function duration and cause high costs.

- Fix: Add query timeout at the client level:

```typescript
// Prisma
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL,
    },
  },
  // Add query timeout
  log: [{ level: 'query', emit: 'event' }],
});

// Postgres (pg package)
const client = new Client({
  connectionString: process.env.DATABASE_URL,
  statement_timeout: 10000, // 10 seconds
});

// Supabase
const { data, error } = await supabase
  .from('posts')
  .select('*')
  .abortSignal(AbortSignal.timeout(10000)); // 10 seconds
```

---

## Check 19: Fetch Deduplication

Use Grep to search for `fetch()` calls in Server Components (files without `'use client'` or `"use client"`).

**INFO — Multiple identical fetch() calls:**
Check for the same URL fetched multiple times in server components or API routes.

Next.js automatically deduplicates `fetch()` requests with the same URL within a single render, but explicit deduplication makes intent clearer and works across renders.

- Fix: Use `React.cache()` for explicit deduplication:

```typescript
import { cache } from 'react';

const getUser = cache(async (id: string) => {
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
});

// Multiple calls to getUser(id) with the same id will only fetch once
```

**INFO — fetch() with cache: 'no-store' in server components:**
Pattern: `fetch(.*cache.*no-store)` or `fetch(.*cache.*no-cache)`

Server components with `cache: 'no-store'` opt out of automatic fetch deduplication and caching. This causes the same data to be fetched multiple times.

- Fix: Remove `cache: 'no-store'` unless the data is truly per-request:

```typescript
// Bad: Opts out of caching
const res = await fetch('https://api.example.com/data', {
  cache: 'no-store',
});

// Good: Use default caching or explicit revalidation
const res = await fetch('https://api.example.com/data', {
  next: { revalidate: 60 }, // Cache for 60 seconds
});

// Or remove options entirely (Next.js caches by default)
const res = await fetch('https://api.example.com/data');
```

---

## Check 20: Build-Time Optimization

Use Read to check `package.json` for dependencies and `next.config.js`/`next.config.ts` for output mode.

**INFO — Large dependencies in package.json:**
Check for heavy dependencies that have lightweight alternatives:
- `moment` → Replace with `date-fns` (92% smaller)
- `lodash` (without tree-shaking) → Replace with `lodash-es` or individual imports
- Large UI libraries → Check for tree-shaking configuration

These dependencies increase:
- Build time (slower deployments)
- Function bundle size (slower cold starts)
- Overall deployment size

- Fix: Replace with smaller alternatives:

```bash
# Replace moment with date-fns
npm uninstall moment
npm install date-fns

# Use tree-shakeable lodash
npm uninstall lodash
npm install lodash-es
```

**INFO — Missing output: 'standalone' in next.config:**
Check if `next.config.js`/`next.config.ts` has `output: 'standalone'`.

Without standalone mode, the deployment bundle includes all of `node_modules`, which can be very large. Standalone mode only includes production dependencies actually used by the app (~80% size reduction).

- Fix: Add `output: 'standalone'` to `next.config.js`:

```javascript
// next.config.js
const nextConfig = {
  output: 'standalone', // Reduces deployment size by ~80%
};
```

Note: This is automatically enabled for Vercel deployments, but worth checking for self-hosted or custom deployment setups.

---

## Check 21: Connection Pooling

Use Read to check database client configuration (already partially covered in Check 14, but this check focuses on configuration details).

**WARNING — Database client without pooling config:**
For projects using database clients (Prisma, Drizzle, Postgres.js, MySQL2), check if connection pooling is configured.

Grep environment variables (`.env.local`, `.env`) for pooling-related config:
- Prisma Accelerate: `PRISMA_ACCELERATE_URL` or `@prisma/extension-accelerate` in code
- Supabase Pooler: Connection string with `:6543` port or `?pgbouncer=true`
- PgBouncer: Separate pooler URL or config
- PlanetScale/Neon: Built-in (no config needed, note as positive finding)

Without pooling, each function instance creates a new database connection:
- Cold start overhead: 100-500ms
- Connection exhaustion: Databases have max connection limits (Postgres default: 100)

- Fix: Enable connection pooling:

**Prisma:**
```typescript
import { PrismaClient } from '@prisma/client';
import { withAccelerate } from '@prisma/extension-accelerate';

const prisma = new PrismaClient().$extends(withAccelerate());
```

**Supabase:**
```bash
# .env.local
# Use pooler connection string instead of direct connection
DATABASE_URL=postgresql://user:pass@pooler.supabase.com:6543/postgres?pgbouncer=true
```

**PgBouncer (self-hosted):**
Set up PgBouncer as a separate service and point your app to it instead of Postgres directly.

**INFO — Redis client without connection reuse:**
Check if the project uses Redis (`@upstash/redis`, `ioredis`, `redis`).

If the Redis client is created inside function handlers (instead of module scope), it's created fresh on every invocation.

- Fix: Move Redis client to module scope with singleton pattern:

```typescript
// Bad: Creates new client on every invocation
export async function GET() {
  const redis = new Redis({ url: process.env.REDIS_URL });
  const data = await redis.get('key');
  return Response.json(data);
}

// Good: Client created once and reused
import { Redis } from '@upstash/redis';

const redis = new Redis({ url: process.env.REDIS_URL }); // Module scope

export async function GET() {
  const data = await redis.get('key');
  return Response.json(data);
}
```

---

## Check 22: App Router Performance

Use Grep to analyze App Router vs Pages Router patterns and detect performance anti-patterns.

**INFO — App Router with many client components:**
Count files with `'use client'` or `"use client"` in the `app/` directory vs total component files.

If the ratio is high (e.g., > 50% of components are client components), the project may not be leveraging App Router benefits (server-first rendering, streaming).

- Fix: Move state to fewer client components and use server components for data fetching:

```typescript
// Bad: Everything is a client component
'use client';

export default function Page() {
  const [data, setData] = useState(null);

  useEffect(() => {
    fetch('/api/data').then(r => r.json()).then(setData);
  }, []);

  return <div>{data?.title}</div>;
}

// Good: Server component for data, client component only for interactivity
// app/page.tsx (Server Component)
async function getData() {
  const res = await fetch('https://api.example.com/data');
  return res.json();
}

export default async function Page() {
  const data = await getData();
  return <ClientComponent data={data} />;
}

// components/ClientComponent.tsx (only interactive parts)
'use client';

export function ClientComponent({ data }) {
  const [isExpanded, setIsExpanded] = useState(false);
  return (
    <div onClick={() => setIsExpanded(!isExpanded)}>
      {data.title}
      {isExpanded && <Details />}
    </div>
  );
}
```

**INFO — Parallel routes or intercepting routes usage:**
Grep for parallel routes (directories with `@` prefix like `@modal`, `@sidebar`) or intercepting routes (directories with `(..)` like `(.)post`).

These are advanced App Router features that can add complexity. If used, ensure proper layout optimization and loading states to avoid performance issues.

- Fix: Ensure proper loading states and suspense boundaries:

```typescript
// app/@modal/loading.tsx
export default function Loading() {
  return <ModalSkeleton />;
}

// app/layout.tsx
export default function Layout({ children, modal }) {
  return (
    <>
      {children}
      <Suspense fallback={<ModalSkeleton />}>
        {modal}
      </Suspense>
    </>
  );
}
```

If parallel/intercepting routes are not adding clear value, consider simplifying to standard routes.

---

## Report Format

After completing all 22 checks, compile findings using the appropriate template below.

**If there are any CRITICAL or WARNING findings**, use Template A.
**If there are 0 critical AND 0 warnings**, use Template B.

---

### Template A: Standard Report

```
# Vercel Cost Guard

**[project name]** — [today's date] — [N] critical · [N] warning · [N] info

---

## Critical Issues

[For each CRITICAL finding, render a detail block like the one below.
Order by estimated savings descending — highest-impact issue first.]

### [CHECK_NAME]: [short description]

**File:** `[path/to/file:line]`

**Impact:** [1-2 sentence explanation of why this is expensive, with dollar estimate.
Use formulas from reference/vercel-pricing.md. State the traffic assumption, e.g.
"At 50K monthly visitors, this 52 MB file transfers ~2.6 TB/mo (~$240 in overage)."]

**Fix:**
```[language or bash]
[Exact command or code snippet to fix the issue.
Use the project's actual file paths.
Pull specific commands from reference/optimization-guide.md.]
```

**Estimated savings:** ~$[amount]/mo (or ~[X]% bandwidth reduction)

---

[Repeat for each critical issue, separated by ---]

## Warnings

[Render ALL warning findings in a compact table. One row per finding.
Do not include full fix details — just a short actionable summary.
Order by estimated impact descending.]

| # | Check | File | Issue | Fix |
|---|-------|------|-------|-----|
| 1 | [CHECK_NAME] | `[path]` | [1-line description] | [1-line action] |
| 2 | ... | ... | ... | ... |

## Info

[If there are 0 info findings, omit this section entirely.
Otherwise, render as a compact bullet list.]

- **[CHECK_NAME]** — `[path]`: [description and suggested action]
- ...

## Implementation Checklist

[Consolidate every critical and warning finding into a single numbered checklist.
Critical items first, then warnings. Each item gets a copy-paste command or code snippet.
Group related items if sensible (e.g., multiple files needing the same conversion).]

- [ ] **1.** [Action description — e.g., "Convert hero.mov to MP4"]
  Saves ~$[amount]/mo
  ```[bash or language]
  [exact command or code change with real file paths]
  ```

- [ ] **2.** [Next action]
  Saves ~$[amount]/mo
  ```[bash or language]
  [exact command or code change]
  ```

[Continue for all critical + warning items...]

## Cost Optimization Summary

**Total estimated monthly savings:** ~$[sum of all critical + warning fixes]

**Breakdown:**
- Bandwidth reduction: ~$[amount from media compression, payload optimization]
- Compute reduction: ~$[amount from parallelization, timeout caps, ISR]
- Invocation reduction: ~$[amount from middleware scoping, static generation, caching]
- Image optimization: ~$[amount from image config improvements]

**Enable Vercel Fluid Compute** in Dashboard → Project Settings → Functions for automatic 20-30% additional savings on function costs (Pro/Enterprise plans only).

## Spend Management

> Review your Spend Management limit at: **Dashboard → Settings → Billing → Spend Management**
> Alerts trigger at 50%, 75%, and 100% of your budget. Enable "Pause production deployment" for a hard cap.
> Recommended: $20-50 for hobby projects, expected traffic + 50% buffer for production.
```

---

### Template B: Clean Project

```
# Vercel Cost Guard

**[project name]** — [today's date] — 0 critical · 0 warnings · [N] info

**No critical or warning-level cost issues found.**

## What You're Doing Right

[List positive findings verified during the audit. Only include items that were
actually checked and confirmed — do not fabricate positive findings.]

- [checkmark] Media files use compressed formats (mp4, mp3, webp)
- [checkmark] All audio/video elements have `preload="none"`
- [checkmark] `next.config` has `minimumCacheTTL` set to [value]
- [checkmark] Middleware is properly scoped with matcher config
- [checkmark] Dynamic routes use `generateStaticParams` or ISR
- [checkmark] API routes include Cache-Control headers
- [checkmark] Total public/ media is only ~[size] MB

## Optional Improvements

[If there are INFO findings, list them here:]

These won't cause billing issues, but could yield minor improvements:

- **[CHECK_NAME]** — `[path]`: [description and suggested action]
- ...

[If there are 0 info findings, replace the list above with:]

No additional recommendations. This project is well-optimized for Vercel's pricing model.

## Spend Management

> Review your Spend Management limit at: **Dashboard → Settings → Billing → Spend Management**
> Alerts trigger at 50%, 75%, and 100% of your budget. Enable "Pause production deployment" for a hard cap.
> Recommended: $20-50 for hobby projects, expected traffic + 50% buffer for production.
```

---
> Source: [jshchnz/vercel-cost-guard](https://github.com/jshchnz/vercel-cost-guard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
