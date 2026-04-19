## fluxcorechat

> **Documento supremo y autónomo. No depende de ningún documento externo.**


es mandato 
# FLUXCORE — Canon Arquitectónico Definitivo (v8.3)

**Documento supremo y autónomo. No depende de ningún documento externo.**
**Si el código contradice este documento, el código está en error.**
**Si este documento se contradice a sí mismo, es un bug crítico del Canon.**

---

## 0. Resumen Ejecutivo

FluxCore es un **Work Operating System (WOS)**: un sistema que permite a una empresa operar su actividad mediante lenguaje natural.

El chat es la interfaz. La IA es un mecanismo. El resultado real son **acciones operativas auditables sobre el mundo**.

El sistema opera sobre cinco invariantes no negociables:

1. El **Kernel** es la única fuente de verdad. Todo estado es derivado de él.
2. Los **proyectores** son funciones puras y atómicas. No tienen efectos secundarios.
3. Los **runtimes** son decisores soberanos. Reciben contexto completo ya resuelto, devuelven acciones. Nunca consultan bases de datos.
4. La **cognición** se ejecuta sobre hechos certificados del Kernel, después de que la proyección estructural mínima esté materializada.
5. La **unidad de decisión cognitiva es el turno conversacional**, no la señal individual.

---

## 1. Ontología del Sistema

### 1.1 Los tres dominios

La arquitectura separa tres dominios con responsabilidades que no se superponen.

#### ChatCore — Sistema de Comunicación

ChatCore es el sistema de mensajería. Persiste conversaciones, mensajes, perfiles, plantillas y contactos. Emite eventos al bus. Renderiza UI conversacional. Provee infraestructura de configuración extensible.

**ChatCore no contiene inteligencia artificial ni lógica operativa de negocio.**
**ChatCore no sabe que existe la IA.**

Únicamente transporta información, dispara eventos y renderiza UI.

#### FluxCore — Work Operating System

FluxCore es el cerebro operativo. Certifica observaciones físicas en el Journal inmutable, convierte señales en contexto de negocio, y orquesta runtimes sobre ese contexto.

FluxCore no es el chat. No es el negocio del cliente. Es el **fedatario de la realidad y el motor de decisión**.

#### Sistemas de Dominio (externos)

Agendas, pedidos, catálogos, ERPs, CRMs, pagos. No pertenecen a ninguno de los dos sistemas anteriores. FluxCore permite operarlos mediante conversación pero no es su propietario.

---

### 1.2 Test ontológico de propiedad

Para determinar si un dato pertenece a ChatCore o a FluxCore:

> **¿Este dato existiría si no hubiera IA en el sistema?**

**Sí → ChatCore:** conversaciones, mensajes, plantillas, perfiles de cuenta (nombre, bio, avatar), horarios comerciales, ubicación, contactos, notas, sitio web.

**No → FluxCore:** configuración de runtimes, instrucciones de asistentes, configuración de modelos y providers, modo de automatización, reglas cognitivas por contacto, vector stores, WorkDefinitions.

> **¿Este dato gobierna cómo opera la IA, independientemente de qué runtime esté activo?**

**Sí → PolicyContext** (política de negocio): tono, idioma, emojis, modo, ventanas de turno, horario de atención.

**No → RuntimeConfig** (configuración técnica del runtime): instrucciones del asistente, modelo, proveedor, temperatura, datos de perfil autorizados.

Esta segunda pregunta es fundamental. PolicyContext y RuntimeConfig son conceptos distintos que el Canon anterior mezclaba.

---

### 1.3 Tres capas de configuración por cuenta

Un operador que configura su sistema trabaja con tres capas:

| Capa | Dónde vive | Qué contiene | Quién la usa |
|---|---|---|---|
| **Perfil de negocio** | ChatCore (`accounts`, `templates`, etc.) | Nombre, horarios, ubicación, plantillas | Todo el sistema |
| **Política de operación** | FluxCore → PolicyContext | Modo, tono, idioma, ventanas, horario IA | CognitionWorker antes de invocar runtime |
| **Configuración del runtime** | FluxCore → `fluxcore_assistants` | Instrucciones, modelo, proveedor, datos autorizados | El runtime específico activo |

Un operador que configura AsistentesLocal:
1. Va al módulo FluxCore → Asistentes.
2. Crea un asistente: escribe instrucciones, elige modelo y proveedor, autoriza qué datos del perfil puede usar (nombre, horarios, etc.), elige qué herramientas puede invocar.
3. Lo marca como activo.
4. Configura la política de operación: tono, modo (auto/suggest/off), ventanas de turno.

El runtime recibe: la política resuelta + su configuración resuelta + el historial conversacional. No necesita consultar nada.

---

## 2. El Kernel — Journal Inmutable (RFC-0001 — CONGELADO)

### 2.1 Propósito

El Kernel registra observaciones físicas certificables y garantiza que todo el estado del sistema pueda reconstruirse leyendo el Journal en orden.

**Distinción fundamental:**
- **Fenómeno**: algo que ocurre en el mundo externo.
- **Observación**: evidencia física que FluxCore recibe de ese fenómeno.
- **SystemFact**: la fila en `fluxcore_signals` que certifica haber recibido esa observación.

El Kernel no afirma el fenómeno. Afirma la observación.

### 2.2 Esquema SQL del Journal

```sql
CREATE TABLE fluxcore_reality_adapters (
    adapter_id      TEXT PRIMARY KEY,
    driver_id       TEXT NOT NULL,
    adapter_class   TEXT NOT NULL CHECK (adapter_class IN ('SENSOR', 'GATEWAY', 'INTERPRETER')),
    description     TEXT NOT NULL,
    signing_secret  TEXT NOT NULL,
    adapter_version TEXT NOT NULL,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE TABLE fluxcore_signals (
    sequence_number        BIGSERIAL PRIMARY KEY,
    signal_fingerprint     TEXT NOT NULL UNIQUE,
    fact_type              TEXT NOT NULL CHECK (fact_type IN (
        'EXTERNAL_INPUT_OBSERVED',
        'EXTERNAL_STATE_OBSERVED',
        'DELIVERY_SIGNAL_OBSERVED',
        'MEDIA_CAPTURED',
        'SYSTEM_TIMER_ELAPSED',
        'CONNECTION_EVENT_OBSERVED'
    )),
    source_namespace       TEXT NOT NULL,
    source_key             TEXT NOT NULL,
    subject_namespace      TEXT,
    subject_key            TEXT,
    object_namespace       TEXT,
    object_key             TEXT,
    evidence_raw           JSONB NOT NULL,
    evidence_format        TEXT NOT NULL,
    evidence_checksum      TEXT NOT NULL,
    provenance_driver_id   TEXT NOT NULL,
    provenance_external_id TEXT,
    provenance_entry_point TEXT,
    certified_by_adapter   TEXT NOT NULL REFERENCES fluxcore_reality_adapters(adapter_id),
    certified_adapter_version TEXT NOT NULL,
    claimed_occurred_at    TIMESTAMPTZ,
    observed_at            TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    CHECK ((subject_namespace IS NULL) = (subject_key IS NULL))
);

CREATE FUNCTION fluxcore_prevent_mutation() RETURNS trigger AS $$
BEGIN RAISE EXCEPTION 'fluxcore_signals is immutable'; END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER fluxcore_no_update
BEFORE UPDATE ON fluxcore_signals FOR EACH ROW
EXECUTE FUNCTION fluxcore_prevent_mutation();

CREATE TRIGGER fluxcore_no_delete
BEFORE DELETE ON fluxcore_signals FOR EACH ROW
EXECUTE FUNCTION fluxcore_prevent_mutation();

CREATE TABLE fluxcore_outbox (
    id              BIGSERIAL PRIMARY KEY,
    sequence_number BIGINT NOT NULL UNIQUE REFERENCES fluxcore_signals(sequence_number),
    created_at      TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

CREATE TABLE fluxcore_projector_cursors (
    projector_name       TEXT PRIMARY KEY,
    last_sequence_number BIGINT NOT NULL DEFAULT 0,
    updated_at           TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);

CREATE TABLE fluxcore_projector_errors (
    id              BIGSERIAL PRIMARY KEY,
    projector_name  TEXT NOT NULL,
    signal_seq      BIGINT NOT NULL REFERENCES fluxcore_signals(sequence_number),
    error_message   TEXT NOT NULL,
    error_stack     TEXT,
    attempts        INT NOT NULL DEFAULT 1,
    first_failed_at TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    last_failed_at  TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
    resolved_at     TIMESTAMPTZ,
    UNIQUE (projector_name, signal_seq)
);
```

### 2.3 Los seis tipos de señal

El Kernel solo acepta estas seis clases de hechos físicos. Toda semántica de negocio se deriva en proyectores.

**`EXTERNAL_INPUT_OBSERVED`** — Un actor externo envió información deliberada. Ejemplos: mensaje de texto, audio, archivo, click en botón. Se proyecta como `Message` en ChatCore.

**`EXTERNAL_STATE_OBSERVED`** — Cambio pasivo en el estado del mundo exterior. Ejemplos: confirmación de lectura, estado "escribiendo", cambio de perfil. Se proyecta como actualización de metadatos de `Conversation` o `Participant`.

**`DELIVERY_SIGNAL_OBSERVED`** — El canal confirma entrega de una salida previa del sistema. Ejemplos: ack de recepción, error de envío. Se proyecta como actualización de estado de `Message`.

**`MEDIA_CAPTURED`** — Ingreso de un activo binario persistente. Ejemplos: imagen en S3, documento adjunto. Se proyecta como creación de `Asset`.

**`SYSTEM_TIMER_ELAPSED`** — Hito temporal agendado internamente. Ejemplos: timeout, cron. Dispara evaluaciones de políticas o transiciones de Work.

**`CONNECTION_EVENT_OBSERVED`** — Cambio en topología de conexión o de identidad. Ejemplos: webhook desconectado, visitante autenticado. El `IdentityProjector` y el `ChatProjector` reaccionan según el subtipo.

### 2.4 Reality Adapters

Los Reality Adapters son el único punto de entrada al Kernel. Reciben observaciones externas, las normalizan a una de las seis clases físicas, las firman criptográficamente y llaman a `kernel.ingestSignal()`.

**Clases permitidas para escribir en el Kernel:** `SENSOR`, `GATEWAY`. Los `INTERPRETER` (IA) jamás escriben en el Kernel.

**Principio de separación:** La validación de firma del canal externo (HMAC de WhatsApp, token de Telegram) ocurre dentro del adapter correspondiente, antes de llamar a `ingestSignal()`. FluxCore no conoce WhatsApp ni Telegram. Solo conoce señales certificadas.

**Adapters del sistema base:**

La distinción entre adapters es **si el backend puede garantizar la identidad del emisor antes de certificar la señal**.

| `adapter_id` | `driver_id` | Cuándo usar | Identidad |
|---|---|---|---|
| `chatcore-gateway` | `chatcore/internal` | Señales desde el backend autenticado: conversaciones entre cuentas logueadas, acciones de operadores. | Siempre resuelta. `accountId` verificado server-side antes de ingestar. |
| `chatcore-webchat-gateway` | `chatcore/webchat` | Señales desde el widget embebible (en cualquier dominio). El mensaje viene del browser; el backend no puede verificar identidad antes de ingestar. | Puede ser anónima. Se usa `visitor_token` como identidad provisional. |

Los adapters de canal externo (WhatsApp, Telegram, etc.) se registran cuando se activa la integración correspondiente. El dominio donde aparecen no es relevante para el Kernel.

**SQL de registro de adapters base (ejecutar en toda instalación):**

```sql
INSERT INTO fluxcore_reality_adapters
  (adapter_id, driver_id, adapter_class, description, signing_secret, adapter_version)
VALUES
  ('chatcore-gateway', 'chatcore/internal', 'GATEWAY',
   'Certifica mensajes desde el backend autenticado de la plataforma.',
   :chatcore_signing_secret, '1.0.0'),
  ('chatcore-webchat-gateway', 'chatcore/webchat', 'GATEWAY',
   'Certifica mensajes desde el widget embebible. Identidad puede ser provisional.',
   :webchat_signing_secret, '1.0.0')
ON CONFLICT (adapter_id) DO NOTHING;
```

### 2.5 Identidad provisional — visitor_token

El widget genera un `visitor_token` (UUID v4) en el browser, persistido en `localStorage`. Es el `subject_key` de todas las señales de ese visitante mientras sea anónimo.

El `IdentityProjector` crea un actor provisional al ver la primera señal de un `visitor_token` desconocido. El actor provisional queda vinculado al `tenant_id` incluido en `evidence_raw` (la cuenta del cliente dueño del widget).

Cuando el visitante se autentica, el backend certifica una señal `CONNECTION_EVENT_OBSERVED` con `certified_by_adapter: 'chatcore-gateway'`, vinculando el `visitor_token` con la cuenta real. **El Journal no se muta.** Las señales anónimas anteriores permanecen anónimas porque eso fue lo que ocurrió. El vínculo es un hecho nuevo.

### 2.6 Contrato de ingestión

```typescript
kernel.ingestSignal(candidate: KernelCandidateSignal): Promise<bigint>
```

El Kernel valida la firma, calcula el fingerprint, persiste en `fluxcore_signals` y escribe en `fluxcore_outbox` en la **misma transacción**. Señales duplicadas devuelven el `sequence_number` existente sin error.

### 2.7 Reglas operativas del Kernel

1. Prohibido introducir `accountId`, `conversationId`, `messageId` o datos de UI en el Journal.
2. Solo adapters registrados con firma válida pueden invocar `ingestSignal()`.
3. Solo adapters clase `SENSOR` o `GATEWAY` pueden certificar observaciones. Los `INTERPRETER` jamás escriben en el Kernel.
4. Toda señal incluye `source` (namespace + key) que describe el origen causal.
5. Todo proyector debe poder reconstruir su estado leyendo `fluxcore_signals ORDER BY sequence_number`. Si no es posible, es un bug.
6. Cada proyector actualiza su cursor en la **misma transacción** en que escribe sus tablas derivadas.
7. Si un proyector falla al procesar una señal, registra el error en `fluxcore_projector_errors` y detiene el loop. El cursor **no avanza**. La señal se reintentará en el próximo ciclo.
8. La vinculación de un visitante anónimo a una cuenta real se certifica como una nueva señal `CONNECTION_EVENT_OBSERVED`. El Journal no se muta.

---

## 3. Principios Fundamentales

### 3.1 Inmutabilidad del Kernel
El Journal es append-only. RFC-0001 está congelado. No hay debates sobre el Kernel.

### 3.2 Proyectores puros y atómicos
Cada proyector lee del Journal, escribe en tablas derivadas y actualiza su cursor en una sola transacción atómica. Nunca llama a servicios con efectos externos. Nunca genera nuevos hechos en el Journal. Si falla, no avanza el cursor.

### 3.3 Cognición post-proyección estructural
La cognición se ejecuta después de que la proyección estructural mínima esté materializada: identidad del actor, `accountId`, canal de origen. El CognitionWorker verifica esto antes de invocar al runtime.

### 3.4 Runtimes soberanos con contexto completo
Los runtimes reciben todo lo que necesitan para decidir. No consultan bases de datos durante `handleMessage`. No tienen estado entre invocaciones. No se invocan entre sí.

El CognitionWorker resuelve todo el contexto antes de invocar al runtime:
- `PolicyContext`: la política de negocio (cómo y cuándo operar).
- `RuntimeConfig`: la configuración técnica del runtime activo (instrucciones, herramientas, datos autorizados).
- `conversationHistory`: historial semántico desde `messages`.

### 3.5 Herramientas mediadas — dos categorías
Los runtimes nunca ejecutan herramientas directamente. Toda interacción con el mundo externo pasa por el `ActionExecutor`. Hay dos categorías:

**Query Services** (consulta bajo demanda): El runtime utiliza servicios externos para refinar su decisión. A diferencia de un RAG tradicional estático, en v8.3 el modelo **decide cuándo solicitar datos adicionales** (ej: `search_knowledge`) si sus instrucciones base son insuficientes. Esto evita ejecuciones redundantes "a ciegas" antes del prompt, optimizando costos y precisión semántica. Se ejecutan durante el ciclo cognitivo del runtime.

**Effect Actions** (efectos en el mundo): El runtime declara una intención que produce un cambio. Ejemplo: enviar un mensaje, crear una cita. Se devuelven como `ExecutionAction[]` y los ejecuta el `ActionExecutor`.

Esta distinción es invariante. Un runtime que ejecuta un efecto directamente viola la soberanía.

### 3.6 Turno conversacional como unidad de decisión
La unidad de ingesta es la señal. La unidad de decisión cognitiva es el turno. Señales de una misma conversación dentro de una ventana temporal producen una única ejecución cognitiva.

### 3.7 Reconstruibilidad total
Todo el estado del sistema debe poder regenerarse leyendo `fluxcore_signals ORDER BY sequence_number`. Si no es posible, es un bug de diseño.

---

## 4. Componentes del Sistema

### 4.1 Proyectores

**Patrón de implementación obligatorio:**

```typescript
async wakeUp() {
  const lastSeq = await this.getCursor();
  const signals = await db.query.fluxcoreSignals.findMany({
    where: gt(fluxcoreSignals.sequenceNumber, lastSeq),
    orderBy: asc(fluxcoreSignals.sequenceNumber),
    limit: 100,
  });

  for (const signal of signals) {
    try {
      await db.transaction(async (tx) => {
        await this.project(signal, tx);
        await this.updateCursor(signal.sequenceNumber, tx);
      });
      // Post-transacción: emitir eventos de dominio (best-effort)
      await this.emitDomainEvent(signal).catch(() => {});
    } catch (error) {
      // Registrar error. El cursor NO avanza. El loop se detiene.
      await db.insert(fluxcoreProjectorErrors)
        .values({
          projectorName: this.name,
          signalSeq: signal.sequenceNumber,
          errorMessage: error instanceof Error ? error.message : String(error),
          errorStack: error instanceof Error ? error.stack : undefined,
        })
        .onConflictDoUpdate({
          target: [fluxcoreProjectorErrors.projectorName, fluxcoreProjectorErrors.signalSeq],
          set: {
            attempts: sql`${fluxcoreProjectorErrors.attempts} + 1`,
            lastFailedAt: new Date(),
          },
        });
      break; // detener loop, no avanzar cursor
    }
  }
}
```

**Tipos de proyectores:**

- **`IdentityProjector`**: Resuelve actores. Para señales del `chatcore-webchat-gateway`, crea actores provisionales. Para `CONNECTION_EVENT_OBSERVED` de vinculación, crea el identity link. Es la proyección estructural mínima requerida antes de cognición.
- **`ChatProjector`**: Crea `messages` y actualiza `conversations`. Dentro de su transacción, escribe en `fluxcore_cognition_queue`. Post-transacción emite `message.received`.
- **`WorkProjector`**: Mantiene estado de Works activos (para Fluxi).
- **`SessionProjector`**: Proyecta sesiones de usuario.

**Idempotencia:** `messages` tiene `UNIQUE (signal_id)`. Toda inserción usa `ON CONFLICT DO NOTHING`.

---

### 4.2 fluxcore_cognition_queue — Turn-Window

```sql
CREATE TABLE fluxcore_cognition_queue (
  id                     BIGSERIAL PRIMARY KEY,
  conversation_id        TEXT NOT NULL,
  account_id             TEXT NOT NULL,
  last_signal_seq        BIGINT NOT NULL REFERENCES fluxcore_signals(sequence_number),
  turn_started_at        TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp(),
  turn_window_expires_at TIMESTAMPTZ NOT NULL,
  processed_at           TIMESTAMPTZ,
  attempts               INT NOT NULL DEFAULT 0,
  last_error             TEXT,
  UNIQUE (conversation_id) WHERE processed_at IS NULL
);

CREATE INDEX idx_cognition_queue_ready
  ON fluxcore_cognition_queue(turn_window_expires_at)
  WHERE processed_at IS NULL;
```

**Dos tipos de señal extienden la ventana. Solo una crea mensaje.**

| Tipo de señal | Acción en `messages` | Acción en cognition_queue |
|---|---|---|
| `EXTERNAL_INPUT_OBSERVED` | Crea registro | Upsert: extiende ventana en `turnWindowMs`, actualiza `last_signal_seq` |
| `EXTERNAL_STATE_OBSERVED` (typing/recording) | No crea mensaje | Upsert: extiende ventana en `turnWindowTypingMs`, **NO actualiza `last_signal_seq`** |
| Cualquier otro tipo | No aplica | No toca la cola |

`turnWindowTypingMs > turnWindowMs` porque un usuario escribiendo puede tardar 10-20 segundos. Ambos viven en PolicyContext.

El `CognitionWorker` consulta:
```sql
SELECT * FROM fluxcore_cognition_queue
WHERE processed_at IS NULL AND turn_window_expires_at < now()
ORDER BY turn_window_expires_at ASC
LIMIT 10
FOR UPDATE SKIP LOCKED;
```

**El CognitionWorker nunca interpreta el tipo de señal.** Solo consume entradas cuya ventana venció.

---

### 4.3 PolicyContext — Política de Negocio

El PolicyContext contiene las decisiones de negocio que gobiernan cómo opera la IA. **No contiene configuración técnica del runtime.**

```typescript
interface FluxPolicyContext {
  // Identidad resuelta (estructural)
  accountId: string;
  contactId: string;
  channel: string;

  // Atención — cómo debe responder
  tone: 'formal' | 'casual' | 'neutral';
  useEmojis: boolean;
  language: string;

  // Automatización — si debe responder y cómo
  mode: 'auto' | 'suggest' | 'off';
  responseDelayMs: number;

  // Turno conversacional
  turnWindowMs: number;
  turnWindowTypingMs: number;
  turnWindowMaxMs: number;

  // Reglas de negocio
  offHoursPolicy: OffHoursPolicy;
  contactRules: ContactRule[];

  // Runtime activo
  activeRuntimeId: string; // ID registrado en RuntimeGateway

  // Autorización de plantillas a nivel de política
  authorizedTemplates: string[];

  // Datos de perfil del negocio resueltos y autorizados para uso por IA
  // Solo incluye los campos que el operador autorizó en la configuración del asistente
  resolvedBusinessProfile: {
    displayName?: string;
    businessHours?: BusinessHours;
    location?: string;
    website?: string;
    [key: string]: unknown; // extensible
  };

  // Estado operativo activo (para Fluxi)
  activeWork?: ActiveWorkContext;
  workDefinitions?: WorkDefinition[];
}
```

**El `mode` es el gate de cognición.** `off` → no se invoca ningún runtime. `suggest` → el resultado se presenta como sugerencia al operador. `auto` → se ejecuta directamente.

**El `responseDelayMs` es distinto del `turnWindowMs`.** La ventana agrega ráfagas. El delay introduce una pausa intencional después del cierre del turno y antes de invocar al runtime (UX: "alguien está pensando"). El CognitionWorker aplica este delay.

**`resolvedBusinessProfile`** contiene solo los campos que el operador autorizó explícitamente en la configuración del asistente. Si el operador desautoriza un campo, desaparece del PolicyContext en el próximo turno.

---

### 4.4 RuntimeConfig — Configuración Técnica del Runtime

El RuntimeConfig contiene la configuración específica del runtime activo para esta cuenta. Es distinto del PolicyContext: PolicyContext gobierna el comportamiento; RuntimeConfig especifica la implementación.

```typescript
interface RuntimeConfig {
  runtimeId: string;
  accountId: string;

  // Instrucciones del asistente (para runtimes cognitivos)
  instructions?: string;

  // Configuración del modelo (para AsistentesLocal)
  provider?: string;       // 'groq' | 'openai' | 'anthropic' | ...
  model?: string;          // 'llama-3.1-8b-instant' | 'gpt-4o-mini' | ...
  temperature?: number;
  vectorStoreId?: string;

  // Referencia a assistant externo (para AsistentesOpenAI)
  externalAssistantId?: string; // formato 'asst_abc123'

  // Herramientas autorizadas por el operador en la configuración del asistente
  authorizedTools: string[];

  // Datos adicionales específicos del runtime
  [key: string]: unknown;
}
```

**Tabla `fluxcore_assistants`:**

```sql
CREATE TABLE fluxcore_assistants (
  id                    TEXT PRIMARY KEY DEFAULT gen_random_uuid()::text,
  account_id            TEXT NOT NULL,
  name                  TEXT NOT NULL,
  runtime_id            TEXT NOT NULL,    -- qué runtime configura
  status                TEXT NOT NULL DEFAULT 'draft'
                        CHECK (status IN ('active', 'draft', 'archived')),
  instructions          TEXT,
  provider              TEXT,
  model                 TEXT,
  temperature           DECIMAL(3,2),
  vector_store_id       TEXT,
  external_assistant_id TEXT,             -- para AsistentesOpenAI
  authorized_tools      TEXT[] NOT NULL DEFAULT '{}',
  authorized_data_scopes TEXT[] NOT NULL DEFAULT '{}', -- campos del perfil autorizados
  created_at            TIMESTAMPTZ NOT NULL DEFAULT now(),
  updated_at            TIMESTAMPTZ NOT NULL DEFAULT now()
);

CREATE UNIQUE INDEX idx_assistants_active_per_account
  ON fluxcore_assistants(account_id, runtime_id)
  WHERE status = 'active';
-- Solo puede haber un asistente activo por (cuenta, runtime)
```

**El operador configura qué campos del perfil de negocio puede usar el asistente** en `authorized_data_scopes`. Ejemplos: `['displayName', 'businessHours', 'location']`. El `FluxPolicyContextService` lee estos scopes, resuelve los valores desde ChatCore, y los incluye en `policyContext.resolvedBusinessProfile`. Si el operador desautoriza un campo, el asistente deja de verlo en el siguiente turno.

---

### 4.5 RuntimeInput — Contrato de Entrada del Runtime

```typescript
interface RuntimeInput {
  policyContext: FluxPolicyContext;      // política de negocio resuelta
  runtimeConfig: RuntimeConfig;         // configuración técnica resuelta
  conversationHistory: ConversationMessage[]; // historial semántico completo
}

interface ConversationMessage {
  role: 'user' | 'assistant' | 'system';
  content: string;
  createdAt: Date;
}
```

**`signal` no está en RuntimeInput.** El runtime es un decisor semántico. Recibe representación lingüística del mundo, no evidencia física cruda. Si el runtime necesita saber el canal, está en `policyContext.channel`. Si necesita saber qué se dijo en este turno, está en `conversationHistory`.

El `CognitionWorker` construye `RuntimeInput` completo antes de invocar al runtime. El runtime no consulta DB en ningún momento.

---

### 4.6 Event Bus

Los eventos se emiten **después** de la transacción exitosa.

| Evento | Origen | Semántica |
|---|---|---|
| `message.received` | `ChatProjector` | Mensaje humano proyectado. Para UI vía WebSocket. |
| `message.sent` | `ActionExecutor` | Mensaje del sistema insertado como `pending`. |
| `work.opened` | `ActionExecutor` | Work creado. |
| `work.advanced` | `ActionExecutor` | Transición de estado en Work. |
| `identity.linked` | `IdentityProjector` | Visitante anónimo vinculado a cuenta real. |

**La distinción `message.received` / `message.sent` es no negociable.** El mismo evento para ambos crearía un loop donde el CognitionWorker reacciona a sus propias respuestas.

---

### 4.7 CognitionWorker y CognitiveDispatcher

El `CognitionWorker` es el latido del sistema. Cada ciclo:

1. Consulta `fluxcore_cognition_queue WHERE turn_window_expires_at < now() FOR UPDATE SKIP LOCKED`.
2. Para cada turno listo:
   a. Verifica que `IdentityProjector` materializó la identidad. Si no, extiende la ventana 500ms y libera el lock.
   b. Lee todos los mensajes del turno desde `messages`.
   c. Construye `PolicyContext` via `FluxPolicyContextService`.
   d. Lee `RuntimeConfig` via `RuntimeConfigService` (consulta `fluxcore_assistants`).
   e. Obtiene historial conversacional previo desde `messages`.
   f. Aplica `responseDelayMs` si > 0.
   g. Invoca al `RuntimeGateway`.

**Por qué el historial viene de `messages` y no del Journal:** El Journal tiene evidencia física cruda. Los runtimes necesitan representación lingüística (`role: user|assistant`, `content: string`). Reconstruir eso desde el Journal equivale a reimplementar el `ChatProjector` en el Dispatcher.

---

### 4.8 Runtime Gateway

Mantiene un registro de `runtimeId → RuntimeAdapter`. Invoca `runtime.handleMessage(input)` y pasa las acciones al `ActionExecutor`.

**El runtime activo lo elige el operador desde la configuración administrativa.** FluxCore solo respeta esa decisión.

**Si el `activeRuntimeId` no tiene runtime registrado:** el CognitionWorker registra el error en `last_error`, hace backoff exponencial sin quemar `attempts`, y notifica operacionalmente.

---

### 4.9 RuntimeAdapter — Contrato de los Runtimes

```typescript
interface RuntimeAdapter {
  readonly runtimeId: string;
  handleMessage(input: RuntimeInput): Promise<ExecutionAction[]>;
}
```

**Invariantes de todos los runtimes:**
- No consultan bases de datos durante `handleMessage`. Todo llega en `RuntimeInput`.
- No tienen estado global entre invocaciones.
- No se invocan entre sí. Son alternativas completas, no colaboradores.
- No ejecutan Effect Actions directamente. Las devuelven.
- Siempre producen un resultado: respuesta, sugerencia, propuesta, o `no_action` justificado.
- No deciden si ejecutarse. Eso lo decide el `CognitionWorker` via `policyContext.mode`.

---

### 4.10 Runtime: AsistentesLocal

**Naturaleza:** Cognitivo probabilístico, ejecución interna. FluxCore controla el prompt, las llamadas al modelo y el loop.

**Cómo opera:**
1. Recibe `RuntimeInput` completo. No consulta DB.
2. `PromptBuilder` combina en secciones distintas:
   - Sección 1 (prioridad): PolicyContext — tono, idioma, emojis, `resolvedBusinessProfile`, reglas del contacto. Es la voz del negocio.
   - Sección 2: instrucciones del asistente desde `runtimeConfig.instructions`.
3. **Query Services (Mediados)**: Si el modelo determina que su contexto es insuficiente, invoca dinámicamente el `KnowledgeService` inyectado. El resultado se integra en una segunda ronda cognitiva (Tool Loop). Esto garantiza que el RAG sea una elección consciente del modelo y no una carga fija por cada turno.
4. Llama al LLM (provider y modelo desde `runtimeConfig`). Fallback al provider secundario si falla.
5. Tool Loop máximo 2 rounds.
6. Devuelve `ExecutionAction[]`.

**`KnowledgeService` es un Query Service, no un Effect Action.** El runtime lo llama para obtener contexto. No modifica el mundo. Usar un servicio inyectado síncrono para esto no viola el principio de soberanía porque no es un efecto — es una consulta necesaria para construir la respuesta.

---

### 4.11 Runtime: AsistentesOpenAI

**Naturaleza:** Cognitivo probabilístico, ejecución remota. FluxCore actúa como puente hacia OpenAI Assistants API.

**Cómo opera:**
1. Recibe `RuntimeInput`. Lee `runtimeConfig.externalAssistantId`.
2. Construye thread de OpenAI desde `conversationHistory`.
3. Inyecta PolicyContext como override de instructions.
4. Ejecuta run. Polling hasta completar.
5. Devuelve `ExecutionAction[]`.

**AsistentesLocal y AsistentesOpenAI son implementaciones paralelas e independientes.** No son sub-modos del mismo runtime. Nunca se invocan entre sí.

---

### 4.12 Runtime: Fluxi

**Naturaleza:** Transaccional determinista. Mismo contrato `RuntimeAdapter`.

**Qué resuelve:** Cuando el usuario dice "quiero agendar un turno", eso no es conversación — es una transacción con pasos, datos a confirmar y efecto real en un sistema externo. Fluxi convierte esa intención en un **Work**: estructura auditurable que avanza paso a paso hasta producir el efecto o fallar de forma controlada.

**Cómo opera el operador:** Activa Fluxi en la configuración y define `WorkDefinitions` ("si alguien quiere agendar, ejecutar este Work con estos slots"). Una vez activado, Fluxi opera autónomamente sin intervención.

**Flujo interno:**

**Fase 1 — Work activo?**
Si existe Work en progreso para la conversación, el turno se entrega al Work Engine. No se buscan nuevas intenciones.

**Fase 2 — Interpretación (si no hay Work activo)**
El WES Interpreter analiza el turno buscando intención transaccional. Recibe el texto y las `WorkDefinitions` disponibles (desde `runtimeConfig.workDefinitions`). Devuelve `ProposedWork` o `null`. Si `null`, Fluxi devuelve `send_message` indicando que no detectó intención operativa. **Nunca cae al runtime de Asistentes.**

**Fase 3 — Gate de apertura (determinista)**
El `ProposedWork` debe superar: WorkDefinition existe y la cuenta tiene permiso; existe al menos un `bindingAttribute` con evidencia no vacía; no hay conflicto de concurrencia.

**Fase 4 — FSM del Work**

| Estado | Significado |
|---|---|
| `CREATED` | Work instanciado, slots iniciales persistidos. |
| `ACTIVE` | Completando slots con mensajes del usuario. |
| `WAITING_USER` | Fluxi hizo una pregunta, espera respuesta. |
| `WAITING_CONFIRMATION` | SemanticContext emitido, espera confirmación. |
| `EXECUTING` | ExternalEffectClaim adquirido, invocando herramienta. |
| `COMPLETED` | Trabajo finalizado con éxito. |
| `FAILED` | Error no recuperable. |
| `EXPIRED` | TTL vencido sin completar. |

**Fase 5 — Confirmación semántica**
Para slots ambiguos: Fluxi genera `SemanticContext` (UUID único) y pregunta al usuario. Cuando el usuario confirma, se resuelve contra el `SemanticContext` pendiente **sin LLM**. Se registra un `SemanticCommit`. Si el Work original expiró entre la pregunta y la confirmación, se reabre desde el contexto sin reinterpretar el mensaje. Un `SemanticContext` solo puede consumirse una vez.

**Fase 6 — Efectos externos (exactly-once)**
Antes de invocar cualquier herramienta irreversible: adquirir `ExternalEffectClaim` (clave `accountId + semanticContextId + effectType`). Si el claim ya existe, abortar. Si tiene éxito, invocar herramienta con `idempotencyKey`.

**Tablas de Fluxi:** `fluxcore_work_definitions`, `fluxcore_works`, `fluxcore_work_slots`, `fluxcore_work_events`, `fluxcore_proposed_works`, `fluxcore_decision_events`, `fluxcore_semantic_contexts`, `fluxcore_external_effect_claims`, `fluxcore_external_effects`.

**Invariantes de Fluxi:**
1. Fluxi nunca llama al runtime de Asistentes.
2. No hay Work sin ProposedWork persistido previo.
3. No hay efecto externo sin `ExternalEffectClaim` adquirido.
4. La IA solo propone; el Work Engine (determinista) dispone.
5. Las confirmaciones se resuelven contra `SemanticContext`, nunca con LLM.
6. Todo cambio de estado genera `WorkEvent` inmutable.
7. Un `SemanticContext` solo puede consumirse una vez.

---

### 4.13 Herramientas — Registro y Mediación

**Dos categorías:**

**Query Services** — Consultas que el runtime necesita para construir su decisión. No producen efectos en el mundo. Son síncronos en v8.3, asíncronos en v8.4.

| Query Service | Implementación | Propiedad |
|---|---|---|
| `search_knowledge` | RAG service de FluxCore | FluxCore |
| `get_contact_notes` | Servicio de contactos de ChatCore | ChatCore |

**Effect Actions** — Intenciones que producen cambios en el mundo. Siempre declaradas como acciones, siempre ejecutadas por `ActionExecutor`.

| Effect Action | Efecto |
|---|---|
| `send_message` | Inserta en `messages` como `pending`. Emite `message.sent`. |
| `send_template` | Igual que `send_message`, usando plantilla autorizada. |
| `propose_work` | Crea `ProposedWork` en Fluxi. |
| `open_work` | Instancia Work desde ProposedWork. |
| `advance_work_state` | Transición de estado en Work activo. |
| `request_slot` | Genera `SemanticContext` y mensaje de solicitud. |
| `close_work` | Cierra Work con `COMPLETED` o `FAILED`. |
| `no_action` | Registra que el runtime procesó el turno sin efectos. |

FluxCore mantiene el registro de todas las herramientas y media el acceso. Los runtimes no saben ni les importa la implementación subyacente de una herramienta.

ChatCore expone sus capacidades como servicios. FluxCore los envuelve como adaptadores. ChatCore no sabe que existe la IA.

---

### 4.14 ActionExecutor

Único componente autorizado a producir Effect Actions en el mundo.

**Responsabilidades:**
- Validar que la acción esté en `policyContext.authorizedTemplates` o `runtimeConfig.authorizedTools`.
- Ejecutar el efecto.
- Registrar log de auditoría por cada acción en `fluxcore_action_audit`.
- Emitir eventos de dominio post-ejecución.
- Marcar `fluxcore_cognition_queue.processed_at` al completar sin error.

```sql
CREATE TABLE fluxcore_action_audit (
  id              BIGSERIAL PRIMARY KEY,
  conversation_id TEXT NOT NULL,
  account_id      TEXT NOT NULL,
  runtime_id      TEXT NOT NULL,
  action_type     TEXT NOT NULL,
  action_payload  JSONB,
  status          TEXT NOT NULL CHECK (status IN ('executed', 'rejected', 'failed')),
  rejection_reason TEXT,
  executed_at     TIMESTAMPTZ NOT NULL DEFAULT clock_timestamp()
);
```

---

## 5. Flujo Completo de un Turno

### Paso 1 — Ingesta
Un mensaje llega por un canal. El Reality Adapter correspondiente lo normaliza a `EXTERNAL_INPUT_OBSERVED`, firma la señal y llama a `kernel.ingestSignal()`. El Kernel persiste en `fluxcore_signals` y escribe en `fluxcore_outbox` en la misma transacción.

### Paso 2 — Proyección estructural
El `IdentityProjector` se despierta. Resuelve la identidad del actor (`accountId`, `contactId`). Todo en una transacción atómica con su cursor.

### Paso 3 — Proyección conversacional y encolado
El `ChatProjector` se despierta. En una sola transacción atómica:
1. Inserta en `messages` con `ON CONFLICT (signal_id) DO NOTHING`.
2. Actualiza `conversations`.
3. Upsert en `fluxcore_cognition_queue`: extiende ventana si ya hay turno pendiente, crea entrada si no.
4. Actualiza su cursor.

Post-transacción: emite `message.received` al Event Bus.

### Paso 4 — Cierre de turno
El `CognitionWorker` espera a que `turn_window_expires_at < now()`. Mientras tanto, señales adicionales del mismo usuario (mensajes o typing) extienden la ventana. Al vencer, el worker toma la entrada.

### Paso 5 — Resolución del contexto
El worker verifica identidad materializada. Construye `PolicyContext` + `RuntimeConfig`. Lee historial conversacional. Aplica `responseDelayMs`.

### Paso 6 — Decisión del runtime
`RuntimeGateway` invoca `runtime.handleMessage(RuntimeInput)`. El runtime procesa y devuelve `ExecutionAction[]`.

### Paso 7 — Ejecución
`ActionExecutor` valida y ejecuta cada acción. Registra auditoría. Marca `processed_at` en la cola.

### Paso 8 — Entrega al canal
Worker dedicado toma mensajes `pending` y los envía al canal real. Estado → `sent`. Al confirmar entrega: el Reality Adapter ingresa `DELIVERY_SIGNAL_OBSERVED`, el `ChatProjector` actualiza estado → `delivered`.

---

## 6. Invariantes del Sistema

Deben ser verdaderas en todo momento. Una violación es un bug crítico.

1. `fluxcore_signals` es append-only. Ningún `UPDATE` o `DELETE` es posible.
2. Todo proyector actualiza su cursor en la misma transacción en que escribe sus tablas derivadas.
3. Si un proyector falla al procesar una señal, el cursor no avanza y el error se registra en `fluxcore_projector_errors`.
4. `messages` tiene `UNIQUE (signal_id)`. Toda inserción usa `ON CONFLICT DO NOTHING`.
5. `fluxcore_cognition_queue` se escribe dentro de la transacción del `ChatProjector`. Nunca después.
6. Existe máximo una entrada pendiente por `conversation_id` en `fluxcore_cognition_queue`.
7. El `CognitionWorker` no procesa un turno hasta que `turn_window_expires_at < now()`.
8. Las señales `EXTERNAL_STATE_OBSERVED` de typing/recording extienden la ventana pero no crean registros en `messages` y no actualizan `last_signal_seq`.
9. El `CognitionWorker` nunca interpreta el tipo de señal. Solo consume entradas cuya ventana venció.
10. Ningún runtime consulta bases de datos durante `handleMessage`. Todo llega en `RuntimeInput`.
11. Los runtimes no se invocan entre sí.
12. `ActionExecutor` es el único componente que produce Effect Actions en el mundo.
13. `message.received` solo lo emite el `ChatProjector`.
14. `message.sent` solo lo emite el `ActionExecutor`.
15. No existe `ExternalEffectClaim` sin `ProposedWork` previo.
16. Un `SemanticContext` solo puede consumirse una vez.
17. Solo adapters clase `SENSOR` o `GATEWAY` pueden invocar `kernel.ingestSignal()`.
18. La vinculación de visitante anónimo a cuenta real se certifica como señal nueva. El Journal no se muta.
19. `FluxPolicyContextService` resuelve `resolvedBusinessProfile` solo con los campos en `authorized_data_scopes` del asistente activo.

---

## 7. Reglas Inviolables

1. ChatCore nunca ejecuta lógica de negocio.
2. ChatCore nunca sabe que existe la IA.
3. FluxCore nunca contiene la operación real del cliente.
4. Los runtimes nunca se invocan entre sí.
5. El operador decide qué runtime opera. FluxCore respeta esa decisión.
6. Toda Effect Action irreversible de Fluxi requiere `ExternalEffectClaim`.
7. Las políticas de atención pertenecen a FluxCore, no a ChatCore.
8. Las herramientas son registradas y mediadas por FluxCore.
9. Un runtime recibe un turno y siempre produce un resultado.
10. Los `INTERPRETER` (IA) jamás escriben en el Kernel.
11. El Kernel es inmutable. Solo puede extenderse con nuevos adapters y proyectores.
12. PolicyContext = política de negocio. RuntimeConfig = configuración técnica del runtime. Son conceptos distintos. No se mezclan.

---

## 8. Riesgos y Mitigaciones

| Riesgo | Severidad | Mitigación |
|---|---|---|
| Señal que falla bloquea el proyector indefinidamente | Alta | `fluxcore_projector_errors` + alerta operacional si `attempts > 3` |
| ChatProjector proyecta antes que IdentityProjector | Alta | CognitionWorker verifica identidad antes de disparar; extiende ventana 500ms si no está lista |
| Turno procesado dos veces | Alta | `UNIQUE (signal_id)` en `messages`; segundo procesamiento hace `ON CONFLICT DO NOTHING` |
| Runtime configurado con ID no registrado | Media | CognitionWorker hace backoff sin quemar attempts; alerta operacional |
| Turno nunca cierra (usuario escribe continuamente) | Media | `turnWindowMaxMs` como techo absoluto |
| Latencia percibida como lentitud | Baja | `turnWindowMs` configurable por canal; web chat puede usar 500ms |

---

## 9. Deuda Técnica Explícita

**v8.4 — Query Services asíncronos:** En v8.3, los Query Services son síncronos (servicios inyectados). En v8.4 se implementa re-invocación con `continuationToken`: el runtime devuelve una acción de consulta, el ActionExecutor la ejecuta, el CognitionWorker re-invoca con el resultado. No requiere cambiar el contrato `RuntimeAdapter`.

**v8.5 — RuntimeRouter:** El diseño actual asume un único runtime activo por cuenta. El routing por tipo de mensaje (Fluxi para transaccionales, Asistentes para el resto) requiere un `RuntimeRouter` no especificado en este Canon.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/harvan88) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
