## cost-optimization-2026

> Cost optimization strategies for APIs and infrastructure (2026)


# Cost Optimization Strategies (2026)

## Claude AI - Prompt Caching (Critical)

**Impact:** 90% cost reduction, 85% latency improvement

```typescript
// ✅ GOOD: Cacheable system prompt (90% savings)
const systemPrompt = realEstateAgentTemplate.system; // Consistent

const response = await claude.sendMessage({
  messages: [{ role: 'user', content: userQuery }],
  systemPrompt, // Cached automatically
  enableCache: true,
});

// First request: $0.001
// Cached requests: $0.0001 (10x cheaper!)

// ❌ BAD: Dynamic prompt (no cache)
const systemPrompt = `You are ${agent} from ${company}...`; // Changes = no cache
// Every request: $0.001 (10x more expensive)
```

## Response Caching

```typescript
// ✅ GOOD: Cache frequent queries
const cacheKey = hashQuery(query);
const cached = await cache.get(cacheKey);
if (cached) return cached; // $0 cost!

const result = await apiCall(query);
await cache.set(cacheKey, result, 3600); // 1 hour TTL

// ❌ BAD: No caching
await apiCall(sameQuery); // Full API cost every time
```

## Model Selection

```typescript
// ✅ GOOD: Use cheapest model that works
const model = isSimpleQuery 
  ? 'claude-3-5-haiku-20241022'    // $0.80/1M input (75% cheaper)
  : 'claude-3-5-sonnet-20241022';  // $3.00/1M input

// ❌ BAD: Always use most expensive model
const model = 'claude-3-opus-20240229'; // $15/1M input (5x more!)
```

## Token Optimization

```typescript
// ✅ GOOD: Appropriate max tokens
const response = await claude.sendMessage({
  maxTokens: 500, // Short answer expected
});

// ❌ BAD: Over-requesting tokens
const response = await claude.sendMessage({
  maxTokens: 4096, // Way more than needed
});
// Costs more + slower responses
```

## Rate Limiting (Prevent Overages)

```typescript
// ✅ GOOD: Enforce limits before API calls
await rateLimiter.checkLimit();
const response = await apiCall();

// ❌ BAD: No rate limiting
for (let i = 0; i < 1000; i++) {
  await apiCall(); // 429 errors guaranteed!
}
```

## Cost Monitoring

```typescript
// ✅ GOOD: Track every request
costTracker.addRequest({
  cost: response.cost.total,
  tokens: response.usage.inputTokens + response.usage.outputTokens,
  cached: response.usage.cacheReadInputTokens > 0,
});

// ❌ BAD: No tracking (surprise bills)
```

## Monthly Cost Targets

| Service | Free Tier | Reasonable Cost | Excessive |
|---------|-----------|-----------------|-----------|
| Claude AI (1K req/day) | N/A | $12/mo (cached) | $120+/mo (no cache) |
| Cloudflare Workers | 100K req/day | Free | $5+/mo |
| Vercel | Hobby | Pro $20/mo | Enterprise $100+/mo |

## Cost Optimization Checklist

- [ ] Claude prompt caching enabled
- [ ] Response caching for duplicate queries
- [ ] Appropriate model selection (Haiku vs Sonnet)
- [ ] Reasonable max_tokens settings
- [ ] Rate limiting active
- [ ] Cost tracking dashboard
- [ ] Monthly budget alerts set
- [ ] Regular cost review (weekly)

---
> Source: [LetMeHelpYouREALTY/opportunityzonespecialists.com](https://github.com/LetMeHelpYouREALTY/opportunityzonespecialists.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
