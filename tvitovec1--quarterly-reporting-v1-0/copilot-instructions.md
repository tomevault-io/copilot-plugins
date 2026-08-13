## quarterly-reporting-v1-0

> Tento soubor poskytuje konkrétní, okamžitě použitelné instrukce pro AI coding agenty a lidské přispěvatele pracující v tomto repozitáři. Pokrývá setup, vývojové workflow, testování a validační kroky specifické pro tento projekt.

# AGENTS.md

## Účel

Tento soubor poskytuje konkrétní, okamžitě použitelné instrukce pro AI coding agenty a lidské přispěvatele pracující v tomto repozitáři. Pokrývá setup, vývojové workflow, testování a validační kroky specifické pro tento projekt.

> **Pravidlo:** Pokud přidáš novou funkci, endpoint nebo změníš strukturu projektu, aktualizuj příslušný AGENTS.md (kořenový, `frontend/AGENTS.md` nebo `backend/AGENTS.md`).

---

## Popis projektu

**Automatizace updatu investičního čtvrtletníku** — webová aplikace pro AI-asistovanou tvorbu a správu investičního čtvrtletníku. Analytici zadávají instrukce a podkladové dokumenty pro každou z 8 kapitol; AI (OpenAI GPT-4o) generuje textové návrhy, které editor reviduje a schvaluje. Finální čtvrtletník se exportuje do PDF nebo Word.

---

## Mapa repozitáře

```
/
├── frontend/               # React 18 + Vite + TypeScript + Tailwind CSS
│   ├── src/
│   │   ├── pages/          # Stránky aplikace (Dashboard, ChapterEditor, ...)
│   │   ├── components/     # Znovupoužitelné UI komponenty
│   │   ├── hooks/          # Custom React hooks (useEditions, useChapter, ...)
│   │   ├── lib/            # API klient (Axios), utility
│   │   └── types/          # TypeScript typy (api.ts, domain.ts)
│   ├── .env.example        # Šablona env proměnných frontendu
│   └── AGENTS.md           # Instrukce specifické pro frontend
│
├── backend/                # NestJS + TypeScript + Prisma ORM
│   ├── src/
│   │   ├── auth/           # JWT autentizace, guards, strategie
│   │   ├── editions/       # CRUD pro vydání čtvrtletníku
│   │   ├── chapters/       # Správa kapitol, workflow schválení
│   │   ├── documents/      # Upload a parsování podkladových dokumentů
│   │   ├── ai/             # OpenAI integrace, prompt management
│   │   ├── export/         # PDF a DOCX export
│   │   ├── prisma/         # PrismaService (NestJS modul)
│   │   └── common/         # Guards, filters, interceptors, decorators
│   ├── prisma/
│   │   ├── schema.prisma   # Databázové schema
│   │   ├── migrations/     # Prisma migrace
│   │   └── seed.ts         # Seed data pro vývoj
│   ├── uploads/            # Nahrané podkladové dokumenty (NIKDY necommitovat)
│   ├── .env.example        # Šablona env proměnných backendu
│   └── AGENTS.md           # Instrukce specifické pro backend
│
├── .gitignore
├── README.md
└── AGENTS.md               # Tento soubor — přehled celého projektu
```

---

## Nasazené aplikace

- Název: `quarterly-reporting-v1.0`
- Lokální cesta: `~/apps/quarterly-reporting-v1.0`
- Git remote: `git@github.com:tvitovec1/quarterly-reporting-v1.0.git`
- Ověřená struktura po naklonování: `frontend/`, `backend/`
- Veřejná URL: `https://vitovec.aibr.cz`
- Backend API přes reverse proxy: `https://vitovec.aibr.cz/api`

---

## Quickstart

**Prerekvizity:**
- Node.js 20 LTS
- npm 9+
- PostgreSQL 15+ (lokálně nebo Docker)

```bash
# 1. Naklonuj repozitář a nainstaluj závislosti
git clone <repo-url>
cd <repo>

# 2. Backend — setup
cd backend
cp .env.example .env          # vyplň hodnoty (viz sekce Env proměnné)
npm install
npx prisma migrate dev        # vytvoří DB a spustí migrace
npx prisma db seed            # seed data: 3 uživatelé, 1 vzorové vydání
npm run start:dev             # backend běží na http://localhost:3000

# 3. Frontend — setup (nový terminál)
cd frontend
cp .env.example .env
npm install
npm run dev                   # frontend běží na http://localhost:5173
```

---

## Časté příkazy

### Instalace
```bash
cd backend && npm install
cd frontend && npm install
```

### Docker
```bash
# Zkopíruj šablonu proměnných pro Docker Compose
cp .env.example .env

# Produkční reverse proxy a TLS
docker compose up -d traefik

# Sestavení image bez spuštění kontejnerů
docker compose build

# Ověření konfigurace Compose
docker compose config
```

### Vývoj
```bash
# Backend
cd backend && npm run start:dev       # s hot-reload

# Frontend
cd frontend && npm run dev            # Vite dev server, port 5173
```

### Build
```bash
cd backend && npm run build           # TypeScript kompilace do dist/
cd frontend && npm run build          # Vite produkční build
```

### Lint / Formát
```bash
cd backend && npm run lint            # ESLint
cd frontend && npm run lint           # ESLint + TypeScript

cd backend && npm run lint:fix        # auto-oprava
```

### Typecheck
```bash
cd backend && npx tsc --noEmit
cd frontend && npx tsc --noEmit
```

### Testy
```bash
# Backend — unit testy
cd backend && npm test

# Backend — konkrétní soubor
cd backend && npm test -- --testPathPattern=auth.service

# Frontend — unit testy (Vitest)
cd frontend && npm test
```

### Databáze
```bash
cd backend
npx prisma migrate dev                        # spustit pending migrace
npx prisma migrate dev --name add_field       # vytvořit novou migraci
npx prisma db seed                            # seed data
npx prisma studio                             # GUI prohlížeč databáze
npx prisma generate                           # regenerovat Prisma klienta
```

---

## Testování & TDD workflow

Následuj **Red → Green → Refactor**:

1. **Red** — Napiš failing test popisující očekávané chování.
2. **Green** — Napiš minimální kód, aby test prošel.
3. **Refactor** — Vyčisti kód, testy musí zůstat zelené.

**Bug fixy:** Vždy začni failing testem reprodukujícím bug, pak teprve piš opravu.

### Umístění testů

| Vrstva | Umístění | Runner | Příkaz |
|--------|----------|--------|--------|
| Backend unit | `backend/src/**/*.spec.ts` | Jest | `cd backend && npm test` |
| Frontend unit | `frontend/src/**/*.test.ts(x)` | Vitest | `cd frontend && npm test` |

### Spuštění cíleného testu
```bash
# Backend — konkrétní modul
cd backend && npm test -- --testPathPattern=chapters.service.spec

# Frontend — konkrétní soubor
cd frontend && npm test -- chapter-editor.test.tsx
```

---

## Pravidla kvality kódu

- **Všechny PR musí projít:** lint, typecheck, test, build.
- **Žádné `any`** — použij striktní TypeScript; `unknown` s type guardy pokud nutné.
- **Žádné `console.log`** v commitnutém kódu — použij NestJS Logger na backendu; odstraň před PR.
- **Pořadí importů** je vynutěno ESLintem. Neřaď ručně.
- **Sdílené typy** drž v `frontend/src/types/api.ts` pro API odpovědi. Nezdvojuj DTOs.
- **Žádné breaking API změny** bez koordinace frontendu a backendu. Při změně tvaru odpovědi nejprve aktualizuj typy v `frontend/src/types/api.ts`.

## Styl kódu (Prettier + ESLint)

- Jednoduché uvozovky, středníky, 2-mezerový indent.
- Arrow funkce preferovány.
- Pouze funkcionální React komponenty (žádné class components).
- Striktní rovnost (`===`).
- PascalCase pro React komponenty a TypeScript typy/interfaces.
- camelCase pro funkce, proměnné, metody.
- Singulární pojmenování pro DB tabulky/entity.

---

## Bezpečnostní pravidla — POVINNÉ

> ⚠️ **Tato pravidla jsou absolutní. Výjimky neexistují.**

- **NIKDY necommituj `.env` soubory.** Jsou v `.gitignore`. Kontroluj `git status` před každým commitem.
- **Všechna tajemství (API klíče, hesla, JWT secret) výhradně přes environment variables.** Nikdy natvrdo do kódu ani do komentářů.
- **NIKDY necommituj obsah složky `backend/uploads/`** — obsahuje dokumenty klientů. Pouze `.gitkeep` je commitnutý.
- **OpenAI API klíč** uchovávej pouze v `backend/.env` pod proměnnou `OPENAI_API_KEY`. Nikde jinde.
- **JWT_SECRET** musí být v produkci náhodný řetězec min. 64 znaků. Nikdy nepoužívej výchozí nebo jednoduché hodnoty.
- Při přidání nové env proměnné: 1) přidej do `.env.example` s placeholderem, 2) zdokumentuj zde v sekci níže.

---

## Env proměnné

Šablona: `backend/.env.example` a `frontend/.env.example`. Nikdy necommituj `.env`.

### Backend (`backend/.env`)

| Proměnná | Popis | Příklad |
|----------|-------|---------|
| `PORT` | Port backendu | `3000` |
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://user:pass@localhost:5432/ctvrtletnik_db` |
| `JWT_SECRET` | Podpisový klíč pro JWT tokeny (min. 64 znaků) | `change-me-in-production-...` |
| `JWT_EXPIRES_IN` | Platnost JWT tokenu | `7d` |
| `OPENAI_API_KEY` | API klíč pro OpenAI (GPT-4o) | `sk-...` |
| `CONTACT_FORM_WEBHOOK_URL` | Production n8n webhook pro kontaktní formulář | `https://n8n.vitovec.aibr.cz/webhook/contact-form` |
| `FRONTEND_URL` | URL frontendu pro CORS | `http://localhost:5173` |
| `NODE_ENV` | Prostředí | `development` / `production` |

### Frontend (`frontend/.env`)

| Proměnná | Popis | Příklad |
|----------|-------|---------|
| `VITE_API_URL` | Základní URL backend API | `http://localhost:3000/api` |

### Přidání nové env proměnné

1. Přidej do `.env.example` s placeholderem a komentářem.
2. Na backendu přidej validaci v `backend/src/config/` (NestJS ConfigModule).
3. Zdokumentuj proměnnou v tabulce výše.

---

## Doménový model (přehled)

```
User          — uživatel systému (role: ADMIN | EDITOR | APPROVER | VIEWER)
Edition       — vydání čtvrtletníku (status: DRAFT | IN_REVIEW | APPROVED | PUBLISHED)
Chapter       — jedna z 8 kapitol vydání (status: DRAFT | GENERATING | IN_REVIEW | APPROVED | REJECTED)
ChapterTemplate — výchozí pokyny pro konkrétní typ kapitoly napříč vydáními
Document      — podkladový dokument nahraný ke kapitole (PDF, DOCX, XLSX, TXT)
```

**8 typů kapitol (ChapterType):**
`INTRO` | `GLOBAL_MACRO` | `CZ_MACRO` | `ASSET_PERFORMANCE` | `INVESTMENT_THEME` | `REVIEW_OUTLOOK` | `CLOSING` | `MARKET_PERFORMANCE_OVERVIEW`

Při vytvoření nového vydání (`POST /api/editions`) se automaticky vytvoří všech 8 kapitol v pořadí 1–8.

`MARKET_PERFORMANCE_OVERVIEW` slouží jako interní zdroj dat: generuje tabulkové makro reporty, nevyžaduje vlastní schválení a nevstupuje do finálního exportu.

Kapitola `GLOBAL_MACRO` může volitelně použít tabulku 1 z `MARKET_PERFORMANCE_OVERVIEW` přes checkbox `Použij data z Přehledu trhu`.
Kapitola `CZ_MACRO` může volitelně použít tabulku 2 z `MARKET_PERFORMANCE_OVERVIEW` přes stejný checkbox.

---

## API přehled (backend, prefix `/api`)

| Metoda | Endpoint | Popis | Role |
|--------|----------|-------|------|
| `GET` | `/health` | Health check | — |
| `POST` | `/contact` | Kontaktní formulář; přepošle payload na n8n webhook | — |
| `POST` | `/auth/login` | Přihlášení, vrací JWT | — |
| `POST` | `/auth/register` | Registrace uživatele | ADMIN |
| `GET` | `/auth/me` | Profil přihlášeného uživatele | všechny |
| `GET` | `/editions` | Seznam vydání | všechny |
| `POST` | `/editions` | Vytvoření vydání + 8 kapitol | EDITOR, ADMIN |
| `GET` | `/editions/:id` | Detail vydání s kapitolami | všechny |
| `PATCH` | `/editions/:id` | Aktualizace metadat | EDITOR, ADMIN |
| `DELETE` | `/editions/:id` | Smazat celé vydání | EDITOR, ADMIN |
| `POST` | `/editions/:id/submit` | Odeslat ke schválení | EDITOR, ADMIN |
| `POST` | `/editions/:id/publish` | Publikovat | APPROVER, ADMIN |
| `POST` | `/editions/:id/export` | Export PDF nebo DOCX | EDITOR, APPROVER, ADMIN |
| `GET` | `/editions/:id/chapters` | Kapitoly vydání | všechny |
| `GET` | `/chapters/:id` | Detail kapitoly | všechny |
| `PATCH` | `/chapters/:id` | Uložit briefing / finální text | EDITOR |
| `POST` | `/chapters/:id/generate` | Spustit AI generování | EDITOR |
| `POST` | `/chapters/:id/generate-report` | Vygenerovat tabulkový report pro kapitolu 8 | EDITOR |
| `POST` | `/chapters/:id/approve` | Schválit kapitolu | EDITOR, APPROVER, ADMIN |
| `POST` | `/chapters/:id/reject` | Zamítnout kapitolu | APPROVER, ADMIN |
| `POST` | `/chapters/:id/documents` | Upload podkladového dokumentu | EDITOR |
| `GET` | `/chapters/:id/documents` | Seznam dokumentů | všechny |
| `DELETE` | `/documents/:id` | Smazat dokument | EDITOR, ADMIN |

---

## Checklist před každým PR

Spusť před každým PR. CI spouští stejné kontroly.

```bash
# Backend
cd backend
npm run lint
npx tsc --noEmit
npm test
npm run build

# Frontend
cd frontend
npm run lint
npx tsc --noEmit
npm test
npm run build
```

Všechny příkazy musí skončit s exit code 0. Oprav všechny chyby před žádostí o review.

## Docker deployment

- Kořenový `docker-compose.yml` definuje služby `traefik`, `postgres`, `backend` a `frontend` a sdílenou síť `traefik-network`.
- `backend/Dockerfile` používá multi-stage build a v produkci spouští `node dist/src/main.js`.
- `frontend/Dockerfile` používá multi-stage build a finální artefakty servíruje přes Nginx.
- `postgres` běží jako interní databázová služba v Compose a ukládá data do volume `postgres-data`.
- `backend` čeká na healthy databázi a při startu provede `npx prisma migrate deploy`.
- Traefik publikuje pouze porty `80` a `443`; `frontend` a `backend` nemají přímé mapování portů na hosta.
- Traefik používá file provider z `traefik/dynamic.yml`, automatický redirect `web -> websecure` a Let's Encrypt ACME HTTP challenge se storage v `/letsencrypt/acme.json`.
- Docker labels na službách zůstávají připravené pro budoucí návrat k auto-discovery, ale na tomto hostu je Traefik Docker provider nekompatibilní s Docker daemon API.
- Router `frontend` obsluhuje `Host("vitovec.aibr.cz")`; router `backend` obsluhuje `Host("vitovec.aibr.cz") && PathPrefix("/api")`.
- Kořenový `.env.example` slouží jako šablona pro Docker Compose; produkční hodnoty patří do lokálního `.env`, který se necommitjuje.

---

## Dokumentační hygiena

- **README.md** — Pro lidi: přehled projektu, setup pro nováčky, architektonická rozhodnutí.
- **AGENTS.md** (tento soubor) — Pro agenty: přesné příkazy, cesty k souborům, pravidla co dělat / nedělat.
- **frontend/AGENTS.md** — Detailní instrukce pro práci na frontendu.
- **backend/AGENTS.md** — Detailní instrukce pro práci na backendu, seznam endpointů, DB schema.

> **Pravidlo:** Pokud při práci v repozitáři narazíš na opakující se problém nebo neočekávané chování, přidej krátkou poznámku do sekce **Troubleshooting** níže.

---

## Troubleshooting

### Prisma klient není synchronizovaný

Pokud vidíš `PrismaClientKnownRequestError` o chybějících polích po stažení nových migrací:

```bash
cd backend
npx prisma migrate dev
npx prisma generate
```

### Port je obsazený

Backend defaultně běží na 3000, frontend na 5173. Pokud jsou porty obsazeny:

```bash
PORT=3001 npm run start:dev     # backend
# nebo v frontend/.env změň VITE_API_URL
```

### CORS chyba v prohlížeči

Zkontroluj, že `FRONTEND_URL` v `backend/.env` odpovídá přesné URL frontendu (včetně portu). Restartuj backend po změně.

### OpenAI API vrací chybu 401

Zkontroluj `OPENAI_API_KEY` v `backend/.env`. Klíč musí začínat `sk-` a mít platný kredit na účtu.

### Generování textu se zasekne ve stavu `GENERATING`

Pokud backend crashne během generování, kapitola zůstane ve stavu `GENERATING`. Manuální oprava:

```bash
cd backend
npx prisma studio    # najdi kapitolu a změň status zpět na DRAFT
```

### `npm install` selže na lockfile mismatch

```bash
npm install --frozen-lockfile    # CI režim — selže při neshodě
npm install                      # lokálně — aktualizuje lockfile
```

---
> Source: [tvitovec1/quarterly-reporting-v1.0](https://github.com/tvitovec1/quarterly-reporting-v1.0) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
