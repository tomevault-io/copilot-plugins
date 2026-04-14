## termostatoac-carrier

> Este proyecto tiene como objetivo analizar los datos emitidos por el termostato **Carrier CRC2-NTC** (un modelo obsoleto) mediante comunicación RS485, con el fin de entender su funcionamiento interno y eventualmente reemplazarlo con un dispositivo compatible que permita programación avanzada, acceso remoto y una interfaz moderna.

# TermostatoAC-Carrier - Guía para Agentes de Código

## Visión General del Proyecto

Este proyecto tiene como objetivo analizar los datos emitidos por el termostato **Carrier CRC2-NTC** (un modelo obsoleto) mediante comunicación RS485, con el fin de entender su funcionamiento interno y eventualmente reemplazarlo con un dispositivo compatible que permita programación avanzada, acceso remoto y una interfaz moderna.

El proyecto combina:
- **Comunicación Serial**: Lectura y escritura de datos del termostato vía USB-RS485
- **Machine Learning**: Entrenamiento de modelos neuronales para predecir el estado del termostato
- **Análisis de Protocolos**: Ingeniería inversa del protocolo CCN (Carrier Comfort Network)

---

## Stack Tecnológico

- **Lenguaje**: Python 3
- **Librerías Principales** (ver `requirements.txt`):
  - `serial` (pyserial) - Comunicación con el puerto serie
  - `tensorflow` / `keras` - Entrenamiento de modelos de machine learning
  - `numpy` - Procesamiento numérico
  - `matplotlib` - Visualización de gráficos
  - `optparse` - Interfaz de línea de comandos

---

## Estructura del Proyecto

```
TermostatoAC-Carrier/
├── curryrtool.py              # Entry point principal de la aplicación (Python)
├── curryrtool                 # Wrapper bash para ejecutar curryrtool.py
├── curryrtoolslib/            # Paquete Python con la lógica completa
│   ├── interfaces/            # Interfaces de usuario (CLI)
│   │   ├── common_interface.py      # Clase base para interfaces
│   │   ├── devices_interface.py     # Comunicación serial (read/write/repeat)
│   │   └── trainning_interface.py   # Entrenamiento y predicción ML
│   ├── utils/                 # Utilidades
│   │   ├── ia.py              # Funciones TensorFlow/Keras
│   │   ├── trama.py           # Gestión de tramas (85/88/91/94 bytes)
│   │   ├── graphics.py        # Visualización de datos
│   │   └── config.py          # Gestión de configuración
│   ├── loader/                # Bootstrap y entry points
│   │   ├── main.py            # Inicialización de la aplicación
│   │   └── entry_points.py    # Router de comandos
│   └── __init__.py            # Configuración global
├── dumps/                     # Datos capturados del termostato (no en git)
│   ├── *.bin                  # Tramas en formato binario
│   ├── *.info                 # Metadatos JSON de las tramas
│   └── *.json                 # Listas de capturas
├── models/                    # Modelos entrenados (Keras .keras) (no en git)
├── images/                    # Documentación visual (fotos, diagramas)
├── docs/                      # Documentación adicional de análisis
├── CH341SER/                  # Drivers para el adaptador USB-RS485 (no en git)
├── requirements.txt           # Dependencias del proyecto
├── README.md                  # Documentación en inglés
├── README_es.md               # Documentación en español (principal)
├── CCN_PROTOCOL_ANALYSIS.md   # Análisis técnico del protocolo CCN
├── CHECKLIST.md               # Guía de ingeniería inversa
└── .gitignore                 # Excluye: *.zip, CH341SER, dumps, models, __pycache__
```

---

## Instalación y Configuración

### Instalación de Dependencias

```bash
pip install -r requirements.txt
```

Contenido actual de `requirements.txt`:
- `serial` (pyserial)
- `tensorflow`
- `types-tensorflow`

### Permisos de Dispositivo (Linux)

Para acceder al puerto USB sin sudo:
```bash
sudo usermod -a -G dialout $USER
# Requiere cerrar sesión y volver a iniciar
```

### Configuración

El proyecto utiliza un archivo de configuración en:
```
~/.config/curryrtools/config.ini
```

Gestión vía `curryrtoolslib/utils/config.py` (clase `ConfigurationManager`).

Valores por defecto:
```ini
[CONFIG]
dumpsfolder = dumps
modelofolder = models
port = /dev/ttyUSB0
baudrate = 2400
timeout = 0.1
posiciones = 000000
savefile = s
repeticionesenvio = 30
portsalida = /dev/ttyUSB1
neuronasfirstlayer = 79
```

---

## Arquitectura del Código

### Flujo de Ejecución

```
curryrtool.py
    └── startup() [loader/main.py]
        └── entry_points.main_devices() / entry_points.main_trainning()
            ├── DeviceInterface [interfaces/devices_interface.py]
            │   ├── read (captura datos)
            │   ├── write (envía tramas)
            │   ├── repeat (modo repetidor)
            │   └── menu (interactivo)
            └── TrainningInterface [interfaces/trainning_interface.py]
                ├── entrenar (entrena modelo)
                ├── predecir (predice estado)
                └── pesos (muestra pesos del modelo)
```

### Módulos Principales

#### 1. curryrtool.py (Entry Point)
**Propósito**: Punto de entrada principal.

```bash
python3 curryrtool.py <comando> <acción> [opciones]
```

Comandos disponibles:
- `device` - Operaciones de comunicación serial
- `trainning` - Entrenamiento y predicción ML

#### 2. curryrtool (Wrapper Bash)
Script bash ejecutable que facilita la ejecución:
```bash
./curryrtool device read -p /dev/ttyUSB0
```

#### 3. devices_interface.py
**Propósito**: Interfaz completa de comunicación con el termostato.

**Acciones disponibles**:

| Acción | Descripción | Parámetros requeridos |
|--------|-------------|----------------------|
| `menu` | Menú interactivo CLI | - |
| `read` | Lee datos del puerto serie | `-p`, `-b`, `-t`, `-o` |
| `write` | Envía trama al termostato | `-p`, `-b`, `-t`, `-f`, `-i` |
| `repeat` | Modo repetidor (sniffer) | `-p`, `-r`, `-b`, `-t` |

**Ejemplos**:
```bash
# Leer datos y guardar en archivo
python3 curryrtool.py device read -p /dev/ttyUSB0 -b 2400 -t 0.1 -o "0,0,1,0,0,1"

# Enviar trama previamente capturada
python3 curryrtool.py device write -p /dev/ttyUSB0 -b 2400 -t 0.1 \
    -f dumps/trama.bin -i 5

# Modo repetidor (lee de un puerto y reenvía a otro)
python3 curryrtool.py device repeat -p /dev/ttyUSB0 -r /dev/ttyUSB1 -b 2400 -t 0.1
```

**Parámetros de línea de comandos**:
- `-p, --port`: Puerto serial (default: `/dev/ttyUSB0`)
- `-b, --baudrate`: Velocidad en baudios (default: `2400`)
- `-t, --timeout`: Timeout de lectura en segundos (default: `0.1`)
- `-o, --posiciones`: Estado de los controles, formato "FAN1,FAN2,FAN3,COLD,HOT,DRY"
- `-f, --tramafile`: Archivo de trama a enviar (formato .bin)
- `-r, --portsalida`: Puerto de salida para modo repetidor
- `-i, --interval`: Tiempo entre envíos en segundos
- `-s, --savefile`: Guardar datos capturados (default: True)

#### 4. trainning_interface.py
**Propósito**: Interfaz de entrenamiento y predicción con Machine Learning.

**Acciones disponibles**:

| Acción | Descripción |
|--------|-------------|
| `entrenar` | Entrena un modelo neuronal con datos capturados |
| `predecir` | Predice estado a partir de una trama |
| `pesos` | Muestra los pesos de las capas del modelo |

**Ejemplos**:
```bash
# Entrenar nuevo modelo
python3 curryrtool.py trainning entrenar -l 50,50,50 -s

# Entrenar modelo con capas específicas
python3 curryrtool.py trainning entrenar -l 8*100 -s

# Entrenar modelo existente
python3 curryrtool.py trainning entrenar -m models/mi_modelo.keras -e 5000 -s

# Realizar predicción
python3 curryrtool.py trainning predecir -m modelo.keras -f dumps/trama.bin
```

**Parámetros de línea de comandos**:
- `-d, --dumpsfolder`: Carpeta de datos de entrenamiento
- `-m, --model_file`: Archivo del modelo Keras
- `-f, --file_name`: Archivo con trama a predecir
- `-l, --layers`: Configuración de capas (ej: "50,50,50" o "8*100")
- `-e, --epocs`: Número de épocas de entrenamiento
- `-s, --savefile`: Guardar modelo entrenado
- `-o, --opt`: Tasa de aprendizaje optimizador (default: 0.001)
- `-n, --neuronas_fisrt_layer`: Neuronas de la primera capa

**Formato de salida del modelo** (1 valor codificado en 8 bits):
- Bits 2-4: Estado del ventilador (100=LOW, 010=MED, 001=HIGH, 000=STOP)
- Bit 5: Modo frío activo
- Bit 6: Modo seco activo
- Bit 7: Modo calor activo

#### 5. trama.py
**Propósito**: Gestión y procesamiento de tramas del protocolo CCN.

**Constantes importantes**:
```python
VALID_SIZES = [85, 88, 91, 94]  # Tamaños de trama válidos
```

**Funciones principales**:
- `validar_trama_bytes()` - Valida estructura de trama (header `0x00, 0x08`)
- `guardar_trama_bytes()` - Guarda trama en formato binario (.bin) + metadatos (.info)
- `cargar_trama_bytes()` - Carga trama desde archivo binario
- `normalizar_datos()` - Prepara datos para entrada a la red neuronal
- `normalizar_bloque()` - Elimina bytes no necesarios (delimitadores)
- `crear_lista_json()` - Crea archivo JSON con lista de capturas
- `resuelve_nombre_binarios_lista()` - Resuelve lista de archivos desde JSON

**Formato de almacenamiento**:
```
dumps/
├── <uuid>.bin       # Datos binarios crudos de la trama
└── <uuid>.info      # Metadatos JSON (timestamp, baudrate, valores, etc.)
```

#### 6. ia.py
**Propósito**: Funciones de bajo nivel para TensorFlow/Keras.

**Constantes**:
```python
TAMANO_LOTE = 10
REPETICIONES = 1
```

**Funciones principales**:
- `init_modelo()` - Crea modelo secuencial con capas Dense
  - Input: Variable según `neuronasfirstlayer` (default 79)
  - Capas ocultas: Configurables (ej: 50,50,50)
  - Output: 1 neurona (sigmoid) - estado codificado en 8 bits
  - Optimizador: Adam
  - Loss: mean_squared_error
- `load_modelo()` / `save_modelo()` - Persistencia de modelos
- `entrenar_datos()` - Wrapper de model.fit()
- `predecir()` - Realiza predicciones y decodifica el resultado
- `crear_dataset()` - Crea dataset TensorFlow a partir de listas
- `normalizar()` - Normaliza datos dividiendo por 255
- `recoge_datos_modelo()` - Extrae información de capas del modelo
- `mostrar_pesos_modelo()` - Muestra pesos y configuración de capas

#### 7. config.py
**Propósito**: Gestión centralizada de configuración.

**Clase**: `ConfigurationManager`

**Métodos principales**:
- `get(key)` - Obtiene valor de configuración
- `set(key, val)` - Establece valor de configuración
- `update(data)` - Actualiza múltiples valores
- `load_config()` - Carga configuración desde archivo
- `save_config_file()` - Guarda configuración a archivo
- `get_keys()` - Obtiene lista de claves configurables

#### 8. graphics.py
**Propósito**: Visualización de datos de entrenamiento.

**Funciones**:
- `mostrar_graf(historial)` - Muestra gráfico de precisión y pérdida

---

## Flujo de Trabajo Típico

### 1. Captura de Datos

```bash
# Capturar datos con estado específico del termostato
python3 curryrtool.py device read -p /dev/ttyUSB0 -b 2400 -t 0.1 -o "0,1,0,1,0,0"

# Los archivos se guardan en dumps/ como:
# - <uuid>.bin (datos binarios)
# - <uuid>.info (metadatos JSON)
```

### 2. Entrenamiento

```bash
# Entrenar modelo con todos los datos capturados
python3 curryrtool.py trainning entrenar -l 50,50,50 -e 20000 -s

# Modelo guardado en models/modelo__<layers>__<epocas>_<timestamp>.keras
```

### 3. Predicción

```bash
# Predecir estado desde archivo de trama
python3 curryrtool.py trainning predecir -m models/modelo.keras -f dumps/<uuid>.bin
```

### 4. Emulación (Repetición de Tramas)

```bash
# Reproducir trama capturada para emular el termostato
python3 curryrtool.py device write \
    -p /dev/ttyUSB0 -b 2400 -t 0.1 \
    -f dumps/<uuid>.bin -i 5
```

---

## Convenciones de Código

- **Idioma**: Código y comentarios en español
- **Nombres de variables**: snake_case (ej: `datos_entrenamiento`, `nombre_modelo`)
- **Constantes globales**: UPPER_CASE (ej: `VALID_SIZES`, `TAMANO_LOTE`)
- **Encoding**: UTF-8 (especificado en shebang `# -*- coding: utf-8 -*-`)
- **Indentación**: 4 espacios
- **Type hints**: Uso de anotaciones de tipo (`List[int]`, `Dict[str, Any]`)

---

## Estrategia de Testing

Actualmente **no hay tests automatizados**. El proyecto se prueba de forma manual:
- Verificación de lectura de datos en tiempo real
- Comprobación visual de gráficas de entrenamiento
- Validación de predicciones contra estados conocidos
- Pruebas de emulación con tramas capturadas

---

## Estructura de Datos

### Formato Binario (dumps/*.bin + *.info)

**Archivo .bin**: Datos crudos de la trama (variable 85-94 bytes)

**Archivo .info** (JSON):
```json
{
    "stream": "dumps/<uuid>.bin",
    "blocks": 10,
    "timeout": 0.1,
    "baudrate": 2400,
    "times": [["2024-11-27T21:16:00", 88], ...],
    "uuid": "<uuid>",
    "values": "0,0,1,0,0,1"
}
```

### Campos de valores

- **values**: 6 valores binarios separados por comas:
  - `[fan1, fan2, fan3, cold, hot, dry]`
  - Ejemplo `0,0,1,0,0,1` = Fan alto (001), Dry activo

### Modelos entrenados

- Guardados en carpeta `models/` (creada automáticamente)
- Formato: Keras (.keras)
- Convención de nombres: `modelo__<layers>__<epocas>_<timestamp>.keras`

---

## Documentación Adicional

| Archivo | Descripción |
|---------|-------------|
| `CCN_PROTOCOL_ANALYSIS.md` | Análisis técnico detallado del protocolo Carrier Comfort Network |
| `CHECKLIST.md` | Guía paso a paso para ingeniería inversa del protocolo |
| `README_es.md` | Documentación general del proyecto (español) |
| `README.md` | Documentación general del proyecto (inglés) |
| `docs/VERIFICACION_RS485_vs_RS232.md` | Verificación de conexión física |
| `docs/INFINITIVE_ANALYSIS.md` | Análisis adicional del protocolo |
| `docs/INFINITUDE_DATA_INTERPRETATION.md` | Interpretación de datos |
| `docs/INFINITUDE_OBSERVATIONS_ANALYSIS.md` | Análisis de observaciones |

---

## Consideraciones de Seguridad

- El módulo `devices_interface.py` requiere acceso al puerto serie (`/dev/ttyUSB0`)
- Los datos del termostato no contienen información sensible
- Los modelos entrenados se guardan localmente
- **Precaución**: El modo `write` puede alterar el estado del termostato si se conecta a un bus con otros dispositivos

---

## Notas para Desarrolladores

1. **No hay build system**: El proyecto se ejecuta directamente con Python, sin necesidad de build ni instalación (solo dependencias).

2. **Múltiples tamaños de trama**: El protocolo CCN genera tramas de diferentes tamaños según el estado:
   - 85 bytes: Estados simples
   - 88 bytes: Estados con modo activo (cool/heat/dry)
   - 91 bytes: Estados extendidos
   - 94 bytes: Estados complejos o con sensores adicionales

3. **Normalización**: Los datos se normalizan dividiendo por 255 (conversión a float 0-1) antes de entrar a la red neuronal.

4. **Arquitectura del modelo**: Red neuronal secuencial con:
   - Capa de entrada variable según configuración `neuronasfirstlayer`
   - Capas ocultas Dense configurables con activación ReLU
   - Capa de salida Dense(1, sigmoid) con codificación de 8 bits

5. **Modo repetidor**: La funcionalidad `repeat` permite usar el adaptador como puente/sniffer entre dos puertos serie.

6. **Gestión de configuración**: La clase `ConfigurationManager` maneja automáticamente la creación de claves faltantes con valores por defecto.

---

*Documento actualizado para reflejar la arquitectura actual del proyecto (Abril 2026)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/darioaxel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
