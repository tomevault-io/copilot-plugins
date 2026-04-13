## proyecto-comunicaci-n-de-datos-2025-aula-iot

> - name: "Terminal Command Safety"


# === Reglas del proyecto para Windsurf IDE ===
# Objetivo: Evitar errores recurrentes y garantizar coherencia entre backend y frontend.

rules:
  - name: "Terminal Command Safety"
    description: >
      No ejecutar comandos incompatibles con PowerShell o el entorno de Windows.
    match:
      - ".*"
    on_edit:
      - reject_commands:
          - "source"
          - "touch"
          - "ls"
          - "mv"
          - "cp"
          - "chmod"
          - "sudo"
          - "cat"
          - "echo >"
          - "export"
      - suggest_replacements:
          - from: "source"
            to: "."
          - from: "touch"
            to: "New-Item"
          - from: "ls"
            to: "dir"
    message: >
      ⚠️ Usa comandos compatibles con PowerShell (no Bash). Ejemplo: "dir" en lugar de "ls".

  - name: "Controlled File Editing"
    description: >
      No modificar ni eliminar archivos que no hayan sido explícitamente indicados.
    on_edit:
      - require_explicit_instruction: true
    message: >
      ⚠️ No edites archivos no solicitados. Realiza solo los cambios mencionados por el usuario.

  - name: "Backend-Frontend Connection Guard"
    description: >
      Evitar errores de conexión entre backend (Express.js + SQLite) y frontend (React).
    check:
      - ensure_ports_match: [frontend:3000, backend:5000]
      - ensure_api_url: "http://localhost:5000"
      - ensure_env_file_exists: ".env"
      - ensure_backend_running: true
    message: >
      ⚠️ Antes de probar la app, asegúrate de que el backend corre en el puerto 5000 
      y que el frontend apunta correctamente a http://localhost:5000.

  - name: "Commit Discipline"
    description: >
      No hacer cambios grandes sin confirmación del usuario.
    on_commit:
      - require_confirmation_if_changes_exceed: 10
    message: >
      ⚠️ Cambios mayores a 10 líneas deben ser aprobados antes del commit.

  - name: "Dependency Safety"
    description: >
      Evitar instalaciones innecesarias o globales de paquetes.
    on_edit:
      - block_commands:
          - "npm install -g"
          - "yarn global add"
      - require_approval_for_install: true
    message: >
      ⚠️ No instales dependencias globales. Usa dependencias locales del proyecto.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AgustinRuizDiaz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
