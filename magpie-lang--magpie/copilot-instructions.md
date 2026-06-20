## magpie

> Comprehensive guide for writing Magpie (.mp) programs, debugging compiler errors, and using the Magpie CLI. Designed for agents who have never seen Magpie before.


# Magpie Language Programming Guide

Use this skill whenever you write, debug, or review Magpie `.mp` programs.

---

## 0) Non-negotiable Rules

1. **Binary-first:** treat compiler diagnostics as ground truth.
   - Run `magpie explain <CODE>` for any error code you don't recognize.
   - Use `--output json` for machine-readable diagnostics.
2. **Diagnose by error code first**, then apply the smallest fix.
3. **Smallest reproducer first** -- one tiny `.mp` file, then grow.
4. **Change one dimension at a time** (syntax, then types, then ownership, then backend).
5. **Prove fixes with commands and exit codes.**

---

## 1) What is Magpie

Magpie is a compiled, SSA-based language with explicit ownership, ARC memory management, and first-class GPU support. Programs are written in a low-level IR-like textual format (CSNF).

Key characteristics:
- **SSA form** -- every value is defined once, used by name (`%x`, `%y`)
- **Explicit ownership** -- you manually `borrow.shared`, `borrow.mut`, `share`, `clone.shared`
- **ARC-managed heap** -- the compiler inserts reference counting; you manage ownership transitions
- **Block-structured** -- functions contain labeled basic blocks (`bb0:`, `bb1:`) with explicit terminators
- **No implicit conversions** -- types must match exactly; `const` suffixes must match declared types
- **5 GPU backends** -- SPIR-V, Metal (MSL), PTX (CUDA), HIP (AMD), WGSL (WebGPU)

How Magpie differs from familiar languages:
- There are no expressions or statements -- only SSA instructions and terminators
- Control flow uses `br`, `cbr`, `switch` -- not `if/else/for/while`
- Variables are SSA locals (`%name`) -- assigned exactly once
- Functions are `@name` -- always prefixed with `@`
- Types are `TName` -- user-defined types start with `T`
- Comments use `;` (not `//` or `#`); doc comments use `;;`

---

## 2) Module Structure

Every `.mp` file must have a strict header in exactly this order:

```mp
module <module.path>
exports { <exported symbols> }
imports { <import groups> }
digest "<hex string>"
```

**The order is mandatory.** Swapping any header line causes a parse error (`MPP0002`).

### Minimal template (copy-paste this to start any program)

```mp
module demo.main
exports { @main }
imports { }
digest "0000000000000000"

fn @main() -> i32 {
bb0:
  ret const.i32 0
}
```

### Exports

List functions (`@name`) and types (`TName`) the module makes visible:

```mp
exports { @main, @helper, TPoint }
```

### Imports

Group imports by source module:

```mp
imports { std.io::{@println}, util.math::{@sum, TVector} }
```

Use imported functions with their full qualified path in calls:

```mp
call_void std.io.@println { args=[%msg] }
```

### Digest

A hex string for content hashing. Use `"0000000000000000"` as a placeholder during development.

---

## 3) Functions

### Function kinds

```mp
fn @name(%param: Type) -> ReturnType { ... }
async fn @name(%param: Type) -> ReturnType { ... }
unsafe fn @name(%param: Type) -> ReturnType { ... }
gpu fn @name(%param: Type) -> unit target(msl) { ... }
```

### Parameters and return types

```mp
fn @add(%a: i64, %b: i64) -> i64 {
bb0:
  %r: i64 = i.add { lhs=%a, rhs=%b }
  ret %r
}
```

Parameters are SSA names (`%name: Type`). Return type follows `->`.

### Meta blocks (optional)

```mp
fn @compute(%x: i64) -> i64 meta {
  uses { @helper, @util }
  effects { io, alloc }
  cost { time=5, space=2 }
} {
bb0:
  %r: i64 = call @helper { x=%x }
  ret %r
}
```

### Basic blocks and terminators

Every function body contains one or more blocks. Each block has a label (`bbN:`) and ends with exactly one terminator.

```
bb0:              ; block label (bb followed by a number)
  <instructions>  ; zero or more SSA assignments or void ops
  <terminator>    ; exactly one: ret, br, cbr, switch, or unreachable
```

#### Terminators

| Terminator | Syntax | Description |
|---|---|---|
| `ret` | `ret %value` or `ret const.i32 0` | Return a value |
| `ret` (void) | `ret` | Return unit (for `-> unit` functions) |
| `br` | `br bb1` | Unconditional branch |
| `cbr` | `cbr %cond bb_true bb_false` | Conditional branch (`%cond` must be `bool`) |
| `switch` | `switch %val { case 0 -> bb1 case 1 -> bb2 } else bb3` | Multi-way branch |
| `unreachable` | `unreachable` | Mark unreachable code path |

#### Branching example

```mp
fn @abs(%x: i64) -> i64 {
bb0:
  %neg: bool = icmp.slt { lhs=%x, rhs=const.i64 0 }
  cbr %neg bb1 bb2

bb1:
  %zero: i64 = const.i64 0
  %r: i64 = i.sub { lhs=%zero, rhs=%x }
  ret %r

bb2:
  ret %x
}
```

#### Phi nodes (merging values from multiple predecessors)

```mp
fn @max(%a: i64, %b: i64) -> i64 {
bb0:
  %cond: bool = icmp.sgt { lhs=%a, rhs=%b }
  cbr %cond bb1 bb2

bb1:
  br bb3

bb2:
  br bb3

bb3:
  %result: i64 = phi i64 { [bb1:%a], [bb2:%b] }
  ret %result
}
```

`phi` selects a value based on which predecessor block was taken. **Borrow values cannot appear in phi nodes** (`MPO0102`).

---

## 4) Type System

### 4.1 Primitive types

| Category | Types |
|---|---|
| Signed integers | `i1 i8 i16 i32 i64 i128` |
| Unsigned integers | `u1 u8 u16 u32 u64 u128` |
| Floats | `f16 f32 f64 bf16` |
| Boolean | `bool` |
| Unit | `unit` |

### 4.2 Builtin heap types

| Type | Description |
|---|---|
| `Str` | UTF-8 string (built-in `hash`, `eq`, `ord` -- no `impl` needed) |
| `Array<T>` | Dynamic array |
| `Map<K, V>` | Hash map (K must have `hash` + `eq` impls) |
| `TOption<T>` | Optional value -- variants: `Some { v: T }`, `None { }` |
| `TResult<Ok, Err>` | Result value -- variants: `Ok { v: Ok }`, `Err { e: Err }` |
| `TStrBuilder` | Mutable string builder |
| `TMutex<T>` | Mutual exclusion lock |
| `TRwLock<T>` | Reader-writer lock |
| `TCell<T>` | Interior mutability cell |
| `TFuture<T>` | Async future |
| `TChannelSend<T>` | Channel send endpoint |
| `TChannelRecv<T>` | Channel receive endpoint |
| `TCallable<TSigName>` | First-class callable (closure/function pointer) |

### 4.3 Ownership modifiers

Place these before a type to change ownership mode:

| Modifier | Meaning | Example |
|---|---|---|
| *(none)* | Unique owner (default) | `%p: TPoint` |
| `shared` | Reference-counted shared | `%s: shared TPoint` |
| `borrow` | Shared read-only borrow | `%b: borrow TPoint` |
| `mutborrow` | Exclusive mutable borrow | `%m: mutborrow TPoint` |
| `weak` | Weak reference (non-owning) | `%w: weak TPoint` |

**Important restrictions:**
- `TOption` and `TResult` reject `shared` and `weak` prefixes (`MPT0002`, `MPT0003`)
- Borrows cannot cross block boundaries, appear in phi, or be returned from functions

### 4.4 User-defined types

#### Heap struct (reference-counted, can contain any types)

```mp
heap struct TPoint {
  field x: i64
  field y: i64
}
```

#### Value struct (stack-allocated, cannot contain heap handles)

```mp
value struct TColor {
  field r: u8
  field g: u8
  field b: u8
}
```

If a `value struct` field contains a heap type (like `Str`), you get `MPT1005`. Use `heap struct` instead.

#### Heap enum

```mp
heap enum TShape {
  variant Circle { field radius: f64 }
  variant Rect { field w: f64, field h: f64 }
}
```

#### Value enum (deferred in v0.1 -- use `heap enum` instead)

`value enum` will produce `MPT1020`. Use `heap enum` for now.

### 4.5 Signatures and callables

```mp
sig TBinaryOp(i64, i64) -> i64
```

Then use `TCallable<TBinaryOp>` as the type for first-class function values.

### 4.6 Raw pointers

```mp
rawptr<T>   ; e.g., rawptr<i64>, rawptr<u8>
```

Raw pointer ops require `unsafe` context.

### 4.7 Type hierarchy diagram

```
                          Types
           ┌───────────────┼───────────────┐
       Primitive        Builtin          User-defined
     i8..i128        Str, Array<T>      heap struct
     u8..u128        Map<K,V>           value struct
     f16 f32 f64     TOption<T>         heap enum
     bf16 bool       TResult<Ok,Err>    (value enum: v0.1 deferred)
     unit            TStrBuilder
                     TMutex<T> ...      rawptr<T>
                     TCallable<TSig>
```

### 4.8 Trait requirements for collections

| Type | hash | eq | ord | Notes |
|---|---|---|---|---|
| All primitives | yes | yes | yes | Built-in |
| `bool` | yes | yes | yes | Built-in |
| `Str` | yes | yes | yes | Built-in -- **no explicit `impl` needed** |
| User `heap struct` | no | no | no | Must provide explicit `impl` declarations |
| User `heap enum` | no | no | no | Must provide explicit `impl` declarations |

`Map<Str, V>` works with no extra code. `Map<TMyStruct, V>` requires:

```mp
fn @hash_my(%v: borrow TMyStruct) -> u64 { bb0: ret const.u64 0 }
impl hash for TMyStruct = @hash_my

fn @eq_my(%a: borrow TMyStruct, %b: borrow TMyStruct) -> bool { bb0: ret const.bool true }
impl eq for TMyStruct = @eq_my
```

Trait impl signature requirements:
- `hash`: `(%v: borrow T) -> u64`
- `eq`: `(%a: borrow T, %b: borrow T) -> bool`
- `ord`: `(%a: borrow T, %b: borrow T) -> i32`

### 4.9 Extern declarations (FFI)

```mp
extern "c" module libc {
  fn @malloc(%size: u64) -> rawptr<u8> attrs { returns="owned" }
  fn @free(%ptr: rawptr<u8>) -> unit
}
```

Extern functions returning `rawptr` must have `attrs { returns="owned" }` or `attrs { returns="borrowed" }` (`MPF0001`).

### 4.10 Globals

```mp
global @MAX_SIZE: i64 = const.i64 1024
```

---

## 5) Ownership & Borrowing Model

### 5.1 Ownership state machine

```
             ┌──────────────────────┐
             │       Unique         │ <── initial state after `new`
             │    (sole owner)      │
             └───┬────────┬─────┬──┘
                 │        │     │
        share{}  │        │     │ borrow.mut{}
    (consumes    │        │     │
     original)   │        │     v
                 v        │  ┌──────────────────┐
          ┌──────────┐    │  │    MutBorrow      │
          │  Shared  │    │  │ (exclusive r/w)   │
          │(ref-cnt) │    │  │ block-scoped only │
          └────┬─────┘    │  └──────────────────┘
               │          │
    clone      │          │ borrow.shared{}
    .shared{}  │          │
               v          v
          ┌──────────┐  ┌──────────────────┐
          │  Shared  │  │     Borrow       │
          │  (copy)  │  │  (shared r/o)    │
          └──────────┘  │  block-scoped    │
                        └──────────────────┘
```

### 5.2 Key rules

| Rule | Consequence |
|---|---|
| `share { v=%x }` | **Consumes** `%x`. `%x` is dead after this. |
| `borrow.mut { v=%x }` | Creates exclusive mutable access, scoped to current block only |
| `borrow.shared { v=%x }` | Creates shared read access, scoped to current block only |
| `clone.shared { v=%x }` | Creates a new shared ref. Original survives. |
| Borrows **cannot** cross block boundaries | `MPO0101` |
| Borrows **cannot** appear in phi nodes | `MPO0102` |
| Borrows **cannot** be returned from functions | `MPHIR03` |
| Cannot use a value after it has been moved/consumed | `MPO0007` |
| Cannot move a value while a borrow is active | `MPO0011` |

### 5.3 Borrow lifecycle per block

```
  Block entry
      │
      │  %x: TPoint = ...              (unique value)
      │
      │  %xb: borrow TPoint =          (borrow starts here ─┐
      │    borrow.shared { v=%x }                             │ valid scope
      │                                                       │
      │  getfield { obj=%xb, field=y }  (OK: same block)     │
      │                                                       │
      │  br bb1                         (block exit ──────────┘ borrow dies)
      │
      v
  bb1:
      │  %xb is DEAD here. Cannot reference it.
      │  To borrow again: %xb2 = borrow.shared { v=%x }
```

### 5.4 Correct multi-block borrow pattern

**Wrong** -- borrow crosses block boundary (`MPO0101`):
```mp
bb0:
  %pb: borrow TPoint = borrow.shared { v=%p }
  br bb1
bb1:
  %x: i64 = getfield { obj=%pb, field=x }   ; ERROR: %pb from bb0
```

**Correct** -- re-borrow in each block:
```mp
bb0:
  %pb: borrow TPoint = borrow.shared { v=%p }
  %x: i64 = getfield { obj=%pb, field=x }   ; OK: same block
  br bb1
bb1:
  %pb2: borrow TPoint = borrow.shared { v=%p }
  %y: i64 = getfield { obj=%pb2, field=y }  ; OK: new borrow in bb1
  ret %y
```

### 5.5 Mutation then read pattern (split across blocks)

To mutate a struct then read it, you must split into separate blocks:

```mp
bb0:
  %p: TPoint = new TPoint { x=const.i64 1, y=const.i64 2 }
  %pm: mutborrow TPoint = borrow.mut { v=%p }
  setfield { obj=%pm, field=y, val=const.i64 99 }
  br bb1                    ; end mutborrow scope

bb1:
  %pb: borrow TPoint = borrow.shared { v=%p }
  %y: i64 = getfield { obj=%pb, field=y }   ; reads 99
  ret %y
```

### 5.6 Mutation vs read receiver requirements

| Operation | Required receiver |
|---|---|
| `getfield` | `borrow T` or `mutborrow T` |
| `setfield` | `mutborrow T` |
| `arr.push`, `arr.set`, `arr.sort`, `arr.pop` | `mutborrow Array<T>` |
| `arr.len`, `arr.get`, `arr.slice`, `arr.contains`, `arr.map`, `arr.filter`, `arr.reduce` | `borrow Array<T>` (or `mutborrow`) |
| `map.set`, `map.delete`, `map.delete_void` | `mutborrow Map<K,V>` |
| `map.len`, `map.get`, `map.get_ref`, `map.contains_key`, `map.keys`, `map.values` | `borrow Map<K,V>` (or `mutborrow`) |
| `str.builder.append_*` | `mutborrow TStrBuilder` (or unique) |

### 5.7 Move/consume semantics

These operations **consume** (move) their input -- the value is dead afterward:
- `share { v=%x }` -- consumes `%x`
- `new TFoo { field=%x }` -- consumes each field value
- `enum.new<V> { v=%x }` -- consumes payload values
- `callable.capture @fn { cap=%x }` -- consumes captured values
- `str.concat { a=%x, b=%y }` -- consumes both strings
- `str.builder.build { b=%sb }` -- consumes the builder
- `setfield { obj=%m, field=f, val=%x }` -- consumes `%x` (the new value)
- `arr.push { arr=%m, val=%x }` -- consumes `%x`
- `map.set { map=%m, key=%k, val=%v }` -- consumes `%k` and `%v`
- `ret %x` -- consumes `%x` for move-only types

---

## 6) Complete Opcode Reference

Every instruction is either a **value-producing op** (appears in an SSA assignment) or a **void op** (standalone statement).

SSA assignment form: `%name: Type = <value-op>`
Void op form: `<void-op>` (no assignment)

### 6.1 Constants

```mp
%x: i32  = const.i32 42
%y: i64  = const.i64 -1
%z: f64  = const.f64 3.14
%b: bool = const.bool true
%s: Str  = const.Str "hello"
%u: u32  = const.u32 7
```

**The type suffix on `const` must exactly match the declared SSA type.** `const.i32` for `i32`, `const.i64` for `i64`, etc. Mismatch causes `MPT2014`/`MPT2015`.

### 6.2 Integer arithmetic / bitwise

All take `{ lhs=V, rhs=V }` and produce the same integer type.

| Op | Description |
|---|---|
| `i.add` | Addition |
| `i.sub` | Subtraction |
| `i.mul` | Multiplication |
| `i.sdiv` | Signed division |
| `i.udiv` | Unsigned division |
| `i.srem` | Signed remainder |
| `i.urem` | Unsigned remainder |
| `i.add.wrap` | Wrapping add |
| `i.sub.wrap` | Wrapping sub |
| `i.mul.wrap` | Wrapping mul |
| `i.add.checked` | Checked add (traps on overflow) |
| `i.sub.checked` | Checked sub |
| `i.mul.checked` | Checked mul |
| `i.and` | Bitwise AND |
| `i.or` | Bitwise OR |
| `i.xor` | Bitwise XOR |
| `i.shl` | Shift left |
| `i.lshr` | Logical shift right |
| `i.ashr` | Arithmetic shift right |

Example:
```mp
%r: i64 = i.add { lhs=%a, rhs=%b }
%c: i64 = i.mul.checked { lhs=%r, rhs=const.i64 2 }
```

### 6.3 Float arithmetic

All take `{ lhs=V, rhs=V }` and produce the same float type.

| Op | Description |
|---|---|
| `f.add` | Addition |
| `f.sub` | Subtraction |
| `f.mul` | Multiplication |
| `f.div` | Division |
| `f.rem` | Remainder |
| `f.add.fast` | Fast-math add |
| `f.sub.fast` | Fast-math sub |
| `f.mul.fast` | Fast-math mul |
| `f.div.fast` | Fast-math div |

### 6.4 Comparisons

All take `{ lhs=V, rhs=V }` and produce `bool`.

**Integer comparisons (`icmp.*`):**

| Op | Meaning |
|---|---|
| `icmp.eq` | Equal |
| `icmp.ne` | Not equal |
| `icmp.slt` | Signed less than |
| `icmp.sgt` | Signed greater than |
| `icmp.sle` | Signed less or equal |
| `icmp.sge` | Signed greater or equal |
| `icmp.ult` | Unsigned less than |
| `icmp.ugt` | Unsigned greater than |
| `icmp.ule` | Unsigned less or equal |
| `icmp.uge` | Unsigned greater or equal |

**Float comparisons (`fcmp.*`):**

| Op | Meaning |
|---|---|
| `fcmp.oeq` | Ordered equal |
| `fcmp.one` | Ordered not equal |
| `fcmp.olt` | Ordered less than |
| `fcmp.ogt` | Ordered greater than |
| `fcmp.ole` | Ordered less or equal |
| `fcmp.oge` | Ordered greater or equal |

### 6.5 Calls

| Op | Syntax | Produces value? |
|---|---|---|
| `call` | `call @fn { key=V, ... }` | Yes |
| `call_void` | `call_void @fn { key=V, ... }` | No (void) |
| `call` with type args | `call @fn<i64> { key=V, ... }` | Yes |
| `try` | `try @fn { key=V, ... }` | Yes |

Qualified calls to imported functions:
```mp
%r: i64 = call util.math.@sum { a=%x, b=%y }
call_void std.io.@println { args=[%msg] }
```

**Note:** call argument keys are used for readability but lowering preserves argument order. Keep key order stable.

### 6.6 Async / suspend

| Op | Syntax | Produces value? |
|---|---|---|
| `suspend.call` | `suspend.call @fn { key=V, ... }` | Yes |
| `suspend.await` | `suspend.await { fut=V }` | Yes |

```mp
async fn @fetch(%id: u64) -> Str meta { uses { @get_data } } {
bb0:
  %data: Str = suspend.call @get_data { id=%id }
  ret %data
}
```

### 6.7 Indirect calls (callables)

| Op | Syntax | Produces value? |
|---|---|---|
| `call.indirect` | `call.indirect %callable { key=V, ... }` | Yes |
| `call_void.indirect` | `call_void.indirect %callable { key=V, ... }` | No (void) |

### 6.8 Struct / enum ops

| Op | Syntax | Produces value? |
|---|---|---|
| `new` | `new TFoo { field1=V, field2=V }` | Yes |
| `getfield` | `getfield { obj=V, field=name }` | Yes |
| `setfield` | `setfield { obj=V, field=name, val=V }` | No (void) |
| `phi` | `phi Type { [bb0:V], [bb1:V] }` | Yes |
| `enum.new` | `enum.new<Variant> { key=V, ... }` | Yes |
| `enum.tag` | `enum.tag { v=V }` | Yes (i32) |
| `enum.is` | `enum.is<Variant> { v=V }` | Yes (bool) |
| `enum.payload` | `enum.payload<Variant> { v=V }` | Yes |

**`getfield` requires a borrow/mutborrow receiver** (`MPHIR01`).
**`setfield` requires a mutborrow receiver** (`MPHIR02`).
**`setfield` uses `val=` key** (not `value=`).
Both `getfield` and `setfield` accept keys in any order.

### 6.9 Ownership conversion ops

| Op | Syntax | Description |
|---|---|---|
| `share` | `share { v=V }` | Unique -> Shared (consumes input) |
| `clone.shared` | `clone.shared { v=V }` | Copy a shared ref (original survives) |
| `clone.weak` | `clone.weak { v=V }` | Copy a weak ref |
| `weak.downgrade` | `weak.downgrade { v=V }` | Shared -> Weak |
| `weak.upgrade` | `weak.upgrade { v=V }` | Weak -> TOption<Shared> |
| `borrow.shared` | `borrow.shared { v=V }` | Create shared read-only borrow |
| `borrow.mut` | `borrow.mut { v=V }` | Create exclusive mutable borrow |
| `cast` | `cast<FromPrim, ToPrim> { v=V }` | Primitive-to-primitive cast |

`cast` only works between primitive types (`MPT2010`/`MPT2011`):
```mp
%narrow: i32 = cast<i64, i32> { v=%wide }
```

### 6.10 Raw pointer ops (unsafe context required)

| Op | Syntax | Produces value? |
|---|---|---|
| `ptr.null` | `ptr.null<Type>` | Yes |
| `ptr.addr` | `ptr.addr<Type> { p=V }` | Yes (u64) |
| `ptr.from_addr` | `ptr.from_addr<Type> { addr=V }` | Yes (rawptr) |
| `ptr.add` | `ptr.add<Type> { p=V, count=V }` | Yes (rawptr) |
| `ptr.load` | `ptr.load<Type> { p=V }` | Yes |
| `ptr.store` | `ptr.store<Type> { p=V, v=V }` | No (void) |

Using any `ptr.*` outside `unsafe fn` or `unsafe { }` causes `MPS0024`.

### 6.11 Callable capture

```mp
%cb: TCallable<TSigName> = callable.capture @fn_name { cap1=%val1, cap2=%val2 }
```

Use `{ }` for no captures. The captures are consumed (moved into the callable).

### 6.12 Array ops

**Value-producing (read):**

| Op | Syntax | Receiver |
|---|---|---|
| `arr.new<T>` | `arr.new<T> { cap=V }` | n/a (creates new) |
| `arr.len` | `arr.len { arr=V }` | borrow/mutborrow |
| `arr.get` | `arr.get { arr=V, idx=V }` | borrow/mutborrow |
| `arr.pop` | `arr.pop { arr=V }` | mutborrow |
| `arr.slice` | `arr.slice { arr=V, start=V, end=V }` | borrow/mutborrow |
| `arr.contains` | `arr.contains { arr=V, val=V }` | borrow/mutborrow (elem needs `eq`) |
| `arr.map` | `arr.map { arr=V, fn=V }` | borrow/mutborrow |
| `arr.filter` | `arr.filter { arr=V, fn=V }` | borrow/mutborrow |
| `arr.reduce` | `arr.reduce { arr=V, init=V, fn=V }` | borrow/mutborrow |

**Void (mutation):**

| Op | Syntax | Receiver |
|---|---|---|
| `arr.push` | `arr.push { arr=V, val=V }` | mutborrow |
| `arr.set` | `arr.set { arr=V, idx=V, val=V }` | mutborrow |
| `arr.sort` | `arr.sort { arr=V }` | mutborrow (elem needs `ord`) |
| `arr.foreach` | `arr.foreach { arr=V, fn=V }` | borrow/mutborrow |

### 6.13 Map ops

**Value-producing (read):**

| Op | Syntax | Receiver |
|---|---|---|
| `map.new<K, V>` | `map.new<K, V> { }` | n/a (K needs `hash` + `eq`) |
| `map.len` | `map.len { map=V }` | borrow/mutborrow |
| `map.get` | `map.get { map=V, key=V }` | borrow/mutborrow (V must be Dupable, else `MPO0103`) |
| `map.get_ref` | `map.get_ref { map=V, key=V }` | borrow/mutborrow |
| `map.delete` | `map.delete { map=V, key=V }` | mutborrow |
| `map.contains_key` | `map.contains_key { map=V, key=V }` | borrow/mutborrow |
| `map.keys` | `map.keys { map=V }` | borrow/mutborrow |
| `map.values` | `map.values { map=V }` | borrow/mutborrow |

**Void (mutation):**

| Op | Syntax | Receiver |
|---|---|---|
| `map.set` | `map.set { map=V, key=V, val=V }` | mutborrow |
| `map.delete_void` | `map.delete_void { map=V, key=V }` | mutborrow |

### 6.14 String / builder / JSON ops

**Value-producing:**

| Op | Syntax | Notes |
|---|---|---|
| `str.concat` | `str.concat { a=V, b=V }` | Consumes both inputs |
| `str.len` | `str.len { s=V }` | `s` must be borrow/mutborrow |
| `str.eq` | `str.eq { a=V, b=V }` | Both must be borrow/mutborrow |
| `str.slice` | `str.slice { s=V, start=V, end=V }` | `s` must be borrow/mutborrow |
| `str.bytes` | `str.bytes { s=V }` | `s` must be borrow/mutborrow |
| `str.builder.new` | `str.builder.new { }` | Creates new builder |
| `str.builder.build` | `str.builder.build { b=V }` | Consumes builder, produces Str |
| `str.parse_i64` | `str.parse_i64 { s=V }` | Returns `TResult<i64, Str>` |
| `str.parse_u64` | `str.parse_u64 { s=V }` | Returns `TResult<u64, Str>` |
| `str.parse_f64` | `str.parse_f64 { s=V }` | Returns `TResult<f64, Str>` |
| `str.parse_bool` | `str.parse_bool { s=V }` | Returns `TResult<bool, Str>` |
| `json.encode<T>` | `json.encode<T> { v=V }` | Returns `TResult<Str, Str>` |
| `json.decode<T>` | `json.decode<T> { s=V }` | Returns `TResult<rawptr<u8>, Str>` |

**Void (builder append):**

| Op | Syntax |
|---|---|
| `str.builder.append_str` | `str.builder.append_str { b=V, s=V }` |
| `str.builder.append_i64` | `str.builder.append_i64 { b=V, v=V }` |
| `str.builder.append_i32` | `str.builder.append_i32 { b=V, v=V }` |
| `str.builder.append_f64` | `str.builder.append_f64 { b=V, v=V }` |
| `str.builder.append_bool` | `str.builder.append_bool { b=V, v=V }` |

### 6.15 GPU ops

**Value-producing:**

| Op | Syntax | Notes |
|---|---|---|
| `gpu.thread_id` | `gpu.thread_id { dim=V }` | dim: `const.u32 0/1/2` (x/y/z) |
| `gpu.workgroup_id` | `gpu.workgroup_id { dim=V }` | dim: 0/1/2 |
| `gpu.workgroup_size` | `gpu.workgroup_size { dim=V }` | dim: 0/1/2 |
| `gpu.global_id` | `gpu.global_id { dim=V }` | dim: 0/1/2 |
| `gpu.buffer_load<T>` | `gpu.buffer_load<T> { buf=V, idx=V }` | Typed load |
| `gpu.buffer_len<T>` | `gpu.buffer_len<T> { buf=V }` | Buffer length |
| `gpu.shared<count, T>` | `gpu.shared<256, f32>` | Shared memory |
| `gpu.launch` | `gpu.launch { device=V, kernel=@fn, grid=V, block=V, args=V }` | **Strict key order** |
| `gpu.launch_async` | `gpu.launch_async { device=V, kernel=@fn, grid=V, block=V, args=V }` | **Strict key order** |

**Void:**

| Op | Syntax |
|---|---|
| `gpu.barrier` | `gpu.barrier` |
| `gpu.buffer_store<T>` | `gpu.buffer_store<T> { buf=V, idx=V, v=V }` |

### 6.16 Other void ops

| Op | Syntax |
|---|---|
| `panic` | `panic { msg=V }` |

### 6.17 Arg value grammar

Arguments in call/GPU forms can be:
1. Value ref: `%x` or `const.i64 42`
2. List: `[%a, %b, @fn_ref]`
3. Function ref: `@fn` or `module.@fn`

### 6.18 Key-ordering summary

| Op family | Key order required? | Notes |
|---|---|---|
| `getfield` | No -- any order | Parser accepts keys in any order |
| `setfield` | No -- any order | Uses `val=` key (not `value=`) |
| `gpu.launch` | **Yes -- strict** | `device, kernel, grid, block, args` |
| `gpu.launch_async` | **Yes -- strict** | `device, kernel, grid, block, args` |
| All other ops | Stable recommended | Keys preserved by order |

---

## 7) GPU Programming

### 7.1 Overview

Magpie supports 5 GPU backends:

| Backend | Target | Output | Control flow |
|---|---|---|---|
| SPIR-V | `spv` | Binary SPIR-V module | Native CFG |
| Metal | `msl` | `.metal` source | Structurized |
| PTX | `ptx` | LLVM IR -> PTX via llc | Native |
| HIP | `hip` | LLVM IR -> HSACO via llc+lld | Native |
| WGSL | `wgsl` | `.wgsl` source | Structurized |

### 7.2 GPU function syntax

```mp
gpu fn @kernel_name(%param: rawptr<f32>) -> unit target(msl) {
bb0:
  ret
}
```

With workgroup size and capability requirements:
```mp
gpu fn @vector_add(%a: rawptr<f32>, %b: rawptr<f32>, %out: rawptr<f32>) -> unit target(msl) workgroup(64, 1, 1) {
bb0:
  %gid: u32 = gpu.global_id { dim=const.u32 0 }
  %x: f32 = gpu.buffer_load<f32> { buf=%a, idx=%gid }
  %y: f32 = gpu.buffer_load<f32> { buf=%b, idx=%gid }
  %z: f32 = f.add { lhs=%x, rhs=%y }
  gpu.buffer_store<f32> { buf=%out, idx=%gid, v=%z }
  ret
}
```

Unsafe GPU function with capability requirements:
```mp
unsafe gpu fn @advanced_kernel(%buf: rawptr<i32>) -> unit target(ptx) workgroup(256, 1, 1) requires(device_malloc) {
bb0:
  ret
}
```

### 7.3 GPU kernel restrictions

GPU kernels **cannot** use:

| Restriction | Diagnostic |
|---|---|
| `new` (heap allocation) | `MPG_CORE_1100` |
| `Str` type | `MPG_CORE_1104` |
| `Array<T>` | `MPG_CORE_1105` |
| `Map<K,V>` | `MPG_CORE_1106` |
| `TCallable` (dynamic dispatch) | `MPG_CORE_1102`, `MPG_CORE_1107` |
| Recursion | `MPG_CORE_1103` |

Use `rawptr<T>` for buffer parameters and primitive types for computation.

### 7.4 Launching kernels from host code

```mp
%result: unit = gpu.launch {
  device=%dev,
  kernel=@vector_add,
  grid=[const.u32 256, const.u32 1, const.u32 1],
  block=[const.u32 64, const.u32 1, const.u32 1],
  args=[%buf_a, %buf_b, %buf_out]
}
```

Grid and block dimensions are 3D: `[x, y, z]`. Key order is strict: `device, kernel, grid, block, args`.

### 7.5 Building GPU programs

```bash
magpie --entry src/main.mp --emit exe,msl build      # Metal
magpie --entry src/main.mp --emit exe,spv build      # Vulkan/SPIR-V
magpie --entry src/main.mp --emit exe,ptx build      # NVIDIA
magpie --entry src/main.mp --emit exe,hip build      # AMD
magpie --entry src/main.mp --emit exe,wgsl build     # WebGPU
```

### 7.6 Manifest GPU config

In `Magpie.toml`:
```toml
[gpu]
enabled = true
backend = "metal"
device_index = 0
```

---

## 8) Complete Working Examples

### 8.1 Minimal hello (i32 return)

```mp
module demo.main
exports { @main }
imports { }
digest "0000000000000000"

fn @main() -> i32 {
bb0:
  ret const.i32 0
}
```

### 8.2 Arithmetic

```mp
module demo.math
exports { @compute }
imports { }
digest "0000000000000000"

fn @compute(%a: i64, %b: i64) -> i64 {
bb0:
  %sum: i64 = i.add { lhs=%a, rhs=%b }
  %doubled: i64 = i.mul { lhs=%sum, rhs=const.i64 2 }
  %checked: i64 = i.add.checked { lhs=%doubled, rhs=const.i64 1 }
  ret %checked
}
```

### 8.3 Struct create + field access (borrow pattern)

```mp
module demo.structs
exports { @get_y }
imports { }
digest "0000000000000000"

heap struct TPoint {
  field x: i64
  field y: i64
}

fn @get_y() -> i64 {
bb0:
  %p: TPoint = new TPoint { x=const.i64 10, y=const.i64 20 }
  %pb: borrow TPoint = borrow.shared { v=%p }
  %y: i64 = getfield { obj=%pb, field=y }
  ret %y
}
```

### 8.4 Struct mutation (mutborrow, split blocks)

```mp
module demo.mutation
exports { @mutate_point }
imports { }
digest "0000000000000000"

heap struct TPoint {
  field x: i64
  field y: i64
}

fn @mutate_point() -> i64 {
bb0:
  %p: TPoint = new TPoint { x=const.i64 1, y=const.i64 2 }
  %pm: mutborrow TPoint = borrow.mut { v=%p }
  setfield { obj=%pm, field=y, val=const.i64 99 }
  br bb1

bb1:
  %pb: borrow TPoint = borrow.shared { v=%p }
  %y: i64 = getfield { obj=%pb, field=y }
  ret %y
}
```

### 8.5 TOption pattern

```mp
module demo.option
exports { @maybe }
imports { }
digest "0000000000000000"

fn @maybe(%flag: bool) -> i64 {
bb0:
  cbr %flag bb1 bb2

bb1:
  %some: TOption<i64> = enum.new<Some> { v=const.i64 42 }
  br bb3

bb2:
  %none: TOption<i64> = enum.new<None> { }
  br bb3

bb3:
  %opt: TOption<i64> = phi TOption<i64> { [bb1:%some], [bb2:%none] }
  %is_some: bool = enum.is<Some> { v=%opt }
  cbr %is_some bb4 bb5

bb4:
  %val: i64 = enum.payload<Some> { v=%opt }
  ret %val

bb5:
  ret const.i64 0
}
```

### 8.6 TResult pattern

```mp
module demo.result
exports { @safe_divide }
imports { }
digest "0000000000000000"

fn @safe_divide(%a: i32, %b: i32) -> TResult<i32, Str> {
bb0:
  %is_zero: bool = icmp.eq { lhs=%b, rhs=const.i32 0 }
  cbr %is_zero bb1 bb2

bb1:
  %msg: Str = const.Str "division by zero"
  %err: TResult<i32, Str> = enum.new<Err> { e=%msg }
  ret %err

bb2:
  %result: i32 = i.sdiv { lhs=%a, rhs=%b }
  %ok: TResult<i32, Str> = enum.new<Ok> { v=%result }
  ret %ok
}
```

### 8.7 Heap enum with tag dispatch

```mp
module demo.enums
exports { @classify }
imports { }
digest "0000000000000000"

heap enum TShape {
  variant Circle { field radius: f64 }
  variant Rect { field w: f64, field h: f64 }
}

fn @classify(%s: borrow TShape) -> i32 {
bb0:
  %tag: i32 = enum.tag { v=%s }
  %is_circle: bool = icmp.eq { lhs=%tag, rhs=const.i32 0 }
  cbr %is_circle bb1 bb2

bb1:
  ret const.i32 1

bb2:
  ret const.i32 2
}
```

### 8.8 Array push/len (mutborrow then borrow across blocks)

```mp
module demo.arrays
exports { @arr_demo }
imports { }
digest "0000000000000000"

fn @arr_demo() -> i64 {
bb0:
  %arr: Array<i64> = arr.new<i64> { cap=const.i64 8 }
  %arrm: mutborrow Array<i64> = borrow.mut { v=%arr }
  arr.push { arr=%arrm, val=const.i64 10 }
  arr.push { arr=%arrm, val=const.i64 20 }
  arr.push { arr=%arrm, val=const.i64 30 }
  br bb1

bb1:
  %arrb: borrow Array<i64> = borrow.shared { v=%arr }
  %len: i64 = arr.len { arr=%arrb }
  ret %len
}
```

### 8.9 Map with Str keys (no impl needed)

```mp
module demo.maps
exports { @map_demo }
imports { }
digest "0000000000000000"

fn @map_demo() -> i64 {
bb0:
  %m: Map<Str, i64> = map.new<Str, i64> { }
  %mb: mutborrow Map<Str, i64> = borrow.mut { v=%m }
  map.set { map=%mb, key=const.Str "alice", val=const.i64 100 }
  map.set { map=%mb, key=const.Str "bob", val=const.i64 200 }
  br bb1

bb1:
  %mr: borrow Map<Str, i64> = borrow.shared { v=%m }
  %len: i64 = map.len { map=%mr }
  ret %len
}
```

### 8.10 String ops and builder

```mp
module demo.strings
exports { @build_greeting }
imports { }
digest "0000000000000000"

fn @build_greeting() -> Str {
bb0:
  %sb: TStrBuilder = str.builder.new { }
  %hello: Str = const.Str "Hello, "
  %hb: borrow Str = borrow.shared { v=%hello }
  str.builder.append_str { b=%sb, s=%hb }
  str.builder.append_i32 { b=%sb, v=const.i32 42 }
  %result: Str = str.builder.build { b=%sb }
  ret %result
}
```

### 8.11 String parsing (TResult)

```mp
module demo.parse
exports { @parse_demo }
imports { }
digest "0000000000000000"

fn @parse_demo() -> i64 {
bb0:
  %s: Str = const.Str "123"
  %sb: borrow Str = borrow.shared { v=%s }
  %parsed: TResult<i64, Str> = str.parse_i64 { s=%sb }
  %is_ok: bool = enum.is<Ok> { v=%parsed }
  cbr %is_ok bb1 bb2

bb1:
  %val: i64 = enum.payload<Ok> { v=%parsed }
  ret %val

bb2:
  ret const.i64 -1
}
```

### 8.12 TCallable capture + indirect call

```mp
module demo.callable
exports { @callable_demo }
imports { }
digest "0000000000000000"

sig TUnaryOp(i64) -> i64

fn @double(%x: i64) -> i64 {
bb0:
  %r: i64 = i.mul { lhs=%x, rhs=const.i64 2 }
  ret %r
}

fn @callable_demo() -> i64 {
bb0:
  %cb: TCallable<TUnaryOp> = callable.capture @double { }
  %result: i64 = call.indirect %cb { x=const.i64 21 }
  ret %result
}
```

### 8.13 Ownership: share, clone.shared

```mp
module demo.sharing
exports { @share_demo }
imports { }
digest "0000000000000000"

heap struct TPerson {
  field name: Str
  field age: i32
}

fn @share_demo() -> i32 {
bb0:
  %name: Str = const.Str "Alice"
  %p: TPerson = new TPerson { name=%name, age=const.i32 30 }
  ; share consumes %p -- %p is dead after this
  %sp: shared TPerson = share { v=%p }
  ; clone.shared copies the shared ref -- %sp survives
  %sp2: shared TPerson = clone.shared { v=%sp }
  %b: borrow TPerson = borrow.shared { v=%sp }
  %age: i32 = getfield { obj=%b, field=age }
  ret %age
}
```

### 8.14 Async function with suspend.call

```mp
module demo.async_example
exports { @fetch_data }
imports { }
digest "0000000000000000"

fn @get_connection() -> Str {
bb0:
  %s: Str = const.Str "conn:1"
  ret %s
}

fn @process(%conn: Str, %id: u64) -> Str {
bb0:
  ret %conn
}

async fn @fetch_data(%id: u64) -> Str meta { uses { @get_connection, @process } } {
bb0:
  %conn: Str = suspend.call @get_connection { }
  %result: Str = call @process { conn=%conn, id=%id }
  ret %result
}
```

### 8.15 GPU kernel with buffer ops

```mp
module demo.gpu
exports { @vector_add }
imports { }
digest "0000000000000000"

gpu fn @vector_add(%a: rawptr<f32>, %b: rawptr<f32>, %out: rawptr<f32>) -> unit target(msl) workgroup(64, 1, 1) {
bb0:
  %gid: u32 = gpu.global_id { dim=const.u32 0 }
  %x: f32 = gpu.buffer_load<f32> { buf=%a, idx=%gid }
  %y: f32 = gpu.buffer_load<f32> { buf=%b, idx=%gid }
  %sum: f32 = f.add { lhs=%x, rhs=%y }
  gpu.buffer_store<f32> { buf=%out, idx=%gid, v=%sum }
  ret
}
```

### 8.16 Unsafe raw pointer block

```mp
module demo.unsafe_demo
exports { @raw_demo }
imports { }
digest "0000000000000000"

unsafe fn @raw_demo() -> i64 {
bb0:
  %p: rawptr<i64> = ptr.null<i64>
  %addr: u64 = ptr.addr<i64> { p=%p }
  ret const.i64 0
}
```

Or inline unsafe block in a normal function:
```mp
fn @inline_unsafe() -> i64 {
bb0:
  unsafe {
    %p: rawptr<i64> = ptr.null<i64>
  }
  ret const.i64 0
}
```

### 8.17 Trait impl for Map key (user struct)

```mp
module demo.traits
exports { @trait_demo }
imports { }
digest "0000000000000000"

heap struct TKey {
  field id: i64
}

fn @hash_key(%k: borrow TKey) -> u64 {
bb0:
  %kb: borrow TKey = borrow.shared { v=%k }
  %id: i64 = getfield { obj=%kb, field=id }
  %h: u64 = cast<i64, u64> { v=%id }
  ret %h
}
impl hash for TKey = @hash_key

fn @eq_key(%a: borrow TKey, %b: borrow TKey) -> bool {
bb0:
  %ab: borrow TKey = borrow.shared { v=%a }
  %bb: borrow TKey = borrow.shared { v=%b }
  %aid: i64 = getfield { obj=%ab, field=id }
  %bid: i64 = getfield { obj=%bb, field=id }
  %eq: bool = icmp.eq { lhs=%aid, rhs=%bid }
  ret %eq
}
impl eq for TKey = @eq_key

fn @trait_demo() -> i64 {
bb0:
  %m: Map<TKey, i64> = map.new<TKey, i64> { }
  %mb: mutborrow Map<TKey, i64> = borrow.mut { v=%m }
  %k: TKey = new TKey { id=const.i64 1 }
  map.set { map=%mb, key=%k, val=const.i64 42 }
  br bb1

bb1:
  %mr: borrow Map<TKey, i64> = borrow.shared { v=%m }
  %len: i64 = map.len { map=%mr }
  ret %len
}
```

---

## 9) Error Code Reference

### 9.1 Diagnostic triage flowchart

```
Got an error code?
       │
       v
  ┌───────────┐
  │ Code      │
  │ prefix?   │
  └─────┬─────┘
        │
   ┌────┼────┬────┬────┬────┐
   │    │    │    │    │    │
  MPP  MPS  MPT  MPHIR MPO  MPG/MPL
   │    │    │    │    │    │
   v    v    v    v    v    v
 Parse Resolve Type HIR  Own  GPU/
 error SSA/  mis-  bor- ship  Link/
 head  dom   match row  mode  Lint
 order unsafe trait viol move
 comma  ctx  impl  ation issue
```

### 9.2 Complete error code table

#### Parse (`MPP*`)

| Code | Description |
|---|---|
| `MPP0001` | Source I/O / lexer read issue |
| `MPP0002` | Syntax error (missing comma, wrong header order, etc.) |
| `MPP0003` | Artifact emission failure |

#### Resolve / SSA (`MPS*`)

| Code | Description |
|---|---|
| `MPS0000` | Generic module resolution failure |
| `MPS0001` | Duplicate definition (module path or SSA local) |
| `MPS0002` | Unresolved reference / use-before-def |
| `MPS0003` | Dominance violation |
| `MPS0004` | Import/local namespace conflict |
| `MPS0005` | Type import/local type conflict |
| `MPS0006` | Ambiguous import name |
| `MPS0008` | Invalid CFG target / phi type legality |
| `MPS0009` | Duplicate block label |
| `MPS0010` | Structural/type invariant failure |
| `MPS0011` | Duplicate SSA local in lowering |
| `MPS0012` | Call arity mismatch |
| `MPS0013` | Expected single arg value |
| `MPS0014` | Invalid fn ref in scalar position |
| `MPS0015` | Invalid fn ref inside list |
| `MPS0016` | Invalid plain-value argument uses fn ref |
| `MPS0017` | Invalid branch predicate type (must be `bool`) |
| `MPS0020` | Duplicate function/global symbol |
| `MPS0021` | Duplicate type symbol |
| `MPS0022` | Duplicate @ namespace symbol |
| `MPS0023` | Duplicate `sig` symbol |
| `MPS0024` | `ptr.*` outside unsafe context |
| `MPS0025` | Unsafe fn call outside unsafe context |

#### FFI (`MPF*`)

| Code | Description |
|---|---|
| `MPF0001` | Extern rawptr return missing `attrs { returns="owned" }` |

#### Type (`MPT*`)

| Code | Description |
|---|---|
| `MPT0001` | Unknown primitive type |
| `MPT0002` | `shared`/`weak` invalid on `TOption` |
| `MPT0003` | `shared`/`weak` invalid on `TResult` |
| `MPT1005` | Value struct contains heap handle |
| `MPT1020` | Value enum deferred in v0.1 |
| `MPT1021` | Aggregate type deferred in v0.1 |
| `MPT1023` | Missing required trait impl (`hash`/`eq`/`ord`) |
| `MPT1030` | `suspend.call` on non-function target in v0.1 |
| `MPT1200` | Orphan impl |
| `MPT2001` | Call arity mismatch |
| `MPT2002` | Call arg unknown type |
| `MPT2003` | Call arg type mismatch |
| `MPT2004` | Call target not found |
| `MPT2005` | Invalid generic type argument |
| `MPT2006` | `getfield` object unknown type |
| `MPT2007` | `getfield` requires borrow/mutborrow struct |
| `MPT2008` | `getfield` target not a struct |
| `MPT2009` | Missing struct field |
| `MPT2010` | `cast` operand unknown |
| `MPT2011` | `cast` only primitive-to-primitive |
| `MPT2012` | Numeric lhs unknown type |
| `MPT2013` | Numeric rhs unknown type |
| `MPT2014` | Numeric operands type mismatch |
| `MPT2015` | Wrong primitive family for numeric op |
| `MPT2016` | Duplicate field in constructor |
| `MPT2017` | Unknown field in constructor |
| `MPT2018` | Field value unknown type |
| `MPT2019` | Field type mismatch in constructor |
| `MPT2020` | Missing required field in constructor |
| `MPT2021` | `new` target must be struct |
| `MPT2022` | Unknown struct target |
| `MPT2023` | Invalid variant for TOption |
| `MPT2024` | Invalid variant for TResult |
| `MPT2025` | `enum.new` result type not enum |
| `MPT2026` | User enum variant not found |
| `MPT2027` | `enum.new` target must be enum |
| `MPT2028` | Trait impl parameter count mismatch |
| `MPT2029` | Trait impl return type mismatch |
| `MPT2030` | Trait impl first param must be `borrow T` |
| `MPT2031` | Trait impl params must both match `borrow T` |
| `MPT2032` | Impl target function missing |
| `MPT2033` | Parse/JSON result type shape mismatch |
| `MPT2034` | Parse/JSON input must be `Str`/`borrow Str` |
| `MPT2035` | `json.encode<T>` value type mismatch |

#### HIR verify (`MPHIR*`)

| Code | Description |
|---|---|
| `MPHIR01` | `getfield` object must be borrow/mutborrow |
| `MPHIR02` | `setfield` object must be mutborrow |
| `MPHIR03` | Borrow escapes via return |

#### Ownership (`MPO*`)

| Code | Description |
|---|---|
| `MPO0003` | Borrow escapes scope |
| `MPO0004` | Wrong ownership mode for mut/read op |
| `MPO0007` | Use after move |
| `MPO0011` | Move while borrowed |
| `MPO0101` | Borrow crosses block boundary |
| `MPO0102` | Borrow in phi |
| `MPO0103` | `map.get` requires Dupable V |
| `MPO0201` | Spawn/send capture rule violation |

#### MPIR / Link / Budget (`MPM*`, `MPLINK*`, `MPL*`)

| Code | Description |
|---|---|
| `MPM0001` | MPIR lowering produced no modules |
| `MPLINK01` | Primary link path failed |
| `MPLINK02` | Fallback link unavailable |
| `MPL0001` | Unknown emit kind |
| `MPL0002` | Requested artifact missing |
| `MPL0801` | LLM budget too small |
| `MPL0802` | Tokenizer fallback |

#### Lint (`MPL2*`)

| Code | Description |
|---|---|
| `MPL2001` | Oversized function body |
| `MPL2002` | Unused/dead symbol |
| `MPL2003` | Unnecessary borrow |
| `MPL2005` | Empty block |
| `MPL2007` | Unreachable code |
| `MPL2020` | Monomorphization pressure too high |
| `MPL2021` | Mixed generics mode conflict |

#### GPU (`MPG*`)

| Code | Description |
|---|---|
| `MPG_CORE_1100` | `new` heap allocation forbidden in kernel |
| `MPG_CORE_1101` | `gpu.TBuffer` (Arc) forbidden in kernel |
| `MPG_CORE_1102` | Dynamic dispatch / `TCallable` forbidden |
| `MPG_CORE_1103` | Recursion forbidden in kernel |
| `MPG_CORE_1104` | `Str` type forbidden in kernel |
| `MPG_CORE_1105` | `Array<T>` forbidden in kernel |
| `MPG_CORE_1106` | `Map<K,V>` forbidden in kernel |
| `MPG_CORE_1107` | `TCallable` parameter forbidden in kernel |
| `MPG_CORE_1200` | `unsafe gpu fn` missing `requires(...)` |
| `MPG_CORE_1201` | Invalid capability in `requires(...)` |
| `MPG_CORE_1301` | PTX/HIP toolchain not found |
| `MPG_TYP_*` | GPU type errors (e.g. bf16 unsupported on backend) |
| `MPG_KRN_*` | Kernel validation errors |
| `MPG_BUF_*` | GPU buffer errors |
| `MPG_SYN_*` | GPU sync errors (barrier misuse) |
| `MPG_CAP_*` | GPU capability errors |
| `MPG_LNK_*` | GPU link errors (llc/lld not found) |
| `MPG_PRF_*` | GPU profiling errors |

### 9.3 Fix playbooks (most common errors)

#### MPHIR01 -- getfield object must be borrow/mutborrow

**Bad:**
```mp
%p: TPoint = new TPoint { x=const.i64 1, y=const.i64 2 }
%x: i64 = getfield { obj=%p, field=x }
```

**Fix:**
```mp
%p: TPoint = new TPoint { x=const.i64 1, y=const.i64 2 }
%pb: borrow TPoint = borrow.shared { v=%p }
%x: i64 = getfield { obj=%pb, field=x }
```

#### MPHIR02 -- setfield object must be mutborrow

**Bad:**
```mp
%pb: borrow TPoint = borrow.shared { v=%p }
setfield { obj=%pb, field=x, val=const.i64 3 }
```

**Fix:**
```mp
%pm: mutborrow TPoint = borrow.mut { v=%p }
setfield { obj=%pm, field=x, val=const.i64 3 }
```

#### MPHIR03 -- borrow escapes via return

**Bad:**
```mp
fn @leak(%p: TPoint) -> borrow TPoint {
bb0:
  %pb: borrow TPoint = borrow.shared { v=%p }
  ret %pb
}
```

**Fix** (return the field value instead):
```mp
fn @ok(%p: TPoint) -> i64 {
bb0:
  %pb: borrow TPoint = borrow.shared { v=%p }
  %x: i64 = getfield { obj=%pb, field=x }
  ret %x
}
```

#### MPO0007 -- use after move

**Bad:**
```mp
%sp: shared TPoint = share { v=%p }
%pb: borrow TPoint = borrow.shared { v=%p }  ; ERROR: %p consumed by share
```

**Fix:**
```mp
%sp: shared TPoint = share { v=%p }
%cp: shared TPoint = clone.shared { v=%sp }
; use %sp or %cp, not %p
```

#### MPO0011 -- move while borrowed

**Bad:**
```mp
%pb: borrow TPoint = borrow.shared { v=%p }
%sp: shared TPoint = share { v=%p }   ; ERROR: %p still borrowed
```

**Fix** (finish borrow, branch, then move):
```mp
bb0:
  %pb: borrow TPoint = borrow.shared { v=%p }
  %x: i64 = getfield { obj=%pb, field=x }
  br bb1
bb1:
  %sp: shared TPoint = share { v=%p }  ; OK: borrow ended at bb0 exit
```

#### MPO0101 -- borrow crosses block boundary

**Bad:**
```mp
bb0:
  %pb: borrow TPoint = borrow.shared { v=%p }
  br bb1
bb1:
  %x: i64 = getfield { obj=%pb, field=x }   ; ERROR: %pb from bb0
```

**Fix:**
```mp
bb0:
  br bb1
bb1:
  %pb: borrow TPoint = borrow.shared { v=%p }
  %x: i64 = getfield { obj=%pb, field=x }   ; OK: %pb defined in bb1
```

#### MPO0102 -- borrow in phi

**Bad:**
```mp
%pb: borrow TPoint = phi borrow TPoint { [bb1:%p1b], [bb2:%p2b] }
```

**Fix** (phi the owned value, then borrow locally):
```mp
%p: TPoint = phi TPoint { [bb1:%p1], [bb2:%p2] }
%pb: borrow TPoint = borrow.shared { v=%p }
```

#### MPT2014 -- numeric operands type mismatch

**Bad:**
```mp
%r: i64 = i.add { lhs=const.i64 1, rhs=const.i32 2 }
```

**Fix:**
```mp
%r: i64 = i.add { lhs=const.i64 1, rhs=const.i64 2 }
```

#### MPT0002 -- shared/weak invalid on TOption

**Bad:**
```mp
%x: shared TOption<i64> = enum.new<None> { }
```

**Fix:**
```mp
%x: TOption<i64> = enum.new<None> { }
```

#### MPT1023 -- missing required trait impl

**Bad** (`Map<TKey, V>` without hash/eq):
```mp
%m: Map<TKey, i64> = map.new<TKey, i64> { }
```

**Fix:**
```mp
fn @hash_key(%k: borrow TKey) -> u64 { bb0: ret const.u64 0 }
impl hash for TKey = @hash_key

fn @eq_key(%a: borrow TKey, %b: borrow TKey) -> bool { bb0: ret const.bool true }
impl eq for TKey = @eq_key

%m: Map<TKey, i64> = map.new<TKey, i64> { }
```

#### MPT1005 -- value struct contains heap handle

**Bad:**
```mp
value struct TBad { field s: Str }
```

**Fix:**
```mp
heap struct TGood { field s: Str }
```

#### MPS0024 -- ptr.* outside unsafe context

**Bad:**
```mp
%p: rawptr<i64> = ptr.null<i64>
```

**Fix:**
```mp
unsafe {
  %p: rawptr<i64> = ptr.null<i64>
}
```

#### MPS0003 -- dominance violation

**Bad** (`%x` only defined in bb1, used in bb3 which also has bb2 as predecessor):
```mp
bb0:
  cbr %c bb1 bb2
bb1:
  %x: i64 = const.i64 1
  br bb3
bb2:
  br bb3
bb3:
  %y: i64 = i.add { lhs=%x, rhs=const.i64 1 }   ; ERROR
```

**Fix** (use phi):
```mp
bb0:
  cbr %c bb1 bb2
bb1:
  %x1: i64 = const.i64 1
  br bb3
bb2:
  %x2: i64 = const.i64 2
  br bb3
bb3:
  %x: i64 = phi i64 { [bb1:%x1], [bb2:%x2] }
  %y: i64 = i.add { lhs=%x, rhs=const.i64 1 }
```

#### MPP0002 -- syntax error (missing comma)

**Bad:**
```mp
%x: i64 = i.add { lhs=const.i64 1 rhs=const.i64 2 }
```

**Fix:**
```mp
%x: i64 = i.add { lhs=const.i64 1, rhs=const.i64 2 }
```

---

## 10) CLI Reference

### 10.1 Global flags (place BEFORE the subcommand)

```bash
magpie [global-flags] <subcommand> [subcommand-args]
```

| Flag | Description | Default |
|---|---|---|
| `--entry <path>` | Entry source file | From `Magpie.toml` |
| `--emit <kinds>` | Artifact types (comma-separated) | — |
| `--output <text\|json\|jsonl>` | Output format | `text` |
| `--color <auto\|always\|never>` | Color mode | `auto` |
| `--log-level <level>` | `error\|warn\|info\|debug\|trace` | `warn` |
| `--profile <dev\|release>` | Build profile | `dev` |
| `--target <triple>` | Target triple | Host |
| `-j, --jobs <n>` | Parallel jobs | — |
| `--max-errors <n>` | Max errors per pass | `20` |
| `--llm` | LLM-optimized output (implies `--output json`) | — |
| `--llm-token-budget <n>` | Token budget for LLM output | — |
| `--shared-generics` | Use vtable-based shared generics | — |
| `--no-auto-fmt` | Disable auto-format in LLM mode | — |

### 10.2 Subcommands

| Command | Description | Example |
|---|---|---|
| `build` | Compile the project | `magpie --entry src/main.mp --emit exe build` |
| `run` | Build and execute | `magpie --entry src/main.mp run -- arg1 arg2` |
| `parse` | Parse and emit AST | `magpie --entry src/main.mp parse` |
| `fmt` | Format source (CSNF) | `magpie fmt` |
| `lint` | Run linter | `magpie --entry src/main.mp lint` |
| `test` | Run tests | `magpie test --filter pattern` |
| `explain <CODE>` | Explain a diagnostic code | `magpie explain MPO0007` |
| `new <name>` | Create a new project | `magpie new my_project` |
| `doc` | Generate documentation | `magpie doc` |
| `repl` | Start REPL | `magpie repl` |
| `mpir verify` | Verify MPIR correctness | `magpie --entry src/main.mp mpir verify` |
| `graph symbols` | Symbol graph | `magpie --entry src/main.mp graph symbols` |
| `graph deps` | Dependency graph | `magpie --entry src/main.mp graph deps` |
| `graph ownership` | Ownership graph | `magpie --entry src/main.mp graph ownership` |
| `graph cfg` | Control flow graph | `magpie --entry src/main.mp graph cfg` |
| `ffi import` | Import C headers | `magpie ffi import --header foo.h --out ffi.mp` |

### 10.3 Emit kinds

| Kind | Description |
|---|---|
| `exe` | Native executable |
| `llvm-ir` | LLVM IR text |
| `llvm-bc` | LLVM bitcode |
| `object` | Object file |
| `asm` | Assembly |
| `shared-lib` | Shared library |
| `mpir` | Magpie IR |
| `mpd` | MPD debug info |
| `mpdbg` | Debug info |
| `ast` | AST dump |
| `spv` | SPIR-V (Vulkan) |
| `msl` | Metal Shading Language |
| `ptx` | PTX (NVIDIA) |
| `hip` | HIP (AMD) |
| `wgsl` | WGSL (WebGPU) |
| `symgraph` | Symbol graph |
| `depsgraph` | Dependency graph |
| `ownershipgraph` | Ownership graph |
| `cfggraph` | Control flow graph |

### 10.4 Common workflows

```bash
; Build and check for errors (machine-readable)
magpie --entry src/main.mp --output json build

; Build with multiple artifacts
magpie --entry src/main.mp --emit exe,llvm-ir,mpir build

; Run the program
magpie --entry src/main.mp run

; Get help for a specific error
magpie explain MPO0101

; Format the source
magpie --entry src/main.mp fmt

; Build GPU program for Metal
magpie --entry src/main.mp --emit exe,msl build
```

---

## 11) Manifest Format (Magpie.toml)

```toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2024"

[build]
entry = "src/main.mp"
profile_default = "dev"

[dependencies]
some_lib = "1.2.3"
util = { path = "../util", version = "1.2.3" }

[gpu]
enabled = true
backend = "metal"
device_index = 0

[llm]
mode_default = false
token_budget = 32000
```

---

## 12) Common Pitfalls (Top 15)

1. **Wrong const suffix** -- `const.i32 0` for `i32`, `const.i64 0` for `i64`. They must match exactly.

2. **Comment syntax** -- use `;` for line comments, `;;` for doc comments. Not `//` or `#`.

3. **Borrow crossing blocks** -- borrows are block-scoped. Re-borrow in each new block.

4. **`getfield` without borrow** -- you must `borrow.shared` or `borrow.mut` before field access.

5. **`setfield` uses `val=`** -- not `value=`. Write `setfield { obj=%m, field=f, val=%x }`.

6. **Missing comma in ops** -- `{ lhs=x rhs=y }` is wrong; use `{ lhs=x, rhs=y }`.

7. **Header order** -- must be exactly: `module`, `exports`, `imports`, `digest`.

8. **Using value after move** -- `share { v=%x }` consumes `%x`. Don't use `%x` after.

9. **Borrow in phi** -- `phi borrow T { ... }` is illegal. Phi the owned value, then borrow.

10. **Map<TUserStruct, V> without impls** -- user types need explicit `impl hash` and `impl eq`. (`Str` doesn't.)

11. **`gpu.launch` key order** -- must be exactly `device, kernel, grid, block, args`.

12. **Value struct with heap field** -- `value struct` cannot contain `Str`, `Array`, etc. Use `heap struct`.

13. **Returning a borrow** -- borrows cannot be function return types. Return the value instead.

14. **`TOption` variant names** -- use `Some`/`None` for `TOption`, `Ok`/`Err` for `TResult`. Don't mix them.

15. **GPU kernel using heap types** -- GPU kernels cannot use `Str`, `Array`, `Map`, or `TCallable`. Use `rawptr<T>` and primitives.

---

## 13) Checklist Before Declaring Done

- [ ] Program has correct header order: `module`, `exports`, `imports`, `digest`
- [ ] Every block ends with exactly one terminator (`ret`, `br`, `cbr`, `switch`, `unreachable`)
- [ ] Every `const` suffix matches the declared SSA type
- [ ] Every `getfield` uses a borrow/mutborrow receiver
- [ ] Every `setfield` uses a mutborrow receiver with `val=` key
- [ ] No borrow crosses a block boundary
- [ ] No borrow appears in a phi node
- [ ] No borrow is returned from a function
- [ ] No value is used after being consumed/moved
- [ ] Collection ops use correct receiver mode (mutborrow for mutation, borrow for reads)
- [ ] Map key types have `hash` + `eq` impls (unless using `Str`)
- [ ] GPU kernels use only primitives and `rawptr<T>` -- no heap types
- [ ] `magpie --entry <file> --output json build` produces zero errors
- [ ] `magpie explain <CODE>` consulted for any unfamiliar error

---
> Source: [magpie-lang/magpie](https://github.com/magpie-lang/magpie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
