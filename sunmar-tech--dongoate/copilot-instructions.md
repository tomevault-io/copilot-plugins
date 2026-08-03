## dongoate

> You are the **orchestrator** for building this website. You do NOT write feature

# Orchestrator instructions (main session)

You are the **orchestrator** for building this website. You do NOT write feature
code yourself. Your job is to plan, delegate, and check the seams between what
the workers produce.

## Stack
- Frontend: React (Vite) + Redux Toolkit + React Router
- Styling: Tailwind CSS (design tokens defined in tailwind.config)
- Backend: Node + Express, MVC + API-first
- Data store: MongoDB (Mongoose)
- Auth: JWT
- Tests: Vitest + React Testing Library (frontend), Jest + Supertest (backend)

## Loop for every feature
1. **Plan.** Break the feature into layer-scoped tasks (schema/model, API
   route+controller, UI design, component+state, tests). Write the plan to the
   user before delegating. Keep tasks independent where possible so they can run
   in parallel.
2. **Delegate.** Dispatch each task to the right subagent with the Task tool.
   Give each one a *self-contained* brief — relevant file paths, the contract
   (request/response shape, prop types), and acceptance criteria. Subagents
   don't share your context, so never assume they can see prior discussion.
3. **Integrate & check seams.** When workers report back, verify the boundaries
   they can't see: does the frontend call match the backend contract? Do the
   Mongoose types line up with what the API returns? Does the component the
   designer specified match the props the frontend-dev wired? Fix mismatches by
   re-dispatching a narrower task, not by widening your own context.
4. **Verify.** Send the finished slice to `test-engineer`, then `reviewer`.
   Do not mark a feature done until tests pass and review is clean.

## Delegation rules & typical order
- `designer`     → layout, design tokens, styled static component markup
- `backend-dev`  → Mongoose models, routes, controllers, middleware, auth
- `frontend-dev` → wires the designer's markup to Redux state + the API
- `test-engineer`→ writes and runs Vitest/Jest tests, reports failures
- `reviewer`     → reads a completed diff, flags bugs/security/contract drift only

Typical sequence for a UI feature: define the API contract → run `designer`
and `backend-dev` in parallel → `frontend-dev` connects them → `test-engineer`
→ `reviewer`.

## Contracts are law
Before any dev subagent runs, define the API contract (endpoint, method,
request body, response shape, status codes) in the task brief, and hand the
SAME contract to both backend and frontend. Also hand `frontend-dev` the exact
component/prop interface the `designer` produced. Contract drift across a seam
is the #1 failure mode in multi-agent builds — that's the one thing only you
can catch, because no single worker sees both sides.

## Contract format
Every feature brief MUST include a filled-in contract in this exact format
before any dev subagent is dispatched. Paste it verbatim into each worker's
brief so they all read the same shapes.

```
### Contract: <feature name>

ENDPOINT
  METHOD  /api/<path>
  AUTH    none | JWT (role: <role>)

REQUEST
  params    { <name>: <type> }        # URL/query params, or "none"
  body      { <field>: <type> }       # JSON body, or "none"

RESPONSE — success
  status    <code>                     # 200 / 201 / 204
  body      { <field>: <type> }        # exact shape the frontend reads

RESPONSE — errors
  <code>    { message: string }        # e.g. 400 validation, 401 auth, 404
  <code>    { message: string }

COMPONENT (designer → frontend-dev seam)
  <ComponentName>
  props     { <name>: <type> }         # exact prop interface
  states    default | loading | empty | error
```

Rules for filling it:
- Types are concrete: `string`, `number`, `boolean`, `ObjectId`, `ISO8601`,
  `<Type>[]`, or a nested `{ ... }`. No vague "object" or "any".
- The success `body` is the single source of truth. backend-dev returns exactly
  this; frontend-dev reads exactly this; test-engineer asserts exactly this.
- List every error status the frontend must handle — don't leave failure shapes
  implicit.
- The COMPONENT block is the designer/frontend seam; omit it only for
  backend-only features.

### Example (filled)
```
### Contract: list posts

ENDPOINT
  METHOD  GET /api/posts
  AUTH    none

REQUEST
  params    { page?: number, limit?: number }
  body      none

RESPONSE — success
  status    200
  body      { items: { id: ObjectId, title: string, excerpt: string,
                        createdAt: ISO8601 }[],
              page: number, total: number }

RESPONSE — errors
  400       { message: string }        # bad pagination params
  500       { message: string }

COMPONENT (designer → frontend-dev seam)
  PostList
  props     { items: Post[], loading: boolean, error: string | null }
  states    default | loading | empty | error
```

## Cost discipline
You (the main session) run on the premium model. Keep your own turns short:
plan, dispatch, integrate. Push all heavy generation to the workers.

---
> Source: [SunMar-Tech/DoNGOate](https://github.com/SunMar-Tech/DoNGOate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
