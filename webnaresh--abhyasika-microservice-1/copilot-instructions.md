## whatsapp-messaging

> When sending WhatsApp messages, always use the [WhatsappBodyDto](mdc:src/utils/whatsapp/dto/whatsapp_body.dto.ts) class to construct the message body.

# WhatsApp Messaging Standards

## WhatsappBodyDto Usage

When sending WhatsApp messages, always use the [WhatsappBodyDto](mdc:src/utils/whatsapp/dto/whatsapp_body.dto.ts) class to construct the message body.

### Required Usage Pattern

```typescript
// Always use the WhatsappBodyDto constructor with required parameters
const whatsappBody = new WhatsappBodyDto(
  'template_name',       // Template name - REQUIRED
  'receiver_phone_number' // Receiver number - REQUIRED
)
  .addBodyComponent([    // Add components as needed
    { type: 'text', text: 'some text' },
    { type: 'text', text: 'more text' }
  ])
  .build();              // Always call build() at the end

// Then use the built body in axios request
const response = await axios.post(
  'https://graph.facebook.com/v20.0/[PHONE_ID]/messages',
  whatsappBody,
  { headers: { /* ... */ } }
);
```

### Benefits

- Type safety for required parameters
- Consistent message structure
- Automatic text length limiting (60 chars)
- Builder pattern for cleaner, more readable code
- Easier maintenance when WhatsApp API changes

### Reference Implementation

See [send_payment_received_notification](mdc:src/utils/whatsapp/whatsapp.service.ts) for a complete example of how to use this class.

### ⚠️ Do Not Use Direct Object Literals

Avoid constructing message bodies directly as object literals. This leads to inconsistency and makes maintenance harder:

```typescript
// ❌ BAD - Don't do this
const body = {
  messaging_product: 'whatsapp',
  to: `91${phoneNumber}`,
  type: 'template',
  template: {
    name: templateName,
    language: { code: 'en' },
    components: [/* ... */]
  }
};
```

Always use the WhatsappBodyDto class for all WhatsApp message construction.

---
> Source: [WebNaresh/abhyasika_microservice_1](https://github.com/WebNaresh/abhyasika_microservice_1) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-29 -->
