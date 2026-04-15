## cursorrules

> Effective Rust rules derived from the book to guide Cursor's AI when working with Rust code.

# Effective Rust Cursor Rule
These guidelines distill key recommendations from *Effective Rust* (2024) and are intended for the Cursor Agent.
Follow them when creating, modifying, or reviewing Rust code.
### 1. Prefer immutable references
```rust
fn add_one(x: &i32) -> i32 { x + 1 }
```
Immutable references (`&T`) allow multiple concurrent readers and reduce the mental burden of tracking mutations.
### 2. Use explicit lifetime annotations only when necessary
```rust
fn longest<'a>(a: &'a str, b: &'a str) -> &'a str { if a.len() > b.len() { a } else { b } }
```
Let the compiler elide lifetimes; add them only to disambiguate.
### 3. Return owned data to avoid borrowed outs
```rust
fn collect_names(src: &[&str]) -> Vec<String> { src.iter().map(|s| s.to_string()).collect() }
```
Returning owned values prevents lifetimes from leaking into callers.
### 4. Prefer `Cow` for conditional ownership
```rust
use std::borrow::Cow;
fn maybe_upper<'a>(s: &'a str) -> Cow<'a, str> { if s.chars().any(|c| c.is_uppercase()) { Cow::Owned(s.to_uppercase()) } else { Cow::Borrowed(s) } }
```
`Cow` minimizes allocations while giving callers owned access when needed.
### 5. Avoid multiple levels of indirection
```rust
struct Node { next: Option<Box<Node>> }
```
Prefer `Box<T>` over `&Box<T>` and similar nested pointers unless required.
### 6. Use `Option<&T>` instead of `&Option<T>`
```rust
match maybe_ref { Some(v) => do_something(v), None => fallback() }
```
`Option<&T>` models optional borrowing directly and avoids double matching.
### 7. Leverage `AsRef` and `Borrow` traits
```rust
fn read_path<P: AsRef<Path>>(p: P) { let path = p.as_ref(); /* ... */ }
```
These traits accept both owned and borrowed forms seamlessly.
### 8. Clone sparingly and explicitly
```rust
let data = expensive.clone(); // Only when needed
```
Make ownership transfers obvious; prefer borrowing when possible.
### 9. Use `Rc` for shared ownership in single‑threaded code
```rust
let rc = Rc::new(data);
```
Avoid `Arc` unless threads are involved; `Rc` is lighter.
### 10. Dereference early in method chains
```rust
let len = (&name).len();
```
Reducing `&` noise clarifies intent.
### 11. Propagate errors with `?`
```rust
let config = read_config()?;
```
`?` is concise and preserves original error context.
### 12. Define custom error enums
```rust
#[derive(thiserror::Error, Debug)]
pub enum MyError { #[error("io")] Io(#[from] std::io::Error) }
```
Custom enums provide rich typing over `Box<dyn Error>`.
### 13. Use `anyhow` for prototypes, `thiserror` for libraries
```rust
anyhow::Result<T> vs Result<T, MyError>
```
Balance convenience and clarity.
### 14. Attach context with `with_context`
```rust
let data = fs::read(path).with_context(|| format!("reading {path:?}"))?;
```
Extra context simplifies debugging.
### 15. Avoid panics in library code
```rust
return Err(MyError::InvalidInput(arg.to_string()));
```
Let callers decide how to handle failures.
### 16. Prefer `Result<Option<T>>` over sentinel values
```rust
Ok(None) signals absence without throwing errors
```
This distinguishes missing data from exceptional cases.
### 17. Map errors to public API types
```rust
inner_call().map_err(MyError::from)
```
Hiding internal error shapes keeps API stable.
### 18. Use `unwrap_or_else` for defaults
```rust
let port = env::var("PORT").ok().and_then(|s| s.parse().ok()).unwrap_or_else(|| 8080);
```
Avoid `unwrap` in production paths.
### 19. Log and propagate, don’t log twice
```rust
error!(%err, "fetch failed") ; return Err(err);
```
Double‑logging clutters output.
### 20. Document error semantics
```rust
/// Returns `Err(DecodeError)` if payload is malformed.
```
Callers should know what to expect.
### 21. Use `Into`/`From` for simple conversions
```rust
impl From<u16> for Port { ... }
```
These traits enable ergonomic `into()` calls.
### 22. Prefer `impl Trait` in return position
```rust
fn iter() -> impl Iterator<Item=u8> { 0..10 }
```
It hides concrete types and reduces generics noise.
### 23. Expose trait bounds in where‑clauses
```rust
fn sort<T>(xs: &mut [T]) where T: Ord { xs.sort(); }
```
Where‑clauses improve readability for complex bounds.
### 24. Derive common traits
```rust
#[derive(Clone, Debug, PartialEq, Eq, Hash)]
```
Automatic derives reduce boilerplate and keep traits consistent.
### 25. Use blanket impls carefully
```rust
impl<T: Display> ToString for T { ... }
```
Overly broad impls can cause coherence conflicts.
### 26. Separate read and write traits
```rust
trait ReadStore { fn get(&self, id: Id) -> Option<Item>; }
trait WriteStore { fn put(&mut self, item: Item); }
```
Segregated interfaces follow the ISP and permit immutable references.
### 27. Prefer trait objects only for dynamic dispatch needs
```rust
fn draw(ui: &mut dyn Widget) { ui.render(); }
```
Static dispatch is zero‑cost; use `dyn` for runtime polymorphism.
### 28. Name generic parameters tersely but meaningfully
```rust
// T, U, or domain abbreviations like Msg
```
Consistent naming improves comprehension.
### 29. Choose `Arc<Mutex<T>>` over `Mutex<Arc<T>>`
```rust
let shared = Arc::new(Mutex::new(state));
```
Arc controls sharing; Mutex protects data.
### 30. Hold locks for the shortest time
```rust
{ let data = map.lock().unwrap(); process(&*data); }
```
Reduce contention and deadlock risk.
### 31. Prefer channels for message passing
```rust
let (tx, rx) = tokio::sync::mpsc::channel(100);
```
Channels decouple tasks and simplify ownership.
### 32. Use `tokio::select!` for cancellation
```rust
tokio::select! { _ = sigint.recv() => { ... } }
```
Reactive patterns keep async listeners responsive.
### 33. Avoid blocking in async contexts
```rust
spawn_blocking(|| compute());
```
Blocking inside async tasks stalls the executor.
### 34. Leverage structured concurrency
```rust
let task = tokio::spawn(async move { ... }); task.await?;
```
Tasks should be joined or cancelled together.
### 35. Prefer iterators to indexed loops
```rust
xs.iter().for_each(|x| println!("{}", x));
```
Iterators allow fusion and lazy evaluation.
### 36. Reserve capacity for `Vec` when size is known
```rust
let mut v = Vec::with_capacity(n);
```
Avoids reallocations.
### 37. Avoid unnecessary `clone()` in hot paths
```rust
for item in &items { process(item); }
```
Profiling often reveals excess clones.
### 38. Use `#[inline]` judiciously
```rust
#[inline] fn fast() { ... }
```
Inlining too much can bloat code size.
### 39. Benchmark with `cargo criterion`
```rust
criterion_group!(benches, my_bench);
```
Measure before optimizing.
### 40. Prefer small, `Copy` types for tight loops
```rust
struct Point { x: f32, y: f32 }
```
`Copy` avoids move overhead.
### 41. Limit `unsafe` to small blocks
```rust
unsafe { ptr::copy(src, dst, n) }
```
Small scopes make auditing easier.
### 42. Document invariants before unsafe blocks
```rust
// SAFETY: src and dst are non‑overlapping and valid for n bytes.
```
Clear invariants assist reviewers.
### 43. Prefer `unsafe fn` only when every call is unsafe
```rust
unsafe fn new_unchecked(ptr: *mut T) { ... }
```
Use `unsafe` blocks otherwise.
### 44. Use `std::ptr` utilities over raw pointer arithmetic
```rust
ptr::copy_nonoverlapping
```
Library functions embed safety checks.
### 45. Add tests exercising unsafe code
```rust
#[test] fn ptr_copy() { ... }
```
Catch subtle UB early.
### 46. Prefer expressive type names
```rust
pub struct TemperatureCelsius(f32);
```
Domain types prevent unit mix‑ups.
### 47. Return iterators instead of collections
```rust
pub fn iter(&self) -> impl Iterator<Item=&T>;
```
Callers choose how to collect.
### 48. Use builder pattern for combinatorial constructors
```rust
ConfigBuilder::default().port(8080).finish()
```
Builders clarify optional parameters.
### 49. Hide implementation details behind modules
```rust
mod imp { ... } pub use imp::Public;
```
Encapsulation enables refactors.
### 50. Document examples with `///` doc tests
```rust
/// ```
/// let v = vec![1,2];
/// assert_eq!(v.len(), 2);
/// ```
```
Executable docs ensure accuracy.
### 51. Favor property‑based tests for invariants
```rust
proptest! { ...(a in 0..100) => { ... } }
```
Randomized inputs uncover edge cases.
### 52. Use `#[should_panic]` for expected panics
```rust
#[test]
#[should_panic]
fn fails() { ... }
```
Documents panic contract.
### 53. Tag slow tests with `#[ignore]`
```rust
#[test]
#[ignore]
fn integration() { ... }
```
Keeps default test runs fast.
### 54. Group integration tests under `tests/`
```rust
tests/api.rs
```
Cargo runs them as separate crates.
### 55. Mock external services with `wiremock`
```rust
MockServer::start()
```
Isolate tests from network variability.
### 56. Use `mod.rs` only for legacy code
```rust
// Prefer foo/mod.rs or foo.rs, not both.
```
Flat file layout reduces nesting.
### 57. Re‑export smartly from `lib.rs`
```rust
pub use crate::sub::types::*;
```
Public API should be discoverable.
### 58. Group tests with `#[cfg(test)]`
```rust
mod tests { ... }
```
Test modules keep crate tidy.
### 59. Expose minimal surface area
```rust
pub(crate) struct Private;
```
Fewer publics, fewer guarantees.
### 60. Organize imports with `rustfmt`
```rust
use std::collections::HashMap;
```
Consistent style aids readers.
### 61. Use declarative macros before proc macros
```rust
macro_rules! vec_of_strings { ... }
```
Declarative macros compile faster and are simpler.
### 62. Export macros with `#[macro_export]` sparingly
```rust
// Place macros in crate root only when necessary.
```
Polluting global namespace hurts ergonomics.
### 63. Document macro syntax thoroughly
```rust
/// `json_object!(key => value, ...)`
```
Macros hide complexity; docs matter.
### 64. Avoid `tt` munchers; prefer `macro` metavariables
```rust
macro_rules! count { ($($x:tt)*) => { ... } }
```
Newer macro system is more robust.
### 65. Provide compile‑time assertions
```rust
static_assertions::const_assert!(SIZE == 64);
```
Catch errors early.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/gabornyergesX)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/gabornyergesX)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
