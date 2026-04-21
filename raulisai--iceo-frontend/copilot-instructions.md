## iceo-frontend

> Reglas Generales para Crear la Web Estilo Videojuego con Agentes de nombre iceo-forntend


Reglas Generales para Crear la Web Estilo Videojuego con Agentes

1. Experiencia Estilo Videojuego
Interfaz inmersiva tipo mapa o simulación (como The Sims).

Elementos UI inspirados en videojuegos: HUD, tiempo, botones animados.

Transiciones suaves entre vistas (sin recargas duras).

2. Navegación y Flujo de Usuario
Ruta principal Home (planta baja): vista general de un edificio central virtual (tipo torre) donde se puede elegir un piso.

Ruta por oficina: al seleccionar una, se ingresa a su espacio virtual con agentes activos.

Ruta individual de agente: se visualiza su estado, acciones recientes, y tareas asignadas.

Fallback o error (404): pantalla con temática tipo “pantalla de glitch” o error visual simulado.

3. Agentes Autónomos
Los agentes deben verse como NPCs con personalidad (nombre, tipo, habilidades).

Acciones posibles:

Escribir código

Enviar correos

Tomar decisiones basadas en eventos

Pueden estar activos/inactivos según la energía o tiempo de tarea.

4. Lógica de Estados y Eventos
El estado de cada agente se actualiza en tiempo real o semirreal (polling/WebSockets).

Eventos tipo “misión”, “error del sistema” o “interacción con otro agente” disparan animaciones o alertas visuales.

Pueden existir acciones en cadena (una acción de un agente detona actividad en otro).

5. Diseño Visual
Aesthetic tipo “retro-futurista”, “minimalista”, o “pixel UI”.

Música o sonidos opcionales tipo juego.

Botones grandes, con hover animado, tipo consola o tablero digital.

6. Modularidad y Escalabilidad
Cada vista debe ser reutilizable para otras oficinas o agentes.

Componentes separados y desacoplados: interfaz, lógica, estado, visualización.

Rutas nuevas deben integrarse sin reestructurar lo anterior.

7. Modo Simulación + Modo Control
Vista libre para explorar como espectador (modo simulación).

Vista editable para intervenir o controlar agentes (modo administrador o modo juego).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/raulisai) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
