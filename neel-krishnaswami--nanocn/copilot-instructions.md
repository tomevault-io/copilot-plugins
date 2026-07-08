## nanocn

> Instructions to Claude for writing OCaml code:

Instructions to Claude for writing OCaml code: 

0. When writing code, do these things first: 

   1. The Ott file (`doc/syntax.ott`) is the source of truth for the language. ANY
      change to syntax or typing rules MUST be made there FIRST, before modifying the
      implementation. 

   2. Write a plan with high-level architectural decisions. Analyze this plan for defects, 
      and keep fixing them until no obvious deficiencies remain. 

   3. Write a detailed design document, with design choices for each module. Again, before
      proceeding to implementaiton, analyze the design for obvious flaws before proceeding. 
      If there is a fundamental design problem, DO NOT try to smooth it over. Instead, consult
      the user about how to proceed, giving them the key options. 

   4. If the detailed design reveals a key flaw, consider whether the high-level plan needs
      to be revised. Consult the user about how to proceed, and give them some options. 

   5. Copy each design document to a file `doc/history/PLAN-NAME.md`, so the user can read it,
      and new sessions can understand the changes. 

1. When you get stuck or make a mistake, output your current hypothesis before switching to a new
   approach or making any changes. 

2. Programs should be composed of small modules, each implementing a single concern or
   data structure. However, mutual recursion between functions is a good reason to
   place them in the same module — prefer a single larger module with `and`-linked
   mutually recursive functions over separate modules connected by callbacks or
   recursive module declarations.

3. Write .mli files first, before writing any part of a module.

   - .mli files should emphasize the algebraic structure of the data structure. 

   - Name the primary type of an .mli file as `t`, so that clients can refer to it as 
     `Foo.t`, or `'a Foo.t`. 

   - Unless required, hide the implementation type so client code is
     programmed against abstract types.

   - Parameterized types of the form `'a t` should expose a `map : ('a -> 'b) -> 'a t -> 'b t`
     primitive in their interface. 

   - If a type constructor has monadic structure, then define `return : 'a -> 'a t` and
     `(let+) : 'a -> ('a -> 'b t) -> 'b t` operations.

   - If a type can be ordered, then expose a `compare : t -> t > int` primitive. 

   - If a type can be printed, expose a `print : Format.formatter -> t -> unit` method in the 
     interface. Use the Format module's indentation directives to ensure that print methods are
     nicely laid out.

   - If a module exposes a parameterized type, give parameterized comparison and printing
     functions. 

   - Every module should have property-based tests using `QCheck`, based on the
     invariants of the module. In particular, if a specification file lists some expected
     properties or theorems, turn these into property-based tests. In a Test submodule, expose
     generators, properties, and a test field using the QCheck library. 

4. Here are some bad language features to avoid: 

   - NEVER use Obj.magic, or any other feature which can break type safety. 

   - NEVER use generic equality, since this violates data abstraction. Always use a 
     type-specific `compare` operation.

5. Unless explicitly instructed otherwise, DO NOT write code which uses effects.

   - Use a monad with a result type instead of exceptions. 

   - Prefer monadic state-passing to mutable data structures. 

   - Permission to use mutable data structures is granted on a per-module basis, and 
     permission in one module does not grant it in any other. 

   - Do not perform IO operations, except in the top-level main function. 

6. Write programs by pattern matching over data structures. Avoid
   using partial accessors or incomplete patterns matches.
  
7. Higher-order functions should be used sparingly, in idiomatic ways.

   - Introducing monadic code to eliminate repeated nested pattern matches is acceptable. 
   - Use of map, filter, and other algebraically well-behaved functions is acceptable. 
   - Avoid the use of folds, because they offer no reasoning advantages over explicit
     structural recursion.

8. When making any changes to the grammar, make sure to use the nanocn-parse-errors skill. 

9. `invariant_at` / `K_internal_invariant` must only be used for conditions that are genuinely
   unreachable from any user program, no matter how ill-typed. If a user can write a program
   (even a nonsensical one) that triggers the error, it must use a proper error kind with a
   helpful message. The parser does not reject all ill-formed refined programs, so assume any
   syntactically valid program can reach the typechecker. See `doc/errors/type-errors.md` for
   error message guidelines.

---
> Source: [neel-krishnaswami/nanocn](https://github.com/neel-krishnaswami/nanocn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
