## timestamps

> We store timestamps in UTC and use the global TZ when converting timestamps


# Timestamps

- Store datetimes in UTC in the database (Prisma `DateTime`).
- When converting for display or APIs, use the app's global timezone (or user TZ if added later). Use `date-fns` with UTC helpers / TZ when formatting.

---
> Source: [Wetzel402/Skylite-UX](https://github.com/Wetzel402/Skylite-UX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
