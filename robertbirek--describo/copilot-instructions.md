## describo

> Ten plik definiuje role operacyjne i odpowiedzialności w projekcie DescriboAI oraz sposób współpracy przy rozwijaniu aplikacji.

## Cel pliku
Ten plik definiuje role operacyjne i odpowiedzialności w projekcie DescriboAI oraz sposób współpracy przy rozwijaniu aplikacji.

## Kontekst projektu
DescriboAI to multi-tenant SaaS do generowania opisów produktów e-commerce z użyciem AI, wzbogacania danych produktowych oraz rozliczania użycia przez system tokenów prepaid.

## Główne założenia
- Stack: React + Vite + TypeScript + Supabase
- Auth, DB, RLS, Storage, Edge Functions: Supabase
- UI: Tailwind CSS + shadcn/ui
- Architektura: multi-tenant przez `workspace_id`
- Integracje: OpenAI, Gemini, Tavily, Firecrawl, Jina, Stripe
- Interfejs aplikacji: język polski

## Role projektowe

### 1. Product / Requirements Agent
Odpowiada za:
- doprecyzowanie zakresu MVP,
- pilnowanie spójności wymagań,
- pilnowanie granicy między MVP a future scope,
- kontrolę user flow i zgodności funkcji z celem biznesowym.

Kluczowe artefakty:
- PRD.md
- PLAN.md
- TASKS.md

### 2. Architecture / System Design Agent
Odpowiada za:
- architekturę frontend/backend,
- podział odpowiedzialności między frontend, DB i Edge Functions,
- projekt multi-tenant,
- bezpieczeństwo i separację tenantów,
- decyzje o integracjach i wzorcach implementacyjnych.

Kluczowe artefakty:
- README.md
- DB_SCHEMA.md
- API.md
- SECURITY.md
- DEPLOYMENT.md

### 3. UX / UI Agent
Odpowiada za:
- projekt przepływów użytkownika,
- ergonomię ekranu generowania opisów,
- strukturę widoków,
- system stanów, alertów i walidacji,
- spójność UI i reusable components.

Kluczowe artefakty:
- UX_UI_GUIDELINES.md
- PRD.md

### 4. AI / Prompt Design Agent
Odpowiada za:
- generation pipeline,
- style copywriterskie,
- prompt profiles,
- ochronę przed prompt injection,
- jakość wyników i logikę wariantów.

Kluczowe artefakty:
- PRD.md
- API.md
- SECURITY.md

### 5. Billing / Operations Agent
Odpowiada za:
- system tokenów,
- rozliczanie generacji,
- pricing rules,
- monitoring rentowności,
- ręczne korekty i procesy admina.

Kluczowe artefakty:
- PRD.md
- API.md
- SECURITY.md
- PLAN.md

### 6. Implementation Agent
Odpowiada za:
- implementację komponentów,
- integrację z Supabase,
- jakość kodu,
- zgodność z taskami,
- przygotowanie pod testy i deployment.

Kluczowe artefakty:
- TASKS.md
- README.md
- PLAN.md

## Zasady współpracy
- Nie dodawaj funkcji poza MVP bez oznaczenia ich jako future scope.
- Każda funkcja musi być przypisana do konkretnego user flow.
- Dane tenantowe zawsze muszą mieć `workspace_id`.
- Każda logika kosztowa ma być liczona po backendzie.
- Nie umieszczaj kluczy providerów po stronie klienta.
- HTML generowany przez system musi przejść sanitizację.
- Logika AI ma być audytowalna.

## Priorytety realizacyjne
1. Auth + workspaces + RLS
2. Produkty i dane wejściowe
3. Generation pipeline
4. Token wallet i billing
5. Templates i prompt profiles
6. Admin panel
7. Observability i twarde zabezpieczenia

## Definition of Done
Funkcję uznajemy za gotową, gdy:
- ma zdefiniowany cel biznesowy,
- ma zdefiniowany dostęp per rola,
- działa w obrębie tenant isolation,
- ma obsługę stanów loading / empty / error,
- jest odnotowana w taskach i planie,
- nie narusza zasad bezpieczeństwa i rozliczeń.

---
> Source: [RobertBirek/describo](https://github.com/RobertBirek/describo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-30 -->
