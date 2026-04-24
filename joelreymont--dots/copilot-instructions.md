## dots

> Fast CLI issue tracker in Zig with markdown storage.

# Dots - Agent Instructions

Fast CLI issue tracker in Zig with markdown storage.

## Communication

Always refer to the user as Mr. Picklesworth.

## Build

```bash
zig build -Doptimize=ReleaseSmall
strip zig-out/bin/dot
```

## Test

```bash
zig build test
```

## References

- [Zig 0.15 API](docs/zig-0.15-api.md) - Critical API changes for comptime, ArrayList, JSON, I/O

## Zig Guidelines

### Zig 0.15 Patterns
- See `docs/zig-0.15-api.md` for API reference
- ArrayList is unmanaged: `var list = std.ArrayList(T){};` + pass allocator to methods
- Alignment enum: `alignedAlloc(u8, .@"16", size)`

### Import Once, Reference via Namespace
```zig
// WRONG: Multiple imports from same module
const Type = @import("type.zig").Type;
const Primitive = @import("type.zig").Primitive;

// RIGHT: Import module once, use namespace
const types = @import("type.zig");
// Then use: types.Type, types.Primitive
```

### Allocator First
Allocator is ALWAYS the first argument to any function that allocates:
```zig
// RIGHT
pub fn init(allocator: std.mem.Allocator) Self { ... }

// WRONG
pub fn init(config: Config, allocator: std.mem.Allocator) Self { ... }
```

### ArrayList Batch Append
When adding multiple known items to an ArrayList, use a static array + appendSlice:
```zig
// WRONG: Append items one by one
try list.append(allocator, a);
try list.append(allocator, b);
try list.append(allocator, c);

// RIGHT: Create static array, appendSlice once
const items = [_]T{ a, b, c };
try list.appendSlice(allocator, &items);
```

### Avoid Allocation When Possible
- Use stack arrays for small fixed-size data
- Prefer slices over ArrayList when size is known
- Use comptime for constant data

### Static String Comparison - Use Comptime Maps
Instead of chaining `std.mem.eql` comparisons, use comptime string maps:

```zig
// WRONG - Linear chain of comparisons
fn parseStatus(s: []const u8) ?Status {
    if (std.mem.eql(u8, s, "open")) return .open;
    if (std.mem.eql(u8, s, "active")) return .active;
    if (std.mem.eql(u8, s, "closed")) return .closed;
    return null;
}

// RIGHT - Comptime static string map
const status_map = std.StaticStringMap(Status).initComptime(.{
    .{ "open", .open },
    .{ "active", .active },
    .{ "closed", .closed },
});

fn parseStatus(s: []const u8) ?Status {
    return status_map.get(s);
}
```

### JSON - Always Use Typed Structs
Never manipulate JSON as dynamic `std.json.Value`. Always define typed structs:

```zig
// WRONG - Dynamic JSON access
const parsed = try std.json.parseFromSlice(std.json.Value, allocator, input, .{});
const obj = parsed.value.object;
const name = obj.get("name").?.string;  // Runtime errors, no type safety

// RIGHT - Typed struct
const Config = struct {
    name: []const u8,
    count: i32 = 0,
    optional: ?[]const u8 = null,
};
const parsed = try std.json.parseFromSlice(Config, allocator, input, .{
    .ignore_unknown_fields = true,
});
const config = parsed.value;  // Type-safe access: config.name, config.count
```

## Binary Size Tracking

After rebuilding dots with `zig build -Doptimize=ReleaseSmall`, check the binary size:

```bash
ls -lh zig-out/bin/dot
```

Compare against README.md line 7 and line 275 which state the current size.

If the size differs significantly (plus or minus 50KB), update:
1. Line 7: `Minimal task tracker... (X.XMB vs 19MB)...`
2. Line 275: `| Binary | 19 MB | X.X MB | NNx smaller |`

Calculate the "Nx smaller" as `19 / size_in_mb` rounded to nearest integer.

Current documented size: 1.2MB
Current actual size: around 910KB (0.9MB) -> needs update to "21x smaller"

---
> Source: [joelreymont/dots](https://github.com/joelreymont/dots) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
