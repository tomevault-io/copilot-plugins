## touchmemoriestest

> "name": "string (UA)",

# Project Constitution: TouchMemories

## Data Schemas

### Categories
```json
{
  "id": "uuid",
  "name": "string (UA)",
  "slug": "string (unique)",
  "description": "text",
  "cover_image": "text (url)",
  "sort_order": "int",
  "is_active": "boolean"
}
```

### Products
```json
{
  "id": "uuid",
  "category_id": "uuid (FK)",
  "name": "string",
  "slug": "string (unique)",
  "description": "text",
  "short_description": "text",
  "price": "numeric (UAH)",
  "price_per_page": "numeric",
  "min_pages": "int",
  "max_pages": "int",
  "cover_options": "jsonb",
  "format_options": "jsonb",
  "images": "string[]",
  "stock": "int",
  "is_active": "boolean",
  "meta_title": "string",
  "meta_description": "string"
}
```

### Customers
```json
{
  "id": "uuid",
  "email": "string",
  "name": "string",
  "phone": "string",
  "auth_user_id": "uuid",
  "total_orders": "int",
  "total_spent": "numeric"
}
```

### Orders
```json
{
  "id": "uuid",
  "order_number": "string",
  "customer_id": "uuid (FK)",
  "customer_name": "string",
  "customer_phone": "string",
  "customer_email": "string",
  "items": "jsonb",
  "subtotal": "numeric",
  "delivery_cost": "numeric",
  "total": "numeric",
  "delivery_method": "string",
  "delivery_address": "jsonb",
  "ttn": "string",
  "order_status": "string",
  "payment_status": "string",
  "mono_invoice_id": "string",
  "mono_payment_id": "string",
  "paid_at": "timestamp",
  "fiscal_status": "string",
  "fiscal_id": "string",
  "fiscal_url": "string",
  "constructor_project_id": "uuid (FK)",
  "notes": "text"
}
```

### Photo Book Projects
```json
{
  "id": "uuid",
  "customer_id": "uuid (FK)",
  "product_id": "uuid (FK)",
  "title": "string",
  "format": "string",
  "cover_type": "string",
  "page_count": "int",
  "canvas_data": "jsonb",
  "thumbnail_url": "string",
  "status": "string"
}
```

## Behavioral Rules
- Build deterministic, self-healing automation.
- Prioritize reliability over speed.
- Never guess at business logic.
- Follow the A.N.T. 3-layer architecture.
- All scraped data, logs, and temporary files live in `.tmp/`.

## Architectural Invariants
1. Logic must be deterministic.
2. Architecture (`architecture/`) must be updated before code (`tools/`).
3. External service keys must be verified in the Link phase.

## Maintenance Log
- **2026-03-01**: Comprehensive database schema implemented with categories, products, customers, orders, projects, and admin users. RLS policies established.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dianahohol97-max)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dianahohol97-max)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
