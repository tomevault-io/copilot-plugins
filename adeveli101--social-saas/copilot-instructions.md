## social-saas

> PostHog analytics integration will be implemented in the future. This file contains guidelines for when the integration is added.

# PostHog Analytics Guidelines (Future Integration)

## Overview
PostHog analytics integration will be implemented in the future. This file contains guidelines for when the integration is added.

## Configuration (Future)
- PostHog client will be configured in `src/lib/posthog.ts`
- Provider will be set up in `src/app/providers.tsx`
- Environment variables: NEXT_PUBLIC_POSTHOG_KEY and NEXT_PUBLIC_POSTHOG_HOST

## Setup (Future)
- PostHog will be initialized only on the client side
- Debug mode will be enabled in development
- Page view tracking will be disabled in development
- Use the PostHogProvider wrapper in the root layout

## Event Tracking (Future)
- Track user actions with posthog.capture()
- Use descriptive event names
- Include relevant properties with events
- Track both user interactions and system events

## Common Events (Future)
```typescript
// User authentication
posthog?.capture('user_signed_up', { email })
posthog?.capture('user_signed_in', { email })

// Feature usage
posthog?.capture('post_created', { platform: 'instagram' })
posthog?.capture('post_scheduled', { platform: 'twitter' })
posthog?.capture('analytics_viewed', { section: 'dashboard' })

// User engagement
posthog?.capture('button_clicked', { button: 'new_post' })
posthog?.capture('page_viewed', { page: 'dashboard' })
```

## Feature Flags (Future)
- Use feature flags for A/B testing
- Implement gradual rollouts
- Use flags for beta features
- Track feature flag usage

## User Identification (Future)
- Identify users after authentication
- Set user properties for segmentation
- Track user properties changes
- Use consistent user identification

## Best Practices (Future)
- Don't track sensitive information
- Use consistent event naming conventions
- Implement proper error handling
- Test analytics in development mode
- Respect user privacy and GDPR compliance

## Current Status
- PostHog integration is planned for future implementation
- No PostHog dependencies are currently installed
- Analytics tracking is not active
- Focus on core functionality first

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adeveli101) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
