## battle-nads-frontend

> Provide guidance on using Edge or Node.js runtimes for Next.js routes.

# Edge vs. Serverless Functions – Use the Right Runtime

## Context
- Next.js supports two server runtimes: Edge and Node.js (Serverless)
- Each has different capabilities, constraints, and performance characteristics

## Requirements
- Choose the appropriate runtime based on the route's specific needs
- Use Edge Runtime (`export const runtime = 'edge'`) for:
  - Global low-latency requirements
  - Simple, lightweight operations
  - Basic personalization at the network edge
- Use Node.js Runtime (default or `export const runtime = 'nodejs'`) for:
  - Complex database operations
  - Heavy computation
  - Large dependencies and packages
  - Node.js-specific APIs or modules
- Document the rationale for choosing Edge runtime when used

## Examples

<example>
// Good: Edge runtime for a lightweight auth check
// app/api/auth/validate/route.ts
export const runtime = 'edge';

export async function GET(request: Request) {
  const token = request.headers.get('Authorization')?.split(' ')[1];
  
  // Simple JWT verification without heavy libraries
  if (!token) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  // Quick validation logic
  const isValid = verifyJWT(token);
  
  return new Response(JSON.stringify({ valid: isValid }), {
    headers: { 'Content-Type': 'application/json' }
  });
}

// Good: Node.js runtime for heavy database operations
// app/dashboard/page.tsx
export const runtime = 'nodejs'; // Optional, as this is the default

export default async function DashboardPage() {
  // Complex DB query with ORM
  const stats = await prisma.analytics.aggregateStats({
    // Complex aggregation that needs full Node.js capabilities
  });
  
  return <DashboardView stats={stats} />;
}
</example>

<example type="invalid">
// Bad: Using Edge runtime with incompatible dependencies
// app/api/generate-pdf/route.ts
export const runtime = 'edge';

export async function POST(request: Request) {
  const data = await request.json();
  
  // This won't work! PDF libraries typically need full Node.js
  // and exceed Edge bundle size limits
  const pdf = await generatePDF(data.template, data.content);
  
  return new Response(pdf, {
    headers: { 'Content-Type': 'application/pdf' }
  });
}
</example>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/FastLane-Labs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/FastLane-Labs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
