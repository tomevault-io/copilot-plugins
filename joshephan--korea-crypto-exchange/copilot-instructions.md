## korea-crypto-exchange

> This project exposes a **single public module** named:

This project exposes a **single public module** named:

It provides a unified interface for accessing Korean cryptocurrency exchanges through a **Facade + Strategy** design pattern architecture.

---

## 🎯 Purpose

To abstract away the differences between **Upbit** and **Bithumb** exchanges while allowing developers to switch or extend trading strategies seamlessly.

---

## 🧱 Architectural Patterns

### ✅ Facade Pattern

- `ExchangeFacade` acts as the main interface.
- It delegates tasks to internal strategies without exposing implementation details.

### ✅ Strategy Pattern

Each exchange (Upbit, Bithumb) defines:
- `quotation.ts` for price data
- `websocket.ts` for real-time stream handling
- `exchange.ts` for placing orders

All strategies conform to shared TypeScript interfaces defined in `core/types.ts`.

---

## 📁 Directory Structure

/src
│
├── core/                    # Common types and util functions
│   ├── types.ts
│   └── logger.ts
│
├── strategies/             # Strategy Objects
│   ├── upbit/
│   │   ├── index.ts
│   │   └── quotation.ts
│   │   └── websocket.ts
│   │   └── exchange.ts
│   │   └── types.ts
│   └── bithumb/
│       ├── index.ts
│       └── quotation.ts
│       └── websocket.ts
│       └── exchange.ts
│       └── types.ts
│
├── facade/                 # Facade: Core module
│   ├── ExchangeFacade.ts
│   └── ExchangeSelector.ts
│
├── index.ts                # Entry
└── config.ts

## 🔌 Public Module: KoreaCryptoExchange

```ts
// src/index.ts
import { ExchangeFacade } from './facade/ExchangeFacade';
import { ExchangeSelector } from './facade/ExchangeSelector';

const KoreaCryptoExchange = new ExchangeFacade(new ExchangeSelector().select());

export default KoreaCryptoExchange;

```

💡 How to Extend
To add a new exchange (e.g. Coinone):

Create /strategies/coinone/

Add: quotation.ts, websocket.ts, exchange.ts

Add: index.ts to export all three

Update ExchangeSelector.ts to support the new provider

All modules must implement the shared interface contracts.

✅ Interface Contract (core/types.ts)

```ts
export interface IQuotation {
  getPrice(symbol: string): Promise<number>;
}

export interface IWebSocket {
  connect(): void;
  disconnect(): void;
}

export interface IExchangeStrategy {
  placeOrder(symbol: string, amount: number): Promise<string>;
}
```

🚀 Design Goals
Fully typed using TypeScript

Modular and extensible exchange strategy architecture

Exports one public object: KoreaCryptoExchange

Easily adaptable to additional providers or new trading logic

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joshephan) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
