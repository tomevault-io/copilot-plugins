## packiq-backend-api

> PackIQ backend API for frontend integration (auth, files, extractions, RAG)


# PackIQ Backend API – Frontend Integration

Use this rule when integrating the PackIQ frontend with the backend. The backend has two entry points: **Java API** (auth, files, extractions, documents) and **OCR/Model API** (RAG: ask, vector search, embed). Prefer the Java API for all calls that it proxies; use the OCR API base only for RAG endpoints if your deployment does not proxy them through Java.

## Base URLs (Next.js)

- **Java API** (primary): Use `NEXT_PUBLIC_API_URL` in `.env.local`. Example: `http://localhost:8080` or `https://your-app.example.com`.
- **OCR/Model API**: For RAG (ask, search, embed) if not proxied, use `NEXT_PUBLIC_OCR_API_URL` (e.g. `http://localhost:8080` when OCR app runs there).

## Authentication

All auth endpoints: `POST /auth/*`. No Bearer token for auth calls; use the returned tokens for subsequent requests.

### 1. Password check (get temp token)

```http
POST /auth/password-check
Content-Type: application/json

{ "email": "user@example.com", "password": "..." }
```

- **200**: `{ "valid": true, "token": "<tempToken>" }` — use `token` for send-otp / verify-otp.
- **200**: `{ "valid": false }` — wrong password.
- Temp token is short-lived (~15 min). Use it only for the OTP flow.

### 2. Send OTP

```http
POST /auth/send-otp
Content-Type: application/json

{ "tempToken": "<from password-check>", "channel": "EMAIL" }
```

- **channel**: `"EMAIL"` or `"SMS"` (SMS uses user's stored phone).
- **200**: `{ "valid": true }`.
- **401**: `{ "valid": false, "message": "Token expired" }` or `"Invalid token"`.
- **400**: missing/invalid body.

### 3. Resend OTP

```http
POST /auth/resend-otp
Content-Type: application/json

{ "tempToken": "<same>", "channel": "EMAIL" }
```

- **429**: rate limit (e.g. "Wait before resending").
- **401**: expired/invalid temp token.

### 4. Verify OTP (get access + refresh tokens)

```http
POST /auth/verify-otp
Content-Type: application/json

{ "tempToken": "<same>", "channel": "EMAIL", "otp": "123456" }
```

- **200**: `{ "valid": true, "token": "<accessToken>", "refreshToken": "<refreshToken>" }`.
- **401**: `{ "valid": false, "message": "OTP expired" }` or `"OTP invalid"` or token message.

### 5. Refresh access token

```http
POST /auth/refresh
Content-Type: application/json

{ "refreshToken": "<refreshToken>" }
```

- **200**: `{ "valid": true, "token": "<newAccessToken>", "refreshToken": "<newRefreshToken>" }`.
- **401**: expired/invalid refresh token.

### Using the access token

For endpoints that require auth (if your Java app enforces it), send:

```http
Authorization: Bearer <accessToken>
```

Store `token` and `refreshToken` (e.g. in memory or secure storage). On 401, retry once with refresh then redirect to login.

---

## Clients (Java API, session-based selection)

Base path: `/clients`. **Session:** Select and get-selected use the server session (cookie `JSESSIONID`). Use `credentials: 'include'` (or equivalent) so the browser sends cookies.

### List clients

```http
GET /clients
```

- **200**: `[ { "id": "<uuid>", "name": "...", "industry": "...", "location": "...", "logo": "..." | null } ]`.

### Get client by ID

```http
GET /clients/{id}
```

- **200**: `{ "id", "name", "industry", "location", "logo" }`.
- **404**: client not found.

### Select client for session

```http
POST /clients/select
Content-Type: application/json

{ "clientId": "<uuid>" }
```

- **200**: Returns the selected client object. Backend stores `clientId` in the session for subsequent requests from the same browser.
- **400**: missing or invalid `clientId`.
- **404**: client not found.

### Get selected client

```http
GET /clients/selected
```

- **200**: `{ "id", "name", "industry", "location" }` — client currently selected for this session.
- **204**: No client selected for this session.

---

## File and extraction endpoints (Java API)

Base path: `/files`. Use `multipart/form-data` for uploads. Responses are JSON. Include `Authorization: Bearer <accessToken>` if your backend requires it.

### Start OCR + extraction (upload file, get job_id)

```http
POST /files/ocr
Content-Type: multipart/form-data

file: <File>
doc_type: "spec" | "bom" | "sales"   (optional, default "spec")
user_id: <UUID>                      (optional)
```

- **202**: `{ "job_id": "<uuid>", "status": "pending", "doc_type": "spec", "document_id": "<uuid>" }`.
- **400**: missing file or invalid doc_type.

Poll for result:

```http
GET /files/extract/{jobId}
```

- **200**: `{ "job_id": "...", "status": "pending" }` or `"running"` — keep polling.
- **200**: `{ "job_id": "...", "status": "completed", "text": "...", "packaging": { ... }, "doc_type": "spec" }`.
- **200**: `{ "status": "failed", "error": "..." }`.
- **404**: job not found.

### Start extract

```http
POST /files/extract
Content-Type: multipart/form-data

file: <File>
user_id: <UUID>   (optional)
```

- **202**: same shape (job_id, document_id). Poll `GET /files/extract/{jobId}` as above.

### Ingest guidance PDF

```http
POST /files/ingest/guidance
Content-Type: multipart/form-data

file: <PDF>
jurisdiction: "California"     (required)
pro: "CalRecycle"             (required)
effective_date: "2025-09-02"   (required)
topics: "topic1,topic2"       (optional)
```

- **200**: `{ "message": "Successfully ingested N chunks", "chunks": N }`.
- **400/500**: error body with `"error"`.

### Ingest categories PDF

```http
POST /files/ingest/categories
Content-Type: multipart/form-data

file: <PDF>
jurisdiction: "California"    (optional)
effective_date: "2026-01-01"   (optional)
```

- **200**: `{ "message": "Successfully ingested N categories", "categories": N }`.

---

## Extractions and documents (Java API)

### Get extraction by business ID

```http
GET /extractions/spec/{id}
GET /extractions/bom/{id}
GET /extractions/sales/{id}
```

- **200**: packaging JSON (spec, bom, or sales structure).
- **404**: not found.

### Download uploaded document

```http
GET /documents/{documentId}/download
```

- Returns file (e.g. PDF/image). Use `document_id` from OCR/extract response.

---

## RAG endpoints (OCR/Model API)

Use **OCR API base URL** for these if your gateway does not proxy them. Request body is JSON.

### Ask (RAG Q&A)

```http
POST /ask
Content-Type: application/json

{
  "question": "What are the source reduction reporting requirements?",
  "top_k": 5,
  "include_sources": true,
  "doc_type": "guidance"
}
```

- **doc_type** (optional): `spec` | `bom` | `sales` | `guidance` | `material_category`.
- **200**: `{ "answer": "...", "sources": [ { "key", "metadata", "distance" } ] }`.
- **400**: missing `question`.
- **500**: `{ "error": "Ask failed", "detail": "..." }`.

### Vector search

```http
POST /extractions/search
Content-Type: application/json

{
  "query": "corrugated kraft carton",
  "top_k": 10,
  "doc_type": "spec",
  "include_packaging": false
}
```

- **200**: `{ "query", "top_k", "distance_metric", "results": [ { "key", "distance", "metadata", "packaging"? } ] }`.

### Create embedding for an extraction

```http
POST /extractions/embed
Content-Type: application/json

{ "job_id": "<uuid>" }
```
or `{ "doc_type": "spec", "extracted_id": "3030051786" }`.

- **200**: `{ "job_id", "doc_type", "extracted_id", "dimensions", "updated": true }`.
- **404**: no extraction found.

---

## CORS and errors

- Backend allows CORS for frontend origin. Use credentials if you send auth headers.
- Auth: 401 for expired/invalid token or OTP; 429 for resend rate limit. Message in `message` or `detail`.
- File/RAG: 400 for validation, 500 for server errors; body often `{ "error": "...", "detail": "..." }`.

## Next.js / TypeScript example (fetch)

```ts
const API_URL = process.env.NEXT_PUBLIC_API_URL ?? '';
const OCR_URL = process.env.NEXT_PUBLIC_OCR_API_URL ?? API_URL;

export async function passwordCheck(email: string, password: string) {
  const res = await fetch(`${API_URL}/auth/password-check`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ email, password }),
  });
  const data = await res.json();
  if (!res.ok) throw new Error(data.message ?? 'Password check failed');
  return data; // { valid, token? }
}

export async function verifyOtp(tempToken: string, channel: string, otp: string) {
  const res = await fetch(`${API_URL}/auth/verify-otp`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ tempToken, channel, otp }),
  });
  const data = await res.json();
  if (!res.ok) throw new Error(data.message ?? 'Verification failed');
  return data; // { valid, token, refreshToken }
}

export async function uploadAndOcr(file: File, docType: 'spec' | 'bom' | 'sales', accessToken?: string) {
  const form = new FormData();
  form.append('file', file);
  form.append('doc_type', docType);
  const res = await fetch(`${API_URL}/files/ocr`, {
    method: 'POST',
    headers: accessToken ? { Authorization: `Bearer ${accessToken}` } : {},
    body: form,
  });
  const data = await res.json();
  if (!res.ok) throw new Error(data.error ?? 'Upload failed');
  return data; // { job_id, status, doc_type, document_id }
}

export async function ask(question: string, options?: { top_k?: number; doc_type?: string }) {
  const res = await fetch(`${OCR_URL}/ask`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ question, top_k: options?.top_k ?? 5, doc_type: options?.doc_type }),
  });
  const data = await res.json();
  if (!res.ok) throw new Error(data.error ?? data.detail ?? 'Ask failed');
  return data; // { answer, sources? }
}

// Clients (use credentials: 'include' so session cookie is sent)
export async function listClients() {
  const res = await fetch(`${API_URL}/clients`, { credentials: 'include' });
  if (!res.ok) throw new Error('Failed to list clients');
  return res.json(); // Client[]
}

export async function getClient(id: string) {
  const res = await fetch(`${API_URL}/clients/${id}`, { credentials: 'include' });
  if (res.status === 404) return null;
  if (!res.ok) throw new Error('Failed to get client');
  return res.json(); // Client
}

export async function selectClient(clientId: string) {
  const res = await fetch(`${API_URL}/clients/select`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    credentials: 'include',
    body: JSON.stringify({ clientId }),
  });
  const data = await res.json();
  if (!res.ok) throw new Error(res.status === 404 ? 'Client not found' : data.error ?? 'Select failed');
  return data; // Client
}

// Client type: { id: string; name: string; industry: string; location: string }
export async function getSelectedClient(): Promise<{ id: string; name: string; industry: string; location: string } | null> {
  const res = await fetch(`${API_URL}/clients/selected`, { credentials: 'include' });
  if (res.status === 204) return null;
  if (!res.ok) throw new Error('Failed to get selected client');
  return res.json();
}
```

Use this rule when adding or changing API calls, auth flows, clients, or RAG integration in the frontend.

---
> Source: [imjamalnasir/Pack-IQ-Frontend](https://github.com/imjamalnasir/Pack-IQ-Frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
