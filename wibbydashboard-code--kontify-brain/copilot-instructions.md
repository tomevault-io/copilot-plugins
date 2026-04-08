## kontify-brain

> - **Fase**: Blueprint (Confirmado)

# 🚀 Proyecto: System Pilot Diagnostic Tool
## Mapa del Proyecto B.L.A.S.T.

### 📍 Estado Actual
- **Fase**: Blueprint (Confirmado)
- **Última Actualización**: 2026-02-05

---

### 🗺️ Blueprint (Plano)
- **North Star**: Herramienta de diagnóstico y precalificación de leads multi-nicho. Resultado: Capturar datos, agradecimiento al usuario y entrega de Diagnóstico de Riesgo y Oportunidad detallado al Administrador.
- **Integraciones**: 
    - **Frontend**: Landing Page (HTML/CSS/JS) con selector de nicho (Poppins font, Azul Eléctrico).
    - **Procesamiento**: API de Gemini/OpenAI para análisis determinista.
    - **Notificaciones/Entrega**: Google Sheets (DB Leads) y Email/Slack para el Administrador.
- **Source of Truth**: 5 archivos .md (holding, constructora, autotransporte, comercializadora, manufactura) con 66 preguntas cada uno.
- **Delivery Payload**: 
    - **Usuario**: Página de éxito: "Datos recibidos, un consultor senior se contactará pronto".
    - **Administrador**: Diagnóstico de Riesgo estructurado (Score, 3 puntos de fallo, Impacto Económico, Sales Pitch).
- **Reglas de Comportamiento**:
    - **Identidad**: "Consultor Senior de Mentores Estratégicos".
    - **Tono**: Profesional, técnico, sobrio y autoritario.
    - **Restricciones**: No inventar datos. Señalar falta de información como "Riesgo por falta de visibilidad".
    - **Estética**: Fuente Poppins, colores: Blanco (#FFFFFF), Gris (#808080), Negro (#000000) y Azul Eléctrico (#007BFF).

### 📊 Esquema de Datos (Data Schema)

#### 📥 Raw Lead Payload (Input)
```json
{
  "lead_metadata": {
    "company_name": "string",
    "niche_id": "holding | constructora | autotransporte | comercializadora | manufactura",
    "contact_name": "string",
    "contact_email": "string",
    "timestamp": "ISO-8601",
    "source_url": "string"
  },
  "responses": [
    {
      "q_index": "number (1-66)",
      "category_id": "string (e.g., 'I. Estructura y Registro')",
      "question": "string",
      "answer": "string | boolean",
      "observation": "string (optional)"
    }
  ]
}
```

#### 📤 Diagnostic Payload (Output)
```json
{
  "admin_report": {
    "risk_score": "number (0-100)",
    "summary": "string",
    "critical_findings": [
      {
        "issue": "string",
        "category": "string",
        "economic_risk": "string (estimated value/impact)",
        "mitigation_strategy": "string"
      }
    ],
    "sales_strategy": {
      "pitch": "string",
      "recommended_service": "string"
    },
    "markdown_content": "string (Full formatted report for delivery)"
  }
}
```

---

### 🔗 Link (Conectividad)
- [ ] Verificación de .env (API Keys, SMTP/Slack Tokens)
- [ ] Handshake: Script para validar conectividad con Gemini/Sheets.

---

### 🏗️ Architect (A.N.T.)
- **Capa 1: Architecture**: [SOPs en architecture/]
- **Capa 2: Navigation**: Antigravity logic (Routing entre prompts y herramientas).
- **Capa 3: Tools**: Scripts de Python para procesamiento de datos y envíos.

---

### 🛰️ Trigger & Maintenance Log
Estado final del despliegue: **ACTIVO**

#### 🛠️ Guía de Mantenimiento (SOP Administrativo)

**1. Actualización de Cuestionarios (66 Preguntas)**
- Edite directamente los archivos `.md` en la raíz (ej. `constructora_diagnostico.md`).
- El sistema de IA leerá automáticamente los cambios en el próximo diagnóstico. 
- *Regla:* No cambie el formato de numeración (Q1, Q2, etc.) para mantener la coherencia del mapeo.

**2. Rotación de API Keys**
- Abra el archivo `.env`.
- Reemplace los valores de `GEMINI_API_KEY`, `SLACK_WEBHOOK_URL` o `GOOGLE_SHEETS_ID`.
- Reinicie el servidor (`python tools/server.py`) para aplicar cambios.

**3. Cómo añadir un Nuevo Nicho (Ej: Sector Médico)**
- **Capa 1 (Architecture):** Cree `architecture/medico_sop.md`. Defina los riesgos específicos (ej. COFEPRIS, Malpraxis).
- **Capa 3 (Tools):** Asegúrese de que el `niche_id` en el Frontend coincida con el nombre del SOP. 
- **Source of Truth:** Cree `medico_diagnostico.md` con las preguntas del quiz.

**4. Verificación de Salud**
- Ejecute `python tools/check_health.py` para validar la estabilidad de las conexiones.

#### 📝 Registro de Actividad
- **2026-02-05:** Lanzamiento oficial marca **Kontify**. Arquitectura A.N.T. desplegada.
- **2026-02-05:** Integración de `gemini-2.0-flash` para diagnósticos deterministas.
- **2026-02-05:** Fallback de logs locales activado para seguridad de leads.

---
**System Pilot: Misión Cumplida.**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wibbydashboard-code)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wibbydashboard-code)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
