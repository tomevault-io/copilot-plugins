## ecom-poc

> A detailed prompt/guide for an AI agent (Gemini) to generate a **full‑stack, production‑grade** web application: a **sci‑fi themed, black‑and‑gold, exclusive PC‑building eCommerce store**.

# GEMINI.md — Sci‑Fi PC‑Builder E‑Commerce (Python + React + PostgreSQL)

A detailed prompt/guide for an AI agent (Gemini) to generate a **full‑stack, production‑grade** web application: a **sci‑fi themed, black‑and‑gold, exclusive PC‑building eCommerce store**.

The app lets users browse curated parts (motherboards, CPUs, coolers, RAM, storage, GPUs, PSUs, cases, etc.), configure builds, add to cart, and checkout (simulated). Use **Python (FastAPI)** for the backend, **React** for the frontend, and **PostgreSQL** for data. Source hardware data via **[`pypartpicker`](https://pypi.org/project/pypartpicker/)**.

---

## 0) High‑Level Goals

* **Theme:** sleek sci‑fi, **black & gold** palette, neon accents, star‑field textures; modern, accessible, responsive UI.
* **Functionality:** browse → filter → compare → configure build → validate part compatibility → add to cart → checkout (simulated payment) → order confirmation.
* **Data:** ingest parts from **PCPartPicker** via `pypartpicker`, normalize into Postgres, and cache.
* **Architecture:** modular monorepo; typed APIs; JWT auth; migrations; CI‑ready.
* **Deliverables:** runnable app (Docker optional), seed scripts, docs, tests.

---

## 1) Project Setup

### 1.1 Repo Layout

```

All new files must be inside the pc-builder/ directory

pc-builder/
├─ apps/
│  ├─ backend/                 # FastAPI service
│  │  ├─ src/
│  │  │  ├─ core/              # settings, logging, security
│  │  │  ├─ db/                # SQLAlchemy models, alembic, repos
│  │  │  ├─ api/               # routers: auth, products, builds, carts, orders
│  │  │  ├─ services/          # business logic (compatibility, pricing, inventory)
│  │  │  ├─ schemas/           # Pydantic request/response models
│  │  │  └─ utils/
│  │  ├─ tests/
│  │  ├─ alembic/
│  │  ├─ pyproject.toml
│  │  └─ Dockerfile
│  └─ frontend/                # React (Vite)
│     ├─ src/
│     │  ├─ app/               # routing, layout, providers
│     │  ├─ components/        # UI widgets
│     │  ├─ features/          # pages: catalog, product, build, cart, account
│     │  ├─ lib/               # api client, hooks, utils
│     │  └─ styles/            # theme tokens, Tailwind config
│     ├─ public/
│     ├─ index.html
│     ├─ package.json
│     └─ Dockerfile
├─ data/
│  ├─ seed/                    # json/csv fixtures
│  └─ ingestion/               # pypartpicker ETL scripts
├─ deploy/
│  ├─ docker-compose.yml
│  ├─ nginx/
│  │  └─ nginx.conf
│  └─ k8s/                     # (optional)
├─ docs/
│  └─ API.md
└─ README.md
```

### 1.2 Technology Choices

* **Backend:** Python 3.11+, **FastAPI**, SQLAlchemy 2.x, Alembic, Uvicorn, Pydantic v2.
* **Auth:** JWT (access + refresh). Password hashing via `passlib[bcrypt]`.
* **DB:** PostgreSQL 14+. Indexing strategy for search (btree, trigram via `pg_trgm`).
* **Frontend:** React 18 + Vite + TypeScript, React Router, TanStack Query, Tailwind CSS.
* **Data Source:** `pypartpicker` for scraping/aggregation; cache requests and handle rate‑limits.
* **Testing:** Pytest, Playwright or Cypress for E2E, Vitest for frontend.
* **Tooling:** Ruff/Black/Isort, ESLint/Prettier, commitlint, GitHub Actions (CI).

### 1.3 Theme & Branding

```ts
// apps/frontend/src/styles/theme-tokens.ts
export const colors = {
  bg: "#0b0e11",       // near-black
  bgAlt: "#13161a",    // panels
  text: "#E6E6E6",    // high-contrast text
  gold: "#D4AF37",    // primary accent
  goldSoft: "#b8932a",
  neon: "#00E5FF",    // small accent for focus/active
};
export const radii = { soft: "16px", pill: "9999px" };
export const shadows = { card: "0 10px 30px rgba(0,0,0,0.4)" };
```

Tailwind: extend palette and apply **black‑and‑gold** with accessible contrast; use subtle star‑field SVG background.

### 1.4 Environment & Secrets

* `POSTGRES_HOST`, `POSTGRES_PORT`, `POSTGRES_DB`, `POSTGRES_USER`, `POSTGRES_PASSWORD`
* `JWT_SECRET`, `JWT_EXPIRES_MIN`, `JWT_REFRESH_EXPIRES_MIN`
* `ALLOWED_ORIGINS`
* `LOG_LEVEL`

### 1.5 Install & Run (Dev)

**Backend**

```bash
cd apps/backend
uvenv() { python -m venv .venv && . .venv/bin/activate; }; uvenv
pip install -U pip
pip install -e .  # use pyproject
alembic upgrade head
uvicorn src.main:app --reload --port 8000
```

**Frontend**

```bash
cd apps/frontend
pnpm i
pnpm dev --port 5173
```

---

## 2) Database Design (PostgreSQL)

### 2.1 Entities

* **users** (id, email, hash, roles)
* **brands** (id, name)
* **categories** (id, slug, name)
* **products** (id, sku, name, brand_id, category_id, price, msrp, rating, image_url, short_desc)
* **product_specs** (id, product_id, key, value) — flexible KV pairs (e.g., "Socket" → "AM5")
* **compat_rules** (id, left_category, right_category, rule_expr, message) — compatibility logic as JSONExpr
* **inventory** (id, product_id, qty, status)
* **carts** (id, user_id, status)
* **cart_items** (id, cart_id, product_id, qty, unit_price)
* **orders** (id, user_id, total, status, created_at)
* **order_items** (id, order_id, product_id, qty, unit_price)
* **builds** (id, user_id, name, visibility)
* **build_items** (id, build_id, product_id, role) — role = cpu|mobo|ram|gpu|case|psu|cooler|storage

### 2.2 Schema (excerpt)

```sql
CREATE EXTENSION IF NOT EXISTS pg_trgm;

CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email TEXT UNIQUE NOT NULL,
  password_hash TEXT NOT NULL,
  roles TEXT[] DEFAULT '{user}',
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE categories (
  id SERIAL PRIMARY KEY,
  slug TEXT UNIQUE NOT NULL,
  name TEXT NOT NULL
);

CREATE TABLE brands (
  id SERIAL PRIMARY KEY,
  name TEXT UNIQUE NOT NULL
);

CREATE TABLE products (
  id BIGSERIAL PRIMARY KEY,
  sku TEXT UNIQUE,
  name TEXT NOT NULL,
  brand_id INT REFERENCES brands(id) ON DELETE SET NULL,
  category_id INT REFERENCES categories(id) ON DELETE CASCADE,
  price NUMERIC(10,2) NOT NULL,
  msrp NUMERIC(10,2),
  rating NUMERIC(2,1),
  image_url TEXT,
  short_desc TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);

CREATE TABLE product_specs (
  id BIGSERIAL PRIMARY KEY,
  product_id BIGINT REFERENCES products(id) ON DELETE CASCADE,
  key TEXT NOT NULL,
  value TEXT NOT NULL
);

CREATE INDEX idx_products_name_trgm ON products USING gin (name gin_trgm_ops);
CREATE INDEX idx_specs_key_value ON product_specs(key, value);

CREATE TABLE builds (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID REFERENCES users(id) ON DELETE CASCADE,
  name TEXT NOT NULL,
  visibility TEXT DEFAULT 'private'
);

CREATE TABLE build_items (
  id BIGSERIAL PRIMARY KEY,
  build_id UUID REFERENCES builds(id) ON DELETE CASCADE,
  product_id BIGINT REFERENCES products(id) ON DELETE CASCADE,
  role TEXT NOT NULL
);

-- carts, orders similar pattern
```

### 2.3 Compatibility Modeling

* **product_specs** holds socket, form factor, wattage, clearance, etc.
* **compat_rules** encodes constraints (e.g., `{ left: "cpu", right: "motherboard", expr: "left.socket == right.socket" }`).
* Service evaluates rules when adding to build/cart.

---

## 3) Backend API (FastAPI)

### 3.1 Dependencies

```toml
# apps/backend/pyproject.toml (deps excerpt)
[project]
dependencies = [
  "fastapi>=0.115",
  "uvicorn[standard]",
  "pydantic>=2",
  "SQLAlchemy>=2",
  "psycopg[binary]",
  "alembic",
  "python-jose[cryptography]",
  "passlib[bcrypt]",
  "python-multipart",
  "orjson",
  "redis>=5",         # optional caching
  "httpx>=0.27",      # if needed
]
```

### 3.2 App Structure

```
src/main.py          # FastAPI app, CORS, routes include
src/core/config.py   # settings via pydantic BaseSettings
src/core/security.py # JWT create/verify, password hashing
src/db/models.py     # SQLAlchemy models
src/db/session.py    # engine, SessionLocal
src/db/repo/*.py     # repositories per aggregate
src/schemas/*.py     # Pydantic models
src/api/routers/*.py # routers: auth, products, categories, builds, carts, orders
src/services/*.py    # business logic (compatibility, pricing)
```

### 3.3 Auth Endpoints

* `POST /auth/register` → create user
* `POST /auth/login` → returns `{access, refresh}`
* `POST /auth/refresh` → new access token

### 3.4 Catalog Endpoints

* `GET /categories` → list
* `GET /products` → pagination, filters (`category`, `brand`, `price_min/max`, `search`)
* `GET /products/{id}` → details (specs)
* `GET /brands` → list

### 3.5 Build Configurator

* `GET /builds` (user) / `POST /builds` / `GET /builds/{id}` / `PATCH /builds/{id}` / `DELETE /builds/{id}`
* `POST /builds/{id}/items` `{ product_id, role }` → compatibility check
* `DELETE /builds/{id}/items/{item_id}`

### 3.6 Cart & Orders

* `GET /carts/me` (create if missing)
* `POST /carts/me/items` `{ product_id, qty }` (reprice from product.price)
* `PATCH /carts/me/items/{item_id}` `{ qty }`
* `DELETE /carts/me/items/{item_id}`
* `POST /orders` → snapshot cart into order (simulated payment)
* `GET /orders/me`

### 3.7 Response Models (excerpt)

```py
class ProductOut(BaseModel):
    id: int
    name: str
    sku: str | None
    brand: str | None
    category: str
    price: Decimal
    rating: float | None
    image_url: HttpUrl | None
    specs: dict[str, str] = {}
```

### 3.8 Compatibility Service (pseudo)

```py
# services/compat.py
CPU= "cpu"; MOBO="motherboard"

def compatible(left: Product, right: Product, role_left: str, role_right: str) -> tuple[bool, str|None]:
    if role_left==CPU and role_right==MOBO:
        return (left.specs.get("Socket") == right.specs.get("Socket"), "CPU and motherboard sockets must match")
    # add PSU wattage vs GPU/CPU TDP, case form factor vs mobo, cooler clearance vs case, RAM gen vs mobo
```

---

## 4) Data Ingestion from PCPartPicker

### 4.1 Install & Notes

```bash
pip install pypartpicker
```

* Respect rate limits; cache responses on disk (`.cache/pypartpicker/…`).
* Map categories from PCPartPicker → local `categories` table.
* Extract normalized **specs** (socket, chipset, memory type, form factor, TDP, wattage, size, etc.).

### 4.2 Category Mapping (example)

| Local Category | pypartpicker Slug(s)         |
| -------------- | ---------------------------- |
| `cpu`          | `cpu`                        |
| `motherboard`  | `motherboard`                |
| `memory`       | `memory`                     |
| `storage`      | `internal-hard-drive`, `ssd` |
| `gpu`          | `video-card`                 |
| `psu`          | `power-supply`               |
| `case`         | `case`                       |
| `cpu-cooler`   | `cpu-cooler`                 |

### 4.3 Ingestion Script (sketch)

```py
# data/ingestion/ingest_pcpp.py
from pypartpicker import Scraper
from sqlalchemy.orm import Session
from models import Product, Brand, Category, ProductSpec

SCRAPE_LIMIT = 200  # for dev

SPEC_KEYS = ["Socket","Chipset","Memory Type","Form Factor","Wattage","TDP","Capacity","Interface","Length"]

def upsert_brand(db: Session, name: str) -> int: ...

def upsert_category(db: Session, slug: str, name: str) -> int: ...

def ingest_category(db: Session, slug: str, name: str):
    items = Scraper().category(slug)
    for i, item in enumerate(items):
        if i >= SCRAPE_LIMIT: break
        brand_id = upsert_brand(db, item.brand)
        cat_id   = upsert_category(db, slug, name)
        p = Product(name=item.name, sku=item.sku, brand_id=brand_id, category_id=cat_id,
                    price=item.price or 0, msrp=item.price, image_url=item.image)  
        db.add(p); db.flush()
        # specs normalize
        for k in SPEC_KEYS:
            v = item.specs.get(k) or item.specs.get(k.lower())
            if v:
                db.add(ProductSpec(product_id=p.id, key=k, value=str(v)))
    db.commit()

if __name__ == "__main__":
    # init db session
    # call ingest_category for each mapping
    ...
```

> Tip: Use retry/backoff and a small random sleep between requests.

---

## 5) Frontend (React + Vite + Tailwind)

### 5.1 App Routes

* `/` — Landing (hero banner, featured builds, call‑to‑action)
* `/catalog` — Browse + filters (category, brand, price, specs)
* `/product/:id` — Product details (gallery, specs, related)
* `/build` — Build configurator (stepper: CPU → Motherboard → RAM → GPU → Storage → Case → PSU → Cooler)
* `/cart` — Cart & totals
* `/checkout` — Simulated checkout
* `/account` — Orders & saved builds
* `/auth/*` — Login/Register/Reset

### 5.2 State & Data

* TanStack Query for server cache.
* Auth via context; store access token in `memory + HttpOnly cookie (refresh)` or secure storage; handle refresh.
* API client via Axios with interceptors.

### 5.3 UI/UX Notes

* **Black & Gold** primary; neon cyan focus; glassmorphism cards.
* Sticky filter panel on desktop; drawer on mobile.
* Compare up to 4 products; highlight compatible parts.
* Accessibility: focus rings, high contrast, prefers‑reduced‑motion.

### 5.4 Example Components

* `PartCard`, `PriceTag`, `SpecList`, `CompatibilityBadge`, `AddToBuildButton`
* `BuildStepper` with role guards (only compatible parts enabled)

---

## 6) Integration Steps

1. **Seed DB** using ingestion script; run Alembic migrations.
2. **Start Backend**, confirm `/docs` Swagger available.
3. **Hook Frontend** API client → env `VITE_API_URL`.
4. Implement **catalog list** → `GET /products` with filters.
5. Implement **product detail** → hydrate specs, related parts by category/brand.
6. Implement **build flow** → request compatible options based on current selection.
7. Implement **cart** and **checkout** (order creation).
8. Add **auth** gates for builds/orders.

---

## 7) Deployment Considerations

### 7.1 Docker Compose (dev/prod‑like)

```yaml
# deploy/docker-compose.yml
services:
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: pcbuilder
      POSTGRES_USER: pcbuilder
      POSTGRES_PASSWORD: pcbuilder
    volumes: [dbdata:/var/lib/postgresql/data]
    ports: ["5432:5432"]

  backend:
    build: ../apps/backend
    env_file: ../apps/backend/.env
    depends_on: [db]
    ports: ["8000:8000"]
    command: uvicorn src.main:app --host 0.0.0.0 --port 8000

  frontend:
    build: ../apps/frontend
    environment:
      - VITE_API_URL=/api
    depends_on: [backend]
    ports: ["5173:80"]

  nginx:
    image: nginx:alpine
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    ports: ["80:80"]
    depends_on: [frontend, backend]

volumes:
  dbdata:
```

**nginx.conf** should reverse proxy `/api` → backend and serve frontend static.

### 7.2 Migrations & Seeding

* On first boot: `alembic upgrade head` then run `ingest_pcpp.py`.
* Provide `make seed` task.

### 7.3 Secrets & Config

* Use env files in Docker secrets; never hardcode JWT secrets.

---

## 8) Testing Strategy

* **Backend:** pytest for services and routers; factory fixtures; sqlite for unit or Postgres test container.
* **Frontend:** Vitest + React Testing Library.
* **E2E:** Playwright/Cypress flows — add parts to build, verify compatibility, checkout.

---

## 9) Non‑Functional Requirements

* **Performance:** pagination + indexed queries; Redis cache for hot endpoints.
* **Security:** input validation; rate‑limit login; store refresh token in HttpOnly cookie; CORS whitelist.
* **Observability:** structured logs, request IDs, basic metrics (latency, errors).
* **Accessibility:** semantically correct HTML; keyboard navigation; color contrast.

---

## 10) Example Prompts for Gemini (Agent Instructions)

> Use these as *system* or *task* prompts when asking the model to generate code.

**Backend scaffolding**

* “Create a FastAPI project with SQLAlchemy models for users, categories, brands, products, product_specs, builds, build_items, carts, orders. Provide Alembic migrations and JWT auth. Expose REST endpoints as specified in §3.3–3.6. Include pagination, search, and spec filters.”

**Compatibility service**

* “Implement a compatibility engine that validates CPU↔Motherboard by socket, RAM gen with motherboard, PSU wattage vs GPU/CPU total TDP + headroom, case form factor vs motherboard, and GPU length vs case clearance.”

**Ingestion**

* “Write `ingest_pcpp.py` using `pypartpicker` to scrape key categories, map to our schema, and upsert entries. Normalize common spec keys and persist to `product_specs`.”

**Frontend**

* “Build React pages in §5.1 with Tailwind. Implement filters, comparison, build stepper, cart flow, and JWT auth integration. Use TanStack Query and Axios. Apply theme tokens in §1.3.”

**Deployment**

* “Generate Dockerfiles and nginx.conf for the layout in §7.1. On container start, run migrations and seed.”

---

## 11) Acceptance Criteria

* Users can **browse**, **filter**, and **search** across categories.
* Users can **configure** a build with enforced compatibility.
* Users can **register/login**, **add to cart**, and **place orders** (simulated payment).
* Database populated from **PCPartPicker** via ingestion script.
* **Black‑and‑gold sci‑fi** theme consistently applied.
* App runs locally (compose) with seeded data and passes smoke tests.

---

## 12) Appendix: Minimal API Contracts (OpenAPI Sketch)

```yaml
GET /products:
  params: [page, page_size, search, category, brand, price_min, price_max]
  200: { items: ProductOut[], total: number }

GET /products/{id}:
  200: ProductOut

POST /builds: { name }
GET /builds/{id}
POST /builds/{id}/items: { product_id, role }

GET /carts/me
POST /carts/me/items: { product_id, qty }
POST /orders
```

---

**End of guide.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anushkrishnav)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/anushkrishnav)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
