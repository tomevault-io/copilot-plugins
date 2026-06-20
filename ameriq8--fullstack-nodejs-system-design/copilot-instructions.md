## fullstack-nodejs-system-design

> Adaptive full-stack system design skill for Node.js backends integrated with React, Next.js, and Vite, supporting monorepo/polyrepo architectures, multiple testing tools, and scalable production systems. Acts as a senior architect that guides developers through key decisions before generating tailored, production-ready code.


# Full-Stack Node.js System Design

A smart, adaptive skill that behaves like a principal architect: it gathers your context first, then produces the right structure, patterns, and code for your specific project — not a one-size-fits-all template.

---

## 1. When to Use This Skill

- Starting any new full-stack project with a Node.js backend
- Choosing between monorepo and polyrepo architecture
- Integrating React (Vite), Next.js (App Router), or both with a backend
- Establishing auth, API contracts, testing, and deployment strategies
- Onboarding a team to consistent, scalable conventions
- Migrating a legacy project to a modern full-stack setup

---

## 2. Interactive Architecture Detection

**Before generating any structure or code, ask the user these questions.**

```
1. What are you building?
   [ ] SaaS product        [ ] Internal dashboard
   [ ] Public API           [ ] Real-time application
   [ ] Microservices system [ ] Not sure yet

2. What frontend framework will you use?
   [ ] React (Vite)         [ ] Next.js (App Router)
   [ ] Both                 [ ] Not decided

3. Architecture preference?
   [ ] Monorepo             [ ] Polyrepo
   [ ] Not sure — recommend based on my scale

4. Team / project scale?
   [ ] Solo / hobby         [ ] Small team (2–5)
   [ ] Startup (5–20)       [ ] Enterprise / large team

5. Primary database?
   [ ] PostgreSQL / MySQL (SQL)
   [ ] MongoDB (NoSQL)
   [ ] Both
   [ ] Undecided

6. Testing preference?
   [ ] Jest                 [ ] Vitest
   [ ] Mocha + Chai         [ ] Playwright (E2E)
   [ ] Cypress (E2E)        [ ] Not sure
```

Use answers to drive every decision below. Revisit defaults only when answers are "not sure."

---

## 3. Architecture Decision Engine

### Monorepo vs Polyrepo

**Recommend Monorepo when:**
- Frontend and backend share TypeScript types (DTOs, schemas)
- One team maintains both layers
- You use Next.js (naturally co-located with its API routes)
- You want a single CI/CD pipeline
- Project scale: solo → startup

**Recommend Polyrepo when:**
- Independent teams own each service
- Services have different release cycles
- You are building true microservices with separate CI/CD
- Scale: large-scale enterprise

**Always explain the trade-off:**

> Monorepo = faster DX, shared types, simpler CI. Cost: repo grows with the project.
> Polyrepo = independent deployability, cleaner ownership. Cost: type duplication, complex CI coordination.

---

## 4. Project Structures

### Option A — Monorepo (Turborepo + pnpm)

**When to use:** Shared types matter. One team. Next.js or React + backend together.

```
apps/
├── web/              # React (Vite) or Next.js frontend
├── admin/            # Optional internal dashboard
└── api/              # Express or Fastify backend

packages/
├── shared/           # DTOs, Zod schemas, ApiResponse types
├── ui/               # Shared component library
├── api-client/       # Typed fetch/axios SDK
└── config/           # ESLint, TSConfig, env schemas

turbo.json
pnpm-workspace.yaml
```

```yaml
# pnpm-workspace.yaml
packages:
  - "apps/*"
  - "packages/*"
```

```json
// turbo.json
{
  "$schema": "https://turbo.build/schema.json",
  "pipeline": {
    "build": { "dependsOn": ["^build"], "outputs": ["dist/**"] },
    "dev": { "cache": false, "persistent": true },
    "test": { "dependsOn": ["build"] },
    "lint": {}
  }
}
```

### Option B — Polyrepo

**When to use:** Independent teams. Different deployment cycles.

```
backend/          # Node.js API (standalone repo)
frontend/         # React or Next.js (standalone repo)
shared-types/     # Optional: published npm package for DTOs
```

Each repo has its own `package.json`, CI pipeline, and deploy target. Sync types via a versioned npm package (`@myorg/shared@1.2.0`).

### Option C — Microservices

**When to use:** High scale, independent scaling requirements, distributed teams.

```
services/
├── gateway/          # API gateway (routes + auth checks)
├── auth-service/     # JWT issuance, refresh, user identity
├── user-service/     # User CRUD
└── notification-service/

packages/
├── shared/           # Shared event types, error classes
└── api-client/       # Internal service clients

infra/
├── docker-compose.yml
└── nginx/
```

> For microservices, each service is an independent Fastify/Express app. Services communicate via HTTP or a message broker (Redis Pub/Sub, RabbitMQ). The gateway is the sole public entry point.

---

## 5. Backend System

### 5.1 Base Setup (Express or Fastify)

**Express:**

```typescript
// apps/api/src/app.ts
import express from "express";
import helmet from "helmet";
import { corsMiddleware } from "./config/cors";
import { sanitize } from "./middleware/sanitize";
import { requestLogger } from "./middleware/logger";
import { errorHandler } from "./middleware/error-handler";
import { apiV1Router } from "./api/v1/routes";

const app = express();

app.use(helmet());
app.use(corsMiddleware);
app.use(express.json({ limit: "10mb" }));
app.use(sanitize);
app.use(requestLogger);

app.use("/api/v1", apiV1Router);
app.use(errorHandler);

export default app;
```

**Fastify (alternative):**

```typescript
// apps/api/src/app.ts
import Fastify from "fastify";
import helmet from "@fastify/helmet";
import cors from "@fastify/cors";
import { userRoutes } from "./api/v1/routes/user.routes";

export const buildApp = async () => {
  const app = Fastify({ logger: true });
  await app.register(helmet);
  await app.register(cors, { origin: process.env.ALLOWED_ORIGINS?.split(",") });
  await app.register(userRoutes, { prefix: "/api/v1" });
  return app;
};
```

### 5.2 Layered Architecture

```
apps/api/src/
├── api/v1/
│   ├── routes/           # Route registration
│   ├── controllers/      # HTTP in/out only
│   ├── services/         # Business logic
│   └── repositories/     # DB access
├── middleware/           # auth, error, logger, sanitize
├── config/               # cors, swagger, env
└── utils/                # errors, response helpers
```

### 5.3 Standard API Response Contract

```typescript
// packages/shared/src/api-response.ts
export interface ApiResponse<T> {
  status: "success" | "error";
  data?: T;
  message?: string;
  errors?: FieldError[];
  pagination?: PaginationMeta;
}

export interface FieldError { field: string; message: string; }

export interface PaginationMeta {
  page: number; limit: number; total: number; pages: number;
}
```

```typescript
// apps/api/src/utils/response.ts
import { Response } from "express";
import type { ApiResponse, PaginationMeta } from "@myapp/shared";

export const ok = <T>(res: Response, data: T) =>
  res.json({ status: "success", data } satisfies ApiResponse<T>);

export const created = <T>(res: Response, data: T) =>
  res.status(201).json({ status: "success", data } satisfies ApiResponse<T>);

export const paginated = <T>(res: Response, data: T[], pagination: PaginationMeta) =>
  res.json({ status: "success", data, pagination } satisfies ApiResponse<T[]>);
```

### 5.4 Query Helpers (Pagination / Filtering / Sorting)

```typescript
// apps/api/src/utils/query.ts
import { Request } from "express";

export interface QueryOptions {
  page: number; limit: number;
  sortBy: string; sortOrder: "ASC" | "DESC";
  filters: Record<string, string>;
}

export const parseQuery = (req: Request): QueryOptions => ({
  page: Math.max(1, parseInt(req.query.page as string) || 1),
  limit: Math.min(100, parseInt(req.query.limit as string) || 20),
  sortBy: (req.query.sortBy as string) || "created_at",
  sortOrder: req.query.sortOrder === "ASC" ? "ASC" : "DESC",
  filters: Object.fromEntries(
    Object.entries(req.query).filter(([k]) =>
      !["page", "limit", "sortBy", "sortOrder"].includes(k)
    )
  ) as Record<string, string>,
});
```

### 5.5 GraphQL (Optional Module)

If `graphql` is selected:

```typescript
// apps/api/src/graphql/schema.ts
import { buildSchema } from "graphql";

export const schema = buildSchema(`
  type User { id: ID! name: String! email: String! }
  type Query { user(id: ID!): User users: [User!]! }
  type Mutation { createUser(name: String!, email: String!): User! }
`);
```

Mount alongside REST:

```typescript
import { graphqlHTTP } from "express-graphql";
app.use("/graphql", authenticate, graphqlHTTP({ schema, rootValue: resolvers, graphiql: true }));
```

---

## 6. Shared Types System

```
packages/shared/src/
├── api-response.ts
├── dtos/
│   ├── user.dto.ts
│   └── auth.dto.ts
├── schemas/
│   ├── user.schema.ts     # Zod — reused in both backend validation and frontend forms
│   └── auth.schema.ts
└── index.ts
```

```typescript
// packages/shared/src/dtos/user.dto.ts
export interface UserDTO {
  id: string; name: string; email: string;
  role: "admin" | "user"; createdAt: string;
}
export interface CreateUserDTO { name: string; email: string; password: string; }
export interface UpdateUserDTO { name?: string; email?: string; }
```

```typescript
// packages/shared/src/schemas/user.schema.ts
import { z } from "zod";

export const createUserSchema = z.object({
  name: z.string().min(1),
  email: z.string().email(),
  password: z.string().min(8),
});

export const updateUserSchema = createUserSchema.partial().omit({ password: true });
export type CreateUserInput = z.infer<typeof createUserSchema>;
export type UpdateUserInput = z.infer<typeof updateUserSchema>;
```

**Backend uses Zod for validation. Frontend uses the same schema for form validation (React Hook Form + Zod resolver). Zero duplication.**

---

## 7. Frontend Integration

### 7a. React (Vite) — SPA Architecture

**Project structure:**

```
apps/web/src/
├── pages/              # Route-level components
├── components/         # Shared UI
├── contexts/           # AuthContext, etc.
├── hooks/              # useUsers, useAuth, useSocket
├── lib/                # queryClient, api-client instance
└── router.tsx          # React Router v6
```

**React Query setup:**

```typescript
// apps/web/src/lib/query-client.ts
import { QueryClient } from "@tanstack/react-query";
import { toast } from "sonner";
import { ApiError } from "@myapp/api-client";

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60_000,
      retry: (n, e) => e instanceof ApiError && e.status >= 500 && n < 2,
    },
  },
});

queryClient.getMutationCache().config.onError = (err) => {
  if (err instanceof ApiError) toast.error(err.message);
};
```

**Route setup with protected routes:**

```typescript
// apps/web/src/router.tsx
import { createBrowserRouter } from "react-router-dom";
import { ProtectedRoute } from "./components/ProtectedRoute";

export const router = createBrowserRouter([
  { path: "/login", element: <LoginPage /> },
  {
    element: <ProtectedRoute />,
    children: [
      { path: "/dashboard", element: <DashboardPage /> },
      { path: "/users", element: <UsersPage /> },
    ],
  },
]);
```

### 7b. Next.js (App Router) — SSR / ISR / RSC

**Project structure:**

```
apps/web-next/
├── app/
│   ├── (auth)/
│   │   └── login/page.tsx
│   ├── (protected)/
│   │   ├── layout.tsx        # Auth check
│   │   └── dashboard/page.tsx
│   └── api/                  # Route handlers (BFF layer)
├── middleware.ts              # Auth gating
└── lib/
    ├── server-api.ts          # Server-side fetch helpers
    └── query-client.ts        # Client-side TanStack Query
```

**Server Component data fetch:**

```typescript
// app/(protected)/users/page.tsx
import { cookies } from "next/headers";
import type { UserDTO } from "@myapp/shared";

async function fetchUsers(): Promise<UserDTO[]> {
  const token = (await cookies()).get("accessToken")?.value;
  const res = await fetch(`${process.env.API_URL}/api/v1/users`, {
    headers: { Authorization: `Bearer ${token}` },
    next: { revalidate: 60 },
  });
  if (!res.ok) throw new Error("Failed to fetch users");
  const { data } = await res.json();
  return data.users;
}

export default async function UsersPage() {
  const users = await fetchUsers();
  return <UserList users={users} />;
}
```

**Route Handler (BFF / proxy):**

```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from "next/server";

export async function GET(req: NextRequest) {
  const token = req.cookies.get("accessToken")?.value;
  const res = await fetch(`${process.env.API_URL}/api/v1/users`, {
    headers: { Authorization: `Bearer ${token}` },
  });
  return NextResponse.json(await res.json());
}
```

**Auth middleware:**

```typescript
// middleware.ts
import { NextRequest, NextResponse } from "next/server";

const PUBLIC = ["/login", "/register"];

export function middleware(req: NextRequest) {
  const token = req.cookies.get("accessToken")?.value;
  const isPublic = PUBLIC.some(p => req.nextUrl.pathname.startsWith(p));
  if (!token && !isPublic)
    return NextResponse.redirect(new URL("/login", req.url));
  return NextResponse.next();
}

export const config = { matcher: ["/((?!_next|favicon.ico).*)"] };
```

### 7c. Both React (Vite) + Next.js

- Both apps import from `@myapp/api-client` and `@myapp/shared`
- Backend is a standalone Express/Fastify server (not Next.js API routes)
- Next.js uses Server Components for SSR pages; Vite app is pure SPA
- Auth cookie is set by the backend and works across both clients

---

## 8. API Client SDK

```
packages/api-client/src/
├── base-client.ts        # fetch wrapper, auto-refresh, error handling
├── modules/
│   ├── auth.module.ts
│   └── users.module.ts
└── index.ts
```

```typescript
// packages/api-client/src/base-client.ts
import type { ApiResponse } from "@myapp/shared";

export class ApiError extends Error {
  constructor(
    message: string,
    public status: number,
    public errors?: any[]
  ) { super(message); }
}

export class BaseClient {
  private token: string | null = null;
  private refreshing: Promise<string> | null = null;

  constructor(private baseURL: string) {}

  setToken(t: string | null) { this.token = t; }

  async request<T>(method: string, path: string, body?: unknown): Promise<ApiResponse<T>> {
    const res = await fetch(`${this.baseURL}${path}`, {
      method,
      headers: {
        "Content-Type": "application/json",
        ...(this.token ? { Authorization: `Bearer ${this.token}` } : {}),
      },
      credentials: "include",
      body: body ? JSON.stringify(body) : undefined,
    });

    if (res.status === 401 && path !== "/api/v1/auth/refresh") {
      await this.handleRefresh();
      return this.request<T>(method, path, body);
    }

    const data: ApiResponse<T> = await res.json();
    if (!res.ok) throw new ApiError(data.message ?? "Request failed", res.status, data.errors);
    return data;
  }

  private handleRefresh() {
    if (!this.refreshing) {
      this.refreshing = fetch(`${this.baseURL}/api/v1/auth/refresh`, {
        method: "POST", credentials: "include",
      })
        .then(r => r.json())
        .then(d => { this.token = d.data.accessToken; return d.data.accessToken; })
        .finally(() => { this.refreshing = null; });
    }
    return this.refreshing;
  }

  get = <T>(path: string) => this.request<T>("GET", path);
  post = <T>(path: string, b: unknown) => this.request<T>("POST", path, b);
  patch = <T>(path: string, b: unknown) => this.request<T>("PATCH", path, b);
  put = <T>(path: string, b: unknown) => this.request<T>("PUT", path, b);
  delete = <T>(path: string) => this.request<T>("DELETE", path);
}
```

```typescript
// packages/api-client/src/index.ts
import { BaseClient } from "./base-client";
import { AuthModule } from "./modules/auth.module";
import { UsersModule } from "./modules/users.module";

export const createApiClient = (baseURL: string) => {
  const base = new BaseClient(baseURL);
  return { _base: base, auth: new AuthModule(base), users: new UsersModule(base) };
};

// Auto-detects Vite vs Next.js env
const apiURL =
  typeof window !== "undefined"
    ? (import.meta as any).env?.VITE_API_URL ?? ""
    : process.env.NEXT_PUBLIC_API_URL ?? "";

export const apiClient = createApiClient(apiURL);
export type ApiClient = ReturnType<typeof createApiClient>;
export { ApiError } from "./base-client";
```

---

## 9. Authentication System

### Backend: JWT + Refresh Cookie

```typescript
// apps/api/src/services/auth.service.ts
import jwt from "jsonwebtoken";
import bcrypt from "bcrypt";
import type { UserRepository } from "../repositories/user.repository";
import { UnauthorizedError } from "../utils/errors";

export class AuthService {
  constructor(private users: UserRepository) {}

  async login(email: string, password: string) {
    const user = await this.users.findByEmail(email);
    if (!user || !(await bcrypt.compare(password, user.password)))
      throw new UnauthorizedError("Invalid credentials");

    return {
      accessToken: this.sign({ userId: user.id, role: user.role }, "15m"),
      refreshToken: this.sign({ userId: user.id }, "7d", true),
      user: { id: user.id, name: user.name, email: user.email, role: user.role },
    };
  }

  async refresh(token: string) {
    try {
      const { userId } = jwt.verify(token, process.env.REFRESH_SECRET!) as any;
      const user = await this.users.findById(userId);
      if (!user) throw new Error();
      return { accessToken: this.sign({ userId: user.id, role: user.role }, "15m") };
    } catch { throw new UnauthorizedError("Invalid refresh token"); }
  }

  private sign(payload: object, exp: string, refresh = false) {
    return jwt.sign(payload, refresh ? process.env.REFRESH_SECRET! : process.env.JWT_SECRET!, {
      expiresIn: exp,
    } as any);
  }
}
```

```typescript
// apps/api/src/controllers/auth.controller.ts
import { Request, Response, NextFunction } from "express";
import { AuthService } from "../services/auth.service";
import { ok } from "../utils/response";

const COOKIE = {
  httpOnly: true, secure: process.env.NODE_ENV === "production",
  sameSite: "strict" as const, maxAge: 7 * 86400 * 1000,
};

export class AuthController {
  constructor(private auth: AuthService) {}

  login = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const result = await this.auth.login(req.body.email, req.body.password);
      res.cookie("refreshToken", result.refreshToken, COOKIE);
      ok(res, { accessToken: result.accessToken, user: result.user });
    } catch (e) { next(e); }
  };

  refresh = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const token = req.cookies.refreshToken;
      if (!token) throw new Error("Missing refresh token");
      ok(res, await this.auth.refresh(token));
    } catch (e) { next(e); }
  };

  logout = (_req: Request, res: Response) => {
    res.clearCookie("refreshToken");
    ok(res, null);
  };
}
```

### Frontend: Auth Context + Protected Route

```typescript
// apps/web/src/contexts/auth.context.tsx
import { createContext, useContext, useState, useEffect, useCallback } from "react";
import { apiClient } from "@myapp/api-client";
import type { UserDTO } from "@myapp/shared";

interface AuthCtx {
  user: UserDTO | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => Promise<void>;
}

const AuthContext = createContext<AuthCtx | null>(null);

export const AuthProvider = ({ children }: { children: React.ReactNode }) => {
  const [user, setUser] = useState<UserDTO | null>(null);

  useEffect(() => {
    apiClient.auth.refresh()
      .then(({ data }) => {
        if (data) { apiClient._base.setToken(data.accessToken); }
      })
      .catch(() => {});
  }, []);

  const login = useCallback(async (email: string, password: string) => {
    const { data } = await apiClient.auth.login(email, password);
    if (data) { apiClient._base.setToken(data.accessToken); setUser(data.user); }
  }, []);

  const logout = useCallback(async () => {
    await apiClient.auth.logout();
    apiClient._base.setToken(null);
    setUser(null);
  }, []);

  return <AuthContext.Provider value={{ user, login, logout }}>{children}</AuthContext.Provider>;
};

export const useAuth = () => {
  const ctx = useContext(AuthContext);
  if (!ctx) throw new Error("useAuth must be inside AuthProvider");
  return ctx;
};
```

```typescript
// apps/web/src/components/ProtectedRoute.tsx
import { Navigate, Outlet } from "react-router-dom";
import { useAuth } from "../contexts/auth.context";

export const ProtectedRoute = ({ roles }: { roles?: string[] }) => {
  const { user } = useAuth();
  if (!user) return <Navigate to="/login" replace />;
  if (roles && user.role && !roles.includes(user.role)) return <Navigate to="/403" replace />;
  return <Outlet />;
};
```

---

## 10. Testing (Multi-Tool Support)

### Folder Structure

```
tests/
├── unit/           # Service-level tests
├── integration/    # API endpoint tests (supertest)
└── e2e/            # Playwright or Cypress
```

### Backend: Jest

```typescript
// jest.config.ts
import type { Config } from "jest";
export default {
  preset: "ts-jest",
  testEnvironment: "node",
  testMatch: ["**/tests/unit/**/*.test.ts", "**/tests/integration/**/*.test.ts"],
} satisfies Config;
```

```typescript
// tests/unit/auth.service.test.ts
import { AuthService } from "../../src/services/auth.service";
import { UnauthorizedError } from "../../src/utils/errors";
import bcrypt from "bcrypt";

const mockRepo = { findByEmail: jest.fn(), findById: jest.fn() };
const svc = new AuthService(mockRepo as any);

describe("AuthService", () => {
  it("throws on unknown email", async () => {
    mockRepo.findByEmail.mockResolvedValueOnce(null);
    await expect(svc.login("x@x.com", "pass")).rejects.toBeInstanceOf(UnauthorizedError);
  });

  it("returns tokens on valid credentials", async () => {
    mockRepo.findByEmail.mockResolvedValueOnce({
      id: "1", email: "x@x.com", role: "user", name: "X",
      password: await bcrypt.hash("pass123!", 10),
    });
    const result = await svc.login("x@x.com", "pass123!");
    expect(result.accessToken).toBeDefined();
  });
});
```

### Backend: Vitest (alternative)

```typescript
// vitest.config.ts
import { defineConfig } from "vitest/config";
export default defineConfig({ test: { environment: "node" } });
```

```typescript
// tests/unit/auth.service.test.ts (Vitest syntax)
import { describe, it, expect, vi } from "vitest";
import { AuthService } from "../../src/services/auth.service";

const mockRepo = { findByEmail: vi.fn(), findById: vi.fn() };
// rest is identical to Jest
```

### Backend: Mocha + Chai (alternative)

```typescript
// .mocharc.json
{ "require": ["ts-node/register"], "spec": "tests/**/*.test.ts" }
```

```typescript
import { expect } from "chai";
import { AuthService } from "../../src/services/auth.service";

describe("AuthService", () => {
  it("throws UnauthorizedError for unknown email", async () => {
    const svc = new AuthService({ findByEmail: async () => null } as any);
    try { await svc.login("a@b.com", "pass"); expect.fail(); }
    catch (e: any) { expect(e.statusCode).to.equal(401); }
  });
});
```

### Integration Test (Supertest)

```typescript
// tests/integration/users.test.ts
import request from "supertest";
import app from "../../src/app";
import jwt from "jsonwebtoken";

const token = jwt.sign({ userId: "1", role: "admin" }, process.env.JWT_SECRET!, { expiresIn: "1h" });

describe("GET /api/v1/users", () => {
  it("401 without token", () => request(app).get("/api/v1/users").expect(401));
  it("200 with valid token", () =>
    request(app).get("/api/v1/users").set("Authorization", `Bearer ${token}`).expect(200));
});
```

### Frontend: React Testing Library + Vitest

```typescript
// apps/web/src/__tests__/LoginForm.test.tsx
import { render, screen, fireEvent, waitFor } from "@testing-library/react";
import { LoginForm } from "../components/LoginForm";
import { vi } from "vitest";
import { apiClient } from "@myapp/api-client";

vi.mock("@myapp/api-client", () => ({
  apiClient: { auth: { login: vi.fn() } },
}));

test("shows error toast on failed login", async () => {
  (apiClient.auth.login as any).mockRejectedValueOnce(new Error("Invalid credentials"));
  render(<LoginForm />);
  fireEvent.change(screen.getByLabelText(/email/i), { target: { value: "a@b.com" } });
  fireEvent.change(screen.getByLabelText(/password/i), { target: { value: "wrong" } });
  fireEvent.click(screen.getByRole("button", { name: /login/i }));
  await waitFor(() => expect(screen.getByRole("alert")).toBeInTheDocument());
});
```

### E2E: Playwright

```typescript
// tests/e2e/login.spec.ts
import { test, expect } from "@playwright/test";

test("login → dashboard flow", async ({ page }) => {
  await page.goto("/login");
  await page.fill('[name="email"]', "admin@example.com");
  await page.fill('[name="password"]', "Password1!");
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL("/dashboard");
  await expect(page.getByText("Welcome")).toBeVisible();
});
```

```typescript
// playwright.config.ts
import { defineConfig } from "@playwright/test";
export default defineConfig({
  testDir: "./tests/e2e",
  use: { baseURL: "http://localhost:5173" },
  webServer: { command: "pnpm dev", port: 5173 },
});
```

### E2E: Cypress (alternative)

```typescript
// cypress/e2e/login.cy.ts
describe("Login flow", () => {
  it("navigates to dashboard after login", () => {
    cy.visit("/login");
    cy.get('[name="email"]').type("admin@example.com");
    cy.get('[name="password"]').type("Password1!");
    cy.get('button[type="submit"]').click();
    cy.url().should("include", "/dashboard");
  });
});
```

---

## 11. Deployment Options

### Option A — Simple (Fastest to ship)

```
Backend → Railway (Node.js server)
Frontend (Vite) → Netlify / Cloudflare Pages
Frontend (Next.js) → Vercel
```

### Option B — Modern (Recommended for startups)

```bash
# Railway deploy (backend)
npm i -g @railway/cli
railway login
railway up
```

```bash
# Next.js on Vercel
vercel --prod
```

### Option C — Advanced (Docker + NGINX)

```dockerfile
# apps/api/Dockerfile
FROM node:22-alpine AS builder
WORKDIR /app
COPY . .
RUN corepack enable && pnpm install --frozen-lockfile
RUN pnpm --filter api build

FROM node:22-alpine AS runner
WORKDIR /app
COPY --from=builder /app/apps/api/dist ./dist
COPY --from=builder /app/apps/api/package.json .
RUN npm install --production
CMD ["node", "dist/server.js"]
```

```yaml
# docker-compose.yml
version: "3.9"
services:
  api:
    build: ./apps/api
    ports: ["3000:3000"]
    env_file: .env
    depends_on: [db, cache]
  db:
    image: postgres:16-alpine
    volumes: [pgdata:/var/lib/postgresql/data]
  cache:
    image: redis:7-alpine
volumes:
  pgdata:
```

```nginx
# infra/nginx/default.conf
server {
  listen 80;
  server_name myapp.com;

  location /api/ {
    proxy_pass http://api:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
  }

  location /socket.io/ {
    proxy_pass http://api:3000;
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
  }

  location / {
    proxy_pass http://web:3001;
  }
}
```

### CI/CD (GitHub Actions)

```yaml
# .github/workflows/ci.yml
name: CI
on: [push]
jobs:
  ci:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo build test lint

  deploy-backend:
    needs: ci
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 22
      - run: npm i -g @railway/cli
      - run: railway up --service backend
        env:
          RAILWAY_TOKEN: ${{ secrets.RAILWAY_TOKEN }}

  deploy-frontend:
    needs: ci
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    steps:
      - run: npx vercel --prod --token=${{ secrets.VERCEL_TOKEN }}
```

---

## 12. Security & Performance

### Security

```typescript
// apps/api/src/config/cors.ts
import cors from "cors";
export const corsMiddleware = cors({
  origin: (origin, cb) => {
    const allowed = process.env.ALLOWED_ORIGINS?.split(",") ?? [];
    (!origin || allowed.includes(origin)) ? cb(null, true) : cb(new Error("CORS blocked"));
  },
  credentials: true,
  methods: ["GET", "POST", "PUT", "PATCH", "DELETE"],
});
```

```typescript
// apps/api/src/middleware/sanitize.ts
import DOMPurify from "isomorphic-dompurify";
import { Request, Response, NextFunction } from "express";

const clean = (val: unknown): unknown => {
  if (typeof val === "string") return DOMPurify.sanitize(val.trim());
  if (Array.isArray(val)) return val.map(clean);
  if (val && typeof val === "object")
    return Object.fromEntries(Object.entries(val).map(([k, v]) => [k, clean(v)]));
  return val;
};

export const sanitize = (req: Request, _: Response, next: NextFunction) => {
  req.body = clean(req.body);
  next();
};
```

```typescript
// Rate limiting per user
import rateLimit from "express-rate-limit";
export const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, max: 5,
  keyGenerator: (req) => req.ip ?? "unknown",
  skipSuccessfulRequests: true,
});
export const apiLimiter = rateLimit({ windowMs: 15 * 60 * 1000, max: 100 });
```

### Performance

```typescript
// Redis cache middleware
export const cache = (ttl: number) =>
  async (req: Request, res: Response, next: NextFunction) => {
    const key = `cache:${req.originalUrl}`;
    const hit = await redis.get(key);
    if (hit) return res.json(JSON.parse(hit));
    const orig = res.json.bind(res);
    res.json = (body) => { redis.setex(key, ttl, JSON.stringify(body)); return orig(body); };
    next();
  };
```

```typescript
// Frontend: Prefetch + lazy routes (Vite)
const UsersPage = lazy(() => import("./pages/Users"));

const prefetchUsers = () =>
  queryClient.prefetchQuery({
    queryKey: ["users", 1],
    queryFn: () => apiClient.users.getAll({ page: 1 }),
  });
```

---

## 13. Environment & Config

```typescript
// packages/shared/src/env.ts
import { z } from "zod";

export const backendEnvSchema = z.object({
  NODE_ENV: z.enum(["development", "production", "test"]).default("development"),
  PORT: z.string().default("3000"),
  JWT_SECRET: z.string().min(32),
  REFRESH_SECRET: z.string().min(32),
  DATABASE_URL: z.string().url(),
  REDIS_URL: z.string().url().optional(),
  ALLOWED_ORIGINS: z.string(),
});

// apps/api/src/config/env.ts
const parsed = backendEnvSchema.safeParse(process.env);
if (!parsed.success) {
  console.error("Invalid env:", parsed.error.format());
  process.exit(1);
}
export const env = parsed.data;
```

```bash
# apps/web/.env.local (Vite)
VITE_API_URL=http://localhost:3000

# apps/web-next/.env.local (Next.js)
NEXT_PUBLIC_API_URL=http://localhost:3000
API_URL=http://api:3000   # server-only (SSR)
```

---

## 14. Best Practices Checklist

### Architecture
- [ ] Answered all 6 interactive questions before picking a structure
- [ ] Chose monorepo/polyrepo based on team & scale reasoning
- [ ] `packages/shared` is the single source of truth for all types
- [ ] `packages/api-client` wraps all HTTP — no raw `fetch` in app code
- [ ] API is versioned under `/api/v1`

### Backend
- [ ] All responses use `ApiResponse<T>` shape
- [ ] Zod validation on every request body using shared schemas
- [ ] JWT access token (15m) + HTTP-only refresh cookie (7d)
- [ ] Role-based middleware (`authorize("admin")`) on sensitive routes
- [ ] Global error handler — no stack traces in production
- [ ] Rate limiting: 100 req/15min general; 5 req/15min on auth
- [ ] CORS restricted to known origins via env var
- [ ] Input sanitization middleware on all POST/PATCH routes

### Frontend
- [ ] Access token stored in memory, never localStorage
- [ ] Silent token refresh in `BaseClient.handleRefresh()`
- [ ] TanStack Query for all server state (no useEffect + fetch)
- [ ] Global mutation error → toast via QueryClient config
- [ ] `<ProtectedRoute>` enforces auth and role checks
- [ ] Next.js middleware gates all server-rendered pages

### Testing
- [ ] Unit tests for all service methods
- [ ] Integration tests for all route handlers (supertest)
- [ ] Frontend component tests with mocked API client
- [ ] E2E test covering login → protected page flow

### Deployment
- [ ] Env variables validated at startup (Zod schema)
- [ ] Dockerfile multi-stage build (builder → runner)
- [ ] Docker Compose for local full-stack development
- [ ] NGINX handles SSL termination + WebSocket upgrade
- [ ] CI/CD: build → test → lint → deploy (per branch)

### Security
- [ ] `helmet()` for secure HTTP headers
- [ ] CSRF protection when using cookie-based auth
- [ ] `sameSite: strict` + `httpOnly` + `secure` on all cookies
- [ ] All DB queries parameterized (no string interpolation)
- [ ] Swagger/OpenAPI docs at `/api/docs` (non-public in production)

---

## Resources

- [Turborepo](https://turbo.build/repo)
- [TanStack Query](https://tanstack.com/query)
- [Next.js App Router](https://nextjs.org/docs/app)
- [Zod](https://zod.dev)
- [Socket.IO](https://socket.io/docs/v4/)
- [Playwright](https://playwright.dev)
- [OWASP API Security Top 10](https://owasp.org/www-project-api-security/)
- [Node.js Best Practices](https://github.com/goldbergyoni/nodebestpractices)
- [Railway Docs](https://docs.railway.com/)
- [Better Auth Docs](https://better-auth.com/docs)
- [Better Auth Options Reference](https://better-auth.com/docs/reference/options)
- [Better Auth LLMs.txt](https://better-auth.com/llms.txt)
- [Better Auth Init Options Source](https://github.com/better-auth/better-auth/blob/main/packages/core/src/types/init-options.ts)
- [Prisma Schema Docs](https://www.prisma.io/docs/concepts/components/prisma-schema)
- [Prisma Migrate Docs](https://www.prisma.io/docs/concepts/components/prisma-migrate)
- [Prisma Performance & Optimization](https://www.prisma.io/docs/guides/performance-and-optimization)
- [Prisma Connection Management](https://www.prisma.io/docs/guides/performance-and-optimization/connection-management)
- [Vercel React Best Practices (Compiled)](https://skills.sh/vercel-labs/agent-skills/vercel-react-best-practices)
- [Vercel React Best Practices Source](https://github.com/vercel-labs/agent-skills/tree/main/vercel-react-best-practices)

---
> Source: [Ameriq8/fullstack-nodejs-system-design](https://github.com/Ameriq8/fullstack-nodejs-system-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
