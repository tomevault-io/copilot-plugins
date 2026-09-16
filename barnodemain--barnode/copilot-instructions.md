## barnode

> Questo file e **vincolante**. Va rispettato al **100%, in modalita enterprise, senza eccezioni**, in ogni sessione e su ogni progetto — con il **minimo consumo di crediti** (massimo rapporto qualita/prezzo). Se una richiesta contraddice queste regole, fermati e segnalalo.

# CLAUDE.md — Standard operativo enterprise (universale)

Questo file e **vincolante**. Va rispettato al **100%, in modalita enterprise, senza eccezioni**, in ogni sessione e su ogni progetto — con il **minimo consumo di crediti** (massimo rapporto qualita/prezzo). Se una richiesta contraddice queste regole, fermati e segnalalo.

## 0 · Protocollo di sessione (deterministico, anti-spreco)

Esegui questi controlli **una sola volta a inizio sessione**, in ordine, poi **fermati**. Ogni passo ha una **condizione**: se gia soddisfatta, **salta senza agire** (non sprecare crediti). NON ripetere questo protocollo a ogni messaggio: si riesegue solo a nuova sessione o dopo un **evento reale** (vedi §1bis).

1. **CLAUDE.md presente?** Se il file `CLAUDE.md` non esiste nella root, o e diverso dal prompt `CLAUDE.MD` su App Control, scaricalo (`GET /rest/v1/prompts?title=eq.CLAUDE.MD&select=full_text`) e scrivilo come `CLAUDE.md` nella root. Se gia presente e identico, **non riscaricarlo**. Da qui Claude Code lo carica da solo a ogni sessione: e la tua cache, non rileggerlo dal DB ogni volta.
2. **DNA letto?** Leggi `DNA/00` e i soli file DNA pertinenti al task corrente. Non leggere tutto il DNA "per sicurezza".
3. **`.env` allineato?** Connettiti ad App Control (§1) e rigenera `.env` SOLO se manca o se mancano variabili. Se `.env` e gia completo e coerente, **non riscriverlo**.
4. **Stato progetto:** nuovo -> struttura allo standard (§4); esistente -> allinea (vale il codice). Verifica connessioni in sola lettura, senza dichiararle ok senza prova.
5. **Output:** un riepilogo breve (stato, connessioni, prossimo passo). Poi **attendi**: non avviare sviluppo, analisi o refactor senza richiesta. Prima di ogni modifica comunica gli step (§1, Regola comunicazione).

**Condizione di stop globale:** completati i 5 punti, il protocollo e CHIUSO per la sessione. Non rilanciarlo, non ri-verificare in loop. Se un passo e gia a posto, dillo in una riga e prosegui.

## 1bis · Quando ri-sincronizzare (gate eventi, non a ogni messaggio)
Ri-esegui la **riconciliazione** (§1: variabili + due link) **solo dopo un evento concreto**, non di continuo:
- hai **creato/cambiato una variabile o un segreto** nel `.env` -> caricala in App Control;
- hai fatto un **deploy** o e cambiato un URL -> aggiorna `LINK_DEPLOY` / `LINK_DEPLOY ADMIN`;
- l'utente **chiede** esplicitamente un sync.
Fuori da questi eventi, **non leggere e non scrivere** App Control: eviti azioni a vuoto e spreco di crediti. Se non c'e nulla di nuovo, non fare nulla.

## 1 · Sincronizzazione App Control (vincolante)
App Control e la **cassaforte centrale** delle variabili di ogni progetto (suo Supabase). La connessione e **remota**, indipendente dal progetto aperto.

- **Bootstrap:** file `.agent/app-control.json` nella root (in `.gitignore`), con 4 chiavi: `projectId`, `agentKey`, `appControlSupabaseUrl`, `appControlSupabaseAnonKey`. Lo leggi a inizio sessione.
- **Accesso:** Supabase REST con header `x-app-control-project-id` + `x-app-control-agent-key` + anon key. **Lettura e scrittura**, limitate al **solo** progetto della chiave.
- **Flusso:** leggi le variabili da App Control -> **generi tu il `.env`** (l'utente non scrive mai a mano nel `.env`). **Riconciliazione (ogni sync):** confronta le chiavi del `.env` reale con quelle gia in `project_env_variables` e **carica in App Control ogni variabile/segreto nuovo o cambiato** (`SESSION_SECRET`, chiavi, URL deploy/repo, qualsiasi segreto). **Escludi** solo le 7 manuali dell'utente e le derivate `VITE_*`/`SUPABASE_DB_URL`. **Colonne valore:** ogni variabile ha `value_text` (NON sensibili) e `value_ciphertext` (sensibili, `is_sensitive=true`) — MAI `value`; in LETTURA prendi il campo giusto in base a `is_sensitive` (altrimenti le sensibili escono vuote e rompi il `.env`); in SCRITTURA includi `project_id` (UUID dalla SELECT su `projects`, non lo slug, o la RLS nega) e metti il valore in `value_ciphertext`+`is_sensitive=true` se segreto, altrimenti `value_text`+`is_sensitive=false`. Se non ci sono variabili nuove, non scrivere nulla. Le nuove appaiono in App Control sotto **"Gestite da Agent"**.
- **Chi inserisce cosa:**
  - **UTENTE** (manuale, alla creazione): `SUPABASE_URL`, `SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`, `DATABASE_URL`, `RENDER_API_KEY`.
  - **AGENT:** `GITHUB_URL` (crei tu il repo con `gh`), `GITHUB_TOKEN` e tutti i segreti generati. **Due link di deploy (dopo il deploy):** `LINK_DEPLOY` = URL pubblico/user; `LINK_DEPLOY ADMIN` = URL **reale** dell'area admin di QUESTO progetto. Il percorso/sottodominio admin **cambia da progetto a progetto**: ricavalo dal codice/config reali (route admin, deploy Render, README), **mai un suffisso fisso**. Se non esiste un'area admin separata, usa l'area di gestione reale del progetto (es. rotta protetta). Scrivi/aggiorna entrambi in `project_env_variables`; se i valori salvati non corrispondono ai link reali, correggili.
- **Nomi:** usa i nomi **canonici** (non rinominare). Le `VITE_*` non si archiviano: le generi solo nel `.env` per i frontend Vite (stesso valore).

**Regola comunicazione (fondamentale).** Prima di ogni modifica o sviluppo, **comunica SEMPRE in modo chiaro tutti gli step necessari** per sincronizzare App Control, separando **"cosa faccio io"** e **"cosa devi fare tu"** — perche l'utente potrebbe dimenticare i passaggi. Una sola azione alla volta:
`AZIONE ORA: <azione>. Poi rispondi "fatto".`

## 2 · Governance non negoziabile
Le regole sono **vincoli, non suggerimenti**: confrontale **prima** di agire; se violi, correggi **subito**.
- **Codice = unica fonte di verita.** Doc/DNA divergente -> vale il codice, riallinea la doc.
- **DB = unica fonte dei dati.** Niente dati hardcoded/mock/locali nel runtime. **Parita admin/user**: ogni entita gestita da admin e letta lato user dalla stessa fonte DB.
- **DB mai distruttivo.** Schema solo come **migrazioni versionate additive**; mai `db push`/`--force`/sync diretti su produzione. **RLS attive** su ogni tabella con dati utente.
- **Segreti:** `.env` sempre in `.gitignore`; **mai stampare** token/password/chiavi/URL con credenziali. Service role solo lato backend, mai nel frontend.
- **Git:** `git status` prima di toccare/committare; **mai committare** `.env`, backup, cache, file generati. Commit/push **solo se richiesto** e dopo che i controlli passano.
- **Push = alto rischio.** Autodeploy su un branch = **deploy in produzione**: mai pushare senza **ok esplicito**, anche se chiesto genericamente di "committare e pushare".
- **Free tier:** Supabase Free + Render (deploy unico frontend+backend). Mai saturare i limiti; segnala **prima** di avvicinarli.
- **Privacy EU:** informativa e cookie banner minimale prima del primo deploy in produzione.

## 2bis · Enforcement (come applicare le regole in tempo reale)
Le regole sopra NON sono suggerimenti: trattale come un **pre-commit hook mentale**. Confronta ogni azione con le regole **PRIMA** di eseguirla, non dopo.
- File oltre il limite di righe -> **dividilo PRIMA** di continuare, non "dopo".
- "Non duplicare logiche" -> **cerca prima** se esiste gia; non scrivere codice nuovo senza aver verificato.
- "Aggiorna la doc" -> **nello stesso intervento**, non in un commit successivo.
- Se scopri di aver violato una regola durante l'esecuzione -> **correggi subito**, non segnalarla come "da fare dopo".
Chi viola una regola produce debito tecnico che un altro operatore dovra correggere.

## 2ter · Selezione livello reasoning (triage prima di iniziare)
Si applica solo se l'ambiente ha livelli di reasoning selezionabili. Default operativo: **Medium**. Non analizzare il progetto solo per decidere il livello: usa prompt e contesto gia disponibili.

Procedi diretto con **Medium** se il task e: domanda/analisi read-only, fix puntuale/localizzato, modifica doc/config semplice, backup/verifica/commit di routine, UI/content change circoscritto senza DB/sicurezza/architettura.

Fermati prima di iniziare e scrivi esattamente `⬆️ SELEZIONA HIGH O EXTRA HIGH E RILANCIA — motivo: <una riga>` se il task richiede:
- **HIGH**: refactor multi-file o modifica architetturale non banale; debug che attraversa 2+ layer (frontend/backend/API/DB); audit ampio di performance/navigazione; bonifica con molte eliminazioni; modifiche a governance, workflow o automazioni con impatto permanente.
- **EXTRA HIGH**: schema DB/migrazioni/hardening/policy RLS; sicurezza/auth/permessi ad ampio impatto; deploy/produzione/incident recovery; eliminazioni massive difficili da annullare; decisioni architetturali permanenti; qualsiasi operazione dove un errore puo causare perdita dati, downtime o esposizione di segreti.

Accessorie: se il prompt e ambiguo ma il rischio e alto -> chiedi upgrade; se il dubbio e solo sulla dimensione del lavoro ma il rischio e basso e reversibile -> procedi. Se DURANTE l'esecuzione il task risulta piu complesso del previsto -> fermati senza lasciare lavoro a meta applicato e chiedi l'upgrade. Se l'owner scrive "PROCEDI COMUNQUE" -> esegui col livello attuale, rispettando comunque tutte le regole di sicurezza, DB, Git, privacy e non distruttivita.

## 3 · Flusso modifiche
1. Riformula in una riga **cosa fai e cosa non tocchi**.
2. Se tocca **DB, auth, deploy, architettura** o **elimina dati** -> **fermati e chiedi conferma** (piano).
3. Implementa il **minimo necessario**; riusa l'esistente; non toccare aree non dichiarate. DURANTE la scrittura verifica che ogni file rispetti i limiti di governance (dimensione, modularita, naming): se un file supera il limite, dividilo subito.
4. Verifica con gli script del progetto (typecheck/lint/build). **Non dichiarare test passati senza eseguirli.**
5. Chiudi **aggiornando doc/DNA nello stesso intervento**; registra le decisioni tecniche rilevanti nel **decision-log** (`DNA/06_DECISION_LOG.md`).

Fix piccoli (un testo, un colore): esegui diretto. Bug: **riproducilo e isola la causa radice** prima di pianificare.

## 4 · Struttura progetto (standard)
- Stack base: **React + Vite + TypeScript**, **Supabase** (dati), **Render** (deploy unico), **GitHub**.
- **File piccoli e modulari** (sotto il limite di righe, applicato in fase di scrittura); niente duplicazione di logica.
- **`DNA/`** come contesto canonico leggero: solo cio che un agent **non** ricava rapidamente dal codice; `00` = indice; numerazione per importanza. Tieni un **decision-log** in `DNA/06_DECISION_LOG.md`.
- App avviabile in locale e verificabile (porta **5001** quando previsto).

## 5 · Efficienza e crediti
- Effort **sobrio** di default; alzalo **solo** per rischio reale (DB/sicurezza/architettura/refactor/bug complesso).
- **Subagent** solo se realmente necessari.
- Comunica **una azione alla volta**, sintetico, linguaggio semplice.

## 6 · Skill on-demand (vivono in App Control, NON qui)
Non appesantire questo file: queste operazioni si invocano **solo su richiesta**. Quando l'utente le chiede, **recupera il prompt corrispondente da App Control ed eseguilo**:
Governance (crea/riallinea regole) · Pulizia e ottimizzazione · Analisi completa (sola diagnosi) · DNA (crea/riallinea) · Aggiorna DNA+Backup+Git · Ottimizzazione navigazione · Responsive mobile nativo · Qualita progetto adattiva · Keepalive Supabase · Testing visivo automatizzato · Fix complesso controllato · Crea/aggiorna `.env` · Refactoring (ripensamento progetto).

---
> Source: [barnodemain/barnode](https://github.com/barnodemain/barnode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
