## newportfolio-1

> Always act like you’re shipping to production. Write clean, defensive, TypeScript-first code for a Next.js App Router project using Contentful + Contentful Live Preview + Ninetailed Personalization.


Always act like you’re shipping to production. Write clean, defensive, TypeScript-first code for a Next.js App Router project using Contentful + Contentful Live Preview + Ninetailed Personalization.

This repo’s architecture has a few non-obvious “rules of the road” (locale middleware, dynamic page composition, Frames→Things rendering, preview/personalization), so follow the discipline below.

Contentful (data is untrusted)
Assume missing/partial content
Any field, link, array, or asset can be null, undefined, empty, unpublished, or missing for a locale.
Never assume entry.fields.* exists without guarding.
Queries must be reusable + typed
Wrap Contentful calls in small reusable functions (getEntries, getAllPageSlugs, etc.), with typed skeletons and clear inputs.
Prefer returning []/null with explicit logging over throwing inside rendering paths.
Fail safely
Prefer “no content” UI / fallbacks over crashing the route.
Missing slug/content should render notFound() (not a 500).
Locales are first-class
Always pass locale explicitly to Contentful queries.
Expect locale mismatches because runtime locales come from lib/locales.json (build-time snapshot).
Frames → Things (composition contract)
This project has two layers of dynamic composition:

Page → Sections: landingPage.fields.sections[] renders content-type components via a component map.
Page → Frames → Things: landingPage.fields.frames[] renders frame, and each frame.fields.things[] renders “things” (e.g. image wrappers, callouts, blog posts).
Rules:

Never render “things” ad-hoc. Use a single resolver/mapper per layer:
sections must go through a contentTypeId → component map.
things must go through a thingTypeId → component map.
Unknown types must degrade gracefully
If a new content type appears in Contentful but no component exists, render a safe placeholder (and log a warning), not a crash.
Keep type boundaries clear
Treat cross-layer entries as unknown at boundaries and narrow safely (guards, discriminants like sys.contentType.sys.id).
Personalization (must never break UX)
Assume user/profile is missing or invalid
Decisioning can fail or return no match.
Always provide a safe default
Baseline variant must render even if Ninetailed mapping/decisioning fails.
No “personalization-only” rendering paths
Personalization enhances; it never blocks core content from rendering.
Unified Tracking (mandatory)
All component-view tracking must be routed through one tiny tracking client abstraction so we can:

Track views in Ninetailed today.
Add future sinks (e.g. Klaviyo, others) without touching every component.
Rules:

Never call Ninetailed (or any vendor SDK) directly inside feature components for “view” tracking.
Components should only call something like:
tracking.trackComponentView({ id, type, locale, route, ... })
The tracking client is responsible for:
fan-out to Ninetailed + any future providers
deduping/throttling
being safe when providers aren’t configured
never throwing (tracking must be non-fatal)
Next.js discipline (App Router correctness)
Server vs client separation is strict
Keep Contentful Delivery/Preview fetching in server components/route handlers unless there’s a specific reason.
Client components may use Live Preview hooks and personalization components, but should receive serializable props only.
Caching/perf awareness
Be intentional about revalidate, fetchCache, and forced dynamic rendering.
Don’t accidentally disable caching globally unless you’re solving a real correctness issue.
Edge-case mentality (assume the bad path)
Plan for:

network/API errors, Contentful rate limits
invalid slugs and missing content
slow asset processing / long operations (like seeding)
locale negotiation mismatch
partial linked entries due to insufficient include
If anything fails, the site should still render something reasonable.

Production quality bar
Strict typing, minimal any, explicit return types where it clarifies behavior.
Small functions, clear naming, single responsibility.
No secrets leakage
Never move server secrets into NEXT_PUBLIC_*.
Treat tokens and user-provided credentials as sensitive (don’t log/store them).
Predictable over clever
Prefer straightforward, testable code paths.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/contentful) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
