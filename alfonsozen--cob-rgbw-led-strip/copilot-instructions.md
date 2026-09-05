## cob-rgbw-led-strip

> Se trata de un sistema de iluminación inteligente montado sobre paneles acústicos de MDF en mi escritorio. El sistema controla 5 tiras de luces FCOB (WS2811 a 24V) mediante un ESP32 corriendo el firmware WLED (Sound Reactive), integrando un micrófono I2S.

# Proyecto: Workspace WLED "Stealth" + Audio Reactive + SinricPro (Node.js)

## Contexto del Proyecto
Se trata de un sistema de iluminación inteligente montado sobre paneles acústicos de MDF en mi escritorio. El sistema controla 5 tiras de luces FCOB (WS2811 a 24V) mediante un ESP32 corriendo el firmware WLED (Sound Reactive), integrando un micrófono I2S. 

El ESP32 opera de forma autónoma para los efectos reactivos al audio, pero necesito un script en **Node.js (TypeScript)** que actúe como puente central para integrarlo con Alexa (vía SinricPro) y para ejecutar una lógica de temporizador Pomodoro controlada por voz.

## 1. Arquitectura de Hardware (Solo como contexto)
- **Cerebro:** ESP32 ejecutando WLED SR (Sound Reactive).
- **Tiras LED:** FCOB WS2811 24V (720 LEDs/m).
  - Tira 1 y 5 (Laterales, 2.4m c/u): Orientación vertical piso-techo.
  - Tira 2, 3 y 4 (Centrales, 1.7m c/u): Orientación vertical sobre el escritorio.
- **Datos:** Conectadas en serie (Zig-zag) a un solo pin (GPIO 16).
- **Micrófono:** INMP441 (I2S) conectado a los pines 14, 15 y 32.

## 2. Configuración Lógica en WLED (Requerimiento para tu código)
Las tiras están divididas en dos segmentos en WLED, con separación de frecuencias para el audio:
- **Segment 0 (Tiras Centrales):** Configurado para reaccionar a frecuencias altas/medias (Agudos/Voces) con sensibilidad normal.
- **Segment 1 (Tiras Laterales):** Configurado para reaccionar solo a frecuencias bajas (Lows/Bass) con alta sensibilidad.

## 3. Tarea a Desarrollar: Script en Node.js (TypeScript) + SinricPro
Necesito que escribas la aplicación en Node.js que correrá de fondo en mi PC. Esta aplicación usará el SDK de SinricPro para exponer dispositivos a Alexa y mandará comandos a la API JSON de WLED (`http://[WLED_IP]/json/state`).

### Requerimientos del Script (Dispositivos en SinricPro):

**Dispositivo A: Control de Iluminación (Smart Light)**
- Debe permitir encender/apagar todas las luces (Segment 0 y 1).
- Debe permitir cambiar el color base.
- Debe actuar como un "passthrough" que traduzca el comando de Alexa a un POST hacia la API JSON de WLED.

**Dispositivo B: Controlador del Pomodoro (Switch o Range Controller)**
- Integrado a Alexa para que al encenderse (o al recibir un valor de tiempo), inicie una sesión de Pomodoro (ej. 25 minutos).
- **Lógica del Pomodoro:**
  - Al iniciar, debe guardar el estado actual de las luces (para restaurarlo al terminar).
  - Debe tomar control **únicamente del Segment 0** (Tiras Centrales).
  - Debe usar la API JSON de WLED para aplicar un efecto visual de "Progreso" (ej. efecto *Percent* o ir llenando la tira de un color que cambie gradualmente de Verde -> Amarillo -> Rojo conforme se acaba el tiempo).
  - Al finalizar el tiempo, las luces del Segment 0 deben parpadear suavemente o cambiar a un color de "Descanso", y luego restaurar el efecto de Audio Reactivo que tenían originalmente.

## 4. Entregables Esperados
1. El código completo en Node.js (TypeScript), modular y comentado.
2. Instrucciones de qué tipos de dispositivos exactos debo crear en el portal web de SinricPro (Smart Light, Custom Device, etc.) para que coincidan con tu código.
3. Un ejemplo de los payloads JSON que tu script enviará a WLED para lograr el efecto del Pomodoro sin afectar a las tiras laterales.

---
> Source: [AlfonsoZen/cob-rgbw-led-strip](https://github.com/AlfonsoZen/cob-rgbw-led-strip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
