## kino-light

> App Android para ver video. Repo propio desde 2026-09-13: nació como la rama `light-magis` de

# Kino Light

App Android para ver video. Repo propio desde 2026-09-13: nació como la rama `light-magis` de
`lordmacu/arkiv` y se separó cuando dejó de tener sentido cargar con la app completa.

## `lordmacu/arkiv` NO es una referencia

Ese repo sigue vivo y tiene la app completa (torrent + web + archive.org + NUC + Magis + Ditu, con
login PocketBase y el gateway `arkiv-api`). **Son codebases distintos, no dos versiones de uno.**
Acá se borró casi todo eso: su `SourceKind` tiene ocho valores, este tiene cinco; su helper de IP
es `graph.torrentEngine.lanIp()`, el de acá es `graph.lanIp()` y no existe `TorrentEngine`.

Código leído allá es **confiadamente incorrecto** acá. Si hace falta mirarlo, que sea para
entender una decisión vieja, nunca para copiar una firma o un nombre de clase.

La historia de este repo sí sirve: los 960 commits son los de verdad de este código, filtrados para
que solo quede lo que existe hoy (`git filter-repo` borró de la historia el backend en Python
`alfa-api`, `balandro-addon` y unos informes sueltos). `git blame` y `git log` de cualquier archivo
de `app/` dicen la verdad.

### Traps medidas en este repo

- **El hook `rtk` recorta `cat` y `grep` sin avisar.** Cuando el contenido exacto importa, usar la
  herramienta Read o `command cat` / `command grep`.
- **Varias sesiones de Claude pueden compartir este árbol.** Nunca `git add -A`; stagear solo los
  archivos que tocaste.
- **Los commits van como `lordmacu`**, nunca la cuenta de trabajo, y nunca con un pie
  `Co-Authored-By: Claude` — revisar el pie de cada commit que haga un subagente.
- **adb:** usar solo el del SDK (`~/Library/Android/sdk/platform-tools/adb`, v37). Mezclarlo con
  `/opt/homebrew/bin/adb` (v36) reinicia el server y tumba todas las conexiones.
- **Instalar:** esta app es debuggable — `assembleDebug`.
- **El `.env` no está en git** (nunca lo estuvo, verificado contra toda la historia). Sin él el
  build NO falla: sale un APK sin llaves y sin firmar, en silencio. Ver `.env.example`.

## Dónde está todo lo demás

`.claude/` tiene el detalle, escrito para que una IA que abre este repo por primera vez se ubique:

- **`.claude/README.md`** — el índice, y lo mínimo si no vas a leer nada más.
- **`.claude/orientacion.md`** — qué es la app, de dónde saca el video, cómo está organizado el código.
- **`.claude/reglas.md`** — las no negociables al completo: los ocho hosts permitidos y cómo
  auditarlos, idioma, commits.
- **`.claude/trampas.md`** — **lo más valioso**: lo medido que costó horas o días y no se deduce
  leyendo el código.
- **`.claude/como-trabajar.md`** — compilar, instalar, los aparatos de prueba, y cómo verificar de
  verdad en un proyecto donde los tests verdes mienten.

### Language

Lo que ve la persona usuaria: español de Bogotá, tuteo, nunca voseo. Lo que ve un desarrollador
—código, identificadores, comentarios, KDoc, logs, mensajes de commit, specs y planes—: **inglés**.

## Regla del proyecto (no negociable)

**Todo corre dentro de la app. Cero servidor propio.**

Nada de PocketBase, gateway propio, mirror de torrents, jackett ni NUC. Si una feature necesita
alguno, o se reescribe para hablar directo desde el cliente Android, o se resigna — nunca se
reintroduce un servidor "solo para esto".

Hay **ocho** destinos de red permitidos, todos de terceros: el portal de Magis y su CDN, TMDB,
AniList, raw.githubusercontent.com, Caracol/Ditu y sus CDNs, Kilo, y el OTA del APK. La lista
completa, con qué archivo llama a cada uno y **cómo auditarla de verdad** (un grep por nombres
propios es ciego), está en **`.claude/reglas.md`**. Ahí también el idioma y las reglas de commit.

Se borra código muerto de verdad: no se comenta ni se deja tras un flag.

Reproductor: **ExoPlayer/media3 y nada más**. libVLC se borró entero — dependencia, `VlcPlayer.kt` y
sus helpers. No hay reproductor de respaldo.

## Spec

Ver `docs/superpowers/specs/2026-09-08-arkiv-light-magis-poda-design.md` (sub-proyecto 1 de 3:
poda estructural). Sub-proyecto 2 (cliente Magis+TMDB directos) y 3 (Ditu/RCN directos) vienen
después, cada uno con su propio spec.

---
> Source: [BSG-Walter/kino-light](https://github.com/BSG-Walter/kino-light) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
