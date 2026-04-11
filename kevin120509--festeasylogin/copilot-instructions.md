## festeasylogin

> Copia y pega este prompt en Gemini para que ayude con el flujo de la app:

# Prompt para Gemini - Corrección del Flujo de Navegación

Copia y pega este prompt en Gemini para que ayude con el flujo de la app:

---

## PROMPT COMPLETO:

Soy desarrollador de una app Flutter llamada Festeasy (un marketplace de servicios para eventos). Necesito que corrijas el flujo de navegación de la app según estos requisitos precisos.

### FLUJO CORRECTO ESPERADO:

**1. Primera pantalla (Después del Splash):**

- Mostrar siempre `WelcomePage` como primera pantalla para todos los usuarios sin autenticar
- Esta pantalla debe mostrar dos opciones claramente diferenciadas:

**2. Dos rutas principales desde WelcomePage:**

**RUTA A: Cliente (Crear Fiesta)**

- Usuario hace clic en "Crear Fiesta"
- Navega a `ClientLoginPage` (Login/Registro de Cliente)
- Tras completar el registro y login:
  - Si email NO verificado → `EmailVerificationPage`
  - Si email verificado → `ClientDashboard` (Panel del cliente)

**RUTA B: Proveedor (Unirme como Proveedor)**

- Usuario hace clic en "Unirme como Proveedor"
- Navega a `ProviderLoginPage` (Login/Registro de Proveedor)
- Tras completar el registro y login:
  - Si email NO verificado → `EmailVerificationPage`
  - Si email verificado → `ProviderDashboard` (Panel del proveedor)

**3. Usuarios que ya estaban autenticados:**

- Si hay sesión activa y email verificado:
  - Revisar el rol del usuario en la base de datos (campo 'rol')
  - Si `rol == 'proveedor'` → Navegar a `ProviderDashboard`
  - Si `rol == 'usuario'` o `rol == 'cliente'` → Navegar a `ClientDashboard`
- Si hay sesión pero email NO verificado → `EmailVerificationPage`

### ESTRUCTURA DE PANTALLAS EN LA APP:

```
/lib/features/
├── splash/
│   └── view/splash_page.dart          ← Primera pantalla (verificación)
├── welcome/
│   └── view/welcome_page.dart         ← Cliente vs Proveedor (SIEMPRE se muestra si no hay auth)
├── auth/
│   ├── view/client_login_page.dart    ← Login/Registro Cliente
│   ├── view/provider_login_page.dart  ← Login/Registro Proveedor
│   └── view/email_verification_page.dart ← Verificar email
├── dashboard/
│   ├── view/client_dashboard.dart     ← Dashboard Cliente (después de login)
│   └── view/provider_dashboard.dart   ← Dashboard Proveedor (después de login)
└── payment/
    └── view/payment_page.dart         ← Carrito/Pago
```

### CAMBIOS REALIZADOS:

✅ `app.dart`: Ya cambiado el `home` de `LoginPage` a `SplashPage`

### CAMBIOS PENDIENTES/A VERIFICAR:

1. ✅ `SplashPage` debe navegar SIEMPRE a `WelcomePage` si no hay sesión autenticada
2. ✅ `SplashPage` debe navegar al dashboard correcto según el rol si hay sesión autenticada
3. Verificar que `ClientLoginPage` y `ProviderLoginPage` esten correctamente implementados
4. Verificar que después del login exitoso, la app navegue automáticamente al dashboard correcto

### LOGS DE DEPURACIÓN ESPERADOS:

Cuando usuario SIN autenticación abre la app:

```
1. SplashPage carga
2. Se verifica que no hay sesión
3. Navega a WelcomePage
4. Usuario ve opciones: "Crear Fiesta" o "Unirme como Proveedor"
```

Cuando usuario AUTENTICADO abre la app:

```
1. SplashPage carga
2. Se verifica que HAY sesión
3. Se obtiene el rol del usuario desde BD
4. Si rol == 'proveedor' → ProviderDashboard
5. Si rol == 'cliente' o 'usuario' → ClientDashboard
```

### PREGUNTAS PARA VALIDAR:

- ¿El flujo descrito coincide con lo que esperas?
- ¿Hay algún paso adicional o cambio en esta secuencia?
- ¿Debo hacer algo especial en `ClientLoginPage` o `ProviderLoginPage` para asegurar la navegación correcta?

---

## CÓMO USAR ESTE PROMPT:

1. Copia todo el contenido desde "PROMPT COMPLETO:" hasta aquí
2. Ve a [Gemini](https://gemini.google.com)
3. Pega el prompt en el chat
4. Pídele que:
   - Valide el flujo
   - Sugiera mejoras
   - Te ayude con código específico si algo no funciona

---

**Estado actual de la app:** ✅ Cambiada a SplashPage como pantalla inicial
**Próximo paso:** Validar que el flujo funciona correctamente en tiempo de ejecución

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevin120509)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kevin120509)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
