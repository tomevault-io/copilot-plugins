## state-and-forms

> TanStack Query, Zustand user store, nuqs, RHF+Zod, dayjs


# State & forms

- **TanStack Query**: server/async data and mutations (via Orval hooks).
- **Zustand**: shell prefs + **panel current user** (`/me`) — not theme.
- **nuqs**: filters and URL-serializable UI state.
- **Theme**: `next-themes` only.
- **Dates**: dayjs + locales.
- Do not mirror arbitrary Query data into Zustand; user session is the allowed exception.
- Load `/me` only inside the panel private shell into the user store.
- Forms: React Hook Form + Zod; inline field errors; use `toast` from `@/components/ui/toast` for global feedback.
- Logout: `useUserStore.logout(queryClient)` — `POST /auth/logout` then clear cookie + user + Query cache (even if API fails).
- Navigation: typed `routes.*()` helpers only.

---
> Source: [soheilhasanjani/contently](https://github.com/soheilhasanjani/contently) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-03 -->
