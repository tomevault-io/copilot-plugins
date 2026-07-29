## nextacular

> validateSession,

# Conventions

Patterns enforced across this codebase. CI catches the mechanical ones (lint, typecheck, format); the rest live in PR review.

If you're unsure about a non-mechanical rule, find the closest existing file and follow its shape — symmetry across the codebase is a feature.

## TypeScript

- `strict: true` and `noUncheckedIndexedAccess: true` are non-negotiable.
- No `any` in source files. If a library returns `unknown`, narrow with a type predicate or a Zod schema; don't pass it through.
- No `@ts-ignore` without a same-line comment that names the underlying bug or upstream issue. Prefer `@ts-expect-error` so the suppression becomes a CI failure once the upstream is fixed.
- Imports use the `@/...` aliases declared in `tsconfig.json`. Never use deep relative imports (`../../../`).
- Prefer `import type { … }` for type-only imports; bundlers and `verbatimModuleSyntax` strip them automatically.

## File organization

Every source file has exactly one of the following responsibilities. If a new file doesn't fit, the convention is wrong, not the file — open a PR to extend this list.

| Directory                     | What lives here                                                               | Imports from           |
| ----------------------------- | ----------------------------------------------------------------------------- | ---------------------- |
| `prisma/services/`            | Database access functions. Only place that imports `@prisma/client` directly. | server only            |
| `src/lib/server/`             | Server-only utilities (auth, mail, Stripe, raw body, authorization).          | server only            |
| `src/lib/common/`             | Code safe to import from either side (e.g. the `apiFetch` helper).            | either                 |
| `src/lib/client/`             | Browser-only utilities (Clipboard helper, Stripe.js).                         | client only            |
| `src/config/api-validation/`  | One Zod schema per endpoint, re-exported from `index.ts`.                     | either                 |
| `src/config/email-templates/` | `html()` + `text()` generators, one file per email.                           | server only            |
| `src/components/`             | Presentational React components. No data fetching.                            | client                 |
| `src/sections/`               | Landing-page composition (Hero, Features, Pricing, …).                        | client                 |
| `src/layouts/`                | Page chrome: `AccountLayout`, `AuthLayout`, `LandingLayout`, `PublicLayout`.  | client                 |
| `src/hooks/data/`             | One `useSWR` wrapper per resource.                                            | client                 |
| `src/providers/`              | React Context providers.                                                      | client                 |
| `src/pages/`                  | Routes (Pages Router).                                                        | either, route by route |
| `src/types/`                  | Ambient `.d.ts` files (module augmentation only).                             | n/a                    |

The server / client boundary is a hard rule: importing `src/lib/server/*` from anything under `src/components/`, `src/sections/`, or a client-only branch of `src/pages/` will leak server secrets into the bundle. Treat it like a type error.

## API routes

Every API route follows the same skeleton. The canonical example is `src/pages/api/workspace/[workspaceSlug]/name.ts`:

```ts
import type { NextApiRequest, NextApiResponse } from 'next';

import {
  updateWorkspaceNameSchema,
  validateSession,
} from '@/config/api-validation';
import { requireWorkspaceOwner } from '@/lib/server/authorization';
import { parseBody } from '@/lib/server/validate';
import { updateName } from '@/prisma/services/workspace';

const handler = async (req: NextApiRequest, res: NextApiResponse) => {
  const { method } = req;

  if (method === 'PUT') {
    const session = await validateSession(req, res);
    const body = parseBody(updateWorkspaceNameSchema, req.body, res);
    if (!body) return;
    const { workspaceSlug } = req.query as { workspaceSlug: string };
    const workspace = await requireWorkspaceOwner(
      req,
      res,
      session,
      workspaceSlug
    );
    if (!workspace) return;
    updateName(
      session.user.userId,
      session.user.email,
      body.name,
      workspaceSlug
    )
      .then((name) => res.status(200).json({ data: { name } }))
      .catch((error: Error) =>
        res.status(404).json({ errors: { error: { msg: error.message } } })
      );
  } else {
    res
      .status(405)
      .json({ errors: { error: { msg: `${method} method unsupported` } } });
  }
};

export default handler;
```

Notes:

- The method switch is a single `if / else if / else` chain. We don't use a framework like next-connect.
- `validateSession` ALWAYS comes first.
- `parseBody` comes next on routes that take a request body.
- The authorization helper comes next on routes that touch workspace data.
- Services return the new state; the route serialises it to `{ data: … }`.
- Errors that originate in services (`throw new Error('Unable to find workspace')`) are caught with `404 + { errors: { error: { msg } } }`. Don't change that shape — the client toaster reads it.

## Error response shape

Both validation failures and business errors use the same envelope:

```json
{ "errors": { "<field>": { "msg": "<message>" } } }
```

- For Zod failures, the `<field>` is the dotted path (`members.0.email`).
- For business errors, the `<field>` is just `error`.
- For 405s, the `<field>` is `error` and the message is `"<METHOD> method unsupported"`.

The client side reads it with `Object.keys(response.errors).forEach((k) => toast.error(response.errors[k].msg))` — keep the toaster compatibility in mind when adding new error paths.

## Validation

- Server: Zod schemas in `src/config/api-validation/`, parsed via `parseBody`.
- Client: `validator/lib/...` for instant pre-submit feedback (matches one-shot DOM events).
- Pages must NOT trust client validation as the source of truth. Zod parsing on the server is the contract.

When you add a new endpoint that takes a body, you almost always need to add a new file under `src/config/api-validation/` and re-export the schema from `index.ts`. Keep one schema per file — when a route changes shape, the diff lives in one place.

## Authorization

See [`ARCHITECTURE.md` → "Authorization on workspace-scoped routes"](ARCHITECTURE.md#authorization-on-workspace-scoped-routes) for the rule. To summarise:

- Authenticated-only reads: `validateSession` alone is fine (e.g. `/api/workspaces` returns the current user's workspaces).
- Workspace-scoped reads (members, domains, settings): `requireWorkspaceMember`.
- Workspace-scoped writes (rename, invite, delete, change domain): `requireWorkspaceOwner`.
- Operations targeting a specific `memberId` without the workspace slug in the URL: `requireMemberInOwnedWorkspace`.

If you can't decide between Owner and Member, default to **Owner** and loosen later. Tightening is harder than loosening.

## Pages

- Files under `src/pages/account/**` use `AccountLayout`. Inside that layout, `useSession` handles the unauthenticated redirect.
- Files under `src/pages/auth/**` use `AuthLayout`.
- Landing and payment-status pages use `LandingLayout` / `PublicLayout`.
- Subdomain / custom-domain tenant content lives in `src/pages/_sites/[site]/`. The middleware rewrites those requests automatically; you never link to `/_sites/...` from a page.

### `getServerSideProps`

- Always type the return: `GetServerSideProps<MyProps>`.
- Get the session with `getSession(context)` from `next-auth/react`.
- If the data fetch requires a session and there is none, return `{ redirect: { destination: '/auth/login', permanent: false } }` rather than letting downstream code crash.
- Narrow `context.params?.foo` from `string | string[] | undefined` before passing it to a service — Prisma queries with `undefined` segments throw.

## React components

- Functional components with destructured-default props. No `defaultProps` (deprecated on function components in React 18.3+).
- Compound components (e.g. `Card`, `Content`) use `React.FC` for sub-components with named `displayName` assignments — see `src/components/Card/index.tsx` for the canonical pattern. This both typechecks cleanly and gives readable names in React DevTools.
- Event handler types: `ChangeEvent<HTMLInputElement>`, `MouseEvent<HTMLButtonElement>`, `KeyboardEvent<...>`. Don't use the bare `React.SyntheticEvent`.
- `children` is typed as `ReactNode` (or `ReactNode | undefined` if optional).

## SWR

- One hook per resource, in `src/hooks/data/`.
- Pattern: `(slug) => useSWR<{ data?: { … } }>(apiRoute)`; return `{ data, isLoading: !error && !data, isError: error }`.
- The fetcher and `onError` come from `src/config/swr/index.ts` and are wired in `_app.tsx`. Don't pass a custom `fetcher` to individual hooks.
- After a mutation, invalidate with `mutate(apiRoute)` from `'swr'`. The settings/domain page is the only example today.

## i18n

- All user-facing strings go through `useTranslation()`.
- Keys live in `src/messages/en.json`. Use dot notation (`settings.workspace.delete`).
- We only ship English today. When adding a new locale, mirror the file structure.
- Don't interpolate untrusted strings into translation values — use placeholders.

## Naming

| Kind                | Convention           | Example                          |
| ------------------- | -------------------- | -------------------------------- |
| Files               | kebab-case           | `api-validation/update-email.ts` |
| Component files     | PascalCase           | `components/Card/index.tsx`      |
| Type names          | PascalCase           | `WorkspaceForDomain`             |
| Zod schemas         | camelCase + `Schema` | `updateWorkspaceNameSchema`      |
| Inferred body types | PascalCase + `Body`  | `UpdateWorkspaceNameBody`        |
| Service functions   | camelCase verbs      | `inviteUsers`, `getOwnWorkspace` |
| API response data   | `{ data: { … } }`    | `res.json({ data: { name } })`   |
| API errors          | `{ errors: { … } }`  | see "Error response shape"       |
| Hooks               | `useThing`           | `useWorkspaces`, `useMembers`    |
| Boolean props       | `is…` / `has…`       | `isLoading`, `isTeamOwner`       |

## Commits and PRs

- Commit messages: imperative, scope-prefixed (`fix(security): ...`, `chore(b5b): ...`, `feat: ...`). The body describes **why** the change is needed; the **what** is in the diff.
- One logical change per PR. If your PR description has to use the word "and" three times, split it.
- For user-facing changes, add a note to `CHANGELOG.md` under `[Unreleased]`.
- Never bypass CI hooks. If a hook fails, fix the underlying issue.

## What we don't do

- We don't use class components.
- We don't use Redux / Zustand / Jotai. React state + SWR + one Context (`WorkspaceProvider`) is enough.
- We don't add helper libraries for things that are one-liners (e.g. lodash, ramda).
- We don't expose server secrets behind `NEXT_PUBLIC_*`. Anything with that prefix is bundled into the client JavaScript.
- We don't use `console.log` outside the dev-mode mail fallback. ESLint warns on it.
- We don't write `// TODO` comments without an issue number. If it's worth tracking, file it.

---
> Source: [nextacular/nextacular](https://github.com/nextacular/nextacular) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
