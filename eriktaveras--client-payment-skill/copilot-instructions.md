## client-payment-skill

> Flujo completo de cobro a clientes. Se activa cuando el usuario menciona enviar un link de pago, cobrar a un cliente, crear cotizacion, generar payment link, facturar, o cualquier variacion de 'voy a cobrarle a [cliente]'. Tambien se activa con frases como 'nuevo cliente', 'enviar cotizacion', 'mandar link de pago', 'cobrar proyecto', 'facturar servicio'. Usa este skill incluso si el usuario solo menciona una parte del flujo (ej: solo cotizacion, solo link de pago, solo email). SIEMPRE usar este skill cuando se mencione Stripe en contexto de cobrar o enviar links de pago.


# Client Payment Link - Flujo de Cobro a Clientes

Este skill maneja el flujo completo de cobro a clientes: desde la recopilacion de informacion del cliente, creacion de productos en Stripe, generacion de links de pago, creacion de cotizaciones en PDF, hasta el envio del email al cliente.

## Flujo Principal

Cuando el usuario indica que va a cobrarle a un cliente o enviar un link de pago, sigue estos pasos en orden:

### Paso 1: Recopilar Informacion del Cliente

Usa la herramienta `AskUserQuestion` para obtener toda la informacion necesaria. Haz las preguntas de forma conversacional y agrupa lo que puedas. Necesitas:

**Informacion obligatoria:**
- Nombre del cliente (persona o empresa)
- Email del cliente
- Servicio o producto a cobrar
- Monto total (y moneda si no es USD)

**Preguntas adicionales que SIEMPRE debes hacer:**
- Necesitas que le haga una cotizacion formal en PDF?
- Quieres que le envie el email con el link de pago directamente?
- Hay alguna nota especial o condiciones para este cliente?
- Es un pago unico o recurrente (mensual, anual)?

### Paso 2: Crear el Producto y Link de Pago en Stripe

Usa las herramientas MCP de Stripe en este orden:

1. **Buscar si el producto ya existe** usando `list_products` o `search_stripe_resources`
2. **Si no existe, crear el producto** con `create_product`:
   - Nombre descriptivo del servicio
   - Descripcion clara
3. **Crear el precio** con `create_price`:
   - Asociado al producto
   - `unit_amount` en centavos (ej: $500 = 50000)
   - `currency`: "usd" por defecto
   - Si es recurrente, incluir `recurring` con el intervalo
4. **Generar el payment link** con `create_payment_link`:
   - Usar el price ID del paso anterior
   - Quantity: 1 (a menos que se indique otra cosa)

Presenta al usuario el link generado antes de continuar.

### Paso 3: Crear Carpeta del Cliente

Crea una carpeta en el workspace del usuario con el nombre del cliente:

```
{workspace}/Clientes/{NombreCliente}/
```

Si la carpeta ya existe (cliente recurrente), simplemente agrega los nuevos documentos sin borrar los existentes.

### Paso 4: Generar Cotizacion PDF (si aplica)

Si el usuario quiere cotizacion, usa el script `scripts/create_quote_pdf.py` que esta en el directorio de este skill.

Pasos:
1. Crea un archivo JSON temporal con los datos de la cotizacion
2. Ejecuta el script apuntando al JSON y a la carpeta del cliente como output
3. El PDF incluira automaticamente: nombre de empresa, datos del cliente, tabla de items, totales, y el link de pago

Formato del JSON de configuracion:
```json
{
    "client_name": "Nombre del Cliente",
    "client_email": "email@cliente.com",
    "quote_number": "QT-2026-001",
    "date": "2026-03-06",
    "valid_until": "2026-04-06",
    "items": [
        {"description": "Desarrollo Web", "quantity": 1, "unit_price": 5000.00}
    ],
    "currency": "USD",
    "tax_rate": 0,
    "notes": "Pago requerido dentro de 30 dias.",
    "payment_link": "https://buy.stripe.com/xxx",
    "document_type": "COTIZACION"
}
```

Para numeros de cotizacion, usa el formato QT-{AÑO}-{NUMERO_SECUENCIAL}. Revisa la carpeta de Clientes para determinar el siguiente numero.

### Paso 5: Redactar y Enviar Email

Redacta un email profesional. El tono debe ser profesional pero cercano, en el idioma que el usuario use normalmente con ese cliente.

**Estructura del email:**
- **Asunto**: Claro y directo (ej: "Cotizacion - [Servicio] | Tu Empresa")
- **Saludo**: Personalizado con el nombre del cliente
- **Cuerpo**: Breve descripcion del servicio, monto, y link de pago
- **Si hay cotizacion**: Mencionar que va adjunta
- **CTA**: Link de pago claro y visible
- **Firma**: Datos de tu empresa

Para enviar, usa el script `scripts/send_email.py`:

```bash
python {skill_path}/scripts/send_email.py \
  --to "cliente@email.com" \
  --subject "Cotizacion - Servicio | Tu Empresa" \
  --body "Texto plano del email" \
  --html "<html>Version HTML</html>" \
  --attachment "/path/to/cotizacion.pdf"
```

**IMPORTANTE**: Antes de enviar, SIEMPRE muestra al usuario el borrador del email y pide confirmacion explicita. Nunca envies un email sin aprobacion del usuario.

### Paso 6: Resumen Final

Al terminar, presenta al usuario un resumen claro:
- Link de pago de Stripe (copiable)
- Documentos generados y donde estan guardados
- Confirmacion de envio de email (si aplica)
- Proximos pasos sugeridos

## Datos de la Empresa

> **PERSONALIZA ESTO** con los datos de tu empresa.

- **Nombre**: Tu Empresa, LLC
- **Email**: hello@tuempresa.com
- **Moneda por defecto**: USD

## Configuracion de Email

El skill usa `.email_config.json` en la carpeta de trabajo para enviar emails via SMTP. Si no existe o la contrasena no esta configurada:
1. Informa al usuario
2. Ofrece crear/actualizar el archivo
3. Como alternativa, genera el borrador para copiar/pegar manualmente

Ver `examples/email_config_example.json` para el formato.

## Notas Importantes

- Siempre confirma montos y datos antes de crear nada en Stripe
- Los montos en Stripe van en centavos: $100 = 10000
- Nunca envies emails sin aprobacion explicita
- Si el cliente es recurrente, busca primero si existe en Stripe
- Guarda todos los documentos en la carpeta del cliente

---
> Source: [eriktaveras/Client-Payment-Skill](https://github.com/eriktaveras/Client-Payment-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
