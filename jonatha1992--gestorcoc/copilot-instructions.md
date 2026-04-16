## gestorcoc

> > **Misión**: GestorCOC es un sistema integral para el control operativo de centros de comando (COC), optimizando la gestión de activos, personal e incidentes con integraciones de IA.

# GEMINI.md — El Cerebro de GestorCOC

> **Misión**: GestorCOC es un sistema integral para el control operativo de centros de comando (COC), optimizando la gestión de activos, personal e incidentes con integraciones de IA.

## 🗺️ Mapa del Proyecto

- `backend/`: Django 5.2 + DRF. Lógica de negocio, API y servicios de IA.
- `frontend/`: Angular 21 + Tailwind CSS v4. SPA dinámica y responsiva.
- `docs/`: Documentación técnica y operativa.
  - `architecture/`: Visión técnica del sistema.
  - `decisions/`: Architecture Decision Records (ADR).
  - `runbooks/`: Guías operativas paso a paso.
- `.claude/skills/`: Automatismos para la IA (pruebas, deploys).
- `tools/scripts/`: Scripts de utilidad para desarrollo.

## 🛠️ Stack Tecnológico Interno

| Componente | Tecnología |
| :--- | :--- |
| **Backend** | Python 3.14+, Django 5.2, DRF, SQLite (Dev), PostgreSQL (Prod) |
| **Frontend** | Angular 21, Tailwind CSS v4, ApexCharts |
| **IA** | Gemini (Primario), OpenRouter, Groq, Ollama |
| **Hosting** | Railway (Gunicorn + WhiteNoise) |

## 📖 Documentación Clave

1. [Índice General](file:///d:/Repositorio/jonatha1992/gestorcoc/docs/README.md)
2. [Visión General de Arquitectura](file:///d:/Repositorio/jonatha1992/gestorcoc/docs/architecture/0-overview.md)
3. [Módulos del Sistema](file:///d:/Repositorio/jonatha1992/gestorcoc/docs/architecture/3-modulos.md)
4. [Roles y Permisos](file:///d:/Repositorio/jonatha1992/gestorcoc/docs/architecture/2-roles-and-permissions.md)
5. [Stack Detallado](file:///d:/Repositorio/jonatha1992/gestorcoc/docs/architecture/1-stack.md)
6. [Guía de Despliegue](file:///d:/Repositorio/jonatha1992/gestorcoc/docs/runbooks/hacer_deploy.md)
7. [Planning y Estado](file:///d:/Repositorio/jonatha1992/gestorcoc/docs/PLANNING.md)

## ⚡ Comandos Rápidos

- **Semilla de datos**: `.\tools\scripts\seed_db.ps1`
- **Pruebas backend**: `python manage.py test records`
- **Levantar entorno**: Ver [deploy_local skill](file:///d:/Repositorio/jonatha1992/gestorcoc/.claude/skills/deploy_local/skill.md)

---
*Este archivo es la fuente de verdad para la navegación y contexto del proyecto.*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonatha1992) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
