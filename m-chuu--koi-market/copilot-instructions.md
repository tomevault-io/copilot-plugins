## koi-market

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

React Koi Fish Store built with Vite. Displays a product grid with bilingual names (English + Chinese) and an interactive shopping cart sidebar.

## Commands

```bash
npm install          # Install dependencies
npm run dev          # Start dev server at http://localhost:5173
npm run build        # Build for production
npm run preview      # Preview production build
```

## Architecture

### State Management

Cart state lives in `MainContent.jsx` using `useState`:
- `cartItems`: Array of products in cart
- `addToCart(product)`: Appends product to cart
- `removeFromCart(productId)`: Removes first occurrence by ID
- `totalPrice`: Computed via `reduce()` on `price.min`

### Data Flow

```
products.js → MainContent → ProductCard (onAddToCart callback)
                         → ShoppingCart (cartItems + onRemoveFromCart)
```

### Product Data Structure

Products are defined in `src/data/products.js`:

```javascript
{
  id: number,
  nameEn: string,      // English/Japanese name
  nameCn: string,      // Chinese name
  sizeRange: string,   // e.g., "15-18cm"
  price: { min, max }, // RM currency; min === max for fixed price
  image: string,       // Vite import from assets/images/
  currency: "RM"
}
```

### Price Display

- Fixed price (min === max): `RM 380`
- Price range: `RM 380 - RM 480`

### Bilingual Display

All UI text shows both languages: `Kohaku 红白`

### Adding New Products

1. Add image to `src/assets/images/`
2. Import image in `src/data/products.js`
3. Add product object to the `products` array

---
> Source: [m-chuu/koi-market](https://github.com/m-chuu/koi-market) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
