## kannon

> > Documento di contesto vivo per Claude Code (e per chiunque legga la repo).

# CLAUDE.md · Kannon Hub Context

> Documento di contesto vivo per Claude Code (e per chiunque legga la repo).
> Aggiornalo ogni volta che cambia qualcosa di strutturale: nuova entità DB, nuova RPC, nuovo ruolo, nuova route, nuovo modulo applicato.
> **Convenzione**: questo file è la fonte di verità per le decisioni architetturali. Se contraddice un singolo file di codice, vince questo.

---

## 1. Cos'è Kannon

Kannon è un'agenzia B2B di user acquisition su TikTok per app mobile. Schiera decine/centinaia di creator su account TikTok freschi e niche-locked per generare views verso campagne dei clienti (Finanz, Easy Papiro, HAT Music, Tot Money, ecc.). I clienti pagano un fisso mensile + un CPM variabile. Kannon paga i creator un fisso + CPM proporzionato.

Il **Kannon Hub** (questa repo) è la piattaforma operativa interna che gestisce:
- pianificazione contenuti (calendario brief),
- esecuzione (creator portal),
- approvazione (client portal),
- scraping performance (Apify TikTok),
- pagamenti (entrate clienti + uscite creator),
- finance unificata,
- analytics multi-dimensionale.

URL produzione: `hub.thekannon.io`. URL landing pubblica: `thekannon.io`.

---

## 2. Stack tecnico

- **Frontend**: React 18 + TypeScript + Vite. UI da shadcn/ui + Tailwind CSS. State server: TanStack Query (react-query v5). Routing: react-router-dom v6. Charts: Recharts.
- **Backend**: Supabase (Postgres + Auth + RLS + Edge Functions in Deno + Storage). Tutte le query passano dal client `@/integrations/supabase/client`.
- **Scraping**: Apify (`clockworks/free-tiktok-scraper`). Edge function `scrape-tiktok` orchestra runs + polling background + ingestion (SP#5: no più webhook come meccanismo primario).
- **Deploy**: Lovable storicamente (preview). Ora il workflow è Claude Code → git push → Lovable preview / Vercel production.

Branding colori (per UI/landing, NON nel hub):
- bg `#DFDFDF`, ink `#000000`, accent `#FF2727`, accent-2 `#FFD0D0`.

Font (landing, NON hub): Inter (body), Instrument Serif italic (display accents), JetBrains Mono (eyebrow labels uppercase).

---

## 3. RBAC: i 7 ruoli

Definiti in `src/lib/roles.ts` come constants. Mai magic string. Usare sempre `ROLES.X` e `ROLE_GROUPS.Y`.

| Ruolo | Cosa fa | Default landing | Sidebar |
|---|---|---|---|
| `admin` | Owner della piattaforma. Accesso totale. | `/dashboard` | Tutte le sezioni + Finance + Settings |
| `team` | Operatori interni. Come admin ma senza Finance/Settings. | `/dashboard` | Tutto eccetto Finance/Settings |
| `outreach` | Single-purpose: solo Recruiting. | `/dashboard/recruiting` | Solo Recruiting |
| `closer` | Single-purpose: solo Closer. | `/dashboard/closer` | Solo Closer |
| `campaign_manager` | Pianifica contenuti, gestisce calendario, analytics format. | `/dashboard/content-calendar` | Campagne · Video Analytics · Calendario Contenuti |
| `creator` | Talento esterno. Solo portale dedicato. | `/creator` | Portale custom (CreatorArea) |
| `client` | Cliente Kannon. Solo portale dedicato. | `/client` | Portale custom (ClientArea) |

Helper functions:
- `canAccess(role, allowedRoles[])` — boolean check
- `isStaff(role)` — true se admin/team
- `ROLE_DEFAULT_ROUTE[role]` — landing post-login
- `ROLE_GROUPS.STAFF` = admin + team
- `ROLE_GROUPS.INTERNAL` = admin + team + outreach + closer + campaign_manager
- `ROLE_GROUPS.SINGLE_PURPOSE` = outreach + closer (NB: campaign_manager rimosso dopo SP#4)
- `ROLE_GROUPS.EXTERNAL` = creator + client

Le RLS Postgres sono **enforcement primario**. Le restrizioni UI sono solo UX. Mai fidarsi di gate solo lato client.

---

## 4. Entità DB principali

Mappa concettuale. Per schema completo consultare `supabase/migrations/`. Le migrations sono ordinate per timestamp.

### Auth e profili
- `auth.users` (Supabase native)
- `profiles` → 1:1 con `auth.users`, contiene full_name, role, ecc.
- `user_roles` → tabella ruoli (per `has_role()` SQL function)

### Business core
- `campaigns` → name, client_name, client_user_id (FK a auth.users), client_fixed, client_cpm, video_views_cap, monthly_spend_cap, start_date, status, **brief_threshold_views**, **brief_threshold_engagement**
- `creators` → name, status, user_id (FK), creator_fixed, creator_cpm, min_videos_per_day
- `campaign_creators` → junction N:N campagne ↔ creator
- `tiktok_accounts` → username, account_type ('creator'|'brand'|...), campaign_id, creator_id, is_active, last_scraped_at
- `contracts` + `contract_campaigns` + `contract_creators` + `contract_signatures` → contratti creator-Kannon, periodi di attività

### Video e performance
- `videos` → tiktok_account_id, tiktok_video_id (UNIQUE), views, likes, comments, published_at, window_expires_at, window_closed, views_final, last_scraped_at, **audio_id**, **audio_name**, **caption**, **hashtags[]**, content_tag
- `scraping_logs` → run_at, status (CHECK `running`/`success`/`error`), accounts_processed, videos_created, videos_updated, error_message, **run_id, dataset_id, started_at, completed_at, progress_note, triggered_by** (SP#5: status sync per polling background)

### Pagamenti
- `client_payments` → entrate da campagne. Generato da cicli. campaign_id, cycle_id, cycle_number, due_date, fixed_amount, cpm_views, cpm_amount, total_amount, is_paid, paid_at, **amount_override**, **notes_override**, **received_at**, **invoice_number**, **invoice_sent_at**, views_paid_cumulative
- `creator_payments` → uscite verso creator. creator_id, period_month, period_year, fixed_amount, cpm_amount, total_amount, is_paid, paid_at, **amount_override**, **notes_override**, **paid_via**
- `payment_cycles` → cicli campagna 30gg, campaign_id, cycle_start_date, cycle_end_date, is_last_cycle

### Finance (post Super Prompt #2)
- `financial_entries` → entries manuali (type, category, description, amount, date, due_date, status, campaign_id, creator_id, brand_name, invoice_number, notes, **recurring_expense_id**)
- `recurring_expenses` → spese ricorrenti (name, amount, category, due_day, start_date, end_date, is_active, vendor, notes)
- `settings` → KV store globale (`key`, `value text`). Contiene `finance_cash` (jsonb in text), `apify_api_key`, ecc.
- `v_financial_movements` → VIEW unificata (UNION ALL di client_payments + creator_payments + financial_entries) con override applicato via COALESCE. Source-of-truth in lettura per dashboard Finance.

### Content (post Super Prompt #4) ✅
- `video_briefs` → entità centrale del modulo Calendario Contenuti. La "riga del Google Doc": campaign_id (FK campaigns ON DELETE CASCADE), planned_publish_date, week_label, reference_type (CHECK video/audio/video_audio/format_audio/format), reference_links jsonb, audio_id, expected_caption_keywords text[], format_id (FK video_formats ON DELETE SET NULL), title, copy_text (NOT NULL), caption, hashtags text[], visual_note, threshold_views_override, threshold_engagement_override, status (draft/in_review/approved/archived), created_by, created_at, updated_at. RLS: staff full; client SELECT su in_review/approved/archived della sua campagna (via `campaigns.client_profile_id = auth.uid()`) + UPDATE limitato per approvare; creator SELECT su in_review/approved delle campagne assegnate (via `creators.profile_id = auth.uid()`). Guard trigger `guard_client_brief_updates` (BEFORE UPDATE): impedisce ai client di modificare qualsiasi colonna eccetto `status` (e solo per transizioni `in_review` ⇄ `approved`). Lo staff bypassa il guard.
- `video_formats` → catalog pre-esistente. SP#4 aggiunge colonna **`is_active boolean NOT NULL DEFAULT true`** + policy RLS per `campaign_manager` (CRUD) e read-all authenticated.
- `content_topics` → seconda dimensione tag (name UNIQUE, is_active). Seed: Pricing, Onboarding, Social Proof, Pain Point, Use Case, Comparison.
- `brief_topics` → N:N brief ↔ topic (PK composta brief_id+topic_id)
- `brief_comments` → commenti su brief (author_id, author_role, body, resolved/resolved_by/resolved_at)
- `brief_change_requests` → richieste modifica su un brief (proposed_copy_text/caption/hashtags/visual_note, reason, status pending/accepted/rejected, resolution_note)
- `video_brief_matches` → N:N video pubblicato ↔ brief. match_method (audio_id/caption_keywords/manual), confidence, UNIQUE(video_id, brief_id)
- `videos` (SP#4): aggiunte colonne **audio_id, audio_name, caption, hashtags text[]** + index trigram su caption (`pg_trgm`).
- `campaigns` (SP#4): aggiunte **brief_threshold_views bigint DEFAULT 50000, brief_threshold_engagement numeric DEFAULT 5.0**.
- VIEW `v_brief_stats` → per-brief: matched_videos_count, total_effective_views, total_engagements, avg_engagement_pct, threshold_views/engagement (override brief → default campagna). Usata solo dentro RPC SECURITY DEFINER.
- Edge function `parse-briefs-from-text` (Anthropic Claude Haiku, `claude-3-5-haiku-latest`) per import bulk da paste testuale (SP#5 Part B). Staff-only. Richiede secret `ANTHROPIC_API_KEY`.
- Funzioni: `match_video_to_briefs(uuid)`, `rematch_all_unmatched_videos(int)`, RPC `get_content_calendar(uuid,date,date)`, `get_content_analytics(text,uuid,uuid,uuid)` (wrappa `get_campaign_manager_data` + breakdown format/topic), `get_content_insights(text,uuid)`, `notify_brief_event(uuid,text,text,text,text[])`.

### Notifiche
- `notifications` → esistente. Colonne: id, campaign_id, type, message, is_read, user_id, severity, link, meta jsonb, created_at. SP#4 inserisce via RPC `notify_brief_event` che risolve i destinatari (clients = `campaigns.client_profile_id`; creators = `creators.profile_id` via campaign_creators; staff = `user_roles` admin/team/campaign_manager) escludendo l'autore.

---

## 5. Meccanismi finanziari fondamentali

### CPM cliente vs CPM creator
- `campaigns.client_cpm`: euro per 1000 views che il cliente paga a Kannon (es. 2€)
- `creators.creator_cpm`: euro per 1000 views che Kannon paga al creator (es. 0.5€)
- Margine lordo unitario = `client_cpm - creator_cpm` (es. 1.5€ ogni 1000 views)

### Window 30 giorni del video
Logica in `src/lib/videoWindow.ts`. Ogni video ha 30 giorni di "vita CPM": dopo `published_at + 30d`, le views si freezano (`views_final`) e il video è `window_closed`. Lo scrape continua ma non aggiorna più il CPM.

- `effective_views(video, cap?)`:
  - se `window_closed` → `views_final ?? views ?? 0`
  - altrimenti → `views ?? 0`
  - se `cap` impostato → `min(views, cap)`
- `windowStatus`: `closed` | `closing` (≤24h) | `open`

### Cap per campagna
- `campaigns.video_views_cap`: tetto views per singolo video (es. 100.000). Sopra non si paga.
- `campaigns.monthly_spend_cap`: tetto spesa mensile cliente per CPM.

### Ciclo di pagamento cliente
Standard: cicli 30gg dalla `start_date` campagna. Per ciclo non ultimo: `client_fixed` + CPM su views nuove del periodo. Ultimo ciclo: solo CPM residuo, niente fisso.

Variante ToT (split): logica diversa, vedere `supabase/functions/regenerate-client-payments/index.ts`.

### Views paid cumulative
Per evitare che la stessa view venga conteggiata due volte tra cicli. `client_payments.views_paid_cumulative` cresce monotonicamente. Quando un ciclo viene pagato, snapshot delle views totali della campagna a quel momento.

### Override amount
Pattern non-distruttivo per correggere manualmente un pagamento auto-calcolato. La view `v_financial_movements` usa `COALESCE(amount_override, total_amount)`. Niente sovrascrittura: l'originale resta per audit.

### Spese ricorrenti
`recurring_expenses` con `due_day` 1-31. Funzione SQL `generate_recurring_expense_entries(months_ahead)` idempotente. Trigger AFTER INSERT/UPDATE per generazione immediata. pg_cron daily 06:00 UTC per maintenance. Gestisce edge case "31 di febbraio" col clamp al last day of month.

---

## 6. Scraping TikTok (Apify)

Edge function: `supabase/functions/scrape-tiktok/index.ts`.

Flow (post SP#5 · polling resiliente, niente più dipendenza dal webhook):
1. "Scrapa ora" (UI) → `handleStartWithPolling`: lancia run Apify SENZA webhook
2. INSERT subito in `scraping_logs` con `status='running'`, `started_at`, `run_id`; ritorna `{ ok, log_id, run_id }` (202) al client
3. `EdgeRuntime.waitUntil(pollAndProcess)`: ogni 10s GET `/v2/actor-runs/{runId}`, aggiorna `progress_note` ad ogni iter (max 15 min)
4. Su `SUCCEEDED` → `runScraping(datasetId)` scarica dataset paginato (1000/page), normalizza, dedupe per `tiktok_video_id` (UNIQUE globale), INSERT batch 200 `upsert(onConflict:tiktok_video_id, ignoreDuplicates)`, UPDATE chunk 50 Promise.allSettled, ritorna i counts → `finalizeLog(status='success')`
5. Su `FAILED/ABORTED/TIMED-OUT` o timeout → `finalizeLog(status='error')` con messaggio
6. Window close: se `window_expires_at <= now()` e `!window_closed`, set `window_closed=true` e `views_final=min(views,cap)`
7. Frontend: `useScrapingStatus` polla `scraping_logs` (3s se running, 30s altrimenti), `ScrapingStatusBanner` mostra `progress_note` live e fa toast + invalidate query su success/error
8. Path secondari: `handleManualImport` ("Importa dataset" con `datasetId`), `handleWebhookFallback` (idempotente: se esiste già un log `running`/`success` per quel `run_id` non riprocessa)

API token Apify: prima cerca `settings.value` con `key='apify_api_key'`, poi fallback su env `APIFY_API_KEY`.

**Anti-pattern storico risolto (15 giu 2026)**: dopo il merge di SP#5 in `main`, il codice nuovo (`handleStartWithPolling`, `pollAndProcess`, `createLog`, `finalizeLog`) NON è stato deployato automaticamente. La function attiva su Supabase era ancora la vecchia versione webhook-based, che leggeva `{{resource.defaultDatasetId}}` come placeholder Apify mai sostituito → 404 "Dataset was not found" a ogni run. Diagnosi: i log dell'edge function deployata mostravano "Step 5: Uso dataset esistente: {{resource.defaultDatasetId}}", stringhe inesistenti nel sorgente locale. Fix: deploy esplicito tramite Supabase MCP `deploy_edge_functions`. **Regola**: dopo ogni modifica a `supabase/functions/*` verificare il deploy ispezionando una stringa unica dei log della function in produzione, non fidarsi del fatto che il merge in `main` abbia propagato il deploy.

**Campi estratti da Apify item**:
- `id` → `tiktok_video_id`
- `playCount` → `views`
- `diggCount` → `likes`
- `commentCount` → `comments`
- `createTime` (unix) → `published_at`
- `musicMeta.musicId` → `audio_id`
- `musicMeta.musicName` → `audio_name`
- `text` → `caption`
- `hashtags[].name` → `hashtags[]`

**Matching video → brief** (post SP#4):
1. `videos.audio_id = video_briefs.audio_id` (entrambi not null) → confidence 1.0
2. Fuzzy similarity (pg_trgm) `videos.caption` contiene `video_briefs.expected_caption_keywords[]` → confidence 0.7
3. Manuale via UI admin → confidence 1.0

Format propagation: se brief ha `format_id` e match riuscito e `videos.content_tag IS NULL` → autoset `content_tag = video_formats.name`. Mai sovrascrive un tag manuale.

---

## 7. Convenzioni di stile (NON negoziabili)

### Copy/UX
- **Niente em dash** (—). Sostituire con virgola, due punti, o frase staccata.
- **Niente AI-pattern** stile "Not 5. Not 20. Hundreds." Mantenere copy professionale e diretto.
- **Toast e UI in italiano**. Codice/commenti possono restare in inglese.
- **Numeri**: sempre `formatViews` o `formatCurrency` da `@/lib/format`. Mai `.toString()` raw.
- **Date**: sempre `it-IT` locale (`new Date(x).toLocaleDateString("it-IT")`).
- **Link esterni TikTok**: sempre `target="_blank" rel="noopener"`. MAI `nofollow`/`ugc`/`sponsored`.

### Codice
- TypeScript strict. Niente `any` se non strettamente necessario (cast `as any` accettabile su `supabase.rpc as any` perché i tipi RPC sono incompleti).
- Hook pattern: `useXxx` con react-query. Sempre `queryKey` strutturato (array).
- Component splitting: NO file > 500 righe. Se cresce, splitta in sub-componenti.
- Path import: usare alias `@/` invece di relative path.

### Database
- Tutte le RPC sono `SECURITY DEFINER` con `SET search_path = public`.
- Tutte iniziano con check `IF NOT public.has_role(auth.uid(),'X') THEN RAISE EXCEPTION 'Forbidden'; END IF;` per ruoli ristretti.
- RLS abilitata su TUTTE le tabelle pubbliche. Mai disabilitarla.
- Migrations idempotenti dove possibile (`CREATE TABLE IF NOT EXISTS`, `ALTER TABLE ... ADD COLUMN IF NOT EXISTS`).
- Trigger `set_updated_at()` su tutte le tabelle con `updated_at`.

---

## 8. Pattern architetturali

### Lettura: view + RPC sopra
Per dashboard complessi, pattern: tabelle raw → VIEW unificata che enrichisce/COALESCE → RPC che aggrega in jsonb → hook `useQuery`. Esempi:
- `v_financial_movements` + `get_finance_dashboard()` (Finance)
- `v_video_performance` + `get_video_analytics()` + `get_top_videos()` (Video Analytics)
- (in arrivo SP#4) `get_content_calendar()` + `get_content_analytics()` + `get_content_insights()`

Vantaggi: una sola query JSON → un solo round trip → react-query cache pulita.

### Scrittura: insert su tabella + trigger
Pattern: il client fa `supabase.from(X).insert(...)`. I trigger DB fanno il side-effect (matching, aggregazioni, propagation). Mai logica di business complessa nel frontend.

### AI-powered import (SP#5)
Pattern per import bulk da testo non strutturato: textarea (paste) → edge function → LLM con structured output (system prompt + schema JSON rigoroso, `JSON.parse` con try/catch e strip dei code fence) → preview editabile in UI → bulk insert. Esempio: `parse-briefs-from-text` (Claude Haiku) → `BriefImportDialog` → `useBulkCreateBriefs`. L'utente rivede sempre prima di scrivere su DB.

### Integrazioni esterne: polling, non webhook (SP#5)
Per integrazioni esterne (Apify, ecc.) non fidarsi della consegna di un webhook: usa polling background (`EdgeRuntime.waitUntil`) con status sync su una tabella di log (`scraping_logs`) che il frontend polla. Webhook eventuale resta solo come fallback idempotente.

### Multi-portale + RLS
Stessa tabella, RLS gating per ruolo. Esempi:
- `video_briefs`: staff vede tutto, client vede solo `in_review/approved/archived` della sua campagna, creator vede solo `in_review/approved` delle campagne assegnate.
- `client_payments`: admin/team modificano, client vede solo i suoi.

I tre portali sono:
- `/dashboard/*` (hub interno)
- `/client` (`src/pages/ClientArea.tsx`)
- `/creator` (`src/pages/CreatorArea.tsx`)

### Backward compatibility delle route
Quando rinomini una pagina, lascia la vecchia URL come `<Navigate to=... replace />`. Notifiche e bookmark esistenti continuano a funzionare. Esempi:
- `/dashboard/payments-receivable` → redirect a `/dashboard/finance?tab=receivable`
- `/dashboard/payments-payable` → redirect a `/dashboard/finance?tab=payable`
- `/dashboard/trend-tiktok` → redirect a `/dashboard/content-calendar` (post SP#4)
- `/dashboard/campaign-manager` → redirect a `/dashboard/content-calendar?tab=analytics` (post SP#4)

### Tab attivo via query param
Le pagine multi-tab sincronizzano lo stato col query param `?tab=X`. Refresh e link condivisi mantengono lo stato. Pattern in `FinancePage.tsx`.

---

## 9. Comandi utili

```bash
# Dev
npm run dev
# Build
npm run build
# Type check
npm run typecheck   # se presente, altrimenti tsc --noEmit
# Lint
npm run lint

# Supabase locale (se serve)
supabase start
supabase db reset   # ricostruisce DB locale da migrations + seed
supabase migration new <nome>
```

Per applicare una nuova migration in produzione: `supabase db push` (chiede conferma per ogni statement nuovo).

---

## 10. Stato del rebuild (cronologia super prompt)

Ogni Super Prompt = una PR grossa che applica un modulo completo. File spec in `Kannon Hub Rebuild/` (cartella Cowork, non in repo).

- **SP #1 · Foundation Pass** ✅ applicato
  - `src/lib/roles.ts` (constants RBAC)
  - Sidebar rifatta (COMMAND CENTER/OPERATIONS/CREATOR/SALES/FINANCE/ANALYTICS)
  - 5 page placeholder per moduli NEW
  - Routing aggiornato

- **SP #2 · Finance Unificata** ✅ applicato (+ Fix 2.1)
  - `recurring_expenses` + funzione generate + cron
  - Override `amount_override` su client/creator payments
  - View `v_financial_movements`
  - RPC `get_finance_dashboard` riscritta sopra la view
  - FinancePage con 8 tab (Cash, Da Ricevere, Da Pagare, Movimenti, Ricavi, Costi, Margini, Forecast)
  - Pagine Pagamenti accorpate come tab via redirect

- **SP #3 · Video Analytics** ✅ applicato (+ Fix 3.1)
  - View `v_video_performance` + RPC `get_video_analytics` + `get_top_videos`
  - Pagina con KPI, time series, top campagne/creator, tabella video paginata
  - AlertDialog conferma su Refresh dati TikTok
  - Sync App.tsx route + AppSidebar (rimuovere badge NEW)

- **SP #4 · Calendario Contenuti** ✅ applicato + migration deployed on DB il 15 giugno 2026
  - `video_briefs` come entità centrale (sostituisce concetto "trend")
  - Format come FK 1:1 (riusa `video_formats`, + colonna `is_active`), Topic N:N (`content_topics` + `brief_topics`)
  - Pagina `/dashboard/content-calendar` con 4 tab (Calendario, Analytics, Insights, Catalog), tab via `?tab=`, selettore campagna persistito in localStorage
  - Assorbe `CampaignManagerPage` come tab Analytics, splittata in componenti modulari in `src/components/content-calendar/analytics/`. La vecchia pagina monolitica resta nel repo ma non è più rotteata (la route `campaign-manager` ora è un redirect)
  - Multi-portale: hub + ClientArea (approva/commenta/CR via top-level Tabs) + CreatorArea (sezioni Contenuti e Calendario con copy-to-clipboard)
  - Workflow: draft → in_review → approved → archived, bulk load settimanale (`useBulkLoadWeek`)
  - Matching video↔brief: `match_video_to_briefs(uuid)` + trigger AFTER INSERT (sempre) e AFTER UPDATE OF audio_id/caption/hashtags (con WHEN su IS DISTINCT) + `rematch_all_unmatched_videos(int)` per backfill. Window ±14 giorni. audio_id confidence 1.0, caption keywords (pg_trgm word_similarity > 0.6 OR ILIKE, tutte le keyword) 0.7, manuale 1.0
  - Format propagation auto: su match, se `videos.content_tag IS NULL` e brief ha format → set content_tag = nome format (manuale ha sempre precedenza)
  - `scrape-tiktok` popola audio_id/audio_name/caption/hashtags su INSERT e UPDATE (UPDATE solo se valore presente)
  - Decisioni effettive: FK client→campagna è `campaigns.client_profile_id`; FK creator→user è `creators.profile_id`; i ruoli sono in `user_roles` (NON `profiles.role`); tutte le policy usano `has_role(auth.uid(),'x'::app_role)`; le RPC sono `v_base || jsonb_build_object(...)` per estendere senza rompere il payload esistente

- **SP #5 · Scraping resiliente + AI-paste import** ✅ applicato
  - Parte A: `scrape-tiktok` refactor da webhook fragile a polling background (`EdgeRuntime.waitUntil` + status sync su `scraping_logs`). Nuove colonne `run_id, dataset_id, started_at, completed_at, progress_note, triggered_by` + CHECK status `running/success/error`. Hook `useScrapingStatus`/`useStartScraping`/`useImportDataset`, componente `ScrapingStatusBanner` integrato in AccountPage, VideoAnalyticsPage, ContentCalendarPage. Webhook resta come fallback idempotente.
  - Parte B: edge function `parse-briefs-from-text` (Claude Haiku) per import bulk da paste Google Doc. Hook `useParseBriefsFromText`/`useBulkCreateBriefs`, `BriefImportDialog` (paste + preview editabile) nel tab Calendario. Richiede secret `ANTHROPIC_API_KEY` (vedi `docs/edge-functions-secrets.md`).

- **SP #6+** (futuri, ordine non fissato):
  - Creator Pipeline (Closer+Onboarding unificati come kanban)
  - Hiring Creator (outreach recruiting)
  - Pipeline B2B (CRM brand prospect)
  - Report dashboard cross-modulo

---

## 11. Cose da NON fare (anti-pattern Kannon)

1. **NON usare la parola "trend"** in DB/codice/UI. L'entità si chiama `video_brief` o "brief", la sezione è "Calendario Contenuti". Decisione presa nel SP#4.
2. **NON duplicare la logica business in TS**. Se calcolare `effective_views` o `is_winner` in JS richiede 20 righe, fallo in SQL via view.
3. **NON modificare `client_payments.total_amount` o `creator_payments.total_amount` per "correggere" un pagamento**. Usa `amount_override` invece. L'originale è source-of-truth contabile.
4. **NON disabilitare RLS** "temporaneamente per debug". Usa service_role da edge function se devi bypassare.
5. **NON inserire em dash, AI-pattern, o `nofollow`** nei testi UI.
6. **NON cambiare lo schema di una RPC esistente** senza aggiornare gli hook che la consumano. Le RPC sono API contracts.
7. **NON usare `useFinanceData` per leggere dati operativi pagamenti**. Usa `usePaymentsData`. Finance è dashboard di lettura, Payments è pannello operativo (mark paid/received).
8. **NON aggiungere route senza aggiornare ProtectedRoute + AppSidebar**. Triplo allineamento obbligatorio: `App.tsx` + `AppSidebar.tsx` + `roles.ts` (se cambia accesso).
9. **NON committare con file generati o secrets**. Verifica `.gitignore`.
10. **I ruoli NON sono su `profiles`**. `profiles` ha solo id/full_name/email/avatar_url. Il ruolo sta in `user_roles(user_id, role app_role)` e si legge via `has_role(auth.uid(),'x'::app_role)`. Per risolvere lo staff in SQL, query su `user_roles`, non su `profiles.role` (che non esiste).
11. **`has_role` richiede il cast `::app_role`**. Sempre `has_role(auth.uid(),'admin'::app_role)`, mai stringa nuda.
12. **FK portali**: client→campagna è `campaigns.client_profile_id`; creator→user è `creators.profile_id`. Entrambe puntano a `profiles.id` (= `auth.uid()`). Non esistono `client_user_id` né `creators.user_id`.
13. **NON usare `WITH CHECK` column-level su UPDATE**. Postgres non supporta UPDATE policies a livello di colonna: una `WITH CHECK` valida solo lo stato finale della riga, non quali colonne sono cambiate. Per restringere quali colonne un ruolo può modificare, usare un trigger `BEFORE UPDATE` come `guard_client_brief_updates` su `video_briefs`.
14. **NON dipendere da webhook esterni senza fallback**. Per integrazioni esterne (Apify, Stripe, ecc.) il webhook può non arrivare (delay, drop silenzioso, IP block). Usa polling background o un reconciliation job come meccanismo primario; il webhook resta solo fallback idempotente.
15. **NON dare per scontato che il merge in `main` deployi le edge functions**. Il deploy delle edge functions Supabase è separato dal deploy frontend. Dopo ogni modifica a `supabase/functions/*`, verifica il deploy: cerca una stringa unica del nuovo codice nei log della function in produzione, oppure forza il redeploy via Supabase MCP `deploy_edge_functions`. Il caso reale: SP#5 mergiato il 15 giu 2026, ma la function deployata era ancora la versione webhook-based pre-SP#5 → ogni "Scrapa ora" falliva con 404 su `{{resource.defaultDatasetId}}` mai sostituito (~€10/run bruciati su Apify).

---

## 12. Come aggiornare questo file

Aggiorna **CLAUDE.md** quando:
- Si aggiunge una tabella DB (sezione 4)
- Si aggiunge una funzione SQL o RPC chiave (sezioni 4 e 8)
- Si aggiunge un ruolo o si cambia accesso (sezione 3)
- Si applica un Super Prompt (sezione 10)
- Si cambia una convenzione (sezioni 7 e 11)

Pattern di update:
1. Modifica le sezioni rilevanti
2. Aggiorna sezione 10 (cronologia)
3. Commit con messaggio `docs: update CLAUDE.md after SP#N`

Mantienilo conciso (target < 600 righe). Se una sezione cresce troppo, splitta in `docs/ARCHITECTURE.md` o `docs/FINANCE.md` e linkali da qui.

---

## 13. Reference rapida file/cartelle chiave

```
src/
  pages/
    Login.tsx
    CreatorArea.tsx           ← portale creator
    ClientArea.tsx            ← portale client
    dashboard/
      GeneralePage.tsx        ← /dashboard home
      CampagnePage.tsx
      CampaignDetailPage.tsx  ⚠ monolitica 1274 righe, future refactor
      FinancePage.tsx
      ContentCalendarPage.tsx ← SP#4 (4 tab: Calendario/Analytics/Insights/Catalog). Route /dashboard/content-calendar
      CampaignManagerPage.tsx ⚠ legacy, non più rotteata (assorbita come tab Analytics). /dashboard/campaign-manager → redirect
      TrendTiktokPage.tsx     ⚠ legacy placeholder, route /dashboard/trends e /dashboard/trend-tiktok → redirect a content-calendar
      VideoAnalyticsPage.tsx  ← route /dashboard/videos
      ...
  components/
    AppSidebar.tsx            ← navigazione, gating ruoli
    ProtectedRoute.tsx        ← route guard
    finance/                  ← FinancePage components
    video-analytics/          ← VideoAnalyticsPage components
    content-calendar/         ← SP#4 (CalendarTab, ContentCalendarGrid, BriefCard, BriefDetailDrawer, BriefFormDialog, BriefCommentsThread, BriefChangeRequestsList, BriefVideoMatchesPanel, InsightsTab, CatalogTab, _helpers; analytics/ sub-cartella per il tab Analytics)
    creator/                  ← CreatorArea components (+ SP#4: CreatorBriefsList, CreatorBriefCard, CreatorBriefCalendar, CopyableField)
    client/                   ← ClientArea components (SP#4: ClientBriefsList, ClientBriefCard, ClientCommentsDrawer, ClientChangeRequestDialog)
  hooks/
    useFinanceData.ts         ← Finance dashboard + recurring + overrides
    useVideoAnalytics.ts      ← Video Analytics dashboard + top videos + refresh scrape
    usePaymentsData.ts        ← Pagamenti operativi (mark paid/received)
    useCampaignManagerData.ts ← Analytics CM legacy (ancora usata da CampaignManagerPage.tsx)
    useContentCalendar.ts     ← SP#4: calendar/analytics/insights + tutte le mutation brief + match + campaign options
    useContentCatalog.ts      ← SP#4: CRUD format e topic
    useClientBriefs.ts        ← SP#4: brief del client (RLS)
    useCreatorBriefs.ts       ← SP#4: brief del creator (RLS)
    ...
  lib/
    roles.ts                  ← RBAC constants + helpers
    format.ts                 ← formatViews / formatCurrency
    videoWindow.ts            ← Window 30gg logic
    creatorPayable.ts         ← Contract-centric SOT compenso creator
    contractPeriods.ts        ← Period helpers
  integrations/
    supabase/client.ts        ← Supabase client istanziato

supabase/
  migrations/                 ← ordinate per timestamp YYYYMMDDHHMMSS_*.sql
  functions/
    scrape-tiktok/            ← Apify scraper orchestration
    regenerate-client-payments/  ← generate ciclo pagamenti
    run-finance-jobs/         ← cron daily finance maintenance
```

---

Ultimo aggiornamento: 15 giugno 2026 (post SP#4).

---
> Source: [infoviralengine-bit/kannon](https://github.com/infoviralengine-bit/kannon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
