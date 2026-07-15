## load-handilng

> Load management and demand card handling

### ✅ **Eurorails Load Management – Contextual Design Rules for Code Generation**

#### 🧠 **Core Concepts & Responsibilities**

- A **Train** object can **carry loads**, has a **capacity**, and can **move across the map**.
- A **LoadChip** is a physical token representing a **type of good**. It can exist:
  - On a train
  - In a city
  - In the tray (available pool)
- A **City** may offer **specific load types** for pickup.
- A **DemandCard** links a **destination city** to up to **3 specific load types** and a **payoff amount**.
- The **Bank** manages:
  - The money supply
  - The pool of available load chips
  - Demand card deck and drawing

---

### 🏗️ **Design Rule Categories**

---

#### 1. 🏙️ **Load Pickup Rules**

- ✅ A train can **pick up a load** only if:
  - It is **currently on** a city that **produces** that load.
  - The **load chip is available** in the tray (i.e., not already in use).
  - The train has **remaining capacity** (≤ 2 or 3 loads depending on train type).
- ✅ Players **do NOT need a Demand card** to pick up a load.
- 🔁 Picking up a load does **not consume movement**.
- 🧾 Load pickup should be recorded in the train’s `currentLoads[]` array and removed from the tray.

---

#### 2. 🚚 **Load Delivery Rules**

- ✅ A player may **deliver a load** if:
  - The train is **at a city** listed on a **Demand card**.
  - The load matches **one of the three load types** on that card.
- ✅ Delivery process:
  - Discard the Demand card.
  - Return the load chip to the tray.
  - Add the card’s payout to the player’s money.
  - Draw a new Demand card to maintain a hand of 3.
- ⛔ A single Demand card may only be used for **one delivery**, even if the train has multiple matching loads.

---

#### 3. 🛑 **Load Dropping Rules**

- ✅ A player may **drop a load** at **any city**, even if it’s not demanded.
  - If the city produces that load → the load goes back to the tray.
  - If it doesn’t → the load stays in that city.
  - If a load is already present in that city:
    - The **existing load is returned to the tray**
    - The **new load replaces it**

---

#### 4. 🚂 **Train Constraints**

- ✅ Train capacity:
  - Freight / Fast Freight → 2 loads
  - Heavy Freight / Superfreight → 3 loads
- ✅ Movement limits:
  - Freight / Heavy → 9 mileposts/turn
  - Fast / Super → 12 mileposts/turn
- ✅ Picking up, dropping, or delivering a load does **not reduce movement**.

---

#### 5. 🧠 **Demand Card Rules**

- ✅ Each player must **always hold exactly 3 Demand cards**.
- ✅ When delivering a load or losing a card (e.g., Event), the player must:
  - Immediately draw a replacement, **even outside their own turn**.
- ✅ Demand cards list:
  - One city
  - Three load types
  - A payoff amount for delivery of any one of the listed loads to that city

---

#### 6. 🚨 **Event Interactions**

- ❗ Event cards may cause load loss (e.g., **Derailment**):
  - If a train loses a load, the **player chooses which one**.
  - The lost load is returned to the tray.
- ❗ Delivery and pickup are blocked in certain events (e.g., **Strike!**).

---

#### 7. ♻️ **Load Chip Lifecycle**

```plaintext
Tray → City (setup/drop) → Train (pickup) → Tray (delivery/loss/drop)
```

- Loads should never be **duplicated**; the pool is fixed and finite.
- Dropping a load must respect **city slot constraints** (max 1 load per city).

---

#### 8. 🧱 **Code Style / Implementation Guidelines**

- Prefer **explicit state** tracking for:
  - LoadChip location (`'tray'`, `'train'`, `'city:<name>'`)
  - Train inventory (`Train.currentLoads[]`)
  - Player hand (`Player.demandCards[]`)
- Validate all actions with **guard clauses** or **`canX` methods**:
  - `canPickUp(load: LoadChip, city: City, train: Train): boolean`
  - `canDeliver(load: LoadChip, city: City, card: DemandCard): boolean`
- Use **pure functions** to enforce business rules where possible.
- Favor **event-sourced** logging if implementing audit/history (optional but useful for testing).

---

### ✅ Example Usage Patterns

```ts
if (canPickUp(load, currentCity, player.train)) {
  pickUpLoad(load, player.train, bank.tray)
}
```

```ts
if (canDeliver(load, currentCity, demandCard)) {
  deliverLoad(load, player, demandCard, bank)
}
```

---

---
> Source: [jeffgabriel/eurorails_ai](https://github.com/jeffgabriel/eurorails_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
