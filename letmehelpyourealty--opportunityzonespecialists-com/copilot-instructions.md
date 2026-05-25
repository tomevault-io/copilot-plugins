## no-duplicate-features

> Prevent duplicating features that platforms already provide natively


# Don't Duplicate Platform Features

**ALWAYS check what platforms already provide before building.**

## Platform Feature Matrix

### Follow Up Boss
✅ **Already Has:**
- Full REST API with webhooks
- Native RealScout integration (2-way sync)
- Built-in automation (action plans, drip campaigns)
- Lead deduplication
- Stage management
- Tag management
- Bulk operations
- Reporting dashboards

❌ **Don't Build:**
- Custom webhook handlers for RealScout sync (use native integration)
- Lead deduplication logic (FUB does this)
- Bulk tag/stage operations (use FUB UI or API directly)

✅ **DO Build (Enhancements):**
- Rate limiting wrapper (FUB API doesn't provide this)
- Response caching (reduce duplicate API calls)
- TypeScript type safety

### RealScout
✅ **Already Has:**
- AI-powered property matching (2025 release)
- Natural language search (December 2025)
- Native Follow Up Boss integration
- Behavioral learning from user activity
- Adaptive property alerts
- Client activity tracking
- Property recommendations

❌ **Don't Build:**
- Property recommendation engine (RealScout AI does this)
- Activity tracking (built-in)
- Search optimization (they just added AI search)
- Lead sync to FUB (native integration)

✅ **DO Build (Enhancements):**
- Custom widget styling
- Analytics tracking beyond RealScout's

### Calendly
✅ **Already Has:**
- Scheduling widget
- Email notifications
- Calendar sync
- Integrations

❌ **Don't Build:**
- Custom scheduling logic
- Notification system

✅ **DO Build:**
- React component wrappers (convenience only)

### Cloudflare
✅ **Already Has:**
- Global CDN
- Basic caching
- DDoS protection

❌ **Don't Build:**
- Redundant CDN layer

✅ **DO Build:**
- Custom Workers for specific logic
- Advanced caching rules
- Edge functions

### Vercel
✅ **Already Has:**
- Auto-deployments
- Preview URLs
- Analytics

❌ **Don't Build:**
- Custom deployment system

✅ **DO Build:**
- Custom GitHub Actions (beyond Vercel's automation)
- Lighthouse CI
- Bundle analysis

## Rule: Always Ask First

**Before building any integration, ask:**

1. Does this platform already have this feature?
2. Is there a native integration I should use instead?
3. Am I adding real value or just wrapping their API?

## Examples

❌ **BAD: Duplicate**
```typescript
// Building custom RealScout → FUB sync
// when native integration exists
```

✅ **GOOD: Enhancement**
```typescript
// Adding rate limiting to FUB API calls
// (FUB API doesn't provide this)
```

---
> Source: [LetMeHelpYouREALTY/opportunityzonespecialists.com](https://github.com/LetMeHelpYouREALTY/opportunityzonespecialists.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
