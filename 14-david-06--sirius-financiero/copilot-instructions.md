## sirius-financiero

> > Archivo leído automáticamente por Claude Code CLI en cada sesión. Documenta el proyecto para todos los agentes de desarrollo.

# CLAUDE.md — Sirius Financiero

> Archivo leído automáticamente por Claude Code CLI en cada sesión. Documenta el proyecto para todos los agentes de desarrollo.

## Stack Tecnológico

- **Framework**: Next.js 15.4.8 con App Router (monorepo — sin separación backend/frontend)
- **React**: 19.1.2
- **TypeScript**: 5.x (strict mode)
- **Estilos**: Tailwind CSS 4 con PostCSS, glass-morphism UI (degradado azul-púrpura)
- **Base de datos**: Airtable (Base principal financiera + Base Pirolisis para producción)
- **AI**: OpenAI API (GPT-4o-mini para agentes, Whisper-1 para transcripción)
- **Auth**: JWT custom (jsonwebtoken), bcryptjs para hashing
- **Cloud**: AWS S3 (documentos), OneDrive/Microsoft Graph (facturas)
- **Integración**: n8n webhooks para procesamiento automático de facturas
- **Charts**: Recharts para visualizaciones

## Estructura del Monorepo

```
src/
├── app/
│   ├── api/                    # Backend — Route handlers (Next.js App Router)
│   │   ├── authenticate/       # Login: cédula + password → JWT
│   │   ├── check-session/      # Validar sesión activa
│   │   ├── logout/             # Cerrar sesión
│   │   ├── setup-password/     # Configurar password inicial
│   │   ├── validate-user/      # Validar usuario existente
│   │   │
│   │   ├── compras/            # CRUD solicitudes de compra
│   │   ├── solicitudes-compra/ # Crear nueva solicitud
│   │   ├── cotizaciones/       # Guardar/extraer cotizaciones
│   │   ├── proveedores/        # Base de proveedores
│   │   ├── catalogo-insumos/   # Catálogo de insumos
│   │   ├── items/              # Items de solicitudes
│   │   ├── generate-orden-compra/
│   │   │
│   │   ├── caja-menor/         # CRUD caja menor
│   │   ├── caja-menor-agent/   # Agente IA GPT-4o-mini
│   │   ├── caja-menor-analisis/
│   │   ├── consolidar-caja-menor/
│   │   │
│   │   ├── facturacion-ingresos/
│   │   ├── facturacion-egresos-callback/   # Webhook n8n
│   │   ├── facturacion-stream/             # SSE streaming
│   │   ├── monitoreo-facturas/
│   │   │
│   │   ├── movimientos-bancarios/          # Multi-banco
│   │   ├── movimientos-bancarios-bancolombia/
│   │   ├── movimientos-bancarios-bbva/
│   │   │
│   │   ├── proyecciones/       # Simulaciones financieras
│   │   ├── indicadores-produccion/
│   │   ├── balances-masa/      # Pirolisis
│   │   ├── centralizacion-general/
│   │   │
│   │   ├── chat-compras/       # Chat en tiempo real
│   │   ├── transcribe-audio/   # Whisper transcripción
│   │   ├── clasificar-item-ia/ # Clasificación automática
│   │   │
│   │   ├── upload-onedrive/    # Microsoft Graph API
│   │   ├── upload-file/        # AWS S3
│   │   └── upload-factura-onedrive/
│   │
│   ├── solicitudes-compra/     # Frontend — Páginas
│   ├── monitoreo-solicitudes/
│   ├── mis-solicitudes-compras/
│   ├── caja-menor/
│   ├── movimientos-bancarios/
│   ├── facturacion-ingresos/
│   ├── facturacion-egresos/
│   ├── resumen-gerencial/
│   ├── indicadores-produccion/
│   ├── simulador-proyecciones/
│   ├── monitoreo-facturas/
│   ├── flujo-caja/
│   ├── analisis-rentabilidad/
│   ├── layout.tsx              # Root layout
│   └── globals.css             # Tailwind 4
│
├── components/                 # Componentes React
│   ├── SolicitudesCompra.tsx   # Crear solicitud + audio
│   ├── MisSolicitudesCompras.tsx
│   ├── MonitoreoSolicitudes.tsx
│   ├── DashboardCompras.tsx    # Panel admin compras
│   ├── CajaMenor.tsx           # Gestión caja menor
│   ├── CajaMenorAgent.tsx      # Chat IA + voz
│   ├── FacturacionIngresos.tsx
│   ├── FacturacionEgresos.tsx
│   ├── MovimientosBancarios.tsx
│   ├── ResumenGerencial.tsx    # Dashboard KPIs
│   ├── IndicadoresProduccion.tsx
│   ├── SimuladorProyecciones.tsx
│   ├── MonitoreoFacturas.tsx
│   ├── ChatCompra.tsx          # Chat solicitudes
│   ├── LoginComponent.tsx      # Login multi-step
│   ├── RoleGuard.tsx           # Control de acceso
│   ├── layout/
│   │   ├── Navbar.tsx          # Navegación principal
│   │   └── Footer.tsx
│   └── ui/
│       ├── Button.tsx          # Variantes: primary, secondary, outline
│       ├── Card.tsx            # Glass-morphism cards
│       └── Toast.tsx           # Notificaciones
│
├── lib/
│   ├── airtable.ts             # Conexión base Airtable
│   ├── security.ts             # Rate limiting, sanitización
│   ├── stream-manager.ts       # Manejo de SSE streams
│   ├── config/
│   │   └── airtable-fields.ts  # Mapeo dinámico de campos
│   ├── hooks/
│   │   ├── useAuthSession.ts   # Manejo de sesión
│   │   ├── useNotifications.ts # Notificaciones push + toast
│   │   └── useMessagePolling.ts # Polling de mensajes
│   ├── utils/
│   └── security/
│
├── types/
│   ├── compras.ts              # Tipos de compras, items, cotizaciones
│   └── speech-recognition.ts   # Web Speech API types
│
└── middleware.ts               # JWT verification + RBAC
```

## Convenciones

- **Idioma**: Español colombiano en UI, comentarios y mensajes de agentes
- **Path alias**: `@/*` → `./src/*`
- **API pattern**: GET/POST/PUT/DELETE en un solo `route.ts` por recurso
- **Auth**: Cookie `auth-token` (24h), middleware verifica JWT
- **RBAC**: 4 categorías — Desarrollador > Gerencia > Administrador > Colaborador
- **Tables Airtable**: Nombres de campos en variables de entorno (no hardcoded)
- **SSE streaming**: `text/event-stream` para facturación y procesos largos
- **Seguridad**: Sanitización de inputs, rate limiting, headers de seguridad

## Patrones Clave del Código

### Autenticación JWT
```typescript
// middleware.ts
const token = request.cookies.get('auth-token')?.value;
const decoded = jwt.verify(token, JWT_SECRET) as {
  cedula: string;
  nombre: string;
  categoria: 'Colaborador' | 'Administrador' | 'Gerencia' | 'Desarrollador';
};
```

### Verificación de Roles
```typescript
// RoleGuard.tsx - Control de acceso por categoría
const allowedCategories = ['Administrador', 'Gerencia', 'Desarrollador'];
if (!allowedCategories.includes(decoded.categoria)) {
  return NextResponse.redirect(new URL('/', request.url));
}
```

### Conexión Airtable
```typescript
// src/lib/airtable.ts
import Airtable from 'airtable';
const base = new Airtable({ apiKey: process.env.AIRTABLE_API_KEY })
  .base(process.env.AIRTABLE_BASE_ID);
```

### Rate Limiting
```typescript
// src/lib/security.ts
SecurityMiddleware.checkRateLimit(request); // 10 req/60s
```

### SSE Streaming Pattern
```typescript
// Para procesos largos (facturación, IA)
return new Response(stream, {
  headers: {
    "Content-Type": "text/event-stream",
    "Cache-Control": "no-cache",
    "Connection": "keep-alive",
  },
});
```

### Hook de Sesión
```typescript
// src/lib/hooks/useAuthSession.ts
const { isAuthenticated, userData, login, logout } = useAuthSession();
```

## Tablas Airtable Principales

| Tabla | Variable de Entorno | Propósito |
|-------|-------------------|----------|
| Equipo Financiero | AIRTABLE_TEAM_TABLE_NAME | Usuarios y autenticación |
| Compras y Adquisiciones | AIRTABLE_COMPRAS_TABLE_ID | Solicitudes de compra |
| Items | AIRTABLE_ITEMS_TABLE_ID | Items de cada solicitud |
| Cotizaciones | AIRTABLE_COTIZACIONES_TABLE_ID | Propuestas de proveedores |
| Items Cotizados | AIRTABLE_ITEMS_COTIZADOS_TABLE_ID | Detalle de cotizaciones |
| Caja Menor | CAJA_MENOR_TABLE_ID | Anticipos mensuales |
| Items Caja Menor | ITEMS_CAJA_MENOR_TABLE_ID | Gastos individuales |
| Movimientos Bancarios | AIRTABLE_MOVIMIENTOS_TABLE_ID | Transacciones bancarias |
| Facturación Ingresos | AIRTABLE_INGRESOS_TABLE_ID | Ventas/Ingresos |
| Conversaciones | AIRTABLE_CONVERSACIONES_TABLE_ID | Chat solicitudes |
| Proveedores | AIRTABLE_PROVEEDORES_TABLE_ID | Base de proveedores |
| Proyecciones | AIRTABLE_PROYECCIONES_TABLE_ID | Simulaciones financieras |

## Integraciones Externas

### Microsoft OneDrive (Graph API)
```
Credenciales: ADM_MICROSOFT_CLIENT_ID, ADM_MICROSOFT_CLIENT_SECRET, ADM_MICROSOFT_TENANT_ID
Uso: Almacenamiento de facturas y documentos
Rutas: /General/Documentos Soporte/{año}/Movimientos {banco}/
```

### AWS S3
```
Credenciales: AWS_ACCESS_KEY_ID, AWS_SECRET_ACCESS_KEY, AWS_S3_BUCKET
Uso: PDFs de caja menor, documentos consolidados
```

### OpenAI
```
Credencial: OPENAI_API_KEY
Modelos: GPT-4o-mini (caja-menor-agent), Whisper-1 (transcribe-audio)
```

### n8n Webhooks
```
Flujo: PDF en OneDrive → Webhook n8n → OCR/AI → Callback API
Endpoints: /api/facturacion-callback, /api/facturacion-stream
```

## Comandos de Desarrollo

```bash
npm run dev       # Servidor de desarrollo
npm run build     # Build de producción
npm run lint      # ESLint
npm run security:check   # Verificar seguridad
npm run security:clean   # Limpiar logs sensibles
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/14-David-06)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/14-David-06)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
