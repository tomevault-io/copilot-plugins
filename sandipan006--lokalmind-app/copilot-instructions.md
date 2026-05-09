## async-patterns

> Async patterns for LLM generation, downloads, and streaming in LokalMind v2.

# Async Patterns

Full reference: `docs/development/CONVENTIONS.md` section 6.

## LLM generation - required

```typescript
// 1. AbortSignal on every call
const controller = new AbortController();
await llmEngine.generateResponse(messages, config, onToken, controller.signal);

// 2. Token debounce - 50ms batch, never per-token runInAction
private buffer = '';
private flushTimer: ReturnType<typeof setTimeout> | null = null;
appendToken(token: string) {
  this.buffer += token;
  if (this.flushTimer) return;
  this.flushTimer = setTimeout(() => {
    runInAction(() => { this.text += this.buffer; this.buffer = ''; this.flushTimer = null; });
  }, 50);
}
```

## Rules

- Every generation: MUST accept `AbortSignal` · ViewModel MUST expose `cancelGeneration()`
- Token updates: debounced 50ms
- Download progress: throttled 250ms
- Memory extraction: fire-and-forget - `.catch(() => {})` - never await in main pipeline
- Never `await` inside MobX `reaction` or `autorun`

---
> Source: [Sandipan006/lokalmind-app](https://github.com/Sandipan006/lokalmind-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
