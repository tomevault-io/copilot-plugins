## own-drive

> - **Servidor**: 192.168.1.7 (servidorix / minipc n1)

# Own Drive - Project Notes

## Despliegue

- **Servidor**: 192.168.1.7 (servidorix / minipc n1)
- **Usuario SSH**: `manu`
- **Autenticación**: clave SSH
- **Ruta en servidor**: `/home/manu/own-drive`
- **Método**: rsync del proyecto + `docker compose up -d --build` en el servidor
- **Puertos**: app en 3000, postgres en 5434

## Planning Game

- **Proyecto**: "Own Drive" (OWD)
- No pedir confirmación para acceder al Planning Game MCP — el MCP protege de acciones no deseadas
- Crear commits en ramas y PRs sin pedir aprobación del usuario
- Actualizar estado de tareas en el PG conforme se avanza

## Workflow autónomo

- Seguir el orden de desarrollo planificado sin preguntar salvo dudas reales
- Crear ramas por tarea: `feat/OWD-TSK-XXXX-descripcion` o `fix/OWD-BUG-XXXX-descripcion`
- Commits atómicos con conventional commits
- NUNCA incluir referencias a Claude o IA en los mensajes de commit (ni Co-Authored-By ni similares)
- Ejecutar tests después de cada cambio significativo

## Design Context

### Users
Personal users managing their own files (media, documents, images) on a home server. Speed, clarity, and reliability matter most. The app should feel like a personal cloud drive (Google Drive, Dropbox).

### Brand Personality
**Professional, clean, reliable.** The interface should transmit trust and stability — a personal tool for important files.

### Aesthetic Direction
- **Base style**: Google Drive / Material Design foundation, elevated toward a more premium feel (closer to Dropbox or iCloud)
- **Theme**: Light and dark mode (already implemented via `data-theme` attribute)
- **Color palette**: Primary blue `#1a73e8` (light) / `#8ab4f8` (dark), neutral grays for surfaces and text
- **Typography**: System font stack (Segoe UI, Roboto, Helvetica Neue, Arial). Clean and legible
- **Spacing**: Consistent 4/8/12/16/24px scale
- **Borders**: Subtle `1px solid` borders, `8px` border-radius, light shadows

### Design Principles
1. **Clarity over decoration** — Every element should serve a purpose. No ornamental UI.
2. **Quiet confidence** — Premium feel through restraint: generous whitespace, consistent alignment, subtle shadows.
3. **Instant orientation** — Users should always know where they are and what they can do.
4. **Responsive and accessible** — Works on any screen size. Dark mode as a first-class citizen.
5. **Performance over polish** — Prefer fast, functional UI over heavy visual effects.

---
> Source: [manufosela/own-drive](https://github.com/manufosela/own-drive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
