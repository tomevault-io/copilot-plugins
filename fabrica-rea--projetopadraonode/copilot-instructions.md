## projetopadraonode

> globs: ["**/routes/**/*.js", "**/index.js"]


---
trigger: glob
globs: ["**/routes/**/*.js", "**/index.js"]
---
<!-- Rules for API development and routing -->

# Standard JSON Response Structure
All API responses MUST follow this structure.
- **Success**: `res.status(200).json({ success: true, data: [...] })`
- **Client Error**: `res.status(400).json({ success: false, message: 'Invalid input provided.' })`
- **Server Error**: `res.status(500).json({ success: false, message: 'An unexpected error occurred.' })`

# HTTP Status Codes
- `200 OK`: Successful GET, PUT/PATCH.
- `201 Created`: Successful POST.
- `204 No Content`: Successful DELETE.
- `400 Bad Request`: Input validation errors.
- `401 Unauthorized`: Missing or invalid authentication.
- `403 Forbidden`: Authenticated user lacks permissions.
- `404 Not Found`: Resource does not exist.
- `500 Internal Server Error`: Unhandled server-side errors.

# API Generation
- Use connection pooling via `mysql2/promise`.
- Wrap all database calls in `try/catch` blocks.
- Use `express-validator` for input validation on all POST and PUT/PATCH routes.

- **GET Routes**: Implement following the pattern in `routes/Usuario.js`.
- **POST Routes**: Follow `router.post('/usuarioapi', ...)` in `routes/Usuario.js`. Return `201 Created`.
- **DELETE Routes**: Follow `router.delete('/usuarioapid/:id', ...)` in `routes/Usuario.js`. Return `204 No Content`.

# Route Registration
When creating `routes/Tabela.js`:
1.  Import in `index.js`: `const tabelaRoutes = require('./routes/Tabela');`
2.  Register in `index.js`: `app.use(config.api.prefix, tabelaRoutes);`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Fabrica-REA) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
