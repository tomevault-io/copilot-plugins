## server-actions

> Server Action, service and tenancy contract


# Server boundary

A Server Action is a public HTTP endpoint. Middleware and layout guards do not
protect it — it can be invoked directly with any payload.

## Actions

Always build them with `defineAction` or `defineWorkspaceAction`
(`src/lib/action.ts`). They run, in order: session check → Zod parse →
membership lookup → handler → error mapping.

```ts
"use server";

export const renameProjectAction = defineWorkspaceAction({
  input: renameProjectSchema,          // must contain workspaceId
  handler: async ({ input, ctx }) => {
    assertCanWrite(ctx.role);          // role rules stay explicit
    await renameProject({ ...input, workspaceId: ctx.workspaceId });
    revalidatePath("/projects");
  },
});
```

- Scope every write with `ctx.workspaceId` and `ctx.userId`, never with an id
  taken from the payload.
- Actions return `ActionResult`; they do not throw at the UI and never leak a
  stack trace.
- Map a domain `code` to specific copy through the `errors` option instead of
  branching on `instanceof` in the handler.

## Services

- The only layer that calls Prisma.
- Load what the domain needs, call the pure rule, persist the result. Never
  re-implement a rule that belongs in `domain/`.
- Scope reads *and* writes by `workspaceId` — that filter, not the action
  wrapper alone, is what makes a cross-tenant id return "not found".
- `import "server-only"` at the top.

## Performance

- Start independent I/O together (`Promise.all`); do not chain awaits that have
  no dependency.
- Wrap per-request reads that several components need in `React.cache`.
- No cross-request caching of tenant data. A revoked membership must take
  effect on the next request.

---
> Source: [krivoox/agent-stack-template](https://github.com/krivoox/agent-stack-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-16 -->
