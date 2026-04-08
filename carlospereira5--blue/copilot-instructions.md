## blue

> Este archivo orienta a Claude Code y cualquier LLM que trabaje en este repositorio.

# CLAUDE.md — Aria Framework

Este archivo orienta a Claude Code y cualquier LLM que trabaje en este repositorio.
Leerlo completo antes de tocar una línea de código.

---

## ¿Qué es Aria?

**Aria es un framework de agente componible — no un agente de negocio.**

El programa en sí es pura orquestación: un mensaje entra por una interfaz `Messenger`,
se construye un prompt desde los módulos registrados, se consulta al LLM, y se ejecutan
las tool calls que devuelve. Eso es todo lo que el framework hace por sí solo.

Lo que Aria *puede hacer* depende completamente de los módulos que se le conecten.
Si le conectás los módulos del kiosco (inventario, contabilidad, personalidad), Aria
es la asistente del kiosco. Si le conectás otros módulos, es otra cosa. El framework
no sabe de negocios — los módulos sí.

> "El programa es un framework, no un agente como tal."

---

## Conceptos Fundamentales

### 1. Módulos — la única forma de extender el sistema

Un módulo es un paquete en `internal/modules/<nombre>/` que implementa una o ambas
de estas interfaces:

```go
// DataReader: provee datos y contexto AL agente.
// Ejemplos: cargar stock, consultar ventas, inyectar personalidad al prompt.
type DataReader interface {
    Name() string
    Init(deps PortDeps) error
    PromptSection(ctx context.Context, userID string) string
    ReadTools() []agentllm.ToolDef
    Read(ctx context.Context, tool string, args map[string]any) (map[string]any, error)
}

// DataWriter: muta estado en el mundo.
// Ejemplos: registrar merma, enviar WhatsApp, guardar deuda, crear tarea.
type DataWriter interface {
    Name() string
    Init(deps PortDeps) error
    WriteTools() []agentllm.ToolDef
    Write(ctx context.Context, tool string, args map[string]any) (map[string]any, error)
}
```

Un módulo puede implementar solo `DataReader` (ej: módulo de personalidad — solo provee
prompt, no muta nada), solo `DataWriter` (ej: módulo de notificaciones), o ambos.

El framework usa type assertions para auto-descubrir qué puede hacer cada módulo:

```go
for _, m := range a.modules {
    if r, ok := m.(DataReader); ok { /* registra read tools */ }
    if w, ok := m.(DataWriter); ok { /* registra write tools */ }
}
```

**Agregar un módulo nuevo = un paquete nuevo + una línea en main.go. Cero cambios al core.**

### 2. Recursos — conexiones al mundo exterior

El framework gestiona recursos externos y los provisiona a los módulos via `PortDeps`.
Los módulos no crean ni poseen recursos — solo los reciben y usan.

```go
type PortDeps struct {
    DB        PortDB           // acceso controlado a SQLite
    Logger    *log.Logger
    LLM       agentllm.LLM    // para módulos que necesiten inferencia (ej: OCR)
    Loyverse  loyverse.Client  // nil si no configurado
    Messenger Messenger        // nil hasta SetMessenger() — circular con WhatsApp
}
```

Recursos actuales: **DB**, **WhatsApp**, **Loyverse**.
Recursos futuros: Calendar, Gmail, cualquier API externa.

### 3. Funciones Puras (Cortex)

La lógica de cómputo que NO toca la DB ni la red vive como funciones puras dentro del
módulo que la usa. Archivos `compute_*.go` dentro del paquete del módulo.
Sin side effects. Testeables sin mocks.

Ejemplos: `compute_search.go` (fuzzy search 5 tiers), `compute_velocity.go` (velocidad
de ventas), `compute_purchase.go` (recomendación de compra).

### 4. El Agente

El agente (`internal/agent/`) orquesta todo pero no conoce el negocio:

- **Sesiones**: historial de conversación por usuario (TTL configurable)
- **Prompt builder**: core + sección de memoria + secciones de cada módulo (`PromptSection`)
- **Tool routing**: cuando el LLM llama una tool, la despacha al módulo correcto
- **Memoria**: cross-sesión, interface `Memory` (hoy SQLite, mañana Engram)
- **Recursos**: los administra y provisiona — no los posee

---

## Estructura de Módulos

Cada módulo vive en `internal/modules/<nombre>/` y es auto-contenido:

```
internal/modules/inventario/
    module.go          → struct, New(), Init(), Name(), Schema()
    reader.go          → implementa DataReader (PromptSection, ReadTools, Read)
    writer.go          → implementa DataWriter (WriteTools, Write)
    handlers_stock.go  → handlers de Read tools
    handlers_waste.go  → handlers de Write tools
    compute_search.go  → funciones puras (sin DB, sin side effects)
    compute_velocity.go
```

La interfaz interna del módulo con la DB es directa via `PortDB` — sin service layer
intermedio a menos que la lógica de dominio lo justifique.

---

## Dos Entry Points

### `cmd/aria` — Bot de WhatsApp

Pool de goroutines escucha mensajes via `Messenger`. Cada mensaje se pasa al agente,
que gestiona la sesión y ejecuta el loop de tool calls.

```
Mensaje WhatsApp → Messenger → agent.Chat() → LLM → tool calls → módulos → respuesta
```

### `cmd/admin` — Admin TUI (Bubble Tea)

Dos modos de uso en una sola app:

1. **Chat** — igual que el bot, pero en terminal. El LLM ejecuta las tools disponibles.
2. **Utilidades de módulo** — apps TUI específicas de cada módulo (ej: escáner de
   inventario, visor de lotes). Aparecen como menú. Usan la DB directamente, sin LLM.

El flujo natural: usás la utilidad de inventario para cargar datos → después en el Chat
le pedís al agente que analice o modifique esos datos.

---

## Reglas de Código (No Negociables)

### Tamaño de archivos
**Máximo 200 líneas por archivo `.go`.** Si crece, partilo. No lo justifiques.
Esta regla existe para que cada archivo sea auditable de un vistazo.

### Comentarios — el "por qué", no el "qué"
Comentar la intención de cada bloque lógico no obvio. El código dice qué hace;
el comentario dice por qué lo hace así y no de otra manera.

```go
// Usamos LIKE '%itemID%' en lugar de JSON_EXTRACT porque el campo items_json
// es un array serializado y SQLite < 3.38 no soporta JSON_EACH en subqueries
// con parámetros de bind. Ver: https://sqlite.org/json1.html
row := db.QueryRow(`SELECT ... WHERE items_json LIKE ?`, "%"+itemID+"%")
```

### Logging
Todo el sistema usa `charmbracelet/log`. **Nunca** `fmt.Printf` ni `log.Printf` para
logging. El logger siempre se inyecta — ningún paquete crea su propio logger.

Niveles:
- `Debug` — flujo interno, valores intermedios (solo en modo debug)
- `Info` — eventos de ciclo de vida (arranque, conexión, sync completado)
- `Warn` — errores recuperables (retry, fallback, dato inesperado)
- `Error` / `Fatal` — errores que afectan la operación

### Integridad Contable
- **DB (SQLite):** dinero guardado como `INTEGER` (monto × 100, en centavos).
- **Código:** `shopspring/decimal` para todos los cálculos monetarios. Nunca `float64`.

### Sin CGO
Compilar con `CGO_ENABLED=0`. Requisito para compatibilidad con Termux (Android).
Usar `modernc.org/sqlite` — driver SQLite puro Go.

### Dependencias
No agregar dependencias externas sin evaluación explícita. Cada dependencia es deuda.

---

## Reglas de Arquitectura (No Negociables)

1. **PROHIBIDO** modificar las interfaces `DataReader` o `DataWriter` del core para
   acomodar un módulo nuevo. Si necesitás algo que no está, extendé el módulo.

2. **PROHIBIDO** que un módulo conozca a otro módulo. Los módulos son independientes.
   La comunicación entre dominios pasa por el agente o por la DB compartida.

3. **PROHIBIDO** lógica de negocio en `internal/agent/`. El agente orquesta, no decide.

4. **PROHIBIDO** acceso directo a `*sql.DB` desde un módulo. Solo via `PortDB`
   (QueryContext, QueryRowContext, ExecContext). El agente controla el acceso.

5. **PROHIBIDO** crear goroutines de fondo en módulos sin implementar `Start(ctx) error`.
   El agente llama `Start()` en los módulos que lo implementen — nunca `go func()` suelto.

6. **PROHIBIDO** parches sobre parches. Si algo está roto, buscá la causa raíz.

---

## Estructura de Paquetes

```
cmd/
  aria/               → Bot WhatsApp: pool goroutines + Messenger
  admin/              → Admin TUI: Chat + Utilidades de módulo

internal/
  agent/              → Core del framework
    aria.go           → struct Aria, New(), Chat(), Start()
    interfaces.go     → DataReader, DataWriter, Module, PortDeps
    memory.go         → interfaz Memory (Reader + Writer)
    messenger.go      → interfaz Messenger agnóstica al canal
    prompt.go         → PromptBuilder (core + memoria + secciones de módulos)
    executor.go       → loop de tool calls
    args.go           → helpers compartidos (StrArg, FloatArg, etc.)
    memstore/
      sqlite.go       → implementación SQLite de Memory

  modules/            → Módulos de dominio (plug-and-play)
    inventario/       → DataReader + DataWriter: stock, lotes, OCR, mermas
    contabilidad/     → DataReader + DataWriter: ventas, deudas, flujo de caja
    tareas/           → DataReader + DataWriter: empleados, tareas, recordatorios WA
    kiosco/           → DataReader only: personalidad + contexto del negocio

  db/                 → Persistencia SQLite
    migrate.go        → Schema base + migraciones
    store.go          → *sql.DB wrapper + pool config

  loyverse/           → Cliente API mínimo (endpoints core)
  whatsapp/           → Integración whatsmeow (bot, handler, messenger)
  sync/               → Sincronización Loyverse → SQLite en background
  webhook/            → Handler HTTP para webhooks de Loyverse
  logger/             → Inicialización charmbracelet/log
  config/             → Variables de entorno
```

---

## Comportamiento WhatsApp

### Modos de operación

| `WHATSAPP_GROUP_JID` | Modo | Escucha |
|---|---|---|
| No configurado | **DM** | Solo mensajes directos |
| Configurado | **Grupo** | Grupo configurado + DMs de autorizados |

### Filtrado (en orden)

1. Mensajes > 30 segundos de antigüedad → ignorar (cola offline al reconectar)
2. Modo DM: mensajes de grupo → ignorar
3. Modo Grupo: mensajes de otro grupo → ignorar
4. `IsFromMe` → ignorar siempre
5. Número no está en `ALLOWED_NUMBERS` → **silencio total** (no revelar existencia)

### Variables de entorno relevantes

| Variable | Descripción |
|---|---|
| `ALLOWED_NUMBERS` | Números autorizados (`+5491112345678`) |
| `WHATSAPP_GROUP_JID` | JID del grupo. Vacío = modo DM. |

---

## Comandos (Nushell)

```nu
task build          # Compila bin/aria y bin/aria-admin
task dev            # Ejecuta el bot
task dev:admin      # Ejecuta la admin TUI
task lint           # Verificaciones de código
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/carlospereira5)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/carlospereira5)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
