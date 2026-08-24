## ib-tracker

> Este proyecto es un **Interactive Brokers (IB) Portfolio Tracker**, una aplicación web diseñada para visualizar y analizar el rendimiento de un portafolio de inversiones a partir de los reportes de transacciones exportados por Interactive Brokers.

# Gemini Context: IB Tracker

Este proyecto es un **Interactive Brokers (IB) Portfolio Tracker**, una aplicación web diseñada para visualizar y analizar el rendimiento de un portafolio de inversiones a partir de los reportes de transacciones exportados por Interactive Brokers.

## 🚀 Descripción General

La aplicación parsea dinámicamente archivos CSV de transacciones de IB, calcula las posiciones actuales (holdings), el costo promedio de compra (incluyendo comisiones), el historial de dividendos y un historial completo de operaciones. Utiliza Yahoo Finance para obtener precios en tiempo real y ofrece un dashboard interactivo con comparativas contra el S&P 500.

### Tecnologías Principales
- **Backend:** Python con Flask, Pandas, yfinance.
- **Frontend:** HTML5, Vanilla CSS, JavaScript (ES6+), Chart.js 4.
- **Datos:** Gestión de múltiples archivos CSV con deduplicación de transacciones.

---

## 📂 Estructura del Proyecto

- `app.py`: Servidor Flask principal. Maneja las rutas de la API, el parsing de múltiples CSVs, la obtención de precios y la subida de archivos.
- `app-web.py`: Versión optimizada del servidor para despliegue online (ej: PythonAnywhere), con rutas ajustadas y proxy configurado.
- `data/`: Directorio donde se almacenan todos los archivos CSV de transacciones (`U13493500*.csv`).
- `portafolio-dashboard.html`: UI del dashboard (SPA) con pestañas para Posiciones, Comparativa vs S&P 500, Dividendos y Transacciones.
- `dashboard.js`: Lógica del frontend (fetch de datos, renderizado de gráficos y gestión de subida de archivos).
- `test_parsing.py`: Script de utilidad para probar la lógica de parsing de forma aislada.
- `requirements.txt`: Dependencias de Python.

---

## 🛠️ Configuración y Ejecución

### Instalación
```bash
pip install -r requirements.txt
```

### Ejecución
1. El servidor se inicia con:
   ```bash
   python app.py
   ```
   *Alternativa rápida:* Haz doble clic en `iniciar-tracker.command`.
2. **Automatización:** El servidor abrirá automáticamente el navegador en `http://localhost:8080`.
3. **Acceso Remoto (Local):** El servidor escucha en `0.0.0.0`, permitiendo el acceso desde otros dispositivos en la misma red Wi-Fi usando la IP local del equipo.

### Gestión de Datos
- **Subida:** Puedes subir nuevos archivos CSV directamente desde el dashboard usando el botón "Subir CSV". Los archivos se guardan en la carpeta `data/` con un timestamp.
- **Procesamiento:** El sistema lee *todos* los CSVs en la carpeta `data/`, combinando las transacciones y eliminando duplicados automáticamente.

### Acceso Remoto (Fuera de casa - TODO EN UNO)
Para iniciar el tracker y habilitar el acceso remoto al mismo tiempo:
1. Ejecuta `iniciar-tracker.command`.
2. El terminal mostrará una dirección similar a `https://algo.serveo.net`. Esa es la URL que puedes abrir desde cualquier lugar.
3. Al cerrar la ventana del terminal, tanto el servidor como el túnel se detendrán automáticamente.

---

## 📝 Convenciones de Desarrollo

- **Deduplicación:** Se utiliza un set de tuplas (`seen_rows`) en el backend para evitar procesar la misma transacción múltiples veces si se suben reportes que se solapan en fechas.
- **Transacciones:** El endpoint `/api/portfolio` devuelve el historial completo de compras y ventas ordenado por fecha descendente.
- **UX:** El dashboard incluye estados de carga (spinners) para la obtención de precios y la subida de archivos.
- **Localización:** Formateo de moneda y números adaptado a `es-VE` para la visualización.

---

## 📌 Notas de Contexto para Gemini

- La lógica de negocio principal reside en las funciones `parse_csv`, `compute_holdings` y `compute_dividends` de `app.py`.
- Al añadir nuevos tickers, actualiza la constante `TICKER_NAMES` en `app.py` para que aparezcan con su nombre completo en el dashboard.
- El servidor está configurado para servir archivos estáticos directamente desde la raíz del proyecto.

---
> Source: [akirintxa/ib-tracker](https://github.com/akirintxa/ib-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
