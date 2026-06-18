## repo

> Este archivo es la referencia principal para que cualquier sesion de Claude entienda el

# AGENTS.md — Guia operativa para Claude

Este archivo es la referencia principal para que cualquier sesion de Claude entienda el
estado actual del proyecto, las convenciones vigentes y como ayudar correctamente.
**Leer completo antes de proponer cambios.**

---

## 1. Que es este proyecto

Sistema de captura de aceleracion en tiempo real con sensores ADXL335 conectados a una
ESP32 via ESP-NOW inalambrico, con un receptor USB que retransmite por serial.
La aplicacion de escritorio (GUI Python/Tkinter) recibe el stream serial, hace un
precheck de integridad dual, captura datos y genera un paquete de archivos estructurado.

**Stack vigente:**
- Firmware ESP32: C++ / PlatformIO (`firmware/single_node_calibration/`)
- GUI de captura live: Python 3.11 / Tkinter (`gui/`)
- Distribucion: ejecutable standalone Windows (`dist/ADXL335_Captura.exe`)
- Analisis historico: MATLAB (en proceso de migracion, ya no es ruta operativa live)

---

## 2. Estado operativo de sensores

| Sensor   | Estado       | Rol                                          |
|----------|--------------|----------------------------------------------|
| sensor_B | **Primario** | Fuente operativa principal. sensor_id = 1.   |
| sensor_A | Backup       | Fallback si sensor_B falla. sensor_id = 2.   |
| sensor_C | Referencia   | Historico/provisional. No usar en produccion.|
| sensor_D | Descartado   | No usar.                                     |

---

## 3. Estructura de archivos del repositorio

```
Repo/
├── gui/
│   ├── adxl_live_gui.py        # Punto de entrada de la aplicacion (GUI Tkinter)
│   └── adxl_live_core.py       # Logica de captura, serial, procesamiento y guardado
│
├── firmware/
│   ├── single_node_calibration/    # Firmware operativo actual (PlatformIO)
│   └── dual_node_espnow/           # Arquitectura futura ESP-NOW (Fase 14B/14C)
│
├── data/
│   ├── raw/sensor_B_live/          # CSVs crudos de sesiones live (NO modificar)
│   └── processed/                  # CSVs procesados + JSONs de sesion
│
├── reports/
│   ├── analysis_outputs/           # _summary.txt y _precheck.txt por sesion
│   └── change_log.md               # Registro tecnico de todos los cambios
│
├── live_session_hub/
│   ├── sensorB_live_gui_session.ps1    # Launcher oficial de la GUI
│   └── sensorB_live_prompt_session.m  # Launcher MATLAB (legado, no usar)
│
├── scripts/
│   ├── install_gui_requirements.ps1   # Instala pyserial en el venv
│   ├── run_sensorB_live_session.ps1   # Lanza GUI con parametros
│   └── run_sensorB_operational_capture.ps1
│
├── config/
│   └── adxl335_module_template.json   # Plantilla de configuracion de modulo
│
├── matlab/                            # Analisis historico (referencia, no operativo)
├── docs/                              # Guia maestra LaTeX + PDF
├── hardware/                          # Pinout y registro de sensores
├── handoff/                           # Exports de handoff por rol (Fase 14C.1)
│
├── adxl_captura.spec          # Spec de PyInstaller para compilar el .exe
├── build_exe.ps1              # Script: compila .exe y genera ZIP de distribucion
├── build_manual_pdf.py        # Script: genera ADXL335_Captura_Manual.pdf
├── ADXL335_Captura_Manual.pdf # Manual de usuario (generado por build_manual_pdf.py)
├── requirements-gui.txt       # Dependencias Python de la GUI (solo pyserial==3.5)
├── AGENTS.md                  # Este archivo
└── README.md                  # Documentacion principal del proyecto
```

Carpetas generadas automaticamente (no commitear):
```
.venv/          # Entorno virtual Python local
dist/           # Ejecutable compilado y ZIP de distribucion
build_work/     # Artefactos intermedios de PyInstaller
```

---

## 4. La aplicacion GUI — como funciona

### Punto de entrada
```
gui/adxl_live_gui.py
```

### Dependencias
- `pyserial==3.5` (unica dependencia externa; ver `requirements-gui.txt`)
- Python stdlib: `tkinter`, `threading`, `csv`, `json`, `pathlib`, etc.

### Flujo de una sesion
1. **Autodeteccion de puerto**: sondea puertos COM buscando el stream del firmware.
2. **Precheck dual** (~10 s): verifica integridad de ambos sensores antes de capturar.
3. **Captura principal** (10-90 s): graba datos, actualiza graficas en tiempo real.
4. **Guardado**: genera los 5 archivos de salida.

### Deteccion del directorio de datos (IMPORTANTE para el exe)
```python
# gui/adxl_live_gui.py — funcion _resolve_repo_root()
if getattr(sys, "frozen", False):
    # Ejecutando como .exe compilado con PyInstaller
    return Path(sys.executable).resolve().parent
else:
    # Ejecutando en desarrollo desde el repo
    return Path(__file__).resolve().parents[1]
```
Esto garantiza que los datos se guarden junto al .exe en distribucion y junto a la
raiz del repo en desarrollo. No cambiar este patron.

---

## 5. Archivos de salida — formato y campos

Cada sesion exitosa genera estos archivos con nombre comun:
```
{prefijo}_{sesion}_{AAAAMMDD}_{HHMMSS}_{tipo}.{ext}
```

### _raw.csv
Lectura directa del firmware. Campos:
```
sensor_id, seq, t_us, wall_s, raw_x, raw_y, raw_z, mv_x, mv_y, mv_z
```
- `sensor_id`: 1=sensor_B (principal), 2=sensor_A (secundario)
- `seq`: numero de secuencia del firmware (saltos = paquetes perdidos)
- `t_us`: microsegundos del reloj interno de la ESP32
- `wall_s`: segundos del reloj de la PC desde la primera muestra
- `raw_x/y/z`: cuentas ADC (0-4095). Saturacion: <=5 o >=4090
- `mv_x/y/z`: tension en mV = raw * (3300 / 4096)

### _processed.csv
Igual que raw + 4 columnas adicionales:
```
..., gx_est, gy_est, gz_est, g_norm_est
```
- `gx/gy/gz_est`: aceleracion en g = (mv - bias) / sensibilidad (300 mV/g nominal)
- `g_norm_est`: norma euclidiana sqrt(gx^2+gy^2+gz^2). En reposo ~1.0 g

### _session.json
Metadatos completos en JSON: configuracion, puerto, precheck, integridad, rutas.

### _precheck.txt / _summary.txt
Texto plano clave:valor. El precheck describe la ventana de calentamiento;
el summary describe la captura principal. Codigos de estado:
- `pass`: datos validos
- `suspect`: usar con precaucion
- `fail`: datos no fiables

Codigos de razon mas comunes: `dual_integrity_ok`, `few_samples`,
`packet_loss_excessive`, `adc_saturation`, `g_norm_implausible`,
`dead_axis_or_disconnected`, `counter_reset_detected`, `sensor_missing`.

---

## 6. Como compilar el ejecutable

Desde la raiz del repo:
```powershell
.\build_exe.ps1
```
Esto instala PyInstaller si falta, compila `dist\ADXL335_Captura.exe` y genera
`dist\ADXL335_Captura_dist.zip` listo para distribuir.

Tambien se puede compilar directamente:
```powershell
.venv\Scripts\pyinstaller.exe adxl_captura.spec --distpath dist --workpath build_work --noconfirm
```

El `.exe` es standalone: no requiere Python en el destino.
Al primer uso crea automaticamente `data\raw\sensor_B_live\`, `data\processed\`
y `reports\analysis_outputs\` junto al ejecutable.

---

## 7. Como generar el manual PDF

```powershell
.venv\Scripts\python.exe build_manual_pdf.py
```
Requiere `fpdf2` en el venv (se instala con `pip install fpdf2`).
Genera `ADXL335_Captura_Manual.pdf` en la raiz del repo.

---

## 8. Como lanzar la GUI en desarrollo

Instalar dependencias (una sola vez):
```powershell
.\scripts\install_gui_requirements.ps1
```

Lanzar la GUI:
```powershell
.\live_session_hub\sensorB_live_gui_session.ps1
```

O con parametros especificos:
```powershell
.venv\Scripts\python.exe gui\adxl_live_gui.py --duration-s 30 --session-name ensayo1
```

---

## 9. Como compilar y cargar el firmware

Build:
```powershell
$env:PLATFORMIO_EXE="$env:USERPROFILE\.platformio\penv\Scripts\pio.exe"
& $env:PLATFORMIO_EXE run -d .\firmware\single_node_calibration
```

Upload (puede requerir mantener BOOT presionado):
```powershell
& $env:PLATFORMIO_EXE run -d .\firmware\single_node_calibration -t upload
```

Pinout ADXL335 -> ESP32:
- VCC -> 3V3
- GND -> GND
- X-OUT -> GPIO32
- Y-OUT -> GPIO33
- Z-OUT -> GPIO34
- ST -> GPIO23 (solo diagnostico)

---

## 10. Reglas de datos e integridad

1. `data/raw/` es evidencia primaria. **Nunca sobrescribir.**
2. El procesamiento va en `data/processed/`.
3. Todo analisis genera reporte en `reports/`.
4. La calibracion es por sensor individual, no por nodo.
5. No maquillar por software una inconsistencia fisica del sensor.
6. Todo cambio relevante se registra en `reports/change_log.md`.

### Criterios de calidad de una captura
Una corrida se considera util si cumple:
- `SEQ_JUMPS = 0` (o muy pocos)
- `95 <= FREQ_HZ <= 105` (por sensor individual)
- `SAT_PCT_ANY_AXIS <= 1.0 %`
- `g_norm_median` entre 0.85 y 1.15 (en reposo)

### Fallback operativo
Si sensor_B falla sanidad en dos corridas cortas consecutivas → usar sensor_A
con el mismo flujo. Registrar incidente en `reports/analysis_outputs/`.

---

## 11. Reglas de ejecucion para Claude

- La ruta operativa live es la GUI Python. MATLAB es solo referencia historica.
- Al modificar `adxl_live_gui.py` o `adxl_live_core.py`, no alterar la logica
  de guardado ni el patron de nombres de archivo sin actualizar el change_log.
- Al proponer cambios al exe, verificar que `_resolve_repo_root()` siga correcto.
- Si se agregan dependencias Python, actualizar `requirements-gui.txt` Y
  los `hiddenimports` en `adxl_captura.spec`.
- Los archivos en `dist/` y `build_work/` estan en `.gitignore` y no se commitean.
- El `build_manual_pdf.py` genera el PDF; el PDF resultante SI se commitea.

---

## 12. Checklist rapido cuando algo no corre

1. Confirmar puerto COM correcto (Administrador de dispositivos).
2. Cerrar cualquier otro programa que use el puerto serial (Arduino IDE, etc.).
3. Verificar que el firmware este cargado en la ESP32.
4. Correr una sesion corta de 10-20 s.
5. Revisar `_precheck.txt` y `_summary.txt` en `reports/analysis_outputs/`.
6. Si falla `sensor_missing`: revisar conexiones fisicas del ADXL335.
7. Si falla `packet_loss_excessive`: cambiar cable USB, cerrar otros monitores.

---
> Source: [hujjika1099-glitch/Repo](https://github.com/hujjika1099-glitch/Repo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
