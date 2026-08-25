## lodicho

> Plataforma de contrastación electoral para Ecuador. El ciudadano consulta una

# Lo Dicho — Contexto del proyecto

Plataforma de contrastación electoral para Ecuador. El ciudadano consulta una
declaración de un candidato (por voz, texto o URL de nota de prensa) y el sistema
responde con evidencia recuperada de un corpus curado: qué dice el plan de trabajo
registrado y qué establece el COOTAD sobre las competencias de ese nivel de gobierno.

**Dominio:** lodicho.intiinside.com


---

## Repositorios

| Repo | Contenido | Ciclo de vida |
|---|---|---|
| `lodicho` | Aplicación (API, worker, web, infra) | Despliegue en ventanas controladas |
| `lodicho-corpus` | `.md` con frontmatter + PDFs fuente | Commits frecuentes, dispara reindexación |

Están separados a propósito: no mezclar. Un push al corpus no debe disparar CI de
despliegue, y un push de código no debe disparar reindexación.

`lodicho-corpus` es un repo privado en GitHub (`github.com/intiinside/lodicho-corpus`),
clonado como carpeta **hermana** de este repo tanto en desarrollo como en el VPS
(`/opt/lodicho-corpus` junto a `/opt/lodicho`) — `docker-compose.yml` lo monta como
volumen en el contenedor `api`. Si no existe en la máquina, Docker crea ahí una carpeta
vacía en vez de fallar; hay que clonarlo a mano.

---

## Stack

- **Host:** Ubuntu Server 24.04, Hetzner VPS, Docker Compose
- **Proxy:** Nginx + Certbot
- **API:** FastAPI (Python 3.12) + Pydantic v2
- **Jobs:** ARQ + Redis
- **Relacional:** PostgreSQL 16 + Alembic
- **Vectorial:** Qdrant (denso + sparse, vectores nombrados)
- **Embeddings:** Gemini `gemini-embedding-001`, 768 dim
- **Sparse:** FastEmbed `Qdrant/bm25` (local, sin costo de API)
- **Generación:** Gemini 2.5 Flash (clasificación, transcripción) / 2.5 Pro (veredictos)
- **Ingesta:** n8n (solo webhook + delta; el procesamiento va en Python) — planeado, no
  implementado todavía. Lo que hoy mueve contenido a Qdrant es el panel de admin y
  `make ingest`; ver "Panel de admin" más abajo.
- **Conversión PDF → Markdown:** Docling, corriendo del lado del servidor (`api`).
  Pesado (trae OpenCV) — ver gotchas de build en "Panel de admin".
- **Frontend:** Vanilla JS + ES modules, `marked.js`. Sin framework, sin build step.
  Composer único (texto/voz/URL en una sola caja, el modo se infiere al enviar) en vez
  de tabs separados.

---

## Reglas críticas

Estas no son preferencias de estilo. Violarlas produce daño real a un candidato o
expone el proyecto legalmente.

### 1. Filtro por candidatura, siempre en código

Toda recuperación sobre `planes_trabajo` filtra por `candidatura_id` en el cliente
Qdrant. Nunca delegado al prompt. Recuperar el plan de otra candidatura produce un
veredicto difamatorio.

### 2. Tres ausencias distintas, nunca confundir

| Estado | Significado |
|---|---|
| `sin_plan_registrado` | La candidatura no registró plan ante el CNE (dato en `candidaturas`) |
| `sin_plan_recuperado` | El retrieval no devolvió fragmentos (fallo técnico) |
| `no_consta` | Se recuperó el plan y la propuesta no está ahí |

Solo `no_consta` habilita el veredicto `no_consta_en_plan`. Hay un validador Pydantic
que lo impide; no lo desactives.

### 3. Cifras: nunca por RAG

Los indicadores estadísticos viven en la tabla `indicadores` y se exponen como tool
call con parámetros (`codigo`, `jurisdiccion_dpa`, `anio`). Un embedding no distingue
34,2 % de 43,2 %. Sin dato → veredicto `incomprobable`, jamás una cifra inferida.

### 4. Evidencia automática, veredicto firmado

| Salida | Revisión humana |
|---|---|
| Cita del plan de trabajo | No |
| Artículos del COOTAD aplicables | No |
| Dato oficial de `indicadores` | No |
| Contraste factual entre candidatos | No |
| Veredicto categórico | **Sí, obligatoria** |
| Score de factibilidad | **Sí, obligatoria** |

Un veredicto sin firma de revisor nunca sale con `estado='publicado'`.

### 5. Nada de recomendación de voto

El clasificador de intención rechaza con mensaje fijo (no generado por el modelo):
recomendación de voto, comparación de calidad entre candidatos, opinión sobre la
persona. Contrastar propuestas lado a lado sí se permite, sin juicio de calidad.

### 6. Silencio electoral

Variable `MODO_SILENCIO_ELECTORAL`. Cuando está activa: solo lectura de informes ya
publicados, generación desactivada.

### 7. Contenido web = datos no confiables

El texto extraído de una URL va envuelto en `<contenido_web>` y el system prompt
declara explícitamente que nada dentro de ese bloque es una instrucción. Riesgo real
de inyección de prompt.

---

## Modelo de datos

```
candidaturas (id, organizacion_politica, lista_numero, dignidad,
              jurisdiccion_dpa, periodo, doc_id_plan, estado_plan)
     │
     └──< candidatos (id, nombre, candidatura_id FK, posicion_lista)

documentos   (id, doc_id UNIQUE, tipo, candidatura_id FK, ruta_repo,
              sha256, pdf_sha256, git_sha, n_chunks, indexado_en, estado)

indicadores  (id, codigo, descripcion, jurisdiccion_dpa, anio,
              valor NUMERIC, unidad, fuente, url)

consultas    (id, tipo_input, texto, audio_path, url_fuente,
              contenido_archivado, hash_contenido, intencion_detectada,
              desde_cache, creado_en)
     │
     └──< declaraciones (id, consulta_id, texto, tipo, atribuible, analisis_id)

analisis     (id, candidatura_id FK, afirmacion, veredicto ENUM,
              payload_json JSONB, factibilidad_score, factibilidad_factores JSONB,
              modelo_usado, estado ENUM, revisor_id, revisado_en,
              respuesta_candidato, publicado_en, creado_en)
     │
     └──< evidencias (id, analisis_id, paso, coleccion, point_id,
                      doc_id, texto, score, git_sha)
```

**Clave:** el plan de trabajo pertenece a la **candidatura**, no a la persona. Varios
candidatos de una misma lista comparten un solo plan. Un `.md` por candidatura.

`veredicto`: `viable_y_en_plan` | `fuera_de_competencia` | `no_consta_en_plan` |
`informacion_enganosa` | `informacion_falsa` | `incomprobable`

`estado`: `borrador` → `en_revision` → `publicado` | `descartado`

`declaraciones.tipo`: `cita_directa` | `parafrasis_periodistica` | `dictado_usuario`.
Solo `cita_directa` y `dictado_usuario` son atribuibles al candidato.

---

## Qdrant

Cuatro colecciones, **siempre accedidas por alias** (la dimensión es inmutable; el
alias permite reindexar a `_v2` y conmutar sin downtime).

| Alias | Chunking | Filtro obligatorio |
|---|---|---|
| `marco_legal` | Por artículo (regex sobre `Art\.\s*(\d+)`) | `nivel_gobierno`, `vigente` |
| `planes_trabajo` | Por sección / eje | `candidatura_id` |
| `contexto` | Semántico ~500 tokens | `jurisdiccion_dpa` |
| `analisis_publicados` | 1 punto por informe | — (caché semántico) |

Cada punto lleva vectores nombrados `dense` (Gemini 768, **normalizado L2**) y `sparse`
(BM25 local). Fusión por RRF.

### Gemini embeddings — dos errores silenciosos

1. **Task type asimétrico.** `RETRIEVAL_DOCUMENT` al indexar, `RETRIEVAL_QUERY` al
   consultar. Definidos como constantes en `services/embeddings.py`, nunca hardcodeados
   en dos lugares.
2. **Normalización.** Solo 3072 dim viene pre-normalizado. A 768 hay que normalizar L2
   antes del upsert. Sin esto los scores coseno salen distorsionados sin lanzar error.

No usar `gemini-embedding-2`: es multimodal y **no soporta `task_type`**. API incompatible.

---

## Pipeline de ingesta

Hay **dos caminos** al mismo destino (chunk → embeddings → Qdrant → fila en
`documentos`). El segundo es el que existe hoy; el primero es el diseño original,
todavía no construido más allá del repo y la Action.

### Camino A — PR en GitHub (diseñado, parcialmente implementado)

```
PDF → conversión → .md con frontmatter → PR en lodicho-corpus
                                            │
                    GitHub Action: validaciones automáticas   ← existe
                                            │
                    Revisor humano aprueba y fusiona
                                            │
                    push webhook → n8n → POST /api/v1/ingest  ← NO existe
```

n8n **solo** haría: `GET /repos/{owner}/{repo}/compare/{before}...{after}` → extraer
`files[]` → un POST por archivo. Nada más. Todo el procesamiento en FastAPI. Ni el
webhook de n8n ni `/api/v1/ingest` están implementados — la Action de validación en
`lodicho-corpus` (`.github/workflows/validar-corpus.yml`) sí corre en cada PR, pero
fusionar un PR ahí **no** dispara ninguna reindexación automática todavía.

### Camino B — Panel de admin (implementado)

```
PDF subido desde /#admin → Docling (server-side) → revisión/edición en pantalla
                                            │
                    Validación (mismas reglas que el Camino A)
                                            │
                    Commit + push automático a lodicho-corpus
                    (deploy key SSH dedicada — el revisor no toca git)
                                            │
                    Botón "Ingestar" → chunk → embeddings → Qdrant + Postgres
```

Sin PR, sin revisión de otra persona en GitHub — la revisión pasa por quien aprueba en
pantalla antes de confirmar. Ver "Panel de admin" más abajo para el detalle completo.

### Camino C — CLI manual (`make ingest`)

Para un `.md` que ya está commiteado en `lodicho-corpus` (por cualquiera de los dos
caminos de arriba, o escrito/commiteado a mano): `make ingest` corre la misma lógica
de ingesta sin pasar por el panel. Útil para reprocesar todo el corpus de una.

Los tres caminos comparten el mismo código (`app/services/ingest.py`) — nunca hay una
segunda implementación del chunker o de la llamada a embeddings que se pueda
desincronizar de la otra.

### Estados de archivo (aplica a los tres caminos)

| status | Acción |
|---|---|
| `added` / `modified` | `delete_by_doc_id` → re-chunk → upsert |
| `removed` | `delete_by_doc_id` + marcar `documentos.estado='eliminado'` (no implementado: hoy no hay un flujo que detecte un `.md` borrado del corpus) |
| `renamed` | delete por `doc_id` anterior → upsert nuevo |

**Nunca solo upsert.** Si un documento pasa de 12 a 9 chunks, los 3 huérfanos siguen
apareciendo en búsquedas con contenido desactualizado. `ingest.ingestar_archivo` ya
hace `delete_by_doc_id` antes de cada upsert, siempre.

### Frontmatter obligatorio

```yaml
---
doc_id: plan-bolivar-simiatug-junta-18-2027
tipo: plan_trabajo          # marco_legal | plan_trabajo | contexto
candidatura_id: 42
dignidad: vocal_junta_parroquial
nivel_gobierno: parroquial_rural
jurisdiccion_dpa: "0207"
organizacion: Partido Y
lista_numero: "18"
periodo: "2027-2031"
fuente_url: https://...
pdf_sha256: 9f2c...
convertido_con: docling-2.14
revisado_por: inti.poaquiza
revisado_en: 2026-08-14
vigente: true
---
```

Validado en CI. PR que no cumpla el esquema se rechaza.

### Validaciones automáticas del corpus

- Frontmatter completo y tipado
- Secuencia de artículos sin saltos (`marco_legal`)
- Ratio caracteres MD / caracteres extraídos del PDF > 0.85
- Encabezados/pies repetidos (misma línea >5 veces)
- Tildes y `ñ` en proporción esperable (detecta corrupción de encoding)
- Ningún chunk sobre el umbral de tokens

Implementadas **dos veces a propósito**, en Python puro ambas, sin importar una de la
otra (los repos no se cruzan): `lodicho-corpus/scripts/validar_frontmatter.py` (corre
en la GitHub Action, Camino A) y `api/app/services/corpus_validation.py` (corre en el
panel de admin, Camino B). Si cambia una regla, cambiar en los dos lugares.

**Conversión:** Docling, corriendo del lado del servidor dentro del panel de admin
(`api/app/services/pdf_conversion.py`). Reservar LLM solo para escaneados — no
implementado todavía. Si se usa LLM, convertir **página por página** con validación de
conteo de caracteres — un LLM parafrasea artículos sin avisar, y eso es catastrófico y
silencioso.

---

## Panel de admin

Subir un PDF, convertirlo, revisarlo y publicarlo al corpus — todo desde
`https://lodicho.intiinside.com/#/admin`, sin que quien revisa toque una terminal ni
git. No enlazado desde la navegación pública a propósito (herramienta interna).

**Auth:** una sola clave compartida (`ADMIN_PASSWORD`), sesión con token HMAC firmado
(`ADMIN_SESSION_SECRET`), sin tabla de usuarios. Alcanza mientras sea un puñado de
revisores de confianza; si eso cambia, pasar a un usuario por persona (ver
`api/app/services/admin_auth.py`).

**Flujo:** subir PDF → `POST /api/v1/admin/documentos/convertir` (Docling, corre en un
thread aparte vía `asyncio.to_thread` para no bloquear el único event loop del
proceso mientras dura la conversión) → revisar/editar el Markdown y completar el
frontmatter en pantalla → `.../borradores/{id}/validar` (mismas reglas que el Camino A)
→ `.../borradores/{id}/confirmar` (escribe el `.md` + PDF al corpus, commit + push
automático) → botón "Ingestar" (`.../documentos/{doc_id}/ingestar`, mismo código que
`make ingest`). Sección "Documentos" del panel: `GET /api/v1/admin/documentos` lista lo
que ya está en Postgres, con botón de reingestar por fila.

Los borradores viven **en memoria del proceso** (no en Postgres) — se pierden si el
contenedor `api` reinicia a mitad de una revisión, y no funciona corriendo más de un
worker de uvicorn. Aceptable al volumen de este piloto.

### Deploy key SSH (commit automático a lodicho-corpus)

El backend pushea con una llave SSH **dedicada** — nunca la personal de nadie: si el
contenedor se compromete, el máximo daño es escritura sobre `lodicho-corpus`, no sobre
toda la cuenta de GitHub. Setup completo en `secrets/README.md`. Pushea directo a
`CORPUS_GIT_REMOTE` (una URL, no depende de cómo esté configurado el remote `origin`
del checkout).

### Gotchas operativas ya encontradas (para no repetirlas)

- **Docker crea una carpeta si el archivo bind-mounteado no existe.** Si
  `secrets/corpus_deploy_key` no existe en el host *antes* de `docker compose up`,
  Docker monta una carpeta vacía en su lugar en vez de fallar — el error que sale
  después (`Permission denied (publickey)` o `bad permissions`) no menciona esto para
  nada. Comprobar con `ls -la` (una carpeta empieza con `d`, un archivo con `-`).
- **SSH rechaza una clave con permisos abiertos.** `chmod 600` es obligatorio en el
  archivo de la llave privada. Un bind mount preserva los permisos del host tal cual.
- **nginx no re-resuelve la IP del contenedor `api` solo.** Cada vez que se reconstruye
  o recrea `api` (`docker compose build api` + `up -d`), su IP interna cambia; nginx la
  cachea al arrancar y no la actualiza sola. Si `nginx` lleva más tiempo arriba que
  `api`, esperar un 502 hasta correr `docker compose restart nginx`.
- **ARQ no arranca con `functions: []`.** El worker queda en loop de reinicio
  (`RuntimeError: at least one function or cron_job must be registered`) hasta que haya
  al menos una función registrada en `app/worker.py` — hoy es un placeholder (`ping`)
  porque no hay ningún job real todavía.
- **Docling necesita libs gráficas que Debian slim no trae.** Usa OpenCV para el
  análisis de layout; sin `libgl1 libglib2.0-0 libsm6 libxext6 libxrender1 libxcb1
  libgomp1` falla con `libxcb.so.1: cannot open shared object file` (headless, sin
  display real, pero las libs igual hacen falta). Ya están en `api/Dockerfile`.
- **Nunca llamar a Docling de forma síncrona dentro de una ruta `async`.** Con un solo
  worker de uvicorn, bloquea el único event loop del proceso entero mientras dura la
  conversión — nadie más se puede atender mientras tanto, ni `/health`. Siempre
  `await asyncio.to_thread(...)`.
- **Timeout de nginx en `/api/` es el default (60s).** Insuficiente para una conversión
  Docling real, sobre todo la primera vez que carga los modelos de layout.
  `/api/v1/admin/` tiene su propio `proxy_read_timeout 300s` en
  `nginx/conf.d/lodicho.conf`.

---

## Pipeline de consulta

```
POST /api/v1/consulta  (SSE)
  │
  1. Normalizar input:
     - audio  → Gemini 2.5 Flash → texto + idioma + confianza
     - url    → extraer + archivar + separar cita_directa / parafrasis
     - texto  → directo
  │
  2. Clasificar intención (Gemini Flash, enum)
     → si es recomendación de voto / opinión: mensaje fijo de rechazo, FIN
  │
  3. Resolver candidatura
     - extraer nombre de la declaración
     - buscar en BD → si hay ambigüedad, devolver opciones para confirmar
     - fallback: selectores en cascada provincia → cantón → parroquia → dignidad
  │
  4. Caché semántico: buscar en analisis_publicados (umbral ~0.88)
     → hit: devolver informe verificado, instantáneo, FIN
  │
  5. Recuperación dirigida (una por paso, no un prompt gigante):
     - planes_trabajo  filtrado por candidatura_id
     - marco_legal     híbrido, filtrado por nivel_gobierno
     - indicadores     tool call SQL, solo si hay cifras
  │
  6. Respuesta de evidencia → se entrega de inmediato, sin revisión
  │
  7. Si el usuario pide veredicto: encolar en ARQ → estado='borrador'
     → banner "verificación preliminar, pendiente de revisión editorial"
```

### Salvaguardas del veredicto

- **Auto-consistencia:** 3 corridas a temperatura 0.3. Coinciden → confianza alta,
  revisión ligera. Divergen → caso ambiguo, cola prioritaria.
- **Verificador de anclaje:** segunda llamada barata a Flash — ¿cada afirmación del
  informe está sustentada en un chunk citado? Detecta razonamiento no anclado.

---

## Rúbrica de factibilidad

El score **nunca lo genera el LLM**. El modelo llena factores discretos; Python calcula
el número con pesos fijos. Reproducible, auditable, explicable.

| Factor | Valores | Peso |
|---|---|---|
| Competencia legal | exclusiva / concurrente / sin_competencia | 35 % |
| Consta en plan | explicito / implicito / no_consta | 20 % |
| Financiamiento identificado | con_monto / mencionado / ausente | 20 % |
| Plazo vs. período de gestión | holgado / ajustado / imposible | 15 % |
| Precedente presupuestario | existe / parcial / ninguno | 10 % |

El dashboard muestra el desglose, no el número solo. La rúbrica se publica en el sitio.

**No existe "veracidad en porcentaje".** La veracidad es categórica.

### Matiz competencial

Distinguir siempre "ejecutaré X" (requiere competencia) de "gestionaré X ante quien la
tiene" (requiere solo capacidad de gestión). Un candidato a Junta Parroquial puede
legítimamente prometer gestionar obra de otro nivel. Marcar eso como extralimitación es
el error más dañino del sistema.

---

## Salida del modelo

**JSON, nunca Markdown.** El modelo devuelve `InformeContrastacion` validado con
Pydantic v2; el frontend renderiza el Markdown a partir del JSON. Motivos: veredicto
como enum consultable, agregados, y validación antes de persistir.

Usar `response_schema` de Gemini + revalidación Pydantic. Los tres validadores
semánticos (`sin_plan_no_es_ausencia`, `veredicto_factico_exige_indicadores`,
`competencia_exige_articulos`) no se pueden expresar en JSON Schema y son la
salvaguarda principal. Si un validador falla: un reintento; si vuelve a fallar,
persistir con `estado='en_revision'` y nota de error.

Todo campo de texto en español, registro periodístico neutro.

---

## Frontend

Vanilla JS + ES modules, PWA instalable (manifest + service worker). Sin framework,
sin build step. Estructura `web/js/{api.js, admin-api.js, views/, components/}`.

**Instalado y funcionando:** shell de la app (header, nav inferior, tema
claro/oscuro), vistas Inicio / Historial / Acerca de, y el panel de admin (`#/admin`,
ver sección propia). Enviar una consulta de **texto** ya funciona de punta a punta
contra `/api/v1/consulta`. Lo que **no** funciona todavía: el composer ya graba/adjunta
audio y acepta URL, pero el backend rechaza ambos sin procesarlos (ver "Estado
actual"), y no hay pantalla de revisión/publicación de veredictos.

**Composer único** (no tabs separados para texto/voz/URL): una sola caja de texto que
crece, con botones para dictar, subir audio pregrabado, y pegar URL desde el
portapapeles. El modo (`texto` / `voz` / `url`) se infiere al enviar, no lo elige el
usuario de antemano.

**Entrada por voz (dictado, no streaming):**
- Web Speech API para vista previa en vivo mientras habla (gratis, en dispositivo)
- `MediaRecorder` graba en paralelo; al soltar sube el blob
- Gemini produce la transcripción autoritativa
- El texto cae en **el mismo campo** donde habría escrito → un solo pipeline
- Si la confianza es baja: mostrar editable con aviso "revisa si esto es lo que dijiste"
- Guardar siempre el audio original (evidencia ante impugnación)
- `getUserMedia` requiere HTTPS — en dev usar `localhost`, no IP de LAN

**Panel de evidencias:** clic en cualquier afirmación del informe muestra el chunk
recuperado con score, `doc_id` y `git_sha`. Esa trazabilidad es el producto.

**CORS:** `ALLOWED_ORIGINS` en `.env`, para poder correr el frontend en `localhost`
(necesario para probar el micrófono en dev — `localhost` es contexto seguro sin
HTTPS) apuntando al backend de producción.

---

## Convenciones

- Nombres de dominio en español (`candidaturas`, `veredicto`, `evidencias`); código
  Python en inglés donde sea idiomático.
- `jurisdiccion_dpa`: código DPA del INEC como string, no nombre libre.
- Migraciones con Alembic, siempre reversibles.
- Ningún secreto en el repo. `.env` fuera de git; `.env.example` con claves vacías.
  Lo mismo para `secrets/` (deploy key SSH): solo `.gitkeep` y `README.md` van al repo.
- Qdrant y Postgres sin puertos publicados al host — solo red interna de Docker.
- Rate limit agresivo por IP en endpoints públicos: la generación cuesta dinero.
- Límite de 10 MB en subida de audio, validado antes de llamar a Gemini.
- Targets de `Makefile`: `up` / `down` / `logs` / `migrate` / `shell` / `init-qdrant` /
  `init-letsencrypt` / `reload-nginx` / `ingest`. `make ingest ARGS=ruta.md` para un
  solo archivo, sin argumentos para todo el corpus.

---

## Estado actual

**Fase 2 — Pipeline de consulta y veredicto, sin contenido real todavía.**
Infraestructura, ingesta y el pipeline de consulta/veredicto (Entregas 1 y 2, ver
`routers/consulta.py` y `routers/veredicto.py`) ya tienen código e implementan los
pasos centrales de CLAUDE.md; lo que falta es sobre todo entradas no-texto,
cache/indicadores (Entrega 3) y el corpus real. Este párrafo se desactualiza rápido —
confiar en el código y en `docker compose ps` antes que en este párrafo.

**Funciona hoy:**
- Infra completa en el VPS: nginx + Certbot, `api`, `worker`, `postgres`, `redis`,
  `qdrant`, todo en Docker Compose.
- Esquema de base de datos completo, migración inicial aplicada.
- Las 4 colecciones de Qdrant creadas con sus alias — **vacías** (sin contenido real).
- `lodicho-corpus` vivo en GitHub (privado), con el validador de frontmatter y la
  GitHub Action corriendo en cada PR.
- Panel de admin (`#/admin`): login, subir PDF, convertir con Docling, revisar,
  commit + push automático al corpus, ingestar a Qdrant, **y alta de candidaturas**
  (`POST /api/v1/admin/candidaturas`, pantalla "Registrar Nueva Candidatura" — ya no
  hace falta SQL a mano). Sección "Documentos" lista lo ingestado. Sin un caso real de
  punta a punta confirmado en producción todavía (ver gotchas operativas).
- `app/services/ingest.py` (chunk → embeddings → Qdrant → Postgres), compartido entre
  `make ingest` y el panel — probado con integración, sin contenido real en producción.
- `GET /api/v1/estado` (bandera de silencio electoral) — implementado.
- `POST /api/v1/consulta` (SSE) — **implementado, pero solo entrada de texto.**
  Clasifica intención (rechazo fijo para recomendación de voto / opinión / comparación
  de calidad), resuelve candidatura (extracción de nombre vía Gemini + búsqueda +
  desambiguación, con fallback por dignidad), recupera evidencia dirigida
  (`planes_trabajo` filtrado por `candidatura_id`, `marco_legal` filtrado por
  `nivel_gobierno`), persiste `Consulta`/`Declaracion`, y devuelve el evento
  `evidencia` de inmediato. Rechaza explícitamente `voz` y `url` con
  "Esta version solo admite entrada de texto", y `comparacion_factual` con "no
  disponible todavía" — el frontend (composer con dictado/adjuntar audio/URL) ya está
  armado esperando que el backend soporte esto.
- `POST /api/v1/veredicto` (SSE) — **implementado.** Encola `generar_veredicto` en ARQ;
  el worker corre las tres salvaguardas: auto-consistencia (3 corridas a temperatura
  0.3), los dos validadores semánticos expresables en código
  (`sin_plan_no_es_ausencia`, `competencia_exige_articulos`), y el verificador de
  anclaje (segunda llamada a Flash). Calcula factibilidad con pesos fijos en Python
  (`services/factibilidad.py`). `estado` sale en `borrador` o `en_revision` según si
  alguna salvaguarda disparó; nunca en `publicado` (ver más abajo).
- `app/worker.py` con el job real `generar_veredicto` registrado (ya no solo `ping`
  de placeholder).
- Suite de tests cubriendo intención, resolución de candidatura, evidencia,
  factibilidad, generación de veredicto, validadores, verificador de anclaje, y los
  routers de consulta/veredicto (`api/tests/`).
- Frontend: shell instalable, vistas Inicio / Historial / Acerca de, composer único,
  panel de admin. Enviar una consulta de texto real ya funciona de punta a punta.

**No existe todavía:**
- **Entrada por voz y por URL en `/api/v1/consulta`.** El composer ya graba/adjunta
  audio y acepta URL, pero el backend las rechaza sin procesar — falta transcripción
  Gemini, extracción/archivado de URL, y separación `cita_directa` / `parafrasis_periodistica`.
- **Comparación factual** (contraste lado a lado entre candidatos) — la intención se
  clasifica pero el endpoint devuelve "no disponible todavía".
- **Caché semántico** sobre `analisis_publicados` (paso 4 del pipeline) — cada consulta
  repite el retrieval completo; no hay chequeo previo contra veredictos ya publicados
  ni función de búsqueda para esa colección en `qdrant_client.py`.
- **Tool call de indicadores** (cifras oficiales, regla crítica 3) — la tabla
  `indicadores` existe pero está vacía y sin lookup real. Hoy se resuelve solo a
  medias: si el modelo se autoreporta `requiere_indicador=True`, una corrección
  determinista en Python fuerza `incomprobable` (`aplicar_correccion_requiere_indicador`)
  — correcto en el resultado, pero no hay tool call que consulte la tabla y le dé al
  modelo la cifra real cuando sí existe.
- **Fallback en cascada completo** (provincia → cantón → parroquia → dignidad) — solo
  existe el nivel "dignidad" (`fallback_<dignidad>`); los selectores geográficos en
  cascada no están.
- **Flujo de revisión editorial / publicación** (regla crítica 4: "veredicto sin firma
  de revisor nunca sale con `estado='publicado'`) — no hay endpoint ni pantalla para
  que un periodista abra un análisis en `borrador`/`en_revision`, lo firme
  (`revisor_id`, `revisado_en`), agregue `respuesta_candidato`, y lo pase a
  `publicado`. El frontend solo muestra el badge de estado (`informe-card.js`), no
  permite actuar sobre él. Sin esto, ningún veredicto puede llegar a publicarse.
- Colección `contexto`: se puede ingestar y hay `search_contexto()`, pero nada del
  pipeline de consulta/veredicto la llama todavía.
- El Camino A de ingesta completo (n8n + webhook + `POST /api/v1/ingest`) — settings
  (`github_webhook_secret`, `ingest_api_token`) ya están provisionados pero sin
  endpoint ni consumidor; fusionar un PR en `lodicho-corpus` no reindexa nada.
- Detección de `.md` eliminado del corpus (status `removed` en la tabla de estados de
  archivo) — sigue sin un flujo que lo detecte.
- Contenido real en el corpus: ni el COOTAD ni ningún plan de trabajo están
  ingestados; `lodicho-corpus` no está ni clonado localmente en este entorno.
- Set de evaluación de 25–30 afirmaciones anotadas a mano — no existe.

**Siguiente paso lógico:** cargar el COOTAD real y al menos un plan de trabajo vía el
panel de admin como primer caso de punta a punta con contenido real, y en paralelo
cerrar la revisión editorial (sin eso ningún veredicto puede publicarse legalmente) y
el tool call de indicadores (regla crítica 3).

---

## Set de evaluación — casos límite obligatorios

Antes de abrir al público, verificar que el sistema acierta en:

| Caso | Qué verifica |
|---|---|
| Propuesta claramente fuera de competencia | Paso 2 básico |
| **Propuesta de gestión ante otro nivel** | Que NO se marque `fuera_de_competencia` |
| Cifra correcta | Paso 3 positivo |
| Cifra correcta pero descontextualizada | Distinción `enganoso` vs `falso` |
| **Cifra sin indicador disponible** | Que devuelva `incomprobable`, no invente |
| Plan no recuperado | Que devuelva `sin_plan_recuperado` |
| Candidato cita un artículo inexistente | Que no lo valide por complacencia |
| Afirmación ambigua | Que baje la confianza |

Si falla en las dos filas marcadas, no está listo para publicar.

---

## Consideración legal

El sistema emite juicios públicos con nombre y apellido sobre candidatos en período
electoral. Bajo el Código de la Democracia y la normativa del CNE, eso conlleva
responsabilidad.

1. Ningún veredicto se publica sin firma humana. El sistema propone; un periodista firma.
2. `respuesta_candidato` existe desde la primera migración. El derecho a réplica no se
   retrofitea.
3. Cada afirmación del informe enlaza a evidencia concreta con el `git_sha` de la
   versión exacta del documento fuente.

---
> Source: [intiinside/lodicho](https://github.com/intiinside/lodicho) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
