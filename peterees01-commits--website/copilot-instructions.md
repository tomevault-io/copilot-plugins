## website

> New products are published through `data/products.json` + the shared

# Product page conventions

New products are published through `data/products.json` + the shared
`products/product.html` / `assets/js/product.js` template — never a bespoke
page per product. Follow the format and section order already established
by DPL (Denver iD Pole) and DSX (D-Series) exactly. If a new product's data
doesn't cleanly fit that template (e.g. a detail DPL/DSX don't have, or a
field neither of them needed), ask before improvising a variation.

- **Multiple sizes/variants of one range** are one product entry (one
  `code`, one page), with each size as a row in `dimensions.variants` —
  not separate products/pages per size. See DSX0/DSX1/DSX2 under the
  `"DSX"` product for the pattern.
- **Hero/thumbnail image** (`images[0]`, used as both the product card
  thumbnail and the default gallery image) should be a clean side-profile
  shot for pole-mounted streetlight/area-lighting products — camera
  perpendicular to the pole, arm and head shown in silhouette, no
  three-quarter perspective. This is what DPL and DSX both use and it's
  the preferred look.
- **When a gallery mixes photos of different sizes** (as DSX does), tag
  each image entry with a `"size"` field (e.g. `"DSX0"`). `product.js`
  renders that as a small badge on the gallery thumbnail and on the main
  image when selected, so it's never ambiguous which size a photo shows.
  Leave `"size"` off images that aren't size-specific (dimension diagrams,
  etc).
- **Unconfirmed spec values** (manufacturer data not supplied, e.g.
  warranty period, IK rating): add the row with an empty `"value"` rather
  than guessing or fabricating a number. Don't invent figures for a public
  spec page.

---
> Source: [peterees01-commits/website](https://github.com/peterees01-commits/website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-02 -->
