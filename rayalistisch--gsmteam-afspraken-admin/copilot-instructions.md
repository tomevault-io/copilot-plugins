## gsmteam-afspraken-admin

> Next.js admin applicatie voor GSM Team (Enschede). Beheer van reparatie-aanvragen die binnenkomen via de Shopify webshop op gsmteam.nl. Draait op Vercel, database op Supabase, mail via Mailgun.

# GSM Team – Afspraken Admin

Next.js admin applicatie voor GSM Team (Enschede). Beheer van reparatie-aanvragen die binnenkomen via de Shopify webshop op gsmteam.nl. Draait op Vercel, database op Supabase, mail via Mailgun.

---

## Tech stack

| Laag | Technologie |
|------|-------------|
| Framework | Next.js 16 (App Router) |
| Database | Supabase (PostgreSQL) |
| Auth | PIN-code via httpOnly cookie |
| Mail | Mailgun (EU regio) |
| PDF | jsPDF + jspdf-autotable |
| Hosting | Vercel |
| Shopify | Embedded via App Bridge (iframe) |

---

## Omgevingsvariabelen (`.env.local` / Vercel)

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...          # alleen voor sidebar badge (pendingCount)
SUPABASE_URL=https://xxx.supabase.co           # server-side
SUPABASE_SERVICE_ROLE_KEY=eyJ...               # bypast RLS, alleen server-side gebruiken

# Auth
ADMIN_PIN=1234                                  # pincode voor inloggen
AUTH_SECRET=<random hex>                        # waarde in de cookie na inloggen

# Mailgun
MAILGUN_API_KEY=key-xxx
MAILGUN_DOMAIN=mail.gsmteam.nl
MAILGUN_REGION=eu                               # eu of us
MAIL_FROM=GSM Team <noreply@mail.gsmteam.nl>
NOTIFY_EMAIL=info@gsmteam.nl                   # CC bij goedkeuring/bevestiging
MAIL_DEBUG_TO=test@example.com                 # optioneel: stuurt alle mail naar dit adres

# Offerte-tokens
OFFER_SECRET=<random hex>                       # HMAC-sleutel voor accept/reject links
```

> **Let op:** `SUPABASE_SERVICE_ROLE_KEY` bypast Row Level Security. Nooit naar de browser sturen — alleen in server-side API routes gebruiken.

---

## Database (Supabase)

### Tabel: `repair_requests`

| Kolom | Type | Omschrijving |
|-------|------|--------------|
| id | uuid | primary key |
| created_at | timestamptz | aanmaakdatum |
| customer_name | text | naam klant |
| customer_email | text | e-mail klant |
| customer_phone | text | telefoon klant |
| brand | text | merk (Apple, Samsung…) |
| model | text | model (iPhone 14 Pro…) |
| color | text | kleur (Space Black…) |
| issue | text | reparatietype (Scherm, Batterij…) |
| quality | text | onderdeel-kwaliteit (Officieel / Compatible) |
| price_text | text | richtprijs als tekst (€79 / Op aanvraag) |
| preferred_date | text | voorkeursdatum (YYYY-MM-DD) |
| preferred_time | text | voorkeurstijd (HH:MM) |
| status | text | zie statussen hieronder |
| condition | text | staat van het toestel |
| warranty | text | garantie-info |
| notes | text | interne notities (admin) |

**Statussen:**
- `pending` — nieuw, wacht op admin
- `awaiting_approval` — offerte verstuurd, wacht op klantbevestiging
- `approved` — goedgekeurd (na directe goedkeuring of klantakkoord)
- `rejected` — afgewezen

> Als je een check constraint hebt op de `status` kolom, voeg `awaiting_approval` toe:
> ```sql
> ALTER TABLE repair_requests DROP CONSTRAINT IF EXISTS repair_requests_status_check;
> ALTER TABLE repair_requests ADD CONSTRAINT repair_requests_status_check
>   CHECK (status IN ('pending', 'awaiting_approval', 'approved', 'rejected'));
> ```

### Tabel: `repair_catalog`

Prijscatalogus per merk/model/kleur/reparatietype. Wordt gebruikt door de Shopify-widget om richtprijzen te tonen.

| Kolom | Type | Omschrijving |
|-------|------|--------------|
| id | uuid | primary key |
| brand | text | merk |
| model | text | model |
| color | text | kleur |
| repair_type | text | reparatietype |
| price | numeric | prijs in euro |
| show_quality | boolean | toon kwaliteitskeuze in offerte/PDF |

**RLS:** Beide tabellen hebben Row Level Security aan. Alle data-toegang gaat via server-side API routes met de service role key.

---

## Authenticatie

PIN-gebaseerd. Middleware (`middleware.ts`) beschermt alle routes behalve:
- `/login`
- `/offer-confirm` (publieke klantpagina)
- `/api/auth/pin`
- `/api/create-request` (Shopify-facing)
- `/api/catalog` (Shopify-facing)
- `/api/offer-confirm` (publieke klant-actie)

Na inloggen wordt een httpOnly cookie `gsm_pin_auth` gezet met `AUTH_SECRET` als waarde. SameSite=None;Secure zodat het werkt in Shopify-iframe (WKWebView).

---

## Pagina's

### `/` — Aanvragen (`app/page.tsx`)
Overzicht van alle reparatie-aanvragen. Filterbaar op status. Per aanvraag:
- Bewerken (prijs, datum, tijd, notities)
- **Stuur Offerte** — verstuurt offerte-mail, status → `awaiting_approval`
- Goedkeuren — directe goedkeuring + bevestigingsmail
- Afwijzen — status → `rejected` + afwijzingsmail

Mobiel: mobiele header + scrollbare filter-pills bovenaan + bottom navigation.

### `/planning` — Planning (`app/planning/page.tsx`)
Maandkalender met goedgekeurde afspraken als chips in dag-cellen. Klik op een chip → modal met klantdetails. Alleen `approved` aanvragen met `preferred_date`.

### `/catalogus` — Catalogus (`app/catalogus/page.tsx`)
Beheer van de prijscatalogus. Apparaten en reparaties toevoegen/bewerken/verwijderen. Bulk prijsupdate per modelserie.

### `/omzet` — Omzet (`app/omzet/page.tsx`)
Omzetdashboard op basis van goedgekeurde aanvragen. KPI-kaarten + staafdiagrammen per maand, merk en reparatietype.

### `/offer-confirm` — Klantbevestiging (`app/offer-confirm/page.tsx`)
**Publieke pagina** (geen login vereist). Toont resultaat nadat klant op akkoord/afwijzen heeft geklikt in de offerte-mail. Query param: `?result=accepted|rejected|invalid`.

### `/login` — Pincode (`app/login/page.tsx`)
PIN-invoerscherm met nummerkeypad. Werkt ook in Shopify WKWebView (150ms delay voor cookie-timing).

---

## API routes

### Publiek (geen auth)

| Route | Methode | Omschrijving |
|-------|---------|--------------|
| `/api/create-request` | POST | Aanvraag aanmaken vanuit Shopify-widget. Stuurt bevestigingsmail naar klant + notificatie naar info@gsmteam.nl |
| `/api/catalog` | GET | Prijscatalogus opvragen. `?brands=1` alle merken, `?models=1&brand=X` modellen, `?colors=1` alle kleuren |
| `/api/offer-confirm` | GET | Klant accepteert/wijst offerte af via token-link uit mail |

### Admin-protected

| Route | Methode | Omschrijving |
|-------|---------|--------------|
| `/api/requests` | GET | Alle aanvragen ophalen. `?status=pending|approved|rejected|awaiting_approval|all` |
| `/api/approve` | POST | Aanvraag direct goedkeuren + bevestigingsmail + PDF |
| `/api/reject` | POST | Aanvraag afwijzen + afwijzingsmail |
| `/api/offer` | POST | Offerte versturen ter klantgoedkeuring (status → `awaiting_approval`) |
| `/api/update-request` | POST | Prijs/datum/tijd/notities bijwerken |
| `/api/catalog` | POST/PATCH/DELETE | Catalogusitems beheren |
| `/api/preview-pdf` | GET | PDF-preview in browser |

---

## Mail-flows

### 1. Nieuwe aanvraag (Shopify → klant)
`/api/create-request` → bevestigingsmail naar klant + interne notificatie naar `NOTIFY_EMAIL`.

### 2. Directe goedkeuring (admin → klant)
`/api/approve` → bevestigingsmail + PDF-bijlage naar klant, CC naar `NOTIFY_EMAIL`.

### 3. Offerte-flow (admin → klant → bevestiging)
1. Admin klikt "Stuur Offerte" → `/api/offer`
2. Offerte-mail met "Ja, akkoord" en "Nee, afwijzen" knoppen + PDF naar klant
3. Klant klikt knop → `/api/offer-confirm?id=X&token=Y&action=accept|reject`
4. Token verificatie via HMAC-SHA256 (geen DB-opslag nodig)
5. **Akkoord:** status → `approved` + bevestigingsmail + PDF, CC naar `NOTIFY_EMAIL`
6. **Afwijzen:** status → `rejected`, redirect naar `/offer-confirm?result=rejected`

### 4. Afwijzing (admin → klant)
`/api/reject` → afwijzingsmail met optionele reden.

---

## PDF generatie (`app/lib/offer-pdf.ts`)

- `buildOfferPdf(input)` — genereert PDF als `Buffer` (jsPDF + autotable)
- `buildOfferEmail(data)` — HTML bevestigingsmail (na goedkeuring/akkoord)
- `buildOfferQuoteEmail(data)` — HTML offerte-mail met akkoord/afwijzen knoppen

Logo: `public/logo.jpeg` (of `.png`, `.jpg`) wordt automatisch ingelezen. Valt terug op tekst "GSMTEAM" als het bestand niet bestaat.

---

## Shopify-integratie

De Shopify-widget op gsmteam.nl POST-t aanvragen naar `/api/create-request`. CORS is geconfigureerd voor:
- `https://gsmteam.nl`
- `https://www.gsmteam.nl`
- `https://gsm-team-2.myshopify.com`

De admin draait embedded in Shopify als iframe-app via App Bridge (`app/components/AppBridgeInit.tsx`).

---

## Navigatie (DashboardShell)

`app/components/DashboardShell.tsx` bevat:
- **Desktop:** sidebar met logo + nav-items
- **Mobiel:** bottom navigation bar (fixed, safe-area-aware)
- Pending-badge op "Aanvragen" (live via Supabase, iedere 30s gerefreshed)

---

## Lokaal ontwikkelen

```bash
npm install
npm run dev        # http://localhost:3000
npm run build      # productie build + type check
```

Vereiste `.env.local` variabelen: zie sectie Omgevingsvariabelen hierboven.

**Mail testen lokaal:** Zet `MAIL_DEBUG_TO=jouw@email.nl` — alle mails gaan dan naar dat adres.

**OFFER_SECRET lokaal:** Niet nodig — fallback op `AUTH_SECRET`.

---
> Source: [Rayalistisch/gsmteam-afspraken-admin](https://github.com/Rayalistisch/gsmteam-afspraken-admin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-12 -->
