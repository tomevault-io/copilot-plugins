## next-js-15-async-request-api-rules

> description: Dictates how asynchronous requests should be handled within Next.js 15, specifically concerning runtime APIs.

---
description: Dictates how asynchronous requests should be handled within Next.js 15, specifically concerning runtime APIs.
globs: app/**/*
---
- Always use async versions of runtime APIs:
  typescript
  const cookieStore = await cookies()
  const headersList = await headers()
  const { isEnabled } = await draftMode()
  
- Handle async params in layouts/pages:
  typescript
  const params = await props.params
  const searchParams = await props.searchParams

---
> Source: [run-llama/auto_rfp](https://github.com/run-llama/auto_rfp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
