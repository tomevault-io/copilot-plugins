## cobrofacil

> 🔄 FLUJO COMPLETO DE TRANSFERENCIA DE MESAS CON LIBERACIÓN AUTOMÁTICA DE MOZOS

🔄 FLUJO COMPLETO DE TRANSFERENCIA DE MESAS CON LIBERACIÓN AUTOMÁTICA DE MOZOS
📋 PROCESO PASO A PASO ACTUALIZADO
1. INICIO DE TRANSFERENCIA
Usuario abre mesa con ítems (estado ROJO) y mozo asignado
En el panel de ventas, hace clic en botón "TRANSFERIR"
Se abre modal de transferencia con 3 pasos
2. PASO 1: TIPO DE TRANSFERENCIA
Opción A: Transferencia COMPLETA → Todos los ítems + mozo se mueven
Opción B: Transferencia PARCIAL → Solo ítems seleccionados (mozo se queda)
Usuario selecciona tipo y presiona "Siguiente"
3. PASO 2: SELECCIÓN DE MESA DESTINO
Sistema muestra mesas disponibles organizadas por sectores
Solo se muestran mesas en estado LIBRE (verde)
Usuario selecciona mesa destino y presiona "Siguiente"
4. PASO 3: SELECCIÓN DE ÍTEMS (Solo si es parcial)
Si eligió transferencia COMPLETA: se transfieren todos automáticamente + mozo
Si eligió transferencia PARCIAL:
Lista de todos los ítems con checkboxes
Botón "Seleccionar/Deseleccionar Todos"
Usuario marca los ítems que quiere transferir
Mozo permanece en mesa origen
5. CONFIRMACIÓN FINAL
Pantalla de resumen mostrando:
Mesa origen y destino
Tipo de transferencia
Ítems a transferir (con cantidades y precios)
Estado del mozo: "Se transferirá" o "Permanece en mesa origen"
Total a transferir
Usuario confirma con "TRANSFERIR"
6. EJECUCIÓN DE LA TRANSFERENCIA
Sistema envía petición al backend
Backend procesa la transferencia:
Paso 1: Remueve ítems de venta origen
Paso 2: Crea/actualiza venta en mesa destino
Paso 3: Si es transferencia completa:
Obtiene mozo asignado de mesa origen
Asigna mozo a mesa destino
Libera mozo de mesa origen
Paso 4: Actualiza estados de ambas mesas
7. VALIDACIÓN AUTOMÁTICA POST-TRANSFERENCIA
Si es TRANSFERENCIA COMPLETA:
✅ Mesa origen: Queda sin ítems + SIN MOZO → Cambia a LIBRE (verde)
✅ Mesa destino: Recibe ítems + RECIBE MOZO → Cambia a OCUPADA (rojo)
✅ Modal de transferir: Se cierra automáticamente
✅ Panel de ventas: Se cierra automáticamente (venta ya no existe)
✅ Mozo: Continúa atendiendo en la nueva mesa sin interrupción
Si es TRANSFERENCIA PARCIAL:
✅ Mesa origen: Mantiene ítems restantes + MANTIENE MOZO → Sigue OCUPADA (rojo)
✅ Mesa destino: Recibe ítems + SIN MOZO → Cambia a OCUPADA (rojo)
✅ Modal de transferir: Se cierra automáticamente
✅ Panel de ventas: Permanece abierto (con ítems restantes)
✅ Mesa destino: Necesitará asignación manual de mozo
8. LIBERACIÓN AUTOMÁTICA EN OTROS ESCENARIOS
Finalización de Venta/Cobro:
Cliente paga y venta se completa
Mesa queda sin productos automáticamente
Mozo se libera automáticamente
Mesa pasa a LIBRE (verde) completamente limpia
Eliminación Manual de Productos:
Usuario elimina productos uno por uno
Al eliminar el último producto de la mesa
Sistema libera automáticamente el mozo
Mesa pasa a LIBRE (verde) sin mozo asignado
Validación Automática del Sistema:
Sistema detecta mesa marcada como ocupada sin productos
Corrige estado a LIBRE automáticamente
Libera mozo asignado automáticamente
Registra motivo: "Mesa vacía - Liberación automática"
9. SINCRONIZACIÓN DE ESTADOS
Sistema valida automáticamente estados de ambas mesas (300ms delay)
Valida asignaciones de mozos en ambas mesas (500ms delay)
Actualiza vista global de todas las mesas (500ms delay)
Refresca sectores para mostrar cambios visuales
10. FEEDBACK AL USUARIO
Transferencia Completa: "Transferencia completa realizada: productos y mozo transferidos"
Transferencia Parcial: "Transferencia parcial realizada: productos seleccionados transferidos"
Liberación Automática: "Mesa liberada - Mozo disponible para nuevas asignaciones"
Cambios visuales inmediatos en el grid de mesas
Sin errores 404 ni mensajes de error
🎯 CASOS DE USO ACTUALIZADOS
CASO 1: Mesa completa se cambia de ubicación
Mesa A3 (Mozo Juan + 2 hamburguesas + 1 bebida) → Mesa B5 (vacía)

ANTES:
- Mesa A3: OCUPADA (rojo) + Mozo Juan + 3 productos
- Mesa B5: LIBRE (verde) + Sin mozo + Sin productos

RESULTADO:
- Mesa A3: LIBRE (verde) + SIN MOZO + Sin productos ← ¡COMPLETAMENTE LIMPIA!
- Mesa B5: OCUPADA (rojo) + Mozo Juan + 3 productos ← ¡TODO TRANSFERIDO!
- Panel se cierra automáticamente
- Mozo Juan continúa atendiendo sin interrupciones
CASO 2: División de grupo - algunos platos van a otra mesa
Mesa A3 (Mozo Juan + 2 hamburguesas + 1 bebida) → Mesa B5 (vacía)
Usuario selecciona: solo 1 hamburguesa

ANTES:
- Mesa A3: OCUPADA + Mozo Juan + 3 productos
- Mesa B5: LIBRE + Sin mozo + Sin productos

RESULTADO:
- Mesa A3: OCUPADA (rojo) + Mozo Juan + 2 productos ← ¡MOZO SE QUEDA!
- Mesa B5: OCUPADA (rojo) + SIN MOZO + 1 producto ← ¡NECESITA MOZO!
- Panel permanece abierto en Mesa A3
- Mesa B5 requiere asignación manual de mozo
CASO 3: Cliente finaliza y paga
Mesa C2 (Mozo María + 3 platos) → Cliente paga y se va

ANTES:
- Mesa C2: OCUPADA (rojo) + Mozo María + 3 productos

PROCESO:
- Cliente solicita cuenta
- Se emite ticket/factura
- Se registra pago
- Venta se completa

RESULTADO:
- Mesa C2: LIBRE (verde) + SIN MOZO + Sin productos ← ¡AUTO-LIMPIEZA!
- María queda disponible para nueva asignación inmediatamente
CASO 4: Cancelación de pedido completo
- Mesa C2: LIBRE (verde) + SIN MOZO + Sin productos ← ¡AUTO-LIMPIEZA!
- María queda disponible para nueva asignación inmediatamente
CASO 4: Cancelación de pedido completo
Mesa D1 (Mozo Pedro + 2 productos) → Se cancelan todos los productos

ANTES:
- Mesa D1: OCUPADA (rojo) + Mozo Pedro + 2 productos

PROCESO:
- Usuario elimina producto 1
- Usuario elimina producto 2 (último)
- Sistema detecta mesa sin productos

RESULTADO:
- Mesa D1: LIBRE (verde) + SIN MOZO + Sin productos ← ¡LIBERACIÓN AUTOMÁTICA!
- Pedro disponible inmediatamente para nueva mesa
CASO 5: Fusión de múltiples mesas ocupadas
Mesa A3 (Mozo Ana + 2 platos) + Mesa B5 (Mozo Luis + 1 bebida) → Mesa C1 (vacía)

PROCESO:
- Transferir completa A3 → C1 (Ana va a C1, A3 queda limpia)
- Transferir completa B5 → C1 (Luis se libera, B5 queda limpia)

RESULTADO:
- Mesa A3: LIBRE (verde) + SIN MOZO + Sin productos
- Mesa B5: LIBRE (verde) + SIN MOZO + Sin productos  
- Mesa C1: OCUPADA (rojo) + Mozo Ana + 3 ítems ← ¡ANA ATIENDE TODO!
- Luis queda disponible para nueva asignación
⚡ OPTIMIZACIONES Y BENEFICIOS IMPLEMENTADOS
UX Mejoradas:
Transferencia transparente de mozos: Servicio continuo sin interrupciones
Liberación automática inteligente: Sin asignaciones fantasma
Validación automática: Estados siempre coherentes sin intervención manual
Feedback diferenciado: Mensajes claros según tipo de transferencia
Sin cierres abruptos: Modales se cierran solo cuando corresponde
Operativas:
Eficiencia de mozos: Disponibilidad inmediata al liberar mesas
Limpieza automática: Mesas siempre listas para nuevos clientes
Consistencia total: Estados de mesa y asignaciones siempre coherentes
Trazabilidad completa: Registro detallado de todos los movimientos
Técnicas:
Validación dual: Se validan ambas mesas y asignaciones de mozo
Timing optimizado: Delays calculados para evitar condiciones de carrera
Rollback automático: Si falla transferencia de mozo, productos se transfieren igual
Sincronización global: Actualiza toda la vista después de cambios
Manejo robusto de errores: Tolerancia a fallos en operaciones de mozo
🔍 PUNTOS CRÍTICOS DE CONTROL ACTUALIZADOS
Antes de transferir:
Verificar que mesa destino existe y está libre
Confirmar disponibilidad de mozo para transferencia
Validar permisos de usuario para transferir
Durante transferencia:
Validar que ítems existen y son transferibles
Verificar éxito en transferencia de productos
Confirmar transferencia exitosa de mozo (si aplica)
Manejar fallos parciales con rollback inteligente
Después de transferir:
Confirmar estados correctos de ambas mesas
Validar asignaciones de mozo en ambas mesas
Verificar liberación automática si aplica
Registrar trazabilidad completa
Sincronización:
Asegurar que todos los dispositivos ven los cambios
Validar consistencia entre estado de mesa y asignación de mozo
Refrescar vista global de mesas y mozos
Limpieza:
Cerrar modales y paneles según el tipo de transferencia
Liberar recursos y limpiar estados temporales
Activar validaciones automáticas post-operación
🎉 RESULTADO FINAL DEL SISTEMA
✅ Para el Negocio:
Operación fluida: Transferencias sin interrupciones del servicio
Eficiencia máxima: Mozos siempre disponibles cuando corresponde
Limpieza automática: Mesas listas inmediatamente para nuevos clientes
Consistencia total: Sin discrepancias entre estados y asignaciones
✅ Para los Mozos:
Continuidad de servicio: Siguen atendiendo tras transferencias completas
Liberación automática: No necesitan desasignarse manualmente
Disponibilidad inmediata: Listos para nuevas mesas automáticamente
Sin confusiones: Estado claro de sus asignaciones en todo momento
✅ Para el Sistema:
Robustez operativa: Manejo inteligente de todos los escenarios
Trazabilidad completa: Registro detallado de todos los movimientos
Mantenimiento automático: Sin intervención manual requerida
Escalabilidad: Funciona con múltiples dispositivos y usuarios
🚀 GARANTÍAS DEL FLUJO
✅ Mesa vacía = Mozo liberado automáticamente (siempre)
✅ Transferencia completa = Mozo se transfiere (siempre)
✅ Transferencia parcial = Mozo se queda en origen (siempre)
✅ Estados consistentes = Validación automática (siempre)
✅ Experiencia fluida = Sin errores ni interrupciones (garantizado)
Este flujo garantiza una operación perfecta, automatizada y libre de errores para la gestión integral de mesas y mozos.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Firbedatomas)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Firbedatomas)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
