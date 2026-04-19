## precios

> - Siempre que se genere un gráfico desde cualquier script, el formato de salida por defecto debe ser SVG (.svg), salvo que no sea técnicamente posible.

# Reglas para scripts y generación de gráficos en este proyecto

- Siempre que se genere un gráfico desde cualquier script, el formato de salida por defecto debe ser SVG (.svg), salvo que no sea técnicamente posible.
- Si se requiere otro formato, debe justificarse en el script o documentación correspondiente. 

## Documentación

### Estructura de Documentación
- Documentación principal en directorio `documentacion/` en la raíz del proyecto
- Crear subdirectorios para cada subproyecto: `documentacion/{subproyecto}/`
- Mantener README.md en la raíz del proyecto
- Crear README.md específico en cada `documentacion/{subproyecto}/`

### Documentos Requeridos por Subproyecto
- **README.md**: Descripción general del subproyecto, propósito y estructura
- **arquitectura.md**: Diseño arquitectónico detallado del subproyecto
- **definicion_tecnica.md**: Especificación técnica exhaustiva y abstracta que permita reconstruir el proyecto en otra tecnología
- **endpoints.md**: Documentación completa de todos los endpoints (solo para APIs)

### Enlaces y Referencias
- Todos los documentos deben estar interconectados mediante enlaces
- Mantener navegación coherente entre documentos
- Usar referencias cruzadas cuando sea apropiado

### Estándares de Documentación
- Usar Markdown para todos los documentos
- Incluir diagramas cuando sea necesario (usar Mermaid o similar)
- Mantener ejemplos de código actualizados
- Documentar configuraciones de deployment
- Incluir guías de instalación y setup
- Documentar APIs con Postman collections
- Comentar código complejo con documentación inline

### Documentación de Endpoints (endpoints.md)
- **Estructura obligatoria** para cada endpoint:
  - Título con método HTTP y ruta
  - Descripción clara del propósito
  - Headers requeridos (especialmente Authorization)
  - Parámetros de query (si aplica)
  - Body de request con ejemplo JSON
  - Respuesta exitosa (200/201) con ejemplo JSON
  - Todas las respuestas de error posibles (400, 401, 403, 404, 422, 500, etc.)
  - Características técnicas del endpoint (autenticación, permisos, etc.)

- **Formato estándar**:
  ```markdown
  ### X.Y Nombre del Endpoint
  **METHOD** `/ruta/endpoint`
  
  Descripción del endpoint.
  
  #### Headers
  ```
  Authorization: Bearer <token>
  Content-Type: application/json
  ```
  
  #### Parámetros
  ```json
  {
    "campo": "valor"
  }
  ```
  
  #### Respuesta Exitosa (200)
  ```json
  {
    "success": true,
    "data": {}
  }
  ```
  
  #### Respuesta de Error (400)
  ```json
  {
    "success": false,
    "message": "Descripción del error"
  }
  ```
  
  #### Características del Endpoint
  - **Autenticación**: Requerida/No requerida (especificar roles si aplica)
  - **Permisos**: Roles específicos si aplica
  - **Validación**: Tipos de validación aplicados
  - **Transaccional**: Si usa transacciones de BD
  - **Rate Limiting**: Límites específicos si aplica
  ```

- **Secciones obligatorias** en endpoints.md:
  - Información General (Base URL, autenticación, formatos)
  - Endpoints agrupados por funcionalidad (Autenticación, Usuarios, etc.)
  - Códigos de Error (HTTP y específicos del sistema)
  - Ejemplos de uso (flujos completos)
  - Navegación entre documentos

## Git y Versionado
- Usar commits descriptivos de hasta 255 caracteres
- Seguir convenciones de branching
- Mantener .gitignore actualizado
- No committear archivos sensibles
- **REGLAS DE COMMIT**: 
  - Antes de commitear cambios en endpoints, verificar que la documentación esté actualizada
  - Incluir documentación actualizada en el mismo commit que los cambios de código
  - Si se acepta una funcionalidad, actualizar documentación y luego proceder con commit

## Performance
- Optimizar consultas de base de datos
- Implementar caching cuando sea apropiado
- Minimizar dependencias innecesarias
- Monitorear rendimiento de APIs

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Greenborn) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
