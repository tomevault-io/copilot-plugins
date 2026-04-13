## library-demo

> Code Design Principles


Code Design Principles
This list has been carefully curated so please take the time to read it in its entirety

Keep it simple, stupid (KISS)
Simple code is easier to understand and maintain
Only add complexity when it is warranted
You aren't gonna need it (YAGNI)
Don’t add functionality until it is needed
https://martinfowler.com/bliki/Yagni.html

“Yagni only applies to capabilities built into the software to support a presumptive feature, it does not apply to effort to make the software easier to modify”
Single responsibility principle (SRP)
“Gather together the things that change for the same reasons. Separate those things that change for different reasons.”

Each component (function, class, file, package, etc.) should have a single, clearly defined responsibility
Don’t repeat yourself (DRY)
“Every piece of knowledge must have a single, unambiguous, authoritative representation within a system” - The Pragmatic Programmer

Don’t have multiple sources of truth (e.g. two separate components / systems that need to be kept in sync because the state they manage overlaps)
Extract common functionality into reusable components
Generally, can follow the rule of three, i.e. refactor to avoid duplication after 3 instances of use

Name things well
A reader should be able to understand the purpose of a function or a variable just by knowing its name
Keep interfaces small for all types of components
Large interfaces often signal a violation of SRP / KISS
They also make it extremely difficult to maintain / refactor code (e.g. ContextModule)

Document public interfaces
e.g. public classes, public methods, proto messages, etc.

Make the types and interfaces enforce the invariants
As much as possible the type system should be used to enforce behavior / expectations instead of relying on written (or unwritten) documentation of invariants
e.g. instead of having two parallel arrays that implicitly represent two fields from a list of objects, have a list of objects that each have the two fields
e.g. a function’s required parameters should be the inputs that are strictly required in all use cases, if some inputs are needed only some of the time make them optional
e.g. if a string should always be one of a fixed set of values, use an enum

Make things private until they should not be (which may be never)
Ideally there are roughly two types of objects
Data objects with public fields that are used to pass data around (e.g. configs)
Actual classes with private fields and a public interface of methods (and also private internal methods)
Don’t expose state / functionality publicly if they are not part of the component’s responsibility / interface
Don’t expose implementation-specific state as it harder to change the implementation in the future
Don’t expose utilities a component uses to but aren’t meant to provide to others (e.g. don’t provide the ApiServerClient through the CascadeManager since it has nothing to do with its responsibilities)


Test behavior, not implementation details
Split large blocks of functionality into separately testable functions

Functions should properly distinguish between required and optional arguments

Avoid having functions with many arguments that are almost always nil or empty

This is likely an indication that the arguments are optional so providing them to the function should be optional

In Go, use the options pattern for optional arguments (options can either modify the struct directly for a constructor or modify a configuration struct)
As you’re editing (especially deleting) code, look out for unused code
e.g. if you’re removing usage of a constant, field, or function check if it’s still needed at all and remove it if not

Avoid Cargo Culting
Cargo culting indicates copying some code or functionality that serves no actual purpose in the new location
This is indicative of lack of understanding of the intent / impact of the code and whether it is required
You should understand the code you are adding and even when following pre-established practices you should understand why those practices exist

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chrisgroks) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
