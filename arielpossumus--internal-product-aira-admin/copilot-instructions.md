## internal-product-aira-admin

> - Usar vite.config.js para configuración principal

- type: build-tool

  value: vite

  rules: >
    # Configuración Base
    - Usar vite.config.js para configuración principal
    - Mantener configuraciones separadas por ambiente (dev/prod)
    - Definir alias de paths para imports más limpios
    - Configurar variables de entorno con .env files
    - Optimizar splitting de código

    # Desarrollo
    - Usar HMR (Hot Module Replacement)
    - Habilitar source maps en desarrollo
    - Configurar proxy para APIs en desarrollo
    - Usar plugins oficiales de Vite cuando sea posible
    - Mantener modo desarrollo con npm run dev

    # Producción
    - Optimizar bundles para producción
    - Implementar code splitting
    - Configurar compresión de assets
    - Habilitar tree shaking
    - Usar build command con npm run build

    # Optimización
    - Configurar lazy loading de módulos
    - Implementar dynamic imports
    - Optimizar tamaño de imágenes
    - Configurar caching apropiado
    - Minimizar JavaScript y CSS

    # Plugins
    - @vitejs/plugin-react para soporte React
    - vite-plugin-eslint para linting
    - vite-plugin-pwa para PWA support
    - @rollup/plugin-alias para alias
    - vite-plugin-compression para compresión

    # Assets
    - Usar import.meta.url para URLs dinámicas
    - Optimizar carga de assets estáticos
    - Configurar public/ para archivos sin proceso
    - Implementar lazy loading de imágenes
    - Usar formatos modernos de imagen

    # Performance
    - Configurar chunks óptimos
    - Implementar preloading
    - Optimizar First Paint
    - Reducir tamaño del bundle
    - Monitorear métricas de performance

    # Testing
    - Configurar Vitest para unit testing
    - Implementar testing-library
    - Configurar coverage reports
    - Mantener tests en modo watch
    - Integrar con CI/CD

    # Debugging
    - Usar source maps en desarrollo
    - Configurar error overlay
    - Implementar logging apropiado
    - Habilitar devtools
    - Mantener error handling

    # Seguridad
    - Configurar CSP headers
    - Implementar sanitización de inputs
    - Proteger contra XSS
    - Configurar CORS apropiadamente
    - Mantener dependencias actualizadas

    # Deployment
    - Configurar build para diferentes ambientes
    - Optimizar assets para producción
    - Implementar CI/CD pipelines
    - Configurar preview builds
    - Mantener scripts de deploy

    # Mantenimiento
    - Actualizar Vite regularmente
    - Mantener plugins actualizados
    - Limpiar cache periódicamente
    - Monitorear tamaño de bundles
    - Documentar configuraciones

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/arielpossumus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
