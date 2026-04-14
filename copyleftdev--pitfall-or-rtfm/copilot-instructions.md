## pitfall-or-rtfm

> **Keywords:** rust, performance, optimization, simd, rayon, memory, allocation

# Rust Optimization Rules

**Keywords:** rust, performance, optimization, simd, rayon, memory, allocation
**Flags:** `rust-specific`, `performance-critical`, `optimization-guide`

## Rust Performance Best Practices

<rust_optimization_principles>
- Prefer borrowing (`&T`) over cloning (`T.clone()`) whenever possible
- Use `String::with_capacity()` when the final size is known or estimable
- Leverage iterator chains instead of intermediate collections
- Use `Vec::with_capacity()` for pre-sized collections
- Prefer `&str` over `String` for read-only string operations
- Use `slice` operations instead of index-based loops when possible
</rust_optimization_principles>

## Memory Management Guidelines

<memory_optimization>
- Avoid unnecessary allocations in hot paths
- Reuse buffers and collections when possible
- Use `std::mem::take()` to move values without cloning
- Prefer stack allocation over heap allocation for small, fixed-size data
- Use `Box<[T]>` instead of `Vec<T>` for immutable, sized collections
- Consider `Cow<str>` for conditional string ownership
</memory_optimization>

## I/O Performance Rules

<io_optimization>
- Always use `BufReader` and `BufWriter` for file operations
- Set appropriate buffer sizes (64KB is often optimal)
- Use `read_to_string()` with pre-allocated capacity when possible
- Prefer streaming operations over loading entire files into memory
- Use memory-mapped files (`memmap2`) for large read-only files
- Batch I/O operations to reduce system call overhead
</io_optimization>

## Parallel Processing Guidelines

<parallelization_rules>
- Use Rayon's `par_iter()` for CPU-bound operations on collections
- Ensure work per thread is substantial enough to overcome overhead
- Use `rayon::scope()` for custom parallel algorithms
- Consider `std::thread::available_parallelism()` for thread count
- Avoid parallelizing I/O-bound operations unless using async
- Use `crossbeam` channels for producer-consumer patterns
</parallelization_rules>

## SIMD Optimization Rules

<simd_guidelines>
- Use `std::simd` for stable SIMD operations (Rust 1.72+)
- Fall back to `wide` crate for portable SIMD
- Ensure data alignment for optimal SIMD performance
- Process data in chunks that match SIMD register width
- Use `slice::chunks_exact()` for SIMD-friendly iteration
- Handle remainder elements separately after SIMD processing
</simd_guidelines>

## Compiler Optimization Settings

<compiler_optimization>
- Use `opt-level = 3` for maximum optimization
- Enable `lto = "fat"` for link-time optimization
- Set `codegen-units = 1` for better optimization opportunities
- Use `panic = "abort"` to reduce binary size and improve performance
- Consider `target-cpu = "native"` for machine-specific optimizations
- Profile with `cargo flamegraph` to identify bottlenecks
</compiler_optimization>

## Benchmarking Best Practices

<benchmarking_rules>
- Always use `criterion::black_box()` to prevent dead code elimination
- Include warmup iterations to account for CPU frequency scaling
- Run benchmarks on isolated CPU cores (`taskset`)
- Disable CPU frequency scaling during benchmarks
- Use consistent input data across benchmark runs
- Measure both throughput and latency metrics
- Include memory allocation profiling with `heaptrack`
</benchmarking_rules>

## Anti-Patterns to Avoid

<rust_antipatterns>
- Don't use `collect()` unnecessarily in iterator chains
- Avoid `clone()` when `&` or `&mut` suffices
- Don't use `String` concatenation with `+` in loops
- Avoid `Vec::push()` in tight loops without pre-allocation
- Don't use `unwrap()` in performance-critical paths
- Avoid recursive algorithms for large datasets (stack overflow risk)
- Don't ignore compiler warnings about unused allocations
</rust_antipatterns>

## Profiling and Analysis

<profiling_guidelines>
- Use `cargo flamegraph` for CPU profiling
- Use `heaptrack` or `valgrind massif` for memory profiling
- Use `perf stat` for hardware performance counters
- Profile both debug and release builds to understand differences
- Use `cargo expand` to see macro expansions that might affect performance
- Monitor cache hit rates and branch prediction accuracy
</profiling_guidelines>

## Error Handling Performance

<error_handling_optimization>
- Use `Result<T, E>` efficiently - avoid unnecessary `unwrap()`/`expect()`
- Consider `Option<T>` over `Result<T, ()>` for simple cases
- Use `?` operator for early returns in fallible functions
- Prefer `match` over multiple `if let` for performance-critical paths
- Consider custom error types that implement `std::error::Error`
- Use `anyhow` for application errors, `thiserror` for library errors
</error_handling_optimization>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/copyleftdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
