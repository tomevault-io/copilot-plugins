## consignadohub-web

> This document defines architecture, conventions, patterns, and expectations for the `consignadohub-web` project.

# ConsignadoHub Frontend — Angular 21 Project Context

This document defines architecture, conventions, patterns, and expectations for the `consignadohub-web` project.

---

## 1) Project Summary

**consignadohub-web** is the Angular 21 frontend for ConsignadoHub. It consumes the backend APIs (CustomerService, ProposalService), authenticates via Keycloak OIDC, and provides a production-grade UI for the consigned credit proposal pipeline.

**Key technical choices:**

- **Angular 21** — standalone components, Signals, new control-flow syntax (`@if`, `@for`, `@defer`)
- **Angular Signals** — primary reactive primitive; RxJS used only for async streams that warrant it
- **Keycloak JS** — OIDC authentication (no third-party Angular wrapper)
- **Functional interceptors + guards** — no class-based boilerplate
- **SCSS** — component-scoped styles
- **Vitest + Angular Testing Library** — unit and component tests

---

## 2) Backend Recap (Contracts the Frontend Must Respect)

| Concern | Rule |
|---|---|
| API versioning | All endpoints under `/v1/...` |
| Error format | RFC 7807 **ProblemDetails** with `extensions.correlationId`, `extensions.errorCode`, `extensions.validationErrors` |
| Auth | Keycloak OIDC; JWT Bearer required on all non-simulate endpoints |
| Correlation ID | Frontend generates UUID per request; sent as `X-Correlation-Id` header |
| CPF format | Backend normalizes to unformatted digits (e.g., `98765432100`) |
| Proposal statuses | `Submitted`, `UnderAnalysis`, `Approved`, `Rejected`, `ContractGenerated`, `Disbursed` |

---

## 3) Tech Stack & Dependencies

```bash
# Create project
ng new consignadohub-web --routing --style=scss --ssr=false
# Auth
npm install keycloak-js
# Correlation ID generation
npm install uuid
npm install --save-dev @types/uuid

```

### Angular Version

- Angular CLI 21 / Angular 21
- Node.js 22 LTS (minimum 20)
- TypeScript strict mode enabled

---

## 4) Repository Layout

```
consignadohub-web/
 src/
   app/
     core/
       auth/
         keycloak.service.ts        # Keycloak init, token, roles
         auth.guard.ts              # Route guard (authenticated)
         role.guard.ts              # Route guard (role-based)
       interceptors/
         auth.interceptor.ts        # Attaches JWT + X-Correlation-Id
         error.interceptor.ts       # Normalizes ProblemDetails errors
       services/
         customer.service.ts
         proposal.service.ts
       models/
         problem-details.model.ts
     shared/
       components/                  # Toast, Spinner, ErrorMessage, ConfirmDialog
       pipes/                       # CpfPipe, CurrencyBrlPipe, ProposalStatusPipe
       directives/
     features/
       customers/
         list/
         create/
         detail/
       proposals/
         simulate/
         submit/
         detail/                    # Status + timeline view
       dashboard/
     app.routes.ts
     app.config.ts
     app.component.ts
   environments/
     environment.ts
     environment.prod.ts
 .github/
   workflows/
     ci.yml
```

---

## 5) Environment Configuration

`src/environments/environment.ts`

```ts
export const environment = {
 production: false,
 apiBaseUrl: 'http://localhost:5100/v1', // CustomerService
 proposalApiBaseUrl: 'http://localhost:5200/v1', // ProposalService
 keycloak: {
   url: 'http://localhost:8080',
   realm: 'consignadohub',
   clientId: 'consignadohub-web' // public PKCE client
 }
};
```

`src/environments/environment.prod.ts`

```ts
export const environment = {
 production: true,
 apiBaseUrl: 'https://api.consignadohub.example.com/v1',
 proposalApiBaseUrl: 'https://proposals.consignadohub.example.com/v1',
 keycloak: {
   url: 'https://auth.consignadohub.example.com',
   realm: 'consignadohub',
   clientId: 'consignadohub-web'
 }
};
```

---

## 6) Core Models

### ProblemDetails

`src/app/core/models/problem-details.model.ts`

```ts
export interface ProblemDetails {
 type?: string;
 title?: string;
 status?: number;
 detail?: string;
 instance?: string;
 extensions?: {
   correlationId?: string;
   errorCode?: string;
   validationErrors?: Record<string, string[]>;
 };
}
```

### Customer

`src/app/core/models/customer.model.ts`

```ts
export interface Customer {
 id: string;
 name: string;
 cpf: string;         // normalized digits only (no dots/dashes)
 email: string;
 phone?: string;
 isActive: boolean;
 createdAt: string;   // ISO 8601 UTC
}

export interface CreateCustomerRequest {
 name: string;
 cpf: string;
 email: string;
 phone?: string;
}

export interface UpdateCustomerRequest {
 name: string;
 email: string;
 phone?: string;
}

export interface CustomerSearchParams {
 name?: string;
 cpf?: string;
 page?: number;
 pageSize?: number;
}
```
### Proposal

`src/app/core/models/proposal.model.ts`

```ts
export type ProposalStatus =
 | 'Submitted'
 | 'UnderAnalysis'
 | 'Approved'
 | 'Rejected'
 | 'ContractGenerated'
 | 'Disbursed';

export interface Proposal {
 id: string;
 customerId: string;
 requestedAmount: number;
 termMonths: number;
 monthlyInstallment: number;
 status: ProposalStatus;
 createdAt: string;
 timeline: ProposalTimelineEntry[];
}

export interface ProposalTimelineEntry {
 id: string;
 status: ProposalStatus;
 occurredAt: string;
 notes?: string;
}

export interface SimulateProposalRequest {
 customerId: string;
 requestedAmount: number;
 termMonths: number;
}

export interface SimulateProposalResponse {
 monthlyInstallment: number;
 totalAmount: number;
 cet: number;
}

export interface SubmitProposalRequest {
 customerId: string;
 requestedAmount: number;
 termMonths: number;
}
```

---

## 7) Authentication (Keycloak OIDC)

Use the **Authorization Code + PKCE** flow (public client). No client secret in the browser.

### Keycloak Service

`src/app/core/auth/keycloak.service.ts`

```ts
import Keycloak from 'keycloak-js';
import { Injectable, signal, computed } from '@angular/core';
import { environment } from '../../../environments/environment';

@Injectable({ providedIn: 'root' })

export class KeycloakService {
 private readonly kc = new Keycloak({
   url: environment.keycloak.url,
   realm: environment.keycloak.realm,
   clientId: environment.keycloak.clientId,
 });

 private readonly _authenticated = signal(false);

 readonly authenticated = this._authenticated.asReadonly();
 readonly username = computed(() => this.kc.tokenParsed?.['preferred_username'] ?? '');
 readonly roles = computed<string[]>(() => (this.kc.tokenParsed?.['realm_access']?.['roles'] ?? []));

 async init(): Promise<void> {
   const authenticated = await this.kc.init({
     onLoad: 'login-required',
     pkceMethod: 'S256',
     checkLoginIframe: false,
   });

   this._authenticated.set(authenticated);
 }

 async getToken(): Promise<string> {
   await this.kc.updateToken(30);
   return this.kc.token ?? '';
 }

 hasRole(role: string): boolean {
   return this.roles().includes(role);
 }

 logout(): void {
   this.kc.logout({ redirectUri: window.location.origin });
 }
}
```

### Roles Constants

`src/app/core/auth/roles.ts`

```ts
export const Roles = {
 Admin: 'consignado-admin',
 Analyst: 'consignado-analyst',
} as const;
```

---

## 8) Interceptors

### Auth Interceptor

`src/app/core/interceptors/auth.interceptor.ts`

```ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { from, switchMap } from 'rxjs';
import { v4 as uuid } from 'uuid';
import { KeycloakService } from '../auth/keycloak.service';

export const authInterceptor: HttpInterceptorFn = (req, next) => {
 const keycloak = inject(KeycloakService);

 return from(keycloak.getToken()).pipe(
   switchMap(token => {
     const cloned = req.clone({
       setHeaders: {
         Authorization: `Bearer ${token}`,
         'X-Correlation-Id': uuid(),
       },
     });

     return next(cloned);
   })
 );
};
```

### Error Interceptor

`src/app/core/interceptors/error.interceptor.ts`

```ts
import { HttpInterceptorFn } from '@angular/common/http';
import { inject } from '@angular/core';
import { catchError, throwError } from 'rxjs';
import { ToastService } from '../../shared/services/toast.service';
import { ProblemDetails } from '../models/problem-details.model';

export const errorInterceptor: HttpInterceptorFn = (req, next) => {
 const toast = inject(ToastService);

 return next(req).pipe(
   catchError(err => {
     const problem = err?.error as ProblemDetails | undefined;

     if (err.status === 401) {
       toast.error('Session expired. Please log in again.');
     } else if (err.status === 403) {
       toast.error('You do not have permission to perform this action.');
     } else if (problem?.detail) {
       toast.error(problem.detail);
     } else if (err.status >= 500) {
       toast.error('An unexpected server error occurred. Please try again later.');
     }

     return throwError(() => problem ?? err);
   })
 );
};
```

---

## 9) Guards

### Auth Guard (route-level)

`src/app/core/auth/auth.guard.ts`

```ts
import { inject } from '@angular/core';
import { CanActivateFn } from '@angular/router';
import { KeycloakService } from './keycloak.service';

export const authGuard: CanActivateFn = () => {
 return inject(KeycloakService).authenticated();
};
```

### Role Guard

`src/app/core/auth/role.guard.ts`

```ts
import { inject } from '@angular/core';
import { CanActivateFn, ActivatedRouteSnapshot } from '@angular/router';
import { KeycloakService } from './keycloak.service';
import { ToastService } from '../../shared/services/toast.service';

export const roleGuard: CanActivateFn = (route: ActivatedRouteSnapshot) => {
 const keycloak = inject(KeycloakService);
 const toast = inject(ToastService);
 const required: string = route.data['role'];

 if (keycloak.hasRole(required)) return true;

 toast.error('Access denied: insufficient permissions.');

 return false;
};
```

---

## 10) API Services

### Customer Service

`src/app/core/services/customer.service.ts`

```ts
import { HttpClient, HttpParams } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { environment } from '../../../environments/environment';
import { Customer, CreateCustomerRequest, UpdateCustomerRequest, CustomerSearchParams } from '../models/customer.model';

@Injectable({ providedIn: 'root' })

export class CustomerService {
 private readonly http = inject(HttpClient);
 private readonly base = `${environment.apiBaseUrl}/customers`;

 search(params: CustomerSearchParams = {}): Observable<Customer[]> {
   const query = new HttpParams({ fromObject: params as Record<string, string> });
   return this.http.get<Customer[]>(this.base, { params: query });
 }

 getById(id: string): Observable<Customer> {
   return this.http.get<Customer>(`${this.base}/${id}`);
 }

 getByCpf(cpf: string): Observable<Customer> {
   return this.http.get<Customer>(`${this.base}/cpf/${cpf}`);
 }

 create(payload: CreateCustomerRequest): Observable<Customer> {
   return this.http.post<Customer>(this.base, payload);
 }

 update(id: string, payload: UpdateCustomerRequest): Observable<Customer> {
   return this.http.put<Customer>(`${this.base}/${id}`, payload);
 }

 deactivate(id: string): Observable<void> {
   return this.http.delete<void>(`${this.base}/${id}`);
 }
}
```

### Proposal Service

`src/app/core/services/proposal.service.ts`

```ts
import { HttpClient } from '@angular/common/http';
import { inject, Injectable } from '@angular/core';
import { Observable } from 'rxjs';
import { environment } from '../../../environments/environment';
import {
 Proposal, SimulateProposalRequest, SimulateProposalResponse, SubmitProposalRequest
} from '../models/proposal.model';

@Injectable({ providedIn: 'root' })

export class ProposalService {
 private readonly http = inject(HttpClient);
 private readonly base = `${environment.proposalApiBaseUrl}/proposals`;

 simulate(payload: SimulateProposalRequest): Observable<SimulateProposalResponse> {
   return this.http.post<SimulateProposalResponse>(`${this.base}/simulate`, payload);
 }

 submit(payload: SubmitProposalRequest): Observable<Proposal> {
   return this.http.post<Proposal>(this.base, payload);
 }

 getById(id: string): Observable<Proposal> {
   return this.http.get<Proposal>(`${this.base}/${id}`);
 }

 getByCustomer(customerId: string): Observable<Proposal[]> {
   return this.http.get<Proposal[]>(`${this.base}?customerId=${customerId}`);
 }
}
```

---

## 11) App Configuration

`src/app/app.config.ts`

```ts
import { ApplicationConfig, inject, provideAppInitializer } from '@angular/core';
import { provideHttpClient, withInterceptors } from '@angular/common/http';
import { provideRouter, withComponentInputBinding } from '@angular/router';
import { KeycloakService } from './core/auth/keycloak.service';
import { authInterceptor } from './core/interceptors/auth.interceptor';
import { errorInterceptor } from './core/interceptors/error.interceptor';
import { routes } from './app.routes';

export const appConfig: ApplicationConfig = {
 providers: [
   provideRouter(routes, withComponentInputBinding()),
   provideHttpClient(withInterceptors([authInterceptor, errorInterceptor])),
   provideAppInitializer(() => inject(KeycloakService).init()),
 ],
};
```

---

## 12) Routing

`src/app/app.routes.ts`

```ts
import { Routes } from '@angular/router';
import { authGuard } from './core/auth/auth.guard';
import { roleGuard } from './core/auth/role.guard';
import { Roles } from './core/auth/roles';

export const routes: Routes = [
 { path: '', redirectTo: 'dashboard', pathMatch: 'full' },
 {
   path: 'dashboard',
   canActivate: [authGuard],
   loadComponent: () => import('./features/dashboard/dashboard.component').then(m => m.DashboardComponent),
 },
 {
   path: 'customers',
   canActivate: [authGuard],
   children: [
     { path: '', loadComponent: () => import('./features/customers/list/customer-list.component').then(m => m.CustomerListComponent) },
     { path: 'new', canActivate: [roleGuard], data: { role: Roles.Admin }, loadComponent: () => import('./features/customers/create/customer-create.component').then(m => m.CustomerCreateComponent) },
     { path: ':id', loadComponent: () => import('./features/customers/detail/customer-detail.component').then(m => m.CustomerDetailComponent) },
   ],
 },
 {
   path: 'proposals',
   canActivate: [authGuard],
   children: [
     { path: 'simulate', loadComponent: () => import('./features/proposals/simulate/proposal-simulate.component').then(m => m.ProposalSimulateComponent) },
     { path: 'new', loadComponent: () => import('./features/proposals/submit/proposal-submit.component').then(m => m.ProposalSubmitComponent) },
     { path: ':id', loadComponent: () => import('./features/proposals/detail/proposal-detail.component').then(m => m.ProposalDetailComponent) },
   ],
 },
 { path: '**', redirectTo: 'dashboard' },
];
```

---



## 13) State Management (Signals)

Prefer **Angular Signals** for component and service-level state. Use `toSignal()` to bridge RxJS Observables from HTTP calls.

### Pattern: resource signal per feature

```ts
// In a component
import { Component, inject, signal, computed } from '@angular/core';
import { toSignal } from '@angular/core/rxjs-interop';
import { CustomerService } from '../../core/services/customer.service';

@Component({ ... })

export class CustomerListComponent {
 private readonly customerService = inject(CustomerService);

 readonly customers = toSignal(this.customerService.search(), { initialValue: [] });
 readonly loading = signal(false);
 readonly error = signal<string | null>(null);
 readonly hasCustomers = computed(() => this.customers().length > 0);
}
```

For mutations (create, update, delete), use local `loading` and `error` signals and subscribe imperatively:

```ts
save() {
 this.loading.set(true);
 this.customerService.create(this.form.value).subscribe({
   next: () => this.router.navigate(['/customers']),
   error: (problem) => this.error.set(problem?.detail ?? 'Unexpected error'),
   complete: () => this.loading.set(false),
 });
}
```

---

## 14) Template Conventions (Angular 21)

Use the **control-flow syntax**. Do not use `*ngIf` / `*ngFor` / `*ngSwitch`.

```html
<!-- Conditional -->
@if (loading()) {
 <app-spinner />
} @else if (error()) {
 <app-error-message [message]="error()!" />
} @else {
 <ul>
   @for (customer of customers(); track customer.id) {
     <li>{{ customer.name }} — {{ customer.cpf | cpf }}</li>
   } @empty {
     <li>No customers found.</li>
   }
 </ul>
}

<!-- Lazy loading heavy sections -->
@defer (on viewport) {
 <app-proposal-timeline [entries]="proposal().timeline" />
}
```

---

## 15) Shared Pipes

| Pipe | Input | Output | Example |
|---|---|---|---|
| `CpfPipe` | `98765432100` | `987.654.321-00` | Customer CPF display |
| `CurrencyBrlPipe` | `5000` | `R$ 5.000,00` | Amounts |
| `ProposalStatusPipe` | `UnderAnalysis` | `Under Analysis` | Timeline labels |

---

## 16) Error Handling Rules

- All HTTP errors flow through `errorInterceptor` → `ToastService`
- Components receive `ProblemDetails` from `throwError()` if they need to show field-level validation errors
- Field-level errors from `extensions.validationErrors` must be displayed inline in forms
- `correlationId` must be shown in error UI (copy-to-clipboard button) for support traceability
- Never display raw `500` stack traces to the user

---

## 17) RBAC Matrix (Frontend)

| Action | Required Role |
|---|---|
| View customers | `consignado-analyst` or `consignado-admin` |
| Create / update / deactivate customer | `consignado-admin` |
| Simulate proposal | Public (no auth required; still sends token if present) |
| Submit proposal | `consignado-analyst` or `consignado-admin` |
| View proposal detail / timeline | `consignado-analyst` or `consignado-admin` |

Route guards enforce role requirements at navigation level. Additionally, hide admin-only UI elements using `keycloakService.hasRole(Roles.Admin)`.

---

## 18) Testing Strategy

### Unit Tests (Vitest or Jest)

- Services: mock `HttpClient` with `provideHttpClientTesting` + `HttpTestingController`
- Pipes: pure function tests, no Angular setup needed
- Guards: mock `KeycloakService`, test return value

### Component Tests (Angular Testing Library)

- Render the component, interact via user events, assert DOM output
- Mock services via `TestBed.configureTestingModule` providers

### Pattern

```ts
// customer.service.spec.ts
it('should GET /v1/customers on search()', () => {
 const { service, httpMock } = setup();
 service.search().subscribe();
 const req = httpMock.expectOne(`${environment.apiBaseUrl}/customers`);
 expect(req.request.method).toBe('GET');
 req.flush([]);
});
```

### What to test

- All API services: correct URL, method, payload
- Auth guard returns `false` when `authenticated` signal is `false`
- Role guard returns `false` when role missing
- Error interceptor calls `ToastService` for 401, 403, 5xx
- ProblemDetails validation errors rendered in form fields

---

## 19) CI/CD

`.github/workflows/ci.yml` (frontend job)

```yaml
- name: Setup Node.js
 uses: actions/setup-node@v4
 with:
   node-version: '22'
   cache: 'npm'
   cache-dependency-path: consignadohub-web/package-lock.json

- name: Install dependencies
 run: npm ci
 working-directory: consignadohub-web

- name: Build
 run: npm run build -- --configuration production
 working-directory: consignadohub-web

- name: Test
 run: npm test -- --watch=false --coverage
 working-directory: consignadohub-web
```

---

## 20) Coding Standards

### Conventions

- **Standalone components everywhere** — no NgModule
- **`inject()`** over constructor injection
- **Signals** for state; RxJS only for streams (HTTP, WebSocket, event buses)
- **Typed forms** (`FormGroup<{...}>`) — no untyped form controls
- **New template control flow** — `@if`, `@for`, `@switch`, `@defer`
- Single-responsibility: one component per file, one concern per service
- Lazy-load all feature components via `loadComponent`

### Naming

| Artifact | Convention | Example |
|---|---|---|
| Component | `feature-name.component.ts` | `customer-list.component.ts` |
| Service | `domain.service.ts` | `customer.service.ts` |
| Guard | `concern.guard.ts` | `auth.guard.ts`, `role.guard.ts` |
| Interceptor | `concern.interceptor.ts` | `auth.interceptor.ts` |
| Model | `domain.model.ts` | `customer.model.ts` |
| Pipe | `name.pipe.ts` | `cpf.pipe.ts` |

### Commit Conventions

Follow the same **Conventional Commits** as the backend:

- `feat:`, `fix:`, `refactor:`, `test:`, `docs:`, `chore:`

---

## 21) Local Development

```bash
cd consignadohub-web
npm install
npm start # ng serve — default http://localhost:4200
```

Requires the backend services and Keycloak running locally.

Set `apiBaseUrl` and `keycloak` in `environment.ts` to match the ports exposed by Docker Compose.

---

## 22) Scope \& Feature Milestones

| Milestone | Features |
|---|---|
| 1 | Project scaffold, auth flow, Keycloak init, interceptors, shared models |
| 2 | Customer list, create, detail (with RBAC) |
| 3 | Proposal simulate + submit + detail with timeline |
| 4 | Dashboard, unit/component tests, CI pipeline |
| 5 | Docker image, production build, deploy to cloud (optional) |

---

## 23) What This Frontend Demonstrates (Portfolio Talking Points)

- Angular 21 idiomatic patterns: Signals, standalone components, functional APIs
- Auth integration with Keycloak OIDC (PKCE flow, token refresh)
- Alignment with backend ProblemDetails error contract + field-level validation display
- RBAC enforcement at routing and UI level
- Observable-to-Signal bridge (`toSignal`) without over-relying on RxJS
- Typed HTTP services matching backend DTOs exactly
- Correlation ID propagation for full-stack traceability

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cvieirasp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
