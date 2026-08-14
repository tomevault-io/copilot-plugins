## jinglemachine

> Promemoria per lavorare bene su questo progetto. Leggi prima di iniziare un task.

# CLAUDE.md — istruzioni per l'assistente

Promemoria per lavorare bene su questo progetto. Leggi prima di iniziare un task.

## 🔚 Comando «chiudo» (rituale di fine sessione) — ISTRUZIONE PERMANENTE

Quando l'utente scrive **`chiudo`** (da solo o in una frase), PRIMA di rispondere:

1. **Analizza la chat corrente** ed estrai ciò che sarà utile al *me* futuro per **decidere meglio e più in fretta**:
   - decisioni prese **+ il perché**; opzioni **scartate** e il motivo (per non riproporle);
   - **preferenze e stile di lavoro** dell'utente emersi;
   - **fatti verificati** (con data, perché deperibili) e link/fonti;
   - trappole/insidie incontrate e come evitarle.
2. **Scrivi** queste indicazioni nel file giusto (senza duplicare ciò che c'è già):
   - **`MEMO.md`** → decisioni tecniche/architetturali, vincoli, fatti verificati, trucchi operativi;
   - **`ROADMAP.md`** → avanzamento (spunta gli item fatti) e prossimi passi;
   - **`CLAUDE.md`** (questo) → convenzioni di lavoro, preferenze dell'utente, come prendere le decisioni.
3. **Riassumi** in chat cosa hai scritto e dove.

> Obiettivo: ogni sessione lascia il progetto più "pronto" della precedente. Conciso e azionabile, niente muri di testo.

### Preferenze dell'utente (aggiornare qui col tempo)
- Vuole soluzioni **gratuite e senza carta di credito**; rifiuta i piani a pagamento se evitabili.
- Vuole **semplicità e leggibilità**, best practice dei framework; **niente over-engineering**.
- Si fida quando **verifico i fatti online** (specie cose che cambiano nel tempo) invece di andare a memoria.
- Fa **domande di pianificazione** prima di codare: meglio chiarire architettura/costi/rischi prima.
  Asseconda: sviscera alternative/costi/rischi **prima** di scrivere codice, senza fretta di codare.
- **Principiante sul backend** → spiegare i concetti in modo **elementare**, con analogie, e indicare il "perché" delle scelte.
- Segnalare sempre **refusi/errori** in italiano e inglese (anche fuori task) — vedi istruzioni globali.
- **"committa"** in questo progetto = **commit + push su `main`** (→ redeploy automatico su GitHub Pages). Non fermarsi al commit locale salvo richiesta esplicita.
- **"crea i pacchetti" / "pubblica gli installer"** = creare e pushare un **tag `v<versione>`** (deve corrispondere a `server/package.json`) → fa partire la CI Electron (`build-packages.yml`, matrix Win+Mac) e pubblica gli installer su **GitHub Releases**. Il solo push su `main` aggiorna **solo** Pages, NON ricostruisce l'app desktop.
- **Signals-first (preferenza esplicita 2026-07-01)**: l'utente preferisce i **signals** al posto di callback/stato imperativo. Per lo stato condiviso → **signal-store nel service** (fonte di verità unica, update ottimistici; i componenti leggono via `computed`), niente `load()` ripetuti. Timer/tempo restano necessariamente imperativi (es. scheduler), ma i dati che alimentano la UI sono signal. Esempio: `ScheduleService` (vedi `MEMO.md` §14).
- Apprezza la **verifica visiva** (screenshot headless della webapp) quando si toccano UI/stili. ⚠️ Attenzione alle trappole degli screenshot headless (vedi `MEMO.md` §7).
- **Versioning delegato a Claude**: bumpa `server/package.json` → `version` autonomamente nello stesso commit della modifica.
  - **patch** (0.1.x) → qualsiasi commit che produce una nuova build (bugfix, CI fix, refactor, script)
  - **minor** (0.x.0) → completamento di una Fase o feature rilevante
  - **major** (x.0.0) → solo su indicazione esplicita dell'utente
  - ⚠️ Bumpa **anche** `client/src/environments/environment.ts` **e** `environment.mock.ts` → campo `version` (mostrato nella UI sotto il titolo): va tenuto in sync con `server/package.json`.
  - ⚠️ Ogni volta che si triggera una nuova build (`yarn release` o tag), la versione deve essere già stata bumpata nel commit precedente.
  - Quando si crea un tag git, assicurarsi che corrisponda alla versione nel `package.json`.

## Come scrivere in MEMO/ROADMAP: separare «verificato» da «ipotesi»

Lezione pagata cara il 2026-07-29. Una voce di `MEMO.md` affermava come **acquisito**
(«`hardenedRuntime: false` è OBBLIGATORIO») quello che era solo una **congettura mai testata su un Mac**.
Era falsa, ha indirizzato male la sessione successiva e sono serviti due giri di build per smontarla.

- Marcare sempre lo stato di una conclusione: **✅ verificato** (con *come* è stato verificato e la data)
  oppure **⏳ ipotesi, non ancora confermata**. Mai scrivere una causa presunta come fosse dimostrata.
- Quando un'ipotesi viene smentita, **correggere la voce vecchia** (barrarla + rimando alla nuova), non
  solo aggiungerne una nuova: chi legge trova prima quella vecchia.
- Annotare le **ipotesi scartate e perché**: valgono quanto la soluzione, evitano di riproporle.

## Bug platform-specific (macOS) non riproducibili da Windows

Non tirare a indovinare a colpi di release: **far dimostrare il fatto alla CI**.

1. Aggiungere al workflow uno **step di ispezione** che stampa lo stato reale dell'artefatto
   (es. *"Inspect the Mac bundle"*: `codesign -dv` + `plutil -p` dell'Info.plist nel job macOS).
2. Iterare con **`gh workflow run build-packages.yml`** → costruisce e ispeziona in **~3 minuti**,
   **senza** pubblicare una Release (lo step di publish è vincolato ai tag) e senza coinvolgere il Mac.
3. **Taggare solo a esito verificato**, per non riempire le Release di tentativi.
4. Chiedere all'utente comandi diagnostici sul Mac (`codesign -dv --verbose=2`, `plutil -p`) **prima** di
   proporre un fix: costano 10 secondi e hanno smentito due mie ipotesi di fila.

⚠️ L'app desktop **non ha DevTools** (`Menu.setApplicationMenu(null)`): sul Mac non si legge nessuna
console. Gli errori vanno resi **visibili nella UI**, altrimenti si debugga alla cieca.

### Messaggi d'errore: mai collassare cause diverse (lezione 2026-07-30)
Un `catch` che risponde «Microfono non disponibile o permesso negato» a *qualunque* fallimento ha
mandato mezza giornata di indagini sui permessi macOS, mentre la causa era una libreria che non
caricava. Regole:
- **Un messaggio per causa**, distinte dal nome dell'errore (`NotAllowedError` ≠ `NotReadableError` ≠ bug).
- Per gli errori **non previsti**, portare nel messaggio anche `err.message` (troncato): in un
  `TypeError` l'informazione sta tutta lì, il nome da solo non dice niente.
- **Nessun fallimento silenzioso**: uno stato `error` senza UI è indistinguibile da "inattivo" —
  è così che un motore rotto è passato per funzionante.

### Una dipendenza che funziona in dev NON è provata
`ng serve` pre-bundla le dipendenze e **sintetizza gli export nominati** delle librerie UMD/CJS;
`ng build` no. Una libreria può quindi funzionare in dev e rompersi **solo nel pacchetto**, a runtime,
con un nome minificato (`i is not a function`) — TypeScript non se ne accorge, la build passa.
Per le dipendenze UMD/CJS verificare nel bundle costruito:
`grep -ao "export[{ ][^;]*" dist/.../chunk-<lib>.js` → se mostra solo `export default`, il
destructuring `import { x }` darà `undefined`. Caso reale: `vosk-browser`, vedi `MEMO.md` §15.

## Cos'è il progetto
Webapp Angular per creare una **libreria di jingle PER-UTENTE**: audio caricati o **estratti da YouTube**, tagliati in MP3.
Due canali: **app desktop standalone (Electron)** con tutte le funzioni, e **GitHub Pages** (stessa webapp ma **senza** YouTube). Dettagli in `MEMO.md`.

**Architettura:**
- **Firebase Auth** → login **obbligatorio** (`authGuard`); sessione **7 giorni** → ci si rilogga una volta a settimana (non a ogni refresh).
- **Firebase Firestore** → metadati jingle. Libreria **privata per utente** (filtro `uid`): due utenti vedono librerie diverse.
- **Cloudinary** (free, no carta) → file MP3 = **storage remoto**. ⚠️ NON usare Firebase Storage (richiede Blaze+carta).
- **Mixer locale** Node + yt-dlp + ffmpeg (in `server/`) → carica/taglia audio da YouTube. In **standalone è embedded in Electron**; in **dev** gira a parte su `localhost:4321`. Su GitHub Pages non c'è → il pulsante YouTube resta nascosto.
- ⚠️ **Requisito**: ottimizzare il consumo di letture/banda Cloudinary (cache HTTP + IndexedDB). Vedi `MEMO.md` §8.

## Struttura del repo
Monorepo "semplice" con due cartelle sorelle indipendenti (ognuna col suo `package.json`):
- `client/` → app Angular. I comandi usano **`npm`**, lanciati **dentro `client/`**.
- `server/` → **app desktop Electron + server locale** (estrazione YouTube → MP3). I comandi usano **`yarn`**, lanciati **dentro `server/`**.
- `firebase/` → security rules (solo Firestore). `.github/` → deploy.
- **`package.json` in radice**: SOLO orchestrazione di progetto (niente codice app, **niente `workspaces`** → client resta npm, server resta yarn). Per ora un comando: `npm run download` (scarica gli installer completi exe+dmg dalla CI in `dist/v{version}/`). Ha anche un campo `packages` che documenta i due sub-package.

## Documenti del progetto — quando consultarli

| File | Quando leggerlo |
|------|-----------------|
| **`MEMO.md`** | **Sempre, per primo.** Setup, comandi, struttura, decisioni architetturali, deploy, limiti. È la fonte di verità operativa. |
| **`ROADMAP.md`** | Per sapere a che punto siamo e cosa manca: piano a fasi con checklist. Aggiornarlo spuntando gli item completati. |
| `README.md` | Panoramica + avvio rapido (client + Mixer locale). |
| `firebase/firestore.rules` | Schema dati Firestore + sicurezza (libreria privata per utente). |
| ~~`firebase/storage.rules`~~ | **Obsoleto** (Firebase Storage rimosso → si usa Cloudinary). |
| `CLAUDE.md` (questo) | All'inizio, per orientarti e per le convenzioni. |
| **`THEMING.md`** | Quando lavori su **tema/stili**. Sistema a **token CSS** (multi-tema, un file per tema), come lo consumano Tailwind/ng-zorro/tag nativi, taxonomy dei token, ricetta per un tema nuovo, checklist di migrazione. |
| `server/README.md` | Quando lavori sul **Mixer locale** (`server/`): come avviarlo, endpoint, comandi di test. |

> Se aggiungi un nuovo `.md`, **aggiungilo a questa tabella**.

## Convenzioni di codice
- Angular **standalone** (niente NgModule), **signals** per lo stato, `inject()` per la DI.
- **Usare sempre i pattern più recenti consigliati** del framework; se in dubbio, **consultare prima la documentazione ufficiale online** (l'utente preferisce questo a soluzioni a memoria/datate).
- **Nomi variabili "puliti"**: niente caratteri come `@` `_` `$`, solo camelCase. Per gli Observable **niente suffisso `$`** → usare nomi espliciti tipo `userStream` o `userQueue` (preferenza esplicita dell'utente).
- Codice **semplice e leggibile**, commenti solo dove servono davvero (senza esagerare).
- **Nuovi componenti**: template SEMPRE in file `.html` separato (`templateUrl`), mai inline (preferenza esplicita 2026-06-07). I componenti vecchi con template inline (button, color-picker, tag-input) si uniformano se ci si lavora sopra.
- **Risposte concise** (preferenza esplicita): andare al punto, niente muri di testo.
- **LINGUA:**
  - **Codice, commenti, log, identificatori SEMPRE in inglese.**
  - **Testi UI visibili all'utente in ITALIANO** (coerenti col mockup Figma e coi colleghi destinatari). Preferenza esplicita dell'utente.
  - La documentazione (`.md`) resta in italiano.
- Service Firebase: iniettare i token `AUTH` / `FIRESTORE` da `client/src/app/core/firebase.providers.ts`
  (NON usare `@angular/fire`: incompatibile con Angular 21). NB: `STORAGE` rimosso → i file MP3 stanno su Cloudinary.
- UI con **ng-zorro-antd**: nell'HTML usare **sempre i componenti ng-zorro dove possibile** (`nz-button`, `nz-input`, ecc.). Nuove icone NgZorro → registrarle in `client/src/app/app.config.ts`.
- **Organizzazione dei componenti (preferenza esplicita dell'utente):**
  - **`client/src/app/views/`** → componenti che gestiscono un'**intera vista/pagina** (es. `login`, `library`, `stylesheet`). I sotto-componenti specifici di una vista (non riusabili altrove) restano **co-locati dentro la cartella della loro view**.
  - **`client/src/app/ui/`** → componenti **globali/riusabili** non legati a una vista (un tipo particolare di button, una card, una lista, ecc.), con selettore **`ui-{component}`** (es. `ui-button`, `ui-color-picker`). Per elementi tipo button si usa il selettore-attributo idiomatico (`button[ui-button]`) così da preservare il comportamento nativo (click, disabled, routerLink).
  - **`client/src/app/core/`** → service, guard, provider (no UI).
- **Stili (preferenza esplicita dell'utente) — vedi `THEMING.md` per il dettaglio:**
  - **Token = slot nativi di Tailwind** (`--color-*`, `--radius-*`, `--font-*`): unica fonte di verità, configurati nel blocco **`@theme static`** (tema default; `static` obbligatorio o le var usate solo da `ng-zorro.scss` vengono eliminate → sfondi trasparenti) → **si usano le utility native** (`bg-primary`, `rounded-xl`, `text-2xl`), niente scala/variabili parallele. **Un file per tema** in `client/src/styles/themes/` (default = `@theme static`; temi extra = override `:root[data-theme="x"]`). **Niente valori letterali** (`#45fff3`, `rounded-[20px]`) → utility-da-token o `var(--color-*)`.
  - **Niente classi di styling custom** (`.jm-*`, `.jw-*`): si stilizzano **tag nativi** (`input`, `button`, `textarea`), i loro **stati/attributi** (`:hover`, `:checked`, `type="password"`) e le **classi ng-zorro** (`.ant-*`), tutto via token.
  - **Cascade layer + niente `!important`**: ordine **dichiarato esplicito** in cima a `styles.css` (Tailwind v4 lo onora): **`@layer theme, base, ngzorro, components, utilities`**. La base ng-zorro è il **CSS precompilato** (`ng-zorro-antd.dark.min.css`) in `@layer ngzorro` (niente più `theme.less`/Less); deve stare **sopra `base`** o il Preflight di Tailwind ne azzera padding/margin/bordi. I nostri override `.ant-*` in **`@layer components`** → sopra `ngzorro` (vincono **senza `!important`**), sotto `utilities`.
  - **Z-index**: scala esplicita allineata a ng-zorro in token `--z-*` (modal 1000, dropdown 1050, tooltip 1070…). Mai numeri arbitrari → `z-[var(--z-modal)]`.
  - **Niente `.scss` per-componente**: layout/utility con **Tailwind inline**; **override ng-zorro** in **un unico file globale** (`ng-zorro.scss`, in `@layer components`, solo `var(--color-*)` + `color-mix()`).
  - Stili globali in **`client/src/styles/`**: `styles.css` (UNICA entry Tailwind, **deve restare `.css`**: `@import` di ng-zorro precompilato + tailwind + tokens + temi), `themes/<nome>.scss` (`@theme` default / override `:root[data-theme]`; `.scss` ok perché letto da Tailwind, ma Sass non ci gira), `tokens.css` (token strutturali: `--z-*`, `--control-height`, body, host `display:block`, focus, reduced-motion), `ng-zorro.scss` (override, entry Sass separata). Referenziati in `angular.json`.
  - **Animazioni: SOLO CSS.** Il DSL `@angular/animations` (`trigger`/`transition`/`animate`, `BrowserAnimationsModule`) è **deprecato in Angular 20+** → non usarlo per animazioni applicative; usa **transizioni/keyframe CSS**. (`provideAnimations()` resta solo perché lo richiede ng-zorro.) Movimento **leggero, Material-like** per evidenziare focus/azioni; **micro-interazioni** (`transition`, `active:scale`) + **transizioni di schermata** via `@keyframes route-enter` sugli host delle viste; rispettare sempre `prefers-reduced-motion`. Dettaglio in `THEMING.md` §13.
  - **Responsive: mobile-first** con i **breakpoint nativi Tailwind** (`sm:`/`md:`/`lg:`), niente media query custom dove evitabile. Host delle viste `display:block`. Bottoni icona-only su mobile (`hidden sm:inline` sulle label), modali `max-width: calc(100vw - 2rem)`, `--control-height` per i controlli, `nzSize` di ng-zorro (small/large) dove opportuno. Dettaglio in `THEMING.md` §14.

## Prima di consegnare una modifica
> Lancia i comandi **dentro `client/`** (`cd client`).
1. `npm run build` deve passare (è il check più affidabile: type-check + bundling).
2. Se tocchi i test: `npm test`.
3. Se cambi versioni di `@ffmpeg/*`: verifica il nome del worker e gli URL CDN in `client/src/app/core/ffmpeg.service.ts` (vedi `MEMO.md` §6).

## Punti delicati (vedi MEMO per il dettaglio)
- **YouTube**: implementato (Fase 2) via il Mixer locale (`/info` + `/extract`). Attivo solo se il Mixer risponde (standalone Electron o dev); su GitHub Pages è nascosto.
- **ffmpeg.wasm**: core single-thread + worker da CDN per compatibilità con GitHub Pages (no COOP/COEP).
- **Sicurezza / segreti**: il repo è **pubblico**. La config web Firebase/Cloudinary è pubblica per natura (sta nel client; protezione = Auth + Security Rules + preset unsigned). Le **password NON vanno mai committate**: stanno in **`CREDENZIALI.local.md`** (gitignored via `*.local.md`).
- **Test locale**: `cd client && npm run start:all` (client + Mixer) oppure `start:all:mock` (userless, senza Firebase/Cloudinary).

---
> Source: [ClemAnto/JingleMachine](https://github.com/ClemAnto/JingleMachine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
