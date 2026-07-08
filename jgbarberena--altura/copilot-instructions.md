## altura

> > Contexto transversal del proyecto. Para trabajo en el panel de administración lee también `CLAUDE_ADMIN.md`. Para trabajo en el frontend público, `CLAUDE_WEB.md`.

# CLAUDE.md — experienciasanfermin.com

> Contexto transversal del proyecto. Para trabajo en el panel de administración lee también `CLAUDE_ADMIN.md`. Para trabajo en el frontend público, `CLAUDE_WEB.md`.

> **Historial de deudas cerradas:** `CLAUDE_ADMIN_BACKLOG.md`. Ese archivo NO se carga en sesiones normales. Para añadir una deuda resuelta al backlog sin cargarlo: `Add-Content -Path 'CLAUDE_ADMIN_BACKLOG.md' -Value '...' -Encoding UTF8`.

---

## 1. Personas

**Javier** (jgbarberena@gmail.com) — desarrollador no profesional con conocimientos sólidos de programación (clases, funciones, variables, contratos, dependencias). Ha programado en VBA, HTML, JS, C++, Matlab, Mathematica y Modelica. Entiende el código cuando lo lee y sabe expresar con precisión lo que quiere. Es quien trabaja con Claude Code.

**Paula Díaz** (paula@experienciasanfermin.com / +34 625 638 977) — usuaria principal del panel de administración. Gestiona clientes por WhatsApp y correo, principalmente desde el móvil. Firma los documentos comerciales (NIF: 72694758S). No es técnica.

**Ander Sagardia** (sistemas@ycgdigitalgroup.com, YCG Digital Group) — gestiona DNS y hosting del servidor de producción.

**Hilario** (goviwebs.com) — desarrollador de tienda.sanfermin.com (WooCommerce). Contacto para cambios en la API sfcom y configuración de productos en la tienda.

---

## 2. Cómo quiero que me ayudes

- Guíame paso a paso con avance real y verificable. No me des todo de golpe.
- Cuando algo falla: diagnóstico primero (qué error exacto, dónde), luego solución mínima y concreta, luego verificación.
- Sé directo y honesto. Si hay una decisión que reconsiderar, dímelo aunque yo haya propuesto algo diferente.
- Cuando tengo que hacer algo en una interfaz externa (Supabase, GitHub, etc.), dime exactamente a qué pantalla ir, qué menú abrir y dónde hacer clic.
- Si ves deuda técnica, dímelo aunque no lo haya preguntado, pero sin insistir si decido dejarlo para después.
- Sin formateo excesivo: prosa cuando explicas, código limpio cuando programas, sin bullets innecesarios.
- No asumir que entiendo una herramienta nueva sin explicar antes qué es y cómo funciona.
- No proponer soluciones que funcionen solo para un caso particular sin pensar en la arquitectura general.

**Al inicio de cada sesión:** si Javier no indica en qué va a trabajar, pregunta si es trabajo en el panel de admin o en la web pública, para cargar el contexto adicional correcto (CLAUDE_ADMIN.md o CLAUDE_WEB.md).

**Flujo de trabajo:**
- Algo nuevo: primero diseño y decisiones (preferiblemente en claude.ai para análisis extenso), luego implementación en orden lógico (dependencias primero), luego verificación con datos reales.
- Algo que falla: diagnóstico → solución mínima → verificación.

---

## 3. El proyecto

Web de reservas de balcones y experiencias para San Fermín en **experienciasanfermin.com** (también vivesanfermin.com), con panel de administración privado para uso propio.

**Volumen:** menos de 200 reservas, menos de 100 proveedores, 2-3 usuarios del panel.

**Repositorio:** https://github.com/jgbarberena/altura  
**GitHub Pages:** https://jgbarberena.github.io/altura/ (dominio propio: experienciasanfermin.com)  
**Deploy adicional:** FTP a servidor externo. Credenciales en `.vscode/sftp.json` — no commitear cambios.  
**Entorno local:** Live Server (VSCode).

---

## 4. Stack técnico

| Capa | Tecnología |
|---|---|
| Frontend público | HTML/CSS/JS puro, sin frameworks |
| Panel de admin | Páginas HTML bajo `/admin/`, JS en módulos ES6, acceso directo a Supabase desde el navegador |
| Base de datos | Supabase (PostgreSQL), @supabase/supabase-js@2 vía CDN |
| AI | Claude API vía Edge Function `claude-proxy` en Supabase. API key en Supabase Vault. Modelos disponibles: `claude-sonnet-4-6` (default), `claude-opus-4-7`, `claude-haiku-4-5-20251001`. |
| Email inbound | Resend (dominio verificado). Inbound webhook para `in.experienciasanfermin.com`. |
| Maps | Leaflet 1.9.4 |
| Excel export | SheetJS (carga dinámica vía `import()`, ~900KB, solo al primer uso) |
| PDF | `window.print()` o jsPDF para propuestas y facturas desde el panel |
| Hosting | GitHub Pages + FTP a servidor externo |

No hay servidor propio. No hay proceso de build, bundler ni transpilación. Lo que ves es lo que se sirve.

---

## 5. Deploy

El script `deploy.ps1` (raíz del proyecto) automatiza el ciclo completo: regenera índices + SEO + sitemap → git commit/push → FTP de los archivos modificados.

**Ejecución:**
```powershell
pwsh deploy.ps1 -Message "descripción breve"
```

Usar siempre `pwsh` (PowerShell 7), no `powershell` (PS5).

**Opciones:**
- `-SkipScripts` — omite regeneración de índices/SEO/sitemap (solo commit + FTP)
- `-SkipFtp` — solo commit/push, sin FTP
- `-SkipGit` — solo FTP, sin commit

**Claude ejecuta el deploy directamente** cuando Javier lo pida ("haz el deploy", "sube los cambios"). Flujo:
1. Revisar qué se ha modificado en la conversación.
2. Redactar mensaje de commit breve en español (máx. 60 caracteres).
3. Ejecutar vía Bash:
   ```powershell
   pwsh -ExecutionPolicy Bypass -File deploy.ps1 -Message "<mensaje>"
   ```
4. Reportar resultado (archivos subidos, errores si los hay).

**Credenciales FTP:** el script las lee de `.vscode/sftp.json`. Esta carpeta está excluida del FTP y del repositorio. Otros archivos excluidos del FTP: `.git/`, `.claude/`, `CLAUDE*.md`, `deploy.ps1`, `*.ps1`, `index-template.html`, `*.zip`.

**Documentar antes de hacer deploy.** Cualquier cambio no trivial debe quedar documentado en `CLAUDE_ADMIN.md` (o `CLAUDE_WEB.md` si es frontend público) antes del deploy. Excepción: bug fixes puros donde el error era una desviación obvia del diseño y no hay nada que explicar — en ese caso basta con marcar "✅ RESUELTO" en la deuda técnica correspondiente si ya estaba anotada, y no es necesario documentar si no estaba. Javier puede dar instrucciones concretas sobre qué y cómo documentar; si no las da, documentar por defecto.

---

## 6. Principios técnicos transversales

**Sin frameworks, sin complejidad innecesaria.** El volumen no lo justifica. Soluciones gratuitas primero; si algo requiere pago, avisar y decidir.

**Fuente de verdad siempre en Supabase.** La BD nunca queda con datos incompletos ni huérfanos. FK siempre presentes. Todo en snake_case y minúsculas (IDs de texto en mayúsculas, reforzados por trigger).

**Lógica de presentación en JS, nunca en BD.** Excepción aceptada: triggers para mantener consistencia de datos, la columna generada `total_amount`, y las vistas de lectura del panel.

**Sin código duplicado.** Si algo se repite, se extrae a función o archivo compartido.

**Módulos ES6 en admin, scripts clásicos en frontend público.** Los scripts del frontend necesitan exponer funciones globales accesibles desde componentes cargados dinámicamente.

**CSS en archivos separados, nunca inline** (salvo `display:none` puntual). Sin bloques `<style>` dentro del HTML salvo en el embed del mapa (standalone, sin acceso a la hoja CSS del proyecto).

**Consistencia sobre perfección.** Si algo funciona de una forma en un sitio, funciona igual en todos.

**Sin comentarios que digan qué hace el código.** Solo los que explican el porqué no obvio. Sin referencias a cambios, fechas, autores ni tareas — eso va en el commit.

---

## 7. Convenciones de nomenclatura

- **Reservas:** `R` + 4 dígitos correlativo (`R0001`). El siguiente se calcula con `select id order by id desc limit 1` → `parseInt(id.slice(1)) + 1`.
- **Servicios:** `TIPO_DIA` en mayúsculas (`ENCIERRO_7`, `CHUPINAZO_6`).
- **Clientes:** texto libre en mayúsculas elegido por el admin (`GARCIA_PEDRO`).
- **Proveedores / venues:** texto libre en mayúsculas (`BALCON_ESTAFETA_1`).
- **Facturas:** serie `VSF-NN/AAAA` (correlativo por ejercicio fiscal).
- **Propuestas:** serie `PRP`.

**IDs de servicios activos:**
```
ENCIERRO_7  ENCIERRO_8  ENCIERRO_9  ENCIERRO_10
ENCIERRO_11  ENCIERRO_12  ENCIERRO_13  ENCIERRO_14
CHUPINAZO_6   PROCESION_7   DESPEDIDA_GIGANTES_14   POBRE_DE_MI
```

---

## 8. Lo que no quiero

- Sin código duplicado. Si algo se repite, se extrae a función o archivo compartido.
- Sin formateo excesivo en las respuestas: prosa cuando explicas, código limpio cuando programas, sin bullets innecesarios.
- No asumir que entiendo una herramienta nueva sin explicarla antes.
- No proponer soluciones de caso particular: antes de implementar algo, valorar si es coherente con el resto del sistema.
- No editar el bloque AUTO-SEO directamente. Siempre editar los elementos fuente y ejecutar el script.
- No editar `guias/index.html` ni `faq/index.html` directamente. Son archivos generados.
- No poner lógica de negocio en la BD salvo razón de peso.
- No añadir features, refactors ni abstracciones más allá de lo que pide la tarea.

---

## 9. Contactos externos

| Persona | Rol | Contacto |
|---|---|---|
| Ander Sagardia | DNS / hosting | sistemas@ycgdigitalgroup.com |
| Hilario | tienda.sanfermin.com (WooCommerce) | goviwebs.com |
| Paula Díaz | Admin del panel, firma comercial | paula@experienciasanfermin.com · +34 625 638 977 |

---
> Source: [jgbarberena/altura](https://github.com/jgbarberena/altura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
