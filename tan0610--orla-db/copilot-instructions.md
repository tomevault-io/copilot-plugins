## orla-db

> **Orla3** is currently in **backend integration phase** with full production infrastructure in place: PostgreSQL on Google Cloud SQL, Bunny.net Stream for video, Google Cloud Run for API, Vercel for frontend, and Stripe for payments.

# Context

**Orla3** is currently in **backend integration phase** with full production infrastructure in place: PostgreSQL on Google Cloud SQL, Bunny.net Stream for video, Google Cloud Run for API, Vercel for frontend, and Stripe for payments.

## Current Work Focus
- **Backend Integration:** ✅ In Progress - Connecting to production infrastructure.
- **Real Adapter Implementation:** ✅ Completed - orlaApiAdapter.ts fully implements all API endpoints for `/v1` backend.
- **Video Infrastructure:** ✅ Bunny.net Stream configured - 119+ PoPs, <10ms EU latency, ~70% cheaper than Mux.
- **Payments:** ✅ Stripe integration - Payment at booking pending, release on download.
- **Pagination Support:** ✅ Completed - Cursor-based pagination types and useInfiniteQuery hooks ready.
- **CI Pipeline:** ✅ Configured - GitHub Actions workflow with full quality gates.
- **System Quality:** Strict TypeScript enforcement and code quality standards maintained.

## Recent Changes (December 2025)
- **Backend Integration Phase:** Transitioned from mock-only to production infrastructure.
- **Video Provider:** Switched from Mux to Bunny.net Stream for 70% cost savings (~$8K vs ~$27K/month at scale).
- **Watermarking Correction:** Portfolio videos NOT watermarked; deliverable previews in bookings/delivered ARE watermarked.
- **Pagination Types:** Added `PaginationParams`, `PaginatedResponse<T>`, and `ListParams` to domain.ts
- **API Interface Updates:** Updated apiClient.ts interfaces with optional pagination parameters
- **Real Adapter Completion:** Implemented all API methods in orlaApiAdapter.ts:
  - Proposals: list, create, markViewed, respond (accept/amend/decline)
  - Bookings: listByRole, get, updateStatus, fundEscrow, startEditing, cancel, respond
  - Reviews: list, create, request, flag, getAverageRating
  - Notifications: list, markRead, markAllRead, delete, getCounts
  - Portfolio: get, uploadVideo, deleteVideo, moveVideoToFolder, createFolder, deleteFolder, setFeaturedVideo
- **Infinite Query Hooks:** Created use-infinite-data.ts with:
  - useInfiniteBookings, useInfiniteNotifications, useInfiniteReviews, useInfiniteProposals
  - Helper functions: flattenInfiniteData, getInfiniteTotal
- **CI Pipeline:** GitHub Actions workflow configured with:
  - Typecheck, Lint, Build quality gates
  - Unit tests, Smoke tests, Dashboard/Feed smoke projects
  - Full E2E regression suite
  - Artifact upload on failure

## Architecture Maturity
- **Production Infrastructure:** PostgreSQL on Cloud SQL, Bunny.net Stream, Cloud Run API, Stripe payments
- **Adapter Pattern:** Switch via `NEXT_PUBLIC_API_ADAPTER=real` for production, `mock` for development
- **Scalable Data Fetching:** useInfiniteQuery hooks enable load-more/infinite-scroll patterns
- **Cursor-Based Pagination:** Efficient pagination for large datasets (1M+ records)
- **Type Safety:** Strict TypeScript with comprehensive domain types
- **CI/CD:** Full quality gate pipeline in GitHub Actions

## Key Files
- `src/shared/types/domain.ts` - Domain types including pagination
- `src/shared/services/apiClient.ts` - API client interfaces
- `src/shared/services/adapters/orlaApiAdapter.ts` - Real adapter implementation
- `src/shared/hooks/use-infinite-data.ts` - Infinite query hooks
- `.github/workflows/ci.yml` - CI pipeline configuration

## Next Steps
- **Full Production Deployment:** Complete Cloud Run API integration and Vercel frontend deployment
- **Stripe Webhooks:** Configure payment webhooks for booking status updates
- **Bunny Webhooks:** Configure video processing webhooks for status updates
- **Performance Testing:** Load test with large datasets to validate scalability
- **Memory Bank Maintenance:** Keep documentation synchronized with code changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tan0610) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
