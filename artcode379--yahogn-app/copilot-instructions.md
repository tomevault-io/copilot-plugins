## yahogn-app

> - **DO NOT explore the project.** The full structure map is below.

# CLAUDE.md — Project Rules

## Efficiency Rules (MANDATORY)
- **DO NOT explore the project.** The full structure map is below.
- **Batch writes:** Write the ENTIRE file in ONE Write command.
- **No re-reading:** After writing, do NOT read back to verify.
- **Silent execution:** Do not explain steps. Just execute.
- **Parallel downloads:** Batch all image downloads into one shell script.
- **Write, don't edit:** Use Write (full file replacement), not Edit.

## Content Rules
- ALL text in English
- No Lorem ipsum, no placeholders — real meaningful content only
- Stock images from Unsplash/Pexels only

## Image Download Rules (MANDATORY)
- After downloading ALL images, validate:
  ```bash
  for f in app/src/main/res/drawable/*.{jpg,png,webp}; do
    [ -f "$f" ] || continue
    mime=$(file --mime-type -b "$f" 2>/dev/null)
    size=$(stat -c%s "$f" 2>/dev/null || echo 0)
    if [[ "$mime" != image/* ]] || [ "$size" -lt 5000 ]; then
      echo "BROKEN: $f" && rm -f "$f"
    fi
  done
  ```

## Interactivity Rules (MANDATORY)
- EVERY onClick must contain real code
- ZERO empty lambdas: `onClick = { }` is FORBIDDEN

## Project Structure (DO NOT explore — use this map)

Package path: yahognapp/com/store

### Files to MODIFY (⚡):
- `data/repository/ProductRepository.kt` — fill with real Product objects
- `data/model/ProductCategory.kt` — enum of product categories
- `ui/composable/screen/home/HomeScreen.kt` — hero carousel, category grid, product cards
- `ui/composable/screen/onboarding/OnboardingScreen.kt` — 3 slides
- `ui/composable/screen/splash/SplashScreen.kt` — gradient + logo animation
- `ui/composable/screen/productdetails/ProductDetailsScreen.kt`
- `ui/composable/screen/cart/CartScreen.kt`
- `ui/composable/screen/checkout/CheckoutScreen.kt`
- `ui/composable/screen/order/OrderScreen.kt`
- `ui/composable/screen/settings/SettingsScreen.kt`
- `ui/theme/Color.kt`, `ui/theme/Theme.kt`, `ui/theme/Type.kt`
- `AOGYHApp.kt`
- `res/values/strings.xml`, `res/drawable/`

### Files to NOT modify:
- `MainActivity.kt`, `data/dao/*`, `data/database/*`, `data/datastore/*`, `data/entity/*`
- `data/model/Product.kt`, `data/repository/CartRepository.kt`, `data/repository/AOGYHOnboardingRepo.kt`
- `data/repository/OrderRepository.kt`, `di/*`, `ui/composable/approot/*`, `ui/composable/navigation/*`
- `ui/composable/screen/checkout/CheckoutDialog.kt`, `ui/composable/shared/*`, `ui/state/*`, `ui/viewmodel/*`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArtCode379) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
