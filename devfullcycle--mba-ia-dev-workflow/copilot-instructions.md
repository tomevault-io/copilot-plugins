## controllers

> Use when writing or modifying Express.js controllers and route handlers. Covers procedural handler pattern, REST status codes, and error propagation.


# Controller Pattern

Controllers are individually exported **procedural functions** — never classes. Each function is a route handler that follows the flow: **extract data from request → call use case → return response**.

## Rules

1. **One function per action**: each handler is an exported named function (`export async function createUser`)
2. **No business logic**: controllers never validate rules, compute values, or access repositories directly — delegate everything to the use case
3. **Standard signature**: `(req: Request, res: Response, next: NextFunction) => Promise<void>`
4. **No silent errors**: every error must be propagated — never use empty catch blocks, never swallow exceptions with `console.log` without re-throwing, never return a generic response ignoring the error. The catch block always delegates to the error middleware via `next(error)`
5. **Correct REST status codes**:
   - `200` — successful read or update
   - `201` — resource created (POST that creates)
   - `204` — no response body (e.g., logout, delete)
   - Never return error status codes manually (4xx/5xx) — the error middleware determines those from use case exceptions
6. **Explicit data extraction**: destructure `req.body`, `req.params`, and `req.query` at the top of the handler, before calling the use case
7. **Direct response**: `res.status(xxx).json(result)` — no helpers or extra abstractions

## Handler structure

```typescript
import { Router } from "express";

const router = Router();

router.post("/<path>", async (req, res, next) => {
  try {
    const { name, email } = req.body;
    const user = await createUserUseCase.execute({ name, email });
    res.status(201).json(user);
  } catch (error) {
    next(error);
  }
});

export default router;
```

---
> Source: [devfullcycle/mba-ia-dev-workflow](https://github.com/devfullcycle/mba-ia-dev-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
