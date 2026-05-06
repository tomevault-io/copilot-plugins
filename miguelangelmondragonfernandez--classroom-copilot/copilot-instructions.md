## 01-backend

> Reglas para el backend Node.js/Express en /backend/src — aplican a rutas, controladores y servicios.


# Backend — Reglas de Arquitectura

## Patrón Ruta → Controlador → Servicio

### Routes (`*.routes.js`)
- Solo declaran endpoints y aplican middlewares (`authMiddleware`, validadores Zod).
- **No** contienen lógica de negocio ni consultas SQL.

```js
// ✅ Correcto
router.post('/planning', authMiddleware, validateBody(planSchema), planningController.create);

// ❌ Incorrecto — lógica en la ruta
router.post('/planning', async (req, res) => {
  const result = await db.query('INSERT INTO ...');
});
```

### Controllers (`*.controller.js`)
- Reciben la `req`, extraen y validan parámetros, delegan al servicio.
- Devuelven respuestas JSON estandarizadas: `{ success: true, data: ... }` o `{ success: false, error: '...' }`.
- Manejan los `try/catch` de la capa HTTP.
- **No** conocen SQL ni llaman a APIs externas directamente.

### Services (`*.service.js`)
- Contienen toda la lógica de negocio y consultas SQL.
- **No** reciben `req` ni `res` — solo reciben parámetros planos.
- Pueden llamar a otras APIs (Google Drive, Classroom, Gemini).

## Base de Datos
- Toda modificación estructural a la BD debe ir como migración numerada en `backend/database/migrations/`.
  - Formato: `005_descripcion_corta.sql`
- Nomenclatura de tablas: `snake_case` plural (ej. `ciclos_escolares`).
- Usar siempre parámetros preparados con `mysql2` — nunca interpolación de strings en SQL.

## Validación con Zod
- Toda entrada de usuario (body, query, params) debe validarse con Zod **antes** de llegar al service.
- Definir los schemas en archivos `*.schema.js` o al inicio del archivo de rutas.

## Tokens de IA
- Toda llamada a Gemini o cualquier IA **debe** descontar tokens de la tabla `perfiles` (`token_balance`, `total_consumed`).
- Si el `token_balance` es 0 o menor, rechazar la petición con un error 402 antes de llamar a la IA.

## Gestión de Google APIs
- Los clientes de Google (`drive`, `classroom`, `docs`, `slides`) se inicializan en `/backend/src/clients/`.
- No instanciar clientes de `googleapis` dentro de los servicios directamente.

---
> Source: [MiguelAngelMondragonFernandez/Classroom-copilot](https://github.com/MiguelAngelMondragonFernandez/Classroom-copilot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
