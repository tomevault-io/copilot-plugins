## led

> A terminal text editor written in Rust using a custom single-threaded push-based FRP framework (`crates/core/src/rx.rs`).

# led2 Editor

A terminal text editor written in Rust using a custom single-threaded push-based FRP framework (`crates/core/src/rx.rs`).

## FRP Architecture Principles

The architecture follows the CycleJS/froop pattern. The core cycle is:

```
Driver inputs -> model() -> fold/reduce -> state -> derived() -> Driver outputs -> cycle back
```

State flows in a single direction. An imitator (`Stream::new()` fed by `fold_into`) allows the state stream to be referenced before it is defined, closing the cycle. Drivers are the boundary where side effects happen — the model and derived layers are pure.

### The Golden Rule: All Logic in Combinators, Reducer Just Assigns

This is the single most important principle. In well-structured FRP code:

1. **Combinator chains** (map, filter, filter_map, sample_combine, dedupe, merge, etc.) contain all decision-making, transformation, and business logic.
2. **The reducer** (fold/fold_into) receives pre-computed values and mechanically assigns them to state fields. No conditionals, no loops, no function calls with logic.
3. **Each stream** produces exactly the `Mut` variant needed — the "what to update" decision is made upstream.

This is how it works in practice in well-structured FRP apps:

```
// Each concern is a small combinator chain that produces an update:
let audio_route_update = merge(
    state.filter_map(|s| s.recorder_state.map(|r| r.audio_route)).dedupe(),
    intent.audio_route_in
).map(StateUpdate::AudioRoute);

let want_camera_update = merge(intent.want_camera, auto_camera_on, auto_camera_off)
    .map(StateUpdate::WantCamera);

// All updates merged:
let state_update = merge(audio_route_update, want_camera_update, /* 40+ more */);

// Reducer: TRIVIAL fold. No logic whatsoever.
let real_state = init.map(|first_state|
    state_update.fold(first_state, |state, update| update.apply_to(state))
).flatten();

// Where apply_to is mechanical:
fn apply_to(self, mut state: AppState) -> AppState {
    match self {
        StateUpdate::AudioRoute(v) => { state.audio_route = v; }
        StateUpdate::WantCamera(v) => { state.want_camera = v; }
        // Each case: one assignment. That's it.
    }
    state
}
```

When a single signal needs to change multiple state fields, it splits into multiple streams — one per field — that each emit a separate update. For example, a "task next" signal:

```
let task_next_step = task_move
    .filter(|m| m.is_next)
    .sample_combine(&state)
    .filter_map(|(_, s)| s.next_task_state())
    .map(StateUpdate::SetGuidedTaskState);

let task_next_visited = task_move
    .filter(|m| m.is_next)
    .map(|_| StateUpdate::SetHasVisitedBrowser(false));

let task_next = merge(task_next_step, task_next_visited);
```

Not one `Mut::TaskNext` with logic in the reducer — two separate streams, each producing a single-field update.

### Principle 1: The Reducer Must Be Trivial — "Just Assign"

Every `Mut` variant handler in the `fold_into` must be 1-3 lines of direct field assignment. No `if`, no loops, no function calls containing logic. All decision-making lives upstream in the combinator chains that produce the `Mut`.

**Bad:**
```rust
Mut::ResumeComplete => {
    s.phase = Phase::Running;
    if let Some(order) = s.session.active_tab_order.take() {
        let non_preview_tabs: Vec<_> = s.tabs.iter().filter(|t| !t.is_preview()).collect();
        if let Some(tab) = non_preview_tabs.get(order) {
            s.active_tab = Some(tab.path().clone());
        }
    }
    // ... 25 more lines
}
```

**Good:**
```rust
Mut::SetPhase(phase) => { s.phase = phase; }
Mut::SetActiveTab(path) => { s.active_tab = Some(path); }
Mut::SetFocus(focus) => { s.focus = focus; }
```

### Principle 2: One Signal, Many Muts — Split via Multiple Streams

When a signal needs to change multiple parts of state, split it into multiple streams that each emit a fine-grained `Mut`. Don't bundle logic into one fat `Mut` variant.

**Bad:** `Mut::SessionRestored` carries 10 fields and the reducer has 40 lines handling it.

**Good:**
```rust
let session_phase_s = session_restored_s
    .map(|sr| if sr.pending_opens.is_empty() {
        Mut::SetPhase(Phase::Running)
    } else {
        Mut::SetPhase(Phase::Resuming)
    });

let session_tabs_s = session_restored_s
    .flat_map(|sr| sr.pending_opens.iter().map(|p| Mut::AddTab(p.clone())));

let session_browser_s = session_restored_s
    .map(|sr| Mut::SetBrowserState {
        selected: sr.browser_selected,
        scroll: sr.browser_scroll_offset,
    });
```

**The pattern: Common parent stream → multiple child chains.** When refactoring a fat reducer handler, first create a stream for the triggering event, then branch it into children that each produce one fine-grained `Mut`:

```rust
// Common parent: computed once, shared by all children
let resume_complete_s = state
    .filter(|s| s.phase == Phase::Resuming)
    .filter(|s| s.session.resume.iter().all(|e| e.state != ResumeState::Pending));

// Child 1: set phase
let resume_phase_s = resume_complete_s
    .map(|_| Mut::SetPhase(Phase::Running));

// Child 2: resolve active tab (all decision-making here, not in reducer)
let resume_tab_s = resume_complete_s
    .filter_map(|s| resolve_session_active_tab(&s))
    .map(Mut::SetActiveTab);

// Child 3: resolve focus
let resume_focus_s = resume_complete_s
    .map(|s| resolve_focus_slot(&s))
    .map(Mut::SetFocus);
```

The reducer just assigns: `Mut::SetPhase(p) => { s.phase = p; }`, `Mut::SetActiveTab(p) => { s.active_tab = Some(p); }`, etc. Zero logic in the reducer — all decision-making lives in the child chains.

### Principle 3: Max ~3 Lines Per Combinator Closure (Maybe 5 in a Pinch)

Each combinator chain step (filter, map, filter_map, etc.) should be at most ~3 lines. 5 lines is the absolute upper bound for exceptional cases.

When a step is too long, **strongly prefer splitting into more combinator chain steps** — additional `.map()`, `.filter()`, `.filter_map()` calls that each do one small thing. Named helper functions are a fallback, not the first choice. The chain itself should tell the story.

**Bad:**
```rust
.filter_map(|(text, s)| {
    let dims = s.dims?;
    let path = s.active_tab.as_ref()?;
    let buf = s.buffers.get(path)?;
    let mut buf = (**buf).clone();
    action::close_group_on_move(&mut buf);
    buf.clear_mark();
    let (r, c, a) = edit::yank(&mut buf, &text);
    buf.set_cursor(...);
    action::close_group_on_move(&mut buf);
    let (sr, ssl) = mov::adjust_scroll(&buf, &dims);
    buf.set_scroll(...);
    Some(Mut::BufferUpdate(path.clone(), buf))
})
```

**Good — split into chain steps:**
```rust
.filter_map(|(text, s)| Some((text, s.dims?, s.active_tab.clone()?, s)))
.filter_map(|(text, dims, path, s)| Some((text, dims, path, s.buffers.get(&path)?.clone())))
.map(|(text, dims, path, buf)| yank_into_buf(&text, &dims, &path, buf))
```

**Acceptable fallback — named helper fn (must be unit tested):**
```rust
.filter_map(|(text, s)| yank_text(&text, &s))
```

Helper functions extracted from combinator chains must have unit tests. This is the trade-off: chain steps are self-documenting and composable, helper functions are opaque and need tests to verify their behavior.

### Principle 4: Map-Dedupe-Filter (MDF) Ordering

When extracting a value from state, deduping it, and filtering on it, the order must always be **Map, then Dedupe, then Filter**. Filtering before deduping is a common source of bugs — the deduper only sees values that pass the filter, so it misses transitions.

**Bad — filter before dedupe:**
```rust
state
    .filter(|s| s.phase == Phase::Running)  // dedupe never sees non-Running phases
    .dedupe_by(|s| s.phase)                 // so it can't detect the *transition* to Running
    .map(|_| Mut::SomeAction)
```

This fires on every emission where phase happens to be Running, not on the transition to Running.

**Good — map, dedupe, filter:**
```rust
state
    .map(|s| s.phase)                       // extract the value
    .dedupe()                               // suppress consecutive duplicates (sees ALL phases)
    .filter(|phase| *phase == Phase::Running) // now filter — this fires once per transition
    .map(|_| Mut::SomeAction)
```

The dedupe sees every phase change (Begin, Resuming, Running, Exiting, ...) so it correctly detects the *transition* to Running. The filter then picks out only the transition we care about.

### Principle 5: Never Derive from State then Sample State

Never `.map()` a field out of the state stream, transform it, then `.sample_combine()` back with the same state stream. You already had the full state — just use it directly.

**Bad:**
```rust
state
    .map(|s| s.phase)
    .dedupe()
    .filter(|p| *p == Phase::Running)
    .sample_combine(&state)              // pointless: we just came from state
    .map(|(_, s)| do_something(&s))
```

**Good:**
```rust
state
    .dedupe_by(|s| s.phase)
    .filter(|s| s.phase == Phase::Running)
    .map(|s| do_something(&s))           // s is the full AppState already
```

**Also good — map out needed fields first, then dedupe/filter on the tuple (preserves MDF):**
```rust
state
    .map(|s| (s.phase, s.active_tab.clone(), s.dims))  // Map
    .dedupe_by(|t| t.0)                                 // Dedupe
    .filter(|(phase, _, _)| *phase == Phase::Running)   // Filter
    .filter_map(|(_, tab, dims)| Some(compute(&tab?, &dims?)))
```

If you need a derived signal (e.g. from a driver input) combined with current state, `sample_combine` is correct. But deriving from the state stream and then sampling the state stream is circular and wasteful.

### Principle 6: Group Related Streams in `_of` Functions

Logically related streams should be composed together in a dedicated `_of` function that returns a single merged stream. The top-level `model()` function should be short — just wiring `_of` functions together.

**Target structure for `model()`:**
```rust
let workspace_s = workspace_of(&drivers.workspace_in, &state);
let lsp_s = lsp_of(&drivers.lsp_in, &state);
let git_s = git_of(&drivers.git_in, &drivers.workspace_in, &state);
let session_s = session_of(&drivers.workspace_in, &state);
let search_s = search_of(&drivers.file_search_in, &drivers.fs_in, &state);
let editing_s = editing_of(&drivers.clipboard_in, &drivers.syntax_in, &state);

let muts = merge![workspace_s, lsp_s, git_s, session_s, search_s, editing_s, ...];
```

The existing `process_of.rs`, `buffers_of.rs`, `actions_of.rs` are good examples of this pattern.

### Principle 7: Fine-Grained `Mut` Variants

Prefer many small `Mut` variants that each set one or two fields over few large variants that carry lots of data and require complex handling.

**Bad:**
```rust
Mut::SessionRestored {
    active_tab_order: Option<usize>,
    show_side_panel: bool,
    positions: HashMap<...>,
    pending_opens: Vec<...>,
    browser_selected: usize,
    // ... 5 more fields
}
```

**Good:**
```rust
Mut::SetPhase(Phase),
Mut::SetActiveTab(Option<CanonPath>),
Mut::SetFocus(PanelSlot),
Mut::SetShowSidePanel(bool),
Mut::AddTab(CanonPath),
Mut::SetBrowserSelected(usize),
```

### Principle 8: Explicit Post-Fold Invariant Maintenance

Post-reducer invariant logic (like orphaned buffer cleanup) is acceptable but must be a named, focused function — not anonymous code tacked onto the end of a match block.

```rust
muts.fold_into(&state, init, |s, m| {
    apply_mut(&mut s, m);       // trivial field assignments per Principle 1
    normalize_buffers(&mut s);  // explicit invariant maintenance
    s
});
```

### Principle 9: Dissolve `Mut::Action` — No Mega-Dispatchers

The `Mut::Action(Action)` pattern that delegates to a giant `handle_action()` function defeats FRP. Each `Action` variant should become its own stream chain producing fine-grained `Mut`s.

**Bad:**
```rust
Mut::Action(a) => {
    action::handle_action(&mut s, a);  // 583 lines of imperative code
}
```

**Good:**
```rust
// In editing_of.rs or per-domain _of files:
let insert_char_s = actions
    .filter_map(|a| match a { Action::InsertChar(ch) => Some(ch), _ => None })
    .sample_combine(&state)
    .map(|(ch, s)| insert_char(&s, ch));  // returns Mut::BufferUpdate

let move_up_s = actions
    .filter(|a| matches!(a, Action::MoveUp))
    .sample_combine(&state)
    .filter_map(|(_, s)| compute_move_up(&s));
```

### Principle 10: No `flat_map` to `Vec` — Avoid Allocation for Multi-Emit

The current `flat_map`/`flatten` requires collecting into `Vec` which forces allocation. Avoid this pattern. If multi-emit is needed, either:
- Fix the framework to accept iterators
- Split into separate streams that each emit one value
- Use a different composition approach

**Bad:**
```rust
.flat_map(|s| {
    s.buffers.values()
        .filter(...)
        .map(Mut::Something)
        .collect::<Vec<_>>()  // forced allocation
})
```

**Good:** Split into separate streams, or fix `flat_map` to work with iterators.

## Key Combinators (rx.rs)

The FRP framework in `crates/core/src/rx.rs` provides:

- `Stream<T>` — push-based reactive stream
- `MemoryStream<T>` — stream that caches the last value
- `.map()`, `.filter()`, `.filter_map()` — standard transforms
- `.flat_map()` — one-to-many (currently requires `IntoIterator`, see Principle 8)
- `.sample_combine(&other)` — when self emits, pair with latest value from other
- `.dedupe()`, `.dedupe_by()` — suppress consecutive duplicates
- `.fold()`, `.fold_into()` — accumulate state
- `.forward(&target)` — fan-in: forward all values from self into target stream
- `.inspect()` — side-effect passthrough (for logging etc.)
- `.remember()`, `.start_with()` — convert to memory stream
- `.flatten_switch()` — switch to latest inner stream
- `combine!()` macro — combine multiple streams into tuple
- `Stream::new()` + `.forward()` — manual merge point

## Architectural Layers

```
main.rs          — wires drivers, calls model(), calls derived()
model/mod.rs     — model() function: assembles streams, fold_into produces state
model/*_of.rs    — per-domain stream assembly (actions_of, buffers_of, process_of, etc.)
derived.rs       — derived(): produces driver output streams from state
crates/state/    — AppState struct and sub-state types
crates/core/rx   — the FRP framework
crates/*/        — driver implementations (lsp, workspace, ui, syntax, etc.)
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/algesten) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
