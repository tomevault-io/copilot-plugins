## tienda3d

> Flujo obligatorio para cambios críticos de estados/stock/ventas


# Critical Flow Guard

## Antes de tocar lógica crítica
- Definir en pocas líneas: entrada, validaciones, transición de estado y rollback.
- Enumerar invariantes que no se pueden romper:
  - No perder trazabilidad de ingresos/gastos.
  - No permitir estados inconsistentes en pedidos.
  - No romper cálculo de stock por color.

## Durante la implementación
- Priorizar cambios pequeños y atómicos.
- Si se modifica un flujo de estado, agregar validación explícita de precondiciones.
- Si hay mensaje de error al usuario, debe mostrarse en UI y no en texto plano.

## Cierre obligatorio
- Ejecutar `go build ./...` y `go test ./...`.
- Reportar: qué cambió, por qué, y qué comportamiento quedó protegido.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phenrril) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
