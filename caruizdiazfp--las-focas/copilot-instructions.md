## las-focas

> - **Rol:** Líder Técnico / Developer Fullstack.

# Nombre de archivo: Gemini.md
# Ubicación de archivo: Gemini.md
# Descripción: Instrucciones para Gemini sobre el proyecto LAS-FOCAS


## 👤 El Usuario (Arquitecto)
- **Rol:** Líder Técnico / Developer Fullstack.
- **Objetivo:** Crecer el proyecto LAS-FOCAS para que sea la base de la automatización operativa de Metrotel.
- **Estilo de Comunicación:** Casual, directo, usa emojis 🚀.
- **Formato de Respuesta:** Español. Detallado, con secciones claras, tablas y fuentes al final.

## 🛠️ Preferencias Técnicas (Strict Mode)
1. **Encabezados:** NUNCA olvides la regla del encabezado de 3 líneas (Nombre, Ubicación, Descripción). Si creas un archivo, ponlo primero. [Fuente: AGENTS.md]
2. **Docker-First:** Si sugieres un script, asume que corre dentro de un contenedor, no en mi máquina local.
3. **Librerías:**
    - Preferencia por `pydantic` para validación de datos.
    - `FastAPI` para endpoints.
    - `SQLAlchemy` + `Alembic` para DB.
4. **Manejo de Errores:** Fail-safe. Si algo falla, loguealo estructuradamente y reintenta o degrada elegantemente.
5. **Legacy:** La carpeta `Legacy/` es LAVA 🌋. Solo lectura. No sugieras editar nada ahí. [Fuente: decisiones.md]

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CARuizDiazFP)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/CARuizDiazFP)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
