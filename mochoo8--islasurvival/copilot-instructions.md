## islasurvival

> Juego de supervivencia en primera persona en una isla (estilo Raft / Stranded Deep).

# IslaSurvival — Guía del proyecto

## Resumen

Juego de supervivencia en primera persona en una isla (estilo Raft / Stranded Deep).
A futuro tendrá construcción en tercera persona, inventario, crafting y enemigos — **nada de eso se implementa todavía**.

- **Unity 6000.5.3f1**, plantilla **3D URP** (URP 17.5.0).
- **Input System nuevo** activo (`activeInputHandler: 1`): NO usar la API vieja `Input.GetAxis` — usar `UnityEngine.InputSystem`.
- El usuario es **principiante**: explicar cada decisión en español simple, sin asumir conocimientos.
- Modelos 3D definitivos los agrega el usuario después (Meshy/Blender). Por ahora **solo primitivas y placeholders de colores**.
- Existe un único **Mundo Principal fijo** de 2400 × 2400 m. No volver a agregar semillas ni variantes aleatorias del mapa.

## Restricción clave de flujo de trabajo

Codex no puede clickear dentro del editor de Unity. Todo lo que requiera armar escenas
(terrain, objetos, luces) se hace con **Editor Scripts** (`[MenuItem]` bajo el menú **"Herramientas"**)
que el usuario ejecuta desde Unity. Cada fase termina indicando: (a) archivos creados,
(b) qué menú apretar, (c) qué debería verse, (d) qué revisar si falla.
Se trabaja **por fases**: al terminar una, PARAR y esperar confirmación del usuario.

## Convenciones de código

- C# con namespaces `IslaSurvival.*` (el código de editor usa `IslaSurvival.EditorTools` para no chocar con `UnityEditor.Editor`).
- Clases y miembros con **nombres en español** (dominio del juego); comentarios en español solo donde aportan.
- `[SerializeField] private` en vez de campos públicos. Excepción: DTOs serializables puros (ej. `PuntosClaveIsla`) pueden usar campos públicos.
- Una responsabilidad por clase. Sin GameManager gigante. Sin `FindObjectOfType` en runtime por frame.
- Validar referencias con errores claros en consola (`Debug.LogError` con prefijo `[NombreClase]`).
- Sin assets pagos ni paquetes externos salvo imprescindible (justificar antes).
- Sin asmdefs por ahora: el código de editor va en carpetas llamadas `Editor/` (Unity las excluye de las builds automáticamente).
- Densidades razonables (cientos de objetos, no miles).
- Las herramientas de editor son **idempotentes**: regenerar reemplaza lo generado (busca por nombre bajo `[Mundo]`), nunca duplica.

## Estructura de carpetas

```
Assets/Game/
  Config/                  → assets de configuración (ScriptableObjects). Creado por las herramientas.
  Generated/               → assets generados por herramientas (TerrainData, texturas, capas, materiales).
  Scripts/
    World/Island/          → generación de la isla (ConfiguracionIsla, GeneradorAlturasIsla, PuntosClaveIsla, InfoIslaGenerada)
    Editor/                → herramientas de editor (menú "Herramientas/Mundo Principal")
    Core/                  → (futuro, Fase 2+) — no crear vacía
    World/Biomes/          → clasificación espacial de los biomas del mundo fijo
    World/ResourceSpawning/→ configuración fija de vegetación placeholder
    World/PointsOfInterest/→ (futuro, Fase 5) — no crear vacía
    World/WorldBounds/     → (futuro, Fase 5) — no crear vacía
    Player/                → controlador FPS y spawn inicial (Fase 2)
    Utilities/             → (futuro, cuando haga falta) — no crear vacía
```

Regla: **no crear carpetas vacías** para sistemas futuros; se crean cuando llega su fase.

## Cómo se construye el Mundo Principal

1. Menú **Herramientas → Mundo Principal → Construir Mundo Principal**.
2. La herramienta restaura los valores oficiales y siempre produce el mismo mapa de 2400 × 2400 m.
3. La base es una llanura insular con playas amplias. Encima se distribuyen un volcán central con cráter, dos cordilleras, colinas, cerros, valles, un cañón y una laguna.
4. Biomas: Playa, Pradera, BosqueCostero, BosqueProfundo, Jungla, BosqueOtonal, Montania, TundraHelada, CumbreNevada, Laguna y Volcan. Cada uno tiene una capa de suelo y una paleta de vegetación propia.
5. La tundra del sudeste es una zona helada baja e independiente de las montañas. La jungla occidental concentra la vegetación más densa.
6. Todo cuelga de `[Mundo]`; `InfoIslaGenerada` guarda la versión del diseño y sus puntos clave.

## Cómo se crea el jugador

1. El Mundo Principal debe estar construido con la versión actual.
2. Menú **Herramientas → Jugador → Crear o actualizar personaje FPS**.
3. La herramienta crea o actualiza `[Mundo]/Jugador`, reutiliza la cámara principal y lo ubica en `PuntosClave.playaBahia`.
4. Controles: WASD/flechas para moverse, mouse para mirar, Shift para correr, Espacio para saltar y Escape para liberar el cursor.
5. `F3` alterna el modo espectador: WASD vuela según la cámara, Espacio sube, Ctrl/Q baja y Shift acelera.
6. Reconstruir el mundo reubica al jugador y mantiene la cámara en su posición local.

Decisiones técnicas registradas:
- `GeneradorAlturasIsla` es matemática runtime pura y describe accidentes geográficos en posiciones fijas; el ruido fijo solo suaviza las formas, no produce mapas alternativos.
- Un solo plano de agua global: la laguna interior es terreno cavado por debajo del nivel del mar (no hay segundo sistema de agua).
- `DisenoMundoPrincipal` es el contrato espacial compartido por terreno, texturas, vegetación y futuros detectores de bioma.
- El alphamap usa resolución 1024 y una mezcla gaussiana de unos 15 m entre capas; no volver a pintar biomas con pesos one-hot sin suavizado.
- La vegetación usa hashes de coordenadas y selecciona como máximo 900 placeholders repartidos por toda la isla. No tiene configuración aleatoria.
- Después de aplicar el límite global, exactamente el 25% de las piedras se convierte en rocas grandes compuestas estilo Valheim.
- `ModoEspectador` es una capacidad administrativa separada del movimiento normal; al activarse deshabilita `CharacterController` y `MovimientoJugador`.
- `EntradaJugador` reutiliza el asset `Assets/InputSystem_Actions.inputactions`; las acciones se buscan una vez al iniciar, nunca por frame.
- La vegetación deja un radio libre alrededor de la playa de la bahía para que el jugador no aparezca dentro de un placeholder.

## Modelos 3D integrados

- El personaje usa `Assets/Game/Art/Characters/Player/CavemanStrongman_Capoeira.fbx`. Su clip Capoeira se conserva, pero no se reproduce como locomocion.
- El cuerpo completo vive bajo `[Mundo]/Jugador/Modelo 3D del Jugador` y se excluye solo de la camara FPS para evitar que la cabeza tape la vista.
- El arbol `EldertreeOfVines` se usa exclusivamente en Jungla.
- El pack general se convirtio a 34 FBX individuales. BosqueOtonal prioriza abedules naranjas; Pradera admite pocos cerezos/ciruelos; bosques, montania y tundra combinan especies acordes al clima.
- Las rocas musgosas se reservan para zonas humedas y las rocas secas para playa, pradera, montania, tundra, nieve y volcan. Se mantiene exactamente 25% de rocas grandes.
- `ImportadorModelosIsla` configura modelos, normales, compresion y materiales URP. `CatalogoModelosNaturaleza` es la unica tabla de asignacion por bioma.
- Menu de instalacion idempotente: **Herramientas -> Modelos 3D -> Integrar Personaje y Naturaleza**.

## Estado de avance por fases

| Fase | Contenido | Estado |
|------|-----------|--------|
| 1 | Mundo Principal fijo de 2400 m: relieves diversos, volcán, playas amplias y 10 biomas orgánicos | ✅ Código entregado — pendiente de verificación del usuario en Unity |
| 2 | Personaje jugable (CharacterController primera persona, spawn en bahía) | ✅ Verificada por el usuario |
| 3 | Biomas (BiomaDefinicion, ZonaBioma, DetectorDeBioma) | ⬜ Pendiente |
| 4 | Recursos por spawn (RecursoDefinicion, ConfigSpawnRecurso, SpawnerDeRecursos) | ⬜ Pendiente |
| 5 | Puntos de interés y límites del mundo | ⬜ Pendiente |

Al completar cada fase, actualizar esta tabla.

---
> Source: [mochoo8/IslaSurvival](https://github.com/mochoo8/IslaSurvival) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-08 -->
