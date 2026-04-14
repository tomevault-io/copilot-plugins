## chitidocs

> Este documento proporciona a los agentes de IA una visión general completa de la arquitectura, flujos, gestión de roles y estado actual del repositorio **EduDoc Perú**. Debe ser utilizado como referencia principal antes de realizar modificaciones en el código.

# EduDoc Perú - Guía para Agentes IA (AGENTS.md)

Este documento proporciona a los agentes de IA una visión general completa de la arquitectura, flujos, gestión de roles y estado actual del repositorio **EduDoc Perú**. Debe ser utilizado como referencia principal antes de realizar modificaciones en el código.

## 1. Arquitectura y Stack Tecnológico

*   **Frontend:** React 18, Vite, TypeScript.
*   **Estilos:** Tailwind CSS, `lucide-react` para iconos, `motion/react` para animaciones.
*   **Backend & Base de Datos:** Firebase (Firestore para base de datos, Auth para autenticación con Google).
*   **Almacenamiento de Archivos:** Cloudflare R2 (gestionado a través de `src/services/storageService.ts`).
*   **Inteligencia Artificial:** Google Gemini API (`@google/genai`) para análisis de documentos (extracción de metadatos) y búsqueda semántica (RAG).

## 2. Estructura del Proyecto

```text
/src
  /components
    Dashboard.tsx        # Vista principal con estadísticas y trámites recientes.
    DocumentLibrary.tsx  # Gestión de documentos e importación desde Google Drive.
    DocumentSearch.tsx   # Búsqueda semántica usando Gemini (RAG).
    DrivePicker.tsx      # Componente modal para seleccionar archivos de Google Drive.
    MesaDePartes.tsx     # Formulario para registrar nuevos trámites y subir archivos.
    TramiteList.tsx      # Listado y gestión de estados de trámites.
    UserManagement.tsx   # Administración de roles de usuarios.
  /lib
    utils.ts             # Utilidades (ej. `cn` para clases de Tailwind).
  /services
    geminiService.ts     # Integración con Gemini para análisis y RAG.
    storageService.ts    # Subida de archivos a Cloudflare R2.
  /types
    index.ts             # Definición de interfaces (UserRole, TramiteStatus, Documento, etc.).
  App.tsx                # Enrutador principal, gestión de estado de autenticación y Layout.
  firebase.ts            # Inicialización de Firebase.
  main.tsx               # Punto de entrada de React.
```

## 3. Gestión de Roles y Permisos (RBAC)

Los roles están definidos en `src/types/index.ts` y protegidos tanto en el Frontend (`App.tsx`) como en el Backend (`firestore.rules`).

### Roles Disponibles:
*   `superadmin`, `director`, `subdirector`, `mesa_partes`, `secretaria`, `jefe_area`, `docente`, `auditor`.

### Permisos por Rol (Firestore Rules & UI):
*   **Superadmin / Director:**
    *   Acceso total.
    *   Pueden ver la vista de "Usuarios" (`UserManagement.tsx`) y cambiar roles.
    *   Pueden eliminar documentos y trámites.
*   **Mesa de Partes / Subdirector / Secretaria:**
    *   Pueden ver la vista "Mesa de Partes".
    *   Pueden crear y actualizar trámites y documentos.
*   **Docente / Jefe de Área:**
    *   Rol operativo asignado por Dirección.
    *   Solo pueden actualizar trámites que les han sido asignados explícitamente (`resource.data.asignadoAId == request.auth.uid`).
*   **Auditor:**
    *   Rol por defecto para nuevos usuarios.
    *   Sin acceso operativo hasta que Dirección asigne un rol funcional.

*Nota: El usuario `mark.romero.dev@gmail.com` tiene el rol de `director` asignado por defecto en el código y en las reglas de Firestore para propósitos de bootstrapping.*

## 4. Flujos de la Aplicación

1.  **Autenticación:** El usuario ingresa con Google (`signInWithPopup`). Si es su primera vez, se crea un registro en la colección `users` de Firestore con el rol `auditor`.
2.  **Mesa de Partes (Ingreso):** El personal autorizado registra un trámite, sube un archivo (PDF/Imagen), el cual se sube a R2. Gemini analiza el archivo para extraer metadatos automáticamente. Se crean registros en las colecciones `tramites` y `documents`.
3.  **Gestión de Trámites:** En la vista "Trámites", los usuarios pueden filtrar por estado, ver los documentos adjuntos, cambiar el estado del trámite (ej. de "registrado" a "en_evaluacion") y derivarlo (asignarlo) a otro usuario.
4.  **Biblioteca e Importación (Drive):** Los usuarios pueden ver todos los documentos. Pueden usar el botón "Conectar Drive" que utiliza la API de Google Drive para listar archivos, descargarlos, subirlos a R2, analizarlos con Gemini y guardarlos en Firestore.
5.  **Búsqueda Inteligente (RAG):** El usuario escribe una consulta natural. El sistema utiliza Gemini para buscar semánticamente entre los metadatos y el contenido de los documentos almacenados.

## 5. Estado Actual y Validaciones

*   **100% Libre de Mocks:** La aplicación no utiliza datos falsos (mocks). Todas las vistas (`Dashboard`, `TramiteList`, `DocumentLibrary`, `UserManagement`, `DocumentSearch`) están conectadas en tiempo real a Firestore usando `onSnapshot`.
*   **Integración con Google Drive:** Completamente funcional. Utiliza el flujo OAuth nativo de Firebase (`signInWithPopup` con el scope de Drive) para obtener el token, listar archivos y descargarlos.
*   **Reglas de Seguridad (Firestore):** Validadas y estrictas. Previenen la escalada de privilegios (un usuario no puede asignarse el rol de admin a sí mismo).
*   **Almacenamiento:** Conectado a R2.
*   **IA:** Conectado a Gemini 3.1 Pro/Flash.

## 6. Directrices para Futuros Agentes

1.  **No usar Mocks:** Cualquier nueva funcionalidad debe conectarse directamente a Firestore o a la API correspondiente.
2.  **Respetar RBAC:** Si creas una nueva vista o acción, asegúrate de ocultarla en la UI para roles no autorizados y de actualizar `firestore.rules` para protegerla en el backend.
3.  **Manejo de Errores:** Utiliza bloques `try/catch` y muestra feedback visual al usuario (ej. alertas o toasts) cuando una operación de red falle.
4.  **Escalabilidad:** Mantén las consultas a Firestore optimizadas (usa `limit`, `orderBy`, `where`) y recuerda crear los índices compuestos en la consola de Firebase si Firestore arroja un error de índice faltante.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcksdbgg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
