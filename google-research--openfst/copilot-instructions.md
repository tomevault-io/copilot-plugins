## openfst

> The OpenFst Libary has various conventions and assumptions about its objects and

# OpenFst Conventions

The OpenFst Libary has various conventions and assumptions about its objects and
coding style.

## Object Conventions

1.  The `StateIds` of an FST are dense integers, numbered from 0 to
    `NumStates()` - 1.
2.  A `StateIterator` returns `StateIds` in numerical order.
3.  A user may not request info about a `StateId` **s** from an FST unless the
    FST has already returned a `StateId` **t** $\ge$ **s** (e.g. from
    `Start()`, `NumStates()`, `StateIterator`, or
    `ArcIterator->Value().nextstate`).
4.  The empty machine (no states) has start state `kNoStateId`; a non-empty
    machine (has states) has a valid start state.
5.  A `Label` is a non-negative integer except `kNoLabel` (or library
    internals). The label `0` is reserved for [epsilon](glossary.md#epsilon).
6.  A `Weight` satisfies the properties described
    [here](weight_requirements.md).
7.  A `StateIterator` is invalidated if the number of states is modified.
8.  An `ArcIterator` for a state is invalidated by any mutation of the arcs at
    that state.
9.  A `MutableArcIterator` is invalidated by any mutation of the arcs at that
    state other than by the iterator itself.
10. A `Matcher` is invalidated by any mutation of an FST.
11. A reference/pointer to an arc is invalidated at the next operation on an
    FST, state iterator, arc iterator, or matcher.
12. State iterators, arc iterators and matchers should be destroyed prior to
    destroying their component FSTs.
13. All FST classes F implement a copy constructor `F(const &F)`.
14. The copy constructor and `Copy()` method of an FST have constant time and
    space complexity (shallow copy) unless otherwise noted.
15. One or more stored FST [properties](advanced_usage.md#properties) may be set
    to unknown once an FST is mutated depending on the type of mutation.

## Coding Conventions

### FST Application Code:

1.  Your code should rarely use `int` or `float` when referring to FST
    components. Use:

*   `StateId` for state Ids and number of states.
*   `Label` for labels.
*   `size_t` for other array lengths.
*   `Weight` for weights.

### New FST classes:

1.  All FST classes will implement working versions of all pure virtual member
    functions; writing dummy or error-raising versions is not permitted.
2.  If class `C` is templated on an Arc, then `C::Arc` will give that type.

---
> Source: [google-research/openfst](https://github.com/google-research/openfst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
