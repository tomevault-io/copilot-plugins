## catcv

> Title: AI (Vercel AI SDK + OpenAI)


Title: AI (Vercel AI SDK + OpenAI)

- **Models**: Default `gpt-5-mini`. Upgrade via flag or explicit request to `gpt-5`.
- **Env**: `AI_MODEL` and `AI_MODEL_PRO`. No hardcoded model strings.
- **Schema-bound**: Use `generateObject` with `zod` for structured outputs.
- **Prompt hygiene**: Sanitize user text. Scope instructions. Avoid leaking system prompts.
- **Rate limit**: Per-user + per-IP. Backoff + friendly errors.
- **Streaming**: Use `streamText` for cover letters. Flush early tokens.
- **Version prompts**: Keep templates in `lib/ai/*`; version changes.

Model Select

```ts
const model = pro ? process.env.AI_MODEL_PRO : process.env.AI_MODEL
```

Structured Output

```ts
const result = await generateObject({
  model,
  schema: ResumeSchema,
  prompt: safePrompt,
})
```

Title: AI (Vercel AI SDK + OpenAI)

- **Models**: Default `gpt-5-mini`. Upgrade via flag or explicit request to `gpt-5`.
- **Env**: `AI_MODEL` and `AI_MODEL_PRO`. No hardcoded model strings.
- **Schema-bound**: Use `generateObject` with `zod` for structured outputs.
- **Prompt hygiene**: Sanitize user text. Scope instructions. Avoid leaking system prompts.
- **Rate limit**: Per-user + per-IP. Backoff + friendly errors.
- **Streaming**: Use `streamText` for cover letters. Flush early tokens.
- **Version prompts**: Keep templates in `lib/ai/*`; version changes.

Model Select

```ts
const model = pro ? process.env.AI_MODEL_PRO : process.env.AI_MODEL
```

Structured Output

```ts
const result = await generateObject({
  model,
  schema: ResumeSchema,
  prompt: safePrompt,
})
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Gurbeh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
