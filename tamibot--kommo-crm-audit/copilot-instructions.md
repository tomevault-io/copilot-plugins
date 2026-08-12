## kommo-crm-audit

> Es un framework reutilizable para auditar, documentar, implementar y mantener automatizaciones en Kommo CRM. Funciona en 5 fases: extracción de datos → documentación → plan de acción → validación → soporte continuo.

# Instrucciones para LLM - Sistema de Auditoría Kommo CRM

## Qué es este proyecto
Es un framework reutilizable para auditar, documentar, implementar y mantener automatizaciones en Kommo CRM. Funciona en 5 fases: extracción de datos → documentación → plan de acción → validación → soporte continuo.

## Tu rol
Cuando trabajes con este proyecto, tu objetivo es ayudar en cualquiera de las 5 fases. Puedes:
1. **Extraer datos** del CRM usando los scripts y la API
2. **Analizar bots** parseando sus JSON y generando diagramas Mermaid
3. **Detectar errores** en los flujos existentes
4. **Proponer mejoras** basándote en datos concretos
5. **Generar documentos de solución** cuando alguien reporte un error o solicite algo nuevo
6. **Validar bots** nuevos contra las reglas establecidas

## Estructura del proyecto
```
docs/00_sistema_auditoria.md    → Documento maestro (leer primero)
docs/01_documentacion_funcionamiento.md → Template de documentación
docs/02_plan_accion_guia_bots.md → Guía para crear bots
docs/03_validacion_flujos.md     → Proceso de validación
docs/04_sistema_soporte.md       → Sistema de soporte continuo
docs/referencia_acciones_bot.md  → TODAS las acciones del Salesbot
docs/reglas_whatsapp.md          → Reglas de WhatsApp Business
docs/integracion_postgresql.md   → Schema y queries de PostgreSQL
credentials/README.md            → Cómo obtener tokens de API
scripts/kommo_api.py             → Extracción de datos
scripts/analizar.py              → Análisis y diagramas
scripts/validar_bot.py           → Validación de bots
scripts/generar_solucion.py      → Generador de docs de solución
scripts/generar_pdf.py           → Generación de PDF
scripts/schema.sql               → Schema de PostgreSQL
```

## API de Kommo
- **Base URL:** `https://{subdomain}.kommo.com/api/v4/`
- **Auth:** Bearer Token (OAuth 2.0 o Long-lived token)
- **Rate Limit:** 7 req/seg
- **Tokens OAuth:** expiran cada 24h (renovar con refresh_token)
- **Long-lived tokens:** configurables de 1 día a 5 años

### Endpoints clave
| Método | Endpoint | Uso |
|--------|----------|-----|
| GET | `/api/v4/leads/pipelines` | Pipelines + etapas |
| GET | `/api/v4/leads/custom_fields` | Campos de leads |
| GET | `/api/v4/contacts/custom_fields` | Campos de contactos |
| GET | `/api/v4/companies/custom_fields` | Campos de empresas |
| POST | `/api/v4/leads/custom_fields` | Crear campos |
| POST | `/api/v4/leads/pipelines/{id}/statuses` | Crear etapas |
| POST | `/api/v4/leads/pipelines` | Crear pipelines |
| GET | `/api/v4/account` | Info de la cuenta |
| POST | `/oauth2/access_token` | Obtener/renovar token |

## Handlers del Salesbot (acciones de los bots)
| Handler | Qué hace |
|---------|----------|
| `send_message` | Enviar mensaje (respeta ventana 24h WhatsApp) |
| `waits` | Esperar evento (mensaje, timer, cambio etapa) |
| `action` | Acciones: change_status, set_custom_fields, add_tag, remove_tag, add_note, create_task, set_responsible, set_budget, create_lead, link_contact, link_company |
| `distribution` | Round robin, random, weighted |
| `start` | Iniciar otro bot (DETENER el actual después) |
| `condition` | Bifurcación condicional |
| `validations` | Validar respuesta (email, phone, regex, etc.) |
| `webhook` / `widget_request` | Llamar servicio externo |
| `send_internal` | Mensaje interno al equipo |
| `goto` | Ir a otro paso |
| `_start` / `_stop` | Inicio y fin del bot |

## Variables disponibles en bots
`{{contact.name}}`, `{{contact.phone}}`, `{{contact.email}}`, `{{lead.id}}`, `{{lead.name}}`, `{{lead.price}}`, `{{lead.cf.FIELD_ID}}`, `{{lead.responsible.name}}`, `{{lead.responsible.email}}`, `{{message_text}}`, `{{message_text.email}}`, `{{message_text.phone}}`, `{{origin}}`, `{{current_date}}`, `{{rand}}`

## Reglas críticas de WhatsApp
- **Ventana 24h:** mensajes libres solo dentro de 24h del último mensaje del contacto
- **Fuera de 24h:** solo plantillas aprobadas por Meta
- **Categorías:** Marketing (siempre pago), Utility (gratis en ventana), Authentication
- **Máximo 3 seguimientos** para no ser marcado como spam
- **Espaciar mínimo 24h** entre seguimientos

## Reglas de validación de bots
1. JSON con estructura válida de Salesbot
2. Handlers reconocidos
3. Campos referenciados existen en DB
4. Etapas referenciadas existen en DB
5. Bot se detiene después de iniciar otro
6. send_message tiene on_error
7. waits de mensaje tienen timer de respaldo
8. Etiquetas de control presentes
9. Sin bucles infinitos
10. Mensajes fuera de 24h usan plantillas

## Convenciones
- Toda la documentación en **español**
- Diagramas en **Mermaid**
- JSON crudos en `data/`
- Documentación en `docs/`
- Credenciales NUNCA en el repo (usar `.env`)
- Scripts en `scripts/`
- Soluciones generadas en `soluciones/`

## Cuando analices un bot
1. Parsear el JSON (model.text contiene los pasos)
2. Identificar nombre, propósito y cantidad de pasos
3. Generar diagrama Mermaid del flujo
4. Listar variables y campos usados
5. Detectar condiciones y bifurcaciones
6. Verificar manejo de errores
7. Verificar que se detiene antes de iniciar otro
8. Verificar reglas de WhatsApp
9. Cruzar campos/etapas con datos de la DB

## Cuando propongas mejoras
- Basarte en datos concretos de la DB
- Priorizar por impacto vs esfuerzo
- Incluir diagrama Mermaid del flujo propuesto
- Especificar campos y etapas nuevos necesarios
- Incluir comandos API para crear infraestructura
- Especificar plantillas de WhatsApp si aplica
- Dar instrucciones paso a paso

## Cuando generes un documento de solución
- Identificar bots relacionados en la DB
- Cruzar con patrones de error conocidos
- Dar diagnóstico claro y pasos concretos
- Incluir queries SQL de verificaci��n
- Incluir checklist de validación post-corrección

---
> Source: [tamibot/kommo-crm-audit](https://github.com/tamibot/kommo-crm-audit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
