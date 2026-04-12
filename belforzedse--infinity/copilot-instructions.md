## infinity

> Backend audit logging lifecycles and gateway audit entries for orders/contracts


# Backend Audit Logging

This project implements lifecycle-based audit logging for core entities. For each entity, a dedicated log content type records Create, Update, and Delete events. Updates capture field-level diffs.

## Conventions

- Logs are written by entity lifecycles (afterCreate, beforeUpdate/afterUpdate, beforeDelete/afterDelete).
- Diff policy excludes meta fields: `updatedAt`, `createdAt`, `id`, `documentId`.
- Controller-level logs are used to capture payment gateway events.

## Entities and their Logs

### Order

- Log model: [order-log schema](mdc:src/api/order-log/content-types/order-log/schema.json)
- Lifecycle: [order lifecycles](mdc:src/api/order/content-types/order/lifecycles.ts)
- Tracked fields: `Status`, `Date`, `Type`, `ShippingCost`, `Description`, `Note`
- Populated for context: `user`, `contract`, `shipping`
- Gateway audit trail (controller-level):
  - Request initiated (stores `requestId`, `refId`, `redirectUrl`)
  - Verification success/failure (Mellat: `ResCode`, `SaleOrderId`, `SaleReferenceId`, `RefId`)
  - Settlement success/failure (error/message as applicable)
  - Callback failures/cancellations
  - See: [Cart finalizeToOrder](mdc:src/api/cart/controllers/cart.ts) and [Order verifyPayment](mdc:src/api/order/controllers/order.ts)
- Fetch logs via CRUD: `GET /order-logs?filters[order][id][$eq]=<orderId>&sort[0]=createdAt:asc` ([router](mdc:src/api/order-log/routes/order-log.ts))

### Contract

- Log model: [contract-log schema](mdc:src/api/contract-log/content-types/contract-log/schema.json)
- Lifecycle: [contract lifecycles](mdc:src/api/contract/content-types/contract/lifecycles.ts)
- Tracked fields: `Type`, `Status`, `Amount`, `TaxPercent`, `Date`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/belforzedse) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
