## data-viz

> You are helping with the **GW Data Visualization Competition 2026**. The dataset is from **The Loop** - GW's sustainable clothing exchange program on Mount Vernon Campus.

# GW Data Viz Challenge - AI Assistant Instructions

## Context
You are helping with the **GW Data Visualization Competition 2026**. The dataset is from **The Loop** - GW's sustainable clothing exchange program on Mount Vernon Campus.

## Dataset Location
```
/Users/nithinsarva/Downloads/data-viz/GW Data Visualization Competition 2026-selected/loop-data-ldw-2026.xlsx
```

## Dataset Overview
- **877 visits** from January - October 2025
- **15 columns**: Timestamp, Visit Date, Demographics, 10 clothing categories, Experience Rating, Donation status

### Key Variables
| Column | Type | Description |
|--------|------|-------------|
| Timestamp | datetime | When form was submitted |
| Visit Date | date | Date of visit |
| Demographics | categorical | Undergraduate, Graduate, Staff, Faculty, Community |
| Tops, Pants, Sweaters, etc. | count | Number of each item taken |
| Experience Rating | 1-5 | Customer satisfaction |
| Brought Donations | categorical | No, Yes, Donated at blue bin |

## Key Insights Already Discovered
1. **Friday phenomenon** - 83% of visits happen on Fridays
2. **September surge** - 205 visits (back to school) vs 19 in June (summer break)
3. **Graduate givers** - 60% of grads donate vs 31% of undergrads
4. **Tops dominate** - 909 tops taken, most popular item
5. **Near-perfect ratings** - 94% give 5 stars

## Existing Code
The visualization script is at:
```
/Users/nithinsarva/Downloads/data-viz/visualization.py
```

Output images are at:
```
/Users/nithinsarva/Downloads/data-viz/output/
```

## Preferred Stack
- Python 3 with pandas, matplotlib, seaborn
- High-quality PNG exports (300 DPI)
- Clean, publication-ready style

## Competition Rules
- Can use any AI tools
- Starting from blank file during competition
- Goal: Create compelling data visualizations

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sarvanithin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
