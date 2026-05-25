## api-optimization-patterns

> API optimization patterns - caching, rate limiting, error handling


# API Optimization Patterns (2026)

## Claude AI - Always Use Prompt Caching

```typescript
// ✅ GOOD: Prompt caching enabled (90% cost savings)
const response = await claude.sendMessage({
  messages: [{ role: 'user', content: userQuery }],
  systemPrompt: reusableTemplate.system, // This gets cached!
  enableCache: true, // Default: true
});

// Cost: First request $0.001, subsequent $0.0001 (cached)

// ❌ BAD: Disabling cache or dynamic prompts
const response = await claude.sendMessage({
  messages: [{ role: 'user', content: userQuery }],
  systemPrompt: `You are ${dynamicAgent}...`, // Changes every time = no cache!
  enableCache: false,
});

// Cost: Every request $0.001 (10x more expensive)
```

## Rate Limiting Pattern

```typescript
// ✅ GOOD: Token bucket algorithm
class RateLimiter {
  private tokens: number;
  private lastRefill: number;

  async checkLimit(): Promise<void> {
    this.refillTokens();
    if (this.tokens < 1) {
      await this.sleep(this.getWaitTime());
    }
    this.tokens -= 1;
  }
}

// ❌ BAD: No rate limiting (causes 429 errors)
```

## Response Caching

```typescript
// ✅ GOOD: Cache identical queries
const cacheKey = hashQuery(query);
const cached = await cache.get(cacheKey);
if (cached) return cached;

const result = await apiCall(query);
await cache.set(cacheKey, result, ttl);

// ❌ BAD: No caching (duplicate API calls)
const result = await apiCall(query); // Same query? Full API call!
```

## Error Handling

```typescript
// ✅ GOOD: Exponential backoff
let delay = 1000;
for (let attempt = 0; attempt < 3; attempt++) {
  try {
    return await apiCall();
  } catch (error) {
    if (error.status === 429) {
      await sleep(delay);
      delay *= 2;
    } else {
      throw error;
    }
  }
}

// ❌ BAD: Immediate retry
try {
  return await apiCall();
} catch {
  return await apiCall(); // Hammers API on failure
}
```

## Cost Tracking

```typescript
// ✅ GOOD: Track every API call
class CostTracker {
  addRequest(cost: number) {
    this.requests.push({ cost, timestamp: Date.now() });
  }
  
  getStats() {
    return {
      last24h: this.requests.filter(r => isLast24h(r)),
      total: this.requests.reduce((sum, r) => sum + r.cost, 0),
    };
  }
}

// ❌ BAD: No cost tracking (surprise bills!)
```

## API Best Practices Checklist

- [ ] Prompt caching enabled (Claude)
- [ ] Response caching implemented
- [ ] Rate limiting active
- [ ] Exponential backoff on errors
- [ ] Cost tracking enabled
- [ ] TypeScript types defined
- [ ] Error messages are helpful
- [ ] Logs include context

---
> Source: [LetMeHelpYouREALTY/opportunityzonespecialists.com](https://github.com/LetMeHelpYouREALTY/opportunityzonespecialists.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
