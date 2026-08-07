## mi-turno

> App PWA de cálculo de nómina para trabajadores por turnos en Colombia. Vanilla JS sin build obligatorio, React via CDN UMD, Supabase como backend. La documentación técnica completa para onboarding está en `ARCHITECTURE.md`.

# Mi Turno — guía para agentes

App PWA de cálculo de nómina para trabajadores por turnos en Colombia. Vanilla JS sin build obligatorio, React via CDN UMD, Supabase como backend. La documentación técnica completa para onboarding está en `ARCHITECTURE.md`.

## Stack

- **Frontend**: vanilla JavaScript ES5-flavored, React 18 (UMD globals: `React`, `ReactDOM`), `h = React.createElement` (no JSX, no transpilación).
- **Persistencia local**: `localStorage` via `leer()`/`grabar()` (en `js/utils/storage.js`).
- **Backend**: Supabase (auth + Postgres + Realtime). Project ID: `yrpqvmqmchsxpotytxiy`.
- **PWA**: Service Worker (`sw.js`) con precache estricto. Cada archivo nuevo necesita registro manual ahí.
- **Deploy**: Vercel (auto-deploy desde `master`).
- **i18n**: español, mezcla tú/vos (vos en frases emocionales, tú en setup formal). UI en español argentino/colombiano.

## Convenciones de código (NO romper)

- Usar `var`, NUNCA `let`/`const` (excepto en `sw.js`/`init.js` que sí usan `const`).
- Usar `function (x) { return ... }`, NUNCA arrow functions (`x => ...`) — rompe la consistencia con el codebase y v37 ya tuvo que revertir una.
- Usar `h(tag, props, ...children)`, NUNCA JSX.
- `async`/`await` y `Promise` SÍ están permitidos (usados en `services/sync-queue.js`, `services/export-email.js`, `utils/otp.js`, `utils/password-hash.js`). Mantener `.then(function (res) {})` cuando la consistencia con el archivo lo requiera, pero `async function foo() { var x = await bar(); }` es válido.
- Sin emojis en código a menos que el usuario los pida explícitamente para UX.
- Comentarios solo cuando el **porqué** no es obvio; nunca explicar **qué** hace el código.

## Carga de scripts

- Todo es global (`window.*`). Orden de `<script>` en `app.html` importa.
- Al agregar un archivo nuevo: hay que ponerlo en **`app.html`** Y en **el array `appResources` de `sw.js`** (precache). Si falta en sw.js, falla offline. Hay un linter rudimentario en `scripts/check.sh`.

## Versionado (las 3 fuentes de verdad)

Tres archivos tienen que decir lo mismo en cada release:

| Archivo | Línea |
|---|---|
| `js/config/globals.js` | `var MT_APP_VERSION = 'vNN';` |
| `sw.js` | `const SHELL_CACHE = 'mt-shell-vNN';` |
| `version.json` | `"v": "vNN"` |

**Usá siempre `scripts/bump.sh NN "Label"` para bumpear** — el sed sincroniza las 3. Hacerlo a mano causó el bug del bucle nuclear infinito en v28 (sw.js iba a v28, globals.js seguía en v27, el detector de reload fantasma loopeaba). `scripts/check.sh` aborta si están desincronizados.

## Datos: qué es local-only vs nube

| Dato | Local | Nube | Notas |
|---|---|---|---|
| Turnos cerrados | `mt_t_<uid>` *(via `dk(uid,"t")`)* | `public.turnos` | Realtime ✓ |
| Turno activo | `mt_a_<uid>` *(via `dk(uid,"a")`)* | `public.turno_activo` | Realtime ✓ (v39) |
| Salario base | `mt_s_<uid>` *(via `dk(uid,"s")`)* | `public.perfiles.salario_base` | Local manda si `sc=true` (v27) |
| Flag salario configurado | `mt_sc_<uid>` *(via `dk(uid,"sc")`)* | — | Local-only por device |
| PIN | `mt_pin_<uid>` | `public.pin_lookup` | PK=pin, UNIQUE(user_id). **Upsert SIEMPRE con `{onConflict:'user_id'}`** (v36) |
| Email | — | `auth.users.email` + `pin_lookup.user_email` + `perfiles.email` | Cambiar email requiere update en cascada (v36) |
| Password | — | `auth.users` | Solo online |
| Profile name (alias "pipe") | `mt_pname_<uid>` *(via `dk(uid,"pname")`)* | — | Local-only por device (v30) |
| Profile photo | `mt_photo_<uid>` *(via `dk(uid,"photo")`)* | — | Local-only, JPEG 240×240 base64 (v30) |
| Prefs (quincenaMode, etc.) | `mt_prefs_<uid>` *(via `dk(uid,"prefs")`)* | — | Local-only |
| Sesión actual | `mt_sess` | — | Limpia en logout (preserva PIN+offline cache) |
| Marcador "device conocido" | `mt_last_user` | — | Habilita FastPinScreen (v37) |
| Sesión offline cacheada | `mt_offline_<base64(email)>` | — | Para login sin red |
| Password offline | `mt_pass_<base64(email)>` | — | PBKDF2-SHA256 + salt random (v49). Schema `{v,s,h}`. Verificación legacy plaintext para migración suave |

## Flujo de sync (offline-first)

```
Usuario hace acción
  → setX local (UI instantánea)
  → queueAction(uid, type, payload)         [sync-queue.js]
  → _scheduleFlush(uid)                     [debounce 250ms]
  → processQueue(uid)                       [IN_FLIGHT guard]
  → supaXxx() helpers                       [supabase.js]
  → Realtime channel del propio user_id dispara
  → otros devices reciben → cargarDatos() → setX
```

**Contrato crítico**: `queueAction()` MUST trigger eventual `processQueue()`. Sin el `_scheduleFlush` la cola se estancaba (v40 fix — bug que persistió varias releases porque atacaba el síntoma equivocado).

**Realtime**: solo habilitado para `turno_activo` y `turnos` (publication `supabase_realtime`). Para agregar otra tabla:
```sql
ALTER PUBLICATION supabase_realtime ADD TABLE public.<tabla>;
ALTER TABLE public.<tabla> REPLICA IDENTITY FULL;
```

## Schemas de Supabase

```
auth.users           (uid PK)              ← fuente de verdad de email + password
public.perfiles      (id PK = uid)         email, salario_base, updated_at
public.pin_lookup    (pin PK)              user_id UNIQUE, user_email
public.turnos        (id PK)               user_id, inicio, fin
public.turno_activo  (user_id PK)          id, inicio
```

**Lookups por `user_id` siempre, NUNCA por `user_email`** — el email es mutable. Hubo un bug donde el lookup por email durante una transición de email regeneraba el PIN encima del real (v36 fix).

**RLS aplicado (v46)**: `public.config`, `public.Empleados` y `public.user_config` son tablas legacy huérfanas (sin referencias en el código). Tienen RLS habilitado con deny-all (sin policies). Si necesitás resucitar alguna, agregar policy explícita. NO eliminar la RLS.

**Hardening adicional aplicado (v46)**:
- `cleanup_old_pin_for_user` y `update_pin_lookup_updated_at`: `SET search_path = pg_catalog, public` (previene hijacking de funciones por usuarios con permisos en otros esquemas)
- `handle_new_user`: revocado `EXECUTE` para `anon, authenticated, public` — sigue ejecutándose como trigger pero no es callable vía REST RPC

**Pendiente (toggle dashboard, no SQL)**: `auth_leaked_password_protection` desactivado. Activar en Auth Settings → Password Strength si se desea bloquear passwords filtradas (HaveIBeenPwned).

## Patrón OTP local (v35)

Cambios de PIN/correo/password en `ManageAccountModal` y el reset en `ForgotPinModal` siguen este patrón:

1. **confirm**: validar credenciales actuales
2. **preparando**: 15s con barra de progreso (friction implícita, no decir "validación humana")
3. **codigo**: revelar 6 dígitos generados con `crypto.getRandomValues`, usuario reescribe
4. **(opcional) newpin**: solo en forgot-pin, 2 fases (ingresar + confirmar)

El código nunca sale del dispositivo. Si necesitás replicarlo, los estilos están en `css/modals/misc.css` (`.otp-prep-*`, `.otp-reveal*`).

## Cosas que ya rompieron y por qué

| Versión | Bug | Causa | Lección |
|---|---|---|---|
| v28 | App no montaba React, pantalla blanca | `init.js` reemplazado por copia del SW | Verificar que el render entrypoint exista antes de mergear |
| v28→v29 | Bucle nuclear infinito de reloads | Las 3 versiones desincronizadas | Usar `scripts/bump.sh` siempre |
| v33→v34 | "Buenos días" en AI vs "Buenas noches" en Historial a la misma hora | Dos cálculos de saludo independientes | Un único `_saludoHora()` compartido |
| v35→v36 | Cambiar PIN tiraba 23505 o creaba filas duplicadas | `upsert` sin `onConflict` y PK=pin | `{onConflict:'user_id'}` siempre |
| v36 | Cambiar email dejaba `pin_lookup.user_email` viejo → regeneración de PIN encima del real | Lookup por email frágil | Lookup siempre por `user_id` |
| v37→v39→v40 | "Iniciar turno" en device A no aparecía en device B | `queueAction` solo guardaba en localStorage, nunca llamaba `processQueue` | El sync queue debe flushear inmediato (debounced), no solo en boot/online |
| v34 | Arrow function en `sync-queue.js` | Inconsistencia con el codebase ES5 | `function(){}` siempre |
| v67 | Experimento de accesibilidad rompió el layout (pantalla blanca/desencaje) | Tocaba CSS estructural y creaba archivos a11y no registrados en `sw.js` | A11y se agrega vía atributos ARIA + HTML semántico, **nunca** reescribiendo CSS de layout. Preservar `className`, solo cambiar el `tag` o añadir props ARIA |
| v160 | Supabase "no conecta" y sync atascado tras ajustar salario (LED ámbar pegado) | (1) `__cloudReady` validaba con `getSession()` — retorna caché aunque el token JWT esté expirado y el refresh falle. `CLOUD_MODE` quedaba `true` con la API rota. (2) `processQueue` reintentaba sin límite — bucle infinito cada 5s. (3) Indicador en Ajustes decía "Sincronizado" aunque hubiera cambios pendientes en cola | **Fix en v161**: (1) `__cloudReady` valida con consulta real a la API (`SUPA.from('perfiles').select(...)`), no solo leer caché. Distingue error de auth de error de red. (2) `MAX_SYNC_RETRIES=5` con contador `action.retries`. (3) Lee `mt_sync_queue` al calcular texto de estado en Ajustes. (4) SRI sha384 en scripts CDN |

## Accesibilidad (WCAG 2.1 AA)

La app pasa auditoría **axe-core** con **0 violaciones** en las 6 pantallas (reglas `wcag2a/2aa/21a/21aa`). Patrones establecidos (v71):

- **Tags semánticos**: `<section>`/`<main>` con `aria-label` por tab, `<h1>`/`<h2>` con jerarquía.
- **Modales**: `role="dialog"` + `aria-modal="true"` + `aria-label`.
- **Toggles**: `role="switch"` + `aria-checked` + `aria-label` con estado ("activado/desactivado").
- **Listas tappables** (filas de turno): `role="button"` + `aria-label` con contexto completo hablado.
- **Stats numéricos** en `<div>`: requieren `role="img"` para que `aria-label` sea válido (un `div` genérico **prohíbe** `aria-label` → `aria-prohibited-attr`).
- **Live regions**: `aria-live="polite"` (chat), `role="alert"`+`aria-live="assertive"` (errores), `role="status"` (turno activo, progreso PIN).
- **Inputs**: `aria-label` aunque la etiqueta visual viva en un hermano.
- **Contraste**: `--muted` = `#646b7d` (5.3:1). NO volver a `#7a8294` (reprueba AA).
- **Viewport**: NO reponer `user-scalable=no`/`maximum-scale` en `index.html` (bloquea zoom, WCAG 1.4.4).

**Reglas de oro**: cambiar `div`→`section`/`main` preserva el `className` (CSS intacto). Íconos decorativos → `aria-hidden="true"`. Nunca poner `aria-label` en un `div` sin `role`.

## Cómo testear cambios

1. `scripts/check.sh` — parse + version drift + smoke + verificación de precache.
2. `npm run test:smoke` — solo el smoke (más rápido).
3. `npm run test:e2e` — Playwright en Chromium + WebKit móvil (motor de iOS Safari).
   - Local requiere `npx playwright install --with-deps chromium webkit` la primera vez.
   - En CI corre automático en cada PR vía `.github/workflows/e2e.yml`.
4. Para sync cross-device real: abrir Chrome + Safari con la **misma cuenta**, hacer la acción, esperar < 1 s.
5. `npm run test:a11y` — auditoría axe-core (WCAG 2.1 A/AA) sobre las 6 pantallas vía `tests/a11y.mjs`. Bloquea el SW e inyecta React local (el CDN se bloquea en sandbox). Sale con código 1 si hay **cualquier** violación. Criterio: **0 violaciones** o no se mergea.

Si CI falla, los artifacts (videos del navegador en el momento del bug) están en la pestaña "Artifacts" del workflow run en GitHub Actions.

## Arquitectura de IA (v317)

El asistente usa un pipeline local por capas. El detalle completo está en `ARCHITECTURE.md`; esta guía conserva los puntos operativos que suelen romperse.

```
mensaje usuario
  -> normalización / NLP
  -> ai.js: rutas canónicas + _aiSignalRoute(t, ent)
  -> buildContext(state)
  -> ai-router.js: selección de herramientas
  -> ai-collector.js: datos locales / Supabase / CPU
  -> ai-reasoning.js: hallazgos, salience, anomalías
  -> ai-responder.js: respuesta, fuentes y acciones
  -> ai-enhanced.js: polish, humanización y verificación
```

Módulos principales:
- Entendimiento: `ai-synonyms.js`, `ai-semantic.js`, `ai-query.js`, `ai-episodes.js`, `ai-nlp.js`
- Core: `ai.js`
- Router/collector/reasoning/response: `ai-router.js`, `ai-collector.js`, `ai-reasoning.js`, `ai-responder.js`
- Dominio: `ai-insights.js`, `ai-advisor.js`, `ai-auditor.js`, `ai-knowledge.js`, `ai-help.js`, `ai-app-kb.js`, `ai-calendar.js`
- Experiencia: `ai-enhanced.js`, `ai-engage.js`, `ai-conversation.js`, `ai-history.js`, `ai-memory.js`, `ai-greeting.js`, `ai-proactive.js`, `ai-psychology.js`, `ai-achievements.js`, `voice-agent.js`, `gender-lang.js`, `audio-sfx.js`

**Pipeline unificado:** todos los módulos reciben o derivan `userContext` (objeto `c` de `buildContext`). Cada etapa debe degradar con fallback local. Si Supabase no está disponible, el asistente usa datos locales y lo comunica en la evidencia.

**Regla v317:** no resolver jerga agregando frases infinitas. Primero detectar señales de dominio con `_aiSignalRoute(t, ent)` y enviar a un intent existente. Prioridad: legal/pago injusto antes que plata genérica; bienestar antes que fallback; hipotéticos de simulación deben quedarse en las rutas de simulador.

**Variables globales de IA:** `aiClassifyIntent`, `aiAnswer`, `aiEnhancedRespond`, `aiHelpAnswer`, `aiKnowledgeSearch`, `aiCheckAchievements`, etc. Todas en `window.*`.

## Helpers útiles

- `leer(key, fallback)` / `grabar(key, val)` / `borrarKey(key)` — wrappers de localStorage con try/catch
- `dk(uid, suffix)` → `'mt_' + suffix + '_' + uid` ⚠️ **key primero, uid último** (no invertir)
- `haptic()` — feedback táctil iOS
- `fCOP(n)` — formato moneda colombiana
- `fDur(mins)` — `"2h 30m"` desde minutos
- `esFest(date)` — ¿es día festivo en CO?
- `doCalc(turnos, activo, ahora, vh)` — el calculador central
- `isOnline()` / `onOnline(fn)` — estado de red
- `withTimeout(promise, ms, label)` — para promesas de red
- `_saludoHora(date)` — saludo compartido (AI + Historial)
- `_aiNombrePersonal({session})` — alias del usuario (perfil o email)
- `window._mtHardReset()` — nuclear: limpia caches + SW + reload
- `aiClassifyIntent(text, conv, ctx)` — clasificador NLP (v77)
- `aiAnalyzeMood(text, ctx)` — detección de tono emocional (v77)
- `aiResetConv()` — resetea el estado conversacional (v95)
- `backupExport()` / `backupImport(cb)` — respaldo/restauración (v96)
- `onboardingDone()` / `onboardingMarkDone()` — flag de tour guiado (v98)
- `_connState()` → `{ k, t }` — estado de conexión: `navigator.onLine` como primer filtro de red, luego `getRealtimeStatus()` para el estado real de Supabase. Los dos checks en cascada son **correctos e intencionales**. Implementación estable — no reescribir.
- `revealConn()` — toggle del banner de conexión. Usa DOM directo intencionalmente (React state causaba re-renders que rompían el overlay, v89). Estable — no refactorizar a estado React.

## Cosas a evitar

- ❌ `let` / `const` en JS de app (sí en `sw.js`/`init.js`)
- ❌ Arrow functions
- ❌ JSX
- ❌ Optional chaining (`?.`) — rompe en Android <80 (v76 fix)
- ❌ Destructuring de arrays (`var [a,b] = ...`) — usar `var s = ...; var a = s[0]` (v76 fix)
- ❌ `transform: translateZ(0)` en contenedores de scroll — crea containing block que rompe `position:fixed` (v94 fix)
- ❌ `contain: paint` en overlays — recorta bottom sheets y banners (v93 fix)
- ❌ `display: none` en `<input type="file">` — `.click()` no funciona (v91 fix)
- ❌ `overflow: hidden` en contenedores de avatar con badge flotante (v79 fix)
- ❌ Crear archivos `*.md` sin pedido explícito
- ❌ Crear archivos nuevos sin agregarlos a `app.html` Y `sw.js`
- ❌ Bumpear versión manual (usar `scripts/bump.sh`)
- ❌ Upsert en `pin_lookup` sin `{onConflict: 'user_id'}`
- ❌ Lookup por `user_email` (usar `user_id`)
- ❌ Pushear a `master` sin pasar `scripts/check.sh`
- ❌ Habilitar RLS en `config` / `Empleados` sin definir policies antes
- ❌ Poner `aria-label` en un `div` sin `role` (usar `role="img"`/`group`)
- ❌ Reponer `user-scalable=no` en el viewport (bloquea zoom, WCAG 1.4.4)
- ❌ Tocar CSS de layout "por accesibilidad" (rompió v67) — solo ARIA + tags semánticos
- ❌ Usar estado React para overlays que pueden ir a document.body — usar DOM directo (v89 fix)

## Comandos rápidos

```bash
scripts/bump.sh 42 "Label de la release"   # bumpear las 3 versiones
scripts/check.sh                           # validar sintaxis + version drift
scripts/build.sh                           # build de producción (56 JS → 1)
git push origin master                     # deploy automático en Vercel
```

---

## Arquitectura de archivos (v112)

La app tiene dos puntos de entrada. NO confundirlos:

| Archivo | Rol | Rewrite Vercel |
|---|---|---|
| `index.html` | **Landing page** | `/` (por defecto) |
| `app.html` | **Aplicación** | `/app` y `/*` (SPA) |
| `privacy.html` | Política de privacidad | `/privacy.html` |

- El PWA manifest tiene `start_url: "/app"` → la PWA siempre abre la app, no la landing.
- La landing tiene un script que redirige a `/app` si detecta sesión previa (excepto con `?show=1`).
- El diamante 💎 en Ajustes usa `/?show=1` para forzar la landing incluso con sesión.
- `check.sh` valida los .js contra `app.html` (no contra index.html).

---

## Lecciones incorporadas (v76–v115)

Registradas durante la sesión del 3-4 de junio de 2026 con DeepSeek v4 Pro + Copilot en VS Code.

| Versión | Lección |
|---|---|
| v76 | Optional chaining (`?.`) y destructuring rompen en Android gama baja. ES5 estricto. |
| v76 | Auxilio de transporte debe usar constante global (`AUX_TRANSPORTE_2026`), nunca hardcodeado. |
| v79 | `overflow: hidden` en avatar recorta el badge de cámara. Usar `overflow: visible`. |
| v84 | LED de conexión: usa `navigator.onLine` primero (filtro de red rápido) **y luego** `getRealtimeStatus()` para el estado real de Supabase. Los dos checks en cascada son correctos e intencionales — `_connState()` los implementa bien. **No es un bug, no reescribir.** |
| v87 | `position: fixed` no funciona dentro de contenedores con `transform`, `backdrop-filter` o `will-change`. Renderizar overlays como siblings, no como children. |
| v89 | Para overlays críticos que se rompen con React state + re-render, usar DOM directo (`document.createElement` + `appendChild`). |
| v91 | `<input type="file">` con `display: none` no responde a `.click()` en muchos navegadores. Usar `position:absolute;opacity:0`. |
| v91 | Imágenes con `position:absolute;inset:0` dentro de botones necesitan `pointer-events:none`. |
| v93 | `contain: paint` en overlays recorta bottom sheets y banners. No usar en contenedores de modales. |
| v94 | `transform: translateZ(0)` en el scroll container crea un containing block que rompe TODOS los `position: fixed` descendientes. **Nunca usar en `.scr`.** |
| v95 | El estado conversacional del NLP debe resetearse al limpiar el chat (`/limpiar` → `aiResetConv()`). |
| v96 | El respaldo de datos debe validar estructura (`app === 'mi-turno'`) antes de restaurar. |
| v97 | Un build script sin tooling (`cat` en orden) reduce 56 requests a 1 sin complejidad. |
| v98 | El onboarding debe usar `localStorage` flag para no repetirse. Spotlight via `getBoundingClientRect()` + ring animado. |
| v106 | iOS: `-webkit-font-smoothing`, `touch-action:manipulation`, `font-size:16px` en inputs previene auto-zoom. |
| v108 | Botón de acción: glass morphism + anillo pulsante. Sin rojo, sin sombra 3D dura. |
| v109 | Landing page como `index.html`. Rewrites de Vercel no son confiables con CDN cache agresivo. |
| v110 | Video demo en landing: `<video autoplay muted loop playsinline>`. iOS requiere los 4 atributos. |
| v111 | La landing necesita parámetro `?show=1` para evitar redirect automático cuando se accede desde la app. |
| v112 | **Swap nuclear:** `index.html` = landing, `app.html` = app. Cero dependencia de rewrites para la raíz. Vercel sirve `index.html` por defecto — imposible de fallar. `check.sh` actualizado para validar contra `app.html`. |
| v113 | iOS auto-zoom: la lección de v106 estaba escrita pero los inputs seguían en 15px. Subidos `.inp` / `.asistente-input` / `.email-card-inp` a **16px** (mínimo real que lo evita). NO bloquear el zoom (`user-scalable=no` rompe WCAG 1.4.4) — se ataca la causa, no el síntoma. |
| v114 | Higiene multi-mano: **todo global nuevo** (backup/onboarding/ai-help/`showToast`) debe registrarse en `eslint.config.js` o el linter queda rojo y se termina commiteando con `--no-verify` (se cae la red de seguridad). `dist/` (salida de `scripts/build.sh`) va en `.gitignore`, **nunca** al repo — Vercel sirve desde la raíz. Un solo landing (`index.html`); sin copias (`landing.html`/`bak_landing.html`). El swap v112 dejó `tests/a11y.mjs` apuntando a `index.html` (landing) → repuntar a `/app.html` y saltar el onboarding (`mt_onboarding_done`) para auditar las 6 tabs. |
| v115 | El **landing también** pasa el gate de accesibilidad: texto sobre `#f5f7fb` necesita ≥4.5:1 (azul de marca `#5b86e5` ≈ 3:1 → `#3a5cb5`; grises `#8e8e93`/`#aeaeb2` → `#646b7d`). Definir overrides de modo oscuro (aclarar a `#8fb3ff`/`#a0a0a6`) para cumplir también sobre `#0a0c12`. Verificar repuntando el harness al landing temporalmente. |
| v121 | WhatsApp share: `api.whatsapp.com/send?text=...` abre chat nativo con mensaje pre-formateado. Sin servidor, sin costo. |
| v124 | TTS (Text-to-Speech): `SpeechSynthesis` API nativa. Limpiar emojis→palabras motivadoras (🏆→"campeón"). Abreviaturas: "mins"→"minutos", "m"→"minutos", "$"→"pesos". |
| v125 | IA potenciada con `ai-enhanced.js` (322 líneas): memoria conversacional (`aiRemember`), sugerencias proactivas, acciones rápidas en chat, personalidad colombiana. Pipeline: memoria → expansión → acciones → personalidad. |
| v126 | Expansión de respuestas: `_aiExpandir()` añade tips contextuales (noches=35% extra, proyección explicada, descanso legal) + ánimo colombiano aleatorio. Default para respuestas cortas. |
| v127 | STT (Speech-to-Text): botón 🎤 con `SpeechRecognition` API. Español Colombia, auto-envío. Feature-detect con fallback graceful. Botón rojo pulsante al escuchar. |
| v128 | Email Compose fix: `_aiDispatchNLP` para `email` ahora usa `_aiBuildEmail()` → devuelve `{text, action}` con EmailComposeCard real. Pipeline NLP soporta respuestas con `.action`. |
| v137-139 | **Barra inferior (home indicator):** Múltiples enfoques probados (escalón de degradado, doble-blur, color sólido). Ninguno funcionó hasta v160. |
| v160 | **Solución definitiva barra inferior (estable — no tocar):** La tab bar flota con `bottom: calc(var(--sab) + 12px)` y `padding: 10px 8px`, **idéntica al header** que usa `margin-top: calc(var(--safe-top) + 12px)`. **`html::after` eliminado** — era la causa del problema: su `z-index: 99` tapaba los **orbs animados** (`.bg-shapes`, z-index: 0) en la zona del home indicator, haciendo que se vea como vidrio sólido distinto al resto. Sin `html::after`, la zona del home indicator muestra el fondo natural (gradiente + orbs azules), igual que la zona de la barra de estado arriba del header. **NO agregar** `html::after` de vuelta. **NO** cambiar `bottom` del `.tabs` a `0` ni agregar `margin-bottom: 0` — convierte la píldora en rectángulo pegado. **NO** cambiar `border-radius` a `32px 32px 0 0`. Diseño estable. |
| v143 | **Recuperación de versiones perdidas:** un force-push reescribió `master` hacia atrás (v142→v138) y se perdieron v140-142 (perfil iOS, asistente, OG dedup, `jcenter`→`mavenCentral`). Recuperado vía `git reflog` (commit `977ce3e`) y fusionado con el linaje del fix de barra inferior — eran ortogonales salvo los 3 archivos de versión. Lección: **nunca force-push a `master`**; si hay divergencia, fusionar por archivo (`git checkout <commit> -- <archivos>`), no resetear. El refinamiento de perfil v140 puso el hint del avatar en `var(--accent)` (#5b86e5, ~3:1 sobre claro, reprueba AA) → corregido a `var(--accent-deep)` sin opacity. |
| v338 | **Tarjeta nativa de desglose reemplaza tablas que se rompían en móvil.** Las tablas HTML del desglose de recargos desbordaban en pantallas angostas. Sustituidas por una tarjeta nativa de lista agrupada (componente React con `h`, sin `<table>`). Patrón a reusar para cualquier dato tabular en chat: lista vertical con etiqueta + valor, no columnas. |
| v334 | **Motor de continuidad conversacional (que el chat no muera ni se vuelva loop).** `aiNextChips(intent, c, baseActions)` (ai-conversation.js, enganchado en ai-enhanced.js) refina los chips de seguimiento a un patrón **"profundizar + abrir"** con tres reglas: (1) **cobertura** — ofrece ángulos no explorados según contexto; (2) **anti-repetición** — no repite el chip de profundización ya mostrado; (3) **dosificación** — a lo sumo 1 chip cada ~4 turnos para dejar respirar. Complementa la revelación progresiva (v324, una sola CTA) y el deliberador (v323). `ai-conversation.js` sumado al harness del smoke. |
| v329–v333 | **Reforma Laboral Ley 2466/2025 + asesor por tiers (cálculo date-aware).** El cambio legal se aplica **por fecha de cada turno**, no con un número fijo: `getRecargoFestivo(fecha)` (dominical/festivo 75→80→90→100%), `getInicioNocturno(fecha)` (21:00→19:00 desde 25-dic-2025), `rcFactor(rk, fecha)` (recompone el componente dominical vigente). `calculator.js` usa el inicio nocturno por fecha; `RC[rk].factor` se conserva solo para labels/UI. **Tiers del asesor** (`ai-advisor.js`, `ai-knowledge.js`): T0 motor legal date-aware (v329), T1 prestaciones/cesantías/verificador "¿me pagan bien?"/fiscal (v330), T2 explicabilidad "¿cómo lo calculaste?" (v332), cierre date-aware del simulador (v331), T4 alertas de cumplimiento + barrido legal (v333). **Regla:** todo intent del asesor que el NLP clasifica DEBE tener handler alcanzable (rutas financieras en `_aiFinancieroIntent`, retorno crudo). Al tocar una fecha/porcentaje legal: actualizar estas funciones + tests smoke + `tests/ai-benchmark.mjs` (sección legal). Los textos del asistente describen el valor vigente, nunca hardcodeado. |
| v325–v327 | **Pulido visual del asistente:** "calma editorial" (v325), iconos del menú lateral más pequeños (v326), icono de app con trazo más fino y aire (v327). Cambios estéticos, sin tocar pipeline ni cálculo. |
| v324 | **Política de respuesta = revelación progresiva (progressive disclosure).** Regla profesional de diseño conversacional: mostrar lo justo, diferir lo avanzado, **una sola CTA por turno**, profundidad bajo demanda. La arquitectura ya estaba repartida; se formaliza y completa: (1) núcleo conciso (calibraciones v318-v320 sacaron el fluff); (2) profundidad ofrecida vía chips deliberados por relevancia (`aiDeliberate`, v323); (3) **`aiFocusClose(text)`** (nuevo): detecta un bloque final de 2+ preguntas apiladas y deja UNA — sin tocar el contenido del medio ni las tablas; (4) detalle completo bajo demanda (`_aiModoVerboso`). Basado en progressive disclosure (Wikipedia/NN-g) + mixed-initiative. Wireado en `_polish` de `aiAnswer` tras humanizar/referring. Benchmark 100/100 intacto (la regla solo toca el bloque de cierre, no el dato). |
| v323 | **Deliberador de módulos: "pausa y piensa qué herramienta aplica" (MRKL + SMART).** En vez de responder como flecha, la IA repasa su INVENTARIO de capacidades y elige las relevantes según el entorno. `aiDeliberate(question, intent, c)` (ai-enhanced.js) = registro de módulos-experto (`_AI_MODULE_REGISTRY`) + scorer de relevancia por contexto (sin salario→configurar=5; necesita descanso→bienestar=4; con datos→auditoría/proyección=3; mes pasado→comparación=2…), **capado a 2** (SMART: anti-saturación) y excluyendo el intent ya pedido. Patrones: **MRKL** (router sobre módulos-experto) + **SMART** (Self-Aware Agent for Tool Overuse Mitigation). Wiring NO invasivo: alimenta los follow-ups cuando no hay acciones específicas (en `aiEnrichResponse`), dando coherencia ("esto te serviría ahora") sin tocar la respuesta principal ni el score del benchmark (100/100 intacto). |
| v322 | **Saludo con chispa: resolver el "lienzo en blanco" (cold-start conversacional).** Un usuario que entra y dice "hola" sin saber qué pedir se va si no lo enganchás. El saludo (handler `intent === 'saludo'`) ahora devuelve `{text, actions}` con **chips tappables** (cero fricción, no hay que escribir) + un **gancho personalizado** con su dato real (curiosity gap: "Este mes llevás $X en N turnos"), adaptado al estado: turno activo / usuario nuevo (onboarding: configurar salario, cómo registro, qué sabés) / usuario con datos (cómo voy, proyección, me pagan bien). Basado en NN/g + diseño conversacional: NO volcar todas las features (overload), usar ejemplos concretos que inviten a explorar, ser proactivo. **Fix de plomería:** el dispatch de `_aiDispatchNLP` preservaba `action`/`chart` del retorno pero descartaba `actions` (chips) tras el enriquecimiento → agregado en `_finishEnriched` (`if (_resp.actions.length) en.actions = _resp.actions`). Cualquier handler que devuelva chips explícitos ahora los conserva. |
| v321 | **Benchmark formal de NLU + detección Out-of-Scope (CheckList + CLINC150).** Nuevo `tests/ai-benchmark.mjs` (`npm run test:benchmark`): mide la IA con metodología profesional — MFT (accuracy de intent), INV (invarianza: paráfrasis/typos/jerga → mismo intent), DIR (direccional: perturbar SÍ cambia el intent), OOS (fuera de dominio) y QLT (calidad: sin leaks/género roto/groundedness vs doCalc). Emite un puntaje compuesto 0-100. **Baseline 90/100; el hueco era OOS (33%):** la IA fabricaba respuestas de nómina para "qué hora es" / "quién ganó el mundial" (sin umbral neuronal, el clasificador/salvataje siempre adivina algo). Fix: `_aiIsOutOfScope(t)` — blocklist de ALTA PRECISIÓN de temas off-domain, gateado ANTES del NLP → declina con gracia y redirige a lo que sí sabe. **Bug que cazó el propio benchmark:** marcador `'cuantos años tiene'` con ñ nunca matcheaba porque `_aiNorm` quita la ñ → usar `'cuantos anos tiene'`. Tras pulir: **100/100** en suite endurecida (86 checks). Lección: un 100% en tu propia suite no es "nivel GPT" — es ausencia de regresiones; el valor está en el harness para seguir agregando casos adversariales. |
| v320 | **Comparación mensual calibrada (no pegarla siempre).** El motor de razonamiento (`aiReason` → `_aiComparePeriods`) añadía el bloque "Comparado con el mes pasado…" siempre que hubiera datos del mes anterior, sin mirar el intent → se pegaba a "horas ayer", "valor hora", etc. (ruido fuera de tema). Fix: `aiReason(bag, ctx, hist, intent)` ahora recibe el intent y solo incluye la comparación para intents donde viene al caso (comparativa_mes/semana, total_ganado, proyeccion, stats, eficiencia); sin intent se permite por compatibilidad. `_aiRunAgentPass` pasa el intent. Misma familia de calibración que v318 (saludo/psicología): inyectar contexto **cuando suma**, no siempre. |
| v319 | **Concordancia de género en lexicalización + misroutes de valor_hora/días (de salida real).** (1) **Bug de género que introduje en v315:** variar sustantivos a sinónimos de OTRO género rompe la concordancia: "tu turno nocturno" → "tu **jornada** nocturno" ✗, "la plata" → "la **lucas**" ✗. Regla: en `_AI_HUM_DOMAIN_SYN` solo sinónimos del MISMO género/número (turno(m)→servicio(m) ✓) o invariables (veloz, ágil, bastante). Removidos jornada/guardia/plata/dinero. (2) **`valor_hora` se manejaba solo en la cascada clásica, NO en `_aiDispatchNLP`** → al clasificar con confianza, el dispatch devolvía null y un handler genérico interceptaba ("Así vas este mes" / "aún no tenés servicios") en vez de la tarifa. Fix: agregar el caso `valor_hora` a `_aiDispatchNLP` + crear el intent `valor_hora` en el NLP ('por hora', 'cuanto gano por hora'…) que antes ni existía (lo ganaba `horas_trabajadas`). (3) "días trabajados este mes" caía en `queja_fatiga`; agregadas keywords de conteo a `stats`. **Lección:** todo intent que el NLP pueda clasificar DEBE tener handler en `_aiDispatchNLP`; si solo vive en la cascada clásica, el dispatch confiado lo saltea. |
| v318 | **Placeholder filtrado + calibración de tono (no pegar fluff a lo factual).** (1) `"Procesando acción..."` es un placeholder INTERNO de acciones del agente que se filtraba al chat: una pregunta ("¿qué incluye el sueldo base?") se misclasificaba como `configurar_salario` (acción) y, sin monto, no generaba `execute` → el placeholder quedaba como respuesta. Fix en `_aiDispatchNLP`: el branch de salario distingue **comando con monto** (ejecuta SET_SALARY), **comando sin monto** (pide el valor) y **pregunta** (explica el concepto: salario ordinario bruto, base del valor hora ÷240). Guard defensivo en `_polish` de `aiAnswer`: nunca mostrar "Procesando acción..." salvo respuesta con `execute` legítimo. (2) **Tono:** el saludo de bienvenida (`aiMemoryOnFirstMessage`) y el mensaje de hora crítica (`aiPsychMensajePorHora`, que ASUME que estás trabajando: "pasaste la noche") se pegaban a respuestas factuales. Calibrado: el welcome solo se computa para intents conversacionales (y no consume el flag si no aplica); el mensaje de hora solo dispara con `c.tieneActivo` o intents emocionales (bienestar/queja_fatiga/motivacion/estado_animo). Tests deterministas pasando `ahora`+`tieneActivo` (sin depender del reloj real). |
| v317 | **Robustez por orquestación, no por cargar frases (captador de señales).** El colombiano habla rarísimo; cargar sinónimo por sinónimo no escala. En vez de eso, `_aiSignalRoute(t, ent)` (ai.js) detecta el DOMINIO por señales sueltas (plata, legal/pago-justo, bienestar/fatiga, ayuda/app, datos-por-tiempo) y devuelve un intent que un handler EXISTENTE ya responde (o 'HELP' para `aiHelpAnswer`), reusando `_aiDispatchNLP`. Corre como último recurso, ANTES del fallback genérico: si hay señal clara, responde; si no, cae a los chips de desambiguación. Cero conocimiento nuevo — orquesta los 28 módulos. **Prioridad importa:** "me pagan mal el recargo" debe ir a legal aunque tenga 'pag' (señal de plata) → el guard de pago-injusto va ANTES de la ruta de plata. **Bug de substring:** 'cansad' no matchea "cansancio" → usar raíz 'cansa'. Guards: hipotéticos ("si meto 4 noches") → null (los maneja simulación); texto sin señal → null (no inventa). Sinónimos de jerga sumados al dict (lana, saqué, junté, me van a pagar). Filosofía: pasar de "¿reconozco esta frase?" a "¿qué señales trae y qué módulo mío las atiende?". |
| v316 | **Cierre de la microplanificación NLG (salience + referring expressions).** Completa la pipeline Reiter & Dale sobre lo de v315. (a) `aiRankFindings` (ai-reasoning.js) — salience por sorpresa: ordena hallazgos por `prioridad + min(0.9, |desvío|/100)`; el bonus acotado <1 garantiza que la prioridad entera SIEMPRE domina entre niveles (un ANOMALY=9 nunca baja de un RISK=7) y la sorpresa solo decide a IGUAL prioridad. Reemplaza el `findings.sort` plano de `aiReason`. (b) `aiReferring` (ai-enhanced.js) — referring expression generation: la 1ª mención de una fecha va completa, las repeticiones → "ese día" (preserva artículo: "del 5 de junio"→"de ese día"). Enganchado en `_polish` entre humanizar y verificar. **Nota de test:** `ai-reasoning.js` NO estaba en la lista `FILES` del smoke (su razonamiento solo se testeaba e2e) → agregado (es autocontenido en carga). Salience por prioridad ya existía; lo nuevo es el peso por sorpresa. Verdadera *aggregation* (fusionar frases) se dejó fuera: es reescritura estructural, muy riesgosa para shippear sin más red de tests. |
| v315 | **Más "inteligencia" = sumar capas, no podar (Reiter & Dale + self-refine anclado).** El razonamiento ya existía (`aiReason`, loop `Tool→Collect→Reason→Generate→Validate`) pero (1) `aiThink` se usaba para **apagar** módulos (`skipModules`) en vez de planear, y (2) la lexicalización (`aiHumanizar`) y el diccionario de sinónimos (`ai-synonyms.js`) estaban infrautilizados — el dict solo se usaba para ENTENDER la entrada, nunca para variar la SALIDA. **Fix aditivo:** (a) `_AI_HUM_DOMAIN_SYN` — sinónimos de dominio curados para salida (turno→jornada/guardia, plata→lucas) fusionados en `_AI_HUM_SYN` sin pisar claves; **se varían sustantivos referenciales, NUNCA verbos factuales** (trabajaste/ganaste se quedan literales: en un recap de datos manda la precisión, y dos tests ya dependían de eso). (b) `aiVerifyNumbers` — self-refine **append-only** anclado al oráculo `doCalc`: verifica solo el claim canónico "llevás $X" contra `truth.totalCOP` y anexa corrección si no cuadra; deliberadamente NO toca cifras por turno/ley/simulación (cero falsos positivos). Enganchado una sola vez en `_polish` de `aiAnswer` (cubre todas las rutas, resuelve la Promise). **Lección:** la auto-corrección sin ground-truth degrada (TACL 2024) — acá es segura porque el oráculo es la tabla real. Verificar SIEMPRE con `npm run test:coverage` que la mejora no "limite/desconecte" (el error histórico). |
| v314 | **Calculadoras del asesor que existían pero eran inalcanzables.** `aiAdvisorAhorro/Fiscal/Optimizador/Informe` estaban definidas, probadas y conectadas a `aiAdvisorRespond`, pero el ruteo nunca las disparaba desde lenguaje natural: "informe completo"→intent `email` (el NLP tenía `['informe',3]` en el intent email), "ganar 200 mil extra"→`reflexion`, "declarar renta"→`queja_fatiga`, "plan de ahorro"→cálculo de meta, y `/ahorro 5000000` lo robaba el **follow-up numérico** de `_aiFinancieroIntent` (leía el número como ingreso de un presupuesto previo). **Fix:** todas las rutas financieras van en `_aiFinancieroIntent` (router canónico, corre pre-NLP, maneja slash+natural, retorna **crudo** sin pasar por `aiEnhancedRespond`→`aiEnrichResponse`, que para intent `ahorro` regenera presupuesto). Los bloques explícitos van **antes** del encadenamiento offer-pick/follow-up. Quitar `['informe',3]` del intent `email` del NLP. **Lección de proceso:** una función "conectada" no es una función "alcanzable" — `tests/ai-coverage.mjs` (`npm run test:coverage`, Chromium real, modo invitado) dispara una frase natural por capacidad y falla si alguna no se renderiza. Correrlo al agregar cualquier feature de IA. Ojo: `aiAnswer` puede devolver **Promesa** (rutas async) → en tests hay que `await Promise.resolve(...)`. El smoke (vm parcial de Node) **enriquece distinto** al navegador (faltan módulos); para rutas que pasan por `aiEnhancedRespond`, validar el render end-to-end en el harness de navegador, no en el vm. |

---
> Source: [aedincen-rgb/mi-turno](https://github.com/aedincen-rgb/mi-turno) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-07 -->
