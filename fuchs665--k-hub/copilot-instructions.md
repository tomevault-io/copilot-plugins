## k-hub

> K-Hub è una piattaforma web per aggregare eventi di rental karting in Italia e (in roadmap) tracciare le statistiche dei piloti a livello nazionale. Target: piloti rental, in particolare neofiti.

# K-Hub — Contesto Progetto

## Cos'è
K-Hub è una piattaforma web per aggregare eventi di rental karting in Italia e (in roadmap) tracciare le statistiche dei piloti a livello nazionale. Target: piloti rental, in particolare neofiti.

## Stack reale (NON Flutter — attenzione, vecchi prompt lo presumevano)
- **Frontend**: React 19 + Vite + Tailwind CSS v4, react-router-dom v7, lucide-react per le icone. Stile "brutalist/sportivo" con CSS custom in `frontend/src/index.css` (variabili tipo `--castrol-red`, classi `card-snappy`, `btn-snappy`).
- **Backend**: Supabase (PostgreSQL + Auth), RLS attiva. Schema base in `database_schema.sql`, esteso dalle migration in `migrations/` (001→004, applicate al DB live a mano via SQL editor: il CLI Supabase non ha access token su questa macchina — mai tentare scritture di test sul DB di produzione).
- **Scraper**: Python, 5 fonti orchestrate da `run_all.py`: SWS (Playwright + playwright-stealth + BeautifulSoup, Chrome reale non-headless, `scraper_base.py`), WeRace/XRace/KRM (HTTP/JSON o BeautifulSoup), RKC ASI (`rkc_asi_scraper.py`, API JSON pubblica del sito — plugin "The Events Calendar", no Playwright necessario). Scrive nella tabella `events` con upsert dedup su `(source_url, event_date)` e arricchisce region/format a scrape-time. Usa la libreria interna `it-scraper-toolkit` (repo locale `C:\Users\FCD\Documents\it-scraper-toolkit`, GitHub privato `Fuchs665/it-scraper-toolkit`, installata editable via requirements.txt): `configure_stdio()` in `scraper_base` rende le print encoding-safe su Windows (niente più crash da emoji/accenti), `HttpClient` con retry/backoff + rate limit 1 req/s + user-agent onesto "K-Hub-Scraper/0.1" per le 4 fonti HTTP (verificato: nessun sito lo blocca), `parse_italian_date` per le date SWS/KRM. `run_all.py --dry-run` esegue tutto senza scrivere sul DB e segnala i duplicati cross-fonte (stessa data+pista da fonti diverse, report-only).
- **Deploy/vincoli**: solo free tier Supabase e Vercel. Nessuna spesa.

## Stato attuale (luglio 2026 — Step 2, 3, 4, 5, 6 e 7 completati)
- DB live (ref `yiysqhbtmjdpooznsgbg`) allineato a `migrations/001→006`: `tracks` (+ seed 14 piste/6 regioni), `events` (+ region, format, created_by, series), `profiles`, `race_results`, `lap_times`, `regulations`; vista `v_pilot_leaderboard` (non più usata in UI, vedi Step 4); RPC `get_pilot_stats(p_pilot_id)` e `get_event_standings(p_event_id)`; indice UNIQUE completo su `events(source_url, event_date)`.
- Frontend: TUTTE le query dati passano dal layer repository in `frontend/src/lib/` (`eventsRepository`, `pilotsRepository`, `tracksRepository`, `resultsRepository`, cache TTL in `cache.js`); il client Supabase esiste solo in `lib/supabase.js` e le pagine lo usano direttamente solo per l'auth.
- Pagine React: Home, Calendar (chip Tipologia Gara + Filtri Avanzati collassabile, chip filtri attivi rimovibili, vista Lista raggruppata per bucket data e vista Mese con griglia cliccabile), TracksDirectory (mappa SVG Italia cliccabile), RkcAsi (tab per regione con dati reali `events.series='rkc_asi'`, vedi Step 7), GuidaRental (guida neofiti, sostituisce la vecchia Leaderboard nazionale), Dashboard pilota (KPI + trend punti + storico), EventDetails (classifica + accordion tempi), OrganizerDashboard (inserimento eventi + risultati/tempi con import bulk), Auth.

## Problemi noti / debito tecnico
1. **RLS lasca su risultati**: le policy su `race_results`/`lap_times` sono `FOR ALL TO authenticated WITH CHECK (true)` — qualsiasi utente loggato può inserire/modificare risultati altrui. Accettato per l'MVP; in futuro serve un ruolo organizer.
2. **Scraper non testato post-refactor**: l'upsert dedup e l'arricchimento region/format non hanno ancora avuto un run reale contro il DB live. Nota 2026-07-20: la pipeline di estrazione è stata verificata con `run_all.py --dry-run` (37 eventi da 4 fonti, zero crash) dopo l'integrazione di it-scraper-toolkit, ma il ramo di INSERT resta non testato; inoltre i file `.env.local` (scraper e frontend) non sono presenti su disco, quindi un run reale richiede prima di ripristinarli.
3. **Nessuna UI di modifica/cancellazione risultati**: OrganizerDashboard permette solo l'inserimento; errori di battitura si correggono solo dal DB.
4. **Artefatti di debug tracciati in git**: `scraper/sws_races.html` + `sws_races_files/`, `debug_screenshot.png` — da pulire dopo aver verificato che l'HTML non serva come fixture.
5. **Possibile duplicazione cross-fonte per le tappe RKC ASI**: alcune tappe condividono la pista con eventi già scrapeati da altre fonti (es. Misanino KCE, già noto a KRM) ma con `source_url` diverso — la dedup upsert è su `(source_url, event_date)`, quindi non rileva duplicati tra fonti diverse per lo stesso giorno/pista. `run_all.py` ora li segnala automaticamente a ogni run (report-only via `toolkit.dedupe.find_duplicates` su data+pista, filtrato su domini sorgente diversi); al run del 2026-07-20 nessun duplicato cross-fonte in pratica. Se in futuro il report ne trova, decidere quale fonte vince prima di automatizzare la rimozione.

## Convenzioni
- Lavorare in iterazioni: proporre → conferma → implementare.
- Query sui tempi/classifiche ottimizzate lato DB (viste o RPC Postgres), non aggregazioni in JS sul client.
- Gestire sempre dati incompleti (circuiti che non forniscono tutti i tempi).
- UI: look sportivo/tecnico, numeri leggibili, tabelle responsive anche su schermi stretti.
- Commit piccoli e descrittivi.

## Roadmap
- **Step 2 — Backend e logiche statistiche**: ✅ completato (luglio 2026) — schema esteso via migration 001→004, viste/RPC, layer repository con caching, deduplica scraper, fix RLS.
- **Step 3 — Frontend e data visualization**: ✅ completato (luglio 2026) — Dashboard Pilota, Leaderboard nazionale, EventDetails con classifiche/tempi, filtri veri da DB, inserimento risultati, fix region/format scraper. Grafici con CSS/SVG custom, nessuna lib aggiuntiva.

### Fase rifinitura UX pre-deploy (concordata luglio 2026 — una chat per Step)
Decisioni prese con l'utente il 2026-07-06. Ogni Step è pensato per essere una sessione/chat autonoma (aprire una chat nuova per ognuno, così si risparmiano token e si mantiene alta la qualità). Ordine deciso: prima le 3 rifiniture UI (quick win testabili dagli amici), RKC ASI per ultimo perché tocca schema+scraper.

- **Step 4 — Rimozione Classifica nazionale + guida neofiti**: ✅ completato (luglio 2026) — `Leaderboard.jsx`, route `/leaderboard` e voce navbar rimossi; `getLeaderboard`/`v_pilot_leaderboard` lasciati intatti nel DB (non collegati alla UI). Nuova pagina `GuidaRental.jsx` con guida "Come iniziare col rental" per neofiti.
- **Step 5 — Piste: mappa SVG Italia cliccabile**: ✅ completato (luglio 2026) — `TracksDirectory.jsx` usa `ItalyMap` (SVG custom, nessuna lib esterna): click regione filtra le piste sotto, evidenziate in rosso Castrol se hanno piste censite. `migrations/005_seed_tracks.sql` applicata sul DB live (14 piste/6 regioni).
- **Step 6 — Calendario: filtri a chip + vista dinamica**: ✅ completato (luglio 2026) — `Calendar.jsx` con chip Tipologia Gara + pannello "Filtri avanzati" collassabile (Kart/Formato/Regione) + chip filtri attivi rimovibili singolarmente. Vista Lista raggruppata in bucket relativi a oggi (Eventi passati/Oggi/Questo weekend/Prossima settimana/Questo mese/mese successivo/Più avanti). Nuova vista Mese (toggle Lista/Mese) con griglia mensile cliccabile, stessi filtri della lista, navigazione mese; su schermi stretti la griglia collassa in lista dei soli giorni con gare.
- **Step 7 — RKC ASI: sorgente dati reale**: ✅ completato (luglio 2026) — `migrations/006_rkc_asi_series.sql` applicata sul DB live, `scraper/run_all.py` eseguito con successo (42 eventi upsertati, 12 tappe RKC ASI con `series='rkc_asi'`), verificato in UI: tab regione popolati (Emilia-Romagna 4, Lazio 3, Lombardia 3). Discovery: rkcasikarting.it espone un'API JSON pubblica (plugin WordPress "The Events Calendar", `wp-json/tribe/events/v1/events`) col calendario tappe (titolo, data, location, prezzo, link) — niente Playwright necessario. I tempi/classifiche reali sono invece su Apex Timing (servizio esterno via WebSocket) — fuori scope per scelta dell'utente: solo link esterno alla pagina evento, nessuno scraping di classifiche/tempi. I "gruppi regionali" dei coordinatori RKC ASI (es. "Sicilia 1", "Emilia Romagna 1 - Marche 1") sono frammentati e non coincidono con le 20 regioni amministrative (una tappa "RKC ASI Toscana" può giocarsi fisicamente in Emilia-Romagna) — deciso con l'utente di NON modellarli e riusare la `region` fisica già esistente, risolta da pista/provincia e mai dal titolo (per non "contaminare" la region condivisa con Calendar/TracksDirectory). Implementato: `migrations/006_rkc_asi_series.sql` (`events.series`, CHECK IN ('rkc_asi')); `scraper/rkc_asi_scraper.py` (testato contro l'API live: 12/12 tappe estratte con region/event_type corretti) wired in `run_all.py`; `getRkcAsiEvents()` in `eventsRepository.js`; `RkcAsi.jsx` con tab regione dinamici (conteggio ed evidenziazione, toggle stile TracksDirectory) + `EventCard` condiviso. Prossimo passo utente: applicare la migration, poi `python scraper/rkc_asi_scraper.py` (o `run_all.py`) per popolare le tappe.
- **Step 8 — Deploy su Vercel** (dopo le rifiniture): `frontend/vercel.json` e `_redirects` già pronti. Mettere online per far testare agli amici e raccogliere feedback.

- **Possibili sviluppi futuri (non concordati)**: run periodico dello scraper, ruolo organizer con RLS più stretta, modifica/cancellazione risultati.

---
> Source: [Fuchs665/K-Hub](https://github.com/Fuchs665/K-Hub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
