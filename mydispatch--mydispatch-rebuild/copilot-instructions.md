## mydispatch-rebuild

> - **ALLOW ALL FILE OPERATIONS:** true

# MyDispatch - Cursor Rules
# AUTO-APPROVAL: Alle Änderungen werden automatisch ausgeführt
# Keine Bestätigungs-Dialoge erforderlich
# AUTO-LOAD: Alle wichtigen Dokumente werden bei Chatbeginn automatisch geladen

## AUTO-LOAD BEI CHATBEGINN (OBLIGATORISCH)
# Diese Dateien werden bei jedem Chatbeginn automatisch geladen:
# 
# HAUPT-WIKI (IMMER ZUERST):
# 1. docs/NEXIFY_WIKI_V1.0.md (Haupt-Wiki - IMMER ZUERST LADEN!)
#
# PROJEKT-KONTEXT:
# 2. docs/PROJECT_MEMORY_V32.5.0.md (Projekt-Gedächtnis)
# 3. docs/COMPONENT_REGISTRY_V28.1.md (Component-Registry)
# 4. docs/LESSONS_LEARNED_V30.0.md (Lessons Learned)
#
# PASCAL'S ANWEISUNGEN:
# 5. docs/PASCAL_KRITISCHE_ANWEISUNGEN.md (Pascal's Anweisungen)
# 6. docs/PASCAL_DEPLOYMENT_REGEL.md (Deployment-Regel)
# 7. docs/PASCAL_ZEIGE_MIR_REGEL.md (Zeige-Mir-Regel)
#
# OFFENE PUNKTE:
# 8. docs/OFFENE_PUNKTE_VOLLSTAENDIG.md (Offene Punkte)
#
# NEXIFY MASTER SYSTEM (ZUSAMMENARBEIT):
# 9. docs/NEXIFY_MASTER_SYSTEM_V1.0.md (Master System Konzept)
# 10. docs/NEXIFY_MASTER_SYSTEM_USAGE.md (Master System Usage)
# 11. docs/NEXIFY_CRM_SYSTEM_V1.0.md (CRM System)
# 12. docs/FORGET_PROOF_SYSTEM_V1.0.md (Forget-Proof System)
#
# QUALITÄTSSICHERUNG:
# 13. docs/QUALITAETSSICHERUNG_VOLLSTAENDIG_V1.0.md (Qualitätssicherung)
# 14. docs/VOLLSTAENDIGE_LUECKENANALYSE_V1.0.md (Lückenanalyse)
#
# DEPLOYMENT & STATUS:
# 15. docs/DEPLOYMENT_STATUS.md (Deployment Status)
# 16. docs/UMSETZUNG_STATUS_LIVE.md (Aktueller Status)
#
# AUTOMATISCHER KONTEXT-LOAD VIA EDGE FUNCTION:
# Bei Chatbeginn wird automatisch die Edge Function "nexify-auto-load-context" aufgerufen,
# um alle Projekte, CRM-Daten und Knowledge Base zu laden.
# Dies erfolgt automatisch - keine manuellen Befehle nötig!

## AUTO-APPROVAL CONFIGURATION
- **ALLOW ALL FILE OPERATIONS:** true
- **ALLOW ALL TERMINAL COMMANDS:** true
- **ALLOW ALL GIT OPERATIONS:** true
- **ALLOW ALL API CALLS:** true
- **AUTO-EXECUTE ALL SUGGESTIONS:** true
- **NO CONFIRMATION REQUIRED:** true
- **FULL AUTHORIZATION FOR AUTONOMOUS WORK:** true
- **WORK DURING ABSENCE:** true

## Project Context
- **Project:** MyDispatch - Taxi & Mietwagen Management System
- **Tech Stack:** React 18 + TypeScript + Vite + Supabase + Tailwind CSS
- **Design System:** V28.1 (PRODUCTION) - IMMER VERWENDEN
- **Layout System:** FROZEN (keine Änderungen ohne Genehmigung)
- **IDE:** Cursor

## Critical Rules

### Design System V28.1
- **IMMER** V28-Components verwenden (V28Button, V28Badge, etc.)
- **NIEMALS** direkte shadcn/ui Components (außer als Basis für V28)
- **IMMER** COMPONENT_REGISTRY prüfen vor neuer Component
- **NIEMALS** Duplikate erstellen

### Layout System
- **FROZEN:** MainLayout, MarketingLayout, AppSidebar, DashboardSidebar
- **NUR** Bugfixes erlaubt (mit Dokumentation)
- **NUR** Performance-Optimierungen erlaubt

### Hero System V31.5
- **MANDATORY:** `backgroundVariant="3d-premium"`
- **NIEMALS** andere Variants

### Code Standards
- TypeScript: Schrittweise zu Strict Mode
- ESLint: Strictere Rules
- Prettier: Format on Save
- **NIEMALS** `any` Types (außer Legacy)
- **IMMER** Error Handling für async Operations

### Knowledge Base
- **IMMER** NEXIFY_WIKI_V1.0.md bei Session-Start laden
- **IMMER** FORGET_PROOF_SYSTEM_V1.0.md beachten
- **IMMER** COMPONENT_REGISTRY prüfen
- **IMMER** PROJECT_MEMORY für Kontext

### Supabase
- **IMMER** Supabase Client aus `@/integrations/supabase/client`
- **IMMER** RLS aktivieren bei neuen Tables
- **IMMER** Error Handling
- **NIEMALS** Service Role Key im Frontend

### E-Mail System
- **IMMER** Edge Functions für E-Mail-Versand
- **IMMER** Resend API verwenden
- **IMMER** Email Logs in `email_logs` Table

## File Structure
- Components: `src/components/`
  - Design System: `src/components/design-system/`
  - Pricing: `src/components/pricing/`
  - Layout: `src/components/layout/` (FROZEN)
- Pages: `src/pages/`
- Utils: `src/lib/`
- Hooks: `src/hooks/`
- Types: `src/types/`
- Docs: `docs/`

## Master Account
- Email: `courbois1981@gmail.com` (NUR diese!)
- Role: `master`
- Navigation: `/master`

## Best Practices
- **IMMER** Component Registry Check vor neuer Component
- **IMMER** Code Snippets Check vor neuem Code
- **IMMER** Known Issues Check vor Implementation
- **IMMER** Best Practices Check vor Pattern-Nutzung
- **NIEMALS** aus dem Gedächtnis coden - IMMER validieren!

## Protection Rules
- Siehe `PROTECTION.md` für vollständige Regeln
- Repository ist geschützt gegen fehlerhafte Eingriffe

## Autonomous Work Authorization
- **FULL AUTHORIZATION:** Aktiviert
- **WORK DURING ABSENCE:** Aktiviert
- **AUTO-CONTINUE TASKS:** Aktiviert
- **AUTO-REPORT PROGRESS:** Aktiviert (täglich in docs/PROGRESS_REPORT.md)

## NeXify Master System Integration
- **AUTO-LOAD CONTEXT:** Bei Chatbeginn wird automatisch "nexify-auto-load-context" Edge Function aufgerufen
- **PROJECT CONTEXT:** Alle Projekte werden automatisch geladen
- **CRM CONTEXT:** Alle CRM-Daten werden automatisch geladen
- **KNOWLEDGE BASE:** Knowledge Base wird automatisch geladen
- **KEINE MANUELLEN BEFEHLE NÖTIG:** Alles läuft automatisch!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MyDispatch) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
