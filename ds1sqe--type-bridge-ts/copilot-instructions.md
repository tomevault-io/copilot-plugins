## type-bridge-ts

> TypeScript ORM for TypeDB - Type-safe abstractions over TypeQL.

# type-bridge-ts

TypeScript ORM for TypeDB - Type-safe abstractions over TypeQL.

## Quick Start

```bash
# Install dependencies
npm install

# Run tests
npm test

# Build
npm run build

# Type check
npm run typecheck
```

## Project Structure

```
src/
├── index.ts              # Public API exports
├── database.ts           # Database connection
├── query.ts              # Query builder
├── validation.ts         # Reserved word validation
├── reserved-words.ts     # TypeQL reserved words
├── attribute/            # Attribute system
│   ├── base.ts           # Abstract Attribute class
│   ├── string.ts         # StringAttribute
│   ├── integer.ts        # IntegerAttribute
│   ├── double.ts         # DoubleAttribute
│   ├── boolean.ts        # BooleanAttribute
│   ├── datetime.ts       # DateTimeAttribute
│   ├── datetime-tz.ts    # DateTimeTZAttribute
│   ├── date.ts           # DateAttribute
│   ├── decimal.ts        # DecimalAttribute
│   ├── duration.ts       # DurationAttribute
│   └── flags.ts          # TypeFlags, AttributeFlags, Card
├── models/               # Entity and Relation classes
│   ├── base.ts           # TypeDBType base class
│   ├── entity.ts         # Entity class
│   ├── relation.ts       # Relation class
│   ├── role.ts           # Role definitions
│   └── utils.ts          # Model utilities
├── crud/                 # CRUD operations
│   ├── exceptions.ts     # CRUD exceptions
│   ├── utils.ts          # Shared utilities
│   ├── entity/           # Entity CRUD
│   │   ├── manager.ts    # EntityManager
│   │   └── query.ts      # EntityQuery
│   └── relation/         # Relation CRUD
│       ├── manager.ts    # RelationManager
│       └── query.ts      # RelationQuery
└── expressions/          # Expression system
    ├── base.ts           # Expression, BooleanExpr
    ├── comparison.ts     # ComparisonExpr, InExpr, RangeExpr
    ├── string.ts         # StringExpr, CaseInsensitiveExpr
    └── aggregate.ts      # AggregateExpr, GroupByExpr

tests/
├── attribute/            # Attribute type tests
├── crud/                 # CRUD operation tests
├── models/               # Entity/Relation model tests
└── expressions/          # Expression tests

docs/
├── attributes.md         # Attribute types and flags
├── models.md             # Entity and Relation definitions
├── crud.md               # CRUD operations
├── expressions.md        # Query expression system
├── database.md           # Connection management
└── query.md              # Low-level query construction
```

## Key Concepts

### Attribute Types

All TypeDB value types are supported:

| TypeScript Class | TypeDB Type |
|-----------------|-------------|
| `StringAttribute` | `string` |
| `IntegerAttribute` | `integer` |
| `DoubleAttribute` | `double` |
| `BooleanAttribute` | `boolean` |
| `DateTimeAttribute` | `datetime` |
| `DateTimeTZAttribute` | `datetime-tz` |
| `DateAttribute` | `date` |
| `DecimalAttribute` | `decimal` |
| `DurationAttribute` | `duration` |

### Basic Usage Pattern

```typescript
import {
  Database,
  Entity,
  StringAttribute,
  IntegerAttribute,
  TypeFlags,
  AttributeFlags,
} from '@type-bridge/type-bridge';

// 1. Define attribute types
class Name extends StringAttribute {
  static override flags = new AttributeFlags({ name: 'name' });
}

class Age extends IntegerAttribute {
  static override flags = new AttributeFlags({ name: 'age' });
}

// 2. Define entity
class Person extends Entity {
  static override flags = new TypeFlags({ name: 'person' });
  declare name: Name;
  declare age: Age;

  static {
    this.ownedAttributes = new Map([
      ['name', { typ: Name, flags: new AttributeFlags({ isKey: true }) }],
      ['age', { typ: Age, flags: new AttributeFlags() }],
    ]);
  }
}

// 3. Connect and use
const db = new Database({ address: 'localhost:1729', database: 'mydb' });
await db.connect();

const manager = Person.manager(db);
await manager.insert(new Person({ name: new Name('Alice'), age: new Age(30) }));
```

### Filter Operators

Django-style lookup operators:

| Operator | Description | Example |
|----------|-------------|---------|
| `exact` / `eq` | Exact match | `{ name: 'Alice' }` |
| `gt` | Greater than | `{ age__gt: 30 }` |
| `gte` | Greater than or equal | `{ age__gte: 18 }` |
| `lt` | Less than | `{ age__lt: 65 }` |
| `lte` | Less than or equal | `{ age__lte: 100 }` |
| `contains` | String contains | `{ name__contains: 'ali' }` |
| `in` | Value in list | `{ status__in: ['active', 'pending'] }` |

## Dependencies

- `typedb-driver-http`: TypeDB HTTP driver for Node.js
- TypeScript 5.0+
- Node.js 18+

## Differences from Python

The TypeScript package is a port of the Python ORM with TypeScript idioms:

1. **Static typing**: Uses TypeScript's type system instead of Python's runtime checks
2. **Async/await**: All database operations are async
3. **Class static blocks**: Uses `static {}` for attribute registration
4. **No schema management**: Schema/migration features not yet ported

## Getting Help

- Report issues at: https://github.com/ds1sqe/type-bridge-ts/issues
- For Python ORM, see: https://github.com/ds1sqe/type-bridge

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ds1sqe) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
