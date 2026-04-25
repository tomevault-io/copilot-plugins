## voices-vercel

> 🛡️ Code Integrity & Forensic Audit Mandate - The unshakeable law for code discipline, ID-First Handshake, and stateless architecture.


# 🛡️ CODE INTEGRITY & ATOMIC AUDIT

Dit is de wet voor technische discipline en integriteit binnen het ecosysteem. Geen excuses, geen slop.

## 🔍 DE ATOMIC CHECK (VÓÓR ELKE EDIT)

### 0. COGNITIVE INTERRUPT (The Source of Truth Mandate)
Vóórdat je een UI-component aanpast om een link, tekst, of weergave-logica te fixen, MOET je je denkproces pauzeren en het volgende bepalen:
- **Komt deze data eigenlijk uit de database?** (bijv. `nav_menus`, `slug_registry`, `market_configs`, `content_articles`).
- **Zo ja:** Dan is het STRIKT VERBODEN om de UI aan te passen met if/else of ternaries (bijv. `isStudio ? '/studio' : '/'`). Je MOET de fix doorvoeren in de database of de API-bridge.
- **Bewijs:** Je eerste actie moet een database-query of config-inspectie zijn, géén UI-edit.

### 1. Forensic Root-Cause: Scan naar de bron (DB, API, of State). Gebruik `system_events` of rauwe SQL. Gissen is verboden.
### 2. Symbol Trace: Traceer elk symbool (markt, taal, config) terug naar de `MarketManager`.
### 3. Hardcode Hunt: Zoek naar verboden strings (`voices.be`, hardcoded emails/telefoons).
### 4. ISO Verify: Taal-codes MOETEN strikt ISO-5 zijn (`nl-BE`).
### 5. Anti-Pleister: Defensieve guards zijn secundair. De fix MOET structureel zijn (DB herstel of strikt TS Contract).

## 🛠️ DE TECHNISCHE WETTEN

*   **ID-First Handshake**: Gebruik uitsluitend UUIDs voor interne logica. Slugs zijn voor de URL.
*   **Pragmatic HTML**: Standaard HTML-tags (`div`, `span`, `p`, `h1`-`h6`) zijn TOEGESTAAN met de juiste Tailwind classes voor simpele layout en tekst. Gebruik `LayoutInstruments` voornamelijk voor complexe, herbruikbare blokken of specifieke interacties.
*   **The Snake-Camel Boundary**: Alle database kolommen, Drizzle schema properties en API payloads MOETEN `snake_case` gebruiken (`first_name`). Echter, Drizzle table exports (`export const actorLanguages`) en interne React state (`const [firstName]`) MOETEN `camelCase` blijven. Dit voorkomt Drizzle query crashes en respecteert React conventies.
*   **Eén Drizzle-schema**: alle `pgTable` / `relations` in **`packages/database/schema.ts`**. **`packages/database/src/schema/index.ts`** is alleen `export * from '../../schema'` (compat-pad). Geen tweede handgeschreven schema-laag. Canon: **`311-ORDERS-TABLE-AND-DRIZZLE-SSOT.mdc` §5**.
*   **Nuclear Speed**: Garandeer 100ms LCP. Gebruik `next/dynamic` met `ssr: false` voor zware instrumenten.
*   **Anti-Drift**: Gebruik `db.execute(sql...)` bij twijfel over ORM-caching (Pooler 6543).
*   **Theme Parity**: UI is pas correct als light en dark allebei leesbaar zijn (zie `301-THEME-PARITY.mdc`). Visueel bewijs voor beide modi is verplicht.
*   **Anti-TDZ (Hoisting Mandate)**: Gebruik voor functies en event handlers binnen React componenten ALTIJD de standaard `function myHandler() {}` syntax in plaats van `const myHandler = () => {}`. Dit garandeert function hoisting en voorkomt 'Temporal Dead Zone' (TDZ) crashes na agressieve Next.js minificatie.
*   **Database Script Hygiene**: Elk tijdelijk of utility script dat de database raakt MOET eindigen met `process.exit(0)` om te voorkomen dat de Supabase connection pool volloopt ("Max client connections reached").
*   **Nuclear Data Safety**: ELKE database-mutatie op meer dan 1 rij vereist een Dry-Run en expliciete bevestiging (zie rule 002 §41).

### Server DB-toegang: `@db` vs `getTable` (convergentie)

- **Nieuwe en actief onderhouden servercode**: gebruik `import { db } from '@db/index'` en tabellen uit `@db/schema` (Drizzle-exportnamen; kolommen volgens `packages/database/schema.ts`).
- **`getTable('…')` in `voices-config`**: legacy/dynamische lader; retour kan **`null`** (Edge/client)  -  bouw **geen** nieuwe kritieke paden hierop. Raak je een bestand aan: migreer waar haalbaar naar `@db` in dezelfde taak.
- **Referentie-migraties (batch)**: o.a. `push-service.ts`, `security-service.ts`, `availability-service.ts`, `db-service.ts` (`getTaxonomies`), `waitlist-service.ts`  -  tabellen via `@db/schema`.

## 🚀 DE PUSH-PROCEDURE

**Leidend:** `.cursor/rules/804-OWNER-LOCAL-ONLY-NO-PUSH.mdc`  -  **geen `git push`** (dus geen automatische **Vercel**-deploy via GitHub) **tenzij de owner dat expliciet vraagt**.

Je mag pas committen als:
1.  `npm run lint` en `npm run type-check` succesvol zijn.
2.  `npx tsx scripts/quick-check.ts` volledig groen is.
3.  De versie is verhoogd in `package.json` en `Providers.tsx`.
4.  **`git push origin main`** (branch = `main`) **alleen** wanneer de owner **expliciet** pushen/deploy naar Vercel vraagt **én** de gates groen zijn; volg anders **803** op feature branch. **Niet** zelf pushen om productie “bij te houden”. Als de owner deploy vraagt: geen nutteloze “mag ik pushen?”-ronde  -  voer 802 uit.

## 🚫 VERBODEN PATRONEN (ZERO TOLERANCE)

*   **GEEN** `any` types.
*   **GEEN** relatieve imports naar andere packages (`../../packages`). Gebruik `@db`.
*   **GEEN** hashtags in URLs (`#demos`). Gebruik clean routes.
*   **GEEN** `console.log` in productie. Gebruik `ServerWatchdog`.

"Code is ofwel Masterclass, ofwel Slop. Er is geen tussenweg."


## 🔒 ZERO-LEAK DTO MANDATE (Data Privacy)
*   **Geen Spread Operators**: Het is STRIKT VERBODEN om database-objecten direct naar de frontend te sturen via spread operators (bijv. `return { ...actor }`).
*   **Expliciete Mapping**: Elke API-route die data naar de client stuurt, MOET een expliciete DTO (Data Transfer Object) mapping gebruiken. Je benoemt elk veld dat naar buiten mag (bijv. `id: actor.id, name: actor.name`).
*   **De Zwarte Lijst**: De volgende velden mogen NOOIT de publieke frontend bereiken: `email` (van actors), `vat_number`, `internal_notes`, `admin_meta`, `cost` (COG), `profit`, `user_id`, en wachtwoorden/tokens.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/johfrah) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
