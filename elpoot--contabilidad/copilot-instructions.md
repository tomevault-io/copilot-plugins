## contabilidad

> Este archivo proporciona el contexto fundamental, reglas de negocio y guías arquitectónicas para el proyecto **Gestor Contable**.

# GEMINI.md - Contexto de Instrucción para Gestor Contable

Este archivo proporciona el contexto fundamental, reglas de negocio y guías arquitectónicas para el proyecto **Gestor Contable**.

## 🌐 Preferencia de Comunicación
**SIEMPRE responde primero en ESPAÑOL.** Este es el idioma principal del proyecto y de sus usuarios.

---

## 🛡️ Normas de Conducta y Seguridad (CRÍTICO)

### 1. Prohibición de Scripts de Parcheo Externos
**NUNCA** generes, sugieras o ejecutes scripts de Python en la raíz del proyecto para modificar el código fuente (ej. `fix_main_window.py`). 
- Estas herramientas son consideradas "basura" y peligrosas, ya que pueden sobrescribir lógica moderna con versiones obsoletas.
- Todas las modificaciones de código deben realizarse mediante ediciones directas y quirúrgicas sobre los archivos en `gestor_contable/` usando las herramientas integradas del agente (ej. `replace`).

### 2. Jerarquía de Instrucciones
Para asegurar la consistencia entre diferentes agentes de IA, Gemini **DEBE** respetar y seguir las directrices establecidas en:
- `CLAUDE.md`: Contiene las reglas de desarrollo y refactorización más detalladas.
- `AGENTS.md`: Contiene la estructura técnica y lógica de negocio.
En caso de conflicto, las reglas de `CLAUDE.md` tienen prioridad.

### 3. Integridad de Git
- **NUNCA** realices cambios que no puedan ser verificados o revertidos fácilmente mediante Git.
- Todos los archivos temporales generados para pruebas (`tmp_*.py`) deben ser eliminados tras su uso.

---

## 🚀 Resumen del Proyecto
**Gestor Contable** es un sistema de clasificación y organización de facturas electrónicas para una firma contable en Costa Rica. El sistema procesa archivos XML y PDF, los vincula mediante la Clave de Hacienda (50 dígitos), permite su clasificación contable y los organiza físicamente en una estructura de red para su posterior procesamiento.

### Tecnologías Principales
- **Lenguaje:** Python 3.10+
- **GUI:** CustomTkinter (Tema oscuro moderno)
- **Procesamiento de Datos:** Pandas, Openpyxl (Excel)
- **PDF:** PyMuPDF (fitz) para renderizado y extracción de texto
- **Persistencia:** SQLite (para rastreo de clasificación y caché de Hacienda)
- **API:** Integración con el API de comprobantes de Hacienda (Costa Rica)

---

## 🏗️ Arquitectura y Estructura
El proyecto se encuentra en una **fase activa de refactorización** hacia una arquitectura desacoplada:

```
gestor_contable/
├── app/            # NUEVO: Capa de aplicación (Casos de Uso, Controladores, State)
│   ├── use_cases/  # Lógica de flujo (Exportación, Clasificación en lote)
│   ├── controllers/# Orquestación entre UI y Dominio
│   └── state/      # ViewModels y estado de la ventana principal
├── core/           # Dominio: Lógica de negocio estable (Modelos, XML, PDF, DB)
├── gui/            # Vistas: Solo renderizado y eventos (CustomTkinter)
├── config.py       # Configuración de rutas y montaje de unidad Z:
└── main.py         # Punto de entrada de la aplicación
```

### Reglas de Desarrollo (CRÍTICO)
1. **Alcance:** SOLO modificar archivos dentro de `gestor_contable/`.
2. **Desacoplamiento:** No agregar lógica de negocio en `gui/`. Las nuevas funcionalidades deben usar la capa `app/`.
3. **Hilo Principal:** La UI corre en el hilo principal. Operaciones pesadas (E/S, API, Movimiento de archivos) DEBEN ir en hilos de trabajo (`ThreadPoolExecutor`).
4. **Seguridad SQLite:** Todo acceso a bases de datos SQLite debe estar protegido por un `threading.Lock()`.
5. **Tipado:** Usar `from __future__ import annotations` y Type Hints en todo código nuevo.

---

## ⚖️ Reglas de Negocio Críticas

### 1. Integridad Fiscal (Movimiento Atómico)
Las facturas son documentos fiscales. El protocolo de movimiento en `core/classifier.py` es inviolable:
1. Calcular SHA256 del original.
2. Copiar preservando metadatos (`shutil.copy2`).
3. Calcular SHA256 de la copia.
4. Si coinciden, borrar el original. Si no, abortar y dejar el original intacto.

### 2. Clasificación de XML por Contenido
**NUNCA** clasificar XMLs por nombre de archivo o sufijos (e.g., `_respuesta.xml`).
**SIEMPRE** leer el contenido del XML y usar el tag raíz (`FacturaElectronica`, `MensajeHacienda`, etc.) para determinar el tipo de documento.

### 3. Vínculo XML-PDF (Clave de 50 Dígitos)
El sistema identifica documentos mediante la **Clave de Hacienda de 50 dígitos**.
- Estrategia de búsqueda: Nombre del PDF -> Texto del PDF -> Bytes del PDF -> Consecutivo (fallback).
- En Notas de Crédito con dos claves, **siempre usar la última encontrada** (la del documento actual).

---

## 🛠️ Comandos Comunes

| Acción | Comando |
|--------|---------|
| **Instalar Dependencias** | `pip install -r requirements.txt` |
| **Ejecutar Aplicación** | `python gestor_contable/main.py` |
| **Compilar (.exe)** | `python build.py` |
| **Logs** | `~/.gestor_contable_logs/gestor_contable.log` |

---

## 📂 Estructura de Datos (Unidad Z:)
El sistema depende de una unidad de red `Z:` (mapeada usualmente a OneDrive) con la siguiente estructura:
- `Z:/DATA/PF-{año}/CLIENTES/{CLIENTE}/XML/`: Fuente de XMLs.
- `Z:/DATA/PF-{año}/CLIENTES/{CLIENTE}/PDF/`: Fuente de PDFs.
- `Z:/DATA/PF-{año}/Contabilidades/{mes}/{cliente}/`: Destino de clasificación.
- `Z:/DATA/hacienda_cache.db`: Caché compartida del API de Hacienda.

---

## 🎨 Paleta de Colores (UI)
Mantener consistencia visual en todos los módulos de GUI:
- **Fondo:** `#0d0f14` (BG), `#13161e` (Surface), `#181c26` (Card)
- **Acentos:** `#2dd4bf` (Teal), `#34d399` (Success), `#f87171` (Danger), `#fbbf24` (Warning)
- **Texto:** `#e8eaf0` (Principal), `#6b7280` (Muted)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ElPoot)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ElPoot)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
