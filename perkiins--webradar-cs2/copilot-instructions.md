## webradar-cs2

> Radar web para CS2. Fork de un proyecto base con mejoras propias.

# CS Radar (WebRadar_CS2)

Radar web para CS2. Fork de un proyecto base con mejoras propias.

## Stack
- **Backend C++**: `usermode/` — proceso que lee memoria del juego (Visual Studio, `.sln`)
- **Frontend**: `webapp/` — React + Vite + Tailwind CSS + WebSocket
- **Config**: `config.json` — configuración del radar

## Estructura
```
usermode/       # C++ - leer memoria CS2, emite datos via WebSocket
webapp/         # React - interfaz del radar en el navegador
install.bat     # instalación
start.bat       # arranque
```

## Comandos
```bash
# Frontend
cd webapp && npm install && npm run dev

# Backend C++
# Compilar con Visual Studio: usermode/cs2_webradar.sln
# O ejecutar start.bat directamente
```

## Flujo de trabajo (dos copias)
- **`E:\Maax\cs-radar`** → desarrollo y testeo. Aquí haces cambios y push.
- **`C:\Cositas\WebRadar_CS2`** → versión estable para ejecutar. Actualizar con:
  ```bash
  git pull
  ```

## Deploy / ejecución
No tiene servidor externo. Se ejecuta localmente mientras juegas a CS2.

---
> Source: [Perkiins/WebRadar_CS2](https://github.com/Perkiins/WebRadar_CS2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
