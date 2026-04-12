## lpmz-portfolio-v2

> API route quality and testability expectations


# API And Testing Quality

- Validate inputs early with guard clauses; fail fast with explicit status codes.
- Return consistent `NextResponse` payloads and headers for success and error paths.
- Keep route handlers thin: move reusable transformations/parsing into `src/lib` helpers.
- Preserve useful error context in logs and client-facing messages without leaking internals.
- When adding logic, structure code so units are testable in isolation (pure helpers, deterministic returns).
- Do not assume a test runner; provide behavior-focused test cases in PR notes until one is introduced.

## Example

```ts
// ❌ BAD: ambiguous failures and inline mixed concerns
export async function GET() {
  const data = await fetch(remoteUrl);
  return new NextResponse(await data.text());
}

// ✅ GOOD: guard clauses + explicit responses + clear errors
export async function GET() {
  try {
    const response = await fetch(remoteUrl);
    if (!response.ok) {
      return NextResponse.json(
        { error: `Upstream request failed: ${response.statusText}` },
        { status: response.status }
      );
    }
    return NextResponse.json({ ok: true }, { status: 200 });
  } catch (error) {
    return NextResponse.json(
      { error: `Unexpected API error: ${error instanceof Error ? error.message : 'Unknown error'}` },
      { status: 500 }
    );
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/linphone2000)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/linphone2000)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
