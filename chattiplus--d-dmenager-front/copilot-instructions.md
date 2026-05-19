## d-dmenager-front

> Questo file descrive:

# DD MANAGER – PROJECT AGENT & TASK LIST

Questo file descrive:
- le REGOLE generali che l’agent deve seguire sul progetto DD Manager;
- le TASK backend (0–13) già eseguite o da usare come riferimento;
- le TASK frontend (FE-0…FE-5) per la web app di gestione.

Usalo come “istruzioni di base” per l’agent (system / project prompt).  
Le task già fatte servono per mantenere coerenza, non per riscrivere tutto da zero.

------------------------------------------------------------
0. LINEE GUIDA GLOBALI PER L’AGENT
------------------------------------------------------------

0.1 Tecnologie e versione
- Backend:
  - Java 17
  - Spring Boot 3.x
  - Maven
  - PostgreSQL
- Frontend:
  - Vite + Vue 3 + TypeScript
  - Vue Router
  - Pinia
  - Axios

0.2 Struttura progetto backend
- Root package: `it.univ.ddmanager`.
- Organizzazione per “feature / modulo”:
  - `user` (model, dto, repository, service, controller, exception)
  - `world` (model, dto, repository, service, controller)
  - `campaign` (model, dto, repository, service, controller)
  - `session` (model, dto, repository, service, controller)
  - `npc`
  - `location`
  - `item`
  - `sessionlog` (timeline / session events)
  - `security` (config, UserPrincipal, CurrentUserService)
  - `common` / `error` (ApiError, eccezioni applicative, RestExceptionHandler)
  - `config` (DataInitializer, ecc.)
  - `controller` generici (health)
- Per ogni modulo:
  - Entity JPA in `*.model`.
  - DTO in `*.dto`.
  - Repository in `*.repository`.
  - Service in `*.service`.
  - Controller REST in `*.controller` o `*.web`.

0.3 Dipendenze e import
- Usare Jakarta:
  - `jakarta.persistence.*` per JPA.
  - `jakarta.validation.*` per validation.
- Usare Lombok per boilerplate (`@Getter`, `@Setter`, `@Builder`, ecc.).
- Usare `org.springframework.security.*`, `org.springframework.web.*`, `org.springframework.data.jpa.*`.

0.4 Convenzioni REST
- Base path API: `/api/...`.
- Convenzioni HTTP:
  - `GET` → 200 OK, ritorna DTO.
  - `POST` → 201 Created, ritorna DTO o location.
  - `PUT` → 200 OK, ritorna DTO aggiornato.
  - `DELETE` → 204 No Content.
- Mai esporre direttamente entity JPA dai controller: usare sempre DTO di risposta.

0.5 Sicurezza e ruoli
- Ruoli: `ROLE_ADMIN`, `ROLE_GM`, `ROLE_PLAYER`, `ROLE_VIEWER`.
- Autenticazione: HTTP Basic (email + password BCryt).
- Regole generali:
  - `ADMIN` / `GM`: possono creare/modificare/eliminare domini di gioco (world, campaign, session, npc, location, item, session events).
  - `PLAYER` / `VIEWER`: solo lettura, con filtri su visibilità (`isVisibleToPlayers`).
- Endpoint pubblici:
  - `POST /api/auth/register`
  - `POST /api/auth/login`
- Tutti gli altri endpoint → autenticazione obbligatoria.
- Si possono usare:
  - `@PreAuthorize("hasAnyRole('ADMIN','GM')")` per mutate,
  - oppure regole in `SecurityFilterChain` con `.requestMatchers()`.

0.6 Gestione errori
- Struttura di errore standard: `ApiError` con almeno:
  - `timestamp`
  - `status`
  - `error`
  - `message`
  - `path`
- Mappatura eccezioni tipiche:
  - `EmailAlreadyUsedException` → 409
  - `InvalidRoleException` → 400
  - `InvalidCredentialsException` → 401
  - `ResourceNotFoundException` → 404
  - `AccessDeniedException` / ruoli non validi → 403
  - tutti gli altri errori inattesi → 500
- Usare un `@RestControllerAdvice` globale per mappare eccezioni in `ApiError`.

0.7 Testing backend
- Usare JUnit 5, Spring Boot Test, MockMvc.
- Pattern tipico:
  - `@SpringBootTest`
  - `@AutoConfigureMockMvc`
- Usare `httpBasic()` di `spring-security-test` per simulare autenticazione.
- Per ogni macro-modulo:
  - almeno 1–2 test “happy path”,
  - almeno 1–2 test di permessi (401/403),
  - almeno 1–2 test “not found” / validazioni.

0.8 Frontend
- SPA Vue 3 (Vite + TS).
- Stato globale:
  - store `auth` in Pinia con:
    - `email`, `password` (solo per dev), `nickname`, `roles`, `isAuthenticated`.
- Axios:
  - client centrale con `baseURL` da `VITE_API_BASE_URL`.
  - interceptor che aggiunge header `Authorization: Basic base64(email:password)` se l’utente è loggato.
- Routing minimo:
  - `/login`
  - `/worlds`
  - `/worlds/:id`
  - `/campaigns/:id`
  - `/sessions/:id` (timeline)

------------------------------------------------------------
1. BACKEND TASK LIST (0–13)
------------------------------------------------------------

Le Task 0–11 risultano già implementate nel progetto corrente (vedi `devlog.md`), ma restano come riferimento.  
Le Task 12–13 sono per documentazione e Postman.

---------------------------------
Task 0 – Setup iniziale (backend)
---------------------------------
- Creare progetto Maven Spring Boot 3 (Java 17) con dipendenze:
  - web, data-jpa, security, validation, Lombok, PostgreSQL driver.
- Configurare `application.yml`:
  - datasource Postgres locale (`dd_manager`),
  - `spring.jpa.hibernate.ddl-auto=update` (solo dev),
  - logging SQL formattato.
- Creare `DdManagerApplication` in `it.univ.ddmanager`.
- Aggiungere entità `Role` e `User` minime con repository:
  - `Role(name)`
  - `User(email, password, nickname, Set<Role>)`
- `DataInitializer` per creare i ruoli base:
  - `ROLE_ADMIN`, `ROLE_GM`, `ROLE_PLAYER`, `ROLE_VIEWER`.

----------------------------
Task 1 – Health check REST
----------------------------
- Aggiungere `HealthController` con:
  - `GET /api/health` → restituisce JSON con:
    - `status: "UP"`,
    - `timestamp` ISO,
    - eventuale `appVersion`.
- Endpoint pubblico o protetto (a scelta, ma nei doc va indicato).

--------------------------------------------------
Task 2 – Registrazione utente + sicurezza base
--------------------------------------------------
- DTO:
  - `RegisterRequest` (email, password, nickname, role opzionale).
  - `UserResponse` (id, email, nickname, roles).
- `UserService.register(RegisterRequest)`:
  - validazione email univoca → `EmailAlreadyUsedException` 409,
  - hashing password con BCrypt,
  - assegnazione ruolo di base (ROLE_PLAYER se role non specificato).
- `UserPrincipal` che implementa `UserDetails` usando ruoli da `User`.
- `SecurityConfig`:
  - disabilita CSRF per le API JSON,
  - configura HTTP Basic,
  - permette senza auth:
    - `POST /api/auth/register`,
    - `POST /api/auth/login` (quando introdotto in Task 4),
  - protegge tutto il resto.
- `AuthController`:
  - `POST /api/auth/register` → 201 + `UserResponse`.

---------------------------------------------
Task 3 – World / Campaign / Session (bozza)
---------------------------------------------
- Entity:
  - `World` (id, name, description, owner optional).
  - `Campaign` (id, name, description, status enum `CampaignStatus`, world).
  - `Session` (id, title, sessionNumber, sessionDate, notes, campaign).
- Repository per ogni entity.
- DTO request/response per World, Campaign, Session.
- Service layer per CRUD base.
- Controller REST:
  - `GET /api/worlds`, `GET /api/worlds/{id}`, `POST /api/worlds`.
  - `GET /api/campaigns`, `GET /api/campaigns/{id}`, `POST /api/campaigns`.
  - `GET /api/campaigns/{campaignId}/sessions`, `POST /api/campaigns/{campaignId}/sessions`.
- Endpoint di creazione protetti con ruoli GM/ADMIN.

--------------------------------
Task 4 – Login e ownership
--------------------------------
- Aggiungere `LoginRequest` DTO.
- `POST /api/auth/login`:
  - verifica credenziali,
  - in caso di errore lancia `InvalidCredentialsException` → 401.
- `GET /api/auth/me`:
  - usa `CurrentUserService` per ottenere l’utente loggato,
  - ritorna `UserResponse`.
- Estendere World/Campaign/Session con `owner` (User):
  - popolato al momento della creazione.
- Aggiornare DTO di risposta con info owner (id + nickname).
- Configurare `AuthenticationEntryPoint` e `AccessDeniedHandler` personalizzati:
  - per restituire sempre `ApiError` JSON su 401/403.

---------------------------------------------------
Task 5 – Ruolo in registrazione + test di base
---------------------------------------------------
- `RegisterRequest.role` (string) mappato a:
  - `PLAYER` → ROLE_PLAYER
  - `DM` → ROLE_GM
  - `VIEWER` → ROLE_VIEWER
- Se role non valido → `InvalidRoleException` (400).
- Consolidare gestione `EmailAlreadyUsedException` (409).
- Aggiornare test di integrazione:
  - `AuthIntegrationTest` per registrazione con ruolo,
  - `WorldCampaignSmokeTest` per chiamate ai world/campaign con utenti GM e player.

------------------------------------------------
Task 6 – Allineamento a Repository Guidelines
------------------------------------------------
- Spostare i controller nei package `*/controller` secondo struttura DDD:
  - `AuthController`, `HealthController`, `WorldController`, `CampaignController`, `SessionController`, ecc.
- Consolidare eccezioni condivise nel package `common.error`:
  - `InvalidCredentialsException`,
  - `EmailAlreadyUsedException`,
  - `InvalidRoleException`,
  - `ResourceNotFoundException`, ecc.
- Implementare `RestExceptionHandler` che mappa:
  - 400/401/403/404/409/500 su `ApiError`.
- `UserService.authenticate`:
  - 404 per email non trovata,
  - 401 per password errata.
- Aggiornare DTO `WorldResponse`, `CampaignResponse`, `SessionResponse`:
  - includere id/nickname owner,
  - eventuale `campaignCount` per world.
- Estendere test di integrazione su:
  - login/email mancante,
  - validità codici di stato,
  - ownership e campi aggiuntivi in response.

--------------------------
Task 7 – NPC module
--------------------------
- Entity `Npc` collegata a `World` e `User` (owner):
  - campi: world, owner, name, race, roleOrClass, description, gmNotes, isVisibleToPlayers.
- DTO request/response con mapper separati:
  - GM → include `gmNotes` e NPC nascosti,
  - Player → non vede `gmNotes`, non vede NPC con `isVisibleToPlayers=false`.
- `NpcRepository`, `NpcService` con regole:
  - GM/ADMIN → vedono tutto, possono creare/modificare/eliminare.
  - PLAYER/VIEWER → solo visibili.
- `NpcController`:
  - `GET /api/npcs`, `GET /api/npcs/{id}`, `GET /api/npcs/world/{worldId}`,
  - `POST /api/npcs`, `PUT /api/npcs/{id}`, `DELETE /api/npcs/{id}`.
- Test di integrazione:
  - creazione/lista lato GM,
  - visibilità NPC lato player,
  - blocco mutate per player.

-----------------------------
Task 8 – Location module
-----------------------------
- Entity `Location` collegata a `World`, owner e parent opzionale:
  - campi: world, parentLocation(optional), owner, name, type, description, gmNotes, isVisibleToPlayers.
- DTO request/response + mapper GM/player (come NPC).
- `LocationRepository`, `LocationService`:
  - filtri per world e visibilità.
- `LocationController`:
  - `POST /api/locations`, `GET /api/locations`, `GET /api/locations/{id}`,
  - `GET /api/locations/world/{worldId}`,
  - `PUT /api/locations/{id}`, `DELETE /api/locations/{id}`.
- Test:
  - creazione/lista GM,
  - visibilità lato player,
  - blocco mutate per player.

---------------------------------------------------
Task 9 – CRUD alignment World / Campaign / User
---------------------------------------------------
- Aggiungere `PUT` + `DELETE` per `World`:
  - regole di ownership (solo owner GM/ADMIN o ADMIN globale).
- Aggiungere `PUT` + `DELETE` per `Campaign`:
  - controllo owner/ruolo,
  - evitare duplicati di nome nella stessa world (400).
- Esporre endpoint per aggiornamento dati utente:
  - `PUT /api/auth/me` con `UserUpdateRequest` (nickname, password).
- Aggiornare test World/Campaign per coprire update/delete e divieti per player.

----------------------------------------
Task 10 – Session Event Log / Timeline
----------------------------------------
- Entity `SessionEvent`:
  - session (ManyToOne),
  - owner (User),
  - title, type, description, inGameTime,
  - isVisibleToPlayers,
  - createdAt.
- DTO request/response con mapper GM/player.
- `SessionEventRepository`, `SessionEventService`:
  - filtri per sessionId e ruolo (GM vede tutto, player solo pubblici).
- `SessionEventController`:
  - `GET /api/sessions/{sessionId}/events`,
  - `POST /api/session-events`,
  - `PUT /api/session-events/{id}`,
  - `DELETE /api/session-events/{id}`.
- Test integrazione:
  - creazione/lista eventi GM,
  - filtraggio eventi per player (solo visibili),
  - blocco mutate per player.

--------------------------
Task 11 – Item module
--------------------------
- Entity `Item` collegata a `World`, owner e opzionale `Location`:
  - campi: world, location(optional), owner, name, type, rarity, description, gmNotes, isVisibleToPlayers.
- DTO request/response + mapper GM/player.
- `ItemRepository`, `ItemService`:
  - filtri per world e visibilità (come NPC/Location).
- `ItemController`:
  - `GET /api/items`, `GET /api/items/{id}`, `GET /api/items/world/{worldId}`,
  - `POST /api/items`, `PUT /api/items/{id}`, `DELETE /api/items/{id}`.
- Test integrazione:
  - creazione/lista GM,
  - visibilità lato player e 404 per item nascosti,
  - blocco mutate per player.

--------------------------------------------
Task 12 – Documentazione completa progetto
--------------------------------------------
- Generare `docs/project-documentation.md` in italiano con:
  - introduzione, obiettivi, contesto;
  - panoramica architetturale;
  - modello di dominio (tutte le entity e relazioni);
  - ruoli e sicurezza;
  - gestione errori e ApiError;
  - flussi principali (register, login, world/campaign/session, NPC, Location, Item, SessionEvent);
  - overview API (tabella endpoint);
  - schema dati logico (tabelle principali e FK);
  - testing (cosa viene coperto da JUnit);
  - possibili estensioni future.
- Documentazione coerente con codice reale (nessun endpoint inventato).

---------------------------------------------
Task 13 – Postman collection completa
---------------------------------------------
- Creare cartella `postman/`.
- Generare `postman/dd-manager.postman_collection.json` (Postman Collection v2.1) con:
  - variabile `{{baseUrl}}` = `http://localhost:8080`,
  - auth di Collection tipo Basic con `{{email}}` / `{{password}}`,
  - folder:
    - Auth
    - Worlds
    - Campaigns
    - Sessions
    - NPCs
    - Locations
    - Items
    - Session Events
  - richieste per tutti gli endpoint principali, con body JSON di esempio.


NOTA DOCUMENTAZIONE (IMPORTANTE)

Oltre ai file di documentazione presenti nel repository backend, è stata copiata la stessa documentazione anche dentro il repository frontend, nella cartella:

- back-end-docs/project-documentation.md
- back-end-docs/frontend-api-reference.md
- back-end-docs/D&d.postman_collection.json  (o nome equivalente della collection Postman)

Quando ti serve sapere:
- quali endpoint esistono,
- quali path esatti usare (/api/...),
- quali campi hanno i DTO request/response,
  puoi usare INDISTINTAMENTE:
- i file originali nel repo backend, oppure
- le copie in `back-end-docs/` nel repo frontend.

Se stai lavorando solo dentro il progetto frontend, preferisci usare i file in `back-end-docs/` per evitare di uscire dal repo corrente.


------------------------------------------------------------
2. FRONTEND TASK LIST (FE-0…FE-5)
------------------------------------------------------------

Queste task definiscono uno SPA Vue 3 semplice per usare il backend.

-----------------------------------
FE-Task 0 – Setup frontend (Vite)
-----------------------------------
- Creare cartella `dd-manager-frontend` a fianco del backend.
- Inizializzare progetto Vite + Vue 3 + TypeScript.
- Aggiungere dipendenze:
  - `vue`, `vue-router`, `pinia`, `axios`.
- Configurare:
  - `vite.config.ts` con plugin Vue,
  - `tsconfig.json` con `"strict": true`.
- `.env.development`:
  - `VITE_API_BASE_URL=http://localhost:8080`.
- Struttura `src/`:
  - `main.ts`, `App.vue`
  - `router/index.ts`
  - `store/authStore.ts`
  - `api/httpClient.ts`
  - `views/LoginView.vue`, `views/WorldsView.vue`
  - `components/AppHeader.vue`, `components/AppLayout.vue`
- `httpClient.ts`:
  - Axios instance con baseURL da `VITE_API_BASE_URL`.
  - funzione `setAuthCredentials(email, password)` che configura un interceptor Basic.
  - funzione `testHealth()` che chiama `GET /api/health`.
- `authStore.ts` (Pinia):
  - stato: email, password, nickname, roles[], isAuthenticated.
  - azioni: `login(email, password)`, `logout()`, usando `httpClient` e `/api/auth/me`.
- Router:
  - rotte `/login`, `/worlds`.
  - guardia che reindirizza a `/login` se non autenticato.
- `LoginView.vue`:
  - form email/password, chiama `authStore.login`, su successo `router.push('/worlds')`.
- `WorldsView.vue`:
  - chiama `testHealth()` e mostra un semplice “API OK / KO” (placeholder per task successive).

--------------------------------------------
FE-Task 1 – Flusso Auth completo (UI)
--------------------------------------------
- Completare `LoginView` con gestione errori:
  - mostra messaggi derivati dal `message` di `ApiError` se login fallisce.
- Aggiungere form di **registrazione** sulla stessa pagina o in view separata:
  - campi: email, password, nickname, ruolo (select `PLAYER`, `DM`, `VIEWER`).
  - chiama `POST /api/auth/register`.
  - su 201 → mostra messaggio “registrato, ora fai login”.
  - gestire errori 400/409 in UI.
- In `authStore`, aggiungere action `register(request)` se serve per centralizzare la logica.
- Aggiornare `AppHeader`:
  - se autenticato → mostra nickname e roles, bottone `Logout`.

----------------------------------------
FE-Task 2 – Dashboard Mondi (Worlds)
----------------------------------------
- In `WorldsView.vue`:
  - chiamare `GET /api/worlds` dopo login,
  - mostrare lista di card con:
    - name, description, ownerNickname, eventuale campaignCount.
- Se utente ha ruolo `ROLE_GM` o `ROLE_ADMIN`:
  - mostrare form/bottone “Crea mondo”:
    - `POST /api/worlds` (name, description).
  - per ogni mondo:
    - opzionale: bottoni “Modifica” (PUT) e “Elimina” (DELETE).
- Se utente è `PLAYER` / `VIEWER`:
  - sola lettura.
- Click su una card mondo → naviga a `/worlds/:id` (Task successiva).

-------------------------------------------------
FE-Task 3 – Dettaglio Mondo, Campaign e Session
-------------------------------------------------
- Creare `WorldDetailView.vue` (route `/worlds/:id`):
  - chiama `GET /api/worlds/{id}`.
  - mostra info del world.
  - sezione “Campagne”:
    - `GET /api/campaigns/world/{worldId}`.
    - lista campagne con nome, status, ownerNickname.
    - se GM/ADMIN:
      - form “Nuova campagna” → `POST /api/campaigns`.
      - eventuale edit/delete via `PUT`/`DELETE`.
- Creare `CampaignDetailView.vue` (route `/campaigns/:id`):
  - chiama `GET /api/campaigns/{id}`.
  - sezione “Sessioni”:
    - `GET /api/campaigns/{campaignId}/sessions`.
    - GM/ADMIN:
      - form “Nuova sessione” → `POST /api/campaigns/{campaignId}/sessions`.
  - Click su una sessione → route `/sessions/:id`.

-------------------------------------------
FE-Task 4 – NPC / Location / Item views
-------------------------------------------
- In `WorldDetailView`, aggiungere tab o sezione:
  - “NPC del mondo”:
    - `GET /api/npcs/world/{worldId}`.
  - “Location del mondo”:
    - `GET /api/locations/world/{worldId}`.
  - “Item del mondo”:
    - `GET /api/items/world/{worldId}`.
- Per PLAYER/VIEWER:
  - mostrare solo dati restituiti (backend già filtra in base a `isVisibleToPlayers`).
- Per GM/ADMIN:
  - opzione minima:
    - form singolo per creare NPC/Location/Item con i campi principali.
    - chiamate:
      - `POST /api/npcs`
      - `POST /api/locations`
      - `POST /api/items`
- Non è necessario implementare subito edit/delete in UI (opzionale).

-------------------------------------------
FE-Task 5 – Session Timeline (SessionEvent)
-------------------------------------------
- Creare `SessionDetailView.vue` (route `/sessions/:id`):
  - recuperare info base della sessione (se esiste un endpoint, altrimenti usare solo id e campaign).
  - chiamare `GET /api/sessions/{sessionId}/events`.
  - visualizzare la timeline come lista:
    - `title`, `description`, `inGameTime`, `createdAt`.
- Per GM/ADMIN:
  - form “Nuovo evento”:
    - `POST /api/session-events`.
  - bottoni opzionali “Modifica”/“Elimina”:
    - `PUT /api/session-events/{id}`,
    - `DELETE /api/session-events/{id}`.
- Per PLAYER/VIEWER:
  - vedere solo eventi pubblici (il backend filtra su `isVisibleToPlayers`).
- Gestire errori 404/403 in modo user-friendly (messaggi brevi in UI).

-------------------------------------------
FE-Task 6 (opzionale) – Cosmesi e extra
-------------------------------------------
Task opzionale, solo se serve:

- Migliorare UI:
  - layout più carino (CSS, magari un framework leggero o solo custom).
  - loader/spinner durante chiamate HTTP.
  - messaggi toast per successi/errori.
- Aggiungere una “Dashboard riassuntiva”:
  - endpoint `/api/summary` (da implementare nel backend se non esiste)
  - counts: num worlds, campaigns, sessions, npc, locations, items.
- Preparare una modalità “demo” con utenti preconfigurati per la presentazione.

------------------------------------------------------------
FINE FILE
------------------------------------------------------------

---
> Source: [chattiplus/D-Dmenager-Front](https://github.com/chattiplus/D-Dmenager-Front) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
