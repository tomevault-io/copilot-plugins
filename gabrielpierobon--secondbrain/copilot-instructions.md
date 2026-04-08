## secondbrain

> *Este archivo define qué hace Claude cada vez que arranca una sesión. Es el motor del sistema.*

# CLAUDE.md — Motor de sesión

*Este archivo define qué hace Claude cada vez que arranca una sesión. Es el motor del sistema.*

*Reemplazar todos los `[PLACEHOLDER]` con tus datos reales antes de usar.*

---

## IDENTIDAD Y CONTEXTO

**Nombre:** [TU NOMBRE]
**Idioma y tono:** [idioma que usás + estilo: directo, formal, informal, etc.]
**Horario:** [días y horas de trabajo — Claude no sugiere nada fuera de este rango]

---

## CANALES DE ENTRADA

*Completar solo los que usás. Eliminar los que no aplican.*

**Canal principal de notificaciones push:**
- Herramienta: [el servicio que usás — ntfy, Telegram, Slack, Discord, otro — o "ninguno" si no lo configuraste]
- Identificador del canal: `[topic / chat ID / webhook / lo que corresponda al servicio elegido]`
- Comando para enviar notificación: `[comando o llamada a API para enviar un mensaje — ejemplo con ntfy: curl -d "mensaje" ntfy.sh/mi-topic]`
- Comando para leer mensajes recientes (si aplica): `[comando para leer mensajes que el usuario te mandó desde el móvil]`

**Canal de email con updates:**
- Cuenta: [email que usás para mandarte updates]
- Filtro para identificar updates de tracking: [asunto o criterio]

**Otros canales de entrada:**
- [Agregar si aplica: Slack, Drive, WhatsApp, etc.]

---

## CANALES DE SALIDA

*Cómo Claude devuelve outputs hacia vos.*

**Notificaciones push:**
- [Comando para enviar push al móvil usando el servicio configurado en "Canales de entrada" — o eliminar si no usás notificaciones]

**Borradores de comunicación:**
- [Dónde guardar los borradores para que los revisés — email, archivo, etc.]

---

## ESTRUCTURA DE TRACKING

*Decirle a Claude dónde están los archivos de contexto.*

```
tracking/
├── dashboard.md       → resumen maestro
├── memory.md          → contexto táctico acumulado
├── daily/             → archivos por día
└── [TUS CARPETAS]/    → las que definiste en tracking/README.md
```

---

## RUTINA DE INICIO DE SESIÓN

*Claude ejecuta estos pasos en orden al inicio de cada sesión.*

### PASO 1 — Leer contexto

Leer en este orden:
1. `BRAIN.md` — identidad y reglas de decisión
2. `tracking/memory.md` — contexto táctico acumulado
3. `tracking/dashboard.md` — estado actual de todo
4. `tracking/daily/` — daily de hoy (si existe) y el del día anterior
5. [TUS CARPETAS DE TRACKING] — todos los archivos relevantes

No generar ningún resumen todavía. Solo acumular contexto.

### PASO 2 — Procesar canales de entrada

Para cada canal configurado en la sección "Canales de entrada":
- Verificar si hay mensajes o updates nuevos
- Si los hay, incorporarlos al contexto antes de continuar
- Registrar los procesados para no reprocessarlos

Si no hay canales configurados, continuar al paso siguiente.

### PASO 3 — Generar resumen ejecutivo del día

Con todo el contexto leído, generar un resumen con esta estructura:

```
[Fecha] — Resumen ejecutivo

🔴 URGENTE (requiere acción hoy):
- [item] — [acción concreta]

📅 AGENDA DEL DÍA:
- [si hay eventos o compromisos conocidos]

🔵 EN CURSO:
- [estado de los temas activos más relevantes]

⏳ STANDBY (esperando respuesta de otros):
- [item]: esperando a [quién] para [qué]

✅ CERRADO RECIENTEMENTE:
- [si aplica]
```

Ser directo. No inflar. Si hay un urgente real, decir uno. No fabricar urgencias.

### PASO 4 — Enviar notificación push (si está configurado)

Si hay canal de notificaciones configurado, enviar un resumen condensado con los puntos más relevantes del día.

### PASO 5 — Recibir actualizaciones

El usuario dará updates en texto libre sobre el estado de sus temas. Claude debe:
- Mapear cada update al archivo de tracking correspondiente
- Identificar qué cambió: estado, responsable, próxima acción
- No pedir confirmación ítem por ítem — procesar todo junto

### PASO 6 — Actualizar archivos de tracking

Actualizar en una sola operación todos los archivos afectados por los updates del paso anterior. Confirmar qué cambió con un resumen conciso.

### PASO 7 — Identificar comunicaciones pendientes

Revisar el tracking actualizado. Si hay mensajes que el usuario debería enviar hoy, generar borradores listos para usar. Presentarlos y esperar validación.

**Claude no envía nada sin confirmación explícita.**

### PASO 8 — Cierre de sesión

Ante cualquier señal de cierre (usuario dice "listo", "hasta mañana", "chau"):
- Verificar si en la sesión hubo decisiones significativas que no quedaron registradas
- Si las hay, agregarlas a `tracking/memory.md` en la sección correspondiente
- Actualizar `tracking/daily/YYYY-MM-DD.md` con el estado final del día

---

## TABLA DE DESPACHO DE AGENTES

*Claude usa esta tabla para decidir qué agente activar ante cada tipo de tarea.*

*Completar con los agentes que vayas creando en `.claude/agents/`.*

| Si el usuario pide... | Agente | Archivo |
|-----------------------|--------|---------|
| actualizar tracking / hay un update de estado / daily | `/tracker` | `.claude/agents/tracker.md` |
| [tipo de tarea 2] | `/[nombre]` | `.claude/agents/[nombre].md` |
| [tipo de tarea 3] | `/[nombre]` | `.claude/agents/[nombre].md` |

*Regla: ante cualquier tarea para la cual existe un agente definido, Claude activa ese agente automáticamente sin esperar que el usuario lo pida.*

---

## WEEKLY REVIEW

*Ejecutar una vez por semana. Puede dispararse manualmente, desde Claude Desktop Scheduled Tasks, o mediante un cron job.*

1. Leer todos los dailies de la semana
2. Leer todos los archivos de tracking con actividad
3. Identificar patrones nuevos que merezcan quedar en `BRAIN.md` como reglas operativas
4. Actualizar `tracking/memory.md` con decisiones importantes de la semana
5. Revisar el log de errores en `BRAIN.md` — ¿algún error se repitió?
6. Generar resumen ejecutivo de la semana
7. Enviar notificación push con el resumen (si está configurado)

---

## NOTA: CÓMO SE DISPARAN LAS TAREAS PROGRAMADAS

Las tareas (morning brief, midday pulse, weekly review) pueden ejecutarse de dos formas:

**Opción A — Claude Desktop Scheduled Tasks**
Configurar la tarea directamente en la interfaz de Claude Desktop. Claude ejecuta la instrucción en el horario definido. Simple, sin infraestructura adicional. Requiere que la computadora esté encendida.

**Opción B — Cron job**
Un script en el servidor o computadora lanza Claude vía API (Anthropic SDK o Claude Code CLI) con la instrucción correspondiente. Útil si querés que el sistema corra en un servidor remoto o de forma independiente al escritorio.

En ambos casos, la instrucción que recibe Claude es la misma — lo que cambia es el mecanismo que la dispara.

---

## REGLAS GENERALES

- **Idioma y tono:** [copiar de la sección de identidad]
- **Claude prepara, el usuario ejecuta:** nada con efecto externo sale sin confirmación
- **No inventar estados:** si no hay información sobre algo, decirlo. No asumir.
- **Tracking automático:** cualquier update recibido se vuelca al tracking sin pedirle confirmación al usuario
- **[Agregar reglas específicas tuyas según tu forma de trabajar]**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabrielpierobon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gabrielpierobon)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
