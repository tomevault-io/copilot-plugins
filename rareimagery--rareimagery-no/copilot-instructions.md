## rareimagery-no

> Multi-creator merchandise platform where X (Twitter) profiles become branded

# RareImagery.net - X Creator Merch Platform

## Project Overview
Multi-creator merchandise platform where X (Twitter) profiles become branded
storefronts selling physical (POD + custom) and digital products.
Built on Drupal 11 + Commerce 3.x.

## Architecture

### Stack
- PHP 8.3 (FPM), Drupal 11, Commerce 3.3.3, PostgreSQL 16, Nginx, Docker Compose

### Custom Module
`web/modules/custom/rareimagery_xstore/`

### Custom Theme
`web/themes/custom/rareimagery/` (extends Olivero)

### Entity Model
- `x_creator_store` (node type) -- Public storefront display, X profile data
- `commerce_store` type `creator` -- Commerce transactional store, linked 1:1 to node
- 3 product types: `physical_pod`, `physical_custom`, `digital_download`
- 3 variation types: `pod_variation`, `custom_variation`, `digital_variation`
- 3 order types: `pod_order`, `custom_order`, `digital_order`
- 5 taxonomy vocabularies: product_category, design_style, audience, animal_type, breed

### Key Services
- `rareimagery_xstore.store_manager` -- Store CRUD, handle lookup, commerce store bridge
- `rareimagery_xstore.printful_sync` -- Printful API product sync and order fulfillment
- `rareimagery_xstore.order_type_resolver` -- Routes variations to correct order types
- `rareimagery_xstore.creator_store_resolver` -- Resolves commerce_store from URL context
- `rareimagery_xstore.platform_fee_subscriber` -- Adds per-order platform fees
- `rareimagery_xstore.stripe_connect_subscriber` -- Adds Stripe Connect transfer_data

### SKU Convention
`[STORE]-[TYPE]-[PRODUCT]-[VARIANT]` e.g. `RAREIMAGERY-POD-001-SM-BLK`

## Key Commands

```bash
make up              # Start Docker containers
make down            # Stop containers
make build           # Rebuild containers
make install         # Fresh Drupal install + enable module
make cr              # Clear Drupal cache
make drush CMD="..." # Run any drush command
make export          # Export config to config/sync
make import          # Import config from config/sync
make reindex         # Clear and rebuild search index
make test            # Run PHPUnit tests
make lint            # Run PHP CodeSniffer
```

## Config Management
All module config lives in `web/modules/custom/rareimagery_xstore/config/install/`.
Site config sync directory is `config/sync/`.

## Payment Flow
Stripe Connect (application fees model):
1. Customer pays full amount to platform's Stripe account
2. `application_fee_amount` retains platform fee ($1.00 physical / $0.05 digital)
3. `transfer_data.destination` routes remainder to creator's Connected Account

## Mixed Cart Splitting
`StoreOrderTypeResolver` maps variation types to order types:
- `pod_variation` -> `pod_order`
- `custom_variation` -> `custom_order`
- `digital_variation` -> `digital_order`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rareimagery) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
