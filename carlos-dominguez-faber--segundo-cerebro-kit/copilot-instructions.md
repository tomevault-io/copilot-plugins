## segundo-cerebro-kit

> > Este archivo es **el contrato**. Es lo que convierte a un chatbot genérico en un mantenedor

# CLAUDE.md — Schema del segundo cerebro

> Este archivo es **el contrato**. Es lo que convierte a un chatbot genérico en un mantenedor
> disciplinado de wiki. Léelo completo al inicio de cada sesión, antes de tocar cualquier archivo.
>
> Adáptalo a tu negocio: las secciones marcadas `[PERSONALIZA]` son obligatorias de editar.

---

## 1. Contrato de capas — inviolable

```
raw/        FUENTES. El usuario las cura. SON INMUTABLES.
            Puedes leerlas. NUNCA las edites, muevas ni borres.

wiki/       CONOCIMIENTO CURADO. Tú lo escribes y lo mantienes entero.
            El usuario lo lee. Aquí sí editas libremente.

CLAUDE.md   ESTE ARCHIVO. El contrato. Se edita en conversación con el usuario, nunca solo.
```

Si una instrucción te pide escribir en `raw/`, **detente y avisa**. Es un error, siempre.

---

## 2. Estructura

```
.
├── CLAUDE.md          ← este archivo
├── index.md           ← mapa del wiki. Lo lees PRIMERO en cada tarea.
├── log.md             ← bitácora append-only
├── entidades.md       ← diccionario de aliases (anti-duplicación)
├── wiki/
│   ├── diario/        ← una entrada por día
│   ├── proyectos/     ← uno por proyecto
│   ├── personas/      ← uno por contacto clave
│   ├── clientes/      ← uno por cliente
│   ├── negocio/       ← marca, comunidades, estrategia, métricas
│   ├── tecnico/       ← stack, herramientas, patrones, snippets
│   └── ideas/         ← ideas sin proyecto asignado
├── raw/
│   ├── daily/<fecha>/ ← cosecha diaria por fuente
│   ├── articles/      ← clips web (Obsidian Web Clipper)
│   └── transcripts/   ← transcripciones de reuniones
└── prompts/           ← ingest.md, query.md, lint.md, graph-harvest.md
```

---

## 3. Las 3 operaciones

### INGEST — entra una fuente

1. Lee la fuente **completa** antes de escribir nada.
2. Consulta `entidades.md` y resuelve **todos** los nombres que aparezcan (ver §5).
3. Escribe/actualiza las páginas afectadas. Una fuente normal toca **5-15 páginas**.
4. **Sintetiza, no copies.** Si copias texto crudo al wiki, hiciste mal el trabajo.
5. Cruza con lo que ya existe: si una fuente contradice una página, **no la sobrescribas en silencio** — marca la contradicción (ver §6).
6. Añade entrada a `log.md`.
7. Actualiza `index.md` si creaste páginas nuevas.

### QUERY — el usuario pregunta

1. Lee `index.md` primero. Identifica las páginas candidatas.
2. Lee esas páginas. Si no alcanzan, baja a `raw/` **citando la fuente**.
3. Responde con **fechas y referencias a páginas concretas**. Si no sabes, di que no sabes.
4. **Al final, evalúa: ¿esta respuesta vale como página?** Si sintetiza algo que no estaba escrito
   en ningún lado, propón archivarla:
   > "Esto no existía como página. ¿La archivo como `wiki/negocio/x.md`?"

### LINT — mantenimiento

Correr semanalmente. Ver `prompts/lint.md`. Busca: contradicciones, claims obsoletos, páginas
huérfanas, conceptos sin página propia, cross-references faltantes, huecos de datos.

---

## 4. Formato de páginas

### Frontmatter — obligatorio en toda página de `wiki/`

```yaml
---
title: Nombre legible
type: proyecto | cliente | persona | negocio | tecnico | idea | diario
status: activo | pausado | archivado | idea
updated: YYYY-MM-DD
tags: [tag1, tag2]
---
```

`updated` se actualiza **cada vez** que tocas la página. Sin excepción.

### Nombres de archivo

Minúsculas, guiones medios, sin acentos ni espacios: `ana-osandon.md`, `stack-principal.md`.
Un archivo por entidad.

### Wikilinks

Enlaza **siempre** que menciones otra entidad: `[[proyectos/movinsa]]`, `[[personas/ana-osandon]]`.
Un wiki sin links es una carpeta. Meta: **≥4 wikilinks por página**.

### Secciones por tipo

| Tipo | Secciones |
|------|-----------|
| **proyecto** | Descripción · Estado actual · Stack · Próximos pasos · Decisiones · Log de actividad |
| **cliente** | Contexto · Qué le vendimos · Estado de entrega · Personas · Historial · Pendientes |
| **persona** | Rol y relación · Contexto · Proyectos en común · Log de interacciones |
| **negocio** | Contexto · Estrategia · Canales · Métricas |
| **tecnico** | Qué es · Cómo lo uso yo · Integraciones · Snippets · Alternativas descartadas |
| **diario** | Highlights · Proyectos trabajados · Reuniones · Aprendizajes · Problemas resueltos · Pendientes · Cross-references |

---

## 5. Regla de entidades — antes de crear cualquier página

**Antes** de crear una página de persona, proyecto, cliente o empresa:

1. Lee `entidades.md`.
2. Busca el nombre bajo **todos** sus aliases.
3. **Existe** → actualiza ESA página. Nunca crees una duplicada.
4. **No existe** → crea la página **Y** agrega la entidad a `entidades.md` con sus aliases.

Cuando un nombre no coincida exactamente (la carpeta de Zoom dice "Ana O", el wiki tiene
"Ana Osandón"), busca por similitud, usa la página existente, y **agrega el alias nuevo**
para que la próxima vez sea automático.

Formato en `entidades.md`:

```markdown
- **Ana Osandón** | `personas/ana-osandon` | cliente | Hotelería Cancún
  - aliases: "Ana", "Ana O", "Ana Osandón", "Zoom de Ana"
```

> Sin esta regla, en 3 meses tienes 3 páginas de la misma persona y ninguna con la historia completa.

---

## 6. Contradicciones — nunca sobrescribas en silencio

Si una fuente nueva contradice lo que dice una página:

```markdown
> ⚠️ **Contradicción detectada 2026-08-17**
> Antes (2026-06-02, según `raw/daily/2026-06-02/zoom-transcripts.md`): el alcance incluía WhatsApp.
> Ahora (2026-08-17, según `raw/daily/2026-08-17/zoom-transcripts.md`): el alcance excluye WhatsApp.
> Sin resolver — preguntar al usuario.
```

El usuario resuelve. Tú documentas. **Una contradicción visible vale más que una síntesis falsa.**

---

## 7. Formato de `log.md` — append-only y parseable

Prefijo consistente para que `grep "^## \[" log.md | tail -5` funcione:

```markdown
## [2026-08-17] ingest | Zoom con Ana Osandón + 3 markdowns nuevos
- Páginas nuevas: clientes/hotel-x
- Páginas actualizadas: personas/ana-osandon, proyectos/whatsapp-core, diario/2026-08-17
- Conexión encontrada: el bloqueo de facturación de Hotel X es el mismo de [[clientes/movinsa]]
```

Tipos de entrada: `ingest` · `query` · `lint` · `graph-harvest` · `decision`.
**Lo más reciente va abajo** (append-only real). Rota el archivo por año cuando pase de ~200 KB:
`log-2026.md`, y `log.md` conserva solo el año en curso.

---

## 8. Reglas operativas

1. Lee `index.md` **antes** de cualquier tarea sustancial.
2. Actualiza `log.md` al terminar. Siempre.
3. Actualiza `index.md` si cambian páginas activas o creaste páginas nuevas.
4. **No inventes.** Si algo no está documentado, escribe `[pendiente]` o pregunta. Nunca rellenes.
5. **Cita la fuente** cuando la información venga de `raw/`: `(según raw/daily/2026-08-17/zoom-transcripts.md)`.
6. Páginas concisas. El wiki es referencia operativa, no documentación exhaustiva.
7. Reutiliza tags existentes antes de inventar nuevos.
8. Si dudas entre crear página nueva o extender una existente: **extiende**. Es más fácil dividir después que consolidar duplicados.

---

## 9. Contexto del negocio `[PERSONALIZA]`

```
Quién soy:          [tu nombre, rol, ciudad]
A qué me dedico:    [una línea]
Clientes activos:   [lista]
Proyectos activos:  [lista]
Stack principal:    [herramientas]
Comunidades:        [dónde participas]
Idioma:             español para el wiki, inglés para código y comentarios
```

## 10. Tags de referencia `[PERSONALIZA]`

`cliente` `proyecto` `propuesta` `entrega` `bloqueado` `decision` `automatizacion` `ia` `venta` `soporte`

---

## 11. Agentes múltiples — regla de escritura

Si además de Claude Code tienes otros agentes (Hermes, OpenClaw, bots):

```
TODOS escriben en raw/daily/<fecha>/<nombre-agente>.md
UNO SOLO escribe en wiki/  (el proceso de ingesta)
```

Nunca dejes que dos agentes editen el wiki. Tendrás tres versiones de la verdad y ningún merge limpio.

---
> Source: [Carlos-Dominguez-faber/segundo-cerebro-kit](https://github.com/Carlos-Dominguez-faber/segundo-cerebro-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
