## presuntamente

> Guía para cualquier agente LLM (Claude Code, Codex, Cursor, etc.) que trabaje en este repositorio. Este fichero es la fuente; `CLAUDE.md` es un symlink a él.

# AGENTS.md — presuntamente

Guía para cualquier agente LLM (Claude Code, Codex, Cursor, etc.) que trabaje en este repositorio. Este fichero es la fuente; `CLAUDE.md` es un symlink a él.

## Qué es este proyecto

**presuntamente** es un inventario interactivo, público y open source de los casos de corrupción más relevantes en España. La misión es **reducir desinformación** ofreciendo una referencia objetiva, trazable y sin cuota política de los procedimientos judiciales relevantes.

- Sitio: `presuntamente.org` (dominio registrado en Cloudflare Registrar el 2026-05-23; canales `contacto@`, `rectificacion@` y `aportar@` operativos vía Cloudflare Email Routing. Publicación técnica vía Cloudflare Pages; DNS apex y `www` se activan desde el panel de Cloudflare cuando el maintainer lo decida).
- Licencia código: AGPL-3.0.
- Licencia contenido editorial: CC BY-SA 4.0.

## Principios irrenunciables

Cualquier acción que hagas debe respetar:

1. **Imputación ≠ condena.** Modelado como `RolEnCaso` con tipos discretos (`investigado`, `procesado`, `acusado`, `condenado`, `absuelto`, `desimputado`) y trayectoria temporal. Nunca afirmaciones que insinúen culpabilidad sin sentencia firme.
2. **Cada afirmación con su fuente y nivel.** Los `Hecho` exigen `Documento` de respaldo con nivel (1-4) visible.
3. **Tratamiento sin cuota política.** Casos vivos o cerrados, de cualquier partido, ordenados por relevancia objetiva.
4. **Presunción de inocencia en el lenguaje.** Verbos prohibidos para hechos no acreditados ("robó", "estafó", "se apropió"). Preferidos: "se investiga", "se atribuye", "consta en el informe X que…".
5. **No exposición innecesaria de personas privadas.** Sólo si tienen rol formal en el procedimiento. V-17: revisión obligatoria de anonimización cuando cierran sus roles.
6. **El git log es el changelog público.** Nunca borres información; corrige con `corregido_por` y conserva el histórico.

## Documentos de diseño

Toda decisión arquitectónica, editorial o legal está justificada en `docs/diseno/`:

- `01-modelo-de-datos.md` — entidades, atributos, 21 reglas CI (V-01..V-21).
- `02-ficha-de-caso.md` — secciones, citación inline, 10 reglas anti-desinformación (P-01..P-10).
- `03-estrategia-de-mantenimiento.md` — watchers, pipeline, uso de LLM con guardarraíles.
- `04-riesgos-legales-y-eticos.md` — marco legal, disclaimer, protocolo querella.
- `05-arquitectura-tecnica.md` — stack.
- `06-roadmap-por-fases.md` — fases ejecutables.

Si vas a tocar algo no trivial, consulta primero el doc correspondiente.

## Estructura del repositorio

```text
/AGENTS.md                  ← este fichero
/CLAUDE.md                  ← symlink → AGENTS.md
/ROADMAP.md                 ← estado operativo vivo del proyecto (leer al iniciar, actualizar al cerrar)
/DESIGN.md                  ← lenguaje visual canónico (Claude Design + Claude Code)
/README.md                  ← descripción público-facing
/CONTRIBUTING.md            ← cómo contribuir
/LICENSE                    ← AGPL-3.0 (código)
/LICENSE-CONTENT.md         ← CC BY-SA 4.0 (contenido editorial)
/LEGAL.md                   ← aviso legal (placeholder hasta publicación con dominio)
/docs/diseno/               ← documentos de diseño
/docs/roadmap/              ← histórico largo, fases cerradas y aprendizajes no canónicos del roadmap
/docs/web/pages/            ← backlog y notas por página visible del sitio (uno por ruta)
/docs/web/features/         ← ficha por feature transversal (feed, búsqueda, RichProse, archive.org, …)
/content/                   ← contenido canónico (YAML)
  casos/<slug>/
    caso.yaml
    NOTES.md                ← anotaciones internas, EXCLUIDO del build público
    hitos/  hechos/
  personas/<slug>.yaml
  organizaciones/<slug>.yaml
  documentos/<slug>.yaml
  delitos/<slug>.yaml
  signals.yaml              ← bandeja del watcher
/schemas/                   ← JSON Schemas que validan los YAML
/scripts/                   ← validación, watchers, builds
/src/                       ← código Astro
  pages/
    index.astro             ← wrapper mínimo → PgHome lang="es"
    cat/                    ← rutas catalanas (vacío en MVP)
  components/
    pages/                  ← Pg* — lógica real de cada página
  layouts/
  styles/
/.agents/skills/            ← skills locales del proyecto (canónico)
/.claude/skills/            ← symlinks de compatibilidad → /.agents/skills/
/.github/workflows/         ← CI
```

## Convenciones

### Patrón `Pg*` para páginas

Las `.astro` en `/src/pages/` son **wrappers mínimos**. La lógica real vive en componentes `Pg*` en `/src/components/pages/`. Las páginas pasan `lang` como prop.

```astro
---
// src/pages/index.astro
import PgHome from '@/components/pages/PgHome.astro';
---
<PgHome lang="es" />
```

```astro
---
// src/components/pages/PgHome.astro
import BaseLayout from '@/layouts/BaseLayout.astro';
interface Props { lang?: 'es' | 'ca'; }
const { lang = 'es' } = Astro.props;

// TODO i18n: cuando se active el catalán, añadir rama 'ca'.
const strings = { /* castellano hardcoded */ };
---
<BaseLayout lang={lang} title={...}>...</BaseLayout>
```

### I18n

Estructura ES + CAT desde día 1 (carpetas `/src/pages/cat/` listas). MVP castellano hardcoded en `Pg*` con `// TODO i18n` donde irán los catalanes. Migración a `astro-i18n` nativo cuando toque.

### Tipografía y pesos visuales

Antes de tocar UI, consulta `DESIGN.md` y respeta su escala tipográfica. Norma operativa: **no usar pesos `800`/`900`** ni tratamientos de "black bold" en componentes, controles, popovers, botones, badges, metadatos o títulos secundarios. El rango permitido en interfaz es:

- `400` para texto normal.
- `500`/`600` para controles, metadatos y énfasis funcional.
- `600`/`700` para headings y énfasis editorial justificado.

Si algo necesita más jerarquía, usar estructura, color, borde, fondo, tamaño moderado o posición; no subir a peso negro. Esta regla aplica también a HTML generado por scripts (`innerHTML`) y a estilos `:global(...)`.

### Dark mode al crear componentes

El sitio tiene tema claro y oscuro (`data-theme` en `<html>`). **Al crear o tocar un componente, piensa el dark mode desde el principio:** usa los tokens `--color-*` (que ya invierten) en vez de hex hardcodeados; si necesitas un color nuevo, añádele su variante en el bloque `:root[data-theme="dark"]` de `global.css`; para texto sobre relleno de color usa `--color-accent-fg` / `--color-accent-secondary-fg`; y nunca uses `@media (prefers-color-scheme: dark)` en un componente (no responde al toggle manual). Detalle en [`docs/web/features/dark-mode.md`](docs/web/features/dark-mode.md).

### NOTES.md por caso

Cada `/content/casos/<slug>/NOTES.md` contiene anotaciones internas: contexto, "ojo con esta fuente porque…", decisiones tomadas, recordatorios para el LLM. **Excluido del build del sitio público.** Vive sólo en el repo para humanos y agentes.

### Backlog por página en `docs/web/pages/`

Cada página visible del sitio tiene (o puede tener) su propia ficha en `docs/web/pages/<ruta>.md` con estado actual, ideas futuras, aprendizajes editoriales y pendientes operativos. Norma incorporada el 2026-05-24 al cerrar la página `/cifras` por feedback del maintainer ("podríamos hacer algo en docs/web/pages, en un fichero por página, para ir anotando ideas").

Sirve para descargar el `ROADMAP.md` global: lo específico de una página vive ahí, el roadmap solo lleva hitos transversales y estado del proyecto. Cuando se te ocurra una idea futura para una página concreta (una iteración v1.x, una variante visual, un dataset distinto), **escríbela en la ficha de esa página**, no en el ROADMAP. Cuando se cierra un cambio importante en una página, actualizar también la ficha (estado actual + aprendizajes).

Convención de nombres: `<ruta-sin-barra-inicial>.md`. Para la home, `inicio.md`. La plantilla mínima vive en [`docs/web/pages/README.md`](docs/web/pages/README.md).

### Ficha por feature transversal en `docs/web/features/`

Convención hermana de la anterior, **obligatoria para toda feature transversal**. Norma incorporada el 2026-05-24 al cerrar el feed RSS/Atom de hitos por feedback del maintainer ("cada vez que hacemos algún cambio a una feature habría que actualizar el de la propia feature [...] cada feature independiente debe de ponerse un fichero .md reflejado").

Una **página** corresponde a una ruta del sitio (`/cifras`, `/casos`); una **feature transversal** corresponde a una capacidad que cruza varias páginas o sirve a un caso de uso completo (`feed-rss-atom`, `pagefind-busqueda`, `richprose-citaciones`, `archive-org-pre-commit`, `og-images-auto`, etc.). Si la cosa cabe en una sola página, va en `docs/web/pages/`; si no, va en `docs/web/features/`.

Cada ficha recoge: qué hace · para qué sirve · cómo funciona (piezas técnicas) · estado actual · decisiones editoriales y aprendizajes · ideas futuras · pendientes operativos. La plantilla mínima vive en [`docs/web/features/README.md`](docs/web/features/README.md).

**Norma operativa, sin excepciones**:

1. **Al entregar una feature nueva**, crear su ficha en el mismo commit. Es parte del cierre de sesión, no opcional.
2. **Al modificar una feature ya entregada**, actualizar su ficha en el mismo commit que toca el código. Especialmente: "Estado actual", "Decisiones editoriales y aprendizajes" (si la modificación enseña algo nuevo) y "Pendientes operativos" (tachando lo cerrado, abriendo lo que aparezca).
3. **Las ideas futuras de la feature van en su ficha, no en el ROADMAP.** El ROADMAP sólo lleva el bullet de alto nivel ("Feed RSS/Atom de hitos `[x]`"); cualquier matiz, mejora propuesta o boceto de v1.x vive en la ficha.

Convención de nombres: `<slug-kebab-case>.md` describiendo la capacidad (no el nombre del fichero de código). Ejemplos: `feed-rss-atom.md`, `archive-org-pre-commit.md`, `richprose-citaciones.md`.

### Centralización documental

**Norma incorporada 2026-05-27.** Al escribir o actualizar documentación en el repo:

1. **Un tema, un documento canónico.** No copies tablas, paletas, reglas completas ni párrafos explicativos en varios sitios. Enlaza al canon con `[doc — "Sección"](ruta/al/doc.md#anchor)`.

2. **Cada ficha documenta sólo lo suyo.** Una ficha en `docs/web/features/` o `docs/web/pages/` recoge lo específico de esa feature o ruta (API, call-sites, decisiones propias, pendientes). Lo transversal vive en su canon:

   | Tema | Canon |
   |---|---|
   | Sistema visual e identidad | [`DESIGN.md`](DESIGN.md) |
   | Modelo de datos, reglas V/P, ficha editorial | [`docs/diseno/`](docs/diseno/) |
   | Feature transversal | su ficha en [`docs/web/features/`](docs/web/features/) |
   | Página concreta | su ficha en [`docs/web/pages/`](docs/web/pages/) |
   | Estado operativo del proyecto | [`ROADMAP.md`](ROADMAP.md) (resumen; histórico largo en [`docs/roadmap/`](docs/roadmap/)) |

3. **Marca lo ajeno como referencia, no como contenido propio.** Si mencionas un componente, schema o convención que **no pertenece** a lo que estás documentando, una línea + enlace bastan. No recapitules su diseño: el lector va al canon.

4. **Al cambiar el canon, actualiza el canon.** Si cambias el sistema de badges, toca [`DESIGN.md`](DESIGN.md#2bis-sistema-de-badges). En fichas afectadas, limita el diff a estado/decisión **propia** de esa feature (fecha + enlace al canon si el detalle ya está allí).

5. **No repitas entre secciones de la misma ficha.** "Estado actual" = hitos de entrega y deltas recientes en una línea; el cómo visual detallado vive en un solo sitio dentro del documento (normalmente "Cómo funciona" o el canon externo).

### Memoria del agente vs documentación del proyecto

**Norma incorporada 2026-05-28.** Este repo es open source y multi-desarrollador. La **memoria del agente** (auto-memoria de Claude Code, reglas de Cursor, equivalentes de otros proveedores) es **local y por desarrollador**: no se versiona, no se comparte y ningún otro agente o developer la ve. No es un canal de documentación del proyecto.

1. **Conocimiento general del proyecto → al repo, nunca sólo a la memoria.** Fuentes, convenciones, decisiones de diseño, contexto recurrente de un caso: si le sirve a *otro* developer o agente que abra el repo, va documentado en su canon (ver tabla en "Centralización documental"). Si no existe canon para ese tema, créalo o pregunta dónde ubicarlo.
2. **La memoria del agente es sólo para lo específico del developer:** preferencias personales de flujo, peculiaridades del entorno local, estilo de colaboración individual. Nada que el proyecto necesite para funcionar.
3. **Regla de decisión:** "¿esto le serviría a otra persona que abra el repo?" → sí: repo; sólo a mí o a mi setup: memoria.
4. **Si encuentras conocimiento general atrapado en memoria** (tuya o heredada de otra sesión), migrarlo al repo y dejar en memoria sólo lo personal.

### Higiene del roadmap y AGENTS.md

**Norma incorporada 2026-05-27.** `ROADMAP.md` no es un diario de sesión. Debe permitir decidir qué hacer ahora en 5 minutos.

1. **Cierre de sesión breve**: máximo 3 bullets o 120 palabras en `ROADMAP.md`. Si hace falta más detalle, va a `docs/roadmap/historial-YYYY-MM.md` y en el roadmap queda una línea con enlace.
2. **Sin listas exhaustivas de archivos en el roadmap** salvo migraciones donde la lista sea la decisión. Para cambios normales, enlaza la ficha de página/feature.
3. **Una sola entrada "Última actualización" y como mucho una "Anterior" reciente.** Las anteriores se compactan al histórico mensual.
4. **Pendientes concretos, no memoria de cómo se llegó.** El roadmap conserva bloqueantes, próximos pasos, decisiones abiertas y riesgos activos. Aprendizajes largos van a `docs/roadmap/aprendizajes.md`.
5. **`AGENTS.md` tampoco es archivo histórico.** Si una norma necesita ejemplos largos, moverlos al doc canónico correspondiente y dejar aquí la regla compacta + enlace.

### Convención de referencias y citas

**Decisión 2026-05-25, no negociable.** El proyecto **no usa el signo de sección ni los glyphs decorativos hermanos** (set retirado el 2026-05-25 — detalle histórico y razones en [`docs/roadmap/aprendizajes.md`](docs/roadmap/aprendizajes.md)) en ninguna parte: ni en prosa, ni en YAML, ni en código, ni en componentes visuales, ni en comentarios. La judicatura española escribe "apartado", el BOE escribe "Artículo 5, apartado 1", la prensa no lo usa: la convención castellana real es "apartado" + enlace markdown al destino, no el signo germánico-académico.

Sustituirlo según contexto:

1. **Referencia cruzada a otro documento del repo** → enlace markdown real al anchor del heading destino, formato `[doc XX — "Nombre de la sección"](ruta/al/doc.md#slug-github-style)`. GitHub autogenera los anchors (minúsculas, espacios → guiones, puntuación eliminada, acentos preservados). Ejemplo: `[doc 04 — "Ética"](docs/diseno/04-riesgos-legales-y-eticos.md#11-ética)`.
2. **Referencia interna al mismo documento** → texto plano entre comillas, sin link: `(ver "Sistema de badges")`. Evita anchors muertos si el heading se renombra.
3. **Cita literal de un fundamento jurídico** (sentencia, auto, BOE) en `Hecho.documentos_respaldo[].pasaje` → usar la palabra `apartado` o la numeración nativa del documento. Ejemplo: `"FJ Segundo, apartado 2.7.2, p. 134"` o `"FJ 2.7.2 (p. 134)"`. Esto reproduce el estilo real de los autos españoles.
4. **Comentario organizativo en código** (`/* === sección X === */`) → sin glyph, sólo el número/nombre.

**Sistema visual de badges**: canon en [`DESIGN.md`](DESIGN.md#2bis-sistema-de-badges). Sin glyphs decorativos retirados (detalle histórico en [`docs/roadmap/aprendizajes.md`](docs/roadmap/aprendizajes.md)).

Si vas a editar un fichero del repo y dudas si una notación es la correcta, consulta esta sección antes de actuar.

### Privacidad

Este repo es público. **No incluyas datos personales del maintainer** (dirección, teléfono, info que pueda avergonzarle). Tono puede ser personal y transparente; contenido no autobiográfico.

### Commits

- Git configurado con email noreply de GitHub (`<id>+davidchicano@users.noreply.github.com`).
- Mensajes en español, imperativo presente: "Añade ficha Plus Ultra", "Corrige tipo de hecho en pieza X".
- Una idea por commit. Commits pequeños y coherentes.

### Workflow de rama y PRs

**Política actual (decidida por el maintainer el 2026-05-21):** trabajar **directamente sobre `main`**, sin ramas ni Pull Requests, mientras dure el MVP y hasta que el maintainer indique lo contrario.

**El agente NO HACE `git add` NI `git commit` durante la sesión. Tampoco hace push.** (Norma incorporada por el maintainer el 2026-05-24 tras dos incidentes de contaminación cruzada entre sesiones paralelas, ver "Repositorio multiagéntico en paralelo" puntos 6-8.) Durante el trabajo el agente **sólo edita archivos en el working tree** y acumula cambios. El **commit final único** se ejecuta cuando el maintainer indica **explícitamente** "cerramos sesión" (o equivalente: "haz el commit", "guarda esto"). Si no hay indicación expresa, no se commitea aunque la tarea esté terminada, aunque `pnpm validate` y `pnpm build` estén verdes, y aunque el agente piense que está listo. El push a `origin/main` lo decide y lo lanza el maintainer cuando le viene bien (norma anterior reforzada el 2026-05-21). Si necesitas saber si puedes commitear, **pregunta primero**.

Excepción única: si la sesión es **monoagente confirmada** (el maintainer dice explícitamente "trabajamos solos" o equivalente, y no hay otros agentes activos sobre el repo), el agente puede hacer commits intermedios siguiendo la norma de granularidad. En la duda, asumir multiagente y no commitear hasta cierre explícito.

Razón: en fase MVP el repo tiene un solo maintainer y los ciclos de feedback se hacen en sesiones de Claude Code, no en una review formal de GitHub. Las ramas + PRs ralentizan sin aportar, pero el push sí es un acto editorial que decide el maintainer (puede querer revisar el árbol de commits, esperar a juntar varios bloques, o vetar uno antes de que salga al repositorio público).

**Cuando el maintainer reactive el modelo de ramas + PRs** (esperable cuando entren contribuyentes externos o cuando se establezcan CODEOWNERS), volver a:

- Una rama por unidad de cambio coherente.
- PR descriptivo con qué, por qué y fuentes.
- CI verde antes de merge.
- Self-merge sólo del maintainer.

Si una sesión va a tocar algo arriesgado (migración, refactor amplio, cambio que rompe convenciones), aún en política directa el agente debe **proponer crear una rama puntual y preguntar** antes de meter el cambio en main.

### Repositorio multiagéntico en paralelo

**Asume siempre que otro agente o el maintainer puede estar editando otros archivos en paralelo sobre la misma working copy.** El protocolo operativo completo (worktrees, manifiesto de sesión, mailbox compartido entre agentes, reglas de calidad de mensajes) vive en la skill **`multi-agent-orchestration`** (`.agents/skills/multi-agent-orchestration/SKILL.md`), agnóstica de proveedor y reutilizable fuera de este repo. Invocable como `/multi-agent-orchestration <acción>`.

**Hook de notificación de inbox (opt-in por proveedor)**: la skill incluye un script de referencia en `.agents/skills/multi-agent-orchestration/inbox-check-hook.py` (Python stdlib, sin dependencias) que se engancha al evento `UserPromptSubmit` y avisa al agente de mensajes nuevos al inicio de cada turn. Activación per-user, no versionada:

- **Claude Code**: editar `.claude/settings.local.json` con un bloque `hooks.UserPromptSubmit` cuyo `command` apunte a `${CLAUDE_PROJECT_DIR}/.agents/skills/multi-agent-orchestration/inbox-check-hook.py`.
- **Codex CLI**: editar `.codex/hooks.json` (repo) o `~/.codex/hooks.json` (global) con la misma estructura, usando la ruta **absoluta** al script (Codex no resuelve `CLAUDE_PROJECT_DIR`).

El bloque JSON exacto está en la sección "Reference inbox-notification hook" de la skill. El hook es seguro de activar siempre: si no hay sesión abierta en el worktree actual, sale en silencio.

Normas mínimas que el agente debe seguir siempre en este repo, aunque no use la skill:

1. **Sesiones paralelas → `git worktree` obligatorio.** Nunca dos sesiones sobre la misma working copy. El comando agnóstico es `git worktree add <ruta> -b session/<slug>`. Razón histórica: el `.git/index` es global del repo; dos sesiones sobre la misma working copy comparten staging, y un `git add` de la otra sesión entra automáticamente en tu siguiente `git commit`. Hubo dos incidentes reales el 2026-05-22 y 2026-05-24 antes de adoptar worktrees.
2. **Stagea siempre por ruta explícita.** Nunca `git add .`, `git add -A`, `git add -u` ni patrones genéricos. Lista las rutas concretas. Antes y después de `git add`, lanza `git status -s` y verifica que sólo aparecen como `A` los archivos que querías stagear.
3. **No hagas `git add` ni `git commit` durante la sesión** (norma reforzada el 2026-05-24). Edita el working tree, acumula cambios, y commitea **un único commit al cierre** cuando el maintainer diga explícitamente "cerramos sesión" (o equivalente). Esto vale incluso con worktrees, por simplicidad y trazabilidad editorial.
4. **Si dependes de un archivo creado por la sesión paralela**, no lo incluyas en tu commit. Sustituye la referencia por otra ya commiteada en main, o resuelve cuando ambos commits hayan aterrizado.
5. **Para `ROADMAP.md` y `docs/roadmap/`** (rutas especialmente calientes): lee primero la versión actual del fichero (puede haber cambiado mientras trabajabas), añade tu sección sin pisar la de otra sesión, y guarda todas las ediciones para el commit final único. `ROADMAP.md` debe conservar contexto operativo suficiente para actuar; el histórico largo, fases cerradas y aprendizajes extensos van en `docs/roadmap/`.

Recuperación si un incidente de contaminación cruzada ocurre: `git reset HEAD~1` no destructivo (los archivos ajenos vuelven al working dir como untracked, intactos). **Antes** del reset, comprueba con `git log` y `git reflog` qué commit está en `HEAD` — si una sesión paralela commiteó entremedias, `HEAD~1` puede ser el commit de ELLA y un reset ciego revertiría su trabajo.

Si detectas que la sesión paralela ya ha commiteado algo entremedias (`git log` muestra un commit que no es tuyo intercalado), no es un problema: tus commits posteriores se aplican normalmente sobre la nueva HEAD.

### Granularidad de commits

**Un commit por sesión de trabajo / objetivo coherente, no por bloque interno arbitrario.** Norma incorporada el 2026-05-22 por feedback del maintainer ("no nos paramos a revisar todos los commits uno por uno"). La política actual de **no push** significa que el log local no es un changelog público todavía; lo lee el maintainer al revisar la sesión, no la comunidad.

Como guía operativa:

- **PR1 del caso X** ⇒ idealmente **un único commit** que arranque caso + personas + organizaciones + documentos + hitos + hechos + roles. Mensaje extenso en el cuerpo describiendo el bloque.
- **PR2/PR3+ del caso X** ⇒ un commit por PR (no por sub-bloque interno).
- **Cambios mixtos que combinen naturalezas distintas** (p. ej. cambio de schema + datos del caso, o cambio de componente UI + datos) sí merecen commits separados: la trazabilidad ahí compensa.
- **Cambios mecánicos masivos** (rename de slugs en muchos ficheros, p. ej.) se mantienen como un único commit por la legibilidad.

Excepción legítima al "uno por sesión": si la sesión incluye un cambio editorialmente delicado del que el maintainer podría querer revertir aisladamente (cambiar un nombre propio, modificar la calificación de un hecho de `acreditado` a `atribuido`, retirar una persona), aislar esa decisión en su propio commit ayuda al rollback selectivo. En el resto de los casos, un commit grande con un mensaje claro es preferible a cinco commits pequeños del mismo bloque.

### Documentos primarios descargados a `/public/documentos/`

> **Para localizar el PDF de un primario oficial**, consulta primero el catálogo técnico de portales en [`docs/fuentes/`](docs/fuentes/README.md): poder judicial, BOE y boletines autonómicos, Fiscalía, Tribunal Constitucional, organismos económicos, Congreso/Senado/Defensor del Pueblo y archivos/mirrors. Cada ficha trae endpoints, parámetros, cobertura temporal real y trampas conocidas. Si descubres algo nuevo, anótalo allí en la misma sesión.

**Norma incorporada el 2026-05-22 tras el PR2 del caso del Fiscal General del Estado.** Cuando un caso tiene asociado un documento jurisdiccional clave (sentencia, auto, BOE, informe oficial pericial), se descarga la copia íntegra al árbol del proyecto. El archivo del PDF/XML/etc. vive en:

```text
/public/documentos/<slug-del-caso>/<id-del-documento>.<ext>
```

Y en el YAML del `Documento` se cumplimentan obligatoriamente los campos:

- `ruta_local`: ruta servida estáticamente por el sitio (`/documentos/<slug-del-caso>/<id-del-documento>.<ext>`, **sin** el prefijo `/public`). Astro/Pagefind sirven `/public/` desde la raíz `/`.
- `hash_sha256`: el hash SHA-256 calculado sobre el fichero descargado (`shasum -a 256`).

**Cuándo descargar (SÍ)**:

- Sentencias jurisdiccionales (1ª instancia, apelación, casación, firme) — siempre. Las sentencias son de dominio público (art. 13 LPI, art. 235.5 LOPJ).
- Autos jurisdiccionales relevantes (procesamiento, apertura juicio oral, archivo, desimputación) cuando estén públicamente disponibles.
- Disposiciones del BOE relacionadas con el procedimiento (Reales Decretos de cese, nombramiento, indulto; Acuerdos del CGPJ publicados). El BOE ofrece PDF estable y XML estructurado en endpoints estables (`boe.es/diario_boe/{txt,pdf,xml}.php?id=BOE-A-YYYY-XXXXX` y `boe.es/boe/dias/YYYY/MM/DD/pdfs/BOE-A-YYYY-XXXXX.pdf`).
- Informes públicos de organismos oficiales (UCO, UDEF, Tribunal de Cuentas, AIReF) cuando hayan sido incorporados a la causa y sean accesibles.
- Notas oficiales largas del CGPJ o de la Fiscalía (cuando el contenido pase de unos pocos KB y sea citable extensamente).
- **BOE-marco que subyace al hecho investigado**: cuando una norma con rango de ley (Real Decreto-ley, Ley orgánica, Real Decreto) crea el instrumento administrativo o legal del que emana el acto presuntamente irregular, esa norma se cataloga como `Documento` propio del caso. Precedente: el RD-ley 25/2020 (BOE-A-2020-7311) que crea el Fondo de Apoyo a la Solvencia de Empresas Estratégicas (FASEE) catalogado en Plus Ultra el 2026-05-24 porque el FASEE es el vehículo del préstamo de 53 M € a Plus Ultra hoy investigado por el JCI nº 4 AN. El criterio operativo es "¿el hecho investigado depende causalmente de esta norma?" — si sí, la norma habilitante merece archivo local con `ruta_local` + `hash_sha256` y enlace desde el hecho por `documentos_respaldo` con cita literal del articulado pertinente. No es scope-creep: es trazabilidad legal completa entre la norma habilitante y el acto presuntamente irregular.

**Cuándo NO descargar**:

- Cobertura periodística N4 en `content/documentos/`. El mirror en archive.org lo gestiona `scripts/archivar-n4.mjs` vía `pnpm archive:catchup` (manual; requiere red); no descargar PDF local salvo investigación periodística con valor probatorio propio. El corpus de cobertura mediática general (`content/cobertura-mediatica/`) es entidad aparte — ver [doc feature — "Archivado en archive.org"](docs/web/features/archive-org-pre-commit.md).
- Autos de instrucción ordinaria no localizados públicamente (no podemos descargar lo que no está accesible).
- Documentos con `estado_acceso: acceso_restringido_pero_citable` o restricciones legales del art. 234 LOPJ / 301 LECrim (escritos de parte no notificados a terceros).

**Convención de procesamiento**:

- Antes de descargar, comprobar el destino de URL (no fetches a fuentes desconocidas sin autorización explícita del maintainer).
- Cuando hay varios mirrors públicos posibles, **cruzar hashes con un segundo mirror** para verificar integridad. Si los hashes difieren, comparar metadatos PDF (`pdfinfo`) y autoría — distintos PDFs del mismo documento pueden coexistir si uno tiene OCR/anotaciones extra.
- Calcular el hash sha256 del archivo final (`shasum -a 256 archivo`) y registrarlo en `hash_sha256` del YAML.
- En el campo `nivel_fuente_justificacion` documentar: metadatos clave del PDF (autor, fecha de creación, productor), el mirror del que se descarga, el segundo mirror usado para cruce de fidelidad si aplica, y la razón del `nivel_fuente` asignado.
- Si el documento debería ser N1 pero la URL oficial no existe todavía (caso típico: sentencia íntegra del TS antes de aparecer en CENDOJ), se modela como N3 `filtrado_verificado` con triangulación documentada; cuando aparezca en CENDOJ se eleva a N1 conservando el mismo `id` y el mismo `hash_sha256` (que sirve como prueba de fidelidad histórica).
- **Mirrors no auditables (Wuolah, Scribd, repositorios universitarios anónimos, copias colgadas en blogs personales)** NO son fuente válida aunque el documento parezca íntegro: el `hash_sha256` sólo acredita fidelidad si la procedencia es verificable (órgano emisor, mirror periodístico con autoría firmada, repositorio institucional). Si la única vía pública es un mirror no auditable, **NO descargar**: anotar el documento como `pendiente_primario` en `content/casos/<caso>/NOTES.md` con la URL candidata y esperar a fuente oficial. Aprendizaje incorporado el 2026-05-24 tras el barrido retrospectivo del caso Begoña Gómez (2026-05-23), que descartó un mirror Wuolah del auto AP Madrid 13-may-2025 por imposibilidad de auditar la cadena de custodia.
- **HTML nativo del órgano emisor es formato válido para `ruta_local`** (precedente: nota CGPJ del auto JCI nº 4 del 19-may-2026 sobre Plus Ultra, incorporada el 2026-05-23). Cuando el órgano emisor publica el documento en HTML nativo (notas CGPJ, comunicados Fiscalía, BOE en formato XML) y no hay PDF anexo, se descarga el HTML/XML íntegro a `/public/documentos/<caso>/<id>.html` (o `.xml`) con `hash_sha256` calculado igual que un PDF. El schema de `Documento.ruta_local` no restringe la extensión; el principio operativo es "archivo fiel al órgano emisor en el formato en que lo publica nativamente".
- **Verificar `fecha_publicacion` contra metadatos reales del BOE al descargar**: en el barrido retrospectivo del caso González Amador (2026-05-23) se detectó que dos `Documento` BOE ya catalogados tenían la fecha del acto administrativo (RD/Acuerdo) en lugar de la fecha del BOE. El XML estructurado del BOE (`boe.es/diario_boe/xml.php?id=BOE-A-YYYY-XXXXX`) trae el campo `<fecha_publicacion>` autoritativo. Al cumplimentar `ruta_local` + `hash_sha256` de un BOE preexistente, **verifica también `fecha_publicacion`** y corrige si está desfasada; las dos fechas suelen diferir 1-15 días (el BOE publica con retraso variable respecto al RD).

**Citas literales en `Hecho.documentos_respaldo[].pasaje`**: una vez el PDF está descargado, las citas en `pasaje` deben ser **precisas con localización** ("FALLO, p. 180", "Fundamento de Derecho Tercero, apartado 3.1, p. 147", "Hechos Probados pp. 18-21"), no genéricas. Esto permite al lector verificar la cita abriendo el PDF en la página exacta y eleva el rigor editorial al nivel de citación académica.

**Dependencia de sistema**: la extracción de texto de PDFs requiere `poppler-utils` (`pdftotext`, `pdfinfo`). En macOS se instala con `brew install poppler` (una vez por máquina).

## Archivado archive.org (manual)

El mirror de fuentes N4 y cobertura mediática lo gestiona [`scripts/archivar-n4.mjs`](scripts/archivar-n4.mjs). **No hay hook de git**: los agentes suelen commitear en sandbox sin red y un hook bloquearía el flujo. El archivado es operación manual del maintainer (o del agente cuando tenga red), fuera del commit.

Comandos:

- `pnpm archive:dry` — Lista URLs pendientes (documentos N4 + noticias de cobertura mediática; dry-run).
- `pnpm archive:catchup` — Archiva TODO el backlog pendiente del repo.
- `pnpm archive:catchup -- --caso=<slug>` — Solo un caso.
- `pnpm archive:catchup -- --limit=3` — Prueba acotada.

Detalle en [`docs/web/features/archive-org-pre-commit.md`](docs/web/features/archive-org-pre-commit.md). Sin autenticación: endpoint anónimo de archive.org, cuota ~8.000 capturas/día.

## Skills locales (`.agents/skills/`)

Skills planeadas para usar con Claude Code en este repo:

- `investigar-caso` — arrancar un caso nuevo desde URL/nombre.
- `incorporar-hito` — añadir Hito + Hechos + Documento desde un PDF de auto.
- `revisar-señales` — procesar bandeja del watcher (`content/signals.yaml`).
- `validar-repo` — ejecutar `pnpm validate` con output amigable (capa schema / V-rules mecánicas).
- `revisar-caso` — auditoría editorial cualitativa por LLM de un caso entero (capa que `validar-repo` no cubre: reglas P del doc 02, presunción de inocencia en el lenguaje, uso correcto de N4, ausencia de cuota política). Sirve tanto para revisar PRs externas de contribuyentes (`gh pr checkout <num>` + `/revisar-caso <slug>` en local) como para auto-revisarse cuando se trabajan varios casos en paralelo. Detalle del diseño en cuatro capas y checklist inicial en [`docs/roadmap/fases-cerradas.md`](docs/roadmap/fases-cerradas.md).
- `rectificar` — gestionar solicitud de rectificación entrante (cauce legal LO 2/1984, [doc 04 — "Mecanismo de rectificación"](docs/diseno/04-riesgos-legales-y-eticos.md#6-mecanismo-de-rectificación)).
- `incorporar-aporte` — procesar aporte editorial entrante desde `content/aportes/<YYYY-MM-DD-slug>.md` clasificando en uno de los tres carriles aceptados (pista a fuente o hito · corrección fáctica menor · idea sobre el sitio) y proponiendo diff editorial sobre `content/` o archivado razonado de la idea en `docs/web/`, más borrador de respuesta al aportante (cauce editorial [doc 04 — "Mecanismo de aportación editorial"](docs/diseno/04-riesgos-legales-y-eticos.md#6bis-mecanismo-de-aportación-editorial), hermano de `rectificar`).
- `anonimizar` — V-17: gestionar anonimización de persona privada.

En Fase 0 son placeholders; se implementan según se necesiten.

**Convención clave**: las skills se moldean con la experiencia, **no se fijan upfront**. Cada caso investigado refina la skill correspondiente. La primera versión es la mínima útil; cada uso aporta mejoras. No esperar a "diseñar la skill perfecta" antes de usarla.

**Ubicación canónica:** las skills viven en `/.agents/skills/`. Para mantener compatibilidad con Claude Code, cada entrada de `/.claude/skills/` debe ser un symlink relativo a su equivalente en `/.agents/skills/`.

## División de trabajo: maintainer ↔ agente

**Norma fundacional, incorporada el 2026-05-24 por feedback explícito del maintainer.** El reparto de roles entre la persona que mantiene el proyecto (de aquí en adelante "el maintainer") y el agente LLM (Claude Code u otro) es:

**El maintainer es**:

1. **Reviewer editorial**: lee diffs, detecta hallazgos, aprueba commits y pushes, vetar contenido.
2. **Mente pensante**: toma decisiones de diseño y prioridad (qué casos entran, en qué orden, qué features se cierran antes del lanzamiento, qué normas se aplican).
3. **Mano operativa para lo que el agente no puede hacer técnicamente**: descargar PDFs bloqueados por el sandbox classifier, navegar a URLs fuera de la lista blanca, ejecutar `git push`, operaciones que requieran credenciales o autenticación, abrir cuentas externas (apartado de correos, dominios, etc.).

**El maintainer NO aporta conocimiento editorial del caso.** El maintainer **no sabe** quiénes están procesados, qué fuentes existen, cuándo se dictaron los autos, qué cobertura periodística cruza qué hecho. Esa investigación es **íntegramente trabajo del agente**.

**El agente es**:

1. **Investigador autónomo**: usa `WebSearch` y `WebFetch` para localizar fuentes (notas CGPJ, BOE, sentencias en CENDOJ, cobertura periodística en líneas editoriales distintas). Cruza fuentes, verifica datos, aplica el lenguaje del [doc 04 — "Presunción de inocencia: reglas de redacción"](docs/diseno/04-riesgos-legales-y-eticos.md#3-presunción-de-inocencia-reglas-de-redacción) y los guardarraíles de las skills de investigación.
2. **Modelador**: genera los YAML conforme a los schemas (Caso · Persona · Organizacion · Documento · Hito · Hecho · RolEnCaso · Glosario · RelacionEntreCasos), aplica las 21 reglas V del modelo, respeta principios irrenunciables.
3. **Auditor cualitativo** (skill `/revisar-caso`): aplica las 10 reglas P del doc 02 y los principios irrenunciables sobre lo ya modelado, clasifica hallazgos en `BLOQUEANTE` / `SUGERENCIA` / `OK`.

**Qué el agente NUNCA pide al maintainer**:

- "¿Quién está procesado en este caso?" → lo investiga el agente.
- "¿Cuál es la URL canónica del auto X?" → la busca el agente.
- "¿Sabes si Y persona aparece en la cobertura?" → cobertura cruzada vía WebSearch.
- "¿Cuándo fue dictado el auto Z?" → busca en `poderjudicial.es`, CENDOJ, prensa N4 cruzada.
- "¿Qué delitos se atribuyen a esta persona?" → lo extrae el agente del auto / nota oficial / cobertura.

**Qué el agente SÍ pide al maintainer**, con propuesta operativa concreta y opciones cuando corresponda:

- Descarga de un PDF que el sandbox classifier bloquea (con la URL exacta y la justificación).
- Decisión editorial vinculante (prioridad entre alternativas, alcance de un PR, si se acepta una regla nueva del schema).
- Operaciones git que requieran credenciales o que sean editoriales del lado público (push, force-push, tags, releases).
- Aprobación para descargar de fuentes fuera de lista blanca DominiosOficiales cuando aplica la norma de "Documentos primarios descargados".

**Regla compacta**: el maintainer aporta **operaciones**, no **conocimiento**. Si el agente se ve necesitando saber un dato del caso, lo investiga; si se ve necesitando hacer una operación que el entorno le impide, lo pide.

## Delegación a sub-agentes por coste cognitivo

**Norma operativa incorporada el 2026-05-27.** El agente principal del proyecto debe **delegar a un sub-agente más barato** todo trabajo trivial-repetitivo, mecánico o pesado en lectura, y reservar su propio razonamiento para lo cognitivamente alto. Esto reduce coste por sesión sin sacrificar calidad, porque la decisión y el design ya han ocurrido en la conversación principal.

**Delegable a sub-agente barato** (Sonnet si el proveedor es Anthropic; el modelo barato equivalente en cualquier otro proveedor — Codex, Cursor, etc., aplica la misma filosofía con su catálogo):

- Migraciones mecánicas multi-fichero ("sustituye estas 14 instancias de `class="aviso--mostaza"` por `<Aclaracion nivel="alta">`").
- Búsqueda y lectura exhaustiva de tokens en el repo cuando ya se sabe qué se busca.
- Aplicación de un patrón ya decidido sobre N ficheros similares.
- Sweep de fuentes web N4 ya identificadas en busca de un dato concreto.
- Validación final (`pnpm validate`, `pnpm build`) y reporte del output.
- Cualquier tarea con criterio de éxito claro y baja ambigüedad.

**No delegable, lo hace el agente principal** (Opus si el proveedor es Anthropic, o el modelo top equivalente del proveedor que toque):

- Diseño de la API de un componente o de un schema.
- Decisiones editoriales sobre presunción de inocencia, lenguaje, anonimización, niveles de fuente.
- Auditoría cualitativa (skills `/revisar-caso`, `/rectificar`) y resolución de conflictos entre principios.
- Conversaciones con el maintainer y decisiones operativas vivas.
- Diff editorial que el LLM va a proponer para revisión humana.
- Cualquier paso donde la elección entre opciones cambia el resultado del proyecto.

**Cómo se delega bien**: el agente principal toma la decisión, escribe el contrato (qué inputs, qué outputs, qué éxito), invoca el sub-agente con ese contrato cerrado, y al volver **lee críticamente** el resultado en lugar de aceptarlo a ciegas. El sub-agente nunca recibe el problema abierto; recibe la tarea ya acotada.

**Cuándo no merece la pena delegar**: trabajo de un sólo fichero pequeño, cambios donde el coste de explicar el contrato supera el coste de hacerlo, situaciones donde la respuesta del sub-agente tendría que ser re-leída casi entera para validar (porque ahí el ahorro de tokens evapora).

**Filosofía agnóstica de proveedor**: la regla en abstracto es "delega lo barato cognitivamente al modelo barato del proveedor que estés usando, reserva tu razonamiento para lo alto". El reparto Opus/Sonnet es la encarnación de esta regla en Anthropic; otros proveedores tienen su pareja análoga (modelo top y modelo medio/barato).

## Workflow para agentes

0. **Lee [`/ROADMAP.md`](ROADMAP.md)** antes de hacer cualquier otra cosa. Es el estado operativo vivo del proyecto: dónde estamos, qué toca, decisiones pendientes y aprendizajes activos. **Obligatorio.** Si necesitas contexto histórico completo, sigue sus enlaces a `docs/roadmap/`; no cargues el histórico por defecto si el roadmap operativo ya basta.
1. **Antes de cambiar algo no trivial**: lee el doc de diseño correspondiente en `docs/diseno/`. Si tocas algo visual o de marca, además [`/DESIGN.md`](DESIGN.md). Si el cambio contradice un principio o regla, NO lo hagas; pregunta.
2. **Para crear/modificar contenido**: edita los archivos en el working tree. **NO hagas `git add` ni `git commit` durante la sesión** salvo que el maintainer lo pida explícitamente. Detalle en "Workflow de rama y PRs".
3. **Valida localmente cuando tengas algo terminado**: `pnpm validate` y `pnpm build`. Reportar el resultado, no commitear todavía.
4. **Si encuentras un dato sensible**: NO lo publiques sin consultar `docs/diseno/04-riesgos-legales-y-eticos.md`.
5. **Al cerrar la sesión** (cuando el maintainer diga explícitamente "cerramos sesión", "haz el commit" o equivalente): actualiza [`/ROADMAP.md`](ROADMAP.md) con estado operativo, próximos pasos y aprendizajes activos. Si el cierre necesita detalle largo, escríbelo en `docs/roadmap/historial-YYYY-MM.md` y deja en `ROADMAP.md` sólo el resumen enlazado. Si se cierra una fase o queda un aprendizaje extenso no canónico, usa `docs/roadmap/fases-cerradas.md` o `docs/roadmap/aprendizajes.md`. Después haz el commit final único con `git add` por ruta explícita + mensaje descriptivo (qué, por qué, fuentes). **No hagas `git push`** — eso sigue siendo decisión del maintainer.

## Cuando dudes

Pregunta antes de actuar. Es mejor un issue extra que un PR que viole un principio del proyecto.

---
> Source: [davidchicano/presuntamente](https://github.com/davidchicano/presuntamente) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-29 -->
