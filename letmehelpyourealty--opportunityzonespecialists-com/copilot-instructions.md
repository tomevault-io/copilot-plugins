## check-platform-features-first

> Always verify platform features before building custom solutions


# Check Platform Features FIRST

**Before building ANY integration, API wrapper, or automation:**

## Step 1: Research What Exists

```bash
# ALWAYS search documentation first:
# - "RealScout features 2026"
# - "Follow Up Boss API capabilities"
# - "Calendly integrations"
```

## Step 2: Ask These Questions

1. **Does the platform already do this?**
   - Check official docs
   - Search for native integrations
   - Look for recent updates (2025-2026)

2. **Is there a native integration?**
   - RealScout ↔ Follow Up Boss: ✅ NATIVE
   - Calendly ↔ Most CRMs: ✅ NATIVE
   - Always use native over custom

3. **Am I adding real value?**
   - Rate limiting: ✅ Useful
   - Caching: ✅ Useful
   - Wrapper around working API: ❌ Not useful

## Step 3: State What You Found

**Before building, say:**

> "I researched [Platform] and found:
> - Native integration with [Other Platform]: YES/NO
> - Built-in [feature]: YES/NO
> - My proposed solution: [enhancement/duplicate]
> 
> Should I proceed?"

## Real Examples

### ❌ BAD: Building Without Research
```typescript
// User: "Integrate RealScout with Follow Up Boss"
// AI: [Builds custom webhook handlers immediately]
// Result: Duplicates native integration
```

### ✅ GOOD: Research First
```typescript
// User: "Integrate RealScout with Follow Up Boss"
// AI: "RealScout has native FUB integration (Admin → Integrations).
//      Should I help configure the native integration instead?"
// Result: Uses what already exists
```

## Platforms to Always Check

**Real Estate:**
- RealScout (has AI search, FUB integration)
- Follow Up Boss (has webhooks, automation, integrations)
- Zillow Premier Agent
- Realtor.com
- MLS systems

**Marketing/CRM:**
- Calendly (scheduling)
- Mailchimp (email)
- HubSpot (CRM)

**Infrastructure:**
- Vercel (deployment)
- Cloudflare (CDN/security)
- GitHub (CI/CD)

## When to Build Custom

✅ Build custom solutions ONLY when:
- Platform doesn't have the feature
- You need specific optimization (caching, rate limiting)
- Native integration doesn't meet requirements
- Adding genuine intelligence (AI analysis)

❌ Don't build when:
- Platform already does it
- Native integration exists
- You're just wrapping their API
- It's "nice to have" convenience

---
> Source: [LetMeHelpYouREALTY/opportunityzonespecialists.com](https://github.com/LetMeHelpYouREALTY/opportunityzonespecialists.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-25 -->
