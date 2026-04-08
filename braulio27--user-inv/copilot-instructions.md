## user-inv

> Eres un asistente especializado en el **Sistema de Inventario de Usuarios** desarrollado para la Municipalidad de Carrillo. Este es un sistema web moderno construido con Next.js, React y Material-UI que permite gestionar usuarios y sus equipos tecnológicos asignados.

# Sistema de Inventario de Usuarios - Contexto para Gemini CLI

## 🎯 Descripción General

Eres un asistente especializado en el **Sistema de Inventario de Usuarios** desarrollado para la Municipalidad de Carrillo. Este es un sistema web moderno construido con Next.js, React y Material-UI que permite gestionar usuarios y sus equipos tecnológicos asignados.

## 🏗️ Arquitectura Técnica

### Stack Tecnológico
- **Frontend**: Next.js 15.4.2 con App Router
- **UI Framework**: Material-UI (MUI) 7.2.0
- **Lenguaje**: TypeScript 5 con configuración estricta
- **Estilos**: CSS Grid, Flexbox, Emotion
- **Estado**: React Hooks (useState, useMemo, useEffect)
- **Herramientas**: ESLint, Next.js Lint

### Estructura del Proyecto
```
src/
├── app/                    # Páginas de Next.js App Router
│   ├── layout.tsx         # Layout principal con metadatos
│   ├── page.tsx           # Dashboard principal
│   ├── login/             # Página de login (básica)
│   ├── users/             # Página de gestión de usuarios
│   │   └── page.tsx
│   └── config/            # Página de configuración (en construcción)
│       └── page.tsx
├── components/            # Componentes reutilizables
│   ├── userItem.tsx       # Tarjeta de usuario individual
│   └── UserModal.tsx      # Modal para crear/editar usuarios
├── layouts/               # Layouts específicos
│   └── LayoutPrivado.tsx  # Layout con navegación lateral
├── types/                 # Definiciones de tipos TypeScript
│   └── user.ts
└── data/                  # Datos mock y utilidades
    └── users.ts
```

## 📊 Modelo de Datos

### Interfaz UserData
```typescript
interface UserData {
  numeroEmpleado: string      // Identificador único
  nombreCompleto: string      // Nombre completo del empleado
  usuario: string            // Nombre de usuario del sistema
  edificio: string           // Ubicación física
  departamento: string       // Departamento asignado
  puesto: string             // Cargo o posición
  estadoEquipo: string       // 'Activo' | 'Inactivo' | 'En reparación'
  estadoUsuario: string      // 'Activo' | 'Inactivo' | 'En reparación'
  sistemaOperativo: string   // Windows 11, Windows 10, macOS, Linux
  serviceTag: string         // Identificador del equipo
  fabricante: string         // Dell, HP, Apple, Lenovo, etc.
  tipo: string              // Laptop, Desktop, Tablet
  modelo: string            // Modelo específico del equipo
  direccionMac: string      // Dirección MAC del equipo
}
```

## 🚀 Funcionalidades Implementadas

### ✅ Dashboard Principal (/)
- **Métricas en tiempo real**: Total de usuarios, usuarios activos, equipos activos, equipos en reparación
- **Usuarios recientes**: Lista de los últimos 5 usuarios
- **Resumen por departamento**: Estadísticas organizadas por departamento
- **Diseño responsive**: Adaptable a diferentes tamaños de pantalla

### ✅ Gestión de Usuarios (/users)
- **CRUD completo**: Crear, leer, actualizar y eliminar usuarios
- **Búsqueda en tiempo real**: Por nombre, usuario, departamento o número de empleado
- **Modal interactivo**: Formulario completo con validación
- **Validaciones robustas**: Campos obligatorios, formato MAC, números únicos
- **Notificaciones**: Feedback visual con Snackbar
- **Indicadores visuales**: Chips de colores para estados

### ✅ Navegación y Layout
- **Menú lateral**: Navegación entre secciones
- **Barra superior**: Título del sistema y avatar de usuario
- **Responsive**: Adaptable a móviles y desktop
- **Indicadores activos**: Resaltado de página actual

### ✅ Configuración (/config)
- **Página en construcción**: Mensaje informativo
- **Roadmap visual**: Lista de funcionalidades pendientes
- **Iconografía**: Indicadores visuales para cada función

## 🔧 Características Técnicas

### Validaciones Implementadas
- **Número de empleado**: Requerido y único
- **Campos obligatorios**: Nombre, usuario, edificio, departamento, puesto
- **Dirección MAC**: Formato válido (00:1B:44:11:3A:B7)
- **Validación en tiempo real**: Errores se muestran mientras el usuario escribe
- **Limpieza automática**: Errores se limpian al corregir

### Experiencia de Usuario
- **Accesibilidad**: Aria-labels, navegación por teclado
- **Feedback inmediato**: Notificaciones de éxito/error
- **Confirmaciones**: Antes de eliminar usuarios
- **Diseño intuitivo**: Interfaz clara y fácil de usar

### Performance
- **Memoización**: useMemo para filtros y cálculos
- **Lazy loading**: Componentes cargados bajo demanda
- **Optimización**: Build optimizado para producción

## 🔄 Funcionalidades Pendientes

### Configuración del Sistema
- [ ] Cambio de idioma (Español, Inglés)
- [ ] Modo nocturno / tema oscuro
- [ ] Sistema de notificaciones
- [ ] Configuración de seguridad y permisos
- [ ] Backup y restauración automática
- [ ] Personalización de interfaz
- [ ] Configuración de reportes

### Otras Funcionalidades
- [ ] Autenticación y autorización real
- [ ] Exportación de datos a CSV/Excel
- [ ] Paginación para listas grandes
- [ ] Filtros avanzados
- [ ] Historial de cambios
- [ ] API REST para persistencia
- [ ] Tests unitarios y de integración

## 🎨 Diseño y UX

### Principios de Diseño
- **Material Design**: Siguiendo las guías de Material-UI
- **Responsive First**: Diseño móvil primero
- **Accesibilidad**: Cumpliendo estándares WCAG
- **Consistencia**: Patrones de diseño uniformes

### Paleta de Colores
- **Primario**: Azul Material-UI
- **Secundario**: Verde para estados activos
- **Error**: Rojo para errores y estados inactivos
- **Advertencia**: Naranja para equipos en reparación

## 📝 Convenciones de Código

### TypeScript
- **Configuración estricta**: `strict: true`
- **Tipos explícitos**: Interfaces bien definidas
- **Sin any**: Uso de tipos específicos

### React
- **Functional Components**: Hooks modernos
- **Props tipadas**: Interfaces para props
- **Estado local**: useState para estado simple
- **Efectos controlados**: useEffect con dependencias

### Estilos
- **Material-UI sx**: Estilos inline cuando sea necesario
- **CSS Grid**: Para layouts complejos
- **Flexbox**: Para alineaciones simples
- **Responsive**: Breakpoints xs, sm, md, lg, xl

## 🚀 Comandos de Desarrollo

```bash
# Instalación
npm install

# Desarrollo
npm run dev

# Construcción
npm run build

# Producción
npm start

# Linting
npm run lint

# Verificación de tipos
npx tsc --noEmit
```

## 🎯 Contexto de Uso

Este sistema está diseñado para:
1. **Gestión de inventario**: Control de equipos tecnológicos
2. **Administración de usuarios**: Gestión de empleados y sus recursos
3. **Reportes**: Generación de estadísticas y métricas
4. **Auditoría**: Seguimiento de cambios y estados

El sistema es escalable y está preparado para integrarse con:
- Sistemas de autenticación corporativos
- Bases de datos empresariales
- APIs de terceros
- Sistemas de notificaciones

## 💡 Notas Importantes

- **Datos mock**: Actualmente usa datos de ejemplo en `src/data/users.ts`
- **Sin persistencia**: Los cambios se pierden al recargar la página
- **Sin autenticación**: Acceso directo sin login
- **Desarrollo activo**: Funcionalidades en constante evolución

Este contexto te permite entender completamente el sistema y proporcionar asistencia técnica precisa para cualquier consulta o mejora que se requiera.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Braulio27)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Braulio27)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
