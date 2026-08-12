## calculations-rules

> Build a precious metals pricing calculator for a gold/silver buying business. The system takes spot prices as inputs and calculates buy/sell prices for bullion products and scrap gold jewelry.


# Precious Metals Pricing Calculator - Implementation Spec

## Overview

Build a precious metals pricing calculator for a gold/silver buying business. The system takes spot prices as inputs and calculates buy/sell prices for bullion products and scrap gold jewelry.

**Buy price** = what the business pays customers who are selling metal
**Sell price** = what the business charges customers who are buying metal

---

## Constants

```
GRAMS_PER_TROY_OZ = 31.12
SCRAP_GOLD_MARGIN = 0.73  // Business pays 73% of calculated scrap value
```

---

## Input Variables

The system requires two primary inputs that change daily:

```
goldSpotCAD   // Current gold price per troy ounce in CAD (e.g., 6667.00)
silverSpotCAD // Current silver price per troy ounce in CAD (e.g., 115.90)
```

Optional: If fetching USD prices, also need:
```
goldSpotUSD
silverSpotUSD
exchangeRate  // USD to CAD

// Then convert:
goldSpotCAD = goldSpotUSD * exchangeRate
silverSpotCAD = silverSpotUSD * exchangeRate
```

---

## Calculation Logic

### 1. Gold Bullion Products

Products priced as multipliers of `goldSpotCAD`:

| Product | Sell Price | Buy Price |
|---------|------------|-----------|
| Gold Maple Leaf 1oz | `goldSpotCAD * 1.0338` | `goldSpotCAD * 0.9814` |
| 1oz Gold Bar | `goldSpotCAD * 1.0278` | `goldSpotCAD * 0.9784` |
| 1/10 oz Canadian Gold Maple Leaf Coin | `(goldSpotCAD ÷ 10) × 1.1102` | `(goldSpotCAD ÷ 10) × 0.9821` |
| 5g Gold Bar | `goldSpotCAD * (5/31.12) * 1.045` | `goldSpotCAD * (5/31.12) * 0.975` |
| 10g Gold Bar | `goldSpotCAD * (10/31.12) * 1.0401` | `goldSpotCAD * (10/31.12) * 0.98` |

### 2. Silver Bullion Products

Products priced as multipliers of `silverSpotCAD`:

| Product | Sell Price | Buy Price |
|---------|------------|-----------|
| Silver Maple Leaf 1oz | `silverSpotCAD * 1.1121` | `silverSpotCAD * 0.9493` |
| American Eagle 1oz | `silverSpotCAD * 1.1087` | `silverSpotCAD * 0.9484` |
| Silver Bar 10oz | `silverSpotCAD * 1.0865 * 10` | `silverSpotCAD * 0.9345 * 10` |
| Silver Bar 100oz | `silverSpotCAD * 1.0785 * 100` | `silverSpotCAD * 0.9329 * 100` |

### 3. Scrap Gold by Karat (Buy Only)

When buying scrap jewelry, calculate the buy price per gram based on karat purity.

**Karat Purity Values:**
```
8k  = 0.333  (33.3% pure gold)
10k = 0.41   (41% pure gold)
12k = 0.50   (50% pure gold)
14k = 0.58   (58% pure gold)
18k = 0.75   (75% pure gold)
22k = 0.90   (90% pure gold)
24k = 0.99   (99% pure gold)
```

**Calculation (3 steps):**

```
function calculateScrapGoldBuyPrice(karat, goldSpotCAD):
    // Step 1: Get purity decimal for the karat
    purity = KARAT_PURITY[karat]

    // Step 2: Calculate value per gram at spot price
    // (purity / grams_per_oz) gives gold content per gram
    // multiply by spot to get dollar value per gram
    valuePerGram = (purity / 31.12) * goldSpotCAD

    // Step 3: Apply margin (business keeps 27%, pays 73%)
    buyPricePerGram = valuePerGram * 0.73

    return buyPricePerGram
```

**Example (14k gold at $6667 spot):**
```
purity = 0.58
valuePerGram = (0.58 / 31.12) * 6667 = $124.27
buyPricePerGram = $124.27 * 0.73 = $90.72
```

---

## Data Structure

```javascript
const pricingConfig = {
  constants: {
    GRAMS_PER_TROY_OZ: 31.12,
    SCRAP_GOLD_MARGIN: 0.73
  },

  karatPurity: {
    "8k": 0.333,
    "10k": 0.41,
    "12k": 0.50,
    "14k": 0.58,
    "18k": 0.75,
    "22k": 0.90,
    "24k": 0.99
  },

  goldBullion: [
    {
      name: "Gold Maple Leaf 1oz",
      sellMultiplier: 1.0338,
      buyMultiplier: 0.9814
    },
    {
      name: "1oz Gold Bar",
      sellMultiplier: 1.0278,
      buyMultiplier: 0.9784
    },
    {
      name: "1/10 oz Canadian Gold Maple Leaf Coin",
      sellMultiplier: (1/10) * 1.1102,
      buyMultiplier: (1/10) * 0.9821
    },
    {
      name: "5g Gold Bar",
      sellMultiplier: (5/31.12) * 1.045,
      buyMultiplier: (5/31.12) * 0.975
    },
    {
      name: "10g Gold Bar",
      sellMultiplier: (10/31.12) * 1.0401,
      buyMultiplier: (10/31.12) * 0.98
    }
  ],

  silverBullion: [
    {
      name: "Silver Maple Leaf 1oz",
      sellMultiplier: 1.1121,
      buyMultiplier: 0.9493,
      weightOz: 1
    },
    {
      name: "American Eagle 1oz",
      sellMultiplier: 1.1087,
      buyMultiplier: 0.9484,
      weightOz: 1
    },
    {
      name: "Silver Bar 10oz",
      sellMultiplier: 1.0865,
      buyMultiplier: 0.9345,
      weightOz: 10
    },
    {
      name: "Silver Bar 100oz",
      sellMultiplier: 1.0785,
      buyMultiplier: 0.9329,
      weightOz: 100
    }
  ]
};
```

---

## Implementation Functions

```javascript
function calculateGoldBullionPrices(goldSpotCAD, config) {
  return config.goldBullion.map(product => ({
    name: product.name,
    sell: goldSpotCAD * product.sellMultiplier,
    buy: goldSpotCAD * product.buyMultiplier
  }));
}

function calculateSilverBullionPrices(silverSpotCAD, config) {
  return config.silverBullion.map(product => ({
    name: product.name,
    sell: silverSpotCAD * product.sellMultiplier * product.weightOz,
    buy: silverSpotCAD * product.buyMultiplier * product.weightOz
  }));
}

function calculateScrapGoldPrices(goldSpotCAD, config) {
  const prices = {};

  for (const [karat, purity] of Object.entries(config.karatPurity)) {
    const valuePerGram = (purity / config.constants.GRAMS_PER_TROY_OZ) * goldSpotCAD;
    prices[karat] = valuePerGram * config.constants.SCRAP_GOLD_MARGIN;
  }

  return prices; // Returns buy price per gram for each karat
}
```

---

## UI Requirements

1. **Input fields** for gold spot price and silver spot price (CAD)
2. **Gold Bullion table** showing product name, sell price, buy price
3. **Silver Bullion table** showing product name, sell price, buy price
4. **Scrap Gold table** showing karat and buy price per gram
5. All prices should **update automatically** when spot prices change
6. Display today's date
7. Format all prices as currency (2 decimal places)

---
> Source: [lucasecerbegovic/sidney-gold-silver](https://github.com/lucasecerbegovic/sidney-gold-silver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
