## b2b-boilerplate

> - If the API route has to be authenticated, use [withAuthRequired.ts](mdc:src/lib/auth/withAuthRequired.ts). Example: [route.ts](mdc:src/app/api/app/me/route.ts)

- If the API route has to be authenticated, use [withAuthRequired.ts](mdc:src/lib/auth/withAuthRequired.ts). Example: [route.ts](mdc:src/app/api/app/me/route.ts)
- If the API route has to be super user authenticated, use [withSuperAdminAuthRequired.ts](mdc:src/lib/auth/withSuperAdminAuthRequired.ts). Example: [route.ts](mdc:src/app/api/super-admin/users/route.ts)
- If you need current user plan access in route, or quota checks, use context from [withAuthRequired.ts](mdc:src/lib/auth/withAuthRequired.ts) to get current plan and quotas, Refer to Db for schema: [plans.ts](mdc:src/db/schema/plans.ts)
- For organization level authentication, use [withOrganizationAuthRequired.ts](mdc:src/lib/auth/withOrganizationAuthRequired.ts) eg: [route.ts](mdc:src/app/api/app/organizations/current/members/route.ts)
- To get current organization use `const currentOrganization = await context.session.organization` in [withOrganizationAuthRequired.ts](mdc:src/lib/auth/withOrganizationAuthRequired.ts)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ReeseAstor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
