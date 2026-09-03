## minimee

> Copia local de pruebas de la tienda **minime lovers** (minime.com.co).

# minime lovers — Contexto del proyecto

## Qué es esto
Copia local de pruebas de la tienda **minime lovers** (minime.com.co).
Aquí se prueban cambios antes de subirlos a producción.
El sitio real está en Netlify. Para correrlo localmente: `netlify dev` → http://localhost:8888

## Stack
- HTML/CSS/JS puro (sin framework)
- Netlify Functions (carpeta `/netlify/functions/`) para backend
- Supabase como base de datos (pedidos, usuarios, stock)
- Bold como pasarela de pago
- EmailJS para correos de confirmación
- GSAP para animaciones

## Variables de entorno necesarias (ya configuradas en Netlify)
- `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`
- `BOLD_SECRET`, `BOLD_API_KEY`
- `SHEETS_URL` (Google Sheets webhook)
- `ADMIN_PASSWORD`

## Cambios ya aplicados — NO revertir
1. **Schema.org corregido**: `paymentAccepted` dice "Bold, WhatsApp" (antes decía "MercadoPago")
2. **Bug `closeAuth()` corregido**: el código de reset del formulario de registro estaba fuera de la función, ahora está dentro
3. **Verificación de email en WhatsApp eliminada**: el email es opcional en pedidos por WhatsApp, el bloqueo confundía a las clientas
4. **Seguridad admin**: la contraseña ya no viaja en la URL (`?pwd=`), ahora va en el header `x-admin-pwd`. Los archivos `get-pedidos.js` y `get-usuarios.js` aceptan ambos métodos por compatibilidad
5. **Menú hamburguesa móvil agregado**: en pantallas < 900px aparece un ícono de 3 líneas que abre un menú pantalla completa con animación. Estilos en el `<style>` del index.html, HTML justo después del `<nav>`, JS antes del bloque SPARKLE KEYFRAME
6. **`reset.html` creado**: página de nueva contraseña. Lee el `access_token` del hash de la URL, muestra formulario si es válido o mensaje de link expirado si no. Usa la anon key pública de Supabase para el PUT a `/auth/v1/user`

## Archivos principales
- `index.html` — tienda completa (productos, carrito, checkout, auth, animaciones)
- `admin.html` — panel de administración (pedidos, clientes, estadísticas, stock)
- `terminos.html` — términos, privacidad, envíos, garantías
- `netlify/functions/` — todas las funciones del backend

---
> Source: [iaaanmb/minimee](https://github.com/iaaanmb/minimee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-03 -->
