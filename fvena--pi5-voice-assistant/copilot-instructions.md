## pi5-voice-assistant

> Servidor FastAPI que corre en Raspberry Pi 5 (8GB RAM, NVMe 256GB) y expone

# Pi 5 Voice Assistant — Servidor de voz local

## Qué es este proyecto

Servidor FastAPI que corre en Raspberry Pi 5 (8GB RAM, NVMe 256GB) y expone
endpoints HTTP para un robot con ESP32 y un asistente de voz. 100% offline,
sin cloud. Stack: faster-whisper (ASR) + llama-cpp-python (LLM, Qwen 3 1.7B
Q4_K_M) + Piper TTS (español).

## Referencia principal

@TUTORIAL.md contiene el tutorial completo con todo el código fuente y las
justificaciones técnicas de cada decisión. DEBES leerlo antes de generar
cualquier archivo.

---

## Estructura objetivo del repositorio

```
pi5-voice-assistant/
├── CLAUDE.md                    # Este archivo (no se despliega en la Pi)
├── TUTORIAL.md                  # Tutorial de referencia (no se despliega)
├── README.md                    # Documentación para clonar y arrancar
├── Makefile                     # make setup / run / test / service / clean
├── requirements.txt             # Deps Python con versiones mínimas
├── .env.example                 # Template de configuración
├── .gitignore
├── app/
│   ├── __init__.py              # Vacío
│   ├── config.py                # Settings desde .env (incluye PIPELINES)
│   ├── pipeline.py              # Clase base Pipeline
│   ├── conversation.py          # Historial con persistencia JSON en NVMe
│   ├── engine.py                # Motor: ASR + LLM + TTS (carga condicional)
│   └── main.py                  # FastAPI app con carga dinámica de pipelines
├── pipelines/
│   ├── README.md                # Tutorial para crear pipelines personalizados
│   ├── robot/
│   │   ├── pipeline.py          # RobotPipeline (keyword router + LLM fallback)
│   │   └── keyword_router.py    # Router regex para comandos robot (~1ms)
│   └── assistant/
│       └── pipeline.py          # AssistantPipeline (streaming LLM→TTS)
├── scripts/
│   ├── system-optimize.sh       # Optimización SO de la Pi 5
│   ├── download-models.sh       # Descarga modelos LLM y TTS
│   └── install-service.sh       # Instala systemd unit
└── tests/
    ├── test_keyword_router.py   # Tests unitarios pytest (60+ casos)
    └── test_endpoints.sh        # Smoke tests con curl
```

---

## Reglas para generar los archivos de app/

IMPORTANTE: La base del código está en @TUTORIAL.md. Los módulos de `app/` han sido
refactorizados para soportar pipelines pluggables. La arquitectura actual es:

| Archivo | Descripción | Origen |
|---------|-------------|--------|
| `app/config.py` | Clase `Settings` con `PIPELINES` (sin prompts, esos viven en los pipelines) | Modificado desde sección 7.2 |
| `app/pipeline.py` | Clase base `Pipeline` con `register_routes()` abstracto | Nuevo |
| `app/conversation.py` | `ConversationManager` con `_save_to_disk` | Sección 7.4 sin cambios |
| `app/engine.py` | Motor con `load_all(requires_tts)` — carga condicional de TTS | Modificado desde sección 7.5 |
| `app/main.py` | Carga dinámica de pipelines via `importlib`, `/health` endpoint | Reescrito (ya no tiene endpoints de robot/assistant) |

### Archivos de pipelines

| Archivo | Descripción | Origen |
|---------|-------------|--------|
| `pipelines/robot/pipeline.py` | `RobotPipeline`: endpoints `/robot/command` y `/robot/reset` | Extraído de sección 7.6 |
| `pipelines/robot/keyword_router.py` | Router regex para comandos robot (~1ms) | Sección 7.3 sin cambios (movido desde `app/`) |
| `pipelines/assistant/pipeline.py` | `AssistantPipeline`: 4 endpoints de asistente | Extraído de sección 7.6 |

- `app/__init__.py` y `pipelines/*/__init__.py` son archivos vacíos
- `conversation.py` y `keyword_router.py` mantienen el código original del tutorial
- NO simplifiques, NO refactorices, NO "mejores" el código existente

---

## Reglas para generar requirements.txt

Extraer de la Parte 6 del tutorial. Contenido exacto:

```
fastapi>=0.104.0
uvicorn[standard]>=0.24.0
python-multipart>=0.0.6
python-dotenv>=1.0.0
llama-cpp-python>=0.3.0
faster-whisper>=1.2.0
piper-tts>=1.4.0
huggingface-hub>=0.20.0
numpy>=1.24.0
```

---

## Reglas para generar .env.example

Extraer de la sección 7.1 del tutorial. Incluir TODOS los valores por defecto
como comentarios pero dejar los valores activos. Incluir `HISTORY_FILE`.
Incluir `PIPELINES=` (vacío por defecto) con comentario explicando el formato.
Los system prompts (`ROBOT_SYSTEM_PROMPT`, `ASSISTANT_SYSTEM_PROMPT`) van
comentados como overrides opcionales — cada pipeline trae su prompt por defecto.

---

## Especificación de scripts/system-optimize.sh

Script bash que optimiza Raspberry Pi OS para inferencia LLM.
DEBE ser idempotente (ejecutable múltiples veces sin romper nada).
DEBE requerir sudo y comprobar que estamos en una Pi 5.

### Paso 1: CPU governor → performance

```bash
# Cambio inmediato
echo performance | sudo tee /sys/devices/system/cpu/cpufreq/policy0/scaling_governor
# Crear servicio systemd para persistencia: /etc/systemd/system/cpu-governor.service
# Contenido del unit file:
#   [Unit]
#   Description=Set CPU governor to performance
#   After=multi-user.target
#   [Service]
#   Type=oneshot
#   ExecStart=/bin/sh -c "echo performance > /sys/devices/system/cpu/cpufreq/policy0/scaling_governor"
#   [Install]
#   WantedBy=multi-user.target
# systemctl daemon-reload + enable + start
```

### Paso 2: Swap — zram + NVMe backup

```bash
# Instalar zram-tools
sudo apt install -y zram-tools
# Configurar /etc/default/zramswap:
#   ALGO=lz4
#   SIZE=2048
#   PRIORITY=100
# Crear swap NVMe solo si /swapfile no existe:
#   sudo fallocate -l 2G /swapfile
#   sudo chmod 600 /swapfile
#   sudo mkswap /swapfile
#   sudo swapon /swapfile
#   Añadir a /etc/fstab: '/swapfile none swap sw,pri=10 0 0'
#   (comprobar que la línea no existe ya en fstab antes de añadir)
```

### Paso 3: Parámetros kernel para LLM

```bash
# Añadir a /etc/sysctl.conf (solo si no existen ya):
#   vm.swappiness=100
#   vm.vfs_cache_pressure=500
#   vm.page-cluster=0
#   vm.dirty_background_ratio=1
#   vm.dirty_ratio=50
# Aplicar: sudo sysctl -p
```

### Paso 4: Desactivar servicios innecesarios

```bash
sudo systemctl disable bluetooth hciuart avahi-daemon triggerhappy 2>/dev/null || true
sudo apt purge -y modemmanager 2>/dev/null || true
sudo systemctl disable NetworkManager-wait-online.service 2>/dev/null || true
```

### Paso 5: Optimización de arranque

```bash
# Añadir initial_turbo=30 a /boot/firmware/config.txt (si no existe)
# Añadir quiet fastboot noatime a /boot/firmware/cmdline.txt (si no existen)
```

### Paso 6: Verificación final

```bash
# Mostrar:
#   - Governor actual
#   - Swap activo
#   - Temperatura (vcgencmd measure_temp)
#   - Throttling (vcgencmd get_throttled)
#   - Mensaje: "Reinicia con sudo reboot para aplicar todos los cambios"
```

---

## Especificación de scripts/download-models.sh

Script bash que descarga los modelos de ML necesarios.
DEBE ser idempotente (no re-descargar si ya existen).
DEBE funcionar sin venv activo (usa pip del sistema solo para huggingface-hub).

### Modelo LLM: Qwen 3 1.7B Q4_K_M

```bash
# Crear directorio: mkdir -p models/
# Si models/Qwen_Qwen3-1.7B-Q4_K_M.gguf NO existe:
#   Activar venv: source venv/bin/activate
#   pip install huggingface-hub (si no está instalado)
#   huggingface-cli download bartowski/Qwen_Qwen3-1.7B-GGUF \
#       --include "Qwen_Qwen3-1.7B-Q4_K_M.gguf" \
#       --local-dir ./models
#   Verificar que el archivo existe y pesa ~1.3 GB
# Si ya existe: mostrar "Modelo LLM ya descargado, saltando"
```

### Voz Piper: es_ES-davefx-medium

```bash
# Crear directorio: mkdir -p voices/
# Si voices/es_ES-davefx-medium.onnx NO existe:
#   wget con progreso:
#     URL .onnx:      https://huggingface.co/rhasspy/piper-voices/resolve/v1.0.0/es/es_ES/davefx/medium/es_ES-davefx-medium.onnx?download=true
#     URL .onnx.json: https://huggingface.co/rhasspy/piper-voices/resolve/v1.0.0/es/es_ES/davefx/medium/es_ES-davefx-medium.onnx.json?download=true
#   Destino: voices/es_ES-davefx-medium.onnx y voices/es_ES-davefx-medium.onnx.json
# Si ya existe: mostrar "Voz Piper ya descargada, saltando"
```

### Verificación final

```bash
# Listar models/ y voices/ con tamaños
# Confirmar que ambos archivos existen
```

---

## Especificación de scripts/install-service.sh

Script bash que instala el servidor como servicio systemd.
DEBE detectar automáticamente usuario y rutas.
DEBE pedir confirmación si el service ya existe.

### Paso 1: Detectar contexto

```bash
CURRENT_USER=$(whoami)
PROJECT_DIR=$(cd "$(dirname "$0")/.." && pwd)
VENV_PATH="$PROJECT_DIR/venv"
# Verificar que venv existe, si no → error
# Verificar que app/main.py existe, si no → error
```

### Paso 2: Generar unit file

```bash
# Escribir /etc/systemd/system/voice-assistant.service con:
# [Unit]
# Description=Unified Voice Assistant Server
# After=network-online.target
# Wants=network-online.target
#
# [Service]
# Type=simple
# User=$CURRENT_USER
# Group=$CURRENT_USER
# WorkingDirectory=$PROJECT_DIR
# Environment="PATH=$VENV_PATH/bin:/usr/bin"
# Environment="PYTHONUNBUFFERED=1"
# Environment="OMP_NUM_THREADS=3"
# Environment="OPENBLAS_NUM_THREADS=3"
# Environment="GOMP_CPU_AFFINITY=0-2"
# ExecStart=$VENV_PATH/bin/uvicorn app.main:app \
#     --host 0.0.0.0 --port 8080 --workers 1 --log-level info
# Restart=always
# RestartSec=5
# StartLimitIntervalSec=60
# StartLimitBurst=5
# Nice=-5
# CPUAffinity=0 1 2 3
# MemoryMax=6G
# MemoryHigh=5G
# StandardOutput=journal
# StandardError=journal
# SyslogIdentifier=voice-assistant
# PrivateTmp=true
# TimeoutStopSec=30
#
# [Install]
# WantedBy=multi-user.target
```

### Paso 3: Activar

```bash
sudo systemctl daemon-reload
sudo systemctl enable voice-assistant.service
sudo systemctl start voice-assistant.service
# Esperar 5 segundos
# Mostrar status
# Intentar curl http://localhost:8080/health
```

---

## Especificación del Makefile

```makefile
.PHONY: setup run test service clean

# Ejecuta los 3 scripts de setup en orden
setup:
 bash scripts/system-optimize.sh
 bash scripts/download-models.sh
 bash scripts/install-service.sh

# Arranca el servidor en foreground (para desarrollo)
run:
 OMP_NUM_THREADS=3 OPENBLAS_NUM_THREADS=3 \
 ./venv/bin/uvicorn app.main:app --host 0.0.0.0 --port 8080 --workers 1

# Ejecuta tests unitarios + smoke tests
test:
 ./venv/bin/pytest tests/test_keyword_router.py -v
 bash tests/test_endpoints.sh

# Solo instala/reinicia el servicio systemd
service:
 bash scripts/install-service.sh

# Limpia archivos generados (NO borra modelos)
clean:
 rm -f conversation_history.json
 rm -f test-*.wav
 find . -type d -name __pycache__ -exec rm -rf {} + 2>/dev/null || true
```

---

## Especificación de tests/test_keyword_router.py

Tests con pytest para el keyword router. Sin dependencias externas.
Basados en la batería de pruebas reales del proyecto, con 60+ casos.

```python
# Debe importar: from pipelines.robot.keyword_router import route_command
#
# Cada test llama route_command(text) y verifica:
#   - result is not None cuando debe matchear (o None cuando no)
#   - result.actions[i].action == acción esperada
#   - result.actions[i].params contiene los params esperados
#   - Para compound commands: len(result.actions) == N esperado
#
# CATEGORÍAS DE TESTS:
#
# 1. STOP (10 casos):
#    "para" → stop | "detente" → stop | "quieto" → stop
#    "no te muevas" → stop | "frena" → stop | "stop" → stop
#    "basta" → stop | "alto" → stop
#    "oye robot para" → stop (con vocativo)
#    "por favor detente" → stop (con cortesía)
#
# 2. MOVE FORWARD (9 casos):
#    "avanza" → move forward distance=1 (default)
#    "avanza dos metros" → move forward distance=2
#    "avanza 3 metros" → move forward distance=3
#    "ve hacia adelante" → move forward
#    "camina" → move forward
#    "muévete" → move forward
#    "anda para delante" → move forward
#    "adelante" → move forward
#    "avanza medio metro" → move forward distance=0.5
#
# 3. MOVE BACKWARD (6 casos):
#    "retrocede" → move backward distance=1
#    "retrocede dos metros" → move backward distance=2
#    "ve hacia atrás" → move backward
#    "marcha atrás" → move backward
#    "atrás" → move backward
#    "retrocede medio metro" → move backward distance=0.5
#
# 4. TURN (9 casos):
#    "gira a la derecha" → turn right angle=90
#    "gira a la izquierda" → turn left angle=90
#    "gira 45 grados a la derecha" → turn right angle=45
#    "gira 180 grados" → turn angle=180
#    "da la vuelta" → turn (any direction)
#    "da una vuelta completa" → turn angle=360
#    "da media vuelta" → turn angle=180
#    "tuerce a la izquierda" → turn left
#    "rota a la derecha" → turn right
#
# 5. SLEEP/WAKE (7 casos):
#    "duérmete" → sleep | "a dormir" → sleep
#    "descansa" → sleep | "modo reposo" → sleep
#    "despierta" → wake | "arriba" → wake | "actívate" → wake
#
# 6. DANCE (2 casos):
#    "baila" → dance | "ponte a bailar" → dance
#
# 7. COMPOUND (5 casos):
#    "avanza dos metros y gira a la derecha"
#      → [move forward 2, turn right 90]
#    "avanza un metro y luego gira a la izquierda"
#      → [move forward 1, turn left 90]
#    "retrocede medio metro y baila"
#      → [move backward 0.5, dance]
#    "gira a la derecha y avanza tres metros"
#      → [turn right 90, move forward 3]
#    "avanza, gira a la derecha y avanza otra vez"
#      → [move forward, turn right, move forward]
#
# 8. NOISE — vocativos y cortesía (4 casos):
#    "oye robot avanza" → move forward
#    "por favor gira a la derecha" → turn right
#    "oye robot por favor avanza dos metros" → move forward 2
#    "venga anda para delante" → move forward
#
# 9. NO MATCH → None (cae al LLM) (4 casos):
#    "canta una canción" → None
#    "cuál es la capital de Francia" → None
#    "llama a mi madre" → None
#    "qué hora es" → None
#
# NOTA: No testear errores STT aquí (esos van contra el LLM).
# El keyword router es estricto: si no matchea, devuelve None
# y el LLM se encarga.
```

---

## Especificación de tests/test_endpoints.sh

Script bash que prueba los endpoints con curl. Asume servidor en localhost:8080.

```bash
# 1. GET /health → verificar que devuelve "status":"ok"
# 2. Generar WAV de prueba con piper: echo "avanza" | piper → /tmp/test.wav
#    POST /robot/command con ese WAV → verificar JSON con "actions" array
#    Verificar que actions[0].action es "move"
# 3. Generar WAV: echo "hola" | piper → /tmp/test2.wav
#    POST /assistant/chat/text con ese WAV → verificar JSON con "response"
# 4. POST /robot/reset → verificar "history_cleared"
# 5. POST /assistant/reset → verificar "history_cleared"
# Mostrar PASS/FAIL por cada test
# Exit code 0 si todos pasan, 1 si alguno falla
```

---

## Especificación de README.md

En español. Orientado a alguien que clona el repo y quiere arrancar.

Secciones:

1. **Título + descripción** de una línea: qué es y para qué sirve
2. **Arquitectura** — diagrama ASCII del tutorial (ESP32 → Pi 5 → respuesta)
3. **Requisitos** — Pi 5 8GB, NVMe, Pi OS Lite 64-bit, refrigeración activa
4. **Instalación rápida** — clonar, crear venv, pip install, make setup
5. **Uso** — make run, endpoints disponibles (tabla del tutorial)
6. **Endpoints** — tabla con los 7 endpoints, método, entrada, salida
7. **Configuración** — explicar .env con las variables principales
8. **Estructura del proyecto** — árbol de archivos con descripción de cada uno
9. **Rendimiento esperado** — tabla de latencias del tutorial
10. **Licencia** — MIT

---

## Especificación de .gitignore

```
venv/
models/
voices/
__pycache__/
*.pyc
.env
conversation_history.json
test-*.wav
*.egg-info/
```

---

## IMPORTANTE — Formato de respuesta del robot

Tanto el keyword router como el LLM devuelven el MISMO formato:

```json
{"actions": [{"action": "tipo", "params": {...}}]}
```

El ESP32 siempre itera `doc["actions"]` — no hay dos formatos distintos.

Acciones disponibles:

| Acción | Parámetros | Defaults |
|--------|-----------|----------|
| `move` | direction (forward/backward), distance (metros) | distance=1, direction=forward |
| `turn` | direction (left/right), angle (grados) | angle=90, direction=right |
| `stop` | {} | — |
| `sleep` | {} | — |
| `wake` | {} | — |
| `dance` | {} | — |
| `grab` | {} | — |
| `release` | {} | — |
| `look_up` | angle (grados) | angle=30 |
| `look_down` | angle (grados) | angle=30 |
| `unknown` | original (texto del usuario) | — |

---

## IMPORTANTE — Reglas globales

- NO generes archivos uno a uno preguntándome entre cada uno
- Genera TODOS los archivos del proyecto en una sola pasada
- Lee @TUTORIAL.md COMPLETO antes de empezar (referencia histórica del código base)
- `conversation.py` y `keyword_router.py` mantienen código del tutorial sin cambios
- `config.py`, `engine.py` y `main.py` han sido modificados para pipelines pluggables
- Los endpoints de robot y assistant viven en `pipelines/`, no en `app/main.py`
- Los scripts siguen las especificaciones de ESTE archivo, no del tutorial
- Todos los scripts deben empezar con `#!/usr/bin/env bash` y `set -euo pipefail`
- Todos los scripts deben imprimir lo que hacen con `echo ">>> Paso N: descripción"`
- No uses colores ANSI en los scripts (complicación innecesaria)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fvena)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fvena)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
