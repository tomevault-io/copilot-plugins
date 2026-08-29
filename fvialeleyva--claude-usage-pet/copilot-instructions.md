## claude-usage-pet

> Widget de escritorio Windows (Electron) estilo Clippy que muestra el uso de

# Claude Usage Pet — estado del proyecto

Widget de escritorio Windows (Electron) estilo Clippy que muestra el uso de
Claude (límite de 5h, semanal, créditos) con una mascota flotante
personalizable. No oficial, no afiliado a Anthropic. Funciona con Claude
Code O con la app de escritorio de Claude "normal" (ver "Fuente de datos"
más abajo) — no hace falta ser desarrollador para usarlo. Plan original en
`C:\Users\f_via\Documents\FV\Job Search 2026\Claude Usage Pet\plan-claude-usage-pet.md`.

Repo público: https://github.com/fvialeleyva/claude_usage_pet (MIT). Local
y GitHub están sincronizados vía `git remote origin` + `gh` CLI (instalado
por Claude Code con `winget install --id GitHub.cli`, autenticado como
`fvialeleyva`). Primer release (`v0.1.0`, marcado pre-release por no estar
firmado) ya está publicado con el instalador de la Fase 5.

## Cómo correr la app

```bash
cd "C:\Users\f_via\Documents\FV\Vibe-Coding\claude-usage-pet"
npm start
```

Es dev-mode puro (`electron .`). Para reiniciar limpio durante desarrollo:
matar todos los `electron.exe` primero (`taskkill //F //IM electron.exe`),
porque el lock de instancia única (ver abajo) hace que una segunda
instancia no haga nada si ya hay una viva.

Para generar el instalador (Fase 5, ver sección dedicada más abajo):

```bash
npm run build
```

Produce `dist/Claude Usage Pet Setup 0.1.0.exe` (NSIS, sin firmar todavía)
y `dist/win-unpacked/Claude Usage Pet.exe` (la app ya armada, sin pasar
por el instalador — útil para smoke-test rápido). Matar los
`Claude Usage Pet.exe`/`electron.exe` corriendo antes de un build nuevo.

## Estado por fase

- **Fase 0** — spike de datos: confirmado que se puede leer el % de uso
  desde `~/.claude/.credentials.json` (token OAuth) contra
  `api.anthropic.com/api/oauth/usage` (endpoint no documentado, el mismo
  que usa `/usage` en Claude Code). Ver `spike/check-usage.js`.
- **Fase 1** — tray icon + panel de detalle (clona el formato de barras de
  5h/semanal/créditos).
- **Fase 2** — mascota flotante arrastrable, siempre-encima.
- **Fase 3** — notificaciones nativas al cruzar 50%/90% de cada límite.
- **Fase 4** — personalización (skins + accesorios). **Completa** — 7
  skins funcionando, ver sección dedicada abajo. Solo queda pulir detalles
  menores si Franco los pide (ver "Pendiente / ideas sueltas").
- **Fase 5** — empaquetado. **En progreso**, ver sección dedicada más
  abajo para el detalle de qué está hecho y qué falta (firma de código,
  repo público + auto-updater son las piezas que dependen de que Franco
  tome decisiones/haga trámites — no bloquean lo demás).

## Arquitectura

```
src/
  data/
    usage.js              — capa de datos primaria (Claude Code): lee el
                            token, llama al endpoint, nunca tira excepción
                            sin contexto ({ok:false,...}). Si Claude Code
                            falla por lo que sea, cae sola a
                            desktop-usage.js antes de reportar error — ver
                            sección dedicada "Fuente de datos" más abajo.
    desktop-usage.js       — capa de datos de respaldo (app de escritorio
                            de Claude "normal"): lee un historial local
                            que esa app ya escribe sola, sin token/OAuth.
  main/
    main.js               — proceso principal: tray, todas las ventanas,
                             polling, IPC
    preload.js             — puente contextBridge, compartido por las 3
                              ventanas renderer (pet, panel, customize)
    pet-state.js            — persiste posición/visibilidad de la mascota
                               (%APPDATA%/claude-usage-pet/pet-window-state.json)
    pet-appearance.js       — persiste skin elegido + accesorios
                               (pet-appearance.json, mismo directorio)
    custom-skin.js           — skin propio del usuario (ver sección
                                dedicada en Personalización)
    notifications.js        — umbrales 50/90, con reseteo cuando el % baja
    settings.js              — preferencias de la app (hoy: autostart),
                               pet-settings.json en el mismo directorio
  renderer/                — panel de detalle (click/hover en el tray)
  renderer-pet/             — la mascota flotante en sí
  renderer-customize/        — ventana "Personalizar mascota…"
assets/
  tray-icon.png, tray-icon-warning.png  — ícono del tray (32px)
  icon.ico                — ícono de la app/instalador (multi-resolución,
                             16 a 256px), generado desde app-icon.svg
  app-icon.svg             — fuente del ícono de app: el diseño de Smiley
                             con el color de severidad fijado en el
                             naranja "normal" (no es un skin, es el ícono
                             de marca de la app en sí)
  skins/                  — floppy-o.png, monitor-max.png, forbino-max.png,
                             calc-a-tron.png, action.png, mug.png (más el
                             skin "smiley" que es SVG a mano, sin imagen)
scripts/
  generate-icon.js         — ícono del tray (PNG puro, sin dependencias)
  generate-app-icon.js     — ícono de la app/instalador (`npm run
                             generate-app-icon`; usa `sharp` para
                             rasterizar app-icon.svg y `png-to-ico` para
                             el .ico multi-res). Re-correr si se ajusta
                             el diseño del ícono.
  generate-skin-placeholders.js / lib/simple-png.js
    — generador de placeholders viejo, ya no se usa (los skins de imagen
      son arte real ahora) pero se deja por si hace falta un fallback
  process-skin-art.js      — script real usado para preparar el arte de
                             Franco: recorta las grillas de Gemini y les
                             saca el fondo a cuadros con flood-fill (usa
                             `sharp`, instalado como devDependency).
                             Re-correrlo si Franco trae arte nuevo — ver
                             sección de skins.
spike/check-usage.js       — script standalone de la Fase 0, sigue sirviendo
                              para diagnosticar la conexión sin abrir la app
```

## Fuente de datos: Claude Code + fallback a la app de escritorio

Agregado 2026-08-16, a pedido de Franco (quería poder pasarle el widget a
amigos que usan Claude fuerte pero nunca instalaron Claude Code).

**El hallazgo:** la app de escritorio de Claude "normal" (no Code) guarda
sola, sin que se le pida nada, un historial local de su propio uso en
`%APPDATA%/Claude/plan-usage-history.json` — formato `{ version, samples:
[{ t, org, u: { fh, sd } }] }` donde `fh` = % del límite de 5h y `sd` = %
del semanal (mismos dos números que ya mostrábamos). **Nunca vence** —
a diferencia del token OAuth de Claude Code, no hace falta refrescar nada,
solo que la app haya escrito una muestra reciente.

Verificado a mano en la máquina de Franco (2026-08-16): escribe una
muestra nueva cada ~15 min mientras la app está activa (de ahí sale el
`STALE_AFTER_MS` de 30 min en `desktop-usage.js` — el doble del intervalo
típico, no un número inventado). **Solo confirmado en esa máquina/versión
— no hay garantía de que exista igual en todas las instalaciones**, mismo
nivel de "no documentado" que el resto del proyecto.

**Cómo se combinan (`getUsage()` en `usage.js`):**
1. Intenta Claude Code primero (token + endpoint en vivo) — es la fuente
   más completa: trae plan, créditos gastados y fecha de reseteo, que el
   historial de escritorio no tiene.
2. Si Claude Code falla por lo que sea (sin token, vencido, o el fetch
   tira error — incluido rate limit 429, verificado en vivo el mismo día
   que se armó esto: nuestro propio testing agresivo dejó el endpoint
   limitado un rato y el fallback lo cubrió sin que hiciera falta ni
   avisar), prueba con el historial de escritorio.
3. Si ninguna de las dos funciona, `combineFailures()` decide qué mensaje
   mostrar: si NINGUNA app está instalada, un mensaje genérico que
   menciona ambas opciones; si al menos una SÍ está instalada, se muestra
   el mensaje específico de esa (más accionable que uno genérico).

**En la UI:** el shape de `usage` ahora tiene un campo `source`
(`"claude-code"` | `"desktop-history"`). Cuando viene del historial de
escritorio, `subscriptionType`, `credits.spentUSD` y los `resetsAt` vienen
en `null` — la UI existente ya sabe mostrar "—"/"N/D" para eso sin cambios
(`renderer/renderer.js`), solo se agregó una nota "· vía app de
escritorio" al lado del timestamp para que no parezca que el panel se
rompió cuando faltan esos campos. `severityFor()` en `pet.js` y
`notifications.js` no necesitaron ningún cambio — ya operaban solo sobre
`fiveHour.usedPct`/`weekly.usedPct`, agnósticos a la fuente.

**No probado en vivo el flujo completo dentro de Electron** (mismo motivo
que el skin propio: la app instalada de Franco estaba corriendo y
comparte el lock de instancia única con dev). Sí se probó `getUsage()`
end-to-end fuera de Electron (`node -e "require('./src/data/usage.js')
.getUsage()..."` — funciona sin Electron porque solo usa `fs`/`os`/
`fetch`, nada de `electron` en este módulo) contra los archivos reales de
Franco, incluido el caso real de fallback por 429. Si algo falla dentro
de la app empaquetada, sospechar primero del lado IPC/main.js (que si no
tocamos) antes que de esta capa de datos.

## Fase 5 — empaquetado (en progreso)

**Hecho:**
- `electron-builder` + `png-to-ico` como devDependencies, config de build
  en `package.json` (campo `"build"`) — NSIS, `appId:
  com.claudeusagepet.app` (mismo ID que ya usaba
  `app.setAppUserModelId`), `perMachine:false` (instala por usuario, sin
  pedir admin/UAC — importante porque todavía no está firmado).
  `files` es una allowlist explícita (`src/**`, `assets/**`,
  `package.json`) para no empaquetar `scripts/`, `spike/` ni
  devDependencies sin usar en runtime.
- Ícono de app (`assets/icon.ico`, ver arriba) — sin esto electron-builder
  usa el ícono default de Electron.
- `npm run build` genera `dist/Claude Usage Pet Setup 0.1.0.exe` y
  `dist/win-unpacked/`. Probado: corre sin errores, respeta la posición y
  el skin ya guardados en `%APPDATA%/claude-usage-pet/` (mismo userData
  que dev, porque Electron lo deriva de `"name"` en `package.json`, no de
  `productName` — no hace falta migrar nada).
- Autoarranque con Windows: toggle "Iniciar con Windows" en el menú del
  tray (`setAutostart` en `main.js` + `src/main/settings.js`). Apagado
  por defecto (opt-in). Ojo con esto si se vuelve a tocar: en dev
  (`npm start`, `app.isPackaged === false`) el toggle persiste la
  preferencia pero **no** llama a `app.setLoginItemSettings` — registrar
  `electron.exe` como ítem de inicio en cada sesión de desarrollo sería
  un efecto secundario no deseado para Franco. Solo se aplica de verdad
  en la app empaquetada.
- `.gitignore` (`node_modules/`, `dist/`, `*.log`) — el proyecto todavía
  no es un repo git, pero queda listo para cuando se inicialice.

**Hecho (actualizado — ya no está pendiente):**
- **Repo público en GitHub** — https://github.com/fvialeleyva/claude_usage_pet
  (MIT, ver arriba). `README.md` con el disclaimer de "no oficial, no
  afiliado a Anthropic" ya escrito. `LICENSE` (MIT) agregado. Primer
  release `v0.1.0` (pre-release, instalador sin firmar) publicado.

**Pendiente — depende de que Franco decida/gestione algo, no de código:**
- **Firma de código Windows.** Sin esto, SmartScreen va a mostrar "editor
  desconocido" a cualquiera que no sea Franco corriendo el instalador.
  **Ojo: el plan original recomendaba Azure Trusted Signing — ya NO es
  viable.** Verificado en esta sesión (2026-08-15): Microsoft pausó el
  alta de desarrolladores individuales desde abril 2025 (solo
  organizaciones de EE.UU./Canadá con 3+ años de historial), y aun antes
  de esa pausa el registro individual ya estaba limitado a EE.UU./Canadá
  — nunca hubiera aplicado para Franco. **Camino elegido por Franco:
  SignPath Foundation** (firma gratis para proyectos open source — este
  proyecto ya califica: licencia MIT/OSI-approved, código público, ya
  tiene un release). Requiere algo de estructura (roles autor/revisor/
  aprobador, 2FA, una "política de firma" publicada) y la aprobación no
  es instantánea — ver guía paso a paso que se le dio a Franco en esta
  sesión (no repetida acá porque el proceso puede cambiar; recheckear
  signpath.org/terms.html si se retoma esto en una sesión futura).
  Alternativa paga descartada por ahora (SSL.com/Certum, ~USD 200-450/año,
  más rápida pero no gratis) — queda como plan B si SignPath no prospera.
- **`electron-updater` contra GitHub Releases** — ahora que el repo es
  público y ya hay un release, es viable técnicamente. No implementado
  todavía; siguiente paso lógico de Fase 5 si Franco lo pide.
- **Mac: pendiente a propósito, no arrancar sin que Franco confirme.**
  Discutido en esta sesión (2026-08-15/16): Franco no tiene Mac. Compilar
  y sobre todo firmar/notarizar para Mac requiere una máquina macOS real
  (Apple no lo permite desde Windows/Linux) más una cuenta de Apple
  Developer (USD 99/año, la tiene que crear y pagar él). Un build de CI
  sin firmar es técnicamente viable gratis vía GitHub Actions (el repo es
  público, da minutos de runners Mac gratis), pero **no sirve de nada sin
  alguien con un Mac real que lo pruebe** — la app tiene bugs de
  compositing/ventanas específicos de Windows ya documentados arriba
  (puntos 2-5 de "Decisiones técnicas no obvias"); Mac casi seguro va a
  tener los suyos propios, y un CI headless no los va a detectar.
  Franco le va a preguntar a un amigo desarrollador si tiene Mac y le
  interesa probarlo — el repo ya es público
  (https://github.com/fvialeleyva/claude_usage_pet) y el README ya tiene
  instrucciones de "Correr desde el código fuente" que deberían andar tal
  cual en Mac (la capa de datos usa `os.homedir()`, no hay nada
  hardcodeado a Windows salvo el propio empaquetado NSIS). No iniciar
  trabajo de empaquetado/CI para Mac hasta que haya confirmación de que
  alguien lo va a probar de verdad.

## Decisiones técnicas no obvias (leer antes de tocar la mascota)

Estas le costaron varias vueltas a Franco y a mí — no las reviertas sin
volver a probar en Windows real:

1. **Single instance lock** (`app.requestSingleInstanceLock()`). Sin esto,
   correr `npm start` dos veces deja dos tray icons y dos mascotas
   superpuestas, cada una polleando por su cuenta → rate limit 429.

2. **Ventana transparente invisible al arrancar.** Crear un
   `BrowserWindow({transparent:true, show:true})` ya posicionado a veces
   pinta completamente invisible en Windows (bug de compositing de
   Electron/Chromium). Fix: `show:false` en el constructor, posicionar y
   mostrar recién en el evento `ready-to-show`.

3. **El bug del "engorde" al arrastrar.** En Windows, mover una ventana
   `transparent:true` repetidamente con `setPosition(x,y)` infla su tamaño
   real (no solo el rect exterior) ~1px por llamada, incluso apuntando al
   mismo lugar dos veces seguidas. Probamos varias correcciones reactivas
   (throttle a rAF, guard timer cada 16ms, `thickFrame:false`) y ninguna
   alcanzaba en arrastres largos/rápidos. **El fix real:** usar
   `setBounds({x,y,width,height})` en vez de `setPosition(x,y)` durante el
   drag, declarando el tamaño explícito en cada llamada. Verificado con la
   API de Windows (`GetWindowRect`) en arrastres largos, repetidos y en
   reposo — se mantiene en 110x110 exacto. Si el bug reaparece, sospechar
   primero de cualquier código nuevo que vuelva a usar `setPosition()` en
   `petWindow`.

4. **Arrastre nativo (`-webkit-app-region:drag`) NO sirve acá.** Se probó
   como alternativa al punto 3 (evitaría el problema de raíz) pero en
   Windows Chromium no dispara `click` de forma confiable sobre esas
   zonas, así que el click-para-abrir-panel dejaba de funcionar. Por eso
   el drag es manual: Pointer Events + `setPointerCapture` (necesario
   porque la ventana es de 110px, y con `mousemove` normal el cursor se
   escapa de los límites en un arrastre rápido y el drag queda colgado).

5. **Always-on-top se "olvida".** Windows a veces saca el estado
   siempre-encima de una ventana frameless/transparente cuando se activa
   otra app. Fix: `setAlwaysOnTop(true, 'screen-saver')` (el nivel más
   alto) + reafirmarlo cada 3s vía `setInterval` mientras esté visible.

6. **Botón × de la mascota vs. drag manual.** `pointerdown`/`pointerup`
   burbujean de cualquier hijo (el botón ×) hasta `root` ANTES de que
   `click` se dispare — sin `stopPropagation()` en esos dos eventos
   (no solo en `click`), cada click en × también arrancaba/terminaba un
   drag y abría el panel de paso.

7. **Sondeo adaptativo.** 90s normalmente, 20s mientras esté en error
   (token vencido, rate limit, red) — así se reconecta solo apenas se
   resuelve, sin esperar el ciclo completo. Ver `scheduleNextPoll` /
   `refreshUsageNow` en `main.js`. El panel tiene botón "Reintentar" que
   fuerza esto manualmente.

8. **Color por severidad vía CSS, no JS.** `#pet-root` recibe una clase
   `severity-{normal,warning,critical,error}` que define
   `--severity-color`; cada skin (SVG a mano o imagen) y `#status-badge`
   leen esa variable. Así el JS no necesita saber qué skin está activo
   para pintarlo bien. Cuidado con especificidad CSS acá: `.skin.active
   .image-skin` (no `.image-skin` a secas) para que gane sobre
   `.skin{display:none}`.

9. **El atributo `hidden` no alcanza contra reglas CSS con clase.** En
   `renderer-customize/customize.js`, ocultar la sección de accesorios
   probó primero con `el.hidden = true` y no funcionó: `.options{display:
   flex}` en el stylesheet le gana en cascada al `[hidden]{display:none}`
   implícito del navegador (misma especificidad, pero el stylesheet carga
   después). Fix: pisar `el.style.display` directo en vez de `.hidden`.
   Mismo tipo de gotcha que el punto 8 — vigilar toggles de visibilidad
   nuevos por esto.

10. **Los JPG de Gemini NO tienen transparencia real.** El cuadriculado
    gris/blanco que se ve como "fondo transparente" en las imágenes que
    genera Gemini es, cuando se exportan/descargan como `.jpg`, píxeles
    opacos de verdad (JPEG no soporta canal alfa, punto). Hay que
    procesarlas con flood-fill antes de poder usarlas como asset
    transparente.

11. **Tampoco asumir que un `.png` de Canva tiene alfa real.** El
    "Quitafondos" de Canva a veces deja la exportación con fondo blanco
    sólido en vez de transparente (pasó con 4 de 5 imágenes en la primera
    entrega — se ve idéntico a simple vista, blanco uniforme en vez de
    cuadriculado, pero el resultado final es el mismo problema). **Siempre
    verificar con código, nunca a ojo:**
    ```js
    const meta = await sharp(archivo).metadata();
    console.log(meta.hasAlpha); // si es false, no hay transparencia real
    ```
    `scripts/finalize-skin-art.js` tiene el flood-fill para fondo blanco
    uniforme (más simple que el del cuadriculado — un solo color de
    referencia en vez de dos alternados). `scripts/process-skin-art.js`
    es el original para las grillas 2x2 sin procesar de Gemini. Si Franco
    trae más arte nuevo, mirar primero con qué formato/fondo viene y
    adaptar el script que corresponda.

## Personalización (Fase 4) — completa, 7 skins de fábrica + skin propio

Todos seleccionables desde "Personalizar mascota…" — se abre desde el
menú del tray (click derecho en el ícono) Y desde click derecho sobre la
mascota misma (mismo menú, ver `pet:context-menu` en `main.js`).

- **`smiley`** — el blob redondo original, dibujado a mano en SVG
  (`renderer-pet/index.html` + `pet.css`). Es el ÚNICO skin con
  accesorios (lentes/gorra/bigote) y expresión de boca animada por
  severidad. La ventana de personalización oculta la sección de
  accesorios por completo salvo que este skin esté elegido.
- **`floppy-o`, `monitor-max`, `forbino-max`, `calc-a-tron`** — los
  "amigos retro de oficina": diskette con anteojos, monitor CRT, diskette
  azul, calculadora. Arte de Franco (Gemini → recortados individualmente
  → pasados por el "Quitafondos" de Canva → `finalize-skin-art.js` para
  el flood-fill final). Fuente en
  `Job Search 2026/Claude Usage Pet/{Floppy-o,Monitor Max,Forbino Max,Calc-a-tron}.png`.
- **`action`** — el karateka, nombre visible "Action Claude". Mismo
  circuito, desde `Job Search 2026/Claude Usage Pet/JCVD no back.png`
  (sheet 2x2 de poses, se usa el cuadrante superior-izquierdo — el
  nombre del archivo es chiste de Franco, el personaje es genérico, no
  la foto de nadie real).
- **`mug`** — la tacita con moño, nombre visible "Muggy". De
  `Job Search 2026/Claude Usage Pet/Tacita no background.png` (esta sí
  salió con alfa real de Canva a la primera).

### Skin propio (v0.1.2)

Octava opción, "Mi skin" (`skin: "custom"`) — a diferencia de los otros
7, no es arte de Franco sino que **cualquier usuario puede subir su
propia imagen** desde "Personalizar mascota…" (botón "Elegir/Cambiar
imagen…", diálogo nativo de archivos).

- `src/main/custom-skin.js`: usa `nativeImage` de Electron (built-in, NO
  `sharp` — esa lib es devDependency de solo build-time, no viaja
  empaquetada en el asar) para leer, redimensionar (máx. 512px por lado,
  preservando proporción) y reexportar como PNG.
- Se guarda en `userData/custom-skin.png` — **no** en `assets/skins/`,
  porque `assets/` queda dentro del `.asar` empaquetado, de solo
  lectura. Esto es justo lo opuesto al resto de los skins (que sí son
  arte estático empaquetado) y es la única imagen de la app que se
  reescribe en runtime.
- La imagen viaja a los renderers como **data URL** (`data:image/png;
  base64,...`) vía IPC, no como ruta de archivo — evita tener que abrir
  la CSP a `file://` arbitrario. Por eso ambos `index.html` (pet y
  customize) tienen `img-src 'self' data:;` agregado a su CSP.
- El radio "Mi skin" arranca `disabled` y solo se habilita cuando ya
  existe una imagen guardada (`customSkin:get` al abrir la ventana). Si
  no, clickearlo no haría nada útil — mejor forzar el flujo por el botón
  "Elegir imagen…" primero. `appearance:set` en `main.js` tiene además
  un guard de respaldo: rechaza `skin:"custom"` si `hasCustomSkin()` es
  falso, por si algo intenta setearlo sin pasar por la UI.
- El anillo/punto de severidad y el manejo de "es un skin de imagen" ya
  eran genéricos (`appearance.skin !== "smiley"`), así que el skin
  propio los hereda gratis, sin tocar esa lógica.
- Sugerencia mostrada en la UI: imagen con fondo transparente (PNG) se
  ve mejor — pero no se exige ni se valida, cualquier formato soportado
  por `nativeImage` (png/jpg/jpeg/bmp/gif) funciona, solo que sin alfa
  se va a ver como un cuadrado en vez de recortada a la silueta.
- **No probado en vivo por mí** — Franco tenía la app instalada y
  corriendo (mismo lock de instancia única que dev, mismo `userData` que
  packaged, comparten `%APPDATA%/claude-usage-pet`), así que no pude
  levantar una instancia de desarrollo en paralelo sin cortarle la app
  que estaba usando activamente. Solo pasó `node --check` en todos los
  archivos tocados. Si algo falla, empezar por revisar acá antes que en
  otro lado.

### Indicador de estado (iterando — no es definitivo)

El indicador pasó por dos versiones y está en una tercera:

1. **Anillo de color** alrededor de la imagen (`border-radius:50% +
   box-shadow`). Se sacó: recortaba arte que no está perfectamente
   centrado — Action Claude (puño extendido) y Forbino Max (manos a los
   costados) quedaban cortados por el círculo.
2. **Punto de color en la esquina** (`#status-badge`, ver `pet.css` /
   `renderer-pet/pet.js` / mismo patrón espejado en
   `renderer-customize/`) — **implementación actual**. Círculo de 16px
   abajo a la derecha, con `--severity-color`, borde oscuro fijo
   (`rgba(0,0,0,.55)`) para que se distinga sobre cualquier fondo de
   escritorio, y un `box-shadow` de resplandor sutil (no agresivo) vía
   `color-mix`. Visible solo en skins de imagen — Smiley sigue mostrando
   el estado recoloreando su propio cuerpo, no necesita el punto
   (`applyAppearance` en `pet.js` / `applyToPreview` en `customize.js`
   togglean `#status-badge.visible` según `appearance.skin !== "smiley"`).
   La imagen ya no se recorta: `object-fit: contain` sin `border-radius`,
   se ve completa.
3. **Resplandor detrás del personaje** (`drop-shadow` siguiendo la
   silueta real, no un marco fijo) — es la dirección que a Franco más le
   gustó estéticamente, pero le preocupaba que se viera "muy luminoso"
   en crítico/rojo. Se implementó el punto primero a propósito, como
   paso intermedio de prueba, antes de invertir en ajustar la intensidad
   del glow. **Si Franco pide seguir iterando, el siguiente paso lógico
   es probar el resplandor con opacidad/blur más bajo** (no el que se
   ve en el mockup de comparación, que fue deliberadamente fuerte para
   que se notara en la comparativa) — no asumir que "no le gustó el
   resplandor", el problema señalado fue de intensidad, no de concepto.

**IMPORTANTE — por qué no hay Clippy real ni fotos de personas:**
Franco propuso originalmente usar el personaje Clippy de Microsoft y una
foto real de un actor conocido (Jean-Claude Van Damme). Se lo desaconsejé:
Clippy es un personaje con marca registrada de un tercero (mismo problema
que el plan original ya marcó para el logo de Claude — la app se
distribuye públicamente) y usar la foto real de una persona viva en un
instalador público no es algo que deba ayudarse a construir. Acordamos
personajes **originales inspirados** en esa vibra — de ahí salieron los
"amigos retro" y el karateka genérico. Si en algún momento aparece un
asset nuevo que se parece sospechosamente a un personaje con marca o a una
persona real identificable, señalarlo antes de integrarlo.

### Si Franco trae más arte

1. Guardar los `.jpg`/`.png` originales donde sea (ya se usó
   `Job Search 2026/Claude Usage Pet/` como carpeta de trabajo).
2. Mirarlos con la herramienta de lectura de imágenes antes de asumir
   nada del layout (sheets 2x2, cuadrantes con texto debajo, marcos de
   color, etc. — cada tanda de Gemini viene distinta).
3. Verificar con código si ya tiene alfa real (`sharp(...).metadata()
   .hasAlpha`) — no asumir por cómo se ve. Si no tiene, usar/adaptar
   `scripts/finalize-skin-art.js` (fondo blanco uniforme) o
   `scripts/process-skin-art.js` (cuadriculado tipo Gemini/JPG, con
   `quadrant()` hardcodeado para sheets 2x2 — ajustar coordenadas si el
   layout cambia).
4. Revisar CADA resultado con la herramienta de lectura de imágenes antes
   de darlo por bueno — el flood-fill de fondo a veces deja fantasmas
   (pasó en la primera pasada con algunos; subir `THRESHOLD`/
   `BORDER_DEPTH` en el script si vuelve a pasar).
5. El orden de los radios en `renderer-customize/index.html` es elegido a
   mano por Franco (Action Claude, Muggy, Floppy-O, Forbino Max,
   Calc-a-Tron, Monitor Max, Smiley) — no es alfabético ni por fecha de
   agregado. Si se suma un skin nuevo, preguntarle dónde va en esa lista
   en vez de agregarlo al final por defecto.
6. Agregar el nuevo skin a: `pet-appearance.js` (`SKINS`),
   `renderer-pet/index.html` (nuevo `<div class="skin image-skin">`),
   `renderer-pet/pet.js` (`SKIN_IDS`), `renderer-customize/index.html`
   (radio + preview) y `renderer-customize/customize.js` (`SKIN_IDS` +
   `skinRadios`). Sí, son 5 archivos — no hay una lista central todavía
   (candidato a refactor si esto crece más).

## Pendiente / ideas sueltas

- Las 7 skins tienen arte final puesto y probado (ya no son placeholders).
  Personalización (Fase 4) se puede dar por cerrada salvo que Franco pida
  ajustes.
- Indicador de estado: en la versión "punto" (ver sección dedicada
  arriba). Pendiente de que Franco lo vea corriendo y decida si se queda
  así o seguimos hacia el resplandor con menos intensidad.
- Fase 5 (empaquetado) es lo próximo obvio si no hay más ajustes de
  personalización pendientes.

## Riesgos / cosas a vigilar

- El endpoint no documentado puede cambiar sin aviso — degradar con
  gracia ya está resuelto (mensajes de error claros + reintento), pero si
  cambia el *formato* de la respuesta, `src/data/usage.js` necesita
  ajuste.
- Mismo riesgo, segunda vez: `plan-usage-history.json` (la fuente de
  `desktop-usage.js`) es igual de no-documentado y solo se verificó en
  una máquina — si Anthropic cambia esa ruta/formato en una actualización
  de la app de escritorio, el fallback deja de funcionar silenciosamente
  (cae a `no-desktop-history`, no rompe nada, pero ya no cubre a la gente
  sin Claude Code).
- Todavía sin firmar/empaquetar — Windows SmartScreen va a asustar a
  cualquiera que no sea Franco corriendo esto. Ver Fase 5 del plan
  original para las opciones de firma.
- No reproducir personajes con marca registrada ni likeness de personas
  reales en ningún asset nuevo (ver sección de Personalización).
- **Decisión deliberada: NO implementar refresh de OAuth token propio.**
  Franco reportó (2026-08-16) que el token expiró y tener Claude Code
  Desktop abierto de fondo no lo renovó — solo se arregló mandándole un
  mensaje a Claude Code. Se evaluó que la app hiciera el refresh sola
  (como hace Claude Code internamente contra su propio endpoint OAuth) y
  se descartó a propósito: implicaría reimplementar un protocolo OAuth no
  documentado (endpoint/client_id no verificados, no confirmar por prueba
  y error contra la cuenta real de Franco) y, más grave, escribir sobre
  `~/.claude/.credentials.json` — el mismo archivo que usa el Claude Code
  real, con riesgo de corromper su sesión de verdad si algo sale mal. En
  su lugar (v0.1.1) se mejoraron los mensajes de error en
  `src/data/usage.js` para ser claros y accionables sin mencionar la
  terminal. Si se reconsidera esto en el futuro, no intentarlo sin
  verificar el protocolo exacto contra documentación real (no
  adivinando), y nunca sin un mecanismo de escritura seguro (archivo
  temporal + rename) que preserve el resto del contenido del archivo.

---
> Source: [fvialeleyva/claude_usage_pet](https://github.com/fvialeleyva/claude_usage_pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
