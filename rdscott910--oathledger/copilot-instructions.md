## oathledger

> Crucial security rules and common AI hallucinations to avoid.

# Security
- Never log tokens or PII (Personally Identifiable Information) to the console.
- Ensure all Supabase database queries respect Row Level Security (RLS) policies.

# Common AI "Gotchas" (AVOID THESE AT ALL COSTS)
- **Expo vs. Bare React Native:** This is an Expo Managed app. NEVER suggest running `npx pod-install` or modifying files inside `/ios` or `/android`. All native modules use Expo Config Plugins.
- **Prisma in the Frontend:** NEVER import `@prisma/client` inside the React Native frontend.
- **FlashList Resizing:** If `@shopify/flash-list` throws a layout warning, DO NOT switch back to `FlatList`. Fix the `estimatedItemSize` property.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rdscott910) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
