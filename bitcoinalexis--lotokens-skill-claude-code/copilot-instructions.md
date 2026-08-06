## lotokens-skill-claude-code

> <!-- Generado automaticamente por la skill LoTokens. Edita o elimina esta seccion para cambiar las preferencias. -->

## LoTokens

<!-- Generado automaticamente por la skill LoTokens. Edita o elimina esta seccion para cambiar las preferencias. -->

### Bloquear .md
- NO uses Write, Edit, ni Bash con comandos que escriban a archivos `.md` (`>`, `>>`, `sed -i`, `tee`, `cp`, `mv`, `awk -i inplace`, etc.)
- Incluye README.md y cualquier otro Markdown
- EXCEPCIONES (permitidos siempre):
  - `AGENTS.md` (global o de proyecto) — para guardar preferencias e instrucciones persistentes
  - `MEMORY.md` y cualquier `.md` dentro de un directorio `memory/` — para el sistema auto-memory
- Si se intenta escribir otro .md, responde exactamente: `Bloqueado por LoTokens: escritura de .md desactivada.`

### Sin emojis
- NO incluyas emojis en respuestas de texto
- NO incluyas emojis en comentarios de codigo, strings ni documentacion inline

### Sin comentarios
- NO escribas comentarios de ningun tipo en el codigo generado
- Incluye: `#` (Python, shell, YAML), `//` (JS, TS, Go, Java, C/C++, Rust), `/* */`, `<!-- -->` (HTML/XML), `--` (SQL, Lua), `;` (Lisp, ASM)
- NO escribas docstrings de Python (`"""..."""` o `'''...'''`) ni JSDoc (`/** */`)
- NO escribas headers decorativos tipo `# ===== CONFIGURACION =====` ni separadores de seccion
- NO escribas comentarios de "que hace este archivo" al inicio del script
- NO escribas comentarios al final de linea explicando codigo (`x = 5  # contador`)
- Si modificas un archivo existente que YA tiene comentarios, no agregues nuevos pero deja los existentes intactos (a menos que el usuario pida quitarlos)
- EXCEPCION UNICA: solo escribe comentarios si el usuario los solicita EXPLICITAMENTE en esa peticion concreta
- Si la sintaxis del lenguaje exige texto en cierta posicion (ej. shebang `#!/usr/bin/env python`), eso NO es comentario y SI va

### Respuesta corta
- Una "tarea" es cualquier accion que requiera usar herramientas (Write, Edit, Bash, Read, Glob, Grep, etc.)
- Una "pregunta" es cualquier solicitud de informacion o explicacion sin uso de herramientas
- Al completar una TAREA: responde UNICAMENTE la palabra `Listo` — sin resumen, sin lista de cambios, sin contexto
- Al responder una PREGUNTA: responde de forma concisa (maximo 2-3 oraciones), sin detalle excesivo
- Si el usuario pide explicitamente que expliques algo: es una pregunta, responde concisamente
- Esta regla aplica a TODAS las respuestas siguientes en la sesion

---
> Source: [bitcoinalexis/Lotokens-Skill-Claude-Code](https://github.com/bitcoinalexis/Lotokens-Skill-Claude-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
