## cotacao-ibov

> API Route conventions for Next.js server handlers


# API Route Conventions

## Imports
Always use `NextRequest` and `NextResponse`:

```typescript
import { NextRequest, NextResponse } from "next/server";
```

## Input Validation
Validate all query parameters before using them:

```typescript
const code = searchParams.get("code");
if (!code || code.trim() === "") {
  return NextResponse.json({ error: "O campo 'code' é obrigatório." }, { status: 400 });
}
```

## Token Handling
Read `BRAPI_TOKEN` from `process.env` — never hardcode or pass to the client:

```typescript
const token = process.env.BRAPI_TOKEN;
if (!token) {
  return NextResponse.json({ error: "Token da API não configurado." }, { status: 500 });
}
```

## Brapi Fetch Pattern
- Normalize tickers to uppercase before calling Brapi
- Use `next: { revalidate: 60 }` to cache responses for 60 seconds
- Mirror Brapi's HTTP status code back to the client

```typescript
const ticker = code.trim().toUpperCase();
const url = `https://brapi.dev/api/quote/${encodeURIComponent(ticker)}?token=${token}`;
const res = await fetch(url, { next: { revalidate: 60 } });
const data: BrapiResponse = await res.json();
return NextResponse.json(data, { status: res.status });
```

## Error Response Shape
Always return `{ error: string }` for error responses so the client can read `data.error`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adrianodonisete) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
