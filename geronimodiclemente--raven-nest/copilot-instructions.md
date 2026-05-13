## raven-nest

> 1. Asegurarse de que `package.json` tiene la versión correcta (campo `"version"`)

# Raven Nest — Instrucciones para Claude

## Hacer una release

1. Asegurarse de que `package.json` tiene la versión correcta (campo `"version"`)
2. Crear el tag y la release en GitHub:
   ```bash
   VERSION=$(node -p "require('./package.json').version")
   gh release create "v$VERSION" --title "v$VERSION" --notes "" --repo GeronimoDiClemente/raven-nest
   ```
3. Triggerear el workflow de build (buildea Windows y Mac en paralelo):
   ```bash
   gh workflow run "Build Windows & Mac" --repo GeronimoDiClemente/raven-nest
   ```
4. Verificar que los artifacts se subieron a la release:
   ```bash
   gh release view "v$VERSION" --repo GeronimoDiClemente/raven-nest
   ```

## Requisitos del usuario (dependencias externas)

- **Voice input**: requiere [openai-whisper](https://github.com/openai/whisper) instalado en Python.
  - Instalación: `pip install openai-whisper`
  - La app lo detecta automáticamente vía PATH, `~/Library/Python/3.x/bin/whisper`, o `python3 -m whisper`
  - Primera vez: descarga el modelo `tiny` (~150MB). Después es instantáneo.
  - Si no está instalado, el botón de micrófono simplemente no transcribe (sin crash).
  - En releases: documentar en README y en el onboarding que voice input requiere whisper.

## Stack
- Electron + Vite + React + TypeScript
- Terminal: xterm.js con PTY (node-pty)
- Build: electron-builder (NSIS para Windows, DMG para Mac)
- El repo usa git-crypt — los archivos sensibles están encriptados

## Estructura
- `src/hooks/useXterm.ts` — terminal xterm.js
- `src/` — renderer (React)
- `.github/workflows/build-windows.yml` — CI para Windows y Mac

## Seguridad — pendiente crítico

### GitHub token en Supabase (PENDIENTE)
La columna `github_token` de la tabla `profiles` en Supabase guarda el OAuth token en texto plano.
**Acción requerida antes de release pública:**
- Verificar en el dashboard de Supabase que la política RLS de `profiles` restrinja `SELECT` de `github_token` solo al propio usuario: `auth.uid() = id`.
- Confirmar que ningún rol de servicio (service_role) exponga la columna a otros usuarios.
- A largo plazo: migrar a tokens efímeros o encriptación a nivel de aplicación.

---
> Source: [GeronimoDiClemente/raven-nest](https://github.com/GeronimoDiClemente/raven-nest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
