## roast-dashboard

> This project interacts with **Cropster** (c-sar.cropster.com), a coffee roastery management platform used by **Good Folks Coffee** (account: zach@goodfolkscoffee.com). Version 26.16.3.

# CLAUDE.md

## Project Context: Cropster — Good Folks Coffee

This project interacts with **Cropster** (c-sar.cropster.com), a coffee roastery management platform used by **Good Folks Coffee** (account: zach@goodfolkscoffee.com). Version 26.16.3.

---

## App Structure & Modules

### Samples
Manage incoming green coffee samples. Users can accept or reject samples individually or in bulk.

### Green Inventory
Tracks raw unroasted green coffee lots.
- ID tags: `PG-####`
- Key fields: weight, run-out estimation, price/lb, location, which roast profiles consume each lot
- Actions: Add lot, Export, Go to Inventory Report

### Roasts
Full roasting operations hub.
- ID tags: `PR-####`
- Sub-sections: Roasts, Profiles, Profile Groups, Post-roast Blend Profiles, Goals, Schedule
- Roast records include: date/time, profile, profile group, machine, location, technician, start/end weight, weight loss %, roasting curves (bean temp, exhaust temp, rate-of-rise), modulation chart, Between Batch Protocol
- Profiles define: batch size, machine, green lot linkage, expected weight loss %

### Roasted Inventory
Tracks post-roast inventory across three views:
- **Packaged Inventory** — on-hand packaged stock per product/variant; tracks scheduled packaging, orders, min stock level, net requirement
- **Loose Inventory** — unpackaged roasted coffee
- **Post-roast Blend Inventory** — blended roasted coffee awaiting packaging

Sub-sections:
- **Packaging Plans** (`PPL-####`) — scheduled packaging jobs with status, product, variant, date, on-hand roasted weight
- **Packaging Execution** — recording completed packaging runs

### Commerce
Sales and order management.
- **Orders** — Customer orders synced from external shops; filterable by fulfillment status, shop, etc.
- **Order Fulfillment** (`OF-####`) — Batch fulfillment groupings. Each has four tabs:
  - *Pick List* — individual orders with product, qty, customer, shop, external order status
  - *Packaging* — packaging demand vs. on-hand stock; create/execute packaging plans
  - *Roasting* — roasting demand vs. on-hand loose inventory; schedule roasts
  - *Blending* — post-roast blending requirements
- **Products** — product catalog with profile group linkage, variants, connected shops. Also has Product Bundles.

### Quality
Cupping and sensorial analysis.
- ID tags: `QC-####`
- Linked to roast lots (`PR-####`)
- Uses SCAA 2015 (Cup Ready) scoring sheets
- Fields: evaluators, final score, rating

### Reports
Pre-built analytical reports:
- **Roast Compare** — overlay 2+ roast curves with quality correlation
- **Production Report** — roasted output by time period, profile, machine
- **Inventory Report** — green inventory across locations; actual vs. initial weights, values
- **Quality Compare** — compare cupping sessions, evaluators, descriptors
- **Roast Goal Report** — track goal achievement (dev time ratio, weight loss)
- **Task Report** — task completion rates across locations and machinery

### Tasks
- **Task Board** — daily task view (filterable: Last 7 days / Today / Next 7 days / Next 30 days)
- **Task Management** — broader task admin and scheduling
- **Task Report** — task completion analytics

### More → Supply Network
Supply chain contact directory (farmers, cooperatives, importers).
- Fields: Member name, Contact person, Email, Member type, Country

---

## Connected Shops
- **Argo Sons Coffee-Backend**
- **Wholesale WooCommerce**

Orders sync from these external platforms into Cropster.

---

## Core Workflow
```
Green Coffee Sourcing
  → Green Inventory (PG-####)
  → Samples (accept/reject)
  → Quality / Cupping (QC-####)
  → Roast Profiles
  → Roast Schedule
  → Roasts (PR-####)
  → Loose Roasted Inventory
  → Packaging Plans (PPL-####)
  → Packaged Inventory
  → Orders (synced from shops)
  → Order Fulfillment (OF-####)
      → Pick → Package → Ship
```

Every step is traceable back to the original green lot via **Traceability Reports**.

---

## Key URL Patterns
- Green Inventory: `/app/inventory/overview`
- Roasts: `/app/roast/overview`
- Roast Profiles: `/app/roast/profiles`
- Roasted Inventory: `/app/roasted-inventory/inventory`
- Packaging Plans: `/app/roasted-inventory/packaging-plans`
- Orders: `/app/orders/overview`
- Order Fulfillment: `/app/orders/order-fulfillment/overview`
- Products: `/app/orders/products-overview`
- Quality: `/apps/quality/sensorial-qcs`
- Tasks: `/app/tasks/task-board`
- Supply Network: `/apps/contacts`

---
> Source: [zhens85/roast-dashboard](https://github.com/zhens85/roast-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-16 -->
