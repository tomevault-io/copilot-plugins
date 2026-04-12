## tt-mem

> Server-side logic, including database interactions (fetching, creating, updating, deleting data), is implemented using Next.js Server Actions.

# Data Fetching & Mutations (Server Actions)

Server-side logic, including database interactions (fetching, creating, updating, deleting data), is implemented using Next.js Server Actions.

- **Location:** Actions are defined in files within `[app/actions/](mdc:app/actions)`, typically marked with `'use server';`.
    - Example: `[app/actions/decks.ts](mdc:app/actions/decks.ts)`, `[app/actions/cards.ts](mdc:app/actions/cards.ts)`, `[app/actions/reviewEvents.ts](mdc:app/actions/reviewEvents.ts)`.
- **Database Interaction:** Actions use the MongoDB client configured in `[app/lib/db.ts](mdc:app/lib/db.ts)` to interact with collections (e.g., `decks`, `cards`, `review_events`).
- **Data Mapping:** MongoDB documents often use `ObjectId` for `_id`. The `mapMongoId` helper in `[app/lib/utils.ts](mdc:app/lib/utils.ts)` is used to convert `_id` to a string `id` within the server actions before returning data to the client. Specific mapping functions (e.g., `mapCardDocument` in `[app/actions/cards.ts](mdc:app/actions/cards.ts)`) may exist.
- **Client Usage:** [TanStack Query (React Query)](mdc:https:/tanstack.com/query/latest) hooks in `[app/hooks/queryHooks.ts](mdc:app/hooks/queryHooks.ts)` call these server actions to fetch data or perform mutations.
    - **New Fetch Hooks:** `useCardsForReview`, `useCard`, `useLatestReviewForCard`.
    - **New Mutation Hooks:** (No new mutation hooks added recently, but existing ones like `useUpdateCardMutation`, `useCreateReviewEventMutation` are used).
- **Authentication:** Actions typically verify user authentication using the `verifyAuthToken` helper from `[app/lib/auth.ts](mdc:app/lib/auth.ts)` before performing protected operations. Failed authentication usually results in `{ success: false, message: 'Unauthorized' }` which is handled globally (see `auth-error-handling.mdc`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tylerthecoder)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tylerthecoder)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
