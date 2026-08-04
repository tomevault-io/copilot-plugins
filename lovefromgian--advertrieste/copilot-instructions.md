## advertrieste

> Piattaforma WordPress: directory su mappa di attività e luoghi del territorio di Trieste, con schede clienti, statistiche, offerte/coupon, eventi e area riservata. Plugin custom che incapsula tutta la logica di progetto.

# CLAUDE.md — AdverTrieste

Piattaforma WordPress: directory su mappa di attività e luoghi del territorio di Trieste, con schede clienti, statistiche, offerte/coupon, eventi e area riservata. Plugin custom che incapsula tutta la logica di progetto.

> Prima di implementare una feature, leggi la sua scheda in `@docs/specifiche-funzionali.md`. Non dare per scontato lo stato: alcune funzioni sono CONFERMATE, altre CONDIZIONATE o IN SOSPESO (vedi sotto).

## Stato del progetto
- Fase: **[DA DEFINIRE: greenfield / in corso]**
- WordPress **[versione]**, PHP **[8.x DA CONFERMARE]**, tema **[nome tema / "custom"]**
- Tutto il codice di progetto vive in un plugin custom: `wp-content/plugins/advertrieste/` **[confermare nome slug]**

## Comandi
> YOU MUST usare i comandi reali del progetto, non assumerli. Completa questa sezione col setup effettivo.
- Ambiente locale: **[DA DEFINIRE: wp-env / Local / Docker / DDEV]**
- Avvio: `[es. wp-env start]`
- Lint PHP: `[es. composer lint / phpcs --standard=WordPress]`
- Lint JS/CSS: `[es. npm run lint]`
- Build asset: `[es. npm run build]` — sorgenti in `assets/src/`, output in `assets/dist/`
- Test: `[es. composer test / vendor/bin/phpunit]`
- **IMPORTANT**: dopo ogni modifica a PHP esegui il lint PHPCS (standard WordPress) prima di considerare il task concluso.

## Stack & vincoli tecnici
- **CMS**: WordPress + plugin custom. NON creare temi/pagine hardcoded fuori dalle API di WP.
- **Mappa**: Leaflet + OpenStreetMap (no Google Maps per la mappa base, per motivi di costo).
- **Recensioni**: Google Places API (SKU "Pro" per le review). YOU MUST usare cache lato server (transient) di pochi giorni e NON persistere le review oltre i limiti consentiti da Google. Prevedi un interruttore per disattivare la feature se si supera la soglia.
- **Pagamenti**: WooCommerce + Subscriptions per abbonamenti e rinnovi automatici. Attivo al lancio.
- **Automazioni**: WP-Cron per scadenze/email. Non usare cron di sistema senza indicarlo.
- **Routing percorsi**: preferire link all'app mappe nativa; il tracciato in-mappa (OSRM/Leaflet Routing) è opzionale.

## Struttura del plugin (convenzione)
```
advertrieste/
├── advertrieste.php        # bootstrap, header plugin, costanti
├── includes/               # logica PHP (una classe per file)
│   ├── cpt/                # registrazione Custom Post Type & tassonomie
│   ├── meta/               # meta box / campi (scheda, evento, punto-qr…)
│   ├── rest/               # endpoint REST custom (namespace advertrieste/v1)
│   ├── cron/               # job scadenze, email, refresh recensioni
│   ├── stats/              # tracking visite/click/coupon + query dashboard
│   └── access/             # controlli di visibilità e capability
├── assets/src/             # JS/CSS sorgente (mappa, dashboard, onboarding)
├── templates/              # template front-end del plugin
└── docs/ -> vedi @docs/
```

## Modello dati (Custom Post Type)
- `locale` — attività commerciale (logo, descrizione, servizi, galleria, coordinate, data_inizio, data_fine, in_evidenza + date evidenza).
- `poi` — punto d'interesse non commerciale (museo, castello). Distinto perché compare a zoom diversi.
- `evento` — evento (grande evento curato **o** evento di organizzatore terzo). Campo `tipo_evento` e workflow di revisione (vedi sotto).
- `punto_qr` — posizione fisica espositore/QR. **Dati riservati** (vedi Sicurezza).
- Categorie: tassonomia condivisa `categoria` con termini per intenzione (mangiare, bere, visitare, shopping, servizi).
- **Zoom a due livelli**: ogni marker ha una soglia di zoom minima; `poi` visibili a zoom basso, `locale` a zoom alto. Filtro lato REST + gestione client Leaflet.

## Regole di sicurezza e accesso — CRITICHE
- **IMPORTANT — Mappa Punti QR**: i dati di `punto_qr` (coordinate) sono visibili SOLO a utenti autenticati con ruolo cliente/admin. YOU MUST filtrare lato server: l'endpoint REST deve rifiutare le richieste non autenticate e non deve mai serializzare coordinate verso il front-end pubblico. Non affidarti a "nascondere il link" o a CSS.
- Ogni endpoint REST che espone dati clienti richiede `permission_callback` reale (mai `__return_true` su dati privati).
- Sanitizza ogni input (`sanitize_text_field`, `wp_kses_post`, ecc.) e usa `$wpdb->prepare` per ogni query custom. Escapa ogni output (`esc_html`, `esc_url`, `esc_attr`).
- Nonce su tutte le azioni di scrittura (form area clienti, AJAX).
- Geolocalizzazione utente e review: solo con consenso; conforme GDPR.

## Workflow eventi (organizzatori terzi) — regola di dominio
- YOU MUST NOT pubblicare un `evento` di organizzatore direttamente. Stati: `bozza → in_revisione → pubblicato`.
- Alla pressione di "Invia" da parte dell'organizzatore lo stato diventa `in_revisione`; la pubblicazione avviene solo dopo approvazione admin.
- **Anche le modifiche a un evento già pubblicato** entrano in revisione: mantieni una versione "pubblica" (visibile) e una "in lavorazione" (in revisione). Il pubblico vede sempre l'ultima versione APPROVATA finché la modifica non è validata.

## Convenzioni di codice
- PHP secondo **WordPress Coding Standards** (PHPCS). Prefisso globale `advtr_` per funzioni/hook/option; namespace `AdverTrieste\` per le classi.
- File: un componente per file. Classi PascalCase, funzioni/hook `advtr_snake_case`, costanti `ADVTR_UPPER`.
- Testo utente sempre traducibile: `__()/_e()` con text domain `advertrieste`.
- REST namespace: `advertrieste/v1`.
- Nessuna dipendenza JS pesante non necessaria; niente framework SPA per il front-end pubblico.

## Cosa NON fare
- NON modificare core di WordPress né file del tema per logica che appartiene al plugin.
- NON committare chiavi/API key: usare `wp-config.php` o variabili d'ambiente. La chiave Places API è segreta.
- NON introdurre scraping di siti terzi per gli eventi. L'import automatico da fonti esterne è **IN SOSPESO** e subordinato a permesso (vedi specifiche). Finché non confermato, la sezione eventi si popola a mano / con inserimento assistito.
- NON mostrare contatori visite "gonfiati": sotto la soglia usa il badge "Novità" (vedi specifiche).
- NON dare per scontati i comandi: se non sono in questo file, chiedi o verifica.

## Workflow di lavoro atteso
- Per feature non banali: prima **pianifica** (ricerca + piano), poi implementa. Puoi usare "ultrathink" per problemi architetturali.
- Crea un branch git per ogni feature/fix: `feature/descrizione` o `fix/descrizione`.
- Commit con messaggi in conventional commits.
- Aggiorna/aggiungi test quando tocchi logica (scadenze, coupon, access control, workflow eventi sono i punti critici da coprire).

## Documenti di riferimento (carica quando serve)
- @docs/specifiche-funzionali.md — cosa fa ogni modulo e il suo stato (fonte di verità sui requisiti)
- @docs/architettura.md — [DA CREARE: schema dati dettagliato, endpoint REST, flussi]
- @docs/deploy.md — [DA CREARE: hosting, ambienti, procedura di rilascio]

---
> Source: [lovefromgian/advertrieste](https://github.com/lovefromgian/advertrieste) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
