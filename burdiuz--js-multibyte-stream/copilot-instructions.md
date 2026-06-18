## js-multibyte-stream

> Bit-level binary serialization for JavaScript. Pack values into the minimum number of bits, then read them back. Useful for compact URL payloads, binary protocols, and data formats where byte-alignment wastes space.

# @actualwave/multibyte-stream

Bit-level binary serialization for JavaScript. Pack values into the minimum number of bits, then read them back. Useful for compact URL payloads, binary protocols, and data formats where byte-alignment wastes space.

## Installation

```bash
npm install @actualwave/multibyte-stream
```

## Core concepts

- **BitWriter / BitReader** — low-level bit I/O on any TypedArray
- **BitStream** — combined reader + writer on a shared buffer
- **Type classes** — encode/decode typed values (bool, int, float, string, array, object, enum, bigint)
- **Schema** — wraps an ObjectType; converts objects ↔ binary or base64

## Quick start

### 1. Raw bit I/O with BitStream

```typescript
import { BitStream } from '@actualwave/multibyte-stream';

const stream = new BitStream();

stream.write(15, 4);    // write value 15 using 4 bits
stream.write(100, 8);   // write value 100 using 8 bits

stream.setPosition(0);

stream.read(4);  // → 15
stream.read(8);  // → 100
```

### 2. Schema — auto-inferred from a sample object

```typescript
import { readSchemaFrom } from '@actualwave/multibyte-stream';

const sample = { active: false, score: 0, label: '' };
const schema = readSchemaFrom(sample);

const binary = schema.saveBase64From({ active: true, score: 42, label: 'hi' });
// → compact base64 string

schema.loadBase64To(binary);
// → { active: true, score: 42, label: 'hi' }
```

> The schema must be identical on both read and write sides — values are stored positionally with no field names.

### 3. Schema — explicit type definitions

Explicit schemas give full control over bit widths, signed/unsigned, float precision, and enumerated sets.

```typescript
import {
  Schema, ObjectType, ArrayType, IntType, BoolType,
  EnumType, StringType, SimpleFloatType,
} from '@actualwave/multibyte-stream';

const STATUS = ['pending', 'active', 'closed'];

const schema = new Schema(
  ObjectType.getInstance(undefined, {
    id:       new IntType(false, 16),          // unsigned 16-bit
    active:   new BoolType(),                  // 1 bit
    score:    new SimpleFloatType(true, 2),    // signed float, 2 decimal places
    status:   new EnumType(STATUS),            // index into STATUS array
    tags:     new ArrayType(new IntType(false, 8)), // array of unsigned bytes
    label:    new StringType(),
  })
);

const encoded = schema.saveBase64From({
  id: 1001, active: true, score: 98.76,
  status: 'active', tags: [1, 2, 3], label: 'hello',
});

schema.loadBase64To(encoded);
// → { id: 1001, active: true, score: 98.76, status: 'active', tags: [1, 2, 3], label: 'hello' }
```

### 4. Persist and restore a schema

Schemas can be serialised to a plain JSON-safe object and restored later.

```typescript
const schemaJson = JSON.stringify(schema.toObject());

// later…
const restored = Schema.fromObject(JSON.parse(schemaJson));
restored.loadBase64To(encoded);
```

## Type reference

| Class | Bits used | Notes |
|---|---|---|
| `BoolType` | 1 | `true` / `false` |
| `IntType(signed, size)` | fixed or variable | `size=0` → variable-length (3-bit length prefix + nibble chunks) |
| `ByteType` | 8 | Signed 8-bit (`IntType(true, 8)`) |
| `UByteType` | 8 | Unsigned 8-bit |
| `ShortType` | 16 | Signed 16-bit |
| `UShortType` | 16 | Unsigned 16-bit |
| `UIntType(size?)` | variable or fixed | Unsigned; `size=0` → variable-length |
| `SimpleFloatType(signed, fractionDigits, size)` | same as IntType | Stores `Math.round(value * 10^fractionDigits)` as int |
| `BigIntType` | variable | Signed BigInt up to 128 bits |
| `StringType` | variable | 7-bit chars; MSB marks start of new character |
| `ArrayType(elementType)` | variable | Length prefix + elements |
| `ObjectType` | sum of fields | Fields read/written in **alphabetical order** |
| `EnumType(values[])` | `ceil(log2(values.length))` bits | Stores index; array must be identical on both sides |

### IntType variable-length encoding

`IntType(signed, 0)` uses a compact prefix format:
- 3 bits: number of additional 4-bit nibble chunks (0–7)
- N×4 bits: the value (in two's complement for signed)

Good for integers whose magnitude is not known in advance.

### ObjectType field order

Fields are sorted alphabetically before encoding. This means `{ z, a }` and `{ a, z }` produce the same binary layout. Always provide all schema fields when reading, even if a field is 0 / false / empty.

To override the default alphabetical order, call `setFieldOrder(keys)` on both the writer and reader instances with the same key array:

```typescript
const writer = ObjectType.getInstance(undefined, { z: new BoolType(), a: new BoolType(), m: new BoolType() });
const reader = ObjectType.getInstance(undefined, { z: new BoolType(), a: new BoolType(), m: new BoolType() });

writer.setFieldOrder(['z', 'm', 'a']);
reader.setFieldOrder(['z', 'm', 'a']);
```

Keys not present in the schema are silently ignored. The order must be identical on both sides.

## BitReader / BitWriter directly

Use these when you need to handle reading and writing separately, or when the source data already exists.

```typescript
import { BitReader, BitWriter } from '@actualwave/multibyte-stream';

// Write
const writer = new BitWriter();
writer.setData();            // allocates default Uint8Array
writer.write(0b1010, 4);
writer.writeBit(1);
writer.writeUByte(255);

// Access written bytes (trim to actual length)
const data = writer.getData().slice(0, Math.ceil(writer.getBytePosition()));

// Read
const reader = new BitReader();
reader.setData(data);
reader.read(4);              // → 10
reader.readBit();            // → 1
reader.readUByte();          // → 255
```

### Endianness

Default is **big-endian** (MSB first). Switch per-read or per-write:

```typescript
import { Endian } from '@actualwave/multibyte-stream';

reader.setBitOrder(Endian.LITTLE);
writer.setBitOrder(Endian.LITTLE);
```

### TypedArray frame size

BitWriter and BitReader work on any TypedArray. The element size determines the "frame size":

```typescript
writer.setData(new Uint32Array(4));  // 32-bit frames
writer.getFrameSize();               // → 32
```

## Custom type classes

```typescript
import {
  IType, ITypeData, IBitWriter, IBitReader,
  TypeRegistry, addTypeDefinition,
} from '@actualwave/multibyte-stream';

class RGBType implements IType {
  static readonly type = 'rgb';

  writeTo(writer: IBitWriter, value: [number, number, number]): void {
    writer.write(value[0], 8);
    writer.write(value[1], 8);
    writer.write(value[2], 8);
  }

  readFrom(reader: IBitReader): [number, number, number] {
    return [reader.read(8), reader.read(8), reader.read(8)];
  }

  toObject(): ITypeData {
    return { type: RGBType.type };
  }

  static getInstance(): RGBType { return new RGBType(); }
  static getInstanceFor(): RGBType { return new RGBType(); }
  static fromObject(): RGBType { return new RGBType(); }

  static getTypeKeys(): Array<string | Function> {
    return [RGBType.type, RGBType];
  }
}

// Register so it can be used inside ObjectType/ArrayType schemas
addTypeDefinition(RGBType);
```

## TypeRegistry

The default registry is pre-loaded with all built-in types. Use it when you want automatic schema inference or `fromObject()` restoration.

```typescript
import {
  defaultTypeRegistry,
  addTypeDefinition,
  addTypeDefinitionFor,
  hasTypeDefinitionFor,
  getTypeDefinitionFor,
} from '@actualwave/multibyte-stream';

// Register a type under all its default keys
addTypeDefinition(MyType);

// Register under a specific key only
addTypeDefinitionFor('custom-key', MyType);

// Look up
hasTypeDefinitionFor('rgb');          // true after addTypeDefinition(RGBType)
getTypeDefinitionFor('rgb');          // → RGBType (the static class)
```

## Bundle formats

| File | Format | Use case |
|---|---|---|
| `index.js` | CommonJS | Node.js / bundlers |
| `multibyte-stream.umd.js` | UMD | Browser `<script>`, AMD, CommonJS |
| `types/` | TypeScript declarations | Editor support & type checking |

---
> Source: [burdiuz/js-multibyte-stream](https://github.com/burdiuz/js-multibyte-stream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
