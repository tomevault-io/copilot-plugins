## project-context

> Project definition — what the app does, for whom, and main constraints


# Project context — next-bouquet-ai

## Problem

Flower shops face two cases: (1) client knows exactly what they want → quote, make, deliver; (2) client says something vague (“a nice bouquet”) → hard to define, risk of a bouquet that doesn’t match expectations.

This app addresses (2): use **AI** to generate an **image** from a short description of what the client asked, **show it to the client**, and **iterate** until the bouquet is clear. Then the shop has a clear spec to produce.

## Users

- **Florist**: defines stock, restock cadence, and prices; **listens to the client** and **enters the prompt** in the app to generate/refine AI previews; obtains a clear, priced bouquet spec.
- **Client**: describes what they want to the florist; **sees** the AI image and gives feedback to iterate. The client does **not** use the prompt themselves.

**Out of scope for now:** Client-facing prompt (self-service). Only the florist triggers image generation (cost control). Client prompts may be considered in a later phase.

## Flower catalogue (florist admin)

The **florist (owner/admin)** has an area to **define flowers** with:

- **Name** (e.g. roses, margaritas)
- **Unit cost** (price per unit)
- **Stock** (current quantity available)

This is implemented as a **database table** with basic **CRUD** operations (create, read, update, delete). These values are used to **adjust bouquet cost** (e.g. when the client has a budget or when composing from available flowers).

## Voice input parser (prompt → structured data)

Before the prompt is sent to the AI for image generation, **voice or text input** must pass through a **parser** that extracts structured data (e.g. `price`, `flowers`, `lazo`, `esRegalo`). This allows:

- **Validation**: Check that important fields are present before generating the bouquet.
- **Re-asking**: If data is missing or unclear, the florist can confirm with the client instead of generating an incorrect bouquet.

Example: _"Quiero comprar un ramo de rosas, que cueste 30€, que tenga alguna margarita y un lazo"_ → `{ price: 30, flowers: ['rosas', 'margaritas'], lazo: true, esRegalo: undefined }`. The UI should surface this summary for confirmation before calling the image generation.

## Search past bouquets

Support **searching previously made bouquets** by **partial criteria** (e.g. by flowers, price, or other stored parameters). Example: _"Enseñame los ramos hechos con rosas"_ → list bouquets that include roses. This enables reusing or referencing past orders and showing the client matching examples.

## Constraints to respect

- **Stock**: Florist defines which flowers are in stock via the **flower catalogue** (unit cost + stock per flower). Bouquets are bought on a cadence (e.g. 2×/month) and stored in a fridge. The **proposed bouquet must respect current stock** when showing the preview.
- **Future delivery**: If the bouquet is for a **date after the next restock**, allow selecting bouquets with **any flowers** (not limited to current stock).
- **Pricing**: Florist sets **unit cost per flower** in the catalogue; the system uses these values when the client has a **budget** (e.g. “something around €40”) to calculate bouquet cost.
- **Unavailable flowers**: When the client asks for a flower that isn’t available, support creating a bouquet from **similar flowers** (AI suggesting alternatives).
- **Speed**: The AI model must be **fast**; prioritise **response time** so users can iterate quickly toward a realistic result.

When suggesting features, flows, or data models, keep these constraints in mind.

**Cost:** Prefer services with a free tier to keep costs low. Image generation (Replicate) has no unlimited free tier and is the main cost; the rest of the stack (Supabase, Vercel, etc.) can use free tiers where possible.

---
> Source: [antonioballesteros/next-bouquet-ai](https://github.com/antonioballesteros/next-bouquet-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
