## quake-wallet

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Quake Wallet** is a React Native financial instruments mobile application that allows users to view instruments, manage their investment portfolio, and send buy/sell orders. Built with Expo, TypeScript, Redux Toolkit, and React Navigation.

**Backend API**: `https://dummy-api-topaz.vercel.app`
**Currency**: All prices are in Argentine pesos (ARS)

## Development Commands

### Running the App
```bash
# Start development server
npm start
# or
npx expo start

# Run on specific platform
npm run android    # Android emulator
npm run ios        # iOS simulator
npm run web        # Web browser
```

### Testing
```bash
# Run all unit tests
npm test

# Run tests in watch mode
npm test -- --watch

# Run specific test file
npm test -- __tests__/unit/calculations.test.ts
```

### Code Quality
```bash
# Lint the codebase
npm run lint
```

## Architecture

### Layer-First Architecture

The codebase follows a **layer-first** architecture (not feature-first), organizing by technical responsibility:

```
src/
├── screens/         # Top-level screen components for navigation
├── components/      # Reusable UI components (organized by domain)
├── navigation/      # React Navigation configuration
├── store/           # Redux Toolkit store, slices, and hooks
├── services/        # API client and endpoint functions
├── schemas/         # Zod validation schemas
├── types/           # TypeScript type definitions
├── utils/           # Pure utility functions (calculations, formatters)
├── hooks/           # Custom React hooks
└── constants/       # App-wide constants
```

### State Management

- **Redux Toolkit** with async thunks for API calls
- Each slice follows the pattern:
  ```typescript
  interface SliceState<T> {
    data: T | null;
    loading: boolean;
    error: string | null;
  }
  ```
- Use typed hooks from `src/store/hooks.ts` (not direct Redux hooks)

### API Integration

- All API calls go through `src/services/api/client.ts` (axios instance)
- API functions are organized by resource: `instruments.api.ts`, `portfolio.api.ts`, `orders.api.ts`
- Error handling is centralized in the axios interceptor

### Navigation Structure

```
Bottom Tabs Navigator
├── Instruments Tab  → InstrumentsScreen
├── Portfolio Tab    → PortfolioScreen
└── Search Tab       → SearchScreen

Modal (overlay)
└── OrderModal (triggered from any screen)
```

## Key Business Logic

### Calculation Functions

All calculation logic is in `src/utils/calculations/`:

1. **Returns** (`returns.ts`):
   ```typescript
   calculateReturn(lastPrice, closePrice)
   // Returns: ((lastPrice - closePrice) / closePrice) * 100
   ```

2. **Portfolio** (`portfolio.ts`):
   ```typescript
   calculateMarketValue(quantity, lastPrice)
   // Returns: quantity × lastPrice

   calculateProfit(quantity, lastPrice, avgCostPrice)
   // Returns: (lastPrice - avgCostPrice) × quantity

   calculateProfitPercentage(lastPrice, avgCostPrice)
   // Returns: ((lastPrice - avgCostPrice) / avgCostPrice) × 100
   ```

3. **Orders** (`orders.ts`):
   ```typescript
   calculateQuantityFromAmount(amount, price)
   // Returns: Math.floor(amount / price)
   // Note: Fractional shares are NOT allowed
   ```

### Order Types and Statuses

**Order Types**:
- `MARKET`: Executes immediately at current market price
- `LIMIT`: Executes only when price reaches specified limit

**Order Statuses**:
- `FILLED`: Order executed (MARKET orders result in FILLED or REJECTED)
- `PENDING`: Order waiting for execution (LIMIT orders result in PENDING or REJECTED)
- `REJECTED`: Order rejected by market (e.g., insufficient funds)

### Form Validation

- Uses **React Hook Form** + **Zod** via `@hookform/resolvers`
- Order schema is in `src/schemas/order.schema.ts`
- Key validation: LIMIT orders require a price field, MARKET orders do not

## API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/instruments` | List all available instruments |
| GET | `/portfolio` | User's current positions |
| GET | `/search?query={ticker}` | Search instruments by ticker |
| POST | `/orders` | Create buy/sell order |

### Order Request Format

```typescript
// MARKET order
{
  "instrument_id": 1,
  "side": "BUY" | "SELL",
  "type": "MARKET",
  "quantity": 100
}

// LIMIT order
{
  "instrument_id": 1,
  "side": "BUY" | "SELL",
  "type": "LIMIT",
  "quantity": 100,
  "price": 45.50  // Required for LIMIT
}
```

## TypeScript Patterns

### Path Aliases

The project uses `@/*` for absolute imports:
```typescript
import { Instrument } from '@/src/types';
import { formatCurrency } from '@/src/utils';
```

### Strict Mode

TypeScript strict mode is enabled. All code must:
- Have explicit return types on exported functions
- Properly type all props and state
- Avoid `any` types

## Testing Strategy

### Unit Tests
- Location: `__tests__/unit/`
- Test pure functions: calculations, formatters, utilities
- Run with `npm test`

### Component Tests
- Location: `__tests__/components/`
- Use `@testing-library/react-native`
- Test user interactions and rendering

### E2E Tests
- Location: `e2e/`
- Test critical user flows (viewing instruments, creating orders)
- See `02_project_plan.md` section 11 for detailed test scenarios

## Important Implementation Notes

1. **No Fractional Shares**: Always use `Math.floor()` when calculating quantity from amount
2. **Currency Formatting**: All amounts should be formatted as ARS (Argentine pesos)
3. **Error Handling**: Display user-friendly messages, not raw API errors
4. **Loading States**: Every async operation must show loading feedback
5. **Debouncing**: Search input uses 300ms debounce (see `src/hooks/useDebounce.ts`)

## Common Patterns

### Adding a New Screen
1. Create screen in `src/screens/`
2. Add to navigator in `src/navigation/`
3. If data is needed, create/update Redux slice in `src/store/slices/`
4. Add API calls in `src/services/api/`

### Adding a Calculation Function
1. Add pure function to appropriate file in `src/utils/calculations/`
2. Export from `src/utils/calculations/index.ts`
3. Write unit test in `__tests__/unit/calculations.test.ts`

### Adding a New API Endpoint
1. Add function to relevant file in `src/services/api/`
2. Use the `apiClient` from `client.ts`
3. Add TypeScript types in `src/types/`
4. Integrate into Redux slice if state management is needed

## Project Documentation

For detailed technical specifications, architecture diagrams, and business requirements, see:
- `02_project_plan.md` - Complete technical documentation
- `01_requerimientos.md` - Initial requirements
- `README.md` - User-facing documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stormwings)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/stormwings)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
