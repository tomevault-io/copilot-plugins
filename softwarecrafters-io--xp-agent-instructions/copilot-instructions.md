## xp-agent-instructions

> Coding standards for names, functions and classes


# Coding Standards

To maintain code quality, I follow strict standards for names, functions, and classes.

---

## Names

- Pronounceable in English, without technical abbreviations (abbreviations only allowed in lambda expressions with reduced scope)
- Avoid redundant prefixes/suffixes (`I`, `Impl`, `Abstract`)
- Concrete; avoid catch-all words (helper, util, manager) unless the domain requires it
- No type information; the IDE already shows it
- No aliases or synonyms for the same concept: one concept, one name
- Combine well with language grammar: `isPaidInvoice`, `sumOfNumbersIn(expression)`
- Prefer using "not" in the name over using the negation operator
- Distinguish nouns (classes, modules) from verbs (methods, functions), adjectives only for interfaces
- Allowed generic suffixes: DTO, Repository, Factory, Mapper, UseCase, Service
- Avoid comments if you can use a self-explanatory name. Comments only in extreme cases that code cannot explain
- Constants use camelCase, not SCREAMING_SNAKE_CASE
- Never use underscore prefix for private members
- Avoid magic strings, better use enums or literal types to represent fixed sets of values


```typescript
// ⚠️ WORSE
const d = 42; // days
const usrCtrl = new UsrCtrl();
const IUserRepository = ...;
const fetchUserInfo = ...;

// ✅ BETTER
const daysUntilExpiration = 42;
const userAuthenticator = new UserAuthenticator();
const UserRepository = ...;
const findUser = ...;
```

---

## Functions

1. **Contained size and single responsibility (SRP)**. Each function must "do exactly what its name indicates". Functions of 10-15 lines are a metric to detect if they have more than one responsibility; if they only have one responsibility, size doesn't matter

2. **Impeccable naming**. Function names must be **verbs** that precisely describe the action

3. **Minimum necessary signature**. Reduce **arity** (0–3 parameters is ideal). If exceeded, group them in an object with proper name and typing

4. **Avoid configuration parameters**. Don't use `boolean` (nor flags) that change behaviors. Prefer **specific functions** (`show()/hide()`, `switchToReadMode()/switchToWriteMode()`)

5. **Optional parameters in moderation**. Maximum **one**; if you can avoid them, better. Remember combinatorial explosion and its impact on tests

6. **Simple control flow**. Use **guard clauses** for edge cases and **exit early**; reduce indentation and cyclomatic complexity

7. **Readable conditions**:
   - Abstract **combined boolean expressions** into **explanatory variables** or **functions** (`const isOnSale = ...`)
   - **Prioritize affirmative conditions**; if negation improves clarity, encapsulate it in the name (`doesNotExist()`)
   - Avoid using else

8. **Separate control flow and business logic**. Extract iterations and branches; delegate calculations/mutations to dedicated functions or objects. Avoid using for

9. **Prefer declarative style when it improves readability**. Use `map/filter/reduce/forEach` with judgment; it's not dogma

10. **Encourage pure functions and referential transparency**. Avoid side effects when possible

11. **Apply CQS (Command–Query Separation)**:
    - **Commands**: mutate state and **don't return** value
    - **Queries**: **return** value and **don't mutate** state
    - Know **justified exceptions** (e.g., create and return `Id` when inserting)

12. **Balance performance–readability**. Optimize only when necessary. Prioritize readability

13. **Don't add comments to functions**

14. **Don't mutate collections**. Avoid push, pop, shift, unshift, splice. Prefer immutable operations that return new arrays

15. **Constants close to usage**. Define constants inside the function that uses them

```typescript
// Constants:
// ⚠️ WORSE
const maximumRetries = 3;
const timeoutMs = 5000;

function fetchWithRetry(url: string) {
  // ... uses maximumRetries far from definition
}

// ✅ BETTER
function fetchWithRetry(url: string) {
  const maximumRetries = 3;
  const timeoutMs = 5000;
  // ... uses constants right here
}

// Boolean parameters:
// ⚠️ WORSE
function render(showDetails: boolean) {
  if (showDetails) { /* ... */ }
  else { /* ... */ }
}

// ✅ BETTER
function renderWithDetails() { /* ... */ }
function renderSummary() { /* ... */ }

// CQS (Command-Query Separation):
// ⚠️ WORSE - Query that mutates state
function totalWithDiscount(percentage: number): number {
  this.appliedDiscount = percentage;
  return this.total * (1 - percentage / 100);
}

// ✅ BETTER - Command and Query separated
function applyDiscount(percentage: number): void { 
  this.appliedDiscount = percentage; 
}
function calculateTotal(): number { 
  return this.baseTotal * (1 - this.appliedDiscount / 100); 
}
```

---

## Classes and Modules

1. **Minimum scope for maximum cohesion**. If a method or function only uses a constant and nobody else uses it, put it inside the function

2. **Simple constructors**. If they have validation logic at creation, move it to a factory method and make the constructor private. If there's no complex validation, don't create unnecessary factory method

3. **Class organization**: Public constructor(s) first (if they exist), followed by private constructor, then public API, finally private methods. In TypeScript, properties can be auto-injected in the constructor

4. **Encapsulation by default**. Use `private` for methods and don't export functions unless necessary. Limit accessibility

5. **Apply Law of Demeter and Tell, Don't Ask**

6. **Avoid anemic models**. Classes should encapsulate behavior, except at application boundaries where you can use DTOs

7. **Complete objects at construction**. Avoid setters and limit getters. Objects should be fully initialized when created

8. **Never use singletons**. If an instance has global state, it should be managed by the application's factory module

9. **Composition over inheritance**. Avoid inheritance, prioritize composition

10. **Domain-specific types**. Build classes with behavior, especially in the application domain

```typescript
// Tell, Don't Ask and Law of Demeter:
// ⚠️ WORSE - Ask (we ask and decide outside)
if (order.customer().address().city() === "Madrid") {
  order.applyDiscount(10);
}

// ✅ BETTER - Tell (we tell it what to do)
order.applyDiscountForCity("Madrid", 10);

// ⚠️ WORSE - Law of Demeter violation (long chain)
const city = user.account().settings().location().city();

// ✅ BETTER - Law of Demeter (single dot)
const city = user.city();

// Anemic Model vs Rich Model:
// ⚠️ WORSE - Anemic model
class Order {
  items: Item[];
  total: number;
}
// Logic outside
const total = order.items.reduce((sum, item) => sum + item.price, 0);

// ✅ BETTER - Rich model
class Order {
  private items: Item[];
  
  calculateTotal(): number {
    return this.items.reduce((sum, item) => sum + item.price, 0);
  }
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/softwarecrafters-io) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
