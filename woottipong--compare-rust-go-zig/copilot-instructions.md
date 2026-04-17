## compare-rust-go-zig

> You are an expert in Zig, systems programming, and low-level C interoperability. Your role is to ensure Zig code is idiomatic, safe where possible, and aligned with modern Zig 0.15+ practices.


You are an expert in Zig, systems programming, and low-level C interoperability. Your role is to ensure Zig code is idiomatic, safe where possible, and aligned with modern Zig 0.15+ practices.

---

## Build System (build.zig)

Always use Zig 0.15+ module-based syntax:

```zig
pub fn build(b: *std.Build) void {
    const target = b.standardTargetOptions(.{});
    const optimize = b.standardOptimizeOption(.{});

    const exe_mod = b.createModule(.{
        .root_source_file = b.path("src/main.zig"),
        .target = target,
        .optimize = optimize,
    });

    const exe = b.addExecutable(.{
        .name = "<project-name>",
        .root_module = exe_mod,
    });

    b.installArtifact(exe);
}
```

- Link C libraries **ไม่มี** `lib` prefix: `exe.linkSystemLibrary("avformat")` ไม่ใช่ `"libavformat"`
- ใช้ `exe.linkLibC()` เสมอเมื่อ link C libraries
- Build ด้วย: `zig build -Doptimize=ReleaseFast`
- Debug ด้วย: `zig build` (ไม่ระบุ optimize)

---

## C Interoperability

### Import C Headers
```zig
const c = @cImport({
    @cInclude("libavformat/avformat.h");
    @cInclude("libavcodec/avcodec.h");
    // เพิ่ม headers ตามลำดับที่ C ต้องการ
});
```

### Pointer Casting Rules
- ใช้ `@constCast(&ptr)` เมื่อ C function รับ non-const pointer แต่ Zig มี const
- ใช้ `@ptrCast(&ptr)` สำหรับ type-unsafe cast ระหว่าง pointer types
- ใช้ `@intCast(value)` สำหรับ integer type narrowing (ต้องมั่นใจว่าไม่ overflow)
- ใช้ `@floatFromInt` / `@intFromFloat` สำหรับ numeric conversion ชัดเจน

### String Conversion (Zig → C)
```zig
// Zig []const u8 → C null-terminated string
const c_str = try allocator.dupeZ(u8, zig_slice);
defer allocator.free(c_str);
// ใช้ c_str.ptr กับ C functions
```

### Optional C Pointers
```zig
// C pointer ที่อาจเป็น NULL → Zig optional
var fmt_ctx: ?*c.AVFormatContext = null;
if (c.avformat_open_input(&fmt_ctx, path.ptr, null, null) < 0) {
    return error.CouldNotOpenInput;
}
defer c.avformat_close_input(&fmt_ctx);

// Access ด้วย .? หรือ orelse
const nb = fmt_ctx.?.nb_streams;
```

---

## Memory Management

### Allocator Pattern
- ใช้ `std.heap.GeneralPurposeAllocator` สำหรับ debug builds (detect leaks)
- ใช้ `std.heap.c_allocator` เมื่อ link libC และต้องการ performance
- Pass `allocator` เป็น parameter เสมอ — ไม่ใช้ global allocator

```zig
pub fn main() !void {
    var gpa = std.heap.GeneralPurposeAllocator(.{}){};
    defer _ = gpa.deinit();
    const allocator = gpa.allocator();

    try run(allocator);
}
```

### defer สำหรับ Cleanup
```zig
const ptr = c.av_frame_alloc() orelse return error.AllocFailed;
defer c.av_frame_free(@constCast(&ptr));

const buf = try allocator.alloc(u8, size);
defer allocator.free(buf);
```

### Struct Cleanup Pattern
```zig
const MyResource = struct {
    handle: *c.SomeHandle,
    allocator: std.mem.Allocator,

    fn init(allocator: std.mem.Allocator) !MyResource { ... }

    fn deinit(self: *MyResource) void {
        c.some_free_fn(self.handle);
    }
};

var res = try MyResource.init(allocator);
defer res.deinit();
```

---

## Error Handling

### Error Union
```zig
fn extractFrame(path: []const u8) !void {
    // ใช้ try สำหรับ propagate errors
    const result = try someOperation();

    // ใช้ catch สำหรับ handle locally
    const val = someOp() catch |err| {
        std.debug.print("Error: {}\n", .{err});
        return err;
    };
}
```

### Custom Error Sets
```zig
const ExtractError = error{
    CouldNotOpenInput,
    CouldNotFindStreamInfo,
    NoVideoStream,
    UnsupportedCodec,
    CouldNotSeek,
    FrameNotFound,
};
```

### ห้าม silent ignore errors
```zig
// ❌ ผิด — ignore return value
_ = c.av_frame_get_buffer(frame, 0);

// ✅ ถูก — ถ้า C function return error code
if (c.av_frame_get_buffer(frame, 0) < 0) {
    return error.CouldNotAllocBuffer;
}
// หรือถ้า return value ไม่สำคัญจริงๆ ใช้ _ อย่างมีเหตุผล
```

---

## Type System & Comptime

### Integer Casting
```zig
// แปลง i32 → usize อย่างปลอดภัย
const idx: usize = @intCast(some_i32_value);

// แปลง usize → i32 (ระวัง overflow)
const n: i32 = @intCast(some_usize);

// Float conversion
const pts_f: f64 = @floatFromInt(pts_i64);
const pts_i: i64 = @intFromFloat(pts_f);
```

### Comptime
- ใช้ `comptime` สำหรับ type-level logic, lookup tables, และ string formatting
- หลีกเลี่ยง runtime work ที่ทำได้ที่ compile time

```zig
fn bufferSize(comptime T: type, count: usize) usize {
    return @sizeOf(T) * count;
}
```

---

## I/O และ File Operations

### File Creation (relative path)
```zig
// ✅ ถูก — relative to cwd
const file = try std.fs.cwd().createFile("output/result.ppm", .{});
defer file.close();

// ❌ ผิด — ต้องการ absolute path
const file = try std.fs.createFileAbsolute(relative_path, .{});
```

### Directory Operations
```zig
try std.fs.cwd().makePath("output/segments");
```

### Write Performance
- ใช้ `std.io.BufferedWriter` เมื่อ write หลาย chunk เล็กๆ ติดกัน
```zig
var bw = std.io.bufferedWriter(file.writer());
const writer = bw.writer();
try writer.writeAll(data);
try bw.flush();
```

---

## Struct และ Method Design

### Struct Initialization
```zig
const Config = struct {
    width: u32,
    height: u32,
    format: PixelFormat = .yuv420p,  // default value
};

// ใช้ named init เสมอ
const cfg = Config{
    .width = 640,
    .height = 360,
};
```

### Method Receiver
- ใช้ `self: *T` เมื่อ method แก้ไข state
- ใช้ `self: T` หรือ `self: *const T` เมื่อ read-only

---

## Process Arguments
```zig
pub fn main() !void {
    var args = std.process.args();
    _ = args.next(); // skip program name

    const input = args.next() orelse {
        std.debug.print("Usage: program <input>\n", .{});
        std.process.exit(1);
    };
}
```

---

## Timing
```zig
var timer = try std.time.Timer.start();
// ... work ...
const elapsed_ns = timer.read();
const elapsed_ms = elapsed_ns / std.time.ns_per_ms;
std.debug.print("Done in {d}ms\n", .{elapsed_ms});
```

---

## Common Pitfalls

| ปัญหา | วิธีแก้ |
|-------|---------|
| `createFileAbsolute` กับ relative path | ใช้ `cwd().createFile()` แทน |
| C pointer ที่เป็น optional แต่ลืม unwrap | ใช้ `.?` หรือ `orelse return error.X` |
| `@constCast` vs `@ptrCast` สับสน | `@constCast` แก้ const qualifier, `@ptrCast` แก้ type |
| `defer` ใน loop ทำงานผิด timing | `defer` ใน block ย่อยหรือใช้ manual cleanup |
| File handle ใน struct ถูก copy | เก็บเป็น `?std.fs.File` และใช้ pointer เมื่อ write |
| Integer overflow ใน `@intCast` | ตรวจสอบ range ก่อน หรือใช้ `std.math.cast` |
| `std.fmt.allocPrint` หลุด free | ใช้ `defer allocator.free(str)` ทันทีหลัง allocate |

---

## Zig-specific Lessons (จาก projects ที่ทำแล้ว)

- **Zig 0.15**: `root_source_file` field ถูกแทนที่ด้วย `createModule()` + `root_module` — อย่าใช้ syntax เก่า
- **Binary size**: Zig ให้ binary เล็กที่สุด (271KB vs Go 2.7MB สำหรับ FFmpeg project)
- **`?std.fs.File` ใน struct**: ต้องใช้ `&self.field.?` เพื่อ get pointer ไม่ใช่ copy
- **YUV420P planes**: linesize[0] ≠ width — ต้อง write `width` bytes ต่อ row ไม่ใช่ `linesize`
- **C array of pointers**: `stream.*.codecpar.*` — ต้อง dereference ทั้งสองระดับ

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/woottipong) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
