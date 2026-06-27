## traductorpdf

> Contexto completo del proyecto para continuar trabajo sin leer el historial completo.

# AGENTS.md — Guía para agentes de IA

Contexto completo del proyecto para continuar trabajo sin leer el historial completo.

---

## Qué hace este proyecto

Traductor de PDF 100% local y gratuito. El usuario elige un PDF, selecciona los idiomas
y obtiene un PDF nuevo con el texto traducido pero el layout, imágenes y colores intactos.
Sin APIs externas, sin internet (después de la descarga inicial del modelo), sin cuentas.

---

## Stack técnico

| Módulo | Librería | Por qué esta elección |
|--------|----------|-----------------------|
| Extracción PDF | `pymupdf` (fitz) | Accede a bounding boxes, spans y flags de fuente |
| Traducción | `ctranslate2` | 3-5× más rápido que transformers en CPU, int8 |
| Tokenización | `sentencepiece` | Tokenizador nativo de los modelos MarianMT |
| Descarga modelos | `huggingface_hub` | Solo en primera ejecución |
| GUI | `tkinter` / `ttk` | Incluido con Python, sin dependencias extra |

**Instalación:**
```
pip install pymupdf ctranslate2 sentencepiece huggingface_hub
```

---

## Archivos y responsabilidades

```
Traductor/
├── main.py       # Único: instancia Tk y TranslatorApp, lanza mainloop
├── gui.py        # TranslatorApp: widgets, threading, queue de mensajes UI↔worker
├── extractor.py  # Abre PDF, extrae bloques con bbox/spans, de-hyphenation
├── translator.py # Carga modelos CT2, split de oraciones, batch translate, anti-loops
├── glossary.py   # Glosario EN→ES de títulos exactos (bypassea el modelo)
├── builder.py    # Redacta texto original, reinserta traducción con tipografía original
├── eval.py       # Harness A/B: run/diff/scan sobre el PDF de referencia
├── _test_dehyphen.py # Tests de de-hyphenation (requiere el PDF de referencia)
└── _test_compact.py  # Tests de _compact_heading y _has_dup_prefixes
```

**Regla de dependencias:** `gui.py` importa los otros tres. Los módulos `extractor`,
`translator` y `builder` son independientes entre sí y no importan `gui`.

---

## Flujo completo (trazado de datos)

```
PDF en disco
    │
    ▼ extractor.extract_blocks(pdf_path)
list[list[dict]]   ← páginas → bloques {bbox, text, spans, page_index}
    │
    │ aplanar: flat_texts = [b["text"] for page in all_pages for b in page]
    ▼
translator.translate_batch(flat_texts, src_sp, tgt_sp, ct2, progress_cb)
    │
    │  internamente:
    │  1. filtrar triviales (_needs_translation)
    │  2. _split_sentences  → oraciones individuales
    │  3. _normalize        → ASCII equiv. de Unicode fancy
    │  4. src_sp.encode     → tokens SentencePiece
    │  5. sort by len       → batches homogéneos
    │  6. ct2.translate_batch (beam=2, ngram=3, rep_pen=1.5, max_tgt escalonado)
    │  7. tgt_sp.decode     → texto
    │  8. _truncate_output  → cortar continuación espuria
    │  9. rejoin sentences  → " ".join por bloque original
    ▼
list[str]  (misma longitud que flat_texts)
    │
    │ reagrupar por página
    ▼
builder.build_translated_pdf(pdf_path, all_pages, translated_pages, output_path)
    │
    │  por página:
    │  1. add_redact_annot + apply_redactions(PDF_REDACT_IMAGE_NONE)
    │  2. _select_font   → fuente PDF estándar según flags/nombre
    │  3. _detect_align  → LEFT / CENTER / JUSTIFY
    │  4. _insert_fitting → retry geométrico (−20%/intento, máx 8)
    ▼
PDF guardado en disco
```

---

## translator.py — Decisiones de diseño críticas

### Por qué sentence-level y no block-level

Los modelos OPUS-MT fueron entrenados con pares de oraciones (corpus paralelos como
OPUS). Un bloque de párrafo de 80-150 tokens supera el rango de entrenamiento y provoca
que el decoder genere loops ("del del del", "diseño diseño diseño").

`_split_sentences` usa `_SENT_RE = re.compile(r'(?<=[.!?])\s+(?=[A-Z"\(\[])')` para
dividir antes de traducir y `" ".join(translated_sents)` para reunir después.

### Cap `max_tgt` escalonado — por qué importa

CTranslate2 no para en EOS automáticamente para entradas cortas; llena el budget.
Con el cap constante antiguo (`max_src + 25`), "2nd Edition" (3 tokens) recibía
28 tokens de margen — suficiente para 9 iteraciones de "Edición 2a edición segunda...".

| max_src del batch | Fórmula | Caso típico |
|---|---|---|
| ≤ 5 tokens | `max_src + 4` | Nombres, headings de 1-2 palabras |
| 6–20 tokens | `max_src + 8` | Oración típica |
| > 20 tokens | `min(max_src + 15, 200)` | Oración larga / compleja |

**Nota sobre batching:** las secuencias se ordenan por longitud antes de agrupar
(bucket batching). En un PDF con miles de frases, las frases cortas forman su propio
batch con `max_src` pequeño. En tests de < 32 frases todas caen en un batch y el
comportamiento se degrada ligeramente — es esperado y no un bug.

### `_truncate_output` — por qué existe

El modelo a veces no emite EOS y genera una segunda traducción pegada a la primera.
Como cada INPUT es una sola oración, el primer boundary de oración en el OUTPUT es
siempre espurio. Tres patrones en orden de prioridad:

```python
_CONT_RE       = r'(?<=[.!?])[ \t]*(?=[A-Z])'       # ". Capital" o ".Capital"
_CONT_LOWER_RE = r'(?<=[.!?])[ \t]+(?=[a-záéíóúüñ]{2,})'  # ". minúscula"
_DOT_DOT_RE    = r'(?<=[.!?])[ \t]*\.'               # ". ." — ruido de tokens
```

Se toma el match más temprano y se devuelve `text[:match.start()]` (la posición
del lookbehind es DESPUÉS del `.`, así que no se incluye el carácter siguiente).

### `_UNICODE_MAP` — por qué normalizar antes de tokenizar

Los modelos OPUS-MT fueron entrenados mayoritariamente con ASCII. Las comillas
tipográficas (`"`, `"`, `'`, `'`), em-dashes (`—`) y elipsis (`…`) son tokens raros
o out-of-vocabulary que desestabilizan el decoder y provocan loops. Se normalizan a
sus equivalentes ASCII ANTES de `src_sp.encode`.

---

## builder.py — Decisiones de diseño

### Por qué 2 pasadas en vez de 1

`insert_textbox` es all-or-nothing: si el texto no cabe, retorna negativo y no inserta
NADA (sin mensaje de error). Hay que hacer la redacción en una pasada completa antes de
intentar insertar, para no terminar con texto original y traducido mezclados.

### `_insert_fitting` — retry geométrico

```python
fs = fontsize
for _ in range(8):
    if page.insert_textbox(..., fontsize=fs) >= 0:
        return
    fs = max(5.0, fs * 0.80)
```

Factor 0.80 (−20%) permite hasta 8 intentos antes de llegar a ≈5pt desde cualquier
tamaño inicial razonable (e.g. 12pt → 9.6 → 7.7 → 6.1 → 5.0).

### `_select_font` — mapa a 14 fuentes PDF estándar

Las fuentes estándar no necesitan ser embebidas en el PDF, lo que mantiene el output
portable. La lógica inspecciona `span["flags"]` (bits: 2=italic, 4=serif, 8=mono,
16=bold) y `span["font"]` (nombre en minúsculas) para elegir entre:
helv/hebo/heit/hebi, tiro/tibo/tiit/tibi, cour/cobo/coit/cobi.

---

## gui.py — Regla de threading (CRÍTICA)

**Nunca tocar widgets desde el worker thread.** Solo desde el main thread (el que corre
`mainloop`). El patrón usado:

```python
# Worker thread → solo pone mensajes en la cola
self.queue.put(("status", "Traduciendo..."))
self.queue.put(("progress", 42))

# Main thread → _poll_queue corre cada 100ms vía root.after
def _poll_queue(self):
    while True:
        msg_type, payload = self.queue.get_nowait()
        # actualiza widgets aquí (hilo correcto)
    self.root.after(100, self._poll_queue)
```

Mensajes posibles: `("status", str)`, `("progress", int 0-100)`,
`("done", output_path)`, `("error", str)`.

---

## Modelos disponibles

Los modelos se guardan en `models_ct2/` después de la primera conversión.
Si se borra la carpeta se vuelven a descargar/convertir automáticamente.

| Par | Modelo | Tamaño aprox. |
|-----|--------|---------------|
| EN→ES | `Helsinki-NLP/opus-mt-tc-big-en-es` | ~300 MB |
| ES→EN | `Helsinki-NLP/opus-mt-es-en` | ~75 MB |

Para agregar un nuevo par, añadir entrada a `SUPPORTED_PAIRS` en `translator.py` y
el correspondiente a `LANGUAGES`. La conversión CT2 es automática.

---

## extractor.py — Detección de bloques TOC (fase 3)

PyMuPDF concatena varias líneas en un mismo `block`. En páginas de tabla de
contenidos esto es catastrófico: las subsecciones de un capítulo
("Operational Versus Analytical Systems    3", "Characterizing Transaction…   5", …)
acaban en un único bloque y se traducen como una larga oración con números
intercalados.

`_looks_like_toc_block(lines)` devuelve `True` cuando ≥3 líneas del bloque
hacen match con `_TOC_LINE_RE = r'[\s.]{3,}[\divxlcdm]+\s*$'` (3+ espacios/dots
seguidos de un número de página). En ese caso, cada línea se emite como un
bloque independiente con su propio `bbox`.

**CRÍTICO**: para líneas TOC, el extractor usa `"".join(s["text"] for s in spans)`
(sin strip, sin separador). El comportamiento por defecto (`" ".join(s.strip()…)`)
colapsa los espacios múltiples entre título y número de página, rompiendo el
`_TOC_SUFFIX_RE` del translator (que requiere `\s{8,}`).

---

## translator.py — Tabla de contenidos (fase 3)

### El "." trick

OPUS-MT fue entrenado en oraciones completas. Un título TOC suelto
("Fault Tolerance") es interpretado como inicio de oración y el modelo intenta
"completarlo" generando paráfrasis: `"Fal Cul falla tolerancia a la tolerancia
de falta"`.

**Solución**: en el path TOC, antes de tokenizar, se añade un "." al final del
título (`_normalize(toc_title).rstrip('.,;: ') + '.'`). El modelo lo reconoce
como oración completa y emite EOS naturalmente. El "." trailing en la
traducción se elimina antes de concatenar con `ch_prefix` y `toc_suffix`.

Antes del fix: cap tight (`max_src + 4`) + `length_penalty=0.4` + batch_size=1
no resolvían el problema porque el modelo seguía sin emitir EOS y rellenaba
hasta el cap con basura.

Después del fix: cap relajado (`max_src + 5`), `length_penalty=1.0`,
`batch_size=32`. Las traducciones son limpias.

### TOC pool vs non-TOC pool

`flat_sentences` se separa en `toc_pool` (índices marcados en `toc_flat_indices`)
y `non_toc_pool`. Se procesan en pasadas independientes para que el `max_src`
de cada batch sea representativo de su contenido (TOC titles son cortos,
párrafos del libro son largos).

### `_compact_toc_title`

Postproceso para títulos TOC: si la primera o segunda palabra (≥4 chars) reaparece
después de la posición 3, trunca ahí. Captura casos como
`"Vistas Materializadas y Cubos De Datos Vistas materializadas, datos…"` →
`"Vistas Materializadas y Cubos De Datos"`.

---

## translator.py — Decisiones adicionales (fase 2)

### `_is_proper_name` — detección de nombres propios

Exactamente 2 palabras alfabéticas en Title Case donde el APELLIDO (segunda palabra)
cumple todos los criterios: ≥5 caracteres, no termina en sufijo morfológico inglés
(`-tion`, `-ity`, `-ance`, `-ing`, etc.), y no está en `_COMMON_SECOND_WORDS`
(lista de sustantivos técnicos comunes: "Source", "Services", "Systems", etc.).

También se llama con el texto de entrada tras hacer `.lstrip('—–-&"\'*() ')` para
capturar bloques del estilo `& Chris Riccomini` o `—Martin Kleppmann`.

### `_is_name_list` — listas de agradecimientos

Bloque con ≥5 comas, ≥8 palabras alfabéticas, >70% en Title Case, y ≤3 palabras
funcionales en minúscula de ≥4 chars (las palabras funcionales revelan estructura
de frase, no lista de nombres). Se pasa sin traducir para evitar alucinaciones masivas.

### `_split_toc` — entradas de tabla de contenidos

Detecta `"título . . . . . . N"` o `"título           N"` (puntos o espacios + número).
Devuelve `(title, suffix, should_translate)`. Los títulos de 1 sola palabra
(`should_translate=False`) se devuelven directamente con el sufijo normalizado
(`"Preface . . . . . . . . . . xvii"`) sin pasar por el modelo.

Para títulos multipalabra, el número de capítulo ("1. ") se extrae antes de
traducir y se prepende al resultado usando `_CHAPTER_PREFIX_RE` + `toc_wrappers`.

### `_needs_translation` — numerales romanos

`_ROMAN_RE = r'^(?:[ivxlcdm]+|[IVXLCDM]+)$'` con longitud ≤6 → no se traduce.
Captura los números de página en portadas de capítulo (`ix`, `xvii`, `xxi`).

### `_truncate_output` — mejoras fase 2

1. **`_UNK_GLYPH = '⁇'` (U+2047)**: SentencePiece emite este carácter (NO U+FFFD)
   para tokens desconocidos. Se elimina antes de aplicar los patrones de boundary.

2. **Patrones actualizados**: `_CONT_RE` y compañía ahora incluyen `⁇` en el grupo
   intermedio para que puedan saltar el glyph UNK entre un `.` y la siguiente oración.
   También `¿`/`¡` en los lookaheads para capturar preguntas españolas post-EOS.

3. **`_DUP_WORD_RE`**: collapsa `\b(\w{3,})\s+\1\b` (palabras adyacentes idénticas).

4. **First-word repetition (k≤2)**: detecta `"Edición 2a edición"` (palabra 0 = palabra 2)
   y trunca a `"Edición 2a"`. El límite es k<3 para NO truncar `"Capítulo 1 El Capítulo…"`
   (k=3 sería agresivo y corta resultado legítimo del modelo).

### `_normalize` — normalización de guiones compuestos

`_TITLE_HYPHEN_RE = r'([A-Z][a-z]+)-([A-Za-z])'` → `r'\1 \2'` convierte
`"Data-Intensive"` → `"Data Intensive"`, `"Trade-Offs"` → `"Trade Offs"`.
Solo afecta palabras con mayúscula inicial; `"co-author"` no cambia.

### `_SENT_RE` — no partir números

`(?<![0-9])(?<=[.!?])\s+(?=[A-Z…])` — el lookbehind negativo `(?<![0-9])` impide
que `"1. Trade-Offs in Data Systems Architecture"` se parta en `["1.", "Trade-Offs…"]`,
lo que causaba que el modelo viera `"1."` suelto y generara `"1) a 1 (1) del 1.1 1."`.

### `repetition_penalty` — subido de 1.5 a 2.0

Penalización más fuerte en repetición de tokens, especialmente útil para bloques
cortos donde los patrones de ngram (ngram_size=3) no capturan variantes con acento
(`"Martin"` vs `"Martín"`).

---

## Fase 5 (junio 2026) — harness de evaluación, de-hyphenation, glosario

### eval.py — harness A/B (medir antes de afirmar)

```
python eval.py run --label X [--pages 1-20]   # traduce el PDF de referencia → eval_runs/X.json
python eval.py diff baseline X                # diff por bloque entre dos runs
python eval.py scan X                         # bloques sospechosos (loops, ⁇, ratios)
```

PDF de referencia: `Downloads/Designing Data-Intensive Applications, 2nd Edition-1-55.pdf`.
Los bloques se emparejan entre runs por `(página, bbox redondeado)`. SIEMPRE correr
un run antes y después de cualquier cambio de calidad; el scan tiene en cuenta
los dot-leaders del TOC para no dar falsos positivos.

### extractor.py — de-hyphenation (líneas cortadas por guion)

Este PDF (typesetting O'Reilly) corta palabras a fin de línea con U+2010 "‐"
(NO el ASCII "-"): "appli‐/cations", "Hernan‐/dez". El join antiguo producía
"appli‐ cations" (tokens rotos → mala traducción; 53 bloques afectados en el
PDF de referencia). `_join_lines(line_texts, vocab)` une líneas así:

1. palabra fusionada ∈ vocabulario del doc → quitar guion ("information")
2. guion tipográfico U+2010/U+2011 + continuación minúscula → quitarlo
   (silabación profesional)
3. otro guion → unir sin espacio ("analysis-friendly", "TCP-IP")
4. sin guion → unir con espacio (comportamiento original)

`_collect_vocab(doc)` hace una pasada previa con `get_text("words")`.

### glossary.py — match exacto de títulos

`glossary.lookup(texto)` devuelve la traducción curada cuando TODO el bloque
(o todo el título TOC, sin prefijo de capítulo ni sufijo de página) coincide
con una entrada. Resuelve de forma determinista los títulos que el modelo
destroza ("Fault Tolerance" → "Tolerancia a fallos", "Data Warehousing",
"Defining Nonfunctional Requirements") y los títulos de 1 palabra que antes
quedaban en inglés (Scalability, Maintainability, Summary, Preface…).
Es extensible: añadir clave en minúsculas → valor con capitalización deseada.

### translator.py — _compact_heading (reemplaza _compact_toc_title)

Aplicado a títulos TOC Y a bloques con sentinel period. Dos cortes:

1. **Boundary con eco**: un límite de oración interno cuya continuación repite
   una palabra de contenido de antes (prefijo 5 chars, sin acentos) es un
   restart de paráfrasis → cortar: "¿Quién debería leer este libro?, quién
   deberia leerlo." → "¿Quién debería leer este libro?". Sin eco NO corta
   ("...Inc. 1005 Gravenstein..." es legítimo). GUARD: solo corta si la parte
   conservada es ≥40% del texto — si no, las referencias bibliográficas donde
   el modelo duplica el autor ("Edgar F. [5] Edgar Edgard F. Codd…") quedarían
   reducidas a "Edgar F".
2. **Eco de palabra**: palabra de contenido que reaparece en el título
   ("Columna…columna", "Región…Regiones") → cortar ahí y limpiar funcionales
   colgantes. Solo en outputs ≤10 palabras Y si la fuente no repite palabras
   (`_has_dup_prefixes`, umbral 4 chars para cubrir "data"/"datos") — si no,
   "Shared-Memory, Shared-Disk…" se truncaría mal. GUARD: la repetición debe
   estar en el 40% final del título (idx/len ≥ 0.6) — el español repite
   legítimamente el sustantivo a mitad en "X versus Y" ("Sistemas operativos
   versus sistemas analíticos" NO debe cortarse).

Además, la compactación solo aplica a títulos TOC, footers y bloques sentinel
con fuente ≤12 palabras (`heading_groups`). Los bloques sentinel largos
(referencias bibliográficas) NO se compactan: perderían contenido real.

### translator.py — footers ("Título | N" / "N | Título")

`_split_footer` detecta running headers/footers con `|` y número de página
(arábigo o romano). El título se traduce por la vía TOC (glosario → modelo
con "." sentinel + cap tight) y el lado del número se restaura intacto.
Convierte ~25 bloques por libro de basura a traducción limpia y consistente.

## Fase 6 (junio 2026) — fase 2 de calidad: refs cruzadas, bibliografía, glifos, figuras

### translator.py — _SENT_RE no parte en iniciales (CAUSA RAÍZ bibliografía)

`(?<!\s[A-Z]\.)` añadido al regex. "Edgar F. Codd" se partía en
["Edgar F.", "Codd…"] y el fragmento "Edgar F." suelto hacía que el modelo
duplicara el autor ("Edgar F. [5] Edgar Edgard F. Codd…"). Con el fix las
referencias se traducen como una unidad. `_ABBR_PROTECT_RE` protege además
"e.g.", "i.e.", "vs.", "Fig." con \x00 temporal alrededor del split.

El MISMO guard se aplica a `_CONT_RE` en `_truncate_output`: sin él, el
OUTPUT "Edgar F. Codd, …" se truncaba a "Edgar F." (las iniciales parecen
fin de oración + restart).

### translator.py — passthrough de autores en bibliografía

`_is_author_sentence`: oración donde TODAS las palabras son nombres
Title-Case, iniciales ("F.") o partículas ("van", "de", "and"). Solo se
evalúa dentro de bloques con prefijo "[ N ]" (`ref_flat_indices`) para que
headings de 2 palabras ("Understanding Load") no se confundan. Las oraciones
de autores van passthrough con solo "and"→"y": el modelo loopea en
secuencias de nombres ("Pedro Rui de Pedro Ruy Pedro Ruí…").

### translator.py — _mask_cross_refs (referencias cruzadas deterministas)

`(see "Title" on page N)` se reemplaza por un marcador numérico "(901)" ANTES
de traducir (los dígitos sobreviven OPUS-MT verbatim) y después se restaura
como `(ver "<título ES del glosario, o EN intacto>" en la página N)`.
Fallback si el modelo pierde el marcador: se anexa al final del bloque.

### translator.py — prefijo bibliográfico "[ N ]"

`_REF_PREFIX_RE` lo separa antes de traducir y se re-antepone después
(`ref_prefixes`), igual que el prefijo de capítulo del TOC.

### translator.py — glifos UNK con acento mayúscula

- Lado fuente: `_UNICODE_MAP` ahora ASCII-folds diacríticos NO españoles
  (Ö→O, ä→a, ß→ss…): "Özcan" → "Ozcan" en vez de " zcan".
- Lado output: `_recover_unk_heads` repara "⁇ndices" → "Índices" con la tabla
  `_UNK_HEAD_FIXES` (palabras españolas frecuentes con mayúscula acentuada
  inicial que el vocab del decoder no puede emitir). Corre ANTES de borrar ⁇.

### builder.py — obstáculos visuales (fix traslape p36)

`_compute_max_y` ahora recibe `_visual_obstacles(page)`: bboxes de imágenes y
dibujos vectoriales 2D (>8×8 pt; las reglas horizontales finas se ignoran
para no reintroducir shrink). Las figuras suelen llevar sus etiquetas como
píxeles/vectores — sin esto, un párrafo encima de una figura se extendía
sobre ella (traslape en p36 del PDF de referencia).

## Fase 7 (junio 2026) — fase 3: layout (extensión horizontal, align derecha, TOC dots)

### builder.py — extensión HORIZONTAL de bbox (fix headings encogidos)

Los headings van en bboxes ajustados al texto y con un párrafo justo debajo:
la extensión vertical no ayuda y el texto español más largo se encogía
brutalmente ("Separation of storage and compute" fs 11.6 → 6.4 en p40, con
fuente original condensada que agrava el problema). `_compute_x_limits`
calcula cuánto puede crecer cada rect hacia los lados sin chocar con vecinos
que se solapen verticalmente (límite: bordes de columna = min x0/max x1 de
los bloques de la página). `_insert_fitting` prueba en orden: rect original →
ancho extendido → alto extendido → ambos → shrink. La dirección respeta la
alineación: RIGHT crece a la izquierda, CENTER simétrico, resto a la derecha.

### builder.py — detección de alineación derecha

`_detect_align(..., col_right)`: bloque cuyo x1 ≈ borde derecho de columna y
x0 > centro de página → TEXT_ALIGN_RIGHT ("Preface" fs 25.2 pegado al margen
derecho se partía en "Prefaci o" en dos líneas; ahora crece a la izquierda y
mantiene una línea y su alineación visual).

### builder.py — _format_toc_line (leader de puntos calculado)

El translator emite un leader FIJO de 10 puntos. `_format_toc_line` mide con
`pymupdf.Font.text_length` el título y el número y rellena con los puntos
necesarios para que el número quede cerca del borde derecho del bloque
(±1 punto). Fallback al texto original si no hay espacio o falla la métrica.

## Fase 8 (junio 2026) — prosa: beam 4, colapso de n-gramas, leading uniforme

### translator.py — beam_size 2 → 4

Mejor gramática general (el reclamo en ficción era gramaticalidad). Costo
medido: +43% de tiempo en CPU (Yumi 45pp: 243s → 348s). Trade-off neto
positivo: sospechosos Yumi 26→25, DDIA 41→46 donde los nuevos eran mayormente
falsos positivos del detector de trigramas + 2 títulos TOC que se fijaron con
glosario. Si la velocidad importara más que la calidad, bajar a 2.

### translator.py — _collapse_repeated_ngrams

El modelo reinicia a mitad de oración y repite frases enteras (frecuente en
prosa de ficción). Colapsa secuencias ADYACENTES idénticas de 2-6 palabras
(case-insensitive) iterando hasta el punto fijo. En Yumi limpió 16/16 bloques
afectados. Repeticiones adyacentes legítimas de n≥2 son prácticamente
inexistentes; n=1 lo maneja _DUP_WORD_RE. Corre en _truncate_output ANTES de
añadir sufijos TOC (los dot leaders ". . . ." parecerían n-gramas repetidos).

### builder.py — leading compacto antes de encoger fuente

El reclamo "diferentes tamaños de letra en los párrafos": el shrink por
bloque dejaba una escalera 15→13.8→12.7→11.7→10.7 mezclada en la misma
página (tamaño dominante solo 37% del texto). Ahora _insert_fitting prueba
lineheight = (ascender−descender)×0.95 y ×0.90 al fontsize ORIGINAL antes del
shrink loop (que también usa ×0.90). Leading 10% más compacto es mucho menos
visible que fuente 8-15% más chica. Resultado Yumi: tamaño dominante 62% del
texto y un escalón más grande (12.7pt vs 11.7pt). El 15pt original completo
requeriría reflow de página (el español ocupa ~20% más).

## Fase 9 (junio 2026) — tamaño uniforme por página + pulido de headings

### builder.py — planificación en 2 pasadas con página scratch

Probado con "Frugal Wizard" (novela densa en diálogos: 632 bloques/45pp).
Cada réplica de diálogo es un bloque de 1 línea sin espacio debajo: el
interlineado no salva el caso 1→2 líneas y cada bloque encogía por su cuenta
(original 80.7% a 10.8pt → traducido 36% + escalera 9.9/9.1/8.4).

Solución (lo que haría un maquetista): `_plan_fit` ejecuta el cascade de
ajuste sobre una página SCRATCH (métricas idénticas, cero efectos) y graba
(fs, lineheight, rect) por bloque. Luego, por clase de tamaño original
(≥3 bloques = cuerpo, no headings) se unifica al MÍNIMO planificado de la
clase con piso 0.78×original, re-planificando los bloques que iban más
grandes. Outliers bajo el piso conservan su tamaño menor. Una página toda a
9.9pt se ve maquetada; una mezcla 10.8/9.9/9.1/8.4 no.

### translator.py — _polish_heading

El modelo des-capitaliza líneas Title-Case y mete espacio antes de
puntuación ("primera parte : la habitación"). En el rejoin de grupos
heading/TOC/footer: colapsar `\s+([:;,.!?%])` y capitalizar la primera letra.

## Fase 10 (junio 2026) — fusión de líneas en párrafos (extractor)

### El problema raíz de la ficción: un bloque por línea

Algunos PDFs (ficción: Yumi, Frugal Wizard) emiten UN BLOQUE POR LÍNEA visual
(todos `nlin=1`, gap uniforme ~5.6, párrafos delimitados solo por indent de
primera línea). Traducir línea por línea destrozaba los TRES problemas que
reportó el usuario a la vez:
- **gramática**: oraciones cortadas se traducían como fragmentos
  ("...smoke and" | "sulfur." → dos traducciones inconexas)
- **indentación**: inicios de párrafo indentados quedaban como bloques sueltos
- **tamaño**: cada línea encogía por su cuenta → mosaico

### `_merge_singles` — fusión por página

`extract_blocks` clasifica cada bloque: `toc` (multilínea TOC) / `tocln`
(línea suelta con dot-leader+número) / `multi` (párrafo ya formado) / `single`
(una línea). **Solo fusiona si la página está dominada por líneas sueltas**
(`n_single ≥ 4` y ≥60% del total) — así DDIA (multilínea) queda intacto
(647→647 bloques verificado). Las líneas tipo-TOC nunca se fusionan
(preservan su spacing crudo para el path TOC del translator).

Regla de párrafo: línea indentada (x0 > margen+6) o gap > 1.8×leading inicia
párrafo nuevo; el resto continúa. Párrafo de 1 línea conserva su bbox
original (títulos centrados, líneas aisladas). Párrafo multilínea se alinea al
margen y guarda su indent en puntos (`block["indent"]`).

### builder — reproduce el indent de primera línea

`insert_textbox` no tiene parámetro de sangría, pero SÍ respeta espacios
iniciales (verificado con render). El builder antepone
`round(indent_pts / ancho_espacio)` espacios al texto, solo en alineación
LEFT/JUSTIFY (nunca centrado/derecha). Frugal p25: 26 líneas → 12 párrafos
con oraciones completas e indent de primera línea preservado.

## Fase 11 (junio 2026) — preservar no-prosa + arreglar títulos destrozados

Probado con "AI Engineering" (Chip Huyen, 535pp). El usuario reportó: tablas
corruptas/en blanco, fórmula intercalada, y "Writing" traducido como "w w w".

### extractor.py — dejar tablas/fórmulas/URLs como píxeles originales

La causa de las tablas rotas: el modelo reflujo las celdas en sopa superpuesta
y borraba el encabezado de color; las "en blanco" eran celdas cuya traducción
quedaba vacía. Fix: **no extraer esas regiones** → el builder nunca las redacta
→ los píxeles originales sobreviven.
- `_table_rects(page)` = `page.find_tables()` con ≥2×2; bloques cuyo centro
  cae dentro se omiten (`_in_any`).
- `_is_formula_block(text)`: bloque corto (<160) sin dot-leader, con <35% de
  palabras reales y (>40% tokens de 1 char o indicador math `_MATH_RE`).
  "P ( x 1 , x 2 )" → omitido. OJO: el chequeo va DESPUÉS de detectar `tocln`
  (las entradas TOC con ". . ." son dot-heavy pero NO son math).
- `_is_url_block(text)`: bloque <90 con http/www/dominio → omitido.
- DDIA: 647→643 (los 4 omitidos son solo URLs de O'Reilly; TOC/footers OK).

### builder.py — red de seguridad anti-blanco

`if not text.strip(): text = block["text"]` (original). El área ya fue
redactada en la pasada 1, así que una traducción vacía dejaría una caja en
blanco — restauramos el original en vez de perder contenido.

### translator.py — palabras-título y guardia anti-degeneración

OPUS-MT destroza títulos de 1 palabra (medido):
`Writing→"Ww w"`, `Coding→""`, `Roleplaying→"Rede de"`, `Tutoring→"Tutotutoring"`.
- **Glosario** (glossary.py): ~60 entradas nuevas de palabras-sección frecuentes
  (Writing→Escritura, Coding→Programación, Roleplaying→Juego de roles, gerundios
  de ML, "foundation models"→"modelos fundacionales", etc.). Es la vía limpia.
- **`_is_degenerate(src, out)`**: red de seguridad para palabras fuera del
  glosario. Solo juzga inputs ≤3 palabras. Degenerado si: output vacío, o
  ningún token con ≥3 letras ("Ww w", "w w w"), o 1 palabra que repite el
  prefijo del source ("Tutotutoring"). Al final del rejoin (no-TOC):
  degenerado → `glossary.lookup(orig)` → `orig`. Nunca mostrar basura.

## Bugs conocidos / limitaciones activas

| Problema | Causa | Estado |
|----------|-------|--------|
| Bloque dirección O'Reilly (p.copyright) → "O'Reilly Media, Inc" | `_CONT_RE` corta ante dígito tras punto ("Inc. 141") — trade-off documentado de fase 4 | Preexistente a fase 5; cosmético (página legal) |
| Referencias bibliográficas con autor duplicado ("[5] Edgar F. Codd… [5] Edgar Edgard…") | El modelo repite el nombre al inicio; las referencias son ≥12 palabras y no se compactan (a propósito) | Contenido completo pero con ruido; candidato fase 2: strip de "[ N ]" + passthrough de autores |
| "Designing Data-Intensive Applications" (portada aislada) | "Data Intensive" descompuesto por `_normalize` produce tokens poco frecuentes; batch pequeño de portada da max_tgt grande | Aceptable: visible solo en portada, párrafos del libro se traducen bien |
| Traducción de "Trade-Offs" → "Operaciones" (en TOC) | "Trade Offs" separado por `_normalize` cambia la tokenización | OPUS-MT no tiene un token específico; resultado coherente aunque impreciso |
| PDFs escaneados sin texto | No hay OCR | No soportado (v1) |
| Fuentes CJK/árabe | Las fuentes PDF estándar no cubren esos glifos | Requeriría incrustar TTF |
| Tablas y 2 columnas | El extractor trata cada bloque independientemente | Layout puede desalinearse |

---

## Pendientes priorizados

1. **Botón swap From↔To** — actualmente hay que cambiar los dos combos manualmente
2. **Progreso por página** — mostrar "Página X de Y" además del conteo de bloques
3. **Soporte para más pares** — FR, DE, PT son viables solo con entradas en `SUPPORTED_PAIRS`
4. **Modo preview** — mostrar texto extraído antes de traducir para detectar PDFs escaneados
5. **Heading "CHAPTER N Title"** — el modelo traduce "CHAPTER 1" → "Capítulo 1" correctamente
   pero a veces con continuaciones; una mejora sería separar el número del título como
   se hace con TOC, pero sin causar regresiones en la traducción del título

---

## Cómo correr el proyecto

```bash
cd "C:\Users\joela\OneDrive\Escritorio\Traductor"
python main.py
```

Primera ejecución descarga y convierte el modelo (~5 min, una sola vez).
Ejecuciones siguientes cargan desde `models_ct2/` en ~3-5 segundos.

---
> Source: [Franklin-Amador/TraductorPDF](https://github.com/Franklin-Amador/TraductorPDF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-27 -->
