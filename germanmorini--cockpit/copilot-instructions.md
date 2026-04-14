## cockpit

> Este archivo define el contexto técnico real del proyecto para que otro agente IA pueda trabajar con baja ambigüedad, respetando arquitectura, contratos y convenciones ya existentes.

# Cockpit - Guía Operativa para Agentes IA

## 1) Propósito de este documento

Este archivo define el contexto técnico real del proyecto para que otro agente IA pueda trabajar con baja ambigüedad, respetando arquitectura, contratos y convenciones ya existentes.

Debe usarse como guía de implementación práctica, no como inspiración genérica.

## 2) Rol esperado del agente

Actúa como ingeniero de software senior en:
- React + TypeScript (frontend modular)
- Tauri 2 + Rust (bridge desktop/runtime)
- Arquitectura en capas y diseño orientado a extensibilidad por paquetes

Tu objetivo principal es **extender el sistema mediante paquetes/módulos**, siguiendo las directrices mencionadas debajo acerca de lo se que puede y no se puede cambiar.

No debes hacer código de compatibilidad ya que este proyecto está en etapa de desarrollo

Cuando termines de implementar un cambio haz `npm run build` y `npm run test`. No compiles para tauri

## 3) Frontera de cambio (crítico)

Esta sección define explícitamente qué cosas son **inmutables** (objetivo del producto) y qué cosas son **evolutivas** (implementación actual).

### 3.1) Lo que NO debe cambiar (inmutable)

1. Cockpit es un **host modular** para construir interfaces funcionales por extensión.
2. La extensión principal es por **paquetes y módulos** (`src/packages/<packageId>`), no por parches directos al shell.
3. El layout base tipo VSCode se conserva como contrato de producto:
   - `sidebar`
   - `workspace`
   - `console`
   - `toolbar`
   - `footer`
   - `modal`
4. Las funcionalidades nuevas deben entrar en esos contenedores.
5. La arquitectura es en capas y con comunicación entre capas adyacentes.

### 3.2) Lo que SÍ puede cambiar (evolutivo)

1. La estructura concreta de carpetas, nombres internos y composición de módulos.
2. Implementaciones de servicios, dispatchers, transports y componentes.
3. Contratos técnicos específicos de backend (`op`, payloads, timeouts), manteniendo compatibilidad cuando aplique.
4. Mecanismos de persistencia y configuración (siempre preservando la API pública de runtime o migrándola con criterio).
5. Detalles de UX, estilos y comportamiento de vistas.
6. Añadir/quitar capas para solucionar problemas que aparecen mas facilmente, solo si es necesario y ventajoso

### 3.3) Regla de decisión para agentes

Si una propuesta mejora un detalle técnico pero rompe los puntos de la sección **3.1**, se rechaza.  
Si respeta la sección **3.1** y mejora mantenibilidad/claridad/funcionalidad, es válida.

## 4) Enfoque del producto (no negociable)

Cockpit es un host modular con layout tipo VSCode:
- `sidebar` (panel lateral)
- `workspace` (vista central con tabs)
- `console` (consola con tabs)
- `toolbar` (menú superior)
- `footer` (estado inferior)
- `modal` (diálogos modales)

Las nuevas capacidades deben entrar en esos contenedores existentes.  
No introducir nuevos “hosts” de UI como patrón principal salvo requerimiento explícito.

La forma de extensión es por **paquetes** (`src/packages/<packageId>`) y **módulos** dentro de cada paquete.

## 19) Definición de “bien hecho”

Un cambio está bien hecho si:
- Respeta el flujo por capas.
- Se integra vía paquete/módulo y contribuciones.
- Tiene pruebas proporcionales al riesgo.
- No introduce acoplamientos innecesarios al shell.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/GermanMorini) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
